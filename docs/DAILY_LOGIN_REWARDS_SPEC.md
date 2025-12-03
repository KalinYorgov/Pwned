# Daily Login Rewards Specification

## Overview

The Daily Login Rewards system encourages players to log in every day by providing escalating rewards through a calendar-based claiming system. This system builds player habits and increases day-over-day retention through reward anticipation.

## Reward Calendar Structure

```typescript
interface DailyLoginCalendar {
  calendarId: string;
  cycleType: 'monthly' | 'rolling';
  cycleDuration: number; // Days

  days: DailyReward[];

  streakBonuses: StreakBonus[];

  resetBehavior: ResetBehavior;
}

interface DailyReward {
  day: number;
  reward: Reward;
  isMilestone: boolean;
  specialAnimation?: string;
}

interface Reward {
  gold?: number;
  doubloons?: number;
  battlePassXP?: number;
  chest?: ChestType;
  items?: ItemReward[];
}

const MONTHLY_CALENDAR: DailyLoginCalendar = {
  calendarId: 'monthly_login_v1',
  cycleType: 'monthly',
  cycleDuration: 28, // 4-week cycle

  days: [
    // Week 1 - Entry rewards
    { day: 1, reward: { gold: 50 }, isMilestone: false },
    { day: 2, reward: { gold: 75 }, isMilestone: false },
    { day: 3, reward: { gold: 100 }, isMilestone: false },
    { day: 4, reward: { gold: 100 }, isMilestone: false },
    { day: 5, reward: { gold: 125 }, isMilestone: false },
    { day: 6, reward: { gold: 150 }, isMilestone: false },
    { day: 7, reward: { gold: 200, doubloons: 5 }, isMilestone: true, specialAnimation: 'week1Complete' },

    // Week 2 - Growing rewards
    { day: 8, reward: { gold: 100 }, isMilestone: false },
    { day: 9, reward: { gold: 100 }, isMilestone: false },
    { day: 10, reward: { gold: 125 }, isMilestone: false },
    { day: 11, reward: { gold: 150 }, isMilestone: false },
    { day: 12, reward: { gold: 150 }, isMilestone: false },
    { day: 13, reward: { gold: 175 }, isMilestone: false },
    { day: 14, reward: { gold: 250, chest: 'common' }, isMilestone: true, specialAnimation: 'week2Complete' },

    // Week 3 - Premium touches
    { day: 15, reward: { gold: 125 }, isMilestone: false },
    { day: 16, reward: { gold: 150 }, isMilestone: false },
    { day: 17, reward: { doubloons: 5 }, isMilestone: false },
    { day: 18, reward: { gold: 175 }, isMilestone: false },
    { day: 19, reward: { gold: 175 }, isMilestone: false },
    { day: 20, reward: { gold: 200 }, isMilestone: false },
    { day: 21, reward: { gold: 300, doubloons: 10 }, isMilestone: true, specialAnimation: 'week3Complete' },

    // Week 4 - Grand finale
    { day: 22, reward: { gold: 150 }, isMilestone: false },
    { day: 23, reward: { gold: 175 }, isMilestone: false },
    { day: 24, reward: { gold: 200 }, isMilestone: false },
    { day: 25, reward: { gold: 200 }, isMilestone: false },
    { day: 26, reward: { gold: 225 }, isMilestone: false },
    { day: 27, reward: { doubloons: 10 }, isMilestone: false },
    { day: 28, reward: { gold: 500, doubloons: 20, chest: 'rare' }, isMilestone: true, specialAnimation: 'monthComplete' }
  ],

  totalRewards: {
    gold: 4575,
    doubloons: 50,
    chests: ['1x Common Chest', '1x Rare Chest']
  }
};
```

## Login Tracking System

```typescript
interface PlayerLoginData {
  playerId: string;

  currentCycle: {
    cycleStartDate: Date;
    cycleEndDate: Date;
    currentDay: number; // 1-28
    claimedDays: number[]; // [1, 2, 3, 5, 6, ...]
  };

  streak: {
    currentStreak: number;
    longestStreak: number;
    streakStartDate: Date;
    lastLoginDate: Date;
    streakBroken: boolean;
  };

  history: {
    totalLogins: number;
    totalRewardsClaimed: number;
    cyclesCompleted: number;
  };
}

const LOGIN_TRACKING = {
  // What counts as a login
  loginDefinition: {
    trigger: 'clientConnect',
    minimumSessionTime: '0 seconds', // Just connecting counts
    multipleLoginsPerDay: 'firstOnly',
    resetTime: '00:00 UTC'
  },

  // Timezone handling
  timezone: {
    serverTime: 'UTC',
    displayToPlayer: 'local',
    resetNotification: 'showBothTimes'
  },

  // Data persistence
  persistence: {
    storage: 'database',
    cacheLayer: 'redis',
    syncOnLogin: true,
    backupFrequency: 'hourly'
  }
};
```

## Calendar UI Design

```typescript
const CALENDAR_UI = {
  layout: {
    style: 'grid',
    columns: 7, // Days of week
    rows: 4, // 4 weeks
    showDayNumbers: true,
    showRewardPreviews: true
  },

  dayStates: {
    claimed: {
      visual: 'checkmark',
      opacity: 0.7,
      glowEffect: false,
      interactive: false
    },
    available: {
      visual: 'highlightGlow',
      opacity: 1.0,
      glowEffect: true,
      interactive: true,
      pulsingAnimation: true
    },
    locked: {
      visual: 'lockIcon',
      opacity: 0.5,
      glowEffect: false,
      interactive: false,
      previewReward: true
    },
    missed: {
      visual: 'xMark',
      opacity: 0.4,
      glowEffect: false,
      interactive: false
    }
  },

  milestoneHighlights: {
    day7: { border: 'gold', size: 'normal', badge: 'Week 1' },
    day14: { border: 'gold', size: 'normal', badge: 'Week 2' },
    day21: { border: 'gold', size: 'normal', badge: 'Week 3' },
    day28: { border: 'epic', size: 'large', badge: 'Month Complete!' }
  },

  rewardPreview: {
    showOnHover: true,
    showAmount: true,
    showIcon: true,
    animation: 'slideIn'
  },

  claimButton: {
    position: 'onTodayCell',
    style: 'prominentGlow',
    text: 'Claim!',
    size: 'large'
  },

  progressIndicator: {
    style: 'progressBar',
    position: 'top',
    showDaysRemaining: true,
    showNextMilestone: true
  }
};
```

## Claim Flow

```typescript
const CLAIM_FLOW = {
  // Access points
  accessPoints: {
    mainMenuPopup: {
      showOnLogin: true,
      autoDismiss: false,
      dismissable: true,
      skipOption: false
    },
    menuButton: {
      location: 'mainMenu',
      notificationBadge: true,
      badgeStyle: 'exclamation'
    },
    profileSection: {
      showStreak: true,
      showNextReward: true
    }
  },

  // Claim action
  claimAction: {
    button: 'single tap',
    confirmation: false, // Instant claim
    animation: 'rewardBurst',
    sound: 'coinCollect',
    hapticFeedback: 'medium'
  },

  // Post-claim
  postClaim: {
    rewardDisplay: {
      duration: '2 seconds',
      animation: 'flyToInventory',
      showTotal: true
    },
    calendarUpdate: {
      markDayClaimed: true,
      highlightNextDay: true,
      animation: 'checkmarkStamp'
    },
    streakUpdate: {
      showNewStreak: true,
      animation: 'streakCounter',
      milestonesCelebration: true
    }
  },

  // Edge cases
  edgeCases: {
    alreadyClaimed: {
      buttonState: 'disabled',
      message: 'Already claimed today!',
      showNextReset: true
    },
    offlineLogin: {
      queueClaim: true,
      syncOnReconnect: true
    },
    serverError: {
      retryAutomatically: true,
      maxRetries: 3,
      fallbackMessage: 'Reward pending...'
    }
  }
};
```

## Streak System

```typescript
const STREAK_SYSTEM = {
  streakDefinition: {
    consecutiveDays: true,
    resetTime: '00:00 UTC',
    gracePeriod: {
      enabled: true,
      duration: '48 hours', // Can miss one day
      usesPerCycle: 1 // Only 1 grace use per month
    }
  },

  streakDisplay: {
    location: 'calendarHeader',
    style: 'flameIcon + number',
    animation: 'flickerOnUpdate',
    milestones: [3, 7, 14, 21, 28]
  },

  streakBonuses: [
    {
      streakDays: 3,
      bonus: { type: 'multiplier', value: 1.1 }, // 10% bonus on day 3 reward
      message: '3-Day Streak! +10% Bonus!'
    },
    {
      streakDays: 7,
      bonus: { type: 'multiplier', value: 1.25 }, // 25% bonus on day 7 reward
      message: '7-Day Streak! +25% Bonus!'
    },
    {
      streakDays: 14,
      bonus: { type: 'multiplier', value: 1.5 }, // 50% bonus on day 14 reward
      message: '2-Week Streak! +50% Bonus!'
    },
    {
      streakDays: 21,
      bonus: { type: 'multiplier', value: 1.75 }, // 75% bonus on day 21 reward
      message: '3-Week Streak! +75% Bonus!'
    },
    {
      streakDays: 28,
      bonus: { type: 'multiplier', value: 2.0 }, // Double day 28 reward
      message: 'Perfect Month! 2x Bonus!'
    }
  ],

  streakBreak: {
    detection: 'noLoginFor48Hours',
    consequence: 'resetToZero',
    notification: {
      send: true,
      message: 'Your streak was reset. Start a new streak today!',
      timing: 'onNextLogin'
    },
    progressPreserved: true // Still on day X of calendar, just streak reset
  }
};
```

## Streak Break and Reset Behavior

```typescript
const RESET_BEHAVIOR = {
  streakReset: {
    trigger: 'missedDay',
    gracePeriodFirst: true,
    resetStreakCounter: true,
    preserveCalendarProgress: true // Continue from day X, not day 1
  },

  calendarReset: {
    trigger: 'cycleComplete', // After day 28 claimed
    timing: 'immediate',
    newCycleStartDay: 1,
    preserveHistory: true
  },

  partialReset: {
    enabled: false, // Full reset on streak break
    alternative: 'gracePeriodOnly'
  },

  monthlyReset: {
    alignWithCalendarMonth: false, // 28-day rolling cycle
    playerSpecificCycle: true, // Starts when player first logs in
    resetNotification: true
  }
};
```

## Returning Player Catch-Up

```typescript
const RETURNING_PLAYER = {
  detection: {
    absenceDays: 7, // Gone for 7+ days
    triggerOn: 'firstLoginAfterAbsence'
  },

  catchUpRewards: {
    enabled: true,
    type: 'welcomeBackPackage',
    rewards: {
      gold: 500,
      battlePassXP: 500,
      chest: 'common'
    },
    separate: true // Doesn't affect calendar progress
  },

  calendarHandling: {
    skipMissedDays: true,
    startFromToday: true, // Resume from current day
    lostRewardsRecovery: false, // Missed days are lost
    graceOnReturn: true // Extra grace period on return
  },

  messaging: {
    welcomeBackPopup: true,
    showMissedRewards: false, // Don't guilt trip
    emphasizeNewRewards: true,
    motivationalMessage: 'Welcome back, Captain! Your daily rewards await!'
  },

  specialIncentive: {
    enabled: true,
    type: 'boostFirstWeek',
    bonus: 'nextSevenDaysDoubleCurrency',
    message: 'Returning Captain Bonus: Double Gold for 7 days!'
  }
};
```

## Monthly Cycle and Escalation

```typescript
const MONTHLY_ESCALATION = {
  cycleProgression: {
    cycle1: {
      name: 'Recruit',
      multiplier: 1.0,
      specialReward: null
    },
    cycle2: {
      name: 'Deckhand',
      multiplier: 1.1, // 10% more rewards
      specialReward: 'exclusive_nameplate'
    },
    cycle3: {
      name: 'Sailor',
      multiplier: 1.2, // 20% more rewards
      specialReward: 'rare_chest_upgrade'
    },
    cycle4: {
      name: 'First Mate',
      multiplier: 1.3, // 30% more rewards
      specialReward: 'exclusive_emote'
    },
    cycle5Plus: {
      name: 'Captain',
      multiplier: 1.5, // 50% more rewards, caps here
      specialReward: 'rotatingExclusive'
    }
  },

  escalationTracking: {
    persistAcrossStreakBreaks: true, // Cycle level doesn't reset
    displayToPlayer: true,
    showNextTier: true
  },

  tierDisplay: {
    badge: true,
    calendarDecoration: true,
    tierNameVisible: true
  }
};
```

## Notification System

```typescript
const LOGIN_NOTIFICATIONS = {
  pushNotifications: {
    dailyReminder: {
      enabled: true,
      timing: 'playerPeakTime', // Based on historical login time
      fallbackTime: '18:00 local',
      message: 'Your daily reward is waiting! Don\'t break your {streak}-day streak!',
      frequencyCap: '1 per day'
    },

    streakAtRisk: {
      enabled: true,
      timing: '2 hours before reset',
      condition: 'notLoggedInToday && streak > 3',
      message: 'Your {streak}-day streak expires soon! Log in to claim your reward!',
      urgency: 'high'
    },

    streakBroken: {
      enabled: true,
      timing: 'immediately after reset',
      condition: 'streakBroken && streak > 7',
      message: 'Your streak ended. Start a new one today and earn bonus rewards!',
      urgency: 'normal'
    },

    milestoneApproaching: {
      enabled: true,
      timing: 'onPreviousDayClaim',
      condition: 'nextDayIsMilestone',
      message: 'Tomorrow is Day {day}! Log in for a special milestone reward!',
      urgency: 'normal'
    }
  },

  inGameReminders: {
    mainMenuBadge: {
      showWhenUnclamed: true,
      pulsingAnimation: true
    },

    loginPopup: {
      showOnFirstLogin: true,
      priority: 'high',
      dismissAfterClaim: true
    }
  },

  playerPreferences: {
    optOutOption: true,
    frequencyControl: true,
    quietHours: true
  }
};
```

## Edge Cases and Error Handling

```typescript
const EDGE_CASES = {
  timezoneIssues: {
    serverAuthority: 'UTC',
    displayConversion: 'clientLocal',
    discrepancyHandling: 'serverWins',
    vpnCheating: 'serverTimeOnly'
  },

  dateLineIssues: {
    resetBasedOnUTC: true,
    playerLocalMidnight: 'displayOnly',
    travelBetweenTimezones: 'noImpact'
  },

  deviceTimeManipulation: {
    detection: 'serverValidation',
    response: 'useServerTime',
    flagForReview: true
  },

  multipleDevices: {
    syncOnLogin: true,
    lastClaimWins: false, // Only one claim per day
    conflictResolution: 'firstClaimOnly'
  },

  serverDowntime: {
    gracePeriodExtension: true,
    compensationIfMissed: true,
    autoRecovery: true
  },

  accountRecovery: {
    preserveProgress: true,
    preserveStreak: 'ifWithinGracePeriod',
    mergeDuplicates: 'keepHigherProgress'
  }
};
```

## Analytics and Metrics

```typescript
const LOGIN_ANALYTICS = {
  keyMetrics: {
    dailyLoginRate: {
      definition: 'uniqueLoginsToday / MAU',
      target: '>40%',
      alert: '<25%'
    },

    streakDistribution: {
      definition: 'histogramOfCurrentStreaks',
      buckets: [1, 3, 7, 14, 21, 28],
      goal: 'shiftRightOverTime'
    },

    claimRate: {
      definition: 'claimsToday / loginsToday',
      target: '>95%',
      alert: '<80%'
    },

    cycleCompletionRate: {
      definition: 'day28Claims / day1Claims',
      target: '>30%',
      alert: '<15%'
    },

    streakRetention: {
      definition: 'playersWithStreak7+ / totalPlayers',
      target: '>20%',
      alert: '<10%'
    }
  },

  cohortAnalysis: {
    byJoinDate: true,
    byFirstLoginRewardClaim: true,
    retentionCorrelation: true
  },

  funnelTracking: [
    'login',
    'calendarViewed',
    'claimButtonClicked',
    'rewardReceived'
  ],

  abTestMetrics: {
    rewardAmounts: true,
    calendarDesign: true,
    streakBonuses: true,
    notificationTiming: true
  }
};
```

## Admin Configuration

```typescript
const ADMIN_CONFIGURATION = {
  rewardEditor: {
    modifyDailyRewards: true,
    modifyMilestoneRewards: true,
    modifyStreakBonuses: true,
    previewBeforePublish: true,
    versionControl: true
  },

  calendarScheduling: {
    createNewCalendars: true,
    scheduleCalendarSwap: true,
    abTestCalendars: true
  },

  playerSupport: {
    viewPlayerProgress: true,
    grantMissedReward: true,
    resetPlayerCalendar: true,
    extendGracePeriod: true,
    restoreStreak: true
  },

  emergencyControls: {
    pauseSystem: true,
    grantGlobalCompensation: true,
    extendGracePeriodGlobally: true
  },

  reporting: {
    dailyReport: true,
    weeklyTrends: true,
    monthlyAnalysis: true,
    exportData: true
  }
};
```

## Integration Points

```typescript
const INTEGRATIONS = {
  battlePass: {
    loginRewardsContributeBPXP: true,
    separateFromDailyQuests: true
  },

  questSystem: {
    notPartOfDailyQuests: true,
    separateFromQuestTab: true,
    noQuestForLogin: true // Login is reward itself
  },

  currencySystem: {
    rewardsGrantedDirectly: true,
    instantToWallet: true
  },

  notificationSystem: {
    usesUnifiedSystem: true,
    respectsPlayerPreferences: true
  },

  analytics: {
    eventsTracked: [
      'login_reward_viewed',
      'login_reward_claimed',
      'streak_milestone_reached',
      'streak_broken',
      'cycle_completed'
    ]
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  dataModel: {
    playerLoginData: 'mongodb',
    calendarDefinition: 'mongodb + cache',
    realtimeStreak: 'redis'
  },

  apiEndpoints: {
    getCalendarStatus: 'GET /api/login-rewards/status',
    claimReward: 'POST /api/login-rewards/claim',
    getHistory: 'GET /api/login-rewards/history'
  },

  caching: {
    calendarDefinition: '1 hour',
    playerStatus: '5 minutes',
    invalidateOnClaim: true
  },

  resetJob: {
    timing: '00:00 UTC',
    process: 'updateAllPlayerStreaks',
    batchSize: 10000,
    timeout: '30 minutes'
  },

  performance: {
    claimLatency: '<100ms',
    statusCheckLatency: '<50ms',
    concurrentClaims: '10000/s'
  }
};
```
