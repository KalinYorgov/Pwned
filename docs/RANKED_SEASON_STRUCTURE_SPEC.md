# Ranked Season Structure Specification

## Document Information
- **Task ID:** LIVE-015
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the ranked season structure for Plunderstorm Mobile, defining season duration, reset mechanics, reward distribution, and the overall competitive cadence that keeps ranked play fresh and rewarding.

---

## Season Structure

### Season Duration

```typescript
const SEASON_DURATION = {
  length: {
    weeks: 8,
    days: 56,
    alignment: 'Aligned with Battle Pass seasons'
  },

  phases: {
    preseason: {
      duration: '3 days before season start',
      activities: [
        'Season preview announcements',
        'Rank reset processing',
        'Reward distribution from previous season'
      ],
      playAvailability: 'Ranked queue disabled'
    },
    earlyseason: {
      duration: 'Days 1-14',
      characteristics: [
        'Placement matches for most players',
        'High volatility in rankings',
        'Climbing opportunities'
      ]
    },
    midseason: {
      duration: 'Days 15-42',
      characteristics: [
        'Stable competition',
        'Players settling into appropriate ranks',
        'Main competitive period'
      ]
    },
    lateseason: {
      duration: 'Days 43-56',
      characteristics: [
        'Final push for rank goals',
        'Increased activity from deadline pressure',
        'Reward anticipation'
      ]
    },
    endofseason: {
      duration: 'Final 24 hours',
      activities: [
        'Last chance notifications',
        'Queue closes 2 hours before reset',
        'Final standings snapshot'
      ]
    }
  }
};
```

### Season Calendar

```typescript
const SEASON_CALENDAR = {
  year1: {
    season1: { start: 'Launch', end: 'Week 8', theme: 'Founding Season' },
    season2: { start: 'Week 9', end: 'Week 16', theme: 'TBD' },
    season3: { start: 'Week 17', end: 'Week 24', theme: 'TBD' },
    season4: { start: 'Week 25', end: 'Week 32', theme: 'TBD' },
    season5: { start: 'Week 33', end: 'Week 40', theme: 'TBD' },
    season6: { start: 'Week 41', end: 'Week 48', theme: 'TBD' }
  },

  naming: {
    format: 'Season {number}: {theme}',
    example: 'Season 1: Founding Season'
  },

  holidays: {
    adjustment: 'Seasons may extend 1-2 weeks around major holidays',
    communication: 'Announced 2+ weeks in advance'
  }
};
```

---

## Rank Reset System

### Soft Reset Algorithm

```typescript
const SOFT_RESET = {
  philosophy: {
    purpose: 'Provide fresh start while respecting skill',
    balance: 'Neither too aggressive nor too lenient',
    excitement: 'Create climbing opportunity for engaged players'
  },

  algorithm: {
    formula: 'New MMR = (Current MMR + Baseline) / 2',
    baseline: 1200,
    example: {
      diamondPlayer: {
        current: 2200,
        calculation: '(2200 + 1200) / 2 = 1700',
        newRank: 'Gold (but with Diamond skill ceiling)'
      },
      bronzePlayer: {
        current: 800,
        calculation: '(800 + 1200) / 2 = 1000',
        newRank: 'Silver (slight boost)'
      }
    }
  },

  rankMapping: {
    postReset: 'Visible rank set to correspond to new MMR',
    rp: 'RP reset to starting point of new rank',
    floor: 'No player below Bronze 1 after reset'
  }
};
```

### Placement Matches

```typescript
const PLACEMENT_MATCHES = {
  count: 10,

  mechanics: {
    rpChange: 'Hidden during placements',
    displayRank: 'Shows "Unranked" or previous rank badge',
    matchmaking: 'Based on hidden MMR from previous season (or new player default)'
  },

  calibration: {
    highVariance: 'MMR adjusts significantly during placements',
    winWeight: 'Wins have higher impact than losses',
    performance: 'Individual performance affects calibration'
  },

  completion: {
    rankReveal: 'Celebration showing new rank',
    startingRp: 'Place at appropriate RP for determined rank',
    protection: 'Cannot drop below placement rank for 5 games'
  },

  returning: {
    previousData: 'Uses previous season MMR as starting point',
    decayFactor: 'If inactive for multiple seasons, additional variance'
  }
};
```

---

## Season Rewards

### Reward Tiers

```typescript
const SEASON_REWARDS = {
  basedOn: 'Peak rank achieved during season',
  distribution: 'Distributed within 48 hours of season end',

  tiers: {
    bronze: {
      badge: 'Season X Bronze Badge',
      title: 'Bronze Sailor',
      gold: 500,
      doubloons: 0,
      cosmetic: null
    },
    silver: {
      badge: 'Season X Silver Badge',
      title: 'Silver Sailor',
      gold: 1000,
      doubloons: 25,
      cosmetic: 'Season themed spray'
    },
    gold: {
      badge: 'Season X Gold Badge',
      title: 'Gold Sailor',
      gold: 2000,
      doubloons: 50,
      cosmetic: 'Season themed trail'
    },
    platinum: {
      badge: 'Season X Platinum Badge',
      title: 'Platinum Sailor',
      gold: 3500,
      doubloons: 100,
      cosmetic: 'Season themed weapon skin'
    },
    diamond: {
      badge: 'Season X Diamond Badge',
      title: 'Diamond Sailor',
      gold: 5000,
      doubloons: 200,
      cosmetic: 'Season themed outfit (uncommon)'
    },
    master: {
      badge: 'Season X Master Badge',
      title: 'Master Sailor',
      gold: 7500,
      doubloons: 350,
      cosmetic: 'Season themed outfit (rare)'
    },
    pirateKing: {
      badge: 'Season X Pirate King Badge',
      title: 'Pirate King',
      gold: 10000,
      doubloons: 500,
      cosmetic: 'Exclusive Pirate King outfit (epic)'
    }
  }
};
```

### Top Player Rewards

```typescript
const TOP_PLAYER_REWARDS = {
  top100: {
    additional: [
      'Season X Top 100 Badge (animated)',
      'Season X Top 100 Title',
      'Exclusive animated profile frame',
      '1000 bonus Doubloons'
    ],
    leaderboard: 'Name displayed on permanent season leaderboard'
  },

  top10: {
    additional: [
      'Season X Top 10 Badge (animated, premium)',
      'Season X Elite Title',
      'Exclusive legendary outfit variant',
      '2500 bonus Doubloons'
    ],
    recognition: 'Featured in in-game Hall of Fame'
  },

  rank1: {
    additional: [
      'Season X Champion Badge (unique design)',
      'Champion Title (unique per season)',
      'Exclusive legendary effect',
      '5000 bonus Doubloons',
      'Custom profile frame'
    ],
    permanentRecognition: 'Name engraved in Hall of Champions'
  }
};
```

### Reward Exclusivity

```typescript
const REWARD_EXCLUSIVITY = {
  badges: {
    exclusivity: 'Never return',
    purpose: 'Permanent record of peak rank per season',
    display: 'Show which seasons player achieved rank'
  },

  titles: {
    exclusivity: 'Never return',
    reUsable: 'Can equip old season titles anytime',
    format: 'S{X} {Rank} (e.g., S1 Diamond Sailor)'
  },

  cosmetics: {
    exclusivity: 'Never return in exact form',
    variants: 'Similar themes may return with variations',
    prestige: 'Early season cosmetics most valuable'
  }
};
```

---

## Season Communication

### Pre-Season

```typescript
const PRESEASON_COMMUNICATION = {
  twoWeeksBefore: {
    content: [
      'Season end date reminder',
      'Current rank and potential rewards',
      'Last chance to climb messaging'
    ],
    channels: ['In-game banner', 'Push notification']
  },

  oneWeekBefore: {
    content: [
      'Next season preview',
      'New season rewards showcase',
      'Season reset explanation'
    ],
    channels: ['In-game news', 'Social media', 'Email']
  },

  finalDays: {
    content: [
      'Final push encouragement',
      'Queue closing time',
      'Reward distribution timeline'
    ],
    channels: ['Push notification', 'In-game popup']
  }
};
```

### Season Start

```typescript
const SEASON_START_COMMUNICATION = {
  announcement: {
    content: [
      'New season is live!',
      'Season theme and duration',
      'Reward preview',
      'Placement match call to action'
    ],
    channels: ['All channels']
  },

  inGame: {
    popup: 'Season start celebration',
    tutorial: 'Placement match explanation for new players',
    banner: 'Season countdown and theme'
  }
};
```

### During Season

```typescript
const MIDSEASON_COMMUNICATION = {
  weekly: {
    leaderboard: 'Weekly leaderboard update',
    topPlayers: 'Spotlight on top performers',
    tips: 'Ranked tips and meta insights'
  },

  personalProgress: {
    reminders: 'Progress toward next rank',
    peakRank: 'Peak rank achievement notifications',
    streaks: 'Win streak and milestone celebrations'
  },

  midseasonEvent: {
    optional: 'Mid-season ranked event (double RP weekend)',
    purpose: 'Re-engage players, create excitement'
  }
};
```

---

## Season History

### Player History

```typescript
const PLAYER_HISTORY = {
  access: 'Profile > Ranked > Season History',

  perSeason: {
    display: [
      'Season name and dates',
      'Peak rank achieved',
      'Final rank',
      'Games played',
      'Win rate',
      'Rewards earned'
    ]
  },

  summary: {
    allTime: [
      'Highest peak rank ever',
      'Total seasons played',
      'Total ranked games',
      'Overall win rate'
    ]
  },

  badges: {
    showcase: 'All season badges visible in chronological order',
    missing: 'Placeholder for seasons not played'
  }
};
```

### Leaderboard History

```typescript
const LEADERBOARD_HISTORY = {
  permanent: {
    storage: 'All season leaderboards preserved',
    access: 'Leaderboards > Past Seasons',
    content: 'Top 1000 per season'
  },

  hallOfFame: {
    top10: 'Permanent display of each season\'s top 10',
    champion: 'Special recognition for #1 each season',
    location: 'Dedicated in-game Hall of Fame screen'
  }
};
```

---

## Special Season Events

### Double RP Events

```typescript
const DOUBLE_RP_EVENTS = {
  midseason: {
    timing: 'Weekend during week 4-5',
    duration: '48-72 hours',
    bonus: '2x RP gains (losses unchanged)',
    purpose: 'Drive engagement, help climbers'
  },

  lateseason: {
    timing: 'Final weekend before season end',
    duration: '48 hours',
    bonus: '1.5x RP gains',
    purpose: 'Final push excitement'
  },

  communication: {
    advance: 'Announce 1 week before',
    reminder: 'Push notification when event starts',
    countdown: 'In-game timer showing remaining time'
  }
};
```

### Season Challenges

```typescript
const SEASON_CHALLENGES = {
  weekly: {
    example: 'Win 10 ranked matches this week',
    reward: 'Bonus RP + Gold',
    purpose: 'Maintain engagement throughout season'
  },

  seasonal: {
    example: 'Reach Gold rank this season',
    reward: 'Exclusive banner or spray',
    tiered: 'Multiple tiers for different goals'
  },

  community: {
    example: 'Community plays 10 million ranked matches',
    reward: 'All participants get bonus rewards',
    tracking: 'Community progress visible'
  }
};
```

---

## Technical Implementation

### Season Configuration

```typescript
const SEASON_CONFIG = {
  schema: {
    id: 'string (e.g., "season_1")',
    number: 'number',
    name: 'string',
    theme: 'string',
    startTime: 'timestamp',
    endTime: 'timestamp',
    status: 'enum (upcoming, active, ended)',
    resetConfig: {
      algorithm: 'soft_reset',
      baseline: 1200
    },
    rewards: 'SeasonRewardConfig[]',
    specialEvents: 'SpecialEventConfig[]'
  },

  management: {
    creation: 'Admin creates season in advance',
    scheduling: 'Automatic activation/deactivation',
    extension: 'Manual extension if needed',
    rewards: 'Trigger reward distribution on end'
  }
};
```

### Data Model

```typescript
const DATA_MODEL = {
  season: {
    id: 'string',
    number: 'number',
    name: 'string',
    theme: 'string',
    startTime: 'timestamp',
    endTime: 'timestamp',
    status: 'enum'
  },

  playerSeasonRecord: {
    playerId: 'string',
    seasonId: 'string',
    currentRank: 'Rank',
    currentRp: 'number',
    peakRank: 'Rank',
    peakRp: 'number',
    gamesPlayed: 'number',
    wins: 'number',
    losses: 'number',
    placementComplete: 'boolean',
    placementGames: 'number',
    rewardsDistributed: 'boolean'
  },

  seasonLeaderboard: {
    seasonId: 'string',
    entries: 'array<{playerId, rank, rp, games}>',
    snapshotTime: 'timestamp'
  }
};
```

### Reward Distribution

```typescript
const REWARD_DISTRIBUTION = {
  timing: {
    trigger: 'Season end time reached',
    processing: 'Begin within 1 hour of end',
    completion: 'All rewards distributed within 48 hours'
  },

  process: {
    snapshot: 'Final leaderboard snapshot',
    calculation: 'Calculate rewards per player based on peak rank',
    distribution: 'Batch process reward grants',
    notification: 'Notify players of rewards'
  },

  verification: {
    integrity: 'Verify peak rank records',
    antiFraud: 'Check for boosting/exploitation',
    manual: 'Flag suspicious accounts for review'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  seasonal: [
    'Season activates at scheduled time',
    'Season deactivates at end time',
    'Queue properly closes before end',
    'Reset algorithm applies correctly'
  ],

  rewards: [
    'Rewards match peak rank achieved',
    'All reward items granted correctly',
    'Top player rewards distributed',
    'Notification sent on reward grant'
  ],

  placements: [
    'Placement match count correct',
    'Calibration affects rank appropriately',
    'Rank reveal celebration displays',
    'Protection period functions'
  ],

  history: [
    'Season history recorded accurately',
    'Leaderboards preserved',
    'Previous season data accessible',
    'Hall of Fame updates'
  ],

  edge: [
    'Player in match when season ends',
    'Player mid-placement when season ends',
    'Multiple seasons with no activity',
    'Account created mid-season'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial ranked season structure specification |
