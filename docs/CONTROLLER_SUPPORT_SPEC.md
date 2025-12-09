# Controller Support Specification

## Document Information
- **Task ID:** UX-007
- **Priority:** P3
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies controller support for Plunderstorm Mobile, enabling players to use external Bluetooth and USB controllers for a console-like gaming experience while maintaining fair play with touch-based players.

---

## Supported Controllers

### Controller Compatibility

```typescript
const SUPPORTED_CONTROLLERS = {
  ios: {
    mfi: {
      name: 'MFi Controllers',
      examples: ['SteelSeries Nimbus+', 'Razer Kishi', 'Backbone One'],
      support: 'Full native support via Game Controller framework'
    },
    playstation: {
      name: 'PlayStation Controllers',
      models: ['DualShock 4', 'DualSense (PS5)'],
      support: 'Native iOS 14.5+ support',
      features: ['Touchpad (limited)', 'Adaptive triggers (DualSense)', 'Haptics']
    },
    xbox: {
      name: 'Xbox Controllers',
      models: ['Xbox One Controller', 'Xbox Series X|S Controller'],
      support: 'Native iOS 14.5+ support',
      features: ['Impulse triggers', 'Share button']
    }
  },

  android: {
    standard: {
      name: 'Standard Android Controllers',
      examples: ['Razer Raiju', 'SteelSeries Stratus'],
      support: 'Android input framework'
    },
    playstation: {
      name: 'PlayStation Controllers',
      models: ['DualShock 4', 'DualSense'],
      support: 'Android 10+ Bluetooth support',
      features: ['Touchpad', 'Motion sensors']
    },
    xbox: {
      name: 'Xbox Controllers',
      models: ['Xbox One', 'Xbox Series X|S'],
      support: 'Native Android support',
      features: ['Bluetooth and USB']
    },
    generic: {
      name: 'Generic HID Controllers',
      support: 'Basic button mapping',
      limitation: 'May require manual configuration'
    }
  },

  attachments: {
    clipOn: {
      examples: ['Razer Kishi', 'Backbone One', 'GameSir X2'],
      type: 'Phone clip controllers',
      connection: 'Lightning/USB-C direct connect',
      latency: 'Lowest latency option'
    }
  }
};
```

### Detection System

```typescript
const CONTROLLER_DETECTION = {
  automatic: {
    polling: 'Check for controllers every 2 seconds',
    events: 'Listen for connection/disconnection events',
    hotPlug: 'Support connecting controller mid-game'
  },

  identification: {
    vendorId: 'Read vendor/product IDs',
    name: 'Display controller name to user',
    type: 'Categorize as PlayStation/Xbox/MFi/Generic',
    capabilities: 'Detect available features (triggers, touchpad, etc.)'
  },

  notification: {
    connected: 'Controller connected toast notification',
    disconnected: 'Controller disconnected warning',
    lowBattery: 'Controller battery low warning (if supported)'
  }
};
```

---

## Button Mapping

### Default Layout

```typescript
const DEFAULT_MAPPING = {
  movement: {
    leftStick: 'Character movement',
    leftStickClick: 'Sprint toggle'
  },

  camera: {
    rightStick: 'Camera/aim control',
    rightStickClick: 'Center camera'
  },

  abilities: {
    rightTrigger: 'Primary ability (Ability 1)',
    leftTrigger: 'Secondary ability (Ability 2)',
    rightBumper: 'Ability 3',
    leftBumper: 'Ability 4 / Ultimate'
  },

  actions: {
    buttonA: 'Interact / Loot / Confirm',
    buttonB: 'Cancel / Close menu',
    buttonX: 'Reload / Use item',
    buttonY: 'Switch ability / Cycle weapon'
  },

  utility: {
    dpadUp: 'Emote wheel',
    dpadDown: 'Ping wheel',
    dpadLeft: 'Map toggle',
    dpadRight: 'Inventory quick access'
  },

  system: {
    start: 'Pause menu',
    select: 'Scoreboard / Map',
    home: 'System (handled by OS)'
  }
};
```

### Alternative Layouts

```typescript
const ALTERNATIVE_LAYOUTS = {
  tactical: {
    name: 'Tactical',
    description: 'Abilities on face buttons',
    changes: {
      buttonA: 'Ability 1',
      buttonB: 'Ability 2',
      buttonX: 'Ability 3',
      buttonY: 'Ability 4',
      rightTrigger: 'Interact',
      leftTrigger: 'Ping'
    }
  },

  legacy: {
    name: 'Legacy',
    description: 'Movement on right stick',
    changes: {
      leftStick: 'Camera/aim',
      rightStick: 'Movement'
    }
  },

  southpaw: {
    name: 'Southpaw',
    description: 'Swapped stick functions',
    changes: {
      leftStick: 'Camera/aim',
      rightStick: 'Movement',
      leftTrigger: 'Primary ability',
      rightTrigger: 'Secondary ability'
    }
  },

  accessible: {
    name: 'One-Handed',
    description: 'All actions on one side',
    changes: {
      description: 'Optimized for single-hand play'
    }
  }
};
```

### Custom Mapping

```typescript
const CUSTOM_MAPPING = {
  enabled: true,
  access: 'Settings > Controls > Controller > Customize',

  options: {
    perButton: 'Remap any button to any action',
    stickSensitivity: {
      movement: { min: 0.5, max: 2.0, default: 1.0 },
      camera: { min: 0.5, max: 3.0, default: 1.0 }
    },
    deadzone: {
      inner: { min: 0.05, max: 0.3, default: 0.15 },
      outer: { min: 0.7, max: 0.95, default: 0.9 }
    },
    triggerThreshold: {
      min: 0.1,
      max: 0.9,
      default: 0.5
    }
  },

  profiles: {
    count: 3,
    naming: 'Custom profile names',
    switching: 'Quick switch between profiles'
  },

  import: {
    share: 'Share custom layouts via code',
    community: 'Browse popular community layouts'
  }
};
```

---

## UI Adaptation

### Button Prompts

```typescript
const BUTTON_PROMPTS = {
  automatic: {
    detection: 'Show prompts matching connected controller',
    switching: 'Instant switch when input source changes'
  },

  iconSets: {
    playstation: {
      confirm: 'Cross (×)',
      cancel: 'Circle (○)',
      action1: 'Square (□)',
      action2: 'Triangle (△)',
      bumpers: 'L1/R1',
      triggers: 'L2/R2'
    },
    xbox: {
      confirm: 'A',
      cancel: 'B',
      action1: 'X',
      action2: 'Y',
      bumpers: 'LB/RB',
      triggers: 'LT/RT'
    },
    nintendo: {
      confirm: 'A (right)',
      cancel: 'B (bottom)',
      action1: 'Y (top)',
      action2: 'X (left)',
      bumpers: 'L/R',
      triggers: 'ZL/ZR'
    },
    generic: {
      confirm: 'Button 1',
      cancel: 'Button 2',
      description: 'Numbered/lettered fallback'
    }
  },

  display: {
    contextual: 'Show prompts near interactive elements',
    hud: 'Ability bar shows mapped buttons',
    menus: 'All menu options show controller shortcuts'
  }
};
```

### Menu Navigation

```typescript
const MENU_NAVIGATION = {
  selection: {
    dpad: 'Navigate between UI elements',
    leftStick: 'Alternative navigation',
    wrap: 'Cursor wraps at edges'
  },

  actions: {
    confirm: 'A/Cross to select',
    back: 'B/Circle to go back',
    tabs: 'Bumpers to switch tabs',
    scroll: 'Right stick to scroll lists'
  },

  cursor: {
    mode: 'Optional virtual cursor mode',
    toggle: 'Right stick click to toggle',
    sensitivity: 'Adjustable cursor speed'
  },

  focus: {
    indicator: 'Clear visual focus state',
    animation: 'Subtle highlight animation',
    audio: 'Navigation sound feedback'
  }
};
```

---

## Hybrid Input

### Simultaneous Support

```typescript
const HYBRID_INPUT = {
  simultaneous: {
    enabled: true,
    description: 'Use touch and controller together',
    useCase: 'Controller for movement, touch for abilities'
  },

  switching: {
    automatic: true,
    detection: 'Last input method takes priority',
    promptSwitch: 'UI prompts update within 0.5s'
  },

  perAction: {
    movement: 'Controller or touch joystick',
    abilities: 'Controller buttons or touch ability buttons',
    menus: 'Controller or touch tap',
    typing: 'Always show on-screen keyboard'
  }
};
```

### Touch Fallback

```typescript
const TOUCH_FALLBACK = {
  disconnection: {
    behavior: 'Seamlessly switch to touch controls',
    notification: 'Alert player of disconnection',
    pauseOption: 'Optional auto-pause on disconnect'
  },

  unavailableFeatures: {
    touchpad: 'Map to touch gesture or button combo',
    motionControls: 'Use device gyro or disable',
    adaptiveTriggers: 'N/A on touch'
  }
};
```

---

## Haptic Feedback

### Vibration Support

```typescript
const HAPTIC_FEEDBACK = {
  supported: {
    xbox: 'Rumble motors (standard)',
    playstation: 'Haptic feedback (DualSense) or rumble (DS4)',
    mfi: 'Basic rumble (device dependent)'
  },

  events: {
    damage: {
      taken: 'Medium rumble on receiving damage',
      dealt: 'Light pulse on hitting enemy'
    },
    abilities: {
      cast: 'Ability-specific feedback',
      cooldown: 'Subtle pulse when ability ready'
    },
    environment: {
      explosion: 'Strong rumble nearby explosions',
      storm: 'Continuous light rumble in storm'
    },
    ui: {
      confirm: 'Light tap on selection',
      error: 'Double tap on invalid action'
    }
  },

  settings: {
    intensity: { min: 0, max: 100, default: 70 },
    disable: 'Option to turn off completely',
    perEvent: 'Granular control per event type'
  }
};
```

### DualSense Features

```typescript
const DUALSENSE_FEATURES = {
  adaptiveTriggers: {
    enabled: true,
    effects: {
      charging: 'Resistance while charging ability',
      cooldown: 'Locked trigger during cooldown',
      bowPull: 'Progressive resistance on bow-type abilities'
    }
  },

  hapticFeedback: {
    enabled: true,
    effects: {
      terrain: 'Different feedback for terrain types',
      weather: 'Rain, storm effects through haptics',
      abilities: 'Unique haptic signature per ability'
    }
  },

  lightbar: {
    enabled: true,
    effects: {
      health: 'Color reflects health status',
      team: 'Team color in squad modes',
      damage: 'Flash red when hit'
    }
  }
};
```

---

## Aim Assist

### Controller Aim Assist

```typescript
const AIM_ASSIST = {
  enabled: true,
  reason: 'Balance controller precision vs touch',

  types: {
    slowdown: {
      description: 'Reduce sensitivity when near target',
      strength: { min: 0, max: 100, default: 50 },
      radius: 'Configurable detection radius'
    },
    magnetism: {
      description: 'Subtle pull toward targets',
      strength: { min: 0, max: 100, default: 30 },
      behavior: 'Only when aiming at enemy'
    },
    rotational: {
      description: 'Assist when target moves',
      strength: { min: 0, max: 100, default: 40 },
      tracking: 'Help track moving enemies'
    }
  },

  settings: {
    master: 'Global aim assist toggle',
    strength: 'Overall strength slider',
    perType: 'Individual type adjustment'
  },

  fairness: {
    touchComparison: 'Tuned to match average touch player accuracy',
    rankedReduction: 'Optional reduced assist in ranked (server config)'
  }
};
```

---

## Matchmaking Considerations

### Input-Based Matching

```typescript
const INPUT_MATCHMAKING = {
  default: {
    mode: 'Mixed input matchmaking',
    description: 'All input types play together',
    reason: 'Faster queue times, aim assist balances'
  },

  optional: {
    sameInput: {
      setting: 'Prefer same input type',
      behavior: 'Try to match with same input, expand if needed',
      queueImpact: 'May increase queue time'
    },
    controllerOnly: {
      setting: 'Controller players only',
      behavior: 'Only match with other controller users',
      queueImpact: 'Significantly longer queues',
      availability: 'May not be available in all regions'
    }
  },

  display: {
    icon: 'Show input type icon next to player names',
    lobby: 'Show input distribution in party/lobby',
    killFeed: 'Optional: show input type on eliminations'
  },

  partyBehavior: {
    mixed: 'Party with mixed inputs joins mixed queue',
    override: 'Party leader preference applies to party'
  }
};
```

---

## Settings UI

### Controller Settings Menu

```typescript
const CONTROLLER_SETTINGS = {
  location: 'Settings > Controls > Controller',

  sections: {
    general: {
      controllerEnabled: 'Enable controller input',
      vibration: 'Vibration intensity',
      layout: 'Button layout preset',
      customize: 'Custom button mapping'
    },
    sticks: {
      moveSensitivity: 'Movement sensitivity',
      aimSensitivity: 'Camera/aim sensitivity',
      invertY: 'Invert Y axis',
      invertX: 'Invert X axis',
      deadzone: 'Stick deadzone'
    },
    triggers: {
      threshold: 'Trigger activation threshold',
      swapTriggers: 'Swap L/R triggers'
    },
    aimAssist: {
      enabled: 'Enable aim assist',
      strength: 'Aim assist strength',
      slowdown: 'Target slowdown',
      magnetism: 'Target magnetism'
    },
    advanced: {
      responseType: 'Stick response curve (linear/exponential)',
      hapticStrength: 'Haptic feedback intensity',
      adaptiveTriggers: 'Enable adaptive triggers (DualSense)'
    }
  },

  preview: {
    testArea: 'Test inputs in settings',
    visualization: 'Show stick/trigger input visualization'
  }
};
```

---

## Technical Implementation

### Platform Integration

```typescript
const PLATFORM_INTEGRATION = {
  ios: {
    framework: 'Game Controller framework',
    detection: 'GCController.controllers()',
    profiles: ['GCExtendedGamepad', 'GCMicroGamepad'],
    events: 'NotificationCenter for connect/disconnect'
  },

  android: {
    api: 'InputDevice and MotionEvent',
    detection: 'InputManager.getInputDeviceIds()',
    mapping: 'KeyEvent for buttons, MotionEvent for sticks',
    events: 'InputDeviceListener'
  },

  unity: {
    inputSystem: 'New Input System package',
    binding: 'InputAction bindings',
    schemes: 'Control schemes for gamepad/touch'
  }
};
```

### Data Model

```typescript
const DATA_MODEL = {
  controllerSettings: {
    playerId: 'string',
    enabled: 'boolean',
    layout: 'string (preset name)',
    customMappings: 'object<action, button>',
    sensitivity: {
      movement: 'number',
      aim: 'number'
    },
    deadzone: {
      inner: 'number',
      outer: 'number'
    },
    invertY: 'boolean',
    invertX: 'boolean',
    vibrationIntensity: 'number',
    aimAssist: {
      enabled: 'boolean',
      strength: 'number'
    }
  },

  controllerProfiles: {
    playerId: 'string',
    profiles: 'array<ControllerProfile>',
    activeProfile: 'number'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  controllers: [
    'Xbox Series X controller (Bluetooth)',
    'Xbox One controller (Bluetooth)',
    'DualShock 4 (Bluetooth)',
    'DualSense (Bluetooth)',
    'MFi controller (various)',
    'Razer Kishi / Backbone (clip-on)',
    'Generic USB/Bluetooth controller'
  ],

  functional: [
    'All buttons map correctly',
    'Sticks respond with correct sensitivity',
    'Hot-plug connection/disconnection',
    'UI prompts update correctly',
    'Haptic feedback works',
    'Settings persist across sessions'
  ],

  platforms: [
    'iOS 14+ devices',
    'Android 10+ devices',
    'Various screen sizes'
  ],

  edge: [
    'Controller disconnects mid-match',
    'Multiple controllers connected',
    'Switching between controller and touch',
    'Low battery handling',
    'Bluetooth interference'
  ],

  performance: [
    'Input latency < 16ms',
    'No frame drops from controller polling',
    'Battery impact acceptable'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial controller support specification |
