# Ability Combo System Specification

## Document Information
- **Task ID:** GAME-010
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies the ability combo system for Plunderstorm Mobile, enabling synergistic interactions between abilities when used in combination. The system adds strategic depth and rewards players who discover and execute complementary ability pairs.

---

## Combo System Architecture

### Core Concepts

```typescript
const COMBO_SYSTEM_CONCEPTS = {
  definition: {
    combo: 'A special effect triggered when two specific abilities interact',
    trigger: 'Abilities used within defined time/space window',
    reward: 'Bonus effect beyond normal ability function'
  },

  designPrinciples: {
    rewarding: 'Combos should feel satisfying when executed',
    balanced: 'Combos are bonuses, not requirements for viability',
    discoverable: 'Players can naturally find combos through play',
    intentional: 'Combos require skill/timing, not accidental',
    fair: 'No combo should be overpowered or mandatory'
  },

  categories: {
    offensive: 'Increased damage or area of effect',
    defensive: 'Enhanced protection or healing',
    utility: 'Improved mobility or crowd control',
    transformation: 'Ability behavior changes fundamentally'
  }
};
```

### Combo Detection System

```typescript
const COMBO_DETECTION = {
  architecture: {
    comboManager: {
      role: 'Central combo tracking and triggering',
      responsibilities: [
        'Track recent ability uses',
        'Detect combo conditions',
        'Trigger combo effects',
        'Manage cooldowns and limits'
      ]
    },
    abilityTracker: {
      role: 'Track ability activations with metadata',
      data: {
        abilityId: 'string',
        casterId: 'string',
        timestamp: 'number',
        position: 'Vector3',
        targets: 'PlayerId[]',
        areaOfEffect: 'Bounds'
      },
      retention: 5000 // ms to keep records
    }
  },

  detectionMethods: {
    temporal: {
      description: 'Abilities used within time window',
      window: 'Defined per combo (typically 2-5 seconds)',
      order: 'Some combos require specific order'
    },
    spatial: {
      description: 'Abilities affect overlapping area',
      method: 'Bounding box or radius intersection',
      tolerance: 'Defined per combo'
    },
    targetBased: {
      description: 'Abilities affect same target',
      method: 'Check target entity IDs'
    },
    stateBased: {
      description: 'Target has specific status effect',
      method: 'Check active buffs/debuffs'
    }
  }
};
```

---

## Launch Combos (5 Minimum)

### Combo 1: Storm Strike

```typescript
const COMBO_STORM_STRIKE = {
  id: 'combo_storm_strike',
  name: 'Storm Strike',

  abilities: {
    primary: 'Smoke Bomb',
    secondary: 'Lightning Strike'
  },

  trigger: {
    condition: 'Lightning Strike hits area covered by Smoke Bomb',
    timeWindow: 4000, // ms
    spatialRequirement: 'Lightning strike within smoke radius',
    order: 'Any order'
  },

  effect: {
    type: 'offensive',
    description: 'Lightning chains between all enemies in smoke',
    mechanics: {
      chainDamage: 40, // Per chain
      maxChains: 3,
      chainRange: 8, // meters within smoke
      stunDuration: 0.5 // seconds
    },
    visualEffect: 'Electric arcs visible through smoke'
  },

  balance: {
    additionalDamage: '+80-120 potential',
    difficulty: 'Medium (requires smoke placement)',
    counterplay: 'Exit smoke quickly'
  }
};
```

### Combo 2: Boarding Action

```typescript
const COMBO_BOARDING_ACTION = {
  id: 'combo_boarding_action',
  name: 'Boarding Action',

  abilities: {
    primary: 'Grappling Hook',
    secondary: 'Cutlass Slash'
  },

  trigger: {
    condition: 'Cutlass Slash used within 1 second of Grapple arrival',
    timeWindow: 1000, // ms after grapple lands
    spatialRequirement: 'None (automatic on arrival)',
    order: 'Grapple first, then Slash'
  },

  effect: {
    type: 'offensive',
    description: 'First strike after grapple deals massive bonus damage',
    mechanics: {
      bonusDamage: 50, // Flat bonus
      damageMultiplier: 1.5, // 50% more total
      cleaveWidth: 'Increased by 30%',
      guaranteedCrit: false
    },
    visualEffect: 'Golden slash trail, impact sparks'
  },

  balance: {
    additionalDamage: '+75-100 total',
    difficulty: 'Easy (natural combo)',
    counterplay: 'Dodge/shield after seeing grapple'
  }
};
```

### Combo 3: Inferno Gust

```typescript
const COMBO_INFERNO_GUST = {
  id: 'combo_inferno_gust',
  name: 'Inferno Gust',

  abilities: {
    primary: 'Wind Burst',
    secondary: 'Fireball'
  },

  trigger: {
    condition: 'Fireball passes through Wind Burst area',
    timeWindow: 2000, // ms
    spatialRequirement: 'Fireball trajectory intersects wind area',
    order: 'Wind Burst first (creates wind zone)'
  },

  effect: {
    type: 'offensive',
    description: 'Fireball gains speed, range, and explosion size',
    mechanics: {
      speedMultiplier: 1.8,
      rangeMultiplier: 1.5,
      explosionRadiusMultiplier: 1.4,
      burnDuration: '+1 second'
    },
    visualEffect: 'Fireball becomes larger, trailing flames intensify'
  },

  balance: {
    additionalDamage: '+30% explosion damage',
    difficulty: 'Hard (requires positioning)',
    counterplay: 'Larger but telegraphed, easier to dodge'
  }
};
```

### Combo 4: Toxic Recovery

```typescript
const COMBO_TOXIC_RECOVERY = {
  id: 'combo_toxic_recovery',
  name: 'Toxic Recovery',

  abilities: {
    primary: 'Poison Mackerel',
    secondary: 'Healing Grog'
  },

  trigger: {
    condition: 'Use Healing Grog while poisoned by own Poison Mackerel',
    timeWindow: 'While self-poison active',
    spatialRequirement: 'None (self-cast)',
    order: 'Poison first (can be reflected/self-hit), then Grog'
  },

  effect: {
    type: 'transformation',
    description: 'Poison converts to healing over time',
    mechanics: {
      poisonToHealing: true,
      healPerTick: 15, // Instead of 10 damage
      duration: 'Remaining poison duration',
      bonusHealing: 20, // Extra on top of Grog heal
      immunityToPoison: 3000 // ms after combo
    },
    visualEffect: 'Green poison clouds turn golden'
  },

  balance: {
    totalHealing: '+45-60 potential',
    difficulty: 'Very Hard (requires self-poisoning)',
    counterplay: 'Prevents offensive poison use'
  }
};
```

### Combo 5: Frozen Wake

```typescript
const COMBO_FROZEN_WAKE = {
  id: 'combo_frozen_wake',
  name: 'Frozen Wake',

  abilities: {
    primary: 'Barrel Roll',
    secondary: 'Frost Nova'
  },

  trigger: {
    condition: 'Frost Nova cast immediately after Barrel Roll',
    timeWindow: 500, // ms after roll ends
    spatialRequirement: 'None',
    order: 'Barrel Roll first'
  },

  effect: {
    type: 'utility',
    description: 'Leave a trail of ice that slows enemies',
    mechanics: {
      iceTrailLength: 'Full barrel roll path',
      trailWidth: 3, // meters
      slowAmount: 40, // percent
      slowDuration: 2, // seconds on contact
      trailDuration: 4, // seconds before melting
      frostNovaCooldownReduction: 2 // seconds
    },
    visualEffect: 'Frost trail left behind roll path'
  },

  balance: {
    utilityValue: 'Strong area denial',
    difficulty: 'Medium',
    counterplay: 'Avoid the visible ice trail'
  }
};
```

---

## Additional Combo Designs (Future)

### Planned Combos

```typescript
const FUTURE_COMBOS = {
  combo6_anchorSlam: {
    name: 'Anchor Slam',
    abilities: ['Grappling Hook (terrain)', 'Ground Pound'],
    effect: 'Pulling to terrain then ground pound creates shockwave',
    release: 'Post-launch update 1'
  },

  combo7_phantomStrike: {
    name: 'Phantom Strike',
    abilities: ['Shadow Step', 'Backstab'],
    effect: 'Backstab from Shadow Step always crits',
    release: 'Post-launch update 1'
  },

  combo8_tidalCrash: {
    name: 'Tidal Crash',
    abilities: ['Tidal Wave', 'Cannonball'],
    effect: 'Riding own wave increases Cannonball damage',
    release: 'Post-launch update 2'
  },

  combo9_thunderCloud: {
    name: 'Thunder Cloud',
    abilities: ['Smoke Bomb', 'Wind Burst'],
    effect: 'Creates storm cloud that randomly strikes',
    release: 'Post-launch update 2'
  },

  combo10_venomBlade: {
    name: 'Venom Blade',
    abilities: ['Poison Mackerel', 'Cutlass Slash'],
    effect: 'Slash applies poison to all hit enemies',
    release: 'Post-launch update 3'
  }
};
```

---

## Combo Feedback System

### Visual Feedback

```typescript
const COMBO_VISUAL_FEEDBACK = {
  comboTrigger: {
    screenEffect: {
      type: 'Brief flash',
      color: 'Gold/yellow',
      duration: 200, // ms
      intensity: 'Subtle (not distracting)'
    },
    textPopup: {
      text: '{Combo Name}!',
      position: 'Center screen, above character',
      style: {
        fontSize: 24,
        fontWeight: 'bold',
        color: '#FFD700',
        outline: '#000000'
      },
      animation: 'Scale up + fade out',
      duration: 1500
    },
    particleEffect: {
      type: 'Combo-specific particles',
      origin: 'Combo impact point',
      duration: 500
    }
  },

  abilityModification: {
    description: 'Visual change to ability when combo active',
    examples: [
      'Fireball glows brighter in wind',
      'Lightning arcs visible in smoke',
      'Ice trail appears behind roll'
    ]
  }
};
```

### Audio Feedback

```typescript
const COMBO_AUDIO_FEEDBACK = {
  comboTrigger: {
    soundEffect: {
      type: 'Unique combo sound',
      style: 'Satisfying "power-up" feel',
      volume: 'Slightly louder than normal ability',
      priority: 'High (don\'t get drowned out)'
    },
    voiceLine: {
      enabled: false, // Keep it snappy
      futureConsideration: 'Character callouts for combos'
    }
  },

  perComboSounds: {
    stormStrike: 'Electric surge + thunder crack',
    boardingAction: 'Blade ring + impact thud',
    infernoGust: 'Roaring flame + wind howl',
    toxicRecovery: 'Bubbling + healing chime',
    frozenWake: 'Ice crystallizing + cold wind'
  }
};
```

### UI Integration

```typescript
const COMBO_UI_INTEGRATION = {
  abilityDescriptions: {
    showCombos: true,
    format: 'Combos with: {Ability Name}',
    location: 'Below ability description',
    style: {
      color: '#FFD700',
      icon: 'combo_icon'
    }
  },

  comboHints: {
    enabled: true,
    trigger: 'When both combo abilities equipped',
    hint: 'Try combining {Ability1} with {Ability2}!',
    showOnce: true
  },

  comboPrediction: {
    enabled: false, // Don't make it too easy
    futureFeature: 'Glow on ability when combo possible'
  },

  comboLog: {
    location: 'Match summary screen',
    shows: 'Combos executed this match',
    format: '{Combo Name} x{count}'
  }
};
```

---

## Balance Framework

### Combo Power Budget

```typescript
const COMBO_BALANCE = {
  powerBudget: {
    principle: 'Combo bonus should be ~30-50% of single ability value',
    reasoning: 'Strong enough to pursue, not mandatory to win',

    damageGuidelines: {
      lowCombo: '+30-50 damage',
      mediumCombo: '+50-80 damage',
      highCombo: '+80-120 damage (requires difficult execution)'
    },

    utilityGuidelines: {
      duration: '+1-2 seconds on CC effects',
      range: '+20-50% on ability range',
      area: '+30-50% on area of effect'
    }
  },

  difficultyScaling: {
    easy: {
      description: 'Natural ability flow',
      bonus: 'Lower end of power budget',
      example: 'Boarding Action (Grapple → Slash)'
    },
    medium: {
      description: 'Requires positioning or timing',
      bonus: 'Mid power budget',
      example: 'Storm Strike (place smoke, aim lightning)'
    },
    hard: {
      description: 'Requires prediction or sacrifice',
      bonus: 'High power budget',
      example: 'Toxic Recovery (must poison self)'
    }
  },

  antiExploit: {
    cooldownSharing: false, // Combos don't reduce cooldowns unless specified
    infiniteLoops: 'No combo can trigger itself',
    stackingLimit: 'Same combo cannot stack effects',
    perTargetLimit: 'Max 1 combo effect per target per 3 seconds'
  }
};
```

### Testing Requirements

```typescript
const COMBO_TESTING = {
  balanceTesting: {
    metrics: [
      'Win rate of players using combos vs not',
      'Average damage from combos per match',
      'Combo execution rate (attempts vs success)',
      'Most/least used combos'
    ],
    targets: {
      winRateDelta: '< 5% advantage for combo users',
      comboDamagePercent: '10-20% of total damage',
      executionRate: '> 60% when attempted'
    }
  },

  exploitTesting: [
    'Infinite combo loops',
    'Stacking same combo multiple times',
    'Unintended ability interactions',
    'Network desync causing double combos'
  ],

  playTesting: {
    focus: [
      'Do combos feel satisfying?',
      'Are combos discoverable naturally?',
      'Is any combo mandatory for competitive play?',
      'Do combos feel unfair to opponents?'
    ]
  }
};
```

---

## Discovery & Progression

### Combo Discovery

```typescript
const COMBO_DISCOVERY = {
  naturalDiscovery: {
    enabled: true,
    method: 'Players find combos through experimentation',
    hint: 'Ability descriptions mention combo potential'
  },

  guidedDiscovery: {
    tutorialMention: 'Brief mention that combos exist',
    loading: 'Loading screen tips about specific combos',
    firstCombo: 'Celebration + explanation on first combo'
  },

  comboCollection: {
    ui: 'Combo codex in player profile',
    tracking: 'Which combos player has discovered',
    rewards: 'XP bonus for first-time combo execution'
  }
};
```

### Achievements

```typescript
const COMBO_ACHIEVEMENTS = {
  discoveries: [
    {
      id: 'achievement_first_combo',
      name: 'Synergy!',
      description: 'Execute your first ability combo',
      reward: { xp: 100, title: null }
    },
    {
      id: 'achievement_all_combos',
      name: 'Master Combiner',
      description: 'Execute all ability combos',
      reward: { xp: 500, title: 'Combo Master' }
    }
  ],

  mastery: [
    {
      id: 'achievement_combo_spree',
      name: 'Combo Spree',
      description: 'Execute 3 combos in one match',
      reward: { xp: 200 }
    },
    {
      id: 'achievement_combo_elimination',
      name: 'Stylish Elimination',
      description: 'Eliminate a player with a combo',
      reward: { xp: 150 }
    }
  ]
};
```

---

## Network Considerations

### Combo Validation

```typescript
const COMBO_NETWORKING = {
  validation: {
    authority: 'Server validates all combos',
    clientPrediction: true, // Show feedback immediately
    serverConfirmation: 'Apply actual effects on confirm',
    rollback: 'Remove predicted effects if server denies'
  },

  timing: {
    latencyTolerance: 150, // ms
    serverTimeWindow: 'Combo window + latency tolerance',
    tieBreaking: 'Server timestamp is authoritative'
  },

  replication: {
    comboEvent: {
      broadcast: 'To all nearby players',
      data: {
        comboId: 'string',
        casterId: 'string',
        position: 'Vector3',
        targets: 'PlayerId[]'
      }
    },
    effects: 'Standard ability effect replication'
  }
};
```

---

## Analytics

```typescript
const COMBO_ANALYTICS = {
  events: [
    {
      name: 'combo_executed',
      properties: {
        combo_id: 'string',
        ability1: 'string',
        ability2: 'string',
        damage_dealt: 'number',
        targets_hit: 'number',
        resulted_in_kill: 'boolean'
      }
    },
    {
      name: 'combo_discovered',
      properties: {
        combo_id: 'string',
        player_matches_played: 'number',
        discovery_method: 'natural | hint | tutorial'
      }
    },
    {
      name: 'combo_attempted',
      properties: {
        combo_id: 'string',
        success: 'boolean',
        failure_reason: 'timing | spatial | cooldown'
      }
    }
  ],

  dashboardMetrics: [
    'Combo execution rate per match',
    'Most popular combos',
    'Combo discovery rate over player lifetime',
    'Combo contribution to kills',
    'Win rate correlation with combo usage'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial ability combo system specification |
