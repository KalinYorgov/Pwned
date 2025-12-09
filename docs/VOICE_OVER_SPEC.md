# Voice Over Specification

## Document Information
- **Task ID:** ART-028
- **Priority:** P3
- **Complexity:** Large
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Voice Over (VO) system for Plunderstorm Mobile, including tutorial narration, match announcements, character barks, and environmental voice cues that enhance immersion with authentic pirate-themed voice acting.

---

## Voice Over Philosophy

### Design Goals

```typescript
const VO_GOALS = {
  immersion: {
    description: 'Enhance pirate world believability',
    style: 'Authentic but accessible pirate speech',
    tone: 'Adventurous, slightly comedic, never too serious'
  },

  clarity: {
    description: 'Information always understandable',
    priority: 'Gameplay info > flavor',
    articulation: 'Clear even with accent'
  },

  accessibility: {
    description: 'Never rely solely on VO',
    subtitles: 'All VO subtitled',
    visualCues: 'Complement with visual indicators'
  },

  restraint: {
    description: 'Quality over quantity',
    principle: 'Meaningful lines, not constant chatter',
    repetition: 'Avoid annoying repeated lines'
  }
};
```

---

## Voice Cast

### Character Voices

```typescript
const VOICE_CAST = {
  announcer: {
    name: 'The Quartermaster',
    role: 'Primary announcer voice',
    description: 'Gruff but friendly veteran pirate',
    accent: 'Classic pirate (West Country English)',
    age: '50-60s',
    personality: 'Authoritative, encouraging, weathered'
  },

  tutorialNarrator: {
    name: 'Captain Whiskers',
    role: 'Tutorial and guide voice',
    description: 'Experienced captain teaching new pirates',
    accent: 'Slightly softer pirate accent',
    age: '40-50s',
    personality: 'Patient, knowledgeable, mentor-like'
  },

  playerCharacter: {
    male: {
      name: 'Young Buccaneer (Male)',
      description: 'Eager young male pirate',
      accent: 'Light pirate accent',
      age: '20-30s',
      barks: 'Combat, ability, reaction sounds'
    },
    female: {
      name: 'Young Buccaneer (Female)',
      description: 'Eager young female pirate',
      accent: 'Light pirate accent',
      age: '20-30s',
      barks: 'Combat, ability, reaction sounds'
    }
  },

  additionalVoices: {
    merchant: {
      name: 'Shop Keeper',
      description: 'Enthusiastic shop owner',
      usage: 'Shop interactions'
    },
    crewMembers: {
      description: 'Background crew voices',
      usage: 'Ambient dialogue, reactions'
    }
  }
};
```

---

## Tutorial Voice Over

### Tutorial Narration

```typescript
const TUTORIAL_VO = {
  narrator: 'Captain Whiskers',
  style: 'Conversational, guiding',

  segments: {
    introduction: {
      lines: [
        {
          id: 'TUT_INTRO_01',
          text: 'Ahoy there, fresh face! Welcome to the finest battle on the Seven Seas!',
          timing: 'Game start'
        },
        {
          id: 'TUT_INTRO_02',
          text: 'I\'m Captain Whiskers, and I\'ll be teachin\' ye the ways of Plunderstorm.',
          timing: 'After intro'
        }
      ]
    },

    movement: {
      lines: [
        {
          id: 'TUT_MOVE_01',
          text: 'First things first - ye need to learn to move! Use the left side of yer screen to walk about.',
          timing: 'Movement tutorial start'
        },
        {
          id: 'TUT_MOVE_02',
          text: 'That\'s it! Steady as she goes!',
          timing: 'Movement success'
        },
        {
          id: 'TUT_MOVE_03',
          text: 'Want to run faster? Double-tap and hold to sprint!',
          timing: 'Sprint tutorial'
        }
      ]
    },

    camera: {
      lines: [
        {
          id: 'TUT_CAM_01',
          text: 'Now, ye need eyes in the back of yer head out there. Drag on the right side to look around.',
          timing: 'Camera tutorial start'
        },
        {
          id: 'TUT_CAM_02',
          text: 'Sharp eyes ye\'ve got! Always be watchin\' for enemies.',
          timing: 'Camera success'
        }
      ]
    },

    looting: {
      lines: [
        {
          id: 'TUT_LOOT_01',
          text: 'See that chest there? That\'s where ye\'ll find yer powers! Get close and tap to open it.',
          timing: 'Chest tutorial'
        },
        {
          id: 'TUT_LOOT_02',
          text: 'A fine find! That ability will serve ye well in battle.',
          timing: 'Ability pickup'
        },
        {
          id: 'TUT_LOOT_03',
          text: 'Ye can carry up to four abilities. Choose wisely!',
          timing: 'Ability slot explanation'
        }
      ]
    },

    abilities: {
      lines: [
        {
          id: 'TUT_ABIL_01',
          text: 'Time to use what ye\'ve found! Tap an ability button to unleash it!',
          timing: 'Ability tutorial start'
        },
        {
          id: 'TUT_ABIL_02',
          text: 'Some abilities need aimin\'. Hold and drag to set yer target!',
          timing: 'Skillshot tutorial'
        },
        {
          id: 'TUT_ABIL_03',
          text: 'Brilliant shot! Ye\'re a natural, ye are!',
          timing: 'Ability hit success'
        }
      ]
    },

    combat: {
      lines: [
        {
          id: 'TUT_COMB_01',
          text: 'Enemy off the port bow! Time to show \'em what ye\'re made of!',
          timing: 'First enemy encounter'
        },
        {
          id: 'TUT_COMB_02',
          text: 'Ye got \'em! One less scallywag to worry about!',
          timing: 'First elimination'
        }
      ]
    },

    storm: {
      lines: [
        {
          id: 'TUT_STORM_01',
          text: 'Hear that thunder? The storm\'s comin\' in! Stay inside the safe zone or ye\'ll take damage.',
          timing: 'Storm warning'
        },
        {
          id: 'TUT_STORM_02',
          text: 'The eye of the storm - that\'s where ye want to be!',
          timing: 'In safe zone'
        }
      ]
    },

    conclusion: {
      lines: [
        {
          id: 'TUT_END_01',
          text: 'Ye\'ve learned the basics! Now get out there and show \'em who\'s the greatest pirate on the seas!',
          timing: 'Tutorial complete'
        },
        {
          id: 'TUT_END_02',
          text: 'May the winds be at yer back, matey!',
          timing: 'Tutorial exit'
        }
      ]
    }
  },

  totalLines: 18,
  estimatedDuration: '5-7 minutes'
};
```

---

## Match Announcements

### Pre-Match Announcements

```typescript
const PREMATCH_VO = {
  narrator: 'The Quartermaster',

  dropship: {
    lines: [
      {
        id: 'PRE_DROP_01',
        text: 'Alright ye sea dogs, prepare to drop!',
        timing: 'Match start, in dropship'
      },
      {
        id: 'PRE_DROP_02',
        text: 'Pick yer landing spot and fight for glory!',
        timing: '5 seconds before drop enabled'
      },
      {
        id: 'PRE_DROP_03',
        text: 'Drop zone is open! Go, go, go!',
        timing: 'Drop enabled'
      }
    ]
  },

  countdown: {
    lines: [
      {
        id: 'PRE_COUNT_01',
        text: 'The battle begins!',
        timing: 'Match officially starts'
      }
    ]
  }
};
```

### Player Count Announcements

```typescript
const PLAYER_COUNT_VO = {
  narrator: 'The Quartermaster',

  milestones: [
    {
      id: 'COUNT_50',
      remaining: 50,
      text: 'Fifty pirates still standin\'!'
    },
    {
      id: 'COUNT_30',
      remaining: 30,
      text: 'Thirty left! Keep yer guard up!'
    },
    {
      id: 'COUNT_20',
      remaining: 20,
      text: 'Twenty remain! The waters run red!'
    },
    {
      id: 'COUNT_10',
      remaining: 10,
      text: 'Only ten left! Victory is within reach!'
    },
    {
      id: 'COUNT_5',
      remaining: 5,
      text: 'Five pirates! It\'s getting tense out there!'
    },
    {
      id: 'COUNT_3',
      remaining: 3,
      text: 'Three remain! Who will claim the treasure?'
    },
    {
      id: 'COUNT_2',
      remaining: 2,
      text: 'Two pirates! It comes down to this!'
    }
  ],

  variants: {
    enabled: true,
    description: 'Multiple variants per milestone to reduce repetition',
    variantsPerMilestone: 3
  }
};
```

### Storm Announcements

```typescript
const STORM_VO = {
  narrator: 'The Quartermaster',

  phases: [
    {
      id: 'STORM_WARN_01',
      event: 'Storm warning',
      text: 'Storm\'s brewin\'! Get to the safe zone!'
    },
    {
      id: 'STORM_CLOSE_01',
      event: 'Storm closing',
      text: 'The storm\'s closin\' in! Move it!'
    },
    {
      id: 'STORM_FINAL_01',
      event: 'Final circle',
      text: 'This is it! The final battle!'
    }
  ],

  damage: {
    id: 'STORM_DMG_01',
    event: 'Player in storm',
    text: 'Get out of the storm! Ye\'re takin\' damage!',
    cooldown: 'Play once per storm phase'
  }
};
```

### Victory and Defeat

```typescript
const ENDGAME_VO = {
  narrator: 'The Quartermaster',

  victory: {
    lines: [
      {
        id: 'WIN_01',
        text: 'Victory! Ye\'re the greatest pirate on the seas!',
        variant: 1
      },
      {
        id: 'WIN_02',
        text: 'Champion! The treasure is yours!',
        variant: 2
      },
      {
        id: 'WIN_03',
        text: 'Legendary! They\'ll sing shanties about ye!',
        variant: 3
      }
    ]
  },

  topPlacements: {
    second: {
      id: 'PLACE_2',
      text: 'Second place! So close to glory!'
    },
    topThree: {
      id: 'PLACE_TOP3',
      text: 'Top three! A worthy showing!'
    },
    topTen: {
      id: 'PLACE_TOP10',
      text: 'Top ten! Well fought, matey!'
    }
  },

  defeat: {
    lines: [
      {
        id: 'LOSE_01',
        text: 'Ye fought well! Return to reclaim yer honor!',
        variant: 1
      },
      {
        id: 'LOSE_02',
        text: 'The seas are unforgiving, but so are ye! Try again!',
        variant: 2
      }
    ]
  }
};
```

---

## Character Barks

### Combat Barks

```typescript
const COMBAT_BARKS = {
  voiceActors: ['Male Buccaneer', 'Female Buccaneer'],

  categories: {
    abilityUse: {
      description: 'Short exclamations when using abilities',
      examples: [
        { id: 'BARK_FIRE_01', text: 'Fire!' },
        { id: 'BARK_TAKE_01', text: 'Take that!' },
        { id: 'BARK_EAT_01', text: 'Eat this!' },
        { id: 'BARK_HA_01', text: 'Ha!' },
        { id: 'BARK_GOTCHA_01', text: 'Gotcha!' }
      ],
      playRate: '30% chance on ability use'
    },

    elimination: {
      description: 'After eliminating enemy',
      examples: [
        { id: 'BARK_ELIM_01', text: 'One down!' },
        { id: 'BARK_ELIM_02', text: 'Back to Davy Jones!' },
        { id: 'BARK_ELIM_03', text: 'That\'ll teach ye!' },
        { id: 'BARK_ELIM_04', text: 'Too easy!' }
      ],
      playRate: '50% chance on elimination'
    },

    takingDamage: {
      description: 'When hit by enemy',
      examples: [
        { id: 'BARK_HIT_01', text: 'Argh!' },
        { id: 'BARK_HIT_02', text: 'Oof!' },
        { id: 'BARK_HIT_03', text: 'That stings!' }
      ],
      playRate: '20% chance on hit',
      cooldown: '3 seconds'
    },

    lowHealth: {
      description: 'When health drops below 30%',
      examples: [
        { id: 'BARK_LOW_01', text: 'I need to heal!' },
        { id: 'BARK_LOW_02', text: 'That\'s a scratch...' },
        { id: 'BARK_LOW_03', text: 'Running low here!' }
      ],
      playRate: 'Once per low health event'
    },

    downed: {
      description: 'When knocked down in squad mode',
      examples: [
        { id: 'BARK_DOWN_01', text: 'I\'m down! Help!' },
        { id: 'BARK_DOWN_02', text: 'Need a hand here!' },
        { id: 'BARK_DOWN_03', text: 'They got me!' }
      ],
      playRate: '100% on down'
    },

    revived: {
      description: 'When revived by teammate',
      examples: [
        { id: 'BARK_REV_01', text: 'Much obliged!' },
        { id: 'BARK_REV_02', text: 'Back in the fight!' },
        { id: 'BARK_REV_03', text: 'I owe ye one!' }
      ],
      playRate: '100% on revive'
    }
  }
};
```

### Ability-Specific Callouts

```typescript
const ABILITY_CALLOUTS = {
  description: 'Unique lines for specific abilities',

  fireball: {
    cast: [
      { id: 'CALL_FIRE_01', text: 'Burn!' },
      { id: 'CALL_FIRE_02', text: 'Feel the heat!' }
    ]
  },

  teleport: {
    cast: [
      { id: 'CALL_TELE_01', text: 'Now ye see me...' },
      { id: 'CALL_TELE_02', text: 'Vanishing!' }
    ]
  },

  heal: {
    cast: [
      { id: 'CALL_HEAL_01', text: 'Patching up!' },
      { id: 'CALL_HEAL_02', text: 'That\'s better!' }
    ]
  },

  ultimate: {
    cast: [
      { id: 'CALL_ULT_01', text: 'Witness my power!' },
      { id: 'CALL_ULT_02', text: 'Unleashing fury!' }
    ]
  },

  playRate: '25% chance per ability use'
};
```

---

## Environmental Voice

### Ambient Dialogue

```typescript
const AMBIENT_VO = {
  location: 'Background NPC chatter',

  tavern: {
    lines: [
      'Did ye hear about the treasure fleet?',
      'Another round of grog!',
      'I\'ve sailed all seven seas, I have!'
    ],
    volume: 'Background level',
    frequency: 'Occasional'
  },

  port: {
    lines: [
      'Fresh fish! Get yer fresh fish!',
      'Mind the cargo!',
      'Ships comin\' in!'
    ]
  },

  usage: 'Main menu background ambiance'
};
```

### Shop Interactions

```typescript
const SHOP_VO = {
  merchant: 'Shop Keeper',

  greetings: [
    { id: 'SHOP_GREET_01', text: 'Welcome to me humble shop!' },
    { id: 'SHOP_GREET_02', text: 'Got the finest wares on the seas!' },
    { id: 'SHOP_GREET_03', text: 'Ah, a customer! What\'ll it be?' }
  ],

  purchase: [
    { id: 'SHOP_BUY_01', text: 'Excellent choice!' },
    { id: 'SHOP_BUY_02', text: 'Ye\'ve got fine taste!' },
    { id: 'SHOP_BUY_03', text: 'Pleasure doin\' business!' }
  ],

  browse: [
    { id: 'SHOP_BROWSE_01', text: 'Take yer time, look around!' },
    { id: 'SHOP_BROWSE_02', text: 'See anything ye fancy?' }
  ],

  exit: [
    { id: 'SHOP_EXIT_01', text: 'Come back soon!' },
    { id: 'SHOP_EXIT_02', text: 'Fair winds to ye!' }
  ]
};
```

---

## Localization

### Supported Languages

```typescript
const LOCALIZATION = {
  launchLanguages: [
    { code: 'en-US', name: 'English (US)', priority: 'Primary' },
    { code: 'en-GB', name: 'English (UK)', priority: 'Primary' },
    { code: 'es-ES', name: 'Spanish', priority: 'High' },
    { code: 'pt-BR', name: 'Portuguese (Brazil)', priority: 'High' },
    { code: 'fr-FR', name: 'French', priority: 'High' },
    { code: 'de-DE', name: 'German', priority: 'High' }
  ],

  postLaunchLanguages: [
    { code: 'ja-JP', name: 'Japanese', priority: 'Medium' },
    { code: 'ko-KR', name: 'Korean', priority: 'Medium' },
    { code: 'zh-CN', name: 'Chinese (Simplified)', priority: 'Medium' },
    { code: 'zh-TW', name: 'Chinese (Traditional)', priority: 'Medium' },
    { code: 'it-IT', name: 'Italian', priority: 'Low' },
    { code: 'ru-RU', name: 'Russian', priority: 'Low' }
  ],

  localizationApproach: {
    method: 'Full voice recording per language',
    fallback: 'English with subtitles if VO unavailable',
    characterization: 'Maintain pirate personality in each language'
  }
};
```

### Subtitle System

```typescript
const SUBTITLES = {
  enabled: true,
  default: 'On for accessibility',

  settings: {
    toggle: 'Enable/disable subtitles',
    size: ['Small', 'Medium', 'Large'],
    background: ['None', 'Semi-transparent', 'Solid'],
    speaker: 'Show speaker name',
    color: 'Color-coded by speaker type'
  },

  display: {
    position: 'Bottom center of screen',
    maxLines: 2,
    duration: 'Matches audio length + 0.5s buffer',
    animation: 'Fade in/out'
  },

  formatting: {
    announcer: {
      color: '#FFD700',
      prefix: '[Quartermaster]'
    },
    tutorial: {
      color: '#87CEEB',
      prefix: '[Captain Whiskers]'
    },
    player: {
      color: '#FFFFFF',
      prefix: 'None'
    },
    ambient: {
      color: '#AAAAAA',
      prefix: 'None'
    }
  }
};
```

---

## Technical Specifications

### Audio Format

```typescript
const AUDIO_FORMAT = {
  recording: {
    format: 'WAV 24-bit',
    sampleRate: 48000,
    channels: 'Mono (dialogue)',
    quality: 'Studio quality'
  },

  delivery: {
    format: 'OGG Vorbis',
    quality: 6,
    sampleRate: 44100,
    compression: 'Balanced quality/size'
  },

  mobile: {
    format: 'AAC (iOS) / OGG (Android)',
    bitrate: '128 kbps',
    streaming: 'Preload common lines'
  }
};
```

### File Naming Convention

```typescript
const FILE_NAMING = {
  pattern: '{CHARACTER}_{CATEGORY}_{ID}_{VARIANT}.ogg',

  examples: [
    'quartermaster_count_50_v1.ogg',
    'captain_tutorial_move_01.ogg',
    'buccaneer_male_bark_fire_01.ogg',
    'buccaneer_female_elim_02.ogg',
    'merchant_greet_03.ogg'
  ],

  languages: {
    pattern: '{LANGUAGE}/{CHARACTER}_{CATEGORY}_{ID}_{VARIANT}.ogg',
    example: 'es-ES/quartermaster_count_50_v1.ogg'
  }
};
```

### VO System Implementation

```typescript
const VO_SYSTEM = {
  playback: {
    priority: {
      highest: 'Critical gameplay (storm warning)',
      high: 'Player actions (elimination)',
      medium: 'Announcements (player count)',
      low: 'Ambient (background chatter)'
    },
    interruption: 'Higher priority interrupts lower',
    queue: 'Same priority queued (max 2)'
  },

  cooldowns: {
    global: '2 seconds between any VO',
    perCategory: {
      barks: '3 seconds',
      announcements: '5 seconds',
      tutorial: 'None (scripted)'
    }
  },

  variation: {
    randomSelection: 'Random from available variants',
    history: 'Avoid repeating last 3 played variants',
    weightedRecency: 'Prefer less recently played'
  },

  volume: {
    master: 'Linked to VO volume setting',
    ducking: 'Music ducks 50% during VO',
    distance: 'Ambient VO spatial falloff'
  }
};
```

---

## Production Pipeline

### Recording Process

```typescript
const RECORDING_PROCESS = {
  preProduction: {
    scriptFinalization: 'Complete all lines before recording',
    characterGuides: 'Character voice direction documents',
    pronunciation: 'Guide for game-specific terms'
  },

  recording: {
    studio: 'Professional recording studio',
    equipment: 'High-quality condenser microphone',
    direction: 'Voice director present for sessions',
    takes: 'Minimum 3 takes per line'
  },

  postProduction: {
    editing: 'Clean up, remove breaths/clicks',
    normalization: 'Consistent levels across all lines',
    processing: 'Light compression for mobile playback',
    export: 'Batch export to delivery format'
  }
};
```

### Line Count Estimate

```typescript
const LINE_COUNT = {
  tutorial: {
    lines: 20,
    variants: 1,
    total: 20
  },

  announcements: {
    lines: 25,
    variants: 3,
    total: 75
  },

  characterBarks: {
    linesPerCategory: 5,
    categories: 6,
    characters: 2,
    total: 60
  },

  abilityCallouts: {
    abilities: 20,
    linesPerAbility: 2,
    characters: 2,
    total: 80
  },

  shopMerchant: {
    lines: 15,
    variants: 1,
    total: 15
  },

  ambient: {
    lines: 20,
    total: 20
  },

  grandTotal: {
    english: 270,
    perLanguage: 250,
    withLocalization: '~1500 total lines (6 languages)'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  quality: [
    'All lines audible and clear',
    'Consistent volume across all VO',
    'No clipping or distortion',
    'Accents understandable'
  ],

  integration: [
    'VO triggers at correct times',
    'Subtitles sync with audio',
    'Priority system works correctly',
    'Cooldowns prevent spam'
  ],

  localization: [
    'All languages complete',
    'Lip sync not required (no character faces)',
    'Subtitles match audio in all languages',
    'Cultural appropriateness verified'
  ],

  accessibility: [
    'Subtitles display correctly',
    'VO can be disabled without losing info',
    'Visual cues accompany all VO'
  ],

  performance: [
    'No audio lag or delay',
    'Memory footprint acceptable',
    'Streaming works on low-end devices',
    'No crashes from VO playback'
  ],

  userTesting: [
    'VO enhances experience (survey)',
    'Character voices liked',
    'Not annoying or repetitive',
    'Pirate theme appreciated'
  ]
};
```

---

## Data Model

### VO Data Structure

```typescript
const DATA_MODEL = {
  voLine: {
    id: 'string',
    text: 'string',
    character: 'string',
    category: 'string',
    subcategory: 'string (optional)',
    priority: 'number',
    cooldownGroup: 'string',
    variants: 'array<string> (file paths)',
    subtitleKey: 'string (localization key)'
  },

  voPlayback: {
    activeLine: 'VoLine (nullable)',
    queue: 'array<VoLine>',
    lastPlayedTimes: 'object<category, timestamp>',
    variantHistory: 'array<string>'
  },

  voSettings: {
    enabled: 'boolean',
    volume: 'number (0-1)',
    subtitlesEnabled: 'boolean',
    subtitleSize: 'small | medium | large',
    language: 'string'
  }
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial voice over specification |
