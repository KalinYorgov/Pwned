# Customer Support Integration Specification

## Overview

The Customer Support Integration system provides players with accessible support channels directly within the game, enabling ticket submission, FAQ access, player reporting, and purchase issue handling while integrating with external support tools for agent management.

## Support Access Points

```typescript
const SUPPORT_ACCESS_POINTS = {
  primary: {
    location: 'settingsMenu',
    section: 'Support',
    icon: 'helpCircle',
    items: [
      'contactSupport',
      'faqHelp',
      'reportProblem',
      'purchaseHistory',
      'reportPlayer'
    ]
  },

  contextual: {
    purchaseScreen: {
      link: 'Having trouble with a purchase?',
      action: 'openPurchaseSupport'
    },

    matchEnd: {
      link: 'Report a player',
      action: 'openPlayerReport'
    },

    errorScreens: {
      link: 'Contact Support',
      action: 'openSupportWithError'
    }
  },

  profileSection: {
    supportButton: true,
    purchaseHistoryButton: true
  },

  floatingButton: {
    enabled: false, // Only in settings
    location: null
  }
};
```

## Ticket Submission System

```typescript
interface SupportTicket {
  ticketId: string;
  playerId: string;

  // Ticket content
  category: TicketCategory;
  subcategory: string;
  subject: string;
  description: string;
  attachments: Attachment[];

  // Auto-captured data
  deviceInfo: DeviceInfo;
  accountInfo: AccountInfo;
  sessionInfo: SessionInfo;

  // Status
  status: 'open' | 'in_progress' | 'waiting_player' | 'resolved' | 'closed';
  priority: 'low' | 'medium' | 'high' | 'urgent';

  // Timestamps
  createdAt: Date;
  updatedAt: Date;
  resolvedAt?: Date;

  // Communication
  messages: TicketMessage[];
  internalNotes?: string[]; // Agent only
}

type TicketCategory =
  | 'technical'
  | 'account'
  | 'billing'
  | 'gameplay'
  | 'report_player'
  | 'bug_report'
  | 'feedback'
  | 'other';

const TICKET_CATEGORIES = {
  technical: {
    label: 'Technical Issue',
    subcategories: [
      'gameNotLoading',
      'crashesAndFreezes',
      'connectionIssues',
      'graphicsProblems',
      'audioIssues',
      'performanceIssues'
    ],
    priority: 'medium',
    autoCapture: ['deviceInfo', 'logs']
  },

  account: {
    label: 'Account Issue',
    subcategories: [
      'loginProblems',
      'accountRecovery',
      'linkAccounts',
      'dataDeletion',
      'nameChange',
      'accountBan'
    ],
    priority: 'high',
    autoCapture: ['accountInfo']
  },

  billing: {
    label: 'Purchase / Billing',
    subcategories: [
      'purchaseNotReceived',
      'wrongItemReceived',
      'duplicateCharge',
      'refundRequest',
      'subscriptionIssue',
      'paymentFailed'
    ],
    priority: 'high',
    autoCapture: ['purchaseHistory', 'accountInfo']
  },

  gameplay: {
    label: 'Gameplay Question',
    subcategories: [
      'howToPlay',
      'rulesAndMechanics',
      'rewardsAndProgress',
      'eventsAndChallenges',
      'cosmeticsAndItems'
    ],
    priority: 'low',
    autoCapture: ['accountInfo']
  },

  report_player: {
    label: 'Report a Player',
    subcategories: [
      'cheating',
      'harassment',
      'inappropriateName',
      'teaming',
      'afkAbuse',
      'otherViolation'
    ],
    priority: 'medium',
    autoCapture: ['matchHistory', 'playerInfo']
  },

  bug_report: {
    label: 'Bug Report',
    subcategories: [
      'gameplayBug',
      'uiBug',
      'questBug',
      'shopBug',
      'matchmakingBug'
    ],
    priority: 'medium',
    autoCapture: ['deviceInfo', 'logs', 'sessionInfo']
  },

  feedback: {
    label: 'Feedback / Suggestion',
    subcategories: [
      'featureRequest',
      'balanceFeedback',
      'uiFeedback',
      'generalFeedback'
    ],
    priority: 'low',
    autoCapture: ['accountInfo']
  },

  other: {
    label: 'Other',
    subcategories: [],
    priority: 'low',
    autoCapture: ['accountInfo']
  }
};
```

## Ticket Form UI

```typescript
const TICKET_FORM_UI = {
  layout: {
    style: 'fullScreen',
    steps: ['category', 'details', 'review', 'submitted']
  },

  step1_category: {
    title: 'What do you need help with?',
    display: 'categoryGrid',
    iconPerCategory: true,
    showSubcategories: true
  },

  step2_details: {
    title: 'Tell us more',
    fields: [
      {
        name: 'subject',
        type: 'text',
        label: 'Subject',
        placeholder: 'Brief description of your issue',
        required: true,
        maxLength: 100
      },
      {
        name: 'description',
        type: 'textarea',
        label: 'Description',
        placeholder: 'Please provide as much detail as possible...',
        required: true,
        minLength: 20,
        maxLength: 2000
      },
      {
        name: 'attachments',
        type: 'fileUpload',
        label: 'Attachments (optional)',
        accept: ['image/*', 'video/*'],
        maxFiles: 3,
        maxFileSize: '10MB'
      }
    ],

    helpText: {
      billing: 'Please include order ID if available',
      technical: 'Describe what happened and when',
      report_player: 'Include the player name and match details'
    }
  },

  step3_review: {
    title: 'Review your request',
    showSummary: true,
    showAutoCapture: {
      deviceInfo: 'summarized',
      accountInfo: false, // Don't confuse player
      purchaseHistory: 'ifBilling'
    },
    editOption: true
  },

  step4_submitted: {
    title: 'Request Submitted',
    showTicketId: true,
    estimatedResponseTime: true,
    checkStatusLink: true,
    returnToGame: true
  }
};
```

## Auto-Captured Information

```typescript
interface DeviceInfo {
  platform: 'iOS' | 'Android';
  osVersion: string;
  deviceModel: string;
  appVersion: string;
  buildNumber: string;
  language: string;
  region: string;
  networkType: string;
  freeStorage: string;
  ramAvailable: string;
}

interface AccountInfo {
  playerId: string;
  displayName: string;
  accountLevel: number;
  createdAt: Date;
  lastLogin: Date;
  linkedAccounts: string[]; // e.g., ['google', 'apple']
  isBanned: boolean;
  isPremium: boolean;
}

interface SessionInfo {
  sessionId: string;
  sessionStart: Date;
  currentScene: string;
  recentActions: string[]; // Last 10 actions
  recentErrors: ErrorLog[];
}

const AUTO_CAPTURE_CONFIG = {
  deviceInfo: {
    capture: 'always',
    display: 'summarized',
    editable: false
  },

  accountInfo: {
    capture: 'always',
    display: 'hidden', // Support sees it, player doesn't
    editable: false
  },

  sessionInfo: {
    capture: 'onTechnicalIssue',
    display: 'hidden',
    editable: false
  },

  logs: {
    capture: 'onBugReport',
    lastNMinutes: 10,
    anonymize: true,
    display: 'hidden'
  },

  purchaseHistory: {
    capture: 'onBillingIssue',
    lastNTransactions: 20,
    display: 'showToPlayer'
  },

  matchHistory: {
    capture: 'onPlayerReport',
    lastNMatches: 5,
    display: 'hidden'
  },

  screenshots: {
    autoCapture: false,
    manualUpload: true
  }
};
```

## Response and Communication

```typescript
const RESPONSE_SYSTEM = {
  channels: {
    inGame: {
      enabled: true,
      notifications: true,
      messageDisplay: 'inboxStyle'
    },

    email: {
      enabled: true,
      fromAddress: 'support@plunderstorm.com',
      replyEnabled: true
    },

    push: {
      enabled: true,
      onNewResponse: true,
      onStatusChange: true
    }
  },

  inGameInbox: {
    location: 'supportSection',
    showActiveTickets: true,
    showClosedTickets: true,
    maxClosedRetention: '30 days',

    ticketView: {
      showConversationThread: true,
      allowReply: true,
      attachmentsInReply: true,
      closeTicket: true,
      reopenTicket: true
    }
  },

  notifications: {
    newResponse: {
      inGame: true,
      push: true,
      email: true,
      badge: 'supportInbox'
    },

    ticketResolved: {
      inGame: true,
      push: true,
      email: true,
      satisfactionSurvey: true
    },

    additionalInfoNeeded: {
      inGame: true,
      push: true,
      email: true,
      urgency: 'medium'
    }
  },

  responseTimeTargets: {
    billing: '4 hours',
    account: '8 hours',
    technical: '24 hours',
    report_player: '48 hours',
    other: '48 hours'
  }
};
```

## FAQ and Help Center

```typescript
const FAQ_SYSTEM = {
  structure: {
    categories: [
      {
        id: 'gettingStarted',
        title: 'Getting Started',
        icon: 'rocket',
        articles: ['howToPlay', 'controls', 'firstMatch', 'abilities']
      },
      {
        id: 'account',
        title: 'Account & Profile',
        icon: 'user',
        articles: ['createAccount', 'linkAccounts', 'changeUsername', 'privacy']
      },
      {
        id: 'purchases',
        title: 'Purchases & Currency',
        icon: 'creditCard',
        articles: ['howToBuy', 'refundPolicy', 'missingPurchase', 'subscriptions']
      },
      {
        id: 'gameplay',
        title: 'Gameplay',
        icon: 'gamepad',
        articles: ['matchTypes', 'ranking', 'quests', 'battlePass']
      },
      {
        id: 'technical',
        title: 'Technical Issues',
        icon: 'wrench',
        articles: ['connectionIssues', 'crashes', 'performance', 'installation']
      },
      {
        id: 'safety',
        title: 'Safety & Reporting',
        icon: 'shield',
        articles: ['reportPlayer', 'blockPlayer', 'safetyFeatures', 'parentalControls']
      }
    ]
  },

  search: {
    enabled: true,
    fullText: true,
    suggestions: true,
    recentSearches: true
  },

  articleFeatures: {
    richText: true,
    images: true,
    videos: true,
    stepByStep: true,
    relatedArticles: true,
    wasThisHelpful: true,
    contactSupportLink: true
  },

  popularArticles: {
    showOnHomepage: true,
    count: 5,
    basedOn: 'viewCount'
  },

  remoteUpdate: {
    enabled: true,
    noAppDeploy: true,
    cdn: true,
    localization: true
  }
};
```

## Player Reporting

```typescript
const PLAYER_REPORT_SYSTEM = {
  reportFlow: {
    trigger: 'postMatch or profile',
    steps: ['selectPlayer', 'selectReason', 'addDetails', 'submit']
  },

  reportReasons: [
    {
      reason: 'cheating',
      label: 'Cheating / Hacking',
      description: 'Using third-party software or exploits',
      severity: 'high',
      autoReview: true
    },
    {
      reason: 'harassment',
      label: 'Harassment / Hate Speech',
      description: 'Abusive behavior or discriminatory language',
      severity: 'high',
      autoReview: true
    },
    {
      reason: 'inappropriateName',
      label: 'Inappropriate Name',
      description: 'Offensive or inappropriate username',
      severity: 'medium',
      autoReview: true
    },
    {
      reason: 'teaming',
      label: 'Teaming',
      description: 'Unfair cooperation with enemies',
      severity: 'medium',
      autoReview: false
    },
    {
      reason: 'afkAbuse',
      label: 'AFK / Intentional Throwing',
      description: 'Not participating or sabotaging matches',
      severity: 'low',
      autoReview: false
    },
    {
      reason: 'other',
      label: 'Other Violation',
      description: 'Other rule violations not listed',
      severity: 'low',
      autoReview: false
    }
  ],

  reportData: {
    reportedPlayerId: true,
    reportedPlayerName: true,
    matchId: true,
    matchTimestamp: true,
    reporterPlayerId: true,
    reason: true,
    additionalDetails: true,
    evidence: 'optional' // Screenshots
  },

  feedback: {
    confirmationMessage: 'Thank you for your report. We take all reports seriously.',
    showCaseNumber: false, // To prevent harassment
    updateOnAction: false // Privacy
  },

  antiAbuse: {
    reportCooldown: '1 per player per day',
    maxReportsPerDay: 10,
    falseReportPenalty: true
  }
};
```

## Purchase History

```typescript
const PURCHASE_HISTORY = {
  display: {
    location: 'supportSection',
    alternativeLocation: 'profileSection',
    title: 'Purchase History'
  },

  transactionList: {
    showAllPurchases: true,
    sortBy: 'date_desc',
    filterBy: ['all', 'currency', 'items', 'subscriptions'],
    pagination: true,
    perPage: 20
  },

  transactionDetails: {
    orderId: true,
    date: true,
    description: true,
    amount: true,
    currency: true,
    paymentMethod: 'lastFourOnly',
    status: true,
    receipt: 'downloadable'
  },

  issueReport: {
    perTransactionButton: true,
    prefillData: true,
    directToBilling: true
  },

  refundEligibility: {
    showStatus: true,
    eligibilityRules: {
      digitalGoods: '14 days if unused',
      currency: 'non-refundable once used',
      subscriptions: 'prorated refund'
    }
  }
};
```

## Bug Report System

```typescript
const BUG_REPORT_SYSTEM = {
  triggerPoints: {
    settingsMenu: true,
    afterCrash: true,
    errorScreen: true,
    shakeToReport: false // Disabled by default
  },

  reportForm: {
    fields: [
      {
        name: 'bugType',
        type: 'select',
        options: ['gameplay', 'ui', 'quest', 'shop', 'matchmaking', 'other']
      },
      {
        name: 'description',
        type: 'textarea',
        placeholder: 'What happened? What were you doing?',
        required: true
      },
      {
        name: 'reproSteps',
        type: 'textarea',
        placeholder: 'How can we reproduce this bug?',
        required: false
      },
      {
        name: 'screenshot',
        type: 'upload',
        required: false
      }
    ]
  },

  autoCapture: {
    deviceInfo: true,
    appVersion: true,
    currentScreen: true,
    recentActions: true,
    errorLogs: true,
    networkStatus: true
  },

  submission: {
    destination: 'supportSystem',
    duplicateDetection: true,
    acknowledgement: 'Thank you for helping us improve the game!'
  }
};
```

## External Support Tool Integration

```typescript
const EXTERNAL_INTEGRATION = {
  supportPlatform: {
    primary: 'zendesk',
    alternatives: ['freshdesk', 'helpshift', 'intercom']
  },

  zendesk: {
    ticketSync: {
      createTicket: true,
      updateTicket: true,
      closeTicket: true,
      bidirectional: true
    },

    dataMapping: {
      playerId: 'custom_field_player_id',
      deviceInfo: 'custom_field_device',
      category: 'ticket_type',
      priority: 'priority',
      subject: 'subject',
      description: 'description'
    },

    agentTools: {
      playerLookup: true,
      accountActions: true,
      purchaseHistory: true,
      matchHistory: true,
      grantRewards: true
    },

    webhooks: {
      onTicketUpdate: true,
      onTicketResolve: true,
      onAgentReply: true
    }
  },

  sso: {
    enabled: true,
    method: 'jwt',
    playerAuth: true
  },

  apiIntegration: {
    authentication: 'apiKey',
    rateLimit: '1000 req/min',
    retryOnFailure: true
  }
};
```

## Agent Admin Tools

```typescript
const AGENT_TOOLS = {
  playerLookup: {
    searchBy: ['playerId', 'displayName', 'email'],
    displayInfo: [
      'accountInfo',
      'deviceInfo',
      'purchaseHistory',
      'matchHistory',
      'ticketHistory',
      'banHistory'
    ]
  },

  accountActions: {
    changeUsername: {
      enabled: true,
      requiresApproval: false,
      logAction: true
    },

    unlinkAccount: {
      enabled: true,
      requiresApproval: true,
      logAction: true
    },

    resetProgress: {
      enabled: true,
      requiresApproval: true,
      logAction: true
    },

    tempBan: {
      enabled: true,
      durations: ['1h', '24h', '7d', '30d'],
      requiresReason: true,
      logAction: true
    },

    permaBan: {
      enabled: true,
      requiresApproval: true,
      requiresReason: true,
      logAction: true
    }
  },

  rewardActions: {
    grantCurrency: {
      enabled: true,
      maxAmount: 5000,
      requiresTicket: true,
      logAction: true
    },

    grantItem: {
      enabled: true,
      itemSelector: true,
      requiresTicket: true,
      logAction: true
    },

    refund: {
      enabled: true,
      requiresApproval: true,
      logAction: true
    }
  },

  auditLog: {
    allActionsLogged: true,
    retention: '2 years',
    exportable: true,
    searchable: true
  }
};
```

## SLA and Metrics

```typescript
const SLA_METRICS = {
  responseTimeSLA: {
    billing: { firstResponse: '4h', resolution: '48h' },
    account: { firstResponse: '8h', resolution: '72h' },
    technical: { firstResponse: '24h', resolution: '5d' },
    other: { firstResponse: '48h', resolution: '7d' }
  },

  metrics: {
    firstResponseTime: true,
    resolutionTime: true,
    customerSatisfaction: true,
    reopenRate: true,
    ticketsPerCategory: true,
    agentPerformance: true
  },

  satisfactionSurvey: {
    trigger: 'onTicketClose',
    questions: [
      { question: 'How satisfied are you with the support?', type: 'rating_5' },
      { question: 'Was your issue resolved?', type: 'yesNo' },
      { question: 'Any additional feedback?', type: 'openText' }
    ]
  },

  escalation: {
    slaBreachAlert: true,
    escalateTo: 'seniorAgent',
    notifyManager: true
  },

  reporting: {
    dailyReport: true,
    weeklyReport: true,
    monthlyAnalysis: true,
    exportFormats: ['pdf', 'csv']
  }
};
```

## Localization

```typescript
const SUPPORT_LOCALIZATION = {
  supportedLanguages: [
    'en', 'es', 'pt-BR', 'fr', 'de', 'it',
    'ja', 'ko', 'zh-CN', 'zh-TW', 'ru', 'pl', 'tr'
  ],

  languageDetection: {
    source: 'playerSettings',
    fallback: 'en'
  },

  localizedContent: {
    ticketForm: true,
    faqArticles: true,
    responseTemplates: true,
    errorMessages: true
  },

  agentLanguages: {
    routing: 'byPlayerLanguage',
    fallback: 'en',
    machineTranslation: 'forReadingOnly'
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  dataModel: {
    tickets: 'mongodb + zendesk sync',
    faqContent: 'cdn',
    purchaseHistory: 'billing service'
  },

  apiEndpoints: {
    createTicket: 'POST /api/support/tickets',
    getTickets: 'GET /api/support/tickets',
    replyToTicket: 'POST /api/support/tickets/{id}/reply',
    getFAQ: 'GET /api/support/faq',
    searchFAQ: 'GET /api/support/faq/search',
    reportPlayer: 'POST /api/support/reports',
    getPurchaseHistory: 'GET /api/support/purchases'
  },

  webhooks: {
    zendeskUpdates: '/api/webhooks/zendesk',
    billingEvents: '/api/webhooks/billing'
  },

  caching: {
    faqContent: 'cdn, 1 hour',
    purchaseHistory: 'redis, 5 min',
    ticketList: 'no cache'
  },

  security: {
    authentication: 'playerSession',
    dataEncryption: 'atRest and inTransit',
    piiHandling: 'gdprCompliant',
    logRetention: '2 years'
  }
};
```
