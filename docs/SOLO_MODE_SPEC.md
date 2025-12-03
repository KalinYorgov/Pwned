# Solo Mode Specification

## Document Information
- **Task ID:** GAME-020
- **Priority:** P0
- **Complexity:** M
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the Solo Mode for Plunderstorm Mobile - the core free-for-all battle royale experience where 30-60 players compete until one remains standing.

### 1.2 Scope
- Match configuration and player count
- Full match flow from queue to victory
- Integration of all core systems
- Placement tracking and rewards
- Bot fill for low population
- Mode-specific rules and balancing

### 1.3 Dependencies
- All P0 GAME tasks (GAME-001 through GAME-013)
- BACK-001: Core Matchmaking Service

---

## 2. Mode Configuration

### 2.1 Basic Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Mode Name | Solo Battle Royale | Display name |
| Player Count | 30-60 | Scalable based on population |
| Team Size | 1 (no teams) | Free-for-all |
| Win Condition | Last player standing | Single winner |
| Match Duration | 10-15 minutes | Target average |
| Queue Time Target | < 30 seconds | With bot fill |

### 2.2 Match Settings

```javascript
const SOLO_MODE_CONFIG = {
    // Player settings
    minPlayers: 20,           // Minimum to start (with bots)
    maxPlayers: 60,           // Maximum capacity
    preferredPlayers: 40,     // Optimal match size

    // Timing
    lobbyWaitTime: 30,        // Seconds before auto-start
    countdownTime: 5,         // Final countdown
    dropPhaseTime: 45,        // Seconds for drop ship path

    // Storm phases (see GAME-003)
    stormPhases: 5,
    totalMatchTime: 900,      // 15 minutes max

    // Bot fill
    botFillEnabled: true,
    botFillDelay: 15,         // Seconds before filling
    minHumanPlayers: 10,      // Minimum humans (rest can be bots)

    // Rewards
    xpEnabled: true,
    rankingEnabled: false     // Casual mode (ranked separate)
};
```

---

## 3. Full Match Flow

### 3.1 Match Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOLO MODE MATCH FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                               │
│  │ 1. QUEUE     │  Player joins solo queue                      │
│  │              │  Matchmaker finds/creates match               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 2. LOBBY     │  Players gather (30s wait)                    │
│  │              │  Bot fill if needed                           │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 3. COUNTDOWN │  5 second countdown                           │
│  │              │  "Get ready!"                                  │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 4. DROP      │  Ship path + player drops                     │
│  │              │  45 second drop window                        │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 5. GAMEPLAY  │  Main battle royale loop                      │
│  │              │  Loot, fight, survive                         │
│  │              │  Storm phases active                          │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 6. END GAME  │  Final players fight                          │
│  │              │  Last standing wins                           │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐                                               │
│  │ 7. RESULTS   │  Victory/defeat screen                        │
│  │              │  Stats, XP, rewards                           │
│  └──────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Phase Details

#### Phase 1: Queue
```javascript
class SoloQueuePhase {
    onPlayerJoinQueue(player) {
        // Add to matchmaking pool
        Matchmaker.addToPool(player, 'SOLO');

        // Show queue UI
        player.showUI('queue_searching', {
            mode: 'Solo Battle Royale',
            estimatedTime: Matchmaker.getEstimatedWait('SOLO')
        });
    }

    onMatchFound(player, match) {
        player.transitionTo(match.lobbyId);
    }
}
```

#### Phase 2: Lobby
```javascript
class SoloLobbyPhase {
    static WAIT_TIME = 30;
    static COUNTDOWN_THRESHOLD = 30; // Players needed to skip wait

    update() {
        const playerCount = this.getPlayerCount();

        // Show player count
        this.broadcastPlayerCount(playerCount);

        // Check start conditions
        if (playerCount >= COUNTDOWN_THRESHOLD) {
            this.startCountdown();
        } else if (this.timeWaiting >= WAIT_TIME) {
            this.fillWithBots();
            this.startCountdown();
        }
    }

    fillWithBots() {
        const currentPlayers = this.getPlayerCount();
        const botsNeeded = SOLO_MODE_CONFIG.minPlayers - currentPlayers;

        for (let i = 0; i < botsNeeded; i++) {
            this.addBot(this.selectBotDifficulty());
        }
    }
}
```

#### Phase 3-4: Countdown and Drop
- Uses GAME-001 (Match Initialization) spec
- Uses GAME-002 (Player Drop-In) spec

#### Phase 5: Gameplay
- All core systems active
- Storm phases from GAME-003
- Loot from GAME-004
- Combat from GAME-008, GAME-011, GAME-012

#### Phase 6: End Game
```javascript
class SoloEndGamePhase {
    checkVictoryCondition() {
        const alivePlayers = this.getAlivePlayers();

        if (alivePlayers.length === 1) {
            return {
                winner: alivePlayers[0],
                type: 'SOLO_VICTORY'
            };
        }

        if (alivePlayers.length === 0) {
            // All players died (simultaneous elimination)
            return {
                winner: this.getLastEliminatedPlayer(),
                type: 'LAST_ALIVE_VICTORY'
            };
        }

        return null; // Match continues
    }

    onVictory(winner) {
        // Broadcast to all players
        this.broadcastVictory(winner);

        // Transition to results
        this.transitionToResults();
    }
}
```

---

## 4. Placement System

### 4.1 Placement Tracking

```javascript
class PlacementTracker {
    constructor(totalPlayers) {
        this.totalPlayers = totalPlayers;
        this.eliminationOrder = [];
        this.currentPlacement = totalPlayers;
    }

    onPlayerEliminated(player) {
        player.placement = this.currentPlacement;
        this.eliminationOrder.push(player.id);
        this.currentPlacement--;

        // Broadcast placement
        player.showPlacement(player.placement, this.totalPlayers);
    }

    onVictory(winner) {
        winner.placement = 1;
        winner.showVictory();
    }

    getPlacement(playerId) {
        if (this.eliminationOrder.includes(playerId)) {
            const index = this.eliminationOrder.indexOf(playerId);
            return this.totalPlayers - index;
        }
        return null; // Still alive
    }
}
```

### 4.2 Placement Display

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLACEMENT DISPLAY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  On Elimination:                                                 │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                   #15 / 60                              │     │
│  │                  ─────────                              │     │
│  │              You placed 15th!                           │     │
│  │                                                         │     │
│  │         Outlived 45 other pirates                       │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  On Victory:                                                     │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                 🏆 #1 VICTORY 🏆                        │     │
│  │                                                         │     │
│  │              You are the champion!                      │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Rewards System

### 5.1 XP Rewards by Placement

| Placement | Base XP | Notes |
|-----------|---------|-------|
| #1 | 300 | Victory bonus |
| #2 | 200 | Runner-up |
| #3 | 150 | Podium finish |
| #4-5 | 100 | Top 5 |
| #6-10 | 75 | Top 10 |
| #11-20 | 50 | Top 20 |
| #21-30 | 35 | Top 30 |
| #31+ | 25 | Participation |

### 5.2 Activity XP

| Action | XP | Cap |
|--------|-----|-----|
| Per Kill | 50 | None |
| Per Assist | 20 | None |
| Per 100 Damage | 10 | 200 XP |
| Per Chest Opened | 5 | 50 XP |
| Per NPC Killed | 10 | 100 XP |

### 5.3 XP Calculation

```javascript
class SoloRewardsCalculator {
    calculateMatchXP(playerStats) {
        let xp = 0;

        // Placement XP
        xp += this.getPlacementXP(playerStats.placement);

        // Kill XP
        xp += playerStats.kills * 50;

        // Assist XP
        xp += playerStats.assists * 20;

        // Damage XP (capped)
        const damageXP = Math.floor(playerStats.damageDealt / 100) * 10;
        xp += Math.min(damageXP, 200);

        // Loot XP (capped)
        const chestXP = playerStats.chestsOpened * 5;
        xp += Math.min(chestXP, 50);

        // NPC XP (capped)
        const npcXP = playerStats.npcsKilled * 10;
        xp += Math.min(npcXP, 100);

        return xp;
    }

    getPlacementXP(placement) {
        if (placement === 1) return 300;
        if (placement === 2) return 200;
        if (placement === 3) return 150;
        if (placement <= 5) return 100;
        if (placement <= 10) return 75;
        if (placement <= 20) return 50;
        if (placement <= 30) return 35;
        return 25;
    }
}
```

---

## 6. Bot Fill System

### 6.1 Bot Configuration

```javascript
const BOT_CONFIG = {
    // Difficulty tiers
    difficulties: ['EASY', 'MEDIUM', 'HARD'],

    // Distribution based on player skill
    difficultyDistribution: {
        newPlayer: { EASY: 0.7, MEDIUM: 0.3, HARD: 0.0 },
        casual: { EASY: 0.3, MEDIUM: 0.5, HARD: 0.2 },
        experienced: { EASY: 0.1, MEDIUM: 0.4, HARD: 0.5 }
    },

    // Bot behavior
    behaviors: {
        EASY: {
            aimAccuracy: 0.3,
            reactionTime: 800,      // ms
            abilityUseDelay: 2000,  // ms
            dodgeChance: 0.1
        },
        MEDIUM: {
            aimAccuracy: 0.5,
            reactionTime: 500,
            abilityUseDelay: 1000,
            dodgeChance: 0.3
        },
        HARD: {
            aimAccuracy: 0.7,
            reactionTime: 300,
            abilityUseDelay: 500,
            dodgeChance: 0.5
        }
    }
};
```

### 6.2 Bot Naming

```javascript
class BotNameGenerator {
    static PREFIXES = [
        'Captain', 'Scurvy', 'Salty', 'Blackbeard', 'RedSail',
        'Bones', 'Cutlass', 'Barnacle', 'Seasdog', 'Plunder'
    ];

    static SUFFIXES = [
        'Pete', 'Jack', 'Morgan', 'Anne', 'Bill',
        'Hook', 'Sparrow', 'Silver', 'Kidd', 'Drake'
    ];

    generateName() {
        const prefix = PREFIXES[Math.floor(Math.random() * PREFIXES.length)];
        const suffix = SUFFIXES[Math.floor(Math.random() * SUFFIXES.length)];
        return `${prefix}${suffix}`;
    }
}
```

### 6.3 Bot Behavior

```javascript
class SoloModeBot {
    update() {
        switch (this.state) {
            case 'DROPPING':
                this.handleDrop();
                break;

            case 'LOOTING':
                this.searchForLoot();
                break;

            case 'FIGHTING':
                this.engageCombat();
                break;

            case 'FLEEING':
                this.retreatFromDanger();
                break;

            case 'ZONE_ROTATING':
                this.moveToSafeZone();
                break;
        }
    }

    decideNextAction() {
        // Check storm
        if (this.isInStorm() || this.stormApproaching()) {
            this.state = 'ZONE_ROTATING';
            return;
        }

        // Check threats
        const nearbyEnemies = this.detectEnemies();
        if (nearbyEnemies.length > 0) {
            if (this.health < 30 || nearbyEnemies.length > 2) {
                this.state = 'FLEEING';
            } else {
                this.state = 'FIGHTING';
            }
            return;
        }

        // Check loot needs
        if (this.hasEmptyAbilitySlots()) {
            this.state = 'LOOTING';
            return;
        }

        // Default: look for fights or zone rotate
        this.state = 'ZONE_ROTATING';
    }
}
```

---

## 7. Mode Selection UI

### 7.1 Queue Entry UI

```
┌─────────────────────────────────────────────────────────────────┐
│                    MODE SELECTION SCREEN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              PLUNDERSTORM MOBILE                        │     │
│  │                                                         │     │
│  │  ┌─────────────────────────────────────────────────┐   │     │
│  │  │                                                  │   │     │
│  │  │   ⚔️ SOLO BATTLE ROYALE                         │   │     │
│  │  │                                                  │   │     │
│  │  │   Free-for-all • 60 Players • ~12 min           │   │     │
│  │  │                                                  │   │     │
│  │  │   Every pirate for themselves!                   │   │     │
│  │  │   Be the last one standing.                      │   │     │
│  │  │                                                  │   │     │
│  │  │              [PLAY SOLO]                         │   │     │
│  │  │                                                  │   │     │
│  │  └─────────────────────────────────────────────────┘   │     │
│  │                                                         │     │
│  │  ┌─────────────────────┐  ┌─────────────────────┐      │     │
│  │  │ 👥 DUOS (P1)       │  │ 🏆 RANKED (P2)      │      │     │
│  │  │ Coming Soon!       │  │ Coming Soon!        │      │     │
│  │  └─────────────────────┘  └─────────────────────┘      │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Queue Status UI

```
┌─────────────────────────────────────────────────────────────────┐
│                    QUEUE STATUS                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              🔍 SEARCHING FOR MATCH...                  │     │
│  │                                                         │     │
│  │              Mode: Solo Battle Royale                   │     │
│  │              Players: 47/60                             │     │
│  │              Wait Time: 0:12                            │     │
│  │                                                         │     │
│  │              [CANCEL]                                   │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Match Found Animation:                                          │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              ⚓ MATCH FOUND! ⚓                         │     │
│  │                                                         │     │
│  │              Preparing to set sail...                   │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. End-of-Match Screen

### 8.1 Results Display

```
┌─────────────────────────────────────────────────────────────────┐
│                    MATCH RESULTS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Victory Screen:                                                 │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              🏆 VICTORY ROYALE! 🏆                      │     │
│  │                                                         │     │
│  │         [Character celebrating animation]               │     │
│  │                                                         │     │
│  │  ─────────────────────────────────────────────────     │     │
│  │                                                         │     │
│  │  YOUR STATS                                             │     │
│  │  Placement:    #1 / 60                                  │     │
│  │  Kills:        8                                        │     │
│  │  Damage:       1,247                                    │     │
│  │  Survival:     12:34                                    │     │
│  │                                                         │     │
│  │  XP EARNED                                              │     │
│  │  Placement:    +300 ⭐                                  │     │
│  │  Kills:        +400 ⭐                                  │     │
│  │  Activity:     +180 ⭐                                  │     │
│  │  ─────────────────                                      │     │
│  │  TOTAL:        +880 ⭐                                  │     │
│  │                                                         │     │
│  │  Level Progress: ████████████░░░░  Lv.15 → Lv.16       │     │
│  │                                                         │     │
│  │     [PLAY AGAIN]         [RETURN TO LOBBY]             │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Statistics Summary

| Stat | Description |
|------|-------------|
| Placement | Final position (#1 - #60) |
| Kills | Players eliminated |
| Assists | Kill participation |
| Damage Dealt | Total damage output |
| Damage Taken | Total damage received |
| Healing | Health restored |
| Survival Time | Time alive in match |
| Distance Traveled | Movement distance |
| Abilities Used | Ability activations |
| Chests Opened | Loot containers opened |

---

## 9. Mode-Specific Rules

### 9.1 Solo-Only Rules

| Rule | Description |
|------|-------------|
| No Teaming | Players cannot ally (bannable offense) |
| No Revival | Once eliminated, match over |
| No Shared Loot | All pickups are individual |
| No Communication | No in-match chat/voice |

### 9.2 Anti-Teaming Detection

```javascript
class AntiTeamingDetector {
    detectTeaming(player1, player2) {
        const indicators = [];

        // Prolonged proximity without fighting
        const proximityTime = this.getProximityTime(player1, player2);
        if (proximityTime > 30 && !this.hasDamagedEachOther(player1, player2)) {
            indicators.push('PROXIMITY_NO_COMBAT');
        }

        // Coordinated movement patterns
        if (this.hasCoordinatedMovement(player1, player2)) {
            indicators.push('COORDINATED_MOVEMENT');
        }

        // Both targeting same third player
        if (this.bothTargetingSameEnemy(player1, player2)) {
            indicators.push('FOCUS_FIRE');
        }

        if (indicators.length >= 2) {
            this.flagForReview(player1, player2, indicators);
        }
    }
}
```

---

## 10. Edge Cases

### 10.1 Match End Scenarios

| Scenario | Resolution |
|----------|------------|
| Last 2 trade kills | Last to deal damage wins |
| All die to storm | Last eliminated wins |
| Server disconnect | Match void, no penalties |
| Mass disconnect | Match continues with remaining |

### 10.2 Player Disconnection

```javascript
class DisconnectionHandler {
    onPlayerDisconnect(player) {
        // Grace period for reconnection
        player.disconnectTime = Time.serverNow();
        player.state = PlayerState.DISCONNECTED;

        // 60 second grace period
        setTimeout(() => {
            if (player.state === PlayerState.DISCONNECTED) {
                this.eliminatePlayer(player, null, 'DISCONNECT');
            }
        }, 60000);
    }

    onPlayerReconnect(player) {
        if (player.state === PlayerState.DISCONNECTED) {
            player.state = PlayerState.ALIVE;
            // Restore at last position
            player.position = player.lastKnownPosition;
        }
    }
}
```

---

## 11. Performance Considerations

### 11.1 60-Player Performance

| System | Target | Notes |
|--------|--------|-------|
| Server Tick | 20 Hz | All 60 players |
| Client Update | 60 FPS | On all devices |
| Network | < 50 KB/s | Per player |
| Memory | < 500 MB | Match total |

### 11.2 Optimization Strategies

- Interest management (only update nearby players)
- LOD for distant players
- Culled VFX beyond visible range
- Batched network updates
- Predictive caching for common events

---

## 12. Testing Requirements

### 12.1 Functional Tests

| Test | Description |
|------|-------------|
| Full match flow | Queue → Victory |
| 60-player match | Maximum capacity |
| Bot fill | Correct behavior |
| Placement tracking | Accurate positions |
| Rewards calculation | Correct XP |

### 12.2 Stress Tests

| Test | Target |
|------|--------|
| Concurrent matches | 100+ |
| Peak queue | 10,000 players |
| Match duration | Consistent 10-15 min |

### 12.3 Playtest Criteria

| Criteria | Target |
|----------|--------|
| Match enjoyment | > 4/5 rating |
| Queue time satisfaction | > 90% acceptable |
| Win feeling | "Rewarding" |
| Loss fairness | "Fair" perception |

---

## 13. Appendices

### Appendix A: Server Capacity Planning

| Metric | Value |
|--------|-------|
| Players per server | 60 |
| Matches per server | 1 |
| Servers per region | Dynamic scaling |
| Minimum active | 10 per region |

### Appendix B: Queue Priority

| Factor | Weight |
|--------|--------|
| Wait time | High |
| Skill matching | Medium (casual) |
| Region | Critical |
| Party size | N/A (solo) |

### Appendix C: Milestone Announcements

| Remaining | Announcement |
|-----------|--------------|
| 30 | "Half the pirates remain!" |
| 20 | "20 pirates left!" |
| 10 | "Final 10!" |
| 5 | "Final 5 - Fight for glory!" |
| 2 | "1v1 - The final duel!" |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
