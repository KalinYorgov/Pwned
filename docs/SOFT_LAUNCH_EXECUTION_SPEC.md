# Soft Launch Execution Specification

## Document Information
- **Task ID:** PROJ-009
- **Priority:** P0
- **Complexity:** Large
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Soft Launch Execution plan for Plunderstorm Mobile, covering market selection, launch procedures, metrics tracking, rapid iteration, and go/no-go decision criteria for proceeding to global launch.

---

## Soft Launch Strategy

### Objectives

```typescript
const SOFT_LAUNCH_OBJECTIVES = {
  primary: {
    validateRetention: 'Confirm D1, D7, D30 meet targets',
    testMonetization: 'Validate IAP conversion and ARPU',
    proveStability: 'Zero critical issues at scale',
    tuneEconomy: 'Balance progression and currency'
  },

  secondary: {
    serverStressTest: 'Validate infrastructure scaling',
    liveOpsValidation: 'Test content update pipeline',
    communityBuilding: 'Gather early feedback and advocates',
    marketingTest: 'Small-scale UA experiments'
  },

  constraints: {
    marketingSpend: 'Minimal (organic + small tests)',
    duration: '4-8 weeks',
    scope: '2-3 markets'
  }
};
```

### Market Selection

```typescript
const SOFT_LAUNCH_MARKETS = {
  primary: {
    philippines: {
      rationale: 'Large mobile gaming population, English-speaking, representative of SEA',
      timezone: 'UTC+8',
      population: '115M',
      mobileGamersPercent: 45,
      avgArpu: 'Low (economy testing)',
      appStores: ['iOS App Store', 'Google Play']
    }
  },

  secondary: {
    netherlands: {
      rationale: 'High-value EU market, English proficiency, small but representative',
      timezone: 'UTC+1',
      population: '17M',
      mobileGamersPercent: 35,
      avgArpu: 'High (monetization testing)',
      appStores: ['iOS App Store', 'Google Play']
    },
    canada: {
      rationale: 'Similar to US market, English/French, good infrastructure',
      timezone: 'UTC-5 to UTC-8',
      population: '38M',
      mobileGamersPercent: 40,
      avgArpu: 'High (monetization testing)',
      appStores: ['iOS App Store', 'Google Play']
    }
  },

  alternates: {
    australia: 'High ARPU, English, good testing ground',
    newZealand: 'Small but high-quality market',
    singapore: 'Tech-savvy, high ARPU, English'
  }
};
```

---

## Launch Procedures

### Pre-Launch Checklist

```typescript
const PRE_LAUNCH_CHECKLIST = {
  appStore: {
    ios: [
      'App submitted to App Store Connect',
      'In-app purchases configured',
      'App Review approved',
      'Release held for manual release',
      'Phased rollout configured (if desired)'
    ],
    android: [
      'App uploaded to Google Play Console',
      'In-app products configured',
      'Internal testing complete',
      'Closed testing (if applicable)',
      'Production track configured for specific countries'
    ]
  },

  backend: [
    'Servers deployed in soft launch regions',
    'Database scaled for projected traffic',
    'CDN configured for asset delivery',
    'Monitoring dashboards active',
    'Alerting thresholds configured',
    'On-call rotation scheduled'
  ],

  analytics: [
    'All events firing correctly',
    'Dashboards displaying data',
    'Funnel reports configured',
    'Cohort analysis ready',
    'Real-time monitoring active'
  ],

  support: [
    'FAQ published',
    'Support email configured',
    'Discord server ready (optional)',
    'Known issues documented',
    'Response templates prepared'
  ]
};
```

### Launch Day Timeline

```typescript
const LAUNCH_DAY_TIMELINE = {
  t_minus_24h: {
    actions: [
      'Final build verification',
      'All team on standby notification',
      'War room scheduled',
      'Monitoring dashboards open'
    ]
  },

  t_minus_2h: {
    actions: [
      'War room assembled',
      'Communication channels open',
      'Final go/no-go check'
    ]
  },

  t_0: {
    actions: [
      'Release app in iOS App Store',
      'Release app in Google Play Store',
      'Announce in Discord (if applicable)',
      'Begin monitoring'
    ]
  },

  t_plus_1h: {
    actions: [
      'First installs verified',
      'First matches completed',
      'No critical errors confirmed',
      'Initial metrics baseline'
    ]
  },

  t_plus_24h: {
    actions: [
      'D0 metrics review',
      'Bug triage',
      'First hotfix if needed',
      'Team debrief'
    ]
  }
};
```

---

## Metrics Framework

### Key Performance Indicators

```typescript
const KPI_TARGETS = {
  retention: {
    d1: { target: 40, minimum: 30, unit: '%' },
    d7: { target: 15, minimum: 10, unit: '%' },
    d30: { target: 5, minimum: 3, unit: '%' }
  },

  engagement: {
    sessionsPerDay: { target: 2.5, minimum: 2.0 },
    avgSessionLength: { target: 15, minimum: 10, unit: 'minutes' },
    matchesPerSession: { target: 3, minimum: 2 }
  },

  monetization: {
    conversionRate: { target: 3, minimum: 1.5, unit: '%' },
    arppu: { target: 15, minimum: 8, unit: 'USD' },
    arpdau: { target: 0.15, minimum: 0.08, unit: 'USD' }
  },

  technical: {
    crashRate: { target: 0.1, maximum: 0.5, unit: '%' },
    matchCompletionRate: { target: 98, minimum: 95, unit: '%' },
    serverUptime: { target: 99.9, minimum: 99.5, unit: '%' }
  }
};
```

### Daily Metrics Review

```typescript
const DAILY_REVIEW = {
  time: '09:00 local (studio time)',
  attendees: ['Producer', 'Analytics Lead', 'Engineering Lead', 'Design Lead'],
  duration: 30,

  agenda: {
    kpiSnapshot: {
      metrics: ['DAU', 'Revenue', 'Retention', 'Crash rate'],
      comparison: 'vs yesterday, vs target'
    },
    issuesReview: {
      bugs: 'New critical/high bugs',
      feedback: 'Top player complaints',
      anomalies: 'Unexpected data patterns'
    },
    actionItems: {
      hotfixes: 'Urgent fixes needed',
      investigations: 'Issues to dig into',
      experiments: 'A/B test status'
    }
  },

  outputs: {
    dailyReport: 'Shared with full team',
    actionLog: 'Tracked in project management tool',
    escalations: 'Flagged to leadership if needed'
  }
};
```

### Weekly Deep Dive

```typescript
const WEEKLY_REVIEW = {
  day: 'Monday',
  time: '14:00',
  attendees: ['Full leads team', 'Studio leadership'],
  duration: 60,

  agenda: {
    weekOverWeek: {
      trends: 'All KPI trends over past week',
      cohortAnalysis: 'Retention curves by cohort',
      funnelAnalysis: 'FTUE, monetization funnels'
    },
    qualitative: {
      reviews: 'App store reviews summary',
      socialMentions: 'Discord/social feedback',
      supportTickets: 'Common issues'
    },
    strategy: {
      priorities: 'Next week focus areas',
      experiments: 'A/B tests to run',
      content: 'Updates planned'
    }
  }
};
```

---

## Rapid Iteration

### Hotfix Process

```typescript
const HOTFIX_PROCESS = {
  trigger: {
    crashRateSpike: '> 1% crash rate',
    criticalBug: 'Blocker or exploit discovered',
    dataLoss: 'Any player data affected',
    serverOutage: 'Widespread connectivity issues'
  },

  timeline: {
    identification: '< 1 hour from report',
    triage: '< 2 hours',
    fixDevelopment: '< 4 hours for critical',
    testing: '< 2 hours',
    deployment: '< 1 hour after approval'
  },

  approval: {
    required: ['Engineering Lead', 'QA Lead', 'Producer'],
    expedited: 'For P0 issues, single lead approval'
  },

  rollout: {
    staged: 'If non-critical, phased rollout',
    immediate: 'If critical, 100% rollout'
  }
};
```

### Update Cadence

```typescript
const UPDATE_CADENCE = {
  hotfixes: {
    frequency: 'As needed',
    content: 'Bug fixes only',
    approval: 'Expedited review'
  },

  weeklyUpdates: {
    frequency: 'Every Thursday',
    content: [
      'Bug fixes accumulated',
      'Balance adjustments',
      'Small improvements'
    ],
    cutoff: 'Tuesday EOD for inclusion'
  },

  majorUpdates: {
    frequency: 'Bi-weekly to monthly',
    content: [
      'New features',
      'Content additions',
      'Significant changes'
    ],
    planning: 'Sprint-aligned'
  }
};
```

### A/B Testing

```typescript
const AB_TESTING = {
  framework: {
    tool: 'Firebase Remote Config / custom',
    segmentation: 'By install cohort',
    analysis: 'Statistical significance required'
  },

  priorityTests: {
    ftue: {
      hypothesis: 'Shorter tutorial improves D1',
      variants: ['Full tutorial', 'Abbreviated', 'Skip option'],
      metric: 'D1 retention'
    },
    monetization: {
      hypothesis: 'Earlier shop prompt increases conversion',
      variants: ['After tutorial', 'After 3 matches', 'After 24h'],
      metric: 'D7 conversion rate'
    },
    economy: {
      hypothesis: 'Adjusted rewards improve session length',
      variants: ['Base rewards', '+20% gold', '+20% XP'],
      metric: 'Matches per session'
    }
  },

  process: {
    duration: 'Minimum 1 week per test',
    sampleSize: 'Minimum 1000 users per variant',
    analysis: 'Weekly review of results',
    graduation: 'Winner becomes default'
  }
};
```

---

## Bug Management

### Triage Process

```typescript
const BUG_TRIAGE = {
  sources: [
    'Crash reporting (Firebase Crashlytics)',
    'Player reports (support/Discord)',
    'App store reviews',
    'Internal QA',
    'Analytics anomalies'
  ],

  severity: {
    p0: {
      definition: 'Crashes, data loss, exploits',
      response: 'Immediate hotfix',
      sla: '< 24 hours'
    },
    p1: {
      definition: 'Major feature broken',
      response: 'Next update',
      sla: '< 1 week'
    },
    p2: {
      definition: 'Minor issues',
      response: 'Scheduled fix',
      sla: '< 2 weeks'
    },
    p3: {
      definition: 'Polish, nice to fix',
      response: 'Backlog',
      sla: 'Before global launch'
    }
  },

  tracking: {
    tool: 'Jira / Linear',
    fields: ['Severity', 'Source', 'Repro steps', 'Affected users', 'Status'],
    dashboard: 'Bug burn-down visible'
  }
};
```

---

## Go/No-Go Decision

### Decision Criteria

```typescript
const GO_NO_GO_CRITERIA = {
  greenLight: {
    retention: 'D1 >= 35%, D7 >= 12%',
    monetization: 'Conversion >= 2%, ARPU >= $10',
    technical: 'Crash rate < 0.3%, uptime > 99.5%',
    sentiment: 'App store rating >= 4.0'
  },

  yellowLight: {
    description: 'Metrics close but not met',
    action: 'Extend soft launch, iterate',
    duration: 'Additional 2-4 weeks',
    focus: 'Address specific weak areas'
  },

  redLight: {
    description: 'Fundamental issues identified',
    action: 'Pause and reassess',
    triggers: [
      'D1 < 25%',
      'Crash rate > 2%',
      'Critical negative feedback',
      'Core loop not engaging'
    ],
    recovery: 'Major iteration before retry'
  }
};
```

### Decision Meeting

```typescript
const DECISION_MEETING = {
  timing: 'End of soft launch period (week 4-6)',
  attendees: [
    'Studio Head',
    'Producer',
    'All domain leads',
    'Analytics Lead',
    'Marketing Lead'
  ],

  presentation: {
    metricsReview: 'All KPIs vs targets',
    trendsAnalysis: 'Week-over-week improvements',
    qualitativeFeedback: 'Player sentiment summary',
    competitiveContext: 'Market positioning',
    riskAssessment: 'Outstanding issues and risks'
  },

  outcomes: {
    proceedToGlobal: 'Targets met, launch globally',
    extendSoftLaunch: 'Iterate further before decision',
    pivotOrPause: 'Major changes needed'
  },

  documentation: {
    decisionRationale: 'Written summary of decision',
    lessonsLearned: 'Key insights documented',
    actionPlan: 'Next steps with owners'
  }
};
```

---

## Operations

### On-Call Rotation

```typescript
const ON_CALL = {
  coverage: '24/7 during soft launch',

  rotation: {
    primary: 'Weekly rotation among engineers',
    backup: 'Secondary on-call if escalation',
    leadership: 'Producer available for decisions'
  },

  responsibilities: {
    monitoring: 'Check dashboards every 2 hours',
    response: 'Acknowledge alerts within 15 minutes',
    escalation: 'Wake up team for P0 issues',
    communication: 'Update status page if outage'
  },

  tools: {
    alerting: 'PagerDuty / Opsgenie',
    communication: 'Slack #incidents channel',
    runbooks: 'Documented response procedures'
  }
};
```

### Incident Response

```typescript
const INCIDENT_RESPONSE = {
  severity: {
    sev1: {
      definition: 'Complete outage, widespread impact',
      response: 'All hands, war room',
      communication: 'Every 30 minutes update'
    },
    sev2: {
      definition: 'Major feature down, significant impact',
      response: 'On-call + backup',
      communication: 'Hourly updates'
    },
    sev3: {
      definition: 'Minor degradation, limited impact',
      response: 'On-call handles',
      communication: 'As resolved'
    }
  },

  process: {
    detect: 'Alert fired or report received',
    acknowledge: 'On-call takes ownership',
    diagnose: 'Identify root cause',
    mitigate: 'Restore service',
    resolve: 'Permanent fix deployed',
    postmortem: 'Blameless review within 48h'
  }
};
```

---

## Marketing (Limited)

### Organic Focus

```typescript
const SOFT_LAUNCH_MARKETING = {
  approach: 'Primarily organic',

  activities: {
    appStoreOptimization: {
      enabled: true,
      scope: 'Basic ASO for soft launch markets'
    },
    socialMedia: {
      enabled: false,
      reason: 'Avoid attention before ready'
    },
    paidUA: {
      enabled: 'Limited testing only',
      budget: '< $5,000 total',
      purpose: 'Validate UA channels'
    },
    influencers: {
      enabled: false,
      reason: 'Save for global launch'
    }
  },

  uaTests: {
    channels: ['Facebook', 'Google UAC'],
    spend: '$500-1000 per channel',
    goal: 'Understand CPI, validate creatives',
    learning: 'Inform global launch strategy'
  }
};
```

---

## Documentation

### Required Documentation

```typescript
const DOCUMENTATION = {
  dailyReports: {
    content: 'KPIs, issues, actions',
    storage: 'Shared drive / Notion',
    retention: 'Permanent'
  },

  weeklyReports: {
    content: 'Comprehensive metrics review',
    distribution: 'Leadership, stakeholders',
    format: 'Slide deck + written summary'
  },

  incidentReports: {
    content: 'Postmortem for any Sev1/Sev2',
    timeline: 'Within 48 hours of resolution',
    sharing: 'Full team for learning'
  },

  lessonsLearned: {
    content: 'Key insights from soft launch',
    timing: 'End of soft launch period',
    usage: 'Inform global launch planning'
  }
};
```

---

## Timeline

### Soft Launch Schedule

```typescript
const TIMELINE = {
  week1: {
    focus: 'Launch and stabilize',
    goals: [
      'Successful launch in all markets',
      'No critical issues',
      'Baseline metrics established'
    ]
  },

  week2: {
    focus: 'First iteration',
    goals: [
      'First update deployed',
      'Top bugs fixed',
      'Initial balance adjustments'
    ]
  },

  week3_4: {
    focus: 'Optimization',
    goals: [
      'A/B tests running',
      'Retention improvements',
      'Monetization tuning'
    ]
  },

  week5_6: {
    focus: 'Decision preparation',
    goals: [
      'Metrics stabilized',
      'Go/no-go data gathered',
      'Global launch planning if green'
    ]
  },

  extension: {
    trigger: 'Yellow light decision',
    duration: '2-4 additional weeks',
    focus: 'Address specific gaps'
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial soft launch execution specification |
