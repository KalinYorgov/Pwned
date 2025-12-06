# Text and UI Scaling Specification

## Document Information
- **Task ID:** UX-019
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies the text and UI scaling system for Plunderstorm Mobile, enabling players to customize the interface for optimal readability and usability across different device sizes, visual preferences, and accessibility needs.

---

## Scaling System Architecture

### Scale Categories

```typescript
const SCALE_CATEGORIES = {
  hudScale: {
    description: 'In-game HUD elements during gameplay',
    affects: [
      'Health/shield bars',
      'Ability buttons',
      'Minimap',
      'Kill feed',
      'Player indicators',
      'Ammo/item counts'
    ],
    range: { min: 75, max: 150, default: 100 },
    unit: '%'
  },

  menuTextScale: {
    description: 'Text in menus and UI screens',
    affects: [
      'Menu titles and labels',
      'Button text',
      'Description text',
      'Player names',
      'Chat messages'
    ],
    range: { min: 80, max: 140, default: 100 },
    unit: '%'
  },

  buttonScale: {
    description: 'Interactive button and touch target sizes',
    affects: [
      'Menu buttons',
      'Action buttons',
      'Navigation elements',
      'Toggles and sliders'
    ],
    range: { min: 90, max: 130, default: 100 },
    linkToHud: true, // Can be linked or independent
    unit: '%'
  }
};
```

### Base Size Definitions

```typescript
const BASE_SIZES = {
  text: {
    tiny: 10,       // Labels, timestamps
    small: 12,      // Secondary info
    body: 14,       // Default body text
    subtitle: 16,   // Subtitles, important info
    title: 20,      // Section titles
    header: 24,     // Screen headers
    display: 32,    // Large display text
    unit: 'sp'      // Scale-independent pixels
  },

  buttons: {
    small: 36,      // Icon-only buttons
    medium: 44,     // Standard buttons (minimum touch target)
    large: 56,      // Primary action buttons
    extraLarge: 72, // Ability buttons in gameplay
    unit: 'dp'      // Density-independent pixels
  },

  hudElements: {
    minimap: 150,   // Minimap diameter
    healthBar: { width: 200, height: 16 },
    abilityButton: 72,
    itemSlot: 56,
    unit: 'dp'
  },

  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32
  }
};
```

---

## HUD Scale System

### Scalable HUD Elements

```typescript
const HUD_SCALING = {
  elements: {
    healthShieldBar: {
      baseSize: { width: 200, height: 16 },
      scaleRange: { min: 0.75, max: 1.5 },
      anchorPoint: 'top-left',
      maintainAspectRatio: true
    },

    abilityButtons: {
      baseSize: 72,
      scaleRange: { min: 0.75, max: 1.5 },
      layout: 'Respects spacing between buttons',
      touchTargetMinimum: 44 // Never scale below this
    },

    minimap: {
      baseSize: 150,
      scaleRange: { min: 0.75, max: 1.5 },
      anchorPoint: 'top-right',
      contentScales: true // Icons inside also scale
    },

    killFeed: {
      baseSize: { width: 250, entryHeight: 24 },
      scaleRange: { min: 0.8, max: 1.3 },
      anchorPoint: 'top-right',
      maxVisibleEntries: 'Adjusts based on scale'
    },

    playerIndicators: {
      baseSize: 48,
      scaleRange: { min: 0.75, max: 1.5 },
      includesNameplate: true
    },

    ammoCounter: {
      baseSize: { width: 80, height: 40 },
      scaleRange: { min: 0.75, max: 1.5 },
      anchorPoint: 'bottom-right'
    },

    itemSlots: {
      baseSize: 56,
      scaleRange: { min: 0.75, max: 1.5 },
      spacing: 8
    }
  },

  groupScaling: {
    description: 'Elements scale together to maintain visual harmony',
    exception: 'Touch targets never go below 44dp'
  }
};
```

### HUD Layout Adaptation

```typescript
const HUD_LAYOUT_ADAPTATION = {
  smallScale: {
    threshold: '<= 85%',
    adaptations: [
      'Condense spacing between elements',
      'Stack overlapping elements vertically',
      'Hide secondary information (tap to reveal)',
      'Use more compact number formatting'
    ]
  },

  largeScale: {
    threshold: '>= 130%',
    adaptations: [
      'Increase spacing to prevent overlap',
      'Adjust anchor points if near screen edge',
      'Wrap text if exceeding bounds',
      'Consider safe area more carefully'
    ]
  },

  overlapPrevention: {
    enabled: true,
    method: 'Automatic repositioning if overlap detected',
    priority: ['Ability buttons', 'Health bar', 'Minimap', 'Kill feed']
  }
};
```

---

## Text Scaling System

### Text Size Calculation

```typescript
const TEXT_SIZE_CALCULATION = {
  formula: 'finalSize = baseSize * (userScale / 100) * systemScale',

  systemScale: {
    description: 'Respect OS-level text size settings',
    iOS: 'Dynamic Type',
    android: 'Font Scale',
    range: { min: 0.85, max: 1.35 },
    respectSystem: true // Default on
  },

  userScale: {
    description: 'In-game text scale slider',
    range: { min: 80, max: 140 },
    default: 100
  },

  minimumReadable: {
    description: 'Never scale below readable threshold',
    value: 11,
    unit: 'sp'
  },

  maximumSize: {
    description: 'Cap to prevent UI breaking',
    titleMax: 40,
    bodyMax: 24,
    unit: 'sp'
  }
};
```

### Text Categories and Scaling

```typescript
const TEXT_CATEGORIES = {
  critical: {
    description: 'Must always be readable',
    examples: ['Health numbers', 'Damage numbers', 'Timer', 'Player count'],
    minimumScale: 90,
    boldOption: true
  },

  important: {
    description: 'Key information',
    examples: ['Player names', 'Ability names', 'Menu options'],
    minimumScale: 85,
    boldOption: true
  },

  secondary: {
    description: 'Supporting information',
    examples: ['Descriptions', 'Tooltips', 'Chat messages'],
    minimumScale: 80,
    boldOption: true
  },

  decorative: {
    description: 'Non-essential text',
    examples: ['Flavor text', 'Version numbers', 'Credits'],
    minimumScale: 75,
    boldOption: false
  }
};
```

---

## Button and Touch Target Scaling

### Touch Target Requirements

```typescript
const TOUCH_TARGET_REQUIREMENTS = {
  minimumSize: {
    value: 44,
    unit: 'dp',
    source: 'Apple HIG / Material Design guidelines',
    enforced: true // Never scale below this
  },

  recommendedSize: {
    value: 48,
    unit: 'dp',
    description: 'Comfortable touch target'
  },

  spacing: {
    minimum: 8,
    recommended: 12,
    unit: 'dp',
    description: 'Space between adjacent touch targets'
  }
};
```

### Button Scaling Behavior

```typescript
const BUTTON_SCALING = {
  linkToHudScale: {
    default: true,
    description: 'Button scale follows HUD scale by default',
    canOverride: true
  },

  independentScaling: {
    enabled: 'When user unchecks "Link to HUD Scale"',
    range: { min: 90, max: 130 },
    default: 100
  },

  scaleComponents: {
    buttonBackground: 'Scales uniformly',
    buttonText: 'Scales with button, respects text minimum',
    buttonIcon: 'Scales with button',
    buttonPadding: 'Scales proportionally'
  },

  adaptiveLayout: {
    description: 'Buttons reflow if needed at larger scales',
    menuButtons: 'Stack vertically if horizontal space insufficient',
    gameplayButtons: 'Maintain relative positions, adjust spacing'
  }
};
```

---

## High Contrast Mode

### Contrast Enhancement

```typescript
const HIGH_CONTRAST_MODE = {
  description: 'Enhance visual contrast for better readability',

  textEnhancements: {
    textColor: '#FFFFFF', // Pure white
    backgroundColor: 'rgba(0, 0, 0, 0.9)', // Near-black
    outlineEnabled: true,
    outlineColor: '#000000',
    outlineWidth: 2
  },

  buttonEnhancements: {
    borderWidth: 3,
    borderColor: '#FFFFFF',
    backgroundColor: 'rgba(0, 0, 0, 0.95)',
    hoverState: {
      backgroundColor: '#FFFFFF',
      textColor: '#000000'
    }
  },

  hudEnhancements: {
    healthBar: {
      background: '#000000',
      border: '#FFFFFF',
      healthColor: '#00FF00',
      shieldColor: '#00FFFF'
    },
    minimap: {
      background: 'rgba(0, 0, 0, 0.9)',
      border: '#FFFFFF',
      iconContrast: 'enhanced'
    }
  },

  colorAdjustments: {
    saturation: 'Increased by 20%',
    brightness: 'Text brightness +30%',
    shadows: 'Stronger drop shadows on text'
  }
};
```

---

## Bold Text Option

### Bold Text Implementation

```typescript
const BOLD_TEXT_OPTION = {
  description: 'Make all text bolder for improved readability',

  implementation: {
    method: 'Font weight adjustment',
    normalWeight: 400,
    boldWeight: 700,
    extraBoldWeight: 800 // For already bold text
  },

  affectedElements: [
    'All menu text',
    'HUD text elements',
    'Player names',
    'Button labels',
    'Tooltips and descriptions',
    'Chat messages'
  ],

  excludedElements: [
    'Decorative text (maintains original weight)',
    'Icons and symbols'
  ],

  antiAliasing: {
    enhanced: true,
    description: 'Improve text rendering when bold is enabled'
  }
};
```

---

## Settings UI

### Scaling Settings Panel

```typescript
const SCALING_SETTINGS_UI = {
  location: 'Settings > Accessibility > Display & Text',

  sections: {
    hudScaling: {
      title: 'HUD Size',
      controls: [
        {
          type: 'slider',
          id: 'hud_scale',
          label: 'HUD Scale',
          range: [75, 150],
          default: 100,
          step: 5,
          unit: '%',
          showValue: true
        },
        {
          type: 'button',
          id: 'reset_hud',
          label: 'Reset to Default',
          action: 'resetHudScale()'
        }
      ]
    },

    textScaling: {
      title: 'Text Size',
      controls: [
        {
          type: 'slider',
          id: 'text_scale',
          label: 'Menu Text Size',
          range: [80, 140],
          default: 100,
          step: 5,
          unit: '%',
          showValue: true
        },
        {
          type: 'toggle',
          id: 'respect_system_text',
          label: 'Use System Text Size',
          default: true,
          description: 'Apply device accessibility text settings'
        }
      ]
    },

    buttonScaling: {
      title: 'Button Size',
      controls: [
        {
          type: 'toggle',
          id: 'link_button_to_hud',
          label: 'Link to HUD Scale',
          default: true
        },
        {
          type: 'slider',
          id: 'button_scale',
          label: 'Button Scale',
          range: [90, 130],
          default: 100,
          step: 5,
          unit: '%',
          showValue: true,
          showWhen: '!link_button_to_hud'
        }
      ]
    },

    visualEnhancements: {
      title: 'Visual Enhancements',
      controls: [
        {
          type: 'toggle',
          id: 'high_contrast',
          label: 'High Contrast Mode',
          default: false,
          description: 'Increase contrast for better visibility'
        },
        {
          type: 'toggle',
          id: 'bold_text',
          label: 'Bold Text',
          default: false,
          description: 'Make all text bolder'
        }
      ]
    }
  }
};
```

### Preview System

```typescript
const SCALING_PREVIEW = {
  description: 'Live preview of scaling changes',

  previewPanel: {
    position: 'Right side of settings (tablet) or below controls (phone)',
    content: {
      hudPreview: {
        description: 'Miniature HUD mockup',
        elements: ['Health bar', 'Ability buttons', 'Minimap corner']
      },
      textPreview: {
        description: 'Sample text at different sizes',
        samples: [
          { label: 'Header Text', size: 'title' },
          { label: 'Body text sample showing readability', size: 'body' },
          { label: 'Small text example', size: 'small' }
        ]
      },
      buttonPreview: {
        description: 'Sample buttons',
        samples: ['Primary Button', 'Secondary']
      }
    }
  },

  liveUpdate: {
    enabled: true,
    delay: 100, // ms debounce
    smoothTransition: true
  },

  beforeAfterToggle: {
    enabled: true,
    button: 'Compare',
    behavior: 'Hold to see default settings'
  }
};
```

---

## Device-Specific Adaptations

### Screen Size Categories

```typescript
const DEVICE_ADAPTATIONS = {
  smallPhone: {
    screenWidth: '< 375dp',
    examples: ['iPhone SE', 'Older Android phones'],
    defaultScales: {
      hud: 90,
      text: 95,
      button: 100
    },
    recommendations: [
      'Suggest slightly smaller HUD',
      'Ensure touch targets remain adequate'
    ]
  },

  standardPhone: {
    screenWidth: '375-428dp',
    examples: ['iPhone 14', 'Samsung Galaxy S23'],
    defaultScales: {
      hud: 100,
      text: 100,
      button: 100
    },
    recommendations: []
  },

  largePhone: {
    screenWidth: '> 428dp',
    examples: ['iPhone 14 Pro Max', 'Galaxy S Ultra'],
    defaultScales: {
      hud: 105,
      text: 100,
      button: 100
    },
    recommendations: [
      'Can accommodate larger HUD comfortably'
    ]
  },

  tablet: {
    screenWidth: '> 600dp',
    examples: ['iPad', 'Samsung Tab'],
    defaultScales: {
      hud: 110,
      text: 110,
      button: 110
    },
    recommendations: [
      'Larger defaults for tablet viewing distance',
      'More screen real estate for HUD'
    ]
  }
};
```

### Orientation Handling

```typescript
const ORIENTATION_HANDLING = {
  landscape: {
    hudPositions: 'Standard layout',
    safeAreas: 'Account for notch/camera cutout'
  },

  portrait: {
    supported: false, // Game is landscape-only
    lockMessage: 'Please rotate device to landscape mode'
  },

  scaleConsistency: {
    description: 'Scale settings persist across orientations',
    note: 'Layout may adapt but relative sizes remain'
  }
};
```

---

## Overlap Prevention System

### Collision Detection

```typescript
const OVERLAP_PREVENTION = {
  detection: {
    method: 'Bounding box collision check',
    frequency: 'On scale change and screen resize',
    elements: 'All HUD elements'
  },

  resolution: {
    priority: [
      'Ability buttons (highest - gameplay critical)',
      'Health bar',
      'Minimap',
      'Kill feed',
      'Item slots',
      'Decorative elements (lowest)'
    ],

    strategies: [
      {
        name: 'Spacing adjustment',
        description: 'Increase space between overlapping elements',
        maxAdjustment: 20 // pixels
      },
      {
        name: 'Repositioning',
        description: 'Move lower priority element to alternate position',
        fallbackPositions: 'Predefined per element'
      },
      {
        name: 'Scale reduction',
        description: 'Slightly reduce scale of lower priority element',
        maxReduction: 10 // percent
      },
      {
        name: 'Stacking',
        description: 'Stack elements vertically if horizontal overlap',
        applies: 'Menu buttons primarily'
      }
    ]
  },

  userWarning: {
    showWhen: 'Scale causes unavoidable overlap',
    message: 'Current scale may cause UI overlap. Consider adjusting.',
    dismissable: true
  }
};
```

---

## Performance Considerations

```typescript
const SCALING_PERFORMANCE = {
  rendering: {
    method: 'Transform scale on UI canvas',
    textRendering: 'Pre-rasterize at target sizes',
    caching: 'Cache scaled sprites for common sizes'
  },

  memory: {
    fontAtlas: 'Generate at maximum supported size',
    dynamicRegeneration: false,
    estimatedOverhead: '~5MB for text atlases'
  },

  frameImpact: {
    staticUI: 'No impact (pre-rendered)',
    dynamicUI: '< 0.1ms additional per frame',
    scaleChange: '~50ms to apply new scale'
  },

  batchOptimization: {
    description: 'Group same-scale elements for batch rendering',
    benefitS: 'Reduced draw calls'
  }
};
```

---

## Accessibility Integration

### System Accessibility Respect

```typescript
const SYSTEM_ACCESSIBILITY_INTEGRATION = {
  iOS: {
    dynamicType: {
      respect: true,
      mapping: {
        xSmall: 0.85,
        small: 0.9,
        medium: 0.95,
        large: 1.0, // Default
        xLarge: 1.1,
        xxLarge: 1.2,
        xxxLarge: 1.35
      }
    },
    boldText: {
      respect: true,
      autoEnable: 'When system bold text is on'
    },
    reduceMotion: {
      respect: true,
      affectsScaleTransitions: 'Instant instead of animated'
    }
  },

  android: {
    fontScale: {
      respect: true,
      range: { min: 0.85, max: 1.3 }
    },
    displaySize: {
      respect: true,
      affectsHudScale: true
    },
    highContrast: {
      respect: true,
      autoEnable: 'When system high contrast is on'
    }
  },

  override: {
    setting: 'Use System Text Size toggle',
    whenOff: 'Ignore system settings, use game settings only'
  }
};
```

---

## Analytics

```typescript
const SCALING_ANALYTICS = {
  events: [
    {
      name: 'scaling_setting_changed',
      properties: {
        setting_type: 'hud | text | button',
        old_value: 'number',
        new_value: 'number',
        device_category: 'small_phone | standard | large | tablet'
      }
    },
    {
      name: 'high_contrast_toggled',
      properties: {
        enabled: 'boolean',
        session_time_before: 'seconds'
      }
    },
    {
      name: 'bold_text_toggled',
      properties: {
        enabled: 'boolean'
      }
    },
    {
      name: 'system_accessibility_detected',
      properties: {
        dynamic_type_category: 'string',
        system_bold_text: 'boolean',
        font_scale: 'number'
      }
    }
  ],

  metrics: [
    'Average HUD scale by device category',
    'Percentage using non-default text scale',
    'High contrast adoption rate',
    'Bold text adoption rate',
    'System accessibility detection rate'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial text and UI scaling specification |
