# Storm/Zone VFX Specification

## Document Information
- **Task ID:** ART-015
- **Priority:** P0
- **Complexity:** M (Medium)
- **Dependencies:** GAME-003 (Storm/Safe Zone System)
- **Last Updated:** 2025-12-01

---

## 1. Overview

### 1.1 Purpose
This specification defines the visual effects for the storm/zone system in Plunderstorm Mobile. The storm is a supernatural curse that shrinks the playable area, forcing player encounters. Clear visual communication is critical for players to understand zone boundaries and make strategic decisions.

### 1.2 Narrative Context

```yaml
storm_lore:
  name: "The Blackstorm Curse"
  description: |
    When the Pirate King Blackstorm opened the forbidden treasure,
    he unleashed a supernatural storm that eternally circles the island.
    The curse manifests as a swirling vortex of dark energy that
    consumes everything outside the safe zone.

  visual_theme:
    style: "Supernatural pirate curse"
    colors: "Purple, black, ghostly blue"
    elements: "Lightning, spectral ships, cursed fog"
```

### 1.3 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Visibility | Storm boundary clear from any distance | Critical |
| Urgency | Communicate danger level effectively | Critical |
| Performance | No FPS drops with full storm render | Critical |
| Atmosphere | Enhance game tension and mood | High |
| Readability | Distinct inside vs outside states | High |
| Theme | Fit pirate supernatural aesthetic | High |

---

## 2. Storm Wall VFX

### 2.1 Primary Storm Wall

```yaml
storm_wall:
  description: "The main visible boundary of the storm"
  visibility_range: "Visible from entire map"

  structure:
    height: "100m (extends into sky)"
    thickness: "15m visual depth"
    shape: "Cylindrical, matches safe zone circle"

  layers:
    outer_layer:
      name: "Fog Boundary"
      depth: "0-5m from edge"
      opacity: "30% -> 60%"
      color: "#1a0a2e"
      behavior: "Slowly swirling"

    mid_layer:
      name: "Core Storm"
      depth: "5-10m"
      opacity: "60% -> 90%"
      color: "#2d1b4e"
      behavior: "Active churning"
      particles: "Debris, spectral effects"

    inner_layer:
      name: "Death Zone"
      depth: "10m+"
      opacity: "90%+"
      color: "#0d0014"
      behavior: "Solid darkness"
      visibility: "< 5m"

  visual_elements:
    swirling_clouds:
      type: "Volumetric scrolling texture"
      scroll_speed: "3m/s horizontal"
      color_primary: "#2d1b4e"
      color_secondary: "#4a2875"
      turbulence: "Medium"

    lightning:
      frequency: "Every 2-5 seconds"
      color: "#9b7fd1"
      duration: 0.2s
      branches: "2-4"
      internal: true (inside storm mass)

    ghostly_ships:
      enabled: true
      count: "3-5 silhouettes visible in storm"
      opacity: 0.3
      movement: "Slow drift, appear/disappear"
      distance: "20-50m inside storm"

    spectral_faces:
      enabled: true
      frequency: "Occasional (every 10-20s)"
      opacity: 0.2
      duration: 2s
      size: "Large (10-20m)"
```

### 2.2 Storm Wall Edge Effects

```yaml
storm_edge:
  description: "The visible boundary line between safe and storm"

  edge_line:
    type: "Glowing boundary"
    color: "#9b7fd1"
    width: "0.5m"
    opacity: 0.8
    animation: "Subtle pulse"
    pulse_speed: 2s

  ground_effect:
    type: "Ground decal/projection"
    color: "#9b7fd1"
    opacity: 0.5
    width: 2m
    pattern: "Rune symbols"
    animation: "Scrolling along edge"

  particle_boundary:
    type: "Rising particles at edge"
    direction: "Upward spiral"
    color: "#7c5cbf"
    density: "Medium"
    height: 5m
    emission: "Continuous along edge"

  audio_zone:
    range: "10m from edge"
    sound: "storm_rumble_loop"
    volume_falloff: "Linear"
```

### 2.3 Storm Wall Vertical Element

```yaml
storm_vertical:
  description: "How the storm appears looking up"

  sky_effect:
    type: "Swirling vortex above"
    center: "Map center (eye of storm)"
    radius: "Matches current safe zone"

    visuals:
      cloud_spiral:
        layers: 3
        rotation_speed: "Slow (30s per rotation)"
        color: "#1a0a2e"

      eye_center:
        visible: true
        color: "#4a2875"
        glow: "Subtle"

      lightning_in_clouds:
        frequency: "Regular"
        color: "#9b7fd1"

  horizon_blend:
    type: "Gradient fade"
    height: "50-100m"
    from: "Storm color"
    to: "Sky color"
```

---

## 3. Inside Storm Effects

### 3.1 Player Inside Storm Experience

```yaml
inside_storm:
  description: "Visual effects when player is in the storm"

  visibility:
    base_visibility: "20m (reduced from normal 100m+)"
    fog_density: "Heavy"
    fog_color: "#1a0a2e"

  screen_effects:
    vignette:
      enabled: true
      color: "#2d1b4e"
      intensity: 0.4
      coverage: "30% from edges"

    color_grade:
      saturation: 0.5
      contrast: 1.2
      tint: "#2d1b4e"

    distortion:
      enabled: true
      type: "Subtle wave"
      intensity: 0.05
      speed: "Slow"

  particle_effects:
    ambient_particles:
      type: "Swirling debris/ash"
      density: "High"
      color: "#4a2875"
      movement: "Chaotic swirl"

    damage_particles:
      type: "Dark energy wisps"
      frequency: "On damage tick"
      color: "#9b7fd1"
      attach_to: "Player"

  post_process:
    grain:
      enabled: true
      intensity: 0.1

    chromatic_aberration:
      enabled: true
      intensity: 0.15
```

### 3.2 Storm Damage Feedback

```yaml
storm_damage:
  description: "Visual feedback when taking storm damage"

  on_damage_tick:
    frequency: "Every 1 second"

    effects:
      screen_flash:
        color: "#9b7fd1"
        intensity: 0.2
        duration: 0.1s

      vignette_pulse:
        intensity_add: 0.1
        duration: 0.3s

      damage_particles:
        type: "Dark energy impact"
        count: 5
        from: "Random direction"
        color: "#2d1b4e"

      character_effect:
        type: "Brief dark aura"
        duration: 0.3s

  damage_number:
    color: "#9b7fd1"
    format: "-[X] STORM"
    position: "Above player"

  audio:
    damage_sound: "storm_damage_tick"
    ambient: "storm_inside_loop"
```

---

## 4. Storm Movement/Shrinking

### 4.1 Storm Closing Animation

```yaml
storm_closing:
  description: "VFX during zone shrink phases"

  shrink_behavior:
    type: "Gradual movement inward"
    speed: "Variable by phase"
    smoothness: "Continuous, not choppy"

  visual_during_shrink:
    edge_intensity:
      increase: 1.5x
      color_shift: "More vibrant purple"

    particle_increase:
      multiplier: 2.0
      direction: "Flowing inward"

    lightning_frequency:
      increase: 2x
      intensity: "Brighter"

    ground_effect:
      type: "Corruption spread"
      speed: "Matches shrink"
      color: "#2d1b4e"
      opacity: 0.3

  special_effects:
    wave_pulse:
      enabled: true
      frequency: "Every 5 seconds during shrink"
      type: "Inward ripple"
      color: "#9b7fd1"
      opacity: 0.5

    debris_increase:
      enabled: true
      multiplier: 3x
      types: ["Leaves", "Papers", "Dust"]
```

### 4.2 Pre-Shrink Warning

```yaml
storm_warning:
  description: "Warning effects before zone shrinks"
  duration: "10 seconds before shrink"

  visual_warnings:
    storm_intensify:
      type: "Storm wall pulses brighter"
      frequency: "Every 2 seconds"
      intensity: 1.3x

    edge_pulse:
      type: "Ground edge glows"
      color: "#FFD700"
      pulse: "Accelerating"

    next_zone_preview:
      type: "Circle outline on map"
      color: "#FFFFFF"
      opacity: 0.5
      dashed: true

  screen_warning:
    text: "STORM CLOSING"
    position: "Top center"
    color: "#9b7fd1"
    size: "Large"
    animation: "Pulse"

  audio:
    warning_sound: "storm_warning"
    rumble_increase: true
```

---

## 5. Safe Zone Indicator

### 5.1 Safe Zone Edge (Inside Looking Out)

```yaml
safe_zone_inside:
  description: "How safe zone appears from inside"

  boundary_marker:
    visibility: "Always visible"
    distance: "See storm wall up to 200m"

    elements:
      glowing_line:
        color: "#9b7fd1"
        opacity: 0.6
        height: "Full storm height"

      distance_markers:
        enabled: true
        interval: 50m
        type: "Pillar of light"
        color: "#7c5cbf"
        opacity: 0.3
```

### 5.2 Minimap Integration

```yaml
minimap_storm:
  description: "Storm representation on minimap"

  current_zone:
    shape: "Circle"
    edge_color: "#9b7fd1"
    edge_width: 3px
    fill: "Transparent"

  storm_area:
    fill_color: "#2d1b4e"
    opacity: 0.5
    pattern: "Solid or light texture"

  next_zone:
    shape: "Circle (dashed)"
    edge_color: "#FFFFFF"
    edge_width: 2px
    visibility: "During warning phase"

  zone_direction:
    arrow: true
    color: "#FFD700"
    visibility: "When zone is off-screen"

  timer:
    position: "Near zone circle"
    format: "0:00"
    color: "#FFFFFF"
    warning_color: "#FF0000"
```

---

## 6. Phase-Specific Effects

### 6.1 Storm Phases

```yaml
storm_phases:
  phase_1:
    name: "Gathering Storm"
    radius: 600m
    damage: 5/sec

    visual_intensity:
      storm_opacity: 0.6
      lightning_frequency: "Low (every 5s)"
      particle_density: "Light"
      color_saturation: 0.7

    mood: "Ominous but distant"

  phase_2:
    name: "Rising Curse"
    radius: 400m
    damage: 8/sec

    visual_intensity:
      storm_opacity: 0.7
      lightning_frequency: "Medium (every 3s)"
      particle_density: "Medium"
      color_saturation: 0.8

    mood: "Threatening"

  phase_3:
    name: "Blackstorm Rising"
    radius: 250m
    damage: 12/sec

    visual_intensity:
      storm_opacity: 0.8
      lightning_frequency: "High (every 2s)"
      particle_density: "High"
      color_saturation: 0.9
      ghostly_ships: "More visible"

    mood: "Dangerous"

  phase_4:
    name: "The Curse Closes"
    radius: 125m
    damage: 18/sec

    visual_intensity:
      storm_opacity: 0.9
      lightning_frequency: "Very high (every 1s)"
      particle_density: "Very high"
      color_saturation: 1.0
      spectral_faces: "More frequent"

    mood: "Urgent"

  phase_5:
    name: "Final Stand"
    radius: 50m
    damage: 25/sec

    visual_intensity:
      storm_opacity: 1.0
      lightning_frequency: "Constant"
      particle_density: "Maximum"
      color_saturation: 1.0
      ground_cracks: true

    mood: "Apocalyptic"

  final_phase:
    name: "Total Curse"
    radius: "Closing to 0"
    damage: 35/sec

    visual_intensity:
      all_max: true
      screen_effects: "Intensified"
      visibility: "Minimal"

    mood: "Inescapable"
```

### 6.2 Final Circle Special Effects

```yaml
final_circle:
  description: "Special effects for endgame small circles"
  trigger: "Circle radius < 50m"

  environmental:
    ground_cracks:
      enabled: true
      pattern: "Radiating from edge"
      color: "#9b7fd1"
      glow: true

    floating_debris:
      increased: true
      types: ["Rocks", "Planks", "Bones"]
      anti_gravity: "Slight lift"

    ambient_screams:
      enabled: true
      volume: "Subtle"
      type: "Ghostly wails"

  atmosphere:
    darkness:
      sky_darken: 0.5
      ambient_light_reduce: 0.3

    wind:
      increased: true
      visual: "Grass/cloth reaction"
      audio: "wind_howl"

  tension_music:
    enabled: true
    track: "final_circle_music"
    intensity: "Building"
```

---

## 7. Technical Specifications

### 7.1 Performance Budget

```yaml
performance_budget:
  storm_wall:
    draw_calls: 3-5
    triangle_count: "< 10,000"
    texture_memory: "2MB max"

  particles:
    storm_edge: "200 max"
    inside_storm: "100 max"
    total: "300 max concurrent"

  shaders:
    complexity: "Medium"
    passes: "Single pass"
    features:
      - "Scrolling cloud texture"
      - "Simple fog"
      - "Edge glow"

  post_process:
    inside_storm_only: true
    budget: "< 1ms GPU"

  total_budget:
    target: "< 3ms GPU total"
    mobile_target: "Maintain 60 FPS"
```

### 7.2 Rendering Approach

```yaml
rendering:
  storm_wall:
    method: "Cylindrical mesh with scrolling texture"
    segments: 64
    update_frequency: "Match zone radius"

    shader:
      name: "Plunderstorm/Storm/Wall"
      features:
        - "UV scrolling (clouds)"
        - "Fresnel edge glow"
        - "Depth fade"
        - "Lightning flash"

  particles:
    method: "GPU particle system"
    pooling: true
    LOD: "Reduce at distance"

  fog:
    method: "Height-based fog"
    mode: "Linear or exponential"
    integration: "Unity fog or custom"

  post_process:
    method: "Post-process volume"
    trigger: "Player enters storm"
    transition: "0.5s blend"
```

### 7.3 LOD System

```yaml
storm_lod:
  LOD0:
    distance: "0-100m"
    storm_detail: "Full"
    particles: 100%
    lightning: "All"
    ghostly_ships: true

  LOD1:
    distance: "100-300m"
    storm_detail: "Reduced"
    particles: 50%
    lightning: "Reduced frequency"
    ghostly_ships: false

  LOD2:
    distance: "300m+"
    storm_detail: "Minimal"
    particles: 25%
    lightning: "Flash only"
    ghostly_ships: false

  quality_settings:
    high:
      LOD_bias: 0
      all_effects: true

    medium:
      LOD_bias: +1
      post_process: "Reduced"

    low:
      LOD_bias: +2
      post_process: "Disabled"
      particles: "Minimal"
```

---

## 8. Asset Deliverables

### 8.1 Texture Assets

```yaml
textures:
  storm_clouds:
    name: "T_Storm_Clouds_Scroll"
    resolution: "512x512"
    format: "ASTC 6x6"
    tiling: true
    channels: "RGB + Alpha"

  storm_noise:
    name: "T_Storm_Noise"
    resolution: "256x256"
    format: "ASTC 6x6"
    tiling: true
    channels: "Grayscale"

  lightning:
    name: "T_Storm_Lightning"
    resolution: "256x512"
    format: "ASTC 4x4"
    sprites: 4

  ground_corruption:
    name: "T_Storm_Ground"
    resolution: "512x512"
    format: "ASTC 6x6"
    tiling: true

  ghost_ships:
    name: "T_Storm_GhostShip"
    resolution: "256x256"
    format: "ASTC 6x6"
    variants: 3
```

### 8.2 Prefab Assets

```yaml
prefabs:
  main:
    - "VFX_Storm_Wall.prefab"
    - "VFX_Storm_Edge.prefab"
    - "VFX_Storm_Interior.prefab"

  particles:
    - "VFX_Storm_Particles_Edge.prefab"
    - "VFX_Storm_Particles_Inside.prefab"
    - "VFX_Storm_Debris.prefab"
    - "VFX_Storm_Lightning.prefab"

  effects:
    - "VFX_Storm_GhostShip.prefab"
    - "VFX_Storm_SpectralFace.prefab"
    - "VFX_Storm_GroundCorruption.prefab"

  post_process:
    - "PP_Storm_Inside.asset"
    - "PP_Storm_Warning.asset"
```

### 8.3 Material Assets

```yaml
materials:
  storm_wall:
    - "M_Storm_Wall_Base.mat"
    - "M_Storm_Wall_Edge.mat"

  particles:
    - "M_Storm_Particle_Cloud.mat"
    - "M_Storm_Particle_Debris.mat"
    - "M_Storm_Particle_Lightning.mat"

  effects:
    - "M_Storm_GhostShip.mat"
    - "M_Storm_Ground.mat"
```

---

## 9. Audio Integration

### 9.1 Storm Audio Zones

```yaml
audio_zones:
  safe_zone:
    ambient: "Normal game ambient"
    storm_distant: "storm_rumble_distant"
    volume: "Based on distance to edge"

  near_edge:
    range: "0-20m from edge"
    ambient: "storm_rumble_near"
    mix: "Blend with safe zone"

  inside_storm:
    ambient: "storm_inside_loop"
    volume: "Full"
    effects:
      - "wind_howl"
      - "ghostly_whispers"
      - "thunder_random"
```

### 9.2 Audio Events

```yaml
audio_events:
  warning:
    trigger: "10s before shrink"
    sound: "storm_warning_horn"

  shrink_start:
    trigger: "Zone begins shrinking"
    sound: "storm_surge"

  enter_storm:
    trigger: "Player crosses into storm"
    sound: "storm_enter"
    transition: "Crossfade 0.5s"

  exit_storm:
    trigger: "Player returns to safe zone"
    sound: "storm_exit"
    transition: "Crossfade 0.5s"

  damage_tick:
    trigger: "Storm damage applied"
    sound: "storm_damage"

  lightning:
    trigger: "Lightning VFX plays"
    sound: "thunder_crack"
    delay: "0.1-0.3s (realistic)"
```

---

## 10. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| Storm wall clearly visible | See from anywhere on map | Distance test |
| Inside storm visually distinct | Reduced visibility, effects | Visual inspection |
| Storm wall performance | No FPS drops | Profiler test |
| Warning VFX before shrink | 10s warning visuals | Timing test |
| Storm closing smooth | No choppy movement | Animation review |
| Safe zone edge clearly visible | Glowing boundary | Visual test |

---

## 11. Implementation Checklist

```yaml
checklist:
  storm_wall:
    - [ ] Base mesh and shader
    - [ ] Cloud scrolling texture
    - [ ] Edge glow effect
    - [ ] Lightning system
    - [ ] Ghostly ships

  inside_storm:
    - [ ] Fog/visibility reduction
    - [ ] Post-process effects
    - [ ] Ambient particles
    - [ ] Damage feedback

  zone_movement:
    - [ ] Smooth shrink animation
    - [ ] Warning effects
    - [ ] Phase-specific intensity

  minimap:
    - [ ] Zone circle display
    - [ ] Storm area fill
    - [ ] Next zone preview
    - [ ] Timer display

  audio:
    - [ ] Ambient zones
    - [ ] Event triggers
    - [ ] Distance-based mixing

  performance:
    - [ ] LOD system working
    - [ ] Quality settings
    - [ ] Mobile FPS targets met
```

---

## 12. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-01 | Initial specification |

---

*This document defines the complete storm/zone VFX system for Plunderstorm Mobile, ensuring clear gameplay communication while maintaining the supernatural pirate atmosphere.*
