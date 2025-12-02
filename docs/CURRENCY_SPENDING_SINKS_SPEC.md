# Currency Spending Sinks Specification

## Overview

This document defines all currency spending destinations for Plunderstorm Mobile, establishing a balanced economy where Gold (soft currency) provides meaningful progression through gameplay while Doubloons (hard currency) offer premium value for paying players.

## Design Philosophy

### Economic Balance Goals
- **Gold Accessibility**: Average player earns enough for meaningful purchases in 1-2 weeks
- **Premium Value**: Doubloon items feel exclusive without being mandatory
- **Sink Variety**: Multiple spending options prevent currency hoarding
- **Price Anchoring**: Clear value perception at each price tier

### Target Metrics
- Gold sink ratio: 70-80% of earned Gold spent within 30 days
- Doubloon retention: 40-60% of purchased Doubloons spent within 7 days
- Free-to-paid conversion: Items feel attainable but premium options compelling

---

## Gold Spending Categories

### 1. Common Cosmetic Outfits

```typescript
interface GoldOutfitPricing {
  category: 'Common Outfit';
  currency: 'Gold';
  priceRange: {
    minimum: 800;
    maximum: 2000;
  };
}

const GOLD_OUTFIT_TIERS: OutfitPriceTier[] = [
  {
    tier: 'Basic',
    goldPrice: 800,
    description: 'Simple color variants and basic designs',
    examples: ['Deckhand Garb', 'Sailor Uniform', 'Harbor Worker Outfit'],
    timeToEarn: '3-4 days of casual play'
  },
  {
    tier: 'Standard',
    goldPrice: 1200,
    description: 'Unique designs with moderate detail',
    examples: ['Merchant Attire', 'Tavern Keeper', 'Island Explorer'],
    timeToEarn: '5-6 days of casual play'
  },
  {
    tier: 'Quality',
    goldPrice: 1600,
    description: 'Detailed outfits with accessories',
    examples: ['Seasoned Sailor', 'Navigator Gear', 'First Mate Uniform'],
    timeToEarn: '7-8 days of casual play'
  },
  {
    tier: 'Premium Common',
    goldPrice: 2000,
    description: 'High-quality common tier items',
    examples: ['Veteran Pirate', 'Storm Chaser', 'Treasure Hunter'],
    timeToEarn: '9-10 days of casual play'
  }
];
```

### 2. Emotes

```typescript
interface EmotePricing {
  category: 'Emote';
  currency: 'Gold';
  priceRange: {
    minimum: 1000;
    maximum: 3000;
  };
}

const EMOTE_PRICE_TIERS: EmotePriceTier[] = [
  {
    tier: 'Basic Emote',
    goldPrice: 1000,
    description: 'Simple animations and gestures',
    examples: ['Wave', 'Salute', 'Thumbs Up', 'Point'],
    animationDuration: '1-2 seconds'
  },
  {
    tier: 'Standard Emote',
    goldPrice: 1500,
    description: 'Character animations with personality',
    examples: ['Dance Jig', 'Laugh', 'Taunt', 'Victory Pose'],
    animationDuration: '2-3 seconds'
  },
  {
    tier: 'Complex Emote',
    goldPrice: 2000,
    description: 'Multi-stage animations with effects',
    examples: ['Sword Flourish', 'Map Reading', 'Telescope Look'],
    animationDuration: '3-4 seconds'
  },
  {
    tier: 'Epic Emote',
    goldPrice: 2500,
    description: 'Elaborate animations with sound effects',
    examples: ['Sea Shanty', 'Cannon Fire Celebration', 'Treasure Discovery'],
    animationDuration: '4-5 seconds',
    includesSound: true
  },
  {
    tier: 'Legendary Emote',
    goldPrice: 3000,
    description: 'Premium animations with particle effects',
    examples: ['Kraken Summon', 'Storm Dance', 'Ghost Ship Vision'],
    animationDuration: '5-6 seconds',
    includesSound: true,
    includesParticles: true
  }
];
```

### 3. Profile Customization

```typescript
interface ProfileCustomizationPricing {
  category: 'Profile';
  currency: 'Gold';
}

const PROFILE_ITEMS: ProfileItemPrice[] = [
  // Player Card Backgrounds
  {
    type: 'Card Background',
    tier: 'Common',
    goldPrice: 200,
    description: 'Basic color/gradient backgrounds'
  },
  {
    type: 'Card Background',
    tier: 'Rare',
    goldPrice: 400,
    description: 'Themed backgrounds with subtle patterns'
  },
  {
    type: 'Card Background',
    tier: 'Epic',
    goldPrice: 600,
    description: 'Animated or detailed scene backgrounds'
  },

  // Player Card Frames
  {
    type: 'Card Frame',
    tier: 'Common',
    goldPrice: 300,
    description: 'Simple border designs'
  },
  {
    type: 'Card Frame',
    tier: 'Rare',
    goldPrice: 500,
    description: 'Ornate border with nautical themes'
  },
  {
    type: 'Card Frame',
    tier: 'Epic',
    goldPrice: 800,
    description: 'Animated frames with effects'
  },

  // Player Titles
  {
    type: 'Title',
    tier: 'Common',
    goldPrice: 250,
    description: 'Basic titles',
    examples: ['Sailor', 'Adventurer', 'Seafarer']
  },
  {
    type: 'Title',
    tier: 'Rare',
    goldPrice: 500,
    description: 'Themed titles',
    examples: ['Storm Rider', 'Wave Walker', 'Tide Turner']
  },
  {
    type: 'Title',
    tier: 'Epic',
    goldPrice: 1000,
    description: 'Prestigious titles',
    examples: ['Sea Legend', 'Pirate King', 'Ocean Master']
  }
];
```

### 4. Name Change

```typescript
interface NameChangeService {
  type: 'Service';
  name: 'Name Change';
  goldPrice: 500;
  restrictions: {
    cooldown: '30 days between changes';
    characterLimit: '3-16 characters';
    profanityFilter: true;
    duplicateCheck: true;
  };
  freebies: {
    firstNameChange: 'Free for first 7 days after account creation';
  };
}
```

### 5. Loot Crate Keys (Optional Sink)

```typescript
interface LootCrateKeyPricing {
  type: 'Consumable';
  name: 'Common Crate Key';
  goldPrice: 500;
  crateContents: {
    possibleItems: ['Common Outfit Piece', 'Basic Emote', 'Profile Item', 'Gold Bonus'],
    guaranteedRarity: 'Common',
    epicChance: 0.05
  };
}

const CRATE_KEYS: CrateKeyConfig[] = [
  {
    keyType: 'Common Key',
    goldPrice: 500,
    crateRarity: 'Common',
    dropRates: {
      common: 0.70,
      rare: 0.25,
      epic: 0.05
    }
  },
  {
    keyType: 'Rare Key',
    goldPrice: 1000,
    crateRarity: 'Rare',
    dropRates: {
      common: 0.40,
      rare: 0.45,
      epic: 0.15
    }
  }
];
```

---

## Doubloon Spending Categories

### 1. Premium Cosmetic Outfits

```typescript
interface DoubloonOutfitPricing {
  category: 'Premium Outfit';
  currency: 'Doubloons';
  priceRange: {
    minimum: 500;
    maximum: 2000;
  };
}

const DOUBLOON_OUTFIT_TIERS: PremiumOutfitTier[] = [
  {
    tier: 'Rare',
    doubloonPrice: 500,
    usdEquivalent: 4.99,
    description: 'Unique designs not available for Gold',
    examples: ['Crimson Captain', 'Emerald Navigator', 'Azure Admiral'],
    features: ['Unique color schemes', 'Exclusive patterns']
  },
  {
    tier: 'Epic',
    doubloonPrice: 800,
    usdEquivalent: 7.99,
    description: 'High-quality outfits with accessories',
    examples: ['Ghost Pirate', 'Volcanic Raider', 'Ice Corsair'],
    features: ['Matching accessories', 'Custom textures', 'Slight VFX']
  },
  {
    tier: 'Legendary',
    doubloonPrice: 1200,
    usdEquivalent: 11.99,
    description: 'Top-tier outfits with unique effects',
    examples: ['Kraken Lord', 'Storm Sovereign', 'Leviathan Hunter'],
    features: ['Full VFX', 'Custom animations', 'Unique sound effects']
  },
  {
    tier: 'Mythic',
    doubloonPrice: 2000,
    usdEquivalent: 19.99,
    description: 'Ultra-rare collector outfits',
    examples: ['Poseidon\'s Chosen', 'Davy Jones\' Heir', 'Flying Dutchman Captain'],
    features: ['Full transformation', 'Unique idle animations', 'Voice lines']
  }
];
```

### 2. Battle Pass

```typescript
interface BattlePassPricing {
  type: 'Battle Pass';
  currency: 'Doubloons';
}

const BATTLE_PASS_PRODUCTS: BattlePassProduct[] = [
  {
    name: 'Battle Pass',
    doubloonPrice: 950,
    usdEquivalent: 9.99,
    description: 'Unlock premium reward track for current season',
    includes: [
      'Access to premium tier rewards',
      'Exclusive season outfit',
      'Premium emotes and cosmetics',
      '950+ Doubloons in rewards (if completed)'
    ],
    valueProposition: 'Completes the pass = earns back cost + bonus'
  },
  {
    name: 'Battle Pass Bundle',
    doubloonPrice: 1500,
    usdEquivalent: 14.99,
    description: 'Battle Pass + instant tier boosts',
    includes: [
      'Everything in standard Battle Pass',
      '+10 instant tier skips',
      'Exclusive bundle frame'
    ],
    savings: '~25% vs buying separately'
  }
];
```

### 3. Battle Pass Tier Skips

```typescript
interface TierSkipPricing {
  type: 'Consumable';
  name: 'Battle Pass Tier Skip';
  doubloonPrice: 150;
  usdEquivalent: 1.49;
  description: 'Skip one Battle Pass tier instantly';
  bulkDiscounts: TierSkipBulk[];
}

const TIER_SKIP_BUNDLES: TierSkipBulk[] = [
  {
    tiers: 1,
    doubloonPrice: 150,
    pricePerTier: 150
  },
  {
    tiers: 5,
    doubloonPrice: 650,
    pricePerTier: 130,
    savings: '~13%'
  },
  {
    tiers: 10,
    doubloonPrice: 1200,
    pricePerTier: 120,
    savings: '~20%'
  },
  {
    tiers: 25,
    doubloonPrice: 2750,
    pricePerTier: 110,
    savings: '~27%'
  }
];
```

### 4. Premium Emotes

```typescript
interface PremiumEmotePricing {
  category: 'Premium Emote';
  currency: 'Doubloons';
}

const PREMIUM_EMOTES: PremiumEmoteTier[] = [
  {
    tier: 'Rare Premium',
    doubloonPrice: 200,
    description: 'Exclusive animated emotes',
    features: ['Unique animations', 'Not earnable through gameplay']
  },
  {
    tier: 'Epic Premium',
    doubloonPrice: 400,
    description: 'Premium emotes with effects',
    features: ['Particle effects', 'Sound effects', 'Longer animations']
  },
  {
    tier: 'Legendary Premium',
    doubloonPrice: 600,
    description: 'Top-tier collectible emotes',
    features: ['Full scene emotes', 'Interactive elements', 'Voice lines']
  }
];
```

### 5. Exclusive Bundles

```typescript
interface ExclusiveBundlePricing {
  type: 'Bundle';
  currency: 'Doubloons';
}

const BUNDLE_TYPES: BundleConfig[] = [
  {
    bundleType: 'Starter Pack',
    doubloonPrice: 0, // IAP only
    iapPrice: 4.99,
    purchaseLimit: 1,
    contents: [
      { item: 'Exclusive Starter Outfit', value: 800 },
      { item: '500 Doubloons', value: 500 },
      { item: '2000 Gold', value: 200 },
      { item: 'Starter Title', value: 100 }
    ],
    totalValue: 1600,
    savings: '~70%'
  },
  {
    bundleType: 'Themed Set Bundle',
    doubloonPrice: 1500,
    contents: [
      { item: 'Matching Outfit', value: 1200 },
      { item: 'Matching Emote', value: 400 },
      { item: 'Matching Card Frame', value: 200 },
      { item: 'Matching Title', value: 200 }
    ],
    totalValue: 2000,
    savings: '25%'
  },
  {
    bundleType: 'Event Bundle',
    doubloonPrice: 2000,
    timeLimit: 'Event duration only',
    contents: [
      { item: 'Limited Edition Outfit', value: 1500 },
      { item: 'Event Emotes x2', value: 600 },
      { item: 'Event Profile Set', value: 400 },
      { item: 'Bonus Doubloons', value: 200 }
    ],
    totalValue: 2700,
    savings: '~26%'
  },
  {
    bundleType: 'Whale Bundle',
    doubloonPrice: 5000,
    contents: [
      { item: 'Mythic Outfit', value: 2000 },
      { item: 'Legendary Emote Set', value: 1800 },
      { item: 'Premium Profile Package', value: 1000 },
      { item: 'Exclusive Title', value: 500 },
      { item: 'Bonus Doubloons', value: 500 }
    ],
    totalValue: 5800,
    savings: '~14%',
    note: 'Targeting high spenders'
  }
];
```

### 6. Premium Loot Crates

```typescript
interface PremiumCratePricing {
  type: 'Loot Crate';
  currency: 'Doubloons';
}

const PREMIUM_CRATES: PremiumCrateConfig[] = [
  {
    crateType: 'Epic Crate',
    doubloonPrice: 200,
    guaranteedRarity: 'Rare',
    dropRates: {
      rare: 0.60,
      epic: 0.30,
      legendary: 0.10
    },
    noDuplicates: 'Re-roll on owned item'
  },
  {
    crateType: 'Legendary Crate',
    doubloonPrice: 500,
    guaranteedRarity: 'Epic',
    dropRates: {
      epic: 0.65,
      legendary: 0.30,
      mythic: 0.05
    },
    noDuplicates: 'Re-roll on owned item'
  }
];
```

---

## Price Tier Documentation

### Gold Item Price Matrix

| Category | Tier | Price (Gold) | Time to Earn | Notes |
|----------|------|--------------|--------------|-------|
| Outfit | Basic | 800 | 3-4 days | Entry-level customization |
| Outfit | Standard | 1,200 | 5-6 days | Good variety |
| Outfit | Quality | 1,600 | 7-8 days | Detailed designs |
| Outfit | Premium Common | 2,000 | 9-10 days | Best Gold outfits |
| Emote | Basic | 1,000 | 4-5 days | Simple gestures |
| Emote | Standard | 1,500 | 6-7 days | Personality emotes |
| Emote | Complex | 2,000 | 8-9 days | Multi-stage |
| Emote | Epic | 2,500 | 10-11 days | With sound |
| Emote | Legendary | 3,000 | 12-14 days | Full effects |
| Profile | Background | 200-600 | 1-3 days | Card customization |
| Profile | Frame | 300-800 | 1-4 days | Border designs |
| Profile | Title | 250-1,000 | 1-5 days | Display titles |
| Service | Name Change | 500 | 2-3 days | 30-day cooldown |

### Doubloon Item Price Matrix

| Category | Tier | Price (Doubloons) | USD Equivalent | Notes |
|----------|------|-------------------|----------------|-------|
| Outfit | Rare | 500 | $4.99 | Exclusive colors |
| Outfit | Epic | 800 | $7.99 | With accessories |
| Outfit | Legendary | 1,200 | $11.99 | Full VFX |
| Outfit | Mythic | 2,000 | $19.99 | Collector tier |
| Battle Pass | Standard | 950 | $9.99 | Full season |
| Battle Pass | Bundle | 1,500 | $14.99 | +10 tier skips |
| Tier Skip | Single | 150 | $1.49 | Convenience |
| Emote | Rare Premium | 200 | $1.99 | Exclusive |
| Emote | Epic Premium | 400 | $3.99 | With effects |
| Emote | Legendary Premium | 600 | $5.99 | Full scene |
| Bundle | Themed | 1,500 | $14.99 | 25% savings |
| Bundle | Event | 2,000 | $19.99 | Limited time |

---

## Conversion Rate Analysis

### Doubloon to USD Ratio

```typescript
interface ConversionAnalysis {
  baseRate: '100 Doubloons = $0.99';
  implicitRate: '$0.0099 per Doubloon';

  purchasePackRates: ConversionRate[];
}

const CONVERSION_RATES: ConversionRate[] = [
  {
    pack: 'Small (100)',
    price: 0.99,
    doubloons: 100,
    ratePerDoubloon: 0.0099,
    bonus: '0%'
  },
  {
    pack: 'Medium (550)',
    price: 4.99,
    doubloons: 550,
    ratePerDoubloon: 0.0091,
    bonus: '10%'
  },
  {
    pack: 'Large (1200)',
    price: 9.99,
    doubloons: 1200,
    ratePerDoubloon: 0.0083,
    bonus: '20%'
  },
  {
    pack: 'Mega (2500)',
    price: 19.99,
    doubloons: 2500,
    ratePerDoubloon: 0.0080,
    bonus: '25%'
  },
  {
    pack: 'Ultra (6500)',
    price: 49.99,
    doubloons: 6500,
    ratePerDoubloon: 0.0077,
    bonus: '30%'
  }
];
```

### Price Consistency Validation

```typescript
interface PriceConsistencyCheck {
  validate(): ValidationResult;
}

const PRICE_VALIDATION_RULES: ValidationRule[] = [
  {
    rule: 'Outfit Price Ratio',
    check: 'Premium outfit tiers maintain ~2x Gold equivalent',
    example: 'Epic Outfit (800 Doubloons) ≈ Premium Common (2000 Gold) in perceived value'
  },
  {
    rule: 'Battle Pass Value',
    check: 'Battle Pass completion returns 100%+ investment',
    validation: '950 Doubloons spent → 1000+ Doubloons in rewards'
  },
  {
    rule: 'Bundle Savings',
    check: 'All bundles offer 15-30% savings over individual purchase',
    minimumSavings: 0.15,
    maximumSavings: 0.35
  },
  {
    rule: 'Time-to-Earn Fairness',
    check: 'F2P player can purchase meaningful item within 1 week',
    target: 'At least one outfit tier < 7 days of casual play'
  }
];
```

---

## Gold Accessibility Analysis

### Play Time to Purchase

```typescript
interface AccessibilityAnalysis {
  playerType: 'Casual' | 'Regular' | 'Hardcore';
  dailyPlayTime: string;
  dailyGoldEarned: number;
  weeklyGoldEarned: number;
}

const PLAYER_ACCESSIBILITY: AccessibilityAnalysis[] = [
  {
    playerType: 'Casual',
    dailyPlayTime: '30 minutes (2-3 matches)',
    dailyGoldEarned: 200,
    weeklyGoldEarned: 1400,
    purchasablePerWeek: [
      'Basic Outfit (800 Gold) + Profile Item',
      'Standard Emote (1500 Gold)',
      'Multiple Profile Items'
    ]
  },
  {
    playerType: 'Regular',
    dailyPlayTime: '1 hour (4-6 matches)',
    dailyGoldEarned: 350,
    weeklyGoldEarned: 2450,
    purchasablePerWeek: [
      'Quality Outfit (1600 Gold)',
      'Complex Emote (2000 Gold)',
      'Premium Common Outfit (2000 Gold)'
    ]
  },
  {
    playerType: 'Hardcore',
    dailyPlayTime: '2+ hours (8+ matches)',
    dailyGoldEarned: 500,
    weeklyGoldEarned: 3500,
    purchasablePerWeek: [
      'Any Gold item',
      'Multiple mid-tier items',
      'Save for expensive items in 2 weeks'
    ]
  }
];
```

### Weekly Quest Impact

```typescript
const WEEKLY_BOOST: WeeklyImpact = {
  questCompletion: {
    dailyQuests: '3 per day × 150 avg = 450 Gold/day',
    weeklyQuests: '3 per week × 750 avg = 2250 Gold/week',
    totalBoost: '+450 daily + 2250 weekly = ~5400 Gold/week potential'
  },
  impact: 'Regular player with quests: 2450 + 5400 = ~7850 Gold/week',
  analysis: 'Quest completion significantly accelerates Gold earning'
};
```

---

## Premium Value Perception

### Value Anchoring Strategy

```typescript
interface ValuePerception {
  strategy: 'Anchor premium items against Gold equivalents';
  implementation: ValueAnchorRule[];
}

const VALUE_ANCHORING: ValueAnchorRule[] = [
  {
    principle: 'Premium = Exclusive, Not Just Better',
    implementation: 'Premium items have unique VFX/designs, not just "better" versions',
    example: 'Epic Premium Outfit has ghostly effects, not just higher resolution'
  },
  {
    principle: 'FOMO Without P2W',
    implementation: 'Limited-time items create urgency without gameplay advantage',
    example: 'Event outfits only available during events, purely cosmetic'
  },
  {
    principle: 'Visible Prestige',
    implementation: 'Premium items should be recognizable in-game',
    example: 'Mythic outfits have unique silhouettes and effects visible to all players'
  },
  {
    principle: 'Fair Free Option',
    implementation: 'Gold items never feel like "poor version"',
    example: 'Gold outfits have quality designs, just different theme/style'
  }
];
```

### Whale vs. Minnow Balance

```typescript
interface SpenderSegmentation {
  segment: 'Non-Spender' | 'Minnow' | 'Dolphin' | 'Whale';
  monthlySpend: string;
  targetedSinks: string[];
}

const SPENDER_SEGMENTS: SpenderSegmentation[] = [
  {
    segment: 'Non-Spender',
    monthlySpend: '$0',
    targetedSinks: [
      'Gold cosmetics',
      'Free Battle Pass track',
      'Profile customization',
      'Name changes'
    ],
    engagement: 'Complete gameplay loop available'
  },
  {
    segment: 'Minnow',
    monthlySpend: '$1-10',
    targetedSinks: [
      'Starter Pack (one-time)',
      'Battle Pass',
      'Occasional Doubloon purchases'
    ],
    engagement: 'Best value purchases'
  },
  {
    segment: 'Dolphin',
    monthlySpend: '$10-50',
    targetedSinks: [
      'Battle Pass + Bundle',
      'Premium outfits',
      'Tier skips',
      'Themed bundles'
    ],
    engagement: 'Regular premium content'
  },
  {
    segment: 'Whale',
    monthlySpend: '$50+',
    targetedSinks: [
      'All premium content',
      'Mythic outfits',
      'Whale bundles',
      'Complete collections',
      'Instant pass completion'
    ],
    engagement: 'Collector completionism'
  }
];
```

---

## Shop Display Integration

### Price Display Rules

```typescript
interface PriceDisplayConfig {
  display: PriceDisplayRule[];
}

const PRICE_DISPLAY_RULES: PriceDisplayRule[] = [
  {
    rule: 'Currency Icon First',
    display: '[Gold Icon] 1,200 or [Doubloon Icon] 800',
    rationale: 'Instant currency recognition'
  },
  {
    rule: 'Original Price on Sales',
    display: 'Strikethrough original + new price',
    example: '[Gold] ~~1,600~~ 1,200'
  },
  {
    rule: 'Bundle Value Display',
    display: 'Show total value + savings percentage',
    example: 'Value: 2,000 | Your Price: 1,500 (25% OFF)'
  },
  {
    rule: 'Owned Item Indicator',
    display: 'Replace price with "OWNED" badge',
    styling: 'Green checkmark, greyed purchase button'
  },
  {
    rule: 'Insufficient Funds',
    display: 'Red price text + "Need X more" helper',
    action: 'Tap opens currency purchase flow'
  }
];
```

### Purchase Flow

```typescript
interface PurchaseFlow {
  steps: PurchaseStep[];
}

const PURCHASE_FLOW: PurchaseStep[] = [
  {
    step: 1,
    action: 'Item Selection',
    ui: 'Tap item in shop grid',
    display: 'Item preview modal with 3D viewer'
  },
  {
    step: 2,
    action: 'Purchase Button',
    ui: 'Large "Purchase for [X Gold/Doubloons]" button',
    validation: 'Check balance before enabling'
  },
  {
    step: 3,
    action: 'Confirmation Dialog',
    ui: 'Modal confirming purchase',
    content: '"Purchase [Item Name] for [Price]?"',
    buttons: ['Cancel', 'Confirm']
  },
  {
    step: 4,
    action: 'Transaction Processing',
    ui: 'Brief loading spinner',
    backend: 'Server-side balance deduction'
  },
  {
    step: 5,
    action: 'Success Feedback',
    ui: 'Celebration animation + "Item Added!"',
    options: ['Equip Now', 'Continue Shopping']
  }
];
```

---

## Economy Health Metrics

### Monitoring Targets

```typescript
interface EconomyMetrics {
  metric: string;
  healthyRange: string;
  alertThreshold: string;
}

const ECONOMY_HEALTH_METRICS: EconomyMetrics[] = [
  {
    metric: 'Daily Gold Spend Rate',
    healthyRange: '60-80% of daily earnings',
    alertThreshold: '<50% or >90%'
  },
  {
    metric: 'Gold Hoarding Rate',
    healthyRange: '<20% of players with >10,000 Gold',
    alertThreshold: '>30% hoarding'
  },
  {
    metric: 'Doubloon Velocity',
    healthyRange: '50% spent within 7 days of purchase',
    alertThreshold: '<40% velocity'
  },
  {
    metric: 'Shop Item Diversity',
    healthyRange: 'No single item >15% of purchases',
    alertThreshold: 'Single item >25%'
  },
  {
    metric: 'Price Point Engagement',
    healthyRange: 'All tiers have purchases',
    alertThreshold: 'Any tier <5% of segment'
  }
];
```

---

## Implementation Checklist

### Gold Sinks
- [ ] Common outfit price tiers (800-2000)
- [ ] Emote price tiers (1000-3000)
- [ ] Profile customization prices
- [ ] Name change service (500 Gold)
- [ ] Optional: Loot crate keys

### Doubloon Sinks
- [ ] Premium outfit tiers (500-2000)
- [ ] Battle Pass pricing (950)
- [ ] Battle Pass Bundle (1500)
- [ ] Tier skip pricing (150 per tier)
- [ ] Premium emote tiers
- [ ] Bundle system with savings display
- [ ] Starter Pack (IAP only)

### Shop Integration
- [ ] Price display with currency icons
- [ ] Owned item indicators
- [ ] Insufficient funds helper
- [ ] Purchase confirmation flow
- [ ] Bundle value display

### Economy Monitoring
- [ ] Spend rate tracking
- [ ] Hoarding detection
- [ ] Price tier engagement analytics
- [ ] Alert system for anomalies

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
