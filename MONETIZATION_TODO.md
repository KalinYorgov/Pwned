# Monetization TODO - In-App Purchases & Economy

This document covers all monetization systems including currencies, store, Battle Pass, IAP integration, and economic balancing.

---

## Table of Contents
1. [Currency System](#1-currency-system)
2. [In-Game Store](#2-in-game-store)
3. [Battle Pass](#3-battle-pass)
4. [IAP Integration](#4-iap-integration)
5. [Advertising](#5-advertising)
6. [Economy Balancing](#6-economy-balancing)

---

## 1. Currency System

### MON-001: Dual Currency System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement dual currency economy with soft currency (earnable) and hard currency (premium, purchasable).

**Currencies:**
- **Gold Coins (Soft):** Earned through gameplay, used for common purchases
- **Doubloons (Hard):** Purchased with real money, used for premium items

**Acceptance Criteria:**
- [x] Both currencies tracked per player account
- [x] Currencies display in HUD/menu header
- [x] Gold earned from: match completion, quests, achievements
- [x] Doubloons purchased via IAP
- [x] Small amount of Doubloons earnable (Battle Pass free tier)
- [x] Currency transactions logged for debugging
- [x] No negative balances possible
- [x] Currency icons distinct and themed

**Definition of Done:**
- Currency system functional end-to-end
- All earn/spend flows working
- Balances persist across sessions
- Economy spreadsheet initialized

**Deliverable:** [Dual Currency System Specification](docs/DUAL_CURRENCY_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-011

---

### MON-002: Currency Earning Rates
**Priority:** P1 | **Complexity:** S

**Description:**
Define and implement earning rates for soft currency.

**Gold Earning Sources:**
- Match completion: 50-200 based on placement
- Daily quest: 100-300 per quest
- Weekly quest: 500-1000 per quest
- Achievement unlock: Variable
- Level up: 100 per level
- First win of day: 200 bonus

**Acceptance Criteria:**
- [x] All earning sources implemented
- [x] Rates balanced against shop prices
- [x] Earning rates documented
- [x] Anti-farming limits (max daily earn cap optional)
- [x] Earning visible in end-of-match summary
- [x] Quests show currency reward

**Definition of Done:**
- Players earn gold at intended rate
- Economy healthy (not too fast or slow)
- Adjusted based on soft launch data

**Deliverable:** [Currency Earning Rates Specification](docs/CURRENCY_EARNING_RATES_SPEC.md)

**Status:** Completed

**Dependencies:** MON-001, LIVE-001

---

### MON-003: Currency Spending Sinks
**Priority:** P1 | **Complexity:** S

**Description:**
Define spending destinations for currencies to maintain healthy economy.

**Gold Sinks:**
- Common cosmetics (500-2000 Gold)
- Emotes (1000-3000 Gold)
- Name change (500 Gold)
- Profile customization (200-1000 Gold)

**Doubloon Sinks:**
- Premium cosmetics (500-2000 Doubloons)
- Battle Pass purchase (950 Doubloons)
- Battle Pass tier skip (150 Doubloons per tier)
- Exclusive bundles (varies)

**Acceptance Criteria:**
- [x] Prices set for all purchasable items
- [x] Gold items accessible through reasonable play
- [x] Premium items priced for value perception
- [x] Price tiers documented
- [x] Conversion rate (implicit $/Doubloon) consistent

**Definition of Done:**
- All items priced
- Economy spreadsheet complete
- Validated through playtesting

**Deliverable:** [Currency Spending Sinks Specification](docs/CURRENCY_SPENDING_SINKS_SPEC.md)

**Status:** Completed

**Dependencies:** MON-001, MON-002

---

## 2. In-Game Store

### MON-004: Store UI Framework
**Priority:** P1 | **Complexity:** M

**Description:**
Create the in-game store interface for browsing and purchasing items.

**Store Sections:**
- Featured (rotating daily/weekly highlights)
- Outfits
- Emotes
- Bundles
- Battle Pass (link)
- Currency purchase

**Acceptance Criteria:**
- [x] Store accessible from main menu
- [x] Categories clearly organized
- [x] Item preview (3D model viewer for outfits)
- [x] Price clearly displayed
- [x] Purchase confirmation dialog
- [x] Purchase success/failure feedback
- [x] Owned items marked
- [x] Filter by: price, type, rarity
- [x] Sort by: newest, price, popularity

**Definition of Done:**
- Store fully functional
- All items purchasable
- UX smooth and clear
- No exploit paths (duplicate purchase, etc.)

**Deliverable:** [Store UI Framework Specification](docs/STORE_UI_FRAMEWORK_SPEC.md)

**Status:** Completed

**Dependencies:** MON-001, UX-014

---

### MON-005: Featured/Daily Shop Rotation
**Priority:** P1 | **Complexity:** M

**Description:**
Implement rotating featured items to create urgency and variety.

**Rotation Schedule:**
- Daily rotation: 4-6 items, refresh every 24 hours
- Weekly rotation: Premium items, refresh weekly
- Event rotation: Special items during events

**Acceptance Criteria:**
- [x] Featured section shows rotation items
- [x] Countdown timer to next rotation
- [x] Items selected from pool (can be configured)
- [x] Same item not in back-to-back rotations (cooldown)
- [x] Admin tool to configure rotation
- [x] Special forced items for events
- [x] Notification when new rotation arrives

**Definition of Done:**
- Rotation system live
- Players engage with FOMO mechanic
- Admin can manage rotations

**Deliverable:** [Featured/Daily Shop Rotation Specification](docs/FEATURED_DAILY_SHOP_ROTATION_SPEC.md)

**Status:** Completed

**Dependencies:** MON-004

---

### MON-006: Bundle System
**Priority:** P1 | **Complexity:** S

**Description:**
Create bundles that offer multiple items at a discount.

**Bundle Types:**
- Starter Pack: Special one-time purchase for new players
- Themed Bundle: Set of matching cosmetics
- Event Bundle: Limited-time event items
- Value Bundle: Currency + bonus items

**Acceptance Criteria:**
- [x] Bundles show total value and discount %
- [x] Items purchasable individually or in bundle
- [x] Bundle price adjusts if player owns some items
- [x] Starter pack limited to one purchase per account
- [x] Bundle preview shows all contents
- [x] Time-limited bundles have countdown

**Definition of Done:**
- Bundle system functional
- Starter pack converts new users
- Bundles provide value perception

**Deliverable:** [Bundle System Specification](docs/BUNDLE_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** MON-004

---

### MON-007: Cosmetic Inventory System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement player inventory for owned cosmetics with equip functionality.

**Inventory Features:**
- View all owned items by category
- Equip/unequip items
- Preview items on character
- Favorite items
- Filter and sort

**Acceptance Criteria:**
- [x] All owned items displayed
- [x] Current equipped items highlighted
- [x] Tap to equip, tap again to unequip
- [x] Preview shows item on character model
- [x] Supports all cosmetic types (outfits, emotes, etc.)
- [x] Search/filter functionality
- [x] Persistence across sessions

**Definition of Done:**
- Players can manage cosmetics easily
- Equip flow smooth
- No inventory bugs

**Deliverable:** [Cosmetic Inventory System Specification](docs/COSMETIC_INVENTORY_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-011

---

## 3. Battle Pass

### MON-008: Battle Pass Framework
**Priority:** P1 | **Complexity:** L

**Description:**
Implement seasonal Battle Pass system with free and premium tiers.

**Battle Pass Structure:**
- 60-80 tiers per season
- Free track: Accessible to all, fewer rewards
- Premium track: Requires purchase, exclusive rewards
- Season duration: 6-8 weeks
- XP earned through play and quests

**Acceptance Criteria:**
- [x] Battle Pass UI shows dual track (free/premium)
- [x] Current tier and XP progress visible
- [x] Preview of all rewards up to next 5-10 tiers
- [x] Full pass preview available
- [x] Purchase premium pass from within UI
- [x] Claim rewards (manual or auto-claim option)
- [x] Season countdown timer
- [x] Season end handling (unclaimed rewards)

**Definition of Done:**
- Battle Pass functional end-to-end
- Players understand and engage with system
- Revenue generated from pass purchases

**Deliverable:** [Battle Pass Framework Specification](docs/BATTLE_PASS_FRAMEWORK_SPEC.md)

**Status:** Completed

**Dependencies:** MON-001, UX-014

---

### MON-009: Battle Pass Rewards Design
**Priority:** P1 | **Complexity:** M

**Description:**
Design reward distribution for Battle Pass tiers.

**Reward Distribution (Example 60-tier pass):**
- **Free Track:** Gold, XP boosts, 1-2 basic cosmetics, small Doubloon drops
- **Premium Track:** Premium cosmetics, exclusive outfits, emotes, Doubloons (enough to buy next pass if completed)

**Key Reward Placements:**
- Tier 1: Immediate reward upon purchase (premium outfit piece)
- Tier 30: Mid-season milestone (premium item)
- Tier 60: Finale reward (legendary outfit or bundle)

**Acceptance Criteria:**
- [x] Rewards spread to maintain engagement
- [x] Premium pass returns ~75-100% of cost in Doubloons
- [x] Exclusive items not available elsewhere
- [x] Variety of reward types (not all cosmetics)
- [x] Rewards documented in design spreadsheet

**Definition of Done:**
- Full season rewards designed
- Pass feels valuable
- Completion rate targets set

**Deliverable:** [Battle Pass Rewards Design Specification](docs/BATTLE_PASS_REWARDS_DESIGN_SPEC.md)

**Status:** Completed

**Dependencies:** MON-008

---

### MON-010: Battle Pass XP System
**Priority:** P1 | **Complexity:** M

**Description:**
Implement XP earning for Battle Pass progression.

**XP Sources:**
- Match completion: 50-200 XP based on placement
- Daily quest completion: 500 XP each
- Weekly quest completion: 2000 XP each
- Bonus XP events (double XP weekends)

**Tier Requirements:**
- Each tier requires ~10,000 XP
- Total pass: ~600,000 XP
- Should be completable in 6-8 weeks with daily play

**Acceptance Criteria:**
- [x] XP tracked separately from player level XP
- [x] XP earned displays in match summary
- [x] Quest XP rewards shown clearly
- [x] Progress bar shows XP to next tier
- [x] XP boost items functional (if included)
- [x] Tier skip option (Doubloon purchase)

**Definition of Done:**
- XP system balanced for season length
- Average player completes 75%+ of pass
- Tier skip provides whale option

**Deliverable:** [Battle Pass XP System Specification](docs/BATTLE_PASS_XP_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** MON-008, LIVE-001

---

### MON-011: Season Management
**Priority:** P1 | **Complexity:** M

**Description:**
Build backend tools for managing Battle Pass seasons.

**Management Features:**
- Create new season with rewards
- Set season start/end dates
- Monitor completion rates
- Handle season transition
- Unclaimed rewards policy

**Acceptance Criteria:**
- [x] Admin can create new season in advance
- [x] Automatic transition at season end
- [x] Player progress resets for new season
- [x] Unclaimed rewards delivered to inbox
- [x] Previous season pass becomes inactive
- [x] Data export for analysis

**Definition of Done:**
- Season rollover tested
- Admin tools functional
- Process documented for LiveOps

**Deliverable:** [Season Management Specification](docs/SEASON_MANAGEMENT_SPEC.md)

**Status:** Completed

**Dependencies:** MON-008, BACK-022

---

## 4. IAP Integration

### MON-012: App Store IAP Integration (iOS)
**Priority:** P0 | **Complexity:** M

**Description:**
Integrate Apple App Store in-app purchases.

**Product Types:**
- Consumable: Currency packs
- Non-consumable: One-time purchases (starter pack)
- Subscription: (Future - VIP pass if implemented)

**Acceptance Criteria:**
- [x] Products created in App Store Connect
- [x] StoreKit integration in game
- [x] Purchase flow (initiate, confirm, deliver)
- [x] Receipt validation (server-side)
- [x] Restore purchases functionality
- [x] Handle purchase interruptions (retry)
- [x] Sandbox testing verified
- [x] Refund handling
- [x] Price localization

**Definition of Done:**
- All IAP products purchasable on iOS
- Receipts validated securely
- No duplicate delivery
- Tested on real devices

**Deliverable:** [App Store IAP Integration Specification](docs/APP_STORE_IAP_INTEGRATION_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-011

---

### MON-013: Google Play IAP Integration (Android)
**Priority:** P0 | **Complexity:** M

**Description:**
Integrate Google Play Billing for in-app purchases.

**Acceptance Criteria:**
- [x] Products created in Play Console
- [x] Google Play Billing Library integration
- [x] Purchase flow (initiate, confirm, deliver)
- [x] Purchase token validation (server-side)
- [x] Restore purchases functionality
- [x] Handle pending purchases
- [x] Test with license testers
- [x] Promo codes support
- [x] Price localization

**Definition of Done:**
- All IAP products purchasable on Android
- Tokens validated securely
- No duplicate delivery
- Tested on real devices

**Deliverable:** [Google Play IAP Integration Specification](docs/GOOGLE_PLAY_IAP_INTEGRATION_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-011

---

### MON-014: IAP Product Catalog
**Priority:** P0 | **Complexity:** S

**Description:**
Define all IAP products with pricing.

**Currency Packs:**
| Pack | Doubloons | Price (USD) | Bonus |
|------|-----------|-------------|-------|
| Small | 100 | $0.99 | - |
| Medium | 550 | $4.99 | +10% |
| Large | 1200 | $9.99 | +20% |
| Mega | 2500 | $19.99 | +25% |
| Ultra | 6500 | $49.99 | +30% |

**Special Packs:**
- Starter Pack: $4.99 (one-time, outfit + 500 Doubloons)
- Battle Pass: $9.99 (equals 950 Doubloons)
- Battle Pass Bundle: $14.99 (Pass + 10 tier skips)

**Acceptance Criteria:**
- [x] All products created in store backends
- [x] Pricing approved for all regions
- [x] Bonus percentages calculated correctly
- [x] Starter pack purchase limited
- [x] Product IDs documented

**Definition of Done:**
- All products live and purchasable
- Pricing competitive with market

**Deliverable:** [IAP Product Catalog Specification](docs/IAP_PRODUCT_CATALOG_SPEC.md)

**Status:** Completed

**Dependencies:** MON-012, MON-013

---

### MON-015: Purchase Verification Service
**Priority:** P0 | **Complexity:** M

**Description:**
Server-side purchase verification to prevent fraud.

**Acceptance Criteria:**
- [x] Server validates receipts/tokens before delivery
- [x] Invalid receipts rejected
- [x] Replay attacks prevented (receipt used once)
- [x] Fraud logging for analysis
- [x] Webhook for refund notifications
- [x] Retry logic for failed validations

**Definition of Done:**
- No fraudulent purchases possible
- Validation fast (< 2 seconds)
- Comprehensive logging

**Deliverable:** [Purchase Verification Service Specification](docs/PURCHASE_VERIFICATION_SERVICE_SPEC.md)

**Status:** Completed

**Dependencies:** MON-012, MON-013, BACK-011

---

## 5. Advertising

### MON-016: Rewarded Video Ads
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Implement optional rewarded video ads for non-paying players.

**Ad Placements:**
- Post-match: "Watch ad for +50% Gold"
- Daily bonus: "Watch ad for double daily login reward"
- Energy/cooldown skip: (If applicable)
- Free spin: "Watch ad for free loot crate"

**Acceptance Criteria:**
- [x] Ad SDK integrated (Unity Ads, AdMob, IronSource)
- [x] Ads strictly opt-in (player choice)
- [x] Clear reward shown before watching
- [x] Reward delivered after completion
- [x] Handle ad not available gracefully
- [x] Daily limit on rewarded ads (5-10 per day)
- [x] No ads during match gameplay
- [x] Mediation for best fill rates

**Definition of Done:**
- Rewarded ads functional
- Revenue generated without hurting UX
- Fill rates acceptable (>90%)

**Deliverables:**
- [Rewarded Video Ads Specification](docs/REWARDED_VIDEO_ADS_SPEC.md)

**Dependencies:** BACK-011

---

### MON-017: Ad Mediation and Analytics
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Implement ad mediation to maximize ad revenue.

**Acceptance Criteria:**
- [x] Multiple ad networks integrated
- [x] Mediation layer optimizes for eCPM
- [x] Ad analytics tracked (impressions, revenue)
- [x] A/B test different placements
- [x] Regional performance monitored

**Definition of Done:**
- Ad revenue optimized
- Analytics actionable
- No negative UX impact

**Deliverables:**
- [Ad Mediation and Analytics Specification](docs/AD_MEDIATION_ANALYTICS_SPEC.md)

**Dependencies:** MON-016

---

## 6. Economy Balancing

### MON-018: Economy Spreadsheet
**Priority:** P0 | **Complexity:** M

**Description:**
Create comprehensive economy model spreadsheet for balancing.

**Spreadsheet Contents:**
- All currency earn rates
- All item prices
- Battle Pass progression math
- Time-to-earn calculations
- Whale vs. free player spending models
- Revenue projections

**Acceptance Criteria:**
- [x] All economy values documented
- [x] Formulas calculate time-to-earn
- [x] Multiple player archetype models
- [x] Sensitivity analysis for key variables
- [x] Version controlled
- [x] Updated with each economy change

**Definition of Done:**
- Economy fully modeled
- Used for all balance decisions
- Validated against live data post-launch

**Deliverable:** [docs/ECONOMY_SPREADSHEET.md](docs/ECONOMY_SPREADSHEET.md)

**Dependencies:** MON-001, MON-002, MON-003

**Status:** Completed

---

### MON-019: Economy Monitoring Dashboard
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Build dashboard to monitor live economy health.

**Metrics Tracked:**
- Currency earned per day (global, per user)
- Currency spent per day
- Currency balance distribution
- Top spenders
- Item purchase rates
- Battle Pass completion rate
- IAP conversion rate

**Acceptance Criteria:**
- [x] Real-time or daily updated dashboard
- [x] Alerts for anomalies (inflation, exploit)
- [x] Historical trend tracking
- [x] Exportable reports
- [x] Accessible to relevant team members

**Definition of Done:**
- Economy health visible at a glance
- Issues detectable early
- Data-driven decisions enabled

**Deliverables:**
- [Economy Monitoring Dashboard Specification](docs/ECONOMY_MONITORING_DASHBOARD_SPEC.md)

**Dependencies:** BACK-019, MON-001

---

### MON-020: Pricing Localization
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Implement region-appropriate pricing for IAP.

**Acceptance Criteria:**
- [x] Prices set per region in store backends
- [x] Lower prices for lower GDP regions (where appropriate)
- [x] Currency display matches region
- [x] Compliance with regional regulations
- [x] Exchange rate considerations for value perception

**Definition of Done:**
- Prices appropriate globally
- No lost revenue from mispricing
- Tested in target regions

**Deliverables:**
- [Pricing Localization Specification](docs/PRICING_LOCALIZATION_SPEC.md)

**Dependencies:** MON-012, MON-013

---

### MON-021: Anti-Exploit Measures
**Priority:** P1 | **Complexity:** M

**Description:**
Prevent economy exploits and abuse.

**Protections:**
- Server-side validation of all economy changes
- Rate limiting on earn actions
- Duplicate purchase prevention
- Rollback capability for exploited currency
- Monitoring for anomalous earning

**Acceptance Criteria:**
- [x] All currency changes server-authoritative
- [x] Rate limits on quests, matches, etc.
- [x] Anomaly detection alerts
- [x] Manual adjustment tools for support
- [x] Audit log of all transactions

**Definition of Done:**
- No known exploits possible
- Quick response to discovered exploits
- Player trust maintained

**Deliverable:** [Anti-Exploit Measures Specification](docs/ANTI_EXPLOIT_MEASURES_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-015, MON-019

---

## Task Summary

| Task ID | Title | Priority | Complexity | Status |
|---------|-------|----------|------------|--------|
| MON-001 | Dual Currency System | P0 | M | Completed |
| MON-002 | Currency Earning Rates | P1 | S | Completed |
| MON-003 | Currency Spending Sinks | P1 | S | Completed |
| MON-004 | Store UI Framework | P1 | M | Completed |
| MON-005 | Featured/Daily Shop Rotation | P1 | M | Completed |
| MON-006 | Bundle System | P1 | S | Completed |
| MON-007 | Cosmetic Inventory System | P0 | M | Completed |
| MON-008 | Battle Pass Framework | P1 | L | Completed |
| MON-009 | Battle Pass Rewards Design | P1 | M | Completed |
| MON-010 | Battle Pass XP System | P1 | M | Completed |
| MON-011 | Season Management | P1 | M | Completed |
| MON-012 | App Store IAP Integration | P0 | M | Completed |
| MON-013 | Google Play IAP Integration | P0 | M | Completed |
| MON-014 | IAP Product Catalog | P0 | S | Completed |
| MON-015 | Purchase Verification Service | P0 | M | Completed |
| MON-016 | Rewarded Video Ads | P2 | M | Completed |
| MON-017 | Ad Mediation and Analytics | P2 | S | Completed |
| MON-018 | Economy Spreadsheet | P0 | M | Completed |
| MON-019 | Economy Monitoring Dashboard | P2 | M | Completed |
| MON-020 | Pricing Localization | P2 | S | Completed |
| MON-021 | Anti-Exploit Measures | P1 | M | Completed |
