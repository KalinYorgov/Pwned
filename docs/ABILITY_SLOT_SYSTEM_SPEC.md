# Ability Slot System Specification

## Document Information
- **Task ID:** GAME-007
- **Priority:** P0
- **Complexity:** M
- **Last Updated:** 2025-12-02
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document specifies the ability slot system for Plunderstorm Mobile. Based on Plunderstorm's design, players have access to a basic attack plus 4 ability slots (2 offensive, 2 utility) that they fill by collecting abilities during the match.

### 1.2 Scope
- Slot layout and categorization
- Basic attack (always available)
- Ability pickup and auto-equip
- Slot swapping mechanics
- UI representation
- Cooldown system integration
- Mobile touch controls

### 1.3 Dependencies
- UX-006: Ability UI Design
- GAME-008: Core Ability Implementation
- GAME-004: Loot Spawn System

---

## 2. Slot Architecture

### 2.1 Slot Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    ABILITY SLOT LAYOUT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Player starts with:                                             │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │   BASIC ATTACK (Always Available)                        │    │
│  │   ┌─────────┐                                            │    │
│  │   │   ⚔️    │  No slot required                          │    │
│  │   │  Basic  │  Always ready (minimal cooldown)           │    │
│  │   │  Attack │                                            │    │
│  │   └─────────┘                                            │    │
│  │                                                          │    │
│  │   OFFENSIVE SLOTS (2)             UTILITY SLOTS (2)      │    │
│  │   ┌─────────┐ ┌─────────┐        ┌─────────┐ ┌─────────┐│    │
│  │   │  🔴1    │ │  🔴2    │        │  🔵1    │ │  🔵2    ││    │
│  │   │ [Empty] │ │ [Empty] │        │ [Empty] │ │ [Empty] ││    │
│  │   │         │ │         │        │         │ │         ││    │
│  │   └─────────┘ └─────────┘        └─────────┘ └─────────┘│    │
│  │                                                          │    │
│  │   Total: 4 ability slots + 1 basic attack                │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Slot Types

| Slot Type | Count | Border Color | Accepted Abilities |
|-----------|-------|--------------|-------------------|
| Basic Attack | 1 | N/A | Fixed, no swap |
| Offensive | 2 | Red | Damage-dealing abilities |
| Utility | 2 | Blue | Movement, healing, CC abilities |

### 2.3 Data Structures

```javascript
class AbilitySlot {
    constructor(type, index) {
        this.id = `${type}_${index}`;
        this.type = type;           // 'OFFENSIVE' or 'UTILITY'
        this.index = index;         // 0 or 1
        this.ability = null;        // Currently equipped ability
        this.cooldownRemaining = 0; // Seconds until ready
        this.isLocked = false;      // For tutorial/restrictions
    }
}

class PlayerAbilityLoadout {
    constructor() {
        // Basic attack is always available
        this.basicAttack = new BasicAttack();

        // 4 ability slots
        this.slots = [
            new AbilitySlot('OFFENSIVE', 0),
            new AbilitySlot('OFFENSIVE', 1),
            new AbilitySlot('UTILITY', 0),
            new AbilitySlot('UTILITY', 1)
        ];
    }

    getSlotsByType(type) {
        return this.slots.filter(s => s.type === type);
    }

    getFirstEmptySlot(type) {
        return this.slots.find(s => s.type === type && s.ability === null);
    }
}
```

---

## 3. Basic Attack

### 3.1 Basic Attack Properties

| Property | Value | Notes |
|----------|-------|-------|
| Availability | Always | Never removed or replaced |
| Cooldown | 0.8s | Fast attack rate |
| Damage | 15 | Base damage, no scaling |
| Range | 15m | Medium range |
| Type | Projectile | Straight-line shot |

### 3.2 Basic Attack Behavior

```javascript
class BasicAttack {
    static COOLDOWN = 0.8;
    static DAMAGE = 15;
    static RANGE = 15;
    static PROJECTILE_SPEED = 50;

    constructor() {
        this.cooldownRemaining = 0;
        this.autoAimAssist = true; // Can be toggled
    }

    canUse() {
        return this.cooldownRemaining <= 0;
    }

    use(player, target) {
        if (!this.canUse()) return { success: false };

        // Fire projectile
        const projectile = new Projectile({
            damage: DAMAGE,
            speed: PROJECTILE_SPEED,
            direction: this.calculateDirection(player, target)
        });

        // Start cooldown
        this.cooldownRemaining = COOLDOWN;

        return { success: true, projectile };
    }

    calculateDirection(player, target) {
        if (this.autoAimAssist && target) {
            // Slight aim assist toward nearest enemy
            return this.applyAimAssist(player.aimDirection, target);
        }
        return player.aimDirection;
    }
}
```

---

## 4. Ability Pickup Flow

### 4.1 Pickup Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│                    ABILITY PICKUP FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Player picks up ability                                         │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────────────┐                                           │
│  │ Determine Ability│                                           │
│  │ Slot Type        │  Offensive or Utility?                    │
│  └────────┬─────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ Check for Empty  │                                           │
│  │ Slot of Type     │                                           │
│  └────────┬─────────┘                                           │
│           │                                                      │
│     ┌─────┴─────┐                                               │
│     ▼           ▼                                               │
│  [Empty]     [Full]                                             │
│     │           │                                               │
│     ▼           ▼                                               │
│  ┌────────┐  ┌──────────────────┐                               │
│  │ Auto   │  │ Check for Same   │                               │
│  │ Equip  │  │ Ability (Upgrade)│                               │
│  └────────┘  └────────┬─────────┘                               │
│                       │                                          │
│                 ┌─────┴─────┐                                    │
│                 ▼           ▼                                    │
│              [Same]      [Different]                             │
│                 │           │                                    │
│                 ▼           ▼                                    │
│           ┌──────────┐  ┌──────────┐                            │
│           │ Upgrade  │  │ Show Swap│                            │
│           │ Ability  │  │ Prompt   │                            │
│           └──────────┘  └──────────┘                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Auto-Equip Logic

```javascript
class AbilityEquipManager {
    handlePickup(player, ability) {
        const slotType = ability.slotType;
        const matchingSlots = player.loadout.getSlotsByType(slotType);

        // Step 1: Check for empty slot
        const emptySlot = matchingSlots.find(s => s.ability === null);
        if (emptySlot) {
            this.equipToSlot(player, emptySlot, ability);
            this.playEquipFeedback(player, 'auto_equip');
            return;
        }

        // Step 2: Check for upgrade opportunity
        const upgradeSlot = matchingSlots.find(
            s => s.ability?.id === ability.id && s.ability.rank < 3
        );
        if (upgradeSlot) {
            this.upgradeAbility(player, upgradeSlot);
            this.playEquipFeedback(player, 'upgrade');
            return;
        }

        // Step 3: All slots full, show swap UI
        this.showSwapPrompt(player, ability, matchingSlots);
    }

    equipToSlot(player, slot, ability) {
        slot.ability = ability;
        slot.cooldownRemaining = 0; // Ready immediately

        // Network sync
        this.broadcastLoadoutUpdate(player);

        // UI update
        this.updateAbilityUI(player, slot);
    }
}
```

### 4.3 Equip Feedback

| Action | Visual | Audio | Duration |
|--------|--------|-------|----------|
| Auto-Equip | Slot glows, ability icon appears | "Whoosh" equip sound | 0.3s |
| Upgrade | Rank stars animate, power burst | Level-up chime | 0.5s |
| Swap Prompt | UI overlay appears | Alert sound | Until decision |

---

## 5. Slot Swapping

### 5.1 Swap Prompt UI

```
┌─────────────────────────────────────────────────────────────────┐
│                    SWAP PROMPT UI                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │              REPLACE ABILITY?                           │     │
│  │                                                         │     │
│  │   Select which ability to replace:                      │     │
│  │                                                         │     │
│  │   ┌───────────────────┐    ┌───────────────────┐       │     │
│  │   │     SLOT 1        │    │     SLOT 2        │       │     │
│  │   │   ┌─────────┐     │    │   ┌─────────┐     │       │     │
│  │   │   │  🔥     │     │    │   │  ⚡     │     │       │     │
│  │   │   │Fireball │     │    │   │Lightning│     │       │     │
│  │   │   │ ★★☆    │     │    │   │ ★☆☆    │     │       │     │
│  │   │   └─────────┘     │    │   └─────────┘     │       │     │
│  │   │                   │    │                   │       │     │
│  │   │  [TAP TO SWAP]    │    │  [TAP TO SWAP]    │       │     │
│  │   └───────────────────┘    └───────────────────┘       │     │
│  │                                                         │     │
│  │   New Ability: Cannonball ★★★                          │     │
│  │                                                         │     │
│  │              [KEEP CURRENT - DROP NEW]                  │     │
│  │                                                         │     │
│  │   ⏱️ Auto-dismiss in 5s                                 │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Swap Mechanics

```javascript
class SwapManager {
    static SWAP_TIMEOUT = 5;           // Seconds before auto-dismiss
    static DROPPED_ABILITY_LIFETIME = 30;

    showSwapPrompt(player, newAbility, slots) {
        // Pause gameplay? No - allow combat during decision
        player.activeSwapPrompt = {
            newAbility: newAbility,
            slots: slots,
            startTime: Time.now(),
            timeout: SWAP_TIMEOUT
        };

        // Send UI to client
        this.sendSwapPromptUI(player, newAbility, slots);

        // Start timeout
        this.scheduleAutoDismiss(player, SWAP_TIMEOUT);
    }

    onSwapDecision(player, selectedSlot) {
        const prompt = player.activeSwapPrompt;
        if (!prompt) return;

        if (selectedSlot === null) {
            // Player chose to keep current abilities
            this.dropAbility(prompt.newAbility, player.position);
        } else {
            // Swap: drop old, equip new
            const oldAbility = selectedSlot.ability;
            this.dropAbility(oldAbility, player.position);
            this.equipToSlot(player, selectedSlot, prompt.newAbility);
        }

        // Clear prompt
        player.activeSwapPrompt = null;
        this.closeSwapPromptUI(player);
    }

    dropAbility(ability, position) {
        // Create ground pickup
        const groundAbility = new GroundAbility({
            ability: ability,
            position: position,
            despawnTime: Time.now() + DROPPED_ABILITY_LIFETIME
        });

        // Spawn with slight bounce
        groundAbility.applyDropPhysics();

        return groundAbility;
    }
}
```

### 5.3 Swap During Combat

- Player CAN be attacked while swap prompt is open
- Taking damage does NOT auto-dismiss prompt
- Player can use existing abilities while prompt is open
- Swap prompt is semi-transparent to maintain visibility

---

## 6. Cooldown System

### 6.1 Cooldown Display

```
┌─────────────────────────────────────────────────────────────────┐
│                    COOLDOWN VISUALIZATION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Ready State:                    On Cooldown:                    │
│  ┌─────────────┐                ┌─────────────┐                 │
│  │   ┌─────┐   │                │   ┌─────┐   │                 │
│  │   │ 🔥  │   │                │   │ 🔥  │   │  Dark overlay   │
│  │   │     │   │                │   │▓▓▓▓▓│   │  fills clock-   │
│  │   │     │   │                │   │▓▓▓▓▓│   │  wise as CD     │
│  │   └─────┘   │                │   └─────┘   │  progresses     │
│  │             │                │    3.2s     │  Seconds shown  │
│  └─────────────┘                └─────────────┘                 │
│                                                                  │
│  Low Cooldown Warning:                                           │
│  ┌─────────────┐                                                │
│  │   ┌─────┐   │  When < 1s remaining:                          │
│  │   │ 🔥  │   │  - Icon pulses                                 │
│  │   │  ░░ │   │  - Glow effect appears                         │
│  │   │     │   │  - "Ready" sound plays at 0                    │
│  │   └─────┘   │                                                │
│  │    0.3s     │                                                │
│  └─────────────┘                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Cooldown Implementation

```javascript
class CooldownManager {
    updateCooldowns(player, deltaTime) {
        // Basic attack cooldown
        if (player.basicAttack.cooldownRemaining > 0) {
            player.basicAttack.cooldownRemaining -= deltaTime;
        }

        // Ability slot cooldowns
        for (const slot of player.loadout.slots) {
            if (slot.cooldownRemaining > 0) {
                slot.cooldownRemaining -= deltaTime;

                // Trigger ready feedback when cooldown ends
                if (slot.cooldownRemaining <= 0) {
                    slot.cooldownRemaining = 0;
                    this.onAbilityReady(player, slot);
                }
            }
        }
    }

    startCooldown(slot, ability) {
        // Apply cooldown reduction from player stats
        const baseCooldown = ability.cooldown;
        const cdr = slot.owner.stats.cooldownReduction;
        const actualCooldown = baseCooldown * (1 - cdr);

        slot.cooldownRemaining = actualCooldown;
    }

    onAbilityReady(player, slot) {
        // Visual feedback
        this.flashSlotReady(slot);

        // Audio cue (subtle)
        player.playSound('ability_ready', { volume: 0.3 });
    }
}
```

### 6.3 Cooldown Reduction

| Source | CDR Amount | Stack |
|--------|------------|-------|
| Base | 0% | - |
| Level 5 | 5% | - |
| Level 10 | 10% | - |
| Rank 2 Ability | -10% on that ability | Per-ability |
| Rank 3 Ability | -20% on that ability | Per-ability |

---

## 7. UI Design

### 7.1 Mobile Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    MOBILE ABILITY UI LAYOUT                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Screen layout (landscape):                                      │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                     [GAME VIEW]                         │     │
│  │                                                         │     │
│  │                                                         │     │
│  │                                                         │     │
│  │                                                         │     │
│  │                                                         │     │
│  │  ┌─────┐                                    ┌──────────┐│     │
│  │  │Move │                                    │ Ability  ││     │
│  │  │Stick│                                    │  Buttons ││     │
│  │  └─────┘                                    └──────────┘│     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Right side ability buttons (detailed):                          │
│                                                                  │
│  ┌───────────────────────────────────────┐                      │
│  │                                        │                      │
│  │        ┌─────┐  ┌─────┐               │                      │
│  │        │ 🔴1 │  │ 🔵1 │               │  Top: Offensive 1,   │
│  │        │     │  │     │               │       Utility 1      │
│  │        └─────┘  └─────┘               │                      │
│  │                                        │                      │
│  │        ┌─────┐  ┌─────┐    ┌─────┐   │                      │
│  │        │ 🔴2 │  │ 🔵2 │    │ ⚔️  │   │  Bottom: Off 2,      │
│  │        │     │  │     │    │Basic│   │  Util 2, Basic Attack │
│  │        └─────┘  └─────┘    └─────┘   │                      │
│  │                                        │                      │
│  └───────────────────────────────────────┘                      │
│                                                                  │
│  Button size: 60x60 dp (touch-friendly)                          │
│  Spacing: 10 dp between buttons                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Button States

| State | Visual | Behavior |
|-------|--------|----------|
| Empty | Gray border, "+" icon | Indicates slot available |
| Ready | Full color, ability icon | Can be activated |
| On Cooldown | Darkened, timer overlay | Cannot activate, shows time |
| Active | Glowing border | Ability is being used |
| Targeting | Highlighted, shows range | For skillshot abilities |

### 7.3 Slot Interaction

```javascript
class AbilityButtonHandler {
    onButtonTap(player, slotIndex) {
        const slot = player.loadout.slots[slotIndex];

        if (!slot.ability) {
            // Empty slot - no action
            this.showEmptySlotFeedback();
            return;
        }

        if (slot.cooldownRemaining > 0) {
            // On cooldown - show feedback
            this.showCooldownFeedback(slot);
            return;
        }

        // Check ability type
        if (slot.ability.requiresTargeting) {
            // Enter targeting mode
            this.enterTargetingMode(player, slot);
        } else {
            // Instant cast
            this.castAbility(player, slot);
        }
    }

    onButtonHold(player, slotIndex) {
        // Show ability details tooltip
        const slot = player.loadout.slots[slotIndex];
        if (slot.ability) {
            this.showAbilityTooltip(slot.ability);
        }
    }
}
```

---

## 8. Targeting Mode

### 8.1 Targeting UI

```
┌─────────────────────────────────────────────────────────────────┐
│                    TARGETING MODE UI                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For skillshot abilities (e.g., Fireball):                       │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                    [GAME VIEW]                          │     │
│  │                                                         │     │
│  │                         ╱                               │     │
│  │                        ╱                                │     │
│  │          [PLAYER]─────╱  Aim line                       │     │
│  │               ◎       ╲                                 │     │
│  │                        ╲                                │     │
│  │                         ╲                               │     │
│  │                          ⊕  Target indicator            │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  For AoE abilities (e.g., Lightning Strike):                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                                                         │     │
│  │                    [GAME VIEW]                          │     │
│  │                                                         │     │
│  │                    ┌─────────┐                          │     │
│  │                    │  ○○○○○  │  AoE range circle        │     │
│  │          [PLAYER]  │  ○   ○  │                          │     │
│  │               ◎    │  ○ ⊕ ○  │  Draggable target       │     │
│  │                    │  ○   ○  │                          │     │
│  │                    │  ○○○○○  │                          │     │
│  │                    └─────────┘                          │     │
│  │                                                         │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Controls:                                                       │
│  - Drag to aim                                                   │
│  - Release to fire                                               │
│  - Tap elsewhere or wait to cancel                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Targeting Implementation

```javascript
class TargetingMode {
    static MAX_RANGE = 20;       // Ability-specific override
    static CANCEL_TIMEOUT = 3;   // Seconds before auto-cancel

    enter(player, slot) {
        player.targetingState = {
            active: true,
            slot: slot,
            targetPosition: player.position + player.forwardDirection * 5,
            startTime: Time.now()
        };

        // Show targeting UI
        this.showTargetingIndicator(slot.ability);

        // Player can still move while targeting
    }

    updateTargetPosition(player, inputPosition) {
        const state = player.targetingState;
        if (!state?.active) return;

        // Convert screen position to world position
        const worldPos = this.screenToWorld(inputPosition);

        // Clamp to max range
        const direction = (worldPos - player.position).normalized;
        const distance = Math.min(
            Vector3.Distance(player.position, worldPos),
            state.slot.ability.maxRange
        );

        state.targetPosition = player.position + direction * distance;

        // Update indicator
        this.updateIndicator(state.targetPosition);
    }

    confirm(player) {
        const state = player.targetingState;
        if (!state?.active) return;

        // Cast ability at target position
        this.castAbilityAtTarget(player, state.slot, state.targetPosition);

        // Exit targeting mode
        this.exit(player);
    }

    cancel(player) {
        player.targetingState = null;
        this.hideTargetingIndicator();
    }
}
```

---

## 9. Network Synchronization

### 9.1 Loadout Sync Packet

```javascript
// Sent when loadout changes
struct LoadoutUpdatePacket {
    uint32 playerId;
    AbilitySlotData[4] slots;
}

struct AbilitySlotData {
    uint8  slotIndex;      // 0-3
    uint8  slotType;       // OFFENSIVE, UTILITY
    uint8  abilityId;      // 0 = empty
    uint8  abilityRank;    // 1-3
    float  cooldownRemaining;
}
```

### 9.2 Ability Use Packet

```javascript
// Sent when ability is used
struct AbilityUsePacket {
    uint32 playerId;
    uint8  slotIndex;
    float3 targetPosition;  // For targeted abilities
    float  timestamp;
}
```

### 9.3 Sync Events

| Event | Sync To | Priority |
|-------|---------|----------|
| Ability Equip | All (nearby) | Medium |
| Ability Use | All (nearby) | High |
| Cooldown State | Self only | Low |
| Upgrade | All (nearby) | Medium |

---

## 10. Audio Design

### 10.1 Sound Effects

| Sound | Trigger | Notes |
|-------|---------|-------|
| ability_equip | Auto-equip to slot | Satisfying "click" |
| ability_upgrade | Ability ranks up | Power-up chime |
| ability_ready | Cooldown ends | Subtle "ding" |
| ability_on_cooldown | Try to use on CD | Error buzz |
| swap_prompt_open | Swap UI appears | Alert sound |
| slot_empty_tap | Tap empty slot | Hollow tap |

### 10.2 Audio Priorities

| Priority | Sounds |
|----------|--------|
| Critical | Ability use, combat |
| High | Ready notifications |
| Medium | Equip, upgrade |
| Low | UI interactions |

---

## 11. Performance Considerations

### 11.1 Performance Targets

| Metric | Target |
|--------|--------|
| Equip Latency | < 50ms |
| UI Update | < 16ms (60 FPS) |
| Cooldown Accuracy | ±10ms |
| Memory per Loadout | < 1 KB |

### 11.2 Optimization Strategies

```javascript
class LoadoutOptimizer {
    // Pool ability icons for reuse
    iconPool = new ObjectPool(AbilityIcon, 10);

    // Cache cooldown calculations
    cachedCooldowns = new Map();

    // Batch UI updates
    updateUIBatched(slots) {
        // Collect all changes
        const updates = slots.filter(s => s.isDirty);

        // Apply in single frame
        requestAnimationFrame(() => {
            for (const slot of updates) {
                this.updateSlotUI(slot);
                slot.isDirty = false;
            }
        });
    }
}
```

---

## 12. Edge Cases

### 12.1 Pickup During Cast

```javascript
// Player picks up ability while casting another
handlePickupDuringCast(player, newAbility) {
    if (player.isCasting) {
        // Queue the pickup for after cast completes
        player.pendingPickup = newAbility;
    } else {
        this.handlePickup(player, newAbility);
    }
}
```

### 12.2 Disconnect with Full Loadout

- Loadout state saved server-side
- On reconnect, loadout restored
- Cooldowns continue during disconnect
- Pending swaps canceled

### 12.3 Swap Timeout

- After 5 seconds, prompt auto-dismisses
- New ability drops on ground
- Player keeps existing loadout
- Dropped ability can be picked up by anyone

---

## 13. Testing Requirements

### 13.1 Unit Tests

| Test Case | Description |
|-----------|-------------|
| Empty slot equip | Ability fills empty slot |
| Full slots swap | Swap prompt appears |
| Upgrade detection | Same ability triggers upgrade |
| Cooldown tracking | Cooldowns decrease accurately |
| Slot type matching | Offensive → Offensive only |

### 13.2 Integration Tests

| Test Case | Description |
|-----------|-------------|
| Full pickup flow | Chest → Equip → Use |
| Network sync | Loadout consistent across clients |
| Combat while swapping | Can fight during prompt |
| Reconnect recovery | Loadout restored correctly |

### 13.3 Playtest Validation

| Criteria | Target |
|----------|--------|
| Slot clarity | > 95% understand slot types |
| Equip speed | < 0.5s perceived delay |
| Swap usability | > 90% complete swaps successfully |
| Button comfort | < 5% accidental presses |

---

## 14. Appendices

### Appendix A: Ability Type Classification

| Ability | Slot Type | Notes |
|---------|-----------|-------|
| Fireball | Offensive | Ranged damage |
| Cutlass Slash | Offensive | Melee damage |
| Cannonball | Offensive | Heavy damage |
| Lightning Strike | Offensive | AoE damage |
| Poison Mackerel | Offensive | DoT damage |
| Grappling Hook | Utility | Mobility |
| Barrel Roll | Utility | Dodge/invuln |
| Healing Grog | Utility | Self-heal |
| Smoke Bomb | Utility | Vision/slow |
| Wind Burst | Utility | Knockback |

### Appendix B: Button Size Guidelines

| Screen Size | Button Size | Spacing |
|-------------|-------------|---------|
| Small (< 5") | 50x50 dp | 8 dp |
| Medium (5-7") | 60x60 dp | 10 dp |
| Large (> 7") | 70x70 dp | 12 dp |
| Tablet | 80x80 dp | 15 dp |

### Appendix C: Accessibility

| Feature | Implementation |
|---------|----------------|
| Color blind | Shape indicators + color |
| Button size | Minimum 48x48 dp touch target |
| Audio cues | All actions have sound feedback |
| Haptic | Vibration on ability use |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Author: Game Design Team*
