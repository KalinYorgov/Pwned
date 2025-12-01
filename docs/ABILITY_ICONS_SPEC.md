# Ability Icons Specification

## Document Information
- **Task ID:** ART-018
- **Priority:** P0
- **Complexity:** S (Small)
- **Dependencies:** ART-017 (UI Theme and Iconography), GAME-008 (Core Ability Implementation)
- **Last Updated:** 2025-12-01

---

## 1. Overview

### 1.1 Purpose
This specification defines the visual design for all ability icons in Plunderstorm Mobile. Ability icons must be instantly recognizable at small sizes on mobile screens while communicating the ability's function and element.

### 1.2 Scope

```yaml
icon_scope:
  mvp_abilities: 10
  launch_abilities: 20+
  icon_variants_per_ability:
    - Base icon
    - Rank 2 variant (border change)
    - Rank 3 variant (golden border)
    - Cooldown state
    - Disabled/locked state
```

### 1.3 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Instant Recognition | Identify ability at glance | Critical |
| Small Screen Readable | Clear at 44x44 pixel display | Critical |
| Element Communication | Color indicates damage type | High |
| Rank Distinction | Upgrade level visible | High |
| Consistency | Unified visual language | High |
| Pirate Theme | Fits game aesthetic | Medium |

---

## 2. Icon Design Standards

### 2.1 Icon Dimensions

```yaml
dimensions:
  base_size:
    design: "256x256 pixels"
    export: "128x128 pixels"

  display_sizes:
    ability_bar: "56x56 dp"
    ability_bar_mini: "44x44 dp"
    tooltip: "80x80 dp"
    shop_preview: "128x128 dp"

  safe_area:
    icon_content: "80% of icon area"
    padding: "10% on all sides"
    border_zone: "Outer 8%"
```

### 2.2 Icon Structure

```yaml
icon_structure:
  layers:
    background:
      shape: "Rounded square"
      corner_radius: "15%"
      purpose: "Element color base"

    inner_glow:
      type: "Radial gradient"
      position: "Center"
      purpose: "Depth and focus"

    symbol:
      position: "Center"
      size: "60-70% of icon"
      purpose: "Ability representation"

    border:
      width: "4% of icon"
      purpose: "Rank indication"

    shine:
      type: "Top highlight"
      opacity: "15-20%"
      purpose: "Polish and depth"

  visual_hierarchy:
    1: "Symbol (most prominent)"
    2: "Background color"
    3: "Border (rank indicator)"
    4: "Shine/polish"
```

### 2.3 Color Coding System

```yaml
element_colors:
  fire:
    background: "#CC3300"
    gradient_light: "#FF6600"
    gradient_dark: "#991100"
    symbol_fill: "#FFCC00"
    symbol_stroke: "#CC6600"

  lightning:
    background: "#CC9900"
    gradient_light: "#FFDD44"
    gradient_dark: "#996600"
    symbol_fill: "#FFFFFF"
    symbol_stroke: "#FFCC00"

  poison:
    background: "#228B22"
    gradient_light: "#44CC44"
    gradient_dark: "#116611"
    symbol_fill: "#99FF99"
    symbol_stroke: "#336633"

  physical:
    background: "#666666"
    gradient_light: "#999999"
    gradient_dark: "#333333"
    symbol_fill: "#CCCCCC"
    symbol_stroke: "#444444"

  utility:
    background: "#336699"
    gradient_light: "#5588BB"
    gradient_dark: "#224466"
    symbol_fill: "#99CCFF"
    symbol_stroke: "#336699"

  healing:
    background: "#339933"
    gradient_light: "#55BB55"
    gradient_dark: "#226622"
    symbol_fill: "#AAFFAA"
    symbol_stroke: "#338833"

  movement:
    background: "#0099CC"
    gradient_light: "#33BBEE"
    gradient_dark: "#006699"
    symbol_fill: "#99EEFF"
    symbol_stroke: "#0088BB"
```

### 2.4 Rank Border System

```yaml
rank_borders:
  rank_1:
    color: "#666666"
    material: "Basic iron"
    glow: "None"
    width: "4%"

  rank_2:
    color: "#3498DB"
    material: "Blue-tinted steel"
    glow: "Subtle blue"
    glow_intensity: 0.2
    width: "5%"
    inner_decoration: "Simple corners"

  rank_3:
    color: "#FFD700"
    material: "Gold with gems"
    glow: "Golden aura"
    glow_intensity: 0.4
    width: "6%"
    inner_decoration: "Ornate corners"
    gem_accents: true
    gem_color: "Match element"
```

---

## 3. MVP Ability Icons

### 3.1 Fireball

```yaml
fireball_icon:
  ability_type: "Offensive"
  element: "Fire"

  symbol:
    primary: "Stylized fireball"
    description: |
      Flaming sphere with trailing flames.
      Core is bright yellow/white fading to orange/red edges.
      Motion implied through shape (elongated trail).

    style:
      shape: "Sphere with 3-4 flame trails"
      direction: "Moving right/upward"
      core_color: "#FFFF00"
      mid_color: "#FF6600"
      outer_color: "#CC3300"

  background:
    base: "#CC3300"
    gradient: "Radial, lighter center"

  silhouette_test: "Recognizable fireball shape"

  visual_reference: "Classic RPG fireball spell"
```

### 3.2 Cutlass Slash

```yaml
cutlass_slash_icon:
  ability_type: "Offensive"
  element: "Physical"

  symbol:
    primary: "Pirate cutlass with slash arc"
    description: |
      Curved pirate sword shown mid-swing.
      White/silver slash arc behind blade.
      Blade positioned diagonally.

    style:
      sword: "Curved cutlass silhouette"
      arc: "Crescent slash trail"
      direction: "Slashing from top-right to bottom-left"
      blade_color: "#CCCCCC"
      arc_color: "#FFFFFF"
      handle_color: "#8B4513"

  background:
    base: "#666666"
    gradient: "Radial, metallic feel"

  silhouette_test: "Sword with motion arc"

  visual_reference: "Pirate cutlass"
```

### 3.3 Cannonball

```yaml
cannonball_icon:
  ability_type: "Offensive"
  element: "Physical"

  symbol:
    primary: "Cannon firing with ball"
    description: |
      Small cannon with cannonball in flight.
      Smoke/fire from cannon barrel.
      Impact lines showing power.

    style:
      cannon: "Compact ship cannon"
      ball: "Dark iron sphere"
      smoke: "Gray/white puff"
      fire: "Orange muzzle flash"
      direction: "Firing right"

  background:
    base: "#666666"
    gradient: "Radial, darker edges"

  silhouette_test: "Cannon with projectile"

  visual_reference: "Ship cannon"
```

### 3.4 Lightning Strike

```yaml
lightning_strike_icon:
  ability_type: "Offensive"
  element: "Lightning"

  symbol:
    primary: "Lightning bolt"
    description: |
      Classic zigzag lightning bolt.
      Striking downward with impact burst.
      Bright white core with yellow glow.

    style:
      bolt: "3-segment zigzag"
      direction: "Striking down"
      core_color: "#FFFFFF"
      glow_color: "#FFD700"
      impact: "Star burst at bottom"

  background:
    base: "#CC9900"
    gradient: "Radial, electric feel"

  silhouette_test: "Lightning bolt shape"

  visual_reference: "Classic lightning symbol"
```

### 3.5 Poison Mackerel

```yaml
poison_mackerel_icon:
  ability_type: "Offensive"
  element: "Poison"

  symbol:
    primary: "Toxic fish"
    description: |
      Fish silhouette with skull marking.
      Dripping with poison droplets.
      Slightly comical but deadly look.

    style:
      fish: "Mackerel shape"
      skull: "Small skull on fish body"
      drops: "2-3 poison drips"
      fish_color: "#228B22"
      skull_color: "#FFFFFF"
      drops_color: "#99FF00"

  background:
    base: "#228B22"
    gradient: "Radial, toxic glow"

  silhouette_test: "Fish with drips"

  visual_reference: "Cartoon toxic fish"
```

### 3.6 Grappling Hook

```yaml
grappling_hook_icon:
  ability_type: "Utility"
  element: "Movement"

  symbol:
    primary: "Hook with rope"
    description: |
      Metal grappling hook with trailing rope.
      Hook shown in flight motion.
      Rope curves behind showing trajectory.

    style:
      hook: "Three-pronged grapple"
      rope: "Curved trailing line"
      direction: "Flying up-right"
      hook_color: "#AAAAAA"
      rope_color: "#D2B48C"

  background:
    base: "#0099CC"
    gradient: "Radial, motion feel"

  silhouette_test: "Hook with rope trail"

  visual_reference: "Pirate boarding hook"
```

### 3.7 Barrel Roll

```yaml
barrel_roll_icon:
  ability_type: "Utility"
  element: "Movement"

  symbol:
    primary: "Rolling barrel"
    description: |
      Wooden barrel shown rolling.
      Motion lines indicating spin.
      Slight dust/speed effect.

    style:
      barrel: "Classic wooden barrel"
      motion: "Curved speed lines"
      direction: "Rolling right"
      barrel_color: "#8B4513"
      band_color: "#555555"
      motion_color: "#FFFFFF"

  background:
    base: "#0099CC"
    gradient: "Radial, dynamic"

  silhouette_test: "Barrel with motion"

  visual_reference: "Pirate barrel"
```

### 3.8 Healing Grog

```yaml
healing_grog_icon:
  ability_type: "Utility"
  element: "Healing"

  symbol:
    primary: "Glowing bottle"
    description: |
      Rum bottle with magical glow.
      Green healing aura around bottle.
      Plus symbol or sparkles nearby.

    style:
      bottle: "Classic rum bottle shape"
      liquid: "Glowing green"
      aura: "Soft green glow"
      sparkles: "2-3 healing sparkles"
      bottle_color: "#8B4513"
      liquid_color: "#66FF66"
      sparkle_color: "#FFFFFF"

  background:
    base: "#339933"
    gradient: "Radial, healthy glow"

  silhouette_test: "Bottle with glow"

  visual_reference: "Magical potion bottle"
```

### 3.9 Smoke Bomb

```yaml
smoke_bomb_icon:
  ability_type: "Utility"
  element: "Utility"

  symbol:
    primary: "Bomb with smoke"
    description: |
      Round bomb shape with smoke billowing.
      Lit fuse visible.
      Smoke cloud expanding.

    style:
      bomb: "Round with lit fuse"
      smoke: "Billowing cloud"
      fuse: "Sparking fuse"
      bomb_color: "#333333"
      smoke_color: "#888888"
      fuse_color: "#FF6600"

  background:
    base: "#336699"
    gradient: "Radial, smoky"

  silhouette_test: "Bomb with smoke cloud"

  visual_reference: "Ninja smoke bomb"
```

### 3.10 Wind Burst

```yaml
wind_burst_icon:
  ability_type: "Utility"
  element: "Utility"

  symbol:
    primary: "Spiral wind"
    description: |
      Spiral/circular wind pattern.
      Expanding outward motion.
      Speed lines radiating out.

    style:
      wind: "Spiral lines"
      motion: "Radiating outward"
      center: "Swirl focus point"
      wind_color: "#AADDFF"
      motion_color: "#FFFFFF"

  background:
    base: "#336699"
    gradient: "Radial, airy"

  silhouette_test: "Spiral wind pattern"

  visual_reference: "Wind gust symbol"
```

---

## 4. Icon States

### 4.1 State Variations

```yaml
icon_states:
  normal:
    description: "Default ready state"
    modifications: "None (base icon)"
    opacity: 1.0

  on_cooldown:
    description: "Ability cooling down"
    modifications:
      - "Grayscale overlay"
      - "Radial wipe reveal (clockwise)"
      - "Cooldown number overlay"
    grayscale: 0.8
    overlay_color: "#00000080"

  insufficient_resource:
    description: "Not enough mana/resource"
    modifications:
      - "Desaturated"
      - "Red tint pulse"
    saturation: 0.3
    tint: "#FF000020"

  disabled:
    description: "Cannot use (silenced, etc.)"
    modifications:
      - "Full grayscale"
      - "Lock icon overlay"
      - "Crossed out"
    grayscale: 1.0
    overlay: "Lock symbol"

  pressed:
    description: "Being activated"
    modifications:
      - "Scale down (95%)"
      - "Brightness increase"
    scale: 0.95
    brightness: 1.2

  highlighted:
    description: "Selected/targeted"
    modifications:
      - "Glowing border"
      - "Pulse animation"
    glow_color: "#FFFFFF"
    glow_intensity: 0.5
```

### 4.2 Cooldown Display

```yaml
cooldown_display:
  visual_method: "Radial wipe"

  wipe_animation:
    direction: "Clockwise"
    start_position: "12 o'clock"
    overlay_color: "#000000"
    overlay_opacity: 0.6

  number_display:
    position: "Center"
    font: "Bold, game UI font"
    size: "40% of icon height"
    color: "#FFFFFF"
    outline: "#000000, 2px"

    format:
      above_10s: "Integer (15)"
      below_10s: "One decimal (9.5)"
      below_1s: "One decimal (0.8)"

  ready_flash:
    enabled: true
    color: "#FFFFFF"
    duration: 0.2s
    type: "Border flash"
```

---

## 5. Technical Specifications

### 5.1 Export Requirements

```yaml
export_specs:
  format: "PNG-24 with alpha"

  sizes:
    primary:
      resolution: "128x128"
      use: "In-game ability bar"

    high_res:
      resolution: "256x256"
      use: "Shop, tooltips"

    atlas_size:
      resolution: "64x64"
      use: "Atlas packing"

  naming_convention:
    pattern: "Icon_Ability_[Name]_[Rank].png"
    examples:
      - "Icon_Ability_Fireball_R1.png"
      - "Icon_Ability_Fireball_R2.png"
      - "Icon_Ability_Fireball_R3.png"

  color_space: "sRGB"
  bit_depth: "8-bit per channel"
```

### 5.2 Atlas Configuration

```yaml
icon_atlas:
  atlas_name: "UI_AbilityIcons_Atlas"
  atlas_size: "1024x1024"
  icon_size: "128x128"
  padding: "2px"
  format: "ASTC 4x4"

  layout:
    icons_per_row: 8
    total_slots: 64
    arrangement: "By ability, then by rank"

  organization:
    row_1: "Offensive abilities (ranks 1-3)"
    row_2: "Offensive abilities continued"
    row_3: "Utility abilities (ranks 1-3)"
    row_4: "Utility abilities continued"
    row_5-6: "Reserved for future abilities"
    row_7-8: "State overlays and extras"
```

### 5.3 Memory Budget

```yaml
memory_budget:
  atlas_texture: "~1.33 MB (1024x1024 ASTC 4x4)"
  per_icon: "~0.02 MB"

  total_icons:
    mvp: "30 (10 abilities × 3 ranks)"
    launch: "60+ (20+ abilities × 3 ranks)"

  runtime:
    loaded: "Always (part of UI)"
    unload: "Never"
```

---

## 6. Asset Deliverables

### 6.1 Icon Files

```yaml
deliverables:
  offensive_abilities:
    - "Icon_Ability_Fireball_R1.png"
    - "Icon_Ability_Fireball_R2.png"
    - "Icon_Ability_Fireball_R3.png"
    - "Icon_Ability_CutlassSlash_R1.png"
    - "Icon_Ability_CutlassSlash_R2.png"
    - "Icon_Ability_CutlassSlash_R3.png"
    - "Icon_Ability_Cannonball_R1.png"
    - "Icon_Ability_Cannonball_R2.png"
    - "Icon_Ability_Cannonball_R3.png"
    - "Icon_Ability_LightningStrike_R1.png"
    - "Icon_Ability_LightningStrike_R2.png"
    - "Icon_Ability_LightningStrike_R3.png"
    - "Icon_Ability_PoisonMackerel_R1.png"
    - "Icon_Ability_PoisonMackerel_R2.png"
    - "Icon_Ability_PoisonMackerel_R3.png"

  utility_abilities:
    - "Icon_Ability_GrapplingHook_R1.png"
    - "Icon_Ability_GrapplingHook_R2.png"
    - "Icon_Ability_GrapplingHook_R3.png"
    - "Icon_Ability_BarrelRoll_R1.png"
    - "Icon_Ability_BarrelRoll_R2.png"
    - "Icon_Ability_BarrelRoll_R3.png"
    - "Icon_Ability_HealingGrog_R1.png"
    - "Icon_Ability_HealingGrog_R2.png"
    - "Icon_Ability_HealingGrog_R3.png"
    - "Icon_Ability_SmokeBomb_R1.png"
    - "Icon_Ability_SmokeBomb_R2.png"
    - "Icon_Ability_SmokeBomb_R3.png"
    - "Icon_Ability_WindBurst_R1.png"
    - "Icon_Ability_WindBurst_R2.png"
    - "Icon_Ability_WindBurst_R3.png"

  state_overlays:
    - "Icon_Overlay_Cooldown.png"
    - "Icon_Overlay_Locked.png"
    - "Icon_Overlay_Disabled.png"

  borders:
    - "Icon_Border_Rank1.png"
    - "Icon_Border_Rank2.png"
    - "Icon_Border_Rank3.png"
```

### 6.2 Source Files

```yaml
source_files:
  format: "PSD or AI"

  organization:
    master_file: "AbilityIcons_Master.psd"
    layers:
      - "Background templates"
      - "Symbol artwork"
      - "Border templates"
      - "State overlays"

  layer_organization:
    per_icon:
      - "Background gradient"
      - "Symbol"
      - "Shine overlay"
      - "Border"
      - "State layers (hidden)"
```

---

## 7. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| Each ability has unique icon | 10 distinct designs | Visual comparison |
| Icons readable at 44px | Clear at minimum size | Mobile device test |
| Element color coding clear | Fire=red, etc. | Color picker verification |
| Rank borders distinguishable | R1/R2/R3 visually different | Side-by-side comparison |
| Cooldown state clear | Grayed with number | Functional test |
| Style matches game UI | Pirate theme consistency | Art Director review |

---

## 8. Implementation Checklist

```yaml
checklist:
  design:
    - [ ] All 10 MVP icon symbols designed
    - [ ] Element colors applied
    - [ ] Rank 1 versions complete
    - [ ] Rank 2 borders added
    - [ ] Rank 3 golden borders added

  production:
    - [ ] High-res exports (256x256)
    - [ ] Standard exports (128x128)
    - [ ] Atlas-ready exports (64x64)
    - [ ] State overlay exports

  technical:
    - [ ] Atlas packed
    - [ ] Naming convention followed
    - [ ] Memory budget verified
    - [ ] Mobile tested at all sizes
```

---

## 9. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-01 | Initial specification |

---

*This document defines the complete ability icon design system for Plunderstorm Mobile, ensuring clear visual communication at mobile screen sizes.*
