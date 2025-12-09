# Referral Program Specification

## Document Information
- **Task ID:** LIVE-018
- **Priority:** P3
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Referral Program for Plunderstorm Mobile, enabling players to invite friends and earn rewards while driving organic player acquisition through word-of-mouth marketing.

---

## Program Philosophy

### Design Goals

```typescript
const REFERRAL_GOALS = {
  acquisition: {
    description: 'Drive organic player growth',
    target: '10-15% of new players from referrals',
    cost: 'Lower CAC than paid acquisition'
  },

  quality: {
    description: 'Referred players are higher quality',
    rationale: 'Social connection increases retention',
    metric: 'Referred players have 20%+ higher D30 retention'
  },

  fairness: {
    description: 'Rewards feel valuable but balanced',
    principle: 'Worth the effort, not exploitable',
    protection: 'Robust fraud prevention'
  },

  simplicity: {
    description: 'Easy to understand and use',
    principle: 'One tap to share, clear rewards',
    friction: 'Minimal steps to participate'
  }
};
```

---

## Referral Code System

### Code Generation

```typescript
const REFERRAL_CODES = {
  format: {
    structure: '{PREFIX}{UNIQUE_ID}',
    prefix: 'PLUNDER',
    uniqueId: '6 alphanumeric characters',
    example: 'PLUNDER7X9K2M',
    caseInsensitive: true
  },

  generation: {
    timing: 'Generated on account creation',
    persistence: 'Permanent, never changes',
    uniqueness: 'Globally unique across all players'
  },

  customCode: {
    enabled: true,
    requirement: 'Account level 20+',
    cost: 500,
    currency: 'doubloons',
    rules: {
      length: '6-12 characters',
      characters: 'Alphanumeric only',
      profanity: 'Filtered',
      reserved: 'Cannot use official/partner codes'
    },
    changes: 'One free change, then 500 Doubloons each'
  }
};
```

### Referral Links

```typescript
const REFERRAL_LINKS = {
  format: {
    url: 'https://plunderstorm.game/invite/{code}',
    shortened: 'https://plndr.gg/{code}',
    qrCode: 'Generated for each player'
  },

  deepLink: {
    installed: 'Opens app directly to referral flow',
    notInstalled: 'Opens landing page with app store links'
  },

  tracking: {
    source: 'Track where link was shared',
    clicks: 'Count link clicks',
    conversions: 'Track successful signups'
  }
};
```

---

## Sharing Mechanisms

### In-App Sharing

```typescript
const SHARING_METHODS = {
  shareButton: {
    location: 'Profile > Invite Friends',
    prominent: 'Also shown on main menu periodically'
  },

  shareSheet: {
    ios: 'Native UIActivityViewController',
    android: 'Native Intent.ACTION_SEND',
    content: {
      text: 'Join me in Plunderstorm! Use my code {CODE} for bonus rewards! {LINK}',
      image: 'Optional: Referral card image'
    }
  },

  directInvite: {
    contacts: 'Optional contacts access for direct invite',
    inApp: 'Send invite to in-game friends list',
    recent: 'Suggest recent contacts'
  },

  platforms: {
    supported: [
      'Messages/SMS',
      'WhatsApp',
      'Facebook Messenger',
      'Discord',
      'Email',
      'Copy Link',
      'QR Code'
    ]
  }
};
```

### Share Card

```typescript
const SHARE_CARD = {
  design: {
    dimensions: '1200x630 (social optimized)',
    content: [
      'Game logo and branding',
      'Referral code prominently displayed',
      'Reward preview',
      'Call to action'
    ]
  },

  personalization: {
    playerName: 'Optional: include inviter name',
    avatar: 'Optional: include inviter character',
    stats: 'Optional: "Join {name} and 1M+ pirates!"'
  }
};
```

---

## Referee Flow

### New Player Experience

```typescript
const REFEREE_FLOW = {
  codeEntry: {
    timing: [
      'During account creation',
      'First 7 days after signup',
      'In settings menu'
    ],
    ui: {
      prompt: 'Have a referral code?',
      input: 'Enter code field',
      validation: 'Real-time code validation'
    }
  },

  deepLinkFlow: {
    click: 'User clicks referral link',
    notInstalled: {
      landing: 'Show referral landing page',
      appStore: 'Redirect to appropriate store',
      tracking: 'Store referral code for attribution'
    },
    installed: {
      open: 'Open app with referral context',
      prompt: 'Show referral acceptance dialog',
      apply: 'Auto-apply referral code'
    }
  },

  confirmation: {
    display: 'You were referred by {PlayerName}!',
    rewards: 'Preview rewards you\'ll earn',
    action: 'Accept referral / Enter different code'
  }
};
```

### Qualification Requirements

```typescript
const QUALIFICATION = {
  referee: {
    requirements: [
      { type: 'newAccount', description: 'Must be new player (account < 7 days)' },
      { type: 'tutorial', description: 'Complete tutorial' },
      { type: 'level', description: 'Reach account level 5' },
      { type: 'matches', description: 'Play 10 matches' }
    ],
    timing: 'Must complete within 14 days of signup',
    verification: 'Server-side validation'
  },

  progress: {
    display: 'Progress bar in referral section',
    milestones: 'Show each requirement status',
    notification: 'Notify when rewards unlocked'
  }
};
```

---

## Reward Structure

### Base Rewards

```typescript
const BASE_REWARDS = {
  referrer: {
    perReferral: {
      gold: 500,
      doubloons: 50,
      chest: 'Rare Chest',
      description: 'Earned when referee qualifies'
    },
    delivery: 'Instant on referee qualification'
  },

  referee: {
    onQualification: {
      gold: 1000,
      doubloons: 100,
      chest: 'Epic Chest',
      booster: '24-hour XP boost',
      description: 'Welcome bonus for joining via referral'
    },
    delivery: 'Instant on qualification'
  },

  both: {
    friendBonus: {
      description: 'Added as friends automatically',
      bonus: '+10% XP when playing together for 7 days'
    }
  }
};
```

### Milestone Rewards

```typescript
const MILESTONE_REWARDS = {
  referrer: {
    milestones: [
      {
        count: 3,
        reward: {
          title: 'Social Butterfly',
          gold: 1000,
          badge: 'Recruiter Badge'
        }
      },
      {
        count: 5,
        reward: {
          gold: 2000,
          doubloons: 100,
          chest: 'Epic Chest'
        }
      },
      {
        count: 10,
        reward: {
          title: 'Fleet Admiral',
          gold: 5000,
          doubloons: 250,
          outfit: 'Recruiter Outfit (Rare)'
        }
      },
      {
        count: 25,
        reward: {
          gold: 10000,
          doubloons: 500,
          outfit: 'Fleet Commander Outfit (Epic)',
          badge: 'Master Recruiter Badge'
        }
      },
      {
        count: 50,
        reward: {
          title: 'Legendary Recruiter',
          gold: 25000,
          doubloons: 1000,
          outfit: 'Admiral\'s Regalia (Legendary)',
          effect: 'Exclusive referral trail'
        }
      },
      {
        count: 100,
        reward: {
          title: 'Pirate Legend',
          gold: 50000,
          doubloons: 2500,
          frame: 'Legendary Recruiter Frame (Animated)',
          recognition: 'Hall of Fame inclusion'
        }
      }
    ]
  }
};
```

### Lifetime Value Bonus

```typescript
const LTV_BONUS = {
  description: 'Bonus rewards when referred players make purchases',
  enabled: true,

  structure: {
    firstPurchase: {
      trigger: 'Referee makes first purchase',
      referrerReward: {
        doubloons: 100,
        description: 'Thank you for bringing a supporter!'
      }
    },
    spendingMilestones: {
      tier1: {
        threshold: 10.00,
        referrerReward: { doubloons: 50 }
      },
      tier2: {
        threshold: 50.00,
        referrerReward: { doubloons: 150 }
      },
      tier3: {
        threshold: 100.00,
        referrerReward: { doubloons: 300, badge: 'Whale Whisperer' }
      }
    }
  },

  cap: {
    perReferee: '$100 spending tracked',
    reason: 'Prevent exploitation, focus on initial conversion'
  }
};
```

---

## Referral Dashboard

### Referrer View

```typescript
const REFERRER_DASHBOARD = {
  location: 'Profile > Referrals',

  summary: {
    code: 'Your referral code (tap to copy)',
    link: 'Your referral link (tap to copy)',
    shareButton: 'Prominent share button',
    qrCode: 'QR code for in-person sharing'
  },

  stats: {
    totalReferred: 'Total qualified referrals',
    pendingReferred: 'Referrals in progress',
    totalEarned: 'Total rewards earned',
    nextMilestone: 'Progress to next milestone'
  },

  referralList: {
    display: 'List of referred players',
    info: ['Player name', 'Status', 'Date', 'Rewards earned'],
    privacy: 'Names partially hidden for privacy'
  },

  milestones: {
    display: 'Milestone progress tracker',
    claimed: 'Checkmarks for claimed milestones',
    next: 'Highlighted next milestone'
  }
};
```

### Referee View

```typescript
const REFEREE_DASHBOARD = {
  location: 'Profile > Referrals',

  status: {
    referredBy: 'Who referred you (if applicable)',
    progress: 'Qualification progress',
    rewards: 'Rewards earned/pending'
  },

  enterCode: {
    visible: 'If no referral code used yet',
    input: 'Enter referral code',
    deadline: 'X days remaining to enter code'
  }
};
```

---

## Fraud Prevention

### Detection Methods

```typescript
const FRAUD_DETECTION = {
  deviceFingerprinting: {
    method: 'Track device identifiers',
    detection: 'Multiple accounts same device',
    action: 'Flag for review, delay rewards'
  },

  ipTracking: {
    method: 'Track IP addresses',
    detection: 'Multiple signups same IP',
    threshold: '3+ accounts same IP = flag',
    action: 'Manual review required'
  },

  behaviorAnalysis: {
    patterns: [
      'Rapid account creation',
      'Minimal gameplay on referred accounts',
      'Same play patterns across accounts',
      'Immediate referral code entry'
    ],
    action: 'ML-based flagging for review'
  },

  paymentVerification: {
    method: 'Link payment methods',
    detection: 'Same payment across accounts',
    action: 'Immediate flag and review'
  }
};
```

### Prevention Measures

```typescript
const FRAUD_PREVENTION = {
  limits: {
    dailyReferrals: 10,
    weeklyReferrals: 25,
    description: 'Max new referral signups per period'
  },

  cooldown: {
    afterFlag: '48-hour reward hold on flagged accounts',
    review: 'Manual review within 48 hours'
  },

  verification: {
    levelRequired: 'Referee must reach level 5',
    matchesRequired: 'Referee must play 10 real matches',
    timeRequired: '14-day window to qualify',
    antiBot: 'CAPTCHA on suspicious signups'
  },

  penalties: {
    warning: 'First offense: warning + reward clawback',
    suspension: 'Second offense: referral program suspension',
    ban: 'Severe abuse: account ban'
  }
};
```

### Clawback Policy

```typescript
const CLAWBACK_POLICY = {
  triggers: [
    'Referred account banned within 30 days',
    'Fraud detected on referrer or referee',
    'Referee requests refund on purchase'
  ],

  process: {
    notification: 'Inform player of clawback reason',
    deduction: 'Remove rewards from inventory/balance',
    negative: 'Balance can go negative (must earn back)',
    appeal: 'Appeal process available'
  }
};
```

---

## Analytics and Tracking

### Metrics

```typescript
const REFERRAL_METRICS = {
  acquisition: {
    totalReferrals: 'Total qualified referrals',
    conversionRate: 'Clicks to qualified referrals',
    shareRate: 'Players who share referral link',
    topReferrers: 'Leaderboard of top referrers'
  },

  quality: {
    referredRetention: 'D1/D7/D30 retention of referred',
    referredLtv: 'LTV of referred vs organic',
    referredEngagement: 'Session metrics of referred'
  },

  program: {
    rewardsCost: 'Total rewards distributed',
    cac: 'Cost per acquired referred player',
    roi: 'ROI vs paid acquisition'
  },

  fraud: {
    flaggedRate: 'Percentage flagged for review',
    confirmedFraud: 'Confirmed fraud cases',
    clawbackAmount: 'Value of clawed back rewards'
  }
};
```

### Attribution

```typescript
const ATTRIBUTION = {
  methods: {
    codeEntry: 'Direct code entry attribution',
    deepLink: 'Deep link click attribution',
    deferred: 'Deferred deep link (click before install)'
  },

  window: {
    click: '7 days from click to install',
    code: '7 days from signup to code entry'
  },

  conflict: {
    priority: 'Code entry > Deep link > Deferred',
    lastTouch: 'Most recent valid attribution wins'
  }
};
```

---

## Communication

### Referrer Notifications

```typescript
const REFERRER_NOTIFICATIONS = {
  referralProgress: {
    signup: '{Name} signed up with your code!',
    progress: '{Name} is making progress (Level 3/5)',
    qualified: '{Name} qualified! You earned rewards!'
  },

  milestones: {
    approaching: 'You\'re 1 referral away from a milestone!',
    reached: 'Milestone reached! Claim your rewards!'
  },

  reminders: {
    periodic: 'Invite friends and earn rewards!',
    afterShare: 'Your link was shared! Keep spreading the word.'
  }
};
```

### Referee Notifications

```typescript
const REFEREE_NOTIFICATIONS = {
  welcome: 'Welcome! You were referred by {Name}',
  progress: 'You\'re close to unlocking referral rewards!',
  qualified: 'Congratulations! You\'ve earned your referral bonus!',
  reminder: 'Complete your referral progress for bonus rewards!'
};
```

---

## Special Campaigns

### Double Referral Events

```typescript
const REFERRAL_EVENTS = {
  doubleRewards: {
    frequency: 'Monthly or during major updates',
    duration: '1 week',
    bonus: '2x rewards for both referrer and referee',
    communication: 'Prominent in-game banner'
  },

  competitionWeek: {
    frequency: 'Quarterly',
    duration: '1 week',
    mechanic: 'Top referrers win bonus prizes',
    prizes: {
      top1: 'Exclusive outfit + 5000 Doubloons',
      top10: '2000 Doubloons',
      top100: '500 Doubloons'
    }
  }
};
```

---

## Technical Implementation

### Data Model

```typescript
const DATA_MODEL = {
  referralCode: {
    playerId: 'string',
    code: 'string (unique)',
    customCode: 'string (nullable)',
    createdAt: 'timestamp',
    totalReferrals: 'number',
    totalQualified: 'number'
  },

  referral: {
    id: 'string',
    referrerId: 'string',
    refereeId: 'string',
    code: 'string',
    source: 'enum (code, deeplink, deferred)',
    status: 'enum (pending, qualified, fraudulent, clawedback)',
    createdAt: 'timestamp',
    qualifiedAt: 'timestamp (nullable)',
    rewardsGranted: 'boolean'
  },

  referralProgress: {
    refereeId: 'string',
    tutorialComplete: 'boolean',
    levelReached: 'number',
    matchesPlayed: 'number',
    qualificationDeadline: 'timestamp'
  },

  referralReward: {
    id: 'string',
    playerId: 'string',
    referralId: 'string',
    type: 'enum (base, milestone, ltv)',
    rewards: 'object',
    grantedAt: 'timestamp',
    clawedBack: 'boolean'
  }
};
```

### API Endpoints

```typescript
const API_ENDPOINTS = {
  referral: {
    getCode: 'GET /api/v1/players/{id}/referral-code',
    applyCode: 'POST /api/v1/referrals/apply',
    getStats: 'GET /api/v1/players/{id}/referral-stats',
    getReferrals: 'GET /api/v1/players/{id}/referrals',
    getMilestones: 'GET /api/v1/players/{id}/referral-milestones'
  },
  admin: {
    flagReferral: 'POST /api/v1/admin/referrals/{id}/flag',
    reviewReferral: 'POST /api/v1/admin/referrals/{id}/review',
    clawback: 'POST /api/v1/admin/referrals/{id}/clawback'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Code generation unique and valid',
    'Code entry works at all valid points',
    'Deep links attribute correctly',
    'Rewards granted on qualification',
    'Milestones tracked and rewarded'
  ],

  fraud: [
    'Device fingerprinting detects duplicates',
    'IP tracking flags suspicious patterns',
    'Rate limits enforced',
    'Clawback process functions correctly'
  ],

  attribution: [
    'Click tracking works',
    'Deferred deep links attribute post-install',
    'Conflict resolution follows priority'
  ],

  edge: [
    'Code entered after 7-day window',
    'Referee never qualifies',
    'Referrer account deleted',
    'Multiple code entries attempted'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial referral program specification |
