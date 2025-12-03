# Analytics Dashboard Specification

## Overview

The Analytics Dashboard provides a comprehensive, real-time view of game health, player behavior, and business performance. This system enables data-driven decision making for LiveOps, product, and business stakeholders.

## Dashboard Architecture

```typescript
const DASHBOARD_ARCHITECTURE = {
  platform: 'web',
  framework: 'customBuilt',
  dataSource: 'dataWarehouse',

  tiers: {
    realTime: {
      latency: '<1 minute',
      dataSource: 'streamingPipeline',
      retention: '24 hours'
    },
    nearRealTime: {
      latency: '<15 minutes',
      dataSource: 'batchProcessing',
      retention: '90 days'
    },
    historical: {
      latency: 'daily',
      dataSource: 'dataWarehouse',
      retention: '2 years'
    }
  },

  access: {
    authentication: 'sso',
    authorization: 'roleBased',
    audit: true
  }
};
```

## Executive Overview Dashboard

```typescript
const EXECUTIVE_DASHBOARD = {
  name: 'Executive Overview',
  audience: ['executives', 'stakeholders'],
  refreshRate: 'hourly',

  sections: {
    keyMetrics: {
      layout: 'topBanner',
      metrics: [
        {
          name: 'DAU',
          value: 'number',
          trend: 'vs yesterday',
          sparkline: '7 days',
          alert: { below: 'target' }
        },
        {
          name: 'Revenue (Today)',
          value: 'currency',
          trend: 'vs same day last week',
          sparkline: '7 days',
          alert: { below: 'target' }
        },
        {
          name: 'New Users (Today)',
          value: 'number',
          trend: 'vs yesterday',
          sparkline: '7 days'
        },
        {
          name: 'D1 Retention',
          value: 'percentage',
          trend: 'vs last week',
          sparkline: '30 days'
        }
      ]
    },

    charts: {
      layout: 'grid2x2',
      charts: [
        {
          name: 'DAU/MAU Trend',
          type: 'lineChart',
          timeRange: '30 days',
          series: ['DAU', 'MAU', 'stickiness']
        },
        {
          name: 'Revenue Breakdown',
          type: 'stackedBar',
          timeRange: '7 days',
          series: ['iap', 'subscriptions', 'ads']
        },
        {
          name: 'Retention Curves',
          type: 'lineChart',
          cohorts: ['D1', 'D7', 'D14', 'D30'],
          comparison: 'lastWeek'
        },
        {
          name: 'Top Performers',
          type: 'table',
          metrics: ['topSellingItem', 'mostPlayedMode', 'topEvent']
        }
      ]
    },

    alerts: {
      layout: 'sidebar',
      showActive: true,
      showResolved: 'last24h'
    }
  }
};
```

## Player Metrics Dashboard

```typescript
const PLAYER_METRICS_DASHBOARD = {
  name: 'Player Metrics',
  audience: ['product', 'liveops', 'analytics'],
  refreshRate: '15 minutes',

  sections: {
    acquisition: {
      title: 'Acquisition',
      metrics: [
        {
          name: 'New Users',
          breakdown: ['organic', 'paid', 'referral'],
          chart: 'stackedArea'
        },
        {
          name: 'Install Sources',
          breakdown: ['appStore', 'playStore', 'direct'],
          chart: 'pieChart'
        },
        {
          name: 'Geographic Distribution',
          breakdown: 'byCountry',
          chart: 'worldMap'
        },
        {
          name: 'Tutorial Completion Rate',
          funnel: ['install', 'open', 'tutorialStart', 'tutorialComplete'],
          chart: 'funnelChart'
        }
      ]
    },

    engagement: {
      title: 'Engagement',
      metrics: [
        {
          name: 'DAU/WAU/MAU',
          chart: 'lineChart',
          timeRange: '90 days'
        },
        {
          name: 'Sessions per DAU',
          average: true,
          distribution: true
        },
        {
          name: 'Session Length',
          average: true,
          distribution: 'histogram',
          buckets: ['<5m', '5-15m', '15-30m', '30-60m', '>60m']
        },
        {
          name: 'Matches per Session',
          average: true,
          distribution: true
        },
        {
          name: 'Time Spent by Activity',
          breakdown: ['matches', 'menu', 'shop', 'quests', 'social'],
          chart: 'donut'
        }
      ]
    },

    retention: {
      title: 'Retention',
      metrics: [
        {
          name: 'Retention Curves',
          days: ['D1', 'D3', 'D7', 'D14', 'D30', 'D60', 'D90'],
          chart: 'lineChart',
          cohortComparison: true
        },
        {
          name: 'Cohort Analysis',
          table: 'cohortHeatmap',
          metric: 'retention',
          granularity: 'weekly'
        },
        {
          name: 'Churn Analysis',
          breakdown: ['churned', 'atRisk', 'engaged', 'loyal'],
          chart: 'segmentedBar'
        },
        {
          name: 'Resurrection Rate',
          definition: 'returning after 7+ days',
          chart: 'lineChart'
        }
      ]
    },

    playerSegments: {
      title: 'Player Segments',
      segments: [
        { name: 'New', definition: 'registered < 7 days' },
        { name: 'Casual', definition: '<3 sessions/week' },
        { name: 'Regular', definition: '3-7 sessions/week' },
        { name: 'Hardcore', definition: '>7 sessions/week' },
        { name: 'Whales', definition: 'spent >$100' },
        { name: 'At Risk', definition: 'no login 5-7 days' },
        { name: 'Churned', definition: 'no login >14 days' }
      ],
      metrics: ['size', 'revenue', 'engagement', 'ltv']
    }
  }
};
```

## Revenue Dashboard

```typescript
const REVENUE_DASHBOARD = {
  name: 'Revenue & Monetization',
  audience: ['business', 'product', 'executives'],
  refreshRate: 'hourly',

  sections: {
    revenueOverview: {
      title: 'Revenue Overview',
      metrics: [
        {
          name: 'Gross Revenue',
          value: 'currency',
          breakdown: ['iap', 'subscriptions', 'ads'],
          chart: 'stackedArea',
          timeRange: '30 days'
        },
        {
          name: 'Net Revenue',
          value: 'currency',
          afterFees: true
        },
        {
          name: 'ARPDAU',
          value: 'currency',
          trend: 'line',
          benchmark: true
        },
        {
          name: 'ARPPU',
          value: 'currency',
          trend: 'line'
        },
        {
          name: 'Paying Users %',
          value: 'percentage',
          trend: 'line'
        }
      ]
    },

    iapMetrics: {
      title: 'In-App Purchases',
      metrics: [
        {
          name: 'Top Selling Items',
          table: 'rankedList',
          columns: ['item', 'sales', 'revenue', 'conversion']
        },
        {
          name: 'Revenue by Category',
          breakdown: ['currency', 'cosmetics', 'battlePass', 'bundles'],
          chart: 'pieChart'
        },
        {
          name: 'Price Point Performance',
          breakdown: ['$0.99', '$4.99', '$9.99', '$19.99', '$49.99', '$99.99'],
          chart: 'bar'
        },
        {
          name: 'First Purchase Conversion',
          funnel: ['installed', 'shopViewed', 'itemViewed', 'purchased'],
          chart: 'funnel'
        }
      ]
    },

    ltv: {
      title: 'Lifetime Value',
      metrics: [
        {
          name: 'LTV by Cohort',
          chart: 'lineChart',
          cohorts: ['byWeek'],
          projection: true
        },
        {
          name: 'LTV by Source',
          breakdown: ['organic', 'facebook', 'google', 'apple'],
          chart: 'bar'
        },
        {
          name: 'LTV:CAC Ratio',
          byChannel: true,
          target: 3.0
        }
      ]
    },

    battlePass: {
      title: 'Battle Pass',
      metrics: [
        {
          name: 'Premium Conversion',
          value: 'percentage',
          trend: 'line'
        },
        {
          name: 'Tier Distribution',
          chart: 'histogram',
          milestone: 'highlightFinalTier'
        },
        {
          name: 'Tier Skip Revenue',
          value: 'currency'
        },
        {
          name: 'Pass Completion Rate',
          value: 'percentage',
          breakdown: ['free', 'premium']
        }
      ]
    }
  }
};
```

## LiveOps Dashboard

```typescript
const LIVEOPS_DASHBOARD = {
  name: 'LiveOps Performance',
  audience: ['liveops', 'product'],
  refreshRate: 'realTime',

  sections: {
    questMetrics: {
      title: 'Quest Performance',
      metrics: [
        {
          name: 'Daily Quest Completion',
          rate: 'percentage',
          breakdown: ['easy', 'medium', 'hard'],
          chart: 'bar'
        },
        {
          name: 'Weekly Quest Completion',
          rate: 'percentage',
          chart: 'bar'
        },
        {
          name: 'Quest Engagement Rate',
          definition: 'players with quest progress / DAU',
          trend: 'line'
        },
        {
          name: 'Quest Time to Complete',
          distribution: 'histogram',
          byDifficulty: true
        },
        {
          name: 'Reroll Rate',
          rate: 'percentage',
          trend: 'line'
        }
      ]
    },

    eventMetrics: {
      title: 'Event Performance',
      metrics: [
        {
          name: 'Active Event Participation',
          rate: 'percentage',
          trend: 'line'
        },
        {
          name: 'Event Quest Completion',
          rate: 'percentage',
          byQuest: true
        },
        {
          name: 'Event Track Progress',
          distribution: 'histogram',
          byLevel: true
        },
        {
          name: 'Event Shop Revenue',
          value: 'currency',
          topItems: true
        }
      ]
    },

    shopMetrics: {
      title: 'Shop Performance',
      metrics: [
        {
          name: 'Featured Item Performance',
          table: 'currentFeatured',
          columns: ['item', 'views', 'purchases', 'conversion']
        },
        {
          name: 'Daily Rotation Performance',
          trend: 'byDay',
          comparison: 'previousWeek'
        },
        {
          name: 'Bundle Performance',
          table: 'activeBundles',
          conversion: true
        },
        {
          name: 'Shop View to Purchase',
          funnel: true
        }
      ]
    },

    loginRewards: {
      title: 'Login & Retention Features',
      metrics: [
        {
          name: 'Login Reward Claims',
          rate: 'percentage',
          byDay: true
        },
        {
          name: 'Streak Distribution',
          chart: 'histogram',
          milestones: [7, 14, 21, 28]
        },
        {
          name: 'First Win Rate',
          rate: 'percentage',
          trend: 'line'
        },
        {
          name: 'Battle Pass XP Sources',
          breakdown: ['matches', 'dailyQuests', 'weeklyQuests', 'events'],
          chart: 'donut'
        }
      ]
    }
  }
};
```

## Gameplay Dashboard

```typescript
const GAMEPLAY_DASHBOARD = {
  name: 'Gameplay Analytics',
  audience: ['design', 'product', 'engineering'],
  refreshRate: 'hourly',

  sections: {
    matchMetrics: {
      title: 'Match Analytics',
      metrics: [
        {
          name: 'Matches Played',
          total: true,
          trend: 'line',
          breakdown: ['solo', 'duo', 'squad']
        },
        {
          name: 'Match Duration',
          average: true,
          distribution: 'histogram'
        },
        {
          name: 'Players per Match',
          average: true,
          fillRate: true
        },
        {
          name: 'Early Quit Rate',
          rate: 'percentage',
          definition: 'quit before top 50%'
        },
        {
          name: 'Match Completion Rate',
          rate: 'percentage'
        }
      ]
    },

    balanceMetrics: {
      title: 'Balance Analytics',
      metrics: [
        {
          name: 'Ability Usage Rate',
          breakdown: 'allAbilities',
          chart: 'bar',
          sortBy: 'usage'
        },
        {
          name: 'Ability Win Rate',
          breakdown: 'allAbilities',
          chart: 'scatter',
          axes: ['usageRate', 'winRate'],
          ideal: 'center'
        },
        {
          name: 'Kill Distribution',
          histogram: true,
          average: true
        },
        {
          name: 'Damage per Match',
          histogram: true,
          average: true
        },
        {
          name: 'Loot Acquisition Rate',
          breakdown: ['common', 'rare', 'epic', 'legendary'],
          chart: 'bar'
        }
      ]
    },

    progression: {
      title: 'Player Progression',
      metrics: [
        {
          name: 'Player Level Distribution',
          histogram: true,
          milestones: [10, 25, 50, 100]
        },
        {
          name: 'XP Earning Rate',
          average: true,
          breakdown: 'source'
        },
        {
          name: 'Skill Rating Distribution',
          histogram: true,
          tiers: true
        },
        {
          name: 'Ranked Participation',
          rate: 'percentage',
          byLevel: true
        }
      ]
    }
  }
};
```

## Real-Time Monitoring

```typescript
const REALTIME_MONITORING = {
  name: 'Real-Time Monitoring',
  audience: ['liveops', 'engineering', 'oncall'],
  refreshRate: '30 seconds',

  sections: {
    currentActivity: {
      title: 'Current Activity',
      metrics: [
        {
          name: 'CCU (Concurrent Users)',
          value: 'number',
          chart: 'liveArea',
          history: '1 hour'
        },
        {
          name: 'Active Matches',
          value: 'number',
          liveUpdate: true
        },
        {
          name: 'Players in Queue',
          value: 'number',
          breakdown: 'byMode'
        },
        {
          name: 'Average Queue Time',
          value: 'seconds',
          alert: { above: '60s' }
        }
      ]
    },

    systemHealth: {
      title: 'System Health',
      metrics: [
        {
          name: 'Server Response Time',
          value: 'milliseconds',
          percentiles: ['p50', 'p95', 'p99'],
          alert: { p99Above: '500ms' }
        },
        {
          name: 'Error Rate',
          value: 'percentage',
          chart: 'liveLine',
          alert: { above: '1%' }
        },
        {
          name: 'Match Server Capacity',
          value: 'percentage',
          alert: { above: '80%' }
        },
        {
          name: 'API Health',
          endpoints: 'critical',
          status: 'upDown'
        }
      ]
    },

    transactionHealth: {
      title: 'Transaction Health',
      metrics: [
        {
          name: 'Purchase Success Rate',
          value: 'percentage',
          alert: { below: '95%' }
        },
        {
          name: 'Failed Transactions',
          count: true,
          details: 'expandable'
        },
        {
          name: 'Revenue per Minute',
          value: 'currency',
          chart: 'liveLine'
        }
      ]
    },

    alerts: {
      title: 'Active Alerts',
      display: 'list',
      severity: ['critical', 'warning', 'info'],
      actions: ['acknowledge', 'escalate', 'resolve']
    }
  }
};
```

## Drill-Down Capability

```typescript
const DRILLDOWN_CAPABILITY = {
  globalFilters: {
    dateRange: {
      presets: ['today', 'yesterday', 'last7Days', 'last30Days', 'custom'],
      comparison: true
    },
    platform: ['all', 'iOS', 'Android'],
    region: ['all', 'NA', 'EU', 'APAC', 'LATAM', 'other'],
    playerSegment: ['all', 'new', 'casual', 'regular', 'hardcore', 'spender'],
    appVersion: 'dropdown',
    abTestGroup: 'dropdown'
  },

  interactiveCharts: {
    hover: 'showDetails',
    click: 'drillDown',
    doubleClick: 'zoomIn',
    rightClick: 'contextMenu'
  },

  drillDownPaths: {
    revenue: ['total', 'byCategory', 'byItem', 'byTransaction'],
    users: ['total', 'bySegment', 'byCohort', 'individual'],
    matches: ['total', 'byMode', 'byMap', 'individual'],
    quests: ['total', 'byType', 'byQuest', 'individual']
  },

  playerLookup: {
    searchBy: ['playerId', 'displayName'],
    display: [
      'accountInfo',
      'sessionHistory',
      'purchaseHistory',
      'matchHistory',
      'progressData'
    ],
    actions: ['none'] // View only in analytics
  }
};
```

## Cohort Analysis

```typescript
const COHORT_ANALYSIS = {
  cohortDefinitions: {
    byJoinDate: {
      granularity: ['day', 'week', 'month'],
      default: 'week'
    },
    byFirstPurchase: {
      granularity: ['week', 'month'],
      default: 'week'
    },
    bySource: {
      values: ['organic', 'facebook', 'google', 'referral']
    },
    byPlatform: {
      values: ['iOS', 'Android']
    },
    byRegion: {
      values: ['NA', 'EU', 'APAC', 'LATAM']
    }
  },

  cohortMetrics: {
    retention: {
      periods: ['D1', 'D3', 'D7', 'D14', 'D30', 'D60', 'D90'],
      visualization: 'heatmap'
    },
    ltv: {
      periods: ['D7', 'D14', 'D30', 'D60', 'D90', 'D180'],
      projection: true
    },
    engagement: {
      metrics: ['sessionsPerWeek', 'matchesPerWeek', 'purchaseRate']
    },
    monetization: {
      metrics: ['conversionRate', 'arpu', 'arppu', 'transactionsPerUser']
    }
  },

  visualization: {
    heatmap: {
      colorScale: ['red', 'yellow', 'green'],
      annotations: true
    },
    lineChart: {
      comparison: true,
      benchmark: true
    },
    table: {
      export: true,
      sorting: true
    }
  }
};
```

## Funnel Visualization

```typescript
const FUNNEL_VISUALIZATION = {
  predefinedFunnels: {
    onboarding: {
      name: 'New User Onboarding',
      steps: [
        'app_install',
        'app_open',
        'tutorial_start',
        'tutorial_complete',
        'first_match_start',
        'first_match_complete'
      ],
      target: { tutorialComplete: 70, firstMatchComplete: 50 }
    },

    firstPurchase: {
      name: 'First Purchase',
      steps: [
        'shop_view',
        'item_view',
        'add_to_cart',
        'checkout_start',
        'purchase_complete'
      ],
      target: { purchaseComplete: 5 }
    },

    battlePassConversion: {
      name: 'Battle Pass Conversion',
      steps: [
        'bp_view',
        'premium_preview',
        'purchase_prompt',
        'purchase_complete'
      ],
      target: { purchaseComplete: 15 }
    },

    matchFlow: {
      name: 'Match Flow',
      steps: [
        'queue_join',
        'match_start',
        'match_active',
        'match_complete',
        'results_view'
      ],
      target: { matchComplete: 95 }
    },

    questEngagement: {
      name: 'Quest Engagement',
      steps: [
        'quest_view',
        'quest_progress_start',
        'quest_50_percent',
        'quest_complete',
        'reward_claim'
      ],
      target: { questComplete: 60 }
    }
  },

  customFunnels: {
    enabled: true,
    eventSelection: 'dropdown',
    maxSteps: 10,
    save: true,
    share: true
  },

  funnelAnalysis: {
    conversionRates: true,
    dropOffPoints: true,
    timeToConvert: true,
    segmentation: true,
    comparison: true
  }
};
```

## Export and Reporting

```typescript
const EXPORT_REPORTING = {
  exportFormats: {
    csv: {
      enabled: true,
      allData: true,
      filtered: true
    },
    excel: {
      enabled: true,
      formatted: true,
      charts: false
    },
    pdf: {
      enabled: true,
      dashboardSnapshot: true
    },
    json: {
      enabled: true,
      api: true
    }
  },

  scheduledReports: {
    enabled: true,
    frequency: ['daily', 'weekly', 'monthly'],
    recipients: 'emailList',
    format: ['pdf', 'csv'],
    customization: {
      dashboards: true,
      dateRange: true,
      filters: true
    }
  },

  apiAccess: {
    enabled: true,
    authentication: 'apiKey',
    rateLimit: '1000 req/min',
    endpoints: {
      metrics: '/api/analytics/metrics',
      cohorts: '/api/analytics/cohorts',
      funnels: '/api/analytics/funnels',
      export: '/api/analytics/export'
    },
    documentation: 'swagger'
  },

  dataWarehouseAccess: {
    enabled: true,
    tool: 'bigQuery / snowflake',
    selfService: true,
    governance: true
  }
};
```

## Alerting System

```typescript
const ALERTING_SYSTEM = {
  alertTypes: {
    threshold: {
      description: 'Metric crosses defined threshold',
      examples: ['DAU < 10000', 'Error Rate > 5%']
    },
    anomaly: {
      description: 'ML-detected anomaly from baseline',
      sensitivity: ['low', 'medium', 'high']
    },
    trend: {
      description: 'Sustained movement in metric',
      examples: ['Retention declining 3 days']
    },
    composite: {
      description: 'Multiple conditions combined',
      logic: ['AND', 'OR']
    }
  },

  alertConfiguration: {
    name: 'string',
    metric: 'dropdown',
    condition: 'builder',
    threshold: 'number',
    window: 'duration',
    severity: ['info', 'warning', 'critical'],
    notification: ['email', 'slack', 'pagerduty']
  },

  predefinedAlerts: [
    { name: 'DAU Drop', metric: 'DAU', condition: 'dropBy', value: '20%', severity: 'critical' },
    { name: 'Revenue Spike', metric: 'revenue', condition: 'increase', value: '50%', severity: 'info' },
    { name: 'Error Spike', metric: 'errorRate', condition: 'above', value: '5%', severity: 'critical' },
    { name: 'Queue Time High', metric: 'queueTime', condition: 'above', value: '120s', severity: 'warning' },
    { name: 'Purchase Failures', metric: 'purchaseFailRate', condition: 'above', value: '5%', severity: 'critical' }
  ],

  alertManagement: {
    acknowledge: true,
    snooze: true,
    escalate: true,
    resolve: true,
    history: true
  }
};
```

## Access Control

```typescript
const ACCESS_CONTROL = {
  roles: {
    viewer: {
      permissions: ['viewDashboards', 'export'],
      dashboards: 'assigned'
    },
    analyst: {
      permissions: ['viewAll', 'export', 'customFunnels', 'scheduledReports'],
      dashboards: 'all'
    },
    liveops: {
      permissions: ['viewAll', 'export', 'realTimeMonitoring', 'playerLookup'],
      dashboards: 'all'
    },
    admin: {
      permissions: ['*'],
      dashboards: 'all'
    }
  },

  dashboardAccess: {
    executive: ['executives', 'admins'],
    revenue: ['business', 'product', 'executives', 'admins'],
    playerMetrics: ['product', 'analytics', 'liveops', 'admins'],
    liveops: ['liveops', 'product', 'admins'],
    gameplay: ['design', 'product', 'engineering', 'admins'],
    realtime: ['liveops', 'engineering', 'oncall', 'admins']
  },

  audit: {
    logAccess: true,
    logExports: true,
    logQueries: true,
    retention: '1 year'
  }
};
```

## Technical Implementation

```typescript
const TECHNICAL_SPEC = {
  dataInfrastructure: {
    streaming: 'kafka / kinesis',
    processing: 'spark / flink',
    storage: 'bigQuery / snowflake',
    caching: 'redis',
    visualization: 'custom + looker / tableau'
  },

  eventTracking: {
    client: 'sdkIntegration',
    server: 'directLogging',
    schema: 'definedAndVersioned',
    validation: 'atIngestion'
  },

  performance: {
    dashboardLoadTime: '<3 seconds',
    queryTimeout: '30 seconds',
    caching: 'aggressive',
    precomputation: 'commonMetrics'
  },

  reliability: {
    uptime: '99.9%',
    dataFreshness: 'slaPerTier',
    backfill: 'supported',
    reconciliation: 'daily'
  },

  security: {
    encryption: 'atRestAndInTransit',
    piiHandling: 'anonymized',
    gdprCompliance: true,
    accessLogging: true
  }
};
```
