# Ability VFX Set Specification

## Document Information
- **Task ID:** ART-013
- **Priority:** P0
- **Complexity:** L (Large)
- **Dependencies:** GAME-008 (Core Ability Implementation), ART-002 (Technical Art Specifications)
- **Last Updated:** 2025-12-01

---

## 1. Overview

### 1.1 Purpose
This specification defines the visual effects (VFX) for all player abilities in Plunderstorm Mobile. Each ability requires distinct, readable VFX that communicate gameplay information while maintaining mobile performance targets.

### 1.2 Scope

```yaml
vfx_scope:
  mvp_abilities: 10
  launch_abilities: 20+

  vfx_components_per_ability:
    - Casting indicator (if channeled)
    - Projectile (if applicable)
    - Impact effect
    - Status effect (if applicable)
    - Upgrade variants (Rank 1-3)
```

### 1.3 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Readability | Instantly recognize ability type at mobile screen size | Critical |
| Performance | Maintain 60 FPS with multiple abilities active | Critical |
| Feedback | Clear hit/miss communication | Critical |
| Differentiation | Each ability visually distinct | High |
| Scalability | Upgrade ranks have enhanced visuals | High |
| Theme | Pirate fantasy aesthetic throughout | High |

---

## 2. VFX Design Principles

### 2.1 Visual Language

```yaml
ability_color_coding:
  offensive_abilities:
    primary_hue: "Warm (red, orange, yellow)"
    indicator: "Red border on UI"
    examples:
      fire: "#FF4500"
      lightning: "#FFD700"
      poison: "#32CD32"
      physical: "#CD853F"

  utility_abilities:
    primary_hue: "Cool (blue, cyan, green)"
    indicator: "Blue border on UI"
    examples:
      mobility: "#00CED1"
      healing: "#7CFC00"
      defensive: "#4169E1"
      crowd_control: "#9370DB"

  rarity_enhancement:
    rank_1:
      particle_density: "Base"
      glow_intensity: 1.0
      trail_length: "Short"

    rank_2:
      particle_density: "+50%"
      glow_intensity: 1.5
      trail_length: "Medium"
      color_shift: "Slightly more saturated"

    rank_3:
      particle_density: "+100%"
      glow_intensity: 2.0
      trail_length: "Long"
      color_shift: "Gold/white highlights added"
      special_effect: "Unique per ability"
```

### 2.2 Readability Guidelines

```yaml
readability_rules:
  minimum_visibility:
    projectile_size: "0.3m minimum diameter"
    impact_radius_visual: "Match actual hitbox"
    ground_indicator: "Clear edge definition"

  contrast_requirements:
    against_environment: "High saturation core, dark outline"
    against_other_vfx: "Distinct silhouette shape"

  motion_clarity:
    projectile_trail: "Always present"
    impact_persistence: "0.3-0.5 seconds"
    telegraph_warning: "0.5+ seconds for delayed abilities"

  mobile_specific:
    particle_size: "Larger than PC equivalent"
    glow_bloom: "Moderate (not overwhelming)"
    screen_effects: "Subtle, toggleable"
```

### 2.3 Performance Budget

```yaml
performance_budget:
  per_ability_limits:
    max_particles: 50
    max_draw_calls: 2
    max_texture_samples: 4
    max_lights: 1 (optional)

  scene_limits:
    concurrent_ability_vfx: 20
    total_particles: 500
    total_vfx_draw_calls: 30

  quality_tiers:
    high:
      particle_multiplier: 1.0
      trails_enabled: true
      distortion_enabled: true

    medium:
      particle_multiplier: 0.6
      trails_enabled: true
      distortion_enabled: false

    low:
      particle_multiplier: 0.3
      trails_enabled: false
      distortion_enabled: false
```

---

## 3. Offensive Abilities VFX

### 3.1 Fireball

```yaml
fireball:
  ability_type: "Offensive"
  mechanic: "Ranged skillshot, medium damage, small AoE"
  theme: "Classic fire magic with pirate flair"

  casting:
    duration: 0.2s
    effects:
      hand_glow:
        color: "#FF4500"
        intensity: 2.0
        duration: 0.2s
      charge_particles:
        type: "Ember gather"
        count: 15
        movement: "Spiral toward hand"

  projectile:
    shape: "Sphere with trailing flames"
    size:
      rank_1: 0.4m
      rank_2: 0.5m
      rank_3: 0.6m
    speed: 25m/s

    core:
      color_inner: "#FFFF00"
      color_outer: "#FF4500"
      glow_radius: 1.5m

    trail:
      type: "Fire ribbon"
      length:
        rank_1: 2m
        rank_2: 3m
        rank_3: 4m
      particles: "Embers falling off"
      particle_count: 20

    distortion:
      enabled: true
      strength: 0.1
      radius: 0.5m

  impact:
    type: "Explosion"
    radius:
      visual: 3m
      actual_hitbox: 2.5m

    effects:
      explosion_sphere:
        duration: 0.3s
        color: "#FF6600"
        expansion_speed: "Fast burst"

      ground_scorch:
        decal: true
        duration: 5s
        radius: 2m
        fade_time: 1s

      sparks:
        count: 30
        spread: "Radial outward"
        lifetime: 0.5s

      smoke:
        count: 5
        color: "#333333"
        rise_speed: 2m/s
        lifetime: 1.5s

    screen_shake:
      intensity: 0.1
      duration: 0.15s

  rank_3_bonus:
    effect: "Fire trail persists on ground"
    trail_duration: 2s
    trail_damage: "Visual only (damage in gameplay)"
    trail_color: "#FF4500"

  audio_sync:
    cast: "fireball_cast"
    travel: "fireball_loop"
    impact: "fireball_explosion"

  technical:
    particle_systems: 3
    materials: 2
    draw_calls: 2
    estimated_particles: 45
```

### 3.2 Cutlass Slash

```yaml
cutlass_slash:
  ability_type: "Offensive"
  mechanic: "Melee arc, high damage, short range"
  theme: "Swift pirate sword strike"

  casting:
    duration: 0.15s
    effects:
      blade_gleam:
        type: "Edge highlight"
        color: "#FFFFFF"
        sweep: "Along blade edge"

  slash_arc:
    shape: "Crescent sweep"
    dimensions:
      arc_angle: 90°
      range: 3m
      height: 2m

    visual:
      slash_trail:
        type: "Ribbon trail"
        color_core: "#FFFFFF"
        color_edge: "#87CEEB"
        duration: 0.2s
        opacity_fade: "Quick"

      speed_lines:
        count: 5
        spread: "Along arc"
        color: "#FFFFFF"
        opacity: 0.6

    rank_variations:
      rank_1:
        trail_width: 0.3m
        color_intensity: 1.0

      rank_2:
        trail_width: 0.4m
        color_intensity: 1.3
        secondary_trail: true

      rank_3:
        trail_width: 0.5m
        color_intensity: 1.6
        afterimage: true
        afterimage_count: 2

  hit_effect:
    type: "Spark burst"

    effects:
      sparks:
        count: 15
        color: "#FFFF00"
        spread: "Directional (slash direction)"
        lifetime: 0.3s

      blood_substitute:
        type: "Golden coins burst"
        count: 5
        physics: true
        lifetime: 1s

      hit_flash:
        color: "#FFFFFF"
        duration: 0.05s
        intensity: 2.0

  rank_3_bonus:
    effect: "Electric arc on hit"
    arc_color: "#00FFFF"
    arc_duration: 0.3s
    chain_visual: false  # Visual only, no chain damage

  audio_sync:
    swing: "sword_swing_*"
    hit_enemy: "sword_hit_flesh"
    hit_object: "sword_hit_metal"
    miss: "sword_whoosh"

  technical:
    particle_systems: 2
    materials: 2
    draw_calls: 1
    estimated_particles: 25
```

### 3.3 Cannonball

```yaml
cannonball:
  ability_type: "Offensive"
  mechanic: "Long range, high damage, slow projectile"
  theme: "Ship cannon shot"

  casting:
    duration: 0.4s
    effects:
      cannon_summon:
        type: "Magical cannon appears"
        animation: "Phase in from smoke"
        duration: 0.3s

      fuse_light:
        color: "#FF6600"
        sparks: true
        duration: 0.2s

  projectile:
    shape: "Iron cannonball"
    size:
      rank_1: 0.3m
      rank_2: 0.35m
      rank_3: 0.4m
    speed: 18m/s
    arc: "Slight arc trajectory"

    visual:
      ball_material: "Dark iron, slight glow"

      smoke_trail:
        type: "Thick smoke ribbon"
        color: "#444444"
        length: 5m
        dissipation: "Gradual"

      spin:
        enabled: true
        speed: "Fast rotation"

    rank_variations:
      rank_2:
        trail_fire: true
        fire_color: "#FF4500"

      rank_3:
        chain_shot_visual: true
        secondary_ball: "Smaller, connected by chain"

  impact:
    type: "Large explosion"
    radius:
      visual: 5m
      actual_hitbox: 4m

    effects:
      explosion:
        type: "Dirt/debris explosion"
        color: "#8B4513"
        secondary_color: "#FF6600"
        duration: 0.5s

      shockwave:
        type: "Ground ring"
        color: "#FFFFFF"
        opacity: 0.5
        expansion: 8m
        duration: 0.3s

      debris:
        count: 20
        types: ["Wood chunks", "Dirt clods", "Sparks"]
        physics: true
        lifetime: 1.5s

      crater_decal:
        enabled: true
        radius: 2m
        duration: 10s
        fade: 2s

    screen_shake:
      intensity: 0.2
      duration: 0.25s

  audio_sync:
    cast: "cannon_fire"
    travel: "cannonball_whistle"
    impact: "cannon_explosion"

  technical:
    particle_systems: 4
    materials: 3
    draw_calls: 2
    estimated_particles: 50
```

### 3.4 Lightning Strike

```yaml
lightning_strike:
  ability_type: "Offensive"
  mechanic: "Target area, delayed AoE damage"
  theme: "Storm magic, wrath of the sea"

  targeting:
    type: "Ground target circle"
    range: 20m

    indicator:
      shape: "Circle with rune pattern"
      radius: 3m
      color: "#FFD700"
      opacity: 0.7
      animation: "Runes rotate, intensity pulses"

    warning:
      duration: 0.8s
      cloud_gather:
        enabled: true
        position: "Above target"
        color: "#333366"
        size_growth: "Expanding"

  strike:
    delay: 0.8s

    lightning_bolt:
      type: "Branching lightning"
      color_core: "#FFFFFF"
      color_glow: "#FFD700"
      branches: 3-5
      width: 0.3m
      duration: 0.15s

    ground_impact:
      type: "Electric explosion"
      radius: 3m

      effects:
        flash:
          color: "#FFFFFF"
          intensity: 3.0
          duration: 0.1s

        electric_arcs:
          count: 8
          color: "#00FFFF"
          duration: 0.4s
          crawl_distance: 2m

        scorched_ground:
          decal: true
          pattern: "Lightning scar"
          duration: 8s

        sparks:
          count: 25
          color: "#FFD700"
          lifetime: 0.5s

    rank_variations:
      rank_2:
        secondary_bolts: 2
        secondary_delay: 0.1s

      rank_3:
        chain_lightning_visual: true
        chain_count: 3
        chain_range: 5m

    screen_flash:
      enabled: true
      color: "#FFFFFF"
      intensity: 0.5
      duration: 0.1s

  audio_sync:
    targeting: "electric_charge_loop"
    strike: "lightning_strike"
    aftermath: "electric_crackle"

  technical:
    particle_systems: 4
    materials: 3
    draw_calls: 2
    estimated_particles: 40
```

### 3.5 Poison Mackerel

```yaml
poison_mackerel:
  ability_type: "Offensive"
  mechanic: "Thrown, DoT effect on hit"
  theme: "Cursed fish, comedic but deadly"

  casting:
    duration: 0.2s
    effects:
      fish_summon:
        type: "Fish appears in hand"
        color: "#32CD32"
        glow: "Sickly green"
        drip_particles: true

  projectile:
    shape: "Fish (mackerel model)"
    size: 0.4m length
    speed: 22m/s
    rotation: "Tumbling spin"

    visual:
      fish_material:
        base_color: "#228B22"
        emission: "#32CD32"
        emission_intensity: 1.5

      poison_drip:
        type: "Dripping particles"
        color: "#32CD32"
        count: 10
        trail: true

      stink_lines:
        enabled: true
        color: "#9ACD32"
        opacity: 0.4

  impact:
    type: "Poison splash"

    effects:
      splat:
        type: "Liquid splash"
        color: "#32CD32"
        radius: 1.5m
        duration: 0.3s

      poison_cloud:
        type: "Lingering mist"
        color: "#228B22"
        opacity: 0.5
        radius: 2m
        duration: 3s
        rise_speed: 0.5m/s

      bubbles:
        count: 15
        color: "#32CD32"
        pop_effect: true

      fish_debris:
        enabled: true
        pieces: 3
        lifetime: 2s

  status_effect:
    name: "Poisoned"
    duration: 4s

    victim_vfx:
      skin_tint:
        color: "#32CD32"
        intensity: 0.3

      poison_particles:
        type: "Rising bubbles from body"
        count: 5
        interval: 0.5s

      damage_tick_flash:
        color: "#32CD32"
        duration: 0.1s
        interval: 1s

    rank_variations:
      rank_2:
        cloud_duration: 4s

      rank_3:
        fish_explodes: true
        explosion_radius: 3m
        mini_fish_count: 3

  audio_sync:
    cast: "fish_throw"
    travel: "fish_spin_whoosh"
    impact: "splat_wet"
    poison_tick: "poison_bubble"

  technical:
    particle_systems: 4
    materials: 2
    draw_calls: 2
    estimated_particles: 35
```

---

## 4. Utility Abilities VFX

### 4.1 Grappling Hook

```yaml
grappling_hook:
  ability_type: "Utility"
  mechanic: "Dash to target location"
  theme: "Pirate boarding rope"

  casting:
    duration: 0.1s
    effects:
      arm_wind:
        type: "Anticipation pose"
        duration: 0.1s

  hook_flight:
    shape: "Metal hook with rope trail"
    size: 0.2m
    speed: 40m/s
    max_range: 15m

    visual:
      hook:
        material: "Rusted iron"
        spin: "Slight tumble"

      rope:
        type: "Dynamic rope trail"
        color: "#D2B48C"
        thickness: 0.05m
        physics: "Slight sway"
        attach_point: "Player hand"

      motion_blur:
        enabled: true
        intensity: 0.3

  hook_attach:
    effects:
      impact_sparks:
        count: 10
        color: "#FFD700"
        spread: "Radial"

      dust_puff:
        enabled: true
        color: "#A0A0A0"
        size: 0.5m

      rope_taut:
        animation: "Snap tight"
        duration: 0.05s

  player_pull:
    effects:
      speed_lines:
        count: 8
        color: "#FFFFFF"
        opacity: 0.4

      wind_particles:
        enabled: true
        direction: "Against movement"
        count: 15

      motion_trail:
        enabled: true
        color: "Player silhouette"
        opacity: 0.3
        count: 3

  landing:
    effects:
      dust_cloud:
        size: 1m
        color: "#A0A0A0"

      ground_impact:
        decal: false
        particles: 10

    rank_variations:
      rank_2:
        rope_glow: true
        glow_color: "#00CED1"

      rank_3:
        arrival_shockwave: true
        shockwave_radius: 2m
        shockwave_damage_visual: true

  audio_sync:
    throw: "rope_throw"
    attach: "hook_clang"
    pull: "rope_zip"
    land: "boots_land"

  technical:
    particle_systems: 3
    materials: 2
    draw_calls: 2
    estimated_particles: 30
```

### 4.2 Barrel Roll

```yaml
barrel_roll:
  ability_type: "Utility"
  mechanic: "Brief invulnerability + movement"
  theme: "Hide in a barrel, roll to safety"

  activation:
    duration: 0.1s
    effects:
      barrel_spawn:
        type: "Barrel appears around player"
        animation: "Pop in with dust"

      player_hide:
        type: "Player model hidden"

  rolling:
    duration: 0.6s
    distance: 6m

    barrel_visual:
      model: "Wooden barrel"
      size: 1.2m height

      roll_animation:
        type: "Physical roll"
        speed: "Matches movement"
        wobble: "Slight"

      dust_trail:
        type: "Ground dust"
        color: "#A0A0A0"
        emission_rate: 20

      wood_particles:
        enabled: true
        count: 5
        interval: 0.2s
        type: "Small splinters"

    invulnerability_indicator:
      barrel_glow:
        color: "#4169E1"
        intensity: 0.5
        pulse: true

      shield_shimmer:
        enabled: true
        color: "#87CEEB"
        opacity: 0.3

  exit:
    effects:
      barrel_break:
        type: "Barrel shatters"
        pieces: 8
        spread: "Radial"
        physics: true
        lifetime: 2s

      dust_burst:
        color: "#A0A0A0"
        radius: 1.5m

      player_reveal:
        animation: "Pop out pose"

    rank_variations:
      rank_2:
        barrel_reinforced: true
        metal_bands_glow: "#FFD700"

      rank_3:
        explosive_exit: true
        explosion_radius: 2m
        explosion_color: "#FF6600"
        knockback_visual: true

  audio_sync:
    enter: "barrel_hide"
    roll: "barrel_roll_loop"
    exit: "barrel_break"

  technical:
    particle_systems: 3
    materials: 2
    draw_calls: 1
    estimated_particles: 25
```

### 4.3 Healing Grog

```yaml
healing_grog:
  ability_type: "Utility"
  mechanic: "Self heal over time"
  theme: "Magical pirate drink"

  casting:
    duration: 0.8s
    interruptible: true

    effects:
      bottle_summon:
        type: "Bottle appears in hand"
        model: "Rum bottle with glow"
        color: "#7CFC00"

      drink_animation:
        type: "Drinking pose"
        duration: 0.6s

      gulp_particles:
        type: "Magical droplets rising"
        color: "#7CFC00"
        count: 10

  healing_effect:
    duration: 4s

    visual:
      body_glow:
        color: "#7CFC00"
        intensity: 0.4
        pulse_speed: 1s

      heal_particles:
        type: "Rising sparkles"
        color_primary: "#7CFC00"
        color_secondary: "#FFFFFF"
        count: 15
        emit_from: "Player body"
        rise_speed: 1m/s

      health_restore_flash:
        color: "#7CFC00"
        intensity: 0.2
        interval: 1s

      bottle_trail:
        enabled: true
        ghost_bottles: 3
        opacity: 0.3
        orbit: true

    status_icon:
      position: "Above player"
      icon: "Bottle"
      color: "#7CFC00"

    rank_variations:
      rank_2:
        particle_intensity: 1.5
        golden_sparkles: true

      rank_3:
        aura_ring: true
        aura_color: "#7CFC00"
        aura_radius: 1.5m
        heal_numbers_golden: true

  audio_sync:
    drink: "gulp_magical"
    healing: "heal_shimmer_loop"
    complete: "heal_complete"

  technical:
    particle_systems: 2
    materials: 2
    draw_calls: 1
    estimated_particles: 20
```

### 4.4 Smoke Bomb

```yaml
smoke_bomb:
  ability_type: "Utility"
  mechanic: "AoE vision block + slow enemies"
  theme: "Ninja-pirate smoke grenade"

  casting:
    duration: 0.2s
    effects:
      throw_prep:
        type: "Hand raise"
        bomb_visible: true
        bomb_color: "#4A4A4A"
        fuse_spark: true

  projectile:
    shape: "Small round bomb"
    size: 0.15m
    speed: 20m/s
    arc: "Lobbed trajectory"

    visual:
      bomb_model: "Metal sphere with fuse"
      fuse_spark:
        color: "#FF6600"
        particles: 5
        trail: true

  detonation:
    delay: 0.3s (after landing)

    effects:
      initial_burst:
        type: "Smoke explosion"
        color: "#333333"
        radius: 1m
        duration: 0.2s

      smoke_cloud:
        type: "Expanding smoke"
        color: "#4A4A4A"
        opacity: 0.8
        radius:
          rank_1: 4m
          rank_2: 5m
          rank_3: 6m
        height: 3m
        duration:
          rank_1: 4s
          rank_2: 5s
          rank_3: 6s
        behavior: "Billowing, slight movement"

      edge_wisps:
        enabled: true
        color: "#666666"
        opacity: 0.5

      ground_fog:
        enabled: true
        height: 0.5m
        color: "#333333"

  inside_cloud:
    visibility: "5m (reduced from normal)"

    player_effects:
      outline_hidden: true
      silhouette_only: true

    enemy_slow:
      visual:
        tint: "#666666"
        particles: "Smoke wisps on body"

    rank_variations:
      rank_2:
        purple_tint: true
        smoke_color: "#4A3366"

      rank_3:
        choking_particles: true
        particle_color: "#32CD32"
        damage_tick_visual: true

  audio_sync:
    throw: "bomb_throw"
    detonate: "smoke_poof"
    ambient: "smoke_hiss_loop"
    dissipate: "smoke_fade"

  technical:
    particle_systems: 3
    materials: 2
    draw_calls: 2
    estimated_particles: 40
```

### 4.5 Wind Burst

```yaml
wind_burst:
  ability_type: "Utility"
  mechanic: "Knockback enemies in radius"
  theme: "Storm magic, gust of wind"

  casting:
    duration: 0.3s

    effects:
      wind_gather:
        type: "Spiral wind inward"
        color: "#87CEEB"
        opacity: 0.4
        radius: 3m (contracting)
        duration: 0.3s

      player_glow:
        color: "#00CED1"
        intensity: 1.5

      hair_cloth_blow:
        direction: "Inward then outward"

  burst:
    radius:
      rank_1: 5m
      rank_2: 6m
      rank_3: 7m

    effects:
      shockwave_ring:
        type: "Expanding ring"
        color: "#87CEEB"
        opacity: 0.6
        thickness: 0.5m
        expansion_speed: "Fast"
        duration: 0.3s

      wind_particles:
        type: "Radial wind lines"
        count: 30
        color: "#FFFFFF"
        opacity: 0.5
        length: 2m

      dust_ring:
        enabled: true
        color: "#A0A0A0"
        height: 0.3m

      debris:
        enabled: true
        types: ["Leaves", "Papers", "Small objects"]
        count: 15
        physics: true

  knockback_visual:
    on_enemy:
      push_direction: "Away from caster"

      effects:
        wind_trail:
          color: "#87CEEB"
          opacity: 0.4
          attached_to: "Enemy"
          duration: 0.5s

        tumble_particles:
          enabled: true
          count: 5

    rank_variations:
      rank_2:
        double_ring: true
        second_ring_delay: 0.1s

      rank_3:
        tornado_center: true
        tornado_height: 3m
        tornado_duration: 0.5s
        lift_enemies: true

  audio_sync:
    charge: "wind_gather"
    burst: "wind_burst"
    knockback: "whoosh_impact"

  technical:
    particle_systems: 3
    materials: 2
    draw_calls: 2
    estimated_particles: 45
```

---

## 5. Technical Specifications

### 5.1 Particle System Standards

```yaml
particle_standards:
  texture_format:
    type: "ASTC 4x4 or 6x6"
    atlas_size: "512x512"

  shader_requirements:
    base_shader: "Plunderstorm/Particles/Additive"
    variants:
      - "Plunderstorm/Particles/SoftAdditive"
      - "Plunderstorm/Particles/Multiply"
      - "Plunderstorm/Particles/Distortion"

  sorting:
    mode: "By distance"
    render_queue: "Transparent (3000+)"

  optimization:
    gpu_instancing: true
    mesh_particles: "For complex shapes only"
    billboard_mode: "View aligned"

  pooling:
    pre_warm_count: 5 per ability
    max_pool_size: 20 per ability
```

### 5.2 Material Specifications

```yaml
material_specs:
  projectile_materials:
    shader: "Plunderstorm/VFX/Projectile"
    properties:
      _MainTex: "Base texture"
      _EmissionColor: "Glow color"
      _EmissionIntensity: "Float 0-5"
      _DistortionStrength: "Float 0-1"
      _FresnelPower: "Edge glow control"

  trail_materials:
    shader: "Plunderstorm/VFX/Trail"
    properties:
      _MainTex: "Gradient texture"
      _Color: "Tint color"
      _Width: "Trail width"
      _Length: "Fade distance"

  impact_materials:
    shader: "Plunderstorm/VFX/Impact"
    properties:
      _MainTex: "Sprite sheet"
      _Columns: "Sheet columns"
      _Rows: "Sheet rows"
      _FPS: "Animation speed"
```

### 5.3 Performance Profiles

```yaml
performance_profiles:
  single_ability:
    target_ms: "<0.5ms GPU"
    max_particles: 50
    max_overdraw: 4x

  combat_scenario:
    description: "6 players fighting, multiple abilities"
    target_ms: "<3ms total VFX"
    max_concurrent: 20 abilities

  stress_test:
    description: "Maximum expected load"
    abilities_active: 30
    target_fps: "Maintain 30 FPS minimum"
```

### 5.4 LOD System

```yaml
vfx_lod:
  distance_based:
    LOD0:
      distance: "0-20m"
      particle_rate: 100%
      all_features: true

    LOD1:
      distance: "20-50m"
      particle_rate: 50%
      disable: ["Secondary particles", "Distortion"]

    LOD2:
      distance: "50-100m"
      particle_rate: 25%
      disable: ["All but core effect"]

    culled:
      distance: ">100m"
      render: false
      audio: "Reduced"

  importance_based:
    player_own_abilities:
      lod_bias: -1 (higher quality)

    enemy_abilities:
      lod_bias: 0 (standard)

    distant_abilities:
      lod_bias: +1 (lower quality)
```

---

## 6. VFX Asset List

### 6.1 Texture Atlas Contents

```yaml
vfx_texture_atlas:
  atlas_1_particles:
    resolution: "512x512"
    contents:
      - "T_Spark_01-04"
      - "T_Smoke_01-04"
      - "T_Fire_01-04"
      - "T_Glow_Soft"
      - "T_Glow_Hard"
      - "T_Ring_01-02"
      - "T_Trail_Gradient"
      - "T_Dust_01-02"

  atlas_2_effects:
    resolution: "512x512"
    contents:
      - "T_Lightning_Branch"
      - "T_Shockwave_Ring"
      - "T_Slash_Arc"
      - "T_Explosion_Sheet"
      - "T_Bubble"
      - "T_Leaf"
      - "T_Debris_01-04"

  atlas_3_special:
    resolution: "256x256"
    contents:
      - "T_Rune_Circle"
      - "T_Target_Indicator"
      - "T_Rope_Segment"
      - "T_Noise_Distort"
```

### 6.2 Prefab Deliverables

```yaml
vfx_prefabs:
  offensive_abilities:
    - "VFX_Fireball_Cast.prefab"
    - "VFX_Fireball_Projectile.prefab"
    - "VFX_Fireball_Impact.prefab"
    - "VFX_CutlassSlash_Swing.prefab"
    - "VFX_CutlassSlash_Hit.prefab"
    - "VFX_Cannonball_Cast.prefab"
    - "VFX_Cannonball_Projectile.prefab"
    - "VFX_Cannonball_Impact.prefab"
    - "VFX_LightningStrike_Target.prefab"
    - "VFX_LightningStrike_Bolt.prefab"
    - "VFX_LightningStrike_Impact.prefab"
    - "VFX_PoisonMackerel_Projectile.prefab"
    - "VFX_PoisonMackerel_Impact.prefab"
    - "VFX_PoisonMackerel_Cloud.prefab"
    - "VFX_PoisonMackerel_Status.prefab"

  utility_abilities:
    - "VFX_GrapplingHook_Hook.prefab"
    - "VFX_GrapplingHook_Rope.prefab"
    - "VFX_GrapplingHook_Pull.prefab"
    - "VFX_BarrelRoll_Barrel.prefab"
    - "VFX_BarrelRoll_Exit.prefab"
    - "VFX_HealingGrog_Drink.prefab"
    - "VFX_HealingGrog_Heal.prefab"
    - "VFX_SmokeBomb_Throw.prefab"
    - "VFX_SmokeBomb_Cloud.prefab"
    - "VFX_WindBurst_Charge.prefab"
    - "VFX_WindBurst_Burst.prefab"

  shared:
    - "VFX_Hit_Marker.prefab"
    - "VFX_Miss_Indicator.prefab"
    - "VFX_Level_Up_Ability.prefab"
```

---

## 7. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| Each ability has distinct VFX | 10 unique ability sets | Visual comparison |
| VFX readable on mobile screens | Minimum sizes met | Device testing |
| Performance optimized | <50 particles per ability | Profiler verification |
| Rarity upgrades have enhanced VFX | Rank 1-3 variations | Visual inspection |
| No VFX obscures gameplay | Clear sightlines maintained | Playtest feedback |
| Color coding helps identify abilities | Warm/cool distinction | User testing |

---

## 8. Implementation Checklist

```yaml
implementation_checklist:
  per_ability:
    - [ ] Cast VFX complete
    - [ ] Projectile VFX complete (if applicable)
    - [ ] Impact VFX complete
    - [ ] Status VFX complete (if applicable)
    - [ ] Rank 1 baseline working
    - [ ] Rank 2 enhancement working
    - [ ] Rank 3 special effect working
    - [ ] Audio sync points set
    - [ ] Performance within budget
    - [ ] LOD system configured
    - [ ] Mobile tested

  overall:
    - [ ] Texture atlases optimized
    - [ ] Material instances created
    - [ ] Particle pooling implemented
    - [ ] Quality settings per device tier
    - [ ] All abilities tested simultaneously
```

---

## 9. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-01 | Initial specification |

---

*This document defines the complete VFX requirements for all MVP abilities. Additional abilities will follow the same specification format.*
