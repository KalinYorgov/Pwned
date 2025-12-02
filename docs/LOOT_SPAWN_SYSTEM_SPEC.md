# Loot Spawn System Specification

## Document Information
- **Task ID:** GAME-004
- **Priority:** P0
- **Complexity:** M
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the loot spawn system for Plunderstorm Mobile. The system controls how abilities appear throughout the map via treasure chests and NPC drops, ensuring balanced distribution and fair early-game opportunities.

### 1.2 Scope
- Loot spawn point distribution
- Treasure chest mechanics and rarity tiers
- Loot table configuration
- Chest interaction and opening sequence
- NPC loot drops
- Ability auto-equip and pickup flow
- Server-authoritative loot management
- Network synchronization

### 1.3 Dependencies
- GAME-005: Ability System
- GAME-007: Ability Slot System
- ART-010: Treasure Chest Assets
- BACK-003: Game Server Infrastructure

---

## 2. Spawn Point System

### 2.1 Spawn Point Distribution

```
┌─────────────────────────────────────────────────────────────────┐
│                    MAP SPAWN POINT LAYOUT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Map divided into zones with spawn density tiers:                │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                                                           │   │
│  │     ○ ○       ●●●●●        ○ ○                           │   │
│  │    ○   ○      ● POI ●     ○   ○                          │   │
│  │     ○ ○       ●●●●●        ○ ○                           │   │
│  │                                                           │   │
│  │  ○ ○ ○        ●●●●●        ●●●●●        ○ ○ ○            │   │
│  │               ● POI ●      ● POI ●                        │   │
│  │               ●●●●●        ●●●●●                          │   │
│  │                                                           │   │
│  │     ○ ○ ○              ○                    ○ ○ ○         │   │
│  │                                                           │   │
│  │  ●●●●●●●●      ○ ○ ○        ●●●●●        ○ ○ ○           │   │
│  │  ● MAJOR ●                  ● POI ●                       │   │
│  │  ●  POI  ●                  ●●●●●                         │   │
│  │  ●●●●●●●●                                                 │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Legend:                                                         │
│  ● = High-density spawn area (POIs)                              │
│  ○ = Low-density spawn area (wilderness)                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Spawn Point Categories

| Category | Density | Rarity Distribution | Location Type |
|----------|---------|---------------------|---------------|
| Major POI | Very High (15-20 chests) | Better rarities | Port Plunder, main locations |
| Minor POI | High (8-12 chests) | Balanced | Named landmarks |
| Structure | Medium (3-5 chests) | Standard | Buildings, camps |
| Wilderness | Low (1-2 chests) | Lower rarities | Open areas, paths |

### 2.3 Spawn Point Data Structure

```javascript
class SpawnPoint {
    constructor() {
        this.id = generateUUID();
        this.position = Vector3.zero;      // World position
        this.category = 'MINOR_POI';       // Spawn category
        this.rarityPool = 'STANDARD';      // Which loot table to use
        this.respawnable = false;          // Chests don't respawn
        this.contested = false;            // Multiple players nearby
    }
}

// Spawn point configuration
const SPAWN_CONFIG = {
    totalSpawnPoints: 400,       // Across entire map
    minimumSpacing: 5,           // Meters between chests
    poiClusterRadius: 30,        // POI spawn area radius
    wildernessSpacing: 50        // Min distance between wilderness spawns
};
```

---

## 3. Treasure Chest System

### 3.1 Chest Rarity Tiers

| Tier | Probability | Visual Appearance | Contents Quality |
|------|-------------|-------------------|------------------|
| Common | 60% | Wooden chest, bronze trim | Rank 1 abilities |
| Uncommon | 25% | Reinforced chest, silver trim | Rank 1-2 abilities |
| Rare | 12% | Ornate chest, gold trim | Rank 2 abilities |
| Epic | 3% | Glowing chest, purple aura | Rank 2-3 abilities |

### 3.2 Chest Visual Design

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHEST VISUAL STATES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CLOSED (Unopened)          OPENING               OPENED         │
│  ┌─────────────┐          ┌─────────────┐      ┌─────────────┐  │
│  │   ╔═════╗   │          │   ╔═══/═╗   │      │   ╔═══──╗   │  │
│  │   ║     ║   │   ──►    │   ║ glow ║   │ ──► │   ║empty ║   │  │
│  │   ║ ▓▓▓ ║   │          │   ║ ▓▓▓ ║   │      │   ║     ║   │  │
│  │   ╚═════╝   │          │   ╚═════╝   │      │   ╚═════╝   │  │
│  │   [GLOW]    │          │   [BURST]   │      │   [DIM]     │  │
│  └─────────────┘          └─────────────┘      └─────────────┘  │
│                                                                  │
│  Rarity Glow Colors:                                             │
│  - Common: Soft white/cream                                      │
│  - Uncommon: Green glow                                          │
│  - Rare: Blue glow                                               │
│  - Epic: Purple glow with particles                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Chest State Machine

```javascript
enum ChestState {
    SPAWNED,        // Chest exists, unopened
    OPENING,        // Player interacting, channel in progress
    OPENED,         // Contents revealed, ability available
    LOOTED,         // Ability taken, chest empty
    DESTROYED       // Removed from world
}

class TreasureChest {
    state = ChestState.SPAWNED;
    rarity = ChestRarity.COMMON;
    contents = null;         // Ability inside
    openingPlayer = null;    // Who is opening
    openProgress = 0;        // 0-1 progress

    // Constants
    static OPEN_DURATION = 0.75;     // Seconds to open
    static PICKUP_RADIUS = 2.5;      // Meters for auto-pickup
    static VISIBLE_RANGE = 50;       // Meters to see glow
}
```

---

## 4. Loot Tables

### 4.1 Ability Drop Rates by Chest Rarity

```javascript
const LOOT_TABLES = {
    COMMON: {
        // Ability pools with weights
        offensive: {
            weight: 0.5,
            abilities: [
                { id: 'fireball', weight: 20, rankDistribution: [100, 0, 0] },
                { id: 'cutlass_slash', weight: 25, rankDistribution: [100, 0, 0] },
                { id: 'cannonball', weight: 15, rankDistribution: [100, 0, 0] },
                { id: 'lightning_strike', weight: 20, rankDistribution: [100, 0, 0] },
                { id: 'poison_mackerel', weight: 20, rankDistribution: [100, 0, 0] }
            ]
        },
        utility: {
            weight: 0.5,
            abilities: [
                { id: 'grappling_hook', weight: 20, rankDistribution: [100, 0, 0] },
                { id: 'barrel_roll', weight: 25, rankDistribution: [100, 0, 0] },
                { id: 'healing_grog', weight: 20, rankDistribution: [100, 0, 0] },
                { id: 'smoke_bomb', weight: 20, rankDistribution: [100, 0, 0] },
                { id: 'wind_burst', weight: 15, rankDistribution: [100, 0, 0] }
            ]
        }
    },

    UNCOMMON: {
        offensive: {
            weight: 0.5,
            abilities: [
                { id: 'fireball', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'cutlass_slash', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'cannonball', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'lightning_strike', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'poison_mackerel', weight: 20, rankDistribution: [70, 30, 0] }
            ]
        },
        utility: {
            weight: 0.5,
            abilities: [
                { id: 'grappling_hook', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'barrel_roll', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'healing_grog', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'smoke_bomb', weight: 20, rankDistribution: [70, 30, 0] },
                { id: 'wind_burst', weight: 20, rankDistribution: [70, 30, 0] }
            ]
        }
    },

    RARE: {
        offensive: {
            weight: 0.5,
            abilities: [
                { id: 'fireball', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'cutlass_slash', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'cannonball', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'lightning_strike', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'poison_mackerel', weight: 20, rankDistribution: [30, 60, 10] }
            ]
        },
        utility: {
            weight: 0.5,
            abilities: [
                { id: 'grappling_hook', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'barrel_roll', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'healing_grog', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'smoke_bomb', weight: 20, rankDistribution: [30, 60, 10] },
                { id: 'wind_burst', weight: 20, rankDistribution: [30, 60, 10] }
            ]
        }
    },

    EPIC: {
        offensive: {
            weight: 0.5,
            abilities: [
                { id: 'fireball', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'cutlass_slash', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'cannonball', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'lightning_strike', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'poison_mackerel', weight: 20, rankDistribution: [0, 50, 50] }
            ]
        },
        utility: {
            weight: 0.5,
            abilities: [
                { id: 'grappling_hook', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'barrel_roll', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'healing_grog', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'smoke_bomb', weight: 20, rankDistribution: [0, 50, 50] },
                { id: 'wind_burst', weight: 20, rankDistribution: [0, 50, 50] }
            ]
        }
    }
};
```

### 4.2 Rank Distribution Summary

| Chest Rarity | Rank 1 Chance | Rank 2 Chance | Rank 3 Chance |
|--------------|---------------|---------------|---------------|
| Common | 100% | 0% | 0% |
| Uncommon | 70% | 30% | 0% |
| Rare | 30% | 60% | 10% |
| Epic | 0% | 50% | 50% |

---

## 5. Chest Interaction

### 5.1 Opening Sequence

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHEST OPENING FLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Player Approaches                                               │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────────┐                                               │
│  │ Enter Range  │  Distance < 2.5m                               │
│  │ (2.5 meters) │  UI prompt appears: "Open Chest"               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼ [Tap Open Button]                                      │
│  ┌──────────────┐                                               │
│  │ Channel Start│  0.75 second channel                           │
│  │ Progress Bar │  Can be interrupted by damage                  │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ├────── [Interrupted] ────► Return to Closed State       │
│         │                                                        │
│         ▼ [Channel Complete]                                     │
│  ┌──────────────┐                                               │
│  │ Chest Opens  │  Lid animation + VFX burst                     │
│  │ Reveal Loot  │  Ability orb floats up                        │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼ [Auto-pickup if slot available]                        │
│  ┌──────────────┐                                               │
│  │ Ability      │  Equips to empty slot                          │
│  │ Acquired     │  Or prompts swap if full                       │
│  └──────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Interaction Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Interaction Range | 2.5m | Distance to start opening |
| Open Duration | 0.75s | Channel time |
| Interrupt Damage | Any | Any damage cancels channel |
| Auto-pickup Range | 2.5m | Same as interaction range |
| Pickup Radius (dropped) | 1.5m | For abilities on ground |

### 5.3 Mobile UI for Opening

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHEST OPEN UI                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When near chest:                                                │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              [GAME VIEW]                                │     │
│  │                                                         │     │
│  │                   📦                                    │     │
│  │                 [Chest]                                 │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  ┌────────────────┐                                             │
│  │   🔓 OPEN      │  ← Large, easy-to-tap button                │
│  │   (0.75s)      │                                             │
│  └────────────────┘                                             │
│                                                                  │
│  During opening:                                                 │
│  ┌────────────────────────────────────────────────────────┐     │
│  │   █████████░░░░░░░░░░░  45%                             │     │
│  │   OPENING...                                            │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Progress bar fills, cancel by moving away or taking damage      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Interrupt Handling

```javascript
class ChestInteraction {
    onDamageReceived(player, chest) {
        if (chest.openingPlayer === player && chest.state === ChestState.OPENING) {
            // Cancel opening
            chest.state = ChestState.SPAWNED;
            chest.openingPlayer = null;
            chest.openProgress = 0;

            // Play cancel feedback
            playSound('chest_open_cancelled');
            showFloatingText(player, 'Interrupted!');
        }
    }

    onPlayerMoveAway(player, chest) {
        const distance = Vector3.Distance(player.position, chest.position);
        if (distance > TreasureChest.PICKUP_RADIUS + 0.5) {
            // Too far, cancel
            this.cancelOpening(player, chest);
        }
    }
}
```

---

## 6. Ability Pickup Flow

### 6.1 Auto-Equip Logic

```javascript
class AbilityPickup {
    handlePickup(player, ability) {
        const slotType = ability.slotType; // 'OFFENSIVE' or 'UTILITY'
        const availableSlots = player.getSlotsByType(slotType);

        // Check for empty slot
        const emptySlot = availableSlots.find(s => s.ability === null);
        if (emptySlot) {
            // Auto-equip to empty slot
            this.equipAbility(player, emptySlot, ability);
            return;
        }

        // Check for upgrade (same ability)
        const upgradableSlot = availableSlots.find(
            s => s.ability?.id === ability.id && s.ability.rank < 3
        );
        if (upgradableSlot) {
            // Upgrade existing ability
            this.upgradeAbility(player, upgradableSlot);
            return;
        }

        // All slots full, prompt swap
        this.promptSwap(player, ability, availableSlots);
    }
}
```

### 6.2 Swap Prompt UI

```
┌─────────────────────────────────────────────────────────────────┐
│                    ABILITY SWAP PROMPT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                    SWAP ABILITY?                        │     │
│  │                                                         │     │
│  │  ┌─────────────┐          ┌─────────────┐              │     │
│  │  │  [CURRENT]  │    →     │   [NEW]     │              │     │
│  │  │  Fireball   │          │  Cannonball │              │     │
│  │  │  ★★☆ Rank 2 │          │  ★★★ Rank 3 │              │     │
│  │  │  Dmg: 45    │          │  Dmg: 80    │              │     │
│  │  │  CD: 6s     │          │  CD: 10s    │              │     │
│  │  └─────────────┘          └─────────────┘              │     │
│  │                                                         │     │
│  │     [SWAP]                      [KEEP]                  │     │
│  │                                                         │     │
│  │  Timer: 5s (auto-dismiss, keeps current)                │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  On swap: Old ability drops on ground                            │
│  On keep: New ability drops on ground (can be picked up)         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Ground Ability Pickup

```javascript
class GroundAbility {
    static DESPAWN_TIME = 30;        // Seconds before disappearing
    static PICKUP_RADIUS = 1.5;      // Auto-pickup distance
    static VISIBLE_RANGE = 30;       // Can see icon from this far

    constructor(ability, position) {
        this.ability = ability;
        this.position = position;
        this.spawnTime = Time.now();
        this.state = 'ACTIVE';
    }

    update() {
        // Check despawn
        if (Time.now() - this.spawnTime > DESPAWN_TIME) {
            this.despawn();
        }

        // Visual pulse effect
        this.pulseEffect();
    }

    onPlayerEnter(player) {
        // Auto-pickup when player walks over
        AbilityPickup.handlePickup(player, this.ability);
        this.state = 'PICKED_UP';
        this.destroy();
    }
}
```

---

## 7. NPC Loot Drops

### 7.1 Drop Configuration by NPC Type

| NPC Type | Drop Chance | Rarity Distribution | XP Bonus |
|----------|-------------|---------------------|----------|
| Skeleton Pirate | 40% | 80% Common, 20% Uncommon | 20 XP |
| Sea Crab | 100% | 60% Common, 40% Uncommon | 30 XP |
| Cursed Sailor | 75% | 50% Common, 40% Uncommon, 10% Rare | 50 XP |
| Mini-Boss | 100% | 20% Rare, 60% Epic, 20% Legendary | 100 XP |

### 7.2 NPC Drop Mechanics

```javascript
class NPCDropSystem {
    onNPCDeath(npc, killer) {
        // Roll for drop
        const dropRoll = Math.random();
        if (dropRoll > npc.dropChance) {
            return; // No drop
        }

        // Determine rarity
        const rarity = this.rollRarity(npc.rarityDistribution);

        // Generate ability from loot table
        const ability = this.generateAbility(rarity);

        // Spawn ground ability at NPC position
        const groundAbility = new GroundAbility(ability, npc.position);

        // Drop lands with slight physics arc
        groundAbility.applyDropPhysics({
            initialVelocity: new Vector3(
                randomRange(-2, 2),
                5,
                randomRange(-2, 2)
            ),
            gravity: 15
        });

        // Give XP to killer
        killer.addXP(npc.xpReward);
    }
}
```

### 7.3 Drop Physics

```
┌─────────────────────────────────────────────────────────────────┐
│                    NPC DROP ANIMATION                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│     ☠️ NPC Death                                                 │
│         │                                                        │
│         ▼                                                        │
│     ┌───────┐                                                    │
│     │ Burst │  Soul orb VFX + ability pop-up                    │
│     └───┬───┘                                                    │
│         │                                                        │
│         │    ↗  (arc trajectory)                                │
│         │   /                                                    │
│         │  /                                                     │
│         │ ↙                                                      │
│         ▼                                                        │
│     ┌───────┐                                                    │
│     │[ABILITY]│  Lands on ground, pulses                        │
│     │  ORB   │  Auto-pickup radius active                       │
│     └───────┘                                                    │
│                                                                  │
│  Drop arc: 0.5s flight time                                      │
│  Landing radius: 2-3m from death location                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Server Authority

### 8.1 Server-Side Loot Generation

```javascript
class LootServer {
    generateMatchLoot(matchSeed) {
        const rng = new SeededRandom(matchSeed);

        // Generate all chests at match start
        const chests = [];
        for (const spawnPoint of this.spawnPoints) {
            const chest = {
                id: generateUUID(),
                position: spawnPoint.position,
                rarity: this.rollChestRarity(rng, spawnPoint.rarityPool),
                contents: null, // Generated on first open
                state: ChestState.SPAWNED
            };
            chests.push(chest);
        }

        return chests;
    }

    onChestOpened(playerId, chestId) {
        const chest = this.getChest(chestId);
        if (!chest || chest.state !== ChestState.SPAWNED) {
            return { success: false, reason: 'INVALID_CHEST' };
        }

        // Generate contents now (lazy generation)
        if (!chest.contents) {
            chest.contents = this.generateChestContents(chest.rarity);
        }

        // Update state
        chest.state = ChestState.OPENED;
        chest.openedBy = playerId;

        // Broadcast to all players
        this.broadcastChestOpen(chest);

        return { success: true, ability: chest.contents };
    }
}
```

### 8.2 Anti-Cheat Measures

| Protection | Implementation |
|------------|----------------|
| No client loot gen | Server generates all loot |
| Proximity check | Must be within range to open |
| Double-loot prevention | State tracking, single owner |
| Speed hack detection | Validate movement to chest |
| Timing validation | Open duration enforced server-side |

### 8.3 Race Condition Handling

```javascript
class ChestLockSystem {
    attemptOpen(playerId, chestId) {
        const chest = this.chests.get(chestId);

        // Atomic lock acquisition
        if (chest.openingPlayer !== null) {
            return { success: false, reason: 'ALREADY_BEING_OPENED' };
        }

        // Lock the chest
        chest.openingPlayer = playerId;
        chest.openStartTime = Time.serverNow();

        // Start open timer
        setTimeout(() => {
            this.completeOpen(playerId, chestId);
        }, TreasureChest.OPEN_DURATION * 1000);

        return { success: true };
    }

    completeOpen(playerId, chestId) {
        const chest = this.chests.get(chestId);

        // Verify still valid
        if (chest.openingPlayer !== playerId) {
            return; // Was cancelled
        }

        // Complete the open
        chest.state = ChestState.OPENED;
        this.awardLoot(playerId, chest);
    }
}
```

---

## 9. Network Synchronization

### 9.1 Chest State Packet

```javascript
// Broadcast when chest state changes
struct ChestUpdatePacket {
    uint32 chestId;
    uint8  state;           // SPAWNED, OPENING, OPENED, LOOTED
    uint32 openingPlayerId; // Who is opening (0 if none)
    float  openProgress;    // 0-1 progress
    uint8  abilityId;       // What's inside (only on OPENED)
    uint8  abilityRank;     // Rank of ability
}
```

### 9.2 Sync Strategy

| Event | Sync Type | Recipients |
|-------|-----------|------------|
| Match Start | Full chest list | All players |
| Open Start | State update | Nearby players (50m) |
| Open Complete | State + contents | All players |
| Pickup | Removal | Nearby players |

### 9.3 Initial Sync (Match Start)

```javascript
// Sent to each player on match join
struct InitialLootSync {
    uint16 chestCount;
    ChestData[] chests; // Position, rarity, state for each
}

struct ChestData {
    uint32 id;
    float3 position;
    uint8  rarity;      // Visual only
    uint8  state;       // Should all be SPAWNED initially
}
```

---

## 10. Visual and Audio Feedback

### 10.1 Chest Visual Effects

| Event | VFX | Duration |
|-------|-----|----------|
| Chest Spawn | Fade-in + glow start | 0.5s |
| Approaching | Glow intensifies | Continuous |
| Opening | Light rays, particles | 0.75s |
| Open Complete | Burst of coins/sparkles | 0.5s |
| Ability Reveal | Ability orb rises | 1s |
| Looted | Glow dims | 0.3s |

### 10.2 Audio Cues

| Sound | Trigger | Properties |
|-------|---------|------------|
| chest_nearby | Enter 10m radius | Soft chime |
| chest_open_start | Begin opening | Mechanical creak |
| chest_open_loop | During channel | Building tension |
| chest_open_success | Complete open | Triumphant fanfare |
| chest_open_cancel | Interrupted | Clunk, close sound |
| ability_pickup | Equip ability | Magical whoosh |
| ability_upgrade | Upgrade triggered | Power-up sound |

### 10.3 Ability Rarity Indicators

```
Visual cues for ability quality:

Rank 1: ★☆☆
- Simple glow
- White/gray particles
- Soft sound

Rank 2: ★★☆
- Medium glow
- Blue particles
- Clear chime

Rank 3: ★★★
- Bright glow
- Gold/purple particles
- Epic fanfare
- Screen flash (subtle)
```

---

## 11. Performance Optimization

### 11.1 Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Active Chests | 400+ | Full map spawn |
| Render Cost | < 1ms | All visible chests |
| Memory | < 20 MB | Chest assets loaded |
| Network | < 5 KB/s | Chest sync traffic |

### 11.2 Optimization Strategies

```javascript
class ChestOptimizer {
    // Only render nearby chests in detail
    updateChestLOD(chests, cameraPosition) {
        for (const chest of chests) {
            const distance = Vector3.Distance(chest.position, cameraPosition);

            if (distance < 20) {
                chest.setLOD('HIGH');    // Full detail
            } else if (distance < 50) {
                chest.setLOD('MEDIUM');  // Reduced particles
            } else if (distance < 100) {
                chest.setLOD('LOW');     // Simple sprite
            } else {
                chest.setLOD('CULLED');  // Not rendered
            }
        }
    }

    // Pool chest objects for reuse
    chestPool = new ObjectPool(TreasureChest, 50);

    // Batch chest rendering
    batchRenderChests(visibleChests) {
        // Group by rarity for instanced rendering
        const byRarity = groupBy(visibleChests, c => c.rarity);
        for (const [rarity, chests] of byRarity) {
            this.renderInstancedChests(rarity, chests);
        }
    }
}
```

### 11.3 Mobile-Specific Optimizations

| Optimization | Implementation |
|--------------|----------------|
| Reduced particles | 50% particle count on low-end |
| Simpler shaders | No real-time shadows on chests |
| LOD distance | Reduced on low-end devices |
| Glow quality | Simpler glow effect options |

---

## 12. Balance Considerations

### 12.1 Loot Distribution Goals

| Goal | Target |
|------|--------|
| Early game ability | 90% of players find ability in 30s |
| Full loadout time | Average 2 minutes to fill slots |
| Rank 3 acquisition | ~20% of players get Rank 3 by endgame |
| Hot drop risk/reward | High-density areas are contested |

### 12.2 Balance Knobs

```javascript
// Tunable parameters for balancing
const BALANCE_CONFIG = {
    // Chest counts
    totalChests: 400,
    poiChestDensity: 1.5,     // Multiplier for POI areas

    // Rarity distribution
    commonRate: 0.60,
    uncommonRate: 0.25,
    rareRate: 0.12,
    epicRate: 0.03,

    // Timing
    openDuration: 0.75,        // Seconds
    swapPromptTimeout: 5,      // Seconds

    // NPC drops
    npcDropMultiplier: 1.0,    // Scale all NPC drops

    // Ground items
    groundItemDespawnTime: 30  // Seconds
};
```

---

## 13. Testing Requirements

### 13.1 Unit Tests

| Test Case | Description |
|-----------|-------------|
| Loot table math | Verify probability distributions |
| Spawn point validity | No overlapping chests |
| State transitions | Valid state machine flow |
| Pickup logic | Auto-equip, upgrade, swap |
| Race conditions | Concurrent open attempts |

### 13.2 Integration Tests

| Test Case | Description |
|-----------|-------------|
| Full loot flow | Spawn → Open → Pickup → Use |
| 60-player loot | Performance with full match |
| Network sync | All clients see same state |
| NPC drops | Kill → Drop → Pickup |
| Edge cases | Disconnect during open, etc. |

### 13.3 Playtest Metrics

| Metric | Target |
|--------|--------|
| Avg abilities found | 4-6 per match |
| Time to first ability | < 30 seconds |
| Swap decisions | ~2 per match average |
| Loot-related deaths | < 5% (vulnerable while looting) |

---

## 14. Appendices

### Appendix A: Spawn Point Density by POI

| POI Name | Chest Count | Rarity Pool |
|----------|-------------|-------------|
| Port Plunder | 20 | BALANCED_HIGH |
| Skeleton Bay | 15 | BALANCED |
| Cursed Temple | 12 | RARE_BIASED |
| Shipwreck Cove | 12 | BALANCED |
| Volcano Peak | 10 | EPIC_BIASED |
| Minor Outposts (x8) | 5 each | BALANCED |
| Wilderness | 1-2 per area | COMMON_BIASED |

### Appendix B: Ability Quick Reference

| Ability | Type | Description |
|---------|------|-------------|
| Fireball | Offensive | Ranged AoE damage |
| Cutlass Slash | Offensive | Melee arc damage |
| Cannonball | Offensive | Long range heavy hit |
| Lightning Strike | Offensive | Delayed AoE |
| Poison Mackerel | Offensive | DoT projectile |
| Grappling Hook | Utility | Dash to location |
| Barrel Roll | Utility | Dodge + invuln |
| Healing Grog | Utility | Self heal |
| Smoke Bomb | Utility | Vision block + slow |
| Wind Burst | Utility | Knockback AoE |

### Appendix C: Performance Profiling

| Operation | Budget |
|-----------|--------|
| Chest spawn (all) | < 50ms |
| Chest render (visible) | < 1ms |
| Open interaction | < 5ms |
| Pickup processing | < 2ms |
| Network sync | < 1ms per update |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
