# NPC Monster Models Specification

**Document Version:** 1.0
**Created:** 2025-12-03
**Status:** Approved
**Task ID:** ART-006
**Priority:** P1
**Complexity:** M

---

## Table of Contents

1. [Overview](#1-overview)
2. [Design Guidelines](#2-design-guidelines)
3. [Monster Specifications](#3-monster-specifications)
4. [Technical Requirements](#4-technical-requirements)
5. [Animation Requirements](#5-animation-requirements)
6. [VFX Integration](#6-vfx-integration)
7. [Performance Optimization](#7-performance-optimization)
8. [Implementation Checklist](#8-implementation-checklist)

---

## 1. Overview

### Purpose

This document specifies the NPC monster models for PvE camps scattered across the battle royale map. Monsters provide an alternative XP source and ability drops for players who prefer PvE-focused gameplay or want to avoid early PvP encounters.

### Monster Roster

| Monster | Role | Triangle Budget | Spawn Frequency |
|---------|------|-----------------|-----------------|
| Skeleton Pirate | Basic Melee | 2,000 | High (Common) |
| Giant Crab | Medium Tank | 3,000 | Medium (Uncommon) |
| Cursed Sailor | Ranged Attacker | 2,500 | Medium (Uncommon) |
| Kraken Tentacle | Mini-Boss | 5,000 | Low (Rare) |

### Design Philosophy

- **Readable Threat:** Players should instantly understand monster danger level from appearance
- **Thematic Cohesion:** All monsters fit the pirate fantasy universe
- **Performance First:** Optimized for 20+ simultaneous monsters on mobile devices
- **Distinct Silhouettes:** Each monster type recognizable at distance
- **Rewarding Combat:** Defeating monsters feels satisfying

---

## 2. Design Guidelines

### Visual Hierarchy

```
Threat Level Indicators:
┌─────────────────────────────────────────────────────────┐
│ LOW THREAT (Skeleton Pirate)                            │
│ - Smaller size (0.8x player height)                     │
│ - Muted colors, weathered appearance                    │
│ - Simple silhouette                                     │
├─────────────────────────────────────────────────────────┤
│ MEDIUM THREAT (Giant Crab, Cursed Sailor)               │
│ - Medium size (1.0-1.2x player height)                  │
│ - More saturated colors, glowing elements               │
│ - More complex silhouette                               │
├─────────────────────────────────────────────────────────┤
│ HIGH THREAT (Kraken Tentacle)                           │
│ - Large size (2.0x+ player height)                      │
│ - Vibrant colors, prominent VFX                         │
│ - Dramatic, intimidating silhouette                     │
└─────────────────────────────────────────────────────────┘
```

### Color Coding System

| Threat Level | Primary Colors | Accent Colors |
|--------------|----------------|---------------|
| Common | Grays, browns, bone white | Rust, faded gold |
| Uncommon | Deep blues, purples | Teal glow, ghostly green |
| Rare/Boss | Deep purple, crimson | Bioluminescent cyan, gold |

### Stylization Guidelines

Following the game's art style (per ART-001 Visual Style Guide):
- Stylized proportions with exaggerated features
- Hand-painted texture style
- Bold shapes with clear silhouettes
- Mobile-optimized designs (readable at small sizes)
- No realistic gore (stylized damage effects only)

---

## 3. Monster Specifications

### 3.1 Skeleton Pirate

**Role:** Basic melee enemy that spawns in groups

**Visual Description:**
- Animated skeleton wearing tattered pirate clothing
- Partial clothing: torn vest, bandana, belt with empty sheath
- Carries rusty cutlass (weapon is part of model)
- Missing jaw or loose hanging jaw for character
- Glowing eye sockets (subtle orange/yellow glow)
- Barnacle and seaweed growth on bones (environmental detail)

**Proportions:**
- Height: 0.85x player character
- Build: Thin, bony frame with exaggerated rib cage
- Head: Slightly oversized skull for readability

**Color Palette:**
- Bones: Weathered ivory (#E8DCC4) with brown aging (#8B7355)
- Clothing: Faded brown (#6B4423) and gray (#5A5A5A)
- Accents: Rust orange (#B7410E) on metal, teal (#2A9D8F) on barnacles
- Eyes: Warm glow (#FFB347)

**Visual Variants:**
1. **Standard** - Basic skeleton with vest
2. **Captain's Guard** - Tricorn hat remnant, more intact clothing
3. **Deckhand** - Minimal clothing, rope belt, more weathered

**Key Features:**
- Jaw can be slightly animated (loose bone physics)
- Weapon glow when attacking
- Bones visible through torn clothing

**Triangle Budget:** 2,000

---

### 3.2 Giant Crab

**Role:** Medium tank enemy that guards treasure areas

**Visual Description:**
- Oversized crustacean with barnacle-encrusted shell
- Two large claws (asymmetric - one larger for attacks)
- Six legs for movement
- Small beady eyes on stalks
- Treasure debris embedded in shell (coins, gems, chain)
- Seaweed and coral growths on carapace

**Proportions:**
- Width: 1.5x player character width
- Height: 0.7x player character (low profile)
- Main claw: 0.8x body width

**Color Palette:**
- Shell: Deep red-orange (#C84630) with purple undertones (#6B3654)
- Underside: Pale cream (#F5E6D3)
- Barnacles: White (#F0F0F0) and gray (#9A9A9A)
- Eyes: Black with yellow highlight (#FFD700)
- Treasure accents: Gold (#FFD700), gem colors (varied)

**Visual Variants:**
1. **Reef Crab** - Coral-colored shell, more colorful
2. **Deep Sea Crab** - Darker shell, bioluminescent spots
3. **Treasure Guardian** - More embedded treasure, golden tint

**Key Features:**
- Asymmetric claws (left large, right small)
- Shell opens slightly when taking damage
- Eye stalks track nearby players
- Subtle shell shimmer (wet look shader)

**Triangle Budget:** 3,000

---

### 3.3 Cursed Sailor

**Role:** Ranged ghostly enemy that fires spectral projectiles

**Visual Description:**
- Ghostly apparition of a drowned sailor
- Translucent body with ethereal glow
- Tattered naval uniform (historical pirate era)
- Carries spectral flintlock pistol
- Chains wrapped around torso (weighted, drowned at sea)
- Seaweed and kelp trailing from body
- Face partially visible through ghostly mist

**Proportions:**
- Height: 1.1x player character
- Build: Gaunt, elongated limbs
- Floats slightly above ground (no legs visible below knee)

**Color Palette:**
- Body: Translucent blue-white (#C5E8F7) with teal core (#48A9A6)
- Clothing: Faded navy (#1E3A5F) visible through transparency
- Chains: Dark iron (#3D3D3D) with rust (#8B4513)
- Eyes: Hollow glow, pale yellow (#FFFACD)
- Weapon: Ghostly silver-blue (#B8D4E3)

**Visual Variants:**
1. **Drowned Deckhand** - Simple clothing, single chain
2. **Phantom Officer** - Naval coat remnants, more imposing
3. **Vengeful Gunner** - Dual pistols, more aggressive stance

**Key Features:**
- Translucent material shader (see-through effect)
- Constant subtle particle emission (mist/ectoplasm)
- Floats/hovers instead of walking
- Chains have subtle physics/sway

**Triangle Budget:** 2,500

---

### 3.4 Kraken Tentacle (Mini-Boss)

**Role:** Large animated obstacle/mini-boss at special locations

**Visual Description:**
- Massive tentacle emerging from water/ground
- Covered in suction cups of varying sizes
- Bioluminescent patterns along length
- Scarred and battle-worn (old harpoon wounds, rope marks)
- Smaller barnacles and sea life attached
- Tip has larger, more dangerous-looking suction cups

**Proportions:**
- Height: 3.0x player character (above ground portion)
- Width: 0.5x height at base, tapering to tip
- Suction cups: Varied sizes (0.1x to 0.3x tentacle width)

**Color Palette:**
- Main body: Deep purple (#4A235A) to crimson (#8B0000) gradient
- Underside: Pale pink-gray (#D4A5A5)
- Suction cups: Darker purple (#2C0A37) with pink centers (#FF9999)
- Bioluminescence: Cyan (#00FFFF) and electric blue (#7DF9FF)
- Scars: Pale white (#F5F5F5)

**Animation States:**
1. **Idle** - Slow swaying, suction cups pulsing
2. **Alert** - Rises higher, faster movement
3. **Attack** - Slam down, sweep, or grab motion
4. **Damaged** - Recoil, color flash
5. **Defeat** - Sinks back down, bioluminescence fades

**Key Features:**
- Bioluminescent pulse VFX (breathing effect)
- Wet/slimy surface shader
- Dramatic emergence animation from spawn point
- Suction cup animation (independent subtle movement)
- Water splash VFX at base

**Triangle Budget:** 5,000

---

## 4. Technical Requirements

### Polygon Budgets

| Monster | High LOD | Medium LOD | Low LOD | Distance Threshold |
|---------|----------|------------|---------|-------------------|
| Skeleton Pirate | 2,000 | 1,000 | 500 | 0-20m / 20-40m / 40m+ |
| Giant Crab | 3,000 | 1,500 | 750 | 0-25m / 25-50m / 50m+ |
| Cursed Sailor | 2,500 | 1,200 | 600 | 0-25m / 25-50m / 50m+ |
| Kraken Tentacle | 5,000 | 2,500 | 1,200 | 0-30m / 30-60m / 60m+ |

### Texture Specifications

| Monster | Diffuse | Normal | Emission | Atlas |
|---------|---------|--------|----------|-------|
| Skeleton Pirate | 1024x1024 | 512x512 | 256x256 | Shared |
| Giant Crab | 1024x1024 | 512x512 | - | Unique |
| Cursed Sailor | 1024x1024 | 512x512 | 512x512 | Shared Ghost |
| Kraken Tentacle | 2048x2048 | 1024x1024 | 512x512 | Unique |

### Material Configuration

```
Standard Monster Material Setup:
├── Skeleton Pirate
│   ├── Body Material (bone + clothing combined)
│   └── Weapon Material (metallic shader)
├── Giant Crab
│   ├── Shell Material (wet subsurface)
│   └── Underside Material (standard PBR)
├── Cursed Sailor
│   ├── Ghost Material (translucent, additive)
│   ├── Chains Material (metallic)
│   └── Weapon Material (ghost variant)
└── Kraken Tentacle
    ├── Tentacle Material (wet subsurface)
    ├── Suction Cup Material (soft, translucent)
    └── Bioluminescence Material (emissive, animated)
```

### Rigging Requirements

| Monster | Bone Count | IK Chains | Physics Bones |
|---------|------------|-----------|---------------|
| Skeleton Pirate | 35 | 2 (arms) | 3 (jaw, belt, bandana) |
| Giant Crab | 25 | 6 (legs) | 2 (eye stalks) |
| Cursed Sailor | 30 | 2 (arms) | 5 (chains, seaweed) |
| Kraken Tentacle | 20 | 1 (tentacle) | 8 (suction cups) |

---

## 5. Animation Requirements

### Animation List Per Monster

#### Skeleton Pirate Animations

| Animation | Duration | Loop | Priority |
|-----------|----------|------|----------|
| Idle | 3.0s | Yes | Low |
| Walk | 1.0s cycle | Yes | Normal |
| Run | 0.7s cycle | Yes | Normal |
| Attack_Slash | 0.8s | No | High |
| Attack_Thrust | 0.6s | No | High |
| Hurt | 0.4s | No | High |
| Death | 1.5s | No | Critical |
| Spawn | 1.2s | No | Critical |

#### Giant Crab Animations

| Animation | Duration | Loop | Priority |
|-----------|----------|------|----------|
| Idle | 4.0s | Yes | Low |
| Walk_Forward | 1.2s cycle | Yes | Normal |
| Walk_Sideways | 1.0s cycle | Yes | Normal |
| Attack_Claw | 1.0s | No | High |
| Attack_Snap | 0.7s | No | High |
| Guard (shell close) | 0.5s | No | High |
| Hurt | 0.5s | No | High |
| Death | 2.0s | No | Critical |

#### Cursed Sailor Animations

| Animation | Duration | Loop | Priority |
|-----------|----------|------|----------|
| Idle_Float | 3.5s | Yes | Low |
| Move_Drift | 1.0s cycle | Yes | Normal |
| Attack_Shoot | 1.2s | No | High |
| Attack_Ghostly_Wail | 1.5s | No | High |
| Hurt | 0.4s | No | High |
| Death_Disperse | 2.0s | No | Critical |
| Spawn_Materialize | 1.5s | No | Critical |

#### Kraken Tentacle Animations

| Animation | Duration | Loop | Priority |
|-----------|----------|------|----------|
| Idle_Sway | 5.0s | Yes | Low |
| Alert_Rise | 1.5s | No | Normal |
| Attack_Slam | 1.8s | No | High |
| Attack_Sweep | 2.0s | No | High |
| Attack_Grab | 1.5s | No | High |
| Hurt_Recoil | 0.6s | No | High |
| Death_Sink | 3.0s | No | Critical |
| Emerge | 2.5s | No | Critical |

### Animation Quality Guidelines

```
Animation Principles:
- Snappy attacks with clear wind-up (telegraph) for player reaction
- Exaggerated movements for mobile screen readability
- Smooth loops with no visible pops or hitches
- Clear hit frames for damage sync
- Death animations satisfying but not too long
```

---

## 6. VFX Integration

### Per-Monster VFX Budget

| Monster | Particle Budget | Draw Call Budget |
|---------|-----------------|------------------|
| Skeleton Pirate | 5 | 1 |
| Giant Crab | 8 | 1 |
| Cursed Sailor | 20 | 2 |
| Kraken Tentacle | 30 | 3 |

### VFX Specifications

#### Skeleton Pirate VFX
- **Eye Glow:** 2 point lights (baked into texture, no runtime lights)
- **Weapon Trail:** Simple slash arc on attack (5 particles)
- **Death:** Bone scatter particles, dust poof

#### Giant Crab VFX
- **Shell Shine:** Wet shader reflection (no particles)
- **Attack:** Sand/debris kick-up (5 particles)
- **Death:** Shell crack effect, treasure scatter (8 particles)

#### Cursed Sailor VFX
- **Ambient Mist:** Constant ghost trail (10 particles)
- **Projectile:** Spectral bullet with trail (5 particles)
- **Death:** Dispersion/fade effect (15 particles)

#### Kraken Tentacle VFX
- **Bioluminescence:** Pulsing glow (shader-based + 5 particles)
- **Water Splash:** Base emergence splash (10 particles)
- **Attack Impact:** Ground slam dust/debris (15 particles)
- **Death:** Ink cloud, water disturbance (20 particles)

---

## 7. Performance Optimization

### Target Performance

```
Performance Targets:
- 20+ monsters on screen simultaneously
- Maintain 30 FPS on low-end devices
- Maintain 60 FPS on high-end devices
- Memory budget: 50MB for all monster assets combined
```

### Optimization Strategies

#### LOD System
- Aggressive LOD transitions based on distance
- Disable cloth/physics on Low LOD
- Simplified animations at distance (reduced bone influence)

#### Instancing
- GPU instancing for Skeleton Pirates (most common)
- Shared materials where possible
- Texture atlasing for common monsters

#### Culling
- Frustum culling (standard)
- Occlusion culling for monsters behind terrain
- Animation culling when off-screen

#### Memory Management
- Shared animation rigs where skeleton structure allows
- Compressed textures (ASTC for mobile)
- Streaming for Kraken Tentacle (load on demand)

### Performance Validation Matrix

| Device Tier | Target FPS | Max Monsters | LOD Level |
|-------------|------------|--------------|-----------|
| Low-end | 30 | 15 | Low |
| Mid-range | 30-60 | 20 | Medium |
| High-end | 60 | 25+ | High |

---

## 8. Implementation Checklist

### Pre-Production

- [x] Monster concepts approved
- [x] Animation lists finalized
- [x] Technical budgets defined
- [x] Art style reference collected
- [x] VFX requirements documented

### Production - Per Monster

#### Skeleton Pirate
- [ ] High-poly sculpt
- [ ] Game mesh (2,000 triangles)
- [ ] UV mapping
- [ ] Texturing (diffuse, normal, emission)
- [ ] Rigging (35 bones)
- [ ] Animation set (8 animations)
- [ ] LOD generation (3 levels)
- [ ] VFX integration
- [ ] Variants (3 visual variants)

#### Giant Crab
- [ ] High-poly sculpt
- [ ] Game mesh (3,000 triangles)
- [ ] UV mapping
- [ ] Texturing (diffuse, normal)
- [ ] Rigging (25 bones)
- [ ] Animation set (8 animations)
- [ ] LOD generation (3 levels)
- [ ] VFX integration
- [ ] Variants (3 visual variants)

#### Cursed Sailor
- [ ] High-poly sculpt
- [ ] Game mesh (2,500 triangles)
- [ ] UV mapping
- [ ] Texturing (diffuse, normal, emission)
- [ ] Translucent shader setup
- [ ] Rigging (30 bones)
- [ ] Animation set (7 animations)
- [ ] LOD generation (3 levels)
- [ ] VFX integration (ghost effects)
- [ ] Variants (3 visual variants)

#### Kraken Tentacle
- [ ] High-poly sculpt
- [ ] Game mesh (5,000 triangles)
- [ ] UV mapping
- [ ] Texturing (diffuse, normal, emission)
- [ ] Bioluminescence shader
- [ ] Rigging (20 bones)
- [ ] Animation set (8 animations)
- [ ] LOD generation (3 levels)
- [ ] VFX integration (water, glow)

### Quality Assurance

- [ ] All monsters display correctly in-game
- [ ] Animations play without errors
- [ ] LOD transitions are smooth
- [ ] Performance validated (20+ on screen)
- [ ] No visual glitches or clipping
- [ ] VFX perform within budget
- [ ] AI correctly uses all animations
- [ ] Hitboxes match visual models
- [ ] Audio sync with animations

### Integration

- [ ] Models imported into engine
- [ ] AI behavior linked to animations
- [ ] Spawn system uses correct prefabs
- [ ] Health bars display correctly
- [ ] Damage numbers appear properly
- [ ] Loot drop system connected
- [ ] XP reward system connected

---

## Appendix A: Monster Behavior Reference

### Combat Ranges

| Monster | Aggro Range | Attack Range | Leash Range |
|---------|-------------|--------------|-------------|
| Skeleton Pirate | 10m | 2m | 30m |
| Giant Crab | 8m | 3m | 25m |
| Cursed Sailor | 15m | 12m | 35m |
| Kraken Tentacle | 20m | 8m | Fixed |

### Reward Values

| Monster | XP Reward | Ability Drop Chance | Gold Drop |
|---------|-----------|---------------------|-----------|
| Skeleton Pirate | 20 | 10% | 5-15 |
| Giant Crab | 50 | 30% | 20-40 |
| Cursed Sailor | 40 | 25% | 15-30 |
| Kraken Tentacle | 150 | 100% (Rare+) | 75-150 |

---

## Appendix B: Audio Integration Notes

Each monster requires the following audio hooks:
- Idle ambient sound (loop)
- Footsteps/movement sounds
- Attack sound(s)
- Hurt/damage sound
- Death sound
- Aggro/alert sound

Audio specifications detailed in ART-027 (Sound Effects Specification).

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-03 | Art Team | Initial specification |

---

*This document is part of the Plunderstorm Mobile Art Production Bible. All specifications are subject to iteration based on technical testing and gameplay balance.*
