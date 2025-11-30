# Map Design Document - Plunderstorm Mobile

**Document ID:** ART-007
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Map Overview](#2-map-overview)
3. [Island Layout](#3-island-layout)
4. [Named Locations (POIs)](#4-named-locations-pois)
5. [Terrain Zones](#5-terrain-zones)
6. [Loot Distribution](#6-loot-distribution)
7. [Monster Camps](#7-monster-camps)
8. [Storm System Integration](#8-storm-system-integration)
9. [Navigation and Landmarks](#9-navigation-and-landmarks)
10. [Gameplay Flow](#10-gameplay-flow)
11. [Technical Specifications](#11-technical-specifications)
12. [Art Direction](#12-art-direction)
13. [Audio Zones](#13-audio-zones)
14. [Appendices](#14-appendices)

---

## 1. Executive Summary

### 1.1 Map Identity

**Name:** Cursed Cove Island

**Theme:** A legendary pirate island shrouded in mystery, featuring diverse biomes from tropical beaches to volcanic mountains, dotted with pirate settlements, ancient ruins, and supernatural locations.

**Narrative Hook:** Once the haven of the infamous Pirate King Blackstorm, this island was cursed when he opened a forbidden treasure. Now, adventurers battle for the legendary loot while the storm closes in.

### 1.2 Design Goals

| Goal | Target | Rationale |
|------|--------|-----------|
| Match Duration | 10-15 minutes | Mobile session-friendly |
| Player Count | 60 players | Optimal for mobile BR |
| POI Count | 10 named locations | Memorable, learnable |
| Engagement Pacing | Action every 30 seconds | Maintain excitement |
| Skill Expression | High | Multiple routes, positioning matters |

### 1.3 Key Features

1. **Diverse Biomes**: Beach, Jungle, Mountain, Volcanic, Ruins
2. **Vertical Gameplay**: Multi-level structures, elevation changes
3. **Risk/Reward Zones**: Hot zones with better loot, higher danger
4. **Environmental Storytelling**: Pirate history throughout
5. **Strategic Depth**: Multiple approach routes to every location

---

## 2. Map Overview

### 2.1 Island Dimensions

| Attribute | Value | Notes |
|-----------|-------|-------|
| Total Area | 1.5 km × 1.5 km | ~2.25 km² playable |
| Playable Diameter | ~1,400 meters | Circular playable zone |
| Maximum Elevation | 120 meters | Volcanic peak |
| Sea Level | 0 meters | Water boundary |
| Water Depth | -5 to -15 meters | Shallow near shore |

### 2.2 Shape and Orientation

The island is roughly **circular with irregular coastline**, featuring:
- **North**: Volcanic mountains and ancient ruins
- **East**: Dense jungle and hidden caves
- **South**: Tropical beaches and coastal town
- **West**: Rocky cliffs and pirate fortress
- **Center**: Cursed temple and central lake

```
                    N
                    |
        [Volcano]  [Ruins]
             \      /
    [Fortress]---[Temple]---[Jungle]
             \      |      /
        [Cliffs] [Lake] [Caves]
             \      |      /
    [Lighthouse]--[Port]--[Beach]
                    |
                    S
```

### 2.3 Terrain Breakdown

| Terrain Type | % of Map | Character |
|--------------|----------|-----------|
| Flat/Rolling | 35% | Beaches, plains |
| Forested | 25% | Jungle, palm groves |
| Elevated | 20% | Hills, mountains |
| Urban | 15% | Towns, structures |
| Water | 5% | Lake, streams |

---

## 3. Island Layout

### 3.1 Sector Grid

The map is divided into a **6x6 grid** (A1-F6) for callouts and navigation.

```
    A       B       C       D       E       F
  +-------+-------+-------+-------+-------+-------+
1 | Water | Skull | Dead  | Ruin  | Water | Water |
  |       | Rock  | Man's | Hills |       |       |
  +-------+-------+-------+-------+-------+-------+
2 | Storm | Vol-  | Vol-  | Ancient| Dark | Water |
  | Cliffs| cano  | cano  | Temple| Jungle|       |
  +-------+-------+-------+-------+-------+-------+
3 | Fort  | Crater| Cursed| Cursed| Jungle| Parrot|
  | Black | Lake  | Temple| Grove | Trails| Perch |
  +-------+-------+-------+-------+-------+-------+
4 | Light-| Golden| Cross-| Witch | Echo  | Siren |
  | house | Sands | roads | Hollow| Caves | Rocks |
  +-------+-------+-------+-------+-------+-------+
5 | Ship- | Port  | Market| Palm  | Hidden| Coral |
  | wreck | Plunder| Square| Beach | Cove | Reef  |
  +-------+-------+-------+-------+-------+-------+
6 | Water | Dock  | Water | Water | Water | Water |
  |       | Yard  |       |       |       |       |
  +-------+-------+-------+-------+-------+-------+
```

### 3.2 Elevation Map

```
Legend:
  0-20m   [░] - Beaches, lowlands
  20-50m  [▒] - Hills, jungle floor
  50-80m  [▓] - Mountains, cliffs
  80-120m [█] - Volcanic peak

    A   B   C   D   E   F
  +---+---+---+---+---+---+
1 | ~ | ▓ | ▓ | ▒ | ~ | ~ |
  +---+---+---+---+---+---+
2 | ▓ | █ | █ | ▒ | ▒ | ~ |
  +---+---+---+---+---+---+
3 | ▓ | ▓ | ▒ | ▒ | ▒ | ▒ |
  +---+---+---+---+---+---+
4 | ▒ | ░ | ░ | ▒ | ▒ | ░ |
  +---+---+---+---+---+---+
5 | ░ | ░ | ░ | ░ | ░ | ░ |
  +---+---+---+---+---+---+
6 | ~ | ░ | ~ | ~ | ~ | ~ |
  +---+---+---+---+---+---+
```

### 3.3 Major Routes

**Primary Routes (Wide, Fast Travel):**
1. **Coastal Path**: Circles the island perimeter
2. **Central Road**: Port Plunder → Crossroads → Cursed Temple
3. **Mountain Pass**: Fort Blackstorm → Volcano → Ancient Temple

**Secondary Routes (Medium):**
- Jungle trails connecting east POIs
- Cave systems with shortcuts
- River/stream paths

**Tertiary Routes (Risky, Shortcuts):**
- Cliff climbing
- Swimming across lake
- Underground tunnels

---

## 4. Named Locations (POIs)

### 4.1 POI Summary

| # | Name | Grid | Tier | Theme | Size |
|---|------|------|------|-------|------|
| 1 | Port Plunder | B5-C5 | S | Pirate Town | Large |
| 2 | Fort Blackstorm | A3 | A | Military Fortress | Large |
| 3 | Cursed Temple | C3-D3 | S | Ancient Ruins | Medium |
| 4 | Volcano Summit | B2-C2 | A | Volcanic | Medium |
| 5 | Ancient Temple | D2 | B | Ruins | Medium |
| 6 | Witch's Hollow | D4 | B | Swamp/Magic | Small |
| 7 | Echo Caves | E4 | B | Underground | Medium |
| 8 | Lighthouse Point | A4-A5 | C | Coastal | Small |
| 9 | Shipwreck Shores | A5 | B | Beach/Wreck | Small |
| 10 | Parrot Perch | F3 | C | Treehouse | Small |

**Tier Legend:**
- **S (Hot)**: High loot, high traffic, 15-20 player capacity
- **A (Warm)**: Good loot, moderate traffic, 10-15 player capacity
- **B (Medium)**: Decent loot, varied traffic, 6-10 player capacity
- **C (Cool)**: Basic loot, low traffic, 4-6 player capacity

### 4.2 Detailed POI Descriptions

---

#### POI 1: Port Plunder (S-Tier)

**Location:** B5-C5 (South-Central)
**Theme:** Bustling pirate port town
**Player Capacity:** 20 players

**Description:**
The largest settlement on the island, Port Plunder is a sprawling pirate haven with docks, taverns, markets, and warehouses. The vertical design includes rooftops for sniping and back alleys for ambushes.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| The Salty Siren Tavern | Multi-floor interior | High |
| Harbor Master's Office | Elevated overview | High |
| Fish Market | Open stalls | Medium |
| Warehouse Row | Cover-heavy | High |
| Dock Yard | Ship access | Medium |
| Clock Tower | Sniper nest | Medium |

**Layout Features:**
- Main street with cover objects (carts, barrels)
- Rooftop traversal system (planks, ropes)
- Underground smuggler tunnels
- Multiple dock piers extending into water

**Entry Points:** 8 (North road, East beach, South docks, West path, 4 water approaches)

**Gameplay Notes:**
- High-risk, high-reward drop location
- Expect early-game chaos
- Central position makes it rotational hub
- Watch the Clock Tower for snipers

---

#### POI 2: Fort Blackstorm (A-Tier)

**Location:** A3 (West)
**Theme:** Imposing military fortress
**Player Capacity:** 15 players

**Description:**
A massive stone fortress built into the western cliffs, once the seat of power for the Pirate King. Features defensive walls, cannon emplacements, and a central keep.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Main Keep | 3-story tower | Epic |
| Armory | Weapons cache | High |
| Barracks | Multiple rooms | Medium |
| Outer Walls | Defensive positions | Low |
| Dungeon | Underground cells | High |
| War Room | Strategy table | High |

**Layout Features:**
- Multiple elevation levels (sea level to 60m)
- Defensive chokepoints (gates, stairs)
- Hidden passages in walls
- Cannon positions with sightlines

**Entry Points:** 5 (Main gate, Sea gate, 2 wall breaches, Secret tunnel)

**Gameplay Notes:**
- Defensible but can be surrounded
- Loot concentrated in keep
- Rotation west is cliff-limited
- Strong for duo/squad holds

---

#### POI 3: Cursed Temple (S-Tier)

**Location:** C3-D3 (Central)
**Theme:** Supernatural ancient temple
**Player Capacity:** 18 players

**Description:**
The heart of the island's curse, an ancient temple complex where the Pirate King opened the forbidden treasure. Features supernatural elements like floating platforms and glowing artifacts.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Main Sanctum | Central chamber | Epic |
| Outer Courtyard | Open area | Medium |
| Priest Quarters | Side buildings | High |
| Treasure Vault | Underground | Epic |
| Ritual Platforms | Floating stones | High |
| Spirit Well | Central landmark | N/A |

**Layout Features:**
- Vertical floating platforms (ability traversal)
- Glowing curse effects (visual noise/cover)
- Underground vault with high loot
- Open courtyard for large fights

**Entry Points:** 6 (4 compass directions, 2 underground)

**Gameplay Notes:**
- Geographic center = rotational importance
- Expect constant action mid-game
- Floating platforms reward mobility abilities
- High visibility (curse glow)

---

#### POI 4: Volcano Summit (A-Tier)

**Location:** B2-C2 (North-Central)
**Theme:** Active volcanic caldera
**Player Capacity:** 12 players

**Description:**
The highest point on the island, featuring an active volcanic crater, sulfur vents, and molten rock flows. Dangerous but rewarding.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Crater Rim | Elevated path | Medium |
| Magma Forge | Smithing area | Epic |
| Observatory | Peak structure | High |
| Sulfur Mines | Cave entrance | High |
| Hot Springs | Rest area | Low |

**Layout Features:**
- Extreme elevation advantage
- Environmental hazards (steam vents, unstable ground)
- Limited cover (rocky terrain)
- Visible from most of map

**Entry Points:** 4 (Mountain paths from each direction)

**Gameplay Notes:**
- High ground advantage but exposed
- Rotations require descent time
- Hazards deal minor damage
- Storm often forces evacuation

---

#### POI 5: Ancient Temple (B-Tier)

**Location:** D2 (Northeast)
**Theme:** Overgrown jungle ruins
**Player Capacity:** 10 players

**Description:**
Pre-pirate civilization ruins, partially reclaimed by jungle. Features crumbling architecture, hidden chambers, and ancient traps.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Grand Hall | Main ruin | High |
| Idol Chamber | Trap room | Epic |
| Collapsed Tower | Vertical cover | Medium |
| Meditation Garden | Open area | Low |
| Hidden Library | Secret room | High |

**Layout Features:**
- Dense vegetation and broken walls
- Trap mechanics (pressure plates, dart launchers)
- Multiple elevation layers (ruins stacked)
- Tight sightlines

**Entry Points:** 5 (Multiple jungle paths)

**Gameplay Notes:**
- Traps can damage enemies
- Good for ambushes
- Less traveled than central POIs
- Strong rotation to Cursed Temple

---

#### POI 6: Witch's Hollow (B-Tier)

**Location:** D4 (East-Central)
**Theme:** Spooky swamp with magic hut
**Player Capacity:** 8 players

**Description:**
A fog-shrouded swamp said to be home to a sea witch. Features murky water, twisted trees, and a central witch's hut on stilts.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Witch's Hut | Elevated structure | High |
| Potion Garden | Ingredient area | Medium |
| Ritual Circle | Open clearing | Medium |
| Sunken Shrine | Underwater access | High |
| Dead Tree | Landmark | N/A |

**Layout Features:**
- Reduced visibility (fog/mist effect)
- Water movement slowdown
- Stilted platforms above water
- Tight pathways between trees

**Entry Points:** 4 (Paths from surrounding areas)

**Gameplay Notes:**
- Fog provides cover but obscures threats
- Water slows movement
- Witch's Hut is defensible
- Good for stealth-oriented players

---

#### POI 7: Echo Caves (B-Tier)

**Location:** E4 (East)
**Theme:** Underground cave network
**Player Capacity:** 10 players

**Description:**
An extensive cave system with bioluminescent plants, underground lakes, and multiple entrances. The caves echo, providing audio cues.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Main Cavern | Central hub | High |
| Crystal Chamber | Glowing crystals | High |
| Underground Lake | Water traversal | Medium |
| Bat Roost | Vertical shaft | Medium |
| Exit Tunnels | Multiple exits | Low |

**Layout Features:**
- Indoor environment (no storm damage inside briefly)
- Limited sightlines (curved tunnels)
- Water sections requiring swimming
- Bioluminescence provides ambient light

**Entry Points:** 6 (Multiple cave mouths across east side)

**Gameplay Notes:**
- Audio cues amplified (footsteps echo)
- Brief storm protection in deep sections
- Tight CQC encounters
- Good rotation shortcut

---

#### POI 8: Lighthouse Point (C-Tier)

**Location:** A4-A5 (West Coast)
**Theme:** Coastal lighthouse station
**Player Capacity:** 6 players

**Description:**
A working lighthouse with keeper's quarters and dock. Provides excellent sightlines but limited escape routes.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Lighthouse Tower | 4-story vertical | High |
| Keeper's House | 2-story building | Medium |
| Small Dock | Water access | Low |
| Storage Shed | Single room | Low |

**Layout Features:**
- Extreme vertical (lighthouse)
- Exposed coastal position
- Limited cover outside buildings
- Single main path in

**Entry Points:** 3 (Main road, Dock, Cliff path)

**Gameplay Notes:**
- Lighthouse top has best western sightlines
- Easily surrounded
- Early rotation required (storm pushes east)
- Good for solo players

---

#### POI 9: Shipwreck Shores (B-Tier)

**Location:** A5 (Southwest)
**Theme:** Beach with beached pirate ships
**Player Capacity:** 8 players

**Description:**
A stretch of beach littered with wrecked ships of various sizes, creating a unique combat environment with ship hulls as cover.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| The Leviathan (Large Wreck) | Multi-deck ship | High |
| Medium Wrecks (×3) | Ship hulls | Medium |
| Small Boats | Scattered cover | Low |
| Beach Camp | Survivor tents | Medium |
| Tide Pools | Environmental | N/A |

**Layout Features:**
- Ship interiors create indoor fights
- Sand reduces movement speed slightly
- Tide pools have visual distinction
- Mixed elevation (deck levels)

**Entry Points:** 4 (Beach from multiple directions)

**Gameplay Notes:**
- Leviathan is the loot focus
- Open beach is dangerous
- Ships provide excellent cover
- Good early loot, then rotate

---

#### POI 10: Parrot Perch (C-Tier)

**Location:** F3 (East Edge)
**Theme:** Treetop village/lookout
**Player Capacity:** 5 players

**Description:**
A series of treehouses and platforms connected by rope bridges, used by lookouts and scouts. High elevation in the jungle canopy.

**Key Structures:**
| Structure | Function | Loot Level |
|-----------|----------|------------|
| Main Treehouse | Central platform | Medium |
| Lookout Posts (×3) | Small platforms | Low |
| Rope Bridges | Connections | N/A |
| Supply Cache | Hidden box | High |
| Bird Aviary | Landmark | N/A |

**Layout Features:**
- All elevated (8-15m off ground)
- Rope bridges are exposed
- Fall damage possible
- Jungle canopy obscures from distance

**Entry Points:** 4 (Rope ladders, tree climbing)

**Gameplay Notes:**
- Small but high-value for position
- Rope bridges are risky
- Good sniper/scout position
- Far from center, requires rotation

---

## 5. Terrain Zones

### 5.1 Biome Overview

| Biome | Grid Regions | Character | Gameplay Effect |
|-------|--------------|-----------|-----------------|
| Beach | Row 5-6, A4 | Sandy, open | Slower movement in sand |
| Jungle | D-E, Rows 2-4 | Dense, vertical | Reduced visibility |
| Mountain | A-C, Rows 1-3 | Rocky, steep | Elevation advantages |
| Volcanic | B-C, Rows 1-2 | Hazardous, barren | Environmental damage |
| Swamp | D4 area | Wet, foggy | Movement penalty |
| Plains | Central areas | Open, rolling | Clear sightlines |

### 5.2 Beach Zone

**Coverage:** ~20% of map
**Character:** White sand beaches, palm trees, coral formations

**Features:**
- Movement speed -10% on sand
- Clear long-range sightlines
- Limited natural cover
- Palm trees destructible
- Water edge provides escape route

**Flora:**
- Palm trees (tall, thin cover)
- Beach grass (no cover)
- Coconut clusters (props)
- Driftwood (low cover)

### 5.3 Jungle Zone

**Coverage:** ~25% of map
**Character:** Dense tropical vegetation, multiple canopy layers

**Features:**
- Reduced visibility (50m effective range)
- Vertical gameplay (vines, trees)
- Ambient sound masks footsteps partially
- Many paths through undergrowth

**Flora:**
- Large tropical trees (hard cover)
- Ferns and bushes (soft cover, destroyable)
- Vines (climbable in designated spots)
- Flowers (color variation)

### 5.4 Mountain Zone

**Coverage:** ~15% of map
**Character:** Rocky outcrops, steep terrain, mountain paths

**Features:**
- Significant elevation changes
- Limited paths (chokepoints)
- Rocky cover abundant
- Fall damage possible
- Strategic high ground

**Flora:**
- Sparse vegetation
- Mountain flowers
- Rocky outcrops (hard cover)
- Cliff faces (impassable)

### 5.5 Volcanic Zone

**Coverage:** ~10% of map
**Character:** Active volcanic terrain, lava flows, steam vents

**Features:**
- Environmental hazards (5 damage/second in hazard zones)
- Steam vents obscure vision periodically
- Highest elevation on map
- Limited cover (open rock)
- Dramatic visual effects

**Hazards:**
| Hazard | Damage | Duration | Tell |
|--------|--------|----------|------|
| Lava Pool | 10/sec | Persistent | Orange glow |
| Steam Vent | 5/burst | 3 sec cycle | Hissing sound |
| Unstable Ground | 15 | One-time | Cracks visible |

### 5.6 Swamp Zone

**Coverage:** ~5% of map
**Character:** Murky water, twisted trees, fog

**Features:**
- Movement speed -20% in water
- Fog reduces visibility to 30m
- Water hides player profile
- Unique audio (bubbling, croaking)

**Flora:**
- Mangrove-style trees
- Spanish moss (visual)
- Lily pads (surface cover)
- Reeds (soft cover)

---

## 6. Loot Distribution

### 6.1 Loot Philosophy

- **Hot Zones**: High risk, high reward - S-tier POIs
- **Medium Zones**: Balanced risk/reward - A/B-tier POIs
- **Cold Zones**: Low risk, basic loot - C-tier POIs, wilderness
- **Wilderness**: Scattered ground loot, low density

### 6.2 Loot Density Map

```
Legend:
  [H] = Hot (15+ chests)
  [M] = Medium (8-14 chests)
  [L] = Low (3-7 chests)
  [S] = Sparse (1-2 chests)

    A   B   C   D   E   F
  +---+---+---+---+---+---+
1 | ~ | S | S | M | ~ | ~ |
  +---+---+---+---+---+---+
2 | M | M | M | M | L | ~ |
  +---+---+---+---+---+---+
3 | H | M | H | H | L | L |
  +---+---+---+---+---+---+
4 | L | S | L | M | M | S |
  +---+---+---+---+---+---+
5 | M | H | M | L | L | S |
  +---+---+---+---+---+---+
6 | ~ | M | ~ | ~ | ~ | ~ |
  +---+---+---+---+---+---+
```

### 6.3 Chest Distribution by POI

| POI | Common | Uncommon | Rare | Epic | Total |
|-----|--------|----------|------|------|-------|
| Port Plunder | 8 | 6 | 4 | 2 | 20 |
| Fort Blackstorm | 6 | 5 | 3 | 2 | 16 |
| Cursed Temple | 7 | 5 | 4 | 2 | 18 |
| Volcano Summit | 4 | 4 | 3 | 1 | 12 |
| Ancient Temple | 4 | 3 | 2 | 1 | 10 |
| Witch's Hollow | 3 | 3 | 2 | 0 | 8 |
| Echo Caves | 4 | 3 | 2 | 1 | 10 |
| Lighthouse Point | 2 | 2 | 1 | 0 | 5 |
| Shipwreck Shores | 3 | 3 | 2 | 0 | 8 |
| Parrot Perch | 2 | 2 | 1 | 0 | 5 |
| Wilderness | 20 | 10 | 5 | 0 | 35 |
| **Total** | **63** | **46** | **29** | **9** | **147** |

### 6.4 Ability Spawn Distribution

Abilities spawn from chests and ability orbs:

| Location Type | Ability Chance | Avg Abilities |
|---------------|----------------|---------------|
| S-Tier POI | 40% per chest | 7-8 |
| A-Tier POI | 35% per chest | 4-6 |
| B-Tier POI | 30% per chest | 2-4 |
| C-Tier POI | 25% per chest | 1-2 |
| Wilderness | 20% per chest | 0-1 |

### 6.5 Ground Loot

Ground loot spawns at pre-determined points outside POIs:

| Item Type | Spawn Rate | Locations |
|-----------|------------|-----------|
| Common Ability | 15% | All spawn points |
| Health Pack | 20% | All spawn points |
| Gold | 30% | All spawn points |
| Empty | 35% | - |

**Total Ground Loot Points:** 150 across wilderness areas

---

## 7. Monster Camps

### 7.1 Monster Camp Overview

Monster camps provide PvE opportunities for loot and XP. They respawn periodically during the match.

| Camp Type | Monsters | Difficulty | Reward | Respawn |
|-----------|----------|------------|--------|---------|
| Small | 2-3 | Easy | Common chest | 2 min |
| Medium | 4-5 | Medium | Uncommon chest | 3 min |
| Large | 6-8 | Hard | Rare chest | 4 min |
| Boss | 1 Elite | Very Hard | Epic chest | Once |

### 7.2 Camp Locations

```
Monster Camp Map:
  S = Small camp (×8)
  M = Medium camp (×5)
  L = Large camp (×3)
  B = Boss camp (×2)

    A   B   C   D   E   F
  +---+---+---+---+---+---+
1 |   |   | S |   |   |   |
  +---+---+---+---+---+---+
2 | S | L |   | M | S |   |
  +---+---+---+---+---+---+
3 |   | B |   |   | M | S |
  +---+---+---+---+---+---+
4 | S |   | M |   | L | S |
  +---+---+---+---+---+---+
5 | M |   |   | S | M | B |
  +---+---+---+---+---+---+
6 |   | S |   |   |   |   |
  +---+---+---+---+---+---+
```

### 7.3 Monster Types

| Monster | Health | Damage | Behavior |
|---------|--------|--------|----------|
| Skeleton Pirate | 100 | 10/hit | Melee, slow |
| Ghost Sailor | 150 | 15/hit | Ranged, phases |
| Cursed Crab | 80 | 8/hit | Melee, fast |
| Sea Serpent (Elite) | 500 | 25/hit | Boss, AoE |
| Kraken Spawn (Elite) | 600 | 30/hit | Boss, tentacles |

### 7.4 Boss Spawn Timing

| Boss | Location | Spawn Time | Announcement |
|------|----------|------------|--------------|
| Sea Serpent | B3 (Crater Lake) | 5:00 remaining | Map ping + audio |
| Kraken Spawn | F5 (Coral Reef) | 5:00 remaining | Map ping + audio |

---

## 8. Storm System Integration

### 8.1 Storm Behavior

The storm is a supernatural curse that closes in on the island, forcing players toward the center.

| Phase | Time | Safe Zone Radius | Damage/sec |
|-------|------|------------------|------------|
| Pre-Storm | 60s | Full map | 0 |
| Phase 1 | 90s | 600m | 5 |
| Phase 2 | 75s | 400m | 8 |
| Phase 3 | 60s | 250m | 12 |
| Phase 4 | 45s | 125m | 18 |
| Phase 5 | 30s | 50m | 25 |
| Final | Until end | 0m (closing) | 35 |

### 8.2 Safe Zone Center Weights

The safe zone center has weighted probabilities to create varied gameplay:

| Region | Weight | Notes |
|--------|--------|-------|
| Central (C3-D3) | 35% | Most common, Cursed Temple area |
| North (B2-D2) | 20% | Mountain/Temple endgames |
| South (B5-D5) | 20% | Port area endgames |
| East (E3-E4) | 12% | Jungle endgames |
| West (A3-A4) | 8% | Fort/Lighthouse endgames |
| Corners | 5% | Rare, creates unique games |

### 8.3 Storm Visual Design

- **Color**: Purple/black swirling clouds
- **Edge Effect**: Crackling lightning at boundary
- **Interior**: Reduced visibility, eerie sounds
- **Damage Indicator**: Screen edge glow, heartbeat sound

---

## 9. Navigation and Landmarks

### 9.1 Major Landmarks

Landmarks are visible from long distances and help with orientation:

| Landmark | Location | Visibility Range | Description |
|----------|----------|------------------|-------------|
| Volcano Peak | B2 | Entire map | Smoking mountain top |
| Lighthouse | A4 | 800m | Rotating light beam |
| Clock Tower | B5 | 500m | Tall tower in Port |
| Cursed Glow | C3 | 600m | Green supernatural light |
| Fort Flags | A3 | 600m | Black flags on towers |
| Giant Skeleton | A5 | 400m | Whale bones on beach |

### 9.2 Directional Aids

| Aid | Description | Location |
|-----|-------------|----------|
| Sun Position | East to West during match | Sky |
| Moon Position | Rises in East | Night sky (visual only) |
| Mountain Silhouette | Always North | Background |
| Wave Direction | South to North | Water |

### 9.3 Map Callout Guide

**Standard Callouts:**
- POI names (e.g., "Port", "Temple", "Fort")
- Grid coordinates (e.g., "B3", "D5")
- Terrain features (e.g., "Hill", "Beach", "Jungle")
- Relative directions (e.g., "North Fort", "South Port")

**Example Callouts:**
- "Enemy at Temple courtyard"
- "Rotating to B4 hill"
- "Team in Port docks"
- "Storm pushing us East"

---

## 10. Gameplay Flow

### 10.1 Match Timeline

| Time | Phase | Player Count | Zone | Activity |
|------|-------|--------------|------|----------|
| 0:00 | Drop | 60 | Full | Choosing drop location |
| 0:30 | Early | 55 | Full | Landing, initial looting |
| 2:00 | Early | 45 | Full | POI fights, looting |
| 3:30 | Mid | 35 | Phase 1 | Rotation begins |
| 5:00 | Mid | 25 | Phase 2 | Positioning, PvP |
| 7:00 | Mid-Late | 18 | Phase 3 | Concentrated fights |
| 9:00 | Late | 12 | Phase 4 | Final rotations |
| 11:00 | End | 6 | Phase 5 | Final fights |
| 12:30 | Final | 1-3 | Closing | Victory |

### 10.2 Drop Pattern Analysis

**Expected Drop Distribution:**
| POI | Expected Players | Survival Rate |
|-----|------------------|---------------|
| Port Plunder | 12-15 | 30% |
| Fort Blackstorm | 8-10 | 40% |
| Cursed Temple | 10-12 | 35% |
| Volcano Summit | 6-8 | 50% |
| Ancient Temple | 5-7 | 55% |
| Witch's Hollow | 4-6 | 60% |
| Echo Caves | 5-7 | 55% |
| Lighthouse Point | 3-4 | 65% |
| Shipwreck Shores | 4-6 | 55% |
| Parrot Perch | 2-3 | 70% |

### 10.3 Rotation Patterns

**Common Rotations:**
1. **Port → Temple → Zone**: Most direct central rotation
2. **Fort → Lake → Temple**: Western approach
3. **Caves → Hollow → Temple**: Eastern approach
4. **Lighthouse → Shipwreck → Port**: Coastal loop
5. **Volcano → Temple → Zone**: High ground route

**Rotation Timing Guidelines:**
- Start rotating when zone is 30 seconds from closing
- Factor in terrain (jungle slower, roads faster)
- Consider enemy positions (audio, visual)

### 10.4 Engagement Zones

| Zone | Character | Best Strategy |
|------|-----------|---------------|
| Open (Beach, Plains) | Long-range | Mobility abilities, cover use |
| Urban (POIs) | Mid-range | Building clearing, angles |
| Dense (Jungle, Caves) | Close-range | Ambushes, AoE abilities |
| Vertical (Fort, Temple) | Multi-range | High ground control |

---

## 11. Technical Specifications

### 11.1 Performance Targets

| Metric | Low-End | Mid-Range | High-End |
|--------|---------|-----------|----------|
| Target FPS | 30 | 45 | 60 |
| Draw Calls | 150 | 250 | 400 |
| Triangles | 200K | 400K | 800K |
| Texture Memory | 256MB | 512MB | 1GB |
| LOD Distances | 50/100/200m | 75/150/300m | 100/200/400m |

### 11.2 Streaming Zones

The map is divided into streaming zones to manage memory:

| Zone | Grid | Priority | Always Loaded |
|------|------|----------|---------------|
| Center | C3-D4 | High | Yes |
| South | B5-D6 | Medium | No |
| North | B1-D2 | Medium | No |
| West | A2-A5 | Low | No |
| East | E2-F5 | Low | No |

### 11.3 Collision Layers

| Layer | Contents | Player Interaction |
|-------|----------|-------------------|
| Terrain | Ground, rocks | Walk, climb |
| Structures | Buildings, walls | Block, cover |
| Props | Barrels, crates | Destructible cover |
| Water | Ocean, lake, streams | Swim, slow |
| Trigger | Zones, spawns | Invisible |

### 11.4 Navigation Mesh

- **Resolution**: 0.5m grid
- **Agent Radius**: 0.5m
- **Step Height**: 0.5m
- **Slope Limit**: 45 degrees
- **Jump Distance**: 3m (ability-enhanced: 6m)

---

## 12. Art Direction

### 12.1 Visual Style per Zone

| Zone | Color Palette | Mood | Key Props |
|------|---------------|------|-----------|
| Beach | Warm yellows, turquoise | Bright, inviting | Palm trees, driftwood |
| Jungle | Deep greens, browns | Dense, mysterious | Vines, exotic flowers |
| Mountain | Grays, browns, whites | Harsh, imposing | Rocks, snow caps |
| Volcanic | Reds, oranges, blacks | Dangerous, dramatic | Lava, ash, smoke |
| Swamp | Murky greens, purples | Eerie, magical | Mist, twisted trees |
| Urban | Weathered woods, metals | Lived-in, pirate | Barrels, ropes, flags |

### 12.2 Prop Density Guidelines

| Zone | Props per 100m² | Density Feel |
|------|-----------------|--------------|
| Beach | 5-10 | Sparse |
| Jungle | 20-30 | Dense |
| Mountain | 8-12 | Moderate |
| Urban | 25-40 | Heavy |
| Plains | 3-5 | Sparse |

### 12.3 Lighting

| Time of Day | Sun Position | Mood | Notes |
|-------------|--------------|------|-------|
| Morning | Low East | Warm, hopeful | Golden hour feel |
| Midday | High | Neutral, bright | Clear visibility |
| Evening | Low West | Dramatic, orange | Long shadows |

**Default Setting**: Late afternoon (4-5 PM equivalent)

### 12.4 Weather Effects

| Effect | Frequency | Visual Impact | Gameplay Impact |
|--------|-----------|---------------|-----------------|
| Clear | 60% | Default | None |
| Cloudy | 25% | Darker, moody | Slightly reduced visibility |
| Rain | 10% | Particle effects, wet surfaces | Audio masking |
| Fog | 5% | Reduced draw distance | Significant visibility reduction |

---

## 13. Audio Zones

### 13.1 Ambient Audio Regions

| Region | Ambient Sounds | Music Mood |
|--------|----------------|------------|
| Beach | Waves, seagulls, wind | Relaxed, adventure |
| Jungle | Birds, insects, rustling | Mysterious, tense |
| Mountain | Wind, distant rumbles | Epic, isolated |
| Volcano | Rumbling, hissing steam | Dangerous, intense |
| Swamp | Frogs, bubbling, eerie whispers | Creepy, magical |
| Urban | Creaking wood, flags flapping | Pirate, lively |
| Caves | Dripping water, echoes | Enclosed, tense |

### 13.2 Dynamic Audio

| Trigger | Audio Response |
|---------|----------------|
| Combat nearby | Music intensifies |
| Storm approaching | Warning rumble |
| Low health | Heartbeat effect |
| Final circles | Dramatic crescendo |
| Victory | Triumphant fanfare |

### 13.3 3D Audio Landmarks

| Landmark | Sound | Range |
|----------|-------|-------|
| Volcano | Rumbling | 300m |
| Lighthouse | Bell | 200m |
| Witch's Hollow | Eerie whispers | 100m |
| Waterfall (Lake) | Rushing water | 150m |
| Port | Crowd murmur | 100m |

---

## 14. Appendices

### Appendix A: POI Quick Reference Card

```
+------------------+------+------+-------+-------+
| POI              | Grid | Tier | Loot  | Risk  |
+------------------+------+------+-------+-------+
| Port Plunder     | B5   | S    | 20    | High  |
| Fort Blackstorm  | A3   | A    | 16    | High  |
| Cursed Temple    | C3   | S    | 18    | High  |
| Volcano Summit   | B2   | A    | 12    | Med   |
| Ancient Temple   | D2   | B    | 10    | Med   |
| Witch's Hollow   | D4   | B    | 8     | Med   |
| Echo Caves       | E4   | B    | 10    | Med   |
| Lighthouse Point | A4   | C    | 5     | Low   |
| Shipwreck Shores | A5   | B    | 8     | Med   |
| Parrot Perch     | F3   | C    | 5     | Low   |
+------------------+------+------+-------+-------+
```

### Appendix B: Distance Matrix (Approximate)

Travel times in seconds (running, no abilities):

| From \ To | Port | Fort | Temple | Volcano |
|-----------|------|------|--------|---------|
| Port | - | 45s | 30s | 50s |
| Fort | 45s | - | 35s | 40s |
| Temple | 30s | 35s | - | 35s |
| Volcano | 50s | 40s | 35s | - |

### Appendix C: Greybox Checklist

- [ ] Terrain mesh complete
- [ ] POI blockout meshes placed
- [ ] Collision volumes set
- [ ] Navigation mesh generated
- [ ] Spawn points placed
- [ ] Loot spawn points placed
- [ ] Monster camp locations marked
- [ ] Storm zone centers defined
- [ ] Streaming zones configured
- [ ] Basic lighting pass
- [ ] Playtest for flow

### Appendix D: Map Evolution Plan

| Phase | Focus | Changes |
|-------|-------|---------|
| Alpha | Greybox testing | Layout adjustments |
| Beta | Art pass | Visual polish |
| Launch | Final tuning | Balance tweaks |
| Post-Launch | New content | Additional POIs, events |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Level Designer | _____________ | _____________ | ___/___/___ |
| Art Director | _____________ | _____________ | ___/___/___ |
| Game Designer | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial map design document |

---

*This document serves as the foundation for level design and will be updated based on playtesting feedback.*
