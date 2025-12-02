# Store UI Framework Specification

## Overview

This document defines the complete UI framework for the Plunderstorm Mobile in-game store, providing players with a seamless browsing and purchasing experience for cosmetic items, Battle Pass, and currency.

## Store Architecture

### Access Points

```typescript
interface StoreAccessPoint {
  location: string;
  trigger: string;
  deepLink?: string;
}

const STORE_ACCESS_POINTS: StoreAccessPoint[] = [
  {
    location: 'Main Menu',
    trigger: 'Store button in bottom navigation bar',
    icon: 'store_icon.png',
    prominence: 'Primary navigation'
  },
  {
    location: 'Character Preview',
    trigger: '"Browse More" button under current outfit',
    deepLink: '/store/outfits'
  },
  {
    location: 'Currency Display',
    trigger: 'Tap on currency balance in header',
    deepLink: '/store/currency'
  },
  {
    location: 'Battle Pass Screen',
    trigger: 'Embedded store link for pass purchase',
    deepLink: '/store/battlepass'
  },
  {
    location: 'Post-Match Screen',
    trigger: 'Featured item promotion card',
    deepLink: '/store/featured/{item_id}'
  },
  {
    location: 'Push Notification',
    trigger: 'New item or sale notification',
    deepLink: '/store/{section}'
  }
];
```

### Store Navigation Structure

```typescript
interface StoreNavigation {
  tabs: StoreTab[];
  defaultTab: string;
}

const STORE_TABS: StoreTab[] = [
  {
    id: 'featured',
    label: 'Featured',
    icon: 'star_icon.png',
    order: 1,
    badge: 'NEW',
    description: 'Daily/weekly rotating highlights'
  },
  {
    id: 'outfits',
    label: 'Outfits',
    icon: 'outfit_icon.png',
    order: 2,
    description: 'Character outfits and skins'
  },
  {
    id: 'emotes',
    label: 'Emotes',
    icon: 'emote_icon.png',
    order: 3,
    description: 'Emotes and expressions'
  },
  {
    id: 'bundles',
    label: 'Bundles',
    icon: 'bundle_icon.png',
    order: 4,
    badge: 'SALE',
    description: 'Value packs and collections'
  },
  {
    id: 'battlepass',
    label: 'Battle Pass',
    icon: 'battlepass_icon.png',
    order: 5,
    description: 'Season pass purchase'
  },
  {
    id: 'currency',
    label: 'Doubloons',
    icon: 'doubloon_icon.png',
    order: 6,
    description: 'Premium currency packs'
  }
];
```

---

## Store Layout Design

### Main Store Screen

```
┌─────────────────────────────────────────────────────────────┐
│  [<Back]        🏴‍☠️ STORE         [🪙 1,250]  [💎 450]      │
├─────────────────────────────────────────────────────────────┤
│  ┌────────┬────────┬────────┬────────┬────────┬────────┐   │
│  │Featured│ Outfits│ Emotes │ Bundles│  Pass  │Currency│   │
│  │   ⭐   │   👕   │   🎭   │   📦   │   🎫   │   💎   │   │
│  └────────┴────────┴────────┴────────┴────────┴────────┘   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │           FEATURED ITEM BANNER (LARGE)             │    │
│  │     [Legendary Kraken Captain Outfit - 1,200 💎]   │    │
│  │              ⏰ Ends in 23:45:12                    │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │   Daily Deal    │  │   Daily Deal    │                  │
│  │   [Item Image]  │  │   [Item Image]  │                  │
│  │   Storm Outfit  │  │   Dance Emote   │                  │
│  │   ~~800~~ 600💎 │  │   1,500 🪙      │                  │
│  │   25% OFF       │  │                 │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                             │
│  [Filter ▼] [Sort: Newest ▼] [Search 🔍]                   │
│                                                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │ [Image] │ │ [Image] │ │ [Image] │ │ [Image] │          │
│  │ Item 1  │ │ Item 2  │ │ Item 3  │ │ Item 4  │          │
│  │ 800 💎  │ │ 1,200🪙 │ │ OWNED ✓ │ │ 500 💎  │          │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
│                                                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │ [Image] │ │ [Image] │ │ [Image] │ │ [Image] │          │
│  │ Item 5  │ │ Item 6  │ │ Item 7  │ │ Item 8  │          │
│  │ 1,600🪙 │ │ 400 💎  │ │ 2,000🪙 │ │ 1,000💎 │          │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Header Component

```typescript
interface StoreHeader {
  elements: HeaderElement[];
}

const STORE_HEADER: StoreHeader = {
  elements: [
    {
      type: 'BackButton',
      position: 'left',
      action: 'Navigate to previous screen'
    },
    {
      type: 'Title',
      position: 'center',
      text: 'STORE',
      icon: 'pirate_flag.png'
    },
    {
      type: 'CurrencyDisplay',
      position: 'right',
      currencies: [
        { type: 'Gold', icon: 'gold_coin.png', tappable: true },
        { type: 'Doubloons', icon: 'doubloon.png', tappable: true }
      ],
      tapAction: 'Open currency purchase modal'
    }
  ]
};
```

---

## Category Sections

### Featured Section

```typescript
interface FeaturedSection {
  layout: 'hero_banner_plus_grid';
  components: FeaturedComponent[];
}

const FEATURED_LAYOUT: FeaturedSection = {
  components: [
    {
      type: 'HeroBanner',
      size: 'full_width',
      height: '200dp',
      content: {
        item: 'Weekly featured item',
        countdown: true,
        autoRotate: true,
        rotationInterval: 5000
      }
    },
    {
      type: 'DailyDealsRow',
      itemCount: 4,
      refreshTime: '00:00 UTC',
      countdown: true
    },
    {
      type: 'NewArrivalsGrid',
      columns: 4,
      maxItems: 8,
      sortBy: 'releaseDate',
      badgeNew: 'Items < 7 days old'
    },
    {
      type: 'EndingSoonSection',
      items: 'Limited time items with < 24h remaining',
      urgencyIndicator: true
    }
  ]
};
```

### Outfits Section

```typescript
interface OutfitSection {
  layout: 'filterable_grid';
  gridConfig: GridConfig;
  filters: FilterOption[];
  sorting: SortOption[];
}

const OUTFIT_SECTION: OutfitSection = {
  gridConfig: {
    columns: 4,
    itemSpacing: 8,
    thumbnailAspectRatio: '3:4',
    showPrice: true,
    showRarity: true,
    showOwnedBadge: true
  },
  filters: [
    {
      id: 'rarity',
      label: 'Rarity',
      options: ['Common', 'Rare', 'Epic', 'Legendary', 'Mythic']
    },
    {
      id: 'currency',
      label: 'Currency',
      options: ['Gold', 'Doubloons', 'Any']
    },
    {
      id: 'price_range',
      label: 'Price',
      type: 'range',
      options: ['Under 500', '500-1000', '1000-2000', '2000+']
    },
    {
      id: 'owned',
      label: 'Ownership',
      options: ['Show All', 'Not Owned', 'Owned']
    },
    {
      id: 'theme',
      label: 'Theme',
      options: ['Pirate', 'Naval', 'Mythical', 'Holiday', 'Seasonal']
    }
  ],
  sorting: [
    { id: 'newest', label: 'Newest First', default: true },
    { id: 'price_low', label: 'Price: Low to High' },
    { id: 'price_high', label: 'Price: High to Low' },
    { id: 'popularity', label: 'Most Popular' },
    { id: 'rarity', label: 'Rarity' }
  ]
};
```

### Emotes Section

```typescript
const EMOTE_SECTION: EmoteSection = {
  gridConfig: {
    columns: 4,
    itemSpacing: 8,
    thumbnailFormat: 'animated_preview',
    previewLoop: true,
    showPrice: true,
    showDuration: true
  },
  filters: [
    {
      id: 'type',
      label: 'Type',
      options: ['Dance', 'Taunt', 'Celebration', 'Communication', 'Action']
    },
    {
      id: 'rarity',
      label: 'Rarity',
      options: ['Basic', 'Standard', 'Complex', 'Epic', 'Legendary']
    },
    {
      id: 'currency',
      label: 'Currency',
      options: ['Gold', 'Doubloons', 'Any']
    },
    {
      id: 'owned',
      label: 'Ownership',
      options: ['Show All', 'Not Owned', 'Owned']
    }
  ],
  sorting: [
    { id: 'newest', label: 'Newest First', default: true },
    { id: 'price_low', label: 'Price: Low to High' },
    { id: 'price_high', label: 'Price: High to Low' },
    { id: 'popularity', label: 'Most Popular' }
  ]
};
```

### Bundles Section

```typescript
const BUNDLE_SECTION: BundleSection = {
  layout: 'card_list',
  cardConfig: {
    orientation: 'horizontal',
    height: '160dp',
    showContents: true,
    showSavings: true,
    showCountdown: true
  },
  categories: [
    {
      id: 'starter',
      label: 'Starter Packs',
      description: 'Best value for new players',
      eligibility: 'One-time purchase items'
    },
    {
      id: 'themed',
      label: 'Themed Sets',
      description: 'Matching outfit collections'
    },
    {
      id: 'event',
      label: 'Event Bundles',
      description: 'Limited-time seasonal items',
      showCountdown: true
    },
    {
      id: 'value',
      label: 'Value Packs',
      description: 'Currency + bonus items'
    }
  ]
};
```

### Battle Pass Section

```typescript
const BATTLEPASS_SECTION: BattlePassSection = {
  layout: 'single_product',
  display: {
    heroImage: 'Current season artwork',
    seasonName: 'Season {X}: {Theme}',
    countdown: 'Time remaining in season',
    progressPreview: 'Current tier / Total tiers'
  },
  purchaseOptions: [
    {
      id: 'standard_pass',
      name: 'Battle Pass',
      price: 950,
      currency: 'Doubloons',
      description: 'Unlock premium rewards track',
      features: [
        'Access 60+ premium rewards',
        'Exclusive season outfit',
        'Earn back 1000+ Doubloons'
      ]
    },
    {
      id: 'pass_bundle',
      name: 'Battle Pass Bundle',
      price: 1500,
      currency: 'Doubloons',
      description: 'Pass + instant boost',
      features: [
        'Everything in standard pass',
        '+10 instant tier skips',
        'Exclusive bundle frame'
      ],
      badge: 'BEST VALUE'
    }
  ],
  links: [
    { label: 'View All Rewards', action: 'OpenBattlePassRewardsPreview' },
    { label: 'FAQ', action: 'OpenBattlePassFAQ' }
  ]
};
```

### Currency Section

```typescript
const CURRENCY_SECTION: CurrencySection = {
  layout: 'iap_grid',
  products: [
    {
      id: 'doubloons_small',
      amount: 100,
      bonus: 0,
      displayPrice: '$0.99',
      icon: 'doubloons_small.png'
    },
    {
      id: 'doubloons_medium',
      amount: 550,
      bonus: 50,
      bonusPercent: '+10%',
      displayPrice: '$4.99',
      icon: 'doubloons_medium.png'
    },
    {
      id: 'doubloons_large',
      amount: 1200,
      bonus: 200,
      bonusPercent: '+20%',
      displayPrice: '$9.99',
      icon: 'doubloons_large.png',
      badge: 'POPULAR'
    },
    {
      id: 'doubloons_mega',
      amount: 2500,
      bonus: 500,
      bonusPercent: '+25%',
      displayPrice: '$19.99',
      icon: 'doubloons_mega.png'
    },
    {
      id: 'doubloons_ultra',
      amount: 6500,
      bonus: 1500,
      bonusPercent: '+30%',
      displayPrice: '$49.99',
      icon: 'doubloons_ultra.png',
      badge: 'BEST VALUE'
    }
  ],
  specialOffers: [
    {
      id: 'starter_pack',
      name: 'Starter Pack',
      displayPrice: '$4.99',
      contents: ['500 Doubloons', 'Starter Outfit', '2000 Gold'],
      oneTimePurchase: true,
      badge: 'ONE TIME'
    }
  ]
};
```

---

## Item Preview System

### 3D Model Viewer (Outfits)

```typescript
interface ModelViewer {
  config: ViewerConfig;
  controls: ViewerControl[];
}

const MODEL_VIEWER_CONFIG: ModelViewer = {
  config: {
    defaultZoom: 1.0,
    minZoom: 0.5,
    maxZoom: 2.0,
    autoRotate: true,
    autoRotateSpeed: 0.5,
    backgroundColor: 'gradient_dark_blue',
    lighting: 'studio_three_point',
    shadowEnabled: true
  },
  controls: [
    {
      gesture: 'Single finger drag',
      action: 'Rotate model horizontally'
    },
    {
      gesture: 'Two finger drag',
      action: 'Rotate model vertically (limited)'
    },
    {
      gesture: 'Pinch',
      action: 'Zoom in/out'
    },
    {
      gesture: 'Double tap',
      action: 'Reset to default view'
    },
    {
      gesture: 'Long press',
      action: 'Play idle animation'
    }
  ]
};
```

### Preview Modal Layout

```
┌─────────────────────────────────────────────────────────────┐
│  [✕ Close]                                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
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
│           🔄 Drag to rotate • 👆👆 Pinch to zoom            │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ⭐ LEGENDARY                                               │
│  Kraken Captain Outfit                                      │
│                                                             │
│  "Command the depths in this fearsome captain's garb,       │
│   adorned with kraken tentacle motifs."                     │
│                                                             │
│  Includes:                                                  │
│  • Kraken Captain Hat                                       │
│  • Tentacle Cape                                            │
│  • Captain's Coat                                           │
│  • Sea Monster Boots                                        │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              PURCHASE FOR 1,200 💎                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│              [Add to Wishlist ♡]                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Emote Preview

```typescript
interface EmotePreview {
  display: EmotePreviewConfig;
}

const EMOTE_PREVIEW: EmotePreview = {
  display: {
    characterModel: 'Player\'s current equipped outfit',
    animationLoop: true,
    loopDelay: 1000,
    soundEnabled: true,
    soundToggle: true,
    backgroundScene: 'Ship deck environment'
  },
  controls: [
    {
      button: 'Play',
      action: 'Play emote animation'
    },
    {
      button: 'Sound Toggle',
      action: 'Enable/disable emote sound effects'
    }
  ]
};
```

---

## Price Display System

### Price Component

```typescript
interface PriceDisplay {
  formats: PriceFormat[];
  states: PriceState[];
}

const PRICE_FORMATS: PriceFormat[] = [
  {
    type: 'Standard',
    format: '[Currency Icon] [Amount]',
    example: '💎 800',
    typography: {
      fontSize: 14,
      fontWeight: 'bold',
      color: 'currency_specific'
    }
  },
  {
    type: 'Sale',
    format: '[Currency Icon] ~~[Original]~~ [Sale Price]',
    example: '💎 ~~1,000~~ 800',
    typography: {
      originalPrice: { strikethrough: true, color: 'grey' },
      salePrice: { fontWeight: 'bold', color: 'sale_red' }
    },
    badge: {
      text: '20% OFF',
      backgroundColor: 'sale_red',
      position: 'corner'
    }
  },
  {
    type: 'Bundle Value',
    format: 'Value: [Total] | Price: [Discounted]',
    example: 'Value: 2,000 | Price: 1,500',
    savingsDisplay: '(Save 25%)'
  },
  {
    type: 'Insufficient Funds',
    format: '[Currency Icon] [Amount] (Need [X] more)',
    styling: {
      priceColor: 'red',
      insufficientText: 'grey_italic'
    },
    action: 'Tap to open currency purchase'
  }
];

const PRICE_STATES: PriceState[] = [
  {
    state: 'Available',
    display: 'Standard price',
    buttonEnabled: true
  },
  {
    state: 'Owned',
    display: 'OWNED ✓',
    buttonLabel: 'EQUIP',
    buttonColor: 'green'
  },
  {
    state: 'InsufficientFunds',
    display: 'Price in red + deficit amount',
    buttonLabel: 'GET CURRENCY',
    action: 'OpenCurrencyPurchase'
  },
  {
    state: 'OnSale',
    display: 'Strikethrough original + sale price',
    urgencyBadge: 'Time remaining'
  },
  {
    state: 'LimitedTime',
    display: 'Standard + countdown timer',
    urgencyBadge: '⏰ Ends in X:XX'
  }
];
```

### Currency Color Coding

```typescript
const CURRENCY_COLORS = {
  gold: {
    icon: 'gold_coin.png',
    primaryColor: '#FFD700',
    backgroundColor: '#FFF8DC',
    textColor: '#8B7500'
  },
  doubloons: {
    icon: 'doubloon.png',
    primaryColor: '#00CED1',
    backgroundColor: '#E0FFFF',
    textColor: '#008B8B'
  }
};
```

---

## Purchase Flow

### Purchase Confirmation Dialog

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    ⚓ CONFIRM PURCHASE ⚓                    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    [Item Thumbnail]                         │
│                                                             │
│                  Kraken Captain Outfit                      │
│                      ⭐ Legendary                           │
│                                                             │
│                                                             │
│                                                             │
│         Purchase for:     💎 1,200 Doubloons               │
│                                                             │
│         Your balance:     💎 1,450 Doubloons               │
│         After purchase:   💎   250 Doubloons               │
│                                                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────┐    ┌──────────────────┐            │
│   │     CANCEL       │    │    PURCHASE      │            │
│   └──────────────────┘    └──────────────────┘            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Purchase Flow States

```typescript
interface PurchaseFlow {
  steps: PurchaseStep[];
}

const PURCHASE_FLOW: PurchaseStep[] = [
  {
    step: 1,
    name: 'Initiate',
    trigger: 'Tap purchase button on item',
    validation: [
      'Check item not already owned',
      'Check sufficient balance',
      'Check item still available'
    ],
    onFail: {
      owned: 'Show "Already Owned" toast',
      insufficientFunds: 'Show currency purchase prompt',
      unavailable: 'Show "Item No Longer Available" dialog'
    }
  },
  {
    step: 2,
    name: 'Confirmation',
    display: 'Purchase confirmation dialog',
    showDetails: [
      'Item name and image',
      'Price',
      'Current balance',
      'Balance after purchase'
    ],
    actions: ['Cancel', 'Confirm']
  },
  {
    step: 3,
    name: 'Processing',
    display: 'Loading spinner with "Processing..."',
    timeout: 10000,
    backendAction: 'Server-side transaction',
    lockUI: true
  },
  {
    step: 4,
    name: 'Success',
    display: 'Success celebration animation',
    content: {
      title: 'Purchase Complete!',
      subtitle: 'Item added to your collection',
      animation: 'confetti_burst',
      sound: 'purchase_success.wav'
    },
    actions: [
      { label: 'EQUIP NOW', action: 'EquipItem', primary: true },
      { label: 'Continue Shopping', action: 'CloseModal' }
    ]
  },
  {
    step: 5,
    name: 'Failure',
    display: 'Error dialog',
    possibleErrors: [
      {
        code: 'INSUFFICIENT_FUNDS',
        message: 'Not enough currency',
        action: 'Offer currency purchase'
      },
      {
        code: 'ITEM_UNAVAILABLE',
        message: 'This item is no longer available',
        action: 'Return to store'
      },
      {
        code: 'NETWORK_ERROR',
        message: 'Connection error. Please try again.',
        action: 'Retry button'
      },
      {
        code: 'ALREADY_OWNED',
        message: 'You already own this item',
        action: 'Return to store'
      },
      {
        code: 'SERVER_ERROR',
        message: 'Something went wrong. Please try again later.',
        action: 'Contact support link'
      }
    ]
  }
];
```

### Success Animation

```typescript
interface SuccessAnimation {
  sequence: AnimationStep[];
}

const SUCCESS_ANIMATION: SuccessAnimation = {
  sequence: [
    {
      step: 1,
      duration: 300,
      animation: 'Modal scales up from center',
      easing: 'easeOutBack'
    },
    {
      step: 2,
      duration: 500,
      animation: 'Item thumbnail glows and pulses',
      particles: 'gold_sparkles'
    },
    {
      step: 3,
      duration: 800,
      animation: 'Confetti burst from top',
      sound: 'celebration_fanfare.wav'
    },
    {
      step: 4,
      duration: 300,
      animation: '"Purchase Complete!" text fades in',
      typography: 'bold_gold'
    },
    {
      step: 5,
      duration: 200,
      animation: 'Action buttons slide up',
      interactive: true
    }
  ],
  totalDuration: 2100
};
```

---

## Owned Items System

### Owned Indicator

```typescript
interface OwnedIndicator {
  gridDisplay: GridOwnedConfig;
  previewDisplay: PreviewOwnedConfig;
}

const OWNED_INDICATOR: OwnedIndicator = {
  gridDisplay: {
    badge: {
      text: 'OWNED',
      icon: 'checkmark.png',
      backgroundColor: 'rgba(0, 200, 0, 0.8)',
      position: 'bottom_overlay'
    },
    thumbnailStyle: {
      opacity: 0.7,
      borderColor: 'green',
      borderWidth: 2
    }
  },
  previewDisplay: {
    priceReplacement: {
      text: 'OWNED',
      icon: 'checkmark_circle.png',
      color: 'green'
    },
    purchaseButton: {
      label: 'EQUIP',
      color: 'green',
      action: 'EquipItem'
    },
    alternativeAction: {
      label: 'View in Locker',
      action: 'OpenInventory'
    }
  }
};
```

### Duplicate Prevention

```typescript
interface DuplicatePrevention {
  checks: PreventionCheck[];
}

const DUPLICATE_PREVENTION: DuplicatePrevention = {
  checks: [
    {
      checkPoint: 'Grid Display',
      action: 'Mark owned items visually',
      implementation: 'Frontend ownership check on render'
    },
    {
      checkPoint: 'Item Preview',
      action: 'Replace purchase with equip',
      implementation: 'Check ownership before showing purchase option'
    },
    {
      checkPoint: 'Purchase Button Tap',
      action: 'Validate not owned',
      implementation: 'Client-side check before API call'
    },
    {
      checkPoint: 'Server Validation',
      action: 'Reject duplicate purchase',
      implementation: 'Server checks ownership before processing',
      response: { code: 'ALREADY_OWNED', refund: false }
    }
  ]
};
```

---

## Filter and Sort System

### Filter UI

```
┌─────────────────────────────────────────────────────────────┐
│                       FILTER OPTIONS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Rarity                                                     │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐   │
│  │ Common │ │  Rare  │ │  Epic  │ │Legend. │ │ Mythic │   │
│  │   ○    │ │   ●    │ │   ●    │ │   ○    │ │   ○    │   │
│  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘   │
│                                                             │
│  Currency                                                   │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│  │   🪙 Gold    │ │ 💎 Doubloons │ │     Any      │        │
│  │      ●       │ │      ○       │ │      ○       │        │
│  └──────────────┘ └──────────────┘ └──────────────┘        │
│                                                             │
│  Price Range                                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │Under 500 │ │ 500-1000 │ │1000-2000 │ │  2000+   │       │
│  │    ●     │ │    ○     │ │    ○     │ │    ○     │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│                                                             │
│  Show Owned                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│  │   Show All   │ │  Not Owned   │ │    Owned     │        │
│  │      ○       │ │      ●       │ │      ○       │        │
│  └──────────────┘ └──────────────┘ └──────────────┘        │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │  RESET FILTERS  │    │  APPLY (24)     │                │
│  └─────────────────┘    └─────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

### Filter Implementation

```typescript
interface FilterSystem {
  config: FilterConfig;
  behavior: FilterBehavior;
}

const FILTER_SYSTEM: FilterSystem = {
  config: {
    displayMode: 'modal', // or 'inline_expandable'
    triggerButton: {
      label: 'Filter',
      icon: 'filter_icon.png',
      badge: 'Active filter count when > 0'
    },
    persistence: 'Session only, reset on app close',
    defaultFilters: {
      rarity: 'all',
      currency: 'all',
      priceRange: 'all',
      owned: 'show_all'
    }
  },
  behavior: {
    multiSelect: {
      rarity: true, // Can select multiple rarities
      currency: false, // Single select
      priceRange: true,
      owned: false
    },
    instantApply: false, // Wait for "Apply" button
    resultCount: 'Show matching item count on Apply button',
    emptyState: {
      message: 'No items match your filters',
      action: 'Reset Filters'
    }
  }
};
```

### Sort Implementation

```typescript
interface SortSystem {
  options: SortOption[];
  ui: SortUI;
}

const SORT_SYSTEM: SortSystem = {
  options: [
    {
      id: 'newest',
      label: 'Newest First',
      field: 'releaseDate',
      direction: 'desc',
      default: true
    },
    {
      id: 'price_low',
      label: 'Price: Low to High',
      field: 'price',
      direction: 'asc',
      note: 'Gold items first, then Doubloons'
    },
    {
      id: 'price_high',
      label: 'Price: High to Low',
      field: 'price',
      direction: 'desc'
    },
    {
      id: 'popularity',
      label: 'Most Popular',
      field: 'purchaseCount',
      direction: 'desc'
    },
    {
      id: 'rarity_high',
      label: 'Rarity: Highest First',
      field: 'rarityTier',
      direction: 'desc'
    },
    {
      id: 'alphabetical',
      label: 'A-Z',
      field: 'name',
      direction: 'asc'
    }
  ],
  ui: {
    displayMode: 'dropdown',
    showCurrentSort: true,
    instantApply: true
  }
};
```

---

## Search Functionality

### Search UI

```typescript
interface SearchConfig {
  ui: SearchUI;
  behavior: SearchBehavior;
}

const SEARCH_CONFIG: SearchConfig = {
  ui: {
    placement: 'Below category tabs',
    expandable: true, // Collapsed to icon by default
    placeholder: 'Search outfits, emotes...',
    clearButton: true,
    voiceSearch: false
  },
  behavior: {
    minCharacters: 2,
    debounceMs: 300,
    searchFields: ['name', 'description', 'tags'],
    highlighting: true,
    recentSearches: {
      enabled: true,
      maxItems: 5,
      clearOption: true
    }
  }
};
```

### Search Results

```typescript
interface SearchResults {
  display: SearchResultsDisplay;
}

const SEARCH_RESULTS: SearchResults = {
  display: {
    layout: 'Grid (same as category view)',
    resultCount: 'Show "X results for [query]"',
    categorization: 'Group by category if across multiple',
    noResults: {
      message: 'No items found for "[query]"',
      suggestions: [
        'Check your spelling',
        'Try different keywords',
        'Browse categories instead'
      ]
    }
  }
};
```

---

## Loading States

### Skeleton Loading

```typescript
interface LoadingStates {
  grid: GridLoadingState;
  preview: PreviewLoadingState;
  purchase: PurchaseLoadingState;
}

const LOADING_STATES: LoadingStates = {
  grid: {
    type: 'Skeleton cards',
    count: 8,
    animation: 'shimmer',
    duration: 'Until data loads or 10s timeout'
  },
  preview: {
    modelLoading: {
      placeholder: 'Silhouette with loading spinner',
      fallback: '2D image after 5s'
    },
    detailsLoading: {
      placeholder: 'Skeleton text blocks',
      animation: 'pulse'
    }
  },
  purchase: {
    processing: {
      overlay: 'Semi-transparent dark overlay',
      spinner: 'Pirate ship wheel spinner',
      text: 'Processing purchase...',
      cancelable: false
    }
  }
};
```

---

## Error Handling

### Error States

```typescript
interface ErrorHandling {
  scenarios: ErrorScenario[];
}

const ERROR_HANDLING: ErrorHandling = {
  scenarios: [
    {
      error: 'Network Error',
      detection: 'API timeout or connection failure',
      display: 'Inline error message with retry',
      recovery: 'Retry button, refresh pull-down'
    },
    {
      error: 'Item Unavailable',
      detection: 'Item removed from store during session',
      display: 'Toast notification',
      recovery: 'Auto-refresh store catalog'
    },
    {
      error: 'Purchase Failed',
      detection: 'Transaction rejected by server',
      display: 'Error dialog with reason',
      recovery: 'Return to store, contact support link'
    },
    {
      error: 'Sync Error',
      detection: 'Inventory mismatch',
      display: 'Brief resync message',
      recovery: 'Auto-sync in background'
    }
  ]
};
```

---

## Performance Optimization

### Loading Strategy

```typescript
interface PerformanceConfig {
  loading: LoadingStrategy;
  caching: CacheStrategy;
}

const PERFORMANCE_CONFIG: PerformanceConfig = {
  loading: {
    initialLoad: 'First 16 items (4x4 grid)',
    lazyLoad: 'Load more on scroll (8 items per batch)',
    imageQuality: {
      thumbnail: 'Low-res (128x128)',
      preview: 'Medium-res on demand',
      fullModel: 'Stream on preview open'
    },
    prefetch: 'Prefetch likely next items'
  },
  caching: {
    catalogCache: '5 minutes',
    imageCache: '24 hours',
    priceCache: '1 minute (for sales)',
    ownershipCache: 'Refresh on store open'
  }
};
```

---

## Accessibility

### Accessibility Features

```typescript
interface AccessibilityConfig {
  features: AccessibilityFeature[];
}

const ACCESSIBILITY: AccessibilityConfig = {
  features: [
    {
      feature: 'Screen Reader',
      implementation: 'All items have descriptive labels',
      example: '"Kraken Captain Outfit, Legendary rarity, 1200 Doubloons"'
    },
    {
      feature: 'Touch Targets',
      implementation: 'Minimum 44x44 dp for all interactive elements'
    },
    {
      feature: 'Color Contrast',
      implementation: 'WCAG AA compliant for text on backgrounds'
    },
    {
      feature: 'Reduce Motion',
      implementation: 'Respect system preference, disable auto-rotate'
    },
    {
      feature: 'Text Scaling',
      implementation: 'Support up to 200% text size'
    }
  ]
};
```

---

## Implementation Checklist

### Store Access
- [ ] Store button in main menu navigation
- [ ] Deep links from other screens
- [ ] Push notification deep linking

### Categories
- [ ] Featured section with hero banner
- [ ] Outfits section with grid
- [ ] Emotes section with animated previews
- [ ] Bundles section with value display
- [ ] Battle Pass purchase section
- [ ] Currency purchase section

### Item Preview
- [ ] 3D model viewer for outfits
- [ ] Touch controls (rotate, zoom)
- [ ] Emote animation preview
- [ ] Item details display

### Price Display
- [ ] Standard price format
- [ ] Sale price with strikethrough
- [ ] Bundle value display
- [ ] Insufficient funds indicator
- [ ] Owned item indicator

### Purchase Flow
- [ ] Purchase confirmation dialog
- [ ] Processing state
- [ ] Success celebration
- [ ] Error handling
- [ ] Equip after purchase option

### Filter/Sort
- [ ] Filter by rarity
- [ ] Filter by currency type
- [ ] Filter by price range
- [ ] Filter by ownership
- [ ] Sort options (newest, price, popularity)
- [ ] Search functionality

### Performance
- [ ] Lazy loading for grid
- [ ] Image caching
- [ ] Skeleton loading states

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-02 | Game Design Team | Initial specification |
