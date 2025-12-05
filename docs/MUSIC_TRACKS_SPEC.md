# Music Tracks Specification

## Document Information
- **Task ID:** ART-026
- **Priority:** P1
- **Complexity:** M
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines all music tracks required for Plunderstorm Mobile at launch. Each track is detailed with mood, instrumentation, structure, and technical requirements to guide composition or licensing.

### 1.2 Music Strategy
- **8 Core Tracks:** Covering all major game contexts
- **Dynamic Layering:** Tracks support intensity transitions
- **Thematic Unity:** Shared motifs connect the soundtrack
- **Memory Efficient:** Optimized for mobile storage

### 1.3 Related Documents
- Audio Style Guide (ART-025)
- Sound Effects Specification (ART-027)

---

## 2. Track List Summary

| ID | Track Name | Context | Duration | Priority |
|----|------------|---------|----------|----------|
| M-01 | Plunderstorm Main Theme | Main Menu | 2:30 | Critical |
| M-02 | Awaiting Adventure | Lobby/Matchmaking | 1:30 | Critical |
| M-03 | Shores of Fortune | In-Match Ambient | 3:00 | Critical |
| M-04 | Storm's Fury | Final Circle | 2:00 | Critical |
| M-05 | Victory Triumphant | Victory Screen | 0:45 | Critical |
| M-06 | Fallen, Not Forgotten | Defeat Screen | 0:30 | Critical |
| M-07 | Trader's Cove | Shop Music | 2:00 | High |
| M-08 | Path of Plunder | Battle Pass | 1:45 | High |

**Total Duration:** ~14 minutes
**Estimated Size:** ~20-25MB (compressed)

---

## 3. Track Specifications

### 3.1 M-01: Plunderstorm Main Theme

```typescript
const MAIN_THEME = {
  id: 'M-01',
  name: 'Plunderstorm Main Theme',
  context: 'Main Menu',

  overview: {
    mood: 'Adventurous, heroic, inviting',
    energy: 'Medium-high, building',
    emotion: 'Excitement, anticipation, belonging',
    duration: '2:30',
    looping: true,
    loopPoint: '0:08' // After intro, seamless loop
  },

  structure: {
    intro: {
      duration: '0:00-0:08',
      description: 'Ocean wave ambience fading into first notes',
      instruments: ['solo_pennywhistle', 'harp_arpeggios'],
      dynamic: 'Soft, mysterious'
    },
    verseA: {
      duration: '0:08-0:40',
      description: 'Main theme statement, warm and welcoming',
      instruments: ['strings_melody', 'french_horns', 'light_percussion'],
      dynamic: 'Medium, building warmth',
      motif: 'Main Plunderstorm theme'
    },
    bridge: {
      duration: '0:40-1:00',
      description: 'Adventurous transition, sense of journey',
      instruments: ['full_strings', 'brass_fanfare', 'timpani'],
      dynamic: 'Building energy'
    },
    verseB: {
      duration: '1:00-1:40',
      description: 'Theme variation with Caribbean influence',
      instruments: ['steel_drums', 'accordion', 'driving_rhythm'],
      dynamic: 'High energy, festive',
      additionalElements: ['sea_shanty_feel']
    },
    climax: {
      duration: '1:40-2:10',
      description: 'Full orchestral statement of theme',
      instruments: ['full_orchestra', 'choir_pad', 'epic_percussion'],
      dynamic: 'Triumphant, powerful'
    },
    outro: {
      duration: '2:10-2:30',
      description: 'Graceful descent back to loop point',
      instruments: ['strings_descending', 'harp', 'solo_flute'],
      dynamic: 'Settling, ready to loop'
    }
  },

  instrumentation: {
    melody: ['french_horns', 'violins', 'pennywhistle'],
    harmony: ['violas', 'cellos', 'choir_pad'],
    rhythm: ['snare', 'bass_drum', 'shaker', 'tambourine'],
    color: ['harp', 'celesta', 'steel_drums', 'accordion']
  },

  technicalNotes: {
    key: 'D Major',
    tempo: '110 BPM',
    timeSignature: '4/4',
    loopSeamless: true,
    fadeInDuration: 0,
    fadeOutDuration: 2.0
  }
};
```

---

### 3.2 M-02: Awaiting Adventure

```typescript
const LOBBY_MUSIC = {
  id: 'M-02',
  name: 'Awaiting Adventure',
  context: 'Lobby / Matchmaking',

  overview: {
    mood: 'Anticipatory, friendly, slightly tense',
    energy: 'Low-medium, steady',
    emotion: 'Ready to play, social, building excitement',
    duration: '1:30',
    looping: true,
    loopPoint: '0:04'
  },

  structure: {
    intro: {
      duration: '0:00-0:04',
      description: 'Ship ambience transition',
      instruments: ['subtle_waves', 'rigging_creak'],
      dynamic: 'Ambient'
    },
    loopSection: {
      duration: '0:04-1:26',
      description: 'Steady, friendly groove with pirate character',
      subsections: [
        {
          name: 'A_section',
          duration: '0:04-0:45',
          instruments: ['acoustic_guitar', 'light_percussion', 'bass'],
          dynamic: 'Relaxed groove'
        },
        {
          name: 'B_section',
          duration: '0:45-1:10',
          instruments: ['add_accordion', 'fiddle_ornaments'],
          dynamic: 'Slightly more energy'
        },
        {
          name: 'A_return',
          duration: '1:10-1:26',
          instruments: ['back_to_basics'],
          dynamic: 'Settling for loop'
        }
      ]
    },
    loopTransition: {
      duration: '1:26-1:30',
      description: 'Seamless return to loop point',
      crossfade: true
    }
  },

  dynamicLayers: {
    enabled: true,
    layers: [
      {
        name: 'base',
        instruments: ['rhythm_section', 'bass'],
        alwaysPlaying: true
      },
      {
        name: 'party_forming',
        instruments: ['add_melody', 'accordion'],
        trigger: 'party_size >= 2'
      },
      {
        name: 'ready_to_go',
        instruments: ['full_arrangement', 'builds'],
        trigger: 'all_ready'
      }
    ]
  },

  instrumentation: {
    melody: ['accordion', 'fiddle', 'pennywhistle'],
    harmony: ['acoustic_guitar', 'ukulele'],
    rhythm: ['cajon', 'shaker', 'foot_stomp'],
    bass: ['upright_bass']
  },

  technicalNotes: {
    key: 'G Major',
    tempo: '100 BPM',
    timeSignature: '4/4',
    dynamicLayersCount: 3
  }
};
```

---

### 3.3 M-03: Shores of Fortune

```typescript
const MATCH_AMBIENT = {
  id: 'M-03',
  name: 'Shores of Fortune',
  context: 'In-Match Ambient / Exploration',

  overview: {
    mood: 'Adventurous, open, subtly tense',
    energy: 'Low-medium, adapts to gameplay',
    emotion: 'Exploration, possibility, underlying danger',
    duration: '3:00',
    looping: true,
    loopPoint: '0:00',
    dynamicIntensity: true
  },

  structure: {
    fullTrack: {
      description: 'Continuous ambient bed with intensity layers',
      baseLayer: {
        duration: '3:00',
        instruments: ['pad_synth', 'distant_strings', 'subtle_percussion'],
        dynamic: 'Quiet, atmospheric'
      },
      intensityLayers: [
        {
          level: 0,
          name: 'peaceful',
          elements: ['base_only'],
          triggers: ['no_combat', 'safe_zone']
        },
        {
          level: 1,
          name: 'exploring',
          elements: ['add_melody_fragments', 'light_rhythm'],
          triggers: ['moving', 'looting']
        },
        {
          level: 2,
          name: 'tension',
          elements: ['darker_harmony', 'pulse_bass', 'ticking'],
          triggers: ['enemy_nearby', 'storm_approaching']
        },
        {
          level: 3,
          name: 'combat',
          elements: ['driving_rhythm', 'brass_stabs', 'intense_strings'],
          triggers: ['in_combat', 'taking_damage']
        }
      ]
    }
  },

  dynamicSystem: {
    transitionTime: 2.0, // Seconds between intensity levels
    combatDetection: {
      enterCombat: 'instant',
      exitCombat: '5s_delay' // Stay intense briefly after combat
    },
    stormInfluence: {
      enabled: true,
      addsLayer: 'storm_rumble',
      intensityBoost: 1
    }
  },

  instrumentation: {
    baseLayer: ['warm_pad', 'cellos_sustained', 'harp_arpeggios'],
    melodyFragments: ['solo_oboe', 'horn_calls', 'pennywhistle'],
    tensionElements: ['low_brass', 'tremolo_strings', 'heartbeat_pulse'],
    combatElements: ['full_strings', 'brass_fanfare', 'war_drums']
  },

  technicalNotes: {
    key: 'A minor / C Major (shifts)',
    tempo: '90 BPM (flexible)',
    stemCount: 8,
    separateStemFiles: true
  }
};
```

---

### 3.4 M-04: Storm's Fury

```typescript
const FINAL_CIRCLE = {
  id: 'M-04',
  name: 'Storm\'s Fury',
  context: 'Final Circle / Endgame',

  overview: {
    mood: 'Intense, desperate, climactic',
    energy: 'High, relentless',
    emotion: 'Heart-pounding tension, do-or-die',
    duration: '2:00',
    looping: true,
    loopPoint: '0:10'
  },

  structure: {
    intro: {
      duration: '0:00-0:10',
      description: 'Dramatic transition sting',
      instruments: ['brass_hit', 'timpani_roll', 'string_crescendo'],
      dynamic: 'Explosive entrance'
    },
    mainLoop: {
      duration: '0:10-1:50',
      description: 'Relentless driving combat music',
      subsections: [
        {
          name: 'driving_section',
          duration: '0:10-0:50',
          instruments: ['pounding_percussion', 'aggressive_strings', 'brass_stabs'],
          dynamic: 'Intense, driving'
        },
        {
          name: 'melodic_break',
          duration: '0:50-1:10',
          instruments: ['heroic_horn', 'strings_counter', 'maintained_rhythm'],
          dynamic: 'Brief heroic moment'
        },
        {
          name: 'intensify',
          duration: '1:10-1:50',
          instruments: ['full_force', 'added_choir', 'double_time_elements'],
          dynamic: 'Maximum intensity'
        }
      ]
    },
    loopTransition: {
      duration: '1:50-2:00',
      description: 'Cymbal crash reset to loop',
      instruments: ['cymbal_crash', 'quick_build'],
      seamless: true
    }
  },

  intensityVariation: {
    enabled: true,
    playerCountInfluence: {
      many_alive: 'standard_intensity',
      few_alive: 'increased_intensity',
      final_two: 'maximum_intensity'
    }
  },

  instrumentation: {
    rhythm: ['taiko_drums', 'snare_rolls', 'bass_drum_hits'],
    strings: ['aggressive_ostinato', 'tremolo_high', 'low_power_chords'],
    brass: ['horn_stabs', 'trombone_hits', 'trumpet_fanfare'],
    color: ['choir_pad', 'synth_bass', 'anvil_hits']
  },

  technicalNotes: {
    key: 'E minor',
    tempo: '150 BPM',
    timeSignature: '4/4',
    energyLevel: 'maximum'
  }
};
```

---

### 3.5 M-05: Victory Triumphant

```typescript
const VICTORY_THEME = {
  id: 'M-05',
  name: 'Victory Triumphant',
  context: 'Victory Screen',

  overview: {
    mood: 'Triumphant, celebratory, rewarding',
    energy: 'High, jubilant',
    emotion: 'Achievement, glory, satisfaction',
    duration: '0:45',
    looping: false,
    oneShot: true
  },

  structure: {
    fanfare: {
      duration: '0:00-0:08',
      description: 'Triumphant brass fanfare',
      instruments: ['trumpet_fanfare', 'french_horns', 'timpani_hit'],
      dynamic: 'Explosive, glorious'
    },
    celebration: {
      duration: '0:08-0:25',
      description: 'Joyful main theme statement',
      instruments: ['full_orchestra', 'tambourine', 'bells'],
      dynamic: 'Celebratory, warm',
      motif: 'Main theme in major, triumphant arrangement'
    },
    resolution: {
      duration: '0:25-0:40',
      description: 'Warm conclusion, sense of accomplishment',
      instruments: ['strings_warm', 'harp_flourish', 'soft_brass'],
      dynamic: 'Settling, satisfied'
    },
    sting: {
      duration: '0:40-0:45',
      description: 'Final punctuation',
      instruments: ['orchestra_hit', 'cymbal_shimmer'],
      dynamic: 'Definitive ending'
    }
  },

  variations: {
    firstPlace: {
      fullTrack: true,
      extraElements: ['choir_ahs', 'extended_fanfare']
    },
    topThree: {
      shortened: '0:30',
      reduced: ['no_choir', 'simpler_fanfare']
    },
    topTen: {
      shortened: '0:20',
      mood: 'Still positive, less grandiose'
    }
  },

  instrumentation: {
    fanfare: ['trumpets', 'french_horns', 'trombones', 'timpani'],
    celebration: ['full_strings', 'woodwinds', 'percussion', 'choir'],
    resolution: ['violins', 'cellos', 'harp', 'soft_brass']
  },

  technicalNotes: {
    key: 'D Major',
    tempo: '130 BPM',
    noLoop: true,
    transitionTo: 'lobby_music'
  }
};
```

---

### 3.6 M-06: Fallen, Not Forgotten

```typescript
const DEFEAT_THEME = {
  id: 'M-06',
  name: 'Fallen, Not Forgotten',
  context: 'Defeat Screen',

  overview: {
    mood: 'Reflective, somber but not crushing',
    energy: 'Low, gentle',
    emotion: 'Acceptance, motivation to try again',
    duration: '0:30',
    looping: false,
    oneShot: true
  },

  structure: {
    opening: {
      duration: '0:00-0:10',
      description: 'Gentle, somber statement',
      instruments: ['solo_cello', 'soft_piano'],
      dynamic: 'Quiet, reflective'
    },
    lift: {
      duration: '0:10-0:22',
      description: 'Subtle shift toward hope',
      instruments: ['add_strings', 'warm_pad', 'gentle_rhythm'],
      dynamic: 'Warming, encouraging',
      motif: 'Main theme fragment in minor, resolving to major'
    },
    resolve: {
      duration: '0:22-0:30',
      description: 'Motivating conclusion',
      instruments: ['strings_swell', 'hopeful_horn'],
      dynamic: 'Uplifting ending'
    }
  },

  designPhilosophy: {
    notPunishing: true,
    encouragesRetry: true,
    briefDuration: 'respects player time',
    emotionalArc: 'disappointment -> acceptance -> motivation'
  },

  instrumentation: {
    melody: ['solo_cello', 'french_horn'],
    harmony: ['strings_ensemble', 'piano_chords'],
    color: ['soft_choir_pad', 'gentle_bells']
  },

  technicalNotes: {
    key: 'E minor -> G Major',
    tempo: '75 BPM',
    noLoop: true,
    transitionTo: 'lobby_music'
  }
};
```

---

### 3.7 M-07: Trader's Cove

```typescript
const SHOP_MUSIC = {
  id: 'M-07',
  name: 'Trader\'s Cove',
  context: 'Shop / Store',

  overview: {
    mood: 'Inviting, mysterious, slightly exotic',
    energy: 'Low-medium, relaxed',
    emotion: 'Browsing pleasure, temptation, wonder',
    duration: '2:00',
    looping: true,
    loopPoint: '0:00'
  },

  structure: {
    fullLoop: {
      duration: '2:00',
      description: 'Relaxed, enticing loop for browsing',
      sections: [
        {
          name: 'mysterious_intro',
          duration: '0:00-0:30',
          instruments: ['dulcimer', 'soft_percussion', 'mystical_pad'],
          dynamic: 'Quiet, intriguing'
        },
        {
          name: 'main_groove',
          duration: '0:30-1:15',
          instruments: ['acoustic_guitar', 'tabla', 'sitar_ornaments', 'bass'],
          dynamic: 'Gentle groove, exotic feel'
        },
        {
          name: 'treasure_theme',
          duration: '1:15-1:45',
          instruments: ['add_strings', 'harp_arpeggios', 'chimes'],
          dynamic: 'Warm, valuable feeling',
          motif: 'Treasure motif'
        },
        {
          name: 'return',
          duration: '1:45-2:00',
          instruments: ['settling_back'],
          dynamic: 'Preparing for loop'
        }
      ]
    }
  },

  premiumSection: {
    trigger: 'viewing_premium_items',
    enhancement: {
      addLayer: 'magical_shimmer',
      intensify: 'treasure_theme',
      sparkle: 'harp_glissandos'
    }
  },

  instrumentation: {
    exotic: ['sitar', 'tabla', 'dulcimer', 'oud'],
    western: ['acoustic_guitar', 'cello', 'harp'],
    atmospheric: ['mystical_pad', 'chimes', 'bells']
  },

  technicalNotes: {
    key: 'B minor',
    tempo: '85 BPM',
    timeSignature: '4/4',
    seamlessLoop: true
  }
};
```

---

### 3.8 M-08: Path of Plunder

```typescript
const BATTLE_PASS_MUSIC = {
  id: 'M-08',
  name: 'Path of Plunder',
  context: 'Battle Pass Screen',

  overview: {
    mood: 'Exciting, progressive, rewarding',
    energy: 'Medium, building',
    emotion: 'Journey, progression, treasure ahead',
    duration: '1:45',
    looping: true,
    loopPoint: '0:05'
  },

  structure: {
    intro: {
      duration: '0:00-0:05',
      description: 'Map unfurl sound',
      instruments: ['parchment_rustle', 'compass_tick'],
      dynamic: 'Thematic transition'
    },
    journey: {
      duration: '0:05-0:50',
      description: 'Adventurous exploration theme',
      instruments: ['strings_melody', 'pennywhistle', 'bodhrán'],
      dynamic: 'Steady adventure'
    },
    discovery: {
      duration: '0:50-1:20',
      description: 'Building excitement, treasure ahead',
      instruments: ['add_brass', 'fuller_arrangement', 'building_percussion'],
      dynamic: 'Growing anticipation'
    },
    treasure: {
      duration: '1:20-1:40',
      description: 'Treasure theme statement',
      instruments: ['full_orchestra_moment', 'choir_pad', 'harp_sparkle'],
      dynamic: 'Rewarding climax',
      motif: 'Treasure motif, triumphant'
    },
    settle: {
      duration: '1:40-1:45',
      description: 'Return to loop',
      instruments: ['settling_strings'],
      dynamic: 'Ready to loop'
    }
  },

  tierUnlockStinger: {
    duration: '2s',
    instruments: ['brass_fanfare', 'sparkle', 'drum_hit'],
    plays_over: true,
    ducking: 'music_ducks_50%'
  },

  instrumentation: {
    adventure: ['strings', 'pennywhistle', 'bodhrán', 'accordion'],
    discovery: ['french_horns', 'brass_section', 'timpani'],
    treasure: ['full_orchestra', 'harp', 'celesta', 'choir']
  },

  technicalNotes: {
    key: 'F Major',
    tempo: '105 BPM',
    timeSignature: '4/4'
  }
};
```

---

## 4. Shared Musical Elements

### 4.1 Main Theme Motif

```typescript
const MAIN_MOTIF = {
  name: 'Plunderstorm Main Motif',
  notes: 'D - F# - A - B - A - G - F# - D',
  rhythm: 'quarter - eighth - eighth - half - quarter - quarter - half - whole',
  character: 'Heroic, adventurous, memorable',

  appearances: {
    mainTheme: { key: 'D Major', full_statement: true },
    victoryTheme: { key: 'D Major', triumphant: true },
    defeatTheme: { key: 'E minor', fragment: true, resolves_major: true },
    matchAmbient: { key: 'Various', subtle_hints: true },
    battlePass: { key: 'F Major', journey_version: true }
  }
};
```

### 4.2 Treasure Motif

```typescript
const TREASURE_MOTIF = {
  name: 'Treasure Motif',
  notes: 'Rising arpeggio with sparkle ornament',
  character: 'Magical, valuable, exciting',

  appearances: {
    shopMusic: 'Background element',
    battlePass: 'Climax moment',
    chestOpenSting: 'Standalone stinger',
    rewardScreens: 'Celebration element'
  }
};
```

### 4.3 Danger Motif

```typescript
const DANGER_MOTIF = {
  name: 'Danger Motif',
  notes: 'Low brass descending tritone',
  character: 'Threatening, ominous',

  appearances: {
    stormApproach: 'Building presence',
    finalCircle: 'Underlying tension',
    matchAmbient: 'Tension layers',
    bossEncounter: 'Emphasis'
  }
};
```

---

## 5. Technical Requirements

### 5.1 File Specifications

```typescript
const MUSIC_TECH_SPECS = {
  format: {
    delivery: 'OGG Vorbis',
    quality: 128, // kbps
    sampleRate: 44100,
    channels: 'Stereo'
  },

  stemDelivery: {
    matchAmbient: true, // Requires stems for dynamic layering
    stems: ['base', 'melody', 'tension', 'combat'],
    format: 'Same as full tracks'
  },

  looping: {
    metadata: 'LOOPSTART/LOOPLENGTH tags',
    seamless: 'Zero-crossing at loop points',
    crossfade: 'Not needed if properly authored'
  },

  volume: {
    peakLevel: -1, // dB
    averageLevel: -12, // LUFS
    dynamicRange: 8 // dB
  }
};
```

### 5.2 Memory Budget

```typescript
const MUSIC_MEMORY = {
  totalBudget: '25MB',

  perTrack: {
    'M-01_MainTheme': '3.5MB',
    'M-02_Lobby': '2.0MB',
    'M-03_Match': '4.5MB', // Includes stems
    'M-04_FinalCircle': '3.0MB',
    'M-05_Victory': '1.0MB',
    'M-06_Defeat': '0.5MB',
    'M-07_Shop': '3.0MB',
    'M-08_BattlePass': '2.5MB'
  },

  streaming: {
    enabled: true,
    bufferSize: '512KB',
    preloadTime: 2.0 // Seconds
  }
};
```

### 5.3 Transition System

```typescript
const MUSIC_TRANSITIONS = {
  types: {
    crossfade: {
      duration: '2s',
      curve: 'equal_power',
      usage: ['menu_to_menu', 'ambient_changes']
    },
    stinger: {
      plays_over: true,
      ducks_current: true,
      usage: ['match_start', 'victory', 'defeat']
    },
    immediate: {
      duration: 0,
      usage: ['combat_enter', 'emergency']
    }
  },

  contextTransitions: {
    mainMenu_to_lobby: { type: 'crossfade', duration: 1.5 },
    lobby_to_match: { type: 'stinger', stinger: 'match_start_sting' },
    match_to_victory: { type: 'stinger', stinger: 'victory_fanfare' },
    match_to_defeat: { type: 'crossfade', duration: 0.5 },
    anyMenu_to_shop: { type: 'crossfade', duration: 2.0 }
  }
};
```

---

## 6. Dynamic Music System

### 6.1 In-Match Music Layers

```typescript
const DYNAMIC_MUSIC_SYSTEM = {
  baseTrack: 'M-03_Shores_of_Fortune',

  layers: [
    {
      name: 'base',
      stems: ['pad', 'subtle_rhythm'],
      always_playing: true,
      volume: 0.6
    },
    {
      name: 'exploration',
      stems: ['melody_fragments', 'light_percussion'],
      trigger: 'player_moving',
      fadeIn: 2.0,
      volume: 0.8
    },
    {
      name: 'tension',
      stems: ['dark_harmony', 'pulse'],
      trigger: 'enemy_within_30m OR storm_close',
      fadeIn: 1.5,
      volume: 0.9
    },
    {
      name: 'combat',
      stems: ['full_combat_arrangement'],
      trigger: 'in_combat',
      fadeIn: 0.5,
      volume: 1.0,
      exclusive: true // Overrides other layers
    }
  ],

  finalCircleSwitch: {
    trigger: 'final_circle_active',
    switchTo: 'M-04_Storms_Fury',
    transition: 'stinger_transition',
    stinger: 'storm_warning_sting'
  }
};
```

### 6.2 State Machine

```typescript
const MUSIC_STATE_MACHINE = {
  states: {
    silent: { volume: 0 },
    menu: { track: 'M-01', volume: 1.0 },
    lobby: { track: 'M-02', volume: 1.0, dynamicLayers: true },
    match_calm: { track: 'M-03', layers: ['base', 'exploration'] },
    match_tense: { track: 'M-03', layers: ['base', 'tension'] },
    match_combat: { track: 'M-03', layers: ['combat'] },
    final_circle: { track: 'M-04', volume: 1.0 },
    victory: { track: 'M-05', oneShot: true },
    defeat: { track: 'M-06', oneShot: true },
    shop: { track: 'M-07', volume: 0.9 },
    battle_pass: { track: 'M-08', volume: 0.9 }
  },

  transitions: {
    menu_to_lobby: { type: 'crossfade', time: 2.0 },
    lobby_to_match: { type: 'fade_out_sting_fade_in', time: 3.0 },
    match_to_final: { type: 'stinger_switch', time: 1.0 }
  }
};
```

---

## 7. Implementation Checklist

### 7.1 Composition/Licensing

| Track | Status | Priority | Notes |
|-------|--------|----------|-------|
| M-01 Main Theme | Pending | Critical | Defines audio identity |
| M-02 Lobby | Pending | Critical | Dynamic layers needed |
| M-03 Match Ambient | Pending | Critical | 8 stems required |
| M-04 Final Circle | Pending | Critical | High intensity |
| M-05 Victory | Pending | Critical | Triumphant fanfare |
| M-06 Defeat | Pending | Critical | Brief, encouraging |
| M-07 Shop | Pending | High | Exotic feel |
| M-08 Battle Pass | Pending | High | Journey theme |

### 7.2 Implementation

- [ ] Music system integrated in engine
- [ ] Streaming playback working
- [ ] Dynamic layer system functional
- [ ] Transitions smooth
- [ ] Volume balancing complete
- [ ] Mobile performance verified

### 7.3 Quality Assurance

- [ ] All tracks loop seamlessly
- [ ] No audio pops or glitches
- [ ] Dynamic transitions feel natural
- [ ] Volume levels consistent
- [ ] Memory budget respected
- [ ] Battery impact acceptable

---

## Appendix A: Stinger List

| Stinger | Duration | Context |
|---------|----------|---------|
| Match Start | 2.5s | Beginning of match |
| Victory Short | 4s | #1 placement |
| Tier Unlock | 2s | Battle Pass tier |
| Chest Open | 1.5s | Loot chest |
| Level Up | 2s | Player level up |
| Storm Warning | 3s | Storm approaching |
| Elimination | 0.5s | Enemy eliminated |

---

## Appendix B: Reference Tempo Chart

| Track | BPM | Feel |
|-------|-----|------|
| M-01 Main Theme | 110 | Adventurous |
| M-02 Lobby | 100 | Relaxed |
| M-03 Match | 90 | Adaptable |
| M-04 Final Circle | 150 | Intense |
| M-05 Victory | 130 | Triumphant |
| M-06 Defeat | 75 | Reflective |
| M-07 Shop | 85 | Browsing |
| M-08 Battle Pass | 105 | Journey |
