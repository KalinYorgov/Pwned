# Rewarded Video Ads Specification

## Document Information
- **Task ID:** MON-016
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the rewarded video ad system for Plunderstorm Mobile, providing opt-in ad viewing opportunities for players in exchange for in-game rewards. The system balances revenue generation with player experience.

---

## Ad SDK Integration

### Provider Selection

```typescript
const AD_SDK_INTEGRATION = {
  primary: {
    provider: 'IronSource',
    rationale: [
      'Strong mediation capabilities',
      'High fill rates globally',
      'Good revenue optimization',
      'Reliable SDK for mobile'
    ]
  },

  adNetworks: {
    integrated: [
      'Unity Ads',
      'AdMob',
      'AppLovin',
      'Vungle',
      'Facebook Audience Network',
      'Chartboost'
    ],
    purpose: 'Mediation for best eCPM and fill rates'
  },

  initialization: {
    timing: 'On app start (after consent)',
    async: true, // Non-blocking
    fallback: 'Proceed without ads if init fails'
  },

  gdprConsent: {
    required: true,
    flow: 'Show consent dialog before ad init',
    tracking: 'Respect ATT status on iOS'
  }
};
```

### SDK Configuration

```typescript
const SDK_CONFIG = {
  unitIds: {
    rewarded: {
      ios: 'ios_rewarded_unit_id',
      android: 'android_rewarded_unit_id'
    }
  },

  preloading: {
    enabled: true,
    timing: 'After SDK init and on ad completion',
    maxRetries: 3,
    retryDelay: 5000 // ms
  },

  testMode: {
    development: true,
    production: false,
    testDeviceIds: ['device_id_1', 'device_id_2']
  },

  callbacks: {
    onAdLoaded: 'Ready to show ad',
    onAdOpened: 'Ad displayed fullscreen',
    onAdClosed: 'Ad dismissed by user',
    onAdRewarded: 'User earned reward',
    onAdFailed: 'Ad failed to load/show'
  }
};
```

---

## Ad Placements

### Placement Locations

```typescript
const AD_PLACEMENTS = {
  postMatch: {
    id: 'post_match_gold_boost',
    location: 'Match summary screen',
    trigger: 'After match completion',
    reward: {
      type: 'gold_multiplier',
      value: 1.5, // 50% bonus gold
      description: '+50% Gold for this match'
    },
    ui: {
      button: 'Watch Ad for +50% Gold',
      icon: 'video_ad_icon',
      position: 'Below gold display',
      highlight: true
    },
    eligibility: {
      earnedGold: '> 0', // Must have earned some gold
      matchType: ['solo', 'duo', 'squad'],
      excludeCustom: true
    }
  },

  dailyLogin: {
    id: 'daily_login_double',
    location: 'Daily login reward popup',
    trigger: 'When claiming daily reward',
    reward: {
      type: 'reward_multiplier',
      value: 2.0, // Double reward
      description: 'Double your daily reward!'
    },
    ui: {
      button: 'Watch Ad to Double',
      alternative: 'Claim (Tap to skip)',
      countdown: 'Show ad button after 2s'
    },
    limit: {
      perDay: 1,
      description: 'Once per daily login'
    }
  },

  freeLootCrate: {
    id: 'free_loot_crate',
    location: 'Shop > Free section',
    trigger: 'Tap free crate button',
    reward: {
      type: 'loot_crate',
      tier: 'common',
      description: 'Free Common Crate'
    },
    ui: {
      button: 'Watch Ad for Free Crate',
      availability: 'Shows timer when unavailable'
    },
    limit: {
      perDay: 3,
      cooldown: 14400 // 4 hours between
    }
  },

  bonusCurrency: {
    id: 'bonus_currency',
    location: 'Shop > Currency section',
    trigger: 'Tap bonus currency button',
    reward: {
      type: 'soft_currency',
      amount: 50, // 50 gold
      description: '50 Gold'
    },
    ui: {
      button: 'Watch Ad for 50 Gold',
      icon: 'gold_coin'
    },
    limit: {
      perDay: 5,
      cooldown: 3600 // 1 hour between
    }
  },

  reviveOption: {
    id: 'revive_ad',
    location: 'Elimination screen (casual only)',
    trigger: 'On player elimination',
    reward: {
      type: 'revive',
      description: 'Return to battle with 50% HP'
    },
    ui: {
      button: 'Watch Ad to Revive',
      timer: '10 seconds to decide',
      alternative: 'Spectate or Leave'
    },
    eligibility: {
      mode: 'casual_only', // Not in ranked
      oncePerMatch: true,
      placement: '> 30' // Not in top 30
    }
  }
};
```

### Placement Strategy

```typescript
const PLACEMENT_STRATEGY = {
  principles: {
    optIn: 'All ads are player-initiated',
    valueExchange: 'Clear reward shown before viewing',
    neverIntrusve: 'No forced ads, no popup ads',
    respectGameplay: 'No ads during active gameplay'
  },

  timing: {
    postMatch: 'Natural break point',
    dailyLogin: 'Low commitment moment',
    shop: 'Player already in spending mindset',
    revive: 'High motivation moment'
  },

  visibility: {
    obvious: 'Ad buttons clearly visible',
    notDeceptive: 'Clear that it is an ad',
    rewardPreview: 'Exact reward shown upfront'
  }
};
```

---

## Reward System

### Reward Delivery

```typescript
const REWARD_DELIVERY = {
  flow: [
    'Player taps ad button',
    'Show loading indicator',
    'Display fullscreen ad',
    'Wait for ad completion callback',
    'Verify completion with server',
    'Grant reward',
    'Show reward confirmation',
    'Update UI with new balance'
  ],

  serverValidation: {
    required: true,
    data: {
      userId: 'Player identifier',
      placementId: 'Which ad placement',
      timestamp: 'When ad was watched',
      adNetworkId: 'Which network served ad'
    },
    antiCheat: 'Server validates timing and frequency'
  },

  rewardGrant: {
    immediate: true, // Grant right after ad completes
    notification: 'Visual + optional haptic',
    animation: 'Reward fly-in animation'
  },

  failureCases: {
    adNotCompleted: {
      reason: 'User skipped or ad errored',
      reward: 'No reward granted',
      message: 'Complete the ad to earn reward'
    },
    adNotAvailable: {
      reason: 'No ad fill',
      fallback: 'Hide ad button or show "Try again later"'
    },
    networkError: {
      retry: 'Retry reward grant 3 times',
      fallback: 'Queue for next session if all fail'
    }
  }
};

const REWARD_TYPES = {
  goldMultiplier: {
    type: 'multiplicative',
    applies: 'Match gold earnings',
    stacking: 'Does not stack with premium multipliers'
  },

  rewardMultiplier: {
    type: 'multiplicative',
    applies: 'Daily login reward only'
  },

  softCurrency: {
    type: 'additive',
    applies: 'Direct gold grant',
    source: 'ad_reward'
  },

  lootCrate: {
    type: 'item_grant',
    applies: 'Grants specific crate',
    source: 'ad_reward'
  },

  revive: {
    type: 'gameplay_effect',
    applies: 'Current match only',
    effect: 'Respawn at 50% HP'
  }
};
```

---

## Daily Limits

### Limit Configuration

```typescript
const AD_LIMITS = {
  global: {
    totalPerDay: 10, // Maximum ads across all placements
    reason: 'Prevent ad fatigue and farming',
    reset: '00:00 UTC'
  },

  perPlacement: {
    post_match_gold_boost: { limit: 5, cooldown: 0 },
    daily_login_double: { limit: 1, cooldown: 0 },
    free_loot_crate: { limit: 3, cooldown: 14400 },
    bonus_currency: { limit: 5, cooldown: 3600 },
    revive_ad: { limit: 2, cooldown: 0 } // Per day, once per match
  },

  enforcement: {
    client: 'Hide/disable button when limit reached',
    server: 'Reject reward if limit exceeded',
    display: 'Show remaining watches or next available time'
  },

  ui: {
    available: 'Colored button with reward preview',
    onCooldown: 'Grayed button with timer',
    limitReached: 'Hidden or "Come back tomorrow"'
  }
};
```

### Cooldown Display

```typescript
const COOLDOWN_DISPLAY = {
  format: {
    underMinute: 'Available in Xs',
    underHour: 'Available in Xm',
    underDay: 'Available in Xh Xm',
    nextDay: 'Available tomorrow'
  },

  countdown: {
    live: true, // Real-time countdown
    updateFrequency: 1000 // ms
  },

  notification: {
    push: false, // No push for ad availability
    inGame: true // Badge/indicator when available
  }
};
```

---

## UI/UX Design

### Ad Button Components

```typescript
const AD_BUTTON_DESIGN = {
  standard: {
    size: 'Large touch target (48dp+ height)',
    icon: 'Video/play icon on left',
    text: 'Clear reward description',
    background: 'Distinct color (gold/green)',
    animation: 'Subtle pulse when available'
  },

  states: {
    available: {
      appearance: 'Fully colored, enabled',
      icon: 'Video play icon',
      text: 'Watch Ad for [Reward]'
    },
    loading: {
      appearance: 'Dimmed with spinner',
      text: 'Loading...',
      interactive: false
    },
    cooldown: {
      appearance: 'Grayed out',
      text: 'Available in [Time]',
      interactive: false
    },
    unavailable: {
      appearance: 'Hidden or "No Ads Available"',
      interactive: false
    }
  },

  rewardPreview: {
    icon: 'Reward item icon',
    amount: 'Quantity if applicable',
    effect: '+50%' or 'x2' for multipliers'
  }
};
```

### Ad Experience Flow

```typescript
const AD_EXPERIENCE = {
  preAd: {
    confirmation: {
      required: false, // Single tap to watch
      exception: 'Revive shows "Watch ad to revive?"'
    },
    loading: {
      indicator: 'Fullscreen loading overlay',
      timeout: 10000, // 10 seconds max
      cancel: 'Allow cancel during load'
    }
  },

  duringAd: {
    audio: {
      gameAudio: 'Muted during ad',
      adAudio: 'Play at system volume'
    },
    display: {
      fullscreen: true,
      orientation: 'Match game orientation'
    },
    interruption: {
      phoneCalls: 'Ad pauses, resumes after',
      appBackground: 'Ad pauses, resumes on return'
    }
  },

  postAd: {
    transition: 'Smooth return to game',
    reward: {
      animation: 'Celebratory reward animation',
      sound: 'Reward earned sound effect',
      duration: 2000 // 2 seconds
    },
    return: 'Return to previous screen'
  }
};
```

---

## Error Handling

### Ad Unavailability

```typescript
const AD_ERROR_HANDLING = {
  noFill: {
    detection: 'onAdFailed callback with no_fill',
    response: {
      ui: 'Hide ad button or show "Try later"',
      retry: 'Preload again in 30 seconds',
      logging: 'Track for fill rate analytics'
    }
  },

  loadFailed: {
    detection: 'onAdFailed with error',
    response: {
      ui: 'Show error, offer retry',
      retry: 'Retry up to 3 times',
      exponentialBackoff: true
    }
  },

  showFailed: {
    detection: 'Ad fails to display',
    response: {
      ui: 'Apologize, suggest retry',
      reward: 'No reward granted',
      logging: 'Track error type'
    }
  },

  networkError: {
    detection: 'Timeout or connection error',
    response: {
      ui: 'Check connection message',
      retry: 'Allow manual retry',
      offline: 'Disable ad buttons when offline'
    }
  },

  userMessage: {
    noFill: 'No ads available right now. Try again later!',
    error: 'Something went wrong. Please try again.',
    network: 'Check your internet connection and try again.'
  }
};
```

---

## Analytics and Tracking

### Ad Analytics Events

```typescript
const AD_ANALYTICS = {
  events: [
    {
      name: 'ad_opportunity_shown',
      properties: {
        placement_id: 'string',
        context: 'Where button was shown'
      }
    },
    {
      name: 'ad_button_clicked',
      properties: {
        placement_id: 'string',
        ad_available: 'boolean'
      }
    },
    {
      name: 'ad_request_sent',
      properties: {
        placement_id: 'string',
        network: 'string'
      }
    },
    {
      name: 'ad_loaded',
      properties: {
        placement_id: 'string',
        network: 'string',
        load_time_ms: 'number'
      }
    },
    {
      name: 'ad_shown',
      properties: {
        placement_id: 'string',
        network: 'string'
      }
    },
    {
      name: 'ad_completed',
      properties: {
        placement_id: 'string',
        network: 'string',
        watch_duration_seconds: 'number'
      }
    },
    {
      name: 'ad_reward_granted',
      properties: {
        placement_id: 'string',
        reward_type: 'string',
        reward_value: 'number'
      }
    },
    {
      name: 'ad_failed',
      properties: {
        placement_id: 'string',
        error_type: 'string',
        error_message: 'string'
      }
    },
    {
      name: 'ad_skipped',
      properties: {
        placement_id: 'string',
        skip_time_seconds: 'number'
      }
    }
  ],

  metrics: [
    'Ads per DAU (daily active user)',
    'Fill rate by placement',
    'Completion rate',
    'Revenue per ad (eCPM)',
    'Revenue per DAU (ARPDAU)',
    'Ad engagement rate by segment',
    'Error rate by network'
  ]
};
```

---

## Revenue Optimization

### Mediation Strategy

```typescript
const MEDIATION_STRATEGY = {
  waterfall: {
    enabled: true,
    networks: [
      { name: 'AppLovin', priority: 1, minEcpm: 20 },
      { name: 'Unity Ads', priority: 2, minEcpm: 15 },
      { name: 'AdMob', priority: 3, minEcpm: 10 },
      { name: 'Vungle', priority: 4, minEcpm: 8 },
      { name: 'Facebook', priority: 5, minEcpm: 5 },
      { name: 'Chartboost', priority: 6, minEcpm: 0 }
    ],
    optimization: 'Auto-optimize based on eCPM data'
  },

  targeting: {
    geo: 'Higher eCPM networks for tier 1 countries',
    segment: 'Different waterfalls for payers vs non-payers'
  },

  testing: {
    abTest: 'Test different waterfall configurations',
    metrics: 'Revenue per impression, fill rate'
  }
};
```

---

## Player Segmentation

### Ad Behavior by Segment

```typescript
const AD_SEGMENTATION = {
  nonPayers: {
    description: 'Players who have never purchased',
    adFrequency: 'Full access to all ad placements',
    limits: 'Standard limits apply',
    messaging: 'Emphasize value of watching ads'
  },

  lowSpenders: {
    description: 'Spent < $10 lifetime',
    adFrequency: 'Standard access',
    limits: 'Standard limits',
    consideration: 'May convert to higher spenders'
  },

  highSpenders: {
    description: 'Spent $10+ lifetime',
    adFrequency: 'Reduced ad prompts',
    limits: 'Standard limits if they choose to watch',
    philosophy: 'Don\'t push ads to paying players'
  },

  subscribers: {
    description: 'Active Battle Pass or subscription',
    adFrequency: 'Optional ads only',
    benefits: 'May have ad-free perks',
    override: 'Can still watch for extra rewards if desired'
  }
};
```

---

## Testing Requirements

```typescript
const AD_TESTING = {
  functional: [
    'Ad loads successfully for each placement',
    'Reward granted after ad completion',
    'No reward for incomplete/skipped ads',
    'Limits enforced correctly',
    'Cooldowns work as expected',
    'UI updates correctly for all states'
  ],

  integration: [
    'All ad networks initialize properly',
    'Mediation selects highest eCPM network',
    'Analytics events fire correctly',
    'Server validation works'
  ],

  userExperience: [
    'Ad buttons are clearly visible',
    'Reward previews are accurate',
    'Loading/completion flow is smooth',
    'Error states are handled gracefully'
  ],

  edge: [
    'Offline behavior handled',
    'App backgrounding during ad',
    'Phone call during ad',
    'Rapid repeated taps',
    'Timezone changes for daily limits'
  ],

  revenue: [
    'Fill rates meet targets (>90%)',
    'eCPM within expected range',
    'No fraudulent impressions'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial rewarded video ads specification |
