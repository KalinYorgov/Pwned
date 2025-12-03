# UX-003: Ability Button Layout Specification

## Overview

This document specifies the touch button layout for abilities and basic attack in Plunderstorm Mobile. The design prioritizes accessibility, reliability, and quick muscle memory development while preventing accidental activations.

## Design Philosophy

Ability buttons must be reachable without looking at the screen, distinguish between ability types at a glance, and provide clear feedback for cooldowns and availability. The layout adapts to various screen sizes while maintaining consistent interaction patterns.

---

## 1. Layout Configuration

### 1.1 Default Layout
```
Right Side of Screen:

           [Utility 1]
                 ╲
    [Off 1]───────[Basic Attack]───────[Off 2]
                 ╱
           [Utility 2]

Layout Style: Diamond/Cross pattern
Basic Attack: Center position (easiest to reach)
Offensive Abilities: Horizontal positions (combat focus)
Utility Abilities: Vertical positions (support actions)
```

### 1.2 Button Positions (Default)
| Button | X Position | Y Position | Description |
|--------|------------|------------|-------------|
| Basic Attack | 85% from left | 70% from top | Center, most accessible |
| Offensive 1 | 72% from left | 70% from top | Left of center |
| Offensive 2 | 98% from left | 70% from top | Right of center |
| Utility 1 | 85% from left | 55% from top | Above center |
| Utility 2 | 85% from left | 85% from top | Below center |

### 1.3 Alternative Layouts
```
Arc Layout:
    [U1] [O1] [BA] [O2] [U2]
    All buttons in a curved arc along thumb reach

Cluster Layout:
    [U1] [O1]
    [U2] [BA] [O2]
    Compact 2x3 grid arrangement

MOBA Layout:
    [O1] [O2]
    [U1] [U2]
       [BA]
    Traditional MOBA-style arrangement
```

---

## 2. Button Specifications

### 2.1 Button Sizes
| Preset | Diameter | Touch Target | Minimum (Accessibility) |
|--------|----------|--------------|------------------------|
| Small | 44pt | 54pt | Apple HIG minimum |
| Medium | 56pt | 66pt | Default |
| Large | 68pt | 78pt | Recommended for tablets |
| Extra Large | 80pt | 90pt | Accessibility option |

### 2.2 Touch Target Expansion
```
Visual Button: 56pt diameter
Touch Target: 66pt diameter (+10pt padding)
Overlap Prevention: 10pt minimum gap between targets
Priority Resolution: Center takes priority over edges
```

### 2.3 Button Spacing
| Configuration | Spacing Between Buttons |
|--------------|------------------------|
| Compact | 10pt (minimum) |
| Default | 20pt |
| Spread | 35pt |
| Custom | 5pt - 50pt range |

---

## 3. Visual Design

### 3.1 Button Anatomy
```
┌─────────────────────┐
│  ┌───────────────┐  │  ← Outer Ring (Type indicator)
│  │   ┌───────┐   │  │
│  │   │ ICON  │   │  │  ← Ability Icon (center)
│  │   └───────┘   │  │
│  │  [1]     CD   │  │  ← Slot number (bottom left)
│  └───────────────┘  │  ← Cooldown overlay
└─────────────────────┘    (sweeps clockwise)

Components:
├── Outer Ring: Type color indicator
├── Icon Area: Ability-specific icon
├── Slot Number: 1-4 for abilities, none for basic
├── Cooldown Overlay: Semi-transparent sweep
└── Rank Indicator: Pips or number for ability rank
```

### 3.2 Color Coding
| Ability Type | Ring Color | Glow Color | RGB Values |
|--------------|------------|------------|------------|
| Basic Attack | Gold/Yellow | Warm yellow | #FFD700 |
| Offensive 1-2 | Red | Red glow | #FF4444 |
| Utility 1-2 | Blue | Blue glow | #4488FF |
| On Cooldown | Gray | None | #666666 |
| Unavailable | Dark Gray | None | #333333 |

### 3.3 Button States
```
Available:
├── Full opacity (100%)
├── Type color ring visible
├── Icon clearly visible
├── Subtle pulse animation (optional)
└── Ready for activation

On Cooldown:
├── Reduced opacity (60%)
├── Gray ring
├── Cooldown sweep overlay
├── Remaining time displayed
└── Unresponsive to taps

Pressed:
├── Scale down to 90%
├── Brighter glow effect
├── Haptic feedback
└── Duration: while finger down

Targeting Mode:
├── Button highlighted
├── Targeting indicator emanates
├── Other buttons dimmed (70%)
└── Cancel zone indicated

Disabled/Locked:
├── Very low opacity (30%)
├── Lock icon overlay
├── No response to input
└── Tooltip shows reason
```

---

## 4. Basic Attack Button

### 4.1 Design
```
Position: Center of ability cluster
Size: Same or slightly larger than abilities
Appearance: Distinct from abilities
├── Sword/cutlass icon
├── Gold/yellow color theme
├── No cooldown display (very short CD)
└── Always available (no slot system)
```

### 4.2 Interaction
| Action | Result |
|--------|--------|
| Tap | Fire basic attack in facing direction |
| Tap + Aim Assist | Fire at nearest enemy in cone |
| Hold | Rapid-fire mode (if enabled) |
| Swipe from | Enter basic attack targeting |

### 4.3 Auto-Attack Option
```
Settings Toggle: Auto-Attack
When Enabled:
├── Basic attack fires automatically at nearby enemies
├── Interval: 0.8s (matches cooldown)
├── Target: Nearest enemy in range
├── Cancel: Player movement input
└── Override: Manual tap always prioritized
```

---

## 5. Ability Buttons

### 5.1 Activation Methods

#### 5.1.1 Instant Abilities
```
Definition: Abilities that don't require targeting
Examples: Barrel Roll, Healing Grog, Smoke Bomb

Activation:
├── Tap button → Immediate activation
├── Cooldown starts immediately
├── Visual/audio feedback confirms use
└── No targeting phase required
```

#### 5.1.2 Skillshot Abilities
```
Definition: Abilities requiring aim direction
Examples: Fireball, Lightning Strike, Cannonball

Activation Options:
1. Drag-and-Release (Default)
   ├── Press and hold button
   ├── Drag to aim direction
   ├── Release to fire
   └── Drag back to button to cancel

2. Tap-Tap Mode
   ├── Tap button to enter targeting
   ├── Tap target location to fire
   └── Tap button again to cancel

3. Quick Cast Mode
   ├── Tap button to immediately fire
   ├── Direction: Player's facing direction
   └── No targeting phase (for experts)
```

### 5.2 Targeting Indicator
```
When Ability Button Held:

Direction Indicator:
├── Line/arrow from player toward drag direction
├── Length = ability range
├── Color = ability type color
├── Thickness: 4pt

Range Indicator:
├── Circle/arc showing max range
├── Semi-transparent fill
├── Edge clearly visible
└── For ground-targeted abilities

Trajectory Preview:
├── Arc path for projectiles (Fireball, Cannonball)
├── Straight line for instant (Lightning Strike)
├── Area effect for AoE abilities
└── Updates in real-time with drag
```

### 5.3 Cancel Mechanism
```
Cancel Options:
1. Drag finger back to button center
2. Lift finger inside cancel zone (20pt ring around button)
3. Tap different button (switches ability)
4. Tap outside all buttons (setting-dependent)

Visual Cancel Zone:
├── Appears as ring around button during targeting
├── Color: Red/warning
├── Text: "Cancel" or X icon
└── Haptic: Light pulse when entering zone
```

---

## 6. Cooldown Display

### 6.1 Visual Representation
```
Cooldown Sweep:
├── Direction: Clockwise from 12 o'clock
├── Overlay Color: Black with 70% opacity
├── Progress: Reveals button as CD progresses
└── Animation: Smooth, no stepping

Time Display:
├── Position: Center of button
├── Font: Bold, easy to read
├── Format: Seconds (round to 0.1 for < 5s)
├── Color: White with shadow for contrast
└── Examples: "5", "2.3", "0.8"
```

### 6.2 Cooldown Values Display
| Time Remaining | Display Format | Update Rate |
|----------------|----------------|-------------|
| > 10 seconds | "12" (integer) | Every second |
| 5-10 seconds | "8" (integer) | Every second |
| 1-5 seconds | "3.5" (decimal) | Every 0.1s |
| < 1 second | "0.8" (decimal) | Every 0.1s |

### 6.3 Ready Notification
```
When Cooldown Ends:
├── Brief flash/glow effect
├── Subtle audio cue (optional, setting)
├── Haptic feedback (optional, setting)
└── Button returns to full opacity
```

---

## 7. Ability Rank Display

### 7.1 Rank Indicator
```
Position: Bottom of button or integrated with icon
Format: Pips (●●○) or number (2/3)

Rank 1: ●○○ or [1]
Rank 2: ●●○ or [2]
Rank 3: ●●● or [3]

Style Options:
├── Pips: Small dots below icon
├── Number: Badge in corner
├── Integrated: Icon changes with rank
└── Border: Thicker/brighter border at higher ranks
```

### 7.2 Upgrade Available Indicator
```
When Upgrade Available:
├── Flashing up-arrow icon
├── Pulsing glow effect
├── Quick-tap to upgrade (hold to cast)
└── Can be upgraded during match from drops
```

---

## 8. Button Customization

### 8.1 Layout Editor
```
Entering Editor:
Settings → Controls → Customize Button Layout

Editor Features:
├── All buttons draggable
├── Grid snapping (optional)
├── Distance guides between buttons
├── Preview mode (simulate gameplay)
├── Reset to default positions
├── Save multiple layouts
└── Import/export layouts
```

### 8.2 Per-Button Settings
| Setting | Options | Default |
|---------|---------|---------|
| Size | Small/Medium/Large/XL | Medium |
| Opacity | 20%-100% | 80% |
| Position | Drag anywhere | Preset |
| Lock Position | On/Off | Off |

### 8.3 Layout Presets
```
Preset Layouts:
├── Default: Diamond pattern
├── MOBA Style: Grid with BA bottom
├── Shooter Style: BA large, abilities small
├── Left-Handed: Mirrored layout
├── Tablet Optimized: Spread layout
└── Custom 1-3: User saved layouts
```

---

## 9. Feedback Systems

### 9.1 Haptic Feedback
| Event | Haptic Type | Intensity |
|-------|-------------|-----------|
| Button Press | Light Impact | Low |
| Ability Activated | Medium Impact | Medium |
| Ability Ready | Soft Tick | Low |
| Cancel Ability | Error Pattern | Medium |
| On Cooldown Tap | None or Light | Very Low |

### 9.2 Audio Feedback
```
Sound Events:
├── Button Press: Soft click
├── Ability Cast: Ability-specific sound
├── Cooldown Ready: Subtle chime (optional)
├── Cancel: Negative feedback sound
└── Cooldown Blocked: Error/locked sound

Settings:
├── UI Sounds: On/Off
├── Cooldown Alerts: On/Off
├── Volume: Independent slider
└── Voice Callouts: On/Off (announces ability)
```

### 9.3 Visual Feedback
```
Press Feedback:
├── Scale: 90% while pressed
├── Glow: Intensified
├── Animation: Slight bounce on release
└── Duration: Instant response

Cast Feedback:
├── Flash effect on button
├── Particle burst from button
├── Icon briefly enlarges
└── Ring pulse outward
```

---

## 10. Screen Size Adaptation

### 10.1 Phone Layouts
```
Small Phone (< 5.5"):
├── Compact spacing
├── Medium button size
├── Buttons closer to edges
└── Reduced visual decoration

Standard Phone (5.5" - 6.5"):
├── Default spacing
├── Medium button size
├── Standard positions
└── Full visual effects

Large Phone (> 6.5"):
├── Increased spacing
├── Medium to Large buttons
├── More centered positions
└── Full visual effects
```

### 10.2 Tablet Layouts
```
Tablet (7"+):
├── Spread spacing
├── Large buttons
├── Multiple layout options
├── Optional split-hand layouts
└── Landscape optimized
```

### 10.3 Aspect Ratio Handling
| Aspect Ratio | Adaptation |
|--------------|------------|
| 16:9 | Standard positions |
| 18:9 | Shift buttons toward center |
| 19.5:9 | Extra edge padding |
| 21:9 | Consider split layout |
| 4:3 (tablet) | Spread for reachability |

---

## 11. Accessibility Features

### 11.1 Button Size Scaling
```
Accessibility Options:
├── Scale: 100% - 200%
├── Touch Target: Extra expansion
├── Spacing: Increased for motor accessibility
└── Override: Works with system settings
```

### 11.2 Color Blindness Support
| Mode | Offensive Color | Utility Color | Basic Color |
|------|-----------------|---------------|-------------|
| Default | Red | Blue | Gold |
| Deuteranopia | Orange | Purple | Gold |
| Protanopia | Orange | Cyan | Gold |
| Tritanopia | Red | Green | Gold |
| High Contrast | Red + Shape | Blue + Shape | Gold + Shape |

### 11.3 Shape Differentiation
```
In Addition to Color:
├── Offensive: Square corners
├── Utility: Rounded corners
├── Basic: Circular
└── Icons: Distinct silhouettes
```

### 11.4 One-Handed Mode
```
Configuration:
├── All buttons reachable by one thumb
├── Arc layout along screen edge
├── Basic attack in easiest position
├── Ability activation via gestures optional
└── Works with either hand
```

---

## 12. Performance Optimization

### 12.1 Rendering Budget
| Component | Budget |
|-----------|--------|
| Button rendering | < 0.5ms/frame |
| Cooldown updates | < 0.1ms/frame |
| Animation updates | < 0.2ms/frame |
| Touch processing | < 0.2ms/frame |
| Total | < 1ms/frame |

### 12.2 Asset Management
```
Button Assets:
├── Pre-rendered at all sizes
├── Atlas-packed for efficiency
├── Cooldown as shader effect
├── Icons: 128x128 max resolution
└── Total memory: ~2MB
```

### 12.3 Optimization Techniques
```
Performance Features:
├── Dirty flag system (only update on change)
├── Batch render all buttons
├── Cooldown shader instead of geometry
├── Disable animations on low-end
└── Reduce effects in battery saver mode
```

---

## 13. Integration Points

### 13.1 System Dependencies
| System | Integration |
|--------|-------------|
| Ability System | Receives activation commands |
| Cooldown Manager | Provides cooldown states |
| Targeting System | Communicates targeting state |
| Input Manager | Touch event handling |
| Network | Sends ability inputs to server |

### 13.2 Events
```csharp
// Button events for other systems
public event Action<int> OnAbilityPressed;      // Slot index
public event Action<int> OnAbilityReleased;     // Slot index
public event Action<int, Vector2> OnAbilityAimed; // Slot, direction
public event Action OnBasicAttackPressed;
public event Action<int> OnAbilityCanceled;     // Slot index
public event Action OnLayoutChanged;
```

### 13.3 Data Flow
```
Input → Button Handler → Validation → Ability System
                      ↓
                 Visual Feedback
                      ↓
                 Audio/Haptic
```

---

## 14. Testing Requirements

### 14.1 Functional Tests
```
Test Cases:
├── Each button responds to tap
├── Targeting mode enters on hold
├── Direction accurately follows drag
├── Cancel works in all methods
├── Cooldowns display correctly
├── Cooldown blocks activation
├── Rank displays update correctly
├── Customization persists
├── Presets load correctly
└── Layout editor functions
```

### 14.2 Usability Tests
```
Metrics to Measure:
├── Time to activate ability (< 100ms)
├── Accuracy of targeting (> 90% hit intended)
├── Accidental activation rate (< 1%)
├── Muscle memory development (by match 3)
├── Player preference for layouts
└── Accessibility compliance
```

### 14.3 Device Matrix
| Device | Priority | Specific Tests |
|--------|----------|----------------|
| iPhone SE | High | Small screen, button reachability |
| iPhone 14 | High | Standard reference device |
| Pixel 7 | High | Android reference |
| iPad | Medium | Tablet layout |
| Galaxy Fold | Medium | Foldable handling |

---

## 15. Default Configuration

```json
{
  "layout": {
    "preset": "default",
    "buttonSize": "medium",
    "spacing": "default",
    "opacity": 0.8
  },
  "positions": {
    "basicAttack": { "x": 0.85, "y": 0.70 },
    "offensive1": { "x": 0.72, "y": 0.70 },
    "offensive2": { "x": 0.98, "y": 0.70 },
    "utility1": { "x": 0.85, "y": 0.55 },
    "utility2": { "x": 0.85, "y": 0.85 }
  },
  "targeting": {
    "method": "dragAndRelease",
    "quickCastEnabled": false,
    "cancelZoneSize": 20
  },
  "feedback": {
    "haptic": true,
    "audio": true,
    "cooldownAlerts": true,
    "pressAnimation": true
  },
  "accessibility": {
    "colorBlindMode": "default",
    "extraLargeTouchTargets": false,
    "shapeIndicators": false
  }
}
```

---

## Appendix A: Visual Reference

```
Default Diamond Layout:
┌────────────────────────────────────────────────────────┐
│                                                        │
│                                          [Utility 1]   │
│                                             Blue       │
│                                                        │
│    Movement Zone      Camera Zone      [O1]  [BA]  [O2]│
│                                        Red   Gold  Red │
│                                                        │
│                                          [Utility 2]   │
│                                             Blue       │
│                                                        │
└────────────────────────────────────────────────────────┘

Button Detail:
    ╭────────────────╮
   │ ╭────────────╮  │
   │ │    ⚔️      │  │  ← Ability Icon
   │ │   ICON     │  │
   │ ╰────────────╯  │
   │ [2]      3.5   │  ← Slot#, Cooldown
   │  ●●○           │  ← Rank pips
    ╰────────────────╯
       Red Ring → Offensive Type
```

---

## Appendix B: State Machine

```
Button States:

     ┌─────────────────────────────────────┐
     │                                     │
     ▼                                     │
┌─────────┐  touch   ┌─────────┐  release  │
│Available│─────────▶│ Pressed │───────────┘
└─────────┘          └─────────┘     (instant ability)
     ▲                    │
     │                    │ hold (skillshot)
     │                    ▼
     │              ┌──────────┐  release
     │              │ Targeting│─────────▶ Cast
     │              └──────────┘
     │                    │
     │                    │ cancel
     │                    ▼
     │              ┌──────────┐
     └──────────────│ Canceled │
                    └──────────┘

Cooldown Flow:
Cast ──▶ Cooldown Active ──▶ Cooldown Ready ──▶ Available
              │                     │
              └─── timer tick ──────┘
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
