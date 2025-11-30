# Base Player Character Model Specification

## Document Information
- **Task ID:** ART-003
- **Priority:** P0
- **Complexity:** L (Large)
- **Dependencies:** ART-001 (Visual Style Guide), ART-002 (Technical Art Specifications)
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the base player character model for Plunderstorm Mobile, establishing the foundation for all player cosmetics, animations, and visual customization. The model serves as the core visual representation of players in the game.

### 1.2 Scope
- Base character mesh (body type variants)
- Modular attachment system
- Skeleton rig for animation
- UV mapping and texture specifications
- LOD (Level of Detail) system
- Performance optimization for mobile

### 1.3 Goals
- Create an appealing, stylized pirate character
- Support extensive cosmetic customization
- Achieve 60 FPS with 60 characters on screen
- Maintain readable silhouette at all distances

---

## 2. Character Design

### 2.1 Style Direction

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Character Style Pillars                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │   STYLIZED      │  │   HEROIC        │  │   READABLE      │          │
│  │   PROPORTIONS   │  │   SILHOUETTE    │  │   AT DISTANCE   │          │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤          │
│  │ • 6.5 head tall │  │ • Broad         │  │ • Strong shapes │          │
│  │ • Large hands   │  │   shoulders     │  │ • High contrast │          │
│  │ • Exaggerated   │  │ • Clear pose    │  │ • Bold colors   │          │
│  │   features      │  │ • Distinct      │  │ • Clean edges   │          │
│  │ • Expressive    │  │   profile       │  │ • No noise      │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
│                                                                          │
│  Reference Style: Fortnite meets Sea of Thieves                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Body Type Variants

| Variant | Description | Proportions | Use Case |
|---------|-------------|-------------|----------|
| Type A (Standard) | Athletic build | 6.5 heads tall, balanced | Default for most cosmetics |
| Type B (Heroic) | Broader, more muscular | 6 heads tall, wide shoulders | Premium "captain" looks |
| Type C (Agile) | Leaner, taller | 7 heads tall, narrow | Acrobatic themed cosmetics |

**Note:** All body types share the same skeleton rig and attachment points for animation compatibility.

### 2.3 Proportions Guide

```
                    ┌─────┐
                    │HEAD │  ═══ 1 head unit
                    ├─────┤
                    │     │
                    │TORSO│  ═══ 2 head units
                    │     │
                    ├─────┤
                    │     │
                    │     │
                    │LEGS │  ═══ 3 head units
                    │     │
                    │     │
                    └─────┘

    Total Height: 6.5 head units (stylized, not realistic)

    Key Proportions:
    ├── Head: 15% of total height (larger than realistic)
    ├── Torso: 31% of total height
    ├── Legs: 46% of total height
    ├── Arms: Reach to mid-thigh
    └── Hands: 1.2x realistic size (for weapon visibility)
```

---

## 3. Modular System

### 3.1 Attachment Point Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Modular Attachment System                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                         ┌───────────┐                                    │
│                         │   HEAD    │                                    │
│                         │  SOCKET   │                                    │
│                         └─────┬─────┘                                    │
│                               │                                          │
│                    ┌──────────┼──────────┐                               │
│                    │          │          │                               │
│              ┌─────┴─────┐    │    ┌─────┴─────┐                         │
│              │ SHOULDER  │    │    │ SHOULDER  │                         │
│              │   LEFT    │    │    │   RIGHT   │                         │
│              └─────┬─────┘    │    └─────┬─────┘                         │
│                    │          │          │                               │
│              ┌─────┴─────┐    │    ┌─────┴─────┐                         │
│              │   HAND    │    │    │   HAND    │                         │
│              │   LEFT    │    │    │   RIGHT   │                         │
│              └───────────┘    │    └───────────┘                         │
│                               │                                          │
│                         ┌─────┴─────┐                                    │
│                         │   TORSO   │                                    │
│                         │   CORE    │                                    │
│                         └─────┬─────┘                                    │
│                               │                                          │
│                    ┌──────────┼──────────┐                               │
│                    │          │          │                               │
│              ┌─────┴─────┐    │    ┌─────┴─────┐                         │
│              │   HIP     │    │    │   HIP     │                         │
│              │   LEFT    │    │    │   RIGHT   │                         │
│              └─────┬─────┘    │    └─────┬─────┘                         │
│                    │          │          │                               │
│              ┌─────┴─────┐    │    ┌─────┴─────┐                         │
│              │   FOOT    │    │    │   FOOT    │                         │
│              │   LEFT    │    │    │   RIGHT   │                         │
│              └───────────┘    │    └───────────┘                         │
│                               │                                          │
│                    Additional Attachment Points:                         │
│                    ├── BACK_UPPER (capes, wings)                         │
│                    ├── BACK_LOWER (tails, backpacks)                     │
│                    ├── BELT_LEFT (scabbard, pouches)                     │
│                    ├── BELT_RIGHT (holster, bottles)                     │
│                    └── CHEST_FRONT (medals, amulets)                     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Modular Slots

| Slot ID | Slot Name | Description | Poly Budget | Examples |
|---------|-----------|-------------|-------------|----------|
| SLOT_01 | Head | Hair, hats, helmets | 800-1500 | Tricorn, bandana, crown |
| SLOT_02 | Face | Facial accessories | 200-500 | Eyepatch, beard, mask |
| SLOT_03 | Torso | Upper body clothing | 1500-2500 | Vest, coat, armor |
| SLOT_04 | Arms | Arm accessories | 400-800 | Gloves, bracers, tattoo sleeves |
| SLOT_05 | Legs | Lower body clothing | 1000-1800 | Pants, skirts, boots |
| SLOT_06 | Back | Back attachments | 500-1200 | Cape, wings, backpack |
| SLOT_07 | Belt | Waist accessories | 300-600 | Belt, sash, pouches |
| SLOT_08 | Weapon_R | Right hand weapon | 400-800 | Cutlass, pistol |
| SLOT_09 | Weapon_L | Left hand/offhand | 300-600 | Shield, hook, lantern |

### 3.3 Slot Configuration File

```json
{
  "modular_slots": {
    "SLOT_HEAD": {
      "bone_parent": "head",
      "position_offset": [0, 0.15, 0],
      "rotation_offset": [0, 0, 0],
      "scale_range": [0.9, 1.1],
      "hide_base_mesh": ["hair_base"],
      "poly_budget": 1500,
      "allowed_types": ["hat", "helmet", "hair", "headband"]
    },
    "SLOT_FACE": {
      "bone_parent": "head",
      "position_offset": [0, 0, 0.05],
      "rotation_offset": [0, 0, 0],
      "scale_range": [0.95, 1.05],
      "hide_base_mesh": [],
      "poly_budget": 500,
      "allowed_types": ["eyepatch", "beard", "mask", "glasses"]
    },
    "SLOT_TORSO": {
      "bone_parent": "spine_02",
      "position_offset": [0, 0, 0],
      "rotation_offset": [0, 0, 0],
      "scale_range": [1.0, 1.0],
      "hide_base_mesh": ["torso_base"],
      "poly_budget": 2500,
      "allowed_types": ["shirt", "vest", "coat", "armor"]
    },
    "SLOT_LEGS": {
      "bone_parent": "pelvis",
      "position_offset": [0, 0, 0],
      "rotation_offset": [0, 0, 0],
      "scale_range": [1.0, 1.0],
      "hide_base_mesh": ["legs_base"],
      "poly_budget": 1800,
      "allowed_types": ["pants", "shorts", "skirt", "boots"]
    },
    "SLOT_BACK": {
      "bone_parent": "spine_03",
      "position_offset": [0, -0.15, -0.1],
      "rotation_offset": [0, 0, 0],
      "scale_range": [0.8, 1.2],
      "hide_base_mesh": [],
      "poly_budget": 1200,
      "allowed_types": ["cape", "wings", "backpack", "quiver"]
    },
    "SLOT_WEAPON_R": {
      "bone_parent": "hand_r",
      "position_offset": [0, 0, 0],
      "rotation_offset": [0, 0, 0],
      "scale_range": [0.9, 1.1],
      "hide_base_mesh": [],
      "poly_budget": 800,
      "allowed_types": ["sword", "pistol", "staff", "hammer"]
    }
  }
}
```

---

## 4. Technical Specifications

### 4.1 Polygon Budgets

```yaml
polygon_budgets:
  base_character:
    high_lod:
      total: 8000 triangles
      breakdown:
        head: 1200
        torso: 2500
        arms: 1600
        legs: 2200
        hands: 500

    medium_lod:
      total: 4000 triangles
      breakdown:
        head: 600
        torso: 1200
        arms: 800
        legs: 1100
        hands: 300

    low_lod:
      total: 2000 triangles
      breakdown:
        head: 300
        torso: 600
        arms: 400
        legs: 550
        hands: 150

    culled_lod:
      total: 500 triangles
      description: "Billboard or impostor for extreme distance"

  with_full_cosmetics:
    high_lod: 14000 triangles max
    medium_lod: 7000 triangles max
    low_lod: 3500 triangles max
```

### 4.2 LOD Distance Thresholds

```yaml
lod_configuration:
  LOD0_High:
    distance: 0-15 meters
    triangles: 8000 (base) + cosmetics
    shadows: Full
    use_case: "Player's own character, nearby enemies"

  LOD1_Medium:
    distance: 15-40 meters
    triangles: 4000 (base) + reduced cosmetics
    shadows: Simplified
    use_case: "Mid-range combat distance"

  LOD2_Low:
    distance: 40-80 meters
    triangles: 2000 (base) + minimal cosmetics
    shadows: None
    use_case: "Distant players, storm edge"

  LOD3_Culled:
    distance: 80+ meters
    triangles: 500 (billboard)
    shadows: None
    use_case: "Extreme distance, spectator mode"

  transition_settings:
    blend_distance: 2 meters
    hysteresis: 3 meters  # Prevents LOD popping
```

### 4.3 Texture Specifications

```yaml
texture_specifications:
  base_character:
    diffuse_albedo:
      resolution: 1024x1024
      format: "ASTC 6x6 (mobile) / BC7 (desktop)"
      channels: "RGB"
      description: "Base color with baked ambient occlusion"

    normal_map:
      resolution: 1024x1024
      format: "ASTC 6x6 / BC5"
      channels: "RG (tangent space)"
      description: "Surface detail, wrinkles, seams"

    mask_map:
      resolution: 512x512
      format: "ASTC 6x6 / BC7"
      channels:
        R: "Metallic"
        G: "Roughness"
        B: "Ambient Occlusion"
        A: "Emission mask (for glowing elements)"

  texture_atlasing:
    shared_atlas_size: 2048x2048
    atlas_contents:
      - "Skin tones (multiple)"
      - "Eye colors"
      - "Base clothing"
      - "Common accessories"
    description: "Single draw call for base character"

  cosmetic_textures:
    simple_cosmetics:
      resolution: 512x512
      format: "ASTC 6x6"

    complex_cosmetics:
      resolution: 1024x1024
      format: "ASTC 6x6"

    legendary_cosmetics:
      resolution: 1024x1024 + 512x512 emission
      format: "ASTC 6x6"
      notes: "May include animated textures"
```

### 4.4 UV Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        UV Layout Specification                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  UV Channel 0 (Main Textures):                                          │
│  ┌─────────────────────────────────────────────────────────┐            │
│  │                                                         │            │
│  │   ┌─────────┐   ┌─────────────────┐   ┌───────────┐    │            │
│  │   │         │   │                 │   │           │    │            │
│  │   │  HEAD   │   │     TORSO       │   │   ARMS    │    │            │
│  │   │  25%    │   │      35%        │   │    20%    │    │            │
│  │   │         │   │                 │   │           │    │            │
│  │   └─────────┘   └─────────────────┘   └───────────┘    │            │
│  │                                                         │            │
│  │   ┌───────────────────────┐   ┌─────────────────┐      │            │
│  │   │                       │   │                 │      │            │
│  │   │        LEGS           │   │     HANDS/      │      │            │
│  │   │         15%           │   │     MISC 5%     │      │            │
│  │   │                       │   │                 │      │            │
│  │   └───────────────────────┘   └─────────────────┘      │            │
│  │                                                         │            │
│  └─────────────────────────────────────────────────────────┘            │
│                                                                          │
│  UV Channel 1 (Lightmap - if needed):                                   │
│  - Non-overlapping UVs for baked lighting                               │
│  - Auto-generated or manually optimized                                  │
│                                                                          │
│  UV Guidelines:                                                          │
│  ├── No UV stretching > 15%                                             │
│  ├── Consistent texel density (10 pixels per unit)                      │
│  ├── Mirror UVs where possible (arms, legs)                             │
│  ├── Seams hidden in natural folds                                      │
│  └── 2 pixel padding between UV islands                                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Skeleton Rig

### 5.1 Bone Hierarchy

```
Root
└── Pelvis
    ├── Spine_01
    │   └── Spine_02
    │       └── Spine_03
    │           ├── Neck
    │           │   └── Head
    │           │       ├── Jaw (optional)
    │           │       ├── Eye_L
    │           │       ├── Eye_R
    │           │       └── Hat_Attach
    │           │
    │           ├── Clavicle_L
    │           │   └── Shoulder_L
    │           │       └── Elbow_L
    │           │           └── Wrist_L
    │           │               └── Hand_L
    │           │                   ├── Thumb_01_L → Thumb_02_L
    │           │                   ├── Index_01_L → Index_02_L
    │           │                   ├── Middle_01_L → Middle_02_L
    │           │                   ├── Ring_01_L → Ring_02_L
    │           │                   └── Pinky_01_L → Pinky_02_L
    │           │
    │           ├── Clavicle_R
    │           │   └── [Mirror of Left Arm]
    │           │
    │           └── Back_Attach
    │
    ├── Thigh_L
    │   └── Knee_L
    │       └── Ankle_L
    │           └── Foot_L
    │               └── Toe_L
    │
    └── Thigh_R
        └── [Mirror of Left Leg]

Total Bones: 52 (within mobile budget of 40-60)
```

### 5.2 Bone Specifications

```yaml
skeleton_specifications:
  total_bones: 52

  bone_groups:
    spine:
      count: 4
      bones: ["Pelvis", "Spine_01", "Spine_02", "Spine_03"]
      purpose: "Core body movement, breathing"

    head:
      count: 5
      bones: ["Neck", "Head", "Jaw", "Eye_L", "Eye_R"]
      purpose: "Look direction, expressions"

    arms_per_side:
      count: 4
      bones: ["Clavicle", "Shoulder", "Elbow", "Wrist"]
      purpose: "Arm movement, weapon handling"

    hands_per_side:
      count: 11
      bones: ["Hand", "Thumb_01/02", "Index_01/02", "Middle_01/02", "Ring_01/02", "Pinky_01/02"]
      purpose: "Grip weapons, gestures"

    legs_per_side:
      count: 4
      bones: ["Thigh", "Knee", "Ankle", "Foot", "Toe"]
      purpose: "Locomotion"

    attachment:
      count: 3
      bones: ["Hat_Attach", "Back_Attach", "Belt_Attach"]
      purpose: "Cosmetic attachment points"

  ik_targets:
    - name: "IK_Foot_L"
      purpose: "Ground contact"
    - name: "IK_Foot_R"
      purpose: "Ground contact"
    - name: "IK_Hand_L"
      purpose: "Two-handed weapons"
    - name: "IK_Hand_R"
      purpose: "Weapon holding"

  bone_constraints:
    max_influences_per_vertex: 4
    weight_threshold: 0.01  # Ignore weights below this
```

### 5.3 Skinning Guidelines

```yaml
skinning_guidelines:
  general:
    - Maximum 4 bone influences per vertex
    - Normalize all weights to sum to 1.0
    - Avoid micro-weights (< 0.01)

  problem_areas:
    shoulder:
      primary_bones: ["Clavicle", "Shoulder"]
      blend_region: "Shoulder cap"
      notes: "Add corrective blend shapes if needed"

    elbow:
      primary_bones: ["Shoulder", "Elbow"]
      blend_region: "Inner elbow crease"
      notes: "Ensure clean deformation at 90° bend"

    hip:
      primary_bones: ["Pelvis", "Thigh"]
      blend_region: "Hip crease"
      notes: "Critical for walking animation"

    wrist:
      primary_bones: ["Elbow", "Wrist", "Hand"]
      blend_region: "Wrist joint"
      notes: "Must support rotation without collapsing"

  testing_poses:
    - "T-Pose (default)"
    - "A-Pose (relaxed)"
    - "Arms fully raised"
    - "Full squat"
    - "Running extreme"
    - "Weapon swing"
```

---

## 6. Blend Shapes (Optional)

### 6.1 Facial Expressions

```yaml
blend_shapes:
  enabled: true
  target_platform: "High/Medium quality settings only"

  expression_set:
    base_expressions:
      - name: "Smile"
        intensity_range: [0, 1]
        affects: ["Mouth corners up", "Cheek raise"]

      - name: "Frown"
        intensity_range: [0, 1]
        affects: ["Mouth corners down", "Brow furrow"]

      - name: "Surprised"
        intensity_range: [0, 1]
        affects: ["Brow raise", "Eyes wide", "Mouth open"]

      - name: "Angry"
        intensity_range: [0, 1]
        affects: ["Brow furrow", "Nose scrunch"]

    phonemes:  # For potential voice chat or emotes
      - "AA"   # "father"
      - "EE"   # "see"
      - "OO"   # "moon"
      - "OH"   # "go"
      - "MM"   # closed lips

    eye_control:
      - "Blink_L"
      - "Blink_R"
      - "Squint_L"
      - "Squint_R"

  performance_settings:
    max_active_blendshapes: 4
    update_frequency: "Every frame for player, every 3rd frame for others"
    lod_disable_distance: 15  # Disable blend shapes beyond this
```

---

## 7. Material System

### 7.1 Shader Configuration

```yaml
character_shader:
  name: "Plunderstorm/Character"
  type: "Lit - Mobile Optimized"

  features:
    - Toon shading (stepped lighting)
    - Outline rendering (optional per quality)
    - Subsurface scattering (skin, simplified)
    - Emission support (glowing effects)
    - Color tinting (team colors, damage flash)

  inputs:
    _BaseColor: "RGB - Albedo"
    _NormalMap: "RG - Normal"
    _MaskMap: "RGBA - M/R/AO/E"
    _TintColor: "RGB - Runtime tint"
    _OutlineColor: "RGB - Outline"
    _OutlineWidth: "Float - 0.001-0.005"
    _EmissionColor: "RGB - Glow color"
    _EmissionIntensity: "Float - 0-10"

  quality_tiers:
    high:
      features: ["Full normal maps", "Outline", "SSS", "Emission"]
      shader_keywords: ["_NORMALMAP", "_OUTLINE", "_SSS", "_EMISSION"]

    medium:
      features: ["Simplified normals", "No outline", "No SSS", "Emission"]
      shader_keywords: ["_NORMALMAP", "_EMISSION"]

    low:
      features: ["No normal maps", "No outline", "No SSS", "Simple emission"]
      shader_keywords: ["_EMISSION_SIMPLE"]
```

### 7.2 Material Instances

```yaml
material_instances:
  skin_materials:
    - name: "Skin_Light"
      base_color: "#FFE0BD"
      subsurface_color: "#FF8080"

    - name: "Skin_Medium"
      base_color: "#D4A574"
      subsurface_color: "#CC6060"

    - name: "Skin_Dark"
      base_color: "#8D5524"
      subsurface_color: "#994040"

    - name: "Skin_Fantasy_Blue"
      base_color: "#7FB3D5"
      subsurface_color: "#4080CC"

    - name: "Skin_Fantasy_Green"
      base_color: "#7DCEA0"
      subsurface_color: "#40CC80"

  clothing_materials:
    - name: "Cloth_Linen"
      roughness: 0.8
      metallic: 0.0

    - name: "Cloth_Leather"
      roughness: 0.6
      metallic: 0.1

    - name: "Metal_Gold"
      roughness: 0.3
      metallic: 1.0
      base_color: "#FFD700"

    - name: "Metal_Steel"
      roughness: 0.4
      metallic: 1.0
      base_color: "#C0C0C0"
```

---

## 8. Performance Optimization

### 8.1 Batching Strategy

```yaml
batching_strategy:
  static_batching:
    enabled: false
    reason: "Characters are dynamic"

  dynamic_batching:
    enabled: true
    max_vertices: 300
    use_case: "Small accessories, particles"

  gpu_instancing:
    enabled: true
    use_case: "Multiple characters with same cosmetics"
    requirements:
      - Same mesh
      - Same material
      - Per-instance properties only

  srp_batcher:
    enabled: true
    use_case: "Primary batching method"
    requirements:
      - Compatible shaders
      - Constant buffer layout
```

### 8.2 Draw Call Budget

```yaml
draw_call_budget:
  per_character:
    target: 2-4 draw calls
    breakdown:
      body: 1
      head_accessory: 1
      weapon: 1
      special_effects: 1 (optional)

  scene_total:
    60_characters:
      max_draw_calls: 240
      instanced_savings: "~50% with common cosmetics"
      target: 120-150 draw calls

  optimization_techniques:
    - "Texture atlasing for base character"
    - "Material sharing where possible"
    - "LOD-based material simplification"
    - "GPU instancing for duplicates"
```

### 8.3 Memory Budget

```yaml
memory_budget:
  per_character:
    mesh_data:
      high_lod: 320 KB
      medium_lod: 160 KB
      low_lod: 80 KB

    textures:
      base_textures: 2.7 MB  # 1024 + 1024 + 512 compressed
      cosmetic_avg: 1.3 MB

    skeleton:
      bone_data: 8 KB
      animation_controller: 16 KB

    total_per_character: ~4.5 MB

  scene_total_60_characters:
    unique_meshes: ~30 (shared via instancing)
    estimated_total: 135 MB
    target_limit: 200 MB
```

### 8.4 Performance Targets

```yaml
performance_targets:
  frame_rate:
    target: 60 FPS
    minimum: 30 FPS

  test_scenarios:
    scenario_1:
      description: "60 characters on screen, combat"
      target_fps: 60
      test_devices: ["iPhone 12", "Samsung S21", "Pixel 6"]

    scenario_2:
      description: "Player close-up (menu, emote)"
      target_fps: 60
      lod_level: "LOD0 only"

    scenario_3:
      description: "Mass combat, 20+ players visible"
      target_fps: 45+
      lod_distribution: "Mixed LOD0-LOD2"

  profiling_metrics:
    - "Character skinning time < 2ms"
    - "Character rendering time < 4ms"
    - "Animation update < 1ms per character"
```

---

## 9. Asset Pipeline

### 9.1 Export Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Asset Export Pipeline                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  DCC Tool (Maya/Blender)                                                │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │  1. Model in T-Pose                                               │   │
│  │  2. Apply modular naming convention                               │   │
│  │  3. Verify poly counts per LOD                                    │   │
│  │  4. Check UV layout                                               │   │
│  │  5. Validate skinning (4 influences max)                          │   │
│  │  6. Test deformation with animation                               │   │
│  └────────────────────────────┬─────────────────────────────────────┘   │
│                               │                                          │
│                               ▼                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                        Export as FBX                              │   │
│  │  Settings:                                                        │   │
│  │  ├── FBX 2020                                                    │   │
│  │  ├── Binary format                                               │   │
│  │  ├── Include: Mesh, Skeleton, BlendShapes                        │   │
│  │  ├── Scale: 1 unit = 1 meter                                     │   │
│  │  └── Axis: Y-Up, -Z Forward                                      │   │
│  └────────────────────────────┬─────────────────────────────────────┘   │
│                               │                                          │
│                               ▼                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                      Unity Import                                 │   │
│  │  Import Settings:                                                 │   │
│  │  ├── Scale Factor: 1                                             │   │
│  │  ├── Mesh Compression: Low                                       │   │
│  │  ├── Read/Write: Disabled (enabled for cloth)                    │   │
│  │  ├── Optimize Mesh: True                                         │   │
│  │  ├── Generate Colliders: False                                   │   │
│  │  ├── Animation Type: Humanoid                                    │   │
│  │  └── Avatar: Create from this model                              │   │
│  └────────────────────────────┬─────────────────────────────────────┘   │
│                               │                                          │
│                               ▼                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                   Validation Script                               │   │
│  │  Automated Checks:                                                │   │
│  │  ├── Poly count within budget                                    │   │
│  │  ├── Bone count matches template                                 │   │
│  │  ├── UV channels present                                         │   │
│  │  ├── Material slots named correctly                              │   │
│  │  └── No import warnings/errors                                   │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Naming Conventions

```yaml
naming_conventions:
  mesh_naming:
    format: "CH_{BodyType}_{Part}_{LOD}"
    examples:
      - "CH_TypeA_Body_LOD0"
      - "CH_TypeA_Head_LOD1"
      - "CH_TypeB_Arms_LOD2"

  bone_naming:
    format: "{BoneName}_{Side}"
    examples:
      - "Spine_01"
      - "Shoulder_L"
      - "Hand_R"

  material_naming:
    format: "M_CH_{MaterialType}_{Variant}"
    examples:
      - "M_CH_Skin_Light"
      - "M_CH_Cloth_Leather"

  texture_naming:
    format: "T_CH_{Part}_{Type}"
    examples:
      - "T_CH_Body_Albedo"
      - "T_CH_Body_Normal"
      - "T_CH_Body_Mask"

  blend_shape_naming:
    format: "BS_{Expression}"
    examples:
      - "BS_Smile"
      - "BS_Blink_L"
```

---

## 10. Quality Assurance

### 10.1 Validation Checklist

```yaml
validation_checklist:
  mesh_validation:
    - [ ] Poly count within LOD budgets
    - [ ] No n-gons (all quads/tris)
    - [ ] No non-manifold geometry
    - [ ] No overlapping vertices
    - [ ] Clean edge flow for deformation
    - [ ] Symmetrical where appropriate

  uv_validation:
    - [ ] No overlapping UVs (except intentional mirrors)
    - [ ] No UV stretching > 15%
    - [ ] Proper padding between islands
    - [ ] Consistent texel density

  rig_validation:
    - [ ] Matches bone template exactly
    - [ ] Proper bone orientations
    - [ ] No scale on bones
    - [ ] IK handles functional
    - [ ] Attachment points positioned

  skinning_validation:
    - [ ] Max 4 influences per vertex
    - [ ] No zero-weight vertices
    - [ ] Clean deformation at all joints
    - [ ] Tested with animation set

  performance_validation:
    - [ ] Renders at 60 FPS with 60 instances
    - [ ] Memory within budget
    - [ ] LOD transitions smooth
    - [ ] No visual artifacts
```

### 10.2 Test Animations

```yaml
test_animations:
  deformation_tests:
    - name: "T-Pose"
      purpose: "Default pose verification"

    - name: "Full_Arm_Raise"
      purpose: "Shoulder deformation"

    - name: "Deep_Squat"
      purpose: "Hip and knee deformation"

    - name: "Arm_Twist"
      purpose: "Forearm twist deformation"

    - name: "Run_Cycle"
      purpose: "Full body locomotion"

    - name: "Combat_Swing"
      purpose: "Extreme poses, weapon handling"

  cosmetic_tests:
    - name: "All_Slots_Equipped"
      purpose: "Verify no clipping with full cosmetics"

    - name: "Animation_With_Cape"
      purpose: "Secondary motion compatibility"
```

---

## 11. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|--------------|---------------------|
| Base mesh clean topology | Quad-based, proper edge flow | Visual inspection, deformation test |
| Modular attachment points | 9 slots defined with bone parents | Unity scene verification |
| UV mapped efficiently | Shared 1024x1024 atlas | UV checker texture |
| Rig with 40-60 bones | 52 bones total | Bone count in Unity |
| Blend shapes for expressions | 15 blend shapes | Blend shape test scene |
| 60 FPS with 60 characters | < 16.6ms frame time | Profiler on target devices |
| LOD variants | High/Medium/Low/Culled | LOD preview in engine |
| Silhouette readable | Distinct at 50+ meters | Distance test scene |

---

## 12. Appendix

### 12.1 Reference Materials

- Visual Style Guide: [docs/VISUAL_STYLE_GUIDE.md](docs/VISUAL_STYLE_GUIDE.md)
- Technical Art Specs: [docs/TECHNICAL_ART_SPECIFICATIONS.md](docs/TECHNICAL_ART_SPECIFICATIONS.md)
- Animation Requirements: See ART-021

### 12.2 Tool Requirements

| Tool | Version | Purpose |
|------|---------|---------|
| Maya/Blender | 2023+/3.6+ | Modeling, rigging |
| Substance Painter | 2023+ | Texturing |
| Unity | 2022.3 LTS | Engine import |
| Marmoset | 4+ | Baking, preview |

### 12.3 File Deliverables

```yaml
deliverables:
  source_files:
    - "CH_Base_TypeA.ma" (Maya scene)
    - "CH_Base_TypeA.blend" (Blender backup)
    - "CH_Base_Textures.spp" (Substance project)

  export_files:
    - "CH_TypeA_Body_LOD0.fbx"
    - "CH_TypeA_Body_LOD1.fbx"
    - "CH_TypeA_Body_LOD2.fbx"
    - "T_CH_Body_Albedo.png"
    - "T_CH_Body_Normal.png"
    - "T_CH_Body_Mask.png"

  unity_assets:
    - "Characters/Base/CH_TypeA.prefab"
    - "Characters/Base/Materials/M_CH_Skin_*.mat"
    - "Characters/Base/Textures/*"
```
