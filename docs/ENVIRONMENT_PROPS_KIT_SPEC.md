# Environment Props Kit Specification

## Overview

This specification defines the modular environment props kit for dressing the battle royale map. These props create visual richness, provide tactical cover, and reinforce the pirate theme while maintaining mobile performance. The kit is designed for efficient reuse across all Points of Interest (POIs) and terrain types.

## Design Principles

### Core Requirements

1. **Modularity**: Props combine and tile seamlessly
2. **Consistency**: Unified scale, style, and quality across all props
3. **Performance**: Aggressive optimization for mobile
4. **Gameplay Clarity**: Cover objects instantly readable
5. **Thematic Cohesion**: Everything reinforces pirate fantasy

### Technical Standards

```typescript
interface PropDefinition {
  id: string;
  name: string;
  category: PropCategory;
  subcategory: string;
  polyBudget: number;
  textureAtlas: string;
  hasCollision: boolean;
  collisionType: 'convex' | 'mesh' | 'primitive' | 'none';
  lodLevels: number;
  isCover: boolean;
  isDestructible: boolean;
  variants: number;
}

enum PropCategory {
  NATURAL = 'natural',
  PIRATE = 'pirate',
  STRUCTURES = 'structures',
  INTERACTIVE = 'interactive',
  DECORATIVE = 'decorative'
}
```

---

## Natural Props

### Vegetation

#### Palm Trees

```typescript
const PALM_TREE_PROPS = {
  // Large palm - landmark/shade
  palm_large: {
    id: 'prop_palm_large',
    polyBudget: 800,
    height: 8, // meters
    variants: 3,
    lod: {
      lod0: { distance: 0, polys: 800 },
      lod1: { distance: 30, polys: 400 },
      lod2: { distance: 60, polys: 150, billboard: false },
      lod3: { distance: 100, polys: 0, billboard: true }
    },
    features: {
      swayAnimation: true,
      coconuts: true,
      frondPhysics: 'simplified'
    }
  },

  // Medium palm - common placement
  palm_medium: {
    id: 'prop_palm_medium',
    polyBudget: 500,
    height: 5,
    variants: 4,
    lod: {
      lod0: { distance: 0, polys: 500 },
      lod1: { distance: 25, polys: 250 },
      lod2: { distance: 50, polys: 100 },
      lod3: { distance: 80, billboard: true }
    }
  },

  // Small palm/bush - ground cover
  palm_small: {
    id: 'prop_palm_small',
    polyBudget: 200,
    height: 2,
    variants: 5,
    lod: {
      lod0: { distance: 0, polys: 200 },
      lod1: { distance: 20, polys: 80 },
      cullDistance: 50
    }
  },

  // Dead/fallen palm
  palm_dead: {
    id: 'prop_palm_dead',
    polyBudget: 400,
    variants: 2,
    isCover: true,
    collision: 'convex'
  }
};
```

#### Tropical Plants

```typescript
const TROPICAL_PLANTS = {
  // Fern clusters
  fern_cluster: {
    id: 'prop_fern_cluster',
    polyBudget: 150,
    variants: 6,
    placement: 'ground_scatter',
    cullDistance: 30
  },

  // Banana plants
  banana_plant: {
    id: 'prop_banana_plant',
    polyBudget: 300,
    height: 3,
    variants: 3,
    features: {
      fruitVisible: true,
      swayAnimation: true
    }
  },

  // Flowering bushes
  flower_bush: {
    id: 'prop_flower_bush',
    polyBudget: 200,
    variants: 4,
    colors: ['hibiscus_red', 'plumeria_white', 'orchid_purple', 'bird_of_paradise'],
    isCover: true,
    coverHeight: 'crouch'
  },

  // Tall grass
  grass_tall: {
    id: 'prop_grass_tall',
    polyBudget: 50,
    variants: 8,
    instanced: true,
    maxInstances: 500,
    swayAnimation: true
  },

  // Seagrass (underwater/beach edge)
  seagrass: {
    id: 'prop_seagrass',
    polyBudget: 30,
    variants: 5,
    instanced: true,
    underwaterShader: true
  }
};
```

#### Rocks and Boulders

```typescript
const ROCK_PROPS = {
  // Large boulder - major cover
  boulder_large: {
    id: 'prop_boulder_large',
    polyBudget: 600,
    variants: 4,
    isCover: true,
    coverHeight: 'full',
    collision: 'convex',
    climbable: true,
    lod: {
      lod0: { distance: 0, polys: 600 },
      lod1: { distance: 40, polys: 300 },
      lod2: { distance: 80, polys: 100 }
    }
  },

  // Medium rock - partial cover
  rock_medium: {
    id: 'prop_rock_medium',
    polyBudget: 300,
    variants: 6,
    isCover: true,
    coverHeight: 'crouch',
    collision: 'convex'
  },

  // Small rocks - scatter decoration
  rock_small: {
    id: 'prop_rock_small',
    polyBudget: 80,
    variants: 10,
    instanced: true,
    collision: 'none',
    cullDistance: 40
  },

  // Rock formation - climbable structure
  rock_formation: {
    id: 'prop_rock_formation',
    polyBudget: 1200,
    variants: 3,
    isCover: true,
    climbable: true,
    collision: 'mesh'
  },

  // Coastal rocks (wet shader)
  rock_coastal: {
    id: 'prop_rock_coastal',
    polyBudget: 400,
    variants: 5,
    shader: 'wet_rock',
    barnacles: true
  }
};
```

#### Beach and Ocean

```typescript
const BEACH_PROPS = {
  // Coral formations
  coral_large: {
    id: 'prop_coral_large',
    polyBudget: 400,
    variants: 4,
    colors: ['pink', 'purple', 'orange', 'blue'],
    underwater: true
  },

  coral_small: {
    id: 'prop_coral_small',
    polyBudget: 100,
    variants: 8,
    instanced: true
  },

  // Seashells
  shell_conch: {
    id: 'prop_shell_conch',
    polyBudget: 50,
    variants: 3,
    collectible: false, // Decorative only
    cullDistance: 20
  },

  shell_scatter: {
    id: 'prop_shell_scatter',
    polyBudget: 30,
    variants: 6,
    instanced: true,
    decalBased: true
  },

  // Starfish
  starfish: {
    id: 'prop_starfish',
    polyBudget: 20,
    variants: 4,
    colors: ['orange', 'red', 'purple', 'pink']
  },

  // Driftwood
  driftwood_large: {
    id: 'prop_driftwood_large',
    polyBudget: 300,
    variants: 4,
    isCover: true,
    coverHeight: 'crouch'
  },

  driftwood_small: {
    id: 'prop_driftwood_small',
    polyBudget: 80,
    variants: 6
  },

  // Beach debris
  kelp_pile: {
    id: 'prop_kelp_pile',
    polyBudget: 100,
    variants: 4
  }
};
```

---

## Pirate Props

### Barrels and Crates

```typescript
const CONTAINER_PROPS = {
  // Standard barrel
  barrel_standard: {
    id: 'prop_barrel_standard',
    polyBudget: 120,
    variants: 3,
    isCover: true,
    coverHeight: 'crouch',
    collision: 'primitive', // Cylinder
    isDestructible: true,
    destructionVFX: 'barrel_break',
    contents: ['empty', 'fish', 'gunpowder']
  },

  // Barrel stack (2-3 barrels)
  barrel_stack: {
    id: 'prop_barrel_stack',
    polyBudget: 300,
    variants: 3,
    isCover: true,
    coverHeight: 'full',
    collision: 'convex'
  },

  // Barrel row (horizontal)
  barrel_row: {
    id: 'prop_barrel_row',
    polyBudget: 350,
    variants: 2,
    isCover: true,
    coverHeight: 'crouch'
  },

  // Wooden crate - small
  crate_small: {
    id: 'prop_crate_small',
    polyBudget: 80,
    variants: 4,
    isCover: true,
    coverHeight: 'crouch',
    collision: 'primitive', // Box
    stackable: true
  },

  // Wooden crate - large
  crate_large: {
    id: 'prop_crate_large',
    polyBudget: 150,
    variants: 3,
    isCover: true,
    coverHeight: 'full',
    collision: 'primitive',
    climbable: true
  },

  // Crate stack
  crate_stack: {
    id: 'prop_crate_stack',
    polyBudget: 400,
    variants: 4,
    isCover: true,
    coverHeight: 'full',
    climbable: true
  },

  // Open crate (with goods visible)
  crate_open: {
    id: 'prop_crate_open',
    polyBudget: 200,
    variants: 3,
    contents: ['cloth', 'bottles', 'fruit']
  },

  // Cargo net with crates
  cargo_net: {
    id: 'prop_cargo_net',
    polyBudget: 500,
    variants: 2,
    hanging: true
  }
};
```

### Nautical Equipment

```typescript
const NAUTICAL_PROPS = {
  // Cannon
  cannon_deck: {
    id: 'prop_cannon_deck',
    polyBudget: 400,
    variants: 2,
    isCover: true,
    coverHeight: 'crouch',
    collision: 'convex'
  },

  cannon_fort: {
    id: 'prop_cannon_fort',
    polyBudget: 600,
    variants: 2,
    isCover: true,
    coverHeight: 'full'
  },

  // Anchor
  anchor_large: {
    id: 'prop_anchor_large',
    polyBudget: 300,
    variants: 2,
    isCover: true,
    coverHeight: 'crouch'
  },

  anchor_small: {
    id: 'prop_anchor_small',
    polyBudget: 150,
    variants: 3
  },

  // Ship wheel
  ship_wheel: {
    id: 'prop_ship_wheel',
    polyBudget: 200,
    variants: 2,
    interactive: false // Decorative
  },

  // Rope coils
  rope_coil: {
    id: 'prop_rope_coil',
    polyBudget: 100,
    variants: 4,
    sizes: ['small', 'medium', 'large']
  },

  rope_hanging: {
    id: 'prop_rope_hanging',
    polyBudget: 50,
    variants: 3,
    physics: 'simple_sway'
  },

  // Fishing nets
  fishing_net_draped: {
    id: 'prop_fishing_net_draped',
    polyBudget: 200,
    variants: 3,
    decal: true
  },

  fishing_net_hung: {
    id: 'prop_fishing_net_hung',
    polyBudget: 150,
    variants: 2,
    withFish: true
  },

  // Buoys
  buoy: {
    id: 'prop_buoy',
    polyBudget: 80,
    variants: 4,
    colors: ['red', 'green', 'yellow', 'striped'],
    floating: true,
    bobAnimation: true
  },

  // Life preserver
  life_preserver: {
    id: 'prop_life_preserver',
    polyBudget: 50,
    variants: 2,
    wallMount: true
  }
};
```

### Pirate Artifacts

```typescript
const PIRATE_ARTIFACTS = {
  // Flags and banners
  flag_pirate: {
    id: 'prop_flag_pirate',
    polyBudget: 100,
    variants: 5,
    designs: ['skull_crossbones', 'red_flag', 'black_flag', 'torn', 'faction'],
    physics: 'cloth_simple',
    attachTo: ['pole', 'wall', 'ship']
  },

  banner_tavern: {
    id: 'prop_banner_tavern',
    polyBudget: 80,
    variants: 3,
    physics: 'cloth_simple'
  },

  // Wanted posters
  wanted_poster: {
    id: 'prop_wanted_poster',
    polyBudget: 10,
    variants: 6,
    decal: true,
    wallMount: true
  },

  // Treasure maps (decorative)
  map_table: {
    id: 'prop_map_table',
    polyBudget: 150,
    variants: 2
  },

  map_wall: {
    id: 'prop_map_wall',
    polyBudget: 20,
    decal: true
  },

  // Lanterns
  lantern_hanging: {
    id: 'prop_lantern_hanging',
    polyBudget: 80,
    variants: 3,
    emissive: true,
    lightSource: true,
    swayAnimation: true
  },

  lantern_post: {
    id: 'prop_lantern_post',
    polyBudget: 150,
    variants: 2,
    emissive: true,
    lightSource: true
  },

  lantern_table: {
    id: 'prop_lantern_table',
    polyBudget: 60,
    variants: 2,
    emissive: true
  },

  // Bottles and mugs
  bottle_rum: {
    id: 'prop_bottle_rum',
    polyBudget: 30,
    variants: 4,
    instanced: true
  },

  mug_tankard: {
    id: 'prop_mug_tankard',
    polyBudget: 40,
    variants: 3
  },

  // Weapons rack
  weapons_rack: {
    id: 'prop_weapons_rack',
    polyBudget: 300,
    variants: 3,
    weapons: ['cutlasses', 'pistols', 'mixed']
  },

  // Treasure pile (decorative)
  treasure_pile_small: {
    id: 'prop_treasure_pile_small',
    polyBudget: 200,
    variants: 3,
    emissive: true, // Gold gleam
    shader: 'metallic_gold'
  },

  treasure_pile_large: {
    id: 'prop_treasure_pile_large',
    polyBudget: 500,
    variants: 2,
    isCover: true,
    coverHeight: 'crouch'
  }
};
```

---

## Structural Props

### Docks and Piers

```typescript
const DOCK_PROPS = {
  // Dock planks (modular)
  dock_plank_straight: {
    id: 'prop_dock_plank_straight',
    polyBudget: 100,
    length: 4, // meters
    width: 2,
    tileable: true,
    collision: 'primitive'
  },

  dock_plank_corner: {
    id: 'prop_dock_plank_corner',
    polyBudget: 120,
    variants: 2, // Left/right
    tileable: true
  },

  dock_plank_end: {
    id: 'prop_dock_plank_end',
    polyBudget: 80,
    variants: 2
  },

  // Dock posts
  dock_post: {
    id: 'prop_dock_post',
    polyBudget: 60,
    variants: 3,
    ropeTiePoint: true
  },

  dock_post_with_rope: {
    id: 'prop_dock_post_with_rope',
    polyBudget: 100,
    variants: 2
  },

  // Dock accessories
  dock_cleat: {
    id: 'prop_dock_cleat',
    polyBudget: 20,
    variants: 2
  },

  dock_ladder: {
    id: 'prop_dock_ladder',
    polyBudget: 80,
    climbable: true,
    collision: 'mesh'
  },

  // Gangplank
  gangplank: {
    id: 'prop_gangplank',
    polyBudget: 150,
    variants: 2,
    walkable: true
  }
};
```

### Fences and Barriers

```typescript
const FENCE_PROPS = {
  // Wooden fence
  fence_wood_straight: {
    id: 'prop_fence_wood_straight',
    polyBudget: 80,
    length: 3,
    height: 1.2,
    tileable: true,
    isCover: true,
    coverHeight: 'crouch',
    vaultable: true
  },

  fence_wood_post: {
    id: 'prop_fence_wood_post',
    polyBudget: 30,
    variants: 2
  },

  fence_wood_gate: {
    id: 'prop_fence_wood_gate',
    polyBudget: 120,
    openable: false, // Static
    variants: 2
  },

  // Stone wall
  wall_stone_straight: {
    id: 'prop_wall_stone_straight',
    polyBudget: 150,
    length: 4,
    height: 1.5,
    tileable: true,
    isCover: true,
    coverHeight: 'full'
  },

  wall_stone_corner: {
    id: 'prop_wall_stone_corner',
    polyBudget: 180,
    variants: 2
  },

  wall_stone_damaged: {
    id: 'prop_wall_stone_damaged',
    polyBudget: 200,
    variants: 3,
    isCover: true,
    coverHeight: 'partial' // Gaps
  },

  // Sandbags
  sandbag_wall: {
    id: 'prop_sandbag_wall',
    polyBudget: 200,
    variants: 3,
    isCover: true,
    coverHeight: 'crouch'
  },

  sandbag_corner: {
    id: 'prop_sandbag_corner',
    polyBudget: 150,
    variants: 2,
    isCover: true
  }
};
```

### Bridges and Walkways

```typescript
const BRIDGE_PROPS = {
  // Rope bridge
  rope_bridge_segment: {
    id: 'prop_rope_bridge_segment',
    polyBudget: 200,
    length: 5,
    tileable: true,
    physics: 'sway_subtle',
    collision: 'mesh',
    walkable: true
  },

  rope_bridge_anchor: {
    id: 'prop_rope_bridge_anchor',
    polyBudget: 100,
    variants: 2
  },

  // Wooden bridge
  bridge_wood_segment: {
    id: 'prop_bridge_wood_segment',
    polyBudget: 150,
    length: 4,
    tileable: true,
    collision: 'primitive'
  },

  bridge_wood_railing: {
    id: 'prop_bridge_wood_railing',
    polyBudget: 50,
    length: 4,
    tileable: true
  },

  // Stone bridge
  bridge_stone_arch: {
    id: 'prop_bridge_stone_arch',
    polyBudget: 800,
    length: 10,
    collision: 'mesh'
  }
};
```

### Ladders and Vertical Access

```typescript
const LADDER_PROPS = {
  // Standard ladder
  ladder_wood: {
    id: 'prop_ladder_wood',
    polyBudget: 60,
    height: 3, // meters
    variants: 2,
    climbable: true,
    collision: 'primitive'
  },

  ladder_wood_tall: {
    id: 'prop_ladder_wood_tall',
    polyBudget: 80,
    height: 5,
    climbable: true
  },

  // Rope ladder
  ladder_rope: {
    id: 'prop_ladder_rope',
    polyBudget: 100,
    height: 4,
    climbable: true,
    physics: 'sway_subtle'
  },

  // Ship rigging (climbable)
  rigging_net: {
    id: 'prop_rigging_net',
    polyBudget: 150,
    variants: 2,
    climbable: true
  }
};
```

---

## Interactive Props

### Gameplay Elements

```typescript
const INTERACTIVE_PROPS = {
  // Zipline
  zipline_post_start: {
    id: 'prop_zipline_post_start',
    polyBudget: 150,
    interactionType: 'zipline_mount',
    collision: 'convex'
  },

  zipline_post_end: {
    id: 'prop_zipline_post_end',
    polyBudget: 150,
    collision: 'convex'
  },

  zipline_cable: {
    id: 'prop_zipline_cable',
    polyBudget: 20,
    proceduralMesh: true, // Generated between posts
    shader: 'rope'
  },

  // Jump pad (if included)
  jump_pad: {
    id: 'prop_jump_pad',
    polyBudget: 200,
    interactionType: 'launch',
    vfx: 'jump_pad_glow',
    audio: 'spring_launch',
    collision: 'trigger'
  },

  // Breakable door
  door_breakable: {
    id: 'prop_door_breakable',
    polyBudget: 150,
    variants: 3,
    isDestructible: true,
    health: 100,
    destructionVFX: 'wood_splinter'
  },

  // Loot spawn marker (editor only, invisible in game)
  loot_spawn_marker: {
    id: 'prop_loot_spawn_marker',
    editorOnly: true,
    spawnTypes: ['chest', 'ground_loot', 'ability_pickup']
  }
};
```

---

## Decorative Props

### Ambient Details

```typescript
const DECORATIVE_PROPS = {
  // Birds (animated)
  seagull_perched: {
    id: 'prop_seagull_perched',
    polyBudget: 100,
    variants: 3,
    animation: 'idle_peck',
    fleeOnApproach: true,
    fleeDistance: 5
  },

  seagull_flying: {
    id: 'prop_seagull_flying',
    polyBudget: 80,
    spline: true, // Follows flight path
    instanced: true
  },

  // Crabs (small, skittering)
  crab_small: {
    id: 'prop_crab_small',
    polyBudget: 50,
    animation: 'idle_scuttle',
    fleeOnApproach: true
  },

  // Fish (schools)
  fish_school: {
    id: 'prop_fish_school',
    polyBudget: 200,
    fishCount: 20,
    instanced: true,
    underwater: true,
    spline: true
  },

  // Butterflies
  butterfly_swarm: {
    id: 'prop_butterfly_swarm',
    polyBudget: 50,
    count: 5,
    instanced: true,
    particle: true
  },

  // Fireflies (night/cave)
  firefly_cluster: {
    id: 'prop_firefly_cluster',
    polyBudget: 0, // Particle only
    particle: true,
    emissive: true
  }
};
```

### Environmental Storytelling

```typescript
const STORY_PROPS = {
  // Skeleton (dead pirate)
  skeleton_sitting: {
    id: 'prop_skeleton_sitting',
    polyBudget: 200,
    variants: 3,
    poses: ['against_wall', 'at_table', 'slumped']
  },

  skeleton_lying: {
    id: 'prop_skeleton_lying',
    polyBudget: 180,
    variants: 2
  },

  // Shipwreck pieces
  shipwreck_bow: {
    id: 'prop_shipwreck_bow',
    polyBudget: 1500,
    isCover: true,
    collision: 'mesh'
  },

  shipwreck_hull_section: {
    id: 'prop_shipwreck_hull_section',
    polyBudget: 800,
    variants: 3,
    isCover: true
  },

  shipwreck_mast_fallen: {
    id: 'prop_shipwreck_mast_fallen',
    polyBudget: 400,
    isCover: true,
    climbable: true
  },

  // Camp remnants
  campfire_dead: {
    id: 'prop_campfire_dead',
    polyBudget: 100,
    variants: 2
  },

  campfire_active: {
    id: 'prop_campfire_active',
    polyBudget: 150,
    vfx: 'fire_small',
    lightSource: true,
    audio: 'fire_crackle'
  },

  tent_pirate: {
    id: 'prop_tent_pirate',
    polyBudget: 250,
    variants: 2,
    enterable: false,
    isCover: true
  },

  // Grave markers
  grave_cross: {
    id: 'prop_grave_cross',
    polyBudget: 50,
    variants: 4
  },

  grave_stone: {
    id: 'prop_grave_stone',
    polyBudget: 80,
    variants: 3
  }
};
```

---

## Technical Specifications

### Texture Atlasing

```typescript
const TEXTURE_ATLAS_CONFIG = {
  atlases: [
    {
      name: 'atlas_natural',
      size: 2048,
      contents: ['palm_trees', 'rocks', 'plants'],
      format: { android: 'ETC2', ios: 'ASTC_4x4' }
    },
    {
      name: 'atlas_pirate',
      size: 2048,
      contents: ['barrels', 'crates', 'nautical', 'artifacts'],
      format: { android: 'ETC2', ios: 'ASTC_4x4' }
    },
    {
      name: 'atlas_structures',
      size: 2048,
      contents: ['docks', 'fences', 'bridges', 'ladders'],
      format: { android: 'ETC2', ios: 'ASTC_4x4' }
    },
    {
      name: 'atlas_decorative',
      size: 1024,
      contents: ['animals', 'story_props', 'small_details'],
      format: { android: 'ETC2', ios: 'ASTC_4x4' }
    }
  ],

  // Shared material properties
  materials: {
    wood: { roughness: 0.7, metallic: 0 },
    metal: { roughness: 0.4, metallic: 0.8 },
    stone: { roughness: 0.9, metallic: 0 },
    cloth: { roughness: 0.8, metallic: 0, subsurface: 0.1 },
    gold: { roughness: 0.3, metallic: 1.0 }
  }
};
```

### LOD Configuration

```typescript
const PROP_LOD_CONFIG = {
  // LOD distances by prop size
  small: { // < 1m
    lod0: 0,
    lod1: 15,
    cull: 30
  },
  medium: { // 1-3m
    lod0: 0,
    lod1: 25,
    lod2: 50,
    cull: 80
  },
  large: { // > 3m
    lod0: 0,
    lod1: 40,
    lod2: 80,
    lod3: 120,
    cull: 200
  },

  // Quality tier adjustments
  qualityMultiplier: {
    high: 1.0,
    medium: 0.75,
    low: 0.5
  }
};
```

### Collision Types

```typescript
const COLLISION_GUIDELINES = {
  // Use primitive collision where possible
  primitive: {
    types: ['box', 'sphere', 'capsule', 'cylinder'],
    useFor: ['barrels', 'crates', 'simple_shapes'],
    performance: 'best'
  },

  // Convex hull for irregular but solid shapes
  convex: {
    maxVerts: 32,
    useFor: ['rocks', 'cannons', 'complex_props'],
    performance: 'good'
  },

  // Mesh collision only when necessary
  mesh: {
    useFor: ['climbable_structures', 'complex_walkways'],
    performance: 'expensive',
    simplified: true, // Use lower poly version
    maxTris: 100
  },

  // No collision for decorative
  none: {
    useFor: ['scatter_props', 'particles', 'distant_detail'],
    performance: 'free'
  }
};
```

### Cover System Integration

```typescript
const COVER_SPECIFICATIONS = {
  // Cover height definitions
  heights: {
    crouch: { min: 0.8, max: 1.2 }, // meters
    full: { min: 1.5, max: 3.0 },
    partial: 'variable' // Gaps in cover
  },

  // Visual indicators
  indicators: {
    showCoverIcon: true,
    iconDistance: 20, // meters
    highlightOnAim: true
  },

  // Validated cover props
  validCover: [
    'barrel_standard',
    'barrel_stack',
    'crate_large',
    'crate_stack',
    'boulder_large',
    'rock_medium',
    'wall_stone_straight',
    'sandbag_wall',
    'cannon_deck',
    'driftwood_large'
  ],

  // Cover quality
  quality: {
    solid: { damageReduction: 1.0 },
    partial: { damageReduction: 0.5 },
    penetrable: { damageReduction: 0.25 }
  }
};
```

---

## Placement Guidelines

### Density Rules

```typescript
const PLACEMENT_DENSITY = {
  // Props per 100m²
  beach: {
    palm_trees: 2,
    rocks: 5,
    shells: 15,
    driftwood: 3
  },

  jungle: {
    palm_trees: 8,
    plants: 20,
    rocks: 3,
    vines: 5
  },

  dock: {
    barrels: 6,
    crates: 8,
    nautical: 4,
    rope: 6
  },

  town: {
    barrels: 4,
    crates: 3,
    lanterns: 5,
    decorative: 10
  }
};
```

### Cover Distribution

```typescript
const COVER_DISTRIBUTION = {
  // Minimum cover per combat area
  minCoverPerArea: {
    small: 3,  // 10x10m
    medium: 6, // 20x20m
    large: 12  // 30x30m
  },

  // Cover spacing
  spacing: {
    minBetweenCover: 3, // meters
    maxBetweenCover: 15
  },

  // Cover variety
  requireMixedHeights: true,
  crouchToFullRatio: 0.6 // 60% crouch, 40% full
};
```

---

## Production Checklist

```typescript
const PROP_QA_CHECKLIST = {
  model: [
    'Within poly budget',
    'Clean topology',
    'Correct pivot point',
    'Proper scale (1 unit = 1 meter)',
    'LODs generated',
    'Naming convention followed'
  ],

  texture: [
    'Using correct atlas',
    'UV efficiency > 70%',
    'No stretching/distortion',
    'Proper mip levels',
    'Compressed correctly'
  ],

  collision: [
    'Appropriate type selected',
    'No player catching/snagging',
    'Cover works correctly',
    'Climbable surfaces function',
    'No floating/gaps'
  ],

  placement: [
    'Grounded properly',
    'No intersection with terrain',
    'Rotation variety applied',
    'Scale variation (if applicable)',
    'LOD transitions smooth'
  ],

  performance: [
    'Instancing enabled (where applicable)',
    'Culling distance set',
    'No overdraw issues',
    'Memory within budget'
  ]
};
```

---

## Dependencies

- ART-001: Visual Style Guide (design standards)
- ART-002: Technical Art Specifications (performance budgets)
- ART-007: Map Design Document (placement locations)
- ART-008: Map Greybox/Blockout (scale reference)
- GAME-003: Zone/Storm System (placement considerations)
