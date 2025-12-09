# Mastery System Specification

## Document Information
- **Task ID:** LIVE-016
- **Priority:** P3
- **Complexity:** Medium
- **Status:** Complete
- **Target Release:** Post-Launch
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Mastery System for Plunderstorm Mobile, providing long-term progression goals through ability-specific mastery tracks that reward dedicated players with exclusive cosmetics and visible recognition for their expertise.

---

## Mastery Philosophy

### Design Goals

```typescript
const MASTERY_GOALS = {
  depth: {
    description: 'Add meaningful long-term progression',
    principle: 'Players always have something to work toward',
    timeline: 'Months of content per ability'
  },

  expertise: {
    description: 'Recognize ability expertise',
    principle: 'Show dedication to specific playstyles',
    visibility: 'Other players can see your mastery'
  },

  diversity: {
    description: 'Encourage trying different abilities',
    principle: 'Rewards for breadth as well as depth',
    balance: 'Don\'t force specific playstyles'
  },

  rewards: {
    description: 'Meaningful exclusive cosmetics',
    principle: 'Mastery rewards feel earned',
    exclusivity: 'Cannot be purchased with currency'
  }
};
```

---

## Mastery Structure

### Per-Ability Mastery

```typescript
const ABILITY_MASTERY = {
  scope: 'Each ability has independent mastery track',
  totalAbilities: 'All abilities in game (20+ at launch)',

  levels: {
    count: 10,
    progression: 'Exponential XP curve',
    xpRequired: [
      { level: 1, xp: 0, cumulative: 0 },
      { level: 2, xp: 1000, cumulative: 1000 },
      { level: 3, xp: 2000, cumulative: 3000 },
      { level: 4, xp: 4000, cumulative: 7000 },
      { level: 5, xp: 6000, cumulative: 13000 },
      { level: 6, xp: 10000, cumulative: 23000 },
      { level: 7, xp: 15000, cumulative: 38000 },
      { level: 8, xp: 25000, cumulative: 63000 },
      { level: 9, xp: 40000, cumulative: 103000 },
      { level: 10, xp: 60000, cumulative: 163000 }
    ]
  },

  timeEstimate: {
    casualPlayer: '~6-8 months to max single ability',
    dedicatedPlayer: '~2-3 months to max single ability',
    allAbilities: 'Years of content'
  }
};
```

### Mastery XP Earning

```typescript
const MASTERY_XP_EARNING = {
  sources: {
    abilityUsage: {
      description: 'XP for using ability in match',
      xpPerUse: 10,
      cap: '50 uses per match (500 XP max)'
    },
    abilityDamage: {
      description: 'XP for damage dealt with ability',
      xpPer100Damage: 5,
      cap: '2000 damage per match (100 XP max)'
    },
    abilityEliminations: {
      description: 'XP for kills with ability',
      xpPerElim: 50,
      cap: 'No cap'
    },
    matchCompletion: {
      description: 'Bonus XP if ability in loadout',
      xpPerMatch: 25,
      requirement: 'Must use ability at least once'
    },
    wins: {
      description: 'Bonus XP on victory',
      xpPerWin: 100,
      requirement: 'Ability in loadout and used'
    }
  },

  bonuses: {
    firstWinOfDay: {
      multiplier: 2.0,
      scope: 'First win with each ability'
    },
    weekendBonus: {
      multiplier: 1.5,
      timing: 'Saturday-Sunday UTC'
    },
    eventBonus: {
      multiplier: 2.0,
      timing: 'During mastery events'
    }
  },

  calculation: {
    perMatch: 'Sum of all XP sources',
    caps: 'Individual caps prevent farming',
    average: '~200-400 Mastery XP per match per ability'
  }
};
```

---

## Mastery Levels

### Level Titles

```typescript
const MASTERY_TITLES = {
  levels: {
    1: { title: 'Novice', color: '#FFFFFF' },
    2: { title: 'Apprentice', color: '#90EE90' },
    3: { title: 'Journeyman', color: '#32CD32' },
    4: { title: 'Adept', color: '#0099FF' },
    5: { title: 'Expert', color: '#0066CC' },
    6: { title: 'Veteran', color: '#9933FF' },
    7: { title: 'Elite', color: '#7700CC' },
    8: { title: 'Master', color: '#FFD700' },
    9: { title: 'Grandmaster', color: '#FFA500' },
    10: { title: 'Legend', color: '#FF4500', animated: true }
  },

  display: {
    format: '{Title} {AbilityName}',
    example: 'Master Fireball User',
    location: 'Profile, loadout, kill feed'
  }
};
```

### Level Rewards

```typescript
const MASTERY_REWARDS = {
  perLevel: {
    level1: {
      reward: 'Mastery Badge Unlocked',
      type: 'badge',
      description: 'Shows mastery initiated'
    },
    level2: {
      reward: 'Ability Spray',
      type: 'spray',
      description: 'Themed spray for ability'
    },
    level3: {
      reward: 'Mastery Emote',
      type: 'emote',
      description: 'Ability-themed emote'
    },
    level4: {
      reward: 'Ability Trail (Common)',
      type: 'trail',
      description: 'Subtle ability-themed trail'
    },
    level5: {
      reward: 'Ability Skin (Rare)',
      type: 'ability_skin',
      description: 'Alternate visual for ability'
    },
    level6: {
      reward: 'Ability Trail (Rare)',
      type: 'trail',
      description: 'Enhanced ability-themed trail'
    },
    level7: {
      reward: 'Mastery Banner',
      type: 'banner',
      description: 'Profile banner featuring ability'
    },
    level8: {
      reward: 'Ability Skin (Epic)',
      type: 'ability_skin',
      description: 'Premium alternate visual'
    },
    level9: {
      reward: 'Mastery Title',
      type: 'title',
      description: '"Grandmaster of {Ability}"'
    },
    level10: {
      reward: 'Legendary Ability Skin + Animated Badge',
      type: 'ability_skin + badge',
      description: 'Ultimate mastery recognition'
    }
  },

  exclusivity: {
    purchasable: false,
    tradeable: false,
    description: 'Must be earned through mastery'
  }
};
```

---

## Ability Skins

### Skin Tiers

```typescript
const ABILITY_SKINS = {
  rare: {
    unlockLevel: 5,
    changes: [
      'Recolored ability effects',
      'Subtle particle changes',
      'Modified trail colors'
    ],
    example: {
      ability: 'Fireball',
      skin: 'Azure Fireball',
      description: 'Blue flames instead of orange'
    }
  },

  epic: {
    unlockLevel: 8,
    changes: [
      'Significant visual overhaul',
      'New particle effects',
      'Unique sound modifications',
      'Enhanced impact effects'
    ],
    example: {
      ability: 'Fireball',
      skin: 'Phoenix Fireball',
      description: 'Golden phoenix-shaped flames with feather particles'
    }
  },

  legendary: {
    unlockLevel: 10,
    changes: [
      'Complete visual redesign',
      'Unique model/shape',
      'Custom sound effects',
      'Special impact animation',
      'Ambient particles'
    ],
    example: {
      ability: 'Fireball',
      skin: 'Solar Inferno',
      description: 'Mini sun with solar flares and cosmic effects'
    }
  }
};
```

### Skin Showcase

```typescript
const SKIN_SHOWCASE = {
  preview: {
    location: 'Mastery screen',
    content: 'Video preview of each skin tier',
    comparison: 'Side-by-side with base ability'
  },

  selection: {
    location: 'Loadout > Ability Customization',
    default: 'Base ability skin',
    change: 'Select any unlocked skin'
  },

  visibility: {
    self: 'Always see own skin',
    others: 'Other players see your skin',
    kill: 'Kill feed shows skin used'
  }
};
```

---

## Mastery Display

### Profile Display

```typescript
const PROFILE_DISPLAY = {
  masterySection: {
    location: 'Profile > Mastery',
    display: 'Grid of all abilities with levels',
    sorting: 'By level (highest first) or alphabetical'
  },

  perAbility: {
    icon: 'Ability icon with mastery border',
    level: 'Current mastery level',
    progress: 'XP progress bar to next level',
    title: 'Current mastery title'
  },

  summary: {
    totalMastery: 'Sum of all ability levels',
    highestLevel: 'Highest single ability mastery',
    masteredCount: 'Abilities at level 10'
  },

  showcase: {
    featured: 'Player can feature 1-3 abilities',
    display: 'Shown prominently on profile card'
  }
};
```

### In-Match Display

```typescript
const IN_MATCH_DISPLAY = {
  loadout: {
    abilityBorder: 'Mastery-tier colored border',
    levelIndicator: 'Small level number'
  },

  killFeed: {
    format: '[PlayerName] [MasteryBadge] eliminated [Target]',
    badge: 'Small mastery icon if level 5+'
  },

  spectator: {
    display: 'Show mastery level when spectating',
    tooltip: 'Hover for full mastery info'
  },

  postMatch: {
    xpGained: 'Mastery XP earned per ability',
    levelUp: 'Celebration if mastery level gained'
  }
};
```

### Mastery Border

```typescript
const MASTERY_BORDER = {
  description: 'Visual border around ability icon based on mastery',

  tiers: {
    none: { levels: [0], border: 'Default gray' },
    bronze: { levels: [1, 2], border: 'Bronze metallic' },
    silver: { levels: [3, 4], border: 'Silver metallic' },
    gold: { levels: [5, 6], border: 'Gold metallic' },
    platinum: { levels: [7, 8], border: 'Platinum with glow' },
    diamond: { levels: [9, 10], border: 'Diamond animated with sparkle' }
  },

  animation: {
    platinum: 'Subtle pulse',
    diamond: 'Rotating sparkle effect'
  }
};
```

---

## Global Mastery Score

### Total Mastery

```typescript
const GLOBAL_MASTERY = {
  score: {
    calculation: 'Sum of all ability mastery levels',
    maximum: 'Total abilities × 10',
    display: 'Profile and leaderboard'
  },

  milestones: {
    10: { reward: 'Mastery Initiate Badge', title: 'Mastery Initiate' },
    25: { reward: 'Mastery Adept Badge', title: 'Mastery Adept' },
    50: { reward: 'Mastery Expert Badge', title: 'Mastery Expert' },
    75: { reward: 'Mastery Elite Badge', title: 'Mastery Elite' },
    100: { reward: 'Mastery Master Badge', title: 'Mastery Master' },
    150: { reward: 'Mastery Legend Badge', title: 'Mastery Legend' },
    max: { reward: 'Ultimate Mastery Badge', title: 'True Master' }
  },

  leaderboard: {
    type: 'Global mastery score leaderboard',
    display: 'Top 1000 globally',
    rewards: 'Seasonal recognition for top performers'
  }
};
```

### Mastery Diversity Bonus

```typescript
const DIVERSITY_BONUS = {
  description: 'Bonus rewards for mastering multiple abilities',

  breadthMilestones: {
    fiveAbilitiesLevel5: {
      reward: 'Versatile Fighter Badge',
      description: 'Master of many trades'
    },
    tenAbilitiesLevel5: {
      reward: 'Jack of All Trades Title',
      description: 'Competent with everything'
    },
    allAbilitiesLevel5: {
      reward: 'Complete Arsenal Badge',
      description: 'Expert with every ability'
    },
    allAbilitiesLevel10: {
      reward: 'Ultimate Master Title + Exclusive Outfit',
      description: 'Maximum dedication achieved'
    }
  }
};
```

---

## Mastery Events

### Double Mastery XP

```typescript
const MASTERY_EVENTS = {
  doubleMasteryWeekend: {
    frequency: 'Monthly',
    duration: 'Friday-Sunday',
    bonus: '2x Mastery XP all abilities',
    communication: 'Announced 1 week ahead'
  },

  abilitySpotlight: {
    frequency: 'Bi-weekly',
    duration: '1 week',
    bonus: '3x Mastery XP for featured ability',
    selection: 'Rotate through all abilities',
    communication: 'In-game banner and notification'
  },

  masteryRush: {
    frequency: 'Special events',
    duration: 'Limited time',
    bonus: '2x XP + bonus rewards',
    challenges: 'Special mastery challenges'
  }
};
```

### Mastery Challenges

```typescript
const MASTERY_CHALLENGES = {
  daily: {
    example: 'Earn 500 Mastery XP with any ability',
    reward: '100 bonus Mastery XP'
  },

  weekly: {
    example: 'Level up any ability mastery',
    reward: '500 bonus Mastery XP + Gold'
  },

  special: {
    example: 'Reach Mastery Level 5 on a new ability',
    reward: 'Exclusive mastery chest'
  }
};
```

---

## Technical Implementation

### Data Model

```typescript
const DATA_MODEL = {
  playerAbilityMastery: {
    playerId: 'string',
    abilityId: 'string',
    currentLevel: 'number (1-10)',
    currentXp: 'number',
    totalXpEarned: 'number',
    usageStats: {
      totalUses: 'number',
      totalDamage: 'number',
      totalEliminations: 'number'
    },
    unlockedRewards: 'array<rewardId>',
    equippedSkin: 'string (skinId, nullable)'
  },

  playerMasterySummary: {
    playerId: 'string',
    totalMasteryScore: 'number',
    highestMasteryAbility: 'string (abilityId)',
    highestMasteryLevel: 'number',
    masteredAbilitiesCount: 'number',
    featuredAbilities: 'array<abilityId> (max 3)'
  },

  masteryXpLog: {
    playerId: 'string',
    abilityId: 'string',
    matchId: 'string',
    xpEarned: 'number',
    breakdown: 'object<source, amount>',
    timestamp: 'timestamp'
  }
};
```

### API Endpoints

```typescript
const API_ENDPOINTS = {
  mastery: {
    getAll: 'GET /api/v1/players/{id}/mastery',
    getAbility: 'GET /api/v1/players/{id}/mastery/{abilityId}',
    setSkin: 'PUT /api/v1/players/{id}/mastery/{abilityId}/skin',
    setFeatured: 'PUT /api/v1/players/{id}/mastery/featured',
    getLeaderboard: 'GET /api/v1/mastery/leaderboard'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Mastery XP earned correctly per source',
    'Level up triggers at correct thresholds',
    'Rewards granted on level up',
    'Ability skins apply correctly',
    'Profile displays mastery accurately'
  ],

  balance: [
    'XP rates feel rewarding but not too fast',
    'Caps prevent unhealthy grinding',
    'Time to max is appropriate'
  ],

  visual: [
    'Mastery borders display correctly',
    'Ability skins render properly',
    'Level up celebration displays'
  ],

  edge: [
    'Ability removed from game (preserve mastery)',
    'New ability added (start at level 1)',
    'Max level reached (no further XP needed)',
    'Multiple abilities level up same match'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial mastery system specification |
