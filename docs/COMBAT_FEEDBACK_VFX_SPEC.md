# Combat Feedback VFX Specification

## Document Information
- **Task ID:** ART-014
- **Priority:** P0
- **Complexity:** S (Small)
- **Dependencies:** ART-013 (Ability VFX Set), GAME-012 (Damage and Health System)
- **Last Updated:** 2025-12-01

---

## 1. Overview

### 1.1 Purpose
This specification defines the visual effects for combat feedback in Plunderstorm Mobile. These VFX communicate critical gameplay information about damage dealt, damage received, kills, deaths, and status changes.

### 1.2 Scope

```yaml
vfx_scope:
  damage_feedback:
    - Hit markers (dealing damage)
    - Damage numbers (floating text)
    - Impact effects (on target)
    - Directional damage indicators (receiving damage)

  status_feedback:
    - Critical hit effects
    - Kill confirmation
    - Death effects
    - Level up effects
    - Heal received effects
    - Shield/buff indicators

  ui_effects:
    - Low health warning
    - Screen damage vignette
    - Kill feed highlights
```

### 1.3 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Immediate Feedback | Players instantly know when they hit/miss | Critical |
| Clear Communication | Damage amount and type readable | Critical |
| Non-Obstructive | VFX don't block gameplay view | Critical |
| Satisfying | Combat feels impactful and rewarding | High |
| Consistent | Same feedback across all damage sources | High |
| Performant | No frame drops during intense combat | High |

---

## 2. Hit Marker System

### 2.1 Standard Hit Marker

```yaml
hit_marker:
  description: "Crosshair indicator when player deals damage"
  position: "Screen center (crosshair location)"

  visual:
    shape: "X pattern with 4 lines"
    size: "30x30 pixels (scalable)"
    color:
      normal_hit: "#FFFFFF"
      critical_hit: "#FFD700"
      headshot: "#FF0000"

    animation:
      appear: "Instant (1 frame)"
      hold: "0.1s"
      fade: "0.15s"
      total_duration: 0.25s

    scaling:
      on_hit: "105% scale pop"
      ease: "Ease out"

  variants:
    normal:
      lines: 4
      gap: "8px center gap"
      thickness: "2px"
      color: "#FFFFFF"

    critical:
      lines: 4
      gap: "8px center gap"
      thickness: "3px"
      color: "#FFD700"
      extra: "Star burst behind"

    kill:
      lines: 4
      gap: "0px (closed X)"
      thickness: "3px"
      color: "#FF0000"
      extra: "Skull icon flash"

  stacking:
    rapid_hits: "Each hit refreshes duration"
    multi_target: "Multiple markers allowed"
    max_concurrent: 5

  audio_sync:
    normal_hit: "hit_marker_tick"
    critical: "hit_marker_crit"
    kill: "hit_marker_kill"
```

### 2.2 Hit Marker Variations

```yaml
hit_marker_by_damage_type:
  physical:
    color: "#FFFFFF"
    shape: "Standard X"

  fire:
    color: "#FF6600"
    shape: "Standard X"
    extra: "Small ember particles"

  lightning:
    color: "#FFD700"
    shape: "Standard X"
    extra: "Electric crackle"

  poison:
    color: "#32CD32"
    shape: "Standard X"
    extra: "Toxic drip"

  healing:
    color: "#7CFC00"
    shape: "Plus sign (+)"
    context: "When healing allies (future)"
```

---

## 3. Damage Numbers

### 3.1 Floating Damage Text

```yaml
damage_numbers:
  description: "Numbers showing damage dealt floating from target"
  position: "Above damaged target, offset randomly"

  text_style:
    font: "Bold sans-serif (game UI font)"
    outline: "2px black outline"
    shadow: "Subtle drop shadow"

  size_scaling:
    base_damage: "Font size 24"
    scale_factor: "log(damage) * 1.2"
    min_size: 20
    max_size: 48

  color_coding:
    normal_damage: "#FFFFFF"
    critical_damage: "#FFD700"
    fire_damage: "#FF6600"
    lightning_damage: "#FFFF00"
    poison_damage: "#32CD32"
    true_damage: "#FF00FF"
    healing: "#7CFC00"
    blocked: "#808080"

  animation:
    spawn:
      offset_x: "Random -20 to +20 pixels"
      offset_y: "0"

    motion:
      direction: "Up and slightly outward"
      distance: "40 pixels"
      duration: 0.8s
      ease: "Ease out cubic"

    fade:
      start: "0.5s"
      duration: "0.3s"
      end_opacity: 0

    scale_pop:
      initial: 0.5
      peak: 1.2
      final: 1.0
      peak_time: 0.1s

  critical_hit_enhancement:
    size_multiplier: 1.5
    color: "#FFD700"
    animation: "Bounce + shake"
    extra_text: "CRIT!" (optional)
    particles: "Gold sparkles"

  stacking_behavior:
    rapid_damage: "Stack vertically"
    max_visible: 8
    combine_threshold: "Same source within 0.1s"
    combined_display: "Sum with '+' indicator"

  performance:
    pooled: true
    pool_size: 20
    billboard: true
```

### 3.2 Damage Number Examples

```
Standard hit (50 damage):
  "50" - White, normal size, floats up

Critical hit (120 damage):
  "120" - Gold, 1.5x size, bounces, sparkles

Poison tick (15 damage):
  "15" - Green, small, subtle float

Healing (80 health):
  "+80" - Bright green, plus prefix

Blocked/Immune (0 damage):
  "BLOCKED" or "IMMUNE" - Gray, no float
```

---

## 4. Impact Effects

### 4.1 Hit Impact VFX (On Target)

```yaml
hit_impact:
  description: "Visual effect on the damaged target"

  physical_hit:
    effect: "Spark burst"
    particles: 8
    color: "#FFFF00"
    spread: "Radial from hit point"
    duration: 0.2s
    size: 0.3m

  energy_hit:
    effect: "Magic ripple"
    shape: "Circular wave"
    color: "Matches damage type"
    duration: 0.25s
    size: 0.5m

  critical_hit:
    base_effect: "Enhanced version of normal"
    additions:
      - "Screen edge flash (subtle)"
      - "Extra particle burst"
      - "Impact ring expansion"
    particle_multiplier: 2.0
    duration: 0.35s

  coin_burst:
    description: "Stylized blood substitute"
    trigger: "On significant damage (>20% HP)"
    particles:
      type: "Gold coins"
      count: 5-10
      physics: true
      bounce: true
      lifetime: 1.5s
      collect: false (decorative only)

  hit_flash:
    target_flash:
      enabled: true
      color: "#FF0000"
      intensity: 0.3
      duration: 0.08s

    additive_overlay:
      enabled: true
      color: "#FFFFFF"
      opacity: 0.2
      duration: 0.05s
```

### 4.2 Elemental Impact Variants

```yaml
elemental_impacts:
  fire:
    base_effect: "Flame burst"
    color: "#FF4500"
    particles: "Embers"
    decal: "Scorch mark (0.5s)"
    audio: "fire_hit"

  lightning:
    base_effect: "Electric arc"
    color: "#FFD700"
    particles: "Sparks"
    chain_visual: "Arc to nearby metal"
    audio: "electric_hit"

  poison:
    base_effect: "Toxic splash"
    color: "#32CD32"
    particles: "Bubbles"
    lingering: "Green mist (0.3s)"
    audio: "poison_hit"

  ice:
    base_effect: "Frost burst"
    color: "#87CEEB"
    particles: "Ice shards"
    surface: "Frost patch (0.5s)"
    audio: "ice_hit"
```

---

## 5. Damage Received Feedback

### 5.1 Directional Damage Indicator

```yaml
directional_indicator:
  description: "Shows direction damage came from"
  position: "Screen edges"

  visual:
    shape: "Arrow/wedge pointing to damage source"
    size: "60x30 pixels"
    color:
      normal: "#FF0000"
      critical: "#FF0000" (larger)

    placement:
      distance_from_edge: "10% screen"
      rotation: "Points toward damage source"

  animation:
    appear: "Instant"
    hold: "0.3s"
    fade: "0.2s"
    pulse: "Once on appear"

  intensity_scaling:
    low_damage: "30% opacity"
    medium_damage: "60% opacity"
    high_damage: "100% opacity"
    critical: "100% + screen shake"

  multiple_sources:
    max_indicators: 4
    stacking: "Combine if same direction (±30°)"
```

### 5.2 Screen Damage Effects

```yaml
screen_damage_effects:
  vignette:
    description: "Red edges on screen when damaged"

    intensity_by_damage:
      light: "5% screen coverage"
      medium: "15% screen coverage"
      heavy: "30% screen coverage"

    color: "#FF000080" (red, 50% alpha)

    animation:
      fade_in: "Instant"
      hold: "0.2s"
      fade_out: "0.3s"

  screen_shake:
    trigger: "Damage > 30% max HP"

    parameters:
      intensity: "Scaled by damage"
      duration: 0.15s
      decay: "Quick falloff"

    limits:
      max_shake: "10 pixels displacement"
      cooldown: "0.5s between shakes"

  chromatic_aberration:
    trigger: "Critical hits received"
    intensity: 0.3
    duration: 0.2s
    fade: "Ease out"

  blood_splatter:
    enabled: false
    alternative: "Gold coin scatter on screen edges"
    particles: 3-5
    duration: 0.5s
```

### 5.3 Low Health Warning

```yaml
low_health_warning:
  trigger: "HP < 30%"

  visual:
    vignette:
      color: "#FF0000"
      opacity: "Pulsing 20%-40%"
      pulse_speed: "1.5s cycle"
      coverage: "15% edges"

    heartbeat_effect:
      enabled: true
      scale_pulse: "2% screen zoom"
      sync: "With audio heartbeat"

  critical_health:
    trigger: "HP < 15%"
    vignette_opacity: "30%-50%"
    pulse_speed: "1.0s cycle"
    intensity: "More pronounced"

  audio:
    heartbeat: "heartbeat_loop"
    volume_scale: "Louder as HP decreases"
```

---

## 6. Kill and Death Effects

### 6.1 Kill Confirmation

```yaml
kill_confirmation:
  description: "Feedback when player eliminates enemy"

  hit_marker:
    shape: "Closed X (kill marker)"
    color: "#FF0000"
    size: "1.5x normal"
    duration: 0.4s
    animation: "Scale pop + hold"

  screen_effect:
    flash:
      color: "#FFFFFF"
      opacity: 0.1
      duration: 0.1s

    zoom:
      enabled: false (optional setting)
      amount: "102%"
      duration: 0.2s

  text_popup:
    enabled: true
    text: "ELIMINATED"
    position: "Center screen, below crosshair"
    color: "#FF0000"
    size: "Large"
    duration: 1.5s
    animation: "Fade in, hold, fade out"

  victim_name:
    display: true
    position: "Below ELIMINATED text"
    format: "[PlayerName]"
    duration: 1.5s

  xp_reward:
    display: true
    format: "+100 XP"
    color: "#FFD700"
    position: "Below victim name"

  audio:
    sound: "kill_confirmation"
    priority: "High"
```

### 6.2 Death Effect (Enemy)

```yaml
enemy_death_effect:
  description: "VFX when enemy player dies"

  visual:
    poof_effect:
      type: "Magical disappearance"
      color_primary: "#9B59B6"
      color_secondary: "#FFD700"
      duration: 0.5s

      particles:
        type: "Sparkle burst"
        count: 30
        spread: "Spherical outward"
        lifetime: 1.0s

      ghost:
        enabled: true
        type: "Silhouette rises"
        color: "#FFFFFF40"
        rise_distance: 2m
        duration: 0.8s
        fade: "Gradual"

    coin_explosion:
      enabled: true
      count: 15-25
      physics: true
      scatter_radius: 3m
      collect: false (decorative)
      lifetime: 2s

    loot_drop:
      visual: "Ability orbs spawn"
      glow: true
      animation: "Pop out from death point"

  ground_effect:
    decal: "Scorch/impact mark"
    duration: 5s
    fade: 1s

  audio:
    death_sound: "player_death_poof"
    coin_scatter: "coins_scatter"
```

### 6.3 Player Death (Self)

```yaml
player_death_self:
  description: "Feedback when player dies"

  screen_effects:
    grayscale:
      enabled: true
      transition: 0.5s
      final_saturation: 0.2

    vignette:
      color: "#000000"
      opacity: 0.5
      coverage: "30%"

    blur:
      enabled: false (optional)
      intensity: "Slight"

  camera:
    behavior: "Pull back and up"
    distance: "3m back, 2m up"
    duration: 1.0s
    look_at: "Death position"

  text:
    primary: "YOU DIED"
    color: "#FF0000"
    size: "Extra large"
    animation: "Fade in with scale"
    position: "Center screen"

    killer_info:
      text: "Eliminated by [PlayerName]"
      position: "Below YOU DIED"
      delay: 0.5s

  spectate_prompt:
    delay: 2.0s
    text: "Tap to Spectate"
    position: "Bottom center"

  audio:
    death_sting: "player_death_self"
    music: "Transition to spectator"
```

---

## 7. Level Up and Progression

### 7.1 Level Up Effect

```yaml
level_up_effect:
  description: "VFX when player gains a level"
  trigger: "XP threshold reached"

  player_effect:
    aura:
      type: "Rising energy column"
      color: "#FFD700"
      height: 5m
      duration: 1.5s
      visible_to: "All players"

    particles:
      type: "Golden sparkles rising"
      count: 50
      lifetime: 2s
      spread: "Cylindrical around player"

    ring:
      type: "Expanding ground ring"
      color: "#FFD700"
      radius: 3m
      duration: 0.5s

    flash:
      color: "#FFFFFF"
      opacity: 0.3
      duration: 0.1s

  ui_effect:
    text:
      content: "LEVEL UP!"
      color: "#FFD700"
      size: "Large"
      position: "Center screen"
      animation: "Scale pop + glow"
      duration: 2s

    level_display:
      content: "Level [X]"
      position: "Below LEVEL UP"
      animation: "Fade in"

    stat_boost:
      display: true
      format: "+10 Max HP"
      color: "#7CFC00"

  audio:
    sound: "level_up_fanfare"
    priority: "High"
```

### 7.2 Ability Upgrade Effect

```yaml
ability_upgrade_effect:
  description: "VFX when ability ranks up"

  player_effect:
    glow:
      type: "Brief aura pulse"
      color: "Matches ability color"
      duration: 0.5s

    particles:
      type: "Ability-colored sparkles"
      count: 20
      duration: 1s

  ui_effect:
    ability_slot:
      animation: "Glow + pulse"
      border_color: "Upgrade tier color"
      duration: 1s

    text_popup:
      content: "[Ability] Rank [X]"
      color: "Tier color"
      position: "Above ability bar"
      duration: 1.5s

  tier_colors:
    rank_1: "#FFFFFF"
    rank_2: "#3498DB"
    rank_3: "#FFD700"

  audio:
    sound: "ability_upgrade"
```

---

## 8. Healing and Buff Effects

### 8.1 Heal Received Effect

```yaml
heal_received:
  description: "VFX when player receives healing"

  player_effect:
    particles:
      type: "Rising green crosses/plus signs"
      color: "#7CFC00"
      count: 10
      lifetime: 1s
      emit_from: "Player body"

    glow:
      type: "Brief body glow"
      color: "#7CFC00"
      intensity: 0.3
      duration: 0.5s

    ring:
      enabled: false (optional)
      color: "#7CFC00"
      radius: 1m

  damage_number:
    format: "+[amount]"
    color: "#7CFC00"
    position: "Above player"

  heal_over_time:
    tick_effect: "Small particle burst"
    tick_interval: "Match heal tick"
    intensity: "Reduced (30%)"

  audio:
    instant_heal: "heal_instant"
    heal_tick: "heal_tick"
```

### 8.2 Shield/Buff Indicators

```yaml
buff_indicators:
  description: "Visual indicators for active buffs"

  shield_buff:
    visual:
      type: "Hexagonal shield overlay"
      color: "#4169E1"
      opacity: 0.4
      animation: "Subtle shimmer"
      position: "Around player model"

    damage_absorption:
      flash_on_hit: true
      flash_color: "#FFFFFF"
      crack_effect: "At 50% remaining"
      break_effect: "Shatter particles"

  speed_buff:
    visual:
      type: "Motion lines attached to player"
      color: "#00CED1"
      count: 3-5
      length: 2m behind

    ground_effect:
      type: "Speed trail"
      duration: 0.3s

  damage_buff:
    visual:
      type: "Red aura"
      color: "#FF4500"
      opacity: 0.2
      pulse: true

    weapon_effect:
      type: "Weapon glow"
      color: "#FF4500"

  invulnerability:
    visual:
      type: "Golden shield sphere"
      color: "#FFD700"
      opacity: 0.5
      animation: "Rotating runes"

    audio:
      activate: "shield_activate"
      deactivate: "shield_fade"
```

---

## 9. Technical Specifications

### 9.1 Performance Budget

```yaml
performance_budget:
  hit_markers:
    max_concurrent: 5
    draw_calls: 1 (batched)

  damage_numbers:
    max_concurrent: 20
    pooled: true
    pool_size: 30
    draw_calls: 1 (batched text)

  impact_effects:
    max_concurrent: 10
    particles_per_impact: 15
    draw_calls: 2

  screen_effects:
    post_process_budget: "1ms GPU"
    vignette: "Shared material"
    shake: "Transform only (no render cost)"

  total_budget:
    target: "<2ms GPU for all combat VFX"
    stress_test: "6 players fighting, multiple hits"
```

### 9.2 Rendering Configuration

```yaml
rendering:
  hit_markers:
    render_mode: "Screen space overlay"
    layer: "UI"
    sort_order: 100

  damage_numbers:
    render_mode: "World space billboard"
    layer: "UI_World"
    sort_order: 50
    depth_test: false

  impact_effects:
    render_mode: "World space"
    layer: "VFX"
    depth_test: true

  screen_effects:
    render_mode: "Post-process"
    layer: "PostProcess"
    priority: "After game render"
```

### 9.3 Quality Settings

```yaml
quality_tiers:
  high:
    damage_numbers: true
    particle_impacts: true
    screen_effects: true
    coin_physics: true

  medium:
    damage_numbers: true
    particle_impacts: true
    screen_effects: true
    coin_physics: false

  low:
    damage_numbers: true
    particle_impacts: "Reduced (50%)"
    screen_effects: "Vignette only"
    coin_physics: false

  accessibility:
    option: "Reduce screen shake"
    option: "Disable screen flash"
    option: "Enlarge damage numbers"
```

---

## 10. Asset Deliverables

### 10.1 Prefab List

```yaml
prefabs:
  hit_markers:
    - "VFX_HitMarker_Normal.prefab"
    - "VFX_HitMarker_Critical.prefab"
    - "VFX_HitMarker_Kill.prefab"

  damage_numbers:
    - "VFX_DamageNumber.prefab"
    - "VFX_HealNumber.prefab"
    - "VFX_CriticalNumber.prefab"

  impact_effects:
    - "VFX_Impact_Physical.prefab"
    - "VFX_Impact_Fire.prefab"
    - "VFX_Impact_Lightning.prefab"
    - "VFX_Impact_Poison.prefab"
    - "VFX_Impact_Critical.prefab"

  screen_effects:
    - "VFX_Screen_Vignette.prefab"
    - "VFX_Screen_LowHealth.prefab"
    - "VFX_Screen_DirectionalDamage.prefab"

  death_effects:
    - "VFX_Death_Enemy.prefab"
    - "VFX_Death_Self.prefab"
    - "VFX_CoinExplosion.prefab"

  progression:
    - "VFX_LevelUp.prefab"
    - "VFX_AbilityUpgrade.prefab"

  buffs:
    - "VFX_Buff_Shield.prefab"
    - "VFX_Buff_Speed.prefab"
    - "VFX_Buff_Damage.prefab"
    - "VFX_Heal_Received.prefab"
```

### 10.2 Texture Atlas

```yaml
texture_atlas:
  combat_feedback_atlas:
    resolution: "256x256"
    format: "ASTC 6x6"
    contents:
      - "T_HitMarker_X"
      - "T_HitMarker_Kill"
      - "T_DirectionalArrow"
      - "T_Spark_Hit"
      - "T_Coin_Small"
      - "T_Ring_Impact"
      - "T_Plus_Heal"
```

---

## 11. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| Hit feedback is satisfying | Instant, clear hit markers | Playtest feedback |
| VFX don't obstruct view | No central screen blocking | Visual inspection |
| Critical hit clearly different | 1.5x size, gold color | Comparison test |
| Effects match art style | Pirate fantasy theme | Art Director review |
| Performance acceptable | <2ms GPU in combat | Profiler verification |
| Damage numbers readable | Clear font, contrast | Mobile device test |
| Direction indicators accurate | Point to damage source | Gameplay test |

---

## 12. Implementation Checklist

```yaml
checklist:
  hit_feedback:
    - [ ] Standard hit marker
    - [ ] Critical hit marker
    - [ ] Kill marker
    - [ ] Elemental variants

  damage_numbers:
    - [ ] Floating number system
    - [ ] Color coding by type
    - [ ] Critical hit enhancement
    - [ ] Pooling system

  damage_received:
    - [ ] Directional indicators
    - [ ] Screen vignette
    - [ ] Low health warning
    - [ ] Screen shake

  kills_deaths:
    - [ ] Kill confirmation
    - [ ] Enemy death effect
    - [ ] Player death sequence

  progression:
    - [ ] Level up effect
    - [ ] Ability upgrade effect

  buffs:
    - [ ] Shield indicator
    - [ ] Heal received
    - [ ] Buff/debuff visuals
```

---

## 13. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-01 | Initial specification |

---

*This document defines all combat feedback VFX required for satisfying and readable gameplay on mobile devices.*
