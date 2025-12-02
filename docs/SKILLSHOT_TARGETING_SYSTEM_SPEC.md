# UX-004: Skillshot Targeting System Specification

## Overview

This document specifies the skillshot targeting system for Plunderstorm Mobile, enabling intuitive aiming for abilities that require directional or positional input. The system supports multiple targeting methods to accommodate different play styles and skill levels.

## Design Philosophy

Skillshot targeting must feel intuitive on touch screens while providing the precision needed for competitive play. The system balances accessibility (making abilities easy to aim) with skill expression (rewarding accurate targeting). Visual feedback is critical for understanding ability trajectories and ranges.

---

## 1. Targeting Method Types

### 1.1 Drag-and-Release (Default)
```
Interaction Flow:
1. Press and hold ability button
2. Targeting UI appears
3. Drag finger to aim direction
4. Targeting indicator follows finger
5. Release finger to fire ability
6. (Optional) Drag back to button to cancel

Use Cases:
├── Primary method for most players
├── Intuitive for touch screens
├── Works well for all skillshot types
└── Natural transition from button press
```

### 1.2 Tap-Tap Mode
```
Interaction Flow:
1. Tap ability button once
2. Targeting UI appears, locked on
3. Move finger anywhere on screen to aim
4. Tap again to confirm and fire
5. (Or tap ability button to cancel)

Use Cases:
├── Players who prefer deliberate aiming
├── Complex abilities requiring precision
├── Situations where drag is awkward
└── Optional per-ability or global setting
```

### 1.3 Quick Cast Mode
```
Interaction Flow:
1. Tap ability button
2. Ability fires immediately in facing direction
3. No targeting phase
4. Relies on aim assist for accuracy

Use Cases:
├── Expert players seeking speed
├── Simple directional abilities
├── High-pressure combat situations
└── Optional, requires manual enable
```

### 1.4 Per-Ability Configuration
| Setting | Options | Scope |
|---------|---------|-------|
| Global Method | Drag/Tap/Quick | All abilities |
| Per-Ability Override | Drag/Tap/Quick/Default | Each ability |
| Quick Cast Toggle | On/Off | Each ability |

---

## 2. Ability Targeting Types

### 2.1 Directional (Line)
```
Description: Fires in a direction from player
Examples: Fireball, Basic Attack

Visual Indicator:
├── Line from player in aim direction
├── Length = ability range
├── Width = projectile width
├── Color = ability type color
└── Arrow head at end
```

### 2.2 Ground Target (Circle)
```
Description: Targets a location on the ground
Examples: Lightning Strike, Smoke Bomb

Visual Indicator:
├── Circle at target location
├── Radius = effect area
├── Dashed outline at max range
├── Fill = semi-transparent ability color
└── Center dot for precision
```

### 2.3 Arc/Projectile Path
```
Description: Arcing projectiles with gravity
Examples: Cannonball, Poison Mackerel (thrown)

Visual Indicator:
├── Curved line showing arc path
├── Landing zone circle
├── Height indicator (optional)
├── Dotted line = predicted bounce path
└── Arc changes with drag distance
```

### 2.4 Cone/Area
```
Description: Affects area in front of player
Examples: Cutlass Slash, Wind Burst

Visual Indicator:
├── Cone shape from player
├── Angle = ability cone width
├── Length = ability range
├── Filled with ability color
└── Edge lines clearly visible
```

### 2.5 Self-Cast
```
Description: Abilities centered on player
Examples: Barrel Roll, Healing Grog

Visual Indicator:
├── Circle around player (for area effects)
├── Direction arrow (for movement abilities)
├── Brief flash on activation
└── Minimal targeting needed
```

---

## 3. Targeting Indicator Design

### 3.1 Visual Elements
```
Common Elements:
├── Direction Line/Arrow
│   ├── Width: 4pt (scales with zoom)
│   ├── Color: Ability type color
│   ├── Style: Solid with gradient fade
│   └── End Cap: Arrow or crosshair
│
├── Range Circle
│   ├── Stroke: 2pt dashed line
│   ├── Color: White with 50% opacity
│   ├── Interior: Subtle gradient fill
│   └── Update: Real-time as player moves
│
├── Target Reticle
│   ├── Crosshair or circle at aim point
│   ├── Size: 20pt diameter
│   ├── Pulse animation when valid target
│   └── Color change: valid (green) / invalid (red)
│
└── Trajectory Arc
    ├── Bezier curve for arcing abilities
    ├── Dotted for predicted path
    ├── Landing zone highlighted
    └── Height visualization (side view optional)
```

### 3.2 Color Scheme
| State | Primary Color | Accent Color |
|-------|---------------|--------------|
| Offensive Normal | #FF4444 (Red) | #FF8888 |
| Offensive Valid Target | #44FF44 (Green) | #88FF88 |
| Utility Normal | #4488FF (Blue) | #88AAFF |
| Out of Range | #888888 (Gray) | #AAAAAA |
| Blocked/Invalid | #FF0000 (Bright Red) | #FF4444 |

### 3.3 Indicator Scaling
```
Screen Size Adaptation:
├── Line thickness: 4pt base, scales with DPI
├── Reticle size: 20pt base, scales with preference
├── Text size: 12pt minimum for readability
├── All elements respect safe areas
└── Tablet mode: Larger indicators optional
```

---

## 4. Range Visualization

### 4.1 Max Range Indicator
```
Display:
├── Dashed circle at maximum ability range
├── Updates position as player moves
├── Always visible during targeting
├── Color: White/light with low opacity

Behavior:
├── Targets beyond range show red reticle
├── Drag stops at range limit (optional)
├── Audio cue when reaching max range
└── Visual pulse at boundary
```

### 4.2 Effective Range
```
For Abilities with Falloff:
├── Inner circle: Full damage range
├── Outer circle: Maximum range
├── Gradient between them
├── Damage number preview (optional)
└── Example: Fireball explosion falloff
```

### 4.3 Minimum Range
```
For Abilities with Min Range:
├── Inner circle: Cannot target inside
├── Darkened/X'd out area
├── Example: Cannonball needs arc distance
└── Audio/haptic warning if targeting too close
```

---

## 5. Target Highlighting

### 5.1 Valid Target Indicators
```
Enemy in Range:
├── Red outline around enemy
├── Health bar appears/brightens
├── Name tag visible
├── Distance indicator (optional)
└── Priority: Closest to reticle center

Friendly Indicators:
├── For abilities that can target allies
├── Green outline
├── Clear name tag
└── Prevents friendly fire accidents
```

### 5.2 Target Priority System
```
Auto-Target Priority:
1. Enemy closest to reticle center
2. Enemy closest to player (if tie)
3. Lowest health enemy (optional setting)
4. Most recent attacker (optional setting)

Target Lock:
├── Brief lock when target highlighted
├── Prevents jittering between targets
├── Duration: 200ms
└── Override: Significant reticle movement
```

### 5.3 Obstructed Target Feedback
```
When Target Behind Obstacle:
├── Dotted line to target (blocked)
├── Red X on obstacle
├── "Blocked" text indicator
├── Different audio tone
└── Reticle stays but dimmed
```

---

## 6. Sensitivity and Precision

### 6.1 Aim Sensitivity
| Level | Description | Degrees per 100pt |
|-------|-------------|-------------------|
| 1 | Very Slow | 15° |
| 2 | Slow | 22.5° |
| 3 | Low | 30° |
| 4 | Medium-Low | 37.5° |
| 5 | Medium (Default) | 45° |
| 6 | Medium-High | 52.5° |
| 7 | High | 60° |
| 8 | Very High | 75° |
| 9 | Ultra | 90° |
| 10 | Maximum | 120° |

### 6.2 Precision Mode
```
Activation: Move finger slowly or hold still
Effect: Reduces sensitivity for fine-tuning

Parameters:
├── Trigger: Finger speed < 50pt/s
├── Precision multiplier: 0.5x
├── Transition: Smooth over 0.1s
├── Indicator: Reticle shrinks slightly
└── Audio: Subtle precision mode sound
```

### 6.3 Sticky Targeting
```
Near Valid Target:
├── Aim slightly "sticks" to enemy
├── Helps with small movements
├── Can be disabled in settings
├── Strength: Configurable (Off/Low/Medium/High)

Implementation:
├── When reticle within 30pt of enemy center
├── Apply 30% pull toward enemy
├── Smooth interpolation
└── Release when moving away
```

---

## 7. Movement During Targeting

### 7.1 Move While Aiming
```
Simultaneous Control:
├── Left thumb: Movement joystick (normal)
├── Right thumb: Aiming (targeting)
├── Both can operate independently
└── Essential for skilled play

Speed Modifier:
├── Optional: Slow movement while targeting
├── Default: Full speed allowed
├── Setting: "Slow when aiming" (On/Off/Auto)
└── Auto: Slows for precision abilities only
```

### 7.2 Joystick Aiming Alternative
```
Optional Mode:
├── Right joystick appears for aiming
├── Similar to twin-stick controls
├── Releases fires ability
└── For players preferring joystick aim

Configuration:
├── Enable: Settings toggle
├── Joystick position: Configurable
├── Sensitivity: Independent setting
└── Can coexist with touch aiming
```

---

## 8. Cancel Mechanisms

### 8.1 Cancel Methods
| Method | How | Priority |
|--------|-----|----------|
| Drag Back | Drag finger to button center | Primary |
| Cancel Zone | Lift in zone around button | Secondary |
| Tap Outside | Tap empty screen area | Optional |
| Different Ability | Tap another ability button | Switches |
| Movement | Any movement (optional) | Disabled default |

### 8.2 Cancel Zone Visualization
```
When Targeting:
├── Ring appears around ability button
├── Radius: 20pt beyond button edge
├── Color: Red with low opacity
├── Text: "Cancel" or X icon
├── Haptic: Light pulse when entering
└── Finger lift in zone = cancel
```

### 8.3 Cancel Feedback
```
On Cancel:
├── Visual: Button returns to normal
├── Audio: Negative/cancel sound
├── Haptic: Light double-tap
├── Targeting indicator disappears
└── Cooldown NOT triggered
```

---

## 9. Ability-Specific Targeting

### 9.1 Fireball
```
Type: Directional (Line)
Range: 25 units
Width: 1 unit (projectile), 3 units (explosion)

Targeting:
├── Line indicator from player
├── Explosion radius preview at end
├── Arc preview if hitting terrain
└── Valid targets highlighted
```

### 9.2 Lightning Strike
```
Type: Ground Target (Circle)
Range: 20 units
Radius: 4 units

Targeting:
├── Circle at target location
├── Range ring around player
├── Snap to valid ground (not water)
└── Shows enemies in AoE
```

### 9.3 Cannonball
```
Type: Arc/Projectile Path
Range: 15-30 units (varies with arc)
Splash Radius: 3 units

Targeting:
├── Arc trajectory line
├── Landing circle at end
├── Height preview (side indicator)
├── Minimum range indicator
└── Bounce prediction (off surfaces)
```

### 9.4 Grappling Hook
```
Type: Directional with Attach Point
Range: 20 units
Attach: Terrain, props, enemies

Targeting:
├── Line indicator
├── Valid attach points highlighted
├── Red if no valid target
├── Green on valid attachment
└── Pull path preview
```

### 9.5 Cutlass Slash
```
Type: Cone in Front
Range: 3 units
Angle: 120 degrees

Targeting:
├── Cone indicator (usually instant)
├── Quick cast recommended
├── Brief preview on hold
└── Enemies in cone highlighted
```

### 9.6 Smoke Bomb
```
Type: Ground Target (Self-centered option)
Range: 10 units
Radius: 5 units

Targeting:
├── Circle at target location
├── Can quick cast at feet
├── Range ring shows throw distance
└── Area of effect preview
```

---

## 10. Visual Feedback During Targeting

### 10.1 Enter Targeting Mode
```
Animation Sequence:
├── 0.0s: Button press registered
├── 0.0s: Button scales down (90%)
├── 0.05s: Range indicator appears (fade in)
├── 0.1s: Direction indicator appears
├── 0.1s: Other buttons dim (70% opacity)
└── 0.15s: Full targeting mode active
```

### 10.2 Aiming Updates
```
Per Frame:
├── Direction indicator follows finger
├── Range ring follows player movement
├── Target highlighting updates
├── Trajectory recalculates
└── Valid/invalid state updates
```

### 10.3 Exit Targeting (Fire)
```
Animation Sequence:
├── 0.0s: Finger released
├── 0.0s: Targeting indicator flashes
├── 0.05s: Ability fires (server confirms)
├── 0.1s: Targeting UI disappears
├── 0.1s: Button returns to normal
├── 0.1s: Cooldown begins
└── 0.15s: Other buttons restore opacity
```

---

## 11. Audio Feedback

### 11.1 Targeting Sounds
| Event | Sound | Volume |
|-------|-------|--------|
| Enter Targeting | Soft activation tone | Low |
| Aim Over Enemy | Target acquired ping | Medium |
| Out of Range | Warning tone | Low |
| Blocked Target | Error/blocked sound | Medium |
| Cancel | Negative/cancel | Low |
| Fire | Ability-specific cast | Full |

### 11.2 Spatial Audio
```
For Enemy Highlighting:
├── Directional audio cue
├── Points toward highlighted enemy
├── Helps with off-screen awareness
└── Can be disabled in settings
```

---

## 12. Performance Optimization

### 12.1 Rendering Budget
| Component | Budget |
|-----------|--------|
| Targeting indicators | < 0.3ms |
| Range calculations | < 0.1ms |
| Target detection | < 0.2ms |
| Path prediction | < 0.2ms |
| Total | < 1ms/frame |

### 12.2 Update Frequencies
```
High Priority (Every Frame):
├── Direction indicator position
├── Finger position tracking
└── Button state

Medium Priority (Every 2-3 Frames):
├── Target highlighting
├── Range validation
└── Path prediction

Low Priority (Every 5 Frames):
├── Target list refresh
├── Obstacle detection
└── Audio state
```

### 12.3 Optimization Techniques
```
Performance Features:
├── Object pooling for indicators
├── Cached range calculations
├── Spatial partitioning for targets
├── LOD for targeting visuals
└── Disable during low performance
```

---

## 13. Settings and Customization

### 13.1 Targeting Settings Menu
```
Targeting Settings:
├── Default Method
│   └── [Drag-and-Release ▼]
│
├── Per-Ability Settings
│   ├── Fireball: [Default ▼]
│   ├── Lightning Strike: [Tap-Tap ▼]
│   └── ... (each ability)
│
├── Quick Cast
│   ├── Enable Quick Cast: [Off]
│   └── Quick Cast Indicator: [On]
│
├── Sensitivity
│   ├── Aim Sensitivity: [████████░░] 8
│   └── Precision Mode: [On]
│
├── Assistance
│   ├── Target Highlighting: [On]
│   ├── Sticky Targeting: [Medium ▼]
│   └── Range Indicator: [On]
│
└── [Reset to Defaults]
```

### 13.2 Accessibility Options
| Option | Description | Default |
|--------|-------------|---------|
| Large Targeting Indicators | 150% size | Off |
| High Contrast Colors | Saturated colors | Off |
| Extended Targeting Time | No timeout | Off |
| Simplified Indicators | Basic shapes only | Off |
| Audio Target Assist | Sound cues for targets | Off |

---

## 14. Integration Points

### 14.1 System Dependencies
| System | Integration |
|--------|-------------|
| Ability System | Receives targeting data |
| Input Manager | Provides touch coordinates |
| Camera System | Screen-to-world conversion |
| AI/Bot System | Target position for bots |
| Network | Sends aim direction to server |

### 14.2 Events
```csharp
// Targeting events for other systems
public event Action<int> OnTargetingStarted;        // Ability slot
public event Action<int, Vector3> OnTargetingUpdated; // Slot, aim point
public event Action<int, Vector3> OnTargetingConfirmed; // Slot, final aim
public event Action<int> OnTargetingCanceled;       // Ability slot
public event Action<Entity> OnTargetHighlighted;    // Highlighted entity
```

### 14.3 Network Synchronization
```
Client to Server:
├── Ability ID
├── Aim direction (Vector3)
├── Target position (for ground-target)
├── Target entity ID (if locked)
├── Client timestamp
└── Player position at fire time

Server Response:
├── Confirm ability cast
├── Adjusted position (if needed)
├── Hit results (for instant abilities)
└── Projectile spawn data
```

---

## 15. Testing Requirements

### 15.1 Functional Tests
```
Core Tests:
├── Each targeting method works
├── All ability types target correctly
├── Range limits enforced
├── Cancel works from all methods
├── Target highlighting accurate
├── Movement during aiming works
├── Quick cast fires correctly
├── Settings apply properly
└── Network sync accurate
```

### 15.2 Usability Tests
```
Metrics:
├── Time to target enemy (< 500ms average)
├── Targeting accuracy (> 85% hit intended target)
├── Cancel success rate (> 95%)
├── Accidental fire rate (< 2%)
├── New player comprehension (< 3 attempts)
└── Preference distribution across methods
```

### 15.3 Device Testing
| Device | Specific Tests |
|--------|----------------|
| Small Phone | Touch precision, indicator visibility |
| Large Phone | Thumb reach, scaling |
| Tablet | Two-thumb ergonomics |
| All | Performance under load |

---

## 16. Default Configuration

```json
{
  "targeting": {
    "defaultMethod": "dragAndRelease",
    "aimSensitivity": 5,
    "precisionMode": true,
    "precisionThreshold": 50,
    "precisionMultiplier": 0.5
  },
  "assistance": {
    "targetHighlighting": true,
    "stickyTargeting": "medium",
    "stickyStrength": 0.3,
    "stickyRange": 30,
    "rangeIndicator": true
  },
  "cancel": {
    "cancelZoneRadius": 20,
    "tapOutsideCancels": false,
    "movementCancels": false
  },
  "visuals": {
    "indicatorOpacity": 0.8,
    "lineThickness": 4,
    "reticleSize": 20,
    "showTrajectory": true
  },
  "perAbility": {
    "fireball": "default",
    "lightningStrike": "default",
    "cannonball": "default",
    "grappleHook": "default",
    "cutlassSlash": "quickCast",
    "smokeBomb": "default"
  }
}
```

---

## Appendix A: Visual Reference

```
Directional Targeting:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                        ○ Enemy                          │
│                       /                                 │
│                      /                                  │
│     ○ Player ──────/──────────→ Range limit             │
│                   Direction Indicator                   │
│                                                         │
│  ╭─────╮                                  ╭─────╮       │
│  │ Move│                                  │Aim  │       │
│  ╰─────╯                                  ╰─────╯       │
│                                                         │
└─────────────────────────────────────────────────────────┘

Ground Target:
┌─────────────────────────────────────────────────────────┐
│              ·····                                      │
│            ·       ·    ← Range Circle                  │
│           ·   ┌───┐  ·                                  │
│          ·    │ ● │   ·  ← Target Circle                │
│           ·   └───┘  ·                                  │
│            ·       ·                                    │
│     ○       ·····                                       │
│   Player                                                │
│                                                         │
└─────────────────────────────────────────────────────────┘

Arc Trajectory:
┌─────────────────────────────────────────────────────────┐
│                    . . .                                │
│                 .        .                              │
│              .             .                            │
│           .                  .  ← Arc Path              │
│        .                       ● Landing Zone           │
│     ○                                                   │
│   Player                                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Appendix B: State Machine

```
Targeting States:

                    ┌──────────────────────────────────┐
                    │                                  │
                    ▼                                  │
              ┌───────────┐                            │
              │   Idle    │                            │
              └───────────┘                            │
                    │                                  │
                    │ Press ability button             │
                    ▼                                  │
              ┌───────────┐                            │
              │ Targeting │                            │
              │  Active   │←────────────────┐          │
              └───────────┘                 │          │
                 │  │  │                    │          │
       ┌─────────┘  │  └─────────┐          │          │
       │            │            │          │          │
       │ Release    │ Cancel     │ Tap      │          │
       │ (valid)    │            │ other    │          │
       ▼            ▼            │ ability  │          │
  ┌─────────┐  ┌─────────┐       │          │          │
  │ Confirm │  │ Cancel  │       └──────────┘          │
  │  Fire   │  │         │                             │
  └─────────┘  └─────────┘                             │
       │            │                                  │
       │            └──────────────────────────────────┘
       │
       │ Cooldown complete
       │
       └───────────────────────────────────────────────┘
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
