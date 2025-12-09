# Survey and Feedback System Specification

## Document Information
- **Task ID:** LIVE-022
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the survey and feedback system for Plunderstorm Mobile, enabling structured collection of player feedback through in-game surveys, ratings, and feedback forms to drive continuous improvement.

---

## Survey Types

### Net Promoter Score (NPS)

```typescript
const NPS_SURVEY = {
  id: 'nps_survey',
  type: 'nps',

  question: {
    primary: 'How likely are you to recommend Plunderstorm to a friend?',
    scale: '0-10 (0 = Not at all, 10 = Extremely likely)'
  },

  followUp: {
    promoters: {
      range: [9, 10],
      question: 'What do you love most about Plunderstorm?',
      type: 'open_text'
    },
    passives: {
      range: [7, 8],
      question: 'What would make you more likely to recommend us?',
      type: 'open_text'
    },
    detractors: {
      range: [0, 6],
      question: 'What could we improve to make your experience better?',
      type: 'open_text'
    }
  },

  triggers: {
    firstTrigger: 'After 20 matches OR 7 days of play',
    recurring: 'Every 90 days',
    postUpdate: '7 days after major update'
  },

  frequency: {
    maxPerQuarter: 1,
    minDaysBetween: 90
  }
};
```

### Customer Satisfaction (CSAT)

```typescript
const CSAT_SURVEY = {
  id: 'csat_survey',
  type: 'csat',

  question: {
    primary: 'How satisfied are you with Plunderstorm?',
    scale: '1-5 stars or emojis'
  },

  followUp: {
    lowRating: {
      threshold: 3,
      question: 'What disappointed you?',
      options: [
        'Matchmaking issues',
        'Performance problems',
        'Unfair gameplay',
        'Lack of content',
        'Technical bugs',
        'Other'
      ]
    },
    highRating: {
      threshold: 4,
      question: 'What do you enjoy most?',
      options: [
        'Combat system',
        'Graphics and style',
        'Variety of abilities',
        'Playing with friends',
        'Events and updates',
        'Other'
      ]
    }
  },

  triggers: {
    periodic: 'Monthly check-in',
    postEvent: 'After major event participation'
  }
};
```

### Feature-Specific Surveys

```typescript
const FEATURE_SURVEYS = {
  newFeature: {
    id: 'new_feature_survey',
    timing: 'After using new feature 3+ times',
    question: 'How do you like the new {featureName}?',
    scale: '1-5 stars',
    followUp: 'Any suggestions to improve it?',
    examples: ['New ability', 'New game mode', 'UI change']
  },

  balanceChange: {
    id: 'balance_survey',
    timing: '1 week after balance patch',
    question: 'How do you feel about recent balance changes?',
    options: ['Much better', 'Slightly better', 'No change', 'Slightly worse', 'Much worse'],
    followUp: 'Which changes do you have feedback on?'
  },

  eventFeedback: {
    id: 'event_survey',
    timing: 'End of event',
    questions: [
      { q: 'How would you rate this event overall?', type: '1-5' },
      { q: 'Was the event too long, too short, or just right?', type: 'choice' },
      { q: 'Did you feel the rewards were worth the effort?', type: 'choice' },
      { q: 'What would you change for future events?', type: 'open_text' }
    ]
  }
};
```

### Post-Match Rating

```typescript
const POST_MATCH_RATING = {
  id: 'post_match_rating',
  type: 'quick_rating',

  display: {
    when: 'Occasionally on post-match screen',
    frequency: '1 in 20 matches',
    position: 'Non-blocking, optional'
  },

  question: {
    primary: 'Rate this match:',
    options: ['👎', '😐', '👍'],
    optional: true
  },

  followUp: {
    negative: {
      question: 'What went wrong?',
      options: [
        'Laggy/poor performance',
        'Cheaters/hackers',
        'Unfair matchmaking',
        'Bugs/glitches',
        'Toxic players',
        'Other'
      ]
    }
  },

  metadata: {
    capture: [
      'Match ID',
      'Game mode',
      'Placement',
      'Performance metrics',
      'Device info'
    ],
    purpose: 'Correlate feedback with match conditions'
  }
};
```

---

## Feedback Forms

### General Feedback

```typescript
const GENERAL_FEEDBACK = {
  access: 'Settings > Help & Feedback > Send Feedback',

  form: {
    category: {
      label: 'What\'s this about?',
      options: [
        'Bug report',
        'Feature request',
        'Balance suggestion',
        'General feedback',
        'Compliment',
        'Other'
      ],
      required: true
    },
    subject: {
      label: 'Brief summary',
      type: 'text',
      maxLength: 100,
      required: true
    },
    details: {
      label: 'Tell us more',
      type: 'textarea',
      maxLength: 1000,
      required: true
    },
    attachments: {
      label: 'Add screenshots (optional)',
      type: 'image',
      maxCount: 3,
      maxSizeMB: 5
    },
    contact: {
      label: 'May we contact you?',
      type: 'checkbox',
      default: true
    }
  },

  autoCapture: {
    deviceInfo: 'Device model, OS version',
    gameInfo: 'Game version, account level',
    recentMatches: 'Last 5 match IDs',
    settings: 'Graphics and audio settings'
  }
};
```

### Bug Report Form

```typescript
const BUG_REPORT = {
  access: 'Settings > Help & Feedback > Report Bug',

  form: {
    bugType: {
      label: 'Bug type',
      options: [
        'Crash',
        'Visual glitch',
        'Gameplay issue',
        'Audio problem',
        'UI/UX issue',
        'Connection/network',
        'Other'
      ]
    },
    frequency: {
      label: 'How often does this happen?',
      options: ['Once', 'Sometimes', 'Often', 'Always']
    },
    steps: {
      label: 'Steps to reproduce',
      type: 'textarea',
      placeholder: '1. Go to...\n2. Click on...\n3. Bug occurs...'
    },
    expected: {
      label: 'What should happen?',
      type: 'textarea'
    },
    actual: {
      label: 'What actually happened?',
      type: 'textarea'
    },
    screenshot: {
      label: 'Screenshot or video',
      type: 'media',
      optional: true
    }
  },

  autoCapture: {
    crashLogs: 'If crash, attach recent logs',
    systemInfo: 'Full device diagnostics',
    networkInfo: 'Connection type and quality'
  }
};
```

---

## Survey Triggers

### Trigger Logic

```typescript
const TRIGGER_LOGIC = {
  conditions: {
    accountAge: 'Minimum days since account creation',
    matchesPlayed: 'Minimum matches completed',
    recentActivity: 'Active within last X days',
    lastSurvey: 'Days since last survey of any type',
    specificSurvey: 'Days since last survey of this type'
  },

  examples: {
    nps: {
      conditions: [
        { field: 'matchesPlayed', operator: '>=', value: 20 },
        { field: 'accountAge', operator: '>=', value: 7 },
        { field: 'lastNPS', operator: '>=', value: 90 }
      ]
    },
    postMatch: {
      conditions: [
        { field: 'matchesPlayed', operator: '>=', value: 5 },
        { field: 'lastPostMatch', operator: '>=', value: 3 }
      ],
      randomChance: 0.05
    }
  },

  timing: {
    immediate: 'Show at natural break point',
    delayed: 'Queue for next session',
    scheduled: 'Show at specific time/date'
  }
};
```

### Trigger Points

```typescript
const TRIGGER_POINTS = {
  sessionStart: {
    after: 'App launch, after login',
    surveys: ['NPS', 'CSAT'],
    frequency: 'Rare'
  },

  postMatch: {
    after: 'Match results viewed',
    surveys: ['Post-match rating', 'Quick polls'],
    frequency: 'Occasional'
  },

  featureUse: {
    after: 'Using specific feature',
    surveys: ['Feature feedback'],
    frequency: 'After threshold uses'
  },

  eventEnd: {
    after: 'Event completion or expiration',
    surveys: ['Event feedback'],
    frequency: 'Per event'
  },

  milestone: {
    after: 'Account milestones (level, rank)',
    surveys: ['General satisfaction'],
    frequency: 'At specific milestones'
  },

  returnFromChurn: {
    after: 'Returning after 7+ day absence',
    surveys: ['Why did you come back?'],
    frequency: 'Once per return'
  }
};
```

---

## Non-Intrusive Design

### Presentation

```typescript
const PRESENTATION = {
  modal: {
    style: 'Overlay with dismiss option',
    animation: 'Subtle slide-in',
    blocking: false,
    dismissable: true
  },

  banner: {
    style: 'Non-modal banner at top/bottom',
    persistence: 'Dismissable, may return once',
    action: 'Tap to expand to full survey'
  },

  inline: {
    style: 'Embedded in existing UI flow',
    location: 'Post-match screen, event summary',
    prominence: 'Low, optional'
  }
};
```

### Respect User Time

```typescript
const TIME_RESPECT = {
  length: {
    quick: 'Under 30 seconds (1-2 questions)',
    standard: '1-2 minutes (3-5 questions)',
    detailed: '3-5 minutes (with warning)'
  },

  optionality: {
    alwaysSkippable: true,
    skipButton: 'Prominent "Not now" or "Skip"',
    noReminder: 'Don\'t re-prompt same survey after skip',
    gracePeriod: '24+ hours before any survey after skip'
  },

  timing: {
    avoidGameplay: 'Never during active match',
    avoidMatchmaking: 'Never during queue',
    preferBreaks: 'Natural pauses in player flow'
  }
};
```

---

## Frequency Limits

### Global Limits

```typescript
const FREQUENCY_LIMITS = {
  perDay: {
    surveys: 1,
    exception: 'Critical bug reports (user-initiated)'
  },

  perWeek: {
    surveys: 2,
    exception: 'Event-specific surveys'
  },

  perMonth: {
    nps: 1,
    csat: 1,
    featureSurveys: 2,
    postMatch: 'No limit (very low frequency)'
  },

  cooldown: {
    afterSurvey: '24 hours before any survey',
    afterSkip: '48 hours before same survey type',
    afterComplete: '7 days before similar survey'
  }
};
```

### Priority System

```typescript
const SURVEY_PRIORITY = {
  levels: {
    critical: {
      priority: 1,
      examples: ['Post-crash feedback', 'Urgent issue detection'],
      override: 'Can override cooldowns'
    },
    high: {
      priority: 2,
      examples: ['NPS', 'Major feature feedback'],
      override: 'Respects cooldowns'
    },
    normal: {
      priority: 3,
      examples: ['CSAT', 'Event feedback'],
      override: 'Respects all limits'
    },
    low: {
      priority: 4,
      examples: ['Post-match rating', 'Quick polls'],
      override: 'Most restrictive'
    }
  },

  queue: {
    behavior: 'Higher priority surveys shown first',
    expiration: 'Queued surveys expire after 7 days'
  }
};
```

---

## Data Collection

### Response Storage

```typescript
const RESPONSE_STORAGE = {
  schema: {
    responseId: 'string (UUID)',
    surveyId: 'string',
    surveyVersion: 'number',
    playerId: 'string',
    timestamp: 'timestamp',
    responses: 'object (question -> answer)',
    metadata: {
      deviceInfo: 'object',
      gameVersion: 'string',
      accountLevel: 'number',
      matchesPlayed: 'number'
    }
  },

  storage: {
    database: 'Analytics data warehouse',
    retention: '2 years',
    anonymization: 'Option to submit anonymously'
  },

  compliance: {
    gdpr: 'Right to access and delete',
    consent: 'Implicit consent via survey completion',
    transparency: 'Privacy policy explains data use'
  }
};
```

### Real-Time Alerts

```typescript
const REALTIME_ALERTS = {
  triggers: {
    lowNPS: {
      condition: 'NPS score drops below 30',
      alert: 'Slack notification to product team'
    },
    bugSpike: {
      condition: '10+ similar bug reports in 1 hour',
      alert: 'PagerDuty alert to engineering'
    },
    negativeSpike: {
      condition: 'Sudden increase in negative feedback',
      alert: 'Email to stakeholders'
    }
  },

  dashboard: {
    realtime: 'Live feed of incoming feedback',
    aggregates: 'Rolling averages and trends',
    alerts: 'Active alert status'
  }
};
```

---

## Analysis and Reporting

### Dashboards

```typescript
const FEEDBACK_DASHBOARDS = {
  executive: {
    metrics: ['NPS trend', 'CSAT score', 'Top issues'],
    frequency: 'Weekly summary',
    audience: 'Leadership'
  },

  product: {
    metrics: ['Feature ratings', 'Request themes', 'Pain points'],
    frequency: 'Daily updates',
    audience: 'Product team'
  },

  engineering: {
    metrics: ['Bug reports', 'Crash feedback', 'Performance issues'],
    frequency: 'Real-time',
    audience: 'Engineering team'
  },

  liveOps: {
    metrics: ['Event feedback', 'Balance sentiment', 'Content requests'],
    frequency: 'Post-event and weekly',
    audience: 'LiveOps team'
  }
};
```

### Text Analysis

```typescript
const TEXT_ANALYSIS = {
  methods: {
    categorization: {
      description: 'Auto-categorize open-text responses',
      categories: ['Bug', 'Feature request', 'Praise', 'Complaint', 'Question']
    },
    sentiment: {
      description: 'Sentiment analysis on responses',
      output: ['Positive', 'Neutral', 'Negative']
    },
    themes: {
      description: 'Extract common themes',
      method: 'NLP clustering of similar responses'
    }
  },

  tools: {
    automated: 'ML-based categorization',
    manual: 'Weekly review of samples',
    hybrid: 'Auto-tag with human verification'
  }
};
```

---

## Action Loop

### Closing the Loop

```typescript
const ACTION_LOOP = {
  acknowledgment: {
    immediate: 'Thank you message after submission',
    followUp: 'Email confirmation (if opted in)'
  },

  communication: {
    fixedIssues: 'Patch notes mention fixed issues from feedback',
    implemented: 'Feature announcements credit player suggestions',
    youSpoke: '"You spoke, we listened" communications'
  },

  transparency: {
    publicRoadmap: 'Share what feedback influenced',
    devBlogs: 'Discuss how feedback shapes decisions',
    community: 'Respond to common themes publicly'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Surveys display at correct triggers',
    'Responses save correctly',
    'Frequency limits enforced',
    'Skip/dismiss works properly'
  ],

  targeting: [
    'Conditions evaluated correctly',
    'Priority system works',
    'Queue system functions',
    'Cooldowns respected'
  ],

  ux: [
    'Non-intrusive presentation',
    'Easy to dismiss',
    'Mobile-friendly forms',
    'Accessible design'
  ],

  data: [
    'Responses reach analytics',
    'Metadata captured correctly',
    'Real-time alerts trigger',
    'Dashboards populate'
  ],

  edge: [
    'Survey during network loss',
    'Very long text responses',
    'Rapid multiple submissions',
    'Survey version changes mid-session'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial survey and feedback system specification |
