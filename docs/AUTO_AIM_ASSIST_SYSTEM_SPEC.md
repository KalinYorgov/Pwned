# Auto-Aim Assist System Specification

## Overview

This document specifies the auto-aim assist system for Plunderstorm Mobile, designed to help mobile players compete effectively while maintaining skill-based gameplay. The system includes multiple assist types with configurable intensity levels.

## Aim Assist Types

### 1. Soft Lock (Crosshair Magnetism)

**Description:** The crosshair subtly pulls toward nearby enemy targets when aiming near them.

```typescript
interface SoftLockConfig {
  enabled: boolean;
  strength: number;           // 0.0 - 1.0
  detectionRadius: number;    // Screen-space radius for detection
  pullSpeed: number;          // Degrees per second
  maxPullAngle: number;       // Maximum angle adjustment
  falloffCurve: 'linear' | 'quadratic';
}

const SOFT_LOCK_SETTINGS: Record<AimAssistLevel, SoftLockConfig> = {
  off: {
    enabled: false,
    strength: 0,
    detectionRadius: 0,
    pullSpeed: 0,
    maxPullAngle: 0,
    falloffCurve: 'linear'
  },
  low: {
    enabled: true,
    strength: 0.3,
    detectionRadius: 50,      // pixels
    pullSpeed: 30,            // degrees/second
    maxPullAngle: 5,
    falloffCurve: 'quadratic'
  },
  medium: {
    enabled: true,
    strength: 0.5,
    detectionRadius: 75,
    pullSpeed: 50,
    maxPullAngle: 10,
    falloffCurve: 'quadratic'
  },
  high: {
    enabled: true,
    strength: 0.7,
    detectionRadius: 100,
    pullSpeed: 70,
    maxPullAngle: 15,
    falloffCurve: 'linear'
  }
};
```

**Behavior:**
- Activates when crosshair is within detection radius of valid target
- Pull strength decreases with distance from target center
- Does not "snap" - provides gentle guidance
- Player can always override by moving crosshair away

### 2. Target Snap (Acquisition Assist)

**Description:** When initiating aim (pressing attack button), crosshair snaps to nearest valid target within a cone.

```typescript
interface TargetSnapConfig {
  enabled: boolean;
  coneAngle: number;          // Detection cone in degrees
  maxDistance: number;        // World units
  snapSpeed: number;          // Time to reach target (ms)
  priorityWeights: {
    distance: number;
    centeredness: number;     // How centered in screen
    threat: number;           // Currently attacking you
  };
}

const TARGET_SNAP_SETTINGS: Record<AimAssistLevel, TargetSnapConfig> = {
  off: {
    enabled: false,
    coneAngle: 0,
    maxDistance: 0,
    snapSpeed: 0,
    priorityWeights: { distance: 0, centeredness: 0, threat: 0 }
  },
  low: {
    enabled: true,
    coneAngle: 20,
    maxDistance: 15,
    snapSpeed: 100,
    priorityWeights: { distance: 0.5, centeredness: 0.3, threat: 0.2 }
  },
  medium: {
    enabled: true,
    coneAngle: 35,
    maxDistance: 20,
    snapSpeed: 80,
    priorityWeights: { distance: 0.4, centeredness: 0.3, threat: 0.3 }
  },
  high: {
    enabled: true,
    coneAngle: 50,
    maxDistance: 25,
    snapSpeed: 60,
    priorityWeights: { distance: 0.3, centeredness: 0.3, threat: 0.4 }
  }
};
```

**Behavior:**
- Only activates on attack input (not continuously)
- Selects best target based on weighted priority
- Smooth snap animation (not instant teleport)
- Cooldown prevents rapid target switching

### 3. Bullet Magnetism

**Description:** Projectiles slightly curve toward nearby targets during flight.

```typescript
interface BulletMagnetismConfig {
  enabled: boolean;
  curveStrength: number;      // Force applied toward target
  detectionRadius: number;    // How close projectile must be
  maxCurveAngle: number;      // Maximum deviation from straight
  affectedProjectiles: string[];  // Which ability projectiles
}

const BULLET_MAGNETISM_SETTINGS: Record<AimAssistLevel, BulletMagnetismConfig> = {
  off: {
    enabled: false,
    curveStrength: 0,
    detectionRadius: 0,
    maxCurveAngle: 0,
    affectedProjectiles: []
  },
  low: {
    enabled: true,
    curveStrength: 0.2,
    detectionRadius: 1.5,     // meters
    maxCurveAngle: 3,
    affectedProjectiles: ['basic_attack', 'fireball']
  },
  medium: {
    enabled: true,
    curveStrength: 0.4,
    detectionRadius: 2.0,
    maxCurveAngle: 5,
    affectedProjectiles: ['basic_attack', 'fireball', 'poison_mackerel']
  },
  high: {
    enabled: true,
    curveStrength: 0.6,
    detectionRadius: 2.5,
    maxCurveAngle: 8,
    affectedProjectiles: ['basic_attack', 'fireball', 'poison_mackerel', 'cannonball']
  }
};
```

**Behavior:**
- Only affects projectiles already heading near a target
- Curve is subtle and appears natural
- Does not redirect completely missed shots
- Server validates within tolerance

## Target Validation

### Valid Target Criteria

```typescript
interface TargetValidation {
  isEnemy: boolean;           // Not teammate
  isAlive: boolean;           // Not downed/dead
  isVisible: boolean;         // Line of sight check
  isInRange: boolean;         // Within ability range
  notBehindCover: boolean;    // Raycast to target clear
}

function isValidAimAssistTarget(
  shooter: Player,
  target: Player,
  aimPosition: Vector3
): boolean {
  // Must be enemy
  if (target.teamId === shooter.teamId) return false;

  // Must be alive
  if (target.isDead || target.isDowned) return false;

  // Must be visible (line of sight)
  if (!hasLineOfSight(shooter.position, target.position)) return false;

  // Must not be behind full cover
  const hitInfo = raycast(shooter.position, target.position);
  if (hitInfo.hitCover && hitInfo.coverType === 'full') return false;

  // Must be within reasonable range
  const distance = Vector3.Distance(shooter.position, target.position);
  if (distance > MAX_ASSIST_RANGE) return false;

  return true;
}
```

### Line of Sight Check

```typescript
function hasLineOfSight(from: Vector3, to: Vector3): boolean {
  const direction = (to - from).normalized;
  const distance = Vector3.Distance(from, to);

  // Raycast with collision mask (walls, terrain)
  const hit = Physics.Raycast(from, direction, distance, COLLISION_MASK);

  // No hit means clear line of sight
  return !hit;
}
```

## Ability-Specific Assist

### Assist Multipliers by Ability

Different abilities receive different levels of aim assist:

| Ability | Assist Multiplier | Reason |
|---------|-------------------|--------|
| Basic Attack | 1.0 | Full assist |
| Fireball | 0.8 | Standard skillshot |
| Cutlass Slash | 0.5 | Melee, lower need |
| Cannonball | 0.6 | Slow projectile |
| Flintlock Pistol | 0.7 | Precision weapon |
| Lightning Strike | 0.3 | Area effect |
| Healing Tide | 0.0 | Self-cast |
| Grappling Hook | 0.4 | Utility focus |

```typescript
const ABILITY_ASSIST_MULTIPLIERS: Record<string, number> = {
  'basic_attack': 1.0,
  'fireball': 0.8,
  'cutlass_slash': 0.5,
  'cannonball': 0.6,
  'flintlock_pistol': 0.7,
  'blunderbuss': 0.5,
  'lightning_strike': 0.3,
  'cannon_barrage': 0.3,
  'healing_tide': 0.0,
  'smoke_bomb': 0.0,
  'grappling_hook': 0.4,
  'wind_burst': 0.2
};

function getEffectiveAssistStrength(
  baseStrength: number,
  abilityId: string
): number {
  const multiplier = ABILITY_ASSIST_MULTIPLIERS[abilityId] ?? 1.0;
  return baseStrength * multiplier;
}
```

## Settings UI

### Aim Assist Settings Panel

```
┌─────────────────────────────────────────────────────────────────┐
│                    AIM ASSIST SETTINGS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Aim Assist Level                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  [OFF]    [LOW]    [MEDIUM]    [HIGH]                     │ │
│  │                       ▲                                    │ │
│  │                   Selected                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌─ Advanced Settings ──────────────────────────────────────┐  │
│  │                                                           │  │
│  │  Soft Lock (Crosshair Pull)     [●━━━━━━━━○] 50%         │  │
│  │  Target Snap                     [●━━━━━━━━○] 50%         │  │
│  │  Bullet Magnetism               [●━━━━━━━━○] 50%         │  │
│  │                                                           │  │
│  │  Show Assist Indicator          [ON]  [OFF]              │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ℹ️ Aim assist helps your crosshair stay on target.            │
│     Higher settings provide more assistance.                    │
│     Ranked matches above Gold may disable aim assist.          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Settings Data Model

```typescript
interface AimAssistSettings {
  level: AimAssistLevel;
  customSettings?: {
    softLockStrength: number;     // 0-100
    targetSnapStrength: number;   // 0-100
    bulletMagnetism: number;      // 0-100
  };
  showIndicator: boolean;
}

enum AimAssistLevel {
  OFF = 'off',
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CUSTOM = 'custom'
}
```

## Visual Indicators

### Aim Assist Active Indicator

When aim assist is helping, show subtle visual feedback:

```
[Normal Crosshair]          [Assist Active]
      +                          ⊕
                           (subtle glow)
```

```typescript
interface AssistIndicator {
  enabled: boolean;
  type: 'glow' | 'color' | 'expand';
  color: Color;
  intensity: number;
}

const ASSIST_INDICATOR: AssistIndicator = {
  enabled: true,
  type: 'glow',
  color: new Color(0.5, 0.8, 1.0, 0.3),  // Light blue
  intensity: 0.5
};

function updateCrosshairIndicator(isAssisting: boolean): void {
  if (!settings.showIndicator) return;

  if (isAssisting) {
    crosshair.setGlow(ASSIST_INDICATOR.color, ASSIST_INDICATOR.intensity);
    crosshair.scale = 1.1;  // Slight expansion
  } else {
    crosshair.clearGlow();
    crosshair.scale = 1.0;
  }
}
```

### Target Highlight

When target snap acquires a target:

```typescript
function showTargetAcquired(target: Player): void {
  // Brief outline flash on target
  target.outlineEffect.flash(Color.yellow, 0.2);

  // Sound feedback
  playSound('sfx_target_lock', 0.3);
}
```

## Implementation Details

### Aim Assist Update Loop

```typescript
class AimAssistSystem {
  private currentTarget: Player | null = null;
  private assistActive: boolean = false;

  update(deltaTime: number): void {
    const player = getLocalPlayer();
    if (!player || !player.isAiming) {
      this.clearAssist();
      return;
    }

    const settings = getAimAssistSettings();
    if (settings.level === AimAssistLevel.OFF) {
      this.clearAssist();
      return;
    }

    // Find potential targets
    const targets = this.findValidTargets(player);
    if (targets.length === 0) {
      this.clearAssist();
      return;
    }

    // Select best target
    const bestTarget = this.selectBestTarget(player, targets);
    this.currentTarget = bestTarget;

    // Apply soft lock
    if (settings.softLockStrength > 0) {
      this.applySoftLock(player, bestTarget, deltaTime);
    }

    this.assistActive = true;
    updateCrosshairIndicator(true);
  }

  private findValidTargets(player: Player): Player[] {
    const nearbyPlayers = getPlayersInRadius(player.position, MAX_ASSIST_RANGE);

    return nearbyPlayers.filter(target =>
      isValidAimAssistTarget(player, target, player.aimPosition)
    );
  }

  private selectBestTarget(player: Player, targets: Player[]): Player {
    const config = getTargetSnapConfig();

    return targets.reduce((best, target) => {
      const score = this.calculateTargetScore(player, target, config);
      const bestScore = this.calculateTargetScore(player, best, config);
      return score > bestScore ? target : best;
    });
  }

  private calculateTargetScore(
    player: Player,
    target: Player,
    config: TargetSnapConfig
  ): number {
    const distance = Vector3.Distance(player.position, target.position);
    const screenPos = worldToScreen(target.position);
    const centerDistance = Vector2.Distance(screenPos, screenCenter);
    const isThreat = target.lastAttackedPlayer === player.id;

    const distanceScore = 1 - (distance / config.maxDistance);
    const centerScore = 1 - (centerDistance / screenDiagonal);
    const threatScore = isThreat ? 1 : 0;

    return (
      distanceScore * config.priorityWeights.distance +
      centerScore * config.priorityWeights.centeredness +
      threatScore * config.priorityWeights.threat
    );
  }

  private applySoftLock(
    player: Player,
    target: Player,
    deltaTime: number
  ): void {
    const config = getSoftLockConfig();
    const targetScreenPos = worldToScreen(target.position);
    const crosshairPos = getCrosshairPosition();

    const toTarget = targetScreenPos - crosshairPos;
    const distance = toTarget.magnitude;

    if (distance > config.detectionRadius) return;

    // Calculate pull strength with falloff
    let pullStrength = config.strength;
    if (config.falloffCurve === 'quadratic') {
      const normalizedDist = distance / config.detectionRadius;
      pullStrength *= (1 - normalizedDist * normalizedDist);
    }

    // Apply pull
    const pullAmount = config.pullSpeed * pullStrength * deltaTime;
    const maxPull = config.maxPullAngle * (Math.PI / 180);

    const pullVector = toTarget.normalized * Math.min(pullAmount, maxPull);
    adjustCrosshairPosition(pullVector);
  }

  onAttackPressed(): void {
    if (!this.currentTarget) return;

    const config = getTargetSnapConfig();
    if (!config.enabled) return;

    // Apply target snap
    this.snapToTarget(this.currentTarget, config.snapSpeed);
    showTargetAcquired(this.currentTarget);
  }

  private snapToTarget(target: Player, duration: number): void {
    const targetScreenPos = worldToScreen(target.position);
    animateCrosshairTo(targetScreenPos, duration);
  }

  private clearAssist(): void {
    this.currentTarget = null;
    this.assistActive = false;
    updateCrosshairIndicator(false);
  }
}
```

### Bullet Magnetism Implementation

```typescript
class BulletMagnetismSystem {
  applyMagnetism(projectile: Projectile, deltaTime: number): void {
    const config = getBulletMagnetismConfig();
    if (!config.enabled) return;

    if (!config.affectedProjectiles.includes(projectile.abilityId)) {
      return;
    }

    // Find nearby targets
    const nearbyTargets = getEnemiesInRadius(
      projectile.position,
      config.detectionRadius
    );

    if (nearbyTargets.length === 0) return;

    // Find closest valid target
    const closestTarget = nearbyTargets.reduce((closest, target) => {
      if (!isValidAimAssistTarget(projectile.owner, target, projectile.position)) {
        return closest;
      }
      const dist = Vector3.Distance(projectile.position, target.position);
      const closestDist = closest ?
        Vector3.Distance(projectile.position, closest.position) : Infinity;
      return dist < closestDist ? target : closest;
    }, null as Player | null);

    if (!closestTarget) return;

    // Calculate curve force
    const toTarget = (closestTarget.position - projectile.position).normalized;
    const currentDir = projectile.velocity.normalized;
    const angleToTarget = Vector3.Angle(currentDir, toTarget);

    // Only curve if within max angle
    if (angleToTarget > config.maxCurveAngle) return;

    // Apply curve
    const curveForce = toTarget * config.curveStrength;
    projectile.velocity = Vector3.Lerp(
      projectile.velocity,
      projectile.velocity + curveForce,
      deltaTime
    ).normalized * projectile.speed;
  }
}
```

## Ranked Mode Restrictions

### Rank-Based Assist Limits

```typescript
interface RankedAssistRestrictions {
  rank: string;
  maxAssistLevel: AimAssistLevel;
  bulletMagnetismAllowed: boolean;
}

const RANKED_RESTRICTIONS: RankedAssistRestrictions[] = [
  { rank: 'deckhand', maxAssistLevel: 'high', bulletMagnetismAllowed: true },
  { rank: 'sailor', maxAssistLevel: 'high', bulletMagnetismAllowed: true },
  { rank: 'boatswain', maxAssistLevel: 'medium', bulletMagnetismAllowed: true },
  { rank: 'first_mate', maxAssistLevel: 'low', bulletMagnetismAllowed: false },
  { rank: 'captain', maxAssistLevel: 'low', bulletMagnetismAllowed: false },
  { rank: 'admiral', maxAssistLevel: 'off', bulletMagnetismAllowed: false },
  { rank: 'pirate_king', maxAssistLevel: 'off', bulletMagnetismAllowed: false }
];

function getEffectiveAssistSettings(
  userSettings: AimAssistSettings,
  matchType: string,
  playerRank: string
): AimAssistSettings {
  // No restrictions in casual
  if (matchType === 'casual') return userSettings;

  // Apply ranked restrictions
  const restriction = RANKED_RESTRICTIONS.find(r => r.rank === playerRank);
  if (!restriction) return userSettings;

  const effectiveLevel = Math.min(
    AIM_ASSIST_LEVELS.indexOf(userSettings.level),
    AIM_ASSIST_LEVELS.indexOf(restriction.maxAssistLevel)
  );

  return {
    ...userSettings,
    level: AIM_ASSIST_LEVELS[effectiveLevel],
    bulletMagnetism: restriction.bulletMagnetismAllowed ?
      userSettings.bulletMagnetism : 0
  };
}
```

## Anti-Exploit Measures

### Server Validation

```typescript
function validateAimAssistBehavior(
  player: Player,
  shotData: ShotData
): ValidationResult {
  // Check for impossible accuracy
  const recentShots = getRecentShots(player.id, 10);
  const hitRate = recentShots.filter(s => s.hit).length / recentShots.length;

  if (hitRate > 0.95 && recentShots.length >= 10) {
    return { valid: false, reason: 'impossibly_high_accuracy' };
  }

  // Check for spin-bot behavior (rapid target switching)
  const targetSwitches = countTargetSwitches(player.id, 5000); // 5 seconds
  if (targetSwitches > 10) {
    return { valid: false, reason: 'excessive_target_switching' };
  }

  // Check bullet magnetism isn't exceeding limits
  if (shotData.curveAngle > MAX_ALLOWED_CURVE * 1.5) {
    return { valid: false, reason: 'excessive_bullet_curve' };
  }

  return { valid: true };
}
```

### Rate Limiting

```typescript
const ASSIST_RATE_LIMITS = {
  targetSnapCooldown: 200,      // ms between snaps
  maxSnapsPerSecond: 3,
  maxCurveApplications: 60      // per second
};
```

## Analytics Events

```typescript
// Aim assist usage tracking
analytics.track("aim_assist_settings", {
  playerId: string,
  level: AimAssistLevel,
  customSettings: object
});

analytics.track("aim_assist_engagement", {
  playerId: string,
  matchId: string,
  assistLevel: string,
  shotsWithAssist: number,
  shotsWithoutAssist: number,
  hitRateWithAssist: number,
  hitRateWithoutAssist: number
});

analytics.track("aim_assist_ranked_restriction", {
  playerId: string,
  requestedLevel: string,
  appliedLevel: string,
  rank: string
});
```

## Testing Requirements

### Functional Tests

- [ ] All assist levels function correctly
- [ ] Soft lock pulls crosshair smoothly
- [ ] Target snap selects appropriate targets
- [ ] Bullet magnetism curves within limits
- [ ] Line of sight blocking works
- [ ] Settings save and load correctly
- [ ] Ranked restrictions apply correctly

### Balance Tests

- [ ] Assist doesn't guarantee hits
- [ ] Skilled players competitive with assist off
- [ ] No assist level feels like "aimbot"
- [ ] Ability multipliers feel balanced

### Anti-Exploit Tests

- [ ] Spin-bot detection works
- [ ] Excessive accuracy flagged
- [ ] Server validation catches invalid curves

## Success Metrics

| Metric | Target |
|--------|--------|
| Players using assist (casual) | 70-80% |
| Hit rate improvement | +15-25% with assist |
| "Feels like cheating" feedback | < 5% |
| Exploit reports | < 1% of matches |

## Dependencies

- **GAME-011**: Basic Attack System (attack integration)
- **GAME-013**: Hit Detection (projectile validation)
- **UX-004**: Skillshot Targeting (ability targeting)
