# Ad Mediation and Analytics Specification

## Document Information
- **Task ID:** MON-017
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-07

---

## Overview

This document specifies the ad mediation layer and analytics system for Plunderstorm Mobile, enabling revenue optimization through multiple ad networks and data-driven placement decisions.

---

## Mediation Architecture

### Mediation Platform

```typescript
const MEDIATION_PLATFORM = {
  primary: {
    provider: 'IronSource LevelPlay',
    rationale: [
      'Industry-leading mediation',
      'Auto-optimization algorithms',
      'Unified dashboard',
      'Strong mobile game support'
    ]
  },

  alternative: {
    provider: 'AppLovin MAX',
    useCase: 'Backup if IronSource underperforms'
  },

  integration: {
    sdk: 'Single SDK handles all networks',
    initialization: 'On app launch after consent',
    configuration: 'Server-side waterfall management'
  }
};
```

### Ad Network Integration

```typescript
const AD_NETWORKS = {
  integrated: [
    {
      name: 'Unity Ads',
      type: 'rewarded',
      tier: 1,
      regions: 'Global',
      strengths: 'Gaming audience, high engagement'
    },
    {
      name: 'AppLovin',
      type: 'rewarded',
      tier: 1,
      regions: 'Global',
      strengths: 'High eCPM, quality ads'
    },
    {
      name: 'AdMob',
      type: 'rewarded',
      tier: 1,
      regions: 'Global',
      strengths: 'High fill rate, Google ecosystem'
    },
    {
      name: 'Vungle',
      type: 'rewarded',
      tier: 2,
      regions: 'NA, EU',
      strengths: 'Premium video ads'
    },
    {
      name: 'Facebook Audience Network',
      type: 'rewarded',
      tier: 2,
      regions: 'Global',
      strengths: 'Targeting, high engagement'
    },
    {
      name: 'Chartboost',
      type: 'rewarded',
      tier: 3,
      regions: 'Global',
      strengths: 'Gaming focused, good fill'
    },
    {
      name: 'InMobi',
      type: 'rewarded',
      tier: 3,
      regions: 'APAC, India',
      strengths: 'Emerging markets'
    },
    {
      name: 'Mintegral',
      type: 'rewarded',
      tier: 3,
      regions: 'APAC, China',
      strengths: 'Asian markets'
    }
  ],

  minimumNetworks: 4,
  recommendedNetworks: 6,
  purpose: 'Maximize fill rate and eCPM competition'
};
```

---

## Waterfall Configuration

### Waterfall Strategy

```typescript
const WATERFALL_CONFIG = {
  strategy: {
    type: 'Hybrid (waterfall + bidding)',
    description: 'Combine traditional waterfall with real-time bidding'
  },

  bidding: {
    enabled: true,
    networks: ['AppLovin', 'Facebook', 'Unity Ads', 'Vungle'],
    priority: 'Highest bidder wins',
    floorPrice: {
      enabled: true,
      default: 5.00, // $5 CPM floor
      byRegion: {
        US: 8.00,
        UK: 7.00,
        DE: 6.50,
        JP: 6.00,
        BR: 3.00,
        IN: 1.50
      }
    }
  },

  waterfall: {
    // Fallback for non-bidding networks
    tiers: [
      {
        tier: 1,
        networks: ['AdMob', 'Chartboost'],
        minEcpm: 10.00,
        timeout: 5000
      },
      {
        tier: 2,
        networks: ['InMobi', 'Mintegral'],
        minEcpm: 5.00,
        timeout: 4000
      },
      {
        tier: 3,
        networks: ['All remaining'],
        minEcpm: 0,
        timeout: 3000
      }
    ]
  },

  optimization: {
    autoOptimize: true,
    algorithm: 'ML-based eCPM prediction',
    reorderFrequency: 'Daily',
    dataDriven: true
  }
};
```

### Regional Configuration

```typescript
const REGIONAL_WATERFALL = {
  regions: {
    tier1: {
      countries: ['US', 'CA', 'UK', 'AU', 'DE', 'FR', 'JP'],
      characteristics: 'High eCPM, competitive bidding',
      strategy: 'Maximize eCPM with bidding priority',
      floorMultiplier: 1.0
    },
    tier2: {
      countries: ['BR', 'MX', 'ES', 'IT', 'KR', 'TW'],
      characteristics: 'Medium eCPM, good volume',
      strategy: 'Balance eCPM and fill rate',
      floorMultiplier: 0.6
    },
    tier3: {
      countries: ['IN', 'ID', 'PH', 'VN', 'TH'],
      characteristics: 'Lower eCPM, high volume',
      strategy: 'Maximize fill rate',
      floorMultiplier: 0.3
    },
    other: {
      countries: 'Rest of world',
      characteristics: 'Variable performance',
      strategy: 'Auto-optimize based on data',
      floorMultiplier: 0.4
    }
  },

  networkPriority: {
    US: ['AppLovin', 'Unity Ads', 'AdMob', 'Vungle'],
    JP: ['Unity Ads', 'AppLovin', 'AdMob', 'Mintegral'],
    IN: ['InMobi', 'AdMob', 'Unity Ads', 'Mintegral'],
    CN: ['Mintegral', 'Unity Ads', 'AdMob'],
    default: ['AppLovin', 'Unity Ads', 'AdMob', 'Facebook']
  }
};
```

---

## Analytics System

### Core Metrics

```typescript
const AD_METRICS = {
  impression: {
    definition: 'Ad successfully displayed to user',
    tracking: 'Network callback + internal verification',
    granularity: ['Total', 'By network', 'By placement', 'By region']
  },

  revenue: {
    definition: 'Earnings from ad impressions',
    source: 'Network reporting APIs',
    currency: 'USD',
    reconciliation: 'Daily sync with network dashboards'
  },

  ecpm: {
    definition: 'Effective cost per mille (revenue per 1000 impressions)',
    formula: '(revenue / impressions) * 1000',
    useCase: 'Network and placement comparison'
  },

  fillRate: {
    definition: 'Percentage of ad requests that return an ad',
    formula: '(fills / requests) * 100',
    target: '> 95%'
  },

  completionRate: {
    definition: 'Percentage of started ads that complete',
    formula: '(completed / started) * 100',
    target: '> 90%'
  },

  arpdau: {
    definition: 'Ad revenue per daily active user',
    formula: 'daily_ad_revenue / DAU',
    benchmark: '$0.05 - $0.15 for mobile games'
  },

  adsPerDau: {
    definition: 'Average ads watched per daily active user',
    formula: 'daily_impressions / DAU',
    target: '2-4 for rewarded ads'
  }
};
```

### Analytics Events

```typescript
const ANALYTICS_EVENTS = {
  adRequest: {
    name: 'ad_request',
    properties: {
      placement_id: 'string',
      network: 'string (if targeted)',
      user_segment: 'string',
      timestamp: 'ISO8601'
    }
  },

  adFilled: {
    name: 'ad_filled',
    properties: {
      placement_id: 'string',
      network: 'string',
      ecpm_estimate: 'number',
      latency_ms: 'number'
    }
  },

  adNotFilled: {
    name: 'ad_not_filled',
    properties: {
      placement_id: 'string',
      networks_tried: 'string[]',
      reason: 'string'
    }
  },

  adImpression: {
    name: 'ad_impression',
    properties: {
      placement_id: 'string',
      network: 'string',
      ecpm: 'number',
      country: 'string',
      user_segment: 'string'
    }
  },

  adCompleted: {
    name: 'ad_completed',
    properties: {
      placement_id: 'string',
      network: 'string',
      watch_duration_seconds: 'number',
      reward_granted: 'boolean'
    }
  },

  adSkipped: {
    name: 'ad_skipped',
    properties: {
      placement_id: 'string',
      network: 'string',
      skip_time_seconds: 'number'
    }
  },

  adError: {
    name: 'ad_error',
    properties: {
      placement_id: 'string',
      network: 'string',
      error_code: 'string',
      error_message: 'string'
    }
  },

  adRevenue: {
    name: 'ad_revenue',
    properties: {
      placement_id: 'string',
      network: 'string',
      revenue_usd: 'number',
      currency: 'string',
      precision: 'estimated | exact'
    }
  }
};
```

---

## Dashboard and Reporting

### Analytics Dashboard

```typescript
const AD_DASHBOARD = {
  overview: {
    metrics: [
      'Total Revenue (today, week, month)',
      'Total Impressions',
      'Average eCPM',
      'Fill Rate',
      'ARPDAU'
    ],
    charts: [
      'Revenue trend (daily)',
      'Impressions trend (daily)',
      'eCPM by network (bar)',
      'Fill rate by network (bar)'
    ]
  },

  networkPerformance: {
    view: 'Table + charts',
    columns: [
      'Network',
      'Impressions',
      'Revenue',
      'eCPM',
      'Fill Rate',
      'Completion Rate',
      'Share %'
    ],
    filters: ['Date range', 'Region', 'Placement'],
    sorting: 'By any column'
  },

  placementPerformance: {
    view: 'Placement breakdown',
    metrics: [
      'Impressions per placement',
      'Revenue per placement',
      'Engagement rate',
      'eCPM variance'
    ],
    insights: 'Best/worst performing placements'
  },

  regionalPerformance: {
    view: 'Geographic breakdown',
    map: 'Revenue heatmap by country',
    table: [
      'Country',
      'Revenue',
      'eCPM',
      'Impressions',
      'Top network'
    ]
  },

  trends: {
    timeframes: ['7 days', '30 days', '90 days', 'Custom'],
    comparisons: 'Period over period',
    annotations: 'Mark events (app updates, config changes)'
  }
};
```

### Automated Reports

```typescript
const AUTOMATED_REPORTS = {
  daily: {
    schedule: '09:00 UTC',
    recipients: ['monetization-team@company.com'],
    contents: [
      'Yesterday\'s revenue',
      'eCPM changes',
      'Fill rate issues',
      'Top/bottom performers'
    ],
    format: 'Email summary + dashboard link'
  },

  weekly: {
    schedule: 'Monday 09:00 UTC',
    recipients: ['leadership@company.com'],
    contents: [
      'Weekly revenue summary',
      'Week-over-week trends',
      'Network performance comparison',
      'Recommendations'
    ],
    format: 'PDF report + email'
  },

  alerts: {
    revenueDropAlert: {
      trigger: 'Daily revenue < 70% of 7-day average',
      action: 'Immediate email + Slack',
      severity: 'High'
    },
    fillRateAlert: {
      trigger: 'Fill rate < 80%',
      action: 'Email to ad ops',
      severity: 'Medium'
    },
    networkDownAlert: {
      trigger: 'Network impressions = 0 for 2 hours',
      action: 'Immediate alert',
      severity: 'High'
    }
  }
};
```

---

## A/B Testing for Ads

### Placement Testing

```typescript
const AD_AB_TESTING = {
  framework: {
    integration: 'Uses BACK-022 A/B Testing Framework',
    adSpecific: 'Ad-focused metrics and segments'
  },

  testableElements: {
    placementPosition: {
      description: 'Where ad button appears',
      variants: ['Above fold', 'Below fold', 'Floating'],
      metric: 'Engagement rate'
    },
    buttonDesign: {
      description: 'Ad button visual design',
      variants: ['Standard', 'Highlighted', 'Animated'],
      metric: 'Click-through rate'
    },
    rewardValue: {
      description: 'Reward amount offered',
      variants: ['+25%', '+50%', '+100%'],
      metric: 'Completion rate, revenue'
    },
    timing: {
      description: 'When to show ad opportunity',
      variants: ['Immediate', 'After 2s delay', 'On scroll'],
      metric: 'Engagement rate'
    },
    frequency: {
      description: 'Daily limit for placements',
      variants: ['3 per day', '5 per day', '10 per day'],
      metric: 'Revenue, user sentiment'
    }
  },

  segmentation: {
    payers: 'Different treatment for paying users',
    engagement: 'Heavy vs light users',
    region: 'Regional testing',
    newVsReturning: 'Experience level'
  },

  guardrails: {
    minSampleSize: 10000,
    maxRevenueDecrease: '10%',
    maxFillRateDecrease: '5%',
    userExperience: 'No increase in negative feedback'
  }
};
```

### Test Analysis

```typescript
const TEST_ANALYSIS = {
  metrics: {
    primary: 'Revenue per user',
    secondary: [
      'Ads per user',
      'Completion rate',
      'Fill rate',
      'User retention impact'
    ]
  },

  statisticalMethods: {
    significance: 'p < 0.05',
    confidence: '95% confidence interval',
    sampleSize: 'Power analysis for required N'
  },

  reporting: {
    realTime: 'Live metric tracking',
    dailySummary: 'Daily performance by variant',
    finalReport: 'Statistical analysis at conclusion'
  },

  decision: {
    winner: 'Variant with highest revenue without UX degradation',
    rollout: 'Gradual rollout of winner',
    documentation: 'Archive test results'
  }
};
```

---

## Data Integration

### Network Data Sync

```typescript
const DATA_SYNC = {
  sources: {
    networkApis: {
      description: 'Official reporting APIs from each network',
      frequency: 'Every 4 hours',
      data: ['Impressions', 'Revenue', 'eCPM', 'Fill rate']
    },
    sdkCallbacks: {
      description: 'Real-time callbacks from SDK',
      frequency: 'Real-time',
      data: ['Impression events', 'Completion events', 'Errors']
    },
    mediationDashboard: {
      description: 'Aggregated data from IronSource',
      frequency: 'Hourly',
      data: ['Combined metrics', 'Waterfall performance']
    }
  },

  reconciliation: {
    frequency: 'Daily',
    process: [
      'Compare SDK reported vs network reported',
      'Identify discrepancies > 5%',
      'Flag for investigation',
      'Adjust estimates if needed'
    ],
    tolerance: '5% variance acceptable'
  },

  storage: {
    rawData: 'Store all raw events for 90 days',
    aggregated: 'Store daily aggregates for 2 years',
    warehouse: 'BigQuery or similar for analysis'
  }
};
```

### Attribution Integration

```typescript
const ATTRIBUTION_INTEGRATION = {
  purpose: 'Connect ad revenue to user acquisition',

  mmpIntegration: {
    providers: ['Adjust', 'AppsFlyer', 'Singular'],
    data: {
      adRevenue: 'Send ad revenue events to MMP',
      userSource: 'Segment by acquisition source'
    }
  },

  analysis: {
    ltv: 'Include ad revenue in user LTV',
    roas: 'Calculate ROAS including ad revenue',
    cohorts: 'Ad revenue by acquisition cohort'
  },

  reporting: {
    dashboard: 'Unified revenue view (IAP + Ads)',
    exportable: 'Data export for BI tools'
  }
};
```

---

## Performance Optimization

### Optimization Strategies

```typescript
const OPTIMIZATION_STRATEGIES = {
  ecpmOptimization: {
    autoFloors: {
      enabled: true,
      algorithm: 'ML-based floor optimization',
      adjustment: 'Automatic based on fill rate'
    },
    networkMix: {
      analysis: 'Regular review of network performance',
      action: 'Adjust waterfall priorities'
    },
    bidding: {
      priority: 'Increase bidder participation',
      floorTuning: 'Optimize floors for bidding'
    }
  },

  fillRateOptimization: {
    networkRedundancy: 'Multiple networks per tier',
    timeoutTuning: 'Optimize request timeouts',
    preloading: 'Preload ads for instant availability'
  },

  userExperienceOptimization: {
    loadTime: 'Minimize ad load latency',
    frequency: 'Avoid ad fatigue',
    quality: 'Filter low-quality ads'
  },

  revenueMaximization: {
    highValueUsers: 'Identify and optimize for high engagers',
    timing: 'Show ads at optimal moments',
    personalization: 'Segment-based optimization'
  }
};
```

---

## Testing Requirements

```typescript
const MEDIATION_TESTING = {
  integration: [
    'All networks initialize successfully',
    'Bidding works correctly',
    'Waterfall fallback functions',
    'Network switching is seamless'
  ],

  analytics: [
    'All events fire correctly',
    'Revenue attribution is accurate',
    'Dashboard data matches network reports',
    'Alerts trigger appropriately'
  ],

  performance: [
    'Ad requests complete in < 5 seconds',
    'No app freezing during ad load',
    'Memory usage acceptable'
  ],

  accuracy: [
    'Revenue reporting within 5% of actuals',
    'Fill rate tracking accurate',
    'Regional data correctly attributed'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-07 | Development Team | Initial ad mediation and analytics specification |
