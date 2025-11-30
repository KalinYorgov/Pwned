# Server-Authoritative Validation Specification - Plunderstorm Mobile

**Document ID:** BACK-015
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Validation Architecture](#2-validation-architecture)
3. [Movement Validation](#3-movement-validation)
4. [Combat Validation](#4-combat-validation)
5. [Ability Validation](#5-ability-validation)
6. [Loot Validation](#6-loot-validation)
7. [Economy Validation](#7-economy-validation)
8. [Rate Limiting](#8-rate-limiting)
9. [Violation Detection](#9-violation-detection)
10. [Logging and Forensics](#10-logging-and-forensics)
11. [False Positive Prevention](#11-false-positive-prevention)
12. [Testing Framework](#12-testing-framework)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the server-authoritative validation system for Plunderstorm Mobile, ensuring all critical game logic is validated server-side to prevent client-side cheating while maintaining smooth gameplay.

### 1.2 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Security | Prevent all common cheats | Critical |
| Fairness | All players compete on equal terms | Critical |
| Performance | Validation adds < 5ms per tick | High |
| Accuracy | False positive rate < 0.01% | High |
| Forensics | Full audit trail for investigations | Medium |

### 1.3 Validation Coverage

| Action | Validation Level | Server Authority |
|--------|------------------|------------------|
| Movement | Full | Authoritative |
| Damage | Full | Authoritative |
| Hit Detection | Full | Authoritative |
| Ability Usage | Full | Authoritative |
| Loot Pickup | Full | Authoritative |
| Currency | Full | Authoritative |
| Inventory | Full | Authoritative |
| Health/Shield | Full | Authoritative |

### 1.4 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     VALIDATION ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CLIENT INPUT                                                        │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    INPUT SANITIZER                           │   │
│  │  • Format validation  • Range checks  • Rate limiting        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                  VALIDATION PIPELINE                         │   │
│  │ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐    │   │
│  │ │ Movement  │ │  Combat   │ │  Ability  │ │   Loot    │    │   │
│  │ │ Validator │ │ Validator │ │ Validator │ │ Validator │    │   │
│  │ └───────────┘ └───────────┘ └───────────┘ └───────────┘    │   │
│  │ ┌───────────┐ ┌───────────┐ ┌───────────┐                  │   │
│  │ │  Economy  │ │   Rate    │ │  Sanity   │                  │   │
│  │ │ Validator │ │  Limiter  │ │  Checker  │                  │   │
│  │ └───────────┘ └───────────┘ └───────────┘                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                  VIOLATION HANDLER                           │   │
│  │  • Logging  • Scoring  • Correction  • Escalation           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   GAME STATE                                 │   │
│  │  • Authoritative world  • Player states  • Entity states    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Validation Architecture

### 2.1 Core Validation System

```cpp
class ValidationSystem {
public:
    ValidationResult ValidateAndProcess(
        PlayerId player,
        const ClientInput& input,
        float deltaTime
    ) {
        ValidationResult result;
        auto& playerState = m_world->GetPlayer(player);

        // Sanitize input first
        if (!m_inputSanitizer.Sanitize(input)) {
            result.AddViolation(ViolationType::MalformedInput);
            return result;
        }

        // Rate limit check
        if (!m_rateLimiter.CheckAndConsume(player, input.type)) {
            result.AddViolation(ViolationType::RateLimitExceeded);
            return result;
        }

        // Run validation pipeline
        auto moveResult = m_movementValidator.Validate(playerState, input, deltaTime);
        result.Merge(moveResult);

        if (input.HasCombat()) {
            auto combatResult = m_combatValidator.Validate(playerState, input);
            result.Merge(combatResult);
        }

        if (input.HasAbility()) {
            auto abilityResult = m_abilityValidator.Validate(playerState, input);
            result.Merge(abilityResult);
        }

        if (input.HasInteraction()) {
            auto interactResult = m_interactionValidator.Validate(playerState, input);
            result.Merge(interactResult);
        }

        // Process violations
        if (result.HasViolations()) {
            m_violationHandler.Process(player, result);
        }

        // Apply validated actions to game state
        if (result.IsActionAllowed()) {
            ApplyValidatedInput(player, input, result);
        }

        return result;
    }

private:
    void ApplyValidatedInput(
        PlayerId player,
        const ClientInput& input,
        const ValidationResult& result
    ) {
        auto& state = m_world->GetPlayer(player);

        // Apply movement (use server-calculated position if corrected)
        if (result.HasCorrection(CorrectionType::Position)) {
            state.position = result.GetCorrectedPosition();
        } else {
            state.position = CalculatePosition(state, input);
        }

        // Apply rotation
        state.rotation = input.lookRotation;

        // Process combat (server-authoritative damage)
        if (input.HasCombat() && result.IsActionAllowed(ActionType::Combat)) {
            ProcessCombat(player, input);
        }

        // Process abilities (server-authoritative)
        if (input.HasAbility() && result.IsActionAllowed(ActionType::Ability)) {
            ProcessAbility(player, input);
        }
    }

    MovementValidator m_movementValidator;
    CombatValidator m_combatValidator;
    AbilityValidator m_abilityValidator;
    InteractionValidator m_interactionValidator;
    EconomyValidator m_economyValidator;
    RateLimiter m_rateLimiter;
    InputSanitizer m_inputSanitizer;
    ViolationHandler m_violationHandler;
};
```

### 2.2 Validation Result

```cpp
struct ValidationResult {
    bool valid = true;
    std::vector<Violation> violations;
    std::map<CorrectionType, CorrectionData> corrections;
    std::set<ActionType> allowedActions;
    std::set<ActionType> blockedActions;

    void AddViolation(ViolationType type, const std::string& details = "") {
        violations.push_back({type, details, Clock::Now()});
        valid = false;
    }

    void AddCorrection(CorrectionType type, const CorrectionData& data) {
        corrections[type] = data;
    }

    void BlockAction(ActionType action) {
        blockedActions.insert(action);
        allowedActions.erase(action);
    }

    bool IsActionAllowed(ActionType action) const {
        return blockedActions.find(action) == blockedActions.end();
    }

    bool HasViolations() const { return !violations.empty(); }
    bool HasCorrection(CorrectionType type) const {
        return corrections.find(type) != corrections.end();
    }

    void Merge(const ValidationResult& other) {
        for (const auto& v : other.violations) {
            violations.push_back(v);
        }
        for (const auto& [type, data] : other.corrections) {
            corrections[type] = data;
        }
        for (auto action : other.blockedActions) {
            blockedActions.insert(action);
        }
        valid = valid && other.valid;
    }
};

enum class ViolationType {
    // Movement
    SpeedHack,
    Teleport,
    FlyHack,
    NoClip,
    OutOfBounds,

    // Combat
    DamageHack,
    AimbotSuspicion,
    ImpossibleHit,
    FireRateExceeded,

    // Abilities
    CooldownBypass,
    InvalidAbility,
    ResourceHack,

    // Loot
    InvalidLoot,
    LootTeleport,
    DuplicateLoot,

    // Economy
    CurrencyHack,
    InvalidTransaction,
    InventoryManipulation,

    // General
    RateLimitExceeded,
    MalformedInput,
    TimestampManipulation,
    StateDesync
};
```

---

## 3. Movement Validation

### 3.1 Speed Validation

```cpp
class MovementValidator {
public:
    ValidationResult Validate(
        const PlayerState& state,
        const ClientInput& input,
        float deltaTime
    ) {
        ValidationResult result;

        // Get claimed new position
        Vector3 claimedPosition = CalculateClaimedPosition(state, input, deltaTime);

        // Calculate maximum allowed distance
        float maxSpeed = CalculateMaxSpeed(state);
        float maxDistance = maxSpeed * deltaTime * SPEED_TOLERANCE;

        // Check distance traveled
        float actualDistance = (claimedPosition - state.position).Length();

        if (actualDistance > maxDistance) {
            result.AddViolation(
                ViolationType::SpeedHack,
                fmt::format("Moved {} units (max: {})", actualDistance, maxDistance)
            );

            // Correct to maximum allowed position
            Vector3 direction = (claimedPosition - state.position).Normalized();
            Vector3 correctedPos = state.position + direction * maxDistance;
            result.AddCorrection(CorrectionType::Position, {correctedPos});

            m_speedViolations[state.playerId]++;
        }

        // Check for teleportation (large instant movement)
        if (actualDistance > TELEPORT_THRESHOLD) {
            result.AddViolation(
                ViolationType::Teleport,
                fmt::format("Teleported {} units", actualDistance)
            );
        }

        // Validate vertical movement
        ValidateVerticalMovement(state, claimedPosition, result);

        // Validate terrain collision
        ValidateTerrainCollision(state.position, claimedPosition, result);

        // Validate bounds
        ValidateBounds(claimedPosition, result);

        return result;
    }

private:
    float CalculateMaxSpeed(const PlayerState& state) {
        float baseSpeed = PLAYER_BASE_SPEED;

        // Sprint modifier
        if (state.isSprinting && state.stamina > 0) {
            baseSpeed *= SPRINT_MULTIPLIER;
        }

        // Crouch modifier
        if (state.isCrouching) {
            baseSpeed *= CROUCH_MULTIPLIER;
        }

        // Status effect modifiers
        for (const auto& effect : state.statusEffects) {
            switch (effect.type) {
                case StatusEffectType::SpeedBoost:
                    baseSpeed *= effect.magnitude;
                    break;
                case StatusEffectType::Slow:
                    baseSpeed *= effect.magnitude;
                    break;
                case StatusEffectType::Root:
                    baseSpeed = 0.0f;
                    break;
                case StatusEffectType::Stun:
                    baseSpeed = 0.0f;
                    break;
            }
        }

        // Clamp to absolute maximum
        return std::min(baseSpeed, ABSOLUTE_MAX_SPEED);
    }

    void ValidateVerticalMovement(
        const PlayerState& state,
        const Vector3& newPos,
        ValidationResult& result
    ) {
        float verticalDelta = newPos.y - state.position.y;

        // Check for impossible upward movement
        if (verticalDelta > 0 && !state.isJumping && !state.isOnLadder) {
            if (verticalDelta > MAX_STEP_HEIGHT) {
                result.AddViolation(
                    ViolationType::FlyHack,
                    fmt::format("Vertical rise {} without jump", verticalDelta)
                );
            }
        }

        // Validate jump height
        if (state.isJumping) {
            float timeSinceJump = m_currentTime - state.jumpStartTime;
            float expectedHeight = CalculateJumpHeight(timeSinceJump);

            if (verticalDelta > expectedHeight * VERTICAL_TOLERANCE) {
                result.AddViolation(
                    ViolationType::FlyHack,
                    fmt::format("Jump height {} exceeds expected {}",
                        verticalDelta, expectedHeight)
                );
            }
        }
    }

    void ValidateTerrainCollision(
        const Vector3& oldPos,
        const Vector3& newPos,
        ValidationResult& result
    ) {
        // Check if new position is inside terrain
        if (m_terrain->IsInsideTerrain(newPos)) {
            result.AddViolation(ViolationType::NoClip, "Position inside terrain");

            // Correct to valid position
            Vector3 validPos = m_terrain->GetNearestValidPosition(newPos);
            result.AddCorrection(CorrectionType::Position, {validPos});
        }

        // Check if movement path goes through solid objects
        RaycastHit hit;
        if (m_terrain->Raycast(oldPos, newPos, hit)) {
            if (hit.distance < (newPos - oldPos).Length() * 0.9f) {
                result.AddViolation(
                    ViolationType::NoClip,
                    "Movement path through solid object"
                );
                result.AddCorrection(CorrectionType::Position, {hit.point});
            }
        }
    }

    void ValidateBounds(const Vector3& pos, ValidationResult& result) {
        if (!m_map->IsWithinBounds(pos)) {
            result.AddViolation(ViolationType::OutOfBounds);
        }
    }

    // Constants
    static constexpr float PLAYER_BASE_SPEED = 8.0f;
    static constexpr float SPRINT_MULTIPLIER = 1.5f;
    static constexpr float CROUCH_MULTIPLIER = 0.5f;
    static constexpr float ABSOLUTE_MAX_SPEED = 30.0f;
    static constexpr float SPEED_TOLERANCE = 1.15f;  // 15% tolerance
    static constexpr float TELEPORT_THRESHOLD = 20.0f;
    static constexpr float MAX_STEP_HEIGHT = 0.5f;
    static constexpr float VERTICAL_TOLERANCE = 1.2f;
};
```

### 3.2 Position History

```cpp
class PositionHistory {
public:
    void RecordPosition(PlayerId player, const Vector3& position, uint64_t tick) {
        auto& history = m_history[player];
        history.push_back({position, tick, Clock::Now()});

        // Keep last N positions
        while (history.size() > MAX_HISTORY_SIZE) {
            history.erase(history.begin());
        }
    }

    Vector3 GetPositionAtTick(PlayerId player, uint64_t tick) const {
        auto it = m_history.find(player);
        if (it == m_history.end()) {
            return Vector3::Zero;
        }

        const auto& history = it->second;

        // Find closest tick
        for (auto rit = history.rbegin(); rit != history.rend(); ++rit) {
            if (rit->tick <= tick) {
                // Interpolate if needed
                auto next = std::prev(rit);
                if (next != history.rend() && next->tick > tick) {
                    float t = static_cast<float>(tick - rit->tick) /
                              static_cast<float>(next->tick - rit->tick);
                    return Vector3::Lerp(rit->position, next->position, t);
                }
                return rit->position;
            }
        }

        return history.front().position;
    }

    std::vector<PositionRecord> GetRecentHistory(
        PlayerId player,
        uint64_t startTick
    ) const {
        std::vector<PositionRecord> result;
        auto it = m_history.find(player);
        if (it == m_history.end()) return result;

        for (const auto& record : it->second) {
            if (record.tick >= startTick) {
                result.push_back(record);
            }
        }
        return result;
    }

private:
    static constexpr size_t MAX_HISTORY_SIZE = 128;  // ~6 seconds at 20Hz
    std::unordered_map<PlayerId, std::vector<PositionRecord>> m_history;
};
```

---

## 4. Combat Validation

### 4.1 Damage Validation

```cpp
class CombatValidator {
public:
    ValidationResult Validate(
        const PlayerState& attacker,
        const ClientInput& input
    ) {
        ValidationResult result;

        if (!input.HasCombat()) return result;

        // Validate attacker can attack
        if (!CanAttack(attacker)) {
            result.AddViolation(ViolationType::InvalidAction);
            result.BlockAction(ActionType::Combat);
            return result;
        }

        // Validate fire rate
        if (!ValidateFireRate(attacker, input)) {
            result.AddViolation(ViolationType::FireRateExceeded);
            result.BlockAction(ActionType::Combat);
            return result;
        }

        // Validate weapon/ability
        if (!ValidateWeapon(attacker, input.weaponId)) {
            result.AddViolation(ViolationType::InvalidWeapon);
            result.BlockAction(ActionType::Combat);
            return result;
        }

        // If claiming a hit, validate it
        if (input.claimsHit) {
            auto hitResult = ValidateHitClaim(attacker, input);
            result.Merge(hitResult);
        }

        return result;
    }

    ValidationResult ValidateHitClaim(
        const PlayerState& attacker,
        const ClientInput& input
    ) {
        ValidationResult result;

        // Get target
        auto* target = m_world->GetPlayer(input.targetId);
        if (!target) {
            result.AddViolation(ViolationType::InvalidTarget);
            return result;
        }

        // Get positions at claimed hit time (lag compensation)
        uint64_t hitTick = input.clientTick;
        Vector3 attackerPosAtHit = m_positionHistory->GetPositionAtTick(
            attacker.playerId, hitTick
        );
        Vector3 targetPosAtHit = m_positionHistory->GetPositionAtTick(
            input.targetId, hitTick
        );

        // Validate distance
        float distance = (targetPosAtHit - attackerPosAtHit).Length();
        float maxRange = GetWeaponRange(input.weaponId);

        if (distance > maxRange * RANGE_TOLERANCE) {
            result.AddViolation(
                ViolationType::ImpossibleHit,
                fmt::format("Distance {} exceeds range {}", distance, maxRange)
            );
            return result;
        }

        // Validate line of sight
        if (!HasLineOfSight(attackerPosAtHit, targetPosAtHit)) {
            result.AddViolation(
                ViolationType::ImpossibleHit,
                "No line of sight to target"
            );
            return result;
        }

        // Validate aim angle
        Vector3 aimDir = input.aimDirection.Normalized();
        Vector3 toTarget = (targetPosAtHit - attackerPosAtHit).Normalized();
        float angle = Vector3::Angle(aimDir, toTarget);

        float maxAngle = GetWeaponSpread(input.weaponId) + AIM_TOLERANCE;
        if (angle > maxAngle) {
            result.AddViolation(
                ViolationType::AimbotSuspicion,
                fmt::format("Aim angle {} exceeds spread {}", angle, maxAngle)
            );

            // Don't immediately reject - record for pattern analysis
            m_aimAnalyzer.RecordSuspiciousAim(attacker.playerId, angle);
        }

        // Validate claimed damage
        float claimedDamage = input.claimedDamage;
        float maxDamage = CalculateMaxDamage(attacker, input, distance);

        if (claimedDamage > maxDamage * DAMAGE_TOLERANCE) {
            result.AddViolation(
                ViolationType::DamageHack,
                fmt::format("Claimed {} damage (max: {})", claimedDamage, maxDamage)
            );
        }

        return result;
    }

private:
    bool ValidateFireRate(const PlayerState& attacker, const ClientInput& input) {
        auto& fireHistory = m_fireHistory[attacker.playerId];
        float minInterval = GetWeaponFireInterval(input.weaponId);

        float timeSinceLastFire = m_currentTime - fireHistory.lastFireTime;

        if (timeSinceLastFire < minInterval * FIRE_RATE_TOLERANCE) {
            fireHistory.violations++;
            return false;
        }

        fireHistory.lastFireTime = m_currentTime;
        return true;
    }

    float CalculateMaxDamage(
        const PlayerState& attacker,
        const ClientInput& input,
        float distance
    ) {
        const auto& weaponData = GetWeaponData(input.weaponId);

        float damage = weaponData.baseDamage;

        // Headshot
        if (input.hitLocation == HitLocation::Head) {
            damage *= weaponData.headshotMultiplier;
        }

        // Distance falloff
        damage *= CalculateFalloff(distance, weaponData);

        // Attacker damage modifiers
        damage *= attacker.damageMultiplier;

        // Maximum possible with all buffs
        return damage * MAX_DAMAGE_BUFF_MULTIPLIER;
    }

    static constexpr float RANGE_TOLERANCE = 1.1f;
    static constexpr float AIM_TOLERANCE = 5.0f;  // degrees
    static constexpr float DAMAGE_TOLERANCE = 1.05f;
    static constexpr float FIRE_RATE_TOLERANCE = 0.9f;
    static constexpr float MAX_DAMAGE_BUFF_MULTIPLIER = 2.0f;
};
```

### 4.2 Server-Authoritative Damage

```cpp
class DamageAuthority {
public:
    // Server calculates all damage - never trust client values
    DamageResult CalculateAndApplyDamage(
        PlayerId attackerId,
        PlayerId targetId,
        const HitData& hit
    ) {
        auto& attacker = m_world->GetPlayer(attackerId);
        auto& target = m_world->GetPlayer(targetId);

        DamageResult result;

        // Calculate base damage
        float damage = GetBaseDamage(hit.weaponId, hit.abilityId);

        // Apply hit location multiplier
        damage *= GetHitLocationMultiplier(hit.location);

        // Apply distance falloff
        float distance = (target.position - attacker.position).Length();
        damage *= CalculateFalloff(distance, hit.weaponId);

        // Apply attacker modifiers
        damage *= GetAttackerDamageModifier(attacker);

        // Apply target damage reduction
        damage *= (1.0f - GetTargetDamageReduction(target));

        // Apply to shield first
        if (target.shield > 0) {
            float shieldDamage = std::min(damage, target.shield);
            target.shield -= shieldDamage;
            result.shieldDamage = shieldDamage;
            damage -= shieldDamage;
        }

        // Apply to health
        if (damage > 0) {
            target.health -= damage;
            result.healthDamage = damage;
        }

        result.totalDamage = result.shieldDamage + result.healthDamage;
        result.targetHealth = target.health;
        result.targetShield = target.shield;

        // Check for elimination
        if (target.health <= 0) {
            ProcessElimination(targetId, attackerId, hit);
            result.eliminated = true;
        }

        // Record for stats
        m_stats->RecordDamage(attackerId, targetId, result);

        // Log for audit
        m_auditLog->LogDamage(attackerId, targetId, hit, result);

        return result;
    }

private:
    float GetHitLocationMultiplier(HitLocation location) {
        switch (location) {
            case HitLocation::Head: return 2.0f;
            case HitLocation::Body: return 1.0f;
            case HitLocation::Limbs: return 0.75f;
            default: return 1.0f;
        }
    }
};
```

---

## 5. Ability Validation

### 5.1 Ability Validator

```cpp
class AbilityValidator {
public:
    ValidationResult Validate(
        const PlayerState& player,
        const ClientInput& input
    ) {
        ValidationResult result;

        if (!input.HasAbility()) return result;

        int slot = input.abilitySlot;

        // Validate slot
        if (slot < 0 || slot >= MAX_ABILITY_SLOTS) {
            result.AddViolation(ViolationType::InvalidAbility);
            result.BlockAction(ActionType::Ability);
            return result;
        }

        const auto& ability = player.abilities[slot];

        // Validate player has this ability
        if (ability.id == AbilityId::None) {
            result.AddViolation(ViolationType::InvalidAbility);
            result.BlockAction(ActionType::Ability);
            return result;
        }

        // Validate cooldown (server tracks cooldowns)
        if (!ValidateCooldown(player.playerId, slot, ability)) {
            result.AddViolation(
                ViolationType::CooldownBypass,
                fmt::format("Ability {} on cooldown ({}s remaining)",
                    slot, GetRemainingCooldown(player.playerId, slot))
            );
            result.BlockAction(ActionType::Ability);
            return result;
        }

        // Validate resources
        if (!ValidateResources(player, ability)) {
            result.AddViolation(
                ViolationType::ResourceHack,
                "Insufficient resources for ability"
            );
            result.BlockAction(ActionType::Ability);
            return result;
        }

        // Validate player state allows ability use
        if (!CanUseAbility(player)) {
            result.AddViolation(ViolationType::InvalidAction);
            result.BlockAction(ActionType::Ability);
            return result;
        }

        // Validate target (if targeted ability)
        if (ability.isTargeted) {
            auto targetResult = ValidateAbilityTarget(player, ability, input);
            result.Merge(targetResult);
        }

        return result;
    }

private:
    bool ValidateCooldown(PlayerId player, int slot, const AbilityState& ability) {
        auto& cooldowns = m_serverCooldowns[player];

        auto it = cooldowns.find(slot);
        if (it == cooldowns.end()) {
            return true;  // No cooldown active
        }

        float remaining = it->second - m_currentTime;
        return remaining <= 0;
    }

    bool ValidateResources(const PlayerState& player, const AbilityState& ability) {
        const auto& abilityData = GetAbilityData(ability.id);

        // Check mana/energy
        if (abilityData.manaCost > 0 && player.mana < abilityData.manaCost) {
            return false;
        }

        // Check charges
        if (abilityData.usesCharges && ability.charges <= 0) {
            return false;
        }

        // Check health cost
        if (abilityData.healthCost > 0 && player.health <= abilityData.healthCost) {
            return false;
        }

        return true;
    }

    bool CanUseAbility(const PlayerState& player) {
        // Cannot use abilities while stunned
        if (player.HasStatusEffect(StatusEffectType::Stun)) {
            return false;
        }

        // Cannot use abilities while silenced
        if (player.HasStatusEffect(StatusEffectType::Silence)) {
            return false;
        }

        // Cannot use abilities while dead
        if (!player.isAlive) {
            return false;
        }

        // Cannot use abilities while knocked
        if (player.isKnocked) {
            return false;
        }

        return true;
    }

    ValidationResult ValidateAbilityTarget(
        const PlayerState& player,
        const AbilityState& ability,
        const ClientInput& input
    ) {
        ValidationResult result;
        const auto& abilityData = GetAbilityData(ability.id);

        // Validate range
        float distance = (input.abilityTarget - player.position).Length();
        if (distance > abilityData.range * RANGE_TOLERANCE) {
            result.AddViolation(
                ViolationType::InvalidAbility,
                fmt::format("Target out of range ({} > {})",
                    distance, abilityData.range)
            );
        }

        // Validate line of sight (for some abilities)
        if (abilityData.requiresLineOfSight) {
            if (!HasLineOfSight(player.position, input.abilityTarget)) {
                result.AddViolation(
                    ViolationType::InvalidAbility,
                    "No line of sight to target"
                );
            }
        }

        return result;
    }

    // Server maintains cooldowns - never trust client
    std::unordered_map<PlayerId, std::map<int, float>> m_serverCooldowns;

    static constexpr float RANGE_TOLERANCE = 1.1f;
    static constexpr int MAX_ABILITY_SLOTS = 4;
};
```

### 5.2 Server-Authoritative Ability Execution

```cpp
class AbilityAuthority {
public:
    AbilityResult ExecuteAbility(
        PlayerId playerId,
        int slot,
        const Vector3& target
    ) {
        auto& player = m_world->GetPlayer(playerId);
        auto& ability = player.abilities[slot];
        const auto& abilityData = GetAbilityData(ability.id);

        AbilityResult result;

        // Consume resources (server-side)
        ConsumeResources(player, abilityData);

        // Start cooldown (server-side)
        StartCooldown(playerId, slot, abilityData.cooldown);

        // Execute ability effect
        switch (abilityData.type) {
            case AbilityType::Projectile:
                SpawnProjectile(playerId, ability.id, target);
                break;

            case AbilityType::Instant:
                ApplyInstantEffect(playerId, ability.id, target);
                break;

            case AbilityType::Area:
                CreateAreaEffect(playerId, ability.id, target);
                break;

            case AbilityType::Buff:
                ApplyBuff(playerId, ability.id);
                break;

            case AbilityType::Movement:
                ExecuteMovementAbility(playerId, ability.id, target);
                break;
        }

        result.success = true;
        result.cooldownStarted = abilityData.cooldown;

        // Log for audit
        m_auditLog->LogAbility(playerId, ability.id, target, result);

        return result;
    }

private:
    void StartCooldown(PlayerId player, int slot, float duration) {
        m_cooldowns[player][slot] = m_currentTime + duration;
    }

    void ConsumeResources(PlayerState& player, const AbilityData& data) {
        if (data.manaCost > 0) {
            player.mana -= data.manaCost;
        }
        if (data.healthCost > 0) {
            player.health -= data.healthCost;
        }
    }

    std::unordered_map<PlayerId, std::map<int, float>> m_cooldowns;
};
```

---

## 6. Loot Validation

### 6.1 Loot Validator

```cpp
class LootValidator {
public:
    ValidationResult ValidatePickup(
        const PlayerState& player,
        EntityId lootId
    ) {
        ValidationResult result;

        // Validate loot exists (server is authority on loot)
        auto* loot = m_world->GetLoot(lootId);
        if (!loot) {
            result.AddViolation(
                ViolationType::InvalidLoot,
                fmt::format("Loot {} does not exist", lootId)
            );
            return result;
        }

        // Validate loot not already picked up
        if (loot->isPickedUp) {
            result.AddViolation(
                ViolationType::DuplicateLoot,
                "Loot already picked up"
            );
            return result;
        }

        // Validate distance
        float distance = (loot->position - player.position).Length();
        if (distance > PICKUP_RANGE * RANGE_TOLERANCE) {
            result.AddViolation(
                ViolationType::LootTeleport,
                fmt::format("Loot pickup distance {} exceeds range {}",
                    distance, PICKUP_RANGE)
            );
            return result;
        }

        // Validate player can pick up (inventory space, etc.)
        if (!CanPickup(player, loot->item)) {
            result.AddViolation(ViolationType::InvalidAction);
            return result;
        }

        return result;
    }

private:
    bool CanPickup(const PlayerState& player, const ItemData& item) {
        // Check inventory space
        if (player.inventory.IsFull()) {
            return false;
        }

        // Check item restrictions
        if (item.isClassRestricted && item.allowedClass != player.playerClass) {
            return false;
        }

        return true;
    }

    static constexpr float PICKUP_RANGE = 3.0f;
    static constexpr float RANGE_TOLERANCE = 1.2f;
};

class LootAuthority {
public:
    // Server spawns all loot - clients cannot create loot
    void SpawnLoot(const LootSpawnPoint& spawn) {
        EntityId lootId = GenerateEntityId();

        LootEntity loot;
        loot.id = lootId;
        loot.position = spawn.position;
        loot.item = RollItem(spawn.lootTable);
        loot.spawnTime = m_currentTime;
        loot.isPickedUp = false;

        m_world->AddLoot(loot);
        m_spawnedLoot.insert(lootId);
    }

    // Server processes all pickups
    PickupResult ProcessPickup(PlayerId playerId, EntityId lootId) {
        auto* loot = m_world->GetLoot(lootId);
        if (!loot || loot->isPickedUp) {
            return PickupResult::Failed("Invalid loot");
        }

        // Mark as picked up
        loot->isPickedUp = true;
        loot->pickedUpBy = playerId;

        // Add to player inventory
        auto& player = m_world->GetPlayer(playerId);
        player.inventory.Add(loot->item);

        // Remove from world
        m_world->RemoveLoot(lootId);

        return PickupResult::Success(loot->item);
    }

private:
    std::set<EntityId> m_spawnedLoot;  // Track all server-spawned loot
};
```

---

## 7. Economy Validation

### 7.1 Currency Validator

```cpp
class EconomyValidator {
public:
    ValidationResult ValidateTransaction(
        PlayerId player,
        const Transaction& transaction
    ) {
        ValidationResult result;

        // Get server-authoritative balance
        auto balance = m_economyService->GetBalance(player);

        switch (transaction.type) {
            case TransactionType::Spend:
                result = ValidateSpend(player, transaction, balance);
                break;

            case TransactionType::Earn:
                result = ValidateEarn(player, transaction);
                break;

            case TransactionType::Trade:
                result = ValidateTrade(player, transaction);
                break;
        }

        return result;
    }

private:
    ValidationResult ValidateSpend(
        PlayerId player,
        const Transaction& txn,
        const Balance& balance
    ) {
        ValidationResult result;

        // Check sufficient funds
        if (txn.currency == Currency::Gold) {
            if (balance.gold < txn.amount) {
                result.AddViolation(
                    ViolationType::CurrencyHack,
                    fmt::format("Insufficient gold: {} < {}",
                        balance.gold, txn.amount)
                );
            }
        } else if (txn.currency == Currency::Gems) {
            if (balance.gems < txn.amount) {
                result.AddViolation(
                    ViolationType::CurrencyHack,
                    fmt::format("Insufficient gems: {} < {}",
                        balance.gems, txn.amount)
                );
            }
        }

        // Validate item being purchased exists
        if (txn.itemId != ItemId::None) {
            if (!ValidatePurchaseItem(txn.itemId, txn.amount)) {
                result.AddViolation(ViolationType::InvalidTransaction);
            }
        }

        return result;
    }

    ValidationResult ValidateEarn(
        PlayerId player,
        const Transaction& txn
    ) {
        ValidationResult result;

        // Validate earning source
        if (!IsValidEarnSource(txn.source)) {
            result.AddViolation(
                ViolationType::CurrencyHack,
                "Invalid currency earn source"
            );
            return result;
        }

        // Validate amount for source
        int maxEarn = GetMaxEarnForSource(txn.source);
        if (txn.amount > maxEarn) {
            result.AddViolation(
                ViolationType::CurrencyHack,
                fmt::format("Earn amount {} exceeds max {} for source",
                    txn.amount, maxEarn)
            );
        }

        // Rate limit earnings
        if (!m_earnRateLimiter.Check(player, txn.source)) {
            result.AddViolation(ViolationType::RateLimitExceeded);
        }

        return result;
    }
};
```

### 7.2 Inventory Validation

```cpp
class InventoryValidator {
public:
    ValidationResult ValidateInventoryChange(
        PlayerId player,
        const InventoryChange& change
    ) {
        ValidationResult result;

        auto& inventory = m_world->GetPlayer(player).inventory;

        switch (change.type) {
            case InventoryChangeType::Add:
                // Validate item can be added
                if (!CanAddItem(inventory, change.item)) {
                    result.AddViolation(ViolationType::InventoryManipulation);
                }
                // Validate item source
                if (!ValidateItemSource(player, change.item, change.source)) {
                    result.AddViolation(ViolationType::InventoryManipulation);
                }
                break;

            case InventoryChangeType::Remove:
                // Validate player has item
                if (!inventory.Contains(change.item)) {
                    result.AddViolation(ViolationType::InventoryManipulation);
                }
                break;

            case InventoryChangeType::Equip:
                // Validate player owns item
                if (!inventory.Contains(change.item)) {
                    result.AddViolation(ViolationType::InventoryManipulation);
                }
                // Validate item can be equipped
                if (!CanEquip(inventory, change.item, change.slot)) {
                    result.AddViolation(ViolationType::InvalidAction);
                }
                break;
        }

        return result;
    }

private:
    bool ValidateItemSource(
        PlayerId player,
        const Item& item,
        ItemSource source
    ) {
        switch (source) {
            case ItemSource::Loot:
                // Verify loot pickup was recorded
                return m_lootAuthority->WasPickedUpBy(item.sourceId, player);

            case ItemSource::Purchase:
                // Verify purchase was recorded
                return m_purchaseLog->WasPurchasedBy(item.id, player);

            case ItemSource::Reward:
                // Verify reward was granted
                return m_rewardLog->WasGrantedTo(item.id, player);

            case ItemSource::Trade:
                // Verify trade was completed
                return m_tradeLog->WasTradedTo(item.id, player);

            default:
                return false;
        }
    }
};
```

---

## 8. Rate Limiting

### 8.1 Rate Limiter

```cpp
class RateLimiter {
public:
    bool CheckAndConsume(PlayerId player, InputType type) {
        auto& buckets = m_buckets[player];
        auto& bucket = buckets[type];

        // Refill bucket based on time elapsed
        RefillBucket(bucket, type);

        // Check if action is allowed
        if (bucket.tokens < 1.0f) {
            RecordRateLimitViolation(player, type);
            return false;
        }

        // Consume token
        bucket.tokens -= 1.0f;
        return true;
    }

    void SetLimit(InputType type, float tokensPerSecond, float maxTokens) {
        m_limits[type] = {tokensPerSecond, maxTokens};
    }

private:
    void RefillBucket(TokenBucket& bucket, InputType type) {
        auto now = Clock::Now();
        float elapsed = Clock::DurationSeconds(bucket.lastRefill, now);
        bucket.lastRefill = now;

        auto& limit = m_limits[type];
        bucket.tokens = std::min(
            bucket.tokens + elapsed * limit.tokensPerSecond,
            limit.maxTokens
        );
    }

    void RecordRateLimitViolation(PlayerId player, InputType type) {
        auto& violations = m_violations[player];
        violations[type]++;

        // Escalate if too many violations
        if (violations[type] > VIOLATION_THRESHOLD) {
            m_violationHandler->Escalate(player, ViolationType::RateLimitExceeded);
        }
    }

    struct TokenBucket {
        float tokens = 0.0f;
        TimePoint lastRefill;
    };

    struct RateLimit {
        float tokensPerSecond;
        float maxTokens;
    };

    std::unordered_map<PlayerId, std::map<InputType, TokenBucket>> m_buckets;
    std::map<InputType, RateLimit> m_limits;
    std::unordered_map<PlayerId, std::map<InputType, int>> m_violations;

    static constexpr int VIOLATION_THRESHOLD = 10;
};

// Default rate limits
const std::map<InputType, RateLimit> DEFAULT_RATE_LIMITS = {
    {InputType::Movement, {20.0f, 25.0f}},      // 20/sec, burst 25
    {InputType::Fire, {10.0f, 12.0f}},          // 10/sec, burst 12
    {InputType::Ability, {4.0f, 5.0f}},         // 4/sec, burst 5
    {InputType::Interact, {5.0f, 8.0f}},        // 5/sec, burst 8
    {InputType::Chat, {2.0f, 5.0f}},            // 2/sec, burst 5
    {InputType::Purchase, {1.0f, 3.0f}},        // 1/sec, burst 3
};
```

### 8.2 Command Rate Limiting

```cpp
class CommandRateLimiter {
public:
    bool CanExecuteCommand(PlayerId player, CommandType command) {
        auto& history = m_commandHistory[player];

        // Get rate limit for command type
        auto limit = GetCommandLimit(command);

        // Count recent commands of this type
        auto cutoff = Clock::Now() - limit.windowSeconds;
        int recentCount = 0;

        for (const auto& record : history) {
            if (record.type == command && record.time > cutoff) {
                recentCount++;
            }
        }

        if (recentCount >= limit.maxCount) {
            return false;
        }

        // Record this command
        history.push_back({command, Clock::Now()});

        // Cleanup old records
        CleanupHistory(player);

        return true;
    }

private:
    struct CommandLimit {
        int maxCount;
        float windowSeconds;
    };

    CommandLimit GetCommandLimit(CommandType command) {
        switch (command) {
            case CommandType::Spawn:
                return {1, 5.0f};   // 1 per 5 seconds
            case CommandType::Respawn:
                return {1, 10.0f};  // 1 per 10 seconds
            case CommandType::Report:
                return {3, 60.0f};  // 3 per minute
            default:
                return {10, 1.0f};  // 10 per second default
        }
    }
};
```

---

## 9. Violation Detection

### 9.1 Violation Handler

```cpp
class ViolationHandler {
public:
    void Process(PlayerId player, const ValidationResult& result) {
        auto& playerViolations = m_violations[player];

        for (const auto& violation : result.violations) {
            // Record violation
            playerViolations.history.push_back(violation);

            // Add to score
            float weight = GetViolationWeight(violation.type);
            playerViolations.score += weight;

            // Log violation
            LogViolation(player, violation);

            // Check for immediate action
            if (RequiresImmediateAction(violation.type)) {
                TakeImmediateAction(player, violation);
            }
        }

        // Decay score over time
        DecayScore(playerViolations);

        // Check thresholds
        CheckThresholds(player, playerViolations);
    }

private:
    float GetViolationWeight(ViolationType type) {
        switch (type) {
            // Severe violations
            case ViolationType::Teleport:
                return 25.0f;
            case ViolationType::DamageHack:
                return 30.0f;
            case ViolationType::CurrencyHack:
                return 50.0f;
            case ViolationType::InventoryManipulation:
                return 40.0f;

            // Moderate violations
            case ViolationType::SpeedHack:
                return 15.0f;
            case ViolationType::FlyHack:
                return 20.0f;
            case ViolationType::NoClip:
                return 20.0f;
            case ViolationType::CooldownBypass:
                return 15.0f;
            case ViolationType::FireRateExceeded:
                return 10.0f;

            // Minor violations
            case ViolationType::RateLimitExceeded:
                return 5.0f;
            case ViolationType::OutOfBounds:
                return 5.0f;
            case ViolationType::AimbotSuspicion:
                return 8.0f;

            // Possible network issues
            case ViolationType::StateDesync:
                return 2.0f;
            case ViolationType::TimestampManipulation:
                return 10.0f;

            default:
                return 5.0f;
        }
    }

    void CheckThresholds(PlayerId player, PlayerViolations& violations) {
        // Warning threshold
        if (violations.score >= WARNING_THRESHOLD && !violations.warned) {
            SendWarning(player);
            violations.warned = true;
        }

        // Flag for review threshold
        if (violations.score >= FLAG_THRESHOLD && !violations.flagged) {
            FlagForReview(player, violations);
            violations.flagged = true;
        }

        // Kick threshold
        if (violations.score >= KICK_THRESHOLD) {
            KickPlayer(player, "Too many violations");
        }

        // Immediate ban threshold
        if (violations.score >= BAN_THRESHOLD) {
            BanPlayer(player, "Severe violations detected");
        }
    }

    void DecayScore(PlayerViolations& violations) {
        float elapsed = Clock::DurationSeconds(violations.lastDecay, Clock::Now());
        violations.lastDecay = Clock::Now();

        // Decay rate: 1 point per second
        violations.score = std::max(0.0f, violations.score - elapsed * DECAY_RATE);
    }

    void FlagForReview(PlayerId player, const PlayerViolations& violations) {
        ReviewRequest request;
        request.playerId = player;
        request.violationScore = violations.score;
        request.recentViolations = GetRecentViolations(violations, 50);
        request.matchId = m_currentMatchId;
        request.timestamp = Clock::Now();

        m_reviewQueue->Submit(request);

        LOG_WARNING("Player {} flagged for review (score: {})",
            player, violations.score);
    }

    static constexpr float WARNING_THRESHOLD = 30.0f;
    static constexpr float FLAG_THRESHOLD = 50.0f;
    static constexpr float KICK_THRESHOLD = 100.0f;
    static constexpr float BAN_THRESHOLD = 200.0f;
    static constexpr float DECAY_RATE = 1.0f;
};
```

### 9.2 Pattern Detection

```cpp
class PatternDetector {
public:
    void AnalyzePlayer(PlayerId player) {
        // Aim analysis
        AnalyzeAimPatterns(player);

        // Movement analysis
        AnalyzeMovementPatterns(player);

        // Timing analysis
        AnalyzeTimingPatterns(player);

        // Statistical analysis
        AnalyzeStatistics(player);
    }

private:
    void AnalyzeAimPatterns(PlayerId player) {
        auto& aimHistory = m_aimHistory[player];

        // Check for inhuman accuracy
        float accuracy = CalculateAccuracy(aimHistory);
        if (accuracy > INHUMAN_ACCURACY_THRESHOLD) {
            FlagSuspicious(player, "Inhuman accuracy", accuracy);
        }

        // Check for snap aiming (instant target acquisition)
        int snapCount = CountSnapAims(aimHistory);
        if (snapCount > SNAP_AIM_THRESHOLD) {
            FlagSuspicious(player, "Snap aiming detected", snapCount);
        }

        // Check for perfect tracking
        float trackingVariance = CalculateTrackingVariance(aimHistory);
        if (trackingVariance < MIN_HUMAN_VARIANCE) {
            FlagSuspicious(player, "Perfect tracking", trackingVariance);
        }

        // Check headshot ratio
        float headshotRatio = CalculateHeadshotRatio(aimHistory);
        if (headshotRatio > SUSPICIOUS_HEADSHOT_RATIO) {
            FlagSuspicious(player, "High headshot ratio", headshotRatio);
        }
    }

    void AnalyzeMovementPatterns(PlayerId player) {
        auto& moveHistory = m_movementHistory[player];

        // Check for frame-perfect inputs
        int perfectInputs = CountFramePerfectInputs(moveHistory);
        if (perfectInputs > PERFECT_INPUT_THRESHOLD) {
            FlagSuspicious(player, "Frame-perfect inputs", perfectInputs);
        }

        // Check for impossible reactions
        auto reactionTimes = CalculateReactionTimes(moveHistory);
        float avgReaction = Average(reactionTimes);
        if (avgReaction < MIN_HUMAN_REACTION) {
            FlagSuspicious(player, "Inhuman reaction time", avgReaction);
        }
    }

    void AnalyzeStatistics(PlayerId player) {
        auto stats = m_statsService->GetPlayerStats(player);

        // Compare to population averages
        float kdRatio = stats.kills / std::max(1.0f, stats.deaths);
        float avgKd = m_populationStats.averageKdRatio;
        float stdDev = m_populationStats.kdStdDev;

        // Flag if significantly above average (> 3 std devs)
        if (kdRatio > avgKd + 3 * stdDev) {
            FlagSuspicious(player, "Statistical anomaly (K/D)", kdRatio);
        }

        // Check win rate
        float winRate = stats.wins / std::max(1.0f, stats.matches);
        if (winRate > SUSPICIOUS_WIN_RATE) {
            FlagSuspicious(player, "Suspicious win rate", winRate);
        }
    }

    static constexpr float INHUMAN_ACCURACY_THRESHOLD = 0.95f;
    static constexpr int SNAP_AIM_THRESHOLD = 10;
    static constexpr float MIN_HUMAN_VARIANCE = 0.5f;
    static constexpr float SUSPICIOUS_HEADSHOT_RATIO = 0.7f;
    static constexpr int PERFECT_INPUT_THRESHOLD = 20;
    static constexpr float MIN_HUMAN_REACTION = 0.1f;  // 100ms
    static constexpr float SUSPICIOUS_WIN_RATE = 0.8f;
};
```

---

## 10. Logging and Forensics

### 10.1 Audit Logger

```cpp
class AuditLogger {
public:
    void LogViolation(
        PlayerId player,
        const Violation& violation,
        const ValidationContext& context
    ) {
        AuditEntry entry;
        entry.timestamp = Clock::Now();
        entry.matchId = m_currentMatchId;
        entry.playerId = player;
        entry.type = AuditType::Violation;

        entry.data = {
            {"violation_type", ToString(violation.type)},
            {"severity", GetSeverity(violation.type)},
            {"details", violation.details},
            {"player_position", context.playerPosition.ToString()},
            {"server_tick", context.serverTick},
            {"client_tick", context.clientTick},
            {"player_state", SerializePlayerState(context.playerState)}
        };

        WriteEntry(entry);
    }

    void LogDamage(
        PlayerId attacker,
        PlayerId target,
        const HitData& hit,
        const DamageResult& result
    ) {
        AuditEntry entry;
        entry.timestamp = Clock::Now();
        entry.matchId = m_currentMatchId;
        entry.playerId = attacker;
        entry.type = AuditType::Combat;

        entry.data = {
            {"target_id", target},
            {"weapon_id", hit.weaponId},
            {"hit_location", ToString(hit.location)},
            {"damage_dealt", result.totalDamage},
            {"target_health_after", result.targetHealth},
            {"eliminated", result.eliminated},
            {"attacker_position", hit.attackerPosition.ToString()},
            {"target_position", hit.targetPosition.ToString()},
            {"distance", hit.distance}
        };

        WriteEntry(entry);
    }

    void LogAbility(
        PlayerId player,
        AbilityId ability,
        const Vector3& target,
        const AbilityResult& result
    ) {
        AuditEntry entry;
        entry.timestamp = Clock::Now();
        entry.matchId = m_currentMatchId;
        entry.playerId = player;
        entry.type = AuditType::Ability;

        entry.data = {
            {"ability_id", ability},
            {"target_position", target.ToString()},
            {"success", result.success},
            {"cooldown_started", result.cooldownStarted}
        };

        WriteEntry(entry);
    }

    void LogTransaction(
        PlayerId player,
        const Transaction& txn,
        const TransactionResult& result
    ) {
        AuditEntry entry;
        entry.timestamp = Clock::Now();
        entry.matchId = m_currentMatchId;
        entry.playerId = player;
        entry.type = AuditType::Economy;

        entry.data = {
            {"transaction_type", ToString(txn.type)},
            {"currency", ToString(txn.currency)},
            {"amount", txn.amount},
            {"balance_before", result.balanceBefore},
            {"balance_after", result.balanceAfter},
            {"item_id", txn.itemId},
            {"success", result.success}
        };

        WriteEntry(entry);
    }

private:
    void WriteEntry(const AuditEntry& entry) {
        // Write to local buffer
        m_buffer.push_back(entry);

        // Flush periodically
        if (m_buffer.size() >= BUFFER_SIZE || ShouldFlush()) {
            Flush();
        }
    }

    void Flush() {
        // Batch write to audit storage
        m_auditStorage->WriteBatch(m_buffer);
        m_buffer.clear();
        m_lastFlush = Clock::Now();
    }

    static constexpr size_t BUFFER_SIZE = 100;
};
```

### 10.2 Replay Data

```cpp
class ForensicReplay {
public:
    void RecordFrame(uint64_t tick) {
        ReplayFrame frame;
        frame.tick = tick;
        frame.serverTime = m_currentTime;

        // Record all player states
        for (const auto& [id, player] : m_world->GetPlayers()) {
            frame.playerStates[id] = SerializeFullPlayerState(player);
        }

        // Record all inputs this tick
        frame.inputs = m_currentTickInputs;

        // Record validation results
        frame.validationResults = m_currentTickValidations;

        // Record events
        frame.events = m_currentTickEvents;

        m_frames.push_back(frame);

        // Limit memory usage
        while (m_frames.size() > MAX_FRAMES) {
            m_frames.erase(m_frames.begin());
        }
    }

    void SaveMatchReplay(const std::string& matchId) {
        MatchReplay replay;
        replay.matchId = matchId;
        replay.version = REPLAY_VERSION;
        replay.frames = m_frames;
        replay.matchResult = m_matchResult;

        // Compress and save
        auto compressed = Compress(replay);
        m_storage->Save(matchId, compressed);
    }

    MatchReplay LoadReplay(const std::string& matchId) {
        auto compressed = m_storage->Load(matchId);
        return Decompress<MatchReplay>(compressed);
    }

    // Analyze replay for investigation
    std::vector<SuspiciousEvent> AnalyzeReplay(const std::string& matchId) {
        auto replay = LoadReplay(matchId);
        std::vector<SuspiciousEvent> events;

        for (const auto& frame : replay.frames) {
            // Check each player's actions
            for (const auto& [playerId, state] : frame.playerStates) {
                auto suspicious = AnalyzePlayerFrame(playerId, frame);
                events.insert(events.end(), suspicious.begin(), suspicious.end());
            }
        }

        return events;
    }

private:
    static constexpr size_t MAX_FRAMES = 36000;  // 30 minutes at 20Hz
    static constexpr int REPLAY_VERSION = 1;
};
```

---

## 11. False Positive Prevention

### 11.1 Tolerance Handling

```cpp
class FalsePositivePrevention {
public:
    bool ShouldApplyViolation(
        PlayerId player,
        ViolationType type,
        const ValidationContext& context
    ) {
        // Check network conditions
        if (HasPoorNetworkConditions(player)) {
            // Increase tolerance for network-related violations
            if (IsNetworkRelatedViolation(type)) {
                return false;  // Don't apply violation
            }
        }

        // Check for legitimate edge cases
        if (IsLegitimateEdgeCase(type, context)) {
            return false;
        }

        // Check recent player history
        auto& history = m_playerHistory[player];
        if (history.recentViolations == 0) {
            // First violation - give benefit of doubt for minor issues
            if (GetSeverity(type) == Severity::Minor) {
                return false;
            }
        }

        return true;
    }

private:
    bool HasPoorNetworkConditions(PlayerId player) {
        auto& conn = m_connectionManager->GetConnection(player);

        // High latency
        if (conn.smoothedRtt > HIGH_LATENCY_THRESHOLD) {
            return true;
        }

        // High packet loss
        if (conn.packetLossRate > HIGH_PACKET_LOSS_THRESHOLD) {
            return true;
        }

        // High jitter
        if (conn.jitter > HIGH_JITTER_THRESHOLD) {
            return true;
        }

        // Recent reconnect
        if (conn.timeSinceReconnect < RECONNECT_GRACE_PERIOD) {
            return true;
        }

        return false;
    }

    bool IsLegitimateEdgeCase(ViolationType type, const ValidationContext& ctx) {
        switch (type) {
            case ViolationType::SpeedHack:
                // Check for launch pads, abilities, etc.
                if (ctx.playerState.HasStatusEffect(StatusEffectType::SpeedBoost)) {
                    return true;
                }
                // Check for knockback
                if (ctx.recentDamage.hasKnockback) {
                    return true;
                }
                break;

            case ViolationType::FlyHack:
                // Check for jump pads
                if (ctx.nearbyEntities.HasJumpPad()) {
                    return true;
                }
                // Check for movement abilities
                if (ctx.recentAbilities.HasMovementAbility()) {
                    return true;
                }
                break;

            case ViolationType::NoClip:
                // Physics edge cases
                if (ctx.physicsSolver.HasCollisionEdgeCase()) {
                    return true;
                }
                break;
        }

        return false;
    }

    static constexpr float HIGH_LATENCY_THRESHOLD = 200.0f;
    static constexpr float HIGH_PACKET_LOSS_THRESHOLD = 0.1f;
    static constexpr float HIGH_JITTER_THRESHOLD = 50.0f;
    static constexpr float RECONNECT_GRACE_PERIOD = 5.0f;
};
```

### 11.2 Adaptive Thresholds

```cpp
class AdaptiveThresholds {
public:
    float GetSpeedTolerance(PlayerId player) {
        float baseTolerance = BASE_SPEED_TOLERANCE;

        // Adjust for network conditions
        float rtt = GetPlayerRTT(player);
        if (rtt > 100.0f) {
            baseTolerance += (rtt - 100.0f) / 1000.0f;  // +0.1% per 10ms over 100
        }

        // Adjust for recent violations
        auto& history = m_playerHistory[player];
        if (history.recentViolations > 0) {
            baseTolerance *= 0.9f;  // Tighter tolerance for suspicious players
        }

        return std::clamp(baseTolerance, MIN_TOLERANCE, MAX_TOLERANCE);
    }

    float GetDamageTolerance(PlayerId player) {
        float baseTolerance = BASE_DAMAGE_TOLERANCE;

        // Tighter tolerance for damage (critical)
        auto& history = m_playerHistory[player];
        if (history.damageViolations > 0) {
            baseTolerance = 1.0f;  // No tolerance for repeat offenders
        }

        return baseTolerance;
    }

private:
    static constexpr float BASE_SPEED_TOLERANCE = 1.15f;
    static constexpr float BASE_DAMAGE_TOLERANCE = 1.05f;
    static constexpr float MIN_TOLERANCE = 1.05f;
    static constexpr float MAX_TOLERANCE = 1.3f;
};
```

---

## 12. Testing Framework

### 12.1 Cheat Simulation Tests

```cpp
class CheatSimulationTests {
public:
    void RunAllTests() {
        TestSpeedHack();
        TestTeleportHack();
        TestFlyHack();
        TestNoClip();
        TestDamageHack();
        TestAimbot();
        TestCooldownBypass();
        TestCurrencyHack();
        TestInventoryManipulation();
        TestRateLimitBypass();
    }

private:
    void TestSpeedHack() {
        // Simulate player moving faster than possible
        SimulatedPlayer player;
        player.SetPosition({0, 0, 0});

        // Move 50 units in one tick (impossible)
        player.SendMovementInput({50, 0, 0});

        // Verify violation detected
        auto result = m_validator->ValidateAndProcess(player.id, player.input, TICK_DELTA);

        ASSERT_TRUE(result.HasViolation(ViolationType::SpeedHack));
        ASSERT_TRUE(result.HasCorrection(CorrectionType::Position));

        // Verify player was corrected
        auto correctedPos = result.GetCorrectedPosition();
        float maxDistance = PLAYER_BASE_SPEED * TICK_DELTA * SPEED_TOLERANCE;
        ASSERT_LE((correctedPos - player.previousPosition).Length(), maxDistance);
    }

    void TestDamageHack() {
        // Simulate player claiming impossible damage
        SimulatedPlayer attacker;
        SimulatedPlayer target;

        // Claim 1000 damage (max possible is ~100)
        HitClaim claim;
        claim.targetId = target.id;
        claim.damage = 1000.0f;
        claim.weapon = WeaponId::Pistol;

        attacker.SendHitClaim(claim);

        auto result = m_validator->ValidateAndProcess(
            attacker.id, attacker.input, TICK_DELTA
        );

        ASSERT_TRUE(result.HasViolation(ViolationType::DamageHack));
        ASSERT_TRUE(result.IsActionBlocked(ActionType::Combat));
    }

    void TestAimbot() {
        // Simulate suspiciously perfect aim
        SimulatedPlayer player;

        // Perfect headshots 50 times in a row
        for (int i = 0; i < 50; i++) {
            HitClaim claim;
            claim.hitLocation = HitLocation::Head;
            claim.aimDeviation = 0.0f;  // Perfect aim

            player.SendHitClaim(claim);
            m_validator->ValidateAndProcess(player.id, player.input, TICK_DELTA);
        }

        // Check pattern detection flagged this
        ASSERT_TRUE(m_patternDetector->IsFlagged(player.id));
        ASSERT_TRUE(m_patternDetector->HasFlag(player.id, "Inhuman accuracy"));
    }

    void TestCooldownBypass() {
        // Simulate using ability faster than cooldown
        SimulatedPlayer player;
        player.abilities[0] = {AbilityId::Fireball, 10.0f};  // 10s cooldown

        // Use ability
        player.SendAbilityInput(0, Vector3::Zero);
        m_validator->ValidateAndProcess(player.id, player.input, TICK_DELTA);

        // Immediately use again (should fail)
        player.SendAbilityInput(0, Vector3::Zero);
        auto result = m_validator->ValidateAndProcess(
            player.id, player.input, TICK_DELTA
        );

        ASSERT_TRUE(result.HasViolation(ViolationType::CooldownBypass));
        ASSERT_TRUE(result.IsActionBlocked(ActionType::Ability));
    }
};
```

### 12.2 False Positive Tests

```cpp
class FalsePositiveTests {
public:
    void RunAllTests() {
        TestHighLatencyPlayer();
        TestPacketLossScenario();
        TestLegitimateMovementAbilities();
        TestEdgeCaseCollisions();
        TestNetworkJitter();
    }

private:
    void TestHighLatencyPlayer() {
        // Simulate high latency player (200ms RTT)
        SimulatedPlayer player;
        player.SetLatency(200.0f);

        // Slightly higher than normal speed (within network tolerance)
        float speedWithLatencyTolerance = PLAYER_BASE_SPEED * 1.1f;
        Vector3 movement = Vector3::Forward * speedWithLatencyTolerance * TICK_DELTA;
        player.SendMovementInput(movement);

        auto result = m_validator->ValidateAndProcess(
            player.id, player.input, TICK_DELTA
        );

        // Should NOT flag as violation due to network tolerance
        ASSERT_FALSE(result.HasViolation(ViolationType::SpeedHack));
    }

    void TestLegitimateMovementAbilities() {
        // Test dash ability doesn't trigger speed hack
        SimulatedPlayer player;
        player.abilities[0] = {AbilityId::Dash, 5.0f};

        // Cast dash
        player.SendAbilityInput(0, Vector3::Forward);
        m_validator->ValidateAndProcess(player.id, player.input, TICK_DELTA);

        // Move fast (dash effect active)
        Vector3 dashMovement = Vector3::Forward * 30.0f * TICK_DELTA;  // 30 units/s
        player.SendMovementInput(dashMovement);

        auto result = m_validator->ValidateAndProcess(
            player.id, player.input, TICK_DELTA
        );

        // Should NOT flag as speed hack
        ASSERT_FALSE(result.HasViolation(ViolationType::SpeedHack));
    }
};
```

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Security Lead | _____________ | _____________ | ___/___/___ |
| Backend Lead | _____________ | _____________ | ___/___/___ |
| Game Director | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial server-authoritative validation specification |

---

*This document contains security-sensitive implementation details. Handle according to company security policies.*
