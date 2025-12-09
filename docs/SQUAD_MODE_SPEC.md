# Squad Mode Specification

## Document Information
- **Task ID:** GAME-022
- **Priority:** P3
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Squad Mode (Trios/Quads) for Plunderstorm Mobile, extending team-based gameplay to support 3-4 player squads for enhanced social play and team coordination experiences.

---

## Squad Configuration

### Mode Types

```typescript
const SQUAD_MODES = {
  trios: {
    name: 'Trios',
    teamSize: 3,
    totalPlayers: 60,
    teamsCount: 20,
    description: 'Battle in squads of three'
  },

  quads: {
    name: 'Squads',
    teamSize: 4,
    totalPlayers: 60,
    teamsCount: 15,
    description: 'Battle in squads of four'
  },

  flexQueue: {
    enabled: true,
    description: 'Join with any party size, fill with randoms',
    minPartySize: 1,
    maxPartySize: 4
  }
};
```

### Team Composition

```typescript
const TEAM_COMPOSITION = {
  partyPlay: {
    fullSquad: {
      trios: 'Party of 3 players',
      quads: 'Party of 4 players',
      behavior: 'Placed together, no fill needed'
    },
    partialSquad: {
      behavior: 'Fill remaining slots with randoms',
      setting: 'No Fill option (play undermanned)'
    }
  },

  soloQueue: {
    enabled: true,
    behavior: 'Match with random squad',
    voiceChat: 'Auto-join team voice channel',
    textChat: 'Team chat enabled'
  },

  noFill: {
    enabled: true,
    description: 'Play with fewer than max team size',
    advantage: 'Faster queue times',
    disadvantage: 'Numerical disadvantage in fights'
  }
};
```

---

## Squad Features

### Communication

```typescript
const SQUAD_COMMUNICATION = {
  voiceChat: {
    enabled: true,
    channels: {
      team: 'Default team voice channel',
      party: 'Private party voice (premade only)',
      mute: 'Per-player mute option'
    },
    settings: {
      autoJoin: 'Join team voice automatically (toggle)',
      pushToTalk: 'Optional push-to-talk mode',
      voiceActivation: 'Sensitivity adjustment',
      outputDevice: 'Speaker or headphone detection'
    }
  },

  textChat: {
    enabled: true,
    channels: {
      team: 'Squad text chat',
      all: 'All chat (optional, off by default)'
    },
    quickMessages: {
      presets: [
        'Good game!',
        'Thanks!',
        'Sorry',
        'Help!',
        'On my way',
        'Fall back',
        'Push forward',
        'Need healing'
      ]
    }
  },

  pingSystem: {
    types: [
      'generic',
      'enemy',
      'loot',
      'danger',
      'help',
      'goingHere',
      'watching',
      'defendHere'
    ],
    squadMarkers: {
      enabled: true,
      count: 4,
      colors: ['Red', 'Blue', 'Green', 'Yellow'],
      persistence: 'Until replaced or cleared'
    }
  }
};
```

### Revive System

```typescript
const SQUAD_REVIVE = {
  downedState: {
    enabled: true,
    health: 100,
    bleedoutTime: 60,
    crawlSpeed: '50% of walk speed',
    selfRevive: false
  },

  teamRevive: {
    duration: 5,
    cancelOnDamage: true,
    multipleRevives: 'Each player can revive once simultaneously',
    healthAfterRevive: 30,
    shieldAfterRevive: 0,
    invulnerabilityFrames: 1.5
  },

  knockdownProgression: {
    firstDown: { bleedoutTime: 60 },
    secondDown: { bleedoutTime: 45 },
    thirdDown: { bleedoutTime: 30 },
    fourthDown: { bleedoutTime: 15 },
    reset: 'Timer resets on full elimination round'
  },

  thirsting: {
    description: 'Finishing downed players',
    enabled: true,
    damageReduction: 'None',
    incentive: 'Grants elimination XP/Plunder'
  }
};
```

### Respawn System

```typescript
const SQUAD_RESPAWN = {
  respawnBeacons: {
    enabled: true,
    count: '8-10 per map',
    locations: 'Distributed evenly across map',
    visibility: 'Marked on map'
  },

  respawnMechanic: {
    deathBox: {
      description: 'Eliminated teammates drop banner',
      pickupTime: 2,
      expiration: 90,
      carrier: 'Any living teammate'
    },
    beaconUse: {
      channelTime: 7,
      cancelOnDamage: true,
      oneTimeUse: true,
      beaconCooldown: 'Beacon destroyed after use'
    },
    respawnDrop: {
      method: 'Sky dive from above beacon',
      loadout: 'Basic loadout only',
      health: 100,
      abilities: 'None (must loot)'
    }
  },

  respawnWindow: {
    enabled: true,
    duration: 'Until final 2 circles',
    restriction: 'No respawns in final 2 storm phases'
  }
};
```

---

## Squad UI

### HUD Elements

```typescript
const SQUAD_HUD = {
  teamPanel: {
    location: 'Left side of screen',
    elements: [
      'Teammate name',
      'Health bar',
      'Shield bar',
      'Status icon (alive/downed/dead)',
      'Distance indicator',
      'Mic activity indicator',
      'Ping cooldown indicator'
    ],
    scaling: {
      trios: 'Compact 3-player panel',
      quads: 'Compact 4-player panel',
      adjustable: 'Size in settings'
    }
  },

  minimapMarkers: {
    teammates: {
      color: 'Unique per teammate (match team panel)',
      icon: 'Arrow showing direction',
      visibility: 'Always visible'
    },
    downedTeammate: {
      icon: 'Skull icon',
      color: 'Dimmed teammate color',
      pulse: 'Pulsing effect'
    },
    respawnBanner: {
      icon: 'Banner icon',
      color: 'Bright yellow',
      timer: 'Expiration countdown'
    }
  },

  worldMarkers: {
    teammates: {
      nameplate: 'Above head',
      healthBar: 'Mini health bar',
      throughWalls: 'Visible through obstacles',
      maxDistance: 'Always visible'
    }
  }
};
```

### Party Interface

```typescript
const PARTY_UI = {
  preLobby: {
    partyPanel: {
      maxSize: 4,
      elements: [
        'Player avatar',
        'Player name',
        'Ready status',
        'Voice status',
        'Equipped cosmetics preview'
      ]
    },
    partyActions: {
      invite: 'Invite friends to party',
      kick: 'Party leader can remove members',
      leave: 'Leave party',
      promote: 'Transfer leader',
      ready: 'Toggle ready status'
    },
    modeSelection: {
      automatic: 'Mode based on party size',
      manual: 'Leader selects Trios or Quads',
      noFill: 'Toggle to play undermanned'
    }
  },

  inviteSystem: {
    methods: [
      'In-game friend list',
      'Recent players',
      'Share invite code',
      'Deep link',
      'Platform friend list (Game Center, Google Play)'
    ],
    notifications: {
      incoming: 'Push + in-game notification',
      accepted: 'Player joined notification',
      declined: 'Optional notification'
    }
  }
};
```

---

## Matchmaking

### Queue System

```typescript
const SQUAD_MATCHMAKING = {
  queueTypes: {
    trios: {
      name: 'Trios Queue',
      targetPlayers: 60,
      minToStart: 45,
      teamSize: 3
    },
    quads: {
      name: 'Squads Queue',
      targetPlayers: 60,
      minToStart: 48,
      teamSize: 4
    }
  },

  filling: {
    enabled: true,
    preference: 'Match party sizes when possible',
    behavior: 'Fill partial squads with solos/duos',
    timeout: 'Start with bots if not enough players after 120s'
  },

  botBackfill: {
    enabled: true,
    scenario: 'Low population times',
    behavior: 'Add bot squads to fill lobby',
    maxBots: '30% of lobby',
    disclosure: 'Bots labeled in kill feed'
  },

  skillMatching: {
    casual: {
      enabled: false,
      description: 'Random skill distribution'
    },
    ranked: {
      enabled: true,
      metric: 'Average party MMR',
      tolerance: 'Expand over time'
    }
  },

  regionMatching: {
    primary: 'Lowest ping server',
    crossRegion: 'Only if queue time excessive',
    threshold: '180 seconds before expanding'
  }
};
```

### Party Handling

```typescript
const PARTY_MATCHMAKING = {
  partySlotting: {
    fullSquad: 'Placed as complete team',
    trio: {
      inQuads: 'Fill 1 random',
      inTrios: 'Complete team'
    },
    duo: {
      inQuads: 'Fill 2 randoms or match with duo',
      inTrios: 'Fill 1 random'
    },
    solo: {
      inQuads: 'Fill 3 randoms or mixed groups',
      inTrios: 'Fill 2 randoms'
    }
  },

  crossPartySize: {
    scenario: 'Party of 4 wants Trios',
    behavior: 'Prevent queue (team too large)',
    message: 'Your party is too large for Trios'
  },

  leaderDisconnect: {
    behavior: 'Promote next player to leader',
    queueStatus: 'Maintain position in queue'
  }
};
```

---

## Gameplay Adjustments

### Loot Scaling

```typescript
const SQUAD_LOOT = {
  chestContents: {
    adjustment: 'Slightly increased loot per chest',
    multiplier: 1.2,
    reason: 'More players need gear'
  },

  floorLoot: {
    density: 'Maintained from solo/duo',
    reasoning: 'Squads split up to loot efficiently'
  },

  hotZones: {
    count: 'Increased from 2 to 3',
    loot: 'High-tier loot concentration',
    risk: 'Multiple squads contest'
  },

  specialDrops: {
    carePackages: {
      frequency: 'Same as other modes',
      contents: 'Enhanced for squad play',
      quantity: 'More items per package'
    }
  }
};
```

### Combat Balance

```typescript
const SQUAD_BALANCE = {
  damageNumbers: {
    adjustment: 'No change from solo/duo',
    reason: 'TTK balanced by squad support'
  },

  abilityBalance: {
    aoeAbilities: {
      adjustment: 'No change',
      impact: 'More valuable in squad fights'
    },
    healingAbilities: {
      adjustment: 'No change',
      impact: 'Critical for squad sustain'
    },
    reviveAbilities: {
      addition: 'Consider squad-specific abilities',
      example: 'Faster revive, remote revive'
    }
  },

  thirdParty: {
    risk: 'Higher due to more teams',
    mitigation: 'Larger storm circles early game',
    pacing: 'Slower early game, faster late game'
  }
};
```

### Storm Adjustments

```typescript
const SQUAD_STORM = {
  timing: {
    earlyGame: {
      adjustment: 'Longer first circle',
      reason: 'More time for squads to loot'
    },
    lateGame: {
      adjustment: 'Similar to solo/duo',
      reason: 'Maintain final fight tension'
    }
  },

  circleSize: {
    initial: 'Slightly larger',
    final: 'Same as other modes',
    reason: 'Accommodate more active teams'
  },

  phases: {
    count: 8,
    timing: [
      { phase: 1, wait: 120, shrink: 60 },
      { phase: 2, wait: 90, shrink: 60 },
      { phase: 3, wait: 90, shrink: 60 },
      { phase: 4, wait: 75, shrink: 45 },
      { phase: 5, wait: 60, shrink: 45 },
      { phase: 6, wait: 45, shrink: 30 },
      { phase: 7, wait: 30, shrink: 30 },
      { phase: 8, wait: 20, shrink: 20 }
    ]
  }
};
```

---

## Scoring and Rewards

### Placement Points

```typescript
const SQUAD_SCORING = {
  placement: {
    1: { points: 100, label: 'Victory Royale!' },
    2: { points: 60, label: 'Runner Up' },
    3: { points: 45, label: 'Top 3' },
    4: { points: 35, label: 'Top 4' },
    5: { points: 30, label: 'Top 5' },
    top10: { points: 20, label: 'Top 10' },
    top15: { points: 10, label: 'Top 15' }
  },

  eliminations: {
    pointsPerElim: 10,
    assists: {
      enabled: true,
      points: 5,
      window: 10
    },
    teamCredit: {
      enabled: true,
      description: 'All squad members share elimination credit'
    }
  },

  revives: {
    pointsPerRevive: 5,
    respawns: {
      pointsPerRespawn: 10,
      description: 'Using respawn beacon'
    }
  }
};
```

### XP Distribution

```typescript
const SQUAD_XP = {
  individual: {
    baseXP: 'Personal performance',
    eliminations: 'Per elimination',
    damage: 'Per 100 damage dealt',
    survival: 'Per minute survived'
  },

  team: {
    bonus: 'Squad placement multiplier',
    victory: '50% XP bonus for wins',
    topFive: '25% XP bonus'
  },

  challenges: {
    squadChallenges: {
      examples: [
        'Win a Squads match',
        'Revive 5 teammates',
        'Respawn a teammate',
        'Win with full squad alive',
        'Get 10 squad eliminations'
      ],
      rewards: 'Bonus XP, Gold, Battle Pass progress'
    }
  }
};
```

---

## Performance Optimization

### Network Optimization

```typescript
const SQUAD_NETWORK = {
  bandwidth: {
    perPlayer: '30-40 KB/s baseline',
    squadData: 'Additional 5 KB/s for team state',
    total: 'Optimized for mobile data'
  },

  prioritization: {
    teammates: 'Higher update rate for squad',
    nearbyEnemies: 'Full update rate',
    distantPlayers: 'Reduced update rate'
  },

  cullDistance: {
    visual: '200m render distance',
    network: '300m state sync',
    audio: '50m directional audio'
  }
};
```

### Client Performance

```typescript
const SQUAD_PERFORMANCE = {
  rendering: {
    maxVisiblePlayers: 20,
    lodDistance: 'Dynamic based on device',
    effectsReduction: 'Scale particles in large fights'
  },

  memory: {
    playerPooling: 'Reuse player objects',
    uiOptimization: 'Lazy load squad panels',
    assetStreaming: 'Progressive cosmetic loading'
  },

  targetFramerate: {
    lowEnd: '30 FPS stable',
    midRange: '45 FPS stable',
    highEnd: '60 FPS stable'
  },

  squadFightOptimization: {
    trigger: '8+ players in proximity',
    actions: [
      'Reduce particle count',
      'Lower effect quality',
      'Simplify distant LODs',
      'Throttle non-essential updates'
    ]
  }
};
```

---

## Anti-Grief Measures

### Team Killing Prevention

```typescript
const TEAM_PROTECTION = {
  friendlyFire: {
    enabled: false,
    description: 'Cannot damage teammates'
  },

  griefingPrevention: {
    blocking: {
      description: 'Can pass through teammates',
      duration: '3 seconds of contact'
    },
    lootSharing: {
      dropForTeam: true,
      pingLoot: true
    }
  },

  reportSystem: {
    categories: [
      'Intentional throwing',
      'Abandonment',
      'Toxicity',
      'Offensive content'
    ],
    consequences: {
      warning: 'First offense',
      timeout: 'Repeated offenses',
      ban: 'Severe cases'
    }
  }
};
```

### AFK and Leave Prevention

```typescript
const SQUAD_ABANDONMENT = {
  afkDetection: {
    threshold: 60,
    warning: 'AFK warning at 45 seconds',
    action: 'Boot from match, no rewards'
  },

  leavepenalty: {
    casual: {
      enabled: false,
      description: 'No penalty in casual'
    },
    ranked: {
      enabled: true,
      firstLeave: '-20 RP warning',
      repeatLeave: '-50 RP + timeout',
      timeout: '5 minutes to 2 hours'
    }
  },

  reconnection: {
    enabled: true,
    window: 180,
    behavior: 'Rejoin as downed or at spawn'
  }
};
```

---

## Data Model

### Squad Data Structure

```typescript
const DATA_MODEL = {
  squad: {
    squadId: 'string',
    matchId: 'string',
    mode: 'trios | quads',
    members: 'array<SquadMember>',
    placement: 'number',
    totalEliminations: 'number',
    isEliminated: 'boolean'
  },

  squadMember: {
    playerId: 'string',
    playerName: 'string',
    status: 'alive | downed | eliminated | disconnected',
    kills: 'number',
    assists: 'number',
    damage: 'number',
    revivesGiven: 'number',
    respawnsGiven: 'number',
    position: 'Vector3',
    health: 'number',
    shield: 'number'
  },

  respawnBanner: {
    playerId: 'string',
    squadId: 'string',
    position: 'Vector3',
    expirationTime: 'timestamp',
    carrierId: 'string (nullable)'
  },

  respawnBeacon: {
    beaconId: 'string',
    position: 'Vector3',
    isUsed: 'boolean'
  }
};
```

### API Endpoints

```typescript
const API_ENDPOINTS = {
  squad: {
    createParty: 'POST /api/v1/party/create',
    joinParty: 'POST /api/v1/party/{code}/join',
    leaveParty: 'POST /api/v1/party/{id}/leave',
    inviteToParty: 'POST /api/v1/party/{id}/invite',
    kickFromParty: 'POST /api/v1/party/{id}/kick',
    setReady: 'PUT /api/v1/party/{id}/ready',
    startMatchmaking: 'POST /api/v1/matchmaking/squad'
  },

  inMatch: {
    reviveTeammate: 'POST /api/v1/match/{id}/revive',
    pickupBanner: 'POST /api/v1/match/{id}/banner/pickup',
    useRespawnBeacon: 'POST /api/v1/match/{id}/respawn'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'All party sizes matchmake correctly',
    'Revive and respawn systems work',
    'Squad UI displays correctly for 3-4 players',
    'Communication systems functional',
    'Banner pickup and expiration correct'
  ],

  performance: [
    '60 players in squads maintains target FPS',
    'Squad fights (8+ players) stay smooth',
    'Network bandwidth acceptable on mobile data',
    'No memory leaks over extended play'
  ],

  edge: [
    'Player disconnects mid-revive',
    'Banner expires while being carried',
    'Respawn beacon used simultaneously',
    'Full squad wipe respawn race',
    'Mixed skill levels in party',
    'Cross-region squad play'
  ],

  social: [
    'Party formation and dissolution',
    'Voice chat quality and toggle',
    'Ping system usability',
    'Report system functionality'
  ],

  balance: [
    'Loot sufficient for squads',
    'Storm timing appropriate',
    'Third-party frequency manageable',
    'Final circle size adequate'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial squad mode specification |
