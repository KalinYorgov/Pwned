# Reconnection System Specification

## Overview

The Reconnection System enables players to rejoin ongoing matches after unexpected disconnections, which are common on mobile devices due to phone calls, app switching, network drops, and other interruptions. This system preserves match state and provides a seamless return-to-game experience.

## Reconnection Architecture

```typescript
interface ReconnectionState {
  matchId: string;
  playerId: string;

  disconnection: {
    disconnectedAt: Date;
    reason: DisconnectReason;
    gracePeriodEnds: Date;
    reconnectAttempts: number;
  };

  preservedState: PlayerMatchState;

  status: ReconnectionStatus;
}

interface PlayerMatchState {
  // Position and movement
  position: Vector3;
  rotation: Quaternion;
  velocity: Vector3;

  // Combat state
  health: number;
  maxHealth: number;
  shield: number;
  isAlive: boolean;

  // Abilities
  abilities: AbilityState[];
  cooldowns: Map<string, number>;

  // Inventory
  inventory: InventoryState;
  equippedItems: EquippedItems;

  // Match progress
  eliminations: number;
  assists: number;
  damageDealt: number;
  placement: number | null;

  // Team state
  teamId: string | null;
  teammates: string[];
}

type DisconnectReason =
  | 'networkLoss'
  | 'appClosed'
  | 'appBackgrounded'
  | 'phoneCall'
  | 'serverKick'
  | 'clientCrash'
  | 'timeout'
  | 'unknown';

type ReconnectionStatus =
  | 'disconnected'
  | 'gracePeriod'
  | 'reconnecting'
  | 'restoring'
  | 'reconnected'
  | 'expired'
  | 'eliminated';
```

## Grace Period Configuration

```typescript
const GRACE_PERIOD_CONFIG = {
  duration: {
    default: 60, // seconds
    rankedMode: 45, // Shorter for competitive
    casualMode: 90, // More lenient for casual
    ltmModes: 60 // Mode-specific
  },

  countdown: {
    startImmediately: true,
    pauseConditions: [], // Never pause
    extendConditions: [] // No extensions
  },

  limits: {
    maxReconnectsPerMatch: 3,
    minTimeBetweenDisconnects: 30, // seconds
    totalGraceTimePerMatch: 180 // Max 3 minutes total
  },

  expiration: {
    onExpire: 'eliminatePlayer',
    notifyTeammates: true,
    preserveStats: true, // Keep elimination count, etc.
    dropLoot: true
  }
};
```

## Disconnect Detection

```typescript
const DISCONNECT_DETECTION = {
  methods: {
    heartbeat: {
      interval: 1000, // ms
      missedThreshold: 3, // 3 missed = disconnect
      protocol: 'UDP'
    },

    tcpDisconnect: {
      detectImmediately: true,
      gracefulClose: 'intentional',
      connectionReset: 'unintentional'
    },

    applicationState: {
      appBackgrounded: {
        detect: true,
        graceBeforeDisconnect: 5, // seconds in background
        allowBriefSwitch: true
      },
      appClosed: {
        detect: 'onHeartbeatLoss',
        immediate: false
      }
    }
  },

  classification: {
    intentional: {
      indicators: ['quitButton', 'menuExit', 'gracefulClose'],
      allowReconnect: false,
      eliminateImmediately: true
    },
    unintentional: {
      indicators: ['heartbeatLoss', 'connectionReset', 'appKilled'],
      allowReconnect: true,
      startGracePeriod: true
    }
  },

  serverSideHandling: {
    onDisconnect: [
      'markPlayerDisconnected',
      'startGracePeriodTimer',
      'notifyTeammates',
      'makeCharacterInvulnerable',
      'hideCharacterFromOthers',
      'preserveMatchState'
    ]
  }
};
```

## Player Character During Disconnect

```typescript
const DISCONNECTED_PLAYER_HANDLING = {
  character: {
    visibility: 'hidden', // Other players can't see
    collision: 'disabled', // No collision
    invulnerable: true, // Can't take damage
    position: 'preserved', // Stay at last position

    // Alternative: visible but invulnerable with indicator
    alternativeMode: {
      visibility: 'visible',
      indicator: 'disconnectedIcon',
      opacity: 0.5,
      invulnerable: true
    }
  },

  zoneHandling: {
    inZone: 'noZoneDamage',
    outsideZone: 'noZoneDamage', // Grace from zone too
    zoneMovement: 'ignored' // Don't push character
  },

  lootHandling: {
    dropOnDisconnect: false,
    dropOnExpire: true,
    preserveInventory: true
  },

  teamBehavior: {
    canBeRevived: false, // Already "safe"
    countAsAlive: true, // Team not eliminated
    markerVisible: 'toTeammatesOnly'
  },

  exploitPrevention: {
    disconnectToAvoidDeath: {
      prevention: 'damageQueueing',
      description: 'Damage received just before disconnect is queued',
      applyOnReconnect: true
    },

    disconnectInCombat: {
      detection: 'recentDamageReceived',
      window: 5, // seconds
      handling: 'remainVulnerableFor5Seconds'
    }
  }
};
```

## Reconnection Flow

```typescript
const RECONNECTION_FLOW = {
  clientSide: {
    onAppLaunch: {
      checkForActiveMatch: true,
      endpoint: '/api/player/active-match',
      timing: 'immediately'
    },

    reconnectPrompt: {
      show: true,
      message: 'You have an active match. Reconnect?',
      options: ['Reconnect', 'Abandon'],
      timeout: 15, // seconds
      defaultAction: 'reconnect',
      showTimeRemaining: true
    },

    reconnectProcess: [
      'showReconnectingScreen',
      'establishConnection',
      'authenticatePlayer',
      'requestStateRestore',
      'downloadMatchState',
      'initializeLocalState',
      'resumeGameplay'
    ]
  },

  serverSide: {
    onReconnectRequest: [
      'validatePlayer',
      'validateMatchActive',
      'validateGracePeriodActive',
      'prepareStateSnapshot',
      'assignToGameServer',
      'sendStateToClient',
      'restorePlayerToMatch',
      'notifyOtherPlayers'
    ],

    stateRestoration: {
      fullSnapshot: true,
      incrementalUpdate: false,
      compressionEnabled: true,
      maxPayloadSize: '1MB'
    }
  },

  timing: {
    targetReconnectTime: 5, // seconds
    maxReconnectTime: 15, // seconds
    connectionTimeout: 10, // seconds
    stateDownloadTimeout: 5 // seconds
  }
};
```

## State Restoration

```typescript
const STATE_RESTORATION = {
  playerState: {
    restore: [
      'position',
      'rotation',
      'health',
      'shield',
      'abilities',
      'cooldowns',
      'inventory',
      'equippedItems',
      'matchStats',
      'teamAssignment'
    ],

    recalculate: [
      'nearbyPlayers',
      'visibleEntities',
      'audioState'
    ],

    reset: [
      'inputBuffer',
      'predictionState',
      'networkBuffers'
    ]
  },

  worldState: {
    sync: [
      'currentZoneState',
      'remainingPlayers',
      'activeSupplyDrops',
      'environmentState'
    ],

    streamOnDemand: [
      'otherPlayerPositions',
      'lootPositions',
      'projectileState'
    ]
  },

  accuracy: {
    positionTolerance: 0.1, // meters
    healthExact: true,
    inventoryExact: true,
    cooldownsRecalculated: true // Based on time elapsed
  },

  verification: {
    clientAcknowledge: true,
    stateChecksum: true,
    mismatchHandling: 'serverAuthoritative'
  }
};
```

## Reconnect UI/UX

```typescript
const RECONNECT_UX = {
  reconnectingScreen: {
    layout: 'fullScreen',
    elements: [
      'reconnectingMessage',
      'loadingSpinner',
      'progressBar',
      'timeRemaining',
      'cancelButton'
    ],

    messages: {
      connecting: 'Reconnecting to match...',
      downloading: 'Downloading match state...',
      restoring: 'Restoring your game...',
      almostReady: 'Almost there...'
    },

    progressStages: [
      { stage: 'connecting', progress: 25 },
      { stage: 'authenticating', progress: 40 },
      { stage: 'downloading', progress: 70 },
      { stage: 'restoring', progress: 90 },
      { stage: 'complete', progress: 100 }
    ]
  },

  reconnectedTransition: {
    countdown: 3, // seconds before gameplay resumes
    message: 'Resuming in {countdown}...',
    showMinimap: true,
    showZoneWarning: true,
    allowLooking: true,
    allowMoving: false // Brief orientation period
  },

  failedReconnect: {
    message: 'Unable to reconnect',
    reasons: {
      matchEnded: 'The match has ended',
      gracePeriodExpired: 'Reconnection window expired',
      serverError: 'Server error occurred',
      networkError: 'Network connection failed'
    },
    returnTo: 'mainMenu',
    showMatchResult: true // If match ended
  }
};
```

## Teammate Notifications

```typescript
const TEAMMATE_NOTIFICATIONS = {
  onDisconnect: {
    notification: {
      message: '{playerName} disconnected',
      icon: 'disconnectIcon',
      sound: 'teammateDisconnect',
      duration: 3 // seconds
    },

    playerMarker: {
      show: true,
      icon: 'disconnectedMarker',
      color: 'gray',
      pulsingAnimation: true,
      showTimer: true // Grace period remaining
    },

    voiceChat: {
      announcement: false, // Just UI notification
      muteDisconnected: true
    }
  },

  onReconnect: {
    notification: {
      message: '{playerName} reconnected',
      icon: 'reconnectIcon',
      sound: 'teammateReconnect',
      duration: 3 // seconds
    },

    playerMarker: {
      restore: 'normal',
      transitionAnimation: true
    }
  },

  onExpire: {
    notification: {
      message: '{playerName} has been eliminated (disconnect timeout)',
      icon: 'eliminatedIcon',
      sound: 'teammateEliminated',
      duration: 5 // seconds
    },

    playerMarker: {
      show: false,
      remove: true
    }
  }
};
```

## Multiple Reconnects Handling

```typescript
const MULTIPLE_RECONNECTS = {
  limits: {
    maxPerMatch: 3,
    cooldownBetween: 30, // seconds after reconnect before another grace
    totalGraceTime: 180 // seconds cumulative
  },

  tracking: {
    perPlayer: {
      reconnectCount: 'number',
      totalGraceTimeUsed: 'seconds',
      lastDisconnectTime: 'timestamp'
    }
  },

  escalation: {
    firstDisconnect: {
      gracePeriod: 60,
      handling: 'normal'
    },
    secondDisconnect: {
      gracePeriod: 45,
      handling: 'normal',
      warning: 'One reconnect remaining'
    },
    thirdDisconnect: {
      gracePeriod: 30,
      handling: 'normal',
      warning: 'Final reconnect'
    },
    fourthDisconnect: {
      gracePeriod: 0,
      handling: 'immediateElimination',
      message: 'Reconnect limit reached'
    }
  },

  abusePrevention: {
    rapidDisconnects: {
      detection: '3 disconnects in 5 minutes',
      action: 'reduceGracePeriod',
      warning: 'Frequent disconnections detected'
    },

    patternDetection: {
      disconnectBeforeDeath: 'queueDamage',
      disconnectInFight: 'remainVulnerable'
    }
  }
};
```

## Cross-App-Restart Support

```typescript
const CROSS_RESTART_SUPPORT = {
  persistence: {
    storage: 'secureLocalStorage',
    data: {
      matchId: true,
      playerId: true,
      serverAddress: true,
      authToken: true, // Short-lived reconnect token
      lastKnownState: false // Too large, fetch from server
    },
    encryption: true,
    expiry: 'matchDuration + 5 minutes'
  },

  onAppStart: {
    checkPersisted: true,
    validateWithServer: true,
    clearIfInvalid: true
  },

  tokenRefresh: {
    reconnectToken: {
      duration: '10 minutes',
      refreshable: true,
      singleUse: false
    }
  },

  cleanup: {
    onMatchEnd: 'clearPersistedData',
    onSuccessfulReconnect: 'keepData',
    onAbandon: 'clearPersistedData',
    onExpire: 'clearPersistedData'
  }
};
```

## Exploit Prevention

```typescript
const EXPLOIT_PREVENTION = {
  disconnectToAvoidDeath: {
    detection: {
      recentDamage: 'last 3 seconds',
      lowHealth: 'below 20%',
      inCombat: 'last 5 seconds'
    },

    prevention: {
      damageQueue: {
        enabled: true,
        queueDamage: true,
        applyOnReconnect: true,
        maxQueued: '100% of max health'
      },

      vulnerabilityWindow: {
        enabled: true,
        duration: 5, // seconds after reconnect
        condition: 'wasInCombat'
      }
    }
  },

  intentionalDisconnect: {
    detection: {
      patternAnalysis: true,
      frequencyThreshold: '3 per match',
      timingAnalysis: true // Always before death?
    },

    consequences: {
      warning: 'first offense',
      reducedGrace: 'second offense',
      noGrace: 'third offense',
      tempBan: 'repeat offender'
    }
  },

  networkManipulation: {
    detection: {
      artificialLatency: true,
      packetDropping: true
    },

    handling: {
      disconnect: 'treatAsUnintentional',
      flag: 'forReview',
      immediateAction: false
    }
  },

  serverValidation: {
    stateIntegrity: true,
    positionValidation: true,
    inventoryValidation: true,
    rejectInvalidState: true
  }
};
```

## Network Handling

```typescript
const NETWORK_HANDLING = {
  reconnection: {
    protocol: 'TCP', // Reliable for reconnection handshake
    fallback: 'WebSocket',
    retries: 3,
    backoff: 'exponential',
    initialDelay: 1000, // ms
    maxDelay: 5000 // ms
  },

  stateTransfer: {
    compression: 'gzip',
    chunking: true,
    chunkSize: '64KB',
    reliability: 'guaranteed'
  },

  networkConditions: {
    poorConnection: {
      detection: 'highLatencyOrPacketLoss',
      handling: 'continueAttempt',
      timeout: 'extended'
    },

    connectionSwitch: {
      wifiToMobile: 'seamlessHandoff',
      mobileToWifi: 'seamlessHandoff',
      handling: 'ipChangeSupported'
    }
  },

  mobileOptimizations: {
    backgroundKeepalive: false, // iOS/Android restrictions
    quickResume: true,
    minimalHandshake: true,
    compressedState: true
  }
};
```

## Analytics and Monitoring

```typescript
const RECONNECTION_ANALYTICS = {
  metrics: {
    disconnectRate: {
      definition: 'disconnects / totalMatchParticipations',
      breakdown: ['byReason', 'byRegion', 'byDevice']
    },

    reconnectSuccessRate: {
      definition: 'successfulReconnects / totalDisconnects',
      target: '>95%'
    },

    averageReconnectTime: {
      definition: 'avgTimeToReconnect',
      target: '<5 seconds'
    },

    gracePeriodUtilization: {
      definition: 'avgGraceTimeUsed / gracePeriodDuration',
      insight: 'howQuicklyPlayersReconnect'
    },

    expiredReconnects: {
      definition: 'expiredGracePeriods / totalDisconnects',
      target: '<10%'
    }
  },

  events: [
    'player_disconnected',
    'grace_period_started',
    'reconnect_attempted',
    'reconnect_succeeded',
    'reconnect_failed',
    'grace_period_expired',
    'state_restored'
  ],

  alerts: {
    highDisconnectRate: {
      threshold: '>5% in 15 minutes',
      action: 'investigate'
    },
    lowReconnectSuccess: {
      threshold: '<90%',
      action: 'alert_oncall'
    }
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  services: {
    reconnectionService: {
      responsibility: 'Track disconnects, manage grace periods',
      stateful: true,
      storage: 'redis'
    },

    stateSnapshotService: {
      responsibility: 'Capture and store player state',
      storage: 'memory + redis backup'
    },

    matchPersistence: {
      responsibility: 'Track active match associations',
      storage: 'redis'
    }
  },

  apiEndpoints: {
    checkActiveMatch: 'GET /api/player/active-match',
    requestReconnect: 'POST /api/match/{matchId}/reconnect',
    getMatchState: 'GET /api/match/{matchId}/state/{playerId}',
    acknowledgeRestore: 'POST /api/match/{matchId}/restore-ack'
  },

  gameServerIntegration: {
    onDisconnect: 'notifyReconnectionService',
    onGracePeriodExpire: 'eliminatePlayer',
    onReconnect: 'restorePlayer',
    stateSnapshot: 'periodicAndOnDisconnect'
  },

  performance: {
    disconnectDetection: '<1 second',
    reconnectHandshake: '<2 seconds',
    stateTransfer: '<3 seconds',
    totalReconnectTime: '<5 seconds'
  },

  reliability: {
    stateBackup: 'redundant',
    failoverSupport: true,
    dataConsistency: 'serverAuthoritative'
  }
};
```
