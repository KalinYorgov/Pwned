# Damage and Health System Specification

## Document Information
- **Task ID:** GAME-012
- **Priority:** P0
- **Complexity:** M
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the health and damage calculation system for Plunderstorm Mobile. The system handles player health pools, damage application, scaling with level, and all combat feedback.

### 1.2 Scope
- Health pool configuration and scaling
- Damage calculation formulas
- Critical hit system
- Health UI display
- Combat feedback (damage numbers, effects)
- Health regeneration rules
- Server-authoritative validation

### 1.3 Dependencies
- GAME-005: Player Elimination System
- BACK-003: Game Server Infrastructure

---

## 2. Health System

### 2.1 Base Health Configuration

| Level | Base Health | Max Health | HP per Level |
|-------|-------------|------------|--------------|
| 1 | 100 | 100 | - |
| 2 | 110 | 110 | +10 |
| 3 | 120 | 120 | +10 |
| 4 | 130 | 130 | +10 |
| 5 | 140 | 140 | +10 |
| 6 | 150 | 150 | +10 |
| 7 | 160 | 160 | +10 |
| 8 | 170 | 170 | +10 |
| 9 | 180 | 180 | +10 |
| 10 | 200 | 200 | +20 (bonus) |

### 2.2 Health Implementation

```javascript
class HealthSystem {
    static BASE_HEALTH = 100;
    static HP_PER_LEVEL = 10;
    static LEVEL_10_BONUS = 10;  // Extra HP at max level

    constructor(player) {
        this.player = player;
        this.maxHealth = this.calculateMaxHealth();
        this.currentHealth = this.maxHealth;
        this.lastDamageTime = 0;
        this.lastDamageSource = null;
    }

    calculateMaxHealth() {
        const level = this.player.level;
        let health = BASE_HEALTH + (level - 1) * HP_PER_LEVEL;

        // Level 10 bonus
        if (level >= 10) {
            health += LEVEL_10_BONUS;
        }

        return health;
    }

    onLevelUp() {
        const oldMax = this.maxHealth;
        const newMax = this.calculateMaxHealth();
        const healthGained = newMax - oldMax;

        this.maxHealth = newMax;
        this.currentHealth += healthGained; // Gain HP on level up

        this.showLevelUpHealth(healthGained);
    }
}
```

### 2.3 Health States

```
┌─────────────────────────────────────────────────────────────────┐
│                    HEALTH STATE DIAGRAM                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                               │
│  │  FULL HEALTH │  100% HP                                      │
│  │              │                                               │
│  └──────┬───────┘                                               │
│         │ [Take damage]                                          │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │   DAMAGED    │  100% > HP > 25%                              │
│  │              │  Normal gameplay                               │
│  └──────┬───────┘                                               │
│         │ [HP drops below 25%]                                   │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │  LOW HEALTH  │  25% > HP > 0%                                │
│  │  ⚠️ WARNING  │  Red vignette, heartbeat SFX                  │
│  └──────┬───────┘                                               │
│         │ [HP reaches 0]                                         │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │    DEAD      │  HP = 0                                       │
│  │  ☠️ ELIMINATED│  Triggers elimination flow                    │
│  └──────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Damage System

### 3.1 Damage Calculation Formula

```javascript
class DamageCalculator {
    calculateDamage(baseDamage, attacker, defender, damageType) {
        let damage = baseDamage;

        // 1. Apply attacker's damage multiplier (from level/buffs)
        damage *= attacker.getDamageMultiplier();

        // 2. Apply critical hit
        const critResult = this.calculateCrit(attacker);
        damage *= critResult.multiplier;

        // 3. Apply defender's damage reduction (if any)
        damage *= (1 - defender.getDamageReduction());

        // 4. Apply damage type modifiers (future: elemental system)
        damage *= this.getDamageTypeModifier(damageType, defender);

        // 5. Apply random variance (optional, ±5%)
        damage *= randomRange(0.95, 1.05);

        // 6. Round to integer
        damage = Math.round(damage);

        // 7. Minimum damage of 1
        damage = Math.max(1, damage);

        return {
            finalDamage: damage,
            isCrit: critResult.isCrit,
            damageType: damageType
        };
    }
}
```

### 3.2 Damage Multipliers by Level

| Level | Damage Multiplier |
|-------|-------------------|
| 1 | 100% |
| 2 | 103% |
| 3 | 106% |
| 4 | 109% |
| 5 | 115% |
| 6 | 118% |
| 7 | 121% |
| 8 | 124% |
| 9 | 127% |
| 10 | 130% |

### 3.3 Damage Types

| Type | Description | Visual Color |
|------|-------------|--------------|
| PHYSICAL | Weapons, melee | White |
| FIRE | Flame abilities | Orange |
| LIGHTNING | Electric abilities | Blue |
| POISON | DoT abilities | Green |
| WIND | Knockback abilities | Cyan |
| BASIC | Basic attack | White |
| STORM | Zone damage | Purple |

---

## 4. Critical Hit System

### 4.1 Critical Hit Rules

| Property | Value |
|----------|-------|
| Base Crit Chance | 10% |
| Crit Damage Multiplier | 1.5x |
| Headshot Multiplier | 2.0x (if implemented) |

### 4.2 Critical Hit Implementation

```javascript
class CriticalHitSystem {
    static BASE_CRIT_CHANCE = 0.10;   // 10%
    static CRIT_MULTIPLIER = 1.5;     // 50% bonus damage

    calculateCrit(attacker) {
        const critChance = this.getCritChance(attacker);
        const roll = Math.random();

        if (roll < critChance) {
            return {
                isCrit: true,
                multiplier: CRIT_MULTIPLIER
            };
        }

        return {
            isCrit: false,
            multiplier: 1.0
        };
    }

    getCritChance(attacker) {
        let chance = BASE_CRIT_CHANCE;

        // Add any crit chance bonuses (future: perks, items)
        chance += attacker.getBonusCritChance();

        // Cap at 50%
        return Math.min(chance, 0.5);
    }
}
```

### 4.3 Crit Visual Feedback

| Element | Normal Hit | Critical Hit |
|---------|------------|--------------|
| Damage Number | White text | Yellow text + larger |
| Sound | Normal impact | Enhanced impact + crit sound |
| Screen Effect | None | Brief screen flash |
| Particle | Standard | Burst effect |

---

## 5. Health UI Display

### 5.1 Health Bar Design

```
┌─────────────────────────────────────────────────────────────────┐
│                    HEALTH BAR DISPLAY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Above Player (Enemy View):                                      │
│  ┌────────────────────────────────────────┐                     │
│  │ ████████████████████░░░░░░░░░░░░░░░░░░│  75 HP / 100 HP     │
│  └────────────────────────────────────────┘                     │
│      PlayerName [Lv.5]                                           │
│                                                                  │
│  Self Health (Bottom of Screen):                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ❤️ ████████████████████████████░░░░░░░░░░░░░░  135/180     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Low Health State (< 25%):                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 💔 ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  22/100 ⚠️       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                 (pulsing red bar)                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Health Bar Configuration

```javascript
class HealthBarUI {
    static CONFIG = {
        // Colors
        fullColor: '#44FF44',      // Green
        midColor: '#FFFF44',       // Yellow (50% HP)
        lowColor: '#FF4444',       // Red (25% HP)
        criticalColor: '#FF0000',  // Flashing red (10% HP)

        // Animation
        damageFlashDuration: 0.1,  // Seconds
        healFlashDuration: 0.2,
        smoothingSpeed: 5,         // HP bar lerp speed

        // Display
        showNumbersAboveEnemy: false,
        showNumbersOnSelf: true,
        showLevelIndicator: true
    };

    getBarColor(healthPercent) {
        if (healthPercent <= 0.10) return CONFIG.criticalColor;
        if (healthPercent <= 0.25) return CONFIG.lowColor;
        if (healthPercent <= 0.50) return CONFIG.midColor;
        return CONFIG.fullColor;
    }
}
```

### 5.3 Enemy Health Visibility

| Condition | Health Bar Shown |
|-----------|------------------|
| Enemy damaged by you | Yes (10 seconds) |
| Enemy in combat nearby | Yes |
| Enemy undamaged/far | No |
| Teammate | Always |
| Self | Always (HUD) |

---

## 6. Damage Numbers (Floating Combat Text)

### 6.1 Damage Number Display

```
┌─────────────────────────────────────────────────────────────────┐
│                    DAMAGE NUMBERS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Standard Damage:       Critical Hit:        Healing:            │
│                                                                  │
│       -45                  -68!                 +25              │
│        ↑                    ↑                    ↑               │
│       (white)          (yellow, large)       (green)             │
│                                                                  │
│  Animation:                                                      │
│  - Pop up above target                                           │
│  - Float upward (0.5s)                                           │
│  - Fade out                                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Damage Number Implementation

```javascript
class FloatingCombatText {
    static FLOAT_SPEED = 50;      // Pixels per second
    static DURATION = 0.8;        // Seconds
    static FADE_START = 0.5;      // Start fading at 50% duration

    createDamageNumber(position, damage, type) {
        const text = {
            value: damage,
            position: worldToScreen(position),
            color: this.getColor(type),
            size: this.getSize(type),
            startTime: Time.now()
        };

        this.activeNumbers.push(text);
    }

    getColor(type) {
        switch (type) {
            case 'NORMAL': return '#FFFFFF';
            case 'CRIT': return '#FFD700';
            case 'HEAL': return '#44FF44';
            case 'POISON': return '#44FF44';
            case 'STORM': return '#9944FF';
            default: return '#FFFFFF';
        }
    }

    getSize(type) {
        switch (type) {
            case 'CRIT': return 1.5;  // 50% larger
            case 'HEAL': return 1.2;
            default: return 1.0;
        }
    }

    update(deltaTime) {
        for (const text of this.activeNumbers) {
            // Float upward
            text.position.y -= FLOAT_SPEED * deltaTime;

            // Calculate alpha
            const age = Time.now() - text.startTime;
            if (age > FADE_START * DURATION) {
                const fadeProgress = (age - FADE_START * DURATION) /
                                   ((1 - FADE_START) * DURATION);
                text.alpha = 1 - fadeProgress;
            }

            // Remove when expired
            if (age > DURATION) {
                this.removeNumber(text);
            }
        }
    }
}
```

---

## 7. Low Health Warning

### 7.1 Warning Effects

| Health % | Effect |
|----------|--------|
| 25% | Health bar turns red |
| 20% | Red vignette starts |
| 15% | Heartbeat sound begins |
| 10% | Intense pulsing, louder heartbeat |
| 5% | Screen edges heavily red |

### 7.2 Warning Implementation

```javascript
class LowHealthWarning {
    static VIGNETTE_START = 0.20;  // 20% HP
    static HEARTBEAT_START = 0.15; // 15% HP

    update(healthPercent) {
        // Vignette effect
        if (healthPercent <= VIGNETTE_START) {
            const intensity = 1 - (healthPercent / VIGNETTE_START);
            this.setVignetteIntensity(intensity * 0.5);
        } else {
            this.setVignetteIntensity(0);
        }

        // Heartbeat audio
        if (healthPercent <= HEARTBEAT_START) {
            const urgency = 1 - (healthPercent / HEARTBEAT_START);
            this.playHeartbeat(urgency);
        } else {
            this.stopHeartbeat();
        }
    }

    playHeartbeat(urgency) {
        // Faster and louder as HP drops
        const interval = lerp(1.2, 0.4, urgency); // 1.2s to 0.4s
        const volume = lerp(0.3, 0.8, urgency);

        if (!this.heartbeatPlaying ||
            Time.now() - this.lastHeartbeat > interval) {
            Audio.play('heartbeat', { volume });
            this.lastHeartbeat = Time.now();
            this.heartbeatPlaying = true;
        }
    }
}
```

---

## 8. Health Regeneration

### 8.1 Regeneration Rules

| Rule | Value | Notes |
|------|-------|-------|
| Passive Regen | None | No natural health recovery |
| Out of Combat | None | Keeps urgency high |
| Ability Healing | Yes | Healing Grog, etc. |
| NPC Drop Healing | None | XP only from orbs |

### 8.2 Healing Sources

| Source | Amount | Notes |
|--------|--------|-------|
| Healing Grog (Rank 1) | 40 over 4s | Ability |
| Healing Grog (Rank 2) | 50 over 4s | Ability |
| Healing Grog (Rank 3) | 60 over 4s | Ability |
| Level Up | +10-20 HP | Based on HP gained |
| Cutlass Slash (Rank 3) | 20% of damage | Lifesteal |

### 8.3 Healing Implementation

```javascript
class HealingSystem {
    applyHealing(target, amount, source) {
        // Calculate actual healing
        const missingHealth = target.maxHealth - target.currentHealth;
        const actualHeal = Math.min(amount, missingHealth);

        if (actualHeal <= 0) return; // Already full

        // Apply healing
        target.currentHealth += actualHeal;

        // Show feedback
        this.showHealingNumber(target.position, actualHeal);
        VFX.spawn('healing_particles', target.position);
        Audio.play('heal_tick', { volume: 0.3 });

        // Track for stats
        target.stats.healingReceived += actualHeal;
        if (source !== target) {
            source.stats.healingDone += actualHeal;
        }
    }

    applyHealOverTime(target, healPerTick, duration, tickRate) {
        const hot = new HoT({
            target: target,
            healPerTick: healPerTick,
            duration: duration,
            tickRate: tickRate
        });

        target.activeHoTs.push(hot);
        return hot;
    }
}
```

---

## 9. Damage Over Time (DoT)

### 9.1 DoT System

```javascript
class DoTSystem {
    applyDoT(target, config) {
        const dot = new DoT({
            type: config.type,        // 'POISON', 'FIRE', etc.
            damagePerTick: config.damage,
            tickRate: config.tickRate || 1, // Seconds
            duration: config.duration,
            source: config.source
        });

        // Check for existing DoT of same type
        const existing = target.activeDoTs.find(d => d.type === config.type);
        if (existing) {
            // Refresh duration, don't stack (or stack based on design)
            existing.duration = config.duration;
            existing.damagePerTick = Math.max(existing.damagePerTick, config.damage);
        } else {
            target.activeDoTs.push(dot);
        }
    }

    updateDoTs(target, deltaTime) {
        for (const dot of target.activeDoTs) {
            dot.timeSinceLastTick += deltaTime;

            // Apply tick damage
            if (dot.timeSinceLastTick >= dot.tickRate) {
                this.applyDotTick(target, dot);
                dot.timeSinceLastTick = 0;
            }

            // Reduce duration
            dot.duration -= deltaTime;

            // Remove expired DoTs
            if (dot.duration <= 0) {
                this.removeDoT(target, dot);
            }
        }
    }

    applyDotTick(target, dot) {
        target.takeDamage(dot.damagePerTick, dot.source, dot.type);

        // Show DoT number
        FloatingCombatText.create(target.position, dot.damagePerTick, dot.type);
    }
}
```

### 9.2 DoT Visual Indicators

| DoT Type | Icon | Bar Effect |
|----------|------|------------|
| Poison | 🤢 | Green tint |
| Fire | 🔥 | Orange pulse |
| Bleed | 🩸 | Red drips |

---

## 10. Server Authority

### 10.1 Health Synchronization

```javascript
// Server-side health management
class ServerHealthManager {
    takeDamage(targetId, damage, sourceId, damageType) {
        const target = this.getPlayer(targetId);
        const source = this.getPlayer(sourceId);

        // Validate damage
        if (!this.validateDamageRequest(source, target, damage)) {
            return { success: false };
        }

        // Calculate final damage
        const result = DamageCalculator.calculateDamage(
            damage, source, target, damageType
        );

        // Apply damage
        target.currentHealth -= result.finalDamage;

        // Broadcast to all clients
        this.broadcastHealthUpdate(target);

        // Check for elimination
        if (target.currentHealth <= 0) {
            this.triggerElimination(target, source);
        }

        return {
            success: true,
            damage: result.finalDamage,
            isCrit: result.isCrit
        };
    }
}
```

### 10.2 Anti-Cheat Validation

```javascript
class DamageValidator {
    validateDamageRequest(source, target, damage) {
        // 1. Source is alive
        if (source.state !== PlayerState.ALIVE) {
            return false;
        }

        // 2. Target is damageable
        if (target.isInvulnerable || target.state !== PlayerState.ALIVE) {
            return false;
        }

        // 3. Damage amount is reasonable
        const maxPossibleDamage = this.getMaxPossibleDamage(source);
        if (damage > maxPossibleDamage) {
            this.flagSuspicious(source, 'EXCESSIVE_DAMAGE');
            return false;
        }

        // 4. Rate limiting
        if (!this.validateDamageRate(source)) {
            return false;
        }

        return true;
    }

    getMaxPossibleDamage(source) {
        // Calculate maximum possible damage based on:
        // - Abilities owned
        // - Level/damage multiplier
        // - Potential crits
        return source.highestAbilityDamage * source.damageMultiplier * 1.5 * 1.1;
    }
}
```

---

## 11. Network Synchronization

### 11.1 Health Update Packet

```javascript
struct HealthUpdatePacket {
    uint32 playerId;
    uint16 currentHealth;
    uint16 maxHealth;
    uint8  lastDamageType;
    int16  healthDelta;        // +/- for damage/healing
    uint32 sourcePlayerId;     // Who dealt damage/healing
    uint8  isCrit;             // Boolean
    float  timestamp;
}
```

### 11.2 Sync Frequency

| Event | Sync | Recipients |
|-------|------|------------|
| Damage Taken | Immediate | All nearby |
| Healing | Immediate | Self + nearby |
| Max Health Change | Immediate | All |
| Full Health | On request | Self |

---

## 12. Balance Considerations

### 12.1 TTK Analysis

| Attacker Level | Defender Level | Defender HP | Avg TTK |
|----------------|----------------|-------------|---------|
| 1 | 1 | 100 | 8-12s |
| 5 | 5 | 140 | 10-14s |
| 10 | 10 | 200 | 12-16s |
| 10 | 1 | 100 | 5-8s |
| 1 | 10 | 200 | 15-20s |

### 12.2 Design Goals

- **Early Game:** Quick fights, high risk
- **Mid Game:** Moderate TTK, ability-dependent
- **Late Game:** Longer fights, level advantages matter
- **Overall:** Skill > Level (but level helps)

---

## 13. Testing Requirements

### 13.1 Unit Tests

| Test | Description |
|------|-------------|
| Damage calculation | Correct damage with all modifiers |
| Health scaling | Correct HP at each level |
| Crit system | Correct chance and multiplier |
| DoT ticks | Accurate timing and damage |
| Healing cap | Cannot exceed max HP |

### 13.2 Integration Tests

| Test | Description |
|------|-------------|
| Full combat flow | Damage → Health → UI → Elimination |
| Network sync | Health matches across clients |
| Level up | HP increases correctly |
| Multiple DoTs | All tick correctly |

### 13.3 Balance Testing

| Metric | Target |
|--------|--------|
| Avg TTK | 10-15 seconds |
| Crit impact | Noticeable but not OP |
| Healing effectiveness | Meaningful but not dominant |

---

## 14. Appendices

### Appendix A: Health Calculation Quick Reference

```
Max Health = 100 + (Level - 1) * 10 + (Level 10 bonus)
Damage = Base * AttackerMultiplier * CritMultiplier * (1 - DefenderReduction) * Variance
```

### Appendix B: Damage Type Future Expansions

| Expansion | Description |
|-----------|-------------|
| Elemental Weakness | Fire > Ice > Lightning > Fire |
| Armor Types | Physical vs Magical damage |
| Shield Systems | Temporary HP pools |

### Appendix C: Performance Budget

| Operation | Budget |
|-----------|--------|
| Damage calculation | < 0.1ms |
| Health UI update | < 0.5ms |
| DoT tick processing | < 0.2ms |
| Network packet | < 0.1ms |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
