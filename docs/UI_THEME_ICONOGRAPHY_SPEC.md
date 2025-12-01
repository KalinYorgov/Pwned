# UI Theme and Iconography Specification

## Document Information
- **Task ID:** ART-017
- **Priority:** P0
- **Complexity:** M (Medium)
- **Dependencies:** ART-001 (Visual Style Guide)
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the complete UI visual theme for Plunderstorm Mobile, establishing a cohesive pirate fantasy aesthetic across all interface elements while maintaining mobile usability and readability standards.

### 1.2 Scope
- Panel and frame styles
- Button designs and states
- Icon system and library
- Typography system
- Progress bars and indicators
- Popup and modal styles
- Color system for UI
- Animation and feedback

### 1.3 Goals
- Create immersive pirate-themed UI
- Ensure excellent mobile readability
- Support all screen sizes and densities
- Maintain consistent visual language
- Enable efficient UI development

---

## 2. Design Philosophy

### 2.1 UI Design Pillars

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        UI Design Pillars                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │   THEMATIC      │  │   READABLE      │  │   RESPONSIVE    │          │
│  │   IMMERSION     │  │   & CLEAR       │  │   & ALIVE       │          │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤          │
│  │ • Pirate motifs │  │ • High contrast │  │ • Touch-friendly│          │
│  │ • Weathered     │  │ • Clear icons   │  │ • Feedback      │          │
│  │   textures      │  │ • Legible text  │  │ • Animations    │          │
│  │ • Nautical      │  │ • Obvious       │  │ • State changes │          │
│  │   elements      │  │   hierarchy     │  │ • Micro-motion  │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
│                                                                          │
│  Balance: 40% Theme | 40% Usability | 20% Polish                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Visual Language

```yaml
visual_language:
  materials:
    primary: "Aged wood planks"
    secondary: "Weathered parchment"
    accent: "Brass/gold metal"
    highlight: "Rope and canvas"

  textures:
    wood_grain: "Visible but subtle"
    paper_texture: "Aged, slight stains"
    metal_patina: "Warm brass tones"
    fabric_weave: "Canvas/sailcloth"

  edges:
    frames: "Rounded wood corners"
    buttons: "Metal-capped edges"
    cards: "Torn parchment edges"
    dividers: "Rope or chain"

  decorations:
    corners: "Brass corner plates"
    dividers: "Anchor motifs"
    accents: "Compass rose, skulls"
    borders: "Rope twist pattern"
```

---

## 3. Color System

### 3.1 UI Color Palette

```yaml
color_palette:
  # Primary Colors
  primary:
    wood_dark:
      hex: "#5D4037"
      usage: "Panel backgrounds, frames"
    wood_medium:
      hex: "#795548"
      usage: "Secondary panels"
    wood_light:
      hex: "#A1887F"
      usage: "Highlights, hover states"

  # Secondary Colors
  secondary:
    parchment:
      hex: "#F5E6D3"
      usage: "Content backgrounds, text areas"
    parchment_dark:
      hex: "#E8D5C4"
      usage: "Parchment shadows"
    canvas:
      hex: "#FFF8E7"
      usage: "Light backgrounds"

  # Accent Colors
  accent:
    gold:
      hex: "#FFD700"
      usage: "Premium, important highlights"
    brass:
      hex: "#B8860B"
      usage: "Metal accents, borders"
    copper:
      hex: "#CD7F32"
      usage: "Secondary metal"

  # Semantic Colors
  semantic:
    success:
      hex: "#27AE60"
      usage: "Confirmations, positive"
    warning:
      hex: "#F39C12"
      usage: "Caution, attention"
    error:
      hex: "#E74C3C"
      usage: "Errors, danger"
    info:
      hex: "#3498DB"
      usage: "Information, help"

  # Rarity Colors
  rarity:
    common:
      hex: "#9E9E9E"
    uncommon:
      hex: "#4CAF50"
    rare:
      hex: "#2196F3"
    epic:
      hex: "#9C27B0"
    legendary:
      hex: "#FF9800"

  # Text Colors
  text:
    primary:
      hex: "#2C1810"
      usage: "Main text on light backgrounds"
    secondary:
      hex: "#5D4037"
      usage: "Secondary text"
    light:
      hex: "#F5E6D3"
      usage: "Text on dark backgrounds"
    disabled:
      hex: "#9E9E9E"
      usage: "Disabled text"
```

### 3.2 Contrast Requirements

```yaml
contrast_requirements:
  text_on_parchment:
    minimum_ratio: 4.5:1
    primary_text: "#2C1810" # 8.2:1 ratio
    secondary_text: "#5D4037" # 5.1:1 ratio

  text_on_wood:
    minimum_ratio: 4.5:1
    light_text: "#F5E6D3" # 6.8:1 ratio
    gold_text: "#FFD700" # 5.2:1 ratio

  interactive_elements:
    focus_indicator: "2px gold outline"
    hover_change: "15% brightness increase"
    pressed_change: "10% brightness decrease"
```

---

## 4. Typography

### 4.1 Font Family

```yaml
typography:
  primary_font:
    name: "Pirata One"
    fallback: "Georgia, serif"
    usage: "Headers, titles, emphasis"
    weights: [400]

  secondary_font:
    name: "Crimson Text"
    fallback: "Times New Roman, serif"
    usage: "Body text, descriptions"
    weights: [400, 600, 700]

  ui_font:
    name: "Open Sans"
    fallback: "Arial, sans-serif"
    usage: "Numbers, stats, small text"
    weights: [400, 600, 700]

  icon_font:
    name: "Plunderstorm Icons"
    usage: "Custom icon glyphs"
```

### 4.2 Type Scale

```yaml
type_scale:
  # Mobile-first sizes
  display:
    size: 48px
    line_height: 1.1
    weight: 400
    font: "Pirata One"
    usage: "Logo, splash screens"

  h1:
    size: 32px
    line_height: 1.2
    weight: 400
    font: "Pirata One"
    usage: "Screen titles"

  h2:
    size: 24px
    line_height: 1.25
    weight: 400
    font: "Pirata One"
    usage: "Section headers"

  h3:
    size: 20px
    line_height: 1.3
    weight: 600
    font: "Crimson Text"
    usage: "Subsections, card titles"

  body:
    size: 16px
    line_height: 1.5
    weight: 400
    font: "Crimson Text"
    usage: "Main content"

  body_small:
    size: 14px
    line_height: 1.4
    weight: 400
    font: "Crimson Text"
    usage: "Secondary content"

  caption:
    size: 12px
    line_height: 1.3
    weight: 400
    font: "Open Sans"
    usage: "Labels, hints"

  stat:
    size: 18px
    line_height: 1.2
    weight: 700
    font: "Open Sans"
    usage: "Numbers, stats"

  stat_large:
    size: 28px
    line_height: 1.1
    weight: 700
    font: "Open Sans"
    usage: "Large numbers"
```

### 4.3 Text Styling

```yaml
text_effects:
  title_shadow:
    offset: [2px, 2px]
    blur: 4px
    color: "#00000080"

  embossed:
    text_shadow: "1px 1px 0 #FFFFFF40, -1px -1px 0 #00000040"

  gold_text:
    gradient: "linear-gradient(180deg, #FFD700, #B8860B)"
    stroke: "1px #8B6914"

  outlined:
    stroke_width: 2px
    stroke_color: "#2C1810"
    fill: "#F5E6D3"
```

---

## 5. Panel and Frame Styles

### 5.1 Panel Types

```yaml
panels:
  primary_panel:
    name: "Wood Frame Panel"
    description: "Main container for content"
    background: "Parchment texture (#F5E6D3)"
    border:
      style: "Wood plank frame"
      width: 12px
      color: "#5D4037"
      corner_radius: 8px
    shadow:
      offset: [4px, 4px]
      blur: 8px
      color: "#00000040"
    inner_padding: 16px

  secondary_panel:
    name: "Simple Parchment"
    description: "Lighter content areas"
    background: "Parchment (#FFF8E7)"
    border:
      style: "Thin wood"
      width: 4px
      color: "#A1887F"
      corner_radius: 4px
    shadow:
      offset: [2px, 2px]
      blur: 4px
      color: "#00000020"
    inner_padding: 12px

  card_panel:
    name: "Item Card"
    description: "Individual item display"
    background: "Dark wood (#5D4037)"
    border:
      style: "Brass corners"
      width: 2px
      color: "#B8860B"
      corner_radius: 6px
    shadow:
      offset: [2px, 2px]
      blur: 4px
      color: "#00000060"
    inner_padding: 8px

  overlay_panel:
    name: "Modal Overlay"
    description: "Popup containers"
    background: "Parchment with vignette"
    border:
      style: "Ornate wood frame"
      width: 16px
      color: "#5D4037"
      corner_radius: 12px
      decoration: "Brass corner plates"
    shadow:
      offset: [8px, 8px]
      blur: 24px
      color: "#00000080"
    inner_padding: 24px

  tooltip_panel:
    name: "Tooltip"
    description: "Hover information"
    background: "#2C1810F0"
    border:
      style: "Thin gold"
      width: 1px
      color: "#B8860B"
      corner_radius: 4px
    shadow:
      offset: [2px, 2px]
      blur: 8px
      color: "#000000A0"
    inner_padding: 8px
    text_color: "#F5E6D3"
```

### 5.2 Panel Decorations

```yaml
decorations:
  corner_plates:
    style: "Brass L-shaped plates"
    size: 24px
    positions: ["top-left", "top-right", "bottom-left", "bottom-right"]
    image: "UI_Corner_Brass.png"

  rope_border:
    style: "Twisted rope"
    thickness: 8px
    color: "#C4A574"
    pattern: "Repeating twist"
    image: "UI_Border_Rope.png"

  divider_anchor:
    style: "Anchor symbol divider"
    width: 120px
    height: 24px
    image: "UI_Divider_Anchor.png"

  divider_wave:
    style: "Wave pattern"
    height: 8px
    pattern: "Repeating waves"
    color: "#3498DB40"
```

---

## 6. Button Styles

### 6.1 Button Types

```yaml
buttons:
  primary_button:
    name: "Gold Action Button"
    usage: "Main actions, confirmations"
    size:
      height: 56px
      min_width: 160px
      padding: "16px 32px"
    background:
      normal: "linear-gradient(180deg, #FFD700, #B8860B)"
      hover: "linear-gradient(180deg, #FFE44D, #D4A017)"
      pressed: "linear-gradient(180deg, #B8860B, #8B6914)"
      disabled: "#9E9E9E"
    border:
      width: 3px
      color_normal: "#8B6914"
      color_hover: "#B8860B"
      radius: 8px
    text:
      color: "#2C1810"
      size: 18px
      weight: 700
      shadow: "1px 1px 0 #FFD70040"
    shadow:
      normal: "0 4px 8px #00000040"
      hover: "0 6px 12px #00000050"
      pressed: "0 2px 4px #00000030"

  secondary_button:
    name: "Wood Button"
    usage: "Secondary actions, navigation"
    size:
      height: 48px
      min_width: 120px
      padding: "12px 24px"
    background:
      normal: "linear-gradient(180deg, #795548, #5D4037)"
      hover: "linear-gradient(180deg, #8D6E63, #6D4C41)"
      pressed: "linear-gradient(180deg, #5D4037, #4E342E)"
      disabled: "#9E9E9E"
    border:
      width: 2px
      color: "#A1887F"
      radius: 6px
    text:
      color: "#F5E6D3"
      size: 16px
      weight: 600
    shadow:
      normal: "0 3px 6px #00000040"

  tertiary_button:
    name: "Text Button"
    usage: "Minor actions, links"
    size:
      height: 40px
      padding: "8px 16px"
    background:
      normal: "transparent"
      hover: "#5D403720"
      pressed: "#5D403740"
    text:
      color_normal: "#5D4037"
      color_hover: "#795548"
      size: 16px
      weight: 600
      underline: true

  icon_button:
    name: "Round Icon Button"
    usage: "Actions with icons only"
    size:
      diameter: 48px
    background:
      normal: "#5D4037"
      hover: "#795548"
      pressed: "#4E342E"
    border:
      width: 2px
      color: "#B8860B"
      radius: "50%"
    icon:
      size: 24px
      color: "#F5E6D3"
    shadow:
      normal: "0 2px 4px #00000040"

  danger_button:
    name: "Danger Action"
    usage: "Destructive actions"
    inherits: "primary_button"
    background:
      normal: "linear-gradient(180deg, #E74C3C, #C0392B)"
      hover: "linear-gradient(180deg, #EC7063, #E74C3C)"
      pressed: "linear-gradient(180deg, #C0392B, #A93226)"
    border:
      color: "#922B21"
```

### 6.2 Button States

```yaml
button_states:
  normal:
    transform: "scale(1)"
    opacity: 1
    transition: "all 0.15s ease"

  hover:
    transform: "scale(1.02)"
    brightness: 1.1
    transition: "all 0.15s ease"

  pressed:
    transform: "scale(0.98)"
    brightness: 0.9
    transition: "all 0.05s ease"

  disabled:
    opacity: 0.5
    cursor: "not-allowed"
    filter: "grayscale(100%)"

  loading:
    opacity: 0.8
    content: "Spinner icon replacing text"

  focus:
    outline: "3px solid #FFD700"
    outline_offset: 2px
```

### 6.3 Touch Feedback

```yaml
touch_feedback:
  tap_highlight:
    type: "Radial ripple"
    color: "#FFD70040"
    duration: 300ms

  press_scale:
    amount: 0.96
    duration: 100ms
    easing: "ease-out"

  haptic:
    light: "Selection tap"
    medium: "Button press"
    heavy: "Important action"
```

---

## 7. Icon System

### 7.1 Icon Grid and Sizes

```yaml
icon_system:
  grid:
    base_unit: 24px
    pixel_perfect: true
    keyline_shapes: ["circle", "square", "rounded-square"]

  sizes:
    xs:
      size: 16px
      usage: "Inline with text"
      stroke_width: 1.5px

    sm:
      size: 24px
      usage: "Standard UI icons"
      stroke_width: 2px

    md:
      size: 32px
      usage: "Feature icons"
      stroke_width: 2px

    lg:
      size: 48px
      usage: "Category icons"
      stroke_width: 2.5px

    xl:
      size: 64px
      usage: "Large feature display"
      stroke_width: 3px

  padding:
    optical_size: "2px internal padding"
    touch_target: "Minimum 44px hit area"
```

### 7.2 Icon Categories

```yaml
icon_categories:
  navigation:
    icons:
      - home
      - back
      - forward
      - menu
      - close
      - settings
      - profile
      - search
    style: "Outlined, rounded"

  gameplay:
    icons:
      - health
      - mana
      - stamina
      - damage
      - armor
      - speed
      - cooldown
      - level_up
    style: "Filled with detail"

  abilities:
    icons:
      - fireball
      - cutlass_slash
      - cannonball
      - lightning
      - poison
      - grapple
      - barrel_roll
      - healing
      - smoke
      - wind
    style: "Detailed with color"
    container: "Rounded square with rarity border"

  currency:
    icons:
      - gold_coin
      - doubloon
      - battle_pass_xp
      - gem
    style: "3D appearance, shiny"

  social:
    icons:
      - friends
      - party
      - chat
      - voice
      - mute
      - block
      - report
    style: "Outlined, friendly"

  equipment:
    icons:
      - weapon
      - armor
      - accessory
      - consumable
      - cosmetic
    style: "Silhouette based"

  system:
    icons:
      - notification
      - warning
      - error
      - success
      - info
      - loading
      - sync
    style: "Clear, universal"

  pirate_themed:
    icons:
      - anchor
      - ship_wheel
      - compass
      - treasure_map
      - skull
      - crossed_swords
      - treasure_chest
      - spyglass
    style: "Decorative, thematic"
```

### 7.3 Icon Design Guidelines

```yaml
icon_guidelines:
  style:
    primary_stroke: 2px
    corner_radius: 2px
    line_caps: "Round"
    line_joins: "Round"
    fill_style: "Solid or gradient"

  colors:
    monochrome: "#2C1810"
    monochrome_light: "#F5E6D3"
    colored: "Per-category palette"
    disabled: "#9E9E9E"

  effects:
    active_glow:
      color: "#FFD700"
      blur: 4px
    notification_badge:
      color: "#E74C3C"
      size: 8px
      position: "Top-right"

  accessibility:
    contrast_ratio: "3:1 minimum"
    distinguishable: "Shape differs when color similar"
```

### 7.4 Icon Asset List

```yaml
icon_deliverables:
  format: "SVG + PNG exports"
  png_densities: ["1x", "2x", "3x"]

  total_icons: ~100

  breakdown:
    navigation: 12
    gameplay: 15
    abilities: 20
    currency: 5
    social: 10
    equipment: 8
    system: 12
    pirate_themed: 10
    misc: 8
```

---

## 8. Progress Bars and Indicators

### 8.1 Health Bar

```yaml
health_bar:
  style: "Segmented with frame"

  dimensions:
    width: 200px
    height: 24px
    segments: 10

  visuals:
    frame:
      color: "#5D4037"
      width: 3px
      corner_radius: 4px
    background:
      color: "#2C181080"
    fill:
      gradient: "linear-gradient(180deg, #E74C3C, #C0392B)"
      segments_visible: true
      segment_gap: 2px
    low_health:
      threshold: 25%
      animation: "Pulse red"
      pulse_speed: 0.5s

  text:
    show: true
    format: "current/max"
    position: "Center"
    color: "#FFFFFF"
    shadow: "1px 1px 2px #000000"
```

### 8.2 XP/Progress Bar

```yaml
xp_bar:
  style: "Parchment fill"

  dimensions:
    width: "Full width"
    height: 16px

  visuals:
    frame:
      color: "#A1887F"
      width: 2px
      corner_radius: 8px
    background:
      texture: "Parchment"
    fill:
      gradient: "linear-gradient(90deg, #F39C12, #FFD700)"
      animation: "Shimmer on gain"
    level_indicator:
      position: "Left side"
      style: "Brass medallion"

  text:
    show: true
    format: "current/needed XP"
    position: "Right of bar"
```

### 8.3 Cooldown Indicator

```yaml
cooldown_indicator:
  style: "Radial sweep"

  dimensions:
    size: 64px (matches ability icon)

  visuals:
    overlay:
      color: "#00000080"
      sweep_direction: "Clockwise from top"
    border:
      color: "#FFFFFF40"
      width: 2px
    ready_flash:
      color: "#FFD700"
      duration: 0.3s

  text:
    show: true
    format: "Seconds remaining"
    size: 20px
    color: "#FFFFFF"
    position: "Center"
```

### 8.4 Loading Indicators

```yaml
loading_indicators:
  spinner:
    type: "Ship wheel rotation"
    size: 48px
    color: "#B8860B"
    speed: 1s per rotation

  progress_bar:
    type: "Horizontal fill"
    width: 300px
    height: 12px
    background: "#5D4037"
    fill: "#FFD700"
    text: "Percentage below"

  dots:
    type: "Three bouncing dots"
    size: 8px each
    color: "#B8860B"
    animation: "Sequential bounce"
```

### 8.5 Stat Bars

```yaml
stat_bars:
  mini_bar:
    width: 60px
    height: 6px
    usage: "Compact stat display"
    no_text: true

  labeled_bar:
    width: 120px
    height: 12px
    usage: "Stat with label"
    label_position: "Above"

  comparison_bar:
    width: 150px
    height: 16px
    usage: "Before/after comparison"
    dual_fill: true
    colors:
      current: "#3498DB"
      change_positive: "#27AE60"
      change_negative: "#E74C3C"
```

---

## 9. Popup and Modal Styles

### 9.1 Modal Types

```yaml
modals:
  confirmation_modal:
    name: "Confirmation Dialog"
    size: "360px x auto"
    elements:
      - title (H2)
      - message (body)
      - button_row (2 buttons)
    animation:
      enter: "Scale up + fade in"
      exit: "Scale down + fade out"
      duration: 200ms

  info_modal:
    name: "Information Panel"
    size: "400px x auto"
    elements:
      - icon (optional)
      - title
      - content (scrollable)
      - close_button
    animation:
      enter: "Slide up + fade"
      duration: 250ms

  reward_modal:
    name: "Reward Reveal"
    size: "Full screen overlay"
    elements:
      - reward_display (animated)
      - reward_name
      - rarity_indicator
      - claim_button
    animation:
      enter: "Dramatic reveal"
      effects: "Particles, glow"
      duration: 1000ms

  inventory_modal:
    name: "Item Details"
    size: "320px x 480px"
    elements:
      - item_preview (3D or 2D)
      - item_name + rarity
      - stats_list
      - description
      - action_buttons
    animation:
      enter: "Slide from right"
      duration: 200ms
```

### 9.2 Overlay Background

```yaml
overlay_background:
  color: "#000000"
  opacity: 0.75
  blur: "Optional 4px backdrop blur"
  click_to_dismiss: "Configurable"
  animation:
    fade_in: 150ms
    fade_out: 100ms
```

### 9.3 Toast Notifications

```yaml
toast_notifications:
  position: "Top center"
  margin_from_top: 60px
  max_width: 320px

  types:
    success:
      background: "#27AE60"
      icon: "checkmark"
    error:
      background: "#E74C3C"
      icon: "x-circle"
    warning:
      background: "#F39C12"
      icon: "warning"
    info:
      background: "#3498DB"
      icon: "info"
    achievement:
      background: "#9C27B0"
      icon: "trophy"
      special: true

  animation:
    enter: "Slide down + fade"
    exit: "Slide up + fade"
    duration:
      enter: 200ms
      display: 3000ms
      exit: 150ms
```

---

## 10. Interactive Components

### 10.1 Toggles and Checkboxes

```yaml
toggles:
  switch:
    width: 52px
    height: 28px
    track:
      off_color: "#5D4037"
      on_color: "#27AE60"
      border: "2px solid #A1887F"
    thumb:
      size: 24px
      color: "#F5E6D3"
      shadow: "0 2px 4px #00000040"
    animation:
      duration: 150ms
      easing: "ease-out"

  checkbox:
    size: 24px
    background:
      unchecked: "#F5E6D3"
      checked: "#27AE60"
    border:
      color: "#5D4037"
      width: 2px
      radius: 4px
    checkmark:
      color: "#FFFFFF"
      stroke_width: 3px
    animation:
      scale_pop: 1.1
      duration: 100ms

  radio:
    size: 24px
    outer_circle:
      color: "#5D4037"
      width: 2px
    inner_circle:
      color: "#27AE60"
      size: 12px
    animation:
      scale_pop: 1.1
```

### 10.2 Sliders

```yaml
sliders:
  standard:
    track:
      height: 8px
      background: "#5D4037"
      fill: "#FFD700"
      border_radius: 4px
    thumb:
      size: 28px
      shape: "Circle"
      background: "#F5E6D3"
      border: "3px solid #B8860B"
      shadow: "0 2px 4px #00000040"
    labels:
      position: "Above"
      show_value: true

  volume:
    inherits: "standard"
    icon_left: "volume_off"
    icon_right: "volume_max"
```

### 10.3 Input Fields

```yaml
input_fields:
  text_input:
    height: 48px
    padding: "12px 16px"
    background: "#FFF8E7"
    border:
      width: 2px
      color_normal: "#A1887F"
      color_focus: "#B8860B"
      color_error: "#E74C3C"
      radius: 6px
    text:
      color: "#2C1810"
      placeholder_color: "#9E9E9E"
      size: 16px
    states:
      focus: "Gold border + subtle glow"
      error: "Red border + error text below"
      disabled: "Grayed out"

  search_input:
    inherits: "text_input"
    icon_left: "search"
    clear_button: true
```

### 10.4 Dropdown/Select

```yaml
dropdown:
  trigger:
    height: 48px
    background: "#F5E6D3"
    border: "2px solid #A1887F"
    border_radius: 6px
    arrow_icon: "chevron_down"

  menu:
    background: "#FFF8E7"
    border: "2px solid #5D4037"
    shadow: "0 4px 12px #00000040"
    max_height: 240px

  option:
    height: 44px
    padding: "12px 16px"
    hover_background: "#E8D5C4"
    selected_background: "#D4C4B0"
    selected_indicator: "Check icon"
```

---

## 11. Responsive Design

### 11.1 Breakpoints

```yaml
breakpoints:
  mobile_small:
    width: "< 360px"
    scale: 0.85

  mobile:
    width: "360px - 414px"
    scale: 1.0

  mobile_large:
    width: "414px - 480px"
    scale: 1.0

  tablet:
    width: "480px - 768px"
    scale: 1.1

  tablet_large:
    width: "> 768px"
    scale: 1.2
```

### 11.2 Scaling Rules

```yaml
scaling:
  text:
    method: "rem-based with viewport clamp"
    min_scale: 0.85
    max_scale: 1.3

  touch_targets:
    minimum: 44px
    recommended: 48px
    comfortable: 56px

  spacing:
    base_unit: 8px
    scales: [4, 8, 12, 16, 24, 32, 48, 64]

  panels:
    max_width: 600px
    side_padding: "16px minimum"
```

### 11.3 Safe Area Handling

```yaml
safe_areas:
  notch_avoidance:
    top: "env(safe-area-inset-top)"
    bottom: "env(safe-area-inset-bottom)"

  critical_ui:
    margin_top: "max(16px, safe-area-top)"
    margin_bottom: "max(16px, safe-area-bottom)"

  home_indicator:
    bottom_padding: 34px
```

---

## 12. Animation Guidelines

### 12.1 Timing Functions

```yaml
timing_functions:
  ease_default: "cubic-bezier(0.4, 0, 0.2, 1)"
  ease_in: "cubic-bezier(0.4, 0, 1, 1)"
  ease_out: "cubic-bezier(0, 0, 0.2, 1)"
  ease_bounce: "cubic-bezier(0.68, -0.55, 0.265, 1.55)"
  ease_spring: "cubic-bezier(0.175, 0.885, 0.32, 1.275)"
```

### 12.2 Duration Standards

```yaml
animation_durations:
  instant: 50ms
  fast: 100ms
  normal: 200ms
  slow: 300ms
  reveal: 500ms
  dramatic: 1000ms

  guidelines:
    - "Micro-interactions: 50-150ms"
    - "State changes: 150-250ms"
    - "Page transitions: 250-400ms"
    - "Celebrations: 500-1500ms"
```

### 12.3 Common Animations

```yaml
animations:
  fade_in:
    opacity: [0, 1]
    duration: 200ms

  scale_in:
    transform: ["scale(0.9)", "scale(1)"]
    opacity: [0, 1]
    duration: 200ms

  slide_up:
    transform: ["translateY(20px)", "translateY(0)"]
    opacity: [0, 1]
    duration: 250ms

  bounce:
    transform: ["scale(1)", "scale(1.1)", "scale(1)"]
    duration: 300ms

  shake:
    transform: ["translateX(0)", "translateX(-5px)", "translateX(5px)", "translateX(0)"]
    duration: 400ms
    iterations: 2

  pulse_glow:
    box_shadow: ["0 0 0 gold", "0 0 20px gold", "0 0 0 gold"]
    duration: 1000ms
    loop: true
```

---

## 13. Asset Specifications

### 13.1 Texture Assets

```yaml
texture_assets:
  ui_atlas:
    name: "T_UI_Atlas"
    size: 2048x2048
    format: "ASTC 6x6"
    contents:
      - "All button states"
      - "Panel frames"
      - "Decorations"
      - "Common icons"

  parchment_texture:
    name: "T_UI_Parchment"
    size: 512x512
    tiling: true

  wood_texture:
    name: "T_UI_Wood"
    size: 512x512
    tiling: true

  rope_border:
    name: "T_UI_Rope"
    size: 256x64
    9_slice: true
```

### 13.2 9-Slice Configuration

```yaml
9_slice:
  panel_wood:
    border: [16, 16, 16, 16]
    center: "Tile"

  panel_parchment:
    border: [12, 12, 12, 12]
    center: "Stretch"

  button_gold:
    border: [12, 12, 12, 12]
    center: "Stretch"

  button_wood:
    border: [8, 8, 8, 8]
    center: "Stretch"
```

---

## 14. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|--------------|---------------------|
| UI theme matches pirate style | Wood, parchment, brass materials | Visual review |
| Readability maintained | 4.5:1 contrast minimum | Contrast checker |
| Icons clear at small sizes | Legible at 16px | Device testing |
| Button states obvious | 4 distinct states per button | Interactive testing |
| Consistent across all screens | Style guide compliance | Screen audit |
| Scalable for different resolutions | Responsive breakpoints | Multi-device testing |

---

## 15. Deliverables

### 15.1 Asset Deliverables

```yaml
deliverables:
  design_files:
    - "UI_StyleGuide.figma"
    - "UI_Components.figma"
    - "UI_Icons.figma"

  textures:
    - "T_UI_Atlas.png"
    - "T_UI_Parchment.png"
    - "T_UI_Wood.png"
    - "T_UI_Rope.png"

  fonts:
    - "PirataOne-Regular.ttf"
    - "CrimsonText-Regular.ttf"
    - "CrimsonText-SemiBold.ttf"
    - "CrimsonText-Bold.ttf"
    - "OpenSans-Regular.ttf"
    - "OpenSans-SemiBold.ttf"
    - "OpenSans-Bold.ttf"
    - "PlunderstormIcons.ttf"

  icons:
    - "Icons_Navigation/" (12 icons)
    - "Icons_Gameplay/" (15 icons)
    - "Icons_Abilities/" (20 icons)
    - "Icons_Currency/" (5 icons)
    - "Icons_Social/" (10 icons)
    - "Icons_System/" (12 icons)
    - "Icons_Pirate/" (10 icons)

  prefabs:
    - "UI_Button_Primary.prefab"
    - "UI_Button_Secondary.prefab"
    - "UI_Panel_Primary.prefab"
    - "UI_Panel_Card.prefab"
    - "UI_Modal_Confirmation.prefab"
    - "UI_Toast.prefab"
    - "UI_ProgressBar_Health.prefab"
    - "UI_ProgressBar_XP.prefab"
```

### 15.2 Documentation

```yaml
documentation:
  - "This specification document"
  - "Component usage guide"
  - "Icon naming conventions"
  - "Animation reference sheet"
```

---

## 16. Appendix

### 16.1 Related Documents

- [VISUAL_STYLE_GUIDE.md](VISUAL_STYLE_GUIDE.md) - Overall art direction
- [TECHNICAL_ART_SPECIFICATIONS.md](TECHNICAL_ART_SPECIFICATIONS.md) - Performance specs
- ART-018: Ability Icons (extends icon system)

### 16.2 Color Reference Quick Sheet

| Name | Hex | Usage |
|------|-----|-------|
| Wood Dark | #5D4037 | Primary frames |
| Parchment | #F5E6D3 | Backgrounds |
| Gold | #FFD700 | Primary actions |
| Brass | #B8860B | Accents |
| Text Dark | #2C1810 | Primary text |
| Success | #27AE60 | Positive |
| Error | #E74C3C | Negative |
| Info | #3498DB | Neutral |

### 16.3 Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-30 | Initial specification |
