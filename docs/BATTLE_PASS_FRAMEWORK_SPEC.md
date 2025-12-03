# Battle Pass Framework Specification

## Overview

This document defines the complete Battle Pass system for Plunderstorm Mobile, providing a seasonal progression framework with free and premium reward tracks that drive engagement and monetization.

## Battle Pass Philosophy

### Design Goals
- **Engagement**: Give players meaningful goals every session
- **Value**: Premium pass feels worth the investment
- **Accessibility**: Free track provides real value
- **Completion**: Achievable by regular players
- **FOMO Management**: Pressure without frustration

### Core Structure

```typescript
interface BattlePassStructure {
  totalTiers: 60;
  seasonDuration: {
    weeks: 8;
    days: 56;
  };
  tracks: {
    free: 'Available to all players';
    premium: 'Requires Battle Pass purchase';
  };
  xpPerTier: 10000;
  totalXpRequired: 600000;
}
```

---

## Season Configuration

### Season Parameters

```typescript
interface SeasonConfig {
  seasonId: string;
  seasonNumber: number;
  name: string;
  theme: string;

  timing: {
    startDate: Date;
    endDate: Date;
    durationDays: number;
    gracePeriodDays: number; // Extra time to claim rewards
  };

  passConfig: {
    totalTiers: number;
    xpPerTier: number;
    freeRewardsCount: number;
    premiumRewardsCount: number;
  };

  pricing: {
    standardPass: number; // Doubloons
    bundlePass: number;   // Pass + tier skips
    tierSkipPrice: number;
  };
}

const SEASON_EXAMPLE: SeasonConfig = {
  seasonId: 'season_001',
  seasonNumber: 1,
  name: 'Tides of Fortune',
  theme: 'Treasure Hunting',

  timing: {
    startDate: new Date('2025-01-15T00:00:00Z'),
    endDate: new Date('2025-03-11T23:59:59Z'),
    durationDays: 56,
    gracePeriodDays: 3
  },

  passConfig: {
    totalTiers: 60,
    xpPerTier: 10000,
    freeRewardsCount: 25,
    premiumRewardsCount: 60
  },

  pricing: {
    standardPass: 950,
    bundlePass: 1500,
    tierSkipPrice: 150
  }
};
```

---

## Tier System

### Tier Structure

```typescript
interface BattlePassTier {
  tierNumber: number;
  xpRequired: number;
  cumulativeXp: number;
  freeReward?: Reward;
  premiumReward: Reward;
  isMilestone: boolean;
}

const TIER_STRUCTURE: TierStructure = {
  tierCount: 60,
  xpPerTier: 10000,

  milestones: [
    { tier: 1, description: 'Immediate reward on purchase' },
    { tier: 10, description: 'Early milestone' },
    { tier: 20, description: 'Quarter completion' },
    { tier: 30, description: 'Halfway point' },
    { tier: 40, description: 'Three-quarter mark' },
    { tier: 50, description: 'Near completion' },
    { tier: 60, description: 'Season finale reward' }
  ],

  xpProgression: 'Linear', // Same XP for each tier

  tierCalculation: (currentXp: number) => {
    const tier = Math.floor(currentXp / 10000);
    const xpInCurrentTier = currentXp % 10000;
    const progressPercent = (xpInCurrentTier / 10000) * 100;

    return {
      currentTier: Math.min(tier, 60),
      xpInCurrentTier,
      xpToNextTier: 10000 - xpInCurrentTier,
      progressPercent,
      isMaxTier: tier >= 60
    };
  }
};
```

### Tier Distribution

| Tier Range | Content Focus | Free Rewards | Premium Rewards |
|------------|---------------|--------------|-----------------|
| 1-10 | Hook players | 4 rewards | 10 rewards |
| 11-20 | Build momentum | 4 rewards | 10 rewards |
| 21-30 | Mid-season | 4 rewards | 10 rewards |
| 31-40 | Keep engagement | 4 rewards | 10 rewards |
| 41-50 | Reward dedication | 4 rewards | 10 rewards |
| 51-60 | Finale celebration | 5 rewards | 10 rewards |

---

## Dual Track System

### Track Configuration

```typescript
interface DualTrackConfig {
  free: TrackConfig;
  premium: TrackConfig;
}

const DUAL_TRACK: DualTrackConfig = {
  free: {
    name: 'Free Track',
    accessible: 'All players',
    rewardFrequency: 'Every 2-3 tiers',
    rewardTypes: [
      'Gold currency',
      'XP boosters',
      'Common cosmetics (1-2 per season)',
      'Profile items',
      'Small Doubloon drops (50-100 total)'
    ],
    totalRewards: 25,
    displayPosition: 'Top row'
  },
  premium: {
    name: 'Premium Track',
    accessible: 'Battle Pass owners',
    rewardFrequency: 'Every tier',
    rewardTypes: [
      'Exclusive outfits',
      'Premium emotes',
      'Legendary items',
      'Doubloons (950-1100 total)',
      'Epic profile customization',
      'Exclusive titles'
    ],
    totalRewards: 60,
    displayPosition: 'Bottom row'
  }
};
```

### Visual Track Layout

```
┌─────────────────────────────────────────────────────────────┐
│  BATTLE PASS - Season 1: Tides of Fortune                   │
│  Tier 23/60  |  23,450 / 240,000 XP  |  ⏰ 34d 12h left     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  FREE TRACK                                                 │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
│  │ 🪙  │ │     │ │ 📦 │ │     │ │ 💎  │ │     │ │ 🎭  │  │
│  │ 500 │ │  -  │ │Crate│ │  -  │ │ 25  │ │  -  │ │Emote│  │
│  │ ✓   │ │ ✓   │ │ ✓   │ │ ✓   │ │ ✓   │ │     │ │     │  │
│  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘  │
│     │      │      │      │      │      │      │           │
│  ───●──────●──────●──────●──────●──────●──────○──────○───  │
│    21     22     23     24     25     26     27     28     │
│     │      │      │      │      │      │      │           │
│  ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐  │
│  │ 🎽  │ │ 💎  │ │ 🪙  │ │ ⭐  │ │ 🎭  │ │ 💎  │ │ 👕  │  │
│  │Hat  │ │ 50  │ │1000 │ │Boost│ │Dance│ │ 75  │ │Outfit │
│  │ ✓   │ │ ✓   │ │ ✓   │ │     │ │     │ │     │ │     │  │
│  │ 🔒  │ │ 🔒  │ │ 🔒  │ │ 🔒  │ │ 🔒  │ │ 🔒  │ │ 🔒  │  │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘  │
│  PREMIUM TRACK                          [🔓 UNLOCK PASS]  │
│                                                             │
│  [◀ Previous]      Current Tier: 23      [Next ▶]          │
│                    ████████░░ 45%                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Battle Pass UI

### Main Battle Pass Screen

```typescript
interface BattlePassUI {
  components: UIComponent[];
}

const BATTLE_PASS_UI: BattlePassUI = {
  components: [
    {
      type: 'Header',
      elements: [
        { id: 'season_name', display: 'Season name and number' },
        { id: 'tier_progress', display: 'Current tier / Total tiers' },
        { id: 'xp_counter', display: 'Current XP / XP to next tier' },
        { id: 'season_timer', display: 'Time remaining in season' }
      ]
    },
    {
      type: 'TrackView',
      layout: 'horizontal_scroll',
      elements: [
        { id: 'free_track', position: 'top', rewards: 'Free tier rewards' },
        { id: 'progress_line', position: 'middle', connects: 'All tiers' },
        { id: 'premium_track', position: 'bottom', rewards: 'Premium tier rewards' }
      ]
    },
    {
      type: 'TierDetail',
      trigger: 'Tap on reward',
      display: 'Modal with reward preview'
    },
    {
      type: 'ActionBar',
      buttons: [
        { id: 'view_all_rewards', label: 'View All Rewards' },
        { id: 'purchase_pass', label: 'Get Battle Pass', conditional: '!hasPremium' },
        { id: 'buy_tiers', label: 'Buy Tiers', conditional: 'hasPremium' }
      ]
    }
  ]
};
```

### Tier Reward Card

```typescript
interface TierRewardCard {
  states: CardState[];
}

const REWARD_CARD_STATES: TierRewardCard = {
  states: [
    {
      state: 'LOCKED',
      description: 'Tier not yet reached',
      appearance: {
        opacity: 0.5,
        overlay: 'lock_icon',
        showPreview: true,
        tapAction: 'Show reward preview'
      }
    },
    {
      state: 'UNLOCKED_UNCLAIMED',
      description: 'Tier reached, reward available',
      appearance: {
        opacity: 1.0,
        highlight: 'golden_glow',
        badge: 'CLAIM',
        animation: 'pulse',
        tapAction: 'Claim reward'
      }
    },
    {
      state: 'CLAIMED',
      description: 'Reward already claimed',
      appearance: {
        opacity: 0.8,
        overlay: 'checkmark',
        tapAction: 'View reward details'
      }
    },
    {
      state: 'PREMIUM_LOCKED',
      description: 'Premium reward without pass',
      appearance: {
        opacity: 0.6,
        overlay: 'premium_lock_icon',
        showPreview: true,
        tapAction: 'Prompt to purchase pass'
      }
    }
  ]
};
```

### Progress Display

```typescript
interface ProgressDisplay {
  elements: ProgressElement[];
}

const PROGRESS_DISPLAY: ProgressDisplay = {
  elements: [
    {
      id: 'tier_counter',
      format: 'Tier {current} / {total}',
      example: 'Tier 23 / 60'
    },
    {
      id: 'xp_progress',
      format: '{currentXP} / {nextTierXP} XP',
      example: '4,500 / 10,000 XP'
    },
    {
      id: 'progress_bar',
      type: 'segmented',
      segments: 60,
      filledColor: 'gold_gradient',
      currentMarker: 'glowing_indicator'
    },
    {
      id: 'tier_preview',
      display: 'Next 5-10 tiers visible',
      scroll: 'Horizontal swipe to view more'
    }
  ]
};
```

---

## Premium Pass Purchase

### Purchase Flow

```typescript
interface PassPurchaseFlow {
  products: PassProduct[];
  flow: PurchaseStep[];
}

const PASS_PURCHASE: PassPurchaseFlow = {
  products: [
    {
      productId: 'battle_pass_standard',
      name: 'Battle Pass',
      price: 950,
      currency: 'Doubloons',
      includes: [
        'Access to premium reward track',
        'Instant unlock of reached premium rewards',
        'Exclusive Season 1 outfit at Tier 60',
        '1000+ Doubloons in rewards'
      ],
      uiEmphasis: 'Standard'
    },
    {
      productId: 'battle_pass_bundle',
      name: 'Battle Pass Bundle',
      price: 1500,
      currency: 'Doubloons',
      includes: [
        'Everything in standard Battle Pass',
        '+10 instant tier skips',
        'Exclusive bundle-only frame'
      ],
      savings: 'Save 100 Doubloons vs separate',
      uiEmphasis: 'Recommended',
      badge: 'BEST VALUE'
    }
  ],

  flow: [
    {
      step: 1,
      screen: 'Pass Purchase Modal',
      display: 'Both options side by side',
      highlight: 'Bundle as best value'
    },
    {
      step: 2,
      screen: 'Confirmation',
      display: 'Selected product details',
      showBalance: true,
      insufficientFunds: 'Link to currency purchase'
    },
    {
      step: 3,
      screen: 'Success',
      animation: 'Premium unlock celebration',
      action: 'Instantly claim all reached premium rewards'
    }
  ]
};
```

### Purchase Modal Layout

```
┌─────────────────────────────────────────────────────────────┐
│                   🏴‍☠️ GET BATTLE PASS 🏴‍☠️                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────┐  ┌──────────────────────┐        │
│  │                      │  │    ⭐ BEST VALUE ⭐   │        │
│  │    BATTLE PASS       │  │                      │        │
│  │                      │  │  BATTLE PASS BUNDLE  │        │
│  │    [Pass Image]      │  │                      │        │
│  │                      │  │    [Bundle Image]    │        │
│  │  ✓ Premium rewards   │  │                      │        │
│  │  ✓ Season outfit     │  │  ✓ Everything in     │        │
│  │  ✓ 1000+ Doubloons   │  │    standard pass     │        │
│  │                      │  │  ✓ +10 Tier Skips    │        │
│  │                      │  │  ✓ Exclusive Frame   │        │
│  │                      │  │                      │        │
│  │    💎 950            │  │    💎 1,500          │        │
│  │                      │  │    Save 100 💎       │        │
│  │  [    SELECT    ]    │  │  [    SELECT    ]    │        │
│  │                      │  │                      │        │
│  └──────────────────────┘  └──────────────────────┘        │
│                                                             │
│  Your Balance: 💎 2,450                                    │
│                                                             │
│  [Preview All 60 Rewards]                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Reward Claiming System

### Claim Options

```typescript
interface ClaimSystem {
  methods: ClaimMethod[];
  configuration: ClaimConfig;
}

const CLAIM_SYSTEM: ClaimSystem = {
  methods: [
    {
      method: 'MANUAL_CLAIM',
      description: 'Player taps each reward to claim',
      animation: 'Reward reveal animation per item',
      satisfaction: 'High - deliberate action',
      default: false
    },
    {
      method: 'CLAIM_ALL',
      description: 'Button to claim all available rewards',
      animation: 'Rapid succession reveal',
      satisfaction: 'Medium - efficient',
      available: 'When 3+ rewards unclaimed'
    },
    {
      method: 'AUTO_CLAIM',
      description: 'Rewards claimed automatically on tier-up',
      animation: 'Toast notification',
      satisfaction: 'Low - passive',
      setting: 'User preference toggle'
    }
  ],

  configuration: {
    defaultMethod: 'MANUAL_CLAIM',
    autoClaimSetting: {
      location: 'Settings > Battle Pass',
      default: false,
      description: 'Automatically claim rewards when unlocked'
    },
    claimAllThreshold: 3,
    missedRewardHandling: 'Sent to inbox at season end'
  }
};
```

### Claim Animation

```typescript
interface ClaimAnimation {
  sequence: AnimationStep[];
}

const CLAIM_ANIMATION: ClaimAnimation = {
  sequence: [
    {
      step: 1,
      duration: 200,
      action: 'Card scales up slightly',
      easing: 'easeOut'
    },
    {
      step: 2,
      duration: 300,
      action: 'Card flips/opens to reveal reward',
      particles: 'golden_sparkles'
    },
    {
      step: 3,
      duration: 500,
      action: 'Reward item zooms to center',
      sound: 'reward_reveal.wav'
    },
    {
      step: 4,
      duration: 400,
      action: 'Item details appear',
      display: ['Name', 'Rarity', 'Description']
    },
    {
      step: 5,
      duration: 200,
      action: 'Item flies to inventory',
      confirmation: '"Added to your collection!"'
    }
  ],
  totalDuration: 1600,
  skipOption: 'Tap anywhere to skip'
};
```

---

## Full Rewards Preview

### Preview Screen

```typescript
interface RewardsPreview {
  layout: PreviewLayout;
  features: PreviewFeature[];
}

const REWARDS_PREVIEW: RewardsPreview = {
  layout: {
    type: 'full_screen_modal',
    display: 'grid_or_list_toggle',
    defaultView: 'grid'
  },

  features: [
    {
      feature: 'Grid View',
      columns: 5,
      showsTier: true,
      showsTrack: 'Color-coded (gold premium, silver free)',
      showsRarity: true,
      showsOwned: 'Checkmark overlay'
    },
    {
      feature: 'List View',
      groupBy: 'Tier ranges (1-10, 11-20, etc.)',
      showsDetails: true,
      itemPreview: 'Expandable on tap'
    },
    {
      feature: 'Filter Options',
      filters: [
        { id: 'all', label: 'All Rewards' },
        { id: 'free', label: 'Free Track Only' },
        { id: 'premium', label: 'Premium Only' },
        { id: 'unclaimed', label: 'Available to Claim' },
        { id: 'type', label: 'By Type', options: ['Outfits', 'Emotes', 'Currency', 'Profile'] }
      ]
    },
    {
      feature: 'Highlight Markers',
      milestones: 'Star icon on milestone tiers',
      exclusive: '"Exclusive" tag on pass-only items',
      current: 'Current tier highlighted'
    }
  ]
};
```

### Preview Grid Layout

```
┌─────────────────────────────────────────────────────────────┐
│  ALL REWARDS - Season 1         [Grid|List] [Filter ▼]     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  TIERS 1-10                                                 │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐  │
│  │ 👕 │ │ 🪙 │ │ 💎 │ │ 🎭 │ │ ⭐ │ │ 💎 │ │ 📦 │ │ 👕 │  │
│  │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │  │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘  │
│    1P     2F     3P     4P     5F     6P     7P     8F     │
│                                                             │
│  ┌────┐ ┌────┐                                             │
│  │ 🎽 │ │ ⭐ │  ← Tier 10 Milestone!                       │
│  │ ✓  │ │ ✓  │                                             │
│  └────┘ └────┘                                             │
│    9P    10P                                                │
│                                                             │
│  TIERS 11-20                                                │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐  │
│  │ 💎 │ │ 🎭 │ │ 🪙 │ │ 👕 │ │ 💎 │ │ 🎭 │ │ 📦 │ │ 💎 │  │
│  │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │ │ ✓  │  │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘  │
│   11P    12F    13P    14P    15F    16P    17P    18F     │
│                                                             │
│  P = Premium  |  F = Free  |  ✓ = Claimed                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Season Timer

### Countdown Display

```typescript
interface SeasonTimer {
  display: TimerDisplay;
  urgencyStates: UrgencyState[];
}

const SEASON_TIMER: SeasonTimer = {
  display: {
    location: 'Battle Pass header',
    format: {
      moreThan7Days: '{X}d {Y}h remaining',
      lessThan7Days: '{X}d {Y}h {Z}m remaining',
      lessThan24Hours: '{X}h {Y}m {Z}s remaining',
      lessThan1Hour: '{X}m {Y}s remaining'
    },
    icon: '⏰',
    tooltip: 'Season ends on {endDate}'
  },

  urgencyStates: [
    {
      timeRemaining: '>14 days',
      styling: 'normal',
      color: 'secondary_text',
      notifications: []
    },
    {
      timeRemaining: '7-14 days',
      styling: 'attention',
      color: 'primary_text',
      notifications: ['In-app banner']
    },
    {
      timeRemaining: '3-7 days',
      styling: 'warning',
      color: 'warning_yellow',
      notifications: ['Push notification', 'In-app popup']
    },
    {
      timeRemaining: '<3 days',
      styling: 'critical',
      color: 'critical_red',
      animation: 'pulse',
      notifications: ['Daily push', 'Prominent banner']
    },
    {
      timeRemaining: '<24 hours',
      styling: 'final',
      color: 'critical_red',
      animation: 'urgent_pulse',
      badge: 'FINAL DAY',
      notifications: ['Multiple pushes', 'Full-screen prompt']
    }
  ]
};
```

---

## Season End Handling

### End-of-Season Process

```typescript
interface SeasonEndProcess {
  phases: SeasonEndPhase[];
}

const SEASON_END: SeasonEndProcess = {
  phases: [
    {
      phase: 'PRE_END_WARNING',
      timing: '7 days before',
      actions: [
        'Banner in Battle Pass: "Season ending soon!"',
        'Push notification to players with unclaimed rewards',
        'Highlight unclaimed rewards in UI'
      ]
    },
    {
      phase: 'FINAL_DAY',
      timing: '<24 hours',
      actions: [
        'Critical urgency styling',
        'Pop-up warning on app open',
        'Disable auto-claim to ensure player sees rewards',
        'Offer tier skip bundles for completion'
      ]
    },
    {
      phase: 'SEASON_END',
      timing: 'Exact end time',
      actions: [
        'Lock all progression (XP no longer earned for this pass)',
        'Grace period begins (3 days)',
        'Display "Season Ended" banner'
      ]
    },
    {
      phase: 'GRACE_PERIOD',
      timing: '1-3 days after end',
      actions: [
        'Players can still claim unlocked rewards',
        'Cannot earn more XP or unlock new tiers',
        'Cannot purchase tier skips',
        'Countdown to grace period end'
      ]
    },
    {
      phase: 'GRACE_END',
      timing: '3 days after season end',
      actions: [
        'Collect all unclaimed rewards',
        'Deliver to player inbox',
        'Clear Battle Pass progress',
        'Prepare for new season'
      ]
    },
    {
      phase: 'NEW_SEASON',
      timing: 'After maintenance',
      actions: [
        'Reset all player progress to Tier 0',
        'New season content loaded',
        'Premium pass status reset (need to repurchase)',
        'Celebration announcement'
      ]
    }
  ]
};
```

### Unclaimed Rewards Handling

```typescript
interface UnclaimedRewardsPolicy {
  policy: RewardPolicy;
}

const UNCLAIMED_POLICY: UnclaimedRewardsPolicy = {
  policy: {
    freeTrackRewards: {
      handling: 'Auto-deliver to inbox',
      expiryInInbox: 'Never (permanent)',
      notification: 'You have unclaimed Battle Pass rewards!'
    },
    premiumTrackRewards: {
      handling: 'Auto-deliver to inbox',
      expiryInInbox: 'Never (permanent)',
      notification: 'Your premium rewards are waiting!'
    },
    unlockedButUnclaimed: {
      handling: 'Auto-deliver',
      description: 'Rewards from tiers reached but not claimed'
    },
    lockedPremium: {
      handling: 'Lost',
      description: 'Premium rewards from tiers not reached',
      mitigation: 'Clear warning before season end'
    },
    specialHandling: {
      doubloons: 'Direct credit to account',
      gold: 'Direct credit to account',
      items: 'Delivered to inbox'
    }
  }
};
```

---

## Tier Purchase System

### Buy Tiers Flow

```typescript
interface TierPurchase {
  options: TierPurchaseOption[];
  restrictions: PurchaseRestriction[];
}

const TIER_PURCHASE: TierPurchase = {
  options: [
    {
      quantity: 1,
      price: 150,
      currency: 'Doubloons',
      display: 'Buy 1 Tier'
    },
    {
      quantity: 5,
      price: 650,
      currency: 'Doubloons',
      originalPrice: 750,
      savings: '13%',
      display: 'Buy 5 Tiers'
    },
    {
      quantity: 10,
      price: 1200,
      currency: 'Doubloons',
      originalPrice: 1500,
      savings: '20%',
      display: 'Buy 10 Tiers',
      badge: 'POPULAR'
    },
    {
      quantity: 25,
      price: 2750,
      currency: 'Doubloons',
      originalPrice: 3750,
      savings: '27%',
      display: 'Buy 25 Tiers',
      badge: 'BEST VALUE'
    }
  ],

  restrictions: [
    {
      restriction: 'Requires Premium Pass',
      description: 'Must own Battle Pass to buy tiers',
      prompt: 'Purchase pass first?'
    },
    {
      restriction: 'Max Tier Cap',
      description: 'Cannot exceed Tier 60',
      behavior: 'Adjust quantity to remaining tiers'
    },
    {
      restriction: 'Season Active',
      description: 'Cannot buy during grace period',
      message: 'Season has ended'
    }
  ]
};
```

### Buy Tiers Modal

```
┌─────────────────────────────────────────────────────────────┐
│                    ⚡ BUY TIERS ⚡                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Current: Tier 23   →   Tiers Available: 37                │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   1 TIER     │  │   5 TIERS    │  │  10 TIERS    │      │
│  │              │  │              │  │   POPULAR    │      │
│  │   💎 150     │  │   💎 650     │  │   💎 1,200   │      │
│  │              │  │   Save 13%   │  │   Save 20%   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │                  25 TIERS - BEST VALUE             │    │
│  │                                                    │    │
│  │           💎 2,750  (Save 27%)                    │    │
│  │           ~~💎 3,750~~                            │    │
│  │                                                    │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
│  Custom: [___] Tiers  =  💎 [calculated]                   │
│                                                             │
│  Your Balance: 💎 2,450                                    │
│                                                             │
│  [    CANCEL    ]        [    PURCHASE    ]                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Player Data Model

### Battle Pass Player State

```typescript
interface BattlePassPlayerState {
  playerId: string;
  seasonId: string;

  progression: {
    currentXp: number;
    currentTier: number;
    xpInCurrentTier: number;
  };

  ownership: {
    hasPremiumPass: boolean;
    purchaseDate?: Date;
    purchaseType: 'standard' | 'bundle' | null;
  };

  rewards: {
    claimedFree: number[];     // Tier numbers
    claimedPremium: number[];  // Tier numbers
    lastClaimDate?: Date;
  };

  settings: {
    autoClaimEnabled: boolean;
    notificationsEnabled: boolean;
  };

  statistics: {
    totalXpEarned: number;
    tiersPurchased: number;
    completionDate?: Date;
  };
}
```

---

## Implementation Checklist

### Core System
- [ ] Season configuration system
- [ ] 60-tier structure with XP requirements
- [ ] Dual track (free/premium) system
- [ ] XP tracking per player

### UI Components
- [ ] Main Battle Pass screen with horizontal scroll
- [ ] Tier reward cards with states (locked, unlocked, claimed)
- [ ] Progress display (tier counter, XP bar)
- [ ] Full rewards preview grid/list

### Purchase System
- [ ] Standard pass purchase flow
- [ ] Bundle pass purchase flow
- [ ] Tier skip purchase (1, 5, 10, 25)
- [ ] Insufficient funds handling

### Claiming System
- [ ] Manual claim per reward
- [ ] Claim all button
- [ ] Auto-claim setting
- [ ] Claim animations

### Season Management
- [ ] Season countdown timer
- [ ] Urgency states (warning, critical)
- [ ] Grace period handling
- [ ] Unclaimed rewards delivery
- [ ] Season transition

### Notifications
- [ ] Season ending reminders
- [ ] Tier unlock notifications
- [ ] Daily/weekly quest completion
- [ ] New season announcement

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
