# UX-015: Settings Menu Specification

## Overview

This document specifies the comprehensive settings menu for Plunderstorm Mobile, covering gameplay, controls, audio, video, and account settings. The settings system must be intuitive, well-organized, and save preferences reliably across sessions.

## Design Philosophy

Settings should be logically organized, easy to navigate, and provide clear explanations for each option. Players should be able to quickly find and adjust any setting, with changes taking effect immediately where possible. Default values are carefully chosen to work well for most players.

---

## 1. Settings Menu Structure

### 1.1 Main Categories
```
Settings Menu:
├── 1. Controls
│   ├── Movement
│   ├── Camera
│   ├── Abilities
│   └── Custom Layout
│
├── 2. Graphics
│   ├── Quality Preset
│   ├── Advanced Settings
│   └── Battery Saver
│
├── 3. Audio
│   ├── Volume
│   ├── Sound Settings
│   └── Voice Chat
│
├── 4. Gameplay
│   ├── HUD Settings
│   ├── Combat Feedback
│   └── Accessibility
│
├── 5. Account
│   ├── Profile
│   ├── Linked Accounts
│   └── Privacy
│
└── 6. About
    ├── Version Info
    ├── Legal
    └── Support
```

### 1.2 Navigation Design
```
Layout:
├── Left: Category list (tabs or list)
├── Right: Setting options for selected category
├── Top: Back button, category name
├── Bottom: Reset to Defaults, Apply (if needed)

Navigation:
├── Tap category to view settings
├── Scroll within categories
├── Back button returns to game
├── Settings save automatically (or Apply button)
```

---

## 2. Controls Settings

### 2.1 Movement Controls
```
Movement Settings:
│
├── Joystick Type
│   └── [Floating] / Fixed
│   └── Description: "Floating joystick appears at touch point"
│
├── Joystick Size
│   └── [Small] [Medium] [Large]
│   └── Preview: Visual representation
│
├── Dead Zone
│   └── [████████░░░░] 20%
│   └── Range: 5% - 40%
│
├── Sprint Activation
│   └── [Double-Tap] / Button / Auto
│   └── Description: "How to activate sprint"
│
├── Movement Smoothing
│   └── [████████████░░░░] 60%
│   └── Description: "Smoother movement, slight delay"
│
└── [Reset Movement Defaults]
```

### 2.2 Camera Controls
```
Camera Settings:
│
├── Horizontal Sensitivity
│   └── [████████░░] 8/10
│   └── Preview: Test area
│
├── Vertical Sensitivity
│   └── [██████░░░░] 6/10
│   └── Preview: Test area
│
├── Invert Horizontal
│   └── [Off] / On
│
├── Invert Vertical
│   └── [Off] / On
│
├── Camera Smoothing
│   └── [Snappy] [Balanced] [Cinematic]
│
├── Auto-Center
│   └── [On] / Off
│   └── Delay: [████░░░░░░] 2.0s
│
├── Gyroscope
│   └── Enable: [Off] / On
│   └── Sensitivity: [█████░░░░░] 5/10
│   └── [Calibrate Gyroscope]
│
└── [Reset Camera Defaults]
```

### 2.3 Ability Controls
```
Ability Settings:
│
├── Default Targeting
│   └── [Drag-and-Release] / Tap-Tap / Quick Cast
│
├── Per-Ability Targeting (Expandable)
│   ├── Fireball: [Default]
│   ├── Lightning Strike: [Tap-Tap]
│   └── ... (each ability)
│
├── Quick Cast Toggle
│   └── [Off] / On
│   └── Warning: "Fires immediately in facing direction"
│
├── Aim Assist
│   └── [Off] [Low] [Medium] [High]
│
├── Cancel Method
│   └── [Drag Back] [Lift Finger] [Both]
│
├── Ability Button Size
│   └── [Small] [Medium] [Large]
│
└── [Reset Ability Defaults]
```

### 2.4 Custom Layout
```
Layout Editor:
│
├── [Edit Button Positions]
│   └── Opens full-screen drag editor
│
├── Preset Layouts
│   └── [Default] [MOBA] [Shooter] [Left-Handed]
│
├── Save Custom Layout
│   └── Slot 1: [Save] [Load]
│   └── Slot 2: [Save] [Load]
│   └── Slot 3: [Save] [Load]
│
├── Import/Export
│   └── [Import Layout Code]
│   └── [Export Layout Code]
│
└── [Reset All Controls]
```

---

## 3. Graphics Settings

### 3.1 Quality Presets
```
Graphics Quality:
│
├── Quality Preset
│   └── [Low] [Medium] [High] [Ultra] [Custom]
│   └── Recommended: [Based on device]
│
├── Preset Details (Collapsed by default)
│   └── Low: 30 FPS, minimal effects
│   └── Medium: 30-60 FPS, standard effects
│   └── High: 60 FPS, enhanced effects
│   └── Ultra: 60 FPS, maximum quality
```

### 3.2 Advanced Graphics
```
Advanced Settings (When Custom selected):
│
├── Frame Rate Cap
│   └── [30 FPS] [60 FPS] [Unlimited]
│   └── Warning: "Higher FPS uses more battery"
│
├── Resolution Scale
│   └── [████████████░░░░] 75%
│   └── Range: 50% - 100%
│
├── Texture Quality
│   └── [Low] [Medium] [High]
│
├── Shadow Quality
│   └── [Off] [Low] [Medium] [High]
│
├── Effects Quality
│   └── [Low] [Medium] [High]
│
├── Anti-Aliasing
│   └── [Off] [Low] [High]
│
├── View Distance
│   └── [Low] [Medium] [High]
│
├── Foliage Density
│   └── [Low] [Medium] [High]
│
└── [Reset to Preset]
```

### 3.3 Battery and Performance
```
Battery Settings:
│
├── Battery Saver Mode
│   └── [Off] / On
│   └── Description: "Reduces quality when battery < 20%"
│
├── Auto-Adjust Quality
│   └── [Off] / On
│   └── Description: "Automatically lower settings if FPS drops"
│
├── Reduce Heat
│   └── [Off] / On
│   └── Description: "Limit performance to reduce device temperature"
│
├── Show FPS Counter
│   └── [Off] / On
│
├── Show Ping
│   └── [Off] / On
│
└── [Run Performance Test]
```

---

## 4. Audio Settings

### 4.1 Volume Controls
```
Volume Settings:
│
├── Master Volume
│   └── [████████████████░░] 80%
│   └── Test: [🔊 Play Sample]
│
├── Music Volume
│   └── [████████████░░░░░░] 60%
│   └── Test: [🎵 Play Sample]
│
├── Sound Effects Volume
│   └── [██████████████████] 100%
│   └── Test: [🔊 Play Sample]
│
├── Voice Chat Volume
│   └── [██████████████░░░░] 70%
│   └── Test: [🎤 Test Mic]
│
├── Ambient Sounds
│   └── [████████████░░░░░░] 60%
│
└── [Mute All]
```

### 4.2 Sound Settings
```
Sound Options:
│
├── Sound Quality
│   └── [Low] [Medium] [High]
│
├── 3D Audio
│   └── [On] / Off
│   └── Description: "Spatial sound for directional awareness"
│
├── Mono Audio
│   └── [Off] / On
│   └── Description: "Combine stereo channels (accessibility)"
│
├── Mute in Background
│   └── [On] / Off
│
├── Vibration/Haptics
│   └── [On] / Off
│   └── Intensity: [Low] [Medium] [High]
│
└── [Reset Audio Defaults]
```

### 4.3 Voice Chat
```
Voice Chat Settings:
│
├── Voice Chat
│   └── [On] / Off
│
├── Microphone Mode
│   └── [Push-to-Talk] / Open Mic
│
├── Push-to-Talk Button
│   └── [Configure Position]
│
├── Microphone Sensitivity
│   └── [████████████░░░░] 70%
│
├── Voice Chat Volume
│   └── [████████████░░░░] 70%
│
├── Teammate Voice Only
│   └── [On] / Off
│
├── Text-to-Speech
│   └── [Off] / On
│   └── Description: "Read text chat aloud"
│
└── [Test Microphone]
```

---

## 5. Gameplay Settings

### 5.1 HUD Settings
```
HUD Options:
│
├── HUD Scale
│   └── [████████████░░░░] 100%
│   └── Range: 75% - 150%
│
├── HUD Opacity
│   └── [████████████████░░] 90%
│   └── Range: 40% - 100%
│
├── Minimap Size
│   └── [Small] [Medium] [Large]
│
├── Minimap Position
│   └── [Top-Left] [Top-Right] [Bottom-Left] [Bottom-Right]
│
├── Minimap Rotation
│   └── [Fixed North] / Rotating
│
├── Kill Feed
│   └── [On] / Off
│
├── Damage Numbers
│   └── [On] / Off
│
├── Compass
│   └── [On] / Off
│
├── FPS Counter
│   └── [Off] / On
│
└── [Reset HUD Defaults]
```

### 5.2 Combat Feedback
```
Combat Feedback:
│
├── Hit Markers
│   └── [On] / Off
│
├── Damage Numbers
│   └── [On] / Off
│   └── Size: [Small] [Medium] [Large]
│
├── Directional Damage
│   └── [On] / Off
│
├── Screen Shake
│   └── [On] / Off
│   └── Intensity: [████████░░░░] 70%
│
├── Low Health Warning
│   └── Visual: [On] / Off
│   └── Audio: [On] / Off
│   └── Threshold: [25%] / 15% / 10%
│
├── Kill Confirmation Sound
│   └── [On] / Off
│
└── [Reset Combat Defaults]
```

### 5.3 Accessibility
```
Accessibility Options:
│
├── Colorblind Mode
│   └── [Off] [Deuteranopia] [Protanopia] [Tritanopia]
│   └── Preview: Color sample display
│
├── High Contrast UI
│   └── [Off] / On
│
├── Text Size
│   └── [Normal] [Large] [Extra Large]
│
├── Button Size
│   └── [Normal] [Large] [Extra Large]
│
├── Reduce Motion
│   └── [Off] / On
│   └── Description: "Disable animations and camera effects"
│
├── Disable Flashing
│   └── [Off] / On
│
├── Screen Reader Support
│   └── [Off] / On
│
├── Subtitles
│   └── [Off] / On
│   └── Size: [Small] [Medium] [Large]
│
├── One-Handed Mode
│   └── [Off] / On
│   └── Hand: [Right] / Left
│
└── [Reset Accessibility Defaults]
```

---

## 6. Account Settings

### 6.1 Profile
```
Profile Settings:
│
├── Display Name
│   └── [PlayerName123]
│   └── [Change Name] (may require currency)
│
├── Avatar
│   └── [Current Avatar Image]
│   └── [Change Avatar]
│
├── Player ID
│   └── #ABC123XYZ (copy button)
│
├── Statistics (View Only)
│   └── Total Matches: 150
│   └── Wins: 12
│   └── Eliminations: 487
│   └── [View Full Stats]
│
└── [View Profile]
```

### 6.2 Linked Accounts
```
Account Linking:
│
├── Current Login
│   └── Guest / Apple / Google / Facebook
│
├── Link Accounts
│   └── Apple: [Link] / [Linked ✓]
│   └── Google: [Link] / [Linked ✓]
│   └── Facebook: [Link] / [Linked ✓]
│
├── Cloud Save
│   └── [On] / Off
│   └── Last Sync: 2025-12-02 14:30
│   └── [Sync Now]
│
├── Log Out
│   └── [Log Out]
│   └── Warning: "Progress may be lost if not linked"
│
└── Delete Account
    └── [Delete Account]
    └── Warning: "This action is permanent"
```

### 6.3 Privacy Settings
```
Privacy Options:
│
├── Online Status
│   └── [Visible] / Friends Only / Hidden
│
├── Friend Requests
│   └── [Everyone] / Friends of Friends / Nobody
│
├── Party Invites
│   └── [Everyone] / Friends Only / Nobody
│
├── Chat Messages
│   └── [Everyone] / Friends Only / Nobody
│
├── Appear Offline
│   └── [Off] / On
│
├── Share Statistics
│   └── [On] / Off
│
├── Data Collection
│   └── Analytics: [On] / Off
│   └── Crash Reports: [On] / Off
│
└── [View Privacy Policy]
```

---

## 7. Notifications Settings

### 7.1 Push Notifications
```
Notifications:
│
├── Enable Notifications
│   └── [On] / Off
│
├── Notification Types
│   ├── Daily Rewards: [On] / Off
│   ├── Events & Updates: [On] / Off
│   ├── Friend Activity: [On] / Off
│   ├── Energy Full: [On] / Off
│   └── Battle Pass: [On] / Off
│
├── Quiet Hours
│   └── [Off] / On
│   └── Start: [22:00]
│   └── End: [08:00]
│
└── [Open System Settings]
```

---

## 8. About Section

### 8.1 Version Information
```
About:
│
├── Version
│   └── 1.0.0 (Build 12345)
│
├── Device Info
│   └── Model: iPhone 14 Pro
│   └── OS: iOS 17.0
│   └── Performance Tier: High
│
├── Server Region
│   └── [Auto] / US East / US West / EU / Asia
│
└── [Check for Updates]
```

### 8.2 Legal
```
Legal:
│
├── Terms of Service
│   └── [View Terms]
│
├── Privacy Policy
│   └── [View Privacy Policy]
│
├── Open Source Licenses
│   └── [View Licenses]
│
└── Copyright
    └── © 2025 Developer Name
```

### 8.3 Support
```
Support:
│
├── FAQ
│   └── [View FAQ]
│
├── Report a Bug
│   └── [Report Bug]
│
├── Contact Support
│   └── [Contact Us]
│
├── Community
│   └── Discord: [Join]
│   └── Twitter: [Follow]
│
└── Clear Cache
    └── [Clear Cache]
    └── Size: 256 MB
```

---

## 9. Settings Persistence

### 9.1 Save System
```
Storage:
├── Local Storage (device)
│   └── Primary storage
│   └── Immediate saves
│   └── Works offline
│
└── Cloud Storage (account)
    └── Backup storage
    └── Syncs when online
    └── Restores on new device
```

### 9.2 Save Timing
| Event | Action |
|-------|--------|
| Setting Changed | Save locally immediately |
| Menu Closed | Sync to cloud |
| Game Launched | Load local, then sync from cloud |
| Conflict | Cloud takes precedence (with warning) |

### 9.3 Migration
```
Version Updates:
├── Preserve existing settings
├── Add new settings with defaults
├── Remove deprecated settings
└── Notify user of significant changes
```

---

## 10. UI/UX Design

### 10.1 Visual Style
```
Design Guidelines:
├── Match game's pirate aesthetic
├── Dark theme for readability
├── High contrast for options
├── Clear typography
├── Consistent spacing
└── Touch-friendly sizes (44pt minimum)
```

### 10.2 Interaction Patterns
```
Controls:
├── Toggles: Binary on/off
├── Sliders: Continuous values
├── Dropdowns: Multiple choices
├── Radio Buttons: Single selection from group
├── Number Input: Direct value entry (rare)
└── Action Buttons: Trigger actions (Reset, Test)
```

### 10.3 Feedback
```
On Change:
├── Visual confirmation (checkmark, highlight)
├── Audio click (if audio enabled)
├── Haptic tap (if haptics enabled)
├── Preview where applicable
└── "Saved" indicator (brief)
```

---

## 11. Device-Specific Recommendations

### 11.1 Auto-Detection
```
On First Launch:
├── Detect device model and specs
├── Set recommended graphics preset
├── Set recommended control size
├── Suggest optimal settings
└── Allow user to override
```

### 11.2 Recommendations Display
```
Format:
├── "Recommended" badge on optimal options
├── Warning for settings that may cause issues
├── Performance impact indicators
└── Battery impact indicators
```

---

## 12. Testing Requirements

### 12.1 Functional Tests
```
Verify:
├── All settings save correctly
├── All settings load on restart
├── Cloud sync works
├── Reset buttons work
├── Preview functions work
├── All sliders responsive
├── All toggles functional
└── Navigation smooth
```

### 12.2 Edge Cases
```
Test:
├── Offline mode (no cloud)
├── Account switch
├── Settings migration
├── Conflicting cloud/local
├── Extreme slider values
├── Rapid setting changes
└── Interrupted saves
```

---

## 13. Default Configuration

```json
{
  "controls": {
    "joystickType": "floating",
    "joystickSize": "medium",
    "deadZone": 0.20,
    "sprintActivation": "doubleTap",
    "cameraSensitivityH": 8,
    "cameraSensitivityV": 6,
    "invertH": false,
    "invertV": false,
    "gyroscopeEnabled": false,
    "targetingMethod": "dragAndRelease",
    "aimAssist": "medium"
  },
  "graphics": {
    "preset": "auto",
    "frameRateCap": 60,
    "resolutionScale": 1.0,
    "shadows": "medium",
    "effects": "medium",
    "antiAliasing": "low",
    "batterySaver": true
  },
  "audio": {
    "masterVolume": 0.8,
    "musicVolume": 0.6,
    "sfxVolume": 1.0,
    "voiceVolume": 0.7,
    "haptics": true,
    "hapticsIntensity": "medium"
  },
  "gameplay": {
    "hudScale": 1.0,
    "hudOpacity": 0.9,
    "minimapSize": "medium",
    "minimapPosition": "topLeft",
    "damageNumbers": true,
    "hitMarkers": true,
    "screenShake": true,
    "screenShakeIntensity": 0.7
  },
  "accessibility": {
    "colorblindMode": "off",
    "highContrast": false,
    "textSize": "normal",
    "reduceMotion": false,
    "disableFlashing": false
  },
  "privacy": {
    "onlineStatus": "visible",
    "friendRequests": "everyone",
    "analytics": true,
    "crashReports": true
  }
}
```

---

## Appendix A: Settings Categories Quick Reference

| Category | Settings Count | Priority |
|----------|---------------|----------|
| Controls | 25+ | Critical |
| Graphics | 15+ | High |
| Audio | 12+ | High |
| Gameplay | 15+ | High |
| Account | 10+ | Medium |
| Privacy | 8+ | Medium |
| Notifications | 6+ | Low |
| About | 5+ | Low |

---

## Appendix B: Visual Reference

```
Settings Menu Layout:

┌─────────────────────────────────────────────────┐
│ ←  Settings                                     │
├──────────────┬──────────────────────────────────┤
│              │                                  │
│ ⚙️ Controls  │  Movement                        │
│              │  ─────────────────────────────   │
│ 🎮 Graphics  │                                  │
│              │  Joystick Type                   │
│ 🔊 Audio     │  [Floating ▼]                    │
│              │                                  │
│ 🎯 Gameplay  │  Joystick Size                   │
│              │  [S] [M] [L]                     │
│ 👤 Account   │       ▲                          │
│              │  Dead Zone                       │
│ ℹ️ About     │  [████████░░] 20%                │
│              │                                  │
│              │  Sprint Activation               │
│              │  [Double-Tap ▼]                  │
│              │                                  │
│              │                                  │
│              │  [Reset to Defaults]             │
└──────────────┴──────────────────────────────────┘
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
