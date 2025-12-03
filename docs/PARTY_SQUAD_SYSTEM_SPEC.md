# Party/Squad System Specification

## Overview

The Party/Squad System enables players to group together before queuing for matches, providing seamless team formation, communication, and coordinated matchmaking across all game modes and platforms.

## Party Architecture

```typescript
interface Party {
  partyId: string;
  createdAt: Date;
  updatedAt: Date;

  // Party state
  state: PartyState;
  gameMode: GameMode | null;

  // Members
  leader: PartyMember;
  members: PartyMember[];
  maxSize: number;

  // Settings
  settings: PartySettings;

  // Voice chat
  voiceChannel: VoiceChannel;

  // Matchmaking
  matchmaking: MatchmakingState | null;
}

interface PartyMember {
  playerId: string;
  displayName: string;
  platform: Platform;
  joinedAt: Date;

  // Status
  isLeader: boolean;
  isReady: boolean;
  status: MemberStatus;

  // Player info
  level: number;
  rank?: RankInfo;
  equippedCosmetics: CosmeticLoadout;

  // Voice
  voiceState: VoiceState;
}

type PartyState =
  | 'forming'        // Party created, accepting members
  | 'ready'          // All members ready, can queue
  | 'queuing'        // In matchmaking queue
  | 'matchFound'     // Match found, confirming
  | 'inMatch'        // Currently in a match
  | 'postMatch';     // Match ended, party persists

type MemberStatus =
  | 'connected'
  | 'away'
  | 'inMenu'
  | 'inMatch'
  | 'disconnected';

type Platform = 'iOS' | 'Android';
```

## Party Creation

```typescript
const PARTY_CREATION = {
  trigger: {
    explicit: 'createPartyButton',
    implicit: 'onInviteSent', // Auto-create if not in party
    autoCreate: false // Don't auto-create on login
  },

  initialization: {
    creator: 'becomesLeader',
    defaultMaxSize: 4, // Squad size
    defaultMode: null, // Set when queuing
    defaultSettings: {
      voiceChat: true,
      allowInvites: 'leader', // Only leader can invite
      joinability: 'invite_only'
    }
  },

  response: {
    success: {
      partyId: 'returned',
      memberList: 'returned',
      voiceChannel: 'created'
    },
    failure: {
      alreadyInParty: 'mustLeaveFirst',
      serverError: 'retryWithBackoff'
    }
  },

  limits: {
    maxPartiesPerPlayer: 1, // Can only be in one party
    partyIdleTimeout: '2 hours', // Disband if no activity
    reconnectWindow: '5 minutes' // Leader disconnect grace
  }
};
```

## Invite System

```typescript
interface PartyInvite {
  inviteId: string;
  partyId: string;

  sender: {
    playerId: string;
    displayName: string;
  };

  recipient: {
    playerId: string;
    method: InviteMethod;
  };

  status: InviteStatus;
  createdAt: Date;
  expiresAt: Date;
}

type InviteMethod =
  | 'friendList'     // Direct from friends
  | 'username'       // Search by username
  | 'shareLink'      // Shareable invite link
  | 'recentPlayers'  // From recent matches
  | 'deepLink';      // External app link

type InviteStatus =
  | 'pending'
  | 'accepted'
  | 'declined'
  | 'expired'
  | 'cancelled'
  | 'partyFull';

const INVITE_SYSTEM = {
  methods: {
    friendList: {
      enabled: true,
      showOnlineStatus: true,
      showCurrentActivity: true,
      oneClickInvite: true
    },

    username: {
      enabled: true,
      searchMinLength: 3,
      exactMatchRequired: false,
      showResults: 10,
      rateLimited: true
    },

    shareLink: {
      enabled: true,
      format: 'plunderstorm://party/{partyId}',
      expiresIn: '24 hours',
      maxUses: 10,
      regeneratable: true
    },

    recentPlayers: {
      enabled: true,
      lookback: '24 hours',
      maxShown: 20,
      excludeReported: true
    },

    deepLink: {
      enabled: true,
      platforms: ['iOS', 'Android'],
      fallbackToStore: true
    }
  },

  settings: {
    timeout: '2 minutes',
    maxPendingInvites: 10, // Per party
    cooldownBetweenInvites: '10 seconds', // To same player
    blockListRespected: true
  },

  notifications: {
    inGame: {
      display: 'popup',
      sound: 'inviteReceived',
      haptic: 'light',
      duration: '30 seconds',
      position: 'topCenter'
    },

    push: {
      enabled: true,
      ifNotInGame: true,
      message: '{playerName} invited you to their party!'
    }
  }
};
```

## Invite Notifications

```typescript
const INVITE_NOTIFICATIONS = {
  display: {
    layout: 'compactCard',
    elements: [
      'senderAvatar',
      'senderName',
      'senderLevel',
      'partySize', // e.g., "1/4 players"
      'acceptButton',
      'declineButton',
      'timer'
    ],

    animation: {
      slideIn: 'fromTop',
      slideOut: 'toTop',
      duration: '300ms'
    }
  },

  actions: {
    accept: {
      button: 'Accept',
      style: 'primary',
      action: 'joinParty',
      haptic: 'medium'
    },

    decline: {
      button: 'Decline',
      style: 'secondary',
      action: 'declineInvite',
      haptic: 'light'
    },

    ignore: {
      onTimeout: 'autoDecline',
      notify: false
    }
  },

  queueBehavior: {
    multipleInvites: 'stackVertically',
    maxVisible: 3,
    olderInvitesQueue: true
  },

  contextualDisplay: {
    inMenu: 'fullNotification',
    inMatch: 'minimizedBanner',
    inQueue: 'fullNotification'
  }
};
```

## Accept/Decline Flow

```typescript
const ACCEPT_DECLINE_FLOW = {
  accept: {
    process: [
      'validateInviteNotExpired',
      'validatePartyNotFull',
      'leaveCurrentPartyIfAny',
      'joinParty',
      'notifyPartyMembers',
      'initializeVoiceChat',
      'syncPartyState'
    ],

    success: {
      transition: 'navigateToPartyLobby',
      notification: '{playerName} joined the party',
      sound: 'memberJoined'
    },

    failure: {
      inviteExpired: { message: 'Invite has expired', action: 'dismiss' },
      partyFull: { message: 'Party is full', action: 'dismiss' },
      partyDisbanded: { message: 'Party no longer exists', action: 'dismiss' },
      error: { message: 'Failed to join party', action: 'retry' }
    }
  },

  decline: {
    process: [
      'markInviteDeclined',
      'notifySender',
      'dismissNotification'
    ],

    senderNotification: {
      show: true,
      message: '{playerName} declined your invite',
      duration: '3 seconds'
    },

    cooldown: {
      reInviteWait: '30 seconds',
      maxDeclinesThenBlock: 3 // Block further invites for 1 hour
    }
  },

  timeout: {
    duration: '2 minutes',
    behavior: 'autoDecline',
    notifySender: true,
    message: 'Invite expired'
  }
};
```

## Party Member Management

```typescript
const MEMBER_MANAGEMENT = {
  kick: {
    permission: 'leaderOnly',
    confirmation: true,
    cooldown: '30 seconds', // Before can reinvite

    process: [
      'validateLeaderPermission',
      'removeFromParty',
      'disconnectFromVoice',
      'notifyKickedPlayer',
      'notifyRemainingMembers'
    ],

    kickedPlayerExperience: {
      notification: 'You were removed from the party',
      returnTo: 'mainMenu',
      canRejoin: 'ifReinvited'
    }
  },

  leave: {
    available: 'allMembers',
    confirmation: false,
    inMatchBehavior: 'leaveAfterMatch', // Don't interrupt match

    process: [
      'removeFromParty',
      'disconnectFromVoice',
      'handleLeadershipTransfer',
      'notifyRemainingMembers'
    ],

    leaverExperience: {
      returnTo: 'mainMenu',
      partyCleared: true
    }
  },

  ready: {
    toggle: true,
    showStatus: true,
    allReadyRequired: false, // Leader can queue without all ready
    readyIndicator: 'checkmark'
  }
};
```

## Leadership Transfer

```typescript
const LEADERSHIP_TRANSFER = {
  triggers: {
    leaderLeaves: 'autoTransfer',
    leaderDisconnects: 'afterGracePeriod',
    leaderRequests: 'manualTransfer',
    leaderKicked: 'n/a' // Leaders can't be kicked
  },

  transferOrder: {
    priority: [
      'longestInParty',
      'highestLevel',
      'alphabetical'
    ],
    explicit: 'leaderCanSelectSuccessor'
  },

  gracePeriod: {
    duration: '2 minutes',
    forDisconnect: true,
    showCountdown: true,
    reconnectRestores: true
  },

  manualTransfer: {
    uiLocation: 'memberContextMenu',
    confirmation: true,
    instantEffect: true
  },

  newLeaderNotification: {
    message: 'You are now the party leader',
    highlight: true,
    showControls: true
  },

  disbandIfEmpty: {
    trigger: 'lastMemberLeaves',
    immediate: true,
    cleanup: 'voiceChannelDeleted'
  }
};
```

## Party Size and Mode Restrictions

```typescript
const PARTY_SIZE_RESTRICTIONS = {
  modes: {
    solo: {
      maxPartySize: 1, // No party in solo
      partyBehavior: 'disallowed',
      message: 'Solo mode does not support parties'
    },

    duo: {
      maxPartySize: 2,
      partyBehavior: 'required', // Must have 2 for duo
      fillBehavior: 'waitForSecond', // Or fill with random
      message: 'Duo mode requires 2 players'
    },

    squad: {
      maxPartySize: 4,
      partyBehavior: 'optional', // Can queue with 1-4
      fillBehavior: 'fillWithRandoms',
      message: 'Squad mode allows 1-4 players'
    },

    ranked: {
      soloRanked: { maxPartySize: 1 },
      duoRanked: { maxPartySize: 2 },
      squadRanked: { maxPartySize: 4 },
      restrictions: 'rankDifferenceLimited' // Can't party with very different ranks
    }
  },

  enforcement: {
    onModeSelect: 'validatePartySize',
    overSize: 'promptToKick',
    underSize: 'offerFillOption'
  },

  rankRestrictions: {
    maxRankDifference: 2, // Tiers (e.g., Gold can't queue with Diamond)
    enabled: 'rankedOnly',
    message: 'Party members are too far apart in rank'
  }
};
```

## Party Voice Chat

```typescript
const PARTY_VOICE_CHAT = {
  lobby: {
    autoConnect: true,
    defaultState: 'unmuted',
    codec: 'opus',
    quality: 'adaptive'
  },

  channelManagement: {
    createOnPartyCreate: true,
    deleteOnPartyDisband: true,
    persistent: true // Stays through matchmaking
  },

  controls: {
    muteSelf: {
      button: 'microphoneToggle',
      shortcut: 'doubleTap',
      indicator: 'micIcon'
    },

    muteOthers: {
      perMember: true,
      location: 'memberContextMenu',
      persistent: 'forSession'
    },

    adjustVolume: {
      master: true,
      perMember: true,
      range: '0-200%'
    },

    pushToTalk: {
      option: true,
      default: false,
      button: 'holdToSpeak'
    }
  },

  inMatch: {
    teamVoice: 'useMatchVoice', // Switch to match team voice
    partyOverlay: 'showPartyIndicators',
    prioritySpeaker: false
  },

  quality: {
    sampleRate: 48000,
    bitrate: 'adaptive', // 24-64 kbps
    echoCancellation: true,
    noiseSuppression: true,
    automaticGainControl: true
  },

  fallback: {
    highLatency: 'reduceBitrate',
    connectionLost: 'reconnectAutomatically',
    serverIssue: 'showOfflineIndicator'
  }
};
```

## Cross-Platform Support

```typescript
const CROSS_PLATFORM = {
  supported: true,
  platforms: ['iOS', 'Android'],

  partyFormation: {
    crossPlatformInvites: true,
    showPlatformIcon: true,
    noRestrictions: true
  },

  matchmaking: {
    crossPlayDefault: true,
    optOutOption: false, // Always cross-play enabled
    balancing: 'skillBased' // Not platform-based
  },

  voiceChat: {
    crossPlatformVoice: true,
    sameCodec: true,
    qualityMatching: true
  },

  display: {
    platformIcon: {
      show: true,
      location: 'nextToName',
      style: 'subtle'
    }
  },

  technicalConsiderations: {
    accountLinking: 'singleAccountCrossDevice',
    progressSync: 'cloudBased',
    friendsCrossPlaftorm: true
  }
};
```

## Party Persistence

```typescript
const PARTY_PERSISTENCE = {
  acrossMatches: {
    persist: true,
    autoRegroup: true,
    postMatchLobby: true
  },

  postMatch: {
    returnToPartyLobby: true,
    showMatchResults: true,
    readyStateReset: true,
    queueAgainOption: 'prominent'
  },

  disconnection: {
    memberDisconnect: {
      gracePeriod: '2 minutes',
      placeholder: 'showAsDisconnected',
      autoKickAfter: '5 minutes',
      reconnectRestores: true
    },

    leaderDisconnect: {
      gracePeriod: '2 minutes',
      leadershipTransfer: 'afterGracePeriod',
      reconnectRestores: 'ifStillLeader'
    }
  },

  appBackground: {
    keepParty: true,
    timeout: '30 minutes',
    notifyOnActivity: true
  },

  appClose: {
    keepParty: '5 minutes', // Brief window for app restart
    cleanupAfter: true
  }
};
```

## Party Lobby UI

```typescript
const PARTY_LOBBY_UI = {
  layout: {
    style: 'memberCards',
    maxVisible: 4, // Squad size
    emptySlots: 'showInviteButton'
  },

  memberCard: {
    elements: [
      'avatar',
      'displayName',
      'level',
      'rank',
      'platformIcon',
      'readyStatus',
      'voiceIndicator',
      'leaderCrown'
    ],

    interactivity: {
      tap: 'showProfile',
      longPress: 'showContextMenu'
    }
  },

  contextMenu: {
    leader: [
      'viewProfile',
      'kickFromParty',
      'transferLeadership',
      'mutePlayer'
    ],
    member: [
      'viewProfile',
      'mutePlayer'
    ],
    self: [
      'viewProfile',
      'leaveParty'
    ]
  },

  partyActions: {
    leader: [
      'invitePlayer',
      'selectGameMode',
      'startQueue',
      'disbandParty'
    ],
    member: [
      'toggleReady',
      'leaveParty'
    ]
  },

  voiceControls: {
    location: 'bottomBar',
    elements: ['micToggle', 'speakerToggle', 'settings']
  },

  queueButton: {
    state: {
      notReady: 'Select Mode',
      ready: 'Start Queue',
      queuing: 'Cancel Queue',
      matchFound: 'Match Found!'
    },
    leaderOnly: true
  }
};
```

## Matchmaking Integration

```typescript
const MATCHMAKING_INTEGRATION = {
  queueControl: {
    initiator: 'leaderOnly',
    memberConsent: 'readyStatusOptional',
    modeSelection: 'leaderOnly'
  },

  partyQueue: {
    queueAsUnit: true,
    keepTogether: true,
    averageMMR: true, // For ranked
    priorityBoost: false // No queue priority for parties
  },

  queueState: {
    showToAllMembers: true,
    syncedDisplay: true,
    estimatedTime: true
  },

  matchFound: {
    notifyAllMembers: true,
    acceptRequired: 'leaderOnly',
    timeout: '30 seconds',
    declineReturnsToLobby: true
  },

  matchStart: {
    transitionTogether: true,
    sameTeam: true,
    voiceTransition: 'toMatchVoice'
  },

  cancelQueue: {
    leaderOnly: true,
    memberCanRequest: true, // Request leader to cancel
    instant: true
  }
};
```

## Error Handling and Edge Cases

```typescript
const ERROR_HANDLING = {
  inviteErrors: {
    playerOffline: 'showOfflineMessage',
    playerInMatch: 'showInMatchMessage',
    playerBlocked: 'silentlyFail',
    partyFull: 'showFullMessage',
    inviteLimit: 'showLimitMessage'
  },

  joinErrors: {
    partyDisbanded: 'returnToMenu',
    partyFull: 'showFullError',
    connectionFailed: 'retryWithBackoff'
  },

  matchmakingErrors: {
    queueFailed: 'showErrorAndRetry',
    memberDisconnected: 'cancelQueueAndNotify',
    modeUnavailable: 'showModeUnavailable'
  },

  voiceErrors: {
    connectionFailed: 'showVoiceOffline',
    micPermissionDenied: 'showPermissionPrompt',
    echoCancellationFailed: 'fallbackToBasic'
  },

  stateReconciliation: {
    onReconnect: 'syncFromServer',
    conflictResolution: 'serverAuthoritative',
    corruptedState: 'recreateParty'
  }
};
```

## Analytics and Metrics

```typescript
const PARTY_ANALYTICS = {
  metrics: {
    partyFormation: {
      partiesCreatedPerDay: true,
      averagePartySize: true,
      inviteAcceptRate: true,
      inviteDeclineRate: true,
      inviteTimeoutRate: true
    },

    partyUsage: {
      averagePartyDuration: true,
      matchesPerParty: true,
      partyRetentionRate: true,
      crossPlatformParties: true
    },

    voiceChat: {
      voiceUsageRate: true,
      averageVoiceDuration: true,
      muteRate: true
    },

    issues: {
      disconnectionRate: true,
      inviteFailureRate: true,
      voiceIssuesRate: true
    }
  },

  events: [
    'party_created',
    'party_invite_sent',
    'party_invite_accepted',
    'party_invite_declined',
    'party_member_joined',
    'party_member_left',
    'party_member_kicked',
    'party_leadership_transferred',
    'party_queue_started',
    'party_match_found',
    'party_disbanded',
    'party_voice_joined',
    'party_voice_left'
  ]
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  architecture: {
    partyService: 'dedicated microservice',
    realTimeSync: 'websocket',
    voiceBackend: 'agora / vivox / custom',
    stateStorage: 'redis (ephemeral) + mongodb (history)'
  },

  apiEndpoints: {
    createParty: 'POST /api/party',
    getParty: 'GET /api/party/{partyId}',
    invitePlayer: 'POST /api/party/{partyId}/invite',
    acceptInvite: 'POST /api/party/invite/{inviteId}/accept',
    declineInvite: 'POST /api/party/invite/{inviteId}/decline',
    leaveParty: 'POST /api/party/{partyId}/leave',
    kickMember: 'POST /api/party/{partyId}/kick/{playerId}',
    transferLeadership: 'POST /api/party/{partyId}/transfer/{playerId}',
    updateSettings: 'PATCH /api/party/{partyId}/settings'
  },

  realTimeEvents: {
    memberJoined: 'party.member.joined',
    memberLeft: 'party.member.left',
    memberKicked: 'party.member.kicked',
    leaderChanged: 'party.leader.changed',
    settingsChanged: 'party.settings.changed',
    matchmakingUpdate: 'party.matchmaking.update',
    voiceStateChanged: 'party.voice.changed'
  },

  performance: {
    inviteLatency: '<100ms',
    joinLatency: '<200ms',
    stateSync: '<50ms',
    voiceLatency: '<100ms'
  },

  scalability: {
    partiesPerServer: 10000,
    horizontalScaling: true,
    loadBalancing: 'consistentHashing'
  }
};
```
