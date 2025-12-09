# Limited-Time Modes (LTM) Specification

## Document Information
- **Task ID:** LIVE-008
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Limited-Time Mode (LTM) system for Plunderstorm Mobile, defining rotating alternative game modes that provide variety, test new mechanics, and keep gameplay fresh for returning players.

---

## LTM Philosophy

### Design Principles

```typescript
const LTM_PRINCIPLES = {
  variety: {
    description: 'LTMs should feel distinctly different from standard modes',
    guideline: 'At least one major rule change that alters strategy'
  },

  accessibility: {
    description: 'LTMs should be easy to understand and jump into',
    guideline: 'Core rules explainable in 2-3 sentences'
  },

  balance: {
    description: 'LTMs should be fun, not necessarily perfectly balanced',
    guideline: 'Prioritize fun and novelty over competitive fairness'
  },

  temporary: {
    description: 'LTMs are intentionally limited to create urgency',
    guideline: 'Rotation keeps modes feeling special when they return'
  },

  testing: {
    description: 'LTMs can test mechanics for potential permanent addition',
    guideline: 'Gather data on player reception to new mechanics'
  }
};
```

---

## Launch LTM Roster

### Swords Only

```typescript
const SWORDS_ONLY_LTM = {
  id: 'ltm_swords_only',
  name: 'Swords Only',
  tagline: 'No ranged, no problem',
  description: 'All ranged abilities are disabled. Victory goes to the mightiest melee combatant.',

  rules: {
    abilityRestriction: {
      disabled: ['All ranged abilities', 'All projectile abilities'],
      enabled: ['Melee abilities only', 'Movement abilities', 'Defensive abilities'],
      exception: 'Utility abilities that don\'t deal ranged damage allowed'
    },
    loot: {
      modification: 'Ranged abilities do not spawn in chests',
      meleeBonus: '+50% melee ability drop rate'
    },
    gameplay: {
      circleSpeed: 'Standard',
      playerCount: 40,
      mapSize: 'Standard'
    }
  },

  strategy: {
    impact: 'Close-quarters combat focus',
    tips: [
      'Movement abilities are crucial for engaging',
      'Defensive abilities provide survivability',
      'Positioning near cover matters less'
    ]
  },

  rewards: {
    firstWin: { item: 'Sword Master Emote', oneTime: true },
    participation: { xpBonus: '+10% match XP' }
  }
};
```

### Chaos Mode

```typescript
const CHAOS_MODE_LTM = {
  id: 'ltm_chaos_mode',
  name: 'Chaos Mode',
  tagline: 'Maximum power, maximum mayhem',
  description: 'All abilities drop at Epic rarity. Unleash devastating power from the first chest.',

  rules: {
    abilityRarity: {
      override: 'All abilities spawn as Epic',
      upgrading: 'Disabled (already max power)',
      duplicates: 'Grant bonus effect duration instead'
    },
    loot: {
      spawnRate: 'Standard chest spawn rate',
      content: 'All abilities Epic, gold reduced 50%'
    },
    gameplay: {
      circleSpeed: '+25% faster shrink',
      playerCount: 40,
      matchDuration: 'Shorter due to faster pace'
    }
  },

  strategy: {
    impact: 'Faster, more explosive matches',
    tips: [
      'Early game is immediately deadly',
      'Every player is dangerous from the start',
      'Aggressive play rewarded'
    ]
  },

  rewards: {
    firstWin: { item: 'Chaos Crown Accessory', oneTime: true },
    eliminations: { bonus: '+5 Gold per elimination (mode bonus)' }
  }
};
```

### Duo Blitz

```typescript
const DUO_BLITZ_LTM = {
  id: 'ltm_duo_blitz',
  name: 'Duo Blitz',
  tagline: 'Fast and furious duos',
  description: 'Duo-only matches with a faster circle and shared respawns.',

  rules: {
    teamSize: {
      requirement: 'Exactly 2 players (duos only)',
      soloQueue: 'Auto-matched with random partner',
      premade: 'Recommended'
    },
    respawn: {
      system: 'Shared respawn pool',
      lives: 2,
      mechanism: 'Partner can revive at beacon',
      penalty: 'Respawn takes 30 seconds'
    },
    circle: {
      speed: '+50% faster shrink',
      initialDelay: '-30 seconds before first shrink',
      finalSize: 'Smaller than standard'
    },
    gameplay: {
      playerCount: '20 duos (40 players)',
      matchDuration: '~8-10 minutes target'
    }
  },

  strategy: {
    impact: 'Team coordination critical',
    tips: [
      'Stay close to your partner',
      'Manage shared lives carefully',
      'Circle awareness even more important'
    ]
  },

  rewards: {
    firstWin: { item: 'Dynamic Duo Badge', oneTime: true },
    matchBonus: { xpBonus: '+15% XP for duo matches' }
  }
};
```

### Boss Rush

```typescript
const BOSS_RUSH_LTM = {
  id: 'ltm_boss_rush',
  name: 'Boss Rush',
  tagline: 'Unite against the monsters',
  description: 'PvE cooperative mode. Teams work together to defeat increasingly difficult waves of bosses.',

  rules: {
    mode: {
      type: 'PvE Cooperative',
      pvp: 'Disabled (friendly fire off)',
      objective: 'Survive all boss waves'
    },
    teams: {
      size: 'Squads of 4',
      matchmaking: 'Auto-fill enabled',
      total: '1-3 squads per instance'
    },
    waves: {
      count: 5,
      structure: [
        { wave: 1, boss: 'Sea Serpent', difficulty: 'Easy' },
        { wave: 2, boss: 'Ghost Captain', difficulty: 'Medium' },
        { wave: 3, boss: 'Kraken Spawn', difficulty: 'Medium-Hard' },
        { wave: 4, boss: 'Cursed Behemoth', difficulty: 'Hard' },
        { wave: 5, boss: 'The Kraken', difficulty: 'Very Hard' }
      ],
      intermission: '60 seconds between waves for loot/healing'
    },
    loot: {
      source: 'Boss drops and intermission chests',
      scaling: 'Better loot in later waves'
    }
  },

  mechanics: {
    bossAbilities: 'Each boss has unique attack patterns',
    teamDamage: 'Combined team damage defeats boss',
    revive: 'Teammates can revive downed players',
    wipe: 'If all players down, wave restarts'
  },

  rewards: {
    perWave: { gold: [100, 200, 300, 500, 1000] },
    completion: { item: 'Boss Slayer Title', oneTime: true },
    speedRun: { leaderboard: true, reward: 'Top times get exclusive cosmetic' }
  }
};
```

### Ability Draft

```typescript
const ABILITY_DRAFT_LTM = {
  id: 'ltm_ability_draft',
  name: 'Ability Draft',
  tagline: 'Random loadout, adapt and overcome',
  description: 'Start with randomized abilities. No ability drops in-match. Master what you\'re given.',

  rules: {
    abilities: {
      assignment: 'Random at match start',
      count: 3,
      rarity: 'Random (Common to Epic)',
      changeable: false,
      drops: 'Abilities do not spawn in chests'
    },
    loot: {
      content: 'Gold, consumables, equipment only',
      noAbilities: true
    },
    reroll: {
      option: 'One free reroll at match start',
      cost: 'Additional rerolls cost 50 Gold'
    },
    gameplay: {
      playerCount: 40,
      circleSpeed: 'Standard'
    }
  },

  strategy: {
    impact: 'Tests adaptability and game knowledge',
    tips: [
      'Learn how to play any ability combination',
      'Adapt strategy to your loadout',
      'Synergies you\'d never choose can be powerful'
    ]
  },

  rewards: {
    firstWin: { item: 'Wildcard Title', oneTime: true },
    variety: { achievement: 'Win with 20 different ability combinations' }
  }
};
```

### Mini Royale

```typescript
const MINI_ROYALE_LTM = {
  id: 'ltm_mini_royale',
  name: 'Mini Royale',
  tagline: 'Small map, big action',
  description: 'Condensed battle on a smaller island. Fewer players, faster matches, constant action.',

  rules: {
    map: {
      size: '50% of standard map',
      variant: 'Mini island with subset of POIs',
      pois: '4-5 main locations'
    },
    players: {
      count: 20,
      spawnDensity: 'Higher than standard'
    },
    circle: {
      initialSize: 'Covers entire mini map',
      shrinkSpeed: '+75% faster',
      finalPhase: 'Reached in ~5 minutes'
    },
    loot: {
      density: '+50% more chests',
      quality: 'Standard distribution'
    },
    gameplay: {
      matchDuration: '5-7 minutes target',
      pace: 'Extremely fast'
    }
  },

  strategy: {
    impact: 'Non-stop action from landing',
    tips: [
      'Every drop is contested',
      'Always be moving toward center',
      'Quick decisions required'
    ]
  },

  rewards: {
    firstWin: { item: 'Speed Demon Trail', oneTime: true },
    dailyBonus: { goldBonus: '+50% Gold (shorter matches)' }
  }
};
```

---

## Additional LTM Concepts

### Future LTM Ideas

```typescript
const FUTURE_LTMS = {
  snipersParadise: {
    name: 'Snipers Paradise',
    concept: 'Long-range abilities only, open map variant',
    targetRelease: 'Month 2'
  },

  lowGravity: {
    name: 'Sky Pirates',
    concept: 'Reduced gravity, extended jumps, aerial combat',
    targetRelease: 'Month 3'
  },

  oneInTheChamber: {
    name: 'One Shot',
    concept: 'Single powerful ability, one-hit eliminations',
    targetRelease: 'Month 3'
  },

  infection: {
    name: 'Cursed Tide',
    concept: 'Eliminated players become ghosts hunting survivors',
    targetRelease: 'Month 4'
  },

  arsenal: {
    name: 'Arsenal',
    concept: 'Abilities cycle on elimination, race to final ability',
    targetRelease: 'Month 4'
  },

  fogOfWar: {
    name: 'Fog of War',
    concept: 'Reduced visibility, minimap disabled',
    targetRelease: 'Month 5'
  },

  protectTheVIP: {
    name: 'Protect the Captain',
    concept: 'Team mode where one player is the VIP',
    targetRelease: 'Month 6'
  }
};
```

---

## LTM Rotation System

### Schedule

```typescript
const LTM_ROTATION = {
  cadence: {
    active: '2-3 days per LTM',
    downtime: '0-1 days between LTMs (minimal)',
    overlap: 'New LTM starts as old one ends'
  },

  schedule: {
    weekly: [
      { day: 'Monday', ltm: 'Rotates', duration: '72 hours' },
      { day: 'Thursday', ltm: 'Rotates', duration: '72 hours' },
      { day: 'Weekend', ltm: 'Popular LTM or new', duration: '48-72 hours' }
    ]
  },

  selection: {
    factors: [
      'Time since last appearance',
      'Player popularity (engagement data)',
      'Seasonal relevance',
      'Balance with current meta',
      'Event tie-ins'
    ],
    avoidance: 'Same LTM not twice in 2 weeks (unless popular)'
  },

  specialRotations: {
    newLtmLaunch: 'Extended duration (4-5 days) for new LTMs',
    eventLtm: 'Event-themed LTM for duration of event',
    playerChoice: 'Community votes on weekend LTM monthly'
  }
};
```

### Availability Display

```typescript
const LTM_AVAILABILITY = {
  mainMenu: {
    display: 'LTM button prominently shown when active',
    indicator: 'Special border/glow effect',
    countdown: 'Time remaining until rotation'
  },

  modeSelect: {
    position: 'Featured position (top or highlighted)',
    info: 'LTM rules summary on hover/tap',
    rewards: 'Show LTM-specific rewards'
  },

  notification: {
    newLtm: 'Push notification when new LTM starts',
    endingSoon: 'Reminder when LTM ending in 6 hours',
    favorite: 'Notify when favorited LTM returns'
  }
};
```

---

## LTM Matchmaking

### Queue System

```typescript
const LTM_MATCHMAKING = {
  queueType: {
    separate: 'LTM has dedicated queue',
    concurrent: 'Can queue for LTM while in standard queue',
    priority: 'First available match wins'
  },

  parameters: {
    skillBased: {
      enabled: 'Soft SBMM (wider skill range than ranked)',
      purpose: 'Faster queue times, more casual',
      exceptions: 'Boss Rush and PvE modes no SBMM'
    },
    regionPriority: {
      local: 'Prefer local region',
      expansion: 'Expand to nearby regions if needed',
      global: 'Global for low-population LTMs'
    }
  },

  targetTimes: {
    peakHours: '< 30 seconds',
    offPeak: '< 90 seconds',
    lowPopulation: '< 180 seconds with bots fill'
  },

  botFill: {
    enabled: true,
    trigger: 'Queue time exceeds 90 seconds',
    ratio: 'Up to 50% bots in LTM matches',
    quality: 'Medium difficulty bots'
  }
};
```

---

## LTM Rewards

### Reward Types

```typescript
const LTM_REWARDS = {
  firstWin: {
    type: 'One-time reward per LTM rotation',
    examples: ['Emotes', 'Trails', 'Titles', 'Badges'],
    reset: 'Resets each time LTM appears',
    tracking: 'First win tracked per rotation, not lifetime'
  },

  participation: {
    type: 'Per-match bonus',
    examples: ['+10-25% XP bonus', '+Gold per elimination'],
    purpose: 'Incentivize trying LTM'
  },

  challenges: {
    type: 'LTM-specific quests',
    examples: ['Win 3 Mini Royale matches', 'Get 10 eliminations in Chaos Mode'],
    rewards: 'Gold, XP, event currency'
  },

  exclusive: {
    type: 'Rare LTM cosmetics',
    acquisition: 'Complete specific LTM achievements',
    examples: ['Play 50 Boss Rush matches for exclusive banner'],
    rarity: 'Shows dedication to specific mode'
  },

  leaderboards: {
    enabled: 'Select LTMs (Boss Rush, Mini Royale)',
    rewards: 'Weekly top performers get bonus rewards',
    reset: 'Weekly reset'
  }
};
```

### Reward Progression

```typescript
const LTM_PROGRESSION = {
  perRotation: {
    firstGame: '50 Gold bonus',
    firstWin: 'Exclusive reward (varies)',
    threeWins: 'XP boost token',
    fiveWins: 'Rare chest'
  },

  lifetime: {
    tracking: 'Total games and wins per LTM type',
    milestones: {
      games10: 'LTM Explorer Badge',
      wins25: 'LTM Specialist Title',
      wins100: 'LTM Master Title'
    }
  }
};
```

---

## LTM Configuration

### Backend Configuration

```typescript
const LTM_CONFIG_SCHEMA = {
  ltm: {
    id: 'string (unique)',
    name: 'string',
    description: 'string',
    tagline: 'string',
    icon: 'string (asset path)',
    enabled: 'boolean',
    version: 'number'
  },

  rules: {
    playerCount: 'number',
    teamSize: 'number',
    circleConfig: {
      initialDelay: 'number (seconds)',
      shrinkSpeed: 'number (multiplier)',
      finalRadius: 'number'
    },
    abilityRules: {
      restrictions: 'array<abilityId>',
      overrides: 'object<abilityId, config>',
      spawnRules: 'object'
    },
    lootRules: {
      spawnMultiplier: 'number',
      restrictions: 'array<itemType>',
      overrides: 'object'
    },
    specialMechanics: 'array<mechanicConfig>'
  },

  matchmaking: {
    sbmmEnabled: 'boolean',
    sbmmStrength: 'number (0-1)',
    botFillEnabled: 'boolean',
    maxBotRatio: 'number',
    regionExpansion: 'array<region>'
  },

  schedule: {
    startTime: 'timestamp',
    endTime: 'timestamp',
    timezone: 'UTC'
  },

  rewards: {
    firstWinReward: 'Reward',
    participationBonus: 'object',
    challenges: 'array<Challenge>'
  }
};
```

### Admin Tools

```typescript
const LTM_ADMIN = {
  dashboard: {
    activeStatus: 'Current LTM and time remaining',
    upcoming: 'Scheduled LTM rotation',
    quickActions: ['Enable/Disable', 'Extend duration', 'Emergency end']
  },

  creation: {
    wizard: 'Step-by-step LTM configuration',
    templates: 'Clone existing LTM as starting point',
    preview: 'Test LTM in development environment'
  },

  scheduling: {
    calendar: 'Visual calendar of LTM rotation',
    dragDrop: 'Reschedule by dragging',
    conflicts: 'Warning if scheduling issues'
  },

  analytics: {
    perLtm: [
      'Participation rate',
      'Queue times',
      'Match completion rate',
      'Player feedback',
      'Win rate distribution'
    ],
    comparison: 'Compare LTM performance over time'
  }
};
```

---

## Technical Implementation

### Mode Switching

```typescript
const MODE_SWITCHING = {
  client: {
    ruleDownload: 'LTM rules downloaded on mode select',
    caching: 'Cache current LTM config',
    validation: 'Verify config version before match'
  },

  server: {
    modeDetection: 'Server determines mode from queue type',
    ruleEnforcement: 'Server enforces all LTM rules',
    mixedLobby: 'Never mix standard and LTM players'
  },

  gameRules: {
    loading: 'Rules loaded at match initialization',
    application: 'Rules applied through modifier system',
    validation: 'Continuous validation during match'
  }
};
```

### Rule Modifiers

```typescript
const RULE_MODIFIERS = {
  system: {
    description: 'Modular system for applying LTM rules',
    architecture: 'Composable modifiers that stack'
  },

  modifierTypes: {
    abilityRestriction: {
      type: 'AbilityRestrictionModifier',
      params: ['disabledAbilities', 'enabledAbilities']
    },
    lootModification: {
      type: 'LootModifier',
      params: ['spawnRates', 'rarityOverrides', 'restrictions']
    },
    circleModification: {
      type: 'CircleModifier',
      params: ['speedMultiplier', 'sizeOverrides']
    },
    playerModification: {
      type: 'PlayerModifier',
      params: ['healthMultiplier', 'damageMultiplier', 'movementModifier']
    },
    teamModification: {
      type: 'TeamModifier',
      params: ['teamSize', 'friendlyFire', 'respawnRules']
    }
  },

  application: {
    order: 'Modifiers applied in priority order',
    conflicts: 'Later modifiers override earlier',
    validation: 'Invalid combinations rejected at config time'
  }
};
```

---

## Testing Requirements

```typescript
const LTM_TESTING = {
  perLtm: [
    'All rules enforced correctly',
    'Restricted abilities cannot be used',
    'Loot drops match configuration',
    'Circle behaves as configured',
    'Rewards granted appropriately'
  ],

  matchmaking: [
    'LTM queue separate from standard',
    'SBMM settings applied correctly',
    'Bot fill works as expected',
    'Region expansion functions'
  ],

  integration: [
    'LTM and standard can run simultaneously',
    'Player can switch between modes',
    'Statistics tracked separately',
    'No interference with ranked'
  ],

  rotation: [
    'LTM activates at scheduled time',
    'LTM deactivates at end time',
    'In-progress matches complete normally',
    'Queue disabled before end time'
  ],

  edge: [
    'Player in LTM queue when mode ends',
    'Mode configuration update mid-rotation',
    'Network failure during LTM match',
    'Bot behavior in LTM-specific rules'
  ]
};
```

---

## Player Communication

### LTM Information

```typescript
const LTM_COMMUNICATION = {
  inGame: {
    rulesDisplay: {
      location: 'Mode select and pre-match',
      content: ['Mode name', 'Key rules summary', 'Tips'],
      detail: 'Expandable for full rules'
    },
    notification: {
      start: 'Push notification when new LTM starts',
      ending: 'In-game banner when LTM ending soon'
    }
  },

  marketing: {
    newLtm: {
      trailer: 'Short video showcasing new LTM',
      socialPost: 'Announcement with key features',
      blogPost: 'Detailed rules explanation'
    },
    rotation: {
      weeklyPreview: 'Announce upcoming week\'s LTM schedule',
      reminder: 'Social post when popular LTM returns'
    }
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial Limited-Time Modes specification |
