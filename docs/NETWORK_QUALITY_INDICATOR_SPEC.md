# Network Quality Indicator Specification

## Overview

This document specifies the Network Quality Indicator system for Plunderstorm Mobile, providing players with real-time visibility into their connection status, helping diagnose performance issues, and managing disconnection/reconnection scenarios gracefully.

## Network Metrics

### Core Metrics

```typescript
interface NetworkMetrics {
  // Latency
  ping: number;                 // Round-trip time in ms
  pingAvg: number;              // Rolling average (last 10)
  pingJitter: number;           // Variation in ping
  pingMin: number;              // Session minimum
  pingMax: number;              // Session maximum

  // Packet Loss
  packetLoss: number;           // Percentage (0-100)
  packetsReceived: number;
  packetsLost: number;

  // Bandwidth
  downloadRate: number;         // KB/s
  uploadRate: number;           // KB/s

  // Connection
  connectionType: ConnectionType;
  signalStrength?: number;      // For WiFi/cellular
  serverRegion: string;
  serverPing: number;

  // Quality
  qualityScore: number;         // 0-100 composite score
  qualityLevel: QualityLevel;
}

enum ConnectionType {
  WIFI = 'wifi',
  CELLULAR_5G = '5g',
  CELLULAR_LTE = 'lte',
  CELLULAR_3G = '3g',
  ETHERNET = 'ethernet',
  UNKNOWN = 'unknown'
}

enum QualityLevel {
  EXCELLENT = 'excellent',      // Green
  GOOD = 'good',                // Light green
  FAIR = 'fair',                // Yellow
  POOR = 'poor',                // Orange
  CRITICAL = 'critical'         // Red
}
```

### Quality Thresholds

| Metric | Excellent | Good | Fair | Poor | Critical |
|--------|-----------|------|------|------|----------|
| Ping | <30ms | 30-60ms | 60-100ms | 100-200ms | >200ms |
| Jitter | <10ms | 10-30ms | 30-50ms | 50-100ms | >100ms |
| Packet Loss | 0% | <1% | 1-3% | 3-10% | >10% |
| Quality Score | 90-100 | 70-89 | 50-69 | 30-49 | <30 |

### Quality Score Calculation

```typescript
function calculateQualityScore(metrics: NetworkMetrics): number {
  // Weighted components
  const pingScore = calculatePingScore(metrics.ping);           // 40%
  const jitterScore = calculateJitterScore(metrics.pingJitter); // 20%
  const lossScore = calculateLossScore(metrics.packetLoss);     // 40%

  const composite = (pingScore * 0.4) + (jitterScore * 0.2) + (lossScore * 0.4);

  return Math.round(composite);
}

function calculatePingScore(ping: number): number {
  if (ping < 30) return 100;
  if (ping < 60) return 80 + (60 - ping) * 0.67;
  if (ping < 100) return 60 + (100 - ping) * 0.5;
  if (ping < 200) return 30 + (200 - ping) * 0.3;
  return Math.max(0, 30 - (ping - 200) * 0.1);
}

function calculateJitterScore(jitter: number): number {
  if (jitter < 10) return 100;
  if (jitter < 30) return 70 + (30 - jitter) * 1.5;
  if (jitter < 50) return 50 + (50 - jitter);
  if (jitter < 100) return 20 + (100 - jitter) * 0.6;
  return Math.max(0, 20 - (jitter - 100) * 0.2);
}

function calculateLossScore(loss: number): number {
  if (loss === 0) return 100;
  if (loss < 1) return 80 + (1 - loss) * 20;
  if (loss < 3) return 60 + (3 - loss) * 10;
  if (loss < 10) return 20 + (10 - loss) * 5.7;
  return Math.max(0, 20 - (loss - 10) * 2);
}

function getQualityLevel(score: number): QualityLevel {
  if (score >= 90) return QualityLevel.EXCELLENT;
  if (score >= 70) return QualityLevel.GOOD;
  if (score >= 50) return QualityLevel.FAIR;
  if (score >= 30) return QualityLevel.POOR;
  return QualityLevel.CRITICAL;
}
```

## HUD Display

### Minimal Indicator (Default)

```
┌──────┐
│ 🟢 │  ← Color-coded signal bars
│ ▌▌▌▌│     (Green = Excellent)
└──────┘

Signal Strength Icons:
▌▌▌▌ = Excellent (4 bars)
▌▌▌░ = Good (3 bars)
▌▌░░ = Fair (2 bars)
▌░░░ = Poor (1 bar)
░░░░ = Critical (0 bars, blinking)
```

### Expanded Indicator

```
┌─────────────────────────┐
│ 🟢 GOOD CONNECTION      │
│ Ping: 45ms | Loss: 0.2% │
└─────────────────────────┘
```

### Detailed Stats Panel

```
┌─────────────────────────────────────────────────────────────────┐
│  NETWORK STATS                                        [X Close] │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CONNECTION                                                     │
│  Type: WiFi                                                     │
│  Server: US-West (Los Angeles)                                  │
│  Status: Connected                                              │
│                                                                 │
│  LATENCY                                                        │
│  Ping: 45ms (avg)                                              │
│  Jitter: 8ms                                                   │
│  Min/Max: 32ms / 78ms                                          │
│                                                                 │
│  [█████████████████░░░]                                        │
│  0ms          100ms        200ms                                │
│                                                                 │
│  RELIABILITY                                                    │
│  Packet Loss: 0.2%                                             │
│  Packets Received: 45,230                                      │
│  Packets Lost: 90                                              │
│                                                                 │
│  BANDWIDTH                                                      │
│  Download: 125 KB/s                                            │
│  Upload: 45 KB/s                                               │
│                                                                 │
│  QUALITY SCORE: 82/100 (Good)                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## HUD Configuration

### Indicator Placement

```typescript
interface NetworkIndicatorConfig {
  // Display
  enabled: boolean;
  position: IndicatorPosition;
  style: 'minimal' | 'expanded' | 'hidden';

  // Thresholds for warnings
  showWarningBelow: QualityLevel;   // Show warning below this
  showDisconnectAt: number;         // ms of no response

  // Behavior
  autoExpand: boolean;              // Expand when issues detected
  expandDuration: number;           // How long to stay expanded
  hideWhenGood: boolean;            // Hide if connection excellent

  // Ping display
  showPingNumber: boolean;
  updateInterval: number;           // ms between updates
}

enum IndicatorPosition {
  TOP_LEFT = 'top_left',
  TOP_RIGHT = 'top_right',
  BOTTOM_LEFT = 'bottom_left',
  BOTTOM_RIGHT = 'bottom_right'
}

const DEFAULT_INDICATOR_CONFIG: NetworkIndicatorConfig = {
  enabled: true,
  position: IndicatorPosition.TOP_RIGHT,
  style: 'minimal',
  showWarningBelow: QualityLevel.FAIR,
  showDisconnectAt: 5000,
  autoExpand: true,
  expandDuration: 5000,
  hideWhenGood: false,
  showPingNumber: false,
  updateInterval: 1000
};
```

### Visual States

```typescript
interface IndicatorVisualState {
  quality: QualityLevel;
  icon: string;
  color: string;
  bars: number;             // 0-4
  pulsing: boolean;
  showWarning: boolean;
  warningMessage?: string;
}

const VISUAL_STATES: Record<QualityLevel, IndicatorVisualState> = {
  [QualityLevel.EXCELLENT]: {
    quality: QualityLevel.EXCELLENT,
    icon: 'icon_signal_full',
    color: '#00FF00',       // Bright green
    bars: 4,
    pulsing: false,
    showWarning: false
  },
  [QualityLevel.GOOD]: {
    quality: QualityLevel.GOOD,
    icon: 'icon_signal_good',
    color: '#90EE90',       // Light green
    bars: 3,
    pulsing: false,
    showWarning: false
  },
  [QualityLevel.FAIR]: {
    quality: QualityLevel.FAIR,
    icon: 'icon_signal_fair',
    color: '#FFFF00',       // Yellow
    bars: 2,
    pulsing: false,
    showWarning: true,
    warningMessage: 'Connection unstable'
  },
  [QualityLevel.POOR]: {
    quality: QualityLevel.POOR,
    icon: 'icon_signal_poor',
    color: '#FFA500',       // Orange
    bars: 1,
    pulsing: true,
    showWarning: true,
    warningMessage: 'Poor connection - expect lag'
  },
  [QualityLevel.CRITICAL]: {
    quality: QualityLevel.CRITICAL,
    icon: 'icon_signal_critical',
    color: '#FF0000',       // Red
    bars: 0,
    pulsing: true,
    showWarning: true,
    warningMessage: 'Connection lost'
  }
};
```

## Warning System

### Warning Notifications

```typescript
interface NetworkWarning {
  type: WarningType;
  severity: 'info' | 'warning' | 'error';
  message: string;
  icon: string;
  autoDismiss: boolean;
  dismissTime: number;
  actions?: WarningAction[];
}

enum WarningType {
  HIGH_PING = 'high_ping',
  PACKET_LOSS = 'packet_loss',
  DISCONNECTING = 'disconnecting',
  DISCONNECTED = 'disconnected',
  RECONNECTING = 'reconnecting',
  RECONNECTED = 'reconnected',
  SERVER_ISSUE = 'server_issue',
  WIFI_WEAK = 'wifi_weak',
  CELLULAR_WEAK = 'cellular_weak'
}

const NETWORK_WARNINGS: Record<WarningType, NetworkWarning> = {
  [WarningType.HIGH_PING]: {
    type: WarningType.HIGH_PING,
    severity: 'warning',
    message: 'High latency detected',
    icon: 'icon_warning_ping',
    autoDismiss: true,
    dismissTime: 5000
  },
  [WarningType.PACKET_LOSS]: {
    type: WarningType.PACKET_LOSS,
    severity: 'warning',
    message: 'Packet loss detected - game may stutter',
    icon: 'icon_warning_loss',
    autoDismiss: true,
    dismissTime: 5000
  },
  [WarningType.DISCONNECTING]: {
    type: WarningType.DISCONNECTING,
    severity: 'error',
    message: 'Connection interrupted...',
    icon: 'icon_disconnecting',
    autoDismiss: false,
    dismissTime: 0
  },
  [WarningType.DISCONNECTED]: {
    type: WarningType.DISCONNECTED,
    severity: 'error',
    message: 'Disconnected from server',
    icon: 'icon_disconnected',
    autoDismiss: false,
    dismissTime: 0,
    actions: [
      { label: 'Reconnect', action: 'reconnect' },
      { label: 'Leave Match', action: 'leave' }
    ]
  },
  [WarningType.RECONNECTING]: {
    type: WarningType.RECONNECTING,
    severity: 'info',
    message: 'Reconnecting...',
    icon: 'icon_reconnecting',
    autoDismiss: false,
    dismissTime: 0
  },
  [WarningType.RECONNECTED]: {
    type: WarningType.RECONNECTED,
    severity: 'info',
    message: 'Connection restored!',
    icon: 'icon_connected',
    autoDismiss: true,
    dismissTime: 3000
  }
};
```

### Warning Display

```
HIGH PING WARNING:
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️ High latency detected (145ms)                              │
│  You may experience delayed actions.                           │
└─────────────────────────────────────────────────────────────────┘

DISCONNECTION WARNING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│           🔴 CONNECTION LOST                                   │
│                                                                 │
│     Attempting to reconnect... (5 seconds)                     │
│                                                                 │
│     [████████████░░░░░░░░░░░░░░]                              │
│                                                                 │
│     [Cancel and Leave Match]                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Reconnection System

### Reconnection Flow

```typescript
interface ReconnectionConfig {
  maxAttempts: 5;
  attemptInterval: number[];    // [2000, 4000, 6000, 8000, 10000]
  gracePeriod: number;          // Time to rejoin match (60s)
  showProgress: boolean;
  allowCancel: boolean;
}

class ReconnectionManager {
  private config: ReconnectionConfig;
  private attempts: number = 0;
  private isReconnecting: boolean = false;
  private lastMatchId?: string;

  async handleDisconnection(matchId: string): Promise<boolean> {
    this.lastMatchId = matchId;
    this.isReconnecting = true;
    this.attempts = 0;

    showReconnectionUI();

    while (this.attempts < this.config.maxAttempts && this.isReconnecting) {
      const delay = this.config.attemptInterval[this.attempts] || 10000;
      updateReconnectionUI(this.attempts + 1, this.config.maxAttempts, delay);

      await sleep(delay);

      if (!this.isReconnecting) break;  // Cancelled

      const success = await this.attemptReconnection();

      if (success) {
        this.isReconnecting = false;
        showReconnectedUI();
        return true;
      }

      this.attempts++;
    }

    // All attempts failed
    this.isReconnecting = false;
    showReconnectionFailedUI();
    return false;
  }

  private async attemptReconnection(): Promise<boolean> {
    try {
      // Attempt to reconnect to server
      const connected = await connectToServer();
      if (!connected) return false;

      // Check if match still active
      const matchStatus = await getMatchStatus(this.lastMatchId);
      if (matchStatus.status !== 'active') {
        return false;  // Match ended
      }

      // Check grace period
      if (matchStatus.disconnectTime > this.config.gracePeriod * 1000) {
        return false;  // Grace period expired
      }

      // Rejoin match
      const rejoined = await rejoinMatch(this.lastMatchId);
      return rejoined;

    } catch (error) {
      return false;
    }
  }

  cancelReconnection(): void {
    this.isReconnecting = false;
  }
}
```

### Reconnection UI

```
RECONNECTING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    🔄 RECONNECTING                             │
│                                                                 │
│     Attempt 2 of 5                                             │
│                                                                 │
│     [████████░░░░░░░░░░░░░░░░░░]  4 seconds                   │
│                                                                 │
│     Your character is protected during reconnection.          │
│                                                                 │
│                  [Cancel and Leave]                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

RECONNECTION FAILED:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    ❌ RECONNECTION FAILED                      │
│                                                                 │
│     Unable to reconnect to the match.                         │
│                                                                 │
│     Your progress has been saved.                             │
│                                                                 │
│              [Return to Main Menu]                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Server Region Selection

### Region Display

```typescript
interface ServerRegion {
  id: string;
  name: string;
  displayName: string;
  ping: number;
  status: 'online' | 'maintenance' | 'offline';
  recommended: boolean;
}

const SERVER_REGIONS: ServerRegion[] = [
  { id: 'us-west', name: 'US West', displayName: 'US West (Los Angeles)', ping: 0, status: 'online', recommended: false },
  { id: 'us-east', name: 'US East', displayName: 'US East (Virginia)', ping: 0, status: 'online', recommended: false },
  { id: 'eu-west', name: 'EU West', displayName: 'Europe (London)', ping: 0, status: 'online', recommended: false },
  { id: 'eu-central', name: 'EU Central', displayName: 'Europe (Frankfurt)', ping: 0, status: 'online', recommended: false },
  { id: 'asia-se', name: 'Asia SE', displayName: 'Asia (Singapore)', ping: 0, status: 'online', recommended: false },
  { id: 'asia-ne', name: 'Asia NE', displayName: 'Asia (Tokyo)', ping: 0, status: 'online', recommended: false },
  { id: 'oceania', name: 'Oceania', displayName: 'Oceania (Sydney)', ping: 0, status: 'online', recommended: false }
];

async function measureRegionPings(): Promise<ServerRegion[]> {
  const measurements = await Promise.all(
    SERVER_REGIONS.map(async region => {
      const ping = await measurePing(region.id);
      return { ...region, ping };
    })
  );

  // Mark recommended (lowest ping that's online)
  const sorted = measurements.filter(r => r.status === 'online').sort((a, b) => a.ping - b.ping);
  if (sorted.length > 0) {
    sorted[0].recommended = true;
  }

  return measurements;
}
```

### Region Selection UI

```
┌─────────────────────────────────────────────────────────────────┐
│  SERVER REGION                                      [Refresh 🔄]│
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Current: US West (Los Angeles)                    45ms        │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ ⭐ US West (Los Angeles)           45ms   🟢 Online       │ │
│  │    US East (Virginia)              78ms   🟢 Online       │ │
│  │    Europe (London)                142ms   🟢 Online       │ │
│  │    Europe (Frankfurt)             156ms   🟢 Online       │ │
│  │    Asia (Singapore)               210ms   🟢 Online       │ │
│  │    Asia (Tokyo)                   185ms   🟢 Online       │ │
│  │    Oceania (Sydney)               165ms   🟢 Online       │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ⭐ = Recommended (lowest ping)                                │
│                                                                 │
│  [ ] Auto-select best region                                   │
│                                                                 │
│                    [Apply]  [Cancel]                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Network Monitoring

### Continuous Monitoring

```typescript
class NetworkMonitor {
  private metrics: NetworkMetrics;
  private history: NetworkMetrics[] = [];
  private listeners: NetworkListener[] = [];
  private updateInterval: number = 1000;

  start(): void {
    setInterval(() => this.update(), this.updateInterval);
  }

  private async update(): Promise<void> {
    // Measure current metrics
    const newMetrics = await this.measureMetrics();

    // Detect changes
    const qualityChanged = this.detectQualityChange(newMetrics);
    const disconnected = this.detectDisconnection(newMetrics);

    // Update state
    this.metrics = newMetrics;
    this.history.push(newMetrics);
    this.trimHistory();

    // Notify listeners
    this.notifyUpdate(newMetrics);

    if (qualityChanged) {
      this.notifyQualityChange(newMetrics.qualityLevel);
    }

    if (disconnected) {
      this.notifyDisconnection();
    }
  }

  private async measureMetrics(): Promise<NetworkMetrics> {
    // Ping measurement
    const pingStart = performance.now();
    await sendPing();
    const ping = performance.now() - pingStart;

    // Get packet stats from network layer
    const packetStats = getPacketStatistics();

    // Calculate derived metrics
    const pingHistory = this.history.slice(-10).map(m => m.ping);
    const pingAvg = pingHistory.reduce((a, b) => a + b, 0) / pingHistory.length;
    const pingJitter = calculateJitter(pingHistory);

    return {
      ping: Math.round(ping),
      pingAvg: Math.round(pingAvg),
      pingJitter: Math.round(pingJitter),
      pingMin: Math.min(...pingHistory, ping),
      pingMax: Math.max(...pingHistory, ping),
      packetLoss: packetStats.lossRate,
      packetsReceived: packetStats.received,
      packetsLost: packetStats.lost,
      downloadRate: packetStats.downloadRate,
      uploadRate: packetStats.uploadRate,
      connectionType: getConnectionType(),
      signalStrength: getSignalStrength(),
      serverRegion: getCurrentRegion(),
      serverPing: ping,
      qualityScore: calculateQualityScore({ ping, pingJitter, packetLoss: packetStats.lossRate }),
      qualityLevel: getQualityLevel(calculateQualityScore({ ping, pingJitter, packetLoss: packetStats.lossRate }))
    };
  }

  private detectDisconnection(metrics: NetworkMetrics): boolean {
    // No response for 5+ seconds
    return metrics.ping > 5000 || metrics.packetLoss > 50;
  }

  private detectQualityChange(newMetrics: NetworkMetrics): boolean {
    if (!this.metrics) return true;
    return this.metrics.qualityLevel !== newMetrics.qualityLevel;
  }
}
```

### Ping Graph (Debug Mode)

```
┌─────────────────────────────────────────────────────────────────┐
│  PING HISTORY (Last 60 seconds)                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  200ms ┤                                                        │
│        │                              ╭╮                        │
│  150ms ┤                         ╭────╯╰╮                       │
│        │                    ╭────╯      ╰╮                      │
│  100ms ┤              ╭─────╯            ╰╮                     │
│        │         ╭────╯                   ╰────╮                │
│   50ms ┤ ────────╯                             ╰───────────     │
│        │                                                        │
│    0ms ┼────────────────────────────────────────────────────    │
│        0s                    30s                    60s         │
│                                                                 │
│  Current: 45ms | Avg: 52ms | Jitter: 8ms                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Settings UI

### Network Settings

```
┌─────────────────────────────────────────────────────────────────┐
│  NETWORK SETTINGS                                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DISPLAY                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Show Network Indicator      [✓]                         │   │
│  │ Indicator Style            [Minimal ▼]                  │   │
│  │ Show Ping Number           [ ]                          │   │
│  │ Indicator Position         [Top Right ▼]                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  WARNINGS                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Show Connection Warnings    [✓]                         │   │
│  │ Auto-expand on Issues      [✓]                         │   │
│  │ Warning Sound              [✓]                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  SERVER                                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Server Region              [Auto (Best Ping) ▼]         │   │
│  │ Current: US West           45ms                         │   │
│  │ [Change Region]                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  DEBUG                                                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Show Detailed Stats        [ ]                          │   │
│  │ Show Ping Graph            [ ]                          │   │
│  │ Log Network Events         [ ]                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Analytics Events

```typescript
// Network quality tracking
analytics.track('network_quality_change', {
  previousLevel: QualityLevel,
  newLevel: QualityLevel,
  ping: number,
  packetLoss: number,
  connectionType: string
});

// Disconnection events
analytics.track('disconnection', {
  matchId: string,
  duration: number,
  reconnected: boolean,
  attempts: number,
  reason: string
});

// Region selection
analytics.track('region_selected', {
  region: string,
  ping: number,
  wasRecommended: boolean,
  previousRegion: string
});

// Network issues during match
analytics.track('network_issue_during_match', {
  matchId: string,
  issueType: string,
  duration: number,
  qualityLevel: string
});
```

## Testing Requirements

### Functional Tests

- [ ] Network indicator displays correctly
- [ ] Quality levels calculated accurately
- [ ] Warnings appear at correct thresholds
- [ ] Reconnection flow works correctly
- [ ] Region selection updates ping
- [ ] Settings persist correctly

### Network Simulation Tests

- [ ] High ping handling (100-500ms)
- [ ] Packet loss handling (1-20%)
- [ ] Complete disconnection
- [ ] Partial packet loss
- [ ] Network type switching (WiFi to cellular)

### Edge Cases

- [ ] Very unstable connection (rapid changes)
- [ ] Server maintenance during match
- [ ] Grace period expiration
- [ ] Multiple simultaneous issues

## Success Metrics

| Metric | Target |
|--------|--------|
| Connection issue reports | < 5% of sessions |
| Successful reconnections | > 80% |
| Average reconnection time | < 10 seconds |
| Players using region selection | 20%+ |

## Dependencies

- **BACK-003**: Real-time Networking
- **UX-015**: Settings Menu
- **GAME-005**: Match State Management
