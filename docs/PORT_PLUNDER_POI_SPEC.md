# Port Plunder - Named Location Specification

## Document Information
- **Task ID:** ART-009
- **Priority:** P0
- **Complexity:** L (Large)
- **Dependencies:** ART-007 (Map Design Document), ART-008 (Map Greybox/Blockout)
- **Last Updated:** 2025-12-01

---

## 1. Overview

### 1.1 Purpose
This specification defines Port Plunder, the primary S-Tier POI (Point of Interest) for Plunderstorm Mobile. As the largest and most iconic named location, Port Plunder serves as the template for environment art quality and sets the visual standard for all other POIs.

### 1.2 Location Summary

```yaml
poi_identity:
  name: "Port Plunder"
  grid_location: "B5-C5"
  map_region: "South-Central"
  tier: "S (Hot Zone)"
  theme: "Bustling Pirate Port Town"

  dimensions:
    width: 200m
    depth: 180m
    height_range: "0m to 35m"
    total_area: "~36,000 m²"

  capacity:
    player_capacity: 20
    recommended_drop: 12-15
    survival_rate: 30%
```

### 1.3 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Visual Identity | Instantly recognizable pirate port aesthetic | Critical |
| Gameplay Depth | Multiple engagement distances and strategies | Critical |
| Performance | 60 FPS on mid-range devices with all players | Critical |
| Verticality | Multi-level combat opportunities | High |
| Navigation | Intuitive layout with memorable landmarks | High |
| Cover Density | Abundant tactical options without clutter | High |

### 1.4 Narrative Context

Port Plunder was once the most prosperous pirate haven in the Caribbean seas. Founded by the legendary Pirate King Blackstorm, it served as the central trading hub for ill-gotten goods, recruits, and secrets. When the curse fell upon the island, the port froze in time - taverns still filled with ghostly echoes, markets still stocked with cursed wares.

---

## 2. Layout Design

### 2.1 District Overview

```
                          NORTH (To Crossroads/Temple)
                                    |
    +------------------------------------------------------------------+
    |                                                                    |
    |   [Warehouse Row]          [Clock Tower]         [Fish Market]    |
    |       ████████               ▲▲▲                    ████          |
    |       ████████               ███                    ████          |
    |                              ███                                   |
    |   -------- MAIN STREET ---------------------------------------   |
    |                                                                    |
    |   [Salty Siren]    [Town Square]    [Harbor Master]               |
    |     ████████         ○ ○ ○ ○          ████████                    |
    |     ████████         ○     ○          ████████                    |
    |     ████████         ○ ○ ○ ○          ████████                    |
    |                                                                    |
    |   -------- HARBOR PROMENADE ----------------------------------   |
    |                                                                    |
    |   [Dock Yard]      [Central Pier]      [Small Docks]             |
    |     ████             ═══════════         ══  ══                   |
    |     ████             ═══════════         ══  ══                   |
    |                      ═══════════                                   |
    |                                                                    |
    |~~~~~~~~~~~~~~~~~~~~~~~~ WATER ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|
    |                                                                    |
    +------------------------------------------------------------------+
                          SOUTH (Ocean/Water)

    Legend:
    ████ = Building footprint
    ▲▲▲  = Tower structure
    ○    = Town square elements
    ═══  = Dock/pier
    ~~~  = Water
```

### 2.2 District Breakdown

```yaml
districts:
  market_district:
    location: "Northeast"
    area: "~6,000 m²"
    character: "Open-air trading, fish stalls, busy atmosphere"
    structures:
      - Fish Market (main building)
      - Market Stalls (×8)
      - Storage Sheds (×3)
    cover_type: "Medium density, destructible"

  warehouse_district:
    location: "Northwest"
    area: "~8,000 m²"
    character: "Dense storage buildings, narrow alleys"
    structures:
      - Large Warehouse (×2)
      - Small Warehouse (×3)
      - Loading Docks
    cover_type: "High density, mixed"

  town_center:
    location: "Central"
    area: "~5,000 m²"
    character: "Social hub, open plaza, landmark tower"
    structures:
      - Clock Tower (landmark)
      - Town Square (open)
      - Notice Board
      - Well/Fountain
    cover_type: "Low density, exposed"

  tavern_district:
    location: "West-Central"
    area: "~4,000 m²"
    character: "Entertainment, multi-floor building"
    structures:
      - The Salty Siren Tavern (3 floors)
      - Back Alley
      - Outdoor Seating
    cover_type: "Interior-focused"

  harbor_office:
    location: "East-Central"
    area: "~3,000 m²"
    character: "Official building, elevated position"
    structures:
      - Harbor Master's Office (2 floors)
      - Lighthouse Signal
      - Customs House
    cover_type: "Medium density"

  dock_district:
    location: "South"
    area: "~10,000 m²"
    character: "Waterfront, open piers, boat access"
    structures:
      - Main Pier
      - Dock Yard (repair area)
      - Small Docks (×4)
      - Moored Boats (×6)
    cover_type: "Low-medium, linear"
```

### 2.3 Elevation Map

```yaml
elevation_zones:
  water_level:
    height: "-1m to 0m"
    areas: ["Docks", "Pier undersides", "Smuggler tunnels"]

  ground_level:
    height: "0m to 3m"
    areas: ["Streets", "Town Square", "Market floor"]

  building_level_1:
    height: "3m to 6m"
    areas: ["Building first floors", "Warehouse platforms"]

  building_level_2:
    height: "6m to 10m"
    areas: ["Tavern second floor", "Office balconies"]

  rooftop_level:
    height: "10m to 15m"
    areas: ["Flat roofs", "Rooftop patios"]

  tower_level:
    height: "15m to 35m"
    areas: ["Clock Tower", "Harbor Master tower"]

elevation_diagram: |
  35m ─ ▲ Clock Tower Peak
  30m ─ │
  25m ─ │
  20m ─ │
  15m ─ ├──────── Rooftops ────────
  10m ─ │    ████████████████
   6m ─ │    ████████████████
   3m ─ │    ████████████████
   0m ─ ├════════════════════════ Ground/Street
  -1m ─ └──── Water Level ────
```

---

## 3. Structures

### 3.1 The Salty Siren Tavern

```yaml
salty_siren_tavern:
  importance: "Primary landmark structure"
  loot_level: "High"
  player_capacity: 6

  dimensions:
    footprint: "20m × 15m"
    height: "12m (3 floors)"

  floors:
    ground_floor:
      name: "Main Hall"
      features:
        - Bar counter (hard cover)
        - Tables and chairs (×8, destructible)
        - Stage area
        - Kitchen entrance
        - Main entrance (south)
        - Back door (north)
      loot_points: 4

    second_floor:
      name: "Guest Rooms"
      features:
        - Balcony overlooking main hall
        - Private rooms (×4)
        - Stairwell access (×2)
        - Window positions (×6)
      loot_points: 3

    third_floor:
      name: "Owner's Suite"
      features:
        - Large room with desk
        - Balcony (exterior, overlooks street)
        - Roof hatch access
      loot_points: 2

    roof:
      name: "Rooftop"
      features:
        - Flat area with chimney cover
        - Connection to adjacent rooftops
      loot_points: 1

  architectural_details:
    exterior:
      - Wooden construction with stone foundation
      - Hanging sign "The Salty Siren" (swinging)
      - Barrel stacks flanking entrance
      - Lanterns on posts (×4)
      - Rope railings on upper floors

    interior:
      - Worn wooden floors (creaking audio)
      - Chandeliers (hanging, swaying)
      - Pirate memorabilia on walls
      - Candle-lit atmosphere
      - Nautical decorations
```

### 3.2 Clock Tower

```yaml
clock_tower:
  importance: "Primary landmark (visible map-wide)"
  loot_level: "Medium"
  player_capacity: 2

  dimensions:
    base_footprint: "8m × 8m"
    height: "35m"

  levels:
    base_level:
      height: "0-5m"
      features:
        - Stone archway entrance
        - Spiral staircase start
        - Storage room
      loot_points: 1

    mid_level:
      height: "15-20m"
      features:
        - Window platforms (×4 directions)
        - Continuation staircase
      loot_points: 1

    clock_level:
      height: "25-30m"
      features:
        - Large clock face (×4 directions)
        - Interior platform
        - External walkway
      loot_points: 1

    belfry:
      height: "30-35m"
      features:
        - Bell (interactive, creates sound)
        - Open-air platform
        - 360° visibility
        - Sniper position
      loot_points: 1

  architectural_details:
    - Stone construction with wooden accents
    - Weathered copper roof
    - Cracked clock faces (non-functional)
    - Exposed bell mechanism
    - Spiral staircase (wood, creaking)

  gameplay_notes:
    - Highest point in Port Plunder
    - Excellent sniper nest but exposed
    - Bell can be rung (audio alert to area)
    - Single access point (easy to trap)
```

### 3.3 Harbor Master's Office

```yaml
harbor_masters_office:
  importance: "Secondary landmark"
  loot_level: "High"
  player_capacity: 4

  dimensions:
    footprint: "18m × 12m"
    height: "10m (2 floors + observation)"

  floors:
    ground_floor:
      name: "Records Office"
      features:
        - Main entrance with double doors
        - Reception counter
        - Filing cabinets (cover)
        - Side exit to customs
      loot_points: 2

    second_floor:
      name: "Harbor Master's Quarters"
      features:
        - Large desk with maps
        - Balcony overlooking harbor
        - Telescope (interactive)
        - Private chambers
      loot_points: 3

    observation_deck:
      name: "Signal Tower"
      features:
        - Open platform
        - Signal flags
        - Lamp post
        - Excellent harbor view
      loot_points: 1

  architectural_details:
    - More formal than tavern (official building)
    - Whitewashed walls with dark timber frame
    - Colonial architecture influence
    - Official signage and seals
    - Flagpoles with torn flags
```

### 3.4 Fish Market

```yaml
fish_market:
  importance: "Open combat area"
  loot_level: "Medium"
  player_capacity: 6

  layout:
    main_building:
      footprint: "25m × 15m"
      height: "6m"
      features:
        - Open-air design (no walls, pillars)
        - Central cleaning station
        - Ice storage

    market_stalls:
      count: 8
      dimensions: "3m × 2m each"
      features:
        - Destructible canopy
        - Counter cover
        - Display goods (fish, seafood)

    crate_storage:
      area: "Adjacent open space"
      features:
        - Stacked crates (various heights)
        - Nets hanging
        - Cart props

  loot_points: 5

  architectural_details:
    - Weathered wood construction
    - Colorful awnings (faded)
    - Fish hanging on hooks
    - Scales and cutting boards
    - Pungent visual storytelling (flies, puddles)

  gameplay_notes:
    - Open area, less cover than other districts
    - Good for AoE abilities
    - Multiple escape routes
    - Stalls provide breakable cover
```

### 3.5 Warehouse Row

```yaml
warehouse_row:
  importance: "Dense cover combat zone"
  loot_level: "High"
  player_capacity: 8

  structures:
    large_warehouse_1:
      footprint: "30m × 20m"
      height: "8m"
      features:
        - Large sliding doors (×2)
        - Interior catwalks
        - Crate stacks (varied heights)
        - Office mezzanine
      loot_points: 4

    large_warehouse_2:
      footprint: "25m × 18m"
      height: "8m"
      features:
        - Cargo nets overhead
        - Loading platform
        - Hidden basement access
      loot_points: 3

    small_warehouses:
      count: 3
      footprint: "12m × 8m each"
      height: "5m"
      features:
        - Single room
        - Roof access ladder
      loot_points: 2 each

  connecting_elements:
    alleys:
      width: "2-4m"
      features:
        - Barrel/crate cover
        - Overhead walkways
        - Ambush points

    loading_dock:
      features:
        - Open platform
        - Crane (decorative)
        - Cart props

  loot_total: 13

  gameplay_notes:
    - Dense CQC environment
    - Multiple vertical options
    - Good for shotgun/melee builds
    - Easy to get lost/ambushed
```

### 3.6 Dock District

```yaml
dock_district:
  importance: "Water access and rotation point"
  loot_level: "Medium"
  player_capacity: 6

  structures:
    main_pier:
      dimensions: "80m × 8m"
      features:
        - Extends into water
        - Crate clusters (cover)
        - Mooring posts
        - End platform
      loot_points: 3

    dock_yard:
      footprint: "20m × 25m"
      features:
        - Ship repair scaffolding
        - Dry dock area
        - Tool storage shed
      loot_points: 3

    small_docks:
      count: 4
      dimensions: "15m × 4m each"
      features:
        - Individual boat moorings
        - Minimal cover
      loot_points: 1 each

    moored_boats:
      count: 6
      types:
        - Small rowboat (×3)
        - Fishing boat (×2)
        - Merchant vessel (×1)
      features:
        - Climbable
        - Interior access (larger boats)
        - Rocking animation
      loot_points: 2

  loot_total: 12

  gameplay_notes:
    - Linear engagement lines
    - Water escape option
    - Exposed positions
    - Good rotation to water edge
```

---

## 4. Traversal Systems

### 4.1 Rooftop Network

```yaml
rooftop_traversal:
  description: "Connected rooftop paths allowing elevated movement"

  connections:
    tavern_to_warehouse:
      method: "Wooden plank bridge"
      width: "1.5m"
      length: "6m"
      risk: "Exposed, can be shot"

    warehouse_to_warehouse:
      method: "Rope swing points"
      distance: "4m gaps"
      risk: "Requires jump/ability"

    harbor_office_balcony:
      method: "Awning jumping"
      drops: "3m between levels"
      risk: "Fall damage if missed"

    clock_tower_access:
      method: "Adjacent building ladder"
      requirement: "Must climb from warehouse roof"

  visual_indicators:
    - Worn plank paths
    - Rope coils at swing points
    - Landing mats (visual, no function)
    - Chimney positions for cover
```

### 4.2 Underground Tunnels

```yaml
smuggler_tunnels:
  description: "Hidden underground passages connecting key locations"

  entrances:
    warehouse_basement:
      location: "Large Warehouse 2"
      hidden: true
      visual_tell: "Trapdoor under crates"

    tavern_cellar:
      location: "Salty Siren basement"
      hidden: false
      visual_tell: "Staircase down"

    dock_yard:
      location: "Under dry dock"
      hidden: true
      visual_tell: "Grate in floor"

    beach_exit:
      location: "East of docks, in rocks"
      hidden: true
      visual_tell: "Cave opening"

  tunnel_specs:
    width: "2.5m"
    height: "2m"
    length_total: "~120m network"
    lighting: "Dim torches, pools of darkness"

  features:
    - Occasional loot caches (×3)
    - Water puddles (audio tell)
    - Branching paths
    - One-way gates (strategic)

  gameplay_notes:
    - Quick rotation underground
    - Audio amplified (footsteps echo)
    - CQC environment
    - Can be trapped/camped at exits
```

### 4.3 Entry Points

```yaml
entry_points:
  total: 8

  primary_entrances:
    north_road:
      description: "Main street entrance from map center"
      width: "Wide (8m)"
      cover: "Sparse"
      typical_use: "Main rotation path"

    south_docks:
      description: "Water approach via pier"
      width: "Variable"
      cover: "Low on pier"
      typical_use: "Water rotation, flanking"

  secondary_entrances:
    east_beach:
      description: "Beach approach past fish market"
      width: "Wide (open beach)"
      cover: "Sparse to none"
      typical_use: "Early drop, risky approach"

    west_path:
      description: "Cliff-side path from lighthouse area"
      width: "Narrow (3m)"
      cover: "Rocky outcrops"
      typical_use: "Flanking, late rotation"

  water_approaches:
    northeast_swim:
      description: "Swimming from coral reef area"
      visibility: "Low in water"
      typical_use: "Stealth approach"

    southeast_swim:
      description: "Direct swim to docks"
      visibility: "Low in water"
      typical_use: "Escape route"

    southwest_swim:
      description: "Around lighthouse peninsula"
      visibility: "Low in water"
      typical_use: "Long flank"

    boat_approach:
      description: "Using rowboat from shipwreck area"
      visibility: "High (above water)"
      typical_use: "Alternative to swimming"
```

---

## 5. Loot Distribution

### 5.1 Loot Summary

```yaml
loot_overview:
  total_chests: 20
  distribution:
    common: 8
    uncommon: 6
    rare: 4
    epic: 2

  ability_spawn_rate: 40%
  expected_abilities: 8
```

### 5.2 Loot Placement

```yaml
loot_placement:
  high_value_locations:
    tavern_third_floor:
      rarity: "Epic"
      chest_count: 1
      notes: "Reward for full building clear"

    harbor_master_office:
      rarity: "Epic"
      chest_count: 1
      notes: "On desk, exposed"

    warehouse_basement:
      rarity: "Rare"
      chest_count: 1
      notes: "Hidden room"

    clock_tower_belfry:
      rarity: "Rare"
      chest_count: 1
      notes: "High risk/reward"

  medium_value_locations:
    warehouse_interiors:
      rarity: "Uncommon-Rare"
      chest_count: 4

    tavern_ground_floor:
      rarity: "Uncommon"
      chest_count: 2

    fish_market:
      rarity: "Uncommon"
      chest_count: 2

  common_locations:
    dock_crates:
      rarity: "Common"
      chest_count: 3

    street_corners:
      rarity: "Common"
      chest_count: 3

    small_buildings:
      rarity: "Common"
      chest_count: 2

  ground_loot:
    spawn_points: 25
    items: "Abilities, health, gold"
```

### 5.3 Loot Spawn Points Map

```
Loot Density by Area:
═══════════════════════

    [Warehouse District]     [Clock Tower]      [Fish Market]
         ▓▓▓▓                    ▓               ▒▒▒▒
         ▓▓▓▓               (High risk)          ▒▒▒▒
       High Density                             Medium

    ┌─────────────── MAIN STREET (sparse) ────────────────┐

    [Salty Siren]          [Town Square]      [Harbor Master]
       ▓▓▓▓                    ░                   ▓▓
       ▓▓▓▓                 (Open)                 ▓▓
     Very High               Sparse               High

    ├────────────── HARBOR PROMENADE (low) ───────────────┤

    [Dock Yard]           [Main Pier]         [Small Docks]
        ▒▒                    ░░░                 ░░
        ▒▒                  (Linear)            (Edge)
      Medium                  Low                Sparse

Legend: ▓ = High  ▒ = Medium  ░ = Low/Sparse
```

---

## 6. Technical Specifications

### 6.1 Performance Budget

```yaml
performance_targets:
  device_tiers:
    low_end:
      target_fps: 30
      max_draw_calls: 100
      max_triangles: 150,000
      texture_budget: 64MB

    mid_range:
      target_fps: 45
      max_draw_calls: 150
      max_triangles: 300,000
      texture_budget: 128MB

    high_end:
      target_fps: 60
      max_draw_calls: 250
      max_triangles: 500,000
      texture_budget: 256MB
```

### 6.2 LOD System

```yaml
lod_configuration:
  buildings:
    LOD0:
      distance: "0-50m"
      detail: "Full detail, all props"
      interiors: "Visible"

    LOD1:
      distance: "50-100m"
      detail: "Reduced props, simplified geometry"
      interiors: "Hidden"

    LOD2:
      distance: "100-200m"
      detail: "Silhouette only"
      interiors: "Hidden"

    LOD3:
      distance: "200m+"
      detail: "Impostor billboards"
      interiors: "Hidden"

  props:
    LOD0_distance: "0-30m"
    LOD1_distance: "30-60m"
    culled: "60m+"

  special_elements:
    clock_tower:
      always_visible: true
      LOD0_distance: "0-100m"
      LOD1_distance: "100-300m"
      impostor_distance: "300m+"
```

### 6.3 Mesh Specifications

```yaml
mesh_budgets:
  buildings:
    large_building:
      LOD0_triangles: 8,000-12,000
      LOD1_triangles: 4,000-6,000
      LOD2_triangles: 1,000-2,000

    small_building:
      LOD0_triangles: 3,000-5,000
      LOD1_triangles: 1,500-2,500
      LOD2_triangles: 500-1,000

    clock_tower:
      LOD0_triangles: 15,000
      LOD1_triangles: 7,000
      LOD2_triangles: 2,000

  props:
    large_prop:
      triangles: 500-1,000
      examples: ["Cannon", "Large crate stack", "Cart"]

    medium_prop:
      triangles: 200-500
      examples: ["Barrel", "Table", "Chair group"]

    small_prop:
      triangles: 50-200
      examples: ["Lantern", "Bottle", "Rope coil"]

  total_static_budget:
    LOD0_total: 180,000 triangles
    LOD1_total: 80,000 triangles
    LOD2_total: 25,000 triangles
```

### 6.4 Texture Specifications

```yaml
texture_specifications:
  texture_atlases:
    buildings_atlas:
      resolution: "2048x2048"
      format: "ASTC 6x6"
      memory: "~8MB"
      contents:
        - Building exteriors
        - Roof materials
        - Window variations

    props_atlas:
      resolution: "1024x1024"
      format: "ASTC 6x6"
      memory: "~2MB"
      contents:
        - Barrels, crates, carts
        - Furniture
        - Street elements

    interior_atlas:
      resolution: "1024x1024"
      format: "ASTC 6x6"
      memory: "~2MB"
      contents:
        - Floor textures
        - Wall details
        - Interior props

  unique_textures:
    tavern_sign:
      resolution: "256x256"
      purpose: "Readable signage"

    clock_faces:
      resolution: "512x512"
      purpose: "Landmark detail"

  material_types:
    wood_weathered:
      shader: "Plunderstorm/Standard"
      properties: [albedo, normal, roughness]

    stone_brick:
      shader: "Plunderstorm/Standard"
      properties: [albedo, normal, roughness]

    metal_rusted:
      shader: "Plunderstorm/Standard"
      properties: [albedo, normal, metallic, roughness]

    fabric_torn:
      shader: "Plunderstorm/TwoSided"
      properties: [albedo, normal, opacity]
```

### 6.5 Collision Configuration

```yaml
collision_setup:
  building_collision:
    type: "Simplified mesh"
    detail_level: "~10% of visual mesh"
    convex_decomposition: true

  prop_collision:
    barrel:
      type: "Cylinder primitive"
    crate:
      type: "Box primitive"
    table:
      type: "Compound (box + legs)"
    furniture:
      type: "Simplified convex hull"

  terrain_collision:
    resolution: "0.5m grid"
    type: "Mesh collider"

  trigger_volumes:
    interior_zones:
      purpose: "Audio/lighting transition"
    loot_zones:
      purpose: "Spawn point registration"
    entry_zones:
      purpose: "Player tracking"
```

---

## 7. Art Direction

### 7.1 Visual Style

```yaml
visual_style:
  architecture:
    era: "17th-18th century Caribbean Colonial"
    condition: "Weathered, well-used, lived-in"
    materials:
      primary: "Weathered wood planks"
      secondary: "Stone foundations"
      accent: "Rusted metal, rope, canvas"

  color_palette:
    buildings:
      primary: "#8B7355"      # Weathered wood brown
      secondary: "#6B4423"    # Dark wood
      accent: "#C4A35A"       # Brass/gold trim

    roofs:
      primary: "#6B4423"      # Dark tile/shingle
      secondary: "#A0522D"    # Terra cotta

    details:
      fabric: "#8B0000"       # Faded red canvas
      metal: "#4A4A4A"        # Rusted iron
      rope: "#D2B48C"         # Tan hemp

  lighting:
    time_of_day: "Late afternoon (4-5 PM)"
    sun_angle: "30-40 degrees"
    shadows: "Long, dramatic"
    bounce_light: "Warm orange from wood surfaces"
```

### 7.2 Environmental Storytelling

```yaml
storytelling_elements:
  recent_activity:
    - Abandoned card game on tavern table
    - Half-eaten meals in fish market
    - Spilled cargo crates
    - Burning torches (time frozen)

  pirate_culture:
    - Wanted posters on walls
    - Graffiti and symbols
    - Tribute shrine to Blackstorm
    - Stolen goods displayed

  curse_effects:
    - Slight purple haze in shadows
    - Frozen flag mid-flutter
    - Stopped clock faces
    - Spectral wisps (subtle VFX)

  prop_vignettes:
    tavern_entrance:
      props: ["Sleeping pirate (statue)", "Spilled mug", "Dog"]
      story: "Patron passed out during curse"

    dock_yard:
      props: ["Half-repaired hull", "Scattered tools", "Plans"]
      story: "Work interrupted mid-task"

    clock_tower:
      props: ["Broken mechanism", "Toolbox", "Notes"]
      story: "Clocksmith trapped inside"
```

### 7.3 Prop List

```yaml
required_props:
  structural:
    - Barrel (3 variants: new, old, broken)
    - Crate (3 sizes: small, medium, large)
    - Cart (2 variants: handcart, horse cart)
    - Ladder (wooden, rope)
    - Plank bridge
    - Rope swing

  furniture:
    - Table (dining, desk, counter)
    - Chair (basic, captain's)
    - Bed (bunks, single)
    - Chandelier (candle)
    - Shelf (empty, stocked)
    - Cabinet (filing, storage)

  maritime:
    - Anchor
    - Cannon (decorative)
    - Fishing net
    - Buoy
    - Oar
    - Rope coils
    - Sail cloth

  market:
    - Fish display
    - Market stall frame
    - Awning/canopy
    - Crate with goods
    - Scale
    - Ice block

  decorative:
    - Lantern (post, hanging, wall)
    - Sign (shop, street, wanted poster)
    - Flag (port flag, pirate flag)
    - Banner
    - Bottle cluster
    - Coin pile

  interactive:
    - Treasure chest (all tiers)
    - Door (swinging, sliding)
    - Bell (clock tower)
    - Telescope (fixed)
```

---

## 8. Audio Design

### 8.1 Ambient Audio

```yaml
ambient_audio:
  base_layer:
    sound: "port_ambient_base"
    description: "Harbor atmosphere - waves, distant seagulls, wood creaking"
    volume: 0.4

  district_variations:
    warehouse:
      addition: "Occasional crate settling, rat scurrying"
      volume_mod: -0.1

    tavern:
      addition: "Ghostly music echo, glass clink"
      volume_mod: +0.05

    docks:
      addition: "Water lapping, boat rocking, rope tension"
      volume_mod: 0

    market:
      addition: "Flies buzzing, wind through awnings"
      volume_mod: 0

  interior_audio:
    tavern_interior:
      reverb: "Medium room"
      ambient: "Muffled exterior + creaking floors"

    warehouse_interior:
      reverb: "Large warehouse"
      ambient: "Echoing space, occasional drip"

    tunnel:
      reverb: "Tunnel/cave"
      ambient: "Dripping water, distant rumble"
```

### 8.2 Interactive Audio

```yaml
interactive_sounds:
  footsteps:
    wood_floor:
      sound: "footstep_wood_*"
      variations: 4

    stone:
      sound: "footstep_stone_*"
      variations: 4

    dock_planks:
      sound: "footstep_dock_*"
      variations: 4
      notes: "Hollow, creaking"

    water_shallow:
      sound: "footstep_splash_*"
      variations: 4

  doors:
    wood_swing:
      open: "door_wood_open"
      close: "door_wood_close"
      creak: "door_creak_*"

    sliding:
      open: "door_slide_open"
      close: "door_slide_close"

  props:
    bell_ring:
      sound: "clock_tower_bell"
      range: 200m
      notes: "Alert sound, player-triggered"

    barrel_destroy:
      sound: "barrel_break"
      debris: "wood_debris_*"

    glass_break:
      sound: "glass_shatter_*"
      variations: 3
```

### 8.3 Combat Audio Adjustments

```yaml
combat_audio:
  interior_combat:
    reverb_preset: "Room medium"
    volume_boost: 1.1

  exterior_combat:
    reverb_preset: "Outdoor open"
    volume_boost: 1.0

  tunnel_combat:
    reverb_preset: "Cave tunnel"
    volume_boost: 1.2
    notes: "Extra echo, intimidating"
```

---

## 9. Gameplay Considerations

### 9.1 Engagement Distances

```yaml
engagement_zones:
  close_quarters:
    distance: "0-10m"
    locations: ["Warehouse alleys", "Tunnels", "Building interiors"]
    dominant_abilities: ["Cutlass Slash", "Smoke Bomb", "Barrel Roll"]

  mid_range:
    distance: "10-30m"
    locations: ["Main street", "Town square", "Dock pier"]
    dominant_abilities: ["Fireball", "Lightning Strike", "Grappling Hook"]

  long_range:
    distance: "30-50m"
    locations: ["Rooftops", "Clock tower", "Open beach"]
    dominant_abilities: ["Cannonball", "Basic attack"]
```

### 9.2 Cover Analysis

```yaml
cover_map:
  hard_cover:
    examples: ["Stone walls", "Building corners", "Large crates"]
    protection: "Full block"
    locations: ["All districts"]

  soft_cover:
    examples: ["Barrels", "Market stalls", "Wooden fences"]
    protection: "Breakable, temporary"
    locations: ["Market", "Docks", "Streets"]

  elevation_cover:
    examples: ["Rooftops", "Balconies", "Clock tower"]
    protection: "Vertical advantage"
    locations: ["All buildings"]

  concealment:
    examples: ["Awnings", "Shadows", "Underwater"]
    protection: "Visual only"
    locations: ["Market", "Tunnels", "Docks"]
```

### 9.3 Rotation Strategy

```yaml
rotation_options:
  from_north:
    into_port:
      route: "Main street"
      cover: "Moderate"
      time: "15 seconds"

  from_south:
    into_port:
      route: "Dock approach"
      cover: "Low"
      time: "20 seconds"

  within_port:
    warehouse_to_tavern:
      route: "Rooftops or alley"
      time: "10 seconds"

    tavern_to_docks:
      route: "Main street or tunnel"
      time: "12 seconds"

  escape_routes:
    water_escape:
      direction: "South into ocean"
      risk: "Exposed while swimming"

    tunnel_escape:
      direction: "East beach exit"
      risk: "Can be camped"
```

---

## 10. Implementation Checklist

### 10.1 Art Production Checklist

```yaml
art_checklist:
  greybox:
    - [ ] All building volumes blocked out
    - [ ] Street layout finalized
    - [ ] Collision working correctly
    - [ ] Entry points tested
    - [ ] Vertical gameplay validated

  structural:
    - [ ] Tavern exterior complete
    - [ ] Tavern interior complete
    - [ ] Clock Tower complete
    - [ ] Harbor Master building complete
    - [ ] Warehouses complete
    - [ ] Fish Market complete
    - [ ] Dock structures complete

  props:
    - [ ] Street props placed
    - [ ] Interior props placed
    - [ ] Cover objects positioned
    - [ ] Destructibles configured

  textures:
    - [ ] Building atlas complete
    - [ ] Props atlas complete
    - [ ] Interior atlas complete
    - [ ] Unique textures complete

  lighting:
    - [ ] Baked lightmaps generated
    - [ ] Light probes placed
    - [ ] Interior lighting tuned
    - [ ] Ambient occlusion applied

  effects:
    - [ ] Ambient particles (dust, etc.)
    - [ ] Water effects at docks
    - [ ] Flag/cloth animation
    - [ ] Torch fire effects

  optimization:
    - [ ] LODs generated and tested
    - [ ] Occlusion culling configured
    - [ ] Draw calls within budget
    - [ ] Memory within budget
    - [ ] Performance validated on target devices
```

### 10.2 Gameplay Integration Checklist

```yaml
gameplay_checklist:
  spawn_points:
    - [ ] 25 ground loot points placed
    - [ ] 20 chest spawn points placed
    - [ ] Spawn point tiers assigned

  navigation:
    - [ ] NavMesh generated
    - [ ] Bot pathing tested
    - [ ] Jump points marked
    - [ ] Ladder interaction working

  audio:
    - [ ] Ambient zones configured
    - [ ] Audio triggers placed
    - [ ] Reverb zones set up
    - [ ] Interactive sounds working

  interactables:
    - [ ] Doors functional
    - [ ] Bell interactive
    - [ ] Destructibles working
    - [ ] Loot spawning correctly
```

---

## 11. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| 3D environment matches style guide | Pirate colonial aesthetic | Art Director review |
| Buildings enterable (key structures) | Tavern, Harbor Office, Warehouses | Walkthrough test |
| Props placed | All prop categories represented | Visual audit |
| Lighting setup (baked for mobile) | Lightmaps, probes configured | Performance test |
| Collision matches visual | No walk-through, no invisible walls | QA collision test |
| Loot spawn points integrated | 20 chest points, 25 ground loot | Gameplay test |
| Performance acceptable | 45+ FPS on mid-range device | Performance profiling |
| Ambient audio zones | District-specific audio | Audio walkthrough |

---

## 12. Deliverables

### 12.1 Asset List

```yaml
deliverables:
  scenes:
    - "Port_Plunder_Main.unity"
    - "Port_Plunder_Interior_Tavern.unity"
    - "Port_Plunder_Interior_Warehouse.unity"

  prefabs:
    buildings:
      - "PRF_Building_Tavern.prefab"
      - "PRF_Building_ClockTower.prefab"
      - "PRF_Building_HarborOffice.prefab"
      - "PRF_Building_Warehouse_Large.prefab"
      - "PRF_Building_Warehouse_Small.prefab"
      - "PRF_Building_FishMarket.prefab"

    props:
      - "PRF_Props_Street_Kit.prefab"
      - "PRF_Props_Maritime_Kit.prefab"
      - "PRF_Props_Market_Kit.prefab"
      - "PRF_Props_Furniture_Kit.prefab"

    interactive:
      - "PRF_Door_Wood_Swing.prefab"
      - "PRF_Door_Warehouse_Slide.prefab"
      - "PRF_Bell_ClockTower.prefab"
      - "PRF_Destructible_Barrel.prefab"
      - "PRF_Destructible_Crate.prefab"

  textures:
    - "T_Port_Buildings_Atlas_Albedo.png"
    - "T_Port_Buildings_Atlas_Normal.png"
    - "T_Port_Buildings_Atlas_Mask.png"
    - "T_Port_Props_Atlas_Albedo.png"
    - "T_Port_Props_Atlas_Normal.png"
    - "T_Port_Interiors_Atlas_Albedo.png"

  materials:
    - "M_Port_Wood_Weathered.mat"
    - "M_Port_Stone_Brick.mat"
    - "M_Port_Metal_Rusted.mat"
    - "M_Port_Fabric_Torn.mat"
    - "M_Port_Water_Surface.mat"

  audio:
    - "AMB_Port_Base.wav"
    - "AMB_Port_Warehouse.wav"
    - "AMB_Port_Tavern.wav"
    - "AMB_Port_Docks.wav"
    - "SFX_Port_Bell.wav"
    - "SFX_Port_Door_*.wav"

  lightmaps:
    - "LM_Port_Plunder_0-*.exr"
```

---

## 13. Appendices

### 13.1 Reference Images

```yaml
visual_references:
  architecture:
    - "Port Royal, Jamaica historical photos"
    - "Pirates of the Caribbean films (Port Royal)"
    - "Sea of Thieves outpost concept art"
    - "Assassin's Creed Black Flag Havana"

  mood:
    - "Caribbean sunset harbor paintings"
    - "Golden hour coastal photography"
    - "Pirate fantasy concept art collections"

  props:
    - "Maritime museum artifact photos"
    - "Historical ship equipment references"
    - "Colonial Caribbean furniture guides"
```

### 13.2 Related Documents

- [MAP_DESIGN_DOCUMENT.md](MAP_DESIGN_DOCUMENT.md) - Overall map layout
- [MAP_GREYBOX_BLOCKOUT_SPEC.md](MAP_GREYBOX_BLOCKOUT_SPEC.md) - Greybox specifications
- [VISUAL_STYLE_GUIDE.md](VISUAL_STYLE_GUIDE.md) - Art direction
- [TECHNICAL_ART_SPECIFICATIONS.md](TECHNICAL_ART_SPECIFICATIONS.md) - Performance targets
- [TREASURE_CHEST_PROPS_SPEC.md](TREASURE_CHEST_PROPS_SPEC.md) - Loot prop details

### 13.3 Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-01 | Initial specification |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Level Designer | _____________ | _____________ | ___/___/___ |
| Environment Artist | _____________ | _____________ | ___/___/___ |
| Art Director | _____________ | _____________ | ___/___/___ |
| Technical Artist | _____________ | _____________ | ___/___/___ |

---

*This document serves as the complete specification for Port Plunder POI development. It should be used as the quality template for all subsequent named location development.*
