# UX-001: Virtual Joystick Movement Specification

## Overview

This document specifies the virtual joystick movement system for Plunderstorm Mobile, providing responsive and intuitive touch-based player movement controls optimized for mobile devices.

## Design Philosophy

The virtual joystick is the primary movement input method for mobile players. It must feel natural, responsive, and reliable across all device sizes and player hand sizes. The system prioritizes low input latency and customization to accommodate diverse player preferences.

---

## 1. Touch Zone Configuration

### 1.1 Activation Zone
| Parameter | Value |
|-----------|-------|
| Zone Location | Left 40% of screen width |
| Zone Height | Full screen height |
| Minimum Touch Size | 44pt (Apple HIG) / 48dp (Material) |
| Touch Priority | Highest in movement zone |

### 1.2 Zone Behavior
- Touch anywhere in the activation zone spawns the joystick
- Zone is exclusive - no other touch actions in this area
- Multi-touch in zone ignored (only first touch registers)
- Zone boundaries visible in control customization screen

---

## 2. Joystick Types

### 2.1 Floating Joystick (Default)
```
Behavior:
- Joystick appears at initial touch point
- Center follows finger if dragged beyond max range
- Disappears when touch released
- No fixed position on screen

Benefits:
- Adapts to player's natural thumb position
- Works for all hand sizes
- No need to look at controls
```

### 2.2 Fixed Position Joystick (Optional)
```
Behavior:
- Joystick always visible at configured position
- Touch must start within joystick bounds
- Center never moves
- Always visible even when not touching

Benefits:
- Consistent muscle memory
- Visual reference for position
- Preferred by some players from other games
```

### 2.3 Configuration Options
| Setting | Options | Default |
|---------|---------|---------|
| Joystick Type | Floating / Fixed | Floating |
| Fixed Position X | 5% - 35% from left | 15% |
| Fixed Position Y | 50% - 90% from bottom | 75% |

---

## 3. Joystick Dimensions

### 3.1 Size Presets
| Preset | Outer Diameter | Inner Diameter | Dead Zone |
|--------|----------------|----------------|-----------|
| Small | 100pt | 40pt | 8pt |
| Medium | 130pt | 52pt | 10pt |
| Large | 160pt | 64pt | 12pt |
| Custom | 80-200pt | 32-80pt | 5-20pt |

### 3.2 Scaling for Device Size
```
Phone (< 7"):
- Default: Medium
- Recommended range: Small to Medium

Tablet (7"+):
- Default: Large
- Recommended range: Medium to Large

Auto-detection:
- System recommends size based on device diagonal
- Player can override at any time
```

---

## 4. Movement Mechanics

### 4.1 Dead Zone
| Parameter | Value |
|-----------|-------|
| Shape | Circular |
| Default Radius | 10pt (Medium preset) |
| Purpose | Prevent drift, allow thumb rest |
| Configurable | Yes (5-20pt range) |

### 4.2 Movement Speed Curve
```
Displacement Zones:
├── Dead Zone (0-10%): No movement
├── Walk Zone (10-50%): Proportional walk speed
│   └── Speed = displacement% × 0.5 × max_walk_speed
├── Run Zone (50-90%): Proportional run speed
│   └── Speed = displacement% × max_run_speed
└── Max Zone (90-100%): Maximum run speed
    └── Speed = max_run_speed
```

### 4.3 Speed Values
| Movement State | Speed (units/sec) | Animation |
|---------------|-------------------|-----------|
| Idle | 0 | Idle animation |
| Slow Walk | 0-100 | Walk (slow) |
| Walk | 100-250 | Walk (normal) |
| Run | 250-400 | Run |
| Max Run | 400 | Run (max) |
| Sprint | 500 | Sprint |

### 4.4 Direction Calculation
```
Algorithm:
1. Get touch position relative to joystick center
2. Calculate angle: atan2(deltaY, deltaX)
3. Convert to world-space direction relative to camera
4. Apply smoothing (lerp with previous direction)

Direction Output:
- 360-degree continuous direction
- 8-direction snapping option (optional accessibility)
- Camera-relative by default
```

---

## 5. Acceleration and Deceleration

### 5.1 Acceleration Curve
```
Acceleration Parameters:
├── Time to Max Speed: 0.15s (from stop)
├── Acceleration Rate: 2666 units/sec²
├── Curve Type: Ease-out (fast start, smooth end)
└── Formula: speed = max_speed × (1 - e^(-t/0.05))
```

### 5.2 Deceleration Curve
```
Deceleration Parameters:
├── Time to Stop: 0.1s (from max)
├── Deceleration Rate: 4000 units/sec²
├── Curve Type: Ease-in (momentum feel)
└── Formula: speed = current_speed × e^(-t/0.04)
```

### 5.3 Direction Change
```
Direction Smoothing:
├── Smooth Turn Rate: 720°/sec
├── Interpolation: Spherical lerp (slerp)
└── Sharp Turn Threshold: 135° (triggers quick turn animation)
```

---

## 6. Sprint Mechanic

### 6.1 Sprint Activation
| Method | Action |
|--------|--------|
| Double-Tap Hold | Tap twice quickly in movement zone, hold on second tap |
| Dedicated Button | Optional sprint button (configurable) |
| Auto-Sprint | Optional: sprint when joystick at max displacement for 1s |

### 6.2 Double-Tap Detection
```
Parameters:
├── Tap Window: 300ms (max time between taps)
├── Hold Threshold: 150ms (min hold time on second tap)
├── Max Movement: 30pt (max movement between taps)
└── Haptic Feedback: Light buzz on sprint activation
```

### 6.3 Sprint Behavior
| Parameter | Value |
|-----------|-------|
| Sprint Speed | 125% of max run speed (500 units/sec) |
| Sprint Stamina | Unlimited (no stamina system in MVP) |
| Sprint Direction | Forward only (camera-relative) |
| Sprint Cancel | Release touch, ability use, or damage taken |

---

## 7. Visual Feedback

### 7.1 Joystick Appearance
```
Base (Outer Ring):
├── Color: Semi-transparent white (#FFFFFF, 60% opacity)
├── Border: 2pt solid white (#FFFFFF, 80% opacity)
├── Style: Nautical compass rose design
└── Glow: Subtle outer glow when active

Stick (Inner Circle):
├── Color: Solid white (#FFFFFF, 90% opacity)
├── Border: 1pt darker outline
├── Icon: Anchor or ship wheel icon
└── Animation: Slight pulse when moving
```

### 7.2 State Indicators
| State | Visual Feedback |
|-------|-----------------|
| Idle | Base opacity 40%, no stick visible |
| Touched | Base opacity 60%, stick appears |
| Moving | Base opacity 80%, stick follows finger |
| Sprint | Gold/amber tint, speed lines effect |
| Disabled | Grayed out, X overlay |

### 7.3 Direction Indicator (Optional)
```
Arc Display:
├── Shows direction of movement
├── Arc length proportional to speed
├── Color gradient: White → Blue (sprint: Orange)
└── Configurable: On/Off in settings
```

---

## 8. Input Processing

### 8.1 Frame Timing
| Metric | Target | Maximum |
|--------|--------|---------|
| Input Polling Rate | Every frame | 16.67ms (60 FPS) |
| Touch-to-Movement Latency | < 16ms | 33ms |
| Visual Feedback Latency | < 8ms | 16ms |

### 8.2 Touch Processing Pipeline
```
Frame N:
1. Poll touch input (OS level)
2. Transform to screen coordinates
3. Check zone boundaries
4. Calculate joystick displacement
5. Apply dead zone
6. Calculate direction and magnitude
7. Apply smoothing
8. Send to movement system
9. Update visual feedback

Total Budget: < 2ms per frame
```

### 8.3 Interpolation
```
Between Frames:
├── Position interpolation for smooth visuals
├── Velocity interpolation for physics
├── Touch prediction (extrapolate 1 frame)
└── Catch-up on frame drops (max 3 frames)
```

---

## 9. Conflict Resolution

### 9.1 Multi-Touch Handling
```
Priority Rules:
1. First touch in zone = movement control
2. Additional touches ignored in movement zone
3. Touch outside zone = camera/ability controls
4. Touch transfer: none (release required to switch)
```

### 9.2 Zone Overlap Prevention
```
Buffer Zones:
├── 20pt buffer between movement and camera zones
├── Ability buttons cannot overlap movement zone
├── Menu buttons clear of all control zones
└── Debug: Show zone boundaries in settings
```

### 9.3 Touch Loss Handling
```
On Touch Lost (finger lifted):
1. Begin deceleration immediately
2. Fade joystick visual (0.15s)
3. Clear sprint state
4. Ready for new touch

On Touch Cancelled (system interrupt):
1. Same as touch lost
2. Log cancellation reason
3. Resume on next valid touch
```

---

## 10. Customization Options

### 10.1 Settings UI
```
Movement Settings:
├── Joystick Type: [Floating] [Fixed]
├── Joystick Size: [Small] [Medium] [Large] [Custom]
├── Dead Zone: ████████░░ (slider)
├── Sprint Mode: [Double-Tap] [Button] [Auto]
├── Direction Indicator: [On] [Off]
└── [Reset to Defaults]
```

### 10.2 Advanced Settings
| Setting | Range | Default |
|---------|-------|---------|
| Acceleration | 0.5x - 2.0x | 1.0x |
| Deceleration | 0.5x - 2.0x | 1.0x |
| Turn Smoothing | 0% - 100% | 50% |
| Opacity | 20% - 100% | 60% |

### 10.3 Layout Editor
```
Drag-and-Drop Editor:
├── Touch and hold joystick position to edit (fixed mode)
├── Grid snapping for alignment
├── Preview mode to test changes
├── Save/Load custom layouts
└── Reset to default position
```

---

## 11. Accessibility Features

### 11.1 One-Handed Mode
```
Configuration:
├── Move joystick zone to center
├── Camera auto-follows movement direction
├── Ability shortcuts on screen edges
└── Larger touch targets throughout
```

### 11.2 Assistive Options
| Feature | Description |
|---------|-------------|
| 8-Direction Lock | Snap to cardinal/diagonal directions |
| Auto-Run | Tap to toggle running toward camera direction |
| Sensitivity Scaling | Extra low to extra high range |
| Visual Boundary | Always show movement zone outline |

### 11.3 Motor Accessibility
```
Options:
├── Sticky Joystick: Movement continues without holding
├── Tap-to-Move: Tap destination to auto-walk
├── Reduced Precision Mode: Larger dead zone, less sensitivity
└── Switch Control Support: External accessibility devices
```

---

## 12. Performance Optimization

### 12.1 Rendering
```
Optimization Techniques:
├── Joystick on separate render layer
├── Pre-rendered at multiple sizes
├── GPU-accelerated alpha blending
├── Batched with other UI elements
└── Skip rendering when not visible (floating mode)
```

### 12.2 Memory Footprint
| Asset | Memory |
|-------|--------|
| Joystick textures (all sizes) | ~512 KB |
| Animation sprites | ~256 KB |
| Direction indicator | ~64 KB |
| Total | ~832 KB |

### 12.3 CPU Usage
```
Target CPU Budget:
├── Touch processing: < 0.1ms/frame
├── Movement calculation: < 0.2ms/frame
├── Visual update: < 0.1ms/frame
└── Total: < 0.5ms/frame
```

---

## 13. Testing Requirements

### 13.1 Device Testing Matrix
| Device Category | Test Devices |
|-----------------|--------------|
| Small Phone | iPhone SE, Galaxy A series |
| Standard Phone | iPhone 14, Pixel 7 |
| Large Phone | iPhone 14 Pro Max, S23 Ultra |
| Tablet | iPad, Galaxy Tab |

### 13.2 Test Cases
```
Functional Tests:
├── Joystick appears on touch
├── Movement in all directions
├── Dead zone prevents drift
├── Sprint activation (all methods)
├── Settings changes apply correctly
├── Multi-touch doesn't interfere
└── Touch release stops movement

Performance Tests:
├── Input latency < 16ms
├── 60 FPS maintained during movement
├── No visual stuttering
├── Memory usage within budget
└── Battery impact acceptable

Usability Tests:
├── New players can move within 5 seconds
├── No accidental movements reported
├── Sprint feels natural
├── Settings meet player preferences
└── Works with various thumb sizes
```

### 13.3 Edge Cases
```
Test Scenarios:
├── Touch at exact zone boundary
├── Very fast swipes across screen
├── Touch during frame drop
├── Touch during scene transition
├── Low battery / thermal throttle
├── Split-screen / PiP mode
└── Accessibility services active
```

---

## 14. Integration Points

### 14.1 Systems Integration
| System | Integration |
|--------|-------------|
| Character Controller | Receives direction + magnitude |
| Animation System | Receives movement state |
| Camera System | Shares touch event data |
| Network | Sends movement input to server |
| Analytics | Logs control preferences |

### 14.2 Event Callbacks
```csharp
// Movement events for other systems
public event Action<Vector2> OnMovementInput;      // Direction + magnitude
public event Action OnMovementStart;
public event Action OnMovementStop;
public event Action OnSprintStart;
public event Action OnSprintEnd;
```

---

## 15. Future Considerations

### 15.1 Potential Enhancements
- Haptic feedback for different terrain
- Contextual speed limits (e.g., in water)
- Vehicle/mount control adaptation
- Replay system recording inputs
- Custom joystick skins (cosmetic)

### 15.2 Platform-Specific
- iOS: Support for ProMotion (120Hz)
- Android: Support for high refresh rate displays
- Cross-platform: Unified feel across devices

---

## Appendix A: Default Configuration

```json
{
  "joystick": {
    "type": "floating",
    "size": "medium",
    "deadZone": 10,
    "opacity": 0.6,
    "fixedPosition": {
      "x": 0.15,
      "y": 0.75
    }
  },
  "movement": {
    "accelerationMultiplier": 1.0,
    "decelerationMultiplier": 1.0,
    "turnSmoothing": 0.5,
    "maxWalkSpeed": 250,
    "maxRunSpeed": 400,
    "sprintSpeed": 500
  },
  "sprint": {
    "mode": "doubleTap",
    "doubleTapWindow": 300,
    "autoSprintDelay": 1000
  },
  "accessibility": {
    "eightDirectionLock": false,
    "stickyJoystick": false,
    "showZoneBoundary": false
  }
}
```

---

## Appendix B: Visual Reference

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   ┌─────────────┐                                       │
│   │  Movement   │         Camera/Action Zone            │
│   │    Zone     │            (60% width)                │
│   │  (40% w)    │                                       │
│   │             │                                       │
│   │    ┌───┐    │                                       │
│   │    │ ● │    │                          [Abilities]  │
│   │    └───┘    │                                       │
│   │  Joystick   │                                       │
│   └─────────────┘                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘

Joystick Detail:
    ╭───────────╮
   ╱             ╲
  │   ╭─────╮     │    Outer Ring: Touch boundary
  │   │  ●  │     │    Inner Circle: Stick position
  │   ╰─────╯     │    ● Center point
   ╲             ╱
    ╰───────────╯
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
