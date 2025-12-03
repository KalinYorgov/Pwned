# Duo Mode Specification

## Overview

This document specifies the Duo Mode for Plunderstorm Mobile, a team-based battle royale mode where teams of 2 compete to be the last team standing. Includes teammate systems, downed/revive mechanics, and party features.

## Mode Configuration

### Match Settings

| Setting | Value |
|---------|-------|
| Players | 30-60 (15-30 teams) |
| Team Size | 2 |
| Win Condition | Last team with alive member(s) |
| Match Duration | 12-18 minutes |
| Fill Option | Yes (solo queue fills with random) |

### Matchmaking Rules

```typescript
interface DuoMatchmakingConfig {
  minPlayers: 30;
  maxPlayers: 60;
  teamSize: 2;
  allowPartialTeams: true;      // Solo queue allowed
  prioritizeParties: true;      // Pre-made parties prioritized
  skillMatchingEnabled: true;   // SBMM for ranked
  maxWaitTime: 60;              // seconds before bot fill
  botFillEnabled: true;
}
```

## Party System

### Party Formation

```typescript
interface Party {
  partyId: string;
  leaderId: string;
  members: PartyMember[];
  maxSize: 2;
  gameMode: 'duo';
  status: PartyStatus;
  inviteCode?: string;
}

interface PartyMember {
  playerId: string;
  displayName: string;
  avatarUrl: string;
  isLeader: boolean;
  isReady: boolean;
  level: number;
}

enum PartyStatus {
  FORMING = 'forming',
  READY = 'ready',
  QUEUING = 'queuing',
  IN_MATCH = 'in_match'
}
```

### Party Actions

| Action | Leader | Member |
|--------|--------|--------|
| Invite Player | ✓ | ✗ |
| Kick Member | ✓ | ✗ |
| Start Queue | ✓ | ✗ |
| Leave Party | ✓ | ✓ |
| Set Ready | ✓ | ✓ |
| Change Mode | ✓ | ✗ |

### Invite System

```typescript
// Invite methods
interface InviteOptions {
  // Direct invite by player ID/name
  directInvite: (playerId: string) => Promise<InviteResult>;

  // Generate shareable code
  createInviteCode: () => string;  // "ABC123"

  // Join via code
  joinByCode: (code: string) => Promise<JoinResult>;

  // Recent players list
  getRecentPlayers: () => RecentPlayer[];

  // Friends list invite
  inviteFromFriends: (friendId: string) => Promise<InviteResult>;
}

// Invite code format: 6 alphanumeric characters
// Expires after 5 minutes or party fills
```

### Party UI

```
┌─────────────────────────────────────────────┐
│  DUO PARTY                    [Invite Code] │
├─────────────────────────────────────────────┤
│  ┌─────┐                                    │
│  │ 👑  │  CaptainJack (You)    ✓ Ready     │
│  │Avatar│  Level 42                         │
│  └─────┘                                    │
│  ┌─────┐                                    │
│  │     │  Waiting for teammate...          │
│  │  +  │  [Invite Friend] [Use Code]       │
│  └─────┘                                    │
├─────────────────────────────────────────────┤
│        [Start Queue - Need 2 Ready]         │
└─────────────────────────────────────────────┘
```

## Teammate Features

### Teammate Visibility

#### Minimap Indicators

```
┌───────────────────┐
│    🔵 (You)       │
│         ↑         │
│    35m            │  ← Distance to teammate
│         🟢        │  ← Teammate icon
│   (Teammate)      │
└───────────────────┘
```

#### Full Map View

- Teammate shown as distinct icon (green pirate icon)
- Teammate's current direction arrow
- Teammate's ping markers visible
- Teammate's drop location during drop phase

### Teammate HUD Elements

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  [Teammate Panel - Bottom Left]                         │
│  ┌───────────────────────────────────┐                 │
│  │ 🟢 TreasureHunter                 │                 │
│  │ HP: ████████████░░  85/100        │                 │
│  │ Status: Alive | 45m away          │                 │
│  │ [Abilities: ⚔️💨🔥🌊]              │                 │
│  └───────────────────────────────────┘                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Teammate Status States

| Status | Icon | Description |
|--------|------|-------------|
| Alive | 🟢 | Normal state |
| Combat | ⚔️ | Taking/dealing damage |
| Low HP | 🟡 | Below 30% HP |
| Downed | 🔴 | In downed state |
| Dead | 💀 | Eliminated (can spectate) |
| Disconnected | ⚫ | Lost connection |

## Downed State System

### Downed Mechanics

When a player's HP reaches 0, they enter a "downed" state instead of being eliminated immediately.

```typescript
interface DownedState {
  playerId: string;
  downedAt: number;
  bleedoutTimer: number;     // 30 seconds default
  bleedoutRate: number;      // HP lost per second
  currentHP: number;         // Downed HP pool (separate)
  maxDownedHP: number;       // 100 HP downed pool
  canCrawl: boolean;
  crawlSpeed: number;        // 1.5 m/s
  downedCount: number;       // Increases bleedout rate
}
```

### Downed State Rules

| Rule | Value |
|------|-------|
| Downed HP | 100 |
| Bleedout Time | 30 seconds |
| Crawl Speed | 1.5 m/s |
| Can Use Abilities | No |
| Can Be Damaged | Yes |
| Execution Damage | 50 (instant finish) |

### Bleedout Scaling

Repeated downs make bleedout faster:

| Down Count | Bleedout Time | Rate |
|------------|---------------|------|
| 1st | 30s | 3.33 HP/s |
| 2nd | 20s | 5 HP/s |
| 3rd | 15s | 6.67 HP/s |
| 4th+ | 10s | 10 HP/s |

### Downed Player Actions

```typescript
interface DownedActions {
  // Movement
  crawl: (direction: Vector2) => void;  // Slow movement

  // Communication
  pingHelp: () => void;                 // "Need revive!" ping
  pingEnemy: (position: Vector3) => void; // Can still ping enemies

  // Self actions
  giveUp: () => void;                   // Skip to death (after 5s)
}
```

### Downed Visual Feedback

```
Player Model:
- Falls to ground animation
- Crawling animation when moving
- Reaching hand animation toward teammate
- Red screen edge vignette
- Heartbeat audio

HUD (Downed Player):
┌─────────────────────────────────────────────┐
│        ⚠️ YOU ARE DOWNED! ⚠️               │
│                                             │
│   ████████████████░░░░  23.5s              │
│                                             │
│   Teammate is 25m away...                  │
│   [Hold to Give Up - 5s]                   │
└─────────────────────────────────────────────┘
```

## Revive System

### Revive Mechanics

```typescript
interface ReviveAction {
  reviverId: string;
  targetId: string;
  channelDuration: number;    // 5 seconds
  currentProgress: number;
  reviveHP: number;           // 50% of max HP
  interruptible: boolean;     // Yes, by damage
}
```

### Revive Process

1. **Initiate:** Alive teammate approaches downed ally (within 2m)
2. **Channel:** Hold interact button for 5 seconds
3. **Progress:** Circular progress indicator shows completion
4. **Interrupt:** Taking damage cancels revive (progress resets)
5. **Complete:** Downed player restored with 50% HP

### Revive UI

```
[Reviver View]
┌─────────────────────────────────────────────┐
│   Reviving TreasureHunter...               │
│                                             │
│          ⭕⭕⭕⭕⭕                          │
│          [████████░░] 3.2s                  │
│                                             │
│   Stay close! Don't take damage!           │
└─────────────────────────────────────────────┘

[Downed Player View]
┌─────────────────────────────────────────────┐
│   CaptainJack is reviving you!             │
│                                             │
│          [████████░░] 3.2s                  │
│                                             │
│   Hold still!                              │
└─────────────────────────────────────────────┘
```

### Revive Audio

| Event | Sound |
|-------|-------|
| Revive Start | Healing chime begins |
| Revive Progress | Ascending tone loop |
| Revive Interrupted | Sharp negative tone |
| Revive Complete | Triumphant fanfare |

## Communication System

### Ping System

```typescript
interface PingType {
  id: string;
  name: string;
  icon: string;
  duration: number;
  voiceLine?: string;
}

const PING_TYPES: PingType[] = [
  { id: 'enemy', name: 'Enemy Here', icon: '⚠️', duration: 5000 },
  { id: 'loot', name: 'Loot Here', icon: '📦', duration: 10000 },
  { id: 'going', name: 'Going Here', icon: '🚩', duration: 8000 },
  { id: 'help', name: 'Need Help', icon: '🆘', duration: 5000 },
  { id: 'danger', name: 'Danger', icon: '☠️', duration: 5000 },
  { id: 'defend', name: 'Defend Here', icon: '🛡️', duration: 8000 }
];
```

### Ping Wheel (Mobile)

```
         [Enemy]
            ↑
  [Loot] ← ● → [Going]
            ↓
         [Help]

Touch and drag to select ping type
Release to place at crosshair location
```

### Quick Ping

- Single tap: Context-aware ping
  - On enemy: "Enemy Here"
  - On loot: "Loot Here"
  - On ground: "Going Here"
  - When downed: "Need Help"

### Voice Chat (Optional)

```typescript
interface VoiceChatConfig {
  enabled: boolean;
  pushToTalk: boolean;
  voiceActivation: boolean;
  teammateOnly: true;         // No all-chat
  proximityChat: false;       // Full team range
  mutedPlayers: string[];
}
```

## Team Elimination

### Elimination Conditions

```typescript
function checkTeamEliminated(team: Team): boolean {
  const alive = team.members.filter(m =>
    m.status === 'alive' || m.status === 'downed'
  );

  // Team eliminated when NO members are alive or downed
  return alive.length === 0;
}
```

### Elimination Flow

```
Player A Downed → Player B Alive → Team Still Active
          ↓
Player A Bleeds Out → Player A Dead → Player B Alive → Team Still Active
          ↓
Player B Downed → No Alive Players → Both Dead → TEAM ELIMINATED
```

### Placement Calculation

```typescript
function calculatePlacement(eliminatedTeams: Team[]): number {
  // Placement based on order of team elimination
  // Last team eliminated = 2nd place
  // Second-to-last = 3rd place, etc.
  return totalTeams - eliminatedTeams.length + 1;
}
```

## Loot Sharing

### Loot Ping System

```typescript
interface LootPing {
  pingerId: string;
  lootId: string;
  lootType: LootType;
  abilityId?: string;
  rarity: Rarity;
  position: Vector3;
  timestamp: number;
}

// When pinging loot
function pingLoot(player: Player, loot: LootItem): void {
  const ping: LootPing = {
    pingerId: player.id,
    lootId: loot.id,
    lootType: loot.type,
    abilityId: loot.abilityId,
    rarity: loot.rarity,
    position: loot.position,
    timestamp: Date.now()
  };

  // Show to teammate
  sendToTeammate(player.teamId, ping);
}
```

### Loot Ping Display

```
[On Minimap]
📦 with ability icon - shows pinged loot location

[In World]
┌────────────────────┐
│ ⚔️ Cutlass Slash   │  ← Ability name
│ [Rare] ★★☆        │  ← Rarity & rank
│ Pinged by Jack     │
│ 15m away           │
└────────────────────┘
```

### Dibs System (Optional)

```typescript
// Teammate can call "dibs" on pinged loot
function callDibs(player: Player, lootPing: LootPing): void {
  lootPing.dibsPlayerId = player.id;

  // Visual indicator changes to show dibs
  // Other teammate sees "Dibs called" message
}
```

## Spectator Mode (After Death)

### Spectating Teammate

When both players die, they can spectate remaining teams.

```typescript
interface SpectatorState {
  spectatorId: string;
  currentTarget: string;
  canSpectateEnemies: boolean;  // Only after team eliminated
  spectatorUI: SpectatorUI;
}
```

### Spectator Rules

| Condition | Can Spectate |
|-----------|--------------|
| Teammate alive | Teammate only |
| Both dead | Any remaining player |
| Can chat | Spectators only (no info leak) |
| Can ping | No |

## Network Synchronization

### Team State Sync

```typescript
interface TeamSyncPacket {
  teamId: string;
  members: {
    playerId: string;
    status: PlayerStatus;
    position: Vector3;
    health: number;
    maxHealth: number;
    downedHP?: number;
    abilities: string[];
  }[];
  activePings: Ping[];
  reviveInProgress?: ReviveState;
}

// Sync rate: 10 Hz for teammate data
```

### Revive Sync

```typescript
interface ReviveSyncPacket {
  reviverId: string;
  targetId: string;
  progress: number;        // 0.0 - 1.0
  isActive: boolean;
  interrupted: boolean;
}

// Sync rate: 20 Hz during active revive
```

## Analytics Events

```typescript
// Team formation
analytics.track("duo_party_formed", {
  partyId: string,
  isFriends: boolean,      // Pre-made vs random fill
  method: 'invite' | 'code' | 'random'
});

// Revive tracking
analytics.track("revive_attempt", {
  teamId: string,
  reviverId: string,
  targetId: string,
  success: boolean,
  duration: number,
  interruptReason?: string
});

// Team performance
analytics.track("duo_match_complete", {
  teamId: string,
  placement: number,
  totalKills: number,
  revives: number,
  pingsUsed: number,
  downedCount: number[]
});
```

## UI Components

### Pre-Game Party Screen

```
┌─────────────────────────────────────────────────────────┐
│                    DUO MODE                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   YOUR TEAM                                             │
│   ┌──────────────────────────────────────────────┐     │
│   │ [Avatar] CaptainJack    Level 42  ✓ Ready    │     │
│   │ [Avatar] TreasureHunter Level 38  ✓ Ready    │     │
│   └──────────────────────────────────────────────┘     │
│                                                         │
│   Party Code: ABC123  [Copy]                           │
│                                                         │
│   ┌────────────────────────────────────────────────┐   │
│   │              [FIND MATCH]                      │   │
│   └────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### In-Match Teammate Panel

```
┌────────────────────────────────┐
│ 🟢 TreasureHunter              │
│ ████████████░░░░  75/100 HP   │
│ 📍 45m NE | ⚔️ In Combat      │
│ [💨][🔥][⚔️][🌊]              │
└────────────────────────────────┘
```

## Testing Requirements

### Functional Tests

- [ ] Party formation works correctly
- [ ] Random fill matches solo players
- [ ] Downed state triggers at 0 HP
- [ ] Revive channel completes correctly
- [ ] Revive interrupted by damage
- [ ] Team eliminated when both dead
- [ ] Ping system works reliably
- [ ] Spectator mode after death

### Network Tests

- [ ] Teammate position syncs accurately
- [ ] Revive progress syncs smoothly
- [ ] High latency doesn't break revive
- [ ] Disconnected teammate handled

### Balance Tests

- [ ] Revive time feels balanced
- [ ] Bleedout time allows reasonable rescue
- [ ] Downed player not too vulnerable

## Success Metrics

| Metric | Target |
|--------|--------|
| Duo queue usage | 40%+ of matches |
| Revive success rate | 50-60% |
| Party vs random fill | 60% pre-made |
| Duo-specific feedback | Positive sentiment |

## Dependencies

- **GAME-020**: Solo Mode (base match systems)
- **BACK-002**: Party/Squad System
- **UX-010**: Ping/Communication System
