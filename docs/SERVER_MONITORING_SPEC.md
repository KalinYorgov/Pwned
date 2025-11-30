# Server Monitoring Specification

## Document Information
- **Task ID:** BACK-020
- **Priority:** P0
- **Complexity:** M (Medium)
- **Dependencies:** BACK-004 (Dedicated Game Servers)
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the comprehensive monitoring system for Plunderstorm Mobile's server infrastructure. The system provides real-time visibility into server health, performance metrics, error tracking, and operational insights to ensure high availability and rapid incident response.

### 1.2 Scope
- Game server health monitoring
- Infrastructure metrics collection
- Log aggregation and search
- Alerting and incident management
- Performance dashboards
- Capacity planning tools
- On-call procedures

### 1.3 Goals
- Detect issues before they impact players
- Reduce mean time to detection (MTTD) to < 1 minute
- Reduce mean time to resolution (MTTR) through actionable alerts
- Provide data-driven capacity planning
- Enable forensic analysis through comprehensive logging

---

## 2. Monitoring Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Monitoring Architecture                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                   │
│  │ Game Server  │  │ Game Server  │  │ Game Server  │   ...             │
│  │   Instance   │  │   Instance   │  │   Instance   │                   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                   │
│         │                  │                  │                          │
│         └──────────────────┼──────────────────┘                          │
│                            │                                             │
│                   ┌────────▼────────┐                                    │
│                   │  Metrics Agent  │  (StatsD / Telegraf / OTel)        │
│                   └────────┬────────┘                                    │
│                            │                                             │
│         ┌──────────────────┼──────────────────┐                          │
│         ▼                  ▼                  ▼                          │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐                    │
│  │   Metrics   │   │    Logs     │   │   Traces    │                    │
│  │   Storage   │   │   Storage   │   │   Storage   │                    │
│  │(Prometheus) │   │(Elasticsearch)│ │  (Jaeger)   │                    │
│  └──────┬──────┘   └──────┬──────┘   └──────┬──────┘                    │
│         │                  │                  │                          │
│         └──────────────────┼──────────────────┘                          │
│                            │                                             │
│                   ┌────────▼────────┐                                    │
│                   │    Grafana      │                                    │
│                   │   Dashboards    │                                    │
│                   └────────┬────────┘                                    │
│                            │                                             │
│                   ┌────────▼────────┐                                    │
│                   │  Alert Manager  │──────▶ PagerDuty / Slack / Email  │
│                   └─────────────────┘                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack

| Component | Primary Choice | Alternative | Purpose |
|-----------|---------------|-------------|---------|
| Metrics Collection | Prometheus | DataDog | Time-series metrics storage |
| Log Aggregation | Elasticsearch | CloudWatch Logs | Log storage and search |
| Visualization | Grafana | DataDog Dashboards | Unified dashboards |
| Alerting | Alertmanager | PagerDuty | Alert routing and escalation |
| APM/Tracing | Jaeger | Datadog APM | Distributed tracing |
| Uptime Monitoring | Pingdom | UptimeRobot | External health checks |

### 2.3 Agent Architecture

```
┌────────────────────────────────────────────────────────┐
│                    Game Server VM                       │
├────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐    ┌─────────────────┐            │
│  │  Game Server    │    │  Game Server    │            │
│  │  Process #1     │    │  Process #2     │            │
│  │  ┌───────────┐  │    │  ┌───────────┐  │            │
│  │  │ StatsD    │  │    │  │ StatsD    │  │            │
│  │  │ Client    │  │    │  │ Client    │  │            │
│  │  └─────┬─────┘  │    │  └─────┬─────┘  │            │
│  └────────┼────────┘    └────────┼────────┘            │
│           │                      │                      │
│           └──────────┬───────────┘                      │
│                      ▼                                  │
│           ┌──────────────────┐                          │
│           │  Telegraf Agent  │                          │
│           │  ┌────────────┐  │                          │
│           │  │ System     │  │                          │
│           │  │ Metrics    │  │                          │
│           │  └────────────┘  │                          │
│           │  ┌────────────┐  │                          │
│           │  │ Game       │  │                          │
│           │  │ Metrics    │  │                          │
│           │  └────────────┘  │                          │
│           └────────┬─────────┘                          │
│                    │                                    │
│           ┌────────▼─────────┐                          │
│           │  Filebeat Agent  │  (Log shipping)          │
│           └────────┬─────────┘                          │
│                    │                                    │
└────────────────────┼────────────────────────────────────┘
                     │
                     ▼ To Monitoring Infrastructure
```

---

## 3. Metrics Collection

### 3.1 System Metrics

#### 3.1.1 Host Metrics
```yaml
system_metrics:
  cpu:
    - cpu_usage_percent       # Overall CPU utilization
    - cpu_usage_user          # User-space CPU time
    - cpu_usage_system        # Kernel CPU time
    - cpu_usage_iowait        # I/O wait time
    - load_average_1m         # 1-minute load average
    - load_average_5m         # 5-minute load average
    - load_average_15m        # 15-minute load average

  memory:
    - mem_total_bytes         # Total memory
    - mem_used_bytes          # Used memory
    - mem_available_bytes     # Available memory
    - mem_cached_bytes        # Cached memory
    - mem_buffered_bytes      # Buffered memory
    - mem_usage_percent       # Memory utilization percentage
    - swap_used_bytes         # Swap usage

  disk:
    - disk_total_bytes        # Total disk space
    - disk_used_bytes         # Used disk space
    - disk_free_bytes         # Free disk space
    - disk_usage_percent      # Disk utilization
    - disk_read_bytes_sec     # Disk read throughput
    - disk_write_bytes_sec    # Disk write throughput
    - disk_iops_read          # Read IOPS
    - disk_iops_write         # Write IOPS

  network:
    - net_bytes_recv_sec      # Network receive rate
    - net_bytes_sent_sec      # Network send rate
    - net_packets_recv_sec    # Packets received per second
    - net_packets_sent_sec    # Packets sent per second
    - net_errors_recv         # Receive errors
    - net_errors_sent         # Send errors
    - net_drop_recv           # Dropped incoming packets
    - net_drop_sent           # Dropped outgoing packets
    - tcp_connections_active  # Active TCP connections
    - tcp_connections_estab   # Established connections
```

#### 3.1.2 Process Metrics
```yaml
process_metrics:
  game_server:
    - process_cpu_percent     # CPU usage per process
    - process_memory_bytes    # Memory per process
    - process_memory_percent  # Memory percentage
    - process_threads         # Thread count
    - process_open_fds        # Open file descriptors
    - process_uptime_seconds  # Process uptime
```

### 3.2 Game Server Metrics

#### 3.2.1 Match Metrics
```yaml
match_metrics:
  lifecycle:
    - matches_created_total          # Total matches created
    - matches_started_total          # Total matches started
    - matches_completed_total        # Total matches completed
    - matches_failed_total           # Total match failures
    - matches_active_current         # Currently active matches
    - matches_waiting_current        # Matches waiting to start

  timing:
    - match_duration_seconds         # Match duration histogram
    - match_warmup_duration_seconds  # Warmup phase duration
    - match_active_duration_seconds  # Active gameplay duration

  players:
    - match_players_joined_total     # Players joining matches
    - match_players_dropped_total    # Players disconnecting
    - match_players_current          # Current players in match
    - match_players_peak             # Peak players in match
    - match_bots_current             # Current bots in match

  labels:
    - server_id
    - region
    - game_mode
    - match_id
```

#### 3.2.2 Performance Metrics
```yaml
performance_metrics:
  tick_rate:
    - server_tick_rate_hz            # Actual tick rate
    - server_tick_duration_ms        # Time per tick
    - server_tick_overrun_total      # Ticks that took too long
    - server_tick_utilization        # % of tick budget used

  simulation:
    - physics_update_duration_ms     # Physics calculation time
    - ai_update_duration_ms          # NPC AI processing time
    - network_update_duration_ms     # Network sync time
    - game_logic_duration_ms         # Game rules processing

  memory:
    - game_entities_active           # Active game entities
    - game_objects_pooled            # Pooled objects available
    - game_memory_allocated_mb       # Game-specific allocations

  network:
    - server_bandwidth_in_kbps       # Incoming bandwidth
    - server_bandwidth_out_kbps      # Outgoing bandwidth
    - server_packets_in_sec          # Incoming packet rate
    - server_packets_out_sec         # Outgoing packet rate
```

#### 3.2.3 Player Experience Metrics
```yaml
player_metrics:
  latency:
    - player_rtt_ms                  # Round-trip time per player
    - player_rtt_p50_ms              # 50th percentile RTT
    - player_rtt_p95_ms              # 95th percentile RTT
    - player_rtt_p99_ms              # 99th percentile RTT

  packet_loss:
    - player_packet_loss_percent     # Packet loss rate
    - player_packets_resent          # Resent packets

  connection:
    - player_connections_total       # Total connections
    - player_disconnections_total    # Total disconnections
    - player_reconnections_total     # Reconnection attempts
    - player_connection_duration_s   # Connection duration

  quality:
    - player_prediction_errors       # Client prediction misses
    - player_reconciliation_count    # State reconciliations
    - player_interpolation_ratio     # Interpolation smoothness
```

### 3.3 Infrastructure Metrics

#### 3.3.1 Orchestration Metrics
```yaml
orchestration_metrics:
  fleet:
    - fleet_servers_total            # Total server capacity
    - fleet_servers_available        # Available servers
    - fleet_servers_allocated        # Servers running matches
    - fleet_servers_draining         # Servers being drained
    - fleet_servers_unhealthy        # Unhealthy servers

  scaling:
    - scaling_events_total           # Auto-scaling events
    - scaling_scale_up_total         # Scale-up events
    - scaling_scale_down_total       # Scale-down events
    - scaling_target_capacity        # Target server count

  allocation:
    - allocation_requests_total      # Server allocation requests
    - allocation_success_total       # Successful allocations
    - allocation_failure_total       # Failed allocations
    - allocation_latency_ms          # Time to allocate server

  regional:
    - region_capacity                # Per-region capacity
    - region_utilization             # Per-region utilization

  labels:
    - region
    - instance_type
    - cloud_provider
```

#### 3.3.2 Matchmaking Metrics
```yaml
matchmaking_metrics:
  queue:
    - queue_players_waiting          # Players in queue
    - queue_time_seconds             # Time spent in queue
    - queue_time_p50_seconds         # 50th percentile queue time
    - queue_time_p95_seconds         # 95th percentile queue time

  matches:
    - matches_created_total          # Matches formed
    - matches_fill_rate              # Average fill rate
    - matches_with_bots_total        # Matches requiring bots

  labels:
    - region
    - game_mode
    - skill_bracket
```

### 3.4 Metric Collection Implementation

```go
// metrics/collector.go
package metrics

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
)

// GameServerMetrics holds all game server metrics
type GameServerMetrics struct {
    // Match metrics
    MatchesActive      prometheus.Gauge
    MatchesCreated     prometheus.Counter
    MatchesCompleted   *prometheus.CounterVec
    MatchDuration      prometheus.Histogram

    // Player metrics
    PlayersActive      prometheus.Gauge
    PlayerRTT          prometheus.Histogram
    PlayerPacketLoss   prometheus.Histogram

    // Performance metrics
    TickDuration       prometheus.Histogram
    TickOverruns       prometheus.Counter
    EntitiesActive     prometheus.Gauge

    // Network metrics
    BandwidthIn        prometheus.Gauge
    BandwidthOut       prometheus.Gauge
    PacketsIn          prometheus.Counter
    PacketsOut         prometheus.Counter
}

// NewGameServerMetrics creates and registers game server metrics
func NewGameServerMetrics(registry prometheus.Registerer) *GameServerMetrics {
    return &GameServerMetrics{
        MatchesActive: promauto.With(registry).NewGauge(prometheus.GaugeOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "matches_active",
            Help:      "Number of currently active matches",
        }),

        MatchesCreated: promauto.With(registry).NewCounter(prometheus.CounterOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "matches_created_total",
            Help:      "Total number of matches created",
        }),

        MatchesCompleted: promauto.With(registry).NewCounterVec(prometheus.CounterOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "matches_completed_total",
            Help:      "Total number of matches completed",
        }, []string{"outcome", "game_mode"}),

        MatchDuration: promauto.With(registry).NewHistogram(prometheus.HistogramOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "match_duration_seconds",
            Help:      "Match duration in seconds",
            Buckets:   []float64{60, 120, 180, 240, 300, 360, 420, 480, 540, 600, 900, 1200},
        }),

        PlayersActive: promauto.With(registry).NewGauge(prometheus.GaugeOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "players_active",
            Help:      "Number of active players across all matches",
        }),

        PlayerRTT: promauto.With(registry).NewHistogram(prometheus.HistogramOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "player_rtt_milliseconds",
            Help:      "Player round-trip time in milliseconds",
            Buckets:   []float64{10, 25, 50, 75, 100, 150, 200, 300, 500, 1000},
        }),

        PlayerPacketLoss: promauto.With(registry).NewHistogram(prometheus.HistogramOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "player_packet_loss_percent",
            Help:      "Player packet loss percentage",
            Buckets:   []float64{0, 0.5, 1, 2, 5, 10, 20, 50},
        }),

        TickDuration: promauto.With(registry).NewHistogram(prometheus.HistogramOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "tick_duration_milliseconds",
            Help:      "Duration of game ticks in milliseconds",
            Buckets:   []float64{10, 20, 30, 40, 50, 60, 80, 100, 150, 200},
        }),

        TickOverruns: promauto.With(registry).NewCounter(prometheus.CounterOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "tick_overruns_total",
            Help:      "Number of ticks that exceeded the target duration",
        }),

        EntitiesActive: promauto.With(registry).NewGauge(prometheus.GaugeOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "entities_active",
            Help:      "Number of active game entities",
        }),

        BandwidthIn: promauto.With(registry).NewGauge(prometheus.GaugeOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "bandwidth_in_kbps",
            Help:      "Incoming bandwidth in kilobits per second",
        }),

        BandwidthOut: promauto.With(registry).NewGauge(prometheus.GaugeOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "bandwidth_out_kbps",
            Help:      "Outgoing bandwidth in kilobits per second",
        }),

        PacketsIn: promauto.With(registry).NewCounter(prometheus.CounterOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "packets_in_total",
            Help:      "Total incoming packets",
        }),

        PacketsOut: promauto.With(registry).NewCounter(prometheus.CounterOpts{
            Namespace: "plunderstorm",
            Subsystem: "game_server",
            Name:      "packets_out_total",
            Help:      "Total outgoing packets",
        }),
    }
}
```

---

## 4. Log Aggregation

### 4.1 Log Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Log Aggregation Pipeline                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐                                                        │
│  │ Game Server  │──┐                                                     │
│  │    Logs      │  │                                                     │
│  └──────────────┘  │     ┌─────────────┐     ┌─────────────┐            │
│  ┌──────────────┐  │     │             │     │             │            │
│  │ API Server   │──┼────▶│   Filebeat  │────▶│    Kafka    │            │
│  │    Logs      │  │     │             │     │   (Buffer)  │            │
│  └──────────────┘  │     └─────────────┘     └──────┬──────┘            │
│  ┌──────────────┐  │                                │                    │
│  │ Orchestrator │──┘                                ▼                    │
│  │    Logs      │                          ┌─────────────┐               │
│  └──────────────┘                          │  Logstash   │               │
│                                            │  (Process)  │               │
│                                            └──────┬──────┘               │
│                                                   │                      │
│                                                   ▼                      │
│                               ┌────────────────────────────────────┐     │
│                               │          Elasticsearch             │     │
│                               │  ┌────────┐ ┌────────┐ ┌────────┐ │     │
│                               │  │ Index  │ │ Index  │ │ Index  │ │     │
│                               │  │ Game   │ │ API    │ │ Error  │ │     │
│                               │  └────────┘ └────────┘ └────────┘ │     │
│                               └────────────────┬───────────────────┘     │
│                                                │                         │
│                                                ▼                         │
│                                        ┌─────────────┐                   │
│                                        │   Kibana    │                   │
│                                        │ (Visualize) │                   │
│                                        └─────────────┘                   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Log Format Standards

#### 4.2.1 Structured Log Format
```json
{
  "timestamp": "2025-11-30T12:34:56.789Z",
  "level": "INFO",
  "service": "game-server",
  "instance_id": "gs-na-east-001",
  "region": "na-east",
  "version": "1.2.3",
  "trace_id": "abc123def456",
  "span_id": "span789",
  "message": "Match started",
  "context": {
    "match_id": "match-uuid-123",
    "game_mode": "solo",
    "player_count": 60,
    "map": "treasure_island"
  }
}
```

#### 4.2.2 Log Levels
```yaml
log_levels:
  TRACE:
    description: "Detailed debugging information"
    retention: 1 day
    enabled_in: development only

  DEBUG:
    description: "Diagnostic information"
    retention: 3 days
    enabled_in: development, staging

  INFO:
    description: "Normal operational events"
    retention: 14 days
    enabled_in: all environments

  WARN:
    description: "Potential issues that don't affect operation"
    retention: 30 days
    enabled_in: all environments

  ERROR:
    description: "Errors that affect specific operations"
    retention: 90 days
    enabled_in: all environments

  FATAL:
    description: "Critical errors that crash the service"
    retention: 1 year
    enabled_in: all environments
```

### 4.3 Log Categories

#### 4.3.1 Game Server Logs
```yaml
game_server_logs:
  match_lifecycle:
    - match_created
    - match_started
    - match_phase_changed
    - match_completed
    - match_failed

  player_events:
    - player_connected
    - player_disconnected
    - player_reconnected
    - player_eliminated
    - player_won

  gameplay_events:
    - ability_used
    - damage_dealt
    - loot_collected
    - zone_damage
    - npc_spawned
    - npc_killed

  performance_events:
    - tick_overrun
    - memory_warning
    - network_congestion
    - entity_limit_warning
```

#### 4.3.2 Error Logs
```yaml
error_logs:
  categories:
    - validation_error      # Input validation failures
    - network_error         # Network communication issues
    - state_error          # Game state inconsistencies
    - resource_error       # Resource allocation failures
    - timeout_error        # Operation timeouts
    - crash_error          # Process crashes

  required_fields:
    - error_code           # Unique error identifier
    - error_message        # Human-readable message
    - stack_trace          # Full stack trace
    - context              # Relevant game/player context
    - severity             # Impact severity
```

### 4.4 Log Processing Pipeline

```yaml
# logstash/pipeline/game-server.conf
input {
  kafka {
    bootstrap_servers => "kafka:9092"
    topics => ["game-server-logs"]
    group_id => "logstash-game-server"
    codec => json
  }
}

filter {
  # Parse timestamp
  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }

  # Add geo information for region
  if [region] {
    mutate {
      add_field => { "geo_region" => "%{region}" }
    }
  }

  # Parse stack traces
  if [level] == "ERROR" or [level] == "FATAL" {
    grok {
      match => {
        "stack_trace" => "%{GREEDYDATA:error_class}: %{GREEDYDATA:error_message}"
      }
    }
  }

  # Enrich with server metadata
  if [instance_id] {
    elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "server-metadata"
      query => "instance_id:%{[instance_id]}"
      fields => {
        "instance_type" => "instance_type"
        "cloud_provider" => "cloud_provider"
      }
    }
  }

  # Calculate latency buckets
  if [player_rtt] {
    ruby {
      code => '
        rtt = event.get("player_rtt").to_f
        bucket = case rtt
          when 0..50 then "good"
          when 50..100 then "acceptable"
          when 100..200 then "degraded"
          else "poor"
        end
        event.set("latency_bucket", bucket)
      '
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "game-server-logs-%{+YYYY.MM.dd}"
    template_name => "game-server-logs"
  }

  # Send errors to dedicated index
  if [level] == "ERROR" or [level] == "FATAL" {
    elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "game-server-errors-%{+YYYY.MM.dd}"
    }
  }
}
```

### 4.5 Log Search Queries

```json
// Find all errors for a specific match
{
  "query": {
    "bool": {
      "must": [
        { "term": { "context.match_id": "match-uuid-123" } },
        { "terms": { "level": ["ERROR", "FATAL"] } }
      ]
    }
  },
  "sort": [{ "@timestamp": "asc" }]
}

// Find players with high packet loss
{
  "query": {
    "bool": {
      "must": [
        { "range": { "player_packet_loss": { "gte": 5 } } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  },
  "aggs": {
    "by_region": {
      "terms": { "field": "region" }
    }
  }
}

// Find tick overruns in last 24 hours
{
  "query": {
    "bool": {
      "must": [
        { "term": { "event_type": "tick_overrun" } },
        { "range": { "@timestamp": { "gte": "now-24h" } } }
      ]
    }
  },
  "aggs": {
    "by_instance": {
      "terms": { "field": "instance_id" },
      "aggs": {
        "avg_overrun_ms": { "avg": { "field": "overrun_duration_ms" } }
      }
    }
  }
}
```

---

## 5. Alerting System

### 5.1 Alert Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Alerting Architecture                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────┐        │
│  │                      Prometheus                              │        │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │        │
│  │  │   Alert     │  │   Alert     │  │   Alert     │          │        │
│  │  │   Rule 1    │  │   Rule 2    │  │   Rule N    │          │        │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │        │
│  └─────────┼────────────────┼────────────────┼──────────────────┘        │
│            │                │                │                           │
│            └────────────────┼────────────────┘                           │
│                             ▼                                            │
│                    ┌─────────────────┐                                   │
│                    │  Alertmanager   │                                   │
│                    │                 │                                   │
│                    │ ┌─────────────┐ │                                   │
│                    │ │ Grouping    │ │                                   │
│                    │ └─────────────┘ │                                   │
│                    │ ┌─────────────┐ │                                   │
│                    │ │ Inhibition  │ │                                   │
│                    │ └─────────────┘ │                                   │
│                    │ ┌─────────────┐ │                                   │
│                    │ │ Silencing   │ │                                   │
│                    │ └─────────────┘ │                                   │
│                    │ ┌─────────────┐ │                                   │
│                    │ │ Routing     │ │                                   │
│                    │ └─────────────┘ │                                   │
│                    └────────┬────────┘                                   │
│                             │                                            │
│            ┌────────────────┼────────────────┐                           │
│            ▼                ▼                ▼                           │
│     ┌───────────┐   ┌───────────┐   ┌───────────┐                       │
│     │ PagerDuty │   │   Slack   │   │   Email   │                       │
│     │ (P1/P2)   │   │ (P2/P3)   │   │ (All)     │                       │
│     └───────────┘   └───────────┘   └───────────┘                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Alert Priority Levels

```yaml
alert_priorities:
  P1_Critical:
    description: "Service outage or severe degradation"
    response_time: "5 minutes"
    notification:
      - PagerDuty (24/7 on-call)
      - Slack #incidents
      - Email to leads
    examples:
      - All game servers down
      - Matchmaking completely failing
      - Database unavailable
      - > 50% error rate

  P2_High:
    description: "Significant impact on player experience"
    response_time: "30 minutes"
    notification:
      - PagerDuty (business hours)
      - Slack #alerts
      - Email
    examples:
      - Single region degraded
      - > 10% match failures
      - High latency (> 200ms p95)
      - Capacity < 20% headroom

  P3_Medium:
    description: "Potential issues requiring attention"
    response_time: "4 hours"
    notification:
      - Slack #alerts
      - Email
    examples:
      - Elevated error rates (> 1%)
      - Single server unhealthy
      - Approaching resource limits

  P4_Low:
    description: "Informational or maintenance"
    response_time: "Next business day"
    notification:
      - Slack #monitoring
      - Email digest
    examples:
      - Scheduled maintenance
      - Non-critical warnings
      - Trend anomalies
```

### 5.3 Alert Rules

```yaml
# prometheus/alerts/game-server.yml
groups:
  - name: game_server_critical
    rules:
      - alert: AllServersDown
        expr: count(up{job="game-server"}) == 0
        for: 1m
        labels:
          severity: P1
        annotations:
          summary: "All game servers are down"
          description: "No game servers are responding to health checks"

      - alert: HighMatchFailureRate
        expr: |
          rate(plunderstorm_game_server_matches_completed_total{outcome="failed"}[5m])
          / rate(plunderstorm_game_server_matches_completed_total[5m]) > 0.5
        for: 5m
        labels:
          severity: P1
        annotations:
          summary: "High match failure rate: {{ $value | humanizePercentage }}"
          description: "More than 50% of matches are failing"

      - alert: MatchmakingStalled
        expr: plunderstorm_matchmaking_queue_players_waiting > 100 and increase(plunderstorm_matchmaking_matches_created_total[5m]) == 0
        for: 5m
        labels:
          severity: P1
        annotations:
          summary: "Matchmaking is stalled"
          description: "{{ $value }} players waiting but no matches being created"

  - name: game_server_high
    rules:
      - alert: RegionDegraded
        expr: |
          count(up{job="game-server"} == 1) by (region)
          / count(up{job="game-server"}) by (region) < 0.5
        for: 5m
        labels:
          severity: P2
        annotations:
          summary: "Region {{ $labels.region }} is degraded"
          description: "Less than 50% of servers healthy in region"

      - alert: HighPlayerLatency
        expr: histogram_quantile(0.95, rate(plunderstorm_game_server_player_rtt_milliseconds_bucket[5m])) > 200
        for: 10m
        labels:
          severity: P2
        annotations:
          summary: "High player latency: {{ $value }}ms p95"
          description: "95th percentile latency exceeds 200ms"

      - alert: LowCapacity
        expr: |
          plunderstorm_orchestration_fleet_servers_available
          / plunderstorm_orchestration_fleet_servers_total < 0.2
        for: 10m
        labels:
          severity: P2
        annotations:
          summary: "Low server capacity: {{ $value | humanizePercentage }} available"
          description: "Less than 20% server capacity remaining"

      - alert: TickRateDegraded
        expr: avg(plunderstorm_game_server_tick_rate_hz) < 18
        for: 5m
        labels:
          severity: P2
        annotations:
          summary: "Server tick rate degraded: {{ $value }}Hz"
          description: "Average tick rate below 18Hz (target: 20Hz)"

  - name: game_server_medium
    rules:
      - alert: ElevatedErrorRate
        expr: rate(plunderstorm_game_server_errors_total[5m]) > 10
        for: 10m
        labels:
          severity: P3
        annotations:
          summary: "Elevated error rate: {{ $value }}/s"
          description: "Error rate exceeds 10 per second"

      - alert: ServerUnhealthy
        expr: up{job="game-server"} == 0
        for: 5m
        labels:
          severity: P3
        annotations:
          summary: "Server {{ $labels.instance_id }} is unhealthy"
          description: "Game server not responding to health checks"

      - alert: HighPacketLoss
        expr: histogram_quantile(0.95, rate(plunderstorm_game_server_player_packet_loss_percent_bucket[5m])) > 5
        for: 15m
        labels:
          severity: P3
        annotations:
          summary: "High packet loss: {{ $value }}% p95"
          description: "95th percentile packet loss exceeds 5%"

      - alert: HighMemoryUsage
        expr: plunderstorm_game_server_memory_usage_percent > 85
        for: 10m
        labels:
          severity: P3
        annotations:
          summary: "High memory usage: {{ $value }}%"
          description: "Server memory usage exceeds 85%"

  - name: game_server_low
    rules:
      - alert: ApproachingCapacity
        expr: |
          plunderstorm_orchestration_fleet_servers_available
          / plunderstorm_orchestration_fleet_servers_total < 0.4
        for: 30m
        labels:
          severity: P4
        annotations:
          summary: "Approaching capacity limit: {{ $value | humanizePercentage }} available"
          description: "Server capacity below 40%"

      - alert: LongQueueTimes
        expr: histogram_quantile(0.95, rate(plunderstorm_matchmaking_queue_time_seconds_bucket[5m])) > 60
        for: 15m
        labels:
          severity: P4
        annotations:
          summary: "Long queue times: {{ $value }}s p95"
          description: "95th percentile queue time exceeds 60 seconds"
```

### 5.4 Alertmanager Configuration

```yaml
# alertmanager/config.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/xxx'
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

route:
  receiver: 'default'
  group_by: ['alertname', 'region', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    # P1 Critical - immediate PagerDuty
    - match:
        severity: P1
      receiver: 'pagerduty-critical'
      continue: true

    # P2 High - PagerDuty during business hours, Slack always
    - match:
        severity: P2
      receiver: 'pagerduty-high'
      continue: true

    # All alerts to Slack
    - match_re:
        severity: P[1-3]
      receiver: 'slack-alerts'

    # Low priority to monitoring channel
    - match:
        severity: P4
      receiver: 'slack-monitoring'

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#monitoring'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<P1_SERVICE_KEY>'
        severity: critical
        description: '{{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ template "pagerduty.default.instances" .Alerts.Firing }}'

  - name: 'pagerduty-high'
    pagerduty_configs:
      - service_key: '<P2_SERVICE_KEY>'
        severity: error
        description: '{{ .CommonAnnotations.summary }}'

  - name: 'slack-alerts'
    slack_configs:
      - channel: '#alerts'
        send_resolved: true
        title: '{{ .Status | toUpper }}: {{ .CommonAnnotations.summary }}'
        text: '{{ .CommonAnnotations.description }}'
        color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'

  - name: 'slack-monitoring'
    slack_configs:
      - channel: '#monitoring'
        send_resolved: true

inhibit_rules:
  # Don't alert on individual servers if entire region is down
  - source_match:
      alertname: RegionDegraded
    target_match:
      alertname: ServerUnhealthy
    equal: ['region']

  # Don't alert on capacity if servers are unhealthy
  - source_match:
      alertname: AllServersDown
    target_match_re:
      alertname: (LowCapacity|ApproachingCapacity)
```

---

## 6. Dashboards

### 6.1 Dashboard Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Dashboard Organization                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Level 1: Executive Overview                                             │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │  Global Health │ Player Count │ Match Status │ Error Rate │ SLA  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                    │                                     │
│                                    ▼                                     │
│  Level 2: Service Dashboards                                             │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐                  │
│  │  Game Server  │ │ Matchmaking   │ │ Orchestration │                  │
│  │  Dashboard    │ │ Dashboard     │ │ Dashboard     │                  │
│  └───────┬───────┘ └───────┬───────┘ └───────┬───────┘                  │
│          │                 │                 │                           │
│          ▼                 ▼                 ▼                           │
│  Level 3: Detailed Dashboards                                            │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │
│  │ Per-Server │ │ Per-Region │ │ Per-Match  │ │ Performance│           │
│  │ Metrics    │ │ Breakdown  │ │ Analysis   │ │ Deep Dive  │           │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Executive Overview Dashboard

```json
{
  "dashboard": {
    "title": "Plunderstorm - Executive Overview",
    "rows": [
      {
        "title": "Key Metrics",
        "panels": [
          {
            "title": "Global Health Score",
            "type": "gauge",
            "targets": [
              {
                "expr": "100 * (1 - (sum(rate(plunderstorm_errors_total[5m])) / sum(rate(plunderstorm_requests_total[5m]))))"
              }
            ],
            "thresholds": [
              { "value": 99, "color": "green" },
              { "value": 95, "color": "yellow" },
              { "value": 0, "color": "red" }
            ]
          },
          {
            "title": "Active Players",
            "type": "stat",
            "targets": [
              { "expr": "sum(plunderstorm_game_server_players_active)" }
            ]
          },
          {
            "title": "Active Matches",
            "type": "stat",
            "targets": [
              { "expr": "sum(plunderstorm_game_server_matches_active)" }
            ]
          },
          {
            "title": "Error Rate",
            "type": "stat",
            "targets": [
              { "expr": "sum(rate(plunderstorm_errors_total[5m])) / sum(rate(plunderstorm_requests_total[5m])) * 100" }
            ],
            "unit": "percent"
          }
        ]
      },
      {
        "title": "Trends",
        "panels": [
          {
            "title": "Players Over Time",
            "type": "graph",
            "targets": [
              { "expr": "sum(plunderstorm_game_server_players_active)", "legendFormat": "Active Players" }
            ]
          },
          {
            "title": "Matches Per Minute",
            "type": "graph",
            "targets": [
              { "expr": "sum(rate(plunderstorm_game_server_matches_completed_total[5m])) * 60", "legendFormat": "Completed" },
              { "expr": "sum(rate(plunderstorm_game_server_matches_failed_total[5m])) * 60", "legendFormat": "Failed" }
            ]
          }
        ]
      },
      {
        "title": "Regional Status",
        "panels": [
          {
            "title": "Players by Region",
            "type": "piechart",
            "targets": [
              { "expr": "sum by (region) (plunderstorm_game_server_players_active)" }
            ]
          },
          {
            "title": "Region Health",
            "type": "table",
            "targets": [
              {
                "expr": "count by (region) (up{job='game-server'} == 1) / count by (region) (up{job='game-server'}) * 100",
                "legendFormat": "{{ region }}"
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### 6.3 Game Server Dashboard

```json
{
  "dashboard": {
    "title": "Game Server Performance",
    "templating": {
      "list": [
        { "name": "region", "type": "query", "query": "label_values(up{job='game-server'}, region)" },
        { "name": "instance", "type": "query", "query": "label_values(up{job='game-server', region='$region'}, instance_id)" }
      ]
    },
    "rows": [
      {
        "title": "Server Health",
        "panels": [
          {
            "title": "Server Status",
            "type": "stat",
            "targets": [
              { "expr": "count(up{job='game-server', region='$region'} == 1)", "legendFormat": "Healthy" },
              { "expr": "count(up{job='game-server', region='$region'} == 0)", "legendFormat": "Unhealthy" }
            ]
          },
          {
            "title": "CPU Usage",
            "type": "graph",
            "targets": [
              { "expr": "avg by (instance_id) (system_cpu_usage_percent{job='game-server', region='$region'})", "legendFormat": "{{ instance_id }}" }
            ]
          },
          {
            "title": "Memory Usage",
            "type": "graph",
            "targets": [
              { "expr": "avg by (instance_id) (system_mem_usage_percent{job='game-server', region='$region'})", "legendFormat": "{{ instance_id }}" }
            ]
          }
        ]
      },
      {
        "title": "Game Performance",
        "panels": [
          {
            "title": "Tick Rate",
            "type": "graph",
            "targets": [
              { "expr": "avg(plunderstorm_game_server_tick_rate_hz{region='$region'})", "legendFormat": "Avg Tick Rate" }
            ],
            "yAxis": { "min": 0, "max": 25 },
            "thresholds": [
              { "value": 20, "color": "green" },
              { "value": 15, "color": "red" }
            ]
          },
          {
            "title": "Tick Duration Distribution",
            "type": "heatmap",
            "targets": [
              { "expr": "rate(plunderstorm_game_server_tick_duration_milliseconds_bucket{region='$region'}[5m])" }
            ]
          },
          {
            "title": "Active Entities",
            "type": "graph",
            "targets": [
              { "expr": "sum(plunderstorm_game_server_entities_active{region='$region'})", "legendFormat": "Total Entities" }
            ]
          }
        ]
      },
      {
        "title": "Network",
        "panels": [
          {
            "title": "Bandwidth",
            "type": "graph",
            "targets": [
              { "expr": "sum(plunderstorm_game_server_bandwidth_in_kbps{region='$region'})", "legendFormat": "Inbound" },
              { "expr": "sum(plunderstorm_game_server_bandwidth_out_kbps{region='$region'})", "legendFormat": "Outbound" }
            ],
            "unit": "kbps"
          },
          {
            "title": "Player RTT Distribution",
            "type": "histogram",
            "targets": [
              { "expr": "plunderstorm_game_server_player_rtt_milliseconds{region='$region'}" }
            ]
          },
          {
            "title": "Packet Loss",
            "type": "graph",
            "targets": [
              { "expr": "histogram_quantile(0.95, rate(plunderstorm_game_server_player_packet_loss_percent_bucket{region='$region'}[5m]))", "legendFormat": "p95" },
              { "expr": "histogram_quantile(0.50, rate(plunderstorm_game_server_player_packet_loss_percent_bucket{region='$region'}[5m]))", "legendFormat": "p50" }
            ]
          }
        ]
      }
    ]
  }
}
```

### 6.4 Matchmaking Dashboard

```json
{
  "dashboard": {
    "title": "Matchmaking Analytics",
    "rows": [
      {
        "title": "Queue Status",
        "panels": [
          {
            "title": "Players in Queue",
            "type": "graph",
            "targets": [
              { "expr": "sum by (game_mode) (plunderstorm_matchmaking_queue_players_waiting)", "legendFormat": "{{ game_mode }}" }
            ]
          },
          {
            "title": "Queue Time (p95)",
            "type": "graph",
            "targets": [
              { "expr": "histogram_quantile(0.95, rate(plunderstorm_matchmaking_queue_time_seconds_bucket[5m]))", "legendFormat": "p95 Queue Time" }
            ],
            "unit": "s",
            "thresholds": [
              { "value": 30, "color": "green" },
              { "value": 60, "color": "red" }
            ]
          }
        ]
      },
      {
        "title": "Match Creation",
        "panels": [
          {
            "title": "Matches Created/min",
            "type": "graph",
            "targets": [
              { "expr": "sum(rate(plunderstorm_matchmaking_matches_created_total[5m])) * 60", "legendFormat": "Matches/min" }
            ]
          },
          {
            "title": "Average Fill Rate",
            "type": "stat",
            "targets": [
              { "expr": "avg(plunderstorm_matchmaking_matches_fill_rate) * 100" }
            ],
            "unit": "percent"
          },
          {
            "title": "Bot Fill Percentage",
            "type": "graph",
            "targets": [
              { "expr": "sum(rate(plunderstorm_matchmaking_matches_with_bots_total[5m])) / sum(rate(plunderstorm_matchmaking_matches_created_total[5m])) * 100", "legendFormat": "% with Bots" }
            ]
          }
        ]
      }
    ]
  }
}
```

---

## 7. Performance Baselines

### 7.1 Baseline Definitions

```yaml
performance_baselines:
  server_performance:
    tick_rate:
      target: 20 Hz
      warning: < 18 Hz
      critical: < 15 Hz

    tick_duration:
      target: < 40ms
      warning: > 45ms
      critical: > 50ms

    cpu_usage:
      target: < 70%
      warning: > 80%
      critical: > 90%

    memory_usage:
      target: < 75%
      warning: > 85%
      critical: > 95%

  player_experience:
    latency_p50:
      target: < 50ms
      warning: > 80ms
      critical: > 120ms

    latency_p95:
      target: < 100ms
      warning: > 150ms
      critical: > 200ms

    packet_loss_p95:
      target: < 1%
      warning: > 3%
      critical: > 5%

  matchmaking:
    queue_time_p50:
      target: < 15s
      warning: > 30s
      critical: > 60s

    queue_time_p95:
      target: < 30s
      warning: > 60s
      critical: > 120s

    match_fill_rate:
      target: > 90%
      warning: < 80%
      critical: < 60%

  reliability:
    match_success_rate:
      target: > 99%
      warning: < 98%
      critical: < 95%

    server_availability:
      target: > 99.9%
      warning: < 99.5%
      critical: < 99%

    error_rate:
      target: < 0.1%
      warning: > 0.5%
      critical: > 1%
```

### 7.2 Capacity Planning Queries

```yaml
# Prometheus queries for capacity planning
capacity_queries:
  peak_concurrent_players:
    query: "max_over_time(sum(plunderstorm_game_server_players_active)[7d])"
    purpose: "Determine peak player load for capacity planning"

  average_daily_players:
    query: "avg_over_time(sum(plunderstorm_game_server_players_active)[7d:1h])"
    purpose: "Calculate average player load"

  server_utilization_trend:
    query: |
      predict_linear(
        avg(plunderstorm_game_server_cpu_percent)[7d:1h],
        86400 * 30
      )
    purpose: "Predict CPU utilization 30 days ahead"

  matches_per_server_hour:
    query: |
      sum(increase(plunderstorm_game_server_matches_completed_total[1h]))
      / count(up{job="game-server"})
    purpose: "Calculate match throughput per server"

  player_to_server_ratio:
    query: |
      sum(plunderstorm_game_server_players_active)
      / count(up{job="game-server"} == 1)
    purpose: "Calculate players per active server"
```

### 7.3 Baseline Recording Rules

```yaml
# prometheus/rules/baselines.yml
groups:
  - name: baseline_recording
    rules:
      # 7-day moving averages
      - record: plunderstorm:player_count:avg7d
        expr: avg_over_time(sum(plunderstorm_game_server_players_active)[7d:1h])

      - record: plunderstorm:match_rate:avg7d
        expr: avg_over_time(sum(rate(plunderstorm_game_server_matches_completed_total[5m]))[7d:1h])

      - record: plunderstorm:error_rate:avg7d
        expr: avg_over_time(sum(rate(plunderstorm_errors_total[5m])) / sum(rate(plunderstorm_requests_total[5m]))[7d:1h])

      # Percentile baselines
      - record: plunderstorm:latency_p95:avg1h
        expr: histogram_quantile(0.95, sum(rate(plunderstorm_game_server_player_rtt_milliseconds_bucket[1h])) by (le))

      - record: plunderstorm:queue_time_p95:avg1h
        expr: histogram_quantile(0.95, sum(rate(plunderstorm_matchmaking_queue_time_seconds_bucket[1h])) by (le))
```

---

## 8. On-Call Procedures

### 8.1 On-Call Structure

```yaml
on_call_structure:
  rotation:
    schedule: weekly
    handoff: Monday 10:00 UTC
    overlap: 1 hour

  tiers:
    tier_1:
      description: "Primary on-call engineer"
      responsibilities:
        - First response to all P1/P2 alerts
        - Initial triage and investigation
        - Escalation if needed
      response_time:
        P1: 5 minutes
        P2: 30 minutes

    tier_2:
      description: "Secondary/specialist support"
      responsibilities:
        - Assist Tier 1 with complex issues
        - Take over if Tier 1 unavailable
        - Specialist knowledge (database, network, etc.)
      response_time:
        P1: 15 minutes
        P2: 1 hour

    tier_3:
      description: "Management escalation"
      responsibilities:
        - Customer communication
        - Resource allocation decisions
        - External vendor coordination
      response_time:
        P1: 30 minutes
```

### 8.2 Incident Response Playbooks

#### 8.2.1 Server Outage Playbook
```yaml
playbook: server_outage
trigger: AllServersDown or RegionDegraded alert
severity: P1

steps:
  1_assess:
    action: "Check monitoring dashboard for scope of outage"
    commands:
      - "kubectl get pods -n game-servers"
      - "aws ec2 describe-instances --filters 'Name=tag:service,Values=game-server'"
    duration: 2 min

  2_communicate:
    action: "Post status update to #incidents Slack channel"
    template: |
      :rotating_light: INVESTIGATING: Game server outage
      - Affected region(s): {{ regions }}
      - First alert: {{ alert_time }}
      - On-call: {{ oncall_name }}
      - Status page updated: [link]
    duration: 1 min

  3_diagnose:
    action: "Identify root cause"
    checks:
      - "Check recent deployments: any changes in last 1 hour?"
      - "Check cloud provider status pages"
      - "Review error logs for common patterns"
      - "Check network connectivity between services"
    commands:
      - "kubectl logs -l app=game-server --tail=100"
      - "journalctl -u game-server -n 100"
    duration: 5 min

  4_mitigate:
    scenarios:
      deployment_issue:
        action: "Rollback to previous version"
        command: "kubectl rollout undo deployment/game-server"

      resource_exhaustion:
        action: "Scale up servers"
        command: "kubectl scale deployment/game-server --replicas=+10"

      cloud_provider_issue:
        action: "Failover to alternate region"
        command: "terraform apply -var 'primary_region=us-west-2'"

      network_issue:
        action: "Restart network components"
        command: "kubectl rollout restart deployment/network-gateway"
    duration: 10 min

  5_verify:
    action: "Confirm service recovery"
    checks:
      - "Monitoring shows servers healthy"
      - "Players can connect to matches"
      - "Error rate returned to baseline"
    duration: 5 min

  6_communicate_resolution:
    action: "Update status and close incident"
    template: |
      :white_check_mark: RESOLVED: Game server outage
      - Duration: {{ duration }}
      - Root cause: {{ root_cause }}
      - Resolution: {{ resolution }}
      - Follow-up: {{ action_items }}

escalation:
  - after: 15 min
    action: "Page Tier 2 on-call"
  - after: 30 min
    action: "Page engineering lead"
  - after: 1 hour
    action: "Page VP Engineering"
```

#### 8.2.2 High Latency Playbook
```yaml
playbook: high_latency
trigger: HighPlayerLatency alert
severity: P2

steps:
  1_assess:
    action: "Determine scope and affected regions"
    commands:
      - "Check latency by region in Grafana"
      - "Compare with baseline metrics"
    queries:
      affected_regions: |
        histogram_quantile(0.95, rate(plunderstorm_game_server_player_rtt_milliseconds_bucket[5m])) > 200

  2_diagnose:
    checks:
      network_issues:
        - "Check ISP status pages for reported issues"
        - "Run traceroute from multiple regions"
        - "Check cloud provider network status"

      server_overload:
        - "Check server CPU/memory utilization"
        - "Check tick rate degradation"
        - "Check entity counts per server"

      matchmaking_issues:
        - "Check if players being matched to wrong regions"
        - "Verify region detection working correctly"

  3_mitigate:
    scenarios:
      network_congestion:
        action: "Redirect traffic to alternate routes"

      server_overload:
        action: "Reduce players per server or scale up"
        command: "kubectl scale deployment/game-server --replicas=+5"

      wrong_region_matching:
        action: "Adjust matchmaking configuration"

  4_verify:
    action: "Confirm latency returned to normal"
    check: "p95 latency < 150ms for 10 minutes"
```

### 8.3 Communication Templates

```yaml
communication_templates:
  status_page_investigating:
    title: "Investigating {{ service }} Issues"
    body: |
      We are investigating reports of {{ symptom }}.

      Started: {{ time }}
      Status: Investigating

      Updates will be provided as available.

  status_page_identified:
    title: "{{ service }} Issue Identified"
    body: |
      We have identified the cause of {{ symptom }}.

      Root cause: {{ root_cause }}
      Impact: {{ impact }}
      ETA for resolution: {{ eta }}

  status_page_resolved:
    title: "{{ service }} Issue Resolved"
    body: |
      The {{ symptom }} has been resolved.

      Duration: {{ duration }}
      Root cause: {{ root_cause }}
      Resolution: {{ resolution }}

      We apologize for any inconvenience caused.

  internal_incident_start:
    channel: "#incidents"
    template: |
      :rotating_light: **INCIDENT STARTED**

      **Alert:** {{ alert_name }}
      **Severity:** {{ severity }}
      **Time:** {{ time }}
      **On-call:** {{ oncall }}

      **Current Status:** Investigating

      Thread for updates below :point_down:

  internal_incident_update:
    template: |
      **UPDATE** ({{ time }})

      {{ update_text }}

      Next update in {{ next_update_time }}
```

---

## 9. SLA Definitions

### 9.1 Service Level Objectives (SLOs)

```yaml
service_level_objectives:
  game_service_availability:
    description: "Game servers available and accepting connections"
    target: 99.9%
    measurement_window: monthly
    calculation: |
      (total_minutes - downtime_minutes) / total_minutes * 100
    exclusions:
      - Scheduled maintenance (with 24h notice)
      - Force majeure events

  match_success_rate:
    description: "Matches that complete successfully"
    target: 99%
    measurement_window: weekly
    calculation: |
      completed_matches / (completed_matches + failed_matches) * 100

  player_latency:
    description: "95th percentile player round-trip time"
    target: "< 150ms"
    measurement_window: daily
    calculation: |
      histogram_quantile(0.95, player_rtt_milliseconds)
    by_region: true

  queue_time:
    description: "95th percentile matchmaking queue time"
    target: "< 60 seconds"
    measurement_window: daily
    calculation: |
      histogram_quantile(0.95, queue_time_seconds)
```

### 9.2 SLO Tracking Dashboard

```json
{
  "dashboard": {
    "title": "SLO Tracking",
    "rows": [
      {
        "title": "Current SLO Status",
        "panels": [
          {
            "title": "Availability SLO",
            "type": "gauge",
            "targets": [
              { "expr": "(1 - avg_over_time(up{job='game-server'}[30d])) * 525600" }
            ],
            "description": "Error budget remaining (minutes)",
            "thresholds": [
              { "value": 43.2, "color": "green" },
              { "value": 21.6, "color": "yellow" },
              { "value": 0, "color": "red" }
            ]
          },
          {
            "title": "Match Success SLO",
            "type": "gauge",
            "targets": [
              {
                "expr": "sum(rate(matches_completed{status='success'}[7d])) / sum(rate(matches_completed[7d])) * 100"
              }
            ]
          },
          {
            "title": "Latency SLO",
            "type": "stat",
            "targets": [
              {
                "expr": "histogram_quantile(0.95, sum(rate(player_rtt_milliseconds_bucket[1d])) by (le))"
              }
            ],
            "unit": "ms"
          }
        ]
      },
      {
        "title": "Error Budget Burn Rate",
        "panels": [
          {
            "title": "Availability Error Budget",
            "type": "graph",
            "targets": [
              {
                "expr": "sum(increase(downtime_minutes[24h]))",
                "legendFormat": "Daily Burn"
              },
              {
                "expr": "43.2 - sum(increase(downtime_minutes[30d]))",
                "legendFormat": "Remaining Budget"
              }
            ]
          }
        ]
      }
    ]
  }
}
```

---

## 10. Testing and Validation

### 10.1 Monitoring Test Scenarios

```yaml
monitoring_tests:
  alert_firing_test:
    description: "Verify alerts fire correctly"
    steps:
      - name: "Inject high error rate"
        action: "Generate synthetic errors exceeding threshold"
        expected: "ElevatedErrorRate alert fires within 10 minutes"

      - name: "Verify notification"
        action: "Check Slack/PagerDuty received alert"
        expected: "Alert message received with correct severity"

      - name: "Clear condition"
        action: "Stop error injection"
        expected: "Alert resolves within 5 minutes"

  dashboard_accuracy_test:
    description: "Verify dashboard data accuracy"
    steps:
      - name: "Generate known traffic"
        action: "Send exactly 1000 requests over 5 minutes"
        expected: "Dashboard shows ~200 req/min"

      - name: "Verify percentiles"
        action: "Send requests with known latency distribution"
        expected: "p50/p95/p99 match expected values within 5%"

  log_pipeline_test:
    description: "Verify logs are collected and searchable"
    steps:
      - name: "Generate unique log message"
        action: "Log message with unique identifier"
        expected: "Message searchable in Kibana within 60 seconds"

      - name: "Verify structured fields"
        action: "Query by specific field values"
        expected: "All fields correctly parsed and indexed"
```

### 10.2 Chaos Engineering Tests

```yaml
chaos_tests:
  server_failure:
    description: "Test monitoring response to server failure"
    procedure:
      - "Terminate a game server instance"
      - "Verify ServerUnhealthy alert fires"
      - "Verify orchestrator detects and replaces server"
      - "Verify alert resolves when replacement is healthy"
    expected_duration: "< 5 minutes"

  network_partition:
    description: "Test monitoring during network issues"
    procedure:
      - "Introduce network latency/packet loss"
      - "Verify latency alerts fire"
      - "Verify player experience metrics reflect degradation"
      - "Remove network impairment"
      - "Verify metrics return to baseline"
    expected_duration: "< 10 minutes"

  log_pipeline_failure:
    description: "Test log pipeline resilience"
    procedure:
      - "Stop Logstash/Filebeat"
      - "Verify logs are buffered locally"
      - "Restart pipeline"
      - "Verify buffered logs are processed"
    expected: "No log loss during outage"
```

---

## 11. Acceptance Criteria Verification

| Criteria | Implementation | Verification |
|----------|---------------|--------------|
| Real-time metrics dashboard | Grafana dashboards with Prometheus | Section 6: Executive, Server, Matchmaking dashboards |
| Alerting for critical issues | Alertmanager with P1-P4 priorities | Section 5: Alert rules and routing |
| Log aggregation and search | ELK stack (Elasticsearch, Logstash, Kibana) | Section 4: Log pipeline and queries |
| Error tracking with stack traces | Structured logging with error categorization | Section 4.3.2: Error log format |
| Performance baselines established | Defined targets for all key metrics | Section 7: Baseline definitions |
| Capacity forecasting data | Prometheus recording rules and queries | Section 7.2: Capacity planning queries |
| On-call procedures documented | Playbooks and escalation paths | Section 8: Complete on-call guide |

---

## 12. Appendix

### 12.1 Metric Naming Conventions

```yaml
naming_conventions:
  format: "{namespace}_{subsystem}_{name}_{unit}"

  namespace: "plunderstorm"

  subsystems:
    - game_server
    - matchmaking
    - orchestration
    - api

  units:
    - total      # Counters
    - seconds    # Duration
    - bytes      # Size
    - percent    # Percentage
    - ratio      # Ratio (0-1)

  examples:
    - plunderstorm_game_server_matches_total
    - plunderstorm_game_server_tick_duration_seconds
    - plunderstorm_matchmaking_queue_time_seconds
```

### 12.2 Required Infrastructure

```yaml
infrastructure_requirements:
  prometheus:
    replicas: 2
    storage: 500GB SSD
    retention: 30 days

  elasticsearch:
    nodes: 3 (cluster)
    storage: 2TB per node
    retention: 90 days (hot), 1 year (cold)

  grafana:
    replicas: 2

  alertmanager:
    replicas: 3 (clustered)

  kafka:
    brokers: 3
    retention: 7 days
```

### 12.3 Related Documents

- [DEDICATED_GAME_SERVERS_SPEC.md](DEDICATED_GAME_SERVERS_SPEC.md) - Server architecture
- [SERVER_ORCHESTRATION_SPEC.md](SERVER_ORCHESTRATION_SPEC.md) - Fleet management
- [REALTIME_NETCODE_SPEC.md](REALTIME_NETCODE_SPEC.md) - Network metrics context
- [SERVER_AUTHORITATIVE_VALIDATION_SPEC.md](SERVER_AUTHORITATIVE_VALIDATION_SPEC.md) - Validation metrics
