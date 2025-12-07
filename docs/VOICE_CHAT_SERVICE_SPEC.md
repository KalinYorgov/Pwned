# Voice Chat Service Specification

## Document Information
- **Task ID:** BACK-009
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the real-time voice chat service for Plunderstorm Mobile, enabling team communication during matches and party chat in lobbies. The system integrates with third-party voice providers while maintaining privacy, moderation, and performance standards.

---

## Service Provider Integration

### Provider Selection

```typescript
const VOICE_PROVIDER = {
  primary: {
    provider: 'Agora', // Or Vivox, Discord SDK
    rationale: [
      'Mobile-optimized SDK',
      'Low latency (<200ms)',
      'Built-in echo cancellation',
      'Scalable pricing model',
      'Cross-platform support'
    ]
  },

  fallback: {
    provider: 'Vivox',
    trigger: 'Primary provider outage',
    autoSwitch: true
  },

  features: {
    spatialAudio: false, // Not used for team chat
    recording: false,    // No server-side recording
    transcription: false // Not needed initially
  }
};

const SDK_INTEGRATION = {
  initialization: {
    timing: 'On app start (lazy)',
    credentials: 'Fetched from backend on demand',
    lifecycle: 'Managed with app lifecycle'
  },

  authentication: {
    method: 'Token-based',
    tokenSource: 'Backend generates provider-specific tokens',
    expiration: 3600, // 1 hour
    refresh: 'Automatic before expiry'
  },

  errorHandling: {
    connectionFailed: 'Retry with exponential backoff',
    tokenExpired: 'Refresh token automatically',
    providerDown: 'Switch to fallback or disable voice'
  }
};
```

---

## Channel Architecture

### Channel Types

```typescript
const CHANNEL_TYPES = {
  party: {
    name: 'Party Voice Channel',
    scope: 'Pre-game lobby',
    maxParticipants: 4, // Match max party size
    persistence: 'Until party disbands',
    privacy: 'Private - party members only',
    creation: 'Automatic on party formation'
  },

  team: {
    name: 'Team Voice Channel',
    scope: 'In-match communication',
    maxParticipants: 4, // Match max team size
    persistence: 'Match duration only',
    privacy: 'Private - teammates only',
    creation: 'Automatic on match start for team modes'
  },

  spectator: {
    name: 'Spectator Voice Channel',
    scope: 'Post-elimination (team modes)',
    maxParticipants: 4,
    persistence: 'Match duration',
    privacy: 'Dead teammates only',
    restriction: 'Can hear alive teammates, cannot speak to them'
  }
};

const CHANNEL_MANAGEMENT = {
  naming: {
    party: 'party_{party_id}',
    team: 'match_{match_id}_team_{team_id}',
    spectator: 'match_{match_id}_team_{team_id}_spectator'
  },

  lifecycle: {
    creation: 'Server creates channel, returns join token',
    joining: 'Client joins with token',
    leaving: 'Client leaves on disconnect/elimination/quit',
    cleanup: 'Server destroys channel when empty for 5 min'
  },

  transition: {
    partyToMatch: {
      trigger: 'Match start',
      action: 'Party members auto-joined to team channel',
      partyChannel: 'Suspended but not destroyed'
    },
    matchToParty: {
      trigger: 'Match end',
      action: 'Party members auto-rejoin party channel',
      teamChannel: 'Destroyed'
    }
  }
};
```

---

## Audio Configuration

### Quality Settings

```typescript
const AUDIO_QUALITY = {
  sampling: {
    rate: 16000,    // 16kHz minimum
    preferred: 24000, // 24kHz for higher quality
    adaptive: true   // Lower quality on poor network
  },

  codec: {
    primary: 'Opus',
    bitrate: {
      voice: 24000,  // 24kbps for voice
      adaptive: true,
      minimum: 12000,
      maximum: 48000
    },
    packetSize: 20 // ms
  },

  latency: {
    target: 100,    // ms
    maximum: 200,   // ms
    measurement: 'End-to-end including network'
  },

  bandwidth: {
    perUser: '20-50 KB/s',
    scaling: 'Adaptive based on network quality'
  }
};

const AUDIO_PROCESSING = {
  echoCancellation: {
    enabled: true,
    type: 'Acoustic Echo Cancellation (AEC)',
    adaptiveTuning: true
  },

  noiseSuppression: {
    enabled: true,
    level: 'Moderate', // Options: Low, Moderate, High
    types: ['Background hum', 'Keyboard', 'Fan noise']
  },

  automaticGainControl: {
    enabled: true,
    targetLevel: -20, // dB
    range: { min: -40, max: -10 }
  },

  voiceActivityDetection: {
    enabled: true,
    sensitivity: 'Medium',
    holdTime: 300 // ms before stopping transmission
  }
};
```

### Mobile-Specific Optimizations

```typescript
const MOBILE_OPTIMIZATIONS = {
  audioRouting: {
    default: 'Speaker',
    withHeadphones: 'Headphones',
    inCall: 'Earpiece optional',
    detection: 'Automatic route detection'
  },

  interruptions: {
    incomingCall: 'Pause voice chat',
    resumeAfterCall: true,
    notification: 'Brief interrupt, auto-resume',
    alarmClock: 'Pause voice chat'
  },

  backgroundAudio: {
    category: 'PlayAndRecord',
    mixWithOthers: true, // Mix with game audio
    duckOthers: false   // Don't lower game audio
  },

  batterySaver: {
    enabled: true,
    actions: [
      'Reduce packet rate',
      'Lower audio quality',
      'Increase VAD threshold'
    ]
  },

  thermalThrottling: {
    detection: true,
    response: 'Reduce audio processing quality'
  }
};
```

---

## User Controls

### Voice Settings

```typescript
const VOICE_SETTINGS = {
  mainToggle: {
    name: 'Voice Chat',
    default: false, // Opt-in
    location: 'Settings > Audio > Voice Chat',
    requirement: 'Microphone permission required'
  },

  inputMode: {
    name: 'Voice Activation Mode',
    options: [
      {
        id: 'push_to_talk',
        name: 'Push-to-Talk',
        description: 'Hold button to speak',
        default: true // Mobile default
      },
      {
        id: 'open_mic',
        name: 'Open Mic',
        description: 'Always transmitting when talking',
        requiresVAD: true
      }
    ]
  },

  pushToTalkButton: {
    location: 'In-match HUD',
    position: 'Customizable',
    size: 'Touch-friendly (60x60dp minimum)',
    hapticFeedback: true
  },

  volumes: {
    masterVoice: {
      name: 'Voice Chat Volume',
      range: { min: 0, max: 100 },
      default: 80
    },
    microphone: {
      name: 'Microphone Sensitivity',
      range: { min: 0, max: 100 },
      default: 70
    },
    perPlayer: {
      name: 'Individual Player Volume',
      range: { min: 0, max: 200 }, // Allow boost
      default: 100
    }
  },

  vadSensitivity: {
    name: 'Voice Detection Sensitivity',
    options: ['Low', 'Medium', 'High'],
    default: 'Medium',
    visibleIf: 'inputMode === open_mic'
  }
};
```

### Player Controls

```typescript
const PLAYER_CONTROLS = {
  mute: {
    mutePlayer: {
      action: 'Stop receiving audio from player',
      duration: 'Session only (resets next match)',
      ui: 'Quick action on player card'
    },
    muteAll: {
      action: 'Stop receiving all voice audio',
      exception: 'Can unmute specific players',
      ui: 'Toggle in voice menu'
    },
    selfMute: {
      action: 'Stop transmitting',
      indicator: 'Muted icon on HUD',
      shortcut: 'Quick tap on PTT button when not held'
    }
  },

  volumeAdjustment: {
    perPlayer: {
      range: '0-200%',
      ui: 'Slider on player card',
      persistence: 'Per session'
    },
    prioritySpeaker: {
      action: 'Boost priority player +50%',
      lowerOthers: '-25% for non-priority',
      useCase: 'Squad leader calls'
    }
  },

  block: {
    action: 'Prevent all voice from player',
    scope: 'Account-level, persists across sessions',
    integration: 'Uses game block system',
    notification: 'Player not notified of block'
  }
};
```

---

## UI Components

### In-Match Voice HUD

```typescript
const VOICE_HUD = {
  pushToTalkButton: {
    position: 'Bottom-right (customizable)',
    size: 60, // dp
    states: {
      idle: 'Microphone icon, greyed',
      active: 'Microphone icon, glowing/pulsing',
      muted: 'Microphone with X, red'
    },
    interaction: {
      hold: 'Transmit',
      tap: 'Toggle mute (when open mic)',
      longPress: 'Open quick settings'
    }
  },

  speakingIndicators: {
    selfIndicator: {
      position: 'Near PTT button',
      visual: 'Waveform animation',
      color: 'Green when transmitting'
    },
    teammateIndicators: {
      position: 'Next to teammate names/icons',
      visual: 'Speaker icon with animation',
      color: 'Green for speaking, grey for silent'
    }
  },

  quickMenu: {
    trigger: 'Long press PTT or dedicated button',
    options: [
      'Mute All',
      'Self Mute Toggle',
      'Open Voice Settings',
      'Leave Voice Channel'
    ],
    animation: 'Radial menu pop-out'
  },

  minimizedState: {
    indicator: 'Small icon showing voice active/muted',
    expandable: 'Tap to show full controls'
  }
};
```

### Voice Settings UI

```typescript
const VOICE_SETTINGS_UI = {
  location: 'Settings > Audio > Voice Chat',

  sections: {
    enable: {
      toggle: 'Enable Voice Chat',
      permission: 'Request microphone if needed',
      warning: 'Voice chat requires microphone access'
    },

    mode: {
      title: 'Voice Activation',
      options: ['Push-to-Talk', 'Open Mic'],
      description: 'How voice transmission is activated'
    },

    volumes: {
      title: 'Voice Volumes',
      sliders: [
        { name: 'Voice Chat Volume', icon: 'speaker' },
        { name: 'Microphone Level', icon: 'microphone', testButton: true }
      ]
    },

    processing: {
      title: 'Audio Processing',
      toggles: [
        { name: 'Echo Cancellation', default: true },
        { name: 'Noise Suppression', default: true }
      ]
    },

    testSection: {
      title: 'Test Your Microphone',
      button: 'Start Test',
      playback: 'Record 3 seconds, play back to user',
      levelMeter: 'Visual input level indicator'
    }
  }
};
```

---

## Moderation System

### Reporting and Abuse Detection

```typescript
const VOICE_MODERATION = {
  reporting: {
    trigger: 'Report option on player card',
    categories: [
      'Harassment/Abuse',
      'Hate Speech',
      'Spam/Disruption',
      'Inappropriate Content',
      'Threats'
    ],
    process: {
      submit: 'Report goes to moderation queue',
      evidence: 'No recordings stored (privacy)',
      context: 'Match ID, timestamps, player IDs'
    },
    outcome: {
      warning: 'First offense - warning issued',
      tempMute: 'Voice chat disabled for duration',
      permaMute: 'Permanent voice chat ban',
      accountBan: 'Severe cases escalated to account ban'
    }
  },

  autoDetection: {
    enabled: false, // Privacy concerns with audio analysis
    future: 'Consider opt-in transcription for moderation'
  },

  muteThreshold: {
    description: 'Auto-mute if frequently muted by others',
    threshold: {
      mutes: 10, // Different players
      window: '7 days',
      action: 'Flag for review'
    }
  },

  penalties: {
    voiceBan: {
      duration: ['24 hours', '7 days', '30 days', 'Permanent'],
      escalation: 'Progressive for repeat offenders',
      appeal: 'Available through support ticket'
    }
  }
};
```

---

## Privacy and Data

### Privacy Protection

```typescript
const VOICE_PRIVACY = {
  recording: {
    serverSide: false, // No server-side recording
    clientSide: false, // No local recording
    evidence: 'Reports handled without recordings'
  },

  dataCollection: {
    metadata: {
      collected: true,
      includes: ['Session duration', 'Channel joins', 'Mute actions'],
      excludes: ['Audio content', 'Voice prints']
    },
    retention: '30 days for metadata'
  },

  consent: {
    optIn: true, // Voice chat disabled by default
    permissionRequest: 'Clear explanation for microphone access',
    disclosure: 'In-app privacy notice for voice features'
  },

  gdpr: {
    rightToAccess: 'Voice metadata exportable',
    rightToDeletion: 'All voice data deleted with account',
    dataPortability: 'Included in account data export'
  },

  minors: {
    ageGating: 'Platform age verification used',
    parentalControls: 'Voice chat can be disabled in family settings'
  }
};
```

---

## Network Requirements

### Connection Handling

```typescript
const VOICE_NETWORKING = {
  protocol: {
    transport: 'UDP with fallback to TCP',
    encryption: 'DTLS',
    ports: 'Provider-specified (usually 3478-3479)'
  },

  qualityOfService: {
    priority: 'High priority packets',
    jitterBuffer: {
      initial: 60, // ms
      adaptive: true,
      maximum: 200 // ms
    }
  },

  networkConditions: {
    excellent: {
      latency: '< 50ms',
      quality: 'Maximum',
      features: 'All enabled'
    },
    good: {
      latency: '50-100ms',
      quality: 'High',
      features: 'All enabled'
    },
    fair: {
      latency: '100-200ms',
      quality: 'Medium',
      features: 'Reduced processing'
    },
    poor: {
      latency: '> 200ms',
      quality: 'Low',
      features: 'Basic voice only'
    }
  },

  adaptation: {
    bitrateScaling: 'Reduce bitrate on congestion',
    packetLossRecovery: 'Forward Error Correction (FEC)',
    reconnection: 'Auto-reconnect on disconnect'
  },

  disconnectHandling: {
    timeout: 5000, // ms
    action: 'Show reconnecting indicator',
    failover: 'Notify user, offer manual reconnect'
  }
};
```

---

## Integration Points

### Game System Integration

```typescript
const VOICE_INTEGRATION = {
  partySystem: {
    onPartyJoin: 'Join party voice channel',
    onPartyLeave: 'Leave voice channel',
    onPartyDisband: 'Destroy voice channel',
    onMemberJoin: 'Notify channel of new member',
    onMemberLeave: 'Notify channel of departure'
  },

  matchmaking: {
    onMatchFound: 'Prepare team voice channel',
    onMatchStart: 'Transition party to team channel',
    onMatchEnd: 'Transition back to party channel'
  },

  teamMode: {
    onElimination: {
      action: 'Move to spectator voice channel',
      canHear: 'Alive teammates (one-way)',
      canSpeak: 'Dead teammates only'
    },
    onTeamWipe: 'All teammates in same spectator channel'
  },

  soloMode: {
    voiceChat: 'Disabled (no team)',
    exception: 'Party voice in lobby only'
  },

  spectatorMode: {
    voiceChat: 'Spectator-only channel',
    isolation: 'Cannot communicate with alive players'
  },

  gameAudio: {
    mixing: 'Voice mixed with game audio',
    ducking: {
      enabled: false, // Don't lower game audio for voice
      option: 'User can enable in settings'
    },
    priority: 'Voice and game audio equal by default'
  }
};
```

---

## Performance Specifications

### Resource Usage

```typescript
const VOICE_PERFORMANCE = {
  cpu: {
    encoding: '< 5% CPU for voice processing',
    decoding: '< 2% CPU per active speaker',
    audioProcessing: '< 3% CPU for AEC/noise suppression'
  },

  memory: {
    sdkOverhead: '< 20 MB',
    audioBuffers: '< 5 MB',
    total: '< 30 MB voice system'
  },

  battery: {
    impact: '< 5% additional drain with voice active',
    optimization: 'VAD prevents transmission when silent'
  },

  startup: {
    sdkInit: '< 500ms',
    channelJoin: '< 1000ms',
    firstAudio: '< 500ms after join'
  }
};
```

---

## Backend Services

### Voice Token Service

```typescript
const VOICE_TOKEN_SERVICE = {
  endpoint: 'POST /api/v1/voice/token',

  request: {
    channelType: 'party | team | spectator',
    channelId: 'string',
    playerId: 'string',
    matchId: 'string (optional)'
  },

  response: {
    token: 'string (provider-specific)',
    channelName: 'string',
    expiresAt: 'ISO8601 timestamp',
    serverUrl: 'string (if applicable)'
  },

  validation: {
    playerInParty: 'Verify player is member of party',
    playerInMatch: 'Verify player is participant in match',
    channelPermission: 'Verify player can join channel type'
  },

  rateLimit: {
    perPlayer: '10 tokens per minute',
    response: '429 Too Many Requests'
  }
};

const VOICE_CHANNEL_SERVICE = {
  createChannel: {
    endpoint: 'POST /api/v1/voice/channels',
    internal: true, // Called by game server only
    request: {
      type: 'party | team',
      partyId: 'string (for party)',
      matchId: 'string (for team)',
      teamId: 'string (for team)'
    },
    response: {
      channelId: 'string',
      channelName: 'string'
    }
  },

  destroyChannel: {
    endpoint: 'DELETE /api/v1/voice/channels/{channelId}',
    internal: true,
    trigger: 'Party disband or match end'
  },

  kickPlayer: {
    endpoint: 'POST /api/v1/voice/channels/{channelId}/kick',
    internal: true,
    useCase: 'Remove banned player from channel'
  }
};
```

---

## Analytics

### Voice Analytics Events

```typescript
const VOICE_ANALYTICS = {
  events: [
    {
      name: 'voice_chat_enabled',
      properties: {
        player_id: 'string',
        mode: 'push_to_talk | open_mic'
      }
    },
    {
      name: 'voice_channel_joined',
      properties: {
        channel_type: 'party | team | spectator',
        player_count: 'number'
      }
    },
    {
      name: 'voice_transmission',
      properties: {
        duration_seconds: 'number',
        channel_type: 'string',
        mode: 'push_to_talk | open_mic'
      }
    },
    {
      name: 'voice_player_muted',
      properties: {
        muted_by: 'user | system',
        reason: 'string (if system)'
      }
    },
    {
      name: 'voice_report_submitted',
      properties: {
        category: 'string',
        match_id: 'string'
      }
    },
    {
      name: 'voice_quality_issue',
      properties: {
        issue_type: 'latency | packet_loss | echo',
        severity: 'low | medium | high'
      }
    }
  ],

  metrics: [
    'Voice chat adoption rate',
    'Average voice session duration',
    'PTT vs Open Mic preference',
    'Mute frequency',
    'Report rate per 1000 sessions',
    'Audio quality scores'
  ]
};
```

---

## Testing Requirements

```typescript
const VOICE_TESTING = {
  functional: [
    'Voice transmission works in PTT mode',
    'Voice transmission works in open mic mode',
    'Party voice channel created and joined correctly',
    'Team voice channel created on match start',
    'Spectator voice isolation enforced',
    'Mute player stops audio reception',
    'Self-mute stops transmission',
    'Volume controls affect audio levels'
  ],

  audioQuality: [
    'Echo cancellation prevents feedback',
    'Noise suppression reduces background noise',
    'VAD correctly detects speech start/stop',
    'Audio latency within 200ms target',
    'No clipping or distortion at normal levels'
  ],

  network: [
    'Voice works on WiFi, 4G, LTE',
    'Graceful degradation on poor network',
    'Reconnection after brief disconnect',
    'Voice quality adapts to bandwidth'
  ],

  integration: [
    'Voice transitions correctly party to match',
    'Voice stops on match end',
    'Elimination moves to spectator channel',
    'Block system prevents voice from blocked players'
  ],

  device: [
    'Works with device speaker',
    'Works with headphones',
    'Handles interruptions (calls, notifications)',
    'Works in background for brief periods'
  ],

  moderation: [
    'Report submission works',
    'Muted players cannot transmit',
    'Voice bans prevent channel join'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial voice chat service specification |
