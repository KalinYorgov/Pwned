# MON-007: Cosmetic Inventory System Specification

## Overview

This document specifies the cosmetic inventory system for Plunderstorm Mobile, managing player-owned items, equip functionality, and cosmetic preview features.

## Design Philosophy

The inventory system should make players feel ownership and pride in their collection while providing easy access to equip items. The interface must support a growing catalog of cosmetics without becoming overwhelming, using intuitive organization and filtering.

---

## 1. Cosmetic Item Types

### 1.1 Item Categories
| Category | Description | Equipped Slots |
|----------|-------------|----------------|
| Outfits | Full character skins | 1 slot |
| Headwear | Hats, helmets, accessories | 1 slot |
| Backpack | Back-worn items, capes | 1 slot |
| Emotes | Victory poses, gestures | 4 slots |
| Sprays | In-game graffiti tags | 4 slots |
| Profile Icons | Account avatar | 1 slot |
| Profile Banners | Profile background | 1 slot |
| Kill Effects | Elimination animations | 1 slot |
| Trails | Movement particle effects | 1 slot |

### 1.2 Item Rarity Tiers
| Rarity | Color | Gold Price | Doubloon Price |
|--------|-------|------------|----------------|
| Common | Gray | 500-1,000 | - |
| Uncommon | Green | 1,500-2,500 | 200-400 |
| Rare | Blue | 3,000-5,000 | 500-800 |
| Epic | Purple | - | 1,000-1,500 |
| Legendary | Orange | - | 1,800-2,500 |
| Mythic | Red | - | Event/Exclusive |

### 1.3 Item Sources
```
Acquisition Methods:
├── Shop Purchase (Gold/Doubloons)
├── Battle Pass Reward
├── Quest Reward
├── Achievement Unlock
├── Event Reward
├── Promotional Code
├── Starter Pack
└── Gift/Transfer (future)
```

---

## 2. Data Model

### 2.1 Item Definition
```json
{
  "itemId": "outfit_pirate_captain_01",
  "name": "Pirate Captain",
  "description": "Command the seven seas in style",
  "category": "outfit",
  "rarity": "epic",
  "releaseDate": "2025-01-15",
  "prices": {
    "gold": null,
    "doubloons": 1200
  },
  "assets": {
    "icon": "items/outfits/pirate_captain_icon.png",
    "model": "items/outfits/pirate_captain.fbx",
    "preview": "items/outfits/pirate_captain_preview.mp4"
  },
  "tags": ["pirate", "captain", "hat", "classic"],
  "isExclusive": false,
  "isLimited": false,
  "availableUntil": null
}
```

### 2.2 Player Inventory Record
```json
{
  "playerId": "uuid",
  "items": [
    {
      "itemId": "outfit_pirate_captain_01",
      "acquiredAt": "2025-02-10T14:30:00Z",
      "source": "shop_purchase",
      "sourceId": "transaction-uuid",
      "isFavorite": true
    },
    {
      "itemId": "emote_wave_01",
      "acquiredAt": "2025-01-01T00:00:00Z",
      "source": "default",
      "sourceId": null,
      "isFavorite": false
    }
  ],
  "equipped": {
    "outfit": "outfit_pirate_captain_01",
    "headwear": null,
    "backpack": null,
    "emotes": ["emote_wave_01", "emote_dance_01", null, null],
    "sprays": ["spray_skull_01", null, null, null],
    "profileIcon": "icon_default_01",
    "profileBanner": "banner_default_01",
    "killEffect": null,
    "trail": null
  },
  "updatedAt": "2025-12-02T14:30:00Z"
}
```

### 2.3 Database Schema
```sql
-- Item catalog (server-side)
CREATE TABLE cosmetic_items (
  item_id VARCHAR(100) PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  description TEXT,
  category VARCHAR(50) NOT NULL,
  rarity VARCHAR(20) NOT NULL,
  gold_price INT,
  doubloon_price INT,
  is_exclusive BOOLEAN DEFAULT FALSE,
  is_limited BOOLEAN DEFAULT FALSE,
  available_until TIMESTAMP,
  release_date DATE,
  tags VARCHAR[] DEFAULT '{}',
  assets JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Player inventory
CREATE TABLE player_inventory (
  player_id UUID REFERENCES players(player_id),
  item_id VARCHAR(100) REFERENCES cosmetic_items(item_id),
  acquired_at TIMESTAMP DEFAULT NOW(),
  source VARCHAR(50) NOT NULL,
  source_id VARCHAR(100),
  is_favorite BOOLEAN DEFAULT FALSE,
  PRIMARY KEY (player_id, item_id)
);

-- Player equipped items
CREATE TABLE player_equipped (
  player_id UUID PRIMARY KEY,
  outfit VARCHAR(100),
  headwear VARCHAR(100),
  backpack VARCHAR(100),
  emote_1 VARCHAR(100),
  emote_2 VARCHAR(100),
  emote_3 VARCHAR(100),
  emote_4 VARCHAR(100),
  spray_1 VARCHAR(100),
  spray_2 VARCHAR(100),
  spray_3 VARCHAR(100),
  spray_4 VARCHAR(100),
  profile_icon VARCHAR(100),
  profile_banner VARCHAR(100),
  kill_effect VARCHAR(100),
  trail VARCHAR(100),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_inventory_player ON player_inventory(player_id);
CREATE INDEX idx_items_category ON cosmetic_items(category);
CREATE INDEX idx_items_rarity ON cosmetic_items(rarity);
```

---

## 3. Inventory UI

### 3.1 Main Inventory Screen
```
┌──────────────────────────────────────────────────────────────┐
│ ←  My Inventory                      🔍 [Search]  ⚙️ [Filter] │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ [All] [Outfits] [Emotes] [Sprays] [Profile] [More...]        │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐               │
│   │ ★✓  │  │  ✓  │  │     │  │     │  │     │               │
│   │[IMG]│  │[IMG]│  │[IMG]│  │[IMG]│  │[IMG]│               │
│   │     │  │     │  │     │  │     │  │     │               │
│   └─────┘  └─────┘  └─────┘  └─────┘  └─────┘               │
│   Pirate   Classic  Ninja   Knight   Samurai                 │
│   Captain  Outfit   Garb    Armor    Set                     │
│                                                              │
│   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐               │
│   │     │  │     │  │     │  │     │  │     │               │
│   │[IMG]│  │[IMG]│  │[IMG]│  │[IMG]│  │[IMG]│               │
│   │     │  │     │  │     │  │     │  │     │               │
│   └─────┘  └─────┘  └─────┘  └─────┘  └─────┘               │
│                                                              │
│                      [Load More...]                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Legend:
★ = Favorited
✓ = Currently Equipped
```

### 3.2 Category Tabs
| Tab | Contents |
|-----|----------|
| All | All owned items |
| Outfits | Outfits + Headwear + Backpack |
| Emotes | Emotes (victory, gesture) |
| Sprays | Spray tags |
| Profile | Icons + Banners |
| More | Kill Effects, Trails, etc. |

### 3.3 Filter Options
```
Filter Menu:
├── Rarity
│   ├── [ ] Common
│   ├── [ ] Uncommon
│   ├── [ ] Rare
│   ├── [ ] Epic
│   ├── [ ] Legendary
│   └── [ ] Mythic
│
├── Source
│   ├── [ ] Shop
│   ├── [ ] Battle Pass
│   ├── [ ] Events
│   └── [ ] Achievements
│
├── Status
│   ├── [ ] Equipped
│   ├── [ ] Favorites
│   └── [ ] New (unviewed)
│
└── [Apply] [Clear]
```

### 3.4 Sort Options
```
Sort By:
├── Newest First (default)
├── Oldest First
├── Rarity (High to Low)
├── Rarity (Low to High)
├── Name (A-Z)
├── Name (Z-A)
├── Recently Equipped
└── Favorites First
```

---

## 4. Item Detail View

### 4.1 Detail Screen Layout
```
┌──────────────────────────────────────────────────────────────┐
│ ←                    Pirate Captain                     [★]  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                   ┌─────────────────────┐                    │
│                   │                     │                    │
│                   │                     │                    │
│                   │   3D Model Viewer   │                    │
│                   │                     │                    │
│                   │     (Rotatable)     │                    │
│                   │                     │                    │
│                   └─────────────────────┘                    │
│                   [◀]      ○○●○○       [▶]                   │
│                         (Style variants)                     │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ★ Epic Outfit                                               │
│                                                              │
│  "Command the seven seas in style. This legendary            │
│   captain's outfit strikes fear into enemy hearts."          │
│                                                              │
│  Acquired: Feb 10, 2025                                      │
│  Source: Shop Purchase                                       │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│          [Equip]                    [Set as Favorite]        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 3D Model Viewer
```
Features:
├── Full 360° rotation (drag)
├── Zoom in/out (pinch)
├── Animation preview (idle, walk)
├── Lighting adjustment (optional)
├── Background options
└── Screenshot/share button
```

### 4.3 Style Variants
```
If Item Has Variants:
├── Dot indicators show variants
├── Swipe or arrows to cycle
├── Each variant shows different color/style
├── Unlock status per variant
└── Price shown if variant not owned
```

---

## 5. Equip System

### 5.1 Equip Flow
```
From Inventory:
1. Tap item
2. View detail screen
3. Tap "Equip" button
4. Item becomes equipped
5. Return to inventory (equipped badge shown)

Quick Equip:
1. Long-press item in grid
2. Context menu appears
3. Tap "Equip"
4. Item equipped immediately
```

### 5.2 Unequip Flow
```
From Equipped Item:
1. Tap equipped item
2. View detail screen
3. Tap "Unequip" button
4. Slot becomes empty (or reverts to default)

Or:
1. Equip different item
2. Previous item automatically unequipped
```

### 5.3 Emote Slot Assignment
```
Emote Wheel Editor:
┌──────────────────────────────────────────────────────────┐
│              Emote Wheel Setup                           │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                     [Slot 1]                             │
│                        ↑                                 │
│              [Slot 4]  ●  [Slot 2]                       │
│                        ↓                                 │
│                     [Slot 3]                             │
│                                                          │
│  Tap a slot to assign an emote                           │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  Available Emotes:                                       │
│  [Wave] [Dance] [Laugh] [Clap] [Bow] ...                 │
└──────────────────────────────────────────────────────────┘

Flow:
1. Tap slot
2. Shows owned emotes
3. Tap emote to assign
4. Emote appears in slot
```

---

## 6. Favorite System

### 6.1 Adding Favorites
```
Methods:
├── Tap star icon on item card
├── Tap "Set as Favorite" in detail view
├── Long-press → "Add to Favorites"
└── Maximum: 50 favorites (configurable)
```

### 6.2 Favorite Usage
```
Benefits:
├── Favorites filter in inventory
├── "Favorites First" sort option
├── Quick access section (optional)
├── Random favorite option (shuffle equipped)
└── Persists across sessions
```

---

## 7. Preview System

### 7.1 In-Menu Preview
```
Outfit Preview:
├── 3D character model
├── Current outfit shown
├── Preview different items without equipping
├── Side-by-side compare (optional)
└── "Try On" button
```

### 7.2 In-Game Preview
```
Training Mode Preview:
├── Enter training ground
├── Access cosmetic menu
├── Preview how items look in-game
├── Test emotes in 3D space
├── Test sprays on surfaces
└── No commitment required
```

### 7.3 Preview Before Purchase
```
Shop Item Preview:
├── Full 3D preview
├── Animation showcase
├── Compare to currently equipped
├── "Add to Wishlist" option
└── "Buy Now" proceeds to purchase
```

---

## 8. Default Items

### 8.1 Starter Set
```
New Player Defaults:
├── 1 Default Outfit
├── 1 Default Emote (Wave)
├── 1 Default Spray
├── Default Profile Icon
├── Default Profile Banner
└── All equipped by default
```

### 8.2 Default Behavior
```
When Slot Empty:
├── Outfit: Default outfit (cannot unequip)
├── Headwear: None (head visible)
├── Backpack: None (back visible)
├── Emotes: Empty slot in wheel
├── Profile: Default icon/banner
├── Effects: None applied
```

---

## 9. Search and Discovery

### 9.1 Search Functionality
```
Search Features:
├── Text search by name
├── Search by tags
├── Recent searches saved
├── Search suggestions
└── Results highlight matches
```

### 9.2 "New" Badges
```
New Item Indicator:
├── Badge on unviewed items
├── Clears when item viewed
├── "New" filter shows all unviewed
├── Count shown in category tab
└── Persists until viewed
```

---

## 10. Integration Points

### 10.1 System Dependencies
| System | Integration |
|--------|-------------|
| Account | Player ID, persistence |
| Shop | Purchase grants items |
| Battle Pass | Rewards grant items |
| Match | Equipped items displayed |
| Profile | Icon/banner display |
| Network | Sync with server |

### 10.2 API Endpoints
```
GET /api/v1/player/{id}/inventory
├── Returns all owned items

GET /api/v1/player/{id}/equipped
├── Returns equipped loadout

POST /api/v1/player/{id}/equip
├── Body: { slot, itemId }
├── Equips item to slot

POST /api/v1/player/{id}/unequip
├── Body: { slot }
├── Clears slot

POST /api/v1/player/{id}/favorite
├── Body: { itemId, isFavorite }
├── Sets favorite status

GET /api/v1/items
├── Query: { category, rarity, limit, offset }
├── Returns item catalog
```

---

## 11. Performance Optimization

### 11.1 Loading Strategy
```
Progressive Loading:
├── Load visible items first
├── Lazy load off-screen items
├── Cache item icons locally
├── Preload 3D models for equipped
└── Load detail view on demand
```

### 11.2 Memory Management
```
Asset Caching:
├── Icon cache: 50MB max
├── Model cache: 100MB max
├── LRU eviction policy
├── Clear on memory pressure
└── Persist across sessions
```

### 11.3 Pagination
```
Inventory Loading:
├── Initial load: 30 items
├── Load more: 30 per page
├── Infinite scroll or "Load More" button
├── Cache previous pages
└── Reset on filter change
```

---

## 12. Testing Requirements

### 12.1 Functional Tests
```
Test Cases:
├── View inventory with items
├── View empty inventory
├── Filter by category
├── Filter by rarity
├── Sort options work
├── Search finds items
├── Equip item updates loadout
├── Unequip reverts to default
├── Favorites persist
├── New badges clear on view
├── 3D viewer rotates/zooms
└── Emote wheel assignment
```

### 12.2 Edge Cases
```
Test Scenarios:
├── Very large inventory (500+ items)
├── No items in category
├── Item removed from catalog
├── Network failure during equip
├── Concurrent equip from multiple devices
└── Item asset missing
```

---

## 13. Default Configuration

```json
{
  "inventory": {
    "pageSize": 30,
    "maxFavorites": 50,
    "sortDefault": "newest",
    "viewDefault": "grid",
    "showNewBadges": true
  },
  "preview": {
    "enable3DViewer": true,
    "autoRotate": true,
    "backgroundDefault": "neutral",
    "animationPreview": true
  },
  "defaults": {
    "outfit": "outfit_default_01",
    "emote": "emote_wave_01",
    "profileIcon": "icon_default_01",
    "profileBanner": "banner_default_01"
  },
  "cache": {
    "iconCacheSize": 52428800,
    "modelCacheSize": 104857600,
    "evictionPolicy": "lru"
  }
}
```

---

## Appendix A: Visual Reference

```
Item Card States:

Normal:              Equipped:           Favorite:
┌─────────┐         ┌─────────┐         ┌─────────┐
│         │         │      ✓  │         │  ★      │
│  [IMG]  │         │  [IMG]  │         │  [IMG]  │
│         │         │         │         │         │
└─────────┘         └─────────┘         └─────────┘
 Item Name           Item Name           Item Name

New:                 Equipped+Fav:       Locked:
┌─────────┐         ┌─────────┐         ┌─────────┐
│    🔴   │         │  ★   ✓  │         │    🔒   │
│  [IMG]  │         │  [IMG]  │         │  [IMG]  │
│         │         │         │         │  (dim)  │
└─────────┘         └─────────┘         └─────────┘
 Item Name           Item Name           Unlock at
   NEW!                                  Level 10
```

---

*Document Version: 1.0*
*Last Updated: 2025-12-02*
*Status: Complete*
