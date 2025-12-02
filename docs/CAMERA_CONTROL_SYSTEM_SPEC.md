# UX-002: Camera Control System Specification

## Overview

This document specifies the touch-based camera control system for Plunderstorm Mobile, enabling intuitive third-person camera manipulation through touch gestures and optional gyroscope controls.

## Design Philosophy

The camera system must feel natural and responsive while preventing motion sickness. Players need precise control for combat while maintaining situational awareness. The system supports multiple input methods to accommodate different play styles and accessibility needs.

---

## 1. Touch Zone Configuration

### 1.1 Camera Control Zone
| Parameter | Value |
|-----------|-------|
| Zone Location | Right 60% of screen width |
| Zone Height | Full screen height |
| Buffer Zone | 20pt overlap prevention with movement zone |
| Touch Priority | Standard (abilities take priority) |

### 1.2 Zone Behavior
- Touch anywhere in zone begins camera rotation
- Multiple touches in zone: first touch controls camera
- Zone shares space with ability buttons (buttons take priority)
- Touch on empty space = camera control

---

## 2. Camera Movement Types

### 2.1 Horizontal Rotation (Yaw)
```
Input: Horizontal swipe/drag
Output: Camera orbits around player on Y-axis

Parameters:
├── Sensitivity: Configurable (1-10 scale)
├── Default Sensitivity: 5 (medium)
├── Degrees per 100pt drag: 15° (at sensitivity 5)
├── Direction: Natural (swipe left = look left)
├── Invert Option: Available in settings
└── Full 360° rotation: Unlimited
```

### 2.2 Vertical Rotation (Pitch)
```
Input: Vertical swipe/drag
Output: Camera tilts up/down

Parameters:
├── Sensitivity: Configurable (1-10 scale, can differ from horizontal)
├── Default Sensitivity: 4 (slightly lower than horizontal)
├── Degrees per 100pt drag: 10° (at sensitivity 4)
├── Minimum Pitch: -60° (looking up)
├── Maximum Pitch: 75° (looking down)
├── Invert Option: Available in settings
└── Soft limits: Eases into limit rather than hard stop
```

### 2.3 Combined Movement
```
Diagonal Drag Handling:
├── Both axes processed simultaneously
├── Independent sensitivity per axis
├── No axis snapping (free movement)
└── Smooth interpolation between frames
```

---

## 3. Sensitivity System

### 3.1 Sensitivity Scale
| Level | Multiplier | Degrees/100pt (H) | Degrees/100pt (V) | Description |
|-------|------------|-------------------|-------------------|-------------|
| 1 | 0.4x | 6° | 4° | Very Slow |
| 2 | 0.55x | 8.25° | 5.5° | Slow |
| 3 | 0.7x | 10.5° | 7° | Low |
| 4 | 0.85x | 12.75° | 8.5° | Medium-Low |
| 5 | 1.0x | 15° | 10° | Medium (Default) |
| 6 | 1.2x | 18° | 12° | Medium-High |
| 7 | 1.45x | 21.75° | 14.5° | High |
| 8 | 1.75x | 26.25° | 17.5° | Very High |
| 9 | 2.1x | 31.5° | 21° | Ultra |
| 10 | 2.5x | 37.5° | 25° | Maximum |

### 3.2 Sensitivity Curves
```
Linear Mode (Default):
rotation = drag_distance × sensitivity_multiplier × base_rate

Acceleration Mode (Optional):
rotation = drag_distance × (1 + (drag_speed / 1000)) × sensitivity_multiplier × base_rate

Purpose: Faster swipes = more rotation (for quick 180s)
```

---

## 4. Camera Smoothing

### 4.1 Smoothing Options
| Setting | Value Range | Default | Description |
|---------|-------------|---------|-------------|
| Smoothing Enabled | On/Off | On | Master toggle |
| Smoothing Amount | 0-100% | 40% | Lerp factor |
| Follow Delay | 0-200ms | 50ms | Input to camera delay |

### 4.2 Smoothing Algorithm
```csharp
// Per-frame camera update
void UpdateCamera()
{
    Vector2 targetRotation = GetTargetFromInput();

    if (smoothingEnabled)
    {
        float lerpFactor = 1.0f - (smoothingAmount * 0.01f);
        currentRotation = Vector2.Lerp(currentRotation, targetRotation, lerpFactor);
    }
    else
    {
        currentRotation = targetRotation;
    }

    ApplyRotation(currentRotation);
}
```

### 4.3 Smoothing Presets
| Preset | Amount | Feel |
|--------|--------|------|
| Snappy | 0% | Instant response, competitive |
| Balanced | 40% | Smooth but responsive |
| Cinematic | 70% | Very smooth, casual |
| Custom | 0-100% | User-defined |

---

## 5. Camera Collision

### 5.1 Collision Detection
```
Raycast System:
├── Ray from player to camera position
├── Collision layers: Terrain, Buildings, Props
├── Ignore layers: Players, Projectiles, VFX
└── Update frequency: Every frame
```

### 5.2 Collision Response
```
On Collision Detected:
1. Calculate closest valid camera position
2. Smoothly move camera to valid position (0.1s)
3. Reduce camera distance if needed
4. Maintain minimum distance from player (1.5m)

On Collision Cleared:
1. Gradually restore preferred camera distance
2. Lerp over 0.3s to prevent pop
3. Maintain target rotation throughout
```

### 5.3 Collision Parameters
| Parameter | Value |
|-----------|-------|
| Default Camera Distance | 8m |
| Minimum Camera Distance | 1.5m |
| Maximum Camera Distance | 12m |
| Collision Sphere Radius | 0.3m |
| Recovery Speed | 3m/s |

---

## 6. Quick Turn Feature

### 6.1 180-Degree Turn
```
Activation: Double-tap on right side of screen
Cooldown: 0.5s between quick turns
Animation Duration: 0.25s
Rotation: Camera rotates 180° around player

Detection Parameters:
├── Double-tap window: 300ms
├── Tap distance limit: 30pt (taps must be close together)
├── Minimum tap time: 50ms (prevents accidental triggers)
└── Haptic feedback: Medium impact on activation
```

### 6.2 Quick Turn Animation
```
Rotation Curve: Ease-in-out
├── 0.00s: Start at current rotation
├── 0.05s: Begin accelerating
├── 0.125s: Maximum speed (midpoint)
├── 0.20s: Begin decelerating
└── 0.25s: Complete at 180° offset

During Turn:
├── Player input paused
├── Visual: Slight motion blur (optional)
├── Audio: Swoosh sound effect
└── Cancel: Touch and hold cancels mid-turn
```

---

## 7. Auto-Center Feature

### 7.1 Auto-Center Behind Player
```
Trigger Conditions:
├── Player moving forward for 2+ seconds
├── No camera input for 2+ seconds
├── Player velocity above walk threshold
└── Not in combat (no recent damage, 5s)

Behavior:
├── Smoothly rotate camera behind player
├── Duration: 1.5s
├── Curve: Ease-in-out
├── Cancel: Any camera input cancels
└── Optional: Can disable in settings
```

### 7.2 Combat Camera Lock
```
During Combat:
├── Auto-center disabled
├── Camera maintains player-controlled rotation
├── Combat state: 5s after last damage dealt/received
└── Exception: Death cam uses auto-center
```

### 7.3 Settings
| Setting | Default |
|---------|---------|
| Auto-Center Enabled | On |
| Auto-Center Delay | 2.0s |
| Auto-Center Speed | 1.5s |
| Auto-Center in Combat | Off |

---

## 8. Gyroscope/Accelerometer Control

### 8.1 Gyroscope Aiming
```
Purpose: Fine-tune aim using device motion
Activation: Toggle in settings or quick toggle button

Input Processing:
├── Read gyroscope rotation rates
├── Apply sensitivity multiplier
├── Convert to camera rotation delta
└── Combine with touch input (additive)
```

### 8.2 Gyroscope Parameters
| Parameter | Range | Default |
|-----------|-------|---------|
| Gyro Enabled | On/Off | Off |
| Gyro Sensitivity | 1-10 | 5 |
| Horizontal Axis | On/Off | On |
| Vertical Axis | On/Off | On |
| Invert Horizontal | On/Off | Off |
| Invert Vertical | On/Off | Off |

### 8.3 Calibration System
```
Calibration Process:
1. User holds device in comfortable position
2. Press "Calibrate" button
3. System records current orientation as neutral
4. All gyro input relative to calibrated position

Auto-Calibration:
├── On game start
├── On respawn
├── When exiting menus
└── Optional: continuous drift correction
```

### 8.4 Gyro + Touch Integration
```
Combined Input Mode:
├── Touch: Large camera movements, quick turns
├── Gyro: Fine aiming adjustments
├── Both active simultaneously
└── Touch takes priority for large movements

Sensitivity Relationship:
├── Gyro typically lower sensitivity than touch
├── Gyro for precision (small adjustments)
├── Touch for speed (large rotations)
└── User can customize ratio
```

---

## 9. Camera Modes

### 9.1 Standard Third-Person
```
Parameters:
├── Distance: 8m from player
├── Height Offset: 2m above player
├── Look-at Offset: 0.5m above player center
├── Field of View: 60°
└── Default mode for gameplay
```

### 9.2 Combat Camera (When Aiming Abilities)
```
Parameters:
├── Distance: 6m from player (closer)
├── Height Offset: 1.5m above player
├── Over-Shoulder Offset: 0.5m right
├── Field of View: 55° (slightly zoomed)
└── Activates when entering targeting mode
```

### 9.3 Sprint Camera
```
Parameters:
├── Distance: 9m from player (farther)
├── Height Offset: 2.5m above player
├── Field of View: 65° (wider)
└── Smoothly transitions when sprinting
```

### 9.4 Camera Transitions
```
Transition Timing:
├── Standard ↔ Combat: 0.3s
├── Standard ↔ Sprint: 0.5s
├── All transitions: Ease-in-out curve
└── Interruptible by new state
```

---

## 10. Visual Feedback

### 10.1 Touch Indicator (Optional)
```
When Enabled:
├── Small dot shows touch position
├── Fades after 0.5s of no movement
├── Color: White with 50% opacity
└── Size: 20pt diameter
```

### 10.2 Gyro Indicator
```
When Gyro Active:
├── Small gyroscope icon in corner
├── Glows when device is tilted
├── Color indicates calibration status
└── Green: Calibrated, Red: Needs calibration
```

### 10.3 Quick Turn Indicator
```
During Quick Turn:
├── Brief motion blur effect (optional)
├── Directional arrows indicate turn direction
├── Duration: Matches turn animation
└── Can be disabled for performance
```

---

## 11. Input Processing

### 11.1 Frame Timing
| Metric | Target | Maximum |
|--------|--------|---------|
| Input Polling | Every frame | 16.67ms |
| Touch-to-Camera Latency | < 16ms | 33ms |
| Gyro Polling | 100Hz | 10ms |
| Visual Update | Every frame | 16.67ms |

### 11.2 Touch Processing Pipeline
```
Frame N:
1. Poll all active touches
2. Filter touches in camera zone
3. Calculate drag delta from last frame
4. Apply sensitivity multiplier
5. Apply axis inversion if enabled
6. Apply smoothing
7. Combine with gyro input
8. Apply collision detection
9. Update camera transform
10. Update visual feedback

Budget: < 1ms per frame
```

### 11.3 Input Prediction
```
For Smooth Response:
├── Extrapolate touch position by 1 frame
├── Predict gyro rotation for next frame
├── Correct on actual input arrival
└── Max prediction: 16ms (1 frame at 60 FPS)
```

---

## 12. Conflict Resolution

### 12.1 Touch Priority
```
Priority Order (Highest to Lowest):
1. UI buttons and menus
2. Ability buttons (when in zone)
3. Ability targeting (drag from button)
4. Camera control (default for zone)
5. Gesture recognition (pinch, etc.)
```

### 12.2 Multi-Touch Handling
```
Rules:
├── First touch in zone = camera control
├── Second touch: If on ability button → ability
├── Second touch: If empty space → ignored for camera
├── Touch transfer: Not supported (release required)
└── Three+ touches: Advanced gestures or ignored
```

### 12.3 Ability Targeting Interaction
```
During Ability Targeting:
├── Camera control temporarily disabled
├── Targeting uses same touch zone
├── Camera frozen at current position
├── Resume camera control on ability release
└── Exception: Gyro still active during targeting
```

---

## 13. Customization Options

### 13.1 Settings Menu Structure
```
Camera Settings:
├── General
│   ├── Horizontal Sensitivity: [████████░░] 8
│   ├── Vertical Sensitivity: [██████░░░░] 6
│   ├── Invert Horizontal: [Off]
│   ├── Invert Vertical: [Off]
│   └── Camera Smoothing: [Balanced ▼]
│
├── Advanced
│   ├── Auto-Center: [On]
│   ├── Auto-Center Delay: [2.0s ▼]
│   ├── Quick Turn: [On]
│   └── Camera Distance: [████████░░] 8m
│
├── Gyroscope
│   ├── Enable Gyro: [Off]
│   ├── Gyro Sensitivity: [█████░░░░░] 5
│   ├── Calibrate Gyro: [Calibrate]
│   └── Gyro Axes: [Both ▼]
│
└── [Reset to Defaults]
```

### 13.2 Quick Access
```
In-Game Quick Settings:
├── Sensitivity adjustment (hold settings icon)
├── Gyro toggle (shake device or button)
├── Quick turn enable/disable
└── Accessible without leaving match
```

---

## 14. Accessibility Features

### 14.1 Reduced Motion
```
When Enabled:
├── Disable motion blur effects
├── Slower camera transitions
├── No camera shake
├── Limit rotation speed
└── Disable auto-center
```

### 14.2 One-Handed Mode Integration
```
When Active:
├── Camera auto-follows movement direction
├── Reduced camera control zone (ability buttons expand)
├── Auto-center more aggressive
├── Gyro recommended for fine control
└── Quick turn via screen edge
```

### 14.3 Large Touch Targets
```
For Accessibility:
├── Increase touch zone margins
├── Larger gesture detection areas
├── Slower double-tap window
└── Configurable in accessibility settings
```

---

## 15. Performance Optimization

### 15.1 CPU Budget
```
Camera System Budget:
├── Touch processing: < 0.2ms
├── Gyro processing: < 0.1ms
├── Collision detection: < 0.3ms
├── Transform update: < 0.1ms
├── Visual effects: < 0.2ms
└── Total: < 1ms per frame
```

### 15.2 Memory Usage
| Component | Memory |
|-----------|--------|
| Camera state | ~256 bytes |
| Touch history | ~1 KB |
| Gyro buffer | ~512 bytes |
| Collision cache | ~2 KB |
| Total | ~4 KB |

### 15.3 Battery Considerations
```
Gyroscope Impact:
├── Gyro sensor uses additional power
├── Option to auto-disable after timeout
├── Battery saver mode disables gyro
└── User informed of battery impact
```

---

## 16. Testing Requirements

### 16.1 Functional Tests
```
Core Tests:
├── Camera rotates with touch drag
├── Pitch limits enforced
├── Sensitivity scales correctly
├── Invert options work
├── Smoothing affects response
├── Collision prevents clipping
├── Quick turn completes correctly
├── Auto-center triggers appropriately
├── Gyro input affects camera
├── Gyro calibration works
└── All settings persist
```

### 16.2 Device Testing
| Device Type | Specific Tests |
|-------------|----------------|
| Low-end phone | Gyro availability, performance |
| High-end phone | High refresh rate support |
| Tablet | Large screen sensitivity scaling |
| All devices | Collision in all levels |

### 16.3 Edge Cases
```
Test Scenarios:
├── Touch at zone boundary
├── Very fast swipes
├── Gyro during screen rotation
├── Camera collision in tight spaces
├── Quick turn near walls
├── Auto-center during combat
├── Settings change mid-match
└── Orientation change handling
```

---

## 17. Integration Points

### 17.1 System Integration
| System | Integration |
|--------|-------------|
| Movement System | Receives camera forward for direction |
| Ability System | Receives camera rotation for targeting |
| UI System | Provides camera transform for 3D UI |
| Network | Sends look direction to server |
| Analytics | Logs sensitivity preferences |

### 17.2 Events
```csharp
// Camera events for other systems
public event Action<Quaternion> OnCameraRotationChanged;
public event Action OnQuickTurnStarted;
public event Action OnQuickTurnCompleted;
public event Action OnAutoCenterStarted;
public event Action<bool> OnGyroToggled;
```

---

## 18. Default Configuration

```json
{
  "camera": {
    "horizontalSensitivity": 5,
    "verticalSensitivity": 4,
    "invertHorizontal": false,
    "invertVertical": false,
    "smoothingPreset": "balanced",
    "smoothingAmount": 40,
    "distance": 8.0,
    "minPitch": -60,
    "maxPitch": 75
  },
  "autoCenter": {
    "enabled": true,
    "delay": 2.0,
    "duration": 1.5,
    "enabledInCombat": false
  },
  "quickTurn": {
    "enabled": true,
    "duration": 0.25,
    "doubleTapWindow": 300
  },
  "gyroscope": {
    "enabled": false,
    "sensitivity": 5,
    "horizontalAxis": true,
    "verticalAxis": true,
    "invertHorizontal": false,
    "invertVertical": false
  },
  "collision": {
    "enabled": true,
    "minDistance": 1.5,
    "maxDistance": 12.0,
    "recoverySpeed": 3.0
  }
}
```

---

## Appendix A: Visual Reference

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Movement Zone    │         Camera Control Zone        │
│   (40% width)      │           (60% width)              │
│                    │                                    │
│                    │    ┌─────────────────────────┐     │
│   ┌───┐            │    │                         │     │
│   │ ○ │            │    │   Touch & Drag Area     │     │
│   └───┘            │    │   for Camera Control    │     │
│  Joystick          │    │                         │     │
│                    │    │   Double-tap: 180° turn │     │
│                    │    └─────────────────────────┘     │
│                    │                     [Abilities]    │
└─────────────────────────────────────────────────────────┘

Camera Movement:
                    ↑ (Pitch up, limited to -60°)
                    │
        ←───────────┼───────────→ (Yaw, unlimited 360°)
                    │
                    ↓ (Pitch down, limited to 75°)
```

---

## Appendix B: Motion Sickness Prevention

```
Design Considerations:
├── Gradual transitions (no instant cuts)
├── Stable horizon line when possible
├── Limit camera shake intensity
├── Provide reduced motion options
├── FOV changes are subtle
├── No forced camera movements during action
└── Player always has camera control
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
