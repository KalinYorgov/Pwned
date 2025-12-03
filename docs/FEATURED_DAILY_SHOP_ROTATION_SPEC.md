# Featured/Daily Shop Rotation Specification

## Overview

This document defines the rotating shop system for Plunderstorm Mobile, creating urgency through time-limited availability while ensuring variety and fairness in item selection.

## Rotation Philosophy

### Design Goals
- **FOMO Balance**: Create urgency without frustration
- **Variety**: Players see different items on each visit
- **Fairness**: Popular items return regularly
- **Revenue**: Premium items highlighted at optimal times
- **Freshness**: New items featured prominently

### Rotation Types

```typescript
enum RotationType {
  DAILY = 'daily',       // Refreshes every 24 hours
  WEEKLY = 'weekly',     // Refreshes every 7 days
  EVENT = 'event',       // Special event-driven rotation
  FLASH = 'flash'        // Short-term (4-8 hour) sales
}
```

---

## Daily Rotation System

### Configuration

```typescript
interface DailyRotationConfig {
  refreshTime: '00:00 UTC';
  itemCount: {
    minimum: 4;
    maximum: 6;
    typical: 5;
  };
  slotDefinitions: DailySlot[];
  selectionRules: SelectionRule[];
}

const DAILY_SLOTS: DailySlot[] = [
  {
    slotId: 'daily_featured',
    name: 'Daily Featured',
    size: 'large',
    position: 1,
    itemPool: 'epic_legendary_outfits',
    saleChance: 0.3,
    saleDiscount: { min: 15, max: 30 }
  },
  {
    slotId: 'daily_outfit',
    name: 'Daily Outfit',
    size: 'medium',
    position: 2,
    itemPool: 'all_outfits',
    excludeRarity: ['Mythic'],
    saleChance: 0.2,
    saleDiscount: { min: 10, max: 25 }
  },
  {
    slotId: 'daily_emote',
    name: 'Daily Emote',
    size: 'medium',
    position: 3,
    itemPool: 'all_emotes',
    saleChance: 0.15,
    saleDiscount: { min: 10, max: 20 }
  },
  {
    slotId: 'daily_deal_1',
    name: 'Daily Deal',
    size: 'small',
    position: 4,
    itemPool: 'gold_purchasable',
    saleChance: 0.4,
    saleDiscount: { min: 20, max: 40 }
  },
  {
    slotId: 'daily_deal_2',
    name: 'Daily Deal',
    size: 'small',
    position: 5,
    itemPool: 'profile_items_emotes',
    saleChance: 0.25,
    saleDiscount: { min: 15, max: 30 }
  }
];
```

### Daily Layout

```
┌─────────────────────────────────────────────────────────────┐
│                      DAILY SHOP                              │
│               ⏰ Refreshes in 14:32:45                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │              🌟 DAILY FEATURED 🌟                   │   │
│  │                                                     │   │
│  │         [Large Item Image - Outfit Preview]        │   │
│  │                                                     │   │
│  │              Storm Sovereign Outfit                │   │
│  │              ⭐ Legendary                          │   │
│  │                                                     │   │
│  │           ~~💎 1,200~~  💎 900  (25% OFF)          │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────┐  ┌──────────────────────┐        │
│  │    Daily Outfit      │  │    Daily Emote       │        │
│  │    [Item Image]      │  │    [Item Image]      │        │
│  │    Wave Rider        │  │    Sea Shanty        │        │
│  │    💎 800            │  │    🪙 2,500          │        │
│  └──────────────────────┘  └──────────────────────┘        │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Daily Deal  │  │ Daily Deal  │  │ Daily Deal  │         │
│  │ [Image]     │  │ [Image]     │  │ [Image]     │         │
│  │ Card Frame  │  │ Taunt Emote │  │ Title       │         │
│  │ ~~🪙800~~   │  │ 🪙 1,000    │  │ 🪙 500      │         │
│  │ 🪙 560      │  │             │  │             │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Item Selection Algorithm

```typescript
interface SelectionAlgorithm {
  process: SelectionStep[];
}

const DAILY_SELECTION: SelectionAlgorithm = {
  process: [
    {
      step: 1,
      name: 'Pool Generation',
      action: 'Generate eligible items for each slot',
      filters: [
        'Remove items in cooldown period',
        'Remove currently owned by player (optional)',
        'Remove items in other daily slots',
        'Apply slot-specific filters (rarity, type, price)'
      ]
    },
    {
      step: 2,
      name: 'Weighting',
      action: 'Apply selection weights',
      weights: {
        newItems: 2.0,          // 7 days or newer
        neverRotated: 1.5,      // First time in rotation
        popularItems: 1.3,      // High purchase rate
        seasonalRelevance: 1.2, // Matches current season/event
        standard: 1.0           // Base weight
      }
    },
    {
      step: 3,
      name: 'Randomized Selection',
      action: 'Weighted random selection',
      method: 'Weighted random without replacement'
    },
    {
      step: 4,
      name: 'Sale Assignment',
      action: 'Determine which items go on sale',
      rules: [
        'Roll for sale chance per slot',
        'Calculate discount within slot range',
        'Apply discount to selected items'
      ]
    },
    {
      step: 5,
      name: 'Validation',
      action: 'Final validation checks',
      checks: [
        'No duplicate items across slots',
        'At least one Gold-purchasable item',
        'At least one Doubloon item',
        'Variety in item types'
      ]
    }
  ]
};
```

### Cooldown System

```typescript
interface CooldownConfig {
  rules: CooldownRule[];
}

const COOLDOWN_RULES: CooldownRule[] = [
  {
    itemRarity: 'Common',
    cooldownDays: 3,
    description: 'Common items wait 3 days before reappearing'
  },
  {
    itemRarity: 'Rare',
    cooldownDays: 5,
    description: 'Rare items wait 5 days before reappearing'
  },
  {
    itemRarity: 'Epic',
    cooldownDays: 7,
    description: 'Epic items wait 7 days before reappearing'
  },
  {
    itemRarity: 'Legendary',
    cooldownDays: 14,
    description: 'Legendary items wait 14 days before reappearing'
  },
  {
    itemRarity: 'Mythic',
    cooldownDays: 30,
    description: 'Mythic items wait 30 days before reappearing'
  },
  {
    condition: 'Previously on sale',
    additionalCooldown: 7,
    description: 'Items that were on sale add 7 days to cooldown'
  }
];

interface ItemCooldownTracker {
  itemId: string;
  lastAppeared: Date;
  wasOnSale: boolean;
  cooldownUntil: Date;
}
```

---

## Weekly Rotation System

### Configuration

```typescript
interface WeeklyRotationConfig {
  refreshDay: 'Tuesday';
  refreshTime: '00:00 UTC';
  duration: 7; // days
  slotDefinitions: WeeklySlot[];
}

const WEEKLY_SLOTS: WeeklySlot[] = [
  {
    slotId: 'weekly_legendary',
    name: 'Legendary of the Week',
    size: 'hero',
    position: 1,
    itemPool: 'legendary_mythic_outfits',
    guaranteed: true,
    saleChance: 0.2,
    saleDiscount: { min: 10, max: 20 }
  },
  {
    slotId: 'weekly_bundle',
    name: 'Weekly Bundle',
    size: 'large',
    position: 2,
    itemPool: 'themed_bundles',
    guaranteed: true,
    displaySavings: true
  },
  {
    slotId: 'weekly_collection',
    name: 'Collection Spotlight',
    size: 'medium',
    position: 3,
    itemPool: 'collection_sets',
    itemCount: 3,
    description: 'Related items from same collection'
  },
  {
    slotId: 'weekly_premium_emote',
    name: 'Premium Emote',
    size: 'medium',
    position: 4,
    itemPool: 'legendary_emotes'
  }
];
```

### Weekly Layout

```
┌─────────────────────────────────────────────────────────────┐
│                      WEEKLY SHOP                             │
│               ⏰ Refreshes in 3d 14:32:45                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │           ✨ LEGENDARY OF THE WEEK ✨              │   │
│  │                                                     │   │
│  │         [Hero Banner - Full Character Preview]     │   │
│  │                                                     │   │
│  │              Kraken Lord Outfit Set                │   │
│  │              🌟 Mythic Collection                  │   │
│  │                                                     │   │
│  │                   💎 2,000                         │   │
│  │                                                     │   │
│  │      [VIEW DETAILS]        [PURCHASE]             │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │               📦 WEEKLY BUNDLE 📦                  │   │
│  │                                                     │   │
│  │  Ocean Explorer Set                                │   │
│  │  [Outfit] [Emote] [Frame] [Title]                 │   │
│  │                                                     │   │
│  │  Value: 💎 2,400    Price: 💎 1,800  (25% OFF)    │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Collection Spotlight: Ghostly Pirates            │      │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐          │      │
│  │  │ Outfit  │  │ Emote   │  │ Effects │          │      │
│  │  │ 💎 1200 │  │ 💎 400  │  │ 💎 300  │          │      │
│  │  └─────────┘  └─────────┘  └─────────┘          │      │
│  │  [Buy Collection - Save 20%]                      │      │
│  └──────────────────────────────────────────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Event Rotation System

### Event Configuration

```typescript
interface EventRotationConfig {
  eventTypes: EventType[];
  overrideRules: OverrideRule[];
}

const EVENT_TYPES: EventType[] = [
  {
    type: 'SEASONAL',
    examples: ['Halloween', 'Winter Holiday', 'Summer'],
    duration: '2-4 weeks',
    shopBehavior: {
      dedicatedSection: true,
      limitedTimeItems: true,
      thematicItems: true,
      returnPolicy: 'May return next year'
    }
  },
  {
    type: 'GAME_ANNIVERSARY',
    examples: ['1 Year Anniversary', 'Launch Week'],
    duration: '1-2 weeks',
    shopBehavior: {
      exclusiveItems: true,
      specialBundles: true,
      retrospectiveItems: true,
      returnPolicy: 'Exclusive to anniversary'
    }
  },
  {
    type: 'COLLABORATION',
    examples: ['Partner events', 'Cross-promotions'],
    duration: 'Variable',
    shopBehavior: {
      partnerItems: true,
      limitedQuantity: 'possible',
      returnPolicy: 'May never return'
    }
  },
  {
    type: 'FLASH_EVENT',
    examples: ['Weekend Sale', 'Midweek Madness'],
    duration: '24-72 hours',
    shopBehavior: {
      deepDiscounts: true,
      bundleDeals: true,
      returnPolicy: 'Regular rotation after'
    }
  }
];
```

### Event Override System

```typescript
interface EventOverride {
  priority: number;
  rules: EventOverrideRule[];
}

const EVENT_OVERRIDE_RULES: EventOverrideRule[] = [
  {
    rule: 'Force Featured Item',
    description: 'Admin can force specific item into featured slot',
    priority: 'HIGHEST',
    useCase: 'New legendary release, event item'
  },
  {
    rule: 'Reserve Slots',
    description: 'Reserve specific slots for event items',
    priority: 'HIGH',
    useCase: 'Event section in daily shop'
  },
  {
    rule: 'Override Cooldown',
    description: 'Bring back items regardless of cooldown',
    priority: 'MEDIUM',
    useCase: 'Popular item return during sale'
  },
  {
    rule: 'Disable Regular Rotation',
    description: 'Replace entire daily shop with event shop',
    priority: 'LOW',
    useCase: 'Major event with full takeover'
  }
];
```

### Event Shop Layout

```
┌─────────────────────────────────────────────────────────────┐
│                  🎃 HALLOWEEN EVENT 🎃                      │
│               ⏰ Event ends in 6d 14:32:45                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         👻 LIMITED EDITION - GHOST CAPTAIN 👻      │   │
│  │                                                     │   │
│  │         [Spooky Character Preview]                 │   │
│  │                                                     │   │
│  │              💎 1,500  ⚠️ LIMITED TIME             │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  EVENT ITEMS                                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │ 🎃      │  │ 👻      │  │ 🦇      │  │ 💀      │       │
│  │ Pumpkin │  │ Ghost   │  │ Bat     │  │ Skeleton│       │
│  │ Outfit  │  │ Emote   │  │ Trail   │  │ Frame   │       │
│  │ 💎 800  │  │ 💎 400  │  │ 💎 300  │  │ 🪙 600  │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
│                                                             │
│  🎃 HALLOWEEN BUNDLE - Save 30%                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ All 4 event items + Exclusive Title                │   │
│  │ Value: 💎 2,100  →  Price: 💎 1,470               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  [View Regular Daily Shop →]                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Countdown Timer System

### Timer Display

```typescript
interface CountdownTimer {
  display: TimerDisplay;
  behavior: TimerBehavior;
}

const COUNTDOWN_DISPLAY: CountdownTimer = {
  display: {
    format: {
      moreThan24h: 'Xd Xh Xm',
      lessThan24h: 'Xh Xm Xs',
      lessThan1h: 'Xm Xs',
      lessThan5m: 'Xs (urgent styling)'
    },
    position: 'Below section header',
    icon: '⏰',
    urgencyThresholds: [
      { time: '24h', styling: 'normal' },
      { time: '6h', styling: 'warning_yellow' },
      { time: '1h', styling: 'urgent_orange' },
      { time: '15m', styling: 'critical_red_pulse' }
    ]
  },
  behavior: {
    updateFrequency: '1 second',
    onExpire: 'Show "Refreshing..." then reload',
    preloadNextRotation: '5 minutes before expiry',
    offlineHandling: 'Sync on reconnect'
  }
};
```

### Timer UI States

```typescript
const TIMER_STATES: TimerState[] = [
  {
    state: 'NORMAL',
    display: '⏰ Refreshes in 14:32:45',
    styling: {
      backgroundColor: 'transparent',
      textColor: 'secondary_grey',
      animation: 'none'
    }
  },
  {
    state: 'WARNING',
    timeRemaining: '<6 hours',
    display: '⏰ Only 5:45:12 left!',
    styling: {
      backgroundColor: 'warning_yellow_subtle',
      textColor: 'warning_yellow',
      animation: 'none'
    }
  },
  {
    state: 'URGENT',
    timeRemaining: '<1 hour',
    display: '⚡ Ending soon! 45:12',
    styling: {
      backgroundColor: 'urgent_orange_subtle',
      textColor: 'urgent_orange',
      animation: 'subtle_pulse'
    }
  },
  {
    state: 'CRITICAL',
    timeRemaining: '<15 minutes',
    display: '🔥 LAST CHANCE! 12:45',
    styling: {
      backgroundColor: 'critical_red_subtle',
      textColor: 'critical_red',
      animation: 'pulse'
    }
  },
  {
    state: 'EXPIRED',
    display: '🔄 Refreshing...',
    styling: {
      backgroundColor: 'loading_grey',
      textColor: 'white',
      animation: 'spinner'
    }
  }
];
```

---

## Admin Configuration Tools

### Rotation Management Interface

```typescript
interface AdminRotationTool {
  features: AdminFeature[];
}

const ADMIN_FEATURES: AdminFeature[] = [
  {
    feature: 'View Current Rotation',
    description: 'See all items in current daily/weekly rotation',
    data: ['Item ID', 'Name', 'Slot', 'Price', 'Sale status', 'Time remaining']
  },
  {
    feature: 'Preview Next Rotation',
    description: 'See auto-generated next rotation before it goes live',
    actions: ['Approve', 'Regenerate', 'Manually adjust']
  },
  {
    feature: 'Force Item',
    description: 'Insert specific item into rotation',
    parameters: {
      itemId: 'required',
      slot: 'optional (auto-assign if not specified)',
      duration: 'optional (default: until next rotation)',
      salePercent: 'optional'
    }
  },
  {
    feature: 'Remove Item',
    description: 'Remove specific item from current rotation',
    parameters: {
      itemId: 'required',
      replacement: 'optional (auto-fill or leave empty)'
    }
  },
  {
    feature: 'Schedule Override',
    description: 'Schedule future rotation changes',
    parameters: {
      startDate: 'required',
      endDate: 'required',
      overrideType: ['force_items', 'event_takeover', 'disable_rotation'],
      items: 'list of item configs'
    }
  },
  {
    feature: 'Cooldown Management',
    description: 'View and modify item cooldowns',
    actions: ['View cooldown list', 'Reset cooldown', 'Add cooldown']
  }
];
```

### Admin Dashboard Layout

```
┌─────────────────────────────────────────────────────────────┐
│  ROTATION ADMIN PANEL                    [Logout]           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CURRENT DAILY ROTATION (Ends in 14:32:45)                 │
│  ┌────────────────────────────────────────────────────┐    │
│  │ Slot 1: Storm Sovereign (Legendary) - 💎900 (-25%) │    │
│  │ Slot 2: Wave Rider (Epic) - 💎800                  │    │
│  │ Slot 3: Sea Shanty (Epic) - 🪙2500                 │    │
│  │ Slot 4: Gold Frame (Rare) - 🪙560 (-30%)          │    │
│  │ Slot 5: Sailor Title (Common) - 🪙500              │    │
│  └────────────────────────────────────────────────────┘    │
│  [Edit Current] [Force Refresh] [Preview Next]             │
│                                                             │
│  SCHEDULED OVERRIDES                                        │
│  ┌────────────────────────────────────────────────────┐    │
│  │ Oct 25-31: Halloween Event Takeover               │    │
│  │ Nov 15: Black Friday Flash Sale (24h)             │    │
│  └────────────────────────────────────────────────────┘    │
│  [+ Add Override]                                          │
│                                                             │
│  ITEM COOLDOWNS                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │ Kraken Lord - Available Dec 15                     │    │
│  │ Ghost Captain - Available Dec 20                   │    │
│  │ Storm Dancer - Available Dec 8 [Reset]            │    │
│  └────────────────────────────────────────────────────┘    │
│  [View All Cooldowns]                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### API Endpoints

```typescript
interface AdminAPIEndpoints {
  endpoints: APIEndpoint[];
}

const ADMIN_API: AdminAPIEndpoints = {
  endpoints: [
    {
      method: 'GET',
      path: '/admin/rotation/current',
      description: 'Get current rotation state',
      response: 'RotationState'
    },
    {
      method: 'GET',
      path: '/admin/rotation/preview',
      description: 'Preview next auto-generated rotation',
      response: 'RotationPreview'
    },
    {
      method: 'POST',
      path: '/admin/rotation/force-item',
      description: 'Force item into rotation',
      body: { itemId: 'string', slot?: 'string', sale?: 'number' }
    },
    {
      method: 'POST',
      path: '/admin/rotation/remove-item',
      description: 'Remove item from rotation',
      body: { itemId: 'string', replacement?: 'string' }
    },
    {
      method: 'POST',
      path: '/admin/rotation/schedule-override',
      description: 'Schedule future override',
      body: 'OverrideConfig'
    },
    {
      method: 'POST',
      path: '/admin/rotation/refresh',
      description: 'Force immediate rotation refresh',
      body: { regenerate?: 'boolean' }
    },
    {
      method: 'GET',
      path: '/admin/cooldowns',
      description: 'Get all item cooldowns',
      response: 'CooldownList'
    },
    {
      method: 'POST',
      path: '/admin/cooldowns/reset',
      description: 'Reset specific item cooldown',
      body: { itemId: 'string' }
    }
  ]
};
```

---

## Notification System

### Rotation Notifications

```typescript
interface RotationNotifications {
  triggers: NotificationTrigger[];
}

const NOTIFICATION_TRIGGERS: NotificationTrigger[] = [
  {
    trigger: 'New Daily Rotation',
    timing: 'At rotation refresh time',
    notification: {
      title: '🏴‍☠️ New Items in Shop!',
      body: 'Fresh loot has arrived! Check out today\'s deals.',
      deepLink: '/store/featured',
      scheduling: 'Only if user has notifications enabled'
    }
  },
  {
    trigger: 'New Weekly Rotation',
    timing: 'At weekly refresh',
    notification: {
      title: '✨ Weekly Legendary!',
      body: 'This week\'s legendary item: {itemName}',
      deepLink: '/store/weekly',
      image: '{itemThumbnail}'
    }
  },
  {
    trigger: 'Event Start',
    timing: 'Event launch',
    notification: {
      title: '🎉 {EventName} is Live!',
      body: 'Limited-time items available now!',
      deepLink: '/store/event',
      priority: 'HIGH'
    }
  },
  {
    trigger: 'Rotation Ending Soon',
    timing: '2 hours before rotation ends',
    notification: {
      title: '⏰ Shop Refreshing Soon!',
      body: 'Last chance for current deals!',
      deepLink: '/store/featured',
      conditions: ['User has viewed shop today', 'User has items in wishlist']
    }
  },
  {
    trigger: 'Wishlisted Item Available',
    timing: 'When wishlisted item enters rotation',
    notification: {
      title: '💝 Your Wishlist Item is Here!',
      body: '{itemName} is now available in the shop!',
      deepLink: '/store/item/{itemId}',
      priority: 'HIGH'
    }
  },
  {
    trigger: 'Flash Sale',
    timing: 'Flash sale start',
    notification: {
      title: '⚡ Flash Sale!',
      body: 'Limited time discounts for {duration}!',
      deepLink: '/store/flash',
      urgency: 'TIME_CRITICAL'
    }
  }
];
```

### Notification Preferences

```typescript
interface NotificationPreferences {
  userSettings: NotificationSetting[];
}

const NOTIFICATION_SETTINGS: NotificationSetting[] = [
  {
    setting: 'Daily Shop Refresh',
    default: false,
    description: 'Notify when new daily items arrive'
  },
  {
    setting: 'Weekly Shop Refresh',
    default: true,
    description: 'Notify about weekly legendary items'
  },
  {
    setting: 'Event Notifications',
    default: true,
    description: 'Notify about special events and limited items'
  },
  {
    setting: 'Wishlist Alerts',
    default: true,
    description: 'Notify when wishlisted items appear'
  },
  {
    setting: 'Sale Notifications',
    default: false,
    description: 'Notify about discounts and flash sales'
  }
];
```

---

## In-App Indicators

### New Rotation Badge

```typescript
interface NewRotationIndicator {
  display: BadgeConfig;
  behavior: BadgeBehavior;
}

const NEW_ROTATION_BADGE: NewRotationIndicator = {
  display: {
    badge: 'NEW',
    color: 'notification_red',
    position: 'Store button in main menu',
    animation: 'subtle_pulse'
  },
  behavior: {
    showCondition: 'Rotation changed since last store visit',
    clearCondition: 'User visits store',
    persistence: 'Until cleared or next rotation'
  }
};
```

### Section Indicators

```typescript
const SECTION_INDICATORS: SectionIndicator[] = [
  {
    section: 'Featured',
    indicators: [
      { type: 'NEW', condition: 'Items not seen before' },
      { type: 'SALE', condition: 'Items on sale' },
      { type: 'ENDING SOON', condition: '<6 hours remaining' }
    ]
  },
  {
    section: 'Weekly',
    indicators: [
      { type: 'NEW', condition: 'First day of rotation' },
      { type: 'LAST DAY', condition: 'Final 24 hours' }
    ]
  },
  {
    section: 'Event',
    indicators: [
      { type: 'LIMITED', condition: 'Always show for event items' },
      { type: 'EXCLUSIVE', condition: 'Items that won\'t return' }
    ]
  }
];
```

---

## Data Analytics

### Rotation Metrics

```typescript
interface RotationAnalytics {
  metrics: AnalyticsMetric[];
}

const ROTATION_METRICS: AnalyticsMetric[] = [
  {
    metric: 'Rotation View Rate',
    description: 'Percentage of DAU who view daily shop',
    target: '>60%'
  },
  {
    metric: 'Featured Click Rate',
    description: 'Clicks on featured item / featured views',
    target: '>30%'
  },
  {
    metric: 'Rotation Conversion Rate',
    description: 'Purchases / rotation views',
    target: '>5%'
  },
  {
    metric: 'Item Performance',
    description: 'Purchase rate by item when in rotation',
    segmentBy: ['rarity', 'price_tier', 'slot_position']
  },
  {
    metric: 'Sale Effectiveness',
    description: 'Lift in purchases when item is on sale',
    comparison: 'Same item, sale vs non-sale'
  },
  {
    metric: 'FOMO Engagement',
    description: 'Purchases in last 2 hours of rotation',
    insight: 'Urgency driver effectiveness'
  },
  {
    metric: 'Event Performance',
    description: 'Revenue during event vs normal period',
    segmentBy: ['event_type', 'duration']
  }
];
```

---

## Implementation Checklist

### Daily Rotation
- [ ] 5-slot daily layout with mixed sizes
- [ ] Item selection algorithm with weighting
- [ ] Cooldown system preventing back-to-back
- [ ] Sale assignment with configurable chances
- [ ] 24-hour countdown timer

### Weekly Rotation
- [ ] Weekly hero item slot
- [ ] Weekly bundle slot
- [ ] Collection spotlight section
- [ ] 7-day countdown timer
- [ ] Tuesday refresh schedule

### Event System
- [ ] Event takeover capability
- [ ] Limited-time item flagging
- [ ] Event countdown display
- [ ] Dedicated event section

### Admin Tools
- [ ] View current rotation
- [ ] Force item into rotation
- [ ] Remove item from rotation
- [ ] Schedule future overrides
- [ ] Cooldown management

### Notifications
- [ ] New rotation notifications
- [ ] Event start notifications
- [ ] Wishlist item alerts
- [ ] User preference settings

### Analytics
- [ ] Rotation view tracking
- [ ] Conversion rate by slot
- [ ] Sale effectiveness measurement
- [ ] Event performance comparison

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
