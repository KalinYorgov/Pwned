# Come Back Rewards Specification

## Document Information
- **Task ID:** LIVE-011
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the come back rewards system for Plunderstorm Mobile, designed to re-engage lapsed players with enticing rewards, a warm welcome, and a smooth reintroduction to the game's current state.

---

## Lapsed Player Detection

### Absence Thresholds

```typescript
const ABSENCE_THRESHOLDS = {
  tiers: {
    shortAbsence: {
      days: 7,
      label: 'Short Break',
      eligibility: 'Basic come back rewards',
      rewardTier: 1
    },
    mediumAbsence: {
      days: 14,
      label: 'Extended Break',
      eligibility: 'Enhanced come back rewards',
      rewardTier: 2
    },
    longAbsence: {
      days: 30,
      label: 'Long Absence',
      eligibility: 'Premium come back rewards',
      rewardTier: 3
    },
    veryLongAbsence: {
      days: 60,
      label: 'Major Return',
      eligibility: 'Maximum come back rewards + tutorial refresh',
      rewardTier: 4
    }
  },

  calculation: {
    metric: 'Days since last login (any session)',
    timezone: 'UTC (server time)',
    reset: 'Resets upon successful login'
  }
};
```

### Eligibility Rules

```typescript
const ELIGIBILITY_RULES = {
  inclusion: {
    minimumLevel: 5,
    minimumPreviousMatches: 10,
    previousPurchase: 'Not required',
    accountAge: 'At least 14 days old'
  },

  exclusion: {
    bannedAccounts: 'Not eligible',
    newAccounts: 'Accounts < 14 days old',
    neverPlayed: 'Must have played at least 10 matches previously'
  },

  frequency: {
    cooldown: '30 days between come back reward eligibility',
    example: 'If player claims rewards, must wait 30 days before next absence triggers rewards',
    reason: 'Prevent intentional churn for rewards'
  }
};
```

---

## Welcome Back Flow

### Return Experience

```typescript
const RETURN_EXPERIENCE = {
  loginDetection: {
    trigger: 'On successful login after absence threshold',
    timing: 'Before reaching main menu',
    priority: 'Highest (before other popups)'
  },

  welcomePopup: {
    content: {
      greeting: 'Welcome Back, Captain!',
      personalMessage: 'We\'ve missed you on the high seas!',
      absenceDuration: 'It\'s been {X} days since your last voyage',
      teaser: 'Here\'s what awaits you...'
    },
    visuals: {
      background: 'Animated ship returning to port',
      character: 'Crew waving welcome',
      treasure: 'Chest of rewards visible',
      mood: 'Warm, celebratory'
    },
    actions: {
      primary: 'Claim Your Rewards',
      secondary: 'See What\'s New',
      dismiss: 'Not prominently shown'
    }
  },

  flowSteps: [
    { step: 1, name: 'Welcome Popup', required: true },
    { step: 2, name: 'Reward Claim', required: true },
    { step: 3, name: 'What\'s New', required: false },
    { step: 4, name: 'Tutorial Refresh', conditional: 'If 60+ days absent' },
    { step: 5, name: 'Main Menu', destination: true }
  ]
};
```

### Reward Claim Screen

```typescript
const REWARD_CLAIM = {
  display: {
    layout: 'Treasure chest opening animation',
    items: 'All rewards revealed with celebration',
    value: 'Total value displayed (e.g., "$10 value!")'
  },

  claimProcess: {
    automatic: false,
    interaction: 'Tap to open chest / claim',
    animation: 'Satisfying chest opening, items flying to inventory',
    confirmation: 'Rewards added to your account!'
  },

  postClaim: {
    summary: 'Quick summary of all received items',
    nextStep: 'View What\'s New or Go to Main Menu'
  }
};
```

---

## Reward Tiers

### Tier 1: Short Break (7-13 days)

```typescript
const TIER_1_REWARDS = {
  tier: 1,
  absenceDays: '7-13',
  label: 'Welcome Back Pack',

  rewards: {
    gold: 500,
    doubloons: 0,
    chest: {
      type: 'Common Chest',
      count: 1
    },
    booster: {
      type: 'XP Boost (25%)',
      duration: '24 hours'
    }
  },

  estimatedValue: '$2-3',
  message: 'Good to see you back, Captain!'
};
```

### Tier 2: Extended Break (14-29 days)

```typescript
const TIER_2_REWARDS = {
  tier: 2,
  absenceDays: '14-29',
  label: 'Prodigal Pirate Pack',

  rewards: {
    gold: 1000,
    doubloons: 50,
    chest: {
      type: 'Rare Chest',
      count: 1
    },
    booster: {
      type: 'XP Boost (50%)',
      duration: '48 hours'
    },
    bonus: {
      type: 'First Win Bonus',
      value: '2x rewards for next 3 wins'
    }
  },

  estimatedValue: '$5-7',
  message: 'The crew is thrilled to have you back!'
};
```

### Tier 3: Long Absence (30-59 days)

```typescript
const TIER_3_REWARDS = {
  tier: 3,
  absenceDays: '30-59',
  label: 'Returning Legend Pack',

  rewards: {
    gold: 2000,
    doubloons: 100,
    chest: {
      type: 'Epic Chest',
      count: 1
    },
    booster: {
      type: 'XP Boost (100%)',
      duration: '72 hours'
    },
    bonus: {
      type: 'First Win Bonus',
      value: '3x rewards for next 5 wins'
    },
    special: {
      type: 'Returning Player Banner',
      exclusive: true
    }
  },

  estimatedValue: '$10-15',
  message: 'A legend returns! The seas have awaited you!'
};
```

### Tier 4: Major Return (60+ days)

```typescript
const TIER_4_REWARDS = {
  tier: 4,
  absenceDays: '60+',
  label: 'Phoenix Pirate Pack',

  rewards: {
    gold: 5000,
    doubloons: 200,
    chest: {
      type: 'Legendary Chest',
      count: 1
    },
    booster: {
      type: 'XP Boost (100%)',
      duration: '7 days'
    },
    bonus: {
      type: 'First Win Bonus',
      value: '3x rewards for next 10 wins'
    },
    special: [
      {
        type: 'Returning Legend Title',
        exclusive: true
      },
      {
        type: 'Phoenix Trail',
        exclusive: true
      }
    ],
    battlePass: {
      type: 'Free tier skip tokens',
      count: 5,
      note: 'If Battle Pass active'
    }
  },

  estimatedValue: '$25-30',
  message: 'The Phoenix rises! Welcome back, legendary Captain!',

  tutorialRefresh: {
    offered: true,
    optional: true,
    content: 'Quick refresher on controls and new features'
  }
};
```

---

## What's New Summary

### Content Highlights

```typescript
const WHATS_NEW = {
  display: {
    format: 'Carousel of highlights',
    maxItems: 5,
    priority: 'Most recent and relevant first'
  },

  categories: {
    newFeatures: {
      label: 'New Features',
      icon: 'star',
      examples: ['New game mode', 'New ability', 'New map area']
    },
    currentEvents: {
      label: 'Current Events',
      icon: 'calendar',
      examples: ['Seasonal event active', 'Limited-time mode', 'Community challenge']
    },
    battlePass: {
      label: 'Battle Pass',
      icon: 'pass',
      examples: ['Current season', 'Days remaining', 'Featured rewards']
    },
    shopHighlights: {
      label: 'Shop Highlights',
      icon: 'shop',
      examples: ['New items', 'Limited offers', 'Returning player discount']
    },
    balanceChanges: {
      label: 'Balance Updates',
      icon: 'scales',
      examples: ['Major ability changes', 'Meta shifts']
    }
  },

  personalization: {
    enabled: true,
    factors: [
      'Player\'s previous favorite abilities (highlight changes)',
      'Player\'s spending history (highlight relevant offers)',
      'Time since absence (more content for longer absence)'
    ]
  }
};
```

### Content Selection

```typescript
const CONTENT_SELECTION = {
  automatic: {
    source: 'Content management system flags',
    criteria: 'Published since player last login',
    ordering: 'By importance and recency'
  },

  manual: {
    liveOpsControl: 'Can feature specific content for returnees',
    priority: 'Manual selections shown first'
  },

  fallback: {
    noNewContent: 'Show evergreen tips or feature highlights',
    minimum: 'Always show at least 1 item'
  }
};
```

---

## Tutorial Refresh

### Refresher Flow

```typescript
const TUTORIAL_REFRESH = {
  trigger: {
    absenceThreshold: 60,
    optional: true,
    prompt: 'Would you like a quick refresher on the game?'
  },

  content: {
    controls: {
      name: 'Controls Refresher',
      duration: '30 seconds',
      content: 'Quick reminder of movement and combat controls'
    },
    newMechanics: {
      name: 'What\'s Changed',
      duration: '1-2 minutes',
      content: 'Overview of new mechanics added since absence'
    },
    currentMeta: {
      name: 'Current Tips',
      duration: '30 seconds',
      content: 'Brief tips on current popular strategies'
    }
  },

  format: {
    type: 'Non-intrusive tooltips and guided highlights',
    skippable: true,
    bookmark: 'Can return to refresher from settings'
  }
};
```

---

## Returning Player Offers

### Special Discounts

```typescript
const RETURNING_OFFERS = {
  enabled: true,
  duration: '7 days from return',
  visibility: 'Highlighted in shop',

  offers: {
    starterRefresh: {
      name: 'Second Chance Starter Pack',
      content: ['1000 Doubloons', 'Epic Chest', 'Exclusive Outfit'],
      originalPrice: 19.99,
      discount: '50%',
      returnPrice: 9.99,
      limit: 'One-time per return'
    },
    battlePassDiscount: {
      name: 'Battle Pass Welcome Back',
      discount: '25%',
      condition: 'If Battle Pass not already owned',
      duration: '48 hours'
    },
    gemDiscount: {
      name: 'Returning Hero Bonus',
      type: 'Extra 30% gems on first purchase',
      duration: '72 hours'
    }
  },

  display: {
    shopBanner: 'Welcome Back Deals section',
    countdown: 'Time remaining on offers',
    emphasis: 'Limited time, just for you'
  }
};
```

---

## Notification Strategy

### Win-Back Notifications

```typescript
const WINBACK_NOTIFICATIONS = {
  schedule: {
    day3: {
      message: 'Your crew misses you! Come back for a free daily reward.',
      cta: 'Open Plunderstorm',
      incentive: 'Reminder of daily login bonus'
    },
    day7: {
      message: 'Captain! Welcome Back rewards await you!',
      cta: 'Claim Your Rewards',
      incentive: 'Preview of Tier 1 rewards'
    },
    day14: {
      message: 'Special treasure chest waiting for returning pirates!',
      cta: 'Open Your Chest',
      incentive: 'Preview of Tier 2 rewards'
    },
    day30: {
      message: 'Legendary loot awaits! Return to claim your fortune.',
      cta: 'Claim Legendary Rewards',
      incentive: 'Preview of Tier 3 rewards'
    }
  },

  rules: {
    optOut: 'Respect user notification preferences',
    frequency: 'Max 1 win-back notification per 3 days',
    cap: 'Stop after 4 notifications if no return',
    reactivate: 'If user returns and churns again, reset sequence'
  }
};
```

### Email Campaign

```typescript
const EMAIL_CAMPAIGN = {
  enabled: true,
  triggers: {
    day7: 'First win-back email',
    day14: 'Second email with increased rewards preview',
    day30: 'Final email with maximum offer'
  },

  content: {
    personalization: [
      'Player name',
      'Last played date',
      'Favorite ability/mode',
      'Missed content highlights'
    ],
    visuals: 'High-quality game art and reward previews',
    cta: 'Deep link to open game'
  },

  compliance: {
    unsubscribe: 'Easy unsubscribe option',
    gdpr: 'Compliant with data regulations',
    canSpam: 'Compliant with email regulations'
  }
};
```

---

## Analytics and Tracking

### Metrics

```typescript
const COMEBACK_METRICS = {
  acquisition: {
    returnRate: 'Percentage of lapsed players who return',
    returnByTier: 'Returns segmented by absence duration',
    returnSource: 'Attribution (notification, email, organic)'
  },

  engagement: {
    rewardClaimRate: 'Percentage who claim rewards',
    whatIsNewViews: 'Engagement with What\'s New content',
    sessionDuration: 'First session length vs. typical',
    matchesPlayed: 'Matches in first 24 hours'
  },

  retention: {
    d1Retention: 'Day 1 retention of returnees',
    d7Retention: 'Day 7 retention of returnees',
    d30Retention: 'Day 30 retention of returnees',
    reChurnRate: 'Percentage who churn again within 30 days'
  },

  monetization: {
    conversionRate: 'Returnees who make purchase',
    offerTakeRate: 'Uptake on returning player offers',
    arpu: 'Revenue from returning players'
  }
};
```

### A/B Testing

```typescript
const COMEBACK_AB_TESTING = {
  testableElements: {
    rewardComposition: 'Different reward mixes',
    rewardValue: 'Higher vs lower reward values',
    messaging: 'Different welcome messages',
    offerDiscount: 'Discount depth on returning offers',
    notificationTiming: 'Optimal notification schedule'
  },

  segments: {
    byAbsence: 'Test within each tier',
    bySpender: 'Test differently for spenders vs non-spenders',
    byEngagement: 'Test by previous engagement level'
  },

  metrics: {
    primary: 'D7 retention of returnees',
    secondary: ['Return rate', 'Revenue', 'Re-churn rate']
  }
};
```

---

## Anti-Exploitation

### Abuse Prevention

```typescript
const ABUSE_PREVENTION = {
  intentionalChurn: {
    detection: {
      pattern: 'Regular absence at exactly threshold days',
      frequency: 'Multiple cycles of churn-return',
      timing: 'Returns immediately on threshold'
    },
    mitigation: {
      cooldown: '30-day cooldown between rewards',
      diminishing: 'Reduced rewards on 3rd+ cycle',
      flag: 'Flag accounts for review'
    }
  },

  multiAccount: {
    detection: 'Device/IP fingerprinting',
    prevention: 'Rewards tied to device, not just account',
    limitation: 'One set of come back rewards per device per period'
  },

  botAccounts: {
    detection: 'Unusual return patterns at scale',
    prevention: 'Manual review for suspicious activity',
    action: 'Revoke rewards, potential ban'
  }
};
```

---

## Technical Implementation

### Backend System

```typescript
const BACKEND_SYSTEM = {
  detection: {
    lastLoginTracking: 'Update on every login',
    absenceCalculation: 'Current time - last login',
    eligibilityCheck: 'On login, check all eligibility rules'
  },

  rewardGrant: {
    atomic: 'All rewards granted in single transaction',
    logging: 'Full audit trail of granted rewards',
    idempotent: 'Prevent duplicate grants'
  },

  configuration: {
    rewardTiers: 'Configurable via admin panel',
    thresholds: 'Adjustable absence thresholds',
    offers: 'Dynamic returning player offers'
  }
};
```

### Data Model

```typescript
const DATA_MODEL = {
  playerComebackState: {
    playerId: 'string',
    lastLoginAt: 'timestamp',
    lastComebackClaimAt: 'timestamp (nullable)',
    comebackCount: 'number',
    eligibleForRewards: 'boolean',
    currentTier: 'number (nullable)'
  },

  comebackRewardLog: {
    id: 'string',
    playerId: 'string',
    tier: 'number',
    absenceDays: 'number',
    rewardsGranted: 'array<Reward>',
    claimedAt: 'timestamp',
    source: 'string (notification, organic, email)'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Correct tier calculation based on absence',
    'Rewards granted match tier configuration',
    'Welcome flow displays correctly',
    'What\'s New shows relevant content',
    'Offers apply correct discounts'
  ],

  edge: [
    'Player at exact threshold (day 7, 14, 30, 60)',
    'Player returns just before threshold',
    'Multiple returns in short period (cooldown)',
    'Return during active event',
    'Return with active Battle Pass vs none'
  ],

  integration: [
    'Works with notification system',
    'Integrates with shop/offers',
    'Compatible with other popups/flows',
    'Analytics events fire correctly'
  ],

  exploitation: [
    'Cooldown prevents rapid cycling',
    'Multi-account prevention works',
    'Diminishing returns on repeated use'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial come back rewards specification |
