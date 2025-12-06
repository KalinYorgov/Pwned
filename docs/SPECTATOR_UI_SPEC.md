# Spectator UI Specification

## Document Information
- **Task ID:** UX-012
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies the spectator mode UI overlay for players watching matches after elimination. The spectator experience should be engaging, informative, and allow players to follow the remaining action while waiting for the match to conclude.

---

## Spectator Mode Entry

### Transition Flow

```typescript
const SPECTATOR_ENTRY_FLOW = {
  onElimination: {
    step1: {
      action: 'Death camera plays',
      duration: 3000, // ms
      shows: 'Killer\'s finishing move or moment of elimination'
    },
    step2: {
      action: 'Elimination summary card',
      duration: 2500,
      shows: ['Killer name', 'Weapon/ability used', 'Your final stats']
    },
    step3: {
      action: 'Spectator prompt',
      options: [
        { label: 'Spectate', action: 'Enter spectator mode' },
        { label: 'Leave Match', action: 'Return to lobby' }
      ],
      timeout: 10000, // Auto-spectate if no choice
      defaultAction: 'spectate'
    }
  },

  squadMode: {
    aliveTeammates: {
      autoSpectate: 'Immediately follow random alive teammate',
      showPrompt: false
    },
    allTeammatesDead: {
      behavior: 'Same as solo mode',
      canSpectateAnyPlayer: true
    }
  }
};
```

---

## UI Layout

### Screen Composition

```
┌─────────────────────────────────────────────────────────────┐
│ [SPECTATING]          Player Name [Kills: 5]    [15 Alive] │  ← Top Bar
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                                                             │
│                     Main Viewport                           │
│                   (Spectated Player)                        │
│                                                             │
│                                                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [◀ Prev]  [▶ Next]   [Free Cam]   [Chat]   [Leave Match]  │  ← Bottom Bar
│                                                             │
│              [ Minimap ]                    [Timer 12:34]   │
└─────────────────────────────────────────────────────────────┘
```

### Layout Specifications

```typescript
const SPECTATOR_UI_LAYOUT = {
  topBar: {
    height: 60, // pixels
    background: 'rgba(0, 0, 0, 0.6)',
    elements: {
      spectatingLabel: {
        position: 'left',
        text: 'SPECTATING',
        style: {
          color: '#FFD700', // Gold
          fontSize: 14,
          fontWeight: 'bold',
          textTransform: 'uppercase'
        }
      },
      playerInfo: {
        position: 'center',
        elements: [
          { type: 'playerName', fontSize: 18, fontWeight: 'bold' },
          { type: 'killCount', fontSize: 14, prefix: 'Kills: ' }
        ]
      },
      matchInfo: {
        position: 'right',
        elements: [
          { type: 'playersRemaining', icon: 'users', fontSize: 16 },
          { type: 'matchTimer', fontSize: 14 }
        ]
      }
    }
  },

  bottomBar: {
    height: 80,
    background: 'rgba(0, 0, 0, 0.5)',
    elements: {
      navigationControls: {
        position: 'left',
        buttons: ['previousPlayer', 'nextPlayer']
      },
      modeControls: {
        position: 'center',
        buttons: ['freeCamera', 'spectatorChat']
      },
      exitControl: {
        position: 'right',
        button: 'leaveMatch'
      }
    }
  },

  minimap: {
    position: 'bottom-left',
    size: 150, // pixels
    margin: 20,
    showsSpectatedPlayer: true,
    highlightColor: '#FFD700'
  },

  timer: {
    position: 'bottom-right',
    margin: 20,
    format: 'MM:SS'
  }
};
```

---

## UI Components

### 1. Spectating Label

```typescript
const SPECTATING_LABEL = {
  states: {
    followingPlayer: {
      text: 'SPECTATING',
      color: '#FFD700',
      animation: 'subtle pulse'
    },
    freeCamera: {
      text: 'FREE CAMERA',
      color: '#4A90D9',
      animation: 'none'
    },
    followingTeammate: {
      text: 'WATCHING TEAMMATE',
      color: '#4CAF50',
      animation: 'none'
    }
  },

  style: {
    background: 'rgba(0, 0, 0, 0.7)',
    padding: { horizontal: 12, vertical: 6 },
    borderRadius: 4,
    border: '1px solid currentColor'
  }
};
```

### 2. Observed Player Info

```typescript
const OBSERVED_PLAYER_INFO = {
  layout: {
    container: {
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center'
    }
  },

  elements: {
    playerName: {
      maxLength: 16, // Truncate with ellipsis
      showPlatformIcon: true, // iOS/Android indicator
      showRankBadge: true // If ranked mode
    },

    stats: {
      kills: {
        icon: 'skull',
        format: '{count}',
        color: '#FF6B6B'
      },
      damageDealt: {
        icon: 'crosshair',
        format: '{amount}',
        color: '#FFB366',
        showOnExpand: true // Hidden by default
      },
      placement: {
        showWhen: 'final_players', // Top 10 remaining
        format: 'Top {remaining}'
      }
    },

    health: {
      showBar: true,
      position: 'below_name',
      width: 120,
      height: 6,
      colors: {
        health: '#4CAF50',
        shield: '#4A90D9',
        background: 'rgba(0, 0, 0, 0.5)'
      }
    },

    equipment: {
      showOnExpand: true,
      elements: [
        'currentWeapon',
        'abilitySlots',
        'itemCount'
      ]
    }
  },

  expandButton: {
    icon: 'chevron_down',
    action: 'Toggle expanded stats view',
    position: 'right of player name'
  }
};
```

### 3. Player Navigation Controls

```typescript
const PLAYER_NAVIGATION = {
  buttons: {
    previousPlayer: {
      icon: 'chevron_left',
      label: 'Prev',
      size: 50,
      behavior: {
        tap: 'Switch to previous player in list',
        hold: 'Cycle through players quickly'
      },
      haptic: 'selection'
    },

    nextPlayer: {
      icon: 'chevron_right',
      label: 'Next',
      size: 50,
      behavior: {
        tap: 'Switch to next player in list',
        hold: 'Cycle through players quickly'
      },
      haptic: 'selection'
    }
  },

  playerList: {
    sortOrder: [
      'Alive teammates first (squad mode)',
      'By kill count (descending)',
      'By proximity to action'
    ],
    excludes: ['Eliminated players', 'Disconnected players']
  },

  transition: {
    type: 'smooth_camera_move',
    duration: 500, // ms
    easing: 'easeInOutCubic'
  },

  swipeGesture: {
    enabled: true,
    direction: 'horizontal',
    swipeLeft: 'nextPlayer',
    swipeRight: 'previousPlayer',
    threshold: 100 // pixels
  }
};
```

### 4. Free Camera Mode

```typescript
const FREE_CAMERA_MODE = {
  toggle: {
    button: {
      icon: 'video_camera',
      label: 'Free Cam',
      size: 50,
      activeState: {
        background: '#4A90D9',
        icon: 'video_camera_filled'
      }
    },
    shortcut: 'Double-tap empty area'
  },

  controls: {
    movement: {
      type: 'virtual_joystick',
      position: 'left_side',
      axes: ['forward/back', 'left/right'],
      speed: {
        normal: 15, // units per second
        fast: 30 // when sprint button held
      }
    },

    rotation: {
      type: 'drag',
      area: 'right_side_of_screen',
      sensitivity: 1.0,
      invertY: 'respect_settings'
    },

    altitude: {
      upButton: { icon: 'arrow_up', position: 'right_of_joystick' },
      downButton: { icon: 'arrow_down', position: 'right_of_joystick' }
    },

    speedBoost: {
      button: { icon: 'fast_forward', label: 'Fast' },
      multiplier: 2.0
    }
  },

  boundaries: {
    withinMapBounds: true,
    maxAltitude: 100, // meters
    minAltitude: 5, // meters
    collisionWithTerrain: false // Camera passes through
  },

  returnToPlayer: {
    button: { icon: 'user', label: 'Follow' },
    behavior: 'Smooth transition back to last spectated player'
  }
};
```

### 5. Players Remaining Counter

```typescript
const PLAYERS_REMAINING = {
  display: {
    icon: 'users',
    format: '{count} Alive',
    position: 'top_right'
  },

  animations: {
    onDecrease: {
      type: 'number_tick_down',
      duration: 300,
      flash: 'brief red highlight'
    },
    finalPlayers: {
      threshold: 5,
      effect: 'Pulsing glow',
      color: '#FF6B6B'
    }
  },

  tooltip: {
    onTap: 'Show remaining player list',
    content: [
      'Player names (if known)',
      'Kill counts',
      'Tap player to spectate'
    ]
  },

  killfeed: {
    position: 'below_counter',
    showRecent: 3,
    fadeTime: 5000, // ms
    format: '{killer} eliminated {victim}'
  }
};
```

### 6. Match Timer

```typescript
const MATCH_TIMER = {
  display: {
    format: 'MM:SS',
    position: 'bottom_right',
    fontSize: 16
  },

  phases: {
    normal: {
      color: '#FFFFFF'
    },
    zoneClosing: {
      color: '#FF6B6B',
      animation: 'pulse',
      showLabel: 'ZONE CLOSING'
    },
    finalCircle: {
      color: '#FF4444',
      animation: 'urgent_pulse'
    }
  },

  zoneTimer: {
    showSeparately: true,
    format: 'Zone in {seconds}s',
    position: 'below_match_timer'
  }
};
```

### 7. Spectator Chat

```typescript
const SPECTATOR_CHAT = {
  isolation: {
    description: 'Spectators can only chat with other spectators',
    cannotMessageAlive: true,
    reason: 'Prevent ghosting/cheating'
  },

  ui: {
    toggle: {
      icon: 'chat_bubble',
      label: 'Chat',
      badgeCount: 'unread messages'
    },

    panel: {
      position: 'bottom_left',
      width: 300,
      height: 200,
      background: 'rgba(0, 0, 0, 0.7)'
    },

    input: {
      placeholder: 'Chat with spectators...',
      maxLength: 100,
      submitButton: 'Send'
    },

    messages: {
      showSender: true,
      timestamp: false,
      maxVisible: 20,
      fadeOldMessages: true
    }
  },

  moderation: {
    profanityFilter: true,
    reportButton: true,
    mutePlayer: true
  },

  quickMessages: {
    enabled: true,
    options: [
      'Nice shot!',
      'RIP',
      'Let\'s go!',
      'gg'
    ]
  }
};
```

### 8. Leave Match Button

```typescript
const LEAVE_MATCH_BUTTON = {
  button: {
    icon: 'exit',
    label: 'Leave',
    size: 50,
    color: '#FF6B6B',
    position: 'bottom_right'
  },

  confirmation: {
    enabled: true,
    title: 'Leave Match?',
    message: 'Are you sure you want to return to the lobby?',
    options: [
      { label: 'Stay', action: 'dismiss' },
      { label: 'Leave', action: 'exitToLobby', color: '#FF6B6B' }
    ]
  },

  squadMode: {
    additionalMessage: 'You can rejoin if teammates are still alive',
    showRejoinOption: false // Cannot rejoin as spectator
  },

  exitBehavior: {
    action: 'Return to main menu',
    showMatchSummary: true, // Brief stats before exiting
    maintainParty: true // Stay in party if in squad
  }
};
```

---

## Camera Modes

### Follow Camera

```typescript
const FOLLOW_CAMERA = {
  default: true,

  behavior: {
    position: 'Third person behind player',
    distance: 8, // meters
    height: 3, // meters above player
    followSmoothing: 0.1, // Lower = more responsive
    lookAhead: true // Camera anticipates movement direction
  },

  playerInput: {
    rotateAroundPlayer: {
      enabled: true,
      input: 'drag on right side',
      returnToDefault: 'slowly when released'
    },
    zoom: {
      enabled: false, // Fixed distance for consistency
      reason: 'Prevent unfair information advantage'
    }
  },

  autoFollow: {
    onCombat: 'Tighten follow distance',
    onMovement: 'Smooth tracking',
    onAbilityUse: 'Brief dramatic angle if applicable'
  }
};
```

### First Person View (Optional)

```typescript
const FIRST_PERSON_VIEW = {
  enabled: false, // Disabled by default
  unlockCondition: 'Developer option or future feature',

  ifEnabled: {
    toggle: 'Double-tap on player',
    shows: 'Player\'s exact POV',
    limitations: [
      'May cause motion sickness',
      'UI elements may be hidden'
    ]
  }
};
```

---

## Mobile Optimizations

### Touch Controls

```typescript
const SPECTATOR_TOUCH_CONTROLS = {
  gestures: {
    swipeHorizontal: 'Switch players',
    doubleTap: 'Toggle free camera',
    pinch: 'N/A (zoom disabled)',
    longPress: 'Open player action menu'
  },

  buttonSizing: {
    minimumTouchTarget: 44, // pixels
    spacing: 16, // pixels between buttons
    bottomSafeArea: true // Account for gesture bar
  },

  oneHandedMode: {
    enabled: true,
    layout: 'All controls reachable with thumb',
    preferredSide: 'auto-detect (left/right handed)'
  }
};
```

### Performance Considerations

```typescript
const SPECTATOR_PERFORMANCE = {
  rendering: {
    targetFPS: 60,
    qualityReduction: 'None (spectating is lighter than playing)',
    effectsVisible: true // Show all VFX
  },

  networking: {
    updateRate: 20, // Hz (lower than player, reduces bandwidth)
    interpolation: true,
    prediction: false // Not needed for spectators
  },

  memory: {
    preloadNextPlayer: true, // Smooth transitions
    unloadDistantPlayers: true
  }
};
```

---

## Squad Mode Specifics

### Teammate Priority

```typescript
const SQUAD_SPECTATOR_MODE = {
  autoSpectate: {
    priority: 'Random alive teammate',
    fallback: 'Any alive player if all teammates dead'
  },

  teammateIndicators: {
    highlight: 'Green border around teammate controls',
    icons: 'Show teammate icons on minimap',
    quickSwitch: 'Tap teammate icon to spectate them'
  },

  reviveNotification: {
    showWhen: 'Teammate has revive ability/item',
    message: 'Your teammate may be able to revive you!',
    dismiss: 'After 5 seconds or on tap'
  },

  voiceChat: {
    canHearTeam: true,
    canSpeak: false, // Prevent ghosting
    indicator: 'Show who is speaking'
  }
};
```

---

## Accessibility

### Spectator Accessibility Features

```typescript
const SPECTATOR_ACCESSIBILITY = {
  screenReader: {
    announcePlayerSwitch: '{Player name}, {kills} kills, {health}% health',
    announceEliminations: '{Killer} eliminated {Victim}',
    announcePlayersRemaining: '{Count} players remaining'
  },

  colorblind: {
    useColorblindPalette: true,
    healthBarPatterns: 'Add texture patterns to bars'
  },

  textSize: {
    respectSystemSettings: true,
    minimumSize: 14
  },

  reducedMotion: {
    disableCameraTransitions: 'Instant cut instead',
    disablePulsingAnimations: true
  }
};
```

---

## Analytics Events

```typescript
const SPECTATOR_ANALYTICS = {
  events: [
    {
      name: 'spectator_mode_entered',
      properties: {
        entrySource: 'elimination | voluntary',
        playersRemaining: 'number',
        squadMode: 'boolean',
        teammatesAlive: 'number'
      }
    },
    {
      name: 'spectator_player_switched',
      properties: {
        method: 'button | swipe | minimap',
        timeWatching: 'seconds with previous player'
      }
    },
    {
      name: 'spectator_free_camera_used',
      properties: {
        duration: 'seconds in free cam',
        timesToggled: 'number'
      }
    },
    {
      name: 'spectator_chat_message_sent',
      properties: {
        messageType: 'custom | quick',
        characterCount: 'number'
      }
    },
    {
      name: 'spectator_mode_exited',
      properties: {
        exitMethod: 'leave_button | match_ended',
        timeSpectating: 'total seconds',
        playersWatched: 'count'
      }
    }
  ],

  metrics: [
    'Average spectate duration',
    'Spectate-to-leave rate',
    'Most popular spectate targets (kill leaders)',
    'Free camera usage rate',
    'Chat engagement rate'
  ]
};
```

---

## Error States

```typescript
const SPECTATOR_ERROR_HANDLING = {
  playerDisconnected: {
    behavior: 'Auto-switch to next player',
    message: 'Player disconnected, switching...',
    duration: 2000
  },

  noPlayersToSpectate: {
    behavior: 'Enable free camera only',
    message: 'No players available to spectate',
    showMatchEndingSoon: true
  },

  networkIssue: {
    behavior: 'Show reconnecting indicator',
    timeout: 10000,
    onTimeout: 'Offer to leave match'
  },

  matchEnded: {
    behavior: 'Show winner announcement',
    duration: 5000,
    thenShow: 'Match summary screen'
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial spectator UI specification |
