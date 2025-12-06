# Spectator Mode Specification

## Document Information
- **Task ID:** GAME-006
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies the spectator mode gameplay system for Plunderstorm Mobile, allowing eliminated players to watch remaining players in the match. The system handles camera controls, player cycling, network optimization, and anti-cheat measures to prevent information leaking.

---

## Spectator Mode Entry

### Elimination to Spectator Transition

```typescript
const SPECTATOR_ENTRY_SYSTEM = {
  trigger: {
    event: 'player_eliminated',
    delay: 3000, // ms for death camera
    automatic: true
  },

  transitionFlow: {
    step1_deathCamera: {
      duration: 3000,
      focus: 'Killer or death location',
      playerControl: false
    },
    step2_eliminationSummary: {
      duration: 2500,
      shows: ['Killer info', 'Your stats', 'Placement'],
      skippable: true
    },
    step3_spectatorPrompt: {
      duration: 10000,
      options: ['Spectate', 'Leave Match'],
      defaultAction: 'spectate',
      timeout: 'Auto-spectate'
    },
    step4_spectatorMode: {
      initialTarget: 'Determined by mode rules',
      cameraMode: 'follow'
    }
  },

  teamModeVariant: {
    aliveTeammates: {
      skipPrompt: true,
      autoSpectate: 'Random alive teammate',
      message: 'Spectating teammate...'
    },
    allTeammatesDead: {
      behavior: 'Same as solo mode',
      canSpectateAny: true
    }
  }
};
```

### Spectator State Management

```typescript
const SPECTATOR_STATE = {
  playerState: {
    isSpectating: true,
    isAlive: false,
    spectatorId: 'unique_spectator_id',
    observedPlayerId: 'target_player_id',
    cameraMode: 'follow | free',
    joinTime: 'timestamp'
  },

  permissions: {
    canMove: false,
    canAttack: false,
    canUseAbilities: false,
    canInteract: false,
    canChat: 'spectator_only',
    canPing: false,
    canSeeAllPlayers: false, // Only observed player's perspective
    canHearVoice: 'teammates_only' // In squad mode
  },

  restrictions: {
    noWorldInteraction: true,
    noCollision: true,
    invisible: true,
    noAudio: 'Except ambient and observed player'
  }
};
```

---

## Camera System

### Follow Camera Mode

```typescript
const FOLLOW_CAMERA = {
  default: true,

  positioning: {
    distance: 8, // meters behind player
    height: 3, // meters above player
    angle: 15, // degrees down
    offset: { x: 0, y: 0, z: 0 }
  },

  behavior: {
    followSmoothing: 0.1,
    rotationSmoothing: 0.15,
    lookAhead: {
      enabled: true,
      distance: 2, // meters
      speed: 0.5
    },
    obstacleAvoidance: {
      enabled: true,
      method: 'Camera collision + zoom'
    }
  },

  spectatorControl: {
    orbitEnabled: true,
    orbitInput: 'Right side drag',
    orbitSpeed: 1.0,
    orbitLimits: {
      horizontal: 360, // Full rotation
      verticalMin: -30, // degrees
      verticalMax: 60
    },
    resetOrbit: {
      trigger: 'Release + 2 seconds',
      smooth: true
    }
  },

  combatAdjustments: {
    onCombat: {
      distance: 10, // Pull back slightly
      fov: 65 // Slightly wider
    },
    onAbilityUse: {
      brief: 'Dramatic angle if applicable',
      duration: 500
    }
  }
};
```

### Free Camera Mode

```typescript
const FREE_CAMERA = {
  activation: {
    toggle: 'Free Camera button',
    doubleTap: 'Double-tap empty screen area'
  },

  movement: {
    inputMethod: 'Virtual joystick (left)',
    axes: ['Forward/Back', 'Left/Right'],
    speed: {
      normal: 15, // units/second
      fast: 30, // With speed boost
      vertical: 10
    },
    acceleration: 0.2,
    deceleration: 0.3
  },

  rotation: {
    inputMethod: 'Drag (right side)',
    sensitivity: 1.0,
    invertY: 'Respect player settings'
  },

  altitude: {
    controls: ['Up button', 'Down button'],
    range: {
      min: 5, // meters
      max: 100 // meters
    }
  },

  boundaries: {
    withinMapBounds: true,
    softBoundary: 10, // meters from edge
    hardBoundary: 'Map edge',
    terrainCollision: false // Camera can pass through
  },

  returnToPlayer: {
    button: 'Return to Player',
    transition: 'Smooth fly to followed player',
    duration: 1000
  }
};
```

---

## Player Cycling System

### Cycle Logic

```typescript
const PLAYER_CYCLING = {
  soloMode: {
    order: 'Random initial, then sequential',
    criteria: 'All alive players',
    sorting: {
      primary: 'Kill count (descending)',
      secondary: 'Health (descending)',
      tertiary: 'Random'
    }
  },

  duoMode: {
    priority: [
      'Alive teammate (if exists)',
      'All alive players by kills'
    ],
    teammateIndicator: 'Green highlight on cycle'
  },

  squadMode: {
    priority: [
      'Alive teammates (cycle through)',
      'All alive players by kills (after team eliminated)'
    ],
    teammateIndicator: 'Green highlight + name tag'
  },

  cycling: {
    nextPlayer: {
      input: 'Next button or swipe left',
      transition: 'Smooth camera move',
      transitionDuration: 500
    },
    previousPlayer: {
      input: 'Previous button or swipe right',
      transition: 'Smooth camera move'
    },
    directSelection: {
      input: 'Tap player on minimap (if visible)',
      available: 'When player list shown'
    }
  },

  excludedFromCycle: [
    'Eliminated players',
    'Disconnected players',
    'Self (eliminated player)'
  ]
};
```

### Player List Management

```typescript
const SPECTATOR_PLAYER_LIST = {
  tracking: {
    alivePlayers: 'Real-time updated list',
    playerInfo: {
      playerId: 'string',
      displayName: 'string',
      kills: 'number',
      health: 'number',
      position: 'Vector3',
      isTeammate: 'boolean'
    }
  },

  updates: {
    onElimination: 'Remove from list',
    onDisconnect: 'Remove from list',
    frequency: 'Real-time events'
  },

  sorting: {
    method: 'Dynamic re-sort on changes',
    criteria: 'Defined by mode (solo/duo/squad)'
  }
};
```

---

## Spectator Information Display

### Observed Player Info

```typescript
const OBSERVED_PLAYER_INFO = {
  alwaysVisible: {
    playerName: {
      maxLength: 16,
      truncation: 'ellipsis'
    },
    health: {
      type: 'Health bar',
      showNumbers: true
    },
    shield: {
      type: 'Shield bar (if applicable)',
      showNumbers: true
    },
    kills: {
      format: 'Kills: {count}',
      icon: 'skull_icon'
    }
  },

  expandedInfo: {
    toggle: 'Tap info panel',
    shows: {
      abilities: 'Equipped abilities with cooldowns',
      weapons: 'Current weapon',
      items: 'Consumables count',
      damageDealt: 'Total damage this match'
    }
  },

  position: {
    location: 'Top center of screen',
    style: 'Semi-transparent panel'
  }
};
```

### Match Information

```typescript
const MATCH_INFO_DISPLAY = {
  playersRemaining: {
    format: '{count} Alive',
    icon: 'users_icon',
    position: 'Top right',
    updateAnimation: 'Flash on decrease'
  },

  matchTimer: {
    format: 'MM:SS',
    position: 'Top right, below players',
    phases: 'Show current zone phase'
  },

  killFeed: {
    visible: true,
    position: 'Top right, below timer',
    entries: 3,
    duration: 5000
  },

  minimap: {
    visible: true,
    highlightObserved: true,
    showZone: true,
    showTeammates: 'In team modes'
  }
};
```

---

## Network Architecture

### Spectator Data Streaming

```typescript
const SPECTATOR_NETWORKING = {
  dataReceived: {
    observedPlayer: {
      position: 'Full precision, 20 Hz',
      rotation: 'Compressed, 20 Hz',
      health: 'On change',
      abilities: 'On use/cooldown change',
      animations: 'State changes'
    },
    otherPlayers: {
      position: 'Reduced precision, 10 Hz',
      visibility: 'Only when in observed player view'
    },
    worldState: {
      zone: 'Phase changes',
      events: 'World events',
      eliminations: 'All eliminations'
    }
  },

  bandwidthOptimization: {
    prioritization: {
      high: 'Observed player data',
      medium: 'Nearby player data',
      low: 'Distant player data'
    },
    culling: {
      enabled: true,
      method: 'Only send visible entities'
    },
    compression: {
      enabled: true,
      method: 'Delta compression for positions'
    }
  },

  estimatedBandwidth: {
    perSpectator: '5-10 KB/s',
    scalability: 'Linear with spectator count'
  }
};
```

### Server-Side Management

```typescript
const SERVER_SPECTATOR_MANAGEMENT = {
  spectatorList: {
    tracking: 'Per-match spectator list',
    maxSpectators: 60, // Same as max players
    lateJoinSpectators: false // Cannot join as spectator
  },

  dataDistribution: {
    method: 'Server broadcasts to spectator group',
    separate: 'Spectator channel separate from player channel',
    efficiency: 'Single broadcast to all spectators observing same player'
  },

  loadBalancing: {
    manySpectators: {
      threshold: 30,
      action: 'Reduce update frequency slightly'
    },
    serverPerformance: {
      priority: 'Alive players over spectators',
      degradation: 'Graceful quality reduction for spectators'
    }
  }
};
```

---

## Anti-Cheat Measures

### Information Leak Prevention

```typescript
const ANTI_CHEAT_SPECTATOR = {
  chatIsolation: {
    spectatorChat: {
      visibleTo: 'Other spectators only',
      invisibleTo: 'All alive players',
      storage: 'Separate chat channel'
    },
    implementation: {
      server: 'Filter chat by player state',
      client: 'Separate UI for spectator chat'
    }
  },

  voiceChatIsolation: {
    squadMode: {
      spectatorCanHear: 'Alive teammates only',
      spectatorCanSpeak: false, // Prevent ghosting
      reason: 'Prevent callouts to alive teammates'
    },
    soloMode: {
      voiceChat: 'Disabled entirely'
    }
  },

  positionHiding: {
    description: 'Spectators cannot reveal enemy positions',
    measures: [
      'No ping system for spectators',
      'No way to communicate with alive players',
      'Voice muted in squad mode'
    ]
  },

  streamSniping: {
    delayOption: {
      enabled: false, // Consider for esports
      delay: 30, // seconds
      futureFeature: true
    }
  }
};
```

### Validation

```typescript
const SPECTATOR_VALIDATION = {
  serverSide: {
    stateValidation: 'Verify player is actually eliminated',
    inputRejection: 'Reject any gameplay inputs from spectators',
    positionCheck: 'Ignore spectator positions for game logic'
  },

  clientSide: {
    uiLockdown: 'Disable all gameplay UI in spectator mode',
    inputBlocking: 'Block gameplay inputs at input layer'
  },

  monitoring: {
    logSuspicious: 'Log any gameplay attempts from spectators',
    flagging: 'Flag accounts with repeated violations'
  }
};
```

---

## Spectator Count Display

### Optional Visibility

```typescript
const SPECTATOR_COUNT_DISPLAY = {
  forAlivePlayer: {
    setting: {
      name: 'Show Spectator Count',
      location: 'Settings > Privacy',
      default: true
    },
    display: {
      icon: 'eye_icon',
      format: '{count}',
      position: 'Near player info',
      visibility: 'Only when count > 0'
    },
    privacy: {
      anonymized: true, // No spectator names shown
      countOnly: true
    }
  },

  psychology: {
    consideration: 'Some players perform better/worse when watched',
    optOut: 'Allow players to hide spectator count'
  }
};
```

---

## Exit Spectator Mode

### Exit Options

```typescript
const EXIT_SPECTATOR = {
  options: {
    leaveMatch: {
      button: 'Leave Match',
      confirmation: true,
      destination: 'Main menu'
    },
    matchEnd: {
      automatic: true,
      transition: 'Victory screen or match summary'
    }
  },

  squadMode: {
    canRejoin: false, // Once eliminated, cannot rejoin
    stayForTeam: {
      encouraged: true,
      message: 'Cheer on your teammates!'
    }
  },

  rewards: {
    spectatorXP: false, // No XP for spectating
    challengeProgress: false,
    battlePassProgress: false
  }
};
```

---

## Performance Optimization

### Client Performance

```typescript
const SPECTATOR_PERFORMANCE = {
  rendering: {
    targetFPS: 60, // Same as gameplay
    qualityLevel: 'Match player settings',
    optimizations: [
      'Reduced particle density for distant effects',
      'Lower LOD for non-observed players',
      'Culling based on camera frustum'
    ]
  },

  memory: {
    unloadPlayerAssets: false, // Keep loaded for quick switching
    preloadNextPlayer: true,
    cacheRecentlyViewed: 3 // Players
  },

  cpu: {
    disablePlayerPhysics: true, // No local physics for spectator
    reducedAI: 'Client-side prediction disabled',
    inputProcessing: 'Minimal (camera only)'
  }
};
```

### Server Performance

```typescript
const SERVER_SPECTATOR_PERFORMANCE = {
  scalability: {
    maxSpectators: 60,
    performanceImpact: 'Linear with spectator count',
    target: '< 5% CPU overhead for spectator system'
  },

  optimization: {
    batchUpdates: true,
    updateCoalescing: 'Combine updates for same observed player',
    spectatorGrouping: 'Group spectators by observed player'
  },

  monitoring: {
    metrics: [
      'Spectator count per match',
      'Bandwidth per spectator',
      'Server CPU for spectator system'
    ]
  }
};
```

---

## Analytics

```typescript
const SPECTATOR_ANALYTICS = {
  events: [
    {
      name: 'spectator_mode_entered',
      properties: {
        placement: 'number',
        match_duration: 'seconds',
        game_mode: 'solo | duo | squad'
      }
    },
    {
      name: 'spectator_player_switched',
      properties: {
        method: 'button | swipe | minimap',
        watch_duration: 'seconds'
      }
    },
    {
      name: 'spectator_mode_exited',
      properties: {
        exit_reason: 'leave | match_end',
        spectate_duration: 'seconds',
        players_watched: 'number'
      }
    },
    {
      name: 'spectator_camera_mode',
      properties: {
        mode: 'follow | free',
        time_in_mode: 'seconds'
      }
    }
  ],

  metrics: [
    'Average spectate duration',
    'Spectate-to-leave rate',
    'Free camera usage rate',
    'Most spectated players (by kills)'
  ]
};
```

---

## Testing Requirements

```typescript
const SPECTATOR_TESTING = {
  functional: [
    'Spectator mode activates on elimination',
    'Camera follows observed player correctly',
    'Player cycling works in all modes',
    'Free camera respects boundaries',
    'Chat isolation is enforced',
    'Exit to lobby works correctly'
  ],

  network: [
    'Spectator receives accurate player data',
    'Bandwidth stays within limits',
    'No desync between spectator and actual game state',
    '30+ spectators supported without degradation'
  ],

  antiCheat: [
    'Spectators cannot send gameplay inputs',
    'Chat is completely isolated',
    'Voice chat restrictions enforced',
    'No information leak vectors'
  ],

  performance: [
    'Stable FPS in spectator mode',
    'Quick player switching (<500ms)',
    'No memory leaks during extended spectating'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial spectator mode specification |
