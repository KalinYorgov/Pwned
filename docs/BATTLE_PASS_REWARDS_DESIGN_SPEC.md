# Battle Pass Rewards Design Specification

## Overview

This document defines the reward distribution strategy for the Plunderstorm Mobile Battle Pass, ensuring engaging progression with meaningful rewards that drive both engagement and monetization.

## Reward Design Philosophy

### Core Principles
- **Immediate Gratification**: First rewards feel valuable instantly
- **Steady Progression**: Meaningful reward every 1-2 tiers
- **Milestone Highlights**: Special rewards at key progression points
- **Value Return**: Premium track returns investment value
- **Exclusive Appeal**: Pass-exclusive items drive purchases

### Engagement Curve

```typescript
interface EngagementCurve {
  phases: EngagementPhase[];
}

const ENGAGEMENT_CURVE: EngagementCurve = {
  phases: [
    {
      phase: 'Hook',
      tiers: '1-10',
      rewardDensity: 'High',
      strategy: 'Frontload appealing rewards to hook players',
      premiumHighlight: 'Immediate exclusive outfit at Tier 1'
    },
    {
      phase: 'Investment',
      tiers: '11-30',
      rewardDensity: 'Medium-High',
      strategy: 'Build sunk cost and progression commitment',
      premiumHighlight: 'Mid-tier legendary emote'
    },
    {
      phase: 'Momentum',
      tiers: '31-50',
      rewardDensity: 'Medium',
      strategy: 'Maintain pace with valuable rewards',
      premiumHighlight: 'Exclusive weapon skin set'
    },
    {
      phase: 'Finale',
      tiers: '51-60',
      rewardDensity: 'Very High',
      strategy: 'Celebrate completion with best rewards',
      premiumHighlight: 'Legendary outfit + finale bundle'
    }
  ]
};
```

---

## Reward Categories

### Currency Rewards

```typescript
interface CurrencyRewards {
  gold: GoldRewardConfig;
  doubloons: DoubloonRewardConfig;
}

const CURRENCY_REWARDS: CurrencyRewards = {
  gold: {
    totalInPass: {
      free: 5000,
      premium: 8000
    },
    distribution: [
      { amount: 200, frequency: 'Every 3-4 tiers (free)' },
      { amount: 500, frequency: 'Every 2-3 tiers (premium)' },
      { amount: 1000, frequency: 'Milestone tiers (10, 30, 50)' }
    ],
    purpose: 'Enable F2P progression, supplement premium rewards'
  },

  doubloons: {
    totalInPass: {
      free: 100,
      premium: 1050
    },
    distribution: {
      free: [
        { tier: 15, amount: 25 },
        { tier: 35, amount: 25 },
        { tier: 50, amount: 50 }
      ],
      premium: [
        { tier: 5, amount: 50 },
        { tier: 12, amount: 75 },
        { tier: 20, amount: 100 },
        { tier: 28, amount: 75 },
        { tier: 38, amount: 100 },
        { tier: 45, amount: 150 },
        { tier: 52, amount: 150 },
        { tier: 58, amount: 200 },
        { tier: 60, amount: 150 }
      ]
    },
    valueReturn: {
      passCost: 950,
      premiumDoubloons: 1050,
      returnPercentage: '110%',
      note: 'Completing pass earns more than cost'
    }
  }
};
```

### Cosmetic Rewards

```typescript
interface CosmeticRewards {
  outfits: OutfitRewardConfig;
  emotes: EmoteRewardConfig;
  profileItems: ProfileRewardConfig;
  effects: EffectRewardConfig;
}

const COSMETIC_REWARDS: CosmeticRewards = {
  outfits: {
    totalInPass: {
      free: 1,
      premium: 4
    },
    distribution: {
      free: [
        {
          tier: 45,
          item: 'Deckhand Outfit',
          rarity: 'Rare',
          description: 'Basic but complete outfit for F2P players'
        }
      ],
      premium: [
        {
          tier: 1,
          item: 'Season Starter Outfit',
          rarity: 'Epic',
          exclusive: true,
          description: 'Immediate reward for pass purchase'
        },
        {
          tier: 20,
          item: 'Treasure Hunter Outfit',
          rarity: 'Epic',
          exclusive: true,
          description: 'Early completion reward'
        },
        {
          tier: 40,
          item: 'Storm Sailor Outfit',
          rarity: 'Legendary',
          exclusive: true,
          description: 'Mid-late game reward'
        },
        {
          tier: 60,
          item: 'Tides of Fortune Captain',
          rarity: 'Legendary',
          exclusive: true,
          description: 'Season finale signature outfit',
          specialFeatures: ['Unique VFX', 'Custom idle animation', 'Season badge']
        }
      ]
    }
  },

  emotes: {
    totalInPass: {
      free: 2,
      premium: 6
    },
    distribution: {
      free: [
        { tier: 10, item: 'Basic Wave', rarity: 'Common' },
        { tier: 30, item: 'Sailor Salute', rarity: 'Rare' }
      ],
      premium: [
        { tier: 8, item: 'Victory Jig', rarity: 'Rare', exclusive: true },
        { tier: 18, item: 'Treasure Discovery', rarity: 'Epic', exclusive: true },
        { tier: 25, item: 'Map Reading', rarity: 'Epic', exclusive: true },
        { tier: 35, item: 'Storm Dance', rarity: 'Legendary', exclusive: true },
        { tier: 48, item: 'Captain\'s Command', rarity: 'Legendary', exclusive: true },
        { tier: 55, item: 'Fortune Celebration', rarity: 'Legendary', exclusive: true }
      ]
    }
  },

  profileItems: {
    totalInPass: {
      free: 4,
      premium: 10
    },
    categories: ['Card Backgrounds', 'Card Frames', 'Titles', 'Banners'],
    distribution: 'Spread every 4-6 tiers'
  },

  effects: {
    totalInPass: {
      free: 0,
      premium: 3
    },
    types: [
      { tier: 15, item: 'Victory Trail', type: 'Movement trail' },
      { tier: 32, item: 'Storm Aura', type: 'Idle effect' },
      { tier: 57, item: 'Fortune\'s Glow', type: 'Kill effect' }
    ]
  }
};
```

### Consumable Rewards

```typescript
interface ConsumableRewards {
  xpBoosts: XPBoostConfig;
  crates: CrateConfig;
}

const CONSUMABLE_REWARDS: ConsumableRewards = {
  xpBoosts: {
    totalInPass: {
      free: 3,
      premium: 5
    },
    types: [
      {
        name: 'XP Boost (1 Hour)',
        effect: '+50% Battle Pass XP',
        distribution: 'Tiers 5, 15, 25 (free), 10, 22, 34, 46, 54 (premium)'
      },
      {
        name: 'XP Boost (24 Hours)',
        effect: '+25% Battle Pass XP',
        distribution: 'Tier 42 (premium)'
      }
    ]
  },

  crates: {
    totalInPass: {
      free: 2,
      premium: 4
    },
    types: [
      {
        name: 'Common Crate',
        guaranteedRarity: 'Common',
        distribution: 'Tiers 8, 28 (free)'
      },
      {
        name: 'Rare Crate',
        guaranteedRarity: 'Rare',
        distribution: 'Tiers 12, 36 (premium)'
      },
      {
        name: 'Epic Crate',
        guaranteedRarity: 'Epic',
        distribution: 'Tiers 24, 50 (premium)'
      }
    ]
  }
};
```

---

## Full 60-Tier Reward Table

### Tiers 1-20: Hook Phase

| Tier | Free Track | Premium Track |
|------|------------|---------------|
| 1 | - | **Season Starter Outfit** (Epic, Exclusive) |
| 2 | Gold (200) | Doubloons (50) |
| 3 | - | Card Background: Ocean Depths |
| 4 | - | Gold (500) |
| 5 | XP Boost (1hr) | Doubloons (50) |
| 6 | - | Title: "Fortune Seeker" |
| 7 | - | Gold (500) |
| 8 | Common Crate | **Victory Jig Emote** (Rare, Exclusive) |
| 9 | - | Card Frame: Golden Compass |
| 10 | **Basic Wave Emote** | Gold (1000) + Profile Banner |
| 11 | - | Doubloons (75) |
| 12 | Gold (200) | Rare Crate |
| 13 | - | Card Background: Storm Clouds |
| 14 | - | Gold (500) |
| 15 | Doubloons (25) | **Victory Trail Effect** |
| 16 | - | Title: "Treasure Hunter" |
| 17 | - | Gold (500) |
| 18 | - | **Treasure Discovery Emote** (Epic, Exclusive) |
| 19 | - | Doubloons (100) |
| 20 | Gold (500) | **Treasure Hunter Outfit** (Epic, Exclusive) |

### Tiers 21-40: Investment & Momentum Phase

| Tier | Free Track | Premium Track |
|------|------------|---------------|
| 21 | - | Gold (500) |
| 22 | XP Boost (1hr) | XP Boost (1hr) |
| 23 | - | Card Background: Treasure Map |
| 24 | - | Epic Crate |
| 25 | Gold (200) | **Map Reading Emote** (Epic, Exclusive) |
| 26 | - | Doubloons (75) |
| 27 | - | Title: "Storm Rider" |
| 28 | Common Crate | Gold (500) |
| 29 | - | Card Frame: Ship's Wheel |
| 30 | **Sailor Salute Emote** | Gold (1000) + **Milestone Badge** |
| 31 | - | Doubloons (100) |
| 32 | - | **Storm Aura Effect** |
| 33 | Gold (200) | Card Background: Lightning Strike |
| 34 | - | XP Boost (1hr) |
| 35 | Doubloons (25) | **Storm Dance Emote** (Legendary, Exclusive) |
| 36 | - | Rare Crate |
| 37 | - | Gold (500) |
| 38 | - | Doubloons (100) |
| 39 | - | Title: "Storm Master" |
| 40 | Gold (500) | **Storm Sailor Outfit** (Legendary, Exclusive) |

### Tiers 41-60: Momentum & Finale Phase

| Tier | Free Track | Premium Track |
|------|------------|---------------|
| 41 | - | Gold (500) |
| 42 | - | XP Boost (24hr) |
| 43 | - | Card Background: Golden Sunset |
| 44 | Gold (200) | Doubloons (150) |
| 45 | **Deckhand Outfit** (Rare) | Card Frame: Captain's Crest |
| 46 | - | XP Boost (1hr) |
| 47 | - | Title: "Fortune's Chosen" |
| 48 | - | **Captain's Command Emote** (Legendary, Exclusive) |
| 49 | - | Gold (1000) |
| 50 | Doubloons (50) + XP Boost | Epic Crate + Gold (1000) + **Milestone Badge** |
| 51 | - | Doubloons (150) |
| 52 | - | Card Background: Legendary Horizon |
| 53 | Gold (500) | Title: "Legend of the Seas" |
| 54 | - | XP Boost (1hr) |
| 55 | - | **Fortune Celebration Emote** (Legendary, Exclusive) |
| 56 | - | Gold (500) |
| 57 | - | **Fortune's Glow Effect** (Kill Effect) |
| 58 | Gold (500) | Doubloons (200) |
| 59 | - | Card Frame: Legendary Captain |
| 60 | Profile Banner | **FINALE BUNDLE**: Tides of Fortune Captain (Legendary Outfit) + Season Badge + 150 Doubloons |

---

## Milestone Rewards

### Key Milestone Tiers

```typescript
interface MilestoneRewards {
  milestones: Milestone[];
}

const MILESTONES: MilestoneRewards = {
  milestones: [
    {
      tier: 1,
      name: 'Season Start',
      premiumReward: 'Season Starter Outfit',
      significance: 'Immediate value for pass purchase',
      visualIndicator: 'Golden tier marker'
    },
    {
      tier: 10,
      name: 'First Milestone',
      freeReward: 'Basic Wave Emote',
      premiumReward: 'Gold (1000) + Profile Banner',
      significance: '~1 week of progress',
      visualIndicator: 'Bronze milestone badge'
    },
    {
      tier: 20,
      name: 'Early Achiever',
      premiumReward: 'Treasure Hunter Outfit (Epic)',
      significance: 'First major cosmetic milestone',
      visualIndicator: 'Silver milestone badge'
    },
    {
      tier: 30,
      name: 'Halfway Point',
      freeReward: 'Sailor Salute Emote',
      premiumReward: 'Gold (1000) + Special Milestone Badge',
      significance: 'Mid-season celebration',
      visualIndicator: 'Gold milestone badge + confetti'
    },
    {
      tier: 40,
      name: 'Dedicated Player',
      premiumReward: 'Storm Sailor Outfit (Legendary)',
      significance: 'First legendary outfit',
      visualIndicator: 'Platinum milestone badge'
    },
    {
      tier: 50,
      name: 'Near Completion',
      freeReward: 'Doubloons (50) + XP Boost',
      premiumReward: 'Epic Crate + Gold (1000) + Milestone Badge',
      significance: 'Final stretch begins',
      visualIndicator: 'Diamond milestone badge'
    },
    {
      tier: 60,
      name: 'Season Champion',
      freeReward: 'Profile Banner',
      premiumReward: 'FINALE BUNDLE (Legendary Outfit + Badge + Doubloons)',
      significance: 'Ultimate achievement',
      visualIndicator: 'Animated champion crown'
    }
  ]
};
```

---

## Value Analysis

### Premium Pass Value Breakdown

```typescript
interface ValueAnalysis {
  passCost: number;
  totalRewardValue: RewardValue;
  valueReturn: number;
}

const VALUE_ANALYSIS: ValueAnalysis = {
  passCost: 950, // Doubloons

  totalRewardValue: {
    doubloons: {
      amount: 1050,
      dollarValue: 10.50 // At base rate
    },
    outfits: {
      count: 4,
      estimatedValue: 3600, // Doubloons equivalent
      breakdown: [
        { item: 'Season Starter (Epic)', value: 800 },
        { item: 'Treasure Hunter (Epic)', value: 800 },
        { item: 'Storm Sailor (Legendary)', value: 1200 },
        { item: 'Finale Captain (Legendary)', value: 1200 }
      ]
    },
    emotes: {
      count: 6,
      estimatedValue: 2200,
      breakdown: [
        { item: 'Victory Jig (Rare)', value: 200 },
        { item: 'Treasure Discovery (Epic)', value: 400 },
        { item: 'Map Reading (Epic)', value: 400 },
        { item: 'Storm Dance (Legendary)', value: 600 },
        { item: 'Captain\'s Command (Legendary)', value: 600 },
        { item: 'Fortune Celebration (Legendary)', value: 600 }
      ]
    },
    effects: {
      count: 3,
      estimatedValue: 900
    },
    profileItems: {
      count: 10,
      estimatedValue: 1500
    },
    consumables: {
      xpBoosts: 5,
      crates: 4,
      estimatedValue: 800
    },
    gold: {
      amount: 8000,
      estimatedValue: 400 // Lower relative value
    },

    totalEstimatedValue: 10450 // Doubloons equivalent
  },

  valueReturn: {
    doubloonsReturn: '110%', // More than pass cost
    totalValueMultiplier: '11x', // Total value vs cost
    marketingClaim: 'Over 10,000 Doubloons worth of rewards!'
  }
};
```

### Free Track Value

```typescript
const FREE_TRACK_VALUE = {
  doubloons: {
    amount: 100,
    breakdown: [25, 25, 50]
  },
  gold: {
    amount: 5000
  },
  outfit: {
    count: 1,
    item: 'Deckhand Outfit (Rare)'
  },
  emotes: {
    count: 2,
    items: ['Basic Wave', 'Sailor Salute']
  },
  profileItems: 4,
  xpBoosts: 3,
  crates: 2,

  totalEstimatedValue: 2500, // Doubloons equivalent
  marketingMessage: 'Great rewards just for playing!'
};
```

---

## Exclusivity Guidelines

### Exclusive Items Policy

```typescript
interface ExclusivityPolicy {
  rules: ExclusivityRule[];
}

const EXCLUSIVITY_POLICY: ExclusivityPolicy = {
  rules: [
    {
      itemType: 'Season Signature Outfit (Tier 60)',
      exclusivity: 'PERMANENT',
      willReturn: false,
      description: 'Never available again after season ends',
      marketing: '"Own a piece of Plunderstorm history"'
    },
    {
      itemType: 'Premium Track Outfits',
      exclusivity: 'SEASONAL',
      willReturn: false,
      description: 'Exclusive to this Battle Pass',
      exception: 'May return in "Legacy Pass" after 1+ year'
    },
    {
      itemType: 'Premium Emotes (Legendary)',
      exclusivity: 'SEASONAL',
      willReturn: false,
      description: 'Not available in shop during season'
    },
    {
      itemType: 'Premium Emotes (Epic/Rare)',
      exclusivity: 'TIMED',
      willReturn: true,
      returnPolicy: 'May appear in shop 6+ months later',
      priceInShop: '150% of Battle Pass equivalent value'
    },
    {
      itemType: 'Effects',
      exclusivity: 'SEASONAL',
      willReturn: false,
      description: 'Unique to Battle Pass'
    },
    {
      itemType: 'Profile Items',
      exclusivity: 'TIMED',
      willReturn: true,
      returnPolicy: 'Some may appear in future passes or shop'
    },
    {
      itemType: 'Free Track Items',
      exclusivity: 'NONE',
      willReturn: true,
      description: 'May appear in shop or future events'
    }
  ]
};
```

### Exclusive Tags in UI

```typescript
const EXCLUSIVE_TAGS: ExclusiveTag[] = [
  {
    tag: 'PASS EXCLUSIVE',
    color: 'premium_gold',
    appliesTo: 'Items only in this Battle Pass',
    tooltip: 'Only available through Battle Pass'
  },
  {
    tag: 'SEASON EXCLUSIVE',
    color: 'season_purple',
    appliesTo: 'Tier 60 signature items',
    tooltip: 'Will never be available again'
  },
  {
    tag: 'LIMITED TIME',
    color: 'urgent_red',
    appliesTo: 'Items that may return eventually',
    tooltip: 'Get it now or wait for potential return'
  }
];
```

---

## Reward Variety Requirements

### Variety Distribution

```typescript
interface VarietyRequirements {
  perTierRange: VarietyRule[];
}

const VARIETY_REQUIREMENTS: VarietyRequirements = {
  perTierRange: [
    {
      range: 'Every 10 tiers',
      requirements: [
        'At least 1 outfit piece or full outfit',
        'At least 1 emote',
        'At least 1 currency drop (Gold or Doubloons)',
        'At least 1 profile customization item',
        'No more than 3 consecutive tiers of same reward type'
      ]
    },
    {
      range: 'Full pass (60 tiers)',
      requirements: [
        'Minimum 4 outfits (premium)',
        'Minimum 6 emotes (premium)',
        'Minimum 3 effects (premium)',
        'Total Doubloons >= pass cost',
        'At least 1 legendary item per 15 tiers',
        'Free track has at least 1 outfit'
      ]
    }
  ]
};
```

### Reward Type Balance

```typescript
const REWARD_TYPE_BALANCE = {
  premium: {
    outfits: '7%', // 4/60
    emotes: '10%', // 6/60
    effects: '5%', // 3/60
    currency: '25%', // Gold + Doubloons
    profileItems: '17%', // 10/60
    consumables: '15%', // XP Boosts + Crates
    milestones: '10%' // Special milestone rewards
  },
  rationale: {
    outfits: 'High value, spread across pass',
    emotes: 'Regular engagement rewards',
    currency: 'Consistent value, flexible use',
    consumables: 'Progression boosters'
  }
};
```

---

## Reward Reveal Experience

### Tier-Up Animation

```typescript
interface TierUpExperience {
  animation: AnimationSequence;
}

const TIER_UP_EXPERIENCE: TierUpExperience = {
  animation: {
    trigger: 'XP reaches next tier threshold',
    sequence: [
      {
        step: 1,
        duration: 500,
        action: 'Progress bar fills to 100%',
        sound: 'progress_fill.wav'
      },
      {
        step: 2,
        duration: 300,
        action: 'Tier number increments with pop',
        particles: 'golden_burst'
      },
      {
        step: 3,
        duration: 800,
        action: 'Reward card(s) fly in from sides',
        sound: 'reward_reveal.wav'
      },
      {
        step: 4,
        duration: 500,
        action: 'Card flips to show reward',
        highlight: 'Golden glow for premium'
      },
      {
        step: 5,
        duration: 300,
        action: 'Claim prompt appears',
        buttons: ['CLAIM', 'VIEW DETAILS']
      }
    ],
    skipOption: 'Tap to skip (rewards auto-claimed if auto-claim enabled)'
  }
};
```

### Milestone Celebration

```typescript
const MILESTONE_CELEBRATION = {
  triggers: [10, 20, 30, 40, 50, 60],
  specialEffects: {
    tier10: 'Bronze confetti burst',
    tier20: 'Silver confetti + fanfare',
    tier30: 'Gold confetti + special badge reveal',
    tier40: 'Platinum effects + legendary outfit preview',
    tier50: 'Diamond effects + near-completion message',
    tier60: 'Full celebration: fireworks, confetti, achievement popup'
  },
  fullScreenTakeover: true,
  duration: '3-5 seconds',
  sharePrompt: 'Share your achievement?'
};
```

---

## Design Spreadsheet Template

### Reward Planning Sheet

```typescript
interface RewardSpreadsheet {
  columns: SpreadsheetColumn[];
}

const SPREADSHEET_TEMPLATE: RewardSpreadsheet = {
  columns: [
    { name: 'Tier', type: 'number', range: '1-60' },
    { name: 'Free Reward', type: 'text', nullable: true },
    { name: 'Free Reward Type', type: 'enum', options: REWARD_TYPES },
    { name: 'Free Reward Value', type: 'number', currency: 'Doubloons' },
    { name: 'Premium Reward', type: 'text', required: true },
    { name: 'Premium Reward Type', type: 'enum', options: REWARD_TYPES },
    { name: 'Premium Rarity', type: 'enum', options: RARITIES },
    { name: 'Premium Value', type: 'number', currency: 'Doubloons' },
    { name: 'Exclusive', type: 'boolean' },
    { name: 'Milestone', type: 'boolean' },
    { name: 'Notes', type: 'text' }
  ],
  calculations: [
    { name: 'Total Free Value', formula: 'SUM(Free Reward Value)' },
    { name: 'Total Premium Value', formula: 'SUM(Premium Value)' },
    { name: 'Doubloon Return %', formula: 'Premium Doubloons / Pass Cost' },
    { name: 'Value Multiplier', formula: 'Total Premium Value / Pass Cost' }
  ],
  validation: [
    'Premium Value Total >= Pass Cost * 5',
    'Doubloon Total >= Pass Cost',
    'At least 1 reward per 2 tiers (free track)',
    'Exactly 1 reward per tier (premium track)'
  ]
};
```

---

## Season Theme Integration

### Theme Application

```typescript
interface ThemeIntegration {
  seasonTheme: string;
  themeApplication: ThemeRule[];
}

const THEME_INTEGRATION: ThemeIntegration = {
  seasonTheme: 'Tides of Fortune',

  themeApplication: [
    {
      category: 'Outfits',
      themeElements: [
        'Nautical color palette (deep blues, golds, sea greens)',
        'Fortune/treasure motifs',
        'Progressive complexity (deckhand → captain)',
        'Weather elements (storm themes)'
      ]
    },
    {
      category: 'Emotes',
      themeElements: [
        'Treasure hunting actions',
        'Sea shanty references',
        'Weather/storm celebrations',
        'Fortune-related gestures'
      ]
    },
    {
      category: 'Effects',
      themeElements: [
        'Golden particle trails',
        'Storm/lightning effects',
        'Treasure chest imagery'
      ]
    },
    {
      category: 'Profile Items',
      themeElements: [
        'Compass and map frames',
        'Ocean/weather backgrounds',
        'Fortune-themed titles'
      ]
    }
  ]
};
```

---

## Implementation Checklist

### Reward Content
- [ ] 60 premium track rewards defined
- [ ] 25 free track rewards defined
- [ ] 4 premium outfits created
- [ ] 6 premium emotes created
- [ ] 3 effects created
- [ ] All profile items designed

### Value Validation
- [ ] Doubloon return >= 100% of pass cost
- [ ] Total value >= 10x pass cost
- [ ] Free track provides meaningful progression

### Variety Checks
- [ ] No 3+ consecutive same reward types
- [ ] Legendary item every 15 tiers
- [ ] Currency rewards well-distributed
- [ ] Milestone rewards at key tiers

### Exclusivity
- [ ] Exclusive tags applied correctly
- [ ] Return policy documented
- [ ] Marketing messaging prepared

### Documentation
- [ ] Full reward spreadsheet completed
- [ ] Theme integration verified
- [ ] Value calculations validated

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
