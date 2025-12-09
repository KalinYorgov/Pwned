# Soft Launch Iteration Specification

## Document Information
- **Task ID:** PROJ-010
- **Priority:** P0
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Soft Launch Iteration process for Plunderstorm Mobile, establishing a structured approach to analyzing soft launch data, implementing targeted improvements, validating changes, and achieving metrics targets required for global launch approval.

---

## Iteration Philosophy

### Core Principles

```typescript
const ITERATION_PRINCIPLES = {
  dataFirst: {
    description: 'All decisions backed by data',
    approach: 'Hypothesize → Test → Measure → Iterate'
  },

  focusedChanges: {
    description: 'One major change at a time',
    rationale: 'Isolate impact of each change',
    exception: 'Bug fixes can be bundled'
  },

  rapidCycles: {
    description: 'Fast iteration loops',
    target: 'Weekly updates minimum',
    goal: 'Maximum learning in soft launch window'
  },

  playerCentric: {
    description: 'Listen to player feedback',
    balance: 'Data + qualitative feedback',
    channels: 'Reviews, support, Discord, surveys'
  }
};
```

---

## Diagnostic Framework

### Retention Diagnosis

```typescript
const RETENTION_DIAGNOSIS = {
  d1Low: {
    threshold: '< 35%',
    possibleCauses: [
      'Tutorial too long or confusing',
      'First match experience poor',
      'Performance issues on device',
      'Core loop not immediately engaging',
      'Technical issues (crashes, bugs)'
    ],
    investigations: [
      'Tutorial completion funnel',
      'First match outcomes',
      'Device performance data',
      'Session 1 duration',
      'Crash reports from new users'
    ],
    solutions: {
      tutorial: 'Shorten, add skip option, improve guidance',
      firstMatch: 'Easier bots, guaranteed loot, smaller lobby',
      performance: 'Lower default settings, optimize loading',
      engagement: 'Better onboarding rewards, clearer goals'
    }
  },

  d7Low: {
    threshold: '< 12%',
    possibleCauses: [
      'Progression feels slow',
      'Core loop gets repetitive',
      'No social hooks',
      'Monetization walls',
      'Balance issues frustrating players'
    ],
    investigations: [
      'Session count days 2-7',
      'Progression curve analysis',
      'Friend/party formation rate',
      'Store visit vs purchase',
      'Ability usage distribution'
    ],
    solutions: {
      progression: 'Adjust XP curve, add milestones, varied rewards',
      variety: 'More modes, events, daily variety',
      social: 'Friend recommendations, party bonuses',
      balance: 'Nerf frustrating abilities, buff weak ones'
    }
  },

  d30Low: {
    threshold: '< 4%',
    possibleCauses: [
      'Content exhaustion',
      'No long-term goals',
      'Better competitors',
      'Technical fatigue',
      'Life changes (external)'
    ],
    investigations: [
      'Content completion rates',
      'Battle Pass progress',
      'Competitive engagement',
      'Win rate trends',
      'Churn surveys'
    ],
    solutions: {
      content: 'Regular new content, seasonal events',
      goals: 'Ranked system, mastery tracks, collections',
      retention: 'Come-back rewards, lapsed player campaigns'
    }
  }
};
```

### Monetization Diagnosis

```typescript
const MONETIZATION_DIAGNOSIS = {
  conversionLow: {
    threshold: '< 2%',
    possibleCauses: [
      'Shop not visible enough',
      'Prices feel too high',
      'Free progression too generous',
      'Items not compelling',
      'Trust issues (new game)'
    ],
    investigations: [
      'Shop visit rate',
      'IAP funnel drop-off',
      'Price point testing results',
      'Item popularity in shop views',
      'Free vs paid player behavior'
    ],
    solutions: {
      visibility: 'Shop prompts, featured items, daily deals',
      pricing: 'Adjust price points, add cheaper options',
      value: 'Better bundles, exclusive items, limited time offers',
      trust: 'Starter packs, low-risk first purchase'
    }
  },

  arpuLow: {
    threshold: '< $0.10 ARPDAU',
    possibleCauses: [
      'Whales not spending enough',
      'Mid-spenders missing',
      'Content runs out for payers',
      'Repeat purchase friction'
    ],
    investigations: [
      'Spending distribution curve',
      'Purchase frequency',
      'Item category performance',
      'Whale behavior analysis'
    ],
    solutions: {
      depth: 'More premium content tiers',
      breadth: 'Items for different player types',
      frequency: 'Consumables, limited offers, season passes'
    }
  }
};
```

### Technical Diagnosis

```typescript
const TECHNICAL_DIAGNOSIS = {
  crashRateHigh: {
    threshold: '> 0.5%',
    possibleCauses: [
      'Memory leaks',
      'Device compatibility',
      'Network edge cases',
      'Asset loading failures',
      'Third-party SDK issues'
    ],
    investigations: [
      'Crash stack traces',
      'Device/OS distribution',
      'Memory usage patterns',
      'Network conditions at crash',
      'Reproduce on test devices'
    ],
    solutions: {
      immediate: 'Hotfix critical crashes',
      systematic: 'Memory profiling, stress testing',
      preventive: 'Crash guards, graceful degradation'
    }
  },

  performancePoor: {
    threshold: 'FPS < 25 on target devices',
    possibleCauses: [
      'Too many draw calls',
      'Unoptimized shaders',
      'Physics overhead',
      'Network sync heavy',
      'Memory pressure'
    ],
    investigations: [
      'Frame time breakdown',
      'GPU vs CPU bound',
      'Specific scenarios (squad fights)',
      'Device thermal state'
    ],
    solutions: {
      rendering: 'LOD, culling, batching',
      effects: 'Reduce particles, simplify shaders',
      settings: 'Auto-adjust based on performance'
    }
  }
};
```

---

## Iteration Process

### Weekly Cycle

```typescript
const WEEKLY_CYCLE = {
  monday: {
    activity: 'Data Review & Planning',
    tasks: [
      'Review previous week metrics',
      'Analyze A/B test results',
      'Identify top issues',
      'Plan week\'s focus areas',
      'Assign investigation tasks'
    ]
  },

  tuesday_wednesday: {
    activity: 'Development',
    tasks: [
      'Implement fixes and improvements',
      'Create new A/B test variants',
      'QA testing',
      'Code review'
    ]
  },

  thursday: {
    activity: 'Release',
    tasks: [
      'Deploy update',
      'Monitor rollout',
      'Verify fixes',
      'Start new A/B tests'
    ]
  },

  friday: {
    activity: 'Monitoring & Documentation',
    tasks: [
      'Monitor new build stability',
      'Document changes made',
      'Prepare weekly report',
      'Plan next week preview'
    ]
  },

  weekend: {
    activity: 'Monitoring (On-call)',
    tasks: [
      'On-call monitoring',
      'Hotfix if critical issues',
      'Data collection continues'
    ]
  }
};
```

### Change Categories

```typescript
const CHANGE_CATEGORIES = {
  hotfixes: {
    description: 'Critical bug fixes',
    approval: 'Engineering lead',
    testing: 'Smoke test only',
    deployment: 'Immediate'
  },

  balanceChanges: {
    description: 'Number tuning, ability adjustments',
    approval: 'Design + Engineering',
    testing: 'Playtest validation',
    deployment: 'Weekly update',
    abTest: 'When significant'
  },

  featureChanges: {
    description: 'UI changes, flow modifications',
    approval: 'Product + Design + Engineering',
    testing: 'Full QA cycle',
    deployment: 'Bi-weekly',
    abTest: 'Required for major changes'
  },

  contentAdditions: {
    description: 'New items, cosmetics, events',
    approval: 'Product + Art',
    testing: 'Standard QA',
    deployment: 'Scheduled',
    abTest: 'Optional'
  }
};
```

---

## Focus Area Playbooks

### Onboarding Improvements

```typescript
const ONBOARDING_PLAYBOOK = {
  trigger: 'D1 retention < 35%',

  quickWins: {
    tutorialLength: {
      action: 'Reduce tutorial from 7 to 4 steps',
      expected: '+5% D1',
      testDuration: '1 week'
    },
    skipOption: {
      action: 'Add "Skip for experienced players"',
      expected: '+3% D1 for churned reinstalls',
      testDuration: '1 week'
    },
    firstMatchEase: {
      action: 'First match vs easier bots',
      expected: '+8% D1',
      testDuration: '1 week'
    }
  },

  mediumEffort: {
    ftueCutscene: {
      action: 'Add engaging intro cinematic',
      expected: '+3% D1',
      timeline: '2 weeks'
    },
    guidedFirstSession: {
      action: 'Quest markers for first 3 matches',
      expected: '+5% D1',
      timeline: '2 weeks'
    }
  },

  majorChanges: {
    tutorialRedesign: {
      action: 'Complete tutorial overhaul',
      expected: '+10% D1',
      timeline: '4 weeks'
    }
  }
};
```

### Core Loop Refinement

```typescript
const CORE_LOOP_PLAYBOOK = {
  trigger: 'D7 retention < 12%',

  quickWins: {
    dailyVariety: {
      action: 'Rotate featured mode daily',
      expected: '+2% D7',
      testDuration: '2 weeks'
    },
    progressionBoost: {
      action: 'Increase XP gains by 20%',
      expected: '+3% D7',
      testDuration: '1 week'
    },
    questRefresh: {
      action: 'More achievable daily quests',
      expected: '+2% D7',
      testDuration: '1 week'
    }
  },

  mediumEffort: {
    milestoneRewards: {
      action: 'Add 1-week milestone rewards',
      expected: '+5% D7',
      timeline: '2 weeks'
    },
    socialFeatures: {
      action: 'Friend play bonuses',
      expected: '+4% D7',
      timeline: '2 weeks'
    }
  },

  majorChanges: {
    newMode: {
      action: 'Add ranked mode or new game mode',
      expected: '+8% D7',
      timeline: '4 weeks'
    }
  }
};
```

### Monetization Optimization

```typescript
const MONETIZATION_PLAYBOOK = {
  trigger: 'Conversion < 2% or ARPU < $0.10',

  quickWins: {
    starterPack: {
      action: 'Prominent starter pack offer',
      expected: '+1% conversion',
      testDuration: '1 week'
    },
    priceTest: {
      action: 'Test lower entry price points',
      expected: '+0.5% conversion',
      testDuration: '2 weeks'
    },
    shopPrompt: {
      action: 'Prompt after key moments',
      expected: '+0.5% conversion',
      testDuration: '1 week'
    }
  },

  mediumEffort: {
    battlePassValue: {
      action: 'Improve Battle Pass rewards',
      expected: '+20% BP purchases',
      timeline: '2 weeks'
    },
    dailyDeals: {
      action: 'Rotating daily deals system',
      expected: '+15% revenue',
      timeline: '2 weeks'
    }
  },

  majorChanges: {
    economyRebalance: {
      action: 'Full economy overhaul',
      expected: '+50% revenue',
      timeline: '4 weeks',
      risk: 'High - test carefully'
    }
  }
};
```

### Balance Adjustments

```typescript
const BALANCE_PLAYBOOK = {
  trigger: 'Player feedback + data shows imbalance',

  process: {
    identification: {
      dataSignals: [
        'Ability win rate deviation > 5%',
        'Pick rate > 40% or < 5%',
        'Complaint volume spike'
      ],
      qualitativeSignals: [
        'Reddit/Discord complaints',
        'Streamer feedback',
        'Support tickets'
      ]
    },

    adjustment: {
      magnitude: 'Start with 10-15% changes',
      direction: 'Nerf overperformers, buff underperformers',
      communication: 'Patch notes explain reasoning'
    },

    validation: {
      duration: '1 week minimum',
      metrics: 'Win rate, pick rate, sentiment',
      iteration: 'Adjust further if needed'
    }
  },

  principles: {
    smallChanges: 'Avoid dramatic swings',
    transparency: 'Communicate changes clearly',
    patience: 'Let meta settle before more changes'
  }
};
```

---

## A/B Testing Strategy

### Test Prioritization

```typescript
const AB_TEST_PRIORITY = {
  highPriority: {
    criteria: 'Direct impact on key metrics',
    examples: [
      'Tutorial flow variants',
      'Pricing experiments',
      'Core loop pacing',
      'First session experience'
    ],
    resources: 'Full test setup, dedicated analysis'
  },

  mediumPriority: {
    criteria: 'Quality of life, engagement',
    examples: [
      'UI layout changes',
      'Notification timing',
      'Reward presentation',
      'Shop layout'
    ],
    resources: 'Standard test setup'
  },

  lowPriority: {
    criteria: 'Polish, minor improvements',
    examples: [
      'Copy changes',
      'Color variations',
      'Animation tweaks'
    ],
    resources: 'Quick test, fast graduation'
  }
};
```

### Test Configuration

```typescript
const AB_TEST_CONFIG = {
  sampleSize: {
    minimum: 1000,
    recommended: 5000,
    perVariant: true
  },

  duration: {
    minimum: '7 days',
    recommended: '14 days',
    maximum: '30 days'
  },

  significance: {
    confidenceLevel: 95,
    minimumLift: '5% relative',
    analysis: 'Bayesian or frequentist'
  },

  segmentation: {
    newUsers: 'Install date within test period',
    existingUsers: 'Careful with experience changes',
    byMarket: 'If market-specific hypothesis'
  }
};
```

---

## Update Pipeline

### Release Process

```typescript
const RELEASE_PROCESS = {
  preparation: {
    codeFreeze: '24 hours before release',
    qaValidation: 'Regression suite pass',
    releaseNotes: 'Drafted and approved',
    rollbackPlan: 'Previous version ready'
  },

  deployment: {
    staging: 'Deploy to staging, smoke test',
    phasedRollout: {
      day1: '10% of users',
      day2: '50% of users',
      day3: '100% of users'
    },
    monitoring: 'Watch crash rate, errors, metrics'
  },

  postRelease: {
    verification: 'Confirm fixes work',
    metrics: 'Track impact on KPIs',
    feedback: 'Monitor player reception',
    hotfixReady: 'Team on standby day 1'
  }
};
```

### Version Management

```typescript
const VERSION_MANAGEMENT = {
  naming: {
    format: 'major.minor.patch',
    example: '1.2.3',
    major: 'Significant content/feature releases',
    minor: 'Weekly updates',
    patch: 'Hotfixes'
  },

  compatibility: {
    serverBackward: 'Support N-2 client versions',
    forceUpdate: 'For critical security/stability',
    gracefulUpgrade: 'Prompt but don\'t force when possible'
  },

  tracking: {
    changelog: 'Maintained for all versions',
    rollback: 'Previous 3 versions preserved',
    analytics: 'Version tagged in all events'
  }
};
```

---

## Success Metrics

### Iteration Targets

```typescript
const ITERATION_TARGETS = {
  weeklyImprovement: {
    retention: '+0.5-1% D1 or D7 per week',
    crashes: '-20% crash rate per week',
    revenue: '+5-10% per week during optimization'
  },

  exitTargets: {
    d1Retention: '>= 38%',
    d7Retention: '>= 14%',
    conversion: '>= 2.5%',
    crashRate: '< 0.3%',
    appRating: '>= 4.2 stars'
  },

  timeline: {
    minimum: '4 weeks',
    typical: '6-8 weeks',
    maximum: '12 weeks (reassess strategy)'
  }
};
```

### Global Launch Readiness

```typescript
const LAUNCH_READINESS = {
  metrics: {
    retention: {
      d1: '>= 38%',
      d7: '>= 14%',
      d30: '>= 5%',
      trend: 'Stable or improving'
    },
    monetization: {
      conversion: '>= 2.5%',
      arpdau: '>= $0.12',
      trend: 'Stable or improving'
    },
    technical: {
      crashRate: '< 0.3%',
      anrRate: '< 0.2%',
      loadTime: '< 8 seconds'
    }
  },

  qualitative: {
    appRating: '>= 4.2 average',
    sentiment: 'Positive overall',
    feedback: 'No major unaddressed issues'
  },

  operational: {
    updatePipeline: 'Proven reliable',
    contentPipeline: 'Ready for live ops',
    teamCapacity: 'Scaled for global'
  }
};
```

---

## Documentation

### Required Reports

```typescript
const DOCUMENTATION = {
  weeklyReport: {
    content: [
      'KPI summary with WoW changes',
      'Tests run and results',
      'Changes deployed',
      'Issues identified',
      'Next week plan'
    ],
    audience: 'Full team + stakeholders',
    timing: 'Friday EOD'
  },

  testReport: {
    content: [
      'Hypothesis',
      'Test configuration',
      'Results with confidence',
      'Recommendation',
      'Next steps'
    ],
    timing: 'Upon test conclusion'
  },

  iterationSummary: {
    content: [
      'Starting metrics',
      'All changes made',
      'Final metrics',
      'Key learnings',
      'Recommendations for global'
    ],
    timing: 'End of soft launch iteration'
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial soft launch iteration specification |
