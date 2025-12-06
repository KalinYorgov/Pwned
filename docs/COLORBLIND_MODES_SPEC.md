# Colorblind Modes Specification

## Document Information
- **Task ID:** UX-018
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies colorblind accessibility options for Plunderstorm Mobile, ensuring players with color vision deficiencies can fully enjoy the game. Approximately 8% of men and 0.5% of women have some form of color vision deficiency.

---

## Color Vision Deficiency Types

### Supported Modes

```typescript
const COLORBLIND_MODES = {
  normal: {
    name: 'Normal Vision',
    description: 'Default color palette',
    prevalence: '~92% of population'
  },

  deuteranopia: {
    name: 'Deuteranopia',
    description: 'Red-green color blindness (green-weak)',
    prevalence: '~6% of males',
    affects: 'Difficulty distinguishing red and green hues',
    severity: 'Most common type'
  },

  protanopia: {
    name: 'Protanopia',
    description: 'Red-green color blindness (red-weak)',
    prevalence: '~2% of males',
    affects: 'Red appears darker, difficulty with red-green',
    severity: 'Common'
  },

  tritanopia: {
    name: 'Tritanopia',
    description: 'Blue-yellow color blindness',
    prevalence: '~0.01% of population',
    affects: 'Difficulty distinguishing blue and yellow',
    severity: 'Rare'
  }
};
```

---

## Color Palette Definitions

### Default Palette (Normal Vision)

```typescript
const DEFAULT_PALETTE = {
  gameplay: {
    friendly: '#4CAF50',      // Green
    enemy: '#F44336',         // Red
    neutral: '#FFEB3B',       // Yellow
    self: '#2196F3',          // Blue
    squad: '#4CAF50',         // Green (same as friendly)
    damage: '#FF5722',        // Orange-red
    healing: '#4CAF50',       // Green
    shield: '#2196F3'         // Blue
  },

  zone: {
    safe: 'transparent',
    warning: '#FFEB3B',       // Yellow
    danger: '#FF5722',        // Orange
    lethal: '#F44336'         // Red
  },

  lootRarity: {
    common: '#9E9E9E',        // Gray
    uncommon: '#4CAF50',      // Green
    rare: '#2196F3',          // Blue
    epic: '#9C27B0',          // Purple
    legendary: '#FF9800'      // Orange/Gold
  },

  abilities: {
    offensive: '#F44336',     // Red
    defensive: '#2196F3',     // Blue
    mobility: '#FFEB3B',      // Yellow
    utility: '#9C27B0'        // Purple
  },

  minimap: {
    player: '#FFFFFF',        // White
    teammate: '#4CAF50',      // Green
    enemy: '#F44336',         // Red (when visible)
    objective: '#FFEB3B',     // Yellow
    zone: '#FF5722'           // Orange
  }
};
```

### Deuteranopia Palette

```typescript
const DEUTERANOPIA_PALETTE = {
  gameplay: {
    friendly: '#0077BB',      // Blue (replaces green)
    enemy: '#EE7733',         // Orange (replaces red)
    neutral: '#CCBB44',       // Yellow-green
    self: '#33BBEE',          // Cyan
    squad: '#0077BB',         // Blue
    damage: '#EE7733',        // Orange
    healing: '#009988',       // Teal
    shield: '#33BBEE'         // Cyan
  },

  zone: {
    safe: 'transparent',
    warning: '#CCBB44',       // Yellow-green
    danger: '#EE7733',        // Orange
    lethal: '#CC3311'         // Dark orange-red
  },

  lootRarity: {
    common: '#BBBBBB',        // Gray
    uncommon: '#009988',      // Teal (replaces green)
    rare: '#33BBEE',          // Cyan (replaces blue)
    epic: '#EE3377',          // Magenta (replaces purple)
    legendary: '#EE7733'      // Orange
  },

  abilities: {
    offensive: '#EE7733',     // Orange
    defensive: '#33BBEE',     // Cyan
    mobility: '#CCBB44',      // Yellow-green
    utility: '#EE3377'        // Magenta
  },

  minimap: {
    player: '#FFFFFF',
    teammate: '#0077BB',      // Blue
    enemy: '#EE7733',         // Orange
    objective: '#CCBB44',
    zone: '#CC3311'
  }
};
```

### Protanopia Palette

```typescript
const PROTANOPIA_PALETTE = {
  gameplay: {
    friendly: '#0077BB',      // Blue
    enemy: '#DDAA33',         // Gold/amber
    neutral: '#CCBB44',       // Yellow-green
    self: '#33BBEE',          // Cyan
    squad: '#0077BB',         // Blue
    damage: '#DDAA33',        // Gold
    healing: '#009988',       // Teal
    shield: '#33BBEE'         // Cyan
  },

  zone: {
    safe: 'transparent',
    warning: '#CCBB44',       // Yellow-green
    danger: '#DDAA33',        // Gold
    lethal: '#AA4499'         // Purple-pink
  },

  lootRarity: {
    common: '#BBBBBB',        // Gray
    uncommon: '#009988',      // Teal
    rare: '#33BBEE',          // Cyan
    epic: '#AA4499',          // Purple-pink
    legendary: '#DDAA33'      // Gold
  },

  abilities: {
    offensive: '#DDAA33',     // Gold
    defensive: '#33BBEE',     // Cyan
    mobility: '#CCBB44',      // Yellow-green
    utility: '#AA4499'        // Purple-pink
  },

  minimap: {
    player: '#FFFFFF',
    teammate: '#0077BB',      // Blue
    enemy: '#DDAA33',         // Gold
    objective: '#CCBB44',
    zone: '#AA4499'
  }
};
```

### Tritanopia Palette

```typescript
const TRITANOPIA_PALETTE = {
  gameplay: {
    friendly: '#33BBEE',      // Cyan (replaces green)
    enemy: '#EE3377',         // Magenta-pink (replaces red)
    neutral: '#DDDDDD',       // Light gray (replaces yellow)
    self: '#0077BB',          // Blue
    squad: '#33BBEE',         // Cyan
    damage: '#EE3377',        // Magenta-pink
    healing: '#33BBEE',       // Cyan
    shield: '#0077BB'         // Blue
  },

  zone: {
    safe: 'transparent',
    warning: '#DDDDDD',       // Light gray
    danger: '#EE7733',        // Orange
    lethal: '#EE3377'         // Magenta-pink
  },

  lootRarity: {
    common: '#888888',        // Gray
    uncommon: '#33BBEE',      // Cyan
    rare: '#0077BB',          // Blue
    epic: '#EE3377',          // Magenta-pink
    legendary: '#EE7733'      // Orange
  },

  abilities: {
    offensive: '#EE3377',     // Magenta-pink
    defensive: '#0077BB',     // Blue
    mobility: '#DDDDDD',      // Light gray
    utility: '#AA4499'        // Purple
  },

  minimap: {
    player: '#FFFFFF',
    teammate: '#33BBEE',      // Cyan
    enemy: '#EE3377',         // Magenta-pink
    objective: '#EE7733',     // Orange
    zone: '#AA4499'
  }
};
```

---

## Implementation Details

### Color Application System

```typescript
const COLOR_APPLICATION_SYSTEM = {
  architecture: {
    approach: 'Shader-based color remapping',
    fallback: 'Texture swapping for complex elements',
    performance: 'Minimal impact (single uniform update)'
  },

  colorRemapping: {
    method: 'LUT (Look-Up Table) based transformation',
    lutSize: '32x32x32 3D texture',
    interpolation: 'Trilinear for smooth transitions'
  },

  affectedElements: [
    {
      element: 'Player outlines',
      method: 'Material color property',
      dynamicUpdate: true
    },
    {
      element: 'Ability VFX',
      method: 'Particle system color modules',
      dynamicUpdate: true
    },
    {
      element: 'UI elements',
      method: 'UI shader with color uniform',
      dynamicUpdate: true
    },
    {
      element: 'Zone visuals',
      method: 'Post-process overlay',
      dynamicUpdate: true
    },
    {
      element: 'Loot highlights',
      method: 'Outline shader color',
      dynamicUpdate: true
    },
    {
      element: 'Minimap icons',
      method: 'Sprite tinting',
      dynamicUpdate: true
    }
  ]
};
```

### Additional Visual Cues

```typescript
const ADDITIONAL_VISUAL_CUES = {
  description: 'Color-independent visual indicators to support colorblind players',

  shapes: {
    friendly: {
      shape: 'circle',
      icon: 'checkmark',
      outline: 'solid'
    },
    enemy: {
      shape: 'diamond',
      icon: 'crosshair',
      outline: 'dashed'
    },
    neutral: {
      shape: 'square',
      icon: 'question',
      outline: 'dotted'
    }
  },

  patterns: {
    enabled: true,
    lootRarity: {
      common: 'solid',
      uncommon: 'single stripe',
      rare: 'double stripe',
      epic: 'crosshatch',
      legendary: 'star pattern'
    }
  },

  icons: {
    abilityTypes: {
      offensive: 'sword icon',
      defensive: 'shield icon',
      mobility: 'boot/arrow icon',
      utility: 'gear icon'
    }
  },

  animations: {
    enemy: 'Subtle pulse animation',
    friendly: 'Gentle glow',
    danger: 'Sharp flashing'
  }
};
```

---

## Settings UI

### Colorblind Settings Panel

```typescript
const COLORBLIND_SETTINGS_UI = {
  location: 'Settings > Accessibility > Colorblind Options',

  layout: {
    header: {
      title: 'Colorblind Options',
      description: 'Adjust colors for better visibility'
    },

    modeSelector: {
      type: 'segmented_control',
      options: [
        { id: 'normal', label: 'Off' },
        { id: 'deuteranopia', label: 'Deuteranopia' },
        { id: 'protanopia', label: 'Protanopia' },
        { id: 'tritanopia', label: 'Tritanopia' }
      ],
      default: 'normal'
    },

    intensitySlider: {
      label: 'Filter Intensity',
      type: 'slider',
      range: [50, 100],
      default: 100,
      unit: '%',
      description: 'Adjust how strongly colors are shifted'
    },

    additionalOptions: [
      {
        id: 'show_patterns',
        label: 'Show Patterns on Loot',
        type: 'toggle',
        default: true,
        description: 'Add patterns to indicate rarity'
      },
      {
        id: 'enhanced_outlines',
        label: 'Enhanced Enemy Outlines',
        type: 'toggle',
        default: true,
        description: 'Thicker outlines around enemies'
      },
      {
        id: 'shape_indicators',
        label: 'Shape-Based Indicators',
        type: 'toggle',
        default: false,
        description: 'Use shapes in addition to colors'
      }
    ]
  }
};
```

### Preview Mode

```typescript
const COLORBLIND_PREVIEW = {
  description: 'Live preview of colorblind settings before applying',

  previewPanel: {
    position: 'right side of settings',
    size: { width: 300, height: 400 },
    content: {
      sceneSample: {
        description: 'Miniature game scene showing:',
        elements: [
          'Player character (self)',
          'Friendly player (green/adjusted)',
          'Enemy player (red/adjusted)',
          'Zone edge',
          'Loot items of each rarity',
          'Minimap preview'
        ]
      },
      abilityIcons: {
        description: 'Grid of ability icons with adjusted colors'
      },
      healthBars: {
        description: 'Health and shield bar samples'
      }
    }
  },

  interactivity: {
    realTimeUpdate: true,
    toggleComparison: {
      button: 'Compare to Normal',
      behavior: 'Hold to see default colors'
    }
  },

  beforeApply: {
    confirmationDialog: false, // Instant apply for accessibility
    undoOption: true,
    undoDuration: 10000 // 10 seconds to undo
  }
};
```

---

## Element-Specific Adjustments

### Enemy vs Teammate Distinction

```typescript
const PLAYER_DISTINCTION = {
  importance: 'Critical - must be instantly distinguishable',

  methods: {
    color: {
      friendly: 'Cool color (blue/cyan/teal)',
      enemy: 'Warm color (orange/gold/magenta)',
      contrast: 'Maximum luminance difference'
    },

    outline: {
      friendly: {
        color: 'Palette friendly color',
        thickness: 2,
        style: 'solid'
      },
      enemy: {
        color: 'Palette enemy color',
        thickness: 3,
        style: 'solid with outer glow'
      }
    },

    nameplate: {
      friendly: {
        background: 'rgba(palette.friendly, 0.8)',
        icon: 'shield icon (optional)'
      },
      enemy: {
        background: 'rgba(palette.enemy, 0.8)',
        icon: 'skull icon (optional)'
      }
    },

    indicator: {
      enabled: 'When shape_indicators setting is on',
      friendly: 'Small circle above head',
      enemy: 'Small diamond above head'
    }
  }
};
```

### Ability Colors

```typescript
const ABILITY_COLOR_ADJUSTMENTS = {
  vfx: {
    approach: 'Shift hue while maintaining value/saturation',
    particleSystems: 'Update start color and color over lifetime',
    trailRenderers: 'Update gradient colors',
    projectiles: 'Material color swap'
  },

  abilitySpecific: {
    fireball: {
      normal: '#FF5722',
      deuteranopia: '#EE7733',
      protanopia: '#DDAA33',
      tritanopia: '#EE3377'
    },
    healingGrog: {
      normal: '#4CAF50',
      deuteranopia: '#009988',
      protanopia: '#009988',
      tritanopia: '#33BBEE'
    },
    barrelRoll: {
      normal: '#FFEB3B',
      deuteranopia: '#CCBB44',
      protanopia: '#CCBB44',
      tritanopia: '#DDDDDD'
    },
    grapple: {
      normal: '#9C27B0',
      deuteranopia: '#EE3377',
      protanopia: '#AA4499',
      tritanopia: '#AA4499'
    }
  },

  telegraphs: {
    friendly: 'Use friendly palette color with 50% alpha',
    enemy: 'Use enemy palette color with 50% alpha',
    edgeHighlight: 'White outline for clarity'
  }
};
```

### Storm/Zone Colors

```typescript
const ZONE_COLOR_ADJUSTMENTS = {
  importance: 'High - survival depends on zone awareness',

  zoneEdge: {
    visual: {
      gradient: true,
      innerColor: 'Adjusted warning color at 30% alpha',
      outerColor: 'Adjusted lethal color at 70% alpha',
      pulseAnimation: true
    },
    additionalCues: {
      soundWarning: 'Always enabled',
      screenEdgeVignette: 'Subtle darkening at screen edges',
      directionalIndicator: 'Arrow pointing to safe zone'
    }
  },

  minimapZone: {
    safeArea: 'White/light outline',
    currentZone: 'Adjusted danger color',
    nextZone: 'Adjusted warning color (dashed)',
    lethalArea: 'Cross-hatched pattern'
  },

  damageIndicator: {
    color: 'Adjusted damage color',
    screenEffect: 'Vignette + heartbeat pulse',
    iconOverlay: 'Storm icon in corner'
  }
};
```

### Loot Rarity Colors

```typescript
const LOOT_RARITY_ADJUSTMENTS = {
  importance: 'Medium - quick identification of value',

  groundLoot: {
    beam: {
      color: 'Adjusted rarity color',
      height: 'Varies by rarity (taller = rarer)',
      animation: 'Pulse speed varies (faster = rarer)'
    },
    outline: {
      color: 'Adjusted rarity color',
      thickness: 'Varies by rarity',
      pattern: 'When patterns enabled'
    }
  },

  inventoryUI: {
    background: 'Adjusted rarity color at 20% alpha',
    border: 'Adjusted rarity color',
    cornerIcon: {
      common: 'No icon',
      uncommon: 'Single dot',
      rare: 'Two dots',
      epic: 'Three dots',
      legendary: 'Star'
    }
  },

  chestGlow: {
    color: 'Adjusted to highest rarity inside',
    intensity: 'Brighter for rarer contents'
  }
};
```

### Minimap Colors

```typescript
const MINIMAP_COLOR_ADJUSTMENTS = {
  importance: 'High - tactical awareness',

  icons: {
    self: {
      color: '#FFFFFF', // Always white
      shape: 'Arrow/chevron',
      size: 'Largest'
    },
    teammates: {
      color: 'Adjusted friendly color',
      shape: 'Circle',
      size: 'Medium',
      showName: true
    },
    enemies: {
      color: 'Adjusted enemy color',
      shape: 'Diamond',
      size: 'Medium',
      showWhenRevealed: true
    },
    objectives: {
      color: 'Adjusted neutral color',
      shape: 'Square with icon',
      pulseAnimation: true
    }
  },

  zones: {
    safe: {
      color: 'White outline',
      fill: 'None'
    },
    danger: {
      color: 'Adjusted danger color',
      fill: '30% alpha',
      pattern: 'Diagonal lines when in colorblind mode'
    }
  }
};
```

---

## Testing Requirements

### Simulation Testing

```typescript
const COLORBLIND_TESTING = {
  simulationTools: [
    {
      name: 'Color Oracle',
      platform: 'Desktop',
      use: 'Screenshot analysis'
    },
    {
      name: 'Sim Daltonism',
      platform: 'macOS',
      use: 'Real-time screen filter'
    },
    {
      name: 'Color Blindness Simulate',
      platform: 'Android',
      use: 'Device-level simulation'
    },
    {
      name: 'Built-in iOS Accessibility',
      platform: 'iOS',
      use: 'Device-level color filters'
    }
  ],

  testScenarios: [
    {
      scenario: 'Combat clarity',
      test: 'Can easily distinguish enemies from teammates',
      importance: 'Critical'
    },
    {
      scenario: 'Zone awareness',
      test: 'Can see zone edge and understand danger',
      importance: 'Critical'
    },
    {
      scenario: 'Loot identification',
      test: 'Can quickly identify loot rarity',
      importance: 'High'
    },
    {
      scenario: 'Ability recognition',
      test: 'Can identify ability types by color/shape',
      importance: 'High'
    },
    {
      scenario: 'Minimap readability',
      test: 'Can distinguish all minimap elements',
      importance: 'High'
    },
    {
      scenario: 'UI element visibility',
      test: 'All UI elements are visible and readable',
      importance: 'Medium'
    }
  ]
};
```

### User Testing

```typescript
const USER_TESTING_PLAN = {
  recruitment: {
    target: '5-10 colorblind players per mode',
    sources: [
      'Gaming accessibility communities',
      'Reddit (r/colorblind)',
      'Accessibility testing services'
    ]
  },

  testProtocol: {
    duration: '30-60 minutes',
    tasks: [
      'Complete tutorial',
      'Play 3 matches',
      'Identify loot rarities',
      'Navigate using minimap',
      'Engage in combat'
    ],
    feedback: [
      'Can you see everything you need?',
      'Any confusion about colors?',
      'Rate ease of play 1-10',
      'Suggestions for improvement'
    ]
  },

  successCriteria: {
    combatClarity: '>= 90% can distinguish friend/foe instantly',
    zoneAwareness: '>= 95% can see zone boundaries',
    lootIdentification: '>= 80% accuracy on rarity identification',
    overallSatisfaction: '>= 4/5 rating'
  }
};
```

---

## Performance Considerations

```typescript
const COLORBLIND_PERFORMANCE = {
  overhead: {
    cpu: '< 0.5% additional',
    gpu: '< 1% additional (shader uniforms)',
    memory: '~256KB for LUT textures'
  },

  optimization: {
    lutCaching: 'Load once, reuse for session',
    shaderVariants: 'Single shader with uniform swap',
    uiRebuild: 'Only on setting change'
  },

  modeSwitch: {
    transitionTime: '< 100ms',
    noRestart: true,
    instantApply: true
  }
};
```

---

## Analytics

```typescript
const COLORBLIND_ANALYTICS = {
  events: [
    {
      name: 'colorblind_mode_enabled',
      properties: {
        mode: 'deuteranopia | protanopia | tritanopia',
        intensity: 'number (50-100)',
        additional_options: 'array of enabled options'
      }
    },
    {
      name: 'colorblind_mode_changed',
      properties: {
        from_mode: 'string',
        to_mode: 'string',
        time_used_previous: 'seconds'
      }
    },
    {
      name: 'colorblind_mode_disabled',
      properties: {
        mode_was: 'string',
        time_used: 'seconds'
      }
    }
  ],

  metrics: [
    'Percentage of players using colorblind modes',
    'Most popular colorblind mode',
    'Retention comparison (colorblind vs normal)',
    'Session length comparison'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial colorblind modes specification |
