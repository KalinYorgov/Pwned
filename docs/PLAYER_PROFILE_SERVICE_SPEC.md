# Player Profile Service Specification - Plunderstorm Mobile

**Document ID:** BACK-011
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Profile Data Model](#2-profile-data-model)
3. [Profile Creation](#3-profile-creation)
4. [Username System](#4-username-system)
5. [Player Statistics](#5-player-statistics)
6. [Inventory Management](#6-inventory-management)
7. [Progression System](#7-progression-system)
8. [Settings Synchronization](#8-settings-synchronization)
9. [Privacy Controls](#9-privacy-controls)
10. [Data Compliance](#10-data-compliance)
11. [API Specification](#11-api-specification)
12. [Client Implementation](#12-client-implementation)
13. [Performance Requirements](#13-performance-requirements)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the Player Profile Service for Plunderstorm Mobile, providing comprehensive management of player identity, statistics, inventory, progression, and settings across all game systems.

### 1.2 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Data Integrity | Player data consistent and never lost | Critical |
| Performance | Profile operations < 200ms | Critical |
| Scalability | Support millions of concurrent players | High |
| Privacy | Player control over data visibility | High |
| Compliance | GDPR and platform requirements | Critical |

### 1.3 Profile Components Overview

| Component | Description | Sync Frequency |
|-----------|-------------|----------------|
| Identity | Username, avatar, display info | On change |
| Statistics | Match performance metrics | After each match |
| Inventory | Owned cosmetics and items | On transaction |
| Progression | Level, XP, achievements | After each match |
| Settings | Game preferences | On change |
| History | Match history, purchases | After events |

### 1.4 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLIENT APPLICATION                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐       │
│  │ Profile       │  │ Inventory     │  │ Settings      │       │
│  │ Manager       │  │ Manager       │  │ Manager       │       │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘       │
│          │                  │                  │                │
│          └──────────────────┼──────────────────┘                │
│                             │                                   │
│                    ┌────────▼────────┐                         │
│                    │ Profile Service │                         │
│                    │ Client          │                         │
│                    └────────┬────────┘                         │
└─────────────────────────────┼───────────────────────────────────┘
                              │ HTTPS/TLS
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     PROFILE SERVICE                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐       │
│  │ Profile       │  │ Stats         │  │ Inventory     │       │
│  │ Controller    │◀─│ Processor     │◀─│ Controller    │       │
│  └───────┬───────┘  └───────────────┘  └───────────────┘       │
│          │                                                      │
│  ┌───────▼───────┐  ┌───────────────┐  ┌───────────────┐       │
│  │ Validation    │  │ Cache         │  │ Event         │       │
│  │ Service       │◀─│ Layer         │◀─│ Publisher     │       │
│  └───────┬───────┘  └───────────────┘  └───────────────┘       │
│          │                                                      │
│  ┌───────▼───────────────────────────────────────────────────┐ │
│  │                    Database Layer                          │ │
│  │  • Player Profiles  • Statistics  • Inventory  • Settings │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Profile Data Model

### 2.1 Core Profile Schema

```json
{
  "player_id": "player_123456",
  "username": "PirateCaptain",
  "display_name": "Pirate Captain",
  "avatar": {
    "type": "character",
    "id": "avatar_pirate_01",
    "frame_id": "frame_gold_01"
  },
  "account_status": "active",
  "account_type": "full",
  "created_at": "2024-01-15T10:00:00Z",
  "last_login": "2024-01-20T15:30:00Z",
  "last_match": "2024-01-20T15:00:00Z",
  "region": "NA",
  "language": "en",
  "timezone": "America/Los_Angeles",
  "platform_history": ["ios", "android"]
}
```

### 2.2 Statistics Schema

```json
{
  "player_id": "player_123456",
  "lifetime_stats": {
    "matches_played": 500,
    "matches_won": 75,
    "total_kills": 2500,
    "total_deaths": 1200,
    "total_assists": 800,
    "total_damage_dealt": 1500000,
    "total_damage_taken": 750000,
    "total_healing": 250000,
    "time_played_seconds": 360000,
    "longest_win_streak": 5,
    "highest_kill_game": 15,
    "chests_opened": 3000,
    "abilities_used": 15000
  },
  "mode_stats": {
    "solo": {
      "matches": 300,
      "wins": 50,
      "top_10": 150,
      "kills": 1500,
      "avg_placement": 12.5
    },
    "duo": {
      "matches": 150,
      "wins": 20,
      "top_5": 60,
      "kills": 800,
      "avg_placement": 8.2
    },
    "squad": {
      "matches": 50,
      "wins": 5,
      "top_3": 20,
      "kills": 200,
      "avg_placement": 6.5
    }
  },
  "ability_stats": {
    "fireball": {
      "times_used": 5000,
      "kills_with": 500,
      "damage_dealt": 250000
    },
    "shield": {
      "times_used": 3000,
      "damage_blocked": 150000
    }
  },
  "seasonal_stats": {
    "season_3": {
      "matches": 100,
      "wins": 15,
      "kills": 500,
      "rank_achieved": "Diamond",
      "battle_pass_level": 100
    }
  },
  "last_updated": "2024-01-20T15:00:00Z"
}
```

### 2.3 Inventory Schema

```json
{
  "player_id": "player_123456",
  "currencies": {
    "gold": 15000,
    "gems": 500,
    "battle_pass_tokens": 10
  },
  "owned_items": [
    {
      "item_id": "skin_pirate_captain",
      "item_type": "character_skin",
      "acquired_at": "2024-01-15T10:00:00Z",
      "acquisition_source": "purchase",
      "is_equipped": true
    },
    {
      "item_id": "emote_wave",
      "item_type": "emote",
      "acquired_at": "2024-01-16T12:00:00Z",
      "acquisition_source": "battle_pass",
      "is_equipped": false
    }
  ],
  "loadout": {
    "character_skin": "skin_pirate_captain",
    "weapon_skin": "weapon_default",
    "emotes": ["emote_wave", "emote_dance", "emote_taunt", null],
    "avatar": "avatar_pirate_01",
    "avatar_frame": "frame_gold_01",
    "title": "title_treasure_hunter"
  },
  "purchase_history": [
    {
      "transaction_id": "txn_abc123",
      "item_ids": ["skin_pirate_captain"],
      "currency_type": "gems",
      "amount": 1000,
      "timestamp": "2024-01-15T10:00:00Z"
    }
  ],
  "version": 42,
  "last_updated": "2024-01-20T15:30:00Z"
}
```

### 2.4 Progression Schema

```json
{
  "player_id": "player_123456",
  "player_level": 45,
  "player_xp": 125000,
  "xp_to_next_level": 3500,
  "battle_pass": {
    "season_id": "season_3",
    "is_premium": true,
    "level": 75,
    "xp": 7500,
    "xp_to_next": 500,
    "claimed_rewards": [1, 2, 3, 4, 5],
    "purchased_at": "2024-01-01T00:00:00Z"
  },
  "ranked": {
    "current_rank": "Diamond",
    "current_tier": 2,
    "rank_points": 2450,
    "peak_rank": "Diamond",
    "peak_tier": 1,
    "games_played": 50,
    "wins": 25
  },
  "mastery": {
    "fireball": {
      "level": 10,
      "xp": 5000
    },
    "shield": {
      "level": 7,
      "xp": 3500
    }
  },
  "quests": {
    "daily": [
      {
        "quest_id": "daily_kills_10",
        "progress": 7,
        "target": 10,
        "expires_at": "2024-01-21T00:00:00Z"
      }
    ],
    "weekly": [
      {
        "quest_id": "weekly_wins_5",
        "progress": 3,
        "target": 5,
        "expires_at": "2024-01-27T00:00:00Z"
      }
    ]
  },
  "last_updated": "2024-01-20T15:00:00Z"
}
```

### 2.5 Settings Schema

```json
{
  "player_id": "player_123456",
  "graphics": {
    "quality_preset": "high",
    "frame_rate_limit": 60,
    "shadows_enabled": true,
    "effects_quality": "high",
    "resolution_scale": 1.0
  },
  "audio": {
    "master_volume": 0.8,
    "music_volume": 0.6,
    "sfx_volume": 0.9,
    "voice_volume": 0.7,
    "voice_chat_enabled": true,
    "push_to_talk": false
  },
  "controls": {
    "sensitivity": 0.5,
    "invert_y": false,
    "auto_aim_strength": 0.3,
    "joystick_size": "medium",
    "joystick_position": {"x": 0.15, "y": 0.3},
    "button_layout": "default"
  },
  "notifications": {
    "push_enabled": true,
    "friend_requests": true,
    "party_invites": true,
    "promotions": false,
    "daily_reminder": true
  },
  "privacy": {
    "profile_visibility": "friends",
    "stats_visibility": "public",
    "online_status": "friends",
    "friend_requests": "everyone",
    "party_invites": "friends"
  },
  "accessibility": {
    "colorblind_mode": "none",
    "text_size": "medium",
    "screen_shake": true,
    "reduced_motion": false,
    "subtitles": true
  },
  "version": 15,
  "last_updated": "2024-01-20T14:00:00Z"
}
```

---

## 3. Profile Creation

### 3.1 Profile Creation Flow

```
┌──────────────────────────────────────────────────────────────┐
│                   PROFILE CREATION FLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. Authentication completes (BACK-010)                       │
│          │                                                    │
│          ▼                                                    │
│  2. Check for existing profile                                │
│     • If exists → Load and return                             │
│     • If not → Continue to creation                           │
│          │                                                    │
│          ▼                                                    │
│  3. Initialize profile with defaults                          │
│     • Generate unique player_id                               │
│     • Set account type (guest/full)                           │
│     • Initialize empty stats                                  │
│     • Create default inventory                                │
│     • Set default settings                                    │
│          │                                                    │
│          ▼                                                    │
│  4. Prompt username selection (or generate)                   │
│     • Guest: Auto-generate (e.g., "Pirate_12345")             │
│     • Full: User selects username                             │
│          │                                                    │
│          ▼                                                    │
│  5. Grant starter items                                       │
│     • Default character skin                                  │
│     • Basic emotes                                            │
│     • Tutorial completion rewards                             │
│          │                                                    │
│          ▼                                                    │
│  6. Profile ready → Enter game                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Profile Creation Request

```
POST /profiles
Authorization: Bearer <access_token>

{
  "suggested_username": "PirateCaptain",
  "language": "en",
  "timezone": "America/Los_Angeles",
  "platform": "ios",
  "device_info": {
    "model": "iPhone 15 Pro",
    "os_version": "17.0"
  }
}

Response (Success):
{
  "player_id": "player_123456",
  "username": "PirateCaptain",
  "display_name": "PirateCaptain",
  "account_type": "full",
  "created_at": "2024-01-15T10:00:00Z",
  "starter_items": [
    {"item_id": "skin_default", "item_type": "character_skin"},
    {"item_id": "emote_wave", "item_type": "emote"}
  ],
  "currencies": {
    "gold": 1000,
    "gems": 100
  }
}
```

### 3.3 Guest Profile Handling

```python
class ProfileService:
    def create_guest_profile(self, auth_result: AuthResult) -> Profile:
        """Create profile for guest user"""

        # Generate guest username
        username = self.generate_guest_username()

        profile = Profile(
            player_id=self.generate_player_id(),
            username=username,
            display_name=username,
            account_type=AccountType.GUEST,
            created_at=datetime.utcnow(),
            region=self.detect_region(auth_result.device_info)
        )

        # Initialize with minimal defaults
        profile.stats = self.create_empty_stats()
        profile.inventory = self.create_starter_inventory()
        profile.settings = self.create_default_settings()
        profile.progression = self.create_initial_progression()

        self.db.save_profile(profile)

        return profile

    def generate_guest_username(self) -> str:
        """Generate random guest username"""
        prefixes = ["Pirate", "Captain", "Sailor", "Buccaneer", "Corsair"]
        prefix = random.choice(prefixes)
        suffix = random.randint(10000, 99999)
        return f"{prefix}_{suffix}"
```

---

## 4. Username System

### 4.1 Username Requirements

| Requirement | Specification |
|-------------|---------------|
| Length | 3-16 characters |
| Characters | Alphanumeric, underscore, hyphen |
| Case | Case-insensitive uniqueness |
| Format | Cannot start/end with special chars |
| Reserved | Cannot use reserved words |
| Offensive | Filtered against word list |

### 4.2 Username Validation

```python
class UsernameValidator:
    MIN_LENGTH = 3
    MAX_LENGTH = 16
    PATTERN = r'^[a-zA-Z][a-zA-Z0-9_-]*[a-zA-Z0-9]$'

    RESERVED_WORDS = [
        'admin', 'moderator', 'support', 'plunderstorm',
        'system', 'official', 'staff', 'dev', 'developer'
    ]

    def validate(self, username: str) -> ValidationResult:
        errors = []

        # Length check
        if len(username) < self.MIN_LENGTH:
            errors.append(f"Username must be at least {self.MIN_LENGTH} characters")
        elif len(username) > self.MAX_LENGTH:
            errors.append(f"Username cannot exceed {self.MAX_LENGTH} characters")

        # Format check
        if not re.match(self.PATTERN, username):
            errors.append("Username can only contain letters, numbers, underscores, and hyphens")

        # Reserved words check
        if username.lower() in self.RESERVED_WORDS:
            errors.append("This username is reserved")

        # Offensive content check
        if self.offensive_filter.contains_offensive(username):
            errors.append("Username contains inappropriate content")

        # Uniqueness check
        if self.db.username_exists(username):
            errors.append("Username is already taken")

        return ValidationResult(
            is_valid=len(errors) == 0,
            errors=errors
        )
```

### 4.3 Username Change

```
POST /profiles/me/username
Authorization: Bearer <access_token>

{
  "new_username": "NewPirateName"
}

Response (Success):
{
  "success": true,
  "username": "NewPirateName",
  "changes_remaining": 2,
  "next_change_available_at": null
}

Response (Failure):
{
  "success": false,
  "error": "username_taken",
  "message": "This username is already in use",
  "suggestions": ["NewPirateName1", "NewPirateName_", "NewPirateName99"]
}
```

### 4.4 Username Change Limits

| Account Type | Free Changes | Cooldown | Paid Changes |
|--------------|--------------|----------|--------------|
| Guest | 1 | 30 days | Not available |
| Full | 3 | 30 days | 500 gems each |

---

## 5. Player Statistics

### 5.1 Statistics Categories

#### Lifetime Statistics

| Stat | Description | Update Trigger |
|------|-------------|----------------|
| matches_played | Total matches completed | Match end |
| matches_won | Total victories | Match win |
| total_kills | Total eliminations | Kill event |
| total_deaths | Total times eliminated | Death event |
| total_assists | Kill assists | Assist event |
| total_damage_dealt | Damage to players | Damage event |
| total_damage_taken | Damage received | Damage event |
| total_healing | Health restored | Heal event |
| time_played_seconds | Total play time | Match end |
| chests_opened | Loot containers opened | Loot event |
| abilities_used | Ability activations | Ability event |

#### Derived Statistics (Calculated)

| Stat | Formula |
|------|---------|
| win_rate | matches_won / matches_played |
| kd_ratio | total_kills / total_deaths |
| kda_ratio | (total_kills + total_assists) / total_deaths |
| avg_kills_per_match | total_kills / matches_played |
| avg_damage_per_match | total_damage_dealt / matches_played |
| avg_survival_time | time_played_seconds / matches_played |

### 5.2 Statistics Update Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Game Server │     │ Stats Queue │     │   Profile   │
│             │     │             │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │ Match Events      │                   │
       │ (kills, damage)   │                   │
       │──────────────────▶│                   │
       │                   │                   │
       │ Match End         │                   │
       │──────────────────▶│                   │
       │                   │                   │
       │                   │ Aggregate Stats   │
       │                   │──────────────────▶│
       │                   │                   │
       │                   │                   │ Validate
       │                   │                   │ Update Profile
       │                   │                   │ Cache Invalidate
       │                   │                   │
       │                   │ Confirmation      │
       │                   │◀──────────────────│
       │                   │                   │
```

### 5.3 Statistics Processing

```python
class StatsProcessor:
    def process_match_stats(self, player_id: str, match_result: MatchResult):
        """Process and store match statistics"""

        # Validate match result
        if not self.validate_match_result(match_result):
            raise InvalidMatchResultError("Match result validation failed")

        # Get current stats
        current_stats = self.get_player_stats(player_id)

        # Update lifetime stats
        current_stats.lifetime.matches_played += 1
        current_stats.lifetime.total_kills += match_result.kills
        current_stats.lifetime.total_deaths += match_result.deaths
        current_stats.lifetime.total_assists += match_result.assists
        current_stats.lifetime.total_damage_dealt += match_result.damage_dealt
        current_stats.lifetime.total_damage_taken += match_result.damage_taken
        current_stats.lifetime.time_played_seconds += match_result.duration_seconds

        if match_result.placement == 1:
            current_stats.lifetime.matches_won += 1

        # Update mode-specific stats
        mode_stats = current_stats.mode_stats[match_result.mode]
        mode_stats.matches += 1
        mode_stats.kills += match_result.kills

        if match_result.placement == 1:
            mode_stats.wins += 1

        # Update ability stats
        for ability_usage in match_result.abilities_used:
            ability_stats = current_stats.ability_stats.get(ability_usage.ability_id, {})
            ability_stats['times_used'] = ability_stats.get('times_used', 0) + ability_usage.count
            ability_stats['kills_with'] = ability_stats.get('kills_with', 0) + ability_usage.kills
            current_stats.ability_stats[ability_usage.ability_id] = ability_stats

        # Update seasonal stats
        current_season = self.get_current_season()
        if current_season.id not in current_stats.seasonal_stats:
            current_stats.seasonal_stats[current_season.id] = SeasonalStats()

        seasonal = current_stats.seasonal_stats[current_season.id]
        seasonal.matches += 1
        seasonal.kills += match_result.kills
        if match_result.placement == 1:
            seasonal.wins += 1

        # Save and invalidate cache
        current_stats.last_updated = datetime.utcnow()
        self.save_stats(player_id, current_stats)
        self.cache.invalidate(f"stats:{player_id}")

        # Publish stats update event
        self.events.publish(StatsUpdatedEvent(player_id, match_result))
```

### 5.4 Anti-Cheat Validation

```python
class StatsValidator:
    # Maximum values per match (sanity checks)
    MAX_KILLS_PER_MATCH = 59  # Can't kill more than all opponents
    MAX_DAMAGE_PER_MATCH = 200000  # Reasonable upper bound
    MAX_HEALING_PER_MATCH = 50000

    def validate_match_result(self, result: MatchResult) -> bool:
        """Validate match result for impossibilities"""

        # Check for impossible values
        if result.kills > self.MAX_KILLS_PER_MATCH:
            self.flag_suspicious(result, "kills_too_high")
            return False

        if result.damage_dealt > self.MAX_DAMAGE_PER_MATCH:
            self.flag_suspicious(result, "damage_too_high")
            return False

        # Check for impossible timing
        if result.kills > 0 and result.duration_seconds < 30:
            self.flag_suspicious(result, "kills_too_fast")
            return False

        # Cross-reference with server logs
        server_record = self.get_server_match_record(result.match_id)
        if not self.records_match(result, server_record):
            self.flag_suspicious(result, "client_server_mismatch")
            return False

        return True
```

---

## 6. Inventory Management

### 6.1 Item Types

| Type | Description | Equip Slot |
|------|-------------|------------|
| character_skin | Character appearance | character_skin |
| weapon_skin | Weapon appearance | weapon_skin |
| emote | Emote animation | emotes[0-3] |
| avatar | Profile picture | avatar |
| avatar_frame | Avatar border | avatar_frame |
| title | Display title | title |
| banner | Profile banner | banner |
| trail | Movement effect | trail |
| elimination_effect | Kill effect | elimination_effect |

### 6.2 Inventory Operations

```
# Get inventory
GET /profiles/me/inventory
Authorization: Bearer <access_token>

Response:
{
  "currencies": {
    "gold": 15000,
    "gems": 500
  },
  "items": [...],
  "loadout": {...},
  "version": 42
}

# Equip item
POST /profiles/me/inventory/equip
{
  "item_id": "skin_pirate_captain",
  "slot": "character_skin"
}

Response:
{
  "success": true,
  "loadout": {...},
  "version": 43
}

# Unequip item
POST /profiles/me/inventory/unequip
{
  "slot": "character_skin"
}
```

### 6.3 Inventory Transaction Processing

```python
class InventoryService:
    def add_item(self, player_id: str, item_id: str, source: str) -> InventoryResult:
        """Add item to player inventory with transaction safety"""

        # Start transaction
        with self.db.transaction() as txn:
            # Get current inventory with lock
            inventory = txn.get_inventory_for_update(player_id)

            # Check if already owned (idempotency)
            if inventory.has_item(item_id):
                return InventoryResult(
                    success=True,
                    already_owned=True,
                    item_id=item_id
                )

            # Validate item exists
            item = self.catalog.get_item(item_id)
            if not item:
                raise InvalidItemError(f"Item {item_id} not found")

            # Add item
            inventory.add_item(OwnedItem(
                item_id=item_id,
                item_type=item.type,
                acquired_at=datetime.utcnow(),
                acquisition_source=source
            ))

            # Increment version
            inventory.version += 1
            inventory.last_updated = datetime.utcnow()

            # Save
            txn.save_inventory(player_id, inventory)
            txn.commit()

            # Invalidate cache
            self.cache.invalidate(f"inventory:{player_id}")

            # Publish event
            self.events.publish(ItemAcquiredEvent(player_id, item_id, source))

            return InventoryResult(
                success=True,
                item_id=item_id,
                new_version=inventory.version
            )

    def spend_currency(self, player_id: str, currency: str, amount: int, reason: str) -> bool:
        """Spend currency with validation"""

        with self.db.transaction() as txn:
            inventory = txn.get_inventory_for_update(player_id)

            current_balance = inventory.currencies.get(currency, 0)

            if current_balance < amount:
                raise InsufficientFundsError(
                    f"Insufficient {currency}: have {current_balance}, need {amount}"
                )

            inventory.currencies[currency] = current_balance - amount
            inventory.version += 1

            # Log transaction
            inventory.add_transaction(CurrencyTransaction(
                currency=currency,
                amount=-amount,
                reason=reason,
                timestamp=datetime.utcnow()
            ))

            txn.save_inventory(player_id, inventory)
            txn.commit()

            return True
```

### 6.4 Inventory Synchronization

```python
class InventorySyncService:
    def sync_inventory(self, player_id: str, client_version: int) -> SyncResult:
        """Sync inventory with optimistic locking"""

        server_inventory = self.get_inventory(player_id)

        # Check version
        if client_version == server_inventory.version:
            # Client is up to date
            return SyncResult(in_sync=True)

        if client_version > server_inventory.version:
            # Client ahead - shouldn't happen, investigate
            self.log_warning(f"Client version ahead: {client_version} > {server_inventory.version}")
            # Force client to accept server state

        # Send delta or full sync
        if server_inventory.version - client_version < 10:
            # Small gap - send changes
            changes = self.get_changes_since(player_id, client_version)
            return SyncResult(
                in_sync=False,
                sync_type="delta",
                changes=changes,
                new_version=server_inventory.version
            )
        else:
            # Large gap - send full inventory
            return SyncResult(
                in_sync=False,
                sync_type="full",
                inventory=server_inventory,
                new_version=server_inventory.version
            )
```

---

## 7. Progression System

### 7.1 Player Level

| Level Range | XP Required | Cumulative XP |
|-------------|-------------|---------------|
| 1-10 | 1,000 per level | 10,000 |
| 11-25 | 1,500 per level | 32,500 |
| 26-50 | 2,000 per level | 82,500 |
| 51-100 | 2,500 per level | 207,500 |
| 101+ | 3,000 per level | Unlimited |

### 7.2 XP Sources

| Source | Base XP | Multipliers |
|--------|---------|-------------|
| Match Completion | 100 | x2 for win |
| Kill | 50 | - |
| Assist | 25 | - |
| Top 10 | 100 | - |
| Top 3 | 200 | - |
| Victory | 300 | - |
| Daily First Win | 500 | - |
| Quest Completion | Varies | - |

### 7.3 Progression Update

```python
class ProgressionService:
    def award_match_xp(self, player_id: str, match_result: MatchResult) -> XPResult:
        """Calculate and award XP for match"""

        # Calculate base XP
        xp = 100  # Match completion
        xp += match_result.kills * 50
        xp += match_result.assists * 25

        # Placement bonuses
        if match_result.placement <= 10:
            xp += 100
        if match_result.placement <= 3:
            xp += 200
        if match_result.placement == 1:
            xp += 300

        # Check first win of day
        if match_result.placement == 1:
            if self.is_first_win_today(player_id):
                xp += 500
                self.mark_first_win(player_id)

        # Apply multipliers (battle pass, events, etc.)
        multiplier = self.get_xp_multiplier(player_id)
        xp = int(xp * multiplier)

        # Award XP
        progression = self.get_progression(player_id)
        old_level = progression.player_level

        progression.player_xp += xp

        # Level up check
        levels_gained = 0
        while progression.player_xp >= self.xp_for_level(progression.player_level + 1):
            progression.player_xp -= self.xp_for_level(progression.player_level + 1)
            progression.player_level += 1
            levels_gained += 1

            # Grant level up rewards
            self.grant_level_rewards(player_id, progression.player_level)

        progression.xp_to_next_level = self.xp_for_level(progression.player_level + 1) - progression.player_xp

        self.save_progression(player_id, progression)

        return XPResult(
            xp_earned=xp,
            total_xp=progression.player_xp,
            new_level=progression.player_level,
            levels_gained=levels_gained
        )
```

### 7.4 Battle Pass Progression

```python
class BattlePassService:
    def award_battle_pass_xp(self, player_id: str, xp: int) -> BattlePassResult:
        """Award XP to battle pass"""

        progression = self.get_progression(player_id)
        battle_pass = progression.battle_pass

        if not battle_pass or battle_pass.season_id != self.current_season_id:
            return BattlePassResult(success=False, reason="no_active_pass")

        old_level = battle_pass.level
        battle_pass.xp += xp

        # Level up battle pass
        levels_gained = 0
        new_rewards = []

        while battle_pass.xp >= self.XP_PER_LEVEL and battle_pass.level < self.MAX_LEVEL:
            battle_pass.xp -= self.XP_PER_LEVEL
            battle_pass.level += 1
            levels_gained += 1

            # Get rewards for this level
            rewards = self.get_level_rewards(battle_pass.level, battle_pass.is_premium)
            new_rewards.extend(rewards)

        battle_pass.xp_to_next = self.XP_PER_LEVEL - battle_pass.xp

        self.save_progression(player_id, progression)

        # Auto-claim rewards (or queue for manual claim based on design)
        for reward in new_rewards:
            self.grant_reward(player_id, reward)

        return BattlePassResult(
            success=True,
            xp_earned=xp,
            new_level=battle_pass.level,
            levels_gained=levels_gained,
            new_rewards=new_rewards
        )
```

---

## 8. Settings Synchronization

### 8.1 Settings Sync Strategy

```
┌──────────────────────────────────────────────────────────────┐
│                   SETTINGS SYNC STRATEGY                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  On Login:                                                    │
│  1. Fetch server settings                                     │
│  2. Compare with local settings                               │
│  3. If server newer → Apply server settings                   │
│  4. If local newer → Upload local settings                    │
│  5. If conflict → Merge (server wins for security settings)   │
│                                                               │
│  On Change:                                                   │
│  1. Apply locally immediately                                 │
│  2. Queue for upload                                          │
│  3. Batch upload every 30 seconds or on app background        │
│  4. Confirm sync success                                      │
│                                                               │
│  Conflict Resolution:                                         │
│  • Security/Privacy: Server wins                              │
│  • Gameplay: Most recent wins                                 │
│  • Device-specific: Local wins                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 8.2 Settings Categories

| Category | Synced | Device-Specific |
|----------|--------|-----------------|
| Graphics | Yes | Override allowed |
| Audio | Yes | No |
| Controls | Yes | Override allowed |
| Notifications | Yes | No |
| Privacy | Yes | No |
| Accessibility | Yes | No |

### 8.3 Settings API

```
# Get settings
GET /profiles/me/settings
Authorization: Bearer <access_token>

Response:
{
  "settings": {...},
  "version": 15,
  "last_updated": "2024-01-20T14:00:00Z"
}

# Update settings
PATCH /profiles/me/settings
{
  "audio": {
    "master_volume": 0.7
  },
  "client_version": 15
}

Response (Success):
{
  "success": true,
  "version": 16,
  "merged_settings": {...}
}

Response (Conflict):
{
  "success": false,
  "error": "version_conflict",
  "server_version": 17,
  "server_settings": {...}
}
```

---

## 9. Privacy Controls

### 9.1 Visibility Settings

| Setting | Options | Default |
|---------|---------|---------|
| profile_visibility | public, friends, private | friends |
| stats_visibility | public, friends, private | public |
| online_status | everyone, friends, invisible | friends |
| match_history | public, friends, private | friends |
| friend_requests | everyone, friends_of_friends, none | everyone |
| party_invites | everyone, friends, none | friends |

### 9.2 Profile Visibility Implementation

```python
class PrivacyService:
    def can_view_profile(self, viewer_id: str, target_id: str) -> bool:
        """Check if viewer can see target's profile"""

        if viewer_id == target_id:
            return True

        target_settings = self.get_privacy_settings(target_id)
        visibility = target_settings.profile_visibility

        if visibility == "public":
            return True

        if visibility == "friends":
            return self.are_friends(viewer_id, target_id)

        if visibility == "private":
            return False

        return False

    def get_visible_profile(self, viewer_id: str, target_id: str) -> ProfileView:
        """Get profile with privacy applied"""

        if not self.can_view_profile(viewer_id, target_id):
            return ProfileView(
                player_id=target_id,
                username="Hidden",
                is_private=True
            )

        profile = self.get_full_profile(target_id)
        settings = self.get_privacy_settings(target_id)

        view = ProfileView(
            player_id=profile.player_id,
            username=profile.username,
            display_name=profile.display_name,
            avatar=profile.avatar,
            is_private=False
        )

        # Apply stats visibility
        if self.can_view_stats(viewer_id, target_id, settings):
            view.stats = profile.stats

        # Apply online status visibility
        if self.can_view_online_status(viewer_id, target_id, settings):
            view.online_status = self.get_online_status(target_id)

        return view
```

### 9.3 Block System Integration

```python
class BlockService:
    def block_player(self, blocker_id: str, blocked_id: str) -> bool:
        """Block a player"""

        # Add to block list
        self.db.add_block(blocker_id, blocked_id)

        # Remove any existing friend relationship
        self.friends_service.remove_friend(blocker_id, blocked_id)

        # Cancel any pending requests
        self.friends_service.cancel_request(blocker_id, blocked_id)
        self.friends_service.cancel_request(blocked_id, blocker_id)

        # Remove from any shared party
        self.party_service.remove_if_shared(blocker_id, blocked_id)

        return True

    def is_blocked(self, viewer_id: str, target_id: str) -> bool:
        """Check if either player has blocked the other"""
        return (
            self.db.has_block(viewer_id, target_id) or
            self.db.has_block(target_id, viewer_id)
        )
```

---

## 10. Data Compliance

### 10.1 GDPR Compliance

| Requirement | Implementation |
|-------------|----------------|
| Right to Access | Data export endpoint |
| Right to Rectification | Profile edit endpoints |
| Right to Erasure | Account deletion endpoint |
| Data Portability | Machine-readable export |
| Consent | Explicit opt-in for analytics |

### 10.2 Data Export

```
POST /profiles/me/export
Authorization: Bearer <access_token>

{
  "format": "json",
  "include": ["profile", "stats", "inventory", "settings", "history"]
}

Response:
{
  "request_id": "export_abc123",
  "status": "processing",
  "estimated_completion": "2024-01-20T16:00:00Z",
  "download_expires_at": "2024-01-27T16:00:00Z"
}

# Download ready notification sent via email/push

GET /profiles/me/export/export_abc123
Authorization: Bearer <access_token>

Response: Downloadable ZIP file containing:
- profile.json
- statistics.json
- inventory.json
- settings.json
- match_history.json
- purchase_history.json
```

### 10.3 Account Deletion

```
┌──────────────────────────────────────────────────────────────┐
│                 ACCOUNT DELETION FLOW                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. User requests deletion                                    │
│     POST /profiles/me/delete                                  │
│          │                                                    │
│          ▼                                                    │
│  2. Verify identity (re-authentication required)              │
│          │                                                    │
│          ▼                                                    │
│  3. Show deletion consequences                                │
│     • All progress will be lost                               │
│     • All purchases will be lost (IAP can be restored)        │
│     • Username will be released                               │
│          │                                                    │
│          ▼                                                    │
│  4. Cooling-off period (30 days)                              │
│     • Account marked for deletion                             │
│     • User can cancel during this period                      │
│     • Login shows "account scheduled for deletion"            │
│          │                                                    │
│          ▼                                                    │
│  5. Permanent deletion                                        │
│     • Profile data deleted                                    │
│     • Statistics anonymized (for aggregate analytics)         │
│     • Purchase records retained (legal requirement)           │
│     • Audit log retained (security requirement)               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 10.4 Data Retention Policy

| Data Type | Retention Period | After Deletion |
|-----------|------------------|----------------|
| Profile Data | Until deletion | Deleted immediately |
| Statistics | Until deletion | Anonymized |
| Inventory | Until deletion | Deleted immediately |
| Settings | Until deletion | Deleted immediately |
| Match History | 90 days | Anonymized |
| Purchase History | 7 years | Retained (legal) |
| Audit Logs | 2 years | Retained |

---

## 11. API Specification

### 11.1 Endpoints Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/profiles` | POST | Create profile |
| `/profiles/me` | GET | Get own profile |
| `/profiles/me` | PATCH | Update profile |
| `/profiles/me/username` | POST | Change username |
| `/profiles/me/avatar` | PUT | Update avatar |
| `/profiles/me/stats` | GET | Get own statistics |
| `/profiles/me/inventory` | GET | Get inventory |
| `/profiles/me/inventory/equip` | POST | Equip item |
| `/profiles/me/inventory/unequip` | POST | Unequip item |
| `/profiles/me/progression` | GET | Get progression |
| `/profiles/me/settings` | GET | Get settings |
| `/profiles/me/settings` | PATCH | Update settings |
| `/profiles/me/privacy` | GET | Get privacy settings |
| `/profiles/me/privacy` | PATCH | Update privacy |
| `/profiles/me/export` | POST | Request data export |
| `/profiles/me/delete` | POST | Request deletion |
| `/profiles/{player_id}` | GET | View other profile |
| `/profiles/{player_id}/stats` | GET | View other stats |
| `/profiles/search` | GET | Search profiles |

### 11.2 Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `profile_not_found` | 404 | Profile doesn't exist |
| `profile_private` | 403 | Profile is private |
| `username_taken` | 409 | Username in use |
| `username_invalid` | 400 | Invalid username format |
| `username_change_limit` | 429 | Too many username changes |
| `item_not_owned` | 400 | Cannot equip unowned item |
| `invalid_slot` | 400 | Item doesn't fit slot |
| `insufficient_funds` | 400 | Not enough currency |
| `version_conflict` | 409 | Optimistic lock failure |
| `export_in_progress` | 429 | Export already requested |
| `deletion_pending` | 400 | Account pending deletion |

### 11.3 Rate Limits

| Endpoint Category | Limit | Window |
|-------------------|-------|--------|
| Profile Read | 100 | 1 minute |
| Profile Write | 20 | 1 minute |
| Settings Sync | 30 | 1 minute |
| Username Change | 3 | 30 days |
| Data Export | 1 | 7 days |

---

## 12. Client Implementation

### 12.1 Profile Manager (Swift)

```swift
class ProfileManager: ObservableObject {
    @Published var profile: PlayerProfile?
    @Published var stats: PlayerStats?
    @Published var inventory: PlayerInventory?
    @Published var progression: PlayerProgression?
    @Published var settings: PlayerSettings?

    private let profileService = ProfileService()
    private let cache = ProfileCache()

    func loadProfile() async throws {
        // Try cache first
        if let cached = cache.loadProfile() {
            self.profile = cached.profile
            self.stats = cached.stats
            self.inventory = cached.inventory
        }

        // Fetch from server
        let serverProfile = try await profileService.getProfile()

        // Update local state
        await MainActor.run {
            self.profile = serverProfile.profile
            self.stats = serverProfile.stats
            self.inventory = serverProfile.inventory
            self.progression = serverProfile.progression
            self.settings = serverProfile.settings
        }

        // Update cache
        cache.save(serverProfile)
    }

    func updateUsername(_ newUsername: String) async throws -> UsernameResult {
        let result = try await profileService.changeUsername(newUsername)

        if result.success {
            await MainActor.run {
                self.profile?.username = result.username
            }
        }

        return result
    }

    func equipItem(_ itemId: String, slot: EquipSlot) async throws {
        // Optimistic update
        let previousLoadout = inventory?.loadout
        await MainActor.run {
            inventory?.loadout[slot] = itemId
        }

        do {
            let result = try await profileService.equipItem(itemId, slot: slot)
            await MainActor.run {
                inventory?.version = result.newVersion
            }
        } catch {
            // Rollback
            await MainActor.run {
                inventory?.loadout = previousLoadout
            }
            throw error
        }
    }

    func updateSettings(_ changes: SettingsChanges) async throws {
        // Apply locally immediately
        await MainActor.run {
            settings?.apply(changes)
        }

        // Sync to server (batched)
        settingsSyncQueue.enqueue(changes)
    }
}
```

### 12.2 Profile Manager (Kotlin)

```kotlin
class ProfileManager(
    private val profileService: ProfileService,
    private val cache: ProfileCache
) : ViewModel() {

    private val _profile = MutableStateFlow<PlayerProfile?>(null)
    val profile: StateFlow<PlayerProfile?> = _profile.asStateFlow()

    private val _stats = MutableStateFlow<PlayerStats?>(null)
    val stats: StateFlow<PlayerStats?> = _stats.asStateFlow()

    private val _inventory = MutableStateFlow<PlayerInventory?>(null)
    val inventory: StateFlow<PlayerInventory?> = _inventory.asStateFlow()

    private val settingsSyncQueue = SettingsSyncQueue()

    fun loadProfile() {
        viewModelScope.launch {
            // Try cache first
            cache.loadProfile()?.let { cached ->
                _profile.value = cached.profile
                _stats.value = cached.stats
                _inventory.value = cached.inventory
            }

            // Fetch from server
            try {
                val serverProfile = profileService.getProfile()

                _profile.value = serverProfile.profile
                _stats.value = serverProfile.stats
                _inventory.value = serverProfile.inventory

                cache.save(serverProfile)
            } catch (e: Exception) {
                // Handle error - use cached data if available
                Log.e("ProfileManager", "Failed to load profile", e)
            }
        }
    }

    suspend fun updateUsername(newUsername: String): UsernameResult {
        val result = profileService.changeUsername(newUsername)

        if (result.success) {
            _profile.value = _profile.value?.copy(username = result.username)
        }

        return result
    }

    suspend fun equipItem(itemId: String, slot: EquipSlot) {
        val previousLoadout = _inventory.value?.loadout?.toMap()

        // Optimistic update
        _inventory.value = _inventory.value?.copy(
            loadout = _inventory.value?.loadout?.toMutableMap()?.apply {
                this[slot] = itemId
            } ?: mapOf()
        )

        try {
            val result = profileService.equipItem(itemId, slot)
            _inventory.value = _inventory.value?.copy(version = result.newVersion)
        } catch (e: Exception) {
            // Rollback
            _inventory.value = _inventory.value?.copy(loadout = previousLoadout ?: mapOf())
            throw e
        }
    }
}
```

---

## 13. Performance Requirements

### 13.1 Response Time Targets

| Operation | Target | Max |
|-----------|--------|-----|
| Get Profile | 100ms | 200ms |
| Get Stats | 100ms | 200ms |
| Get Inventory | 100ms | 200ms |
| Update Profile | 150ms | 300ms |
| Equip Item | 100ms | 200ms |
| Settings Sync | 100ms | 200ms |
| Search Profiles | 200ms | 500ms |
| Data Export | Background | 24 hours |

### 13.2 Caching Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                      CACHING LAYERS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  L1: Client Memory Cache                                         │
│  • Profile, stats, inventory in memory                           │
│  • Invalidated on logout/update                                  │
│  • TTL: Session duration                                         │
│                                                                  │
│  L2: Client Disk Cache                                           │
│  • Full profile data persisted                                   │
│  • Used for offline/quick start                                  │
│  • TTL: 7 days                                                   │
│                                                                  │
│  L3: Server Redis Cache                                          │
│  • Hot profile data                                              │
│  • Invalidated on writes                                         │
│  • TTL: 1 hour                                                   │
│                                                                  │
│  L4: Server Database                                             │
│  • Source of truth                                               │
│  • Read replicas for queries                                     │
│  • Primary for writes                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 13.3 Database Schema

```sql
-- Players table (core identity)
CREATE TABLE players (
    player_id VARCHAR(36) PRIMARY KEY,
    username VARCHAR(16) UNIQUE NOT NULL,
    display_name VARCHAR(32),
    account_type ENUM('guest', 'full') NOT NULL,
    account_status ENUM('active', 'suspended', 'banned', 'deleted') NOT NULL,
    created_at TIMESTAMP NOT NULL,
    last_login TIMESTAMP,
    last_match TIMESTAMP,
    region VARCHAR(8),
    language VARCHAR(8),
    INDEX idx_username (username),
    INDEX idx_last_login (last_login)
);

-- Player statistics
CREATE TABLE player_stats (
    player_id VARCHAR(36) PRIMARY KEY,
    matches_played INT DEFAULT 0,
    matches_won INT DEFAULT 0,
    total_kills INT DEFAULT 0,
    total_deaths INT DEFAULT 0,
    total_assists INT DEFAULT 0,
    total_damage_dealt BIGINT DEFAULT 0,
    time_played_seconds BIGINT DEFAULT 0,
    last_updated TIMESTAMP,
    FOREIGN KEY (player_id) REFERENCES players(player_id)
);

-- Player inventory
CREATE TABLE player_inventory (
    player_id VARCHAR(36) PRIMARY KEY,
    gold INT DEFAULT 0,
    gems INT DEFAULT 0,
    loadout JSON,
    version INT DEFAULT 0,
    last_updated TIMESTAMP,
    FOREIGN KEY (player_id) REFERENCES players(player_id)
);

-- Owned items
CREATE TABLE owned_items (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    player_id VARCHAR(36) NOT NULL,
    item_id VARCHAR(64) NOT NULL,
    item_type VARCHAR(32) NOT NULL,
    acquired_at TIMESTAMP NOT NULL,
    acquisition_source VARCHAR(32),
    FOREIGN KEY (player_id) REFERENCES players(player_id),
    UNIQUE INDEX idx_player_item (player_id, item_id),
    INDEX idx_item_type (player_id, item_type)
);

-- Player settings
CREATE TABLE player_settings (
    player_id VARCHAR(36) PRIMARY KEY,
    settings JSON NOT NULL,
    version INT DEFAULT 0,
    last_updated TIMESTAMP,
    FOREIGN KEY (player_id) REFERENCES players(player_id)
);

-- Privacy settings
CREATE TABLE player_privacy (
    player_id VARCHAR(36) PRIMARY KEY,
    profile_visibility ENUM('public', 'friends', 'private') DEFAULT 'friends',
    stats_visibility ENUM('public', 'friends', 'private') DEFAULT 'public',
    online_status ENUM('everyone', 'friends', 'invisible') DEFAULT 'friends',
    friend_requests ENUM('everyone', 'friends_of_friends', 'none') DEFAULT 'everyone',
    FOREIGN KEY (player_id) REFERENCES players(player_id)
);
```

### 13.4 Monitoring Metrics

| Metric | Alert Threshold |
|--------|-----------------|
| Profile fetch p99 latency | > 200ms |
| Cache hit rate | < 90% |
| Profile creation errors | > 1% |
| Stats update failures | > 0.1% |
| Inventory consistency errors | > 0 |
| Settings sync failures | > 1% |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Backend Lead | _____________ | _____________ | ___/___/___ |
| Database Admin | _____________ | _____________ | ___/___/___ |
| Product Manager | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial player profile service specification |

---

*This document contains implementation details. Handle according to company security policies.*
