# MON-012: App Store IAP Integration Specification

## Overview

This document specifies the integration with Apple's App Store for in-app purchases (IAP) in Plunderstorm Mobile, covering StoreKit implementation, receipt validation, and purchase flow handling.

## Design Philosophy

IAP integration must be reliable, secure, and compliant with Apple's guidelines. The system prioritizes user trust through proper purchase handling, clear communication, and robust error recovery. Server-side validation prevents fraud while maintaining a smooth purchase experience.

---

## 1. Product Types

### 1.1 Consumable Products
```
Usage: Currency packs that can be purchased multiple times

Products:
├── doubloons_100     - 100 Doubloons
├── doubloons_550     - 550 Doubloons (+10% bonus)
├── doubloons_1200    - 1,200 Doubloons (+20% bonus)
├── doubloons_2500    - 2,500 Doubloons (+25% bonus)
├── doubloons_6500    - 6,500 Doubloons (+30% bonus)

Behavior:
├── Can purchase unlimited times
├── Delivered immediately
├── Cannot be restored
└── Receipt must be validated each time
```

### 1.2 Non-Consumable Products
```
Usage: One-time purchases permanently owned

Products:
├── starter_pack      - Starter Pack ($4.99)
├── founders_pack     - Founder's Pack (if applicable)

Behavior:
├── Can only purchase once
├── Persists forever
├── Can be restored on new device
└── Check ownership before offering
```

### 1.3 Auto-Renewable Subscriptions (Future)
```
Usage: VIP membership or similar

Products:
├── vip_monthly       - VIP Monthly ($4.99/month)
├── vip_annual        - VIP Annual ($39.99/year)

Behavior:
├── Auto-renews until cancelled
├── Apple handles billing
├── Server notified via webhook
└── Grace period for failed payments
```

---

## 2. App Store Connect Configuration

### 2.1 Product Setup
```
For Each Product:
├── Product ID: Unique identifier (com.company.game.product_id)
├── Reference Name: Internal name for tracking
├── Type: Consumable / Non-Consumable / Subscription
├── Price: Select price tier
├── Availability: Select regions
├── Display Name: Localized name (all languages)
├── Description: Localized description
└── Screenshot: Required for review
```

### 2.2 Pricing Tiers
| Product | Price Tier | USD | EUR | GBP |
|---------|------------|-----|-----|-----|
| doubloons_100 | Tier 1 | $0.99 | €0.99 | £0.79 |
| doubloons_550 | Tier 5 | $4.99 | €4.99 | £4.49 |
| doubloons_1200 | Tier 10 | $9.99 | €9.99 | £8.99 |
| doubloons_2500 | Tier 20 | $19.99 | €19.99 | £17.99 |
| doubloons_6500 | Tier 50 | $49.99 | €49.99 | £44.99 |
| starter_pack | Tier 5 | $4.99 | €4.99 | £4.49 |
| battle_pass | Tier 10 | $9.99 | €9.99 | £8.99 |

### 2.3 Sandbox Configuration
```
Test Accounts:
├── Create sandbox tester accounts
├── Use different email from production
├── Configure test payment methods
├── Enable receipt validation in sandbox
└── Document test account credentials
```

---

## 3. StoreKit Implementation

### 3.1 StoreKit 2 (iOS 15+)
```swift
// Product loading
func loadProducts() async {
    do {
        let productIds = Set([
            "com.company.game.doubloons_100",
            "com.company.game.doubloons_550",
            // ... other products
        ])

        let products = try await Product.products(for: productIds)

        for product in products {
            // Cache product info
            productCache[product.id] = product
        }
    } catch {
        // Handle error
        logError("Failed to load products: \(error)")
    }
}

// Purchase flow
func purchase(_ productId: String) async throws -> Transaction {
    guard let product = productCache[productId] else {
        throw PurchaseError.productNotFound
    }

    let result = try await product.purchase()

    switch result {
    case .success(let verification):
        let transaction = try checkVerified(verification)
        // Send to server for validation
        try await validateOnServer(transaction)
        await transaction.finish()
        return transaction

    case .pending:
        throw PurchaseError.pending

    case .userCancelled:
        throw PurchaseError.cancelled

    @unknown default:
        throw PurchaseError.unknown
    }
}
```

### 3.2 StoreKit 1 Fallback (iOS 14 and earlier)
```swift
// For devices on iOS 14 or earlier
class StoreKitDelegate: NSObject, SKProductsRequestDelegate, SKPaymentTransactionObserver {

    func productsRequest(_ request: SKProductsRequest, didReceive response: SKProductsResponse) {
        // Cache products
        for product in response.products {
            productCache[product.productIdentifier] = product
        }
    }

    func paymentQueue(_ queue: SKPaymentQueue, updatedTransactions transactions: [SKPaymentTransaction]) {
        for transaction in transactions {
            switch transaction.transactionState {
            case .purchased:
                handlePurchased(transaction)
            case .restored:
                handleRestored(transaction)
            case .failed:
                handleFailed(transaction)
            case .deferred:
                handleDeferred(transaction)
            case .purchasing:
                break
            @unknown default:
                break
            }
        }
    }
}
```

### 3.3 Transaction Listening
```swift
// Listen for transactions in background
func startTransactionListener() {
    Task {
        for await result in Transaction.updates {
            do {
                let transaction = try checkVerified(result)
                // Handle transaction (may be from previous session)
                await handleTransaction(transaction)
                await transaction.finish()
            } catch {
                logError("Transaction verification failed: \(error)")
            }
        }
    }
}
```

---

## 4. Purchase Flow

### 4.1 Complete Purchase Flow
```
1. User taps "Buy" in UI
2. Client checks if product loaded
3. Client initiates purchase with StoreKit
4. System shows Apple payment sheet
5. User authenticates (Face ID, password, etc.)
6. Apple processes payment
7. StoreKit returns transaction
8. Client sends receipt to server
9. Server validates receipt with Apple
10. Server grants content (Doubloons)
11. Server confirms to client
12. Client updates UI
13. Client finishes transaction

Error Handling at Each Step:
├── Product not found → Show error, reload products
├── User cancelled → Silent, no error shown
├── Payment failed → Show Apple's error message
├── Receipt invalid → Contact support prompt
├── Server error → Retry with exponential backoff
├── Timeout → Queue for retry
└── Already purchased (non-consumable) → Inform user
```

### 4.2 Flow Diagram
```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  User   │     │  Client │     │  Apple  │     │  Server │
└────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘
     │               │               │               │
     │──Buy Button──>│               │               │
     │               │──Purchase────>│               │
     │               │               │               │
     │<──Payment Sheet──────────────>│               │
     │               │               │               │
     │──Confirm─────>│               │               │
     │               │<──Transaction─│               │
     │               │               │               │
     │               │──Validate───────────────────>│
     │               │               │               │
     │               │               │──Verify──────>│
     │               │               │<──Result──────│
     │               │               │               │
     │               │<──Grant Doubloons────────────│
     │               │               │               │
     │<──Success─────│               │               │
     │               │               │               │
```

---

## 5. Receipt Validation

### 5.1 Server-Side Validation Flow
```
Server receives receipt:
1. Decode base64 receipt data
2. Send to Apple's verifyReceipt endpoint
3. Parse Apple's response
4. Extract transaction details
5. Verify product ID matches expected
6. Check transaction hasn't been used before
7. Grant content to player
8. Store transaction record
9. Return success to client
```

### 5.2 Apple Verification Endpoint
```
Production: https://buy.itunes.apple.com/verifyReceipt
Sandbox: https://sandbox.itunes.apple.com/verifyReceipt

Request:
{
  "receipt-data": "<base64-encoded-receipt>",
  "password": "<shared-secret>",
  "exclude-old-transactions": true
}

Response (Success):
{
  "status": 0,
  "receipt": {
    "bundle_id": "com.company.game",
    "in_app": [
      {
        "product_id": "com.company.game.doubloons_550",
        "transaction_id": "1000000012345678",
        "purchase_date_ms": "1638316800000",
        ...
      }
    ]
  },
  "environment": "Production"
}
```

### 5.3 Status Codes
| Code | Meaning | Action |
|------|---------|--------|
| 0 | Valid receipt | Process transaction |
| 21000 | App Store unavailable | Retry later |
| 21002 | Receipt data malformed | Client error, reject |
| 21003 | Receipt not authenticated | Possible fraud |
| 21004 | Shared secret mismatch | Server config error |
| 21005 | Server unavailable | Retry later |
| 21006 | Subscription expired | Handle subscription |
| 21007 | Sandbox receipt to production | Use sandbox URL |
| 21008 | Production receipt to sandbox | Use production URL |
| 21010 | Account does not exist | Possible fraud |

### 5.4 Retry Logic
```
On Network/Server Error:
├── Attempt 1: Immediate
├── Attempt 2: Wait 2 seconds
├── Attempt 3: Wait 4 seconds
├── Attempt 4: Wait 8 seconds
├── Attempt 5: Wait 16 seconds
└── Final: Store receipt, manual review

On Status 21007/21008 (Wrong Environment):
├── Automatically retry with other endpoint
└── Log for investigation (shouldn't happen in production)
```

---

## 6. Restore Purchases

### 6.1 Restore Flow
```swift
func restorePurchases() async {
    do {
        // StoreKit 2
        for await result in Transaction.currentEntitlements {
            let transaction = try checkVerified(result)

            // Only process non-consumables and subscriptions
            if transaction.productType == .nonConsumable ||
               transaction.productType == .autoRenewable {
                await restoreTransaction(transaction)
            }
        }

        showRestoreSuccess()
    } catch {
        showRestoreError(error)
    }
}
```

### 6.2 Restore UI
```
Restore Button:
├── Location: Settings → Account → Restore Purchases
├── Also accessible from Store if prompted
├── Show progress indicator during restore
├── List items restored (or "No purchases to restore")
└── Handle errors gracefully
```

---

## 7. Error Handling

### 7.1 User-Facing Errors
| Error Type | Message | Action |
|------------|---------|--------|
| Cancelled | (None) | Silent dismissal |
| Network | "Unable to connect. Please check your internet." | Retry option |
| Payment Failed | "Payment could not be processed." | Contact support |
| Already Owned | "You already own this item." | Dismiss |
| Not Allowed | "Purchases are restricted on this device." | Settings link |
| Server Error | "Please try again later." | Retry option |

### 7.2 Error Logging
```
Log for All Errors:
├── Error code and description
├── Product ID attempted
├── Player ID
├── Device info
├── Timestamp
├── Receipt data (if applicable)
├── Server response (if applicable)
└── Stack trace
```

---

## 8. Testing

### 8.1 Sandbox Testing
```
Test Scenarios:
├── Successful purchase (all products)
├── User cancellation
├── Payment failure
├── Network interruption mid-purchase
├── App crash during purchase
├── Purchase restoration
├── Multiple rapid purchases
├── Purchase on different devices
└── Receipt validation edge cases
```

### 8.2 Test Account Setup
```
Create Test Accounts:
├── Different regions (US, EU, Asia)
├── Different payment states
├── Account with parental controls
├── Account requiring password always
└── Document credentials securely
```

### 8.3 StoreKit Testing (Xcode)
```
Local Testing:
├── Use StoreKit Configuration file
├── Test without network
├── Simulate error conditions
├── Test transaction interruptions
├── Verify UI behavior
└── Performance testing
```

---

## 9. Compliance

### 9.1 Apple Guidelines
```
Requirements:
├── Use Apple's payment system (no alternatives)
├── Show prices in user's currency
├── Clear purchase descriptions
├── Restore purchases option
├── No fake IAP buttons
├── Handle family sharing (if applicable)
├── Age-appropriate content
└── Clear privacy policy
```

### 9.2 App Review Considerations
```
Common Rejection Reasons:
├── Missing restore purchases button
├── Unclear product descriptions
├── Prices not matching App Store
├── Gambling mechanics without disclosure
├── Manipulative purchase UI
└── Not using IAP for digital content

Prevention:
├── Follow HIG for purchase UI
├── Clear pricing display
├── Prominent restore option
├── Honest descriptions
└── Test on actual devices
```

---

## 10. Integration Points

### 10.1 System Dependencies
| System | Integration |
|--------|-------------|
| Currency System | Grant Doubloons |
| Inventory System | Grant items (starter pack) |
| Server | Receipt validation |
| Analytics | Purchase events |
| Account | Link purchases to player |

### 10.2 Events
```swift
// Purchase events for analytics and UI
protocol IAPEventDelegate {
    func purchaseStarted(productId: String)
    func purchaseCompleted(productId: String, transactionId: String)
    func purchaseFailed(productId: String, error: Error)
    func purchaseCancelled(productId: String)
    func purchaseRestored(productId: String)
}
```

---

## 11. Default Configuration

```json
{
  "storekit": {
    "productPrefix": "com.company.plunderstorm",
    "sharedSecretEnvVar": "APPLE_SHARED_SECRET",
    "validationEndpoint": "/api/v1/iap/apple/validate",
    "retryAttempts": 5,
    "retryBaseDelay": 2000,
    "sandboxEnabled": true
  },
  "products": {
    "consumables": [
      "doubloons_100",
      "doubloons_550",
      "doubloons_1200",
      "doubloons_2500",
      "doubloons_6500"
    ],
    "nonConsumables": [
      "starter_pack",
      "battle_pass_s1"
    ]
  },
  "ui": {
    "showPricesFromStore": true,
    "cacheProductsFor": 3600,
    "purchaseTimeout": 60000
  }
}
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
