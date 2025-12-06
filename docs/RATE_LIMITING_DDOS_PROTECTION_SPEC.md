# Rate Limiting and DDoS Protection Specification

## Overview

This specification covers the protection of game services from abuse, denial-of-service attacks, and traffic spikes. The system implements multi-layer protection including API rate limiting, game server DDoS mitigation, abuse detection, and graceful degradation strategies.

## Goals

1. **Service Availability**: Maintain 99.9% uptime even under attack
2. **Fair Access**: Prevent single users from consuming excessive resources
3. **Attack Mitigation**: Automatically detect and block malicious traffic
4. **Graceful Degradation**: Continue serving legitimate users during stress
5. **Cost Protection**: Prevent unexpected infrastructure costs from attacks

---

## Rate Limiting Architecture

### Rate Limit Configuration

```typescript
interface RateLimitConfig {
  // Identifier for rate limit rule
  id: string;
  name: string;

  // What to rate limit by
  keyType: 'ip' | 'user_id' | 'device_id' | 'api_key' | 'composite';

  // Limits
  requests: number;
  windowSeconds: number;

  // Burst handling
  burstAllowance?: number;

  // Actions when exceeded
  action: 'reject' | 'throttle' | 'queue' | 'degraded';

  // Response when rate limited
  responseCode: number;
  responseMessage: string;

  // Retry-After header value
  retryAfterSeconds?: number;
}
```

### API Rate Limits

```typescript
const API_RATE_LIMITS: Record<string, RateLimitConfig> = {
  // Global per-user limit
  global_user: {
    id: 'global_user',
    name: 'Global User Rate Limit',
    keyType: 'user_id',
    requests: 600,
    windowSeconds: 60, // 10 requests/second average
    burstAllowance: 50,
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Too many requests. Please slow down.',
    retryAfterSeconds: 60
  },

  // Global per-IP limit (for unauthenticated)
  global_ip: {
    id: 'global_ip',
    name: 'Global IP Rate Limit',
    keyType: 'ip',
    requests: 300,
    windowSeconds: 60,
    burstAllowance: 30,
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Too many requests from this IP.',
    retryAfterSeconds: 60
  },

  // Authentication endpoints (stricter)
  auth_login: {
    id: 'auth_login',
    name: 'Login Rate Limit',
    keyType: 'ip',
    requests: 10,
    windowSeconds: 300, // 10 attempts per 5 minutes
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Too many login attempts. Please wait.',
    retryAfterSeconds: 300
  },

  // Registration
  auth_register: {
    id: 'auth_register',
    name: 'Registration Rate Limit',
    keyType: 'ip',
    requests: 5,
    windowSeconds: 3600, // 5 per hour
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Registration limit reached.',
    retryAfterSeconds: 3600
  },

  // Matchmaking
  matchmaking_queue: {
    id: 'matchmaking_queue',
    name: 'Queue Rate Limit',
    keyType: 'user_id',
    requests: 30,
    windowSeconds: 60, // Can queue/cancel frequently
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Queue rate limit reached.',
    retryAfterSeconds: 10
  },

  // Social/Friends
  friend_requests: {
    id: 'friend_requests',
    name: 'Friend Request Rate Limit',
    keyType: 'user_id',
    requests: 20,
    windowSeconds: 3600, // 20 per hour
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Friend request limit reached.',
    retryAfterSeconds: 60
  },

  // Purchases
  purchase_initiate: {
    id: 'purchase_initiate',
    name: 'Purchase Rate Limit',
    keyType: 'user_id',
    requests: 10,
    windowSeconds: 60,
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Too many purchase attempts.',
    retryAfterSeconds: 60
  },

  // Reports
  player_reports: {
    id: 'player_reports',
    name: 'Player Report Rate Limit',
    keyType: 'user_id',
    requests: 10,
    windowSeconds: 86400, // 10 per day
    action: 'reject',
    responseCode: 429,
    responseMessage: 'Daily report limit reached.',
    retryAfterSeconds: 3600
  },

  // Chat messages
  chat_messages: {
    id: 'chat_messages',
    name: 'Chat Rate Limit',
    keyType: 'user_id',
    requests: 30,
    windowSeconds: 60, // 30 messages per minute
    action: 'throttle',
    responseCode: 429,
    responseMessage: 'Sending messages too fast.',
    retryAfterSeconds: 5
  }
};
```

### Rate Limiter Implementation

```typescript
class RateLimiter {
  private readonly redis: RedisClient;

  async checkRateLimit(
    config: RateLimitConfig,
    key: string
  ): Promise<RateLimitResult> {
    const redisKey = `ratelimit:${config.id}:${key}`;
    const now = Date.now();
    const windowStart = now - config.windowSeconds * 1000;

    // Use sliding window algorithm
    const pipeline = this.redis.pipeline();

    // Remove old entries
    pipeline.zremrangebyscore(redisKey, 0, windowStart);

    // Count current entries
    pipeline.zcard(redisKey);

    // Add current request
    pipeline.zadd(redisKey, now, `${now}:${Math.random()}`);

    // Set expiration
    pipeline.expire(redisKey, config.windowSeconds + 60);

    const results = await pipeline.exec();
    const currentCount = results[1][1] as number;

    // Check if exceeded
    const effectiveLimit = config.requests + (config.burstAllowance ?? 0);

    if (currentCount >= effectiveLimit) {
      return {
        allowed: false,
        limit: config.requests,
        remaining: 0,
        resetAt: now + config.windowSeconds * 1000,
        retryAfter: config.retryAfterSeconds ?? config.windowSeconds
      };
    }

    return {
      allowed: true,
      limit: config.requests,
      remaining: Math.max(0, config.requests - currentCount - 1),
      resetAt: now + config.windowSeconds * 1000
    };
  }

  // Check multiple rate limits for a request
  async checkMultipleLimits(
    request: APIRequest
  ): Promise<RateLimitResult> {
    const applicableLimits = this.getApplicableLimits(request);

    for (const limit of applicableLimits) {
      const key = this.extractKey(request, limit.keyType);
      const result = await this.checkRateLimit(limit, key);

      if (!result.allowed) {
        return result;
      }
    }

    return { allowed: true };
  }
}
```

### Token Bucket for Burst Handling

```typescript
class TokenBucketLimiter {
  private readonly buckets: Map<string, TokenBucket> = new Map();

  async consume(
    key: string,
    config: BucketConfig,
    tokens: number = 1
  ): Promise<BucketResult> {
    let bucket = this.buckets.get(key);

    if (!bucket) {
      bucket = {
        tokens: config.maxTokens,
        lastRefill: Date.now()
      };
    }

    // Refill tokens based on time passed
    const now = Date.now();
    const elapsed = now - bucket.lastRefill;
    const refillAmount = (elapsed / 1000) * config.refillRate;
    bucket.tokens = Math.min(config.maxTokens, bucket.tokens + refillAmount);
    bucket.lastRefill = now;

    // Check if enough tokens
    if (bucket.tokens < tokens) {
      const waitTime = (tokens - bucket.tokens) / config.refillRate * 1000;
      return {
        allowed: false,
        waitTimeMs: Math.ceil(waitTime),
        tokensRemaining: bucket.tokens
      };
    }

    // Consume tokens
    bucket.tokens -= tokens;
    this.buckets.set(key, bucket);

    return {
      allowed: true,
      tokensRemaining: bucket.tokens
    };
  }
}

// Bucket configurations per endpoint type
const TOKEN_BUCKET_CONFIGS: Record<string, BucketConfig> = {
  api_default: {
    maxTokens: 100,
    refillRate: 10 // 10 tokens per second
  },

  matchmaking: {
    maxTokens: 20,
    refillRate: 2
  },

  store_purchase: {
    maxTokens: 10,
    refillRate: 0.5 // 1 token per 2 seconds
  },

  game_server: {
    maxTokens: 1000,
    refillRate: 100 // High capacity for game traffic
  }
};
```

---

## DDoS Protection

### Multi-Layer Defense

```typescript
const DDOS_DEFENSE_LAYERS = {
  // Layer 1: CDN/Edge (Cloudflare, AWS CloudFront)
  edge: {
    provider: 'cloudflare',
    features: [
      'ddos_mitigation',
      'waf_rules',
      'bot_management',
      'rate_limiting',
      'geo_blocking'
    ],
    config: {
      securityLevel: 'high',
      challengePassage: 1800, // 30 minutes
      browserIntegrityCheck: true
    }
  },

  // Layer 2: Load Balancer
  loadBalancer: {
    provider: 'aws_alb',
    features: [
      'connection_draining',
      'health_checks',
      'sticky_sessions',
      'request_timeout'
    ],
    config: {
      idleTimeout: 60,
      connectionLimit: 10000,
      requestTimeout: 30000
    }
  },

  // Layer 3: Application Level
  application: {
    features: [
      'rate_limiting',
      'request_validation',
      'payload_inspection',
      'behavior_analysis'
    ]
  },

  // Layer 4: Game Server (UDP)
  gameServer: {
    protection: 'aws_shield_advanced',
    features: [
      'udp_reflection_mitigation',
      'amplification_protection',
      'custom_mitigation_rules'
    ]
  }
};
```

### AWS Shield Configuration

```typescript
const AWS_SHIELD_CONFIG = {
  // Shield Advanced for game servers
  advanced: {
    enabled: true,
    resources: [
      'game_server_elb',
      'api_cloudfront',
      'matchmaking_service'
    ],
    responseTeamAccess: true,
    healthBasedDetection: true
  },

  // Custom mitigation rules
  mitigationRules: [
    {
      name: 'block_large_udp',
      condition: 'UDP packet size > 1500 bytes',
      action: 'block'
    },
    {
      name: 'rate_limit_syn',
      condition: 'SYN packets from single IP > 100/s',
      action: 'rate_limit'
    },
    {
      name: 'block_reflection',
      condition: 'Known reflection amplification patterns',
      action: 'block'
    }
  ],

  // Alerting
  alerting: {
    snsTopicArn: 'arn:aws:sns:us-east-1:xxx:ddos-alerts',
    thresholds: {
      volumeBasedTrigger: '1 Gbps',
      packetBasedTrigger: '100k pps',
      requestBasedTrigger: '10k rps'
    }
  }
};
```

### Cloudflare WAF Rules

```typescript
const CLOUDFLARE_WAF_RULES = {
  // Block known attack patterns
  blockPatterns: [
    {
      id: 'block_sql_injection',
      expression: '(http.request.uri.query contains "UNION SELECT")',
      action: 'block'
    },
    {
      id: 'block_suspicious_ua',
      expression: '(http.user_agent contains "sqlmap") or (http.user_agent contains "nikto")',
      action: 'block'
    }
  ],

  // Rate limiting rules
  rateLimiting: [
    {
      id: 'api_rate_limit',
      expression: '(http.request.uri.path matches "^/api/")',
      characteristics: ['ip.src'],
      period: 60,
      requestsPerPeriod: 300,
      action: 'challenge'
    },
    {
      id: 'auth_rate_limit',
      expression: '(http.request.uri.path matches "^/api/auth/")',
      characteristics: ['ip.src'],
      period: 300,
      requestsPerPeriod: 20,
      action: 'block'
    }
  ],

  // Bot management
  botManagement: {
    enabled: true,
    mode: 'challenge', // challenge | managed_challenge | block
    exceptions: [
      'known_good_bots',
      'monitoring_services'
    ]
  },

  // Geo blocking
  geoBlocking: {
    enabled: true,
    allowedCountries: ['US', 'CA', 'GB', 'DE', 'FR', 'JP', 'KR', 'AU', 'BR'],
    action: 'challenge' // For countries not in list
  }
};
```

### Game Server DDoS Protection

```typescript
class GameServerProtection {
  // UDP flood protection
  private readonly udpRateLimiter = new Map<string, UDPRateLimit>();

  async handleIncomingPacket(
    sourceIP: string,
    packet: Buffer
  ): Promise<PacketResult> {
    // 1. Check IP reputation
    const reputation = await this.checkIPReputation(sourceIP);
    if (reputation.blocked) {
      return { action: 'drop', reason: 'blocked_ip' };
    }

    // 2. Check packet rate
    const rateResult = await this.checkPacketRate(sourceIP);
    if (!rateResult.allowed) {
      // Add to temporary block list
      await this.addToBlockList(sourceIP, 60); // 60 second block
      return { action: 'drop', reason: 'rate_exceeded' };
    }

    // 3. Validate packet structure
    const validPacket = this.validatePacketStructure(packet);
    if (!validPacket) {
      return { action: 'drop', reason: 'invalid_packet' };
    }

    // 4. Check for amplification attack patterns
    if (this.isAmplificationAttempt(packet)) {
      await this.addToBlockList(sourceIP, 3600); // 1 hour block
      return { action: 'drop', reason: 'amplification_attempt' };
    }

    return { action: 'allow' };
  }

  private async checkPacketRate(ip: string): Promise<RateLimitResult> {
    const now = Date.now();
    let limiter = this.udpRateLimiter.get(ip);

    if (!limiter) {
      limiter = {
        count: 0,
        windowStart: now,
        blocked: false
      };
    }

    // Reset window if expired
    if (now - limiter.windowStart > 1000) {
      limiter.count = 0;
      limiter.windowStart = now;
    }

    limiter.count++;

    // Max 1000 packets per second per IP
    if (limiter.count > 1000) {
      return { allowed: false };
    }

    this.udpRateLimiter.set(ip, limiter);
    return { allowed: true };
  }

  private isAmplificationAttempt(packet: Buffer): boolean {
    // Check for small request that expects large response
    // These are common in amplification attacks
    const requestSize = packet.length;
    const expectedResponseSize = this.estimateResponseSize(packet);

    // Amplification factor > 10 is suspicious
    return expectedResponseSize / requestSize > 10;
  }
}
```

---

## Abuse Detection

### Behavior Analysis

```typescript
interface AbuseIndicator {
  type: string;
  severity: 'low' | 'medium' | 'high' | 'critical';
  score: number;
  details: string;
}

class AbuseDetector {
  async analyzeUserBehavior(
    userId: string,
    recentActivity: UserActivity[]
  ): Promise<AbuseAnalysis> {
    const indicators: AbuseIndicator[] = [];

    // 1. Request pattern analysis
    const patternIndicator = this.analyzeRequestPatterns(recentActivity);
    if (patternIndicator) indicators.push(patternIndicator);

    // 2. Timing analysis (bot detection)
    const timingIndicator = this.analyzeRequestTiming(recentActivity);
    if (timingIndicator) indicators.push(timingIndicator);

    // 3. Geographic anomaly
    const geoIndicator = await this.analyzeGeographicPattern(userId, recentActivity);
    if (geoIndicator) indicators.push(geoIndicator);

    // 4. Credential stuffing detection
    const credentialIndicator = this.detectCredentialStuffing(recentActivity);
    if (credentialIndicator) indicators.push(credentialIndicator);

    // 5. Resource exhaustion attempts
    const resourceIndicator = this.detectResourceExhaustion(recentActivity);
    if (resourceIndicator) indicators.push(resourceIndicator);

    // Calculate overall risk score
    const riskScore = this.calculateRiskScore(indicators);

    return {
      userId,
      indicators,
      riskScore,
      recommendation: this.getRecommendation(riskScore)
    };
  }

  private analyzeRequestTiming(activity: UserActivity[]): AbuseIndicator | null {
    // Bots often have very consistent request timing
    const intervals = [];
    for (let i = 1; i < activity.length; i++) {
      intervals.push(activity[i].timestamp - activity[i-1].timestamp);
    }

    if (intervals.length < 10) return null;

    // Calculate standard deviation
    const avg = intervals.reduce((a, b) => a + b, 0) / intervals.length;
    const variance = intervals.reduce((sum, val) => sum + Math.pow(val - avg, 2), 0) / intervals.length;
    const stdDev = Math.sqrt(variance);

    // Very low variance suggests automated behavior
    if (stdDev < 10 && avg < 100) {
      return {
        type: 'bot_timing',
        severity: 'high',
        score: 30,
        details: `Suspicious request timing: avg=${avg.toFixed(0)}ms, stdDev=${stdDev.toFixed(0)}ms`
      };
    }

    return null;
  }

  private detectCredentialStuffing(activity: UserActivity[]): AbuseIndicator | null {
    // Look for many failed login attempts with different credentials
    const loginAttempts = activity.filter(a => a.type === 'login_attempt');
    const failedAttempts = loginAttempts.filter(a => !a.success);
    const uniqueUsernames = new Set(failedAttempts.map(a => a.metadata?.username));

    if (failedAttempts.length > 10 && uniqueUsernames.size > 5) {
      return {
        type: 'credential_stuffing',
        severity: 'critical',
        score: 50,
        details: `${failedAttempts.length} failed logins with ${uniqueUsernames.size} different usernames`
      };
    }

    return null;
  }

  private getRecommendation(score: number): AbuseRecommendation {
    if (score >= 80) {
      return {
        action: 'block',
        duration: 86400, // 24 hours
        requireCaptcha: true
      };
    } else if (score >= 60) {
      return {
        action: 'challenge',
        duration: 3600, // 1 hour
        requireCaptcha: true
      };
    } else if (score >= 40) {
      return {
        action: 'rate_limit',
        factor: 0.5, // Reduce rate limits by 50%
        requireCaptcha: false
      };
    } else if (score >= 20) {
      return {
        action: 'monitor',
        escalateAfter: 3600
      };
    }

    return { action: 'none' };
  }
}
```

### IP Reputation System

```typescript
interface IPReputation {
  ip: string;
  score: number; // 0-100, higher is more trustworthy
  lastUpdated: string;
  factors: {
    knownProxy: boolean;
    knownVPN: boolean;
    knownTor: boolean;
    datacenterIP: boolean;
    previousAbuse: boolean;
    countryRisk: number;
    asnReputation: number;
  };
  blocked: boolean;
  blockReason?: string;
  blockExpiry?: string;
}

class IPReputationService {
  private readonly cache = new Map<string, IPReputation>();

  async getReputation(ip: string): Promise<IPReputation> {
    // Check cache
    const cached = this.cache.get(ip);
    if (cached && this.isFresh(cached)) {
      return cached;
    }

    // Build reputation
    const reputation = await this.buildReputation(ip);
    this.cache.set(ip, reputation);

    return reputation;
  }

  private async buildReputation(ip: string): Promise<IPReputation> {
    // Parallel queries
    const [
      proxyCheck,
      geoData,
      abuseHistory,
      asnData
    ] = await Promise.all([
      this.checkProxy(ip),
      this.getGeoData(ip),
      this.getAbuseHistory(ip),
      this.getASNData(ip)
    ]);

    const factors = {
      knownProxy: proxyCheck.isProxy,
      knownVPN: proxyCheck.isVPN,
      knownTor: proxyCheck.isTor,
      datacenterIP: asnData.isDatacenter,
      previousAbuse: abuseHistory.hasAbuse,
      countryRisk: this.getCountryRisk(geoData.country),
      asnReputation: asnData.reputation
    };

    // Calculate score
    let score = 100;

    if (factors.knownProxy) score -= 20;
    if (factors.knownVPN) score -= 15;
    if (factors.knownTor) score -= 30;
    if (factors.datacenterIP) score -= 25;
    if (factors.previousAbuse) score -= 40;
    score -= factors.countryRisk;
    score -= (100 - factors.asnReputation) * 0.2;

    score = Math.max(0, Math.min(100, score));

    return {
      ip,
      score,
      lastUpdated: new Date().toISOString(),
      factors,
      blocked: score < 20 || factors.previousAbuse
    };
  }

  // Check external IP reputation services
  private async checkProxy(ip: string): Promise<ProxyCheckResult> {
    // Integration with services like ip-api, ipqualityscore, etc.
    try {
      const response = await fetch(
        `https://ipqualityscore.com/api/json/ip/${this.apiKey}/${ip}`
      );
      const data = await response.json();

      return {
        isProxy: data.proxy,
        isVPN: data.vpn,
        isTor: data.tor
      };
    } catch {
      return { isProxy: false, isVPN: false, isTor: false };
    }
  }
}
```

---

## Graceful Degradation

### Load Shedding

```typescript
interface LoadLevel {
  level: 'normal' | 'elevated' | 'high' | 'critical';
  threshold: number;
  actions: DegradationAction[];
}

const LOAD_LEVELS: LoadLevel[] = [
  {
    level: 'normal',
    threshold: 0.7,
    actions: []
  },
  {
    level: 'elevated',
    threshold: 0.85,
    actions: [
      { type: 'reduce_rate_limits', factor: 0.8 },
      { type: 'disable_feature', feature: 'analytics_tracking' }
    ]
  },
  {
    level: 'high',
    threshold: 0.95,
    actions: [
      { type: 'reduce_rate_limits', factor: 0.5 },
      { type: 'disable_feature', feature: 'leaderboards' },
      { type: 'disable_feature', feature: 'friend_status' },
      { type: 'enable_cache_only', services: ['player_profiles'] }
    ]
  },
  {
    level: 'critical',
    threshold: 1.0,
    actions: [
      { type: 'reduce_rate_limits', factor: 0.2 },
      { type: 'queue_new_requests' },
      { type: 'shed_non_essential', priority_threshold: 'high' },
      { type: 'enable_maintenance_mode' }
    ]
  }
];

class LoadShedder {
  private currentLevel: LoadLevel = LOAD_LEVELS[0];

  async assessLoad(): Promise<LoadAssessment> {
    const metrics = await this.collectMetrics();

    const loadScore = this.calculateLoadScore(metrics);
    const newLevel = this.determineLevel(loadScore);

    if (newLevel.level !== this.currentLevel.level) {
      await this.transitionToLevel(newLevel);
    }

    return {
      loadScore,
      level: this.currentLevel.level,
      activeActions: this.currentLevel.actions
    };
  }

  private calculateLoadScore(metrics: SystemMetrics): number {
    // Weighted average of various metrics
    const weights = {
      cpu: 0.25,
      memory: 0.2,
      connections: 0.2,
      requestRate: 0.2,
      responseTime: 0.15
    };

    return (
      metrics.cpuUsage * weights.cpu +
      metrics.memoryUsage * weights.memory +
      (metrics.connections / metrics.maxConnections) * weights.connections +
      (metrics.requestRate / metrics.maxRequestRate) * weights.requestRate +
      (metrics.avgResponseTime / metrics.targetResponseTime) * weights.responseTime
    );
  }

  private async transitionToLevel(newLevel: LoadLevel): Promise<void> {
    console.log(`Transitioning from ${this.currentLevel.level} to ${newLevel.level}`);

    // Apply new actions
    for (const action of newLevel.actions) {
      await this.applyAction(action);
    }

    // Remove actions no longer needed
    const removedActions = this.currentLevel.actions.filter(
      a => !newLevel.actions.some(na => na.type === a.type)
    );
    for (const action of removedActions) {
      await this.revertAction(action);
    }

    this.currentLevel = newLevel;

    // Alert operations team
    await this.sendAlert({
      type: 'load_level_change',
      from: this.currentLevel.level,
      to: newLevel.level,
      timestamp: new Date().toISOString()
    });
  }
}
```

### Request Prioritization

```typescript
enum RequestPriority {
  CRITICAL = 0,  // Authentication, active game sessions
  HIGH = 1,      // Matchmaking, purchases
  NORMAL = 2,    // Profile updates, friend requests
  LOW = 3,       // Analytics, leaderboards
  BACKGROUND = 4 // Batch jobs, optional features
}

const ENDPOINT_PRIORITIES: Record<string, RequestPriority> = {
  '/api/auth/login': RequestPriority.CRITICAL,
  '/api/auth/refresh': RequestPriority.CRITICAL,
  '/api/game/heartbeat': RequestPriority.CRITICAL,
  '/api/game/action': RequestPriority.CRITICAL,

  '/api/matchmaking/queue': RequestPriority.HIGH,
  '/api/matchmaking/status': RequestPriority.HIGH,
  '/api/store/purchase': RequestPriority.HIGH,

  '/api/profile/update': RequestPriority.NORMAL,
  '/api/friends/request': RequestPriority.NORMAL,
  '/api/party/invite': RequestPriority.NORMAL,

  '/api/leaderboard': RequestPriority.LOW,
  '/api/stats/detailed': RequestPriority.LOW,
  '/api/news': RequestPriority.LOW,

  '/api/analytics': RequestPriority.BACKGROUND,
  '/api/telemetry': RequestPriority.BACKGROUND
};

class RequestPrioritizer {
  async handleRequest(
    request: APIRequest,
    currentLoad: LoadLevel
  ): Promise<PrioritizationResult> {
    const priority = this.getPriority(request.path);

    // Under high load, shed low priority requests
    if (currentLoad.level === 'high' && priority >= RequestPriority.LOW) {
      return {
        action: 'reject',
        code: 503,
        message: 'Service temporarily unavailable. Please try again.',
        retryAfter: 30
      };
    }

    // Under critical load, only allow critical requests
    if (currentLoad.level === 'critical' && priority > RequestPriority.CRITICAL) {
      return {
        action: 'reject',
        code: 503,
        message: 'System under maintenance. Critical functions only.',
        retryAfter: 60
      };
    }

    return { action: 'allow', priority };
  }
}
```

---

## Monitoring and Alerting

### Metrics Collection

```typescript
const PROTECTION_METRICS = {
  // Rate limiting
  rateLimitHits: 'protection_rate_limit_hits_total',
  rateLimitByEndpoint: 'protection_rate_limit_by_endpoint',
  rateLimitByUser: 'protection_rate_limit_by_user',

  // DDoS
  droppedPackets: 'protection_dropped_packets_total',
  blockedIPs: 'protection_blocked_ips_total',
  attacksDetected: 'protection_attacks_detected_total',

  // Abuse
  abuseFlagged: 'protection_abuse_flagged_total',
  abuseBlocked: 'protection_abuse_blocked_total',

  // Load shedding
  currentLoadLevel: 'protection_load_level',
  requestsShed: 'protection_requests_shed_total'
};

class ProtectionMonitor {
  async collectMetrics(): Promise<ProtectionMetrics> {
    return {
      rateLimiting: {
        hitsLastHour: await this.getMetric('rate_limit_hits_1h'),
        topLimitedEndpoints: await this.getTopLimitedEndpoints(),
        topLimitedUsers: await this.getTopLimitedUsers()
      },

      ddos: {
        currentAttackStatus: await this.getCurrentAttackStatus(),
        packetsDroppedLastHour: await this.getMetric('packets_dropped_1h'),
        activeIPBlocks: await this.getActiveIPBlockCount()
      },

      abuse: {
        usersBlocked: await this.getBlockedUserCount(),
        suspiciousActivityCount: await this.getSuspiciousActivityCount()
      },

      loadShedding: {
        currentLevel: await this.getCurrentLoadLevel(),
        requestsShedLastHour: await this.getMetric('requests_shed_1h')
      }
    };
  }
}
```

### Alert Configuration

```typescript
const PROTECTION_ALERTS = {
  ddos_attack_detected: {
    condition: 'attack_score > 80',
    severity: 'critical',
    channels: ['pagerduty', 'slack'],
    message: 'DDoS attack detected: ${attack_type} from ${source_count} sources'
  },

  rate_limit_spike: {
    condition: 'rate_limit_hits_5m > 1000',
    severity: 'warning',
    channels: ['slack'],
    message: 'High rate limit hits: ${count} in last 5 minutes'
  },

  load_level_critical: {
    condition: 'load_level == critical',
    severity: 'critical',
    channels: ['pagerduty', 'slack'],
    message: 'System in CRITICAL load level, shedding requests'
  },

  suspicious_ip_activity: {
    condition: 'blocked_ips_1h > 100',
    severity: 'warning',
    channels: ['slack'],
    message: 'Unusual IP blocking activity: ${count} IPs blocked in 1 hour'
  },

  credential_stuffing: {
    condition: 'failed_logins_5m > 500 AND unique_usernames > 100',
    severity: 'critical',
    channels: ['pagerduty', 'slack'],
    message: 'Credential stuffing attack in progress'
  }
};
```

---

## Response Procedures

### Incident Response

```typescript
const INCIDENT_PLAYBOOKS = {
  ddos_attack: {
    severity: 'critical',
    steps: [
      'Verify attack via CloudFlare/Shield dashboards',
      'Enable I\'m Under Attack mode in CloudFlare',
      'Activate additional Shield protections',
      'Monitor traffic patterns',
      'Engage ISP/upstream provider if needed',
      'Document attack vectors for post-mortem'
    ],
    escalation: {
      '5min': 'On-call engineer',
      '15min': 'Security team lead',
      '30min': 'VP Engineering'
    }
  },

  rate_limit_abuse: {
    severity: 'medium',
    steps: [
      'Identify source (user ID, IP)',
      'Check if legitimate use case',
      'If abuse: add to block list',
      'If legitimate: adjust rate limits',
      'Monitor for continued activity'
    ]
  },

  load_spike: {
    severity: 'high',
    steps: [
      'Assess source of load',
      'If attack: follow ddos_attack playbook',
      'If organic: scale up resources',
      'Enable load shedding if needed',
      'Monitor resource utilization'
    ]
  }
};
```

---

## Testing Strategy

### Load Testing

```typescript
const LOAD_TEST_SCENARIOS = {
  baseline: {
    duration: '10m',
    users: 1000,
    rampUp: '2m',
    requests: 'normal_distribution'
  },

  stress: {
    duration: '15m',
    users: 10000,
    rampUp: '5m',
    requests: 'normal_distribution'
  },

  spike: {
    duration: '5m',
    users: 50000,
    rampUp: '30s', // Sudden spike
    requests: 'burst'
  },

  ddos_simulation: {
    duration: '5m',
    sources: 1000, // IPs
    requestRate: 100000, // per second
    patterns: ['syn_flood', 'udp_flood', 'http_flood']
  }
};
```

### Rate Limit Testing

```typescript
describe('Rate Limiting', () => {
  it('should enforce global rate limit', async () => {
    // Send 600 requests in 60 seconds (at limit)
    for (let i = 0; i < 600; i++) {
      const response = await makeRequest('/api/test');
      expect(response.status).toBe(200);
    }

    // Next request should be rate limited
    const limitedResponse = await makeRequest('/api/test');
    expect(limitedResponse.status).toBe(429);
  });

  it('should respect burst allowance', async () => {
    // Send 50 requests immediately (burst)
    const responses = await Promise.all(
      Array(50).fill(null).map(() => makeRequest('/api/test'))
    );

    // All should succeed due to burst allowance
    expect(responses.every(r => r.status === 200)).toBe(true);
  });
});
```

---

## Integration Points

### Dependencies

- CloudFlare or AWS CloudFront (CDN/WAF)
- AWS Shield Advanced (DDoS protection)
- Redis (rate limiting storage)
- Monitoring stack (Prometheus/Grafana)

### Configuration Interface

```typescript
// Rate limits can be adjusted via admin API
POST /admin/rate-limits/:id
{
  "requests": 1000,
  "windowSeconds": 60
}

// IP blocking
POST /admin/block-ip
{
  "ip": "1.2.3.4",
  "reason": "DDoS source",
  "duration": 86400
}

// Load level override (emergency)
POST /admin/load-level
{
  "level": "critical",
  "reason": "Manual override due to attack"
}
```
