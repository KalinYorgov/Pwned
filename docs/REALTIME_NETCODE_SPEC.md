# Real-Time Netcode Specification - Plunderstorm Mobile

**Document ID:** BACK-007
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Network Architecture](#2-network-architecture)
3. [Protocol Design](#3-protocol-design)
4. [State Synchronization](#4-state-synchronization)
5. [Client-Side Prediction](#5-client-side-prediction)
6. [Server Reconciliation](#6-server-reconciliation)
7. [Lag Compensation](#7-lag-compensation)
8. [Delta Compression](#8-delta-compression)
9. [Interest Management](#9-interest-management)
10. [Mobile Network Optimization](#10-mobile-network-optimization)
11. [Packet Loss Handling](#11-packet-loss-handling)
12. [Bandwidth Management](#12-bandwidth-management)
13. [Performance Requirements](#13-performance-requirements)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the real-time netcode system for Plunderstorm Mobile, providing smooth, responsive gameplay synchronization for 60 players across variable mobile network conditions.

### 1.2 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Responsiveness | Actions feel instant despite latency | Critical |
| Smoothness | No visible jitter or rubberbanding | Critical |
| Fairness | Combat feels fair for all players | Critical |
| Efficiency | Minimize bandwidth usage | High |
| Resilience | Handle packet loss gracefully | High |
| Scalability | Support 60 players at 20Hz | Critical |

### 1.3 Key Metrics

| Metric | Target | Maximum |
|--------|--------|---------|
| Sync Rate | 20 Hz | - |
| Input Latency (perceived) | < 50ms | 100ms |
| Network Bandwidth (per player) | < 30 KB/s | 50 KB/s |
| Supported RTT | < 150ms | 250ms |
| Packet Loss Tolerance | < 5% | 10% |
| Position Accuracy | < 0.5 units | 1 unit |

### 1.4 System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT                                       │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │   Input     │  │ Prediction  │  │   Render    │                 │
│  │   System    │─▶│   System    │─▶│   System    │                 │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
│        │                │                                           │
│        ▼                ▼                                           │
│  ┌─────────────┐  ┌─────────────┐                                  │
│  │   Network   │  │Reconciliation│                                  │
│  │   Layer     │◀─│   System    │                                  │
│  └──────┬──────┘  └─────────────┘                                  │
└─────────┼───────────────────────────────────────────────────────────┘
          │ UDP/DTLS
          │
┌─────────┼───────────────────────────────────────────────────────────┐
│         ▼                        SERVER                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │   Network   │  │    Game     │  │    Lag      │                 │
│  │   Layer     │─▶│   State     │◀─│Compensation │                 │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
│                          │                                          │
│                          ▼                                          │
│                   ┌─────────────┐                                   │
│                   │   Delta     │                                   │
│                   │ Compression │                                   │
│                   └─────────────┘                                   │
│                          │                                          │
│                   ┌──────▼──────┐                                   │
│                   │  Interest   │                                   │
│                   │ Management  │                                   │
│                   └─────────────┘                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Network Architecture

### 2.1 Transport Layer

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRANSPORT ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Application Layer                         │   │
│  │  • Game State  • Player Input  • Events  • RPCs             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Reliability Layer                         │   │
│  │  • Unreliable (state sync)  • Reliable (events, RPCs)       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Security Layer                            │   │
│  │  • DTLS 1.3 Encryption  • Connection Authentication         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Transport Layer                           │   │
│  │  • UDP  • Congestion Control  • MTU Discovery               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Channel Types

| Channel | Reliability | Ordering | Use Case |
|---------|-------------|----------|----------|
| State | Unreliable | None | Position/state updates |
| Input | Unreliable | Sequenced | Player inputs |
| Events | Reliable | Ordered | Eliminations, loot pickup |
| RPC | Reliable | Ordered | Ability casts, interactions |
| Voice | Unreliable | Sequenced | Voice chat |

### 2.3 Connection Management

```cpp
class NetworkConnection {
public:
    // Connection states
    enum class State {
        Disconnected,
        Connecting,
        Connected,
        Reconnecting
    };

    // Establish connection
    async Task<bool> Connect(const ServerEndpoint& endpoint, const AuthToken& token) {
        m_state = State::Connecting;

        // DTLS handshake
        if (!await m_dtls.Handshake(endpoint)) {
            m_state = State::Disconnected;
            return false;
        }

        // Send authentication
        auto authPacket = CreateAuthPacket(token);
        await SendReliable(authPacket);

        // Wait for acknowledgment
        auto response = await WaitForAuth(TIMEOUT_AUTH);
        if (!response.success) {
            m_state = State::Disconnected;
            return false;
        }

        m_state = State::Connected;
        m_playerId = response.playerId;
        m_sessionId = response.sessionId;

        // Start receive loop
        StartReceiveLoop();

        return true;
    }

    // Send unreliable data (state sync)
    void SendUnreliable(const Packet& packet) {
        auto encrypted = m_dtls.Encrypt(packet.Serialize());
        m_socket.SendTo(m_serverEndpoint, encrypted);
        m_stats.bytesSent += encrypted.size();
    }

    // Send reliable data (events, RPCs)
    async Task SendReliable(const Packet& packet) {
        uint32_t sequenceNum = m_reliableSequence++;
        auto reliablePacket = WrapReliable(packet, sequenceNum);

        // Add to pending acknowledgments
        m_pendingAcks[sequenceNum] = {
            .packet = reliablePacket,
            .sendTime = Clock::Now(),
            .retryCount = 0
        };

        SendUnreliable(reliablePacket);

        // Wait for ACK
        while (!m_acknowledgedSequences.contains(sequenceNum)) {
            await Task::Delay(RETRY_INTERVAL);

            if (m_pendingAcks[sequenceNum].retryCount >= MAX_RETRIES) {
                throw ConnectionLostException();
            }

            // Resend
            m_pendingAcks[sequenceNum].retryCount++;
            SendUnreliable(reliablePacket);
        }

        m_pendingAcks.erase(sequenceNum);
    }

private:
    static constexpr int MAX_RETRIES = 10;
    static constexpr float RETRY_INTERVAL = 0.1f;  // 100ms
    static constexpr float TIMEOUT_AUTH = 10.0f;   // 10s
};
```

---

## 3. Protocol Design

### 3.1 Packet Structure

```cpp
// Base packet header (4 bytes)
struct PacketHeader {
    uint8_t type : 4;        // Packet type (0-15)
    uint8_t flags : 4;       // Flags (reliable, compressed, etc.)
    uint8_t channel;         // Channel ID
    uint16_t sequence;       // Sequence number
};

// Packet types
enum class PacketType : uint8_t {
    // Unreliable
    StateUpdate = 0,         // Server → Client: World state delta
    PlayerInput = 1,         // Client → Server: Player input
    Ping = 2,                // Bidirectional: Latency measurement

    // Reliable
    Event = 3,               // Bidirectional: Game events
    RPC = 4,                 // Bidirectional: Remote procedure calls
    Ack = 5,                 // Acknowledgment

    // Connection
    Connect = 6,             // Client → Server: Connection request
    ConnectResponse = 7,     // Server → Client: Connection response
    Disconnect = 8,          // Bidirectional: Disconnect notification
    Heartbeat = 9,           // Bidirectional: Keep-alive

    // State
    FullState = 10,          // Server → Client: Full world state
    RequestState = 11        // Client → Server: Request full state
};

// Flags
enum PacketFlags : uint8_t {
    FLAG_RELIABLE = 0x01,
    FLAG_COMPRESSED = 0x02,
    FLAG_FRAGMENTED = 0x04,
    FLAG_LAST_FRAGMENT = 0x08
};
```

### 3.2 State Update Packet

```cpp
// Server → Client: State update packet
struct StateUpdatePacket {
    PacketHeader header;

    // Timing
    uint64_t serverTick;          // Current server tick
    uint64_t clientAckTick;       // Last acknowledged client tick
    float serverTime;             // Server match time

    // Delta base
    uint64_t deltaBaseTick;       // Tick this delta is relative to

    // Player states (delta compressed)
    uint8_t playerCount;
    PlayerStateDelta players[playerCount];

    // Entity states
    uint8_t entityCount;
    EntityStateDelta entities[entityCount];

    // World state
    StormStateDelta storm;
    EventList events;
};

// Player state delta
struct PlayerStateDelta {
    uint16_t playerId;
    uint8_t changedFlags;         // Which fields changed

    // Position (if changed) - quantized
    int16_t posX;                 // 0.01 unit precision
    int16_t posY;
    int16_t posZ;

    // Rotation (if changed)
    int8_t rotation;              // 0-255 maps to 0-360 degrees

    // Velocity (if changed)
    int8_t velX;                  // Normalized -128 to 127
    int8_t velY;
    int8_t velZ;

    // Health/Shield (if changed)
    uint8_t health;               // 0-255 maps to 0-maxHealth
    uint8_t shield;               // 0-255 maps to 0-maxShield

    // Animation state (if changed)
    uint8_t animState;
    uint8_t animFrame;
};
```

### 3.3 Input Packet

```cpp
// Client → Server: Input packet
struct InputPacket {
    PacketHeader header;

    // Timing
    uint64_t clientTick;          // Client tick when input generated
    uint64_t lastServerTick;      // Last received server tick
    float clientTime;             // Client timestamp

    // Input data (multiple inputs for redundancy)
    uint8_t inputCount;           // Number of inputs (1-3)
    PlayerInput inputs[inputCount];
};

// Single input frame
struct PlayerInput {
    uint64_t tick;                // Tick this input is for
    uint32_t sequence;            // Sequence number

    // Movement (2 bytes)
    int8_t moveX;                 // -128 to 127 (normalized)
    int8_t moveY;                 // -128 to 127 (normalized)

    // Look direction (2 bytes)
    uint16_t lookAngle;           // 0-65535 maps to 0-360 degrees

    // Actions (1 byte bitfield)
    uint8_t actions;              // Jump, sprint, crouch, etc.

    // Combat (3 bytes)
    uint8_t fireState;            // 0=none, 1=primary, 2=secondary
    uint16_t aimAngle;            // Vertical aim angle

    // Ability (2 bytes)
    int8_t abilitySlot;           // -1 = none, 0-3 = ability
    uint8_t abilityTarget;        // Target type/direction
};

// Action flags
enum ActionFlags : uint8_t {
    ACTION_JUMP = 0x01,
    ACTION_SPRINT = 0x02,
    ACTION_CROUCH = 0x04,
    ACTION_INTERACT = 0x08,
    ACTION_RELOAD = 0x10,
    ACTION_USE_ITEM = 0x20
};
```

---

## 4. State Synchronization

### 4.1 Sync Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                   STATE SYNCHRONIZATION                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  SERVER (20 Hz tick rate)                                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Tick 100: Update world state                              │  │
│  │           │                                                │  │
│  │           ▼                                                │  │
│  │ For each player:                                          │  │
│  │   1. Get relevant entities (interest management)          │  │
│  │   2. Calculate delta from last acked state                │  │
│  │   3. Compress delta                                       │  │
│  │   4. Send state update packet                             │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              │ ~50ms network latency             │
│                              ▼                                   │
│  CLIENT                                                          │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Receive state update for tick 100                         │  │
│  │           │                                                │  │
│  │           ▼                                                │  │
│  │ 1. Decompress delta                                       │  │
│  │ 2. Apply to snapshot buffer                               │  │
│  │ 3. Interpolate between snapshots for rendering            │  │
│  │ 4. Reconcile local player prediction                      │  │
│  │ 5. Send ACK                                               │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 4.2 Server-Side Sync

```cpp
class ServerStateSync {
public:
    void Tick() {
        m_currentTick++;

        // Update world state
        m_world->Tick(m_tickDeltaTime);

        // Send state updates to each player
        for (auto& [playerId, connection] : m_connections) {
            SendStateUpdate(playerId, connection);
        }
    }

    void SendStateUpdate(PlayerId playerId, PlayerConnection& conn) {
        // Get player position for interest management
        auto& playerState = m_world->GetPlayer(playerId);

        // Get relevant entities
        auto relevantEntities = m_interestManager->GetRelevantEntities(
            playerState.position,
            playerId
        );

        // Calculate delta from last acknowledged state
        uint64_t baseTick = conn.lastAckedTick;
        auto baseSnapshot = m_snapshotBuffer.Get(baseTick);

        StateUpdatePacket packet;
        packet.serverTick = m_currentTick;
        packet.clientAckTick = conn.lastReceivedClientTick;
        packet.deltaBaseTick = baseTick;

        // Build player deltas
        for (auto& [id, state] : m_world->GetPlayers()) {
            if (!relevantEntities.contains(id) && id != playerId) {
                continue;  // Not relevant
            }

            auto* baseState = baseSnapshot ?
                baseSnapshot->GetPlayer(id) : nullptr;

            auto delta = CalculatePlayerDelta(baseState, &state);
            if (delta.hasChanges) {
                packet.players.push_back(delta);
            }
        }

        // Build entity deltas (NPCs, projectiles, loot)
        for (auto& entityId : relevantEntities) {
            auto* entity = m_world->GetEntity(entityId);
            if (!entity) continue;

            auto* baseEntity = baseSnapshot ?
                baseSnapshot->GetEntity(entityId) : nullptr;

            auto delta = CalculateEntityDelta(baseEntity, entity);
            if (delta.hasChanges) {
                packet.entities.push_back(delta);
            }
        }

        // Add storm state
        packet.storm = CalculateStormDelta(
            baseSnapshot ? &baseSnapshot->storm : nullptr,
            &m_world->GetStorm()
        );

        // Compress and send
        auto compressed = m_compressor->Compress(packet);
        conn.SendUnreliable(compressed);

        // Store snapshot for future deltas
        m_snapshotBuffer.Store(m_currentTick, m_world->CreateSnapshot());
    }

private:
    PlayerStateDelta CalculatePlayerDelta(
        const PlayerState* base,
        const PlayerState* current
    ) {
        PlayerStateDelta delta;
        delta.playerId = current->id;
        delta.changedFlags = 0;

        // Position
        if (!base || base->position != current->position) {
            delta.changedFlags |= CHANGED_POSITION;
            delta.posX = QuantizePosition(current->position.x);
            delta.posY = QuantizePosition(current->position.y);
            delta.posZ = QuantizePosition(current->position.z);
        }

        // Rotation
        if (!base || base->rotation != current->rotation) {
            delta.changedFlags |= CHANGED_ROTATION;
            delta.rotation = QuantizeRotation(current->rotation);
        }

        // Velocity
        if (!base || base->velocity != current->velocity) {
            delta.changedFlags |= CHANGED_VELOCITY;
            delta.velX = QuantizeVelocity(current->velocity.x);
            delta.velY = QuantizeVelocity(current->velocity.y);
            delta.velZ = QuantizeVelocity(current->velocity.z);
        }

        // Health/Shield
        if (!base || base->health != current->health) {
            delta.changedFlags |= CHANGED_HEALTH;
            delta.health = QuantizeHealth(current->health, current->maxHealth);
        }

        if (!base || base->shield != current->shield) {
            delta.changedFlags |= CHANGED_SHIELD;
            delta.shield = QuantizeHealth(current->shield, current->maxShield);
        }

        delta.hasChanges = delta.changedFlags != 0;
        return delta;
    }

    int16_t QuantizePosition(float value) {
        return static_cast<int16_t>(value * 100.0f);  // 0.01 unit precision
    }

    uint8_t QuantizeRotation(float degrees) {
        return static_cast<uint8_t>((degrees / 360.0f) * 255.0f);
    }
};
```

### 4.3 Client-Side Sync

```cpp
class ClientStateSync {
public:
    void ReceiveStateUpdate(const StateUpdatePacket& packet) {
        // Update network stats
        m_lastServerTick = packet.serverTick;
        m_serverTime = packet.serverTime;

        // Calculate RTT from our acknowledged tick
        if (packet.clientAckTick > 0) {
            auto sentTime = m_inputHistory.GetSendTime(packet.clientAckTick);
            m_rtt = Clock::Now() - sentTime;
            m_smoothedRtt = m_smoothedRtt * 0.9f + m_rtt * 0.1f;
        }

        // Decompress and apply state
        auto snapshot = DecompressSnapshot(packet);

        // Store in snapshot buffer for interpolation
        m_snapshotBuffer.Store(packet.serverTick, snapshot);

        // Apply to other players (interpolated)
        for (auto& playerDelta : snapshot.players) {
            if (playerDelta.playerId == m_localPlayerId) {
                // Local player - handle through reconciliation
                continue;
            }

            ApplyRemotePlayerState(playerDelta);
        }

        // Apply entity states
        for (auto& entityDelta : snapshot.entities) {
            ApplyEntityState(entityDelta);
        }

        // Update storm
        ApplyStormState(snapshot.storm);

        // Process events
        ProcessEvents(packet.events);

        // Reconcile local player prediction
        ReconcileLocalPlayer(packet);

        // Send ACK
        SendAck(packet.serverTick);
    }

    void ApplyRemotePlayerState(const PlayerStateDelta& delta) {
        auto* player = m_world->GetPlayer(delta.playerId);
        if (!player) {
            // New player - create
            player = m_world->CreatePlayer(delta.playerId);
        }

        // Store authoritative state for interpolation
        PlayerState authState;
        authState.position = DequantizePosition(delta.posX, delta.posY, delta.posZ);
        authState.rotation = DequantizeRotation(delta.rotation);
        authState.velocity = DequantizeVelocity(delta.velX, delta.velY, delta.velZ);
        authState.health = DequantizeHealth(delta.health, player->maxHealth);
        authState.shield = DequantizeHealth(delta.shield, player->maxShield);

        player->SetAuthoritativeState(authState, m_lastServerTick);
    }

    // Interpolation for remote players
    void InterpolateRemotePlayers(float renderTime) {
        // Render at time = server_time - interpolation_delay
        float interpTime = m_serverTime - INTERPOLATION_DELAY;

        for (auto& [id, player] : m_world->GetPlayers()) {
            if (id == m_localPlayerId) continue;

            // Get two snapshots to interpolate between
            auto [before, after] = m_snapshotBuffer.GetBracketingSnapshots(interpTime);

            if (!before || !after) {
                // Not enough data - extrapolate or use last known
                continue;
            }

            // Calculate interpolation factor
            float t = (interpTime - before->time) / (after->time - before->time);
            t = std::clamp(t, 0.0f, 1.0f);

            // Interpolate position
            player->renderPosition = Vector3::Lerp(
                before->GetPlayer(id)->position,
                after->GetPlayer(id)->position,
                t
            );

            // Interpolate rotation (shortest path)
            player->renderRotation = LerpAngle(
                before->GetPlayer(id)->rotation,
                after->GetPlayer(id)->rotation,
                t
            );
        }
    }

private:
    static constexpr float INTERPOLATION_DELAY = 0.1f;  // 100ms (2 ticks at 20Hz)
};
```

---

## 5. Client-Side Prediction

### 5.1 Prediction System

```cpp
class ClientPrediction {
public:
    void ProcessLocalInput(const PlayerInput& input) {
        // Store input for reconciliation
        m_inputHistory.Store(input);

        // Apply input locally immediately
        ApplyInput(m_localPlayer, input);

        // Store predicted state
        PredictedState prediction;
        prediction.tick = input.tick;
        prediction.position = m_localPlayer->position;
        prediction.velocity = m_localPlayer->velocity;
        prediction.rotation = m_localPlayer->rotation;
        m_predictionHistory.Store(prediction);

        // Send input to server
        SendInput(input);
    }

    void ApplyInput(PlayerState* player, const PlayerInput& input) {
        // Calculate movement direction
        Vector3 moveDir(
            input.moveX / 127.0f,
            0,
            input.moveY / 127.0f
        );

        // Rotate by look direction
        float lookRad = (input.lookAngle / 65535.0f) * 2.0f * PI;
        moveDir = RotateVector(moveDir, lookRad);

        // Calculate speed
        float speed = PLAYER_MOVE_SPEED;
        if (input.actions & ACTION_SPRINT) {
            speed *= SPRINT_MULTIPLIER;
        }
        if (input.actions & ACTION_CROUCH) {
            speed *= CROUCH_MULTIPLIER;
        }

        // Apply status effects
        speed *= player->GetSpeedMultiplier();

        // Update velocity
        player->velocity = moveDir * speed;

        // Apply gravity
        if (!player->isGrounded) {
            player->velocity.y -= GRAVITY * TICK_DELTA;
        }

        // Update position
        Vector3 newPos = player->position + player->velocity * TICK_DELTA;

        // Collision detection (client-side approximation)
        newPos = m_collision->ResolveCollision(player->position, newPos);

        player->position = newPos;

        // Update rotation
        player->rotation = input.lookAngle / 65535.0f * 360.0f;

        // Handle jump
        if ((input.actions & ACTION_JUMP) && player->isGrounded) {
            player->velocity.y = JUMP_VELOCITY;
            player->isGrounded = false;
        }
    }

private:
    static constexpr float PLAYER_MOVE_SPEED = 8.0f;
    static constexpr float SPRINT_MULTIPLIER = 1.5f;
    static constexpr float CROUCH_MULTIPLIER = 0.5f;
    static constexpr float GRAVITY = 20.0f;
    static constexpr float JUMP_VELOCITY = 10.0f;
    static constexpr float TICK_DELTA = 1.0f / 20.0f;
};
```

### 5.2 Visual Prediction

```cpp
class VisualPrediction {
public:
    // Predict visual-only effects immediately
    void PredictAbilityCast(AbilityId ability, const Vector3& target) {
        // Play cast animation immediately
        m_localPlayer->PlayAnimation(GetCastAnimation(ability));

        // Spawn predicted visual effects
        SpawnPredictedVFX(ability, target);

        // Play sound effect
        PlaySound(GetCastSound(ability));

        // Store for potential rollback
        m_predictedEffects.push_back({
            .ability = ability,
            .target = target,
            .tick = m_currentTick,
            .vfxId = m_lastVfxId
        });
    }

    void ConfirmOrRollback(uint64_t serverTick, bool confirmed) {
        // Find predicted effect
        auto it = std::find_if(
            m_predictedEffects.begin(),
            m_predictedEffects.end(),
            [serverTick](const auto& e) { return e.tick == serverTick; }
        );

        if (it == m_predictedEffects.end()) return;

        if (!confirmed) {
            // Rollback - remove predicted VFX
            DestroyVFX(it->vfxId);

            // Show correction effect (subtle)
            SpawnCorrectionEffect(it->target);
        }

        m_predictedEffects.erase(it);
    }

    // Predict hit feedback
    void PredictHit(PlayerId target, float damage, bool isHeadshot) {
        // Show hit marker immediately
        ShowHitMarker(isHeadshot);

        // Play hit sound
        PlaySound(isHeadshot ? SFX_HEADSHOT : SFX_HIT);

        // Predict damage number
        SpawnDamageNumber(target, damage, isHeadshot);
    }
};
```

---

## 6. Server Reconciliation

### 6.1 Reconciliation System

```cpp
class ServerReconciliation {
public:
    void Reconcile(const StateUpdatePacket& serverState) {
        // Get the server's acknowledged client tick
        uint64_t serverAckTick = serverState.clientAckTick;

        // Find server state for local player
        const PlayerStateDelta* serverPlayerState = nullptr;
        for (const auto& player : serverState.players) {
            if (player.playerId == m_localPlayerId) {
                serverPlayerState = &player;
                break;
            }
        }

        if (!serverPlayerState) return;

        // Get our predicted state at that tick
        auto predictedState = m_predictionHistory.Get(serverAckTick);
        if (!predictedState) return;

        // Calculate error
        Vector3 serverPos = DequantizePosition(
            serverPlayerState->posX,
            serverPlayerState->posY,
            serverPlayerState->posZ
        );

        Vector3 error = serverPos - predictedState->position;
        float errorMagnitude = error.Length();

        // Record for stats
        m_predictionErrorHistory.Record(errorMagnitude);

        // Small errors - smoothly correct
        if (errorMagnitude < SMOOTH_CORRECTION_THRESHOLD) {
            SmoothCorrect(serverPos, error);
            return;
        }

        // Large errors - full reconciliation
        FullReconciliation(serverState, serverAckTick);
    }

    void SmoothCorrect(const Vector3& serverPos, const Vector3& error) {
        // Gradually correct position over multiple frames
        m_positionCorrection += error;

        // Apply correction smoothly
        float correctionRate = 0.1f;  // 10% per frame
        Vector3 correction = m_positionCorrection * correctionRate;

        m_localPlayer->position += correction;
        m_positionCorrection -= correction;

        // Clear if small enough
        if (m_positionCorrection.Length() < 0.01f) {
            m_positionCorrection = Vector3::Zero;
        }
    }

    void FullReconciliation(
        const StateUpdatePacket& serverState,
        uint64_t serverTick
    ) {
        LOG_DEBUG("Full reconciliation at tick {}", serverTick);

        // Reset to server state
        const auto& serverPlayer = FindPlayer(serverState, m_localPlayerId);
        m_localPlayer->position = DequantizePosition(
            serverPlayer.posX, serverPlayer.posY, serverPlayer.posZ
        );
        m_localPlayer->velocity = DequantizeVelocity(
            serverPlayer.velX, serverPlayer.velY, serverPlayer.velZ
        );
        m_localPlayer->rotation = DequantizeRotation(serverPlayer.rotation);

        // Re-apply all inputs since server tick
        auto unackedInputs = m_inputHistory.GetInputsSince(serverTick);

        for (const auto& input : unackedInputs) {
            m_prediction->ApplyInput(m_localPlayer, input);

            // Update prediction history
            PredictedState newPrediction;
            newPrediction.tick = input.tick;
            newPrediction.position = m_localPlayer->position;
            newPrediction.velocity = m_localPlayer->velocity;
            m_predictionHistory.Store(newPrediction);
        }

        // Clear position correction
        m_positionCorrection = Vector3::Zero;
    }

private:
    static constexpr float SMOOTH_CORRECTION_THRESHOLD = 0.5f;  // 0.5 units
};
```

### 6.2 Reconciliation Metrics

```cpp
class ReconciliationMetrics {
public:
    void Update() {
        // Calculate statistics
        float avgError = CalculateAverage(m_errorHistory);
        float maxError = CalculateMax(m_errorHistory);
        float p95Error = CalculatePercentile(m_errorHistory, 95);

        // Record metrics
        m_metrics.Record("prediction_error_avg", avgError);
        m_metrics.Record("prediction_error_max", maxError);
        m_metrics.Record("prediction_error_p95", p95Error);
        m_metrics.Record("reconciliation_count", m_reconciliationCount);

        // Alert if prediction quality is poor
        if (avgError > ERROR_THRESHOLD) {
            LOG_WARNING("High prediction error: {} (threshold: {})",
                avgError, ERROR_THRESHOLD);

            // Could indicate:
            // - Client/server desync
            // - Cheating attempt
            // - Network issues
            // - Bug in prediction code
        }
    }

    void RecordError(float error) {
        m_errorHistory.push_back(error);
        if (m_errorHistory.size() > HISTORY_SIZE) {
            m_errorHistory.erase(m_errorHistory.begin());
        }

        if (error > RECONCILIATION_THRESHOLD) {
            m_reconciliationCount++;
        }
    }

private:
    static constexpr float ERROR_THRESHOLD = 1.0f;
    static constexpr float RECONCILIATION_THRESHOLD = 0.5f;
    static constexpr size_t HISTORY_SIZE = 100;
};
```

---

## 7. Lag Compensation

### 7.1 Server-Side Lag Compensation

```cpp
class LagCompensation {
public:
    HitResult ProcessHit(
        PlayerId attacker,
        const HitRequest& hit,
        uint64_t clientTick
    ) {
        // Get attacker's RTT
        float rtt = m_connections[attacker].smoothedRtt;

        // Calculate which server tick the client was seeing
        // Client saw state from: current_tick - (rtt / 2) / tick_duration
        uint64_t clientViewTick = CalculateClientViewTick(clientTick, rtt);

        // Get historical world state
        auto* historicState = m_stateBuffer.Get(clientViewTick);
        if (!historicState) {
            // State too old - use oldest available
            historicState = m_stateBuffer.GetOldest();
            if (!historicState) {
                return HitResult::Invalid("No historic state available");
            }
        }

        // Validate the hit against historic state
        auto& attackerState = historicState->GetPlayer(attacker);
        auto& targetState = historicState->GetPlayer(hit.targetId);

        // Verify attacker position matches (within tolerance)
        float posDiff = (attackerState.position - hit.attackerPosition).Length();
        if (posDiff > POSITION_TOLERANCE) {
            return HitResult::Invalid("Attacker position mismatch");
        }

        // Perform hit detection on historic state
        bool hitValid = PerformHitDetection(
            attackerState,
            targetState,
            hit
        );

        if (!hitValid) {
            return HitResult::Miss();
        }

        // Apply damage to current state (not historic)
        auto& currentTarget = m_world->GetPlayer(hit.targetId);
        float damage = CalculateDamage(hit, targetState);

        return HitResult::Hit(damage, hit.hitLocation);
    }

private:
    uint64_t CalculateClientViewTick(uint64_t clientTick, float rtt) {
        // Client tick represents when input was generated
        // They were seeing state from roughly rtt/2 ago

        float tickDuration = 1.0f / 20.0f;  // 50ms per tick
        int64_t ticksAgo = static_cast<int64_t>((rtt / 2.0f) / tickDuration);

        // Clamp to reasonable range
        ticksAgo = std::clamp(ticksAgo, 0L, MAX_REWIND_TICKS);

        return m_currentTick - ticksAgo;
    }

    bool PerformHitDetection(
        const PlayerState& attacker,
        const PlayerState& target,
        const HitRequest& hit
    ) {
        // Get target hitboxes at historic position
        auto hitboxes = GetPlayerHitboxes(target);

        // Create ray from attacker
        Ray ray(attacker.position + CAMERA_OFFSET, hit.direction);

        // Check intersection
        for (const auto& hitbox : hitboxes) {
            float distance;
            if (ray.Intersects(hitbox, distance)) {
                if (distance <= hit.maxRange) {
                    return true;
                }
            }
        }

        return false;
    }

    static constexpr int64_t MAX_REWIND_TICKS = 10;  // 500ms at 20Hz
    static constexpr float POSITION_TOLERANCE = 2.0f;  // 2 units
};
```

### 7.2 State Buffer

```cpp
class StateBuffer {
public:
    StateBuffer(size_t maxSize = 128) : m_maxSize(maxSize) {}

    void Store(uint64_t tick, WorldSnapshot snapshot) {
        m_buffer[tick] = std::move(snapshot);

        // Remove old snapshots
        while (m_buffer.size() > m_maxSize) {
            auto oldest = m_buffer.begin();
            m_buffer.erase(oldest);
        }
    }

    WorldSnapshot* Get(uint64_t tick) {
        auto it = m_buffer.find(tick);
        if (it != m_buffer.end()) {
            return &it->second;
        }
        return nullptr;
    }

    WorldSnapshot* GetClosest(uint64_t tick) {
        if (m_buffer.empty()) return nullptr;

        // Find closest tick
        auto it = m_buffer.lower_bound(tick);

        if (it == m_buffer.end()) {
            // All ticks are older - return newest
            return &m_buffer.rbegin()->second;
        }

        if (it == m_buffer.begin()) {
            return &it->second;
        }

        // Check if previous is closer
        auto prev = std::prev(it);
        if (tick - prev->first < it->first - tick) {
            return &prev->second;
        }

        return &it->second;
    }

    WorldSnapshot* GetOldest() {
        if (m_buffer.empty()) return nullptr;
        return &m_buffer.begin()->second;
    }

private:
    std::map<uint64_t, WorldSnapshot> m_buffer;
    size_t m_maxSize;
};
```

---

## 8. Delta Compression

### 8.1 Compression Strategy

```cpp
class DeltaCompressor {
public:
    std::vector<uint8_t> Compress(const StateUpdatePacket& packet) {
        BitWriter writer;

        // Write header
        writer.WriteBits(packet.serverTick - m_lastTick, 16);
        writer.WriteBits(packet.playerCount, 6);

        // Write player deltas
        for (const auto& player : packet.players) {
            CompressPlayerDelta(writer, player);
        }

        // Write entity count and deltas
        writer.WriteBits(packet.entityCount, 8);
        for (const auto& entity : packet.entities) {
            CompressEntityDelta(writer, entity);
        }

        // Write storm delta
        CompressStormDelta(writer, packet.storm);

        // Apply general compression (LZ4)
        return LZ4Compress(writer.GetData());
    }

    void CompressPlayerDelta(BitWriter& writer, const PlayerStateDelta& delta) {
        // Player ID (10 bits for up to 1024 players)
        writer.WriteBits(delta.playerId, 10);

        // Changed flags (8 bits)
        writer.WriteBits(delta.changedFlags, 8);

        // Position (if changed) - use variable encoding
        if (delta.changedFlags & CHANGED_POSITION) {
            // Delta from last known position
            int16_t dx = delta.posX - m_lastPlayerPos[delta.playerId].x;
            int16_t dy = delta.posY - m_lastPlayerPos[delta.playerId].y;
            int16_t dz = delta.posZ - m_lastPlayerPos[delta.playerId].z;

            WriteVariableInt(writer, dx);
            WriteVariableInt(writer, dy);
            WriteVariableInt(writer, dz);

            m_lastPlayerPos[delta.playerId] = {delta.posX, delta.posY, delta.posZ};
        }

        // Rotation (if changed) - 8 bits
        if (delta.changedFlags & CHANGED_ROTATION) {
            writer.WriteBits(delta.rotation, 8);
        }

        // Velocity (if changed) - 3x8 bits
        if (delta.changedFlags & CHANGED_VELOCITY) {
            writer.WriteBits(static_cast<uint8_t>(delta.velX + 128), 8);
            writer.WriteBits(static_cast<uint8_t>(delta.velY + 128), 8);
            writer.WriteBits(static_cast<uint8_t>(delta.velZ + 128), 8);
        }

        // Health (if changed) - 8 bits
        if (delta.changedFlags & CHANGED_HEALTH) {
            writer.WriteBits(delta.health, 8);
        }

        // Shield (if changed) - 8 bits
        if (delta.changedFlags & CHANGED_SHIELD) {
            writer.WriteBits(delta.shield, 8);
        }
    }

    void WriteVariableInt(BitWriter& writer, int16_t value) {
        // Use fewer bits for small deltas
        int16_t absValue = std::abs(value);

        if (absValue < 8) {
            writer.WriteBit(0);  // 4-bit encoding
            writer.WriteBits(value + 8, 4);
        } else if (absValue < 128) {
            writer.WriteBit(1);
            writer.WriteBit(0);  // 8-bit encoding
            writer.WriteBits(static_cast<uint8_t>(value + 128), 8);
        } else {
            writer.WriteBit(1);
            writer.WriteBit(1);  // 16-bit encoding
            writer.WriteBits(static_cast<uint16_t>(value + 32768), 16);
        }
    }
};
```

### 8.2 Compression Ratios

| Data Type | Uncompressed | Compressed | Ratio |
|-----------|--------------|------------|-------|
| Position (xyz) | 12 bytes | 2-6 bytes | 2-6x |
| Rotation | 4 bytes | 1 byte | 4x |
| Velocity | 12 bytes | 3 bytes | 4x |
| Health/Shield | 8 bytes | 2 bytes | 4x |
| Full Player | 48+ bytes | 8-16 bytes | 3-6x |
| 60 Players | 2880 bytes | ~600 bytes | ~5x |

---

## 9. Interest Management

### 9.1 Spatial Interest Management

```cpp
class InterestManager {
public:
    std::set<EntityId> GetRelevantEntities(
        const Vector3& viewerPosition,
        PlayerId viewerId
    ) {
        std::set<EntityId> relevant;

        // Always include self
        relevant.insert(viewerId);

        // Get entities in view distance
        auto nearbyEntities = m_spatialHash.QueryRadius(
            viewerPosition,
            VIEW_DISTANCE
        );

        for (auto entityId : nearbyEntities) {
            // Check visibility rules
            if (ShouldSync(viewerId, entityId, viewerPosition)) {
                relevant.insert(entityId);
            }
        }

        // Always include priority entities
        for (auto& priorityEntity : m_priorityEntities) {
            relevant.insert(priorityEntity);
        }

        return relevant;
    }

    bool ShouldSync(
        PlayerId viewer,
        EntityId entity,
        const Vector3& viewerPos
    ) {
        auto* entityState = m_world->GetEntity(entity);
        if (!entityState) return false;

        float distance = (entityState->position - viewerPos).Length();

        // Different sync distances by entity type
        float syncDistance = GetSyncDistance(entityState->type);

        if (distance > syncDistance) {
            return false;
        }

        // Check line of sight for some entities
        if (RequiresLineOfSight(entityState->type)) {
            if (!HasLineOfSight(viewerPos, entityState->position)) {
                // Still sync if recently visible (hysteresis)
                if (!WasRecentlyVisible(viewer, entity)) {
                    return false;
                }
            } else {
                MarkVisible(viewer, entity);
            }
        }

        return true;
    }

private:
    float GetSyncDistance(EntityType type) {
        switch (type) {
            case EntityType::Player:
                return PLAYER_SYNC_DISTANCE;
            case EntityType::Projectile:
                return PROJECTILE_SYNC_DISTANCE;
            case EntityType::Loot:
                return LOOT_SYNC_DISTANCE;
            case EntityType::NPC:
                return NPC_SYNC_DISTANCE;
            default:
                return DEFAULT_SYNC_DISTANCE;
        }
    }

    static constexpr float VIEW_DISTANCE = 200.0f;
    static constexpr float PLAYER_SYNC_DISTANCE = 150.0f;
    static constexpr float PROJECTILE_SYNC_DISTANCE = 100.0f;
    static constexpr float LOOT_SYNC_DISTANCE = 50.0f;
    static constexpr float NPC_SYNC_DISTANCE = 100.0f;
    static constexpr float DEFAULT_SYNC_DISTANCE = 100.0f;
};
```

### 9.2 Priority-Based Sync

```cpp
class PrioritizedSync {
public:
    std::vector<EntityId> GetSyncPriorities(
        PlayerId viewer,
        const std::set<EntityId>& relevantEntities
    ) {
        std::vector<std::pair<EntityId, float>> priorities;

        auto& viewerState = m_world->GetPlayer(viewer);

        for (auto entityId : relevantEntities) {
            float priority = CalculatePriority(viewerState, entityId);
            priorities.emplace_back(entityId, priority);
        }

        // Sort by priority (highest first)
        std::sort(priorities.begin(), priorities.end(),
            [](const auto& a, const auto& b) {
                return a.second > b.second;
            });

        // Return ordered entity list
        std::vector<EntityId> result;
        result.reserve(priorities.size());
        for (const auto& [id, _] : priorities) {
            result.push_back(id);
        }

        return result;
    }

    float CalculatePriority(
        const PlayerState& viewer,
        EntityId entityId
    ) {
        auto* entity = m_world->GetEntity(entityId);
        if (!entity) return 0.0f;

        float priority = 0.0f;

        // Distance factor (closer = higher priority)
        float distance = (entity->position - viewer.position).Length();
        priority += 100.0f / (distance + 1.0f);

        // Velocity factor (moving = higher priority)
        float velocity = entity->velocity.Length();
        priority += velocity * 2.0f;

        // Entity type factor
        switch (entity->type) {
            case EntityType::Player:
                priority += 50.0f;

                // Enemy players get higher priority
                if (!AreTeammates(viewer.id, entityId)) {
                    priority += 30.0f;
                }

                // Players in combat get higher priority
                if (IsInCombat(entityId)) {
                    priority += 20.0f;
                }
                break;

            case EntityType::Projectile:
                // Projectiles heading toward viewer get high priority
                if (IsHeadingToward(entity, viewer.position)) {
                    priority += 80.0f;
                } else {
                    priority += 30.0f;
                }
                break;

            case EntityType::Loot:
                priority += 10.0f;
                break;

            case EntityType::NPC:
                priority += 25.0f;
                if (IsAggroed(entityId, viewer.id)) {
                    priority += 40.0f;
                }
                break;
        }

        // Recently changed entities get priority
        if (WasRecentlyChanged(entityId)) {
            priority += 20.0f;
        }

        return priority;
    }
};
```

---

## 10. Mobile Network Optimization

### 10.1 Adaptive Sync Rate

```cpp
class AdaptiveSyncRate {
public:
    void UpdateSyncRate(PlayerConnection& conn) {
        // Measure connection quality
        ConnectionQuality quality = MeasureQuality(conn);

        // Adjust sync rate based on quality
        if (quality.packetLoss > 0.05f || quality.jitter > 50.0f) {
            // Poor connection - reduce sync rate
            conn.syncRate = std::max(conn.syncRate - 5, MIN_SYNC_RATE);
        } else if (quality.packetLoss < 0.01f && quality.jitter < 20.0f) {
            // Good connection - increase sync rate
            conn.syncRate = std::min(conn.syncRate + 5, MAX_SYNC_RATE);
        }

        // Adjust packet size
        if (quality.rtt > 100.0f) {
            // High latency - use smaller packets, more frequent
            conn.maxPacketSize = SMALL_PACKET_SIZE;
        } else {
            conn.maxPacketSize = NORMAL_PACKET_SIZE;
        }
    }

    ConnectionQuality MeasureQuality(const PlayerConnection& conn) {
        ConnectionQuality quality;

        // Packet loss
        quality.packetLoss = conn.stats.GetPacketLossRate();

        // Jitter (variation in latency)
        quality.jitter = conn.stats.GetJitter();

        // RTT
        quality.rtt = conn.smoothedRtt;

        // Bandwidth
        quality.bandwidth = conn.stats.GetAvailableBandwidth();

        return quality;
    }

private:
    static constexpr int MIN_SYNC_RATE = 10;   // 10 Hz minimum
    static constexpr int MAX_SYNC_RATE = 20;   // 20 Hz maximum
    static constexpr int SMALL_PACKET_SIZE = 512;
    static constexpr int NORMAL_PACKET_SIZE = 1200;
};
```

### 10.2 Network Type Detection

```cpp
class NetworkTypeDetector {
public:
    NetworkType DetectNetworkType() {
        #if PLATFORM_IOS
        return DetectNetworkType_iOS();
        #elif PLATFORM_ANDROID
        return DetectNetworkType_Android();
        #endif
    }

    NetworkConfig GetConfigForNetworkType(NetworkType type) {
        NetworkConfig config;

        switch (type) {
            case NetworkType::WiFi:
                config.syncRate = 20;
                config.maxBandwidth = 100000;  // 100 KB/s
                config.inputRedundancy = 1;
                config.interpolationDelay = 0.05f;
                break;

            case NetworkType::LTE:
                config.syncRate = 20;
                config.maxBandwidth = 50000;   // 50 KB/s
                config.inputRedundancy = 2;
                config.interpolationDelay = 0.08f;
                break;

            case NetworkType::FourG:
                config.syncRate = 15;
                config.maxBandwidth = 30000;   // 30 KB/s
                config.inputRedundancy = 2;
                config.interpolationDelay = 0.1f;
                break;

            case NetworkType::ThreeG:
                config.syncRate = 10;
                config.maxBandwidth = 15000;   // 15 KB/s
                config.inputRedundancy = 3;
                config.interpolationDelay = 0.15f;
                break;

            default:
                // Conservative defaults
                config.syncRate = 10;
                config.maxBandwidth = 10000;
                config.inputRedundancy = 3;
                config.interpolationDelay = 0.15f;
        }

        return config;
    }

private:
    #if PLATFORM_IOS
    NetworkType DetectNetworkType_iOS() {
        // Use NWPathMonitor on iOS
        // ...
    }
    #endif

    #if PLATFORM_ANDROID
    NetworkType DetectNetworkType_Android() {
        // Use ConnectivityManager on Android
        // ...
    }
    #endif
};
```

### 10.3 Background/Foreground Handling

```cpp
class AppLifecycleHandler {
public:
    void OnAppBackground() {
        // App going to background
        m_isBackground = true;

        // Reduce network activity
        m_networkManager->SetBackgroundMode(true);

        // Send background notification to server
        SendBackgroundNotification();

        // Keep minimal connection alive
        StartBackgroundHeartbeat();
    }

    void OnAppForeground() {
        // App coming to foreground
        m_isBackground = false;

        // Restore full network activity
        m_networkManager->SetBackgroundMode(false);

        // Request full state sync
        RequestFullStateSync();

        // Stop background heartbeat
        StopBackgroundHeartbeat();
    }

    void StartBackgroundHeartbeat() {
        // Send heartbeat every 5 seconds in background
        // Just enough to maintain connection
        m_backgroundHeartbeat = ScheduleRepeating(5.0f, [this]() {
            SendHeartbeat();
        });
    }

    void RequestFullStateSync() {
        // After returning to foreground, request full state
        // to resync after potentially missed updates
        auto request = CreateFullStateRequest();
        m_networkManager->SendReliable(request);

        // Server will send full state snapshot
        // Client will reconcile predictions
    }
};
```

---

## 11. Packet Loss Handling

### 11.1 Input Redundancy

```cpp
class InputRedundancy {
public:
    InputPacket CreateInputPacket(const PlayerInput& currentInput) {
        InputPacket packet;
        packet.inputCount = m_redundancyLevel;

        // Include current input
        packet.inputs[0] = currentInput;

        // Include previous inputs for redundancy
        for (int i = 1; i < m_redundancyLevel; i++) {
            auto prevInput = m_inputHistory.Get(currentInput.tick - i);
            if (prevInput) {
                packet.inputs[i] = *prevInput;
            }
        }

        return packet;
    }

    void SetRedundancyLevel(int level) {
        // 1 = no redundancy
        // 2 = current + 1 previous
        // 3 = current + 2 previous
        m_redundancyLevel = std::clamp(level, 1, MAX_REDUNDANCY);
    }

    void AdjustRedundancy(float packetLoss) {
        if (packetLoss > 0.05f) {
            // High packet loss - increase redundancy
            SetRedundancyLevel(3);
        } else if (packetLoss > 0.02f) {
            SetRedundancyLevel(2);
        } else {
            SetRedundancyLevel(1);
        }
    }

private:
    int m_redundancyLevel = 2;
    static constexpr int MAX_REDUNDANCY = 3;
};
```

### 11.2 State Interpolation

```cpp
class StateInterpolation {
public:
    PlayerState InterpolatePlayer(
        PlayerId playerId,
        float renderTime
    ) {
        // Get snapshots bracketing render time
        auto* before = m_snapshotBuffer.GetBefore(renderTime);
        auto* after = m_snapshotBuffer.GetAfter(renderTime);

        // Handle edge cases
        if (!before && !after) {
            // No data - return last known state
            return GetLastKnownState(playerId);
        }

        if (!after) {
            // No future data - extrapolate
            return ExtrapolateState(playerId, before, renderTime);
        }

        if (!before) {
            // No past data - use earliest
            return after->GetPlayer(playerId);
        }

        // Normal interpolation
        float t = (renderTime - before->time) / (after->time - before->time);
        t = std::clamp(t, 0.0f, 1.0f);

        return InterpolateBetween(
            before->GetPlayer(playerId),
            after->GetPlayer(playerId),
            t
        );
    }

    PlayerState ExtrapolateState(
        PlayerId playerId,
        const WorldSnapshot* lastSnapshot,
        float targetTime
    ) {
        auto& lastState = lastSnapshot->GetPlayer(playerId);
        float dt = targetTime - lastSnapshot->time;

        // Limit extrapolation time
        dt = std::min(dt, MAX_EXTRAPOLATION_TIME);

        PlayerState extrapolated = lastState;

        // Extrapolate position based on velocity
        extrapolated.position += lastState.velocity * dt;

        // Don't extrapolate rotation (too error-prone)
        // Don't extrapolate health/shield

        return extrapolated;
    }

    PlayerState InterpolateBetween(
        const PlayerState& from,
        const PlayerState& to,
        float t
    ) {
        PlayerState result;

        // Interpolate position
        result.position = Vector3::Lerp(from.position, to.position, t);

        // Interpolate rotation (shortest path)
        result.rotation = LerpAngle(from.rotation, to.rotation, t);

        // Interpolate velocity
        result.velocity = Vector3::Lerp(from.velocity, to.velocity, t);

        // Snap health/shield (don't interpolate)
        result.health = t < 0.5f ? from.health : to.health;
        result.shield = t < 0.5f ? from.shield : to.shield;

        return result;
    }

private:
    static constexpr float MAX_EXTRAPOLATION_TIME = 0.2f;  // 200ms
};
```

### 11.3 Jitter Buffer

```cpp
class JitterBuffer {
public:
    JitterBuffer(float targetDelay = 0.1f)
        : m_targetDelay(targetDelay) {}

    void AddSnapshot(const WorldSnapshot& snapshot, float receiveTime) {
        // Calculate actual delay
        float delay = receiveTime - snapshot.time;

        // Update jitter measurement
        float jitter = std::abs(delay - m_lastDelay);
        m_smoothedJitter = m_smoothedJitter * 0.9f + jitter * 0.1f;
        m_lastDelay = delay;

        // Add to buffer
        m_buffer.push_back({snapshot, receiveTime});

        // Sort by snapshot time
        std::sort(m_buffer.begin(), m_buffer.end(),
            [](const auto& a, const auto& b) {
                return a.snapshot.time < b.snapshot.time;
            });

        // Remove old snapshots
        RemoveOldSnapshots();

        // Adapt target delay based on jitter
        AdaptTargetDelay();
    }

    float GetRenderTime(float currentTime) {
        // Render time = current time - buffer delay
        return currentTime - m_currentDelay;
    }

private:
    void AdaptTargetDelay() {
        // Increase delay if jitter is high
        float desiredDelay = m_targetDelay + m_smoothedJitter * 2.0f;

        // Smoothly adjust current delay
        float maxAdjustment = 0.01f;  // 10ms per frame max
        if (desiredDelay > m_currentDelay) {
            m_currentDelay = std::min(
                m_currentDelay + maxAdjustment,
                desiredDelay
            );
        } else {
            m_currentDelay = std::max(
                m_currentDelay - maxAdjustment * 0.5f,
                desiredDelay
            );
        }

        // Clamp to reasonable range
        m_currentDelay = std::clamp(m_currentDelay, MIN_DELAY, MAX_DELAY);
    }

    float m_targetDelay;
    float m_currentDelay;
    float m_smoothedJitter = 0.0f;
    float m_lastDelay = 0.0f;

    static constexpr float MIN_DELAY = 0.05f;   // 50ms minimum
    static constexpr float MAX_DELAY = 0.25f;   // 250ms maximum
};
```

---

## 12. Bandwidth Management

### 12.1 Bandwidth Limiter

```cpp
class BandwidthLimiter {
public:
    BandwidthLimiter(int maxBytesPerSecond)
        : m_maxBytesPerSecond(maxBytesPerSecond) {}

    bool CanSend(int bytes) {
        UpdateBudget();
        return m_budget >= bytes;
    }

    void RecordSend(int bytes) {
        m_budget -= bytes;
        m_bytesSent += bytes;
    }

    int GetAvailableBudget() {
        UpdateBudget();
        return std::max(0, m_budget);
    }

    void SetMaxBandwidth(int bytesPerSecond) {
        m_maxBytesPerSecond = bytesPerSecond;
    }

private:
    void UpdateBudget() {
        auto now = Clock::Now();
        float dt = Clock::DurationSeconds(m_lastUpdate, now);
        m_lastUpdate = now;

        // Add budget based on time passed
        m_budget += static_cast<int>(dt * m_maxBytesPerSecond);

        // Cap at max (allow 100ms burst)
        int maxBudget = m_maxBytesPerSecond / 10;
        m_budget = std::min(m_budget, maxBudget);
    }

    int m_maxBytesPerSecond;
    int m_budget = 0;
    int m_bytesSent = 0;
    TimePoint m_lastUpdate;
};
```

### 12.2 Bandwidth Statistics

```cpp
class BandwidthStats {
public:
    void RecordSend(int bytes) {
        m_sendWindow.Add(bytes);
        m_totalSent += bytes;
    }

    void RecordReceive(int bytes) {
        m_receiveWindow.Add(bytes);
        m_totalReceived += bytes;
    }

    float GetSendRate() {
        return m_sendWindow.GetRate();  // bytes/second
    }

    float GetReceiveRate() {
        return m_receiveWindow.GetRate();
    }

    BandwidthReport GetReport() {
        return {
            .sendRateBps = GetSendRate(),
            .receiveRateBps = GetReceiveRate(),
            .totalSent = m_totalSent,
            .totalReceived = m_totalReceived,
            .averagePacketSize = m_sendWindow.GetAverageSize()
        };
    }

private:
    SlidingWindow m_sendWindow{1.0f};     // 1 second window
    SlidingWindow m_receiveWindow{1.0f};
    uint64_t m_totalSent = 0;
    uint64_t m_totalReceived = 0;
};

class SlidingWindow {
public:
    SlidingWindow(float windowSeconds) : m_windowSeconds(windowSeconds) {}

    void Add(int bytes) {
        m_samples.push_back({Clock::Now(), bytes});
        Cleanup();
    }

    float GetRate() {
        Cleanup();
        int totalBytes = 0;
        for (const auto& sample : m_samples) {
            totalBytes += sample.bytes;
        }
        return totalBytes / m_windowSeconds;
    }

private:
    void Cleanup() {
        auto cutoff = Clock::Now() - m_windowSeconds;
        m_samples.erase(
            std::remove_if(m_samples.begin(), m_samples.end(),
                [cutoff](const auto& s) { return s.time < cutoff; }),
            m_samples.end()
        );
    }

    float m_windowSeconds;
    std::vector<Sample> m_samples;
};
```

---

## 13. Performance Requirements

### 13.1 Benchmarks

| Scenario | Players | Tick Rate | Bandwidth/Player | CPU/Server |
|----------|---------|-----------|------------------|------------|
| Normal | 60 | 20 Hz | 25 KB/s | < 50% |
| Combat Heavy | 60 | 20 Hz | 35 KB/s | < 70% |
| End Game | 20 | 20 Hz | 40 KB/s | < 40% |
| Worst Case | 60 | 20 Hz | 50 KB/s | < 80% |

### 13.2 Latency Budget

| Component | Budget | Notes |
|-----------|--------|-------|
| Input Processing | < 5ms | Client-side |
| Network Transit | Variable | ~50ms typical |
| Server Processing | < 20ms | Per tick |
| State Sync | < 10ms | Serialization |
| Client Receive | < 5ms | Deserialization |
| Reconciliation | < 5ms | When needed |
| Interpolation | < 2ms | Per frame |
| **Total (visible)** | **< 50ms** | With prediction |

### 13.3 Testing Requirements

| Test | Description | Success Criteria |
|------|-------------|------------------|
| Latency Test | Simulate 50-200ms RTT | No visible lag with prediction |
| Packet Loss | 5% packet loss | Smooth gameplay |
| Jitter | ±50ms jitter | No rubberbanding |
| Bandwidth | Limit to 30KB/s | Full gameplay possible |
| Player Count | 60 concurrent players | 20Hz maintained |
| Combat | 60 players all fighting | Fair hit detection |
| Reconnect | Drop and reconnect | < 5s reconnect time |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Network Lead | _____________ | _____________ | ___/___/___ |
| Backend Lead | _____________ | _____________ | ___/___/___ |
| Game Director | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial real-time netcode specification |

---

*This document contains implementation details. Handle according to company security policies.*
