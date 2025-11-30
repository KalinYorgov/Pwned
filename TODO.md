# Plunderstorm Mobile - Master TODO

A comprehensive task breakdown for building a mobile battle royale game inspired by Blizzard's Plunderstorm mode.

## Project Overview

**Genre:** Mobile Battle Royale with RPG/Ability-based Combat
**Theme:** Pirate Fantasy
**Target Platforms:** iOS & Android
**Match Size:** 30-60 players
**Match Duration:** 10-15 minutes

---

## TODO File Index

| File | Domain | Description |
|------|--------|-------------|
| [GAMEPLAY_TODO.md](./GAMEPLAY_TODO.md) | Core Gameplay | Battle royale mechanics, abilities, combat, PvE elements |
| [MOBILE_UX_TODO.md](./MOBILE_UX_TODO.md) | Mobile UX/UI | Touch controls, interface design, mobile optimization |
| [BACKEND_TODO.md](./BACKEND_TODO.md) | Backend/Networking | Multiplayer infrastructure, servers, matchmaking |
| [ART_TODO.md](./ART_TODO.md) | Art & Assets | Visual style, characters, environments, VFX, audio |
| [MONETIZATION_TODO.md](./MONETIZATION_TODO.md) | Monetization | IAP, Battle Pass, currencies, economy |
| [LIVEOPS_TODO.md](./LIVEOPS_TODO.md) | Live Operations | Retention, daily quests, events, seasons |
| [PROJECT_MILESTONES_TODO.md](./PROJECT_MILESTONES_TODO.md) | Project Management | Development phases, milestones, team planning |

---

## Quick Reference: Task Priority Legend

| Priority | Meaning |
|----------|---------|
| **P0** | Critical - Must have for MVP |
| **P1** | High - Required for soft launch |
| **P2** | Medium - Required for global launch |
| **P3** | Low - Post-launch enhancement |

---

## Core Pillars

### 1. Fast, Skill-Driven Matches
- 10-15 minute matches optimized for mobile sessions
- Shrinking storm zone forcing player encounters
- Skill-based combat with abilities (no traditional weapons/gear)

### 2. Ability-Based Gameplay
- No fixed classes - players collect abilities during match
- 4 ability slots (2 offensive, 2 utility)
- Ability upgrades through duplicate collection
- Rarity tiers: Common, Uncommon, Rare, Epic

### 3. PvE Integration
- NPC monster camps for XP farming
- Alternative progression path for different playstyles
- In-match leveling (1-10) for power scaling

### 4. Pirate Fantasy Theme
- Cohesive swashbuckling aesthetic
- Treasure chests, islands, shipwrecks
- Whimsical abilities (barrel roll, poisonous mackerel)

### 5. Mobile-First Design
- Intuitive touch controls
- Short session optimization
- Performance across device tiers
- Quick matchmaking with bot fill

### 6. Rewarding Progression
- Battle Pass system
- Cosmetic unlocks
- Daily/weekly quests
- Seasonal content updates

---

## High-Level Feature Breakdown

### MVP Features (P0)
- [ ] Core battle royale loop (drop, loot, fight, survive)
- [ ] Basic ability system (10 abilities minimum)
- [ ] Touch control scheme
- [ ] Single map
- [ ] Solo game mode
- [ ] Basic matchmaking
- [ ] Player accounts and authentication
- [ ] Essential UI (HUD, menus)

### Soft Launch Features (P1)
- [ ] Full ability roster (20+ abilities)
- [ ] Ability upgrade system
- [ ] Duo game mode
- [ ] PvE monster camps
- [ ] In-match leveling
- [ ] Basic monetization (starter packs, cosmetics)
- [ ] Daily quests
- [ ] Tutorial/onboarding
- [ ] Bot backfill

### Global Launch Features (P2)
- [ ] Battle Pass system
- [ ] Premium currency and store
- [ ] Multiple maps/variants
- [ ] Ranked mode
- [ ] Friends and parties
- [ ] Spectator mode
- [ ] Achievement system
- [ ] Push notifications

### Post-Launch Features (P3)
- [ ] Guilds/Crews system
- [ ] Tournaments
- [ ] New game modes
- [ ] Cross-platform play
- [ ] Seasonal events
- [ ] Character/hero variations

---

## Success Metrics

### Engagement KPIs
- Day 1 Retention: Target 40%+
- Day 7 Retention: Target 20%+
- Day 30 Retention: Target 10%+
- Average Session Length: 12-20 minutes
- Sessions per Day: 2-3

### Monetization KPIs
- Conversion Rate: 2-5%
- ARPDAU: $0.05-0.15
- Battle Pass Purchase Rate: 10-15% of active users

### Performance KPIs
- Crash Rate: <1%
- Match Completion Rate: >85%
- Matchmaking Time: <30 seconds
- Frame Rate: 30 FPS (low-end), 60 FPS (high-end)

---

## Technology Stack Recommendations

| Component | Recommended | Alternative |
|-----------|-------------|-------------|
| Game Engine | Unity | Unreal Engine 5 |
| Networking | Photon Fusion | Mirror (self-hosted) |
| Backend Services | PlayFab | Firebase + Custom |
| Analytics | Unity Analytics | GameAnalytics |
| Ads | Unity Ads + AdMob | ironSource |
| Auth | PlayFab Auth | Firebase Auth |
| Cloud Save | PlayFab | Firebase Firestore |

---

## Document Maintenance

- **Created:** 2025-11-30
- **Last Updated:** 2025-11-30
- **Version:** 1.0

Each TODO file follows this structure per task:
- Task ID and Title
- Priority Level
- Detailed Description
- Acceptance Criteria (bulleted list)
- Definition of Done
- Dependencies (if any)
- Estimated Complexity (S/M/L/XL)
