# Ability Upgrade System Specification

## Overview

This document specifies the ability upgrade system for Plunderstorm Mobile, where collecting duplicate abilities increases their power. The system matches the core Plunderstorm design with abilities upgradeable from Rank 1 to Rank 3.

## Upgrade Tier System

### Rank Definitions

| Rank | Name | Visual | Damage/Effect | Cooldown | Special |
|------|------|--------|---------------|----------|---------|
| 1 | Common | White border | Base (100%) | Base (100%) | None |
| 2 | Rare | Blue border | +25% (125%) | -10% (90%) | None |
| 3 | Epic | Purple border | +50% (150%) | -20% (80%) | Bonus effect |

### Rank 3 Bonus Effects by Ability

| Ability | Rank 3 Bonus Effect |
|---------|---------------------|
| Cutlass Slash | Applies 2s bleed (10% max HP) |
| Flintlock Pistol | Pierces through first target |
| Blunderbuss | Knockback increased by 50% |
| Cannon Barrage | Leaves burning ground (3s) |
| Grappling Hook | Stuns target on hit (0.5s) |
| Smoke Bomb | Grants invisibility for 1s after exiting |
| Wind Burst | Launches self in aimed direction |
| Lightning Strike | Chains to 1 nearby enemy (50% damage) |
| Healing Tide | Leaves healing pool (3s, 5 HP/s) |
| Treasure Sense | Reveals enemy positions briefly (1s) |

## Upgrade Flow

### Acquisition Logic

```
Player picks up ability loot:
├── If ability slot is empty:
│   └── Equip ability at Rank 1
├── If same ability already equipped:
│   ├── If current rank < 3:
│   │   └── Upgrade to next rank
│   └── If current rank = 3:
│       └── Convert to small XP bonus (+25 XP)
└── If different ability:
    └── Prompt to swap (existing ability stays at current rank)
```

### State Machine

```
[Not Equipped] --pickup--> [Rank 1]
[Rank 1] --duplicate--> [Rank 2]
[Rank 2] --duplicate--> [Rank 3]
[Rank 3] --duplicate--> [Rank 3] + XP Bonus
```

## Visual Feedback System

### Rank Indicators

#### Ability Icon Display
- **Border Color**: White (R1) → Blue (R2) → Purple (R3)
- **Star Indicators**: 1-3 stars below icon
- **Glow Effect**: None (R1) → Subtle (R2) → Prominent (R3)

```
┌─────────────────┐
│ ╔═════════════╗ │
│ ║             ║ │  Border color indicates rank
│ ║  [ABILITY]  ║ │
│ ║    ICON     ║ │
│ ║             ║ │
│ ╚═════════════╝ │
│     ★ ★ ★       │  Stars show current rank (filled = achieved)
│   COOLDOWN      │
└─────────────────┘
```

### Upgrade Animation Sequence

1. **Pickup Detection** (0ms)
   - Duplicate ability detected
   - Loot item disappears with absorption effect

2. **Ability Button Flash** (0-200ms)
   - Current ability button pulses white
   - Scale up to 120% then back to 100%

3. **Rank Up Burst** (200-500ms)
   - Particle burst in rank color (blue for R2, purple for R3)
   - Star indicator fills in
   - "+RANK UP!" text floats above button

4. **Border Transition** (500-700ms)
   - Border color animates from old to new
   - Glow effect intensifies momentarily

5. **Sound Effect** (0ms)
   - Rank-specific upgrade chime
   - R2: Bright ascending tone
   - R3: Epic fanfare with reverb

### Rank 3 Enhanced VFX

Each ability receives visually distinct effects at Rank 3:

| Ability | Base VFX | Rank 3 VFX Enhancement |
|---------|----------|------------------------|
| Cutlass Slash | White arc | Purple arc with trailing particles |
| Flintlock Pistol | Smoke puff | Electric trail effect |
| Blunderbuss | Spread pellets | Golden pellets with sparkles |
| Cannon Barrage | Cannonball impacts | Larger explosions with fire rings |
| Grappling Hook | Rope trail | Chain with lightning crackle |
| Smoke Bomb | Grey smoke | Purple mystical smoke |
| Wind Burst | Air ripple | Tornado effect |
| Lightning Strike | Single bolt | Branching lightning |
| Healing Tide | Blue wave | Golden wave with sparkles |
| Treasure Sense | Pulse wave | Expanding golden radar effect |

## UI Components

### Ability Tooltip (Expanded)

```
┌────────────────────────────────────────┐
│ ⚔️ CUTLASS SLASH                       │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Rank: ★★☆ (2/3)                        │
│                                        │
│ CURRENT STATS (Rank 2):               │
│ • Damage: 75 (+25% from base)         │
│ • Cooldown: 4.5s (-10% from base)     │
│ • Range: 3m                            │
│                                        │
│ NEXT RANK (Rank 3):                   │
│ • Damage: 90 (+50% from base)         │
│ • Cooldown: 4.0s (-20% from base)     │
│ • BONUS: Applies bleed (10% HP/2s)    │
│                                        │
│ [Find another Cutlass Slash to upgrade]│
└────────────────────────────────────────┘
```

### Rank 3 Tooltip (Max Rank)

```
┌────────────────────────────────────────┐
│ ⚔️ CUTLASS SLASH                       │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Rank: ★★★ MAX                          │
│                                        │
│ CURRENT STATS (Rank 3):               │
│ • Damage: 90 (+50% from base)         │
│ • Cooldown: 4.0s (-20% from base)     │
│ • Range: 3m                            │
│ • BONUS: Applies bleed (10% HP/2s)    │
│                                        │
│ ✓ Maximum rank achieved!               │
│ [Duplicates grant +25 XP]             │
└────────────────────────────────────────┘
```

### HUD Rank Display

During gameplay, the ability buttons show rank at a glance:

```
Rank 1:          Rank 2:          Rank 3:
┌─────────┐     ┌─────────┐     ┌─────────┐
│░░░░░░░░░│     │▒▒▒▒▒▒▒▒▒│     │▓▓▓▓▓▓▓▓▓│
│░ ICON  ░│     │▒ ICON  ▒│     │▓ ICON  ▓│
│░░░░░░░░░│     │▒▒▒▒▒▒▒▒▒│     │▓▓▓▓▓▓▓▓▓│
│    ★    │     │   ★★    │     │  ★★★    │
└─────────┘     └─────────┘     └─────────┘
 (White)         (Blue)          (Purple)
```

## Loot Drop Modifications

### Pre-Upgraded Ability Drops

Higher tier loot sources can drop abilities at higher ranks:

| Loot Source | Rank 1 | Rank 2 | Rank 3 |
|-------------|--------|--------|--------|
| Common Chest | 100% | 0% | 0% |
| Rare Chest | 60% | 35% | 5% |
| Epic Chest | 20% | 50% | 30% |
| Supply Drop | 10% | 40% | 50% |
| Boss Drop | 0% | 30% | 70% |

### Visual Distinction for Dropped Abilities

Loot items on ground show rank visually:

- **Rank 1**: Standard item glow (white)
- **Rank 2**: Blue glow with subtle particles
- **Rank 3**: Purple glow with prominent particle effect

## Network Synchronization

### Upgrade Event Packet

```typescript
interface AbilityUpgradeEvent {
  playerId: string;
  abilitySlot: number;        // 0-3
  abilityId: string;
  previousRank: number;       // 1-2
  newRank: number;            // 2-3
  timestamp: number;
  source: UpgradeSource;      // 'DUPLICATE_PICKUP' | 'CHEST_DROP'
}

enum UpgradeSource {
  DUPLICATE_PICKUP = 'duplicate_pickup',
  CHEST_DROP = 'chest_drop',
  BOSS_DROP = 'boss_drop'
}
```

### Server Validation

Server validates upgrade legitimacy:

1. Player has ability equipped in specified slot
2. Ability matches picked up loot
3. Current rank < 3
4. Loot item exists and is accessible
5. No duplicate upgrade within 100ms (anti-exploit)

### Client Prediction

- Client immediately shows upgrade visuals
- Server confirms within 50ms
- Rollback if server denies (rare edge case)

## Stat Calculation System

### Base Stat Structure

```typescript
interface AbilityStats {
  damage: number;
  cooldown: number;
  range: number;
  effectDuration?: number;
  specialValue?: number;      // Ability-specific
}

interface RankModifiers {
  damageMultiplier: number;   // 1.0, 1.25, 1.50
  cooldownMultiplier: number; // 1.0, 0.9, 0.8
  bonusEffect?: BonusEffect;  // Rank 3 only
}
```

### Calculation Formula

```typescript
function calculateRankStats(
  baseStats: AbilityStats,
  rank: number
): AbilityStats {
  const modifiers = RANK_MODIFIERS[rank];

  return {
    damage: Math.round(baseStats.damage * modifiers.damageMultiplier),
    cooldown: baseStats.cooldown * modifiers.cooldownMultiplier,
    range: baseStats.range,  // Range unchanged by rank
    effectDuration: baseStats.effectDuration,
    bonusEffect: rank === 3 ? getBonusEffect(abilityId) : null
  };
}

const RANK_MODIFIERS: Record<number, RankModifiers> = {
  1: { damageMultiplier: 1.0, cooldownMultiplier: 1.0 },
  2: { damageMultiplier: 1.25, cooldownMultiplier: 0.9 },
  3: { damageMultiplier: 1.5, cooldownMultiplier: 0.8 }
};
```

## Complete Ability Stats Table

### Melee Abilities

| Ability | Stat | Rank 1 | Rank 2 | Rank 3 |
|---------|------|--------|--------|--------|
| Cutlass Slash | Damage | 60 | 75 | 90 |
| | Cooldown | 5s | 4.5s | 4s |
| | Bonus | - | - | 2s Bleed |

### Ranged Abilities

| Ability | Stat | Rank 1 | Rank 2 | Rank 3 |
|---------|------|--------|--------|--------|
| Flintlock Pistol | Damage | 45 | 56 | 68 |
| | Cooldown | 3s | 2.7s | 2.4s |
| | Bonus | - | - | Pierce |
| Blunderbuss | Damage/pellet | 15 | 19 | 23 |
| | Cooldown | 6s | 5.4s | 4.8s |
| | Bonus | - | - | +50% Knockback |
| Cannon Barrage | Damage/hit | 80 | 100 | 120 |
| | Cooldown | 12s | 10.8s | 9.6s |
| | Bonus | - | - | Burning Ground |

### Utility Abilities

| Ability | Stat | Rank 1 | Rank 2 | Rank 3 |
|---------|------|--------|--------|--------|
| Grappling Hook | Damage | 20 | 25 | 30 |
| | Cooldown | 8s | 7.2s | 6.4s |
| | Bonus | - | - | 0.5s Stun |
| Smoke Bomb | Duration | 4s | 5s | 6s |
| | Cooldown | 15s | 13.5s | 12s |
| | Bonus | - | - | 1s Invis after |
| Wind Burst | Knockback | 5m | 6.25m | 7.5m |
| | Cooldown | 10s | 9s | 8s |
| | Bonus | - | - | Self-launch |

### Ultimate Abilities

| Ability | Stat | Rank 1 | Rank 2 | Rank 3 |
|---------|------|--------|--------|--------|
| Lightning Strike | Damage | 120 | 150 | 180 |
| | Cooldown | 20s | 18s | 16s |
| | Bonus | - | - | Chain to 1 target |
| Healing Tide | Heal | 80 | 100 | 120 |
| | Cooldown | 18s | 16.2s | 14.4s |
| | Bonus | - | - | Healing pool |
| Treasure Sense | Duration | 3s | 3.75s | 4.5s |
| | Cooldown | 25s | 22.5s | 20s |
| | Bonus | - | - | Reveal enemies |

## Edge Cases and Error Handling

### Rapid Sequential Upgrades

- Debounce upgrade events (100ms minimum between upgrades)
- Queue system for multiple ability pickups
- Process in order received

### Full Inventory Scenarios

When all slots full and duplicate found:
1. Check if any equipped ability matches
2. If match found → upgrade that ability
3. If no match → standard swap prompt

### Disconnect During Upgrade

- Server persists pre-upgrade state
- On reconnect, server sends authoritative ability states
- Client reconciles any visual discrepancies

### Rank 3 Duplicate Handling

```typescript
function handleRank3Duplicate(player: Player, ability: Ability): void {
  // Grant XP bonus instead of upgrade
  player.grantXP(25);

  // Show feedback
  showFloatingText(player.position, "+25 XP", "gold");
  playSound("xp_bonus");

  // Log for analytics
  analytics.track("rank3_duplicate_collected", {
    playerId: player.id,
    abilityId: ability.id,
    matchTime: getCurrentMatchTime()
  });
}
```

## Balance Considerations

### Power Curve Analysis

| Player State | Effective Power | Notes |
|--------------|-----------------|-------|
| All Rank 1 | 100% | Baseline |
| Mixed R1/R2 | ~115% | Early-mid game |
| All Rank 2 | 125% | Strong but beatable |
| Mixed R2/R3 | ~140% | Late game advantage |
| All Rank 3 | 150% | Maximum power |

### Catch-Up Mechanics

- Late-game supply drops favor Rank 2-3 abilities
- Boss encounters guarantee Rank 3 drops
- Smaller circle = higher rank drop rates

### Anti-Snowball Design

- Rank advantage is significant but not insurmountable
- Skill and positioning can overcome 1-rank disadvantage
- Rank 3 bonus effects are utility-focused, not pure damage

## Audio Design

### Upgrade Sound Effects

| Event | Sound Design |
|-------|--------------|
| Rank 1 → 2 | Ascending chime (C-E-G) |
| Rank 2 → 3 | Epic fanfare with shimmer |
| Rank 3 duplicate | Soft XP gain sound |
| Pre-upgraded drop pickup | Rank sound plays on equip |

### Rank 3 Ability Sounds

Enhanced sound effects for Rank 3 abilities:
- +20% bass/impact
- Added reverb tail
- Unique "power" layer

## Analytics Events

```typescript
// Upgrade tracking
analytics.track("ability_upgraded", {
  abilityId: string,
  fromRank: number,
  toRank: number,
  matchTime: number,
  source: string
});

// Rank 3 achievement
analytics.track("ability_maxed", {
  abilityId: string,
  matchTime: number,
  totalUpgrades: number  // How many abilities player has at R3
});

// Duplicate at max
analytics.track("rank3_duplicate", {
  abilityId: string,
  xpGranted: number
});
```

## Implementation Phases

### Phase 1: Core System
- Upgrade state management
- Basic visual indicators (border, stars)
- Stat modifications

### Phase 2: Visual Polish
- Upgrade animations
- Rank 3 enhanced VFX
- Tooltip improvements

### Phase 3: Balance & Tuning
- Rank 3 bonus effects
- Drop rate adjustments
- Power curve validation

## Testing Requirements

### Functional Tests
- [ ] Upgrade triggers correctly on duplicate pickup
- [ ] Stats calculate correctly at each rank
- [ ] Rank 3 bonus effects apply properly
- [ ] Max rank handles duplicates (XP grant)
- [ ] Pre-upgraded drops work from all sources

### Visual Tests
- [ ] All rank borders display correctly
- [ ] Upgrade animation plays smoothly
- [ ] Rank 3 VFX distinguishable from lower ranks
- [ ] Tooltips show accurate information

### Network Tests
- [ ] Upgrade syncs across all clients
- [ ] Server validates upgrade legitimacy
- [ ] Rollback handles denied upgrades
- [ ] Rapid upgrades don't cause desync

### Performance Tests
- [ ] Upgrade animation maintains 60fps
- [ ] Rank 3 VFX don't cause frame drops
- [ ] Multiple simultaneous upgrades handled

## Success Metrics

| Metric | Target |
|--------|--------|
| Upgrade engagement | 90%+ players upgrade at least one ability |
| Rank 3 achievement | 40%+ players achieve at least one Rank 3 |
| Average rank at match end | 1.8-2.2 across all abilities |
| Power differential complaints | <5% of feedback |

## Dependencies

- **GAME-008**: Core Ability Implementation (ability base system)
- **GAME-004**: Loot Spawn System (drop mechanics)
- **ART-013**: Ability VFX Set (Rank 3 enhanced effects)
- **ART-018**: Ability Icons (rank border assets)
