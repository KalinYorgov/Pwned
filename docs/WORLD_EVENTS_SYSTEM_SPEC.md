# World Events System Specification

## Document Information
- **Task ID:** GAME-017
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies the world events system for Plunderstorm Mobile, implementing dynamic events that occur during matches to create excitement, player convergence, and high-risk/high-reward gameplay moments.

---

## Event System Architecture

### Core Design Principles

```typescript
const EVENT_DESIGN_PRINCIPLES = {
  excitement: {
    principle: 'Events should create memorable match moments',
    implementation: 'Dramatic announcements, unique visuals, valuable rewards'
  },

  optionality: {
    principle: 'Events are bonuses, not requirements to win',
    implementation: 'Can ignore events and still place well'
  },

  convergence: {
    principle: 'Events draw players together for conflict',
    implementation: 'Single location, limited time, contested rewards'
  },

  balance: {
    principle: 'Risk should match reward',
    implementation: 'Better loot = more danger + exposure'
  },

  fairness: {
    principle: 'All players have equal opportunity',
    implementation: 'Global announcements, visible markers, travel time'
  }
};
```

### Event Manager

```typescript
const EVENT_MANAGER = {
  responsibilities: [
    'Track match timeline',
    'Determine event spawning',
    'Manage active events',
    'Handle event rewards',
    'Broadcast announcements'
  ],

  configuration: {
    eventsPerMatch: {
      minimum: 1,
      maximum: 3,
      typical: 2
    },
    eventTiming: {
      firstEventEarliest: 120, // seconds into match
      firstEventLatest: 180,
      minimumBetweenEvents: 90,
      noEventsAfter: 60 // seconds before match typically ends
    },
    simultaneousEvents: {
      maximum: 1,
      overlap: false
    }
  },

  eventSelection: {
    method: 'Weighted random',
    weights: {
      supplyDrop: 40,
      treasureShip: 25,
      krakenSighting: 20,
      ghostGalleon: 15
    },
    noRepeat: 'Same event type won\'t occur twice in a match'
  }
};
```

---

## Event Types

### Event 1: Supply Drop

```typescript
const EVENT_SUPPLY_DROP = {
  id: 'event_supply_drop',
  name: 'Supply Drop',
  rarity: 'Common',

  description: {
    flavor: 'A cargo crate falls from the sky!',
    gameplay: 'Marked location receives rare+ loot'
  },

  spawn: {
    location: {
      method: 'Random within current safe zone',
      avoidPlayers: 20, // meters minimum from any player
      avoidEdge: 30, // meters from zone edge
      terrainRequirement: 'Open area, no buildings'
    },
    announcement: {
      lead: 15, // seconds before drop lands
      message: 'Supply Drop incoming!',
      markerType: 'Parachute icon descending'
    }
  },

  mechanics: {
    dropSequence: {
      warningTime: 15, // seconds
      fallingAnimation: 5, // seconds
      landingEffect: 'Impact dust cloud, beacon light'
    },
    crate: {
      health: 0, // Instantly openable
      openTime: 3, // seconds channel
      interruptible: true
    },
    despawn: {
      timer: 90, // seconds after landing
      warning: 'Supply crate despawning in 30 seconds!',
      warningAt: 60
    }
  },

  rewards: {
    guaranteed: [
      { type: 'ability', rarity: 'rare+', count: 1 },
      { type: 'consumable', rarity: 'uncommon+', count: 2 }
    ],
    bonus: {
      chance: 30,
      item: { type: 'ability', rarity: 'epic' }
    }
  },

  visuals: {
    skyMarker: 'Falling crate with parachute',
    groundMarker: 'Smoke flare, pulsing light beam',
    minimapIcon: 'Crate icon (yellow)'
  },

  audio: {
    announcement: 'Announcer: "Supply drop incoming!"',
    falling: 'Whistling descent',
    landing: 'Heavy thud, crate settling'
  }
};
```

### Event 2: Treasure Ship

```typescript
const EVENT_TREASURE_SHIP = {
  id: 'event_treasure_ship',
  name: 'Treasure Ship',
  rarity: 'Uncommon',

  description: {
    flavor: 'A merchant vessel passes by!',
    gameplay: 'Board the ship for epic loot, limited time window'
  },

  spawn: {
    location: {
      method: 'Edge of map, water area',
      path: 'Sails across map edge over 60 seconds',
      boardingZones: 3 // Points where players can board
    },
    announcement: {
      lead: 20, // seconds before ship arrives
      message: 'Treasure Ship approaching!',
      markerType: 'Ship icon on trajectory'
    }
  },

  mechanics: {
    ship: {
      speed: 8, // meters per second
      pathDuration: 60, // seconds visible
      boardingMethod: 'Grapple, jump from shore, or swim'
    },
    deck: {
      size: { length: 30, width: 12 }, // meters
      hazards: ['Patrolling NPC guards (2)', 'Cannon fire zones'],
      cover: 'Crates, masts, railings'
    },
    lootChests: {
      count: 4,
      openTime: 2, // seconds each
      distribution: 'Spread across deck'
    },
    exitWindow: {
      warning: 'Ship leaving the area!',
      warningAt: 15, // seconds before despawn
      forcedExit: 'Teleport to nearest shore'
    }
  },

  enemies: {
    shipGuards: {
      count: 2,
      type: 'Skeleton Pirate (elite)',
      behavior: 'Patrol deck, attack boarders',
      respawn: false
    }
  },

  rewards: {
    perChest: [
      { type: 'ability', rarity: 'epic', chance: 50 },
      { type: 'ability', rarity: 'rare', chance: 50 },
      { type: 'gold', amount: [100, 200] }
    ],
    captainChest: {
      location: 'Ship helm',
      guardedBy: 'Elite guard',
      rewards: [
        { type: 'ability', rarity: 'legendary', chance: 40 },
        { type: 'ability', rarity: 'epic', chance: 60 }
      ]
    }
  },

  visuals: {
    ship: 'Ornate merchant galleon',
    wake: 'Trailing water effects',
    minimapIcon: 'Ship icon with path trajectory'
  },

  audio: {
    announcement: 'Announcer: "Treasure ship on the horizon!"',
    ambient: 'Creaking wood, sails flapping, waves',
    combat: 'Sword clashing, cannon booms'
  }
};
```

### Event 3: Kraken Sighting

```typescript
const EVENT_KRAKEN_SIGHTING = {
  id: 'event_kraken',
  name: 'Kraken Sighting',
  rarity: 'Rare',

  description: {
    flavor: 'The legendary Kraken surfaces!',
    gameplay: 'Defeat the mini-boss for legendary rewards'
  },

  spawn: {
    location: {
      method: 'Deep water area within safe zone',
      requirement: 'At least 30m from shore',
      areaRadius: 40 // meters engagement zone
    },
    announcement: {
      lead: 10, // seconds (less warning = more surprise)
      message: 'The Kraken stirs below!',
      markerType: 'Kraken icon, water disturbance'
    }
  },

  mechanics: {
    boss: {
      health: 3000,
      phases: 2,
      enrage: 'After 90 seconds, damage increases 50%'
    },
    phase1: {
      name: 'Tentacle Assault',
      attacks: [
        {
          name: 'Tentacle Slam',
          damage: 80,
          area: 8, // meters radius
          telegraph: 2 // seconds
        },
        {
          name: 'Ink Spray',
          effect: 'Blinds players for 3 seconds',
          cooldown: 15
        },
        {
          name: 'Whirlpool',
          effect: 'Pulls players toward center',
          damage: 20, // per second
          duration: 5
        }
      ]
    },
    phase2: {
      trigger: 'Health below 40%',
      name: 'Kraken\'s Fury',
      changes: [
        'Attacks 30% faster',
        'Adds: Tentacle Sweep (line attack)',
        'Whirlpool spawns twice'
      ]
    },
    defeat: {
      animation: 'Kraken sinks below',
      duration: 5, // seconds
      lootSpawn: 'Floating crate surfaces'
    },
    timeout: {
      timer: 120, // seconds
      onTimeout: 'Kraken submerges, no rewards'
    }
  },

  rewards: {
    guaranteed: [
      { type: 'ability', rarity: 'legendary', count: 1 },
      { type: 'ability', rarity: 'epic', count: 1 }
    ],
    bonus: [
      { type: 'consumable', rarity: 'epic', count: 3 },
      { type: 'gold', amount: 300 }
    ],
    participation: {
      description: 'Damage dealers get credit',
      minimumDamage: 200,
      reward: { type: 'ability', rarity: 'rare' }
    }
  },

  pvpInteraction: {
    enabled: true,
    description: 'Players can fight each other during event',
    lastHitReward: true, // Killing blow gets legendary
    damageCredit: true // Others get participation rewards
  },

  visuals: {
    kraken: 'Massive tentacles, glowing eyes below surface',
    waterEffects: 'Churning water, spray, ink clouds',
    minimapIcon: 'Kraken icon (purple)'
  },

  audio: {
    announcement: 'Announcer: "By Davy Jones... it\'s the Kraken!"',
    ambient: 'Deep rumbling, water churning',
    attacks: 'Massive splashing, roars'
  }
};
```

### Event 4: Ghost Galleon

```typescript
const EVENT_GHOST_GALLEON = {
  id: 'event_ghost_galleon',
  name: 'Ghost Galleon',
  rarity: 'Rare',

  description: {
    flavor: 'A cursed ship phases into existence!',
    gameplay: 'Clear the ghost ship to claim its cursed treasures'
  },

  spawn: {
    location: {
      method: 'Beach or dock area within safe zone',
      requirement: 'Near water but accessible from land',
      shipPosition: 'Anchored, gangplank to shore'
    },
    announcement: {
      lead: 15,
      message: 'A Ghost Galleon materializes!',
      markerType: 'Ghost ship icon, ethereal glow'
    }
  },

  mechanics: {
    ship: {
      size: { length: 40, width: 15 },
      decks: 2, // Upper deck and cargo hold
      entryPoints: 2, // Gangplank + hole in hull
      etherealEffect: 'Transparent, eerie green glow'
    },
    clearObjective: {
      type: 'Defeat all ghosts',
      waves: 3,
      waveComposition: {
        wave1: { ghostPirates: 4 },
        wave2: { ghostPirates: 6, ghostCaptain: 1 },
        wave3: { ghostPirates: 4, ghostCaptain: 2 }
      }
    },
    progress: {
      sharedAcrossPlayers: true,
      killCredit: 'Damage dealt determines loot share',
      waveAnnouncement: 'Wave {n} incoming!'
    },
    timeout: {
      timer: 150, // seconds
      onTimeout: 'Ship fades, remaining ghosts despawn'
    }
  },

  enemies: {
    ghostPirate: {
      health: 150,
      damage: 30,
      behavior: 'Aggressive, phases through cover briefly',
      drops: { gold: [10, 25] }
    },
    ghostCaptain: {
      health: 400,
      damage: 50,
      abilities: ['Spectral Slash (cone)', 'Terrify (fear effect)'],
      drops: { ability: 'rare', chance: 40 }
    }
  },

  rewards: {
    completion: {
      treasureRoom: 'Unlocks after wave 3',
      chests: 3,
      chestContents: [
        { type: 'ability', rarity: 'epic+', count: 1 },
        { type: 'consumable', rarity: 'rare+', count: 2 }
      ]
    },
    contribution: {
      topDamage: 'First pick of chests',
      participation: 'Access to remaining chests'
    },
    cursedBonus: {
      chance: 20,
      item: 'Cursed Legendary Ability (unique variants)'
    }
  },

  pvpInteraction: {
    enabled: true,
    note: 'Players can attack each other inside ship',
    ghostAggro: 'Ghosts attack all players equally'
  },

  visuals: {
    ship: 'Translucent galleon, ghostly glow',
    ghosts: 'Spectral pirates, chains, tattered clothes',
    environment: 'Fog, floating candles, ghost fire',
    minimapIcon: 'Ghost ship icon (green)'
  },

  audio: {
    announcement: 'Announcer: "The Ghost Galleon appears!"',
    ambient: 'Creaking, chains rattling, whispers',
    combat: 'Ethereal screams, ghostly moans'
  }
};
```

---

## Event Announcement System

### Global Announcements

```typescript
const EVENT_ANNOUNCEMENTS = {
  components: {
    voiceOver: {
      enabled: true,
      voice: 'Match announcer',
      priority: 'High (interrupts other VO)'
    },
    uiNotification: {
      type: 'Large banner',
      position: 'Top center',
      duration: 5000, // ms
      animation: 'Slide down + pulse'
    },
    minimapMarker: {
      appear: 'On announcement',
      style: 'Event-specific icon',
      pulse: true,
      persist: 'Until event ends'
    },
    screenEdgeIndicator: {
      enabled: true,
      pointsTo: 'Event location',
      distance: 'Shows approximate range'
    }
  },

  timing: {
    preAnnouncement: {
      time: 'Event-specific (10-20 seconds)',
      message: 'Event incoming!'
    },
    eventStart: {
      message: 'Event name + brief description'
    },
    eventWarning: {
      trigger: '30 seconds remaining',
      message: 'Event ending soon!'
    },
    eventEnd: {
      message: 'Event concluded',
      duration: 3000
    }
  }
};
```

### Event UI Elements

```typescript
const EVENT_UI = {
  activeEventPanel: {
    position: 'Left side of screen',
    shows: {
      eventName: true,
      timeRemaining: true,
      objectiveProgress: 'If applicable',
      rewardPreview: 'Icon hints'
    }
  },

  eventMarker: {
    worldSpace: {
      icon: 'Event-specific',
      distanceText: true,
      visible: 'Through walls/terrain',
      fade: 'At close range'
    }
  },

  eventCompass: {
    position: 'Top of screen',
    showsEventDirection: true,
    icon: 'Event type icon'
  }
};
```

---

## Event Rewards Framework

### Reward Tiers

```typescript
const EVENT_REWARD_TIERS = {
  supplyDrop: {
    tier: 'Standard',
    expectedValue: 'Rare+ ability guaranteed',
    risk: 'Low-Medium (exposure while opening)'
  },

  treasureShip: {
    tier: 'High',
    expectedValue: '2-3 Epic abilities possible',
    risk: 'Medium (NPC guards + other players)'
  },

  krakenSighting: {
    tier: 'Premium',
    expectedValue: 'Legendary ability guaranteed',
    risk: 'Very High (boss + PvP + time limit)'
  },

  ghostGalleon: {
    tier: 'High',
    expectedValue: 'Multiple Epic+ abilities',
    risk: 'High (waves of enemies + PvP)'
  }
};
```

### Loot Distribution

```typescript
const LOOT_DISTRIBUTION = {
  contested: {
    description: 'Multiple players participate',
    methods: {
      firstOpen: 'First to open chest claims it',
      damageShare: 'Boss rewards split by damage dealt',
      lastHit: 'Killing blow gets best item',
      proximity: 'Closest player to dropped loot'
    }
  },

  fairness: {
    announceEarly: 'Give time for all players to arrive',
    multipleRewards: 'Enough loot for several players',
    participationRewards: 'Consolation prizes for effort'
  },

  antiGriefing: {
    cantDestroyLoot: true,
    lootDespawnTimer: 60, // seconds on ground
    openingProtection: 'Damage doesn\'t interrupt first 0.5s'
  }
};
```

---

## Server Implementation

### Event Spawning Logic

```typescript
const EVENT_SPAWNING = {
  matchStart: {
    scheduleEvents: true,
    method: 'Pre-determine event times and types',
    storage: 'Server match state'
  },

  spawnConditions: {
    playerCount: {
      minimum: 20, // Don't spawn if too few players
      ideal: 30+
    },
    zonePhase: {
      earliest: 2, // After first zone closure
      latest: 'One before final zone'
    }
  },

  locationSelection: {
    algorithm: 'Weighted random with constraints',
    constraints: [
      'Within current/next safe zone',
      'Minimum distance from players',
      'Appropriate terrain (water for Kraken)',
      'Accessible location'
    ]
  },

  networkReplication: {
    announcement: 'Reliable broadcast to all',
    eventState: 'Server authoritative',
    lootSpawns: 'Server determines, clients display'
  }
};
```

### Performance Considerations

```typescript
const EVENT_PERFORMANCE = {
  npcManagement: {
    maxNPCsPerEvent: 12,
    lodDistance: 100, // meters
    culling: 'Standard enemy culling rules'
  },

  effectsOptimization: {
    eventVFX: 'Use LOD for distant events',
    particleBudget: 'Share with normal gameplay budget',
    audioDistance: 'Falloff based on distance'
  },

  serverLoad: {
    eventProcessing: '< 5% additional CPU',
    broadcastOptimization: 'Batch event updates',
    staggeredSpawns: 'Don\'t spawn all NPCs at once'
  }
};
```

---

## Analytics

```typescript
const EVENT_ANALYTICS = {
  events: [
    {
      name: 'world_event_spawned',
      properties: {
        event_type: 'string',
        match_time: 'seconds',
        players_remaining: 'number',
        location: 'Vector3'
      }
    },
    {
      name: 'world_event_participated',
      properties: {
        event_type: 'string',
        player_count: 'number',
        time_to_arrival: 'seconds',
        damage_dealt: 'number (if applicable)'
      }
    },
    {
      name: 'world_event_completed',
      properties: {
        event_type: 'string',
        duration: 'seconds',
        participants: 'number',
        rewards_claimed: 'number',
        pvp_kills_during: 'number'
      }
    },
    {
      name: 'world_event_ignored',
      properties: {
        event_type: 'string',
        closest_player_distance: 'meters',
        reason: 'timeout | avoided'
      }
    }
  ],

  metrics: [
    'Event participation rate',
    'Average players per event',
    'Event-related kills',
    'Win rate of event participants vs non-participants',
    'Most popular event types'
  ]
};
```

---

## Testing Requirements

```typescript
const EVENT_TESTING = {
  functional: [
    'Each event spawns correctly',
    'Announcements trigger properly',
    'Rewards drop as specified',
    'Events despawn after timeout',
    'Location selection avoids invalid areas'
  ],

  balance: [
    'Events don\'t heavily favor certain playstyles',
    'Rewards feel worth the risk',
    'Event frequency doesn\'t overwhelm matches',
    'Final zones don\'t have active events'
  ],

  performance: [
    'Server handles events with 60 players',
    'Client FPS stable during events',
    'Network traffic within budget',
    'NPC AI doesn\'t cause lag spikes'
  ],

  edge_cases: [
    'Event location overlaps with zone closure',
    'All players ignore event',
    'Single player contests event',
    'Event spawns but all players eliminated'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial world events system specification |
