# Basic Attack System Specification

## Document Information
- **Task ID:** GAME-011
- **Priority:** P0
- **Complexity:** M
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the basic attack system for Plunderstorm Mobile. The basic attack is always available to players regardless of looted abilities, serving as a reliable fallback during early game and cooldown periods.

### 1.2 Scope
- Basic attack properties and damage
- Aim assist system for mobile
- Manual aim mode for skilled players
- Fire rate and cooldown
- Visual and audio feedback
- Network synchronization

### 1.3 Dependencies
- UX-003: Touch Controls
- GAME-012: Damage and Health System

---

## 2. Basic Attack Properties

### 2.1 Core Statistics

| Property | Value | Notes |
|----------|-------|-------|
| Damage | 15 | Per shot |
| Cooldown | 0.8s | Fire rate limit |
| Range | 15m | Maximum effective range |
| Projectile Speed | 50 m/s | Fast, nearly hitscan feel |
| Projectile Type | Energy bolt / Pistol shot | Visual theme |

### 2.2 Availability

```
┌─────────────────────────────────────────────────────────────────┐
│                    BASIC ATTACK AVAILABILITY                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✅ Always available - no pickup required                        │
│  ✅ No cooldown at match start (ready immediately)               │
│  ✅ Available during ability cooldowns                           │
│  ✅ Available while abilities charging/targeting                 │
│                                                                  │
│  ❌ NOT available during:                                        │
│     - Stun/CC states                                             │
│     - Death/Eliminated state                                     │
│     - Certain ability cast animations                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Comparison to Abilities

| Aspect | Basic Attack | Abilities |
|--------|--------------|-----------|
| Availability | Always | Must be found |
| Damage | Low (15) | Medium-High (40-80) |
| Cooldown | Very short (0.8s) | Medium-Long (6-15s) |
| Special Effects | None | Varies |
| Skill Expression | Aim accuracy | Timing, positioning |

---

## 3. Control Systems

### 3.1 Mobile Control Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    BASIC ATTACK CONTROLS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                    [GAME VIEW]                          │     │
│  │                                                         │     │
│  │                                                         │     │
│  │                                                         │     │
│  │                                                         │     │
│  │  ┌─────┐                              ┌─────┐ ┌─────┐  │     │
│  │  │Move │                              │Aim/ │ │ ⚔️  │  │     │
│  │  │Stick│                              │Look │ │Fire │  │     │
│  │  └─────┘                              └─────┘ └─────┘  │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Fire Button Options:                                            │
│  - Dedicated fire button (⚔️)                                    │
│  - Tap on aim area (auto-fire on tap)                           │
│  - Hold for rapid fire (if enabled)                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Fire Input Methods

| Method | Description | Best For |
|--------|-------------|----------|
| Tap Fire | Single tap = single shot | Precision |
| Hold Fire | Hold = continuous fire | DPS sustained |
| Aim + Fire | Drag aim, release = fire | Advanced control |

### 3.3 Control Configuration

```javascript
class BasicAttackControls {
    // Configuration options
    static CONFIG = {
        // Fire button behavior
        tapToFire: true,           // Single tap fires
        holdToFire: true,          // Hold for continuous
        holdFireDelay: 0.2,        // Seconds before hold-fire starts

        // Aim assist
        aimAssistEnabled: true,
        aimAssistStrength: 0.4,    // 0-1 scale
        aimAssistRadius: 30,       // Degrees

        // Visual aids
        showCrosshair: true,
        showProjectileTrail: true,
        showHitMarker: true
    };
}
```

---

## 4. Aim Assist System

### 4.1 Aim Assist Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    AIM ASSIST VISUALIZATION                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Without Aim Assist:              With Aim Assist:               │
│                                                                  │
│        ⊕ (crosshair)                    ⊕ (crosshair)           │
│         ↓                                ↓                       │
│         ↓                                 ↘                      │
│         ↓                                   ↘                    │
│         ↓                                     ↘                  │
│         ↓ (miss)                               ↘ (hit!)          │
│                         [Enemy]              [Enemy]             │
│                                                                  │
│  Aim assist "bends" the shot toward nearby enemies               │
│  within the assist cone                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Aim Assist Implementation

```javascript
class AimAssist {
    static ASSIST_CONE_ANGLE = 30;   // Degrees from center
    static ASSIST_STRENGTH = 0.4;    // 0-1 blend factor
    static ASSIST_RANGE = 20;        // Meters
    static STICKY_AIM_SLOWDOWN = 0.7; // Reduce turn speed on target

    calculateAssistedDirection(player, aimDirection) {
        // Find best target in cone
        const target = this.findBestTarget(player, aimDirection);

        if (!target) {
            return aimDirection; // No assist available
        }

        // Calculate direction to target
        const toTarget = (target.position - player.position).normalized;

        // Check if target is within assist cone
        const angle = Vector3.Angle(aimDirection, toTarget);
        if (angle > ASSIST_CONE_ANGLE) {
            return aimDirection; // Outside cone
        }

        // Blend aim direction toward target
        const assistFactor = ASSIST_STRENGTH * (1 - angle / ASSIST_CONE_ANGLE);
        const assistedDirection = Vector3.Lerp(
            aimDirection,
            toTarget,
            assistFactor
        );

        return assistedDirection.normalized;
    }

    findBestTarget(player, aimDirection) {
        const enemies = this.getEnemiesInRange(player, ASSIST_RANGE);

        let bestTarget = null;
        let bestScore = 0;

        for (const enemy of enemies) {
            // Score based on: angle to crosshair, distance, visibility
            const toEnemy = (enemy.position - player.position).normalized;
            const angle = Vector3.Angle(aimDirection, toEnemy);
            const distance = Vector3.Distance(player.position, enemy.position);

            // Skip if outside cone
            if (angle > ASSIST_CONE_ANGLE) continue;

            // Skip if not visible (behind cover)
            if (!this.hasLineOfSight(player, enemy)) continue;

            // Calculate score (lower angle = better, closer = better)
            const angleScore = 1 - (angle / ASSIST_CONE_ANGLE);
            const distanceScore = 1 - (distance / ASSIST_RANGE);
            const score = angleScore * 0.7 + distanceScore * 0.3;

            if (score > bestScore) {
                bestScore = score;
                bestTarget = enemy;
            }
        }

        return bestTarget;
    }
}
```

### 4.3 Aim Assist Settings

| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| Enable | On | On/Off | Master toggle |
| Strength | Medium | Low/Med/High | Assist intensity |
| Sticky Aim | On | On/Off | Slow turn on target |

### 4.4 Sticky Aim (Target Slowdown)

```javascript
class StickyAim {
    static SLOWDOWN_FACTOR = 0.7;  // 30% slower turn
    static ACTIVATION_ANGLE = 15;  // Degrees

    applySlowdown(player, aimTarget) {
        if (!aimTarget) return 1.0; // Normal speed

        const toTarget = (aimTarget.position - player.position).normalized;
        const angle = Vector3.Angle(player.aimDirection, toTarget);

        if (angle < ACTIVATION_ANGLE) {
            // On target - slow down turn speed
            return SLOWDOWN_FACTOR;
        }

        return 1.0; // Normal speed
    }
}
```

---

## 5. Manual Aim Mode

### 5.1 Skilled Player Options

```javascript
class ManualAimMode {
    // Players can disable aim assist for pure skill
    static SETTINGS = {
        aimAssistOff: false,      // Completely disable assist
        reducedAssist: false,     // 50% assist strength
        crosshairOnly: false,     // No auto-tracking
        advancedReticle: true     // Show projectile lead indicator
    };
}
```

### 5.2 Lead Indicator

```
┌─────────────────────────────────────────────────────────────────┐
│                    LEAD INDICATOR SYSTEM                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For moving targets, show where to aim:                          │
│                                                                  │
│                 ○ (lead indicator - aim here)                   │
│                ╱                                                 │
│               ╱                                                  │
│  [Player] ───╱                                                   │
│              ↘                                                   │
│               ↘                                                  │
│         [Enemy] ─────► (moving right)                           │
│                                                                  │
│  Lead indicator accounts for:                                    │
│  - Enemy velocity                                                │
│  - Projectile travel time                                        │
│  - Distance to target                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Fire Rate System

### 6.1 Cooldown Implementation

```javascript
class BasicAttackCooldown {
    static COOLDOWN_DURATION = 0.8; // Seconds between shots

    constructor() {
        this.cooldownRemaining = 0;
        this.lastFireTime = 0;
    }

    canFire() {
        return this.cooldownRemaining <= 0;
    }

    fire() {
        if (!this.canFire()) {
            this.showCooldownFeedback();
            return false;
        }

        this.cooldownRemaining = COOLDOWN_DURATION;
        this.lastFireTime = Time.now();
        return true;
    }

    update(deltaTime) {
        if (this.cooldownRemaining > 0) {
            this.cooldownRemaining -= deltaTime;
        }
    }

    showCooldownFeedback() {
        // Brief UI flash indicating not ready
        UI.flashCooldown('basic_attack');
        Audio.play('cooldown_click', { volume: 0.2 });
    }
}
```

### 6.2 Anti-Spam Protection

```javascript
// Prevent input spam faster than server tick rate
class InputRateLimiter {
    static MIN_FIRE_INTERVAL = 0.05; // 50ms minimum between inputs

    constructor() {
        this.lastInputTime = 0;
    }

    validateInput() {
        const now = Time.now();
        if (now - this.lastInputTime < MIN_FIRE_INTERVAL) {
            return false; // Too fast, likely spam
        }
        this.lastInputTime = now;
        return true;
    }
}
```

---

## 7. Projectile System

### 7.1 Projectile Properties

```javascript
class BasicAttackProjectile {
    static SPEED = 50;           // m/s
    static SIZE = 0.15;          // Hitbox radius
    static LIFETIME = 0.5;       // Seconds (max travel time)
    static DAMAGE = 15;

    constructor(origin, direction, owner) {
        this.position = origin;
        this.direction = direction;
        this.owner = owner;
        this.velocity = direction * SPEED;
        this.spawnTime = Time.now();
    }

    update(deltaTime) {
        // Move projectile
        this.position += this.velocity * deltaTime;

        // Check for hits
        const hit = Physics.sphereCast(
            this.position,
            this.direction,
            SIZE,
            SPEED * deltaTime
        );

        if (hit && hit.entity !== this.owner) {
            this.onHit(hit);
            return true; // Destroy projectile
        }

        // Check lifetime
        if (Time.now() - this.spawnTime > LIFETIME) {
            return true; // Destroy projectile
        }

        return false; // Keep alive
    }

    onHit(hitInfo) {
        // Apply damage
        if (hitInfo.entity.takeDamage) {
            hitInfo.entity.takeDamage(DAMAGE, this.owner, 'BASIC');
        }

        // Spawn impact VFX
        VFX.spawn('basic_attack_hit', hitInfo.point);
        Audio.play('basic_hit', hitInfo.point);

        // Hit marker for shooter
        this.owner.showHitMarker();
    }
}
```

### 7.2 Projectile Visuals

| Element | Description |
|---------|-------------|
| Shape | Small energy bolt / bullet |
| Color | Cyan/white glow |
| Trail | Short glowing trail (0.2s) |
| Size | ~0.3m visible, 0.15m hitbox |

---

## 8. Visual Feedback

### 8.1 Crosshair

```
┌─────────────────────────────────────────────────────────────────┐
│                    CROSSHAIR SYSTEM                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Default Crosshair:          On Enemy:           On Cooldown:    │
│                                                                  │
│       ┌─┐                      ┌─┐                   ┌─┐        │
│       │ │                      │ │ (red)             │█│        │
│    ───┘ └───                ───┘ └───             ───┘ └───     │
│                                                                  │
│    ───┐ ┌───                ───┐ ┌───             ───┐ ┌───     │
│       │ │                      │ │                   │█│        │
│       └─┘                      └─┘                   └─┘        │
│                                                                  │
│  White (neutral)        Red (enemy targeted)    Gray (cooling)   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Hit Markers

```javascript
class HitMarker {
    static DURATION = 0.2;
    static SIZE_NORMAL = 20;     // px
    static SIZE_CRIT = 30;       // px

    show(hitType) {
        const marker = {
            type: hitType,  // 'NORMAL', 'HEADSHOT', 'KILL'
            position: screenCenter,
            size: hitType === 'HEADSHOT' ? SIZE_CRIT : SIZE_NORMAL,
            color: hitType === 'KILL' ? 'red' : 'white',
            animation: 'scale_fade'
        };

        UI.showHitMarker(marker, DURATION);
        Audio.play('hit_marker', { pitch: hitType === 'HEADSHOT' ? 1.2 : 1.0 });
    }
}
```

### 8.3 Muzzle Flash

```javascript
class MuzzleFlash {
    static DURATION = 0.08;

    show(player) {
        // Spawn flash at player's weapon position
        VFX.spawn('muzzle_flash', player.weaponPosition, {
            rotation: player.aimDirection,
            duration: DURATION,
            scale: 0.5
        });
    }
}
```

---

## 9. Audio Design

### 9.1 Sound Effects

| Sound | Trigger | Properties |
|-------|---------|------------|
| Fire | On shot | Short, punchy |
| Hit | Damage dealt | Satisfying thwack |
| Miss | Projectile expires | Subtle whoosh |
| Cooldown | Fire while on CD | Click/denied |
| Kill | Elimination with basic | Special chime |

### 9.2 Audio Implementation

```javascript
class BasicAttackAudio {
    static SOUNDS = {
        fire: 'basic_fire',
        hit: 'basic_hit',
        miss: 'basic_miss',
        cooldown: 'ui_denied',
        kill: 'basic_kill'
    };

    playFire() {
        Audio.play(SOUNDS.fire, {
            volume: 0.7,
            pitch: randomRange(0.95, 1.05), // Slight variation
            position: this.player.position
        });
    }

    playHit(isKill) {
        const sound = isKill ? SOUNDS.kill : SOUNDS.hit;
        Audio.play(sound, {
            volume: isKill ? 0.9 : 0.6
        });
    }
}
```

---

## 10. Network Synchronization

### 10.1 Fire Packet

```javascript
struct BasicAttackPacket {
    uint32 playerId;
    float3 origin;           // Fire position
    float3 direction;        // Aim direction
    float  timestamp;        // Server time
    uint8  sequenceNumber;   // For ordering
}
```

### 10.2 Hit Validation

```javascript
class ServerHitValidation {
    validateHit(packet, target) {
        // 1. Check fire rate
        if (!this.validateFireRate(packet.playerId)) {
            return { valid: false, reason: 'FIRE_RATE' };
        }

        // 2. Check range
        const distance = Vector3.Distance(packet.origin, target.position);
        if (distance > BasicAttackProjectile.SPEED * BasicAttackProjectile.LIFETIME * 1.1) {
            return { valid: false, reason: 'OUT_OF_RANGE' };
        }

        // 3. Check line of sight
        if (!Physics.raycast(packet.origin, target.position)) {
            return { valid: false, reason: 'NO_LOS' };
        }

        // 4. Lag compensation check
        const rewindTime = this.calculateRewindTime(packet.timestamp);
        const rewindedTargetPos = target.getPositionAtTime(rewindTime);

        // Validate hit against rewinded position
        return { valid: true };
    }
}
```

### 10.3 Client Prediction

```javascript
class ClientPrediction {
    predictShot(origin, direction) {
        // Immediately show:
        // - Muzzle flash
        // - Projectile visual
        // - Sound effect

        // Send to server
        this.sendFirePacket(origin, direction);

        // Predict hit locally
        const hit = this.localRaycast(origin, direction);
        if (hit) {
            // Show hit marker immediately
            this.showHitMarker();
            // Wait for server confirmation for damage
        }
    }

    onServerResponse(response) {
        if (!response.hitConfirmed && this.predictedHit) {
            // Server rejected hit - hide marker
            this.hideHitMarker();
        }
    }
}
```

---

## 11. Balance Considerations

### 11.1 DPS Analysis

| Scenario | DPS | Notes |
|----------|-----|-------|
| Perfect accuracy | 18.75 | 15 dmg / 0.8s |
| 50% accuracy | 9.375 | Realistic |
| With abilities | ~5-10 | Basic as filler |

### 11.2 Time to Kill (TTK)

| Target Health | Shots | Time |
|---------------|-------|------|
| 100 HP | 7 | 5.6s |
| 150 HP | 10 | 8.0s |
| 200 HP | 14 | 11.2s |

### 11.3 Design Intent

- Basic attack is **reliable but not dominant**
- Encourages finding abilities for power spikes
- Provides consistent damage during cooldowns
- Rewards aim skill without aim assist dependency

---

## 12. Testing Requirements

### 12.1 Unit Tests

| Test | Description |
|------|-------------|
| Damage | Exactly 15 damage per hit |
| Cooldown | 0.8s enforced accurately |
| Range | Projectile despawns at correct range |
| Aim assist | Correct target acquisition |

### 12.2 Integration Tests

| Test | Description |
|------|-------------|
| Network sync | Hits confirmed server-side |
| Input latency | < 50ms from tap to visual |
| Aim assist balance | Not OP, not useless |
| Mobile controls | Touch controls responsive |

### 12.3 Playtest Validation

| Criteria | Target |
|----------|--------|
| Aim assist satisfaction | > 80% find it helpful |
| Skilled player opt-out | Manual mode viable |
| Basic attack usage | 30-40% of total damage |

---

## 13. Appendices

### Appendix A: Control Presets

| Preset | Fire | Aim Assist | Best For |
|--------|------|------------|----------|
| Casual | Tap + Hold | Strong | New players |
| Balanced | Tap + Hold | Medium | Most players |
| Competitive | Tap only | Weak/Off | Skilled players |

### Appendix B: Aim Assist Strength Values

| Setting | Cone Angle | Blend Factor |
|---------|------------|--------------|
| Low | 20° | 0.25 |
| Medium | 30° | 0.40 |
| High | 40° | 0.55 |
| Off | 0° | 0 |

### Appendix C: Performance Budget

| Operation | Budget |
|-----------|--------|
| Aim assist calculation | < 0.5ms |
| Projectile update | < 0.1ms |
| Hit detection | < 0.2ms |
| VFX spawn | < 0.5ms |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
