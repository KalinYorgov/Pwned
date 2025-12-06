# Mobile UX TODO - Touch Controls & User Interface

This document covers all mobile-specific user experience elements including touch controls, UI/HUD design, accessibility, and platform-specific optimizations.

---

## Table of Contents
1. [Touch Control System](#1-touch-control-system)
2. [HUD and In-Game UI](#2-hud-and-in-game-ui)
3. [Menu and Navigation](#3-menu-and-navigation)
4. [Accessibility Features](#4-accessibility-features)
5. [Platform Optimization](#5-platform-optimization)
6. [Onboarding and FTUE](#6-onboarding-and-ftue)

---

## 1. Touch Control System

### UX-001: Virtual Joystick Movement
**Priority:** P0 | **Complexity:** M

**Description:**
Implement responsive virtual joystick for player movement. The joystick should appear where the player touches on the left side of the screen and provide smooth, predictable movement control.

**Acceptance Criteria:**
- [x] Touch anywhere on left 40% of screen activates movement joystick
- [x] Joystick appears at touch point (floating joystick)
- [x] Joystick has dead zone to prevent accidental movement
- [x] Full stick displacement = max speed; partial = proportional speed
- [x] Visual feedback shows joystick position and direction
- [x] Sprint option: double-tap and hold for sprint
- [x] Joystick size adjustable in settings (small/medium/large)
- [x] Optional fixed position joystick mode
- [x] Smooth acceleration/deceleration (not instant)

**Definition of Done:**
- Movement feels responsive (< 16ms input lag)
- Works on all screen sizes (phones and tablets)
- No conflicts with other touch zones
- Tested with various thumb sizes
- Player feedback positive in playtesting

**Deliverable:** [Virtual Joystick Movement Specification](docs/VIRTUAL_JOYSTICK_MOVEMENT_SPEC.md)

**Status:** Completed

**Dependencies:** None

---

### UX-002: Camera Control System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement touch-based camera control for the third-person view. Players swipe on the right side of the screen to rotate the camera.

**Acceptance Criteria:**
- [x] Touch and drag on right 60% of screen rotates camera
- [x] Horizontal swipe = camera yaw (rotate around player)
- [x] Vertical swipe = camera pitch (look up/down, with limits)
- [x] Camera sensitivity adjustable in settings (1-10 scale)
- [x] Camera smoothing option (for those who prefer less snappy)
- [x] Camera doesn't clip through walls (collision detection)
- [x] Quick 180-degree turn option (double-tap right side)
- [x] Camera auto-centers behind player when moving (optional)
- [x] Gyroscope/accelerometer aiming option

**Definition of Done:**
- Camera feels natural and responsive
- No motion sickness from camera movement
- Works correctly in all gameplay situations
- Gyro aiming calibration tested
- Settings remembered per device

**Deliverable:** [Camera Control System Specification](docs/CAMERA_CONTROL_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** None

---

### UX-003: Ability Button Layout
**Priority:** P0 | **Complexity:** M

**Description:**
Design and implement the touch button layout for abilities and basic attack. Must balance accessibility with preventing accidental presses.

**Layout Design:**
- Right side: 4 ability buttons + basic attack button
- Arranged in arc or cluster for thumb reach
- Each button sized for reliable touch (minimum 44pt)

**Acceptance Criteria:**
- [x] Basic attack button in easy-to-reach position (bottom right)
- [x] 4 ability buttons arranged around basic attack
- [x] Clear visual distinction between offensive (red) and utility (blue) abilities
- [x] Button shows ability icon, cooldown overlay, and hotkey number
- [x] Press to activate instant abilities
- [x] Press and hold to enter targeting mode for skillshots
- [x] Drag from button to aim, release to fire
- [x] Cancel skillshot by dragging back to button
- [x] Buttons gray out when on cooldown
- [x] Customizable button positions via drag-and-drop editor

**Definition of Done:**
- All abilities usable without looking at controls
- Muscle memory develops within 2-3 matches
- No accidental ability activations in testing
- Layout works on 5.5" to 12.9" screens
- Touch targets meet accessibility guidelines

**Deliverable:** [Ability Button Layout Specification](docs/ABILITY_BUTTON_LAYOUT_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-007 (Ability Slot System)

---

### UX-004: Skillshot Targeting System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement intuitive skillshot targeting for abilities that require aiming (projectiles, ground-targeted spells).

**Targeting Methods:**
1. **Drag-and-Release:** Hold ability, drag to aim direction, release to fire
2. **Tap-Tap:** Tap ability to activate, tap target location to fire
3. **Quick Cast:** Tap ability to immediately fire in facing direction

**Acceptance Criteria:**
- [x] All three targeting methods supported
- [x] Player can set preferred method in settings (per ability optional)
- [x] Targeting indicator shows trajectory/landing zone
- [x] Range indicator shows max ability range
- [x] Valid targets highlighted (enemies in range)
- [x] Invalid areas grayed out (out of range, blocked by terrain)
- [x] Targeting can be canceled without firing
- [x] Aim sensitivity adjustable
- [x] Joystick can be used simultaneously while aiming (move + aim)

**Definition of Done:**
- Skillshots feel accurate and satisfying to land
- New players can understand targeting within tutorial
- No frustration from "misfire" issues
- Performance stable during targeting mode
- Tested with all skillshot abilities

**Deliverable:** [Skillshot Targeting System Specification](docs/SKILLSHOT_TARGETING_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** UX-003, GAME-008

---

### UX-005: Auto-Aim Assist System
**Priority:** P1 | **Complexity:** M

**Description:**
Implement optional aim assist to help mobile players land shots. Should help accuracy without feeling like cheating.

**Aim Assist Features:**
- **Soft Lock:** Crosshair slightly pulls toward nearby enemies
- **Target Snap:** Basic attack auto-targets nearest enemy in cone
- **Bullet Magnetism:** Projectiles slightly curve toward targets

**Acceptance Criteria:**
- [x] Aim assist is optional (can disable in settings)
- [x] Assist strength adjustable (off, low, medium, high)
- [x] Only assists with visible, valid targets
- [x] Assist does not work through walls
- [x] Higher-skill abilities (like sniper shot) have less assist
- [x] Visual indicator shows when aim assist is active
- [x] No assist in ranked mode above certain tier (optional rule)
- [x] Assist works with both basic attack and abilities

**Definition of Done:**
- Casual players feel capable with assist on
- Skilled players can compete with assist off
- No exploitable behavior (spin-botting, etc.)
- Assist values tuned through playtesting
- Clear communication in settings about assist effects

**Dependencies:** GAME-011, GAME-013

**Deliverable:** [Auto-Aim Assist System Specification](docs/AUTO_AIM_ASSIST_SYSTEM_SPEC.md)

**Status:** Completed

---

### UX-006: Gesture Controls
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Implement additional gesture controls for quality-of-life actions.

**Gestures:**
- **Double-tap ground:** Interact with nearby chest/loot
- **Pinch:** Zoom minimap
- **Three-finger tap:** Open quick menu (optional)
- **Swipe from edge:** Open scoreboard (optional)

**Acceptance Criteria:**
- [x] Gestures do not conflict with core controls
- [x] Gestures are optional and can be disabled
- [x] Gesture actions are also available via buttons
- [x] Visual feedback confirms gesture recognized
- [x] Gestures documented in settings/help

**Definition of Done:**
- Gestures feel natural and discoverable
- No accidental gesture triggers during combat
- Tested on various device sizes

**Deliverables:**
- [Gesture Controls Specification](docs/GESTURE_CONTROLS_SPEC.md)

**Dependencies:** UX-001, UX-002

---

### UX-007: Controller Support
**Priority:** P3 | **Complexity:** M

**Description:**
Add support for external Bluetooth/USB controllers for players who prefer physical controls.

**Acceptance Criteria:**
- [ ] Auto-detect connected controllers
- [ ] Standard mobile controller mapping (MFi, Xbox, PlayStation)
- [ ] All actions mappable to controller buttons
- [ ] Controller and touch can be used simultaneously
- [ ] Controller-specific UI (button prompts change to icons)
- [ ] Vibration/haptic feedback on controller
- [ ] Fair matchmaking consideration (optional separate queue)

**Definition of Done:**
- Smooth experience on supported controllers
- Button prompts accurate for controller type
- No advantage over touch players (or balanced)
- Tested with popular mobile controllers

**Dependencies:** All UX tasks

---

## 2. HUD and In-Game UI

### UX-008: Main HUD Layout
**Priority:** P0 | **Complexity:** M

**Description:**
Design and implement the in-game HUD showing all essential information without cluttering the screen.

**HUD Elements:**
- Health bar (center bottom or top)
- Ability slots (right side)
- Minimap (top corner)
- Kill feed (top corner, opposite minimap)
- Player count remaining (top center)
- Storm timer/phase indicator
- XP bar and level indicator
- Ammo/cooldown indicators

**Acceptance Criteria:**
- [x] All essential info visible at a glance
- [x] HUD elements do not obscure central gameplay area
- [x] Elements scaled appropriately for device size
- [x] Critical info (health, abilities) always visible
- [x] Secondary info (kill feed) can be minimized
- [x] HUD transparency adjustable
- [x] Safe area respected (notch, rounded corners)
- [x] Landscape orientation optimized
- [x] Portrait mode not supported (or limited)

**Definition of Done:**
- HUD readable on 5" phone screens
- No information overload
- Quick access to all needed data
- Tested on various aspect ratios (16:9, 18:9, 19.5:9)
- Themed to match pirate aesthetic without sacrificing clarity

**Deliverable:** [Main HUD Layout Specification](docs/MAIN_HUD_LAYOUT_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-007, GAME-018

---

### UX-009: Minimap System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement a minimap showing player position, storm location, teammates, and points of interest.

**Minimap Features:**
- Player arrow showing position and facing
- Storm circle current and next position
- Teammate positions (in team modes)
- Ping markers
- Drop/loot locations (optional toggle)
- Enemy sighting (briefly shown when shooting)

**Acceptance Criteria:**
- [x] Minimap in corner (adjustable: any corner)
- [x] Zoomable minimap (pinch or button toggle)
- [x] Tap to enlarge to full-screen map view
- [x] Full map shows all named locations
- [x] Storm position clearly marked (current + next)
- [x] Team colors consistent
- [x] Pings appear on minimap with icons
- [x] North indicator for orientation
- [x] Treasure map aesthetic (styled border)

**Definition of Done:**
- Minimap provides tactical advantage without distraction
- All information readable at small size
- Full map usable for strategic planning
- Performance impact negligible
- Works correctly as storm shrinks

**Deliverable:** [Minimap System Specification](docs/MINIMAP_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-003, UX-010

---

### UX-010: Ping/Communication System
**Priority:** P1 | **Complexity:** M

**Description:**
Implement a quick communication system for team modes using contextual pings and preset messages.

**Ping Types:**
- **Generic ping:** "Look here"
- **Enemy ping:** "Enemy spotted"
- **Loot ping:** "Loot here"
- **Danger ping:** "Danger"
- **Help ping:** "I need help"
- **Going here ping:** "I'm going here"

**Acceptance Criteria:**
- [x] Tap and hold on screen opens ping wheel
- [x] Contextual pings (ping on enemy = "Enemy spotted")
- [x] Pings appear on minimap and in 3D world
- [x] Audio cue for teammates
- [x] Ping cooldown to prevent spam
- [x] Preset text messages for quick communication
- [x] Voice chat integration (push-to-talk button)
- [x] Mute/block options for abusive players

**Definition of Done:**
- Teams can communicate without voice
- Pings are informative and non-intrusive
- Works reliably in all network conditions
- No ping spam abuse possible
- Voice chat quality acceptable

**Deliverable:** [Ping/Communication System Specification](docs/PING_COMMUNICATION_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-021, BACK-003

---

### UX-011: Damage Feedback System
**Priority:** P0 | **Complexity:** S

**Description:**
Implement clear visual and audio feedback when dealing or receiving damage.

**Feedback Elements:**
- **Dealing damage:** Hit markers, damage numbers, hit sound
- **Receiving damage:** Screen red flash, directional indicator, hurt sound
- **Critical hit:** Special marker, enhanced sound
- **Kill confirmation:** Distinct sound and visual

**Acceptance Criteria:**
- [x] Hit marker appears on crosshair when damage dealt
- [x] Damage numbers float above damaged target
- [x] Directional damage indicator shows where attack came from
- [x] Screen edges flash red when taking damage (intensity = damage)
- [x] Low health persistent warning (heartbeat, red vignette)
- [x] Kill sound is satisfying and distinct
- [x] Optional screen shake on big hits (toggleable)
- [x] Haptic feedback on device (toggleable)

**Definition of Done:**
- Player always knows when they deal/receive damage
- Directional indicator accurate
- Feedback doesn't obscure gameplay
- All feedback toggleable for accessibility
- Tested across ability types

**Deliverable:** [Damage Feedback System Specification](docs/DAMAGE_FEEDBACK_SYSTEM_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-012

---

### UX-012: Spectator UI
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Create the UI overlay for spectating matches after elimination.

**Acceptance Criteria:**
- [x] Clear "SPECTATING" label
- [x] Observed player's name and stats visible
- [x] Controls to switch between players
- [x] Free camera toggle
- [x] Leave match button
- [x] Remaining player count
- [x] Match timer
- [x] Spectator chat (isolated from alive players)

**Definition of Done:**
- Spectating is engaging experience
- UI does not obscure action
- Player switching is smooth
- All controls intuitive

**Deliverables:**
- [Spectator UI Specification](docs/SPECTATOR_UI_SPEC.md)

**Dependencies:** GAME-006

---

### UX-013: Victory/Defeat Screen
**Priority:** P1 | **Complexity:** S

**Description:**
Design end-of-match screens showing results, stats, and rewards.

**Screen Elements:**
- Placement (1st, 2nd, Top 10, etc.)
- Kill count
- Damage dealt
- Survival time
- XP/currency earned
- Battle Pass progress
- Play again / Return to lobby buttons

**Acceptance Criteria:**
- [x] Victory screen is celebratory (confetti, music)
- [x] Defeat screen is encouraging (show what went well)
- [x] All relevant stats displayed
- [x] Rewards clearly shown with animation
- [x] Share button for results (social)
- [x] Quick rematch option
- [x] Return to lobby option
- [x] Loading next match while viewing results (background)

**Definition of Done:**
- Players feel rewarded for match completion
- Stats accurate
- Smooth transition to next match
- Social sharing functional

**Deliverable:** [Victory/Defeat Screen Specification](docs/VICTORY_DEFEAT_SCREEN_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-005, LIVE-003

---

## 3. Menu and Navigation

### UX-014: Main Menu Design
**Priority:** P1 | **Complexity:** M

**Description:**
Design the main menu hub where players access all game features.

**Main Menu Sections:**
- Play (mode selection)
- Armory (loadout/cosmetics)
- Battle Pass
- Shop
- Social (friends, party)
- Settings
- Profile

**Acceptance Criteria:**
- [x] Main menu loads quickly (< 3 seconds)
- [x] Clear visual hierarchy guiding to Play button
- [x] All sections accessible within 2 taps
- [x] Animated character/ship in background (themed)
- [x] News/events banner for announcements
- [x] Daily quest preview on main screen
- [x] Battle Pass progress visible
- [x] Currency displays (gold, premium)
- [x] Settings accessible from any screen

**Definition of Done:**
- Navigation is intuitive for new players
- Theme is cohesive and appealing
- Performance is smooth (60 FPS menus)
- All features discoverable
- Tested with new users for UX issues

**Deliverable:** [Main Menu Design Specification](docs/MAIN_MENU_DESIGN_SPEC.md)

**Status:** Completed

**Dependencies:** MON-001, LIVE-001

---

### UX-015: Settings Menu
**Priority:** P0 | **Complexity:** M

**Description:**
Create comprehensive settings for gameplay, controls, audio, video, and account.

**Settings Categories:**
1. **Controls:** Sensitivity, button layout, aim assist, gestures
2. **Graphics:** Quality preset, FPS cap, effects toggle
3. **Audio:** Master, music, SFX, voice chat volumes
4. **Gameplay:** Auto-pickup, HUD scale, notifications
5. **Account:** Login, link accounts, privacy, data

**Acceptance Criteria:**
- [x] All settings organized in logical categories
- [x] Settings save immediately on change
- [x] Reset to defaults option
- [x] Preview changes before applying (for controls)
- [x] Device-specific recommendations shown
- [x] Cloud save settings to account
- [x] Import/export layout option
- [x] FPS counter toggle
- [x] Network stats display toggle

**Definition of Done:**
- All player preferences addressable
- Settings persist across sessions
- No settings cause crashes or issues
- Tooltips explain complex options
- Tested on low-end and high-end devices

**Deliverable:** [Settings Menu Specification](docs/SETTINGS_MENU_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-005

---

### UX-016: Mode Selection Screen
**Priority:** P1 | **Complexity:** S

**Description:**
Screen for selecting game mode before matchmaking.

**Modes Displayed:**
- Solo (MVP)
- Duo
- Ranked (when unlocked)
- Limited Time Modes (events)
- Practice/Training

**Acceptance Criteria:**
- [x] Each mode clearly described
- [x] Player count for each mode shown
- [x] Estimated wait time displayed
- [x] Locked modes show unlock requirements
- [x] LTM highlighted with event theming
- [x] One-tap to start matchmaking
- [x] Cancel matchmaking option
- [x] Map selection (if multiple maps)

**Definition of Done:**
- Mode selection is quick and clear
- Wait times reasonably accurate
- LTMs draw attention appropriately
- Works with all planned modes

**Deliverable:** [Mode Selection Screen Specification](docs/MODE_SELECTION_SCREEN_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-020, GAME-021, GAME-023

---

### UX-017: Loading Screens
**Priority:** P1 | **Complexity:** S

**Description:**
Design loading screens that maintain engagement and provide useful info.

**Acceptance Criteria:**
- [x] Loading bar or spinner shows progress
- [x] Gameplay tips displayed during load
- [x] Character/cosmetic preview shown
- [x] Map preview for match loading
- [x] Loading time minimized (< 10 seconds target)
- [x] Themed artwork on loading screens
- [x] No false "100%" stuck issues

**Definition of Done:**
- Players stay engaged during loads
- Loading feels fast
- Tips are actually helpful
- No jarring transitions

**Deliverable:** [Loading Screens Specification](docs/LOADING_SCREENS_SPEC.md)

**Status:** Completed

**Dependencies:** None

---

## 4. Accessibility Features

### UX-018: Colorblind Modes
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Implement colorblind accessibility options for players with color vision deficiencies.

**Modes:**
- Deuteranopia (red-green)
- Protanopia (red-green)
- Tritanopia (blue-yellow)

**Acceptance Criteria:**
- [x] Colorblind modes adjustable in settings
- [x] Enemy vs teammate colors adjusted per mode
- [x] Ability colors adjusted for clarity
- [x] Storm/zone colors adjusted
- [x] Loot rarity colors adjusted
- [x] Minimap colors adjusted
- [x] Preview mode before applying

**Definition of Done:**
- Game playable for colorblind users
- No critical information lost
- Tested with colorblind users or simulation tools

**Deliverables:**
- [Colorblind Modes Specification](docs/COLORBLIND_MODES_SPEC.md)

**Dependencies:** UX-008

---

### UX-019: Text and UI Scaling
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Allow scaling of text and UI elements for accessibility and device size optimization.

**Acceptance Criteria:**
- [x] HUD scale slider (75% to 150%)
- [x] Text size adjustment for menus
- [x] Button size linked to HUD scale or separate
- [x] High contrast mode option
- [x] Bold text option
- [x] Preview before applying

**Definition of Done:**
- UI usable on small phones and tablets
- Text readable at all sizes
- No UI overlap at extreme scales

**Deliverables:**
- [Text and UI Scaling Specification](docs/TEXT_UI_SCALING_SPEC.md)

**Dependencies:** UX-008

---

### UX-020: Audio Accessibility
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Add audio accessibility features for hearing-impaired players.

**Features:**
- Subtitle system for voice/narration
- Visual cues for audio-only signals
- Vibration for audio events

**Acceptance Criteria:**
- [x] Subtitles for any voiced content
- [x] Visual ping when enemy fires nearby (directional)
- [x] Visual storm warning (not just audio)
- [x] Low health visual indicator (not just heartbeat)
- [x] Mono audio option
- [x] Vibration patterns for different events

**Definition of Done:**
- Deaf players can play effectively
- No critical info audio-only
- Visual indicators don't clutter HUD

**Deliverables:**
- [Audio Accessibility Specification](docs/AUDIO_ACCESSIBILITY_SPEC.md)

**Dependencies:** UX-011

---

### UX-021: Reduced Motion Mode
**Priority:** P3 | **Complexity:** S

**Description:**
Add option to reduce screen effects for players sensitive to motion or flashing.

**Acceptance Criteria:**
- [ ] Disable screen shake
- [ ] Reduce camera bob
- [ ] Limit flashy VFX
- [ ] Disable blood/gore effects
- [ ] Static UI elements (no bounce animations)

**Definition of Done:**
- Players with motion sensitivity can play comfortably
- Core gameplay unchanged

**Dependencies:** None

---

## 5. Platform Optimization

### UX-022: Device Performance Detection
**Priority:** P1 | **Complexity:** M

**Description:**
Automatically detect device capabilities and set optimal default settings.

**Detection Factors:**
- GPU model
- RAM
- Screen resolution
- Thermal state
- Battery level

**Acceptance Criteria:**
- [x] Device benchmark on first launch
- [x] Auto-set graphics quality (Low/Medium/High/Ultra)
- [x] Recommend FPS cap based on device
- [x] Warn if device below minimum specs
- [x] Thermal throttling detection and adjustment
- [x] Battery saver mode recommendation when low
- [x] Store device profiles for common devices

**Definition of Done:**
- 90%+ devices get optimal settings automatically
- No crashes from settings too high
- Performance smooth on detected settings

**Deliverable:** [Device Performance Detection Specification](docs/DEVICE_PERFORMANCE_DETECTION_SPEC.md)

**Status:** Completed

**Dependencies:** None

---

### UX-023: Battery Optimization
**Priority:** P2 | **Complexity:** S

**Description:**
Implement battery-saving features for extended mobile play sessions.

**Acceptance Criteria:**
- [ ] Battery saver mode (reduced FPS, lower graphics)
- [ ] Auto-activate when battery < 20%
- [ ] Show battery level in UI (optional)
- [ ] Reduce background processes when on battery
- [ ] Heat monitoring and throttling

**Definition of Done:**
- Noticeable battery life improvement in saver mode
- Game still playable at reduced settings
- No unexpected shutdowns

**Dependencies:** UX-022

---

### UX-024: Notch and Safe Area Handling
**Priority:** P0 | **Complexity:** S

**Description:**
Ensure UI respects device safe areas (notches, rounded corners, home indicators).

**Acceptance Criteria:**
- [x] UI elements inside safe area boundaries
- [x] No critical info hidden by notch
- [x] Works on iPhone (all notch types)
- [x] Works on Android (various punch-holes, cutouts)
- [x] Landscape both directions supported
- [x] Tablet layouts respected

**Definition of Done:**
- No UI occlusion on any device
- Tested on 20+ device configurations
- Dynamic safe area detection

**Deliverable:** [Notch and Safe Area Handling Specification](docs/NOTCH_SAFE_AREA_HANDLING_SPEC.md)

**Status:** Completed

**Dependencies:** None

---

### UX-025: Network Quality Indicator
**Priority:** P1 | **Complexity:** S

**Description:**
Display network quality to help players understand performance issues.

**Acceptance Criteria:**
- [x] Ping display (ms) in HUD (optional)
- [x] Connection quality icon (green/yellow/red bars)
- [x] Warning when connection poor
- [x] Packet loss indicator
- [x] Server region shown
- [x] Reconnection status when disconnected

**Definition of Done:**
- Players can diagnose connection issues
- Indicators accurate and real-time
- Reconnection works smoothly

**Deliverable:** [Network Quality Indicator Specification](docs/NETWORK_QUALITY_INDICATOR_SPEC.md)

**Status:** Completed

**Dependencies:** BACK-003

---

## 6. Onboarding and FTUE

### UX-026: First-Time User Experience (FTUE)
**Priority:** P1 | **Complexity:** L

**Description:**
Design the first-time user journey from app install to first match completion.

**FTUE Flow:**
1. Splash screen with pirate intro
2. Quick account creation (or guest play)
3. Character customization (basic)
4. Mandatory tutorial
5. First match (with bots, easier)
6. Reward and progression intro
7. Prompt to play more

**Acceptance Criteria:**
- [x] FTUE completable in < 10 minutes
- [x] Can skip tutorial after first time
- [x] Progress saved immediately (no lost progress on crash)
- [x] Rewarding feeling after tutorial (free cosmetic)
- [x] Introduces core loop naturally
- [x] No overwhelming info dumps
- [x] Guest play option (link account later)
- [x] Age gate if required by region

**Definition of Done:**
- Day 1 retention improved by FTUE
- 80%+ complete tutorial
- Players understand how to play after FTUE
- A/B tested for optimization

**Deliverable:** [First-Time User Experience Specification](docs/FIRST_TIME_USER_EXPERIENCE_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-024, BACK-005

---

### UX-027: Interactive Tutorial
**Priority:** P1 | **Complexity:** M

**Description:**
Create step-by-step interactive tutorial teaching core mechanics.

**Tutorial Steps:**
1. Movement controls
2. Camera controls
3. Basic attack
4. Opening chests
5. Picking up abilities
6. Using abilities
7. Storm warning
8. Eliminating an enemy
9. Victory (tutorial bot match)

**Acceptance Criteria:**
- [x] Each step has clear instruction
- [x] Action is gated until player performs it
- [x] Visual arrows/highlights guide player
- [x] Voice-over or text narration (pirate themed)
- [x] Can pause and resume tutorial
- [x] Skip option (for returning players)
- [x] Tutorial progress saved
- [x] Reward upon completion (gold, skin)

**Definition of Done:**
- New players competent after tutorial
- Tutorial is engaging, not boring
- No softlocks or progression bugs
- Tested with actual new players

**Deliverable:** [Interactive Tutorial Specification](docs/INTERACTIVE_TUTORIAL_SPEC.md)

**Status:** Completed

**Dependencies:** GAME-024

---

### UX-028: Contextual Hints System
**Priority:** P2 | **Complexity:** S

**Description:**
Display helpful hints during gameplay for new players without being intrusive for experienced players.

**Hint Triggers:**
- First time near a chest: "Tap to open chests"
- First ability found: "Abilities go in your slots"
- Storm approaching: "Get inside the safe zone!"
- Low health: "Find healing abilities"
- Enemy nearby: "Enemy spotted! Use abilities to attack"

**Acceptance Criteria:**
- [ ] Hints appear at appropriate moments
- [ ] Hints can be dismissed
- [ ] Hints can be disabled in settings
- [ ] Hints don't repeat excessively
- [ ] Hint text is concise
- [ ] Hints don't block important UI

**Definition of Done:**
- New players helped, veterans not annoyed
- Hints contextually appropriate
- System extensible for new features

**Dependencies:** UX-008

---

## Task Summary

| Task ID | Title | Priority | Complexity | Status |
|---------|-------|----------|------------|--------|
| UX-001 | Virtual Joystick Movement | P0 | M | Completed |
| UX-002 | Camera Control System | P0 | M | Completed |
| UX-003 | Ability Button Layout | P0 | M | Completed |
| UX-004 | Skillshot Targeting System | P0 | M | Completed |
| UX-005 | Auto-Aim Assist System | P1 | M | Completed |
| UX-006 | Gesture Controls | P2 | S | Completed |
| UX-007 | Controller Support | P3 | M | Not Started |
| UX-008 | Main HUD Layout | P0 | M | Completed |
| UX-009 | Minimap System | P0 | M | Completed |
| UX-010 | Ping/Communication System | P1 | M | Completed |
| UX-011 | Damage Feedback System | P0 | S | Completed |
| UX-012 | Spectator UI | P2 | S | Completed |
| UX-013 | Victory/Defeat Screen | P1 | S | Completed |
| UX-014 | Main Menu Design | P1 | M | Completed |
| UX-015 | Settings Menu | P0 | M | Completed |
| UX-016 | Mode Selection Screen | P1 | S | Completed |
| UX-017 | Loading Screens | P1 | S | Completed |
| UX-018 | Colorblind Modes | P2 | S | Completed |
| UX-019 | Text and UI Scaling | P2 | S | Completed |
| UX-020 | Audio Accessibility | P2 | S | Completed |
| UX-021 | Reduced Motion Mode | P3 | S | Not Started |
| UX-022 | Device Performance Detection | P1 | M | Completed |
| UX-023 | Battery Optimization | P2 | S | Not Started |
| UX-024 | Notch and Safe Area Handling | P0 | S | Completed |
| UX-025 | Network Quality Indicator | P1 | S | Completed |
| UX-026 | First-Time User Experience | P1 | L | Completed |
| UX-027 | Interactive Tutorial | P1 | M | Completed |
| UX-028 | Contextual Hints System | P2 | S | Not Started |
