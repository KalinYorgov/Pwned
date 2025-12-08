# Economy Monitoring Dashboard Specification

## Document Information
- **Task ID:** MON-019
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-08

---

## Overview

This document specifies the economy monitoring dashboard for Plunderstorm Mobile, providing real-time visibility into the game's virtual economy health, currency flows, and player spending behaviors.

---

## Dashboard Architecture

### System Design

```typescript
const DASHBOARD_ARCHITECTURE = {
  dataSource: {
    primary: 'Game analytics database',
    realTime: 'Event streaming (Kafka/Kinesis)',
    aggregated: 'Pre-computed daily/hourly rollups'
  },

  backend: {
    api: 'REST API for dashboard queries',
    caching: 'Redis for frequently accessed metrics',
    computation: 'Batch jobs for heavy aggregations'
  },

  frontend: {
    framework: 'React-based dashboard',
    visualization: 'Recharts or D3.js',
    hosting: 'Internal tools platform'
  },

  refresh: {
    realTime: 'WebSocket for live metrics',
    nearRealTime: '5-minute refresh for key metrics',
    daily: 'Full recalculation at midnight UTC'
  }
};
```

---

## Core Metrics

### Currency Metrics

```typescript
const CURRENCY_METRICS = {
  goldEarned: {
    name: 'Gold Earned',
    description: 'Total gold earned by players',
    granularity: ['Hourly', 'Daily', 'Weekly', 'Monthly'],
    breakdowns: {
      bySource: ['Match rewards', 'Quest rewards', 'Daily login', 'Ad rewards', 'Events'],
      bySegment: ['New users', 'Returning users', 'Payers', 'Non-payers'],
      byRegion: 'Country-level breakdown'
    },
    aggregations: {
      total: 'Sum across all players',
      perUser: 'Average per active user',
      median: 'Median per active user',
      percentiles: ['p25', 'p50', 'p75', 'p90', 'p99']
    }
  },

  goldSpent: {
    name: 'Gold Spent',
    description: 'Total gold spent by players',
    granularity: ['Hourly', 'Daily', 'Weekly', 'Monthly'],
    breakdowns: {
      bySink: ['Item purchases', 'Ability upgrades', 'Cosmetics', 'Battle Pass tiers'],
      byItem: 'Individual item purchase rates',
      bySegment: 'Player segment breakdown'
    },
    aggregations: {
      total: 'Sum across all players',
      perUser: 'Average per active user',
      perSpender: 'Average per user who spent'
    }
  },

  goldBalance: {
    name: 'Gold Balance Distribution',
    description: 'Current gold holdings across players',
    metrics: {
      totalInCirculation: 'Sum of all player gold balances',
      averageBalance: 'Mean balance per player',
      medianBalance: 'Median balance',
      distribution: 'Histogram of balance ranges'
    },
    alerts: {
      inflation: 'Total balance growing faster than spend rate',
      hoarding: 'High % of players at balance cap'
    }
  },

  premiumCurrency: {
    name: 'Gems (Premium Currency)',
    earned: {
      sources: ['IAP purchases', 'Battle Pass', 'Special events', 'First-time bonuses'],
      tracking: 'Same granularity as gold'
    },
    spent: {
      sinks: ['Premium cosmetics', 'Battle Pass', 'Special offers', 'Convenience items'],
      tracking: 'Same granularity as gold'
    },
    balance: 'Distribution tracking similar to gold'
  },

  netFlow: {
    name: 'Currency Net Flow',
    formula: 'Earned - Spent',
    interpretation: {
      positive: 'Economy inflating (more earning than spending)',
      negative: 'Economy deflating (more spending than earning)',
      balanced: 'Healthy equilibrium'
    },
    target: 'Slightly negative to drive engagement'
  }
};
```

### Spending Metrics

```typescript
const SPENDING_METRICS = {
  iapRevenue: {
    name: 'IAP Revenue',
    metrics: {
      gross: 'Total IAP revenue before platform fees',
      net: 'Revenue after 30% platform fee',
      byProduct: 'Revenue per IAP product',
      byRegion: 'Revenue by country'
    },
    kpis: {
      arpu: 'Average Revenue Per User',
      arppu: 'Average Revenue Per Paying User',
      conversionRate: 'Payers / Total users',
      ltv: 'Lifetime Value estimates'
    }
  },

  purchaseRates: {
    name: 'Item Purchase Rates',
    tracking: {
      byItem: 'Purchases per item',
      byCategory: 'Cosmetics, Abilities, Consumables',
      byPrice: 'Sales by price tier',
      byTime: 'Purchase timing patterns'
    },
    analysis: {
      topSellers: 'Best selling items',
      underperformers: 'Items with low sales',
      priceElasticity: 'Sales vs price correlation'
    }
  },

  topSpenders: {
    name: 'Top Spenders (Whales)',
    identification: {
      threshold: 'Top 1% by spend',
      lifetime: 'Total spend > $100',
      recent: 'Spend in last 30 days'
    },
    tracking: {
      count: 'Number of whale players',
      revenue: '% of revenue from whales',
      behavior: 'What whales buy',
      retention: 'Whale retention rates'
    },
    alerts: {
      whaleChurn: 'Alert if whale stops spending',
      unusualSpend: 'Alert for abnormal purchase patterns'
    }
  },

  battlePassMetrics: {
    name: 'Battle Pass Performance',
    metrics: {
      purchaseRate: '% of players who bought pass',
      completionRate: '% who reach max tier',
      tierDistribution: 'Player distribution across tiers',
      revenue: 'Total Battle Pass revenue',
      upgradeRate: '% who buy premium track'
    },
    timing: {
      purchaseTiming: 'When in season players buy',
      progressionRate: 'Daily tier advancement'
    }
  }
};
```

### Player Behavior Metrics

```typescript
const BEHAVIOR_METRICS = {
  earnRates: {
    name: 'Earn Rate Analysis',
    metrics: {
      goldPerMatch: 'Average gold earned per match',
      goldPerHour: 'Gold earned per hour played',
      xpPerMatch: 'XP earned per match',
      progressionSpeed: 'Time to reach level milestones'
    },
    segmentation: {
      bySkill: 'Earn rates by player skill level',
      byMode: 'Solo vs duo vs squad',
      byEngagement: 'Casual vs hardcore'
    }
  },

  spendBehavior: {
    name: 'Spending Behavior',
    patterns: {
      firstPurchase: 'Time to first purchase',
      repeatPurchase: 'Time between purchases',
      basketSize: 'Average transaction value',
      purchaseFrequency: 'Purchases per week/month'
    },
    triggers: {
      afterWin: 'Purchase likelihood after winning',
      afterLoss: 'Purchase likelihood after losing',
      afterPromotion: 'Response to sales/offers'
    }
  },

  economyHealth: {
    name: 'Economy Health Indicators',
    inflation: {
      metric: 'Gold supply growth rate',
      healthy: '< 5% weekly growth',
      concern: '> 10% weekly growth'
    },
    velocity: {
      metric: 'Gold spent / Gold in circulation',
      healthy: '> 50% weekly velocity',
      concern: '< 30% (hoarding)'
    },
    distribution: {
      gini: 'Gini coefficient of gold distribution',
      healthy: '< 0.6',
      concern: '> 0.8 (too concentrated)'
    }
  }
};
```

---

## Dashboard Views

### Overview Dashboard

```typescript
const OVERVIEW_DASHBOARD = {
  layout: 'Single page with key metrics',

  sections: {
    topKPIs: {
      position: 'Top row',
      metrics: [
        { name: 'Daily Revenue', comparison: 'vs yesterday' },
        { name: 'DAU', comparison: 'vs yesterday' },
        { name: 'Gold Earned', comparison: 'vs 7-day avg' },
        { name: 'Gold Spent', comparison: 'vs 7-day avg' },
        { name: 'Net Flow', indicator: 'positive/negative' }
      ],
      style: 'Large numbers with trend arrows'
    },

    revenueTrend: {
      position: 'Main chart area',
      chart: 'Line chart',
      data: ['Daily IAP revenue', 'Daily ad revenue'],
      timeRange: 'Last 30 days',
      annotations: ['Events', 'Sales', 'Updates']
    },

    currencyFlow: {
      position: 'Secondary chart',
      chart: 'Stacked area chart',
      data: {
        earn: 'Gold earned by source',
        spend: 'Gold spent by sink'
      },
      timeRange: 'Last 14 days'
    },

    quickStats: {
      position: 'Side panel',
      metrics: [
        'Conversion rate',
        'ARPDAU',
        'Battle Pass purchase rate',
        'Top selling item today'
      ]
    },

    alerts: {
      position: 'Bottom panel',
      content: 'Active alerts and warnings',
      priority: 'Sorted by severity'
    }
  }
};
```

### Currency Detail View

```typescript
const CURRENCY_DETAIL_VIEW = {
  goldTab: {
    earnBreakdown: {
      chart: 'Pie chart + trend lines',
      data: 'Gold earned by source',
      drilldown: 'Click source for details'
    },
    spendBreakdown: {
      chart: 'Pie chart + trend lines',
      data: 'Gold spent by sink',
      drilldown: 'Click sink for item details'
    },
    balanceDistribution: {
      chart: 'Histogram',
      data: 'Player count by balance range',
      stats: ['Mean', 'Median', 'Percentiles']
    },
    netFlowTrend: {
      chart: 'Line chart with zero line',
      data: 'Daily net flow (earn - spend)',
      target: 'Target range overlay'
    }
  },

  gemsTab: {
    purchaseBreakdown: {
      chart: 'Bar chart',
      data: 'Gems purchased by IAP product',
      revenue: 'Associated revenue'
    },
    spendBreakdown: {
      chart: 'Pie chart',
      data: 'Gems spent by category'
    },
    conversion: {
      funnel: 'View gems -> Consider -> Purchase',
      rates: 'Conversion at each step'
    }
  }
};
```

### Player Segments View

```typescript
const SEGMENT_VIEW = {
  segments: {
    definition: {
      nonPayers: 'Never purchased',
      minnows: 'Spent $1-$10',
      dolphins: 'Spent $10-$100',
      whales: 'Spent $100+'
    },
    metrics: [
      'Segment size',
      'Revenue contribution',
      'Avg gold balance',
      'Avg earn rate',
      'Avg spend rate',
      'Retention rate'
    ]
  },

  comparison: {
    chart: 'Grouped bar chart',
    data: 'Metrics comparison across segments',
    insights: 'Automated insights generation'
  },

  cohorts: {
    byJoinDate: 'Players by registration week',
    byFirstPurchase: 'Players by first purchase week',
    retention: 'Cohort retention curves',
    ltv: 'LTV by cohort'
  }
};
```

---

## Alerting System

### Alert Configuration

```typescript
const ALERT_SYSTEM = {
  types: {
    threshold: {
      description: 'Metric crosses defined threshold',
      examples: [
        'Daily gold earned > 150% of average',
        'Net flow > +10% (inflation)',
        'Conversion rate drops below 2%'
      ]
    },
    anomaly: {
      description: 'Statistical anomaly detected',
      method: 'Z-score or ML-based detection',
      examples: [
        'Unusual spend pattern by single user',
        'Unexpected spike in gold generation',
        'Item purchase rate deviation'
      ]
    },
    trend: {
      description: 'Sustained trend in wrong direction',
      examples: [
        'Revenue declining for 5+ consecutive days',
        'Gold inflation accelerating',
        'Whale count decreasing'
      ]
    }
  },

  severity: {
    critical: {
      description: 'Immediate attention required',
      examples: ['Exploit detected', 'Revenue drop > 50%'],
      notification: 'PagerDuty + Email + Slack',
      response: 'Within 1 hour'
    },
    high: {
      description: 'Same-day attention needed',
      examples: ['Significant metric deviation', 'Whale churn'],
      notification: 'Email + Slack',
      response: 'Within 4 hours'
    },
    medium: {
      description: 'Review within 24 hours',
      examples: ['Trend concerns', 'Minor anomalies'],
      notification: 'Email',
      response: 'Next business day'
    },
    low: {
      description: 'Informational',
      examples: ['Metric approaching threshold'],
      notification: 'Dashboard only',
      response: 'Weekly review'
    }
  }
};
```

### Pre-configured Alerts

```typescript
const DEFAULT_ALERTS = {
  exploitDetection: {
    name: 'Potential Exploit Detection',
    triggers: [
      'Player gold earned > 10x daily average',
      'Gold source "unknown" or "error"',
      'Rapid repeated transactions'
    ],
    severity: 'Critical',
    action: 'Flag player for review, potentially pause account'
  },

  inflationAlert: {
    name: 'Economy Inflation Warning',
    trigger: 'Total gold in circulation growing > 10% weekly',
    severity: 'High',
    action: 'Review earn rates, consider sink adjustments'
  },

  revenueDropAlert: {
    name: 'Revenue Drop Alert',
    trigger: 'Daily revenue < 70% of 7-day moving average',
    severity: 'High',
    action: 'Investigate cause, check for technical issues'
  },

  conversionDropAlert: {
    name: 'Conversion Rate Drop',
    trigger: 'Daily conversion rate drops > 20% vs average',
    severity: 'Medium',
    action: 'Review store, offers, and user flow'
  },

  whaleChurnAlert: {
    name: 'Whale Churn Warning',
    trigger: 'Whale player inactive for 7+ days',
    severity: 'Medium',
    action: 'Consider re-engagement campaign'
  },

  battlePassPaceAlert: {
    name: 'Battle Pass Completion Pace',
    trigger: 'Completion rate behind schedule by > 20%',
    severity: 'Low',
    action: 'Review XP earn rates, consider catch-up mechanics'
  }
};
```

---

## Reporting

### Automated Reports

```typescript
const AUTOMATED_REPORTS = {
  daily: {
    name: 'Daily Economy Report',
    schedule: '09:00 UTC',
    recipients: ['economy-team@company.com'],
    contents: {
      summary: 'Key metrics vs yesterday and 7-day avg',
      highlights: 'Notable changes or events',
      alerts: 'Any triggered alerts',
      topItems: 'Best selling items',
      concerns: 'Metrics trending poorly'
    },
    format: 'Email with inline charts + PDF attachment'
  },

  weekly: {
    name: 'Weekly Economy Review',
    schedule: 'Monday 09:00 UTC',
    recipients: ['leadership@company.com', 'economy-team@company.com'],
    contents: {
      weekSummary: 'Revenue, DAU, key metrics',
      trends: 'Week-over-week comparisons',
      cohortAnalysis: 'New user cohort performance',
      recommendations: 'Suggested actions',
      forecast: 'Next week projections'
    },
    format: 'PDF report + dashboard link'
  },

  monthly: {
    name: 'Monthly Economy Analysis',
    schedule: '1st of month, 09:00 UTC',
    recipients: ['exec-team@company.com'],
    contents: {
      executiveSummary: 'High-level performance',
      deepDive: 'Detailed metric analysis',
      segmentAnalysis: 'Player segment performance',
      productPerformance: 'IAP and item analysis',
      strategicRecommendations: 'Economy adjustments needed'
    },
    format: 'Presentation deck + detailed PDF'
  }
};
```

### Export Capabilities

```typescript
const EXPORT_FEATURES = {
  dataExport: {
    formats: ['CSV', 'JSON', 'Excel'],
    scope: ['Current view', 'Date range', 'Custom query'],
    scheduling: 'One-time or recurring exports'
  },

  reportGeneration: {
    templates: ['Executive summary', 'Detailed analysis', 'Custom'],
    formats: ['PDF', 'PowerPoint'],
    branding: 'Company logo and styling'
  },

  apiAccess: {
    purpose: 'Integration with BI tools',
    authentication: 'API key based',
    endpoints: [
      'GET /api/economy/metrics',
      'GET /api/economy/timeseries',
      'GET /api/economy/segments'
    ],
    rateLimit: '1000 requests/hour'
  }
};
```

---

## Access Control

### Role-Based Access

```typescript
const ACCESS_CONTROL = {
  roles: {
    viewer: {
      permissions: ['View dashboards', 'Export data'],
      users: 'All team members'
    },
    analyst: {
      permissions: ['View', 'Export', 'Create custom views', 'Set personal alerts'],
      users: 'Economy and analytics team'
    },
    admin: {
      permissions: ['Full access', 'Configure alerts', 'Manage users'],
      users: 'Economy lead, Engineering lead'
    }
  },

  dataAccess: {
    aggregated: 'All roles can see aggregated metrics',
    playerLevel: 'Only admin can see individual player data',
    revenue: 'Revenue data restricted to specific roles',
    pii: 'No PII exposed in dashboard'
  },

  auditLog: {
    tracked: ['Login', 'Data export', 'Alert changes', 'User management'],
    retention: '1 year'
  }
};
```

---

## Technical Implementation

### Data Pipeline

```typescript
const DATA_PIPELINE = {
  ingestion: {
    source: 'Game event stream',
    processing: 'Real-time aggregation',
    storage: {
      raw: 'Data lake (S3/GCS)',
      processed: 'Time-series database (InfluxDB/TimescaleDB)',
      serving: 'PostgreSQL + Redis cache'
    }
  },

  aggregations: {
    realTime: {
      metrics: ['Active users', 'Current revenue', 'Live transactions'],
      latency: '< 1 minute'
    },
    hourly: {
      metrics: ['Hourly totals', 'Hourly averages'],
      job: 'Scheduled aggregation job'
    },
    daily: {
      metrics: ['All daily rollups', 'Cohort calculations'],
      job: 'Nightly batch job at 00:30 UTC'
    }
  },

  performance: {
    dashboardLoad: '< 2 seconds',
    chartRender: '< 500ms',
    exportLarge: '< 30 seconds for 1M rows'
  }
};
```

---

## Testing Requirements

```typescript
const DASHBOARD_TESTING = {
  functional: [
    'All metrics calculate correctly',
    'Filters work as expected',
    'Date range selection accurate',
    'Drill-downs navigate correctly',
    'Exports contain correct data'
  ],

  accuracy: [
    'Metrics match source systems',
    'Aggregations sum correctly',
    'Time zones handled properly',
    'Currency conversions accurate'
  ],

  alerting: [
    'Alerts trigger at correct thresholds',
    'Notifications delivered to right recipients',
    'Alert history maintained',
    'Acknowledgment flow works'
  ],

  performance: [
    'Dashboard loads within 2 seconds',
    'Large date ranges don\'t timeout',
    'Concurrent users supported (50+)',
    'Export performance acceptable'
  ],

  access: [
    'Role permissions enforced',
    'Unauthorized access blocked',
    'Audit logs captured'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-08 | Development Team | Initial economy monitoring dashboard specification |
