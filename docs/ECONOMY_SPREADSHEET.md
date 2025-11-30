# Economy Spreadsheet - Plunderstorm Mobile

**Document ID:** MON-018
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Active - Living Document

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Currency System](#2-currency-system)
3. [Currency Earning Rates](#3-currency-earning-rates)
4. [Item Pricing Structure](#4-item-pricing-structure)
5. [Battle Pass Economy](#5-battle-pass-economy)
6. [Time-to-Earn Calculations](#6-time-to-earn-calculations)
7. [Player Archetype Models](#7-player-archetype-models)
8. [Revenue Projections](#8-revenue-projections)
9. [Sensitivity Analysis](#9-sensitivity-analysis)
10. [Economy Health Metrics](#10-economy-health-metrics)
11. [Balancing Guidelines](#11-balancing-guidelines)
12. [Version History](#12-version-history)

---

## 1. Executive Summary

### 1.1 Economy Philosophy

Plunderstorm Mobile employs a **dual-currency free-to-play model** designed to:

1. **Respect Player Time**: Free players can earn meaningful rewards through gameplay
2. **Reward Engagement**: Consistent play yields better returns than sporadic sessions
3. **Offer Value**: Premium purchases feel worthwhile without being mandatory
4. **Maintain Fairness**: No pay-to-win mechanics; all monetization is cosmetic

### 1.2 Key Economy Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Soft Currency | Gold Doubloons | Earned through gameplay |
| Hard Currency | Gems | Premium currency (IAP) |
| Gem-to-USD Ratio | 100 Gems = $0.99 | Base conversion rate |
| Target Session Length | 20-30 minutes | 3-4 matches |
| Target Sessions/Day | 2-3 sessions | Casual player |
| F2P Earning Rate | ~5,000 Gold/day | Active play |
| Battle Pass Price | 950 Gems (~$9.99) | Per season |
| Season Length | 8 weeks | ~56 days |

### 1.3 Economy Goals

| Metric | Target | Rationale |
|--------|--------|-----------|
| D1 Retention | 40%+ | Strong first session economy |
| D7 Retention | 20%+ | Meaningful weekly progression |
| D30 Retention | 10%+ | Long-term engagement loops |
| Conversion Rate | 3-5% | IAP purchasers |
| ARPDAU | $0.08-0.15 | Average revenue per daily user |
| ARPPU | $15-25 | Average revenue per paying user |

---

## 2. Currency System

### 2.1 Primary Currencies

#### 2.1.1 Gold Doubloons (Soft Currency)

| Attribute | Value |
|-----------|-------|
| Symbol | 🪙 |
| Max Storage | Unlimited |
| Primary Source | Match rewards |
| Primary Sink | Common cosmetics, upgrades |
| Daily Soft Cap | 15,000 (reduced rewards after) |

#### 2.1.2 Gems (Hard Currency)

| Attribute | Value |
|-----------|-------|
| Symbol | 💎 |
| Max Storage | Unlimited |
| Primary Source | IAP, Battle Pass, rare rewards |
| Primary Sink | Premium cosmetics, Battle Pass |
| Free Gems/Week | ~100 (from various sources) |

### 2.2 Currency Conversion

**Gold to Gems:** Not allowed (prevents economy exploitation)
**Gems to Gold:** Allowed at controlled rate

| Gem Amount | Gold Received | Efficiency |
|------------|---------------|------------|
| 100 Gems | 5,000 Gold | 50 Gold/Gem |
| 500 Gems | 27,500 Gold | 55 Gold/Gem (+10%) |
| 1,000 Gems | 60,000 Gold | 60 Gold/Gem (+20%) |

### 2.3 Secondary Currencies/Tokens

| Token | Earn Method | Use | Expiry |
|-------|-------------|-----|--------|
| Battle Pass XP | Quests, matches | BP progression | End of season |
| Event Tokens | Event activities | Event rewards | End of event |
| Ranked Points | Ranked matches | Rank progression | Season reset |
| Dust | Duplicate cosmetics | Craft cosmetics | Never |

---

## 3. Currency Earning Rates

### 3.1 Match Rewards (Gold Doubloons)

#### Base Match Rewards

| Placement | Solo | Duo (per player) | Squad (per player) |
|-----------|------|------------------|-------------------|
| 1st | 500 | 450 | 400 |
| 2nd | 350 | 315 | 280 |
| 3rd | 275 | 248 | 220 |
| 4th-5th | 200 | 180 | 160 |
| 6th-10th | 150 | 135 | 120 |
| 11th-20th | 100 | 90 | 80 |
| 21st-40th | 60 | 54 | 48 |
| 41st-60th | 30 | 27 | 24 |

#### Performance Bonuses

| Achievement | Gold Bonus |
|-------------|------------|
| Per Elimination | +25 |
| Per Assist | +10 |
| Damage Dealt (per 100) | +5 |
| Survival Time (per minute) | +8 |
| First Blood | +50 |
| Most Eliminations | +75 |
| Most Damage | +50 |

#### Match Multipliers

| Condition | Multiplier |
|-----------|------------|
| First Win of Day | 2.0x |
| Battle Pass Active | 1.2x |
| Weekend Bonus | 1.25x |
| VIP Status | 1.1x |
| Event Period | 1.5x |

### 3.2 Quest Rewards

#### Daily Quests

| Quest Type | Gold Reward | XP Reward | Refresh |
|------------|-------------|-----------|---------|
| Easy (e.g., Play 1 match) | 200 | 500 | Daily |
| Medium (e.g., 3 eliminations) | 400 | 1,000 | Daily |
| Hard (e.g., Top 10 finish) | 600 | 1,500 | Daily |
| **Daily Total** | **1,200** | **3,000** | - |

#### Weekly Quests

| Quest Type | Gold Reward | XP Reward | Refresh |
|------------|-------------|-----------|---------|
| Play 20 matches | 1,500 | 5,000 | Weekly |
| Deal 10,000 damage | 1,500 | 5,000 | Weekly |
| Win 5 matches | 2,000 | 7,500 | Weekly |
| Complete all dailies (7 days) | 2,500 | 10,000 | Weekly |
| **Weekly Total** | **7,500** | **27,500** | - |

### 3.3 Achievement Rewards

#### One-Time Achievements

| Category | Examples | Gold Range | Gem Range |
|----------|----------|------------|-----------|
| Beginner | First match, First win | 100-500 | 10-25 |
| Progression | Level 10, 25, 50 | 500-2,500 | 25-100 |
| Mastery | 100 wins, 1000 eliminations | 2,500-10,000 | 100-500 |
| Collection | Own 10, 25, 50 cosmetics | 1,000-5,000 | 50-250 |
| Social | Add 5 friends, Play with squad | 500-1,500 | 25-75 |

**Total Achievable Gems (All-Time):** ~2,500 Gems (~$25 value)

### 3.4 Login Rewards

#### Daily Login Calendar (7-Day Cycle)

| Day | Reward |
|-----|--------|
| 1 | 500 Gold |
| 2 | 750 Gold |
| 3 | 25 Gems |
| 4 | 1,000 Gold |
| 5 | 1,500 Gold |
| 6 | 50 Gems |
| 7 | 2,500 Gold + Rare Chest |
| **Weekly Total** | **6,250 Gold + 75 Gems** |

#### Monthly Cumulative Rewards

| Days Logged | Bonus Reward |
|-------------|--------------|
| 7 days | Common Cosmetic Chest |
| 14 days | 100 Gems |
| 21 days | Rare Cosmetic Chest |
| 28 days | 200 Gems + Epic Cosmetic |

### 3.5 Free Gem Sources

| Source | Amount | Frequency |
|--------|--------|-----------|
| Daily Login (Day 3, 6) | 75 | Weekly |
| Free Battle Pass Tier | ~150 | Per Season |
| Achievements | Variable | One-time |
| Events | ~50-100 | Per Event |
| Compensation/Gifts | Variable | Occasional |
| **Average Weekly Free Gems** | **~100** | - |

---

## 4. Item Pricing Structure

### 4.1 Cosmetic Pricing Tiers

#### Gold Doubloon Pricing

| Rarity | Price Range | Examples |
|--------|-------------|----------|
| Common | 500-1,500 Gold | Basic recolors, simple patterns |
| Uncommon | 2,000-4,000 Gold | Themed variants, patterns |
| Rare | 5,000-10,000 Gold | Unique designs, effects |
| Epic | 15,000-25,000 Gold | Premium designs (limited selection) |

#### Gem Pricing

| Rarity | Price Range | Examples |
|--------|-------------|----------|
| Rare | 200-400 Gems | Exclusive designs |
| Epic | 500-800 Gems | Themed sets, effects |
| Legendary | 1,000-1,500 Gems | Ultimate skins, full sets |
| Mythic | 2,000-2,500 Gems | Limited edition, animated |

### 4.2 Cosmetic Categories

#### Character Skins

| Type | Gold Price | Gem Price |
|------|------------|-----------|
| Recolor | 1,000 | - |
| Uncommon Outfit | 3,500 | - |
| Rare Outfit | 8,000 | 350 |
| Epic Outfit | 20,000 | 650 |
| Legendary Outfit | - | 1,200 |
| Mythic Outfit | - | 2,200 |

#### Ability Effects

| Type | Gold Price | Gem Price |
|------|------------|-----------|
| Basic Trail/Effect | 2,000 | - |
| Themed Effect | 5,000 | 250 |
| Premium Effect | - | 500 |
| Ultimate Effect | - | 800 |

#### Emotes

| Type | Gold Price | Gem Price |
|------|------------|-----------|
| Common Emote | 1,500 | - |
| Rare Emote | 4,000 | 200 |
| Epic Emote | - | 400 |
| Legendary Emote | - | 700 |

#### Victory Animations

| Type | Gold Price | Gem Price |
|------|------------|-----------|
| Basic Victory | 3,000 | - |
| Themed Victory | 7,500 | 300 |
| Premium Victory | - | 600 |
| Ultimate Victory | - | 1,000 |

### 4.3 Bundle Pricing

#### Starter Pack (One-Time)
| Contents | Value | Price | Discount |
|----------|-------|-------|----------|
| 500 Gems | $4.99 | - | - |
| Rare Outfit | $3.50 | - | - |
| 5,000 Gold | $0.83 | - | - |
| **Total Value** | **$9.32** | **$4.99** | **46%** |

#### Value Bundles

| Bundle | Contents | Value | Price | Discount |
|--------|----------|-------|-------|----------|
| Small Bundle | Rare Skin + 200 Gems | $5.50 | $3.99 | 27% |
| Medium Bundle | Epic Skin + 500 Gems | $11.50 | $7.99 | 31% |
| Large Bundle | Legendary Skin + 1000 Gems | $22.00 | $14.99 | 32% |
| Ultimate Bundle | Mythic Skin + 2000 Gems | $42.00 | $29.99 | 29% |

### 4.4 IAP Gem Packages

| Package | Gems | Bonus | Total | Price | $/Gem |
|---------|------|-------|-------|-------|-------|
| Handful | 100 | 0 | 100 | $0.99 | $0.0099 |
| Pouch | 550 | 50 | 550 | $4.99 | $0.0091 |
| Sack | 1,200 | 200 | 1,200 | $9.99 | $0.0083 |
| Chest | 2,500 | 500 | 2,500 | $19.99 | $0.0080 |
| Treasure | 5,500 | 1,300 | 5,500 | $39.99 | $0.0073 |
| Hoard | 12,000 | 3,500 | 12,000 | $79.99 | $0.0067 |
| Fortune | 25,000 | 8,000 | 25,000 | $149.99 | $0.0060 |

**Best Value Indicator:** Fortune package (40% more value than Handful)

---

## 5. Battle Pass Economy

### 5.1 Battle Pass Structure

| Attribute | Free Track | Premium Track |
|-----------|------------|---------------|
| Price | Free | 950 Gems (~$9.99) |
| Total Tiers | 100 | 100 |
| Rewards per Tier | Variable | Variable |
| XP per Tier | 1,000 base | Same |
| Total XP Required | 100,000 | Same |

### 5.2 Battle Pass Rewards Distribution

#### Free Track Rewards (100 Tiers)

| Reward Type | Quantity | Per-Item Value | Total Value |
|-------------|----------|----------------|-------------|
| Gold | 25,000 total | - | ~$4.50 |
| Gems | 150 total | $1.50 | $1.50 |
| Common Cosmetics | 3 | $0.50 | $1.50 |
| Uncommon Cosmetics | 2 | $1.00 | $2.00 |
| Rare Cosmetics | 1 | $3.00 | $3.00 |
| Chests | 5 | $1.00 | $5.00 |
| **Free Track Total Value** | | | **~$17.50** |

#### Premium Track Rewards (100 Tiers)

| Reward Type | Quantity | Per-Item Value | Total Value |
|-------------|----------|----------------|-------------|
| Gold | 35,000 total | - | ~$6.00 |
| Gems | 500 total | $5.00 | $5.00 |
| Common Cosmetics | 5 | $0.50 | $2.50 |
| Uncommon Cosmetics | 6 | $1.00 | $6.00 |
| Rare Cosmetics | 8 | $3.00 | $24.00 |
| Epic Cosmetics | 4 | $6.50 | $26.00 |
| Legendary Cosmetics | 2 | $12.00 | $24.00 |
| Mythic Cosmetic (Tier 100) | 1 | $22.00 | $22.00 |
| Chests | 8 | $1.50 | $12.00 |
| **Premium Track Total Value** | | | **~$127.50** |

#### Combined Value

| Track | Cost | Value | ROI |
|-------|------|-------|-----|
| Free | $0 | $17.50 | ∞ |
| Premium | $9.99 | $145.00 | 14.5x |

### 5.3 Battle Pass XP Economy

#### XP Progression Curve

| Tier Range | XP per Tier | Cumulative XP |
|------------|-------------|---------------|
| 1-20 | 800 | 16,000 |
| 21-40 | 900 | 34,000 |
| 41-60 | 1,000 | 54,000 |
| 61-80 | 1,100 | 76,000 |
| 81-100 | 1,200 | 100,000 |

#### XP Earning Methods

| Source | XP Amount | Frequency |
|--------|-----------|-----------|
| Match Completion | 100-300 | Per match |
| Match Win | +150 | Per win |
| Daily Quest (Easy) | 500 | Daily |
| Daily Quest (Medium) | 1,000 | Daily |
| Daily Quest (Hard) | 1,500 | Daily |
| Weekly Quest | 5,000-10,000 | Weekly |
| BP Challenge | 2,500-5,000 | Per challenge |

#### Daily/Weekly XP Caps

| Source | Daily Cap | Weekly Cap |
|--------|-----------|------------|
| Matches | 3,000 XP | 21,000 XP |
| Daily Quests | 3,000 XP | 21,000 XP |
| Weekly Quests | - | 27,500 XP |
| Challenges | - | 15,000 XP |
| **Total Available** | **6,000/day** | **84,500/week** |

### 5.4 Battle Pass Completion Math

**Season Length:** 56 days (8 weeks)

| Player Type | Weekly XP | Total XP | Completion |
|-------------|-----------|----------|------------|
| Casual (3 days/week) | 25,000 | 200,000 | 200% (Tier 100+) |
| Regular (5 days/week) | 50,000 | 400,000 | 400% (Tier 100+) |
| Hardcore (7 days/week) | 84,500 | 676,000 | 676% (Tier 100+) |
| Minimal (2 days/week) | 15,000 | 120,000 | 120% (Tier 100) |

**Conclusion:** Even minimal engagement completes Battle Pass

### 5.5 Premium BP+ Option

| Attribute | Value |
|-----------|-------|
| Price | 1,800 Gems (~$18.99) |
| Includes | Premium BP + 25 Tier Skips |
| Best For | Late-season purchasers |
| Value vs Standard | +$5 for 25 tiers (~$15 value) |

---

## 6. Time-to-Earn Calculations

### 6.1 Assumptions

| Parameter | Value |
|-----------|-------|
| Average Match Duration | 15 minutes |
| Average Placement | 20th (median) |
| Average Eliminations | 2 |
| Quest Completion Rate | 80% |
| Daily Play Sessions | 2 |
| Session Duration | 30 minutes (2 matches) |

### 6.2 Daily Earning Potential

#### Casual Player (30 min/day)

| Source | Gold | Gems | Time |
|--------|------|------|------|
| 2 Matches | 350 | 0 | 30 min |
| 2 Daily Quests | 600 | 0 | Passive |
| Daily Login | ~900 | ~11 | 1 min |
| **Daily Total** | **1,850** | **11** | **31 min** |

#### Regular Player (1 hour/day)

| Source | Gold | Gems | Time |
|--------|------|------|------|
| 4 Matches | 800 | 0 | 60 min |
| 3 Daily Quests | 1,200 | 0 | Passive |
| Daily Login | ~900 | ~11 | 1 min |
| First Win Bonus | +250 | 0 | Included |
| **Daily Total** | **3,150** | **11** | **61 min** |

#### Hardcore Player (2+ hours/day)

| Source | Gold | Gems | Time |
|--------|------|------|------|
| 8 Matches | 2,000 | 0 | 120 min |
| 3 Daily Quests | 1,200 | 0 | Passive |
| Daily Login | ~900 | ~11 | 1 min |
| First Win Bonus | +500 | 0 | Included |
| Weekly Quests (÷7) | 1,070 | 0 | Passive |
| **Daily Total** | **5,670** | **11** | **121 min** |

### 6.3 Time-to-Earn Specific Items

#### Gold-Purchasable Items

| Item | Price | Casual | Regular | Hardcore |
|------|-------|--------|---------|----------|
| Common Skin | 1,000 | 1 day | <1 day | <1 day |
| Uncommon Skin | 3,500 | 2 days | 1 day | <1 day |
| Rare Skin | 8,000 | 4 days | 3 days | 1.5 days |
| Epic Skin | 20,000 | 11 days | 6 days | 4 days |

#### Gem-Purchasable Items (Free Gems Only)

| Item | Price | Weekly Gems | Time |
|------|-------|-------------|------|
| Battle Pass | 950 | ~100 | ~10 weeks |
| Rare Effect | 250 | ~100 | ~2.5 weeks |
| Epic Skin | 650 | ~100 | ~6.5 weeks |
| Legendary Skin | 1,200 | ~100 | ~12 weeks |

### 6.4 Collection Completion Time

#### Complete Common Collection (F2P)

| Items | Total Cost | Daily Gold | Days |
|-------|------------|------------|------|
| 20 Common | 20,000 Gold | 3,000 | 7 days |

#### Complete Rare Collection (F2P)

| Items | Total Cost | Daily Gold | Days |
|-------|------------|------------|------|
| 15 Rare | 120,000 Gold | 3,000 | 40 days |

#### Complete Full Collection (Impossible F2P)

Many Legendary/Mythic items are Gem-only, requiring purchase or years of free gems.

---

## 7. Player Archetype Models

### 7.1 Archetype Definitions

| Archetype | Description | % of Players |
|-----------|-------------|--------------|
| Non-Payer | Plays for free, never purchases | 70% |
| Minnow | Small occasional purchases | 20% |
| Dolphin | Regular medium purchases | 7% |
| Whale | Heavy spenders | 3% |

### 7.2 Non-Payer Profile

| Attribute | Value |
|-----------|-------|
| Monthly Spend | $0 |
| Play Frequency | 4 days/week |
| Session Length | 30 min |
| Monthly Gold Earned | ~25,000 |
| Monthly Gems Earned | ~400 |
| Items Obtained | 2-3 Common/Uncommon per month |

**Engagement Strategy:**
- Daily login rewards
- Free Battle Pass track
- Achievement gems for aspirational purchases
- Social features to encourage play

### 7.3 Minnow Profile

| Attribute | Value |
|-----------|-------|
| Monthly Spend | $5-15 |
| Typical Purchases | Battle Pass, Starter Pack |
| Play Frequency | 5 days/week |
| Session Length | 45 min |
| Monthly Value Received | $15-45 (Battle Pass value) |

**Typical Monthly Purchases:**
| Purchase | Frequency | Cost |
|----------|-----------|------|
| Battle Pass | Every other season | $5/month avg |
| Small Gem Pack | Occasional | $2.50/month avg |
| Starter Pack | Once | $5 (one-time) |

**Engagement Strategy:**
- Battle Pass value proposition
- Limited-time bundles
- Sale notifications
- Exclusive BP cosmetics

### 7.4 Dolphin Profile

| Attribute | Value |
|-----------|-------|
| Monthly Spend | $20-50 |
| Typical Purchases | BP, Gem packs, Bundles |
| Play Frequency | 6 days/week |
| Session Length | 60 min |
| Monthly Value Received | $60-150 |

**Typical Monthly Purchases:**
| Purchase | Frequency | Cost |
|----------|-----------|------|
| Battle Pass | Every season | $10/month |
| Medium Gem Pack | Monthly | $10-20/month |
| Featured Bundles | Occasional | $8-15/month |

**Engagement Strategy:**
- Exclusive offers
- Bundle discounts
- Early access to new cosmetics
- VIP benefits consideration

### 7.5 Whale Profile

| Attribute | Value |
|-----------|-------|
| Monthly Spend | $100-500+ |
| Typical Purchases | All premium content |
| Play Frequency | Daily |
| Session Length | 90+ min |
| Monthly Value Received | $300-1,500+ |

**Typical Monthly Purchases:**
| Purchase | Frequency | Cost |
|----------|-----------|------|
| Battle Pass + Tiers | Every season | $20-50/month |
| Large Gem Packs | Multiple | $40-160/month |
| All Featured Items | Ongoing | $50-200/month |
| Limited Edition Items | All | $50-100/month |

**Engagement Strategy:**
- Exclusive limited items
- Early/exclusive access
- VIP customer support
- Personalized offers
- Collector achievements

### 7.6 Lifetime Value (LTV) Estimates

| Archetype | % Players | Monthly Spend | 12-Month LTV | 24-Month LTV |
|-----------|-----------|---------------|--------------|--------------|
| Non-Payer | 70% | $0 | $0 | $0 |
| Minnow | 20% | $8 | $96 | $144 |
| Dolphin | 7% | $35 | $420 | $630 |
| Whale | 3% | $200 | $2,400 | $3,600 |
| **Blended Average** | 100% | **$9.25** | **$111** | **$166.50** |

---

## 8. Revenue Projections

### 8.1 Assumptions

| Parameter | Soft Launch | Global Launch | Year 1 Avg |
|-----------|-------------|---------------|------------|
| DAU | 10,000 | 100,000 | 75,000 |
| MAU | 30,000 | 300,000 | 225,000 |
| Conversion Rate | 3% | 4% | 4% |
| ARPDAU | $0.08 | $0.12 | $0.10 |
| ARPPU | $15 | $20 | $18 |

### 8.2 Revenue Breakdown by Source

| Source | % of Revenue | Notes |
|--------|--------------|-------|
| Battle Pass | 35% | Consistent, predictable |
| Gem Purchases | 30% | Direct IAP |
| Bundles | 20% | High conversion |
| Event Purchases | 10% | Seasonal spikes |
| Ads (Optional) | 5% | Rewarded video only |

### 8.3 Monthly Revenue Projections

#### Soft Launch (Month 1-2)

| Metric | Value |
|--------|-------|
| DAU | 10,000 |
| ARPDAU | $0.08 |
| Daily Revenue | $800 |
| Monthly Revenue | $24,000 |

#### Global Launch (Month 3)

| Metric | Value |
|--------|-------|
| DAU | 100,000 |
| ARPDAU | $0.15 (launch boost) |
| Daily Revenue | $15,000 |
| Monthly Revenue | $450,000 |

#### Stabilized (Month 6+)

| Metric | Value |
|--------|-------|
| DAU | 75,000 |
| ARPDAU | $0.10 |
| Daily Revenue | $7,500 |
| Monthly Revenue | $225,000 |

### 8.4 Annual Revenue Projection

| Quarter | DAU (Avg) | Monthly Rev | Quarterly Rev |
|---------|-----------|-------------|---------------|
| Q1 | 25,000 | $62,500 | $187,500 |
| Q2 | 100,000 | $300,000 | $900,000 |
| Q3 | 80,000 | $240,000 | $720,000 |
| Q4 | 70,000 | $210,000 | $630,000 |
| **Year 1 Total** | | | **$2,437,500** |

### 8.5 Revenue per Feature

| Feature | Est. Annual Revenue | % of Total |
|---------|---------------------|------------|
| Battle Pass | $850,000 | 35% |
| Direct Gem Sales | $730,000 | 30% |
| Bundles | $487,500 | 20% |
| Event Items | $243,750 | 10% |
| Rewarded Ads | $126,250 | 5% |

---

## 9. Sensitivity Analysis

### 9.1 Key Variables

| Variable | Base Case | Low Case | High Case |
|----------|-----------|----------|-----------|
| DAU | 75,000 | 50,000 | 150,000 |
| Conversion Rate | 4% | 2% | 6% |
| ARPPU | $18 | $12 | $30 |
| Retention D30 | 10% | 5% | 15% |

### 9.2 Revenue Sensitivity Matrix

#### DAU Impact

| DAU | Monthly Revenue | vs Base |
|-----|-----------------|---------|
| 50,000 | $150,000 | -33% |
| 75,000 | $225,000 | Base |
| 100,000 | $300,000 | +33% |
| 150,000 | $450,000 | +100% |

#### Conversion Rate Impact

| Conv. Rate | Paying Users | Monthly Revenue | vs Base |
|------------|--------------|-----------------|---------|
| 2% | 1,500 | $135,000 | -40% |
| 4% | 3,000 | $225,000 | Base |
| 6% | 4,500 | $315,000 | +40% |
| 8% | 6,000 | $405,000 | +80% |

#### ARPPU Impact

| ARPPU | Monthly Revenue | vs Base |
|-------|-----------------|---------|
| $12 | $144,000 | -36% |
| $18 | $225,000 | Base |
| $25 | $312,500 | +39% |
| $35 | $437,500 | +94% |

### 9.3 Scenario Analysis

#### Pessimistic Scenario

| Variable | Value |
|----------|-------|
| DAU | 40,000 |
| Conversion | 2.5% |
| ARPPU | $14 |
| Monthly Revenue | **$56,000** |
| Annual Revenue | **$672,000** |

#### Base Scenario

| Variable | Value |
|----------|-------|
| DAU | 75,000 |
| Conversion | 4% |
| ARPPU | $18 |
| Monthly Revenue | **$225,000** |
| Annual Revenue | **$2,700,000** |

#### Optimistic Scenario

| Variable | Value |
|----------|-------|
| DAU | 120,000 |
| Conversion | 5.5% |
| ARPPU | $24 |
| Monthly Revenue | **$633,600** |
| Annual Revenue | **$7,603,200** |

### 9.4 Break-Even Analysis

| Cost Category | Monthly Cost |
|---------------|--------------|
| Server Costs | $15,000 |
| Team Salaries | $80,000 |
| Marketing | $30,000 |
| Tools/Services | $5,000 |
| **Total Monthly** | **$130,000** |

**Break-Even Requirements:**
- At $0.10 ARPDAU: 43,333 DAU needed
- At $0.08 ARPDAU: 54,167 DAU needed
- At $0.12 ARPDAU: 36,111 DAU needed

---

## 10. Economy Health Metrics

### 10.1 Key Performance Indicators (KPIs)

| KPI | Target | Warning | Critical |
|-----|--------|---------|----------|
| ARPDAU | $0.08-0.15 | <$0.05 | <$0.03 |
| Conversion Rate | 3-5% | <2% | <1% |
| ARPPU | $15-25 | <$10 | <$8 |
| D1 Retention | 40%+ | <35% | <25% |
| D7 Retention | 20%+ | <15% | <10% |
| D30 Retention | 10%+ | <7% | <5% |
| BP Attach Rate | 15-25% | <10% | <5% |
| Gold Inflation | <5%/month | >10%/month | >20%/month |

### 10.2 Currency Health Metrics

| Metric | Formula | Healthy Range |
|--------|---------|---------------|
| Gold Velocity | Gold Spent / Gold Earned | 0.7-0.9 |
| Gold Stock | Avg Gold Balance / Daily Earn | 3-7 days |
| Gem Conversion | Free Gems / Total Gems Used | 0.2-0.4 |
| Sink Efficiency | Gold Sunk / Gold Available | 0.8-0.95 |

### 10.3 Red Flag Indicators

| Indicator | Possible Cause | Action |
|-----------|----------------|--------|
| Gold accumulating (Stock >10 days) | Insufficient sinks | Add gold sinks, limited items |
| Gold depleting rapidly | Earn rates too low | Increase quest rewards |
| Gem conversion declining | Free gems too low | Add gem sources |
| ARPPU dropping | Value perception down | Improve bundle value |
| Conversion dropping | Paywall too hard | Adjust starter offers |

### 10.4 Monitoring Dashboard

**Daily Checks:**
- Revenue (total, per source)
- DAU, paying users
- Gold earned/spent (global)
- Gem purchased/spent

**Weekly Checks:**
- Retention cohorts
- ARPDAU trend
- Conversion funnel
- Top-selling items

**Monthly Checks:**
- LTV calculations
- Economy balance review
- Price point analysis
- Archetype distribution

---

## 11. Balancing Guidelines

### 11.1 Price Adjustment Rules

#### When to Raise Prices

| Signal | Threshold | Action |
|--------|-----------|--------|
| Item sells out quickly | <1 hour | Consider 10-20% increase |
| Buy rate very high | >25% of eligible users | Test higher price |
| Currency too easy to earn | Inflation >10%/month | Reduce earn, raise prices |

#### When to Lower Prices

| Signal | Threshold | Action |
|--------|-----------|--------|
| Item barely sells | <1% purchase rate | Discount or reprice |
| Gold accumulating | Avg balance >15 days | Add sinks, may lower prices |
| Player complaints | High volume | Review perceived value |

### 11.2 Earn Rate Adjustment Rules

#### Gold Earn Rate

| Situation | Adjustment |
|-----------|------------|
| New players churning before first purchase | Increase early rewards |
| Gold inflation detected | Reduce match rewards 10-20% |
| Engagement dropping | Add bonus events |
| F2P players converting | Maintain (system working) |

#### Gem Earn Rate

| Situation | Adjustment |
|-----------|------------|
| Conversion too low | Increase free gems slightly |
| Conversion healthy | Maintain |
| Gems being hoarded | Add gem-only flash sales |

### 11.3 Battle Pass Tuning

| Issue | Solution |
|-------|----------|
| Completion rate <80% | Reduce XP requirements |
| Completion rate >95% at week 4 | Increase XP curve |
| BP attach rate <10% | Increase visible value |
| BP attach rate >30% | Consider price increase |

### 11.4 Seasonal Adjustments

| Season | Adjustment |
|--------|------------|
| Launch Month | Generous rewards, low prices |
| Month 2-3 | Normalize to base economy |
| Major Update | Temporary boost (1.5x) |
| Holiday Event | Special currency, themed items |
| Low Engagement Period | Comeback rewards, sales |

---

## 12. Version History

### Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial economy model |

### Planned Updates

| Update | Trigger | Notes |
|--------|---------|-------|
| v1.1 | Post-soft launch | Adjust based on real data |
| v1.2 | Post-global launch | Full economy rebalance |
| v2.0 | Year 1 review | Major revision with live data |

### Data Sources

- Industry benchmarks (Sensor Tower, data.ai)
- Competitor analysis
- Internal playtesting
- Focus group feedback

---

## Appendix A: Formula Reference

### Currency Formulas

```
Daily Gold (Casual) = (Matches × Avg Reward) + Quest Gold + Login Gold
                    = (2 × 175) + 600 + 900 = 1,850

Time to Item = Item Price / Daily Earn Rate
             = 8,000 / 3,150 = 2.5 days (Regular player, Rare item)

Battle Pass Value = (Premium Rewards Value) - (BP Cost)
                  = $127.50 - $9.99 = $117.51 net value
```

### Revenue Formulas

```
ARPDAU = Daily Revenue / DAU
       = $7,500 / 75,000 = $0.10

Monthly Revenue = DAU × ARPDAU × 30
                = 75,000 × $0.10 × 30 = $225,000

LTV = ARPDAU × Avg Lifetime Days
    = $0.10 × 180 = $18
```

### Economy Health Formulas

```
Gold Velocity = Total Gold Spent / Total Gold Earned
              = 85,000,000 / 100,000,000 = 0.85 (Healthy)

Gold Stock Days = Avg Player Gold / Daily Gold Earn
                = 15,000 / 3,000 = 5 days (Healthy)

Conversion Rate = Paying Users / Total Users × 100
                = 3,000 / 75,000 × 100 = 4%
```

---

## Appendix B: Competitive Benchmarks

### Similar Mobile Games

| Game | ARPDAU | Conversion | ARPPU | BP Price |
|------|--------|------------|-------|----------|
| Game A | $0.12 | 4.5% | $22 | $9.99 |
| Game B | $0.08 | 3.2% | $18 | $7.99 |
| Game C | $0.15 | 5.1% | $28 | $12.99 |
| **Our Target** | **$0.10** | **4%** | **$18** | **$9.99** |

### Industry Averages (Mobile F2P)

| Metric | Casual | Mid-Core | Our Target |
|--------|--------|----------|------------|
| D1 Retention | 35% | 40% | 40% |
| D7 Retention | 15% | 18% | 20% |
| D30 Retention | 6% | 9% | 10% |
| Conversion | 2-3% | 3-5% | 4% |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Game Designer | _____________ | _____________ | ___/___/___ |
| Producer | _____________ | _____________ | ___/___/___ |
| Finance | _____________ | _____________ | ___/___/___ |

---

*This is a living document. Update with each economy change and validate against live data post-launch.*
