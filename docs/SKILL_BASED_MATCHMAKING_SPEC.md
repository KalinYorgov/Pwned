# Skill-Based Matchmaking (SBMM) Backend Specification

## Document Information
- **Task ID:** BACK-003
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the backend implementation of the Skill-Based Matchmaking (SBMM) system for Plunderstorm Mobile. The system manages hidden MMR (Matchmaking Rating), matches players within skill ranges, handles placement calibration, and includes smurf detection mechanisms.

---

## MMR System Architecture

### Rating Structure

```typescript
const MMR_SYSTEM = {
  rating: {
    type: 'Hidden numerical rating',
    range: { min: 0, max: 5000 },
    precision: 'Integer',
    separate: 'Per game mode (solo, duo)'
  },

  baseline: {
    newPlayer: 1000, // Starting MMR
    uncertainty: {
      initial: 350, // High uncertainty for new players
      minimum: 75,  // After many games
      decayRate: 0.98 // Per game reduction
    }
  },

  storage: {
    primary: 'Player profile database',
    cache: 'Redis for quick access during matchmaking',
    history: 'Last 100 MMR changes per player'
  }
};

interface PlayerMMR {
  playerId: string;
  solo: {
    rating: number;
    uncertainty: number;
    gamesPlayed: number;
    lastUpdated: timestamp;
    peakRating: number;
    seasonStartRating: number;
  };
  duo: {
    rating: number;
    uncertainty: number;
    gamesPlayed: number;
    lastUpdated: timestamp;
    peakRating: number;
    seasonStartRating: number;
  };
  placementStatus: {
    solo: { complete: boolean; gamesRemaining: number };
    duo: { complete: boolean; gamesRemaining: number };
  };
}
```

### MMR Calculation Algorithm

```typescript
const MMR_CALCULATION = {
  // Modified Glicko-2 inspired system
  algorithm: 'Placement-aware Elo with uncertainty',

  matchResult: {
    // Inputs for MMR calculation
    inputs: {
      placement: 'Final placement (1-60 for solo)',
      kills: 'Total kills',
      assists: 'Total assists',
      damageDealt: 'Total damage dealt',
      survivalTime: 'Time survived in seconds',
      lobbyAverageMMR: 'Average MMR of lobby',
      playerUncertainty: 'Current uncertainty value'
    }
  },

  expectedPlacement: (playerMMR: number, lobbyMMRs: number[]): number => {
    // Calculate expected placement based on MMR distribution
    const sortedMMRs = [...lobbyMMRs].sort((a, b) => b - a);
    let expected = 1;
    for (const mmr of sortedMMRs) {
      if (mmr > playerMMR) expected++;
      else break;
    }
    return expected;
  },

  placementScore: (actual: number, expected: number, totalPlayers: number): number => {
    // Convert placement to normalized score
    const actualNorm = 1 - (actual - 1) / (totalPlayers - 1); // 1 for 1st, 0 for last
    const expectedNorm = 1 - (expected - 1) / (totalPlayers - 1);
    return actualNorm - expectedNorm; // Positive if better than expected
  },

  killBonus: {
    // Additional MMR for kills
    perKill: 2, // Base MMR per kill
    diminishing: {
      // Diminishing returns after certain kills
      threshold: 5,
      multiplier: 0.5 // Half value after 5 kills
    },
    cap: 15 // Maximum kill bonus contribution
  },

  damageBonus: {
    // Damage dealt consideration
    weight: 0.1, // 0.1 MMR per 100 damage
    cap: 10 // Maximum damage bonus
  }
};

const calculateMMRChange = (
  player: PlayerMMR,
  matchResult: MatchResult,
  lobby: LobbyData
): number => {
  // Base K factor (higher for new players)
  const baseK = player.gamesPlayed < 30 ? 48 : 32;
  const uncertaintyMultiplier = player.uncertainty / 100;
  const kFactor = baseK * uncertaintyMultiplier;

  // Calculate expected vs actual placement
  const expected = MMR_CALCULATION.expectedPlacement(
    player.rating,
    lobby.mmrs
  );
  const placementScore = MMR_CALCULATION.placementScore(
    matchResult.placement,
    expected,
    lobby.playerCount
  );

  // Base change from placement
  let change = kFactor * placementScore;

  // Kill bonus
  let killBonus = matchResult.kills * MMR_CALCULATION.killBonus.perKill;
  if (matchResult.kills > MMR_CALCULATION.killBonus.diminishing.threshold) {
    const excess = matchResult.kills - MMR_CALCULATION.killBonus.diminishing.threshold;
    killBonus = MMR_CALCULATION.killBonus.diminishing.threshold * MMR_CALCULATION.killBonus.perKill +
      excess * MMR_CALCULATION.killBonus.perKill * MMR_CALCULATION.killBonus.diminishing.multiplier;
  }
  killBonus = Math.min(killBonus, MMR_CALCULATION.killBonus.cap);

  // Add kill bonus (weighted by match result)
  if (matchResult.placement <= 10) {
    change += killBonus; // Full bonus for top 10
  } else if (matchResult.placement <= 30) {
    change += killBonus * 0.5; // Half bonus for mid-placement
  }
  // No kill bonus for bottom half

  // Damage bonus for close but lost fights
  const damageBonus = Math.min(
    (matchResult.damageDealt / 100) * MMR_CALCULATION.damageBonus.weight,
    MMR_CALCULATION.damageBonus.cap
  );
  if (matchResult.placement > 30) {
    change += damageBonus * 0.5; // Partial credit for damage even when dying early
  }

  // Floor and ceiling
  change = Math.max(-50, Math.min(75, change)); // Limit extreme swings

  return Math.round(change);
};
```

---

## Placement Matches

### Calibration System

```typescript
const PLACEMENT_SYSTEM = {
  games: {
    required: 10,
    description: 'First 10 ranked games calibrate initial MMR'
  },

  initialRange: {
    // Placement can result in wide range of starting MMR
    minimum: 500,   // Very poor performance
    maximum: 1800,  // Exceptional performance
    average: 1000   // Average performance
  },

  algorithm: {
    // Track performance across all 10 games
    tracking: {
      totalPlacements: 'Sum of placements',
      averagePlacement: 'Average placement across games',
      totalKills: 'Sum of kills',
      wins: 'Number of first place finishes',
      topTens: 'Number of top 10 finishes'
    },

    scoring: {
      // Point system for final placement determination
      win: 100,
      topThree: 80,
      topFive: 60,
      topTen: 40,
      topTwenty: 20,
      perKill: 5,
      maxKillPoints: 100
    },

    calculation: (stats: PlacementStats): number => {
      let points = 0;

      // Placement points
      points += stats.wins * 100;
      points += (stats.topThrees - stats.wins) * 80;
      points += (stats.topFives - stats.topThrees) * 60;
      points += (stats.topTens - stats.topFives) * 40;
      points += (stats.topTwenties - stats.topTens) * 20;

      // Kill points (capped)
      points += Math.min(stats.totalKills * 5, 100);

      // Convert to MMR (scaled to range)
      const maxPoints = 10 * 100 + 100; // 10 wins + max kills
      const ratio = points / maxPoints;
      const mmr = PLACEMENT_SYSTEM.initialRange.minimum +
        ratio * (PLACEMENT_SYSTEM.initialRange.maximum - PLACEMENT_SYSTEM.initialRange.minimum);

      return Math.round(mmr);
    }
  },

  provisionalRank: {
    showAfter: 5, // Show provisional rank after 5 games
    disclaimer: 'Provisional rank - may change significantly'
  },

  matchmaking: {
    // During placements, wider matching
    tolerance: 300, // MMR tolerance during placements
    prioritize: 'Other placement players when possible',
    lobby: 'Mixed with calibrated players for fair assessment'
  }
};
```

---

## Matchmaking Algorithm

### Queue and Matching

```typescript
const MATCHMAKING_ALGORITHM = {
  queue: {
    structure: 'Priority queue ordered by wait time',
    grouping: 'Bucketed by MMR range (100 point buckets)',
    refresh: 1000 // ms between matching attempts
  },

  matching: {
    // Initial tolerance
    initialTolerance: 50, // MMR points

    // Tolerance expansion
    expansion: {
      interval: 15000, // Every 15 seconds
      increment: 25,   // Expand by 25 MMR
      maximum: 300,    // Max 300 MMR difference
      accelerated: {
        // Faster expansion for high/low MMR
        threshold: {
          high: 2500,
          low: 500
        },
        multiplier: 1.5 // 1.5x expansion rate
      }
    },

    // Quality scoring
    quality: {
      mmrSpread: {
        weight: 0.4,
        ideal: 50,  // Best if all within 50 MMR
        acceptable: 150 // Acceptable if within 150
      },
      waitTime: {
        weight: 0.3,
        targetSeconds: 30
      },
      lobbyFill: {
        weight: 0.3,
        target: 60 // Full lobby
      }
    }
  },

  process: async (queuedPlayers: QueueEntry[]): Promise<Match | null> => {
    // Sort by wait time (longest waiting first)
    const sorted = [...queuedPlayers].sort((a, b) => a.queueTime - b.queueTime);

    // Try to form match around longest-waiting player
    const anchor = sorted[0];
    const tolerance = calculateTolerance(anchor);

    // Find compatible players
    const compatible = sorted.filter(p =>
      Math.abs(p.mmr - anchor.mmr) <= tolerance
    );

    if (compatible.length >= 50) { // Minimum for ranked
      // Select best 60 for match quality
      return formMatch(compatible.slice(0, 60));
    }

    return null; // No match possible yet
  }
};
```

### Party MMR Handling

```typescript
const PARTY_MMR = {
  calculation: {
    method: 'Weighted average with boost',

    formula: (partyMembers: PlayerMMR[]): number => {
      // Calculate average
      const sum = partyMembers.reduce((acc, p) => acc + p.rating, 0);
      const average = sum / partyMembers.length;

      // Add party coordination boost
      // Parties have advantage, so match slightly higher
      const boost = 50 + (partyMembers.length - 1) * 25;
      // 2-player party: +75, 3-player: +100, 4-player: +125

      return average + boost;
    }
  },

  restrictions: {
    // Prevent boosting with wide skill gaps
    maxDifference: {
      divisions: 2,   // Max 2 divisions apart
      mmr: 400        // Or 400 MMR
    },

    exceptions: {
      placements: true, // Ignore during placements
      newPartyMember: true // First 10 games with new party member
    }
  },

  matchingBehavior: {
    // Use party MMR for all members
    partyAsUnit: true,
    soloFill: 'Match at party MMR level',
    opposingParty: 'Prefer matching party vs party'
  }
};
```

---

## Smurf Detection

### Detection Mechanisms

```typescript
const SMURF_DETECTION = {
  metrics: {
    // Performance indicators that suggest smurfing
    winRate: {
      threshold: 0.75, // 75%+ win rate over 20+ games
      minimumGames: 20
    },

    killAverage: {
      threshold: 8, // 8+ kills average at low MMR
      mmrThreshold: 1200 // Only flag below this MMR
    },

    rapidClimb: {
      mmrGain: 500, // Gained 500+ MMR
      games: 15,    // In 15 games
      rate: 33.33   // ~33 MMR per game sustained
    },

    placementPerformance: {
      wins: 7, // 7+ wins in placements
      averagePlacement: 3 // Average top 3
    },

    inputPatterns: {
      // Advanced detection (future)
      aimAccuracy: 'Unusually high for new account',
      reactionTime: 'Unusually fast',
      movementEfficiency: 'Expert-level pathing'
    }
  },

  scoring: {
    // Aggregate smurf likelihood score
    weights: {
      winRate: 30,
      killAverage: 25,
      rapidClimb: 25,
      placementPerformance: 20
    },

    thresholds: {
      suspicious: 60,  // Score 60+ = suspicious
      likely: 80,      // Score 80+ = likely smurf
      confirmed: 95    // Score 95+ = almost certain
    }
  },

  responses: {
    suspicious: {
      action: 'Accelerate MMR gains',
      multiplier: 1.5, // 50% faster MMR gains
      review: false
    },

    likely: {
      action: 'Aggressive MMR acceleration',
      multiplier: 2.0, // 2x faster MMR gains
      review: 'Flag for manual review',
      matchHigher: true // Match at higher MMR than shown
    },

    confirmed: {
      action: 'Maximum acceleration',
      multiplier: 3.0, // 3x faster gains
      review: 'Priority review',
      matchHigher: true,
      placementSkip: true // Skip to higher MMR bracket
    }
  }
};

const evaluateSmurfLikelihood = (player: PlayerProfile): SmurfEvaluation => {
  let score = 0;

  // Win rate check
  if (player.gamesPlayed >= 20) {
    const winRate = player.wins / player.gamesPlayed;
    if (winRate >= 0.75) {
      score += 30 * (winRate / 0.75); // Scale beyond threshold
    }
  }

  // Kill average at low MMR
  if (player.mmr < 1200) {
    const avgKills = player.totalKills / player.gamesPlayed;
    if (avgKills >= 8) {
      score += 25 * (avgKills / 8);
    }
  }

  // Rapid climb detection
  const recentGames = player.recentMatches.slice(0, 15);
  const mmrChange = player.mmr - player.mmrAfterPlacements;
  const gamesCount = Math.min(player.gamesPlayed - 10, 15);
  if (gamesCount > 0 && mmrChange / gamesCount > 33) {
    score += 25 * ((mmrChange / gamesCount) / 33);
  }

  // Placement performance
  if (player.placementStats) {
    if (player.placementStats.wins >= 7 || player.placementStats.avgPlacement <= 3) {
      score += 20;
    }
  }

  return {
    score: Math.min(100, score),
    category: score >= 95 ? 'confirmed' :
              score >= 80 ? 'likely' :
              score >= 60 ? 'suspicious' : 'normal',
    recommendations: SMURF_DETECTION.responses[this.category]
  };
};
```

---

## Seasonal Reset

### Reset Mechanics

```typescript
const SEASONAL_RESET = {
  type: 'Soft reset',

  schedule: {
    frequency: 60, // Days per season
    preseason: 3,  // Days for transition
  },

  calculation: {
    // Compress MMR toward median
    median: 1000,
    compression: 0.5, // 50% compression

    formula: (currentMMR: number): number => {
      const median = SEASONAL_RESET.calculation.median;
      const compression = SEASONAL_RESET.calculation.compression;

      // New MMR = median + (current - median) * compression
      const newMMR = median + (currentMMR - median) * compression;

      // Ensure within bounds
      return Math.max(400, Math.min(2500, Math.round(newMMR)));
    },

    examples: [
      { current: 3000, new: 2000, note: 'High MMR compressed significantly' },
      { current: 2000, new: 1500, note: 'Above average compressed' },
      { current: 1000, new: 1000, note: 'Median unchanged' },
      { current: 500, new: 750, note: 'Below median boosted' }
    ]
  },

  uncertaintyReset: {
    increase: 100, // Add 100 uncertainty
    cap: 250      // Don't exceed 250
  },

  placementReset: {
    required: false, // Don't require new placements
    optional: {
      enabled: true,
      description: 'Players can opt into fresh placements',
      newRange: 'Based on compressed MMR as starting point'
    }
  }
};
```

---

## Data Storage

### Database Schema

```typescript
const MMR_DATABASE = {
  tables: {
    player_mmr: {
      columns: {
        player_id: 'UUID PRIMARY KEY',
        solo_mmr: 'INTEGER NOT NULL DEFAULT 1000',
        solo_uncertainty: 'FLOAT NOT NULL DEFAULT 350',
        solo_games_played: 'INTEGER NOT NULL DEFAULT 0',
        solo_peak_mmr: 'INTEGER NOT NULL DEFAULT 1000',
        duo_mmr: 'INTEGER NOT NULL DEFAULT 1000',
        duo_uncertainty: 'FLOAT NOT NULL DEFAULT 350',
        duo_games_played: 'INTEGER NOT NULL DEFAULT 0',
        duo_peak_mmr: 'INTEGER NOT NULL DEFAULT 1000',
        placement_solo_complete: 'BOOLEAN DEFAULT FALSE',
        placement_duo_complete: 'BOOLEAN DEFAULT FALSE',
        last_updated: 'TIMESTAMP',
        season_id: 'INTEGER'
      },
      indexes: [
        'CREATE INDEX idx_solo_mmr ON player_mmr(solo_mmr)',
        'CREATE INDEX idx_duo_mmr ON player_mmr(duo_mmr)'
      ]
    },

    mmr_history: {
      columns: {
        id: 'BIGSERIAL PRIMARY KEY',
        player_id: 'UUID NOT NULL',
        match_id: 'UUID NOT NULL',
        mode: 'VARCHAR(10) NOT NULL', // solo, duo
        mmr_before: 'INTEGER NOT NULL',
        mmr_after: 'INTEGER NOT NULL',
        mmr_change: 'INTEGER NOT NULL',
        placement: 'INTEGER NOT NULL',
        kills: 'INTEGER NOT NULL',
        timestamp: 'TIMESTAMP NOT NULL'
      },
      retention: '90 days',
      indexes: [
        'CREATE INDEX idx_mmr_history_player ON mmr_history(player_id, timestamp DESC)'
      ]
    },

    placement_stats: {
      columns: {
        player_id: 'UUID PRIMARY KEY',
        mode: 'VARCHAR(10) NOT NULL',
        season_id: 'INTEGER NOT NULL',
        games_completed: 'INTEGER NOT NULL DEFAULT 0',
        total_placements: 'INTEGER NOT NULL DEFAULT 0',
        total_kills: 'INTEGER NOT NULL DEFAULT 0',
        wins: 'INTEGER NOT NULL DEFAULT 0',
        top_tens: 'INTEGER NOT NULL DEFAULT 0',
        calculated_mmr: 'INTEGER'
      }
    },

    smurf_evaluations: {
      columns: {
        player_id: 'UUID PRIMARY KEY',
        score: 'FLOAT NOT NULL',
        category: 'VARCHAR(20) NOT NULL',
        last_evaluated: 'TIMESTAMP NOT NULL',
        manual_review_status: 'VARCHAR(20)',
        mmr_multiplier: 'FLOAT DEFAULT 1.0'
      }
    }
  },

  caching: {
    engine: 'Redis',
    keys: {
      playerMMR: 'mmr:{player_id}',
      queueBucket: 'queue:{mode}:{mmr_bucket}',
      matchQuality: 'match_quality:{match_id}'
    },
    ttl: {
      playerMMR: 3600, // 1 hour
      queueBucket: 60,  // 1 minute
      matchQuality: 86400 // 24 hours
    }
  }
};
```

---

## API Endpoints

### MMR Service API

```typescript
const MMR_API = {
  endpoints: {
    getPlayerMMR: {
      method: 'GET',
      path: '/api/v1/mmr/{player_id}',
      response: {
        playerId: 'string',
        solo: {
          mmr: 'number',
          gamesPlayed: 'number',
          placementComplete: 'boolean'
        },
        duo: {
          mmr: 'number',
          gamesPlayed: 'number',
          placementComplete: 'boolean'
        }
      },
      authorization: 'Player can view own, admins can view any'
    },

    updateMMRPostMatch: {
      method: 'POST',
      path: '/api/v1/mmr/update',
      body: {
        matchId: 'string',
        mode: 'solo | duo',
        results: [{
          playerId: 'string',
          placement: 'number',
          kills: 'number',
          assists: 'number',
          damageDealt: 'number',
          survivalTime: 'number'
        }]
      },
      internal: true, // Only game server can call
      response: {
        updates: [{
          playerId: 'string',
          mmrBefore: 'number',
          mmrAfter: 'number',
          change: 'number'
        }]
      }
    },

    getMMRHistory: {
      method: 'GET',
      path: '/api/v1/mmr/{player_id}/history',
      query: {
        mode: 'solo | duo',
        limit: 'number (default 20, max 100)',
        offset: 'number'
      },
      response: {
        history: [{
          matchId: 'string',
          mmrBefore: 'number',
          mmrAfter: 'number',
          change: 'number',
          placement: 'number',
          kills: 'number',
          timestamp: 'ISO8601'
        }]
      }
    },

    seasonReset: {
      method: 'POST',
      path: '/api/v1/mmr/season-reset',
      internal: true, // Scheduled job only
      body: {
        newSeasonId: 'number'
      },
      response: {
        playersReset: 'number',
        averageMMRChange: 'number'
      }
    },

    evaluateSmurf: {
      method: 'POST',
      path: '/api/v1/mmr/smurf-check/{player_id}',
      internal: true,
      response: {
        score: 'number',
        category: 'normal | suspicious | likely | confirmed',
        recommendedMultiplier: 'number'
      }
    }
  }
};
```

---

## Matchmaking Queue Service

### Queue Management

```typescript
const QUEUE_SERVICE = {
  implementation: {
    technology: 'Redis Sorted Sets',
    structure: 'Bucketed by MMR range'
  },

  operations: {
    joinQueue: {
      steps: [
        'Validate player eligibility',
        'Get current MMR',
        'Calculate party MMR if applicable',
        'Add to appropriate bucket',
        'Start wait time tracking'
      ],
      data: {
        playerId: 'string',
        partyId: 'string | null',
        mmr: 'number',
        queueTime: 'timestamp',
        region: 'string'
      }
    },

    leaveQueue: {
      steps: [
        'Remove from bucket',
        'Clean up wait time tracking',
        'Notify party members if applicable'
      ]
    },

    matchmaking: {
      interval: 1000, // Run every second
      process: [
        'Get all players with wait time > threshold',
        'Group by region',
        'For each region, attempt match formation',
        'Notify matched players',
        'Remove matched players from queue'
      ]
    }
  },

  monitoring: {
    metrics: [
      'Queue depth by MMR bracket',
      'Average wait time by bracket',
      'Match formation success rate',
      'Match quality scores',
      'Players who left queue before match'
    ]
  }
};
```

---

## Performance Requirements

```typescript
const PERFORMANCE_REQUIREMENTS = {
  mmrLookup: {
    p50: '5ms',
    p99: '20ms',
    source: 'Redis cache with DB fallback'
  },

  mmrUpdate: {
    p50: '10ms',
    p99: '50ms',
    async: 'History write can be async'
  },

  matchmaking: {
    targetTime: '30s average',
    maximum: '180s before timeout',
    matchQuality: 'Minimize MMR spread while meeting time target'
  },

  throughput: {
    concurrent: '10,000 players in queue',
    matchesPerMinute: 500,
    updatesPerSecond: 5000
  },

  availability: {
    sla: '99.9%',
    fallback: 'Casual matchmaking if SBMM unavailable'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  unit: [
    'MMR calculation produces expected results',
    'Placement scoring is accurate',
    'Party MMR weighting correct',
    'Smurf detection scoring accurate',
    'Season reset calculation correct'
  ],

  integration: [
    'Queue join/leave operations work correctly',
    'Match formation creates balanced lobbies',
    'MMR updates persist correctly',
    'History tracking works',
    'Redis and DB stay in sync'
  ],

  load: [
    '10,000 concurrent queue entries',
    '500 matches formed per minute',
    '5,000 MMR updates per second',
    'No degradation under peak load'
  ],

  quality: [
    'Match quality scores meet targets',
    'MMR spread in matches within tolerance',
    'Wait times meet targets by bracket',
    'Smurf detection catches test cases'
  ],

  regression: [
    'MMR stability over many games',
    'No MMR inflation/deflation over seasons',
    'Fair distribution across ranks'
  ]
};
```

---

## Analytics

```typescript
const SBMM_ANALYTICS = {
  events: [
    {
      name: 'mmr_changed',
      properties: {
        player_id: 'string',
        mode: 'string',
        old_mmr: 'number',
        new_mmr: 'number',
        change: 'number',
        match_placement: 'number'
      }
    },
    {
      name: 'placement_complete',
      properties: {
        player_id: 'string',
        mode: 'string',
        final_mmr: 'number',
        wins: 'number',
        avg_placement: 'number'
      }
    },
    {
      name: 'smurf_detected',
      properties: {
        player_id: 'string',
        score: 'number',
        category: 'string',
        games_played: 'number'
      }
    },
    {
      name: 'match_quality',
      properties: {
        match_id: 'string',
        mmr_spread: 'number',
        avg_wait_time: 'number',
        player_count: 'number',
        quality_score: 'number'
      }
    }
  ],

  dashboards: [
    'MMR distribution histogram',
    'Average match quality over time',
    'Wait time by MMR bracket',
    'Smurf detection effectiveness',
    'Season reset impact analysis'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial SBMM backend specification |
