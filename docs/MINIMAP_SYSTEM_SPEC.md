# UX-009: Minimap System Specification

## Overview

This document specifies the minimap system for Plunderstorm Mobile, providing players with tactical awareness of their position, the storm, teammates, and points of interest on the game map.

## Design Philosophy

The minimap serves as a critical navigation and tactical tool. It must provide quick spatial awareness without requiring sustained attention. The design balances information density with readability, using a pirate-themed treasure map aesthetic while maintaining functional clarity.

---

## 1. Minimap Configuration

### 1.1 Default Settings
| Parameter | Value | Range |
|-----------|-------|-------|
| Size | 100pt × 100pt | 60pt - 150pt |
| Position | Top-left corner | Any corner |
| Opacity | 80% | 40% - 100% |
| Zoom Level | 1.0x | 0.5x - 2.0x |
| Rotation Mode | Fixed (north up) | Fixed/Rotating |

### 1.2 Size Presets
| Preset | Dimensions | Use Case |
|--------|------------|----------|
| Small | 60pt × 60pt | Minimal distraction |
| Medium | 100pt × 100pt | Default balance |
| Large | 150pt × 150pt | Navigation focus |
| Custom | 60-150pt | User preference |

---

## 2. Visual Design

### 2.1 Map Aesthetic
```
Theme: Aged treasure map / nautical chart
├── Parchment background texture
├── Compass rose decoration (optional)
├── Weathered border frame
├── Ink-style terrain features
└── Gold accents for important markers

Note: Style should not compromise readability
Option: "Simple" mode for pure functionality
```

### 2.2 Map Frame
```
┌──────────────────────┐
│ N                    │ ← North indicator
│ ┌──────────────────┐ │
│ │                  │ │
│ │   MAP CONTENT    │ │
│ │                  │ │
│ └──────────────────┘ │
│        ⚙️ ⊕ ⊖       │ ← Controls (optional)
└──────────────────────┘
  └── Tap to expand

Border: 4pt pirate rope/chain motif
Corner: Decorative anchors (toggleable)
```

### 2.3 Terrain Representation
| Feature | Representation | Color |
|---------|---------------|-------|
| Land | Solid fill | Tan/beige (#E8D5B7) |
| Water | Solid fill | Blue (#4A90D9) |
| Buildings | Outlined rectangles | Brown (#8B4513) |
| Trees/Forest | Dot clusters | Dark green (#2E8B57) |
| Roads/Paths | Lines | Light brown (#D2B48C) |
| Cliffs/Edges | Thick outline | Dark gray (#333333) |
| Named Locations | Text label | Black |

---

## 3. Player Representation

### 3.1 Player Arrow
```
Shape: Triangle/arrow pointing facing direction
Size: 8pt at default zoom
Color: White with dark outline

         ▲
        ╱ ╲
       ╱   ╲
      ╱     ╲
     ╱───────╲

Behavior:
├── Always centered (in standard mode)
├── Rotates to show facing direction
├── Pulses subtly for visibility
└── Scales with zoom level
```

### 3.2 Player Field of View (Optional)
```
When Enabled:
├── Cone extending from player arrow
├── Shows camera view direction
├── Semi-transparent fill
├── 60° angle default (matches FOV)
└── Helps with spatial orientation
```

---

## 4. Team Information

### 4.1 Teammate Markers
```
Solo Mode: N/A
Duo/Team Mode:

Teammate Dot:
├── Shape: Filled circle
├── Size: 6pt
├── Color: Blue (#4488FF)
├── Border: White 1pt outline
└── Shows facing direction (small line)

Eliminated Teammate:
├── Shape: X mark
├── Color: Gray
├── Shows last known position
└── Fades after 30 seconds
```

### 4.2 Team Colors (Team Mode)
| Team Position | Color | Name |
|--------------|-------|------|
| Player | White | Self |
| Teammate 1 | Blue | Partner |
| Teammate 2 | Green | Ally |
| Teammate 3 | Yellow | Squad |

---

## 5. Storm Visualization

### 5.1 Current Storm
```
Representation:
├── Filled area outside safe zone
├── Color: Red/purple gradient (#8B0000 to #4B0082)
├── Opacity: 50%
├── Edge: Clear boundary line
└── Animation: Subtle swirl effect

Safe Zone Interior:
├── Unfilled (shows map terrain)
├── Bright compared to storm
└── Clear visual distinction
```

### 5.2 Safe Zone Circle
```
Current Safe Zone:
├── White/light circle outline
├── Thickness: 2pt
├── Solid line style
└── Center dot for reference

Next Safe Zone:
├── Yellow/gold dashed circle
├── Thickness: 2pt
├── Animated dashes (moving)
└── Shows where zone will shrink to

Transition:
├── During shrink: Both circles visible
├── After shrink: Current becomes next
└── Animation: Smooth interpolation
```

### 5.3 Storm Phase Indicators
```
On Minimap Edge or Nearby:
├── Phase number (1-5)
├── Time until next phase
├── Distance to safe zone (if outside)
└── Direction arrow to safe zone
```

---

## 6. Points of Interest

### 6.1 Named Locations
```
Display:
├── Text label over location
├── Font: 8pt bold
├── Color: Dark with light outline
├── Only shows nearby locations at default zoom
└── All visible in expanded view
```

### 6.2 Chest/Loot Markers (Optional)
```
When Enabled:
├── Small gold chest icon
├── Only shows unopened chests
├── Fades when far away
├── Configurable: On/Off/Nearby Only

Note: Optional feature to prevent overwhelming
```

### 6.3 Combat Indicators
```
Enemy Sighting:
├── Red dot at location
├── Duration: 3 seconds after shooting
├── Fades out gradually
├── Only shows when enemy fires

Explosion/Ability Use:
├── Flash icon at location
├── Brief duration (1 second)
├── Helps locate combat
└── Optional setting
```

---

## 7. Ping System Integration

### 7.1 Ping Markers on Minimap
```
Ping Types:
├── Generic: White diamond ◆
├── Enemy: Red skull ☠
├── Loot: Gold chest 📦
├── Danger: Yellow warning ⚠
├── Help: Blue flag 🚩
└── Going Here: Green arrow ➤

Size: 10pt
Animation: Pulse 3 times, then static
Duration: 10 seconds
```

### 7.2 Placing Pings
```
From Minimap:
├── Tap and hold on minimap
├── Ping wheel appears
├── Drag to select ping type
├── Release to place
└── Audio feedback to team

Quick Ping:
├── Double-tap on minimap
├── Places generic ping
├── Fastest method
└── Useful in combat
```

---

## 8. Zoom and Pan

### 8.1 Zoom Levels
| Level | Scale | Coverage | Use |
|-------|-------|----------|-----|
| 0.5x | Half | Wide area | Navigation |
| 1.0x | Normal | Balanced | Default |
| 1.5x | Closer | Local detail | Combat |
| 2.0x | Maximum | Immediate area | Precision |

### 8.2 Zoom Controls
```
Methods:
├── Pinch gesture on minimap
├── + / - buttons (if enabled)
├── Double-tap to toggle zoom
└── Settings slider for default

Behavior:
├── Smooth zoom animation
├── Center on player during zoom
├── Remember zoom between matches (optional)
└── Reset to default on match start
```

### 8.3 Pan Controls (Expanded View Only)
```
In Full Map View:
├── Drag to pan map
├── Player indicator stays visible
├── Tap anywhere to recenter on player
├── Edge pan (drag to edge continues)
└── Button to recenter
```

---

## 9. Full Map View

### 9.1 Activation
```
Trigger: Tap on minimap
Result: Expands to full-screen map view

Expansion Animation:
├── Duration: 0.3s
├── Easing: Ease-out
├── Minimap scales to cover screen
└── Other HUD fades out
```

### 9.2 Full Map Features
```
Full Map View Includes:
├── Entire game map visible
├── All named locations labeled
├── All teammates visible
├── Storm current and next
├── All active pings
├── Player facing indicator
├── Compass rose
└── Scale indicator

Additional Controls:
├── Zoom in/out (pinch or buttons)
├── Pan to explore
├── Place pings
├── Close button or tap outside
```

### 9.3 Closing Full Map
```
Methods:
├── Tap X/close button
├── Tap outside map
├── Swipe down gesture
├── Press back/escape
└── Auto-close in 10s (optional setting)

Behavior During Combat:
├── Taking damage closes map
├── Warning if storm approaching
└── Quick close prioritized
```

---

## 10. Rotation Modes

### 10.1 Fixed North (Default)
```
Behavior:
├── North always at top
├── Map doesn't rotate
├── Player arrow rotates
└── Traditional map behavior

Benefits:
├── Consistent orientation
├── Easy to learn locations
├── Matches real maps
└── Less disorienting
```

### 10.2 Rotating Map (Optional)
```
Behavior:
├── Player arrow always points up
├── Map rotates around player
├── North indicator shows north direction
└── Camera-relative movement

Benefits:
├── Intuitive direction matching
├── Turn left = left on map
├── Good for navigation
└── Some players prefer
```

### 10.3 Setting Toggle
```
Settings → HUD → Minimap → Rotation
├── [Fixed North] (default)
├── [Rotating]
└── Preview toggle effect
```

---

## 11. Information Priority

### 11.1 Layer Order (Bottom to Top)
```
Rendering Order:
1. Terrain (land, water)
2. Buildings and structures
3. Named location labels
4. Storm/safe zone
5. Points of interest
6. Ping markers
7. Teammate markers
8. Enemy indicators (brief)
9. Player marker (always on top)
10. UI frame and controls
```

### 11.2 Density Management
```
When Cluttered:
├── Nearby items have priority
├── Far items may be hidden
├── Important items always show
├── Labels can overlap-avoid
└── Zoom adjusts density

Always Visible:
├── Player
├── Teammates
├── Storm zone
├── Active pings
└── Named locations (nearby)
```

---

## 12. Performance Optimization

### 12.1 Update Frequency
| Element | Update Rate | Trigger |
|---------|-------------|---------|
| Player position | Every frame | Movement |
| Teammate positions | 10 Hz | Network sync |
| Storm position | 1 Hz | Timer |
| Pings | On event | Player action |
| Terrain | Never | Static |
| Combat markers | On event | Combat |

### 12.2 Rendering Optimization
```
Techniques:
├── Pre-rendered terrain texture
├── Only update changed elements
├── LOD for distant markers
├── Batch icon rendering
├── Cache full map texture
└── Async loading for expansion

Memory Budget:
├── Terrain texture: ~512 KB
├── Icons atlas: ~128 KB
├── Runtime data: ~32 KB
└── Total: ~700 KB
```

### 12.3 CPU Budget
```
Per Frame:
├── Minimap update: < 0.2ms
├── Position calculations: < 0.1ms
├── Render to texture: < 0.3ms
└── Total: < 0.6ms

Full Map Expansion:
├── One-time cost: ~5ms
├── Subsequent updates: < 1ms
└── Acceptable delay
```

---

## 13. Accessibility

### 13.1 Size Options
```
Settings:
├── Minimap Size: Small / Medium / Large
├── Icon Size: Normal / Large / Extra Large
├── Text Size: Normal / Large
└── Touch Target: Normal / Expanded
```

### 13.2 Colorblind Support
| Element | Default | CB Mode |
|---------|---------|---------|
| Safe zone | White | White |
| Storm | Red/Purple | Red/Orange |
| Teammate | Blue | Cyan |
| Enemy | Red | Orange |
| Ping types | Various | Shape + Color |

### 13.3 High Contrast Mode
```
When Enabled:
├── Increased border thickness
├── Brighter colors
├── Darker backgrounds
├── Larger icons
└── Bolder text
```

---

## 14. Settings and Customization

### 14.1 Minimap Settings Menu
```
Settings → HUD → Minimap:
├── Size
│   └── [Small] [Medium] [Large] [Custom: 100pt]
│
├── Position
│   └── [Top-Left] [Top-Right] [Bottom-Left] [Bottom-Right]
│
├── Opacity
│   └── [████████░░] 80%
│
├── Rotation
│   └── [Fixed North] [Rotating]
│
├── Default Zoom
│   └── [█████░░░░░] 1.0x
│
├── Show Elements
│   ├── Named Locations: [On]
│   ├── Chest Markers: [Off]
│   ├── Combat Indicators: [On]
│   └── Field of View: [Off]
│
├── Style
│   └── [Treasure Map] [Simple]
│
└── [Reset to Defaults]
```

### 14.2 Quick Settings
```
Accessible from minimap:
├── Long-press opens quick menu
├── Zoom slider
├── Opacity slider
├── Rotation toggle
└── Expand to full map
```

---

## 15. Integration Points

### 15.1 System Dependencies
| System | Data Received |
|--------|---------------|
| Player System | Position, facing |
| Team System | Teammate positions |
| Storm System | Zone positions, timing |
| Ping System | Ping locations, types |
| Combat System | Damage events |
| Map Data | Terrain, locations |

### 15.2 Events
```csharp
// Minimap events
public event Action OnMinimapExpanded;
public event Action OnMinimapCollapsed;
public event Action<Vector2> OnMinimapTapped;  // World position
public event Action<PingType, Vector2> OnPingPlaced;
public event Action<float> OnZoomChanged;
```

---

## 16. Testing Requirements

### 16.1 Functional Tests
```
Verify:
├── Player position accurate
├── Facing direction correct
├── Storm visualization accurate
├── Teammate positions synced
├── Ping placement works
├── Zoom controls function
├── Full map expansion works
├── All settings apply correctly
└── Rotation modes work
```

### 16.2 Visual Tests
```
Verify:
├── All icons distinguishable
├── Text readable at all sizes
├── Storm boundaries clear
├── Colors work in CB modes
├── No visual glitches
├── Smooth animations
└── Proper layering
```

### 16.3 Performance Tests
```
Verify:
├── 60 FPS maintained
├── No frame drops on expansion
├── Memory within budget
├── CPU usage acceptable
└── Works on low-end devices
```

---

## 17. Default Configuration

```json
{
  "minimap": {
    "enabled": true,
    "size": 100,
    "position": "top-left",
    "opacity": 0.8,
    "rotation": "fixed",
    "defaultZoom": 1.0,
    "style": "treasureMap"
  },
  "display": {
    "namedLocations": true,
    "chestMarkers": false,
    "combatIndicators": true,
    "fieldOfView": false,
    "teamColors": true,
    "compassRose": true
  },
  "interaction": {
    "tapToExpand": true,
    "pinchToZoom": true,
    "holdToPing": true,
    "doubleTapQuickPing": true,
    "autoCloseFullMap": false
  },
  "storm": {
    "showCurrentZone": true,
    "showNextZone": true,
    "showPhaseInfo": true,
    "directionArrow": true
  },
  "accessibility": {
    "iconSize": "normal",
    "textSize": "normal",
    "highContrast": false,
    "colorblindMode": "default"
  }
}
```

---

## Appendix A: Visual Reference

```
Minimap Layout:
┌──────────────────────────┐
│ N                        │
│ ┌──────────────────────┐ │
│ │    ·····             │ │
│ │  ··     ··  Storm    │ │
│ │ ·  ┌───┐  ·          │ │
│ │·   │SAF│   ·  ● Team │ │
│ │ · ▲│ E │  ·          │ │
│ │  · └───┘ ·   Player  │ │
│ │   ··   ··            │ │
│ │     ···   - - -      │ │
│ │           Next Zone  │ │
│ └──────────────────────┘ │
│        Tap to expand     │
└──────────────────────────┘

Full Map View:
┌──────────────────────────────────────────┐
│  [X]                           ⊕ ⊖ 🔄   │
│                                          │
│     PIRATE'S                             │
│     COVE        ● You                    │
│                 ↑                        │
│                ╱ ╲                       │
│     ┌─────────╱   ╲──────────┐           │
│     │ SAFE   ╱     ╲         │           │
│     │  ZONE ╱       ╲        │           │
│     │      ╱  SKULL  ╲       │           │
│     │     ╱   ROCK    ╲      │           │
│     └────╱─────────────╲─────┘           │
│         ╱               ╲                │
│        ● Teammate        ╲               │
│                           ╲              │
│    STORM AREA         SHIPWRECK          │
│                        BAY               │
└──────────────────────────────────────────┘
```

---

## Appendix B: Icon Reference

| Icon | Meaning | Size |
|------|---------|------|
| ▲ | Player (facing up) | 8pt |
| ● | Teammate | 6pt |
| ☠ | Enemy ping | 10pt |
| ◆ | Generic ping | 10pt |
| ⚠ | Danger ping | 10pt |
| 📦 | Loot marker | 8pt |
| ⭐ | Point of interest | 6pt |
| N | North indicator | 10pt |

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
