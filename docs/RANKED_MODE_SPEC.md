# Ranked Mode Specification

## Document Information
- **Task ID:** GAME-023
- **Priority:** P2
- **Complexity:** Large
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the competitive ranked mode system for Plunderstorm Mobile, providing skill-based matchmaking, visible progression through rank tiers, seasonal resets, and exclusive rewards for competitive players.

---

## Rank Tiers

### Rank Hierarchy

```typescript
const RANK_TIERS = {
  tiers: [
    {
      id: 'deckhand',
      name: 'Deckhand',
      displayName: 'Bronze',
      divisions: 4, // IV, III, II, I
      rpRange: { min: 0, max: 399 },
      rpPerDivision: 100,
      icon: 'rank_deckhand',
      color: '#CD7F32',
      benefits: []
    },
    {
      id: 'sailor',
      name: 'Sailor',
      displayName: 'Silver',
      divisions: 4,
      rpRange: { min: 400, max: 799 },
      rpPerDivision: 100,
      icon: 'rank_sailor',
      color: '#C0C0C0',
      benefits: ['Ranked frame unlocked']
    },
    {
      id: 'boatswain',
      name: 'Boatswain',
      displayName: 'Gold',
      divisions: 4,
      rpRange: { min: 800, max: 1299 },
      rpPerDivision: 125,
      icon: 'rank_boatswain',
      color: '#FFD700',
      benefits: ['Gold ranked frame', 'Ranked emote access']
    },
    {
      id: 'first_mate',
      name: 'First Mate',
      displayName: 'Platinum',
      divisions: 4,
      rpRange: { min: 1300, max: 1899 },
      rpPerDivision: 150,
      icon: 'rank_first_mate',
      color: '#00CED1',
      benefits: ['Platinum frame', 'Ranked title']
    },
    {
      id: 'captain',
      name: 'Captain',
      displayName: 'Diamond',
      divisions: 4,
      rpRange: { min: 1900, max: 2599 },
      rpPerDivision: 175,
      icon: 'rank_captain',
      color: '#B9F2FF',
      benefits: ['Diamond frame', 'Ship trail effect']
    },
    {
      id: 'admiral',
      name: 'Admiral',
      displayName: 'Master',
      divisions: 1, // No divisions - single tier
      rpRange: { min: 2600, max: 3499 },
      rpPerDivision: null,
      icon: 'rank_admiral',
      color: '#9932CC',
      benefits: ['Master frame', 'Exclusive admiral title', 'Lobby badge']
    },
    {
      id: 'pirate_king',
      name: 'Pirate King',
      displayName: 'Pirate King',
      divisions: 1,
      rpRange: { min: 3500, max: null }, // No cap
      rpPerDivision: null,
      icon: 'rank_pirate_king',
      color: '#FF4500',
      requirements: 'Top 100 players in region',
      benefits: ['Legendary frame', 'Animated emblem', 'Exclusive cosmetic set']
    }
  ],

  divisionLabels: {
    4: 'IV', // Lowest in tier
    3: 'III',
    2: 'II',
    1: 'I' // Highest in tier
  }
};
```

### Rank Determination

```typescript
const RANK_CALCULATION = {
  getRank(rp: number, regionLeaderboard: Leaderboard): RankInfo {
    // Check Pirate King first (top 100)
    if (rp >= 3500 && regionLeaderboard.isTop100(this.playerId)) {
      return {
        tier: 'pirate_king',
        division: null,
        rp: rp,
        leaderboardRank: regionLeaderboard.getPosition(this.playerId)
      };
    }

    // Find tier by RP
    const tier = RANK_TIERS.tiers.find(t =>
      rp >= t.rpRange.min && (t.rpRange.max === null || rp <= t.rpRange.max)
    );

    // Calculate division within tier
    const divisionRP = rp - tier.rpRange.min;
    const division = tier.divisions > 1
      ? Math.min(tier.divisions, Math.floor(divisionRP / tier.rpPerDivision) + 1)
      : null;

    return {
      tier: tier.id,
      tierName: tier.name,
      displayName: tier.displayName,
      division: division ? RANK_TIERS.divisionLabels[tier.divisions - division + 1] : null,
      rp: rp,
      rpToNextDivision: tier.rpPerDivision
        ? tier.rpPerDivision - (divisionRP % tier.rpPerDivision)
        : null
    };
  }
};
```

---

## Ranked Points (RP) System

### RP Gain/Loss Calculation

```typescript
const RP_SYSTEM = {
  placement: {
    // Solo mode (60 players)
    solo: {
      1: { base: 100, bonus: 'Win streak bonus' },
      2: { base: 75 },
      3: { base: 60 },
      4: { base: 50 },
      5: { base: 45 },
      '6-10': { base: 35 },
      '11-15': { base: 25 },
      '16-20': { base: 15 },
      '21-30': { base: 5 },
      '31-40': { base: -5 },
      '41-50': { base: -15 },
      '51-60': { base: -25 }
    },

    // Duo mode (30 teams)
    duo: {
      1: { base: 100 },
      2: { base: 70 },
      3: { base: 55 },
      '4-5': { base: 40 },
      '6-10': { base: 25 },
      '11-15': { base: 10 },
      '16-20': { base: -5 },
      '21-25': { base: -15 },
      '26-30': { base: -25 }
    }
  },

  kills: {
    rpPerKill: 5,
    maxKillRP: 50, // Cap at 10 kills
    assistMultiplier: 0.5 // 2.5 RP per assist
  },

  modifiers: {
    // Rank differential modifier
    rankDifferential: {
      // Beating higher-ranked players gives bonus
      higherRankKill: {
        oneDivisionUp: 1.1,
        oneTierUp: 1.25,
        twoTiersUp: 1.5
      },
      // Less RP for lower-ranked opponents
      lowerRankKill: {
        oneDivisionDown: 0.9,
        oneTierDown: 0.75,
        twoTiersDown: 0.5
      }
    },

    // Win streak bonus
    winStreak: {
      2: 1.1,  // 10% bonus
      3: 1.15, // 15% bonus
      4: 1.2,  // 20% bonus
      5: 1.25  // 25% bonus (max)
    },

    // Loss protection for new ranks
    demotion: {
      protectionGames: 3, // 3 games at 0 RP before demotion
      minimumRP: 0 // Cannot go below 0 in current tier initially
    }
  }
};

const calculateRP = (matchResult: MatchResult, playerRank: RankInfo): number => {
  // Base RP from placement
  let rp = RP_SYSTEM.placement[matchResult.mode][matchResult.placement].base;

  // Add kill RP (capped)
  const killRP = Math.min(
    matchResult.kills * RP_SYSTEM.kills.rpPerKill,
    RP_SYSTEM.kills.maxKillRP
  );
  const assistRP = matchResult.assists * RP_SYSTEM.kills.rpPerKill * RP_SYSTEM.kills.assistMultiplier;
  rp += killRP + assistRP;

  // Apply win streak modifier
  if (matchResult.placement === 1 && matchResult.winStreak >= 2) {
    const streakBonus = RP_SYSTEM.modifiers.winStreak[
      Math.min(matchResult.winStreak, 5)
    ];
    rp *= streakBonus;
  }

  // Apply rank differential for kills
  matchResult.killDetails.forEach(kill => {
    const differential = compareRanks(playerRank, kill.victimRank);
    // Adjust RP based on differential
  });

  return Math.round(rp);
};
```

### Entry Cost

```typescript
const ENTRY_COST = {
  // RP entry cost by tier
  byTier: {
    deckhand: 0,
    sailor: 10,
    boatswain: 15,
    first_mate: 20,
    captain: 25,
    admiral: 30,
    pirate_king: 35
  },

  description: 'Entry cost is deducted at match start. ' +
    'Net RP change = placement RP + kill RP - entry cost',

  protection: {
    newToTier: true, // First 3 games in new tier have no entry cost
    demotionProtection: true // Cannot demote from first 3 losses at 0 RP
  }
};
```

---

## Skill-Based Matchmaking (SBMM)

### Matchmaking Parameters

```typescript
const RANKED_MATCHMAKING = {
  primary: {
    metric: 'MMR', // Hidden matchmaking rating
    tolerance: {
      initial: 50, // MMR points
      expansion: {
        enabled: true,
        interval: 15000, // 15 seconds
        increment: 25, // Expand by 25 MMR each interval
        maxExpansion: 200 // Max 200 MMR difference
      }
    }
  },

  secondary: {
    metric: 'VisibleRank',
    tolerance: {
      maxTierDifference: 1, // Can only match 1 tier apart
      sameDivisionPreferred: true
    }
  },

  mmrSystem: {
    // Hidden MMR separate from visible RP
    initial: 1000,
    kFactor: {
      placement: 16,
      newPlayer: 32, // Higher K for first 30 games
      established: 16
    },
    calculation: 'Modified Elo with placement consideration'
  },

  queueRules: {
    minPlayersToStart: 50, // Minimum lobby size
    maxWaitTime: 180000, // 3 minutes
    botFill: false, // No bots in ranked
    fallback: {
      onTimeout: 'Return to lobby with message',
      offPeakHours: 'Wider MMR tolerance after 2 minutes'
    }
  },

  partyRestrictions: {
    duo: {
      maxRankDifference: 2, // Max 2 divisions apart
      maxTierDifference: 1, // Cannot queue Gold + Diamond
      exception: 'First 10 placement games ignore restrictions'
    }
  }
};
```

### MMR Calculation

```typescript
const MMR_SYSTEM = {
  calculation: {
    // MMR changes based on expected vs actual performance
    expectedPlacement: (playerMMR: number, lobbyMMRs: number[]): number => {
      // Calculate expected placement based on MMR distribution
      const playerRank = lobbyMMRs.filter(mmr => mmr > playerMMR).length + 1;
      return playerRank;
    },

    mmrChange: (expected: number, actual: number, kFactor: number): number => {
      // Higher gain for exceeding expectations
      const difference = expected - actual;
      const change = kFactor * (difference / 10); // Scaled
      return Math.round(change);
    }
  },

  convergence: {
    // MMR and RP should roughly align over time
    checkInterval: 50, // Every 50 games
    adjustment: {
      mmrTooHigh: 'Slightly reduce gains until aligned',
      mmrTooLow: 'Slightly increase gains until aligned'
    }
  },

  decay: {
    enabled: true,
    inactivityPeriod: 7, // Days
    decayPerDay: 25, // MMR/RP loss per day
    minimumRank: 'sailor_iv', // Cannot decay below Silver IV
    reactivation: {
      games: 1, // 1 game to stop decay
      immediateStop: true
    }
  }
};
```

---

## Seasonal System

### Season Structure

```typescript
const SEASON_CONFIG = {
  duration: {
    length: 60, // Days
    splitStructure: {
      split1: 30, // Days
      split2: 30  // Days
    }
  },

  timeline: {
    preseason: {
      duration: 3, // Days
      features: ['Rank reset preview', 'Practice mode', 'Last season wrap-up']
    },
    season: {
      duration: 60,
      features: ['Full ranked enabled', 'Split rewards at midpoint']
    },
    endSeason: {
      duration: 2,
      features: ['Rank lock', 'Reward distribution', 'Stats finalization']
    }
  },

  rewards: {
    timing: {
      splitRewards: 'After each 30-day split',
      seasonRewards: 'At season end',
      peakRewards: 'Based on highest rank achieved'
    }
  }
};
```

### Seasonal Reset

```typescript
const SEASON_RESET = {
  type: 'soft', // Soft reset - partial RP reduction

  calculation: {
    method: 'Compressed toward median',
    formula: (currentRP: number): number => {
      const median = 800; // Gold IV equivalent
      const resetRP = median + (currentRP - median) * 0.5;
      return Math.max(0, Math.round(resetRP));
    },

    examples: [
      { current: 'Pirate King (3500 RP)', newRP: 2150, newRank: 'Captain II' },
      { current: 'Admiral (2800 RP)', newRP: 1800, newRank: 'First Mate II' },
      { current: 'Captain (2200 RP)', newRP: 1500, newRank: 'First Mate IV' },
      { current: 'First Mate (1500 RP)', newRP: 1150, newRank: 'Boatswain II' },
      { current: 'Boatswain (1000 RP)', newRP: 900, newRank: 'Boatswain IV' },
      { current: 'Sailor (600 RP)', newRP: 700, newRank: 'Sailor II' },
      { current: 'Deckhand (200 RP)', newRP: 500, newRank: 'Sailor III' }
    ]
  },

  placementGames: {
    required: 10,
    description: 'First 10 games of season determine starting rank',
    bonusRP: {
      enabled: true,
      perWin: 20, // Extra RP per win during placements
      maxBonus: 100
    },
    rankDisplay: 'Provisional rank shown after 5 games'
  },

  mmrReset: {
    type: 'partial',
    compression: 0.7 // MMR compressed 30% toward median
  }
};
```

---

## Season Rewards

### Reward Tiers

```typescript
const SEASON_REWARDS = {
  peakRankBased: true, // Rewards based on highest rank achieved, not end rank

  byTier: {
    deckhand: {
      frame: 'Bronze Season Frame',
      currency: 100, // Premium currency
      title: null
    },
    sailor: {
      frame: 'Silver Season Frame',
      currency: 200,
      title: 'Deck Swabber',
      emote: null
    },
    boatswain: {
      frame: 'Gold Season Frame',
      currency: 400,
      title: 'Sea Worthy',
      emote: 'Golden Salute',
      trail: null
    },
    first_mate: {
      frame: 'Platinum Season Frame',
      currency: 600,
      title: 'Storm Chaser',
      emote: 'Platinum Salute',
      trail: 'Platinum Wake',
      skin: null
    },
    captain: {
      frame: 'Diamond Season Frame (Animated)',
      currency: 1000,
      title: 'Legendary Captain',
      emote: 'Diamond Salute',
      trail: 'Diamond Wake',
      skin: 'Diamond Captain (weapon skin)'
    },
    admiral: {
      frame: 'Master Season Frame (Animated)',
      currency: 1500,
      title: 'Grand Admiral',
      emote: 'Admiral Salute',
      trail: 'Master Wake (Animated)',
      skin: 'Admiral Regalia (outfit)',
      exclusive: 'Season emblem'
    },
    pirate_king: {
      frame: 'Pirate King Frame (Animated + Effects)',
      currency: 2500,
      title: 'Pirate King Season X',
      emote: 'Royal Decree',
      trail: 'Legendary Wake',
      skin: 'Pirate King Regalia (full set)',
      exclusive: ['Leaderboard trophy', 'In-game statue recognition']
    }
  },

  splitRewards: {
    enabled: true,
    description: 'Smaller rewards at season midpoint',
    scaling: 0.3 // 30% of full season rewards
  },

  exclusivity: {
    seasonal: true,
    description: 'Season rewards never return in same form',
    variants: 'Similar rewards with different colors in future seasons'
  }
};
```

### Reward Claiming

```typescript
const REWARD_CLAIMING = {
  automatic: true, // Rewards auto-claimed at season end

  notification: {
    inGame: 'Popup showing all earned rewards',
    lobby: 'Season recap screen with stats',
    push: 'Push notification for reward availability'
  },

  display: {
    seasonHistory: {
      location: 'Profile > Ranked History',
      shows: ['Peak rank', 'Final rank', 'Rewards earned', 'Stats']
    },
    currentSeason: {
      location: 'Ranked lobby',
      shows: ['Current rank', 'RP progress', 'Projected rewards']
    }
  }
};
```

---

## Ranked Exclusives

### Exclusive Cosmetics

```typescript
const RANKED_EXCLUSIVES = {
  frameBorders: {
    description: 'Animated profile frames showing rank',
    levels: 'One per tier + seasonal variants',
    visibility: 'Shown in lobby, match, and friends list'
  },

  titles: {
    description: 'Text titles displayed with name',
    examples: [
      'Deck Swabber', 'Sea Worthy', 'Storm Chaser',
      'Legendary Captain', 'Grand Admiral', 'Pirate King'
    ],
    customizable: true // Player can choose which earned title to display
  },

  emotes: {
    description: 'Exclusive in-match emotes',
    rankLocked: true, // Only usable at certain ranks
    permanent: 'Seasonal emotes kept forever'
  },

  trails: {
    description: 'Visual effects during drop-in',
    visibility: 'Seen by all players during match',
    tiers: ['Gold', 'Platinum', 'Diamond', 'Master', 'Pirate King']
  },

  skins: {
    description: 'Exclusive weapon and outfit skins',
    availability: 'Platinum+ ranks',
    quality: 'Increases with rank tier'
  },

  charms: {
    description: 'Small icons attached to weapons',
    seasonal: true,
    collectible: 'Different charm each season'
  }
};
```

---

## Entry Requirements

### Minimum Requirements

```typescript
const RANKED_REQUIREMENTS = {
  accountLevel: {
    minimum: 10,
    reason: 'Ensures basic game understanding'
  },

  matchesPlayed: {
    minimum: 20,
    mode: 'Casual solo or duo',
    reason: 'Prevents smurfs and ensures competence'
  },

  tutorialComplete: {
    required: true,
    specificModules: ['Advanced combat', 'Ability combos']
  },

  accountAge: {
    minimum: 3, // Days
    reason: 'Anti-smurf measure'
  },

  verification: {
    optional: true,
    benefits: 'Verified accounts get priority queue',
    methods: ['Phone number', 'Social link']
  }
};
```

---

## Penalties and Fair Play

### AFK and Leaving Penalties

```typescript
const RANKED_PENALTIES = {
  leaving: {
    detection: {
      disconnected: 'Grace period of 60 seconds to reconnect',
      intentional: 'Closing app without disconnect',
      afk: 'No input for 120 seconds'
    },

    penalties: {
      first: {
        rpLoss: 50,
        cooldown: 0, // Warning only
        notification: 'Warning: Leaving ranked matches results in penalties'
      },
      second: {
        rpLoss: 75,
        cooldown: 5, // 5 minute queue ban
        notification: 'Penalty applied'
      },
      third: {
        rpLoss: 100,
        cooldown: 30, // 30 minute queue ban
        notification: 'Repeated leaving detected'
      },
      repeated: {
        rpLoss: 150,
        cooldown: 120, // 2 hour queue ban
        escalation: 'Each subsequent offense doubles cooldown',
        maximum: 1440 // 24 hour max
      }
    },

    forgiveness: {
      decayPeriod: 7, // Days
      cleanSlate: 'Penalty tier resets after 7 days without offense'
    }
  },

  afk: {
    warning: {
      threshold: 60, // Seconds
      notification: 'You will be removed for inactivity'
    },
    removal: {
      threshold: 120, // Seconds
      treatment: 'Same as leaving'
    }
  },

  partyPenalties: {
    enabled: true,
    description: 'Party members share lighter penalties when teammate leaves',
    rpLoss: 25, // Reduced RP loss for innocent party members
    cooldown: 0 // No queue ban for party members
  }
};
```

### Anti-Cheat and Smurf Detection

```typescript
const RANKED_INTEGRITY = {
  antiCheat: {
    enhanced: true,
    description: 'Stricter anti-cheat in ranked mode',
    measures: [
      'Real-time input validation',
      'Server-authoritative hit detection',
      'Position verification',
      'Ability cooldown enforcement'
    ],
    penalties: {
      detected: 'Immediate removal from match + RP loss',
      confirmed: 'Season ban + rank reset',
      repeated: 'Permanent account ban'
    }
  },

  smurfDetection: {
    metrics: [
      'Unusual win rate at low ranks',
      'High kill averages for rank tier',
      'Rapid rank progression',
      'Account age vs performance correlation'
    },
    action: {
      suspected: 'Accelerated MMR adjustment',
      confirmed: 'Automatic placement in higher MMR lobbies'
    }
  },

  boosting: {
    detection: [
      'Unusual duo queue patterns',
      'Significant performance change with specific party member',
      'Account sharing indicators'
    ],
    penalties: {
      boostee: 'Season rank reset',
      booster: 'Season ban'
    }
  }
};
```

---

## UI and Visibility

### Rank Display

```typescript
const RANK_UI = {
  profile: {
    currentRank: {
      icon: 'Tier emblem',
      division: 'Roman numeral',
      rp: 'Current RP / Next division RP',
      progressBar: 'Visual progress to next division'
    },
    peakRank: {
      display: 'Highest rank achieved this season',
      icon: 'Small emblem with crown indicator'
    },
    seasonHistory: {
      display: 'Past 5 seasons with peak ranks',
      expandable: true
    }
  },

  lobby: {
    partyRanks: 'Visible to party members',
    restrictionIndicator: 'Show if party has rank restriction issues'
  },

  inMatch: {
    playerRanks: {
      visibility: 'Hidden during match (anti-toxicity)',
      revealOnEliminate: 'Show rank of player you eliminated',
      endScreen: 'All ranks visible on final scoreboard'
    }
  },

  leaderboard: {
    regional: {
      display: 'Top 200 players',
      refresh: 'Real-time for top 100, hourly for 101-200'
    },
    global: {
      display: 'Top 500 global',
      refresh: 'Daily'
    },
    friends: {
      display: 'All friends with ranks',
      sorting: 'By rank or alphabetical'
    }
  }
};
```

### Progress Animations

```typescript
const RANK_ANIMATIONS = {
  promotion: {
    division: {
      animation: 'Progress bar fills + flash',
      duration: 2000,
      sound: 'Promotion fanfare (small)'
    },
    tier: {
      animation: 'Full screen celebration + new emblem reveal',
      duration: 4000,
      sound: 'Tier promotion fanfare (epic)',
      confetti: true
    }
  },

  demotion: {
    division: {
      animation: 'Progress bar depletes + dim flash',
      duration: 1500,
      sound: 'Subtle negative tone'
    },
    tier: {
      animation: 'Emblem cracks + falls',
      duration: 2500,
      sound: 'Demotion sound (serious but not punishing)',
      message: 'Encouraging message about climbing back'
    }
  },

  matchComplete: {
    rpChange: {
      animation: 'Numbers count up/down',
      breakdown: 'Show placement RP, kill RP, entry cost separately'
    }
  }
};
```

---

## Performance and Infrastructure

### Server Requirements

```typescript
const RANKED_INFRASTRUCTURE = {
  serverPriority: {
    ranked: 'High priority server allocation',
    tickRate: 64, // Higher tick rate for ranked
    region: 'Stricter region matching (lower latency)'
  },

  dataSync: {
    rpUpdates: 'Immediate on match end',
    leaderboard: 'Near real-time for top players',
    crossPlatform: 'Unified ranked across all platforms'
  },

  reliability: {
    serverCrash: {
      detection: 'Automatic crash detection',
      handling: 'No RP loss for players in crashed matches',
      notification: 'Explain situation to affected players'
    },
    networkIssues: {
      detection: 'Detect widespread connectivity problems',
      handling: 'Pause ranked queue temporarily',
      rollback: 'Rollback RP changes for affected matches'
    }
  }
};
```

### Analytics

```typescript
const RANKED_ANALYTICS = {
  events: [
    {
      name: 'ranked_match_complete',
      properties: {
        placement: 'number',
        rp_change: 'number',
        kills: 'number',
        tier_before: 'string',
        tier_after: 'string'
      }
    },
    {
      name: 'rank_promotion',
      properties: {
        from_tier: 'string',
        to_tier: 'string',
        games_in_tier: 'number'
      }
    },
    {
      name: 'rank_demotion',
      properties: {
        from_tier: 'string',
        to_tier: 'string',
        cause: 'loss | decay | penalty'
      }
    },
    {
      name: 'season_end',
      properties: {
        peak_rank: 'string',
        final_rank: 'string',
        games_played: 'number',
        win_rate: 'number'
      }
    }
  ],

  metrics: [
    'Rank distribution across tiers',
    'Average games to reach each tier',
    'Season engagement (games per player)',
    'Penalty rate and types',
    'Queue times by rank tier',
    'Match quality (MMR spread in matches)'
  ]
};
```

---

## Testing Requirements

```typescript
const RANKED_TESTING = {
  functional: [
    'RP calculation is accurate for all placements',
    'Rank promotions and demotions trigger correctly',
    'SBMM creates balanced matches',
    'Season reset applies correct RP compression',
    'Placement games determine starting rank properly',
    'Entry costs are deducted correctly'
  ],

  integrity: [
    'Cannot queue with too large rank difference',
    'Penalties apply correctly for leavers',
    'Anti-cheat detects common exploits',
    'Smurf detection accelerates MMR appropriately'
  ],

  progression: [
    'Rewards distribute at season end',
    'Peak rank tracking is accurate',
    'Leaderboards update correctly',
    'Decay applies after inactivity period'
  ],

  ux: [
    'Rank UI is clear and informative',
    'Promotion/demotion animations play correctly',
    'RP breakdown is understandable',
    'Season timeline displays accurately'
  ],

  performance: [
    'Matchmaking completes within acceptable time',
    'RP updates are immediate after match',
    'Leaderboard queries are performant',
    'No RP desync between client and server'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial ranked mode specification |
