# Player Elimination System Specification

## Document Information
- **Task ID:** GAME-005
- **Priority:** P0
- **Complexity:** S
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the player elimination system for Plunderstorm Mobile. The system handles player death, elimination announcements, loot drops, kill attribution, and match end conditions.

### 1.2 Scope
- Player death and elimination flow
- Kill attribution and assists
- Death drops (soul orb/treasure)
- Kill feed and notifications
- Player count tracking
- Spectator transition
- Match end conditions
- Victory/defeat screens
- Statistics tracking

### 1.3 Dependencies
- GAME-012: Damage and Health System
- BACK-005: Stats Tracking Service
- UX-012: Spectator UI (for eliminated players)

---

## 2. Elimination Flow

### 2.1 State Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    ELIMINATION STATE FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐                                                    │
│  │  ALIVE   │  Normal gameplay state                             │
│  └────┬─────┘                                                    │
│       │                                                          │
│       │ [Health reaches 0]                                       │
│       ▼                                                          │
│  ┌──────────┐                                                    │
│  │  DYING   │  Brief death animation (0.5s)                      │
│  │          │  Player loses control                              │
│  └────┬─────┘                                                    │
│       │                                                          │
│       │ [Animation complete]                                     │
│       ▼                                                          │
│  ┌──────────┐                                                    │
│  │ELIMINATED│  Death finalized                                   │
│  │          │  Drops spawn                                       │
│  │          │  Stats recorded                                    │
│  └────┬─────┘                                                    │
│       │                                                          │
│       ▼                                                          │
│  ┌───────────────────────────────────────────────────┐          │
│  │            POST-ELIMINATION OPTIONS                │          │
│  │                                                    │          │
│  │   ┌─────────────┐        ┌─────────────┐          │          │
│  │   │  SPECTATE   │        │   LOBBY     │          │          │
│  │   │  Watch game │        │   Return    │          │          │
│  │   └─────────────┘        └─────────────┘          │          │
│  └───────────────────────────────────────────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Elimination Trigger

```javascript
class EliminationSystem {
    onPlayerHealthZero(player, damageSource) {
        // Validate on server
        if (!this.isServer) return;

        // Prevent double elimination
        if (player.state === PlayerState.ELIMINATED) return;

        // Transition to dying state
        player.state = PlayerState.DYING;
        player.deathTime = Time.serverNow();

        // Determine killer
        const killer = this.determineKiller(player, damageSource);

        // Queue elimination processing
        this.scheduleElimination(player, killer, DEATH_ANIMATION_DURATION);
    }

    scheduleElimination(player, killer, delay) {
        setTimeout(() => {
            this.processElimination(player, killer);
        }, delay * 1000);
    }
}
```

---

## 3. Kill Attribution

### 3.1 Kill Credit Rules

| Scenario | Killer Credit | Notes |
|----------|---------------|-------|
| Direct damage | Last damager | Standard kill |
| Storm death | No credit | "Consumed by storm" |
| Fall damage | Last damager (10s) | If damaged recently |
| Disconnect | Last damager (30s) | If damaged recently |
| Self-damage | No credit | "Eliminated themselves" |

### 3.2 Assist System

```javascript
class AssistTracker {
    static ASSIST_WINDOW = 10;      // Seconds
    static ASSIST_THRESHOLD = 30;   // Minimum damage for assist

    trackDamage(attacker, victim, damage) {
        // Record damage contribution
        const record = {
            attackerId: attacker.id,
            damage: damage,
            timestamp: Time.serverNow()
        };

        victim.damageHistory.push(record);

        // Prune old records
        this.pruneOldRecords(victim);
    }

    getAssists(victim, killer) {
        const assists = [];
        const now = Time.serverNow();

        for (const record of victim.damageHistory) {
            // Skip killer (they get the kill)
            if (record.attackerId === killer?.id) continue;

            // Check time window
            if (now - record.timestamp > ASSIST_WINDOW) continue;

            // Accumulate damage from this attacker
            const totalDamage = this.sumDamageFrom(
                victim.damageHistory,
                record.attackerId,
                ASSIST_WINDOW
            );

            if (totalDamage >= ASSIST_THRESHOLD) {
                assists.push({
                    playerId: record.attackerId,
                    damage: totalDamage
                });
            }
        }

        return assists;
    }
}
```

### 3.3 Kill Attribution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    KILL ATTRIBUTION FLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Player Takes Fatal Damage                                       │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────────────┐                                           │
│  │ Check Last       │                                           │
│  │ Damage Source    │                                           │
│  └────────┬─────────┘                                           │
│           │                                                      │
│     ┌─────┴──────┬────────────┬────────────┐                    │
│     ▼            ▼            ▼            ▼                    │
│  [Player]    [Storm]     [Fall]      [Self]                     │
│     │            │            │            │                    │
│     ▼            ▼            ▼            ▼                    │
│  Killer =    Killer =    Check        Killer =                  │
│  Attacker    null        Recent       null                      │
│                          Damage                                  │
│     │            │            │            │                    │
│     ▼            ▼            ▼            ▼                    │
│  Award        "Storm      If < 10s:   "Eliminated               │
│  Kill         consumed"   Award Kill  themselves"               │
│                           Else: null                            │
│     │            │            │            │                    │
│     └────────────┴────────────┴────────────┘                    │
│                          │                                       │
│                          ▼                                       │
│                  Calculate Assists                               │
│                          │                                       │
│                          ▼                                       │
│                  Update Statistics                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Death Drops

### 4.1 Soul Orb System

```javascript
class DeathDrops {
    static XP_BASE_REWARD = 50;
    static XP_PER_VICTIM_LEVEL = 20;
    static ORB_LIFETIME = 60;        // Seconds before despawn
    static ORB_PICKUP_RADIUS = 2;    // Meters

    createDeathDrop(eliminatedPlayer, position) {
        const drop = {
            id: generateUUID(),
            position: position,
            type: 'SOUL_ORB',

            // XP reward scales with victim's level
            xpReward: XP_BASE_REWARD + (eliminatedPlayer.level * XP_PER_VICTIM_LEVEL),

            // Spawn time for despawn calculation
            spawnTime: Time.serverNow(),

            // Visual properties
            glowIntensity: eliminatedPlayer.level / 10,
            color: this.getOrbColor(eliminatedPlayer.level)
        };

        // Spawn with slight arc physics
        drop.initialVelocity = new Vector3(
            randomRange(-1, 1),
            3,
            randomRange(-1, 1)
        );

        return drop;
    }

    getOrbColor(level) {
        if (level >= 8) return 'GOLD';
        if (level >= 5) return 'BLUE';
        return 'GREEN';
    }
}
```

### 4.2 Drop Contents

| Drop Type | Contents | Notes |
|-----------|----------|-------|
| Soul Orb | XP only | Primary drop |
| Abilities | Victim's loadout | Dropped on ground nearby |

### 4.3 Ability Drops on Death

```javascript
class AbilityDropOnDeath {
    handleDeathDrops(player, position) {
        const drops = [];

        // Drop all equipped abilities
        for (const slot of player.abilitySlots) {
            if (slot.ability) {
                const dropPosition = this.calculateDropPosition(
                    position,
                    drops.length
                );

                drops.push({
                    ability: slot.ability,
                    position: dropPosition,
                    despawnTime: Time.serverNow() + 30 // 30 second despawn
                });
            }
        }

        return drops;
    }

    calculateDropPosition(center, index) {
        // Spread abilities in a circle around death location
        const angle = (index / 4) * Math.PI * 2;
        const radius = 1.5;

        return new Vector3(
            center.x + Math.cos(angle) * radius,
            center.y,
            center.z + Math.sin(angle) * radius
        );
    }
}
```

---

## 5. Kill Feed

### 5.1 Kill Feed Messages

```
┌─────────────────────────────────────────────────────────────────┐
│                    KILL FEED DISPLAY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Top-right corner of screen:                                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │  🗡️ RedBeard eliminated BlackSail                      │     │
│  │     └── (Fireball - 45 damage)                          │     │
│  │                                                         │     │
│  │  💀 StormChaser was consumed by the storm               │     │
│  │                                                         │     │
│  │  🏆 YOU eliminated PirateKing                           │     │
│  │     └── +150 XP  |  8 players remaining                 │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Feed settings:                                                  │
│  - Max visible entries: 5                                        │
│  - Entry duration: 5 seconds                                     │
│  - Fade out duration: 0.5 seconds                                │
│  - Own kills highlighted (gold border)                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Kill Feed Data Structure

```javascript
struct KillFeedEntry {
    uint32 killerPlayerId;      // 0 if environmental
    uint32 victimPlayerId;
    string killerName;
    string victimName;
    uint8  eliminationType;      // PLAYER, STORM, FALL, SELF
    uint8  weaponId;            // Ability used (if player kill)
    uint16 damage;              // Final damage dealt
    uint8  remainingPlayers;    // Players left after this kill
    float  timestamp;
}
```

### 5.3 Elimination Types

| Type | Icon | Message Format |
|------|------|----------------|
| PLAYER | ⚔️ | "{killer} eliminated {victim}" |
| STORM | 🌩️ | "{victim} was consumed by the storm" |
| FALL | 💀 | "{victim} fell to their death" |
| SELF | 💀 | "{victim} eliminated themselves" |

---

## 6. Player Notifications

### 6.1 Kill Notification (Killer's Screen)

```
┌─────────────────────────────────────────────────────────────────┐
│                    KILL NOTIFICATION                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Center of screen, brief display:                                │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              ⚔️ ELIMINATED                              │     │
│  │              BlackSail                                  │     │
│  │                                                         │     │
│  │              +1 KILL  |  +150 XP                        │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Display duration: 2 seconds                                     │
│  Animation: Slide in, scale up, fade out                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Death Notification (Victim's Screen)

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEATH NOTIFICATION                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Full screen overlay:                                            │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                    💀 ELIMINATED 💀                     │     │
│  │                                                         │     │
│  │              Eliminated by: RedBeard                    │     │
│  │              Weapon: Fireball                           │     │
│  │                                                         │     │
│  │              ───────────────────────                    │     │
│  │                                                         │     │
│  │              Your Match Stats:                          │     │
│  │              Placement: #15                             │     │
│  │              Kills: 3                                   │     │
│  │              Damage Dealt: 450                          │     │
│  │              Survival Time: 8:32                        │     │
│  │                                                         │     │
│  │              [SPECTATE]     [RETURN TO LOBBY]           │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Player Count Tracking

### 7.1 Remaining Players Display

```javascript
class PlayerCountTracker {
    constructor(totalPlayers) {
        this.totalPlayers = totalPlayers;
        this.eliminatedPlayers = 0;
        this.remainingPlayers = totalPlayers;
    }

    onPlayerEliminated() {
        this.eliminatedPlayers++;
        this.remainingPlayers--;

        // Broadcast update
        this.broadcastPlayerCount();

        // Check for win condition
        this.checkWinCondition();
    }

    broadcastPlayerCount() {
        // Send to all clients
        broadcast({
            type: 'PLAYER_COUNT_UPDATE',
            remaining: this.remainingPlayers,
            eliminated: this.eliminatedPlayers
        });
    }
}
```

### 7.2 Player Count UI

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLAYER COUNT UI                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Top-left corner:                                                │
│                                                                  │
│  ┌──────────────────┐                                           │
│  │  👥 23 Alive     │  Updates in real-time                     │
│  └──────────────────┘                                           │
│                                                                  │
│  Milestones (brief animation):                                   │
│  - 30 remaining: "Half the pirates remain!"                      │
│  - 10 remaining: "Final 10!"                                     │
│  - 5 remaining: "Final 5!"                                       │
│  - 2 remaining: "1v1 - Fight for the crown!"                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Match End Conditions

### 8.1 Victory Conditions

| Mode | Win Condition |
|------|---------------|
| Solo | Last player standing |
| Duo | Last team with alive member(s) |
| Squad | Last team with alive member(s) |

### 8.2 Win Detection

```javascript
class WinConditionChecker {
    checkWinCondition(remainingPlayers, mode) {
        switch (mode) {
            case 'SOLO':
                if (remainingPlayers.length === 1) {
                    return {
                        winner: remainingPlayers[0],
                        type: 'SOLO_VICTORY'
                    };
                }
                break;

            case 'DUO':
            case 'SQUAD':
                const remainingTeams = this.getUniqueTeams(remainingPlayers);
                if (remainingTeams.length === 1) {
                    return {
                        winner: remainingTeams[0],
                        type: 'TEAM_VICTORY'
                    };
                }
                break;
        }

        return null; // No winner yet
    }

    handleDraw() {
        // Edge case: All remaining players die simultaneously
        // Award victory to last player eliminated (by timestamp)
        return this.getLastEliminatedPlayer();
    }
}
```

### 8.3 Match End Broadcast

```javascript
struct MatchEndPacket {
    uint8  endType;           // VICTORY, DEFEAT, DRAW
    uint32 winnerPlayerId;    // Or team ID
    string winnerName;
    uint16 winnerKills;
    uint8  totalPlayers;
    float  matchDuration;
}
```

---

## 9. Victory Screen

### 9.1 Winner Display

```
┌─────────────────────────────────────────────────────────────────┐
│                    VICTORY SCREEN                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                    🏆 VICTORY! 🏆                       │     │
│  │                                                         │     │
│  │              [Winner Character Pose]                    │     │
│  │                                                         │     │
│  │              RedBeard                                   │     │
│  │              Champion of the Seas!                      │     │
│  │                                                         │     │
│  │              ───────────────────────                    │     │
│  │                                                         │     │
│  │              Match Statistics                           │     │
│  │              ─────────────────                          │     │
│  │              Kills: 8                                   │     │
│  │              Damage Dealt: 1,247                        │     │
│  │              Abilities Used: 34                         │     │
│  │              Distance Traveled: 2.3 km                  │     │
│  │              Survival Time: 12:45                       │     │
│  │                                                         │     │
│  │              XP Earned: 450                             │     │
│  │              Battle Pass Progress: +150                 │     │
│  │                                                         │     │
│  │              [PLAY AGAIN]     [RETURN TO LOBBY]         │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Winner display also broadcast to all spectators                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Defeat Screen (for eliminated players)

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEFEAT SCREEN                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Shown when match ends while spectating:                         │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                    MATCH COMPLETE                       │     │
│  │                                                         │     │
│  │              Winner: RedBeard                           │     │
│  │              8 Kills                                    │     │
│  │                                                         │     │
│  │              ───────────────────────                    │     │
│  │                                                         │     │
│  │              Your Performance                           │     │
│  │              ─────────────────                          │     │
│  │              Placement: #15 / 60                        │     │
│  │              Kills: 3                                   │     │
│  │              Damage Dealt: 450                          │     │
│  │                                                         │     │
│  │              XP Earned: 180                             │     │
│  │              Battle Pass Progress: +60                  │     │
│  │                                                         │     │
│  │              [PLAY AGAIN]     [RETURN TO LOBBY]         │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Statistics Tracking

### 10.1 Match Statistics

```javascript
class MatchStatistics {
    // Per-player statistics tracked during match
    static TRACKED_STATS = {
        // Combat
        kills: 0,
        assists: 0,
        deaths: 0,              // Always 0 or 1 in BR
        damageDealt: 0,
        damageTaken: 0,
        headshotKills: 0,

        // Abilities
        abilitiesUsed: 0,
        abilityDamage: 0,
        abilitiesCollected: 0,
        abilitiesUpgraded: 0,

        // Movement
        distanceTraveled: 0,
        glidesUsed: 0,          // Grappling hook, etc.

        // Survival
        survivalTime: 0,        // Seconds
        stormDamageTaken: 0,
        chestsOpened: 0,
        npcsKilled: 0,

        // Placement
        placement: 0,           // 1-60
        playersOutlived: 0
    };
}
```

### 10.2 Statistics Update Flow

```javascript
class StatisticsManager {
    onPlayerEliminated(player, killer, matchContext) {
        // Calculate final stats
        player.stats.survivalTime = Time.serverNow() - matchContext.startTime;
        player.stats.placement = matchContext.remainingPlayers + 1;
        player.stats.playersOutlived = matchContext.totalPlayers - player.stats.placement;

        // Update killer stats
        if (killer) {
            killer.stats.kills++;

            // Check for special conditions
            if (this.wasHeadshot(player.lastDamageInfo)) {
                killer.stats.headshotKills++;
            }
        }

        // Update assists
        for (const assist of player.assists) {
            const assister = this.getPlayer(assist.playerId);
            if (assister) {
                assister.stats.assists++;
            }
        }

        // Send stats to backend
        this.reportStatistics(player.stats);
    }
}
```

### 10.3 XP Calculation

```javascript
class XPCalculator {
    static XP_REWARDS = {
        // Base rewards
        participation: 25,
        kill: 50,
        assist: 20,

        // Placement bonuses
        placement: {
            1: 300,     // Victory
            2: 200,
            3: 150,
            4: 100,
            5: 75,
            10: 50,
            20: 25
        },

        // Activity bonuses
        damagePerHundred: 10,
        chestOpened: 5,
        npcKilled: 10
    };

    calculateMatchXP(stats) {
        let xp = XP_REWARDS.participation;

        // Kills and assists
        xp += stats.kills * XP_REWARDS.kill;
        xp += stats.assists * XP_REWARDS.assist;

        // Placement bonus
        xp += this.getPlacementBonus(stats.placement);

        // Activity bonuses
        xp += Math.floor(stats.damageDealt / 100) * XP_REWARDS.damagePerHundred;
        xp += stats.chestsOpened * XP_REWARDS.chestOpened;
        xp += stats.npcsKilled * XP_REWARDS.npcKilled;

        return xp;
    }
}
```

---

## 11. Death Animation and VFX

### 11.1 Death Animation Sequence

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEATH ANIMATION TIMELINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  0.0s ─────────────────────────────────────────────────► 0.5s   │
│                                                                  │
│  ├── 0.00s: Health reaches 0                                    │
│  │   - Player loses control                                     │
│  │   - Hit reaction animation triggers                          │
│  │                                                               │
│  ├── 0.05s: Death cry audio                                     │
│  │   - "Arrgh!" voiceline plays                                 │
│  │                                                               │
│  ├── 0.10s: Ragdoll or death pose begins                        │
│  │   - Physics takes over                                       │
│  │   - Character falls/slumps                                   │
│  │                                                               │
│  ├── 0.30s: Soul departure VFX                                  │
│  │   - Ghostly effect rises from body                           │
│  │   - Glowing orb appears                                      │
│  │                                                               │
│  ├── 0.50s: Body fades                                          │
│  │   - Character model becomes transparent                      │
│  │   - Soul orb solidifies                                      │
│  │   - Ability drops spawn                                      │
│  │                                                               │
│  └── 1.00s: Complete                                            │
│       - Body fully removed                                       │
│       - Drops pickupable                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 11.2 Death VFX

| Effect | Description | Duration |
|--------|-------------|----------|
| Impact Flash | Screen flash (killer's view) | 0.1s |
| Blood/Sparkle | Contact point particles | 0.3s |
| Soul Rise | Ghost effect from body | 0.5s |
| Orb Coalesce | Particles form soul orb | 0.3s |
| Body Fade | Character transparency | 0.5s |
| Drop Spawn | Abilities pop out | 0.2s |

### 11.3 Audio Cues

| Sound | Trigger | Description |
|-------|---------|-------------|
| Death cry | Health = 0 | Character voiceline |
| Elimination confirm | Kill registered | Satisfying "ding" |
| Soul release | Soul VFX starts | Ethereal whoosh |
| Drop spawn | Abilities appear | Item spawn sound |

---

## 12. Server Authority

### 12.1 Server-Side Validation

```javascript
class EliminationValidator {
    validateElimination(player, reportedKiller, damageInfo) {
        // Verify player was actually alive
        if (player.state !== PlayerState.ALIVE) {
            return { valid: false, reason: 'ALREADY_ELIMINATED' };
        }

        // Verify health actually reached zero
        if (player.health > 0) {
            return { valid: false, reason: 'HEALTH_NOT_ZERO' };
        }

        // Verify damage source is legitimate
        if (reportedKiller && !this.validateDamageSource(reportedKiller, damageInfo)) {
            return { valid: false, reason: 'INVALID_DAMAGE_SOURCE' };
        }

        return { valid: true };
    }

    validateDamageSource(killer, damageInfo) {
        // Check killer was in valid state
        if (killer.state !== PlayerState.ALIVE) return false;

        // Check ability was off cooldown
        if (damageInfo.abilityId) {
            const ability = killer.getAbility(damageInfo.abilityId);
            if (!ability || ability.isOnCooldown()) return false;
        }

        // Check range validity
        const distance = Vector3.Distance(killer.position, damageInfo.hitPosition);
        const maxRange = this.getAbilityMaxRange(damageInfo.abilityId);
        if (distance > maxRange * 1.2) return false; // 20% tolerance for lag

        return true;
    }
}
```

### 12.2 Anti-Exploit Measures

| Exploit | Prevention |
|---------|------------|
| Fake deaths | Server tracks actual health |
| Zombie state | State machine prevents invalid transitions |
| Death dodging | Server-authoritative elimination |
| XP farming | Diminishing returns on same-player kills |

---

## 13. Edge Cases

### 13.1 Simultaneous Deaths

```javascript
class SimultaneousDeathHandler {
    handleSimultaneousDeaths(player1, player2, timestamp) {
        // If players kill each other at same server tick
        // Award kills to both, eliminate both

        // Determine order by damage timestamp (microseconds)
        const order = this.determineMicrosecondOrder(player1, player2);

        // Process in order (for placement purposes)
        this.processElimination(order.first, order.second);
        this.processElimination(order.second, order.first);

        // Both get credit for the kill
        order.first.stats.kills++;
        order.second.stats.kills++;
    }
}
```

### 13.2 Disconnect During Death

```javascript
class DisconnectHandler {
    onPlayerDisconnect(player) {
        if (player.state === PlayerState.DYING) {
            // Complete the elimination
            this.processElimination(player, player.lastAttacker);
        } else if (player.state === PlayerState.ALIVE) {
            // Give kill credit if damaged recently
            if (player.lastDamageTime > Time.serverNow() - 30) {
                this.processElimination(player, player.lastAttacker);
            } else {
                this.processElimination(player, null);
            }
        }
    }
}
```

### 13.3 Last Two Players Trade

```javascript
class FinalTradeHandler {
    handleFinalTrade() {
        // Both final players die simultaneously
        // Award victory to player who dealt damage first

        const player1DamageTime = this.getFirstDamageDealtTime(player1, player2);
        const player2DamageTime = this.getFirstDamageDealtTime(player2, player1);

        if (player1DamageTime < player2DamageTime) {
            return { winner: player1, reason: 'FIRST_DAMAGE' };
        } else if (player2DamageTime < player1DamageTime) {
            return { winner: player2, reason: 'FIRST_DAMAGE' };
        } else {
            // True tie - random selection
            return { winner: Math.random() > 0.5 ? player1 : player2, reason: 'TIE_BREAKER' };
        }
    }
}
```

---

## 14. Testing Requirements

### 14.1 Unit Tests

| Test Case | Description |
|-----------|-------------|
| Kill attribution | Correct killer credited |
| Assist calculation | Assists awarded properly |
| Storm kills | No killer, correct message |
| XP calculation | Correct XP for all scenarios |
| State transitions | Valid state machine flow |

### 14.2 Integration Tests

| Test Case | Description |
|-----------|-------------|
| Full elimination flow | Death → Drops → Stats → UI |
| 60-player rapid kills | Performance under load |
| Network sync | All clients see same kills |
| Match end | Victory detected correctly |
| Disconnect handling | Graceful elimination on DC |

### 14.3 Playtest Validation

| Criteria | Target |
|----------|--------|
| Kill feed clarity | > 95% understand who killed who |
| Death feedback | Death feels impactful but fair |
| Stats accuracy | All stats match player perception |
| Victory celebration | Feels rewarding |

---

## 15. Appendices

### Appendix A: Elimination Message Templates

| Language | Template |
|----------|----------|
| EN | "{killer} eliminated {victim}" |
| EN (Storm) | "{victim} was consumed by the storm" |
| EN (Self) | "{victim} eliminated themselves" |
| EN (Fall) | "{victim} fell to their death" |

### Appendix B: XP Reward Reference

| Action | XP Reward |
|--------|-----------|
| Participation | 25 |
| Kill | 50 |
| Assist | 20 |
| Victory (1st) | 300 |
| Top 3 | 150 |
| Top 10 | 50 |
| Per 100 damage | 10 |
| Chest opened | 5 |
| NPC killed | 10 |

### Appendix C: Performance Budgets

| Operation | Budget |
|-----------|--------|
| Elimination processing | < 5ms |
| Kill feed update | < 1ms |
| Death VFX | < 2ms GPU |
| Stats calculation | < 2ms |
| Network broadcast | < 1ms |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
