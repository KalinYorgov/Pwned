# Push Notifications Specification

## Document Information
- **Task ID:** LIVE-012
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the push notification system for Plunderstorm Mobile, defining notification types, delivery strategy, personalization, and user controls to drive player re-engagement while respecting user preferences.

---

## SDK Integration

### Platform Setup

```typescript
const SDK_INTEGRATION = {
  primary: {
    provider: 'Firebase Cloud Messaging (FCM)',
    platforms: ['iOS', 'Android'],
    features: [
      'Cross-platform delivery',
      'Topic subscriptions',
      'Rich notifications',
      'Analytics integration'
    ]
  },

  secondary: {
    provider: 'OneSignal',
    purpose: 'Advanced segmentation and A/B testing',
    features: [
      'Advanced scheduling',
      'User segmentation',
      'A/B testing built-in',
      'Delivery optimization'
    ]
  },

  ios: {
    requirements: [
      'APNs certificate setup',
      'Provisional authorization (iOS 12+)',
      'Critical alerts capability (if approved)',
      'Notification Service Extension for rich media'
    ],
    permissions: 'Request on first meaningful moment, not app launch'
  },

  android: {
    requirements: [
      'FCM configuration',
      'Notification channels setup',
      'Foreground service for time-sensitive'
    ],
    channels: 'Separate channels for different notification types'
  }
};
```

### Initialization

```typescript
const INITIALIZATION = {
  timing: {
    sdk: 'Initialize on app launch',
    token: 'Register token after user login',
    permissions: 'Request after tutorial completion or first win'
  },

  tokenManagement: {
    storage: 'Store token on backend per user per device',
    refresh: 'Update on token refresh callback',
    multiDevice: 'Support multiple devices per user',
    cleanup: 'Remove stale tokens after 30 days no activity'
  },

  errorHandling: {
    permissionDenied: 'Track in analytics, no retry spam',
    tokenFailure: 'Retry with exponential backoff',
    deliveryFailure: 'Log for monitoring, adjust strategy'
  }
};
```

---

## Notification Types

### Gameplay Notifications

```typescript
const GAMEPLAY_NOTIFICATIONS = {
  dailyQuestAvailable: {
    id: 'daily_quest_available',
    trigger: 'Daily reset time',
    title: 'New Daily Quests!',
    body: 'Your daily quests have reset. Complete them for rewards!',
    timing: 'At reset time (configurable per user timezone)',
    frequency: 'Daily',
    priority: 'Normal',
    category: 'gameplay'
  },

  weeklyQuestReminder: {
    id: 'weekly_quest_reminder',
    trigger: '24 hours before weekly reset',
    title: 'Weekly Quests Expiring Soon!',
    body: 'You have {count} incomplete weekly quests. Finish them before reset!',
    timing: 'Sunday evening (user timezone)',
    frequency: 'Weekly (if incomplete quests)',
    priority: 'Normal',
    category: 'gameplay'
  },

  firstWinBonus: {
    id: 'first_win_bonus',
    trigger: 'Daily reset if player engaged yesterday',
    title: 'First Win Bonus Ready!',
    body: 'Your First Win bonus has reset. Win a match for bonus rewards!',
    timing: 'Morning (user timezone)',
    frequency: 'Daily (for active players)',
    priority: 'Low',
    category: 'gameplay'
  },

  battlePassProgress: {
    id: 'battle_pass_progress',
    trigger: 'Close to next tier',
    title: 'Almost There!',
    body: 'You\'re {xp} XP away from your next Battle Pass reward!',
    timing: 'Evening (user timezone)',
    frequency: 'Max once per 3 days',
    priority: 'Low',
    category: 'gameplay'
  }
};
```

### Event Notifications

```typescript
const EVENT_NOTIFICATIONS = {
  eventStarting: {
    id: 'event_starting',
    trigger: 'Event start time',
    title: '{eventName} Starts Now!',
    body: 'A new event has begun! Jump in for exclusive rewards.',
    timing: 'At event start',
    frequency: 'Per event (max 2/week)',
    priority: 'High',
    category: 'events',
    richMedia: 'Event banner image'
  },

  eventEndingSoon: {
    id: 'event_ending_soon',
    trigger: '24-48 hours before event end',
    title: '{eventName} Ends Soon!',
    body: 'Only {hours} hours left! Don\'t miss out on event rewards.',
    timing: 'When threshold reached',
    frequency: 'Once per event',
    priority: 'Normal',
    category: 'events'
  },

  ltmAvailable: {
    id: 'ltm_available',
    trigger: 'New LTM rotation',
    title: '{modeName} is Live!',
    body: 'A new limited-time mode is available. Play now!',
    timing: 'At LTM start',
    frequency: '2-3 per week',
    priority: 'Normal',
    category: 'events'
  },

  communityChallengeMilestone: {
    id: 'community_milestone',
    trigger: 'Major milestone reached',
    title: 'Milestone Reached!',
    body: 'The community hit {milestone}! Claim your reward.',
    timing: 'On milestone',
    frequency: 'Per milestone',
    priority: 'Normal',
    category: 'events'
  }
};
```

### Social Notifications

```typescript
const SOCIAL_NOTIFICATIONS = {
  friendsOnline: {
    id: 'friends_online',
    trigger: 'Friends come online',
    title: '{friendName} is online!',
    body: 'Your friend is ready to play. Party up?',
    timing: 'When friend logs in',
    frequency: 'Max 3 per day',
    priority: 'Low',
    category: 'social',
    conditions: {
      playerOffline: 'Only if player not in game',
      closeFriends: 'Only for frequently played-with friends',
      cooldown: '4 hours per friend'
    }
  },

  partyInvite: {
    id: 'party_invite',
    trigger: 'Friend sends party invite',
    title: '{friendName} invited you to party!',
    body: 'Join the party and play together.',
    timing: 'Immediate',
    frequency: 'Per invite (max 5/hour)',
    priority: 'High',
    category: 'social'
  },

  giftReceived: {
    id: 'gift_received',
    trigger: 'Gift sent from friend',
    title: 'You received a gift!',
    body: '{friendName} sent you a gift. Open it now!',
    timing: 'Immediate',
    frequency: 'Per gift',
    priority: 'Normal',
    category: 'social'
  }
};
```

### Shop and Offers

```typescript
const SHOP_NOTIFICATIONS = {
  dailyShopRefresh: {
    id: 'daily_shop_refresh',
    trigger: 'Daily shop rotation',
    title: 'New Items in Shop!',
    body: 'Today\'s shop is now available. Check out the new items!',
    timing: 'At shop reset',
    frequency: 'Daily (for engaged players)',
    priority: 'Low',
    category: 'shop'
  },

  specialOffer: {
    id: 'special_offer',
    trigger: 'Limited-time offer available',
    title: 'Special Offer!',
    body: '{discountPercent}% off {itemName}. Limited time only!',
    timing: 'At offer activation',
    frequency: 'Max 2 per week',
    priority: 'Normal',
    category: 'shop'
  },

  battlePassExpiring: {
    id: 'battle_pass_expiring',
    trigger: '7 days before Battle Pass ends',
    title: 'Battle Pass Ends Soon!',
    body: 'Only {days} days left to complete your Battle Pass. You\'re at tier {tier}.',
    timing: 'When threshold reached',
    frequency: 'Days 7, 3, 1',
    priority: 'Normal',
    category: 'shop'
  },

  itemReturning: {
    id: 'item_returning',
    trigger: 'Wishlisted item returns to shop',
    title: '{itemName} is Back!',
    body: 'An item from your wishlist is now available.',
    timing: 'At item availability',
    frequency: 'Per item',
    priority: 'Normal',
    category: 'shop'
  }
};
```

### Win-Back Notifications

```typescript
const WINBACK_NOTIFICATIONS = {
  day3Reminder: {
    id: 'winback_day3',
    trigger: '3 days since last login',
    title: 'Your crew misses you!',
    body: 'Come back and claim your daily rewards.',
    timing: 'Evening (user timezone)',
    priority: 'Normal',
    category: 'winback'
  },

  day7Reward: {
    id: 'winback_day7',
    trigger: '7 days since last login',
    title: 'Welcome Back Rewards Await!',
    body: 'Special rewards are waiting for you. Return to claim them!',
    timing: 'Evening (user timezone)',
    priority: 'Normal',
    category: 'winback'
  },

  day14Event: {
    id: 'winback_day14',
    trigger: '14 days since last login (during event)',
    title: 'Don\'t Miss {eventName}!',
    body: 'A special event is happening. Return for exclusive rewards!',
    timing: 'When event is active',
    priority: 'Normal',
    category: 'winback'
  },

  day30LastChance: {
    id: 'winback_day30',
    trigger: '30 days since last login',
    title: 'Legendary Loot Awaits!',
    body: 'Your welcome back rewards have grown. Return for legendary loot!',
    timing: 'Evening (user timezone)',
    priority: 'Normal',
    category: 'winback'
  }
};
```

---

## Personalization

### Dynamic Tokens

```typescript
const PERSONALIZATION_TOKENS = {
  player: {
    '{playerName}': 'Player display name',
    '{level}': 'Player level',
    '{rank}': 'Ranked tier name'
  },

  progress: {
    '{tier}': 'Current Battle Pass tier',
    '{xp}': 'XP to next tier/level',
    '{count}': 'Number of incomplete quests',
    '{percentage}': 'Completion percentage'
  },

  time: {
    '{hours}': 'Hours remaining',
    '{days}': 'Days remaining',
    '{time}': 'Specific time'
  },

  social: {
    '{friendName}': 'Friend display name',
    '{partySize}': 'Current party size'
  },

  content: {
    '{eventName}': 'Event name',
    '{modeName}': 'LTM name',
    '{itemName}': 'Shop item name',
    '{discountPercent}': 'Discount percentage'
  }
};
```

### Behavioral Targeting

```typescript
const BEHAVIORAL_TARGETING = {
  segments: {
    engaged: {
      criteria: 'Played in last 3 days',
      notifications: ['Gameplay updates', 'Events', 'Social'],
      frequency: 'Standard'
    },
    casual: {
      criteria: 'Played 1-2 times per week',
      notifications: ['Major events only', 'Social'],
      frequency: 'Reduced'
    },
    lapsed: {
      criteria: 'Not played in 7+ days',
      notifications: ['Win-back series'],
      frequency: 'Limited (prevent fatigue)'
    },
    competitive: {
      criteria: 'Plays ranked regularly',
      notifications: ['Ranked updates', 'Leaderboard changes'],
      frequency: 'Standard'
    },
    spender: {
      criteria: 'Has made purchases',
      notifications: ['Exclusive offers', 'Sales'],
      frequency: 'Standard'
    }
  },

  personalization: {
    favoriteMode: 'Highlight when favorite mode/LTM available',
    playTime: 'Send at player\'s typical play time',
    timezone: 'Respect local timezone'
  }
};
```

---

## User Controls

### Permission Flow

```typescript
const PERMISSION_FLOW = {
  initialPrompt: {
    timing: 'After tutorial OR after first win',
    priming: 'Show value proposition before system prompt',
    message: 'Get notified about events, rewards, and when friends are online!',
    options: ['Enable Notifications', 'Maybe Later']
  },

  systemPrompt: {
    ios: 'After priming, trigger iOS permission dialog',
    android: 'Android 13+ requires runtime permission'
  },

  deferral: {
    maybelater: 'Prompt again after 3 days or major event',
    maxRetries: 3,
    finalDeferral: 'Show in settings only'
  },

  postDenial: {
    detection: 'Track if user denied permissions',
    recovery: 'Periodically remind via in-game banner',
    settings: 'Deep link to settings to enable'
  }
};
```

### Settings Panel

```typescript
const NOTIFICATION_SETTINGS = {
  location: 'Settings > Notifications',

  masterToggle: {
    name: 'Push Notifications',
    description: 'Receive notifications from Plunderstorm',
    default: true
  },

  categories: {
    gameplay: {
      name: 'Gameplay Updates',
      description: 'Daily quests, Battle Pass progress, rewards',
      default: true,
      includes: ['daily_quest_available', 'battle_pass_progress', 'first_win_bonus']
    },
    events: {
      name: 'Events & LTMs',
      description: 'New events, limited-time modes, community challenges',
      default: true,
      includes: ['event_starting', 'event_ending_soon', 'ltm_available']
    },
    social: {
      name: 'Social',
      description: 'Friend activity, party invites, gifts',
      default: true,
      includes: ['friends_online', 'party_invite', 'gift_received']
    },
    shop: {
      name: 'Shop & Offers',
      description: 'New items, special offers, sales',
      default: true,
      includes: ['daily_shop_refresh', 'special_offer', 'item_returning']
    },
    winback: {
      name: 'Come Back Reminders',
      description: 'Reminders if you haven\'t played in a while',
      default: true,
      hidden: false,
      includes: ['winback_day3', 'winback_day7', 'winback_day14', 'winback_day30']
    }
  },

  quietHours: {
    enabled: true,
    default: { start: '22:00', end: '08:00' },
    custom: 'Player can set custom quiet hours',
    override: 'Critical notifications (party invite) can override'
  }
};
```

---

## Scheduling and Delivery

### Timing Optimization

```typescript
const TIMING_OPTIMIZATION = {
  userTimezone: {
    detection: 'Infer from device or explicit setting',
    fallback: 'UTC',
    storage: 'Store per user'
  },

  optimalTiming: {
    method: 'Send Time Optimization (STO)',
    logic: 'Analyze when user typically engages',
    window: 'Deliver within user\'s active window'
  },

  quietHours: {
    respect: 'Queue notifications during quiet hours',
    delivery: 'Deliver at quiet hours end',
    exception: 'Time-sensitive (party invite) can override'
  },

  batching: {
    enabled: true,
    logic: 'Batch multiple low-priority notifications',
    maxDelay: '30 minutes',
    summary: 'You have {count} updates'
  }
};
```

### Frequency Capping

```typescript
const FREQUENCY_CAPPING = {
  global: {
    daily: 5,
    weekly: 15,
    description: 'Maximum notifications per player per period'
  },

  perCategory: {
    gameplay: { daily: 2 },
    events: { daily: 2 },
    social: { daily: 3 },
    shop: { daily: 1 },
    winback: { daily: 1 }
  },

  cooldown: {
    afterSend: '2 hours minimum between notifications',
    afterOpen: '4 hours after player opens app',
    afterDismiss: 'No immediate follow-up'
  },

  priority: {
    high: 'Can exceed caps slightly',
    normal: 'Subject to caps',
    low: 'First to be dropped when at cap'
  }
};
```

---

## A/B Testing

### Testing Framework

```typescript
const NOTIFICATION_TESTING = {
  testableElements: {
    title: 'Different title variations',
    body: 'Different body copy',
    timing: 'Different send times',
    frequency: 'Different notification frequency',
    richMedia: 'With/without images'
  },

  experiments: {
    copy: {
      example: {
        control: 'New Daily Quests!',
        variant: 'Ready to earn rewards?'
      },
      metric: 'Open rate'
    },
    timing: {
      example: {
        control: 'At reset time',
        variant: 'During typical play window'
      },
      metric: 'Open rate + session'
    }
  },

  analysis: {
    sampleSize: 'Minimum 1000 per variant',
    significance: '95% confidence',
    duration: 'Minimum 7 days',
    metrics: ['Open rate', 'Session start', 'Conversion']
  }
};
```

---

## Analytics

### Metrics

```typescript
const NOTIFICATION_ANALYTICS = {
  delivery: {
    sent: 'Total notifications sent',
    delivered: 'Successfully delivered',
    failed: 'Failed delivery (token issues, etc.)',
    deliveryRate: 'Delivered / Sent'
  },

  engagement: {
    opened: 'Notifications opened',
    openRate: 'Opened / Delivered',
    dismissed: 'Swiped away without opening',
    sessionAfterOpen: 'Sessions started from notification'
  },

  conversion: {
    questCompleted: 'Completed quest after notification',
    purchaseMade: 'Purchase after shop notification',
    eventParticipation: 'Joined event after notification'
  },

  optOut: {
    categoryDisabled: 'Users who disabled category',
    fullOptOut: 'Users who disabled all notifications',
    permissionDenied: 'Users who denied system permission'
  }
};
```

### Dashboards

```typescript
const NOTIFICATION_DASHBOARDS = {
  overview: {
    metrics: [
      'Daily send volume',
      'Delivery rate',
      'Open rate',
      'Opt-out rate'
    ],
    trends: 'Week over week comparison'
  },

  perNotification: {
    metrics: [
      'Send count',
      'Open rate',
      'Conversion rate',
      'Revenue attributed'
    ],
    ranking: 'Best and worst performers'
  },

  segments: {
    breakdown: 'Performance by player segment',
    comparison: 'Engaged vs casual vs lapsed'
  },

  alerts: {
    lowDelivery: 'Alert if delivery rate < 90%',
    highOptOut: 'Alert if opt-out spike > 10%',
    failedCampaign: 'Alert on campaign failure'
  }
};
```

---

## Rich Notifications

### Media Support

```typescript
const RICH_NOTIFICATIONS = {
  images: {
    support: 'iOS and Android',
    size: 'Max 1MB, 1024x512 recommended',
    format: 'PNG or JPEG',
    usage: 'Event banners, new items, achievements'
  },

  actions: {
    ios: 'Up to 4 action buttons',
    android: 'Up to 3 action buttons',
    examples: [
      { label: 'Play Now', action: 'open_game' },
      { label: 'View Event', action: 'open_event' },
      { label: 'Dismiss', action: 'dismiss' }
    ]
  },

  deepLinks: {
    support: 'Deep link to specific screens',
    examples: [
      { action: 'open_quest', destination: '/quests' },
      { action: 'open_event', destination: '/events/{eventId}' },
      { action: 'open_shop', destination: '/shop' },
      { action: 'open_friends', destination: '/social/friends' }
    ]
  },

  badges: {
    ios: 'Update app badge count',
    android: 'Notification dot on app icon',
    logic: 'Increment on new notification, clear on app open'
  }
};
```

---

## Technical Implementation

### Backend Architecture

```typescript
const BACKEND_ARCHITECTURE = {
  notificationService: {
    role: 'Central notification orchestration',
    responsibilities: [
      'Receive notification requests',
      'Apply personalization',
      'Apply frequency caps',
      'Route to delivery provider'
    ]
  },

  scheduler: {
    role: 'Scheduled notification management',
    responsibilities: [
      'Queue scheduled notifications',
      'Trigger at appropriate time',
      'Handle timezone conversion'
    ]
  },

  segmentation: {
    role: 'User targeting',
    responsibilities: [
      'Evaluate user segments',
      'Apply behavioral rules',
      'Select notification variant'
    ]
  },

  delivery: {
    role: 'Send to FCM/APNs',
    responsibilities: [
      'Format for platform',
      'Handle rate limits',
      'Process delivery receipts'
    ]
  }
};
```

### Data Model

```typescript
const DATA_MODEL = {
  notificationToken: {
    userId: 'string',
    deviceId: 'string',
    platform: 'enum (ios, android)',
    token: 'string',
    createdAt: 'timestamp',
    lastActiveAt: 'timestamp'
  },

  notificationPreferences: {
    userId: 'string',
    enabled: 'boolean',
    categories: 'object<category, boolean>',
    quietHoursStart: 'string (HH:mm)',
    quietHoursEnd: 'string (HH:mm)',
    timezone: 'string'
  },

  notificationLog: {
    id: 'string',
    userId: 'string',
    notificationType: 'string',
    title: 'string',
    body: 'string',
    sentAt: 'timestamp',
    deliveredAt: 'timestamp (nullable)',
    openedAt: 'timestamp (nullable)',
    dismissed: 'boolean',
    variant: 'string (nullable)'
  }
};
```

---

## Compliance

### Regulations

```typescript
const COMPLIANCE = {
  gdpr: {
    consent: 'Explicit opt-in for marketing notifications',
    dataAccess: 'User can request notification history',
    deletion: 'Remove all data on account deletion',
    crossBorder: 'Handle EU data properly'
  },

  tcpa: {
    consent: 'Clear consent for marketing messages',
    optOut: 'Easy and immediate opt-out',
    documentation: 'Log consent for compliance'
  },

  appStore: {
    ios: 'Follow Apple notification guidelines',
    android: 'Follow Google Play policies',
    content: 'No misleading notifications'
  },

  bestPractices: {
    value: 'Only send valuable notifications',
    frequency: 'Respect user attention',
    transparency: 'Clear what each category means',
    control: 'Easy to manage preferences'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'All notification types deliver correctly',
    'Deep links navigate to correct screens',
    'Rich media displays properly',
    'Actions work as expected'
  ],

  preferences: [
    'Master toggle works',
    'Category toggles work',
    'Quiet hours respected',
    'Settings persist across sessions'
  ],

  targeting: [
    'Segments receive correct notifications',
    'Personalization tokens replaced',
    'Frequency caps enforced',
    'Timing optimization works'
  ],

  platform: [
    'iOS notifications render correctly',
    'Android notifications render correctly',
    'Badge counts update properly',
    'Background/foreground handling'
  ],

  edge: [
    'Token refresh handled',
    'Offline device handling',
    'App uninstall/reinstall',
    'Multiple device sync'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial push notifications specification |
