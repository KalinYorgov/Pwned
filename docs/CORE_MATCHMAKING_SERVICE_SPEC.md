# Core Matchmaking Service Specification

## Document Information
- **Task ID:** BACK-001
- **Priority:** P0
- **Complexity:** L (Large)
- **Dependencies:** BACK-002 (Party System), BACK-003 (SBMM)
- **Last Updated:** 2025-12-02

---

## 1. Overview

### 1.1 Purpose
This specification defines the core matchmaking service for Plunderstorm Mobile. The matchmaking system is responsible for grouping players into matches based on region, skill, party status, and game mode while maintaining rapid queue times and fair match composition.

### 1.2 Scope

```yaml
matchmaking_scope:
  game_modes:
    - Solo (60 players)
    - Duos (30 teams of 2)
    - Squads (20 teams of 3)

  features:
    - Regional matchmaking
    - Party queuing
    - Bot backfill
    - Skill-based matching (ranked)
    - Cross-platform support
```

### 1.3 Design Goals

| Goal | Target | Priority |
|------|--------|----------|
| Queue Time | < 30 seconds average | Critical |
| Match Quality | Regional + skill balanced | Critical |
| Reliability | 99.9% uptime | Critical |
| Scalability | 100K+ concurrent players | Critical |
| Party Support | Seamless party queuing | High |
| Bot Fill | Transparent backfill | High |

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MATCHMAKING SYSTEM                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │
│  │   Client     │───▶│   Gateway    │───▶│  MM Service  │          │
│  │   (Mobile)   │◀───│   (API)      │◀───│  (Core)      │          │
│  └──────────────┘    └──────────────┘    └──────────────┘          │
│                                                 │                    │
│                                                 ▼                    │
│                      ┌──────────────────────────────────────┐       │
│                      │         Queue Manager                 │       │
│                      │  ┌────────┐ ┌────────┐ ┌────────┐   │       │
│                      │  │ Solo   │ │ Duos   │ │ Squads │   │       │
│                      │  │ Queue  │ │ Queue  │ │ Queue  │   │       │
│                      │  └────────┘ └────────┘ └────────┘   │       │
│                      └──────────────────────────────────────┘       │
│                                                 │                    │
│                                                 ▼                    │
│                      ┌──────────────────────────────────────┐       │
│                      │         Match Composer                │       │
│                      │  • Player grouping                    │       │
│                      │  • Team balancing                     │       │
│                      │  • Bot fill decisions                 │       │
│                      └──────────────────────────────────────┘       │
│                                                 │                    │
│                                                 ▼                    │
│                      ┌──────────────────────────────────────┐       │
│                      │       Server Allocator                │       │
│                      │  • Request game server                │       │
│                      │  • Region selection                   │       │
│                      │  • Connection info distribution       │       │
│                      └──────────────────────────────────────┘       │
│                                                 │                    │
│                                                 ▼                    │
│                      ┌──────────────────────────────────────┐       │
│                      │       Game Server Pool                │       │
│                      │  ┌────┐ ┌────┐ ┌────┐ ┌────┐       │       │
│                      │  │ NA │ │ EU │ │ AS │ │ SA │       │       │
│                      │  └────┘ └────┘ └────┘ └────┘       │       │
│                      └──────────────────────────────────────┘       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Responsibilities

```yaml
components:
  gateway_api:
    responsibility: "Handle client requests, authentication"
    endpoints:
      - POST /matchmaking/queue
      - DELETE /matchmaking/queue
      - GET /matchmaking/status
      - POST /matchmaking/accept

  matchmaking_service:
    responsibility: "Core matchmaking logic"
    functions:
      - Queue management
      - Player matching
      - Match composition
      - Timeout handling

  queue_manager:
    responsibility: "Manage player queues by mode/region"
    features:
      - Queue insertion/removal
      - Priority handling
      - Queue statistics

  match_composer:
    responsibility: "Build balanced matches"
    features:
      - Player grouping
      - Team assignment
      - Skill balancing
      - Bot fill decisions

  server_allocator:
    responsibility: "Allocate game servers"
    features:
      - Server pool management
      - Region selection
      - Health monitoring
```

### 2.3 Data Flow

```yaml
matchmaking_flow:
  step_1_queue_request:
    input:
      player_id: "string"
      party_id: "string (optional)"
      game_mode: "solo | duos | squads"
      region_preference: "auto | specific"

    validation:
      - Player authenticated
      - Player not already in queue
      - Player not in active match
      - Party valid (if provided)

    output:
      queue_ticket: "string"
      estimated_wait: "seconds"

  step_2_queue_processing:
    frequency: "Every 1 second"
    actions:
      - Group players by region
      - Group players by skill tier
      - Form potential matches
      - Check match viability

  step_3_match_found:
    trigger: "Sufficient players for match"
    actions:
      - Lock players in match
      - Request server allocation
      - Send match found notification
      - Start accept timer (optional)

  step_4_server_allocation:
    trigger: "Match confirmed"
    actions:
      - Select optimal region
      - Allocate server instance
      - Configure match settings
      - Return connection details

  step_5_player_connection:
    trigger: "Server ready"
    actions:
      - Send connection info to all players
      - Track connection status
      - Handle connection failures
      - Start match when ready
```

---

## 3. Matchmaking Logic

### 3.1 Queue Configuration

```yaml
queue_config:
  solo:
    mode_id: "solo"
    players_needed: 60
    min_players: 30
    bot_fill: true
    max_queue_time: 60s
    skill_matching: "relaxed"

  duos:
    mode_id: "duos"
    teams_needed: 30
    team_size: 2
    min_teams: 15
    bot_fill: true
    max_queue_time: 90s
    skill_matching: "relaxed"

  squads:
    mode_id: "squads"
    teams_needed: 20
    team_size: 3
    min_teams: 10
    bot_fill: true
    max_queue_time: 90s
    skill_matching: "relaxed"

  ranked_solo:
    mode_id: "ranked_solo"
    players_needed: 60
    min_players: 50
    bot_fill: false
    max_queue_time: 180s
    skill_matching: "strict"
```

### 3.2 Regional Matchmaking

```yaml
regions:
  north_america:
    code: "NA"
    servers: ["us-east", "us-west"]
    default_server: "us-east"

  europe:
    code: "EU"
    servers: ["eu-west", "eu-central"]
    default_server: "eu-west"

  asia_pacific:
    code: "APAC"
    servers: ["ap-southeast", "ap-northeast"]
    default_server: "ap-southeast"

  south_america:
    code: "SA"
    servers: ["sa-east"]
    default_server: "sa-east"

region_selection:
  auto_detect:
    method: "Ping-based"
    ping_endpoints: true
    cache_duration: 300s

  priority:
    1: "Player preference (if set)"
    2: "Party leader preference"
    3: "Lowest ping region"

  cross_region:
    enabled: false
    fallback_only: true
    max_ping_difference: 100ms
```

### 3.3 Match Composition Algorithm

```yaml
match_composition:
  algorithm: "Tiered Bucket Matching"

  process:
    step_1:
      name: "Bucket Players"
      action: "Group by region, then skill tier"

    step_2:
      name: "Form Candidate Matches"
      action: "Select players from same bucket"

    step_3:
      name: "Balance Check"
      action: "Verify skill distribution acceptable"

    step_4:
      name: "Fill Gaps"
      action: "Add bots or expand search"

  skill_tiers:
    bronze:
      mmr_range: [0, 999]
    silver:
      mmr_range: [1000, 1499]
    gold:
      mmr_range: [1500, 1999]
    platinum:
      mmr_range: [2000, 2499]
    diamond:
      mmr_range: [2500, 2999]
    master:
      mmr_range: [3000, 9999]

  tier_expansion:
    initial_wait: 10s
    expansion_rate: "1 tier per 15s"
    max_expansion: 2 tiers

  team_balancing:
    duos:
      method: "Pair by complementary skill"
      solo_player_fill: true

    squads:
      method: "Distribute skill across team"
      partial_party_fill: true
```

### 3.4 Bot Fill System

```yaml
bot_fill:
  enabled: true
  trigger_conditions:
    - "Queue time > max_queue_time"
    - "Player count < min_players after 30s"
    - "Off-peak hours with low population"

  fill_rules:
    max_bots_percentage: 60%
    min_real_players: 20
    bot_skill_range: "Match average player skill ± 1 tier"

  bot_distribution:
    pattern: "Even distribution across spawn points"
    naming: "Pirate-themed bot names"
    behavior_variety: true

  transparency:
    show_bot_count: false
    bot_identification: "None (appear as players)"

  priority:
    1: "Real players always placed first"
    2: "Bots fill remaining slots"
    3: "Match starts when min threshold met"
```

---

## 4. API Specification

### 4.1 Queue Endpoints

```yaml
endpoints:
  enter_queue:
    method: POST
    path: "/api/v1/matchmaking/queue"

    request:
      headers:
        Authorization: "Bearer {token}"
        Content-Type: "application/json"

      body:
        game_mode: "string (required)"
        party_id: "string (optional)"
        region: "string (optional, default: auto)"

    response_success:
      status: 200
      body:
        queue_ticket: "uuid"
        queue_position: "integer"
        estimated_wait_seconds: "integer"
        queue_status: "searching"

    response_errors:
      400: "Invalid game mode"
      401: "Unauthorized"
      409: "Already in queue"
      423: "Currently in match"

  leave_queue:
    method: DELETE
    path: "/api/v1/matchmaking/queue"

    request:
      headers:
        Authorization: "Bearer {token}"

    response_success:
      status: 200
      body:
        message: "Left queue successfully"

    response_errors:
      401: "Unauthorized"
      404: "Not in queue"

  get_queue_status:
    method: GET
    path: "/api/v1/matchmaking/queue/status"

    request:
      headers:
        Authorization: "Bearer {token}"

    response_success:
      status: 200
      body:
        in_queue: "boolean"
        queue_ticket: "uuid (if in queue)"
        game_mode: "string"
        queue_status: "searching | match_found | connecting"
        queue_time_seconds: "integer"
        estimated_wait_seconds: "integer"
        players_found: "integer (if match_found)"

  accept_match:
    method: POST
    path: "/api/v1/matchmaking/accept"

    request:
      headers:
        Authorization: "Bearer {token}"
      body:
        match_id: "uuid"

    response_success:
      status: 200
      body:
        accepted: true
        all_accepted: "boolean"
        connection_info: "object (if all accepted)"
```

### 4.2 WebSocket Events

```yaml
websocket_events:
  client_to_server:
    queue_heartbeat:
      event: "mm:heartbeat"
      interval: "Every 10 seconds"
      purpose: "Keep queue position, detect disconnects"

  server_to_client:
    queue_update:
      event: "mm:queue_update"
      payload:
        queue_position: "integer"
        estimated_wait: "integer"
        players_in_queue: "integer"

    match_found:
      event: "mm:match_found"
      payload:
        match_id: "uuid"
        game_mode: "string"
        players_found: "integer"
        accept_deadline: "timestamp"

    match_confirmed:
      event: "mm:match_confirmed"
      payload:
        match_id: "uuid"
        server_address: "string"
        server_port: "integer"
        connection_token: "string"
        team_assignment: "integer"

    match_cancelled:
      event: "mm:match_cancelled"
      payload:
        reason: "player_declined | timeout | error"
        requeue: "boolean"

    queue_error:
      event: "mm:error"
      payload:
        error_code: "string"
        message: "string"
```

---

## 5. Party Integration

### 5.1 Party Queue Handling

```yaml
party_matchmaking:
  queue_rules:
    - "Entire party queues together"
    - "Party leader initiates queue"
    - "All members must be ready"
    - "Party placed on same team"

  party_validation:
    pre_queue_checks:
      - "All members online"
      - "No member in another queue"
      - "No member in active match"
      - "Party size matches mode requirements"

  mode_restrictions:
    solo:
      max_party_size: 1
      message: "Solo mode does not allow parties"

    duos:
      max_party_size: 2
      allow_solo: true
      solo_fill: "Match with another solo player"

    squads:
      max_party_size: 3
      allow_smaller: true
      smaller_fill: "Match with other partial parties/solos"

  party_priority:
    full_parties: "Slight priority in queue"
    partial_parties: "Normal priority"
    reason: "Full parties have higher commitment"
```

### 5.2 Cross-Platform Parties

```yaml
cross_platform:
  enabled: true

  party_formation:
    supported: ["iOS", "Android"]
    friend_system: "Unified friend list"
    invite_method: "Username or share link"

  matchmaking:
    pool: "Unified cross-platform pool"
    input_separation: false  # All mobile, same input
    display: "Platform icon next to name"

  restrictions:
    none: "Mobile-to-mobile has no restrictions"
```

---

## 6. Performance Requirements

### 6.1 Latency Targets

```yaml
latency_targets:
  queue_entry:
    target: "< 100ms"
    p99: "< 500ms"

  queue_update:
    frequency: "Every 1 second"
    delivery: "< 50ms"

  match_found_notification:
    target: "< 100ms"
    p99: "< 300ms"

  server_allocation:
    target: "< 2 seconds"
    p99: "< 5 seconds"

  total_queue_to_connect:
    average: "< 35 seconds"
    p99: "< 90 seconds"
```

### 6.2 Scalability Targets

```yaml
scalability:
  concurrent_players:
    target: 100,000
    peak: 500,000

  queue_throughput:
    matches_per_minute: 1,000
    players_per_minute: 60,000

  horizontal_scaling:
    matchmaking_instances: "Auto-scale 2-20"
    queue_sharding: "By region"
    stateless_design: true

  database:
    type: "Redis cluster"
    purpose: "Queue state, player status"
    replication: "Multi-region"
```

### 6.3 Reliability

```yaml
reliability:
  uptime_target: "99.9%"

  failure_handling:
    player_disconnect:
      action: "Remove from queue after 30s timeout"
      requeue: "Automatic if reconnects"

    service_failure:
      action: "Failover to backup instance"
      queue_state: "Preserved in Redis"

    match_creation_failure:
      action: "Return players to queue with priority"
      notification: "Error message to players"

  circuit_breaker:
    enabled: true
    threshold: "50% failure rate"
    recovery: "Gradual increase"
```

---

## 7. Monitoring and Analytics

### 7.1 Key Metrics

```yaml
metrics:
  queue_metrics:
    - name: "queue_size"
      type: "gauge"
      labels: ["region", "game_mode"]

    - name: "queue_wait_time"
      type: "histogram"
      buckets: [5, 10, 20, 30, 45, 60, 90, 120]
      labels: ["region", "game_mode"]

    - name: "matches_created"
      type: "counter"
      labels: ["region", "game_mode"]

    - name: "queue_entries"
      type: "counter"
      labels: ["region", "game_mode"]

    - name: "queue_cancellations"
      type: "counter"
      labels: ["region", "game_mode", "reason"]

  match_metrics:
    - name: "match_fill_rate"
      type: "histogram"
      description: "Percentage of real players vs bots"

    - name: "match_skill_variance"
      type: "histogram"
      description: "Skill spread within match"

    - name: "server_allocation_time"
      type: "histogram"
      buckets: [0.5, 1, 2, 3, 5, 10]
```

### 7.2 Alerting

```yaml
alerts:
  critical:
    - name: "High Queue Times"
      condition: "avg(queue_wait_time) > 60s for 5m"
      action: "Page on-call, increase bot fill"

    - name: "Matchmaking Service Down"
      condition: "health_check fails for 30s"
      action: "Page on-call, failover"

    - name: "No Servers Available"
      condition: "available_servers == 0 for region"
      action: "Page on-call, spin up emergency capacity"

  warning:
    - name: "Elevated Queue Times"
      condition: "avg(queue_wait_time) > 45s for 10m"
      action: "Notify team"

    - name: "High Bot Fill Rate"
      condition: "avg(bot_fill_rate) > 50% for 15m"
      action: "Notify team"
```

---

## 8. Edge Cases and Error Handling

### 8.1 Edge Cases

```yaml
edge_cases:
  player_disconnect_during_queue:
    detection: "Missing heartbeat for 30s"
    action: "Remove from queue"
    reconnect: "Can rejoin queue normally"

  player_disconnect_during_match_found:
    detection: "No accept response within timeout"
    action: "Cancel match, requeue others with priority"
    penalty: "Possible queue cooldown if repeated"

  party_member_disconnects:
    detection: "Party service notification"
    action: "Remove entire party from queue"
    notification: "Inform remaining party members"

  server_allocation_failure:
    detection: "Allocation timeout (5s)"
    action: "Retry with different server"
    fallback: "Different region if available"
    failure: "Cancel match, requeue players"

  duplicate_queue_entry:
    detection: "Player ID already in queue"
    action: "Reject with 409 Conflict"
    message: "Already in queue"

  match_during_queue:
    detection: "Player state check"
    action: "Reject queue entry"
    message: "Cannot queue during active match"
```

### 8.2 Error Codes

```yaml
error_codes:
  MM_001:
    message: "Already in queue"
    http_status: 409

  MM_002:
    message: "Invalid game mode"
    http_status: 400

  MM_003:
    message: "Currently in active match"
    http_status: 423

  MM_004:
    message: "Party validation failed"
    http_status: 400

  MM_005:
    message: "Region unavailable"
    http_status: 503

  MM_006:
    message: "Queue service unavailable"
    http_status: 503

  MM_007:
    message: "Match creation failed"
    http_status: 500

  MM_008:
    message: "Server allocation failed"
    http_status: 503
```

---

## 9. Security Considerations

### 9.1 Authentication

```yaml
authentication:
  queue_entry:
    required: true
    method: "JWT token"
    validation: "Token not expired, player ID matches"

  websocket:
    required: true
    method: "Token in connection params"
    heartbeat: "Required every 30s"

  match_connection:
    method: "One-time connection token"
    expiry: "60 seconds"
    single_use: true
```

### 9.2 Rate Limiting

```yaml
rate_limits:
  queue_entry:
    limit: "5 per minute per player"
    cooldown: "10s between attempts"

  queue_cancel:
    limit: "10 per minute per player"

  status_check:
    limit: "60 per minute per player"

  penalty_system:
    repeated_decline:
      threshold: "3 declines in 10 minutes"
      penalty: "5 minute queue ban"

    repeated_abandon:
      threshold: "2 abandons in 1 hour"
      penalty: "15 minute queue ban"
```

---

## 10. Implementation Checklist

### 10.1 Core Features

```yaml
checklist:
  queue_management:
    - [ ] Queue entry endpoint
    - [ ] Queue exit endpoint
    - [ ] Queue status endpoint
    - [ ] WebSocket connection
    - [ ] Heartbeat handling
    - [ ] Timeout handling

  match_composition:
    - [ ] Player grouping algorithm
    - [ ] Regional matching
    - [ ] Skill tier matching
    - [ ] Bot fill logic
    - [ ] Team assignment

  server_allocation:
    - [ ] Server pool integration
    - [ ] Region selection
    - [ ] Connection token generation
    - [ ] Health checking

  party_support:
    - [ ] Party queue validation
    - [ ] Party placement (same team)
    - [ ] Partial party fill

  notifications:
    - [ ] Queue update events
    - [ ] Match found event
    - [ ] Match confirmed event
    - [ ] Error events

  monitoring:
    - [ ] Metrics collection
    - [ ] Dashboard setup
    - [ ] Alerting configuration

  testing:
    - [ ] Unit tests
    - [ ] Integration tests
    - [ ] Load tests (1000+ concurrent)
    - [ ] Chaos testing
```

---

## 11. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| Solo/team queue support | All modes working | Functional test |
| Regional grouping | < 100ms ping difference | Network test |
| Queue time < 30s | Average with sufficient players | Load test |
| Party queue together | Same team assignment | Integration test |
| Skill matching (ranked) | Tier variance < 2 | Match analysis |
| Cancel queue instant | < 100ms response | Latency test |
| Queue status visible | Real-time updates | UI test |
| Bot fill working | Matches start with min players | Functional test |
| Disconnect handling | Graceful removal | Edge case test |

---

## 12. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-02 | Initial specification |

---

*This document defines the complete matchmaking service specification for Plunderstorm Mobile, ensuring fast, fair, and reliable player matching.*
