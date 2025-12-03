# Match Initialization System Specification

## Document Information
- **Task ID:** GAME-001
- **Priority:** P0
- **Complexity:** L (Large)
- **Dependencies:** BACK-001 (Matchmaking), BACK-004 (Game Server)
- **Last Updated:** 2025-12-02

---

## 1. Overview

### 1.1 Purpose
This specification defines the match initialization system for Plunderstorm Mobile. The system handles the transition from matchmaking to active gameplay, ensuring all 30-60 players join synchronously, game state initializes correctly, and the match begins with all players on equal footing.

### 1.2 Scope

```yaml
initialization_scope:
  player_handling:
    - Connection management
    - Lobby synchronization
    - Ready state tracking
    - Disconnect/reconnect handling

  world_setup:
    - Map loading
    - Loot spawn generation
    - NPC/Monster camp placement
    - Storm initial position

  match_start:
    - Countdown synchronization
    - Drop phase transition
    - State broadcast to all clients
```

### 1.3 Design Goals

| Goal | Target | Priority |
|------|--------|----------|
| Sync Accuracy | All players start within 100ms | Critical |
| Load Time | < 5 seconds initialization | Critical |
| Reliability | 99.9% successful starts | Critical |
| Reconnection | 30s grace period | High |
| Scalability | 60 concurrent players | High |
| Fairness | No player advantage from load time | High |

---

## 2. Match Initialization Flow

### 2.1 State Machine

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MATCH INITIALIZATION STATE MACHINE                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐     │
│  │ CREATED  │───▶│ LOADING  │───▶│  LOBBY   │───▶│COUNTDOWN │     │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘     │
│       │               │               │               │             │
│       │               │               │               ▼             │
│       │               │               │         ┌──────────┐        │
│       │               │               │         │  DROP    │        │
│       │               │               │         │  PHASE   │        │
│       │               │               │         └──────────┘        │
│       │               │               │               │             │
│       │               │               │               ▼             │
│       │               │               │         ┌──────────┐        │
│       │               │               │         │  ACTIVE  │        │
│       │               │               │         │  MATCH   │        │
│       │               │               │         └──────────┘        │
│       │               │               │               │             │
│       ▼               ▼               ▼               ▼             │
│  ┌─────────────────────────────────────────────────────────┐       │
│  │                    CANCELLED / FAILED                    │       │
│  └─────────────────────────────────────────────────────────┘       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Phase Definitions

```yaml
phases:
  created:
    description: "Match instance created by matchmaking"
    duration: "Instant"
    actions:
      - Generate match ID
      - Allocate server resources
      - Initialize match configuration

  loading:
    description: "Players connecting and loading assets"
    duration: "10-30 seconds max"
    actions:
      - Accept player connections
      - Track loading progress
      - Generate world state
      - Prepare loot spawns

  lobby:
    description: "Players loaded, waiting for minimum count"
    duration: "Up to 60 seconds"
    actions:
      - Display player count
      - Show map preview
      - Allow team communication
      - Fill bots if needed

  countdown:
    description: "All players ready, final countdown"
    duration: "5 seconds"
    actions:
      - Lock player roster
      - Broadcast countdown
      - Prepare drop sequence

  drop_phase:
    description: "Players selecting and executing drop"
    duration: "15-20 seconds"
    actions:
      - Show drop ship path
      - Enable drop selection
      - Execute player drops

  active_match:
    description: "Core gameplay active"
    duration: "10-15 minutes"
    actions:
      - Normal gameplay loop
      - Storm progression
      - Victory conditions
```

### 2.3 Detailed Flow Sequence

```yaml
initialization_sequence:
  step_1_match_creation:
    trigger: "Matchmaking service request"
    server_actions:
      - Create match instance with unique ID
      - Load match configuration (mode, player count)
      - Reserve server resources
      - Initialize random seed for deterministic generation
    duration: "< 500ms"

  step_2_player_connection:
    trigger: "Players receive connection info"
    server_actions:
      - Accept WebSocket/UDP connections
      - Authenticate connection tokens
      - Track connected player count
      - Send world loading data
    client_actions:
      - Connect to game server
      - Begin asset loading
      - Report loading progress
    timeout: "30 seconds"

  step_3_world_generation:
    trigger: "First player connects"
    server_actions:
      - Generate loot spawn table
      - Place treasure chests
      - Configure NPC camps
      - Set initial storm parameters
      - Calculate drop ship path
    deterministic: true
    duration: "< 2 seconds"

  step_4_loading_sync:
    trigger: "All expected players connected"
    server_actions:
      - Wait for all clients to report ready
      - Handle slow loaders (progress updates)
      - Timeout stragglers after grace period
    client_actions:
      - Load map geometry
      - Load character assets
      - Load UI elements
      - Report ready state
    timeout: "30 seconds"

  step_5_lobby_phase:
    trigger: "Minimum players ready"
    server_actions:
      - Broadcast player list
      - Start lobby timer (if not full)
      - Add bots if timer expires
    client_display:
      - Player count (X/60)
      - Map preview
      - Team roster (if applicable)
    duration: "0-60 seconds"

  step_6_countdown:
    trigger: "Player threshold met OR lobby timer expires"
    server_actions:
      - Lock player roster
      - Finalize bot additions
      - Broadcast countdown start
    client_display:
      - 5, 4, 3, 2, 1 countdown
      - Match starting animation
    duration: "5 seconds"

  step_7_drop_phase_start:
    trigger: "Countdown reaches 0"
    server_actions:
      - Transition to DROP_PHASE state
      - Start drop ship movement
      - Enable drop inputs
    client_actions:
      - Show drop ship view
      - Enable drop controls
      - Display map with drop zones
```

---

## 3. Player Connection Management

### 3.1 Connection Protocol

```yaml
connection_protocol:
  transport:
    primary: "UDP (gameplay)"
    fallback: "WebSocket (if UDP blocked)"
    handshake: "TCP for initial auth"

  authentication:
    method: "One-time connection token"
    token_source: "Matchmaking service"
    token_expiry: "60 seconds"
    validation: "Server-side verification"

  connection_sequence:
    1_handshake:
      client_sends: "CONNECTION_REQUEST + token"
      server_validates: "Token authenticity"
      server_responds: "CONNECTION_ACCEPTED + player_id"

    2_sync:
      server_sends: "WORLD_STATE packet"
      client_responds: "STATE_RECEIVED"

    3_ready:
      client_sends: "LOADING_COMPLETE"
      server_responds: "READY_CONFIRMED"
```

### 3.2 Player States

```yaml
player_states:
  connecting:
    description: "TCP handshake in progress"
    timeout: "10 seconds"
    on_timeout: "Connection failed"

  loading:
    description: "Client loading assets"
    timeout: "30 seconds"
    progress_updates: "Every 2 seconds"
    on_timeout: "Kicked with error"

  ready:
    description: "Fully loaded, waiting for match"
    timeout: "None (in lobby)"

  dropping:
    description: "In drop phase"
    actions_allowed: ["drop_select", "drop_execute"]

  active:
    description: "Normal gameplay"
    actions_allowed: "All gameplay actions"

  disconnected:
    description: "Lost connection"
    grace_period: "30 seconds"
    on_reconnect: "Restore to last state"
    on_timeout: "Eliminated from match"

  eliminated:
    description: "Killed in match"
    actions_allowed: ["spectate", "leave"]
```

### 3.3 Reconnection Handling

```yaml
reconnection:
  grace_period: 30 seconds

  during_loading:
    action: "Allow immediate reconnect"
    state_restore: "Continue from loading"

  during_lobby:
    action: "Allow immediate reconnect"
    state_restore: "Rejoin lobby"

  during_countdown:
    action: "Allow reconnect if < 3s elapsed"
    state_restore: "Join countdown"

  during_drop:
    action: "Allow reconnect"
    state_restore: "Auto-drop at random safe location"

  during_active:
    action: "Allow reconnect within grace period"
    character_state: "Frozen/invulnerable during disconnect"
    state_restore: "Resume at last position"

  reconnection_flow:
    1: "Client connects with original token + reconnect flag"
    2: "Server validates player was in match"
    3: "Server sends current match state"
    4: "Client syncs and resumes"
```

---

## 4. World State Initialization

### 4.1 Deterministic Generation

```yaml
deterministic_generation:
  seed_source: "Match ID hash + timestamp"

  generated_elements:
    loot_spawns:
      method: "Seeded random from spawn point pool"
      consistency: "All clients see same spawns"

    chest_contents:
      method: "Pre-rolled on server"
      reveal: "On open (anti-cheat)"

    storm_center:
      method: "Weighted random (favor center areas)"
      all_phases: "Pre-calculated at init"

    drop_ship_path:
      method: "Random angle across map center"
      duration: "30 seconds flight time"

    npc_spawns:
      method: "Fixed locations, randomized difficulty"
```

### 4.2 Loot Spawn Generation

```yaml
loot_generation:
  spawn_point_pools:
    high_tier_poi:
      chest_count: "15-20"
      rarity_weights:
        common: 40%
        uncommon: 35%
        rare: 20%
        epic: 5%

    medium_tier_poi:
      chest_count: "8-12"
      rarity_weights:
        common: 55%
        uncommon: 30%
        rare: 13%
        epic: 2%

    low_tier_area:
      chest_count: "3-6"
      rarity_weights:
        common: 70%
        uncommon: 22%
        rare: 7%
        epic: 1%

  total_chests:
    target: "150-200"
    minimum: "120"

  ability_distribution:
    ensure_variety: true
    no_duplicate_epic_same_area: true
```

### 4.3 Initial State Packet

```yaml
initial_state_packet:
  header:
    match_id: "uuid"
    game_mode: "solo | duos | squads"
    random_seed: "int64"
    server_time: "timestamp"

  map_data:
    map_id: "string"
    storm_phases: "array of {center, radius, duration}"
    drop_ship_path: "{start_pos, end_pos, speed}"

  player_roster:
    players: "array of {id, name, team_id, cosmetics}"
    bot_ids: "array of bot player IDs"

  spawn_data:
    chest_locations: "array of {id, position, rarity}"
    npc_camps: "array of {id, position, difficulty}"
    ground_loot: "array of {id, position, type}"

  packet_size:
    target: "< 50KB compressed"
    compression: "LZ4"
```

---

## 5. Synchronization System

### 5.1 Time Synchronization

```yaml
time_sync:
  method: "Server-authoritative clock"

  sync_protocol:
    frequency: "On connect, then every 30 seconds"
    packets:
      client_request: "TIME_SYNC_REQUEST + client_time"
      server_response: "TIME_SYNC_RESPONSE + server_time + client_time"
    calculation: "RTT/2 + server_time = estimated server time"

  tolerance:
    max_drift: "100ms"
    resync_threshold: "50ms drift"

  countdown_sync:
    broadcast: "COUNTDOWN_START + end_timestamp"
    client_display: "server_time - current_time"
    precision: "10ms accuracy"
```

### 5.2 State Synchronization

```yaml
state_sync:
  loading_progress:
    report_interval: "Every 2 seconds or 10% progress"
    server_broadcast: "Aggregate progress to all clients"
    display: "Loading... X players ready"

  lobby_sync:
    player_join: "Broadcast to all on join"
    player_leave: "Broadcast to all on leave"
    ready_state: "Broadcast on change"

  countdown_sync:
    start: "Single broadcast with end_timestamp"
    tick: "Clients calculate locally"
    zero: "Server broadcasts MATCH_START"

  drop_sync:
    selection: "Broadcast drop markers to team"
    execution: "Server validates, broadcasts position"
    landing: "Server confirms, enables gameplay"
```

### 5.3 Failure Handling

```yaml
sync_failures:
  player_fails_to_load:
    timeout: "30 seconds"
    action: "Remove from match, notify others"
    replacement: "Add bot if below minimum"

  player_disconnects_during_countdown:
    if_time_remaining: "> 3 seconds"
    action: "Allow reconnect"
    else: "Replace with bot"

  state_desync_detected:
    detection: "Checksum mismatch on key state"
    action: "Force resync packet"
    repeated: "Kick player with error"

  server_failure:
    detection: "Health check fails"
    action: "Cancel match, return to queue"
    notification: "Error message to all players"
```

---

## 6. Lobby System

### 6.1 Lobby Configuration

```yaml
lobby_config:
  minimum_players:
    solo: 30
    duos: 20  # 10 teams
    squads: 15  # 5 teams

  maximum_players:
    solo: 60
    duos: 60  # 30 teams
    squads: 60  # 20 teams

  lobby_timeout:
    initial_wait: "60 seconds"
    extension_on_progress: "15 seconds if players joining"
    hard_limit: "90 seconds"

  bot_fill:
    trigger: "Timeout OR minimum players met"
    fill_to: "Maximum player count"
```

### 6.2 Lobby UI Data

```yaml
lobby_display:
  player_count:
    format: "X / 60 Players"
    update: "Real-time"

  countdown_timer:
    visibility: "When minimum met OR timeout approaching"
    format: "Match starting in X seconds"

  player_list:
    show: "Player names (teammates highlighted)"
    limit: "Scrollable list"

  map_preview:
    show: "Minimap with POI names"
    interactable: "Zoom/pan"

  team_info:
    duos_squads: "Show team roster"
    communication: "Voice chat enabled"
```

### 6.3 Ready System

```yaml
ready_system:
  implicit_ready:
    trigger: "Loading complete"
    action: "Auto-mark as ready"

  explicit_ready:
    enabled: false  # Not required for BR
    future: "Could add for custom matches"

  ready_tracking:
    server_state: "Track per player"
    broadcast: "On each ready change"
    threshold: "All loaded = proceed to countdown"
```

---

## 7. Bot Integration

### 7.1 Bot Addition Rules

```yaml
bot_rules:
  timing:
    add_after: "Lobby timeout OR minimum threshold"
    add_before: "Countdown start"

  quantity:
    fill_to: "Maximum player count"
    example: "45 real players → 15 bots added"

  distribution:
    spread: "Even across skill tiers"
    naming: "Pirate-themed names"
    cosmetics: "Randomized from available sets"

  initialization:
    state: "Same as real player (loading complete)"
    drop_ai: "Pre-programmed drop locations"
```

### 7.2 Bot Data Structure

```yaml
bot_player:
  id: "BOT_uuid"
  name: "Generated pirate name"
  is_bot: true  # Server-side only
  display_as_bot: false  # Appear as normal player
  team_id: "Assigned team (if team mode)"
  skill_tier: "Matched to lobby average"
  cosmetics:
    outfit: "Random from common pool"
    emotes: "Default set"
  ai_profile: "Random from behavior profiles"
```

---

## 8. Performance Requirements

### 8.1 Timing Budgets

```yaml
timing_budgets:
  match_creation:
    target: "< 500ms"
    includes: "ID generation, resource allocation"

  world_generation:
    target: "< 2 seconds"
    includes: "Loot, NPCs, storm calculation"

  player_loading:
    target: "< 15 seconds average"
    timeout: "30 seconds max"

  lobby_to_countdown:
    target: "< 60 seconds typical"
    max: "90 seconds"

  countdown:
    fixed: "5 seconds"

  total_init_to_drop:
    target: "< 45 seconds"
    max: "90 seconds"
```

### 8.2 Network Budgets

```yaml
network_budgets:
  initial_state_packet:
    size: "< 50KB compressed"
    delivery: "Reliable (TCP/reliable UDP)"

  lobby_updates:
    frequency: "On change (not polling)"
    size: "< 1KB per update"

  countdown_broadcast:
    size: "< 100 bytes"
    delivery: "Reliable"

  concurrent_connections:
    target: "60 players + spectators"
    bandwidth_per_player: "< 5KB/s during init"
```

---

## 9. Error Handling

### 9.1 Error Codes

```yaml
error_codes:
  INIT_001:
    message: "Match creation failed"
    action: "Return to matchmaking"

  INIT_002:
    message: "Connection timeout"
    action: "Show retry option"

  INIT_003:
    message: "Loading failed"
    action: "Return to main menu"

  INIT_004:
    message: "Match cancelled - insufficient players"
    action: "Return to matchmaking queue"

  INIT_005:
    message: "Server error during initialization"
    action: "Return to main menu"

  INIT_006:
    message: "Synchronization failed"
    action: "Reconnect or return to menu"
```

### 9.2 Recovery Procedures

```yaml
recovery:
  client_crash_during_load:
    detection: "Heartbeat timeout"
    action: "Mark disconnected, allow reconnect"
    player_experience: "Rejoin in progress match"

  server_crash_during_init:
    detection: "Health check failure"
    action: "Cancel match instance"
    player_experience: "Error message, return to queue"

  network_partition:
    detection: "Players split into groups"
    action: "Cancel match if > 50% affected"
    player_experience: "Error message, return to queue"
```

---

## 10. Implementation Checklist

```yaml
checklist:
  match_creation:
    - [ ] Match ID generation
    - [ ] Server resource allocation
    - [ ] Configuration loading
    - [ ] Random seed generation

  player_connection:
    - [ ] Connection token validation
    - [ ] Player authentication
    - [ ] State tracking
    - [ ] Timeout handling

  world_generation:
    - [ ] Loot spawn generation
    - [ ] Chest placement
    - [ ] NPC camp setup
    - [ ] Storm phase calculation
    - [ ] Drop ship path

  synchronization:
    - [ ] Time sync protocol
    - [ ] State broadcast
    - [ ] Loading progress tracking
    - [ ] Countdown sync

  lobby:
    - [ ] Player count display
    - [ ] Timer management
    - [ ] Bot fill logic
    - [ ] Ready state tracking

  transitions:
    - [ ] Lobby to countdown
    - [ ] Countdown to drop
    - [ ] Drop to active match

  error_handling:
    - [ ] Disconnect handling
    - [ ] Reconnection flow
    - [ ] Timeout handling
    - [ ] Error notifications

  testing:
    - [ ] 60 player load test
    - [ ] Reconnection testing
    - [ ] Desync detection
    - [ ] Performance profiling
```

---

## 11. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| Queue to lobby transition | Players placed in lobby | Integration test |
| Player count display | Real-time updates | UI test |
| Match starts at threshold | Minimum or timer | Functional test |
| Synchronized start | All within 100ms | Network test |
| State initialization | Loot, storm, NPCs ready | State verification |
| Reconnection works | 30s grace period | Disconnect test |

---

## 12. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-02 | Initial specification |

---

*This document defines the complete match initialization system for Plunderstorm Mobile, ensuring reliable and synchronized match starts for all players.*
