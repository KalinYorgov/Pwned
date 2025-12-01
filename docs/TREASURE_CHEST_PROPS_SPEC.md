# Treasure Chest Props Specification

## Document Information
- **Task ID:** ART-010
- **Priority:** P0
- **Complexity:** S (Small)
- **Dependencies:** ART-001 (Visual Style Guide), GAME-004 (Loot Spawn System)
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the treasure chest props for Plunderstorm Mobile's loot system. Chests serve as the primary visual indicator for loot spawns, with distinct tiers that communicate rarity and reward value to players at a glance.

### 1.2 Scope
- Four chest rarity tiers (Common, Uncommon, Rare, Epic)
- Open and closed states for each tier
- Opening animation/VFX
- Visual effects for rarity indication
- LOD system for performance
- Audio cues

### 1.3 Goals
- Instant rarity recognition at distance
- Satisfying loot discovery experience
- Mobile-optimized performance
- Cohesive pirate fantasy aesthetic

---

## 2. Design Direction

### 2.1 Visual Style

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Treasure Chest Design Pillars                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │   AUTHENTIC     │  │   READABLE      │  │   REWARDING     │          │
│  │   PIRATE LOOT   │  │   AT DISTANCE   │  │   DISCOVERY     │          │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤          │
│  │ • Classic chest │  │ • Clear shape   │  │ • Satisfying    │          │
│  │   silhouette    │  │ • Color coding  │  │   to find       │          │
│  │ • Wood + metal  │  │ • Glow effects  │  │ • Exciting to   │          │
│  │ • Treasure feel │  │ • Size scaling  │  │   open          │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
│                                                                          │
│  Reference: Classic pirate treasure chests with fantasy embellishment   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Rarity Color System

```yaml
rarity_colors:
  common:
    primary: "#808080"      # Gray
    secondary: "#A0A0A0"    # Light gray
    glow: none
    particle: none

  uncommon:
    primary: "#2ECC71"      # Green
    secondary: "#27AE60"    # Dark green
    glow: "Subtle green"
    particle: "Faint sparkle"

  rare:
    primary: "#3498DB"      # Blue
    secondary: "#2980B9"    # Dark blue
    glow: "Medium blue"
    particle: "Blue sparkles"

  epic:
    primary: "#9B59B6"      # Purple
    secondary: "#8E44AD"    # Dark purple
    glow: "Strong purple"
    particle: "Purple + gold sparkles"
```

---

## 3. Chest Variants

### 3.1 Common Chest

```yaml
common_chest:
  name: "Wooden Chest"
  rarity: "Common"
  drop_rate: "60%"

  design:
    description: |
      A simple wooden chest, weathered by salt and sea.
      Basic construction, no embellishments.

    materials:
      body: "Weathered oak wood"
      bands: "Rusted iron"
      lock: "Simple iron padlock"
      hinges: "Basic iron"

    details:
      - "Visible wood grain"
      - "Some planks slightly warped"
      - "Rust stains from metal"
      - "No decorations"

  visual_indicators:
    silhouette: "Standard chest shape"
    color_accent: "Gray iron bands"
    glow: "None"
    particles: "None"
    size_scale: 1.0

  technical:
    triangles_closed: 200
    triangles_open: 250
    texture_size: "256x256 (shared atlas)"
    material_count: 1

  states:
    closed:
      - Lid shut
      - Lock visible

    open:
      - Lid at 110° angle
      - Interior visible
      - Lock broken/removed

  audio:
    discovery: "wood_creak_subtle"
    open: "chest_open_basic"
```

### 3.2 Uncommon Chest

```yaml
uncommon_chest:
  name: "Reinforced Chest"
  rarity: "Uncommon"
  drop_rate: "25%"

  design:
    description: |
      A sturdier chest with brass reinforcements.
      Better crafted, shows some care.

    materials:
      body: "Quality oak wood"
      bands: "Polished brass"
      lock: "Brass lock mechanism"
      hinges: "Brass hinges"
      corners: "Brass corner guards"

    details:
      - "Cleaner wood finish"
      - "Decorative brass studs"
      - "Simple carved border"
      - "Green cloth lining visible when open"

  visual_indicators:
    silhouette: "Standard with corner accents"
    color_accent: "Green glow, brass shine"
    glow: "Subtle green aura (2m radius)"
    particles: "Faint green sparkles (5/sec)"
    size_scale: 1.05

  technical:
    triangles_closed: 280
    triangles_open: 350
    texture_size: "256x256 (shared atlas)"
    material_count: 1

  states:
    closed:
      - Lid shut
      - Brass fittings gleam
      - Subtle glow effect

    open:
      - Lid at 110° angle
      - Green cloth interior
      - Brass catches glint

  audio:
    discovery: "brass_shimmer"
    open: "chest_open_quality"
```

### 3.3 Rare Chest

```yaml
rare_chest:
  name: "Ornate Chest"
  rarity: "Rare"
  drop_rate: "12%"

  design:
    description: |
      A beautifully crafted chest with silver filigree.
      Clearly valuable, draws the eye.

    materials:
      body: "Rich mahogany wood"
      bands: "Silver-plated metal"
      lock: "Ornate silver lock"
      hinges: "Decorative silver"
      accents: "Silver filigree patterns"
      gems: "Small blue gems (4)"

    details:
      - "Dark polished wood"
      - "Intricate silver scrollwork"
      - "Inlaid wave patterns"
      - "Blue velvet interior"
      - "Embedded gems on corners"

  visual_indicators:
    silhouette: "Ornate with decorative top"
    color_accent: "Blue glow, silver shine"
    glow: "Medium blue aura (4m radius)"
    particles: "Blue sparkles (15/sec)"
    size_scale: 1.1

  technical:
    triangles_closed: 380
    triangles_open: 450
    texture_size: "512x512"
    material_count: 2  # Wood + metal/emissive

  states:
    closed:
      - Lid shut
      - Silver filigree glows
      - Blue gems pulse softly
      - Particle effect active

    open:
      - Lid at 120° angle
      - Blue velvet interior shimmers
      - Light rays from inside

  audio:
    discovery: "magical_chime_soft"
    open: "chest_open_ornate"
```

### 3.4 Epic Chest

```yaml
epic_chest:
  name: "Legendary Chest"
  rarity: "Epic"
  drop_rate: "3%"

  design:
    description: |
      A magnificent chest radiating magical energy.
      Unmistakably legendary, beacon of treasure.

    materials:
      body: "Ancient enchanted wood"
      bands: "Gold-plated metal"
      lock: "Golden skull lock"
      hinges: "Gold dragon hinges"
      accents: "Golden chains and runes"
      gems: "Large purple gems (6)"
      magic: "Floating rune particles"

    details:
      - "Glowing golden runes carved in wood"
      - "Animated golden chains"
      - "Purple gems pulse with light"
      - "Magical mist seeps from seams"
      - "Royal purple velvet interior"
      - "Golden coin pile visible inside"

  visual_indicators:
    silhouette: "Large ornate with crown-like top"
    color_accent: "Purple glow, gold shine"
    glow: "Strong purple aura (6m radius)"
    particles: "Purple + gold sparkles (30/sec)"
    light_beam: "Vertical light pillar"
    size_scale: 1.2

  technical:
    triangles_closed: 500
    triangles_open: 600
    texture_size: "512x512"
    material_count: 3  # Wood, metal, emissive

  states:
    closed:
      - Lid shut with golden seal
      - Runes animate/pulse
      - Purple mist effect
      - Light pillar above
      - Heavy particle effect

    open:
      - Lid at 130° angle
      - Golden light burst on open
      - Treasure visible inside
      - Intensified particles

  audio:
    discovery: "legendary_fanfare"
    open: "chest_open_legendary"
```

---

## 4. Technical Specifications

### 4.1 Mesh Requirements

```yaml
mesh_specifications:
  common:
    closed:
      triangles: 200
      vertices: 120
    open:
      triangles: 250
      vertices: 150
    collision: "Box collider (simplified)"

  uncommon:
    closed:
      triangles: 280
      vertices: 170
    open:
      triangles: 350
      vertices: 210
    collision: "Box collider"

  rare:
    closed:
      triangles: 380
      vertices: 230
    open:
      triangles: 450
      vertices: 280
    collision: "Box collider"

  epic:
    closed:
      triangles: 500
      vertices: 310
    open:
      triangles: 600
      vertices: 380
    collision: "Box collider"

  lod_system:
    LOD0:
      distance: "0-20m"
      quality: "Full detail"

    LOD1:
      distance: "20-50m"
      quality: "50% triangles"
      disable: "Interior detail"

    LOD2:
      distance: "50-100m"
      quality: "25% triangles"
      disable: "All small details"

    culled:
      distance: ">100m"
      render: "Glow only (rare+)"
```

### 4.2 Texture Specifications

```yaml
texture_specifications:
  atlas_layout:
    name: "T_Chests_Atlas"
    resolution: "1024x1024"
    format: "ASTC 6x6"

    allocation:
      common: "25% (256x256 region)"
      uncommon: "25% (256x256 region)"
      rare: "25% (256x256 region)"
      epic: "25% (256x256 region)"

  maps_per_variant:
    albedo:
      description: "Base color with baked AO"
      format: "RGB"

    normal:
      description: "Surface details"
      format: "RG compressed"
      details:
        - "Wood grain"
        - "Metal edges"
        - "Gem facets"
        - "Carved patterns"

    mask:
      description: "Material properties"
      channels:
        R: "Metallic"
        G: "Roughness"
        B: "Emission mask"
        A: "Reserved"

  emission_textures:
    uncommon: "Subtle green glow regions"
    rare: "Blue gem glow + filigree"
    epic: "Full rune patterns + gems"
```

### 4.3 Material Setup

```yaml
material_configuration:
  common_chest:
    shader: "Plunderstorm/Props"
    properties:
      _BaseColor: "Albedo texture"
      _NormalMap: "Normal texture"
      _Roughness: 0.7
      _Metallic: 0.1
      _EmissionColor: "Black"

  uncommon_chest:
    shader: "Plunderstorm/Props_Emissive"
    properties:
      _BaseColor: "Albedo texture"
      _NormalMap: "Normal texture"
      _Roughness: 0.5
      _Metallic: 0.3
      _EmissionColor: "#2ECC71"
      _EmissionIntensity: 0.5

  rare_chest:
    shader: "Plunderstorm/Props_Emissive"
    properties:
      _BaseColor: "Albedo texture"
      _NormalMap: "Normal texture"
      _Roughness: 0.4
      _Metallic: 0.5
      _EmissionColor: "#3498DB"
      _EmissionIntensity: 1.0
      _PulseSpeed: 1.5

  epic_chest:
    shader: "Plunderstorm/Props_Legendary"
    properties:
      _BaseColor: "Albedo texture"
      _NormalMap: "Normal texture"
      _Roughness: 0.3
      _Metallic: 0.7
      _EmissionColor: "#9B59B6"
      _EmissionIntensity: 2.0
      _PulseSpeed: 2.0
      _RuneScrollSpeed: 0.5
```

---

## 5. Animation System

### 5.1 Opening Animation

```yaml
chest_animations:
  common:
    open_duration: 0.5s
    animation_type: "Simple rotation"
    keyframes:
      - time: 0.0
        lid_angle: 0°
      - time: 0.3
        lid_angle: 120°
      - time: 0.5
        lid_angle: 110°  # Slight settle

  uncommon:
    open_duration: 0.6s
    animation_type: "Rotation with bounce"
    keyframes:
      - time: 0.0
        lid_angle: 0°
        glow_intensity: 1.0
      - time: 0.3
        lid_angle: 130°
        glow_intensity: 2.0
      - time: 0.5
        lid_angle: 105°
        glow_intensity: 1.5
      - time: 0.6
        lid_angle: 110°
        glow_intensity: 1.0

  rare:
    open_duration: 0.8s
    animation_type: "Rotation with light burst"
    keyframes:
      - time: 0.0
        lid_angle: 0°
        light_intensity: 0
      - time: 0.2
        lid_angle: 30°
        light_intensity: 0.5
      - time: 0.4
        lid_angle: 140°
        light_intensity: 3.0  # Burst
      - time: 0.6
        lid_angle: 115°
        light_intensity: 1.5
      - time: 0.8
        lid_angle: 120°
        light_intensity: 1.0
    vfx_trigger: "Light rays at 0.3s"

  epic:
    open_duration: 1.2s
    animation_type: "Dramatic reveal"
    keyframes:
      - time: 0.0
        lid_angle: 0°
        scale: 1.0
        light_intensity: 2.0
      - time: 0.3
        lid_angle: 20°
        scale: 1.05
        light_intensity: 4.0
        shake: true
      - time: 0.5
        lid_angle: 150°
        scale: 1.1
        light_intensity: 8.0  # Major burst
      - time: 0.8
        lid_angle: 125°
        scale: 1.0
        light_intensity: 3.0
      - time: 1.2
        lid_angle: 130°
        light_intensity: 2.0
    vfx_trigger:
      - "Screen flash at 0.5s"
      - "Particle burst at 0.5s"
      - "Golden coins spray at 0.6s"
```

### 5.2 Idle Animations

```yaml
idle_animations:
  common:
    enabled: false

  uncommon:
    enabled: true
    type: "Subtle glow pulse"
    duration: 2.0s
    loop: true
    properties:
      emission_range: [0.3, 0.7]

  rare:
    enabled: true
    type: "Glow pulse + gem sparkle"
    duration: 3.0s
    loop: true
    properties:
      emission_range: [0.5, 1.2]
      particle_rate: [10, 20]

  epic:
    enabled: true
    type: "Full magical animation"
    duration: 4.0s
    loop: true
    properties:
      emission_range: [1.0, 2.5]
      rune_scroll: true
      particle_rate: [25, 40]
      light_flicker: true
```

---

## 6. Visual Effects

### 6.1 Particle Systems

```yaml
particle_systems:
  uncommon_sparkle:
    type: "Simple sparkle"
    emission_rate: 5/sec
    lifetime: 1.5s
    size: 0.05m
    color: "#2ECC71"
    velocity: "0.2m/s upward"
    spawn_area: "Chest surface"

  rare_sparkle:
    type: "Magical sparkle"
    emission_rate: 15/sec
    lifetime: 2.0s
    size: 0.08m
    color_gradient:
      - time: 0.0
        color: "#FFFFFF"
      - time: 0.5
        color: "#3498DB"
      - time: 1.0
        color: "#3498DB00"
    velocity: "0.3m/s spiral upward"
    spawn_area: "Chest + 0.5m radius"

  epic_sparkle:
    type: "Legendary aura"
    emission_rate: 30/sec
    lifetime: 2.5s
    size_range: [0.05m, 0.15m]
    color_gradient:
      - time: 0.0
        color: "#FFD700"  # Gold
      - time: 0.3
        color: "#9B59B6"  # Purple
      - time: 1.0
        color: "#9B59B600"
    velocity: "0.5m/s orbit + rise"
    spawn_area: "Chest + 1m radius"
    sub_emitters:
      - type: "Gold coin particles"
        rate: 5/sec

  open_burst_epic:
    type: "One-shot burst"
    particle_count: 100
    lifetime: 1.5s
    size_range: [0.1m, 0.3m]
    colors: ["#FFD700", "#9B59B6", "#FFFFFF"]
    velocity: "3m/s outward sphere"
    gravity: -2.0
```

### 6.2 Light Effects

```yaml
light_effects:
  uncommon_glow:
    type: "Point light"
    color: "#2ECC71"
    intensity: 0.5
    range: 2m
    shadow: false

  rare_glow:
    type: "Point light"
    color: "#3498DB"
    intensity: 1.0
    range: 4m
    shadow: false
    flicker:
      enabled: true
      speed: 2.0
      intensity_range: [0.8, 1.2]

  epic_glow:
    type: "Point light + spot"
    point_light:
      color: "#9B59B6"
      intensity: 2.0
      range: 6m
    spot_light:
      color: "#FFD700"
      intensity: 3.0
      angle: 30°
      direction: "Up"
      range: 15m
    shadow: false

  epic_light_pillar:
    type: "Volumetric beam"
    color: "#9B59B640"
    width: 0.5m
    height: 10m
    rotation: "Slow spin"
    visible_distance: 100m
```

### 6.3 Post-Process Effects

```yaml
post_process_open:
  epic_only: true

  screen_flash:
    color: "#FFD70080"
    duration: 0.3s
    fade_out: true

  chromatic_aberration:
    intensity: 0.5
    duration: 0.5s

  bloom_boost:
    intensity_add: 0.5
    duration: 0.8s
```

---

## 7. Audio Design

### 7.1 Sound Effects

```yaml
audio_effects:
  common:
    ambient: null
    discover: "sfx_chest_creak_wood"
    open: "sfx_chest_open_basic"
    loot_reveal: "sfx_loot_common"

  uncommon:
    ambient: "sfx_chest_hum_subtle"
    ambient_volume: 0.3
    discover: "sfx_chest_shimmer_brass"
    open: "sfx_chest_open_quality"
    loot_reveal: "sfx_loot_uncommon"

  rare:
    ambient: "sfx_chest_magical_pulse"
    ambient_volume: 0.5
    discover: "sfx_chest_chime_magical"
    open: "sfx_chest_open_ornate"
    open_extra: "sfx_magical_whoosh"
    loot_reveal: "sfx_loot_rare"

  epic:
    ambient: "sfx_chest_legendary_aura"
    ambient_volume: 0.8
    discover: "sfx_chest_fanfare_legendary"
    open: "sfx_chest_open_legendary"
    open_extra:
      - "sfx_magical_explosion"
      - "sfx_coins_scatter"
      - "sfx_choir_hit"
    loot_reveal: "sfx_loot_epic"

  audio_settings:
    spatial_blend: 1.0  # Full 3D
    max_distance: 30m
    rolloff: "Logarithmic"
```

### 7.2 Audio Cue Distances

```yaml
audio_distances:
  common:
    ambient_range: 0m  # No ambient
    open_range: 15m

  uncommon:
    ambient_range: 10m
    open_range: 20m

  rare:
    ambient_range: 20m
    open_range: 30m

  epic:
    ambient_range: 40m
    open_range: 50m
```

---

## 8. Interaction System

### 8.1 Interaction Configuration

```yaml
interaction:
  detection:
    method: "Sphere overlap"
    radius: 2.5m
    angle_requirement: 180°  # Front hemisphere

  prompt:
    text: "Open Chest"
    key_binding: "Interact"
    mobile: "Tap chest"
    hold_time: 0s  # Instant

  timing:
    channel_time: 0s  # No channel
    animation_lock: "Until open animation complete"
    cooldown: 0s

  states:
    closed:
      interactable: true
      collision: true
      highlight_on_hover: true

    opening:
      interactable: false
      collision: true

    open:
      interactable: false
      collision: true
      despawn_timer: 30s  # Empty chest despawns

  multiplayer:
    first_interact_wins: true
    network_authority: "Server"
```

### 8.2 Highlight System

```yaml
highlight_system:
  hover_highlight:
    type: "Outline + glow boost"
    outline_color: "#FFFFFF"
    outline_width: 2px
    glow_multiplier: 1.5
    activation: "Player within 5m and looking at"

  interact_prompt:
    position: "Above chest"
    offset: "0.5m up"
    billboard: true
    scale_with_distance: true
```

---

## 9. Spawning and Placement

### 9.1 Spawn Point Configuration

```yaml
spawn_configuration:
  placement:
    ground_snap: true
    rotation: "Random Y-axis"
    min_spacing: 3m
    max_slope: 15°

  spawn_timing:
    match_start: "All chests spawn"
    respawn: false  # One-time loot

  rarity_distribution:
    method: "Weighted random per zone"

    poi_high_tier:
      common: 40%
      uncommon: 35%
      rare: 20%
      epic: 5%

    poi_medium_tier:
      common: 55%
      uncommon: 30%
      rare: 13%
      epic: 2%

    poi_low_tier:
      common: 70%
      uncommon: 22%
      rare: 7%
      epic: 1%

    wilderness:
      common: 80%
      uncommon: 15%
      rare: 4%
      epic: 1%
```

### 9.2 Visual Spawn Markers (Editor)

```yaml
editor_markers:
  spawn_point_gizmo:
    shape: "Chest icon"
    color_by_zone:
      high_tier: "#FFD700"
      medium_tier: "#C0C0C0"
      low_tier: "#CD7F32"
      wilderness: "#808080"

  debug_view:
    show_rarity_preview: true
    show_spawn_radius: true
    show_probability: true
```

---

## 10. Performance Optimization

### 10.1 Draw Call Budget

```yaml
performance_budget:
  per_chest:
    draw_calls: 1-2
    target: 1 (single material)

  scene_total:
    max_visible_chests: 50
    max_draw_calls: 75

  optimization_strategies:
    - "Shared texture atlas"
    - "GPU instancing where possible"
    - "LOD system"
    - "Distance-based particle culling"
    - "Light culling for glow effects"
```

### 10.2 Memory Budget

```yaml
memory_budget:
  textures:
    shared_atlas: 1.33 MB  # 1024x1024 ASTC
    emission_maps: 0.33 MB  # 512x512 ASTC

  meshes:
    all_variants: 0.5 MB

  particles:
    systems: 0.2 MB

  total: ~2.5 MB
```

### 10.3 Performance Settings by Quality

```yaml
quality_tiers:
  high:
    particles: "Full"
    lights: "All enabled"
    emission: "Full quality"
    lod_bias: 0

  medium:
    particles: "50% rate"
    lights: "Rare+ only"
    emission: "Reduced intensity"
    lod_bias: 1

  low:
    particles: "25% rate, no epic burst"
    lights: "Epic only"
    emission: "Minimal"
    lod_bias: 2
```

---

## 11. Integration

### 11.1 Loot System Integration

```csharp
// ChestController.cs
public class ChestController : MonoBehaviour, IInteractable
{
    [SerializeField] private ChestRarity rarity;
    [SerializeField] private ChestVisuals visuals;
    [SerializeField] private ChestAudio audio;

    private bool isOpen = false;
    private LootTable lootTable;

    public void Initialize(ChestRarity rarity, LootTable loot)
    {
        this.rarity = rarity;
        this.lootTable = loot;
        visuals.SetRarity(rarity);
        audio.SetRarity(rarity);
    }

    public bool CanInteract(Player player)
    {
        return !isOpen && player.CanLoot();
    }

    public void OnInteract(Player player)
    {
        if (isOpen) return;

        isOpen = true;
        StartCoroutine(OpenSequence(player));
    }

    private IEnumerator OpenSequence(Player player)
    {
        // Play opening animation
        visuals.PlayOpenAnimation();
        audio.PlayOpenSound();

        // Wait for animation
        yield return new WaitForSeconds(visuals.GetOpenDuration());

        // Spawn loot
        var loot = lootTable.Roll(rarity);
        LootSpawner.SpawnLoot(loot, transform.position);

        // Notify UI
        player.OnChestOpened(rarity, loot);

        // Schedule despawn
        StartCoroutine(DespawnAfterDelay(30f));
    }
}

public enum ChestRarity
{
    Common,
    Uncommon,
    Rare,
    Epic
}
```

### 11.2 Prefab Structure

```yaml
prefab_hierarchy:
  Chest_[Rarity]:
    components:
      - ChestController
      - Animator
      - AudioSource
      - BoxCollider
      - InteractionTrigger

    children:
      - Mesh_Closed
      - Mesh_Open (disabled)
      - VFX_Idle (rarity-dependent)
      - VFX_Open (rarity-dependent)
      - Light_Glow (uncommon+)
      - Light_Pillar (epic only)
      - UI_InteractPrompt
```

---

## 12. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|--------------|---------------------|
| Each tier visually distinct | 4 unique designs with color coding | Visual inspection |
| Open and closed states | Both states modeled per variant | Animation test |
| Opening animation or VFX | Animated lid + effects | Playback verification |
| Color coding matches rarity | Gray/Green/Blue/Purple | Color picker verification |
| Readable at distance | Glow visible 50m+ (epic) | Distance test |
| Low poly (200-500 triangles) | 200-500 per variant | Mesh statistics |

---

## 13. Deliverables

### 13.1 Asset List

```yaml
deliverables:
  meshes:
    - "SM_Chest_Common_Closed.fbx"
    - "SM_Chest_Common_Open.fbx"
    - "SM_Chest_Uncommon_Closed.fbx"
    - "SM_Chest_Uncommon_Open.fbx"
    - "SM_Chest_Rare_Closed.fbx"
    - "SM_Chest_Rare_Open.fbx"
    - "SM_Chest_Epic_Closed.fbx"
    - "SM_Chest_Epic_Open.fbx"

  textures:
    - "T_Chests_Atlas_Albedo.png"
    - "T_Chests_Atlas_Normal.png"
    - "T_Chests_Atlas_Mask.png"
    - "T_Chest_Epic_Emission.png"

  materials:
    - "M_Chest_Common.mat"
    - "M_Chest_Uncommon.mat"
    - "M_Chest_Rare.mat"
    - "M_Chest_Epic.mat"

  animations:
    - "Anim_Chest_Open_Common.anim"
    - "Anim_Chest_Open_Uncommon.anim"
    - "Anim_Chest_Open_Rare.anim"
    - "Anim_Chest_Open_Epic.anim"
    - "Anim_Chest_Idle_Uncommon.anim"
    - "Anim_Chest_Idle_Rare.anim"
    - "Anim_Chest_Idle_Epic.anim"

  vfx:
    - "VFX_Chest_Sparkle_Uncommon.prefab"
    - "VFX_Chest_Sparkle_Rare.prefab"
    - "VFX_Chest_Aura_Epic.prefab"
    - "VFX_Chest_Open_Burst_Epic.prefab"

  prefabs:
    - "Chest_Common.prefab"
    - "Chest_Uncommon.prefab"
    - "Chest_Rare.prefab"
    - "Chest_Epic.prefab"

  audio:
    - "SFX_Chest_Open_*.wav" (x4)
    - "SFX_Chest_Ambient_*.wav" (x3)
    - "SFX_Chest_Discover_*.wav" (x4)
```

---

## 14. Appendix

### 14.1 Related Documents

- [VISUAL_STYLE_GUIDE.md](VISUAL_STYLE_GUIDE.md) - Art direction
- [MAP_GREYBOX_BLOCKOUT_SPEC.md](MAP_GREYBOX_BLOCKOUT_SPEC.md) - Spawn placement
- GAME-004: Loot Spawn System (gameplay integration)

### 14.2 Reference Images

```
Visual Reference Descriptions:
- Common: Sea of Thieves basic supply crate
- Uncommon: Classic RPG reinforced chest
- Rare: Diablo 3 resplendent chest
- Epic: World of Warcraft legendary cache
```

### 14.3 Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-30 | Initial specification |
