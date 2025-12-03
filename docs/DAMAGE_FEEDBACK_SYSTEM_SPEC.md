# UX-011: Damage Feedback System Specification

## Overview

This document specifies the damage feedback system for Plunderstorm Mobile, providing clear visual, audio, and haptic feedback when players deal or receive damage. Effective feedback is critical for gameplay feel and player satisfaction.

## Design Philosophy

Damage feedback must be immediate, clear, and satisfying. Players need to instantly understand when they've hit an enemy, taken damage, and from which direction. The system balances informative feedback with avoiding screen clutter or distraction during intense combat.

---

## 1. Hit Confirmation (Dealing Damage)

### 1.1 Hit Marker
```
Position: Center of screen (crosshair area)
Appearance: X-shaped or cross markers

Standard Hit:
    \   /
     \ /
      X
     / \
    /   \

Color: White
Duration: 0.15 seconds
Size: 20pt

Critical Hit:
    \ | /
     \|/
    ──X──
     /|\
    / | \

Color: Yellow/Gold
Duration: 0.25 seconds
Size: 28pt
Animation: Scale up then fade
```

### 1.2 Hit Marker Variations
| Hit Type | Color | Size | Sound |
|----------|-------|------|-------|
| Normal | White (#FFFFFF) | 20pt | Light hit |
| Critical | Yellow (#FFD700) | 28pt | Critical ding |
| Headshot | Red (#FF4444) | 32pt | Headshot ping |
| Kill | Red + X | 36pt | Kill confirm |
| Shield | Blue (#4488FF) | 20pt | Shield impact |

### 1.3 Hit Marker Animation
```
Timeline:
├── 0.00s: Appear at full size
├── 0.05s: Scale to 120%
├── 0.10s: Return to 100%
├── 0.15s: Begin fade out
└── 0.20s: Fully invisible

Critical/Kill:
├── Additional rotation (15°)
├── Particle burst effect
├── Longer duration (0.3s)
└── Screen pulse (optional)
```

---

## 2. Damage Numbers

### 2.1 Floating Combat Text
```
Position: Above damaged target's head
Movement: Float upward and fade

Standard Damage:
   "47"
   └── White text, float up

Critical Damage:
   "94!"
   └── Yellow text, larger, exclamation

Healing:
   "+25"
   └── Green text, float up

Shield Damage:
   "(30)"
   └── Blue text with parentheses
```

### 2.2 Damage Number Properties
| Type | Color | Size | Duration | Movement |
|------|-------|------|----------|----------|
| Normal | White | 18pt | 1.0s | Up 30pt |
| Critical | Yellow | 24pt | 1.2s | Up 40pt |
| Ability | Orange | 20pt | 1.0s | Up 30pt |
| Heal | Green | 20pt | 1.0s | Up 30pt |
| Shield | Blue | 16pt | 0.8s | Up 25pt |
| DoT Tick | Red | 14pt | 0.6s | Up 20pt |

### 2.3 Number Stacking
```
Multiple Hits in Quick Succession:
├── Stack vertically (offset each number)
├── Max 5 visible at once
├── Older numbers push up
├── Combine rapid hits option (setting)

Combined Display:
"47 + 23 + 18 = 88"
└── Shows rapid combo damage
```

### 2.4 Settings
```
Damage Numbers Settings:
├── Enable: [On] / Off
├── Show My Damage: [On] / Off
├── Show Damage to Me: [On] / Off
├── Combine Rapid Hits: On / [Off]
├── Size: Small / [Medium] / Large
└── Duration: Short / [Normal] / Long
```

---

## 3. Receiving Damage Feedback

### 3.1 Screen Edge Vignette
```
On Taking Damage:
├── Red vignette at screen edges
├── Intensity based on damage amount
├── Direction indicates source
├── Fades over 0.5 seconds

Damage Levels:
├── Light (1-20 HP): 20% opacity vignette
├── Medium (21-50 HP): 50% opacity vignette
├── Heavy (51+ HP): 80% opacity vignette
└── Critical (>50% HP): Full flash + pulse
```

### 3.2 Directional Damage Indicator
```
Position: Screen edge, pointing toward damage source

Visual:
      ▼ (damage from front)

◀────┼────▶ (damage from sides)

      ▲ (damage from behind)

Properties:
├── Color: Red (#FF4444)
├── Opacity: Based on damage amount
├── Size: 40pt arrow/wedge
├── Duration: 2 seconds, fade out
└── Multiple indicators if multiple sources
```

### 3.3 Directional Indicator Accuracy
```
Calculation:
1. Get damage source world position
2. Calculate angle from player to source
3. Map angle to screen position
4. Display indicator at edge

Update: Fixed position (doesn't track if source moves)
Purpose: Show where damage came from, not current enemy position
```

---

## 4. Low Health Warning

### 4.1 Critical Health State
```
Trigger: Health below 25% (or 30 HP, whichever is lower)

Visual Effects:
├── Persistent red vignette (20% opacity)
├── Health bar pulses red
├── Screen slightly desaturated
└── Subtle pulse effect on edges

Audio Effects:
├── Heartbeat sound (rhythmic)
├── Reduced ambient audio
├── Warning tone (one-time)
└── Intensity increases as health drops
```

### 4.2 Near-Death State
```
Trigger: Health below 10%

Additional Effects:
├── Stronger vignette (40% opacity)
├── Faster heartbeat
├── Screen edges darken
├── "DANGER" text (optional)
└── Haptic pattern (if enabled)
```

### 4.3 Settings
```
Low Health Warning:
├── Enable Visual: [On] / Off
├── Enable Audio: [On] / Off
├── Warning Threshold: [25%] / 15% / 10%
├── Heartbeat Volume: ████░░░░ 50%
└── Vignette Intensity: Low / [Medium] / High
```

---

## 5. Kill Confirmation

### 5.1 Kill Visual Feedback
```
On Eliminating Enemy:

Hit Marker:
├── Large red X
├── Size: 36pt
├── Scale animation (pop)
├── Duration: 0.4 seconds

Screen Effect:
├── Subtle screen flash (optional)
├── Kill feed highlight
├── XP number popup
└── Particle burst at crosshair
```

### 5.2 Kill Audio
```
Sound Effects:
├── Distinct "kill confirmed" sound
├── Satisfying impact tone
├── Voice line (optional): "Enemy eliminated"
└── Different sound for melee vs ranged

Audio Properties:
├── Priority: High (don't overlap other sounds)
├── Volume: Louder than normal hit
├── No attenuation (always audible)
└── Spatial: None (UI sound)
```

### 5.3 Kill Streak Feedback
```
Multiple Kills:
├── 2 kills: "Double Kill" banner
├── 3 kills: "Triple Kill" banner
├── 4+ kills: "Rampage" banner
└── Each with unique sound

Banner Display:
├── Position: Top-center screen
├── Duration: 2 seconds
├── Animation: Slide in, hold, slide out
└── Color: Gold gradient
```

---

## 6. Special Damage Types

### 6.1 Fire/Burning Damage
```
Visual:
├── Orange damage numbers
├── Flame icon with tick damage
├── Screen edge flicker (orange)
└── Character model flames (3D)

Format: "🔥 12" per tick
Sound: Crackling fire loop
```

### 6.2 Poison Damage
```
Visual:
├── Green damage numbers
├── Poison drop icon
├── Screen edge tint (green)
└── Character model green particles

Format: "☠ 8" per tick
Sound: Bubbling/hissing
```

### 6.3 Storm Damage
```
Visual:
├── Purple damage numbers
├── Storm cloud icon
├── Screen edge storm effect
├── Increasing intensity

Format: "⚡ 15" per tick
Sound: Thunder rumble
Special: Directional indicator points to safe zone
```

---

## 7. Screen Shake

### 7.1 Screen Shake Parameters
| Event | Intensity | Duration | Frequency |
|-------|-----------|----------|-----------|
| Light hit received | 2pt | 0.1s | 30 Hz |
| Heavy hit received | 5pt | 0.15s | 25 Hz |
| Critical hit received | 8pt | 0.2s | 20 Hz |
| Explosion nearby | 10pt | 0.3s | 15 Hz |
| Landing (fall) | 3pt | 0.1s | 25 Hz |

### 7.2 Shake Implementation
```
Shake Algorithm:
├── Random offset within intensity bounds
├── Perlin noise for smooth randomness
├── Decay over duration
├── Additive (multiple sources stack)
└── Capped at maximum (15pt)

Camera Application:
├── Offset applied to camera transform
├── Does not affect gameplay (aim)
├── UI may or may not shake (setting)
└── Optional: Screen shake affects touch zones (not recommended)
```

### 7.3 Settings
```
Screen Shake:
├── Enable: [On] / Off
├── Intensity: [███████░░░] 70%
├── Include UI: [Off] / On
└── Maximum Shake: Low / [Medium] / High
```

---

## 8. Haptic Feedback

### 8.1 Haptic Events
| Event | Pattern | Intensity |
|-------|---------|-----------|
| Deal damage | Single tap | Light |
| Deal critical | Double tap | Medium |
| Get kill | Triple tap | Strong |
| Take damage | Single pulse | Medium |
| Take heavy damage | Double pulse | Strong |
| Low health warning | Heartbeat pattern | Light (repeating) |
| Cooldown ready | Single tick | Light |

### 8.2 Haptic Patterns
```
Damage Dealt:
├── Light: 10ms vibration
├── Medium: 20ms vibration
└── Heavy: 30ms vibration

Damage Received:
├── Light: 15ms vibration, 10ms pause, 10ms vibration
├── Heavy: 25ms vibration, 15ms pause, 20ms vibration
└── Critical: Full 100ms rumble

Kill Confirmation:
├── 20ms, pause, 15ms, pause, 25ms (success pattern)
└── Distinct from damage patterns
```

### 8.3 Platform Support
```
iOS:
├── UIImpactFeedbackGenerator
├── Light, Medium, Heavy impacts
├── Custom patterns via CoreHaptics
└── Taptic Engine support

Android:
├── VibrationEffect API (API 26+)
├── Custom waveforms
├── Fallback for older devices
└── Device-specific testing required
```

### 8.4 Settings
```
Haptic Feedback:
├── Enable: [On] / Off
├── Damage Dealt: [On] / Off
├── Damage Received: [On] / Off
├── Kill Confirmation: [On] / Off
├── Low Health: [On] / Off
├── Overall Intensity: Low / [Medium] / High
└── Respect System Settings: [On] / Off
```

---

## 9. Audio Feedback

### 9.1 Impact Sounds
| Event | Sound Type | Volume | Priority |
|-------|------------|--------|----------|
| Hit enemy | Impact thud | Medium | High |
| Critical hit | Enhanced impact + ping | High | High |
| Kill | Kill confirm chime | High | Highest |
| Miss | None or whoosh | Low | Low |
| Hit shield | Metallic clang | Medium | High |

### 9.2 Received Damage Sounds
```
Grunt/Pain Vocalization:
├── Light damage: Soft grunt
├── Medium damage: Louder grunt
├── Heavy damage: Pain cry
├── Fire damage: Fire-specific vocalization
├── Poison damage: Coughing
└── Cooldown: Don't spam (1s minimum between)

Impact Sound:
├── Varies by damage type
├── Spatial audio (from direction)
├── Layered with vocalization
└── Volume scales with damage
```

### 9.3 Audio Settings
```
Combat Audio:
├── Hit Sound: [On] / Off
├── Kill Sound: [On] / Off
├── Damage Vocalization: [On] / Off
├── Heartbeat: [On] / Off
├── Combat Audio Volume: [████████░░] 80%
└── Respect Silent Mode: [On] / Off
```

---

## 10. Accessibility Options

### 10.1 Visual Alternatives
```
For Hearing Impaired:
├── Enhanced visual indicators
├── Longer flash durations
├── Stronger screen effects
├── Text notifications for audio cues
└── Vibration for all sounds

Options:
├── Visual Sound Indicators: On / [Off]
├── Extended Flash Duration: On / [Off]
├── High Contrast Indicators: On / [Off]
└── Always Show Damage Direction: On / [Off]
```

### 10.2 Reduced Effects
```
For Photosensitive Users:
├── Disable screen flash
├── Disable screen shake
├── Reduce vignette intensity
├── Slower animations
└── No particle bursts

Options:
├── Reduce Screen Effects: On / [Off]
├── Disable Flashing: On / [Off]
├── Safe Mode (all reduced): On / [Off]
└── Test Effects: [Test Button]
```

### 10.3 Colorblind Support
| Element | Default | Deuteranopia | Protanopia |
|---------|---------|--------------|------------|
| Damage taken | Red | Orange | Orange |
| Health numbers | Green | Blue | Cyan |
| Critical | Yellow | Yellow | Yellow |
| Poison | Green | Purple | Purple |
| Fire | Orange | Orange | Orange |

---

## 11. Performance Optimization

### 11.1 Rendering Budget
| Effect | Budget |
|--------|--------|
| Hit markers | < 0.1ms |
| Damage numbers | < 0.2ms |
| Screen effects | < 0.3ms |
| Particle systems | < 0.4ms |
| Total | < 1ms/frame |

### 11.2 Pooling Systems
```
Object Pools:
├── Damage number pool: 20 instances
├── Hit marker pool: 10 instances
├── Directional indicator pool: 8 instances
└── Pre-instantiated, recycled

Benefits:
├── No runtime allocation
├── Consistent performance
├── No garbage collection spikes
└── Memory usage predictable
```

### 11.3 Quality Scaling
```
Low Performance Mode:
├── Reduce particle counts
├── Simpler damage numbers (no animation)
├── Shorter effect durations
├── Skip non-critical effects
└── Maintain essential feedback only
```

---

## 12. Integration Points

### 12.1 System Dependencies
| System | Data Provided |
|--------|--------------|
| Combat System | Damage amount, type, source |
| Health System | Current health, threshold events |
| Network | Damage confirmation, sync |
| Audio Manager | Sound playback |
| Haptic Manager | Vibration triggers |

### 12.2 Events
```csharp
// Damage feedback events
public event Action<DamageInfo> OnDamageDealt;     // Player deals damage
public event Action<DamageInfo> OnDamageReceived;  // Player takes damage
public event Action<KillInfo> OnKillConfirmed;     // Player gets kill
public event Action OnLowHealthEntered;
public event Action OnLowHealthExited;
public event Action<DamageSource> OnDirectionalDamage;
```

### 12.3 Data Structures
```csharp
public struct DamageInfo
{
    public int amount;
    public DamageType type;       // Normal, Critical, Fire, Poison, etc.
    public Vector3 sourcePosition;
    public Entity target;
    public Entity source;
    public bool isKillingBlow;
}

public enum DamageType
{
    Normal,
    Critical,
    Fire,
    Poison,
    Storm,
    Fall,
    Melee,
    Ability
}
```

---

## 13. Testing Requirements

### 13.1 Functional Tests
```
Verify:
├── Hit marker appears on damage dealt
├── Damage numbers display correctly
├── Critical hits show enhanced feedback
├── Kill confirmation triggers
├── Directional indicator accurate
├── Low health warning activates
├── Screen shake functions
├── Haptic feedback triggers
├── All settings apply correctly
└── No feedback on miss
```

### 13.2 Visual Tests
```
Verify:
├── All indicators visible on all backgrounds
├── Colors distinguishable
├── Text readable at all sizes
├── Animations smooth (60 FPS)
├── No visual artifacts
├── Proper layering (not obscured)
└── Works in colorblind modes
```

### 13.3 Performance Tests
```
Verify:
├── 60 FPS maintained during intense combat
├── No frame drops with multiple damage sources
├── Memory stable (no leaks)
├── Object pools working correctly
├── Haptics don't cause frame drops
└── Audio doesn't clip or lag
```

---

## 14. Default Configuration

```json
{
  "hitMarker": {
    "enabled": true,
    "duration": 0.15,
    "size": "medium",
    "showCritical": true
  },
  "damageNumbers": {
    "enabled": true,
    "showDealt": true,
    "showReceived": true,
    "combineRapid": false,
    "size": "medium",
    "duration": 1.0
  },
  "screenEffects": {
    "damageVignette": true,
    "vignetteIntensity": "medium",
    "screenShake": true,
    "shakeIntensity": 0.7,
    "directionalIndicator": true
  },
  "lowHealth": {
    "visualWarning": true,
    "audioWarning": true,
    "threshold": 0.25,
    "heartbeatVolume": 0.5
  },
  "killConfirmation": {
    "hitMarker": true,
    "sound": true,
    "screenFlash": false,
    "killStreak": true
  },
  "haptic": {
    "enabled": true,
    "damageDealt": true,
    "damageReceived": true,
    "killConfirm": true,
    "lowHealth": true,
    "intensity": "medium"
  },
  "audio": {
    "hitSound": true,
    "killSound": true,
    "vocalizations": true,
    "volume": 0.8
  },
  "accessibility": {
    "reducedEffects": false,
    "disableFlashing": false,
    "colorblindMode": "default",
    "visualSoundIndicators": false
  }
}
```

---

## Appendix A: Visual Reference

```
Hit Marker Appearance:

  Standard:     Critical:      Kill:

    \   /        \ | /        ╲ │ ╱
     \ /          \|/          ╲│╱
      X           ─X─          ─X─
     / \          /|\          ╱│╲
    /   \        / | \        ╱ │ ╲

  White         Yellow        Red

Directional Damage:
┌─────────────────────────────────────┐
│                ▼                    │
│          (damage from              │
│             above)                  │
│                                     │
│  ◀──────── PLAYER ────────▶         │
│  (left)              (right)        │
│                                     │
│                ▲                    │
│          (damage from              │
│            behind)                  │
└─────────────────────────────────────┘

Low Health Warning:
┌─────────────────────────────────────┐
│ ╔═══════════════════════════════╗   │
│ ║░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░║   │
│ ║░░░                        ░░░║   │
│ ║░░                          ░░║   │
│ ║░         [GAMEPLAY]         ░║   │
│ ║░                            ░║   │
│ ║░░                          ░░║   │
│ ║░░░                        ░░░║   │
│ ║░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░║   │
│ ╚═══════════════════════════════╝   │
│        ↑ Red vignette ↑             │
└─────────────────────────────────────┘
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
