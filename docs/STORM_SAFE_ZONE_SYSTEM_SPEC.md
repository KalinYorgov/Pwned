# Storm/Safe Zone System Specification

## Document Information
- **Task ID:** GAME-003
- **Priority:** P0
- **Complexity:** M
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the storm/safe zone system for Plunderstorm Mobile. The shrinking safe zone is a core battle royale mechanic that forces player confrontation and ensures matches conclude within a predictable timeframe.

### 1.2 Scope
- Safe zone shrinking phases and timing
- Storm damage mechanics
- Visual representation of storm wall
- Minimap and HUD indicators
- Audio warning systems
- Network synchronization
- Mobile performance optimization

### 1.3 Dependencies
- GAME-001: Match Initialization System
- ART-015: Storm/Zone VFX
- BACK-003: Game Server Infrastructure

---

## 2. Core Mechanics

### 2.1 Zone Behavior Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    ZONE PHASE PROGRESSION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Match Start                                            Match End│
│      │                                                      │    │
│      ▼                                                      ▼    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Phase 1 │ Phase 2 │ Phase 3 │ Phase 4 │ Phase 5 │ Final  │  │
│  │  80%    │  60%    │  40%    │  20%    │  10%    │  ~0%   │  │
│  │ (2 min) │ (2 min) │ (2 min) │ (1.5min)│ (1 min) │ (30s)  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Zone Coverage (% of map)                                        │
│  100% ────────────────────────────────────────────────────────  │
│   80% ════════╗                                                  │
│   60%         ╚═══════╗                                          │
│   40%                 ╚═══════╗                                  │
│   20%                         ╚═══════╗                          │
│   10%                                 ╚═════╗                    │
│    0%                                       ╚════════            │
│        ───────────────────────────────────────────► Time        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Phase Configuration

| Phase | Safe Zone % | Warning Duration | Shrink Duration | Stable Duration | Storm Damage |
|-------|-------------|------------------|-----------------|-----------------|--------------|
| 1 | 80% → 60% | 30s | 60s | 30s | 2 DPS |
| 2 | 60% → 40% | 25s | 50s | 25s | 4 DPS |
| 3 | 40% → 20% | 20s | 40s | 20s | 6 DPS |
| 4 | 20% → 10% | 15s | 30s | 15s | 8 DPS |
| 5 | 10% → ~0% | 10s | 20s | 0s | 10 DPS |

**Total Match Duration:** 10-15 minutes (target: 12 minutes average)

### 2.3 Zone States

```javascript
enum ZoneState {
    WAITING,      // Initial state, zone not yet active
    STABLE,       // Zone is stationary, next shrink pending
    WARNING,      // Visual/audio warning, shrink incoming
    SHRINKING,    // Zone actively contracting
    FINAL         // Zone at minimum size, high damage
}
```

---

## 3. Zone Positioning

### 3.1 Next Zone Calculation

```javascript
class ZonePositionCalculator {
    calculateNextZone(currentZone, phase, mapBounds, seed) {
        // Next zone center is weighted toward map center
        // but with randomization for variety

        const mapCenter = mapBounds.center;
        const currentCenter = currentZone.center;

        // Calculate new center with bias toward map center
        const centerBias = 0.3; // 30% pull toward map center
        const randomOffset = getSeededRandom(seed, phase);

        // New center within current zone, biased toward map center
        const direction = (mapCenter - currentCenter).normalized;
        const maxOffset = currentZone.radius * 0.4; // Can offset up to 40% of current radius

        const newCenter = currentCenter
            + direction * (maxOffset * centerBias)
            + randomOffset * (maxOffset * (1 - centerBias));

        // Ensure new center is within current zone
        const newRadius = currentZone.radius * this.getRadiusMultiplier(phase);

        // Validate new zone fits within current zone
        return this.validateAndAdjust(newCenter, newRadius, currentZone);
    }

    getRadiusMultiplier(phase) {
        const multipliers = {
            1: 0.75,  // 80% → 60%
            2: 0.67,  // 60% → 40%
            3: 0.50,  // 40% → 20%
            4: 0.50,  // 20% → 10%
            5: 0.30   // 10% → ~3%
        };
        return multipliers[phase] || 0.5;
    }
}
```

### 3.2 Zone Constraints

| Constraint | Description |
|------------|-------------|
| Map Bounds | Zone must remain within playable map |
| POI Inclusion | Final zones should include at least one landmark |
| Water Avoidance | Avoid zones centered on deep water |
| Terrain Accessibility | Final zone must be accessible (not cliff-locked) |
| Fairness | Zone movement shouldn't favor any drop location |

### 3.3 Zone Shape

```
Primary: Circular zones (standard)
┌───────────────────────────────┐
│                               │
│        ╭──────────╮          │
│       ╱            ╲          │
│      │   SAFE ZONE  │         │
│       ╲            ╱          │
│        ╰──────────╯          │
│                               │
│    STORM (damage zone)        │
│                               │
└───────────────────────────────┘

The circular shape simplifies:
- Distance calculations
- Visual representation
- Network synchronization
- Player understanding
```

---

## 4. Storm Damage System

### 4.1 Damage Calculation

```javascript
class StormDamageSystem {
    static DAMAGE_TICK_RATE = 1.0; // Damage applied every 1 second

    calculateDamage(player, zone, currentPhase) {
        // Check if player is outside safe zone
        const distanceToCenter = Vector3.Distance(player.position, zone.center);
        const isInStorm = distanceToCenter > zone.radius;

        if (!isInStorm) {
            return 0;
        }

        // Get phase damage
        const baseDamage = this.getPhaseDamage(currentPhase);

        // Optional: Scale damage by distance outside zone
        const distanceOutside = distanceToCenter - zone.radius;
        const distanceMultiplier = 1.0; // Could scale: 1 + (distanceOutside / 100)

        return baseDamage * distanceMultiplier;
    }

    getPhaseDamage(phase) {
        const damageTable = {
            1: 2,   // 2 DPS
            2: 4,   // 4 DPS
            3: 6,   // 6 DPS
            4: 8,   // 8 DPS
            5: 10   // 10 DPS
        };
        return damageTable[phase] || 10;
    }

    applyDamage(player, damage) {
        // Server-authoritative damage application
        player.health -= damage;

        // Trigger visual feedback
        this.triggerStormDamageFeedback(player);

        // Check for elimination
        if (player.health <= 0) {
            this.handleStormElimination(player);
        }
    }
}
```

### 4.2 Damage Feedback

| Feedback Type | Description |
|---------------|-------------|
| Screen Effect | Red vignette pulse on damage tick |
| Sound | Ominous wind/crackling sound |
| Health Bar | Flash on damage |
| Character VFX | Lightning/storm particles on player |
| Damage Numbers | Show storm damage (different color) |

### 4.3 Storm Elimination

```javascript
class StormElimination {
    handleStormDeath(player) {
        // Record cause of death
        player.deathCause = 'STORM';

        // No killer attribution
        player.killedBy = null;

        // Special death animation
        player.playAnimation('storm_death');

        // Kill feed message
        broadcastKillFeed({
            message: `${player.name} was consumed by the storm`,
            icon: 'storm_skull'
        });

        // No XP/loot drop (storm kills give nothing)
        // Prevents exploits where players farm low-health enemies in storm
    }
}
```

---

## 5. Visual Representation

### 5.1 Storm Wall Appearance

```
┌─────────────────────────────────────────────────────────────────┐
│                    STORM WALL VISUAL LAYERS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Height: 500+ meters (appears to reach sky)                      │
│                                                                  │
│  ╔════════════════════════════════════════════════════════════╗ │
│  ║ OUTER GLOW                                                  ║ │
│  ║   - Soft purple/blue haze                                   ║ │
│  ║   - Visible from anywhere on map                            ║ │
│  ║   - Fades with distance                                     ║ │
│  ╠════════════════════════════════════════════════════════════╣ │
│  ║ STORM WALL SURFACE                                          ║ │
│  ║   - Swirling clouds texture                                 ║ │
│  ║   - Lightning flashes                                       ║ │
│  ║   - Ghostly pirate ship silhouettes                         ║ │
│  ║   - Animated fog particles                                  ║ │
│  ╠════════════════════════════════════════════════════════════╣ │
│  ║ INNER EDGE                                                  ║ │
│  ║   - Sharp boundary line (glowing)                           ║ │
│  ║   - Transition particles                                    ║ │
│  ║   - Clear "you are here" indicator                          ║ │
│  ╚════════════════════════════════════════════════════════════╝ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Storm Wall Shader

```javascript
// Storm wall material properties
const StormWallMaterial = {
    // Base appearance
    baseColor: new Color(0.3, 0.1, 0.5, 0.8), // Purple
    emissionColor: new Color(0.5, 0.3, 0.8, 1.0), // Bright purple
    emissionIntensity: 2.0,

    // Animation
    scrollSpeed: 0.5,          // Cloud movement speed
    distortionStrength: 0.3,   // Wave distortion
    noiseScale: 5.0,           // Cloud detail

    // Lightning
    lightningFrequency: 0.5,   // Flashes per second
    lightningIntensity: 5.0,   // Flash brightness

    // Performance (mobile)
    useSimplifiedShader: isMobileLowEnd(),
    maxParticles: 500,
    lodDistance: 100           // Simplify beyond this distance
};
```

### 5.3 LOD System for Storm

| Distance | Detail Level |
|----------|--------------|
| 0-50m | Full detail (particles, lightning, volumetric) |
| 50-150m | Medium (reduced particles, no volumetric) |
| 150-300m | Low (flat shader, occasional lightning) |
| 300m+ | Minimal (solid color with animated edge) |

---

## 6. UI Indicators

### 6.1 Minimap Display

```
┌─────────────────────────────────────────────────────────────────┐
│                       MINIMAP STORM UI                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────┐                                         │
│  │     MINIMAP        │                                         │
│  │  ┌──────────────┐  │                                         │
│  │  │ ░░░░░░░░░░░░ │  │  ░ = Storm (dark overlay)               │
│  │  │ ░░┌──────┐░░ │  │                                         │
│  │  │ ░░│      │░░ │  │  White circle = Current safe zone       │
│  │  │ ░░│ ○ ●  │░░ │  │  ○ = Next safe zone (dashed line)       │
│  │  │ ░░│      │░░ │  │  ● = Player position                    │
│  │  │ ░░└──────┘░░ │  │                                         │
│  │  │ ░░░░░░░░░░░░ │  │                                         │
│  │  └──────────────┘  │                                         │
│  │                    │                                         │
│  │  ⏱ Next shrink: 45s│                                         │
│  └────────────────────┘                                         │
│                                                                  │
│  Color coding:                                                   │
│  - White/Blue: Current safe zone boundary                        │
│  - Orange/Yellow: Next safe zone (during warning phase)          │
│  - Red pulse: Storm is shrinking NOW                             │
│  - Dark overlay: Storm-covered area                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 HUD Elements

```
┌─────────────────────────────────────────────────────────────────┐
│                       STORM HUD ELEMENTS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Top Center:                                                     │
│  ┌──────────────────────────────────────┐                       │
│  │  🌀 STORM SHRINKING IN: 0:45         │                       │
│  │  ████████████████░░░░░░ (progress)   │                       │
│  └──────────────────────────────────────┘                       │
│                                                                  │
│  In Storm Warning:                                               │
│  ┌──────────────────────────────────────┐                       │
│  │  ⚠️ YOU ARE IN THE STORM!            │                       │
│  │  Taking 4 damage per second          │                       │
│  │  Distance to safe zone: 45m →        │                       │
│  └──────────────────────────────────────┘                       │
│                                                                  │
│  Edge of Screen Indicator:                                       │
│  ┌─────────────────────┐                                        │
│  │ [◄ Safe Zone 120m]  │  Shows direction to safe zone          │
│  └─────────────────────┘  when zone is off-screen                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Compass Integration

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMPASS STORM MARKER                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Standard compass with storm markers:                            │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │  N    ↟    E    ⚡    S    ↟    W    ⚡    N            │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  ↟ = Direction to safe zone center (when outside)                │
│  ⚡ = Storm wall crossing compass direction                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Audio System

### 7.1 Sound Effects

| Sound | Trigger | Properties |
|-------|---------|------------|
| Storm Ambient | In storm | Loud wind, thunder rumbles |
| Storm Edge | Near boundary | Crackling, electric hum |
| Warning Alarm | Phase warning starts | Alert sound, increasing urgency |
| Shrink Start | Zone begins shrinking | Deep rumble, wind pickup |
| Shrink Active | During shrink | Constant rushing wind |
| Safe Arrival | Enter safe zone | Relief sound, wind dies down |
| Storm Damage | Taking damage | Painful crackling, impact |

### 7.2 Audio Zones

```javascript
class StormAudioManager {
    // Distance-based audio
    calculateAudioMix(playerPosition, zoneCenter, zoneRadius) {
        const distanceToCenter = Vector3.Distance(playerPosition, zoneCenter);
        const distanceToEdge = zoneRadius - distanceToCenter;

        let audioMix = {
            stormAmbient: 0,
            edgeCrackling: 0,
            safeZoneAmbient: 1
        };

        if (distanceToEdge < 0) {
            // In storm
            audioMix.stormAmbient = 1.0;
            audioMix.safeZoneAmbient = 0;
            audioMix.edgeCrackling = Math.max(0, 1 - (Math.abs(distanceToEdge) / 50));
        } else if (distanceToEdge < 30) {
            // Near edge (inside)
            const proximity = distanceToEdge / 30;
            audioMix.edgeCrackling = 1 - proximity;
            audioMix.stormAmbient = (1 - proximity) * 0.3;
            audioMix.safeZoneAmbient = proximity;
        }

        return audioMix;
    }
}
```

### 7.3 Warning Escalation

```
Warning Phase Timeline:
├── 30s: Soft warning chime (once)
├── 20s: Repeated chime every 5s
├── 10s: Urgent alarm, faster chimes
├── 5s: Constant alarm, screen edge glow
├── 0s: Shrink begins, intense audio cue
```

---

## 8. Network Synchronization

### 8.1 Zone State Packet

```javascript
// Sent from server to all clients
struct ZoneStatePacket {
    // Current zone
    float3 currentCenter;
    float  currentRadius;

    // Next zone (for UI display)
    float3 nextCenter;
    float  nextRadius;

    // Timing
    uint8  currentPhase;        // 1-5
    uint8  zoneState;           // STABLE, WARNING, SHRINKING
    float  stateTimeRemaining;  // Seconds until next state
    float  serverTimestamp;

    // Damage
    uint8  currentDamagePerSecond;
}
```

### 8.2 Sync Frequency

| State | Update Rate | Notes |
|-------|-------------|-------|
| STABLE | 1 Hz | Low frequency, no movement |
| WARNING | 2 Hz | Show countdown updates |
| SHRINKING | 10 Hz | Smooth visual movement |
| State Change | Immediate | Reliable delivery |

### 8.3 Client-Side Interpolation

```javascript
class ZoneSmoother {
    // Smoothly interpolate zone position/radius between updates
    interpolateZone(previousState, targetState, interpolationFactor) {
        return {
            center: Vector3.Lerp(
                previousState.center,
                targetState.center,
                interpolationFactor
            ),
            radius: Mathf.Lerp(
                previousState.radius,
                targetState.radius,
                interpolationFactor
            )
        };
    }

    // Predict zone state between server updates
    predictZoneState(lastKnownState, timeSinceUpdate) {
        if (lastKnownState.state !== 'SHRINKING') {
            return lastKnownState;
        }

        // Calculate expected shrink progress
        const shrinkProgress = timeSinceUpdate / lastKnownState.shrinkDuration;
        const predictedRadius = Mathf.Lerp(
            lastKnownState.startRadius,
            lastKnownState.targetRadius,
            shrinkProgress
        );

        return {
            ...lastKnownState,
            radius: predictedRadius
        };
    }
}
```

### 8.4 Desync Prevention

| Issue | Solution |
|-------|----------|
| Radius mismatch | Server periodically broadcasts authoritative state |
| Timing drift | Include server timestamp, clients adjust |
| State transition | Reliable messaging for state changes |
| Damage sync | Server-authoritative, client predicts visuals |

---

## 9. Performance Optimization

### 9.1 Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Frame Impact | < 2ms | Storm rendering overhead |
| Draw Calls | < 10 | Storm visuals |
| Particle Count | < 500 | Active storm particles |
| Memory | < 15 MB | Storm textures/meshes |

### 9.2 Mobile Optimization Strategies

```javascript
class StormOptimizer {
    optimizeForDevice(deviceTier) {
        switch(deviceTier) {
            case 'HIGH':
                return {
                    particleCount: 500,
                    shaderComplexity: 'FULL',
                    lightningEnabled: true,
                    volumetricEnabled: true,
                    lodDistances: [100, 200, 400]
                };

            case 'MEDIUM':
                return {
                    particleCount: 200,
                    shaderComplexity: 'MEDIUM',
                    lightningEnabled: true,
                    volumetricEnabled: false,
                    lodDistances: [50, 100, 200]
                };

            case 'LOW':
                return {
                    particleCount: 50,
                    shaderComplexity: 'SIMPLE',
                    lightningEnabled: false,
                    volumetricEnabled: false,
                    lodDistances: [30, 60, 100]
                };
        }
    }
}
```

### 9.3 Occlusion and Culling

```javascript
class StormCulling {
    shouldRenderStormWall(cameraPosition, cameraDirection, zoneCenter, zoneRadius) {
        // Only render storm wall sections visible to camera
        const toZoneCenter = zoneCenter - cameraPosition;
        const angleToCenter = Vector3.Angle(cameraDirection, toZoneCenter);

        // Calculate visible arc of storm wall
        const distanceToZone = toZoneCenter.magnitude;
        const angularSize = Math.atan(zoneRadius / distanceToZone);

        // Determine which segments to render (8 segments around circle)
        const visibleSegments = this.calculateVisibleSegments(
            cameraPosition,
            cameraDirection,
            zoneCenter,
            zoneRadius
        );

        return visibleSegments;
    }
}
```

---

## 10. Gameplay Balancing

### 10.1 Zone Speed Tuning

```javascript
// Zone shrink speed calculation
class ZoneShrinkCalculator {
    calculateShrinkSpeed(currentRadius, targetRadius, shrinkDuration) {
        // Linear shrink
        const linearSpeed = (currentRadius - targetRadius) / shrinkDuration;

        // Eased shrink (slow start, fast middle, slow end)
        // Provides better gameplay feel
        return {
            linearSpeed: linearSpeed,
            easedSpeed: (progress) => {
                // Ease in-out curve
                const t = progress;
                return linearSpeed * (3 * t * t - 2 * t * t * t);
            }
        };
    }
}
```

### 10.2 Balance Considerations

| Factor | Consideration |
|--------|---------------|
| Run Speed | Players can outrun storm at all phases |
| Heal Items | Healing in storm should be viable but risky |
| Edge Play | Playing zone edge should be strategic, not required |
| Catch-up | Late rotations possible but punishing |
| Final Zones | Should have cover/terrain variety |

### 10.3 Anti-Camping Measures

- Storm damage scales up significantly in later phases
- Final zone shrinks to near-zero forcing confrontation
- No healing in final phase (optional rule)
- Zone movement can catch campers off-guard

---

## 11. Edge Cases

### 11.1 Terrain Interactions

| Scenario | Behavior |
|----------|----------|
| Zone on water | Players swim, storm damage applies in water |
| Zone on cliff | Zone boundary is 2D (height ignored) |
| Buildings | Storm passes through buildings |
| Caves/tunnels | Storm applies if 2D position is outside zone |

### 11.2 Player State Interactions

```javascript
class StormInteractions {
    handleSpecialStates(player, isInStorm) {
        // Downed players take storm damage (no immunity)
        if (player.state === 'DOWNED' && isInStorm) {
            this.applyDamage(player);
        }

        // Reviving in storm - both players take damage
        if (player.state === 'REVIVING' && isInStorm) {
            this.applyDamage(player);
            this.applyDamage(player.reviveTarget);
        }

        // Abilities don't block storm damage
        // (e.g., Barrel Roll invulnerability doesn't prevent storm damage)

        // Respawning places player in safe zone
        if (player.isRespawning) {
            player.respawnPosition = this.findSafeRespawnPoint();
        }
    }
}
```

### 11.3 Network Edge Cases

| Scenario | Handling |
|----------|----------|
| Disconnect during shrink | Player takes server-side storm damage |
| Reconnect in storm | Sync current position, apply catch-up damage |
| High latency | Server authoritative, client predicts visuals |
| Zone desync | Force re-sync on significant mismatch |

---

## 12. Testing Requirements

### 12.1 Unit Tests

| Test Case | Description |
|-----------|-------------|
| Zone math | Verify radius/center calculations |
| Damage calculation | Correct damage per phase |
| Timing | Phase durations accurate |
| Boundaries | In/out zone detection |
| Position validity | Next zone within current zone |

### 12.2 Integration Tests

| Test Case | Description |
|-----------|-------------|
| Full match | All phases complete correctly |
| 60 players | Performance with full lobby |
| Network sync | No desync between clients |
| Edge play | Combat at zone edge works |
| Mobile performance | 60 FPS maintained |

### 12.3 Playtest Validation

| Criteria | Target |
|----------|--------|
| Match duration | 10-15 minutes average |
| Storm deaths | < 15% of total deaths |
| Zone clarity | > 95% understand zone system |
| Fairness perception | Zone feels fair to players |

---

## 13. Implementation Checklist

### Phase 1: Core System
- [ ] Zone data structures
- [ ] Phase timing system
- [ ] Basic shrink mechanic
- [ ] Damage calculation

### Phase 2: Visuals
- [ ] Storm wall rendering
- [ ] Minimap display
- [ ] HUD indicators
- [ ] Warning effects

### Phase 3: Audio
- [ ] Storm ambient sounds
- [ ] Warning alarms
- [ ] Damage feedback
- [ ] Edge crackling

### Phase 4: Network
- [ ] State synchronization
- [ ] Client interpolation
- [ ] Damage authority
- [ ] Desync handling

### Phase 5: Polish
- [ ] Performance optimization
- [ ] Mobile-specific tuning
- [ ] Balance adjustments
- [ ] Edge case handling

---

## 14. Appendices

### Appendix A: Phase Timing Reference

```
Phase 1 (2:00 total):
  - Warning: 0:00 - 0:30
  - Shrinking: 0:30 - 1:30
  - Stable: 1:30 - 2:00

Phase 2 (1:40 total):
  - Warning: 2:00 - 2:25
  - Shrinking: 2:25 - 3:15
  - Stable: 3:15 - 3:40

Phase 3 (1:20 total):
  - Warning: 3:40 - 4:00
  - Shrinking: 4:00 - 4:40
  - Stable: 4:40 - 5:00

Phase 4 (1:00 total):
  - Warning: 5:00 - 5:15
  - Shrinking: 5:15 - 5:45
  - Stable: 5:45 - 6:00

Phase 5 (0:30 total):
  - Warning: 6:00 - 6:10
  - Shrinking: 6:10 - 6:30
  - Final: 6:30+

Total: ~9-11 minutes of zone phases
Additional early game time: ~1-2 minutes
Match total: 10-15 minutes
```

### Appendix B: Color Palette

| Element | Color | Hex |
|---------|-------|-----|
| Storm Base | Deep Purple | #4A1A6B |
| Storm Glow | Electric Purple | #8B5CF6 |
| Lightning | Bright White | #FFFFFF |
| Safe Zone Edge | Cyan | #00D4FF |
| Warning Zone | Orange | #FF9500 |
| Danger | Red | #FF3B30 |

### Appendix C: Audio Asset List

| Asset Name | Description | Duration |
|------------|-------------|----------|
| storm_ambient_loop | Wind and thunder | Loop |
| storm_edge_crackling | Electric edge sound | Loop |
| storm_warning_chime | Phase warning | 1s |
| storm_warning_urgent | 10s warning | 2s |
| storm_shrink_start | Shrink begins | 2s |
| storm_damage_tick | Taking damage | 0.5s |
| storm_enter | Entering storm | 1s |
| storm_exit | Leaving storm | 1s |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
