# Anti-Cheat System Specification

## Overview

The Anti-Cheat System provides comprehensive protection against cheating beyond server-authoritative validation. This includes client-side integrity checks, behavioral analysis, player reporting, and a fair but firm ban system. The system is designed for mobile gaming where traditional anti-cheat solutions have limitations.

## Goals

1. **Prevent Cheating**: Stop common cheats before they impact gameplay
2. **Detect Anomalies**: Identify suspicious behavior through statistics
3. **Empower Players**: Give players tools to report cheaters
4. **Fair Enforcement**: Graduated penalties with appeal process
5. **Low False Positives**: Target < 0.1% false positive rate

---

## Defense Layers

### Layer 1: Server Authority (Foundation)

The server-authoritative validation (BACK-015) provides the foundation:
- All critical calculations server-side
- Movement speed validation
- Damage calculation
- Hit registration with position validation
- Cooldown enforcement

### Layer 2: Client Integrity

```typescript
interface IntegrityCheckResult {
  passed: boolean;
  checks: {
    name: string;
    status: 'pass' | 'fail' | 'warning';
    details?: string;
  }[];
  timestamp: number;
  deviceFingerprint: string;
}

class ClientIntegrityChecker {
  private readonly CHECK_INTERVAL = 60000; // 1 minute

  async performIntegrityCheck(): Promise<IntegrityCheckResult> {
    const checks: IntegrityCheckResult['checks'] = [];

    // 1. Code signature verification
    checks.push(await this.verifyCodeSignature());

    // 2. Memory integrity
    checks.push(await this.checkMemoryIntegrity());

    // 3. Debug detection
    checks.push(await this.detectDebugger());

    // 4. Root/Jailbreak detection
    checks.push(await this.detectRootJailbreak());

    // 5. Emulator detection
    checks.push(await this.detectEmulator());

    // 6. Time manipulation detection
    checks.push(await this.detectTimeManipulation());

    // 7. Hook detection
    checks.push(await this.detectHooks());

    return {
      passed: checks.every(c => c.status !== 'fail'),
      checks,
      timestamp: Date.now(),
      deviceFingerprint: await this.getDeviceFingerprint()
    };
  }
}
```

### Layer 3: Behavioral Analysis

```typescript
interface BehavioralMetrics {
  playerId: string;
  sessionId: string;
  metrics: {
    // Accuracy metrics
    shotsHit: number;
    shotsFired: number;
    headshots: number;
    headshotRate: number;

    // Movement metrics
    averageSpeed: number;
    speedViolations: number;
    teleportations: number;

    // Combat metrics
    averageTimeToKill: number;
    killsAtMaxRange: number;
    throughWallKills: number;

    // Reaction metrics
    averageReactionTime: number;
    impossibleReactions: number;

    // Session metrics
    matchCount: number;
    winRate: number;
    averageKills: number;
    averageSurvivalTime: number;
  };
}
```

### Layer 4: Community Reports

Player-driven reporting with review process.

---

## Client Integrity Checks

### Code Signature Verification

```typescript
class CodeSignatureVerifier {
  private readonly EXPECTED_HASH: string; // Per version

  async verifyCodeSignature(): Promise<CheckResult> {
    try {
      // Get current executable hash
      const currentHash = await this.calculateExecutableHash();

      // Compare with expected
      if (currentHash !== this.EXPECTED_HASH) {
        return {
          name: 'code_signature',
          status: 'fail',
          details: 'Executable modified'
        };
      }

      // Verify critical game files
      const fileIntegrity = await this.verifyGameFiles();
      if (!fileIntegrity.valid) {
        return {
          name: 'code_signature',
          status: 'fail',
          details: `Modified files: ${fileIntegrity.modified.join(', ')}`
        };
      }

      return { name: 'code_signature', status: 'pass' };
    } catch (error) {
      return {
        name: 'code_signature',
        status: 'warning',
        details: 'Unable to verify'
      };
    }
  }

  private async verifyGameFiles(): Promise<FileIntegrityResult> {
    const criticalFiles = [
      'game_logic.bundle',
      'network.bundle',
      'rendering.bundle'
    ];

    const modified: string[] = [];

    for (const file of criticalFiles) {
      const expected = await this.getExpectedHash(file);
      const actual = await this.calculateFileHash(file);

      if (expected !== actual) {
        modified.push(file);
      }
    }

    return {
      valid: modified.length === 0,
      modified
    };
  }
}
```

### Memory Integrity

```typescript
class MemoryIntegrityChecker {
  private readonly PROTECTED_REGIONS: MemoryRegion[] = [];
  private readonly checksums: Map<string, string> = new Map();

  initialize(): void {
    // Store checksums of critical memory regions
    this.PROTECTED_REGIONS.forEach(region => {
      const checksum = this.calculateChecksum(region);
      this.checksums.set(region.name, checksum);
    });
  }

  async checkMemoryIntegrity(): Promise<CheckResult> {
    const violations: string[] = [];

    for (const region of this.PROTECTED_REGIONS) {
      const currentChecksum = this.calculateChecksum(region);
      const expectedChecksum = this.checksums.get(region.name);

      if (currentChecksum !== expectedChecksum) {
        violations.push(region.name);
      }
    }

    if (violations.length > 0) {
      return {
        name: 'memory_integrity',
        status: 'fail',
        details: `Modified regions: ${violations.join(', ')}`
      };
    }

    return { name: 'memory_integrity', status: 'pass' };
  }

  // Detect memory scanning tools
  async detectMemoryScanner(): Promise<boolean> {
    const suspiciousProcesses = [
      'gameguardian',
      'cheatengine',
      'hackode',
      'freedom'
    ];

    // Platform-specific process detection
    const runningProcesses = await this.getRunningProcesses();

    return runningProcesses.some(p =>
      suspiciousProcesses.some(sp =>
        p.toLowerCase().includes(sp)
      )
    );
  }
}
```

### Root/Jailbreak Detection

```typescript
class RootJailbreakDetector {
  async detectRootJailbreak(): Promise<CheckResult> {
    const isRooted = await this.checkRootedDevice();
    const isJailbroken = await this.checkJailbrokenDevice();

    if (isRooted || isJailbroken) {
      return {
        name: 'root_jailbreak',
        status: 'warning',
        details: isRooted ? 'Device rooted' : 'Device jailbroken'
      };
    }

    return { name: 'root_jailbreak', status: 'pass' };
  }

  private async checkRootedDevice(): Promise<boolean> {
    // Android-specific checks
    const indicators = [
      // Check for su binary
      this.fileExists('/system/bin/su'),
      this.fileExists('/system/xbin/su'),
      this.fileExists('/sbin/su'),

      // Check for Magisk
      this.fileExists('/sbin/.magisk'),

      // Check for common root apps
      this.packageInstalled('com.noshufou.android.su'),
      this.packageInstalled('eu.chainfire.supersu'),
      this.packageInstalled('com.topjohnwu.magisk'),

      // Check build tags
      this.checkBuildTags()
    ];

    return (await Promise.all(indicators)).some(v => v);
  }

  private async checkJailbrokenDevice(): Promise<boolean> {
    // iOS-specific checks
    const indicators = [
      // Check for Cydia
      this.fileExists('/Applications/Cydia.app'),
      this.fileExists('/private/var/lib/cydia'),

      // Check for common jailbreak files
      this.fileExists('/usr/sbin/sshd'),
      this.fileExists('/etc/apt'),
      this.fileExists('/bin/bash'),

      // Check if can write outside sandbox
      this.canWriteOutsideSandbox(),

      // Check for Cydia URL scheme
      this.canOpenCydiaURL()
    ];

    return (await Promise.all(indicators)).some(v => v);
  }
}
```

### Hook Detection

```typescript
class HookDetector {
  async detectHooks(): Promise<CheckResult> {
    const hookDetected = await this.checkForHooks();

    if (hookDetected) {
      return {
        name: 'hook_detection',
        status: 'fail',
        details: 'Function hooking detected'
      };
    }

    return { name: 'hook_detection', status: 'pass' };
  }

  private async checkForHooks(): Promise<boolean> {
    // Check if critical functions have been hooked
    const criticalFunctions = [
      'ProcessNetworkMessage',
      'SendPlayerInput',
      'ApplyDamage',
      'ValidatePosition'
    ];

    for (const funcName of criticalFunctions) {
      if (await this.isFunctionHooked(funcName)) {
        return true;
      }
    }

    // Check for Frida
    if (await this.detectFrida()) {
      return true;
    }

    // Check for Xposed
    if (await this.detectXposed()) {
      return true;
    }

    return false;
  }

  private async detectFrida(): Promise<boolean> {
    // Check for Frida server
    const fridaIndicators = [
      this.checkFridaPort(),
      this.checkFridaLibrary(),
      this.checkFridaThreads()
    ];

    return (await Promise.all(fridaIndicators)).some(v => v);
  }
}
```

---

## Behavioral Analysis System

### Statistical Anomaly Detection

```typescript
interface AnomalyThresholds {
  // Accuracy thresholds (per weapon type)
  accuracy: {
    rifle: { max: 0.45, suspicious: 0.35 };
    shotgun: { max: 0.60, suspicious: 0.45 };
    sniper: { max: 0.40, suspicious: 0.30 };
  };

  // Headshot rate thresholds
  headshotRate: {
    max: 0.35;
    suspicious: 0.25;
  };

  // Reaction time (milliseconds)
  reactionTime: {
    min: 150; // Impossible below this
    suspicious: 180;
  };

  // Kill statistics
  killsPerMatch: {
    max: 25; // Theoretical max in 60 player match
    suspicious: 15;
  };

  // Win rate over time
  winRate: {
    suspicious: 0.25; // 25% win rate over 100+ matches
  };
}

class AnomalyDetector {
  private readonly thresholds: AnomalyThresholds;

  async analyzePlayer(
    playerId: string,
    recentMatches: number = 20
  ): Promise<AnomalyReport> {
    const metrics = await this.getPlayerMetrics(playerId, recentMatches);
    const anomalies: Anomaly[] = [];

    // Check accuracy
    const accuracyAnomaly = this.checkAccuracy(metrics);
    if (accuracyAnomaly) anomalies.push(accuracyAnomaly);

    // Check headshot rate
    const headshotAnomaly = this.checkHeadshotRate(metrics);
    if (headshotAnomaly) anomalies.push(headshotAnomaly);

    // Check reaction time
    const reactionAnomaly = this.checkReactionTime(metrics);
    if (reactionAnomaly) anomalies.push(reactionAnomaly);

    // Check kill patterns
    const killAnomaly = this.checkKillPatterns(metrics);
    if (killAnomaly) anomalies.push(killAnomaly);

    // Check win rate
    const winAnomaly = this.checkWinRate(metrics);
    if (winAnomaly) anomalies.push(winAnomaly);

    // Check for impossible events
    const impossibleEvents = await this.checkImpossibleEvents(playerId);
    anomalies.push(...impossibleEvents);

    // Calculate overall suspicion score
    const suspicionScore = this.calculateSuspicionScore(anomalies);

    return {
      playerId,
      matchesAnalyzed: recentMatches,
      anomalies,
      suspicionScore,
      recommendation: this.getRecommendation(suspicionScore)
    };
  }

  private checkAccuracy(metrics: BehavioralMetrics): Anomaly | null {
    const accuracy = metrics.metrics.shotsHit / metrics.metrics.shotsFired;

    if (accuracy > this.thresholds.accuracy.rifle.max) {
      return {
        type: 'accuracy',
        severity: 'high',
        value: accuracy,
        threshold: this.thresholds.accuracy.rifle.max,
        description: `Accuracy ${(accuracy * 100).toFixed(1)}% exceeds maximum`
      };
    }

    if (accuracy > this.thresholds.accuracy.rifle.suspicious) {
      return {
        type: 'accuracy',
        severity: 'medium',
        value: accuracy,
        threshold: this.thresholds.accuracy.rifle.suspicious,
        description: `Accuracy ${(accuracy * 100).toFixed(1)}% is suspiciously high`
      };
    }

    return null;
  }

  private checkReactionTime(metrics: BehavioralMetrics): Anomaly | null {
    const avgReaction = metrics.metrics.averageReactionTime;

    if (avgReaction < this.thresholds.reactionTime.min) {
      return {
        type: 'reaction_time',
        severity: 'critical',
        value: avgReaction,
        threshold: this.thresholds.reactionTime.min,
        description: `Average reaction time ${avgReaction}ms is impossibly fast`
      };
    }

    if (avgReaction < this.thresholds.reactionTime.suspicious) {
      return {
        type: 'reaction_time',
        severity: 'medium',
        value: avgReaction,
        threshold: this.thresholds.reactionTime.suspicious,
        description: `Average reaction time ${avgReaction}ms is suspiciously fast`
      };
    }

    return null;
  }

  private async checkImpossibleEvents(playerId: string): Promise<Anomaly[]> {
    const anomalies: Anomaly[] = [];
    const events = await this.getRecentEvents(playerId);

    // Check for through-wall kills
    const wallKills = events.filter(e =>
      e.type === 'kill' && e.metadata?.throughWall
    );
    if (wallKills.length > 0) {
      anomalies.push({
        type: 'through_wall_kill',
        severity: 'critical',
        value: wallKills.length,
        description: `${wallKills.length} kills through solid walls`
      });
    }

    // Check for impossible movement
    const teleports = events.filter(e =>
      e.type === 'movement' && e.metadata?.teleport
    );
    if (teleports.length > 0) {
      anomalies.push({
        type: 'teleportation',
        severity: 'critical',
        value: teleports.length,
        description: `${teleports.length} impossible position changes`
      });
    }

    // Check for speed violations
    const speedViolations = events.filter(e =>
      e.type === 'movement' && e.metadata?.speedExceeded
    );
    if (speedViolations.length > 5) {
      anomalies.push({
        type: 'speed_hack',
        severity: 'high',
        value: speedViolations.length,
        description: `${speedViolations.length} speed limit violations`
      });
    }

    return anomalies;
  }

  private calculateSuspicionScore(anomalies: Anomaly[]): number {
    let score = 0;

    for (const anomaly of anomalies) {
      switch (anomaly.severity) {
        case 'critical':
          score += 40;
          break;
        case 'high':
          score += 25;
          break;
        case 'medium':
          score += 10;
          break;
        case 'low':
          score += 5;
          break;
      }
    }

    return Math.min(100, score);
  }

  private getRecommendation(score: number): string {
    if (score >= 80) return 'immediate_ban';
    if (score >= 60) return 'manual_review_priority';
    if (score >= 40) return 'manual_review';
    if (score >= 20) return 'flag_for_monitoring';
    return 'no_action';
  }
}
```

### Pattern Recognition

```typescript
class PatternRecognizer {
  // Detect aimbot patterns
  async detectAimbot(
    playerId: string,
    aimData: AimTrackingData[]
  ): Promise<PatternResult> {
    const indicators: string[] = [];

    // Check for instant aim snapping
    const snapCount = this.countAimSnaps(aimData);
    if (snapCount > 10) {
      indicators.push(`${snapCount} instant aim snaps detected`);
    }

    // Check for perfect tracking
    const trackingPerfection = this.measureTrackingPerfection(aimData);
    if (trackingPerfection > 0.95) {
      indicators.push(`Tracking perfection: ${(trackingPerfection * 100).toFixed(1)}%`);
    }

    // Check for unnatural aim smoothness
    const smoothness = this.measureAimSmoothness(aimData);
    if (smoothness > 0.98) {
      indicators.push(`Unnatural aim smoothness: ${(smoothness * 100).toFixed(1)}%`);
    }

    // Check for FOV-based targeting
    const fovPattern = this.detectFOVTargeting(aimData);
    if (fovPattern.detected) {
      indicators.push(`FOV-based targeting pattern (FOV: ${fovPattern.estimatedFOV}°)`);
    }

    return {
      type: 'aimbot',
      detected: indicators.length >= 2,
      confidence: indicators.length / 4,
      indicators
    };
  }

  // Detect wallhack patterns
  async detectWallhack(
    playerId: string,
    behaviorData: PlayerBehaviorData[]
  ): Promise<PatternResult> {
    const indicators: string[] = [];

    // Check pre-aiming at hidden enemies
    const preAimScore = this.measurePreAiming(behaviorData);
    if (preAimScore > 0.7) {
      indicators.push(`Pre-aiming at hidden enemies: ${(preAimScore * 100).toFixed(1)}%`);
    }

    // Check for tracking through walls
    const wallTracking = this.measureWallTracking(behaviorData);
    if (wallTracking > 0.5) {
      indicators.push(`Tracking enemies through walls: ${(wallTracking * 100).toFixed(1)}%`);
    }

    // Check for perfect information decisions
    const decisionAccuracy = this.measureDecisionAccuracy(behaviorData);
    if (decisionAccuracy > 0.9) {
      indicators.push(`Perfect positioning decisions: ${(decisionAccuracy * 100).toFixed(1)}%`);
    }

    return {
      type: 'wallhack',
      detected: indicators.length >= 2,
      confidence: indicators.length / 3,
      indicators
    };
  }

  private countAimSnaps(aimData: AimTrackingData[]): number {
    let snaps = 0;

    for (let i = 1; i < aimData.length; i++) {
      const prev = aimData[i - 1];
      const curr = aimData[i];
      const timeDelta = curr.timestamp - prev.timestamp;
      const angleDelta = this.calculateAngleDelta(prev, curr);

      // Snap detected if large angle change in very short time
      if (timeDelta < 50 && angleDelta > 30) {
        snaps++;
      }
    }

    return snaps;
  }
}
```

---

## Player Report System

### Report Categories

```typescript
enum ReportCategory {
  AIMBOT = 'aimbot',
  WALLHACK = 'wallhack',
  SPEEDHACK = 'speedhack',
  TEAMING = 'teaming',
  EXPLOITING = 'exploiting',
  HARASSMENT = 'harassment',
  INAPPROPRIATE_NAME = 'inappropriate_name',
  OTHER = 'other'
}

interface PlayerReport {
  id: string;
  reporterId: string;
  reportedId: string;
  matchId?: string;
  category: ReportCategory;
  description?: string;
  evidence?: {
    timestamps: number[];
    videoClipId?: string;
  };
  status: 'pending' | 'reviewing' | 'resolved';
  resolution?: {
    action: 'no_action' | 'warning' | 'temp_ban' | 'perm_ban';
    reason?: string;
    reviewerId?: string;
    reviewedAt: string;
  };
  createdAt: string;
}
```

### Report Submission

```typescript
class ReportService {
  private readonly REPORTS_PER_DAY_LIMIT = 10;

  async submitReport(
    reporterId: string,
    report: Omit<PlayerReport, 'id' | 'status' | 'createdAt'>
  ): Promise<ReportResult> {
    // 1. Rate limit check
    const todayReports = await this.getReportCount(reporterId, 'today');
    if (todayReports >= this.REPORTS_PER_DAY_LIMIT) {
      return {
        success: false,
        error: 'rate_limited',
        message: 'Daily report limit reached'
      };
    }

    // 2. Check for duplicate report
    const existing = await this.findDuplicateReport(
      reporterId,
      report.reportedId,
      report.matchId
    );
    if (existing) {
      return {
        success: false,
        error: 'duplicate',
        message: 'You already reported this player for this match'
      };
    }

    // 3. Validate report
    const validation = await this.validateReport(report);
    if (!validation.valid) {
      return {
        success: false,
        error: 'invalid',
        message: validation.reason
      };
    }

    // 4. Create report
    const newReport: PlayerReport = {
      id: generateId(),
      ...report,
      reporterId,
      status: 'pending',
      createdAt: new Date().toISOString()
    };

    await this.saveReport(newReport);

    // 5. Trigger automatic analysis
    this.triggerAutoAnalysis(newReport);

    // 6. Check if player should be auto-flagged
    await this.checkAutoFlag(report.reportedId);

    return {
      success: true,
      reportId: newReport.id
    };
  }

  private async checkAutoFlag(reportedId: string): Promise<void> {
    // Count recent reports against this player
    const recentReports = await this.getReportsAgainst(reportedId, {
      since: Date.now() - 24 * 60 * 60 * 1000 // Last 24 hours
    });

    // Different thresholds for different categories
    const aimbotReports = recentReports.filter(
      r => r.category === ReportCategory.AIMBOT
    );
    const uniqueReporters = new Set(recentReports.map(r => r.reporterId));

    // Flag if many unique players report for same reason
    if (aimbotReports.length >= 5 && uniqueReporters.size >= 4) {
      await this.flagForPriorityReview(reportedId, 'multiple_aimbot_reports');
    }
  }
}
```

### Review Queue

```typescript
interface ReviewQueueItem {
  playerId: string;
  priority: 'critical' | 'high' | 'normal' | 'low';
  reports: PlayerReport[];
  autoAnalysis?: AnomalyReport;
  replayData?: string[];
  matchHistory: MatchSummary[];
  previousViolations: Violation[];
  queuedAt: string;
}

class ReviewQueue {
  async getNextForReview(reviewerId: string): Promise<ReviewQueueItem | null> {
    // Get items sorted by priority and time
    const items = await this.getQueuedItems({
      notAssignedTo: reviewerId,
      orderBy: ['priority', 'queuedAt']
    });

    if (items.length === 0) return null;

    // Assign to reviewer
    const item = items[0];
    await this.assignToReviewer(item.playerId, reviewerId);

    return item;
  }

  async submitReview(
    reviewerId: string,
    playerId: string,
    decision: ReviewDecision
  ): Promise<void> {
    // Validate reviewer has authority
    const reviewer = await this.getReviewer(reviewerId);
    if (!reviewer || !this.canMakeDecision(reviewer, decision)) {
      throw new Error('Unauthorized');
    }

    // Apply decision
    await this.applyDecision(playerId, decision, reviewerId);

    // Update reports
    await this.resolveReports(playerId, decision);

    // Remove from queue
    await this.removeFromQueue(playerId);

    // Log for audit
    await this.logReviewAction(reviewerId, playerId, decision);
  }
}
```

---

## Ban System

### Ban Types

```typescript
enum BanType {
  WARNING = 'warning',           // No gameplay restriction
  CHAT_MUTE = 'chat_mute',       // Can play, cannot chat
  TEMP_BAN_1H = 'temp_ban_1h',   // 1 hour ban
  TEMP_BAN_24H = 'temp_ban_24h', // 24 hour ban
  TEMP_BAN_7D = 'temp_ban_7d',   // 7 day ban
  TEMP_BAN_30D = 'temp_ban_30d', // 30 day ban
  PERM_BAN = 'perm_ban',         // Permanent ban
  HWID_BAN = 'hwid_ban'          // Hardware ID ban
}

interface Ban {
  id: string;
  playerId: string;
  type: BanType;
  reason: string;
  category: ReportCategory;
  evidence?: string;
  issuedBy: string; // 'system' or reviewer ID
  issuedAt: string;
  expiresAt?: string;
  appealed: boolean;
  appealResult?: 'upheld' | 'overturned' | 'reduced';
}
```

### Escalation Policy

```typescript
const ESCALATION_POLICY: Record<ReportCategory, BanType[]> = {
  [ReportCategory.AIMBOT]: [
    BanType.WARNING,        // First offense (if borderline)
    BanType.TEMP_BAN_7D,    // Confirmed first offense
    BanType.TEMP_BAN_30D,   // Second offense
    BanType.PERM_BAN        // Third offense
  ],

  [ReportCategory.WALLHACK]: [
    BanType.WARNING,
    BanType.TEMP_BAN_7D,
    BanType.TEMP_BAN_30D,
    BanType.PERM_BAN
  ],

  [ReportCategory.SPEEDHACK]: [
    BanType.TEMP_BAN_24H,   // Usually clear-cut
    BanType.TEMP_BAN_7D,
    BanType.PERM_BAN
  ],

  [ReportCategory.TEAMING]: [
    BanType.WARNING,
    BanType.TEMP_BAN_1H,
    BanType.TEMP_BAN_24H,
    BanType.TEMP_BAN_7D
  ],

  [ReportCategory.EXPLOITING]: [
    BanType.WARNING,
    BanType.TEMP_BAN_24H,
    BanType.TEMP_BAN_7D,
    BanType.PERM_BAN
  ],

  [ReportCategory.HARASSMENT]: [
    BanType.CHAT_MUTE,
    BanType.TEMP_BAN_1H,
    BanType.TEMP_BAN_24H,
    BanType.TEMP_BAN_7D
  ],

  [ReportCategory.INAPPROPRIATE_NAME]: [
    BanType.WARNING         // Force name change
  ],

  [ReportCategory.OTHER]: [
    BanType.WARNING,
    BanType.TEMP_BAN_1H,
    BanType.TEMP_BAN_24H
  ]
};

class BanEscalator {
  async determineBan(
    playerId: string,
    category: ReportCategory,
    severity: 'minor' | 'moderate' | 'severe'
  ): Promise<BanType> {
    // Get previous violations in this category
    const previousBans = await this.getPreviousBans(playerId, category);
    const offenseCount = previousBans.length;

    const escalationPath = ESCALATION_POLICY[category];

    // Adjust starting point based on severity
    let index = offenseCount;
    if (severity === 'severe' && index < escalationPath.length - 1) {
      index++;
    }

    // Cap at max punishment
    index = Math.min(index, escalationPath.length - 1);

    return escalationPath[index];
  }
}
```

### Ban Enforcement

```typescript
class BanEnforcement {
  async applyBan(ban: Ban): Promise<void> {
    // Store ban
    await this.storeBan(ban);

    // Apply effects based on ban type
    switch (ban.type) {
      case BanType.WARNING:
        await this.sendWarningNotification(ban.playerId, ban.reason);
        break;

      case BanType.CHAT_MUTE:
        await this.applyChatMute(ban.playerId, ban.expiresAt);
        break;

      case BanType.HWID_BAN:
        // Also ban all known hardware IDs
        const hwids = await this.getPlayerHWIDs(ban.playerId);
        for (const hwid of hwids) {
          await this.banHWID(hwid, ban);
        }
        // Fall through to disconnect

      case BanType.PERM_BAN:
      case BanType.TEMP_BAN_1H:
      case BanType.TEMP_BAN_24H:
      case BanType.TEMP_BAN_7D:
      case BanType.TEMP_BAN_30D:
        // Kick from current game
        await this.disconnectPlayer(ban.playerId, ban.reason);
        // Send ban notification
        await this.sendBanNotification(ban);
        break;
    }

    // Log ban for analytics
    await Analytics.track('player_banned', {
      playerId: ban.playerId,
      type: ban.type,
      category: ban.category,
      issuedBy: ban.issuedBy
    });
  }

  async checkBanStatus(playerId: string): Promise<BanStatus> {
    // Check for active ban
    const activeBan = await this.getActiveBan(playerId);

    if (!activeBan) {
      // Check HWID ban
      const hwid = await this.getDeviceHWID();
      const hwidBan = await this.getHWIDBan(hwid);

      if (hwidBan) {
        return {
          banned: true,
          type: BanType.HWID_BAN,
          reason: 'This device is banned',
          expiresAt: hwidBan.expiresAt
        };
      }

      return { banned: false };
    }

    // Check if expired
    if (activeBan.expiresAt && new Date(activeBan.expiresAt) < new Date()) {
      await this.expireBan(activeBan.id);
      return { banned: false };
    }

    return {
      banned: true,
      type: activeBan.type,
      reason: activeBan.reason,
      expiresAt: activeBan.expiresAt,
      canAppeal: !activeBan.appealed
    };
  }
}
```

### Ban Appeals

```typescript
interface BanAppeal {
  id: string;
  banId: string;
  playerId: string;
  reason: string;
  evidence?: string;
  status: 'pending' | 'reviewing' | 'resolved';
  result?: 'upheld' | 'overturned' | 'reduced';
  reviewerNotes?: string;
  reviewerId?: string;
  createdAt: string;
  resolvedAt?: string;
}

class BanAppealService {
  async submitAppeal(
    playerId: string,
    banId: string,
    reason: string
  ): Promise<AppealResult> {
    // 1. Get ban
    const ban = await BanService.getBan(banId);
    if (!ban || ban.playerId !== playerId) {
      return { success: false, error: 'ban_not_found' };
    }

    // 2. Check if already appealed
    if (ban.appealed) {
      return { success: false, error: 'already_appealed' };
    }

    // 3. Check if appeal allowed
    if (!this.isAppealable(ban)) {
      return { success: false, error: 'not_appealable' };
    }

    // 4. Create appeal
    const appeal: BanAppeal = {
      id: generateId(),
      banId,
      playerId,
      reason,
      status: 'pending',
      createdAt: new Date().toISOString()
    };

    await this.saveAppeal(appeal);

    // 5. Mark ban as appealed
    await BanService.markAsAppealed(banId);

    return {
      success: true,
      appealId: appeal.id,
      estimatedReviewTime: '24-48 hours'
    };
  }

  private isAppealable(ban: Ban): boolean {
    // Can't appeal warnings
    if (ban.type === BanType.WARNING) return false;

    // Can't appeal very short bans
    if (ban.type === BanType.TEMP_BAN_1H) return false;

    // Can only appeal within 7 days
    const daysSinceBan = this.daysSince(ban.issuedAt);
    if (daysSinceBan > 7) return false;

    return true;
  }

  async resolveAppeal(
    appealId: string,
    reviewerId: string,
    result: 'upheld' | 'overturned' | 'reduced',
    notes?: string
  ): Promise<void> {
    const appeal = await this.getAppeal(appealId);

    appeal.status = 'resolved';
    appeal.result = result;
    appeal.reviewerId = reviewerId;
    appeal.reviewerNotes = notes;
    appeal.resolvedAt = new Date().toISOString();

    await this.updateAppeal(appeal);

    // Apply result
    const ban = await BanService.getBan(appeal.banId);

    switch (result) {
      case 'overturned':
        await BanService.liftBan(ban.id, 'appeal_overturned');
        await this.notifyPlayer(appeal.playerId, 'Your appeal was successful');
        break;

      case 'reduced':
        await BanService.reduceBan(ban.id);
        await this.notifyPlayer(appeal.playerId, 'Your ban has been reduced');
        break;

      case 'upheld':
        await this.notifyPlayer(appeal.playerId, 'Your appeal was denied');
        break;
    }
  }
}
```

---

## Hardware ID Collection

```typescript
interface DeviceFingerprint {
  // Android
  androidId?: string;
  serialNumber?: string;
  buildId?: string;
  deviceModel?: string;
  manufacturer?: string;

  // iOS
  identifierForVendor?: string;
  deviceModel?: string;

  // Generic
  screenResolution?: string;
  timezone?: string;
  language?: string;

  // Generated composite ID
  compositeId: string;
}

class HWIDCollector {
  async collectFingerprint(): Promise<DeviceFingerprint> {
    const fingerprint: Partial<DeviceFingerprint> = {};

    if (Platform.isAndroid()) {
      fingerprint.androidId = await this.getAndroidId();
      fingerprint.serialNumber = await this.getSerialNumber();
      fingerprint.buildId = await this.getBuildId();
    }

    if (Platform.isIOS()) {
      fingerprint.identifierForVendor = await this.getIDFV();
    }

    fingerprint.deviceModel = await this.getDeviceModel();
    fingerprint.manufacturer = await this.getManufacturer();
    fingerprint.screenResolution = await this.getScreenResolution();
    fingerprint.timezone = Intl.DateTimeFormat().resolvedOptions().timeZone;
    fingerprint.language = navigator.language;

    // Generate composite ID from all factors
    fingerprint.compositeId = await this.generateCompositeId(fingerprint);

    return fingerprint as DeviceFingerprint;
  }

  private async generateCompositeId(
    fingerprint: Partial<DeviceFingerprint>
  ): Promise<string> {
    // Create stable hash from multiple factors
    const components = [
      fingerprint.androidId || fingerprint.identifierForVendor || '',
      fingerprint.deviceModel || '',
      fingerprint.manufacturer || '',
      fingerprint.screenResolution || ''
    ];

    const combined = components.join('|');
    return await this.sha256(combined);
  }
}
```

---

## Analytics and Monitoring

### Anti-Cheat Metrics

```typescript
const ANTI_CHEAT_METRICS = {
  // Detection metrics
  integrityCheckResults: 'ac_integrity_check_total',
  behavioralAnomalies: 'ac_behavioral_anomaly_total',
  detectionsByType: 'ac_detections_by_type',

  // Report metrics
  reportsSubmitted: 'ac_reports_submitted_total',
  reportsByCategory: 'ac_reports_by_category',
  reportResolutionTime: 'ac_report_resolution_time_ms',

  // Ban metrics
  bansIssued: 'ac_bans_issued_total',
  bansByType: 'ac_bans_by_type',
  appealsSubmitted: 'ac_appeals_submitted_total',
  appealResults: 'ac_appeal_results',

  // False positive tracking
  overturndBans: 'ac_overturned_bans_total',
  falsePositiveRate: 'ac_false_positive_rate'
};

class AntiCheatAnalytics {
  async generateDailyReport(): Promise<DailyACReport> {
    const today = new Date();

    return {
      date: today.toISOString(),
      integrityChecks: {
        total: await this.getMetric('integrity_checks_total'),
        passed: await this.getMetric('integrity_checks_passed'),
        failed: await this.getMetric('integrity_checks_failed'),
        failureRate: await this.calculateFailureRate()
      },
      detections: {
        total: await this.getMetric('detections_total'),
        byType: await this.getDetectionsByType()
      },
      reports: {
        submitted: await this.getMetric('reports_submitted'),
        resolved: await this.getMetric('reports_resolved'),
        avgResolutionTime: await this.getAvgResolutionTime()
      },
      bans: {
        issued: await this.getMetric('bans_issued'),
        byType: await this.getBansByType(),
        appealed: await this.getMetric('bans_appealed'),
        overturned: await this.getMetric('bans_overturned')
      },
      falsePositiveRate: await this.calculateFalsePositiveRate()
    };
  }
}
```

---

## Integration Points

### Dependencies

- BACK-015: Server-Authoritative Validation (foundation)
- BACK-011: Player Profile Service (ban status storage)
- BACK-020: Server Monitoring (alerts)
- Push notification service

### Event Hooks

```typescript
// Events emitted by this service
const ANTI_CHEAT_EVENTS = [
  'integrity_check_failed',
  'anomaly_detected',
  'report_submitted',
  'player_flagged',
  'ban_issued',
  'ban_expired',
  'appeal_submitted',
  'appeal_resolved'
];
```

---

## Testing Strategy

### Test Scenarios

1. **Integrity Check Testing**
   - Verify detection of modified binaries
   - Test root/jailbreak detection accuracy
   - Validate hook detection

2. **Behavioral Analysis Testing**
   - Create synthetic cheat patterns
   - Verify statistical anomaly detection
   - Test false positive rates with legitimate pro players

3. **Report System Testing**
   - Rate limiting verification
   - Duplicate detection
   - Auto-flag thresholds

4. **Ban System Testing**
   - Escalation policy enforcement
   - Ban expiration handling
   - Appeal workflow

5. **HWID Ban Testing**
   - Device fingerprint consistency
   - Ban evasion detection
