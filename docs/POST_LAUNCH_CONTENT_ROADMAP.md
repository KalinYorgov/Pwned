# Post-Launch Content Roadmap Specification

## Document Information
- **Task ID:** PROJ-014
- **Priority:** P1
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document outlines the 12-month post-launch content roadmap for Plunderstorm Mobile, detailing new abilities, cosmetics, seasonal events, Battle Pass seasons, map content, and major features planned for Year 1.

---

## Year 1 Roadmap Overview

### High-Level Timeline

```
MONTH 1-3 (Q1): Foundation & Growth
├── Season 1: Launch Season
├── Season 2: Rising Tide
├── 4 new abilities
├── Valentine's Event
└── Ranked Mode (Beta)

MONTH 4-6 (Q2): Expansion
├── Season 3: Kraken's Wake
├── Season 4: Treasure Hunt
├── 4 new abilities
├── Easter Event
├── New Map Area: Sunken City
└── Guild System

MONTH 7-9 (Q3): Maturation
├── Season 5: Storm Surge
├── Season 6: Cursed Waters
├── 4 new abilities
├── Summer Event
└── Ranked Season 1 (Full)

MONTH 10-12 (Q4): Celebration
├── Season 7: Ghost Fleet
├── Season 8: Anniversary
├── 4 new abilities
├── Halloween Event
├── Winter Event
├── New Map: Volcanic Island
└── Custom Lobbies
```

---

## Detailed Quarter 1 Plan

### Month 1: Launch & Stabilization

#### Week 1-2: Launch Window
```typescript
const LAUNCH_WINDOW_FOCUS = {
  stability: {
    serverMonitoring: '24/7 coverage',
    hotfixReadiness: 'Same-day deployment capability',
    crashRateTarget: '< 1%'
  },
  content: {
    battlePass: 'Season 1: Launch Season',
    shopRotation: 'Daily featured items',
    ltmSchedule: 'Standard modes only'
  },
  community: {
    responseTime: '< 4 hours for critical issues',
    socialCadence: '2-3 posts per day',
    streamSchedule: 'Launch celebration stream'
  }
};
```

#### Week 3-4: First Update
- **Minor Update 1.1**
  - Balance adjustments based on launch data
  - Bug fixes from launch week
  - QoL improvements from community feedback
  - First shop rotation refresh

### Month 2: First Major Content Drop

#### Season 1 Progress
- Week 5-6: Mid-season event
- Week 7-8: Season 1 finale preparation

#### New Ability #1: Smoke Bomb
```typescript
const SMOKE_BOMB_ABILITY = {
  name: 'Smoke Bomb',
  type: 'Tactical',
  rarity: 'Epic',
  releaseWeek: 5,

  mechanics: {
    throwDistance: 15, // meters
    cloudRadius: 8, // meters
    duration: 6, // seconds
    effects: [
      'Blocks enemy vision',
      'Highlights enemies inside (to user)',
      'Reduces incoming damage by 15%'
    ]
  },

  unlockMethod: {
    free: 'Battle Pass Level 45',
    premium: 'Battle Pass Level 1',
    shop: 'Available after Season 2'
  }
};
```

#### New Ability #2: Chain Lightning
```typescript
const CHAIN_LIGHTNING_ABILITY = {
  name: 'Chain Lightning',
  type: 'Offensive',
  rarity: 'Legendary',
  releaseWeek: 7,

  mechanics: {
    damage: 60, // initial target
    chainDamage: 40, // chained targets
    maxChains: 3,
    chainRange: 10, // meters
    cooldown: 25 // seconds
  },

  unlockMethod: {
    free: 'Season 2 Battle Pass',
    shop: '1500 Doubloons'
  }
};
```

#### Valentine's Day Event (Week 6-7)
```typescript
const VALENTINES_EVENT = {
  name: 'Pirate\'s Heart',
  duration: {
    start: 'February 10',
    end: 'February 17'
  },

  theme: {
    mapDecor: 'Heart-shaped islands, rose petals in water',
    musicOverlay: 'Romantic sea shanty remix',
    uiTheme: 'Pink and red accents'
  },

  content: {
    ltm: {
      name: 'Duo\'s Delight',
      description: 'Duo-only mode with shared abilities',
      exclusive: true
    },
    cosmetics: [
      { type: 'Skin', name: 'Cupid Captain', rarity: 'Epic' },
      { type: 'Hat', name: 'Heart Crown', rarity: 'Rare' },
      { type: 'Trail', name: 'Rose Petals', rarity: 'Epic' },
      { type: 'Emote', name: 'Blow Kiss', rarity: 'Rare' }
    ],
    challenges: {
      count: 7,
      rewards: 'Event-exclusive spray and banner'
    }
  }
};
```

### Month 3: Ranked Beta & Season 2

#### Ranked Mode Beta Launch
```typescript
const RANKED_BETA_CONFIG = {
  launchWeek: 9,

  ranks: [
    { name: 'Deckhand', tier: 1, divisions: 4 },
    { name: 'Sailor', tier: 2, divisions: 4 },
    { name: 'Mate', tier: 3, divisions: 4 },
    { name: 'Officer', tier: 4, divisions: 4 },
    { name: 'Captain', tier: 5, divisions: 4 },
    { name: 'Admiral', tier: 6, divisions: 1 },
    { name: 'Pirate Legend', tier: 7, divisions: 1 }
  ],

  matchmaking: {
    algorithm: 'Elo-based with placement matches',
    placementMatches: 10,
    rankDecay: false, // Not in beta
    seasonLength: '6 weeks (beta)'
  },

  rewards: {
    seasonal: [
      'Exclusive rank frame',
      'Rank-specific spray',
      'Premium currency (top 500)'
    ]
  }
};
```

#### Season 2: Rising Tide
```typescript
const SEASON_2_OVERVIEW = {
  name: 'Rising Tide',
  duration: {
    start: 'Week 9',
    end: 'Week 16',
    totalWeeks: 8
  },

  theme: {
    narrative: 'A mysterious tide brings new dangers',
    aesthetics: 'Deeper blues, bioluminescent accents',
    newPOI: 'The Sunken Shrine (preview)'
  },

  battlePass: {
    freeTier: {
      rewards: 50,
      highlights: [
        'New ability: Tidal Wave (Level 35)',
        'Uncommon skin at Level 50'
      ]
    },
    premiumTier: {
      price: 950, // Doubloons
      rewards: 100,
      highlights: [
        'Legendary skin: Deep Sea Diver',
        'Instant ability unlock: Tidal Wave',
        'Epic weapon skin set'
      ]
    }
  }
};
```

---

## Quarter 2-4 Content Overview

### Quarter 2 (Months 4-6)

#### Season 3: Kraken's Wake
- **Theme:** Sea monsters awakened
- **New Map Area:** Sunken City (underwater ruins)
- **New Abilities:**
  - Tentacle Grab (Epic) - Pull enemies toward you
  - Ink Cloud (Rare) - AoE blind effect

#### Season 4: Treasure Hunt
- **Theme:** Lost treasure maps, gold rush
- **New Feature:** Guild System
- **New Abilities:**
  - Metal Detector (Rare) - Locate nearby loot
  - Booby Trap (Epic) - Place explosive trap

#### Easter Event
- **Name:** "Golden Egg Hunt"
- **LTM:** Egg collection mode with special rewards
- **Cosmetics:** Bunny pirate themed items

### Quarter 3 (Months 7-9)

#### Season 5: Storm Surge
- **Theme:** Elemental storms, chaos
- **Major Feature:** Ranked Season 1 (Full Launch)
- **New Abilities:**
  - Lightning Rod (Legendary) - Redirect lightning damage
  - Wind Dash (Epic) - Enhanced mobility in storms

#### Season 6: Cursed Waters
- **Theme:** Ghostly pirates, supernatural
- **New Abilities:**
  - Phantom Form (Legendary) - Temporary invulnerability + invisibility
  - Curse Mark (Epic) - Tag enemies for team damage bonus

#### Summer Event
- **Name:** "Beach Brawl"
- **Duration:** 3 weeks
- **LTM:** Beach-themed arena mode
- **Cosmetics:** Swimwear pirates, beach toys as weapons

### Quarter 4 (Months 10-12)

#### Season 7: Ghost Fleet
- **Theme:** Haunted ships, undead crew
- **New Map:** Volcanic Island
- **New Abilities:**
  - Cannon Barrage (Legendary) - Call artillery strike
  - Ghost Ship (Epic) - Summon spectral vessel

#### Season 8: Anniversary
- **Theme:** Celebration of Year 1
- **Major Feature:** Custom Lobbies
- **Special Content:**
  - Returning limited items
  - Anniversary Battle Pass with callbacks
  - Community choice cosmetics

#### Halloween Event
- **Name:** "Curse of the Black Pearl"
- **Duration:** 3 weeks
- **LTM:** Survival horror mode
- **Cosmetics:** Monster pirates, spooky effects

#### Winter Event
- **Name:** "Frostbeard's Festival"
- **Duration:** 4 weeks (covers holidays)
- **LTM:** Winter wonderland map variant
- **Cosmetics:** Holiday pirates, festive weapons

---

## Content Calendar

### Year 1 Content Calendar Matrix

```
|  Month  |   Season    | Major Event | New Abilities | Major Feature |
|---------|-------------|-------------|---------------|---------------|
| Month 1 | S1: Launch  |      -      |       -       |    Launch     |
| Month 2 | S1: Launch  | Valentine's | Smoke Bomb,   |       -       |
|         |             |             | Chain Light.  |               |
| Month 3 | S2: Rising  |      -      | Tidal Wave,   | Ranked Beta   |
|         | Tide        |             | Water Shield  |               |
| Month 4 | S3: Kraken  | Easter      | Tentacle Grab |       -       |
| Month 5 | S3: Kraken  |      -      | Ink Cloud     | Sunken City   |
| Month 6 | S4: Treasure|      -      | Metal Detect, | Guild System  |
|         | Hunt        |             | Booby Trap    |               |
| Month 7 | S5: Storm   | Summer      | Lightning Rod | Ranked S1     |
| Month 8 | S5: Storm   |      -      | Wind Dash     |       -       |
| Month 9 | S6: Cursed  |      -      | Phantom Form, |       -       |
|         |             |             | Curse Mark    |               |
| Month 10| S7: Ghost   | Halloween   | Cannon Barrage|       -       |
| Month 11| S7: Ghost   |      -      | Ghost Ship    | Volcanic Isle |
| Month 12| S8: Anniv.  | Winter      | Community     | Custom        |
|         |             |             | Choice        | Lobbies       |
```

### Weekly Content Cadence

```typescript
const WEEKLY_CONTENT_CADENCE = {
  monday: {
    activities: [
      'New weekly challenges released',
      'Shop rotation (featured section)',
      'LTM rotation check'
    ]
  },

  tuesday: {
    activities: [
      'Maintenance window (if needed)',
      'Hotfix deployment day',
      'Community blog post'
    ]
  },

  wednesday: {
    activities: [
      'Mid-week social content',
      'Esports/competitive highlights'
    ]
  },

  thursday: {
    activities: [
      'New LTM rotation (bi-weekly)',
      'Dev update stream (monthly)'
    ]
  },

  friday: {
    activities: [
      'Weekend event launch',
      'Shop weekend special'
    ]
  },

  saturday_sunday: {
    activities: [
      'Weekend bonus events',
      'Community tournaments',
      'Double XP windows'
    ]
  }
};
```

### Monthly Deliverables Schedule

```typescript
const MONTHLY_DELIVERABLES = {
  week1: {
    focus: 'Season/Event maintenance',
    deliverables: [
      'Patch notes for major update',
      'Community recap of previous month',
      'Upcoming content teaser'
    ]
  },

  week2: {
    focus: 'Mid-content drop',
    deliverables: [
      'New ability or cosmetic bundle',
      'QoL update',
      'Bug fix patch'
    ]
  },

  week3: {
    focus: 'Engagement push',
    deliverables: [
      'Special weekend event',
      'Community challenge',
      'Creator spotlight'
    ]
  },

  week4: {
    focus: 'Season transition prep',
    deliverables: [
      'Season finale events',
      'Next season teaser',
      'Balance preview'
    ]
  }
};
```

---

## Resource Allocation Plan

### Team Structure for Live Ops

```typescript
const LIVE_OPS_TEAM = {
  core: {
    producer: {
      count: 1,
      responsibilities: [
        'Content scheduling',
        'Team coordination',
        'Stakeholder communication'
      ]
    },
    gameDesigner: {
      count: 2,
      responsibilities: [
        'New ability design (1)',
        'Balance and economy (1)',
        'Event design (shared)'
      ]
    },
    programmer: {
      count: 3,
      responsibilities: [
        'Feature development (1)',
        'Live ops tools (1)',
        'Bug fixes and maintenance (1)'
      ]
    },
    artist: {
      count: 2,
      responsibilities: [
        'Cosmetic creation (1)',
        'UI/Marketing (1)'
      ]
    },
    qa: {
      count: 2,
      responsibilities: [
        'Update testing (1)',
        'Live monitoring (1)'
      ]
    },
    communityManager: {
      count: 1,
      responsibilities: [
        'Social media',
        'Discord management',
        'Player feedback collection'
      ]
    }
  },

  outsourced: {
    animation: 'Per ability, ~2 weeks lead time',
    audio: 'Per event, ~3 weeks lead time',
    localization: 'Per update, ~1 week lead time',
    customerSupport: 'Scaled based on player count'
  },

  totalHeadcount: {
    internal: 11,
    outsourcedFTE: '~2-3 equivalent'
  }
};
```

### Content Production Pipeline

```typescript
const CONTENT_PIPELINE = {
  newAbility: {
    design: {
      duration: '2 weeks',
      team: ['Game Designer'],
      deliverable: 'Ability design document'
    },
    implementation: {
      duration: '2 weeks',
      team: ['Programmer', 'QA'],
      deliverable: 'Functional ability'
    },
    art: {
      duration: '3 weeks (parallel)',
      team: ['Artist', 'Outsourced Animation'],
      deliverable: 'VFX, UI icons, animations'
    },
    audio: {
      duration: '1 week',
      team: ['Outsourced Audio'],
      deliverable: 'SFX'
    },
    testing: {
      duration: '1 week',
      team: ['QA', 'Design'],
      deliverable: 'Balance validation'
    },
    totalLeadTime: '6 weeks'
  },

  cosmeticBundle: {
    concept: '3 days',
    production: '1-2 weeks',
    review: '2 days',
    totalLeadTime: '2-3 weeks'
  },

  seasonalEvent: {
    planning: '4 weeks before',
    artProduction: '4-6 weeks before',
    implementation: '3 weeks before',
    testing: '2 weeks before',
    totalLeadTime: '6-8 weeks'
  },

  battlePassSeason: {
    planning: '8 weeks before',
    contentProduction: '6-8 weeks',
    testing: '2 weeks',
    totalLeadTime: '10-12 weeks'
  }
};
```

### Resource Budget Allocation

```typescript
const ANNUAL_RESOURCE_BUDGET = {
  // Time allocation per quarter (percentages)
  q1: {
    stabilization: 30,
    newFeatures: 25,
    contentCreation: 30,
    bugFixing: 15
  },

  q2: {
    stabilization: 15,
    newFeatures: 35,
    contentCreation: 35,
    bugFixing: 15
  },

  q3: {
    stabilization: 10,
    newFeatures: 30,
    contentCreation: 40,
    bugFixing: 20
  },

  q4: {
    stabilization: 10,
    newFeatures: 25,
    contentCreation: 45,
    bugFixing: 20
  },

  // Content budget per quarter
  contentBudget: {
    newAbilities: 4, // per quarter
    cosmeticSets: 20, // per quarter (various rarities)
    mapUpdates: 1, // per quarter (Q2, Q4 major)
    events: 1, // major per quarter
    battlePassSeasons: 2 // per quarter
  }
};
```

---

## Community Teaser Strategy

### Pre-Launch Teasers

```typescript
const TEASER_SCHEDULE = {
  monthlyTeasers: {
    format: [
      'Silhouette reveal (2 weeks before)',
      'Ability/feature name reveal (1 week before)',
      'Full reveal trailer (3 days before)',
      'Dev blog with details (launch day)'
    ],
    channels: [
      'Twitter/X',
      'Discord announcements',
      'Instagram stories',
      'YouTube Community',
      'In-game news panel'
    ]
  },

  seasonTeasers: {
    week_minus_3: 'Cryptic lore post',
    week_minus_2: 'Theme reveal video',
    week_minus_1: 'Battle Pass trailer',
    launchDay: 'Full patch notes + gameplay'
  }
};
```

### Community Engagement Content

```typescript
const COMMUNITY_CONTENT_PLAN = {
  weekly: [
    'Highlight clip of the week',
    'Community art feature',
    'Tips and tricks post',
    'Meme Monday participation'
  ],

  monthly: [
    'Developer AMA (Discord/Reddit)',
    'Balance philosophy blog',
    'Behind the scenes: Making of [content]',
    'Community stats infographic'
  ],

  seasonal: [
    'Season recap video',
    'Top plays compilation',
    'Community tournament',
    'Roadmap preview stream'
  ]
};
```

### Year 1 Teaser Moments

```typescript
const YEAR_ONE_TEASER_MOMENTS = {
  launch: {
    message: 'The storm is here. Will you conquer it?',
    content: 'Launch trailer, social blitz'
  },

  month2: {
    message: 'Love is in the sea air...',
    content: 'Valentine\'s event teaser'
  },

  month3: {
    message: 'Prove yourself. Ranked arrives.',
    content: 'Ranked mode announcement trailer'
  },

  month5: {
    message: 'Beneath the waves, something stirs...',
    content: 'Sunken City map teaser'
  },

  month6: {
    message: 'Stronger together. Guilds are coming.',
    content: 'Guild system announcement'
  },

  month7: {
    message: 'The heat is on! Beach Brawl begins.',
    content: 'Summer event trailer'
  },

  month10: {
    message: 'The veil thins... What lurks beyond?',
    content: 'Halloween event teaser'
  },

  month11: {
    message: 'Fire and fury. New lands await.',
    content: 'Volcanic Island reveal'
  },

  month12: {
    message: 'One year of plunder. Thank you, pirates!',
    content: 'Anniversary celebration + Year 2 teaser'
  }
};
```

---

## Success Metrics

### Content Performance KPIs

```typescript
const CONTENT_KPIS = {
  battlePass: {
    purchaseRate: '>= 15% of MAU',
    completionRate: '>= 40% reach level 50',
    satisfactionScore: '>= 4.0/5.0'
  },

  newAbility: {
    adoptionRate: '>= 30% use within first week',
    balanceHealth: 'Within 5% of average win rate',
    playerFeedback: '>= 70% positive sentiment'
  },

  event: {
    participationRate: '>= 60% of DAU',
    completionRate: '>= 25% complete all challenges',
    revenueUplift: '>= 20% vs non-event period'
  },

  season: {
    retentionImpact: 'D30 retention stable or improved',
    revenueTarget: 'Meet quarterly targets',
    contentVolume: 'Deliver all planned content'
  }
};
```

### Review Cadence

```typescript
const REVIEW_SCHEDULE = {
  weekly: {
    meeting: 'Content Review',
    participants: ['Producer', 'Design Lead', 'Art Lead'],
    agenda: [
      'Current content progress',
      'Blockers and risks',
      'Community feedback review'
    ]
  },

  monthly: {
    meeting: 'Roadmap Review',
    participants: ['Full team', 'Stakeholders'],
    agenda: [
      'Month performance review',
      'KPI analysis',
      'Roadmap adjustments',
      'Resource reallocation if needed'
    ]
  },

  quarterly: {
    meeting: 'Strategic Planning',
    participants: ['Leadership', 'Department heads'],
    agenda: [
      'Quarter retrospective',
      'Next quarter planning',
      'Annual roadmap updates',
      'Budget review'
    ]
  }
};
```

---

## Appendix

### Ability Release Schedule Detail

| Ability | Type | Rarity | Release | Battle Pass |
|---------|------|--------|---------|-------------|
| Smoke Bomb | Tactical | Epic | Month 2 | S1 Level 45 |
| Chain Lightning | Offensive | Legendary | Month 2 | S2 Premium |
| Tidal Wave | Offensive | Epic | Month 3 | S2 Level 35 |
| Water Shield | Defensive | Rare | Month 3 | S2 Level 20 |
| Tentacle Grab | Offensive | Epic | Month 4 | S3 Level 45 |
| Ink Cloud | Tactical | Rare | Month 5 | S3 Level 25 |
| Metal Detector | Utility | Rare | Month 6 | S4 Level 15 |
| Booby Trap | Tactical | Epic | Month 6 | S4 Level 40 |
| Lightning Rod | Defensive | Legendary | Month 7 | S5 Premium |
| Wind Dash | Mobility | Epic | Month 8 | S5 Level 45 |
| Phantom Form | Tactical | Legendary | Month 9 | S6 Premium |
| Curse Mark | Offensive | Epic | Month 9 | S6 Level 40 |
| Cannon Barrage | Offensive | Legendary | Month 10 | S7 Premium |
| Ghost Ship | Mobility | Epic | Month 11 | S7 Level 45 |
| Community Choice | TBD | Epic | Month 12 | S8 Level 35 |
| Anniversary Special | TBD | Legendary | Month 12 | S8 Premium |

### Event Calendar

| Event | Month | Duration | Type |
|-------|-------|----------|------|
| Valentine's - Pirate's Heart | 2 | 1 week | Themed |
| Easter - Golden Egg Hunt | 4 | 2 weeks | Themed |
| Summer - Beach Brawl | 7 | 3 weeks | Major |
| Halloween - Curse of Black Pearl | 10 | 3 weeks | Major |
| Winter - Frostbeard's Festival | 12 | 4 weeks | Major |
| Anniversary | 12 | 2 weeks | Celebration |

### Battle Pass Season Schedule

| Season | Name | Duration | Key Content |
|--------|------|----------|-------------|
| S1 | Launch Season | Weeks 1-8 | Foundation content |
| S2 | Rising Tide | Weeks 9-16 | Tidal abilities, ranked beta |
| S3 | Kraken's Wake | Weeks 17-24 | Sea monsters theme |
| S4 | Treasure Hunt | Weeks 25-32 | Guilds launch |
| S5 | Storm Surge | Weeks 33-40 | Full ranked launch |
| S6 | Cursed Waters | Weeks 41-48 | Supernatural theme |
| S7 | Ghost Fleet | Weeks 49-56 | Halloween, new map |
| S8 | Anniversary | Weeks 57-64 | Year 1 celebration |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial roadmap specification |
