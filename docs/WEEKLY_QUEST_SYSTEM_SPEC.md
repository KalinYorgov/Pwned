# Weekly Quest System Specification

## Overview

The Weekly Quest System provides longer-term engagement goals with significantly larger rewards than daily quests. Weekly quests encourage sustained play throughout the week and contribute meaningfully to Battle Pass progression.

## Quest Slot Configuration

```typescript
const WEEKLY_QUEST_SLOTS: WeeklyQuestSlotConfig = {
  totalSlots: 7,
  slots: [
    {
      slotId: 'weekly_1',
      difficulty: 'Easy',
      reward: { gold: 300, battlePassXP: 1000 },
      category: 'matches'
    },
    {
      slotId: 'weekly_2',
      difficulty: 'Easy',
      reward: { gold: 350, battlePassXP: 1000 },
      category: 'loot'
    },
    {
      slotId: 'weekly_3',
      difficulty: 'Medium',
      reward: { gold: 400, battlePassXP: 1500 },
      category: 'combat'
    },
    {
      slotId: 'weekly_4',
      difficulty: 'Medium',
      reward: { gold: 450, battlePassXP: 1500 },
      category: 'survival'
    },
    {
      slotId: 'weekly_5',
      difficulty: 'Hard',
      reward: { gold: 500, battlePassXP: 2000 },
      category: 'wins'
    },
    {
      slotId: 'weekly_6',
      difficulty: 'Hard',
      reward: { gold: 500, battlePassXP: 2000 },
      category: 'abilities'
    },
    {
      slotId: 'weekly_7',
      difficulty: 'Challenge',
      reward: { gold: 600, doubloons: 5, battlePassXP: 2500, bonusChest: 'rare' },
      category: 'milestone'
    }
  ],
  totalRewards: {
    gold: 3100,
    doubloons: 5,
    battlePassXP: 11500,
    chests: ['Rare Chest']
  }
};
```

## Weekly Reset Schedule

```typescript
const WEEKLY_RESET_CONFIG = {
  resetDay: 'Monday',
  resetTime: '00:00 UTC',

  // Grace period for stragglers
  gracePeriod: {
    enabled: true,
    duration: '4 hours',
    notifyBefore: ['24h', '6h', '1h', '15min']
  },

  // Week cycle
  weekDuration: 7 * 24 * 60 * 60 * 1000, // 7 days in ms

  // Alignment with Battle Pass
  alignWithSeason: true, // Week 1 starts with season start
  weekNumber: 'calculated' // Week 1, Week 2, etc. of the season
};
```

## Quest Categories and Templates

### 1. Matches Category
```typescript
const WEEKLY_MATCHES_QUESTS = [
  {
    questId: 'w_matches_play_15',
    template: 'Play {count} matches this week',
    requirements: { matchesPlayed: 15 },
    difficulty: 'Easy',
    estimatedTime: '~3 hours total'
  },
  {
    questId: 'w_matches_play_25',
    template: 'Play {count} matches this week',
    requirements: { matchesPlayed: 25 },
    difficulty: 'Medium',
    estimatedTime: '~5 hours total'
  },
  {
    questId: 'w_matches_complete_20',
    template: 'Complete {count} full matches (no early quit)',
    requirements: { matchesCompleted: 20 },
    difficulty: 'Medium',
    estimatedTime: '~4 hours total'
  }
];
```

### 2. Loot Category
```typescript
const WEEKLY_LOOT_QUESTS = [
  {
    questId: 'w_loot_chests_30',
    template: 'Open {count} treasure chests',
    requirements: { chestsOpened: 30 },
    difficulty: 'Easy',
    estimatedTime: 'Passive across matches'
  },
  {
    questId: 'w_loot_chests_50',
    template: 'Open {count} treasure chests',
    requirements: { chestsOpened: 50 },
    difficulty: 'Medium',
    estimatedTime: 'Passive across matches'
  },
  {
    questId: 'w_loot_rare_items_10',
    template: 'Collect {count} rare or better items',
    requirements: { rareItemsCollected: 10 },
    difficulty: 'Medium',
    estimatedTime: 'RNG dependent'
  },
  {
    questId: 'w_loot_gold_collect_2000',
    template: 'Collect {count} in-match gold from loot',
    requirements: { matchGoldCollected: 2000 },
    difficulty: 'Easy',
    estimatedTime: 'Passive across matches'
  }
];
```

### 3. Combat Category
```typescript
const WEEKLY_COMBAT_QUESTS = [
  {
    questId: 'w_combat_elims_25',
    template: 'Get {count} eliminations',
    requirements: { eliminations: 25 },
    difficulty: 'Medium',
    estimatedTime: '~4 hours aggressive play'
  },
  {
    questId: 'w_combat_elims_40',
    template: 'Get {count} eliminations',
    requirements: { eliminations: 40 },
    difficulty: 'Hard',
    estimatedTime: '~6 hours aggressive play'
  },
  {
    questId: 'w_combat_damage_10000',
    template: 'Deal {count} damage to enemies',
    requirements: { damageDealt: 10000 },
    difficulty: 'Medium',
    estimatedTime: 'Passive across matches'
  },
  {
    questId: 'w_combat_damage_25000',
    template: 'Deal {count} damage to enemies',
    requirements: { damageDealt: 25000 },
    difficulty: 'Hard',
    estimatedTime: 'Passive across matches'
  },
  {
    questId: 'w_combat_knockdowns_30',
    template: 'Knock down {count} enemies',
    requirements: { knockdowns: 30 },
    difficulty: 'Medium',
    estimatedTime: '~4 hours'
  }
];
```

### 4. Survival Category
```typescript
const WEEKLY_SURVIVAL_QUESTS = [
  {
    questId: 'w_survival_top10_10',
    template: 'Finish in top 10 in {count} matches',
    requirements: { top10Finishes: 10 },
    difficulty: 'Medium',
    estimatedTime: '~3-4 hours'
  },
  {
    questId: 'w_survival_top5_7',
    template: 'Finish in top 5 in {count} matches',
    requirements: { top5Finishes: 7 },
    difficulty: 'Hard',
    estimatedTime: '~4-5 hours'
  },
  {
    questId: 'w_survival_minutes_300',
    template: 'Survive for {count} total minutes',
    requirements: { survivalMinutes: 300 },
    difficulty: 'Easy',
    estimatedTime: 'Passive'
  },
  {
    questId: 'w_survival_outlive_500',
    template: 'Outlive {count} opponents total',
    requirements: { opponentsOutlived: 500 },
    difficulty: 'Medium',
    estimatedTime: 'Passive across matches'
  }
];
```

### 5. Wins Category
```typescript
const WEEKLY_WINS_QUESTS = [
  {
    questId: 'w_wins_3',
    template: 'Win {count} matches this week',
    requirements: { wins: 3 },
    difficulty: 'Hard',
    estimatedTime: 'Skill dependent'
  },
  {
    questId: 'w_wins_5',
    template: 'Win {count} matches this week',
    requirements: { wins: 5 },
    difficulty: 'Challenge',
    estimatedTime: 'Skill dependent'
  },
  {
    questId: 'w_wins_top3_5',
    template: 'Finish in top 3 in {count} matches',
    requirements: { top3Finishes: 5 },
    difficulty: 'Hard',
    estimatedTime: '~3-4 hours'
  }
];
```

### 6. Abilities Category
```typescript
const WEEKLY_ABILITIES_QUESTS = [
  {
    questId: 'w_abilities_cast_100',
    template: 'Cast {count} abilities',
    requirements: { abilitiesCast: 100 },
    difficulty: 'Easy',
    estimatedTime: 'Passive'
  },
  {
    questId: 'w_abilities_offensive_50',
    template: 'Use {count} offensive abilities',
    requirements: { offensiveAbilitiesUsed: 50 },
    difficulty: 'Medium',
    estimatedTime: 'Passive'
  },
  {
    questId: 'w_abilities_utility_40',
    template: 'Use {count} utility abilities',
    requirements: { utilityAbilitiesUsed: 40 },
    difficulty: 'Medium',
    estimatedTime: 'Passive'
  },
  {
    questId: 'w_abilities_ability_kills_15',
    template: 'Get {count} ability eliminations',
    requirements: { abilityKills: 15 },
    difficulty: 'Hard',
    estimatedTime: '~4 hours'
  },
  {
    questId: 'w_abilities_upgrade_20',
    template: 'Upgrade abilities {count} times in matches',
    requirements: { abilityUpgrades: 20 },
    difficulty: 'Easy',
    estimatedTime: 'Passive'
  }
];
```

### 7. Milestone Category (Slot 7 - Challenge Quest)
```typescript
const WEEKLY_MILESTONE_QUESTS = [
  {
    questId: 'w_milestone_complete_all_daily',
    template: 'Complete all daily quests for {count} days',
    requirements: { dailyQuestCompleteDays: 5 },
    difficulty: 'Challenge',
    estimatedTime: 'Requires daily engagement'
  },
  {
    questId: 'w_milestone_multi_objective',
    template: 'Win a match, get 5+ eliminations, and open 10+ chests',
    requirements: {
      wins: 1,
      eliminations: 5,
      chestsOpened: 10,
      inSingleMatch: true
    },
    difficulty: 'Challenge',
    estimatedTime: 'Single exceptional match'
  },
  {
    questId: 'w_milestone_versatile',
    template: 'Use 6 different ability types this week',
    requirements: { uniqueAbilityTypesUsed: 6 },
    difficulty: 'Challenge',
    estimatedTime: 'Requires variety'
  },
  {
    questId: 'w_milestone_survivor',
    template: 'Survive to final 5 without any eliminations',
    requirements: {
      top5Finishes: 1,
      maxEliminations: 0,
      inSingleMatch: true
    },
    difficulty: 'Challenge',
    estimatedTime: 'Playstyle challenge'
  }
];
```

## Quest Assignment Algorithm

```typescript
interface WeeklyQuestAssignment {
  assignQuestsForPlayer(playerId: string, weekNumber: number): AssignedWeeklyQuests;
}

const WEEKLY_ASSIGNMENT_ALGORITHM = {
  // Seed based on week number for consistency
  seedGeneration: (weekNumber: number, playerId: string) => {
    return hash(`${weekNumber}_${playerId}`);
  },

  // Assignment rules
  rules: {
    // Each slot gets a quest from its designated category
    slotCategoryBinding: true,

    // No duplicate quests
    uniqueQuests: true,

    // Difficulty matches slot configuration
    difficultyMatching: true,

    // Consider player level for scaling
    levelScaling: {
      enabled: true,
      bands: [
        { minLevel: 1, maxLevel: 10, modifier: 0.7 },
        { minLevel: 11, maxLevel: 25, modifier: 0.85 },
        { minLevel: 26, maxLevel: 50, modifier: 1.0 },
        { minLevel: 51, maxLevel: 100, modifier: 1.15 },
        { minLevel: 101, maxLevel: Infinity, modifier: 1.3 }
      ]
    },

    // Recently assigned quests less likely to repeat
    recencyPenalty: {
      enabled: true,
      lookbackWeeks: 3,
      penaltyMultiplier: 0.3
    }
  },

  // Weekly variety tracking
  weeklyVariety: {
    rotateCategories: false, // Categories are slot-bound
    rotateQuestsWithinCategory: true
  }
};
```

## Progress Tracking System

```typescript
interface WeeklyQuestProgress {
  playerId: string;
  weekNumber: number;
  weekStartDate: Date;
  weekEndDate: Date;

  quests: {
    slotId: string;
    questId: string;
    progress: {
      current: number;
      target: number;
      percentage: number;
    };
    status: 'active' | 'completed' | 'claimed';
    completedAt?: Date;
    claimedAt?: Date;
  }[];

  weeklyStats: {
    questsCompleted: number;
    questsClaimed: number;
    totalGoldEarned: number;
    totalBPXPEarned: number;
  };
}

const PROGRESS_TRACKING = {
  // Events that update progress
  matchEndEvents: [
    'matchesPlayed',
    'matchesCompleted',
    'eliminations',
    'damageDealt',
    'knockdowns',
    'placement',
    'survivalTime',
    'chestsOpened',
    'abilitiesUsed',
    'wins'
  ],

  // Real-time updates
  updateFrequency: 'endOfMatch',

  // Progress persistence
  persistence: {
    storage: 'server',
    syncInterval: 'immediate',
    offlineQueue: true
  },

  // Cross-week handling
  weekTransition: {
    incompleteQuestsBehavior: 'expire', // No carryover
    notifyBeforeExpiry: true,
    expiryNotificationTiming: ['24h', '6h', '1h']
  }
};
```

## Reward Structure Comparison

```typescript
const WEEKLY_VS_DAILY_REWARDS = {
  dailyQuests: {
    totalSlots: 3,
    maxGoldPerDay: 450, // 100 + 150 + 200
    maxBPXPPerDay: 1550, // 300 + 500 + 750
    weeklyTotal: {
      gold: 3150, // 450 * 7
      battlePassXP: 10850 // 1550 * 7
    }
  },

  weeklyQuests: {
    totalSlots: 7,
    totalGold: 3100,
    totalDoubloons: 5,
    totalBPXP: 11500,
    bonusChest: 'Rare Chest'
  },

  combinedWeeklyMax: {
    gold: 6250,
    doubloons: 5,
    battlePassXP: 22350,
    chests: ['Rare Chest']
  },

  // Weekly quests provide ~49% of quest-based rewards
  weeklyQuestValue: 'Essential for efficient Battle Pass progression'
};
```

## Weekly Summary and Recap

```typescript
const WEEKLY_SUMMARY_CONFIG = {
  // When to show summary
  triggerConditions: [
    'onWeekReset',
    'onFirstLoginAfterReset'
  ],

  // Summary content
  summaryContent: {
    completedQuests: true,
    missedQuests: true,
    totalRewardsEarned: true,
    comparisonToPreviousWeek: true,
    battlePassProgressContribution: true,
    performanceHighlights: [
      'mostEliminationsInMatch',
      'bestPlacement',
      'longestSurvival'
    ]
  },

  // UI presentation
  presentation: {
    style: 'fullScreenModal',
    animation: 'slideIn',
    dismissible: true,
    duration: 'untilDismissed'
  },

  // Transition to new week
  newWeekPreview: {
    showNewQuests: true,
    highlightChanges: false, // Don't spoil difficulty
    motivationalMessage: true
  }
};
```

## Battle Pass XP Contribution

```typescript
const BATTLE_PASS_XP_CONTRIBUTION = {
  // Weekly quests are major BP XP source
  weeklyQuestContribution: {
    totalXPAvailable: 11500,
    percentageOfWeeklyBPProgress: '~25-30%',

    // XP per tier at mid-pass
    tierXPRequirement: 4000,
    tiersFromWeeklyQuests: '~2.8 tiers per week'
  },

  // Importance messaging
  playerCommunication: {
    tooltipText: 'Weekly quests are crucial for Battle Pass progression!',
    progressTracking: 'Show BP XP earned from weekly quests separately'
  },

  // Premium Battle Pass bonus
  premiumBonus: {
    enabled: true,
    bonusPercentage: 10, // +10% BP XP from weekly quests
    stackWithXPBoosts: true
  }
};
```

## Weekly Quest UI Layout

```typescript
const WEEKLY_QUEST_UI = {
  // Access point
  location: 'questsTab', // Same tab as daily quests

  // Section layout
  layout: {
    sectionTitle: 'Weekly Quests',
    weekIndicator: 'Week {N} of Season {S}',
    timeRemaining: 'Resets in {D}d {H}h {M}m',

    // Quest list
    questDisplay: {
      style: 'verticalList',
      sortOrder: ['active', 'completed_unclaimed', 'claimed'],
      progressBar: true,
      rewardPreview: true
    },

    // Completion tracker
    completionTracker: {
      style: 'segmentedBar', // 7 segments for 7 quests
      showBonus: true, // All complete bonus indicator
      bonusReward: null // No additional bonus currently
    }
  },

  // Quest card design
  questCard: {
    elements: [
      'questIcon',
      'questDescription',
      'progressBar',
      'progressText', // e.g., "15/25 matches"
      'rewardDisplay',
      'claimButton' // When complete
    ],

    states: {
      active: { opacity: 1.0, glowEffect: false },
      completed: { opacity: 1.0, glowEffect: true, claimButtonVisible: true },
      claimed: { opacity: 0.6, checkmark: true }
    }
  }
};
```

## Week Counter and Season Alignment

```typescript
const WEEK_TRACKING = {
  // Season alignment
  seasonAlignment: {
    weekOneStartsWithSeason: true,
    totalWeeksPerSeason: 8, // 8-week seasons

    // Week numbering
    weekNumber: {
      display: 'Week {N}',
      inSeasonContext: 'Week {N} of {total}'
    }
  },

  // Season boundary handling
  seasonBoundary: {
    lastWeekOfSeason: {
      specialQuests: false, // Same quests as normal
      bonusRewards: true, // Extra season-end rewards
      urgencyMessaging: true
    },

    // Uncompleted quests at season end
    seasonEndBehavior: 'expire' // Same as weekly reset
  },

  // New season start
  newSeasonStart: {
    weekResetAligned: true, // New season = Week 1 start
    freshQuestPool: true // May include new quest types
  }
};
```

## Edge Cases and Error Handling

```typescript
const EDGE_CASE_HANDLING = {
  // Timezone issues
  timezone: {
    allTimesInUTC: true,
    playerLocalTimeDisplay: true,
    resetTimeConversion: 'showBothUTCAndLocal'
  },

  // Missed week
  missedWeek: {
    noQuestsAssigned: false, // Always have quests ready
    catchUp: false, // No catchup for missed weeks
    returnExperience: 'normalWeeklyQuests'
  },

  // Mid-week join
  midWeekNewPlayer: {
    behavior: 'assignReducedQuests',
    questReduction: 'proportionalToTimeRemaining',
    minimumQuests: 3 // At least 3 quests even late in week
  },

  // Server issues
  serverDowntime: {
    progressPreserved: true,
    extendDeadlineIfSignificant: true,
    significantDowntimeThreshold: '4 hours'
  },

  // Progress discrepancy
  progressDispute: {
    serverAuthoritative: true,
    clientReconciliation: 'onNextSync',
    supportTicketOption: true
  }
};
```

## Analytics and Metrics

```typescript
const WEEKLY_QUEST_ANALYTICS = {
  // Completion metrics
  completionMetrics: {
    perQuestCompletionRate: true,
    perSlotCompletionRate: true,
    overallWeeklyCompletionRate: true,
    completionByDifficulty: true,
    completionByCategory: true
  },

  // Engagement patterns
  engagementPatterns: {
    questProgressDistribution: 'daily', // When during week
    earlyVsLateWeekCompletion: true,
    peakPlayDays: true
  },

  // Player segmentation
  playerSegments: {
    completionists: 'all7QuestsComplete',
    casualWeekly: '3-5QuestsComplete',
    minimal: '1-2QuestsComplete',
    nonEngaged: '0QuestsComplete'
  },

  // Quest balance metrics
  questBalance: {
    averageTimeToComplete: 'perQuest',
    difficultyPerception: 'surveyBased',
    rewardSatisfaction: 'surveyBased',
    questSkipPatterns: true // If re-roll added
  },

  // Battle Pass correlation
  battlePassCorrelation: {
    weeklyQuestCompletionVsBPProgress: true,
    tierVelocityByWeeklyEngagement: true
  }
};
```

## Integration with Daily Quests

```typescript
const DAILY_WEEKLY_INTEGRATION = {
  // UI integration
  sharedQuestTab: true,
  visualSeparation: {
    dailySection: 'top',
    weeklySection: 'bottom',
    clearDivider: true
  },

  // Progress overlap
  progressSharing: {
    enabled: true,
    examples: [
      'Daily: 3 elims → also counts toward Weekly: 25 elims',
      'Daily: Play 2 matches → also counts toward Weekly: Play 15 matches'
    ]
  },

  // Reward stacking
  rewardStacking: {
    earnBothRewards: true, // Complete both = get both rewards
    noConflict: true
  },

  // Milestone quest synergy
  milestoneSynergy: {
    dailyCompletionCountsForMilestone: true,
    example: 'Weekly Milestone: Complete all dailies for 5 days'
  }
};
```

## Future Enhancements

```typescript
const FUTURE_ENHANCEMENTS = {
  // Quest re-roll for weekly
  weeklyReroll: {
    planned: true,
    cost: '50 Doubloons per reroll',
    limit: '1 per week',
    restrictions: 'Cannot reroll Milestone quest'
  },

  // Weekly streak bonus
  streakBonus: {
    planned: true,
    mechanic: 'Complete all 7 weekly quests for X weeks in a row',
    rewards: 'Escalating bonus rewards'
  },

  // Weekly leaderboard
  weeklyLeaderboard: {
    planned: false,
    concept: 'Fastest to complete all weekly quests'
  },

  // Community weekly goals
  communityWeekly: {
    planned: true,
    concept: 'Server-wide weekly contribution goals',
    integration: 'Separate from individual weekly quests'
  }
};
```
