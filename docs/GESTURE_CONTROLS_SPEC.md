# Gesture Controls Specification

## Document Information
- **Task ID:** UX-006
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies additional gesture controls for quality-of-life actions in Plunderstorm Mobile. These gestures provide alternative input methods that complement (not replace) existing button-based controls.

---

## Gesture Definitions

### Core Gestures

```typescript
const GESTURE_DEFINITIONS = {
  doubleTapGround: {
    name: 'Double-Tap Ground',
    gesture: {
      type: 'multi-tap',
      taps: 2,
      maxInterval: 300, // ms between taps
      targetZone: 'gameplay_area', // Not on UI elements
      fingerCount: 1
    },
    action: 'Interact with nearest loot/chest',
    priority: 'High', // Most commonly used
    defaultEnabled: true
  },

  pinchMinimap: {
    name: 'Pinch Minimap',
    gesture: {
      type: 'pinch',
      targetZone: 'minimap_area',
      fingerCount: 2,
      minScale: 0.5,
      maxScale: 2.0
    },
    action: 'Zoom minimap in/out',
    priority: 'Medium',
    defaultEnabled: true
  },

  threeFingerTap: {
    name: 'Three-Finger Tap',
    gesture: {
      type: 'tap',
      fingerCount: 3,
      targetZone: 'anywhere',
      holdDuration: 0 // Instant tap
    },
    action: 'Open quick menu',
    priority: 'Low', // Optional feature
    defaultEnabled: false // Disabled by default
  },

  edgeSwipeScoreboard: {
    name: 'Edge Swipe',
    gesture: {
      type: 'swipe',
      startZone: 'screen_edge_right',
      direction: 'left',
      minDistance: 50, // pixels
      fingerCount: 1
    },
    action: 'Open scoreboard',
    priority: 'Low', // Optional feature
    defaultEnabled: false // Disabled by default
  }
};
```

---

## Gesture Recognition System

### Recognition Architecture

```typescript
const GESTURE_RECOGNIZER = {
  pipeline: {
    stage1_input: {
      name: 'Touch Input Collection',
      responsibilities: [
        'Collect all touch points',
        'Track touch lifecycle (down, move, up)',
        'Calculate touch velocities',
        'Identify touch zones'
      ]
    },

    stage2_classification: {
      name: 'Gesture Classification',
      responsibilities: [
        'Match touch patterns to gesture definitions',
        'Handle gesture priority conflicts',
        'Track multi-touch states',
        'Detect gesture start/progress/complete'
      ]
    },

    stage3_validation: {
      name: 'Context Validation',
      responsibilities: [
        'Check if gesture is enabled in settings',
        'Verify gesture zone is valid',
        'Ensure no conflict with core controls',
        'Validate game state allows gesture'
      ]
    },

    stage4_execution: {
      name: 'Action Execution',
      responsibilities: [
        'Trigger associated action',
        'Provide visual/haptic feedback',
        'Log gesture analytics',
        'Handle edge cases'
      ]
    }
  }
};
```

### Touch Zone Definitions

```typescript
const TOUCH_ZONES = {
  gameplay_area: {
    description: 'Main gameplay viewport',
    bounds: {
      excludes: ['all_ui_elements', 'minimap', 'ability_buttons', 'joystick_area']
    },
    gestures: ['doubleTapGround']
  },

  minimap_area: {
    description: 'Minimap region',
    bounds: {
      type: 'ui_element',
      elementId: 'minimap_container'
    },
    gestures: ['pinchMinimap']
  },

  screen_edge_right: {
    description: 'Right edge of screen',
    bounds: {
      type: 'edge',
      edge: 'right',
      width: 30 // pixels from edge
    },
    gestures: ['edgeSwipeScoreboard']
  },

  anywhere: {
    description: 'Entire screen',
    bounds: {
      type: 'full_screen'
    },
    gestures: ['threeFingerTap']
  }
};
```

---

## Gesture Implementations

### 1. Double-Tap Ground (Interact)

```typescript
const DOUBLE_TAP_INTERACT = {
  detection: {
    firstTap: {
      record: 'position and timestamp',
      startTimer: 300 // ms window for second tap
    },
    secondTap: {
      validate: [
        'Within 300ms of first tap',
        'Within 50px of first tap position',
        'Not on UI element',
        'Not during ability cast'
      ],
      onSuccess: 'triggerInteraction()'
    }
  },

  interaction: {
    searchRadius: 3, // meters from character
    priority: [
      'Treasure chest (unopened)',
      'Ground loot (weapon)',
      'Ground loot (ability)',
      'Ground loot (consumable)',
      'Interactive object'
    ],
    behavior: {
      noTarget: 'Show "Nothing nearby" indicator briefly',
      singleTarget: 'Interact immediately',
      multipleTargets: 'Interact with highest priority/nearest'
    }
  },

  feedback: {
    visual: {
      onFirstTap: 'Subtle ripple effect at tap position',
      onSuccess: 'Highlight interacted object briefly',
      onNoTarget: 'Dim pulse at tap position'
    },
    haptic: {
      onSuccess: 'Light impact (UIImpactFeedbackGenerator.light)',
      onNoTarget: 'None'
    },
    audio: {
      onSuccess: 'Pickup/interact sound from object',
      onNoTarget: 'None'
    }
  },

  conflictPrevention: {
    cooldown: 200, // ms before another double-tap can register
    cancelConditions: [
      'Touch moves > 50px (becomes drag)',
      'Third tap detected (ignore sequence)',
      'UI element receives touch',
      'Ability button pressed'
    ]
  }
};
```

### 2. Pinch Minimap (Zoom)

```typescript
const PINCH_MINIMAP_ZOOM = {
  detection: {
    start: {
      condition: 'Two fingers touch minimap area',
      record: 'Initial finger distance'
    },
    update: {
      calculateScale: 'current_distance / initial_distance',
      applyScale: 'Continuously update minimap zoom'
    },
    end: {
      condition: 'Either finger lifted',
      persist: 'Remember zoom level for session'
    }
  },

  zoomLevels: {
    minimum: 0.5, // Zoomed out (larger area visible)
    default: 1.0, // Default view
    maximum: 2.0, // Zoomed in (smaller area, more detail)
    stepSize: 0.1, // Granularity of zoom
    snapToDefault: {
      enabled: true,
      threshold: 0.9, // Snap to 1.0 if within ±0.1
      triggerOnRelease: true
    }
  },

  feedback: {
    visual: {
      duringPinch: 'Minimap border highlight',
      zoomIndicator: 'Small "x1.5" text showing current zoom',
      fadeIndicator: 'After 1 second of no pinching'
    },
    haptic: {
      onZoomLimit: 'Light impact when hitting min/max',
      onSnapToDefault: 'Selection tick'
    }
  },

  persistence: {
    sessionBased: true,
    saveToSettings: false, // Resets each match
    defaultOnNewMatch: 1.0
  }
};
```

### 3. Three-Finger Tap (Quick Menu)

```typescript
const THREE_FINGER_QUICK_MENU = {
  detection: {
    requirement: 'Three fingers touch screen simultaneously',
    tolerance: {
      timing: 100, // ms window for all fingers to land
      spread: 'Any spread, no minimum distance'
    },
    validation: [
      'Not during active combat (recently damaged)',
      'Not while casting ability',
      'Menu not already open'
    ]
  },

  quickMenu: {
    contents: [
      { icon: 'settings', action: 'Open Settings', position: 'top' },
      { icon: 'map', action: 'Expand Map', position: 'left' },
      { icon: 'emote', action: 'Emote Wheel', position: 'right' },
      { icon: 'surrender', action: 'Surrender (if available)', position: 'bottom' }
    ],
    layout: 'radial',
    behavior: {
      onOpen: 'Pause game input briefly',
      selection: 'Drag to option and release',
      dismiss: 'Tap outside or three-finger tap again'
    }
  },

  feedback: {
    visual: {
      onTrigger: 'Menu appears with scale-up animation',
      options: 'Icons pulse subtly when hovered'
    },
    haptic: {
      onOpen: 'Medium impact',
      onHover: 'Selection tick',
      onSelect: 'Success notification'
    }
  },

  defaultState: {
    enabled: false,
    reason: 'Advanced feature, may conflict with three-finger screenshot on some devices'
  }
};
```

### 4. Edge Swipe (Scoreboard)

```typescript
const EDGE_SWIPE_SCOREBOARD = {
  detection: {
    startZone: {
      edge: 'right',
      width: 30, // pixels from screen edge
      excludeNotch: true // Account for notched displays
    },
    swipe: {
      direction: 'left',
      minDistance: 80, // pixels
      maxDuration: 500, // ms
      minVelocity: 200 // pixels/second
    }
  },

  scoreboard: {
    display: 'Slide in from right',
    contents: [
      'Player rankings (kills, placement)',
      'Squad status (if squad mode)',
      'Players remaining',
      'Match time'
    ],
    behavior: {
      holdToView: false, // Stays open until dismissed
      dismiss: 'Swipe right or tap outside'
    }
  },

  feedback: {
    visual: {
      onSwipeStart: 'Scoreboard edge peek (10px visible)',
      onSwipeProgress: 'Scoreboard follows finger',
      onSwipeComplete: 'Scoreboard slides fully in'
    },
    haptic: {
      onOpen: 'Light impact',
      onClose: 'Light impact'
    }
  },

  conflictPrevention: {
    disableDuring: [
      'Aiming with right thumb',
      'Using ability with right-side buttons',
      'Camera rotation active'
    ],
    alternativeAccess: 'Tap scoreboard button in HUD'
  },

  defaultState: {
    enabled: false,
    reason: 'May conflict with camera controls on right side'
  }
};
```

---

## Conflict Prevention System

### Priority Resolution

```typescript
const GESTURE_PRIORITY_SYSTEM = {
  priorities: {
    // Lower number = higher priority
    1: 'Core movement (joystick)',
    2: 'Core combat (attack, abilities)',
    3: 'Camera control',
    4: 'UI button interactions',
    5: 'Optional gestures'
  },

  conflictRules: [
    {
      gesture: 'doubleTapGround',
      conflicts: ['attack_button', 'ability_buttons'],
      resolution: 'Buttons always win, gesture only on empty ground'
    },
    {
      gesture: 'pinchMinimap',
      conflicts: ['none'], // Minimap is dedicated zone
      resolution: 'Pinch only active within minimap bounds'
    },
    {
      gesture: 'threeFingerTap',
      conflicts: ['three_finger_screenshot', 'ability_combos'],
      resolution: 'Disabled by default, user opt-in required'
    },
    {
      gesture: 'edgeSwipeScoreboard',
      conflicts: ['camera_pan_right', 'right_thumb_actions'],
      resolution: 'Disabled by default, only activates from screen edge'
    }
  ],

  safetyMargins: {
    uiElements: 20, // pixels buffer around UI
    edgeZones: 30, // pixels for edge detection
    gestureDeadzone: 50 // pixels movement cancels tap gestures
  }
};
```

### Combat Lockout

```typescript
const COMBAT_GESTURE_LOCKOUT = {
  lockConditions: [
    {
      condition: 'Player taking damage',
      lockDuration: 500, // ms after last damage
      affectedGestures: ['threeFingerTap']
    },
    {
      condition: 'Player attacking',
      lockDuration: 'Until attack animation complete',
      affectedGestures: ['threeFingerTap', 'edgeSwipeScoreboard']
    },
    {
      condition: 'Ability casting',
      lockDuration: 'Until cast complete',
      affectedGestures: ['all_optional_gestures']
    }
  ],

  neverLocked: [
    'doubleTapGround', // Important for quick looting mid-combat
    'pinchMinimap' // Safe zone, doesn't affect combat
  ]
};
```

---

## Settings Configuration

### Gesture Settings UI

```typescript
const GESTURE_SETTINGS = {
  location: 'Settings > Controls > Gestures',

  options: {
    masterToggle: {
      name: 'Enable Gesture Controls',
      type: 'toggle',
      default: true,
      description: 'Turn all gesture controls on or off'
    },

    individualToggles: [
      {
        name: 'Double-Tap to Interact',
        key: 'gesture_double_tap',
        default: true,
        description: 'Double-tap ground to pick up nearby loot'
      },
      {
        name: 'Pinch to Zoom Minimap',
        key: 'gesture_pinch_minimap',
        default: true,
        description: 'Use two fingers to zoom the minimap'
      },
      {
        name: 'Three-Finger Quick Menu',
        key: 'gesture_three_finger',
        default: false,
        description: 'Tap with three fingers to open quick menu'
      },
      {
        name: 'Edge Swipe Scoreboard',
        key: 'gesture_edge_swipe',
        default: false,
        description: 'Swipe from right edge to open scoreboard'
      }
    ],

    sensitivity: {
      name: 'Gesture Sensitivity',
      type: 'slider',
      range: [0.5, 1.5],
      default: 1.0,
      description: 'Adjust how easily gestures are detected'
    }
  }
};
```

### Help Documentation

```typescript
const GESTURE_HELP_CONTENT = {
  location: 'Settings > Help > Gesture Controls',

  content: {
    overview: {
      title: 'Gesture Controls',
      text: 'Use touch gestures for quick actions. All gestures can also be performed using on-screen buttons.'
    },

    gestureGuides: [
      {
        name: 'Double-Tap to Interact',
        animation: 'double_tap_tutorial.gif',
        steps: [
          'Tap the ground twice quickly',
          'Nearby loot will be picked up automatically',
          'Works within 3 meters of your character'
        ]
      },
      {
        name: 'Pinch to Zoom Minimap',
        animation: 'pinch_minimap_tutorial.gif',
        steps: [
          'Place two fingers on the minimap',
          'Pinch in to zoom out (see more area)',
          'Pinch out to zoom in (see more detail)'
        ]
      },
      {
        name: 'Three-Finger Quick Menu',
        animation: 'three_finger_tutorial.gif',
        steps: [
          'Tap the screen with three fingers at once',
          'Quick menu will appear',
          'Drag to select an option'
        ],
        note: 'Must be enabled in Settings > Controls > Gestures'
      },
      {
        name: 'Edge Swipe Scoreboard',
        animation: 'edge_swipe_tutorial.gif',
        steps: [
          'Start your finger at the right edge of the screen',
          'Swipe left quickly',
          'Scoreboard will slide in'
        ],
        note: 'Must be enabled in Settings > Controls > Gestures'
      }
    ],

    troubleshooting: [
      {
        problem: 'Gesture not working',
        solutions: [
          'Check if gesture is enabled in Settings',
          'Make sure you\'re not touching UI buttons',
          'Try adjusting Gesture Sensitivity'
        ]
      },
      {
        problem: 'Accidental gesture triggers',
        solutions: [
          'Disable specific gestures in Settings',
          'Lower Gesture Sensitivity',
          'Use button alternatives instead'
        ]
      }
    ]
  }
};
```

---

## Visual Feedback System

### Feedback Specifications

```typescript
const GESTURE_VISUAL_FEEDBACK = {
  doubleTap: {
    firstTap: {
      effect: 'RippleIndicator',
      color: 'rgba(255, 255, 255, 0.3)',
      size: 60, // pixels diameter
      duration: 300, // ms
      animation: 'scale up and fade out'
    },
    success: {
      effect: 'InteractHighlight',
      target: 'interacted object',
      color: 'rgba(255, 215, 0, 0.5)', // Gold
      duration: 400,
      animation: 'pulse once'
    },
    noTarget: {
      effect: 'DimPulse',
      color: 'rgba(128, 128, 128, 0.3)',
      size: 40,
      duration: 200
    }
  },

  pinchMinimap: {
    active: {
      effect: 'BorderHighlight',
      color: 'rgba(100, 200, 255, 0.5)',
      borderWidth: 2
    },
    zoomIndicator: {
      text: 'x{zoom_level}',
      position: 'bottom-right of minimap',
      fadeDelay: 1000 // ms after pinch ends
    }
  },

  threeFingerTap: {
    trigger: {
      effect: 'RadialMenuAppear',
      animation: 'scale from 0.8 to 1.0',
      duration: 200,
      easing: 'easeOutBack'
    }
  },

  edgeSwipe: {
    peek: {
      effect: 'ScoreboardPeek',
      peekWidth: 10, // pixels visible
      shadowOpacity: 0.3
    },
    slideIn: {
      animation: 'translateX from 100% to 0',
      duration: 250,
      easing: 'easeOutCubic'
    }
  }
};
```

---

## Testing Requirements

### Device Compatibility

```typescript
const GESTURE_TESTING_MATRIX = {
  deviceCategories: [
    {
      category: 'Small phones',
      examples: ['iPhone SE', 'iPhone 13 mini'],
      concerns: ['Smaller touch zones', 'Edge swipe accuracy']
    },
    {
      category: 'Standard phones',
      examples: ['iPhone 14', 'Samsung Galaxy S23'],
      concerns: ['Baseline testing']
    },
    {
      category: 'Large phones',
      examples: ['iPhone 14 Pro Max', 'Samsung Galaxy S23 Ultra'],
      concerns: ['Reach for three-finger tap']
    },
    {
      category: 'Tablets',
      examples: ['iPad', 'Samsung Galaxy Tab'],
      concerns: ['Pinch scale different', 'Edge swipe distance']
    },
    {
      category: 'Foldables',
      examples: ['Samsung Galaxy Fold', 'Pixel Fold'],
      concerns: ['Screen fold affects gestures', 'Multiple form factors']
    }
  ],

  testCases: [
    'All gestures work as specified',
    'Gestures don\'t trigger during combat',
    'Gestures don\'t conflict with core controls',
    'Settings toggles work correctly',
    'Visual feedback appears correctly',
    'Haptic feedback triggers appropriately',
    'Performance impact minimal',
    'Works with wet/dry fingers',
    'Works with screen protectors'
  ]
};
```

### Performance Metrics

```typescript
const GESTURE_PERFORMANCE_TARGETS = {
  recognition: {
    latency: '< 16ms', // Single frame
    accuracy: '> 95% intended gesture recognized',
    falsePositive: '< 1% accidental triggers'
  },

  feedback: {
    visualLatency: '< 32ms from gesture to visual',
    hapticLatency: '< 16ms from gesture to haptic'
  },

  cpuOverhead: {
    idle: '< 0.5% CPU',
    activeGesture: '< 2% CPU'
  }
};
```

---

## Analytics Events

```typescript
const GESTURE_ANALYTICS = {
  events: [
    {
      name: 'gesture_performed',
      properties: {
        gesture_type: 'string', // 'double_tap', 'pinch', etc.
        success: 'boolean',
        context: 'string', // 'combat', 'looting', 'menu'
        device_type: 'string'
      }
    },
    {
      name: 'gesture_setting_changed',
      properties: {
        gesture_type: 'string',
        enabled: 'boolean',
        sensitivity: 'number'
      }
    },
    {
      name: 'gesture_conflict_detected',
      properties: {
        gesture_type: 'string',
        conflicting_action: 'string',
        resolution: 'string'
      }
    }
  ],

  dashboardMetrics: [
    'Gesture usage rate per type',
    'Most popular gestures',
    'Gesture-related bug reports',
    'A/B test: gestures vs buttons for looting speed'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial gesture controls specification |
