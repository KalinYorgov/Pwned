# MON-015: Purchase Verification Service Specification

## Overview

This document specifies the server-side purchase verification service for Plunderstorm Mobile, handling receipt/token validation for both iOS and Android to prevent fraud and ensure secure content delivery.

---

## 1. Architecture

### 1.1 Service Overview
```
┌─────────┐     ┌──────────────────┐     ┌────────────────┐
│  Client │────>│ Verification     │────>│ Apple/Google   │
│  (App)  │<────│ Service          │<────│ APIs           │
└─────────┘     └──────────────────┘     └────────────────┘
                       │
                       ▼
                ┌──────────────────┐
                │ Database         │
                │ - Transactions   │
                │ - Used receipts  │
                └──────────────────┘
```

### 1.2 Responsibilities
```
Verification Service:
├── Validate iOS receipts with Apple
├── Validate Android tokens with Google
├── Prevent duplicate content delivery
├── Grant content on successful validation
├── Log all transactions
├── Handle refunds (webhooks)
└── Fraud detection and alerting
```

---

## 2. API Endpoints

### 2.1 iOS Validation
```
POST /api/v1/iap/apple/validate

Request:
{
  "playerId": "uuid",
  "receiptData": "<base64-encoded-receipt>",
  "productId": "doubloons_550",
  "transactionId": "1000000012345678",  // Optional, for idempotency
  "sandbox": false  // Environment hint
}

Response (Success):
{
  "success": true,
  "transactionId": "1000000012345678",
  "productId": "doubloons_550",
  "granted": {
    "doubloons": 550
  },
  "newBalance": {
    "doubloons": 1050
  }
}

Response (Error):
{
  "success": false,
  "error": "INVALID_RECEIPT",
  "message": "Receipt validation failed"
}
```

### 2.2 Android Validation
```
POST /api/v1/iap/google/validate

Request:
{
  "playerId": "uuid",
  "purchaseToken": "<purchase-token>",
  "productId": "doubloons_550",
  "orderId": "GPA.1234-5678-9012-34567"  // For idempotency
}

Response (Success):
{
  "success": true,
  "orderId": "GPA.1234-5678-9012-34567",
  "productId": "doubloons_550",
  "granted": {
    "doubloons": 550
  },
  "newBalance": {
    "doubloons": 1050
  }
}
```

---

## 3. Validation Flow

### 3.1 iOS Validation Flow
```
1. Receive receipt from client
2. Check if transaction ID already processed (idempotency)
3. Call Apple's verifyReceipt endpoint:
   - Production: https://buy.itunes.apple.com/verifyReceipt
   - Sandbox: https://sandbox.itunes.apple.com/verifyReceipt
4. If status 21007, retry with sandbox URL
5. If status 21008, retry with production URL
6. Parse response, extract latest transaction
7. Verify product ID matches
8. Verify bundle ID matches
9. Begin database transaction:
   a. Mark receipt/transaction as used
   b. Grant content to player
   c. Log transaction
10. Commit transaction
11. Return success to client
```

### 3.2 Android Validation Flow
```
1. Receive purchase token from client
2. Check if order ID already processed (idempotency)
3. Get OAuth access token (cached or refresh)
4. Call Google Play Developer API:
   GET /androidpublisher/v3/applications/{package}/
       purchases/products/{product}/tokens/{token}
5. Verify response:
   - purchaseState == 0 (purchased)
   - orderId matches
   - Product ID matches
6. Begin database transaction:
   a. Mark token as used
   b. Grant content to player
   c. Log transaction
7. Commit transaction
8. Return success to client
```

---

## 4. Fraud Prevention

### 4.1 Duplicate Prevention
```sql
-- Used receipts table
CREATE TABLE used_receipts (
  id SERIAL PRIMARY KEY,
  platform VARCHAR(10) NOT NULL,  -- 'ios' or 'android'
  receipt_hash VARCHAR(64) NOT NULL,  -- SHA256 of receipt/token
  transaction_id VARCHAR(100) NOT NULL,
  player_id UUID NOT NULL,
  product_id VARCHAR(100) NOT NULL,
  amount_granted JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(platform, transaction_id)
);

CREATE INDEX idx_receipt_hash ON used_receipts(receipt_hash);
CREATE INDEX idx_transaction_id ON used_receipts(transaction_id);
```

### 4.2 Validation Checks
```
Pre-Grant Checks:
├── Transaction ID not in used_receipts
├── Receipt hash not in used_receipts
├── Product ID exists in catalog
├── Player account exists and active
├── Environment matches (prod/sandbox)
└── Receipt timestamp is recent (< 7 days)

Anomaly Detection:
├── Multiple purchases in short time
├── Purchase from suspicious IP
├── Receipt from different region than account
├── Modified/jailbroken device indicators
└── Velocity checks (purchases per hour/day)
```

### 4.3 Fraud Response
```
On Suspected Fraud:
├── Log full request details
├── Don't grant content
├── Return generic error to client
├── Alert fraud team
├── Consider account review

On Confirmed Fraud:
├── Revoke granted content
├── Flag account
├── Consider ban
└── Report to platform (if applicable)
```

---

## 5. Content Granting

### 5.1 Grant Logic
```python
def grant_purchase_content(player_id, product_id, transaction_id):
    product = get_product_config(product_id)

    with database.transaction():
        # Check idempotency
        if is_transaction_processed(transaction_id):
            return get_previous_result(transaction_id)

        # Grant content based on product type
        if product.type == 'consumable':
            grant_currency(player_id, product.grant.doubloons)

        elif product.type == 'non_consumable':
            if product.oneTime and has_purchased(player_id, product_id):
                raise AlreadyOwnedException()

            grant_items(player_id, product.grant.items)
            grant_currency(player_id, product.grant.doubloons)
            mark_as_owned(player_id, product_id)

        # Record transaction
        record_transaction(player_id, product_id, transaction_id)

        return get_player_balance(player_id)
```

### 5.2 Atomic Operations
```
All grants must be atomic:
├── Use database transactions
├── All-or-nothing (no partial grants)
├── Record before commit
├── Retry on transient failures
└── Manual recovery for edge cases
```

---

## 6. Refund Handling

### 6.1 Apple Server Notifications
```
Setup:
├── Configure URL in App Store Connect
├── Enable server-to-server notifications
├── Handle notification types

Notification Types:
├── REFUND: User refunded purchase
├── CONSUMPTION_REQUEST: Apple reviewing
├── REVOKE: Family sharing revoked

On REFUND:
├── Lookup original transaction
├── Calculate content to revoke
├── Deduct currency (may go negative)
├── Log refund
├── Alert if suspicious pattern
```

### 6.2 Google Real-Time Notifications
```
Setup:
├── Configure Cloud Pub/Sub topic
├── Grant publisher access to Google
├── Subscribe to notifications

Notification Types:
├── SUBSCRIPTION_PURCHASED
├── SUBSCRIPTION_RENEWED
├── SUBSCRIPTION_CANCELED
├── SUBSCRIPTION_REVOKED

On Revocation:
├── Similar to Apple refund handling
├── Deduct content
├── Log event
└── Alert if needed
```

---

## 7. Logging and Monitoring

### 7.1 Transaction Logs
```json
{
  "logId": "uuid",
  "timestamp": "2025-12-02T14:30:00Z",
  "type": "PURCHASE_VALIDATED",
  "platform": "ios",
  "playerId": "player-uuid",
  "productId": "doubloons_550",
  "transactionId": "1000000012345678",
  "amount": {
    "doubloons": 550
  },
  "balanceBefore": {
    "doubloons": 500
  },
  "balanceAfter": {
    "doubloons": 1050
  },
  "validationDuration": 234,
  "clientIp": "1.2.3.4",
  "deviceInfo": {
    "model": "iPhone14,2",
    "os": "iOS 17.0"
  }
}
```

### 7.2 Metrics
```
Track:
├── Validation success rate
├── Validation latency (p50, p95, p99)
├── Platform API availability
├── Fraud detection rate
├── Revenue by product
├── Refund rate
└── Errors by type
```

### 7.3 Alerts
```
Alert On:
├── Validation success rate < 95%
├── Validation latency p95 > 5s
├── Apple/Google API errors > 10/min
├── Suspected fraud > 5/hour
├── Unusual refund volume
└── Database connection issues
```

---

## 8. Error Handling

### 8.1 Error Codes
| Code | Description | Client Action |
|------|-------------|---------------|
| INVALID_RECEIPT | Receipt validation failed | Contact support |
| ALREADY_PROCESSED | Transaction already granted | Show success |
| PRODUCT_NOT_FOUND | Unknown product ID | Update client |
| PLAYER_NOT_FOUND | Invalid player ID | Re-authenticate |
| PLATFORM_ERROR | Apple/Google API error | Retry later |
| INTERNAL_ERROR | Server error | Retry later |
| FRAUD_SUSPECTED | Blocked for review | Contact support |

### 8.2 Retry Strategy
```
Client Retry:
├── Attempt 1: Immediate
├── Attempt 2: 2 seconds
├── Attempt 3: 5 seconds
├── Attempt 4: 10 seconds
├── Final: Store for background retry

Server Retry (Platform API):
├── Attempt 1: Immediate
├── Attempt 2: 1 second
├── Attempt 3: 2 seconds
└── Fail: Return PLATFORM_ERROR
```

---

## 9. Configuration

```json
{
  "verification": {
    "apple": {
      "productionUrl": "https://buy.itunes.apple.com/verifyReceipt",
      "sandboxUrl": "https://sandbox.itunes.apple.com/verifyReceipt",
      "sharedSecretEnv": "APPLE_SHARED_SECRET",
      "bundleId": "com.company.plunderstorm",
      "timeoutMs": 10000,
      "maxRetries": 3
    },
    "google": {
      "packageName": "com.company.plunderstorm",
      "serviceAccountEnv": "GOOGLE_SERVICE_ACCOUNT_JSON",
      "timeoutMs": 10000,
      "maxRetries": 3
    },
    "fraud": {
      "maxPurchasesPerHour": 10,
      "receiptMaxAgeDays": 7,
      "enableVelocityChecks": true
    },
    "logging": {
      "logAllRequests": true,
      "logReceiptData": false,
      "retentionDays": 365
    }
  }
}
```

---

## 10. Testing

### 10.1 Test Scenarios
```
Functional Tests:
├── Valid iOS receipt → Success
├── Valid Android token → Success
├── Invalid receipt → Rejection
├── Duplicate transaction → Idempotent success
├── Unknown product → Error
├── Platform API timeout → Retry + error
├── Database failure → Rollback
└── Concurrent same transaction → Only one grant
```

### 10.2 Load Testing
```
Targets:
├── 100 validations/second sustained
├── p99 latency < 3 seconds
├── Zero duplicate grants under load
└── Graceful degradation on overload
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
