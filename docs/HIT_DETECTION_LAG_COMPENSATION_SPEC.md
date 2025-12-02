# Hit Detection and Lag Compensation Specification

## Document Information
- **Task ID:** GAME-013
- **Priority:** P0
- **Complexity:** L
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the hit detection and lag compensation system for Plunderstorm Mobile. The system ensures fair combat regardless of network latency while preventing cheating and exploits.

### 1.2 Scope
- Server-authoritative hit detection
- Client-side prediction
- Lag compensation (server rewind)
- Projectile simulation
- AoE hit detection
- Anti-cheat validation
- Mobile network considerations

### 1.3 Dependencies
- BACK-003: Game Server Infrastructure
- BACK-004: Network Protocol

---

## 2. System Architecture

### 2.1 Authority Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    AUTHORITY MODEL                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐         ┌─────────────────┐                │
│  │    CLIENT A     │         │     SERVER      │                │
│  │                 │         │                 │                │
│  │  - Input        │ ──────► │  - AUTHORITATIVE│                │
│  │  - Prediction   │         │  - Hit detection│                │
│  │  - Visual only  │ ◄────── │  - Damage calc  │                │
│  │                 │         │  - State sync   │                │
│  └─────────────────┘         └─────────────────┘                │
│                                      │                           │
│                                      │                           │
│                                      ▼                           │
│                              ┌─────────────────┐                │
│                              │    CLIENT B     │                │
│                              │                 │                │
│                              │  - Receives     │                │
│                              │    confirmed    │                │
│                              │    hits         │                │
│                              └─────────────────┘                │
│                                                                  │
│  KEY PRINCIPLE: Client shows prediction, Server decides truth    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Hit Detection Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    HIT DETECTION FLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CLIENT SIDE:                                                    │
│  ┌──────────────┐                                               │
│  │ 1. Player    │  Player presses attack button                 │
│  │    Input     │                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 2. Client    │  Show muzzle flash, projectile                │
│  │    Prediction│  Play sound immediately                       │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 3. Send to   │  Attack packet with timestamp                 │
│  │    Server    │  position, direction, target                  │
│  └──────────────┘                                               │
│                                                                  │
│  SERVER SIDE:                                                    │
│  ┌──────────────┐                                               │
│  │ 4. Receive   │  Validate packet integrity                    │
│  │    & Validate│                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 5. Lag       │  Rewind world state to client's              │
│  │    Compensation│ view time                                   │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 6. Hit       │  Check collision against rewound state        │
│  │    Detection │                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 7. Broadcast │  Send hit confirmation to all clients         │
│  │    Result    │                                               │
│  └──────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Lag Compensation

### 3.1 Server Rewind System

```javascript
class LagCompensation {
    static MAX_REWIND_TIME = 200;     // ms, max compensation
    static HISTORY_DURATION = 500;    // ms, state buffer size
    static HISTORY_TICK_RATE = 20;    // ticks per second

    constructor() {
        this.stateHistory = new RingBuffer(
            HISTORY_DURATION / (1000 / HISTORY_TICK_RATE)
        );
    }

    // Store world state every tick
    recordState(timestamp) {
        const state = {
            timestamp: timestamp,
            players: this.capturePlayerStates(),
            projectiles: this.captureProjectileStates()
        };
        this.stateHistory.push(state);
    }

    capturePlayerStates() {
        return players.map(p => ({
            id: p.id,
            position: p.position.clone(),
            rotation: p.rotation.clone(),
            hitboxes: p.getHitboxes()
        }));
    }

    // Rewind to client's view time
    getStateAtTime(clientTime) {
        // Find surrounding states for interpolation
        const states = this.stateHistory.getAll();

        let before = null;
        let after = null;

        for (let i = 0; i < states.length - 1; i++) {
            if (states[i].timestamp <= clientTime &&
                states[i + 1].timestamp > clientTime) {
                before = states[i];
                after = states[i + 1];
                break;
            }
        }

        if (!before || !after) {
            // Client time too old or too new
            return this.clampToValidState(clientTime);
        }

        // Interpolate between states
        const t = (clientTime - before.timestamp) /
                  (after.timestamp - before.timestamp);

        return this.interpolateStates(before, after, t);
    }

    interpolateStates(before, after, t) {
        const result = { players: [], projectiles: [] };

        for (const beforePlayer of before.players) {
            const afterPlayer = after.players.find(p => p.id === beforePlayer.id);
            if (!afterPlayer) continue;

            result.players.push({
                id: beforePlayer.id,
                position: Vector3.Lerp(beforePlayer.position, afterPlayer.position, t),
                rotation: Quaternion.Slerp(beforePlayer.rotation, afterPlayer.rotation, t),
                hitboxes: beforePlayer.hitboxes // Use before state for hitboxes
            });
        }

        return result;
    }
}
```

### 3.2 Rewind Limits

| Parameter | Value | Reason |
|-----------|-------|--------|
| Max Rewind | 200ms | Prevents extreme lag advantage |
| Min Latency | 0ms | No artificial delay |
| Buffer Size | 500ms | Covers reconnection scenarios |
| Tick Rate | 20 Hz | Balance between precision and memory |

### 3.3 Client Time Calculation

```javascript
class ClientTimeSynchronization {
    // Calculate client's perceived time for hit detection
    calculateClientTime(packet) {
        // Server receives packet at serverReceiveTime
        // Packet was sent at packet.clientSendTime
        // Client's "view time" was earlier due to network delay

        const estimatedRTT = this.getPlayerRTT(packet.playerId);
        const oneWayLatency = estimatedRTT / 2;

        // Client saw the world at this server time
        const clientViewTime = packet.serverReceiveTime - oneWayLatency;

        // Clamp to valid range
        const maxRewindTime = Time.serverNow() - LagCompensation.MAX_REWIND_TIME;
        return Math.max(clientViewTime, maxRewindTime);
    }
}
```

---

## 4. Hit Detection Types

### 4.1 Projectile Hit Detection

```javascript
class ProjectileHitDetection {
    // Server-side projectile simulation
    simulateProjectile(origin, direction, speed, maxDistance, clientTime) {
        // Get world state at client's view time
        const worldState = lagCompensation.getStateAtTime(clientTime);

        // Simulate projectile path
        let position = origin.clone();
        const velocity = direction.normalized.multiply(speed);
        const stepTime = 0.01; // 10ms steps
        let distance = 0;

        while (distance < maxDistance) {
            // Move projectile
            const step = velocity.multiply(stepTime);
            const newPosition = position.add(step);

            // Check for collisions along this step
            const hit = this.raycastAgainstState(
                position,
                newPosition,
                worldState
            );

            if (hit) {
                return {
                    hit: true,
                    target: hit.entity,
                    hitPoint: hit.point,
                    distance: distance + hit.distance
                };
            }

            position = newPosition;
            distance += step.magnitude;
        }

        return { hit: false };
    }

    raycastAgainstState(start, end, worldState) {
        for (const player of worldState.players) {
            // Check against player's hitboxes
            for (const hitbox of player.hitboxes) {
                const hit = this.raycastCapsule(start, end, hitbox);
                if (hit) {
                    return {
                        entity: player,
                        point: hit.point,
                        distance: hit.distance
                    };
                }
            }
        }

        // Check against world geometry (no rewind needed)
        return this.raycastWorld(start, end);
    }
}
```

### 4.2 Hitscan Detection

```javascript
class HitscanHitDetection {
    // For instant-hit weapons (if any)
    detectHitscan(origin, direction, maxRange, shooter, clientTime) {
        const worldState = lagCompensation.getStateAtTime(clientTime);
        const endpoint = origin.add(direction.multiply(maxRange));

        // Sort potential targets by distance
        const potentialHits = [];

        for (const player of worldState.players) {
            if (player.id === shooter.id) continue; // Can't hit self

            for (const hitbox of player.hitboxes) {
                const intersection = this.lineIntersectsCapsule(
                    origin, endpoint, hitbox
                );

                if (intersection) {
                    potentialHits.push({
                        player: player,
                        hitbox: hitbox,
                        distance: intersection.distance,
                        point: intersection.point
                    });
                }
            }
        }

        // Check world geometry
        const worldHit = Physics.raycast(origin, direction, maxRange);

        // Return closest hit
        potentialHits.sort((a, b) => a.distance - b.distance);

        if (potentialHits.length > 0) {
            const closest = potentialHits[0];
            // Verify world geometry doesn't block
            if (!worldHit || worldHit.distance > closest.distance) {
                return closest;
            }
        }

        return null;
    }
}
```

### 4.3 AoE Hit Detection

```javascript
class AoEHitDetection {
    // For area-of-effect abilities
    detectAoE(center, radius, caster, timestamp) {
        // AoE uses current server state, not rewound
        // (Players have warning indicator to dodge)

        const hits = [];
        const players = this.getPlayersInRadius(center, radius);

        for (const player of players) {
            if (player.id === caster.id) continue; // Can't hit self (usually)

            // Check if player's center is in AoE
            const distance = Vector3.Distance(center, player.position);
            if (distance <= radius) {
                hits.push({
                    player: player,
                    distance: distance,
                    // Damage falloff based on distance (optional)
                    falloff: 1 - (distance / radius) * 0.3
                });
            }
        }

        return hits;
    }
}
```

### 4.4 Melee Arc Detection

```javascript
class MeleeArcDetection {
    detectMeleeArc(origin, direction, range, arcAngle, attacker, clientTime) {
        const worldState = lagCompensation.getStateAtTime(clientTime);
        const halfArc = arcAngle / 2;
        const hits = [];

        for (const player of worldState.players) {
            if (player.id === attacker.id) continue;

            // Check distance
            const toPlayer = player.position.subtract(origin);
            const distance = toPlayer.magnitude;

            if (distance > range) continue;

            // Check angle
            const angle = Vector3.Angle(direction, toPlayer.normalized);
            if (angle > halfArc) continue;

            // Check line of sight (optional for melee)
            const hasLOS = !Physics.raycast(origin, toPlayer.normalized, distance);

            if (hasLOS) {
                hits.push({
                    player: player,
                    distance: distance
                });
            }
        }

        return hits;
    }
}
```

---

## 5. Client-Side Prediction

### 5.1 Prediction System

```javascript
class ClientPrediction {
    predictAttack(attackData) {
        // Immediately show:
        // 1. Muzzle flash / cast animation
        this.showAttackVisual(attackData);

        // 2. Sound effect
        this.playAttackSound(attackData);

        // 3. Projectile (visual only)
        if (attackData.hasProjectile) {
            const visualProjectile = this.spawnVisualProjectile(attackData);
            this.predictedProjectiles.set(attackData.sequenceId, visualProjectile);
        }

        // 4. Predicted hit (if applicable)
        const predictedHit = this.localHitCheck(attackData);
        if (predictedHit) {
            // Show hit marker speculatively
            this.showPredictedHitMarker(predictedHit);
            this.predictedHits.set(attackData.sequenceId, predictedHit);
        }

        // Send to server
        this.sendAttackToServer(attackData);
    }

    onServerConfirmation(sequenceId, result) {
        // Remove predicted state
        const predictedHit = this.predictedHits.get(sequenceId);
        const predictedProjectile = this.predictedProjectiles.get(sequenceId);

        if (result.confirmed) {
            // Server confirmed hit - keep visual
            if (predictedProjectile) {
                // Replace visual projectile with confirmed
                predictedProjectile.confirm();
            }
        } else {
            // Server rejected - hide prediction
            if (predictedHit) {
                this.hidePredictedHitMarker();
            }
            if (predictedProjectile && !result.hit) {
                // Projectile missed - show miss instead
                predictedProjectile.showMiss();
            }
        }

        this.predictedHits.delete(sequenceId);
        this.predictedProjectiles.delete(sequenceId);
    }
}
```

### 5.2 Hit Marker Prediction

```javascript
class HitMarkerPrediction {
    showPredictedHitMarker(hit) {
        // Show hit marker immediately
        this.hitMarker = UI.showHitMarker({
            style: 'predicted', // Slightly different style
            duration: 0.5
        });

        // Start timeout - if no confirmation, hide
        this.confirmationTimeout = setTimeout(() => {
            if (!this.confirmed) {
                this.hidePredictedHitMarker();
            }
        }, 150); // 150ms grace period
    }

    onConfirmation(confirmed) {
        clearTimeout(this.confirmationTimeout);
        this.confirmed = true;

        if (confirmed) {
            // Upgrade to confirmed hit marker
            this.hitMarker.setStyle('confirmed');
        } else {
            this.hidePredictedHitMarker();
        }
    }
}
```

---

## 6. Server Validation

### 6.1 Hit Validation Checks

```javascript
class HitValidator {
    validateHit(attackPacket, claimedHit) {
        const checks = [
            this.validateAttackerState(attackPacket),
            this.validateTiming(attackPacket),
            this.validateRange(attackPacket, claimedHit),
            this.validateLineOfSight(attackPacket, claimedHit),
            this.validateTargetState(claimedHit),
            this.validateAbilityCooldown(attackPacket),
            this.validateRateLimit(attackPacket)
        ];

        for (const check of checks) {
            if (!check.valid) {
                this.logRejection(attackPacket, check.reason);
                return { valid: false, reason: check.reason };
            }
        }

        return { valid: true };
    }

    validateAttackerState(packet) {
        const attacker = this.getPlayer(packet.attackerId);

        if (!attacker || attacker.state !== PlayerState.ALIVE) {
            return { valid: false, reason: 'ATTACKER_NOT_ALIVE' };
        }

        if (attacker.hasCC('STUN') || attacker.hasCC('SILENCE')) {
            return { valid: false, reason: 'ATTACKER_CC' };
        }

        return { valid: true };
    }

    validateTiming(packet) {
        const serverTime = Time.serverNow();
        const packetAge = serverTime - packet.timestamp;

        if (packetAge > LagCompensation.MAX_REWIND_TIME + 50) {
            return { valid: false, reason: 'PACKET_TOO_OLD' };
        }

        if (packetAge < -50) { // 50ms tolerance for clock skew
            return { valid: false, reason: 'PACKET_FROM_FUTURE' };
        }

        return { valid: true };
    }

    validateRange(packet, hit) {
        const attacker = this.getPlayer(packet.attackerId);
        const ability = this.getAbility(packet.abilityId);

        const distance = Vector3.Distance(
            packet.attackOrigin,
            hit.hitPoint
        );

        // Allow 10% tolerance for network jitter
        if (distance > ability.range * 1.1) {
            return { valid: false, reason: 'OUT_OF_RANGE' };
        }

        return { valid: true };
    }

    validateLineOfSight(packet, hit) {
        const origin = packet.attackOrigin;
        const target = hit.hitPoint;

        // Raycast against world geometry
        const obstruction = Physics.raycast(origin, target);

        if (obstruction && obstruction.distance < Vector3.Distance(origin, target)) {
            return { valid: false, reason: 'LINE_OF_SIGHT_BLOCKED' };
        }

        return { valid: true };
    }
}
```

### 6.2 Rate Limiting

```javascript
class AttackRateLimiter {
    static MAX_ATTACKS_PER_SECOND = 5;
    static COOLDOWN_TOLERANCE = 50; // ms

    validateRateLimit(packet) {
        const attacker = this.getPlayer(packet.attackerId);
        const now = Time.serverNow();

        // Check overall attack rate
        const recentAttacks = attacker.attackHistory.filter(
            a => now - a.timestamp < 1000
        );

        if (recentAttacks.length >= MAX_ATTACKS_PER_SECOND) {
            this.flagSuspicious(attacker, 'ATTACK_RATE_EXCEEDED');
            return { valid: false, reason: 'RATE_LIMITED' };
        }

        // Check ability-specific cooldown
        const ability = this.getAbility(packet.abilityId);
        const lastUse = attacker.abilityLastUse[ability.id];

        if (lastUse) {
            const timeSinceUse = now - lastUse;
            const expectedCooldown = ability.cooldown * 1000;

            if (timeSinceUse < expectedCooldown - COOLDOWN_TOLERANCE) {
                this.flagSuspicious(attacker, 'COOLDOWN_VIOLATION');
                return { valid: false, reason: 'ABILITY_ON_COOLDOWN' };
            }
        }

        // Record attack
        attacker.attackHistory.push({ timestamp: now, abilityId: ability.id });
        attacker.abilityLastUse[ability.id] = now;

        return { valid: true };
    }
}
```

---

## 7. Anti-Cheat Measures

### 7.1 Cheat Detection

```javascript
class CheatDetector {
    static SUSPICION_THRESHOLDS = {
        PERFECT_AIM: 10,        // Consecutive perfect hits
        IMPOSSIBLE_REACTION: 5, // Reaction time < 100ms
        POSITION_MISMATCH: 3,   // Client/server position diff > 10m
        RATE_VIOLATION: 3       // Attack rate exceeds limits
    };

    checkForCheats(player, attackData, result) {
        // Perfect aim detection
        if (result.hit && result.isHeadshot) {
            player.perfectHitStreak++;
            if (player.perfectHitStreak >= SUSPICION_THRESHOLDS.PERFECT_AIM) {
                this.flagForReview(player, 'AIMBOT_SUSPECTED');
            }
        } else if (!result.hit) {
            player.perfectHitStreak = 0;
        }

        // Position mismatch
        const clientPos = attackData.clientPosition;
        const serverPos = player.position;
        const mismatch = Vector3.Distance(clientPos, serverPos);

        if (mismatch > 10) { // 10 meter discrepancy
            player.positionMismatchCount++;
            if (player.positionMismatchCount >= SUSPICION_THRESHOLDS.POSITION_MISMATCH) {
                this.flagForReview(player, 'POSITION_HACK_SUSPECTED');
            }
        }

        // Speed hack detection
        const timeSinceLastPos = Time.serverNow() - player.lastPositionTime;
        const distanceMoved = Vector3.Distance(player.position, player.lastPosition);
        const speed = distanceMoved / (timeSinceLastPos / 1000);

        if (speed > player.maxPossibleSpeed * 1.5) {
            this.flagForReview(player, 'SPEED_HACK_SUSPECTED');
        }
    }

    flagForReview(player, reason) {
        // Log for manual review
        this.logSuspicion(player, reason);

        // Optionally: soft-ban (shadow realm)
        // Don't immediately kick - might be false positive
    }
}
```

### 7.2 Secure Data Flow

```javascript
// What clients can trust
const CLIENT_AUTHORITATIVE = {
    inputDirection: true,     // Where player wants to aim
    inputMovement: true,      // Where player wants to move
    abilityUse: true          // Which ability to use
};

// What server controls
const SERVER_AUTHORITATIVE = {
    hitDetection: true,       // Did hit actually happen
    damageAmount: true,       // How much damage
    playerPositions: true,    // Where players actually are
    abilityAvailability: true,// Can ability be used
    healthValues: true        // Current HP
};
```

---

## 8. Mobile Network Considerations

### 8.1 Network Conditions

| Condition | Expected Latency | Handling |
|-----------|------------------|----------|
| WiFi | 20-50ms | Standard |
| 4G | 50-100ms | Standard + prediction |
| 3G | 100-300ms | Aggressive prediction |
| Packet Loss | Variable | Redundancy |

### 8.2 Connection Quality Adaptation

```javascript
class NetworkAdaptation {
    adaptToQuality(player, metrics) {
        const { latency, packetLoss, jitter } = metrics;

        if (latency > 150) {
            // High latency - increase prediction
            player.predictionSettings.enabled = true;
            player.predictionSettings.aggressiveness = 'HIGH';
        }

        if (packetLoss > 0.05) {
            // 5%+ packet loss - enable redundancy
            player.networkSettings.redundantPackets = true;
        }

        if (jitter > 50) {
            // High jitter - increase input buffer
            player.networkSettings.inputBufferSize = 3;
        }
    }
}
```

### 8.3 Graceful Degradation

```javascript
class GracefulDegradation {
    handlePoorConnection(player) {
        // Visual indicator for player
        UI.showConnectionWarning(player.latency);

        // Reduce update rate to save bandwidth
        player.updateRate = Math.min(player.updateRate, 10);

        // Allow more lag compensation (up to limit)
        player.maxRewindAllowed = Math.min(
            player.latency * 1.5,
            LagCompensation.MAX_REWIND_TIME
        );

        // Disable cosmetic effects to reduce bandwidth
        player.cosmeticEffects = false;
    }
}
```

---

## 9. Hit Confirmation Protocol

### 9.1 Packet Structures

```javascript
// Client → Server: Attack Request
struct AttackPacket {
    uint32 playerId;
    uint32 sequenceId;       // For matching response
    uint8  abilityId;
    float3 origin;           // Attack origin position
    float3 direction;        // Attack direction
    float  clientTime;       // Client's perceived time
    uint32 targetId;         // Optional: predicted target
}

// Server → Client: Attack Result
struct AttackResultPacket {
    uint32 sequenceId;       // Matches request
    uint8  result;           // HIT, MISS, BLOCKED
    uint32 targetId;         // Who was hit (if any)
    uint16 damage;           // Damage dealt
    uint8  flags;            // CRIT, HEADSHOT, etc.
    float3 hitPoint;         // Where hit landed
}

// Server → All: Damage Event
struct DamageEventPacket {
    uint32 attackerId;
    uint32 targetId;
    uint16 damage;
    uint8  damageType;
    float3 hitPoint;
    uint8  flags;
}
```

### 9.2 Confirmation Flow

```
Client A                Server                Client B
    │                     │                      │
    │──Attack Request────►│                      │
    │   (fire projectile) │                      │
    │                     │                      │
    │                     │──Rewind & Detect───  │
    │                     │                      │
    │◄──Attack Result─────│                      │
    │   (hit confirmed)   │                      │
    │                     │                      │
    │                     │──Damage Event───────►│
    │                     │   (you got hit)      │
    │                     │                      │
```

---

## 10. Edge Cases

### 10.1 Simultaneous Hits

```javascript
class SimultaneousHitHandler {
    handleSimultaneousHits(hit1, hit2) {
        // Both players hit each other at same server tick

        // Process in timestamp order
        const orderedHits = [hit1, hit2].sort(
            (a, b) => a.clientTime - b.clientTime
        );

        // Both hits are valid
        for (const hit of orderedHits) {
            this.processHit(hit);
        }

        // Both players take damage
        // (no "first hit wins" - both are valid)
    }
}
```

### 10.2 Disconnection During Attack

```javascript
class DisconnectionHandler {
    handleAttackerDisconnect(attackPacket) {
        // Attack was already fired - process normally
        // (projectile keeps flying)
        return this.processAttack(attackPacket);
    }

    handleTargetDisconnect(hit) {
        // Target disconnected during hit
        // Damage still counts, target eliminated
        return this.applyDamage(hit);
    }
}
```

### 10.3 Extreme Lag Spikes

```javascript
class LagSpikeHandler {
    handleLagSpike(player, spike) {
        if (spike.duration > 2000) { // 2 second spike
            // Void all attacks during spike
            player.attacksDuringSpike.forEach(attack => {
                this.voidAttack(attack);
            });

            // Teleport player to server position
            player.forcePositionSync();
        }
    }
}
```

---

## 11. Performance Optimization

### 11.1 Performance Targets

| Operation | Budget | Notes |
|-----------|--------|-------|
| State Storage | < 10ms/tick | Recording history |
| State Rewind | < 2ms | Retrieving past state |
| Hit Detection | < 1ms | Per projectile |
| Validation | < 0.5ms | Per attack |
| Total Tick | < 16ms | 60 Hz server |

### 11.2 Optimization Strategies

```javascript
class HitDetectionOptimizer {
    // Spatial partitioning for faster queries
    spatialGrid = new SpatialHashGrid(50); // 50 unit cells

    // Object pooling for projectiles
    projectilePool = new ObjectPool(Projectile, 100);

    // Batch hit detection
    processBatchedHits(attacks) {
        // Group attacks by region
        const regionAttacks = this.groupByRegion(attacks);

        // Process each region in parallel (if multi-threaded)
        for (const [region, regionAttacks] of regionAttacks) {
            this.processRegionHits(region, regionAttacks);
        }
    }

    // Early-out checks
    canHit(attacker, target) {
        // Quick distance check before detailed hit detection
        const dist = Vector3.DistanceSquared(attacker.position, target.position);
        const maxRange = attacker.weapon.maxRange * 1.5; // Buffer

        return dist <= maxRange * maxRange;
    }
}
```

---

## 12. Testing Requirements

### 12.1 Unit Tests

| Test | Description |
|------|-------------|
| Lag compensation | Correct state at any past time |
| Hit detection | Accurate collision detection |
| Validation | Catches all invalid attacks |
| Rate limiting | Enforces attack limits |

### 12.2 Integration Tests

| Test | Description |
|------|-------------|
| High latency | 200ms+ latency works |
| Packet loss | 10% loss handled |
| 60 players | Performance at scale |
| Cheat simulation | Detects obvious cheats |

### 12.3 Stress Tests

| Test | Target |
|------|--------|
| Simultaneous attacks | 60 attacks/tick handled |
| State history | 500ms buffer maintained |
| Validation throughput | 1000 validations/second |

---

## 13. Appendices

### Appendix A: Hitbox Configuration

| Body Part | Priority | Multiplier |
|-----------|----------|------------|
| Head | 1 | 1.5x (headshot) |
| Torso | 2 | 1.0x |
| Arms | 3 | 0.8x |
| Legs | 4 | 0.7x |

### Appendix B: Latency Compensation Values

| Latency | Max Rewind | Prediction |
|---------|------------|------------|
| 0-50ms | 50ms | Low |
| 50-100ms | 100ms | Medium |
| 100-150ms | 150ms | High |
| 150-200ms | 200ms | Maximum |
| 200ms+ | 200ms (capped) | Maximum |

### Appendix C: Network Packet Sizes

| Packet | Size | Frequency |
|--------|------|-----------|
| Attack | 32 bytes | On action |
| Result | 24 bytes | On hit |
| State | 64 bytes | 20 Hz |
| Position | 16 bytes | 30 Hz |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
