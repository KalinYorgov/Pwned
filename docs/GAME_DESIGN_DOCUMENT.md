# Game Design Document (GDD)
# Plunderstorm Mobile - Pirate Battle Royale

**Document Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Draft
**Task ID:** PROJ-001

---

## Table of Contents

1. [Game Overview](#1-game-overview)
2. [Core Gameplay Loop](#2-core-gameplay-loop)
3. [Game Mechanics](#3-game-mechanics)
4. [Ability System Design](#4-ability-system-design)
5. [Progression Systems](#5-progression-systems)
6. [Monetization Design](#6-monetization-design)
7. [Technical Requirements](#7-technical-requirements)
8. [Art Direction Summary](#8-art-direction-summary)
9. [Audio Design](#9-audio-design)
10. [User Interface](#10-user-interface)
11. [Appendices](#11-appendices)

---

## 1. Game Overview

### 1.1 High Concept

**Plunderstorm Mobile** is a free-to-play mobile battle royale game set in a vibrant pirate fantasy world. Unlike traditional shooters, players collect magical abilities instead of weapons, creating a unique spell-slinging battle royale experience. Drop onto a mystical island, scavenge for powerful abilities, fight monsters and players, and be the last pirate standing.

### 1.2 Genre

- **Primary:** Battle Royale
- **Secondary:** Action RPG, Ability-based Combat
- **Platform:** iOS and Android Mobile

### 1.3 Target Audience

| Demographic | Description |
|-------------|-------------|
| **Age** | 13-35 years old |
| **Primary** | Mobile gamers who enjoy battle royale (PUBG Mobile, Free Fire, Fortnite) |
| **Secondary** | MOBA players (Mobile Legends, Wild Rift) looking for shorter sessions |
| **Tertiary** | Fantasy RPG fans interested in ability-based combat |
| **Region** | Global, with focus on NA, EU, SEA, LATAM |

### 1.4 Unique Selling Points (USPs)

1. **Ability-Based Combat:** No guns or traditional weapons - collect and combine magical abilities
2. **Pirate Fantasy Theme:** Whimsical, colorful world with pirates, magic, and sea creatures
3. **PvE + PvP Hybrid:** Fight both monsters and players for progression
4. **Quick Sessions:** 10-15 minute matches perfect for mobile
5. **Fair Play:** No pay-to-win; skill and strategy determine winners
6. **Upgrade During Match:** Abilities can be upgraded by finding duplicates

### 1.5 Game Pillars

These are the core values that guide all design decisions:

| Pillar | Description |
|--------|-------------|
| **Accessible** | Easy to learn, intuitive controls, quick to start playing |
| **Skill-Based** | Player skill matters more than loadout or spending |
| **Dynamic** | Every match feels different due to ability combinations |
| **Rewarding** | Clear progression, satisfying combat feedback, generous rewards |
| **Social** | Fun with friends, team modes, social features |

### 1.6 Competitive Analysis

| Game | Similarity | Our Differentiation |
|------|------------|---------------------|
| Free Fire | Mobile BR, quick matches | We use abilities instead of guns |
| PUBG Mobile | Mobile BR, realistic | We're stylized and fantasy-themed |
| Fortnite | BR with building | We have ability combos instead of building |
| Brawl Stars | Mobile, abilities | We're larger scale BR, not arena |
| Spellbreak | Ability BR | We're mobile-first, pirate theme |

### 1.7 Reference Games

- **Blizzard's Plunderstorm** - Primary inspiration for ability-based BR
- **Fortnite** - Art style, battle pass, seasonal model
- **Free Fire** - Mobile optimization, short matches
- **Brawl Stars** - Touch controls, character abilities
- **Sea of Thieves** - Pirate theme and aesthetics

---

## 2. Core Gameplay Loop

### 2.1 Match Flow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        MATCH FLOW                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  QUEUE ──► LOBBY ──► DROP ──► LOOT ──► FIGHT ──► VICTORY/DEFEAT │
│   │         │         │        │        │            │          │
│   │         │         │        │        │            │          │
│   ▼         ▼         ▼        ▼        ▼            ▼          │
│  ~10s      ~30s      ~15s    Ongoing  Ongoing     Results       │
│                                                                  │
│              Total Match Time: 10-15 minutes                     │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Detailed Match Phases

#### Phase 1: Queue & Matchmaking (10-30 seconds)
- Player selects game mode (Solo, Duo)
- Matchmaking finds appropriate players
- Bot fill if insufficient players
- Regional matching for low latency

#### Phase 2: Lobby (30 seconds)
- Players spawn in "Tavern" lobby area
- Can preview other players, emote
- Countdown timer to match start
- Quick tips and loading

#### Phase 3: Drop Phase (15-20 seconds)
- Map overview shown
- Players select drop location (tap to mark)
- Pirates launched from flying ship via cannon
- Glide control during descent
- Land and begin looting immediately

#### Phase 4: Early Game (0-4 minutes)
- Loot abilities from treasure chests
- Clear monster camps for XP
- Avoid or engage other players
- Storm circle is large, minimal pressure

#### Phase 5: Mid Game (4-8 minutes)
- Storm begins shrinking
- Players have 2-4 abilities
- PvP encounters more frequent
- Level 5-7 typical
- World events may spawn

#### Phase 6: Late Game (8-12 minutes)
- Storm forces remaining players together
- Most players near max level (10)
- High-tier abilities equipped
- Intense PvP combat
- Final circle showdowns

#### Phase 7: End Game (12-15 minutes)
- Final players battle
- Tiny safe zone
- Victory for last player/team standing

#### Phase 8: Results
- Placement shown (#1, Top 10, etc.)
- Stats displayed (kills, damage, abilities used)
- Rewards granted (XP, currency, battle pass progress)
- Option to play again or return to lobby

### 2.3 Moment-to-Moment Gameplay

**Every 30 seconds, a player should:**
- Make a meaningful decision (loot this chest? attack this player? farm this camp?)
- Experience excitement or tension
- Progress toward victory (XP gain, ability upgrade, elimination)

**Combat Rhythm:**
1. Spot enemy (visual/audio cue)
2. Evaluate (their level? abilities? health?)
3. Engage or evade
4. If engage: Use ability combo (2-4 abilities in sequence)
5. React to enemy abilities (dodge, counter)
6. Secure kill or disengage
7. Loot and reposition

### 2.4 Session Structure

| Session Type | Duration | Activities |
|--------------|----------|------------|
| Quick Session | 5-10 min | 1 match (may die early) |
| Regular Session | 20-30 min | 2-3 matches |
| Extended Session | 45-60 min | Complete daily quests, 4-5 matches |

---

## 3. Game Mechanics

### 3.1 Movement System

| Action | Control | Speed | Notes |
|--------|---------|-------|-------|
| Walk | Joystick partial | 3 m/s | Default movement |
| Run | Joystick full | 5 m/s | Standard speed |
| Sprint | Double-tap + hold | 7 m/s | Drains stamina (optional) |
| Jump | Jump button | - | 2m height |
| Double Jump | Jump in air | - | Additional 1.5m (unlocked by default) |
| Swim | In water | 3 m/s | Can be attacked while swimming |
| Climb | Near climbable surface | 2 m/s | Auto-vault low obstacles |

### 3.2 Combat System

#### 3.2.1 Basic Attack
- **Always available** (not an ability slot)
- **Type:** Ranged magical bolt
- **Damage:** 15 per hit
- **Rate of Fire:** 2 shots/second
- **Range:** 25 meters
- **Aim Assist:** Optional soft-lock on mobile

#### 3.2.2 Health System
| Level | Base HP | HP Regen |
|-------|---------|----------|
| 1 | 100 | None |
| 5 | 150 | None |
| 10 | 200 | None |

- No passive health regeneration
- Healing only through abilities or consumables
- Shield system: Future consideration (not MVP)

#### 3.2.3 Damage Types
| Type | Color | Notes |
|------|-------|-------|
| Physical | White | Basic attacks, melee |
| Fire | Orange | DoT potential |
| Ice | Blue | Slow effect |
| Lightning | Yellow | Chain potential |
| Poison | Green | DoT |
| Arcane | Purple | Pure magic |

#### 3.2.4 Combat Flow
```
ENGAGE ──► OPENER ──► COMBO ──► FINISH ──► DISENGAGE
   │          │          │         │           │
   ▼          ▼          ▼         ▼           ▼
 Spot     Gap Close   Damage    Execute    Loot/Flee
 Enemy    or Poke     Rotation   or Chase
```

### 3.3 Loot System

#### 3.3.1 Treasure Chests
| Rarity | Color | Spawn Rate | Contents |
|--------|-------|------------|----------|
| Common | Brown | 60% | 1 Common ability |
| Uncommon | Green | 25% | 1-2 Uncommon abilities |
| Rare | Blue | 12% | 1 Rare ability + consumable |
| Epic | Purple | 3% | 1-2 Epic abilities |

#### 3.3.2 Ability Acquisition
1. Approach chest (auto-highlight within 5m)
2. Tap interact (0.75s channel, interruptible)
3. Ability auto-equips to empty slot
4. If slots full, choose to swap or leave

#### 3.3.3 Loot Locations
- **High Density:** Named POIs (Port Plunder, Skull Cove)
- **Medium Density:** Smaller landmarks
- **Low Density:** Wilderness areas
- **Monster Drops:** Guaranteed from elite monsters

### 3.4 Storm/Zone System

#### 3.4.1 Storm Phases
| Phase | Time | Safe Zone | Damage/sec | Warning |
|-------|------|-----------|------------|---------|
| 1 | 0:00-3:00 | 100% → 70% | 2 | 30s |
| 2 | 3:00-6:00 | 70% → 40% | 4 | 20s |
| 3 | 6:00-9:00 | 40% → 20% | 6 | 15s |
| 4 | 9:00-12:00 | 20% → 5% | 8 | 10s |
| 5 | 12:00+ | 5% → 0% | 10 | 10s |

#### 3.4.2 Storm Behavior
- Storm is a mystical magical barrier
- Visibility reduced inside storm
- Audio muffled in storm
- Players can survive briefly in storm with healing
- Storm center is semi-random (weighted toward map center)

### 3.5 PvE System

#### 3.5.1 Monster Types
| Monster | HP | Damage | XP | Drop |
|---------|-----|--------|-----|------|
| Skeleton Pirate | 50 | 10 | 20 | 20% Common |
| Giant Crab | 100 | 20 | 40 | 50% Uncommon |
| Cursed Sailor | 75 | 15 (ranged) | 35 | 40% Uncommon |
| Sea Serpent (Elite) | 300 | 30 | 100 | 100% Rare |
| Kraken Tentacle (Boss) | 500 | 40 | 200 | 100% Epic |

#### 3.5.2 Monster Camp Respawn
- Camps respawn 90 seconds after cleared
- Visual indicator when camp is available
- Camps are contested (multiple players can fight)

---

## 4. Ability System Design

### 4.1 Ability Slot Structure

```
┌─────────────────────────────────────────────────┐
│              ABILITY LOADOUT                     │
├─────────────────────────────────────────────────┤
│                                                  │
│  [BASIC ATTACK] - Always Available (No Slot)    │
│                                                  │
│  ┌─────────┐  ┌─────────┐                       │
│  │OFFENSIVE│  │OFFENSIVE│  Slots 1-2            │
│  │  SLOT 1 │  │  SLOT 2 │  (Red Border)         │
│  └─────────┘  └─────────┘                       │
│                                                  │
│  ┌─────────┐  ┌─────────┐                       │
│  │ UTILITY │  │ UTILITY │  Slots 3-4            │
│  │  SLOT 3 │  │  SLOT 4 │  (Blue Border)        │
│  └─────────┘  └─────────┘                       │
│                                                  │
└─────────────────────────────────────────────────┘
```

### 4.2 Ability Rarity & Upgrades

| Rank | Name | Border | Damage Mult | Cooldown Mult | How to Get |
|------|------|--------|-------------|---------------|------------|
| 1 | Common | Gray | 1.0x | 1.0x | Loot drop |
| 2 | Rare | Blue | 1.25x | 0.9x | Rare chest or duplicate |
| 3 | Epic | Purple | 1.5x | 0.8x | Epic chest or duplicate |

**Upgrade Mechanic:** Finding a duplicate ability upgrades the existing one to the next rank.

### 4.3 Complete Ability Roster

#### 4.3.1 Offensive Abilities (10 Total)

| # | Name | Type | Damage | Cooldown | Range | Description |
|---|------|------|--------|----------|-------|-------------|
| 1 | **Fireball** | Projectile | 45 | 6s | 30m | Launch a fireball that explodes on impact, dealing AoE damage |
| 2 | **Cutlass Slash** | Melee | 60 | 4s | 5m | Wide arc melee attack, hits all enemies in front |
| 3 | **Cannonball** | Projectile | 70 | 10s | 50m | Slow, heavy projectile with large AoE explosion |
| 4 | **Lightning Strike** | Ground Target | 55 | 8s | 25m | Strike target area after 0.5s delay |
| 5 | **Poison Mackerel** | Thrown | 25+30 DoT | 7s | 20m | Throw a poisonous fish, applies 3s poison |
| 6 | **Frost Bolt** | Projectile | 35 | 5s | 35m | Fast projectile that slows target by 40% for 2s |
| 7 | **Anchor Drop** | Ground Target | 80 | 12s | 15m | Drop anchor at location, stuns 1s on hit |
| 8 | **Kraken's Grasp** | Skillshot | 40 | 8s | 20m | Tentacle grabs and pulls first enemy hit |
| 9 | **Explosive Barrel** | Placed | 90 | 15s | 10m throw | Place barrel, explodes on contact or after 3s |
| 10 | **Chain Lightning** | Target | 30 x3 | 9s | 25m | Hits target, chains to 2 nearby enemies |

#### 4.3.2 Utility Abilities (10 Total)

| # | Name | Type | Effect | Cooldown | Description |
|---|------|------|--------|----------|-------------|
| 1 | **Grappling Hook** | Dash | Move to target location | 8s | Dash 15m to target point |
| 2 | **Barrel Roll** | Dash + Immunity | 0.5s invulnerable | 10s | Roll 8m, immune during roll |
| 3 | **Healing Grog** | Self Heal | +50 HP over 4s | 12s | Drink healing potion, HoT effect |
| 4 | **Smoke Bomb** | Zone | 4s vision block, 20% slow | 14s | 6m radius smoke cloud |
| 5 | **Wind Burst** | Knockback | Pushes enemies 8m | 10s | AoE knockback around player |
| 6 | **Phantom Step** | Teleport | Blink 10m | 12s | Instant teleport, leaves decoy |
| 7 | **Sea Legs** | Buff | +30% speed for 5s | 15s | Movement speed boost |
| 8 | **Parrot Scout** | Reveal | Reveals enemies in 30m for 5s | 20s | Deploy parrot that reveals area |
| 9 | **Siren's Call** | CC | 2s charm, enemies walk toward you | 16s | AoE charm in 10m radius |
| 10 | **Treasure Shield** | Defense | Absorbs 60 damage, 3s duration | 14s | Temporary shield |

### 4.4 Ability Synergies (Combos)

| Combo Name | Abilities | Effect | Notes |
|------------|-----------|--------|-------|
| **Shock and Awe** | Smoke Bomb + Lightning Strike | +25% damage in smoke | Blind then zap |
| **Hook, Line, Sinker** | Grappling Hook + Cutlass Slash | +30% damage on arrival | Gap close combo |
| **Barrel Bomb** | Explosive Barrel + Wind Burst | Barrel knockback doubled | Push barrel into enemy |
| **Frost Fire** | Frost Bolt + Fireball | Explosion +20% larger | Thermal shock |
| **Poison Cure** | Poison Mackerel + Healing Grog | Poison heals you | Thematic healing |
| **Ghost Pirate** | Phantom Step + Smoke Bomb | 1s extra invis after blink | Escape artist |

### 4.5 Ability Balance Philosophy

1. **No Ability is Mandatory:** All builds can win
2. **Counters Exist:** Every strong ability has counterplay
3. **Skill Expression:** Higher skill ceiling abilities have higher reward
4. **Rarity = Power + Rarity:** Epic abilities are stronger but rarer
5. **Cooldown Matters:** Longer CD = more impactful

---

## 5. Progression Systems

### 5.1 In-Match Progression

#### 5.1.1 Leveling
| Level | XP Required | Total XP | HP | Damage Bonus |
|-------|-------------|----------|-----|--------------|
| 1 | 0 | 0 | 100 | +0% |
| 2 | 100 | 100 | 110 | +3% |
| 3 | 150 | 250 | 120 | +6% |
| 4 | 200 | 450 | 130 | +9% |
| 5 | 250 | 700 | 140 | +12% |
| 6 | 300 | 1000 | 150 | +15% |
| 7 | 350 | 1350 | 160 | +18% |
| 8 | 400 | 1750 | 175 | +21% |
| 9 | 450 | 2200 | 190 | +24% |
| 10 | 500 | 2700 | 200 | +30% |

#### 5.1.2 XP Sources
| Source | XP | Notes |
|--------|-----|-------|
| Player Kill | 100 + (10 × victim level) | Higher level = more XP |
| Monster Kill | 20-200 | Based on monster type |
| Chest Opened | 10 | Small but adds up |
| Survival (per 30s) | 5 | Passive gain |
| Assist | 50 | Damage within 5s of kill |

### 5.2 Meta Progression (Account Level)

#### 5.2.1 Account Level
| Level Range | XP/Level | Unlocks |
|-------------|----------|---------|
| 1-10 | 1,000 | Basic features, ranked at 10 |
| 11-30 | 2,000 | Cosmetic rewards |
| 31-50 | 3,000 | Exclusive emotes |
| 51-100 | 5,000 | Prestige borders |
| 100+ | 10,000 | Prestige levels, flex rewards |

#### 5.2.2 Account XP Sources
| Source | XP |
|--------|-----|
| Match Completion | 50-200 (based on placement) |
| Win | +100 bonus |
| Daily Quest | 100-300 |
| Weekly Quest | 500-1000 |
| Achievement | Variable |

### 5.3 Battle Pass

#### 5.3.1 Structure
- **Duration:** 8 weeks per season
- **Tiers:** 60 tiers
- **Tracks:** Free + Premium
- **Price:** 950 Doubloons (~$9.99)

#### 5.3.2 Tier Rewards Distribution
| Tier Range | Free Track | Premium Track |
|------------|------------|---------------|
| 1-10 | Gold, basic items | Outfit piece, Doubloons |
| 11-30 | Gold, common cosmetic | Rare cosmetics, Doubloons |
| 31-50 | Gold, Doubloon drops | Epic cosmetics, emotes |
| 51-60 | Gold, rare cosmetic | Legendary outfit, title |

#### 5.3.3 Battle Pass XP
- **Per Tier:** 10,000 XP
- **Total for Pass:** 600,000 XP
- **Daily Average Needed:** ~10,700 XP (completable with daily play)

### 5.4 Achievements

#### 5.4.1 Achievement Categories
| Category | Examples |
|----------|----------|
| Combat | "100 eliminations", "Win without taking damage" |
| Abilities | "Use Fireball 500 times", "Land 50 Kraken's Grasp" |
| Exploration | "Visit all POIs", "Open 1000 chests" |
| Social | "Win 10 duo matches", "Play 100 games with friends" |
| Collection | "Own 25 outfits", "Complete a Battle Pass" |

#### 5.4.2 Achievement Rewards
- Gold (100-1000)
- Doubloons (rare achievements)
- Exclusive titles
- Profile badges
- Unique cosmetics (very rare achievements)

---

## 6. Monetization Design

### 6.1 Monetization Philosophy

**Core Principles:**
1. **Fair Play First:** No gameplay advantages for purchase
2. **Cosmetic Focus:** Monetize appearance, not power
3. **Earnable Premium:** Some premium currency earnable in-game
4. **Value Perception:** Players feel purchases are worth the cost
5. **Regional Pricing:** Accessible in all markets

### 6.2 Currency System

| Currency | Type | Acquisition | Primary Use |
|----------|------|-------------|-------------|
| **Gold Coins** | Soft | Gameplay, quests | Common cosmetics, name change |
| **Doubloons** | Hard | Purchase, Battle Pass | Premium cosmetics, Battle Pass |

#### 6.2.1 Gold Economy
| Source | Amount |
|--------|--------|
| Match (loss) | 50 |
| Match (top 10) | 100 |
| Match (win) | 200 |
| Daily Quest | 100-300 |
| Weekly Quest | 500-1000 |
| Level Up | 100 |
| First Win of Day | +200 |

**Daily Average Earning:** ~800-1200 Gold

#### 6.2.2 Gold Sinks
| Item | Cost |
|------|------|
| Common Emote | 500 |
| Common Outfit | 1,000 |
| Uncommon Outfit | 2,500 |
| Name Change | 500 |
| Profile Items | 200-1000 |

### 6.3 IAP Products

#### 6.3.1 Doubloon Packs
| Pack | Doubloons | Price | Bonus |
|------|-----------|-------|-------|
| Handful | 100 | $0.99 | - |
| Pouch | 550 | $4.99 | +10% |
| Chest | 1,200 | $9.99 | +20% |
| Treasure | 2,500 | $19.99 | +25% |
| Hoard | 6,500 | $49.99 | +30% |

#### 6.3.2 Special Packs
| Pack | Contents | Price | Limit |
|------|----------|-------|-------|
| Starter Pack | 500 Doubloons + Rare Outfit | $4.99 | 1 per account |
| Battle Pass | Season pass | $9.99 | Per season |
| Battle Pass Bundle | Pass + 10 tier skips | $14.99 | Per season |

### 6.4 Store Design

#### 6.4.1 Store Sections
1. **Featured:** 4-6 rotating items (daily)
2. **Outfits:** Character skins
3. **Emotes:** Dances, gestures
4. **Bundles:** Themed collections
5. **Battle Pass:** Current season

#### 6.4.2 Pricing Tiers
| Rarity | Doubloon Price | USD Equivalent |
|--------|----------------|----------------|
| Uncommon | 200-400 | $2-4 |
| Rare | 500-800 | $5-8 |
| Epic | 1,000-1,500 | $10-15 |
| Legendary | 2,000-2,500 | $20-25 |

### 6.5 Rewarded Ads (Optional)

| Placement | Reward | Daily Limit |
|-----------|--------|-------------|
| Post-match | +50% Gold | 3 |
| Daily bonus | 2x login reward | 1 |
| Free chest | 1 Common chest | 2 |

---

## 7. Technical Requirements

### 7.1 Target Platforms

| Platform | Minimum Version | Target Devices |
|----------|-----------------|----------------|
| iOS | iOS 13+ | iPhone 8 and newer |
| Android | Android 8.0+ | Mid-range 2020+ devices |

### 7.2 Performance Targets

| Metric | Low-End | Mid-Range | High-End |
|--------|---------|-----------|----------|
| Target FPS | 30 | 30-60 | 60 |
| Resolution | 720p | 1080p | Native |
| Draw Calls | <150 | <250 | <400 |
| Memory | <1GB | <1.5GB | <2GB |

### 7.3 Network Requirements

| Metric | Target |
|--------|--------|
| Min Bandwidth | 256 Kbps |
| Target Latency | <150ms |
| Max Acceptable Latency | 300ms |
| Tick Rate | 20Hz |
| Packet Size | <1KB average |

### 7.4 Technology Stack (Recommended)

| Component | Recommendation | Alternative |
|-----------|----------------|-------------|
| Game Engine | Unity 2022 LTS | Unreal Engine 5 |
| Networking | Photon Fusion | Mirror + Custom |
| Backend | PlayFab | Firebase + Custom |
| Analytics | Unity Analytics | GameAnalytics |
| Crash Reporting | Firebase Crashlytics | Sentry |
| Ads | Unity Ads + AdMob | ironSource |

### 7.5 Server Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE OVERVIEW                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CLIENTS ◄──────► LOAD BALANCER ◄──────► GAME SERVERS       │
│                         │                     │              │
│                         ▼                     ▼              │
│                   MATCHMAKING            GAME STATE          │
│                         │                     │              │
│                         ▼                     ▼              │
│                 ┌───────────────────────────────┐            │
│                 │        BACKEND SERVICES        │            │
│                 │  ┌─────┐ ┌─────┐ ┌─────────┐  │            │
│                 │  │Auth │ │Store│ │Analytics│  │            │
│                 │  └─────┘ └─────┘ └─────────┘  │            │
│                 │  ┌─────┐ ┌─────┐ ┌─────────┐  │            │
│                 │  │Save │ │Quest│ │Leaderbd │  │            │
│                 │  └─────┘ └─────┘ └─────────┘  │            │
│                 └───────────────────────────────┘            │
│                              │                               │
│                              ▼                               │
│                         DATABASE                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 7.6 Security Requirements

1. **Server-Authoritative:** All game logic validated server-side
2. **Encryption:** TLS for all communication
3. **Anti-Cheat:** Speed, damage, position validation
4. **Purchase Verification:** Server-side receipt validation
5. **Rate Limiting:** Prevent spam and abuse

---

## 8. Art Direction Summary

### 8.1 Visual Style

**Style Keywords:**
- Stylized / Cartoony
- Vibrant / Colorful
- Whimsical / Playful
- Fantasy / Magical
- Pirate / Nautical

**Reference Styles:**
- Fortnite (proportions, colors)
- Sea of Thieves (pirate theme)
- World of Warcraft (fantasy elements)
- Brawl Stars (mobile readability)

### 8.2 Color Palette

| Use | Colors | Hex Examples |
|-----|--------|--------------|
| **Primary** | Ocean blues, sandy beige | #1E90FF, #F4A460 |
| **Secondary** | Tropical greens, sunset orange | #32CD32, #FF6347 |
| **Accent** | Gold, crimson red | #FFD700, #DC143C |
| **UI** | Parchment, dark wood | #FFF8DC, #8B4513 |

### 8.3 Character Design

- **Proportions:** Slightly exaggerated (larger heads, hands)
- **Silhouettes:** Distinct, readable at distance
- **Customization:** Modular (head, body, accessories)
- **Personality:** Each outfit tells a story

### 8.4 Environment Design

- **Biomes:** Tropical beach, volcanic, jungle, ruins, ocean
- **POIs:** 8-12 named locations with unique themes
- **Props:** Pirate ships, treasure, barrels, palm trees
- **Lighting:** Golden hour feel, warm and inviting

### 8.5 Technical Art Specs

| Asset Type | Triangle Budget | Texture Size |
|------------|-----------------|--------------|
| Player Character | 8,000 | 1024x1024 |
| NPC Monster | 3,000 | 512x512 |
| Environment Prop | 500-2,000 | 256-512 |
| Building | 5,000-10,000 | 1024x1024 |

---

## 9. Audio Design

### 9.1 Music Style

- **Genre:** Orchestral with sea shanty influences
- **Mood:** Adventurous, exciting, slightly comedic
- **Dynamic:** Intensity increases with storm/combat

### 9.2 Music Tracks Needed

| Track | Duration | Mood |
|-------|----------|------|
| Main Menu | 2-3 min loop | Adventurous, welcoming |
| Lobby | 1 min loop | Anticipation, light |
| In-Match Ambient | 3-5 min loop | Subtle tension |
| Combat | 2 min loop | Intense, fast |
| Final Circle | 1 min loop | Maximum intensity |
| Victory | 30 sec | Triumphant |
| Defeat | 15 sec | Encouraging |

### 9.3 Sound Effects Categories

1. **Combat:** Ability sounds, hits, impacts
2. **Movement:** Footsteps, jumps, swimming
3. **UI:** Button clicks, notifications, rewards
4. **Ambient:** Wind, waves, wildlife
5. **Character:** Voice barks (optional), emote sounds

### 9.4 Audio Technical Specs

- **Format:** OGG/MP3 (mobile optimized)
- **Sample Rate:** 44.1kHz
- **Bit Depth:** 16-bit
- **Spatial Audio:** 3D positional for gameplay sounds

---

## 10. User Interface

### 10.1 UI Principles

1. **Clarity Over Style:** Gameplay info always readable
2. **Thumb-Friendly:** All controls reachable
3. **Consistent:** Same patterns across screens
4. **Themed:** Pirate aesthetic without sacrificing usability
5. **Scalable:** Works on phones and tablets

### 10.2 Key Screens

| Screen | Priority | Purpose |
|--------|----------|---------|
| Main Menu | P0 | Hub for all features |
| HUD | P0 | In-game information |
| Inventory | P0 | Manage cosmetics |
| Store | P1 | Purchases |
| Battle Pass | P1 | Season progression |
| Settings | P0 | Player preferences |
| End of Match | P1 | Results and rewards |

### 10.3 HUD Layout

```
┌─────────────────────────────────────────────────────────────┐
│ [MINI]                            [KILL FEED] [ALIVE: 45]  │
│ [MAP]                                                       │
│                                                             │
│                                                             │
│                                                             │
│                    GAMEPLAY AREA                            │
│                                                             │
│                                                             │
│                                                             │
│ [HEALTH BAR]        [STORM TIMER]         [ABILITY 1] [2]  │
│ [XP BAR][LVL]       [JOYSTICK]            [ABILITY 3] [4]  │
│                                           [BASIC ATTACK]    │
└─────────────────────────────────────────────────────────────┘
```

### 10.4 Control Layout

- **Left Side (40%):** Movement joystick
- **Right Side (60%):** Camera control, ability buttons
- **Ability Buttons:** Arc arrangement for thumb reach
- **All Buttons:** Minimum 44pt touch targets

---

## 11. Appendices

### Appendix A: Map POI List

| POI Name | Theme | Loot Tier | Size |
|----------|-------|-----------|------|
| Port Plunder | Pirate town | High | Large |
| Skull Cove | Skeleton lair | High | Medium |
| Shipwreck Bay | Crashed ships | Medium | Medium |
| Treasure Trove | Gold cave | Very High | Small |
| Mermaid Lagoon | Underwater theme | Medium | Medium |
| Volcano Peak | Fiery mountain | High | Large |
| Jungle Temple | Ancient ruins | High | Medium |
| Fisherman's Wharf | Coastal village | Low | Small |

### Appendix B: Milestone Deliverables

| Milestone | GDD Sections Required |
|-----------|----------------------|
| Concept | Sections 1-4 (Overview, Loop, Mechanics, Abilities) |
| Alpha | + Sections 7-8 (Technical, Art) |
| Beta | + Sections 5-6 (Progression, Monetization) |
| Launch | All sections complete and validated |

### Appendix C: Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-30 | Initial GDD creation |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Game Designer | | | |
| Lead Programmer | | | |
| Art Director | | | |
| Producer | | | |

---

**End of Game Design Document**
