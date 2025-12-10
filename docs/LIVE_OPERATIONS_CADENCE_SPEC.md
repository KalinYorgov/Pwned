# Live Operations Cadence Specification

## Document Information
- **Task ID:** PROJ-013
- **Priority:** P0
- **Complexity:** Large
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Live Operations Cadence for Plunderstorm Mobile, establishing the ongoing operational rhythm, team responsibilities, content schedules, and processes required to sustain and grow the game post-launch.

---

## Operational Rhythm

### Daily Operations

```typescript
const DAILY_OPS = {
  morning: {
    time: '09:00',
    activities: [
      {
        task: 'Metrics Review',
        owner: 'Analytics Lead',
        duration: 15,
        outputs: 'Daily metrics summary'
      },
      {
        task: 'Issue Triage',
        owner: 'QA Lead',
        duration: 15,
        outputs: 'Priority bug list'
      },
      {
        task: 'Support Review',
        owner: 'Support Lead',
        duration: 15,
        outputs: 'Top issues summary'
      }
    ]
  },

  standup: {
    time: '09:30',
    duration: 15,
    attendees: ['Producer', 'Leads'],
    agenda: [
      'Overnight metrics',
      'Critical issues',
      'Day priorities',
      'Blockers'
    ]
  },

  ongoing: {
    monitoring: {
      frequency: 'Continuous',
      owner: 'On-call engineer',
      scope: 'Server health, error rates'
    },
    support: {
      frequency: 'Continuous',
      owner: 'Support team',
      scope: 'Player tickets, reviews'
    },
    social: {
      frequency: '3x per day',
      owner: 'Community Manager',
      scope: 'Social posts, engagement'
    }
  },

  evening: {
    time: '17:00',
    activities: [
      {
        task: 'EOD Summary',
        owner: 'Producer',
        content: 'Key updates, overnight notes'
      },
      {
        task: 'On-call Handoff',
        owner: 'Engineering',
        content: 'Status, known issues'
      }
    ]
  }
};
```

### Weekly Operations

```typescript
const WEEKLY_OPS = {
  monday: {
    name: 'Planning Day',
    activities: [
      {
        task: 'Week Planning Meeting',
        time: '10:00',
        duration: 60,
        attendees: 'All leads',
        agenda: [
          'Previous week review',
          'This week priorities',
          'Resource allocation',
          'Risk assessment'
        ]
      },
      {
        task: 'Content Queue Review',
        time: '14:00',
        duration: 30,
        owner: 'Content Team',
        scope: 'Upcoming content status'
      }
    ]
  },

  tuesday: {
    name: 'Development Day',
    activities: [
      {
        task: 'Feature Development',
        scope: 'Sprint work'
      },
      {
        task: 'Store Rotation Prep',
        owner: 'LiveOps',
        scope: 'Prepare weekly shop update'
      }
    ]
  },

  wednesday: {
    name: 'Development Day',
    activities: [
      {
        task: 'Feature Development',
        scope: 'Sprint work'
      },
      {
        task: 'QA Testing',
        owner: 'QA Team',
        scope: 'Weekly update validation'
      }
    ]
  },

  thursday: {
    name: 'Release Day',
    activities: [
      {
        task: 'Weekly Update Deploy',
        time: '10:00',
        owner: 'Engineering',
        scope: 'Bug fixes, balance updates'
      },
      {
        task: 'Store Rotation',
        time: '00:00 UTC',
        owner: 'LiveOps',
        scope: 'Shop content refresh'
      },
      {
        task: 'LTM Rotation',
        time: '00:00 UTC',
        owner: 'LiveOps',
        scope: 'Game mode rotation'
      }
    ]
  },

  friday: {
    name: 'Review Day',
    activities: [
      {
        task: 'Week Retrospective',
        time: '15:00',
        duration: 30,
        attendees: 'All leads',
        agenda: ['What worked', 'What didn\'t', 'Improvements']
      },
      {
        task: 'Weekend Prep',
        owner: 'On-call',
        scope: 'Ensure stability for weekend'
      }
    ]
  },

  weekend: {
    name: 'Monitoring Only',
    activities: [
      {
        task: 'On-call Monitoring',
        owner: 'On-call engineer',
        scope: 'Critical issues only'
      },
      {
        task: 'Community Engagement',
        owner: 'Community Manager',
        scope: 'Social, Discord activity'
      }
    ]
  }
};
```

### Bi-Weekly Operations

```typescript
const BIWEEKLY_OPS = {
  sprint: {
    duration: '2 weeks',
    ceremonies: {
      planning: {
        when: 'Sprint Day 1',
        duration: 120,
        output: 'Sprint backlog'
      },
      review: {
        when: 'Sprint Day 10',
        duration: 60,
        output: 'Demo of completed work'
      },
      retrospective: {
        when: 'Sprint Day 10',
        duration: 45,
        output: 'Process improvements'
      }
    }
  },

  updates: {
    minorUpdate: {
      frequency: 'Every 2 weeks',
      content: [
        'Bug fixes',
        'Balance adjustments',
        'Minor features',
        'Quality of life improvements'
      ],
      process: {
        codeFreeze: 'Day 8',
        qa: 'Days 9-10',
        release: 'Day 11 (Thursday)'
      }
    }
  },

  reviews: {
    metricsDeepDive: {
      frequency: 'Bi-weekly',
      duration: 90,
      attendees: ['Producer', 'Analytics', 'Design', 'Monetization'],
      content: [
        'Retention trends',
        'Revenue analysis',
        'Feature performance',
        'A/B test results'
      ]
    }
  }
};
```

### Monthly Operations

```typescript
const MONTHLY_OPS = {
  majorUpdate: {
    frequency: 'Monthly',
    content: [
      'New abilities (1-2)',
      'New cosmetics batch',
      'New features',
      'Major balance patch',
      'Event content'
    ],
    timeline: {
      week1: 'Development',
      week2: 'Development + Internal testing',
      week3: 'QA + Polish',
      week4: 'Release + Monitor'
    }
  },

  reviews: {
    monthlyBusinessReview: {
      when: 'First Monday of month',
      duration: 120,
      attendees: ['Leadership', 'All leads'],
      agenda: [
        'KPI review vs targets',
        'Revenue and spend',
        'User acquisition performance',
        'Competitive landscape',
        'Next month planning'
      ]
    },
    contentReview: {
      when: 'Second Monday of month',
      duration: 60,
      attendees: ['Content team', 'Design', 'Art'],
      agenda: [
        'Content performance',
        'Upcoming content pipeline',
        'Resource needs'
      ]
    }
  },

  planning: {
    nextMonthPlanning: {
      when: 'Third week of month',
      output: 'Next month roadmap finalized'
    }
  }
};
```

### Seasonal Operations

```typescript
const SEASONAL_OPS = {
  battlePass: {
    frequency: '6-8 weeks',
    timeline: {
      week_minus_8: 'Theme and rewards finalized',
      week_minus_6: 'Art production begins',
      week_minus_4: 'Implementation begins',
      week_minus_2: 'QA and polish',
      week_minus_1: 'Marketing prep',
      week_0: 'Season launch'
    },
    content: {
      tiers: 100,
      freeRewards: 30,
      premiumRewards: 70,
      exclusiveSkin: 1
    }
  },

  majorEvents: {
    frequency: 'Quarterly',
    types: [
      { name: 'Summer Splash', timing: 'June-July' },
      { name: 'Halloween Horror', timing: 'October' },
      { name: 'Winter Wonderland', timing: 'December' },
      { name: 'Anniversary', timing: 'Launch month' }
    ],
    timeline: {
      planning: '8 weeks before',
      production: '6 weeks before',
      qa: '2 weeks before',
      launch: 'Event start',
      duration: '2-4 weeks'
    }
  },

  rankedSeasons: {
    frequency: '8 weeks',
    reset: 'Soft reset of MMR',
    rewards: 'Based on peak rank',
    communication: '1 week notice before reset'
  }
};
```

---

## Team Roles and Responsibilities

### Live Operations Team

```typescript
const LIVEOPS_TEAM = {
  producer: {
    title: 'Live Operations Producer',
    responsibilities: [
      'Overall live ops coordination',
      'Schedule management',
      'Stakeholder communication',
      'Priority decisions',
      'Team health'
    ],
    time: '100%'
  },

  liveOpsManager: {
    title: 'LiveOps Manager',
    responsibilities: [
      'Content scheduling',
      'Store management',
      'Event execution',
      'Remote config updates',
      'Data analysis'
    ],
    time: '100%'
  },

  communityManager: {
    title: 'Community Manager',
    responsibilities: [
      'Social media management',
      'Discord moderation',
      'Player communication',
      'Feedback collection',
      'Content creator relations'
    ],
    time: '100%'
  },

  supportLead: {
    title: 'Support Lead',
    responsibilities: [
      'Support team management',
      'Escalation handling',
      'FAQ maintenance',
      'Policy decisions',
      'Review responses'
    ],
    time: '100%'
  }
};
```

### Engineering Support

```typescript
const ENGINEERING_SUPPORT = {
  onCall: {
    title: 'On-Call Engineer',
    rotation: 'Weekly',
    responsibilities: [
      'Monitor alerts',
      'First response to incidents',
      'Hotfix deployment',
      'Escalation as needed'
    ],
    coverage: '24/7'
  },

  liveEngineer: {
    title: 'Live Services Engineer',
    responsibilities: [
      'Server maintenance',
      'Performance monitoring',
      'Infrastructure updates',
      'Capacity planning'
    ],
    time: '100%'
  },

  releaseEngineer: {
    title: 'Release Engineer',
    responsibilities: [
      'Build management',
      'Deployment automation',
      'Release coordination',
      'Rollback capability'
    ],
    time: '50%'
  }
};
```

---

## Runbooks

### Store Rotation

```typescript
const STORE_ROTATION_RUNBOOK = {
  frequency: 'Weekly (Thursday 00:00 UTC)',
  owner: 'LiveOps Manager',

  preparation: {
    timing: 'Tuesday',
    steps: [
      'Select items for rotation',
      'Set pricing and discounts',
      'Prepare featured item banner',
      'Configure in CMS/admin tool',
      'QA verify in staging'
    ]
  },

  execution: {
    timing: 'Thursday 00:00 UTC',
    steps: [
      'Verify previous rotation ended',
      'Activate new rotation',
      'Verify items appear correctly',
      'Check pricing accuracy',
      'Monitor for issues'
    ]
  },

  rollback: {
    trigger: 'Pricing error or missing items',
    steps: [
      'Disable problematic items',
      'Revert to previous rotation',
      'Investigate and fix',
      'Communicate if needed'
    ]
  }
};
```

### LTM Rotation

```typescript
const LTM_ROTATION_RUNBOOK = {
  frequency: 'Weekly',
  owner: 'LiveOps Manager',

  schedule: {
    rotation: [
      { week: 1, mode: 'Swords Only' },
      { week: 2, mode: 'Chaos Mode' },
      { week: 3, mode: 'Duo Blitz' },
      { week: 4, mode: 'Boss Rush' }
    ],
    cycle: 'Repeating with seasonal variations'
  },

  execution: {
    steps: [
      'Update game config for new mode',
      'Verify mode selection screen',
      'Test mode functionality',
      'Update promotional messaging',
      'Monitor queue times'
    ]
  },

  specialEvents: {
    trigger: 'Holiday or special occasion',
    process: 'Custom LTM outside rotation',
    approval: 'Producer approval required'
  }
};
```

### Hotfix Deployment

```typescript
const HOTFIX_RUNBOOK = {
  trigger: 'Critical bug requiring immediate fix',
  owner: 'On-Call Engineer',

  assessment: {
    steps: [
      'Confirm severity (P0/P1)',
      'Identify affected players',
      'Determine fix approach',
      'Estimate fix time'
    ],
    approval: 'Engineering Lead for P0'
  },

  development: {
    steps: [
      'Create hotfix branch',
      'Implement fix',
      'Self-test fix',
      'Code review (expedited)',
      'QA smoke test'
    ],
    timeline: '< 4 hours for critical'
  },

  deployment: {
    steps: [
      'Build release candidate',
      'Deploy to staging',
      'Final verification',
      'Deploy to production',
      'Monitor for 30 minutes',
      'Confirm fix effective'
    ]
  },

  communication: {
    internal: 'Slack #incidents',
    external: 'Status page if visible impact',
    postmortem: 'Required for all P0'
  }
};
```

### Server Scaling

```typescript
const SCALING_RUNBOOK = {
  automatic: {
    trigger: 'CPU > 70% or CCU threshold',
    action: 'Autoscaler adds instances',
    monitoring: 'Verify new instances healthy'
  },

  manual: {
    trigger: 'Planned event or predicted load',
    steps: [
      'Calculate required capacity',
      'Update autoscaler limits',
      'Pre-warm additional instances',
      'Verify capacity available',
      'Monitor during event'
    ],
    timing: '2 hours before expected load'
  },

  scaleDown: {
    trigger: 'Load decreased',
    delay: '30 minutes after threshold',
    steps: [
      'Autoscaler removes instances',
      'Verify player experience stable',
      'Update capacity notes'
    ]
  }
};
```

---

## Escalation Paths

### Technical Escalation

```typescript
const TECHNICAL_ESCALATION = {
  level1: {
    owner: 'On-Call Engineer',
    scope: 'Known issues, minor fixes',
    response: '< 15 minutes'
  },

  level2: {
    owner: 'Engineering Lead',
    trigger: 'Complex issues, decisions needed',
    response: '< 30 minutes'
  },

  level3: {
    owner: 'CTO / Technical Director',
    trigger: 'Major outage, architectural decisions',
    response: '< 1 hour'
  },

  external: {
    owner: 'Vendor contacts',
    trigger: 'Third-party service issues',
    contacts: 'AWS, Firebase, etc.'
  }
};
```

### Business Escalation

```typescript
const BUSINESS_ESCALATION = {
  level1: {
    owner: 'LiveOps Manager',
    scope: 'Content issues, minor decisions'
  },

  level2: {
    owner: 'Producer',
    trigger: 'Schedule changes, resource needs'
  },

  level3: {
    owner: 'Studio Head',
    trigger: 'Major decisions, public communications'
  },

  legal: {
    owner: 'Legal counsel',
    trigger: 'Policy violations, legal threats'
  }
};
```

---

## On-Call Rotation

### Schedule

```typescript
const ONCALL_SCHEDULE = {
  rotation: {
    type: 'Weekly',
    handoff: 'Monday 09:00',
    team: '4-6 engineers in rotation'
  },

  coverage: {
    primary: 'First responder',
    secondary: 'Backup if primary unavailable',
    lead: 'Escalation point'
  },

  compensation: {
    weekday: 'Standard pay + on-call stipend',
    weekend: 'On-call stipend + time off',
    incident: 'Additional compensation for callouts'
  }
};
```

### Responsibilities

```typescript
const ONCALL_RESPONSIBILITIES = {
  monitoring: {
    frequency: 'Check every 2 hours minimum',
    dashboards: 'All critical metrics',
    alerts: 'Respond to all pages'
  },

  response: {
    acknowledge: '< 15 minutes',
    diagnose: '< 30 minutes',
    escalate: 'If unable to resolve in 1 hour'
  },

  handoff: {
    documentation: 'Update incident log',
    meeting: '15 minute handoff with next on-call',
    notes: 'Any ongoing issues or concerns'
  }
};
```

---

## Content Calendar

### Annual View

```typescript
const ANNUAL_CALENDAR = {
  q1: {
    battlePass: ['Season 3', 'Season 4'],
    events: ['Anniversary Event'],
    features: ['Ranked Season 2', 'New Abilities'],
    holidays: ['Valentine\'s', 'St. Patrick\'s']
  },

  q2: {
    battlePass: ['Season 5', 'Season 6'],
    events: ['Summer Splash'],
    features: ['Guilds Update', 'New Map Area'],
    holidays: ['Easter', 'Memorial Day']
  },

  q3: {
    battlePass: ['Season 7', 'Season 8'],
    events: ['Back to School'],
    features: ['Tournament Mode', 'Spectator Update'],
    holidays: ['Labor Day']
  },

  q4: {
    battlePass: ['Season 9', 'Season 10'],
    events: ['Halloween Horror', 'Winter Wonderland'],
    features: ['Year 2 Preview'],
    holidays: ['Halloween', 'Thanksgiving', 'Christmas', 'New Year']
  }
};
```

### Monthly Template

```typescript
const MONTHLY_TEMPLATE = {
  week1: {
    content: 'New Battle Pass season (if applicable)',
    store: 'Season launch bundle',
    event: 'Season kickoff challenges'
  },

  week2: {
    content: 'Mid-season update',
    store: 'Featured skin rotation',
    event: 'Community challenge'
  },

  week3: {
    content: 'Balance patch',
    store: 'Throwback items',
    event: 'Weekend event mode'
  },

  week4: {
    content: 'Quality of life update',
    store: 'Last chance items',
    event: 'Season end push'
  }
};
```

---

## Metrics and Reporting

### Daily Report

```typescript
const DAILY_REPORT = {
  timing: '10:00 daily',
  distribution: 'Team Slack + email',

  content: {
    kpis: ['DAU', 'Revenue', 'D1 Retention', 'CCU Peak'],
    comparison: 'vs yesterday, vs last week',
    highlights: 'Notable events or anomalies',
    issues: 'Open P0/P1 bugs'
  }
};
```

### Weekly Report

```typescript
const WEEKLY_REPORT = {
  timing: 'Friday EOD',
  distribution: 'Leadership + stakeholders',

  content: {
    summary: 'Week overview',
    metrics: {
      engagement: 'WAU, sessions, playtime',
      monetization: 'Revenue, ARPU, conversion',
      retention: 'D1, D7 trends',
      technical: 'Uptime, crash rate'
    },
    achievements: 'What shipped',
    challenges: 'Issues faced',
    nextWeek: 'Upcoming priorities'
  }
};
```

---

## Success Metrics

```typescript
const LIVEOPS_SUCCESS = {
  operational: {
    uptime: '> 99.9%',
    scheduleAdherence: '> 95% on-time releases',
    incidentResponse: '< 15 min acknowledgment',
    hotfixTime: '< 4 hours for critical'
  },

  content: {
    releaseFrequency: 'Weekly updates minimum',
    eventExecution: 'All planned events launched',
    qualityScore: '< 5 rollbacks per quarter'
  },

  team: {
    burnout: 'Sustainable pace maintained',
    coverage: 'No gaps in on-call',
    satisfaction: 'Team health surveys positive'
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial live operations cadence specification |
