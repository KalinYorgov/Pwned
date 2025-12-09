# Community Challenges Specification

## Document Information
- **Task ID:** LIVE-007
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the community challenge system for Plunderstorm Mobile, where all players collectively work toward server-wide goals. Community challenges foster a sense of shared purpose, encourage engagement, and create memorable community moments.

---

## Challenge Types

### Collective Challenges

```typescript
const COLLECTIVE_CHALLENGES = {
  description: 'All players contribute to a shared goal',

  structure: {
    goal: 'Single target number for entire community',
    contribution: 'Every player action counts toward total',
    reward: 'All active participants receive reward when complete'
  },

  examples: [
    {
      id: 'community_elims_1',
      name: 'The Great Hunt',
      description: 'Community: Eliminate 1,000,000 enemies',
      goal: 1000000,
      metric: 'total_eliminations',
      duration: '7 days',
      reward: { type: 'all_participants', items: ['500 Gold', 'Hunter Badge'] }
    },
    {
      id: 'community_chests_1',
      name: 'Treasure Hunters United',
      description: 'Community: Open 500,000 treasure chests',
      goal: 500000,
      metric: 'chests_opened',
      duration: '7 days',
      reward: { type: 'all_participants', items: ['Treasure Hunter Emote'] }
    },
    {
      id: 'community_wins_1',
      name: 'Victory Armada',
      description: 'Community: Win 100,000 matches',
      goal: 100000,
      metric: 'matches_won',
      duration: '7 days',
      reward: { type: 'all_participants', items: ['Champion Banner'] }
    },
    {
      id: 'community_damage_1',
      name: 'Firepower Festival',
      description: 'Community: Deal 1,000,000,000 damage',
      goal: 1000000000,
      metric: 'total_damage_dealt',
      duration: '7 days',
      reward: { type: 'all_participants', items: ['Explosive Trail'] }
    }
  ]
};
```

### Milestone Challenges

```typescript
const MILESTONE_CHALLENGES = {
  description: 'Incremental goals with rewards at each milestone',

  structure: {
    milestones: '4-5 checkpoints (25%, 50%, 75%, 100%, stretch)',
    rewards: 'Escalating rewards at each milestone',
    visibility: 'Progress bar with milestone markers'
  },

  example: {
    id: 'milestone_event_1',
    name: 'Race to Riches',
    description: 'Community gold collection challenge',
    metric: 'gold_collected',
    duration: '14 days',

    milestones: [
      {
        name: 'Bronze Hoard',
        target: 10000000,
        percentage: 25,
        reward: { item: '100 Gold Bonus', type: 'all_active' }
      },
      {
        name: 'Silver Hoard',
        target: 25000000,
        percentage: 50,
        reward: { item: 'Silver Coin Emote', type: 'all_active' }
      },
      {
        name: 'Gold Hoard',
        target: 50000000,
        percentage: 75,
        reward: { item: 'Gold Coin Trail', type: 'all_active' }
      },
      {
        name: 'Legendary Hoard',
        target: 75000000,
        percentage: 100,
        reward: { item: 'Treasure King Title', type: 'all_active' }
      },
      {
        name: 'Mythic Hoard',
        target: 100000000,
        percentage: 'stretch',
        reward: { item: 'Golden Crown Cosmetic', type: 'all_active' }
      }
    ]
  }
};
```

### Competitive Community Challenges

```typescript
const COMPETITIVE_CHALLENGES = {
  description: 'Individual contributions tracked with leaderboard',

  structure: {
    goal: 'Community goal + individual leaderboard',
    leaderboard: 'Top contributors ranked',
    rewards: 'Community reward + placement rewards'
  },

  example: {
    id: 'competitive_boss_1',
    name: 'Kraken Hunters',
    description: 'Community vs Boss - Defeat the Kraken 10,000 times',
    communityGoal: 10000,
    metric: 'kraken_kills',
    duration: '10 days',

    communityReward: {
      completion: 'All participants: Kraken Slayer Badge',
      bonus: 'If completed early: +200 Gold bonus'
    },

    leaderboardRewards: {
      top1: 'Legendary Kraken Hunter Title + Exclusive Outfit',
      top10: 'Epic Kraken Weapon Skin',
      top100: 'Rare Kraken Trail',
      top1000: 'Kraken Emote',
      participant: '50 Gold'
    }
  }
};
```

### Cooperative Challenges

```typescript
const COOPERATIVE_CHALLENGES = {
  description: 'Party/squad-based community challenges',

  structure: {
    requirement: 'Must be in party to contribute',
    tracking: 'Party actions count toward community goal',
    bonus: 'Extra contribution weight for parties'
  },

  example: {
    id: 'coop_party_1',
    name: 'Stronger Together',
    description: 'Community: Win 25,000 matches as a party',
    requirement: 'Party of 2+ players',
    goal: 25000,
    metric: 'party_wins',
    duration: '7 days',

    rewards: {
      community: 'Party Power Badge (all participants)',
      partyBonus: '+50% Gold for party matches during challenge'
    }
  }
};
```

---

## Progress Tracking

### Real-Time Display

```typescript
const PROGRESS_DISPLAY = {
  inGame: {
    mainMenu: {
      widget: 'Community Challenge banner on main menu',
      content: ['Challenge name', 'Progress bar', 'Current/Goal', 'Time remaining'],
      interaction: 'Tap to expand details'
    },
    lobbyScreen: {
      widget: 'Compact progress indicator',
      content: ['Progress percentage', 'Goal status'],
      animation: 'Pulse when milestone reached'
    },
    postMatch: {
      widget: 'Contribution summary',
      content: ['Your contribution this match', 'Total progress', 'Milestone status'],
      celebration: 'Animation when milestone unlocked'
    }
  },

  detailsPanel: {
    access: 'Tap community challenge widget',
    content: [
      'Full challenge description',
      'Detailed progress (exact numbers)',
      'All milestones and their status',
      'Time remaining',
      'Your personal contribution',
      'Leaderboard (if competitive)',
      'Rewards preview'
    ]
  }
};
```

### Progress Calculation

```typescript
const PROGRESS_CALCULATION = {
  aggregation: {
    method: 'Real-time aggregation from match results',
    delay: 'Up to 1 minute for display update',
    accuracy: 'Server authoritative, client displays estimate'
  },

  contribution: {
    perPlayer: {
      tracking: 'Each player contribution tracked individually',
      validation: 'Server validates all contributions',
      antiFraud: 'Anomaly detection for impossible contributions'
    }
  },

  synchronization: {
    clientUpdate: 'Every 30 seconds while in-game',
    serverPush: 'On milestone completion',
    sessionEnd: 'Full sync on match end'
  }
};
```

### Live Counter

```typescript
const LIVE_COUNTER = {
  display: {
    format: 'Animated counter showing progress',
    updates: 'Smooth animation as number increases',
    milestoneEffect: 'Special celebration at milestone'
  },

  implementation: {
    backend: {
      storage: 'Redis counter with atomic increments',
      aggregation: 'Per-shard counters merged to global',
      persistence: 'Periodic snapshot to database'
    },
    frontend: {
      polling: 'Poll every 30 seconds',
      interpolation: 'Smooth animation between updates',
      notification: 'Push notification on milestone'
    }
  }
};
```

---

## Participation Requirements

### Active Participation

```typescript
const PARTICIPATION_RULES = {
  eligibility: {
    requirement: 'Must contribute at least once to challenge',
    minimumAction: 'Complete 1 match during challenge period',
    trackingStart: 'Contribution counted from challenge start'
  },

  activeStatus: {
    definition: 'Player who has logged in and played during challenge',
    windowforRewards: 'Must have contributed before challenge completion',
    lateJoiners: 'Can still earn rewards if challenge not yet complete'
  },

  afkPrevention: {
    detection: 'AFK matches do not count contributions',
    minimum: 'Must deal damage or interact with game',
    enforcement: 'Server-side validation of valid contributions'
  }
};
```

### Contribution Weighting

```typescript
const CONTRIBUTION_WEIGHTING = {
  standard: {
    description: 'Equal weight per action',
    example: '1 elimination = 1 point toward goal'
  },

  bonusMultipliers: {
    firstTimeContribution: {
      multiplier: 2.0,
      duration: 'First match only',
      purpose: 'Encourage initial participation'
    },
    dailyBonus: {
      multiplier: 1.5,
      duration: 'First match each day',
      purpose: 'Drive daily engagement'
    },
    partyBonus: {
      multiplier: 1.25,
      requirement: 'Playing in party of 2+',
      purpose: 'Encourage social play'
    }
  },

  caps: {
    perMatch: 'No cap (all valid actions count)',
    perDay: 'No hard cap (soft diminishing returns optional)',
    perChallenge: 'No cap on total contribution'
  }
};
```

---

## Reward Distribution

### Reward Timing

```typescript
const REWARD_TIMING = {
  milestoneRewards: {
    trigger: 'Immediately when milestone reached',
    delivery: 'Added to player inventory within 1 minute',
    notification: 'Push notification + in-game popup'
  },

  completionRewards: {
    trigger: 'When challenge goal is met',
    delivery: 'Distributed to all eligible players',
    window: 'Available to claim for 7 days after challenge',
    notification: 'Major celebration + push notification'
  },

  leaderboardRewards: {
    trigger: 'When challenge ends (time-based)',
    calculation: 'Final leaderboard snapshot at end time',
    delivery: 'Within 24 hours of challenge end',
    notification: 'Personal placement notification'
  }
};
```

### Reward Claiming

```typescript
const REWARD_CLAIMING = {
  automaticClaim: {
    enabled: true,
    behavior: 'Rewards auto-added to inventory',
    notification: 'Player notified of new rewards'
  },

  claimUI: {
    location: 'Community Challenge results screen',
    display: 'All earned rewards with claim animation',
    history: 'Past challenge rewards viewable'
  },

  missedRewards: {
    policy: 'Unclaimed rewards available for 7 days',
    reminder: 'Push notification if rewards unclaimed',
    expiration: 'Rewards expire after grace period'
  }
};
```

---

## Challenge Scheduling

### Challenge Cadence

```typescript
const CHALLENGE_CADENCE = {
  regular: {
    frequency: '1-2 challenges active at any time',
    duration: '5-14 days typical',
    gap: 'Minimal gap between challenges'
  },

  types: {
    weekly: {
      duration: '7 days',
      start: 'Monday 00:00 UTC',
      end: 'Sunday 23:59 UTC'
    },
    biweekly: {
      duration: '14 days',
      usage: 'Larger milestone challenges'
    },
    event: {
      duration: 'Tied to event duration',
      usage: 'Seasonal event community challenges'
    }
  },

  overlap: {
    allowed: true,
    maximum: 2,
    types: 'Different types can overlap (e.g., weekly + event)'
  }
};
```

### Challenge Configuration

```typescript
const CHALLENGE_CONFIG = {
  creation: {
    tool: 'Admin dashboard for challenge creation',
    scheduling: 'Set start/end times in advance',
    preview: 'Preview challenge before activation',
    editing: 'Can edit future challenges, not active ones'
  },

  parameters: {
    required: [
      'Challenge name',
      'Description',
      'Goal metric',
      'Target value',
      'Duration',
      'Rewards'
    ],
    optional: [
      'Milestones',
      'Leaderboard enabled',
      'Bonus multipliers',
      'Eligibility requirements'
    ]
  },

  validation: {
    goalReasonable: 'Target achievable based on player base',
    rewardsSet: 'All milestone and completion rewards configured',
    noConflict: 'No conflicting challenges on same metric'
  }
};
```

---

## Analytics and Monitoring

### Real-Time Monitoring

```typescript
const CHALLENGE_MONITORING = {
  dashboard: {
    metrics: [
      'Current progress vs expected pace',
      'Unique participants',
      'Contribution rate (per hour)',
      'Top contributors',
      'Participation by region'
    ],
    alerts: [
      'Progress significantly behind pace',
      'Progress significantly ahead (adjust future goals)',
      'Anomaly detection (cheating)',
      'Technical issues affecting tracking'
    ]
  },

  liveOpsActions: {
    adjustGoal: {
      when: 'Goal clearly unachievable or too easy',
      action: 'Adjust target (with player communication)',
      limitation: 'Only within first 25% of challenge'
    },
    extendDuration: {
      when: 'Close to goal but time running out',
      action: 'Extend deadline',
      communication: 'Announce extension'
    },
    bonusEvent: {
      when: 'Engagement lower than expected',
      action: 'Add bonus multiplier period',
      example: '2x contribution for final 24 hours'
    }
  }
};
```

### Post-Challenge Analysis

```typescript
const POST_CHALLENGE_ANALYSIS = {
  metrics: {
    engagement: {
      totalParticipants: 'Unique players who contributed',
      participationRate: 'Participants / DAU',
      contributionDistribution: 'Contributions per player breakdown',
      peakActivity: 'Highest activity periods'
    },
    completion: {
      goalReached: 'Yes/No',
      timeToComplete: 'Days to reach goal (if reached)',
      milestonesReached: 'Number of milestones achieved',
      finalProgress: 'Final progress vs goal'
    },
    rewards: {
      rewardsClaimed: 'Number of rewards distributed',
      rewardValue: 'Total value of rewards given',
      claimRate: 'Percentage of eligible who claimed'
    }
  },

  learnings: {
    questions: [
      'Was the goal appropriately difficult?',
      'Did participation meet expectations?',
      'Were rewards motivating?',
      'Any technical issues?',
      'Player sentiment?'
    ],
    adjustments: 'Apply learnings to future challenge calibration'
  }
};
```

---

## Communication

### Announcement

```typescript
const CHALLENGE_ANNOUNCEMENTS = {
  preChallenge: {
    timing: '24-48 hours before start',
    content: [
      'Challenge name and description',
      'Start time',
      'Goal and milestones',
      'Rewards preview'
    ],
    channels: ['In-game news', 'Push notification', 'Social media']
  },

  challengeStart: {
    timing: 'At challenge start time',
    content: ['Challenge is live!', 'Join now', 'Goal reminder'],
    channels: ['In-game popup', 'Push notification']
  },

  milestoneReached: {
    timing: 'Immediately on milestone',
    content: [
      'Milestone reached!',
      'Next milestone goal',
      'Rewards distributed'
    ],
    channels: ['In-game celebration', 'Push notification']
  },

  challengeComplete: {
    timing: 'On goal completion',
    content: [
      'Challenge complete!',
      'Community achievement',
      'Rewards available',
      'Stats summary'
    ],
    channels: ['In-game celebration', 'Push notification', 'Social media']
  },

  challengeFailed: {
    timing: 'On challenge end (if not completed)',
    content: [
      'Challenge ended',
      'Progress achieved',
      'Milestone rewards still given',
      'Better luck next time'
    ],
    approach: 'Positive framing, celebrate effort'
  }
};
```

### Summary Report

```typescript
const CHALLENGE_SUMMARY = {
  content: {
    overview: [
      'Challenge name and duration',
      'Final progress (X of Y)',
      'Completion status',
      'Total participants'
    ],
    yourContribution: [
      'Your total contribution',
      'Percentage of goal',
      'Rank (if leaderboard)',
      'Rewards earned'
    ],
    communityHighlights: [
      'Top contributors (names)',
      'Fun stats (peak hour, etc.)',
      'Community achievement unlocked'
    ]
  },

  display: {
    location: 'Dedicated results screen',
    duration: 'Available for 7 days after challenge',
    sharing: 'Share results to social media'
  }
};
```

---

## Technical Implementation

### Backend Architecture

```typescript
const BACKEND_ARCHITECTURE = {
  counters: {
    technology: 'Redis atomic counters',
    sharding: 'Per-region shards for low latency',
    aggregation: 'Global aggregation service',
    persistence: 'Periodic snapshots to PostgreSQL'
  },

  contribution: {
    ingestion: 'Match result processor',
    validation: 'Anti-cheat validation',
    recording: 'Player contribution ledger',
    deduplication: 'Idempotent contribution recording'
  },

  leaderboard: {
    technology: 'Redis sorted sets',
    update: 'Real-time on contribution',
    query: 'Efficient rank lookup'
  }
};
```

### Data Model

```typescript
const DATA_MODEL = {
  challenge: {
    id: 'string (UUID)',
    name: 'string',
    description: 'string',
    metric: 'string (enum)',
    goal: 'number',
    milestones: 'array<Milestone>',
    startTime: 'timestamp',
    endTime: 'timestamp',
    status: 'enum (scheduled, active, completed, failed)',
    currentProgress: 'number',
    rewards: 'array<Reward>',
    leaderboardEnabled: 'boolean'
  },

  milestone: {
    name: 'string',
    target: 'number',
    reward: 'Reward',
    reached: 'boolean',
    reachedAt: 'timestamp (nullable)'
  },

  contribution: {
    challengeId: 'string',
    playerId: 'string',
    matchId: 'string',
    amount: 'number',
    timestamp: 'timestamp',
    validated: 'boolean'
  },

  participation: {
    challengeId: 'string',
    playerId: 'string',
    totalContribution: 'number',
    firstContributionAt: 'timestamp',
    rewardsClaimed: 'array<string>'
  }
};
```

### API Endpoints

```typescript
const API_ENDPOINTS = {
  challenges: {
    getActive: 'GET /api/v1/challenges/active',
    getDetails: 'GET /api/v1/challenges/{id}',
    getProgress: 'GET /api/v1/challenges/{id}/progress',
    getMyContribution: 'GET /api/v1/challenges/{id}/my-contribution',
    getLeaderboard: 'GET /api/v1/challenges/{id}/leaderboard',
    claimRewards: 'POST /api/v1/challenges/{id}/claim'
  },

  responses: {
    challengeProgress: {
      challengeId: 'string',
      currentProgress: 'number',
      goal: 'number',
      percentComplete: 'number',
      milestonesReached: 'array<string>',
      timeRemaining: 'number (seconds)',
      participantCount: 'number'
    }
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Contributions tracked correctly across all metrics',
    'Progress updates in real-time',
    'Milestones trigger at correct thresholds',
    'Rewards distributed to all eligible players',
    'Leaderboard ranks correctly'
  ],

  scale: [
    'Handle 100,000+ concurrent contributors',
    'Counter accuracy under high load',
    'Leaderboard performance with millions of entries'
  ],

  edge: [
    'Player contributes at exact milestone threshold',
    'Challenge ends mid-match',
    'Player joins challenge on final day',
    'Network failure during contribution',
    'Clock sync issues across regions'
  ],

  antiCheat: [
    'Detect automated contribution attempts',
    'Validate contribution reasonableness',
    'Handle duplicate contribution attempts'
  ],

  recovery: [
    'Counter recovery after Redis failure',
    'Reward distribution retry on failure',
    'Progress recalculation capability'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial community challenges specification |
