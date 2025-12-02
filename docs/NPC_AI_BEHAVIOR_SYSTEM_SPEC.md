# NPC AI Behavior System Specification

## Overview

This document specifies the AI behavior system for NPC monsters in Plunderstorm Mobile. The system uses a state machine architecture optimized for server performance while creating engaging PvE encounters.

## State Machine Architecture

### Core States

```
┌─────────────────────────────────────────────────────────────────┐
│                       MONSTER STATE MACHINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────┐     player in range      ┌───────┐                   │
│   │ IDLE │ ─────────────────────────▶│ ALERT │                   │
│   └──────┘                           └───────┘                   │
│       │                                  │                       │
│       │ patrol timer                     │ approach complete     │
│       ▼                                  ▼                       │
│   ┌────────┐                        ┌────────┐                   │
│   │ PATROL │◀───────────────────────│ COMBAT │                   │
│   └────────┘    player left range   └────────┘                   │
│       │                                  │                       │
│       │ player detected                  │ leash exceeded        │
│       ▼                                  ▼                       │
│   ┌───────┐                         ┌────────┐                   │
│   │ ALERT │                         │ RETURN │                   │
│   └───────┘                         └────────┘                   │
│                                          │                       │
│                                          │ reached spawn         │
│                                          ▼                       │
│                                     ┌──────┐                     │
│                                     │ IDLE │                     │
│                                     └──────┘                     │
│                                                                  │
│   Any State ────── HP <= 0 ──────▶ ┌───────┐                    │
│                                     │ DEATH │                    │
│                                     └───────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

### State Definitions

#### IDLE State

**Description:** Monster is at spawn point, not actively engaged.

```typescript
interface IdleState {
  duration: number;         // Time in idle before patrol
  animationSet: 'idle_1' | 'idle_2' | 'idle_ambient';
  lookAroundTimer: number;  // Random head turns
}

function updateIdleState(monster: Monster, dt: number): StateTransition {
  monster.idleTimer += dt;

  // Random ambient animations
  if (monster.lookAroundTimer <= 0) {
    playAnimation(monster, 'idle_look_around');
    monster.lookAroundTimer = randomRange(3, 8);
  }

  // Check for nearby players
  const nearestPlayer = findNearestPlayer(monster.position, monster.aggroRange);
  if (nearestPlayer) {
    return { nextState: 'ALERT', target: nearestPlayer };
  }

  // Transition to patrol after idle duration
  if (monster.idleTimer >= monster.idleDuration) {
    return { nextState: 'PATROL' };
  }

  return { nextState: 'IDLE' };
}
```

#### PATROL State

**Description:** Monster moves between waypoints within its camp.

```typescript
interface PatrolState {
  waypoints: Vector3[];
  currentWaypointIndex: number;
  patrolSpeed: number;
  waitAtWaypoint: number;
}

function updatePatrolState(monster: Monster, dt: number): StateTransition {
  const target = monster.waypoints[monster.currentWaypointIndex];
  const distance = Vector3.Distance(monster.position, target);

  // Check for aggro first
  const nearestPlayer = findNearestPlayer(monster.position, monster.aggroRange);
  if (nearestPlayer) {
    return { nextState: 'ALERT', target: nearestPlayer };
  }

  // Reached waypoint
  if (distance < 0.5) {
    monster.waypointWaitTimer -= dt;
    if (monster.waypointWaitTimer <= 0) {
      monster.currentWaypointIndex =
        (monster.currentWaypointIndex + 1) % monster.waypoints.length;
      monster.waypointWaitTimer = monster.waitAtWaypoint;
    }
    return { nextState: 'PATROL' };
  }

  // Move toward waypoint
  moveToward(monster, target, monster.patrolSpeed * dt);
  return { nextState: 'PATROL' };
}
```

#### ALERT State

**Description:** Monster has detected a player and is preparing to engage.

```typescript
interface AlertState {
  alertDuration: number;     // Time before chase (reaction time)
  targetPlayer: string;
}

function updateAlertState(monster: Monster, dt: number): StateTransition {
  monster.alertTimer += dt;

  // Face the target
  const targetPos = getPlayerPosition(monster.targetPlayer);
  rotateToward(monster, targetPos);

  // Play alert animation/sound
  if (monster.alertTimer < 0.1) {
    playAnimation(monster, 'alert_notice');
    playSound(monster, 'sfx_aggro');
  }

  // Transition to combat after alert duration
  if (monster.alertTimer >= monster.alertDuration) {
    return { nextState: 'COMBAT', target: monster.targetPlayer };
  }

  // Target left range during alert
  const distance = Vector3.Distance(monster.position, targetPos);
  if (distance > monster.aggroRange * 1.5) {
    return { nextState: 'IDLE' };
  }

  return { nextState: 'ALERT' };
}
```

#### COMBAT State

**Description:** Monster is actively fighting a player.

```typescript
interface CombatState {
  attackCooldown: number;
  currentTarget: string;
  chaseSpeed: number;
  attackRange: number;
}

function updateCombatState(monster: Monster, dt: number): StateTransition {
  const targetPos = getPlayerPosition(monster.currentTarget);
  const distance = Vector3.Distance(monster.position, targetPos);
  const distanceFromSpawn = Vector3.Distance(monster.position, monster.spawnPoint);

  // Check leash distance
  if (distanceFromSpawn > monster.leashRange) {
    return { nextState: 'RETURN' };
  }

  // Target died or disconnected
  if (!isValidTarget(monster.currentTarget)) {
    const newTarget = findHighestThreat(monster);
    if (newTarget) {
      monster.currentTarget = newTarget;
    } else {
      return { nextState: 'RETURN' };
    }
  }

  // In attack range
  if (distance <= monster.attackRange) {
    monster.attackCooldown -= dt;
    if (monster.attackCooldown <= 0) {
      executeAttack(monster, monster.currentTarget);
      monster.attackCooldown = monster.attackSpeed;
    }
    // Face target while attacking
    rotateToward(monster, targetPos);
  } else {
    // Chase target
    const path = getNavMeshPath(monster.position, targetPos);
    moveAlongPath(monster, path, monster.chaseSpeed * dt);
  }

  return { nextState: 'COMBAT' };
}
```

#### RETURN State

**Description:** Monster is returning to spawn after losing aggro or exceeding leash.

```typescript
interface ReturnState {
  returnSpeed: number;       // Faster than patrol
  healRate: number;          // HP regen while returning
  invulnerable: boolean;     // Cannot be damaged while returning
}

function updateReturnState(monster: Monster, dt: number): StateTransition {
  const distanceToSpawn = Vector3.Distance(monster.position, monster.spawnPoint);

  // Reached spawn point
  if (distanceToSpawn < 1.0) {
    monster.currentHP = monster.maxHP;  // Full heal on return
    monster.aggroTable.clear();
    return { nextState: 'IDLE' };
  }

  // Move toward spawn
  const path = getNavMeshPath(monster.position, monster.spawnPoint);
  moveAlongPath(monster, path, monster.returnSpeed * dt);

  // Heal while returning
  monster.currentHP = Math.min(
    monster.maxHP,
    monster.currentHP + monster.healRate * dt
  );

  return { nextState: 'RETURN' };
}
```

#### DEATH State

**Description:** Monster has been killed.

```typescript
interface DeathState {
  deathAnimationDuration: number;
  lootDropped: boolean;
}

function updateDeathState(monster: Monster, dt: number): StateTransition {
  if (!monster.deathAnimStarted) {
    playAnimation(monster, 'death');
    playSound(monster, 'sfx_death');
    monster.deathAnimStarted = true;
  }

  monster.deathTimer += dt;

  // Drop loot after brief delay
  if (!monster.lootDropped && monster.deathTimer >= 0.3) {
    spawnLootDrops(monster);
    awardXP(monster);
    monster.lootDropped = true;
  }

  // Remove from world after animation
  if (monster.deathTimer >= monster.deathAnimationDuration) {
    scheduleRespawn(monster);
    return { nextState: 'REMOVED' };
  }

  return { nextState: 'DEATH' };
}
```

## Pathfinding System

### NavMesh Configuration

```typescript
interface NavMeshConfig {
  agentRadius: 0.5;          // Monster collision radius
  agentHeight: 2.0;          // Standing height
  maxSlope: 45;              // Maximum walkable slope (degrees)
  stepHeight: 0.4;           // Maximum step height
  cellSize: 0.3;             // NavMesh cell resolution
}
```

### Path Request System

```typescript
class PathfindingManager {
  private pendingRequests: PathRequest[] = [];
  private maxRequestsPerFrame: number = 10;

  requestPath(
    start: Vector3,
    end: Vector3,
    callback: (path: Vector3[]) => void
  ): void {
    this.pendingRequests.push({ start, end, callback });
  }

  update(): void {
    // Process limited requests per frame to avoid spikes
    const toProcess = Math.min(
      this.maxRequestsPerFrame,
      this.pendingRequests.length
    );

    for (let i = 0; i < toProcess; i++) {
      const request = this.pendingRequests.shift()!;
      const path = this.calculatePath(request.start, request.end);
      request.callback(path);
    }
  }

  private calculatePath(start: Vector3, end: Vector3): Vector3[] {
    // A* on NavMesh
    return NavMesh.FindPath(start, end);
  }
}
```

### Path Smoothing

```typescript
function smoothPath(rawPath: Vector3[]): Vector3[] {
  if (rawPath.length <= 2) return rawPath;

  const smoothed: Vector3[] = [rawPath[0]];
  let current = 0;

  while (current < rawPath.length - 1) {
    // Find furthest visible point
    let furthest = current + 1;
    for (let i = rawPath.length - 1; i > current + 1; i--) {
      if (hasLineOfSight(rawPath[current], rawPath[i])) {
        furthest = i;
        break;
      }
    }
    smoothed.push(rawPath[furthest]);
    current = furthest;
  }

  return smoothed;
}
```

## Attack Patterns

### Basic Monster Attacks

#### Skeleton Pirate - Cutlass Swing

```typescript
const SKELETON_ATTACK: AttackPattern = {
  name: 'cutlass_swing',
  windupTime: 0.3,           // Telegraph time
  activeTime: 0.2,           // Damage window
  recoveryTime: 0.3,         // Cooldown after attack
  damage: 8,
  range: 2.0,
  arc: 90,                   // Frontal cone degrees
  animation: 'attack_swing',
  vfx: 'vfx_sword_swing',
  sfx: 'sfx_sword_whoosh'
};
```

#### Sea Crab - Claw Pinch

```typescript
const CRAB_ATTACK: AttackPattern = {
  name: 'claw_pinch',
  windupTime: 0.4,
  activeTime: 0.15,
  recoveryTime: 0.5,
  damage: 15,
  range: 2.5,
  arc: 60,
  stunDuration: 0.3,         // Mini-stun on hit
  animation: 'attack_pinch',
  vfx: 'vfx_claw_snap',
  sfx: 'sfx_claw_snap'
};
```

#### Cursed Sailor - Spectral Shot

```typescript
const SAILOR_ATTACK: AttackPattern = {
  name: 'spectral_shot',
  windupTime: 0.5,
  activeTime: 0.1,           // Instant projectile spawn
  recoveryTime: 0.8,
  damage: 20,
  range: 12.0,
  projectileSpeed: 15,
  projectileSize: 0.3,
  animation: 'attack_shoot',
  vfx: 'vfx_ghost_bullet',
  sfx: 'sfx_ghost_shot'
};
```

### Mini-Boss Attack Phases

#### Captain Deadbeard - Phase System

```typescript
interface BossPhase {
  healthThreshold: number;   // Transition when HP below this %
  attackSet: AttackPattern[];
  specialAbility?: SpecialAbility;
  enrageMultiplier: number;
}

const CAPTAIN_DEADBEARD_PHASES: BossPhase[] = [
  {
    healthThreshold: 100,    // Phase 1: 100-70% HP
    attackSet: [CUTLASS_COMBO, PISTOL_SHOT],
    specialAbility: null,
    enrageMultiplier: 1.0
  },
  {
    healthThreshold: 70,     // Phase 2: 70-30% HP
    attackSet: [CUTLASS_COMBO, PISTOL_SHOT, SUMMON_CREW],
    specialAbility: SUMMON_CREW,
    enrageMultiplier: 1.15
  },
  {
    healthThreshold: 30,     // Phase 3: 30-0% HP
    attackSet: [CUTLASS_COMBO, PISTOL_SHOT, CAPTAINS_ROAR],
    specialAbility: CAPTAINS_ROAR,
    enrageMultiplier: 1.3
  }
];

// Special Abilities
const SUMMON_CREW: SpecialAbility = {
  name: 'summon_crew',
  cooldown: 30,
  castTime: 1.5,
  spawnCount: 2,
  spawnType: 'skeleton_pirate',
  animation: 'cast_summon',
  vfx: 'vfx_summon_circle',
  voiceLine: 'vo_all_hands'
};

const CAPTAINS_ROAR: SpecialAbility = {
  name: 'captains_roar',
  cooldown: 45,
  castTime: 1.0,
  effect: 'fear',
  effectDuration: 3,
  radius: 8,
  animation: 'cast_roar',
  vfx: 'vfx_fear_wave',
  sfx: 'sfx_boss_roar'
};
```

### Attack Selection Logic

```typescript
function selectAttack(monster: Monster): AttackPattern {
  const availableAttacks = monster.attackSet.filter(
    attack => monster.attackCooldowns[attack.name] <= 0
  );

  if (availableAttacks.length === 0) {
    return null; // No attacks ready
  }

  // Weight selection based on situation
  const weights = availableAttacks.map(attack => {
    let weight = 1.0;

    // Prefer ranged if target is far
    const distance = getDistanceToTarget(monster);
    if (attack.range > 5 && distance > 5) {
      weight *= 1.5;
    }

    // Prefer melee if target is close
    if (attack.range <= 3 && distance <= 3) {
      weight *= 1.5;
    }

    // Vary attacks (don't repeat same attack)
    if (attack.name === monster.lastAttack) {
      weight *= 0.5;
    }

    return weight;
  });

  return weightedRandomSelect(availableAttacks, weights);
}
```

## Behavior Variants by Monster Type

### Skeleton Pirate Behavior

```typescript
const SKELETON_BEHAVIOR: BehaviorConfig = {
  // States
  idleDuration: randomRange(2, 5),
  patrolRadius: 5,
  alertDuration: 0.5,

  // Combat
  aggroRange: 8,
  leashRange: 20,
  attackRange: 2,
  chaseSpeed: 4,
  patrolSpeed: 2,

  // Group behavior
  alertsNearby: true,        // Alerts other skeletons
  alertRadius: 10,

  // Tactics
  prefersGrouping: true,     // Tries to stay near allies
  fleeAtHealth: 0,           // Never flees
};
```

### Sea Crab Behavior

```typescript
const CRAB_BEHAVIOR: BehaviorConfig = {
  idleDuration: randomRange(5, 10),
  patrolRadius: 3,
  alertDuration: 0.3,

  aggroRange: 6,
  leashRange: 15,
  attackRange: 2.5,
  chaseSpeed: 3,
  patrolSpeed: 1.5,

  // Unique: Burrow ambush
  burrowOnIdle: true,
  burrowEmergeDuration: 0.5,

  alertsNearby: false,
  prefersGrouping: false,
  fleeAtHealth: 0,
};
```

### Cursed Sailor Behavior

```typescript
const SAILOR_BEHAVIOR: BehaviorConfig = {
  idleDuration: randomRange(3, 6),
  patrolRadius: 8,
  alertDuration: 0.6,

  aggroRange: 12,
  leashRange: 25,
  attackRange: 12,
  chaseSpeed: 3.5,
  patrolSpeed: 2,

  // Ranged kiting
  preferredRange: 8,         // Tries to maintain this distance
  kiteThreshold: 4,          // Kites if player closer than this

  alertsNearby: true,
  alertRadius: 15,
  prefersGrouping: false,
  fleeAtHealth: 0,
};
```

### Mini-Boss Behavior

```typescript
const BOSS_BEHAVIOR: BehaviorConfig = {
  idleDuration: 0,           // Always alert
  patrolRadius: 0,           // Stationary until aggro
  alertDuration: 1.0,        // Dramatic alert

  aggroRange: 15,
  leashRange: 30,
  attackRange: 3,            // Variable by attack
  chaseSpeed: 5,
  patrolSpeed: 0,

  // Boss-specific
  phaseTransitions: true,
  announcePhaseChange: true,
  immuneDuringTransition: true,
  transitionDuration: 2.0,

  alertsNearby: false,       // Doesn't call adds (except via ability)
  prefersGrouping: false,
  fleeAtHealth: 0,
};
```

## Performance Optimization

### AI Budget System

```typescript
class AIBudgetManager {
  private budgetPerFrame: number = 5; // milliseconds
  private updateQueue: Monster[] = [];
  private priorityMultipliers = {
    inCombat: 3.0,
    nearPlayer: 2.0,
    idle: 0.5
  };

  update(): void {
    const startTime = performance.now();

    // Sort by priority
    this.updateQueue.sort((a, b) =>
      this.getPriority(b) - this.getPriority(a)
    );

    // Process until budget exhausted
    while (this.updateQueue.length > 0) {
      if (performance.now() - startTime > this.budgetPerFrame) {
        break; // Budget exhausted
      }

      const monster = this.updateQueue.shift()!;
      this.updateMonster(monster);
    }
  }

  private getPriority(monster: Monster): number {
    if (monster.state === 'COMBAT') {
      return 100 * this.priorityMultipliers.inCombat;
    }
    if (this.isNearAnyPlayer(monster)) {
      return 50 * this.priorityMultipliers.nearPlayer;
    }
    return 10 * this.priorityMultipliers.idle;
  }
}
```

### Update Frequency Tiers

| Tier | Condition | Update Rate | Features |
|------|-----------|-------------|----------|
| High | In combat | 20 Hz | Full AI, pathfinding, attacks |
| Medium | Player within 30m | 10 Hz | Patrol, aggro checks |
| Low | No players nearby | 5 Hz | Idle animation only |
| Dormant | 60m+ from players | 1 Hz | Existence check only |

### Batch Processing

```typescript
function batchPathfindingRequests(monsters: Monster[]): void {
  const requests: PathRequest[] = [];

  for (const monster of monsters) {
    if (monster.needsNewPath) {
      requests.push({
        monsterId: monster.id,
        start: monster.position,
        end: monster.targetPosition
      });
    }
  }

  // Process in batch to share computation
  const paths = NavMesh.BatchFindPaths(requests);

  for (let i = 0; i < monsters.length; i++) {
    monsters[i].currentPath = paths[i];
    monsters[i].needsNewPath = false;
  }
}
```

## Edge Case Handling

### Target Validation

```typescript
function isValidTarget(playerId: string): boolean {
  const player = getPlayer(playerId);
  if (!player) return false;
  if (player.isDead) return false;
  if (player.isDisconnected) return false;
  if (player.isInvulnerable) return false;
  return true;
}
```

### Stuck Detection and Recovery

```typescript
function checkStuck(monster: Monster): boolean {
  if (monster.state !== 'COMBAT' && monster.state !== 'RETURN') {
    return false;
  }

  // Check if position hasn't changed
  const moved = Vector3.Distance(
    monster.position,
    monster.lastPosition
  );

  if (moved < 0.1) {
    monster.stuckCounter++;
  } else {
    monster.stuckCounter = 0;
  }

  monster.lastPosition = monster.position;

  // Stuck for too long
  if (monster.stuckCounter > 30) { // 3 seconds at 10Hz
    recoverFromStuck(monster);
    return true;
  }

  return false;
}

function recoverFromStuck(monster: Monster): void {
  // Try teleport to nearest valid NavMesh point
  const validPoint = NavMesh.FindNearestPoint(monster.position, 5);
  if (validPoint) {
    monster.position = validPoint;
    monster.stuckCounter = 0;
    return;
  }

  // Fallback: Return to spawn
  monster.state = 'RETURN';
}
```

### Multi-Player Aggro Handling

```typescript
function handleMultipleAttackers(monster: Monster): void {
  // Update threat table
  for (const [playerId, entry] of monster.aggroTable) {
    // Decay old threat
    if (Date.now() - entry.lastDamageTime > 5000) {
      entry.threat *= 0.95;
    }

    // Remove if too low
    if (entry.threat < 1) {
      monster.aggroTable.delete(playerId);
    }
  }

  // Select highest threat as target
  let highestThreat = 0;
  let newTarget = null;

  for (const [playerId, entry] of monster.aggroTable) {
    if (entry.threat > highestThreat && isValidTarget(playerId)) {
      highestThreat = entry.threat;
      newTarget = playerId;
    }
  }

  if (newTarget && newTarget !== monster.currentTarget) {
    monster.currentTarget = newTarget;
    // Visual feedback for target swap
    showTargetSwapIndicator(monster);
  }
}
```

### Player Disconnect During Combat

```typescript
function handlePlayerDisconnect(playerId: string): void {
  for (const monster of activeMonsters) {
    if (monster.currentTarget === playerId) {
      // Remove from aggro table
      monster.aggroTable.delete(playerId);

      // Find new target or reset
      const newTarget = monster.aggroTable.getHighestThreat();
      if (newTarget) {
        monster.currentTarget = newTarget;
      } else {
        monster.state = 'RETURN';
      }
    }
  }
}
```

## Animation Integration

### Animation State Machine

```typescript
interface MonsterAnimator {
  currentAnimation: string;
  blendTime: number;
  animationSpeed: number;
}

const ANIMATION_MAP = {
  'IDLE': ['idle_breathe', 'idle_look', 'idle_scratch'],
  'PATROL': ['walk_forward'],
  'ALERT': ['alert_notice'],
  'COMBAT': ['run_forward'],
  'ATTACK': ['attack_swing', 'attack_heavy'],
  'RETURN': ['run_forward'],
  'DEATH': ['death_fall', 'death_dramatic']
};

function updateAnimation(monster: Monster): void {
  const targetAnim = selectAnimation(monster);

  if (targetAnim !== monster.animator.currentAnimation) {
    crossfadeAnimation(
      monster,
      targetAnim,
      monster.animator.blendTime
    );
  }
}
```

### Animation Events

```typescript
interface AnimationEvent {
  time: number;              // Time in animation
  event: string;             // Event name
  data?: any;
}

// Skeleton attack animation events
const SKELETON_ATTACK_EVENTS: AnimationEvent[] = [
  { time: 0.0, event: 'windup_start' },
  { time: 0.25, event: 'sfx_whoosh' },
  { time: 0.3, event: 'damage_window_start' },
  { time: 0.5, event: 'damage_window_end' },
  { time: 0.8, event: 'recovery_complete' }
];
```

## Debug Visualization

### AI Debug Display (Development Only)

```typescript
function drawAIDebug(monster: Monster): void {
  // Draw aggro range
  drawCircle(monster.position, monster.aggroRange, Color.yellow);

  // Draw leash range
  drawCircle(monster.spawnPoint, monster.leashRange, Color.red);

  // Draw current path
  if (monster.currentPath) {
    drawPath(monster.currentPath, Color.green);
  }

  // Draw state label
  drawText(monster.position + Vector3.up * 2, monster.state);

  // Draw threat values
  for (const [playerId, entry] of monster.aggroTable) {
    const player = getPlayer(playerId);
    drawLine(monster.position, player.position, Color.orange);
    drawText(midpoint, `Threat: ${entry.threat.toFixed(0)}`);
  }
}
```

## Testing Requirements

### Functional Tests

- [ ] All states transition correctly
- [ ] Pathfinding navigates around obstacles
- [ ] Leash mechanics prevent infinite chasing
- [ ] Attack patterns execute with correct timing
- [ ] Multi-player threat system works correctly
- [ ] Mini-boss phases transition at correct HP

### Performance Tests

- [ ] 100 monsters update within 5ms budget
- [ ] Pathfinding doesn't cause frame spikes
- [ ] Memory stable over extended sessions
- [ ] No AI entities left behind on death

### Edge Case Tests

- [ ] Stuck detection and recovery works
- [ ] Disconnect handling doesn't crash
- [ ] Invalid targets are handled gracefully
- [ ] Simultaneous aggro from multiple monsters

## Success Metrics

| Metric | Target |
|--------|--------|
| AI update time (100 monsters) | < 5ms |
| Pathfinding success rate | > 99% |
| Stuck occurrence rate | < 0.1% of monsters |
| Player-reported AI issues | < 2% of feedback |

## Dependencies

- **GAME-015**: NPC Monster System (monster entities)
- **ART-006**: NPC Monster Models (animations)
- **ART-027**: Sound Effects (AI sounds)
