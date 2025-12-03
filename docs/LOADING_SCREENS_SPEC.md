# Loading Screens Specification

## Overview

This document specifies the Loading Screen system for Plunderstorm Mobile, designed to maintain player engagement during asset loading while providing useful information and maintaining the pirate theme aesthetic.

## Loading Screen Types

### Screen Categories

| Type | Context | Duration Target | Content Priority |
|------|---------|-----------------|------------------|
| Initial Load | App launch | < 5 seconds | Branding, progress |
| Menu Load | Entering main menu | < 3 seconds | Quick tips |
| Match Load | Entering match | < 10 seconds | Tips, map preview, player info |
| Mode Transition | Switching modes | < 2 seconds | Minimal overlay |
| Asset Download | Downloading content | Variable | Progress, size info |

## Initial Load Screen

### App Launch Screen

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                                                                         │
│                                                                         │
│                    [Game Logo - Animated]                              │
│                    PLUNDERSTORM MOBILE                                  │
│                                                                         │
│                    [Ship sailing animation]                            │
│                                                                         │
│                                                                         │
│                    [████████████████░░░░░░]                            │
│                    Loading assets... 73%                                │
│                                                                         │
│                                                                         │
│                    Version 1.2.3                                        │
│                    © 2024 Plunderstorm Studios                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Initial Load Configuration

```typescript
interface InitialLoadConfig {
  // Visual
  logoAnimation: 'wave_in';
  backgroundImage: 'bg_ocean_sunset';
  ambientAnimation: 'ship_sailing';

  // Progress
  showProgressBar: true;
  showProgressText: true;
  showPhaseText: true;

  // Branding
  showVersion: true;
  showCopyright: true;

  // Performance
  preloadCriticalAssets: true;
  parallelLoading: true;
}

interface LoadPhase {
  id: string;
  name: string;
  weight: number;         // Percentage of total
  assets: string[];
}

const INITIAL_LOAD_PHASES: LoadPhase[] = [
  {
    id: 'core',
    name: 'Initializing...',
    weight: 10,
    assets: ['config', 'localization', 'audio_engine']
  },
  {
    id: 'connect',
    name: 'Connecting...',
    weight: 15,
    assets: ['auth', 'server_connection']
  },
  {
    id: 'profile',
    name: 'Loading profile...',
    weight: 20,
    assets: ['player_data', 'inventory', 'progression']
  },
  {
    id: 'ui',
    name: 'Loading interface...',
    weight: 25,
    assets: ['ui_sprites', 'fonts', 'icons']
  },
  {
    id: 'menu',
    name: 'Preparing menu...',
    weight: 30,
    assets: ['character_model', 'menu_background', 'effects']
  }
];
```

## Match Loading Screen

### Match Load Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  MODE: Solo Battle Royale                      60 Players               │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │                                                                   │ │
│  │                    [Map Preview Image]                            │ │
│  │                    TORTUGA BAY                                    │ │
│  │                                                                   │ │
│  │                    "The legendary pirate haven..."               │ │
│  │                                                                   │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │                                                                   │ │
│  │  💡 TIP: Use cover to avoid enemy abilities!                     │ │
│  │                                                                   │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ [Your Character]   CaptainJack    Level 42                      │   │
│  │  Loadout: Cutlass Slash, Fire Bomb, Healing Tide, Grappling Hook│   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  [█████████████████████████░░░░░░░░░░░]  Loading match... 72%          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Match Load Configuration

```typescript
interface MatchLoadScreenConfig {
  // Header
  showMode: true;
  showPlayerCount: true;

  // Map preview
  showMapPreview: true;
  mapPreviewSize: 'large';
  showMapName: true;
  showMapDescription: true;

  // Tips
  showTips: true;
  tipRotationInterval: 5000;  // ms
  tipCategories: ['combat', 'abilities', 'strategy', 'controls'];

  // Player info
  showPlayerCard: true;
  showLoadout: true;
  showCosmetic: true;

  // Progress
  progressBarStyle: 'detailed';
  showLoadingPhase: true;
}
```

### Map Preview Component

```typescript
interface MapPreview {
  mapId: string;
  name: string;
  description: string;
  previewImage: string;
  keyFeatures: string[];
  playerCapacity: number;
}

const MAP_PREVIEWS: Record<string, MapPreview> = {
  'tortuga_bay': {
    mapId: 'tortuga_bay',
    name: 'Tortuga Bay',
    description: 'The legendary pirate haven with sandy beaches, dense jungle, and a bustling port town.',
    previewImage: 'map_tortuga_preview_hd.jpg',
    keyFeatures: ['Central Port Town', 'Jungle Ruins', 'Coastal Caves'],
    playerCapacity: 60
  },
  'skull_island': {
    mapId: 'skull_island',
    name: 'Skull Island',
    description: 'A mysterious volcanic island with ancient temples and treacherous terrain.',
    previewImage: 'map_skull_preview_hd.jpg',
    keyFeatures: ['Volcano Center', 'Temple Ruins', 'Underground Tunnels'],
    playerCapacity: 60
  }
};
```

## Gameplay Tips System

### Tip Categories

```typescript
interface GameplayTip {
  id: string;
  category: TipCategory;
  text: string;
  icon: string;
  priority: number;           // Higher = more likely to show
  minLevel?: number;          // Only show above this level
  maxLevel?: number;          // Only show below this level
  newPlayerOnly?: boolean;
  condition?: TipCondition;
}

enum TipCategory {
  COMBAT = 'combat',
  ABILITIES = 'abilities',
  STRATEGY = 'strategy',
  CONTROLS = 'controls',
  ADVANCED = 'advanced',
  SOCIAL = 'social'
}

interface TipCondition {
  type: 'first_time' | 'recent_death' | 'mode_specific' | 'ability_equipped';
  value?: string;
}

const GAMEPLAY_TIPS: GameplayTip[] = [
  // Combat tips
  {
    id: 'cover_tip',
    category: TipCategory.COMBAT,
    text: 'Use cover to avoid enemy abilities! Walls and rocks block most projectiles.',
    icon: 'icon_tip_cover',
    priority: 10
  },
  {
    id: 'headshots_tip',
    category: TipCategory.COMBAT,
    text: 'Aim for critical hits! Hitting enemies from behind deals bonus damage.',
    icon: 'icon_tip_damage',
    priority: 8
  },

  // Ability tips
  {
    id: 'ability_combo_tip',
    category: TipCategory.ABILITIES,
    text: 'Combine abilities for powerful combos! Try slowing enemies before using damage abilities.',
    icon: 'icon_tip_combo',
    priority: 9
  },
  {
    id: 'upgrade_tip',
    category: TipCategory.ABILITIES,
    text: 'Picking up duplicate abilities upgrades them! Higher ranks deal more damage.',
    icon: 'icon_tip_upgrade',
    priority: 10
  },

  // Strategy tips
  {
    id: 'loot_early_tip',
    category: TipCategory.STRATEGY,
    text: 'Loot early, fight later! Gear up before engaging enemies.',
    icon: 'icon_tip_loot',
    priority: 10,
    maxLevel: 10
  },
  {
    id: 'storm_tip',
    category: TipCategory.STRATEGY,
    text: 'Watch the storm! Stay inside the safe zone or take damage over time.',
    icon: 'icon_tip_storm',
    priority: 10,
    newPlayerOnly: true
  },

  // Controls tips
  {
    id: 'aim_assist_tip',
    category: TipCategory.CONTROLS,
    text: 'Enable aim assist in settings for easier targeting on mobile!',
    icon: 'icon_tip_aim',
    priority: 7,
    maxLevel: 5
  },

  // Advanced tips
  {
    id: 'audio_cues_tip',
    category: TipCategory.ADVANCED,
    text: 'Listen for footsteps and ability sounds to track nearby enemies!',
    icon: 'icon_tip_audio',
    priority: 6,
    minLevel: 10
  }
];
```

### Tip Selection Algorithm

```typescript
function selectTipsForPlayer(player: Player, count: number = 5): GameplayTip[] {
  const eligibleTips = GAMEPLAY_TIPS.filter(tip => {
    // Level requirements
    if (tip.minLevel && player.level < tip.minLevel) return false;
    if (tip.maxLevel && player.level > tip.maxLevel) return false;

    // New player only
    if (tip.newPlayerOnly && player.matchesPlayed > 10) return false;

    // Conditional tips
    if (tip.condition) {
      if (!evaluateTipCondition(tip.condition, player)) return false;
    }

    return true;
  });

  // Sort by priority with randomization
  const weighted = eligibleTips.map(tip => ({
    tip,
    weight: tip.priority + Math.random() * 5
  }));

  weighted.sort((a, b) => b.weight - a.weight);

  return weighted.slice(0, count).map(w => w.tip);
}

function getRotatingTip(tips: GameplayTip[], intervalMs: number): GameplayTip {
  const index = Math.floor(Date.now() / intervalMs) % tips.length;
  return tips[index];
}
```

### Tip Display

```
Standard Tip Display:
┌─────────────────────────────────────────────────────────────────────────┐
│  💡 TIP: Use cover to avoid enemy abilities! Walls and rocks block    │
│         most projectiles.                                              │
└─────────────────────────────────────────────────────────────────────────┘

Animated Tip Carousel:
┌─────────────────────────────────────────────────────────────────────────┐
│  💡 TIP: [Fade in animation]                                          │
│                                                                         │
│  Watch the storm! Stay inside the safe zone or take damage over time. │
│                                                                         │
│  ● ○ ○ ○ ○                                           [5 second timer] │
└─────────────────────────────────────────────────────────────────────────┘
```

## Player Card Component

### Player Info Display

```typescript
interface PlayerLoadCard {
  // Identity
  playerId: string;
  displayName: string;
  avatar: string;
  level: number;

  // Rank (if ranked mode)
  rank?: {
    tier: string;
    division: number;
    icon: string;
  };

  // Loadout
  loadout: {
    abilities: AbilitySlot[];
  };

  // Cosmetics
  cosmetics: {
    character: string;
    skin: string;
    weapon: string;
  };

  // Stats preview (optional)
  stats?: {
    matchesPlayed: number;
    wins: number;
    killsTotal: number;
  };
}

interface AbilitySlot {
  slotIndex: number;
  abilityId: string;
  abilityName: string;
  abilityIcon: string;
}
```

### Player Card Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  [Character Preview]                                                    │
│  ┌─────────────┐                                                       │
│  │             │   CaptainJack                                         │
│  │  [3D Model] │   Level 42  ★★★                                      │
│  │             │                                                       │
│  │             │   LOADOUT:                                            │
│  └─────────────┘   [⚔️] [🔥] [💧] [🪝]                                  │
│                    Cutlass  Fire   Heal  Grapple                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Progress Indicators

### Progress Bar Styles

```typescript
interface ProgressBarConfig {
  style: 'simple' | 'detailed' | 'segmented';
  showPercentage: boolean;
  showPhaseText: boolean;
  animateProgress: boolean;
  color: string;
  backgroundColor: string;
}

// Simple style
// [████████████░░░░░░░░]  75%

// Detailed style
// [████████████░░░░░░░░]  Loading textures... 75%

// Segmented style
// [████|████|████|░░░░|░░░░]
//  Init  Net  Assets  Map  Spawn
```

### Progress Implementation

```typescript
interface LoadProgress {
  overall: number;            // 0-100
  phase: string;
  phaseProgress: number;      // 0-100 within phase
  phases: PhaseProgress[];
}

interface PhaseProgress {
  id: string;
  name: string;
  status: 'pending' | 'loading' | 'complete' | 'error';
  progress: number;
}

class LoadingProgressManager {
  private progress: LoadProgress;
  private listeners: ((progress: LoadProgress) => void)[] = [];

  updateProgress(phase: string, progress: number): void {
    const phaseConfig = LOAD_PHASES.find(p => p.id === phase);
    if (!phaseConfig) return;

    // Update phase progress
    const phaseIndex = this.progress.phases.findIndex(p => p.id === phase);
    if (phaseIndex >= 0) {
      this.progress.phases[phaseIndex].progress = progress;
      this.progress.phases[phaseIndex].status = progress >= 100 ? 'complete' : 'loading';
    }

    // Calculate overall progress
    this.progress.overall = this.calculateOverallProgress();
    this.progress.phase = phaseConfig.name;
    this.progress.phaseProgress = progress;

    // Notify listeners
    this.notifyListeners();
  }

  private calculateOverallProgress(): number {
    let total = 0;
    for (const phase of this.progress.phases) {
      const config = LOAD_PHASES.find(p => p.id === phase.id);
      if (config) {
        total += (phase.progress / 100) * config.weight;
      }
    }
    return Math.round(total);
  }
}
```

## Loading Animations

### Animation Types

```typescript
interface LoadingAnimation {
  type: 'spinner' | 'ship' | 'compass' | 'waves' | 'treasure';
  config: AnimationConfig;
}

interface AnimationConfig {
  size: number;
  speed: number;
  color: string;
  loop: boolean;
}

const LOADING_ANIMATIONS: Record<string, LoadingAnimation> = {
  'spinner': {
    type: 'spinner',
    config: { size: 48, speed: 1.0, color: '#FFD700', loop: true }
  },
  'ship_sailing': {
    type: 'ship',
    config: { size: 120, speed: 0.5, color: '#FFFFFF', loop: true }
  },
  'compass_spin': {
    type: 'compass',
    config: { size: 64, speed: 0.3, color: '#FFD700', loop: true }
  },
  'wave_motion': {
    type: 'waves',
    config: { size: 200, speed: 0.8, color: '#4169E1', loop: true }
  }
};
```

### Themed Animations

```
Ship Sailing Animation:
   ⛵
  ~~~~

Compass Spinning:
   [Rotating compass needle]

Treasure Chest:
   [Opening/closing chest with sparkles]

Wave Motion:
   ≋≋≋≋≋≋≋≋≋≋≋≋≋≋≋≋≋
   (Animated sine wave)
```

## Error Handling

### Load Error States

```typescript
enum LoadErrorType {
  NETWORK_ERROR = 'network_error',
  SERVER_ERROR = 'server_error',
  ASSET_ERROR = 'asset_error',
  TIMEOUT = 'timeout',
  STORAGE_FULL = 'storage_full'
}

interface LoadError {
  type: LoadErrorType;
  phase: string;
  message: string;
  retryable: boolean;
}

const ERROR_DISPLAYS: Record<LoadErrorType, ErrorConfig> = {
  [LoadErrorType.NETWORK_ERROR]: {
    title: 'Connection Error',
    message: 'Unable to connect. Check your internet connection.',
    icon: 'icon_no_connection',
    actions: ['retry', 'offline_mode']
  },
  [LoadErrorType.SERVER_ERROR]: {
    title: 'Server Error',
    message: 'Our servers are having trouble. Please try again.',
    icon: 'icon_server_error',
    actions: ['retry', 'report']
  },
  [LoadErrorType.ASSET_ERROR]: {
    title: 'Download Error',
    message: 'Failed to download game assets. Please try again.',
    icon: 'icon_download_error',
    actions: ['retry', 'clear_cache']
  },
  [LoadErrorType.TIMEOUT]: {
    title: 'Loading Timeout',
    message: 'Loading took too long. Please try again.',
    icon: 'icon_timeout',
    actions: ['retry']
  },
  [LoadErrorType.STORAGE_FULL]: {
    title: 'Storage Full',
    message: 'Not enough storage space. Free up space and try again.',
    icon: 'icon_storage',
    actions: ['open_settings']
  }
};
```

### Error Screen

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                                                                         │
│                    [Error Icon]                                        │
│                                                                         │
│                    Connection Error                                     │
│                                                                         │
│                    Unable to connect to servers.                       │
│                    Check your internet connection.                     │
│                                                                         │
│                                                                         │
│                    [Retry]    [Settings]                               │
│                                                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Background Loading

### Preload Strategy

```typescript
interface PreloadConfig {
  // Initial load
  criticalAssets: string[];     // Block until loaded
  priorityAssets: string[];     // Load immediately after
  deferredAssets: string[];     // Load in background

  // Match load
  matchCritical: string[];      // Required for spawn
  matchPriority: string[];      // Load before game starts
  matchDeferred: string[];      // Stream during match
}

const PRELOAD_CONFIG: PreloadConfig = {
  criticalAssets: [
    'ui_core',
    'fonts',
    'config'
  ],
  priorityAssets: [
    'menu_ui',
    'character_base',
    'audio_sfx'
  ],
  deferredAssets: [
    'cosmetics_all',
    'voice_lines',
    'high_res_textures'
  ],
  matchCritical: [
    'map_geometry',
    'player_model',
    'ability_vfx',
    'hud_elements'
  ],
  matchPriority: [
    'map_textures',
    'ambient_audio',
    'enemy_models'
  ],
  matchDeferred: [
    'high_detail_textures',
    'distant_scenery',
    'optional_vfx'
  ]
};
```

### Streaming During Match

```typescript
interface StreamingConfig {
  enabled: boolean;
  bandwidthLimit: number;       // KB/s
  priorityDistance: number;     // Load assets within this range first
  unloadDistance: number;       // Unload assets beyond this range
}

const STREAMING_CONFIG: StreamingConfig = {
  enabled: true,
  bandwidthLimit: 500,          // 500 KB/s during gameplay
  priorityDistance: 100,        // meters
  unloadDistance: 300           // meters
};
```

## Performance Optimization

### Load Time Targets

| Screen | Target | Maximum |
|--------|--------|---------|
| Initial Load | 3s | 5s |
| Menu Load | 2s | 3s |
| Match Load | 6s | 10s |
| Mode Transition | 1s | 2s |

### Optimization Techniques

```typescript
interface LoadOptimization {
  // Parallel loading
  parallelLoads: number;        // Max concurrent downloads

  // Caching
  cacheEnabled: boolean;
  cacheSize: number;            // MB

  // Compression
  useCompressedAssets: boolean;
  textureCompression: 'ASTC' | 'ETC2' | 'PVRTC';

  // Progressive loading
  progressiveTextures: boolean;
  lodStreaming: boolean;

  // Prefetching
  prefetchOnMenuHover: boolean;
  prefetchPopularModes: boolean;
}

const LOAD_OPTIMIZATION: LoadOptimization = {
  parallelLoads: 4,
  cacheEnabled: true,
  cacheSize: 500,               // 500 MB cache
  useCompressedAssets: true,
  textureCompression: 'ASTC',
  progressiveTextures: true,
  lodStreaming: true,
  prefetchOnMenuHover: true,
  prefetchPopularModes: true
};
```

## Transition Animations

### Screen Transitions

```typescript
interface TransitionAnimation {
  type: 'fade' | 'slide' | 'zoom' | 'pirate_wipe';
  duration: number;
  easing: string;
}

const TRANSITIONS: Record<string, TransitionAnimation> = {
  'menu_to_match': {
    type: 'pirate_wipe',        // Custom pirate-themed wipe
    duration: 800,
    easing: 'ease-in-out'
  },
  'match_to_results': {
    type: 'fade',
    duration: 500,
    easing: 'ease-out'
  },
  'tab_switch': {
    type: 'slide',
    duration: 250,
    easing: 'ease-out'
  }
};

// Pirate wipe animation
// Ship sails across screen, revealing new content behind
```

## Analytics Events

```typescript
// Load timing
analytics.track('load_complete', {
  screenType: string,
  duration: number,
  phases: PhaseTimings[],
  cached: boolean,
  networkSpeed: string
});

// Load errors
analytics.track('load_error', {
  screenType: string,
  errorType: string,
  phase: string,
  retried: boolean,
  resolved: boolean
});

// Tip engagement
analytics.track('tip_displayed', {
  tipId: string,
  category: string,
  duration: number
});
```

## Testing Requirements

### Functional Tests

- [ ] Initial load completes within target time
- [ ] Progress bar updates smoothly
- [ ] Tips rotate correctly
- [ ] Map preview displays correctly
- [ ] Player card shows correct loadout
- [ ] Error states display properly
- [ ] Retry functionality works

### Performance Tests

- [ ] Load times meet targets on low-end devices
- [ ] Memory usage during load acceptable
- [ ] No frame drops during animations
- [ ] Background loading doesn't affect gameplay

### Edge Cases

- [ ] Very slow network handled
- [ ] Interrupted downloads resume
- [ ] Cache invalidation works
- [ ] Large asset downloads show progress

## Success Metrics

| Metric | Target |
|--------|--------|
| Initial load time | < 5 seconds |
| Match load time | < 10 seconds |
| Load timeout rate | < 1% |
| Tip engagement (stayed on screen) | 3+ seconds avg |

## Dependencies

- **BACK-003**: Asset Delivery System
- **UX-022**: Device Performance Detection
- **GAME-024**: Practice Mode (tutorial tips)
