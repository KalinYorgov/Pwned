# Character Animation Set Specification

## Document Information
- **Task ID:** ART-021
- **Priority:** P0
- **Complexity:** L (Large)
- **Dependencies:** ART-003 (Base Player Character Model)
- **Last Updated:** 2025-12-01

---

## 1. Overview

### 1.1 Purpose
This specification defines all player character animations for Plunderstorm Mobile. Animations must be responsive, readable, and performant on mobile devices while maintaining the swashbuckling pirate character.

### 1.2 Scope

```yaml
animation_scope:
  locomotion:
    - Idle variations
    - Walk/run cycles
    - Sprint
    - Jump/fall/land
    - Swim

  combat:
    - Basic attack combo
    - Hit reactions
    - Death
    - Revival (if applicable)

  interaction:
    - Loot pickup
    - Chest open
    - Item use

  emotes:
    - Victory poses
    - Basic communication (wave, point)
```

### 1.3 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Responsiveness | Animations feel snappy, not floaty | Critical |
| Readability | Clear silhouettes during action | Critical |
| Performance | Optimized for mobile (bone count, etc.) | Critical |
| Personality | Pirate swagger and attitude | High |
| Blending | Smooth transitions between states | High |
| Scalability | Support for cosmetic variations | Medium |

---

## 2. Animation Rig Specifications

### 2.1 Skeleton Structure

```yaml
skeleton:
  total_bones: 45
  hierarchy:
    root:
      name: "Root"
      children: ["Hips"]

    spine_chain:
      - "Hips"
      - "Spine"
      - "Spine1"
      - "Spine2"
      - "Chest"
      - "Neck"
      - "Head"

    left_arm:
      parent: "Chest"
      chain:
        - "L_Shoulder"
        - "L_UpperArm"
        - "L_Forearm"
        - "L_Hand"
        - "L_Fingers" (simplified: 1 bone per finger)

    right_arm:
      parent: "Chest"
      chain:
        - "R_Shoulder"
        - "R_UpperArm"
        - "R_Forearm"
        - "R_Hand"
        - "R_Fingers"

    left_leg:
      parent: "Hips"
      chain:
        - "L_UpperLeg"
        - "L_LowerLeg"
        - "L_Foot"
        - "L_Toe"

    right_leg:
      parent: "Hips"
      chain:
        - "R_UpperLeg"
        - "R_LowerLeg"
        - "R_Foot"
        - "R_Toe"

  extra_bones:
    - "Weapon_R" (attachment point)
    - "Weapon_L" (attachment point)
    - "Back_Attach" (backpack/cape)
    - "Hat_Attach" (headwear)
```

### 2.2 Technical Requirements

```yaml
technical_specs:
  bone_limits:
    mobile_low: 35 bones max
    mobile_high: 50 bones max
    active_bones_per_frame: "<40 recommended"

  skinning:
    max_influences: 4 per vertex
    recommended: 2-3 per vertex

  animation_format:
    type: "Skeletal animation"
    compression: "Keyframe reduction"
    sample_rate: 30 FPS

  lod_bones:
    LOD0: "Full skeleton (45)"
    LOD1: "Simplified (30)"
    LOD2: "Basic (20)"
```

---

## 3. Locomotion Animations

### 3.1 Idle Animations

```yaml
idle_base:
  name: "Idle_Base"
  duration: 4.0s
  loop: true

  description: |
    Relaxed standing pose with subtle breathing motion.
    Pirate swagger - slight weight shift, confident posture.
    Occasional glance around.

  key_poses:
    rest_pose:
      stance: "Feet shoulder-width, slight asymmetry"
      arms: "Relaxed at sides or hand on hip"
      spine: "Slight S-curve, casual lean"
      head: "Slight tilt, looking forward"

    breathing:
      chest_expansion: "2-3%"
      shoulder_rise: "Subtle"
      cycle: 3s

    weight_shift:
      frequency: "Every 4-6 seconds"
      amount: "Subtle hip sway"

  variations:
    idle_look_left:
      duration: 2.0s
      blend_in: 0.3s
      trigger: "Random (10% chance per 5s)"

    idle_look_right:
      duration: 2.0s
      blend_in: 0.3s

    idle_scratch:
      duration: 2.5s
      trigger: "After 15s of no input"

    idle_coin_flip:
      duration: 3.0s
      trigger: "After 30s of no input"
      prop: "Coin (optional)"
```

### 3.2 Walk Animation

```yaml
walk:
  name: "Walk_Forward"
  duration: 1.0s (one cycle)
  loop: true
  speed: 2.5 m/s

  description: |
    Casual walking with pirate swagger.
    Slight upper body sway for personality.
    Arms swing naturally.

  key_poses:
    contact_L:
      frame: 0
      left_foot: "Heel strike"
      right_foot: "Toe off"
      hips: "Centered"

    passing_L:
      frame: 7
      left_leg: "Supporting"
      right_leg: "Passing through"
      hips: "Shift to left"

    contact_R:
      frame: 15
      right_foot: "Heel strike"
      left_foot: "Toe off"
      hips: "Centered"

    passing_R:
      frame: 22
      right_leg: "Supporting"
      left_leg: "Passing through"
      hips: "Shift to right"

  upper_body:
    spine_twist: "Counter-rotate to hips"
    shoulder_rotation: "Follow arm swing"
    head: "Stable, slight bob"

  arm_swing:
    amplitude: 30°
    sync: "Opposite to legs"

  directional_variants:
    walk_backward:
      speed_modifier: 0.7
      description: "Cautious backward movement"

    walk_left:
      type: "Strafe"
      speed_modifier: 0.8

    walk_right:
      type: "Strafe"
      speed_modifier: 0.8
```

### 3.3 Run Animation

```yaml
run:
  name: "Run_Forward"
  duration: 0.67s (one cycle)
  loop: true
  speed: 5.0 m/s

  description: |
    Energetic running animation.
    More pronounced body lean forward.
    Aggressive arm pump.

  key_poses:
    flight_L:
      both_feet: "Off ground"
      body_lean: "15° forward"

    contact_L:
      left_foot: "Ball strike"
      right_leg: "Driving back"
      arms: "Counter-balanced"

    flight_R:
      both_feet: "Off ground"
      mirror: flight_L

    contact_R:
      mirror: contact_L

  characteristics:
    body_lean: 15°
    arm_swing: 45°
    vertical_bounce: "Moderate"
    hip_rotation: "More pronounced than walk"

  directional_variants:
    run_backward:
      speed_modifier: 0.6
      description: "Quick backward retreat"

    run_strafe_L:
      speed_modifier: 0.85
      body_orientation: "Face forward, move left"

    run_strafe_R:
      speed_modifier: 0.85
      body_orientation: "Face forward, move right"
```

### 3.4 Sprint Animation

```yaml
sprint:
  name: "Sprint"
  duration: 0.55s (one cycle)
  loop: true
  speed: 7.0 m/s

  description: |
    Maximum speed running.
    Heavy forward lean.
    Arms pumping hard.
    Full-out pirate dash.

  differences_from_run:
    body_lean: "25° forward"
    arm_swing: "60° amplitude"
    stride_length: "+30%"
    vertical_amplitude: "+20%"
    breathing: "Heavy, visible"

  transition:
    from_run: 0.2s blend
    to_run: 0.2s blend
```

### 3.5 Jump Animations

```yaml
jump_set:
  jump_start:
    name: "Jump_Takeoff"
    duration: 0.25s
    loop: false

    description: |
      Crouch and spring upward.
      Arms swing up for momentum.

    phases:
      anticipation:
        duration: 0.1s
        crouch_depth: "15% height"
        arms: "Pull back"

      launch:
        duration: 0.15s
        leg_extension: "Full"
        arms: "Swing up"
        body: "Extend upward"

  jump_air:
    name: "Jump_Airborne"
    duration: 0.5s
    loop: true

    description: |
      Airborne pose with slight movements.
      Arms out for balance.
      Legs slightly tucked.

    pose:
      legs: "Slightly bent, trailing"
      arms: "Out to sides for balance"
      spine: "Slight arch"

    motion:
      type: "Subtle floating motion"
      intensity: "Minimal"

  jump_apex:
    name: "Jump_Peak"
    duration: 0.2s
    loop: false

    description: |
      Brief moment at jump apex.
      Extended pose.

  jump_fall:
    name: "Jump_Falling"
    duration: 0.4s
    loop: true

    description: |
      Falling animation.
      Arms may windmill slightly.
      Legs prepare for landing.

    pose:
      legs: "Extending for landing"
      arms: "Raised for balance"
      body: "Slightly leaned forward"

  jump_land:
    name: "Jump_Landing"
    duration: 0.3s
    loop: false

    description: |
      Impact absorption.
      Crouch on landing.
      Quick recovery.

    phases:
      impact:
        duration: 0.1s
        crouch: "Deep knee bend"
        arms: "Forward for balance"

      recovery:
        duration: 0.2s
        return_to: "Idle or locomotion"

    variants:
      land_soft:
        trigger: "Fall distance < 2m"
        recovery: 0.2s

      land_hard:
        trigger: "Fall distance > 4m"
        recovery: 0.4s
        stagger: true
```

### 3.6 Swimming Animations

```yaml
swimming:
  swim_idle:
    name: "Swim_Idle"
    duration: 2.0s
    loop: true

    description: |
      Treading water.
      Arms making small circular motions.
      Legs kicking gently.

  swim_forward:
    name: "Swim_Forward"
    duration: 1.2s
    loop: true
    speed: 3.0 m/s

    description: |
      Front crawl swimming motion.
      Alternating arm strokes.
      Flutter kick.

    phases:
      stroke_L:
        left_arm: "Pull through water"
        right_arm: "Recovery above"
        body_roll: "Slight left"

      stroke_R:
        mirror: stroke_L

  swim_surface:
    name: "Swim_Surface"
    duration: 0.5s
    loop: false

    description: |
      Emerging from underwater.
      Head breaks surface.
      Water splash effect trigger.

  swim_dive:
    name: "Swim_Dive"
    duration: 0.4s
    loop: false

    description: |
      Diving underwater.
      Head goes under.
      Body follows.
```

---

## 4. Combat Animations

### 4.1 Basic Attack Combo

```yaml
basic_attack_combo:
  attack_1:
    name: "Attack_Light_1"
    duration: 0.5s
    damage_frame: 0.2s

    description: |
      Quick horizontal slash.
      Fast startup, moderate recovery.

    phases:
      windup:
        duration: 0.15s
        weapon_position: "Pulled back right"
        body: "Slight right rotation"

      swing:
        duration: 0.15s
        weapon_arc: "Right to left horizontal"
        body: "Rotate into swing"

      recovery:
        duration: 0.2s
        weapon: "Return to ready"
        cancel_window: "0.15s into recovery"

  attack_2:
    name: "Attack_Light_2"
    duration: 0.55s
    damage_frame: 0.25s

    description: |
      Backhand swing following attack_1.
      Slightly slower but more powerful look.

    phases:
      windup:
        duration: 0.18s
        weapon_position: "Left side from attack_1"
        body: "Continue rotation momentum"

      swing:
        duration: 0.15s
        weapon_arc: "Left to right diagonal"
        body: "Rotate with swing"

      recovery:
        duration: 0.22s
        weapon: "Return to ready"
        cancel_window: "0.17s into recovery"

  attack_3:
    name: "Attack_Light_3"
    duration: 0.7s
    damage_frame: 0.35s

    description: |
      Heavy overhead finisher.
      Longest windup, satisfying impact.

    phases:
      windup:
        duration: 0.25s
        weapon_position: "Raised overhead"
        body: "Rise up on toes slightly"

      swing:
        duration: 0.2s
        weapon_arc: "Overhead downward"
        body: "Commit into strike"

      recovery:
        duration: 0.25s
        weapon: "Pull up from ground"
        full_recovery: true

  combo_timing:
    attack_1_to_2: "Cancel window: 0.35-0.5s"
    attack_2_to_3: "Cancel window: 0.38-0.55s"
    reset_time: "0.8s of no input"
```

### 4.2 Hit Reactions

```yaml
hit_reactions:
  hit_light_front:
    name: "Hit_Light_F"
    duration: 0.3s

    description: |
      Small flinch from frontal hit.
      Upper body recoil.
      Quick recovery to not interrupt gameplay.

    motion:
      upper_body: "Jerk backward 5°"
      head: "Snap back"
      recovery: "Quick return"

  hit_light_back:
    name: "Hit_Light_B"
    duration: 0.3s

    description: |
      Small flinch from rear hit.
      Stumble forward slightly.

    motion:
      upper_body: "Jerk forward 5°"
      step: "Small forward"

  hit_light_left:
    name: "Hit_Light_L"
    duration: 0.3s

    motion:
      upper_body: "Jerk right"
      shoulder: "Left shoulder dip"

  hit_light_right:
    name: "Hit_Light_R"
    duration: 0.3s

    motion:
      upper_body: "Jerk left"
      shoulder: "Right shoulder dip"

  hit_heavy:
    name: "Hit_Heavy"
    duration: 0.5s

    description: |
      Significant stagger from heavy hit.
      Nearly loses balance.

    motion:
      full_body: "Stagger backward"
      steps: 2
      arms: "Flail for balance"

  hit_knockdown:
    name: "Hit_Knockdown"
    duration: 1.2s

    description: |
      Knocked to ground.
      Used for special abilities.

    phases:
      fall:
        duration: 0.4s
        type: "Fall backward"

      ground:
        duration: 0.5s
        type: "Lie on ground"

      getup:
        duration: 0.3s
        type: "Quick scramble up"
```

### 4.3 Death Animation

```yaml
death:
  death_main:
    name: "Death_Default"
    duration: 1.5s
    loop: false

    description: |
      Dramatic death animation.
      Stagger, spin, fall.
      Ends in ragdoll or final pose.

    phases:
      reaction:
        duration: 0.3s
        motion: "Clutch wound area"
        expression: "Pain/surprise"

      fall:
        duration: 0.5s
        type: "Spin and collapse"
        direction: "Opposite to damage source"

      ground:
        duration: 0.4s
        type: "Hit ground, bounce slightly"

      final:
        duration: 0.3s
        type: "Settle into death pose"
        trigger: "Death VFX"

  death_variants:
    death_forward:
      direction: "Fall forward"

    death_backward:
      direction: "Fall backward"

    death_explosion:
      type: "Blown back"
      trigger: "Explosive damage"

  post_death:
    type: "Ragdoll or static pose"
    duration: "Until despawn (5s)"
```

---

## 5. Interaction Animations

### 5.1 Loot Pickup

```yaml
loot_pickup:
  pickup_ground:
    name: "Pickup_Ground"
    duration: 0.6s
    interruptible: true

    description: |
      Quick bend to grab item from ground.
      One-handed scoop motion.

    phases:
      reach:
        duration: 0.25s
        bend: "Quick squat"
        arm: "Right arm reaches down"

      grab:
        duration: 0.1s
        hand: "Close on item"
        trigger: "Item pickup event"

      recover:
        duration: 0.25s
        return_to: "Standing"

  pickup_air:
    name: "Pickup_Air"
    duration: 0.3s

    description: |
      Grab floating item (ability orb).
      Quick snatch motion.

    motion:
      arm: "Quick reach and grab"
      body: "Minimal movement"
```

### 5.2 Chest Open

```yaml
chest_interaction:
  chest_open:
    name: "Chest_Open"
    duration: 0.8s
    interruptible: false

    description: |
      Kneel and open treasure chest.
      Dramatic lid flip.
      Reaction to contents.

    phases:
      approach:
        duration: 0.2s
        motion: "Kneel at chest"
        hands: "Grip lid"

      open:
        duration: 0.3s
        motion: "Flip lid open"
        sync: "Chest animation"

      reveal:
        duration: 0.3s
        motion: "Look at contents"
        expression: "Excitement"
        trigger: "Loot spawn"
```

### 5.3 Ability Use Poses

```yaml
ability_poses:
  cast_instant:
    name: "Cast_Instant"
    duration: 0.3s

    description: |
      Quick hand gesture for instant abilities.
      Minimal interruption to movement.

    motion:
      arm: "Quick thrust forward"
      hand: "Open palm"

  cast_channel:
    name: "Cast_Channel"
    duration: "Variable"
    loop: true

    description: |
      Sustained casting pose.
      Arms out, energy flowing.

    motion:
      arms: "Extended forward or sides"
      body: "Slight lean"
      loop_motion: "Subtle energy pulse"

  cast_throw:
    name: "Cast_Throw"
    duration: 0.5s

    description: |
      Throwing motion for projectiles.
      Wind up and release.

    phases:
      windup:
        duration: 0.2s
        arm: "Pull back"
        body: "Rotate back"

      throw:
        duration: 0.15s
        arm: "Whip forward"
        trigger: "Projectile spawn"

      follow_through:
        duration: 0.15s
        arm: "Complete motion"
```

---

## 6. Animation Blending

### 6.1 Blend Trees

```yaml
locomotion_blend_tree:
  type: "2D Freeform Directional"

  parameters:
    - name: "MoveX"
      range: [-1, 1]
    - name: "MoveY"
      range: [-1, 1]

  poses:
    center: "Idle"
    north: "Run_Forward"
    south: "Run_Backward"
    east: "Run_Right"
    west: "Run_Left"
    northeast: "Run_Forward_Right"
    northwest: "Run_Forward_Left"
    southeast: "Run_Backward_Right"
    southwest: "Run_Backward_Left"

speed_blend:
  type: "1D Blend"

  parameter:
    - name: "Speed"
      range: [0, 7]

  thresholds:
    0.0: "Idle"
    2.5: "Walk"
    5.0: "Run"
    7.0: "Sprint"
```

### 6.2 Layer System

```yaml
animation_layers:
  base_layer:
    name: "Locomotion"
    weight: 1.0
    mask: "Full body"
    contents:
      - "Idle"
      - "Walk/Run"
      - "Jump"
      - "Swim"

  upper_body_layer:
    name: "UpperBody"
    weight: "Variable"
    mask: "Spine and above"
    blend_mode: "Override or Additive"
    contents:
      - "Combat attacks"
      - "Ability casting"
      - "Interactions"

  additive_layer:
    name: "Additives"
    weight: "Variable"
    blend_mode: "Additive"
    contents:
      - "Hit reactions"
      - "Breathing"
      - "Head look"
```

### 6.3 Transition Rules

```yaml
transitions:
  default_blend_time: 0.15s

  specific_transitions:
    idle_to_walk:
      blend_time: 0.2s
      curve: "EaseIn"

    walk_to_run:
      blend_time: 0.15s
      curve: "Linear"

    run_to_idle:
      blend_time: 0.25s
      curve: "EaseOut"

    any_to_jump:
      blend_time: 0.1s
      interrupt: true

    any_to_death:
      blend_time: 0.05s
      interrupt: true

    attack_chain:
      blend_time: 0.08s
      sync: "Normalized time"
```

---

## 7. Technical Specifications

### 7.1 Performance Budget

```yaml
performance:
  bones_per_character:
    target: 45
    LOD1: 30
    LOD2: 20

  characters_on_screen:
    max: 20
    typical: 10

  animation_updates:
    full_rate: "Characters within 30m"
    half_rate: "Characters 30-60m"
    quarter_rate: "Characters beyond 60m"

  blend_operations:
    max_per_character: 3 layers
    max_blend_time: 0.3s

  memory_per_clip:
    target: "< 100KB compressed"
    locomotion: "50-80KB"
    combat: "30-60KB"
```

### 7.2 Export Settings

```yaml
export:
  format: "FBX"
  version: "2020"

  settings:
    bake_animation: true
    sample_rate: 30
    compression: "Keyframe Reduction"
    compression_ratio: "Lossy 0.5"

  naming:
    character: "CH_Player"
    animation: "Anim_[Category]_[Name]"

  file_organization:
    locomotion: "Animations/Locomotion/"
    combat: "Animations/Combat/"
    interaction: "Animations/Interaction/"
    emotes: "Animations/Emotes/"
```

---

## 8. Asset Deliverables

### 8.1 Animation List

```yaml
deliverables:
  locomotion:
    - "Anim_Idle_Base.fbx"
    - "Anim_Idle_Variation_1.fbx"
    - "Anim_Idle_Variation_2.fbx"
    - "Anim_Walk_Forward.fbx"
    - "Anim_Walk_Backward.fbx"
    - "Anim_Walk_Left.fbx"
    - "Anim_Walk_Right.fbx"
    - "Anim_Run_Forward.fbx"
    - "Anim_Run_Backward.fbx"
    - "Anim_Run_Left.fbx"
    - "Anim_Run_Right.fbx"
    - "Anim_Sprint.fbx"
    - "Anim_Jump_Start.fbx"
    - "Anim_Jump_Air.fbx"
    - "Anim_Jump_Land.fbx"
    - "Anim_Swim_Idle.fbx"
    - "Anim_Swim_Forward.fbx"

  combat:
    - "Anim_Attack_1.fbx"
    - "Anim_Attack_2.fbx"
    - "Anim_Attack_3.fbx"
    - "Anim_Hit_Light_F.fbx"
    - "Anim_Hit_Light_B.fbx"
    - "Anim_Hit_Light_L.fbx"
    - "Anim_Hit_Light_R.fbx"
    - "Anim_Hit_Heavy.fbx"
    - "Anim_Death.fbx"

  interaction:
    - "Anim_Pickup_Ground.fbx"
    - "Anim_Pickup_Air.fbx"
    - "Anim_Chest_Open.fbx"
    - "Anim_Cast_Instant.fbx"
    - "Anim_Cast_Channel.fbx"
    - "Anim_Cast_Throw.fbx"

  controller:
    - "AnimController_Player.controller"
```

---

## 9. Acceptance Criteria Verification

| Criteria | Specification | Verification Method |
|----------|---------------|---------------------|
| All locomotion animations implemented | Walk, run, sprint, jump, swim | Animation playback test |
| 3-hit combo system working | Smooth combo chains | Combat test |
| Animations blend smoothly | No popping or snapping | Transition test |
| Performance targets met | 60 FPS with 10+ players | Profiler verification |
| Pirate personality visible | Swagger in movement | Art Director review |
| Responsive feel | Input to animation < 0.1s | Input latency test |

---

## 10. Implementation Checklist

```yaml
checklist:
  locomotion:
    - [ ] Idle base + variations
    - [ ] Walk (all directions)
    - [ ] Run (all directions)
    - [ ] Sprint
    - [ ] Jump set (start, air, land)
    - [ ] Swim set

  combat:
    - [ ] Basic attack combo (3 hits)
    - [ ] Hit reactions (all directions)
    - [ ] Death animation

  interaction:
    - [ ] Pickup animations
    - [ ] Chest open
    - [ ] Ability poses

  technical:
    - [ ] Blend trees configured
    - [ ] Layer system working
    - [ ] Transitions smooth
    - [ ] LOD system implemented
    - [ ] Mobile performance verified
```

---

## 11. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-01 | Initial specification |

---

*This document defines the complete character animation set for Plunderstorm Mobile, ensuring responsive, readable, and personality-filled movement.*
