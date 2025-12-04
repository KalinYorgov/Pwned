# Premium Cosmetic Outfit Set Specification

## Overview

This specification defines the launch set of 10 premium cosmetic outfits for monetization. These outfits represent the high-quality, aspirational content that drives player engagement and revenue. Each outfit must be visually distinct, appeal to different player fantasies, and maintain competitive fairness.

## Design Principles

### Core Requirements

1. **Style Guide Compliance**: All outfits follow the established visual style guide
2. **Visual Distinction**: Each outfit instantly recognizable at gameplay distances
3. **Fair Play**: No competitive advantage (identical hitboxes)
4. **Performance**: Meet mobile optimization targets
5. **Animation Compatibility**: Work flawlessly with all character animations
6. **Shop Appeal**: Look stunning in preview and promotional materials

### Rarity Tiers

```typescript
enum CosmeticRarity {
  RARE = 'rare',           // 800 Doubloons
  EPIC = 'epic',           // 1500 Doubloons
  LEGENDARY = 'legendary', // 2000 Doubloons
  MYTHIC = 'mythic'        // 2500 Doubloons (future)
}

interface OutfitDefinition {
  id: string;
  name: string;
  rarity: CosmeticRarity;
  description: string;
  theme: string;
  hasVFX: boolean;
  price: number;
  components: OutfitComponent[];
  unlockMethod: 'shop' | 'battle_pass' | 'event' | 'bundle';
}
```

---

## Launch Outfit Collection

### 1. Dread Captain

**Rarity:** Legendary (2000 Doubloons)

**Theme:** Fearsome pirate lord commanding terror across the seas

**Visual Design:**
- Long black captain's coat with gold trim and skull buttons
- Weathered tricorn hat with black feathers and gold skull emblem
- Dark leather boots with silver buckles
- Crimson sash and ornate belt with pistol holsters
- Eyepatch with decorative gold filigree
- Intimidating presence with billowing coat physics

**Color Palette:**
- Primary: Midnight black (#1a1a1a)
- Secondary: Antique gold (#d4af37)
- Accent: Crimson red (#8b0000)
- Highlights: Silver (#c0c0c0)

**Technical Specs:**
```typescript
const DREAD_CAPTAIN: OutfitDefinition = {
  id: 'outfit_dread_captain',
  name: 'Dread Captain',
  rarity: 'legendary',
  description: 'The seas tremble at the sight of the Dread Captain\'s black sails.',
  theme: 'dark_commander',
  hasVFX: false,
  price: 2000,
  components: [
    { type: 'head', asset: 'dread_captain_hat' },
    { type: 'body', asset: 'dread_captain_coat' },
    { type: 'legs', asset: 'dread_captain_boots' },
    { type: 'accessory', asset: 'dread_captain_eyepatch' }
  ],
  unlockMethod: 'shop'
};
```

**Cloth Simulation:**
- Coat tails with physics simulation
- Cape movement during running/combat
- Performance-optimized cloth bones

---

### 2. Tropical Swashbuckler

**Rarity:** Epic (1500 Doubloons)

**Theme:** Colorful Caribbean freedom fighter with island flair

**Visual Design:**
- Vibrant open shirt with tropical floral pattern
- Bright bandana with contrasting colors
- White cotton pants with colorful sash
- Leather sandals with beaded accents
- Multiple gold earrings and beaded necklaces
- Sun-kissed, adventurous appearance

**Color Palette:**
- Primary: Ocean turquoise (#40e0d0)
- Secondary: Sunset orange (#ff6b35)
- Accent: Hibiscus pink (#ff69b4)
- Base: Clean white (#ffffff)

**Technical Specs:**
```typescript
const TROPICAL_SWASHBUCKLER: OutfitDefinition = {
  id: 'outfit_tropical_swashbuckler',
  name: 'Tropical Swashbuckler',
  rarity: 'epic',
  description: 'Dance through battle with island rhythm and style.',
  theme: 'caribbean_freedom',
  hasVFX: false,
  price: 1500,
  components: [
    { type: 'head', asset: 'tropical_bandana' },
    { type: 'body', asset: 'tropical_shirt' },
    { type: 'legs', asset: 'tropical_pants' },
    { type: 'accessory', asset: 'tropical_jewelry' }
  ],
  unlockMethod: 'shop'
};
```

---

### 3. Ghost Pirate

**Rarity:** Legendary (2000 Doubloons)

**Theme:** Spectral undead pirate with ethereal glow effects

**Visual Design:**
- Translucent spectral body with visible bones
- Tattered captain's uniform, partially transparent
- Glowing blue-green ethereal particles
- Ghostly trail effect during movement
- Empty glowing eye sockets
- Chains and anchors as ghostly accessories

**Color Palette:**
- Primary: Ethereal blue (#4169e1)
- Secondary: Spectral green (#00ff7f)
- Accent: Bone white (#f0f8ff)
- Glow: Cyan (#00ffff)

**VFX Components:**
```typescript
const GHOST_PIRATE_VFX = {
  idle: {
    effect: 'ethereal_particles',
    intensity: 'low',
    color: '#00ffff'
  },
  movement: {
    effect: 'ghost_trail',
    trailLength: 0.5, // seconds
    opacity: 0.3
  },
  eyes: {
    effect: 'glow_pulse',
    color: '#4169e1',
    frequency: 1.0 // Hz
  }
};

const GHOST_PIRATE: OutfitDefinition = {
  id: 'outfit_ghost_pirate',
  name: 'Ghost Pirate',
  rarity: 'legendary',
  description: 'Death could not end the voyage. Neither will you.',
  theme: 'spectral_undead',
  hasVFX: true,
  price: 2000,
  components: [
    { type: 'full_body', asset: 'ghost_pirate_spectral' },
    { type: 'vfx', asset: 'ghost_pirate_particles' }
  ],
  unlockMethod: 'shop'
};
```

**VFX Guidelines:**
- Particles must not obscure gameplay vision
- Effects subtle enough for competitive play
- Disable option in settings if needed
- Performance budget: 50 particles max

---

### 4. Sea Witch/Wizard

**Rarity:** Legendary (2000 Doubloons)

**Theme:** Magical pirate with arcane powers and mystical aesthetic

**Visual Design:**
- Flowing mystical robes with oceanic patterns
- Conical wizard hat with nautical star emblem
- Glowing runes on fabric and skin
- Tentacle-like rope accessories
- Crystal ball or magical orb accessory
- Seaweed and coral integrated into design

**Color Palette:**
- Primary: Deep purple (#4b0082)
- Secondary: Sea green (#2e8b57)
- Accent: Mystic gold (#ffd700)
- Glow: Arcane purple (#9400d3)

**Technical Specs:**
```typescript
const SEA_WITCH: OutfitDefinition = {
  id: 'outfit_sea_witch',
  name: 'Sea Witch',
  rarity: 'legendary',
  description: 'Master of tide and tempest, weaver of ocean magic.',
  theme: 'arcane_oceanic',
  hasVFX: true,
  price: 2000,
  components: [
    { type: 'head', asset: 'sea_witch_hat' },
    { type: 'body', asset: 'sea_witch_robes' },
    { type: 'accessory', asset: 'sea_witch_orb' },
    { type: 'vfx', asset: 'sea_witch_runes' }
  ],
  unlockMethod: 'shop'
};
```

**VFX Components:**
- Subtle rune glow on fabric
- Magical aura around hands (non-combat)
- Orb particle effects

---

### 5. Navy Admiral

**Rarity:** Epic (1500 Doubloons)

**Theme:** Military rival faction with commanding presence

**Visual Design:**
- Formal navy blue military coat with brass buttons
- White pants with gold stripe
- Polished black boots
- Bicorn admiral hat with white plume
- Ceremonial sword sheath
- Medals and military decorations

**Color Palette:**
- Primary: Navy blue (#000080)
- Secondary: Pristine white (#f5f5f5)
- Accent: Brass gold (#b5a642)
- Trim: Red (#b22222)

**Technical Specs:**
```typescript
const NAVY_ADMIRAL: OutfitDefinition = {
  id: 'outfit_navy_admiral',
  name: 'Navy Admiral',
  rarity: 'epic',
  description: 'The Crown\'s finest, hunting pirates on royal orders.',
  theme: 'military_authority',
  hasVFX: false,
  price: 1500,
  components: [
    { type: 'head', asset: 'admiral_bicorn' },
    { type: 'body', asset: 'admiral_coat' },
    { type: 'legs', asset: 'admiral_pants' },
    { type: 'accessory', asset: 'admiral_medals' }
  ],
  unlockMethod: 'shop'
};
```

---

### 6. Kraken Hunter

**Rarity:** Epic (1500 Doubloons)

**Theme:** Seasoned monster hunter with trophies and specialized gear

**Visual Design:**
- Rugged leather armor with sea monster scales
- Harpoon strapped to back
- Kraken tentacle trophy belt
- Scarred, weathered appearance
- Utility pouches and hooks
- Shark tooth necklace

**Color Palette:**
- Primary: Worn leather brown (#8b4513)
- Secondary: Sea monster purple (#663399)
- Accent: Bone ivory (#fffff0)
- Metal: Oxidized bronze (#cd7f32)

**Technical Specs:**
```typescript
const KRAKEN_HUNTER: OutfitDefinition = {
  id: 'outfit_kraken_hunter',
  name: 'Kraken Hunter',
  rarity: 'epic',
  description: 'The deep holds no terror for one who hunts the hunters.',
  theme: 'monster_slayer',
  hasVFX: false,
  price: 1500,
  components: [
    { type: 'head', asset: 'hunter_hood' },
    { type: 'body', asset: 'hunter_armor' },
    { type: 'back', asset: 'hunter_harpoon' },
    { type: 'accessory', asset: 'hunter_trophies' }
  ],
  unlockMethod: 'shop'
};
```

---

### 7. Treasure Seeker

**Rarity:** Legendary (2000 Doubloons)

**Theme:** Wealthy, gold-obsessed pirate dripping with riches

**Visual Design:**
- Opulent velvet coat with gold thread
- Excessive gold jewelry (rings, chains, bracelets)
- Crown or elaborate golden headpiece
- Treasure map scroll accessory
- Gem-encrusted belt buckle
- Golden boots with jewel inlays

**Color Palette:**
- Primary: Rich gold (#ffd700)
- Secondary: Royal purple (#800080)
- Accent: Ruby red (#e0115f)
- Trim: Emerald green (#50c878)

**Technical Specs:**
```typescript
const TREASURE_SEEKER: OutfitDefinition = {
  id: 'outfit_treasure_seeker',
  name: 'Treasure Seeker',
  rarity: 'legendary',
  description: 'Why steal gold when you can wear all of it?',
  theme: 'wealthy_excess',
  hasVFX: false,
  price: 2000,
  components: [
    { type: 'head', asset: 'seeker_crown' },
    { type: 'body', asset: 'seeker_coat' },
    { type: 'legs', asset: 'seeker_boots' },
    { type: 'accessory', asset: 'seeker_jewelry' }
  ],
  unlockMethod: 'shop'
};
```

**Material Specs:**
- High metallic shader for gold elements
- Gem refraction effects
- Subtle sparkle particles (optional)

---

### 8. Mermaid/Merman

**Rarity:** Legendary (2000 Doubloons)

**Theme:** Aquatic fantasy hybrid with oceanic beauty

**Visual Design:**
- Scale-textured skin with iridescent shimmer
- Seashell and coral accessories
- Flowing seaweed-like hair or headpiece
- Pearl jewelry and decorations
- Fin-like extensions on arms and legs
- Trident-inspired weapon cosmetic option

**Color Palette:**
- Primary: Ocean teal (#008080)
- Secondary: Coral pink (#ff7f50)
- Accent: Pearl white (#fdeef4)
- Shimmer: Iridescent gradient

**VFX Components:**
```typescript
const MERMAID_VFX = {
  idle: {
    effect: 'water_droplets',
    intensity: 'low',
    rate: 2 // particles per second
  },
  scales: {
    effect: 'iridescent_shimmer',
    intensity: 'medium',
    lightResponsive: true
  }
};

const MERMAID: OutfitDefinition = {
  id: 'outfit_mermaid',
  name: 'Ocean\'s Chosen',
  rarity: 'legendary',
  description: 'Born of wave and foam, they walk between two worlds.',
  theme: 'aquatic_fantasy',
  hasVFX: true,
  price: 2000,
  components: [
    { type: 'full_body', asset: 'mermaid_scaled_body' },
    { type: 'head', asset: 'mermaid_crown' },
    { type: 'accessory', asset: 'mermaid_pearls' },
    { type: 'vfx', asset: 'mermaid_shimmer' }
  ],
  unlockMethod: 'shop'
};
```

---

### 9. Skeleton Crew

**Rarity:** Epic (1500 Doubloons)

**Theme:** Classic undead pirate skeleton with dark humor

**Visual Design:**
- Exposed skull head with glowing eyes
- Tattered pirate clothing revealing bones
- Rusty cutlass and pistol accessories
- Barnacles and sea growth on bones
- Missing limbs replaced with peg/hook
- Humorous skull expressions

**Color Palette:**
- Primary: Bone white (#e3dac9)
- Secondary: Tattered brown (#654321)
- Accent: Ghostly blue (#6495ed)
- Decay: Seaweed green (#2e8b57)

**Technical Specs:**
```typescript
const SKELETON_CREW: OutfitDefinition = {
  id: 'outfit_skeleton_crew',
  name: 'Skeleton Crew',
  rarity: 'epic',
  description: 'The crew that never dies, because they\'re already dead.',
  theme: 'undead_classic',
  hasVFX: true,
  price: 1500,
  components: [
    { type: 'full_body', asset: 'skeleton_body' },
    { type: 'clothing', asset: 'skeleton_rags' },
    { type: 'accessory', asset: 'skeleton_hook' },
    { type: 'vfx', asset: 'skeleton_eye_glow' }
  ],
  unlockMethod: 'shop'
};
```

---

### 10. Steampunk Corsair

**Rarity:** Legendary (2000 Doubloons)

**Theme:** Clockwork and brass Victorian-era pirate inventor

**Visual Design:**
- Leather aviator coat with brass fittings
- Goggles with multiple lenses
- Mechanical arm or prosthetic
- Gear and cog accessories
- Steam vents and pipes on back
- Pocket watch chains and instruments

**Color Palette:**
- Primary: Aged brass (#b5651d)
- Secondary: Dark leather (#3d2314)
- Accent: Copper (#b87333)
- Steam: Misty white (#f5f5f5)

**VFX Components:**
```typescript
const STEAMPUNK_VFX = {
  idle: {
    effect: 'steam_puffs',
    interval: 3, // seconds
    locations: ['back_vents', 'arm_joint']
  },
  gears: {
    effect: 'rotating_gears',
    speed: 30 // degrees per second
  }
};

const STEAMPUNK_CORSAIR: OutfitDefinition = {
  id: 'outfit_steampunk_corsair',
  name: 'Steampunk Corsair',
  rarity: 'legendary',
  description: 'The future of piracy runs on steam and ingenuity.',
  theme: 'clockwork_inventor',
  hasVFX: true,
  price: 2000,
  components: [
    { type: 'head', asset: 'steampunk_goggles' },
    { type: 'body', asset: 'steampunk_coat' },
    { type: 'arm', asset: 'steampunk_mech_arm' },
    { type: 'back', asset: 'steampunk_apparatus' },
    { type: 'vfx', asset: 'steampunk_steam' }
  ],
  unlockMethod: 'shop'
};
```

---

## Technical Specifications

### Polygon Budgets

```typescript
const OUTFIT_POLY_BUDGETS = {
  // Per-component limits
  head: 1500,      // triangles
  body: 4000,      // triangles
  legs: 1500,      // triangles
  back: 1000,      // triangles
  accessory: 500,  // triangles

  // Total outfit limits by rarity
  total: {
    epic: 7000,      // triangles
    legendary: 8500, // triangles (allows for more detail)
    mythic: 10000    // triangles (future premium tier)
  },

  // LOD requirements
  lod: {
    lod0: 1.0,    // Full detail (close range)
    lod1: 0.5,    // Half detail (medium range)
    lod2: 0.25    // Quarter detail (far range)
  }
};
```

### Texture Specifications

```typescript
const OUTFIT_TEXTURE_SPECS = {
  // Texture sizes
  sizes: {
    diffuse: 1024,    // pixels (1024x1024)
    normal: 512,      // pixels
    emissive: 256,    // pixels (if needed)
    mask: 256         // pixels (metallic/roughness)
  },

  // Compression
  format: {
    android: 'ETC2',
    ios: 'ASTC_4x4',
    fallback: 'RGBA32'
  },

  // Atlas optimization
  atlasing: {
    enabled: true,
    maxAtlasSize: 2048,
    minItemsPerAtlas: 4
  }
};
```

### VFX Performance Budget

```typescript
const VFX_OUTFIT_BUDGET = {
  // Per-outfit limits
  maxParticles: 50,
  maxEmitters: 3,
  maxDrawCalls: 2,

  // Quality tiers
  quality: {
    high: {
      particleMultiplier: 1.0,
      effectsEnabled: true
    },
    medium: {
      particleMultiplier: 0.5,
      effectsEnabled: true
    },
    low: {
      particleMultiplier: 0,
      effectsEnabled: false
    }
  },

  // Player settings
  reduceOtherPlayersVFX: true,
  selfVFXAlwaysFull: true
};
```

---

## Hitbox Compliance

### Fair Play Standards

```typescript
const HITBOX_COMPLIANCE = {
  // All outfits must use identical hitbox
  standardHitbox: {
    capsuleHeight: 180, // cm
    capsuleRadius: 35,  // cm
    headSphereRadius: 20 // cm
  },

  // Visual bounds can exceed hitbox
  maxVisualExtension: {
    shoulder: 15, // cm beyond hitbox
    back: 20,     // cm (capes, backpacks)
    head: 25      // cm (tall hats)
  },

  // Prohibited visual elements
  prohibited: [
    'thin_silhouettes',      // Must maintain readable shape
    'camouflage_patterns',   // No environmental blending
    'excessive_glow',        // No visibility obstruction
    'distracting_animation'  // No attention-drawing loops
  ]
};
```

---

## Animation Compatibility

### Required Animation Tests

```typescript
const ANIMATION_COMPATIBILITY_TESTS = [
  // Locomotion
  'idle',
  'walk',
  'run',
  'sprint',
  'strafe_left',
  'strafe_right',
  'jump',
  'land',
  'crouch',

  // Combat
  'attack_melee',
  'attack_ranged',
  'ability_cast_generic',
  'hurt_front',
  'hurt_back',
  'death',

  // Special
  'barrel_roll',
  'grapple_throw',
  'grapple_pull',
  'drink_grog',
  'emote_wave',
  'emote_dance',
  'victory_pose',
  'defeat_pose'
];

// Validation requirements
const ANIMATION_REQUIREMENTS = {
  noClipping: true,
  noFloatingElements: true,
  weightPaintingSmooth: true,
  blendShapesWorking: true,
  physicsSimStable: true
};
```

---

## Shop Integration

### Preview System

```typescript
interface OutfitPreview {
  // 3D preview
  turntable: {
    rotationSpeed: 15, // degrees per second
    autoRotate: true,
    dragToRotate: true,
    zoomEnabled: true,
    zoomRange: { min: 0.8, max: 1.5 }
  };

  // Animation preview
  animations: [
    { name: 'Idle', default: true },
    { name: 'Walk', },
    { name: 'Combat Stance' },
    { name: 'Victory Pose' }
  ];

  // Environment
  previewEnvironment: 'pirate_ship_deck';
  lighting: 'golden_hour';

  // Try-on
  tryInMatch: false; // Not supported
  tryInLobby: true;  // 24 hour trial
}
```

### Bundle Configuration

```typescript
const LAUNCH_OUTFIT_BUNDLES = [
  {
    id: 'bundle_undead_pack',
    name: 'Undead Pack',
    outfits: ['outfit_ghost_pirate', 'outfit_skeleton_crew'],
    originalPrice: 3500,
    bundlePrice: 2800, // 20% discount
    featured: true
  },
  {
    id: 'bundle_legends_collection',
    name: 'Legends Collection',
    outfits: ['outfit_dread_captain', 'outfit_treasure_seeker', 'outfit_steampunk_corsair'],
    originalPrice: 6000,
    bundlePrice: 4500, // 25% discount
    featured: true
  }
];
```

---

## Production Pipeline

### Asset Creation Workflow

1. **Concept Phase**
   - Character artist creates concept sketches
   - Review against style guide
   - Approval before 3D production

2. **Modeling Phase**
   - Base mesh creation
   - Component separation
   - UV mapping and optimization

3. **Texturing Phase**
   - Albedo/diffuse creation
   - Normal map baking
   - Material definition

4. **Rigging Phase**
   - Bind to base rig
   - Weight painting
   - Physics bone setup (cloth)

5. **VFX Phase** (if applicable)
   - Particle system creation
   - Shader effects
   - Performance optimization

6. **Testing Phase**
   - Animation compatibility
   - Hitbox verification
   - Performance testing
   - Visual QA across devices

### Quality Checklist

```typescript
const OUTFIT_QA_CHECKLIST = {
  visual: [
    'Matches approved concept',
    'Follows style guide colors',
    'Readable silhouette at distance',
    'No texture stretching',
    'Materials render correctly'
  ],

  technical: [
    'Within poly budget',
    'Textures compressed correctly',
    'LODs generated and working',
    'No mesh errors (holes, inverted normals)',
    'Proper naming conventions'
  ],

  animation: [
    'No clipping with base animations',
    'Physics simulation stable',
    'Blending smooth between states',
    'No floating elements',
    'Victory/defeat poses work'
  ],

  gameplay: [
    'Hitbox unchanged',
    'No competitive advantage',
    'VFX not distracting to others',
    'Visible in all map areas',
    'Works with all abilities'
  ],

  platform: [
    'iOS performance acceptable',
    'Android performance acceptable',
    'Low-end device rendering correct',
    'Memory within budget',
    'Loading time acceptable'
  ]
};
```

---

## Monetization Strategy

### Pricing Tiers

| Rarity    | Price (Doubloons) | USD Equivalent |
|-----------|-------------------|----------------|
| Rare      | 800               | ~$8            |
| Epic      | 1500              | ~$15           |
| Legendary | 2000              | ~$20           |
| Mythic    | 2500              | ~$25           |

### Launch Collection Value

```typescript
const LAUNCH_COLLECTION = {
  totalOutfits: 10,
  breakdown: {
    epic: 4,      // 4 × 1500 = 6000
    legendary: 6  // 6 × 2000 = 12000
  },
  totalValue: 18000, // Doubloons
  usdEquivalent: 180 // Approximate
};
```

### Release Schedule

- **Launch Week**: All 10 outfits available
- **Featured Rotation**: 3 outfits featured per week
- **Bundle Deals**: 2 bundles with discount
- **Battle Pass Exclusive**: Consider 1 variant for S1 BP

---

## Dependencies

- ART-003: Base Player Character Model (rigging compatibility)
- ART-004: Default Pirate Outfit Set (style reference)
- ART-001: Visual Style Guide (design standards)
- ART-002: Technical Art Specifications (performance budgets)
- MON-001: Premium Currency System (pricing integration)
