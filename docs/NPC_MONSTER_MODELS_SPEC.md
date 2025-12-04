# NPC Monster Models Specification

## Overview

This specification defines the monster models for PvE camps throughout the battle royale map. These monsters provide optional combat encounters that reward loot and abilities, adding strategic depth to matches. Each monster type must be visually distinct, thematically appropriate, and optimized for mobile performance.

## Design Principles

### Core Requirements

1. **Instant Recognition**: Players identify monster type and threat at a glance
2. **Pirate Theme**: All monsters fit the pirate fantasy aesthetic
3. **Visual Hierarchy**: Size and complexity indicate difficulty
4. **Mobile Optimization**: Support 20+ monsters on screen at 60 FPS
5. **Animation Clarity**: Attack telegraphs readable for counterplay

### Monster Difficulty Tiers

```typescript
enum MonsterTier {
  MINION = 'minion',     // Easy, quick kills
  STANDARD = 'standard', // Moderate challenge
  ELITE = 'elite',       // Significant threat
  MINI_BOSS = 'mini_boss' // High risk, high reward
}

interface MonsterDefinition {
  id: string;
  name: string;
  tier: MonsterTier;
  polyBudget: number;
  textureBudget: number;
  animationSet: string[];
  spawnLocations: string[];
  lootTable: string;
}
```

---

## Monster Collection

### 1. Skeleton Pirate

**Tier:** Standard (Basic Melee Enemy)

**Lore:**
*The cursed remains of fallen pirates, bound to guard their buried treasure for eternity. They shamble endlessly, cutlass in hand, attacking any who dare approach their domain.*

**Visual Design:**
- Humanoid skeleton with pirate attire remnants
- Tattered clothing (vest, bandana, boots)
- Rusty cutlass weapon
- Glowing eye sockets (blue-green)
- Barnacles and seaweed growth on bones
- Varied accessories (eyepatch, hook hand, peg leg variants)

**Color Palette:**
- Primary: Bone ivory (#e8e4d9)
- Secondary: Tattered brown (#5c4033)
- Accent: Ghostly teal (#20b2aa)
- Metal: Rusty iron (#8b4513)

**Technical Specifications:**
```typescript
const SKELETON_PIRATE: MonsterDefinition = {
  id: 'monster_skeleton_pirate',
  name: 'Skeleton Pirate',
  tier: 'standard',
  polyBudget: 2000,
  textureBudget: 512, // 512x512
  animationSet: [
    'idle',
    'idle_variant',
    'walk',
    'run',
    'attack_slash',
    'attack_thrust',
    'hurt',
    'death',
    'spawn_rise' // Rising from ground
  ],
  spawnLocations: ['shipwrecks', 'beaches', 'caves', 'graveyards'],
  lootTable: 'common_monster_loot'
};
```

**Model Breakdown:**
| Component | Triangles | Notes |
|-----------|-----------|-------|
| Skull | 300 | Expressive eye sockets |
| Torso | 400 | Ribcage visible |
| Arms | 300 | Includes weapon |
| Legs | 350 | Peg leg variant |
| Clothing | 450 | Tattered vest/hat |
| Accessories | 200 | Variable |
| **Total** | **2000** | |

**Variants:**
```typescript
const SKELETON_VARIANTS = [
  {
    id: 'skeleton_pirate_basic',
    accessories: ['bandana'],
    weapon: 'rusty_cutlass'
  },
  {
    id: 'skeleton_pirate_captain',
    accessories: ['tricorn_hat', 'eyepatch'],
    weapon: 'ornate_saber',
    isElite: true,
    polyBudget: 2500
  },
  {
    id: 'skeleton_pirate_hookhand',
    accessories: ['hook_hand', 'bandana'],
    weapon: 'rusty_cutlass'
  },
  {
    id: 'skeleton_pirate_pegleg',
    accessories: ['peg_leg', 'eyepatch'],
    weapon: 'rusty_cutlass'
  }
];
```

**Animation Requirements:**
- **Idle**: Swaying slightly, head scanning
- **Walk**: Shambling gait, uneven steps
- **Run**: Aggressive charge, arms raised
- **Attack Slash**: Wide horizontal swing (0.8s)
- **Attack Thrust**: Quick forward stab (0.5s)
- **Hurt**: Stagger back, bones rattle
- **Death**: Collapse into pile, bones scatter
- **Spawn**: Rise from ground dramatically

---

### 2. Giant Crab

**Tier:** Elite (Medium Tank Enemy)

**Lore:**
*Massive crustaceans mutated by cursed waters, these armored beasts guard tide pools and rocky shores. Their shells deflect most attacks, but their soft underbelly reveals their weakness.*

**Visual Design:**
- Oversized crab (2-3x player height when standing)
- Thick armored shell with barnacles
- Massive front claws (asymmetric sizes)
- Multiple eyes on stalks
- Colorful shell patterns (blues, oranges)
- Seaweed and coral growths

**Color Palette:**
- Primary: Shell red-orange (#cd5c5c)
- Secondary: Deep blue (#1e3a5f)
- Accent: Coral pink (#f08080)
- Highlights: Pearl white (#faf0e6)

**Technical Specifications:**
```typescript
const GIANT_CRAB: MonsterDefinition = {
  id: 'monster_giant_crab',
  name: 'Giant Crab',
  tier: 'elite',
  polyBudget: 3000,
  textureBudget: 512,
  animationSet: [
    'idle',
    'walk_sideways',
    'walk_forward',
    'attack_claw_left',
    'attack_claw_right',
    'attack_slam',
    'defend_shell', // Retreats into shell
    'hurt',
    'death',
    'burrow_emerge'
  ],
  spawnLocations: ['beaches', 'tide_pools', 'caves', 'docks'],
  lootTable: 'elite_monster_loot'
};
```

**Model Breakdown:**
| Component | Triangles | Notes |
|-----------|-----------|-------|
| Shell/Body | 1200 | Detailed armor plates |
| Large Claw | 500 | Primary attack |
| Small Claw | 350 | Secondary attack |
| Legs (8) | 600 | 75 each |
| Eyes/Face | 200 | Expressive stalks |
| Details | 150 | Barnacles, seaweed |
| **Total** | **3000** | |

**Combat Mechanics Influence:**
```typescript
const CRAB_COMBAT_VISUALS = {
  // Shell provides damage reduction from front
  armorZones: {
    shell: { damageReduction: 0.5, visualFeedback: 'sparks' },
    underbelly: { damageReduction: 0, visualFeedback: 'blood_splash' },
    claws: { damageReduction: 0.25 }
  },

  // Attack patterns visible in animations
  attacks: {
    clawSwipe: {
      telegraph: 0.6, // seconds
      visualCue: 'claw_raise_glow',
      damage: 'high'
    },
    groundSlam: {
      telegraph: 1.0,
      visualCue: 'both_claws_raise',
      damage: 'very_high',
      aoe: true
    }
  },

  // Defensive behavior
  shellDefense: {
    trigger: 'low_health',
    animation: 'retreat_into_shell',
    vulnerability: 'rear_only'
  }
};
```

**Animation Requirements:**
- **Idle**: Subtle claw movements, eye stalks scanning
- **Walk Sideways**: Classic crab movement
- **Walk Forward**: Aggressive advance toward player
- **Claw Attack**: Wide sweeping arc, clear telegraph
- **Slam**: Both claws raised, ground pound
- **Shell Defense**: Tuck in, expose nothing
- **Death**: Flip over, legs curl, shell cracks

---

### 3. Cursed Sailor

**Tier:** Standard (Ranged Ghostly Enemy)

**Lore:**
*Souls of drowned sailors, forever trapped between life and death. They hurl spectral energy at intruders, their mournful wails echoing across the waters.*

**Visual Design:**
- Semi-transparent ghostly humanoid
- Tattered naval uniform (waterlogged)
- Ethereal glow with dripping water effects
- Distorted, anguished face
- Chains and anchor weight accessories
- Trailing spectral mist

**Color Palette:**
- Primary: Spectral blue (#4682b4)
- Secondary: Ghostly white (#f0f8ff)
- Accent: Sickly green (#90ee90)
- Glow: Ethereal cyan (#00ced1)

**Technical Specifications:**
```typescript
const CURSED_SAILOR: MonsterDefinition = {
  id: 'monster_cursed_sailor',
  name: 'Cursed Sailor',
  tier: 'standard',
  polyBudget: 2500,
  textureBudget: 512,
  animationSet: [
    'idle_float',
    'move_float',
    'move_fast',
    'attack_throw',
    'attack_scream', // AoE fear
    'hurt',
    'death_dissolve',
    'spawn_materialize'
  ],
  spawnLocations: ['shipwrecks', 'lighthouses', 'docks', 'ghost_ships'],
  lootTable: 'common_monster_loot'
};
```

**Model Breakdown:**
| Component | Triangles | Notes |
|-----------|-----------|-------|
| Body | 800 | Semi-transparent mesh |
| Head/Face | 400 | Expressive anguish |
| Arms | 400 | Throwing poses |
| Legs (fading) | 300 | Dissolves to mist |
| Clothing | 400 | Waterlogged uniform |
| Accessories | 200 | Chains, anchor |
| **Total** | **2500** | |

**VFX Integration:**
```typescript
const CURSED_SAILOR_VFX = {
  // Constant effects
  ambient: {
    ghostlyAura: {
      type: 'particle_system',
      particles: 15,
      color: '#00ced1',
      opacity: 0.6
    },
    waterDrip: {
      type: 'particle_system',
      particles: 5,
      rate: 2 // per second
    }
  },

  // Attack effects
  attacks: {
    spectralBolt: {
      projectile: 'ghost_orb',
      trail: 'ethereal_wisp',
      impact: 'ghost_splash'
    },
    terrorScream: {
      effect: 'expanding_ring',
      color: '#90ee90',
      range: 5 // meters
    }
  },

  // Death
  death: {
    effect: 'dissolve_particles',
    duration: 1.5,
    linger: 0.5
  }
};
```

**Animation Requirements:**
- **Idle Float**: Hovering slightly, swaying
- **Move Float**: Drifting movement, not walking
- **Attack Throw**: Wind up, hurl spectral bolt
- **Scream**: Head back, arms wide, AoE attack
- **Death**: Dissolve into mist particles

**Shader Requirements:**
- Transparency with ghostly fresnel effect
- Animated UV for flowing spectral texture
- Rim lighting for ethereal glow
- Dissolve shader for death animation

---

### 4. Kraken Tentacle (Mini-Boss)

**Tier:** Mini-Boss (Large Animated Obstacle)

**Lore:**
*A single massive tentacle of the legendary Kraken, breaking through the surface to crush any who venture too close. The beast beneath remains unseen, but its power is undeniable.*

**Visual Design:**
- Massive squid/octopus tentacle
- Emerges from water/ground
- Suction cups with teeth
- Bioluminescent patterns
- Slimy, wet texture
- Scarred from previous battles

**Color Palette:**
- Primary: Deep purple (#4b0082)
- Secondary: Slimy green (#3cb371)
- Accent: Bioluminescent blue (#00bfff)
- Suckers: Pale pink (#dda0dd)

**Technical Specifications:**
```typescript
const KRAKEN_TENTACLE: MonsterDefinition = {
  id: 'monster_kraken_tentacle',
  name: 'Kraken Tentacle',
  tier: 'mini_boss',
  polyBudget: 5000,
  textureBudget: 1024, // Higher for boss
  animationSet: [
    'idle_sway',
    'emerge',
    'attack_slam',
    'attack_sweep',
    'attack_grab',
    'hurt_recoil',
    'death_retreat',
    'submerge'
  ],
  spawnLocations: ['kraken_cove', 'deep_harbor', 'sea_caves'],
  lootTable: 'boss_monster_loot'
};
```

**Model Breakdown:**
| Component | Triangles | Notes |
|-----------|-----------|-------|
| Tentacle Base | 1500 | Where it emerges |
| Mid Section | 1500 | Main body |
| Tip Section | 800 | Attack point |
| Suction Cups | 800 | 50 cups @ 16 tri |
| Details | 400 | Scars, barnacles |
| **Total** | **5000** | |

**Scale and Presence:**
```typescript
const KRAKEN_SCALE = {
  height: 15, // meters when fully extended
  baseRadius: 3, // meters
  reachRadius: 12, // attack range

  // Segments for animation
  segments: 8,
  boneChain: 12,

  // Environmental interaction
  causesWaves: true,
  destroysProps: true,
  screenShake: {
    onSlam: { intensity: 0.3, duration: 0.5 },
    onEmerge: { intensity: 0.5, duration: 1.0 }
  }
};
```

**Combat Mechanics Influence:**
```typescript
const KRAKEN_COMBAT_VISUALS = {
  // Attack patterns
  attacks: {
    slam: {
      telegraph: 1.5, // seconds
      visualCue: 'tentacle_raises_high',
      aoeRadius: 4,
      damage: 'massive'
    },
    sweep: {
      telegraph: 1.0,
      visualCue: 'tentacle_coils_back',
      arc: 180, // degrees
      damage: 'high'
    },
    grab: {
      telegraph: 0.8,
      visualCue: 'tip_opens_wide',
      effect: 'player_held', // CC effect
      damage: 'medium_dot'
    }
  },

  // Weak points
  weakPoints: {
    suckerRows: {
      damageMultiplier: 1.5,
      visualFeedback: 'sucker_burst'
    },
    woundedSection: {
      damageMultiplier: 2.0,
      visualFeedback: 'ichor_spray',
      appearsAt: 0.5 // 50% health
    }
  },

  // Phase transitions
  phases: {
    phase1: { health: '100-60%', pattern: ['slam', 'sweep'] },
    phase2: { health: '60-30%', pattern: ['slam', 'sweep', 'grab'] },
    phase3: { health: '30-0%', pattern: ['enraged_flurry'] }
  }
};
```

**Animation Requirements:**
- **Emerge**: Dramatic rise from water/ground (2s)
- **Idle Sway**: Constant subtle movement
- **Slam**: Full extension, massive downward strike
- **Sweep**: Wide horizontal sweep attack
- **Grab**: Tip opens, attempts to catch player
- **Hurt**: Recoil, writhe in pain
- **Death**: Slow retreat back into water/ground

**VFX Integration:**
```typescript
const KRAKEN_VFX = {
  ambient: {
    waterSplash: { rate: 3, scale: 'large' },
    bioluminescence: { pulse: true, frequency: 0.5 },
    slimeTrail: { enabled: true }
  },

  attacks: {
    slamImpact: {
      effect: 'water_explosion',
      particles: 100,
      debris: true,
      screenShake: true
    },
    sweepTrail: {
      effect: 'water_spray',
      followTentacle: true
    }
  },

  damage: {
    ichorSpray: {
      color: '#3cb371',
      particles: 20
    }
  }
};
```

---

## Technical Specifications

### LOD System

```typescript
const MONSTER_LOD_CONFIG = {
  skeleton_pirate: {
    lod0: { distance: 0, polyCount: 2000, textureSize: 512 },
    lod1: { distance: 20, polyCount: 1000, textureSize: 256 },
    lod2: { distance: 40, polyCount: 500, textureSize: 128 },
    cullDistance: 80
  },

  giant_crab: {
    lod0: { distance: 0, polyCount: 3000, textureSize: 512 },
    lod1: { distance: 25, polyCount: 1500, textureSize: 256 },
    lod2: { distance: 50, polyCount: 750, textureSize: 128 },
    cullDistance: 100
  },

  cursed_sailor: {
    lod0: { distance: 0, polyCount: 2500, textureSize: 512 },
    lod1: { distance: 20, polyCount: 1250, textureSize: 256 },
    lod2: { distance: 40, polyCount: 625, textureSize: 128 },
    cullDistance: 80
  },

  kraken_tentacle: {
    lod0: { distance: 0, polyCount: 5000, textureSize: 1024 },
    lod1: { distance: 30, polyCount: 2500, textureSize: 512 },
    lod2: { distance: 60, polyCount: 1250, textureSize: 256 },
    cullDistance: 150 // Boss always visible from far
  }
};
```

### Performance Budgets

```typescript
const MONSTER_PERFORMANCE_BUDGET = {
  // Maximum on-screen counts
  maxCounts: {
    skeleton_pirate: 15,
    giant_crab: 5,
    cursed_sailor: 10,
    kraken_tentacle: 1
  },

  // Combined limits
  maxTotalMonsters: 25,
  maxTotalTriangles: 50000,
  maxSimultaneousAnimations: 20,

  // Quality scaling
  qualityTiers: {
    high: { monsterMultiplier: 1.0, lodBias: 0 },
    medium: { monsterMultiplier: 0.75, lodBias: 10 },
    low: { monsterMultiplier: 0.5, lodBias: 20 }
  }
};
```

### Rig Specifications

```typescript
const MONSTER_RIG_SPECS = {
  skeleton_pirate: {
    boneCount: 35,
    ikChains: ['arm_left', 'arm_right', 'spine'],
    ragdollEnabled: true
  },

  giant_crab: {
    boneCount: 45,
    ikChains: ['claw_left', 'claw_right'],
    legIK: true, // Procedural ground adaptation
    ragdollEnabled: false // Shell stays intact
  },

  cursed_sailor: {
    boneCount: 30,
    ikChains: ['arm_left', 'arm_right'],
    clothBones: 8, // For flowing clothing
    ragdollEnabled: false // Dissolves instead
  },

  kraken_tentacle: {
    boneCount: 50,
    splineIK: true, // Smooth tentacle movement
    proceduralAnimation: true, // Secondary motion
    ragdollEnabled: false
  }
};
```

### Texture Specifications

```typescript
const MONSTER_TEXTURE_SPECS = {
  // Standard monsters
  standard: {
    diffuse: 512,
    normal: 256,
    mask: 256, // Metallic/roughness/emission
    format: {
      android: 'ETC2',
      ios: 'ASTC_4x4'
    }
  },

  // Mini-boss (Kraken)
  miniBoss: {
    diffuse: 1024,
    normal: 512,
    mask: 512,
    subsurface: 256, // For translucent suckers
    format: {
      android: 'ETC2',
      ios: 'ASTC_4x4'
    }
  }
};
```

---

## Animation Guidelines

### Attack Telegraph Standards

```typescript
const ATTACK_TELEGRAPH_STANDARDS = {
  // All attacks must have visual warning
  minimumTelegraph: 0.4, // seconds

  // Telegraph elements
  visualCues: [
    'pose_change',      // Monster changes stance
    'effect_warning',   // VFX indicator
    'audio_cue',        // Sound warning
    'ground_indicator'  // AoE markers
  ],

  // By difficulty
  telegraphDuration: {
    minion: 0.4,
    standard: 0.6,
    elite: 0.8,
    mini_boss: 1.0
  }
};
```

### Animation Priorities

```typescript
const ANIMATION_PRIORITIES = {
  // Highest priority (cannot interrupt)
  death: 100,
  spawn: 95,

  // High priority
  hurt: 80,
  attack: 70,

  // Normal priority
  move: 50,

  // Lowest priority
  idle: 10
};
```

### Blend Times

```typescript
const ANIMATION_BLEND_TIMES = {
  idle_to_walk: 0.2,
  walk_to_run: 0.15,
  any_to_attack: 0.1, // Quick response
  attack_to_idle: 0.3,
  any_to_hurt: 0.05, // Immediate
  any_to_death: 0.1
};
```

---

## Visual Hierarchy

### Threat Indicators

```typescript
const THREAT_VISUAL_HIERARCHY = {
  // Size indicates danger
  sizeScale: {
    minion: 0.8,    // Smaller than player
    standard: 1.0,  // Player-sized
    elite: 1.5,     // Larger
    mini_boss: 3.0  // Massive
  },

  // Color saturation
  colorIntensity: {
    minion: 0.7,
    standard: 0.85,
    elite: 1.0,
    mini_boss: 1.2 // Slightly oversaturated
  },

  // Glow/effects
  effectIntensity: {
    minion: 'none',
    standard: 'subtle',
    elite: 'moderate',
    mini_boss: 'prominent'
  }
};
```

### Silhouette Requirements

```typescript
const SILHOUETTE_REQUIREMENTS = {
  // Must be identifiable at distance
  identificationDistance: 30, // meters

  // Unique shape features
  skeleton_pirate: 'humanoid_with_hat',
  giant_crab: 'wide_low_claws',
  cursed_sailor: 'floating_ghostly',
  kraken_tentacle: 'massive_vertical_tendril',

  // No confusion with
  mustNotResemble: ['player_character', 'other_monsters', 'props']
};
```

---

## Spawn and Behavior Visuals

### Spawn Animations

```typescript
const SPAWN_ANIMATIONS = {
  skeleton_pirate: {
    type: 'rise_from_ground',
    duration: 1.5,
    vfx: 'dirt_particles',
    audio: 'bones_rattling'
  },

  giant_crab: {
    type: 'burrow_emerge',
    duration: 2.0,
    vfx: 'sand_explosion',
    audio: 'chittering_screech'
  },

  cursed_sailor: {
    type: 'materialize',
    duration: 1.0,
    vfx: 'ghost_coalesce',
    audio: 'ethereal_wail'
  },

  kraken_tentacle: {
    type: 'dramatic_emerge',
    duration: 3.0,
    vfx: 'massive_water_splash',
    audio: 'deep_rumble',
    screenShake: true
  }
};
```

### Alert States

```typescript
const ALERT_STATE_VISUALS = {
  // Visual changes when player detected
  passive: {
    eyeGlow: 'dim',
    posture: 'relaxed',
    animation: 'idle_passive'
  },

  alert: {
    eyeGlow: 'bright',
    posture: 'attentive',
    animation: 'idle_alert',
    vfx: 'alert_indicator' // ! above head
  },

  aggressive: {
    eyeGlow: 'intense',
    posture: 'combat_ready',
    animation: 'idle_aggressive',
    vfx: 'aggro_aura'
  }
};
```

---

## Production Pipeline

### Asset Creation Workflow

1. **Concept Phase**
   - Reference gathering
   - Silhouette sketches
   - Color studies
   - Approval checkpoint

2. **Modeling Phase**
   - Base mesh blockout
   - Detail pass
   - UV mapping
   - LOD generation

3. **Texturing Phase**
   - Albedo/diffuse
   - Normal maps
   - Material masks
   - Emission maps (if needed)

4. **Rigging Phase**
   - Skeleton creation
   - Skinning
   - IK setup
   - Facial controls (if applicable)

5. **Animation Phase**
   - Core movement set
   - Combat animations
   - Special states
   - Blend tree setup

6. **VFX Integration**
   - Ambient effects
   - Attack effects
   - Death effects

7. **QA Phase**
   - Performance testing
   - Animation review
   - Visual consistency check

### Quality Checklist

```typescript
const MONSTER_QA_CHECKLIST = {
  model: [
    'Within poly budget',
    'Clean topology for animation',
    'No inverted normals',
    'Proper UV layout',
    'LODs generated correctly'
  ],

  texture: [
    'Correct resolution',
    'Proper compression',
    'No stretching/distortion',
    'Material definition correct'
  ],

  rig: [
    'All bones named correctly',
    'Proper weight painting',
    'No skinning artifacts',
    'IK working correctly'
  ],

  animation: [
    'All required anims present',
    'Smooth transitions',
    'Attack telegraphs clear',
    'Death satisfying',
    'No foot sliding'
  ],

  performance: [
    'FPS target met with max count',
    'LOD transitions smooth',
    'VFX within budget',
    'Memory within limits'
  ]
};
```

---

## Integration with Gameplay

### AI Behavior Visualization

```typescript
const AI_BEHAVIOR_VISUALS = {
  // Patrol
  patrol: {
    animation: 'walk',
    speed: 0.5,
    headLook: 'scanning'
  },

  // Chase
  chase: {
    animation: 'run',
    speed: 1.0,
    eyeGlow: 'intense',
    vfx: 'aggro_particles'
  },

  // Attack
  attack: {
    animation: 'attack_[type]',
    telegraph: 'attack_windup_vfx',
    impact: 'attack_impact_vfx'
  },

  // Return to spawn
  leash: {
    animation: 'walk',
    vfx: 'confusion_particles',
    eyeGlow: 'dimming'
  }
};
```

### Loot Drop Visuals

```typescript
const LOOT_DROP_VISUALS = {
  // Death triggers loot
  deathSequence: [
    { time: 0, event: 'play_death_animation' },
    { time: 0.5, event: 'spawn_loot_vfx' },
    { time: 0.8, event: 'drop_loot_items' },
    { time: 1.5, event: 'fade_corpse' }
  ],

  // Loot quality indication
  lootGlow: {
    common: '#808080',
    uncommon: '#00ff00',
    rare: '#0080ff',
    epic: '#a020f0'
  }
};
```

---

## Dependencies

- ART-001: Visual Style Guide (design standards)
- ART-002: Technical Art Specifications (performance budgets)
- GAME-015: PvE Monster Camps (gameplay integration)
- ART-024: Monster Animations (animation set)
