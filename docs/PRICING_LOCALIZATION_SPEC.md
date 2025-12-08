# Pricing Localization Specification

## Document Information
- **Task ID:** MON-020
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-08

---

## Overview

This document specifies the pricing localization strategy for Plunderstorm Mobile, ensuring region-appropriate pricing for in-app purchases that maximizes revenue while maintaining fair value perception across global markets.

---

## Regional Pricing Strategy

### Market Tiers

```typescript
const MARKET_TIERS = {
  tier1: {
    name: 'Premium Markets',
    countries: ['US', 'CA', 'UK', 'AU', 'DE', 'FR', 'JP', 'KR', 'CH', 'NO', 'SE'],
    characteristics: {
      gdpPerCapita: 'High ($40,000+)',
      mobileSpend: 'High',
      paymentMethods: 'Credit cards dominant'
    },
    pricingStrategy: 'Full price (baseline)',
    priceMultiplier: 1.0
  },

  tier2: {
    name: 'Growth Markets',
    countries: ['ES', 'IT', 'PL', 'TW', 'HK', 'SG', 'NZ', 'IE', 'BE', 'AT'],
    characteristics: {
      gdpPerCapita: 'Medium-High ($20,000-$40,000)',
      mobileSpend: 'Medium-High',
      paymentMethods: 'Mixed'
    },
    pricingStrategy: 'Slight discount',
    priceMultiplier: 0.85
  },

  tier3: {
    name: 'Emerging Markets',
    countries: ['BR', 'MX', 'AR', 'CL', 'CO', 'RU', 'TR', 'ZA', 'MY', 'TH'],
    characteristics: {
      gdpPerCapita: 'Medium ($8,000-$20,000)',
      mobileSpend: 'Medium',
      paymentMethods: 'Local methods important'
    },
    pricingStrategy: 'Moderate discount',
    priceMultiplier: 0.65
  },

  tier4: {
    name: 'Value Markets',
    countries: ['IN', 'ID', 'PH', 'VN', 'EG', 'NG', 'PK', 'BD'],
    characteristics: {
      gdpPerCapita: 'Lower ($1,000-$8,000)',
      mobileSpend: 'Price sensitive',
      paymentMethods: 'Carrier billing, local wallets'
    },
    pricingStrategy: 'Significant discount',
    priceMultiplier: 0.40
  },

  tier5: {
    name: 'Emerging Value Markets',
    countries: ['Other developing markets'],
    characteristics: {
      gdpPerCapita: 'Low',
      mobileSpend: 'Very price sensitive',
      paymentMethods: 'Limited options'
    },
    pricingStrategy: 'Maximum discount',
    priceMultiplier: 0.30
  }
};
```

### Base Price Matrix

```typescript
const BASE_PRICES_USD = {
  // Consumables
  smallGemPack: { amount: 100, basePrice: 0.99 },
  mediumGemPack: { amount: 550, basePrice: 4.99 },
  largeGemPack: { amount: 1200, basePrice: 9.99 },
  megaGemPack: { amount: 2500, basePrice: 19.99 },
  ultraGemPack: { amount: 6500, basePrice: 49.99 },
  maxGemPack: { amount: 14000, basePrice: 99.99 },

  // Battle Pass
  battlePass: { basePrice: 9.99 },
  battlePassBundle: { basePrice: 24.99 }, // Pass + 25 tier skips

  // Starter Packs (one-time)
  starterPack: { basePrice: 4.99 },
  advancedStarterPack: { basePrice: 9.99 },
  eliteStarterPack: { basePrice: 19.99 },

  // Subscriptions
  monthlyPass: { basePrice: 4.99 },
  premiumMonthlyPass: { basePrice: 9.99 }
};
```

---

## Regional Price Calculation

### Price Conversion

```typescript
const PRICE_CALCULATION = {
  formula: {
    description: 'Base USD price × Tier multiplier × Local adjustments',
    steps: [
      'Start with base USD price',
      'Apply tier multiplier',
      'Convert to local currency',
      'Round to price point',
      'Apply store-specific adjustments'
    ]
  },

  localCurrency: {
    conversion: 'Use store exchange rates (not live FX)',
    update: 'Review quarterly or on major FX moves',
    rounding: 'Round to local price points'
  },

  pricePoints: {
    description: 'Psychologically optimal price endings',
    usd: [0.99, 1.99, 2.99, 4.99, 9.99, 14.99, 19.99, 24.99, 49.99, 99.99],
    eur: [0.99, 1.99, 2.99, 4.99, 9.99, 14.99, 19.99, 24.99, 49.99, 99.99],
    gbp: [0.99, 1.99, 2.99, 4.99, 7.99, 12.99, 17.99, 22.99, 44.99, 89.99],
    jpy: [120, 250, 370, 610, 1220, 1840, 2440, 3060, 6100, 12200],
    inr: [79, 159, 249, 399, 799, 1199, 1599, 1999, 3999, 7999],
    brl: [4.90, 9.90, 14.90, 27.90, 54.90, 84.90, 109.90, 139.90, 279.90, 549.90]
  }
};

const calculateLocalPrice = (
  baseUSD: number,
  country: string,
  currency: string
): LocalPrice => {
  // Get tier for country
  const tier = getMarketTier(country);

  // Apply tier multiplier
  const adjustedUSD = baseUSD * tier.priceMultiplier;

  // Convert to local currency
  const exchangeRate = getStoreExchangeRate(currency);
  const localAmount = adjustedUSD * exchangeRate;

  // Round to nearest price point
  const pricePoint = findNearestPricePoint(localAmount, currency);

  return {
    currency: currency,
    amount: pricePoint,
    displayPrice: formatPrice(pricePoint, currency)
  };
};
```

### Example Pricing Table

```typescript
const EXAMPLE_PRICING = {
  mediumGemPack: {
    baseUSD: 4.99,
    byRegion: {
      US: { currency: 'USD', price: 4.99, display: '$4.99' },
      UK: { currency: 'GBP', price: 4.99, display: '£4.99' },
      EU: { currency: 'EUR', price: 4.99, display: '€4.99' },
      JP: { currency: 'JPY', price: 610, display: '¥610' },
      BR: { currency: 'BRL', price: 14.90, display: 'R$14,90' },
      IN: { currency: 'INR', price: 159, display: '₹159' },
      ID: { currency: 'IDR', price: 29000, display: 'Rp29.000' },
      MX: { currency: 'MXN', price: 49.00, display: '$49.00 MXN' },
      RU: { currency: 'RUB', price: 199, display: '199 ₽' },
      KR: { currency: 'KRW', price: 5500, display: '₩5,500' }
    }
  }
};
```

---

## Store Configuration

### Apple App Store

```typescript
const APP_STORE_CONFIG = {
  priceTiers: {
    description: 'Apple uses fixed price tiers',
    configuration: 'Select tier per product per region',
    flexibility: 'Limited to Apple tier options'
  },

  territories: {
    supported: '175 countries/regions',
    currencies: '45 currencies',
    configuration: 'App Store Connect > Pricing'
  },

  setup: {
    steps: [
      'Create IAP products in App Store Connect',
      'For each product, go to Pricing and Availability',
      'Set base price tier',
      'Customize per-territory if needed',
      'Save and submit for review'
    ]
  },

  considerations: {
    taxes: 'Apple handles tax collection in most regions',
    payout: 'Net revenue after 30% commission and taxes',
    updates: 'Price changes take 24 hours to propagate'
  }
};
```

### Google Play Store

```typescript
const PLAY_STORE_CONFIG = {
  pricing: {
    description: 'Google allows custom pricing per country',
    flexibility: 'More granular than Apple',
    configuration: 'Play Console > Monetize > Products'
  },

  currencies: {
    supported: '130+ currencies',
    autoConvert: 'Google can auto-convert from base price',
    override: 'Manual override per country recommended'
  },

  setup: {
    steps: [
      'Create in-app products in Play Console',
      'Set default price',
      'Use "Manage prices" to set per-country prices',
      'Enable/disable countries as needed',
      'Publish changes'
    ]
  },

  considerations: {
    taxes: 'Google handles tax in most regions',
    localPayments: 'Supports carrier billing in select regions',
    payout: 'Net revenue after 15-30% commission'
  }
};
```

---

## Currency Display

### Localization Rules

```typescript
const CURRENCY_DISPLAY = {
  format: {
    symbolPosition: {
      before: ['USD', 'GBP', 'EUR', 'JPY', 'CNY', 'KRW'],
      after: ['SEK', 'NOK', 'DKK', 'PLN', 'CZK']
    },
    decimalSeparator: {
      period: ['US', 'UK', 'JP', 'CN', 'KR', 'AU'],
      comma: ['DE', 'FR', 'ES', 'IT', 'BR', 'RU']
    },
    thousandsSeparator: {
      comma: ['US', 'UK', 'JP', 'AU'],
      period: ['DE', 'ES', 'IT', 'BR'],
      space: ['FR', 'RU', 'PL']
    }
  },

  display: {
    inStore: 'Show local currency and price',
    inGame: 'Show gems/gold (virtual currency)',
    receipts: 'Show local currency paid'
  },

  examples: {
    US: '$9.99',
    UK: '£7.99',
    DE: '9,99 €',
    FR: '9,99 €',
    JP: '¥1,220',
    BR: 'R$ 27,90',
    IN: '₹399',
    RU: '499 ₽'
  }
};
```

---

## Regulatory Compliance

### Regional Requirements

```typescript
const REGULATORY_COMPLIANCE = {
  eu: {
    requirements: [
      'Prices must include VAT',
      'Clear display of total price',
      'Consumer protection disclosures',
      'Right of withdrawal information'
    ],
    vatHandling: 'Stores handle VAT collection',
    disclosure: 'Price includes all taxes'
  },

  japan: {
    requirements: [
      'Specific kompu gacha regulations',
      'Probability disclosure for random items',
      'Clear pricing in JPY',
      'Age rating compliance'
    ],
    disclosure: 'Gacha rates must be displayed'
  },

  china: {
    requirements: [
      'Real-name registration',
      'Spending limits for minors',
      'Government approval required',
      'CNY pricing only'
    ],
    note: 'Special version and publisher required'
  },

  korea: {
    requirements: [
      'Probability disclosure mandatory',
      'Refund policies',
      'Clear KRW pricing',
      'Age verification'
    ],
    disclosure: 'Gacha rates in game and store'
  },

  brazil: {
    requirements: [
      'BRL pricing required',
      'Consumer protection laws',
      'Clear refund policy',
      'Local entity may be required'
    ],
    taxes: 'Complex tax structure'
  },

  india: {
    requirements: [
      'INR pricing',
      'GST compliance',
      'Local payment methods support',
      'Data localization considerations'
    ],
    payments: 'UPI integration valuable'
  },

  general: {
    ageRatings: 'Comply with ESRB, PEGI, CERO, etc.',
    gambling: 'Avoid mechanics classified as gambling',
    disclosure: 'Clear about in-app purchases',
    minors: 'Parental controls and limits'
  }
};
```

---

## Value Perception

### Purchasing Power Parity

```typescript
const VALUE_PERCEPTION = {
  principle: 'Same perceived value across regions',

  approach: {
    description: 'Price should feel similar relative to local income',
    example: '$4.99 in US should feel like ₹159 in India',
    adjustment: 'Beyond pure exchange rate conversion'
  },

  gemValue: {
    consistent: 'Gems purchase same items globally',
    note: 'Lower regional prices = same gem amounts',
    fairness: 'No gameplay advantage by region'
  },

  bonusValue: {
    regional: 'Consider regional bonus amounts',
    example: 'First purchase bonus may vary by region',
    purpose: 'Drive conversion in price-sensitive markets'
  }
};
```

### Price Testing

```typescript
const PRICE_TESTING = {
  abTesting: {
    enabled: true,
    framework: 'Uses BACK-022 A/B Testing',
    metrics: ['Conversion rate', 'Revenue', 'ARPDAU']
  },

  testTypes: {
    pricePoints: {
      description: 'Test different price points per region',
      example: 'Test $4.99 vs $3.99 for gem pack in BR'
    },
    bundleComposition: {
      description: 'Test different bundle values',
      example: 'Test 500 gems vs 550 gems at same price'
    },
    discountDepth: {
      description: 'Test sale discount percentages',
      example: 'Test 20% vs 30% off in specific markets'
    }
  },

  constraints: {
    frequency: 'Max 1 active price test per region',
    duration: 'Minimum 2 weeks for significance',
    rollback: 'Easy revert if negative impact'
  }
};
```

---

## Implementation

### Price Configuration System

```typescript
const PRICE_SYSTEM = {
  storage: {
    source: 'Backend price configuration service',
    cache: 'Client caches prices on app start',
    refresh: 'Daily refresh or on store change'
  },

  structure: {
    product: {
      id: 'string',
      basePrice: 'number (USD)',
      regionalPrices: 'Map<country, LocalPrice>',
      storeProductIds: {
        ios: 'string',
        android: 'string'
      }
    }
  },

  fallback: {
    unknownRegion: 'Use Tier 3 pricing',
    missingPrice: 'Calculate from base USD',
    storeError: 'Fetch from store directly'
  }
};
```

### Validation

```typescript
const PRICE_VALIDATION = {
  preSubmit: [
    'Verify all regions have prices set',
    'Check price points are valid for stores',
    'Confirm tier multipliers applied correctly',
    'Validate currency formatting'
  ],

  postLaunch: [
    'Monitor conversion by region',
    'Compare to expected revenue',
    'Check for arbitrage opportunities',
    'Verify store prices match config'
  ],

  alerts: {
    conversionDrop: 'Alert if region conversion drops > 20%',
    revenueAnomaly: 'Alert if region revenue unexpected',
    exchangeRate: 'Alert if FX moves > 10%'
  }
};
```

---

## Testing Requirements

```typescript
const PRICING_TESTING = {
  functional: [
    'Correct prices display per region',
    'Currency symbols and formatting correct',
    'Store purchases process at listed price',
    'Gems credited match purchase'
  ],

  regional: [
    'Test in each tier representative country',
    'Verify local currency display',
    'Confirm store integration works',
    'Check receipt shows correct amount'
  ],

  edge: [
    'VPN/region mismatch handling',
    'Currency change mid-session',
    'Store unavailable fallback',
    'Price update during session'
  ],

  compliance: [
    'VAT displayed correctly (EU)',
    'Probability disclosures visible (JP, KR)',
    'Age gating functions (where required)'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-08 | Development Team | Initial pricing localization specification |
