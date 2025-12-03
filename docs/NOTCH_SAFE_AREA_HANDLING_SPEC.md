# UX-024: Notch and Safe Area Handling Specification

## Overview

This document specifies how Plunderstorm Mobile handles device safe areas, including notches, punch-holes, rounded corners, and home indicators across all supported iOS and Android devices. Proper safe area handling ensures no UI elements are obscured or difficult to interact with.

## Design Philosophy

UI must adapt automatically to all device form factors without manual configuration. Critical interactive elements and information must remain fully visible and accessible. The gameplay area can extend edge-to-edge for immersion, but UI overlays must respect safe boundaries.

---

## 1. Safe Area Types

### 1.1 Safe Area Definitions
| Area Type | Description | Affected Devices |
|-----------|-------------|------------------|
| Notch | Top display cutout | iPhone X-14, various Android |
| Punch-hole | Camera cutout in display | Samsung, Pixel, OnePlus |
| Dynamic Island | Floating pill cutout | iPhone 14 Pro+ |
| Rounded Corners | Display corner rounding | Most modern phones |
| Home Indicator | Bottom gesture bar | iPhone X+, gesture Android |
| Status Bar | Top system bar | All devices |
| Navigation Bar | Bottom navigation | Android devices |

### 1.2 Safe Area Insets
```
┌──────────────────────────────────────────────────────────┐
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ ← Top inset
│ ▓                                                      ▓ │
│ ▓                                                      ▓ │
│ ▓                                                      ▓ │
│ ▓              SAFE CONTENT AREA                       ▓ │ ← Side insets
│ ▓                                                      ▓ │   (minimal on most)
│ ▓                                                      ▓ │
│ ▓                                                      ▓ │
│ ▓                                                      ▓ │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ ← Bottom inset
└──────────────────────────────────────────────────────────┘

Legend:
▓ = Unsafe area (may be obscured)
  = Safe area (UI content goes here)
```

---

## 2. Device-Specific Safe Areas

### 2.1 iOS Devices (Landscape)
| Device | Top | Bottom | Left | Right | Notes |
|--------|-----|--------|------|-------|-------|
| iPhone SE | 0pt | 0pt | 0pt | 0pt | No notch |
| iPhone 8/7 | 0pt | 0pt | 0pt | 0pt | No notch |
| iPhone X/XS | 0pt | 21pt | 44pt | 44pt | Notch left |
| iPhone XR/11 | 0pt | 21pt | 48pt | 48pt | Wider notch |
| iPhone 12/13 | 0pt | 21pt | 47pt | 47pt | Standard notch |
| iPhone 14 | 0pt | 21pt | 47pt | 47pt | Standard notch |
| iPhone 14 Pro | 0pt | 21pt | 59pt | 59pt | Dynamic Island |
| iPhone 15 Pro | 0pt | 21pt | 59pt | 59pt | Dynamic Island |
| iPad Pro | 20pt | 20pt | 20pt | 20pt | Rounded corners |

### 2.2 Android Devices (Landscape)
| Device Type | Top | Bottom | Left | Right | Notes |
|-------------|-----|--------|------|-------|-------|
| No Cutout | 0dp | 48dp* | 0dp | 0dp | Nav bar only |
| Center Notch | 0dp | 48dp* | 30dp | 30dp | Typical notch |
| Corner Punch | 0dp | 48dp* | 36dp | 0dp | Samsung style |
| Pill Punch | 0dp | 48dp* | 32dp | 32dp | Pixel style |
| Gesture Nav | 0dp | 16dp | varies | varies | No nav bar |

*48dp for 3-button nav, 16dp for gesture nav

### 2.3 Orientation-Specific
```
Portrait Mode:
├── Notch at top
├── Home indicator at bottom
├── Safe areas: top + bottom
└── Sides typically safe

Landscape Left (Home button right):
├── Notch on left
├── Home indicator on right
├── Safe areas: left + right
└── Top/bottom typically safe

Landscape Right (Home button left):
├── Notch on right
├── Home indicator on left
├── Safe areas: left + right
└── Top/bottom typically safe
```

---

## 3. UI Element Placement Rules

### 3.1 Critical Elements
```
MUST Be in Safe Area:
├── Health bar (critical info)
├── Ability buttons (interaction)
├── Minimap (important info)
├── Settings buttons (interaction)
├── Menu buttons (interaction)
├── Text labels (readability)
└── Interactive touch targets

CAN Extend Beyond:
├── Background visuals
├── Decorative elements
├── Non-interactive graphics
├── Screen effects (vignette)
└── Particle effects
```

### 3.2 Placement by Zone
```
Top-Left Zone (Minimap):
├── Offset from top: MAX(safeTop, 8pt)
├── Offset from left: MAX(safeLeft, 8pt)
└── Additional padding: 4pt

Top-Right Zone (Kill Feed):
├── Offset from top: MAX(safeTop, 8pt)
├── Offset from right: MAX(safeRight, 8pt)
└── Additional padding: 4pt

Top-Center Zone (Player Count):
├── Offset from top: MAX(safeTop, 8pt)
├── Avoid notch if centered
└── May need to shift for Dynamic Island

Bottom-Left Zone (Joystick):
├── Offset from bottom: MAX(safeBottom, 8pt)
├── Offset from left: MAX(safeLeft, 8pt)
└── Touch zone respects safe area

Bottom-Right Zone (Abilities):
├── Offset from bottom: MAX(safeBottom, 8pt)
├── Offset from right: MAX(safeRight, 8pt)
└── Touch zone respects safe area

Bottom-Center Zone (Health):
├── Offset from bottom: MAX(safeBottom, 8pt)
├── Centered horizontally
└── Width fits between side safe areas
```

---

## 4. Implementation

### 4.1 iOS Implementation
```swift
// Get safe area insets
let window = UIApplication.shared.windows.first
let safeAreaInsets = window?.safeAreaInsets ?? .zero

// Apply to UI
let topInset = safeAreaInsets.top
let bottomInset = safeAreaInsets.bottom
let leftInset = safeAreaInsets.left
let rightInset = safeAreaInsets.right

// In landscape, left/right become relevant
// Top/bottom swap based on orientation
```

### 4.2 Android Implementation
```kotlin
// Using WindowInsets API (API 30+)
ViewCompat.setOnApplyWindowInsetsListener(view) { v, insets ->
    val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
    val displayCutout = insets.getInsets(WindowInsetsCompat.Type.displayCutout())

    val topInset = maxOf(systemBars.top, displayCutout.top)
    val bottomInset = maxOf(systemBars.bottom, displayCutout.bottom)
    val leftInset = maxOf(systemBars.left, displayCutout.left)
    val rightInset = maxOf(systemBars.right, displayCutout.right)

    // Apply insets to UI
    insets
}
```

### 4.3 Unity/Cross-Platform
```csharp
// Unity Screen.safeArea
Rect safeArea = Screen.safeArea;
float topInset = Screen.height - safeArea.yMax;
float bottomInset = safeArea.yMin;
float leftInset = safeArea.xMin;
float rightInset = Screen.width - safeArea.xMax;

// Convert to UI coordinates
// Apply to RectTransform anchors
```

---

## 5. Dynamic Safe Area Handling

### 5.1 Orientation Changes
```
On Orientation Change:
1. Detect new orientation
2. Query new safe area insets
3. Recalculate UI positions
4. Animate transition (0.3s)
5. Validate touch targets
6. Update input zones
```

### 5.2 Foldable Devices
```
Samsung Fold/Flip Handling:
├── Detect fold state change
├── Query new screen dimensions
├── Query new safe areas
├── Reposition all UI elements
├── Consider split-screen mode
└── Test both unfolded and folded

Edge Cases:
├── App continuing when folding
├── Different safe areas per state
├── Aspect ratio changes
└── Input zone changes
```

### 5.3 Split-Screen/PiP
```
When in Split-Screen:
├── Reduced screen area
├── Different safe areas
├── Adjust UI scaling
├── Validate minimum sizes
└── Consider disabling (if too small)
```

---

## 6. Corner Radius Handling

### 6.1 Rounded Corner Detection
```
Corner Radius by Device:
├── iPhone X+: ~40pt radius
├── iPhone 14 Pro: ~55pt radius
├── iPad Pro: ~20pt radius
├── Android varies: 10-50dp
└── Some: Sharp corners (0dp)
```

### 6.2 Corner Avoidance
```
For Corner Elements:
├── Calculate corner danger zone
├── Inset UI from corners
├── Example: Minimap needs ~15pt extra inset on iPhone
├── Button corners need ~10pt extra

Corner Safe Zone Calculation:
cornerInset = cornerRadius - (cornerRadius * 0.7)
// Allows content in slightly rounded area
// but not extreme corner
```

### 6.3 Visual Reference
```
Rounded Corner Device:
╭──────────────────────────────────────────╮
│ ╲                                    ╱   │
│   ╲ Corner                      ╱        │
│     ╲ danger               ╱             │
│       ╲ zone           ╱                 │
│         ╲─────────────╱                  │
│                                          │
│              SAFE AREA                   │
│                                          │
│         ╱─────────────╲                  │
│       ╱                 ╲                │
│     ╱                     ╲              │
│   ╱                         ╲            │
│ ╱                             ╲          │
╰──────────────────────────────────────────╯
```

---

## 7. Touch Target Considerations

### 7.1 Minimum Touch Sizes
```
In Safe Area:
├── Minimum: 44pt × 44pt (Apple HIG)
├── Minimum: 48dp × 48dp (Material)
├── Recommended: 56pt/dp for primary actions
└── Comfortable: 68pt/dp for abilities

Near Safe Area Edges:
├── Add extra padding (8-16pt)
├── Ensure full touch target in safe area
├── Consider accidental edge touches
└── Test on actual devices
```

### 7.2 Edge Gestures
```
System Gesture Conflicts:
├── iOS: Swipe from left edge = back
├── iOS: Swipe from bottom = home/switcher
├── Android: Swipe from sides = back
├── Android: Swipe from bottom = home

Solutions:
├── Add 20pt buffer from gesture edges
├── No interactive elements at extreme edges
├── Let system gestures take priority
├── Consider edge gesture override (with caution)
```

---

## 8. Specific Element Handling

### 8.1 Minimap
```
Position Calculation:
├── baseX = safeLeft + margin (8pt)
├── baseY = safeTop + margin (8pt)
├── cornerAdjust = if (hasRoundedCorner) 10pt else 0pt
├── finalX = baseX + cornerAdjust
├── finalY = baseY + cornerAdjust

Result: Minimap appears 18pt from top-left on rounded devices
```

### 8.2 Ability Buttons
```
Position Calculation:
├── baseX = screenWidth - safeRight - margin - buttonWidth
├── baseY = screenHeight - safeBottom - margin - buttonHeight
├── cornerAdjust = if (hasRoundedCorner) 10pt else 0pt
├── Ensure all buttons in safe area
├── Maintain spacing between buttons

Touch Target:
├── Must extend full 44pt minimum
├── Cannot overlap safe area boundary
├── Add visual inset if touch extends beyond
```

### 8.3 Health Bar
```
Position Calculation:
├── centerX = screenWidth / 2
├── baseY = screenHeight - safeBottom - margin - barHeight
├── maxWidth = screenWidth - safeLeft - safeRight - (margin * 2)
├── actualWidth = min(desiredWidth, maxWidth)

Centered, respects both side safe areas
```

### 8.4 Full-Screen Overlays
```
Modal Dialogs:
├── Content inside safe area
├── Background can extend edge-to-edge
├── Close button inside safe area
├── Text fully visible

Loading Screens:
├── Progress indicators inside safe area
├── Background imagery edge-to-edge
├── Text and tips inside safe area
```

---

## 9. Testing Requirements

### 9.1 Device Matrix
| Priority | Devices | Reason |
|----------|---------|--------|
| Critical | iPhone 14 Pro | Dynamic Island |
| Critical | iPhone 12/13 | Common notch |
| Critical | Samsung S23 | Punch-hole |
| High | iPhone SE | No notch (small) |
| High | Pixel 7 | Center punch-hole |
| High | iPad Pro | Tablet + corners |
| Medium | iPhone 11 | Wider notch |
| Medium | OnePlus | Edge cutouts |
| Low | Older Android | Varying cutouts |

### 9.2 Test Cases
```
For Each Device:
├── All UI elements visible
├── No overlap with notch/cutout
├── Touch targets functional
├── Corners not cutting off UI
├── Orientation changes work
├── Text fully readable
├── Buttons fully tappable
└── No visual glitches

Edge Cases:
├── Extreme notch (max size)
├── Multiple cutouts
├── Asymmetric cutouts
├── Low resolution displays
├── High resolution displays
├── Foldable devices
└── Split-screen mode
```

### 9.3 Simulator vs Real Device
```
Simulator Testing:
├── Good for layout verification
├── May not match exact insets
├── Cannot test touch near edges
├── Good for initial development

Real Device Testing:
├── Required for final validation
├── Exact safe area values
├── Touch zone testing
├── Edge gesture conflicts
├── Performance verification
```

---

## 10. Debug Tools

### 10.1 Safe Area Visualization
```
Debug Mode:
├── Draw safe area boundaries (green)
├── Draw danger zones (red)
├── Show inset values on screen
├── Highlight elements outside safe area
├── Toggle via settings/developer menu
```

### 10.2 Device Info Display
```
Debug Overlay:
├── Device model
├── Screen resolution
├── Safe area insets (all 4)
├── Orientation
├── Scale factor
├── Corner radius (estimated)
└── Toggle via triple-tap version number
```

---

## 11. Fallback Handling

### 11.1 Unknown Devices
```
When Safe Area Unknown:
├── Use conservative defaults
├── Minimum insets: 20pt all sides
├── Additional corner insets: 10pt
├── Log warning for analytics
└── Manual override in settings (advanced)
```

### 11.2 Edge Cases
```
If Safe Area is 0 on All Sides:
├── Likely old device or simulator
├── Apply minimum margins anyway (8pt)
├── Log for debugging
└── Prevent edge-to-edge placement

If Safe Area is Unusually Large:
├── May be accessibility setting
├── Respect reported values
├── Scale UI if needed
├── Ensure minimum gameplay area
```

---

## 12. Performance

### 12.1 Optimization
```
Safe Area Queries:
├── Cache safe area values
├── Only recalculate on orientation change
├── Avoid per-frame queries
├── Use event-driven updates
└── Pre-calculate positions on change
```

### 12.2 Layout Updates
```
On Safe Area Change:
├── Batch UI position updates
├── Single layout pass
├── Avoid cascading recalculations
├── Animate transitions smoothly
└── Target: < 16ms layout time
```

---

## 13. Default Configuration

```json
{
  "safeArea": {
    "enabled": true,
    "additionalPadding": {
      "top": 8,
      "bottom": 8,
      "left": 8,
      "right": 8
    },
    "cornerHandling": true,
    "cornerPadding": 10,
    "debugVisualization": false
  },
  "fallback": {
    "defaultInset": 20,
    "minGameplayArea": {
      "width": 320,
      "height": 180
    }
  },
  "touchZones": {
    "edgeBuffer": 20,
    "systemGestureBuffer": 16,
    "minimumTouchTarget": 44
  }
}
```

---

## Appendix A: Visual Reference

```
iPhone 14 Pro (Landscape Left):
┌──────────────────────────────────────────────────────────────┐
│        ╭───────╮                                             │
│        │ISLAND │                                             │
│        ╰───────╯                                             │
│ ▓▓                                                        ▓▓ │
│ ▓▓ ┌─────────┐                           ┌─────────┐      ▓▓ │
│ ▓▓ │ Minimap │     SAFE GAMEPLAY AREA    │Kill Feed│      ▓▓ │
│ ▓▓ └─────────┘                           └─────────┘      ▓▓ │
│ ▓▓                                                        ▓▓ │
│ ▓▓                                                        ▓▓ │
│ ▓▓ ┌─────┐     ┌──────────────────┐      ┌─────────────┐  ▓▓ │
│ ▓▓ │ Joy │     │    Health Bar    │      │  Abilities  │  ▓▓ │
│ ▓▓ └─────┘     └──────────────────┘      └─────────────┘  ▓▓ │
│                        ───────                               │
│                       Home Indicator                         │
└──────────────────────────────────────────────────────────────┘

Android with Punch-Hole (Landscape):
┌──────────────────────────────────────────────────────────────┐
│ ●                                                            │
│ ▓▓                                                        ▓▓ │
│ ▓▓ ┌─────────┐                           ┌─────────┐      ▓▓ │
│ ▓▓ │ Minimap │     SAFE GAMEPLAY AREA    │Kill Feed│      ▓▓ │
│ ▓▓ └─────────┘                           └─────────┘      ▓▓ │
│ ▓▓                                                        ▓▓ │
│ ▓▓ ┌─────┐     ┌──────────────────┐      ┌─────────────┐  ▓▓ │
│ ▓▓ │ Joy │     │    Health Bar    │      │  Abilities  │  ▓▓ │
│ ▓▓ └─────┘     └──────────────────┘      └─────────────┘  ▓▓ │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│                    [  ][  ][  ] Navigation Bar               │
└──────────────────────────────────────────────────────────────┘

● = Camera punch-hole
▓ = Unsafe/inset area
```

---

## Appendix B: Common Mistakes

```
DON'T:
├── Place interactive elements at screen edges
├── Assume all devices have same safe areas
├── Ignore home indicator on iPhone
├── Put text in corner danger zones
├── Make touch targets that extend beyond safe area
├── Forget to test both landscape orientations
├── Use hardcoded positions without safe area offset
└── Ignore foldable device states

DO:
├── Query safe area insets from OS
├── Add padding beyond safe area for comfort
├── Test on real devices
├── Handle orientation changes
├── Animate UI transitions on safe area change
├── Log unknown device configurations
├── Provide debug visualization
└── Design with largest notch in mind
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
