# Core Ability Implementation Specification

## Document Information
- **Task ID:** GAME-008
- **Priority:** P0
- **Complexity:** XL
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the core ability system and initial roster of 10 MVP abilities for Plunderstorm Mobile. Each ability is designed with clear visual feedback, mobile-friendly controls, and balanced gameplay in mind.

### 1.2 Scope
- Ability system architecture
- 10 MVP abilities (5 offensive, 5 utility)
- Ability properties and scaling
- VFX and SFX specifications
- Targeting and input systems
- Cooldown and damage calculations
- Counterplay considerations

### 1.3 Dependencies
- GAME-007: Ability Slot System
- UX-003: Touch Controls
- ART-013: Ability VFX

---

## 2. Ability System Architecture

### 2.1 Base Ability Class

```javascript
class Ability {
    // Identity
    id = '';                    // Unique identifier
    name = '';                  // Display name
    description = '';           // Tooltip text
    icon = '';                  // Icon asset reference
    slotType = 'OFFENSIVE';     // 'OFFENSIVE' or 'UTILITY'

    // Stats (base values at Rank 1)
    damage = 0;                 // Base damage
    cooldown = 10;              // Seconds
    range = 15;                 // Meters
    castTime = 0;               // Instant by default
    duration = 0;               // For channeled/DoT abilities

    // Scaling per rank
    damagePerRank = 0;
    cooldownReductionPerRank = 0;

    // Targeting
    targetType = 'SKILLSHOT';   // SKILLSHOT, AOE, SELF, MELEE_ARC
    aoeRadius = 0;              // For AOE abilities
    projectileSpeed = 0;        // For projectiles

    // Animation
    castAnimation = '';
    impactAnimation = '';

    // Audio
    castSound = '';
    impactSound = '';
}
```

### 2.2 Ability Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    ABILITY EXECUTION FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Player Input                                                    │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────────┐                                               │
│  │ Check        │  Cooldown ready?                               │
│  │ Availability │  Not stunned/silenced?                         │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ Targeting    │  Get target position/direction                 │
│  │ Phase        │  (if required)                                 │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ Cast Start   │  Play animation, lock movement (if any)        │
│  │              │  Deduct resources (if any)                     │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ Execute      │  Spawn projectile/effect                       │
│  │ Ability      │  Apply damage/effects                          │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ Start        │  Begin cooldown timer                          │
│  │ Cooldown     │  Update UI                                     │
│  └──────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Rank System

| Rank | Stars | Damage Multiplier | Cooldown Reduction | Special |
|------|-------|-------------------|-------------------|---------|
| 1 | ★☆☆ | 100% | 0% | Base ability |
| 2 | ★★☆ | 125% | 10% | Enhanced VFX |
| 3 | ★★★ | 150% | 20% | Bonus effect + Epic VFX |

---

## 3. Offensive Abilities

### 3.1 Fireball

```
┌─────────────────────────────────────────────────────────────────┐
│                       FIREBALL                                   │
│                    Type: Offensive                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Launch a blazing fireball that explodes on impact,              │
│  dealing area damage to enemies.                                 │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │         🔥                                                │  │
│  │       🔥🔥🔥  ──────────────────────►  💥                │  │
│  │         🔥                              (impact)          │  │
│  │                                                            │  │
│  │      [Player]        [Projectile]        [Explosion]      │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Damage | 40 | 50 | 60 |
| Cooldown | 8s | 7.2s | 6.4s |
| Range | 20m | 20m | 20m |
| AoE Radius | 2m | 2.5m | 3m |
| Projectile Speed | 35 m/s | 35 m/s | 35 m/s |
| **Rank 3 Bonus** | - | - | Leaves burning ground (2s, 10 DPS) |

#### Implementation

```javascript
class Fireball extends Ability {
    id = 'fireball';
    name = 'Fireball';
    slotType = 'OFFENSIVE';
    targetType = 'SKILLSHOT';

    // Base stats (Rank 1)
    damage = 40;
    cooldown = 8;
    range = 20;
    aoeRadius = 2;
    projectileSpeed = 35;

    execute(caster, targetDirection) {
        // Spawn projectile
        const projectile = new Projectile({
            position: caster.position,
            direction: targetDirection,
            speed: this.projectileSpeed,
            maxDistance: this.range,
            onHit: (hitPoint) => this.onImpact(caster, hitPoint)
        });

        return projectile;
    }

    onImpact(caster, hitPoint) {
        // Get all enemies in AoE
        const enemies = Physics.overlapSphere(hitPoint, this.aoeRadius);

        for (const enemy of enemies) {
            enemy.takeDamage(this.getDamage(caster), caster, 'FIRE');
        }

        // Spawn explosion VFX
        VFX.spawn('fireball_explosion', hitPoint);
        Audio.play('fireball_impact', hitPoint);

        // Rank 3: Leave burning ground
        if (this.rank >= 3) {
            this.createBurningGround(hitPoint);
        }
    }

    createBurningGround(position) {
        const zone = new DamageZone({
            position: position,
            radius: this.aoeRadius,
            damage: 10,
            tickRate: 1,
            duration: 2,
            damageType: 'FIRE'
        });

        zone.spawn();
    }
}
```

#### VFX Specification

| Phase | Effect | Duration |
|-------|--------|----------|
| Cast | Orange muzzle flash, hand glow | 0.2s |
| Projectile | Flaming ball with trail | Until impact |
| Impact | Explosion with fire particles | 0.5s |
| Rank 3 Ground | Flickering fire on ground | 2s |

---

### 3.2 Cutlass Slash

```
┌─────────────────────────────────────────────────────────────────┐
│                     CUTLASS SLASH                                │
│                    Type: Offensive                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Perform a devastating melee slash in an arc,                    │
│  dealing high damage to nearby enemies.                          │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │              ╱───╲                                         │  │
│  │             ╱  ⚔️  ╲   180° arc                            │  │
│  │            ╱       ╲                                       │  │
│  │           ╱─────────╲                                      │  │
│  │          [Player]                                          │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Damage | 55 | 69 | 83 |
| Cooldown | 6s | 5.4s | 4.8s |
| Range | 4m | 4m | 4.5m |
| Arc Angle | 180° | 180° | 180° |
| **Rank 3 Bonus** | - | - | 20% lifesteal on hit |

#### Implementation

```javascript
class CutlassSlash extends Ability {
    id = 'cutlass_slash';
    name = 'Cutlass Slash';
    slotType = 'OFFENSIVE';
    targetType = 'MELEE_ARC';

    damage = 55;
    cooldown = 6;
    range = 4;
    arcAngle = 180;

    execute(caster, facingDirection) {
        // Play slash animation
        caster.playAnimation('cutlass_slash');

        // Get all enemies in arc
        const enemies = Physics.overlapArc(
            caster.position,
            facingDirection,
            this.range,
            this.arcAngle
        );

        for (const enemy of enemies) {
            const damage = this.getDamage(caster);
            enemy.takeDamage(damage, caster, 'PHYSICAL');

            // Rank 3: Lifesteal
            if (this.rank >= 3) {
                caster.heal(damage * 0.2);
            }
        }

        // VFX
        VFX.spawn('slash_arc', caster.position, facingDirection);
        Audio.play('cutlass_slash', caster.position);
    }
}
```

---

### 3.3 Cannonball

```
┌─────────────────────────────────────────────────────────────────┐
│                      CANNONBALL                                  │
│                    Type: Offensive                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Fire a slow but devastating cannonball that deals               │
│  massive damage on direct hit.                                   │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │                      ⚫ ─ ─ ─ ─ ─ ─ ►                     │  │
│  │      [Player]     (slow, heavy)                           │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Damage | 80 | 100 | 120 |
| Cooldown | 12s | 10.8s | 9.6s |
| Range | 30m | 30m | 35m |
| Projectile Speed | 20 m/s | 20 m/s | 22 m/s |
| **Rank 3 Bonus** | - | - | Knockback on hit (8m) |

#### Implementation

```javascript
class Cannonball extends Ability {
    id = 'cannonball';
    name = 'Cannonball';
    slotType = 'OFFENSIVE';
    targetType = 'SKILLSHOT';

    damage = 80;
    cooldown = 12;
    range = 30;
    projectileSpeed = 20;
    projectileSize = 0.5; // Larger hitbox

    execute(caster, targetDirection) {
        const projectile = new Projectile({
            position: caster.position,
            direction: targetDirection,
            speed: this.projectileSpeed,
            maxDistance: this.range,
            size: this.projectileSize,
            onHit: (target, hitPoint) => this.onImpact(caster, target, hitPoint)
        });

        // Visual: Heavy trail
        projectile.addTrail('cannonball_smoke');

        return projectile;
    }

    onImpact(caster, target, hitPoint) {
        target.takeDamage(this.getDamage(caster), caster, 'PHYSICAL');

        // Rank 3: Knockback
        if (this.rank >= 3) {
            const knockbackDir = (target.position - caster.position).normalized;
            target.applyKnockback(knockbackDir, 8);
        }

        VFX.spawn('cannonball_impact', hitPoint);
        Audio.play('cannon_boom', hitPoint);
    }
}
```

---

### 3.4 Lightning Strike

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIGHTNING STRIKE                              │
│                    Type: Offensive                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Call down lightning at a target area after a short delay.       │
│  Enemies in the area take heavy damage.                          │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │              ⚡                                            │  │
│  │              ⚡                                            │  │
│  │              ⚡                                            │  │
│  │           ┌──────┐  ← Targeting indicator (0.8s delay)    │  │
│  │           │ 💥💥 │                                        │  │
│  │           │ 💥💥 │  ← AoE damage zone                     │  │
│  │           └──────┘                                        │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Damage | 50 | 63 | 75 |
| Cooldown | 10s | 9s | 8s |
| Range | 18m | 18m | 20m |
| AoE Radius | 3m | 3m | 3.5m |
| Delay | 0.8s | 0.7s | 0.6s |
| **Rank 3 Bonus** | - | - | 1s stun on hit |

#### Implementation

```javascript
class LightningStrike extends Ability {
    id = 'lightning_strike';
    name = 'Lightning Strike';
    slotType = 'OFFENSIVE';
    targetType = 'AOE';

    damage = 50;
    cooldown = 10;
    range = 18;
    aoeRadius = 3;
    delay = 0.8;

    execute(caster, targetPosition) {
        // Show warning indicator
        const indicator = VFX.spawn('lightning_indicator', targetPosition);
        indicator.setRadius(this.aoeRadius);

        // Schedule strike after delay
        setTimeout(() => {
            this.strike(caster, targetPosition);
            indicator.destroy();
        }, this.delay * 1000);
    }

    strike(caster, position) {
        // VFX: Lightning bolt from sky
        VFX.spawn('lightning_bolt', position);
        Audio.play('thunder_strike', position);

        // Damage enemies in radius
        const enemies = Physics.overlapSphere(position, this.aoeRadius);

        for (const enemy of enemies) {
            enemy.takeDamage(this.getDamage(caster), caster, 'LIGHTNING');

            // Rank 3: Stun
            if (this.rank >= 3) {
                enemy.applyCC('STUN', 1.0);
            }
        }
    }
}
```

---

### 3.5 Poison Mackerel

```
┌─────────────────────────────────────────────────────────────────┐
│                    POISON MACKEREL                               │
│                    Type: Offensive                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Throw a rotten fish that poisons enemies on hit,                │
│  dealing damage over time.                                       │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │        🐟~~~~~~~~~~~~~~~►                                 │  │
│  │      [Player]           [Enemy] 🤢 (poisoned)             │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Impact Damage | 15 | 19 | 23 |
| DoT Damage | 8/s | 10/s | 12/s |
| DoT Duration | 4s | 4s | 5s |
| **Total Damage** | 47 | 59 | 83 |
| Cooldown | 7s | 6.3s | 5.6s |
| Range | 15m | 15m | 18m |
| Projectile Speed | 30 m/s | 30 m/s | 30 m/s |
| **Rank 3 Bonus** | - | - | Spreads to nearby enemy (5m) |

#### Implementation

```javascript
class PoisonMackerel extends Ability {
    id = 'poison_mackerel';
    name = 'Poison Mackerel';
    slotType = 'OFFENSIVE';
    targetType = 'SKILLSHOT';

    damage = 15;  // Impact damage
    dotDamage = 8;
    dotDuration = 4;
    cooldown = 7;
    range = 15;
    projectileSpeed = 30;

    execute(caster, targetDirection) {
        const projectile = new Projectile({
            position: caster.position,
            direction: targetDirection,
            speed: this.projectileSpeed,
            maxDistance: this.range,
            onHit: (target) => this.onHit(caster, target)
        });

        return projectile;
    }

    onHit(caster, target) {
        // Impact damage
        target.takeDamage(this.damage, caster, 'POISON');

        // Apply poison DoT
        target.applyDoT({
            type: 'POISON',
            damage: this.dotDamage,
            duration: this.dotDuration,
            source: caster
        });

        // Rank 3: Spread
        if (this.rank >= 3) {
            const nearbyEnemy = this.findNearbyEnemy(target, 5);
            if (nearbyEnemy) {
                nearbyEnemy.applyDoT({
                    type: 'POISON',
                    damage: this.dotDamage * 0.5,
                    duration: this.dotDuration,
                    source: caster
                });
            }
        }

        VFX.spawn('poison_splash', target.position);
        Audio.play('fish_splat', target.position);
    }
}
```

---

## 4. Utility Abilities

### 4.1 Grappling Hook

```
┌─────────────────────────────────────────────────────────────────┐
│                    GRAPPLING HOOK                                │
│                     Type: Utility                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Fire a grappling hook that pulls you to the target location.    │
│  Great for mobility and escapes.                                 │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │      [Player] ═══════════════► [Hook Point]               │  │
│  │                    ↓                                       │  │
│  │              [Player Dashes] ──────────────►              │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Cooldown | 10s | 9s | 8s |
| Range | 15m | 18m | 20m |
| Pull Speed | 25 m/s | 27 m/s | 30 m/s |
| **Rank 3 Bonus** | - | - | Gain 25% movement speed for 2s after landing |

#### Implementation

```javascript
class GrapplingHook extends Ability {
    id = 'grappling_hook';
    name = 'Grappling Hook';
    slotType = 'UTILITY';
    targetType = 'SKILLSHOT';

    cooldown = 10;
    range = 15;
    pullSpeed = 25;

    execute(caster, targetPosition) {
        // Clamp to max range
        const direction = (targetPosition - caster.position).normalized;
        const distance = Math.min(
            Vector3.Distance(caster.position, targetPosition),
            this.range
        );
        const hookPoint = caster.position + direction * distance;

        // Check for terrain/obstacles
        const hit = Physics.raycast(caster.position, direction, distance);
        const finalPoint = hit ? hit.point : hookPoint;

        // Fire hook
        this.fireHook(caster, finalPoint);
    }

    fireHook(caster, targetPoint) {
        // VFX: Hook projectile
        const hookVFX = VFX.spawn('grapple_hook', caster.position);
        hookVFX.animateToPosition(targetPoint, 0.1);

        // Pull player after hook lands
        setTimeout(() => {
            this.pullPlayer(caster, targetPoint);
        }, 100);
    }

    pullPlayer(caster, targetPoint) {
        // Disable normal movement during pull
        caster.movementLocked = true;

        // Dash to point
        caster.dashTo(targetPoint, this.pullSpeed, () => {
            caster.movementLocked = false;

            // Rank 3: Speed boost
            if (this.rank >= 3) {
                caster.applyBuff('MOVEMENT_SPEED', 0.25, 2);
            }
        });

        Audio.play('grapple_pull', caster.position);
    }
}
```

---

### 4.2 Barrel Roll

```
┌─────────────────────────────────────────────────────────────────┐
│                      BARREL ROLL                                 │
│                     Type: Utility                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Perform a quick dodge roll, becoming briefly invulnerable.      │
│  Essential for avoiding damage.                                  │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │      [Player] ──🔄──🔄──🔄──► [New Position]              │  │
│  │              (invulnerable during roll)                    │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Cooldown | 8s | 7.2s | 6.4s |
| Roll Distance | 6m | 7m | 8m |
| Invulnerability | 0.4s | 0.4s | 0.5s |
| Roll Speed | 20 m/s | 22 m/s | 24 m/s |
| **Rank 3 Bonus** | - | - | Cleanse all CC effects on use |

#### Implementation

```javascript
class BarrelRoll extends Ability {
    id = 'barrel_roll';
    name = 'Barrel Roll';
    slotType = 'UTILITY';
    targetType = 'SELF';

    cooldown = 8;
    rollDistance = 6;
    invulnDuration = 0.4;
    rollSpeed = 20;

    execute(caster, moveDirection) {
        // Use movement input direction, or facing if not moving
        const direction = moveDirection.magnitude > 0
            ? moveDirection.normalized
            : caster.facingDirection;

        // Rank 3: Cleanse CC
        if (this.rank >= 3) {
            caster.clearAllCC();
        }

        // Grant invulnerability
        caster.setInvulnerable(true);

        // Perform roll
        caster.playAnimation('barrel_roll');
        caster.dashInDirection(direction, this.rollDistance, this.rollSpeed);

        // Remove invulnerability after duration
        setTimeout(() => {
            caster.setInvulnerable(false);
        }, this.invulnDuration * 1000);

        VFX.spawn('roll_dust', caster.position);
        Audio.play('barrel_roll', caster.position);
    }
}
```

---

### 4.3 Healing Grog

```
┌─────────────────────────────────────────────────────────────────┐
│                     HEALING GROG                                 │
│                     Type: Utility                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Take a swig of magical grog that heals you over time.           │
│  Drink up, ye scurvy dog!                                        │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │      [Player] 🍺  +❤️ +❤️ +❤️ +❤️                        │  │
│  │              (healing over 4 seconds)                      │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Total Healing | 40 | 50 | 60 |
| Duration | 4s | 4s | 4s |
| Heal/Second | 10 | 12.5 | 15 |
| Cooldown | 15s | 13.5s | 12s |
| **Rank 3 Bonus** | - | - | 20% damage reduction while healing |

#### Implementation

```javascript
class HealingGrog extends Ability {
    id = 'healing_grog';
    name = 'Healing Grog';
    slotType = 'UTILITY';
    targetType = 'SELF';

    totalHealing = 40;
    healDuration = 4;
    cooldown = 15;

    execute(caster) {
        // Play drinking animation (brief)
        caster.playAnimation('drink_grog', 0.3);

        // Apply healing over time
        const healPerTick = this.totalHealing / this.healDuration;

        caster.applyHoT({
            healing: healPerTick,
            duration: this.healDuration,
            tickRate: 1
        });

        // Rank 3: Damage reduction
        if (this.rank >= 3) {
            caster.applyBuff('DAMAGE_REDUCTION', 0.20, this.healDuration);
        }

        // VFX: Green healing particles
        VFX.attachToPlayer('healing_particles', caster, this.healDuration);
        Audio.play('drink_gulp', caster.position);
    }
}
```

---

### 4.4 Smoke Bomb

```
┌─────────────────────────────────────────────────────────────────┐
│                      SMOKE BOMB                                  │
│                     Type: Utility                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Throw a smoke bomb that creates a vision-blocking cloud         │
│  and slows enemies within.                                       │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │           ░░░░░░░░░░░                                      │  │
│  │          ░░░░░░░░░░░░░  Smoke cloud                        │  │
│  │         ░░░░░░💨░░░░░░░ (blocks vision)                   │  │
│  │          ░░░░░░░░░░░░░  (slows enemies)                    │  │
│  │           ░░░░░░░░░░░                                      │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Cooldown | 12s | 10.8s | 9.6s |
| Range | 12m | 14m | 16m |
| Cloud Radius | 4m | 4.5m | 5m |
| Duration | 3s | 3.5s | 4s |
| Slow Amount | 30% | 35% | 40% |
| **Rank 3 Bonus** | - | - | Also silences enemies for 1s on entry |

#### Implementation

```javascript
class SmokeBomb extends Ability {
    id = 'smoke_bomb';
    name = 'Smoke Bomb';
    slotType = 'UTILITY';
    targetType = 'AOE';

    cooldown = 12;
    range = 12;
    cloudRadius = 4;
    cloudDuration = 3;
    slowAmount = 0.30;

    execute(caster, targetPosition) {
        // Throw bomb
        const bomb = new Projectile({
            position: caster.position,
            target: targetPosition,
            speed: 25,
            arc: true,
            onLand: (landPos) => this.createCloud(caster, landPos)
        });

        return bomb;
    }

    createCloud(caster, position) {
        const cloud = new AreaEffect({
            position: position,
            radius: this.cloudRadius,
            duration: this.cloudDuration,
            onEnter: (entity) => this.onCloudEnter(caster, entity),
            onStay: (entity) => this.onCloudStay(entity),
            onExit: (entity) => this.onCloudExit(entity)
        });

        // VFX: Smoke particles
        cloud.attachVFX('smoke_cloud');

        // Block vision
        cloud.blocksVision = true;

        Audio.play('smoke_poof', position);
    }

    onCloudEnter(caster, entity) {
        if (entity.team !== caster.team) {
            entity.applySlow(this.slowAmount);

            // Rank 3: Silence
            if (this.rank >= 3) {
                entity.applyCC('SILENCE', 1.0);
            }
        }
    }

    onCloudStay(entity) {
        // Maintain slow while in cloud
    }

    onCloudExit(entity) {
        entity.removeSlow(this.slowAmount);
    }
}
```

---

### 4.5 Wind Burst

```
┌─────────────────────────────────────────────────────────────────┐
│                      WIND BURST                                  │
│                     Type: Utility                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Description:                                                    │
│  Release a powerful gust of wind that knocks back                │
│  all enemies around you.                                         │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │              ↖ [Enemy]                                     │  │
│  │         ← [Enemy]     [Enemy] →                           │  │
│  │              💨 [Player] 💨                               │  │
│  │         ← [Enemy]     [Enemy] →                           │  │
│  │              ↙ [Enemy]                                     │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Properties

| Property | Rank 1 | Rank 2 | Rank 3 |
|----------|--------|--------|--------|
| Damage | 20 | 25 | 30 |
| Cooldown | 14s | 12.6s | 11.2s |
| Radius | 5m | 6m | 7m |
| Knockback | 6m | 7m | 8m |
| **Rank 3 Bonus** | - | - | Also reflects projectiles back |

#### Implementation

```javascript
class WindBurst extends Ability {
    id = 'wind_burst';
    name = 'Wind Burst';
    slotType = 'UTILITY';
    targetType = 'SELF';

    damage = 20;
    cooldown = 14;
    radius = 5;
    knockbackDistance = 6;

    execute(caster) {
        // VFX: Circular wind wave
        VFX.spawn('wind_burst_ring', caster.position);
        Audio.play('wind_gust', caster.position);

        // Get all enemies in radius
        const enemies = Physics.overlapSphere(caster.position, this.radius);

        for (const enemy of enemies) {
            if (enemy.team === caster.team) continue;

            // Calculate knockback direction
            const direction = (enemy.position - caster.position).normalized;

            // Apply damage
            enemy.takeDamage(this.getDamage(caster), caster, 'WIND');

            // Apply knockback
            enemy.applyKnockback(direction, this.knockbackDistance);
        }

        // Rank 3: Reflect projectiles
        if (this.rank >= 3) {
            this.reflectProjectiles(caster);
        }
    }

    reflectProjectiles(caster) {
        const projectiles = Physics.getProjectilesInRadius(caster.position, this.radius);

        for (const proj of projectiles) {
            if (proj.owner !== caster) {
                proj.reverseDirection();
                proj.owner = caster; // Reflected projectiles belong to caster
            }
        }
    }
}
```

---

## 5. Ability Balance Summary

### 5.1 Damage Comparison (Rank 1)

| Ability | Base Damage | DPS Potential | Notes |
|---------|-------------|---------------|-------|
| Fireball | 40 | ~5 (8s CD) | AoE potential |
| Cutlass Slash | 55 | ~9.2 (6s CD) | High risk melee |
| Cannonball | 80 | ~6.7 (12s CD) | Slow, telegraphed |
| Lightning Strike | 50 | ~5 (10s CD) | Delayed, dodgeable |
| Poison Mackerel | 47 total | ~6.7 (7s CD) | DoT spread |
| Wind Burst | 20 | ~1.4 (14s CD) | Utility focus |

### 5.2 Counterplay Matrix

| Ability | Countered By |
|---------|--------------|
| Fireball | Barrel Roll (dodge) |
| Cutlass Slash | Grappling Hook (escape), Wind Burst (knockback) |
| Cannonball | Any mobility (slow projectile) |
| Lightning Strike | Movement (indicator visible) |
| Poison Mackerel | Healing Grog (out-heal DoT) |
| Grappling Hook | Wind Burst (interrupt landing) |
| Barrel Roll | None (invuln frames) |
| Healing Grog | Burst damage, Poison (reduces healing) |
| Smoke Bomb | Wind Burst (clear smoke) |
| Wind Burst | Barrel Roll (avoid knockback) |

---

## 6. VFX and Audio Summary

### 6.1 VFX Asset List

| Ability | Cast VFX | Projectile VFX | Impact VFX |
|---------|----------|----------------|------------|
| Fireball | Hand flame | Fire trail | Explosion |
| Cutlass Slash | - | Slash arc | Sparks |
| Cannonball | Muzzle smoke | Smoke trail | Debris burst |
| Lightning Strike | Sky charge | Lightning bolt | Ground scorch |
| Poison Mackerel | - | Fish + stink | Green splash |
| Grappling Hook | - | Hook + rope | Dust puff |
| Barrel Roll | - | Motion blur | Dust trail |
| Healing Grog | Drink anim | - | Green hearts |
| Smoke Bomb | - | Arc trail | Smoke cloud |
| Wind Burst | Wind spiral | - | Gust lines |

### 6.2 Audio Asset List

| Ability | Cast Sound | Impact Sound |
|---------|------------|--------------|
| Fireball | Woosh flame | Boom + crackle |
| Cutlass Slash | Sword swing | Metal clash |
| Cannonball | Cannon fire | Heavy thud |
| Lightning Strike | Thunder rumble | Electric crack |
| Poison Mackerel | Throw grunt | Wet splat |
| Grappling Hook | Hook launch | Metal clank |
| Barrel Roll | Roll whoosh | Thud |
| Healing Grog | Gulp | Ahh |
| Smoke Bomb | Throw | Poof |
| Wind Burst | Inhale | Gust |

---

## 7. Testing Requirements

### 7.1 Unit Tests Per Ability

| Test | Description |
|------|-------------|
| Damage calculation | Correct damage at each rank |
| Cooldown timing | Accurate cooldown duration |
| Range enforcement | Cannot exceed max range |
| Targeting | Correct target acquisition |
| Rank bonuses | Special effects activate at Rank 3 |

### 7.2 Integration Tests

| Test | Description |
|------|-------------|
| All abilities in combat | No crashes or exploits |
| Ability interactions | Combos work as expected |
| Network sync | All clients see same results |
| Mobile input | Touch controls responsive |

### 7.3 Balance Testing

| Metric | Target |
|--------|--------|
| Time to kill (same level) | 8-15 seconds |
| Ability diversity in wins | All abilities viable |
| Rank 3 advantage | Significant but not OP |

---

## 8. Appendices

### Appendix A: Damage Type Reference

| Type | Description | Interactions |
|------|-------------|--------------|
| FIRE | Magical flame | No special |
| PHYSICAL | Weapon strikes | No special |
| LIGHTNING | Electrical | Bonus vs wet targets (future) |
| POISON | Toxic DoT | Reduced by healing |
| WIND | Force damage | Knockback |

### Appendix B: Targeting Type Reference

| Type | Description | Input |
|------|-------------|-------|
| SKILLSHOT | Aimed projectile | Drag to aim |
| AOE | Ground target circle | Tap to place |
| SELF | Centered on caster | Single tap |
| MELEE_ARC | Frontal cone | Facing direction |

### Appendix C: Mobile Control Mappings

| Ability Type | Primary Action | Secondary Action |
|--------------|----------------|------------------|
| Skillshot | Drag + Release | Tap for auto-aim |
| AOE | Drag to position | Release to cast |
| Self | Single tap | N/A |
| Melee Arc | Single tap | N/A |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
