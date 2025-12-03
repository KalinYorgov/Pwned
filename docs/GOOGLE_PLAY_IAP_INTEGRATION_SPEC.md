# MON-013: Google Play IAP Integration Specification

## Overview

This document specifies the integration with Google Play Billing for in-app purchases in Plunderstorm Mobile on Android, covering billing library implementation, purchase token validation, and purchase flow handling.

## Design Philosophy

Google Play Billing must be implemented reliably with proper handling of all purchase states. The system uses server-side validation to prevent fraud and handles Android-specific scenarios like pending purchases and promo codes.

---

## 1. Product Types

### 1.1 In-App Products (One-Time)
```
Consumable Products:
├── doubloons_100     - 100 Doubloons ($0.99)
├── doubloons_550     - 550 Doubloons ($4.99)
├── doubloons_1200    - 1,200 Doubloons ($9.99)
├── doubloons_2500    - 2,500 Doubloons ($19.99)
├── doubloons_6500    - 6,500 Doubloons ($49.99)

Non-Consumable Products:
├── starter_pack      - Starter Pack ($4.99)
├── battle_pass_s1    - Battle Pass Season 1 ($9.99)
```

### 1.2 Subscriptions (Future)
```
Auto-Renewing Subscriptions:
├── vip_monthly       - VIP Monthly ($4.99/month)
├── vip_annual        - VIP Annual ($39.99/year)
```

---

## 2. Google Play Console Configuration

### 2.1 Product Setup
```
For Each Product:
├── Product ID: Unique identifier (doubloons_100)
├── Product Type: Managed product / Subscription
├── Default Price: Set base price
├── Pricing Template: Auto-convert or manual per country
├── Title: Localized product name
├── Description: Localized description
├── Status: Active / Inactive
└── Linked to app: Select APK/AAB
```

### 2.2 Pricing Configuration
| Product | Base Price | Notes |
|---------|------------|-------|
| doubloons_100 | $0.99 | Auto-converted prices |
| doubloons_550 | $4.99 | +10% bonus |
| doubloons_1200 | $9.99 | +20% bonus |
| doubloons_2500 | $19.99 | +25% bonus |
| doubloons_6500 | $49.99 | +30% bonus |
| starter_pack | $4.99 | One-time |
| battle_pass_s1 | $9.99 | Per season |

### 2.3 License Testing
```
Test Configuration:
├── Add license testers (email addresses)
├── Testers can make purchases without charges
├── Purchases can be refunded/reset
├── Test all purchase flows
└── Enable license testing in Play Console
```

---

## 3. Google Play Billing Library

### 3.1 Dependencies
```gradle
// build.gradle
dependencies {
    implementation "com.android.billingclient:billing-ktx:6.1.0"
}
```

### 3.2 BillingClient Setup
```kotlin
class BillingManager(private val context: Context) {

    private lateinit var billingClient: BillingClient

    fun initialize() {
        billingClient = BillingClient.newBuilder(context)
            .setListener(purchasesUpdatedListener)
            .enablePendingPurchases()
            .build()

        billingClient.startConnection(object : BillingClientStateListener {
            override fun onBillingSetupFinished(result: BillingResult) {
                if (result.responseCode == BillingClient.BillingResponseCode.OK) {
                    // Ready to query products
                    queryProducts()
                }
            }

            override fun onBillingServiceDisconnected() {
                // Retry connection
                reconnect()
            }
        })
    }

    private val purchasesUpdatedListener = PurchasesUpdatedListener { result, purchases ->
        when (result.responseCode) {
            BillingClient.BillingResponseCode.OK -> {
                purchases?.forEach { handlePurchase(it) }
            }
            BillingClient.BillingResponseCode.USER_CANCELED -> {
                // User cancelled
            }
            else -> {
                // Handle error
                logError("Purchase failed: ${result.debugMessage}")
            }
        }
    }
}
```

### 3.3 Query Products
```kotlin
suspend fun queryProducts() {
    val productList = listOf(
        QueryProductDetailsParams.Product.newBuilder()
            .setProductId("doubloons_100")
            .setProductType(BillingClient.ProductType.INAPP)
            .build(),
        // ... other products
    )

    val params = QueryProductDetailsParams.newBuilder()
        .setProductList(productList)
        .build()

    val result = billingClient.queryProductDetails(params)

    if (result.billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
        result.productDetailsList?.forEach { product ->
            productCache[product.productId] = product
        }
    }
}
```

### 3.4 Launch Purchase Flow
```kotlin
fun launchPurchaseFlow(activity: Activity, productId: String) {
    val product = productCache[productId] ?: return

    val productDetailsParams = BillingFlowParams.ProductDetailsParams.newBuilder()
        .setProductDetails(product)
        .build()

    val billingFlowParams = BillingFlowParams.newBuilder()
        .setProductDetailsParamsList(listOf(productDetailsParams))
        .build()

    billingClient.launchBillingFlow(activity, billingFlowParams)
}
```

---

## 4. Purchase Flow

### 4.1 Complete Purchase Flow
```
1. User taps "Buy" in UI
2. Client queries product if not cached
3. Client launches billing flow
4. Google Play shows payment UI
5. User selects payment method
6. User confirms purchase
7. Google processes payment
8. PurchasesUpdatedListener receives result
9. Client sends purchase token to server
10. Server validates with Google Play API
11. Server grants content (Doubloons)
12. Client acknowledges purchase
13. UI updates to show new balance
```

### 4.2 Handle Purchase
```kotlin
private suspend fun handlePurchase(purchase: Purchase) {
    if (purchase.purchaseState == Purchase.PurchaseState.PURCHASED) {
        // Verify on server
        val isValid = validateOnServer(purchase.purchaseToken, purchase.products.first())

        if (isValid) {
            // Consume for consumables
            if (isConsumable(purchase.products.first())) {
                consumePurchase(purchase)
            } else {
                // Acknowledge for non-consumables
                acknowledgePurchase(purchase)
            }
        }
    } else if (purchase.purchaseState == Purchase.PurchaseState.PENDING) {
        // Handle pending purchase
        handlePendingPurchase(purchase)
    }
}

private suspend fun consumePurchase(purchase: Purchase) {
    val params = ConsumeParams.newBuilder()
        .setPurchaseToken(purchase.purchaseToken)
        .build()

    val result = billingClient.consumePurchase(params)

    if (result.billingResult.responseCode != BillingClient.BillingResponseCode.OK) {
        // Handle error, retry
    }
}

private suspend fun acknowledgePurchase(purchase: Purchase) {
    if (!purchase.isAcknowledged) {
        val params = AcknowledgePurchaseParams.newBuilder()
            .setPurchaseToken(purchase.purchaseToken)
            .build()

        billingClient.acknowledgePurchase(params)
    }
}
```

---

## 5. Purchase Token Validation

### 5.1 Server-Side Validation
```
Server Flow:
1. Receive purchase token from client
2. Call Google Play Developer API
3. Verify purchase details
4. Check token hasn't been used
5. Grant content to player
6. Store token as used
7. Return success to client
```

### 5.2 Google Play Developer API
```
Endpoint:
GET https://androidpublisher.googleapis.com/androidpublisher/v3/
    applications/{packageName}/purchases/products/{productId}/tokens/{token}

Authorization: OAuth 2.0 service account

Response:
{
  "purchaseTimeMillis": "1638316800000",
  "purchaseState": 0,
  "consumptionState": 0,
  "developerPayload": "",
  "orderId": "GPA.1234-5678-9012-34567",
  "purchaseType": 0,
  "acknowledgementState": 1,
  "kind": "androidpublisher#productPurchase",
  "regionCode": "US"
}

Purchase States:
├── 0: Purchased
├── 1: Canceled
└── 2: Pending
```

### 5.3 Service Account Setup
```
Setup Steps:
1. Create service account in Google Cloud Console
2. Download JSON credentials
3. Grant API access in Play Console
4. Configure server with credentials
5. Use OAuth 2.0 for API calls
```

---

## 6. Pending Purchases

### 6.1 What Are Pending Purchases
```
Pending purchases occur when:
├── User selects slow payment method (bank transfer)
├── Parental approval required
├── Carrier billing approval pending
└── Other delayed payment scenarios
```

### 6.2 Handling Pending Purchases
```kotlin
private fun handlePendingPurchase(purchase: Purchase) {
    // Store pending purchase for later
    savePendingPurchase(purchase.purchaseToken, purchase.products.first())

    // Show UI indicating pending
    showPendingPurchaseUI()

    // Check status periodically or on app launch
}

// On app launch, check pending purchases
suspend fun checkPendingPurchases() {
    val params = QueryPurchasesParams.newBuilder()
        .setProductType(BillingClient.ProductType.INAPP)
        .build()

    val result = billingClient.queryPurchasesAsync(params)

    result.purchasesList.forEach { purchase ->
        if (purchase.purchaseState == Purchase.PurchaseState.PURCHASED) {
            // Pending purchase now complete
            handlePurchase(purchase)
        }
    }
}
```

---

## 7. Promo Codes

### 7.1 Promo Code Support
```
Google Play supports promo codes for:
├── Free products
├── Discounted subscriptions
├── One-time promotional items

Implementation:
├── Promo codes redeemed in Google Play
├── Appear as normal purchases
├── purchaseType = 1 (promo)
├── No charge to user
└── Grant content normally
```

### 7.2 Handling Promo Purchases
```kotlin
// Check if purchase is promotional
if (purchase.purchaseType == 1) {
    // This is a promo code redemption
    logAnalytics("promo_redemption", purchase.orderId)
}

// Process same as regular purchase
handlePurchase(purchase)
```

---

## 8. Error Handling

### 8.1 Billing Response Codes
| Code | Name | Action |
|------|------|--------|
| 0 | OK | Success |
| 1 | USER_CANCELED | Silent close |
| 2 | SERVICE_UNAVAILABLE | Retry |
| 3 | BILLING_UNAVAILABLE | Check Play Store |
| 4 | ITEM_UNAVAILABLE | Product issue |
| 5 | DEVELOPER_ERROR | Check implementation |
| 6 | ERROR | Generic error |
| 7 | ITEM_ALREADY_OWNED | Non-consumable owned |
| 8 | ITEM_NOT_OWNED | Consume error |

### 8.2 Retry Logic
```kotlin
private suspend fun connectWithRetry(maxAttempts: Int = 5) {
    var attempts = 0
    var delay = 1000L

    while (attempts < maxAttempts) {
        try {
            connect()
            return
        } catch (e: Exception) {
            attempts++
            delay(delay)
            delay *= 2 // Exponential backoff
        }
    }

    throw BillingConnectionException("Failed to connect after $maxAttempts attempts")
}
```

---

## 9. Restore Purchases

### 9.1 Query Existing Purchases
```kotlin
suspend fun restorePurchases() {
    val params = QueryPurchasesParams.newBuilder()
        .setProductType(BillingClient.ProductType.INAPP)
        .build()

    val result = billingClient.queryPurchasesAsync(params)

    result.purchasesList
        .filter { !it.isAcknowledged || isConsumable(it.products.first()) }
        .forEach { purchase ->
            // Validate and grant
            handlePurchase(purchase)
        }
}
```

---

## 10. Testing

### 10.1 Testing Modes
```
License Testing:
├── Add testers in Play Console
├── Real purchase flow, no charges
├── Can test all scenarios
├── Receipts are test receipts

Static Responses (Debug):
├── Use reserved product IDs
├── android.test.purchased (success)
├── android.test.canceled (cancel)
├── android.test.item_unavailable (error)
└── Quick testing without setup
```

### 10.2 Test Scenarios
```
Test Cases:
├── Successful purchase (all products)
├── User cancellation
├── Network interruption
├── Pending purchase flow
├── Promo code redemption
├── Restore purchases
├── App kill during purchase
├── Multiple devices
└── Subscription lifecycle (if applicable)
```

---

## 11. Integration Points

### 11.1 System Dependencies
| System | Integration |
|--------|-------------|
| Currency System | Grant Doubloons |
| Inventory System | Grant items |
| Server | Token validation |
| Analytics | Purchase events |
| Account | Link to player |

---

## 12. Default Configuration

```json
{
  "billing": {
    "productIds": {
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
    "validation": {
      "endpoint": "/api/v1/iap/google/validate",
      "retryAttempts": 3,
      "retryDelayMs": 2000
    },
    "reconnection": {
      "maxAttempts": 5,
      "baseDelayMs": 1000,
      "maxDelayMs": 30000
    }
  }
}
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
