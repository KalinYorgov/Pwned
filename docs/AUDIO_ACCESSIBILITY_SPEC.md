# Audio Accessibility Specification

## Document Information
- **Task ID:** UX-020
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies audio accessibility features for Plunderstorm Mobile, ensuring players who are deaf, hard of hearing, or playing without sound can fully enjoy and compete in the game. All critical audio information must have visual or haptic alternatives.

---

## Subtitle System

### Subtitle Configuration

```typescript
const SUBTITLE_SYSTEM = {
  types: {
    voiceover: {
      description: 'Tutorial narration, announcer voice',
      priority: 'High',
      defaultEnabled: true
    },
    characterVoice: {
      description: 'Player character callouts, emotes',
      priority: 'Medium',
      defaultEnabled: true
    },
    environmentNarration: {
      description: 'Zone announcements, event callouts',
      priority: 'High',
      defaultEnabled: true
    },
    systemMessages: {
      description: 'Match start, victory, elimination',
      priority: 'High',
      defaultEnabled: true
    }
  },

  settings: {
    enabled: {
      type: 'toggle',
      default: false,
      description: 'Enable subtitles'
    },
    size: {
      type: 'slider',
      range: ['Small', 'Medium', 'Large', 'Extra Large'],
      default: 'Medium',
      fontSizes: { small: 14, medium: 18, large: 24, extraLarge: 32 }
    },
    background: {
      type: 'slider',
      range: [0, 100],
      default: 75,
      description: 'Background opacity percentage'
    },
    speakerNames: {
      type: 'toggle',
      default: true,
      description: 'Show who is speaking'
    }
  }
};
```

### Subtitle Display

```typescript
const SUBTITLE_DISPLAY = {
  position: {
    location: 'bottom-center',
    safeArea: true,
    offsetFromBottom: 80, // pixels
    maxWidth: '80%' // of screen width
  },

  appearance: {
    font: 'System default (supports all languages)',
    color: '#FFFFFF',
    shadow: {
      enabled: true,
      color: '#000000',
      blur: 2,
      offset: { x: 1, y: 1 }
    },
    background: {
      color: '#000000',
      opacity: 0.75, // Adjustable
      padding: { horizontal: 12, vertical: 8 },
      borderRadius: 4
    }
  },

  behavior: {
    maxLines: 2,
    fadeInDuration: 150, // ms
    fadeOutDuration: 300, // ms
    minimumDisplayTime: 1500, // ms
    wordsPerMinute: 180, // Reading speed calculation
    queueing: 'FIFO with priority override'
  },

  speakerFormat: {
    template: '[{speaker}]: {text}',
    speakerColor: '#FFD700', // Gold for speaker name
    colonIncluded: true
  }
};
```

### Subtitle Content Examples

```typescript
const SUBTITLE_EXAMPLES = {
  announcer: [
    { speaker: 'Announcer', text: 'Match starting in 10 seconds!' },
    { speaker: 'Announcer', text: 'The storm is closing!' },
    { speaker: 'Announcer', text: '10 pirates remain!' },
    { speaker: 'Announcer', text: 'Victory! You are the last pirate standing!' }
  ],

  tutorial: [
    { speaker: 'Captain', text: 'Welcome aboard, recruit! Let me show ye the ropes.' },
    { speaker: 'Captain', text: 'Tap and hold to move. Swipe to look around.' }
  ],

  character: [
    { speaker: 'Your Pirate', text: 'Reloading!' },
    { speaker: 'Your Pirate', text: 'I need healing!' },
    { speaker: 'Teammate', text: 'Enemy spotted!' }
  ],

  environment: [
    { speaker: 'System', text: '[Storm approaching from the east]' },
    { speaker: 'System', text: '[Supply drop incoming]' }
  ]
};
```

---

## Visual Combat Indicators

### Directional Damage Indicator

```typescript
const DIRECTIONAL_DAMAGE_INDICATOR = {
  description: 'Visual indicator showing direction of incoming fire',

  appearance: {
    type: 'Arc segment on screen edge',
    color: {
      gunfire: '#FF4444',
      ability: '#FF8800',
      zone: '#9933FF'
    },
    opacity: {
      initial: 0.9,
      fadeTarget: 0
    },
    size: {
      arcAngle: 45, // degrees
      thickness: 8, // pixels
      distanceFromEdge: 20 // pixels
    }
  },

  positioning: {
    method: '360-degree around screen center',
    accuracy: '8 directional zones (N, NE, E, SE, S, SW, W, NW)',
    multipleSourcesMerge: false // Show each source separately
  },

  timing: {
    appearDuration: 100, // ms
    holdDuration: 500, // ms
    fadeDuration: 300, // ms
    cooldownPerDirection: 200 // ms before same direction can trigger again
  },

  intensity: {
    scaleWithDamage: true,
    lowDamage: { opacity: 0.6, thickness: 6 },
    highDamage: { opacity: 1.0, thickness: 12 }
  }
};
```

### Enemy Gunfire Indicator

```typescript
const ENEMY_GUNFIRE_INDICATOR = {
  description: 'Visual ping when enemy fires nearby',

  trigger: {
    condition: 'Enemy weapon fired within audio range',
    range: 50, // meters
    excludeSilenced: true // If silenced weapons exist
  },

  display: {
    type: 'Directional chevron/arrow',
    position: 'Screen edge in direction of sound',
    appearance: {
      icon: 'Muzzle flash icon or chevron',
      color: '#FF6B6B',
      size: 32, // pixels
      animation: 'Pulse once'
    }
  },

  behavior: {
    stackable: true, // Multiple enemies can show
    maxIndicators: 4, // Prevent clutter
    duration: 800, // ms
    fadeOut: 200 // ms
  },

  distanceHint: {
    enabled: true,
    method: 'Icon size varies with distance',
    close: { size: 40, opacity: 1.0 },
    medium: { size: 32, opacity: 0.8 },
    far: { size: 24, opacity: 0.6 }
  }
};
```

### Footstep Indicator

```typescript
const FOOTSTEP_INDICATOR = {
  description: 'Visual indicator for nearby enemy footsteps',

  trigger: {
    condition: 'Enemy footsteps within hearing range',
    range: 20, // meters
    excludeTeammates: true,
    excludeCrouching: true // If crouch reduces sound
  },

  display: {
    type: 'Footprint icon on screen edge',
    position: 'Direction of footsteps',
    appearance: {
      icon: 'Boot/footprint silhouette',
      color: '#FFAA00', // Orange/amber
      size: 28,
      animation: 'Step rhythm pulse'
    }
  },

  intensityLevels: {
    walking: { pulseSpeed: 'slow', opacity: 0.6 },
    running: { pulseSpeed: 'fast', opacity: 0.9 },
    sprinting: { pulseSpeed: 'veryFast', opacity: 1.0 }
  },

  setting: {
    name: 'Footstep Visualizer',
    type: 'toggle',
    default: false, // Optional feature
    description: 'Show visual indicator for nearby enemy footsteps'
  }
};
```

---

## Zone/Storm Visual Warnings

### Storm Warning System

```typescript
const STORM_VISUAL_WARNING = {
  description: 'Visual indicators replacing audio storm warnings',

  preWarning: {
    trigger: '30 seconds before zone moves',
    display: {
      type: 'Screen edge pulse',
      color: '#9933FF', // Purple
      intensity: 'Low pulse',
      frequency: 2, // seconds between pulses
      position: 'Direction of safe zone'
    },
    icon: {
      show: true,
      type: 'Storm cloud icon',
      position: 'Top of screen',
      animation: 'Gentle sway'
    },
    text: {
      show: true,
      message: 'Storm closing soon',
      position: 'Below icon'
    }
  },

  activeWarning: {
    trigger: 'Zone is moving',
    display: {
      type: 'Screen edge glow',
      color: '#FF4444', // Red
      intensity: 'Strong pulse',
      frequency: 1, // second
      affectedEdges: 'Edge(s) where storm is approaching'
    },
    icon: {
      animation: 'Urgent shake'
    },
    text: {
      message: 'STORM CLOSING!'
    }
  },

  inStorm: {
    trigger: 'Player is in storm',
    display: {
      type: 'Screen vignette',
      color: '#660066', // Dark purple
      opacity: 0.4,
      pulseWithDamage: true
    },
    directionalArrow: {
      show: true,
      pointsTo: 'Nearest safe zone edge',
      color: '#00FF00', // Green
      size: 48
    }
  }
};
```

---

## Low Health Visual Indicator

### Health Warning System

```typescript
const LOW_HEALTH_VISUAL = {
  description: 'Visual replacement for audio heartbeat/warning sounds',

  thresholds: {
    warning: {
      healthPercent: 30,
      display: {
        vignette: {
          color: '#FF0000',
          opacity: 0.15,
          pulseEnabled: true,
          pulseFrequency: 1.5 // seconds
        },
        healthBarFlash: true
      }
    },

    critical: {
      healthPercent: 15,
      display: {
        vignette: {
          color: '#FF0000',
          opacity: 0.25,
          pulseEnabled: true,
          pulseFrequency: 0.8 // seconds (faster)
        },
        screenEdgeGlow: {
          enabled: true,
          color: '#FF0000',
          thickness: 6
        },
        healthBarFlash: {
          speed: 'fast'
        },
        icon: {
          show: true,
          type: 'Heart icon',
          position: 'Near health bar',
          animation: 'Pulsing'
        }
      }
    }
  },

  settings: {
    intensitySlider: {
      name: 'Low Health Visual Intensity',
      range: [0, 100],
      default: 75,
      description: 'How prominent the low health warning appears'
    },
    disableOption: {
      name: 'Disable Low Health Effects',
      type: 'toggle',
      default: false,
      description: 'Turn off screen effects when low on health'
    }
  }
};
```

---

## Mono Audio Option

### Audio Channel Configuration

```typescript
const MONO_AUDIO = {
  description: 'Combine stereo audio into mono for single-ear hearing',

  settings: {
    enabled: {
      type: 'toggle',
      default: false,
      description: 'Combine left and right audio channels'
    },
    balance: {
      type: 'slider',
      range: [-100, 100], // -100 = full left, 100 = full right
      default: 0,
      description: 'Adjust audio balance between left and right',
      showWhen: 'mono disabled'
    }
  },

  implementation: {
    method: 'Mix L+R channels equally',
    preserveVolume: true,
    affectsAllAudio: true
  },

  visualCompensation: {
    description: 'When mono is enabled, enhance directional visual cues',
    enhancements: [
      'Auto-enable directional damage indicators',
      'Auto-enable footstep visualizer',
      'Auto-enable gunfire indicators'
    ]
  }
};
```

---

## Haptic/Vibration Feedback

### Vibration Pattern System

```typescript
const VIBRATION_PATTERNS = {
  description: 'Distinct vibration patterns for different audio events',

  settings: {
    masterToggle: {
      name: 'Vibration Feedback',
      type: 'toggle',
      default: true
    },
    intensity: {
      name: 'Vibration Strength',
      type: 'slider',
      range: ['Light', 'Medium', 'Strong'],
      default: 'Medium'
    },
    categoryToggles: {
      combat: { default: true, description: 'Damage, hits, eliminations' },
      environment: { default: true, description: 'Zone, events, pickups' },
      ui: { default: false, description: 'Button presses, notifications' }
    }
  },

  patterns: {
    // Combat patterns
    takeDamage: {
      category: 'combat',
      pattern: [100], // Single short burst
      intensity: 'Scales with damage amount',
      cooldown: 100 // ms
    },
    dealDamage: {
      category: 'combat',
      pattern: [50], // Very short tap
      intensity: 'light'
    },
    elimination: {
      category: 'combat',
      pattern: [100, 50, 100, 50, 200], // Victory pattern
      intensity: 'strong'
    },
    death: {
      category: 'combat',
      pattern: [300, 100, 500], // Dramatic pattern
      intensity: 'strong'
    },

    // Environment patterns
    zoneWarning: {
      category: 'environment',
      pattern: [200, 200, 200], // Three pulses
      intensity: 'medium',
      repeat: true,
      repeatInterval: 3000 // ms
    },
    zoneDamage: {
      category: 'environment',
      pattern: [150], // Continuous ticks
      repeatWhileInZone: true,
      repeatInterval: 1000
    },
    lootPickup: {
      category: 'environment',
      pattern: [30], // Quick tap
      intensity: 'light'
    },
    legendaryLoot: {
      category: 'environment',
      pattern: [50, 30, 50, 30, 100], // Special pattern
      intensity: 'medium'
    },
    supplyDrop: {
      category: 'environment',
      pattern: [100, 100, 100], // Alert pattern
      intensity: 'medium'
    },

    // UI patterns
    buttonPress: {
      category: 'ui',
      pattern: [10], // Minimal tap
      intensity: 'light'
    },
    abilityReady: {
      category: 'ui',
      pattern: [50, 50], // Double tap
      intensity: 'light'
    },
    matchFound: {
      category: 'ui',
      pattern: [100, 50, 100, 50, 100], // Attention pattern
      intensity: 'strong'
    }
  }
};
```

### Directional Haptics (Advanced)

```typescript
const DIRECTIONAL_HAPTICS = {
  description: 'Use haptic patterns to convey direction (iOS Taptic Engine)',

  availability: {
    iOS: 'iPhone 8 and later with Taptic Engine',
    android: 'Devices with advanced haptic motors'
  },

  implementation: {
    method: 'Vary intensity based on direction quadrant',
    leftPattern: 'Emphasized left intensity',
    rightPattern: 'Emphasized right intensity',
    frontPattern: 'Full intensity',
    backPattern: 'Muted intensity'
  },

  useCase: {
    enemyDirection: 'Haptic hints at enemy location',
    zoneDirection: 'Haptic points toward safe zone'
  },

  setting: {
    name: 'Directional Haptics',
    type: 'toggle',
    default: false,
    description: 'Use vibration patterns to indicate directions',
    experimental: true
  }
};
```

---

## Settings UI

### Audio Accessibility Settings Panel

```typescript
const AUDIO_ACCESSIBILITY_SETTINGS = {
  location: 'Settings > Accessibility > Audio & Hearing',

  sections: {
    subtitles: {
      title: 'Subtitles',
      controls: [
        {
          type: 'toggle',
          id: 'subtitles_enabled',
          label: 'Enable Subtitles',
          default: false
        },
        {
          type: 'segmented',
          id: 'subtitle_size',
          label: 'Subtitle Size',
          options: ['S', 'M', 'L', 'XL'],
          default: 'M',
          showWhen: 'subtitles_enabled'
        },
        {
          type: 'slider',
          id: 'subtitle_background',
          label: 'Background Opacity',
          range: [0, 100],
          default: 75,
          unit: '%',
          showWhen: 'subtitles_enabled'
        },
        {
          type: 'toggle',
          id: 'speaker_names',
          label: 'Show Speaker Names',
          default: true,
          showWhen: 'subtitles_enabled'
        }
      ]
    },

    visualIndicators: {
      title: 'Visual Sound Indicators',
      controls: [
        {
          type: 'toggle',
          id: 'directional_damage',
          label: 'Directional Damage Indicator',
          default: true,
          description: 'Show direction of incoming damage'
        },
        {
          type: 'toggle',
          id: 'gunfire_indicator',
          label: 'Nearby Gunfire Indicator',
          default: false,
          description: 'Visual ping when enemies fire nearby'
        },
        {
          type: 'toggle',
          id: 'footstep_indicator',
          label: 'Footstep Visualizer',
          default: false,
          description: 'Show direction of enemy footsteps'
        },
        {
          type: 'toggle',
          id: 'enhanced_storm_visual',
          label: 'Enhanced Storm Warnings',
          default: true,
          description: 'Additional visual cues for zone changes'
        }
      ]
    },

    audioOptions: {
      title: 'Audio Options',
      controls: [
        {
          type: 'toggle',
          id: 'mono_audio',
          label: 'Mono Audio',
          default: false,
          description: 'Combine stereo channels for single-ear listening'
        },
        {
          type: 'slider',
          id: 'audio_balance',
          label: 'Audio Balance',
          range: [-100, 100],
          default: 0,
          labels: { left: 'L', right: 'R' },
          showWhen: '!mono_audio'
        }
      ]
    },

    vibration: {
      title: 'Vibration Feedback',
      controls: [
        {
          type: 'toggle',
          id: 'vibration_enabled',
          label: 'Enable Vibration',
          default: true
        },
        {
          type: 'segmented',
          id: 'vibration_intensity',
          label: 'Vibration Strength',
          options: ['Light', 'Medium', 'Strong'],
          default: 'Medium',
          showWhen: 'vibration_enabled'
        },
        {
          type: 'toggle',
          id: 'vibration_combat',
          label: 'Combat Vibration',
          default: true,
          showWhen: 'vibration_enabled'
        },
        {
          type: 'toggle',
          id: 'vibration_environment',
          label: 'Environment Vibration',
          default: true,
          showWhen: 'vibration_enabled'
        }
      ]
    }
  },

  quickPreset: {
    name: 'Deaf/HoH Mode',
    button: 'Enable Full Audio Accessibility',
    description: 'Enables all visual and haptic alternatives to audio cues',
    settings: {
      subtitles_enabled: true,
      directional_damage: true,
      gunfire_indicator: true,
      footstep_indicator: true,
      enhanced_storm_visual: true,
      vibration_enabled: true,
      vibration_combat: true,
      vibration_environment: true
    }
  }
};
```

---

## Critical Audio-to-Visual Mapping

### Complete Audio Event Coverage

```typescript
const AUDIO_TO_VISUAL_MAPPING = {
  critical: {
    description: 'Must have visual alternative',
    events: [
      {
        audio: 'Enemy gunfire',
        visual: 'Directional indicator + muzzle flash icon',
        haptic: 'Optional vibration'
      },
      {
        audio: 'Taking damage',
        visual: 'Directional damage arc + health bar flash',
        haptic: 'Damage vibration'
      },
      {
        audio: 'Zone warning',
        visual: 'Screen pulse + icon + text',
        haptic: 'Warning pattern'
      },
      {
        audio: 'Zone damage',
        visual: 'Vignette + directional arrow to safety',
        haptic: 'Damage ticks'
      },
      {
        audio: 'Low health heartbeat',
        visual: 'Red vignette pulse + heart icon',
        haptic: 'Heartbeat pattern'
      },
      {
        audio: 'Enemy footsteps',
        visual: 'Footstep indicator (optional)',
        haptic: 'Step pattern (optional)'
      },
      {
        audio: 'Announcer callouts',
        visual: 'Subtitles + banner notification',
        haptic: 'None'
      }
    ]
  },

  important: {
    description: 'Should have visual alternative',
    events: [
      {
        audio: 'Ability ready',
        visual: 'Ability button glow + icon change',
        haptic: 'Ready tap'
      },
      {
        audio: 'Reload complete',
        visual: 'Ammo counter flash',
        haptic: 'Light tap'
      },
      {
        audio: 'Supply drop',
        visual: 'Map marker + screen indicator',
        haptic: 'Alert pattern'
      },
      {
        audio: 'Teammate eliminated',
        visual: 'Team UI update + notification',
        haptic: 'Sad pattern'
      }
    ]
  },

  optional: {
    description: 'Nice to have visual alternative',
    events: [
      {
        audio: 'Ambient sounds',
        visual: 'None needed',
        haptic: 'None'
      },
      {
        audio: 'Music',
        visual: 'None needed',
        haptic: 'None'
      },
      {
        audio: 'UI sounds',
        visual: 'Existing UI feedback sufficient',
        haptic: 'Optional taps'
      }
    ]
  }
};
```

---

## Testing Requirements

```typescript
const AUDIO_ACCESSIBILITY_TESTING = {
  testWithSoundOff: {
    description: 'All testers should complete full match with audio muted',
    checkpoints: [
      'Can identify enemy direction when shot',
      'Can navigate to safe zone',
      'Knows when health is critical',
      'Understands match announcements via subtitles',
      'Can locate nearby threats'
    ]
  },

  userTesting: {
    recruitment: 'Deaf and hard-of-hearing gamers',
    sources: [
      'Deaf gaming communities',
      'AbleGamers charity',
      'SpecialEffect organization'
    ],
    feedback: [
      'Can you play competitively without sound?',
      'Are any critical cues missing?',
      'Do visual indicators clutter the screen?',
      'Is vibration feedback helpful?'
    ]
  },

  successCriteria: {
    competitiveParity: 'Deaf players should not be at significant disadvantage',
    informationAccess: '100% of critical audio has visual alternative',
    userSatisfaction: '>= 4/5 rating from deaf testers'
  }
};
```

---

## Analytics

```typescript
const AUDIO_ACCESSIBILITY_ANALYTICS = {
  events: [
    {
      name: 'audio_accessibility_enabled',
      properties: {
        feature: 'subtitles | gunfire_indicator | footstep_indicator | etc',
        preset_used: 'boolean (quick preset)'
      }
    },
    {
      name: 'mono_audio_enabled',
      properties: {
        balance_setting: 'number'
      }
    },
    {
      name: 'vibration_settings_changed',
      properties: {
        intensity: 'light | medium | strong',
        categories_enabled: 'array'
      }
    }
  ],

  metrics: [
    'Percentage of players using subtitles',
    'Percentage using visual combat indicators',
    'Percentage using mono audio',
    'Deaf/HoH preset adoption rate',
    'Session length comparison with accessibility features'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial audio accessibility specification |
