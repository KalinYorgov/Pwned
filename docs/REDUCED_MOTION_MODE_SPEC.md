# Reduced Motion Mode Specification

## Document Information
- **Task ID:** UX-021
- **Priority:** P3
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Reduced Motion Mode for Plunderstorm Mobile, providing accessibility options for players sensitive to motion, flashing lights, or excessive visual effects while maintaining full gameplay functionality.

---

## Design Philosophy

### Accessibility Goals

```typescript
const ACCESSIBILITY_GOALS = {
  comfort: {
    description: 'Allow motion-sensitive players to play comfortably',
    conditions: ['Vestibular disorders', 'Motion sickness', 'Photosensitive epilepsy'],
    approach: 'Reduce or eliminate triggering visual elements'
  },

  functionality: {
    description: 'Maintain full gameplay capability',
    principle: 'No competitive disadvantage',
    requirement: 'All information still communicated clearly'
  },

  choice: {
    description: 'Granular control over effects',
    options: 'Individual toggles for different effect types',
    presets: 'Quick presets for common needs'
  },

  systemIntegration: {
    ios: 'Respect system "Reduce Motion" setting',
    android: 'Respect system "Remove animations" setting',
    override: 'Allow in-game override of system setting'
  }
};
```

---

## Motion Reduction Features

### Screen Shake

```typescript
const SCREEN_SHAKE = {
  default: {
    triggers: [
      'Taking damage',
      'Explosions nearby',
      'Ability impacts',
      'Landing from height',
      'Storm damage'
    ],
    intensity: 'Variable based on event'
  },

  reducedMotion: {
    setting: 'Disable Screen Shake',
    behavior: 'All screen shake eliminated',
    alternative: 'Visual flash or vignette instead',
    feedback: 'Other feedback channels maintained (haptic, audio)'
  },

  options: {
    off: 'No screen shake (reduced motion)',
    low: '25% intensity',
    medium: '50% intensity (default)',
    high: '100% intensity'
  }
};
```

### Camera Effects

```typescript
const CAMERA_EFFECTS = {
  cameraBob: {
    description: 'Camera movement while walking/running',
    default: 'Subtle bob synced with movement',
    reducedMotion: {
      setting: 'Disable Camera Bob',
      behavior: 'Camera remains steady during movement'
    }
  },

  cameraSmoothing: {
    description: 'Smooth camera transitions',
    default: 'Eased transitions between states',
    reducedMotion: {
      setting: 'Instant Camera Transitions',
      behavior: 'Immediate cuts instead of smooth pans'
    }
  },

  fovChanges: {
    description: 'FOV shifts during sprint/abilities',
    default: 'Dynamic FOV based on speed',
    reducedMotion: {
      setting: 'Static FOV',
      behavior: 'Constant FOV regardless of action'
    }
  },

  motionBlur: {
    description: 'Blur effect during fast movement',
    default: 'Subtle blur on quick turns',
    reducedMotion: {
      setting: 'Disable Motion Blur',
      behavior: 'No blur effects applied'
    }
  }
};
```

### Visual Effects (VFX)

```typescript
const VFX_REDUCTION = {
  particleEffects: {
    description: 'Particle systems for abilities, impacts, environment',
    default: 'Full particle density and movement',
    reducedMotion: {
      setting: 'Reduced Particle Effects',
      behavior: [
        'Reduce particle count by 50%',
        'Slower particle movement',
        'Shorter particle lifetimes',
        'Static particles where possible'
      ]
    }
  },

  abilityEffects: {
    description: 'Visual effects for abilities',
    default: 'Full animated effects',
    reducedMotion: {
      setting: 'Simplified Ability Effects',
      behavior: [
        'Reduced animation complexity',
        'Solid colors instead of gradients',
        'Fewer secondary effects',
        'Core indicator always visible'
      ]
    }
  },

  environmentalEffects: {
    description: 'Weather, ambient particles, water',
    default: 'Full environmental animation',
    reducedMotion: {
      setting: 'Reduced Environmental Effects',
      behavior: [
        'Static or slow-moving clouds',
        'Reduced rain/snow density',
        'Calmer water surfaces',
        'Fewer ambient particles'
      ]
    }
  }
};
```

---

## Flashing and Strobing

### Flash Reduction

```typescript
const FLASH_REDUCTION = {
  triggers: {
    explosions: 'Bright flash on explosions',
    abilities: 'Flash effects on ability cast/impact',
    lightning: 'Weather lightning flashes',
    muzzleFlash: 'Weapon/ability muzzle effects',
    damage: 'Screen flash when taking damage',
    pickups: 'Item pickup flash'
  },

  reducedMotion: {
    setting: 'Reduce Flashing Effects',
    behavior: {
      intensity: 'Reduce flash brightness by 70%',
      duration: 'Extend flash duration (slower fade)',
      frequency: 'Limit to max 2 flashes per second',
      alternative: 'Use color tint instead of bright flash'
    }
  },

  photosensitivity: {
    setting: 'Photosensitive Mode',
    behavior: {
      elimination: 'Remove all rapid flashing',
      strobing: 'No strobing effects',
      threshold: 'Never exceed 3Hz flash rate',
      warning: 'Pre-game warning for unavoidable effects'
    }
  }
};
```

### Seizure Safety

```typescript
const SEIZURE_SAFETY = {
  compliance: {
    standard: 'Follow WCAG 2.1 guidelines',
    threshold: 'No more than 3 flashes per second',
    area: 'Flashing covers less than 25% of screen'
  },

  implementation: {
    analysis: 'All effects tested for flash rate',
    flagging: 'High-risk effects flagged in editor',
    override: 'Photosensitive mode removes all flagged effects'
  }
};
```

---

## UI Animation Reduction

### Menu Animations

```typescript
const UI_ANIMATIONS = {
  transitions: {
    default: 'Animated screen transitions (slide, fade)',
    reducedMotion: {
      setting: 'Instant UI Transitions',
      behavior: 'Immediate cuts between screens'
    }
  },

  buttonEffects: {
    default: 'Bounce, pulse on interaction',
    reducedMotion: {
      setting: 'Static UI Elements',
      behavior: 'Simple highlight, no animation'
    }
  },

  notifications: {
    default: 'Slide-in, bounce notifications',
    reducedMotion: {
      behavior: 'Fade-in only, no movement'
    }
  },

  loadingIndicators: {
    default: 'Spinning, animated loaders',
    reducedMotion: {
      behavior: 'Static progress bar or simple pulse'
    }
  },

  scrolling: {
    default: 'Momentum scrolling with bounce',
    reducedMotion: {
      behavior: 'Direct scrolling, no overscroll bounce'
    }
  }
};
```

### HUD Elements

```typescript
const HUD_ANIMATIONS = {
  healthBar: {
    default: 'Animated fill/drain, shake on damage',
    reducedMotion: 'Instant fill changes, no shake'
  },

  abilityIcons: {
    default: 'Cooldown spin, ready pulse',
    reducedMotion: 'Static cooldown overlay, color change only'
  },

  minimap: {
    default: 'Animated pings, smooth player movement',
    reducedMotion: 'Static pings, direct position updates'
  },

  killFeed: {
    default: 'Slide-in animation',
    reducedMotion: 'Instant appear/disappear'
  },

  damageNumbers: {
    default: 'Float up with fade',
    reducedMotion: 'Static display, quick fade'
  }
};
```

---

## Content Filters

### Blood and Gore

```typescript
const BLOOD_GORE = {
  setting: 'Disable Blood Effects',

  default: {
    hitEffects: 'Blood splatter on damage',
    elimination: 'Death effects',
    environment: 'Blood decals on surfaces'
  },

  disabled: {
    hitEffects: 'Replaced with sparks/impact dust',
    elimination: 'Clean fade-out animation',
    environment: 'No blood decals',
    color: 'Red effects changed to neutral'
  },

  options: {
    full: 'All blood effects enabled',
    reduced: 'Minimal blood, no gore',
    none: 'No blood or gore effects'
  }
};
```

---

## Settings Interface

### Reduced Motion Settings

```typescript
const SETTINGS_UI = {
  location: 'Settings > Accessibility > Motion & Effects',

  quickToggle: {
    name: 'Reduced Motion Mode',
    description: 'Apply all motion reduction settings',
    behavior: 'Single toggle for all reductions',
    default: 'Follows system setting'
  },

  individualSettings: {
    screenShake: {
      name: 'Screen Shake',
      type: 'slider',
      options: ['Off', 'Low', 'Medium', 'High'],
      default: 'Medium'
    },
    cameraBob: {
      name: 'Camera Bob',
      type: 'toggle',
      default: true
    },
    motionBlur: {
      name: 'Motion Blur',
      type: 'toggle',
      default: true
    },
    particleEffects: {
      name: 'Particle Density',
      type: 'slider',
      options: ['Minimal', 'Reduced', 'Normal', 'High'],
      default: 'Normal'
    },
    flashingEffects: {
      name: 'Flashing Effects',
      type: 'slider',
      options: ['Off', 'Reduced', 'Normal'],
      default: 'Normal'
    },
    uiAnimations: {
      name: 'UI Animations',
      type: 'toggle',
      default: true
    },
    bloodEffects: {
      name: 'Blood Effects',
      type: 'slider',
      options: ['Off', 'Reduced', 'Full'],
      default: 'Full'
    }
  },

  presets: {
    comfortable: {
      name: 'Comfortable',
      description: 'Moderate reduction for mild sensitivity',
      settings: {
        screenShake: 'Low',
        cameraBob: true,
        motionBlur: false,
        particleEffects: 'Reduced',
        flashingEffects: 'Reduced',
        uiAnimations: true
      }
    },
    minimal: {
      name: 'Minimal Motion',
      description: 'Maximum reduction for high sensitivity',
      settings: {
        screenShake: 'Off',
        cameraBob: false,
        motionBlur: false,
        particleEffects: 'Minimal',
        flashingEffects: 'Off',
        uiAnimations: false
      }
    },
    photosensitive: {
      name: 'Photosensitive',
      description: 'Optimized for light sensitivity',
      settings: {
        screenShake: 'Low',
        flashingEffects: 'Off',
        particleEffects: 'Reduced'
      }
    }
  }
};
```

### System Integration

```typescript
const SYSTEM_INTEGRATION = {
  ios: {
    setting: 'UIAccessibility.isReduceMotionEnabled',
    behavior: 'Auto-apply reduced motion on first launch',
    override: 'Player can override in-game'
  },

  android: {
    setting: 'Settings.Global.ANIMATOR_DURATION_SCALE',
    behavior: 'Detect if animations disabled',
    override: 'Player can override in-game'
  },

  prompt: {
    trigger: 'System reduced motion detected',
    message: 'We noticed you have reduced motion enabled. Would you like to apply comfortable settings?',
    options: ['Apply Comfortable Settings', 'Keep Default', 'Customize']
  }
};
```

---

## Implementation Details

### Effect Replacement

```typescript
const EFFECT_REPLACEMENT = {
  screenShake: {
    original: 'Camera position offset over time',
    replacement: 'Screen edge vignette pulse'
  },

  flash: {
    original: 'Bright white flash',
    replacement: 'Subtle color tint overlay'
  },

  explosion: {
    original: 'Expanding particle burst',
    replacement: 'Static expanding ring + dust'
  },

  damage: {
    original: 'Red flash + shake',
    replacement: 'Red vignette + haptic'
  },

  ability: {
    original: 'Complex animated effect',
    replacement: 'Simplified core indicator'
  }
};
```

### Performance Considerations

```typescript
const PERFORMANCE = {
  benefit: {
    description: 'Reduced motion often improves performance',
    particleReduction: 'Less GPU particle processing',
    animationReduction: 'Less CPU animation updates'
  },

  lowEndDevices: {
    recommendation: 'Suggest reduced motion for struggling devices',
    automatic: 'Auto-enable on very low-end devices'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'All motion reduction settings work correctly',
    'System setting detection works',
    'Presets apply correct combinations',
    'No visual glitches with effects disabled'
  ],

  accessibility: [
    'Test with motion-sensitive users',
    'Verify flash rates comply with guidelines',
    'Ensure gameplay information preserved',
    'Check all alternatives provide adequate feedback'
  ],

  combinations: [
    'All individual settings combinable',
    'No conflicts between settings',
    'Settings persist across sessions'
  ],

  gameplay: [
    'No competitive disadvantage',
    'All abilities clearly visible',
    'Damage/healing clearly communicated',
    'Enemy positions visible'
  ],

  platforms: [
    'iOS system integration',
    'Android system integration',
    'Various device performance levels'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial reduced motion mode specification |
