# MON-001: Dual Currency System Specification

## Overview

This document specifies the dual currency economy system for Plunderstorm Mobile, implementing both soft currency (Gold Coins) earned through gameplay and hard currency (Doubloons) purchased with real money.

## Design Philosophy

The dual currency system creates a balanced economy where free players can earn most items through dedicated play while premium currency provides convenience and access to exclusive cosmetics. The system must prevent inflation while maintaining player engagement.

---

## 1. Currency Types

### 1.1 Gold Coins (Soft Currency)
```
Name: Gold Coins (or "Gold")
Symbol: 🪙 or coin icon
Color: Gold (#FFD700)

Purpose:
├── Primary earnable currency
├── Rewards gameplay effort
├── Purchases common items
├── Maintains engagement loop
└── Cannot be directly purchased
```

### 1.2 Doubloons (Hard Currency)
```
Name: Doubloons
Symbol: 💎 or gem icon
Color: Blue/Purple (#8A2BE2)

Purpose:
├── Premium currency
├── Purchased with real money
├── Buys exclusive items
├── Provides convenience (skips)
├── Small amount earnable (Battle Pass)
└── Primary revenue source
```

### 1.3 Currency Comparison
| Aspect | Gold Coins | Doubloons |
|--------|------------|-----------|
| Acquisition | Gameplay | IAP (primarily) |
| Earn Rate | Moderate | Very limited |
| Spending | Common items | Premium items |
| Cap | No hard cap | No cap |
| Conversion | Cannot buy Doubloons | Cannot buy Gold |
| Value Perception | Time investment | Money investment |

---

## 2. Gold Coins Economy

### 2.1 Earning Sources
| Source | Amount | Frequency |
|--------|--------|-----------|
| Match Completion | 50-200 | Per match |
| Victory (1st place) | 200 + bonus | Per win |
| Top 10 Finish | 100-150 | Per match |
| Kill/Elimination | 10 | Per kill |
| Daily Quest | 100-300 | 3x daily |
| Weekly Quest | 500-1000 | 3x weekly |
| Level Up | 100 | Per level |
| First Win of Day | 200 bonus | 1x daily |
| Achievement | 50-500 | One-time |

### 2.2 Match Rewards Breakdown
```
Placement Rewards (Solo):
├── 1st: 200 Gold
├── 2nd-5th: 150 Gold
├── 6th-10th: 100 Gold
├── 11th-20th: 75 Gold
├── 21st-30th: 50 Gold
└── 31st+: 50 Gold (minimum)

Bonuses:
├── Per kill: +10 Gold
├── First win of day: +200 Gold
├── Win streak (3+): +50 Gold
└── Full match (no disconnect): +25 Gold
```

### 2.3 Gold Spending Destinations
| Item Type | Price Range | Notes |
|-----------|-------------|-------|
| Common Outfit | 500-1,500 | Always available |
| Uncommon Outfit | 2,000-3,500 | Limited selection |
| Common Emote | 500-1,000 | Always available |
| Uncommon Emote | 1,500-2,500 | Limited selection |
| Profile Icon | 200-500 | Various designs |
| Profile Banner | 500-1,000 | Various designs |
| Name Change | 500 | Per change |
| Spray/Tag | 300-800 | Various designs |

### 2.4 Anti-Inflation Mechanics
```
Prevent Gold Inflation:
├── Soft daily cap: 2,000 Gold from matches
├── Quest limits: Fixed daily/weekly count
├── No Gold trading between players
├── High-value Gold sinks (expensive items)
├── Server-authoritative earn validation
└── Anomaly detection for excessive earning
```

---

## 3. Doubloons Economy

### 3.1 Acquisition Methods
```
Primary:
├── In-App Purchases (IAP)
│   ├── Small Pack: 100 Doubloons - $0.99
│   ├── Medium Pack: 550 Doubloons - $4.99
│   ├── Large Pack: 1,200 Doubloons - $9.99
│   ├── Mega Pack: 2,500 Doubloons - $19.99
│   └── Ultra Pack: 6,500 Doubloons - $49.99

Secondary (Earnable):
├── Battle Pass Free Track: ~100-200 total per season
├── Battle Pass Premium Track: ~950+ total (returns cost)
├── Special Events: Limited amounts
├── Achievement Milestones: Rare, one-time
└── New Player Welcome Gift: 50 Doubloons
```

### 3.2 Doubloons Spending Destinations
| Item Type | Price Range | Notes |
|-----------|-------------|-------|
| Rare Outfit | 500-1,000 | Premium quality |
| Epic Outfit | 1,200-1,800 | High quality |
| Legendary Outfit | 2,000-2,500 | Highest quality |
| Premium Emote | 400-800 | Exclusive |
| Battle Pass | 950 | Per season |
| Battle Pass + Bundle | 1,400 | Pass + 10 tiers |
| Tier Skip | 150 | Per tier |
| Exclusive Bundle | 1,500-3,000 | Limited time |
| Starter Pack | 500 (in $4.99 pack) | One-time |

### 3.3 Doubloons Value Perception
```
Base Conversion: ~$0.01 per Doubloon
Bulk Bonus: Up to 30% more at higher tiers

Price Anchoring:
├── Battle Pass = 950 (~$10 value)
├── Legendary Outfit = 2,000 (~$20 value)
├── This makes packs feel like good value
└── Encourages larger pack purchases
```

---

## 4. Data Model

### 4.1 Player Currency Record
```json
{
  "playerId": "uuid",
  "currencies": {
    "gold": {
      "balance": 15230,
      "lifetimeEarned": 125000,
      "lifetimeSpent": 109770,
      "lastUpdated": "2025-12-02T14:30:00Z"
    },
    "doubloons": {
      "balance": 450,
      "lifetimePurchased": 2000,
      "lifetimeEarned": 350,
      "lifetimeSpent": 1900,
      "lastUpdated": "2025-12-02T14:25:00Z"
    }
  }
}
```

### 4.2 Transaction Record
```json
{
  "transactionId": "uuid",
  "playerId": "uuid",
  "timestamp": "2025-12-02T14:30:00Z",
  "currency": "gold",
  "amount": 150,
  "type": "earn",
  "source": "match_reward",
  "sourceId": "match-uuid",
  "balanceBefore": 15080,
  "balanceAfter": 15230,
  "metadata": {
    "placement": 3,
    "kills": 2,
    "bonuses": ["first_win_of_day"]
  }
}
```

### 4.3 Database Schema
```sql
-- Player currency balances
CREATE TABLE player_currencies (
  player_id UUID PRIMARY KEY,
  gold_balance BIGINT DEFAULT 0,
  gold_lifetime_earned BIGINT DEFAULT 0,
  gold_lifetime_spent BIGINT DEFAULT 0,
  doubloons_balance BIGINT DEFAULT 0,
  doubloons_lifetime_purchased BIGINT DEFAULT 0,
  doubloons_lifetime_earned BIGINT DEFAULT 0,
  doubloons_lifetime_spent BIGINT DEFAULT 0,
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Transaction log
CREATE TABLE currency_transactions (
  transaction_id UUID PRIMARY KEY,
  player_id UUID REFERENCES player_currencies(player_id),
  currency VARCHAR(20) NOT NULL,
  amount BIGINT NOT NULL,
  transaction_type VARCHAR(20) NOT NULL,
  source VARCHAR(50) NOT NULL,
  source_id VARCHAR(100),
  balance_before BIGINT NOT NULL,
  balance_after BIGINT NOT NULL,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_transactions_player ON currency_transactions(player_id);
CREATE INDEX idx_transactions_time ON currency_transactions(created_at);
CREATE INDEX idx_transactions_source ON currency_transactions(source);
```

---

## 5. Currency Operations

### 5.1 Earn Currency
```
Earn Flow:
1. Action triggers earn (match complete, quest done)
2. Server calculates amount
3. Server validates earn is legitimate
4. Begin transaction
5. Read current balance
6. Add amount to balance
7. Create transaction record
8. Commit transaction
9. Send balance update to client
10. Client displays earn notification

Validation Checks:
├── Source is valid (match exists, quest exists)
├── Player eligible (participated in match)
├── Amount within expected range
├── Not duplicate (idempotency check)
├── Daily caps not exceeded
└── No suspicious patterns
```

### 5.2 Spend Currency
```
Spend Flow:
1. Client requests purchase
2. Server receives request with item ID
3. Server looks up item price
4. Server checks player balance
5. If balance >= price:
   a. Begin transaction
   b. Deduct currency
   c. Grant item/reward
   d. Create transaction record
   e. Commit transaction
   f. Send success to client
6. If balance < price:
   a. Send insufficient funds error
   b. Client shows "need more" prompt
```

### 5.3 IAP Currency Grant
```
IAP Flow:
1. Player purchases via store
2. Store sends receipt/token
3. Server validates receipt (see MON-015)
4. If valid:
   a. Look up product → Doubloon amount
   b. Grant Doubloons (transactional)
   c. Mark receipt as used
   d. Send confirmation
5. If invalid:
   a. Log attempt
   b. Return error
   c. Alert if repeated
```

---

## 6. UI/UX Design

### 6.1 Currency Display
```
Header Bar (Always Visible):
┌────────────────────────────────────────────────┐
│  🪙 15,230    💎 450         [+] [Shop]       │
└────────────────────────────────────────────────┘

Format:
├── Icon + Amount
├── Abbreviated large numbers (15.2K)
├── [+] button opens currency purchase
├── Tap currency opens wallet/history
└── Animate on change (count up/down)
```

### 6.2 Earn Notification
```
On Currency Earned:
┌──────────────────────────────────────┐
│        🪙 +150 Gold                  │
│     Match Completion Reward          │
└──────────────────────────────────────┘

Animation:
├── Coins fly to header bar
├── Count increases
├── Brief glow effect
└── Sound effect (optional)
```

### 6.3 Spend Confirmation
```
Purchase Confirmation:
┌──────────────────────────────────────────────────┐
│              Purchase Item?                      │
│                                                  │
│     [Item Preview Image]                         │
│     "Pirate Captain Outfit"                      │
│                                                  │
│     Price: 💎 1,200 Doubloons                    │
│     Your Balance: 💎 450                         │
│                                                  │
│  [Cancel]                [Get More Doubloons]    │
└──────────────────────────────────────────────────┘

Or if sufficient:
│  [Cancel]                      [Purchase]        │
```

### 6.4 Insufficient Funds Flow
```
When Balance < Price:
1. Show price and current balance
2. Calculate shortfall
3. Offer "Get More" option
4. Link to currency shop
5. Pre-select appropriate pack

Example:
"You need 750 more Doubloons"
[Get 1,200 for $9.99] ← Suggested pack
[View All Packs]
```

---

## 7. Icon and Visual Design

### 7.1 Gold Coins
```
Icon Design:
├── Circular gold coin
├── Skull or ship emblem in center
├── Shiny metallic effect
├── Stack of coins variant
└── Color: #FFD700 (gold)

Animation:
├── Subtle shine/gleam
├── Coin flip on earn
├── Pile grows animation
└── Scatter on spend
```

### 7.2 Doubloons
```
Icon Design:
├── Gemstone or crystalline shape
├── Blue/purple coloring
├── Faceted, premium look
├── Slight glow effect
└── Color: #8A2BE2 (blue-violet)

Animation:
├── Sparkle effect
├── Rotate/shine on earn
├── Premium feel
└── Magical particles on spend
```

---

## 8. Security and Validation

### 8.1 Server-Authoritative
```
All Operations Server-Side:
├── Client cannot modify balances
├── Client sends requests only
├── Server validates all actions
├── Server calculates all amounts
├── Client displays server state
└── Discrepancies flagged
```

### 8.2 Fraud Prevention
```
Checks:
├── Rate limiting on earn actions
├── Match completion verified
├── Quest completion verified
├── IAP receipts validated
├── No negative balances
├── Transaction logs immutable
├── Anomaly detection alerts
└── Daily earn caps enforced
```

### 8.3 Audit Trail
```
Log All:
├── Every balance change
├── Transaction type and source
├── Before/after balances
├── Timestamp
├── Related IDs (match, quest, IAP)
├── IP address (for fraud)
└── Device ID (for fraud)

Retention: 1+ year for financial
```

---

## 9. Integration Points

### 9.1 System Dependencies
| System | Integration |
|--------|-------------|
| Match System | Rewards on completion |
| Quest System | Rewards on completion |
| Shop System | Spend for items |
| Battle Pass | Rewards and purchase |
| IAP System | Doubloon purchases |
| Analytics | Economy metrics |
| Account System | Balance persistence |

### 9.2 API Endpoints
```
GET /api/v1/player/{id}/currencies
├── Returns current balances

POST /api/v1/player/{id}/currencies/earn
├── Body: { source, sourceId, amount, currency }
├── Server validates and grants

POST /api/v1/player/{id}/currencies/spend
├── Body: { itemId, currency, amount }
├── Server validates, deducts, grants item

GET /api/v1/player/{id}/currencies/transactions
├── Query: { currency, limit, offset, from, to }
├── Returns transaction history
```

---

## 10. Testing Requirements

### 10.1 Unit Tests
```
Test Cases:
├── Earn increases balance
├── Spend decreases balance
├── Cannot spend more than balance
├── Cannot have negative balance
├── Transaction records created
├── Concurrent operations safe
├── Idempotency works
└── Validation rejects invalid
```

### 10.2 Integration Tests
```
Test Scenarios:
├── Match completion grants gold
├── Quest completion grants gold
├── IAP grants doubloons
├── Shop purchase spends correctly
├── Insufficient funds handled
├── Daily cap enforcement
└── Multi-device sync
```

### 10.3 Load Tests
```
Stress Test:
├── 1000 concurrent transactions
├── No balance corruption
├── Acceptable latency (< 500ms)
├── Database handles load
└── No deadlocks
```

---

## 11. Default Configuration

```json
{
  "currencies": {
    "gold": {
      "name": "Gold Coins",
      "icon": "gold_coin",
      "color": "#FFD700",
      "startingBalance": 500,
      "dailyEarnCap": 2000,
      "abbreviateThreshold": 10000
    },
    "doubloons": {
      "name": "Doubloons",
      "icon": "doubloon",
      "color": "#8A2BE2",
      "startingBalance": 0,
      "welcomeGift": 50,
      "abbreviateThreshold": 10000
    }
  },
  "validation": {
    "maxTransactionAmount": 100000,
    "rateLimit": {
      "earnPerMinute": 10,
      "spendPerMinute": 20
    }
  },
  "display": {
    "animateChanges": true,
    "showEarnNotification": true,
    "soundOnEarn": true,
    "soundOnSpend": true
  }
}
```

---

## Appendix A: Visual Reference

```
Currency Display States:

Normal:
┌────────────────────────────────────┐
│  🪙 15,230         💎 450          │
└────────────────────────────────────┘

Earning (Animation):
┌────────────────────────────────────┐
│  🪙 15,230 → 15,380  💎 450        │
│     ↑ +150 Gold                    │
└────────────────────────────────────┘

Insufficient Funds:
┌────────────────────────────────────┐
│  🪙 15,230         💎 450 ⚠️       │
│                    Need: 1,200     │
└────────────────────────────────────┘

Large Numbers:
┌────────────────────────────────────┐
│  🪙 1.25M           💎 12.5K       │
└────────────────────────────────────┘
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
