# Crowd Control System Specification

## Overview

This document specifies the crowd control (CC) system for Plunderstorm Mobile, including all CC types, diminishing returns, immunity windows, and visual feedback systems to ensure fair and readable combat interactions.

## CC Type Definitions

### 1. Stun

**Effect:** Target cannot move or use abilities. Basic attacks also disabled.

| Property | Value |
|----------|-------|
| Movement | Disabled |
| Abilities | Disabled |
| Basic Attack | Disabled |
| Camera | Player retains control |
| Typical Duration | 0.5s - 2.0s |

**Visual Indicators:**
- Yellow stars circling above head
- Character model plays stunned idle animation
- Yellow outline glow on character
- "STUNNED" text briefly appears

**Audio:**
- Impact sound on application
- Dazed/ringing loop while active

### 2. Slow

**Effect:** Reduced movement speed. Abilities and attacks still usable.

| Property | Value |
|----------|-------|
| Movement | Reduced (25-75%) |
| Abilities | Enabled |
| Basic Attack | Enabled |
| Camera | Normal |
| Typical Duration | 2.0s - 5.0s |

**Slow Tiers:**
| Tier | Speed Reduction | Examples |
|------|-----------------|----------|
| Minor | 25% | Glancing hits, area effects |
| Moderate | 50% | Direct ability hits |
| Severe | 75% | Ultimate abilities, stacking |

**Visual Indicators:**
- Blue ice/frost particles on feet
- Movement trail shows blue tint
- Blue debuff icon with percentage
- Character animation plays at reduced speed

**Audio:**
- Crystalline/frost sound on application
- Muffled footstep sounds while active

### 3. Root

**Effect:** Target cannot move but can use abilities and attack.

| Property | Value |
|----------|-------|
| Movement | Disabled |
| Abilities | Enabled |
| Basic Attack | Enabled |
| Camera | Normal |
| Typical Duration | 1.0s - 3.0s |

**Visual Indicators:**
- Green vines/seaweed wrapping feet
- Ground crack effect at character position
- Green glow on lower body
- Character upper body animates normally

**Audio:**
- Vine/rope tightening sound
- Struggle sounds from character

### 4. Knockback

**Effect:** Forced movement away from source. Brief loss of control.

| Property | Value |
|----------|-------|
| Movement | Forced direction |
| Abilities | Disabled during travel |
| Basic Attack | Disabled during travel |
| Camera | Follows character |
| Typical Duration | 0.3s - 0.8s (travel time) |

**Knockback Properties:**
| Property | Range |
|----------|-------|
| Distance | 3m - 10m |
| Travel Speed | 15-25 m/s |
| Recovery Time | 0.2s after landing |

**Visual Indicators:**
- Wind/force trail during travel
- Impact dust cloud on landing
- Character plays thrown animation
- Screen shake on landing (self only)

**Audio:**
- Whoosh sound during travel
- Impact thud on landing
- Character grunt/yell

### 5. Silence

**Effect:** Cannot use abilities. Movement and basic attacks allowed.

| Property | Value |
|----------|-------|
| Movement | Enabled |
| Abilities | Disabled |
| Basic Attack | Enabled |
| Camera | Normal |
| Typical Duration | 2.0s - 4.0s |

**Visual Indicators:**
- Purple X over character's head
- Ability buttons show locked/grayed state
- Purple mist around character hands
- "SILENCED" text briefly appears

**Audio:**
- Magical suppression sound
- Muffled ambient audio while active

## Ability CC Assignments

### Abilities with CC Effects

| Ability | CC Type | Duration (Base) | Notes |
|---------|---------|-----------------|-------|
| Wind Burst | Knockback | 5m distance | Rank 3: 7.5m |
| Smoke Bomb | Slow (50%) | 3s | While in smoke |
| Grappling Hook | Stun | 0.5s | Rank 3 only |
| Blunderbuss | Knockback | 3m distance | Rank 3: 4.5m |
| Lightning Strike | Stun | 1.0s | Center hit only |
| Cutlass Slash | Slow (25%) | 1.5s | Rank 3 bleed slow |

### Basic Attack CC
- No inherent CC effects
- Some future items/perks may add minor effects

## Diminishing Returns (DR) System

### DR Mechanics

Repeated application of the same CC type within a time window results in reduced effectiveness.

```
DR Formula:
effective_duration = base_duration * DR_multiplier

DR Multiplier Table:
- 1st application: 100% (full duration)
- 2nd application (within 15s): 50%
- 3rd application (within 15s): 25%
- 4th+ application (within 15s): Immune (0%)
```

### DR Categories

CC types are grouped into DR categories. Same-category CCs share DR.

| Category | CC Types |
|----------|----------|
| Hard CC | Stun, Root |
| Displacement | Knockback |
| Soft CC | Slow, Silence |

**Example:**
1. Player hit by Stun (1.0s) → Full 1.0s duration
2. Player hit by Root within 15s → 50% = 0.5s (same category)
3. Player hit by Slow → Full duration (different category)

### DR Window Reset

- DR window: 15 seconds from first CC application
- After window expires, DR fully resets
- Each CC category has independent DR tracking

### DR State Tracking

```typescript
interface DRState {
  category: DRCategory;
  applicationCount: number;
  windowStartTime: number;
  windowDuration: number;  // 15000ms
}

interface PlayerCCState {
  hardCC: DRState;
  displacement: DRState;
  softCC: DRState;
}
```

## CC Immunity System

### Post-CC Immunity

After extended CC, targets receive brief immunity to prevent chain-locking.

| Trigger Condition | Immunity Duration | Immunity Type |
|-------------------|-------------------|---------------|
| 2+ seconds of Hard CC in 10s | 2.0s | Hard CC only |
| 3+ knockbacks in 10s | 1.5s | Displacement only |
| Full DR reached (4+ hits) | 3.0s | That category |

### Ability-Granted Immunity

Some abilities grant CC immunity during their effect:

| Ability | Immunity Window | Immunity Type |
|---------|-----------------|---------------|
| Barrel Roll | During roll (0.5s) | All CC |
| Healing Tide | During cast (0.3s) | Displacement |
| Grappling Hook | During travel | Knockback |

### Immunity Visual Feedback

- Golden shield icon above character
- Brief golden glow on character
- "IMMUNE" text on blocked CC attempts
- Distinct sound when CC is negated

## CC Interaction Rules

### CC Stacking

| Scenario | Result |
|----------|--------|
| Same CC type applied | Duration refreshes (doesn't stack) |
| Different CC types | Both apply simultaneously |
| Multiple slows | Strongest slow applies (no stacking) |
| Stun during Root | Root ends, Stun applies |
| Knockback during Stun | Stun ends, Knockback applies |

### CC Priority

When multiple CCs would conflict, higher priority CC takes precedence:

1. Knockback (always displaces)
2. Stun (overrides root)
3. Root (overrides slow)
4. Silence (independent)
5. Slow (lowest priority)

### Terrain Interactions

**Knockback Terrain Rules:**
- Wall collision: Stop at wall, take 10% max HP damage
- Water edge: Stop at edge, no water entry
- Cliff edge: Stop at edge, no fall damage
- Destructible objects: Pass through, destroy object

```typescript
function calculateKnockbackDestination(
  origin: Vector3,
  direction: Vector3,
  distance: number
): KnockbackResult {
  const targetPos = origin + (direction.normalized * distance);

  // Raycast for obstacles
  const hit = Physics.Raycast(origin, direction, distance, COLLISION_MASK);

  if (hit) {
    return {
      finalPosition: hit.point - (direction.normalized * 0.5),
      hitObstacle: true,
      bonusDamage: hit.isWall ? calculateWallDamage() : 0
    };
  }

  return {
    finalPosition: clampToNavMesh(targetPos),
    hitObstacle: false,
    bonusDamage: 0
  };
}
```

## UI Components

### Debuff Display

```
┌─────────────────────────────────────────┐
│  Character Nameplate                    │
├─────────────────────────────────────────┤
│  [Health Bar]                           │
│  ┌────┐ ┌────┐ ┌────┐                  │
│  │STUN│ │SLOW│ │ROOT│   ← Debuff icons │
│  │1.2s│ │ 50%│ │2.0s│   ← Duration/% │
│  └────┘ └────┘ └────┘                  │
└─────────────────────────────────────────┘
```

### Self Debuff Display (HUD)

```
┌──────────────────────────┐
│   STUNNED                │
│   ████████░░ 1.2s        │  ← Progress bar
│   [Break free: Tap rapidly] │  ← Optional mechanic
└──────────────────────────┘
```

### DR Indicator

When DR reduces a CC:

```
┌────────────────────┐
│  DIMINISHED!       │
│  Stun: 1.0s → 0.5s │
└────────────────────┘
```

## Network Synchronization

### CC Application Packet

```typescript
interface CCApplicationEvent {
  targetId: string;
  sourceId: string;
  ccType: CCType;
  baseDuration: number;
  effectiveDuration: number;  // After DR
  drMultiplier: number;
  timestamp: number;
  knockbackData?: {
    direction: Vector3;
    distance: number;
    force: number;
  };
}
```

### Server Authority

- Server calculates all CC durations including DR
- Server validates CC immunity states
- Client receives authoritative CC state
- Client handles visual/audio feedback locally

### CC State Sync

```typescript
interface PlayerCCSyncState {
  playerId: string;
  activeEffects: {
    ccType: CCType;
    remainingDuration: number;
    sourceId: string;
  }[];
  immunities: {
    category: DRCategory;
    remainingDuration: number;
  }[];
  drStates: {
    category: DRCategory;
    applicationCount: number;
    windowRemaining: number;
  }[];
}
```

### Lag Compensation

- CC application uses hit time, not receive time
- Maximum rollback: 200ms
- CC that would have expired is not applied

## NPC CC Interactions

### NPC CC Vulnerability

| NPC Type | Stun | Slow | Root | Knockback | Silence |
|----------|------|------|------|-----------|---------|
| Skeleton Pirates | Full | Full | Full | Reduced (50%) | N/A |
| Sea Crabs | Full | Reduced | Full | Full | N/A |
| Cursed Sailors | Reduced | Full | Reduced | Full | N/A |
| Mini-Bosses | Reduced | Reduced | Immune | Reduced | N/A |

**Reduced:** 50% duration
**Immune:** No effect

### Boss CC Immunity Phases

Mini-bosses gain temporary CC immunity during special attacks:
- Wind-up animations: Immune to all CC
- Recovery frames: Full CC vulnerability
- Enrage phase: Reduced CC duration (50%)

## Balance Parameters

### CC Duration Limits

| CC Type | Minimum | Maximum | Recommended |
|---------|---------|---------|-------------|
| Stun | 0.3s | 2.0s | 0.5-1.5s |
| Slow | 1.0s | 5.0s | 2.0-3.0s |
| Root | 0.5s | 3.0s | 1.0-2.0s |
| Knockback | 3m | 10m | 5-7m |
| Silence | 1.0s | 4.0s | 2.0-3.0s |

### DR Tuning Variables

```typescript
const DR_CONFIG = {
  windowDuration: 15000,        // 15 seconds
  multipliers: [1.0, 0.5, 0.25, 0],
  immunityDuration: {
    hardCC: 2000,               // 2 seconds
    displacement: 1500,         // 1.5 seconds
    softCC: 1500                // 1.5 seconds
  },
  immunityTrigger: {
    hardCC_duration: 2000,      // 2s of hard CC triggers immunity
    displacement_count: 3,      // 3 knockbacks trigger immunity
    softCC_duration: 4000       // 4s of soft CC triggers immunity
  }
};
```

## Animation Requirements

### CC Animation States

| CC Type | Animation Name | Duration | Interruptible |
|---------|----------------|----------|---------------|
| Stun | `anim_stunned_idle` | Loop | By CC end |
| Slow | `anim_walk_slow` | Loop | Normal |
| Root | `anim_rooted_struggle` | Loop | By CC end |
| Knockback | `anim_knockback_air` | Travel time | No |
| Silence | `anim_idle_silenced` | Loop | Normal |

### Transition Animations

| From → To | Transition Animation |
|-----------|---------------------|
| Any → Stun | `anim_stun_hit` (0.15s) |
| Stun → Normal | `anim_stun_recover` (0.2s) |
| Any → Knockback | `anim_knockback_start` (0.1s) |
| Knockback → Normal | `anim_knockback_land` (0.25s) |
| Root → Normal | `anim_root_break` (0.2s) |

## Audio Design

### CC Sound Effects

| CC Type | Application SFX | Loop SFX | End SFX |
|---------|-----------------|----------|---------|
| Stun | `sfx_stun_impact` | `sfx_stun_loop` | `sfx_stun_recover` |
| Slow | `sfx_frost_apply` | `sfx_frost_ambient` | `sfx_frost_thaw` |
| Root | `sfx_vine_grab` | `sfx_vine_strain` | `sfx_vine_break` |
| Knockback | `sfx_knockback_hit` | `sfx_wind_travel` | `sfx_knockback_land` |
| Silence | `sfx_silence_apply` | `sfx_silence_muffle` | `sfx_silence_lift` |

### DR/Immunity Audio

| Event | Sound Effect |
|-------|--------------|
| CC reduced by DR | `sfx_diminish_proc` |
| CC blocked by immunity | `sfx_immunity_block` |
| Immunity gained | `sfx_immunity_gain` |

## Testing Requirements

### Functional Tests

- [ ] Each CC type applies correct movement/ability restrictions
- [ ] CC durations match specifications
- [ ] DR reduces subsequent CC correctly
- [ ] DR window resets after 15 seconds
- [ ] Immunity triggers at correct thresholds
- [ ] Ability-granted immunity works correctly
- [ ] CC stacking/priority works as specified
- [ ] Knockback terrain collisions work correctly

### Visual Tests

- [ ] Each CC has distinct visual indicator
- [ ] Duration timers display accurately
- [ ] DR reduction shown to players
- [ ] Immunity shield displays on activation
- [ ] Animations transition smoothly

### Network Tests

- [ ] CC syncs correctly across clients
- [ ] High latency doesn't extend CC duration
- [ ] Server authority prevents CC exploitation
- [ ] DR state syncs correctly after reconnect

### Balance Tests

- [ ] No infinite CC chains possible
- [ ] CC doesn't feel frustrating in normal gameplay
- [ ] DR prevents permastun scenarios
- [ ] NPCs react appropriately to CC

## Implementation Phases

### Phase 1: Core CC Types
- Implement Stun, Slow, Root mechanics
- Basic visual indicators
- Server-authoritative validation

### Phase 2: Advanced CC
- Knockback with terrain collision
- Silence effect
- DR system implementation

### Phase 3: Polish
- Full visual/audio feedback
- Immunity system
- UI components for debuffs

### Phase 4: Balance
- Tune durations and DR values
- Playtest all CC combinations
- Adjust based on feedback

## Success Metrics

| Metric | Target |
|--------|--------|
| CC feels fair | <10% of feedback mentions CC frustration |
| DR prevents permastun | 0 reports of infinite CC chains |
| Visual clarity | 90%+ players can identify CC type |
| Network accuracy | CC desync reports <1% of matches |

## Dependencies

- **GAME-008**: Core Ability Implementation (abilities that apply CC)
- **GAME-012**: Damage and Health System (CC interaction with combat)
- **ART-013**: Ability VFX Set (CC visual effects)
- **ART-027**: Sound Effects (CC audio)
