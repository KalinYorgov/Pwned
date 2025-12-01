# Map Greybox/Blockout Specification

## Document Information
- **Task ID:** ART-008
- **Priority:** P0
- **Complexity:** M (Medium)
- **Dependencies:** ART-007 (Map Design Document)
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the greybox/blockout version of the Plunderstorm Mobile battle royale map. The greybox serves as a fully playable prototype for testing gameplay flow, scale, pacing, and balance before committing art resources to final production.

### 1.2 Scope
- Complete terrain blockout
- All Points of Interest (POIs) in simplified form
- Functional collision and navigation
- Spawn point and loot placement systems
- Storm zone testing capability
- Performance baseline establishment

### 1.3 Goals
- Validate map layout through playtesting
- Establish correct scale for movement speed
- Identify and fix gameplay flow issues
- Lock layout before art production begins
- Create foundation for final art pass

---

## 2. Map Overview

### 2.1 Island Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Treasure Isle - Greybox Layout                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                              N                                           │
│                              ↑                                           │
│                                                                          │
│         ┌─────────────────────────────────────────────┐                  │
│        /    [Shipwreck         [Skull                  \                 │
│       /      Shoals]            Cave]                   \                │
│      │                                                   │               │
│      │   [Mermaid     ████████████     [Cursed          │               │
│      │    Lagoon]     ████████████      Lighthouse]     │               │
│      │                █ VOLCANO █                       │               │
│      │                ████████████                      │               │
│   W ←│    [Smuggler's     ↑      [Port                  │→ E            │
│      │     Cove]      (center)    Plunder]              │               │
│      │                                                   │               │
│      │   [Banana           ██          [Kraken's        │               │
│      │    Bay]          ██████          Reach]          │               │
│      │               (small isle)                       │               │
│      │                                                   │               │
│       \    [Tortuga      [Golden      [Navy            /                │
│        \    Tavern]       Beach]       Fortress]      /                 │
│         └─────────────────────────────────────────────┘                  │
│                                                                          │
│                              ↓                                           │
│                              S                                           │
│                                                                          │
│  Map Dimensions: 1000m x 1000m                                          │
│  Playable Area: ~785,000 m² (circular with irregular coastline)         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Map Specifications

```yaml
map_specifications:
  dimensions:
    total_size: "1000m x 1000m"
    playable_diameter: "~1000m"
    playable_area: "785,000 m²"

  terrain_height:
    sea_level: 0m
    beach_level: "1-3m"
    inland_level: "3-15m"
    hill_peaks: "15-40m"
    volcano_peak: "80m"
    max_height: "100m"

  water:
    ocean_depth: "-10m to -30m"
    shallow_areas: "-1m to -3m"
    swim_zones: "Beaches, lagoon, cove"

  zones:
    poi_count: 10
    monster_camps: 15
    loot_zones: 50+
```

---

## 3. Points of Interest (POIs)

### 3.1 POI Summary Table

| POI Name | Grid | Size | Loot Tier | Monster Camp | Player Capacity |
|----------|------|------|-----------|--------------|-----------------|
| Port Plunder | E4 | Large | High | Yes | 8-12 |
| Volcano Crater | D4 | Medium | Epic | Yes (Boss) | 4-6 |
| Skull Cave | E2 | Medium | High | Yes | 4-6 |
| Cursed Lighthouse | F3 | Small | Medium | Yes | 2-4 |
| Smuggler's Cove | B4 | Medium | High | No | 4-6 |
| Mermaid Lagoon | B3 | Medium | Medium | Yes | 4-6 |
| Shipwreck Shoals | C2 | Large | High | Yes | 6-8 |
| Tortuga Tavern | C6 | Medium | Medium | No | 4-6 |
| Golden Beach | D6 | Small | Low | No | 2-4 |
| Navy Fortress | F6 | Large | High | Yes | 6-10 |
| Banana Bay | B5 | Small | Low | No | 2-4 |
| Kraken's Reach | F5 | Medium | High | Yes | 4-6 |

### 3.2 POI Blockout Details

#### 3.2.1 Port Plunder (Primary POI)

```yaml
port_plunder:
  description: "Main pirate town with harbor, market, and tavern"
  grid_location: "E4"
  size: "150m x 120m"

  greybox_elements:
    buildings:
      - name: "Harbor Master's Office"
        footprint: "15m x 10m"
        height: "8m"
        floors: 2
        enterable: true

      - name: "Market Stalls"
        footprint: "30m x 15m"
        height: "4m"
        style: "Open-air with pillars"
        enterable: true

      - name: "The Salty Dog Tavern"
        footprint: "20m x 15m"
        height: "10m"
        floors: 2
        enterable: true

      - name: "Warehouse"
        footprint: "25m x 20m"
        height: "8m"
        floors: 1
        enterable: true

      - name: "Residential Row"
        count: 4
        footprint: "8m x 8m each"
        height: "6m"
        enterable: false

    exterior:
      - "Dock platforms (3 piers)"
      - "Central plaza with fountain"
      - "Crate stacks for cover"
      - "Barrel clusters"

  gameplay_features:
    loot_spawns: 25
    chest_locations: 8
    cover_density: "High"
    sightlines: "Medium (blocked by buildings)"
    verticality: "Medium (2-story buildings)"

  collision_notes:
    - "All building exteriors solid"
    - "Interior doorways 2m wide"
    - "Rooftops accessible via ladder"
    - "Pier railings as partial cover"
```

#### 3.2.2 Volcano Crater

```yaml
volcano_crater:
  description: "Central volcano with interior crater access"
  grid_location: "D4"
  size: "100m diameter"

  greybox_elements:
    terrain:
      - name: "Outer Slopes"
        angle: "30-45 degrees"
        traversable: "Yes, with reduced speed"

      - name: "Crater Rim"
        width: "10m"
        height: "80m above sea level"

      - name: "Inner Crater"
        diameter: "60m"
        depth: "30m from rim"
        floor: "Lava pools (damage zones)"

    structures:
      - name: "Ancient Temple Ruins"
        location: "Crater floor"
        footprint: "20m x 20m"
        cover: "Broken pillars, walls"

      - name: "Rim Outposts"
        count: 3
        purpose: "Sniper positions"

    access_points:
      - "North path (gradual slope)"
      - "South cave entrance"
      - "East rope bridge (from lighthouse)"

  gameplay_features:
    loot_spawns: 15
    chest_locations: 5 (including 1 Epic)
    boss_spawn: true
    environmental_hazard: "Lava pools (tick damage)"

  collision_notes:
    - "Slope collision allows movement"
    - "Lava pools as trigger volumes"
    - "Crater walls block vertical movement"
```

#### 3.2.3 Other POIs (Simplified)

```yaml
skull_cave:
  size: "80m x 60m"
  features:
    - "Large skull-shaped entrance (30m tall)"
    - "Interior cave system (3 chambers)"
    - "Underground waterway"
    - "Multiple entrances/exits"
  loot_spawns: 12
  monster_camp: true

cursed_lighthouse:
  size: "40m diameter"
  features:
    - "Lighthouse tower (40m tall)"
    - "Keeper's house"
    - "Rocky cliffs around"
    - "Spiral staircase interior"
  loot_spawns: 8
  verticality: "High"

smugglers_cove:
  size: "100m x 80m"
  features:
    - "Hidden bay with cave entrance"
    - "Smuggler camp (tents, crates)"
    - "Beached rowboats"
    - "Hidden treasure alcove"
  loot_spawns: 14

mermaid_lagoon:
  size: "90m x 70m"
  features:
    - "Shallow lagoon (swim zone)"
    - "Central rock formation"
    - "Coral reef obstacles"
    - "Underwater cave entrance"
  loot_spawns: 10
  special: "Some loot underwater"

shipwreck_shoals:
  size: "150m x 100m"
  features:
    - "3 shipwrecks of varying sizes"
    - "Shallow sandbars"
    - "Debris fields"
    - "Crow's nest sniper points"
  loot_spawns: 18

navy_fortress:
  size: "120m x 100m"
  features:
    - "Stone walls with battlements"
    - "Central courtyard"
    - "Tower (4 stories)"
    - "Underground dungeon"
  loot_spawns: 20
  verticality: "High"
```

---

## 4. Terrain System

### 4.1 Terrain Types

```yaml
terrain_types:
  beach:
    height_range: "0-3m"
    movement_modifier: 1.0
    material: "Sand"
    greybox_color: "#F5DEB3"  # Wheat

  grass:
    height_range: "3-15m"
    movement_modifier: 1.0
    material: "Grass"
    greybox_color: "#90EE90"  # Light green

  jungle:
    height_range: "5-20m"
    movement_modifier: 0.9
    material: "Dense vegetation"
    greybox_color: "#228B22"  # Forest green
    visibility: "Reduced"

  rock:
    height_range: "10-80m"
    movement_modifier: 0.8
    material: "Stone"
    greybox_color: "#808080"  # Gray

  volcanic:
    height_range: "40-100m"
    movement_modifier: 0.7
    material: "Volcanic rock"
    greybox_color: "#2F4F4F"  # Dark slate

  water_shallow:
    height_range: "-3m to 0m"
    movement_modifier: 0.6
    material: "Water"
    greybox_color: "#87CEEB"  # Sky blue
    swim: true

  water_deep:
    height_range: "< -3m"
    movement_modifier: 0.4
    material: "Deep water"
    greybox_color: "#4169E1"  # Royal blue
    swim: true
```

### 4.2 Heightmap Specifications

```yaml
heightmap:
  resolution: "1024 x 1024"
  meters_per_pixel: "~1m"
  height_encoding: "16-bit grayscale"

  generation_rules:
    - "Central volcano peak at 80m"
    - "Coastal areas slope to sea level"
    - "Hills between 15-40m"
    - "Valley paths for natural routes"
    - "Cliff faces for tactical positions"

  performance:
    terrain_chunks: "16 x 16 grid"
    chunk_size: "62.5m x 62.5m"
    lod_levels: 4
```

### 4.3 Terrain Blockout Mesh

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Terrain Cross-Section (W to E)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Height                                                                  │
│  (m)                                                                     │
│                                                                          │
│  80 ─                        ▲ Volcano                                   │
│      │                      /█\                                          │
│  60 ─│                     / █ \                                         │
│      │                    /  █  \                                        │
│  40 ─│      ╱╲           /   █   \          ╱╲                           │
│      │     ╱  ╲         /    █    \        ╱  ╲    Navy                  │
│  20 ─│    ╱    ╲       /     █     \      ╱    ╲   Fortress              │
│      │   ╱      ╲     /      █      \    ╱      ╲  ┌──┐                  │
│  10 ─│  ╱ Cove   ╲   /       █       \  ╱ Port   ╲ │  │                  │
│      │ ╱          ╲_/        █        ╲╱ Plunder  ╲│  │                  │
│   0 ─│_____╲______/~~~~~~~~~~█~~~~~~~~~~\__________╲__│___               │
│      │     Water            Center           Water                       │
│ -10 ─│~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~              │
│                                                                          │
│      W ─────────────────────────────────────────────────── E             │
│      0m                     500m                        1000m            │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Collision System

### 5.1 Collision Layers

```yaml
collision_layers:
  terrain:
    layer: 0
    description: "Ground collision for walking"
    type: "Mesh collider (simplified)"

  buildings:
    layer: 1
    description: "Building exteriors and walls"
    type: "Box/mesh colliders"

  props:
    layer: 2
    description: "Cover objects, crates, barrels"
    type: "Box colliders"

  water:
    layer: 3
    description: "Water volume triggers"
    type: "Trigger volumes"

  damage_zones:
    layer: 4
    description: "Lava, storm damage"
    type: "Trigger volumes"

  navigation_blockers:
    layer: 5
    description: "Invisible walls, boundaries"
    type: "Box colliders"
```

### 5.2 Collision Mesh Guidelines

```yaml
collision_guidelines:
  terrain:
    simplification: "10% of visual mesh"
    max_triangles: 50000
    update_frequency: "Static"

  buildings:
    approach: "Primitive shapes preferred"
    convex_only: true
    max_colliders_per_building: 10

  cover_objects:
    height_categories:
      low_cover: "0.5-1.0m (crouch behind)"
      medium_cover: "1.0-1.5m (stand behind)"
      full_cover: "1.5m+ (blocks all)"
    width_minimum: "1m"

  performance_budget:
    max_active_colliders: 5000
    physics_update_rate: "50 Hz"
```

### 5.3 Navigation Mesh

```yaml
navmesh:
  agent_settings:
    radius: 0.5m
    height: 2.0m
    step_height: 0.4m
    max_slope: 45°

  generation:
    voxel_size: 0.2m
    region_min_size: 8
    merge_distance: 0.3m

  areas:
    walkable:
      cost: 1.0
      color: "#00FF00"

    sand:
      cost: 1.0
      color: "#FFFF00"

    water_shallow:
      cost: 1.5
      color: "#00FFFF"

    jungle:
      cost: 1.2
      color: "#008000"

    not_walkable:
      cost: -1
      color: "#FF0000"

  off_mesh_links:
    - type: "Jump down"
      max_height: 4m
    - type: "Ladder"
      bidirectional: true
    - type: "Zipline"
      one_way: true
```

---

## 6. Spawn System

### 6.1 Player Spawn Points

```yaml
player_spawns:
  drop_zone:
    type: "Air drop pattern"
    altitude: "200m above terrain"
    spawn_area: "Full map diameter"
    pattern: "Random within plane path"

  plane_paths:
    count: 8  # Randomized per match
    examples:
      - start: "NW corner"
        end: "SE corner"
      - start: "N edge"
        end: "S edge"
      - start: "W edge"
        end: "E edge"
    flight_time: "60 seconds edge to edge"

  landing_zones:
    safe_radius: "5m (no overlap)"
    terrain_check: "Must be walkable"
    water_landing: "Allowed (swim to shore)"
```

### 6.2 Loot Spawn System

```yaml
loot_spawns:
  total_spawn_points: 500+

  spawn_categories:
    floor_loot:
      description: "Ground pickups"
      spawn_rate: 80%
      respawn: false
      marker: "Yellow sphere (greybox)"

    chest_spawns:
      description: "Treasure chests"
      spawn_rate: 100%
      respawn: false
      marker: "Orange box (greybox)"
      tiers:
        common: 60%
        uncommon: 25%
        rare: 12%
        epic: 3%

    ability_orbs:
      description: "Ability pickups"
      spawn_rate: 70%
      respawn: false
      marker: "Purple sphere (greybox)"

  poi_loot_density:
    high_tier_poi: "15-25 spawns"
    medium_tier_poi: "8-15 spawns"
    low_tier_poi: "4-8 spawns"
    wilderness: "1-3 spawns per 100m²"

  greybox_visualization:
    spawn_point_mesh: "Cube (0.5m)"
    color_coding:
      floor_loot: "#FFFF00"  # Yellow
      chest_common: "#808080"  # Gray
      chest_uncommon: "#00FF00"  # Green
      chest_rare: "#0000FF"  # Blue
      chest_epic: "#800080"  # Purple
      ability_orb: "#FF00FF"  # Magenta
```

### 6.3 Monster Camp Spawns

```yaml
monster_camps:
  total_camps: 15

  camp_sizes:
    small:
      monsters: "3-5"
      area: "10m radius"
      reward: "Common-Uncommon loot"
      respawn: "3 minutes"

    medium:
      monsters: "5-8"
      area: "15m radius"
      reward: "Uncommon-Rare loot"
      respawn: "4 minutes"

    large:
      monsters: "8-12"
      area: "20m radius"
      reward: "Rare-Epic loot"
      respawn: "5 minutes"

    boss:
      monsters: "1 boss + 4 minions"
      area: "25m radius"
      reward: "Epic loot guaranteed"
      respawn: "Never (once per match)"
      location: "Volcano crater"

  greybox_visualization:
    camp_boundary: "Red circle on ground"
    spawn_points: "Red cubes (0.3m)"
    boss_spawn: "Large red sphere"
```

---

## 7. Storm/Zone System

### 7.1 Storm Configuration

```yaml
storm_system:
  phases:
    - phase: 0
      name: "Pre-game"
      safe_zone_radius: 500m  # Full map
      duration: 60s
      storm_damage: 0

    - phase: 1
      name: "First Circle"
      safe_zone_radius: 400m
      shrink_time: 90s
      hold_time: 120s
      storm_damage: 1/s

    - phase: 2
      name: "Second Circle"
      safe_zone_radius: 280m
      shrink_time: 75s
      hold_time: 90s
      storm_damage: 2/s

    - phase: 3
      name: "Third Circle"
      safe_zone_radius: 180m
      shrink_time: 60s
      hold_time: 75s
      storm_damage: 5/s

    - phase: 4
      name: "Fourth Circle"
      safe_zone_radius: 100m
      shrink_time: 45s
      hold_time: 60s
      storm_damage: 8/s

    - phase: 5
      name: "Fifth Circle"
      safe_zone_radius: 50m
      shrink_time: 30s
      hold_time: 45s
      storm_damage: 10/s

    - phase: 6
      name: "Final Circle"
      safe_zone_radius: 10m
      shrink_time: 30s
      hold_time: "Until end"
      storm_damage: 15/s

  zone_center_rules:
    - "Always on land (not water)"
    - "Weighted toward POIs"
    - "Never same spot twice in row"
    - "Randomized within constraints"
```

### 7.2 Storm Visualization (Greybox)

```yaml
storm_greybox_visuals:
  safe_zone:
    indicator: "White circle on minimap"
    edge_marker: "White vertical planes (transparent)"
    ground_decal: "White ring"

  storm_wall:
    color: "#800080" (purple, semi-transparent)
    height: "100m (blocks sky)"
    thickness: "5m visual"

  storm_interior:
    post_process: "Purple tint"
    fog_density: "Increased"
    visibility: "50m max"

  warning_indicators:
    phase_warning: "30s before shrink"
    audio_cue: "Horn sound"
    ui_element: "Timer + next zone preview"
```

### 7.3 Zone Testing Tools

```yaml
zone_testing_tools:
  admin_commands:
    - "/zone skip" - Skip to next phase
    - "/zone pause" - Pause current phase
    - "/zone center [x,y]" - Set zone center
    - "/zone radius [m]" - Set zone radius
    - "/zone damage [n]" - Set storm damage

  visualization_toggles:
    - Show all phase circles
    - Show zone center markers
    - Show damage zone boundary
    - Display phase timeline

  testing_scenarios:
    - "Full match simulation"
    - "Rapid zone collapse (2x speed)"
    - "Static zone (no shrink)"
    - "Custom zone placement"
```

---

## 8. Scale and Movement

### 8.1 Movement Speed Reference

```yaml
movement_speeds:
  walk: 3.5 m/s
  run: 6.0 m/s
  sprint: 8.0 m/s
  swim: 4.0 m/s
  crouch: 2.0 m/s

  traversal_times:
    edge_to_edge_sprint: "125 seconds (1000m)"
    poi_to_adjacent_poi: "20-40 seconds"
    storm_outrun_phase1: "Possible at run speed"
    storm_outrun_phase5: "Difficult, need sprint"

  scale_validation:
    poi_spacing: "100-200m between major POIs"
    cover_spacing: "10-20m in open areas"
    engagement_range: "20-50m typical combat"
```

### 8.2 Scale Testing Checklist

```yaml
scale_tests:
  traversal:
    - [ ] Can cross map in reasonable time (<3 min sprint)
    - [ ] POI-to-POI feels appropriate (30-60s)
    - [ ] Can outrun Phase 1-2 storm walking
    - [ ] Swimming doesn't feel punishing

  combat:
    - [ ] Typical engagement at 20-50m
    - [ ] Sniper positions effective at 80-100m
    - [ ] Melee viable when closing distance
    - [ ] Cover spaced appropriately

  pacing:
    - [ ] Early game: spread out, looting
    - [ ] Mid game: rotations, encounters
    - [ ] Late game: tight fights, tension
    - [ ] Match length: 10-15 minutes
```

### 8.3 Landmark Visibility

```yaml
landmark_visibility:
  requirements:
    - "Volcano visible from anywhere on map"
    - "Each POI has unique silhouette"
    - "Lighthouse visible from 500m"
    - "Navy Fortress visible from 400m"

  greybox_markers:
    volcano:
      marker: "Tall cone (80m)"
      color: "#8B0000"

    lighthouse:
      marker: "Tall cylinder (40m)"
      color: "#FFFFFF"

    fortress:
      marker: "Large cube cluster"
      color: "#4682B4"

    other_pois:
      marker: "Labeled boxes"
      color: "Varies by POI"
```

---

## 9. Performance Requirements

### 9.1 Greybox Performance Targets

```yaml
performance_targets:
  frame_rate:
    target: 60 FPS
    minimum: 30 FPS
    test_devices:
      - "iPhone 12 (baseline)"
      - "Samsung Galaxy S21"
      - "Mid-range Android (SD 700 series)"

  draw_calls:
    terrain: "<100"
    poi_structures: "<200"
    total_greybox: "<400"

  memory:
    terrain_mesh: "<50 MB"
    collision_data: "<20 MB"
    total_greybox: "<100 MB"

  loading:
    initial_load: "<10 seconds"
    streaming: "Not required for greybox"
```

### 9.2 Optimization Strategies

```yaml
optimization_greybox:
  terrain:
    - "LOD system with 4 levels"
    - "Chunk-based loading"
    - "Simplified collision mesh"

  structures:
    - "Primitive shapes only"
    - "Instanced rendering where possible"
    - "No interior detail meshes"

  culling:
    - "Frustum culling enabled"
    - "Occlusion culling for POIs"
    - "Distance-based detail reduction"
```

---

## 10. Greybox Asset Specifications

### 10.1 Material Palette

```yaml
greybox_materials:
  terrain_grass:
    color: "#90EE90"
    roughness: 0.8

  terrain_sand:
    color: "#F5DEB3"
    roughness: 0.9

  terrain_rock:
    color: "#808080"
    roughness: 0.7

  terrain_volcanic:
    color: "#2F4F4F"
    roughness: 0.6

  water_shallow:
    color: "#87CEEB"
    transparency: 0.5

  water_deep:
    color: "#4169E1"
    transparency: 0.7

  structure_wood:
    color: "#8B4513"
    roughness: 0.8

  structure_stone:
    color: "#696969"
    roughness: 0.6

  cover_object:
    color: "#A0522D"
    roughness: 0.7

  danger_zone:
    color: "#FF0000"
    transparency: 0.3
```

### 10.2 Primitive Shapes Library

```yaml
primitive_library:
  cubes:
    sizes: [1m, 2m, 4m, 8m]
    uses: "Buildings, crates, walls"

  cylinders:
    sizes: [1m, 2m, 4m diameter]
    uses: "Towers, barrels, pillars"

  ramps:
    angles: [15°, 30°, 45°]
    uses: "Terrain transitions, stairs"

  wedges:
    sizes: [2m, 4m]
    uses: "Roofs, slopes"

  planes:
    uses: "Floors, platforms, water surface"

  custom:
    cliff_face: "Modular cliff sections"
    cave_entrance: "Arch primitive"
    dock_section: "Platform with supports"
```

---

## 11. Testing Protocol

### 11.1 Playtest Schedule

```yaml
playtest_schedule:
  phase_1_layout:
    duration: "Week 1-2"
    focus: "Terrain and POI placement"
    player_count: "10-20 (internal)"
    feedback_areas:
      - "Scale feels right?"
      - "POIs balanced?"
      - "Navigation intuitive?"

  phase_2_gameplay:
    duration: "Week 3-4"
    focus: "Loot, spawns, storm"
    player_count: "30-40"
    feedback_areas:
      - "Loot distribution fair?"
      - "Storm pacing good?"
      - "Enough cover?"

  phase_3_polish:
    duration: "Week 5"
    focus: "Final adjustments"
    player_count: "60 (full match)"
    feedback_areas:
      - "Match flow satisfying?"
      - "Any dead zones?"
      - "Ready for art pass?"
```

### 11.2 Feedback Categories

```yaml
feedback_tracking:
  categories:
    layout:
      - POI size
      - POI spacing
      - Terrain flow
      - Choke points

    gameplay:
      - Loot balance
      - Combat distances
      - Cover availability
      - Verticality

    pacing:
      - Match length
      - Storm timing
      - Player density
      - Engagement frequency

    technical:
      - Performance
      - Collision issues
      - Navigation problems
      - Visual clarity

  tracking_tool: "Spreadsheet with heat maps"
```

### 11.3 Iteration Process

```yaml
iteration_process:
  daily:
    - Review playtest feedback
    - Identify critical issues
    - Implement quick fixes

  weekly:
    - Larger layout changes
    - Rebalance POIs
    - Adjust storm timing

  milestone_gates:
    - "Layout Lock: No more POI changes"
    - "Gameplay Lock: Loot/spawn frozen"
    - "Art Pass Ready: Greybox complete"
```

---

## 12. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|--------------|---------------------|
| All terrain features blocked out | 1000x1000m terrain with varied elevation | Visual inspection in editor |
| All POIs represented | 10 POIs with basic geometry | Checklist walkthrough |
| Collision working correctly | No fall-through, proper blocking | QA collision test pass |
| Scale feels right for movement | Edge-to-edge <3min sprint | Timed traversal tests |
| Spawn points placeable | 500+ loot, 15 camps defined | Spawn visualization |
| Storm zones testable | All 6 phases functional | Storm test scenarios |
| Playable for full matches | 60-player matches complete | Full playtest sessions |

---

## 13. Deliverables

### 13.1 Asset Deliverables

```yaml
deliverables:
  unity_scene:
    - "Map_Greybox_TreasureIsle.unity"

  terrain:
    - "Terrain_Greybox.asset"
    - "Heightmap_TreasureIsle.raw"

  prefabs:
    - "POI_PortPlunder_Greybox.prefab"
    - "POI_Volcano_Greybox.prefab"
    - "POI_[Name]_Greybox.prefab" (x10)
    - "SpawnPoint_Loot.prefab"
    - "SpawnPoint_Monster.prefab"
    - "StormController.prefab"

  materials:
    - "M_Greybox_[Type].mat" (x10)

  documentation:
    - "This specification"
    - "Playtest feedback summary"
    - "Known issues list"
```

### 13.2 Handoff to Art

```yaml
art_handoff:
  locked_elements:
    - Terrain heightmap
    - POI locations and sizes
    - Building footprints
    - Collision volumes

  flexible_elements:
    - Visual style within footprints
    - Prop placement density
    - Material choices
    - Lighting approach

  reference_materials:
    - Scale markers in scene
    - POI naming conventions
    - Grid overlay system
    - Screenshot documentation
```

---

## 14. Appendix

### 14.1 Related Documents

- [MAP_DESIGN_DOCUMENT.md](MAP_DESIGN_DOCUMENT.md) - Original map design
- [VISUAL_STYLE_GUIDE.md](VISUAL_STYLE_GUIDE.md) - Art direction for final pass
- [TECHNICAL_ART_SPECIFICATIONS.md](TECHNICAL_ART_SPECIFICATIONS.md) - Performance targets

### 14.2 Greybox Color Legend

| Color | Hex | Meaning |
|-------|-----|---------|
| Light Green | #90EE90 | Grass terrain |
| Wheat | #F5DEB3 | Sand/beach |
| Gray | #808080 | Rock |
| Dark Slate | #2F4F4F | Volcanic |
| Sky Blue | #87CEEB | Shallow water |
| Royal Blue | #4169E1 | Deep water |
| Yellow | #FFFF00 | Floor loot spawn |
| Orange | #FFA500 | Chest spawn |
| Purple | #800080 | Epic chest / Storm |
| Red | #FF0000 | Monster camp / Danger |

### 14.3 Coordinate System

```yaml
coordinate_system:
  origin: "Center of map (500, 500)"
  x_axis: "West (-) to East (+)"
  y_axis: "Up (+)"
  z_axis: "South (-) to North (+)"

  grid_system:
    cells: "A1 to J10 (100m squares)"
    naming: "Column (A-J) + Row (1-10)"
    example: "Port Plunder = E4"
```
