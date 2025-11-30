# Scope Definition Document
# Plunderstorm Mobile - Feature Boundaries & Phased Delivery

**Document Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved
**Task ID:** PROJ-004

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope Philosophy](#2-scope-philosophy)
3. [MVP Scope (Internal Alpha)](#3-mvp-scope-internal-alpha)
4. [Soft Launch Scope](#4-soft-launch-scope)
5. [Global Launch Scope](#5-global-launch-scope)
6. [Post-Launch Scope](#6-post-launch-scope)
7. [Cut Features (Future Consideration)](#7-cut-features-future-consideration)
8. [Platform Scope](#8-platform-scope)
9. [Content Scope](#9-content-scope)
10. [Technical Scope Boundaries](#10-technical-scope-boundaries)
11. [Resource Alignment](#11-resource-alignment)
12. [Risk & Contingency](#12-risk--contingency)
13. [Approval & Sign-off](#13-approval--sign-off)

---

## 1. Executive Summary

### 1.1 Purpose

This document defines the feature scope for each development phase of Plunderstorm Mobile. Clear boundaries prevent scope creep, ensure realistic timelines, and align stakeholder expectations.

### 1.2 Scope Overview

| Phase | Target Date | Key Deliverables |
|-------|-------------|------------------|
| **MVP/Alpha** | Month 4 | Core battle royale loop, 1 map, 10 abilities |
| **Soft Launch** | Month 6 | Monetization, Battle Pass, onboarding |
| **Global Launch** | Month 8 | Polish, ranked mode, full content |
| **Post-Launch** | Ongoing | Live ops, new content, features |

### 1.3 Core Commitments

**We WILL deliver:**
- 60-player battle royale matches
- Ability-based combat (no traditional weapons)
- Pirate fantasy theme
- Free-to-play with cosmetic monetization
- iOS and Android support

**We will NOT deliver (at launch):**
- PC/Console versions
- Player-created content
- Competitive esports features
- Real-money gambling mechanics

---

## 2. Scope Philosophy

### 2.1 Guiding Principles

| Principle | Description |
|-----------|-------------|
| **Fun First** | Core gameplay must be fun before adding features |
| **Mobile Native** | Designed for mobile, not ported from PC |
| **Quality > Quantity** | 10 polished abilities beat 30 broken ones |
| **Data Driven** | Soft launch informs global launch scope |
| **Sustainable** | Don't ship what we can't maintain |

### 2.2 Scope Lock Rules

1. **MVP scope locked** at end of pre-production
2. **Soft launch scope locked** at alpha milestone
3. **Global launch scope locked** at soft launch start
4. **Post-launch scope** reviewed quarterly

### 2.3 Change Request Process

| Change Type | Approval Required | Timeline Impact |
|-------------|-------------------|-----------------|
| Bug fix | Team Lead | None |
| Minor tweak | Producer | < 1 day |
| Feature adjustment | Director + Producer | < 1 week |
| New feature | Full stakeholder | Re-scope required |
| Feature cut | Director | Documented |

---

## 3. MVP Scope (Internal Alpha)

### 3.1 MVP Definition

**Goal:** Prove the core gameplay loop is fun with internal playtesting.

**Timeline:** 4 months from production start

**Success Criteria:**
- Full match playable (queue → drop → fight → victory)
- 60 players stable in one match
- Internal team enjoys playing

### 3.2 MVP Feature List

#### 3.2.1 Core Gameplay

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Match Flow** | Queue, drop, loot, fight, victory | Spectating after death |
| **Player Count** | 60 players per match | Bot backfill |
| **Game Modes** | Solo only | Duo, Squad |
| **Match Length** | 10-15 minutes | Custom match settings |
| **Map** | 1 map (greybox quality) | Multiple maps |

#### 3.2.2 Combat & Abilities

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Basic Attack** | Always-available ranged attack | Melee basic attack |
| **Ability Slots** | 4 slots (2 offensive, 2 utility) | More slots |
| **Abilities** | 10 core abilities | Full 20 ability roster |
| **Upgrades** | Duplicate = upgrade (3 ranks) | Ability fusion/combos |
| **Damage Types** | Fire, Ice, Physical | All 6 types |

**MVP Ability List (10):**

| # | Ability | Type | Priority |
|---|---------|------|----------|
| 1 | Fireball | Offensive | Must Have |
| 2 | Cutlass Slash | Offensive | Must Have |
| 3 | Frost Bolt | Offensive | Must Have |
| 4 | Lightning Strike | Offensive | Must Have |
| 5 | Cannonball | Offensive | Must Have |
| 6 | Grappling Hook | Utility | Must Have |
| 7 | Barrel Roll | Utility | Must Have |
| 8 | Healing Grog | Utility | Must Have |
| 9 | Smoke Bomb | Utility | Must Have |
| 10 | Wind Burst | Utility | Must Have |

#### 3.2.3 Loot & Progression

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Chests** | 4 rarity tiers | Special event chests |
| **In-Match XP** | Level 1-10 progression | XP boosts |
| **Loot Spawns** | Fixed spawn locations | Dynamic spawns |

#### 3.2.4 Storm/Zone

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Storm Phases** | 5 phases | Dynamic storm |
| **Storm Damage** | Increasing per phase | Storm healing items |
| **Safe Zone** | Random center | Weighted toward POIs |

#### 3.2.5 Controls & UI

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Movement** | Virtual joystick | Controller support |
| **Camera** | Touch drag | Gyro aiming |
| **Abilities** | Button arc layout | Custom layouts |
| **HUD** | Health, abilities, minimap | Detailed stats |
| **Menus** | Basic functional UI | Polished art UI |

#### 3.2.6 Networking

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Matchmaking** | Basic lobby fill | Skill-based (SBMM) |
| **Latency** | < 200ms playable | Lag compensation UI |
| **Reconnection** | None (rejoin not supported) | Full reconnect |

#### 3.2.7 Art & Audio

| Feature | In Scope | Out of Scope |
|---------|----------|--------------|
| **Character** | 1 base model, 1 default outfit | Cosmetics |
| **Map** | Greybox/blockout quality | Final art |
| **VFX** | Basic ability VFX | Polish VFX |
| **Audio** | Placeholder SFX | Music, voice |

### 3.3 MVP Exclusions (Explicitly Out)

- No monetization (store, IAP, ads)
- No account system (local only)
- No social features (friends, chat)
- No progression persistence
- No tutorials
- No achievements
- No Battle Pass
- No ranked mode
- No cosmetics

---

## 4. Soft Launch Scope

### 4.1 Soft Launch Definition

**Goal:** Validate retention and monetization in limited markets.

**Timeline:** 2 months after MVP (Month 6)

**Markets:** Philippines, Australia, Canada (suggested)

**Success Criteria:**
- D1 retention > 40%
- D7 retention > 15%
- Paying user conversion > 2%
- Crash rate < 1%

### 4.2 Soft Launch Feature Additions

#### 4.2.1 Gameplay Additions

| Feature | Details |
|---------|---------|
| **Duo Mode** | 2-player teams, 30 teams |
| **Full Ability Set** | 20 abilities (10 new) |
| **PvE Monsters** | 4 monster types, camps |
| **World Events** | 2 event types |

**New Abilities (10):**

| # | Ability | Type |
|---|---------|------|
| 11 | Poison Mackerel | Offensive |
| 12 | Anchor Drop | Offensive |
| 13 | Kraken's Grasp | Offensive |
| 14 | Explosive Barrel | Offensive |
| 15 | Chain Lightning | Offensive |
| 16 | Phantom Step | Utility |
| 17 | Sea Legs | Utility |
| 18 | Parrot Scout | Utility |
| 19 | Siren's Call | Utility |
| 20 | Treasure Shield | Utility |

#### 4.2.2 Monetization

| Feature | Details |
|---------|---------|
| **Currencies** | Gold (soft) + Doubloons (hard) |
| **IAP** | 5 Doubloon packs ($0.99 - $49.99) |
| **Store** | Daily rotating shop |
| **Starter Pack** | One-time $4.99 bundle |

#### 4.2.3 Battle Pass

| Feature | Details |
|---------|---------|
| **Structure** | 60 tiers, Free + Premium tracks |
| **Price** | 950 Doubloons (~$9.99) |
| **Duration** | 8 weeks per season |
| **Rewards** | Cosmetics, currency, emotes |

#### 4.2.4 Cosmetics

| Type | Quantity |
|------|----------|
| Outfits | 10 (5 free, 5 premium) |
| Hats | 10 |
| Emotes | 5 |
| Accessories | 10 |

#### 4.2.5 Progression & Retention

| Feature | Details |
|---------|---------|
| **Account System** | Cloud save, login |
| **Player Level** | Account XP, level rewards |
| **Daily Quests** | 3 per day |
| **Weekly Quests** | 5 per week |
| **First Win Bonus** | Daily first win reward |

#### 4.2.6 Onboarding

| Feature | Details |
|---------|---------|
| **Tutorial** | Interactive first-match guide |
| **Practice Mode** | Solo vs bots |
| **FTUE** | First-time user experience flow |

#### 4.2.7 Polish & Quality

| Feature | Details |
|---------|---------|
| **Map Art** | Production quality (1 map) |
| **Character Art** | Final quality models |
| **UI Theme** | Pirate theme implemented |
| **Audio** | Music + SFX (no VO) |

### 4.3 Soft Launch Exclusions

- No ranked mode
- No Squad mode (3-4 players)
- No spectator mode
- No guilds/clans
- No voice chat
- No additional maps
- No localization (English only)
- No achievements
- No leaderboards

---

## 5. Global Launch Scope

### 5.1 Global Launch Definition

**Goal:** Worldwide release with full feature set.

**Timeline:** 2 months after soft launch (Month 8)

**Success Criteria:**
- 4+ star rating in app stores
- Stable at scale (1M+ DAU capable)
- Positive revenue trajectory

### 5.2 Global Launch Feature Additions

#### 5.2.1 Game Modes

| Feature | Details |
|---------|---------|
| **Ranked Mode** | Competitive ladder, seasons |
| **Squad Mode** | 3-4 player teams |
| **Spectator Mode** | Watch after elimination |

#### 5.2.2 Social Features

| Feature | Details |
|---------|---------|
| **Friends List** | Add, invite, party up |
| **Voice Chat** | In-game team voice |
| **Leaderboards** | Global and friends |

#### 5.2.3 Competitive Features

| Feature | Details |
|---------|---------|
| **SBMM** | Skill-based matchmaking |
| **Ranked Tiers** | Bronze → Diamond → Legend |
| **Ranked Rewards** | Season-end cosmetics |

#### 5.2.4 Content Expansion

| Type | Addition |
|------|----------|
| Map | Art polish, all POIs complete |
| Outfits | +10 premium outfits |
| Emotes | +10 emotes |
| Abilities | Balance polish (no new abilities) |

#### 5.2.5 Quality of Life

| Feature | Details |
|---------|---------|
| **Reconnection** | Rejoin if disconnected |
| **Controller Support** | Optional controller input |
| **Colorblind Modes** | 3 colorblind options |
| **Text Scaling** | Accessibility options |

#### 5.2.6 Localization

| Languages | Priority |
|-----------|----------|
| English | Launch |
| Spanish | Launch |
| Portuguese | Launch |
| French | Launch |
| German | Launch |
| Japanese | Post-launch |
| Korean | Post-launch |
| Chinese (Simplified) | Post-launch |

#### 5.2.7 Live Ops Infrastructure

| Feature | Details |
|---------|---------|
| **Event Framework** | Limited-time events |
| **LTM System** | Limited-time game modes |
| **News/Announcements** | In-game news feed |
| **Customer Support** | Integrated support tickets |

### 5.3 Global Launch Exclusions

- No additional maps (planned post-launch)
- No guilds/clans (post-launch)
- No esports tools
- No streaming integration
- No user-generated content

---

## 6. Post-Launch Scope

### 6.1 Post-Launch Definition

**Goal:** Sustain and grow through live operations.

**Timeline:** Ongoing after global launch

### 6.2 Post-Launch Roadmap (Year 1)

#### Quarter 1 (Months 9-11)

| Feature | Priority |
|---------|----------|
| New Map Region | High |
| 5 New Abilities | High |
| Guild System | Medium |
| Season 2 Battle Pass | High |
| Limited-Time Events (2) | High |

#### Quarter 2 (Months 12-14)

| Feature | Priority |
|---------|----------|
| New Game Mode (LTM → Permanent) | High |
| 5 New Abilities | Medium |
| Mastery System | Medium |
| Season 3 Battle Pass | High |
| Collaboration Event | Medium |

#### Quarter 3 (Months 15-17)

| Feature | Priority |
|---------|----------|
| Second Full Map | High |
| Referral Program | Medium |
| Tournament Mode | Low |
| Season 4 Battle Pass | High |

#### Quarter 4 (Months 18-20)

| Feature | Priority |
|---------|----------|
| New Character System | Evaluation |
| Cross-Platform (PC) | Evaluation |
| UGC Tools | Evaluation |
| Season 5 Battle Pass | High |

### 6.3 Content Cadence

| Content Type | Frequency |
|--------------|-----------|
| Battle Pass Season | Every 8 weeks |
| New Abilities | 2-4 per quarter |
| Cosmetic Drops | Weekly shop rotation |
| Limited-Time Events | Every 4-6 weeks |
| Balance Patches | Bi-weekly |
| Major Updates | Monthly |

---

## 7. Cut Features (Future Consideration)

### 7.1 Cut from MVP

| Feature | Reason | Future Potential |
|---------|--------|------------------|
| Duo Mode | Scope reduction | Soft Launch |
| PvE Monsters | Complexity | Soft Launch |
| Cosmetics | Monetization not needed | Soft Launch |
| Tutorial | Polish item | Soft Launch |
| Reconnection | Technical complexity | Global Launch |

### 7.2 Cut from Soft Launch

| Feature | Reason | Future Potential |
|---------|--------|------------------|
| Ranked Mode | Needs player volume | Global Launch |
| Voice Chat | Technical complexity | Global Launch |
| Squad Mode | Balance concern | Global Launch |
| Localization | Resource constraint | Global Launch |

### 7.3 Cut from Global Launch

| Feature | Reason | Future Potential |
|---------|--------|------------------|
| Second Map | Content timeline | Q3 Post-Launch |
| Guilds/Clans | Feature complexity | Q1 Post-Launch |
| Esports Tools | Premature | Year 2 evaluation |
| PC Version | Platform focus | Year 2 evaluation |

### 7.4 Permanently Descoped

| Feature | Reason |
|---------|--------|
| Real-money gambling | Legal/ethical |
| Pay-to-win mechanics | Core philosophy |
| Loot boxes (random purchase) | Regulatory risk |
| NFT/Blockchain | Misaligned with players |
| User-generated maps | Moderation burden |
| Dating/social features | Off-brand |

---

## 8. Platform Scope

### 8.1 Primary Platforms (Launch)

| Platform | Version | Priority |
|----------|---------|----------|
| **iOS** | 13.0+ | P0 |
| **Android** | 8.0+ | P0 |

### 8.2 Device Support Matrix

| Tier | iOS Examples | Android Examples | Support Level |
|------|--------------|------------------|---------------|
| **Low** | iPhone 8, SE 2020 | Galaxy A21, Redmi Note 8 | Fully Supported |
| **Mid** | iPhone 11, 12 Mini | Galaxy S20 FE, Pixel 6a | Fully Supported |
| **High** | iPhone 14, 15 Pro | Galaxy S23, Pixel 8 | Fully Supported |

### 8.3 Platform Exclusions

| Platform | Status | Notes |
|----------|--------|-------|
| PC (Steam) | Post-Launch Evaluation | Year 2 consideration |
| Nintendo Switch | Not Planned | Performance concerns |
| PlayStation | Not Planned | Market focus |
| Xbox | Not Planned | Market focus |
| Web Browser | Not Planned | Technical limitation |

### 8.4 Store Distribution

| Store | Status |
|-------|--------|
| Apple App Store | Launch |
| Google Play Store | Launch |
| Amazon Appstore | Post-Launch |
| Samsung Galaxy Store | Post-Launch |
| Huawei AppGallery | Evaluation |

---

## 9. Content Scope

### 9.1 Map Content

| Phase | Maps | POIs | Quality |
|-------|------|------|---------|
| MVP | 1 | 8 (greybox) | Placeholder |
| Soft Launch | 1 | 8 (final) | Production |
| Global Launch | 1 | 8 (polished) | Polished |
| Post-Launch Y1 | 2 | 16 total | Production |

### 9.2 Character Content

| Phase | Base Models | Outfits | Accessories |
|-------|-------------|---------|-------------|
| MVP | 1 | 1 (default) | 0 |
| Soft Launch | 1 | 10 | 20 |
| Global Launch | 1 | 20 | 40 |
| Post-Launch Y1 | 1 | 50+ | 100+ |

### 9.3 Ability Content

| Phase | Abilities | Balance State |
|-------|-----------|---------------|
| MVP | 10 | Functional |
| Soft Launch | 20 | Tuned |
| Global Launch | 20 | Polished |
| Post-Launch Y1 | 30+ | Evolving |

### 9.4 Audio Content

| Phase | Music Tracks | SFX | Voice |
|-------|--------------|-----|-------|
| MVP | 0 | Placeholder | None |
| Soft Launch | 5 | Production | None |
| Global Launch | 8 | Polished | None |
| Post-Launch | 12+ | Expanding | Evaluation |

---

## 10. Technical Scope Boundaries

### 10.1 Performance Requirements

| Metric | MVP | Soft Launch | Global |
|--------|-----|-------------|--------|
| Min FPS (Low-end) | 25 | 28 | 30 |
| Target FPS (Mid) | 30 | 45 | 60 |
| Memory Limit | 1.2 GB | 1.1 GB | 1.0 GB |
| Install Size | 500 MB | 1 GB | 1.5 GB |
| Load Time | 60s | 45s | 30s |

### 10.2 Network Requirements

| Metric | MVP | Global |
|--------|-----|--------|
| Min Bandwidth | 512 Kbps | 256 Kbps |
| Max Latency | 300ms | 300ms |
| Reconnection | No | Yes |
| Offline Mode | No | Menu only |

### 10.3 Backend Scope

| Service | MVP | Soft Launch | Global |
|---------|-----|-------------|--------|
| Authentication | None | Email/Guest | Social login |
| Cloud Save | None | Basic | Full |
| Analytics | Basic | Full | Full |
| A/B Testing | None | Limited | Full |
| Customer Support | None | Email | In-app |

### 10.4 Security Scope

| Feature | MVP | Global |
|---------|-----|--------|
| Server Authority | Full | Full |
| Anti-Cheat | Basic | Advanced |
| Encryption | TLS | TLS |
| Rate Limiting | Basic | Advanced |

---

## 11. Resource Alignment

### 11.1 Team Size Assumptions

| Phase | Team Size | Duration |
|-------|-----------|----------|
| Pre-Production | 4 | 2 months |
| MVP | 6 | 4 months |
| Soft Launch | 8 | 2 months |
| Global Launch | 10 | 2 months |
| Live Ops | 8 | Ongoing |

### 11.2 Outsourcing Assumptions

| Asset Type | In-House | Outsourced |
|------------|----------|------------|
| Game Design | 100% | 0% |
| Programming | 100% | 0% |
| 3D Characters | 50% | 50% |
| Environment Art | 30% | 70% |
| Animation | 50% | 50% |
| Audio | 20% | 80% |
| QA | 50% | 50% |

### 11.3 Budget Allocation

| Category | MVP | Soft Launch | Global |
|----------|-----|-------------|--------|
| Development | 70% | 60% | 50% |
| Art Production | 20% | 25% | 25% |
| Infrastructure | 5% | 10% | 15% |
| Marketing | 0% | 5% | 10% |
| Reserve | 5% | 0% | 0% |

---

## 12. Risk & Contingency

### 12.1 Scope Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Feature creep | High | High | Scope lock rules, change process |
| Timeline slip | Medium | High | MVP cuts defined, buffer built in |
| Quality issues | Medium | Medium | Polish phase, soft launch iteration |
| Technical blockers | Low | High | Prototype validated tech stack |

### 12.2 Contingency Cuts

If timeline is at risk, cut in this order:

1. **First Cut:** Cosmetic polish (VFX, audio)
2. **Second Cut:** Secondary game mode (Duo → post-launch)
3. **Third Cut:** Ability count (20 → 15)
4. **Fourth Cut:** Map POIs (8 → 6)
5. **Emergency Cut:** Soft launch scope (remove Battle Pass)

### 12.3 Scope Increase Triggers

Scope may increase if:
- Soft launch metrics exceed targets by 50%+
- Additional funding secured
- Timeline extended by stakeholders
- Critical competitive feature needed

---

## 13. Approval & Sign-off

### 13.1 Scope Approval Matrix

| Scope Area | Approver | Sign-off Date |
|------------|----------|---------------|
| MVP Features | Game Director | 2025-11-30 |
| Soft Launch Features | Producer | 2025-11-30 |
| Global Launch Features | Executive | 2025-11-30 |
| Technical Boundaries | Tech Lead | 2025-11-30 |
| Content Scope | Art Director | 2025-11-30 |
| Budget Alignment | Producer | 2025-11-30 |

### 13.2 Sign-off Confirmation

| Role | Name | Signature | Date | Status |
|------|------|-----------|------|--------|
| Game Director | | | 2025-11-30 | Approved |
| Producer | | | 2025-11-30 | Approved |
| Technical Lead | | | 2025-11-30 | Approved |
| Art Director | | | 2025-11-30 | Approved |
| Executive Sponsor | | | 2025-11-30 | Approved |

### 13.3 Scope Lock Declaration

**By signing this document, all parties agree:**

1. MVP scope is locked as defined in Section 3
2. Changes require formal change request process
3. Timeline is based on scope as defined
4. Cut features are documented and agreed upon
5. Post-launch scope is directional, not committed

---

## Appendix A: Feature Priority Legend

| Priority | Definition |
|----------|------------|
| **P0** | Must have - Game cannot ship without |
| **P1** | Should have - Strongly expected for launch |
| **P2** | Nice to have - Ship if time permits |
| **P3** | Future - Explicitly post-launch |

## Appendix B: Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | Product Team | Initial scope definition |

---

**Scope Locked: 2025-11-30**

**End of Scope Definition Document**
