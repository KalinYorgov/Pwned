# Default Pirate Outfit Set Specification

## Document Information
- **Task ID:** ART-004
- **Priority:** P0
- **Complexity:** M (Medium)
- **Dependencies:** ART-003 (Base Player Character Model)
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the default starting outfit for all new players in Plunderstorm Mobile. The outfit serves as the baseline cosmetic that every player begins with, establishing the visual identity while clearly indicating "starter" status to encourage cosmetic upgrades.

### 1.2 Design Goals
- **Appealing but Basic:** Attractive enough to enjoy, simple enough to want upgrades
- **Universally Pirate:** Instantly readable as a pirate character
- **Performance Optimal:** Lightweight for mobile rendering
- **Animation Compatible:** No clipping with any character animations

### 1.3 Scope
- Head accessories (bandana, basic hat options)
- Upper body clothing (vest, shirt)
- Lower body clothing (pants, boots)
- Basic accessories (belt, pouches)
- Color variants for variety

---

## 2. Design Direction

### 2.1 Visual Style

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Default Outfit Design Pillars                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │    HUMBLE       │  │    PRACTICAL    │  │    HOPEFUL      │          │
│  │    BEGINNINGS   │  │    SEAFARER     │  │    ADVENTURER   │          │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤          │
│  │ • Worn fabrics  │  │ • Functional    │  │ • Clean design  │          │
│  │ • Muted colors  │  │   not fancy     │  │ • Room to grow  │          │
│  │ • Simple cuts   │  │ • Work clothes  │  │ • Aspirational  │          │
│  │ • No gold/gems  │  │ • Durable look  │  │ • Likeable      │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
│                                                                          │
│  Inspiration: New crew member on their first voyage                     │
│  NOT: Captain, veteran pirate, or fancy nobleman                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Color Palette

```yaml
default_outfit_colors:
  primary_palette:
    cream_white:
      hex: "#F5E6D3"
      usage: "Shirt base"

    weathered_brown:
      hex: "#8B7355"
      usage: "Vest, belt, boots"

    faded_navy:
      hex: "#4A5568"
      usage: "Pants, bandana option"

    rope_tan:
      hex: "#C4A574"
      usage: "Accents, stitching"

  accent_colors:
    rust_red:
      hex: "#A0522D"
      usage: "Bandana option, trim"

    sea_green:
      hex: "#5F9EA0"
      usage: "Alternate bandana"

    worn_brass:
      hex: "#B8860B"
      usage: "Buckles (subtle, not gold)"

  restrictions:
    - "No bright gold (reserved for premium)"
    - "No pure white (gets dirty at sea)"
    - "No black (reserved for captain outfits)"
    - "No glowing/emissive elements"
```

### 2.3 Reference Mood Board

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Visual References                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Similar Characters:                                                     │
│  ├── Sea of Thieves: Default pirate (practical, worn)                   │
│  ├── Assassin's Creed Black Flag: Young Edward (pre-captain)            │
│  ├── Fortnite: Uncommon tier outfits (simple but polished)              │
│  └── One Piece: Early Luffy outfit (simple vest, shorts)                │
│                                                                          │
│  Material References:                                                    │
│  ├── Linen shirts (sun-bleached, soft wrinkles)                         │
│  ├── Leather (worn but maintained, not cracked)                         │
│  ├── Canvas (sturdy work fabric)                                        │
│  └── Simple cotton bandana (soft, tied)                                 │
│                                                                          │
│  Avoid:                                                                  │
│  ├── Elaborate embroidery                                               │
│  ├── Feathered hats                                                     │
│  ├── Jewelry or gold chains                                             │
│  └── Military/naval uniforms                                            │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Outfit Components

### 3.1 Component Breakdown

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Default Outfit Assembly                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                         ┌───────────────┐                                │
│                         │   BANDANA     │  SLOT_HEAD                     │
│                         │   (Option A)  │                                │
│                         └───────┬───────┘                                │
│                                 │                                        │
│                         ┌───────┴───────┐                                │
│                         │     HEAD      │  Base character                │
│                         └───────┬───────┘                                │
│                                 │                                        │
│      ┌──────────────────────────┼──────────────────────────┐            │
│      │                          │                          │            │
│ ┌────┴────┐              ┌──────┴──────┐             ┌────┴────┐        │
│ │  SHIRT  │              │    VEST     │             │  SHIRT  │        │
│ │ (Under) │              │   (Over)    │             │ (Sleeve)│        │
│ └────┬────┘              └──────┬──────┘             └────┬────┘        │
│      │                          │                          │            │
│      └──────────────────────────┼──────────────────────────┘            │
│                                 │                                        │
│                         ┌───────┴───────┐                                │
│                         │     BELT      │  SLOT_BELT                     │
│                         │  + POUCHES    │                                │
│                         └───────┬───────┘                                │
│                                 │                                        │
│                         ┌───────┴───────┐                                │
│                         │    PANTS      │  SLOT_LEGS                     │
│                         └───────┬───────┘                                │
│                                 │                                        │
│                         ┌───────┴───────┐                                │
│                         │    BOOTS      │  (Part of SLOT_LEGS)           │
│                         └───────────────┘                                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Head Slot - Bandana

```yaml
component_bandana:
  name: "Deckhand's Bandana"
  slot: SLOT_HEAD

  description: |
    A simple cloth bandana tied around the head, keeping hair
    and sweat out of the eyes during hard work on deck.

  design_details:
    style: "Tied at back, slight drape"
    knot_position: "Back of head, slightly off-center"
    fabric_behavior: "Soft drape, subtle wind movement"
    coverage: "Forehead to crown, ears exposed"

  technical_specs:
    triangles: 400
    texture_resolution: 256x256
    uv_space: "Shared with outfit atlas"
    bones_affected: ["Head", "Hat_Attach"]

  color_variants:
    - name: "Faded Red"
      primary: "#A0522D"
    - name: "Navy Blue"
      primary: "#4A5568"
    - name: "Sea Green"
      primary: "#5F9EA0"
    - name: "Sand Tan"
      primary: "#C4A574"

  animation_notes:
    - "Slight secondary motion on tail ends"
    - "No collision with hair (hair hidden under bandana)"
    - "Stays secure during combat animations"
```

### 3.3 Torso Slot - Shirt and Vest

```yaml
component_torso:
  name: "Deckhand's Shirt & Vest"
  slot: SLOT_TORSO

  shirt:
    name: "Worn Linen Shirt"
    description: |
      A loose-fitting linen shirt, sleeves rolled up for work.
      Sun-bleached and well-worn but clean.

    design_details:
      collar: "Open V-neck, no buttons visible"
      sleeves: "Rolled to elbow, loose billow"
      fit: "Loose, tucked into pants at waist"
      wear_details: "Subtle wrinkles, slight discoloration"

    color:
      primary: "#F5E6D3"  # Cream white
      shadows: "#E6D5C3"  # Warm shadow

  vest:
    name: "Simple Leather Vest"
    description: |
      A practical sleeveless vest of worn leather,
      providing protection without restricting movement.

    design_details:
      cut: "Waist-length, open front"
      closure: "Two simple brass buttons (not gold)"
      details: "Visible stitching, subtle wear on edges"
      pockets: "One small chest pocket"

    color:
      primary: "#8B7355"  # Weathered brown
      stitching: "#C4A574"  # Rope tan
      buttons: "#B8860B"  # Worn brass

  technical_specs:
    triangles: 1800  # Combined shirt + vest
    texture_resolution: 512x512
    uv_space: "Primary outfit atlas"
    deformation_zones:
      - "Shoulder (shirt billow)"
      - "Elbow (sleeve roll)"
      - "Waist (tuck behavior)"

  animation_notes:
    - "Vest stays relatively rigid"
    - "Shirt sleeves have slight secondary motion"
    - "No clipping at shoulder during arm raise"
```

### 3.4 Belt Slot - Belt and Pouches

```yaml
component_belt:
  name: "Seafarer's Belt"
  slot: SLOT_BELT

  description: |
    A sturdy leather belt with a simple brass buckle,
    featuring a small pouch for essentials.

  design_details:
    belt:
      width: "Medium (not too wide)"
      buckle: "Simple square brass, worn finish"
      holes: "Multiple, showing use adjustment"

    pouch:
      position: "Right hip"
      size: "Small, utilitarian"
      closure: "Flap with button"
      contents_implied: "Coins, small tools"

    optional_elements:
      rope_coil: "Small coiled rope on left hip"
      knife_sheath: "Empty (weapon slot separate)"

  technical_specs:
    triangles: 350
    texture_resolution: 256x256
    bones_affected: ["Pelvis", "Belt_Attach"]

  color:
    belt_leather: "#8B7355"
    buckle: "#B8860B"
    pouch: "#A0522D"
    rope: "#C4A574"

  animation_notes:
    - "Pouch has subtle physics sway"
    - "Belt stays fixed to waist"
    - "No clipping during crouch/sit"
```

### 3.5 Legs Slot - Pants and Boots

```yaml
component_legs:
  name: "Deckhand's Pants & Boots"
  slot: SLOT_LEGS

  pants:
    name: "Canvas Work Pants"
    description: |
      Durable canvas pants, practical for ship work.
      Slightly loose fit, tucked into boots.

    design_details:
      fit: "Relaxed, not baggy"
      length: "Full length, tucked into boots"
      details: "Simple pockets, visible seams"
      wear: "Faded at knees, slight patches"

    color:
      primary: "#4A5568"  # Faded navy
      seams: "#3D4852"
      patches: "#5A6978"  # Slightly lighter

  boots:
    name: "Worn Leather Boots"
    description: |
      Sturdy mid-calf boots, well-worn but maintained.
      Good grip for wet deck work.

    design_details:
      height: "Mid-calf, below knee"
      sole: "Thick, practical"
      lacing: "Simple front lacing"
      wear: "Scuffed toes, salt stains"

    color:
      leather: "#6B5344"  # Dark brown
      sole: "#4A3C2F"
      laces: "#C4A574"

  technical_specs:
    triangles: 1400  # Combined pants + boots
    texture_resolution: 512x512
    deformation_zones:
      - "Hip (crouch)"
      - "Knee (bend)"
      - "Ankle (boot flex)"

  animation_notes:
    - "Pants fabric follows leg movement"
    - "Boots maintain shape, minimal flex"
    - "No clipping at knee during run cycle"
```

---

## 4. Technical Specifications

### 4.1 Polygon Budget

```yaml
polygon_budget:
  total_outfit: 4000 triangles max

  breakdown:
    bandana: 400
    torso_combined: 1800
    belt_accessories: 350
    legs_combined: 1400
    buffer: 50

  lod_variants:
    LOD0_High:
      total: 4000
      distance: "0-15m"

    LOD1_Medium:
      total: 2000
      distance: "15-40m"
      simplifications:
        - "Merge small accessories"
        - "Reduce cloth detail"
        - "Simplify stitching"

    LOD2_Low:
      total: 1000
      distance: "40-80m"
      simplifications:
        - "Remove pouches"
        - "Flatten all detail"
        - "Single material"
```

### 4.2 Texture Specifications

```yaml
texture_specs:
  atlas_layout:
    resolution: 1024x1024
    format: "ASTC 6x6 (mobile)"

    allocation:
      torso: "50% of atlas"
      legs: "30% of atlas"
      head_accessories: "10% of atlas"
      belt: "10% of atlas"

  texture_maps:
    albedo:
      description: "Base color with baked AO"
      bit_depth: 8

    normal:
      description: "Surface detail"
      bit_depth: 8
      format: "RG compressed"
      details:
        - "Fabric weave"
        - "Leather grain"
        - "Stitching depth"
        - "Wrinkle folds"

    mask:
      description: "Material properties"
      channels:
        R: "Roughness (fabric rough, leather medium)"
        G: "Metallic (only buckles)"
        B: "AO (seams, folds)"
        A: "Unused (future)"

  texel_density:
    target: "10 pixels per unit"
    consistency: "Within 15% variance"
```

### 4.3 Material Setup

```yaml
material_configuration:
  shader: "Plunderstorm/Character"

  material_instances:
    outfit_main:
      name: "M_Outfit_Default"
      properties:
        _BaseColor: "T_Outfit_Default_Albedo"
        _NormalMap: "T_Outfit_Default_Normal"
        _MaskMap: "T_Outfit_Default_Mask"
        _RoughnessMultiplier: 1.0
        _NormalStrength: 1.0

    belt_metal:
      name: "M_Outfit_Default_Metal"
      properties:
        _Metallic: 0.8
        _Roughness: 0.4
      notes: "Only for buckle elements"

  draw_calls:
    target: 1
    method: "Single atlas material"
```

### 4.4 Attachment Configuration

```json
{
  "outfit_id": "default_deckhand",
  "display_name": "Deckhand's Garb",
  "rarity": "Common",
  "unlock_condition": "Default",

  "components": [
    {
      "slot": "SLOT_HEAD",
      "mesh": "SM_Outfit_Default_Bandana",
      "material": "M_Outfit_Default",
      "hide_base_mesh": ["hair_base"],
      "bone_parent": "Hat_Attach"
    },
    {
      "slot": "SLOT_TORSO",
      "mesh": "SM_Outfit_Default_Torso",
      "material": "M_Outfit_Default",
      "hide_base_mesh": ["torso_base"],
      "bone_parent": "Spine_02"
    },
    {
      "slot": "SLOT_BELT",
      "mesh": "SM_Outfit_Default_Belt",
      "material": "M_Outfit_Default",
      "hide_base_mesh": [],
      "bone_parent": "Pelvis"
    },
    {
      "slot": "SLOT_LEGS",
      "mesh": "SM_Outfit_Default_Legs",
      "material": "M_Outfit_Default",
      "hide_base_mesh": ["legs_base"],
      "bone_parent": "Pelvis"
    }
  ],

  "color_variants": [
    {
      "id": "default_red",
      "name": "Crimson Deckhand",
      "bandana_color": "#A0522D"
    },
    {
      "id": "default_blue",
      "name": "Navy Deckhand",
      "bandana_color": "#4A5568"
    },
    {
      "id": "default_green",
      "name": "Seafoam Deckhand",
      "bandana_color": "#5F9EA0"
    },
    {
      "id": "default_tan",
      "name": "Sandy Deckhand",
      "bandana_color": "#C4A574"
    }
  ]
}
```

---

## 5. Animation Compatibility

### 5.1 Deformation Testing

```yaml
animation_tests:
  locomotion:
    idle:
      concerns: ["Shirt settling", "Belt position"]
      pass_criteria: "No visible clipping or stretching"

    walk:
      concerns: ["Pants at hip", "Boot flex"]
      pass_criteria: "Natural fabric movement"

    run:
      concerns: ["Vest flapping", "Bandana stability"]
      pass_criteria: "Secondary motion without chaos"

    sprint:
      concerns: ["Extreme poses", "Arm swing"]
      pass_criteria: "No shoulder clipping"

  combat:
    attack_melee:
      concerns: ["Torso twist", "Arm extension"]
      pass_criteria: "No vest clipping through arm"

    attack_ability:
      concerns: ["Various casting poses"]
      pass_criteria: "Sleeves don't clip through body"

    hurt_reaction:
      concerns: ["Rapid motion", "Bend back"]
      pass_criteria: "Outfit follows body"

    death:
      concerns: ["Ground contact", "Ragdoll"]
      pass_criteria: "No major interpenetration"

  utility:
    jump:
      concerns: ["Air pose", "Landing impact"]
      pass_criteria: "Outfit settles naturally"

    crouch:
      concerns: ["Knee bend", "Waist compression"]
      pass_criteria: "No pants stretching artifacts"

    emotes:
      concerns: ["Dance moves", "Sit pose"]
      pass_criteria: "Compatible with all launch emotes"
```

### 5.2 Clipping Prevention

```yaml
clipping_solutions:
  shoulder_arm:
    problem: "Vest clips through arm during raise"
    solution: "Add armhole cutout, push weights to arm"

  waist_belt:
    problem: "Belt floats during crouch"
    solution: "Parent to pelvis with slight offset"

  knee_pants:
    problem: "Pants stretch at extreme bend"
    solution: "Add knee geometry, blend weights"

  neck_collar:
    problem: "Shirt collar clips through neck"
    solution: "Lower collar geometry, wider opening"

  boot_pants:
    problem: "Pants pop out of boots"
    solution: "Extend boot geometry over pants cuff"
```

### 5.3 Secondary Motion

```yaml
secondary_motion:
  enabled_elements:
    bandana_tails:
      type: "Cloth simulation (simplified)"
      bones: 2
      stiffness: 0.7
      damping: 0.5

    shirt_sleeves:
      type: "Bone-based jiggle"
      bones: 1 per sleeve
      subtle: true

    belt_pouch:
      type: "Spring bone"
      bones: 1
      max_angle: 15°

  performance_settings:
    update_rate: "Every frame for player, every 3rd for others"
    lod_disable: "LOD2 and below"
    quality_tier: "Medium and above only"
```

---

## 6. Variant System

### 6.1 Color Variants

```yaml
color_variants:
  implementation: "Material property override"

  variants:
    crimson_deckhand:
      id: "default_01"
      bandana: "#A0522D"
      vest_tint: 1.0  # No change
      unlock: "Default"

    navy_deckhand:
      id: "default_02"
      bandana: "#4A5568"
      vest_tint: 1.0
      unlock: "Default"

    seafoam_deckhand:
      id: "default_03"
      bandana: "#5F9EA0"
      vest_tint: 1.0
      unlock: "Default"

    sandy_deckhand:
      id: "default_04"
      bandana: "#C4A574"
      vest_tint: 1.0
      unlock: "Default"

  randomization:
    new_player: "Random from 4 variants"
    change_option: "Settings menu, free"
```

### 6.2 Future Variant Expansion

```yaml
future_variants:
  seasonal_colors:
    winter: "Blue and white tones"
    summer: "Bright tropical colors"
    halloween: "Orange and black"
    note: "Same geometry, material swap"

  achievement_unlocks:
    first_win: "Gold-stitched variant"
    level_10: "Slightly upgraded vest"
    note: "Still clearly 'starter tier'"
```

---

## 7. Quality Assurance

### 7.1 Visual Checklist

```yaml
visual_qa_checklist:
  style_compliance:
    - [ ] Matches visual style guide proportions
    - [ ] Colors within approved palette
    - [ ] No premium-tier visual elements
    - [ ] Clearly reads as "starter" outfit

  technical_quality:
    - [ ] Poly count within budget
    - [ ] Textures at correct resolution
    - [ ] UV layout efficient
    - [ ] Normal map detail appropriate

  silhouette:
    - [ ] Recognizable at 50m distance
    - [ ] Distinct from premium outfits
    - [ ] Character shape readable

  material_quality:
    - [ ] Fabric looks like fabric
    - [ ] Leather has appropriate sheen
    - [ ] Metal only on buckles
    - [ ] No glowing elements
```

### 7.2 Animation Checklist

```yaml
animation_qa_checklist:
  clipping_tests:
    - [ ] Idle: No clipping
    - [ ] Walk: No clipping
    - [ ] Run: No clipping
    - [ ] Jump: No clipping
    - [ ] Crouch: No clipping
    - [ ] All attacks: No clipping
    - [ ] All abilities: No clipping
    - [ ] Death: Acceptable clipping only

  deformation_tests:
    - [ ] Shoulder raises cleanly
    - [ ] Elbow bends naturally
    - [ ] Waist bends without breaking
    - [ ] Knees bend without stretching
    - [ ] Ankles rotate properly
```

### 7.3 Performance Checklist

```yaml
performance_qa_checklist:
  rendering:
    - [ ] Single draw call achieved
    - [ ] LOD transitions smooth
    - [ ] No visual pop on LOD change

  memory:
    - [ ] Texture memory within budget
    - [ ] Mesh memory within budget

  frame_rate:
    - [ ] 60 FPS with 60 characters (default outfit)
    - [ ] No frame drops during combat
```

---

## 8. Integration Requirements

### 8.1 Character Customization System

```yaml
customization_integration:
  selection_ui:
    location: "Locker / Outfit menu"
    preview: "Full character rotation"
    equip_action: "Single tap"

  persistence:
    storage: "Player profile (cloud)"
    default: "Random variant on new account"

  loading:
    preload: "Always loaded (default fallback)"
    streaming: "Not applicable"
```

### 8.2 Inventory System

```yaml
inventory_integration:
  item_entry:
    id: "outfit_default_deckhand"
    type: "outfit_full"
    rarity: "common"
    tradeable: false
    dismantleable: false

  display:
    icon: "T_Icon_Outfit_Default"
    name_localization_key: "OUTFIT_DEFAULT_NAME"
    description_key: "OUTFIT_DEFAULT_DESC"
```

### 8.3 New Player Flow

```yaml
new_player_experience:
  account_creation:
    - Assign random color variant
    - Equip as default outfit
    - No tutorial for outfit (assumed)

  first_match:
    - Player wears default outfit
    - Clear visual difference from any premium players
    - Aspiration to upgrade established
```

---

## 9. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|--------------|---------------------|
| Outfit fits base character model | Uses ART-003 slots | Attachment test in engine |
| Follows style guide | Colors, proportions match | Art director review |
| Visually distinct at distance | Readable silhouette at 50m | Distance test scene |
| No clipping issues | All animations tested | Animation QA checklist |
| Texture budget met | 1024x1024 atlas | Texture memory check |
| Gender variants if applicable | Single unisex design | N/A (body type handles this) |

---

## 10. Deliverables

### 10.1 Asset List

```yaml
deliverables:
  meshes:
    - "SM_Outfit_Default_Bandana.fbx"
    - "SM_Outfit_Default_Torso.fbx"
    - "SM_Outfit_Default_Belt.fbx"
    - "SM_Outfit_Default_Legs.fbx"
    - "SM_Outfit_Default_LOD1.fbx"
    - "SM_Outfit_Default_LOD2.fbx"

  textures:
    - "T_Outfit_Default_Albedo.png"
    - "T_Outfit_Default_Normal.png"
    - "T_Outfit_Default_Mask.png"

  materials:
    - "M_Outfit_Default.mat"

  prefabs:
    - "Outfit_Default_Deckhand.prefab"

  icons:
    - "T_Icon_Outfit_Default.png" (256x256)
    - "T_Icon_Outfit_Default_Small.png" (64x64)

  source_files:
    - "Outfit_Default.ma" (Maya)
    - "Outfit_Default.spp" (Substance)
```

### 10.2 Documentation

```yaml
documentation:
  - This specification document
  - Texture breakdown diagram
  - Animation compatibility notes
  - Color variant hex values
```

---

## 11. Appendix

### 11.1 Related Documents

- [BASE_PLAYER_CHARACTER_MODEL_SPEC.md](BASE_PLAYER_CHARACTER_MODEL_SPEC.md) - Base character
- [docs/VISUAL_STYLE_GUIDE.md](docs/VISUAL_STYLE_GUIDE.md) - Style reference
- [docs/TECHNICAL_ART_SPECIFICATIONS.md](docs/TECHNICAL_ART_SPECIFICATIONS.md) - Technical limits

### 11.2 Localization Keys

```yaml
localization:
  OUTFIT_DEFAULT_NAME: "Deckhand's Garb"
  OUTFIT_DEFAULT_DESC: "The practical attire of a new sailor, ready for adventure on the high seas."
  OUTFIT_DEFAULT_VARIANT_RED: "Crimson Deckhand"
  OUTFIT_DEFAULT_VARIANT_BLUE: "Navy Deckhand"
  OUTFIT_DEFAULT_VARIANT_GREEN: "Seafoam Deckhand"
  OUTFIT_DEFAULT_VARIANT_TAN: "Sandy Deckhand"
```

### 11.3 Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-30 | Initial specification |
