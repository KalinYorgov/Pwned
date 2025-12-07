# A/B Testing Framework Specification

## Document Information
- **Task ID:** BACK-022
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the A/B testing framework for Plunderstorm Mobile, enabling controlled feature rollouts, experiment management, user segmentation, and statistical analysis of variant performance.

---

## System Architecture

### Core Components

```typescript
const AB_TESTING_ARCHITECTURE = {
  components: {
    configService: {
      purpose: 'Store and serve experiment configurations',
      storage: 'Database + distributed cache',
      sync: 'Real-time updates to clients'
    },

    assignmentService: {
      purpose: 'Assign users to experiment variants',
      algorithm: 'Deterministic hash-based assignment',
      persistence: 'Assignment stored for consistency'
    },

    metricsCollector: {
      purpose: 'Collect and aggregate experiment metrics',
      integration: 'Analytics pipeline',
      realTime: 'Near real-time metric updates'
    },

    analysisEngine: {
      purpose: 'Statistical analysis and significance testing',
      methods: ['T-test', 'Chi-squared', 'Bayesian'],
      automation: 'Continuous monitoring for significance'
    },

    dashboard: {
      purpose: 'Experiment management and results viewing',
      access: 'Product and engineering teams',
      features: ['Create', 'Monitor', 'Analyze', 'Archive']
    }
  },

  dataFlow: [
    'Experiment created in dashboard',
    'Config synced to clients via config service',
    'Client requests variant assignment',
    'Assignment service returns consistent variant',
    'Client implements variant behavior',
    'Metrics collected and attributed to variant',
    'Analysis engine computes significance',
    'Results displayed in dashboard'
  ]
};
```

---

## Feature Flags

### Flag Configuration

```typescript
const FEATURE_FLAGS = {
  structure: {
    id: 'Unique flag identifier',
    name: 'Human-readable name',
    description: 'Purpose and expected behavior',
    type: 'boolean | percentage | variant',
    enabled: 'Master kill switch',
    targeting: 'User segments',
    variants: 'Possible values',
    defaultValue: 'Fallback if evaluation fails'
  },

  types: {
    boolean: {
      description: 'Simple on/off toggle',
      values: [true, false],
      useCase: 'Enable/disable feature globally'
    },

    percentage: {
      description: 'Gradual rollout by percentage',
      values: [0, 100], // 0-100%
      useCase: 'Progressive feature launch'
    },

    variant: {
      description: 'Multiple value variants',
      values: ['control', 'variant_a', 'variant_b', ...],
      useCase: 'A/B/n testing'
    }
  },

  example: {
    id: 'new_tutorial_flow',
    name: 'New Tutorial Experience',
    description: 'Test redesigned onboarding tutorial',
    type: 'variant',
    enabled: true,
    variants: [
      { id: 'control', weight: 50, description: 'Original tutorial' },
      { id: 'streamlined', weight: 25, description: 'Shorter tutorial' },
      { id: 'interactive', weight: 25, description: 'Interactive tutorial' }
    ],
    defaultValue: 'control',
    targeting: {
      newUsersOnly: true,
      regions: ['NA', 'EU']
    }
  }
};

interface FeatureFlag {
  id: string;
  name: string;
  description: string;
  type: 'boolean' | 'percentage' | 'variant';
  enabled: boolean;
  variants?: Variant[];
  defaultValue: any;
  targeting?: TargetingRules;
  metadata: {
    createdAt: Date;
    createdBy: string;
    updatedAt: Date;
    status: 'draft' | 'active' | 'paused' | 'completed' | 'archived';
  };
}
```

### Remote Configuration

```typescript
const REMOTE_CONFIG = {
  sync: {
    onAppStart: 'Fetch latest config',
    periodic: 'Every 15 minutes',
    onDemand: 'Pull-to-refresh in dev mode',
    realTime: 'WebSocket for critical updates'
  },

  caching: {
    local: {
      storage: 'Device storage',
      ttl: 86400, // 24 hours
      fallback: 'Use cached config if fetch fails'
    },
    server: {
      cdn: 'Edge-cached config endpoints',
      ttl: 300 // 5 minutes
    }
  },

  versioning: {
    configVersion: 'Incremented on any change',
    deltaUpdates: 'Only changed flags sent',
    compatibility: 'Client version targeting'
  },

  failsafe: {
    fetchFailure: 'Use cached config',
    parseError: 'Use default values',
    networkTimeout: 5000 // ms
  }
};
```

---

## User Assignment

### Assignment Algorithm

```typescript
const ASSIGNMENT_ALGORITHM = {
  method: 'Deterministic hash-based',

  algorithm: {
    description: 'Hash(userId + experimentId) -> bucket',
    properties: [
      'Consistent: Same user always gets same variant',
      'Random: Uniform distribution across variants',
      'Independent: Different experiments assigned independently'
    ],

    implementation: (userId: string, experimentId: string, buckets: number): number => {
      const hashInput = `${userId}:${experimentId}`;
      const hash = murmurHash3(hashInput);
      return hash % buckets; // 0 to buckets-1
    }
  },

  variantAssignment: (
    userId: string,
    experiment: Experiment
  ): string => {
    const bucket = assignmentAlgorithm.implementation(
      userId,
      experiment.id,
      100 // 100 buckets for percentage precision
    );

    let cumulative = 0;
    for (const variant of experiment.variants) {
      cumulative += variant.weight;
      if (bucket < cumulative) {
        return variant.id;
      }
    }

    return experiment.defaultValue;
  },

  stickyAssignment: {
    enabled: true,
    storage: 'User profile + local cache',
    purpose: 'Ensure consistent experience across sessions'
  }
};
```

### User Segmentation

```typescript
const USER_SEGMENTATION = {
  segments: {
    newUsers: {
      rule: 'accountAge < 7 days',
      priority: 1
    },
    returningUsers: {
      rule: 'accountAge >= 7 days && daysInactive < 30',
      priority: 2
    },
    lapsedUsers: {
      rule: 'daysInactive >= 30',
      priority: 3
    },
    payers: {
      rule: 'totalSpend > 0',
      priority: 4
    },
    whales: {
      rule: 'totalSpend > 100',
      priority: 5
    }
  },

  targeting: {
    include: {
      segments: ['Specific segments to target'],
      regions: ['Region codes'],
      platforms: ['ios', 'android'],
      appVersions: ['Minimum version', 'Maximum version'],
      deviceTiers: ['low', 'mid', 'high']
    },

    exclude: {
      segments: ['Segments to exclude'],
      userIds: ['Specific users to exclude']
    },

    customRules: {
      description: 'Custom targeting expressions',
      examples: [
        'level >= 10 && wins >= 5',
        'tutorialComplete == true',
        'platform == "ios" && osVersion >= "16.0"'
      ]
    }
  },

  evaluation: {
    order: [
      'Check exclusion rules first',
      'Check inclusion rules',
      'Evaluate custom rules',
      'Apply segment targeting',
      'Default to control if no match'
    ],
    caching: 'Cache segment membership for 1 hour'
  }
};
```

---

## Experiment Management

### Experiment Lifecycle

```typescript
const EXPERIMENT_LIFECYCLE = {
  states: {
    draft: {
      description: 'Experiment being configured',
      actions: ['Edit', 'Delete', 'Start']
    },
    active: {
      description: 'Experiment running, collecting data',
      actions: ['Pause', 'Stop', 'Adjust weights']
    },
    paused: {
      description: 'Temporarily stopped',
      actions: ['Resume', 'Stop']
    },
    completed: {
      description: 'Experiment finished, winner selected',
      actions: ['Archive', 'Create follow-up']
    },
    archived: {
      description: 'Historical record only',
      actions: ['View results']
    }
  },

  transitions: {
    'draft -> active': 'Start experiment',
    'active -> paused': 'Pause for investigation',
    'paused -> active': 'Resume experiment',
    'active -> completed': 'End experiment, select winner',
    'completed -> archived': 'Archive old experiments'
  },

  scheduling: {
    startDate: 'Optional scheduled start',
    endDate: 'Optional auto-end date',
    duration: 'Minimum run time recommendation'
  }
};

interface Experiment {
  id: string;
  name: string;
  description: string;
  hypothesis: string;
  primaryMetric: string;
  secondaryMetrics: string[];
  variants: Variant[];
  targeting: TargetingRules;
  schedule: {
    startDate?: Date;
    endDate?: Date;
    minDuration: number; // days
  };
  status: ExperimentStatus;
  results?: ExperimentResults;
  metadata: {
    owner: string;
    team: string;
    createdAt: Date;
    updatedAt: Date;
  };
}
```

### Kill Switch

```typescript
const KILL_SWITCH = {
  purpose: 'Emergency disable of experiment or feature',

  levels: {
    experiment: {
      action: 'Immediately stop experiment',
      effect: 'All users get control/default',
      reversible: true
    },
    flag: {
      action: 'Disable feature flag entirely',
      effect: 'Feature disabled for all users',
      reversible: true
    },
    global: {
      action: 'Disable all experiments',
      effect: 'All users get defaults',
      trigger: 'Major incident only'
    }
  },

  implementation: {
    trigger: 'Dashboard button or API call',
    propagation: '< 60 seconds to all clients',
    notification: 'Alert sent to owners'
  },

  automaticTriggers: {
    errorSpike: {
      threshold: '5x baseline errors',
      action: 'Pause experiment, alert team'
    },
    crashRate: {
      threshold: '2x baseline crashes',
      action: 'Kill switch, revert to control'
    },
    metricGuardrail: {
      threshold: 'Defined per experiment',
      action: 'Pause and alert'
    }
  }
};
```

---

## Metrics Collection

### Metric Types

```typescript
const METRIC_TYPES = {
  conversion: {
    description: 'Binary outcome (did/did not)',
    examples: ['Completed tutorial', 'Made purchase', 'Reached level 10'],
    analysis: 'Chi-squared test'
  },

  count: {
    description: 'Discrete count metrics',
    examples: ['Matches played', 'Abilities used', 'Items purchased'],
    analysis: 'Poisson regression or Mann-Whitney'
  },

  continuous: {
    description: 'Continuous value metrics',
    examples: ['Session duration', 'Revenue per user', 'Retention days'],
    analysis: 'T-test or Mann-Whitney'
  },

  ratio: {
    description: 'Calculated ratios',
    examples: ['Win rate', 'Kill/death ratio', 'Spend per session'],
    analysis: 'Bootstrap confidence intervals'
  },

  time: {
    description: 'Time-based metrics',
    examples: ['D1 retention', 'D7 retention', 'Time to first purchase'],
    analysis: 'Survival analysis'
  }
};

const METRIC_DEFINITIONS = {
  primary: {
    description: 'Main success metric for experiment',
    limit: 1, // Only one primary metric
    required: true
  },

  secondary: {
    description: 'Supporting metrics for context',
    limit: 5, // Max 5 secondary metrics
    required: false
  },

  guardrail: {
    description: 'Metrics that should not degrade',
    examples: ['Crash rate', 'Load time', 'Error rate'],
    threshold: 'Alert if degraded beyond threshold'
  }
};
```

### Metric Attribution

```typescript
const METRIC_ATTRIBUTION = {
  exposure: {
    definition: 'User was exposed to experiment',
    tracking: 'Log when variant assigned/displayed',
    required: 'Must be exposed to count in analysis'
  },

  attribution: {
    method: 'Last-touch attribution',
    window: {
      default: 7, // days
      configurable: true,
      maximum: 30
    },
    logic: 'Metric attributed to variant user was in when exposed'
  },

  deduplication: {
    method: 'Count unique users, not events',
    granularity: 'Per user per experiment per metric'
  },

  storage: {
    exposures: {
      table: 'experiment_exposures',
      columns: ['user_id', 'experiment_id', 'variant_id', 'timestamp']
    },
    events: {
      table: 'experiment_events',
      columns: ['user_id', 'experiment_id', 'variant_id', 'metric_id', 'value', 'timestamp']
    }
  }
};
```

---

## Statistical Analysis

### Significance Testing

```typescript
const STATISTICAL_ANALYSIS = {
  frequentist: {
    methods: {
      tTest: {
        useCase: 'Continuous metrics (revenue, time)',
        assumptions: 'Normal distribution or large n',
        output: 'p-value, confidence interval'
      },
      chiSquared: {
        useCase: 'Conversion/binary metrics',
        assumptions: 'Expected count >= 5',
        output: 'p-value, lift percentage'
      },
      mannWhitneyU: {
        useCase: 'Non-normal continuous metrics',
        assumptions: 'None (non-parametric)',
        output: 'p-value, effect size'
      }
    },

    parameters: {
      confidenceLevel: 0.95, // 95% confidence
      significanceThreshold: 0.05, // p < 0.05
      minimumSampleSize: 1000 // per variant
    }
  },

  bayesian: {
    enabled: true,
    method: 'Beta-binomial for conversion, Normal-Normal for continuous',
    output: {
      probability: 'P(variant > control)',
      credibleInterval: '95% credible interval',
      expectedLoss: 'Expected loss if wrong'
    },
    decisionRule: {
      winProbability: 0.95, // 95% probability to call winner
      minLift: 0.01 // Minimum 1% lift to be meaningful
    }
  },

  sampleSize: {
    calculation: {
      inputs: ['Baseline rate', 'Minimum detectable effect', 'Power', 'Significance'],
      method: 'Power analysis',
      recommendation: 'Show required sample size before starting'
    },
    monitoring: {
      warning: 'Alert if sample size insufficient',
      early_stopping: 'Allow early stopping if clear winner'
    }
  }
};
```

### Results Calculation

```typescript
const RESULTS_CALCULATION = {
  metrics: {
    perVariant: {
      sampleSize: 'Number of users',
      mean: 'Mean value of metric',
      standardDeviation: 'Variance measure',
      confidenceInterval: '[lower, upper] bounds'
    },

    comparison: {
      absoluteLift: 'Variant mean - Control mean',
      relativeLift: '(Variant - Control) / Control * 100',
      pValue: 'Statistical significance',
      significant: 'p < threshold'
    }
  },

  timeline: {
    daily: 'Daily metric values for trend',
    cumulative: 'Running totals over time',
    trend: 'Is metric improving/declining?'
  },

  segments: {
    breakdown: 'Results by user segment',
    interaction: 'Different effects on different segments?'
  },

  recommendation: {
    automatic: {
      clearWinner: 'Variant X is significantly better',
      noSignificance: 'No significant difference detected',
      inconclusive: 'Need more data'
    },
    confidence: 'Confidence level for recommendation'
  }
};
```

---

## Dashboard

### Experiment Dashboard UI

```typescript
const DASHBOARD_UI = {
  views: {
    experimentList: {
      columns: ['Name', 'Status', 'Start Date', 'Primary Metric', 'Significance'],
      filters: ['Status', 'Owner', 'Team', 'Date range'],
      actions: ['Create', 'View', 'Edit', 'Archive']
    },

    experimentDetail: {
      sections: {
        overview: ['Name', 'Description', 'Hypothesis', 'Status', 'Schedule'],
        variants: ['Variant details', 'Traffic allocation', 'Targeting'],
        results: ['Primary metric results', 'Secondary metrics', 'Segment breakdown'],
        timeline: ['Metric trends over time', 'Sample size growth'],
        settings: ['Edit targeting', 'Adjust weights', 'Kill switch']
      }
    },

    createExperiment: {
      steps: [
        'Basic info (name, description, hypothesis)',
        'Variants (define variants and weights)',
        'Metrics (select primary and secondary)',
        'Targeting (who sees the experiment)',
        'Schedule (start/end dates)',
        'Review and launch'
      ]
    }
  },

  visualizations: {
    resultsChart: {
      type: 'Bar chart with confidence intervals',
      comparison: 'Control vs variants'
    },
    timelineChart: {
      type: 'Line chart',
      data: 'Metric value over time per variant'
    },
    funnelChart: {
      type: 'Funnel visualization',
      data: 'Conversion funnel per variant'
    },
    distributionChart: {
      type: 'Histogram',
      data: 'Value distribution per variant'
    }
  },

  access: {
    viewer: 'View experiments and results',
    editor: 'Create and modify experiments',
    admin: 'Manage settings, users, integrations'
  }
};
```

---

## API Endpoints

### Client SDK API

```typescript
const CLIENT_API = {
  getAssignment: {
    method: 'GET',
    path: '/api/v1/experiments/assignment',
    query: {
      userId: 'User identifier',
      experiments: 'Comma-separated experiment IDs (optional)'
    },
    response: {
      assignments: [{
        experimentId: 'string',
        variantId: 'string',
        parameters: 'object (optional variant params)'
      }],
      configVersion: 'number'
    },
    caching: {
      client: '1 hour',
      cdn: '5 minutes'
    }
  },

  trackExposure: {
    method: 'POST',
    path: '/api/v1/experiments/exposure',
    body: {
      userId: 'string',
      experimentId: 'string',
      variantId: 'string',
      timestamp: 'ISO8601'
    },
    batching: 'Can batch multiple exposures',
    async: 'Fire-and-forget'
  },

  trackEvent: {
    method: 'POST',
    path: '/api/v1/experiments/event',
    body: {
      userId: 'string',
      experimentId: 'string',
      variantId: 'string',
      metricId: 'string',
      value: 'number',
      timestamp: 'ISO8601'
    },
    batching: 'Can batch multiple events',
    async: 'Fire-and-forget'
  },

  getConfig: {
    method: 'GET',
    path: '/api/v1/experiments/config',
    response: {
      experiments: 'Active experiment configs',
      flags: 'Feature flag configs',
      version: 'Config version number'
    },
    caching: {
      cdn: '5 minutes',
      etag: true
    }
  }
};
```

### Admin API

```typescript
const ADMIN_API = {
  experiments: {
    list: 'GET /api/v1/admin/experiments',
    get: 'GET /api/v1/admin/experiments/{id}',
    create: 'POST /api/v1/admin/experiments',
    update: 'PUT /api/v1/admin/experiments/{id}',
    delete: 'DELETE /api/v1/admin/experiments/{id}',
    start: 'POST /api/v1/admin/experiments/{id}/start',
    pause: 'POST /api/v1/admin/experiments/{id}/pause',
    stop: 'POST /api/v1/admin/experiments/{id}/stop',
    results: 'GET /api/v1/admin/experiments/{id}/results'
  },

  flags: {
    list: 'GET /api/v1/admin/flags',
    get: 'GET /api/v1/admin/flags/{id}',
    create: 'POST /api/v1/admin/flags',
    update: 'PUT /api/v1/admin/flags/{id}',
    delete: 'DELETE /api/v1/admin/flags/{id}',
    toggle: 'POST /api/v1/admin/flags/{id}/toggle'
  },

  authorization: {
    method: 'Bearer token',
    rbac: 'Role-based access control',
    audit: 'All changes logged'
  }
};
```

---

## Data Storage

### Database Schema

```typescript
const AB_DATABASE_SCHEMA = {
  tables: {
    experiments: {
      columns: {
        id: 'UUID PRIMARY KEY',
        name: 'VARCHAR(255)',
        description: 'TEXT',
        hypothesis: 'TEXT',
        primary_metric: 'VARCHAR(100)',
        secondary_metrics: 'JSONB',
        variants: 'JSONB',
        targeting: 'JSONB',
        status: 'VARCHAR(50)',
        start_date: 'TIMESTAMP',
        end_date: 'TIMESTAMP',
        created_by: 'UUID',
        created_at: 'TIMESTAMP',
        updated_at: 'TIMESTAMP'
      }
    },

    feature_flags: {
      columns: {
        id: 'VARCHAR(100) PRIMARY KEY',
        name: 'VARCHAR(255)',
        type: 'VARCHAR(50)',
        enabled: 'BOOLEAN',
        variants: 'JSONB',
        targeting: 'JSONB',
        default_value: 'JSONB',
        created_at: 'TIMESTAMP',
        updated_at: 'TIMESTAMP'
      }
    },

    user_assignments: {
      columns: {
        user_id: 'VARCHAR(255)',
        experiment_id: 'UUID',
        variant_id: 'VARCHAR(100)',
        assigned_at: 'TIMESTAMP'
      },
      primaryKey: '(user_id, experiment_id)'
    },

    experiment_exposures: {
      columns: {
        id: 'BIGSERIAL PRIMARY KEY',
        user_id: 'VARCHAR(255)',
        experiment_id: 'UUID',
        variant_id: 'VARCHAR(100)',
        exposed_at: 'TIMESTAMP'
      },
      indexes: ['(experiment_id, variant_id, exposed_at)']
    },

    experiment_events: {
      columns: {
        id: 'BIGSERIAL PRIMARY KEY',
        user_id: 'VARCHAR(255)',
        experiment_id: 'UUID',
        variant_id: 'VARCHAR(100)',
        metric_id: 'VARCHAR(100)',
        value: 'DECIMAL',
        event_at: 'TIMESTAMP'
      },
      indexes: ['(experiment_id, metric_id, event_at)']
    },

    experiment_results: {
      columns: {
        experiment_id: 'UUID',
        variant_id: 'VARCHAR(100)',
        metric_id: 'VARCHAR(100)',
        sample_size: 'INTEGER',
        sum_value: 'DECIMAL',
        sum_squared: 'DECIMAL',
        calculated_at: 'TIMESTAMP'
      },
      primaryKey: '(experiment_id, variant_id, metric_id)',
      description: 'Pre-aggregated results for performance'
    }
  }
};
```

---

## Performance Requirements

```typescript
const AB_PERFORMANCE = {
  assignment: {
    latency: {
      p50: '5ms',
      p99: '20ms'
    },
    throughput: '10,000 requests/second',
    caching: 'Assignment cached client-side'
  },

  config: {
    latency: {
      p50: '50ms',
      p99: '200ms'
    },
    propagation: '< 60 seconds for changes',
    caching: 'CDN + client cache'
  },

  events: {
    latency: 'Async, no blocking',
    throughput: '100,000 events/second',
    delivery: 'At-least-once guarantee'
  },

  analysis: {
    realTime: 'Results update every 5 minutes',
    fullRecalc: '< 1 minute for 1M users'
  },

  storage: {
    eventRetention: '90 days',
    resultsRetention: 'Permanent for experiments'
  }
};
```

---

## Security and Audit

```typescript
const AB_SECURITY = {
  authentication: {
    client: 'API key for client SDK',
    admin: 'OAuth/OIDC for dashboard'
  },

  authorization: {
    rbac: {
      viewer: 'Read experiments and results',
      editor: 'Create/modify experiments',
      admin: 'Full access including settings'
    },
    teamScoping: 'Experiments scoped to teams'
  },

  auditLog: {
    events: [
      'Experiment created/modified/started/stopped',
      'Flag toggled/modified',
      'Weights changed',
      'Kill switch activated'
    ],
    retention: '1 year',
    fields: ['who', 'what', 'when', 'details']
  },

  dataProtection: {
    pii: 'User IDs hashed for storage',
    gdpr: 'Respect data deletion requests',
    anonymization: 'Aggregate results only in reports'
  }
};
```

---

## Testing Requirements

```typescript
const AB_TESTING_TESTS = {
  unit: [
    'Assignment algorithm is deterministic',
    'Hash distribution is uniform',
    'Targeting rules evaluate correctly',
    'Statistical calculations are accurate'
  ],

  integration: [
    'Config sync works across services',
    'Exposures and events are tracked correctly',
    'Results aggregate properly',
    'Kill switch propagates to clients'
  ],

  performance: [
    'Assignment latency under load',
    'Event ingestion throughput',
    'Analysis computation time'
  ],

  statistical: [
    'A/A tests show no significant difference',
    'Known effect sizes detected correctly',
    'Confidence intervals are calibrated'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial A/B testing framework specification |
