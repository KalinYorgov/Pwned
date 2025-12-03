# Mode Selection Screen Specification

## Overview

This document specifies the Mode Selection Screen for Plunderstorm Mobile, where players choose their game mode before matchmaking. The screen provides clear mode descriptions, player counts, wait times, and manages the matchmaking queue.

## Screen Layout

### Mode Selection UI

```
┌─────────────────────────────────────────────────────────────────────────┐
│  [← Back]            SELECT MODE                     [Map: Tortuga Bay]│
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │  ⚔️ SOLO                                        60 Players     │   │
│  │  Every pirate for themselves!                                  │   │
│  │                                                                 │   │
│  │  🟢 125 in queue                    Est. wait: <15s            │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │  👥 DUO                                       60 Players (30T)  │   │
│  │  Team up with a friend or find a partner!                      │   │
│  │                                                                 │   │
│  │  🟢 89 in queue                     Est. wait: <20s            │   │
│  │  [Fill Teammate] [Invite Friend]                               │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │  🏆 RANKED SOLO                                    Unlocks L10 │   │
│  │  Competitive matches with skill-based matchmaking              │   │
│  │                                                                 │   │
│  │  🔒 Reach Level 10 to unlock                                   │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │  🎯 PRACTICE                                     vs Bots       │   │
│  │  Train your skills against AI opponents                        │   │
│  │                                                                 │   │
│  │  🟢 Always available                       Instant start       │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  🌟 LIMITED TIME: Kraken Attack!              Ends in 2d 14h   │   │
│  │  Battle the Kraken boss with 20 players!                       │   │
│  │  🟢 45 in queue                                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Game Modes

### Mode Configuration

```typescript
interface GameMode {
  id: string;
  name: string;
  icon: string;
  description: string;
  shortDescription: string;

  // Player settings
  playerCount: number | PlayerCountRange;
  teamSize: number;
  teamsCount?: number;

  // Availability
  status: ModeStatus;
  unlockRequirement?: UnlockRequirement;
  schedule?: ModeSchedule;

  // Queue settings
  queueConfig: QueueConfig;

  // Visual
  bannerImage?: string;
  accentColor: string;
  featured: boolean;
}

interface PlayerCountRange {
  min: number;
  max: number;
  optimal: number;
}

enum ModeStatus {
  AVAILABLE = 'available',
  LOCKED = 'locked',
  COMING_SOON = 'coming_soon',
  MAINTENANCE = 'maintenance',
  LIMITED_TIME = 'limited_time'
}

interface UnlockRequirement {
  type: 'level' | 'matches_played' | 'achievement';
  value: number;
  description: string;
}

interface ModeSchedule {
  startTime: Date;
  endTime: Date;
  recurring?: RecurringSchedule;
}
```

### Available Modes

```typescript
const GAME_MODES: GameMode[] = [
  {
    id: 'solo',
    name: 'Solo',
    icon: 'icon_solo',
    description: 'Every pirate for themselves! Be the last one standing.',
    shortDescription: 'Free-for-all battle royale',
    playerCount: { min: 30, max: 60, optimal: 60 },
    teamSize: 1,
    status: ModeStatus.AVAILABLE,
    queueConfig: {
      matchmakingType: 'skill_based',
      maxWaitTime: 60,
      botFillEnabled: true,
      botFillDelay: 30
    },
    accentColor: '#FFD700',
    featured: false
  },
  {
    id: 'duo',
    name: 'Duo',
    icon: 'icon_duo',
    description: 'Team up with a friend or find a partner! Last team standing wins.',
    shortDescription: 'Pairs battle royale',
    playerCount: { min: 30, max: 60, optimal: 60 },
    teamSize: 2,
    teamsCount: 30,
    status: ModeStatus.AVAILABLE,
    queueConfig: {
      matchmakingType: 'skill_based',
      maxWaitTime: 60,
      botFillEnabled: true,
      botFillDelay: 30,
      allowSoloQueue: true,
      partnerFillEnabled: true
    },
    accentColor: '#00BFFF',
    featured: false
  },
  {
    id: 'ranked_solo',
    name: 'Ranked Solo',
    icon: 'icon_ranked',
    description: 'Competitive matches with skill-based matchmaking. Climb the ranks!',
    shortDescription: 'Competitive ranked mode',
    playerCount: { min: 40, max: 60, optimal: 60 },
    teamSize: 1,
    status: ModeStatus.LOCKED,
    unlockRequirement: {
      type: 'level',
      value: 10,
      description: 'Reach Level 10 to unlock'
    },
    queueConfig: {
      matchmakingType: 'ranked',
      maxWaitTime: 120,
      botFillEnabled: false,
      minPlayersToStart: 40
    },
    accentColor: '#9932CC',
    featured: false
  },
  {
    id: 'practice',
    name: 'Practice',
    icon: 'icon_practice',
    description: 'Train your skills against AI opponents. Perfect your abilities!',
    shortDescription: 'Bot matches for training',
    playerCount: 60,
    teamSize: 1,
    status: ModeStatus.AVAILABLE,
    queueConfig: {
      matchmakingType: 'instant',
      maxWaitTime: 0,
      botFillEnabled: true,
      fullBotMatch: true
    },
    accentColor: '#32CD32',
    featured: false
  }
];
```

### Limited Time Modes (LTM)

```typescript
interface LimitedTimeMode extends GameMode {
  eventId: string;
  schedule: ModeSchedule;
  rewards?: LTMRewards;
  leaderboard?: boolean;
}

const EXAMPLE_LTM: LimitedTimeMode = {
  id: 'kraken_attack',
  name: 'Kraken Attack!',
  icon: 'icon_kraken',
  description: 'Battle the legendary Kraken with 20 players! Work together to defeat the beast.',
  shortDescription: 'PvE Boss Battle',
  playerCount: 20,
  teamSize: 20,
  status: ModeStatus.LIMITED_TIME,
  schedule: {
    startTime: new Date('2024-01-15'),
    endTime: new Date('2024-01-22')
  },
  queueConfig: {
    matchmakingType: 'quick',
    maxWaitTime: 90,
    botFillEnabled: false,
    minPlayersToStart: 15
  },
  accentColor: '#FF4500',
  featured: true,
  bannerImage: 'banner_kraken_attack.jpg',
  eventId: 'event_kraken_s2',
  rewards: {
    participation: { gold: 100, xp: 200 },
    victory: { gold: 500, xp: 1000, cosmetic: 'kraken_slayer_title' }
  },
  leaderboard: true
};
```

## Mode Cards

### Card Design

```typescript
interface ModeCard {
  mode: GameMode;
  state: CardState;
  queueInfo: QueueInfo;
  actions: CardAction[];
}

enum CardState {
  AVAILABLE = 'available',
  SELECTED = 'selected',
  LOCKED = 'locked',
  FEATURED = 'featured',
  DISABLED = 'disabled'
}

interface QueueInfo {
  playersInQueue: number;
  estimatedWait: number;        // seconds
  averageWait: number;          // Recent average
  trend: 'faster' | 'slower' | 'stable';
  lastUpdated: Date;
}

interface CardAction {
  id: string;
  label: string;
  icon?: string;
  primary: boolean;
  action: () => void;
}
```

### Card Visual States

```
AVAILABLE STATE:
┌─────────────────────────────────────────────────────────────────┐
│  ⚔️ SOLO                                        60 Players     │
│  Every pirate for themselves!                                  │
│  🟢 125 in queue                    Est. wait: <15s            │
└─────────────────────────────────────────────────────────────────┘
- White background with subtle border
- Green dot for healthy queue
- Tap to select and start matchmaking

SELECTED STATE:
┌─────────────────────────────────────────────────────────────────┐
│  ⚔️ SOLO                              ✓ SELECTED              │
│  Every pirate for themselves!                                  │
│  🟢 125 in queue                    Est. wait: <15s            │
│                                                                 │
│                    [START MATCHMAKING]                          │
└─────────────────────────────────────────────────────────────────┘
- Highlighted border with accent color
- Checkmark indicator
- Start button appears

LOCKED STATE:
┌─────────────────────────────────────────────────────────────────┐
│  🏆 RANKED SOLO                          🔒 LOCKED             │
│  Competitive matches with skill-based matchmaking              │
│                                                                 │
│  Unlock at Level 10 (You: Level 7)                             │
│  [████████████░░░░]  70%                                       │
└─────────────────────────────────────────────────────────────────┘
- Grayed out appearance
- Lock icon
- Progress toward unlock shown

FEATURED/LTM STATE:
┌─────────────────────────────────────────────────────────────────┐
│  ★ LIMITED TIME ★                           Ends in 2d 14h    │
│  🌟 KRAKEN ATTACK!                          20 Players        │
│  Battle the Kraken boss with other pirates!                    │
│  🟢 45 in queue                              Special Rewards!  │
└─────────────────────────────────────────────────────────────────┘
- Special border animation (glow)
- Event banner background
- Countdown timer
- Reward indicator
```

## Queue Information

### Queue Status Display

```typescript
interface QueueStatusConfig {
  refreshInterval: 5000;        // ms
  showExactCount: boolean;
  showEstimatedWait: boolean;
  showTrend: boolean;
}

function formatQueueInfo(info: QueueInfo): QueueDisplay {
  // Format player count
  let playersText: string;
  if (info.playersInQueue > 1000) {
    playersText = `${Math.floor(info.playersInQueue / 1000)}k+ in queue`;
  } else {
    playersText = `${info.playersInQueue} in queue`;
  }

  // Format wait time
  let waitText: string;
  if (info.estimatedWait < 15) {
    waitText = 'Est. wait: <15s';
  } else if (info.estimatedWait < 60) {
    waitText = `Est. wait: ~${Math.round(info.estimatedWait / 10) * 10}s`;
  } else {
    waitText = `Est. wait: ~${Math.round(info.estimatedWait / 60)}min`;
  }

  // Queue health indicator
  let healthColor: string;
  if (info.estimatedWait < 20) {
    healthColor = '#00FF00';  // Green - healthy
  } else if (info.estimatedWait < 60) {
    healthColor = '#FFFF00';  // Yellow - moderate
  } else {
    healthColor = '#FF0000';  // Red - long wait
  }

  return { playersText, waitText, healthColor };
}
```

### Queue Health Indicators

| Queue Status | Color | Icon | Wait Time |
|--------------|-------|------|-----------|
| Healthy | Green | 🟢 | < 20s |
| Moderate | Yellow | 🟡 | 20s - 60s |
| Long Wait | Orange | 🟠 | 1-3 min |
| Very Long | Red | 🔴 | > 3 min |
| Offline | Gray | ⚫ | N/A |

## Duo Mode Special Features

### Team Formation UI

```
┌─────────────────────────────────────────────────────────────────┐
│  👥 DUO                                       60 Players        │
│  Team up with a friend or find a partner!                      │
│                                                                 │
│  YOUR TEAM:                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ [Your Avatar]  You          ✓ Ready                     │   │
│  │ [Empty Slot]   Waiting...   [Fill] [Invite]             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  🟢 89 in queue                     Est. wait: <20s            │
│                                                                 │
│  [ ] Fill with random teammate                                  │
│                                                                 │
│                    [START MATCHMAKING]                          │
└─────────────────────────────────────────────────────────────────┘
```

### Duo Options

```typescript
interface DuoModeOptions {
  fillWithRandom: boolean;      // Auto-fill teammate
  inviteFriend: () => void;
  inviteRecent: () => void;
  enterPartyCode: () => void;
}

interface DuoTeamState {
  player1: {
    id: string;
    name: string;
    avatar: string;
    isReady: boolean;
    isLeader: boolean;
  };
  player2?: {
    id: string;
    name: string;
    avatar: string;
    isReady: boolean;
    status: 'connected' | 'invited' | 'fill';
  };
}

function renderDuoTeamSlot(slot: 'filled' | 'empty' | 'pending', data?: any): void {
  switch (slot) {
    case 'filled':
      // Show teammate info
      renderTeammateInfo(data);
      renderReadyStatus(data.isReady);
      renderLeaveButton();
      break;

    case 'pending':
      // Show pending invite
      renderPendingInvite(data.invitedPlayer);
      renderCancelButton();
      break;

    case 'empty':
      // Show options
      renderEmptySlotOptions();
      break;
  }
}
```

### Invite Flow

```
Invite Options:
┌─────────────────────────────────────────────────────────────────┐
│  INVITE TEAMMATE                                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FRIENDS ONLINE (5)                                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ [Avatar] CaptainJack        Level 42    [Invite]        │   │
│  │ [Avatar] TreasureHunter     Level 38    [Invite]        │   │
│  │ [Avatar] SeaDog99           Level 25    [Invite]        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  RECENT PLAYERS                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ [Avatar] WaveRider          1 hour ago  [Invite]        │   │
│  │ [Avatar] SaltySailor        3 hours ago [Invite]        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ─────────────── OR ───────────────                            │
│                                                                 │
│  PARTY CODE: [______]  [Join]                                  │
│                                                                 │
│  YOUR CODE: ABC123  [Copy] [Share]                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Ranked Mode

### Ranked Display

```
┌─────────────────────────────────────────────────────────────────┐
│  🏆 RANKED SOLO                               Season 2          │
│  Competitive matches with skill-based matchmaking               │
│                                                                 │
│  YOUR RANK:                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │    [Rank Icon]                                          │   │
│  │    GOLD II                                              │   │
│  │    1,245 RP                                             │   │
│  │    [████████░░]  Next: Gold I (55 RP)                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  🟢 312 in queue                    Est. wait: <30s            │
│  Match quality: High                                            │
│                                                                 │
│                    [START RANKED MATCH]                         │
└─────────────────────────────────────────────────────────────────┘
```

### Rank Display Component

```typescript
interface RankDisplay {
  tier: RankTier;
  division: number;           // I, II, III, IV
  rankPoints: number;
  pointsToNext: number;
  peakRank?: RankTier;
  seasonReset?: Date;
}

enum RankTier {
  BRONZE = 'bronze',
  SILVER = 'silver',
  GOLD = 'gold',
  PLATINUM = 'platinum',
  DIAMOND = 'diamond',
  MASTER = 'master',
  GRANDMASTER = 'grandmaster'
}

function getRankDisplayInfo(rank: RankDisplay): RankDisplayInfo {
  return {
    icon: `rank_${rank.tier}_${rank.division}`,
    name: `${capitalize(rank.tier)} ${romanNumeral(rank.division)}`,
    color: RANK_COLORS[rank.tier],
    progress: rank.rankPoints / (rank.rankPoints + rank.pointsToNext),
    progressText: `${rank.pointsToNext} RP to next rank`
  };
}
```

## Map Selection

### Map Selector (Optional)

```
┌─────────────────────────────────────────────────────────────────┐
│  SELECT MAP                                    [Random ✓]       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐                  │
│  │           │  │           │  │           │                  │
│  │ [Preview] │  │ [Preview] │  │ [Preview] │                  │
│  │           │  │           │  │           │                  │
│  │  Tortuga  │  │  Skull    │  │  Mermaid  │                  │
│  │    Bay    │  │  Island   │  │   Cove    │                  │
│  │   ✓ ✓     │  │   🔒      │  │   ✓       │                  │
│  └───────────┘  └───────────┘  └───────────┘                  │
│                                                                 │
│  Currently selected: Tortuga Bay                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Map Configuration

```typescript
interface MapConfig {
  id: string;
  name: string;
  description: string;
  previewImage: string;
  thumbnailImage: string;
  playerCapacity: PlayerCountRange;
  supportedModes: string[];
  unlockRequirement?: UnlockRequirement;
  status: 'available' | 'locked' | 'coming_soon';
}

const MAPS: MapConfig[] = [
  {
    id: 'tortuga_bay',
    name: 'Tortuga Bay',
    description: 'The classic pirate haven with beaches, jungle, and a central port town.',
    previewImage: 'map_tortuga_preview.jpg',
    thumbnailImage: 'map_tortuga_thumb.jpg',
    playerCapacity: { min: 30, max: 60, optimal: 60 },
    supportedModes: ['solo', 'duo', 'ranked_solo', 'practice'],
    status: 'available'
  },
  {
    id: 'skull_island',
    name: 'Skull Island',
    description: 'A mysterious volcanic island with treacherous caves and ancient ruins.',
    previewImage: 'map_skull_preview.jpg',
    thumbnailImage: 'map_skull_thumb.jpg',
    playerCapacity: { min: 40, max: 60, optimal: 60 },
    supportedModes: ['solo', 'duo'],
    unlockRequirement: {
      type: 'matches_played',
      value: 25,
      description: 'Play 25 matches to unlock'
    },
    status: 'locked'
  }
];
```

## Matchmaking Flow

### Matchmaking States

```typescript
enum MatchmakingState {
  IDLE = 'idle',
  SEARCHING = 'searching',
  FOUND = 'found',
  CONFIRMING = 'confirming',
  LOADING = 'loading',
  FAILED = 'failed',
  CANCELLED = 'cancelled'
}

interface MatchmakingStatus {
  state: MatchmakingState;
  mode: GameMode;
  startTime: Date;
  elapsedTime: number;
  estimatedTime: number;
  playersFound?: number;
  playersNeeded?: number;
  matchId?: string;
}
```

### Matchmaking UI

```
SEARCHING STATE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    ⚔️ SEARCHING FOR MATCH                      │
│                                                                 │
│                    [Spinning Animation]                        │
│                                                                 │
│                    Solo - Tortuga Bay                          │
│                                                                 │
│                    Time: 0:15                                  │
│                    Estimated: <30s                             │
│                                                                 │
│                    Players found: 42/60                        │
│                                                                 │
│                    [CANCEL]                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

MATCH FOUND STATE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    ⚓ MATCH FOUND!                              │
│                                                                 │
│                    [Ship Animation]                            │
│                                                                 │
│                    Solo - Tortuga Bay                          │
│                    60 Players                                  │
│                                                                 │
│                    Loading match...                            │
│                    [████████████░░░░]  75%                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Matchmaking Implementation

```typescript
class MatchmakingManager {
  private status: MatchmakingStatus;
  private updateInterval: number;

  async startMatchmaking(mode: GameMode, options: MatchmakingOptions): Promise<void> {
    this.status = {
      state: MatchmakingState.SEARCHING,
      mode: mode,
      startTime: new Date(),
      elapsedTime: 0,
      estimatedTime: mode.queueConfig.maxWaitTime
    };

    // Show matchmaking UI
    showMatchmakingOverlay(this.status);

    // Start status updates
    this.updateInterval = setInterval(() => this.updateStatus(), 1000);

    try {
      // Request matchmaking from server
      const match = await requestMatchmaking(mode.id, options);

      this.status.state = MatchmakingState.FOUND;
      this.status.matchId = match.id;

      // Show match found
      showMatchFoundUI(match);

      // Transition to loading
      await loadMatch(match);

    } catch (error) {
      this.handleMatchmakingError(error);
    }
  }

  cancelMatchmaking(): void {
    this.status.state = MatchmakingState.CANCELLED;
    clearInterval(this.updateInterval);
    sendCancelRequest();
    hideMatchmakingOverlay();
  }

  private updateStatus(): void {
    this.status.elapsedTime = (Date.now() - this.status.startTime.getTime()) / 1000;

    // Update UI
    updateMatchmakingUI(this.status);

    // Check for timeout
    if (this.status.elapsedTime > this.status.mode.queueConfig.maxWaitTime) {
      this.handleTimeout();
    }
  }

  private handleTimeout(): void {
    if (this.status.mode.queueConfig.botFillEnabled) {
      // Fill with bots and start
      requestBotFill();
    } else {
      // Show timeout message
      this.status.state = MatchmakingState.FAILED;
      showTimeoutMessage();
    }
  }
}
```

## Error Handling

### Error States

```typescript
enum MatchmakingError {
  NETWORK_ERROR = 'network_error',
  SERVER_ERROR = 'server_error',
  TIMEOUT = 'timeout',
  BANNED = 'banned',
  MAINTENANCE = 'maintenance',
  PARTY_ERROR = 'party_error',
  RANK_RESTRICTION = 'rank_restriction'
}

interface ErrorDisplay {
  error: MatchmakingError;
  title: string;
  message: string;
  actions: ErrorAction[];
}

const ERROR_DISPLAYS: Record<MatchmakingError, ErrorDisplay> = {
  [MatchmakingError.NETWORK_ERROR]: {
    title: 'Connection Lost',
    message: 'Unable to connect to matchmaking servers. Check your connection.',
    actions: [
      { label: 'Retry', action: 'retry' },
      { label: 'Cancel', action: 'cancel' }
    ]
  },
  [MatchmakingError.TIMEOUT]: {
    title: 'Queue Timeout',
    message: 'Unable to find a match. The queue may be too long.',
    actions: [
      { label: 'Try Again', action: 'retry' },
      { label: 'Try Different Mode', action: 'change_mode' }
    ]
  },
  [MatchmakingError.MAINTENANCE]: {
    title: 'Server Maintenance',
    message: 'Matchmaking is temporarily unavailable for maintenance.',
    actions: [
      { label: 'OK', action: 'cancel' }
    ]
  }
};
```

## Analytics Events

```typescript
// Mode selection
analytics.track('mode_selected', {
  mode: string,
  previousMode: string,
  timeSinceScreenOpen: number
});

// Matchmaking
analytics.track('matchmaking_started', {
  mode: string,
  map: string,
  partySize: number,
  fillEnabled: boolean
});

analytics.track('matchmaking_completed', {
  mode: string,
  waitTime: number,
  playersInMatch: number,
  botsInMatch: number
});

analytics.track('matchmaking_cancelled', {
  mode: string,
  waitTime: number,
  reason: 'user' | 'timeout' | 'error'
});

// Duo specific
analytics.track('duo_invite_sent', {
  method: 'friend' | 'recent' | 'code'
});

analytics.track('duo_filled_random', {
  waitTime: number
});
```

## Testing Requirements

### Functional Tests

- [ ] All modes display correctly
- [ ] Locked modes show unlock progress
- [ ] Queue info updates in real-time
- [ ] Matchmaking starts and completes
- [ ] Cancel matchmaking works
- [ ] Duo invite flow works
- [ ] Random fill works for duo
- [ ] Map selection works (if enabled)
- [ ] LTM displays with countdown
- [ ] Ranked shows current rank

### Network Tests

- [ ] Queue info syncs accurately
- [ ] Matchmaking handles disconnection
- [ ] Timeout triggers bot fill correctly
- [ ] Party sync works in duo

### Edge Cases

- [ ] Very long queue times handled
- [ ] Server maintenance handled
- [ ] All maps locked handled
- [ ] Solo player in duo (fill required)

## Success Metrics

| Metric | Target |
|--------|--------|
| Mode selection time | < 5 seconds |
| Matchmaking success rate | > 95% |
| Average wait time (Solo) | < 30 seconds |
| Duo fill usage | 20%+ of solo queuers |
| LTM participation | 30%+ when active |

## Dependencies

- **GAME-020**: Solo Mode
- **GAME-021**: Duo Mode
- **GAME-023**: Ranked Mode
- **GAME-024**: Practice Mode
- **BACK-003**: Matchmaking Service
- **BACK-002**: Party System
