# Emote Animations Specification

## Document Information
- **Task ID:** ART-023
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-08

---

## Overview

This document specifies the emote animation system for Plunderstorm Mobile, enabling player expression through character animations that serve both social engagement and monetization purposes.

---

## Launch Emote Roster

### Free Emotes (Unlocked by Default)

```typescript
const FREE_EMOTES = {
  wave: {
    id: 'emote_wave',
    name: 'Wave',
    description: 'A friendly greeting',
    category: 'greeting',
    duration: 2.0, // seconds
    looping: false,
    audio: 'None',
    rarity: 'Common',
    unlockMethod: 'Default'
  },

  salute: {
    id: 'emote_salute',
    name: 'Pirate Salute',
    description: 'A respectful pirate salute',
    category: 'greeting',
    duration: 1.8,
    looping: false,
    audio: 'Subtle cloth rustle',
    rarity: 'Common',
    unlockMethod: 'Default'
  },

  sit: {
    id: 'emote_sit',
    name: 'Sit',
    description: 'Take a rest',
    category: 'idle',
    duration: null, // Until cancelled
    looping: true,
    audio: 'None',
    rarity: 'Common',
    unlockMethod: 'Default'
  },

  bow: {
    id: 'emote_bow',
    name: 'Bow',
    description: 'A respectful bow',
    category: 'greeting',
    duration: 2.2,
    looping: false,
    audio: 'None',
    rarity: 'Common',
    unlockMethod: 'Default'
  }
};
```

### Earnable Emotes

```typescript
const EARNABLE_EMOTES = {
  laugh: {
    id: 'emote_laugh',
    name: 'Hearty Laugh',
    description: 'Throw your head back in laughter',
    category: 'expression',
    duration: 2.5,
    looping: false,
    audio: {
      type: 'Voice line',
      sound: 'Pirate laugh',
      volume: 0.7
    },
    rarity: 'Uncommon',
    unlockMethod: {
      type: 'Achievement',
      requirement: 'Win 10 matches'
    }
  },

  taunt: {
    id: 'emote_taunt',
    name: 'Taunt',
    description: 'Point and challenge your foes',
    category: 'combat',
    duration: 2.0,
    looping: false,
    audio: {
      type: 'Voice line',
      sound: 'Challenging grunt',
      volume: 0.6
    },
    rarity: 'Uncommon',
    unlockMethod: {
      type: 'Level',
      requirement: 'Reach level 15'
    }
  },

  surrender: {
    id: 'emote_surrender',
    name: 'Surrender',
    description: 'Raise the white flag',
    category: 'expression',
    duration: 3.0,
    looping: true,
    audio: 'None',
    rarity: 'Uncommon',
    unlockMethod: {
      type: 'Quest',
      requirement: 'Complete "The Defeated" quest'
    }
  },

  victoryDance: {
    id: 'emote_victory_dance',
    name: 'Victory Jig',
    description: 'Celebrate your triumph',
    category: 'dance',
    duration: 4.5,
    looping: true,
    audio: {
      type: 'Music',
      sound: 'Upbeat sea shanty clip',
      volume: 0.5
    },
    rarity: 'Rare',
    unlockMethod: {
      type: 'Battle Pass',
      tier: 'Free track, tier 50'
    }
  }
};
```

### Premium Emotes (Monetized)

```typescript
const PREMIUM_EMOTES = {
  pirateJig: {
    id: 'emote_pirate_jig',
    name: 'Pirate Jig',
    description: 'The classic pirate dance',
    category: 'dance',
    duration: 6.0,
    looping: true,
    audio: {
      type: 'Music',
      sound: 'Lively sea shanty',
      volume: 0.6,
      syncPoints: [0.5, 1.0, 1.5, 2.0] // Beat sync frames
    },
    rarity: 'Epic',
    unlockMethod: {
      type: 'Shop',
      price: { gems: 500 }
    },
    vfx: {
      footsteps: 'Dust particles',
      ambient: 'Musical notes floating'
    }
  },

  legendaryDance: {
    id: 'emote_legendary_dance',
    name: 'Kraken\'s Dance',
    description: 'An legendary dance invoking the Kraken',
    category: 'dance',
    duration: 8.0,
    looping: true,
    audio: {
      type: 'Music',
      sound: 'Epic orchestral sea theme',
      volume: 0.7
    },
    rarity: 'Legendary',
    unlockMethod: {
      type: 'Shop',
      price: { gems: 1200 }
    },
    vfx: {
      tentacles: 'Spectral tentacles appear',
      water: 'Water splash effects',
      glow: 'Character glows during dance'
    }
  },

  battlePassDance: {
    id: 'emote_bp_dance',
    name: 'Season 1 Celebration',
    description: 'Exclusive Season 1 dance',
    category: 'dance',
    duration: 5.5,
    looping: true,
    audio: {
      type: 'Music',
      sound: 'Season 1 theme remix',
      volume: 0.6
    },
    rarity: 'Legendary',
    unlockMethod: {
      type: 'Battle Pass',
      tier: 'Premium track, tier 100'
    },
    exclusive: true,
    vfx: {
      seasonal: 'Season 1 themed particles'
    }
  }
};
```

---

## Animation Specifications

### Animation Requirements

```typescript
const ANIMATION_SPECS = {
  skeleton: {
    rig: 'Standard humanoid rig',
    bones: 65, // Full body + fingers
    compatibility: 'All player character models'
  },

  quality: {
    frameRate: 30, // FPS for animation
    keyframes: {
      short: '60-90 frames (2-3 seconds)',
      looping: '120-180 frames (4-6 seconds)',
      complex: '180-240 frames (6-8 seconds)'
    }
  },

  blending: {
    enterBlend: 0.2, // Seconds to blend into emote
    exitBlend: 0.15, // Seconds to blend out
    interruptBlend: 0.1 // Quick blend on cancel
  },

  rootMotion: {
    movement: 'Minimal (character stays in place)',
    rotation: 'None (face same direction)',
    exception: 'Sit may lower character'
  },

  ik: {
    feet: 'Ground contact IK for uneven terrain',
    hands: 'Prop holding for applicable emotes',
    look: 'Optional look-at target'
  }
};
```

### Animation Categories

```typescript
const ANIMATION_CATEGORIES = {
  greeting: {
    characteristics: [
      'Short duration (1.5-2.5s)',
      'Single play, no loop',
      'Arm-focused movement',
      'Friendly/neutral tone'
    ],
    examples: ['Wave', 'Salute', 'Bow']
  },

  expression: {
    characteristics: [
      'Medium duration (2-3s)',
      'Facial expression emphasis',
      'May include voice audio',
      'Emotional communication'
    ],
    examples: ['Laugh', 'Surrender', 'Cry']
  },

  dance: {
    characteristics: [
      'Long duration (4-8s per loop)',
      'Full body movement',
      'Music synchronized',
      'Looping animation',
      'Premium monetization focus'
    ],
    examples: ['Pirate Jig', 'Victory Dance', 'Legendary Dance']
  },

  combat: {
    characteristics: [
      'Provocative/competitive',
      'Medium duration',
      'May include voice',
      'Taunt-style animations'
    ],
    examples: ['Taunt', 'Point', 'Flex']
  },

  idle: {
    characteristics: [
      'Indefinite looping',
      'Relaxed pose',
      'Low movement',
      'Can be interrupted anytime'
    ],
    examples: ['Sit', 'Lean', 'Sleep']
  }
};
```

---

## Emote System

### Emote Wheel UI

```typescript
const EMOTE_WHEEL = {
  access: {
    button: 'Dedicated emote button on HUD',
    gesture: 'Long press on character (alternative)',
    keyboard: 'Quick key bindings (1-4 for favorites)'
  },

  layout: {
    type: 'Radial wheel',
    slots: 8, // 8 equipped emotes
    center: 'Cancel/close button',
    expansion: 'Swipe for more pages'
  },

  display: {
    icon: 'Emote thumbnail',
    name: 'Emote name on hover/select',
    preview: 'Mini animation preview on hold',
    locked: 'Greyed out with lock icon'
  },

  selection: {
    method: 'Drag to select, release to play',
    alternative: 'Tap slot directly',
    cancel: 'Tap center or outside wheel'
  },

  customization: {
    location: 'Locker > Emotes',
    equip: 'Drag emote to wheel slot',
    favorites: 'Mark favorites for quick access'
  }
};
```

### Playback System

```typescript
const EMOTE_PLAYBACK = {
  trigger: {
    input: 'Select from emote wheel',
    validation: 'Check if emote owned and equipped',
    state: 'Check player can emote (not downed, etc.)'
  },

  execution: {
    blend: 'Blend from current animation',
    camera: 'Optional camera adjustment for dance',
    movement: 'Disable movement input during emote',
    collision: 'Maintain collision (can take damage)'
  },

  interruption: {
    movementInput: 'Any movement cancels emote',
    damage: 'Taking damage cancels emote',
    abilityUse: 'Using ability cancels emote',
    manualCancel: 'Emote button cancels',
    death: 'Death immediately stops emote'
  },

  looping: {
    behavior: 'Loop until interrupted',
    maxDuration: 30, // Seconds max loop time
    autoExit: 'Blend to idle after max duration'
  },

  networking: {
    broadcast: 'Emote trigger sent to all nearby players',
    range: 50, // Meters visibility range
    optimization: 'Don\'t send to players who can\'t see'
  }
};
```

---

## Audio System

### Audio Specifications

```typescript
const EMOTE_AUDIO = {
  types: {
    voiceLine: {
      source: 'Character-specific voice',
      variants: 'Multiple per emote for variety',
      volume: '60-80% of game audio',
      range: 'Audible within 20 meters'
    },
    music: {
      source: 'Emote-specific track',
      duration: 'Matches animation loop',
      volume: '50-70% of game audio',
      range: 'Audible within 15 meters',
      spatialization: '3D positioned audio'
    },
    sfx: {
      source: 'Foley and effects',
      examples: ['Footsteps', 'Cloth rustle', 'Prop sounds'],
      volume: '40-60% of game audio',
      sync: 'Keyframe synchronized'
    }
  },

  mixing: {
    ducking: 'Slightly duck game audio during emote music',
    priority: 'Emote audio below combat sounds',
    overlap: 'Multiple emotes mix (don\'t stack volume)'
  },

  settings: {
    emoteVolume: 'Player can adjust emote audio volume',
    muteOthers: 'Option to mute other players\' emote audio',
    default: 'On at 80% volume'
  }
};
```

### Music Synchronization

```typescript
const MUSIC_SYNC = {
  implementation: {
    method: 'Animation keyframe markers',
    beatMarkers: 'Define beat points in animation',
    audioSync: 'Music aligned to animation start'
  },

  looping: {
    seamless: 'Music loops match animation loops',
    crossfade: 'Brief crossfade at loop point',
    timing: 'Exact frame alignment'
  },

  networkSync: {
    approach: 'Local playback, not networked audio',
    timing: 'Each client plays from animation start',
    tolerance: 'Minor desync acceptable (<100ms)'
  }
};
```

---

## Visual Effects

### VFX Specifications

```typescript
const EMOTE_VFX = {
  tiers: {
    common: {
      vfx: 'None or minimal',
      examples: 'Basic dust on footsteps'
    },
    uncommon: {
      vfx: 'Subtle effects',
      examples: 'Small particles, simple trails'
    },
    rare: {
      vfx: 'Noticeable effects',
      examples: 'Sparkles, colored particles'
    },
    epic: {
      vfx: 'Prominent effects',
      examples: 'Musical notes, energy effects, props'
    },
    legendary: {
      vfx: 'Elaborate effects',
      examples: 'Environmental changes, complex particles, auras'
    }
  },

  performance: {
    budget: {
      particles: 'Max 100 particles per emote',
      drawCalls: 'Max 3 additional draw calls',
      overdraw: 'Minimal screen coverage'
    },
    lod: {
      distance: 'Reduce VFX at distance',
      quality: 'Scale with device tier'
    }
  },

  examples: {
    pirateJig: {
      footDust: 'Puff on each stomp',
      musicNotes: 'Floating note particles',
      ambient: 'Subtle sparkle trail'
    },
    krakenDance: {
      tentacles: 'Spectral tentacle overlays',
      water: 'Splashing water particles',
      glow: 'Character rim lighting',
      ground: 'Wet ground decal'
    }
  }
};
```

---

## Shop Integration

### Preview System

```typescript
const EMOTE_PREVIEW = {
  location: 'Shop item detail page',

  preview: {
    automatic: 'Auto-play preview on page load',
    loop: 'Loop for duration of page visit',
    camera: 'Dedicated preview camera angle',
    background: 'Neutral background or thematic'
  },

  interaction: {
    rotate: 'Swipe to rotate character',
    zoom: 'Pinch to zoom',
    replay: 'Tap to replay from start'
  },

  audio: {
    default: 'Muted by default',
    unmute: 'Sound icon to enable audio',
    volume: 'Respects app audio settings'
  },

  comparison: {
    owned: 'Show "Owned" badge if already unlocked',
    equipped: 'Option to equip directly from preview'
  }
};
```

### Purchase Flow

```typescript
const EMOTE_PURCHASE = {
  display: {
    price: 'Gem cost clearly shown',
    rarity: 'Rarity indicator',
    preview: 'Full animation preview',
    description: 'Emote description and lore'
  },

  purchase: {
    button: 'Purchase button with price',
    confirmation: 'Confirm purchase dialog',
    insufficientFunds: 'Link to gem purchase if not enough'
  },

  postPurchase: {
    celebration: 'Unlock celebration animation',
    notification: 'Added to collection notification',
    equip: 'Option to equip immediately',
    share: 'Optional social share'
  }
};
```

---

## Locker/Collection

### Emote Collection UI

```typescript
const EMOTE_COLLECTION = {
  location: 'Main Menu > Locker > Emotes',

  layout: {
    grid: 'Grid view of all emotes',
    sorting: ['Rarity', 'Category', 'Newest', 'Alphabetical'],
    filtering: ['Owned', 'Not owned', 'Category', 'Rarity']
  },

  emoteCard: {
    thumbnail: 'Animated thumbnail',
    name: 'Emote name',
    rarity: 'Color-coded border',
    status: 'Owned/locked indicator',
    equipped: 'Equipped badge if in wheel'
  },

  detail: {
    tap: 'Opens detail view',
    fullPreview: 'Full animation playback',
    equip: 'Add to emote wheel',
    source: 'How to unlock (if locked)'
  },

  statistics: {
    total: 'X of Y emotes collected',
    byRarity: 'Breakdown by rarity',
    completion: 'Collection progress %'
  }
};
```

---

## Testing Requirements

```typescript
const EMOTE_TESTING = {
  animation: [
    'All emotes play correctly',
    'Blending transitions are smooth',
    'Looping is seamless',
    'Interruption works properly',
    'Works with all character models'
  ],

  audio: [
    'Audio syncs with animation',
    'Volume levels appropriate',
    'Spatial audio works correctly',
    'Mute settings respected',
    'No audio glitches on loop'
  ],

  vfx: [
    'Effects display correctly',
    'Performance within budget',
    'LOD works at distance',
    'No visual artifacts'
  ],

  networking: [
    'Emotes visible to other players',
    'Reasonable network bandwidth',
    'Handles latency gracefully',
    'Disconnect during emote handled'
  ],

  ui: [
    'Emote wheel functions correctly',
    'Preview works in shop',
    'Collection displays properly',
    'Equipping works'
  ],

  monetization: [
    'Purchase flow complete',
    'Correct prices displayed',
    'Unlocks grant properly',
    'Battle Pass emotes unlock at correct tier'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-08 | Development Team | Initial emote animations specification |
