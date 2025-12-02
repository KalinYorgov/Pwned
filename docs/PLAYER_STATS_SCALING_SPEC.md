# Player Stats and Scaling Specification

## Overview

This document specifies the player stat system for Plunderstorm Mobile, defining core stats that scale with level and can be modified by abilities, items, and future systems. The system is designed for extensibility while maintaining balance through server-authoritative calculations.

## Core Stats

### Primary Stats

| Stat | Base Value | Description | Unit |
|------|------------|-------------|------|
| Health | 100 | Maximum hit points | HP |
| Damage | 1.0 | Damage multiplier | Multiplier |
| Cooldown Reduction | 0.0 | Ability cooldown reduction | Percentage |
| Movement Speed | 6.0 | Base movement rate | m/s |

### Secondary Stats (Derived)

| Stat | Formula | Description |
|------|---------|-------------|
| Effective HP | Health × (1 + Armor/100) | Survivability metric |
| DPS Potential | BaseDamage × Damage × AttackSpeed | Theoretical damage output |
| Ability Uptime | 1 / (1 - CDR) | Effective cooldown reduction |

## Stat Data Model

### Player Stats Structure

```typescript
interface PlayerStats {
  // Base stats (before modifiers)
  base: {
    health: number;
    damage: number;
    cooldownReduction: number;
    movementSpeed: number;
  };

  // Current effective stats (after all modifiers)
  current: {
    maxHealth: number;
    currentHealth: number;
    damageMultiplier: number;
    cooldownReduction: number;
    movementSpeed: number;
  };

  // Stat modifiers from various sources
  modifiers: StatModifierCollection;

  // Temporary buffs/debuffs
  temporaryEffects: TemporaryEffect[];
}

interface StatModifier {
  id: string;
  source: ModifierSource;
  stat: StatType;
  type: ModifierType;
  value: number;
  duration?: number;       // undefined = permanent
  stackId?: string;        // For stacking rules
}

enum ModifierSource {
  LEVEL = 'level',
  ABILITY = 'ability',
  ABILITY_RANK = 'ability_rank',
  BUFF = 'buff',
  DEBUFF = 'debuff',
  ITEM = 'item',           // Future
  PERK = 'perk'            // Future
}

enum ModifierType {
  FLAT = 'flat',           // +10 HP
  PERCENTAGE = 'percentage' // +10% damage
}

enum StatType {
  HEALTH = 'health',
  DAMAGE = 'damage',
  COOLDOWN_REDUCTION = 'cooldownReduction',
  MOVEMENT_SPEED = 'movementSpeed'
}
```

## Stat Calculation

### Calculation Order

Stats are calculated in a specific order to ensure consistency:

```typescript
function calculateStats(player: Player): PlayerStats {
  const stats = getBaseStats();

  // 1. Apply level bonuses (flat)
  applyLevelBonuses(stats, player.level);

  // 2. Apply permanent modifiers (abilities, ranks)
  applyPermanentModifiers(stats, player.modifiers);

  // 3. Apply temporary buffs (flat first)
  applyTemporaryFlat(stats, player.temporaryEffects);

  // 4. Apply temporary buffs (percentage)
  applyTemporaryPercentage(stats, player.temporaryEffects);

  // 5. Apply caps and floors
  applyStatLimits(stats);

  return stats;
}
```

### Stacking Rules

**Additive Stacking (Default):**
All modifiers of the same type stack additively to prevent exponential scaling.

```typescript
// Example: Two +10% damage buffs = +20% total, not +21%
function calculateDamageMultiplier(modifiers: StatModifier[]): number {
  let flatBonus = 0;
  let percentBonus = 0;

  for (const mod of modifiers) {
    if (mod.stat !== StatType.DAMAGE) continue;

    if (mod.type === ModifierType.FLAT) {
      flatBonus += mod.value;
    } else {
      percentBonus += mod.value;
    }
  }

  // Base (1.0) + flat + (base × percent)
  return 1.0 + flatBonus + percentBonus;
}
```

**Non-Stacking (Highest Only):**
Some effects don't stack - only the highest value applies.

```typescript
// Movement speed buffs: only highest applies
function calculateMovementSpeed(
  base: number,
  modifiers: StatModifier[]
): number {
  const speedMods = modifiers.filter(m =>
    m.stat === StatType.MOVEMENT_SPEED &&
    m.source === ModifierSource.BUFF
  );

  // Get highest buff only
  const highestBuff = Math.max(...speedMods.map(m => m.value), 0);

  return base * (1 + highestBuff);
}
```

### Stat Limits

```typescript
const STAT_LIMITS = {
  health: {
    min: 1,
    max: 500,
    softCap: 300  // Reduced effectiveness above this
  },
  damage: {
    min: 0.5,
    max: 3.0,
    softCap: 2.0
  },
  cooldownReduction: {
    min: 0,
    max: 0.50,    // 50% CDR cap
    softCap: 0.40
  },
  movementSpeed: {
    min: 3.0,
    max: 12.0,
    softCap: 9.0
  }
};

function applyStatLimits(stats: PlayerStats): void {
  for (const [stat, limits] of Object.entries(STAT_LIMITS)) {
    const value = stats.current[stat];

    // Hard cap
    stats.current[stat] = Math.max(limits.min, Math.min(limits.max, value));

    // Soft cap (diminishing returns)
    if (value > limits.softCap) {
      const excess = value - limits.softCap;
      const reduced = excess * 0.5; // 50% effectiveness above soft cap
      stats.current[stat] = limits.softCap + reduced;
    }
  }
}
```

## Level-Based Scaling

### Stats per Level

| Level | Health | Damage | CDR | Move Speed |
|-------|--------|--------|-----|------------|
| 1 | 100 | 1.00 | 0% | 6.0 |
| 2 | 110 | 1.03 | 0% | 6.0 |
| 3 | 120 | 1.06 | 0% | 6.0 |
| 4 | 130 | 1.09 | 0% | 6.0 |
| 5 | 150 | 1.15 | 0% | 6.0 |
| 6 | 160 | 1.18 | 2% | 6.0 |
| 7 | 170 | 1.21 | 4% | 6.0 |
| 8 | 180 | 1.24 | 6% | 6.0 |
| 9 | 190 | 1.27 | 8% | 6.0 |
| 10 | 200 | 1.30 | 10% | 6.0 |

### Level Bonus Implementation

```typescript
const LEVEL_BONUSES: Record<number, StatModifier[]> = {
  1: [],
  2: [
    { stat: 'health', type: 'flat', value: 10 },
    { stat: 'damage', type: 'percentage', value: 0.03 }
  ],
  // ... levels 3-9
  10: [
    { stat: 'health', type: 'flat', value: 100 },
    { stat: 'damage', type: 'percentage', value: 0.30 },
    { stat: 'cooldownReduction', type: 'flat', value: 0.10 }
  ]
};

function getLevelBonuses(level: number): StatModifier[] {
  return LEVEL_BONUSES[level] || [];
}
```

## Temporary Effects

### Buff/Debuff System

```typescript
interface TemporaryEffect {
  id: string;
  name: string;
  source: string;           // Ability or item that applied it
  modifiers: StatModifier[];
  duration: number;         // Remaining duration (ms)
  maxDuration: number;      // Original duration
  stackCount: number;       // Current stacks
  maxStacks: number;
  refreshable: boolean;     // Can duration be refreshed
  icon: string;
  isDebuff: boolean;
}

// Example buff: Healing Tide's lingering heal
const HEALING_TIDE_BUFF: TemporaryEffect = {
  id: 'healing_tide_regen',
  name: 'Healing Waters',
  source: 'ability_healing_tide',
  modifiers: [
    { stat: 'health', type: 'flat', value: 5 } // Regen component
  ],
  duration: 3000,
  maxDuration: 3000,
  stackCount: 1,
  maxStacks: 1,
  refreshable: true,
  icon: 'icon_buff_heal',
  isDebuff: false
};

// Example debuff: Slow from Smoke Bomb
const SMOKE_SLOW_DEBUFF: TemporaryEffect = {
  id: 'smoke_bomb_slow',
  name: 'Obscured',
  source: 'ability_smoke_bomb',
  modifiers: [
    { stat: 'movementSpeed', type: 'percentage', value: -0.50 }
  ],
  duration: 3000,
  maxDuration: 3000,
  stackCount: 1,
  maxStacks: 1,
  refreshable: true,
  icon: 'icon_debuff_slow',
  isDebuff: true
};
```

### Effect Processing

```typescript
class TemporaryEffectManager {
  private effects: Map<string, TemporaryEffect[]> = new Map();

  applyEffect(playerId: string, effect: TemporaryEffect): void {
    const playerEffects = this.effects.get(playerId) || [];
    const existing = playerEffects.find(e => e.id === effect.id);

    if (existing) {
      if (effect.refreshable) {
        // Refresh duration
        existing.duration = effect.maxDuration;
      }
      if (existing.stackCount < existing.maxStacks) {
        // Add stack
        existing.stackCount++;
        this.recalculateModifiers(existing);
      }
    } else {
      // New effect
      playerEffects.push({ ...effect });
    }

    this.effects.set(playerId, playerEffects);
    this.notifyStatsChanged(playerId);
  }

  update(deltaTime: number): void {
    for (const [playerId, effects] of this.effects) {
      let changed = false;

      for (let i = effects.length - 1; i >= 0; i--) {
        effects[i].duration -= deltaTime;

        if (effects[i].duration <= 0) {
          effects.splice(i, 1);
          changed = true;
        }
      }

      if (changed) {
        this.notifyStatsChanged(playerId);
      }
    }
  }
}
```

## UI Components

### Stats Panel (Pause Menu)

```
┌─────────────────────────────────────────────┐
│             PLAYER STATS                    │
├─────────────────────────────────────────────┤
│                                             │
│  ❤️ Health        180/180  (+80 from level) │
│  ⚔️ Damage        +24%     (Level 8)        │
│  ⏱️ Cooldown      -6%      (Level 8)        │
│  🏃 Move Speed    6.0 m/s  (Base)           │
│                                             │
├─────────────────────────────────────────────┤
│  ACTIVE EFFECTS                             │
│  ┌─────────────────────────────────────┐   │
│  │ 🔵 Healing Waters    2.3s remaining │   │
│  │    +5 HP/s regeneration             │   │
│  └─────────────────────────────────────┘   │
│                                             │
└─────────────────────────────────────────────┘
```

### Buff/Debuff Bar (HUD)

```
┌─────────────────────────────────────┐
│  [🔵2.3s] [🔴1.5s] [🟢5.0s]         │  ← Active effects
│   Heal    Slow     Speed            │
└─────────────────────────────────────┘
```

### Stat Change Indicators

```typescript
// Show stat changes on level-up or buff application
function showStatChange(
  player: Player,
  stat: StatType,
  oldValue: number,
  newValue: number
): void {
  const delta = newValue - oldValue;
  const color = delta > 0 ? Color.green : Color.red;
  const sign = delta > 0 ? '+' : '';

  const text = formatStatChange(stat, delta);
  // Example: "+10 HP" or "-50% Speed"

  spawnFloatingText({
    text: `${sign}${text}`,
    color,
    position: player.position + Vector3.up * 2.5,
    duration: 1.5
  });
}
```

## Server Authority

### Stat Validation

```typescript
class ServerStatsManager {
  private playerStats: Map<string, PlayerStats> = new Map();

  validateStatModification(
    playerId: string,
    modifier: StatModifier
  ): boolean {
    // Validate source
    if (!this.isValidSource(playerId, modifier.source)) {
      log.warn(`Invalid modifier source: ${modifier.source}`);
      return false;
    }

    // Validate value ranges
    if (!this.isValidValue(modifier)) {
      log.warn(`Invalid modifier value: ${modifier.value}`);
      return false;
    }

    // Check for duplicate application
    if (this.isDuplicate(playerId, modifier)) {
      log.warn(`Duplicate modifier: ${modifier.id}`);
      return false;
    }

    return true;
  }

  applyDamage(
    attackerId: string,
    targetId: string,
    baseDamage: number
  ): DamageResult {
    const attackerStats = this.playerStats.get(attackerId);
    const targetStats = this.playerStats.get(targetId);

    // Server calculates actual damage
    const finalDamage = Math.round(
      baseDamage * attackerStats.current.damageMultiplier
    );

    // Apply to target
    targetStats.current.currentHealth -= finalDamage;

    return {
      damage: finalDamage,
      targetHealthRemaining: targetStats.current.currentHealth,
      killed: targetStats.current.currentHealth <= 0
    };
  }
}
```

### Network Sync

```typescript
interface StatsSyncPacket {
  playerId: string;
  stats: {
    maxHealth: number;
    currentHealth: number;
    damageMultiplier: number;
    cooldownReduction: number;
    movementSpeed: number;
  };
  activeEffects: {
    id: string;
    icon: string;
    duration: number;
    isDebuff: boolean;
  }[];
  timestamp: number;
}

// Sync on:
// - Level up
// - Buff/debuff applied or expired
// - Health change
// Full sync every 2 seconds for drift correction
```

## Extensibility

### Future Stat Types

The system is designed to easily add new stats:

```typescript
// Example: Adding Armor stat
interface ExtendedStats extends PlayerStats {
  armor: number;
  criticalChance: number;
  criticalDamage: number;
  lifeSteal: number;
}

// Register new stat
StatRegistry.register({
  name: 'armor',
  baseValue: 0,
  limits: { min: 0, max: 100, softCap: 50 },
  stackingRule: 'additive',
  displayFormat: '{value}',
  icon: 'icon_stat_armor'
});
```

### Modifier Hooks

```typescript
// Allow systems to hook into stat calculations
interface StatModifierHook {
  priority: number;
  onCalculate: (stat: StatType, value: number, player: Player) => number;
}

// Example: Item system hook
const ITEM_STAT_HOOK: StatModifierHook = {
  priority: 100, // After level bonuses
  onCalculate: (stat, value, player) => {
    const itemBonus = player.equipment.getStatBonus(stat);
    return value + itemBonus;
  }
};
```

## Performance Considerations

### Caching

```typescript
class StatsCache {
  private cache: Map<string, CachedStats> = new Map();
  private dirty: Set<string> = new Set();

  getStats(playerId: string): PlayerStats {
    if (this.dirty.has(playerId)) {
      this.recalculate(playerId);
      this.dirty.delete(playerId);
    }
    return this.cache.get(playerId)!.stats;
  }

  markDirty(playerId: string): void {
    this.dirty.add(playerId);
  }

  // Batch recalculation for efficiency
  recalculateAll(): void {
    for (const playerId of this.dirty) {
      this.recalculate(playerId);
    }
    this.dirty.clear();
  }
}
```

### Update Frequency

| Event | Recalculation |
|-------|---------------|
| Level up | Immediate |
| Buff applied | Immediate |
| Buff expired | On next tick |
| Periodic sync | Every 2 seconds |

## Testing Requirements

### Functional Tests

- [ ] Base stats initialize correctly
- [ ] Level bonuses apply correctly
- [ ] Temporary effects apply and expire
- [ ] Stat caps enforce properly
- [ ] Stacking rules work as designed
- [ ] Server authority prevents exploits

### Edge Case Tests

- [ ] Negative modifiers don't go below minimum
- [ ] Overflow protection on high values
- [ ] Multiple simultaneous buffs/debuffs
- [ ] Rapid application and removal
- [ ] Reconnect restores stats correctly

### Balance Tests

- [ ] Level 10 vs Level 1 power difference
- [ ] Buff impact on combat outcomes
- [ ] CDR cap prevents ability spam

## Success Metrics

| Metric | Target |
|--------|--------|
| Stat calculation time | < 0.1ms per player |
| Sync accuracy | 100% match client/server |
| Exploit reports | 0 stat manipulation exploits |

## Dependencies

- **GAME-018**: In-Match Leveling System (level bonuses)
- **GAME-012**: Damage and Health System (damage calculation)
- **GAME-014**: Crowd Control System (debuff modifiers)
