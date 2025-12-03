# Player Level System Specification

## Overview

The Player Level System provides persistent account-wide progression that tracks a player's overall experience with the game. Unlike Battle Pass levels which reset seasonally, account level persists forever and represents a player's lifetime investment in the game.

## Level Structure

```typescript
interface PlayerLevel {
  playerId: string;
  currentLevel: number;
  currentXP: number;
  xpToNextLevel: number;
  totalXP: number;
  prestigeLevel: number; // For players who hit max level
}

const LEVEL_CONFIGURATION = {
  startingLevel: 1,
  softCap: 500, // Normal progression
  hardCap: null, // No hard cap, continues infinitely

  // XP requirements per level band
  xpRequirements: [
    { levelRange: [1, 10], xpPerLevel: 1000 },
    { levelRange: [11, 30], xpPerLevel: 2000 },
    { levelRange: [31, 50], xpPerLevel: 3000 },
    { levelRange: [51, 100], xpPerLevel: 5000 },
    { levelRange: [101, 200], xpPerLevel: 7500 },
    { levelRange: [201, 500], xpPerLevel: 10000 },
    { levelRange: [501, Infinity], xpPerLevel: 15000 }
  ],

  // Level milestones
  milestones: [5, 10, 25, 50, 75, 100, 150, 200, 300, 400, 500],

  // Feature unlocks
  featureUnlocks: [
    { level: 5, feature: 'dailyQuests' },
    { level: 10, feature: 'rankedMode' },
    { level: 15, feature: 'weeklyQuests' },
    { level: 20, feature: 'customLoadouts' },
    { level: 25, feature: 'duoMode' },
    { level: 30, feature: 'squadMode' }
  ]
};
```

## XP Sources

```typescript
const XP_SOURCES = {
  matchXP: {
    // Base match XP
    baseXP: {
      participation: 50, // Just for completing a match
      minimumMatchLength: '3 minutes'
    },

    // Placement XP (Solo)
    placementXP: {
      placement1: 300,
      placement2to5: 200,
      placement6to10: 150,
      placement11to20: 100,
      placement21to30: 75,
      placement31to40: 50,
      placement41plus: 25
    },

    // Performance XP
    performanceXP: {
      perElimination: 25,
      perAssist: 15,
      per500Damage: 20,
      perMinuteSurvived: 5,
      perChestOpened: 5,
      perAbilityUsed: 2, // Capped at 50 per match
      perRevive: 30 // Team modes
    },

    // Bonus XP
    bonusXP: {
      firstWinOfDay: 150, // Stacks with FWOTD gold
      winBonus: 100, // Any win
      squadBonus: 50, // Playing with friends
      eventBonus: 'varies' // During special events
    }
  },

  questXP: {
    dailyQuestCompletion: 'included in quest rewards', // BP XP, not account XP
    weeklyQuestCompletion: 'included in quest rewards',
    achievementUnlock: 100 // Per achievement
  },

  otherXP: {
    dailyLoginBonus: 50, // Small XP with login rewards
    tutorialCompletion: 500, // One-time
    profileCompletion: 200 // Link accounts, set avatar, etc.
  },

  // XP modifiers
  modifiers: {
    premiumBattlePass: 1.1, // +10% XP
    xpBooster: 1.5, // Consumable item
    eventBonus: 'varies', // Double XP weekends, etc.
    stackingRules: 'multiplicative'
  }
};
```

## Level Up Rewards

```typescript
const LEVEL_REWARDS = {
  // Standard level-up rewards (every level)
  standardRewards: {
    everyLevel: { gold: 50 },
    every5Levels: { gold: 100, chest: 'common' },
    every10Levels: { gold: 200, chest: 'rare' },
    every25Levels: { gold: 500, chest: 'epic', doubloons: 10 },
    every50Levels: { gold: 1000, chest: 'legendary', doubloons: 25 },
    every100Levels: { gold: 2000, chest: 'legendary', doubloons: 50, exclusiveCosmetic: true }
  },

  // Milestone rewards
  milestoneRewards: [
    { level: 5, rewards: { gold: 200, title: 'Newcomer' } },
    { level: 10, rewards: { gold: 300, chest: 'rare', feature: 'rankedMode', title: 'Sailor' } },
    { level: 25, rewards: { gold: 500, chest: 'epic', doubloons: 20, badge: 'level25' } },
    { level: 50, rewards: { gold: 1000, chest: 'legendary', doubloons: 50, outfit: 'level50Exclusive', title: 'Veteran' } },
    { level: 75, rewards: { gold: 1500, chest: 'legendary', doubloons: 75, badge: 'level75' } },
    { level: 100, rewards: { gold: 2500, chest: 'legendary', doubloons: 100, outfit: 'centurion', title: 'Centurion', playerFrame: 'golden' } },
    { level: 150, rewards: { gold: 3000, doubloons: 150, badge: 'level150', emote: 'exclusive' } },
    { level: 200, rewards: { gold: 5000, doubloons: 200, outfit: 'legend', title: 'Legend' } },
    { level: 300, rewards: { gold: 7500, doubloons: 300, playerFrame: 'diamond' } },
    { level: 500, rewards: { gold: 10000, doubloons: 500, outfit: 'prestigeReady', title: 'Grand Admiral', prestige: 'unlocked' } }
  ],

  // Feature unlocks at levels
  featureUnlockRewards: {
    level5: { feature: 'dailyQuests', message: 'Daily Quests Unlocked!' },
    level10: { feature: 'rankedMode', message: 'Ranked Mode Unlocked!' },
    level15: { feature: 'weeklyQuests', message: 'Weekly Quests Unlocked!' },
    level20: { feature: 'customLoadouts', message: 'Custom Loadouts Unlocked!' },
    level25: { feature: 'duoMode', message: 'Duo Mode Unlocked!' },
    level30: { feature: 'squadMode', message: 'Squad Mode Unlocked!' }
  }
};
```

## UI Integration

```typescript
const UI_INTEGRATION = {
  // Profile display
  profileDisplay: {
    levelBadge: {
      location: 'playerCard',
      style: 'circularWithBorder',
      borderColorByTier: {
        '1-24': 'bronze',
        '25-49': 'silver',
        '50-99': 'gold',
        '100-199': 'platinum',
        '200-499': 'diamond',
        '500+': 'legendary'
      }
    },

    xpBar: {
      location: 'belowPlayerCard',
      style: 'horizontalProgressBar',
      showNumerical: true, // "1250/2000 XP"
      showPercentage: false,
      animateOnGain: true
    },

    levelNumber: {
      format: 'Level {N}',
      prestigeFormat: 'Prestige {P} - Level {N}'
    }
  },

  // Main menu
  mainMenu: {
    xpBarLocation: 'topBar',
    style: 'compact',
    showOnlyProgress: true, // Not full bar
    clickToExpand: true
  },

  // Post-match screen
  postMatch: {
    xpGainDisplay: {
      location: 'resultsScreen',
      style: 'animatedFlyIn',
      breakdown: true, // Show sources
      showLevelProgress: true
    },

    levelUpCelebration: {
      trigger: 'onLevelUp',
      animation: 'fullScreenBurst',
      sound: 'levelUpFanfare',
      showRewards: true,
      duration: '5 seconds'
    }
  },

  // Nameplate
  nameplate: {
    showLevel: true,
    format: '[{level}] {playerName}',
    colorByTier: true,
    toggleOption: true // Player can hide
  }
};
```

## XP Progress Bar

```typescript
const XP_PROGRESS_BAR = {
  display: {
    style: 'segmentedProgress',
    showCurrentLevel: true,
    showNextLevel: true,
    showXPNumbers: true,
    format: '{current}/{required} XP'
  },

  animation: {
    xpGainAnimation: 'fillFromLeft',
    levelUpAnimation: 'burstAndReset',
    speed: 'matchedToXPAmount', // More XP = longer animation
    sound: 'xpTick'
  },

  interactivity: {
    hoverDetails: true,
    clickToExpand: true,
    expandedView: {
      showXPSources: true,
      showNextRewards: true,
      showMilestoneProgress: true
    }
  },

  compactMode: {
    enabled: true,
    location: 'mainMenuHeader',
    showOnlyBar: true,
    miniLevel: true
  }
};
```

## Level Up Celebration

```typescript
const LEVEL_UP_CELEBRATION = {
  standard: {
    trigger: 'normalLevelUp',
    animation: {
      type: 'radialBurst',
      color: 'gold',
      particles: true,
      duration: '2 seconds'
    },
    sound: 'levelUp',
    haptic: 'medium',
    showReward: true
  },

  milestone: {
    trigger: 'milestoneLevelUp',
    animation: {
      type: 'fullScreenCelebration',
      color: 'rainbow',
      particles: true,
      duration: '4 seconds'
    },
    sound: 'milestoneFanfare',
    haptic: 'heavy',
    showReward: true,
    showUnlock: true // If feature unlocked
  },

  featureUnlock: {
    trigger: 'featureUnlockLevel',
    animation: {
      type: 'featureSpotlight',
      highlightFeature: true,
      duration: '5 seconds'
    },
    sound: 'unlockFanfare',
    message: '{feature} Unlocked!',
    tutorialPrompt: true
  },

  rewardDisplay: {
    layout: 'centered',
    showItems: true,
    animateItems: true,
    claimButton: false, // Auto-claimed
    dismissable: true
  }
};
```

## Milestone System

```typescript
const MILESTONE_SYSTEM = {
  milestones: [
    {
      level: 10,
      name: 'Rookie',
      badge: 'rookieBadge',
      celebration: 'enhanced',
      featureUnlock: 'rankedMode'
    },
    {
      level: 25,
      name: 'Seasoned',
      badge: 'seasonedBadge',
      celebration: 'enhanced',
      exclusiveReward: true
    },
    {
      level: 50,
      name: 'Veteran',
      badge: 'veteranBadge',
      celebration: 'major',
      exclusiveOutfit: true
    },
    {
      level: 100,
      name: 'Centurion',
      badge: 'centurionBadge',
      celebration: 'major',
      playerFrame: 'golden',
      exclusiveOutfit: true
    },
    {
      level: 200,
      name: 'Legend',
      badge: 'legendBadge',
      celebration: 'legendary',
      exclusiveOutfit: true,
      title: 'Legend'
    },
    {
      level: 500,
      name: 'Grand Admiral',
      badge: 'grandAdmiralBadge',
      celebration: 'legendary',
      prestigeUnlock: true,
      exclusiveOutfit: true
    }
  ],

  milestoneTracking: {
    showNextMilestone: true,
    progressToMilestone: true,
    milestoneRewardPreview: true
  },

  milestoneGallery: {
    location: 'profileTab',
    showAchieved: true,
    showUpcoming: true,
    showRewards: true
  }
};
```

## Feature Gating

```typescript
const FEATURE_GATING = {
  gatedFeatures: [
    {
      feature: 'dailyQuests',
      unlockLevel: 5,
      reason: 'Ensure basic understanding first',
      previewAvailable: true
    },
    {
      feature: 'rankedMode',
      unlockLevel: 10,
      reason: 'Ensure baseline skill and understanding',
      previewAvailable: true,
      additionalRequirements: ['completeTutorial']
    },
    {
      feature: 'weeklyQuests',
      unlockLevel: 15,
      reason: 'After players understand daily quests',
      previewAvailable: true
    },
    {
      feature: 'customLoadouts',
      unlockLevel: 20,
      reason: 'Advanced feature after basics mastered',
      previewAvailable: false
    },
    {
      feature: 'duoMode',
      unlockLevel: 25,
      reason: 'Social feature after solo mastery',
      previewAvailable: true
    },
    {
      feature: 'squadMode',
      unlockLevel: 30,
      reason: 'Team play after duo experience',
      previewAvailable: true
    }
  ],

  gatedUI: {
    lockedState: {
      visible: true,
      dimmed: true,
      lockIcon: true,
      unlockRequirement: 'Unlocks at Level {X}'
    },

    previewMode: {
      showDescription: true,
      showPreviewVideo: false,
      countdownToUnlock: true
    },

    unlockAnimation: {
      trigger: 'onReachingLevel',
      animation: 'unlockReveal',
      tutorialPrompt: true
    }
  }
};
```

## Prestige System (Post Level 500)

```typescript
const PRESTIGE_SYSTEM = {
  unlockLevel: 500,
  optional: true, // Player chooses to prestige

  prestigeProcess: {
    confirmation: true,
    resetLevel: true, // Back to level 1
    preserveUnlocks: true, // Keep all feature unlocks
    preserveCosmetics: true, // Keep all earned items
    prestigeBadge: true,
    prestigeCounter: true
  },

  prestigeBonuses: {
    perPrestigeLevel: {
      xpMultiplier: 0.05, // +5% XP per prestige
      exclusiveCosmetic: true,
      prestigeBorder: true
    },

    maxPrestige: 10,
    maxBonuses: {
      xpMultiplier: 1.5, // +50% at max prestige
      masterTitle: 'Grand Master',
      uniqueEffects: true
    }
  },

  prestigeRewards: [
    { prestige: 1, reward: { outfit: 'prestige1', border: 'bronze_prestige' } },
    { prestige: 2, reward: { outfit: 'prestige2', border: 'silver_prestige' } },
    { prestige: 3, reward: { outfit: 'prestige3', border: 'gold_prestige' } },
    { prestige: 5, reward: { outfit: 'prestige5', border: 'platinum_prestige', title: 'Prestige Master' } },
    { prestige: 10, reward: { outfit: 'prestige10', border: 'legendary_prestige', title: 'Grand Master', effect: 'aura' } }
  ],

  prestigeDisplay: {
    format: 'P{X} Level {Y}',
    starIcons: true, // Stars for prestige level
    distinctiveStyling: true
  }
};
```

## Analytics and Metrics

```typescript
const LEVEL_ANALYTICS = {
  progressionMetrics: {
    averageLevelByDaysPlayed: true,
    levelDistribution: true,
    xpSourcesBreakdown: true,
    timeToLevel: true,
    dropoffPoints: true // Where players stop progressing
  },

  engagementCorrelation: {
    levelVsRetention: true,
    levelVsSpending: true,
    levelVsSessionLength: true,
    unlockLevelVsFeatureUsage: true
  },

  milestonetTracking: {
    milestoneReachRate: true,
    timeToMilestone: true,
    postMilestoneRetention: true
  },

  xpEconomyHealth: {
    averageXPPerSession: true,
    xpInflation: false, // Monitor for balance
    boostedVsOrganicXP: true
  },

  reporting: {
    dailyProgressReport: true,
    weeklyLevelDistribution: true,
    monthlyMilestoneAnalysis: true
  }
};
```

## Admin Configuration

```typescript
const ADMIN_CONFIG = {
  xpAdjustment: {
    modifyXPSources: true,
    modifyLevelRequirements: true,
    previewImpact: true,
    approvalRequired: true
  },

  rewardConfiguration: {
    modifyLevelRewards: true,
    addMilestones: true,
    modifyMilestoneRewards: true,
    versionControl: true
  },

  playerSupport: {
    viewPlayerLevel: true,
    grantXP: true,
    adjustLevel: true,
    resetPrestige: true,
    requiresTicket: true
  },

  featureGating: {
    modifyUnlockLevels: true,
    temporaryUnlocks: true, // For events
    abTestUnlocks: true
  },

  xpEvents: {
    createDoubleXPEvent: true,
    scheduleXPBonuses: true,
    targetedXPBoosts: true // For segments
  }
};
```

## Integration Points

```typescript
const INTEGRATIONS = {
  matchSystem: {
    xpCalculation: 'matchEndEvent',
    xpGrant: 'postMatchRewards'
  },

  battlePass: {
    separateProgression: true,
    premiumXPBonus: 'toAccountXP',
    noXPSharing: true
  },

  questSystem: {
    questXPSeparate: true, // Quests give BP XP, not account XP
    achievementXP: 'toAccountLevel'
  },

  currencySystem: {
    levelUpRewards: 'directGrant',
    transactionLog: true
  },

  socialSystem: {
    levelOnProfile: true,
    levelInChat: true,
    levelOnNameplate: true
  },

  matchmaking: {
    levelNotUsedForMM: true, // Skill-based instead
    levelDisplayInMatch: true
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  dataModel: {
    playerLevelData: 'mongodb',
    xpHistory: 'mongodb',
    realtimeXP: 'redis'
  },

  apiEndpoints: {
    getPlayerLevel: 'GET /api/player-level/{playerId}',
    grantXP: 'POST /api/player-level/grant-xp (internal)',
    getXPHistory: 'GET /api/player-level/history',
    getLeaderboard: 'GET /api/player-level/leaderboard'
  },

  xpGrantFlow: {
    source: 'matchService or questService',
    calculation: 'xpCalculationService',
    grant: 'playerLevelService',
    notification: 'websocket push'
  },

  caching: {
    playerLevel: 'redis, 5 min TTL',
    levelRequirements: 'memory, refresh hourly',
    invalidateOnLevelUp: true
  },

  performance: {
    xpGrantLatency: '<50ms',
    levelCheckLatency: '<20ms',
    concurrentUpdates: '10000/s'
  },

  reliability: {
    idempotentXPGrant: true,
    transactionLogging: true,
    rollbackCapability: true
  }
};
```
