# In-Game News and Announcements Specification

## Overview

The In-Game News and Announcements system provides a centralized communication channel for delivering important information, updates, and promotional content directly to players within the game client. This system supports real-time updates without requiring app deployments.

## Announcement Types

```typescript
type AnnouncementType =
  | 'news'           // General news and updates
  | 'event'          // Event announcements
  | 'patch'          // Patch notes
  | 'maintenance'    // Server maintenance alerts
  | 'promotion'      // Shop promotions
  | 'community'      // Community spotlights
  | 'urgent'         // Critical alerts
  | 'seasonal';      // Season announcements

interface Announcement {
  announcementId: string;
  type: AnnouncementType;
  priority: 'low' | 'medium' | 'high' | 'critical';

  content: {
    title: string;
    summary: string;
    body: string; // Rich text/markdown
    imageUrl?: string;
    videoUrl?: string;
    ctaButton?: {
      text: string;
      action: string;
      destination: string;
    };
  };

  targeting: {
    allPlayers: boolean;
    segments?: string[];
    regions?: string[];
    platforms?: string[];
    minLevel?: number;
    maxLevel?: number;
  };

  schedule: {
    publishDate: Date;
    expiryDate?: Date;
    showInFeed: boolean;
    showAsBanner: boolean;
  };

  localization: {
    defaultLanguage: string;
    translations: Map<string, LocalizedContent>;
  };

  metadata: {
    createdBy: string;
    createdAt: Date;
    lastModified: Date;
    version: number;
    tags: string[];
  };
}
```

## News Section Layout

```typescript
const NEWS_SECTION_CONFIG = {
  location: 'mainMenu',
  accessMethod: 'newsButton',

  layout: {
    style: 'feedWithFeatured',
    featuredCount: 1,
    feedItemsPerPage: 10,
    infiniteScroll: true
  },

  featuredSection: {
    position: 'top',
    style: 'largeCard',
    showImage: true,
    showVideo: true,
    autoRotate: {
      enabled: true,
      interval: '10 seconds',
      pauseOnInteraction: true
    }
  },

  feedSection: {
    style: 'compactCards',
    showThumbnail: true,
    showSummary: true,
    showDate: true,
    showTypeIndicator: true,
    groupByDate: true
  },

  filtering: {
    enabled: true,
    filters: ['all', 'news', 'events', 'patches', 'promotions'],
    defaultFilter: 'all'
  },

  sorting: {
    default: 'publishDate_desc',
    options: ['publishDate_desc', 'priority_desc']
  }
};
```

## Banner System

```typescript
const BANNER_SYSTEM = {
  bannerTypes: {
    topBanner: {
      location: 'topOfScreen',
      height: '48px',
      persistent: false,
      dismissible: true,
      autoDismiss: '10 seconds'
    },

    urgentBanner: {
      location: 'centerScreen',
      style: 'modal',
      persistent: true,
      dismissible: true,
      requiresAcknowledge: false
    },

    maintenanceBanner: {
      location: 'topOfScreen',
      height: '48px',
      persistent: true,
      dismissible: false,
      countdown: true
    },

    promotionalBanner: {
      location: 'mainMenu',
      style: 'slideIn',
      dismissible: true,
      autoDismiss: '15 seconds',
      showOnce: true
    }
  },

  priorityDisplay: {
    critical: {
      backgroundColor: '#FF4444',
      textColor: '#FFFFFF',
      icon: 'alert',
      sound: 'urgentAlert'
    },
    high: {
      backgroundColor: '#FF8800',
      textColor: '#FFFFFF',
      icon: 'warning',
      sound: 'notification'
    },
    medium: {
      backgroundColor: '#4488FF',
      textColor: '#FFFFFF',
      icon: 'info',
      sound: null
    },
    low: {
      backgroundColor: '#44AA44',
      textColor: '#FFFFFF',
      icon: 'info',
      sound: null
    }
  },

  bannerBehavior: {
    queueMultiple: true,
    maxQueueSize: 3,
    rotationInterval: '5 seconds',
    priorityOrdering: true
  }
};
```

## Rich Text and Media Support

```typescript
const CONTENT_SUPPORT = {
  textFormatting: {
    markdown: true,
    supportedElements: [
      'headers', // h1-h3
      'bold',
      'italic',
      'links',
      'lists',
      'blockquotes',
      'codeBlocks',
      'horizontalRules',
      'tables'
    ],
    maxLength: 10000 // Characters
  },

  images: {
    supported: true,
    formats: ['jpg', 'png', 'webp', 'gif'],
    maxSize: '2MB',
    aspectRatios: {
      featured: '16:9',
      thumbnail: '1:1',
      inline: 'flexible'
    },
    lazyLoading: true,
    fallbackImage: 'defaultNewsImage'
  },

  videos: {
    supported: true,
    providers: ['cdn', 'youtube', 'vimeo'],
    autoplay: false,
    muted: true,
    showControls: true,
    maxDuration: '5 minutes'
  },

  embeds: {
    gameItems: true, // Item cards
    events: true, // Event cards
    shopItems: true, // Shop previews
    profiles: false // No player profiles
  },

  ctaButtons: {
    maxPerAnnouncement: 2,
    actions: [
      'openUrl',
      'openShop',
      'openEvent',
      'openBattlePass',
      'openSettings',
      'playMatch',
      'openNews'
    ]
  }
};
```

## Read/Unread Tracking

```typescript
interface PlayerNewsState {
  playerId: string;

  readAnnouncements: {
    announcementId: string;
    readAt: Date;
  }[];

  dismissedBanners: {
    announcementId: string;
    dismissedAt: Date;
  }[];

  preferences: {
    showBanners: boolean;
    autoMarkAsRead: boolean;
    notifyNewAnnouncements: boolean;
  };
}

const READ_TRACKING = {
  // When to mark as read
  markAsReadTrigger: {
    onOpen: true, // When full article opened
    onScroll: false, // Not just scrolling past
    onDismiss: false // Dismissed != read
  },

  // Unread indicators
  unreadIndicators: {
    newsButton: {
      badge: 'number', // Shows count
      maxDisplay: '9+',
      animation: 'pulse'
    },

    feedItem: {
      indicator: 'dot',
      color: '#FF4444',
      position: 'topLeft'
    }
  },

  // New content notification
  newContentNotification: {
    enabled: true,
    location: 'mainMenuPopup',
    showOnLogin: true,
    frequency: 'oncePerSession',
    message: 'New announcements available!'
  },

  // Persistence
  persistence: {
    storage: 'server',
    sync: 'onLogin',
    maxHistory: 100 // Last 100 announcements
  }
};
```

## Remote Update System

```typescript
const REMOTE_UPDATE_SYSTEM = {
  contentDelivery: {
    source: 'cdn',
    format: 'json',
    compression: 'gzip',
    caching: {
      clientCache: '5 minutes',
      cdnCache: '1 minute'
    }
  },

  updateMechanism: {
    polling: {
      enabled: true,
      interval: '5 minutes',
      backgroundPolling: false // Only when app active
    },

    pushNotification: {
      enabled: true,
      trigger: 'urgentAnnouncements',
      fallbackToPolling: true
    }
  },

  versionControl: {
    contentVersion: true,
    incrementalUpdates: true,
    fullRefreshTrigger: 'versionMismatch'
  },

  offlineSupport: {
    cacheRecentAnnouncements: true,
    maxCachedItems: 20,
    showCachedWhenOffline: true,
    markAsCached: true
  },

  noDeployRequired: {
    textContent: true,
    imageContent: true,
    videoContent: true,
    ctaActions: true, // Limited to predefined actions
    localization: true
  }
};
```

## Localization System

```typescript
interface LocalizedContent {
  language: string;
  title: string;
  summary: string;
  body: string;
  imageUrl?: string; // Localized images if needed
  ctaText?: string;
}

const LOCALIZATION_CONFIG = {
  supportedLanguages: [
    'en', 'es', 'pt-BR', 'fr', 'de', 'it',
    'ja', 'ko', 'zh-CN', 'zh-TW', 'ru', 'pl', 'tr'
  ],

  defaultLanguage: 'en',

  fallbackBehavior: {
    missingTranslation: 'useDefault',
    partialTranslation: 'mergeWithDefault',
    showLanguageIndicator: true
  },

  contentManagement: {
    perLanguageContent: true,
    sharedImages: true, // Unless localized
    separateApproval: false // Approve all at once
  },

  playerLanguageDetection: {
    source: 'playerSettings',
    fallback: 'deviceLanguage',
    finalFallback: 'en'
  },

  rtlSupport: {
    enabled: true,
    languages: ['ar', 'he'],
    layoutMirroring: true
  }
};
```

## Announcement Scheduling

```typescript
const SCHEDULING_SYSTEM = {
  publishOptions: {
    immediate: true,
    scheduled: true,
    timezone: 'UTC',
    previewBeforePublish: true
  },

  expiryOptions: {
    noExpiry: true,
    fixedDate: true,
    durationBased: true, // e.g., "7 days"
    removeFromFeed: true,
    archiveOnExpiry: true
  },

  recurringAnnouncements: {
    enabled: false, // Not for news (for notifications yes)
    templates: false
  },

  schedulingUI: {
    calendar: true,
    conflictDetection: true, // Warn if too many at once
    previewSchedule: true
  },

  timezoneHandling: {
    storeInUTC: true,
    displayInLocal: true,
    showBothTimezones: true
  }
};
```

## Admin CMS Interface

```typescript
const CMS_INTERFACE = {
  dashboard: {
    overview: {
      activeAnnouncements: true,
      scheduledAnnouncements: true,
      recentlyExpired: true,
      engagementMetrics: true
    }
  },

  announcementEditor: {
    richTextEditor: {
      type: 'WYSIWYG',
      markdownMode: true,
      previewMode: true,
      splitView: true
    },

    mediaManager: {
      upload: true,
      library: true,
      cdn: true,
      crop: true,
      resize: true
    },

    localizationEditor: {
      sideBySide: true,
      machineTranslate: false, // Human translation only
      copyFromDefault: true
    },

    targetingBuilder: {
      visualBuilder: true,
      segmentPicker: true,
      estimatedReach: true
    }
  },

  approvalWorkflow: {
    stages: ['draft', 'review', 'approved', 'published'],
    requiredApprovers: 1,
    autoPublish: true, // After approval, on schedule
    rollback: true
  },

  versionHistory: {
    enabled: true,
    maxVersions: 10,
    diff: true,
    restore: true
  },

  templates: {
    enabled: true,
    types: ['patchNotes', 'eventAnnouncement', 'maintenance', 'promotion'],
    customizable: true
  }
};
```

## Targeting and Segmentation

```typescript
const TARGETING_CONFIG = {
  targetingOptions: {
    allPlayers: true,

    bySegment: {
      enabled: true,
      segments: [
        'newPlayers',      // < 7 days
        'casual',          // < 3 sessions/week
        'regular',         // 3-7 sessions/week
        'hardcore',        // > 7 sessions/week
        'lapsed',          // No login in 7+ days
        'spenders',        // Has made purchase
        'nonSpenders',     // Never purchased
        'battlePassOwners',
        'rankedPlayers'
      ]
    },

    byRegion: {
      enabled: true,
      regions: ['NA', 'EU', 'LATAM', 'APAC', 'OCE', 'ME']
    },

    byPlatform: {
      enabled: true,
      platforms: ['iOS', 'Android']
    },

    byLevel: {
      enabled: true,
      minLevel: true,
      maxLevel: true
    },

    byVersion: {
      enabled: true,
      minVersion: true,
      maxVersion: true
    }
  },

  reachEstimation: {
    enabled: true,
    showEstimate: true,
    warnIfLow: true,
    warnThreshold: 1000
  }
};
```

## Analytics and Metrics

```typescript
const NEWS_ANALYTICS = {
  engagementMetrics: {
    impressions: true, // Banner/feed views
    opens: true, // Full article opens
    readTime: true, // Time spent reading
    ctaClicks: true, // Action button clicks
    shares: false // No sharing feature
  },

  perAnnouncementMetrics: {
    totalImpressions: true,
    uniqueImpressions: true,
    openRate: true,
    averageReadTime: true,
    ctaClickRate: true,
    dismissRate: true // For banners
  },

  aggregateMetrics: {
    dailyImpressions: true,
    dailyOpens: true,
    engagementByType: true,
    engagementBySegment: true
  },

  reporting: {
    realTimeMetrics: true,
    dailyReport: true,
    perAnnouncementReport: true,
    exportToCSV: true
  },

  abTesting: {
    enabled: true,
    testableElements: ['title', 'image', 'cta'],
    statisticalSignificance: true
  }
};
```

## Notification Integration

```typescript
const NOTIFICATION_INTEGRATION = {
  pushNotifications: {
    enabled: true,
    triggerConditions: {
      urgentAnnouncements: true,
      highPriorityAnnouncements: 'optional',
      normalAnnouncements: false
    },

    content: {
      useAnnouncementTitle: true,
      useAnnouncementSummary: true,
      customMessage: false
    },

    timing: {
      immediate: true,
      scheduled: false // Use announcement schedule
    },

    playerPreferences: {
      respectOptOut: true,
      categoryOptOut: true
    }
  },

  inGameNotification: {
    newAnnouncementDot: true,
    popupOnLogin: true,
    soundNotification: false
  }
};
```

## Maintenance Alerts

```typescript
const MAINTENANCE_ALERTS = {
  alertTypes: {
    scheduled: {
      advanceWarning: ['24h', '6h', '1h', '15m'],
      bannerPersistent: true,
      countdownTimer: true,
      dismissible: false
    },

    emergency: {
      advanceWarning: ['15m', '5m'],
      bannerPersistent: true,
      countdownTimer: true,
      forceDisconnect: true
    },

    ongoing: {
      showWhenOnline: true,
      message: 'Maintenance in progress. Thank you for your patience.',
      estimatedCompletion: true
    },

    completed: {
      notification: true,
      message: 'Maintenance complete! Jump back in!',
      pushNotification: true
    }
  },

  maintenanceDisplay: {
    bannerColor: '#FF8800',
    icon: 'maintenance',
    format: 'Maintenance in {time}',
    countdownFormat: 'HH:MM:SS'
  },

  inMatchHandling: {
    showWarning: true,
    gracePeriod: '15 minutes',
    forceEndMatch: false, // Let matches finish
    noNewMatches: true
  }
};
```

## Content Types Templates

```typescript
const CONTENT_TEMPLATES = {
  patchNotes: {
    structure: {
      header: 'Version {version} - {date}',
      sections: ['highlights', 'newFeatures', 'balanceChanges', 'bugFixes', 'knownIssues'],
      footer: 'Thanks for playing!'
    },
    styling: {
      showVersion: true,
      categorizedList: true,
      expandable: true
    }
  },

  eventAnnouncement: {
    structure: {
      header: '{eventName}',
      sections: ['eventDescription', 'rewards', 'schedule', 'howToParticipate'],
      cta: 'View Event'
    },
    styling: {
      eventTheme: true,
      showEventImage: true,
      countdownToStart: true
    }
  },

  seasonAnnouncement: {
    structure: {
      header: 'Season {number}: {name}',
      sections: ['overview', 'newContent', 'battlePass', 'rewards'],
      cta: 'View Battle Pass'
    },
    styling: {
      seasonTheme: true,
      cinematic: true
    }
  },

  promotion: {
    structure: {
      header: '{promotionName}',
      sections: ['offer', 'terms'],
      cta: 'View in Shop'
    },
    styling: {
      promotional: true,
      showDiscount: true,
      urgency: true
    }
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  dataModel: {
    announcements: 'mongodb',
    playerReadState: 'mongodb + redis',
    contentAssets: 'cdn'
  },

  apiEndpoints: {
    getAnnouncements: 'GET /api/news/announcements',
    getAnnouncement: 'GET /api/news/announcements/{id}',
    markAsRead: 'POST /api/news/read/{id}',
    dismissBanner: 'POST /api/news/dismiss/{id}',
    getUnreadCount: 'GET /api/news/unread-count'
  },

  caching: {
    announcementList: 'redis, 5 min TTL',
    playerReadState: 'redis, 1 hour TTL',
    cdnCaching: '1 min for JSON, 24h for assets'
  },

  contentDelivery: {
    source: 'CDN',
    compression: true,
    incrementalLoading: true,
    preloadImages: true
  },

  performance: {
    listLoadTime: '<200ms',
    articleLoadTime: '<500ms',
    imageLoadTime: 'progressive'
  },

  reliability: {
    fallbackContent: true,
    offlineCache: true,
    retryOnFailure: true
  }
};
```
