# Bundle System Specification

## Overview

This document defines the bundle system for Plunderstorm Mobile, enabling value packs that combine multiple items at discounted prices, driving higher average order values while providing compelling offers to players.

## Bundle Philosophy

### Design Goals
- **Value Perception**: Bundles must feel like genuine deals
- **Flexibility**: Support various bundle types for different purposes
- **Fairness**: Handle partial ownership gracefully
- **Conversion**: Convert browsers to buyers with compelling offers
- **Retention**: Use bundles for progression milestones and events

### Bundle Categories

```typescript
enum BundleCategory {
  STARTER = 'starter',       // One-time new player offers
  THEMED = 'themed',         // Matching cosmetic sets
  EVENT = 'event',           // Limited-time event bundles
  VALUE = 'value',           // Currency packs with bonuses
  PROGRESSION = 'progression', // Level/milestone rewards
  FLASH = 'flash'            // Short-term promotional bundles
}
```

---

## Bundle Type Definitions

### Starter Pack

```typescript
interface StarterPack {
  bundleId: 'starter_pack_v1';
  category: BundleCategory.STARTER;
  name: 'Starter Pack';
  description: 'The perfect beginning for your pirate adventure!';

  purchaseLimit: 1;
  eligibility: {
    accountAge: 'Any';
    purchaseHistory: 'First-time buyer or never purchased';
    displayUntil: '30 days after account creation';
  };

  pricing: {
    iapOnly: true;
    price: 4.99; // USD
    currency: null; // Not purchasable with in-game currency
  };

  contents: StarterPackContent[];
  totalValue: number;
  savings: string;
}

const STARTER_PACK_CONTENTS: StarterPackContent[] = [
  {
    item: 'Doubloons',
    amount: 500,
    individualValue: 4.99,
    description: 'Premium currency to spend as you wish'
  },
  {
    item: 'Adventurer Outfit',
    type: 'Cosmetic',
    rarity: 'Rare',
    exclusive: true,
    individualValue: 5.00,
    description: 'Exclusive outfit only available in Starter Pack'
  },
  {
    item: 'Gold',
    amount: 2000,
    individualValue: 2.00,
    description: 'Soft currency for common purchases'
  },
  {
    item: 'Adventurer Title',
    type: 'Profile',
    exclusive: true,
    individualValue: 1.00,
    description: 'Exclusive title badge'
  }
];

const STARTER_PACK_VALUE = {
  totalIndividualValue: 12.99,
  bundlePrice: 4.99,
  savings: '62%',
  displayText: 'Value: $12.99 | Price: $4.99 (Save 62%)'
};
```

### Themed Bundles

```typescript
interface ThemedBundle {
  bundleId: string;
  category: BundleCategory.THEMED;
  name: string;
  theme: string;

  pricing: {
    doubloonPrice: number;
    goldPrice?: number; // Some bundles Gold-purchasable
  };

  contents: ThemedBundleContent[];
  setBonus?: SetBonus;
  totalValue: number;
  savingsPercent: number;
}

const THEMED_BUNDLE_EXAMPLES: ThemedBundle[] = [
  {
    bundleId: 'ghost_pirate_set',
    category: BundleCategory.THEMED,
    name: 'Ghost Pirate Collection',
    theme: 'Ghostly/Ethereal',

    pricing: {
      doubloonPrice: 1500
    },

    contents: [
      {
        item: 'Ghost Captain Outfit',
        type: 'Outfit',
        rarity: 'Epic',
        individualPrice: 800,
        currency: 'Doubloons'
      },
      {
        item: 'Spectral Wave Emote',
        type: 'Emote',
        rarity: 'Rare',
        individualPrice: 400,
        currency: 'Doubloons'
      },
      {
        item: 'Haunted Frame',
        type: 'Profile',
        rarity: 'Rare',
        individualPrice: 200,
        currency: 'Doubloons'
      },
      {
        item: 'Phantom Title',
        type: 'Profile',
        rarity: 'Rare',
        individualPrice: 150,
        currency: 'Doubloons'
      }
    ],

    setBonus: {
      description: 'Equipping all items grants ghostly particle effect',
      effect: 'ghost_particle_trail'
    },

    totalValue: 1550,
    savingsPercent: 3 // Minimal savings but set bonus value
  },
  {
    bundleId: 'royal_navy_set',
    category: BundleCategory.THEMED,
    name: 'Royal Navy Admiral Set',
    theme: 'Naval/Military',

    pricing: {
      doubloonPrice: 2200
    },

    contents: [
      {
        item: 'Admiral Uniform',
        type: 'Outfit',
        rarity: 'Legendary',
        individualPrice: 1200,
        currency: 'Doubloons'
      },
      {
        item: 'Command Salute Emote',
        type: 'Emote',
        rarity: 'Epic',
        individualPrice: 500,
        currency: 'Doubloons'
      },
      {
        item: 'Navy Crest Frame',
        type: 'Profile',
        rarity: 'Epic',
        individualPrice: 400,
        currency: 'Doubloons'
      },
      {
        item: 'Admiral Title',
        type: 'Profile',
        rarity: 'Epic',
        individualPrice: 300,
        currency: 'Doubloons'
      }
    ],

    totalValue: 2400,
    savingsPercent: 8
  }
];
```

### Event Bundles

```typescript
interface EventBundle {
  bundleId: string;
  category: BundleCategory.EVENT;
  eventId: string;
  name: string;

  availability: {
    startDate: Date;
    endDate: Date;
    displayCountdown: true;
  };

  exclusivity: {
    willReturn: boolean;
    returnPolicy: string;
  };

  pricing: {
    doubloonPrice: number;
  };

  contents: EventBundleContent[];
  totalValue: number;
  savingsPercent: number;
}

const EVENT_BUNDLE_EXAMPLE: EventBundle = {
  bundleId: 'halloween_2024_mega',
  category: BundleCategory.EVENT,
  eventId: 'halloween_2024',
  name: 'Halloween Mega Bundle',

  availability: {
    startDate: new Date('2024-10-25'),
    endDate: new Date('2024-11-01'),
    displayCountdown: true
  },

  exclusivity: {
    willReturn: true,
    returnPolicy: 'Available during Halloween event each year'
  },

  pricing: {
    doubloonPrice: 2500
  },

  contents: [
    {
      item: 'Pumpkin Pirate Outfit',
      type: 'Outfit',
      rarity: 'Legendary',
      exclusive: true,
      individualPrice: 1500
    },
    {
      item: 'Spooky Dance Emote',
      type: 'Emote',
      rarity: 'Epic',
      exclusive: true,
      individualPrice: 500
    },
    {
      item: 'Haunted Ship Trail',
      type: 'Effect',
      rarity: 'Epic',
      exclusive: true,
      individualPrice: 400
    },
    {
      item: 'Jack-O-Lantern Frame',
      type: 'Profile',
      exclusive: true,
      individualPrice: 200
    },
    {
      item: 'Trick or Treat Title',
      type: 'Profile',
      exclusive: true,
      individualPrice: 150
    },
    {
      item: 'Bonus Doubloons',
      type: 'Currency',
      amount: 300,
      individualPrice: 300
    }
  ],

  totalValue: 3050,
  savingsPercent: 18
};
```

### Value Bundles

```typescript
interface ValueBundle {
  bundleId: string;
  category: BundleCategory.VALUE;
  name: string;

  pricing: {
    iapOnly: boolean;
    price?: number; // USD for IAP
    doubloonPrice?: number; // For in-game purchase
  };

  contents: ValueBundleContent[];
  bonusPercentage: number;
  targetAudience: string;
}

const VALUE_BUNDLES: ValueBundle[] = [
  {
    bundleId: 'doubloon_value_small',
    category: BundleCategory.VALUE,
    name: 'Doubloon Stash',

    pricing: {
      iapOnly: true,
      price: 9.99
    },

    contents: [
      {
        item: 'Doubloons',
        amount: 1000,
        baseValue: 9.99
      },
      {
        item: 'Bonus Doubloons',
        amount: 200,
        bonusValue: 1.99
      },
      {
        item: 'Random Epic Item',
        type: 'Loot',
        guaranteedRarity: 'Epic',
        bonusValue: 3.00
      }
    ],

    bonusPercentage: 50,
    targetAudience: 'Minnows/Dolphins'
  },
  {
    bundleId: 'premium_chest',
    category: BundleCategory.VALUE,
    name: 'Premium Treasure Chest',

    pricing: {
      iapOnly: true,
      price: 19.99
    },

    contents: [
      {
        item: 'Doubloons',
        amount: 2500,
        baseValue: 19.99
      },
      {
        item: 'Random Legendary Item',
        type: 'Loot',
        guaranteedRarity: 'Legendary'
      },
      {
        item: '5 Epic Crates',
        type: 'Loot'
      },
      {
        item: 'Exclusive Treasure Hunter Frame',
        type: 'Profile',
        exclusive: true
      }
    ],

    bonusPercentage: 75,
    targetAudience: 'Dolphins/Whales'
  }
];
```

### Progression Bundles

```typescript
interface ProgressionBundle {
  bundleId: string;
  category: BundleCategory.PROGRESSION;
  name: string;

  unlockCondition: {
    type: 'level' | 'wins' | 'playtime' | 'achievement';
    requirement: number | string;
  };

  availability: {
    displayDuration: number; // Hours after unlock
    oneTimePurchase: true;
  };

  pricing: {
    doubloonPrice?: number;
    price?: number; // USD for special milestones
  };

  contents: ProgressionContent[];
  personalizedText: string;
}

const PROGRESSION_BUNDLES: ProgressionBundle[] = [
  {
    bundleId: 'level_10_bundle',
    category: BundleCategory.PROGRESSION,
    name: 'Level 10 Milestone',

    unlockCondition: {
      type: 'level',
      requirement: 10
    },

    availability: {
      displayDuration: 72, // 3 days
      oneTimePurchase: true
    },

    pricing: {
      doubloonPrice: 800
    },

    contents: [
      {
        item: 'Seasoned Sailor Outfit',
        type: 'Outfit',
        rarity: 'Rare'
      },
      {
        item: 'Level 10 Badge Frame',
        type: 'Profile'
      },
      {
        item: '1000 Gold',
        type: 'Currency'
      }
    ],

    personalizedText: 'Congratulations on reaching Level 10!'
  },
  {
    bundleId: 'first_win_bundle',
    category: BundleCategory.PROGRESSION,
    name: 'First Victory Bundle',

    unlockCondition: {
      type: 'wins',
      requirement: 1
    },

    availability: {
      displayDuration: 48,
      oneTimePurchase: true
    },

    pricing: {
      price: 2.99
    },

    contents: [
      {
        item: '300 Doubloons',
        type: 'Currency'
      },
      {
        item: 'Champion Emote',
        type: 'Emote',
        exclusive: true
      },
      {
        item: 'Victor Title',
        type: 'Profile'
      }
    ],

    personalizedText: 'Celebrate your first victory!'
  }
];
```

---

## Value Display System

### Value Calculation

```typescript
interface ValueCalculation {
  calculate(bundle: Bundle, ownedItems: string[]): ValueResult;
}

const VALUE_CALCULATION: ValueCalculation = {
  calculate: (bundle, ownedItems) => {
    const totalValue = bundle.contents.reduce((sum, item) => {
      return sum + item.individualPrice;
    }, 0);

    const ownedValue = bundle.contents
      .filter(item => ownedItems.includes(item.itemId))
      .reduce((sum, item) => sum + item.individualPrice, 0);

    const adjustedValue = totalValue - ownedValue;
    const originalPrice = bundle.pricing.doubloonPrice || bundle.pricing.price;
    const adjustedPrice = calculateAdjustedPrice(originalPrice, ownedValue, totalValue);

    const savingsAmount = adjustedValue - adjustedPrice;
    const savingsPercent = Math.round((savingsAmount / adjustedValue) * 100);

    return {
      totalValue,
      ownedValue,
      adjustedValue,
      originalPrice,
      adjustedPrice,
      savingsAmount,
      savingsPercent,
      displayText: formatValueDisplay(adjustedValue, adjustedPrice, savingsPercent)
    };
  }
};
```

### Value Display Component

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                 📦 GHOST PIRATE COLLECTION 📦               │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │[Outfit] │  │ [Emote] │  │ [Frame] │  │ [Title] │       │
│  │ Ghost   │  │Spectral │  │ Haunted │  │ Phantom │       │
│  │ Captain │  │  Wave   │  │  Frame  │  │  Title  │       │
│  │ 💎 800  │  │ 💎 400  │  │ 💎 200  │  │ 💎 150  │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
│                                                             │
│  ────────────────────────────────────────────────────────  │
│                                                             │
│    Individual Total:          💎 1,550                     │
│    Bundle Price:              💎 1,500                     │
│                               ─────────                     │
│    You Save:                  💎 50 (3%)                   │
│                                                             │
│    ✨ SET BONUS: Equip all for ghostly particle effect!    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            PURCHASE BUNDLE - 💎 1,500               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Partial Ownership Display

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                 📦 GHOST PIRATE COLLECTION 📦               │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │[Outfit] │  │ [Emote] │  │ [Frame] │  │ [Title] │       │
│  │ Ghost   │  │Spectral │  │ Haunted │  │ Phantom │       │
│  │ Captain │  │  Wave   │  │  Frame  │  │  Title  │       │
│  │ OWNED ✓ │  │ 💎 400  │  │ 💎 200  │  │ 💎 150  │       │
│  │ ─────── │  │         │  │         │  │         │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
│                                                             │
│  ────────────────────────────────────────────────────────  │
│                                                             │
│    Items You'll Get:          3 of 4                       │
│    Value of New Items:        💎 750                       │
│    Adjusted Bundle Price:     💎 725                       │
│                               ─────────                     │
│    You Save:                  💎 25 (3%)                   │
│                                                             │
│    ℹ️ Price adjusted because you own Ghost Captain         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          COMPLETE COLLECTION - 💎 725               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Price Adjustment System

### Adjustment Algorithm

```typescript
interface PriceAdjustment {
  policy: AdjustmentPolicy;
  calculate(bundle: Bundle, ownedItems: Item[]): AdjustedPrice;
}

const PRICE_ADJUSTMENT: PriceAdjustment = {
  policy: {
    enabled: true,
    minimumItemsForAdjustment: 1,
    adjustmentMethod: 'proportional',
    minimumDiscount: 0.03, // Always maintain at least 3% discount
    roundingRule: 'floor_to_nearest_25'
  },

  calculate: (bundle, ownedItems) => {
    const totalItems = bundle.contents.length;
    const ownedCount = ownedItems.length;
    const remainingItems = totalItems - ownedCount;

    if (remainingItems === 0) {
      return { purchasable: false, reason: 'ALL_ITEMS_OWNED' };
    }

    // Calculate value of remaining items
    const remainingValue = bundle.contents
      .filter(item => !ownedItems.includes(item))
      .reduce((sum, item) => sum + item.individualPrice, 0);

    // Calculate proportional reduction
    const originalPrice = bundle.pricing.doubloonPrice;
    const ownedValue = bundle.totalValue - remainingValue;
    const reductionRatio = ownedValue / bundle.totalValue;
    const reduction = originalPrice * reductionRatio;

    // Apply adjustment maintaining minimum discount
    let adjustedPrice = originalPrice - reduction;
    const minPriceForDiscount = remainingValue * (1 - 0.03);
    adjustedPrice = Math.min(adjustedPrice, minPriceForDiscount);

    // Round to nearest 25
    adjustedPrice = Math.floor(adjustedPrice / 25) * 25;

    return {
      purchasable: true,
      originalPrice,
      adjustedPrice,
      savings: remainingValue - adjustedPrice,
      savingsPercent: Math.round((1 - adjustedPrice / remainingValue) * 100),
      ownedItemsCount: ownedCount,
      remainingItemsCount: remainingItems
    };
  }
};
```

### Adjustment Examples

```typescript
const ADJUSTMENT_EXAMPLES: AdjustmentExample[] = [
  {
    scenario: 'Own 1 of 4 items (Epic Outfit worth 800)',
    bundleValue: 1550,
    bundlePrice: 1500,
    ownedValue: 800,
    remainingValue: 750,
    calculation: {
      reductionRatio: 800 / 1550, // 0.516
      priceReduction: 1500 * 0.516, // 774
      rawAdjusted: 1500 - 774, // 726
      roundedAdjusted: 725 // Floor to nearest 25
    },
    result: {
      adjustedPrice: 725,
      savings: 750 - 725, // 25
      savingsPercent: 3
    }
  },
  {
    scenario: 'Own 2 of 4 items (Outfit + Emote worth 1200)',
    bundleValue: 1550,
    bundlePrice: 1500,
    ownedValue: 1200,
    remainingValue: 350,
    calculation: {
      reductionRatio: 1200 / 1550,
      priceReduction: 1500 * 0.774,
      rawAdjusted: 339,
      roundedAdjusted: 325
    },
    result: {
      adjustedPrice: 325,
      savings: 25,
      savingsPercent: 7
    }
  },
  {
    scenario: 'Own 3 of 4 items (only Title remaining)',
    bundleValue: 1550,
    bundlePrice: 1500,
    ownedValue: 1400,
    remainingValue: 150,
    result: {
      adjustedPrice: 125, // Slightly below individual price
      recommendation: 'Consider buying individual item instead',
      showIndividualOption: true
    }
  }
];
```

---

## Bundle Preview System

### Preview Modal

```typescript
interface BundlePreview {
  layout: PreviewLayout;
  interactions: PreviewInteraction[];
}

const BUNDLE_PREVIEW: BundlePreview = {
  layout: {
    header: {
      bundleName: true,
      bundleCategory: true,
      countdown: 'If time-limited'
    },
    contentGrid: {
      columns: 4,
      itemDisplay: {
        thumbnail: true,
        name: true,
        rarity: true,
        individualPrice: true,
        ownedBadge: true
      }
    },
    valueSection: {
      totalValue: true,
      bundlePrice: true,
      savings: true,
      adjustmentNote: 'If owns any items'
    },
    setBonus: {
      display: 'If applicable',
      preview: 'Animated preview of bonus'
    },
    actions: {
      primaryButton: 'Purchase',
      secondaryButton: 'View Individual Items'
    }
  },
  interactions: [
    {
      action: 'Tap item in grid',
      result: 'Open individual item preview'
    },
    {
      action: 'Swipe horizontally',
      result: 'Cycle through item previews'
    },
    {
      action: 'Tap set bonus area',
      result: 'Play set bonus preview animation'
    }
  ]
};
```

### Item Detail View Within Bundle

```
┌─────────────────────────────────────────────────────────────┐
│  [< Back to Bundle]                                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌─────────────────┐                      │
│                    │                 │                      │
│                    │   3D CHARACTER  │                      │
│                    │     MODEL       │                      │
│                    │                 │                      │
│                    │   [Rotatable]   │                      │
│                    │                 │                      │
│                    └─────────────────┘                      │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ⭐ EPIC                                                    │
│  Ghost Captain Outfit                                       │
│                                                             │
│  "Command the spirit realm with this ethereal captain's     │
│   attire, glowing with otherworldly energy."                │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Individual Price:   💎 800                               │
│    In Bundle:          💎 ~386 (calculated share)          │
│                        ─────────                            │
│    Bundle Savings:     💎 414 per item                     │
│                                                             │
│  ┌────────────────────┐  ┌────────────────────┐            │
│  │    BUY SINGLE      │  │   BUY IN BUNDLE    │            │
│  │      💎 800        │  │      💎 1,500      │            │
│  └────────────────────┘  └────────────────────┘            │
│                                                             │
│  [◀ Prev Item]              [Next Item ▶]                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Purchase Limit System

### Limit Types

```typescript
interface PurchaseLimits {
  limitTypes: LimitType[];
}

const PURCHASE_LIMITS: PurchaseLimits = {
  limitTypes: [
    {
      type: 'ONE_TIME',
      description: 'Can only be purchased once per account',
      applies: ['Starter Pack', 'Progression Bundles'],
      enforcement: 'Server-side, permanent flag on account'
    },
    {
      type: 'PER_EVENT',
      description: 'Once per event occurrence',
      applies: ['Event Bundles'],
      enforcement: 'Reset when event returns',
      example: 'Halloween bundle once per Halloween event'
    },
    {
      type: 'QUANTITY_LIMITED',
      description: 'Limited stock globally',
      applies: ['Exclusive collaboration bundles'],
      enforcement: 'Global counter, first-come-first-served',
      display: 'Show remaining quantity'
    },
    {
      type: 'DAILY_LIMIT',
      description: 'Can purchase X per day',
      applies: ['Value bundles', 'Currency packs'],
      enforcement: 'Resets at 00:00 UTC'
    }
  ]
};
```

### Starter Pack Enforcement

```typescript
interface StarterPackEnforcement {
  rules: EnforcementRule[];
}

const STARTER_PACK_RULES: StarterPackEnforcement = {
  rules: [
    {
      rule: 'Account Flag',
      implementation: 'has_purchased_starter_pack: boolean',
      persistence: 'Permanent, never resets'
    },
    {
      rule: 'Display Control',
      implementation: 'Hide from store if already purchased',
      fallback: 'Show "Purchased" badge if accessed via deep link'
    },
    {
      rule: 'Purchase Validation',
      implementation: 'Server rejects if flag is true',
      errorMessage: 'You have already purchased the Starter Pack'
    },
    {
      rule: 'Refund Handling',
      implementation: 'Flag remains true even after refund',
      rationale: 'Prevent abuse through purchase-refund cycles'
    }
  ]
};
```

---

## Time-Limited Bundle System

### Countdown Implementation

```typescript
interface TimeLimitedBundle {
  countdown: CountdownConfig;
  expiration: ExpirationBehavior;
}

const TIME_LIMITED_CONFIG: TimeLimitedBundle = {
  countdown: {
    display: {
      position: 'Top of bundle card',
      format: {
        daysPlus: '⏰ {X}d {Y}h remaining',
        hoursOnly: '⏰ {X}h {Y}m remaining',
        urgentMinutes: '🔥 {X}m {Y}s - ENDING SOON!'
      },
      urgencyThreshold: '2 hours',
      urgencyStyling: {
        textColor: 'critical_red',
        animation: 'pulse',
        badge: 'LAST CHANCE'
      }
    },
    behavior: {
      realTimeUpdate: true,
      updateFrequency: '1 second when < 1 hour',
      timezoneHandling: 'UTC, converted to local for display'
    }
  },
  expiration: {
    onExpire: {
      action: 'Remove from store immediately',
      midPurchase: 'Complete if payment initiated',
      notification: 'Push notification 2 hours before if wishlisted'
    },
    afterExpire: {
      display: 'Bundle no longer available',
      returnPolicy: 'Check event schedule for return'
    }
  }
};
```

### Expiration Warning Flow

```typescript
const EXPIRATION_WARNINGS: ExpirationWarning[] = [
  {
    timeRemaining: '24 hours',
    action: 'Add "ENDING SOON" badge',
    notification: false
  },
  {
    timeRemaining: '6 hours',
    action: 'Yellow urgency styling',
    notification: 'If bundle in wishlist'
  },
  {
    timeRemaining: '2 hours',
    action: 'Red urgency styling + pulse',
    notification: 'Push notification to all who viewed'
  },
  {
    timeRemaining: '30 minutes',
    action: 'Critical styling + "LAST CHANCE"',
    notification: 'Final reminder push'
  }
];
```

---

## Bundle Analytics

### Tracking Metrics

```typescript
interface BundleAnalytics {
  metrics: AnalyticsMetric[];
}

const BUNDLE_METRICS: BundleAnalytics = {
  metrics: [
    {
      metric: 'Bundle View Rate',
      description: 'Percentage of store visitors who view bundle details',
      calculation: 'Bundle previews / store sessions',
      target: '>40%'
    },
    {
      metric: 'Bundle Conversion Rate',
      description: 'Purchases per bundle view',
      calculation: 'Bundle purchases / bundle previews',
      target: '>8%'
    },
    {
      metric: 'Average Bundle Revenue',
      description: 'Revenue per bundle sold',
      segmentBy: ['bundle_category', 'bundle_id']
    },
    {
      metric: 'Price Adjustment Impact',
      description: 'Effect of partial ownership on conversion',
      analysis: 'Compare conversion: full price vs adjusted'
    },
    {
      metric: 'Starter Pack Conversion',
      description: 'New players who purchase starter pack',
      calculation: 'Starter purchases / new accounts',
      target: '>5%'
    },
    {
      metric: 'Urgency Effectiveness',
      description: 'Purchases in final hours vs earlier',
      insight: 'FOMO timing optimization'
    },
    {
      metric: 'Individual vs Bundle Choice',
      description: 'When both available, which do players choose',
      insight: 'Bundle value perception'
    }
  ]
};
```

---

## Bundle Creation Workflow

### Admin Bundle Creation

```typescript
interface BundleCreationWorkflow {
  steps: CreationStep[];
}

const BUNDLE_CREATION: BundleCreationWorkflow = {
  steps: [
    {
      step: 1,
      name: 'Basic Info',
      fields: [
        { field: 'bundleId', type: 'auto-generated' },
        { field: 'name', type: 'text', required: true },
        { field: 'category', type: 'enum', options: BundleCategory },
        { field: 'description', type: 'textarea' }
      ]
    },
    {
      step: 2,
      name: 'Content Selection',
      actions: [
        'Search/browse item catalog',
        'Add items to bundle',
        'Set individual values for each item',
        'Define set bonus (optional)'
      ]
    },
    {
      step: 3,
      name: 'Pricing',
      fields: [
        { field: 'priceType', type: 'enum', options: ['Doubloons', 'USD', 'Both'] },
        { field: 'price', type: 'number' },
        { field: 'autoCalculateSavings', type: 'boolean' }
      ],
      validation: 'Savings must be between 5-50%'
    },
    {
      step: 4,
      name: 'Availability',
      fields: [
        { field: 'startDate', type: 'datetime' },
        { field: 'endDate', type: 'datetime', optional: true },
        { field: 'purchaseLimit', type: 'enum', options: LimitTypes },
        { field: 'eligibility', type: 'rules' }
      ]
    },
    {
      step: 5,
      name: 'Preview & Publish',
      actions: [
        'Preview bundle as player would see',
        'Test partial ownership scenarios',
        'Schedule or publish immediately'
      ]
    }
  ]
};
```

---

## Implementation Checklist

### Bundle Types
- [ ] Starter Pack with one-time purchase limit
- [ ] Themed bundles with set bonuses
- [ ] Event bundles with time limits
- [ ] Value bundles with bonus items
- [ ] Progression milestone bundles

### Value Display
- [ ] Total value calculation
- [ ] Bundle price display
- [ ] Savings percentage display
- [ ] Individual item prices in grid

### Price Adjustment
- [ ] Detect owned items in bundle
- [ ] Calculate proportional adjustment
- [ ] Display adjusted price
- [ ] Show adjustment explanation

### Purchase Limits
- [ ] One-time purchase enforcement
- [ ] Per-event limits
- [ ] Purchased badge display
- [ ] Server-side validation

### Time-Limited Features
- [ ] Countdown timer display
- [ ] Urgency styling transitions
- [ ] Expiration handling
- [ ] Warning notifications

### Preview System
- [ ] Bundle content grid
- [ ] Individual item preview
- [ ] Set bonus preview
- [ ] Navigation between items

### Analytics
- [ ] View tracking
- [ ] Conversion tracking
- [ ] Adjustment impact analysis
- [ ] Urgency effectiveness

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
