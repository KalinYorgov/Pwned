# Game Analytics Specification

## Overview

The Game Analytics system tracks player behavior, game events, and business metrics to inform game design decisions, balance updates, and business strategy. This specification covers event tracking, data pipelines, analysis dashboards, and privacy compliance.

## Goals

1. **Data-Driven Decisions**: Provide actionable insights for game balance and features
2. **Player Understanding**: Deep understanding of player behavior and preferences
3. **Business Intelligence**: Track monetization, retention, and growth metrics
4. **Real-Time Monitoring**: Live visibility into game health and player activity
5. **Privacy Compliance**: GDPR and CCPA compliant data collection

---

## Event Taxonomy

### Session Events

```typescript
interface SessionStartEvent {
  eventName: 'session_start';
  timestamp: string;
  properties: {
    userId: string;
    sessionId: string;
    deviceId: string;
    platform: 'ios' | 'android';
    osVersion: string;
    deviceModel: string;
    appVersion: string;
    buildNumber: string;
    isFirstSession: boolean;
    daysSinceLastSession: number;
    sessionCount: number;
    installDate: string;
    country: string;
    language: string;
    networkType: 'wifi' | 'cellular' | 'unknown';
  };
}

interface SessionEndEvent {
  eventName: 'session_end';
  timestamp: string;
  properties: {
    userId: string;
    sessionId: string;
    sessionLengthSeconds: number;
    matchesPlayed: number;
    goldEarned: number;
    xpEarned: number;
    itemsPurchased: number;
    screenFlow: string[]; // Sequence of screens visited
    endReason: 'user_quit' | 'app_backgrounded' | 'crash' | 'timeout';
  };
}
```

### Match Events

```typescript
interface MatchStartEvent {
  eventName: 'match_start';
  timestamp: string;
  properties: {
    userId: string;
    matchId: string;
    gameMode: string;
    queueTime: number; // Seconds
    teamSize: number;
    isRanked: boolean;
    region: string;
    serverLatency: number;
    loadTime: number;
    selectedCharacter: string;
    selectedLoadout: string;
  };
}

interface MatchEndEvent {
  eventName: 'match_end';
  timestamp: string;
  properties: {
    userId: string;
    matchId: string;
    gameMode: string;

    // Performance
    placement: number;
    playersInMatch: number;
    kills: number;
    deaths: number;
    assists: number;
    damageDealt: number;
    damageTaken: number;
    healingDone: number;
    revives: number;

    // Survival
    survivalTimeSeconds: number;
    distanceTraveled: number;
    zonesVisited: number;
    finalZone: number;

    // Abilities
    abilitiesUsed: {
      abilityId: string;
      usageCount: number;
      damage: number;
      kills: number;
    }[];

    // Loot
    itemsLooted: number;
    rarityBreakdown: Record<string, number>;
    weaponsUsed: string[];

    // Outcome
    result: 'win' | 'loss' | 'disconnect';
    disconnected: boolean;
    reconnected: boolean;

    // Rewards
    xpEarned: number;
    bpXpEarned: number;
    goldEarned: number;
    questProgress: string[];

    // Meta
    matchDuration: number;
    averageFPS: number;
    averageLatency: number;
    packetLossPercent: number;
  };
}

interface KillEvent {
  eventName: 'kill';
  timestamp: string;
  properties: {
    userId: string;
    matchId: string;
    victimId: string;
    weaponId: string;
    abilityId?: string;
    distance: number;
    isHeadshot: boolean;
    attackerHealth: number;
    victimHealth: number;
    killTime: number; // Seconds into match
    killLocation: { x: number; y: number };
    zonesRemaining: number;
    playersRemaining: number;
  };
}

interface DeathEvent {
  eventName: 'death';
  timestamp: string;
  properties: {
    userId: string;
    matchId: string;
    killerId?: string;
    causeOfDeath: 'player' | 'storm' | 'fall' | 'environment' | 'disconnect';
    weaponId?: string;
    deathTime: number;
    deathLocation: { x: number; y: number };
    placement: number;
    wasRevived: boolean;
  };
}
```

### Progression Events

```typescript
interface LevelUpEvent {
  eventName: 'level_up';
  timestamp: string;
  properties: {
    userId: string;
    newLevel: number;
    previousLevel: number;
    xpRequired: number;
    totalXpEarned: number;
    timeSinceLastLevel: number;
    rewardsEarned: string[];
  };
}

interface BattlePassProgressEvent {
  eventName: 'battle_pass_progress';
  timestamp: string;
  properties: {
    userId: string;
    seasonId: string;
    newTier: number;
    previousTier: number;
    isPremium: boolean;
    xpEarned: number;
    totalBpXp: number;
    rewardsClaimed: string[];
    daysIntoSeason: number;
  };
}

interface QuestCompletedEvent {
  eventName: 'quest_completed';
  timestamp: string;
  properties: {
    userId: string;
    questId: string;
    questType: 'daily' | 'weekly' | 'event' | 'achievement';
    questCategory: string;
    timeToComplete: number; // Seconds from assignment
    matchesRequired: number;
    rewardsEarned: {
      type: string;
      amount: number;
    }[];
  };
}

interface RankedProgressEvent {
  eventName: 'ranked_progress';
  timestamp: string;
  properties: {
    userId: string;
    newTier: string;
    newDivision: number;
    previousTier: string;
    previousDivision: number;
    rpChange: number;
    currentRp: number;
    isPromotion: boolean;
    isDemotion: boolean;
    matchesAtRank: number;
    seasonId: string;
  };
}
```

### Economy Events

```typescript
interface PurchaseEvent {
  eventName: 'purchase';
  timestamp: string;
  properties: {
    userId: string;
    transactionId: string;
    productId: string;
    productType: 'iap' | 'gold' | 'gems' | 'battle_pass' | 'bundle';
    price: number;
    currency: string;
    priceUSD: number;
    platform: 'ios' | 'android';
    isFirstPurchase: boolean;
    daysSinceInstall: number;
    sessionNumber: number;
    triggerLocation: string; // Where in game purchase was initiated
    goldBalanceBefore?: number;
    gemsBalanceBefore?: number;
  };
}

interface VirtualCurrencySpendEvent {
  eventName: 'currency_spend';
  timestamp: string;
  properties: {
    userId: string;
    currencyType: 'gold' | 'gems' | 'event_tokens';
    amount: number;
    balanceBefore: number;
    balanceAfter: number;
    itemPurchased: string;
    itemCategory: string;
    spendLocation: string;
  };
}

interface VirtualCurrencyEarnEvent {
  eventName: 'currency_earn';
  timestamp: string;
  properties: {
    userId: string;
    currencyType: 'gold' | 'gems' | 'event_tokens';
    amount: number;
    balanceBefore: number;
    balanceAfter: number;
    earnSource: 'match' | 'quest' | 'battle_pass' | 'daily_login' | 'purchase' | 'gift';
    earnContext?: string;
  };
}
```

### Engagement Events

```typescript
interface FeatureUsageEvent {
  eventName: 'feature_usage';
  timestamp: string;
  properties: {
    userId: string;
    featureName: string;
    featureCategory: string;
    interactionType: 'view' | 'click' | 'complete';
    duration?: number;
    metadata?: Record<string, any>;
  };
}

interface TutorialEvent {
  eventName: 'tutorial';
  timestamp: string;
  properties: {
    userId: string;
    step: number;
    stepName: string;
    action: 'start' | 'complete' | 'skip' | 'fail';
    timeOnStep: number;
    totalTutorialTime: number;
    attempts?: number;
  };
}

interface SocialEvent {
  eventName: 'social';
  timestamp: string;
  properties: {
    userId: string;
    actionType: 'friend_request' | 'party_invite' | 'party_join' | 'share' | 'invite_sent';
    targetUserId?: string;
    result: 'success' | 'declined' | 'expired' | 'error';
    platform?: string;
  };
}

interface MenuNavigationEvent {
  eventName: 'menu_navigation';
  timestamp: string;
  properties: {
    userId: string;
    screenName: string;
    previousScreen: string;
    timeOnPreviousScreen: number;
    navigationPath: string[];
    sessionDepth: number;
  };
}
```

### Technical Events

```typescript
interface PerformanceEvent {
  eventName: 'performance';
  timestamp: string;
  properties: {
    userId: string;
    sessionId: string;
    averageFPS: number;
    minFPS: number;
    maxFPS: number;
    frameDrops: number;
    memoryUsageMB: number;
    cpuUsagePercent: number;
    thermalState: 'nominal' | 'fair' | 'serious' | 'critical';
    batteryLevel: number;
    batteryCharging: boolean;
    duration: number;
    context: 'menu' | 'match' | 'loading';
  };
}

interface ErrorEvent {
  eventName: 'error';
  timestamp: string;
  properties: {
    userId: string;
    sessionId: string;
    errorType: string;
    errorMessage: string;
    errorCode?: string;
    stackTrace?: string;
    context: string;
    severity: 'info' | 'warning' | 'error' | 'fatal';
    deviceState: Record<string, any>;
  };
}

interface NetworkEvent {
  eventName: 'network';
  timestamp: string;
  properties: {
    userId: string;
    eventType: 'disconnect' | 'reconnect' | 'latency_spike' | 'packet_loss';
    latency?: number;
    packetLoss?: number;
    networkType: string;
    region: string;
    duration?: number;
    matchId?: string;
  };
}
```

---

## Data Pipeline Architecture

### Event Collection

```typescript
class AnalyticsClient {
  private eventQueue: AnalyticsEvent[] = [];
  private readonly MAX_QUEUE_SIZE = 100;
  private readonly FLUSH_INTERVAL = 30000; // 30 seconds

  // Track an event
  track(event: AnalyticsEvent): void {
    // Add common properties
    const enrichedEvent = this.enrichEvent(event);

    // Validate event
    if (!this.validateEvent(enrichedEvent)) {
      console.warn('Invalid analytics event:', event.eventName);
      return;
    }

    // Add to queue
    this.eventQueue.push(enrichedEvent);

    // Flush if queue is full
    if (this.eventQueue.length >= this.MAX_QUEUE_SIZE) {
      this.flush();
    }
  }

  private enrichEvent(event: AnalyticsEvent): AnalyticsEvent {
    return {
      ...event,
      properties: {
        ...event.properties,
        // Common properties added to all events
        _timestamp: Date.now(),
        _sessionId: SessionManager.getSessionId(),
        _platform: Platform.OS,
        _appVersion: Config.APP_VERSION,
        _buildNumber: Config.BUILD_NUMBER,
        _deviceId: DeviceInfo.getDeviceId(),
        _country: GeoService.getCountry(),
        _abTests: ABTestManager.getActiveTests()
      }
    };
  }

  async flush(): Promise<void> {
    if (this.eventQueue.length === 0) return;

    const batch = [...this.eventQueue];
    this.eventQueue = [];

    try {
      await this.sendBatch(batch);
    } catch (error) {
      // Store failed events locally for retry
      await this.storeForRetry(batch);
    }
  }

  private async sendBatch(events: AnalyticsEvent[]): Promise<void> {
    // Compress batch
    const compressed = await this.compressBatch(events);

    // Send to ingestion endpoint
    await fetch(`${Config.ANALYTICS_ENDPOINT}/v1/events`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Content-Encoding': 'gzip',
        'X-API-Key': Config.ANALYTICS_API_KEY
      },
      body: compressed
    });
  }
}
```

### Data Ingestion

```typescript
// Server-side ingestion service
class EventIngestionService {
  async ingestEvents(
    events: AnalyticsEvent[],
    source: string
  ): Promise<IngestionResult> {
    const results = {
      accepted: 0,
      rejected: 0,
      errors: [] as string[]
    };

    for (const event of events) {
      try {
        // Validate schema
        await this.validateSchema(event);

        // Sanitize PII
        const sanitized = this.sanitizePII(event);

        // Enrich with server-side data
        const enriched = await this.serverEnrich(sanitized);

        // Write to stream
        await this.writeToStream(enriched);

        results.accepted++;
      } catch (error) {
        results.rejected++;
        results.errors.push(`${event.eventName}: ${error.message}`);
      }
    }

    return results;
  }

  private sanitizePII(event: AnalyticsEvent): AnalyticsEvent {
    // Hash user IDs for external analytics
    // Remove IP addresses
    // Redact any personal information
    return {
      ...event,
      properties: {
        ...event.properties,
        userId: this.hashUserId(event.properties.userId),
        deviceId: this.hashDeviceId(event.properties.deviceId)
      }
    };
  }

  private async writeToStream(event: AnalyticsEvent): Promise<void> {
    // Write to Kafka/Kinesis for real-time processing
    await KafkaProducer.send({
      topic: `analytics.${event.eventName}`,
      messages: [{ value: JSON.stringify(event) }]
    });

    // Also write to data lake for batch processing
    await S3Client.putObject({
      Bucket: 'analytics-data-lake',
      Key: `events/${event.eventName}/${this.getDatePartition()}/${event.timestamp}.json`,
      Body: JSON.stringify(event)
    });
  }
}
```

### Data Processing

```typescript
// Apache Spark job for batch processing
const ANALYTICS_JOBS = {
  // Daily aggregation job
  dailyAggregation: {
    schedule: '0 2 * * *', // 2 AM daily
    queries: [
      // DAU calculation
      `
        SELECT
          DATE(timestamp) as date,
          COUNT(DISTINCT userId) as dau,
          COUNT(DISTINCT CASE WHEN isFirstSession THEN userId END) as new_users,
          COUNT(*) as total_sessions,
          AVG(sessionLengthSeconds) as avg_session_length
        FROM session_events
        WHERE DATE(timestamp) = CURRENT_DATE - 1
        GROUP BY DATE(timestamp)
      `,

      // Match metrics
      `
        SELECT
          DATE(timestamp) as date,
          gameMode,
          COUNT(*) as matches_played,
          AVG(survivalTimeSeconds) as avg_survival,
          AVG(kills) as avg_kills,
          SUM(CASE WHEN placement = 1 THEN 1 ELSE 0 END) as total_wins
        FROM match_end_events
        WHERE DATE(timestamp) = CURRENT_DATE - 1
        GROUP BY DATE(timestamp), gameMode
      `,

      // Revenue metrics
      `
        SELECT
          DATE(timestamp) as date,
          SUM(priceUSD) as revenue,
          COUNT(DISTINCT userId) as paying_users,
          COUNT(*) as transactions,
          COUNT(DISTINCT CASE WHEN isFirstPurchase THEN userId END) as new_payers
        FROM purchase_events
        WHERE DATE(timestamp) = CURRENT_DATE - 1
        GROUP BY DATE(timestamp)
      `
    ]
  },

  // Retention cohort calculation
  retentionCohorts: {
    schedule: '0 4 * * *', // 4 AM daily
    query: `
      WITH cohorts AS (
        SELECT
          userId,
          DATE(MIN(timestamp)) as cohort_date
        FROM session_events
        WHERE isFirstSession = true
        GROUP BY userId
      ),
      activity AS (
        SELECT
          userId,
          DATE(timestamp) as activity_date
        FROM session_events
        GROUP BY userId, DATE(timestamp)
      )
      SELECT
        c.cohort_date,
        DATEDIFF(a.activity_date, c.cohort_date) as day_number,
        COUNT(DISTINCT a.userId) as retained_users
      FROM cohorts c
      JOIN activity a ON c.userId = a.userId
      WHERE c.cohort_date >= CURRENT_DATE - 30
      GROUP BY c.cohort_date, DATEDIFF(a.activity_date, c.cohort_date)
    `
  },

  // Ability balance analysis
  abilityBalance: {
    schedule: '0 5 * * *', // 5 AM daily
    query: `
      SELECT
        abilityId,
        COUNT(DISTINCT matchId) as matches_with_ability,
        COUNT(*) as total_uses,
        SUM(damage) as total_damage,
        SUM(kills) as total_kills,
        AVG(kills) as avg_kills_per_use,
        -- Calculate win rate for players who used this ability
        AVG(CASE WHEN placement = 1 THEN 1.0 ELSE 0.0 END) as win_rate
      FROM match_ability_usage
      JOIN match_end_events USING (matchId, userId)
      WHERE DATE(timestamp) >= CURRENT_DATE - 7
      GROUP BY abilityId
      ORDER BY total_uses DESC
    `
  }
};
```

---

## Dashboards and Metrics

### Key Performance Indicators (KPIs)

```typescript
const CORE_KPIS = {
  // Engagement
  dau: {
    name: 'Daily Active Users',
    query: 'COUNT(DISTINCT userId) FROM sessions WHERE date = today',
    target: 'maximize',
    alerts: { warning: -10, critical: -20 } // % change
  },
  wau: {
    name: 'Weekly Active Users',
    query: 'COUNT(DISTINCT userId) FROM sessions WHERE date >= today - 7'
  },
  mau: {
    name: 'Monthly Active Users',
    query: 'COUNT(DISTINCT userId) FROM sessions WHERE date >= today - 30'
  },
  stickiness: {
    name: 'Stickiness (DAU/MAU)',
    formula: 'DAU / MAU * 100',
    target: '> 20%'
  },
  avgSessionLength: {
    name: 'Average Session Length',
    query: 'AVG(sessionLengthSeconds) / 60',
    unit: 'minutes',
    target: '> 15 minutes'
  },
  sessionsPerDAU: {
    name: 'Sessions per DAU',
    formula: 'total_sessions / DAU',
    target: '> 1.5'
  },

  // Retention
  d1Retention: {
    name: 'Day 1 Retention',
    formula: 'users_returned_day_1 / users_installed * 100',
    target: '> 40%'
  },
  d7Retention: {
    name: 'Day 7 Retention',
    target: '> 20%'
  },
  d30Retention: {
    name: 'Day 30 Retention',
    target: '> 10%'
  },

  // Monetization
  revenue: {
    name: 'Daily Revenue',
    query: 'SUM(priceUSD) FROM purchases WHERE date = today'
  },
  arpu: {
    name: 'Average Revenue Per User',
    formula: 'revenue / DAU'
  },
  arppu: {
    name: 'Average Revenue Per Paying User',
    formula: 'revenue / paying_users'
  },
  conversionRate: {
    name: 'Payer Conversion Rate',
    formula: 'paying_users / DAU * 100',
    target: '> 3%'
  },

  // Game Health
  matchesPerDAU: {
    name: 'Matches per DAU',
    target: '> 3'
  },
  averageMatchDuration: {
    name: 'Average Match Duration',
    unit: 'minutes'
  },
  winRateDistribution: {
    name: 'Win Rate Distribution',
    description: 'Should approximate normal distribution'
  },
  disconnectRate: {
    name: 'Disconnect Rate',
    formula: 'disconnected_matches / total_matches * 100',
    target: '< 5%'
  }
};
```

### Dashboard Definitions

```typescript
const DASHBOARDS = {
  // Executive Overview
  executive: {
    name: 'Executive Overview',
    refresh: '1h',
    widgets: [
      {
        type: 'metric_card',
        metrics: ['dau', 'revenue', 'd1Retention', 'conversionRate'],
        comparison: 'week_over_week'
      },
      {
        type: 'time_series',
        metric: 'dau',
        timeRange: '30d',
        granularity: 'day'
      },
      {
        type: 'time_series',
        metric: 'revenue',
        timeRange: '30d',
        granularity: 'day'
      },
      {
        type: 'cohort_retention',
        days: [1, 3, 7, 14, 30],
        cohorts: 8 // weeks
      }
    ]
  },

  // Player Behavior
  playerBehavior: {
    name: 'Player Behavior',
    refresh: '15m',
    widgets: [
      {
        type: 'funnel',
        steps: [
          'app_install',
          'tutorial_start',
          'tutorial_complete',
          'first_match',
          'first_win',
          'first_purchase'
        ]
      },
      {
        type: 'distribution',
        metric: 'sessionsPerDay',
        buckets: [1, 2, 3, 5, 10, 20]
      },
      {
        type: 'heatmap',
        metric: 'sessionStart',
        dimensions: ['dayOfWeek', 'hourOfDay']
      },
      {
        type: 'sankey',
        source: 'menu_navigation',
        limit: 10
      }
    ]
  },

  // Game Balance
  gameBalance: {
    name: 'Game Balance',
    refresh: '1h',
    widgets: [
      {
        type: 'table',
        title: 'Ability Performance',
        columns: ['ability', 'pickRate', 'winRate', 'avgKills', 'avgDamage'],
        sortBy: 'pickRate',
        alerts: {
          winRate: { high: 0.55, low: 0.45 }
        }
      },
      {
        type: 'table',
        title: 'Weapon Performance',
        columns: ['weapon', 'pickRate', 'killsPerMatch', 'headshotRate'],
        sortBy: 'pickRate'
      },
      {
        type: 'bar_chart',
        title: 'Game Mode Popularity',
        metric: 'matchesPlayed',
        dimension: 'gameMode'
      },
      {
        type: 'distribution',
        title: 'Match Duration Distribution',
        metric: 'matchDuration',
        buckets: [5, 10, 15, 20, 25, 30]
      }
    ]
  },

  // Monetization
  monetization: {
    name: 'Monetization',
    refresh: '30m',
    widgets: [
      {
        type: 'metric_card',
        metrics: ['revenue', 'arppu', 'conversionRate', 'ltv']
      },
      {
        type: 'time_series',
        metrics: ['revenue', 'transactions'],
        timeRange: '30d'
      },
      {
        type: 'pie_chart',
        title: 'Revenue by Product Type',
        dimension: 'productType',
        metric: 'revenue'
      },
      {
        type: 'funnel',
        title: 'Purchase Funnel',
        steps: [
          'store_view',
          'product_view',
          'purchase_start',
          'purchase_complete'
        ]
      },
      {
        type: 'table',
        title: 'Top Selling Items',
        columns: ['item', 'purchases', 'revenue'],
        limit: 20
      }
    ]
  },

  // Technical Health
  technical: {
    name: 'Technical Health',
    refresh: '5m',
    widgets: [
      {
        type: 'metric_card',
        metrics: ['crashRate', 'avgFPS', 'avgLatency', 'disconnectRate']
      },
      {
        type: 'time_series',
        title: 'Crash Rate by Version',
        metric: 'crashRate',
        dimension: 'appVersion'
      },
      {
        type: 'distribution',
        title: 'FPS Distribution',
        metric: 'averageFPS',
        buckets: [15, 30, 45, 60, 90, 120]
      },
      {
        type: 'table',
        title: 'Top Errors',
        columns: ['errorType', 'count', 'affectedUsers'],
        sortBy: 'count',
        limit: 10
      }
    ]
  },

  // Real-Time
  realTime: {
    name: 'Real-Time Monitor',
    refresh: '10s',
    widgets: [
      {
        type: 'live_counter',
        metrics: ['currentOnline', 'matchesInProgress', 'queueSize']
      },
      {
        type: 'live_chart',
        title: 'Sessions per Minute',
        metric: 'sessionStarts',
        window: '60m'
      },
      {
        type: 'live_chart',
        title: 'Purchases per Minute',
        metric: 'purchases',
        window: '60m'
      },
      {
        type: 'geo_map',
        metric: 'activePlayers',
        dimension: 'region'
      }
    ]
  }
};
```

### Retention Analysis

```typescript
interface RetentionCohort {
  cohortDate: string;
  cohortSize: number;
  retention: {
    [day: number]: {
      count: number;
      percentage: number;
    };
  };
}

class RetentionAnalyzer {
  async calculateCohortRetention(
    startDate: string,
    endDate: string,
    retentionDays: number[] = [1, 3, 7, 14, 30]
  ): Promise<RetentionCohort[]> {
    const query = `
      WITH cohorts AS (
        SELECT
          userId,
          DATE(MIN(timestamp)) as cohort_date
        FROM session_events
        WHERE isFirstSession = true
          AND DATE(timestamp) BETWEEN '${startDate}' AND '${endDate}'
        GROUP BY userId
      ),
      activity AS (
        SELECT DISTINCT
          userId,
          DATE(timestamp) as activity_date
        FROM session_events
      )
      SELECT
        c.cohort_date,
        COUNT(DISTINCT c.userId) as cohort_size,
        ${retentionDays.map(d => `
          COUNT(DISTINCT CASE
            WHEN DATEDIFF(a.activity_date, c.cohort_date) = ${d}
            THEN a.userId
          END) as day_${d}_retained
        `).join(',')}
      FROM cohorts c
      LEFT JOIN activity a ON c.userId = a.userId
      GROUP BY c.cohort_date
      ORDER BY c.cohort_date
    `;

    const results = await Database.query(query);
    return this.formatCohortResults(results, retentionDays);
  }

  async calculateRollingRetention(
    days: number = 7
  ): Promise<RollingRetention> {
    const query = `
      WITH first_sessions AS (
        SELECT
          userId,
          DATE(MIN(timestamp)) as install_date
        FROM session_events
        WHERE isFirstSession = true
          AND DATE(timestamp) = CURRENT_DATE - ${days}
        GROUP BY userId
      ),
      returned AS (
        SELECT DISTINCT userId
        FROM session_events
        WHERE DATE(timestamp) = CURRENT_DATE
      )
      SELECT
        COUNT(DISTINCT f.userId) as cohort_size,
        COUNT(DISTINCT r.userId) as retained,
        COUNT(DISTINCT r.userId) * 100.0 / COUNT(DISTINCT f.userId) as retention_rate
      FROM first_sessions f
      LEFT JOIN returned r ON f.userId = r.userId
    `;

    return await Database.queryOne(query);
  }
}
```

### Funnel Analysis

```typescript
interface FunnelStep {
  name: string;
  eventName: string;
  filter?: Record<string, any>;
}

class FunnelAnalyzer {
  async analyzeFunnel(
    steps: FunnelStep[],
    timeRange: { start: string; end: string },
    options?: {
      conversionWindow?: number; // hours
      breakdown?: string;
    }
  ): Promise<FunnelResult> {
    const conversionWindow = options?.conversionWindow ?? 24;

    const stepQueries = steps.map((step, index) => {
      if (index === 0) {
        return `
          step_${index} AS (
            SELECT DISTINCT userId, MIN(timestamp) as step_time
            FROM ${step.eventName}
            WHERE timestamp BETWEEN '${timeRange.start}' AND '${timeRange.end}'
            ${this.buildFilter(step.filter)}
            GROUP BY userId
          )
        `;
      }

      return `
        step_${index} AS (
          SELECT DISTINCT s.userId, MIN(e.timestamp) as step_time
          FROM step_${index - 1} s
          JOIN ${step.eventName} e ON s.userId = e.userId
          WHERE e.timestamp > s.step_time
            AND e.timestamp < s.step_time + INTERVAL ${conversionWindow} HOUR
            ${this.buildFilter(step.filter)}
          GROUP BY s.userId
        )
      `;
    });

    const countQueries = steps.map((_, index) =>
      `(SELECT COUNT(*) FROM step_${index}) as step_${index}_count`
    );

    const query = `
      WITH ${stepQueries.join(', ')}
      SELECT ${countQueries.join(', ')}
    `;

    const result = await Database.queryOne(query);

    return {
      steps: steps.map((step, index) => ({
        name: step.name,
        count: result[`step_${index}_count`],
        conversionFromPrevious: index === 0
          ? 100
          : (result[`step_${index}_count`] / result[`step_${index - 1}_count`]) * 100,
        conversionFromFirst:
          (result[`step_${index}_count`] / result.step_0_count) * 100
      }))
    };
  }
}

// Example funnels
const STANDARD_FUNNELS = {
  newUserActivation: [
    { name: 'Install', eventName: 'session_start', filter: { isFirstSession: true } },
    { name: 'Tutorial Start', eventName: 'tutorial', filter: { action: 'start' } },
    { name: 'Tutorial Complete', eventName: 'tutorial', filter: { action: 'complete' } },
    { name: 'First Match', eventName: 'match_start' },
    { name: 'First Win', eventName: 'match_end', filter: { placement: 1 } }
  ],

  purchaseFunnel: [
    { name: 'Store View', eventName: 'feature_usage', filter: { featureName: 'store' } },
    { name: 'Product View', eventName: 'feature_usage', filter: { featureName: 'store_item' } },
    { name: 'Purchase Start', eventName: 'purchase', filter: { status: 'started' } },
    { name: 'Purchase Complete', eventName: 'purchase', filter: { status: 'completed' } }
  ],

  battlePassConversion: [
    { name: 'BP Screen View', eventName: 'feature_usage', filter: { featureName: 'battle_pass' } },
    { name: 'Tier Earned', eventName: 'battle_pass_progress' },
    { name: 'Premium Purchase', eventName: 'purchase', filter: { productType: 'battle_pass' } }
  ]
};
```

---

## Privacy and Compliance

### Consent Management

```typescript
interface AnalyticsConsent {
  userId: string;
  consents: {
    analytics: boolean;
    personalization: boolean;
    advertising: boolean;
  };
  timestamp: string;
  ipAddress?: string;
  version: string;
}

class ConsentManager {
  // Get required consents based on region
  getRequiredConsents(region: string): string[] {
    const gdprRegions = ['EU', 'EEA', 'UK'];
    const ccpaRegions = ['US-CA'];

    if (gdprRegions.some(r => region.startsWith(r))) {
      return ['analytics', 'personalization', 'advertising'];
    }

    if (ccpaRegions.includes(region)) {
      return ['advertising']; // CCPA focuses on sale of data
    }

    return []; // No explicit consent required
  }

  // Check if analytics can be collected
  async canCollectAnalytics(userId: string): Promise<boolean> {
    const consent = await this.getConsent(userId);
    const region = await GeoService.getUserRegion(userId);
    const required = this.getRequiredConsents(region);

    if (!required.includes('analytics')) {
      return true; // No consent required
    }

    return consent?.consents.analytics ?? false;
  }

  // Record consent
  async recordConsent(
    userId: string,
    consents: AnalyticsConsent['consents']
  ): Promise<void> {
    const consent: AnalyticsConsent = {
      userId,
      consents,
      timestamp: new Date().toISOString(),
      version: '1.0'
    };

    await this.storeConsent(consent);

    // Track consent for audit
    Analytics.track('consent_updated', {
      userId: this.hashUserId(userId),
      consents,
      timestamp: consent.timestamp
    });
  }
}
```

### Data Retention

```typescript
const DATA_RETENTION_POLICY = {
  // Raw events
  rawEvents: {
    retention: 90, // days
    action: 'delete'
  },

  // Aggregated data
  dailyAggregates: {
    retention: 365 * 2, // 2 years
    action: 'archive'
  },

  // User-level data
  userProfiles: {
    retention: 365 * 3, // 3 years from last activity
    action: 'anonymize'
  },

  // Financial data
  purchaseHistory: {
    retention: 365 * 7, // 7 years for compliance
    action: 'archive'
  }
};

class DataRetentionService {
  async enforceRetention(): Promise<RetentionReport> {
    const report: RetentionReport = {
      deletedRecords: 0,
      archivedRecords: 0,
      anonymizedRecords: 0
    };

    for (const [dataType, policy] of Object.entries(DATA_RETENTION_POLICY)) {
      const cutoffDate = new Date();
      cutoffDate.setDate(cutoffDate.getDate() - policy.retention);

      switch (policy.action) {
        case 'delete':
          report.deletedRecords += await this.deleteOldData(dataType, cutoffDate);
          break;
        case 'archive':
          report.archivedRecords += await this.archiveData(dataType, cutoffDate);
          break;
        case 'anonymize':
          report.anonymizedRecords += await this.anonymizeData(dataType, cutoffDate);
          break;
      }
    }

    return report;
  }

  private async anonymizeData(
    dataType: string,
    cutoffDate: Date
  ): Promise<number> {
    // Replace PII with hashed values
    const query = `
      UPDATE ${dataType}
      SET
        userId = SHA256(userId),
        deviceId = SHA256(deviceId),
        ipAddress = NULL,
        email = NULL
      WHERE lastActivityDate < '${cutoffDate.toISOString()}'
        AND isAnonymized = false
    `;

    const result = await Database.execute(query);
    return result.affectedRows;
  }
}
```

### Data Export (GDPR)

```typescript
class DataExportService {
  async exportUserData(userId: string): Promise<UserDataExport> {
    const [
      profile,
      sessions,
      matches,
      purchases,
      preferences
    ] = await Promise.all([
      this.getProfileData(userId),
      this.getSessionData(userId),
      this.getMatchData(userId),
      this.getPurchaseData(userId),
      this.getPreferenceData(userId)
    ]);

    return {
      exportDate: new Date().toISOString(),
      userId,
      data: {
        profile,
        activity: {
          sessions: sessions.slice(0, 1000), // Last 1000 sessions
          matches: matches.slice(0, 500) // Last 500 matches
        },
        financial: {
          purchases: purchases // All purchases
        },
        preferences
      }
    };
  }

  async deleteUserData(userId: string): Promise<DeletionReport> {
    // GDPR "Right to be forgotten"
    const tables = [
      'session_events',
      'match_events',
      'kill_events',
      'death_events',
      'purchase_events',
      'quest_events',
      'user_profiles'
    ];

    const report: DeletionReport = {
      userId,
      deletedAt: new Date().toISOString(),
      tables: {}
    };

    for (const table of tables) {
      const result = await Database.execute(`
        DELETE FROM ${table} WHERE userId = '${userId}'
      `);
      report.tables[table] = result.affectedRows;
    }

    return report;
  }
}
```

---

## Integration Points

### Analytics Services

```typescript
const ANALYTICS_INTEGRATIONS = {
  // Primary analytics
  amplitude: {
    apiKey: process.env.AMPLITUDE_API_KEY,
    events: 'all'
  },

  // Firebase for mobile
  firebase: {
    projectId: process.env.FIREBASE_PROJECT,
    events: ['session', 'match', 'purchase']
  },

  // Attribution
  adjust: {
    appToken: process.env.ADJUST_TOKEN,
    events: ['install', 'purchase', 'level_up']
  },

  // Revenue tracking
  appsflyer: {
    devKey: process.env.APPSFLYER_KEY,
    events: ['purchase', 'subscription']
  }
};

class AnalyticsRouter {
  async routeEvent(event: AnalyticsEvent): Promise<void> {
    const destinations = this.getDestinations(event.eventName);

    await Promise.all(
      destinations.map(dest => this.sendToDestination(dest, event))
    );
  }
}
```

### Internal Dependencies

- BACK-011: Player Profile Service (user data)
- BACK-020: Server Monitoring (technical metrics)
- BACK-021: Crash Reporting (error tracking)
- BACK-016: Anti-Cheat (behavioral data validation)
