# Practice/Tutorial Mode Specification

## Overview

This document specifies the Practice and Tutorial systems for Plunderstorm Mobile, providing new player onboarding and a safe environment for skill development. The system includes a guided tutorial, free-form training ground, and bot matches.

## System Components

### 1. Interactive Tutorial
Guided introduction for new players covering all core mechanics.

### 2. Training Ground
Free-form sandbox for testing abilities and practicing.

### 3. Bot Match
Full match experience against AI opponents.

---

## Interactive Tutorial

### Tutorial Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    TUTORIAL PROGRESSION                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Welcome] → [Movement] → [Camera] → [Basic Attack]             │
│      │           │           │            │                     │
│      ↓           ↓           ↓            ↓                     │
│  [Ability   → [Ability  → [Combat   → [Storm     → [Complete!] │
│   Pickup]      Use]        Practice]   Awareness]               │
│                                                                  │
│  Estimated Time: 3-5 minutes                                    │
│  Skip Option: Available after first completion                  │
└─────────────────────────────────────────────────────────────────┘
```

### Tutorial Stages

#### Stage 1: Welcome (30 seconds)

```typescript
interface TutorialStage {
  id: string;
  name: string;
  duration: number;        // Estimated seconds
  objectives: Objective[];
  voiceOver?: string;
  skipable: boolean;
}

const STAGE_WELCOME: TutorialStage = {
  id: 'welcome',
  name: 'Welcome to Plunderstorm',
  duration: 30,
  objectives: [],
  voiceOver: 'vo_tutorial_welcome',
  skipable: false
};
```

**Content:**
- Brief intro cinematic (15 seconds)
- "Welcome, pirate! Let's learn the basics."
- Overview of game objective: "Be the last one standing!"

#### Stage 2: Movement (45 seconds)

```typescript
const STAGE_MOVEMENT: TutorialStage = {
  id: 'movement',
  name: 'Movement Controls',
  duration: 45,
  objectives: [
    {
      id: 'move_forward',
      text: 'Move forward using the left joystick',
      completionCondition: 'move_distance_5m',
      highlight: 'left_joystick'
    },
    {
      id: 'move_all_directions',
      text: 'Practice moving in all directions',
      completionCondition: 'move_all_cardinal',
      highlight: 'left_joystick'
    }
  ],
  voiceOver: 'vo_tutorial_movement',
  skipable: false
};
```

**UI Highlights:**
- Virtual joystick pulses with attention indicator
- Arrow shows movement direction
- Checkmark appears on completion

#### Stage 3: Camera Control (30 seconds)

```typescript
const STAGE_CAMERA: TutorialStage = {
  id: 'camera',
  name: 'Camera Controls',
  duration: 30,
  objectives: [
    {
      id: 'rotate_camera',
      text: 'Drag the right side of screen to look around',
      completionCondition: 'rotate_180_degrees',
      highlight: 'camera_zone'
    }
  ],
  voiceOver: 'vo_tutorial_camera',
  skipable: false
};
```

#### Stage 4: Basic Attack (45 seconds)

```typescript
const STAGE_BASIC_ATTACK: TutorialStage = {
  id: 'basic_attack',
  name: 'Basic Attack',
  duration: 45,
  objectives: [
    {
      id: 'aim_target',
      text: 'Aim at the target dummy',
      completionCondition: 'crosshair_on_target',
      highlight: 'crosshair'
    },
    {
      id: 'fire_attack',
      text: 'Tap the attack button to fire',
      completionCondition: 'deal_damage_to_dummy',
      highlight: 'attack_button'
    },
    {
      id: 'destroy_dummy',
      text: 'Destroy the target dummy',
      completionCondition: 'dummy_destroyed',
      highlight: null
    }
  ],
  voiceOver: 'vo_tutorial_attack',
  skipable: false
};
```

**Tutorial Environment:**
- Static target dummy (100 HP, doesn't fight back)
- Clear arena with no obstacles
- Hit marker feedback on successful hits

#### Stage 5: Ability Pickup (45 seconds)

```typescript
const STAGE_ABILITY_PICKUP: TutorialStage = {
  id: 'ability_pickup',
  name: 'Finding Abilities',
  duration: 45,
  objectives: [
    {
      id: 'approach_chest',
      text: 'Walk to the glowing treasure chest',
      completionCondition: 'near_chest_3m',
      highlight: 'chest_marker'
    },
    {
      id: 'open_chest',
      text: 'Tap the chest to open it',
      completionCondition: 'chest_opened',
      highlight: 'interact_prompt'
    },
    {
      id: 'pickup_ability',
      text: 'Pick up the ability that dropped',
      completionCondition: 'ability_equipped',
      highlight: 'loot_item'
    }
  ],
  voiceOver: 'vo_tutorial_loot',
  skipable: false
};
```

#### Stage 6: Ability Use (60 seconds)

```typescript
const STAGE_ABILITY_USE: TutorialStage = {
  id: 'ability_use',
  name: 'Using Abilities',
  duration: 60,
  objectives: [
    {
      id: 'view_ability',
      text: 'Your new ability is shown here',
      completionCondition: 'ability_button_tapped',
      highlight: 'ability_slot_1'
    },
    {
      id: 'aim_ability',
      text: 'Drag to aim the ability at the target',
      completionCondition: 'ability_aimed',
      highlight: 'targeting_indicator'
    },
    {
      id: 'release_ability',
      text: 'Release to cast the ability',
      completionCondition: 'ability_cast',
      highlight: null
    },
    {
      id: 'hit_target',
      text: 'Hit the target with your ability',
      completionCondition: 'ability_hit_target',
      highlight: null
    }
  ],
  voiceOver: 'vo_tutorial_ability',
  skipable: false
};
```

**Ability Given:** Fireball (simple skillshot, easy to understand)

#### Stage 7: Combat Practice (90 seconds)

```typescript
const STAGE_COMBAT: TutorialStage = {
  id: 'combat_practice',
  name: 'Combat Practice',
  duration: 90,
  objectives: [
    {
      id: 'defeat_bot',
      text: 'Defeat the enemy pirate',
      completionCondition: 'enemy_eliminated',
      highlight: 'enemy_marker'
    }
  ],
  voiceOver: 'vo_tutorial_combat',
  skipable: false
};
```

**Environment:**
- Easy bot opponent (50% damage, slower reactions)
- Small arena with cover
- Bot has basic attack only (no abilities)

#### Stage 8: Storm Awareness (45 seconds)

```typescript
const STAGE_STORM: TutorialStage = {
  id: 'storm_awareness',
  name: 'The Storm',
  duration: 45,
  objectives: [
    {
      id: 'observe_storm',
      text: 'Watch the storm approach',
      completionCondition: 'storm_visible',
      highlight: 'storm_indicator'
    },
    {
      id: 'check_minimap',
      text: 'Check the safe zone on your minimap',
      completionCondition: 'minimap_viewed',
      highlight: 'minimap'
    },
    {
      id: 'enter_safe_zone',
      text: 'Move to the safe zone before the storm hits',
      completionCondition: 'player_in_safe_zone',
      highlight: 'safe_zone_marker'
    }
  ],
  voiceOver: 'vo_tutorial_storm',
  skipable: false
};
```

### Tutorial Completion

```typescript
interface TutorialCompletion {
  playerId: string;
  completedAt: Date;
  totalTime: number;         // seconds
  stagesCompleted: string[];
  skipped: boolean;
  reward: TutorialReward;
}

interface TutorialReward {
  goldCoins: 100;
  doubloons: 10;
  cosmetic?: 'tutorial_badge';
}
```

### Tutorial UI Elements

```
┌─────────────────────────────────────────────────────────────────┐
│                      TUTORIAL OVERLAY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │          "Tap the attack button to fire!"               │   │
│  │                         ↓                                │   │
│  │                    [PULSING HIGHLIGHT]                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Progress: ████████░░░░░░░░  Stage 4/8                         │
│                                                                  │
│  [Skip Tutorial - Available after first completion]             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Training Ground

### Training Ground Features

| Feature | Description |
|---------|-------------|
| All Abilities | Access to every ability in the game |
| Target Dummies | Stationary targets for aim practice |
| Moving Targets | Targets that strafe for advanced practice |
| Bot Sparring | Optional AI opponent to fight |
| Ability Rack | UI to spawn any ability instantly |
| Infinite Health | Player can toggle invincibility |
| Reset Button | Instantly reset health and cooldowns |

### Training Ground Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRAINING GROUND MAP                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│     [Target Range]           [Open Arena]        [Obstacle      │
│      ○ ○ ○ ○ ○                                    Course]       │
│      ↑ ↑ ↑ ↑ ↑                  ★                    ▓▓        │
│     Stationary               Player              ▓▓  ▓▓        │
│      Targets                 Spawn               ▓▓▓▓▓▓        │
│                                                                  │
│     [Moving Targets]        [Bot Arena]          [Ability       │
│      → ○ →                    🤖                   Rack]        │
│      ← ○ ←                  Sparring           [All Abilities] │
│                              Partner                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Ability Rack UI

```
┌─────────────────────────────────────────────────────────────────┐
│                      ABILITY RACK                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  OFFENSIVE                           UTILITY                     │
│  ┌─────┐ ┌─────┐ ┌─────┐           ┌─────┐ ┌─────┐ ┌─────┐    │
│  │ 🔥  │ │ ⚔️  │ │ 💣  │           │ 🪝  │ │ 💨  │ │ 💚  │    │
│  │Fire │ │Cut- │ │Can- │           │Grap │ │Wind │ │Heal │    │
│  │ball │ │lass │ │non  │           │ple  │ │Burst│ │Tide │    │
│  └─────┘ └─────┘ └─────┘           └─────┘ └─────┘ └─────┘    │
│                                                                  │
│  Tap ability to equip | Long-press for info                     │
│                                                                  │
│  RANK: [★☆☆] [★★☆] [★★★]  ← Select ability rank               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Training Options

```typescript
interface TrainingOptions {
  infiniteHealth: boolean;
  infiniteCooldowns: boolean;   // No ability cooldowns
  showHitboxes: boolean;        // Debug visualization
  damageNumbers: boolean;
  targetDifficulty: 'stationary' | 'slow' | 'fast';
  botEnabled: boolean;
  botDifficulty: 'easy' | 'medium' | 'hard';
}
```

---

## Bot Match

### Bot Configuration

```typescript
interface BotConfig {
  difficulty: BotDifficulty;
  behaviorProfile: BehaviorProfile;
  aimAccuracy: number;          // 0.0 - 1.0
  reactionTime: number;         // milliseconds
  abilityUsage: number;         // How often they use abilities
  aggressiveness: number;       // 0.0 - 1.0
  movementSkill: number;        // Dodging, positioning
}

enum BotDifficulty {
  EASY = 'easy',
  MEDIUM = 'medium',
  HARD = 'hard'
}
```

### Difficulty Settings

| Setting | Easy | Medium | Hard |
|---------|------|--------|------|
| Aim Accuracy | 40% | 65% | 85% |
| Reaction Time | 800ms | 400ms | 150ms |
| Ability Usage | Low | Medium | High |
| Aggressiveness | 0.3 | 0.5 | 0.8 |
| Movement Skill | Poor | Decent | Advanced |
| Looting Speed | Slow | Normal | Fast |

### Bot Behavior Profiles

```typescript
interface BehaviorProfile {
  name: string;
  playstyle: 'aggressive' | 'defensive' | 'balanced' | 'camper';
  preferredAbilities: string[];
  decisionWeights: {
    fight: number;
    flee: number;
    loot: number;
    heal: number;
  };
}

const BOT_PROFILES: BehaviorProfile[] = [
  {
    name: 'Aggressive',
    playstyle: 'aggressive',
    preferredAbilities: ['cutlass_slash', 'fireball'],
    decisionWeights: { fight: 0.7, flee: 0.1, loot: 0.15, heal: 0.05 }
  },
  {
    name: 'Cautious',
    playstyle: 'defensive',
    preferredAbilities: ['healing_tide', 'smoke_bomb'],
    decisionWeights: { fight: 0.3, flee: 0.3, loot: 0.25, heal: 0.15 }
  },
  {
    name: 'Balanced',
    playstyle: 'balanced',
    preferredAbilities: ['fireball', 'grappling_hook'],
    decisionWeights: { fight: 0.4, flee: 0.2, loot: 0.3, heal: 0.1 }
  }
];
```

### Bot AI Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│                    BOT DECISION TREE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Check Situation]                                               │
│         │                                                        │
│         ├── HP < 30%? ──Yes──► [Find cover / Use heal]          │
│         │                                                        │
│         ├── Enemy nearby? ──Yes──► [Evaluate fight]             │
│         │         │                                              │
│         │         ├── Can win? ──Yes──► [Engage]                │
│         │         └── No ──► [Flee / Find advantage]            │
│         │                                                        │
│         ├── Storm approaching? ──Yes──► [Move to safe zone]     │
│         │                                                        │
│         ├── Loot nearby? ──Yes──► [Loot if safe]                │
│         │                                                        │
│         └── Default ──► [Patrol / Move toward center]           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Bot Match Settings

```typescript
interface BotMatchConfig {
  playerCount: number;         // Total including player
  botDifficulty: BotDifficulty;
  mixedDifficulty: boolean;    // Vary bot difficulties
  stormEnabled: boolean;
  matchDuration: 'normal' | 'quick';  // Quick = faster storm
  rewards: {
    enabled: boolean;
    multiplier: number;        // 0.5x for bot matches
  };
}

// Default bot match: 30 players, mixed difficulty, normal storm
const DEFAULT_BOT_MATCH: BotMatchConfig = {
  playerCount: 30,
  botDifficulty: BotDifficulty.MEDIUM,
  mixedDifficulty: true,
  stormEnabled: true,
  matchDuration: 'normal',
  rewards: {
    enabled: true,
    multiplier: 0.5
  }
};
```

### Bot Name Generator

```typescript
const BOT_NAME_PREFIXES = [
  'Captain', 'First Mate', 'Swashbuckler', 'Scallywag',
  'Buccaneer', 'Corsair', 'Privateer', 'Sea Dog'
];

const BOT_NAME_SUFFIXES = [
  'Jack', 'Anne', 'Morgan', 'Drake', 'Silver',
  'Bones', 'Hook', 'Sparrow', 'Teach', 'Flint'
];

function generateBotName(): string {
  const prefix = randomChoice(BOT_NAME_PREFIXES);
  const suffix = randomChoice(BOT_NAME_SUFFIXES);
  return `${prefix} ${suffix}`;
}

// Examples: "Captain Morgan", "Scallywag Bones", "First Mate Anne"
```

---

## First-Time User Experience

### New Player Flow

```
[First Launch]
     │
     ▼
[Account Creation / Login]
     │
     ▼
[Mandatory Tutorial - Cannot skip]
     │
     ▼
[Tutorial Complete - Receive Rewards]
     │
     ▼
[Main Menu - All modes available]
     │
     ├── [Play Solo/Duo] - Real matches
     ├── [Training Ground] - Practice
     └── [Bot Match] - Safe practice with rewards
```

### Tutorial Skip Conditions

```typescript
interface TutorialSkipRules {
  // Can skip after first completion
  firstCompletion: boolean;

  // Optional: Can skip if linked account shows experience
  linkedAccountLevel?: number;

  // Never skip on completely new accounts
  allowSkipNewAccount: false;
}

function canSkipTutorial(player: Player): boolean {
  return player.tutorialCompleted ||
         (player.linkedAccount?.level >= 5);
}
```

### Onboarding Rewards

| Milestone | Reward |
|-----------|--------|
| Tutorial Complete | 100 Gold + 10 Doubloons + Badge |
| First Bot Match Win | 50 Gold |
| First Real Match | 100 Gold |
| First Kill | 25 Gold |
| First Victory | 200 Gold + Exclusive Emote |

---

## Analytics Events

```typescript
// Tutorial tracking
analytics.track("tutorial_started", {
  playerId: string,
  isNewAccount: boolean
});

analytics.track("tutorial_stage_completed", {
  playerId: string,
  stageId: string,
  timeSpent: number,
  attemptsNeeded: number
});

analytics.track("tutorial_completed", {
  playerId: string,
  totalTime: number,
  skipped: boolean
});

analytics.track("tutorial_abandoned", {
  playerId: string,
  lastStage: string,
  timeSpent: number
});

// Training ground
analytics.track("training_ground_session", {
  playerId: string,
  duration: number,
  abilitiesUsed: string[],
  botFightsCompleted: number
});

// Bot match
analytics.track("bot_match_completed", {
  playerId: string,
  difficulty: string,
  placement: number,
  kills: number,
  duration: number
});
```

---

## UI Screens

### Mode Selection

```
┌─────────────────────────────────────────────────────────────────┐
│                        PLAY                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────┐  ┌─────────────────────┐              │
│  │                     │  │                     │              │
│  │     QUICK PLAY      │  │     PRACTICE        │              │
│  │                     │  │                     │              │
│  │   Solo / Duo        │  │  Training Ground    │              │
│  │   Real Players      │  │  Test Abilities     │              │
│  │                     │  │                     │              │
│  └─────────────────────┘  └─────────────────────┘              │
│                                                                  │
│  ┌─────────────────────┐  ┌─────────────────────┐              │
│  │                     │  │                     │              │
│  │     BOT MATCH       │  │     TUTORIAL        │              │
│  │                     │  │                     │              │
│  │   Play vs AI        │  │  Learn the Basics   │              │
│  │   Earn 50% Rewards  │  │  (Replay Anytime)   │              │
│  │                     │  │                     │              │
│  └─────────────────────┘  └─────────────────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Bot Match Lobby

```
┌─────────────────────────────────────────────────────────────────┐
│                     BOT MATCH SETUP                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Difficulty:    [Easy]  [Medium]  [Hard]                        │
│                            ▲                                     │
│                         Selected                                │
│                                                                  │
│  Bot Count:     30 players                                      │
│                                                                  │
│  Match Type:    [Normal]  [Quick]                               │
│                    ▲                                             │
│                                                                  │
│  Rewards:       50% of normal (Gold + XP)                       │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    [START MATCH]                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Testing Requirements

### Tutorial Tests

- [ ] All stages complete correctly
- [ ] Progress saves if player exits
- [ ] Skip function works after completion
- [ ] Rewards grant correctly
- [ ] Voice-over plays at appropriate times
- [ ] Highlights draw attention correctly

### Training Ground Tests

- [ ] All abilities spawn correctly
- [ ] Target dummies respond to damage
- [ ] Moving targets move as expected
- [ ] Bot sparring partner works
- [ ] Reset function works
- [ ] Options toggle correctly

### Bot Match Tests

- [ ] Bots spawn and behave appropriately
- [ ] Difficulty settings affect bot performance
- [ ] Bots loot, fight, and navigate correctly
- [ ] Match completes normally
- [ ] Rewards grant at reduced rate
- [ ] Bots don't exploit or break

## Success Metrics

| Metric | Target |
|--------|--------|
| Tutorial completion rate | > 80% |
| Time to complete tutorial | 3-5 minutes |
| Training ground usage | 20%+ of players use at least once |
| Bot match usage | 15%+ of casual matches |
| Tutorial abandonment | < 15% |

## Dependencies

- **GAME-016**: NPC AI Behavior System (bot AI foundation)
- **UX-002**: Camera Control System (tutorial controls)
- **ART-027**: Sound Effects (tutorial voice-over)
