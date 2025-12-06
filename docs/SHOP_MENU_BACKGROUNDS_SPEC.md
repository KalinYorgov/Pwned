# Shop and Menu Backgrounds Specification

## Document Information
- **Task ID:** ART-020
- **Priority:** P1
- **Complexity:** S
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines background art for all menu screens and shop interfaces in Plunderstorm Mobile. These backgrounds establish atmosphere, reinforce the pirate theme, and provide visual context for UI elements while maintaining readability.

### 1.2 Design Goals
- **Atmospheric Immersion:** Each screen feels like a location in the pirate world
- **UI Readability:** Backgrounds enhance, never compete with UI elements
- **Visual Variety:** Different screens have distinct but cohesive looks
- **Performance Friendly:** Optimized for mobile memory and rendering
- **Subtle Animation:** Living backgrounds without distraction

### 1.3 Art Style Alignment
Following the Visual Style Guide:
- Stylized, painterly aesthetic with bold colors
- Golden hour tropical lighting as primary mood
- Rich blues, warm golds, and weathered wood tones
- Soft focus for depth-of-field effect in backgrounds

---

## 2. Main Menu Background

### 2.1 Scene Concept: Harbor at Golden Hour

```typescript
const MAIN_MENU_BACKGROUND = {
  name: 'harbor_sunset',
  concept: 'Player\'s ship anchored in a tropical harbor at golden hour',

  composition: {
    foreground: {
      elements: ['ship_railing', 'hanging_lanterns', 'rope_coils'],
      blur: 0, // Sharp
      parallax: 0.1
    },
    midground: {
      elements: ['main_ship_deck', 'mast_silhouettes', 'crew_silhouettes'],
      blur: 0.2,
      parallax: 0.05
    },
    background: {
      elements: ['harbor_town', 'distant_ships', 'mountains'],
      blur: 0.5,
      parallax: 0.02
    },
    sky: {
      elements: ['sunset_gradient', 'clouds', 'flying_birds'],
      blur: 0,
      parallax: 0
    }
  },

  lighting: {
    mood: 'golden_hour',
    primaryLight: {
      direction: 'left',
      color: '#FFB366',
      intensity: 1.0
    },
    ambientColor: '#4A6B8A',
    rimLight: {
      color: '#FF8C42',
      intensity: 0.6
    }
  },

  colorPalette: {
    dominant: '#1E3A5F', // Deep ocean blue
    secondary: '#FFB366', // Golden sunset
    accent: '#8B4513', // Warm wood
    highlight: '#FFD700' // Gold accents
  }
};
```

### 2.2 Animated Elements

```typescript
const MAIN_MENU_ANIMATIONS = {
  water: {
    type: 'gentle_wave',
    speed: 0.5,
    amplitude: 5, // Pixels
    reflection: true
  },

  clouds: {
    type: 'slow_drift',
    speed: 0.1,
    direction: 'right'
  },

  lanterns: {
    type: 'gentle_sway',
    speed: 0.3,
    amplitude: 3,
    glow: {
      enabled: true,
      flicker: true,
      color: '#FFB347'
    }
  },

  flags: {
    type: 'wind_flutter',
    speed: 0.4,
    amplitude: 8
  },

  birds: {
    type: 'distant_flight',
    count: 3,
    speed: 0.2,
    path: 'gentle_curve'
  },

  shipBob: {
    type: 'subtle_rock',
    speed: 0.1,
    amplitude: 2
  }
};
```

### 2.3 UI Integration

```typescript
const MAIN_MENU_UI_ZONES = {
  // Dark/neutral areas for UI placement
  safeZones: [
    {
      name: 'center_content',
      position: { x: 0.2, y: 0.15, width: 0.6, height: 0.7 },
      darkenOverlay: 0.3
    },
    {
      name: 'bottom_nav',
      position: { x: 0, y: 0.85, width: 1, height: 0.15 },
      darkenOverlay: 0.5
    }
  ],

  // Areas to keep visually interesting
  featureZones: [
    {
      name: 'sunset_view',
      position: { x: 0.7, y: 0, width: 0.3, height: 0.5 }
    }
  ]
};
```

---

## 3. Shop Background

### 3.1 Scene Concept: Trader's Emporium

```typescript
const SHOP_BACKGROUND = {
  name: 'traders_emporium',
  concept: 'Interior of a mystical trader\'s shop with exotic goods',

  composition: {
    foreground: {
      elements: ['counter_edge', 'hanging_trinkets', 'candle_flames'],
      blur: 0,
      parallax: 0.08
    },
    midground: {
      elements: ['display_shelves', 'treasure_piles', 'magical_artifacts'],
      blur: 0.15,
      parallax: 0.04
    },
    background: {
      elements: ['mysterious_curtains', 'window_light', 'shadow_depth'],
      blur: 0.4,
      parallax: 0.01
    }
  },

  lighting: {
    mood: 'warm_mysterious',
    primaryLight: {
      type: 'multiple_point',
      sources: [
        { position: 'upper_left', color: '#FFB347', intensity: 0.8 },
        { position: 'center', color: '#FF6B35', intensity: 0.6 }
      ]
    },
    ambientColor: '#2D1B0E',
    magicalGlow: {
      enabled: true,
      color: '#E040FB',
      spots: ['artifacts', 'premium_items']
    }
  },

  colorPalette: {
    dominant: '#3D2914', // Rich dark wood
    secondary: '#8B4513', // Warm leather
    accent: '#FFD700', // Gold treasures
    magical: '#E040FB' // Premium glow
  },

  atmosphere: {
    dustParticles: {
      enabled: true,
      count: 15,
      speed: 0.1,
      color: '#FFD700',
      opacity: 0.3
    },
    incenseSmoke: {
      enabled: true,
      wisps: 3,
      color: '#C9B896',
      opacity: 0.2
    }
  }
};
```

### 3.2 Shop Section Variants

```typescript
const SHOP_SECTION_BACKGROUNDS = {
  featured: {
    variant: 'spotlight_display',
    specialLighting: {
      spotlight: true,
      color: '#FFD700',
      rays: true
    }
  },

  cosmetics: {
    variant: 'wardrobe_area',
    elements: ['mannequins', 'mirrors', 'clothing_racks'],
    lighting: 'warm_inviting'
  },

  premium: {
    variant: 'treasure_vault',
    elements: ['gold_piles', 'gem_displays', 'ornate_cases'],
    lighting: 'magical_glow',
    specialEffects: {
      shimmer: true,
      particles: 'gold_dust'
    }
  },

  bundles: {
    variant: 'treasure_chest_display',
    elements: ['open_chests', 'spilling_loot', 'maps'],
    lighting: 'dramatic_spotlit'
  },

  currency: {
    variant: 'coin_vault',
    elements: ['coin_stacks', 'weighing_scales', 'ledger_books'],
    lighting: 'golden_ambient'
  }
};
```

### 3.3 Shop Animations

```typescript
const SHOP_ANIMATIONS = {
  candleFlicker: {
    type: 'flame_dance',
    speed: 0.5,
    intensity: 0.3
  },

  hangingItems: {
    type: 'gentle_sway',
    speed: 0.2,
    amplitude: 2
  },

  magicalArtifacts: {
    type: 'hover_rotate',
    speed: 0.1,
    glow: {
      pulse: true,
      color: '#E040FB'
    }
  },

  dustMotes: {
    type: 'floating_drift',
    speed: 0.05,
    random: true
  },

  treasureShimmer: {
    type: 'sparkle_random',
    frequency: 2.0, // Per second
    color: '#FFD700'
  }
};
```

---

## 4. Battle Pass Background

### 4.1 Scene Concept: Treasure Map Journey

```typescript
const BATTLE_PASS_BACKGROUND = {
  name: 'treasure_map_journey',
  concept: 'Stylized treasure map showing progression path',

  composition: {
    base: {
      element: 'aged_parchment',
      texture: 'weathered_paper',
      color: '#D4B896'
    },
    mapElements: {
      style: 'illustrated_cartography',
      elements: ['islands', 'sea_monsters', 'compass_rose', 'dotted_paths']
    },
    decorative: {
      elements: ['wax_seals', 'ink_splatters', 'torn_edges', 'rope_border'],
      position: 'edges'
    },
    overlay: {
      type: 'vignette',
      color: '#3D2914',
      intensity: 0.4
    }
  },

  progressionVisual: {
    path: {
      style: 'dotted_trail',
      color: '#8B4513',
      glowOnProgress: {
        color: '#FFD700',
        intensity: 0.6
      }
    },
    currentPosition: {
      marker: 'ship_icon',
      animated: true,
      trail: 'wake_effect'
    },
    milestones: {
      style: 'x_marks',
      completedColor: '#FFD700',
      lockedColor: '#666666'
    }
  },

  colorPalette: {
    dominant: '#D4B896', // Parchment
    secondary: '#8B4513', // Ink brown
    accent: '#FFD700', // Gold highlights
    sea: '#5B8FA8' // Ocean areas
  }
};
```

### 4.2 Battle Pass Animations

```typescript
const BATTLE_PASS_ANIMATIONS = {
  shipMarker: {
    type: 'gentle_bob',
    speed: 0.3,
    amplitude: 3
  },

  seaAreas: {
    type: 'subtle_wave_pattern',
    speed: 0.1,
    animated_lines: true
  },

  compassRose: {
    type: 'slow_rotation',
    speed: 0.02,
    continuous: true
  },

  goldHighlights: {
    type: 'shimmer_sweep',
    interval: 5.0,
    color: '#FFD700'
  },

  seamonsters: {
    type: 'occasional_movement',
    interval: 10.0,
    animation: 'tentacle_wave'
  }
};
```

### 4.3 Premium Track Enhancement

```typescript
const BATTLE_PASS_PREMIUM_OVERLAY = {
  freeTrack: {
    style: 'standard_parchment',
    decorations: 'minimal'
  },

  premiumTrack: {
    style: 'gilded_parchment',
    border: {
      type: 'gold_filigree',
      animated: true
    },
    background: {
      overlay: 'royal_purple_tint',
      opacity: 0.1
    },
    particles: {
      type: 'gold_sparkle',
      count: 5,
      continuous: true
    }
  }
};
```

---

## 5. Profile/Captain's Quarters Background

### 5.1 Scene Concept: Personal Cabin

```typescript
const PROFILE_BACKGROUND = {
  name: 'captains_quarters',
  concept: 'Cozy captain\'s cabin with personal effects',

  composition: {
    foreground: {
      elements: ['desk_corner', 'quill_inkpot', 'compass'],
      blur: 0
    },
    midground: {
      elements: ['bookshelf', 'world_map', 'trophy_display', 'portrait_frame'],
      blur: 0.1
    },
    background: {
      elements: ['cabin_window', 'ocean_view', 'hanging_lantern'],
      blur: 0.3
    }
  },

  lighting: {
    mood: 'warm_intimate',
    primaryLight: {
      source: 'window_daylight',
      direction: 'right',
      color: '#FFE4B5'
    },
    secondaryLight: {
      source: 'lantern',
      position: 'upper_left',
      color: '#FFB347'
    },
    ambientColor: '#4A3728'
  },

  colorPalette: {
    dominant: '#5D4037', // Rich wood
    secondary: '#8D6E63', // Leather
    accent: '#FFD700', // Brass/gold
    highlight: '#FFFAF0' // Window light
  },

  personalElements: {
    achievementWall: {
      position: 'left_side',
      dynamic: true, // Shows player's achievements
      frameStyle: 'ornate_wood'
    },
    statDisplay: {
      position: 'desk_area',
      style: 'ledger_book'
    }
  }
};
```

### 5.2 Profile Animations

```typescript
const PROFILE_ANIMATIONS = {
  lanternSway: {
    type: 'gentle_pendulum',
    speed: 0.15,
    amplitude: 2,
    lightFlicker: true
  },

  windowView: {
    type: 'ocean_parallax',
    speed: 0.05,
    waves: true
  },

  compassNeedle: {
    type: 'subtle_wobble',
    speed: 0.1
  },

  curtainMovement: {
    type: 'breeze_flutter',
    speed: 0.2,
    amplitude: 4
  },

  dustInLight: {
    type: 'floating_particles',
    count: 8,
    zone: 'window_beam',
    color: '#FFFAF0',
    opacity: 0.4
  }
};
```

---

## 6. Loading Screens

### 6.1 Loading Screen Scenes

```typescript
const LOADING_SCREENS = {
  count: 8,
  rotationType: 'random',

  scenes: [
    {
      id: 'ship_at_sea',
      concept: 'Pirate ship sailing through dramatic waves',
      mood: 'adventurous',
      elements: ['ship_bow', 'crashing_waves', 'stormy_sky', 'lightning'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'treasure_discovery',
      concept: 'Opening a glowing treasure chest on beach',
      mood: 'exciting',
      elements: ['beach', 'palm_trees', 'treasure_glow', 'gold_coins'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'sea_battle',
      concept: 'Two ships exchanging cannon fire',
      mood: 'intense',
      elements: ['ships', 'cannon_smoke', 'sunset', 'explosions'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'island_approach',
      concept: 'Ship approaching mysterious island',
      mood: 'mysterious',
      elements: ['ship_silhouette', 'island', 'fog', 'moonlight'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'tavern_celebration',
      concept: 'Pirates celebrating victory in tavern',
      mood: 'jovial',
      elements: ['tavern_interior', 'pirates', 'mugs', 'fireplace'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'kraken_encounter',
      concept: 'Ship versus giant kraken',
      mood: 'epic',
      elements: ['kraken_tentacles', 'ship', 'dramatic_sky', 'waves'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'map_planning',
      concept: 'Captain planning route on map',
      mood: 'strategic',
      elements: ['map_table', 'candles', 'compass', 'figurines'],
      tipPosition: 'bottom_center'
    },
    {
      id: 'sunset_harbor',
      concept: 'Peaceful harbor at sunset',
      mood: 'serene',
      elements: ['harbor', 'boats', 'sunset', 'seagulls'],
      tipPosition: 'bottom_center'
    }
  ]
};
```

### 6.2 Loading Screen Layout

```typescript
const LOADING_SCREEN_LAYOUT = {
  background: {
    coverage: 'full_screen',
    aspectHandling: 'cover_crop'
  },

  progressBar: {
    position: 'bottom',
    yOffset: 80, // Pixels from bottom
    width: '60%',
    height: 8,
    style: 'rope_fill',
    colors: {
      background: 'rgba(0,0,0,0.5)',
      fill: '#FFD700',
      glow: '#FF8C42'
    }
  },

  loadingText: {
    position: 'above_bar',
    yOffset: 20,
    style: {
      font: 'pirate_header',
      size: 24,
      color: '#FFD700',
      shadow: true
    },
    variants: [
      'Charting course...',
      'Hoisting the sails...',
      'Gathering the crew...',
      'Loading the cannons...',
      'Checking the map...'
    ]
  },

  tips: {
    position: 'bottom_center',
    yOffset: 140,
    maxWidth: '80%',
    style: {
      font: 'body',
      size: 16,
      color: '#FFFFFF',
      background: 'rgba(0,0,0,0.6)',
      padding: 12,
      borderRadius: 8
    }
  },

  logo: {
    position: 'top_center',
    yOffset: 60,
    size: 'medium',
    opacity: 0.9
  }
};
```

### 6.3 Loading Screen Animations

```typescript
const LOADING_SCREEN_ANIMATIONS = {
  background: {
    type: 'subtle_parallax',
    layers: 3,
    speed: 0.02
  },

  progressBar: {
    fillAnimation: 'smooth_wave',
    glowPulse: true
  },

  loadingText: {
    animation: 'fade_cycle',
    interval: 3.0
  },

  tips: {
    animation: 'fade_in_out',
    interval: 8.0
  },

  sceneSpecific: {
    ship_at_sea: {
      waves: 'animated_motion',
      lightning: 'random_flash'
    },
    treasure_discovery: {
      glow: 'pulsing_light',
      coins: 'sparkle'
    },
    kraken_encounter: {
      tentacles: 'subtle_movement'
    }
  }
};
```

---

## 7. Lobby/Matchmaking Background

### 7.1 Scene Concept: Deck Preparation

```typescript
const LOBBY_BACKGROUND = {
  name: 'ship_deck_ready',
  concept: 'Ship deck with crew preparing for battle',

  composition: {
    foreground: {
      elements: ['cannon_row', 'weapon_racks', 'supplies'],
      blur: 0
    },
    midground: {
      elements: ['crew_silhouettes', 'mast_base', 'rigging'],
      blur: 0.15
    },
    background: {
      elements: ['other_ships', 'island_destination', 'sky'],
      blur: 0.4
    }
  },

  lighting: {
    mood: 'anticipation',
    timeOfDay: 'early_morning',
    primaryColor: '#FFE4B5',
    ambientColor: '#6B8BA4'
  },

  matchmakingIntegration: {
    playerCount: {
      display: 'crew_gathering',
      visualFeedback: 'more_silhouettes'
    },
    readyState: {
      visual: 'increased_activity',
      animations: 'faster'
    },
    countdown: {
      visual: 'anchor_raising',
      intensity: 'building'
    }
  }
};
```

### 7.2 Lobby Animations

```typescript
const LOBBY_ANIMATIONS = {
  crewActivity: {
    type: 'ambient_movement',
    intensity: 'based_on_player_count'
  },

  flagWaving: {
    type: 'wind_flutter',
    speed: 0.4
  },

  waterReflection: {
    type: 'ripple_effect',
    speed: 0.2
  },

  destinationIsland: {
    type: 'slow_approach',
    speed: 0.01
  },

  matchFound: {
    sequence: [
      { action: 'screen_flash', color: '#FFD700' },
      { action: 'anchor_raise_animation' },
      { action: 'ship_movement_start' }
    ]
  }
};
```

---

## 8. Settings/Options Background

### 8.1 Scene Concept: Navigation Table

```typescript
const SETTINGS_BACKGROUND = {
  name: 'navigation_table',
  concept: 'Close-up of captain\'s navigation table with instruments',

  composition: {
    surface: {
      type: 'wooden_table',
      texture: 'aged_oak',
      color: '#5D4037'
    },
    elements: {
      scattered: ['maps', 'compass', 'sextant', 'spyglass', 'candle'],
      arrangement: 'natural_clutter'
    },
    lighting: {
      type: 'overhead_warm',
      color: '#FFB347',
      softShadows: true
    }
  },

  colorPalette: {
    dominant: '#5D4037', // Wood
    secondary: '#D4B896', // Paper/parchment
    accent: '#B8860B', // Brass instruments
    highlight: '#FFD700' // Candlelight
  },

  depthOfField: {
    focus: 'center_table',
    backgroundBlur: 0.6,
    edgeBlur: 0.3
  }
};
```

---

## 9. Technical Specifications

### 9.1 Resolution and Format

```typescript
const BACKGROUND_TECH_SPECS = {
  resolution: {
    base: { width: 2048, height: 2048 },
    layers: { width: 2048, height: 1024 }, // For parallax layers
    mobile_optimized: { width: 1024, height: 1024 }
  },

  format: {
    source: 'PSD', // Layered source files
    export: 'PNG', // For transparency support
    compressed: 'ASTC_4x4', // Mobile texture compression
    fallback: 'ETC2' // Older device support
  },

  aspectRatio: {
    design: '16:9', // Base design
    safeArea: '18:9', // Extended for tall screens
    ultrawide: '21:9' // Maximum supported
  },

  memoryBudget: {
    perBackground: '4MB', // Compressed
    totalLoaded: '16MB', // Maximum at once
    streamingSupport: true
  }
};
```

### 9.2 Parallax Layer System

```typescript
const PARALLAX_SYSTEM = {
  maxLayers: 4,

  layerConfig: [
    {
      name: 'sky',
      depth: 0,
      parallaxFactor: 0,
      scrollSpeed: 0.01
    },
    {
      name: 'far_background',
      depth: 1,
      parallaxFactor: 0.02,
      scrollSpeed: 0.02
    },
    {
      name: 'mid_background',
      depth: 2,
      parallaxFactor: 0.05,
      scrollSpeed: 0.03
    },
    {
      name: 'foreground',
      depth: 3,
      parallaxFactor: 0.1,
      scrollSpeed: 0
    }
  ],

  inputResponse: {
    gyroscope: {
      enabled: true,
      sensitivity: 0.3,
      smoothing: 0.8
    },
    touch: {
      enabled: true,
      sensitivity: 0.5
    }
  }
};
```

### 9.3 Animation Performance

```typescript
const ANIMATION_PERFORMANCE = {
  targetFrameRate: 30, // Background animations
  maxAnimatedElements: 10,
  maxParticles: 50,

  qualityTiers: {
    low: {
      animations: false,
      parallax: false,
      particles: false,
      staticFallback: true
    },
    medium: {
      animations: true,
      animationCount: 5,
      parallax: true,
      parallaxLayers: 2,
      particles: false
    },
    high: {
      animations: true,
      animationCount: 10,
      parallax: true,
      parallaxLayers: 4,
      particles: true,
      particleCount: 50
    }
  },

  batteryMode: {
    reduceAnimations: true,
    disableParticles: true,
    staticBackground: false
  }
};
```

---

## 10. UI Overlay System

### 10.1 Darkening Overlays

```typescript
const UI_OVERLAYS = {
  gradientOverlay: {
    type: 'radial_vignette',
    centerOpacity: 0,
    edgeOpacity: 0.6,
    color: '#000000'
  },

  bottomBar: {
    type: 'linear_gradient',
    direction: 'bottom_to_top',
    startOpacity: 0.8,
    endOpacity: 0,
    height: '20%',
    color: '#1A1A2E'
  },

  contentArea: {
    type: 'solid_overlay',
    opacity: 0.3,
    color: '#000000',
    blur: 4 // Pixels
  },

  modalBackground: {
    type: 'full_screen',
    opacity: 0.7,
    color: '#000000',
    blur: 8
  }
};
```

### 10.2 Blur Effects

```typescript
const BLUR_EFFECTS = {
  backgroundBlur: {
    type: 'gaussian',
    radius: 8,
    usage: ['modals', 'popups']
  },

  depthBlur: {
    type: 'depth_of_field',
    focusDistance: 'mid',
    blurAmount: 4
  },

  qualitySettings: {
    low: {
      blurEnabled: false,
      darkenFallback: true
    },
    medium: {
      blurEnabled: true,
      blurRadius: 4
    },
    high: {
      blurEnabled: true,
      blurRadius: 8
    }
  }
};
```

---

## 11. Screen Transitions

### 11.1 Transition Effects

```typescript
const SCREEN_TRANSITIONS = {
  menuToMenu: {
    type: 'crossfade',
    duration: 0.3,
    easing: 'ease_in_out'
  },

  menuToGame: {
    type: 'anchor_drop',
    duration: 0.5,
    elements: ['anchor_chain', 'splash'],
    sound: 'anchor_splash'
  },

  shopEntry: {
    type: 'door_open',
    duration: 0.4,
    elements: ['wooden_doors', 'light_reveal']
  },

  battlePassEntry: {
    type: 'map_unroll',
    duration: 0.5,
    elements: ['scroll_unfurl', 'compass_spin']
  },

  profileEntry: {
    type: 'cabin_door',
    duration: 0.4,
    elements: ['door_swing', 'lantern_light']
  }
};
```

### 11.2 Loading Transitions

```typescript
const LOADING_TRANSITIONS = {
  toLoading: {
    type: 'fade_to_black',
    duration: 0.3
  },

  fromLoading: {
    type: 'iris_open',
    duration: 0.5,
    shape: 'compass_rose'
  },

  matchStart: {
    type: 'dramatic_zoom',
    duration: 0.8,
    target: 'island_destination',
    sound: 'battle_horn'
  }
};
```

---

## 12. Implementation Checklist

### 12.1 Asset Delivery

| Screen | Static BG | Layers | Animations | Priority |
|--------|-----------|--------|------------|----------|
| Main Menu | ✓ | 4 | 6 | Critical |
| Shop | ✓ | 3 | 5 | Critical |
| Battle Pass | ✓ | 2 | 4 | High |
| Profile | ✓ | 3 | 5 | High |
| Lobby | ✓ | 3 | 4 | High |
| Settings | ✓ | 1 | 2 | Medium |
| Loading (8) | ✓ | 3 each | 2 each | High |

### 12.2 Quality Testing

| Test | Criteria |
|------|----------|
| UI Readability | All text legible over backgrounds |
| Animation Performance | 30 FPS minimum |
| Memory Usage | Under 16MB total |
| Load Time | Under 500ms per screen |
| Battery Impact | Minimal drain in menus |
| Device Coverage | Works on min spec devices |

### 12.3 File Naming Convention

```
bg_[screen]_[layer]_[variant].[format]

Examples:
bg_mainmenu_sky_sunset.png
bg_mainmenu_midground_harbor.png
bg_shop_foreground_counter.png
bg_loading_01_treasure.png
```

---

## Appendix A: Color Reference

### Background Color Palettes

| Screen | Dominant | Secondary | Accent | Highlight |
|--------|----------|-----------|--------|-----------|
| Main Menu | #1E3A5F | #FFB366 | #8B4513 | #FFD700 |
| Shop | #3D2914 | #8B4513 | #FFD700 | #E040FB |
| Battle Pass | #D4B896 | #8B4513 | #FFD700 | #5B8FA8 |
| Profile | #5D4037 | #8D6E63 | #FFD700 | #FFFAF0 |
| Lobby | #6B8BA4 | #FFE4B5 | #8B4513 | #FFD700 |
| Settings | #5D4037 | #D4B896 | #B8860B | #FFD700 |

---

## Appendix B: Reference Art

### Style References
- Sea of Thieves menu screens (lighting, atmosphere)
- Hearthstone tavern aesthetic (warm, inviting interiors)
- Fortnite lobby transitions (dynamic, exciting)
- Legends of Runeterra card backs (illustrated style)

### Mood References
- Golden hour Caribbean photography
- Classic pirate movie poster art
- Illustrated treasure maps
- Nautical chart illustrations
