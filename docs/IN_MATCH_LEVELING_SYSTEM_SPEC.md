# In-Match Leveling System Specification

## Overview

This document specifies the within-match leveling system for Plunderstorm Mobile. Players level from 1-10 during each match, gaining stat bonuses that increase their combat effectiveness. XP is earned through kills, PvE, and survival.

## Level Progression

### Level Stat Scaling

| Level | HP Bonus | Damage Bonus | CDR | Total Power |
|-------|----------|--------------|-----|-------------|
| 1 | +0 | +0% | 0% | 100% |
| 2 | +10 | +3% | 0% | 106% |
| 3 | +20 | +6% | 0% | 112% |
| 4 | +30 | +9% | 0% | 118% |
| 5 | +50 | +15% | 0% | 130% |
| 6 | +60 | +18% | 2% | 138% |
| 7 | +70 | +21% | 4% | 146% |
| 8 | +80 | +24% | 6% | 154% |
| 9 | +90 | +27% | 8% | 162% |
| 10 | +100 | +30% | 10% | 170% |

### Power Curve Visualization

```
Power
  ↑
170%│                              ★ (Level 10)
    │                         ○
    │                    ○
    │               ○
    │          ○
130%│     ★ (Level 5)
    │    ○
    │   ○
    │  ○
100%│★ (Level 1)
    └──────────────────────────────→ Time
       0    3    6    9    12   15 min
```

## XP Sources

### Kill XP

```typescript
function calculateKillXP(killer: Player, victim: Player): number {
  const baseXP = 100;
  const levelBonus = victim.level * 20;
  const streakBonus = killer.killStreak > 1 ? killer.killStreak * 10 : 0;

  return baseXP + levelBonus + streakBonus;
}
```

| Victim Level | Base XP | Level Bonus | Total |
|--------------|---------|-------------|-------|
| 1 | 100 | 20 | 120 |
| 3 | 100 | 60 | 160 |
| 5 | 100 | 100 | 200 |
| 7 | 100 | 140 | 240 |
| 10 | 100 | 200 | 300 |

### Assist XP

```typescript
function calculateAssistXP(
  assister: Player,
  killer: Player,
  victim: Player,
  damageDealt: number,
  totalDamage: number
): number {
  const killXP = calculateKillXP(killer, victim);
  const damageRatio = damageDealt / totalDamage;

  // Assists get 50% of kill XP proportional to damage
  return Math.floor(killXP * 0.5 * damageRatio);
}
```

### Monster XP

| Monster Type | Base XP | Level Scaling |
|--------------|---------|---------------|
| Skeleton Pirate | 15 | +2 per player level |
| Sea Crab | 25 | +3 per player level |
| Cursed Sailor | 40 | +5 per player level |
| Mini-Boss | 200 | +20 per player level |

### Activity XP

| Activity | XP Amount | Frequency |
|----------|-----------|-----------|
| Chest opened | 10 | Per chest |
| Supply drop looted | 25 | Per drop |
| Survival | 5 | Per 30 seconds |
| First blood | 50 | First kill of match |
| Ability pickup | 5 | Per new ability |

## XP Requirements

### Per-Level Requirements

| Level | XP Required | Cumulative XP | Avg Time to Reach |
|-------|-------------|---------------|-------------------|
| 1→2 | 100 | 100 | 30s |
| 2→3 | 150 | 250 | 1m |
| 3→4 | 200 | 450 | 2m |
| 4→5 | 300 | 750 | 3m |
| 5→6 | 400 | 1150 | 4.5m |
| 6→7 | 500 | 1650 | 6m |
| 7→8 | 600 | 2250 | 8m |
| 8→9 | 750 | 3000 | 10m |
| 9→10 | 1000 | 4000 | 12m |

### XP Curve Formula

```typescript
function getXPForLevel(level: number): number {
  if (level <= 1) return 0;

  // Progressive curve
  const base = 100;
  const growth = 1.25;

  return Math.floor(base * Math.pow(growth, level - 2) * (level - 1));
}

function getTotalXPForLevel(level: number): number {
  let total = 0;
  for (let i = 2; i <= level; i++) {
    total += getXPForLevel(i);
  }
  return total;
}
```

## Level-Up Mechanics

### Level-Up Event

```typescript
interface LevelUpEvent {
  playerId: string;
  previousLevel: number;
  newLevel: number;
  timestamp: number;
  matchTime: number;
  xpSource: XPSource;
}

function onLevelUp(player: Player, newLevel: number): void {
  // Apply stat bonuses
  const bonuses = LEVEL_BONUSES[newLevel];
  player.maxHP += bonuses.hpBonus;
  player.currentHP += bonuses.hpBonus; // Heal the bonus amount
  player.damageMultiplier = 1 + bonuses.damageBonus;
  player.cooldownReduction = bonuses.cdr;

  // Trigger effects
  playLevelUpVFX(player);
  playLevelUpSFX(player);
  showLevelUpUI(player);

  // Broadcast to all players
  broadcastLevelUp(player.id, newLevel);

  // Analytics
  analytics.track("player_level_up", {
    playerId: player.id,
    newLevel,
    matchTime: getMatchTime(),
    xpSources: player.xpBreakdown
  });
}
```

### Stat Application

```typescript
interface LevelBonuses {
  hpBonus: number;
  damageBonus: number;  // As decimal (0.15 = 15%)
  cdr: number;          // As decimal (0.10 = 10%)
}

const LEVEL_BONUSES: Record<number, LevelBonuses> = {
  1:  { hpBonus: 0,   damageBonus: 0.00, cdr: 0.00 },
  2:  { hpBonus: 10,  damageBonus: 0.03, cdr: 0.00 },
  3:  { hpBonus: 20,  damageBonus: 0.06, cdr: 0.00 },
  4:  { hpBonus: 30,  damageBonus: 0.09, cdr: 0.00 },
  5:  { hpBonus: 50,  damageBonus: 0.15, cdr: 0.00 },
  6:  { hpBonus: 60,  damageBonus: 0.18, cdr: 0.02 },
  7:  { hpBonus: 70,  damageBonus: 0.21, cdr: 0.04 },
  8:  { hpBonus: 80,  damageBonus: 0.24, cdr: 0.06 },
  9:  { hpBonus: 90,  damageBonus: 0.27, cdr: 0.08 },
  10: { hpBonus: 100, damageBonus: 0.30, cdr: 0.10 }
};

function calculateDamage(
  baseDamage: number,
  attackerLevel: number,
  abilityRank: number
): number {
  const levelBonus = LEVEL_BONUSES[attackerLevel].damageBonus;
  const rankBonus = RANK_MODIFIERS[abilityRank].damageMultiplier;

  return Math.round(baseDamage * (1 + levelBonus) * rankBonus);
}
```

## UI Components

### XP Bar (HUD)

```
┌─────────────────────────────────────────────────────────┐
│  LVL 5  ████████████████░░░░░░░░░░  750/1150 XP        │
│         ▲               ▲          ▲                   │
│      Current         Progress    Next Level            │
│       Level           Bar        Requirement           │
└─────────────────────────────────────────────────────────┘
```

### Compact XP Bar (Mobile)

```
┌───────────────────────┐
│ [5] ████████░░░  65%  │
└───────────────────────┘
```

### Level-Up Notification

```
┌─────────────────────────────────────┐
│           ⬆️ LEVEL UP! ⬆️            │
│                                     │
│         ★ LEVEL 5 ★                 │
│                                     │
│     +50 HP  |  +15% Damage         │
│                                     │
│   [Dramatic animation plays]        │
└─────────────────────────────────────┘
```

### XP Gain Floating Text

```typescript
interface XPFloatingText {
  text: string;
  color: Color;
  position: Vector3;
  duration: number;
}

function showXPGain(player: Player, amount: number, source: XPSource): void {
  const colors = {
    'kill': Color.red,
    'assist': Color.orange,
    'monster': Color.green,
    'chest': Color.yellow,
    'survival': Color.white
  };

  spawnFloatingText({
    text: `+${amount} XP`,
    color: colors[source],
    position: player.position + Vector3.up * 2,
    duration: 1.5
  });
}
```

### Player Nameplate with Level

```
┌─────────────────────────┐
│   [5] PlayerName        │
│   ████████████░░░░      │  ← Health bar
└─────────────────────────┘
     ▲
  Level badge
```

## Catch-Up Mechanics

### Underdog XP Bonus

Players significantly behind in levels receive bonus XP:

```typescript
function calculateUnderdogBonus(
  playerLevel: number,
  averageLevel: number
): number {
  const levelDiff = averageLevel - playerLevel;

  if (levelDiff <= 0) return 1.0; // No bonus if at or above average

  // 10% bonus per level behind, max 50%
  return Math.min(1.5, 1.0 + (levelDiff * 0.10));
}

function applyXPGain(player: Player, baseXP: number): void {
  const avgLevel = calculateAverageAlivePlayerLevel();
  const underdogMultiplier = calculateUnderdogBonus(player.level, avgLevel);

  const finalXP = Math.floor(baseXP * underdogMultiplier);
  player.xp += finalXP;

  if (underdogMultiplier > 1.0) {
    showBonusXPIndicator(player, underdogMultiplier);
  }
}
```

### Late-Game XP Scaling

As the match progresses, passive XP increases:

| Match Phase | Survival XP (per 30s) | Monster XP Bonus |
|-------------|----------------------|------------------|
| 0-5 min | 5 | +0% |
| 5-10 min | 8 | +25% |
| 10+ min | 12 | +50% |

### Level Advantage Limits

While levels provide advantages, they're designed to be surmountable:

| Level Difference | Advantage | Counterplay |
|------------------|-----------|-------------|
| 1 level | Slight (~6%) | Skill overcomes |
| 2 levels | Noticeable (~12%) | Good play + abilities |
| 3 levels | Significant (~18%) | Outplay + Rank 3 abilities |
| 5+ levels | Major (~30%) | Avoid, use terrain |

## Server Architecture

### XP State Tracking

```typescript
interface PlayerXPState {
  playerId: string;
  currentXP: number;
  level: number;
  xpToNextLevel: number;
  xpBreakdown: {
    kills: number;
    assists: number;
    monsters: number;
    chests: number;
    survival: number;
    other: number;
  };
  levelUpTimes: number[];  // Match timestamps of level-ups
}
```

### Server Validation

```typescript
class XPManager {
  private playerXP: Map<string, PlayerXPState> = new Map();

  grantXP(playerId: string, amount: number, source: XPSource): void {
    const state = this.playerXP.get(playerId);
    if (!state) return;

    // Validate XP source
    if (!this.validateXPSource(playerId, amount, source)) {
      log.warn(`Invalid XP grant attempt: ${playerId}, ${amount}, ${source}`);
      return;
    }

    // Apply underdog bonus
    const avgLevel = this.getAverageLevel();
    const bonus = calculateUnderdogBonus(state.level, avgLevel);
    const finalAmount = Math.floor(amount * bonus);

    // Update state
    state.currentXP += finalAmount;
    state.xpBreakdown[source] += finalAmount;

    // Check for level up
    while (state.currentXP >= state.xpToNextLevel && state.level < 10) {
      this.levelUp(playerId);
    }

    // Broadcast XP gain
    this.broadcastXPGain(playerId, finalAmount, source);
  }

  private levelUp(playerId: string): void {
    const state = this.playerXP.get(playerId)!;

    state.currentXP -= state.xpToNextLevel;
    state.level++;
    state.xpToNextLevel = getXPForLevel(state.level + 1);
    state.levelUpTimes.push(getMatchTime());

    // Apply stat bonuses server-side
    this.applyLevelBonuses(playerId, state.level);

    // Broadcast level up
    this.broadcastLevelUp(playerId, state.level);
  }
}
```

### Network Sync

```typescript
interface XPSyncPacket {
  playerId: string;
  currentXP: number;
  level: number;
  xpToNextLevel: number;
}

interface XPGainPacket {
  playerId: string;
  amount: number;
  source: XPSource;
  newTotal: number;
  levelUp?: {
    newLevel: number;
    bonuses: LevelBonuses;
  };
}

// Sync frequency: On every XP gain (event-driven)
// Full state sync: Every 5 seconds (drift correction)
```

## Visual and Audio Feedback

### Level-Up VFX

```typescript
interface LevelUpVFX {
  // Self view
  screenFlash: { color: Color.gold, duration: 0.3 };
  particleBurst: { prefab: 'vfx_levelup_burst', position: 'character_center' };
  auraEffect: { prefab: 'vfx_levelup_aura', duration: 2.0 };

  // Third-person view (seen by others)
  pillarLight: { prefab: 'vfx_levelup_pillar', height: 10, duration: 1.5 };
  groundRing: { prefab: 'vfx_levelup_ring', radius: 3, duration: 1.0 };
}
```

### Level-Up SFX

| Event | Sound | Duration |
|-------|-------|----------|
| XP gain | `sfx_xp_tick` | 0.1s |
| Near level (90%+) | `sfx_xp_almost` | 0.2s |
| Level up | `sfx_levelup_fanfare` | 1.5s |
| Max level (10) | `sfx_levelup_max` | 2.0s |

### Level Badge Animation

```typescript
// Animate level number change
function animateLevelBadge(oldLevel: number, newLevel: number): void {
  // Scale up
  tween(badge.scale, 1.5, 0.2);

  // Flash gold
  tween(badge.color, Color.gold, 0.1);

  // Update number
  badge.text = newLevel.toString();

  // Scale back with bounce
  tween(badge.scale, 1.0, 0.3, Ease.BounceOut);

  // Return to normal color
  tween(badge.color, Color.white, 0.3);
}
```

## Balance Considerations

### XP Economy Targets

| Player Type | Expected Level at 10min | XP/min |
|-------------|------------------------|--------|
| Aggressive (5+ kills) | 7-8 | 180-220 |
| Balanced (2-3 kills) | 5-6 | 120-150 |
| Passive (PvE focus) | 4-5 | 80-100 |
| Survival-only | 3-4 | 50-70 |

### Anti-Snowball Measures

1. **Underdog bonus**: Behind players gain up to 50% more XP
2. **Kill XP scaling**: High-level kills give more XP
3. **Death penalty**: None (no XP loss on death)
4. **Level cap**: Hard cap at 10 prevents runaway

### Power Budget Analysis

```
At Level 10 vs Level 1:
- HP: 200 vs 100 (100% more survivability)
- Damage: 130% vs 100% (30% more damage)
- Effective TTK difference: ~35% longer to kill L10

This is significant but skill + positioning can overcome.
A Level 5 with Rank 3 abilities vs Level 10 with Rank 1 is competitive.
```

## Analytics Events

```typescript
// Level-up tracking
analytics.track("level_up", {
  playerId: string,
  newLevel: number,
  matchTime: number,
  xpBreakdown: XPBreakdown,
  underdogBonusApplied: boolean
});

// XP source tracking
analytics.track("xp_gained", {
  playerId: string,
  amount: number,
  source: XPSource,
  matchTime: number
});

// Match-end level summary
analytics.track("match_level_summary", {
  averageLevel: number,
  levelDistribution: number[],  // Count at each level
  maxLevelReached: number,
  firstMaxLevelTime: number
});
```

## Testing Requirements

### Functional Tests

- [ ] XP grants correctly for all sources
- [ ] Level-up triggers at correct XP thresholds
- [ ] Stats apply immediately on level-up
- [ ] Level cap prevents going above 10
- [ ] Underdog bonus calculates correctly

### Balance Tests

- [ ] Average player reaches level 5 by mid-match
- [ ] Max level reachable before match end
- [ ] Level advantages meaningful but not overwhelming
- [ ] Various playstyles can level effectively

### Network Tests

- [ ] XP syncs correctly across clients
- [ ] Level-up effects visible to all players
- [ ] No XP desync after extended matches
- [ ] Reconnect restores correct level/XP

### UI Tests

- [ ] XP bar updates smoothly
- [ ] Level-up animation plays correctly
- [ ] Floating XP text is readable
- [ ] Nameplates show correct levels

## Implementation Phases

### Phase 1: Core System
- XP tracking and level calculation
- Basic stat application
- Server-authoritative validation

### Phase 2: UI Integration
- XP bar and level display
- Floating XP text
- Nameplate integration

### Phase 3: Visual Polish
- Level-up VFX and SFX
- Animation improvements
- UI polish

### Phase 4: Balance
- XP curve tuning
- Catch-up mechanic adjustment
- Stat bonus refinement

## Success Metrics

| Metric | Target |
|--------|--------|
| Average level at match end | 6-8 |
| % players reaching max level | 10-20% |
| Level variance between players | < 3 levels |
| Feedback about level snowballing | < 10% |

## Dependencies

- **GAME-012**: Damage and Health System (stat integration)
- **GAME-015**: NPC Monster System (monster XP)
- **UX-008**: Main HUD Layout (XP bar placement)
