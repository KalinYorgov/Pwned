# MON-014: IAP Product Catalog Specification

## Overview

This document specifies all in-app purchase products for Plunderstorm Mobile across both iOS App Store and Google Play Store, including pricing, content, and configuration details.

---

## 1. Currency Packs

### 1.1 Doubloon Packs
| Pack ID | Doubloons | Bonus | USD | EUR | GBP | Notes |
|---------|-----------|-------|-----|-----|-----|-------|
| doubloons_100 | 100 | 0% | $0.99 | €0.99 | £0.79 | Entry tier |
| doubloons_550 | 550 | 10% | $4.99 | €4.99 | £4.49 | Best for casual |
| doubloons_1200 | 1,200 | 20% | $9.99 | €9.99 | £8.99 | Most popular |
| doubloons_2500 | 2,500 | 25% | $19.99 | €19.99 | £17.99 | Value pack |
| doubloons_6500 | 6,500 | 30% | $49.99 | €49.99 | £44.99 | Best value |

### 1.2 Pack Details
```
doubloons_100:
├── Product ID (iOS): com.company.plunderstorm.doubloons_100
├── Product ID (Android): doubloons_100
├── Type: Consumable
├── Display Name: "100 Doubloons"
├── Description: "A small pouch of premium Doubloons"
├── Icon: Small gem pouch

doubloons_550:
├── Product ID (iOS): com.company.plunderstorm.doubloons_550
├── Product ID (Android): doubloons_550
├── Type: Consumable
├── Display Name: "550 Doubloons"
├── Description: "A chest of Doubloons with 10% bonus!"
├── Icon: Small treasure chest
├── Badge: "+10% BONUS"

doubloons_1200:
├── Product ID (iOS): com.company.plunderstorm.doubloons_1200
├── Product ID (Android): doubloons_1200
├── Type: Consumable
├── Display Name: "1,200 Doubloons"
├── Description: "A large chest of Doubloons with 20% bonus!"
├── Icon: Medium treasure chest
├── Badge: "MOST POPULAR" + "+20% BONUS"

doubloons_2500:
├── Product ID (iOS): com.company.plunderstorm.doubloons_2500
├── Product ID (Android): doubloons_2500
├── Type: Consumable
├── Display Name: "2,500 Doubloons"
├── Description: "A captain's treasure with 25% bonus!"
├── Icon: Large treasure chest
├── Badge: "+25% BONUS"

doubloons_6500:
├── Product ID (iOS): com.company.plunderstorm.doubloons_6500
├── Product ID (Android): doubloons_6500
├── Type: Consumable
├── Display Name: "6,500 Doubloons"
├── Description: "A legendary hoard with 30% bonus - best value!"
├── Icon: Overflowing treasure chest
├── Badge: "BEST VALUE" + "+30% BONUS"
```

---

## 2. Special Packs

### 2.1 Starter Pack
```
Product Details:
├── Product ID (iOS): com.company.plunderstorm.starter_pack
├── Product ID (Android): starter_pack
├── Type: Non-Consumable (one-time)
├── Price: $4.99 / €4.99 / £4.49
├── Display Name: "Starter Pack"
├── Description: "The perfect way to begin your voyage!"

Contents:
├── 500 Doubloons
├── "First Mate" Outfit (Rare)
├── "Ahoy!" Emote
├── "Skull & Crossbones" Spray
└── 3-Day XP Boost

Value: ~$12 worth for $4.99 (60% off)

Restrictions:
├── One purchase per account
├── Cannot be gifted
├── Shows "OWNED" if purchased
└── Removed from store after purchase
```

### 2.2 Battle Pass
```
Product Details:
├── Product ID (iOS): com.company.plunderstorm.battle_pass_s1
├── Product ID (Android): battle_pass_s1
├── Type: Non-Consumable (per season)
├── Price: $9.99 / €9.99 / £8.99
├── Display Name: "Battle Pass - Season 1"
├── Description: "Unlock the premium reward track with exclusive cosmetics!"

Contents:
├── Premium Battle Pass Track
├── Instant unlock: "Pirate King" Title
├── 60+ Premium Rewards
├── Exclusive Legendary Outfit at Tier 60
└── ~950 Doubloons throughout pass

Note: New product ID each season
```

### 2.3 Battle Pass Bundle
```
Product Details:
├── Product ID (iOS): com.company.plunderstorm.battle_pass_bundle_s1
├── Product ID (Android): battle_pass_bundle_s1
├── Type: Non-Consumable (per season)
├── Price: $14.99 / €14.99 / £13.99
├── Display Name: "Battle Pass + 10 Tiers"
├── Description: "Get the Battle Pass plus skip 10 tiers instantly!"

Contents:
├── Premium Battle Pass
├── 10 Tier Skips (1,500 Doubloons value)
├── "Early Bird" Badge
└── All Battle Pass benefits

Value: $9.99 (Pass) + $15 (Tier skips) = ~$25 for $14.99
```

---

## 3. Regional Pricing

### 3.1 Tier Mapping
| Product | Apple Tier | Google Equivalent |
|---------|------------|-------------------|
| doubloons_100 | Tier 1 | $0.99 base |
| doubloons_550 | Tier 5 | $4.99 base |
| doubloons_1200 | Tier 10 | $9.99 base |
| doubloons_2500 | Tier 20 | $19.99 base |
| doubloons_6500 | Tier 50 | $49.99 base |

### 3.2 Regional Adjustments
| Region | Adjustment | Reason |
|--------|------------|--------|
| US, EU, UK | Base price | Reference markets |
| Brazil | -20% | Lower purchasing power |
| India | -30% | Market penetration |
| Russia | -25% | Economic factors |
| Japan | +5% | Market standard |
| Australia | +10% | Tax/forex |

---

## 4. Product Configuration

### 4.1 Server Configuration
```json
{
  "products": {
    "doubloons_100": {
      "type": "consumable",
      "grant": { "doubloons": 100 },
      "platforms": ["ios", "android"]
    },
    "doubloons_550": {
      "type": "consumable",
      "grant": { "doubloons": 550 },
      "platforms": ["ios", "android"]
    },
    "doubloons_1200": {
      "type": "consumable",
      "grant": { "doubloons": 1200 },
      "platforms": ["ios", "android"]
    },
    "doubloons_2500": {
      "type": "consumable",
      "grant": { "doubloons": 2500 },
      "platforms": ["ios", "android"]
    },
    "doubloons_6500": {
      "type": "consumable",
      "grant": { "doubloons": 6500 },
      "platforms": ["ios", "android"]
    },
    "starter_pack": {
      "type": "non_consumable",
      "oneTime": true,
      "grant": {
        "doubloons": 500,
        "items": ["outfit_first_mate", "emote_ahoy", "spray_skull"],
        "boosts": [{ "type": "xp", "duration": 259200 }]
      },
      "platforms": ["ios", "android"]
    },
    "battle_pass_s1": {
      "type": "non_consumable",
      "season": 1,
      "grant": { "battlePass": true, "tierSkips": 0 },
      "platforms": ["ios", "android"]
    },
    "battle_pass_bundle_s1": {
      "type": "non_consumable",
      "season": 1,
      "grant": { "battlePass": true, "tierSkips": 10 },
      "platforms": ["ios", "android"]
    }
  }
}
```

---

## 5. Store Display

### 5.1 Currency Shop Layout
```
┌──────────────────────────────────────────────────────────────┐
│                     💎 GET DOUBLOONS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │   100    │  │   550    │  │  1,200   │                   │
│  │    💎    │  │    💎    │  │    💎    │                   │
│  │  $0.99   │  │  $4.99   │  │  $9.99   │                   │
│  │          │  │  +10%    │  │  +20%    │                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
│                             [MOST POPULAR]                   │
│                                                              │
│  ┌──────────┐  ┌──────────┐                                 │
│  │  2,500   │  │  6,500   │                                 │
│  │    💎    │  │    💎    │                                 │
│  │ $19.99   │  │ $49.99   │                                 │
│  │  +25%    │  │  +30%    │                                 │
│  └──────────┘  └──────────┘                                 │
│               [BEST VALUE]                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 Special Offers Section
```
┌──────────────────────────────────────────────────────────────┐
│                    ⭐ SPECIAL OFFERS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │  🎁 STARTER PACK          LIMITED - ONE TIME ONLY  │     │
│  │                                                    │     │
│  │  • 500 Doubloons                                   │     │
│  │  • "First Mate" Outfit                             │     │
│  │  • "Ahoy!" Emote                                   │     │
│  │  • 3-Day XP Boost                                  │     │
│  │                                                    │     │
│  │  ~~$12.00~~  $4.99     [BUY NOW]                   │     │
│  │              60% OFF                               │     │
│  └────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. Compliance

### 6.1 Store Requirements
```
iOS App Store:
├── All prices from App Store Connect
├── No external payment links
├── Clear product descriptions
├── Restore purchases option
└── Accurate screenshots

Google Play:
├── Prices via Play Billing only
├── No alternative payment
├── Clear disclosure of IAP
├── Comply with Families policy
└── Regional compliance
```

### 6.2 Legal Requirements
```
Disclosure:
├── Loot boxes: Disclose odds (if applicable)
├── Consumables: Clear that content is digital
├── Subscriptions: Clear renewal terms
├── Refunds: Platform handles refunds
└── Age restrictions: Platform parental controls
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
