# Social Sharing Specification

## Document Information
- **Task ID:** LIVE-019
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the social sharing system for Plunderstorm Mobile, enabling players to share achievements, match results, and memorable moments to social media platforms, driving organic visibility and player acquisition.

---

## Sharing Triggers

### Shareable Moments

```typescript
const SHAREABLE_MOMENTS = {
  matchResults: {
    trigger: 'Post-match results screen',
    content: {
      victory: 'Victory celebration card',
      stats: 'Eliminations, placement, damage',
      mode: 'Match type and mode'
    },
    priority: 'High - primary sharing moment'
  },

  achievements: {
    trigger: 'Achievement unlock notification',
    content: {
      badge: 'Achievement icon and name',
      description: 'What was accomplished',
      rarity: 'Achievement rarity indicator'
    },
    priority: 'Medium'
  },

  rankUp: {
    trigger: 'Rank promotion',
    content: {
      newRank: 'Rank icon and name',
      progress: 'Previous rank to new rank',
      season: 'Current season identifier'
    },
    priority: 'High'
  },

  battlePassTier: {
    trigger: 'Battle Pass tier unlock (milestones)',
    content: {
      tier: 'Tier number',
      reward: 'Reward earned',
      progress: 'Season progress'
    },
    milestones: [10, 25, 50, 75, 100],
    priority: 'Medium'
  },

  seasonEnd: {
    trigger: 'Season end summary',
    content: {
      peakRank: 'Highest rank achieved',
      stats: 'Season statistics',
      rewards: 'Rewards earned'
    },
    priority: 'High'
  },

  cosmetic: {
    trigger: 'Rare cosmetic unlock',
    content: {
      item: 'Cosmetic preview',
      rarity: 'Item rarity',
      source: 'How it was earned'
    },
    rarityThreshold: 'Epic or higher',
    priority: 'Medium'
  },

  screenshot: {
    trigger: 'Manual screenshot button',
    content: {
      capture: 'Current screen',
      watermark: 'Game branding overlay'
    },
    priority: 'User-initiated'
  }
};
```

---

## Share Card Design

### Victory Card

```typescript
const VICTORY_CARD = {
  dimensions: {
    standard: '1200x630 (Facebook/Twitter optimized)',
    story: '1080x1920 (Instagram/TikTok Stories)',
    square: '1080x1080 (Instagram feed)'
  },

  elements: {
    background: {
      type: 'Dynamic based on match outcome',
      victory: 'Golden celebration theme',
      topPlacement: 'Silver/bronze for top 5/10'
    },
    playerInfo: {
      avatar: 'Player outfit/character',
      name: 'Player display name',
      title: 'Equipped title (if any)'
    },
    stats: {
      placement: '#1 Victory!',
      eliminations: 'X Eliminations',
      damage: 'X Damage Dealt'
    },
    branding: {
      logo: 'Plunderstorm logo',
      tagline: 'Join the plunder!',
      callToAction: 'Download Now' or app store link'
    },
    qrCode: {
      optional: true,
      content: 'Deep link to app store'
    }
  },

  variations: {
    soloWin: 'Solo victory celebration',
    duoWin: 'Both players featured',
    squadWin: 'Squad composition featured',
    elimination: 'High elimination game highlight'
  }
};
```

### Achievement Card

```typescript
const ACHIEVEMENT_CARD = {
  dimensions: {
    standard: '1200x630',
    square: '1080x1080'
  },

  elements: {
    background: 'Achievement category themed',
    achievementIcon: {
      size: 'Large, centered',
      effect: 'Glow based on rarity'
    },
    text: {
      title: 'Achievement Unlocked!',
      name: 'Achievement name',
      description: 'What was accomplished'
    },
    rarity: {
      display: 'Rarity badge',
      color: 'Matching rarity tier'
    },
    branding: {
      logo: 'Plunderstorm logo',
      callToAction: 'How many can you unlock?'
    }
  }
};
```

### Rank Card

```typescript
const RANK_CARD = {
  dimensions: {
    standard: '1200x630',
    story: '1080x1920'
  },

  elements: {
    background: 'Rank-themed (colors/effects)',
    rankIcon: {
      previous: 'Small, to the left',
      arrow: 'Upgrade arrow animation',
      new: 'Large, prominent'
    },
    text: {
      title: 'Rank Up!',
      rank: 'New rank name',
      season: 'Season X'
    },
    playerInfo: {
      name: 'Player name',
      badge: 'Current badges'
    },
    branding: 'Game logo and CTA'
  }
};
```

---

## Platform Integration

### iOS Sharing

```typescript
const IOS_SHARING = {
  implementation: {
    framework: 'UIActivityViewController',
    customActivities: 'Optional custom share extensions'
  },

  supported: {
    native: [
      'Messages',
      'Mail',
      'AirDrop',
      'Notes',
      'Save to Photos'
    ],
    thirdParty: [
      'Facebook',
      'Twitter/X',
      'Instagram',
      'WhatsApp',
      'Snapchat',
      'TikTok',
      'Discord'
    ]
  },

  content: {
    image: 'Share card image',
    text: 'Pre-filled caption',
    url: 'App Store link'
  },

  stories: {
    instagram: 'Deep link to Instagram Stories',
    facebook: 'Facebook Stories integration',
    snapchat: 'Snapchat Stories kit'
  }
};
```

### Android Sharing

```typescript
const ANDROID_SHARING = {
  implementation: {
    intent: 'ACTION_SEND with MIME type',
    chooser: 'Intent.createChooser'
  },

  supported: {
    native: [
      'Messages',
      'Gmail',
      'Google Photos',
      'Nearby Share'
    ],
    thirdParty: [
      'Facebook',
      'Twitter/X',
      'Instagram',
      'WhatsApp',
      'Snapchat',
      'TikTok',
      'Discord'
    ]
  },

  content: {
    image: 'Share card as URI',
    text: 'Pre-filled caption',
    url: 'Play Store link'
  }
};
```

---

## Share Flow

### User Experience

```typescript
const SHARE_FLOW = {
  trigger: {
    button: 'Share button on shareable screens',
    icon: 'Standard share icon (platform native)',
    position: 'Prominent but not intrusive'
  },

  preview: {
    enabled: true,
    content: 'Preview of share card',
    edit: 'Option to customize caption',
    format: 'Choose format (standard/story/square)'
  },

  shareSheet: {
    ios: 'Native UIActivityViewController',
    android: 'Native Intent chooser',
    preselect: 'Recently used apps at top'
  },

  postShare: {
    confirmation: 'Share successful toast',
    reward: 'Small reward for first share (one-time)',
    cooldown: 'No spam prevention needed (user-initiated)'
  }
};
```

### Caption Templates

```typescript
const CAPTION_TEMPLATES = {
  victory: {
    default: '🏴‍☠️ Victory in Plunderstorm! {eliminations} eliminations 💀 #Plunderstorm #BattleRoyale',
    variants: [
      'Just claimed victory in Plunderstorm! 🏆',
      'Captain of the seas! Victory is mine! 🏴‍☠️',
      '{eliminations} eliminations and the crown! 👑'
    ]
  },

  achievement: {
    default: '🎮 Achievement Unlocked: {achievementName} in Plunderstorm! #Plunderstorm',
    variants: [
      'Unlocked {achievementName}! Can you beat that? 🏆',
      'Another achievement down! {achievementName} ✅'
    ]
  },

  rankUp: {
    default: '📈 Ranked up to {rank} in Plunderstorm Season {season}! #Ranked #Plunderstorm',
    variants: [
      'Climbing the ranks! Now a {rank} 🏴‍☠️',
      'New rank achieved: {rank}! Season {season} going strong 💪'
    ]
  },

  editable: true,
  maxLength: 280,
  hashtags: ['#Plunderstorm', '#MobileGaming', '#BattleRoyale']
};
```

---

## Screenshot System

### In-Game Screenshot

```typescript
const SCREENSHOT_SYSTEM = {
  capture: {
    button: 'Camera icon in HUD or pause menu',
    hotkey: 'Volume buttons (optional)',
    timing: 'Capture current frame'
  },

  processing: {
    resolution: 'Device native resolution',
    watermark: {
      logo: 'Plunderstorm logo corner',
      playerTag: 'Player name (optional)',
      opacity: '80% transparency'
    },
    filters: 'Optional post-capture filters'
  },

  storage: {
    local: 'Save to device gallery',
    cloud: 'Optional cloud storage for history',
    quality: 'High quality JPEG/PNG'
  },

  sharing: {
    immediate: 'Share button after capture',
    gallery: 'Access from screenshot gallery in-game',
    editing: 'Basic crop/filter before sharing'
  }
};
```

### Screenshot Gallery

```typescript
const SCREENSHOT_GALLERY = {
  access: 'Profile > Screenshots',

  display: {
    grid: 'Thumbnail grid view',
    detail: 'Full-size view on tap',
    metadata: 'Date, match type, stats'
  },

  actions: {
    share: 'Share to social',
    save: 'Save to device',
    delete: 'Remove from gallery'
  },

  storage: {
    limit: 'Last 100 screenshots',
    cleanup: 'Auto-delete oldest when at limit'
  }
};
```

---

## Deep Linking

### Share Links

```typescript
const SHARE_LINKS = {
  format: {
    domain: 'plunderstorm.game',
    structure: 'https://plunderstorm.game/share/{contentType}/{id}'
  },

  contentTypes: {
    match: '/share/match/{matchId}',
    achievement: '/share/achievement/{achievementId}',
    profile: '/share/player/{playerId}',
    invite: '/share/invite/{referralCode}'
  },

  behavior: {
    installed: 'Open app to relevant content',
    notInstalled: 'Web preview page with app store links'
  },

  tracking: {
    source: 'Track which platform share came from',
    conversion: 'Track installs from shares',
    attribution: 'Link to sharing player for referral'
  }
};
```

### Landing Page

```typescript
const LANDING_PAGE = {
  uninstalledUser: {
    content: [
      'Share preview (image)',
      'Game description',
      'App store buttons',
      'Play on web option (if available)'
    ],
    design: 'Mobile-optimized, fast loading'
  },

  seo: {
    metadata: 'Open Graph and Twitter Card tags',
    preview: 'Rich preview in social feeds',
    image: 'Share card image'
  }
};
```

---

## Privacy Controls

### Settings

```typescript
const PRIVACY_SETTINGS = {
  location: 'Settings > Privacy > Sharing',

  options: {
    includeName: {
      setting: 'Include my name in shares',
      default: true,
      description: 'Show your display name on share cards'
    },
    includeStats: {
      setting: 'Include match stats',
      default: true,
      description: 'Show eliminations, damage, etc.'
    },
    includeRank: {
      setting: 'Show my rank',
      default: true,
      description: 'Display your current rank on share cards'
    }
  },

  preview: {
    enabled: true,
    description: 'See exactly what will be shared',
    edit: 'Remove sensitive info before sharing'
  }
};
```

---

## Rewards and Incentives

### First Share Reward

```typescript
const SHARE_REWARDS = {
  firstShare: {
    reward: '100 Gold',
    type: 'One-time',
    trigger: 'First successful share to any platform'
  },

  milestones: {
    shares5: { reward: '200 Gold', badge: 'Social Butterfly' },
    shares25: { reward: '500 Gold', badge: 'Influencer' },
    shares100: { reward: 'Social Star Title', badge: 'Social Star' }
  },

  referral: {
    integration: 'Link to referral program (LIVE-018)',
    tracking: 'Track installs from shared content',
    reward: 'Referral rewards for successful installs'
  }
};
```

---

## Analytics

### Metrics

```typescript
const SHARE_ANALYTICS = {
  volume: {
    totalShares: 'Total shares per day/week/month',
    byType: 'Shares by content type',
    byPlatform: 'Shares by destination platform'
  },

  engagement: {
    clickThrough: 'Clicks on shared links',
    appStoreVisits: 'Store visits from shares',
    installs: 'Installs attributed to shares'
  },

  conversion: {
    shareToInstall: 'Share to install conversion rate',
    installToPlayer: 'Install to active player rate',
    playerToSharer: 'Players who become sharers'
  },

  content: {
    topSharedMoments: 'Most shared content types',
    viralContent: 'Shares that drive most installs',
    engagement: 'Which share cards get most clicks'
  }
};
```

---

## Technical Implementation

### Image Generation

```typescript
const IMAGE_GENERATION = {
  method: {
    client: 'Generate on device for privacy',
    fallback: 'Server-side if device constraints'
  },

  technology: {
    ios: 'Core Graphics / UIKit',
    android: 'Canvas / Bitmap',
    unity: 'RenderTexture to PNG'
  },

  performance: {
    async: 'Generate in background',
    caching: 'Cache templates, compose dynamically',
    size: 'Optimize file size for sharing'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Share button appears on all shareable screens',
    'Share sheet opens correctly on both platforms',
    'Share cards generate with correct content',
    'Deep links resolve correctly'
  ],

  platforms: [
    'Test on major social platforms (FB, Twitter, Instagram)',
    'Test on messaging apps (WhatsApp, Discord)',
    'Test native sharing (Messages, Mail)'
  ],

  visual: [
    'Share cards display correctly in all formats',
    'Text is readable and not cut off',
    'Branding is visible and correct'
  ],

  privacy: [
    'Settings correctly hide name/stats',
    'Preview shows accurate content',
    'No PII shared without consent'
  ],

  edge: [
    'Sharing with no network',
    'Very long player names',
    'Unusual characters in names',
    'App not installed on link click'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial social sharing specification |
