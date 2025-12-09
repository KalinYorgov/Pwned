# Player Titles and Badges Specification

## Document Information
- **Task ID:** LIVE-014
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the player titles and badges system for Plunderstorm Mobile, providing players with prestigious display elements that recognize accomplishments, create prestige, and allow for personal expression.

---

## System Philosophy

### Design Goals

```typescript
const DESIGN_GOALS = {
  prestige: {
    description: 'Titles and badges should feel meaningful',
    principle: 'Earned, not bought (mostly)',
    rarity: 'Some should be rare and highly desirable'
  },

  identity: {
    description: 'Allow players to express their playstyle',
    principle: 'Multiple options to showcase preferences',
    customization: 'Player chooses what to display'
  },

  progression: {
    description: 'Provide long-term goals',
    principle: 'Always something new to work toward',
    tiers: 'Progressive difficulty for completionists'
  },

  recognition: {
    description: 'Visible to other players',
    principle: 'Show off accomplishments in-game',
    social: 'Inspire others, create conversation'
  }
};
```

---

## Titles System

### Title Categories

```typescript
const TITLE_CATEGORIES = {
  combat: {
    name: 'Combat Titles',
    theme: 'Prowess in battle',
    examples: [
      { title: 'Bloodthirsty', requirement: '1,000 eliminations', rarity: 'common' },
      { title: 'Slayer', requirement: '5,000 eliminations', rarity: 'rare' },
      { title: 'Executioner', requirement: '25,000 eliminations', rarity: 'epic' },
      { title: 'Death Incarnate', requirement: '100,000 eliminations', rarity: 'legendary' }
    ]
  },

  victory: {
    name: 'Victory Titles',
    theme: 'Match wins',
    examples: [
      { title: 'Victor', requirement: '10 wins', rarity: 'common' },
      { title: 'Champion', requirement: '100 wins', rarity: 'rare' },
      { title: 'Conqueror', requirement: '500 wins', rarity: 'epic' },
      { title: 'Legendary Pirate', requirement: '1,000 wins', rarity: 'legendary' }
    ]
  },

  ranked: {
    name: 'Ranked Titles',
    theme: 'Competitive achievement',
    examples: [
      { title: 'Ranked Warrior', requirement: 'Reach Silver', rarity: 'common' },
      { title: 'Ranked Elite', requirement: 'Reach Diamond', rarity: 'rare' },
      { title: 'Pirate King', requirement: 'Reach Pirate King tier', rarity: 'legendary' },
      { title: 'Top 100', requirement: 'Reach Top 100 leaderboard', rarity: 'legendary' }
    ]
  },

  exploration: {
    name: 'Exploration Titles',
    theme: 'Map and content discovery',
    examples: [
      { title: 'Explorer', requirement: 'Visit all POIs in one match', rarity: 'common' },
      { title: 'Treasure Hunter', requirement: 'Open 500 chests', rarity: 'rare' },
      { title: 'Cartographer', requirement: 'Discover all secret locations', rarity: 'epic' }
    ]
  },

  boss: {
    name: 'Boss Titles',
    theme: 'PvE encounters',
    examples: [
      { title: 'Monster Hunter', requirement: 'Defeat 100 monsters', rarity: 'common' },
      { title: 'Kraken Slayer', requirement: 'Defeat Kraken 10 times', rarity: 'rare' },
      { title: 'Bane of Beasts', requirement: 'Defeat all boss types', rarity: 'epic' }
    ]
  },

  social: {
    name: 'Social Titles',
    theme: 'Playing with others',
    examples: [
      { title: 'Team Player', requirement: 'Win 50 party matches', rarity: 'common' },
      { title: 'Party Captain', requirement: 'Lead 100 party victories', rarity: 'rare' },
      { title: 'Legend Among Friends', requirement: 'Win with 50 unique friends', rarity: 'epic' }
    ]
  },

  seasonal: {
    name: 'Seasonal Titles',
    theme: 'Time-limited achievements',
    examples: [
      { title: 'Founding Pirate', requirement: 'Play during launch month', rarity: 'rare', exclusive: true },
      { title: 'Season 1 Veteran', requirement: 'Complete Season 1 Battle Pass', rarity: 'rare', exclusive: true },
      { title: 'Holiday Hero', requirement: 'Complete winter event challenges', rarity: 'rare', exclusive: true }
    ]
  },

  mastery: {
    name: 'Mastery Titles',
    theme: 'Ability expertise',
    examples: [
      { title: 'Sword Master', requirement: 'Win 100 games with melee abilities', rarity: 'rare' },
      { title: 'Spell Slinger', requirement: 'Cast 10,000 abilities', rarity: 'rare' },
      { title: 'Jack of All Trades', requirement: 'Win with every ability', rarity: 'epic' }
    ]
  },

  special: {
    name: 'Special Titles',
    theme: 'Unique accomplishments',
    examples: [
      { title: 'The Untouchable', requirement: 'Win without taking damage', rarity: 'legendary' },
      { title: 'Pacifist Victor', requirement: 'Win with 0 eliminations', rarity: 'legendary' },
      { title: 'Against All Odds', requirement: 'Win 1v4 in final circle', rarity: 'legendary' }
    ]
  }
};
```

### Title Display

```typescript
const TITLE_DISPLAY = {
  location: {
    underName: 'Display below player name',
    profileCard: 'Show on profile inspection',
    killFeed: 'Show in elimination notifications',
    leaderboard: 'Display on leaderboards'
  },

  formatting: {
    prefix: 'None (appears below name)',
    color: 'Based on rarity',
    animation: 'Legendary titles have subtle glow',
    length: 'Max 20 characters'
  },

  rarityColors: {
    common: '#FFFFFF',
    rare: '#0099FF',
    epic: '#9933FF',
    legendary: '#FFD700'
  }
};
```

---

## Badges System

### Badge Categories

```typescript
const BADGE_CATEGORIES = {
  rankedSeason: {
    name: 'Ranked Season Badges',
    description: 'Awarded based on peak rank each season',
    examples: [
      { badge: 'S1 Bronze', requirement: 'Peak Bronze in Season 1' },
      { badge: 'S1 Silver', requirement: 'Peak Silver in Season 1' },
      { badge: 'S1 Gold', requirement: 'Peak Gold in Season 1' },
      { badge: 'S1 Diamond', requirement: 'Peak Diamond in Season 1' },
      { badge: 'S1 Master', requirement: 'Peak Master in Season 1' },
      { badge: 'S1 Pirate King', requirement: 'Reach Pirate King in Season 1' }
    ],
    exclusivity: 'Never returns, permanent record of rank'
  },

  battlePass: {
    name: 'Battle Pass Badges',
    description: 'Awarded for Battle Pass completion',
    examples: [
      { badge: 'S1 Pass Complete', requirement: 'Complete Season 1 Battle Pass' },
      { badge: 'S1 Premium Pirate', requirement: 'Complete Premium Battle Pass' }
    ],
    exclusivity: 'Never returns'
  },

  event: {
    name: 'Event Badges',
    description: 'Awarded for event participation',
    examples: [
      { badge: 'Cursed Seas Survivor', requirement: 'Complete Halloween event' },
      { badge: 'Frostbitten Champion', requirement: 'Win during Winter event' },
      { badge: 'Anniversary Celebrant', requirement: 'Play during anniversary' }
    ],
    exclusivity: 'May return annually'
  },

  achievement: {
    name: 'Achievement Badges',
    description: 'Tied to specific achievements',
    examples: [
      { badge: 'Completionist', requirement: 'Unlock all base achievements' },
      { badge: 'Win Streak 10', requirement: '10 consecutive wins' },
      { badge: 'Perfect Game', requirement: 'Win with 10+ eliminations' }
    ],
    exclusivity: 'Permanent, always earnable'
  },

  milestone: {
    name: 'Milestone Badges',
    description: 'Long-term progression markers',
    examples: [
      { badge: 'Level 50', requirement: 'Reach account level 50' },
      { badge: 'Level 100', requirement: 'Reach account level 100' },
      { badge: '365 Days', requirement: 'Play for a full year' },
      { badge: '1000 Matches', requirement: 'Play 1000 matches' }
    ],
    exclusivity: 'Permanent, always earnable'
  },

  collection: {
    name: 'Collection Badges',
    description: 'Cosmetic collection milestones',
    examples: [
      { badge: 'Fashionista', requirement: 'Own 25 outfits' },
      { badge: 'Emote Collector', requirement: 'Own 50 emotes' },
      { badge: 'Full Set', requirement: 'Own complete themed set' }
    ],
    exclusivity: 'Permanent, always earnable'
  },

  community: {
    name: 'Community Badges',
    description: 'Special recognition',
    examples: [
      { badge: 'Content Creator', requirement: 'Verified creator program member' },
      { badge: 'Beta Tester', requirement: 'Participated in beta' },
      { badge: 'Bug Hunter', requirement: 'Reported impactful bugs' },
      { badge: 'Top Contributor', requirement: 'Community recognition' }
    ],
    exclusivity: 'Invite/application only'
  },

  rare: {
    name: 'Ultra-Rare Badges',
    description: 'Extremely difficult achievements',
    examples: [
      { badge: 'First Blood Legend', requirement: 'Get first blood 100 times' },
      { badge: 'Solo Squad Wipe', requirement: 'Eliminate full squad solo' },
      { badge: 'Impossible Victory', requirement: 'Win from 1% health in final circle' }
    ],
    exclusivity: 'Permanent, extremely rare'
  }
};
```

### Badge Display

```typescript
const BADGE_DISPLAY = {
  showcase: {
    location: 'Profile card badge showcase',
    slots: 3,
    selection: 'Player chooses which 3 to display',
    order: 'Player can reorder'
  },

  visual: {
    shape: 'Shield/medallion shape',
    size: '64x64 pixels',
    quality: 'Animated for legendary badges',
    frame: 'Rarity-colored border'
  },

  inspection: {
    hover: 'Show badge name and requirement',
    click: 'Show full details and earn date',
    compare: 'Compare badges with friends'
  }
};
```

---

## Unlock System

### Unlock Sources

```typescript
const UNLOCK_SOURCES = {
  achievements: {
    link: 'Tied to achievement system (LIVE-003)',
    automatic: 'Unlocks when achievement completes',
    notification: 'Badge/title unlock notification'
  },

  seasonEnd: {
    timing: 'Distributed at season end',
    ranked: 'Based on peak rank',
    battlePass: 'Based on completion status'
  },

  events: {
    timing: 'During or after event',
    requirement: 'Complete event milestones',
    claim: 'May require manual claim'
  },

  manual: {
    admin: 'Granted by admin for special cases',
    creator: 'Content creator program badges',
    support: 'Bug hunter, community contributor'
  }
};
```

### Unlock Notification

```typescript
const UNLOCK_NOTIFICATION = {
  inGame: {
    popup: 'Celebratory unlock popup',
    animation: 'Badge/title reveal animation',
    sound: 'Achievement-style sound',
    timing: 'Immediate on unlock (or end of match)'
  },

  display: {
    preview: 'Show the unlocked item',
    requirement: 'Show what was completed',
    rarity: 'Emphasize rare unlocks',
    action: 'Equip now / View in Profile'
  },

  social: {
    share: 'Option to share unlock',
    feed: 'Appear in friends activity feed'
  }
};
```

---

## Profile Integration

### Profile Card

```typescript
const PROFILE_CARD = {
  title: {
    position: 'Below player name',
    display: 'Selected title',
    fallback: 'Player level if no title selected'
  },

  badges: {
    position: 'Badge showcase area',
    slots: 3,
    layout: 'Horizontal row',
    empty: 'Empty slot prompts to earn badges'
  },

  inspection: {
    access: 'Tap on player name anywhere',
    content: [
      'Player name and title',
      'Badge showcase (3 selected)',
      'Level and rank',
      'Quick stats'
    ],
    expand: 'Full profile for more details'
  }
};
```

### Collection View

```typescript
const COLLECTION_VIEW = {
  access: 'Profile > Titles & Badges',

  titles: {
    tabs: 'Organized by category',
    display: 'Grid of all titles (owned and locked)',
    locked: 'Show requirement to unlock',
    equipped: 'Highlight currently equipped',
    action: 'Tap to equip or view requirement'
  },

  badges: {
    tabs: 'Organized by category',
    display: 'Grid of all badges (owned and locked)',
    locked: 'Show requirement and progress',
    showcased: 'Indicate if in showcase',
    action: 'Tap to add to showcase or view details'
  },

  stats: {
    totalOwned: 'X/Y titles, X/Y badges',
    completion: 'Completion percentage',
    rareCount: 'Number of rare/legendary items'
  }
};
```

---

## Rarity System

### Rarity Tiers

```typescript
const RARITY_TIERS = {
  common: {
    color: '#FFFFFF',
    glow: 'None',
    dropRate: '~40% of titles/badges',
    requirement: 'Relatively easy to achieve'
  },

  rare: {
    color: '#0099FF',
    glow: 'Subtle blue',
    dropRate: '~35% of titles/badges',
    requirement: 'Requires moderate dedication'
  },

  epic: {
    color: '#9933FF',
    glow: 'Purple shimmer',
    dropRate: '~20% of titles/badges',
    requirement: 'Significant accomplishment'
  },

  legendary: {
    color: '#FFD700',
    glow: 'Golden animated glow',
    dropRate: '~5% of titles/badges',
    requirement: 'Exceptional achievement or exclusive'
  }
};
```

### Prestige Value

```typescript
const PRESTIGE_VALUE = {
  visibility: {
    inGame: 'Titles visible in match',
    lobby: 'Badges visible on profile',
    leaderboards: 'Top players show badges'
  },

  exclusivity: {
    seasonal: 'Seasonal badges never return',
    ranked: 'Rank badges are permanent records',
    event: 'Event badges may return annually',
    achievement: 'Always earnable but difficulty varies'
  },

  recognition: {
    rare: 'Other players notice rare titles/badges',
    social: 'Creates conversation and aspiration',
    community: 'Community knows what badges mean'
  }
};
```

---

## Technical Implementation

### Data Model

```typescript
const DATA_MODEL = {
  title: {
    id: 'string',
    name: 'string',
    category: 'string',
    rarity: 'enum',
    requirement: 'string (display text)',
    achievementId: 'string (nullable, link to achievement)',
    exclusive: 'boolean',
    releaseSeason: 'string (nullable)'
  },

  badge: {
    id: 'string',
    name: 'string',
    category: 'string',
    rarity: 'enum',
    requirement: 'string (display text)',
    icon: 'string (asset path)',
    achievementId: 'string (nullable)',
    exclusive: 'boolean',
    releaseSeason: 'string (nullable)'
  },

  playerTitleCollection: {
    playerId: 'string',
    ownedTitles: 'array<{titleId, unlockedAt}>',
    equippedTitle: 'string (titleId, nullable)'
  },

  playerBadgeCollection: {
    playerId: 'string',
    ownedBadges: 'array<{badgeId, unlockedAt}>',
    showcaseBadges: 'array<string> (max 3 badgeIds)'
  }
};
```

### API Endpoints

```typescript
const API_ENDPOINTS = {
  titles: {
    getAll: 'GET /api/v1/titles',
    getOwned: 'GET /api/v1/players/{id}/titles',
    equip: 'PUT /api/v1/players/{id}/titles/equipped'
  },

  badges: {
    getAll: 'GET /api/v1/badges',
    getOwned: 'GET /api/v1/players/{id}/badges',
    setShowcase: 'PUT /api/v1/players/{id}/badges/showcase'
  },

  unlock: {
    grant: 'POST /api/v1/players/{id}/titles/{titleId}/grant (admin)',
    revoke: 'DELETE /api/v1/players/{id}/titles/{titleId} (admin)'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Titles unlock on achievement completion',
    'Badges unlock on achievement completion',
    'Title equip/unequip works correctly',
    'Badge showcase selection works (max 3)',
    'Display appears correctly in all locations'
  ],

  visual: [
    'Rarity colors display correctly',
    'Legendary glow effects render',
    'Badge icons display at all sizes',
    'Profile card layout handles all combinations'
  ],

  integration: [
    'Achievement system triggers unlocks',
    'Season end distributes ranked badges',
    'Event completion grants event badges',
    'Notification system shows unlocks'
  ],

  edge: [
    'Player with no titles/badges',
    'Player with all titles/badges',
    'Selecting showcase with fewer than 3 badges',
    'Title/badge granted while offline'
  ]
};
```

---

## Launch Content

### Initial Title Count

```typescript
const LAUNCH_CONTENT = {
  titles: {
    combat: 8,
    victory: 6,
    ranked: 5,
    exploration: 5,
    boss: 4,
    social: 4,
    mastery: 6,
    special: 4,
    total: 42
  },

  badges: {
    achievement: 15,
    milestone: 8,
    collection: 6,
    rankedSeason: 6,
    battlePass: 2,
    event: 4,
    rare: 5,
    total: 46
  },

  totalAtLaunch: 88
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial player titles and badges specification |
