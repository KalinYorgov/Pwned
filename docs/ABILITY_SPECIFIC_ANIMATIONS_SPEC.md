# Ability-Specific Animations Specification

## Document Information
- **Task ID:** ART-022
- **Priority:** P1
- **Complexity:** M
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines unique character animations for signature abilities in Plunderstorm Mobile. These animations replace generic casting poses with ability-specific movements that enhance visual feedback, player satisfaction, and ability recognition.

### 1.2 Design Goals
- **Visual Distinction:** Each highlighted ability has recognizable animation
- **VFX Synchronization:** Animations perfectly timed with visual effects
- **Smooth Transitions:** Seamless blend to and from locomotion
- **Responsive Feel:** Animation canceling feels natural and fair
- **Mobile Performance:** Animations optimized for 30/60 FPS targets

### 1.3 Animation Philosophy
- Exaggerated, readable movements for mobile screens
- Clear anticipation, action, and recovery phases
- Silhouette-focused design for instant recognition
- Consistent timing that supports gameplay responsiveness

---

## 2. Animation Framework

### 2.1 Animation Phase Structure

```typescript
const ANIMATION_PHASES = {
  // Standard ability animation structure
  phases: {
    anticipation: {
      purpose: 'Wind-up/preparation',
      duration: '15-25%', // Of total animation
      blendIn: true,
      cancelable: true
    },
    action: {
      purpose: 'Main ability execution',
      duration: '40-60%',
      vfxSync: true,
      cancelable: false // Commitment point
    },
    recovery: {
      purpose: 'Follow-through',
      duration: '20-35%',
      blendOut: true,
      cancelable: true // Early exit allowed
    }
  },

  timing: {
    minimum: 0.3, // Seconds
    maximum: 1.5,
    recommended: 0.5 // For responsive feel
  }
};
```

### 2.2 Blend Configuration

```typescript
const BLEND_CONFIG = {
  intoAbility: {
    blendTime: 0.1, // Seconds
    priority: 'ability_high',
    interruptible: true
  },

  outOfAbility: {
    blendTime: 0.15,
    targetState: 'locomotion_current',
    smoothing: 'ease_out'
  },

  abilityChain: {
    enabled: true,
    minGap: 0.05, // Seconds between abilities
    blendOverlap: 0.1
  },

  movementBlend: {
    allowMoveDuring: 'ability_specific', // Some abilities allow movement
    upperBodyOverride: true,
    lowerBodyMaintain: 'locomotion'
  }
};
```

---

## 3. Barrel Roll Animation

### 3.1 Animation Overview

```typescript
const BARREL_ROLL_ANIMATION = {
  abilityName: 'Barrel Roll',
  abilityType: 'mobility',
  totalDuration: 0.6, // Seconds

  description: 'Character tucks into a barrel and rolls forward, ' +
               'becoming temporarily invulnerable',

  phases: {
    anticipation: {
      duration: 0.1,
      keyframes: [
        { time: 0, pose: 'standing' },
        { time: 0.05, pose: 'crouch_begin' },
        { time: 0.1, pose: 'tucked_ready' }
      ],
      bodyParts: {
        torso: 'lean_forward_45deg',
        arms: 'crossed_chest',
        legs: 'bent_ready'
      }
    },

    action: {
      duration: 0.35,
      type: 'full_body_roll',
      rotations: 1.5, // Full rotations
      trajectory: 'forward_arc',
      barrelAppear: {
        timing: 0.05, // After action start
        type: 'spawn_around_character',
        material: 'wooden_barrel_rings'
      },
      keyframes: [
        { time: 0, pose: 'roll_entry' },
        { time: 0.12, pose: 'roll_mid_1' },
        { time: 0.24, pose: 'roll_mid_2' },
        { time: 0.35, pose: 'roll_exit' }
      ]
    },

    recovery: {
      duration: 0.15,
      keyframes: [
        { time: 0, pose: 'emerging_crouch' },
        { time: 0.08, pose: 'standing_momentum' },
        { time: 0.15, pose: 'idle_ready' }
      ],
      barrelDisappear: {
        timing: 0, // At recovery start
        type: 'burst_apart',
        debris: true
      }
    }
  },

  rootMotion: {
    enabled: true,
    distance: 8, // Meters forward
    curve: 'ease_in_out',
    height: {
      max: 0.3, // Slight hop at peak
      curve: 'parabolic'
    }
  }
};
```

### 3.2 Barrel Roll Technical Details

```typescript
const BARREL_ROLL_TECHNICAL = {
  boneAnimations: {
    spine: {
      rotation: { x: 360, y: 0, z: 0 }, // Forward roll
      during: 'action_phase'
    },
    arms: {
      wrap: 'around_body',
      position: 'tucked'
    },
    head: {
      follow: 'spine_rotation',
      damping: 0.8
    },
    legs: {
      position: 'tucked_knees_to_chest'
    }
  },

  additiveAnimations: {
    barrelSpin: {
      separate: true,
      syncTo: 'body_rotation',
      wobble: 0.1
    }
  },

  vfxTiming: {
    dustCloud: { start: 0, duration: 0.6 },
    barrelAppear: { start: 0.05, duration: 0.45 },
    barrelBreak: { start: 0.5, duration: 0.2 }
  },

  sfxTiming: {
    rollWhoosh: { start: 0, duration: 0.5 },
    barrelCreak: { start: 0.1, duration: 0.3 },
    barrelBurst: { start: 0.5, duration: 0.15 }
  }
};
```

---

## 4. Grappling Hook Animation

### 4.1 Animation Overview

```typescript
const GRAPPLING_HOOK_ANIMATION = {
  abilityName: 'Grappling Hook',
  abilityType: 'mobility',
  totalDuration: 0.8, // Variable based on distance

  description: 'Character throws a grappling hook and pulls themselves ' +
               'rapidly to the target location',

  phases: {
    anticipation: {
      duration: 0.15,
      keyframes: [
        { time: 0, pose: 'standing' },
        { time: 0.08, pose: 'wind_up_arm_back' },
        { time: 0.15, pose: 'throw_ready' }
      ],
      bodyParts: {
        dominantArm: 'pulled_back_overhead',
        torso: 'twisted_throwing',
        offArm: 'balance_forward',
        legs: 'wide_stance'
      },
      hookVisible: {
        timing: 0,
        position: 'in_hand'
      }
    },

    throw: {
      duration: 0.1,
      type: 'overhand_throw',
      keyframes: [
        { time: 0, pose: 'throw_ready' },
        { time: 0.05, pose: 'throw_release' },
        { time: 0.1, pose: 'throw_follow_through' }
      ],
      hookRelease: {
        timing: 0.05,
        trajectory: 'aim_direction',
        ropeTrail: true
      }
    },

    flight: {
      duration: 'dynamic', // Based on distance
      minDuration: 0.2,
      maxDuration: 0.6,
      type: 'pulled_forward',
      keyframes: [
        { time: 0, pose: 'rope_grab' },
        { time: 0.5, pose: 'flying_forward' }, // Normalized
        { time: 1.0, pose: 'approaching_target' }
      ],
      bodyParts: {
        dominantArm: 'extended_gripping_rope',
        torso: 'leaning_forward',
        offArm: 'trailing_back',
        legs: 'streaming_behind'
      }
    },

    recovery: {
      duration: 0.15,
      keyframes: [
        { time: 0, pose: 'landing_impact' },
        { time: 0.08, pose: 'absorb_momentum' },
        { time: 0.15, pose: 'ready_stance' }
      ],
      hookRetract: {
        timing: 0,
        speed: 'fast'
      }
    }
  },

  rootMotion: {
    enabled: true,
    distance: 'dynamic', // To target
    maxDistance: 25, // Meters
    curve: 'accelerate_then_decelerate',
    height: {
      arcType: 'straight_line', // Direct path
      groundClearance: 1.0
    }
  }
};
```

### 4.2 Grappling Hook Technical Details

```typescript
const GRAPPLING_HOOK_TECHNICAL = {
  ikTargets: {
    dominantHand: {
      target: 'rope_attachment_point',
      weight: 1.0,
      during: ['throw_follow_through', 'flight', 'landing']
    }
  },

  ropeSimulation: {
    type: 'spline_rope',
    segments: 12,
    stiffness: 0.8,
    damping: 0.3,
    thickness: 0.05
  },

  hookProp: {
    model: 'grappling_hook_prop',
    attachPoint: 'hand_r',
    releaseEvent: 'throw_release',
    returnEvent: 'recovery_start'
  },

  vfxTiming: {
    throwTrail: { start: 0.15, duration: 'until_hit' },
    hookImpact: { start: 'on_hit', duration: 0.2 },
    ropeGlow: { start: 0.2, duration: 'flight_duration' },
    landingDust: { start: 'on_land', duration: 0.3 }
  },

  sfxTiming: {
    throwWhoosh: { start: 0.12, duration: 0.15 },
    ropeUnwind: { start: 0.15, duration: 0.3 },
    hookClang: { start: 'on_hit', duration: 0.1 },
    pullSound: { start: 'flight_start', duration: 'flight_duration' },
    landThud: { start: 'on_land', duration: 0.15 }
  }
};
```

---

## 5. Cutlass Slash Animation

### 5.1 Animation Overview

```typescript
const CUTLASS_SLASH_ANIMATION = {
  abilityName: 'Cutlass Slash',
  abilityType: 'melee_damage',
  totalDuration: 0.7, // For full combo

  description: 'Three-hit melee combo with cutlass, ' +
               'each swing dealing damage in an arc',

  comboStructure: {
    hits: 3,
    timings: [0.23, 0.23, 0.24], // Duration per hit
    canChain: true,
    chainWindow: 0.15 // Seconds after hit
  },

  slashes: {
    slash1: {
      name: 'horizontal_right',
      duration: 0.23,
      phases: {
        anticipation: {
          duration: 0.06,
          pose: 'sword_chambered_right'
        },
        action: {
          duration: 0.1,
          arc: {
            direction: 'right_to_left',
            angle: 120, // Degrees
            height: 'chest_level'
          }
        },
        recovery: {
          duration: 0.07,
          pose: 'sword_left_follow_through'
        }
      },
      damage: {
        timing: 0.08, // Into action phase
        hitbox: 'horizontal_arc'
      }
    },

    slash2: {
      name: 'horizontal_left',
      duration: 0.23,
      phases: {
        anticipation: {
          duration: 0.05,
          pose: 'sword_chambered_left'
        },
        action: {
          duration: 0.1,
          arc: {
            direction: 'left_to_right',
            angle: 120,
            height: 'chest_level'
          }
        },
        recovery: {
          duration: 0.08,
          pose: 'sword_overhead_ready'
        }
      },
      damage: {
        timing: 0.07,
        hitbox: 'horizontal_arc'
      }
    },

    slash3: {
      name: 'overhead_slam',
      duration: 0.24,
      phases: {
        anticipation: {
          duration: 0.08,
          pose: 'sword_raised_overhead'
        },
        action: {
          duration: 0.08,
          arc: {
            direction: 'top_to_bottom',
            angle: 90,
            height: 'overhead_to_ground'
          }
        },
        recovery: {
          duration: 0.08,
          pose: 'sword_low_finish',
          longerIfComboEnd: true
        }
      },
      damage: {
        timing: 0.06,
        hitbox: 'vertical_slam',
        bonusDamage: 1.3 // Finisher bonus
      }
    }
  },

  rootMotion: {
    enabled: true,
    perSlash: [
      { distance: 0.5, direction: 'forward' },
      { distance: 0.3, direction: 'forward' },
      { distance: 0.8, direction: 'forward_lunge' }
    ]
  }
};
```

### 5.2 Cutlass Slash Technical Details

```typescript
const CUTLASS_SLASH_TECHNICAL = {
  weaponTrail: {
    enabled: true,
    startBone: 'weapon_base',
    endBone: 'weapon_tip',
    trailDuration: 0.15,
    color: {
      base: '#FFFFFF',
      edge: '#ADD8E6'
    },
    emission: 0.5
  },

  boneAnimations: {
    slash1: {
      spine: { rotation: { y: -30 }, then: { y: 30 } },
      shoulder_r: { followArc: true },
      wrist_r: { snap: 'end_of_arc' }
    },
    slash2: {
      spine: { rotation: { y: 30 }, then: { y: -30 } },
      shoulder_r: { followArc: true },
      wrist_r: { snap: 'end_of_arc' }
    },
    slash3: {
      spine: { rotation: { x: -20 }, then: { x: 30 } },
      shoulder_r: { overhead: true },
      knees: { bend: 15 } // Power stance
    }
  },

  vfxTiming: {
    slashArc: { perSlash: true, during: 'action', duration: 0.12 },
    hitSparks: { onDamage: true, duration: 0.1 },
    groundImpact: { slash3Only: true, duration: 0.2 }
  },

  sfxTiming: {
    swordSwing: { perSlash: true, start: 'action', duration: 0.1 },
    hitFlesh: { onDamage: true, duration: 0.08 },
    slam: { slash3Only: true, duration: 0.15 }
  },

  comboIndicators: {
    visual: 'glow_intensity_increase',
    audio: 'pitch_increase',
    screenShake: [0, 0.5, 1.5] // Per hit
  }
};
```

---

## 6. Healing Grog Animation

### 6.1 Animation Overview

```typescript
const HEALING_GROG_ANIMATION = {
  abilityName: 'Healing Grog',
  abilityType: 'self_heal',
  totalDuration: 1.2, // Seconds (channeled)

  description: 'Character takes a swig of magical healing grog, ' +
               'restoring health over time',

  interruptible: true,
  channelType: 'drink',

  phases: {
    anticipation: {
      duration: 0.15,
      keyframes: [
        { time: 0, pose: 'standing' },
        { time: 0.08, pose: 'reach_for_bottle' },
        { time: 0.15, pose: 'bottle_in_hand' }
      ],
      bottleSpawn: {
        timing: 0.05,
        position: 'hip_pouch',
        animation: 'pull_out'
      }
    },

    drink: {
      duration: 0.8,
      type: 'channeled_loop',
      loopCount: 3, // Number of gulps
      gulpDuration: 0.267, // Per gulp
      keyframes: [
        { time: 0, pose: 'bottle_to_mouth' },
        { time: 0.13, pose: 'head_back_drinking' },
        { time: 0.267, pose: 'gulp_complete' }
      ],
      bodyParts: {
        dominantArm: 'bottle_raised_to_mouth',
        head: 'tilted_back_15deg',
        throat: 'gulping_motion',
        offArm: 'relaxed_side'
      },
      healTick: {
        perGulp: true,
        timing: 0.2 // Into each gulp
      }
    },

    recovery: {
      duration: 0.25,
      keyframes: [
        { time: 0, pose: 'drinking_finish' },
        { time: 0.1, pose: 'satisfied_exhale' },
        { time: 0.18, pose: 'bottle_away' },
        { time: 0.25, pose: 'ready_stance' }
      ],
      bottleDespawn: {
        timing: 0.15,
        animation: 'tuck_away'
      },
      facialExpression: 'satisfied_ahh'
    }
  },

  movementAllowed: {
    during: 'drink',
    speedMultiplier: 0.5 // Slowed while drinking
  }
};
```

### 6.2 Healing Grog Technical Details

```typescript
const HEALING_GROG_TECHNICAL = {
  propHandling: {
    bottle: {
      model: 'healing_grog_bottle',
      attachPoint: 'hand_r',
      spawnEvent: 'anticipation_0.05',
      despawnEvent: 'recovery_0.15'
    },
    liquid: {
      visible: true,
      color: '#4CAF50', // Healing green
      glow: true,
      decreasePerGulp: true
    }
  },

  additiveAnimations: {
    throatGulp: {
      bone: 'neck',
      animation: 'gulp_bob',
      frequency: 0.267
    },
    bottleTilt: {
      increasing: true,
      perGulp: 20 // Degrees
    }
  },

  vfxTiming: {
    bottleGlow: { start: 0, duration: 'channelDuration' },
    healParticles: { perGulp: true, duration: 0.3 },
    satisfiedBurst: { start: 'recovery', duration: 0.2 },
    steamBreath: { start: 'recovery_0.1', duration: 0.15 }
  },

  sfxTiming: {
    bottleUncork: { start: 0, duration: 0.1 },
    gulp: { perGulp: true, duration: 0.2 },
    satisfiedAhh: { start: 'recovery_0.1', duration: 0.3 },
    bottleAway: { start: 'recovery_0.15', duration: 0.1 }
  },

  interruptBehavior: {
    animation: 'drink_interrupted',
    bottleHandling: 'quick_stash',
    partialHeal: true // Heal for gulps completed
  }
};
```

---

## 7. Fireball Animation

### 7.1 Animation Overview

```typescript
const FIREBALL_ANIMATION = {
  abilityName: 'Fireball',
  abilityType: 'ranged_damage',
  totalDuration: 0.5,

  description: 'Character conjures and hurls a blazing fireball',

  phases: {
    anticipation: {
      duration: 0.15,
      keyframes: [
        { time: 0, pose: 'standing' },
        { time: 0.08, pose: 'arm_gathering_back' },
        { time: 0.15, pose: 'fireball_charged' }
      ],
      fireCharge: {
        timing: 0.05,
        buildUp: 'swirling_flames_in_palm',
        growthCurve: 'exponential'
      }
    },

    throw: {
      duration: 0.15,
      type: 'overhand_throw',
      keyframes: [
        { time: 0, pose: 'throw_windup' },
        { time: 0.08, pose: 'throw_release' },
        { time: 0.15, pose: 'throw_follow_through' }
      ],
      projectileRelease: {
        timing: 0.08,
        fromBone: 'hand_r',
        velocity: 'aim_direction'
      }
    },

    recovery: {
      duration: 0.2,
      keyframes: [
        { time: 0, pose: 'extended_arm' },
        { time: 0.12, pose: 'arm_returning' },
        { time: 0.2, pose: 'ready_stance' }
      ],
      residualFlames: {
        enabled: true,
        duration: 0.15,
        onHand: true
      }
    }
  },

  bodyLanguage: {
    dominantArm: 'throwing_motion',
    torso: 'rotation_with_throw',
    offArm: 'balance_counter',
    legs: 'power_stance_step'
  }
};
```

### 7.2 Fireball Technical Details

```typescript
const FIREBALL_TECHNICAL = {
  fireBuildup: {
    startSize: 0.1,
    endSize: 0.5,
    particles: {
      count: 30,
      swirl: true,
      converge: 'palm_center'
    },
    lightSource: {
      enabled: true,
      color: '#FF6600',
      intensity: { start: 0.5, peak: 2.0 },
      range: 3
    }
  },

  throwMechanics: {
    armRotation: 180, // Degrees
    releaseAngle: 'aim_pitch',
    followThrough: 30 // Degrees past release
  },

  vfxTiming: {
    chargeSwirl: { start: 0.05, duration: 0.1 },
    fireballForm: { start: 0.1, duration: 0.05 },
    throwTrail: { start: 0.23, duration: 'until_impact' },
    handEmbers: { start: 0.35, duration: 0.15 }
  },

  sfxTiming: {
    fireCharge: { start: 0.05, duration: 0.15 },
    fireballWhoosh: { start: 0.23, duration: 0.15 },
    flameLoop: { start: 0.23, duration: 'until_impact' }
  }
};
```

---

## 8. Cannonball Animation

### 8.1 Animation Overview

```typescript
const CANNONBALL_ANIMATION = {
  abilityName: 'Cannonball',
  abilityType: 'area_damage',
  totalDuration: 0.7,

  description: 'Character pulls out a hand cannon and fires an explosive round',

  phases: {
    anticipation: {
      duration: 0.2,
      keyframes: [
        { time: 0, pose: 'standing' },
        { time: 0.1, pose: 'reaching_for_cannon' },
        { time: 0.2, pose: 'cannon_aimed' }
      ],
      cannonDraw: {
        timing: 0.05,
        from: 'back_holster',
        to: 'two_hand_grip'
      }
    },

    fire: {
      duration: 0.15,
      keyframes: [
        { time: 0, pose: 'aiming' },
        { time: 0.05, pose: 'recoil_peak' },
        { time: 0.15, pose: 'recoil_absorbed' }
      ],
      projectileRelease: {
        timing: 0.02,
        muzzleFlash: true
      },
      recoil: {
        bodyPush: -0.3, // Meters back
        armKick: 30, // Degrees up
        shoulderRotation: 15
      }
    },

    recovery: {
      duration: 0.35,
      keyframes: [
        { time: 0, pose: 'post_recoil' },
        { time: 0.15, pose: 'cannon_lowering' },
        { time: 0.28, pose: 'cannon_stowing' },
        { time: 0.35, pose: 'ready_stance' }
      ],
      cannonStow: {
        timing: 0.2,
        to: 'back_holster'
      },
      smokeClear: true
    }
  },

  rootMotion: {
    enabled: true,
    recoilPush: {
      distance: -0.3,
      duration: 0.1,
      recover: 0.2
    }
  }
};
```

---

## 9. Lightning Strike Animation

### 9.1 Animation Overview

```typescript
const LIGHTNING_STRIKE_ANIMATION = {
  abilityName: 'Lightning Strike',
  abilityType: 'targeted_damage',
  totalDuration: 0.6,

  description: 'Character calls down lightning on a targeted area',

  phases: {
    anticipation: {
      duration: 0.25,
      keyframes: [
        { time: 0, pose: 'standing' },
        { time: 0.1, pose: 'arm_raising' },
        { time: 0.25, pose: 'arm_skyward_charged' }
      ],
      electricBuildup: {
        timing: 0.1,
        location: 'raised_hand',
        arcs: true
      }
    },

    cast: {
      duration: 0.1,
      keyframes: [
        { time: 0, pose: 'arm_skyward' },
        { time: 0.05, pose: 'point_forward' },
        { time: 0.1, pose: 'casting_release' }
      ],
      targetingCircle: {
        visible: 'during_anticipation',
        confirmed: 'at_cast'
      },
      lightningCall: {
        timing: 0.05,
        delay: 0.1 // Lightning hits 0.1s after cast
      }
    },

    recovery: {
      duration: 0.25,
      keyframes: [
        { time: 0, pose: 'arm_extended' },
        { time: 0.12, pose: 'arm_lowering' },
        { time: 0.25, pose: 'ready_stance' }
      ],
      residualSparks: {
        enabled: true,
        duration: 0.2
      }
    }
  }
};
```

---

## 10. Generic Ability Animation Fallbacks

### 10.1 Generic Cast Poses

```typescript
const GENERIC_ABILITY_ANIMATIONS = {
  // Used for abilities without unique animations
  oneHanded: {
    name: 'generic_cast_one_hand',
    duration: 0.4,
    phases: {
      anticipation: { duration: 0.1, pose: 'arm_raised_palm_out' },
      cast: { duration: 0.15, pose: 'thrust_forward' },
      recovery: { duration: 0.15, pose: 'arm_return' }
    },
    suitableFor: ['ranged_instant', 'buff', 'debuff']
  },

  twoHanded: {
    name: 'generic_cast_two_hands',
    duration: 0.5,
    phases: {
      anticipation: { duration: 0.15, pose: 'both_arms_gathering' },
      cast: { duration: 0.2, pose: 'push_forward' },
      recovery: { duration: 0.15, pose: 'arms_return' }
    },
    suitableFor: ['area_effect', 'powerful_spell']
  },

  groundSlam: {
    name: 'generic_ground_slam',
    duration: 0.6,
    phases: {
      anticipation: { duration: 0.15, pose: 'arms_raised_high' },
      cast: { duration: 0.2, pose: 'slam_down' },
      recovery: { duration: 0.25, pose: 'recovering_from_slam' }
    },
    suitableFor: ['ground_aoe', 'shockwave']
  },

  throw: {
    name: 'generic_throw',
    duration: 0.4,
    phases: {
      anticipation: { duration: 0.1, pose: 'wind_up' },
      cast: { duration: 0.15, pose: 'throw_release' },
      recovery: { duration: 0.15, pose: 'follow_through' }
    },
    suitableFor: ['projectile', 'grenade']
  }
};
```

---

## 11. Animation State Machine

### 11.1 Ability Animation States

```typescript
const ABILITY_STATE_MACHINE = {
  states: {
    idle: {
      transitions: ['anticipation'],
      canInterrupt: true
    },
    anticipation: {
      transitions: ['action', 'cancelled'],
      canInterrupt: true,
      onInterrupt: 'blend_to_idle'
    },
    action: {
      transitions: ['recovery'],
      canInterrupt: false, // Commitment
      onComplete: 'auto_to_recovery'
    },
    recovery: {
      transitions: ['idle', 'anticipation'],
      canInterrupt: true,
      earlyCancelWindow: 0.5 // Normalized
    },
    cancelled: {
      transitions: ['idle'],
      playAnimation: 'ability_cancel_flinch'
    }
  },

  interrupts: {
    stun: {
      fromStates: ['anticipation', 'recovery'],
      toState: 'cancelled'
    },
    knockback: {
      fromStates: ['anticipation', 'action', 'recovery'],
      toState: 'cancelled',
      forceAnimation: 'knockback_reaction'
    },
    death: {
      fromStates: ['all'],
      toState: 'death',
      immediate: true
    }
  }
};
```

### 11.2 Animation Canceling Rules

```typescript
const ANIMATION_CANCEL_RULES = {
  // What can cancel ability animations
  cancelSources: {
    player_input: {
      dodge: { cancelsPhases: ['anticipation', 'recovery'] },
      abilityChain: { cancelsPhases: ['recovery'], afterWindow: 0.5 },
      jump: { cancelsPhases: ['recovery'], afterWindow: 0.7 }
    },
    external: {
      damage: { cancelsPhases: ['anticipation'] },
      stun: { cancelsPhases: ['anticipation', 'recovery'] },
      knockback: { cancelsPhases: ['all'], overridesAll: true }
    }
  },

  cancelAnimations: {
    quickCancel: {
      duration: 0.1,
      blendWeight: 0.5
    },
    interruptedFlinch: {
      duration: 0.15,
      playOnDamage: true
    },
    chainCancel: {
      duration: 0.05, // Nearly instant for combos
      seamless: true
    }
  }
};
```

---

## 12. Performance Specifications

### 12.1 Animation Performance Budget

```typescript
const ANIMATION_PERFORMANCE = {
  boneCount: {
    maxPerCharacter: 60,
    criticalBones: 30, // Always animated
    secondaryBones: 30 // LOD-dependent
  },

  frameRate: {
    keyframeRate: 30, // Keyframes per second
    interpolation: 'linear', // Between keyframes
    mobileOptimized: true
  },

  blendWeights: {
    maxLayers: 3,
    maxActiveBlends: 2
  },

  lod: {
    near: { distance: 0, boneRatio: 1.0, keyframeRatio: 1.0 },
    mid: { distance: 15, boneRatio: 0.7, keyframeRatio: 0.5 },
    far: { distance: 30, boneRatio: 0.5, keyframeRatio: 0.25 }
  }
};
```

### 12.2 Memory Budget

```typescript
const ANIMATION_MEMORY = {
  perAbilityAnimation: {
    average: '50KB',
    max: '100KB',
    compressed: true
  },

  totalBudget: {
    allAbilityAnimations: '1.5MB',
    activeInMemory: '500KB'
  },

  streaming: {
    enabled: true,
    preloadOnSelect: true, // Load when ability equipped
    unloadDelay: 30 // Seconds after last use
  }
};
```

---

## 13. Testing and Validation

### 13.1 Animation Testing Checklist

| Test | Criteria |
|------|----------|
| VFX Sync | Effects align with animation keyframes |
| Damage Timing | Hitbox active at correct frame |
| Blend Quality | No popping or jarring transitions |
| Cancel Response | Interrupts within 1 frame of input |
| Root Motion | Character moves correct distance |
| LOD Quality | Acceptable at all distances |
| Performance | 60 FPS with multiple abilities |

### 13.2 Animation Sync Validation

```typescript
const SYNC_VALIDATION = {
  vfxAlignment: {
    maxOffset: 0.05, // Seconds
    criticalPoints: ['release', 'impact', 'effect_start']
  },

  audioAlignment: {
    maxOffset: 0.03, // Seconds (audio more sensitive)
    criticalPoints: ['swing', 'impact', 'cast']
  },

  gameplayAlignment: {
    damageWindow: 'exact_match', // Must be precise
    invulnerabilityWindow: 'exact_match',
    movementTiming: 'within_1_frame'
  }
};
```

---

## Appendix A: Animation Clip Summary

| Ability | Total Duration | Phases | Root Motion | Priority |
|---------|---------------|--------|-------------|----------|
| Barrel Roll | 0.6s | 3 | Yes (8m) | High |
| Grappling Hook | 0.8s (var) | 4 | Yes (dynamic) | High |
| Cutlass Slash | 0.7s | 3x3 | Yes (1.6m) | High |
| Healing Grog | 1.2s | 3 | No | High |
| Fireball | 0.5s | 3 | No | Medium |
| Cannonball | 0.7s | 3 | Recoil | Medium |
| Lightning Strike | 0.6s | 3 | No | Medium |

---

## Appendix B: Bone Reference

### Critical Bones for Ability Animations

| Bone | Purpose | Priority |
|------|---------|----------|
| Spine_01-03 | Torso rotation/lean | Critical |
| Clavicle_L/R | Shoulder base | Critical |
| UpperArm_L/R | Arm swing | Critical |
| LowerArm_L/R | Elbow bend | Critical |
| Hand_L/R | Grip/gesture | Critical |
| Neck | Head follow | High |
| Head | Look direction | High |
| Thigh_L/R | Stance | Medium |
| Calf_L/R | Knee bend | Medium |
| Foot_L/R | Grounding | Medium |
| Weapon_attach | Prop holding | Critical |
