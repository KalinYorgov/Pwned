# Contextual Hints System Specification

## Document Information
- **Task ID:** UX-028
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies the contextual hints system for Plunderstorm Mobile, providing helpful guidance to new players during gameplay without being intrusive to experienced players. The system intelligently triggers tips based on player actions and game state.

---

## System Architecture

### Hint Manager

```typescript
const HINT_SYSTEM_ARCHITECTURE = {
  components: {
    hintManager: {
      description: 'Central controller for hint display logic',
      responsibilities: [
        'Track hint triggers',
        'Manage hint queue',
        'Enforce cooldowns',
        'Persist shown hints'
      ]
    },

    hintTriggerSystem: {
      description: 'Monitors game state for hint conditions',
      responsibilities: [
        'Subscribe to game events',
        'Evaluate trigger conditions',
        'Request hint display'
      ]
    },

    hintDisplayController: {
      description: 'Handles UI rendering of hints',
      responsibilities: [
        'Position hints on screen',
        'Animate hint appearance/dismissal',
        'Handle user interactions'
      ]
    },

    hintPersistence: {
      description: 'Tracks which hints have been shown',
      storage: 'Local device storage',
      data: {
        hintsShown: 'Map<hintId, { count, lastShown, dismissed }>',
        globalSettings: '{ enabled, showCount }'
      }
    }
  }
};
```

### Hint Priority System

```typescript
const HINT_PRIORITY = {
  levels: {
    critical: {
      priority: 1,
      description: 'Survival-critical information',
      examples: ['Storm warning', 'Very low health'],
      canInterrupt: true,
      maxQueue: 1
    },
    important: {
      priority: 2,
      description: 'Key gameplay mechanics',
      examples: ['First chest', 'First ability', 'First enemy'],
      canInterrupt: false,
      maxQueue: 2
    },
    helpful: {
      priority: 3,
      description: 'Nice-to-know tips',
      examples: ['Ping system', 'Emote usage', 'Map features'],
      canInterrupt: false,
      maxQueue: 3
    },
    informational: {
      priority: 4,
      description: 'General game tips',
      examples: ['Control reminders', 'Strategy tips'],
      canInterrupt: false,
      maxQueue: 1
    }
  },

  queueBehavior: {
    maxTotalQueued: 3,
    dropLowestPriority: true,
    clearOnDeath: true
  }
};
```

---

## Hint Definitions

### Core Gameplay Hints

```typescript
const CORE_GAMEPLAY_HINTS = {
  // Loot & Items
  firstChest: {
    id: 'hint_first_chest',
    trigger: {
      event: 'player_near_chest',
      condition: 'First unopened chest within 5m',
      playerState: 'never_opened_chest'
    },
    content: {
      text: 'Tap the chest to open it!',
      icon: 'chest_icon',
      pointer: 'Arrow pointing to chest'
    },
    priority: 'important',
    showCount: 1,
    cooldown: null // Show once ever
  },

  firstAbilityPickup: {
    id: 'hint_first_ability',
    trigger: {
      event: 'ability_in_range',
      condition: 'Ability loot within 3m',
      playerState: 'no_abilities_equipped'
    },
    content: {
      text: 'Pick up abilities to use special powers!',
      icon: 'ability_icon'
    },
    priority: 'important',
    showCount: 1
  },

  firstWeaponPickup: {
    id: 'hint_first_weapon',
    trigger: {
      event: 'weapon_in_range',
      condition: 'Better weapon within 3m',
      playerState: 'using_default_weapon'
    },
    content: {
      text: 'Grab weapons to deal more damage!',
      icon: 'weapon_icon'
    },
    priority: 'important',
    showCount: 1
  },

  abilitySlotFull: {
    id: 'hint_ability_swap',
    trigger: {
      event: 'ability_in_range',
      condition: 'All ability slots full',
      playerState: 'first_time_full_slots'
    },
    content: {
      text: 'Hold an ability button to swap abilities',
      icon: 'swap_icon'
    },
    priority: 'helpful',
    showCount: 2
  },

  // Combat
  firstEnemyNearby: {
    id: 'hint_first_enemy',
    trigger: {
      event: 'enemy_detected',
      condition: 'Enemy within 30m, first encounter',
      playerState: 'never_fought'
    },
    content: {
      text: 'Enemy nearby! Tap attack or use abilities!',
      icon: 'combat_icon'
    },
    priority: 'important',
    showCount: 1
  },

  abilityReady: {
    id: 'hint_ability_ready',
    trigger: {
      event: 'ability_off_cooldown',
      condition: 'In combat, ability ready, not used recently',
      playerState: 'first_combat_with_abilities'
    },
    content: {
      text: 'Your ability is ready! Tap to use it!',
      icon: 'ability_ready_icon',
      highlightElement: 'ability_button'
    },
    priority: 'helpful',
    showCount: 2
  },

  // Survival
  lowHealth: {
    id: 'hint_low_health',
    trigger: {
      event: 'health_changed',
      condition: 'Health < 30%',
      playerState: 'first_time_low_health'
    },
    content: {
      text: 'Health low! Find healing items or abilities!',
      icon: 'health_icon'
    },
    priority: 'critical',
    showCount: 2,
    cooldown: 60000 // Once per minute max
  },

  stormApproaching: {
    id: 'hint_storm_warning',
    trigger: {
      event: 'zone_warning',
      condition: 'Player outside next safe zone',
      playerState: 'first_storm_experience'
    },
    content: {
      text: 'Storm closing! Get to the safe zone!',
      icon: 'storm_icon',
      highlightElement: 'minimap'
    },
    priority: 'critical',
    showCount: 3,
    cooldown: 30000
  },

  inStormDamage: {
    id: 'hint_in_storm',
    trigger: {
      event: 'storm_damage_tick',
      condition: 'Taking storm damage',
      playerState: 'first_storm_damage'
    },
    content: {
      text: 'You\'re in the storm! Move to safety!',
      icon: 'danger_icon',
      urgent: true
    },
    priority: 'critical',
    showCount: 2,
    cooldown: 45000
  }
};
```

### Navigation & UI Hints

```typescript
const NAVIGATION_UI_HINTS = {
  minimapUsage: {
    id: 'hint_minimap',
    trigger: {
      event: 'match_start',
      condition: 'After 30 seconds of match',
      playerState: 'never_opened_full_map'
    },
    content: {
      text: 'Tap the minimap to see the full map!',
      icon: 'map_icon',
      highlightElement: 'minimap'
    },
    priority: 'helpful',
    showCount: 1
  },

  pingSystem: {
    id: 'hint_ping',
    trigger: {
      event: 'squad_mode_start',
      condition: 'In squad mode, first match',
      playerState: 'never_pinged'
    },
    content: {
      text: 'Double-tap the map to ping locations for your team!',
      icon: 'ping_icon'
    },
    priority: 'helpful',
    showCount: 1
  },

  inventoryManagement: {
    id: 'hint_inventory',
    trigger: {
      event: 'inventory_full',
      condition: 'Trying to pick up item with full inventory',
      playerState: 'first_full_inventory'
    },
    content: {
      text: 'Inventory full! Drop items to make room.',
      icon: 'inventory_icon'
    },
    priority: 'important',
    showCount: 2
  },

  settingsReminder: {
    id: 'hint_settings',
    trigger: {
      event: 'matches_played',
      condition: 'After 3 matches, never opened settings',
      playerState: 'never_opened_settings'
    },
    content: {
      text: 'Customize controls and graphics in Settings!',
      icon: 'settings_icon'
    },
    priority: 'informational',
    showCount: 1
  }
};
```

### Advanced Mechanic Hints

```typescript
const ADVANCED_HINTS = {
  abilityCombo: {
    id: 'hint_ability_combo',
    trigger: {
      event: 'multiple_abilities_equipped',
      condition: 'Has 2+ abilities, after 5 matches',
      playerState: 'ready_for_advanced_tips'
    },
    content: {
      text: 'Try combining abilities for powerful combos!',
      icon: 'combo_icon'
    },
    priority: 'informational',
    showCount: 1
  },

  highGroundAdvantage: {
    id: 'hint_high_ground',
    trigger: {
      event: 'combat_started',
      condition: 'Enemy has high ground advantage',
      playerState: 'first_elevation_combat'
    },
    content: {
      text: 'Take the high ground for an advantage!',
      icon: 'elevation_icon'
    },
    priority: 'helpful',
    showCount: 2
  },

  crouchMechanic: {
    id: 'hint_crouch',
    trigger: {
      event: 'standing_still',
      condition: 'Not moving for 5+ seconds in combat area',
      playerState: 'never_crouched'
    },
    content: {
      text: 'Crouch to reduce visibility and steady aim!',
      icon: 'crouch_icon'
    },
    priority: 'informational',
    showCount: 1
  }
};
```

---

## Display System

### Hint UI Component

```typescript
const HINT_UI_DESIGN = {
  container: {
    position: 'top-center',
    offset: { y: 80 }, // Below top status bar
    maxWidth: 320,
    minWidth: 200
  },

  appearance: {
    background: {
      color: 'rgba(0, 0, 0, 0.85)',
      borderRadius: 12,
      border: '2px solid rgba(255, 215, 0, 0.5)' // Gold accent
    },
    padding: {
      horizontal: 16,
      vertical: 12
    },
    shadow: {
      color: 'rgba(0, 0, 0, 0.3)',
      blur: 10,
      offset: { y: 4 }
    }
  },

  content: {
    icon: {
      size: 24,
      position: 'left',
      marginRight: 12
    },
    text: {
      fontSize: 14,
      fontWeight: 500,
      color: '#FFFFFF',
      lineHeight: 1.4,
      maxLines: 2
    },
    dismissButton: {
      icon: 'close',
      size: 20,
      position: 'top-right',
      hitArea: 44
    }
  },

  pointer: {
    enabled: true,
    type: 'Arrow or highlight',
    color: '#FFD700',
    animation: 'Gentle pulse'
  },

  urgentVariant: {
    border: '2px solid #FF4444',
    iconColor: '#FF4444',
    pulseAnimation: true
  }
};
```

### Animation System

```typescript
const HINT_ANIMATIONS = {
  appear: {
    type: 'slideDown + fadeIn',
    duration: 300,
    easing: 'easeOutBack',
    delay: 0
  },

  dismiss: {
    type: 'slideUp + fadeOut',
    duration: 200,
    easing: 'easeInQuad'
  },

  autoDismiss: {
    defaultDuration: 5000, // 5 seconds
    urgentDuration: 8000, // 8 seconds for critical
    pauseOnInteraction: true
  },

  attention: {
    type: 'Subtle scale pulse',
    trigger: 'After 2 seconds if not dismissed',
    animation: 'scale(1) -> scale(1.02) -> scale(1)',
    duration: 600
  },

  elementHighlight: {
    type: 'Glowing outline on UI element',
    color: '#FFD700',
    pulseSpeed: 1000,
    duration: 'Until hint dismissed'
  }
};
```

### Positioning Rules

```typescript
const HINT_POSITIONING = {
  defaultPosition: 'top-center',

  contextualPositioning: {
    enabled: true,
    rules: [
      {
        condition: 'Hint references bottom UI element',
        position: 'bottom-center',
        offset: { y: -100 }
      },
      {
        condition: 'Hint references left UI element',
        position: 'left-center',
        offset: { x: 100 }
      },
      {
        condition: 'Hint references minimap',
        position: 'Near minimap but not overlapping'
      }
    ]
  },

  avoidOverlap: {
    elements: [
      'Health bar',
      'Ability buttons',
      'Minimap',
      'Kill feed',
      'Active combat indicators'
    ],
    method: 'Shift position if overlap detected'
  },

  safeAreas: {
    respectNotch: true,
    respectHomeIndicator: true
  }
};
```

---

## Trigger System

### Event Subscriptions

```typescript
const HINT_TRIGGER_EVENTS = {
  gameEvents: [
    'player_near_loot',
    'player_near_chest',
    'ability_equipped',
    'weapon_equipped',
    'enemy_detected',
    'damage_taken',
    'health_changed',
    'zone_warning',
    'zone_damage',
    'match_start',
    'match_phase_change'
  ],

  playerStateEvents: [
    'inventory_changed',
    'ability_cooldown_complete',
    'standing_still',
    'in_combat',
    'out_of_combat'
  ],

  progressionEvents: [
    'matches_completed',
    'first_kill',
    'first_win',
    'tutorial_completed'
  ]
};
```

### Condition Evaluation

```typescript
const CONDITION_EVALUATION = {
  playerState: {
    tracking: {
      chestsOpened: 'number',
      abilitiesUsed: 'number',
      matchesPlayed: 'number',
      hintsShown: 'Map<hintId, count>',
      hintsDismissed: 'Set<hintId>',
      featuresUsed: 'Set<featureId>'
    },
    persistence: 'Local storage, synced with account'
  },

  cooldownManagement: {
    globalCooldown: 10000, // 10 seconds between any hints
    perHintCooldown: 'Defined per hint',
    combatCooldown: 5000 // Delay hints during active combat
  },

  suppressionRules: [
    {
      condition: 'Player in active combat (last 3 seconds)',
      suppress: 'All except critical hints'
    },
    {
      condition: 'Another hint currently displayed',
      suppress: 'Lower priority hints (queue higher priority)'
    },
    {
      condition: 'Player has dismissed this hint type',
      suppress: 'This hint permanently'
    },
    {
      condition: 'Hints disabled in settings',
      suppress: 'All hints'
    }
  ]
};
```

---

## Settings & Preferences

### Hint Settings UI

```typescript
const HINT_SETTINGS = {
  location: 'Settings > Gameplay > Hints',

  options: {
    masterToggle: {
      id: 'hints_enabled',
      label: 'Show Gameplay Hints',
      type: 'toggle',
      default: true,
      description: 'Display helpful tips during gameplay'
    },

    hintFrequency: {
      id: 'hint_frequency',
      label: 'Hint Frequency',
      type: 'segmented',
      options: [
        { value: 'all', label: 'All Hints' },
        { value: 'important', label: 'Important Only' },
        { value: 'critical', label: 'Critical Only' }
      ],
      default: 'all',
      showWhen: 'hints_enabled'
    },

    resetHints: {
      id: 'reset_hints',
      label: 'Reset All Hints',
      type: 'button',
      action: 'resetAllHintProgress()',
      confirmationRequired: true,
      confirmMessage: 'This will show all hints again. Continue?'
    }
  },

  categoryToggles: {
    enabled: false, // Keep it simple for now
    futureFeature: 'Allow toggling hint categories individually'
  }
};
```

### Player Progression Integration

```typescript
const PROGRESSION_INTEGRATION = {
  newPlayerDetection: {
    criteria: [
      'matchesPlayed < 5',
      'tutorialCompleted === false',
      'accountAge < 7 days'
    ],
    behavior: 'Show all applicable hints'
  },

  experiencedPlayerDetection: {
    criteria: [
      'matchesPlayed >= 20',
      'averageKills >= 2',
      'winsCount >= 1'
    ],
    behavior: 'Only show critical and new feature hints'
  },

  returningPlayerDetection: {
    criteria: [
      'lastPlayedDate > 30 days ago',
      'matchesPlayed >= 10'
    ],
    behavior: 'Show refresher hints for major mechanics'
  },

  adaptiveBehavior: {
    reduceHintsOnDismiss: true,
    trackInteractionPatterns: true,
    adjustFrequency: 'Based on player behavior'
  }
};
```

---

## Hint Content Guidelines

### Writing Style

```typescript
const HINT_WRITING_GUIDELINES = {
  tone: {
    style: 'Friendly, concise, action-oriented',
    voice: 'Second person ("you", "your")',
    avoid: ['Jargon', 'Lengthy explanations', 'Condescending language']
  },

  formatting: {
    maxCharacters: 60,
    preferredCharacters: 40,
    useActionVerbs: true,
    endWithExclamation: 'For encouragement, not shouting'
  },

  examples: {
    good: [
      'Tap the chest to open it!',
      'Storm closing! Head to safety!',
      'Ability ready - tap to use!'
    ],
    avoid: [
      'You should probably consider opening that chest over there',
      'WARNING: The storm is approaching your location',
      'Did you know you can use abilities?'
    ]
  },

  localization: {
    required: true,
    maxExpansion: '30%', // Account for longer translations
    testLanguages: ['German', 'French', 'Japanese', 'Portuguese']
  }
};
```

---

## Analytics

```typescript
const HINT_ANALYTICS = {
  events: [
    {
      name: 'hint_displayed',
      properties: {
        hint_id: 'string',
        trigger_event: 'string',
        player_matches: 'number',
        time_in_match: 'seconds'
      }
    },
    {
      name: 'hint_dismissed',
      properties: {
        hint_id: 'string',
        dismiss_method: 'tap | timeout | auto',
        display_duration: 'milliseconds'
      }
    },
    {
      name: 'hint_action_taken',
      properties: {
        hint_id: 'string',
        action: 'string', // e.g., 'opened_chest', 'used_ability'
        time_after_hint: 'seconds'
      }
    },
    {
      name: 'hints_disabled',
      properties: {
        player_matches: 'number',
        hints_seen_count: 'number'
      }
    }
  ],

  metrics: [
    'Hint display rate by player experience level',
    'Hint dismiss rate (manual vs timeout)',
    'Action completion rate after hint',
    'Hint disable rate',
    'Most effective hints (by action rate)'
  ],

  optimization: {
    abTesting: 'Test different hint texts',
    triggerTiming: 'Optimize when hints appear',
    contentIteration: 'Refine based on effectiveness'
  }
};
```

---

## Extensibility

### Adding New Hints

```typescript
const HINT_EXTENSIBILITY = {
  newHintTemplate: {
    id: 'hint_unique_id',
    trigger: {
      event: 'game_event_name',
      condition: 'Boolean expression or function',
      playerState: 'Required player state check'
    },
    content: {
      text: 'Concise hint text',
      icon: 'icon_name',
      highlightElement: 'optional_ui_element_id'
    },
    priority: 'critical | important | helpful | informational',
    showCount: 1, // Times to show before permanently hiding
    cooldown: null // Milliseconds, or null for no repeat
  },

  registration: {
    method: 'HintManager.registerHint(hintDefinition)',
    timing: 'During game initialization',
    validation: 'Schema validation of hint definition'
  },

  featureFlags: {
    enabled: true,
    useCase: 'Enable/disable specific hints remotely',
    fallback: 'Local defaults if server unavailable'
  }
};
```

---

## Testing Requirements

```typescript
const HINT_TESTING = {
  functionalTests: [
    'All hints trigger at correct moments',
    'Hints dismiss correctly (tap, timeout)',
    'Hints respect cooldowns',
    'Hints persist across sessions',
    'Hint settings work correctly',
    'Reset hints function works'
  ],

  uxTests: [
    'Hints don\'t obscure critical gameplay',
    'Hints are readable in all lighting conditions',
    'Hints don\'t appear too frequently',
    'New players find hints helpful',
    'Experienced players aren\'t annoyed'
  ],

  performanceTests: [
    'Hint system has minimal CPU overhead',
    'No frame drops when hints appear',
    'Hint persistence doesn\'t affect load times'
  ],

  localizationTests: [
    'All hints fit in UI at max text expansion',
    'Hints are culturally appropriate',
    'No text truncation in any language'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial contextual hints system specification |
