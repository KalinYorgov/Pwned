# Interactive Tutorial Specification

## Overview

This document specifies the Interactive Tutorial system for Plunderstorm Mobile, providing step-by-step guidance for new players to learn core game mechanics through hands-on practice in a controlled environment.

## Tutorial Structure

### Tutorial Overview

```
INTERACTIVE TUTORIAL
├── Stage 1: Movement Basics
│   ├── Step 1.1: Virtual joystick introduction
│   ├── Step 1.2: Move to waypoint
│   └── Step 1.3: Sprint mechanic
├── Stage 2: Camera Controls
│   ├── Step 2.1: Look around
│   ├── Step 2.2: Find the treasure
│   └── Step 2.3: Camera settings tip
├── Stage 3: Interaction
│   ├── Step 3.1: Approach chest
│   └── Step 3.2: Open chest
├── Stage 4: Abilities
│   ├── Step 4.1: Pick up ability
│   ├── Step 4.2: Ability slots explained
│   └── Step 4.3: Multiple abilities
├── Stage 5: Combat Basics
│   ├── Step 5.1: Basic attack
│   ├── Step 5.2: Damage numbers
│   └── Step 5.3: Health and damage
├── Stage 6: Using Abilities
│   ├── Step 6.1: Instant ability
│   ├── Step 6.2: Skillshot ability
│   └── Step 6.3: Cooldowns
├── Stage 7: Storm Survival
│   ├── Step 7.1: Storm warning
│   ├── Step 7.2: Reading the minimap
│   └── Step 7.3: Safe zone movement
├── Stage 8: Victory
│   ├── Step 8.1: Final combat
│   ├── Step 8.2: Elimination
│   └── Step 8.3: Victory celebration
└── Complete: Rewards & Summary
```

### Stage Configuration

```typescript
interface TutorialStage {
  id: string;
  name: string;
  description: string;
  steps: TutorialStep[];
  environment: EnvironmentConfig;
  onEnter?: () => void;
  onComplete?: () => void;
  canSkip: boolean;
  estimatedTime: number;        // seconds
}

interface TutorialStep {
  id: string;
  instruction: Instruction;
  objective: Objective;
  hints: Hint[];
  highlights: Highlight[];
  blockers?: Blocker[];         // What's disabled during this step
  validation: ValidationRule;
  onComplete?: () => void;
  timeout?: number;             // Auto-advance after timeout
}

interface Instruction {
  text: string;
  voiceover?: string;
  animation?: AnimationConfig;
  position: 'top' | 'center' | 'bottom';
}

interface Objective {
  text: string;
  icon: string;
  progress?: ProgressConfig;
}

interface Hint {
  text: string;
  delay: number;                // Show after X seconds
  icon?: string;
}
```

## Tutorial Environment

### Tutorial Island Map

```typescript
interface TutorialEnvironment {
  mapId: 'tutorial_island';
  size: { width: 200, height: 200 };   // meters
  zones: TutorialZone[];
  props: TutorialProp[];
  npcs: TutorialNPC[];
  weather: 'clear';
  lighting: 'daytime';
}

interface TutorialZone {
  id: string;
  name: string;
  bounds: Bounds;
  purpose: string;
  activeInStages: string[];
}

const TUTORIAL_ZONES: TutorialZone[] = [
  {
    id: 'spawn_area',
    name: 'Starting Beach',
    bounds: { x: 0, y: 0, width: 50, height: 50 },
    purpose: 'Movement and camera learning',
    activeInStages: ['movement', 'camera']
  },
  {
    id: 'chest_area',
    name: 'Treasure Cove',
    bounds: { x: 50, y: 0, width: 40, height: 40 },
    purpose: 'Chest and ability learning',
    activeInStages: ['interaction', 'abilities']
  },
  {
    id: 'combat_area',
    name: 'Training Grounds',
    bounds: { x: 100, y: 0, width: 50, height: 50 },
    purpose: 'Combat and ability use',
    activeInStages: ['combat', 'ability_use']
  },
  {
    id: 'storm_area',
    name: 'Storm Zone',
    bounds: { x: 0, y: 50, width: 100, height: 50 },
    purpose: 'Storm mechanics',
    activeInStages: ['storm']
  },
  {
    id: 'final_area',
    name: 'Final Arena',
    bounds: { x: 100, y: 50, width: 50, height: 50 },
    purpose: 'Final tutorial combat',
    activeInStages: ['victory']
  }
];
```

### Tutorial Props

```typescript
interface TutorialProp {
  id: string;
  type: PropType;
  position: Vector3;
  activeInStages: string[];
  highlight: boolean;
  interactable: boolean;
}

enum PropType {
  WAYPOINT_MARKER = 'waypoint_marker',
  TREASURE_CHEST = 'treasure_chest',
  TRAINING_DUMMY = 'training_dummy',
  ABILITY_DROP = 'ability_drop',
  HINT_SIGN = 'hint_sign',
  BOUNDARY_BARRIER = 'boundary_barrier'
}

const TUTORIAL_PROPS: TutorialProp[] = [
  // Movement waypoints
  {
    id: 'waypoint_1',
    type: PropType.WAYPOINT_MARKER,
    position: { x: 20, y: 0, z: 20 },
    activeInStages: ['movement'],
    highlight: true,
    interactable: false
  },
  // Chests
  {
    id: 'tutorial_chest_1',
    type: PropType.TREASURE_CHEST,
    position: { x: 60, y: 0, z: 15 },
    activeInStages: ['interaction', 'abilities'],
    highlight: true,
    interactable: true
  },
  // Training dummies
  {
    id: 'dummy_1',
    type: PropType.TRAINING_DUMMY,
    position: { x: 110, y: 0, z: 20 },
    activeInStages: ['combat', 'ability_use'],
    highlight: true,
    interactable: true
  },
  // Guaranteed ability drops
  {
    id: 'ability_fire_bomb',
    type: PropType.ABILITY_DROP,
    position: { x: 62, y: 0, z: 20 },
    activeInStages: ['abilities'],
    highlight: true,
    interactable: true
  }
];
```

## Step-by-Step Tutorial Content

### Stage 1: Movement Basics

```typescript
const MOVEMENT_STAGE: TutorialStage = {
  id: 'movement',
  name: 'Movement Basics',
  description: 'Learn how to move your pirate',
  estimatedTime: 60,
  canSkip: false,
  steps: [
    {
      id: 'movement_intro',
      instruction: {
        text: 'Welcome, pirate! Let\'s learn how to move.',
        voiceover: 'vo_movement_intro',
        position: 'center'
      },
      objective: {
        text: 'Tap anywhere to continue',
        icon: 'icon_tap'
      },
      hints: [],
      highlights: [],
      validation: { type: 'tap_anywhere' }
    },
    {
      id: 'movement_joystick',
      instruction: {
        text: 'Touch and drag on the LEFT side of the screen to move',
        voiceover: 'vo_movement_joystick',
        animation: { type: 'hand_drag', target: 'left_screen' },
        position: 'top'
      },
      objective: {
        text: 'Move to the glowing marker',
        icon: 'icon_waypoint',
        progress: { current: 0, target: 1 }
      },
      hints: [
        { text: 'Hold your thumb down and drag to move', delay: 5000 },
        { text: 'The joystick appears where you touch', delay: 10000 }
      ],
      highlights: [
        { element: 'left_screen_zone', type: 'area_highlight' },
        { element: 'waypoint_1', type: 'world_marker' }
      ],
      blockers: [
        { type: 'disable_zone', zone: 'right_screen' }
      ],
      validation: {
        type: 'reach_position',
        target: 'waypoint_1',
        radius: 3
      }
    },
    {
      id: 'movement_sprint',
      instruction: {
        text: 'Double-tap and hold to sprint!',
        voiceover: 'vo_movement_sprint',
        animation: { type: 'double_tap', target: 'left_screen' },
        position: 'top'
      },
      objective: {
        text: 'Sprint to the next marker',
        icon: 'icon_sprint'
      },
      hints: [
        { text: 'Sprinting uses stamina - watch your energy!', delay: 3000 }
      ],
      highlights: [
        { element: 'waypoint_2', type: 'world_marker' }
      ],
      validation: {
        type: 'reach_position_sprinting',
        target: 'waypoint_2',
        radius: 3
      }
    }
  ]
};
```

### Stage 2: Camera Controls

```typescript
const CAMERA_STAGE: TutorialStage = {
  id: 'camera',
  name: 'Camera Controls',
  description: 'Learn how to look around',
  estimatedTime: 45,
  canSkip: false,
  steps: [
    {
      id: 'camera_intro',
      instruction: {
        text: 'Now let\'s learn to look around!',
        voiceover: 'vo_camera_intro',
        position: 'center'
      },
      objective: {
        text: 'Touch and drag on the RIGHT side to look around',
        icon: 'icon_camera'
      },
      hints: [],
      highlights: [
        { element: 'right_screen_zone', type: 'area_highlight' }
      ],
      validation: { type: 'camera_moved', threshold: 90 }  // degrees
    },
    {
      id: 'camera_find',
      instruction: {
        text: 'Look for the treasure chest and focus on it',
        voiceover: 'vo_camera_find',
        position: 'top'
      },
      objective: {
        text: 'Find the treasure chest',
        icon: 'icon_chest'
      },
      hints: [
        { text: 'Spin around to find the glowing chest!', delay: 5000 },
        { text: 'The chest is behind you', delay: 10000 }
      ],
      highlights: [
        { element: 'tutorial_chest_1', type: 'world_beacon' }
      ],
      validation: {
        type: 'look_at_object',
        target: 'tutorial_chest_1',
        fovAngle: 30,
        duration: 1000
      }
    },
    {
      id: 'camera_tip',
      instruction: {
        text: 'Tip: You can adjust camera sensitivity in Settings!',
        voiceover: 'vo_camera_tip',
        position: 'center'
      },
      objective: {
        text: 'Tap to continue',
        icon: 'icon_tap'
      },
      hints: [],
      highlights: [],
      validation: { type: 'tap_anywhere' },
      timeout: 5000
    }
  ]
};
```

### Stage 5: Combat Basics

```typescript
const COMBAT_STAGE: TutorialStage = {
  id: 'combat',
  name: 'Combat Basics',
  description: 'Learn how to fight',
  estimatedTime: 90,
  canSkip: false,
  steps: [
    {
      id: 'combat_intro',
      instruction: {
        text: 'Time to learn combat! Every pirate needs to fight.',
        voiceover: 'vo_combat_intro',
        position: 'center'
      },
      objective: {
        text: 'Approach the training dummy',
        icon: 'icon_dummy'
      },
      hints: [],
      highlights: [
        { element: 'dummy_1', type: 'world_marker' }
      ],
      validation: {
        type: 'reach_position',
        target: 'dummy_1',
        radius: 5
      }
    },
    {
      id: 'combat_attack',
      instruction: {
        text: 'Tap the ATTACK button to swing your cutlass!',
        voiceover: 'vo_combat_attack',
        animation: { type: 'button_tap', target: 'attack_button' },
        position: 'top'
      },
      objective: {
        text: 'Hit the dummy 3 times',
        icon: 'icon_attack',
        progress: { current: 0, target: 3 }
      },
      hints: [
        { text: 'Tap the crossed swords button on the right', delay: 3000 },
        { text: 'Keep attacking!', delay: 8000 }
      ],
      highlights: [
        { element: 'attack_button', type: 'ui_pulse' },
        { element: 'dummy_1', type: 'world_target' }
      ],
      validation: {
        type: 'damage_dealt',
        target: 'dummy_1',
        hits: 3
      }
    },
    {
      id: 'combat_damage',
      instruction: {
        text: 'See the damage numbers? That shows how much damage you deal!',
        voiceover: 'vo_combat_damage',
        position: 'center'
      },
      objective: {
        text: 'Observe the damage numbers',
        icon: 'icon_damage'
      },
      hints: [],
      highlights: [
        { element: 'damage_number_example', type: 'ui_highlight' }
      ],
      validation: { type: 'tap_anywhere' },
      timeout: 4000
    },
    {
      id: 'combat_health',
      instruction: {
        text: 'Your health is shown at the bottom. Don\'t let it reach zero!',
        voiceover: 'vo_combat_health',
        position: 'top'
      },
      objective: {
        text: 'Note your health bar',
        icon: 'icon_health'
      },
      hints: [],
      highlights: [
        { element: 'health_bar', type: 'ui_pulse' }
      ],
      validation: { type: 'tap_anywhere' },
      timeout: 4000
    }
  ]
};
```

### Stage 6: Using Abilities

```typescript
const ABILITY_USE_STAGE: TutorialStage = {
  id: 'ability_use',
  name: 'Using Abilities',
  description: 'Learn to use your special abilities',
  estimatedTime: 120,
  canSkip: false,
  steps: [
    {
      id: 'ability_instant',
      instruction: {
        text: 'Let\'s use an ability! Some abilities activate instantly.',
        voiceover: 'vo_ability_instant',
        position: 'top'
      },
      objective: {
        text: 'Tap an ability button to use it',
        icon: 'icon_ability'
      },
      hints: [
        { text: 'Look for the colored ability buttons on the right', delay: 3000 }
      ],
      highlights: [
        { element: 'ability_slot_1', type: 'ui_bounce' }
      ],
      validation: { type: 'ability_used' }
    },
    {
      id: 'ability_skillshot_intro',
      instruction: {
        text: 'Some abilities need to be aimed! Press and HOLD to aim.',
        voiceover: 'vo_ability_skillshot_intro',
        position: 'center'
      },
      objective: {
        text: 'Learn about skillshots',
        icon: 'icon_aim'
      },
      hints: [],
      highlights: [],
      validation: { type: 'tap_anywhere' }
    },
    {
      id: 'ability_skillshot',
      instruction: {
        text: 'Press, hold, and DRAG to aim. Release to fire!',
        voiceover: 'vo_ability_skillshot',
        animation: { type: 'drag_aim', target: 'ability_slot_2' },
        position: 'top'
      },
      objective: {
        text: 'Hit the dummy with a skillshot ability',
        icon: 'icon_skillshot'
      },
      hints: [
        { text: 'Press and hold the Fire Bomb ability', delay: 3000 },
        { text: 'Drag toward the dummy and release', delay: 6000 },
        { text: 'The targeting line shows where it will go', delay: 9000 }
      ],
      highlights: [
        { element: 'ability_slot_2', type: 'ui_pulse' },
        { element: 'dummy_1', type: 'world_target' }
      ],
      validation: {
        type: 'skillshot_hit',
        target: 'dummy_1'
      }
    },
    {
      id: 'ability_cooldown',
      instruction: {
        text: 'Abilities have cooldowns. Wait for them to recharge!',
        voiceover: 'vo_ability_cooldown',
        position: 'center'
      },
      objective: {
        text: 'Watch the cooldown timer',
        icon: 'icon_cooldown'
      },
      hints: [],
      highlights: [
        { element: 'ability_slot_2_cooldown', type: 'ui_highlight' }
      ],
      validation: { type: 'ability_off_cooldown', target: 'ability_slot_2' }
    }
  ]
};
```

## Highlight System

### Highlight Types

```typescript
enum HighlightType {
  // UI Highlights
  UI_PULSE = 'ui_pulse',           // Pulsing glow on UI element
  UI_BOUNCE = 'ui_bounce',         // Bouncing animation
  UI_HIGHLIGHT = 'ui_highlight',   // Static highlight
  UI_ARROW = 'ui_arrow',           // Arrow pointing to element

  // Screen Area Highlights
  AREA_HIGHLIGHT = 'area_highlight',  // Highlight screen region
  FOCUS_MASK = 'focus_mask',          // Dim everything except target

  // World Highlights
  WORLD_MARKER = 'world_marker',      // 3D marker in world
  WORLD_BEACON = 'world_beacon',      // Vertical beam of light
  WORLD_TARGET = 'world_target',      // Target reticle on object
  WORLD_PATH = 'world_path'           // Path line to destination
}

interface Highlight {
  element: string;
  type: HighlightType;
  color?: string;
  intensity?: number;
  animation?: HighlightAnimation;
}

interface HighlightAnimation {
  type: 'pulse' | 'bounce' | 'rotate' | 'fade';
  speed: number;
  loop: boolean;
}
```

### Highlight Implementation

```typescript
class TutorialHighlightManager {
  private activeHighlights: Map<string, HighlightInstance> = new Map();

  showHighlight(highlight: Highlight): void {
    const instance = this.createHighlight(highlight);
    this.activeHighlights.set(highlight.element, instance);
    instance.show();
  }

  hideHighlight(element: string): void {
    const instance = this.activeHighlights.get(element);
    if (instance) {
      instance.hide();
      this.activeHighlights.delete(element);
    }
  }

  hideAllHighlights(): void {
    for (const instance of this.activeHighlights.values()) {
      instance.hide();
    }
    this.activeHighlights.clear();
  }

  private createHighlight(config: Highlight): HighlightInstance {
    switch (config.type) {
      case HighlightType.UI_PULSE:
        return new UIPulseHighlight(config);
      case HighlightType.UI_BOUNCE:
        return new UIBounceHighlight(config);
      case HighlightType.WORLD_MARKER:
        return new WorldMarkerHighlight(config);
      case HighlightType.WORLD_BEACON:
        return new WorldBeaconHighlight(config);
      case HighlightType.FOCUS_MASK:
        return new FocusMaskHighlight(config);
      // ... other types
    }
  }
}

class WorldBeaconHighlight implements HighlightInstance {
  private beam: ParticleSystem;
  private groundDecal: Decal;

  constructor(config: Highlight) {
    const position = getWorldPosition(config.element);

    this.beam = createVerticalBeam({
      position,
      color: config.color || '#FFD700',
      height: 20,
      width: 1
    });

    this.groundDecal = createGroundDecal({
      position,
      radius: 2,
      color: config.color || '#FFD700'
    });
  }

  show(): void {
    this.beam.play();
    this.groundDecal.visible = true;
  }

  hide(): void {
    this.beam.stop();
    this.groundDecal.visible = false;
  }
}
```

## Input Blocking

### Blocker System

```typescript
interface Blocker {
  type: BlockerType;
  target?: string;
  zone?: string;
  message?: string;
}

enum BlockerType {
  DISABLE_ZONE = 'disable_zone',       // Disable touch zone
  DISABLE_BUTTON = 'disable_button',   // Disable specific button
  DISABLE_MOVEMENT = 'disable_movement',
  DISABLE_CAMERA = 'disable_camera',
  DISABLE_ALL_ABILITIES = 'disable_all_abilities',
  DISABLE_ABILITY = 'disable_ability', // Specific ability
  RESTRICT_AREA = 'restrict_area'      // Invisible wall
}

class TutorialBlockerManager {
  private activeBlockers: Blocker[] = [];

  applyBlockers(blockers: Blocker[]): void {
    for (const blocker of blockers) {
      this.applyBlocker(blocker);
    }
    this.activeBlockers = blockers;
  }

  clearBlockers(): void {
    for (const blocker of this.activeBlockers) {
      this.removeBlocker(blocker);
    }
    this.activeBlockers = [];
  }

  private applyBlocker(blocker: Blocker): void {
    switch (blocker.type) {
      case BlockerType.DISABLE_ZONE:
        disableTouchZone(blocker.zone);
        break;
      case BlockerType.DISABLE_BUTTON:
        disableButton(blocker.target);
        break;
      case BlockerType.RESTRICT_AREA:
        createInvisibleWall(blocker.zone);
        break;
      // ... other types
    }
  }
}
```

## Validation System

### Validation Rules

```typescript
interface ValidationRule {
  type: ValidationType;
  target?: string;
  value?: any;
  duration?: number;
  radius?: number;
}

enum ValidationType {
  TAP_ANYWHERE = 'tap_anywhere',
  TAP_ELEMENT = 'tap_element',
  REACH_POSITION = 'reach_position',
  REACH_POSITION_SPRINTING = 'reach_position_sprinting',
  CAMERA_MOVED = 'camera_moved',
  LOOK_AT_OBJECT = 'look_at_object',
  OPEN_CHEST = 'open_chest',
  PICKUP_ITEM = 'pickup_item',
  ABILITY_USED = 'ability_used',
  DAMAGE_DEALT = 'damage_dealt',
  SKILLSHOT_HIT = 'skillshot_hit',
  ABILITY_OFF_COOLDOWN = 'ability_off_cooldown',
  IN_ZONE = 'in_zone',
  ELIMINATE_ENEMY = 'eliminate_enemy'
}

class ValidationManager {
  async validateStep(step: TutorialStep): Promise<boolean> {
    const rule = step.validation;

    switch (rule.type) {
      case ValidationType.TAP_ANYWHERE:
        return await waitForTap();

      case ValidationType.REACH_POSITION:
        return await waitForPosition(rule.target, rule.radius);

      case ValidationType.LOOK_AT_OBJECT:
        return await waitForLookAt(rule.target, rule.fovAngle, rule.duration);

      case ValidationType.DAMAGE_DEALT:
        return await waitForDamage(rule.target, rule.hits);

      case ValidationType.SKILLSHOT_HIT:
        return await waitForSkillshotHit(rule.target);

      // ... other validations
    }
  }
}

async function waitForPosition(
  targetId: string,
  radius: number
): Promise<boolean> {
  const targetPos = getObjectPosition(targetId);

  return new Promise(resolve => {
    const checkInterval = setInterval(() => {
      const playerPos = getPlayerPosition();
      const distance = Vector3.distance(playerPos, targetPos);

      if (distance <= radius) {
        clearInterval(checkInterval);
        resolve(true);
      }
    }, 100);
  });
}
```

## Voice-Over System

### Voice-Over Configuration

```typescript
interface VoiceOverConfig {
  enabled: boolean;
  volume: number;
  language: string;
  narrator: 'male' | 'female' | 'neutral';
  subtitles: boolean;
  subtitleStyle: SubtitleStyle;
}

interface VoiceOverLine {
  id: string;
  text: string;
  audioFile: string;
  duration: number;
  priority: number;
}

const TUTORIAL_VOICE_LINES: VoiceOverLine[] = [
  {
    id: 'vo_movement_intro',
    text: "Welcome aboard, pirate! Before we set sail, let's learn the basics.",
    audioFile: 'vo/tutorial/movement_intro.mp3',
    duration: 4500,
    priority: 1
  },
  {
    id: 'vo_movement_joystick',
    text: "Touch the left side of the screen and drag to move your pirate.",
    audioFile: 'vo/tutorial/movement_joystick.mp3',
    duration: 4000,
    priority: 1
  },
  // ... more lines
];

class VoiceOverManager {
  private config: VoiceOverConfig;
  private currentLine?: VoiceOverLine;
  private audioSource: AudioSource;

  async playLine(lineId: string): Promise<void> {
    const line = TUTORIAL_VOICE_LINES.find(l => l.id === lineId);
    if (!line) return;

    // Stop current line if playing
    if (this.currentLine) {
      this.audioSource.stop();
    }

    this.currentLine = line;

    // Show subtitle
    if (this.config.subtitles) {
      showSubtitle(line.text, line.duration);
    }

    // Play audio
    if (this.config.enabled) {
      await this.audioSource.play(line.audioFile);
    }

    this.currentLine = undefined;
  }
}
```

## Progress Tracking

### Tutorial Progress

```typescript
interface TutorialProgress {
  tutorialId: string;
  currentStage: string;
  currentStep: string;
  completedStages: string[];
  completedSteps: string[];
  startTime: Date;
  totalTime: number;           // seconds spent
  deaths: number;
  hintsShown: number;
  skippedSteps: string[];
}

class TutorialProgressManager {
  private progress: TutorialProgress;

  saveProgress(): void {
    // Save locally
    saveLocal('tutorial_progress', this.progress);

    // Sync to server if online
    if (isOnline()) {
      syncTutorialProgress(this.progress);
    }
  }

  loadProgress(): TutorialProgress | null {
    return loadLocal('tutorial_progress');
  }

  completeStep(stepId: string): void {
    this.progress.completedSteps.push(stepId);
    this.saveProgress();

    // Analytics
    trackTutorialStep(stepId, this.progress);
  }

  completeStage(stageId: string): void {
    this.progress.completedStages.push(stageId);
    this.saveProgress();

    // Analytics
    trackTutorialStage(stageId, this.progress);
  }

  skipStep(stepId: string): void {
    this.progress.skippedSteps.push(stepId);
    this.saveProgress();
  }
}
```

## Skip System

### Skip Options

```typescript
interface SkipConfig {
  stageSkipEnabled: boolean;
  fullSkipEnabled: boolean;
  skipConfirmation: boolean;
  skipAfterAttempts: number;     // Offer skip after N failed attempts
  skipConditions: SkipCondition[];
}

interface SkipCondition {
  type: 'returning_player' | 'linked_account' | 'high_level';
  enabled: boolean;
}

const SKIP_CONFIG: SkipConfig = {
  stageSkipEnabled: true,
  fullSkipEnabled: true,
  skipConfirmation: true,
  skipAfterAttempts: 3,
  skipConditions: [
    { type: 'returning_player', enabled: true },
    { type: 'linked_account', enabled: false }
  ]
};

async function handleSkipRequest(type: 'step' | 'stage' | 'full'): Promise<boolean> {
  if (SKIP_CONFIG.skipConfirmation) {
    const confirmed = await showSkipConfirmation(type);
    if (!confirmed) return false;
  }

  switch (type) {
    case 'step':
      await skipCurrentStep();
      break;
    case 'stage':
      await skipCurrentStage();
      break;
    case 'full':
      await skipEntireTutorial();
      break;
  }

  return true;
}
```

## Rewards

### Completion Rewards

```typescript
interface TutorialReward {
  type: 'gold' | 'xp' | 'item' | 'title' | 'unlock';
  value: number | string;
  displayName: string;
  icon: string;
}

const TUTORIAL_COMPLETION_REWARDS: TutorialReward[] = [
  {
    type: 'gold',
    value: 500,
    displayName: '500 Gold',
    icon: 'icon_gold'
  },
  {
    type: 'title',
    value: 'Rookie Pirate',
    displayName: '"Rookie Pirate" Title',
    icon: 'icon_title'
  },
  {
    type: 'item',
    value: 'starter_chest',
    displayName: 'Starter Skin Chest',
    icon: 'icon_chest'
  },
  {
    type: 'xp',
    value: 100,
    displayName: '100 Battle Pass XP',
    icon: 'icon_xp'
  },
  {
    type: 'unlock',
    value: 'practice_mode',
    displayName: 'Practice Mode Unlocked',
    icon: 'icon_practice'
  }
];

async function grantTutorialRewards(completed: boolean, skipped: boolean): Promise<void> {
  const rewards = completed ? TUTORIAL_COMPLETION_REWARDS :
                  skipped ? getPartialRewards() : [];

  for (const reward of rewards) {
    await grantReward(reward);
  }

  // Show rewards UI
  await showRewardsScreen(rewards);
}
```

## Analytics Events

```typescript
// Tutorial tracking
analytics.track('tutorial_started', {
  isReturningPlayer: boolean,
  accountType: string
});

analytics.track('tutorial_step_completed', {
  stageId: string,
  stepId: string,
  duration: number,
  hintsShown: number,
  attempts: number
});

analytics.track('tutorial_step_skipped', {
  stageId: string,
  stepId: string,
  reason: string
});

analytics.track('tutorial_stage_completed', {
  stageId: string,
  duration: number,
  stepsSkipped: number
});

analytics.track('tutorial_completed', {
  totalDuration: number,
  stagesCompleted: number,
  stepsSkipped: number,
  deaths: number,
  fullCompletion: boolean
});

analytics.track('tutorial_abandoned', {
  lastStage: string,
  lastStep: string,
  duration: number,
  reason?: string
});
```

## Testing Requirements

### Functional Tests

- [ ] All stages complete in sequence
- [ ] All validation rules trigger correctly
- [ ] Highlights appear and disappear correctly
- [ ] Voice-overs play and sync with text
- [ ] Progress saves on crash/exit
- [ ] Skip options work correctly
- [ ] Rewards granted on completion

### Edge Cases

- [ ] Tutorial bot doesn't softlock
- [ ] Player can't escape tutorial area
- [ ] All abilities work as expected
- [ ] Storm doesn't kill player unfairly
- [ ] Network disconnect handled

## Success Metrics

| Metric | Target |
|--------|--------|
| Tutorial completion rate | > 85% |
| Average completion time | 4-6 minutes |
| Skip rate | < 20% |
| Post-tutorial match play | > 90% |

## Dependencies

- **UX-026**: First-Time User Experience
- **GAME-024**: Practice Mode
- **UX-003**: Ability Button Layout
- **UX-004**: Skillshot Targeting System
