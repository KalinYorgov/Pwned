# Dedicated Game Servers Specification - Plunderstorm Mobile

**Document ID:** BACK-004
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Server Architecture](#2-server-architecture)
3. [Game State Management](#3-game-state-management)
4. [Player Systems](#4-player-systems)
5. [Combat Systems](#5-combat-systems)
6. [NPC and AI Systems](#6-npc-and-ai-systems)
7. [World Systems](#7-world-systems)
8. [Anti-Cheat Validation](#8-anti-cheat-validation)
9. [Server Lifecycle](#9-server-lifecycle)
10. [Deployment Architecture](#10-deployment-architecture)
11. [Performance Requirements](#11-performance-requirements)
12. [Monitoring and Logging](#12-monitoring-and-logging)
13. [Cloud Provider Integration](#13-cloud-provider-integration)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the dedicated game server infrastructure for Plunderstorm Mobile, providing authoritative server-side game simulation for 60-player battle royale matches with full anti-cheat capabilities.

### 1.2 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Authority | Server is sole source of truth for game state | Critical |
| Performance | 60 players at 20+ tick rate | Critical |
| Stability | Zero crashes under normal operation | Critical |
| Scalability | Support 100+ concurrent matches per region | High |
| Efficiency | Multiple servers per VM for cost optimization | High |
| Security | Prevent all forms of cheating | Critical |

### 1.3 Server Responsibilities Overview

| System | Responsibility | Validation Level |
|--------|---------------|------------------|
| Game State | Authoritative world state | Full |
| Player Movement | Position validation | Full |
| Combat | Hit detection, damage | Full |
| Loot | Spawn and acquisition | Full |
| Abilities | Cooldowns, effects | Full |
| NPCs | AI behavior, combat | Full |
| Storm | Zone management | Full |
| Match Flow | Start, end, victory | Full |

### 1.4 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         GAME SERVER VM                               │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Game Server  │  │ Game Server  │  │ Game Server  │  ...         │
│  │ Instance 1   │  │ Instance 2   │  │ Instance 3   │              │
│  │ (Match A)    │  │ (Match B)    │  │ (Match C)    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                 │                       │
│  ┌──────▼─────────────────▼─────────────────▼──────┐               │
│  │              Server Manager Process              │               │
│  │  • Instance lifecycle  • Health monitoring      │               │
│  │  • Resource allocation • Log aggregation        │               │
│  └──────────────────────┬──────────────────────────┘               │
│                         │                                           │
└─────────────────────────┼───────────────────────────────────────────┘
                          │
            ┌─────────────▼─────────────┐
            │    Orchestration Service   │
            │  • Server allocation       │
            │  • Fleet management        │
            │  • Auto-scaling            │
            └───────────────────────────┘
```

---

## 2. Server Architecture

### 2.1 Server Process Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      GAME SERVER PROCESS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                      Main Game Loop                          │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │   │
│  │  │ Input   │─▶│ Update  │─▶│ Physics │─▶│ Output  │        │   │
│  │  │ Process │  │ Systems │  │ Step    │  │ Sync    │        │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Network    │  │    Game      │  │   World      │              │
│  │   Layer      │  │    Systems   │  │   State      │              │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤              │
│  │ • UDP Socket │  │ • Combat     │  │ • Entities   │              │
│  │ • Packet     │  │ • Movement   │  │ • Terrain    │              │
│  │   Handler    │  │ • Abilities  │  │ • Loot       │              │
│  │ • Connection │  │ • Inventory  │  │ • NPCs       │              │
│  │   Manager    │  │ • Scoring    │  │ • Storm      │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │  Validation  │  │     AI       │  │   Logging    │              │
│  │   Layer      │  │   System     │  │   System     │              │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤              │
│  │ • Movement   │  │ • NPC Brain  │  │ • Events     │              │
│  │ • Combat     │  │ • Pathfind   │  │ • Metrics    │              │
│  │ • Actions    │  │ • Behavior   │  │ • Replay     │              │
│  │ • Rate Limit │  │   Trees      │  │   Data       │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Core Server Components

```cpp
// Game Server Main Class
class GameServer {
public:
    // Lifecycle
    bool Initialize(const ServerConfig& config);
    void Run();
    void Shutdown();

    // Match management
    void StartMatch(const MatchConfig& config);
    void EndMatch(MatchEndReason reason);

    // Tick
    void Tick(float deltaTime);

private:
    // Core systems
    NetworkManager* m_network;
    GameStateManager* m_gameState;
    PlayerManager* m_players;
    CombatSystem* m_combat;
    AbilitySystem* m_abilities;
    LootSystem* m_loot;
    StormSystem* m_storm;
    AISystem* m_ai;
    ValidationSystem* m_validation;
    ReplaySystem* m_replay;

    // Server state
    ServerState m_state;
    MatchPhase m_matchPhase;
    uint64_t m_tickCount;
    float m_tickRate;
};

// Server Configuration
struct ServerConfig {
    uint16_t port;
    uint32_t maxPlayers = 60;
    float tickRate = 20.0f;          // Ticks per second
    float maxTickTime = 50.0f;       // Max ms per tick
    std::string region;
    std::string matchId;
    LogLevel logLevel;
};
```

### 2.3 Game Loop Implementation

```cpp
void GameServer::Run() {
    const float targetTickTime = 1.0f / m_tickRate;  // 50ms at 20Hz
    auto lastTickTime = Clock::Now();

    while (m_state == ServerState::Running) {
        auto tickStart = Clock::Now();
        float deltaTime = Clock::DurationSeconds(lastTickTime, tickStart);
        lastTickTime = tickStart;

        // Process incoming network messages
        m_network->ProcessIncoming();

        // Update all game systems
        Tick(deltaTime);

        // Send state updates to clients
        m_network->SendStateUpdates();

        // Calculate remaining time in tick
        auto tickEnd = Clock::Now();
        float tickDuration = Clock::DurationMs(tickStart, tickEnd);

        // Log if tick took too long
        if (tickDuration > m_config.maxTickTime) {
            LOG_WARNING("Tick {} took {}ms (target: {}ms)",
                m_tickCount, tickDuration, targetTickTime * 1000);
            m_metrics.RecordSlowTick(tickDuration);
        }

        // Sleep for remaining tick time
        float sleepTime = (targetTickTime * 1000) - tickDuration;
        if (sleepTime > 0) {
            Thread::SleepMs(sleepTime);
        }

        m_tickCount++;
    }
}

void GameServer::Tick(float deltaTime) {
    // Update match state
    UpdateMatchPhase(deltaTime);

    // Process player inputs
    m_players->ProcessInputs();

    // Validate and apply player movements
    m_players->UpdateMovement(deltaTime);
    m_validation->ValidateMovements();

    // Update abilities
    m_abilities->Update(deltaTime);

    // Process combat
    m_combat->ProcessCombat(deltaTime);

    // Update NPCs
    m_ai->Update(deltaTime);

    // Update storm
    m_storm->Update(deltaTime);

    // Update loot
    m_loot->Update(deltaTime);

    // Check victory conditions
    CheckMatchEnd();

    // Record replay data
    m_replay->RecordFrame(m_tickCount);
}
```

---

## 3. Game State Management

### 3.1 World State Structure

```cpp
// Complete world state at any tick
struct WorldState {
    uint64_t tick;
    float matchTime;
    MatchPhase phase;

    // Entity containers
    std::unordered_map<PlayerId, PlayerState> players;
    std::unordered_map<EntityId, NPCState> npcs;
    std::unordered_map<EntityId, LootState> loot;
    std::unordered_map<EntityId, ProjectileState> projectiles;

    // World state
    StormState storm;
    std::vector<ZoneEvent> activeEvents;

    // Match data
    uint32_t playersAlive;
    uint32_t totalKills;
    LeaderboardState leaderboard;
};

// Player state (server-authoritative)
struct PlayerState {
    PlayerId id;
    std::string username;
    TeamId team;

    // Transform
    Vector3 position;
    Vector3 velocity;
    float rotation;

    // Combat
    float health;
    float maxHealth;
    float shield;
    float maxShield;
    bool isAlive;
    bool isKnocked;

    // Abilities
    std::array<AbilityState, 4> abilities;

    // Inventory
    std::vector<ItemState> inventory;
    LoadoutState loadout;

    // Status effects
    std::vector<StatusEffect> effects;

    // Input state (last received)
    PlayerInput lastInput;
    uint64_t lastInputTick;

    // Connection state
    ConnectionState connection;
    float ping;
};
```

### 3.2 State Synchronization

```cpp
class StateSyncManager {
public:
    // Full state sync (on connect/reconnect)
    void SendFullState(PlayerId player);

    // Delta state sync (every tick)
    void SendDeltaState();

    // Priority-based sync
    void SendPriorityUpdates(PlayerId player);

private:
    // Interest management - only sync nearby entities
    std::vector<EntityId> GetRelevantEntities(const PlayerState& player);

    // Delta compression
    DeltaPacket ComputeDelta(const WorldState& prev, const WorldState& current);

    // Snapshot history for lag compensation
    CircularBuffer<WorldState, 64> m_stateHistory;
};

// Delta sync packet structure
struct DeltaPacket {
    uint64_t baseTick;
    uint64_t currentTick;

    // Changed entities only
    std::vector<PlayerDelta> playerDeltas;
    std::vector<NPCDelta> npcDeltas;
    std::vector<LootDelta> lootDeltas;
    std::vector<ProjectileDelta> projectileDeltas;

    // World changes
    std::optional<StormDelta> stormDelta;
    std::vector<EventNotification> events;
};

// Interest management zones
class InterestManager {
public:
    void UpdatePlayerZone(PlayerId player, Vector3 position);
    std::vector<EntityId> GetVisibleEntities(PlayerId player);

private:
    // Spatial partitioning
    SpatialHash<EntityId> m_entityGrid;

    // View distance by entity type
    static constexpr float PLAYER_VIEW_DISTANCE = 150.0f;
    static constexpr float LOOT_VIEW_DISTANCE = 50.0f;
    static constexpr float PROJECTILE_VIEW_DISTANCE = 100.0f;
};
```

### 3.3 State Persistence

```cpp
class MatchStateManager {
public:
    // Save match state for recovery
    void SaveCheckpoint();

    // Restore from crash
    bool RestoreFromCheckpoint(const std::string& matchId);

    // End-of-match persistence
    void SaveMatchResults(const MatchResults& results);

private:
    // Checkpoint frequency
    static constexpr float CHECKPOINT_INTERVAL = 30.0f;  // seconds

    // State serialization
    std::vector<uint8_t> SerializeState(const WorldState& state);
    WorldState DeserializeState(const std::vector<uint8_t>& data);

    // Storage backend
    std::unique_ptr<StateStorage> m_storage;
};
```

---

## 4. Player Systems

### 4.1 Player Connection Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                   PLAYER CONNECTION FLOW                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Client connects to server                                     │
│     • TCP handshake → UDP channel established                     │
│          │                                                        │
│          ▼                                                        │
│  2. Authentication verification                                   │
│     • Verify token with auth service                              │
│     • Check player is assigned to this match                      │
│          │                                                        │
│          ▼                                                        │
│  3. Player state initialization                                   │
│     • Create PlayerState object                                   │
│     • Assign spawn position (waiting area)                        │
│     • Load player loadout/cosmetics                               │
│          │                                                        │
│          ▼                                                        │
│  4. Full state sync                                               │
│     • Send current world state                                    │
│     • Send other player states                                    │
│          │                                                        │
│          ▼                                                        │
│  5. Ready for play                                                │
│     • Client confirms ready                                       │
│     • Start receiving delta updates                               │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 4.2 Player Input Processing

```cpp
// Client input packet
struct PlayerInput {
    uint64_t tick;                    // Client tick when input was generated
    uint32_t sequenceNumber;          // For ordering
    float timestamp;                  // Client timestamp

    // Movement
    Vector2 moveDirection;            // Normalized (-1 to 1)
    float lookRotation;               // Degrees
    bool jump;
    bool sprint;

    // Combat
    bool primaryFire;
    bool secondaryFire;
    Vector3 aimTarget;                // World position being aimed at

    // Abilities
    int8_t abilitySlot;              // -1 = none, 0-3 = ability slot
    Vector3 abilityTarget;           // Target position/direction

    // Interactions
    bool interact;
    EntityId interactTarget;
};

class PlayerInputProcessor {
public:
    void ProcessInput(PlayerId player, const PlayerInput& input) {
        auto& state = m_players->GetState(player);

        // Validate input timing
        if (!ValidateInputTiming(state, input)) {
            LOG_WARNING("Invalid input timing from player {}", player);
            return;
        }

        // Validate input values
        if (!ValidateInputValues(input)) {
            LOG_WARNING("Invalid input values from player {}", player);
            m_validation->FlagPlayer(player, "invalid_input");
            return;
        }

        // Store input for processing
        state.inputQueue.Push(input);

        // Update last input tick
        state.lastInputTick = input.tick;
        state.lastInputTime = Clock::Now();
    }

private:
    bool ValidateInputTiming(const PlayerState& state, const PlayerInput& input) {
        // Check sequence number
        if (input.sequenceNumber <= state.lastSequenceNumber) {
            return false;  // Old/duplicate input
        }

        // Check tick is reasonable
        int64_t tickDiff = input.tick - m_currentTick;
        if (tickDiff > MAX_FUTURE_TICKS || tickDiff < -MAX_PAST_TICKS) {
            return false;  // Too far in future/past
        }

        return true;
    }

    bool ValidateInputValues(const PlayerInput& input) {
        // Validate move direction is normalized
        if (input.moveDirection.Length() > 1.1f) {
            return false;
        }

        // Validate rotation is in valid range
        if (input.lookRotation < 0 || input.lookRotation >= 360) {
            return false;
        }

        // Validate ability slot
        if (input.abilitySlot < -1 || input.abilitySlot > 3) {
            return false;
        }

        return true;
    }

    static constexpr int MAX_FUTURE_TICKS = 5;
    static constexpr int MAX_PAST_TICKS = 20;
};
```

### 4.3 Movement Validation

```cpp
class MovementValidator {
public:
    ValidationResult ValidateMovement(
        const PlayerState& prevState,
        const PlayerState& newState,
        const PlayerInput& input,
        float deltaTime
    ) {
        ValidationResult result;

        // Calculate expected position based on input
        Vector3 expectedPos = CalculateExpectedPosition(
            prevState, input, deltaTime
        );

        // Check distance traveled
        float distance = (newState.position - prevState.position).Length();
        float maxDistance = CalculateMaxDistance(prevState, deltaTime);

        if (distance > maxDistance * SPEED_TOLERANCE) {
            result.AddViolation(ValidationType::SpeedHack,
                "Moved {} units in {} seconds (max: {})",
                distance, deltaTime, maxDistance);
        }

        // Check vertical movement
        if (!ValidateVerticalMovement(prevState, newState, input)) {
            result.AddViolation(ValidationType::FlyHack,
                "Invalid vertical movement detected");
        }

        // Check terrain collision
        if (!ValidateTerrainCollision(newState.position)) {
            result.AddViolation(ValidationType::NoClip,
                "Player inside terrain at {}", newState.position);
        }

        // Check bounds
        if (!IsWithinMapBounds(newState.position)) {
            result.AddViolation(ValidationType::OutOfBounds,
                "Player outside map bounds at {}", newState.position);
        }

        return result;
    }

private:
    float CalculateMaxDistance(const PlayerState& state, float deltaTime) {
        float baseSpeed = PLAYER_BASE_SPEED;

        // Apply sprint modifier
        if (state.isSprinting) {
            baseSpeed *= SPRINT_MULTIPLIER;
        }

        // Apply status effects
        for (const auto& effect : state.effects) {
            if (effect.type == EffectType::SpeedBoost) {
                baseSpeed *= effect.magnitude;
            }
            if (effect.type == EffectType::Slow) {
                baseSpeed *= effect.magnitude;
            }
        }

        return baseSpeed * deltaTime;
    }

    static constexpr float PLAYER_BASE_SPEED = 8.0f;      // units/second
    static constexpr float SPRINT_MULTIPLIER = 1.5f;
    static constexpr float SPEED_TOLERANCE = 1.15f;       // 15% tolerance
};
```

### 4.4 Player Elimination System

```cpp
class EliminationSystem {
public:
    void ProcessElimination(PlayerId victim, PlayerId killer, DamageSource source) {
        auto& victimState = m_players->GetState(victim);
        auto& killerState = m_players->GetState(killer);

        // Update victim state
        victimState.isAlive = false;
        victimState.health = 0;
        victimState.eliminatedAt = m_currentTime;
        victimState.eliminatedBy = killer;
        victimState.placement = m_playersAlive;

        // Update killer stats
        killerState.kills++;
        killerState.matchKills++;

        // Award XP/resources to killer
        AwardKillRewards(killer, victim, source);

        // Drop victim's loot
        m_loot->SpawnDeathLoot(victimState);

        // Update alive count
        m_playersAlive--;

        // Broadcast elimination event
        BroadcastEvent(EliminationEvent{
            .victim = victim,
            .victimName = victimState.username,
            .killer = killer,
            .killerName = killerState.username,
            .source = source,
            .playersRemaining = m_playersAlive
        });

        // Record for replay
        m_replay->RecordElimination(victim, killer, source);

        // Check for victory
        CheckVictoryConditions();
    }

private:
    void CheckVictoryConditions() {
        if (m_matchPhase != MatchPhase::InProgress) return;

        // Solo mode - 1 player remaining
        if (m_gameMode == GameMode::Solo && m_playersAlive <= 1) {
            EndMatch(FindLastAlivePlayer());
        }

        // Team modes - 1 team remaining
        if (m_gameMode == GameMode::Duo || m_gameMode == GameMode::Squad) {
            auto aliveTeams = GetAliveTeams();
            if (aliveTeams.size() <= 1) {
                EndMatch(aliveTeams.empty() ? TeamId::None : aliveTeams[0]);
            }
        }
    }
};
```

---

## 5. Combat Systems

### 5.1 Hit Detection System

```cpp
class HitDetectionSystem {
public:
    HitResult ProcessAttack(
        PlayerId attacker,
        const AttackData& attack,
        uint64_t clientTick
    ) {
        // Get attacker state
        auto& attackerState = m_players->GetState(attacker);

        // Lag compensation - rewind world state
        const WorldState& historicState = m_stateHistory.GetStateAtTick(clientTick);

        // Validate attack is possible
        if (!ValidateAttack(attackerState, attack)) {
            return HitResult::Invalid();
        }

        // Perform hit detection on historic state
        std::vector<HitInfo> hits;

        switch (attack.type) {
            case AttackType::Hitscan:
                hits = ProcessHitscan(attackerState, attack, historicState);
                break;

            case AttackType::Projectile:
                // Projectiles are spawned and tracked
                SpawnProjectile(attackerState, attack);
                return HitResult::Pending();

            case AttackType::Melee:
                hits = ProcessMelee(attackerState, attack, historicState);
                break;

            case AttackType::Area:
                hits = ProcessAreaAttack(attackerState, attack, historicState);
                break;
        }

        // Process hits
        HitResult result;
        for (const auto& hit : hits) {
            DamageResult dmg = ApplyDamage(hit, attackerState);
            result.AddHit(hit, dmg);
        }

        return result;
    }

private:
    std::vector<HitInfo> ProcessHitscan(
        const PlayerState& attacker,
        const AttackData& attack,
        const WorldState& state
    ) {
        std::vector<HitInfo> hits;

        // Create ray from attacker
        Ray ray(attacker.position + CAMERA_OFFSET, attack.direction);

        // Check against all valid targets in historic state
        for (const auto& [id, target] : state.players) {
            if (id == attacker.id) continue;  // Can't hit self
            if (!target.isAlive) continue;
            if (AreTeammates(attacker.id, id)) continue;

            // Get hitboxes for target
            auto hitboxes = GetPlayerHitboxes(target);

            for (const auto& hitbox : hitboxes) {
                float distance;
                if (ray.Intersects(hitbox, distance)) {
                    if (distance <= attack.range) {
                        hits.push_back(HitInfo{
                            .target = id,
                            .hitbox = hitbox.type,
                            .distance = distance,
                            .position = ray.GetPoint(distance)
                        });
                        break;  // One hit per player
                    }
                }
            }
        }

        return hits;
    }
};
```

### 5.2 Damage System

```cpp
class DamageSystem {
public:
    DamageResult ApplyDamage(
        const HitInfo& hit,
        const PlayerState& attacker,
        float baseDamage
    ) {
        auto& target = m_players->GetState(hit.target);
        DamageResult result;

        // Calculate damage
        float damage = baseDamage;

        // Headshot multiplier
        if (hit.hitbox == HitboxType::Head) {
            damage *= HEADSHOT_MULTIPLIER;
            result.isHeadshot = true;
        }

        // Distance falloff
        damage *= CalculateFalloff(hit.distance);

        // Apply attacker modifiers
        damage *= attacker.damageMultiplier;

        // Apply target damage reduction
        damage *= (1.0f - target.damageReduction);

        result.rawDamage = damage;

        // Apply to shield first
        if (target.shield > 0) {
            float shieldDamage = std::min(damage, target.shield);
            target.shield -= shieldDamage;
            result.shieldDamage = shieldDamage;
            damage -= shieldDamage;
        }

        // Apply remaining to health
        if (damage > 0) {
            target.health -= damage;
            result.healthDamage = damage;
        }

        // Track damage for stats
        m_stats->RecordDamage(attacker.id, hit.target, result);

        // Check for elimination
        if (target.health <= 0) {
            ProcessElimination(hit.target, attacker.id, DamageSource::Combat);
            result.isElimination = true;
        }

        // Broadcast damage event
        BroadcastDamageEvent(hit, result);

        return result;
    }

private:
    float CalculateFalloff(float distance) {
        if (distance <= FALLOFF_START) {
            return 1.0f;
        }
        if (distance >= FALLOFF_END) {
            return MIN_FALLOFF_DAMAGE;
        }

        float t = (distance - FALLOFF_START) / (FALLOFF_END - FALLOFF_START);
        return 1.0f - (t * (1.0f - MIN_FALLOFF_DAMAGE));
    }

    static constexpr float HEADSHOT_MULTIPLIER = 2.0f;
    static constexpr float FALLOFF_START = 30.0f;
    static constexpr float FALLOFF_END = 100.0f;
    static constexpr float MIN_FALLOFF_DAMAGE = 0.5f;
};
```

### 5.3 Ability System

```cpp
class AbilitySystem {
public:
    AbilityResult UseAbility(
        PlayerId player,
        int slot,
        const Vector3& target
    ) {
        auto& state = m_players->GetState(player);

        // Validate slot
        if (slot < 0 || slot >= 4) {
            return AbilityResult::InvalidSlot();
        }

        auto& ability = state.abilities[slot];

        // Check if ability exists
        if (ability.id == AbilityId::None) {
            return AbilityResult::NoAbility();
        }

        // Check cooldown
        if (ability.currentCooldown > 0) {
            return AbilityResult::OnCooldown(ability.currentCooldown);
        }

        // Check resources (mana, charges, etc.)
        if (!HasResources(state, ability)) {
            return AbilityResult::NoResources();
        }

        // Validate target
        if (!ValidateAbilityTarget(state, ability, target)) {
            return AbilityResult::InvalidTarget();
        }

        // Execute ability
        ExecuteAbility(state, ability, target);

        // Start cooldown
        ability.currentCooldown = ability.baseCooldown;

        // Consume resources
        ConsumeResources(state, ability);

        // Record for stats/replay
        m_stats->RecordAbilityUse(player, ability.id);
        m_replay->RecordAbility(player, ability.id, target);

        return AbilityResult::Success();
    }

private:
    void ExecuteAbility(PlayerState& caster, AbilityState& ability, const Vector3& target) {
        const AbilityData& data = GetAbilityData(ability.id);

        switch (data.type) {
            case AbilityType::Projectile:
                SpawnAbilityProjectile(caster, data, target);
                break;

            case AbilityType::Instant:
                ApplyInstantEffect(caster, data, target);
                break;

            case AbilityType::Area:
                CreateAreaEffect(caster, data, target);
                break;

            case AbilityType::Buff:
                ApplyBuff(caster, data);
                break;

            case AbilityType::Movement:
                ApplyMovementAbility(caster, data, target);
                break;
        }
    }
};
```

---

## 6. NPC and AI Systems

### 6.1 NPC Management

```cpp
class NPCManager {
public:
    void SpawnNPCs(const MapData& map) {
        for (const auto& spawnPoint : map.npcSpawns) {
            SpawnNPC(spawnPoint);
        }
    }

    void Update(float deltaTime) {
        for (auto& [id, npc] : m_npcs) {
            if (!npc.isAlive) continue;

            // Update AI
            m_aiSystem->UpdateNPC(npc, deltaTime);

            // Process movement
            UpdateNPCMovement(npc, deltaTime);

            // Process combat
            ProcessNPCCombat(npc);

            // Check storm damage
            if (m_storm->IsInStorm(npc.position)) {
                ApplyStormDamage(npc, deltaTime);
            }
        }
    }

private:
    void SpawnNPC(const NPCSpawnPoint& spawn) {
        NPCState npc;
        npc.id = GenerateEntityId();
        npc.type = spawn.npcType;
        npc.position = spawn.position;
        npc.rotation = spawn.rotation;

        // Set stats from NPC type
        const NPCData& data = GetNPCData(spawn.npcType);
        npc.health = data.health;
        npc.maxHealth = data.health;
        npc.damage = data.damage;
        npc.moveSpeed = data.moveSpeed;
        npc.aggroRange = data.aggroRange;
        npc.attackRange = data.attackRange;

        npc.isAlive = true;
        npc.aiState = AIState::Idle;

        m_npcs[npc.id] = npc;
    }
};
```

### 6.2 AI Behavior System

```cpp
class AISystem {
public:
    void UpdateNPC(NPCState& npc, float deltaTime) {
        // Update perception
        UpdatePerception(npc);

        // Run behavior tree
        BehaviorResult result = m_behaviorTrees[npc.type]->Tick(npc, deltaTime);

        // Execute actions
        ProcessBehaviorResult(npc, result);
    }

private:
    void UpdatePerception(NPCState& npc) {
        npc.perception.nearbyPlayers.clear();
        npc.perception.nearbyNPCs.clear();

        // Find players in aggro range
        for (const auto& [id, player] : m_players->GetAllPlayers()) {
            if (!player.isAlive) continue;

            float distance = (player.position - npc.position).Length();
            if (distance <= npc.aggroRange) {
                // Check line of sight
                if (HasLineOfSight(npc.position, player.position)) {
                    npc.perception.nearbyPlayers.push_back({id, distance});
                }
            }
        }

        // Sort by distance
        std::sort(npc.perception.nearbyPlayers.begin(),
                  npc.perception.nearbyPlayers.end(),
                  [](const auto& a, const auto& b) {
                      return a.distance < b.distance;
                  });

        // Update current target
        if (!npc.perception.nearbyPlayers.empty()) {
            npc.currentTarget = npc.perception.nearbyPlayers[0].id;
        } else {
            npc.currentTarget = PlayerId::Invalid;
        }
    }

    std::unordered_map<NPCType, std::unique_ptr<BehaviorTree>> m_behaviorTrees;
};

// Example behavior tree for monster NPC
class MonsterBehaviorTree : public BehaviorTree {
public:
    MonsterBehaviorTree() {
        // Root selector
        auto root = std::make_unique<Selector>();

        // Combat sequence (if has target)
        auto combat = std::make_unique<Sequence>();
        combat->AddChild(std::make_unique<HasTarget>());
        combat->AddChild(std::make_unique<Selector>(
            std::make_unique<Sequence>(
                std::make_unique<InAttackRange>(),
                std::make_unique<Attack>()
            ),
            std::make_unique<ChaseTarget>()
        ));

        // Patrol sequence (if no target)
        auto patrol = std::make_unique<Sequence>();
        patrol->AddChild(std::make_unique<NoTarget>());
        patrol->AddChild(std::make_unique<Patrol>());

        root->AddChild(std::move(combat));
        root->AddChild(std::move(patrol));

        m_root = std::move(root);
    }
};
```

---

## 7. World Systems

### 7.1 Storm System

```cpp
class StormSystem {
public:
    void Initialize(const MatchConfig& config) {
        m_phases = config.stormPhases;
        m_currentPhase = 0;
        m_phaseTimer = m_phases[0].waitTime;
        m_isClosing = false;

        // Initial safe zone covers entire map
        m_currentZone = Circle{config.mapCenter, config.mapRadius};
        m_targetZone = m_currentZone;
    }

    void Update(float deltaTime) {
        m_phaseTimer -= deltaTime;

        if (m_phaseTimer <= 0) {
            if (m_isClosing) {
                // Finished closing, start next phase wait
                AdvancePhase();
            } else {
                // Finished waiting, start closing
                StartClosing();
            }
        }

        if (m_isClosing) {
            UpdateStormPosition(deltaTime);
        }

        // Apply damage to players in storm
        ApplyStormDamage(deltaTime);
    }

    bool IsInStorm(const Vector3& position) const {
        return !m_currentZone.Contains(Vector2(position.x, position.z));
    }

    float GetDamagePerSecond() const {
        return m_phases[m_currentPhase].damagePerSecond;
    }

private:
    void StartClosing() {
        m_isClosing = true;

        // Calculate next zone
        const StormPhase& phase = m_phases[m_currentPhase];
        m_targetZone = CalculateNextZone(phase);

        m_phaseTimer = phase.closeTime;
        m_closeStartZone = m_currentZone;

        // Broadcast storm warning
        BroadcastStormWarning(m_targetZone, phase.closeTime);
    }

    void UpdateStormPosition(float deltaTime) {
        const StormPhase& phase = m_phases[m_currentPhase];
        float progress = 1.0f - (m_phaseTimer / phase.closeTime);

        // Interpolate zone
        m_currentZone.center = Vector2::Lerp(
            m_closeStartZone.center,
            m_targetZone.center,
            progress
        );
        m_currentZone.radius = std::lerp(
            m_closeStartZone.radius,
            m_targetZone.radius,
            progress
        );
    }

    void ApplyStormDamage(float deltaTime) {
        float dps = GetDamagePerSecond();

        for (auto& [id, player] : m_players->GetAllPlayers()) {
            if (!player.isAlive) continue;

            if (IsInStorm(player.position)) {
                float damage = dps * deltaTime;
                m_damage->ApplyStormDamage(id, damage);
            }
        }
    }

    Circle CalculateNextZone(const StormPhase& phase) {
        // Random position within current zone
        Vector2 offset = Random::InsideUnitCircle() *
                        (m_currentZone.radius * phase.radiusMultiplier * 0.5f);
        Vector2 newCenter = m_currentZone.center + offset;

        // Ensure new zone fits within current
        float newRadius = m_currentZone.radius * phase.radiusMultiplier;

        return Circle{newCenter, newRadius};
    }

    std::vector<StormPhase> m_phases;
    int m_currentPhase;
    float m_phaseTimer;
    bool m_isClosing;
    Circle m_currentZone;
    Circle m_targetZone;
    Circle m_closeStartZone;
};

// Storm phase configuration
struct StormPhase {
    float waitTime;           // Seconds before closing starts
    float closeTime;          // Seconds to close
    float radiusMultiplier;   // New radius = current * multiplier
    float damagePerSecond;    // DPS while in storm
};

// Default storm phases
const std::vector<StormPhase> DEFAULT_STORM_PHASES = {
    {120.0f, 60.0f, 0.7f, 1.0f},    // Phase 1: 2min wait, 1min close
    {90.0f, 45.0f, 0.6f, 2.0f},     // Phase 2: 1.5min wait, 45s close
    {60.0f, 30.0f, 0.5f, 5.0f},     // Phase 3: 1min wait, 30s close
    {45.0f, 20.0f, 0.4f, 10.0f},    // Phase 4: 45s wait, 20s close
    {30.0f, 15.0f, 0.3f, 15.0f},    // Phase 5: 30s wait, 15s close
    {20.0f, 10.0f, 0.2f, 20.0f},    // Phase 6: 20s wait, 10s close
    {10.0f, 5.0f, 0.0f, 25.0f},     // Phase 7: Final close
};
```

### 7.2 Loot System

```cpp
class LootSystem {
public:
    void Initialize(const MapData& map) {
        // Spawn loot at designated spawn points
        for (const auto& spawnPoint : map.lootSpawns) {
            SpawnLoot(spawnPoint);
        }
    }

    void Update(float deltaTime) {
        // Process loot interactions
        ProcessPendingPickups();

        // Respawn loot if configured
        if (m_respawnEnabled) {
            UpdateRespawns(deltaTime);
        }
    }

    void SpawnDeathLoot(const PlayerState& player) {
        // Create death box at player location
        DeathBox box;
        box.id = GenerateEntityId();
        box.position = player.position;
        box.playerId = player.id;
        box.items = player.inventory;
        box.spawnTime = m_currentTime;

        m_deathBoxes[box.id] = box;

        // Broadcast death box spawn
        BroadcastEvent(DeathBoxSpawnEvent{box.id, box.position, player.username});
    }

    LootResult TryPickup(PlayerId player, EntityId lootId) {
        // Validate loot exists
        auto it = m_loot.find(lootId);
        if (it == m_loot.end()) {
            return LootResult::NotFound();
        }

        auto& loot = it->second;
        auto& playerState = m_players->GetState(player);

        // Validate distance
        float distance = (playerState.position - loot.position).Length();
        if (distance > PICKUP_RANGE) {
            return LootResult::TooFar();
        }

        // Validate can pick up
        if (!CanPickup(playerState, loot)) {
            return LootResult::CannotPickup();
        }

        // Add to player inventory
        AddToInventory(playerState, loot.item);

        // Remove loot from world
        m_loot.erase(it);

        // Broadcast pickup
        BroadcastEvent(LootPickupEvent{player, lootId, loot.item});

        return LootResult::Success(loot.item);
    }

private:
    void SpawnLoot(const LootSpawnPoint& spawn) {
        // Roll for loot rarity
        LootRarity rarity = RollRarity(spawn.rarityWeights);

        // Select item from loot table
        ItemId item = SelectItem(spawn.lootTable, rarity);

        // Create loot entity
        LootState loot;
        loot.id = GenerateEntityId();
        loot.position = spawn.position;
        loot.item = item;
        loot.rarity = rarity;
        loot.spawnTime = m_currentTime;

        m_loot[loot.id] = loot;
    }

    static constexpr float PICKUP_RANGE = 3.0f;
};
```

---

## 8. Anti-Cheat Validation

### 8.1 Validation Framework

```cpp
class ValidationSystem {
public:
    void ValidatePlayer(PlayerId player, float deltaTime) {
        auto& state = m_players->GetState(player);
        auto& violations = m_violations[player];

        // Movement validation
        auto moveResult = m_movementValidator->Validate(state, deltaTime);
        ProcessViolations(player, moveResult);

        // Combat validation
        auto combatResult = m_combatValidator->Validate(state);
        ProcessViolations(player, combatResult);

        // Resource validation
        auto resourceResult = m_resourceValidator->Validate(state);
        ProcessViolations(player, resourceResult);

        // Rate limit validation
        auto rateResult = m_rateValidator->Validate(state);
        ProcessViolations(player, rateResult);

        // Check violation threshold
        if (violations.score >= KICK_THRESHOLD) {
            KickPlayer(player, "Too many violations");
        } else if (violations.score >= FLAG_THRESHOLD) {
            FlagForReview(player);
        }
    }

private:
    void ProcessViolations(PlayerId player, const ValidationResult& result) {
        auto& violations = m_violations[player];

        for (const auto& violation : result.violations) {
            // Log violation
            LOG_WARNING("Player {} violation: {} - {}",
                player, ToString(violation.type), violation.message);

            // Add to violation score
            violations.score += GetViolationWeight(violation.type);
            violations.history.push_back(violation);

            // Correct state if possible
            if (violation.canCorrect) {
                ApplyCorrection(player, violation);
            }
        }

        // Decay violation score over time
        violations.score = std::max(0.0f, violations.score - DECAY_RATE);
    }

    float GetViolationWeight(ValidationType type) {
        switch (type) {
            case ValidationType::SpeedHack: return 10.0f;
            case ValidationType::FlyHack: return 15.0f;
            case ValidationType::NoClip: return 20.0f;
            case ValidationType::DamageHack: return 25.0f;
            case ValidationType::RateExceed: return 5.0f;
            default: return 1.0f;
        }
    }

    static constexpr float FLAG_THRESHOLD = 50.0f;
    static constexpr float KICK_THRESHOLD = 100.0f;
    static constexpr float DECAY_RATE = 0.1f;  // Per second
};
```

### 8.2 Specific Validators

```cpp
// Combat Validator
class CombatValidator {
public:
    ValidationResult Validate(const PlayerState& state) {
        ValidationResult result;

        // Validate damage dealt this tick
        for (const auto& damage : state.pendingDamage) {
            // Check damage is possible
            if (damage.amount > GetMaxPossibleDamage(state, damage.source)) {
                result.AddViolation(ValidationType::DamageHack,
                    "Damage {} exceeds max {} for source {}",
                    damage.amount, GetMaxPossibleDamage(state, damage.source),
                    ToString(damage.source));
            }

            // Check target was visible
            if (!WasTargetVisible(state, damage.target, damage.tick)) {
                result.AddViolation(ValidationType::Aimbot,
                    "Target {} was not visible at tick {}",
                    damage.target, damage.tick);
            }

            // Check fire rate
            if (!ValidateFireRate(state, damage.source)) {
                result.AddViolation(ValidationType::RateExceed,
                    "Fire rate exceeded for {}", ToString(damage.source));
            }
        }

        return result;
    }
};

// Resource Validator
class ResourceValidator {
public:
    ValidationResult Validate(const PlayerState& state) {
        ValidationResult result;

        // Validate health
        if (state.health > state.maxHealth) {
            result.AddViolation(ValidationType::ResourceHack,
                "Health {} exceeds max {}", state.health, state.maxHealth);
        }

        // Validate shield
        if (state.shield > state.maxShield) {
            result.AddViolation(ValidationType::ResourceHack,
                "Shield {} exceeds max {}", state.shield, state.maxShield);
        }

        // Validate ability cooldowns
        for (int i = 0; i < 4; i++) {
            const auto& ability = state.abilities[i];
            if (ability.currentCooldown < 0) {
                result.AddViolation(ValidationType::CooldownHack,
                    "Ability {} has negative cooldown", i);
            }
        }

        // Validate inventory
        for (const auto& item : state.inventory) {
            if (!IsValidItem(item.id)) {
                result.AddViolation(ValidationType::InventoryHack,
                    "Invalid item {} in inventory", item.id);
            }
        }

        return result;
    }
};
```

---

## 9. Server Lifecycle

### 9.1 Server States

```
┌─────────────────────────────────────────────────────────────────┐
│                    SERVER STATE MACHINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                │
│  │  INIT    │────▶│ WAITING  │────▶│ STARTING │                │
│  └──────────┘     └──────────┘     └──────────┘                │
│                        │                │                        │
│                        │                ▼                        │
│                        │          ┌──────────┐                  │
│                        │          │ IN_MATCH │                  │
│                        │          └──────────┘                  │
│                        │                │                        │
│                        │                ▼                        │
│                        │          ┌──────────┐                  │
│                        └─────────▶│ ENDING   │                  │
│                                   └──────────┘                  │
│                                        │                        │
│                                        ▼                        │
│                                   ┌──────────┐                  │
│                                   │ CLEANUP  │                  │
│                                   └──────────┘                  │
│                                        │                        │
│                                        ▼                        │
│                                   ┌──────────┐                  │
│                                   │ SHUTDOWN │                  │
│                                   └──────────┘                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Match Flow

```cpp
class MatchManager {
public:
    void StartMatch() {
        m_phase = MatchPhase::WarmUp;
        m_phaseTimer = WARMUP_DURATION;

        // Initialize world
        m_world->Initialize(m_mapData);
        m_storm->Initialize(m_matchConfig);
        m_loot->Initialize(m_mapData);
        m_npcs->SpawnNPCs(m_mapData);

        // Move all players to waiting area
        for (auto& [id, player] : m_players->GetAllPlayers()) {
            player.position = GetWaitingAreaPosition();
            player.canMove = true;
            player.canAttack = false;
        }

        LOG_INFO("Match {} starting warmup", m_matchId);
    }

    void Update(float deltaTime) {
        m_phaseTimer -= deltaTime;

        switch (m_phase) {
            case MatchPhase::WarmUp:
                if (m_phaseTimer <= 0) {
                    StartDrop();
                }
                break;

            case MatchPhase::Dropping:
                UpdateDrop(deltaTime);
                if (AllPlayersDropped()) {
                    StartGameplay();
                }
                break;

            case MatchPhase::InProgress:
                // Main game loop handled elsewhere
                break;

            case MatchPhase::Ending:
                if (m_phaseTimer <= 0) {
                    FinalizeMatch();
                }
                break;
        }
    }

    void EndMatch(PlayerId winner) {
        m_phase = MatchPhase::Ending;
        m_phaseTimer = END_SCREEN_DURATION;
        m_winner = winner;

        // Stop all combat
        DisableCombat();

        // Broadcast victory
        BroadcastVictory(winner);

        // Calculate final standings
        CalculateFinalPlacements();

        LOG_INFO("Match {} ending, winner: {}", m_matchId, winner);
    }

private:
    void StartDrop() {
        m_phase = MatchPhase::Dropping;

        // Enable dropping mechanics
        m_dropSystem->Enable();

        // Spawn all players on drop path
        for (auto& [id, player] : m_players->GetAllPlayers()) {
            player.position = GetDropStartPosition();
            player.isDropping = true;
        }

        LOG_INFO("Match {} starting drop phase", m_matchId);
    }

    void StartGameplay() {
        m_phase = MatchPhase::InProgress;
        m_matchStartTime = Clock::Now();

        // Enable all combat
        for (auto& [id, player] : m_players->GetAllPlayers()) {
            player.canAttack = true;
        }

        // Start storm
        m_storm->Start();

        LOG_INFO("Match {} gameplay started with {} players",
            m_matchId, m_players->GetAliveCount());
    }

    void FinalizeMatch() {
        // Save match results
        MatchResults results;
        results.matchId = m_matchId;
        results.duration = Clock::DurationSeconds(m_matchStartTime, Clock::Now());
        results.winner = m_winner;
        results.placements = m_placements;

        m_matchStorage->SaveResults(results);

        // Send results to players
        for (auto& [id, player] : m_players->GetAllPlayers()) {
            SendMatchResults(id, results);
        }

        // Report to orchestrator
        m_orchestrator->ReportMatchComplete(m_matchId, results);

        // Begin shutdown
        m_server->BeginShutdown();
    }

    static constexpr float WARMUP_DURATION = 60.0f;
    static constexpr float END_SCREEN_DURATION = 15.0f;
};
```

### 9.3 Graceful Shutdown

```cpp
class ServerShutdown {
public:
    void BeginShutdown(ShutdownReason reason) {
        LOG_INFO("Server shutdown initiated: {}", ToString(reason));

        m_shutdownReason = reason;
        m_state = ShutdownState::Draining;

        // Stop accepting new connections
        m_network->StopAccepting();

        // If match in progress, handle gracefully
        if (m_match->IsInProgress()) {
            if (reason == ShutdownReason::Emergency) {
                // Force end match
                m_match->ForceEnd("Server shutdown");
            } else {
                // Wait for match to complete naturally
                m_state = ShutdownState::WaitingForMatch;
                return;
            }
        }

        ContinueShutdown();
    }

    void ContinueShutdown() {
        // Disconnect all players
        for (auto& [id, player] : m_players->GetAllPlayers()) {
            m_network->Disconnect(id, "Server shutting down");
        }

        // Save state if needed
        if (m_config.saveStateOnShutdown) {
            m_state->SaveCheckpoint();
        }

        // Flush logs
        m_logger->Flush();

        // Report to orchestrator
        m_orchestrator->ReportServerShutdown(m_serverId, m_shutdownReason);

        // Cleanup resources
        Cleanup();

        m_state = ShutdownState::Complete;
        LOG_INFO("Server shutdown complete");
    }

private:
    void Cleanup() {
        // Release all resources
        m_network.reset();
        m_match.reset();
        m_world.reset();

        // Close connections
        m_database->Close();
        m_metrics->Close();
    }
};
```

---

## 10. Deployment Architecture

### 10.1 Server Binary Structure

```
game-server/
├── bin/
│   ├── game_server           # Main server executable
│   └── server_manager        # Process manager
├── config/
│   ├── server.json           # Server configuration
│   ├── game.json             # Game balance config
│   └── logging.json          # Logging configuration
├── data/
│   ├── maps/                 # Map data
│   ├── npcs/                 # NPC definitions
│   └── items/                # Item definitions
├── logs/                     # Runtime logs
└── state/                    # State checkpoints
```

### 10.2 Container Configuration

```dockerfile
# Dockerfile for game server
FROM ubuntu:22.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    libssl3 \
    libcurl4 \
    libc6 \
    && rm -rf /var/lib/apt/lists/*

# Create server user
RUN useradd -m -s /bin/bash gameserver

# Copy server files
COPY --chown=gameserver:gameserver bin/ /app/bin/
COPY --chown=gameserver:gameserver config/ /app/config/
COPY --chown=gameserver:gameserver data/ /app/data/

# Set permissions
RUN chmod +x /app/bin/*

# Create log and state directories
RUN mkdir -p /app/logs /app/state && \
    chown gameserver:gameserver /app/logs /app/state

# Switch to server user
USER gameserver
WORKDIR /app

# Expose game port
EXPOSE 7777/udp

# Health check
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s \
    CMD /app/bin/health_check || exit 1

# Run server
ENTRYPOINT ["/app/bin/server_manager"]
CMD ["--config", "/app/config/server.json"]
```

### 10.3 VM Configuration

```yaml
# Terraform configuration for game server VM
resource "aws_instance" "game_server" {
  ami           = var.game_server_ami
  instance_type = "c5.2xlarge"  # 8 vCPU, 16GB RAM

  vpc_security_group_ids = [aws_security_group.game_server.id]
  subnet_id              = var.private_subnet_id
  iam_instance_profile   = aws_iam_instance_profile.game_server.name

  root_block_device {
    volume_type = "gp3"
    volume_size = 50
    iops        = 3000
    throughput  = 125
  }

  user_data = <<-EOF
    #!/bin/bash
    # Install server manager
    curl -sSL https://releases.plunderstorm.com/server-manager/latest/install.sh | bash

    # Configure server
    cat > /etc/game-server/config.json <<EOC
    {
      "region": "${var.region}",
      "instances_per_vm": 4,
      "orchestrator_url": "${var.orchestrator_url}",
      "metrics_url": "${var.metrics_url}"
    }
    EOC

    # Start server manager
    systemctl enable game-server-manager
    systemctl start game-server-manager
  EOF

  tags = {
    Name        = "game-server-${var.region}-${count.index}"
    Role        = "game-server"
    Region      = var.region
    Environment = var.environment
  }
}

resource "aws_security_group" "game_server" {
  name        = "game-server-sg"
  description = "Security group for game servers"
  vpc_id      = var.vpc_id

  # Game traffic (UDP)
  ingress {
    from_port   = 7777
    to_port     = 7800
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Health check (TCP)
  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [var.orchestrator_sg_id]
  }

  # Outbound
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## 11. Performance Requirements

### 11.1 Performance Targets

| Metric | Target | Maximum |
|--------|--------|---------|
| Tick Rate | 20 Hz | - |
| Tick Duration | < 30ms | 50ms |
| Player Capacity | 60 | 60 |
| Network Bandwidth (per player) | < 30 KB/s | 50 KB/s |
| Memory Usage | < 2 GB | 3 GB |
| CPU Usage | < 60% | 80% |
| Server Start Time | < 5s | 10s |
| Match Load Time | < 3s | 5s |

### 11.2 Optimization Strategies

```cpp
// Object Pooling
class EntityPool {
public:
    template<typename T>
    T* Acquire() {
        auto& pool = GetPool<T>();
        if (pool.available.empty()) {
            // Grow pool
            GrowPool<T>(POOL_GROWTH_SIZE);
        }

        T* obj = pool.available.back();
        pool.available.pop_back();
        pool.active.insert(obj);
        return obj;
    }

    template<typename T>
    void Release(T* obj) {
        auto& pool = GetPool<T>();
        pool.active.erase(obj);
        obj->Reset();
        pool.available.push_back(obj);
    }

private:
    static constexpr size_t POOL_GROWTH_SIZE = 100;
};

// Spatial Partitioning
class SpatialHash {
public:
    void Insert(EntityId id, Vector3 position) {
        CellKey key = GetCellKey(position);
        m_cells[key].insert(id);
        m_entityCells[id] = key;
    }

    void Update(EntityId id, Vector3 newPosition) {
        CellKey newKey = GetCellKey(newPosition);
        CellKey oldKey = m_entityCells[id];

        if (newKey != oldKey) {
            m_cells[oldKey].erase(id);
            m_cells[newKey].insert(id);
            m_entityCells[id] = newKey;
        }
    }

    std::vector<EntityId> QueryRadius(Vector3 center, float radius) {
        std::vector<EntityId> result;

        // Get all cells that intersect with radius
        auto cells = GetCellsInRadius(center, radius);

        for (const auto& cellKey : cells) {
            auto it = m_cells.find(cellKey);
            if (it != m_cells.end()) {
                for (EntityId id : it->second) {
                    result.push_back(id);
                }
            }
        }

        return result;
    }

private:
    static constexpr float CELL_SIZE = 50.0f;

    CellKey GetCellKey(Vector3 pos) {
        int x = static_cast<int>(pos.x / CELL_SIZE);
        int z = static_cast<int>(pos.z / CELL_SIZE);
        return {x, z};
    }

    std::unordered_map<CellKey, std::set<EntityId>> m_cells;
    std::unordered_map<EntityId, CellKey> m_entityCells;
};

// Delta Compression
class DeltaCompressor {
public:
    std::vector<uint8_t> Compress(
        const WorldState& base,
        const WorldState& current
    ) {
        BitWriter writer;

        // Write tick delta
        writer.WriteVarInt(current.tick - base.tick);

        // Write changed players only
        uint32_t changedCount = 0;
        for (const auto& [id, player] : current.players) {
            auto baseIt = base.players.find(id);
            if (baseIt == base.players.end() ||
                HasChanged(baseIt->second, player)) {
                changedCount++;
            }
        }

        writer.WriteVarInt(changedCount);

        for (const auto& [id, player] : current.players) {
            auto baseIt = base.players.find(id);
            if (baseIt == base.players.end()) {
                // New player - write full state
                writer.WriteBit(true);
                WriteFullPlayer(writer, player);
            } else if (HasChanged(baseIt->second, player)) {
                // Changed player - write delta
                writer.WriteBit(false);
                WritePlayerDelta(writer, baseIt->second, player);
            }
        }

        return writer.GetData();
    }
};
```

### 11.3 Load Testing Requirements

| Test | Parameters | Success Criteria |
|------|------------|------------------|
| Full Match | 60 players, 20 min | No tick > 50ms |
| Stress Test | 60 players + 100 NPCs | < 5% slow ticks |
| Combat Heavy | 60 players all fighting | No desync |
| Reconnection | 30 disconnects/reconnects | 100% success |
| Long Running | 10 matches sequential | No memory leak |

---

## 12. Monitoring and Logging

### 12.1 Metrics Collection

```cpp
class ServerMetrics {
public:
    void RecordTick(float duration) {
        m_tickDuration.Record(duration);
        if (duration > TARGET_TICK_MS) {
            m_slowTicks.Increment();
        }
    }

    void RecordPlayerCount(int count) {
        m_playerCount.Set(count);
    }

    void RecordNetworkStats(const NetworkStats& stats) {
        m_bytesIn.Add(stats.bytesReceived);
        m_bytesOut.Add(stats.bytesSent);
        m_packetsIn.Add(stats.packetsReceived);
        m_packetsOut.Add(stats.packetsSent);
    }

    void RecordMatchEvent(MatchEventType type) {
        m_matchEvents[type].Increment();
    }

    // Export to Prometheus/StatsD
    void Export() {
        m_exporter->Gauge("game_server_tick_duration_ms", m_tickDuration.Average());
        m_exporter->Counter("game_server_slow_ticks_total", m_slowTicks.Value());
        m_exporter->Gauge("game_server_player_count", m_playerCount.Value());
        m_exporter->Counter("game_server_bytes_in_total", m_bytesIn.Value());
        m_exporter->Counter("game_server_bytes_out_total", m_bytesOut.Value());
        m_exporter->Counter("game_server_packets_in_total", m_packetsIn.Value());
        m_exporter->Counter("game_server_packets_out_total", m_packetsOut.Value());
    }

private:
    Histogram m_tickDuration;
    Counter m_slowTicks;
    Gauge m_playerCount;
    Counter m_bytesIn;
    Counter m_bytesOut;
    Counter m_packetsIn;
    Counter m_packetsOut;
    std::unordered_map<MatchEventType, Counter> m_matchEvents;

    static constexpr float TARGET_TICK_MS = 50.0f;
};
```

### 12.2 Logging System

```cpp
// Structured logging
class GameLogger {
public:
    void LogMatchEvent(const MatchEvent& event) {
        LOG_INFO_JSON({
            {"type", "match_event"},
            {"match_id", m_matchId},
            {"tick", m_currentTick},
            {"event_type", ToString(event.type)},
            {"data", event.ToJson()}
        });
    }

    void LogPlayerAction(PlayerId player, const std::string& action, const json& data) {
        LOG_DEBUG_JSON({
            {"type", "player_action"},
            {"match_id", m_matchId},
            {"tick", m_currentTick},
            {"player_id", player},
            {"action", action},
            {"data", data}
        });
    }

    void LogValidationViolation(PlayerId player, const ValidationViolation& violation) {
        LOG_WARNING_JSON({
            {"type", "validation_violation"},
            {"match_id", m_matchId},
            {"tick", m_currentTick},
            {"player_id", player},
            {"violation_type", ToString(violation.type)},
            {"message", violation.message},
            {"severity", violation.severity}
        });
    }

    void LogServerError(const std::string& component, const std::string& error) {
        LOG_ERROR_JSON({
            {"type", "server_error"},
            {"match_id", m_matchId},
            {"tick", m_currentTick},
            {"component", component},
            {"error", error}
        });
    }
};

// Log levels and retention
struct LogConfig {
    LogLevel minLevel = LogLevel::Info;
    bool enableReplayLog = true;
    std::string logPath = "/app/logs";

    // Retention (handled by log aggregator)
    int retentionDays = 30;
    int replayRetentionDays = 7;
};
```

### 12.3 Replay System

```cpp
class ReplaySystem {
public:
    void Initialize(const std::string& matchId) {
        m_matchId = matchId;
        m_writer = std::make_unique<ReplayWriter>(
            GetReplayPath(matchId)
        );

        // Write header
        ReplayHeader header;
        header.version = REPLAY_VERSION;
        header.matchId = matchId;
        header.startTime = Clock::Now();
        header.tickRate = m_tickRate;
        m_writer->WriteHeader(header);
    }

    void RecordFrame(uint64_t tick) {
        if (!m_enabled) return;

        ReplayFrame frame;
        frame.tick = tick;
        frame.worldState = m_world->GetState();
        frame.events = m_pendingEvents;

        m_writer->WriteFrame(frame);
        m_pendingEvents.clear();
    }

    void RecordEvent(const GameEvent& event) {
        if (!m_enabled) return;
        m_pendingEvents.push_back(event);
    }

    void Finalize() {
        // Write match summary
        ReplaySummary summary;
        summary.duration = m_matchDuration;
        summary.winner = m_winner;
        summary.placements = m_placements;
        summary.totalFrames = m_frameCount;

        m_writer->WriteSummary(summary);
        m_writer->Finalize();

        // Upload to storage
        UploadReplay(m_matchId, m_writer->GetPath());
    }

private:
    static constexpr uint32_t REPLAY_VERSION = 1;

    std::string m_matchId;
    std::unique_ptr<ReplayWriter> m_writer;
    std::vector<GameEvent> m_pendingEvents;
    bool m_enabled = true;
};
```

---

## 13. Cloud Provider Integration

### 13.1 Multi-Cloud Support

```cpp
// Cloud provider abstraction
class CloudProvider {
public:
    virtual ~CloudProvider() = default;

    // Instance management
    virtual std::string LaunchInstance(const InstanceConfig& config) = 0;
    virtual void TerminateInstance(const std::string& instanceId) = 0;
    virtual InstanceStatus GetInstanceStatus(const std::string& instanceId) = 0;

    // Networking
    virtual std::string GetPublicIP(const std::string& instanceId) = 0;
    virtual void AttachToLoadBalancer(const std::string& instanceId) = 0;

    // Storage
    virtual void UploadFile(const std::string& bucket, const std::string& key,
                           const std::vector<uint8_t>& data) = 0;
    virtual std::vector<uint8_t> DownloadFile(const std::string& bucket,
                                              const std::string& key) = 0;
};

// AWS Implementation
class AWSProvider : public CloudProvider {
public:
    std::string LaunchInstance(const InstanceConfig& config) override {
        Aws::EC2::Model::RunInstancesRequest request;
        request.SetImageId(config.ami);
        request.SetInstanceType(ToEC2InstanceType(config.instanceType));
        request.SetMinCount(1);
        request.SetMaxCount(1);
        request.SetSubnetId(config.subnetId);
        request.SetSecurityGroupIds({config.securityGroupId});

        // Add user data
        std::string userData = GenerateUserData(config);
        request.SetUserData(Base64Encode(userData));

        auto outcome = m_ec2Client.RunInstances(request);
        if (!outcome.IsSuccess()) {
            throw CloudException("Failed to launch instance: " +
                outcome.GetError().GetMessage());
        }

        return outcome.GetResult().GetInstances()[0].GetInstanceId();
    }

    // ... other implementations
};

// GCP Implementation
class GCPProvider : public CloudProvider {
    // ... GCP-specific implementations
};

// Azure Implementation
class AzureProvider : public CloudProvider {
    // ... Azure-specific implementations
};
```

### 13.2 Regional Configuration

```json
{
  "regions": {
    "na-east": {
      "provider": "aws",
      "region": "us-east-1",
      "ami": "ami-xxxxx",
      "instance_type": "c5.2xlarge",
      "subnet_id": "subnet-xxxxx",
      "security_group_id": "sg-xxxxx",
      "min_instances": 5,
      "max_instances": 50,
      "target_utilization": 0.7
    },
    "na-west": {
      "provider": "aws",
      "region": "us-west-2",
      "ami": "ami-yyyyy",
      "instance_type": "c5.2xlarge",
      "subnet_id": "subnet-yyyyy",
      "security_group_id": "sg-yyyyy",
      "min_instances": 3,
      "max_instances": 30,
      "target_utilization": 0.7
    },
    "eu-west": {
      "provider": "gcp",
      "region": "europe-west1",
      "machine_type": "c2-standard-8",
      "image": "projects/plunderstorm/images/game-server-v1",
      "network": "game-server-network",
      "min_instances": 5,
      "max_instances": 40,
      "target_utilization": 0.7
    },
    "asia-east": {
      "provider": "aws",
      "region": "ap-northeast-1",
      "ami": "ami-zzzzz",
      "instance_type": "c5.2xlarge",
      "subnet_id": "subnet-zzzzz",
      "security_group_id": "sg-zzzzz",
      "min_instances": 3,
      "max_instances": 30,
      "target_utilization": 0.7
    }
  }
}
```

### 13.3 Health Check API

```cpp
class HealthCheckServer {
public:
    void Start(int port) {
        m_server.Get("/health", [this](const Request& req, Response& res) {
            HealthStatus status = GetHealthStatus();
            res.set_content(status.ToJson(), "application/json");
            res.status = status.isHealthy ? 200 : 503;
        });

        m_server.Get("/ready", [this](const Request& req, Response& res) {
            bool ready = m_gameServer->IsReady();
            res.status = ready ? 200 : 503;
        });

        m_server.Get("/metrics", [this](const Request& req, Response& res) {
            std::string metrics = m_metrics->ExportPrometheus();
            res.set_content(metrics, "text/plain");
        });

        m_server.listen("0.0.0.0", port);
    }

private:
    HealthStatus GetHealthStatus() {
        HealthStatus status;
        status.isHealthy = true;
        status.serverState = m_gameServer->GetState();
        status.matchId = m_gameServer->GetMatchId();
        status.playerCount = m_gameServer->GetPlayerCount();
        status.tickRate = m_gameServer->GetActualTickRate();
        status.memoryUsage = GetMemoryUsage();
        status.cpuUsage = GetCPUUsage();
        status.uptime = m_gameServer->GetUptime();

        // Check for issues
        if (status.tickRate < 15.0f) {
            status.isHealthy = false;
            status.issues.push_back("Low tick rate");
        }

        if (status.memoryUsage > 0.9f) {
            status.isHealthy = false;
            status.issues.push_back("High memory usage");
        }

        return status;
    }
};
```

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Backend Lead | _____________ | _____________ | ___/___/___ |
| Infrastructure Lead | _____________ | _____________ | ___/___/___ |
| Game Director | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial dedicated game servers specification |

---

*This document contains implementation details. Handle according to company security policies.*
