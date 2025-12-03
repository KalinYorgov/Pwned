# UX-008: Main HUD Layout Specification

## Overview

This document specifies the in-game Heads-Up Display (HUD) layout for Plunderstorm Mobile, organizing all essential gameplay information without cluttering the screen or obscuring the action.

## Design Philosophy

The HUD must provide at-a-glance access to critical information while maintaining visual clarity during intense combat. Elements are organized by priority and frequency of use, with the most important information always visible and secondary information accessible on demand.

---

## 1. HUD Element Hierarchy

### 1.1 Priority Levels
| Priority | Elements | Visibility |
|----------|----------|------------|
| Critical | Health, Abilities, Storm Timer | Always visible |
| Important | Minimap, Player Count, Level/XP | Always visible |
| Secondary | Kill Feed, Cooldowns | Visible, can minimize |
| Contextual | Interact Prompts, Notifications | Shown when relevant |
| Optional | FPS Counter, Network Stats | Toggle in settings |

### 1.2 Information Density
```
Target: Show all critical info at a glance
Rule: No more than 7±2 distinct elements visible
Principle: Clarity over completeness
Mobile: Optimize for small screen readability
```

---

## 2. Screen Layout

### 2.1 Zone Map
```
┌────────────────────────────────────────────────────────────┐
│ [Top-Left Zone]       [Top-Center Zone]     [Top-Right Zone]│
│   - Minimap             - Player Count          - Kill Feed │
│   - Compass             - Storm Timer           - Event Log │
│                                                             │
│                                                             │
│                                                             │
│                    [Central Play Area]                      │
│                    (Keep clear for gameplay)                │
│                                                             │
│                                                             │
│                                                             │
│ [Bottom-Left Zone]  [Bottom-Center Zone]  [Bottom-Right Zone]│
│   - Movement          - Health Bar           - Ability Buttons│
│   - Sprint             - XP/Level            - Basic Attack │
│                        - Status Effects                     │
└────────────────────────────────────────────────────────────┘
```

### 2.2 Zone Specifications
| Zone | Width | Height | Purpose |
|------|-------|--------|---------|
| Top-Left | 25% | 20% | Navigation info |
| Top-Center | 50% | 10% | Match status |
| Top-Right | 25% | 20% | Event feed |
| Central | 100% | 60% | Gameplay (keep clear) |
| Bottom-Left | 40% | 30% | Movement controls |
| Bottom-Center | 20% | 20% | Player status |
| Bottom-Right | 40% | 30% | Combat controls |

---

## 3. Health Display

### 3.1 Health Bar Design
```
Position: Bottom center of screen
Size: 250pt width × 24pt height (default)

Structure:
┌──────────────────────────────────────────────┐
│ ❤️ [█████████████████████░░░░░] 175/200  L8  │
└──────────────────────────────────────────────┘
  │    └── Fill bar ────────────┘   │     │
  │                                 │     └── Level indicator
  └── Health icon                   └── Current/Max HP
```

### 3.2 Health Bar States
| State | Visual | Animation |
|-------|--------|-----------|
| Full | Green fill (#44FF44) | None |
| High (>75%) | Green fill | None |
| Medium (50-75%) | Yellow fill (#FFFF44) | None |
| Low (25-50%) | Orange fill (#FF8844) | Slow pulse |
| Critical (<25%) | Red fill (#FF4444) | Fast pulse, vignette |
| Healing | Current + green preview | Shimmer effect |
| Damage | Red flash, shake | Quick reduction |

### 3.3 Shield/Armor Display
```
If Shield Active:
┌──────────────────────────────────────────────┐
│ ❤️ [████████████████░░░░░░░░░░] 150/200      │
│ 🛡️ [█████████░░░░░░░░░░░░░░░░░]  50/100      │
└──────────────────────────────────────────────┘
Shield bar appears below health when active
```

---

## 4. XP and Level Display

### 4.1 XP Bar
```
Position: Below health bar or integrated
Size: 200pt width × 8pt height

Structure:
Level 8 [████████████░░░░░░] 720/1000 XP
        └── Progress to next level ──┘
```

### 4.2 Level-Up Notification
```
On Level Up:
├── Flash effect on health bar
├── Level number animates (scale up)
├── "+1 LEVEL" text briefly appears
├── Sound effect plays
└── Health fully restored
```

---

## 5. Minimap

### 5.1 Minimap Design
```
Position: Top-left corner
Size: 100pt × 100pt (default), scalable

┌───────────────────┐
│ N                 │ ← North indicator
│    ·  ·          │
│  ·    ▲    ·     │ ← Player arrow (center)
│    ·      ·      │ ← Points of interest
│         ●        │ ← Teammate (in team modes)
│ ╭───────╮        │
│ │ STORM │        │ ← Storm boundary
│ ╰───────╯        │
└───────────────────┘
  Tap to expand
```

### 5.2 Minimap Elements
| Element | Symbol | Color |
|---------|--------|-------|
| Player | ▲ (arrow) | White |
| Teammate | ● (dot) | Blue |
| Enemy (recent) | ● (dot) | Red (fades) |
| Storm Zone | Filled area | Red/purple |
| Safe Zone | Circle outline | White |
| Next Zone | Dashed circle | Yellow |
| Chest (optional) | □ | Gold |
| Ping | ◆ | Ping color |

### 5.3 Minimap Interactions
| Action | Result |
|--------|--------|
| Tap | Expand to full map view |
| Pinch | Zoom in/out |
| Hold + tap | Place ping |
| Double-tap | Reset zoom |

---

## 6. Player Count and Match Info

### 6.1 Player Count Display
```
Position: Top center
Size: Minimal footprint

Format: ⚔️ 47/60
        └── Alive/Total players

Or with placement: #12 of 60
```

### 6.2 Storm Timer
```
Position: Top center, below player count

Phases:
├── Waiting: "Storm: 2:30" (white)
├── Moving: "Storm moving: 0:45" (yellow)
├── Closing: "STORM CLOSING" (red, animated)
└── Damage: Storm icon + DPS number
```

### 6.3 Match Timer
```
Position: Near storm timer
Format: 08:42 (MM:SS)
Visibility: Optional in settings
```

---

## 7. Kill Feed

### 7.1 Design
```
Position: Top-right corner
Direction: New entries at top, push down
Max visible: 4 entries
Entry duration: 5 seconds, then fade

Entry Format:
┌────────────────────────────────────┐
│ PlayerName ⚔️ VictimName           │
│ [Ability Icon]                     │
└────────────────────────────────────┘

Your kills highlighted:
┌────────────────────────────────────┐
│ 🎯 YOU eliminated VictimName       │
│ [Cutlass Slash]          +100 XP   │
└────────────────────────────────────┘
```

### 7.2 Kill Feed Information
| Event | Display |
|-------|---------|
| Kill | Killer ⚔️ Victim |
| Your Kill | 🎯 YOU eliminated Victim (+XP) |
| Your Death | 💀 Killer eliminated YOU |
| Assist | Assisted in eliminating Victim |
| Storm Kill | 🌀 Victim died to Storm |

---

## 8. Ability and Combat HUD

### 8.1 Ability Slots (Reference: UX-003)
```
Position: Bottom-right
Layout: Diamond pattern or customized

See: ABILITY_BUTTON_LAYOUT_SPEC.md
```

### 8.2 Cooldown Timers
```
Display: On ability buttons
Format: Numeric seconds + sweep overlay
Ready: Button fully visible, subtle glow
```

### 8.3 Ammo/Charges (If Applicable)
```
For abilities with charges:
Position: Below/beside ability button
Format: ●●○ (filled = available)
```

---

## 9. Status Effects Display

### 9.1 Active Effects
```
Position: Above or beside health bar
Size: 24pt × 24pt per icon

Layout:
┌─────────────────────────────────┐
│ [🔥 3s] [🐌 5s] [💚 2s]         │
│ Health Bar below...             │
└─────────────────────────────────┘

Icon shows effect type
Timer shows remaining duration
Stack count if applicable
```

### 9.2 Effect Types
| Effect | Icon | Color | Description |
|--------|------|-------|-------------|
| Burning | 🔥 | Orange | Damage over time |
| Slowed | 🐌 | Blue | Movement reduced |
| Healing | 💚 | Green | Regenerating HP |
| Shielded | 🛡️ | Blue | Absorbing damage |
| Haste | 💨 | Yellow | Speed increased |
| Stunned | ⭐ | Yellow | Cannot act |

---

## 10. Contextual Elements

### 10.1 Interact Prompts
```
When Near Interactive Object:

┌─────────────────────────────────┐
│      [TAP] Open Chest           │
│         ⬇️                       │
│      💎 [Chest Image]           │
└─────────────────────────────────┘

Position: Center-bottom of screen
Appears when within interact range
Touch anywhere on prompt or object
```

### 10.2 Notification Banner
```
Position: Top-center, below match info
Duration: 3-5 seconds

Types:
├── Zone Warning: "Storm approaching! 30 seconds"
├── Achievement: "First Blood! +50 XP"
├── Item Acquired: "Fireball Rank 2"
├── Team Message: "Teammate eliminated"
└── System: "Player reconnecting..."
```

### 10.3 Danger Indicators
```
Storm Warning:
├── Screen edge tint (purple/red)
├── Direction arrow if storm nearby
├── "GET TO SAFE ZONE" when in storm
└── Increasing intensity as damage stacks

Damage Direction:
├── Red arc on screen edge
├── Points toward damage source
├── Intensity = damage amount
└── Fades over 1-2 seconds
```

---

## 11. HUD Customization

### 11.1 Scale Options
| Setting | Range | Default |
|---------|-------|---------|
| HUD Scale | 75% - 150% | 100% |
| Health Bar Size | Small/Medium/Large | Medium |
| Minimap Size | Small/Medium/Large | Medium |
| Text Size | Small/Medium/Large | Medium |

### 11.2 Toggle Options
| Element | Default | Notes |
|---------|---------|-------|
| Kill Feed | On | Can minimize to icon |
| Damage Numbers | On | Floating combat text |
| XP Bar | On | Hide when level 10 |
| FPS Counter | Off | Developer/debug |
| Ping Display | Off | Network latency |
| Compass | On | North indicator |

### 11.3 Transparency Options
```
Settings:
├── HUD Opacity: 50% - 100%
├── Inactive Opacity: 30% - 100%
├── Combat Fade: On/Off (HUD fades when not in combat)
└── Auto-Hide: Never/5s/10s/30s
```

---

## 12. Aspect Ratio Handling

### 12.1 Standard Ratios
| Ratio | Adaptation |
|-------|------------|
| 16:9 | Standard layout |
| 18:9 | Extra horizontal space, spread elements |
| 19.5:9 | Notch accommodation, safe areas |
| 21:9 | Ultra-wide, consider pillarbox |
| 4:3 (tablet) | More vertical space, adjust positions |

### 12.2 Safe Area Compliance
```
All HUD Elements Must:
├── Respect device safe areas
├── Stay clear of notches and cutouts
├── Avoid home indicator zone
├── Not overlap rounded corners
└── Auto-adjust on orientation change

See: UX-024 (Notch and Safe Area Handling)
```

---

## 13. Performance Optimization

### 13.1 Rendering
```
Optimization Techniques:
├── Single atlas for all HUD icons
├── Text rendered to texture (update on change)
├── Batched UI draw calls
├── Skip rendering for hidden elements
├── Dirty flag system for updates
└── Separate canvas for frequently updated elements
```

### 13.2 Update Frequencies
| Element | Update Frequency |
|---------|-----------------|
| Health Bar | On change + every 100ms |
| Cooldowns | Every frame |
| Minimap | Every 100ms |
| Kill Feed | On event |
| Timers | Every second |
| XP Bar | On change |

### 13.3 Memory Budget
| Component | Budget |
|-----------|--------|
| HUD textures | ~2 MB |
| Icon atlas | ~1 MB |
| Fonts | ~512 KB |
| Runtime state | ~64 KB |
| Total | ~4 MB |

---

## 14. Accessibility

### 14.1 Visual Accessibility
```
Features:
├── High contrast mode
├── Larger text options
├── Icon + text labels (not icon only)
├── Colorblind-safe palette
├── Screen reader compatibility hints
└── Adjustable opacity/brightness
```

### 14.2 Colorblind Modes
| Element | Default | Deuteranopia | Protanopia | Tritanopia |
|---------|---------|--------------|------------|------------|
| Health Low | Red | Orange | Orange | Red |
| Health High | Green | Blue | Blue | Green |
| Teammate | Blue | Blue | Cyan | Blue |
| Enemy | Red | Orange | Yellow | Red |
| Storm | Purple | Purple | Purple | Pink |

### 14.3 Motion Reduction
```
When Enabled:
├── Disable pulse animations
├── Static health bar (no shake)
├── Reduce notification movement
├── No screen shake effects
└── Simplified transitions
```

---

## 15. Localization

### 15.1 Text Considerations
```
Requirements:
├── All text strings externalized
├── Dynamic text wrapping
├── Support for RTL languages
├── Number formatting per locale
├── Variable text length accommodation
└── Icon + text for critical elements
```

### 15.2 Text Length Handling
| Element | Max Characters | Overflow |
|---------|---------------|----------|
| Player Name | 16 | Truncate + "..." |
| Kill Feed | 40 | Two lines if needed |
| Notifications | 60 | Word wrap |
| Timer | 5 | Fixed format |
| Health | 7 | "175/200" format |

---

## 16. Testing Requirements

### 16.1 Visual Tests
```
Verify:
├── All elements visible on all screen sizes
├── No overlap between elements
├── Text readable at minimum size
├── Colors distinguishable in colorblind modes
├── Safe areas respected
├── Orientation changes handled
└── Notch/cutout avoidance
```

### 16.2 Functional Tests
```
Verify:
├── Health updates correctly
├── Cooldowns accurate
├── Kill feed shows events
├── Minimap reflects world state
├── Timers count correctly
├── Notifications appear/dismiss
└── All toggles work
```

### 16.3 Performance Tests
```
Verify:
├── 60 FPS maintained with full HUD
├── No memory leaks
├── Minimal CPU for HUD updates
├── Texture memory within budget
└── Battery impact acceptable
```

---

## 17. Default Configuration

```json
{
  "hud": {
    "scale": 1.0,
    "opacity": 0.9,
    "inactiveOpacity": 0.7,
    "autoHide": false
  },
  "healthBar": {
    "size": "medium",
    "position": "bottom-center",
    "showNumeric": true,
    "showLevel": true
  },
  "minimap": {
    "size": "medium",
    "position": "top-left",
    "rotation": "fixed",
    "zoom": 1.0
  },
  "killFeed": {
    "enabled": true,
    "maxEntries": 4,
    "duration": 5.0,
    "position": "top-right"
  },
  "stormInfo": {
    "showTimer": true,
    "showWarnings": true,
    "position": "top-center"
  },
  "xpBar": {
    "enabled": true,
    "showNumeric": false,
    "position": "below-health"
  },
  "statusEffects": {
    "position": "above-health",
    "showDuration": true,
    "maxVisible": 5
  },
  "optional": {
    "fpsCounter": false,
    "pingDisplay": false,
    "compass": true,
    "damageNumbers": true
  }
}
```

---

## Appendix A: Visual Reference

```
Complete HUD Layout:

┌────────────────────────────────────────────────────────────────┐
│ ┌───────────┐                    ⚔️ 47           ┌───────────┐ │
│ │  MINIMAP  │               Storm: 2:30          │ Kill Feed │ │
│ │     ▲     │                                    │ A ⚔️ B    │ │
│ │   N ┼ S   │                                    │ C ⚔️ D    │ │
│ └───────────┘                                    └───────────┘ │
│                                                                │
│                                                                │
│                     [ GAMEPLAY AREA ]                          │
│                       Keep Clear                               │
│                                                                │
│                                                                │
│                                                                │
│                                                                │
│   ┌─────┐         [🔥][🛡️]                        ┌────────┐  │
│   │     │    ❤️ [████████████░░] 175/200 L8        │  [U1]  │  │
│   │ JOY │      [████████░░░░░░░] 720/1000 XP     [O1][BA][O2]│  │
│   │     │                                          │  [U2]  │  │
│   └─────┘                                        └────────┘  │
└────────────────────────────────────────────────────────────────┘
```

---

## Appendix B: Element Specifications

| Element | Size | Position | Priority |
|---------|------|----------|----------|
| Health Bar | 250×24pt | Bottom-Center | Critical |
| Minimap | 100×100pt | Top-Left | Important |
| Kill Feed | 200×100pt | Top-Right | Secondary |
| Player Count | 60×24pt | Top-Center | Important |
| Storm Timer | 100×24pt | Top-Center | Critical |
| XP Bar | 200×8pt | Below Health | Important |
| Status Effects | 120×24pt | Above Health | Important |
| Ability Buttons | 56pt each | Bottom-Right | Critical |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
