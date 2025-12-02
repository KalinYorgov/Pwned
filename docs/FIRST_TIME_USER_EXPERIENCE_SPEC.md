# First-Time User Experience (FTUE) Specification

## Overview

This document specifies the First-Time User Experience (FTUE) for Plunderstorm Mobile, guiding new players from app launch through their first match while teaching core mechanics and establishing engagement hooks. The FTUE should be completable in under 10 minutes while creating a positive first impression.

## FTUE Flow

### Journey Overview

```
App Install
    │
    ▼
┌─────────────────────┐
│  1. Splash Screen   │ (5 seconds)
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  2. Age Gate        │ (if required)
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  3. Account Setup   │ (30 seconds - 2 minutes)
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  4. Character       │ (1 minute)
│     Creation        │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  5. Tutorial        │ (3-5 minutes)
│     Match           │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  6. Rewards &       │ (30 seconds)
│     Next Steps      │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  7. Main Menu       │
│     (First Visit)   │
└─────────────────────┘
```

### Time Targets

| Stage | Target Duration | Maximum |
|-------|-----------------|---------|
| Splash | 3-5s | 5s |
| Age Gate | 5s | 15s |
| Account | 30s-2min | 3min |
| Character | 30s-1min | 2min |
| Tutorial | 3-5min | 7min |
| Rewards | 20-30s | 1min |
| **Total** | **5-10min** | **15min** |

## Stage 1: Splash Screen

### Splash Design

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                                                                         │
│                                                                         │
│                    [Animated Game Logo]                                │
│                                                                         │
│                    PLUNDERSTORM                                         │
│                        MOBILE                                          │
│                                                                         │
│                    [Ship sailing animation]                            │
│                                                                         │
│                                                                         │
│                    Loading...                                          │
│                                                                         │
│                                                                         │
│                    © 2024 Plunderstorm Studios                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Splash Configuration

```typescript
interface SplashConfig {
  duration: {
    minimum: 3000;          // Always show at least 3s (branding)
    maximum: 5000;          // Never show more than 5s
    loadBased: true;        // Dismiss when assets ready
  };

  animation: {
    logoFadeIn: true;
    shipAnimation: true;
    waveEffect: true;
  };

  preload: {
    criticalAssets: string[];
    parallelAuth: true;     // Start auth check during splash
  };
}
```

## Stage 2: Age Gate

### Age Verification

```typescript
interface AgeGateConfig {
  enabled: boolean;
  requiredRegions: string[];    // Countries requiring age gate
  minimumAge: number;           // Default 13 (COPPA)
  method: 'date_picker' | 'year_dropdown' | 'confirmation';
  rememberChoice: boolean;
}

const AGE_GATE_CONFIG: AgeGateConfig = {
  enabled: true,
  requiredRegions: ['US', 'EU', 'UK', 'CA', 'AU'],
  minimumAge: 13,
  method: 'date_picker',
  rememberChoice: true
};
```

### Age Gate UI

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    WELCOME, PIRATE!                                    │
│                                                                         │
│                    Before we set sail, please                          │
│                    enter your date of birth:                           │
│                                                                         │
│                    ┌────────────────────────┐                          │
│                    │  [Month ▼] [Day ▼] [Year ▼]                       │
│                    └────────────────────────┘                          │
│                                                                         │
│                    Your privacy is important to us.                    │
│                    [Privacy Policy]                                    │
│                                                                         │
│                    [Continue]                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Under-Age Handling

```typescript
function handleUnderAge(age: number): void {
  if (age < AGE_GATE_CONFIG.minimumAge) {
    showUnderAgeScreen({
      message: "We're sorry, but you must be 13 or older to play Plunderstorm Mobile.",
      showParentInfo: true,
      allowRetry: false
    });
  }
}
```

## Stage 3: Account Setup

### Account Options

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    LET'S GET STARTED!                                  │
│                                                                         │
│    Choose how you want to play:                                        │
│                                                                         │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │                                                                 │ │
│    │  [Apple Logo]  Continue with Apple                             │ │
│    │                                                                 │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │                                                                 │ │
│    │  [Google Logo]  Continue with Google                           │ │
│    │                                                                 │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │                                                                 │ │
│    │  [Guest Icon]  Play as Guest                                   │ │
│    │  (You can link an account later)                               │ │
│    │                                                                 │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│                    [Terms of Service] | [Privacy Policy]               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Account Configuration

```typescript
interface AccountSetupConfig {
  authProviders: AuthProvider[];
  guestPlayEnabled: true;
  guestLinkReminder: {
    enabled: true;
    afterMatches: 3;
    afterDays: 1;
  };
  nameGeneration: {
    enabled: true;
    format: 'Adjective_Noun_Number';   // "Swift_Pirate_42"
    allowCustom: true;
    minLength: 3;
    maxLength: 16;
    profanityFilter: true;
  };
}

enum AuthProvider {
  APPLE = 'apple',
  GOOGLE = 'google',
  FACEBOOK = 'facebook',
  EMAIL = 'email'
}

async function setupAccount(method: AuthProvider | 'guest'): Promise<Account> {
  if (method === 'guest') {
    // Create guest account with generated name
    const guestName = generatePirateName();
    const account = await createGuestAccount(guestName);

    // Mark for link reminder
    setLinkReminder(ACCOUNT_CONFIG.guestLinkReminder);

    return account;
  } else {
    // OAuth flow
    const authResult = await performOAuth(method);
    const account = await createOrLinkAccount(authResult);

    return account;
  }
}
```

### Name Selection

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    CHOOSE YOUR PIRATE NAME                             │
│                                                                         │
│    Every great pirate needs a legendary name!                          │
│                                                                         │
│    Suggested names:                                                     │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │  🏴‍☠️ Swift_Corsair_42                                           │ │
│    │  🏴‍☠️ Daring_Buccaneer_17                                         │ │
│    │  🏴‍☠️ Salty_Mariner_88                                            │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│    Or enter your own:                                                  │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │  [                              ]                               │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│    3-16 characters, letters and numbers only                           │
│                                                                         │
│                    [Continue]                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Stage 4: Character Creation

### Character Selection

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    CREATE YOUR PIRATE                                  │
│                                                                         │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │                                                                 │ │
│    │                    [3D Character Preview]                       │ │
│    │                                                                 │ │
│    │                    ← Swipe to rotate →                          │ │
│    │                                                                 │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│    BASE STYLE:                                                          │
│    [  🧔 Male  ]  [  👩 Female  ]  [  🎭 Other  ]                       │
│                                                                         │
│    SKIN TONE:                                                           │
│    [ 🏻 ] [ 🏼 ] [ 🏽 ] [ 🏾 ] [ 🏿 ]                                    │
│                                                                         │
│    STARTER OUTFIT: (More unlockable!)                                  │
│    [ Red Bandana ]  [ Blue Coat ]  [ Classic ]                         │
│                                                                         │
│                    [Set Sail! →]                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Character Configuration

```typescript
interface CharacterCreationConfig {
  baseStyles: CharacterStyle[];
  skinTones: string[];
  starterOutfits: Outfit[];
  previewRotation: boolean;
  randomizeButton: boolean;
  skipOption: boolean;        // Use default character
}

interface CharacterStyle {
  id: string;
  name: string;
  bodyType: 'masculine' | 'feminine' | 'neutral';
  icon: string;
}

const CHARACTER_CONFIG: CharacterCreationConfig = {
  baseStyles: [
    { id: 'male', name: 'Male', bodyType: 'masculine', icon: '🧔' },
    { id: 'female', name: 'Female', bodyType: 'feminine', icon: '👩' },
    { id: 'other', name: 'Other', bodyType: 'neutral', icon: '🎭' }
  ],
  skinTones: ['#FFE0BD', '#E8C4A0', '#C68642', '#8D5524', '#4A2912'],
  starterOutfits: [
    { id: 'red_bandana', name: 'Red Bandana', preview: 'outfit_red.png' },
    { id: 'blue_coat', name: 'Blue Coat', preview: 'outfit_blue.png' },
    { id: 'classic', name: 'Classic Pirate', preview: 'outfit_classic.png' }
  ],
  previewRotation: true,
  randomizeButton: true,
  skipOption: true
};
```

## Stage 5: Tutorial Match

### Tutorial Structure

```typescript
interface TutorialConfig {
  type: 'guided_match';
  environment: 'tutorial_island';     // Smaller, controlled map
  opponents: 'bots';
  botDifficulty: 'very_easy';
  stages: TutorialStage[];
  skipOption: boolean;
  skipUnlocksAfter: 'first_completion';
}

interface TutorialStage {
  id: string;
  name: string;
  objective: string;
  instructions: Instruction[];
  completionCondition: CompletionCondition;
  highlights: HighlightConfig[];
  voiceover?: string;
  canSkip: boolean;
}

const TUTORIAL_STAGES: TutorialStage[] = [
  {
    id: 'movement',
    name: 'Movement',
    objective: 'Learn to move your pirate',
    instructions: [
      { text: 'Touch and drag on the LEFT side to move', highlight: 'left_screen' },
      { text: 'Move to the marked location', highlight: 'waypoint' }
    ],
    completionCondition: { type: 'reach_location', target: 'waypoint_1' },
    highlights: [{ element: 'joystick_area', type: 'pulse' }],
    voiceover: 'vo_tutorial_movement',
    canSkip: false
  },
  {
    id: 'camera',
    name: 'Camera Control',
    objective: 'Learn to look around',
    instructions: [
      { text: 'Touch and drag on the RIGHT side to look around', highlight: 'right_screen' },
      { text: 'Look at the treasure chest', highlight: 'chest' }
    ],
    completionCondition: { type: 'look_at', target: 'tutorial_chest' },
    highlights: [{ element: 'camera_area', type: 'pulse' }],
    voiceover: 'vo_tutorial_camera',
    canSkip: false
  },
  {
    id: 'chest',
    name: 'Opening Chests',
    objective: 'Get your first ability',
    instructions: [
      { text: 'Move to the chest', highlight: 'chest' },
      { text: 'Tap the INTERACT button to open it', highlight: 'interact_button' }
    ],
    completionCondition: { type: 'open_chest', target: 'tutorial_chest' },
    highlights: [{ element: 'interact_button', type: 'bounce' }],
    voiceover: 'vo_tutorial_chest',
    canSkip: false
  },
  {
    id: 'ability_pickup',
    name: 'Abilities',
    objective: 'Pick up an ability',
    instructions: [
      { text: 'Abilities give you special powers!', highlight: 'ability_drop' },
      { text: 'Walk over the ability to pick it up', highlight: 'ability_drop' }
    ],
    completionCondition: { type: 'pickup_ability' },
    highlights: [{ element: 'ability_slots', type: 'glow' }],
    voiceover: 'vo_tutorial_ability',
    canSkip: false
  },
  {
    id: 'basic_attack',
    name: 'Basic Attack',
    objective: 'Attack the training dummy',
    instructions: [
      { text: 'Tap the ATTACK button to swing your cutlass', highlight: 'attack_button' },
      { text: 'Hit the training dummy 3 times', highlight: 'dummy' }
    ],
    completionCondition: { type: 'deal_damage', target: 'training_dummy', amount: 3 },
    highlights: [{ element: 'attack_button', type: 'pulse' }],
    voiceover: 'vo_tutorial_attack',
    canSkip: false
  },
  {
    id: 'ability_use',
    name: 'Using Abilities',
    objective: 'Use your new ability',
    instructions: [
      { text: 'Press and hold the ability button', highlight: 'ability_1' },
      { text: 'Drag to aim, release to fire!', highlight: 'targeting' }
    ],
    completionCondition: { type: 'use_ability', target: 'ability_1' },
    highlights: [{ element: 'ability_1', type: 'bounce' }],
    voiceover: 'vo_tutorial_use_ability',
    canSkip: false
  },
  {
    id: 'storm',
    name: 'The Storm',
    objective: 'Learn about the storm',
    instructions: [
      { text: 'The storm is coming! Stay in the safe zone', highlight: 'minimap' },
      { text: 'Move inside the white circle', highlight: 'safe_zone' }
    ],
    completionCondition: { type: 'in_zone', duration: 3000 },
    highlights: [{ element: 'minimap', type: 'pulse' }],
    voiceover: 'vo_tutorial_storm',
    canSkip: true
  },
  {
    id: 'combat',
    name: 'Pirate Combat',
    objective: 'Defeat the enemy pirate',
    instructions: [
      { text: 'An enemy approaches! Use everything you learned', highlight: 'enemy' },
      { text: 'Defeat them to win!', highlight: 'enemy' }
    ],
    completionCondition: { type: 'eliminate', target: 'tutorial_bot' },
    highlights: [],
    voiceover: 'vo_tutorial_combat',
    canSkip: true
  }
];
```

### Tutorial UI Overlay

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  TUTORIAL: Movement (1/8)                                    [Skip ▷]  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │  Touch and drag on the LEFT side to move your pirate.         │   │
│  │                                                                 │   │
│  │                    [Hand Animation]                            │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│                    ┌─────────────┐                                     │
│                    │   Objective │                                     │
│                    │  Move to    │                                     │
│                    │  the marker │                                     │
│                    │     🎯      │                                     │
│                    └─────────────┘                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Tutorial Highlight System

```typescript
interface HighlightConfig {
  element: string;
  type: 'pulse' | 'bounce' | 'glow' | 'arrow' | 'circle';
  color?: string;
  duration?: number;
  repeat?: boolean;
}

function highlightElement(config: HighlightConfig): void {
  const element = findElement(config.element);

  switch (config.type) {
    case 'pulse':
      // Subtle pulsing glow
      element.classList.add('highlight-pulse');
      break;

    case 'bounce':
      // Bouncing animation
      element.classList.add('highlight-bounce');
      break;

    case 'glow':
      // Static glow outline
      element.classList.add('highlight-glow');
      break;

    case 'arrow':
      // Arrow pointing to element
      showArrowPointer(element);
      break;

    case 'circle':
      // Circle around element
      showCircleHighlight(element);
      break;
  }

  // Dim rest of screen
  showFocusMask(element);
}
```

## Stage 6: Rewards

### Tutorial Completion Rewards

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    🎉 TUTORIAL COMPLETE! 🎉                            │
│                                                                         │
│    You're ready to become a legendary pirate!                          │
│                                                                         │
│    YOUR REWARDS:                                                        │
│    ┌─────────────────────────────────────────────────────────────────┐ │
│    │                                                                 │ │
│    │    💰 500 Gold                                                 │ │
│    │                                                                 │ │
│    │    🎨 "Rookie Pirate" Title                                    │ │
│    │                                                                 │ │
│    │    👕 Starter Skin Chest (1x)                                  │ │
│    │        [Open Now]                                              │ │
│    │                                                                 │ │
│    └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│                    [Continue to Main Menu →]                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Reward Configuration

```typescript
interface TutorialRewards {
  gold: number;
  title: string;
  chests: ChestReward[];
  battlePassXP: number;
  unlocks: string[];
}

const TUTORIAL_REWARDS: TutorialRewards = {
  gold: 500,
  title: 'Rookie Pirate',
  chests: [
    { type: 'starter_skin', count: 1, guaranteed: ['common_skin_random'] }
  ],
  battlePassXP: 100,
  unlocks: ['practice_mode', 'daily_quests']
};

async function grantTutorialRewards(): Promise<void> {
  // Grant gold
  await addCurrency('gold', TUTORIAL_REWARDS.gold);

  // Grant title
  await unlockTitle(TUTORIAL_REWARDS.title);

  // Grant chest(s)
  for (const chest of TUTORIAL_REWARDS.chests) {
    await grantChest(chest);
  }

  // Grant XP
  await addBattlePassXP(TUTORIAL_REWARDS.battlePassXP);

  // Unlock features
  for (const unlock of TUTORIAL_REWARDS.unlocks) {
    await unlockFeature(unlock);
  }

  // Show rewards animation
  await showRewardsAnimation(TUTORIAL_REWARDS);
}
```

## Stage 7: First Main Menu Visit

### Main Menu Introduction

```typescript
interface FirstMenuConfig {
  highlightFeatures: string[];
  showTooltips: boolean;
  guidedTour: boolean;
  deferredPrompts: DeferredPrompt[];
}

interface DeferredPrompt {
  id: string;
  triggerAfter: 'matches' | 'time' | 'level';
  triggerValue: number;
  prompt: PromptConfig;
}

const FIRST_MENU_CONFIG: FirstMenuConfig = {
  highlightFeatures: ['play_button', 'quests', 'battle_pass'],
  showTooltips: true,
  guidedTour: false,    // Optional guided tour
  deferredPrompts: [
    {
      id: 'link_account',
      triggerAfter: 'matches',
      triggerValue: 3,
      prompt: {
        title: 'Link Your Account',
        message: 'Save your progress by linking an account!',
        action: 'open_account_link'
      }
    },
    {
      id: 'rate_app',
      triggerAfter: 'matches',
      triggerValue: 5,
      prompt: {
        title: 'Enjoying Plunderstorm?',
        message: 'Rate us on the App Store!',
        action: 'open_rating'
      }
    }
  ]
};
```

### Feature Tooltips

```
First time seeing Battle Pass:
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│    ┌─────────────────────────────────────────────────────┐             │
│    │  💡 NEW! BATTLE PASS                                │             │
│    │                                                     │             │
│    │  Earn exclusive rewards by                         │             │
│    │  playing matches and completing                    │             │
│    │  challenges!                                       │             │
│    │                                                     │             │
│    │  [Got it!]                                         │             │
│    └─────────────────────────────────────────────────────┘             │
│                              ▼                                         │
│                    [Battle Pass Tab]                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Progress Saving

### Save Points

```typescript
interface FTUEProgress {
  currentStage: FTUEStage;
  completedStages: FTUEStage[];
  characterData?: CharacterData;
  accountType: 'guest' | 'linked';
  tutorialProgress?: {
    currentStep: number;
    completedSteps: string[];
  };
  timestamp: Date;
}

enum FTUEStage {
  SPLASH = 'splash',
  AGE_GATE = 'age_gate',
  ACCOUNT = 'account',
  CHARACTER = 'character',
  TUTORIAL = 'tutorial',
  REWARDS = 'rewards',
  COMPLETE = 'complete'
}

async function saveFTUEProgress(progress: FTUEProgress): Promise<void> {
  // Save locally first (instant)
  await saveLocal('ftue_progress', progress);

  // Sync to cloud if account linked
  if (progress.accountType === 'linked') {
    await syncToCloud('ftue_progress', progress);
  }
}

async function resumeFTUE(): Promise<FTUEProgress | null> {
  const savedProgress = await loadLocal('ftue_progress');

  if (savedProgress && savedProgress.currentStage !== FTUEStage.COMPLETE) {
    // Resume from saved point
    return savedProgress;
  }

  return null;
}
```

### Crash Recovery

```typescript
async function handleFTUECrash(): Promise<void> {
  const progress = await resumeFTUE();

  if (progress) {
    // Show recovery message
    showToast('Continuing where you left off...');

    // Resume from last completed stage
    navigateToFTUEStage(progress.currentStage);
  } else {
    // Start fresh
    startFTUE();
  }
}
```

## Skip Options

### Tutorial Skip

```typescript
interface SkipConfig {
  tutorialSkipEnabled: boolean;
  skipUnlocksAfter: 'never' | 'first_completion' | 'always';
  skipConfirmation: boolean;
  skipGivesRewards: boolean;    // Partial rewards for skipping
}

const SKIP_CONFIG: SkipConfig = {
  tutorialSkipEnabled: true,
  skipUnlocksAfter: 'first_completion',
  skipConfirmation: true,
  skipGivesRewards: true
};

async function handleTutorialSkip(): Promise<void> {
  if (SKIP_CONFIG.skipConfirmation) {
    const confirmed = await showConfirmation({
      title: 'Skip Tutorial?',
      message: 'Are you sure? The tutorial teaches important controls.',
      confirmText: 'Skip',
      cancelText: 'Continue Tutorial'
    });

    if (!confirmed) return;
  }

  if (SKIP_CONFIG.skipGivesRewards) {
    // Grant partial rewards
    await grantTutorialRewards({ partial: true, amount: 0.5 });
  }

  // Mark tutorial complete
  await completeTutorial({ skipped: true });

  // Continue to rewards/menu
  navigateToPostTutorial();
}
```

## Analytics Events

```typescript
// FTUE tracking
analytics.track('ftue_started', {
  deviceInfo: DeviceInfo,
  timestamp: Date
});

analytics.track('ftue_stage_completed', {
  stage: FTUEStage,
  duration: number,
  skipped: boolean
});

analytics.track('ftue_account_created', {
  method: 'guest' | AuthProvider,
  nameGenerated: boolean
});

analytics.track('ftue_character_created', {
  style: string,
  skinTone: string,
  outfit: string,
  customized: boolean
});

analytics.track('ftue_tutorial_step', {
  stepId: string,
  stepNumber: number,
  duration: number,
  attempts: number,
  skipped: boolean
});

analytics.track('ftue_completed', {
  totalDuration: number,
  stagesSkipped: string[],
  accountType: string
});

analytics.track('ftue_abandoned', {
  lastStage: FTUEStage,
  duration: number,
  reason?: string
});
```

## Testing Requirements

### Functional Tests

- [ ] Full FTUE completes successfully
- [ ] Age gate blocks underage users
- [ ] Guest account creates correctly
- [ ] OAuth flows work correctly
- [ ] Character creation saves correctly
- [ ] Tutorial stages complete in order
- [ ] Tutorial can be skipped
- [ ] Rewards granted correctly
- [ ] Progress saves and resumes

### Edge Cases

- [ ] App crash during FTUE
- [ ] Network disconnect during FTUE
- [ ] Very slow network
- [ ] Account already exists
- [ ] Tutorial softlock recovery

### Metrics Tests

- [ ] FTUE under 10 minutes
- [ ] Each stage under time limits
- [ ] 90%+ completion rate target

## Success Metrics

| Metric | Target |
|--------|--------|
| FTUE completion rate | > 80% |
| FTUE time | < 10 minutes |
| Tutorial skip rate | < 30% |
| Day 1 retention | > 40% |
| Account link (Day 7) | > 50% |

## Dependencies

- **BACK-005**: Account System
- **UX-027**: Interactive Tutorial
- **GAME-024**: Practice Mode
- **MON-001**: Currency System
