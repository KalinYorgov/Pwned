# Technical Art Specifications
# Plunderstorm Mobile - Asset Production Guidelines

**Document Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved
**Task ID:** ART-002

---

## Table of Contents

1. [Overview](#1-overview)
2. [Target Platforms & Performance](#2-target-platforms--performance)
3. [Character Specifications](#3-character-specifications)
4. [Environment Specifications](#4-environment-specifications)
5. [Texture Specifications](#5-texture-specifications)
6. [LOD System](#6-lod-system)
7. [Animation Specifications](#7-animation-specifications)
8. [VFX Specifications](#8-vfx-specifications)
9. [UI Art Specifications](#9-ui-art-specifications)
10. [Audio Specifications](#10-audio-specifications)
11. [Quality Tiers](#11-quality-tiers)
12. [Performance Budgets](#12-performance-budgets)
13. [Asset Pipeline](#13-asset-pipeline)
14. [Validation Checklist](#14-validation-checklist)
15. [Appendix](#15-appendix)

---

## 1. Overview

### 1.1 Purpose

This document defines the technical specifications for all art assets in Plunderstorm Mobile. Adherence to these specifications ensures:

- Consistent visual quality across all assets
- Optimal performance on target mobile devices
- Efficient memory usage
- Smooth gameplay experience at 30-60 FPS

### 1.2 Scope

These specifications apply to:
- Character models and cosmetics
- Environment assets (terrain, props, buildings)
- Visual effects (particles, shaders)
- User interface elements
- Animations
- Audio assets

### 1.3 Compliance

All assets MUST meet these specifications before being integrated into the game. Assets exceeding budgets will be rejected and require optimization.

### 1.4 Reference Documents

| Document | Content |
|----------|---------|
| [VISUAL_STYLE_GUIDE.md](VISUAL_STYLE_GUIDE.md) | Art style direction |
| [ART_STYLE_EXPLORATION.md](ART_STYLE_EXPLORATION.md) | Style selection rationale |
| [GAME_DESIGN_DOCUMENT.md](GAME_DESIGN_DOCUMENT.md) | Technical requirements overview |

---

## 2. Target Platforms & Performance

### 2.1 Target Devices

| Tier | Example Devices | RAM | GPU | Market Share |
|------|-----------------|-----|-----|--------------|
| **Low-End** | iPhone 8, Samsung A21 | 2-3 GB | Apple A11, Mali-G52 | ~30% |
| **Mid-Range** | iPhone 11, Samsung S20 FE | 4-6 GB | Apple A13, Adreno 650 | ~50% |
| **High-End** | iPhone 14, Samsung S23 | 6-8 GB | Apple A16, Adreno 740 | ~20% |

### 2.2 Minimum Specifications

| Metric | Requirement |
|--------|-------------|
| **OS** | iOS 13+ / Android 8.0+ |
| **RAM** | 3 GB minimum |
| **Storage** | 2 GB install size target |
| **GPU** | OpenGL ES 3.0 / Metal |

### 2.3 Performance Targets

| Metric | Low-End | Mid-Range | High-End |
|--------|---------|-----------|----------|
| **Target FPS** | 30 stable | 30-60 | 60 stable |
| **Resolution** | 720p | 1080p | Native (up to 1440p) |
| **Frame Time** | 33ms | 16-33ms | 16ms |
| **Memory Budget** | 1.0 GB | 1.5 GB | 2.0 GB |

### 2.4 Scene Complexity Limits

**Maximum simultaneous on-screen elements:**

| Element | Low-End | Mid-Range | High-End |
|---------|---------|-----------|----------|
| Player Characters | 60 | 60 | 60 |
| NPC Monsters | 20 | 30 | 40 |
| Active VFX | 30 | 50 | 80 |
| Dynamic Lights | 1 | 2 | 4 |
| Shadow Casters | 0 | 10 | 30 |

---

## 3. Character Specifications

### 3.1 Player Character Model

| Specification | LOD0 (High) | LOD1 (Medium) | LOD2 (Low) |
|---------------|-------------|---------------|------------|
| **Triangle Count** | 8,000 | 4,000 | 2,000 |
| **Bone Count** | 55 | 55 | 30 |
| **Materials** | 2 | 2 | 1 |
| **Texture Resolution** | 1024x1024 | 512x512 | 256x256 |
| **LOD Distance** | 0-20m | 20-50m | 50m+ |

**Mesh Requirements:**
- Clean quad-based topology for deformation
- No N-gons or triangulated faces in source mesh
- Proper edge flow around joints
- Single mesh per character (no separate parts)
- Consistent UV layout across all characters

**Skeleton Requirements:**
- Standardized rig for all humanoid characters
- Root bone at origin (0, 0, 0)
- Y-up orientation
- Bones named using convention: `[Side]_[Bone]_[Index]`
  - Example: `L_Arm_Upper`, `R_Hand_01`

### 3.2 Character Bone Hierarchy

```
Root
├── Pelvis
│   ├── Spine_01
│   │   ├── Spine_02
│   │   │   ├── Spine_03
│   │   │   │   ├── Neck
│   │   │   │   │   └── Head
│   │   │   │   │       ├── Jaw (optional)
│   │   │   │   │       ├── L_Eye (optional)
│   │   │   │   │       └── R_Eye (optional)
│   │   │   │   ├── L_Clavicle
│   │   │   │   │   └── L_Arm_Upper
│   │   │   │   │       └── L_Arm_Lower
│   │   │   │   │           └── L_Hand
│   │   │   │   │               ├── L_Thumb_01-02
│   │   │   │   │               ├── L_Index_01-02
│   │   │   │   │               └── L_Fingers_01-02 (combined)
│   │   │   │   └── R_Clavicle (mirror of L_)
│   ├── L_Leg_Upper
│   │   └── L_Leg_Lower
│   │       └── L_Foot
│   │           └── L_Toe
│   └── R_Leg_Upper (mirror of L_)
├── Weapon_Socket_R
├── Weapon_Socket_L
├── Back_Socket
└── Head_Socket (for hats/helmets)
```

**Total Bones: 50-55 (LOD0), 30 (LOD2)**

### 3.3 Cosmetic Items

| Item Type | Triangle Budget | Texture Size | Notes |
|-----------|-----------------|--------------|-------|
| **Hat/Helmet** | 500-1,000 | 512x512 | Shared atlas when possible |
| **Full Outfit** | 2,000-3,000 | 1024x1024 | Replaces body mesh |
| **Accessory** | 200-500 | 256x256 | Belts, jewelry, etc. |
| **Back Item** | 500-1,500 | 512x512 | Capes, backpacks |
| **Weapon** | 500-1,000 | 512x512 | Melee weapons, tools |

**Cosmetic Rules:**
- Must use same skeleton as base character
- No additional bones (except cloth sim bones, max 8)
- Must pass clipping test with all animations
- Weight painting must match base character

### 3.4 NPC Monster Models

| Monster Type | Triangles | Bones | Texture | Materials |
|--------------|-----------|-------|---------|-----------|
| **Small (Skeleton)** | 2,000 | 25 | 512x512 | 1 |
| **Medium (Crab)** | 3,000 | 30 | 512x512 | 1 |
| **Large (Serpent)** | 5,000 | 40 | 1024x1024 | 2 |
| **Boss (Kraken)** | 8,000 | 50 | 1024x1024 | 2 |

---

## 4. Environment Specifications

### 4.1 Terrain

| Specification | Value | Notes |
|---------------|-------|-------|
| **Terrain Size** | 2km x 2km | Playable area |
| **Heightmap Resolution** | 1025x1025 | Unity terrain standard |
| **Base Texture Layers** | 4-6 | Splatmap based |
| **Detail Meshes** | Grass, rocks | GPU instanced |
| **Detail Density** | 0.5-1.0 per m² | Adjustable by quality |

### 4.2 Props - Polygon Budgets

| Prop Category | Triangle Budget | Texture Size | LOD Levels |
|---------------|-----------------|--------------|------------|
| **Scatter (shells, coins)** | 50-100 | 128x128 atlas | 1 (billboard) |
| **Small (barrels, crates)** | 200-500 | 256x256 | 2 |
| **Medium (carts, cannons)** | 500-1,500 | 512x512 | 3 |
| **Large (ships, towers)** | 2,000-5,000 | 1024x1024 | 3 |
| **Hero (treasure chest)** | 500-1,000 | 512x512 | 2 |

### 4.3 Buildings

| Building Type | Triangle Budget | Texture Size | LOD Levels |
|---------------|-----------------|--------------|------------|
| **Small (shack, stall)** | 2,000-3,000 | 512x512 | 3 |
| **Medium (house, tavern)** | 5,000-8,000 | 1024x1024 | 3 |
| **Large (warehouse, fort)** | 8,000-15,000 | 1024x1024 | 4 |
| **Landmark (lighthouse)** | 10,000-20,000 | 2048x2048 | 4 |

**Interior Requirements:**
- Separate interior mesh from exterior
- Interior only rendered when player inside
- Simpler geometry for interiors (50% of exterior)
- Pre-baked lighting for interiors

### 4.4 Vegetation

| Vegetation Type | Triangles | Texture | Rendering |
|-----------------|-----------|---------|-----------|
| **Grass Blade** | 4-8 | 128x128 atlas | GPU Instanced |
| **Flower** | 8-16 | 256x256 atlas | GPU Instanced |
| **Bush** | 100-300 | 512x512 | Standard |
| **Palm Tree** | 500-1,000 | 512x512 | Billboards at distance |
| **Large Tree** | 1,000-2,000 | 1024x1024 | Billboards at distance |

**Vegetation Rendering:**
- Use alpha-tested cutout shader
- Billboard LODs for trees beyond 50m
- Impostor atlas for forests
- Wind animation via shader (no bones)

### 4.5 Water

| Specification | Low Quality | Medium | High |
|---------------|-------------|--------|------|
| **Mesh Resolution** | 1 vert/4m | 1 vert/2m | 1 vert/1m |
| **Shader Complexity** | Simple color | Basic waves | Full SSR |
| **Foam Texture** | 256x256 | 512x512 | 512x512 |
| **Reflection** | None | Planar simple | SSR |
| **Refraction** | None | Depth-based | Full |

---

## 5. Texture Specifications

### 5.1 Texture Sizes by Asset Type

| Asset Type | Diffuse/Albedo | Normal Map | Mask/ORM | Notes |
|------------|----------------|------------|----------|-------|
| **Player Character** | 1024x1024 | 512x512 | 512x512 | Per-character |
| **Cosmetic Item** | 512x512 | 256x256 | - | Or atlas |
| **Monster (Large)** | 1024x1024 | 512x512 | 512x512 | - |
| **Monster (Small)** | 512x512 | 256x256 | - | - |
| **Hero Prop** | 1024x1024 | 512x512 | - | Treasure chest |
| **Standard Prop** | 512x512 | 256x256 | - | Or atlas |
| **Environment Atlas** | 2048x2048 | 1024x1024 | - | Shared props |
| **Building** | 1024x1024 | 512x512 | - | Tiling + trim |
| **Terrain Splat** | 512x512 each | 512x512 | - | 4-6 layers |
| **UI Element** | Power of 2 | - | - | Max 2048 |
| **VFX Sprite** | 256x256 max | - | - | Or atlas |

### 5.2 Texture Formats

| Platform | Diffuse/Albedo | Normal Map | UI/Alpha |
|----------|----------------|------------|----------|
| **iOS** | ASTC 4x4 | ASTC 4x4 | ASTC 4x4 |
| **Android (High)** | ASTC 4x4 | ASTC 4x4 | ASTC 4x4 |
| **Android (Low)** | ETC2 | ETC2 | ETC2 + Alpha |
| **Fallback** | RGBA32 | RGB24 | RGBA32 |

### 5.3 Texture Channels

**Standard Material (ORM Mask):**
| Channel | Content |
|---------|---------|
| R | Ambient Occlusion |
| G | Roughness |
| B | Metallic |
| A | Unused / Emission mask |

**Character Material:**
| Texture | Channels |
|---------|----------|
| Diffuse | RGB: Color, A: Opacity |
| Normal | RG: Normal XY (BC5 or ASTC) |
| Mask | R: AO, G: Roughness, B: Metallic |

### 5.4 Texture Atlas Strategy

**Environment Atlas (2048x2048):**
- Contains 16-64 props per atlas
- Organized by biome
- Trim sheets for architectural details

**Character Atlas:**
- Shared accessories per rarity tier
- Common items share 1024x1024 atlas

### 5.5 Mipmap Requirements

| Asset Type | Mipmaps | Streaming |
|------------|---------|-----------|
| **Characters** | Yes | Priority High |
| **Environment** | Yes | Priority Medium |
| **UI** | No | Always Loaded |
| **VFX** | Yes | Priority Low |

---

## 6. LOD System

### 6.1 LOD Distance Thresholds

| LOD Level | Distance Range | Use Case |
|-----------|----------------|----------|
| **LOD0** | 0-20m | Full detail, hero shots |
| **LOD1** | 20-50m | Medium distance gameplay |
| **LOD2** | 50-100m | Distant, minimal detail |
| **LOD3** | 100-150m | Billboard/Impostor |
| **Culled** | 150m+ | Not rendered |

### 6.2 LOD Reduction Ratios

| Transition | Triangle Reduction | Texture Reduction |
|------------|-------------------|-------------------|
| LOD0 → LOD1 | 50% | 50% |
| LOD1 → LOD2 | 50% | 50% |
| LOD2 → LOD3 | Billboard | 128x128 impostor |

### 6.3 LOD Transition Settings

| Setting | Value | Notes |
|---------|-------|-------|
| **Transition Type** | Cross-fade | Smooth blending |
| **Fade Duration** | 0.3 seconds | Prevents pop |
| **Screen Size Bias** | 1.0 | Adjustable per quality tier |
| **Shadow LOD Bias** | +1 level | Shadows use lower LOD |

### 6.4 LOD Requirements by Asset

| Asset | Required LODs | Notes |
|-------|---------------|-------|
| **Player Character** | 3 (LOD0-2) | Critical for 60-player scenes |
| **NPC Monster** | 2-3 | Based on size |
| **Small Prop** | 2 | Billboard for LOD2 |
| **Large Prop** | 3 | Full reduction chain |
| **Building** | 3-4 | Include roof-only LOD |
| **Vegetation** | 2 + Billboard | Required for forests |

### 6.5 Impostor/Billboard System

**For vegetation and distant props:**
- 8-direction impostor atlas
- 512x512 per object (64x64 per direction)
- Auto-generated from LOD0
- Used beyond 100m

---

## 7. Animation Specifications

### 7.1 Animation Clip Settings

| Setting | Value | Notes |
|---------|-------|-------|
| **Frame Rate** | 30 FPS | Standard for gameplay |
| **Compression** | Optimal | Unity keyframe reduction |
| **Root Motion** | Optional | Per-ability decision |
| **Import Scale** | 1.0 | Matches model scale |

### 7.2 Animation Budgets

| Animation Type | Max Length | Max Keyframes | File Size Target |
|----------------|------------|---------------|------------------|
| **Idle** | 2-4 sec | 120 | 50 KB |
| **Locomotion** | 1-2 sec | 60 | 30 KB |
| **Combat** | 0.5-1.5 sec | 45 | 25 KB |
| **Ability** | 0.5-2 sec | 60 | 40 KB |
| **Emote** | 2-10 sec | 300 | 100 KB |

### 7.3 Animation Bone Limits

| LOD | Max Bones | Active Bones | Notes |
|-----|-----------|--------------|-------|
| **LOD0** | 55 | 55 | Full skeleton |
| **LOD1** | 55 | 40 | Finger simplification |
| **LOD2** | 30 | 30 | Major joints only |

### 7.4 Blend Tree Requirements

**Locomotion Blend Tree:**
- 8-direction movement (forward, back, strafe x2, diagonals x4)
- Speed parameter for walk/run blending
- All clips same length (1 second normalized)

**Combat Blend Tree:**
- Upper/lower body separation
- Additive layers for hit reactions
- Ability animation overrides

### 7.5 Animation Memory Budget

| Platform Tier | Animation Memory | Notes |
|---------------|------------------|-------|
| **Low-End** | 50 MB | Essential anims only |
| **Mid-Range** | 75 MB | Standard set |
| **High-End** | 100 MB | Full set + variants |

---

## 8. VFX Specifications

### 8.1 Particle System Limits

| Quality Tier | Max Particles (Scene) | Max Emitters | Max Particle Size |
|--------------|----------------------|--------------|-------------------|
| **Low** | 500 | 10 | 2m |
| **Medium** | 1,000 | 20 | 3m |
| **High** | 2,000 | 40 | 4m |

### 8.2 Individual VFX Budgets

| VFX Type | Particles | Emitters | Draw Calls | Duration |
|----------|-----------|----------|------------|----------|
| **Projectile (Active)** | 20-50 | 2-3 | 2 | Continuous |
| **Impact (One-shot)** | 30-100 | 3-5 | 3 | 0.5-1s |
| **Ability (Complex)** | 50-150 | 5-8 | 5 | 1-3s |
| **Ambient (Loop)** | 10-30 | 1-2 | 1 | Continuous |
| **Status Effect** | 20-40 | 2-3 | 2 | Continuous |

### 8.3 VFX Texture Specifications

| Texture Type | Size | Format | Notes |
|--------------|------|--------|-------|
| **Particle Atlas** | 512x512 | ASTC 4x4 | 4x4 or 8x8 grid |
| **Flipbook** | 256x256 | ASTC 4x4 | 4x4 or 8x8 frames |
| **Noise/Gradient** | 128x128 | ASTC 6x6 | Tileable |
| **Distortion** | 256x256 | ASTC 4x4 | Normal map format |

### 8.4 VFX Shader Limits

| Shader Feature | Low | Medium | High |
|----------------|-----|--------|------|
| **Soft Particles** | No | Yes | Yes |
| **Distortion** | No | Simple | Full |
| **Lighting** | Unlit | Unlit | Basic lit |
| **Custom Data** | 1 channel | 2 channels | 4 channels |

### 8.5 VFX Performance Guidelines

**DO:**
- Use GPU instancing for repeated particles
- Pool and reuse particle systems
- Use simple blend modes (Alpha, Additive)
- Batch similar materials

**DON'T:**
- Use expensive shaders (refraction, complex math)
- Exceed particle limits per system
- Use physics simulation on particles
- Create new particle systems at runtime

---

## 9. UI Art Specifications

### 9.1 UI Resolution Standards

| Target Resolution | Scale Factor | Base Design |
|-------------------|--------------|-------------|
| **720p** | 0.75x | 960x540 |
| **1080p** | 1.0x | 1920x1080 |
| **1440p** | 1.33x | 2560x1440 |

**Design at 1x (1080p) and scale up/down.**

### 9.2 UI Element Sizes

| Element | Minimum Size | Recommended | Touch Target |
|---------|--------------|-------------|--------------|
| **Button** | 32x32 | 48x48 | 44x44 |
| **Icon** | 24x24 | 32x32 | - |
| **Text (Body)** | 12pt | 14pt | - |
| **Text (Header)** | 18pt | 24pt | - |
| **Ability Button** | 64x64 | 80x80 | 88x88 |

### 9.3 UI Texture Specifications

| Asset Type | Size | Format | Notes |
|------------|------|--------|-------|
| **Button States** | 256x64 | ASTC 4x4 | 9-slice |
| **Icon Atlas** | 1024x1024 | ASTC 4x4 | 64x64 per icon |
| **Background** | 2048x1024 | ASTC 6x6 | Can be compressed more |
| **Frame/Border** | 256x256 | ASTC 4x4 | 9-slice |
| **Progress Bar** | 512x64 | ASTC 4x4 | Sliced |

### 9.4 UI Atlas Strategy

| Atlas | Contents | Size |
|-------|----------|------|
| **Common UI** | Buttons, frames, basic icons | 2048x2048 |
| **HUD** | In-game elements | 1024x1024 |
| **Ability Icons** | All ability icons | 1024x1024 |
| **Currency/Items** | Economy icons | 512x512 |

### 9.5 UI Animation Guidelines

| Animation | Duration | Easing |
|-----------|----------|--------|
| **Button Press** | 0.1s | Ease Out |
| **Panel Open** | 0.2s | Ease Out Back |
| **Panel Close** | 0.15s | Ease In |
| **Notification** | 0.3s | Ease Out |
| **Number Pop** | 0.4s | Bounce |

---

## 10. Audio Specifications

### 10.1 Audio Formats

| Platform | Music | SFX | Voice |
|----------|-------|-----|-------|
| **iOS** | AAC 128kbps | AAC 96kbps | AAC 64kbps |
| **Android** | OGG 128kbps | OGG 96kbps | OGG 64kbps |

### 10.2 Audio File Specifications

| Audio Type | Sample Rate | Bit Depth | Channels | Max Duration |
|------------|-------------|-----------|----------|--------------|
| **Music** | 44.1 kHz | 16-bit | Stereo | 3-5 min loop |
| **SFX** | 44.1 kHz | 16-bit | Mono | 0.1-5 sec |
| **Ambient** | 44.1 kHz | 16-bit | Stereo | 30-60 sec loop |
| **Voice** | 22.05 kHz | 16-bit | Mono | 0.5-10 sec |
| **UI** | 22.05 kHz | 16-bit | Mono | 0.1-1 sec |

### 10.3 Audio Memory Budget

| Platform Tier | Total Audio | Music | SFX | Voice |
|---------------|-------------|-------|-----|-------|
| **Low-End** | 50 MB | 20 MB | 25 MB | 5 MB |
| **Mid-Range** | 75 MB | 30 MB | 35 MB | 10 MB |
| **High-End** | 100 MB | 40 MB | 45 MB | 15 MB |

### 10.4 Simultaneous Audio Limits

| Category | Low | Medium | High |
|----------|-----|--------|------|
| **Music Tracks** | 1 | 1 | 2 |
| **SFX Voices** | 16 | 24 | 32 |
| **Ambient Loops** | 2 | 4 | 6 |
| **3D Sound Sources** | 8 | 16 | 24 |

---

## 11. Quality Tiers

### 11.1 Graphics Quality Presets

| Feature | Low | Medium | High | Ultra |
|---------|-----|--------|------|-------|
| **Resolution Scale** | 0.75 | 1.0 | 1.0 | 1.0+ |
| **Texture Quality** | 50% | 75% | 100% | 100% |
| **Shadow Quality** | Off | Low | Medium | High |
| **Shadow Distance** | 0m | 30m | 50m | 80m |
| **Post-Processing** | Off | Basic | Standard | Full |
| **Anti-Aliasing** | Off | FXAA | SMAA | TAA |
| **VFX Quality** | 50% | 75% | 100% | 125% |
| **LOD Bias** | +2 | +1 | 0 | -1 |
| **Vegetation Density** | 30% | 60% | 100% | 100% |
| **Water Quality** | Simple | Basic | Full | Full |
| **Ambient Occlusion** | Off | Off | SSAO | SSAO |
| **Bloom** | Off | On | On | On |

### 11.2 Automatic Quality Selection

| Device RAM | Default Quality |
|------------|-----------------|
| < 3 GB | Low |
| 3-4 GB | Medium |
| 4-6 GB | High |
| > 6 GB | Ultra |

### 11.3 Quality-Specific Asset Loading

| Quality | Texture Mips | LOD Offset | Audio Quality |
|---------|--------------|------------|---------------|
| **Low** | Skip 2 | +2 | Low bitrate |
| **Medium** | Skip 1 | +1 | Standard |
| **High** | Full | 0 | Full |
| **Ultra** | Full | -1 | Full |

---

## 12. Performance Budgets

### 12.1 Frame Budget (30 FPS = 33ms)

| System | Low-End | Mid-Range | High-End |
|--------|---------|-----------|----------|
| **CPU Gameplay** | 8ms | 6ms | 4ms |
| **CPU Animation** | 4ms | 3ms | 2ms |
| **CPU Physics** | 2ms | 2ms | 2ms |
| **CPU Audio** | 2ms | 2ms | 2ms |
| **CPU Render Prep** | 4ms | 3ms | 2ms |
| **GPU Render** | 12ms | 10ms | 6ms |
| **Buffer** | 1ms | 7ms | 14ms |
| **Total** | 33ms | 33ms | 32ms |

### 12.2 Draw Call Budget

| Category | Low | Medium | High |
|----------|-----|--------|------|
| **Characters (60)** | 60 | 80 | 120 |
| **Environment** | 60 | 120 | 200 |
| **VFX** | 20 | 40 | 60 |
| **UI** | 10 | 10 | 20 |
| **Total** | 150 | 250 | 400 |

### 12.3 Memory Budget

| Category | Low (1GB) | Medium (1.5GB) | High (2GB) |
|----------|-----------|----------------|------------|
| **Textures** | 256 MB | 384 MB | 512 MB |
| **Meshes** | 128 MB | 192 MB | 256 MB |
| **Animations** | 50 MB | 75 MB | 100 MB |
| **Audio** | 50 MB | 75 MB | 100 MB |
| **VFX** | 32 MB | 48 MB | 64 MB |
| **UI** | 64 MB | 96 MB | 128 MB |
| **Scripts/Data** | 128 MB | 192 MB | 256 MB |
| **System/Buffer** | 292 MB | 438 MB | 584 MB |
| **Total** | 1000 MB | 1500 MB | 2000 MB |

### 12.4 Batching Targets

| Batch Type | Target | Notes |
|------------|--------|-------|
| **Static Batching** | 80% of static props | Pre-combined |
| **Dynamic Batching** | Small objects < 300 verts | Automatic |
| **GPU Instancing** | All repeated objects | Vegetation, crowds |
| **SRP Batcher** | All standard materials | Shader variants |

---

## 13. Asset Pipeline

### 13.1 Export Settings

**3D Models (FBX):**
```
- Scale: 1 unit = 1 meter
- Up Axis: Y
- Forward Axis: -Z
- Triangulate: No (preserve quads)
- Embed Textures: No
- Animation: Separate FBX per clip
```

**Textures:**
```
- Format: PNG (lossless source)
- Color Space: sRGB for diffuse, Linear for data
- Resolution: Power of 2
- No embedded ICC profiles
```

**Audio:**
```
- Format: WAV 44.1kHz 16-bit (source)
- Normalized to -3dB peak
- No silence padding
- Clearly named
```

### 13.2 Naming Conventions

**Pattern:** `[Category]_[Name]_[Variant]_[LOD].[ext]`

| Category | Prefix | Example |
|----------|--------|---------|
| Character | CHAR | `CHAR_Pirate_Default_LOD0.fbx` |
| Prop | PROP | `PROP_Barrel_Explosive.fbx` |
| Building | BLD | `BLD_Tavern_Large_LOD1.fbx` |
| Vegetation | VEG | `VEG_PalmTree_01.fbx` |
| VFX | VFX | `VFX_Fireball_Impact.prefab` |
| UI | UI | `UI_Button_Primary.png` |
| Texture | T | `T_Pirate_D.png` (Diffuse) |
| Animation | ANIM | `ANIM_Pirate_Run.fbx` |

**Texture Suffixes:**
- `_D` = Diffuse/Albedo
- `_N` = Normal
- `_M` = Mask (ORM)
- `_E` = Emission

### 13.3 Folder Structure

```
Assets/
├── Art/
│   ├── Characters/
│   │   ├── Player/
│   │   │   ├── Models/
│   │   │   ├── Textures/
│   │   │   └── Materials/
│   │   └── Monsters/
│   ├── Environment/
│   │   ├── Biomes/
│   │   │   ├── Beach/
│   │   │   ├── Jungle/
│   │   │   └── Volcanic/
│   │   ├── Props/
│   │   └── Buildings/
│   ├── VFX/
│   │   ├── Abilities/
│   │   ├── Combat/
│   │   └── Environment/
│   └── UI/
│       ├── HUD/
│       ├── Menus/
│       └── Icons/
├── Animations/
│   ├── Characters/
│   └── Monsters/
└── Audio/
    ├── Music/
    ├── SFX/
    └── Voice/
```

### 13.4 Version Control Rules

- Binary files (FBX, textures) in Git LFS
- Max file size: 100 MB
- No duplicate assets
- Delete unused assets promptly
- Source files stored separately (not in repo)

---

## 14. Validation Checklist

### 14.1 Model Checklist

- [ ] Triangle count within budget
- [ ] Proper UV unwrap (no overlaps except mirrored)
- [ ] Clean topology (no N-gons, proper edge flow)
- [ ] Correct scale (1 unit = 1 meter)
- [ ] Origin at logical point (center bottom)
- [ ] Named according to convention
- [ ] All LODs created
- [ ] LOD transitions tested
- [ ] Collision mesh created (if needed)
- [ ] No flipped normals

### 14.2 Texture Checklist

- [ ] Correct resolution (power of 2)
- [ ] Proper format for platform
- [ ] Mipmaps enabled (except UI)
- [ ] Correct color space
- [ ] No wasted texture space
- [ ] Proper channel packing
- [ ] Named according to convention

### 14.3 Animation Checklist

- [ ] Correct FPS (30)
- [ ] No foot sliding
- [ ] Smooth transitions
- [ ] Proper root motion (if used)
- [ ] Keyframe compression applied
- [ ] Named according to convention
- [ ] Works with all LODs

### 14.4 VFX Checklist

- [ ] Particle count within budget
- [ ] Pooled/reusable
- [ ] Proper sorting/blending
- [ ] No texture streaming issues
- [ ] Performance tested in worst case
- [ ] Scales properly with quality settings

### 14.5 Performance Checklist

- [ ] Tested on low-end device
- [ ] Maintains target framerate
- [ ] No memory spikes
- [ ] No loading hitches
- [ ] Draw calls within budget
- [ ] LODs popping verified acceptable

---

## 15. Appendix

### Appendix A: Device Testing Matrix

| Device | OS | RAM | Target FPS | Quality |
|--------|----|----|------------|---------|
| iPhone 8 | iOS 15 | 2 GB | 30 | Low |
| iPhone 11 | iOS 16 | 4 GB | 60 | High |
| iPhone 14 | iOS 17 | 6 GB | 60 | Ultra |
| Samsung A21 | Android 11 | 3 GB | 30 | Low |
| Samsung S20 FE | Android 12 | 6 GB | 60 | High |
| Samsung S23 | Android 13 | 8 GB | 60 | Ultra |
| Pixel 6a | Android 13 | 6 GB | 60 | High |
| Xiaomi 11T | Android 12 | 8 GB | 60 | Ultra |

### Appendix B: Shader Complexity Limits

| Shader Type | ALU Ops | Texture Samples | Interpolators |
|-------------|---------|-----------------|---------------|
| **Character** | 30 | 3 | 4 |
| **Environment** | 20 | 4 | 3 |
| **VFX** | 15 | 2 | 2 |
| **UI** | 10 | 2 | 2 |
| **Water** | 40 | 4 | 4 |

### Appendix C: Compression Comparison

| Format | Quality | Size (1024x1024) | Decode Speed |
|--------|---------|------------------|--------------|
| **ASTC 4x4** | High | 1 MB | Fast |
| **ASTC 6x6** | Medium | 0.5 MB | Fast |
| **ASTC 8x8** | Low | 0.25 MB | Fast |
| **ETC2** | Medium | 1 MB | Medium |
| **PVRTC** | Medium | 0.5 MB | Fast |

### Appendix D: Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Texture pop-in | Slow streaming | Increase priority, reduce resolution |
| FPS drops | Too many draw calls | Batch more, reduce unique materials |
| Memory spike | Large asset load | Async loading, streaming |
| LOD pop | Wrong distances | Increase crossfade, adjust distances |
| Animation jitter | Low bone count at distance | Increase LOD distance thresholds |
| VFX lag | Too many particles | Reduce count, use GPU instancing |

### Appendix E: Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | Technical Art Team | Initial specifications |

---

## Document Approval

| Role | Name | Signature | Date | Status |
|------|------|-----------|------|--------|
| Technical Art Director | | | 2025-11-30 | Approved |
| Lead Programmer | | | 2025-11-30 | Approved |
| Art Director | | | 2025-11-30 | Approved |
| Producer | | | 2025-11-30 | Approved |

---

**End of Technical Art Specifications**
