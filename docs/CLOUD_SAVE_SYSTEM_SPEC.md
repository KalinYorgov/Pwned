# Cloud Save System Specification

## Overview

The Cloud Save System ensures player progress, purchases, and settings are securely stored in the cloud and accessible across all devices. This is critical for mobile gaming where players frequently switch devices, reinstall apps, or play on multiple platforms.

## Goals

1. **Zero Data Loss**: Player progress never lost under normal operation
2. **Cross-Device Play**: Seamless experience across iOS, Android, and future platforms
3. **Offline Support**: Players can progress offline with automatic sync
4. **Fast Sync**: Minimal wait times during login and gameplay
5. **Conflict Resolution**: Intelligent handling of competing save states

---

## Saved Data Categories

### 1. Player Progression

```typescript
interface PlayerProgressionData {
  // Account Level
  accountLevel: number;
  accountXP: number;

  // Battle Pass
  battlePassSeason: number;
  battlePassTier: number;
  battlePassXP: number;
  battlePassPremium: boolean;

  // Ranked
  rankedTier: string;
  rankedDivision: number;
  rankedPoints: number;
  rankedPeakThisSeason: string;

  // Statistics
  lifetimeStats: {
    matchesPlayed: number;
    wins: number;
    kills: number;
    deaths: number;
    damageDealt: number;
    timePlayedSeconds: number;
    topFiveFinishes: number;
    winStreak: number;
    bestWinStreak: number;
  };

  // Per-mode statistics
  modeStats: {
    [modeId: string]: ModeStatistics;
  };

  // Seasonal stats
  seasonStats: {
    [seasonId: string]: SeasonStatistics;
  };
}
```

### 2. Inventory and Purchases

```typescript
interface InventoryData {
  // Owned items
  ownedItems: {
    itemId: string;
    acquiredAt: string;
    source: 'purchase' | 'battle_pass' | 'event' | 'reward' | 'gift';
    transactionId?: string;
  }[];

  // Currencies
  currencies: {
    gold: number;
    gems: number;
    eventTokens: { [eventId: string]: number };
  };

  // Loadouts
  equippedLoadout: {
    character: string;
    outfit: string;
    weapon: string;
    backBling: string;
    emote: string[];
    banner: string;
    title: string;
  };

  // Custom loadout presets
  savedLoadouts: LoadoutPreset[];

  // Purchase history (for restoration)
  purchaseReceipts: {
    platform: 'ios' | 'android';
    transactionId: string;
    productId: string;
    purchaseDate: string;
    verified: boolean;
  }[];
}
```

### 3. Quest Progress

```typescript
interface QuestProgressData {
  // Active daily quests
  dailyQuests: {
    questId: string;
    progress: number;
    completed: boolean;
    claimedReward: boolean;
    assignedAt: string;
    expiresAt: string;
  }[];

  // Weekly quests
  weeklyQuests: {
    questId: string;
    progress: number;
    completed: boolean;
    claimedReward: boolean;
    weekNumber: number;
    seasonId: string;
  }[];

  // Event quests
  eventQuests: {
    eventId: string;
    quests: EventQuestProgress[];
  }[];

  // Quest history
  completedQuestHistory: {
    questId: string;
    completedAt: string;
    rewardsClaimed: string[];
  }[];

  // Daily reset tracking
  lastDailyReset: string;
  lastWeeklyReset: string;
}
```

### 4. Settings

```typescript
interface SettingsData {
  // Graphics
  graphics: {
    quality: 'low' | 'medium' | 'high' | 'ultra';
    frameRateLimit: 30 | 60 | 90 | 120;
    shadows: boolean;
    postProcessing: boolean;
    antiAliasing: boolean;
  };

  // Audio
  audio: {
    masterVolume: number;
    musicVolume: number;
    sfxVolume: number;
    voiceVolume: number;
    voiceChatEnabled: boolean;
    pushToTalk: boolean;
  };

  // Controls
  controls: {
    sensitivity: number;
    aimAssist: boolean;
    autoAim: boolean;
    invertY: boolean;
    controlScheme: 'default' | 'claw' | 'custom';
    customLayout?: CustomControlLayout;
    gyroAiming: boolean;
    gyroSensitivity: number;
  };

  // HUD
  hud: {
    minimapPosition: 'top-left' | 'top-right';
    minimapSize: number;
    healthBarStyle: 'default' | 'minimal' | 'detailed';
    damageNumbers: boolean;
    hitMarkers: boolean;
    crosshairStyle: number;
    crosshairColor: string;
  };

  // Social
  social: {
    allowFriendRequests: boolean;
    allowPartyInvites: 'all' | 'friends' | 'none';
    showOnlineStatus: boolean;
    allowSpectators: boolean;
    profileVisibility: 'public' | 'friends' | 'private';
  };

  // Notifications
  notifications: {
    pushEnabled: boolean;
    dailyReminderEnabled: boolean;
    dailyReminderTime: string;
    friendOnlineNotifications: boolean;
    eventNotifications: boolean;
    shopRefreshNotifications: boolean;
  };

  // Accessibility
  accessibility: {
    colorblindMode: 'none' | 'protanopia' | 'deuteranopia' | 'tritanopia';
    subtitles: boolean;
    screenReaderSupport: boolean;
    reducedMotion: boolean;
    largeText: boolean;
  };

  // Language and region
  locale: {
    language: string;
    region: string;
    preferredServer: string;
  };
}
```

### 5. Match History

```typescript
interface MatchHistoryData {
  recentMatches: {
    matchId: string;
    playedAt: string;
    mode: string;
    placement: number;
    kills: number;
    assists: number;
    damageDealt: number;
    damageTaken: number;
    survivalTime: number;
    squadMembers?: string[];
    xpEarned: number;
    bpXpEarned: number;
    goldEarned: number;
  }[];

  // Limited to last 20 matches for storage efficiency
  maxStoredMatches: 20;
}
```

---

## Cloud Save Architecture

### Data Storage Structure

```typescript
interface CloudSaveDocument {
  // Metadata
  playerId: string;
  schemaVersion: number;
  lastModified: string;
  lastSyncedDevice: string;

  // Data sections
  progression: PlayerProgressionData;
  inventory: InventoryData;
  quests: QuestProgressData;
  settings: SettingsData;
  matchHistory: MatchHistoryData;

  // Sync metadata
  syncMetadata: {
    localChanges: number;
    conflictsResolved: number;
    lastFullSync: string;
    incrementalSyncs: number;
  };
}
```

### Storage Backend

```typescript
// Primary: Cloud Firestore with regional replication
const CLOUD_SAVE_CONFIG = {
  primaryStorage: 'firestore',
  regions: ['us-central1', 'europe-west1', 'asia-east1'],

  // Backup storage
  backupStorage: 's3',
  backupFrequency: 'daily',
  backupRetention: 30, // days

  // Cache layer
  cacheLayer: 'redis',
  cacheTTL: 300, // 5 minutes

  // Compression
  compression: 'gzip',
  compressionThreshold: 1024 // bytes
};
```

---

## Sync Mechanisms

### 1. Login Sync

```typescript
class CloudSaveService {
  async onLogin(playerId: string, deviceId: string): Promise<SyncResult> {
    // 1. Check for local data
    const localData = await this.getLocalData(playerId);

    // 2. Fetch cloud data
    const cloudData = await this.fetchCloudData(playerId);

    // 3. Compare and resolve
    if (!localData) {
      // Fresh device - download cloud data
      await this.applyCloudData(cloudData);
      return { status: 'downloaded', conflicts: 0 };
    }

    if (!cloudData) {
      // New account - upload local data
      await this.uploadToCloud(localData);
      return { status: 'uploaded', conflicts: 0 };
    }

    // 4. Conflict resolution
    const mergedData = await this.resolveConflicts(localData, cloudData);

    // 5. Apply merged data
    await this.applyMergedData(mergedData);
    await this.uploadToCloud(mergedData.data);

    return {
      status: 'merged',
      conflicts: mergedData.conflictsResolved
    };
  }
}
```

### 2. Automatic Save Triggers

```typescript
const SAVE_TRIGGERS = {
  // Immediate saves (critical data)
  immediate: [
    'purchase_completed',
    'battle_pass_tier_claimed',
    'item_acquired',
    'currency_changed',
    'quest_reward_claimed'
  ],

  // Deferred saves (batched)
  deferred: [
    'match_completed',
    'settings_changed',
    'xp_gained',
    'stat_updated'
  ],

  // Periodic saves
  periodic: {
    interval: 60000, // 1 minute during active play
    idleInterval: 300000 // 5 minutes when idle
  }
};
```

### 3. Incremental Sync

```typescript
interface IncrementalSync {
  // Only sync changed data sections
  async syncChanges(playerId: string): Promise<void> {
    const dirtyFlags = this.getDirtyFlags();

    const updates: Partial<CloudSaveDocument> = {};

    if (dirtyFlags.progression) {
      updates.progression = this.getLocalProgression();
    }
    if (dirtyFlags.inventory) {
      updates.inventory = this.getLocalInventory();
    }
    if (dirtyFlags.quests) {
      updates.quests = this.getLocalQuests();
    }
    if (dirtyFlags.settings) {
      updates.settings = this.getLocalSettings();
    }

    // Batch update
    await this.cloudStorage.update(playerId, updates, {
      merge: true,
      timestamp: Date.now()
    });

    this.clearDirtyFlags();
  }
}
```

---

## Conflict Resolution

### Resolution Strategies

```typescript
const CONFLICT_STRATEGIES: Record<string, ConflictStrategy> = {
  // Progression: Always take higher values
  progression: {
    accountLevel: 'max',
    accountXP: 'max',
    battlePassTier: 'max',
    lifetimeStats: 'sum_merge'
  },

  // Inventory: Union of items
  inventory: {
    ownedItems: 'union',
    currencies: 'max',
    purchaseReceipts: 'union'
  },

  // Quests: Complex merge
  quests: {
    dailyQuests: 'latest_if_same_quest',
    weeklyQuests: 'latest_if_same_quest',
    completedQuestHistory: 'union'
  },

  // Settings: Latest device wins (user preference)
  settings: {
    default: 'latest_timestamp'
  },

  // Match History: Union, dedupe, sort, truncate
  matchHistory: {
    recentMatches: 'union_truncate_20'
  }
};
```

### Conflict Resolution Implementation

```typescript
class ConflictResolver {
  resolve(local: CloudSaveDocument, cloud: CloudSaveDocument): MergeResult {
    const merged: CloudSaveDocument = {
      playerId: local.playerId,
      schemaVersion: Math.max(local.schemaVersion, cloud.schemaVersion),
      lastModified: new Date().toISOString(),
      lastSyncedDevice: this.getCurrentDeviceId()
    };

    let conflictsResolved = 0;

    // Progression - take max values
    merged.progression = this.mergeProgression(
      local.progression,
      cloud.progression
    );

    // Inventory - union with verification
    const inventoryResult = this.mergeInventory(
      local.inventory,
      cloud.inventory
    );
    merged.inventory = inventoryResult.data;
    conflictsResolved += inventoryResult.conflicts;

    // Quests - intelligent merge
    const questResult = this.mergeQuests(local.quests, cloud.quests);
    merged.quests = questResult.data;
    conflictsResolved += questResult.conflicts;

    // Settings - latest wins
    merged.settings = local.lastModified > cloud.lastModified
      ? local.settings
      : cloud.settings;

    // Match history - union and truncate
    merged.matchHistory = this.mergeMatchHistory(
      local.matchHistory,
      cloud.matchHistory
    );

    return { data: merged, conflictsResolved };
  }

  private mergeProgression(
    local: PlayerProgressionData,
    cloud: PlayerProgressionData
  ): PlayerProgressionData {
    return {
      accountLevel: Math.max(local.accountLevel, cloud.accountLevel),
      accountXP: Math.max(local.accountXP, cloud.accountXP),

      battlePassSeason: Math.max(local.battlePassSeason, cloud.battlePassSeason),
      battlePassTier: Math.max(local.battlePassTier, cloud.battlePassTier),
      battlePassXP: Math.max(local.battlePassXP, cloud.battlePassXP),
      battlePassPremium: local.battlePassPremium || cloud.battlePassPremium,

      rankedTier: this.higherRank(local.rankedTier, cloud.rankedTier),
      rankedDivision: Math.max(local.rankedDivision, cloud.rankedDivision),
      rankedPoints: Math.max(local.rankedPoints, cloud.rankedPoints),
      rankedPeakThisSeason: this.higherRank(
        local.rankedPeakThisSeason,
        cloud.rankedPeakThisSeason
      ),

      lifetimeStats: {
        matchesPlayed: Math.max(
          local.lifetimeStats.matchesPlayed,
          cloud.lifetimeStats.matchesPlayed
        ),
        wins: Math.max(local.lifetimeStats.wins, cloud.lifetimeStats.wins),
        kills: Math.max(local.lifetimeStats.kills, cloud.lifetimeStats.kills),
        deaths: Math.max(local.lifetimeStats.deaths, cloud.lifetimeStats.deaths),
        damageDealt: Math.max(
          local.lifetimeStats.damageDealt,
          cloud.lifetimeStats.damageDealt
        ),
        timePlayedSeconds: Math.max(
          local.lifetimeStats.timePlayedSeconds,
          cloud.lifetimeStats.timePlayedSeconds
        ),
        topFiveFinishes: Math.max(
          local.lifetimeStats.topFiveFinishes,
          cloud.lifetimeStats.topFiveFinishes
        ),
        winStreak: Math.max(
          local.lifetimeStats.winStreak,
          cloud.lifetimeStats.winStreak
        ),
        bestWinStreak: Math.max(
          local.lifetimeStats.bestWinStreak,
          cloud.lifetimeStats.bestWinStreak
        )
      },

      modeStats: this.mergeStatMaps(local.modeStats, cloud.modeStats),
      seasonStats: this.mergeStatMaps(local.seasonStats, cloud.seasonStats)
    };
  }

  private mergeInventory(
    local: InventoryData,
    cloud: InventoryData
  ): { data: InventoryData; conflicts: number } {
    let conflicts = 0;

    // Union of owned items
    const itemMap = new Map<string, InventoryData['ownedItems'][0]>();

    [...local.ownedItems, ...cloud.ownedItems].forEach(item => {
      const existing = itemMap.get(item.itemId);
      if (!existing || new Date(item.acquiredAt) < new Date(existing.acquiredAt)) {
        itemMap.set(item.itemId, item);
      }
    });

    // Currencies - take max (prevents loss)
    const currencies = {
      gold: Math.max(local.currencies.gold, cloud.currencies.gold),
      gems: Math.max(local.currencies.gems, cloud.currencies.gems),
      eventTokens: this.mergeEventTokens(
        local.currencies.eventTokens,
        cloud.currencies.eventTokens
      )
    };

    // Check for significant currency discrepancy (potential exploit)
    if (Math.abs(local.currencies.gold - cloud.currencies.gold) > 10000) {
      conflicts++;
      // Flag for review
      this.flagForReview('currency_discrepancy', { local, cloud });
    }

    // Equipped loadout - latest wins
    const equippedLoadout = local.lastModified > cloud.lastModified
      ? local.equippedLoadout
      : cloud.equippedLoadout;

    // Saved loadouts - union
    const savedLoadouts = this.mergeLoadoutPresets(
      local.savedLoadouts,
      cloud.savedLoadouts
    );

    // Purchase receipts - union (critical for restoration)
    const purchaseReceipts = this.unionByKey(
      local.purchaseReceipts,
      cloud.purchaseReceipts,
      'transactionId'
    );

    return {
      data: {
        ownedItems: Array.from(itemMap.values()),
        currencies,
        equippedLoadout,
        savedLoadouts,
        purchaseReceipts
      },
      conflicts
    };
  }
}
```

---

## Offline Play Support

### Local Cache

```typescript
class LocalSaveCache {
  private readonly CACHE_KEY = 'plunderstorm_local_save';

  async saveLocally(data: CloudSaveDocument): Promise<void> {
    const compressed = await this.compress(data);
    const encrypted = await this.encrypt(compressed);

    // Primary storage
    await AsyncStorage.setItem(this.CACHE_KEY, encrypted);

    // Backup to platform-specific secure storage
    await this.secureBackup(encrypted);
  }

  async getLocalData(playerId: string): Promise<CloudSaveDocument | null> {
    try {
      const encrypted = await AsyncStorage.getItem(this.CACHE_KEY);
      if (!encrypted) return null;

      const compressed = await this.decrypt(encrypted);
      const data = await this.decompress(compressed);

      if (data.playerId !== playerId) {
        // Different account - clear local data
        await this.clearLocalData();
        return null;
      }

      return data;
    } catch (error) {
      // Corrupted data - try backup
      return this.restoreFromBackup(playerId);
    }
  }

  // Track changes made offline
  private offlineChanges: OfflineChange[] = [];

  recordOfflineChange(change: OfflineChange): void {
    this.offlineChanges.push({
      ...change,
      timestamp: Date.now(),
      deviceId: this.getDeviceId()
    });
  }

  async syncOfflineChanges(): Promise<SyncResult> {
    if (this.offlineChanges.length === 0) {
      return { status: 'no_changes', synced: 0 };
    }

    // Sort by timestamp
    const sortedChanges = [...this.offlineChanges].sort(
      (a, b) => a.timestamp - b.timestamp
    );

    // Apply to cloud in order
    for (const change of sortedChanges) {
      await this.applyChangeToCloud(change);
    }

    const syncedCount = this.offlineChanges.length;
    this.offlineChanges = [];

    return { status: 'synced', synced: syncedCount };
  }
}
```

### Offline Queue

```typescript
interface OfflineChange {
  type: 'progression' | 'inventory' | 'quest' | 'settings' | 'match';
  action: string;
  data: any;
  timestamp: number;
  deviceId: string;
}

class OfflineQueue {
  private queue: OfflineChange[] = [];

  enqueue(change: OfflineChange): void {
    this.queue.push(change);
    this.persistQueue();
  }

  async flush(): Promise<void> {
    if (!this.isOnline()) return;

    const batch = [...this.queue];
    this.queue = [];

    try {
      await CloudSaveService.batchSync(batch);
      this.persistQueue();
    } catch (error) {
      // Restore queue on failure
      this.queue = [...batch, ...this.queue];
      this.persistQueue();
      throw error;
    }
  }

  private persistQueue(): void {
    AsyncStorage.setItem('offline_queue', JSON.stringify(this.queue));
  }
}
```

---

## Purchase Restoration

### Receipt Verification

```typescript
class PurchaseRestoration {
  async restorePurchases(playerId: string): Promise<RestorationResult> {
    // 1. Get receipts from platform
    const platformReceipts = await this.getPlatformReceipts();

    // 2. Get receipts from cloud
    const cloudReceipts = await this.getCloudReceipts(playerId);

    // 3. Find unverified purchases
    const unverified = platformReceipts.filter(
      r => !cloudReceipts.some(c => c.transactionId === r.transactionId)
    );

    // 4. Verify and grant
    const results: RestorationResult = {
      restored: [],
      failed: [],
      alreadyOwned: []
    };

    for (const receipt of unverified) {
      try {
        const verified = await this.verifyReceipt(receipt);
        if (verified) {
          await this.grantPurchase(playerId, receipt);
          results.restored.push(receipt.productId);
        }
      } catch (error) {
        results.failed.push({
          productId: receipt.productId,
          error: error.message
        });
      }
    }

    return results;
  }

  private async verifyReceipt(receipt: PurchaseReceipt): Promise<boolean> {
    // Platform-specific verification
    if (receipt.platform === 'ios') {
      return this.verifyAppleReceipt(receipt);
    } else {
      return this.verifyGoogleReceipt(receipt);
    }
  }
}
```

---

## Data Versioning and Migration

### Schema Versioning

```typescript
const CURRENT_SCHEMA_VERSION = 5;

interface SchemaVersion {
  version: number;
  migrateFrom: number | null;
  migrations: Migration[];
}

const SCHEMA_VERSIONS: SchemaVersion[] = [
  {
    version: 1,
    migrateFrom: null,
    migrations: []
  },
  {
    version: 2,
    migrateFrom: 1,
    migrations: [
      {
        name: 'add_battle_pass_premium_field',
        transform: (data) => ({
          ...data,
          progression: {
            ...data.progression,
            battlePassPremium: false
          }
        })
      }
    ]
  },
  {
    version: 3,
    migrateFrom: 2,
    migrations: [
      {
        name: 'add_accessibility_settings',
        transform: (data) => ({
          ...data,
          settings: {
            ...data.settings,
            accessibility: {
              colorblindMode: 'none',
              subtitles: false,
              screenReaderSupport: false,
              reducedMotion: false,
              largeText: false
            }
          }
        })
      }
    ]
  },
  {
    version: 4,
    migrateFrom: 3,
    migrations: [
      {
        name: 'add_gyro_controls',
        transform: (data) => ({
          ...data,
          settings: {
            ...data.settings,
            controls: {
              ...data.settings.controls,
              gyroAiming: false,
              gyroSensitivity: 0.5
            }
          }
        })
      }
    ]
  },
  {
    version: 5,
    migrateFrom: 4,
    migrations: [
      {
        name: 'add_mode_stats',
        transform: (data) => ({
          ...data,
          progression: {
            ...data.progression,
            modeStats: {},
            seasonStats: {}
          }
        })
      }
    ]
  }
];

class SchemaMigration {
  async migrate(
    data: CloudSaveDocument,
    targetVersion: number = CURRENT_SCHEMA_VERSION
  ): Promise<CloudSaveDocument> {
    let currentData = { ...data };
    let currentVersion = data.schemaVersion || 1;

    while (currentVersion < targetVersion) {
      const nextVersion = SCHEMA_VERSIONS.find(
        v => v.migrateFrom === currentVersion
      );

      if (!nextVersion) {
        throw new Error(`No migration path from v${currentVersion}`);
      }

      for (const migration of nextVersion.migrations) {
        currentData = migration.transform(currentData);
      }

      currentVersion = nextVersion.version;
      currentData.schemaVersion = currentVersion;
    }

    return currentData;
  }
}
```

---

## Error Handling

### Sync Error Recovery

```typescript
class SyncErrorHandler {
  private retryCount = 0;
  private maxRetries = 3;

  async handleSyncError(error: SyncError): Promise<RecoveryAction> {
    switch (error.type) {
      case 'network_error':
        return this.handleNetworkError(error);

      case 'conflict_error':
        return this.handleConflictError(error);

      case 'corruption_error':
        return this.handleCorruptionError(error);

      case 'version_mismatch':
        return this.handleVersionMismatch(error);

      case 'quota_exceeded':
        return this.handleQuotaExceeded(error);

      default:
        return this.handleUnknownError(error);
    }
  }

  private async handleNetworkError(error: SyncError): Promise<RecoveryAction> {
    if (this.retryCount < this.maxRetries) {
      this.retryCount++;
      const delay = Math.pow(2, this.retryCount) * 1000;
      await this.sleep(delay);
      return { action: 'retry' };
    }

    // Switch to offline mode
    return {
      action: 'offline_mode',
      message: 'Unable to sync. Playing in offline mode.'
    };
  }

  private async handleConflictError(error: SyncError): Promise<RecoveryAction> {
    // Force fetch latest and re-merge
    const cloudData = await this.forceRefresh(error.playerId);
    const localData = await this.getLocalData(error.playerId);

    const merged = await this.conflictResolver.resolve(localData, cloudData);
    await this.forceSave(merged);

    return { action: 'resolved', data: merged };
  }

  private async handleCorruptionError(error: SyncError): Promise<RecoveryAction> {
    // Try to restore from backup
    const backup = await this.getLatestBackup(error.playerId);

    if (backup) {
      return {
        action: 'restored_from_backup',
        data: backup,
        dataLoss: true,
        lossDescription: 'Restored from backup. Some recent progress may be lost.'
      };
    }

    // Last resort: contact support
    return {
      action: 'support_required',
      message: 'Data corruption detected. Please contact support.',
      errorCode: 'SYNC_CORRUPT_001'
    };
  }
}
```

---

## Security Measures

### Encryption

```typescript
const ENCRYPTION_CONFIG = {
  algorithm: 'AES-256-GCM',
  keyDerivation: 'PBKDF2',
  iterations: 100000,
  saltLength: 32,
  ivLength: 16
};

class DataEncryption {
  private async getEncryptionKey(playerId: string): Promise<CryptoKey> {
    // Derive key from player ID and device-specific secret
    const deviceSecret = await this.getDeviceSecret();
    const salt = await this.getOrCreateSalt(playerId);

    return crypto.subtle.deriveKey(
      {
        name: 'PBKDF2',
        salt,
        iterations: ENCRYPTION_CONFIG.iterations,
        hash: 'SHA-256'
      },
      await this.importPassword(deviceSecret),
      { name: 'AES-GCM', length: 256 },
      false,
      ['encrypt', 'decrypt']
    );
  }

  async encrypt(data: string, playerId: string): Promise<string> {
    const key = await this.getEncryptionKey(playerId);
    const iv = crypto.getRandomValues(new Uint8Array(ENCRYPTION_CONFIG.ivLength));

    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      key,
      new TextEncoder().encode(data)
    );

    // Combine IV and encrypted data
    const combined = new Uint8Array(iv.length + encrypted.byteLength);
    combined.set(iv);
    combined.set(new Uint8Array(encrypted), iv.length);

    return this.base64Encode(combined);
  }
}
```

### Anti-Tampering

```typescript
class IntegrityVerification {
  async verifyDataIntegrity(data: CloudSaveDocument): Promise<boolean> {
    // 1. Check schema version validity
    if (data.schemaVersion > CURRENT_SCHEMA_VERSION) {
      return false; // Future version - suspicious
    }

    // 2. Validate inventory against purchase history
    const validInventory = await this.validateInventory(data.inventory);
    if (!validInventory) {
      this.flagSuspiciousActivity(data.playerId, 'inventory_mismatch');
      return false;
    }

    // 3. Validate statistics are possible
    const validStats = this.validateStatistics(data.progression);
    if (!validStats) {
      this.flagSuspiciousActivity(data.playerId, 'impossible_stats');
      return false;
    }

    // 4. Verify currency changes are tracked
    const validCurrency = await this.validateCurrencyHistory(data);
    if (!validCurrency) {
      this.flagSuspiciousActivity(data.playerId, 'currency_manipulation');
      return false;
    }

    return true;
  }

  private validateStatistics(progression: PlayerProgressionData): boolean {
    // Win rate can't exceed 100%
    if (progression.lifetimeStats.wins > progression.lifetimeStats.matchesPlayed) {
      return false;
    }

    // Kill/death ratio sanity check
    const avgKillsPerMatch = progression.lifetimeStats.kills /
      progression.lifetimeStats.matchesPlayed;
    if (avgKillsPerMatch > 50) { // Impossible average
      return false;
    }

    // Time played sanity
    const avgMatchLength = progression.lifetimeStats.timePlayedSeconds /
      progression.lifetimeStats.matchesPlayed;
    if (avgMatchLength > 3600) { // 1 hour average per match
      return false;
    }

    return true;
  }
}
```

---

## Performance Optimization

### Compression

```typescript
const COMPRESSION_CONFIG = {
  algorithm: 'gzip',
  level: 6,
  threshold: 1024 // Only compress if > 1KB
};

class DataCompression {
  async compress(data: CloudSaveDocument): Promise<Uint8Array> {
    const json = JSON.stringify(data);

    if (json.length < COMPRESSION_CONFIG.threshold) {
      return new TextEncoder().encode(json);
    }

    const compressed = await this.gzipCompress(json);

    // Only use compressed if it's actually smaller
    if (compressed.length < json.length) {
      return this.addCompressionHeader(compressed);
    }

    return new TextEncoder().encode(json);
  }
}
```

### Batching

```typescript
class SaveBatcher {
  private pendingChanges: Map<string, any> = new Map();
  private batchTimeout: NodeJS.Timeout | null = null;
  private readonly BATCH_DELAY = 1000; // 1 second

  queueChange(section: string, data: any): void {
    this.pendingChanges.set(section, data);

    if (!this.batchTimeout) {
      this.batchTimeout = setTimeout(() => this.flush(), this.BATCH_DELAY);
    }
  }

  private async flush(): Promise<void> {
    if (this.pendingChanges.size === 0) return;

    const batch = new Map(this.pendingChanges);
    this.pendingChanges.clear();
    this.batchTimeout = null;

    await CloudSaveService.batchUpdate(Object.fromEntries(batch));
  }
}
```

---

## Analytics and Monitoring

### Sync Metrics

```typescript
const SYNC_METRICS = {
  // Timing
  syncDuration: 'cloud_save_sync_duration_ms',
  uploadDuration: 'cloud_save_upload_duration_ms',
  downloadDuration: 'cloud_save_download_duration_ms',

  // Success/Failure
  syncSuccess: 'cloud_save_sync_success_total',
  syncFailure: 'cloud_save_sync_failure_total',

  // Conflicts
  conflictsResolved: 'cloud_save_conflicts_resolved_total',
  conflictsByType: 'cloud_save_conflicts_by_type',

  // Data
  saveSize: 'cloud_save_size_bytes',
  compressionRatio: 'cloud_save_compression_ratio'
};

class SyncAnalytics {
  trackSync(result: SyncResult): void {
    Analytics.track('cloud_save_sync', {
      status: result.status,
      duration: result.duration,
      conflicts: result.conflictsResolved,
      dataSize: result.dataSize,
      offline: result.wasOffline
    });
  }
}
```

---

## Testing Strategy

### Test Scenarios

1. **Clean Install**
   - New device, no local data
   - Download from cloud
   - Verify all data present

2. **Cross-Device Sync**
   - Play on Device A
   - Login on Device B
   - Verify progress synced

3. **Offline Play**
   - Go offline
   - Complete matches
   - Restore connection
   - Verify offline progress synced

4. **Conflict Resolution**
   - Play offline on Device A
   - Play on Device B
   - Connect Device A
   - Verify merge is correct

5. **Data Migration**
   - Save with old schema
   - Update app
   - Verify migration successful

6. **Purchase Restoration**
   - Make purchase
   - Clear app data
   - Restore purchases
   - Verify items granted

---

## Integration Points

### Platform-Specific

- **iOS**: Game Center cloud save integration as fallback
- **Android**: Google Play Games save integration as fallback
- **Cross-Platform**: Primary cloud save via game servers

### Dependencies

- BACK-010: Authentication System (player identity)
- BACK-011: Player Profile Service (data source)
- Store services for purchase verification
