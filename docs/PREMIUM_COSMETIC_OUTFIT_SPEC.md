# Premium Cosmetic Outfit Set Specification

**Document Version:** 1.0
**Created:** 2025-12-03
**Status:** Approved
**Task ID:** ART-005
**Priority:** P1
**Complexity:** L

---

## Table of Contents

1. [Overview](#1-overview)
2. [Design Guidelines](#2-design-guidelines)
3. [Outfit Specifications](#3-outfit-specifications)
4. [Technical Requirements](#4-technical-requirements)
5. [VFX-Enhanced Outfits](#5-vfx-enhanced-outfits)
6. [Rarity and Pricing](#6-rarity-and-pricing)
7. [Implementation Checklist](#7-implementation-checklist)

---

## 1. Overview

### Purpose

This document specifies the initial set of 10 premium cosmetic outfits for the soft launch monetization strategy. These outfits represent the highest quality cosmetics in the game and serve as aspirational content for players.

### Design Philosophy

- **Diversity:** Each outfit targets different player fantasies and aesthetic preferences
- **Quality:** Premium outfits should feel significantly more elaborate than the default outfit
- **Fairness:** Cosmetic-only changes with no gameplay advantages
- **Visibility:** Outfits should be distinctive and recognizable at distance
- **Theme Cohesion:** All outfits fit within the pirate fantasy universe

### Target Audience Segments

| Segment | Targeted Outfits |
|---------|------------------|
| Classic Fantasy Fans | Dread Captain, Navy Admiral |
| Horror/Spooky Enthusiasts | Ghost Pirate, Skeleton Crew |
| Whimsical/Fun Players | Sea Witch/Wizard, Mermaid/Merman |
| Adventure Seekers | Kraken Hunter, Treasure Seeker |
| Style-Conscious | Tropical Swashbuckler, Steampunk Corsair |

---

## 2. Design Guidelines

### Visual Quality Standards

```
Premium Quality Checklist:
- Multiple color variations or accent options
- Unique silhouette elements (hats, capes, accessories)
- High-detail textures with hand-painted elements
- Animated elements (cloth physics, glowing effects)
- Distinctive sound effects on movement (optional)
```

### Technical Constraints

| Specification | Requirement |
|---------------|-------------|
| Max Triangles | 10,000 (High LOD) |
| Texture Resolution | 2048x2048 (diffuse), 1024x1024 (normal/specular) |
| Material Count | Max 3 materials per outfit |
| Bone Count | Must work with standard 50-bone rig |
| VFX Particles | Max 50 per VFX-enhanced outfit |

### Animation Compatibility

All outfits must:
- Work with all existing character animations without clipping
- Support all emotes and ability animations
- Have proper cloth/accessory physics that don't interfere with gameplay
- Maintain readable silhouette during all animation states

### Hitbox Requirements

**CRITICAL:** All outfits must use the identical hitbox as the default character. No outfit may provide:
- Smaller collision volume
- Visual mismatch with collision (e.g., large visual, small hitbox)
- Any competitive advantage through silhouette manipulation

---

## 3. Outfit Specifications

### 3.1 Dread Captain

**Theme:** The fearsome pirate captain who commands fear on the seas

**Visual Description:**
- Long black leather coat with gold trim and brass buttons
- Tricorn hat with skull emblem and feather plume
- Eye patch and scarred face option
- High leather boots with silver buckles
- Ornate cutlass sheath on hip
- Dark beard/facial hair options

**Color Variants:**
1. **Blackheart** - Primary black with gold accents
2. **Crimson Fury** - Dark red coat with silver trim
3. **Midnight Storm** - Deep navy blue with bronze details

**Key Elements:**
- Flowing coat with cloth physics
- Intimidating silhouette with hat and coat
- Metallic shine on buttons and buckles

**Rarity:** Legendary

---

### 3.2 Tropical Swashbuckler

**Theme:** Colorful Caribbean pirate with vibrant island flair

**Visual Description:**
- Open colorful vest with tropical patterns
- Loose-fitting pants with sash belt
- Bandana headwrap with beads
- Barefoot or simple sandals
- Multiple earrings and necklaces
- Feathered accessories

**Color Variants:**
1. **Sunset Isles** - Orange, yellow, and red tones
2. **Ocean Paradise** - Turquoise, teal, and white
3. **Jungle Fever** - Green, gold, and brown

**Key Elements:**
- Bright saturated colors that stand out
- Layered accessories with subtle animation
- Relaxed, carefree aesthetic

**Rarity:** Epic

---

### 3.3 Ghost Pirate

**Theme:** Spectral undead corsair from beyond the grave

**Visual Description:**
- Translucent ghostly skin with ethereal glow
- Tattered period naval uniform
- Ghostly tricorn hat with spectral flames
- Chains and shackles on wrists
- Hollow glowing eyes
- Wispy particle effects trailing from body

**VFX Elements:**
- Constant subtle ghost particle emission
- Ethereal glow around character outline
- Footstep VFX (ghostly mist)
- Death VFX enhanced with spectral explosion

**Color Variants:**
1. **Phantom Blue** - Classic blue-white ghost aesthetic
2. **Cursed Green** - Eerie green undead glow
3. **Wraith Gray** - Pale silver ethereal appearance

**Key Elements:**
- Translucent material shader
- Particle effects integrated with model
- Glowing eye sockets

**Rarity:** Legendary

**Performance Note:** VFX particle count must stay under 30 for mobile optimization

---

### 3.4 Sea Witch/Wizard

**Theme:** Magical pirate with oceanic sorcery powers

**Visual Description:**
- Flowing robes with oceanic patterns (waves, tentacles)
- Staff or wand accessory (cosmetic only)
- Coral crown or seaweed headdress
- Bioluminescent jewelry
- Mystic runes on clothing
- Fish-scale armor accents

**VFX Elements:**
- Subtle magical aura around hands
- Bioluminescent glow pulses
- Small floating magical particles

**Color Variants:**
1. **Deep Abyss** - Dark purple and black with glowing teal accents
2. **Coral Enchanter** - Pink, coral, and seafoam green
3. **Storm Caller** - Dark blue with lightning white accents

**Key Elements:**
- Magical aesthetic with ocean theme
- Distinctive robe silhouette
- Subtle glow effects on accessories

**Rarity:** Legendary

---

### 3.5 Navy Admiral

**Theme:** Rival faction military commander

**Visual Description:**
- Pristine white and blue naval uniform
- Epaulettes with gold fringe
- Bicorne hat with naval insignia
- Polished black boots
- Ceremonial sword in scabbard
- Medals and decorations on chest

**Color Variants:**
1. **Royal Navy** - White and navy blue with gold
2. **Imperial Black** - Black uniform with silver trim
3. **Commodore Crimson** - Red and white formal uniform

**Key Elements:**
- Clean, disciplined aesthetic contrasting pirate theme
- Represents the "villain" faction visually
- Military precision in details

**Rarity:** Epic

---

### 3.6 Kraken Hunter

**Theme:** Legendary monster slayer with trophies

**Visual Description:**
- Heavy leather armor with sea monster scales
- Tentacle trophies worn as accessories
- Harpoon holster on back (cosmetic)
- Scarred, weathered appearance
- Eye made from kraken tooth (cosmetic)
- Heavy boots with monster-hide wrappings

**Color Variants:**
1. **Deep Hunter** - Dark brown leather with purple tentacle accents
2. **Storm Chaser** - Gray and blue with silver hardware
3. **Abyss Walker** - Black with bioluminescent trophy parts

**Key Elements:**
- Rugged, battle-worn aesthetic
- Monster trophy accessories
- Intimidating but heroic silhouette

**Rarity:** Epic

---

### 3.7 Treasure Seeker

**Theme:** Wealthy pirate adorned with plundered riches

**Visual Description:**
- Opulent clothing with gold thread embroidery
- Multiple gold chains and medallions
- Gem-encrusted accessories
- Crown or tiara made of gold coins
- Rings on every finger
- Money pouch overflowing with coins

**VFX Elements:**
- Subtle gold sparkle effect
- Coin particle trail when running (very subtle)

**Color Variants:**
1. **Golden Glory** - White and gold primary theme
2. **Ruby Rich** - Red velvet with gold and rubies
3. **Emerald Empire** - Green silk with gold and emeralds

**Key Elements:**
- Maximum bling aesthetic
- Represents wealth and success
- Aspirational for new players

**Rarity:** Legendary

---

### 3.8 Mermaid/Merman

**Theme:** Aquatic fantasy hybrid character

**Visual Description:**
- Scale-textured skin on arms and torso
- Seashell accessories and jewelry
- Fish fin elements on arms/legs (cosmetic, not actual fins)
- Iridescent coloring on scales
- Trident back accessory (cosmetic)
- Pearl decorations

**VFX Elements:**
- Subtle water droplet particles
- Iridescent shimmer on scales
- Small bubble effects occasionally

**Color Variants:**
1. **Ocean Blue** - Blue and silver scales
2. **Coral Reef** - Orange and pink tropical coloring
3. **Deep Sea** - Dark purple and green bioluminescent

**Key Elements:**
- Fantasy transformation theme
- Iridescent material shaders
- Aquatic accessories

**Rarity:** Legendary

**Note:** Character still has humanoid legs for animation compatibility

---

### 3.9 Skeleton Crew

**Theme:** Undead skeletal pirate

**Visual Description:**
- Exposed skeleton with tattered clothing
- Glowing soul in ribcage (VFX)
- Pirate hat with holes and wear
- Bones visible through torn sleeves
- Rusty cutlass at hip
- Jaw can be slightly animated/loose

**VFX Elements:**
- Glowing soul ember in chest
- Occasional bone rattle particle
- Ghostly eye glow

**Color Variants:**
1. **Bleached Bones** - White bones with gray tattered clothes
2. **Cursed Gold** - Golden bone tint with regal torn clothing
3. **Ancient Mariner** - Aged yellow bones with barnacle growth

**Key Elements:**
- Undead horror aesthetic
- Full skeleton visibility (stylized, not realistic)
- Tattered clothing shows bone underneath

**Rarity:** Epic

---

### 3.10 Steampunk Corsair

**Theme:** Clockwork and brass technology meets pirate

**Visual Description:**
- Brass goggles on forehead or eyes
- Clockwork arm or leg augmentation (cosmetic)
- Leather trench coat with brass fittings
- Steam-powered backpack accessory
- Mechanical eye option
- Gears and cog accessories

**VFX Elements:**
- Occasional steam puff from backpack
- Glowing mechanical eye
- Small gear rotation on accessories

**Color Variants:**
1. **Brass Baron** - Brown leather with brass and copper
2. **Silver Artificer** - Black leather with silver mechanisms
3. **Emerald Engineer** - Green-tinted goggles and accents

**Key Elements:**
- Victorian steampunk aesthetic
- Mechanical moving parts (subtle animation)
- Unique technological twist on pirate theme

**Rarity:** Epic

---

## 4. Technical Requirements

### Asset Specifications

| Component | High LOD | Medium LOD | Low LOD |
|-----------|----------|------------|---------|
| Triangles | 10,000 | 5,000 | 2,500 |
| View Distance | 0-30m | 30-60m | 60m+ |
| Texture Size | 2048px | 1024px | 512px |

### Material Configuration

```
Standard Premium Outfit Material Setup:
├── Main Body Material
│   ├── Diffuse Map (2048x2048)
│   ├── Normal Map (1024x1024)
│   ├── Metallic/Roughness Map (1024x1024)
│   └── Emission Map (optional, 512x512)
├── Accessory Material
│   ├── Diffuse Map (1024x1024)
│   └── Normal Map (512x512)
└── VFX Material (if applicable)
    ├── Particle Texture Atlas (512x512)
    └── Shader Configuration
```

### Animation Requirements

Each outfit must pass validation against:
- All 15 locomotion animations
- All 10 combat animations
- All 8 utility animations
- All 10 emote animations (existing)

### Cloth Physics Configuration

| Element | Simulation | Collision |
|---------|------------|-----------|
| Coats/Robes | Full cloth sim | Body, legs |
| Capes | Simplified cloth | Back only |
| Belts/Sashes | Bone-based | None |
| Hair/Feathers | Bone-based | Head |

---

## 5. VFX-Enhanced Outfits

### VFX Budget Allocation

| Outfit | Particle Budget | Effect Types |
|--------|-----------------|--------------|
| Ghost Pirate | 30 | Constant glow, trail |
| Sea Witch/Wizard | 20 | Aura, glow pulses |
| Treasure Seeker | 15 | Sparkle, coin trail |
| Mermaid/Merman | 20 | Droplets, bubbles, shimmer |
| Skeleton Crew | 15 | Soul ember, eye glow |
| Steampunk Corsair | 15 | Steam puffs, gear rotation |

### VFX Performance Guidelines

```
Mobile Performance Rules:
- Total particles: Max 50 per outfit
- Draw calls: Max 2 additional for VFX
- Overdraw: Minimal transparent surfaces
- Update rate: VFX can update at 15fps
- Distance culling: VFX hidden beyond 30m
```

### Non-Distracting VFX Requirements

VFX must NOT:
- Obscure the player's own vision
- Create visual noise that distracts from combat
- Be mistakable for ability effects
- Provide any visibility advantage in smoke/fog
- Flash or strobe rapidly (accessibility)

---

## 6. Rarity and Pricing

### Rarity Distribution

| Rarity | Count | Price Range (Doubloons) |
|--------|-------|------------------------|
| Legendary | 5 | 1,800 - 2,000 |
| Epic | 5 | 1,200 - 1,500 |

### Individual Outfit Pricing

| Outfit | Rarity | Price (Doubloons) | USD Equivalent |
|--------|--------|-------------------|----------------|
| Dread Captain | Legendary | 2,000 | ~$16 |
| Ghost Pirate | Legendary | 2,000 | ~$16 |
| Sea Witch/Wizard | Legendary | 1,800 | ~$14 |
| Treasure Seeker | Legendary | 1,800 | ~$14 |
| Mermaid/Merman | Legendary | 1,800 | ~$14 |
| Tropical Swashbuckler | Epic | 1,200 | ~$10 |
| Navy Admiral | Epic | 1,200 | ~$10 |
| Kraken Hunter | Epic | 1,500 | ~$12 |
| Skeleton Crew | Epic | 1,500 | ~$12 |
| Steampunk Corsair | Epic | 1,500 | ~$12 |

### Bundle Opportunities

**Undead Collection Bundle:**
- Ghost Pirate + Skeleton Crew
- Bundle Price: 3,000 Doubloons (15% discount)

**Fantasy Creatures Bundle:**
- Sea Witch/Wizard + Mermaid/Merman
- Bundle Price: 3,000 Doubloons (17% discount)

**Complete Premium Collection:**
- All 10 outfits
- Bundle Price: 12,000 Doubloons (25% discount)

---

## 7. Implementation Checklist

### Pre-Production Phase

- [x] Concept art for all 10 outfits
- [x] Color variant concepts approved
- [x] Technical specifications finalized
- [x] VFX concepts for enhanced outfits
- [x] Pricing and rarity approved

### Production Phase

For each outfit:
- [ ] High-poly sculpt
- [ ] Low-poly game mesh
- [ ] UV mapping and texturing
- [ ] Rigging to base skeleton
- [ ] Cloth/physics setup
- [ ] LOD generation
- [ ] Color variant texture sets
- [ ] VFX implementation (if applicable)

### Quality Assurance

- [ ] Animation compatibility testing (all animations)
- [ ] Clipping check across all emotes
- [ ] Performance testing on low-end devices
- [ ] VFX performance validation
- [ ] Hitbox verification (no advantage)
- [ ] Distance readability check
- [ ] Color variant consistency

### Store Integration

- [ ] Store preview rendering
- [ ] 3D model viewer integration
- [ ] Purchase flow testing
- [ ] Bundle configuration
- [ ] Price point validation
- [ ] Icon generation for inventory

### Launch Readiness

- [ ] All 10 outfits in game and purchasable
- [ ] No visual bugs or clipping issues
- [ ] Performance verified on all target devices
- [ ] Store displays correctly
- [ ] Analytics tracking purchase events
- [ ] Player feedback collection system ready

---

## Appendix A: Reference Imagery

### Mood Boards (To Be Created)

1. **Dark/Intimidating Set:** Dread Captain, Ghost Pirate, Skeleton Crew
2. **Colorful/Whimsical Set:** Tropical Swashbuckler, Mermaid/Merman, Treasure Seeker
3. **Unique Theme Set:** Navy Admiral, Sea Witch, Kraken Hunter, Steampunk Corsair

### Art Style References

- Fortnite character skin quality and silhouette clarity
- Sea of Thieves pirate costume design language
- World of Warcraft stylized character proportions
- Mobile Legends: Bang Bang skin detail level

---

## Appendix B: Localization Notes

Outfit names should be localized for all supported languages:

| English | Localization Notes |
|---------|-------------------|
| Dread Captain | Emphasis on fear/intimidation |
| Tropical Swashbuckler | Emphasis on colorful/adventurous |
| Ghost Pirate | Emphasis on supernatural/spectral |
| Sea Witch/Wizard | Gender-appropriate variant names |
| Navy Admiral | Military ranking terminology |
| Kraken Hunter | Monster hunter theme |
| Treasure Seeker | Wealth/fortune theme |
| Mermaid/Merman | Gender-appropriate variant names |
| Skeleton Crew | Wordplay on nautical term |
| Steampunk Corsair | Technology + pirate fusion |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-03 | Art Team | Initial specification |

---

*This document is part of the Plunderstorm Mobile Art Production Bible. All specifications are subject to iteration based on technical testing and player feedback.*
