# NPC Monster System Specification

## Overview

This document specifies the NPC monster system for Plunderstorm Mobile, providing PvE content through monster camps scattered across the map. Monsters offer alternative XP sources and ability drops, accommodating various playstyles.

## Monster Types

### 1. Skeleton Pirates (Common)

**Description:** Undead crew members that patrol in small groups. Easy targets for early-game XP farming.

| Property | Value |
|----------|-------|
| HP | 50 |
| Damage | 8 per hit |
| Attack Speed | 1.0s |
| Attack Range | Melee (2m) |
| Movement Speed | 4 m/s |
| Aggro Range | 8m |
| Leash Range | 20m |
| Group Size | 3-5 |

**Behavior:**
- Patrols small area when idle
- Charges toward player when aggro'd
- Simple melee attacks
- Returns to spawn if player exceeds leash range

**Visual Design:**
- Tattered pirate clothes
- Glowing eye sockets
- Cutlass weapon (non-functional cosmetic)
- Shambling walk animation

### 2. Sea Crabs (Common)

**Description:** Large crustaceans that guard beach areas. Guaranteed ability drop makes them valuable targets.

| Property | Value |
|----------|-------|
| HP | 80 |
| Damage | 15 per hit |
| Attack Speed | 1.5s |
| Attack Range | Melee (2.5m) |
| Movement Speed | 3 m/s |
| Aggro Range | 6m |
| Leash Range | 15m |
| Group Size | 1-2 |

**Behavior:**
- Burrows partially into sand when idle
- Emerges and attacks when player approaches
- Claw pinch attack with brief stun (0.3s)
- Higher HP makes them tankier than skeletons

**Visual Design:**
- Oversized crab (1.5m tall)
- Barnacle-covered shell
- Red/orange coloration
- Snapping claw animations

### 3. Cursed Sailors (Uncommon)

**Description:** Ghostly mariners with ranged attacks. Higher risk but better rewards.

| Property | Value |
|----------|-------|
| HP | 100 |
| Damage | 20 per hit |
| Attack Speed | 2.0s |
| Attack Range | Ranged (12m) |
| Movement Speed | 3.5 m/s |
| Aggro Range | 12m |
| Leash Range | 25m |
| Group Size | 2-3 |

**Behavior:**
- Floats eerily when idle
- Fires spectral projectile at player
- Maintains distance (kites away if player closes)
- Projectile has travel time (can be dodged)

**Visual Design:**
- Translucent ghostly appearance
- Tattered naval uniform
- Ethereal blue glow
- Flintlock pistol (ghostly)

### 4. Mini-Bosses (Rare)

**Description:** Powerful unique enemies at special locations. High risk, epic rewards.

#### 4a. Captain Deadbeard (Skeleton Boss)

| Property | Value |
|----------|-------|
| HP | 500 |
| Damage | 30-50 per hit |
| Attack Speed | Variable |
| Attack Range | Melee + Ranged |
| Movement Speed | 5 m/s |
| Aggro Range | 15m |
| Leash Range | 30m |

**Attack Patterns:**
1. **Cutlass Combo:** 3-hit melee chain (30 damage each)
2. **Pistol Shot:** Ranged attack (40 damage, 2s cooldown)
3. **Summon Crew:** Spawns 2 Skeleton Pirates (30s cooldown)
4. **Captain's Roar:** AoE fear (3s, 8m radius, 45s cooldown)

#### 4b. The Kraken Spawn (Tentacle Boss)

| Property | Value |
|----------|-------|
| HP | 600 |
| Damage | 40-60 per hit |
| Attack Speed | Variable |
| Attack Range | Melee (extended) |
| Movement Speed | Stationary |
| Aggro Range | 20m |

**Attack Patterns:**
1. **Tentacle Slam:** Ground pound AoE (40 damage, 5m radius)
2. **Tentacle Sweep:** Linear attack (50 damage, 10m line)
3. **Ink Cloud:** Vision block AoE (5s duration, 8m radius)
4. **Grab:** Single target root + damage over time (20/s for 3s)

## Monster Camp Locations

### Camp Distribution

| Zone Type | Camp Count | Monster Types |
|-----------|------------|---------------|
| Beach Areas | 6 | Sea Crabs |
| Forest/Jungle | 8 | Skeleton Pirates |
| Ruins/Structures | 4 | Cursed Sailors |
| Named Locations | 3 | Mixed + Mini-Boss |

### Camp Layout Example

```
[Beach Camp - Sea Crabs]
┌─────────────────────────────┐
│    🦀                       │
│          🦀                 │
│  [Sand Dune]                │
│               Spawn Point   │
│    [Palm Tree]        ⚓    │
│                             │
└─────────────────────────────┘

[Forest Camp - Skeleton Pirates]
┌─────────────────────────────┐
│  💀    [Campfire]    💀    │
│           🔥               │
│    💀          💀          │
│  [Tent]    Spawn    💀     │
│           Point            │
└─────────────────────────────┘
```

### Mini-Boss Locations

| Boss | Location | Respawn Time |
|------|----------|--------------|
| Captain Deadbeard | Shipwreck Cove | 5 minutes |
| The Kraken Spawn | Whirlpool Bay | 6 minutes |
| Ghost Admiral | Haunted Lighthouse | 5 minutes |

## XP and Rewards

### XP Values

| Monster Type | Base XP | Per-Level Bonus |
|--------------|---------|-----------------|
| Skeleton Pirate | 15 | +2 per player level |
| Sea Crab | 25 | +3 per player level |
| Cursed Sailor | 40 | +5 per player level |
| Mini-Boss | 200 | +20 per player level |

### Drop Tables

#### Skeleton Pirates

| Drop | Chance | Notes |
|------|--------|-------|
| Nothing | 70% | - |
| Gold (10-25) | 20% | Soft currency |
| Common Ability | 10% | Rank 1 only |

#### Sea Crabs

| Drop | Chance | Notes |
|------|--------|-------|
| Common Ability | 60% | Rank 1 |
| Uncommon Ability | 30% | Rank 1-2 |
| Rare Ability | 10% | Rank 1-2 |

#### Cursed Sailors

| Drop | Chance | Notes |
|------|--------|-------|
| Nothing | 40% | - |
| Gold (25-50) | 25% | Soft currency |
| Uncommon Ability | 25% | Rank 1-2 |
| Rare Ability | 10% | Rank 2 |

#### Mini-Bosses

| Drop | Chance | Notes |
|------|--------|-------|
| Epic Ability | 70% | Rank 2-3 |
| Legendary Ability | 25% | Rank 3 |
| Gold (100-200) | 100% | Always drops |
| Special Item | 5% | Cosmetic or bonus |

### Drop Mechanics

```typescript
interface MonsterDrop {
  type: 'ability' | 'gold' | 'item';
  abilityId?: string;
  abilityRank?: number;
  goldAmount?: number;
  itemId?: string;
}

function rollMonsterDrop(
  monsterType: MonsterType,
  killerLevel: number
): MonsterDrop[] {
  const dropTable = DROP_TABLES[monsterType];
  const drops: MonsterDrop[] = [];

  for (const entry of dropTable) {
    if (Math.random() < entry.chance) {
      drops.push(generateDrop(entry, killerLevel));
    }
  }

  return drops;
}
```

## Spawn System

### Initial Spawn

- All camps spawn at match start
- Monsters spawn in predefined formations
- Spawn is staggered (0.5s between each monster)

### Respawn Mechanics

| Monster Type | Respawn Timer | Notes |
|--------------|---------------|-------|
| Skeleton Pirates | 60 seconds | Full group respawns |
| Sea Crabs | 75 seconds | Individual respawn |
| Cursed Sailors | 90 seconds | Full group respawns |
| Mini-Bosses | 5-6 minutes | Global announcement |

### Respawn State Machine

```
[Alive] --death--> [Dead] --timer--> [Respawning] --spawn--> [Alive]

Dead State:
- Monster removed from world
- Respawn timer starts
- Spawn point marked as empty

Respawning State:
- Visual effect at spawn point (dust cloud, ghostly shimmer)
- 2 second spawn animation
- Monster invulnerable during animation
```

### Storm Interaction

- Monsters inside storm take damage (same as players)
- Monsters do NOT flee from storm
- Camps in storm become inactive (no respawn until storm passes)
- Mini-bosses immune to storm damage

## Combat Mechanics

### Aggro System

```typescript
interface AggroEntry {
  playerId: string;
  threatValue: number;
  lastDamageTime: number;
}

class MonsterAggroTable {
  entries: Map<string, AggroEntry> = new Map();

  addThreat(playerId: string, amount: number): void {
    const entry = this.entries.get(playerId) || {
      playerId,
      threatValue: 0,
      lastDamageTime: 0
    };
    entry.threatValue += amount;
    entry.lastDamageTime = Date.now();
    this.entries.set(playerId, entry);
  }

  getTopThreat(): string | null {
    let highest = 0;
    let target = null;
    for (const [id, entry] of this.entries) {
      if (entry.threatValue > highest) {
        highest = entry.threatValue;
        target = id;
      }
    }
    return target;
  }

  decayThreat(): void {
    const now = Date.now();
    for (const [id, entry] of this.entries) {
      // Decay threat if no damage for 5 seconds
      if (now - entry.lastDamageTime > 5000) {
        entry.threatValue *= 0.9;
      }
      // Remove if threat too low
      if (entry.threatValue < 1) {
        this.entries.delete(id);
      }
    }
  }
}
```

### Threat Generation

| Action | Threat Generated |
|--------|------------------|
| Enter aggro range | 10 (proximity) |
| Basic attack hit | Damage dealt |
| Ability hit | Damage dealt × 1.5 |
| Healing (in combat) | Heal amount × 0.5 |

### Multi-Player Combat

- Monster attacks highest threat target
- All players who dealt damage share XP on kill
- XP split: Killer gets 50%, rest split by damage %
- Drops go to killing blow player
- Last hit determines drop ownership

### Kiting Rules

- Monsters have maximum chase range (leash)
- Exceeding leash causes monster to return
- Returning monsters are invulnerable
- Health resets when monster returns to spawn
- Threat table clears on leash break

## Health Bar System

### Health Bar Display

```
[Monster Nameplate]
┌─────────────────────────────────┐
│  SKELETON PIRATE                │
│  ████████████░░░░░░  50/50 HP   │
│  [Skull Icon] Common            │
└─────────────────────────────────┘

[Mini-Boss Nameplate]
┌─────────────────────────────────┐
│  ⚔️ CAPTAIN DEADBEARD ⚔️        │
│  ██████████████░░░  350/500 HP  │
│  [Crown Icon] MINI-BOSS         │
│  [Phase 2 of 3]                 │
└─────────────────────────────────┘
```

### Health Bar Visibility

- Health bars appear when:
  - Player deals damage to monster
  - Monster deals damage to player
  - Player aims at monster (soft target)
- Health bars fade after 5 seconds out of combat
- Mini-boss health bars visible to all nearby players

## Server Architecture

### Monster Entity

```typescript
interface MonsterEntity {
  id: string;
  type: MonsterType;
  campId: string;
  position: Vector3;
  rotation: number;
  currentHP: number;
  maxHP: number;
  state: MonsterState;
  targetPlayerId: string | null;
  aggroTable: AggroTable;
  spawnPoint: Vector3;
  lastAttackTime: number;
  respawnTime: number | null;  // null if alive
}

enum MonsterState {
  IDLE = 'idle',
  PATROL = 'patrol',
  CHASE = 'chase',
  ATTACK = 'attack',
  RETURN = 'return',
  DEAD = 'dead',
  RESPAWNING = 'respawning'
}
```

### Server Update Loop

```typescript
function updateMonsters(deltaTime: number): void {
  for (const monster of activeMonsters) {
    switch (monster.state) {
      case MonsterState.IDLE:
        checkForAggroTargets(monster);
        break;
      case MonsterState.PATROL:
        updatePatrol(monster, deltaTime);
        checkForAggroTargets(monster);
        break;
      case MonsterState.CHASE:
        updateChase(monster, deltaTime);
        checkAttackRange(monster);
        checkLeashRange(monster);
        break;
      case MonsterState.ATTACK:
        updateAttack(monster, deltaTime);
        break;
      case MonsterState.RETURN:
        updateReturn(monster, deltaTime);
        break;
      case MonsterState.DEAD:
        checkRespawnTimer(monster);
        break;
    }
  }
}
```

### Network Sync

```typescript
interface MonsterSyncPacket {
  monsterId: string;
  position: Vector3;
  rotation: number;
  state: MonsterState;
  targetId: string | null;
  currentHP: number;
  animationState: string;
}

// Sync frequency
const MONSTER_SYNC_RATE = 100; // 10 Hz
const MONSTER_COMBAT_SYNC_RATE = 50; // 20 Hz when in combat
```

## Performance Optimization

### Monster Budget

| Metric | Budget |
|--------|--------|
| Max active monsters | 100 per match |
| AI update rate | 10 Hz (idle), 20 Hz (combat) |
| Pathfinding calls | Max 10 per frame |
| Network bandwidth | 2KB/s per monster in combat |

### LOD System

| Distance | Update Rate | Animation | Network Sync |
|----------|-------------|-----------|--------------|
| 0-30m | Full (20 Hz) | Full | Full |
| 30-60m | Reduced (10 Hz) | Reduced | Position only |
| 60m+ | Minimal (5 Hz) | Idle only | None |

### Spatial Partitioning

```typescript
class MonsterSpatialGrid {
  cellSize: number = 20; // meters
  cells: Map<string, MonsterEntity[]> = new Map();

  getNearbyMonsters(position: Vector3, radius: number): MonsterEntity[] {
    const cellKeys = this.getCellsInRadius(position, radius);
    const monsters: MonsterEntity[] = [];

    for (const key of cellKeys) {
      const cell = this.cells.get(key);
      if (cell) {
        for (const monster of cell) {
          if (Vector3.Distance(monster.position, position) <= radius) {
            monsters.push(monster);
          }
        }
      }
    }
    return monsters;
  }
}
```

## Audio Design

### Monster Sounds

| Monster | Idle | Aggro | Attack | Death |
|---------|------|-------|--------|-------|
| Skeleton Pirate | Bone rattle | Roar | Sword swing | Bone collapse |
| Sea Crab | Clicking | Screech | Claw snap | Shell crack |
| Cursed Sailor | Ghostly moan | Wail | Gunshot | Fade out |
| Mini-Boss | Ambient theme | Boss music | Unique per attack | Epic death |

### Audio Zones

- Monster ambient sounds audible within 15m
- Combat sounds audible within 30m
- Mini-boss music audible within 50m

## Visual Effects

### Combat VFX

| Monster | Attack VFX | Hit VFX | Death VFX |
|---------|------------|---------|-----------|
| Skeleton | Sword trail | Bone sparks | Bone explosion |
| Sea Crab | None | Shell particles | Shell fragments |
| Cursed Sailor | Ghost bullet trail | Spirit impact | Fade dissolve |
| Mini-Boss | Unique per attack | Heavy impact | Elaborate death |

### State Indicators

- Aggro'd: Red eye glow / exclamation mark
- Returning: Blue trail, ghost effect
- Respawning: Spawn point glow, particle buildup

## Analytics Events

```typescript
// Monster kill tracking
analytics.track("monster_killed", {
  monsterType: string,
  campId: string,
  killerPlayerId: string,
  damageContributors: string[],
  timeToKill: number,
  matchTime: number
});

// Drop tracking
analytics.track("monster_drop", {
  monsterType: string,
  dropType: string,
  abilityId?: string,
  abilityRank?: number,
  playerId: string
});

// Camp engagement
analytics.track("camp_engagement", {
  campId: string,
  playersInvolved: number,
  success: boolean,
  matchTime: number
});
```

## Testing Requirements

### Functional Tests

- [ ] All monster types spawn correctly
- [ ] Aggro triggers at correct range
- [ ] Monsters deal correct damage
- [ ] XP awards correctly to all contributors
- [ ] Drops appear and can be picked up
- [ ] Respawn timers work correctly
- [ ] Leash mechanics prevent infinite kiting

### Performance Tests

- [ ] 100 active monsters maintain 60fps (server)
- [ ] AI updates don't cause frame spikes
- [ ] Network bandwidth within budget
- [ ] Memory usage stable over long matches

### Balance Tests

- [ ] Monster XP competitive with PvP kills
- [ ] Drop rates produce healthy economy
- [ ] Mini-bosses defeatable solo (skill-based)
- [ ] Monster damage doesn't one-shot players

## Implementation Phases

### Phase 1: Basic Monsters
- Skeleton Pirates implementation
- Basic aggro and combat
- Simple drops

### Phase 2: Full Roster
- Sea Crabs and Cursed Sailors
- Complete drop tables
- Health bar UI

### Phase 3: Mini-Bosses
- Boss encounters
- Multi-phase combat
- Epic loot drops

### Phase 4: Polish
- VFX and audio
- Performance optimization
- Balance tuning

## Success Metrics

| Metric | Target |
|--------|--------|
| PvE engagement | 60%+ players kill at least one monster |
| Mini-boss attempts | 30%+ matches have boss contested |
| PvE vs PvP XP | 30-40% of avg player XP from PvE |
| Monster camp complaints | <5% feedback about camp balance |

## Dependencies

- **ART-006**: NPC Monster Models
- **GAME-016**: NPC AI Behavior System
- **GAME-004**: Loot Spawn System (drop mechanics)
- **GAME-012**: Damage and Health System
