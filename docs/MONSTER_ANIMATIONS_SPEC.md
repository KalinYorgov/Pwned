# Monster Animations Specification

## Document Information
- **Task ID:** ART-024
- **Priority:** P1
- **Complexity:** M
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines animation sets for all NPC monsters in Plunderstorm Mobile. These animations bring PvE enemies to life, communicate threat level and behavior, and ensure clear attack telegraphing for fair gameplay.

### 1.2 Design Goals
- **Personality Expression:** Each monster type has distinct movement character
- **Attack Telegraphing:** Clear visual cues before attacks for player reaction
- **Death Satisfaction:** Rewarding death animations for combat feedback
- **Performance Optimized:** Efficient animations for multiple monsters on screen
- **AI Integration:** Animations support behavior state machine

### 1.3 Related Documents
- NPC Monster Models Specification (ART-006)
- Combat Feedback VFX Specification (ART-014)
- Sound Effects Specification (ART-027)

---

## 2. Animation Framework

### 2.1 Standard Animation Set Per Monster

```typescript
const MONSTER_ANIMATION_SET = {
  // Required animations for all monsters
  required: {
    idle: {
      description: 'Standing/waiting state',
      looping: true,
      variations: 2 // At least 2 to avoid repetition
    },
    patrol: {
      description: 'Moving while unaware of player',
      looping: true,
      variants: ['walk', 'slow_walk']
    },
    alert: {
      description: 'Transition when spotting player',
      looping: false,
      duration: '0.5-0.8s'
    },
    chase: {
      description: 'Aggressive pursuit movement',
      looping: true,
      faster_than: 'patrol'
    },
    attack: {
      description: 'Primary attack animation',
      looping: false,
      telegraph: 'required',
      minCount: 1
    },
    hurt: {
      description: 'Reaction to taking damage',
      looping: false,
      duration: '0.3-0.5s',
      variations: 2
    },
    death: {
      description: 'Dying animation',
      looping: false,
      duration: '1.0-2.0s',
      variations: 2
    }
  },

  // Optional animations for enhanced monsters
  optional: {
    idle_special: 'Personality fidgets',
    taunt: 'Pre-combat intimidation',
    attack_secondary: 'Alternative attack',
    attack_special: 'Rare/powerful attack',
    stagger: 'Heavy hit reaction',
    spawn: 'Appearance animation',
    despawn: 'Fade out animation'
  }
};
```

### 2.2 Animation Timing Guidelines

```typescript
const ANIMATION_TIMING = {
  telegraph: {
    // Time before damage/effect for player to react
    minimum: 0.4, // Seconds
    recommended: 0.6,
    maximum: 1.2 // Longer for powerful attacks
  },

  transitions: {
    idleToAlert: 0.2,
    alertToChase: 0.15,
    chaseToAttack: 0.1,
    attackToIdle: 0.2,
    anyToHurt: 0.05 // Near instant
  },

  recovery: {
    afterAttack: 0.3, // Vulnerability window
    afterHurt: 0.2,
    afterStagger: 0.5
  }
};
```

---

## 3. Skeleton Pirate Animations

### 3.1 Character Overview
- **Type:** Basic melee enemy
- **Poly Count:** 2,000 triangles
- **Bone Count:** 35
- **Movement Style:** Shambling, jerky undead motion

### 3.2 Animation Set

```typescript
const SKELETON_PIRATE_ANIMATIONS = {
  idle: {
    variations: [
      {
        name: 'idle_stand',
        duration: 3.0,
        loop: true,
        description: 'Standing with slight sway, head occasionally tilts',
        keyPoses: ['weight_shift', 'head_tilt', 'arm_adjust']
      },
      {
        name: 'idle_scratch',
        duration: 2.5,
        loop: true,
        description: 'Scratching skull with bony fingers',
        keyPoses: ['reach_head', 'scratch_scratch', 'arm_down']
      }
    ],
    blendTree: {
      type: 'random_selection',
      weight: [0.7, 0.3] // Probability
    }
  },

  patrol: {
    name: 'walk_shamble',
    duration: 1.2, // Per cycle
    loop: true,
    speed: 2.0, // Meters per second
    description: 'Uneven, lurching walk with dragging leg',
    style: {
      asymmetric: true,
      leftLegDrag: true,
      armSwing: 'loose_hanging',
      headBob: 'irregular'
    },
    rootMotion: {
      forward: true,
      perCycle: 1.5 // Meters
    }
  },

  alert: {
    name: 'spot_player',
    duration: 0.6,
    loop: false,
    description: 'Snaps head toward player, raises weapon, hostile stance',
    phases: {
      headSnap: { duration: 0.15, target: 'player_direction' },
      bodyTurn: { duration: 0.25, towardPlayer: true },
      weaponRaise: { duration: 0.2, threatPose: true }
    },
    sfx: 'skeleton_rattle_alert',
    vfx: 'eye_socket_glow'
  },

  chase: {
    name: 'run_aggressive',
    duration: 0.8, // Per cycle
    loop: true,
    speed: 4.0, // Meters per second
    description: 'Faster shambling run, weapon raised',
    style: {
      moreErratic: true,
      weaponForward: true,
      leaningAggressive: 15 // Degrees forward
    },
    rootMotion: {
      forward: true,
      perCycle: 2.5 // Meters
    }
  },

  attacks: {
    primary: {
      name: 'cutlass_swing',
      duration: 0.8,
      loop: false,
      description: 'Overhead cutlass slash',
      phases: {
        telegraph: {
          duration: 0.4,
          animation: 'raise_weapon_overhead',
          visual: 'weapon_glint',
          audio: 'skeleton_grunt'
        },
        strike: {
          duration: 0.15,
          animation: 'swing_down',
          damageFrame: 0.1,
          hitbox: 'frontal_arc_2m'
        },
        recovery: {
          duration: 0.25,
          animation: 'recoil_return',
          vulnerable: true
        }
      }
    },
    secondary: {
      name: 'thrust_stab',
      duration: 0.7,
      loop: false,
      description: 'Quick forward thrust',
      phases: {
        telegraph: {
          duration: 0.3,
          animation: 'pull_back_thrust_ready'
        },
        strike: {
          duration: 0.15,
          animation: 'lunge_forward',
          damageFrame: 0.1,
          hitbox: 'frontal_line_2.5m'
        },
        recovery: {
          duration: 0.25,
          animation: 'recover_stance'
        }
      }
    }
  },

  hurt: {
    variations: [
      {
        name: 'hurt_front',
        duration: 0.35,
        description: 'Staggers backward, bones rattle',
        displacement: -0.5 // Meters back
      },
      {
        name: 'hurt_side',
        duration: 0.35,
        description: 'Stumbles sideways',
        displacement: 0.3 // Meters side
      }
    ],
    sfx: 'bone_clatter',
    vfx: 'bone_chip_particles'
  },

  death: {
    variations: [
      {
        name: 'death_collapse',
        duration: 1.5,
        description: 'Bones disconnect and collapse into pile',
        phases: {
          reaction: { duration: 0.3, animation: 'death_gasp' },
          collapse: { duration: 0.7, animation: 'bones_separate' },
          settle: { duration: 0.5, animation: 'bones_scatter' }
        },
        ragdoll: false,
        lootSpawn: 0.8 // Timing for loot
      },
      {
        name: 'death_shatter',
        duration: 1.2,
        description: 'Explodes into bone fragments',
        phases: {
          impact: { duration: 0.2, animation: 'hit_reaction' },
          shatter: { duration: 0.3, animation: 'explode_outward' },
          fade: { duration: 0.7, animation: 'bones_dissolve' }
        },
        useFor: 'overkill_damage',
        lootSpawn: 0.5
      }
    ],
    sfx: 'skeleton_death_rattle',
    vfx: 'soul_wisp_release'
  }
};
```

---

## 4. Giant Crab Animations

### 4.1 Character Overview
- **Type:** Medium tank enemy
- **Poly Count:** 3,000 triangles
- **Bone Count:** 40 (extra for claws/legs)
- **Movement Style:** Scuttling, side-stepping crustacean

### 4.2 Animation Set

```typescript
const GIANT_CRAB_ANIMATIONS = {
  idle: {
    variations: [
      {
        name: 'idle_breathe',
        duration: 2.5,
        loop: true,
        description: 'Shell rises/falls with breathing, claws twitch',
        keyPoses: ['shell_expand', 'shell_contract', 'claw_adjust']
      },
      {
        name: 'idle_bubble',
        duration: 3.0,
        loop: true,
        description: 'Blows bubbles from mouth, eyes swivel',
        vfx: 'bubble_particles'
      }
    ]
  },

  patrol: {
    name: 'scuttle_walk',
    duration: 1.0,
    loop: true,
    speed: 1.5, // Slower, tanky
    description: 'Sideways scuttling movement, all legs coordinated',
    style: {
      direction: 'sideways',
      legWave: 'synchronized',
      bodyBob: 'slight',
      clawsReady: true
    },
    legCycle: {
      legs: 6,
      pattern: 'tripod_gait', // Alternating tripod
      phaseOffset: 0.167 // Per leg pair
    },
    rootMotion: {
      sideways: true,
      perCycle: 1.2
    }
  },

  alert: {
    name: 'threat_display',
    duration: 0.8,
    loop: false,
    description: 'Raises claws high, snaps aggressively, eyes lock on',
    phases: {
      clawRaise: { duration: 0.3, both_claws: 'overhead' },
      snap: { duration: 0.2, claws: 'aggressive_snap' },
      ready: { duration: 0.3, stance: 'combat_low' }
    },
    sfx: 'crab_hiss_snap',
    vfx: 'sand_puff'
  },

  chase: {
    name: 'scuttle_fast',
    duration: 0.6,
    loop: true,
    speed: 3.0,
    description: 'Rapid scuttling, claws forward',
    style: {
      direction: 'forward_bias', // Mostly sideways but angling forward
      legSpeed: 'fast',
      shellRattle: true
    },
    rootMotion: {
      forward: true,
      perCycle: 1.8
    }
  },

  attacks: {
    primary: {
      name: 'claw_snap',
      duration: 0.9,
      loop: false,
      description: 'Big claw snaps shut on target area',
      phases: {
        telegraph: {
          duration: 0.5,
          animation: 'claw_raise_open',
          visual: 'claw_glow_outline',
          targetIndicator: 'cone_3m'
        },
        strike: {
          duration: 0.15,
          animation: 'claw_slam_shut',
          damageFrame: 0.1,
          hitbox: 'cone_3m',
          sfx: 'claw_snap_loud'
        },
        recovery: {
          duration: 0.25,
          animation: 'claw_retract',
          vulnerable: true
        }
      }
    },
    secondary: {
      name: 'ground_pound',
      duration: 1.2,
      loop: false,
      description: 'Slams both claws down, area damage',
      phases: {
        telegraph: {
          duration: 0.6,
          animation: 'rear_up_claws_high',
          visual: 'ground_circle_indicator',
          targetIndicator: 'aoe_4m'
        },
        strike: {
          duration: 0.2,
          animation: 'slam_down',
          damageFrame: 0.15,
          hitbox: 'aoe_4m',
          sfx: 'ground_shake',
          vfx: 'sand_explosion'
        },
        recovery: {
          duration: 0.4,
          animation: 'recover_from_slam',
          vulnerable: true,
          stuck: 0.2 // Claws stuck briefly
        }
      }
    }
  },

  hurt: {
    variations: [
      {
        name: 'hurt_shell_ring',
        duration: 0.4,
        description: 'Shell vibrates, crab stumbles sideways',
        displacement: -0.3
      },
      {
        name: 'hurt_claw_flinch',
        duration: 0.35,
        description: 'Pulls claws in protectively'
      }
    ],
    sfx: 'shell_thunk',
    vfx: 'shell_chip_particles'
  },

  death: {
    variations: [
      {
        name: 'death_flip',
        duration: 2.0,
        description: 'Flips onto back, legs twitch, goes still',
        phases: {
          knockback: { duration: 0.4, animation: 'hit_hard' },
          flip: { duration: 0.6, animation: 'tumble_over' },
          twitch: { duration: 0.7, animation: 'legs_twitch' },
          still: { duration: 0.3, animation: 'legs_curl' }
        },
        lootSpawn: 1.0
      },
      {
        name: 'death_shell_break',
        duration: 1.5,
        description: 'Shell cracks and collapses',
        phases: {
          crack: { duration: 0.5, animation: 'shell_fracture' },
          collapse: { duration: 0.5, animation: 'body_crumple' },
          settle: { duration: 0.5, animation: 'shell_pieces_fall' }
        },
        useFor: 'heavy_damage',
        lootSpawn: 0.8
      }
    ],
    sfx: 'crab_death_screech',
    vfx: 'crab_innards_burst'
  },

  special: {
    burrow: {
      name: 'burrow_hide',
      duration: 1.5,
      description: 'Digs into sand to hide or reposition',
      phases: {
        dig: { duration: 0.7, animation: 'dig_down' },
        underground: { duration: 'variable' },
        emerge: { duration: 0.8, animation: 'burst_up' }
      },
      vfx: 'sand_spray'
    }
  }
};
```

---

## 5. Cursed Sailor Animations

### 5.1 Character Overview
- **Type:** Ranged ghostly enemy
- **Poly Count:** 2,500 triangles
- **Bone Count:** 38
- **Movement Style:** Ethereal floating, sudden movements

### 5.2 Animation Set

```typescript
const CURSED_SAILOR_ANIMATIONS = {
  idle: {
    variations: [
      {
        name: 'idle_hover',
        duration: 3.0,
        loop: true,
        description: 'Floats slightly off ground, ghostly sway',
        style: {
          hoverHeight: 0.3, // Meters
          swayAmplitude: 5, // Degrees
          flickerRate: 0.1 // Occasional transparency
        }
      },
      {
        name: 'idle_lament',
        duration: 4.0,
        loop: true,
        description: 'Head bowed, arms limp, occasional sob motion',
        emotional: true
      }
    ]
  },

  patrol: {
    name: 'glide_wander',
    duration: 1.5,
    loop: true,
    speed: 2.5,
    description: 'Drifts forward without leg movement, ghostly trail',
    style: {
      noLegs: true, // Lower body fades/trails
      armsDangle: true,
      headForward: true,
      ghostTrail: {
        enabled: true,
        length: 2.0,
        opacity: 0.3
      }
    },
    rootMotion: {
      forward: true,
      floating: true, // No ground contact
      perCycle: 2.5
    }
  },

  alert: {
    name: 'ghost_turn',
    duration: 0.5,
    loop: false,
    description: 'Snaps to face player, eyes flare, hostile wail',
    phases: {
      turnPhase: { duration: 0.2, animation: 'instant_face' },
      hostilePhase: { duration: 0.3, animation: 'aggressive_hover' }
    },
    sfx: 'ghostly_wail',
    vfx: 'eye_flare_green'
  },

  chase: {
    name: 'ghost_rush',
    duration: 1.0,
    loop: true,
    speed: 4.5, // Fast but stops to attack
    description: 'Rushes forward, arm reaching out',
    style: {
      leadingArm: 'reaching',
      bodyAngle: 30, // Degrees forward
      trailIntensity: 'strong'
    }
  },

  attacks: {
    primary: {
      name: 'spectral_bolt',
      duration: 0.9,
      loop: false,
      type: 'ranged',
      description: 'Conjures and hurls ghostly energy bolt',
      phases: {
        telegraph: {
          duration: 0.4,
          animation: 'arm_charge_gather',
          visual: 'spectral_orb_grow',
          audio: 'ghost_charge'
        },
        cast: {
          duration: 0.2,
          animation: 'throw_forward',
          projectileSpawn: 0.1,
          projectile: {
            type: 'spectral_bolt',
            speed: 15, // Meters per second
            homing: false
          }
        },
        recovery: {
          duration: 0.3,
          animation: 'arm_return',
          canMove: true
        }
      },
      range: 15 // Meters
    },
    secondary: {
      name: 'ghostly_scream',
      duration: 1.2,
      loop: false,
      type: 'aoe',
      description: 'Screams, damaging nearby enemies',
      phases: {
        telegraph: {
          duration: 0.5,
          animation: 'inhale_gather',
          visual: 'energy_pull_inward'
        },
        scream: {
          duration: 0.3,
          animation: 'head_back_scream',
          damageFrame: 0.15,
          hitbox: 'aoe_5m',
          sfx: 'banshee_scream',
          vfx: 'sound_wave_ring'
        },
        recovery: {
          duration: 0.4,
          animation: 'exhausted_hover'
        }
      }
    }
  },

  hurt: {
    variations: [
      {
        name: 'hurt_flicker',
        duration: 0.4,
        description: 'Form flickers and distorts',
        effect: 'transparency_pulse'
      },
      {
        name: 'hurt_disperse',
        duration: 0.5,
        description: 'Briefly disperses then reforms',
        effect: 'particle_scatter_reform'
      }
    ],
    sfx: 'ghost_pained_moan',
    vfx: 'ectoplasm_splash'
  },

  death: {
    variations: [
      {
        name: 'death_dissolve',
        duration: 2.0,
        description: 'Slowly fades away with peaceful expression',
        phases: {
          reaction: { duration: 0.3, animation: 'surprised_release' },
          fade: { duration: 1.2, animation: 'dissolve_upward' },
          gone: { duration: 0.5, animation: 'last_wisps' }
        },
        lootSpawn: 0.8
      },
      {
        name: 'death_banish',
        duration: 1.5,
        description: 'Screams and is pulled into void',
        phases: {
          scream: { duration: 0.4, animation: 'death_scream' },
          pull: { duration: 0.8, animation: 'sucked_into_point' },
          pop: { duration: 0.3, animation: 'implode_vanish' }
        },
        useFor: 'magical_damage',
        lootSpawn: 0.5
      }
    ],
    sfx: 'ghost_final_moan',
    vfx: 'soul_release_wisps'
  },

  special: {
    teleport: {
      name: 'blink_reposition',
      duration: 0.8,
      description: 'Fades out and reappears elsewhere',
      phases: {
        fadeOut: { duration: 0.3, animation: 'dissolve_quick' },
        travel: { duration: 0.2, invisible: true },
        fadeIn: { duration: 0.3, animation: 'materialize' }
      },
      vfx: 'ghost_portal'
    }
  }
};
```

---

## 6. Kraken Tentacle Animations

### 6.1 Character Overview
- **Type:** Mini-boss large obstacle
- **Poly Count:** 5,000 triangles
- **Bone Count:** 50 (tentacle chain)
- **Movement Style:** Undulating, whipping tentacle

### 6.2 Animation Set

```typescript
const KRAKEN_TENTACLE_ANIMATIONS = {
  idle: {
    name: 'tentacle_sway',
    duration: 4.0,
    loop: true,
    description: 'Massive tentacle sways menacingly above water',
    style: {
      baseFixed: 'water_surface',
      tipFree: 'organic_sway',
      suctionCups: 'pulsing',
      waveMotion: {
        frequency: 0.5,
        amplitude: 2.0, // Meters
        propagation: 'base_to_tip'
      }
    },
    environment: {
      waterSplash: 'continuous_drips',
      shadow: 'moves_with_sway'
    }
  },

  alert: {
    name: 'tentacle_rear',
    duration: 1.0,
    loop: false,
    description: 'Rises higher, tip orients toward target',
    phases: {
      rise: { duration: 0.5, animation: 'extend_upward' },
      orient: { duration: 0.5, animation: 'tip_track_target' }
    },
    sfx: 'water_surge',
    vfx: 'water_cascade'
  },

  attacks: {
    slam: {
      name: 'tentacle_slam',
      duration: 1.8,
      loop: false,
      description: 'Raises high and slams down on area',
      phases: {
        telegraph: {
          duration: 0.8,
          animation: 'rise_high_coil',
          visual: 'ground_shadow_target',
          targetIndicator: 'line_8m',
          audio: 'kraken_rumble'
        },
        strike: {
          duration: 0.3,
          animation: 'whip_down',
          damageFrame: 0.2,
          hitbox: 'line_8m_wide_2m',
          sfx: 'massive_impact',
          vfx: 'ground_crack_dust',
          screenShake: { amplitude: 10, duration: 0.3 }
        },
        recovery: {
          duration: 0.7,
          animation: 'peel_off_ground',
          stuck: 0.3, // Stuck briefly
          vulnerable: true
        }
      },
      damage: 'high',
      knockback: 5 // Meters
    },

    sweep: {
      name: 'tentacle_sweep',
      duration: 1.5,
      loop: false,
      description: 'Sweeps across ground in wide arc',
      phases: {
        telegraph: {
          duration: 0.5,
          animation: 'coil_side',
          visual: 'arc_indicator',
          targetIndicator: 'arc_180_10m'
        },
        strike: {
          duration: 0.4,
          animation: 'sweep_across',
          damageWindow: { start: 0.1, end: 0.35 },
          hitbox: 'arc_10m',
          sfx: 'whoosh_heavy',
          vfx: 'dust_wave'
        },
        recovery: {
          duration: 0.6,
          animation: 'return_position'
        }
      },
      damage: 'medium',
      knockback: 8
    },

    grab: {
      name: 'tentacle_grab',
      duration: 2.5,
      loop: false,
      description: 'Attempts to grab and squeeze player',
      phases: {
        telegraph: {
          duration: 0.6,
          animation: 'tip_open_reach',
          visual: 'target_circle',
          targetIndicator: 'circle_3m'
        },
        grab: {
          duration: 0.3,
          animation: 'snap_wrap',
          hitCheck: 0.2,
          onHit: 'player_grabbed_state'
        },
        squeeze: {
          duration: 1.0,
          animation: 'constrict_pulse',
          damagePerSecond: 'high',
          breakFree: 'button_mash'
        },
        release: {
          duration: 0.6,
          animation: 'toss_away',
          throwDistance: 6
        }
      }
    }
  },

  hurt: {
    name: 'tentacle_flinch',
    duration: 0.5,
    description: 'Recoils from damage, thrashes briefly',
    effect: {
      recoil: 'away_from_damage',
      thrash: 'brief_random'
    },
    sfx: 'kraken_pain_groan',
    vfx: 'kraken_blood_spray'
  },

  death: {
    name: 'tentacle_retreat',
    duration: 3.0,
    description: 'Thrashes wildly, then sinks beneath waves',
    phases: {
      thrash: {
        duration: 1.5,
        animation: 'wild_thrashing',
        dangerous: true, // Still damages
        hitbox: 'random_sweep'
      },
      sink: {
        duration: 1.0,
        animation: 'slide_under_water',
        sfx: 'water_suction'
      },
      gone: {
        duration: 0.5,
        animation: 'ripples_only'
      }
    },
    lootSpawn: 2.0,
    vfx: 'massive_water_splash'
  },

  spawn: {
    name: 'tentacle_emerge',
    duration: 2.0,
    description: 'Bursts from water dramatically',
    phases: {
      rumble: { duration: 0.5, animation: 'water_bubble' },
      burst: { duration: 0.5, animation: 'explosive_emerge' },
      establish: { duration: 1.0, animation: 'rise_to_idle' }
    },
    sfx: 'water_explosion',
    vfx: 'water_geyser',
    screenShake: { amplitude: 8, duration: 0.5 }
  }
};
```

---

## 7. Animation Technical Specifications

### 7.1 Bone Count Budgets

```typescript
const MONSTER_BONE_BUDGETS = {
  skeleton_pirate: {
    total: 35,
    spine: 4,
    arms: 6, // Per arm
    legs: 5, // Per leg
    head: 4,
    weapon: 2,
    cloth: 2
  },

  giant_crab: {
    total: 40,
    body: 5,
    claws: 6, // Per claw
    legs: 3, // Per leg (6 legs)
    eyes: 2,
    antennae: 2
  },

  cursed_sailor: {
    total: 38,
    spine: 5,
    arms: 6, // Per arm
    legs: 4, // Per leg (may be faded)
    head: 4,
    ghostTrail: 5
  },

  kraken_tentacle: {
    total: 50,
    tentacleChain: 40, // Main articulation
    suctionCups: 8,
    tip: 2
  }
};
```

### 7.2 Animation Memory Budgets

```typescript
const ANIMATION_MEMORY = {
  perMonsterType: {
    skeleton_pirate: {
      animations: 12,
      totalSize: '400KB',
      compressed: true
    },
    giant_crab: {
      animations: 14,
      totalSize: '500KB',
      compressed: true
    },
    cursed_sailor: {
      animations: 13,
      totalSize: '450KB',
      compressed: true
    },
    kraken_tentacle: {
      animations: 10,
      totalSize: '600KB',
      compressed: true
    }
  },

  totalBudget: '2MB',

  streaming: {
    preloadDistance: 30, // Meters
    unloadDistance: 50
  }
};
```

### 7.3 LOD Animation Settings

```typescript
const ANIMATION_LOD = {
  levels: {
    high: {
      distance: '0-15m',
      boneRatio: 1.0,
      sampleRate: 30, // FPS
      blendQuality: 'full'
    },
    medium: {
      distance: '15-30m',
      boneRatio: 0.7,
      sampleRate: 20,
      blendQuality: 'reduced'
    },
    low: {
      distance: '30-50m',
      boneRatio: 0.5,
      sampleRate: 15,
      blendQuality: 'simple'
    },
    culled: {
      distance: '50m+',
      animation: 'paused',
      visibleCheck: 'interval_1s'
    }
  },

  boneReduction: {
    priority: ['spine', 'arms', 'legs', 'secondary'],
    alwaysAnimate: ['root', 'spine_01', 'weapon']
  }
};
```

---

## 8. AI State Integration

### 8.1 Animation State Triggers

```typescript
const AI_ANIMATION_TRIGGERS = {
  states: {
    idle: {
      enterAnimation: 'idle_transition',
      loopAnimation: 'idle',
      exitConditions: ['spot_player', 'take_damage', 'patrol_timer']
    },
    patrol: {
      enterAnimation: 'start_walk',
      loopAnimation: 'patrol',
      exitConditions: ['spot_player', 'take_damage', 'reach_waypoint']
    },
    alert: {
      enterAnimation: 'alert',
      duration: 'animation_length',
      nextState: 'chase',
      interruptible: false
    },
    chase: {
      enterAnimation: 'chase_start',
      loopAnimation: 'chase',
      exitConditions: ['reach_attack_range', 'lose_target', 'take_damage']
    },
    attack: {
      enterAnimation: 'attack_chosen',
      duration: 'animation_length',
      interruptible: 'during_recovery',
      onComplete: 'return_to_chase'
    },
    hurt: {
      enterAnimation: 'hurt_random',
      priority: 'high',
      interruptsCurrent: true,
      duration: 'animation_length',
      onComplete: 'return_to_previous'
    },
    death: {
      enterAnimation: 'death_random',
      priority: 'highest',
      interruptsCurrent: true,
      final: true
    }
  }
};
```

### 8.2 Attack Selection Logic

```typescript
const ATTACK_SELECTION = {
  skeleton_pirate: {
    attacks: ['cutlass_swing', 'thrust_stab'],
    selection: 'weighted_random',
    weights: [0.7, 0.3],
    cooldowns: {
      global: 1.0, // Seconds
      perAttack: { cutlass_swing: 0, thrust_stab: 2.0 }
    }
  },

  giant_crab: {
    attacks: ['claw_snap', 'ground_pound'],
    selection: 'range_based',
    ranges: {
      claw_snap: { min: 0, max: 3 },
      ground_pound: { min: 0, max: 4, preferred: 'multiple_targets' }
    },
    cooldowns: {
      global: 1.5,
      perAttack: { claw_snap: 0, ground_pound: 5.0 }
    }
  },

  cursed_sailor: {
    attacks: ['spectral_bolt', 'ghostly_scream'],
    selection: 'range_based',
    ranges: {
      spectral_bolt: { min: 5, max: 15 },
      ghostly_scream: { min: 0, max: 5, preferred: 'close_range' }
    },
    cooldowns: {
      global: 2.0,
      perAttack: { spectral_bolt: 1.0, ghostly_scream: 8.0 }
    }
  },

  kraken_tentacle: {
    attacks: ['tentacle_slam', 'tentacle_sweep', 'tentacle_grab'],
    selection: 'pattern_based',
    pattern: ['slam', 'sweep', 'random', 'grab'],
    cooldowns: {
      global: 2.0,
      perAttack: { slam: 3.0, sweep: 4.0, grab: 10.0 }
    }
  }
};
```

---

## 9. Performance Optimization

### 9.1 Multi-Monster Scenarios

```typescript
const MULTI_MONSTER_OPTIMIZATION = {
  maxSimultaneous: {
    skeleton_pirate: 15,
    giant_crab: 5,
    cursed_sailor: 10,
    kraken_tentacle: 1,
    total: 25
  },

  optimization: {
    animationBatching: true,
    sharedSkeletons: true, // Same type shares rig
    staggeredUpdates: {
      enabled: true,
      framesPerBatch: 3 // Update 1/3 per frame
    },
    distanceCulling: {
      pause: 50, // Meters
      hide: 75
    }
  },

  qualityTiers: {
    low: {
      maxMonsters: 10,
      animationLOD: 'aggressive',
      effectsReduced: true
    },
    medium: {
      maxMonsters: 20,
      animationLOD: 'standard',
      effectsReduced: false
    },
    high: {
      maxMonsters: 25,
      animationLOD: 'full',
      effectsReduced: false
    }
  }
};
```

---

## 10. Testing Checklist

### 10.1 Animation Quality Tests

| Test | Criteria |
|------|----------|
| Telegraph clarity | Attack warning visible for 0.4s+ |
| Hitbox alignment | Damage matches animation visually |
| Blend quality | No popping between states |
| Loop seamless | No visible seam in looping anims |
| Death satisfaction | Deaths feel impactful |
| Personality match | Animation matches monster theme |

### 10.2 Performance Tests

| Test | Criteria |
|------|----------|
| 15 skeletons | 60 FPS maintained |
| 5 crabs | 60 FPS maintained |
| 10 sailors | 60 FPS maintained |
| 1 kraken + 10 others | 60 FPS maintained |
| Max monsters (25) | 30+ FPS minimum |

---

## Appendix A: Animation Clip Inventory

| Monster | Animation Count | Total Duration | Memory |
|---------|----------------|----------------|--------|
| Skeleton Pirate | 12 | 18.5s | 400KB |
| Giant Crab | 14 | 23.0s | 500KB |
| Cursed Sailor | 13 | 21.0s | 450KB |
| Kraken Tentacle | 10 | 28.0s | 600KB |
| **Total** | **49** | **90.5s** | **1.95MB** |

---

## Appendix B: VFX/SFX Sync Points

### Per Monster Critical Sync Events

| Monster | Animation | VFX Event | SFX Event | Frame |
|---------|-----------|-----------|-----------|-------|
| Skeleton | cutlass_swing | slash_arc | sword_whoosh | 0.14s |
| Skeleton | death_shatter | bone_explode | bone_clatter | 0.2s |
| Crab | claw_snap | claw_impact | snap_sound | 0.55s |
| Crab | ground_pound | dust_ring | boom | 0.75s |
| Sailor | spectral_bolt | bolt_spawn | ghost_fire | 0.5s |
| Sailor | death_dissolve | soul_wisps | fade_moan | 0.3s |
| Kraken | tentacle_slam | ground_crack | massive_thud | 1.0s |
| Kraken | spawn | water_geyser | water_burst | 0.5s |
