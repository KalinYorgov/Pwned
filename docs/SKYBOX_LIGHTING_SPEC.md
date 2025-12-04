# Skybox and Lighting Specification

## Document Information
- **Task ID:** ART-012
- **Priority:** P1
- **Complexity:** M
- **Last Updated:** 2025-12-04
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines the skybox, environmental lighting, and atmospheric effects for Plunderstorm Mobile's battle royale map. The lighting system establishes the visual mood, ensures gameplay clarity, and maintains mobile performance while creating an immersive tropical pirate atmosphere.

### 1.2 Design Goals
- **Golden Hour Tropical Feel:** Warm, inviting atmosphere with dramatic shadows
- **Gameplay Clarity:** Player visibility and readability always prioritized
- **Atmospheric Depth:** Visual layering that enhances the sense of scale
- **Mobile Performance:** Optimized lighting that runs smoothly on target devices
- **Future Extensibility:** Foundation for optional time-of-day system

### 1.3 Art Style Alignment
The lighting should complement the stylized, cartoony aesthetic defined in the Visual Style Guide:
- Bold, saturated colors enhanced by warm lighting
- Clear silhouettes maintained through strategic shadow placement
- Fantasy atmosphere supporting the pirate adventure theme

---

## 2. Skybox System

### 2.1 Skybox Configuration

```typescript
const SKYBOX_CONFIG = {
  type: 'procedural_cubemap',
  resolution: 1024, // Per face
  format: 'ASTC_6x6', // Mobile compressed format

  atmosphere: {
    skyColor: {
      zenith: '#4A90D9', // Bright tropical blue
      horizon: '#FFB366', // Golden hour orange blend
      groundColor: '#2E5A8C' // Deep ocean reflection
    },
    sunsetGradient: {
      enabled: true,
      startAngle: 15, // Degrees from horizon
      colors: ['#FF6B35', '#FF9B4F', '#FFD700', '#87CEEB']
    }
  },

  sun: {
    discSize: 2.5, // Degrees
    discColor: '#FFF5E6',
    coronaSize: 8.0,
    coronaColor: '#FFCC66',
    coronaIntensity: 0.6
  }
};
```

### 2.2 Cloud System

```typescript
const CLOUD_SYSTEM = {
  type: 'billboard_layers',

  layers: [
    {
      name: 'high_cirrus',
      altitude: 'zenith_area',
      texture: 'cloud_cirrus_atlas',
      count: 8,
      scale: { min: 0.3, max: 0.6 },
      opacity: 0.4,
      scrollSpeed: 0.001,
      parallax: true
    },
    {
      name: 'mid_cumulus',
      altitude: 'horizon_band',
      texture: 'cloud_cumulus_atlas',
      count: 12,
      scale: { min: 0.15, max: 0.35 },
      opacity: 0.8,
      scrollSpeed: 0.003,
      parallax: true,
      sunlit: true // Receives golden hour coloring
    },
    {
      name: 'low_atmosphere',
      altitude: 'near_horizon',
      texture: 'cloud_haze_atlas',
      count: 6,
      scale: { min: 0.4, max: 0.8 },
      opacity: 0.3,
      scrollSpeed: 0.005,
      parallax: false
    }
  ],

  textureAtlas: {
    size: 1024,
    cloudVariants: 8,
    format: 'ASTC_4x4_ALPHA'
  },

  animation: {
    globalWindDirection: { x: 1.0, y: 0.0, z: 0.3 },
    turbulence: 0.1,
    morphing: false // Disabled for mobile performance
  }
};
```

### 2.3 Sun Position

```typescript
const SUN_POSITION_CONFIG = {
  // Golden hour positioning (sun low in sky)
  elevation: 25, // Degrees above horizon
  azimuth: 280, // West-southwest (late afternoon feel)

  // Shadow direction derived from sun position
  shadowDirection: {
    x: -0.423,
    y: -0.906,
    z: 0.0
  },

  // Future: Time of day variation
  timeOfDaySupport: {
    enabled: false, // Disabled for MVP
    presets: [
      { name: 'dawn', elevation: 5, azimuth: 90 },
      { name: 'morning', elevation: 35, azimuth: 120 },
      { name: 'noon', elevation: 75, azimuth: 180 },
      { name: 'afternoon', elevation: 45, azimuth: 240 },
      { name: 'golden_hour', elevation: 25, azimuth: 280 }, // Default
      { name: 'sunset', elevation: 5, azimuth: 270 },
      { name: 'night', elevation: -20, azimuth: 0 }
    ]
  }
};
```

### 2.4 Skybox Optimization

```typescript
const SKYBOX_OPTIMIZATION = {
  rendering: {
    renderOrder: 'first', // Background layer
    depthWrite: false,
    depthTest: false,
    cullMode: 'front' // Inside-out rendering
  },

  qualityTiers: {
    low: {
      resolution: 512,
      cloudLayers: 1,
      cloudCount: 4,
      animatedClouds: false
    },
    medium: {
      resolution: 1024,
      cloudLayers: 2,
      cloudCount: 8,
      animatedClouds: true
    },
    high: {
      resolution: 1024,
      cloudLayers: 3,
      cloudCount: 16,
      animatedClouds: true
    }
  },

  memoryBudget: {
    skyboxTexture: '2MB',
    cloudAtlas: '1MB',
    total: '3MB'
  }
};
```

---

## 3. Directional Light (Sun)

### 3.1 Primary Directional Light Configuration

```typescript
const DIRECTIONAL_LIGHT_CONFIG = {
  name: 'sun_light',
  type: 'directional',

  color: {
    value: '#FFF2E0', // Warm golden white
    temperature: 5500, // Kelvin (warm daylight)
    saturation: 0.15
  },

  intensity: {
    value: 1.0,
    unit: 'lux',
    realWorldEquivalent: 100000 // Bright daylight
  },

  direction: {
    // Derived from sun position
    pitch: -25, // Degrees
    yaw: 280
  },

  shadows: {
    enabled: true,
    type: 'cascaded_shadow_map',
    // See Section 6 for shadow configuration
  }
};
```

### 3.2 Light Color by Context

```typescript
const DIRECTIONAL_LIGHT_COLORS = {
  // Main map areas
  default: {
    color: '#FFF2E0',
    intensity: 1.0
  },

  // Beach/coastal areas (brighter, more neutral)
  beach: {
    color: '#FFFAF0',
    intensity: 1.1
  },

  // Jungle/forest (filtered through canopy)
  jungle: {
    color: '#E8F5E0',
    intensity: 0.8
  },

  // Cave entrances (transitional)
  caveEntrance: {
    color: '#FFE4C4',
    intensity: 0.5
  },

  // Interior spaces (use ambient/baked only)
  interior: {
    directionalEnabled: false
  }
};
```

---

## 4. Ambient Light System

### 4.1 Ambient Light Configuration

```typescript
const AMBIENT_LIGHT_CONFIG = {
  type: 'gradient_ambient',

  // Three-color gradient ambient
  skyColor: '#87CEEB', // Light from sky (blue)
  equatorColor: '#FFE4B5', // Horizon bounce (warm)
  groundColor: '#8B7355', // Ground bounce (earth)

  intensity: {
    global: 0.4,
    shadowAreas: 0.6 // Boost in shadowed regions
  },

  // Ambient occlusion
  ao: {
    enabled: true,
    type: 'baked', // Baked into lightmaps
    intensity: 0.8,
    radius: 1.5 // World units
  }
};
```

### 4.2 Shadow Fill Light

```typescript
const SHADOW_FILL_CONFIG = {
  // Secondary light to soften shadows
  fillLight: {
    enabled: true,
    type: 'directional',
    color: '#B4D7FF', // Cool blue (sky reflection)
    intensity: 0.25,
    direction: {
      // Opposite-ish to sun for fill
      pitch: -45,
      yaw: 100
    },
    shadowsEnabled: false // No additional shadows
  },

  // Bounce light simulation
  bounceLight: {
    enabled: true,
    type: 'hemisphere',
    skyColor: '#ADD8E6',
    groundColor: '#DEB887',
    intensity: 0.2
  }
};
```

### 4.3 Dynamic Ambient Adjustment

```typescript
const DYNAMIC_AMBIENT_CONFIG = {
  // Adjust ambient based on player context
  contextualAdjustment: {
    enabled: true,

    transitions: {
      outdoor_to_covered: {
        duration: 0.5, // Seconds
        intensityMultiplier: 0.7,
        colorShift: '#E8E0D8'
      },
      outdoor_to_interior: {
        duration: 1.0,
        intensityMultiplier: 0.4,
        colorShift: '#D4C4B0'
      }
    }
  },

  // Area-based ambient zones
  ambientZones: [
    { type: 'outdoor', multiplier: 1.0 },
    { type: 'covered', multiplier: 0.75 }, // Under trees, awnings
    { type: 'shaded', multiplier: 0.6 }, // Alleys, cliffs
    { type: 'interior', multiplier: 0.4 }, // Inside buildings
    { type: 'cave', multiplier: 0.2 } // Deep cave areas
  ]
};
```

---

## 5. Fog and Atmospheric Effects

### 5.1 Distance Fog Configuration

```typescript
const FOG_CONFIG = {
  type: 'exponential_squared',

  // Primary fog (distance fade)
  distanceFog: {
    enabled: true,
    color: '#C9DDE8', // Desaturated sky blue
    density: 0.0008,
    startDistance: 100, // Meters
    endDistance: 500, // Full fog
    heightFalloff: 0.1 // Less fog at altitude
  },

  // Horizon fog (atmospheric perspective)
  horizonFog: {
    enabled: true,
    color: '#FFE4C4', // Warm horizon haze
    intensity: 0.3,
    blendHeight: 50 // Meters above sea level
  },

  // Water surface mist
  waterMist: {
    enabled: true,
    color: '#E0F4FF',
    density: 0.002,
    maxHeight: 5, // Meters above water
    opacity: 0.4
  }
};
```

### 5.2 Height Fog

```typescript
const HEIGHT_FOG_CONFIG = {
  type: 'volumetric_height',

  layers: [
    {
      name: 'sea_level_mist',
      baseHeight: 0, // Sea level
      height: 8, // Fog thickness in meters
      density: 0.015,
      color: '#B8D4E8',
      scattering: 0.3
    },
    {
      name: 'valley_fog',
      baseHeight: -5, // Below sea level (valleys)
      height: 15,
      density: 0.02,
      color: '#A8C8D8',
      scattering: 0.4
    }
  ],

  sunInteraction: {
    enabled: true,
    scatteringColor: '#FFD89B',
    scatteringIntensity: 0.5,
    anisotropy: 0.6 // Forward scattering
  }
};
```

### 5.3 Fog Optimization

```typescript
const FOG_OPTIMIZATION = {
  qualityTiers: {
    low: {
      distanceFog: true,
      heightFog: false,
      waterMist: false,
      sunScattering: false
    },
    medium: {
      distanceFog: true,
      heightFog: true,
      waterMist: false,
      sunScattering: false
    },
    high: {
      distanceFog: true,
      heightFog: true,
      waterMist: true,
      sunScattering: true
    }
  },

  renderingMethod: {
    low: 'shader_fog', // Simple shader-based
    medium: 'shader_fog',
    high: 'volumetric_approx' // Approximated volumetric
  }
};
```

---

## 6. Shadow System

### 6.1 Real-Time Character Shadows

```typescript
const REALTIME_SHADOW_CONFIG = {
  type: 'cascaded_shadow_map',

  cascades: {
    count: 3, // Mobile-friendly cascade count
    distances: [15, 40, 100], // Meters
    resolution: [1024, 512, 256], // Per cascade
    blendDistance: 2.0 // Cascade blend overlap
  },

  filtering: {
    type: 'pcf_soft', // Percentage Closer Filtering
    samples: 4, // Mobile-optimized
    softness: 2.0, // Penumbra size
    normalBias: 0.02,
    depthBias: 0.005
  },

  // What casts real-time shadows
  casters: [
    'player_characters',
    'npc_monsters',
    'dropped_loot',
    'active_projectiles'
  ],

  // What receives real-time shadows
  receivers: [
    'terrain',
    'static_props',
    'characters'
  ]
};
```

### 6.2 Shadow Quality Tiers

```typescript
const SHADOW_QUALITY_TIERS = {
  low: {
    enabled: true,
    cascadeCount: 2,
    cascadeResolution: [512, 256],
    cascadeDistances: [20, 50],
    filtering: 'hard', // No PCF
    characterShadowsOnly: true
  },

  medium: {
    enabled: true,
    cascadeCount: 3,
    cascadeResolution: [1024, 512, 256],
    cascadeDistances: [15, 40, 80],
    filtering: 'pcf_2tap',
    characterShadowsOnly: false
  },

  high: {
    enabled: true,
    cascadeCount: 4,
    cascadeResolution: [2048, 1024, 512, 256],
    cascadeDistances: [10, 30, 60, 120],
    filtering: 'pcf_4tap',
    characterShadowsOnly: false
  }
};
```

### 6.3 Soft Shadow Implementation

```typescript
const SOFT_SHADOW_CONFIG = {
  // No harsh shadow edges
  edgeSoftening: {
    enabled: true,
    method: 'screen_space_blur',
    blurRadius: 3, // Pixels
    fadeDistance: {
      start: 30, // Meters - shadow starts fading
      end: 50 // Meters - shadow fully faded
    }
  },

  // Contact shadows for small details
  contactShadows: {
    enabled: true, // Medium+ quality only
    maxDistance: 0.5, // Meters
    thickness: 0.1,
    samples: 4
  },

  // Shadow color (not pure black)
  shadowColor: {
    tint: '#1A2A3A', // Dark blue tint
    transparency: 0.7 // Allow some light through
  }
};
```

---

## 7. Baked Lighting System

### 7.1 Lightmap Configuration

```typescript
const LIGHTMAP_CONFIG = {
  // Static object lighting
  resolution: {
    terrain: 4, // Texels per meter
    buildings: 8,
    largeProps: 4,
    smallProps: 2
  },

  format: 'RGBM', // HDR-capable mobile format
  compression: 'ASTC_4x4',

  maxAtlasSize: 2048,
  padding: 2, // Texels between charts

  // What gets baked lighting
  bakedObjects: [
    'terrain_mesh',
    'buildings',
    'static_props',
    'large_rocks',
    'vegetation' // Trees, bushes
  ]
};
```

### 7.2 Light Probe System

```typescript
const LIGHT_PROBE_CONFIG = {
  // For dynamic objects in baked environments
  type: 'spherical_harmonics',
  order: 2, // L2 spherical harmonics (9 coefficients)

  placement: {
    gridSpacing: 10, // Meters
    heightLayers: 3, // Ground, mid, high
    adaptiveDensity: true, // More probes near POIs
  },

  blending: {
    method: 'tetrahedral',
    interpolation: 'smooth'
  },

  // Probe groups for streaming
  groups: [
    { name: 'port_plunder', probeCount: 150 },
    { name: 'skull_island', probeCount: 100 },
    { name: 'treasure_cove', probeCount: 80 },
    { name: 'wilderness', probeCount: 200 }
  ]
};
```

### 7.3 Baking Pipeline

```typescript
const BAKE_PIPELINE = {
  // Baking settings
  bounces: 3, // Light bounces
  samples: 512, // Per texel

  // Contribution sources
  lightSources: [
    {
      type: 'directional',
      name: 'sun',
      contribution: 0.7
    },
    {
      type: 'ambient',
      name: 'sky',
      contribution: 0.25
    },
    {
      type: 'emissive',
      name: 'emissive_props',
      contribution: 0.05
    }
  ],

  // Post-processing
  postProcess: {
    denoising: true,
    dilation: 2, // Texels - prevent seams
    aoMultiplier: 0.8
  }
};
```

---

## 8. Map-Wide Lighting Consistency

### 8.1 Lighting Zones

```typescript
const LIGHTING_ZONES = {
  // Ensure consistent lighting across entire map
  globalConsistency: {
    sunDirection: 'locked', // Same everywhere
    shadowDirection: 'locked',
    ambientColor: 'consistent' // Minor variations only
  },

  zones: [
    {
      id: 'beach_coast',
      ambientMultiplier: 1.1,
      fogDensity: 0.0006,
      colorGrading: 'warm_saturated'
    },
    {
      id: 'jungle_forest',
      ambientMultiplier: 0.8,
      fogDensity: 0.001,
      colorGrading: 'cool_green',
      canopyShadow: true
    },
    {
      id: 'mountain_highlands',
      ambientMultiplier: 1.0,
      fogDensity: 0.0004,
      colorGrading: 'neutral_clear'
    },
    {
      id: 'port_town',
      ambientMultiplier: 0.9,
      fogDensity: 0.0008,
      colorGrading: 'warm_nostalgic'
    },
    {
      id: 'cave_interior',
      ambientMultiplier: 0.3,
      fogDensity: 0.002,
      colorGrading: 'cool_mysterious',
      usePointLights: true
    }
  ],

  transitions: {
    blendDistance: 15, // Meters
    blendCurve: 'smooth_step'
  }
};
```

### 8.2 Point of Interest Lighting

```typescript
const POI_LIGHTING = {
  // Special lighting setups for named locations
  portPlunder: {
    additionalLights: [
      {
        type: 'point',
        position: 'tavern_interior',
        color: '#FFB347',
        intensity: 2.0,
        range: 15,
        shadows: false
      },
      {
        type: 'spot',
        position: 'lighthouse',
        color: '#FFFACD',
        intensity: 3.0,
        range: 50,
        angle: 30,
        animated: true
      }
    ]
  },

  skullIsland: {
    additionalLights: [
      {
        type: 'point',
        position: 'skull_eyes',
        color: '#FF4444',
        intensity: 1.5,
        range: 20,
        flicker: true
      }
    ]
  },

  treasureCove: {
    additionalLights: [
      {
        type: 'point',
        position: 'treasure_glow',
        color: '#FFD700',
        intensity: 2.0,
        range: 10,
        animated: 'pulse'
      }
    ]
  }
};
```

### 8.3 Interior Lighting

```typescript
const INTERIOR_LIGHTING = {
  // Transition from outdoor to indoor
  transitionSettings: {
    entryFadeDuration: 0.5, // Seconds
    ambientReduction: 0.6, // Multiplier
    fogDensityIncrease: 2.0
  },

  // Standard interior light kit
  interiorLightKit: {
    window: {
      type: 'area',
      color: '#FFF5E0',
      intensity: 1.5,
      castShadows: true
    },
    torch: {
      type: 'point',
      color: '#FF9933',
      intensity: 1.2,
      range: 8,
      flicker: {
        enabled: true,
        frequency: 5,
        amplitude: 0.3
      }
    },
    lantern: {
      type: 'point',
      color: '#FFB366',
      intensity: 0.8,
      range: 5,
      flicker: {
        enabled: true,
        frequency: 3,
        amplitude: 0.15
      }
    },
    candle: {
      type: 'point',
      color: '#FFA500',
      intensity: 0.4,
      range: 3,
      flicker: {
        enabled: true,
        frequency: 8,
        amplitude: 0.4
      }
    }
  }
};
```

---

## 9. Performance Optimization

### 9.1 Performance Budgets

```typescript
const LIGHTING_PERFORMANCE_BUDGET = {
  // Target frame time allocation
  frameTimeBudget: {
    shadowRendering: '2ms',
    lightingCalculation: '1.5ms',
    fogRendering: '0.5ms',
    total: '4ms' // At 60 FPS = 16.67ms frame budget
  },

  // Draw call budget
  drawCalls: {
    shadowPasses: 3, // One per cascade
    lightingPasses: 1,
    fogPass: 1
  },

  // Memory budget
  memory: {
    shadowMaps: '8MB',
    lightmaps: '16MB',
    lightProbes: '2MB',
    skybox: '3MB',
    total: '29MB'
  }
};
```

### 9.2 Level of Detail Settings

```typescript
const LIGHTING_LOD = {
  shadowLOD: {
    distance0_30m: {
      cascadeResolution: 'full',
      filtering: 'full'
    },
    distance30_60m: {
      cascadeResolution: 'half',
      filtering: 'reduced'
    },
    distance60plus: {
      cascadeResolution: 'quarter',
      filtering: 'none' // Hard shadows or disabled
    }
  },

  lightProbeLOD: {
    nearPlayer: 'full_interpolation',
    midDistance: 'nearest_probe',
    farDistance: 'ambient_only'
  },

  fogLOD: {
    nearCamera: 'full_volumetric',
    midDistance: 'height_fog_only',
    farDistance: 'distance_fog_only'
  }
};
```

### 9.3 Quality Scalability Matrix

```typescript
const QUALITY_SCALABILITY = {
  // Complete quality tier configurations
  ultra_low: {
    realtimeShadows: false,
    bakedShadowsOnly: true,
    shadowMapSize: 0,
    lightProbes: false,
    ambientOnly: true,
    fog: 'simple',
    skybox: 'static_512',
    targetDevices: ['budget_android']
  },

  low: {
    realtimeShadows: true,
    cascadeCount: 2,
    shadowMapSize: 512,
    lightProbes: true,
    probeBlending: 'nearest',
    fog: 'distance_only',
    skybox: 'static_512',
    targetDevices: ['low_end_mobile']
  },

  medium: {
    realtimeShadows: true,
    cascadeCount: 3,
    shadowMapSize: 1024,
    softShadows: true,
    lightProbes: true,
    probeBlending: 'tetrahedral',
    fog: 'height_distance',
    skybox: 'animated_1024',
    targetDevices: ['mid_range_mobile']
  },

  high: {
    realtimeShadows: true,
    cascadeCount: 4,
    shadowMapSize: 2048,
    softShadows: true,
    contactShadows: true,
    lightProbes: true,
    probeBlending: 'smooth',
    fog: 'volumetric_approx',
    skybox: 'animated_1024',
    sunScattering: true,
    targetDevices: ['high_end_mobile', 'tablets']
  }
};
```

---

## 10. Shader Specifications

### 10.1 Standard Lit Shader

```glsl
// Mobile-optimized standard shader features
SHADER_FEATURES = {
  lighting: {
    directionalLight: true,
    maxPointLights: 2, // Per object
    maxSpotLights: 1,
    lightmapSupport: true,
    lightProbeSupport: true
  },

  shadows: {
    receiveShadows: true,
    shadowCascades: 3,
    softShadowSamples: 4
  },

  fog: {
    distanceFog: true,
    heightFog: 'medium_plus', // Quality tier dependent
    fogMode: 'exp2'
  },

  optimization: {
    lowPrecision: true, // Use half precision where safe
    approximateFunctions: true, // Faster math
    branchless: true // Avoid dynamic branching
  }
};
```

### 10.2 Character Shader Lighting

```typescript
const CHARACTER_SHADER_LIGHTING = {
  // Specialized lighting for characters
  subsurfaceScattering: {
    enabled: false, // Too expensive for mobile
  },

  rimLight: {
    enabled: true,
    color: '#FFFFFF',
    intensity: 0.3,
    power: 3.0,
    fresnel: true
  },

  characterAO: {
    enabled: true,
    selfShadow: true,
    vertexAO: true
  },

  shadowReceive: {
    selfShadow: true,
    environmentShadow: true,
    shadowStrength: 0.8
  }
};
```

---

## 11. Time of Day System (Future)

### 11.1 Time of Day Framework

```typescript
const TIME_OF_DAY_FRAMEWORK = {
  // Disabled for MVP, ready for future implementation
  enabled: false,

  architecture: {
    type: 'preset_blend',
    presetCount: 7,
    transitionDuration: 30, // Seconds real-time

    presets: [
      { time: 0.0, name: 'dawn' },
      { time: 0.15, name: 'morning' },
      { time: 0.35, name: 'noon' },
      { time: 0.55, name: 'afternoon' },
      { time: 0.7, name: 'golden_hour' }, // Current default
      { time: 0.85, name: 'sunset' },
      { time: 1.0, name: 'dusk' }
    ]
  },

  // What changes with time
  affectedProperties: [
    'sun_position',
    'sun_color',
    'sun_intensity',
    'ambient_colors',
    'fog_color',
    'skybox_colors',
    'cloud_colors'
  ],

  // Match-based time (not real-time)
  matchIntegration: {
    startTime: 'golden_hour',
    endTime: 'golden_hour', // Static for MVP
    // Future: progression through match
    progressionEnabled: false
  }
};
```

---

## 12. Testing and Validation

### 12.1 Visual Testing Checklist

```typescript
const VISUAL_TESTING = {
  visibilityTests: [
    'Players visible at max render distance',
    'Enemy silhouettes readable against all backgrounds',
    'Loot items visible on ground',
    'Storm wall clearly visible',
    'POI landmarks recognizable from distance'
  ],

  lightingConsistencyTests: [
    'No harsh lighting transitions between zones',
    'Shadow direction consistent across map',
    'Interior/exterior transitions smooth',
    'No overly dark gameplay areas',
    'No blown-out bright areas'
  ],

  shadowTests: [
    'Character shadows visible but not harsh',
    'No shadow acne or artifacts',
    'Cascade transitions not visible',
    'Shadow fade at distance is smooth',
    'No shadow pop-in issues'
  ],

  atmosphericTests: [
    'Fog provides depth without obscuring gameplay',
    'Skybox looks good from all angles',
    'Clouds animate smoothly',
    'No visible seams or artifacts'
  ]
};
```

### 12.2 Performance Testing Matrix

```typescript
const PERFORMANCE_TESTING = {
  testDevices: [
    { name: 'iPhone SE 2020', tier: 'low', targetFPS: 30 },
    { name: 'Samsung A52', tier: 'medium', targetFPS: 45 },
    { name: 'iPhone 13', tier: 'high', targetFPS: 60 },
    { name: 'iPad Pro', tier: 'high', targetFPS: 60 }
  ],

  testScenarios: [
    {
      name: 'open_area',
      description: 'Beach with clear sky',
      expectedLoad: 'light'
    },
    {
      name: 'dense_foliage',
      description: 'Jungle with many shadows',
      expectedLoad: 'heavy'
    },
    {
      name: 'poi_interior',
      description: 'Inside building at Port Plunder',
      expectedLoad: 'medium'
    },
    {
      name: 'combat_heavy',
      description: '10 players fighting with abilities',
      expectedLoad: 'heavy'
    }
  ],

  metrics: [
    'frame_time_ms',
    'shadow_render_time_ms',
    'lighting_pass_time_ms',
    'gpu_memory_usage_mb',
    'cpu_lighting_time_ms'
  ]
};
```

### 12.3 Acceptance Criteria Validation

| Criterion | Test Method | Pass Condition |
|-----------|-------------|----------------|
| Skybox with clouds, sun position | Visual inspection | Clouds visible, sun positioned for golden hour |
| Directional light for sun | Shadow check | Consistent shadows across map |
| Ambient light for shadow areas | Shadow brightness test | Shadows not pure black, detail visible |
| Fog for depth and distant fade | Distance view | Distant objects fade naturally |
| Baked lighting for static objects | Performance test | No realtime cost for static lighting |
| Real-time shadows for characters | Runtime check | Character shadows move with characters |
| No harsh shadow edges | Visual inspection | Soft shadow transitions |
| Consistent lighting across map | Full map tour | No jarring transitions |

---

## 13. Implementation Checklist

### 13.1 Phase 1: Core Lighting Setup
- [ ] Configure primary directional light (sun)
- [ ] Set up ambient/hemisphere lighting
- [ ] Implement basic fog system
- [ ] Create initial skybox

### 13.2 Phase 2: Shadow System
- [ ] Implement cascaded shadow maps
- [ ] Configure shadow quality tiers
- [ ] Add soft shadow filtering
- [ ] Test shadow performance

### 13.3 Phase 3: Baked Lighting
- [ ] Set up lightmap baking pipeline
- [ ] Bake terrain lightmaps
- [ ] Bake POI lightmaps
- [ ] Configure light probe grid

### 13.4 Phase 4: Polish and Optimization
- [ ] Fine-tune fog colors and density
- [ ] Adjust cloud animation
- [ ] Optimize for all quality tiers
- [ ] Performance testing pass

### 13.5 Phase 5: Validation
- [ ] Visual quality review
- [ ] Performance benchmarks
- [ ] Cross-device testing
- [ ] Final approval

---

## 14. Asset Deliverables

### 14.1 Texture Assets
| Asset | Resolution | Format | Size |
|-------|------------|--------|------|
| Skybox cubemap | 1024 per face | ASTC 6x6 | ~2MB |
| Cloud atlas | 1024x1024 | ASTC 4x4 Alpha | ~1MB |
| Sun/corona texture | 256x256 | ASTC 4x4 | ~64KB |

### 14.2 Configuration Files
- `lighting_settings.json` - Global lighting parameters
- `lighting_zones.json` - Zone-specific settings
- `shadow_config.json` - Shadow system settings
- `fog_config.json` - Fog parameters
- `quality_presets.json` - Quality tier configurations

### 14.3 Documentation
- Lighting artist guide
- Performance tuning guide
- Zone setup instructions
- Time-of-day extension guide (for future)

---

## Appendix A: Color Reference

### Golden Hour Palette
| Element | Hex | RGB | Description |
|---------|-----|-----|-------------|
| Sun color | #FFF2E0 | 255, 242, 224 | Warm white |
| Sky zenith | #4A90D9 | 74, 144, 217 | Tropical blue |
| Sky horizon | #FFB366 | 255, 179, 102 | Orange blend |
| Shadow tint | #1A2A3A | 26, 42, 58 | Cool blue |
| Fog color | #C9DDE8 | 201, 221, 232 | Atmospheric haze |
| Ambient sky | #87CEEB | 135, 206, 235 | Sky blue |
| Ambient ground | #8B7355 | 139, 115, 85 | Earth tone |

---

## Appendix B: Reference Images

### Target Mood
- Sea of Thieves golden hour sailing
- Fortnite tropical paradise locations
- Legend of Zelda: Wind Waker ocean scenes
- Overwatch Havana/Rialto warm lighting

### Technical Reference
- Unity HDRP mobile lighting samples
- Unreal mobile rendering documentation
- Mobile shadow cascade implementations
