# Limited-Time Event Framework Specification

## Overview

The Limited-Time Event (LTE) Framework enables the creation and management of time-bounded special events that provide unique content, quests, rewards, and gameplay experiences. This framework supports concurrent events, scheduling, and seamless integration with existing game systems.

## Event Architecture

```typescript
interface LimitedTimeEvent {
  eventId: string;
  version: number;
  status: EventStatus;

  // Core identity
  identity: {
    name: string;
    displayName: string;
    description: string;
    theme: EventTheme;
    iconUrl: string;
    bannerUrl: string;
  };

  // Scheduling
  schedule: {
    startDate: Date;
    endDate: Date;
    timezone: 'UTC';
    gracePeriod: {
      beforeStart: number; // Hours for pre-event teasers
      afterEnd: number; // Hours for final claims
    };
  };

  // Content
  content: {
    quests: EventQuest[];
    rewardsTrack: EventRewardsTrack;
    shopItems: EventShopItem[];
    gameMode?: EventGameMode;
    uiTheme: EventUITheme;
  };

  // Configuration
  configuration: {
    concurrent: boolean;
    priority: number;
    targetAudience: AudienceFilter;
    notifications: NotificationConfig;
  };

  // Metadata
  metadata: {
    createdBy: string;
    createdAt: Date;
    approvedBy: string;
    approvedAt: Date;
    tags: string[];
  };
}

type EventStatus =
  | 'draft'
  | 'scheduled'
  | 'teaser'      // Pre-event period
  | 'active'
  | 'grace'       // Post-event claim period
  | 'ended'
  | 'archived';
```

## Event Scheduling System

```typescript
const EVENT_SCHEDULING = {
  // Advance scheduling
  advanceScheduling: {
    minAdvanceTime: '24 hours',
    maxAdvanceTime: '90 days',
    requiredForApproval: '48 hours before start',

    scheduling: {
      dateTimeInput: 'calendar',
      timezone: 'UTC',
      displayLocalTime: true,
      conflictDetection: true
    }
  },

  // Event duration guidelines
  durationGuidelines: {
    miniEvent: { min: '1 day', max: '3 days' },
    standardEvent: { min: '5 days', max: '14 days' },
    majorEvent: { min: '14 days', max: '28 days' }
  },

  // Lifecycle phases
  phases: {
    teaser: {
      duration: '24-48 hours before start',
      visibility: 'partialReveal',
      actions: ['showBanner', 'sendNotification', 'hideRewards']
    },
    active: {
      duration: 'startDate to endDate',
      visibility: 'full',
      actions: ['enableQuests', 'enableShop', 'enableGameMode', 'trackProgress']
    },
    grace: {
      duration: '24-48 hours after end',
      visibility: 'limited',
      actions: ['disableNewProgress', 'allowClaims', 'warnExpiry']
    },
    ended: {
      visibility: 'none',
      actions: ['removeContent', 'archiveData', 'processUnclaimed']
    }
  },

  // Automatic transitions
  transitions: {
    toTeaser: 'automatic at teaserStart',
    toActive: 'automatic at startDate',
    toGrace: 'automatic at endDate',
    toEnded: 'automatic at graceEnd',
    toArchived: 'manual or after 30 days'
  }
};
```

## Event Content Configuration

### Event Quests

```typescript
interface EventQuest {
  questId: string;
  eventId: string;

  definition: {
    title: string;
    description: string;
    category: 'event';
    difficulty: 'Easy' | 'Medium' | 'Hard' | 'Challenge';
    questType: 'event';
  };

  objectives: {
    type: ObjectiveType;
    target: number;
    eventSpecific: boolean; // Only counts during event
    customTracking?: string;
  }[];

  rewards: {
    eventCurrency?: number;
    eventPoints?: number;
    gold?: number;
    battlePassXP?: number;
    items?: ItemReward[];
  };

  availability: {
    unlockCondition?: string; // e.g., 'complete_quest_X'
    dailyLimit?: number;
    repeatableCount?: number;
  };
}

const EVENT_QUEST_CONFIGURATION = {
  questSlots: {
    standard: {
      count: 6,
      structure: [
        { slot: 1, difficulty: 'Easy', reward: 'low' },
        { slot: 2, difficulty: 'Easy', reward: 'low' },
        { slot: 3, difficulty: 'Medium', reward: 'medium' },
        { slot: 4, difficulty: 'Medium', reward: 'medium' },
        { slot: 5, difficulty: 'Hard', reward: 'high' },
        { slot: 6, difficulty: 'Challenge', reward: 'premium' }
      ]
    },
    miniEvent: {
      count: 3,
      structure: [
        { slot: 1, difficulty: 'Easy', reward: 'low' },
        { slot: 2, difficulty: 'Medium', reward: 'medium' },
        { slot: 3, difficulty: 'Hard', reward: 'high' }
      ]
    }
  },

  questBehavior: {
    displayAlongsideDaily: true,
    separateSection: true,
    progressAcrossMatches: true,
    resetBehavior: 'neverReset', // Event quests don't reset daily
    completionOrder: 'any' // Can complete in any order
  },

  eventCurrency: {
    name: 'Event Tokens',
    iconTemplate: 'themed', // Uses event theme
    earnedFromQuests: true,
    spentInEventShop: true,
    expiresWithEvent: true
  }
};
```

### Event Rewards Track (Mini Battle Pass)

```typescript
interface EventRewardsTrack {
  trackId: string;
  eventId: string;

  structure: {
    totalLevels: number; // Typically 10-20 levels
    freeTrack: boolean;
    premiumTrack: boolean; // Optional premium upgrade
    premiumPrice?: { doubloons: number };
  };

  levels: EventRewardsLevel[];

  progression: {
    pointsPerLevel: number;
    pointsFromQuests: boolean;
    pointsFromMatches: boolean;
    pointsPerMatch: number;
    bonusMultiplier: number; // For premium track
  };

  display: {
    layout: 'horizontal',
    currentLevelHighlight: true,
    claimButton: true,
    previewLocked: true
  };
}

interface EventRewardsLevel {
  level: number;
  pointsRequired: number;
  freeReward: Reward;
  premiumReward?: Reward;
  milestone: boolean; // Special milestone level (5, 10, etc.)
}

const EVENT_REWARDS_TRACK_TEMPLATE = {
  standardEvent: {
    levels: 15,
    pointsPerLevel: 100,

    rewardStructure: [
      // Levels 1-5: Entry rewards
      { levels: [1, 2, 3, 4, 5], freeRewards: 'gold/chests', premiumRewards: 'enhanced' },
      // Levels 6-10: Mid-tier rewards
      { levels: [6, 7, 8, 9, 10], freeRewards: 'cosmetics/currency', premiumRewards: 'exclusives' },
      // Levels 11-14: Premium rewards
      { levels: [11, 12, 13, 14], freeRewards: 'currency', premiumRewards: 'rare_items' },
      // Level 15: Grand prize
      { level: 15, freeReward: 'eventExclusiveItem', premiumReward: 'legendaryItem' }
    ],

    milestones: {
      level5: { special: true, animation: 'enhanced' },
      level10: { special: true, animation: 'enhanced' },
      level15: { special: true, animation: 'grandPrize' }
    }
  },

  miniEvent: {
    levels: 8,
    pointsPerLevel: 75,
    premiumTrack: false // Mini events typically free-only
  }
};
```

### Event Shop Items

```typescript
interface EventShopItem {
  itemId: string;
  eventId: string;

  item: {
    type: 'cosmetic' | 'chest' | 'currency' | 'bundle';
    referenceId: string;
    displayName: string;
    description: string;
  };

  pricing: {
    eventCurrency?: number;
    gold?: number;
    doubloons?: number;
  };

  availability: {
    stock: number | 'unlimited';
    limitPerPlayer: number;
    requiresLevel?: number; // Event track level
  };

  display: {
    featured: boolean;
    sortOrder: number;
    badge?: 'new' | 'limited' | 'best_value';
  };
}

const EVENT_SHOP_CONFIGURATION = {
  layout: {
    sections: [
      {
        name: 'Featured',
        items: 3,
        displayStyle: 'large'
      },
      {
        name: 'Cosmetics',
        items: 'dynamic',
        displayStyle: 'grid'
      },
      {
        name: 'Bundles',
        items: 2,
        displayStyle: 'wide'
      },
      {
        name: 'Currency Exchange',
        items: 3,
        displayStyle: 'compact'
      }
    ]
  },

  eventCurrencyExchange: {
    enabled: true,
    exchangeRate: {
      toGold: 10, // 1 event token = 10 gold
      toDoubloons: 0 // Cannot convert to premium
    },
    endOfEventConversion: {
      enabled: true,
      automaticAtEnd: true,
      notifyPlayer: true
    }
  },

  purchaseTracking: {
    perPlayer: true,
    analytics: true,
    receiptGeneration: true
  }
};
```

## Event UI Theming

```typescript
interface EventUITheme {
  themeId: string;
  eventId: string;

  colors: {
    primary: string;
    secondary: string;
    accent: string;
    background: string;
    text: string;
  };

  assets: {
    logo: string;
    banner: string;
    background: string;
    iconSet: string;
    particles?: string;
    loadingScreen?: string;
  };

  audio: {
    menuMusic?: string;
    questComplete?: string;
    levelUp?: string;
  };

  animations: {
    transitionIn: string;
    transitionOut: string;
    celebrationEffect: string;
  };
}

const UI_THEMING_SYSTEM = {
  applicationScope: {
    mainMenu: {
      banner: true,
      backgroundOverlay: true,
      buttonStyling: false // Keep consistent
    },
    questTab: {
      eventSection: true,
      headerStyling: true,
      questCards: true
    },
    eventPage: {
      fullTheming: true,
      customLayout: true,
      customAnimations: true
    },
    inMatch: {
      questProgress: true,
      hudElements: false // Gameplay clarity
    }
  },

  themeTemplates: {
    halloween: {
      colors: { primary: '#FF6B00', secondary: '#2D0A4E', accent: '#00FF88' },
      mood: 'spooky',
      particles: 'ghosts'
    },
    winter: {
      colors: { primary: '#00BFFF', secondary: '#1a237e', accent: '#FFD700' },
      mood: 'festive',
      particles: 'snowflakes'
    },
    summer: {
      colors: { primary: '#FFD700', secondary: '#006994', accent: '#FF6347' },
      mood: 'tropical',
      particles: 'sunRays'
    },
    anniversary: {
      colors: { primary: '#FFD700', secondary: '#4A0080', accent: '#FF1493' },
      mood: 'celebration',
      particles: 'confetti'
    }
  },

  fallbackBehavior: {
    missingAsset: 'useDefault',
    loadError: 'hideTheming',
    lowMemory: 'disableParticles'
  }
};
```

## Event Game Modes (Optional)

```typescript
interface EventGameMode {
  modeId: string;
  eventId: string;

  definition: {
    name: string;
    description: string;
    basedOn: 'standard' | 'custom';
    ruleset: GameModeRuleset;
  };

  availability: {
    alwaysAvailable: boolean;
    scheduledHours?: { start: number; end: number }[];
    rotationSlot?: number;
  };

  rewards: {
    matchXP: number;
    eventPoints: number;
    eventQuestProgress: boolean;
    standardQuestProgress: boolean;
  };
}

interface GameModeRuleset {
  playerCount: { min: number; max: number };
  matchDuration: { min: number; max: number };

  modifiers: {
    abilityRestrictions?: string[];
    abilityEnhancements?: string[];
    lootModifiers?: object;
    zoneModifiers?: object;
    spawnRules?: object;
  };
}

const EVENT_GAME_MODE_EXAMPLES = {
  cursedSeas: {
    name: 'Cursed Seas',
    description: 'Ghost pirates haunt the map. Survive the curse!',
    modifiers: {
      pveEnemies: true,
      reducedPlayerCount: 30,
      fogEffect: true,
      specialLoot: 'cursed_chests'
    }
  },

  treasureHunt: {
    name: 'Treasure Hunt',
    description: 'Collect treasure for bonus points!',
    modifiers: {
      objectiveOverlay: true,
      treasureSpawns: 'increased',
      combatReduced: false,
      pointsForTreasure: true
    }
  },

  chaosMode: {
    name: 'Chaos Mode',
    description: 'All abilities at maximum power!',
    modifiers: {
      abilityPowerMultiplier: 2.0,
      abilityCooldownReduction: 0.5,
      fastPacedMatch: true
    }
  }
};
```

## Concurrent Events Support

```typescript
const CONCURRENT_EVENTS = {
  maxConcurrentEvents: 3,

  prioritySystem: {
    levels: ['primary', 'secondary', 'tertiary'],
    primaryEventPrivileges: [
      'mainMenuBanner',
      'pushNotificationPriority',
      'featuredShopSection'
    ]
  },

  conflictResolution: {
    uiSpace: {
      bannerRotation: true,
      rotationInterval: '10 seconds',
      eventTabSeparation: true
    },

    questDisplay: {
      separateSections: true,
      collapsible: true,
      sortByPriority: true
    },

    notifications: {
      aggregateIfMultiple: true,
      prioritizeByEndDate: true
    }
  },

  resourceSharing: {
    eventCurrencies: 'separate', // Each event has own currency
    eventProgress: 'separate',
    matchContributions: 'toAll' // Match progress counts for all active events
  },

  scheduling: {
    overlapAllowed: true,
    overlapWarning: true,
    suggestStaggeredDates: true,
    maxOverlapDays: 7
  }
};
```

## Player Notification System

```typescript
const EVENT_NOTIFICATIONS = {
  notificationTypes: [
    {
      type: 'eventAnnouncement',
      timing: '24 hours before start',
      channels: ['push', 'inGame'],
      content: {
        title: '{eventName} is Coming!',
        body: 'Get ready for {eventName} starting {startDate}',
        action: 'openEventPreview'
      }
    },
    {
      type: 'eventStart',
      timing: 'at start',
      channels: ['push', 'inGame'],
      content: {
        title: '{eventName} Has Started!',
        body: 'Jump in now to earn exclusive rewards!',
        action: 'openEvent'
      }
    },
    {
      type: 'eventReminder',
      timing: '48 hours before end',
      channels: ['push'],
      content: {
        title: '{eventName} Ending Soon!',
        body: 'Only {timeRemaining} left to complete your quests!',
        action: 'openEvent'
      }
    },
    {
      type: 'eventEndWarning',
      timing: '6 hours before end',
      channels: ['push', 'inGame'],
      content: {
        title: 'Final Hours of {eventName}!',
        body: 'Claim your rewards before time runs out!',
        action: 'openEvent'
      }
    },
    {
      type: 'eventEnd',
      timing: 'at end',
      channels: ['inGame'],
      content: {
        title: '{eventName} Has Ended',
        body: 'Thanks for participating! Your rewards have been delivered.',
        action: 'viewRewards'
      }
    },
    {
      type: 'gracePeriodEnd',
      timing: '1 hour before grace end',
      channels: ['push', 'inGame'],
      content: {
        title: 'Last Chance to Claim!',
        body: 'Event rewards expire in 1 hour!',
        action: 'claimRewards'
      }
    }
  ],

  playerPreferences: {
    respectOptOut: true,
    frequencyCapping: {
      maxPerEvent: 5,
      minInterval: '4 hours'
    },
    personalizedTiming: {
      basedOnPlaytime: true,
      avoidSleepHours: true
    }
  },

  inGamePresentation: {
    banner: {
      position: 'top',
      autoDismiss: '10 seconds',
      dismissable: true
    },
    popup: {
      forMajorEvents: true,
      showOnce: true,
      dismissable: true
    }
  }
};
```

## Event Lifecycle Management

```typescript
const EVENT_LIFECYCLE = {
  preEvent: {
    activities: [
      'contentPreparation',
      'qaValidation',
      'stagingDeployment',
      'approvalProcess',
      'notificationScheduling'
    ],

    checklist: [
      { item: 'All quests configured', required: true },
      { item: 'Rewards track complete', required: true },
      { item: 'Shop items set', required: true },
      { item: 'Theme assets uploaded', required: true },
      { item: 'Notifications scheduled', required: true },
      { item: 'QA sign-off', required: true },
      { item: 'Rollback plan ready', required: true }
    ]
  },

  duringEvent: {
    monitoring: [
      'participationRate',
      'questCompletionRates',
      'rewardsTrackProgress',
      'shopPurchases',
      'technicalIssues'
    ],

    interventions: {
      adjustRewards: 'if completion too low',
      extendDuration: 'if technical issues caused downtime',
      hotfixBugs: 'immediate deployment',
      disableContent: 'if critical issues'
    },

    communicationPlan: {
      dailyMetricsReview: true,
      issueEscalation: 'immediate',
      playerCommunication: 'as needed'
    }
  },

  postEvent: {
    activities: [
      'finalRewardsDistribution',
      'eventCurrencyConversion',
      'contentRemoval',
      'dataArchival',
      'retrospectiveAnalysis'
    ],

    rewardsDelivery: {
      unclaimedRewards: 'autoDeliverToInventory',
      notifyPlayer: true,
      expiryWarning: 'none' // Delivered items don't expire
    },

    currencyConversion: {
      automatic: true,
      conversionRate: 'eventTokens -> gold',
      notifyPlayer: true
    },

    retrospective: {
      generateReport: true,
      metrics: [
        'participation',
        'completion',
        'revenue',
        'playerSatisfaction',
        'technicalIssues'
      ],
      lessonsLearned: true
    }
  }
};
```

## Event Progress Tracking

```typescript
interface EventProgress {
  playerId: string;
  eventId: string;

  questProgress: {
    questId: string;
    status: 'locked' | 'active' | 'completed' | 'claimed';
    currentProgress: number;
    targetProgress: number;
    completedAt?: Date;
    claimedAt?: Date;
  }[];

  trackProgress: {
    currentLevel: number;
    currentPoints: number;
    pointsToNextLevel: number;
    totalPointsEarned: number;
    claimedLevels: number[];
    premiumUnlocked: boolean;
  };

  shopHistory: {
    itemId: string;
    purchasedAt: Date;
    price: object;
  }[];

  currencyBalance: {
    eventTokens: number;
  };

  participation: {
    firstParticipation: Date;
    lastActivity: Date;
    matchesPlayedDuringEvent: number;
    totalTimePlayed: number;
  };
}

const PROGRESS_TRACKING_SYSTEM = {
  persistence: {
    storage: 'database',
    syncFrequency: 'realtime',
    backupFrequency: 'hourly'
  },

  tracking: {
    questProgress: 'matchEnd',
    trackPoints: 'questCompletion',
    matchParticipation: 'matchEnd'
  },

  display: {
    progressBars: true,
    percentages: true,
    leaderboardPosition: 'optional',
    estimatedCompletion: true
  },

  analytics: {
    trackAllProgress: true,
    cohortAnalysis: true,
    funnelTracking: true
  }
};
```

## Event Content Removal

```typescript
const CONTENT_REMOVAL = {
  removalTiming: {
    questsAndProgress: 'at event end',
    shopAccess: 'at event end',
    uiTheming: 'at event end',
    eventCurrency: 'after grace period',
    claimableRewards: 'after grace period'
  },

  removalProcess: {
    softRemoval: {
      hideFromUI: true,
      disableInteraction: true,
      preserveData: true
    },

    dataArchival: {
      archiveProgress: true,
      archivePurchases: true,
      archiveAnalytics: true,
      retentionPeriod: '2 years'
    },

    cleanup: {
      removeFromCache: '24 hours after end',
      unloadAssets: '24 hours after end',
      databaseCleanup: '30 days after end'
    }
  },

  unclaimedHandling: {
    trackRewards: {
      behavior: 'autoDeliverOnGraceEnd',
      notification: 'inGame',
      receipt: 'email'
    },
    eventCurrency: {
      behavior: 'convertToGold',
      conversionRate: '10 tokens = 100 gold',
      notification: 'inGame'
    },
    shopItems: {
      behavior: 'expired',
      refund: 'none' // Never purchased
    }
  },

  playerCommunication: {
    contentRemovedNotice: true,
    rewardsDeliveredConfirmation: true,
    eventRecapAvailable: true
  }
};
```

## Event Analytics

```typescript
const EVENT_ANALYTICS = {
  realTimeMetrics: {
    dashboard: 'dedicated',
    refresh: '1 minute',

    metrics: [
      'activeParticipants',
      'questsCompletedPerHour',
      'trackProgressDistribution',
      'shopRevenue',
      'eventCurrencyEarned',
      'matchesWithEventProgress'
    ]
  },

  participationAnalysis: {
    funnels: [
      'eventViewed -> questStarted -> questCompleted',
      'eventViewed -> shopViewed -> purchase',
      'eventViewed -> trackViewed -> levelCompleted'
    ],

    segments: [
      'newPlayers',
      'returningPlayers',
      'highSpenders',
      'casualPlayers'
    ],

    cohorts: [
      'byJoinDate',
      'byPlayerLevel',
      'byPreviousEventParticipation'
    ]
  },

  performanceMetrics: {
    questCompletion: {
      overall: true,
      byQuest: true,
      byDifficulty: true,
      timeToComplete: true
    },

    trackProgression: {
      averageLevel: true,
      completionRate: true,
      premiumConversion: true
    },

    shopPerformance: {
      totalRevenue: true,
      byItem: true,
      conversionRate: true,
      averageSpend: true
    }
  },

  postEventReport: {
    generatedAutomatically: true,
    sections: [
      'executiveSummary',
      'participationMetrics',
      'questPerformance',
      'trackProgression',
      'shopRevenue',
      'technicalIssues',
      'playerFeedback',
      'recommendations'
    ],
    distribution: ['liveops', 'product', 'executive']
  }
};
```

## Admin Interface

```typescript
const EVENT_ADMIN_INTERFACE = {
  eventCreation: {
    wizard: {
      steps: [
        'basicInfo',
        'scheduling',
        'questConfiguration',
        'rewardsTrack',
        'shopSetup',
        'theming',
        'notifications',
        'review'
      ]
    },

    templates: {
      miniEvent: 'preconfigured 3-day template',
      standardEvent: 'preconfigured 7-day template',
      majorEvent: 'preconfigured 14-day template',
      custom: 'blank slate'
    },

    cloning: {
      enabled: true,
      copyAll: true,
      updateDates: true,
      resetStatus: true
    }
  },

  eventCalendar: {
    view: 'calendar',
    showDraft: true,
    showScheduled: true,
    showActive: true,
    conflictHighlight: true,
    dragToReschedule: true
  },

  liveControls: {
    pause: {
      enabled: true,
      freezeProgress: true,
      extendDuration: true
    },
    extend: {
      enabled: true,
      maxExtension: '7 days',
      notifyPlayers: true
    },
    earlyEnd: {
      enabled: true,
      requireApproval: true,
      gracePeriodMandatory: true
    },
    emergencyDisable: {
      enabled: true,
      immediateEffect: true,
      compensationTriggered: true
    }
  },

  approvalWorkflow: {
    required: true,
    stages: ['draft', 'review', 'approved', 'scheduled'],
    approvers: ['liveopsLead', 'productManager'],
    audit: true
  }
};
```

## Integration with Other Systems

```typescript
const SYSTEM_INTEGRATIONS = {
  questSystem: {
    eventQuestsInQuestTab: true,
    separateSection: true,
    progressSharedWhereApplicable: true
  },

  battlePass: {
    eventXPContributesToBP: true,
    bpProgressFromEventQuests: true,
    separateFromEventTrack: true
  },

  shop: {
    eventShopAsTab: true,
    eventCurrencyVisible: true,
    bundleIntegration: true
  },

  notifications: {
    unifiedSystem: true,
    eventNotificationPriority: 'high',
    respectPlayerPreferences: true
  },

  analytics: {
    eventMetricsInMainDashboard: true,
    dedicatedEventDashboard: true,
    crossEventComparison: true
  },

  matchmaking: {
    eventModeInRotation: true,
    separateQueueIfNeeded: true,
    priorityDuringEvent: true
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_IMPLEMENTATION = {
  dataStructure: {
    eventDefinition: 'mongodb',
    playerProgress: 'mongodb + redis cache',
    analytics: 'clickhouse',
    assets: 'cdn'
  },

  apiEndpoints: {
    getActiveEvents: 'GET /api/events/active',
    getEventDetails: 'GET /api/events/{eventId}',
    getPlayerProgress: 'GET /api/events/{eventId}/progress/{playerId}',
    claimReward: 'POST /api/events/{eventId}/claim',
    purchaseItem: 'POST /api/events/{eventId}/shop/purchase'
  },

  caching: {
    eventDefinitions: '5 minutes',
    playerProgress: '30 seconds',
    shopInventory: '1 minute'
  },

  failover: {
    eventUnavailable: 'hideGracefully',
    progressSyncFailed: 'queueForRetry',
    purchaseFailed: 'retryWithIdempotency'
  },

  performance: {
    targetLatency: '<100ms',
    concurrentPlayers: '100,000+',
    loadTesting: 'required'
  }
};
```
