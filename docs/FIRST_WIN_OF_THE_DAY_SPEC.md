# First Win of the Day Specification

## Overview

The First Win of the Day (FWOTD) system provides a significant bonus reward for a player's first victory each day, encouraging daily competitive play and creating a compelling reason to return and play until achieving a win.

## Core Mechanics

```typescript
interface FirstWinBonus {
  bonusId: string;
  status: 'available' | 'earned' | 'claimed';

  rewards: {
    gold: number;
    battlePassXP: number;
    bonusMatchXP: number;
  };

  tracking: {
    resetTime: string; // UTC time
    lastWinDate: Date | null;
    winsToday: number;
    firstWinAchieved: boolean;
  };
}

const FIRST_WIN_CONFIG = {
  rewards: {
    gold: 200,
    battlePassXP: 500,
    bonusMatchXP: 150 // Added on top of normal match XP
  },

  // Win definition
  winCondition: {
    placement: 1, // Must be #1
    soloMode: true,
    duoMode: true, // Team must win
    squadMode: true // Team must win
  },

  // Reset timing
  reset: {
    time: '00:00 UTC',
    gracePeriod: '5 minutes', // Buffer for in-progress matches
    resetNotification: false // Silent reset
  },

  // Availability
  availability: {
    alwaysAvailable: true,
    requiresAccountLevel: 1, // Available immediately
    enabledModes: ['solo', 'duo', 'squad', 'ranked']
  }
};
```

## Win Tracking System

```typescript
interface DailyWinTracking {
  playerId: string;
  date: string; // YYYY-MM-DD in UTC

  winData: {
    firstWinTime: Date | null;
    firstWinMatchId: string | null;
    firstWinMode: string | null;
    totalWinsToday: number;
  };

  bonusData: {
    earned: boolean;
    earnedAt: Date | null;
    claimed: boolean;
    claimedAt: Date | null;
    rewardsGranted: Reward | null;
  };
}

const WIN_TRACKING = {
  // What counts as a win
  winDetection: {
    trigger: 'matchEndWithPlacement1',
    teamModes: 'anyTeamMemberWinCounts',
    disconnectHandling: 'mustBeConnectedAtVictory',
    afkHandling: 'afkPlayersDoNotCount'
  },

  // Tracking persistence
  persistence: {
    storage: 'database',
    cache: 'redis',
    syncOnMatchEnd: true
  },

  // Daily reset
  resetProcess: {
    timing: '00:00 UTC',
    batchProcess: true,
    preserveHistory: true, // For analytics
    historyRetention: '90 days'
  },

  // Edge cases
  edgeCases: {
    winDuringReset: 'countForNewDay',
    multipleWinsSimultaneous: 'firstProcessedWins',
    serverCrashDuringMatch: 'matchInvalidated'
  }
};
```

## Visual Indicator System

```typescript
const VISUAL_INDICATORS = {
  // Pre-match indicators
  preMatch: {
    mainMenuBadge: {
      location: 'playButton',
      style: 'goldenGlow',
      icon: 'crownWithPlus',
      tooltip: 'First Win Bonus Available!',
      animation: 'subtlePulse'
    },

    modeSelectIndicator: {
      showOnAllModes: true,
      badge: '+200 Gold +500 BP XP',
      position: 'topRight',
      style: 'golden'
    },

    loadingScreenReminder: {
      enabled: true,
      message: 'First Win Bonus: +200 Gold, +500 BP XP!',
      position: 'bottomThird'
    }
  },

  // In-match indicators
  inMatch: {
    hudIndicator: {
      enabled: true,
      location: 'topRight',
      style: 'smallIcon',
      showOnlyIfAvailable: true,
      tooltip: 'Win for bonus!'
    }
  },

  // Post-win indicators
  postWin: {
    earned: {
      celebrationOverlay: true,
      specialAnimation: 'goldenBurst',
      soundEffect: 'victoryFanfare',
      hapticFeedback: 'heavy'
    }
  },

  // Already earned today
  alreadyEarned: {
    mainMenuState: {
      badge: 'checkmark',
      style: 'dimmed',
      tooltip: 'Earned today! Resets at {resetTime}'
    },

    modeSelectState: {
      badge: 'completed',
      showNextReset: true
    }
  }
};
```

## Victory Celebration Flow

```typescript
const CELEBRATION_FLOW = {
  // Normal victory screen
  victoryScreen: {
    baseElements: [
      'victoryBanner',
      'placementDisplay',
      'matchStats',
      'xpEarned',
      'currencyEarned'
    ]
  },

  // First Win bonus overlay
  firstWinOverlay: {
    trigger: 'firstWinOfDay',
    timing: 'afterBaseVictoryScreen',
    duration: '4 seconds',

    elements: [
      {
        type: 'banner',
        text: 'FIRST WIN OF THE DAY!',
        style: 'golden',
        animation: 'slideInWithGlow'
      },
      {
        type: 'rewardDisplay',
        items: [
          { icon: 'gold', amount: '+200', animation: 'countUp' },
          { icon: 'bpXP', amount: '+500', animation: 'countUp' },
          { icon: 'matchXP', amount: '+150', animation: 'countUp' }
        ]
      },
      {
        type: 'message',
        text: 'Come back tomorrow for another bonus!',
        delay: '2 seconds'
      }
    ],

    effects: {
      screenFlash: 'golden',
      particles: 'goldCoins',
      sound: 'bonusFanfare',
      haptic: 'celebration'
    }
  },

  // Results screen integration
  resultsScreen: {
    firstWinSection: {
      visible: true,
      position: 'aboveNormalRewards',
      highlight: true,
      showSeparately: true
    },

    breakdown: {
      showBaseRewards: true,
      showFirstWinBonus: true,
      showTotal: true,
      animatedTally: true
    }
  }
};
```

## Reward Distribution

```typescript
const REWARD_DISTRIBUTION = {
  timing: {
    trigger: 'matchEndConfirmed',
    instantGrant: true, // No claim button needed
    confirmation: 'resultsScreen'
  },

  rewards: {
    gold: {
      amount: 200,
      addedTo: 'wallet',
      animation: 'flyToBalance',
      receipt: 'transactionLog'
    },

    battlePassXP: {
      amount: 500,
      addedTo: 'battlePassProgress',
      tierProgressUpdate: true,
      mayTriggerTierUp: true
    },

    bonusMatchXP: {
      amount: 150,
      addedTo: 'playerLevel',
      stacksWithMatchXP: true,
      shownSeparately: true
    }
  },

  // Stacking with other bonuses
  stacking: {
    withMatchRewards: true,
    withQuestRewards: true,
    withEventBonuses: true,
    withPremiumBonus: true, // If Battle Pass premium

    xpMultipliers: {
      appliesTo: 'bonusMatchXP',
      premiumMultiplier: 1.1, // If applicable
      eventMultiplier: 'varies'
    }
  },

  // Error handling
  errorHandling: {
    grantFailed: 'queueForRetry',
    maxRetries: 5,
    fallback: 'grantOnNextLogin',
    playerNotification: 'yourRewardIsPending'
  }
};
```

## Daily Reset Mechanism

```typescript
const DAILY_RESET = {
  timing: {
    resetTime: '00:00 UTC',
    displayTime: 'playerLocal',
    countdown: true
  },

  resetProcess: {
    steps: [
      'markPreviousDayComplete',
      'archiveStatistics',
      'resetBonusStatus',
      'updateUIIndicators'
    ],

    playerImpact: {
      inMatch: 'noImpact', // Current match unaffected
      postMatchWin: 'newDayBonus', // Win after reset = new bonus
      inMenu: 'instantUpdate'
    }
  },

  countdownDisplay: {
    showInMenu: true,
    location: 'nearPlayButton',
    format: 'HH:MM:SS until reset',
    showWhenEarned: true,
    showWhenAvailable: false // No countdown when can still earn
  },

  notifications: {
    resetComplete: false, // Silent reset
    bonusAvailable: false, // Visual indicator sufficient
    earned: true // Celebrate at victory
  }
};
```

## Mode-Specific Behavior

```typescript
const MODE_BEHAVIOR = {
  soloMode: {
    winCondition: 'placement1',
    bonusEligible: true,
    fullReward: true
  },

  duoMode: {
    winCondition: 'teamPlacement1',
    bonusEligible: true,
    fullReward: true,
    bothTeammatesGetBonus: true // If both haven't earned today
  },

  squadMode: {
    winCondition: 'teamPlacement1',
    bonusEligible: true,
    fullReward: true,
    allTeammatesGetBonus: true // If haven't earned today
  },

  rankedMode: {
    winCondition: 'placement1',
    bonusEligible: true,
    fullReward: true,
    additionalRankXP: false // Separate from FWOTD
  },

  eventModes: {
    winCondition: 'modeSpecific',
    bonusEligible: true,
    countsForFWOTD: true
  },

  ltmModes: {
    winCondition: 'modeSpecific',
    bonusEligible: 'configurable', // Some LTMs may not count
    defaultEligible: true
  }
};
```

## Player Profile Integration

```typescript
const PROFILE_INTEGRATION = {
  profileDisplay: {
    showTodayStatus: true,
    showStreak: true, // Consecutive days with FWOTD
    showTotalFWOTD: true
  },

  statistics: {
    tracked: [
      'totalFirstWins',
      'currentStreak',
      'longestStreak',
      'averageTimeToFirstWin',
      'preferredModeForFirstWin'
    ],

    displayLocation: 'statsTab',
    updateFrequency: 'realtime'
  },

  achievements: {
    firstWinRelated: [
      { name: 'First Victory', requirement: 'earnFirstFWOTD', reward: 'title' },
      { name: 'Weekly Winner', requirement: '7dayStreak', reward: 'badge' },
      { name: 'Monthly Champion', requirement: '30dayStreak', reward: 'cosmetic' },
      { name: 'Century Club', requirement: '100totalFWOTD', reward: 'exclusiveTitle' }
    ]
  }
};
```

## Edge Cases and Error Handling

```typescript
const EDGE_CASES = {
  // Match timing
  matchDuringReset: {
    scenario: 'match started before reset, won after reset',
    behavior: 'countsForNewDay',
    rationale: 'positive player experience'
  },

  winJustBeforeReset: {
    scenario: 'won 1 minute before reset',
    behavior: 'normalFWOTD',
    noCarryover: true
  },

  // Connection issues
  disconnectBeforeVictory: {
    scenario: 'player disconnects, team wins',
    behavior: 'noFWOTD',
    requirement: 'mustBeConnectedAtVictory'
  },

  serverCrash: {
    scenario: 'server crashes mid-match',
    behavior: 'matchInvalidated',
    compensation: 'none', // Match didn't complete
    retryAvailable: true
  },

  // Multiple devices
  multiDeviceLogin: {
    scenario: 'logged in on multiple devices',
    behavior: 'firstWinOnAnyDevice',
    sync: 'immediate'
  },

  // Account issues
  accountRecovery: {
    scenario: 'account recovered mid-day',
    behavior: 'preserveFWOTDStatus',
    sync: 'fromServer'
  },

  // Exploits
  timeManipulation: {
    detection: 'serverTimeOnly',
    response: 'useServerTime',
    flagging: true
  },

  intentionalLossExploit: {
    detection: 'notApplicable', // Must win to get bonus
    response: 'n/a'
  }
};
```

## Analytics and Metrics

```typescript
const FWOTD_ANALYTICS = {
  keyMetrics: {
    dailyFWOTDRate: {
      definition: 'playersEarnedFWOTD / activePlayersToday',
      target: '>25%',
      alert: '<15%'
    },

    averageTimeToFWOTD: {
      definition: 'avgMinutesFromFirstMatchToWin',
      target: '<60 minutes',
      insight: 'playerSkillIndicator'
    },

    fwotdRetentionImpact: {
      definition: 'D1retention for FWOTD earners vs non-earners',
      expected: 'significant positive correlation'
    },

    sessionsPerFWOTD: {
      definition: 'avgSessionsBeforeFWOTD',
      insight: 'engagementDepth'
    }
  },

  behaviorAnalysis: {
    playUntilWin: {
      definition: 'players who play until FWOTD',
      segmentation: true
    },

    modePReference: {
      definition: 'which mode most often yields FWOTD',
      actionable: true
    },

    timeOfDay: {
      definition: 'when players typically earn FWOTD',
      insight: 'peakHoursAnalysis'
    }
  },

  cohortAnalysis: {
    bySkillLevel: true,
    byAccountAge: true,
    bySpendLevel: true
  },

  reporting: {
    daily: true,
    weekly: true,
    alertOnAnomaly: true
  }
};
```

## Notification and Reminder System

```typescript
const FWOTD_NOTIFICATIONS = {
  inGame: {
    availableReminder: {
      trigger: 'onLogin',
      display: 'mainMenuBadge',
      message: null // Visual indicator only
    },

    lossReminder: {
      trigger: 'afterNonWinMatch',
      display: 'resultsScreenSubtle',
      message: 'Keep playing for your First Win Bonus!',
      frequency: 'everyMatch'
    }
  },

  push: {
    dailyReminder: {
      enabled: true,
      timing: 'playerPeakPlaytime',
      condition: 'hasNotEarnedToday && hasNotPlayedToday',
      message: 'Your First Win Bonus is waiting! Win a match for +200 Gold!',
      frequencyCap: '1 per day'
    },

    streakAtRisk: {
      enabled: true,
      timing: '3 hours before reset',
      condition: 'hasStreak >= 3 && hasNotEarnedToday',
      message: 'Your {streak}-day First Win streak is at risk! Play now!',
      urgency: 'high'
    }
  },

  playerPreferences: {
    optOut: true,
    customTiming: false,
    quietHours: true
  }
};
```

## Admin Configuration

```typescript
const ADMIN_CONFIG = {
  rewardAdjustment: {
    modifyGold: true,
    modifyBPXP: true,
    modifyBonusXP: true,
    requiresApproval: true,
    liveUpdate: true
  },

  modeConfiguration: {
    enableDisablePerMode: true,
    ltmEligibility: true,
    eventOverrides: true
  },

  playerSupport: {
    viewPlayerFWOTDHistory: true,
    grantMissedFWOTD: true,
    resetPlayerStatus: true,
    requiresTicket: true
  },

  analytics: {
    realTimeDashboard: true,
    customReports: true,
    abTesting: true
  },

  emergencyControls: {
    disableGlobally: true,
    adjustRewardsLive: true,
    compensationGrant: true
  }
};
```

## Integration Points

```typescript
const INTEGRATIONS = {
  matchSystem: {
    winDetection: 'matchEndEvent',
    placementData: 'matchResults',
    modeInfo: 'matchConfig'
  },

  currencySystem: {
    goldGrant: 'directDeposit',
    transactionLog: true
  },

  battlePass: {
    xpContribution: true,
    triggerTierUp: true,
    premiumBonus: 'ifApplicable'
  },

  questSystem: {
    questProgress: false, // FWOTD is separate
    dailyQuestInteraction: 'none'
  },

  loginRewards: {
    separateSystem: true,
    noInteraction: true
  },

  analytics: {
    eventTracking: [
      'fwotd_available',
      'fwotd_earned',
      'fwotd_rewards_granted',
      'fwotd_streak_updated'
    ]
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  dataModel: {
    playerFWOTDData: 'mongodb',
    dailyStatus: 'redis (24h TTL)',
    history: 'mongodb'
  },

  apiEndpoints: {
    getStatus: 'GET /api/fwotd/status',
    recordWin: 'POST /api/fwotd/record (internal)',
    getHistory: 'GET /api/fwotd/history'
  },

  eventFlow: {
    matchEnd: 'matchService -> fwotdService',
    winDetected: 'check eligibility -> grant rewards -> update status',
    notifyClient: 'websocket push'
  },

  caching: {
    playerStatus: 'redis, 5 min TTL',
    invalidateOnWin: true
  },

  performance: {
    winProcessingLatency: '<50ms',
    statusCheckLatency: '<20ms',
    concurrentWins: '5000/s'
  },

  reliability: {
    retryOnFailure: true,
    idempotentGrant: true,
    auditLog: true
  }
};
```
