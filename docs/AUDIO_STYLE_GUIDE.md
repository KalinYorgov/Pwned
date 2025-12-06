# Audio Style Guide

## Document Information
- **Task ID:** ART-025
- **Priority:** P1
- **Complexity:** S
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document defines the audio direction for all music, sound effects, and ambient audio in Plunderstorm Mobile. It establishes the sonic identity that supports the pirate fantasy theme while ensuring clarity, satisfaction, and mobile optimization.

### 1.2 Audio Vision Statement
*"The sound of Plunderstorm is an adventurous, whimsical pirate journey—orchestral swashbuckling meets Caribbean carnival, with punchy combat sounds that make every hit satisfying and victory feel like treasure found."*

### 1.3 Core Audio Pillars

1. **Adventurous Spirit:** Audio should evoke excitement, discovery, and high-seas adventure
2. **Satisfying Feedback:** Every action feels impactful through punchy, responsive sounds
3. **Thematic Cohesion:** All audio reinforces the pirate fantasy consistently
4. **Mobile Clarity:** Sounds cut through on small speakers while remaining pleasant on headphones
5. **Accessible Volume:** Well-balanced mixing that works at any player volume setting

---

## 2. Musical Style Direction

### 2.1 Genre and Influences

```typescript
const MUSIC_STYLE = {
  primaryGenre: 'Orchestral Adventure',
  secondaryInfluences: [
    'Sea Shanty Folk',
    'Caribbean/Calypso',
    'Epic Film Score',
    'Whimsical Fantasy'
  ],

  referenceComposers: [
    'Klaus Badelt / Hans Zimmer (Pirates of the Caribbean)',
    'Jeremy Soule (Elder Scrolls - exploration)',
    'Austin Wintory (Journey - emotional)',
    'Koji Kondo (Wind Waker - whimsy)'
  ],

  moodBalance: {
    adventurous: 0.35,
    epic: 0.25,
    whimsical: 0.20,
    mysterious: 0.15,
    tender: 0.05
  }
};
```

### 2.2 Instrumentation Palette

```typescript
const INSTRUMENTATION = {
  core: {
    strings: {
      usage: 'Primary melodic and harmonic foundation',
      instruments: ['violins', 'violas', 'cellos', 'basses'],
      style: 'Energetic, driving for action; lush for emotional moments'
    },
    brass: {
      usage: 'Heroic themes, fanfares, power moments',
      instruments: ['french_horns', 'trumpets', 'trombones'],
      style: 'Bold and triumphant, not militaristic'
    },
    woodwinds: {
      usage: 'Whimsy, wonder, lighter moments',
      instruments: ['flutes', 'clarinets', 'oboes', 'pennywhistle'],
      style: 'Playful and dancing'
    },
    percussion: {
      usage: 'Rhythm, energy, combat intensity',
      instruments: ['timpani', 'snare', 'bass_drum', 'cymbals'],
      style: 'Driving but not overwhelming'
    }
  },

  thematic: {
    nautical: {
      instruments: ['accordion', 'concertina', 'ship_bells', 'rope_creaks'],
      usage: 'Pirate theme reinforcement'
    },
    caribbean: {
      instruments: ['steel_drums', 'marimba', 'bongos', 'shakers'],
      usage: 'Tropical, festive moments'
    },
    celtic_folk: {
      instruments: ['bodhrán', 'tin_whistle', 'fiddle'],
      usage: 'Sea shanty influence'
    }
  },

  special: {
    mystical: ['harp', 'celesta', 'choir_pads'],
    tension: ['low_brass_drones', 'tremolo_strings', 'ticking'],
    victory: ['full_orchestra', 'chimes', 'triumphant_brass']
  }
};
```

### 2.3 Key Signatures and Tempo Guidelines

```typescript
const MUSIC_THEORY = {
  preferredKeys: {
    adventurous: ['D_major', 'G_major', 'A_major'],
    mysterious: ['E_minor', 'B_minor', 'D_minor'],
    epic: ['C_major', 'E_flat_major'],
    whimsical: ['F_major', 'B_flat_major']
  },

  tempoRanges: {
    lobby_menu: { bpm: '90-110', feel: 'Relaxed anticipation' },
    exploration: { bpm: '100-120', feel: 'Steady adventure' },
    combat_ambient: { bpm: '120-140', feel: 'Building tension' },
    intense_combat: { bpm: '140-170', feel: 'Heart-pounding action' },
    victory: { bpm: '130-150', feel: 'Triumphant celebration' },
    defeat: { bpm: '70-90', feel: 'Reflective, motivating' }
  },

  timeSignatures: {
    standard: '4/4', // Primary
    shanty: '6/8', // For folk-influenced sections
    waltz: '3/4' // Occasional whimsy
  }
};
```

### 2.4 Melodic Themes

```typescript
const MELODIC_THEMES = {
  mainTheme: {
    description: 'The core Plunderstorm identity',
    style: 'Bold, memorable, 4-bar phrase',
    usage: ['main_menu', 'victory_fanfare', 'trailer'],
    variations: ['full_orchestra', 'solo_pennywhistle', 'strings_only']
  },

  adventureMotif: {
    description: 'Quick ascending phrase suggesting journey',
    style: 'Uplifting, hopeful',
    usage: ['match_start', 'looting', 'positive_events']
  },

  dangerMotif: {
    description: 'Low, ominous phrase',
    style: 'Threatening, builds tension',
    usage: ['storm_approach', 'low_health', 'final_circles']
  },

  treasureMotif: {
    description: 'Sparkling, magical phrase',
    style: 'Whimsical, rewarding',
    usage: ['chest_open', 'rare_loot', 'level_up']
  },

  battleMotif: {
    description: 'Driving, rhythmic phrase',
    style: 'Aggressive, energetic',
    usage: ['combat_encounters', 'pvp_engagement']
  }
};
```

---

## 3. Sound Effects Direction

### 3.1 SFX Design Philosophy

```typescript
const SFX_PHILOSOPHY = {
  principles: {
    punchy: 'Quick attack, impactful transient, clean release',
    satisfying: 'Every action feels rewarding through sound',
    readable: 'Distinct sounds that players can identify quickly',
    themed: 'Sounds reinforce pirate fantasy appropriately'
  },

  characteristics: {
    attack: 'Fast (< 20ms)',
    body: 'Rich but brief (50-200ms typical)',
    tail: 'Minimal reverb (mobile clarity)',
    frequency: 'Mid-focused (cuts through on phone speakers)'
  },

  exaggeration: {
    level: 'Moderate', // Stylized but not cartoon
    reason: 'Mobile needs clarity over realism'
  }
};
```

### 3.2 Sound Categories and Tones

```typescript
const SFX_CATEGORIES = {
  combat: {
    tone: 'Impactful, metallic, visceral',
    characteristics: {
      meleeHits: 'Sharp attack, meaty body, slight ring',
      rangedFire: 'Punchy blast, slight boom tail',
      explosions: 'Big initial boom, crackle decay',
      abilities: 'Magical whoosh + elemental character'
    },
    avoidance: ['Overly realistic gore', 'Ear-piercing frequencies', 'Muddy bass']
  },

  movement: {
    tone: 'Responsive, physical, grounded',
    characteristics: {
      footsteps: 'Clear surface material, rhythmic',
      jumps: 'Effort grunt + landing thud',
      abilities: 'Whoosh + character (roll, grapple)'
    },
    importance: 'Critical for spatial awareness'
  },

  ui: {
    tone: 'Thematic, satisfying, non-intrusive',
    characteristics: {
      buttons: 'Subtle click with wood/metal character',
      navigation: 'Soft swoosh transitions',
      rewards: 'Celebratory coins, sparkles, fanfares'
    },
    constraint: 'Must not become annoying on repeat'
  },

  ambient: {
    tone: 'Immersive, natural, atmospheric',
    characteristics: {
      ocean: 'Waves, water lapping, distant calls',
      wildlife: 'Seabirds, insects, wind',
      environment: 'Creaking wood, flags, rope'
    },
    priority: 'Background layer, never dominates'
  },

  feedback: {
    tone: 'Clear, urgent when needed, informative',
    characteristics: {
      damage: 'Quick pain indicator',
      lowHealth: 'Heartbeat, warning tone',
      stormWarning: 'Ominous rumble, siren-like'
    },
    requirement: 'Must be audible over combat'
  }
};
```

### 3.3 Signature Sounds

```typescript
const SIGNATURE_SOUNDS = {
  // Sounds that define the Plunderstorm audio brand
  iconicSounds: {
    matchStart: {
      description: 'Ship bell + horn + adventure sting',
      duration: '2-3s',
      energy: 'Exciting, anticipatory'
    },
    elimination: {
      description: 'Skull crack + soul departure whoosh',
      duration: '0.5s',
      energy: 'Impactful, slightly dark'
    },
    victory: {
      description: 'Triumphant fanfare + treasure sound',
      duration: '3-4s',
      energy: 'Celebratory, rewarding'
    },
    chestOpen: {
      description: 'Lock click + lid creak + treasure sparkle',
      duration: '1-2s',
      energy: 'Exciting, mysterious'
    },
    abilityReady: {
      description: 'Magical chime + power surge',
      duration: '0.3s',
      energy: 'Ready, empowering'
    },
    stormApproach: {
      description: 'Thunder rumble + wind howl + danger tone',
      duration: '2-3s',
      energy: 'Threatening, urgent'
    }
  }
};
```

---

## 4. Ambient Audio Direction

### 4.1 Environment Soundscapes

```typescript
const AMBIENT_SOUNDSCAPES = {
  beach: {
    baseLayer: {
      waves: 'Gentle rolling waves, occasional crash',
      wind: 'Light tropical breeze',
      birds: 'Seagulls distant, occasional close fly-by'
    },
    details: ['Sand crunch', 'Palm rustle', 'Crab scuttle'],
    mood: 'Peaceful, tropical paradise'
  },

  jungle: {
    baseLayer: {
      foliage: 'Rustling leaves, dense vegetation',
      insects: 'Tropical insect chorus',
      birds: 'Exotic birds, parrots squawk'
    },
    details: ['Monkey calls', 'Frog croaks', 'Branch snaps'],
    mood: 'Mysterious, alive, slightly dangerous'
  },

  harbor: {
    baseLayer: {
      water: 'Water lapping against docks',
      ships: 'Creaking hulls, rope strain',
      activity: 'Distant voices, work sounds'
    },
    details: ['Bell chimes', 'Flag flap', 'Seagulls'],
    mood: 'Bustling, adventurous, starting point'
  },

  caves: {
    baseLayer: {
      echo: 'Cavernous reverb on sounds',
      water: 'Dripping, underground streams',
      air: 'Low wind through tunnels'
    },
    details: ['Bat squeaks', 'Rock falls', 'Mysterious tones'],
    mood: 'Mysterious, treasure-holding, slightly eerie'
  },

  ruins: {
    baseLayer: {
      wind: 'Howling through broken structures',
      debris: 'Settling stone, dust movement',
      supernatural: 'Faint whispers, ghostly tones'
    },
    details: ['Chain rattle', 'Door creak', 'Distant moans'],
    mood: 'Haunted, ancient, cursed'
  }
};
```

### 4.2 Dynamic Ambient Layers

```typescript
const DYNAMIC_AMBIENT = {
  layers: {
    base: {
      description: 'Constant environment tone',
      volume: 0.3,
      always_playing: true
    },
    activity: {
      description: 'Occasional sounds and events',
      volume: 0.5,
      frequency: 'variable',
      triggers: ['time', 'random', 'proximity']
    },
    reactive: {
      description: 'Responds to gameplay',
      volume: 0.6,
      triggers: ['combat', 'looting', 'storm']
    }
  },

  transitions: {
    zoneCrossfade: '2-3s',
    intensityRamp: '1-2s',
    combatDucking: '0.5s'
  },

  weatherEffects: {
    storm: {
      thunder: 'Distant rumbles building to close cracks',
      rain: 'Increasing intensity',
      wind: 'Howling gusts'
    }
  }
};
```

---

## 5. Technical Audio Specifications

### 5.1 Format Requirements

```typescript
const AUDIO_FORMATS = {
  music: {
    format: 'OGG Vorbis',
    quality: 128, // kbps (mobile balance)
    channels: 'stereo',
    sampleRate: 44100
  },

  sfx: {
    format: 'OGG Vorbis',
    quality: 96, // kbps
    channels: 'mono', // Most SFX
    channels_stereo: ['ambience', 'ui_whoosh'],
    sampleRate: 44100
  },

  ambient: {
    format: 'OGG Vorbis',
    quality: 96,
    channels: 'stereo',
    sampleRate: 44100,
    looping: true
  },

  voiceover: {
    format: 'OGG Vorbis',
    quality: 64, // Lower for speech
    channels: 'mono',
    sampleRate: 22050 // Lower sample rate acceptable
  }
};
```

### 5.2 File Size Budgets

```typescript
const AUDIO_BUDGETS = {
  total: '50MB', // Maximum audio package

  breakdown: {
    music: {
      budget: '25MB',
      typical_track: '2-3MB',
      tracks_count: 8
    },
    sfx: {
      budget: '15MB',
      typical_effect: '10-50KB',
      effects_count: '300+'
    },
    ambient: {
      budget: '8MB',
      loops: '500KB-1MB each'
    },
    ui: {
      budget: '2MB',
      typical_sound: '5-20KB'
    }
  },

  streaming: {
    music: true, // Stream from storage
    sfx: false, // Load into memory
    ambient: true // Stream loops
  }
};
```

### 5.3 Mixing Guidelines

```typescript
const MIXING_GUIDELINES = {
  masterLevels: {
    music: -12, // dB (base reference)
    sfx: -6, // Louder for clarity
    ambient: -18, // Background layer
    ui: -9,
    voice: -6 // Clear and present
  },

  ducking: {
    combatMusic: {
      trigger: 'combat_active',
      reduction: -6, // dB
      attackTime: 0.3, // seconds
      releaseTime: 1.0
    },
    importantEvents: {
      trigger: ['victory', 'defeat', 'announcement'],
      reduction: -12,
      attackTime: 0.1,
      releaseTime: 0.5
    }
  },

  priorities: {
    1: ['player_damage', 'elimination', 'low_health'],
    2: ['ability_fire', 'ability_hit', 'chest_open'],
    3: ['combat_sfx', 'enemy_attacks'],
    4: ['movement', 'ambient_details'],
    5: ['background_ambient', 'music']
  },

  polyphony: {
    maxVoices: 32, // Simultaneous sounds
    perSoundLimit: 3, // Same sound concurrently
    cullingDistance: 50 // Meters
  },

  compression: {
    master: {
      threshold: -12,
      ratio: 3,
      attack: 10, // ms
      release: 100
    },
    limiter: {
      ceiling: -0.5 // dB
    }
  }
};
```

### 5.4 Spatialization

```typescript
const SPATIAL_AUDIO = {
  mode: '3D_positioned',

  settings: {
    rolloff: 'logarithmic',
    minDistance: 1, // meters
    maxDistance: 50,
    dopplerLevel: 0 // Disabled for mobile
  },

  categories: {
    positioned: ['combat', 'enemy', 'world_objects'],
    headRelative: ['ui', 'feedback', 'announcements'],
    ambient: '2D_stereo'
  },

  occlusion: {
    enabled: false, // Too expensive for mobile
    simplified: true // Basic line-of-sight check
  },

  directionality: {
    enabled: true,
    importance: 'high', // Enemy positions
    indicatorSounds: ['footsteps', 'abilities', 'gunfire']
  }
};
```

---

## 6. Audio Feedback Systems

### 6.1 Combat Audio Feedback

```typescript
const COMBAT_AUDIO_FEEDBACK = {
  damageDealt: {
    hitMarker: {
      sound: 'hit_confirm_tick',
      variation: 'pitch_by_damage',
      volume: 'consistent'
    },
    criticalHit: {
      sound: 'critical_hit_sting',
      additional: 'screen_punch_sound'
    },
    killConfirm: {
      sound: 'elimination_sound',
      style: 'Satisfying, clear'
    }
  },

  damageReceived: {
    hit: {
      sound: 'player_hurt_grunt',
      variation: 'random_from_set',
      layered: 'impact_thud'
    },
    lowHealth: {
      sound: 'heartbeat_pulse',
      trigger: 'health < 25%',
      loop: true,
      intensity: 'increases_with_damage'
    },
    nearDeath: {
      sound: 'flatline_warning',
      trigger: 'health < 10%',
      urgent: true
    }
  },

  abilityFeedback: {
    ready: {
      sound: 'ability_ready_chime',
      subtle: true
    },
    cooldown: {
      sound: 'ability_tick_progress',
      optional: true
    },
    failed: {
      sound: 'ability_fail_buzz',
      reason: 'out_of_range / on_cooldown'
    }
  }
};
```

### 6.2 Progression Audio Feedback

```typescript
const PROGRESSION_AUDIO = {
  levelUp: {
    sound: 'level_up_fanfare',
    duration: '2s',
    style: 'Triumphant, rewarding',
    layers: ['brass_sting', 'sparkle_rise', 'drum_hit']
  },

  xpGain: {
    sound: 'xp_tick',
    style: 'Subtle, satisfying',
    variation: 'pitch_increases_with_amount'
  },

  currencyGain: {
    gold: {
      sound: 'coin_collect',
      layers: ['coin_clink', 'pouch_jingle'],
      stacking: true // Multiple coins = richer sound
    },
    premium: {
      sound: 'premium_currency_gain',
      style: 'More magical, special',
      layers: ['gem_sparkle', 'magical_chime']
    }
  },

  achievement: {
    unlock: {
      sound: 'achievement_unlock_fanfare',
      duration: '1.5s',
      style: 'Celebratory'
    },
    progress: {
      sound: 'achievement_progress_tick',
      subtle: true
    }
  },

  battlePass: {
    tierUnlock: {
      sound: 'tier_unlock_celebration',
      style: 'Exciting, rewarding'
    },
    premiumUnlock: {
      sound: 'premium_tier_fanfare',
      enhanced: true,
      style: 'Extra special, valuable'
    }
  }
};
```

---

## 7. Contextual Audio Guidelines

### 7.1 Match Phase Audio

```typescript
const MATCH_PHASE_AUDIO = {
  preGame: {
    music: 'lobby_anticipation',
    ambient: 'ship_preparation',
    sfx: 'normal_responsiveness',
    mood: 'Building excitement'
  },

  earlyGame: {
    music: 'exploration_theme',
    ambient: 'full_environment',
    sfx: 'normal',
    mood: 'Adventure, discovery'
  },

  midGame: {
    music: 'tension_building',
    ambient: 'reduced_peaceful',
    sfx: 'heightened_combat',
    mood: 'Stakes rising'
  },

  lateGame: {
    music: 'intense_combat',
    ambient: 'storm_dominant',
    sfx: 'maximum_impact',
    mood: 'Life or death'
  },

  finalCircle: {
    music: 'climactic_battle',
    ambient: 'minimal',
    sfx: 'enhanced_all',
    mood: 'Heart-pounding finale'
  },

  victory: {
    music: 'victory_fanfare',
    sfx: 'celebration',
    duration: '5-10s',
    mood: 'Triumphant elation'
  },

  defeat: {
    music: 'defeat_somber',
    sfx: 'minimal',
    duration: '3-5s',
    mood: 'Reflective but motivating'
  }
};
```

### 7.2 UI Audio Guidelines

```typescript
const UI_AUDIO_GUIDELINES = {
  navigation: {
    buttonPress: {
      style: 'Quick, satisfying click',
      thematic: 'Wood knock / metal tap',
      volume: 'Moderate',
      repetition: 'Must not annoy'
    },
    screenTransition: {
      style: 'Subtle whoosh',
      thematic: 'Sail unfurl / wave',
      volume: 'Low',
      duration: '0.2-0.4s'
    },
    menuOpen: {
      style: 'Unfolding sound',
      thematic: 'Map unroll / door creak',
      volume: 'Low-moderate'
    }
  },

  feedback: {
    success: {
      style: 'Positive chime',
      thematic: 'Ship bell / treasure sparkle',
      volume: 'Moderate'
    },
    error: {
      style: 'Gentle negative',
      thematic: 'Dull thud / rope snap',
      volume: 'Low-moderate',
      notPunishing: true
    },
    warning: {
      style: 'Attention-getting',
      thematic: 'Crow call / alert horn',
      volume: 'Moderate-high'
    }
  },

  shop: {
    purchase: {
      style: 'Satisfying transaction',
      layers: ['coin_exchange', 'success_chime'],
      volume: 'Moderate'
    },
    preview: {
      style: 'Subtle selection',
      volume: 'Low'
    }
  }
};
```

---

## 8. Platform Considerations

### 8.1 Mobile-Specific Guidelines

```typescript
const MOBILE_AUDIO = {
  speakerOptimization: {
    frequencyFocus: {
      primary: '500Hz - 4kHz', // Phone speaker sweet spot
      avoid: 'Heavy sub-bass (< 100Hz)',
      boost: 'Upper mids for clarity'
    },
    dynamicRange: {
      compressed: true,
      reason: 'Noisy environments, small speakers'
    }
  },

  headphoneSupport: {
    spatialAudio: true,
    bassExtension: 'enabled_when_detected',
    dynamicRange: 'expanded'
  },

  silentMode: {
    respectSystemSettings: true,
    vibrateAlternative: 'optional'
  },

  interruptions: {
    phoneCalls: 'pause_all',
    notifications: 'duck_briefly',
    backgrounding: 'pause_or_mute'
  },

  battery: {
    optimizedProcessing: true,
    reducedPolyphony: 'on_low_battery'
  }
};
```

### 8.2 Accessibility

```typescript
const AUDIO_ACCESSIBILITY = {
  volumeControls: {
    master: { default: 80, range: '0-100' },
    music: { default: 70, range: '0-100' },
    sfx: { default: 100, range: '0-100' },
    voice: { default: 100, range: '0-100' }
  },

  visualAlternatives: {
    subtitles: true,
    soundIndicators: 'directional_icons', // For deaf players
    combatFeedback: 'screen_flash_option'
  },

  monoAudio: {
    available: true,
    reason: 'Single-ear listening'
  },

  reducedAudio: {
    available: true,
    removes: ['ambient_details', 'musical_layers'],
    keeps: ['critical_feedback', 'UI']
  }
};
```

---

## 9. Reference Tracks and Inspirations

### 9.1 Musical References

```typescript
const MUSIC_REFERENCES = {
  adventure: [
    { title: 'He\'s a Pirate', source: 'Pirates of the Caribbean', notes: 'Energy, theme recognition' },
    { title: 'Concerning Hobbits', source: 'Lord of the Rings', notes: 'Warm, adventurous feel' },
    { title: 'Dragon Roost Island', source: 'Wind Waker', notes: 'Whimsy, oceanic theme' }
  ],

  combat: [
    { title: 'One Final Effort', source: 'Halo 3', notes: 'Driving, heroic combat' },
    { title: 'Ezio\'s Family', source: 'Assassin\'s Creed', notes: 'Emotional intensity' }
  ],

  ambient: [
    { title: 'Ambience tracks', source: 'Sea of Thieves', notes: 'Oceanic atmosphere' },
    { title: 'Main Theme', source: 'Subnautica', notes: 'Underwater mystery' }
  ],

  ui_reward: [
    { title: 'Victory Fanfare', source: 'Final Fantasy', notes: 'Iconic reward sound' },
    { title: 'Level Up', source: 'Overwatch', notes: 'Satisfying progression' }
  ]
};
```

### 9.2 SFX References

```typescript
const SFX_REFERENCES = {
  combat: [
    { game: 'Fortnite', notes: 'Punchy, readable, satisfying hits' },
    { game: 'Overwatch', notes: 'Ability sound design, clarity' },
    { game: 'Apex Legends', notes: 'Weapon sounds, impact feedback' }
  ],

  ui: [
    { game: 'Hearthstone', notes: 'Thematic, satisfying, non-annoying' },
    { game: 'Clash Royale', notes: 'Mobile-optimized, clear feedback' }
  ],

  movement: [
    { game: 'Sea of Thieves', notes: 'Nautical character, footsteps' },
    { game: 'Breath of the Wild', notes: 'Environmental responsiveness' }
  ]
};
```

---

## 10. Implementation Checklist

### 10.1 Pre-Production
- [ ] Audio style guide approved by stakeholders
- [ ] Reference library compiled
- [ ] Audio engine selected and configured
- [ ] File naming conventions established
- [ ] Asset pipeline set up

### 10.2 Production
- [ ] Music tracks composed/licensed
- [ ] Core SFX created and implemented
- [ ] Ambient soundscapes layered
- [ ] UI sounds designed
- [ ] Mixing pass completed

### 10.3 Polish
- [ ] Dynamic audio systems tested
- [ ] Volume balancing refined
- [ ] Performance optimization
- [ ] Accessibility features verified
- [ ] Final mix approved

---

## Appendix A: File Naming Convention

```
audio_[category]_[subcategory]_[name]_[variation].ogg

Examples:
audio_music_combat_intense_01.ogg
audio_sfx_weapon_cutlass_swing_01.ogg
audio_ambient_beach_waves_loop.ogg
audio_ui_button_press_standard.ogg
```

---

## Appendix B: Volume Reference Chart

| Category | dB Level | Notes |
|----------|----------|-------|
| Master | 0 dB | Reference |
| Music (quiet) | -18 dB | During combat |
| Music (full) | -12 dB | Menus, exploration |
| Combat SFX | -6 dB | Prominent |
| Movement SFX | -12 dB | Background |
| UI | -9 dB | Clear but not loud |
| Ambient | -18 dB | Subtle layer |
| Voice/Announcements | -6 dB | Always clear |
| Critical Feedback | -3 dB | Must be heard |

---

## Appendix C: Glossary

| Term | Definition |
|------|------------|
| Attack | Initial transient of a sound |
| Ducking | Lowering volume of one sound when another plays |
| Polyphony | Number of simultaneous sounds |
| Stem | Individual musical component (e.g., drums only) |
| Stinger | Short musical accent for events |
| Sweetener | Additional layer to enhance a sound |
| Tail | Decay/reverb portion of a sound |
