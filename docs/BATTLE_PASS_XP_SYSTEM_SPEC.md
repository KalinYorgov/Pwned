# Battle Pass XP System Specification

## Overview

This document defines the XP earning system for Battle Pass progression in Plunderstorm Mobile, ensuring balanced progression that rewards regular play while remaining achievable for dedicated players.

## XP System Philosophy

### Design Goals
- **Achievability**: Regular players complete pass in 6-8 weeks
- **Engagement**: Daily play rewarded, but not required
- **Fairness**: Skill contributes but doesn't dominate
- **Flexibility**: Multiple paths to earn XP
- **Monetization**: Tier skips available but not necessary

### Progression Math

```typescript
interface ProgressionMath {
  tierRequirements: TierRequirement;
  seasonCompletion: SeasonCompletion;
}

const PROGRESSION_MATH: ProgressionMath = {
  tierRequirements: {
    xpPerTier: 10000,
    totalTiers: 60,
    totalXpRequired: 600000
  },

  seasonCompletion: {
    seasonDuration: {
      weeks: 8,
      days: 56
    },
    dailyXpRequired: {
      forFullCompletion: 10714, // 600,000 / 56
      casual: 7500, // ~75% completion
      minimum: 5000 // ~50% completion
    },
    weeklyXpRequired: {
      forFullCompletion: 75000,
      casual: 52500,
      minimum: 35000
    }
  }
};
```

---

## XP Sources

### Match Completion XP

```typescript
interface MatchXP {
  baseSources: MatchXPSource[];
  bonuses: MatchXPBonus[];
}

const MATCH_XP: MatchXP = {
  baseSources: [
    {
      source: 'Match Completion',
      description: 'XP earned just for finishing a match',
      baseAmount: 50,
      notes: 'Earned regardless of placement'
    },
    {
      source: 'Placement Bonus',
      description: 'Additional XP based on final placement',
      amounts: [
        { placement: 1, xp: 150 },        // Total: 200
        { placement: [2, 3], xp: 120 },   // Total: 170
        { placement: [4, 5], xp: 100 },   // Total: 150
        { placement: [6, 10], xp: 80 },   // Total: 130
        { placement: [11, 20], xp: 60 },  // Total: 110
        { placement: [21, 30], xp: 40 },  // Total: 90
        { placement: [31, 60], xp: 20 }   // Total: 70
      ]
    },
    {
      source: 'Survival Time',
      description: 'XP for time survived in match',
      rate: '2 XP per minute survived',
      cap: 30, // 15 minutes max
      notes: 'Rewards engagement even without high placement'
    },
    {
      source: 'Eliminations',
      description: 'XP for eliminating opponents',
      amount: 10,
      perElimination: true,
      cap: 100, // 10 eliminations max counted
      notes: 'Rewards aggressive play style'
    }
  ],

  bonuses: [
    {
      bonus: 'First Match of Day',
      multiplier: 1.5,
      duration: 'First match after daily reset',
      description: 'Encourages daily login'
    },
    {
      bonus: 'Premium Pass Bonus',
      multiplier: 1.1,
      duration: 'Permanent for pass owners',
      description: '+10% XP on all match sources'
    }
  ]
};
```

### Match XP Calculation Examples

```typescript
const MATCH_XP_EXAMPLES: MatchXPExample[] = [
  {
    scenario: 'Victory with 5 eliminations, 12 min survival',
    calculation: {
      baseCompletion: 50,
      placementBonus: 150, // 1st place
      survivalTime: 24, // 12 * 2
      eliminations: 50, // 5 * 10
      subtotal: 274,
      firstMatchBonus: '×1.5 = 411',
      premiumBonus: '×1.1 = 302 (or 452 with first match)'
    },
    totalXP: {
      freePlayer: 274,
      freeFirstMatch: 411,
      premiumPlayer: 302,
      premiumFirstMatch: 452
    }
  },
  {
    scenario: 'Top 10 with 2 eliminations, 8 min survival',
    calculation: {
      baseCompletion: 50,
      placementBonus: 80, // 6th-10th
      survivalTime: 16, // 8 * 2
      eliminations: 20, // 2 * 10
      subtotal: 166
    },
    totalXP: {
      freePlayer: 166,
      premiumPlayer: 183
    }
  },
  {
    scenario: 'Early elimination (30th place), 3 min survival, 0 kills',
    calculation: {
      baseCompletion: 50,
      placementBonus: 40, // 21st-30th
      survivalTime: 6, // 3 * 2
      eliminations: 0,
      subtotal: 96
    },
    totalXP: {
      freePlayer: 96,
      premiumPlayer: 106
    }
  }
];
```

### Daily Quest XP

```typescript
interface DailyQuestXP {
  questSlots: number;
  questTypes: QuestType[];
  refreshSchedule: RefreshSchedule;
}

const DAILY_QUEST_XP: DailyQuestXP = {
  questSlots: 3,
  refreshTime: '00:00 UTC',
  carryOver: false, // Must complete same day

  questTypes: [
    {
      difficulty: 'Easy',
      xpReward: 300,
      examples: [
        'Play 2 matches',
        'Deal 500 damage',
        'Survive for 5 minutes total',
        'Open 3 supply crates'
      ],
      frequency: 'Always 1 per day'
    },
    {
      difficulty: 'Medium',
      xpReward: 500,
      examples: [
        'Play 4 matches',
        'Eliminate 3 opponents',
        'Finish in Top 20 twice',
        'Deal 1500 damage'
      ],
      frequency: 'Always 1 per day'
    },
    {
      difficulty: 'Hard',
      xpReward: 750,
      examples: [
        'Win a match',
        'Finish in Top 5 twice',
        'Eliminate 6 opponents',
        'Deal 3000 damage'
      ],
      frequency: 'Always 1 per day'
    }
  ],

  totalDailyQuestXP: {
    all: 1550, // 300 + 500 + 750
    easy: 300,
    easyAndMedium: 800
  },

  refreshSchedule: {
    time: '00:00 UTC',
    notification: 'New daily quests available!',
    gracePeriod: 'None - complete or lose'
  }
};
```

### Weekly Quest XP

```typescript
interface WeeklyQuestXP {
  questSlots: number;
  questTypes: QuestType[];
  refreshSchedule: RefreshSchedule;
}

const WEEKLY_QUEST_XP: WeeklyQuestXP = {
  questSlots: 3,
  refreshDay: 'Tuesday',
  refreshTime: '00:00 UTC',
  carryOver: false,

  questTypes: [
    {
      difficulty: 'Standard',
      xpReward: 1500,
      examples: [
        'Play 15 matches',
        'Eliminate 15 opponents',
        'Deal 10,000 damage total',
        'Finish in Top 10 five times'
      ],
      frequency: 'Always 1 per week'
    },
    {
      difficulty: 'Challenging',
      xpReward: 2500,
      examples: [
        'Win 2 matches',
        'Finish in Top 5 five times',
        'Eliminate 25 opponents',
        'Survive 60 minutes total'
      ],
      frequency: 'Always 1 per week'
    },
    {
      difficulty: 'Elite',
      xpReward: 4000,
      examples: [
        'Win 5 matches',
        'Finish in Top 3 eight times',
        'Eliminate 50 opponents',
        'Deal 25,000 damage total'
      ],
      frequency: 'Always 1 per week'
    }
  ],

  totalWeeklyQuestXP: {
    all: 8000, // 1500 + 2500 + 4000
    standardOnly: 1500,
    standardAndChallenging: 4000
  },

  refreshSchedule: {
    day: 'Tuesday',
    time: '00:00 UTC',
    notification: 'New weekly challenges!',
    reminderAt: '24 hours before reset'
  }
};
```

### Special Event XP

```typescript
interface EventXP {
  eventTypes: EventXPType[];
}

const EVENT_XP: EventXP = {
  eventTypes: [
    {
      type: 'Double XP Weekend',
      multiplier: 2.0,
      appliesTo: 'Match XP only (not quests)',
      frequency: 'Once per season (typically mid-season)',
      duration: 'Friday 00:00 to Monday 00:00 UTC',
      notification: 'Major push notification + in-game banner'
    },
    {
      type: 'XP Happy Hour',
      multiplier: 1.5,
      appliesTo: 'All XP sources',
      frequency: '2-3 times per season',
      duration: '4 hours',
      notification: 'In-game banner + push'
    },
    {
      type: 'Season Finale Boost',
      multiplier: 1.25,
      appliesTo: 'All XP sources',
      frequency: 'Final week of season',
      duration: '7 days',
      notification: 'Help players catch up'
    },
    {
      type: 'Limited Time Event',
      bonusXP: 'Event-specific quests',
      amount: 'Variable (2000-5000 bonus)',
      frequency: 'During special events',
      duration: 'Event duration'
    }
  ]
};
```

---

## Weekly XP Budget Analysis

### Average Player Progression

```typescript
interface WeeklyXPBudget {
  playerProfiles: PlayerProfile[];
}

const WEEKLY_XP_BUDGET: WeeklyXPBudget = {
  playerProfiles: [
    {
      profile: 'Casual',
      playPattern: {
        matchesPerDay: 2,
        daysPerWeek: 4,
        matchesPerWeek: 8
      },
      questCompletion: {
        dailyQuests: '2 of 3 (easy + medium)',
        weeklyQuests: '1 of 3 (standard)',
        daysPlayed: 4
      },
      weeklyXP: {
        matchXP: 8 * 130, // Avg 130 XP/match
        dailyQuests: 4 * 800, // 4 days × easy+medium
        weeklyQuests: 1500, // Standard only
        total: 1040 + 3200 + 1500 // = 5,740
      },
      seasonProgression: {
        weeklyXP: 5740,
        totalIn8Weeks: 45920,
        tiersEarned: 4.6, // Per week
        seasonCompletion: '~77%'
      }
    },
    {
      profile: 'Regular',
      playPattern: {
        matchesPerDay: 4,
        daysPerWeek: 6,
        matchesPerWeek: 24
      },
      questCompletion: {
        dailyQuests: 'All 3',
        weeklyQuests: '2 of 3',
        daysPlayed: 6
      },
      weeklyXP: {
        matchXP: 24 * 150, // Avg 150 XP/match (better placement)
        dailyQuests: 6 * 1550, // 6 days × all quests
        weeklyQuests: 4000, // Standard + Challenging
        total: 3600 + 9300 + 4000 // = 16,900
      },
      seasonProgression: {
        weeklyXP: 16900,
        totalIn8Weeks: 135200,
        tiersEarned: 10.7, // Per week (will cap at 60)
        seasonCompletion: '100%+ (complete in ~5 weeks)'
      }
    },
    {
      profile: 'Hardcore',
      playPattern: {
        matchesPerDay: 8,
        daysPerWeek: 7,
        matchesPerWeek: 56
      },
      questCompletion: {
        dailyQuests: 'All 3',
        weeklyQuests: 'All 3',
        daysPlayed: 7
      },
      weeklyXP: {
        matchXP: 56 * 180, // Avg 180 XP/match (high skill)
        dailyQuests: 7 * 1550, // 7 days × all quests
        weeklyQuests: 8000, // All weekly quests
        total: 10080 + 10850 + 8000 // = 28,930
      },
      seasonProgression: {
        weeklyXP: 28930,
        totalIn8Weeks: 231440,
        tiersEarned: 18.4, // Per week (complete in ~3.5 weeks)
        seasonCompletion: '100% (complete ~Week 4)'
      }
    }
  ]
};
```

---

## XP Display System

### Match Summary XP Display

```
┌─────────────────────────────────────────────────────────────┐
│                    MATCH COMPLETE                           │
│                   🥈 2nd Place!                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  BATTLE PASS XP EARNED                                      │
│  ─────────────────────────────────────────                  │
│                                                             │
│  Match Completion .................... +50 XP               │
│  Placement (2nd) ..................... +120 XP              │
│  Survival (11 min) ................... +22 XP               │
│  Eliminations (×4) ................... +40 XP               │
│                                       ────────              │
│  Subtotal ............................ 232 XP               │
│                                                             │
│  ✨ First Match Bonus (×1.5) ......... +116 XP              │
│  🎫 Premium Pass Bonus (+10%) ........ +35 XP               │
│                                       ════════              │
│  TOTAL XP ............................ 383 XP               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  BATTLE PASS PROGRESS                                       │
│  Tier 23 ████████████░░░░░░░░ 7,850/10,000 XP              │
│                                                             │
│  +383 XP this match  |  2,150 XP to Tier 24                │
│                                                             │
│  [VIEW BATTLE PASS]                 [CONTINUE]             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Quest Progress Display

```
┌─────────────────────────────────────────────────────────────┐
│  DAILY QUESTS                          Resets in 8h 32m    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Play 2 matches ...................... 300 XP [CLAIMED] │
│     ████████████████████ 2/2                               │
│                                                             │
│  ⏳ Eliminate 3 opponents ............... 500 XP           │
│     ████████████░░░░░░░░ 2/3                               │
│                                                             │
│  ⏳ Win a match ......................... 750 XP           │
│     ░░░░░░░░░░░░░░░░░░░░ 0/1                               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  WEEKLY QUESTS                         Resets in 3d 8h     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ⏳ Play 15 matches ..................... 1,500 XP         │
│     ████████████████░░░░ 12/15                             │
│                                                             │
│  ⏳ Win 2 matches ....................... 2,500 XP         │
│     ██████████░░░░░░░░░░ 1/2                               │
│                                                             │
│  ⏳ Win 5 matches ....................... 4,000 XP         │
│     ████░░░░░░░░░░░░░░░░ 1/5                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Battle Pass Progress Bar

```typescript
interface ProgressBarDisplay {
  elements: ProgressElement[];
  animations: ProgressAnimation[];
}

const PROGRESS_BAR: ProgressBarDisplay = {
  elements: [
    {
      element: 'Tier Counter',
      format: 'Tier {current} / 60',
      position: 'Left of progress bar'
    },
    {
      element: 'XP Progress',
      format: '{currentXP} / {tierXP} XP',
      position: 'Below progress bar'
    },
    {
      element: 'Progress Fill',
      visual: 'Gradient gold bar',
      animation: 'Fill on XP gain'
    },
    {
      element: 'Next Tier Preview',
      visual: 'Reward icon at bar end',
      tap: 'Opens reward preview'
    }
  ],

  animations: [
    {
      trigger: 'XP Gained',
      animation: 'Progress bar fills smoothly',
      duration: '500ms',
      easing: 'easeOut'
    },
    {
      trigger: 'Tier Complete',
      animation: 'Bar fills to 100%, bursts, resets',
      duration: '1200ms',
      sound: 'tier_up.wav'
    },
    {
      trigger: 'Tier Up',
      animation: 'Counter increments with pop',
      particles: 'golden_sparkles'
    }
  ]
};
```

---

## XP Boost System

### XP Boost Items

```typescript
interface XPBoostSystem {
  boostTypes: XPBoost[];
  stacking: StackingRules;
}

const XP_BOOST_SYSTEM: XPBoostSystem = {
  boostTypes: [
    {
      boostId: 'xp_boost_1hr',
      name: 'XP Boost (1 Hour)',
      duration: 3600, // seconds
      multiplier: 1.5, // +50%
      appliesTo: 'All XP sources',
      source: 'Battle Pass rewards, special events',
      activationMethod: 'Manual from inventory',
      stackable: false, // Cannot use multiple
      pauseOnLogout: true // Time only counts during active play
    },
    {
      boostId: 'xp_boost_24hr',
      name: 'XP Boost (24 Hours)',
      duration: 86400,
      multiplier: 1.25, // +25%
      appliesTo: 'All XP sources',
      source: 'Battle Pass premium track',
      activationMethod: 'Manual from inventory',
      stackable: false,
      pauseOnLogout: false // Real time countdown
    },
    {
      boostId: 'premium_pass_bonus',
      name: 'Premium Pass Bonus',
      duration: 'Season duration',
      multiplier: 1.1, // +10%
      appliesTo: 'Match XP only',
      source: 'Battle Pass ownership',
      activationMethod: 'Automatic',
      stackable: true // Stacks with other boosts
    }
  ],

  stacking: {
    rules: [
      'Only one timed boost active at a time',
      'Premium Pass bonus always applies',
      'Event multipliers stack with boosts',
      'Order: Base × Event × Boost × Premium'
    ],
    example: {
      scenario: 'Double XP event + 1hr Boost + Premium Pass',
      calculation: '100 × 2.0 × 1.5 × 1.1 = 330 XP'
    }
  }
};
```

### Boost Activation UI

```
┌─────────────────────────────────────────────────────────────┐
│                   ACTIVATE XP BOOST                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│              ⚡ XP BOOST (1 HOUR) ⚡                        │
│                                                             │
│              [Boost Icon Animation]                        │
│                                                             │
│         Earn +50% Battle Pass XP for 1 hour!               │
│                                                             │
│         • Applies to all XP sources                        │
│         • Timer pauses when not playing                    │
│         • Stacks with event bonuses                        │
│                                                             │
│         You have: 3 remaining                              │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────┐    ┌──────────────────┐              │
│  │     CANCEL       │    │    ACTIVATE      │              │
│  └──────────────────┘    └──────────────────┘              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Active Boost Indicator

```typescript
const BOOST_INDICATOR: BoostIndicator = {
  location: 'Top right of screen, below currency',
  display: {
    icon: 'boost_icon.png',
    timer: 'Countdown remaining',
    format: '⚡ 45:32'
  },
  colors: {
    active: 'Electric blue',
    expiring: 'Warning yellow (<5 min)',
    expired: 'Fade out animation'
  },
  tapAction: 'Open boost details/extend option'
};
```

---

## Tier Skip Purchase

### Skip Options

```typescript
interface TierSkipPurchase {
  options: SkipOption[];
  ui: SkipPurchaseUI;
}

const TIER_SKIP: TierSkipPurchase = {
  options: [
    {
      quantity: 1,
      price: 150,
      currency: 'Doubloons',
      pricePerTier: 150,
      discount: '0%'
    },
    {
      quantity: 5,
      price: 650,
      currency: 'Doubloons',
      pricePerTier: 130,
      discount: '13%',
      badge: null
    },
    {
      quantity: 10,
      price: 1200,
      currency: 'Doubloons',
      pricePerTier: 120,
      discount: '20%',
      badge: 'POPULAR'
    },
    {
      quantity: 25,
      price: 2750,
      currency: 'Doubloons',
      pricePerTier: 110,
      discount: '27%',
      badge: 'BEST VALUE'
    }
  ],

  ui: {
    accessPoint: 'Battle Pass screen "Buy Tiers" button',
    requirement: 'Must own Premium Pass',
    preview: 'Show rewards that will be unlocked',
    maxPurchase: 'Cannot exceed Tier 60',
    adjustQuantity: 'Auto-adjust to remaining tiers'
  }
};
```

### Skip Purchase Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    SKIP TO TIER 35                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Current: Tier 25  →  Target: Tier 35                      │
│  Tiers to skip: 10                                          │
│                                                             │
│  REWARDS YOU'LL UNLOCK:                                     │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                   │
│  │ 💎  │ │ 🎭  │ │ 👕  │ │ 🪙  │ │ ⭐  │  +5 more        │
│  │ 100 │ │Emote│ │Frame│ │1000 │ │Boost│                   │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                   │
│                                                             │
│  ─────────────────────────────────────────                  │
│                                                             │
│  Cost: 💎 1,200  (20% savings!)                            │
│  Your Balance: 💎 2,450                                    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────┐    ┌──────────────────┐              │
│  │     CANCEL       │    │    PURCHASE      │              │
│  └──────────────────┘    └──────────────────┘              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## XP Data Tracking

### Player XP State

```typescript
interface PlayerXPState {
  playerId: string;
  seasonId: string;

  progression: {
    totalXpEarned: number;
    currentTier: number;
    xpInCurrentTier: number;
  };

  sources: {
    matchXP: number;
    dailyQuestXP: number;
    weeklyQuestXP: number;
    eventXP: number;
    bonusXP: number;
  };

  boosts: {
    activeBoost?: {
      boostId: string;
      expiresAt: Date;
      remainingTime: number;
    };
    boostsUsed: number;
    boostsRemaining: number;
  };

  quests: {
    dailyQuests: Quest[];
    weeklyQuests: Quest[];
    lastDailyReset: Date;
    lastWeeklyReset: Date;
  };

  tiersPurchased: number;
}
```

### Analytics Events

```typescript
const XP_ANALYTICS_EVENTS: AnalyticsEvent[] = [
  {
    event: 'xp_earned',
    properties: ['source', 'amount', 'total_xp', 'current_tier'],
    trigger: 'Any XP earned'
  },
  {
    event: 'tier_up',
    properties: ['new_tier', 'time_in_season', 'method'],
    trigger: 'Player reaches new tier'
  },
  {
    event: 'quest_completed',
    properties: ['quest_id', 'quest_type', 'xp_earned'],
    trigger: 'Quest completion'
  },
  {
    event: 'boost_activated',
    properties: ['boost_type', 'remaining_boosts'],
    trigger: 'Player activates XP boost'
  },
  {
    event: 'tiers_purchased',
    properties: ['quantity', 'cost', 'new_tier'],
    trigger: 'Player buys tier skips'
  },
  {
    event: 'pass_completion',
    properties: ['days_to_complete', 'tiers_purchased', 'total_xp'],
    trigger: 'Player reaches Tier 60'
  }
];
```

---

## Balance Considerations

### Completion Rate Targets

```typescript
interface CompletionTargets {
  targets: CompletionTarget[];
  adjustmentLevers: AdjustmentLever[];
}

const COMPLETION_TARGETS: CompletionTargets = {
  targets: [
    {
      playerSegment: 'All Premium Pass Owners',
      targetCompletion: 85,
      rationale: 'High completion drives satisfaction and rebuy'
    },
    {
      playerSegment: 'Daily Players',
      targetCompletion: 95,
      rationale: 'Reward dedication'
    },
    {
      playerSegment: 'Casual (3-4 days/week)',
      targetCompletion: 70,
      rationale: 'Still achievable with effort'
    },
    {
      playerSegment: 'F2P Players (Free Track)',
      targetCompletion: 80,
      rationale: 'Show pass value, drive conversion'
    }
  ],

  adjustmentLevers: [
    {
      lever: 'XP Per Tier',
      current: 10000,
      adjustRange: '8000-12000',
      impact: 'Overall pacing'
    },
    {
      lever: 'Quest XP',
      current: '1550 daily, 8000 weekly',
      impact: 'Rewards quest focus'
    },
    {
      lever: 'Match XP',
      current: '50-200 per match',
      impact: 'Grind intensity'
    },
    {
      lever: 'Double XP Events',
      current: '1 per season',
      impact: 'Catch-up opportunity'
    },
    {
      lever: 'Season Duration',
      current: '8 weeks',
      adjustRange: '6-10 weeks',
      impact: 'Time pressure'
    }
  ]
};
```

---

## Implementation Checklist

### XP Sources
- [ ] Match completion XP (base + placement + survival + kills)
- [ ] First match of day bonus
- [ ] Premium pass bonus
- [ ] Daily quests (3 slots)
- [ ] Weekly quests (3 slots)
- [ ] Event XP bonuses

### XP Display
- [ ] Match summary XP breakdown
- [ ] Quest progress UI
- [ ] Battle Pass progress bar
- [ ] Tier-up animation
- [ ] XP gain toast notifications

### XP Boosts
- [ ] Boost activation from inventory
- [ ] Active boost indicator
- [ ] Timer management (pause on logout for 1hr boost)
- [ ] Stacking rules implementation

### Tier Skips
- [ ] Skip purchase flow
- [ ] Quantity options (1, 5, 10, 25)
- [ ] Reward preview before purchase
- [ ] Max tier validation

### Analytics
- [ ] XP earning tracking by source
- [ ] Tier progression tracking
- [ ] Quest completion tracking
- [ ] Boost usage tracking
- [ ] Completion rate monitoring

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
