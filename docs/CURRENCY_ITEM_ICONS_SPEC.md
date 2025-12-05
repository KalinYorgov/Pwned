# Currency and Item Icons Specification

## Document Information
- **Task ID:** ART-019
- **Priority:** P1
- **Complexity:** S
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines all currency, item, and collectible icons for Plunderstorm Mobile's economy and inventory systems. These icons must be instantly recognizable, visually distinct from each other, and work effectively at various sizes across the UI.

### 1.2 Design Goals
- **Instant Recognition:** Each icon identifiable at a glance
- **Clear Differentiation:** Currencies clearly distinct from each other
- **Premium Feel:** Premium currency looks valuable and special
- **Size Flexibility:** Readable from 16px to 128px
- **Theme Consistency:** Pirate aesthetic throughout

### 1.3 Art Style Alignment
Following the Visual Style Guide:
- Bold, saturated colors with strong outlines
- Stylized proportions with slight exaggeration
- Gold, teal, and purple as key accent colors
- Clean silhouettes for mobile readability

---

## 2. Currency Icons

### 2.1 Gold (Soft Currency)

```typescript
const GOLD_ICON = {
  name: 'gold_coin',
  purpose: 'Primary soft currency for in-game purchases',

  design: {
    shape: 'circular_coin',
    primaryColor: '#FFD700', // Gold
    secondaryColor: '#B8860B', // Dark gold for shading
    accentColor: '#FFF8DC', // Highlight

    elements: {
      face: {
        symbol: 'skull_and_crossbones',
        style: 'embossed',
        color: '#B8860B'
      },
      edge: {
        style: 'ridged',
        color: '#DAA520'
      },
      shine: {
        type: 'arc_highlight',
        position: 'top_left',
        color: '#FFFFFF',
        opacity: 0.6
      }
    },

    outline: {
      color: '#8B6914',
      width: 2 // Pixels at base size
    }
  },

  sizes: {
    tiny: { size: 16, detail: 'minimal' },
    small: { size: 24, detail: 'reduced' },
    medium: { size: 32, detail: 'standard' },
    large: { size: 48, detail: 'full' },
    xlarge: { size: 64, detail: 'full' },
    display: { size: 128, detail: 'enhanced' }
  },

  variants: {
    standard: 'single_coin',
    stack_small: 'three_coins_stacked',
    stack_large: 'pile_of_coins',
    animated: 'spinning_coin' // For rewards
  },

  states: {
    normal: { saturation: 1.0, brightness: 1.0 },
    insufficient: { saturation: 0.3, brightness: 0.7 },
    highlighted: { saturation: 1.0, brightness: 1.2, glow: true }
  }
};
```

### 2.2 Doubloons (Premium Currency)

```typescript
const DOUBLOONS_ICON = {
  name: 'doubloon',
  purpose: 'Premium currency purchased with real money',

  design: {
    shape: 'octagonal_coin', // Distinct from round gold
    primaryColor: '#9C27B0', // Purple
    secondaryColor: '#6A1B9A', // Deep purple
    accentColor: '#E1BEE7', // Light purple highlight

    elements: {
      face: {
        symbol: 'anchor_with_gem',
        style: 'inlaid',
        gemColor: '#00BCD4', // Cyan gem
        metalColor: '#FFD700' // Gold inlay
      },
      edge: {
        style: 'beveled',
        color: '#7B1FA2'
      },
      shine: {
        type: 'diamond_sparkle',
        positions: ['top_left', 'center', 'bottom_right'],
        color: '#FFFFFF',
        animated: true
      },
      aura: {
        enabled: true,
        color: '#E040FB',
        opacity: 0.3,
        pulse: true
      }
    },

    outline: {
      color: '#4A148C',
      width: 2
    }
  },

  sizes: {
    tiny: { size: 16, detail: 'minimal', aura: false },
    small: { size: 24, detail: 'reduced', aura: false },
    medium: { size: 32, detail: 'standard', aura: true },
    large: { size: 48, detail: 'full', aura: true },
    xlarge: { size: 64, detail: 'full', aura: true },
    display: { size: 128, detail: 'enhanced', aura: true }
  },

  variants: {
    standard: 'single_doubloon',
    stack: 'three_doubloons',
    bundle_small: 'pouch_of_doubloons',
    bundle_medium: 'chest_of_doubloons',
    bundle_large: 'treasure_hoard'
  },

  premiumIndicators: {
    shimmer: {
      enabled: true,
      interval: 3.0, // Seconds
      color: '#FFFFFF'
    },
    sparkles: {
      enabled: true,
      count: 3,
      continuous: true
    }
  }
};
```

### 2.3 Battle Pass XP

```typescript
const BATTLE_PASS_XP_ICON = {
  name: 'battle_pass_xp',
  purpose: 'Experience points for Battle Pass progression',

  design: {
    shape: 'star_badge',
    primaryColor: '#00BCD4', // Cyan
    secondaryColor: '#0097A7', // Dark cyan
    accentColor: '#B2EBF2', // Light cyan

    elements: {
      center: {
        symbol: 'compass_rose',
        color: '#FFFFFF'
      },
      rays: {
        count: 8,
        style: 'pointed',
        gradient: true
      },
      border: {
        style: 'nautical_rope',
        color: '#FFD700'
      }
    },

    outline: {
      color: '#006064',
      width: 2
    }
  },

  sizes: {
    tiny: { size: 16, rays: 4 },
    small: { size: 24, rays: 4 },
    medium: { size: 32, rays: 8 },
    large: { size: 48, rays: 8 },
    xlarge: { size: 64, rays: 8 }
  },

  variants: {
    standard: 'single_star',
    bonus: 'star_with_plus', // Bonus XP
    multiplier: 'star_with_x2' // Double XP
  }
};
```

### 2.4 Player XP / Account Level

```typescript
const PLAYER_XP_ICON = {
  name: 'player_xp',
  purpose: 'Account-level experience points',

  design: {
    shape: 'shield_badge',
    primaryColor: '#4CAF50', // Green
    secondaryColor: '#2E7D32', // Dark green
    accentColor: '#A5D6A7', // Light green

    elements: {
      center: {
        symbol: 'rising_anchor',
        color: '#FFFFFF'
      },
      background: {
        pattern: 'wave_lines',
        opacity: 0.2
      },
      banner: {
        position: 'bottom',
        style: 'ribbon',
        color: '#FFD700'
      }
    },

    outline: {
      color: '#1B5E20',
      width: 2
    }
  },

  variants: {
    xp_orb: 'floating_orb', // In-match pickup
    xp_gained: 'orb_with_plus',
    level_badge: 'shield_with_number'
  }
};
```

---

## 3. Loot Chest Icons

### 3.1 Chest Tier Icons

```typescript
const CHEST_ICONS = {
  common: {
    name: 'common_chest',
    design: {
      baseColor: '#8D6E63', // Brown wood
      metalColor: '#9E9E9E', // Grey iron
      style: 'simple_wooden',
      lock: 'iron_latch',
      glow: false
    },
    indicator: {
      color: '#9E9E9E',
      particles: false
    }
  },

  uncommon: {
    name: 'uncommon_chest',
    design: {
      baseColor: '#6D4C41', // Dark wood
      metalColor: '#4CAF50', // Green brass
      style: 'reinforced_wooden',
      lock: 'brass_lock',
      gems: false,
      glow: {
        enabled: true,
        color: '#4CAF50',
        intensity: 0.3
      }
    },
    indicator: {
      color: '#4CAF50',
      particles: true,
      particleCount: 2
    }
  },

  rare: {
    name: 'rare_chest',
    design: {
      baseColor: '#3E2723', // Rich dark wood
      metalColor: '#2196F3', // Blue
      style: 'ornate_ironbound',
      lock: 'ornate_lock',
      gems: {
        enabled: true,
        count: 2,
        color: '#2196F3'
      },
      glow: {
        enabled: true,
        color: '#2196F3',
        intensity: 0.5
      }
    },
    indicator: {
      color: '#2196F3',
      particles: true,
      particleCount: 4
    }
  },

  epic: {
    name: 'epic_chest',
    design: {
      baseColor: '#1A1A2E', // Dark purple wood
      metalColor: '#9C27B0', // Purple
      accentColor: '#FFD700', // Gold trim
      style: 'royal_treasure',
      lock: 'gem_encrusted_lock',
      gems: {
        enabled: true,
        count: 4,
        color: '#9C27B0'
      },
      glow: {
        enabled: true,
        color: '#9C27B0',
        intensity: 0.7,
        pulse: true
      }
    },
    indicator: {
      color: '#9C27B0',
      particles: true,
      particleCount: 6,
      sparkles: true
    }
  },

  legendary: {
    name: 'legendary_chest',
    design: {
      baseColor: '#1A1A1A', // Black
      metalColor: '#FF9800', // Orange gold
      accentColor: '#FFD700', // Bright gold
      style: 'ancient_treasure',
      lock: 'dragon_lock',
      gems: {
        enabled: true,
        count: 6,
        colors: ['#FF9800', '#FFD700', '#F44336']
      },
      glow: {
        enabled: true,
        color: '#FF9800',
        intensity: 1.0,
        pulse: true,
        rays: true
      },
      flames: {
        enabled: true,
        color: '#FF9800',
        subtle: true
      }
    },
    indicator: {
      color: '#FF9800',
      particles: true,
      particleCount: 10,
      sparkles: true,
      beams: true
    }
  }
};

const CHEST_ICON_SIZES = {
  inventory: 48,
  shop: 64,
  reward_screen: 96,
  opening_animation: 128
};
```

### 3.2 Chest State Variants

```typescript
const CHEST_STATES = {
  locked: {
    modifier: 'padlock_overlay',
    brightness: 0.8
  },
  unlocked: {
    modifier: 'none',
    brightness: 1.0
  },
  opening: {
    modifier: 'lid_raised',
    glow: 'intensified',
    particles: 'burst'
  },
  empty: {
    modifier: 'lid_open',
    brightness: 0.6,
    saturation: 0.5
  }
};
```

---

## 4. Cosmetic Category Icons

### 4.1 Outfit Categories

```typescript
const COSMETIC_CATEGORY_ICONS = {
  fullOutfit: {
    name: 'outfit_icon',
    symbol: 'pirate_silhouette',
    color: '#FFD700',
    description: 'Complete outfit sets'
  },

  headwear: {
    name: 'headwear_icon',
    symbol: 'tricorn_hat',
    color: '#FF7043',
    description: 'Hats, bandanas, helmets'
  },

  torso: {
    name: 'torso_icon',
    symbol: 'vest_shirt',
    color: '#42A5F5',
    description: 'Shirts, vests, coats'
  },

  legs: {
    name: 'legs_icon',
    symbol: 'pants',
    color: '#66BB6A',
    description: 'Pants, shorts, skirts'
  },

  footwear: {
    name: 'footwear_icon',
    symbol: 'boots',
    color: '#8D6E63',
    description: 'Boots, shoes, bare feet'
  },

  accessories: {
    name: 'accessories_icon',
    symbol: 'belt_buckle',
    color: '#AB47BC',
    description: 'Belts, jewelry, eyepatches'
  },

  backBling: {
    name: 'back_bling_icon',
    symbol: 'backpack_cape',
    color: '#26A69A',
    description: 'Capes, backpacks, wings'
  },

  weapons: {
    name: 'weapon_skin_icon',
    symbol: 'cutlass_crossed',
    color: '#EF5350',
    description: 'Weapon cosmetics'
  },

  emotes: {
    name: 'emote_icon',
    symbol: 'dancing_figure',
    color: '#FFA726',
    description: 'Emotes and dances'
  },

  victoryPoses: {
    name: 'victory_pose_icon',
    symbol: 'trophy_pose',
    color: '#FFEE58',
    description: 'Victory screen poses'
  }
};

const CATEGORY_ICON_STYLE = {
  baseShape: 'rounded_square',
  cornerRadius: 8,
  size: {
    small: 32,
    medium: 48,
    large: 64
  },
  outline: {
    width: 2,
    color: 'category_specific'
  },
  background: {
    style: 'gradient_subtle',
    opacity: 0.3
  }
};
```

### 4.2 Rarity Frame Overlays

```typescript
const RARITY_FRAMES = {
  common: {
    border: {
      color: '#9E9E9E',
      width: 2,
      style: 'solid'
    },
    corner: null,
    glow: false
  },

  uncommon: {
    border: {
      color: '#4CAF50',
      width: 2,
      style: 'solid'
    },
    corner: {
      style: 'simple',
      color: '#4CAF50'
    },
    glow: false
  },

  rare: {
    border: {
      color: '#2196F3',
      width: 3,
      style: 'double'
    },
    corner: {
      style: 'ornate',
      color: '#2196F3'
    },
    glow: {
      enabled: true,
      color: '#2196F3',
      intensity: 0.3
    }
  },

  epic: {
    border: {
      color: '#9C27B0',
      width: 3,
      style: 'ornate'
    },
    corner: {
      style: 'gem_encrusted',
      color: '#9C27B0',
      gems: true
    },
    glow: {
      enabled: true,
      color: '#9C27B0',
      intensity: 0.5,
      animated: true
    }
  },

  legendary: {
    border: {
      color: '#FF9800',
      width: 4,
      style: 'legendary_ornate'
    },
    corner: {
      style: 'flame_corner',
      color: '#FF9800',
      animated: true
    },
    glow: {
      enabled: true,
      color: '#FF9800',
      intensity: 0.8,
      animated: true,
      rays: true
    }
  },

  mythic: {
    border: {
      color: '#FF5722',
      width: 4,
      style: 'mythic_flame',
      animated: true
    },
    corner: {
      style: 'dragon_corner',
      animated: true
    },
    glow: {
      enabled: true,
      color: '#FF5722',
      intensity: 1.0,
      animated: true,
      particles: true
    },
    backgroundEffect: {
      style: 'shimmer',
      color: '#FFD700'
    }
  }
};
```

---

## 5. Consumable and Item Icons

### 5.1 In-Match Consumables

```typescript
const CONSUMABLE_ICONS = {
  healthPotion: {
    name: 'health_grog',
    design: {
      container: 'rum_bottle',
      liquidColor: '#F44336', // Red
      labelSymbol: 'heart_cross',
      cork: true
    },
    sizes: [24, 32, 48]
  },

  shieldPotion: {
    name: 'shield_elixir',
    design: {
      container: 'potion_flask',
      liquidColor: '#2196F3', // Blue
      labelSymbol: 'shield',
      magical_glow: true
    },
    sizes: [24, 32, 48]
  },

  speedBoost: {
    name: 'swift_rum',
    design: {
      container: 'hip_flask',
      liquidColor: '#FFEB3B', // Yellow
      labelSymbol: 'wind_swirl',
      motion_lines: true
    },
    sizes: [24, 32, 48]
  },

  ammoPack: {
    name: 'powder_keg',
    design: {
      container: 'small_barrel',
      color: '#795548', // Brown
      labelSymbol: 'cannonball',
      fuse: true
    },
    sizes: [24, 32, 48]
  }
};
```

### 5.2 Battle Pass Items

```typescript
const BATTLE_PASS_ITEM_ICONS = {
  tierReward: {
    free: {
      shape: 'simple_badge',
      color: '#9E9E9E',
      symbol: 'star'
    },
    premium: {
      shape: 'ornate_badge',
      color: '#FFD700',
      symbol: 'crown_star',
      shimmer: true
    }
  },

  xpBoost: {
    design: {
      shape: 'potion_bottle',
      color: '#00BCD4',
      symbol: 'up_arrow',
      multiplier_badge: true
    },
    variants: {
      small: { multiplier: '1.5x' },
      medium: { multiplier: '2x' },
      large: { multiplier: '3x' }
    }
  },

  exclusiveTag: {
    design: {
      shape: 'ribbon_banner',
      color: '#E040FB',
      text: 'EXCLUSIVE',
      sparkle: true
    }
  }
};
```

---

## 6. UI Element Icons

### 6.1 Navigation Icons

```typescript
const NAVIGATION_ICONS = {
  home: {
    symbol: 'anchor',
    style: 'filled',
    color: '#FFD700'
  },

  shop: {
    symbol: 'treasure_chest',
    style: 'filled',
    color: '#FFD700'
  },

  inventory: {
    symbol: 'backpack',
    style: 'filled',
    color: '#FFD700'
  },

  battlePass: {
    symbol: 'compass_map',
    style: 'filled',
    color: '#FFD700'
  },

  profile: {
    symbol: 'pirate_portrait',
    style: 'filled',
    color: '#FFD700'
  },

  settings: {
    symbol: 'ship_wheel',
    style: 'filled',
    color: '#FFD700'
  },

  friends: {
    symbol: 'crew_silhouettes',
    style: 'filled',
    color: '#FFD700'
  },

  notifications: {
    symbol: 'bell',
    style: 'filled',
    color: '#FFD700',
    badge_position: 'top_right'
  }
};

const NAVIGATION_ICON_STATES = {
  inactive: {
    color: '#9E9E9E',
    opacity: 0.7
  },
  active: {
    color: '#FFD700',
    opacity: 1.0,
    glow: true
  },
  disabled: {
    color: '#616161',
    opacity: 0.4
  }
};
```

### 6.2 Action Icons

```typescript
const ACTION_ICONS = {
  play: {
    symbol: 'crossed_swords',
    color: '#4CAF50'
  },

  equip: {
    symbol: 'checkmark_badge',
    color: '#4CAF50'
  },

  purchase: {
    symbol: 'coin_stack',
    color: '#FFD700'
  },

  gift: {
    symbol: 'wrapped_box',
    color: '#E040FB'
  },

  share: {
    symbol: 'message_bottle',
    color: '#2196F3'
  },

  info: {
    symbol: 'spyglass',
    color: '#9E9E9E'
  },

  close: {
    symbol: 'x_mark',
    color: '#F44336'
  },

  back: {
    symbol: 'arrow_left',
    color: '#FFFFFF'
  },

  refresh: {
    symbol: 'compass_spin',
    color: '#2196F3'
  },

  search: {
    symbol: 'spyglass_search',
    color: '#FFFFFF'
  },

  filter: {
    symbol: 'filter_funnel',
    color: '#FFFFFF'
  },

  sort: {
    symbol: 'sort_arrows',
    color: '#FFFFFF'
  }
};
```

### 6.3 Status Icons

```typescript
const STATUS_ICONS = {
  online: {
    symbol: 'green_lantern',
    color: '#4CAF50'
  },

  offline: {
    symbol: 'grey_lantern',
    color: '#9E9E9E'
  },

  inMatch: {
    symbol: 'crossed_swords_active',
    color: '#FF9800'
  },

  inLobby: {
    symbol: 'anchor_down',
    color: '#2196F3'
  },

  away: {
    symbol: 'hourglass',
    color: '#FFEB3B'
  },

  doNotDisturb: {
    symbol: 'red_flag',
    color: '#F44336'
  },

  newItem: {
    symbol: 'exclamation_star',
    color: '#FF4444',
    animated: true
  },

  locked: {
    symbol: 'padlock',
    color: '#9E9E9E'
  },

  unlocked: {
    symbol: 'open_padlock',
    color: '#4CAF50'
  },

  owned: {
    symbol: 'checkmark_circle',
    color: '#4CAF50'
  },

  equipped: {
    symbol: 'equipped_badge',
    color: '#FFD700',
    glow: true
  }
};
```

---

## 7. Social and Communication Icons

### 7.1 Friend System Icons

```typescript
const FRIEND_ICONS = {
  addFriend: {
    symbol: 'anchor_plus',
    color: '#4CAF50'
  },

  removeFriend: {
    symbol: 'anchor_minus',
    color: '#F44336'
  },

  pendingRequest: {
    symbol: 'anchor_clock',
    color: '#FF9800'
  },

  invite: {
    symbol: 'scroll_send',
    color: '#2196F3'
  },

  party: {
    symbol: 'crew_group',
    color: '#9C27B0'
  },

  partyLeader: {
    symbol: 'captain_hat',
    color: '#FFD700'
  },

  voice: {
    symbol: 'speaking_horn',
    color: '#4CAF50'
  },

  muted: {
    symbol: 'speaking_horn_crossed',
    color: '#F44336'
  },

  blocked: {
    symbol: 'skull_crossed',
    color: '#F44336'
  }
};
```

### 7.2 Achievement Icons

```typescript
const ACHIEVEMENT_ICONS = {
  // Achievement category badges
  categories: {
    combat: {
      symbol: 'crossed_cutlasses',
      color: '#F44336'
    },
    exploration: {
      symbol: 'treasure_map',
      color: '#4CAF50'
    },
    social: {
      symbol: 'pirate_crew',
      color: '#2196F3'
    },
    collection: {
      symbol: 'treasure_hoard',
      color: '#FF9800'
    },
    mastery: {
      symbol: 'captain_wheel',
      color: '#9C27B0'
    }
  },

  // Achievement tier badges
  tiers: {
    bronze: {
      frame: 'bronze_shield',
      color: '#CD7F32'
    },
    silver: {
      frame: 'silver_shield',
      color: '#C0C0C0'
    },
    gold: {
      frame: 'gold_shield',
      color: '#FFD700',
      glow: true
    }
  }
};
```

---

## 8. Technical Specifications

### 8.1 Icon Sizes and Formats

```typescript
const ICON_TECHNICAL_SPECS = {
  baseSizes: [16, 24, 32, 48, 64, 96, 128],

  formats: {
    source: 'SVG', // Vector source
    export: 'PNG', // Rasterized for game
    compressed: 'ASTC_4x4' // Mobile texture compression
  },

  padding: {
    safeArea: '10%', // Content margin
    touchTarget: 44 // Minimum touch size
  },

  atlas: {
    maxSize: 2048,
    padding: 2, // Between icons
    organization: 'category_based'
  }
};
```

### 8.2 Icon Atlas Organization

```typescript
const ICON_ATLASES = {
  currencies: {
    name: 'atlas_currencies',
    size: 512,
    contents: ['gold', 'doubloons', 'xp', 'battle_pass_xp'],
    sizes_included: [16, 24, 32, 48, 64]
  },

  chests: {
    name: 'atlas_chests',
    size: 1024,
    contents: ['all_chest_tiers', 'all_chest_states'],
    sizes_included: [48, 64, 96, 128]
  },

  categories: {
    name: 'atlas_categories',
    size: 512,
    contents: ['cosmetic_categories', 'rarity_frames'],
    sizes_included: [32, 48, 64]
  },

  navigation: {
    name: 'atlas_navigation',
    size: 512,
    contents: ['nav_icons', 'action_icons', 'status_icons'],
    sizes_included: [24, 32, 48]
  },

  social: {
    name: 'atlas_social',
    size: 256,
    contents: ['friend_icons', 'achievement_icons'],
    sizes_included: [24, 32, 48]
  }
};
```

### 8.3 Color Guidelines

```typescript
const ICON_COLOR_PALETTE = {
  // Currency colors
  gold: '#FFD700',
  doubloons: '#9C27B0',
  xp: '#00BCD4',
  battlePassXp: '#00BCD4',

  // Rarity colors
  common: '#9E9E9E',
  uncommon: '#4CAF50',
  rare: '#2196F3',
  epic: '#9C27B0',
  legendary: '#FF9800',
  mythic: '#FF5722',

  // Status colors
  positive: '#4CAF50',
  negative: '#F44336',
  warning: '#FF9800',
  info: '#2196F3',
  neutral: '#9E9E9E',

  // UI colors
  primary: '#FFD700',
  secondary: '#2196F3',
  accent: '#E040FB',

  // Outline/border
  outline_dark: '#1A1A1A',
  outline_light: '#FFFFFF'
};
```

---

## 9. Animation Specifications

### 9.1 Animated Icons

```typescript
const ANIMATED_ICONS = {
  spinningCoin: {
    icon: 'gold_coin',
    animation: 'y_axis_spin',
    duration: 0.8,
    loop: true,
    usage: ['rewards', 'currency_gain']
  },

  shimmeringDoubloon: {
    icon: 'doubloon',
    animation: 'shine_sweep',
    duration: 2.0,
    loop: true,
    usage: ['shop', 'premium_display']
  },

  pulsingChest: {
    icon: 'legendary_chest',
    animation: 'glow_pulse',
    duration: 1.5,
    loop: true,
    usage: ['reward_preview', 'shop_featured']
  },

  bouncingNotification: {
    icon: 'notification_badge',
    animation: 'attention_bounce',
    duration: 0.5,
    loop: false,
    trigger: 'on_appear'
  },

  loadingWheel: {
    icon: 'ship_wheel',
    animation: 'continuous_rotate',
    duration: 1.0,
    loop: true,
    usage: ['loading_states']
  }
};
```

### 9.2 State Transitions

```typescript
const ICON_TRANSITIONS = {
  hover: {
    scale: 1.1,
    duration: 0.15,
    easing: 'ease_out'
  },

  press: {
    scale: 0.95,
    duration: 0.1,
    easing: 'ease_out'
  },

  select: {
    glow: true,
    glowIntensity: 0.5,
    duration: 0.2
  },

  enable: {
    saturation: { from: 0, to: 1 },
    brightness: { from: 0.5, to: 1 },
    duration: 0.3
  },

  disable: {
    saturation: { from: 1, to: 0.3 },
    brightness: { from: 1, to: 0.7 },
    duration: 0.2
  }
};
```

---

## 10. Accessibility Considerations

### 10.1 Color Blind Support

```typescript
const ACCESSIBILITY_ICONS = {
  colorBlindMode: {
    // Add shape variations for color-coded items
    rarityShapes: {
      common: 'circle',
      uncommon: 'triangle',
      rare: 'diamond',
      epic: 'hexagon',
      legendary: 'star'
    },

    // Add patterns to currency
    currencyPatterns: {
      gold: 'solid',
      doubloons: 'striped',
      xp: 'dotted'
    }
  },

  highContrast: {
    outlineWidth: 3,
    outlineColor: '#000000',
    backgroundColor: '#FFFFFF'
  }
};
```

### 10.2 Size and Readability

```typescript
const READABILITY_GUIDELINES = {
  minimumSize: {
    interactive: 44, // Pixels, touch target
    informational: 16 // Pixels, display only
  },

  contrast: {
    minimum: 4.5, // WCAG AA standard
    enhanced: 7.0 // WCAG AAA
  },

  complexity: {
    tiny: 'silhouette_only', // Under 24px
    small: 'simple_detail', // 24-32px
    medium: 'standard_detail', // 32-48px
    large: 'full_detail' // 48px+
  }
};
```

---

## 11. Testing Checklist

### 11.1 Visual Tests

| Test | Criteria |
|------|----------|
| Currency distinction | Gold vs Doubloons immediately clear |
| Rarity recognition | Each tier visually distinct |
| Size scalability | Readable at all defined sizes |
| Color consistency | Matches defined palette |
| Theme alignment | Fits pirate aesthetic |
| Silhouette clarity | Recognizable at 16px |

### 11.2 Technical Tests

| Test | Criteria |
|------|----------|
| Atlas generation | All icons fit in atlases |
| Compression quality | No visible artifacts |
| Memory usage | Within budget (< 4MB total) |
| Load time | Atlas loads < 100ms |
| Animation performance | 60 FPS maintained |

### 11.3 Accessibility Tests

| Test | Criteria |
|------|----------|
| Color blind mode | All icons distinguishable |
| Touch targets | Minimum 44px achieved |
| High contrast | Passes WCAG AA |
| Screen reader labels | All icons have alt text |

---

## Appendix A: Icon Inventory

### Complete Icon List

| Category | Count | Priority |
|----------|-------|----------|
| Currencies | 4 | Critical |
| Chest tiers | 5 | Critical |
| Chest states | 4 | Critical |
| Cosmetic categories | 10 | High |
| Rarity frames | 6 | High |
| Navigation | 8 | Critical |
| Actions | 12 | High |
| Status | 12 | High |
| Social | 10 | Medium |
| Achievements | 8 | Medium |
| Consumables | 4 | Medium |
| **Total** | **83** | - |

---

## Appendix B: Asset Delivery

### File Naming Convention
```
icon_[category]_[name]_[size].[format]
Example: icon_currency_gold_48.png
```

### Folder Structure
```
icons/
├── currencies/
│   ├── gold/
│   ├── doubloons/
│   └── xp/
├── chests/
│   ├── common/
│   ├── uncommon/
│   └── ...
├── categories/
├── navigation/
├── actions/
├── status/
└── atlases/
```
