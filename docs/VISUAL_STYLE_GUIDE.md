# Visual Style Guide
# Plunderstorm Mobile - Art Direction & Style Standards

**Document Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Draft - Pending Stakeholder Approval
**Task ID:** ART-001

---

## Table of Contents

1. [Style Overview](#1-style-overview)
2. [Color Palette](#2-color-palette)
3. [Character Style](#3-character-style)
4. [Environment Style](#4-environment-style)
5. [UI Style](#5-ui-style)
6. [Visual Effects Style](#6-visual-effects-style)
7. [Technical Art Specifications](#7-technical-art-specifications)
8. [Mobile Optimization Guidelines](#8-mobile-optimization-guidelines)
9. [Reference Gallery](#9-reference-gallery)
10. [Appendices](#10-appendices)

---

## 1. Style Overview

### 1.1 Visual Identity Statement

**Plunderstorm Mobile** embraces a **stylized, vibrant, and whimsical** visual identity that combines the adventurous spirit of pirate fantasy with the accessibility of mobile gaming. Our art style prioritizes **readability, charm, and performance** while delivering a visually rich experience that stands out in the mobile battle royale market.

### 1.2 Style Pillars

| Pillar | Description | Implementation |
|--------|-------------|----------------|
| **Stylized Realism** | Exaggerated proportions and bold shapes over realistic detail | Larger heads/hands, simplified geometry, clear silhouettes |
| **Vibrant Colors** | Saturated but harmonious palette that pops on mobile screens | High contrast, warm dominant palette, strategic accent colors |
| **Whimsical Fantasy** | Magic meets pirate adventure; playful, not gritty | Magical effects, fantastical creatures, lighthearted themes |
| **Mobile Optimized** | Clear, readable visuals that work on small screens | High contrast, simple shapes, readable at arm's length |
| **Timeless Appeal** | Avoid trends, focus on classic pirate fantasy | Classic pirate tropes, universal fantasy elements |

### 1.3 Style Keywords

**DO:**
- Adventurous
- Colorful
- Playful
- Magical
- Heroic
- Cartoony
- Inviting
- Readable

**DON'T:**
- Gritty
- Realistic
- Dark
- Gore/Violence
- Muddy colors
- Overly detailed
- Complex textures
- Photorealistic

### 1.4 Target Age Rating

Visual style should support **PEGI 7 / ESRB E10+** rating:
- No blood or gore (use stylized hit effects)
- Violence is cartoonish and consequence-free
- Death effects are "poof" or teleport animations
- Skeleton pirates are comical, not scary
- All characters are fully clothed appropriately

---

## 2. Color Palette

### 2.1 Primary Palette

The primary palette represents the core visual identity of Plunderstorm and should dominate all scenes.

| Color Name | Hex Code | RGB | Use Case |
|------------|----------|-----|----------|
| **Ocean Blue** | `#1E90FF` | 30, 144, 255 | Water, sky accents, UI highlights |
| **Deep Sea** | `#0066CC` | 0, 102, 204 | Shadows, depth, secondary water |
| **Sandy Beige** | `#F4A460` | 244, 164, 96 | Beach, sand, terrain base |
| **Warm Sand** | `#DEB887` | 222, 184, 135 | Light terrain, highlights |
| **Coconut Brown** | `#8B4513` | 139, 69, 19 | Wood, ships, UI frames |

### 2.2 Secondary Palette

Secondary colors add variety and help distinguish different areas and elements.

| Color Name | Hex Code | RGB | Use Case |
|------------|----------|-----|----------|
| **Tropical Green** | `#32CD32` | 50, 205, 50 | Jungle, vegetation, health |
| **Palm Leaf** | `#228B22` | 34, 139, 34 | Dense foliage, shadows |
| **Sunset Orange** | `#FF6347` | 255, 99, 71 | Sunset sky, fire, danger |
| **Coral Pink** | `#FF7F7F` | 255, 127, 127 | Coral reefs, magical effects |
| **Sky Azure** | `#87CEEB` | 135, 206, 235 | Daytime sky, ice effects |

### 2.3 Accent Palette

Accent colors are used sparingly for important gameplay elements, rewards, and UI highlights.

| Color Name | Hex Code | RGB | Use Case |
|------------|----------|-----|----------|
| **Treasure Gold** | `#FFD700` | 255, 215, 0 | Currency, legendary items, rewards |
| **Crimson Red** | `#DC143C` | 220, 20, 60 | Damage, enemies, danger zones |
| **Royal Purple** | `#9932CC` | 153, 50, 204 | Epic rarity, magic, abilities |
| **Mystic Cyan** | `#00CED1` | 0, 206, 209 | Rare items, special effects |
| **Silver Gleam** | `#C0C0C0` | 192, 192, 192 | Metal, weapons, UI elements |

### 2.4 UI Palette

Specific colors for user interface elements ensuring readability and theme consistency.

| Color Name | Hex Code | RGB | Use Case |
|------------|----------|-----|----------|
| **Parchment** | `#FFF8DC` | 255, 248, 220 | Text backgrounds, paper elements |
| **Old Paper** | `#F5DEB3` | 245, 222, 179 | Aged document backgrounds |
| **Dark Wood** | `#654321` | 101, 67, 33 | UI frames, borders |
| **Ink Black** | `#2C2C2C` | 44, 44, 44 | Text, shadows |
| **Bone White** | `#FFFAF0` | 255, 250, 240 | Primary text, highlights |

### 2.5 Rarity Color Coding

Consistent color coding for item and ability rarities across all UI and VFX.

| Rarity | Border Color | Glow Color | Hex (Border) | Hex (Glow) |
|--------|--------------|------------|--------------|------------|
| **Common** | Gray | None | `#9E9E9E` | - |
| **Uncommon** | Green | Soft Green | `#4CAF50` | `#81C784` |
| **Rare** | Blue | Bright Blue | `#2196F3` | `#64B5F6` |
| **Epic** | Purple | Purple Glow | `#9C27B0` | `#BA68C8` |
| **Legendary** | Gold | Golden Aura | `#FF9800` | `#FFD54F` |

### 2.6 Damage Type Colors

Visual identification of damage types through consistent color coding.

| Damage Type | Primary Color | Hex Code | VFX Style |
|-------------|---------------|----------|-----------|
| **Physical** | White/Gray | `#E0E0E0` | Sharp, solid impacts |
| **Fire** | Orange | `#FF5722` | Flames, embers |
| **Ice** | Light Blue | `#4FC3F7` | Crystals, frost |
| **Lightning** | Yellow | `#FFEB3B` | Electric arcs |
| **Poison** | Green | `#8BC34A` | Bubbles, mist |
| **Arcane** | Purple | `#7E57C2` | Magic particles, glyphs |

### 2.7 Color Usage Guidelines

**Contrast Requirements:**
- Text on backgrounds: Minimum 4.5:1 contrast ratio
- Interactive elements: Clearly distinguishable from background
- Enemy vs ally: Distinct color coding (red/green or team colors)

**Color Balance:**
- 60% Primary palette (environment, base elements)
- 30% Secondary palette (variety, biome-specific)
- 10% Accent palette (important elements only)

---

## 3. Character Style

### 3.1 Proportions

Characters in Plunderstorm use stylized proportions for personality and mobile readability.

**Standard Character Proportions:**

| Body Part | Proportion | Notes |
|-----------|------------|-------|
| **Head** | 1.5x realistic | Larger for expressions and recognition |
| **Hands** | 1.3x realistic | Exaggerated for gestures and weapon grips |
| **Feet** | 1.2x realistic | Slightly larger for grounded stance |
| **Body** | 0.9x realistic | Slightly compact torso |
| **Arms** | 1.1x realistic | Slightly longer for dynamic poses |
| **Overall Height** | 5.5-6 heads tall | Hero proportions, not realistic 7-8 heads |

**Character Height Reference:**
```
                    ___
                   /   \      <- HEAD (1.5 units)
                  | o o |
                   \___/
                    | |       <- NECK (0.3 units)
                 /--|--|--\
                /   |  |   \  <- TORSO (1.5 units)
               |    |  |    |
                \   |  |   /
                 \__|  |__/   <- HIPS (0.5 units)
                   /    \
                  /      \    <- LEGS (2.0 units)
                 /        \
                /__________\  <- FEET (0.2 units)

Total: ~6 heads tall
```

### 3.2 Face Style

**Facial Features:**
- **Eyes:** Large, expressive, simple shape (2-3 color gradients max)
- **Eyebrows:** Thick, highly mobile for expressions
- **Nose:** Small to medium, simple geometric shape
- **Mouth:** Medium-sized, wide range of expressions
- **Ears:** Slightly oversized, add to silhouette
- **Facial Hair:** Stylized, chunky shapes (no fine detail)

**Expression Guidelines:**
- All characters should have a neutral expression that reads as "confident"
- Eyebrow and mouth shapes drive most expression
- Avoid realistic facial anatomy
- Eye highlights add life and personality

### 3.3 Body Types

Support diverse body types while maintaining style consistency.

| Body Type | Description | Silhouette |
|-----------|-------------|------------|
| **Standard** | Athletic, balanced proportions | Medium width |
| **Sturdy** | Broad shoulders, heavier build | Wide, triangular |
| **Lithe** | Slender, agile appearance | Narrow, elongated |
| **Stout** | Compact, powerful stance | Square, grounded |

**Note:** All body types must fit the same hitbox for gameplay fairness.

### 3.4 Clothing & Costume Approach

**Design Principles:**
1. **Layered Construction:** Base + mid-layer + accessories
2. **Silhouette First:** Distinctive outline visible at distance
3. **Theme Clarity:** Costume communicates character fantasy immediately
4. **Modular Design:** Pieces can mix-and-match within reason
5. **No Clipping:** Designed to work with all animations

**Costume Elements:**

| Slot | Examples | Design Notes |
|------|----------|--------------|
| **Head** | Hats, bandanas, helmets, masks | Distinctive silhouette toppers |
| **Face** | Eyepatches, goggles, face paint | Small but impactful details |
| **Body** | Coats, vests, armor, robes | Main costume mass |
| **Legs** | Pants, skirts, leg armor | Support overall silhouette |
| **Feet** | Boots, sandals, peg legs | Ground the character |
| **Back** | Capes, wings, backpacks | Enhance silhouette in motion |
| **Accessories** | Belts, jewelry, weapons | Add detail and personality |

**Material Guidelines:**
- **Fabric:** Soft, flowing, shows movement
- **Leather:** Matte, slightly worn appearance
- **Metal:** Polished with clear highlights, not too shiny
- **Wood:** Warm, visible grain (stylized)
- **Magical:** Glowing edges, particle attachments

### 3.5 Skin Tone Diversity

Support diverse character representation.

| Skin Tone | Hex Range | Notes |
|-----------|-----------|-------|
| **Light** | `#FFE0BD` - `#F5DEB3` | Pink/peach undertones |
| **Medium Light** | `#E0AC69` - `#C68642` | Warm golden undertones |
| **Medium** | `#B07D56` - `#8D5524` | Rich brown tones |
| **Medium Dark** | `#6B4423` - `#5C4033` | Deep warm tones |
| **Dark** | `#4A3728` - `#3B2F2F` | Rich, deep tones |

All skin tones should have warm undertones consistent with the tropical setting.

### 3.6 Character Silhouette Requirements

**Readability Test:**
1. Fill character with solid black
2. Must be identifiable at 50% screen zoom
3. Costume theme should be clear from silhouette alone
4. No two costumes should have identical silhouettes

---

## 4. Environment Style

### 4.1 Geometry Style

**Modeling Approach:**
- **Primary Forms:** Bold, simplified shapes
- **Edge Treatment:** Soft bevels, no hard 90-degree angles in organic forms
- **Detail Level:** Minimal fine detail; rely on texture and lighting
- **Scale:** Slightly exaggerated for gameplay clarity

**Geometry Guidelines:**

| Element | Approach | Notes |
|---------|----------|-------|
| **Terrain** | Smooth, rounded hills | Avoid realistic noise |
| **Rocks** | Chunky, faceted crystals | Stylized geological forms |
| **Trees** | Simple trunk + crown | Readable mass shapes |
| **Buildings** | Slightly warped/leaning | Whimsical, not straight |
| **Props** | Oversized, readable | Exaggerated for visibility |

### 4.2 Biome Styles

**Beach/Coastal:**
- Sandy beige terrain with pale blue water
- Palm trees, coral formations, seashells
- Bright, high-key lighting
- Pirate debris: barrels, crates, rope

**Jungle:**
- Dense tropical greens
- Large leaves, twisted vines
- Dappled lighting through canopy
- Ancient stone ruins peeking through

**Volcanic:**
- Dark grays and blacks with orange/red accents
- Smoke, embers, lava flows
- Harsh shadows, dramatic lighting
- Volcanic rock formations

**Ruins/Temple:**
- Weathered stone in warm beige
- Overgrown with vegetation
- Mysterious glowing elements
- Treasure and skeleton decorations

**Town/Port:**
- Wooden structures, colorful fabrics
- Busy, cluttered aesthetic
- Warm wood tones with accent colors
- Signs, flags, market stalls

### 4.3 Texture Approach

**General Principles:**
- **Hand-Painted Style:** Visible brush strokes, not photo-based
- **Color Variation:** Subtle hue shifts within materials
- **Simplified Detail:** Suggest detail, don't render it literally
- **Consistent Light Direction:** Baked highlights follow world lighting

**Texture Types:**

| Material | Style Notes | Color Variation |
|----------|-------------|-----------------|
| **Sand** | Soft gradients, minimal noise | Warm tan to light beige |
| **Grass** | Painted clumps, directional | Multiple greens, yellow tips |
| **Stone** | Chunky highlights, simple cracks | Gray with warm/cool variation |
| **Wood** | Bold grain, warm colors | Brown with orange/yellow |
| **Water** | Stylized caustics, foam | Blue-green gradient |
| **Metal** | Sharp highlights, patina | Base color + rust/wear |

### 4.4 Lighting Design

**Global Lighting:**
- **Time of Day:** Perpetual "golden hour" (late afternoon)
- **Sun Position:** 20-30 degrees above horizon
- **Sun Color:** Warm yellow-orange (`#FFE4B5`)
- **Ambient Color:** Soft blue (`#B0C4DE`)
- **Shadow Softness:** Medium-soft, no harsh edges

**Lighting Principles:**
1. Characters should always be well-lit for gameplay
2. No areas so dark players can hide unfairly
3. Environmental lighting tells story but doesn't impede gameplay
4. Special lighting for POIs and landmarks

**Lighting by Location:**

| Location Type | Key Light | Fill Light | Mood |
|---------------|-----------|------------|------|
| **Beach** | Bright sun | Sky reflection | Open, inviting |
| **Jungle** | Dappled | Ambient green | Mysterious |
| **Cave/Interior** | Torch/magical | Dim ambient | Tense |
| **Town** | Warm sun | Bounce from buildings | Busy, lively |

### 4.5 Prop Design

**Prop Hierarchy:**
1. **Hero Props:** Story-important, highly detailed (treasure chests, cannons)
2. **Kit Props:** Reusable environment builders (barrels, crates, fences)
3. **Scatter Props:** Small detail items (shells, coins, bottles)

**Prop Guidelines:**
- Oversized by 20-30% for visibility
- Bright colors for important/interactive props
- Consistent wear patterns (salt spray, sun fade)
- Pirate theme in everything

**Interactive Prop Indicators:**
- Subtle glow/highlight
- Slight animation (floating, bobbing)
- Distinct silhouette from static props

---

## 5. UI Style

### 5.1 UI Theme

**Overall Aesthetic:** Pirate ship meets treasure map
- Wooden frames and rope borders
- Parchment and aged paper textures
- Nautical decorations (anchors, compasses, skulls)
- Hand-drawn quality to typography

### 5.2 Panel Styles

**Primary Panel:**
```
┌─────────────────────────────┐
│ ╔═══════════════════════╗   │
│ ║                       ║   │  Dark wood outer frame
│ ║    Parchment          ║   │  Rope/nail decoration
│ ║    Content Area       ║   │  Parchment inner panel
│ ║                       ║   │
│ ╚═══════════════════════╝   │
└─────────────────────────────┘
```

**Panel Specifications:**
| Element | Color | Notes |
|---------|-------|-------|
| **Outer Frame** | Dark Wood `#654321` | 8-12px border |
| **Inner Border** | Rope/Gold `#D4A574` | 2-4px accent |
| **Background** | Parchment `#FFF8DC` | Subtle texture |
| **Shadow** | Soft black | 15% opacity drop shadow |

**Secondary Panel (Tooltips, Small Info):**
- Simpler frame (single border)
- Slightly transparent background
- Quick fade in/out animations

### 5.3 Button Styles

**Primary Button (Actions, Confirm):**
| State | Background | Border | Text |
|-------|------------|--------|------|
| Normal | Gold `#D4A574` | Dark wood `#654321` | Dark `#2C2C2C` |
| Hover/Focus | Light gold `#E5C69B` | Gold `#FFD700` | Dark `#2C2C2C` |
| Pressed | Dark gold `#B8860B` | Dark `#654321` | White `#FFFFFF` |
| Disabled | Gray `#9E9E9E` | Dark gray `#666666` | Light gray `#CCCCCC` |

**Secondary Button (Cancel, Back):**
| State | Background | Border | Text |
|-------|------------|--------|------|
| Normal | Wood `#8B4513` | Dark `#654321` | Parchment `#FFF8DC` |
| Hover/Focus | Light wood `#A0522D` | Gold `#D4A574` | White `#FFFFFF` |
| Pressed | Dark wood `#5D3A1A` | Dark `#654321` | Parchment `#FFF8DC` |
| Disabled | Gray `#666666` | Dark gray `#444444` | Light gray `#999999` |

**Button Minimum Size:** 44x44 pixels (touch target requirement)

### 5.4 Typography

**Font Hierarchy:**

| Use | Style | Size (Relative) | Color |
|-----|-------|-----------------|-------|
| **Headers (H1)** | Bold, Decorative | 2.0em | Treasure Gold `#FFD700` |
| **Subheaders (H2)** | Bold | 1.5em | Dark Wood `#654321` |
| **Body Text** | Regular | 1.0em | Ink Black `#2C2C2C` |
| **Captions** | Light | 0.85em | Dark Gray `#666666` |
| **Numbers (Currency)** | Bold | 1.2em | Gold `#FFD700` |
| **Damage Numbers** | Bold, Outline | 1.5-2.0em | Type-specific colors |

**Typography Guidelines:**
- Primary font: Sans-serif with slight personality (rounded, friendly)
- Decorative font: For headers only, pirate-themed
- All text must pass contrast requirements
- Numbers should be tabular (equal width) for UI alignment

**Recommended Font Pairings:**
- Headers: Pirata One, Treasure Map Deadhand, or similar
- Body: Nunito, Quicksand, or similar rounded sans-serif

### 5.5 Iconography

**Icon Style:**
- Filled icons with 2px stroke outline
- Simple, recognizable silhouettes
- Consistent 2-3 color maximum per icon
- Pirate theme integrated where appropriate

**Icon Sizes:**
| Context | Size | Notes |
|---------|------|-------|
| **Ability Buttons** | 64x64px | In-game HUD |
| **Inventory Items** | 80x80px | Menus and shop |
| **Navigation** | 32x32px | Tab bars, small buttons |
| **Currency** | 24x24px | Inline with text |

**Icon Categories:**

1. **Abilities:** Action silhouette + damage type color
2. **Items:** Object silhouette, rarity border
3. **Navigation:** Simple universal symbols
4. **Currency:** Distinctive coin/gem shapes
5. **Status:** Warning, info, success indicators

### 5.6 Progress Bars & Indicators

**Health Bar:**
- Red gradient (`#FF4444` to `#CC0000`)
- Black outline, 2px
- Chunky, readable at distance
- Damage flash effect (white)

**XP/Progress Bar:**
- Blue gradient (`#4488FF` to `#2266CC`)
- Fill animation (smooth)
- Level number clearly visible

**Cooldown Indicators:**
- Radial sweep animation
- Darkened/desaturated ability icon
- Countdown number overlay

---

## 6. Visual Effects Style

### 6.1 VFX Philosophy

**Core Principles:**
1. **Readability:** Effects should enhance, not obscure gameplay
2. **Satisfaction:** Combat should feel impactful and rewarding
3. **Performance:** Effects must run smoothly on target devices
4. **Consistency:** Similar abilities have similar effect languages

### 6.2 Particle Style

**Shape Language:**
- Soft-edged particles (no hard pixel edges)
- Additive blending for magical effects
- Simple shapes: circles, stars, streaks
- Avoid complex sprite sheets when possible

**Animation Style:**
- Snappy attack effects (quick in, medium out)
- Smooth ambient effects (gentle loops)
- Satisfying impact effects (scale pop + fade)

### 6.3 Effect Categories

**Projectiles:**
- Core shape + trail + ambient particles
- Color-coded to damage type
- Size indicates power level
- Trail length indicates speed

**Impacts:**
- Initial flash/pop
- Outward particle burst
- Ground decal (temporary)
- Screen shake (subtle)

**Status Effects:**
- Persistent aura around character
- Color indicates effect type
- Intensity indicates stacks/duration
- Subtle to avoid visual noise

**Environmental:**
- Ambient particles (dust, leaves, water)
- Weather effects (rain, mist)
- Transition effects (zone entering)

### 6.4 Screen Effects

**Damage Taken:**
- Brief red vignette
- Screen shake (subtle)
- No persistent red overlay

**Low Health:**
- Subtle pulsing vignette
- Heartbeat audio cue
- Clear but not distracting

**Storm Zone:**
- Visibility reduction (fog/color grade)
- Edge screen effect
- Audio muffling

---

## 7. Technical Art Specifications

### 7.1 Polygon Budgets

| Asset Type | High LOD | Medium LOD | Low LOD |
|------------|----------|------------|---------|
| **Player Character** | 8,000 tris | 4,000 tris | 2,000 tris |
| **NPC Monster (Small)** | 2,000 tris | 1,000 tris | 500 tris |
| **NPC Monster (Large)** | 5,000 tris | 2,500 tris | 1,000 tris |
| **Environment Prop (Hero)** | 2,000 tris | 1,000 tris | 500 tris |
| **Environment Prop (Kit)** | 500 tris | 250 tris | 100 tris |
| **Building (Small)** | 5,000 tris | 2,500 tris | 1,000 tris |
| **Building (Large)** | 10,000 tris | 5,000 tris | 2,000 tris |
| **Weapon/Accessory** | 1,000 tris | 500 tris | - |

### 7.2 Texture Specifications

| Asset Type | Diffuse | Normal | Mask | Format |
|------------|---------|--------|------|--------|
| **Player Character** | 1024x1024 | 512x512 | 512x512 | ASTC 4x4 |
| **NPC Monster** | 512x512 | 256x256 | 256x256 | ASTC 4x4 |
| **Environment Hero** | 1024x1024 | 512x512 | - | ASTC 4x4 |
| **Environment Kit** | 512x512 | 256x256 | - | ASTC 6x6 |
| **UI Elements** | Power of 2 | - | - | ASTC 4x4 |
| **VFX Sprites** | 256x256 max | - | - | ASTC 4x4 |

**Texture Guidelines:**
- Use texture atlases for related props
- Trim unused alpha space
- No textures larger than 2048x2048
- Target 2 materials per character maximum

### 7.3 Animation Specifications

**Rig Requirements:**
| Rig Type | Bone Count | Notes |
|----------|------------|-------|
| **Player Character** | 50-60 bones | Full body + fingers (simplified) |
| **Monster (Simple)** | 20-30 bones | Basic locomotion |
| **Monster (Complex)** | 40-50 bones | More expressive |
| **Props (Animated)** | 1-10 bones | Minimal for performance |

**Animation Guidelines:**
- 30 FPS minimum for gameplay animations
- 24 FPS acceptable for ambient/background
- Blend trees for locomotion
- Root motion for abilities (optional)

### 7.4 LOD (Level of Detail) Strategy

**LOD Distances:**
| LOD Level | Distance | Use Case |
|-----------|----------|----------|
| **LOD0** | 0-20m | Full detail, hero shots |
| **LOD1** | 20-50m | Medium distance gameplay |
| **LOD2** | 50-100m | Distant, minimal detail |
| **LOD3/Culled** | 100m+ | Billboard or hidden |

**LOD Transition:**
- Smooth dithering transition (no pop)
- Test on lowest-end target device
- Characters: Always at least LOD1 visible

### 7.5 Draw Call Budgets

| Quality Level | Total Draw Calls | Characters | Environment | VFX |
|---------------|------------------|------------|-------------|-----|
| **Low** | 150 max | 60 | 60 | 30 |
| **Medium** | 250 max | 80 | 120 | 50 |
| **High** | 400 max | 100 | 200 | 100 |

**Optimization Strategies:**
- GPU instancing for repeated props
- Material atlasing for environment
- Dynamic batching for small objects
- Occlusion culling for indoor areas

---

## 8. Mobile Optimization Guidelines

### 8.1 Performance Targets

| Metric | Low-End | Mid-Range | High-End |
|--------|---------|-----------|----------|
| **Target FPS** | 30 stable | 30-60 | 60 stable |
| **Resolution** | 720p | 1080p | Native |
| **Memory Budget** | 1GB | 1.5GB | 2GB |
| **Texture Memory** | 256MB | 384MB | 512MB |
| **Battery Drain** | Moderate | Moderate | Accept higher |

### 8.2 Visual Clarity on Mobile

**Readability Checklist:**
- [ ] UI elements readable at arm's length
- [ ] Character silhouettes clear against any background
- [ ] Important gameplay elements visible on small screens
- [ ] Colors distinguishable for colorblind players
- [ ] Text size minimum 12pt at 1x scale

**Contrast Guidelines:**
- Foreground characters pop from background
- Interactive elements highlighted
- No critical gameplay in low-contrast areas
- HUD always visible regardless of scene

### 8.3 Scalable Quality Settings

**Visual Features by Quality:**

| Feature | Low | Medium | High |
|---------|-----|--------|------|
| Shadow Quality | None/Blob | Simple | Soft shadows |
| Post-Processing | None | Minimal | Full |
| VFX Particles | 50% | 75% | 100% |
| Texture Quality | 50% | 75% | 100% |
| LOD Bias | +2 | +1 | 0 |
| Foliage Density | 30% | 60% | 100% |
| Water Reflections | None | SSR lite | SSR full |
| Anti-Aliasing | None | FXAA | MSAA 2x |

### 8.4 Memory Management

**Asset Loading:**
- Stream textures based on camera distance
- Pool and reuse particle systems
- Unload unused areas/assets
- Compressed audio (OGG format)

**Texture Streaming:**
- Mipmaps enabled for all textures
- Lower mips on distant objects
- VFX textures lower priority

### 8.5 Battery & Thermal Considerations

**Design for Extended Play:**
- Avoid expensive real-time effects when possible
- Bake lighting wherever feasible
- Limit simultaneous particle systems
- Provide quality options for players

---

## 9. Reference Gallery

### 9.1 Primary Style References

**Fortnite (Epic Games)**
- Character proportions and silhouettes
- Vibrant, saturated color palette
- Clean, readable environments
- UI clarity and polish

**Sea of Thieves (Rare)**
- Pirate theme and aesthetics
- Water and ocean rendering
- Ship and nautical prop design
- Environmental mood and lighting

**World of Warcraft (Blizzard)**
- Stylized fantasy elements
- Exaggerated proportions
- Hand-painted texture style
- Readable at various zoom levels

**Brawl Stars (Supercell)**
- Mobile-optimized visuals
- Character design clarity
- UI for small screens
- VFX readability

### 9.2 Additional References

**Environment:**
- Oceanhorn 2 (tropical environments)
- Legends of Kingdom Rush (stylized fantasy)
- Moana (Disney) - color and mood

**Characters:**
- Overwatch (silhouette design)
- Splatoon (personality and color)
- Team Fortress 2 (readable classes)

**UI:**
- Clash Royale (mobile UI excellence)
- Legends of Runeterra (fantasy UI)
- Hearthstone (thematic elements)

### 9.3 Anti-References (What to Avoid)

**Do NOT Reference:**
- Realistic/photorealistic games
- Gritty survival games
- Horror themes or aesthetics
- Overly complex, noisy visuals
- Dark, desaturated color palettes

---

## 10. Appendices

### Appendix A: Color Accessibility

**Colorblind Considerations:**

| Colorblind Type | Problem Colors | Solution |
|-----------------|----------------|----------|
| **Protanopia** | Red-Green confusion | Use shapes + luminance difference |
| **Deuteranopia** | Red-Green confusion | Same as above |
| **Tritanopia** | Blue-Yellow confusion | Avoid pure blue/yellow combos for critical info |

**Accessibility Modes:**
- High contrast option
- Color filter adjustments
- Icon + color (never color alone)

### Appendix B: Asset Naming Conventions

**File Naming:**
```
[Category]_[AssetName]_[Variant]_[Type].[ext]

Examples:
CHAR_Pirate_Default_Diffuse.png
PROP_Barrel_Explosive_LOD1.fbx
VFX_Fireball_Impact_01.prefab
UI_Button_Primary_Pressed.png
```

**Categories:**
- `CHAR` - Characters
- `PROP` - Environment props
- `ENV` - Environment (terrain, buildings)
- `VFX` - Visual effects
- `UI` - User interface
- `ANIM` - Animations
- `SFX` - Sound effects

### Appendix C: Approval Checklist

Before an asset is considered complete:

**Character Assets:**
- [ ] Matches style guide proportions
- [ ] Silhouette test passed
- [ ] All LODs created and tested
- [ ] Animations work correctly
- [ ] Materials within budget
- [ ] Tested on lowest-end device

**Environment Assets:**
- [ ] Consistent with biome style
- [ ] LODs and billboards created
- [ ] Collision matches visual
- [ ] Tested in context
- [ ] Performance verified

**UI Assets:**
- [ ] All states created (normal, hover, pressed, disabled)
- [ ] Contrast requirements met
- [ ] Touch targets adequate
- [ ] Scales correctly across devices

### Appendix D: Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | Art Team | Initial style guide creation |

---

## Document Approval

| Role | Name | Signature | Date | Status |
|------|------|-----------|------|--------|
| Art Director | | | | Pending |
| Lead Artist | | | | Pending |
| Game Designer | | | | Pending |
| Technical Artist | | | | Pending |
| Producer | | | | Pending |

---

**End of Visual Style Guide**
