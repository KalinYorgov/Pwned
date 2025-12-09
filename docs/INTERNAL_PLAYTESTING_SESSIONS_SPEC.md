# Internal Playtesting Sessions Specification

## Document Information
- **Task ID:** PROJ-006
- **Priority:** P0
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Internal Playtesting Sessions process for Plunderstorm Mobile, establishing a structured approach to regular playtesting, feedback collection, iteration cycles, and quality improvement throughout development.

---

## Playtest Program Structure

### Session Cadence

```typescript
const PLAYTEST_CADENCE = {
  regular: {
    frequency: 'Weekly',
    day: 'Thursday',
    time: '14:00-17:00 local time',
    duration: 180
  },

  intensive: {
    trigger: 'Pre-milestone or major feature',
    frequency: 'Daily',
    duration: 120
  },

  adhoc: {
    trigger: 'Critical bug fix or feature completion',
    notice: 'Same day or next day',
    duration: 60
  },

  minimumSessions: {
    beforeBeta: 10,
    perMilestone: 3,
    perMajorFeature: 2
  }
};
```

### Participant Requirements

```typescript
const PARTICIPANTS = {
  mandatory: {
    roles: [
      'Game Designer',
      'Lead Programmer',
      'QA Lead',
      'Producer'
    ],
    minimum: 4
  },

  rotating: {
    roles: [
      'Engineers',
      'Artists',
      'Audio Designer',
      'UX Designer'
    ],
    perSession: '2-3 additional'
  },

  external: {
    frequency: 'Monthly',
    participants: 'Friends/family under NDA',
    purpose: 'Fresh perspective'
  },

  totalPerSession: {
    minimum: 6,
    optimal: 10,
    maximum: 20
  }
};
```

---

## Session Structure

### Pre-Session Preparation

```typescript
const PRE_SESSION = {
  buildPreparation: {
    timing: 'Day before playtest',
    tasks: [
      'Create stable playtest build',
      'Deploy to test servers',
      'Verify all features functional',
      'Document known issues',
      'Prepare build notes'
    ]
  },

  focusAreas: {
    defined: true,
    examples: [
      'New ability balance',
      'Storm pacing',
      'Control responsiveness',
      'Specific bug verification'
    ],
    communication: 'Sent to participants 2 hours before'
  },

  environmentSetup: {
    servers: 'Dedicated playtest server instance',
    devices: 'Range of target devices available',
    recording: 'Screen recording enabled',
    analytics: 'Debug telemetry active'
  }
};
```

### Session Flow

```typescript
const SESSION_FLOW = {
  briefing: {
    duration: 15,
    activities: [
      'Review build changes since last test',
      'Explain focus areas',
      'Assign observation roles',
      'Technical setup verification'
    ]
  },

  warmup: {
    duration: 15,
    activities: [
      'Free exploration',
      'Control familiarization',
      'Initial impressions noted'
    ]
  },

  structuredPlay: {
    duration: 90,
    format: {
      matchCount: '4-6 full matches',
      teamComposition: 'Mixed skill levels',
      scenarios: [
        'Normal gameplay',
        'Specific feature testing',
        'Edge case exploration'
      ]
    }
  },

  breakAndNotes: {
    duration: 15,
    activities: [
      'Individual note-taking',
      'Bug report filing',
      'Quick discussions'
    ]
  },

  debrief: {
    duration: 45,
    activities: [
      'Round-table feedback',
      'Priority voting',
      'Action item assignment',
      'Next session planning'
    ]
  }
};
```

---

## Feedback Collection

### Feedback Categories

```typescript
const FEEDBACK_CATEGORIES = {
  gameplay: {
    areas: [
      'Core loop satisfaction',
      'Ability balance',
      'Combat feel',
      'Pacing',
      'Win/loss satisfaction'
    ],
    rating: '1-5 scale'
  },

  controls: {
    areas: [
      'Movement responsiveness',
      'Camera smoothness',
      'Ability targeting',
      'Button layout',
      'Overall comfort'
    ],
    rating: '1-5 scale'
  },

  technical: {
    areas: [
      'Performance/FPS',
      'Network stability',
      'Visual quality',
      'Audio quality',
      'Bug frequency'
    ],
    rating: '1-5 scale'
  },

  funFactor: {
    question: 'How much fun did you have?',
    rating: '1-10 scale',
    followUp: 'What would make it more fun?'
  }
};
```

### Feedback Forms

```typescript
const FEEDBACK_FORMS = {
  postMatch: {
    timing: 'After each match',
    duration: '1-2 minutes',
    fields: [
      { name: 'enjoyment', type: 'rating', scale: '1-5' },
      { name: 'frustrations', type: 'multiSelect', options: ['Controls', 'Balance', 'Bugs', 'Performance', 'Other'] },
      { name: 'highlights', type: 'text', optional: true }
    ]
  },

  postSession: {
    timing: 'End of playtest',
    duration: '5-10 minutes',
    fields: [
      { name: 'overallRating', type: 'rating', scale: '1-10' },
      { name: 'topIssues', type: 'ranking', count: 3 },
      { name: 'topPositives', type: 'ranking', count: 3 },
      { name: 'suggestions', type: 'text' },
      { name: 'wouldPlayAgain', type: 'boolean' }
    ]
  },

  bugReport: {
    timing: 'As encountered',
    fields: [
      { name: 'severity', type: 'select', options: ['Blocker', 'Critical', 'Major', 'Minor'] },
      { name: 'category', type: 'select', options: ['Gameplay', 'UI', 'Performance', 'Network', 'Visual', 'Audio'] },
      { name: 'reproduction', type: 'text' },
      { name: 'expected', type: 'text' },
      { name: 'actual', type: 'text' },
      { name: 'screenshot', type: 'file', optional: true }
    ]
  }
};
```

---

## Data Tracking

### Balance Spreadsheet

```typescript
const BALANCE_TRACKING = {
  abilities: {
    metrics: [
      'Pick rate',
      'Win rate when equipped',
      'Damage per use',
      'Usage frequency',
      'Player satisfaction rating'
    ],
    updateFrequency: 'After each playtest'
  },

  match: {
    metrics: [
      'Average match duration',
      'Eliminations per match',
      'Storm deaths vs combat deaths',
      'Loot distribution fairness',
      'Final circle fight frequency'
    ]
  },

  progression: {
    metrics: [
      'Time to first elimination',
      'Survival curve',
      'Skill gap observations',
      'New player performance'
    ]
  }
};
```

### Trend Analysis

```typescript
const TREND_ANALYSIS = {
  weekOverWeek: {
    tracked: [
      'Overall fun rating',
      'Bug count',
      'Performance scores',
      'Control satisfaction',
      'Return intent'
    ],
    visualization: 'Line chart per metric'
  },

  issueRecurrence: {
    tracked: 'Issues reported multiple sessions',
    threshold: '3+ sessions = priority escalation',
    review: 'Weekly trend meeting'
  },

  improvementValidation: {
    process: 'Compare before/after scores for changes',
    requirement: 'Measurable improvement expected'
  }
};
```

---

## Bug Management

### Bug Triage Process

```typescript
const BUG_TRIAGE = {
  timing: 'Same day as playtest',
  participants: ['QA Lead', 'Lead Programmer', 'Producer'],

  classification: {
    blocker: {
      definition: 'Prevents testing or causes crashes',
      response: 'Fix before next playtest',
      sla: '24 hours'
    },
    critical: {
      definition: 'Major gameplay impact',
      response: 'Prioritize for next sprint',
      sla: '1 week'
    },
    major: {
      definition: 'Noticeable but workaround exists',
      response: 'Schedule fix',
      sla: '2 weeks'
    },
    minor: {
      definition: 'Polish issues',
      response: 'Backlog',
      sla: 'Before launch'
    }
  },

  tracking: {
    tool: 'Issue tracker (Jira/Linear/GitHub)',
    fields: ['Severity', 'Component', 'Assignee', 'Session Found', 'Status'],
    dashboard: 'Bug burn-down visible to team'
  }
};
```

### Resolution Verification

```typescript
const VERIFICATION = {
  fixedBugs: {
    process: 'Re-test in next playtest',
    criteria: 'Bug not reproducible',
    status: 'Move to Verified/Closed'
  },

  regressions: {
    detection: 'Previously fixed bug reappears',
    priority: 'Escalate to Critical',
    prevention: 'Add to regression test suite'
  },

  notFixed: {
    process: 'Reopen with new information',
    escalation: 'If 3+ attempts, escalate'
  }
};
```

---

## Playtest Reports

### Session Report Template

```typescript
const SESSION_REPORT = {
  header: {
    date: 'Session date',
    build: 'Build version',
    participants: 'List of testers',
    duration: 'Actual session length'
  },

  summary: {
    overallRating: 'Average fun score',
    matchesPlayed: 'Number of matches',
    bugsFound: 'Count by severity',
    keyFindings: '3-5 bullet points'
  },

  detailedFeedback: {
    positives: 'What worked well',
    negatives: 'What needs improvement',
    suggestions: 'Player ideas'
  },

  metrics: {
    balanceData: 'Ability statistics',
    performanceData: 'FPS, network stats',
    behaviorData: 'Player actions analysis'
  },

  actionItems: {
    format: 'Issue | Owner | Priority | Target Date',
    tracking: 'Linked to issue tracker'
  },

  nextSession: {
    date: 'Scheduled date',
    focusAreas: 'Planned testing focus'
  }
};
```

### Cumulative Report

```typescript
const CUMULATIVE_REPORT = {
  frequency: 'Monthly',

  contents: {
    sessionSummary: 'All sessions in period',
    trendCharts: 'Metrics over time',
    bugStatistics: 'Found vs fixed',
    milestoneProgress: 'Quality gate status',
    recommendations: 'Strategic suggestions'
  },

  audience: 'Leadership, stakeholders',
  format: 'Slide deck + written summary'
};
```

---

## Footage and Recording

### Capture Requirements

```typescript
const RECORDING = {
  gameplay: {
    format: '1080p 30fps minimum',
    storage: 'Cloud storage per session',
    retention: '30 days minimum',
    highlights: 'Clip notable moments'
  },

  bugs: {
    requirement: 'Record reproduction steps',
    attachment: 'Link to bug report',
    format: 'MP4 or GIF'
  },

  presentations: {
    purpose: 'Stakeholder demos',
    curation: 'Best moments per milestone',
    editing: 'Light editing for clarity'
  }
};
```

### Analysis Tools

```typescript
const ANALYSIS_TOOLS = {
  heatmaps: {
    tracked: ['Death locations', 'Popular routes', 'Loot spots'],
    generation: 'Post-session processing',
    usage: 'Map balance analysis'
  },

  telemetry: {
    realtime: ['FPS', 'Network latency', 'Memory usage'],
    aggregate: ['Session statistics', 'Player behaviors'],
    dashboard: 'Live during playtest'
  },

  replays: {
    availability: 'If replay system implemented',
    usage: 'Review specific incidents'
  }
};
```

---

## Iteration Process

### Fix-Test-Verify Cycle

```typescript
const ITERATION_CYCLE = {
  identify: {
    source: 'Playtest feedback',
    prioritization: 'Severity + frequency + impact',
    assignment: 'Owner designated'
  },

  fix: {
    timeline: 'Before next playtest',
    verification: 'Developer self-test',
    review: 'Code review if applicable'
  },

  test: {
    method: 'Include in next playtest',
    focus: 'Specific testing of fix',
    comparison: 'Before/after metrics'
  },

  verify: {
    criteria: 'Issue no longer reported',
    metrics: 'Improvement in scores',
    closure: 'Update issue status'
  }
};
```

### Balance Iteration

```typescript
const BALANCE_ITERATION = {
  dataCollection: {
    duration: '2-3 sessions minimum',
    requirement: 'Statistical significance'
  },

  analysis: {
    outliers: 'Identify over/under performing elements',
    rootCause: 'Why is this unbalanced?',
    proposal: 'Specific number changes'
  },

  implementation: {
    approach: 'Small incremental changes',
    documentation: 'Record all balance changes',
    rollback: 'Ability to revert if worse'
  },

  validation: {
    sessions: 'Test with updated values',
    target: 'Metrics within acceptable range',
    iteration: 'Repeat if needed'
  }
};
```

---

## Success Metrics

### Quality Improvement Tracking

```typescript
const SUCCESS_METRICS = {
  funScore: {
    baseline: 'First playtest average',
    target: '8+ out of 10',
    trend: 'Improving week over week'
  },

  bugCount: {
    openBlockers: 0,
    openCritical: '< 5',
    trend: 'Decreasing over time'
  },

  returnIntent: {
    question: 'Would you play again?',
    target: '> 90% yes',
    tracking: 'Per session'
  },

  sessionEngagement: {
    metric: 'Players stay full session',
    target: '> 80%',
    indicator: 'Game is engaging'
  }
};
```

### Exit Criteria

```typescript
const EXIT_CRITERIA = {
  minimumSessions: 10,

  qualityGates: {
    funScore: '>= 7.5 average last 3 sessions',
    blockerBugs: 0,
    criticalBugs: '< 3 open',
    returnIntent: '>= 85%'
  },

  teamConfidence: {
    requirement: 'Unanimous agreement to proceed',
    vote: 'Go/No-Go meeting'
  },

  documentation: {
    allReportsComplete: true,
    balanceSpreadsheetCurrent: true,
    bugBacklogManaged: true
  }
};
```

---

## Tools and Infrastructure

### Required Tools

```typescript
const TOOLS = {
  buildDistribution: {
    ios: 'TestFlight',
    android: 'Firebase App Distribution',
    internal: 'Direct APK/IPA sharing'
  },

  feedbackCollection: {
    forms: 'Google Forms / Typeform',
    realtime: 'Slack channel',
    structured: 'Issue tracker'
  },

  communication: {
    scheduling: 'Calendar invites',
    reminders: 'Slack notifications',
    documentation: 'Confluence / Notion'
  },

  analytics: {
    telemetry: 'Custom dashboard',
    video: 'Cloud recording storage',
    spreadsheet: 'Google Sheets / Excel'
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial playtest sessions specification |
