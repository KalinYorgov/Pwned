# Leaderboards Service Specification

## Document Information
- **Task ID:** BACK-014
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the leaderboards service for Plunderstorm Mobile, providing global, regional, friends, and time-based leaderboards to drive competitive engagement. The system handles real-time updates, anti-cheat validation, and efficient querying at scale.

---

## Leaderboard Types

### Leaderboard Categories

```typescript
const LEADERBOARD_TYPES = {
  global: {
    name: 'Global Leaderboards',
    scope: 'All players worldwide',
    displayLimit: 100, // Top 100 shown
    categories: [
      {
        id: 'ranked_rating',
        name: 'Ranked Rating',
        metric: 'Current MMR/RP',
        sort: 'descending',
        tiebreaker: 'wins'
      },
      {
        id: 'total_wins',
        name: 'Total Wins',
        metric: 'Lifetime wins',
        sort: 'descending',
        tiebreaker: 'win_rate'
      },
      {
        id: 'total_kills',
        name: 'Total Kills',
        metric: 'Lifetime eliminations',
        sort: 'descending',
        tiebreaker: 'kills_per_match'
      },
      {
        id: 'win_rate',
        name: 'Win Rate',
        metric: 'Wins / Matches played',
        sort: 'descending',
        minimumGames: 50, // Require 50+ games
        tiebreaker: 'total_wins'
      }
    ]
  },

  regional: {
    name: 'Regional Leaderboards',
    scope: 'Players in same region',
    regions: ['NA', 'EU', 'APAC', 'SA'],
    displayLimit: 200,
    categories: 'Same as global'
  },

  seasonal: {
    name: 'Seasonal Leaderboards',
    scope: 'Current season only',
    reset: 'Each new season',
    displayLimit: 500, // Extended for season
    categories: [
      {
        id: 'season_rating',
        name: 'Season Rating',
        metric: 'Current season RP'
      },
      {
        id: 'season_wins',
        name: 'Season Wins',
        metric: 'Wins this season'
      },
      {
        id: 'season_kills',
        name: 'Season Kills',
        metric: 'Kills this season'
      }
    ]
  },

  weekly: {
    name: 'Weekly Leaderboards',
    scope: 'Current week',
    reset: 'Monday 00:00 UTC',
    displayLimit: 100,
    categories: [
      { id: 'weekly_wins', name: 'Weekly Wins' },
      { id: 'weekly_kills', name: 'Weekly Kills' }
    ],
    rewards: true // Weekly rewards for top performers
  },

  daily: {
    name: 'Daily Leaderboards',
    scope: 'Current day',
    reset: '00:00 UTC',
    displayLimit: 50,
    categories: [
      { id: 'daily_wins', name: 'Daily Wins' },
      { id: 'daily_kills', name: 'Daily Kills' }
    ],
    rewards: false // No daily rewards
  },

  friends: {
    name: 'Friends Leaderboard',
    scope: 'Player\'s friends only',
    displayLimit: 'All friends',
    categories: [
      { id: 'friends_rating', name: 'Rating' },
      { id: 'friends_wins', name: 'Total Wins' },
      { id: 'friends_season_wins', name: 'Season Wins' }
    ]
  }
};
```

### Stat Categories

```typescript
const STAT_CATEGORIES = {
  rating: {
    id: 'rating',
    name: 'Rating',
    description: 'Competitive matchmaking rating',
    source: 'player_mmr table',
    format: 'number',
    decimals: 0
  },

  wins: {
    id: 'wins',
    name: 'Wins',
    description: 'Total first-place finishes',
    source: 'player_stats table',
    format: 'number'
  },

  kills: {
    id: 'kills',
    name: 'Kills',
    description: 'Total eliminations',
    source: 'player_stats table',
    format: 'number'
  },

  winRate: {
    id: 'win_rate',
    name: 'Win Rate',
    description: 'Wins / Matches (percentage)',
    source: 'calculated',
    formula: '(wins / matches_played) * 100',
    format: 'percentage',
    decimals: 1
  },

  kd: {
    id: 'kd_ratio',
    name: 'K/D Ratio',
    description: 'Kills / Deaths',
    source: 'calculated',
    formula: 'kills / deaths',
    format: 'decimal',
    decimals: 2
  },

  avgPlacement: {
    id: 'avg_placement',
    name: 'Avg Placement',
    description: 'Average final placement',
    source: 'calculated',
    formula: 'sum(placements) / matches_played',
    format: 'decimal',
    decimals: 1,
    sort: 'ascending' // Lower is better
  },

  matchesPlayed: {
    id: 'matches_played',
    name: 'Matches Played',
    description: 'Total matches completed',
    source: 'player_stats table',
    format: 'number'
  },

  damageDealt: {
    id: 'damage_dealt',
    name: 'Damage Dealt',
    description: 'Total damage dealt',
    source: 'player_stats table',
    format: 'number',
    display: 'abbreviated' // 1.2M instead of 1,200,000
  }
};
```

---

## Data Architecture

### Database Schema

```typescript
const LEADERBOARD_SCHEMA = {
  tables: {
    // Real-time leaderboard using sorted sets
    redis: {
      keys: {
        global: 'lb:global:{category}',
        regional: 'lb:regional:{region}:{category}',
        seasonal: 'lb:season:{season_id}:{category}',
        weekly: 'lb:weekly:{week_id}:{category}',
        daily: 'lb:daily:{date}:{category}',
        friends: 'lb:friends:{player_id}:{category}'
      },
      structure: 'Sorted Set (player_id -> score)',
      ttl: {
        global: 'No expiry',
        regional: 'No expiry',
        seasonal: '90 days after season end',
        weekly: '30 days',
        daily: '7 days'
      }
    },

    // Persistent storage for historical data
    postgres: {
      leaderboard_snapshots: {
        columns: {
          id: 'BIGSERIAL PRIMARY KEY',
          leaderboard_type: 'VARCHAR(50)',
          category: 'VARCHAR(50)',
          snapshot_time: 'TIMESTAMP',
          data: 'JSONB', // Top 100 snapshot
          season_id: 'INTEGER',
          week_id: 'INTEGER'
        },
        indexes: [
          'CREATE INDEX idx_lb_snapshot ON leaderboard_snapshots(leaderboard_type, category, snapshot_time)'
        ]
      },

      player_rankings: {
        columns: {
          player_id: 'UUID',
          leaderboard_type: 'VARCHAR(50)',
          category: 'VARCHAR(50)',
          rank: 'INTEGER',
          score: 'BIGINT',
          updated_at: 'TIMESTAMP'
        },
        primaryKey: '(player_id, leaderboard_type, category)',
        indexes: [
          'CREATE INDEX idx_player_rank ON player_rankings(leaderboard_type, category, rank)'
        ]
      }
    }
  }
};
```

### Update Flow

```typescript
const LEADERBOARD_UPDATES = {
  realTime: {
    trigger: 'Match completion',
    process: [
      'Calculate new stats from match result',
      'Update player stats in database',
      'Update relevant Redis sorted sets',
      'Invalidate cached rankings if needed'
    ],
    latency: '< 1 second'
  },

  matchCompletion: async (matchResult: MatchResult): Promise<void> => {
    for (const player of matchResult.players) {
      // Update all relevant leaderboards
      const updates = [
        // Global leaderboards
        updateSortedSet(`lb:global:total_wins`, player.id, player.totalWins),
        updateSortedSet(`lb:global:total_kills`, player.id, player.totalKills),

        // Regional
        updateSortedSet(`lb:regional:${player.region}:total_wins`, player.id, player.totalWins),

        // Seasonal
        updateSortedSet(`lb:season:${currentSeason}:wins`, player.id, player.seasonWins),

        // Time-based
        updateSortedSet(`lb:weekly:${currentWeek}:wins`, player.id, player.weeklyWins),
        updateSortedSet(`lb:daily:${today}:wins`, player.id, player.dailyWins)
      ];

      await Promise.all(updates);
    }
  },

  periodicRecalculation: {
    frequency: 'Hourly',
    purpose: 'Ensure consistency, handle edge cases',
    scope: 'Top 1000 players in each leaderboard'
  },

  seasonReset: {
    trigger: 'New season start',
    actions: [
      'Archive current season leaderboards',
      'Create new season leaderboards',
      'Reset seasonal stats (keep all-time)'
    ]
  }
};
```

---

## Ranking Algorithms

### Tie Breaking

```typescript
const TIE_BREAKING = {
  rules: [
    {
      primary: 'score',
      description: 'Primary stat determines rank'
    },
    {
      secondary: 'win_rate',
      description: 'Higher win rate wins tie'
    },
    {
      tertiary: 'total_matches',
      description: 'More matches wins tie'
    },
    {
      final: 'earlier_achievement',
      description: 'Player who reached score first wins'
    }
  ],

  implementation: {
    // Store composite score: main_score * 10^9 + tiebreaker * 10^6 + timestamp_inverse
    compositeScore: (mainScore: number, tiebreaker: number, timestamp: number): number => {
      const maxTimestamp = 2000000000; // Far future
      const timestampScore = maxTimestamp - timestamp; // Earlier = higher
      return mainScore * 1e9 + tiebreaker * 1e6 + timestampScore;
    }
  },

  display: {
    showTies: 'Same rank number for tied players',
    example: '1, 2, 2, 4' // Two players tied for 2nd
  }
};
```

### Minimum Requirements

```typescript
const LEADERBOARD_REQUIREMENTS = {
  eligibility: {
    minimumMatches: {
      allTime: 10,    // At least 10 matches for all-time
      seasonal: 5,    // At least 5 for seasonal
      weekly: 1,      // At least 1 for weekly
      daily: 1        // At least 1 for daily
    },
    accountAge: {
      days: 1,        // Account at least 1 day old
      reason: 'Prevent smurf accounts'
    },
    standing: {
      notBanned: true,
      noActiveRestrictions: true
    }
  },

  ratioLeaderboards: {
    // For win rate, K/D, etc.
    minimumGames: 50, // Require 50 games for ratio boards
    reason: 'Prevent inflated ratios from few games'
  }
};
```

---

## API Endpoints

### Leaderboard Queries

```typescript
const LEADERBOARD_API = {
  getLeaderboard: {
    method: 'GET',
    path: '/api/v1/leaderboards/{type}/{category}',
    params: {
      type: 'global | regional | seasonal | weekly | daily | friends',
      category: 'rating | wins | kills | win_rate | etc'
    },
    query: {
      region: 'Optional region filter',
      season_id: 'Optional season (defaults to current)',
      limit: 'Number of entries (max 100, default 50)',
      offset: 'Pagination offset'
    },
    response: {
      leaderboard: {
        type: 'string',
        category: 'string',
        lastUpdated: 'ISO8601',
        entries: [{
          rank: 'number',
          playerId: 'string',
          playerName: 'string',
          score: 'number',
          formattedScore: 'string',
          avatar: 'string (URL)',
          tier: 'string (rank tier if applicable)'
        }]
      }
    },
    caching: {
      ttl: 60, // 1 minute cache
      staleWhileRevalidate: true
    }
  },

  getPlayerRank: {
    method: 'GET',
    path: '/api/v1/leaderboards/{type}/{category}/player/{playerId}',
    response: {
      rank: 'number',
      score: 'number',
      formattedScore: 'string',
      percentile: 'number', // Top X%
      nearbyPlayers: [{
        // Players immediately above and below
        rank: 'number',
        playerId: 'string',
        playerName: 'string',
        score: 'number'
      }]
    },
    caching: {
      ttl: 30 // 30 second cache
    }
  },

  getFriendsLeaderboard: {
    method: 'GET',
    path: '/api/v1/leaderboards/friends/{category}',
    authorization: 'Requires player auth',
    response: {
      entries: [{
        rank: 'number', // Rank among friends
        playerId: 'string',
        playerName: 'string',
        score: 'number',
        isOnline: 'boolean',
        lastActive: 'ISO8601'
      }],
      playerRank: 'number' // Requesting player's rank among friends
    },
    caching: {
      ttl: 120 // 2 minute cache (friends less volatile)
    }
  },

  getMyRankings: {
    method: 'GET',
    path: '/api/v1/leaderboards/me',
    authorization: 'Requires player auth',
    response: {
      rankings: [{
        leaderboardType: 'string',
        category: 'string',
        rank: 'number',
        score: 'number',
        percentile: 'number'
      }]
    },
    description: 'Get all rankings for the requesting player'
  }
};
```

---

## Anti-Cheat Validation

### Leaderboard Integrity

```typescript
const LEADERBOARD_ANTICHEAT = {
  preSubmission: {
    serverValidation: {
      description: 'All stats come from authoritative game server',
      clientTrust: 'Zero - clients cannot report stats'
    },
    matchValidation: {
      verify: 'Match completed normally',
      reject: ['Abandoned matches', 'Server crash', 'Exploit detected']
    }
  },

  anomalyDetection: {
    rapidIncrease: {
      description: 'Flag unusual stat jumps',
      threshold: {
        wins: 20, // 20+ wins in 24 hours
        kills: 200 // 200+ kills in 24 hours
      },
      action: 'Flag for review, delay leaderboard update'
    },

    impossibleStats: {
      examples: [
        'Win rate > 90% over 100+ games',
        'K/D > 20 sustained',
        'Damage dealt exceeds theoretical maximum'
      ],
      action: 'Automatic review, temporary leaderboard exclusion'
    }
  },

  banHandling: {
    onBan: {
      action: 'Remove from all leaderboards',
      historical: 'Optionally remove past rankings'
    },
    onUnban: {
      action: 'Recalculate rankings from clean stats'
    }
  },

  reviewQueue: {
    flaggedPlayers: 'Queue for manual review',
    priority: 'Top 100 players reviewed within 24 hours',
    outcome: ['Cleared', 'Stats adjusted', 'Banned']
  }
};
```

---

## Caching Strategy

### Multi-Layer Caching

```typescript
const CACHING_STRATEGY = {
  layers: {
    redis: {
      purpose: 'Primary leaderboard storage',
      ttl: 'No expiry (real-time updates)',
      operations: 'ZADD, ZRANK, ZRANGE, ZREVRANGE'
    },

    application: {
      purpose: 'Reduce Redis calls',
      storage: 'In-memory LRU cache',
      ttl: 30, // seconds
      maxEntries: 1000
    },

    cdn: {
      purpose: 'Reduce API load for top rankings',
      ttl: 60, // seconds
      scope: 'Top 100 of each leaderboard',
      invalidation: 'On significant rank changes'
    }
  },

  invalidation: {
    perPlayer: 'On match completion for that player',
    bulk: 'On season reset or major recalculation',
    manual: 'Admin trigger for fixes'
  },

  staleWhileRevalidate: {
    enabled: true,
    maxStale: 300, // 5 minutes max stale
    behavior: 'Return cached, update in background'
  }
};
```

---

## UI Integration

### Leaderboard Display

```typescript
const LEADERBOARD_UI = {
  location: 'Main menu > Leaderboards',

  layout: {
    tabs: ['Global', 'Regional', 'Season', 'Weekly', 'Friends'],
    categorySelector: 'Dropdown or horizontal scroll',
    list: 'Scrollable list with virtualization'
  },

  entryDisplay: {
    rank: 'Medal icons for top 3',
    playerInfo: {
      avatar: 'Player avatar',
      name: 'Display name',
      tier: 'Rank tier emblem if applicable'
    },
    score: 'Formatted score',
    trend: 'Up/down arrow vs previous period (optional)'
  },

  playerHighlight: {
    own: {
      visible: 'Always show player\'s own rank',
      sticky: 'Sticky footer showing own position',
      jumpTo: 'Button to scroll to own position'
    }
  },

  refresh: {
    manual: 'Pull-to-refresh',
    auto: 'Every 5 minutes when viewing',
    animation: 'Subtle rank change animations'
  },

  empty: {
    noData: 'Play matches to appear on leaderboard',
    noFriends: 'Add friends to see friends leaderboard'
  }
};
```

---

## Performance Requirements

```typescript
const PERFORMANCE_REQUIREMENTS = {
  query: {
    topN: {
      p50: '10ms',
      p99: '50ms',
      operation: 'ZREVRANGE (get top 100)'
    },
    playerRank: {
      p50: '5ms',
      p99: '20ms',
      operation: 'ZRANK (single player lookup)'
    },
    friendsLeaderboard: {
      p50: '20ms',
      p99: '100ms',
      operation: 'Multi-ZSCORE for friend list'
    }
  },

  update: {
    singlePlayer: {
      p50: '2ms',
      p99: '10ms',
      operation: 'ZADD (update score)'
    },
    batchUpdate: {
      p50: '50ms',
      p99: '200ms',
      operation: 'ZADD x 60 (full match)'
    }
  },

  scale: {
    players: '10 million',
    leaderboards: '50 (types x categories)',
    queriesPerSecond: 10000,
    updatesPerSecond: 5000
  },

  storage: {
    perPlayer: '< 1KB across all leaderboards',
    total: '< 50GB for full leaderboard data'
  }
};
```

---

## Rewards Integration

### Leaderboard Rewards

```typescript
const LEADERBOARD_REWARDS = {
  weekly: {
    enabled: true,
    distribution: 'Monday 00:00 UTC',
    tiers: [
      { rank: '1', reward: { currency: 500, exclusive: 'Weekly Champion Title' } },
      { rank: '2-3', reward: { currency: 300 } },
      { rank: '4-10', reward: { currency: 200 } },
      { rank: '11-50', reward: { currency: 100 } },
      { rank: '51-100', reward: { currency: 50 } }
    ]
  },

  seasonal: {
    enabled: true,
    distribution: 'Season end',
    tiers: [
      { rank: '1', reward: { currency: 5000, cosmetic: 'Season Champion Skin' } },
      { rank: '2-10', reward: { currency: 2500, cosmetic: 'Top 10 Frame' } },
      { rank: '11-100', reward: { currency: 1000, cosmetic: 'Top 100 Badge' } },
      { rank: '101-500', reward: { currency: 500 } }
    ]
  },

  distribution: {
    automation: 'Scheduled job at reset time',
    notification: 'Push notification for reward recipients',
    claim: 'Auto-claimed to inventory'
  }
};
```

---

## Analytics

```typescript
const LEADERBOARD_ANALYTICS = {
  events: [
    {
      name: 'leaderboard_viewed',
      properties: {
        type: 'string',
        category: 'string',
        player_rank: 'number'
      }
    },
    {
      name: 'leaderboard_rank_changed',
      properties: {
        type: 'string',
        category: 'string',
        old_rank: 'number',
        new_rank: 'number'
      }
    },
    {
      name: 'leaderboard_reward_distributed',
      properties: {
        type: 'string',
        rank: 'number',
        reward_value: 'number'
      }
    }
  ],

  metrics: [
    'Leaderboard page views',
    'Time spent on leaderboards',
    'Players participating in weekly challenges',
    'Rank volatility (average rank changes)',
    'Friends leaderboard engagement'
  ]
};
```

---

## Testing Requirements

```typescript
const LEADERBOARD_TESTING = {
  functional: [
    'Leaderboards update after match completion',
    'Rankings are accurate and sorted correctly',
    'Tie-breaking works as specified',
    'Player can view own rank in any leaderboard',
    'Friends leaderboard shows only friends',
    'Time-based leaderboards reset correctly'
  ],

  performance: [
    'Top 100 query under 50ms at p99',
    'Player rank lookup under 20ms at p99',
    'System handles 10k queries/second',
    'Updates process within 1 second of match end'
  ],

  integrity: [
    'Banned players excluded from leaderboards',
    'Anomalous stats flagged for review',
    'Minimum game requirements enforced',
    'No duplicate entries possible'
  ],

  edge: [
    'New player appears after first eligible match',
    'Tied players share same rank',
    'Season reset creates new leaderboards correctly',
    'Player removed from leaderboard on ban'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial leaderboards service specification |
