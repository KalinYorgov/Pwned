# Friends and Social Service Specification

## Overview

The Friends and Social Service enables players to connect with each other, form lasting gaming relationships, and enhance the multiplayer experience through social features. This includes friend management, online status tracking, recent players, blocking, and platform friend integration.

## Goals

1. **Easy Discovery**: Find and add friends through multiple methods
2. **Real-Time Status**: Always know when friends are online and what they're playing
3. **Seamless Integration**: Connect with platform friends (Game Center, Google Play)
4. **Privacy Control**: Granular control over visibility and interactions
5. **Low Latency**: Presence updates feel instant

---

## Friend System Architecture

### Data Models

```typescript
interface FriendRelationship {
  id: string;
  playerId: string;
  friendId: string;
  status: 'pending' | 'accepted' | 'blocked';
  initiator: string; // Who sent the request
  createdAt: string;
  acceptedAt?: string;
  nickname?: string; // Custom nickname for friend
  favorite: boolean;
  notes?: string; // Private notes about friend
}

interface PlayerPresence {
  playerId: string;
  status: 'online' | 'away' | 'in_match' | 'in_lobby' | 'offline';
  lastSeen: string;
  currentActivity?: {
    type: 'menu' | 'matchmaking' | 'in_match' | 'party';
    mode?: string;
    partyId?: string;
    matchId?: string;
    canJoin: boolean;
  };
  device: 'ios' | 'android' | 'unknown';
  gameVersion: string;
}

interface RecentPlayer {
  playerId: string;
  username: string;
  matchId: string;
  playedAt: string;
  wasTeammate: boolean;
  result: 'win' | 'loss' | 'draw';
  interactionType: 'teammate' | 'opponent' | 'killer' | 'killed_by';
}

interface BlockedPlayer {
  playerId: string;
  blockedPlayerId: string;
  reason?: 'harassment' | 'cheating' | 'spam' | 'other';
  blockedAt: string;
  notes?: string;
}
```

### Friend List Limits

```typescript
const FRIEND_LIMITS = {
  maxFriends: 500,
  maxPendingIncoming: 100,
  maxPendingOutgoing: 50,
  maxBlockedPlayers: 200,
  maxFavorites: 50,
  recentPlayersRetention: 50, // Last 50 unique players
  recentPlayersExpiry: 7 * 24 * 60 * 60 * 1000 // 7 days
};
```

---

## Friend Request System

### Sending Requests

```typescript
interface FriendRequestPayload {
  targetType: 'username' | 'player_id' | 'link' | 'qr_code';
  targetValue: string;
  message?: string; // Optional message with request
}

class FriendRequestService {
  async sendFriendRequest(
    senderId: string,
    payload: FriendRequestPayload
  ): Promise<FriendRequestResult> {
    // 1. Validate sender can send requests
    await this.validateSenderEligibility(senderId);

    // 2. Resolve target player
    const targetId = await this.resolveTarget(payload);

    // 3. Check existing relationship
    const existing = await this.getRelationship(senderId, targetId);
    if (existing) {
      return this.handleExistingRelationship(existing);
    }

    // 4. Check if blocked (either direction)
    const blocked = await this.checkBlocked(senderId, targetId);
    if (blocked) {
      return {
        success: false,
        error: 'request_not_allowed',
        message: 'Unable to send friend request'
      };
    }

    // 5. Check target's friend settings
    const targetSettings = await this.getPrivacySettings(targetId);
    if (!this.canReceiveRequest(targetSettings, senderId)) {
      return {
        success: false,
        error: 'requests_disabled',
        message: 'This player is not accepting friend requests'
      };
    }

    // 6. Create pending request
    const request = await this.createRequest(senderId, targetId, payload.message);

    // 7. Send notification to target
    await this.notifyTarget(targetId, senderId, request);

    return {
      success: true,
      requestId: request.id,
      status: 'pending'
    };
  }

  private async resolveTarget(payload: FriendRequestPayload): Promise<string> {
    switch (payload.targetType) {
      case 'username':
        const player = await PlayerService.findByUsername(payload.targetValue);
        if (!player) throw new Error('Player not found');
        return player.id;

      case 'player_id':
        const exists = await PlayerService.exists(payload.targetValue);
        if (!exists) throw new Error('Player not found');
        return payload.targetValue;

      case 'link':
        return this.parseFriendLink(payload.targetValue);

      case 'qr_code':
        return this.parseQRCode(payload.targetValue);

      default:
        throw new Error('Invalid target type');
    }
  }
}
```

### Friend Links

```typescript
class FriendLinkService {
  // Generate shareable friend link
  generateFriendLink(playerId: string): FriendLink {
    const token = this.generateSecureToken();
    const expiresAt = Date.now() + 7 * 24 * 60 * 60 * 1000; // 7 days

    const link: FriendLink = {
      playerId,
      token,
      expiresAt,
      uses: 0,
      maxUses: 10, // Limit uses to prevent spam
      url: `plunderstorm://friend/${token}`
    };

    this.storeFriendLink(link);
    return link;
  }

  // Parse incoming friend link
  async parseFriendLink(url: string): Promise<string> {
    const token = url.split('/').pop();
    const link = await this.getFriendLink(token);

    if (!link) {
      throw new Error('Invalid or expired friend link');
    }

    if (Date.now() > link.expiresAt) {
      throw new Error('Friend link has expired');
    }

    if (link.uses >= link.maxUses) {
      throw new Error('Friend link has reached maximum uses');
    }

    // Increment usage
    await this.incrementLinkUsage(link.token);

    return link.playerId;
  }
}
```

### Accepting/Declining Requests

```typescript
class FriendRequestHandler {
  async acceptRequest(
    playerId: string,
    requestId: string
  ): Promise<AcceptResult> {
    const request = await this.getRequest(requestId);

    // Validate request belongs to this player
    if (request.friendId !== playerId) {
      throw new Error('Unauthorized');
    }

    if (request.status !== 'pending') {
      throw new Error('Request already processed');
    }

    // Check friend limits
    const [senderCount, receiverCount] = await Promise.all([
      this.getFriendCount(request.playerId),
      this.getFriendCount(playerId)
    ]);

    if (senderCount >= FRIEND_LIMITS.maxFriends) {
      return {
        success: false,
        error: 'sender_limit_reached',
        message: 'The sender has reached their friend limit'
      };
    }

    if (receiverCount >= FRIEND_LIMITS.maxFriends) {
      return {
        success: false,
        error: 'your_limit_reached',
        message: 'You have reached your friend limit'
      };
    }

    // Accept the request
    request.status = 'accepted';
    request.acceptedAt = new Date().toISOString();
    await this.updateRequest(request);

    // Notify sender
    await NotificationService.send(request.playerId, {
      type: 'friend_request_accepted',
      data: {
        friendId: playerId,
        friendName: await this.getDisplayName(playerId)
      }
    });

    return {
      success: true,
      friendId: request.playerId
    };
  }

  async declineRequest(
    playerId: string,
    requestId: string
  ): Promise<DeclineResult> {
    const request = await this.getRequest(requestId);

    if (request.friendId !== playerId) {
      throw new Error('Unauthorized');
    }

    // Simply delete the request
    await this.deleteRequest(requestId);

    // Don't notify sender (privacy)
    return { success: true };
  }
}
```

---

## Friends List

### Retrieving Friends

```typescript
interface FriendListOptions {
  includePresence: boolean;
  filter?: 'all' | 'online' | 'favorites' | 'in_game';
  sortBy: 'name' | 'status' | 'recent' | 'favorite';
  limit?: number;
  offset?: number;
}

interface FriendEntry {
  playerId: string;
  username: string;
  displayName: string;
  avatar: string;
  level: number;
  presence?: PlayerPresence;
  relationship: {
    since: string;
    nickname?: string;
    favorite: boolean;
  };
  canInvite: boolean;
  canJoin: boolean;
}

class FriendListService {
  async getFriends(
    playerId: string,
    options: FriendListOptions
  ): Promise<PaginatedResult<FriendEntry>> {
    // 1. Get all friend relationships
    const relationships = await this.getAcceptedRelationships(playerId);

    // 2. Get friend IDs
    const friendIds = relationships.map(r =>
      r.playerId === playerId ? r.friendId : r.playerId
    );

    // 3. Batch fetch player profiles
    const profiles = await PlayerService.batchGetProfiles(friendIds);

    // 4. Optionally fetch presence
    let presenceMap: Map<string, PlayerPresence> = new Map();
    if (options.includePresence) {
      const presences = await PresenceService.batchGetPresence(friendIds);
      presenceMap = new Map(presences.map(p => [p.playerId, p]));
    }

    // 5. Build friend entries
    let entries: FriendEntry[] = relationships.map(rel => {
      const friendId = rel.playerId === playerId ? rel.friendId : rel.playerId;
      const profile = profiles.get(friendId);
      const presence = presenceMap.get(friendId);

      return {
        playerId: friendId,
        username: profile.username,
        displayName: profile.displayName,
        avatar: profile.avatar,
        level: profile.level,
        presence,
        relationship: {
          since: rel.acceptedAt,
          nickname: rel.nickname,
          favorite: rel.favorite
        },
        canInvite: this.canInviteToParty(presence),
        canJoin: presence?.currentActivity?.canJoin ?? false
      };
    });

    // 6. Apply filter
    if (options.filter) {
      entries = this.applyFilter(entries, options.filter);
    }

    // 7. Apply sort
    entries = this.applySort(entries, options.sortBy);

    // 8. Paginate
    return this.paginate(entries, options.limit, options.offset);
  }

  private applyFilter(
    entries: FriendEntry[],
    filter: string
  ): FriendEntry[] {
    switch (filter) {
      case 'online':
        return entries.filter(e =>
          e.presence?.status !== 'offline'
        );
      case 'favorites':
        return entries.filter(e => e.relationship.favorite);
      case 'in_game':
        return entries.filter(e =>
          e.presence?.status === 'in_match' ||
          e.presence?.status === 'in_lobby'
        );
      default:
        return entries;
    }
  }

  private applySort(
    entries: FriendEntry[],
    sortBy: string
  ): FriendEntry[] {
    switch (sortBy) {
      case 'status':
        // Online first, then by name
        return entries.sort((a, b) => {
          const statusOrder = { online: 0, in_match: 1, in_lobby: 1, away: 2, offline: 3 };
          const aOrder = statusOrder[a.presence?.status ?? 'offline'];
          const bOrder = statusOrder[b.presence?.status ?? 'offline'];
          if (aOrder !== bOrder) return aOrder - bOrder;
          return a.displayName.localeCompare(b.displayName);
        });

      case 'recent':
        return entries.sort((a, b) =>
          new Date(b.relationship.since).getTime() -
          new Date(a.relationship.since).getTime()
        );

      case 'favorite':
        return entries.sort((a, b) => {
          if (a.relationship.favorite !== b.relationship.favorite) {
            return a.relationship.favorite ? -1 : 1;
          }
          return a.displayName.localeCompare(b.displayName);
        });

      case 'name':
      default:
        return entries.sort((a, b) =>
          a.displayName.localeCompare(b.displayName)
        );
    }
  }
}
```

### Managing Friends

```typescript
class FriendManagementService {
  // Set custom nickname
  async setNickname(
    playerId: string,
    friendId: string,
    nickname: string | null
  ): Promise<void> {
    const relationship = await this.getRelationship(playerId, friendId);
    if (!relationship || relationship.status !== 'accepted') {
      throw new Error('Not friends');
    }

    // Validate nickname
    if (nickname) {
      if (nickname.length > 20) {
        throw new Error('Nickname too long');
      }
      if (!this.isValidNickname(nickname)) {
        throw new Error('Invalid nickname');
      }
    }

    relationship.nickname = nickname;
    await this.updateRelationship(relationship);
  }

  // Toggle favorite
  async toggleFavorite(
    playerId: string,
    friendId: string
  ): Promise<boolean> {
    const relationship = await this.getRelationship(playerId, friendId);
    if (!relationship || relationship.status !== 'accepted') {
      throw new Error('Not friends');
    }

    // Check favorite limit
    if (!relationship.favorite) {
      const favoriteCount = await this.getFavoriteCount(playerId);
      if (favoriteCount >= FRIEND_LIMITS.maxFavorites) {
        throw new Error('Maximum favorites reached');
      }
    }

    relationship.favorite = !relationship.favorite;
    await this.updateRelationship(relationship);

    return relationship.favorite;
  }

  // Remove friend
  async removeFriend(
    playerId: string,
    friendId: string
  ): Promise<void> {
    const relationship = await this.getRelationship(playerId, friendId);
    if (!relationship) {
      throw new Error('Not friends');
    }

    await this.deleteRelationship(relationship.id);

    // Don't notify the removed friend (privacy)
  }
}
```

---

## Presence System

### Real-Time Presence Updates

```typescript
class PresenceService {
  private readonly presenceCache = new Map<string, PlayerPresence>();
  private readonly subscribers = new Map<string, Set<string>>();

  // Update own presence
  async updatePresence(
    playerId: string,
    update: Partial<PlayerPresence>
  ): Promise<void> {
    const current = await this.getPresence(playerId);
    const newPresence: PlayerPresence = {
      ...current,
      ...update,
      playerId,
      lastSeen: new Date().toISOString()
    };

    // Store in cache and persistent storage
    this.presenceCache.set(playerId, newPresence);
    await this.persistPresence(newPresence);

    // Notify subscribers (friends)
    await this.notifySubscribers(playerId, newPresence);
  }

  // Subscribe to friend presence updates
  subscribeToFriends(playerId: string, friendIds: string[]): void {
    friendIds.forEach(friendId => {
      if (!this.subscribers.has(friendId)) {
        this.subscribers.set(friendId, new Set());
      }
      this.subscribers.get(friendId).add(playerId);
    });
  }

  // Notify friends of presence change
  private async notifySubscribers(
    playerId: string,
    presence: PlayerPresence
  ): Promise<void> {
    const subscribers = this.subscribers.get(playerId);
    if (!subscribers || subscribers.size === 0) return;

    const notification: PresenceUpdate = {
      type: 'presence_update',
      playerId,
      presence: this.sanitizePresence(presence)
    };

    // Send via WebSocket to all online subscribers
    await Promise.all(
      Array.from(subscribers).map(subscriberId =>
        WebSocketService.send(subscriberId, notification)
      )
    );
  }

  // Sanitize presence for privacy
  private sanitizePresence(presence: PlayerPresence): PlayerPresence {
    // Remove sensitive activity details if player has privacy settings
    return {
      ...presence,
      currentActivity: presence.currentActivity
        ? {
            type: presence.currentActivity.type,
            canJoin: presence.currentActivity.canJoin,
            mode: presence.currentActivity.mode
            // Exclude matchId, partyId for privacy
          }
        : undefined
    };
  }
}
```

### Activity States

```typescript
const ACTIVITY_STATES = {
  // Main menu
  menu: {
    displayText: 'In Menus',
    canJoin: false,
    canInvite: true
  },

  // Matchmaking queue
  matchmaking: {
    displayText: 'In Queue',
    canJoin: true, // Can join party if has room
    canInvite: true
  },

  // In active match
  in_match: {
    displayText: 'In Match',
    canJoin: false,
    canInvite: false
  },

  // In party lobby
  party: {
    displayText: 'In Party',
    canJoin: true, // If party has room and is open
    canInvite: true
  }
};

class ActivityTracker {
  updateActivity(playerId: string, event: GameEvent): void {
    switch (event.type) {
      case 'enter_main_menu':
        PresenceService.updatePresence(playerId, {
          status: 'online',
          currentActivity: { type: 'menu', canJoin: false }
        });
        break;

      case 'enter_queue':
        PresenceService.updatePresence(playerId, {
          status: 'online',
          currentActivity: {
            type: 'matchmaking',
            mode: event.mode,
            canJoin: event.partyHasRoom ?? false
          }
        });
        break;

      case 'match_started':
        PresenceService.updatePresence(playerId, {
          status: 'in_match',
          currentActivity: {
            type: 'in_match',
            mode: event.mode,
            matchId: event.matchId,
            canJoin: false
          }
        });
        break;

      case 'match_ended':
        PresenceService.updatePresence(playerId, {
          status: 'online',
          currentActivity: { type: 'menu', canJoin: false }
        });
        break;

      case 'join_party':
        PresenceService.updatePresence(playerId, {
          status: 'in_lobby',
          currentActivity: {
            type: 'party',
            partyId: event.partyId,
            canJoin: event.partyHasRoom ?? false
          }
        });
        break;

      case 'leave_party':
        PresenceService.updatePresence(playerId, {
          status: 'online',
          currentActivity: { type: 'menu', canJoin: false }
        });
        break;

      case 'go_away':
        PresenceService.updatePresence(playerId, {
          status: 'away'
        });
        break;

      case 'disconnect':
        PresenceService.updatePresence(playerId, {
          status: 'offline',
          currentActivity: undefined
        });
        break;
    }
  }
}
```

---

## Recent Players

### Tracking Recent Players

```typescript
class RecentPlayersService {
  async recordMatch(
    playerId: string,
    matchData: MatchEndData
  ): Promise<void> {
    const recentPlayers: RecentPlayer[] = [];

    // Add teammates
    for (const teammate of matchData.teammates) {
      if (teammate.playerId !== playerId) {
        recentPlayers.push({
          playerId: teammate.playerId,
          username: teammate.username,
          matchId: matchData.matchId,
          playedAt: matchData.endTime,
          wasTeammate: true,
          result: matchData.result,
          interactionType: 'teammate'
        });
      }
    }

    // Add notable opponents
    if (matchData.killedBy) {
      recentPlayers.push({
        playerId: matchData.killedBy.playerId,
        username: matchData.killedBy.username,
        matchId: matchData.matchId,
        playedAt: matchData.endTime,
        wasTeammate: false,
        result: matchData.result,
        interactionType: 'killed_by'
      });
    }

    // Add players we eliminated
    for (const eliminated of matchData.eliminatedPlayers) {
      recentPlayers.push({
        playerId: eliminated.playerId,
        username: eliminated.username,
        matchId: matchData.matchId,
        playedAt: matchData.endTime,
        wasTeammate: false,
        result: matchData.result,
        interactionType: 'killed_by'
      });
    }

    // Store recent players
    await this.storeRecentPlayers(playerId, recentPlayers);
  }

  async getRecentPlayers(
    playerId: string,
    options: { limit?: number; filterTeammates?: boolean }
  ): Promise<RecentPlayer[]> {
    let recent = await this.fetchRecentPlayers(playerId);

    // Filter expired entries
    const cutoff = Date.now() - FRIEND_LIMITS.recentPlayersExpiry;
    recent = recent.filter(
      r => new Date(r.playedAt).getTime() > cutoff
    );

    // Optional teammate filter
    if (options.filterTeammates !== undefined) {
      recent = recent.filter(r => r.wasTeammate === options.filterTeammates);
    }

    // Deduplicate by player (keep most recent)
    const seen = new Set<string>();
    recent = recent.filter(r => {
      if (seen.has(r.playerId)) return false;
      seen.add(r.playerId);
      return true;
    });

    // Apply limit
    return recent.slice(0, options.limit ?? FRIEND_LIMITS.recentPlayersRetention);
  }

  // Quick add from recent players
  async sendRequestFromRecent(
    playerId: string,
    recentPlayerId: string
  ): Promise<FriendRequestResult> {
    // Verify they are in recent players list
    const recent = await this.getRecentPlayers(playerId, {});
    const found = recent.find(r => r.playerId === recentPlayerId);

    if (!found) {
      throw new Error('Player not in recent list');
    }

    return FriendRequestService.sendFriendRequest(playerId, {
      targetType: 'player_id',
      targetValue: recentPlayerId
    });
  }
}
```

---

## Blocking System

### Block Operations

```typescript
class BlockService {
  async blockPlayer(
    playerId: string,
    targetId: string,
    reason?: string
  ): Promise<BlockResult> {
    // 1. Check limit
    const blockCount = await this.getBlockCount(playerId);
    if (blockCount >= FRIEND_LIMITS.maxBlockedPlayers) {
      return {
        success: false,
        error: 'limit_reached',
        message: 'Maximum blocked players reached'
      };
    }

    // 2. Remove any existing friend relationship
    const relationship = await FriendService.getRelationship(playerId, targetId);
    if (relationship) {
      await FriendService.deleteRelationship(relationship.id);
    }

    // 3. Create block record
    const block: BlockedPlayer = {
      playerId,
      blockedPlayerId: targetId,
      reason,
      blockedAt: new Date().toISOString()
    };
    await this.createBlock(block);

    // 4. Effects of blocking
    await this.applyBlockEffects(playerId, targetId);

    return { success: true };
  }

  private async applyBlockEffects(
    blockerId: string,
    blockedId: string
  ): Promise<void> {
    // Remove from party if in same party
    const party = await PartyService.getPlayerParty(blockerId);
    if (party && party.members.includes(blockedId)) {
      await PartyService.kickMember(party.id, blockedId, 'blocked');
    }

    // Cancel any pending friend requests
    await FriendRequestService.cancelPendingBetween(blockerId, blockedId);

    // Remove from recent players display (optional, for privacy)
    // The blocked player still sees the blocker in their recent
  }

  async unblockPlayer(
    playerId: string,
    targetId: string
  ): Promise<void> {
    const block = await this.getBlock(playerId, targetId);
    if (!block) {
      throw new Error('Player not blocked');
    }

    await this.deleteBlock(block);
    // Note: Unblocking does not restore friendship
  }

  async getBlockedPlayers(playerId: string): Promise<BlockedPlayer[]> {
    return this.fetchBlocks(playerId);
  }

  // Check if interaction is blocked (either direction)
  async isBlocked(playerA: string, playerB: string): Promise<boolean> {
    const [aBlocksB, bBlocksA] = await Promise.all([
      this.getBlock(playerA, playerB),
      this.getBlock(playerB, playerA)
    ]);
    return !!(aBlocksB || bBlocksA);
  }
}
```

### Block Effects

```typescript
const BLOCK_EFFECTS = {
  // Matchmaking: Blocked players should not be matched together
  matchmaking: {
    preventTeammates: true,
    preventOpponents: false // Too easy to abuse
  },

  // Social features
  social: {
    hideFriendRequest: true,
    hidePartyInvite: true,
    hideMessages: true,
    hideFromSearch: true
  },

  // In-game
  inGame: {
    muteVoiceChat: true,
    hideTextChat: true,
    hideEmotes: false // Can still see their character emote
  },

  // Profile
  profile: {
    hideFromViewers: false, // Public profile still visible
    hideActivity: true // Don't show what they're doing
  }
};
```

---

## Platform Friend Import

### Integration with Platform Services

```typescript
interface PlatformFriend {
  platformId: string;
  platformType: 'game_center' | 'google_play' | 'facebook';
  displayName: string;
  avatar?: string;
}

class PlatformFriendService {
  // Import friends from platform
  async importPlatformFriends(
    playerId: string,
    platform: 'game_center' | 'google_play' | 'facebook'
  ): Promise<ImportResult> {
    // 1. Get platform friends
    const platformFriends = await this.fetchPlatformFriends(playerId, platform);

    // 2. Find matches in our system
    const matches: PlatformFriendMatch[] = [];
    for (const pf of platformFriends) {
      const linkedPlayer = await this.findLinkedPlayer(pf.platformId, platform);
      if (linkedPlayer) {
        matches.push({
          platformFriend: pf,
          gamePlayer: linkedPlayer
        });
      }
    }

    // 3. Filter out existing friends
    const existingFriends = await FriendService.getFriendIds(playerId);
    const newMatches = matches.filter(
      m => !existingFriends.includes(m.gamePlayer.id)
    );

    // 4. Filter out blocked players
    const blocked = await BlockService.getBlockedIds(playerId);
    const availableMatches = newMatches.filter(
      m => !blocked.includes(m.gamePlayer.id)
    );

    return {
      totalPlatformFriends: platformFriends.length,
      matchesFound: matches.length,
      alreadyFriends: matches.length - newMatches.length,
      available: availableMatches
    };
  }

  // Bulk send friend requests
  async bulkSendRequests(
    playerId: string,
    targetIds: string[]
  ): Promise<BulkRequestResult> {
    const results: BulkRequestResult = {
      sent: [],
      failed: [],
      skipped: []
    };

    for (const targetId of targetIds) {
      try {
        const result = await FriendRequestService.sendFriendRequest(playerId, {
          targetType: 'player_id',
          targetValue: targetId
        });

        if (result.success) {
          results.sent.push(targetId);
        } else {
          results.failed.push({ targetId, reason: result.error });
        }
      } catch (error) {
        results.failed.push({ targetId, reason: error.message });
      }
    }

    return results;
  }

  // iOS Game Center integration
  private async fetchGameCenterFriends(playerId: string): Promise<PlatformFriend[]> {
    // This would use native Game Center APIs
    const gcToken = await this.getGameCenterToken(playerId);
    const friends = await GameCenterAPI.getFriends(gcToken);

    return friends.map(f => ({
      platformId: f.gamePlayerId,
      platformType: 'game_center',
      displayName: f.displayName,
      avatar: f.avatar
    }));
  }

  // Android Google Play Games integration
  private async fetchGooglePlayFriends(playerId: string): Promise<PlatformFriend[]> {
    const gpToken = await this.getGooglePlayToken(playerId);
    const friends = await GooglePlayGamesAPI.getFriends(gpToken);

    return friends.map(f => ({
      platformId: f.playerId,
      platformType: 'google_play',
      displayName: f.displayName,
      avatar: f.iconImageUrl
    }));
  }
}
```

---

## Social Interactions

### Invite to Party

```typescript
class PartyInviteService {
  async inviteFriendToParty(
    inviterId: string,
    friendId: string
  ): Promise<InviteResult> {
    // 1. Verify friendship
    const areFriends = await FriendService.areFriends(inviterId, friendId);
    if (!areFriends) {
      return {
        success: false,
        error: 'not_friends',
        message: 'You can only invite friends to your party'
      };
    }

    // 2. Check friend's settings
    const friendSettings = await this.getPrivacySettings(friendId);
    if (!this.canReceiveInvite(friendSettings, inviterId)) {
      return {
        success: false,
        error: 'invites_disabled',
        message: 'This friend is not accepting party invites'
      };
    }

    // 3. Get or create party
    let party = await PartyService.getPlayerParty(inviterId);
    if (!party) {
      party = await PartyService.createParty(inviterId);
    }

    // 4. Check party has room
    if (party.members.length >= party.maxSize) {
      return {
        success: false,
        error: 'party_full',
        message: 'Your party is full'
      };
    }

    // 5. Send invite
    const invite = await PartyService.createInvite(party.id, friendId, inviterId);

    // 6. Send notification
    await NotificationService.send(friendId, {
      type: 'party_invite',
      data: {
        partyId: party.id,
        inviterId,
        inviterName: await this.getDisplayName(inviterId),
        inviteId: invite.id
      }
    });

    return {
      success: true,
      inviteId: invite.id
    };
  }
}
```

### Join Friend's Game

```typescript
class JoinFriendService {
  async joinFriend(
    playerId: string,
    friendId: string
  ): Promise<JoinResult> {
    // 1. Verify friendship
    const areFriends = await FriendService.areFriends(playerId, friendId);
    if (!areFriends) {
      throw new Error('Not friends');
    }

    // 2. Check friend's presence
    const presence = await PresenceService.getPresence(friendId);
    if (!presence.currentActivity?.canJoin) {
      return {
        success: false,
        error: 'cannot_join',
        message: 'Your friend is not in a joinable state'
      };
    }

    // 3. Attempt to join based on activity type
    switch (presence.currentActivity.type) {
      case 'party':
        return this.joinParty(playerId, presence.currentActivity.partyId);

      case 'matchmaking':
        // Can only join if friend is in a party with room
        if (presence.currentActivity.partyId) {
          return this.joinParty(playerId, presence.currentActivity.partyId);
        }
        return {
          success: false,
          error: 'cannot_join',
          message: 'Your friend is in solo queue'
        };

      default:
        return {
          success: false,
          error: 'cannot_join',
          message: 'Your friend is not in a joinable state'
        };
    }
  }

  private async joinParty(
    playerId: string,
    partyId: string
  ): Promise<JoinResult> {
    const party = await PartyService.getParty(partyId);

    if (!party) {
      return {
        success: false,
        error: 'party_not_found'
      };
    }

    if (party.members.length >= party.maxSize) {
      return {
        success: false,
        error: 'party_full'
      };
    }

    if (!party.settings.allowJoin) {
      return {
        success: false,
        error: 'party_closed'
      };
    }

    await PartyService.addMember(partyId, playerId);

    return {
      success: true,
      partyId
    };
  }
}
```

---

## Privacy Settings

### Privacy Configuration

```typescript
interface SocialPrivacySettings {
  // Friend requests
  allowFriendRequests: 'everyone' | 'friends_of_friends' | 'nobody';

  // Party invites
  allowPartyInvites: 'everyone' | 'friends' | 'nobody';

  // Join game
  allowJoinGame: 'everyone' | 'friends' | 'nobody';

  // Profile visibility
  profileVisibility: 'public' | 'friends' | 'private';

  // Online status
  showOnlineStatus: boolean;
  showActivityStatus: boolean;

  // Recent players
  appearInRecentPlayers: boolean;

  // Platform friends
  syncPlatformFriends: boolean;
}

const DEFAULT_PRIVACY_SETTINGS: SocialPrivacySettings = {
  allowFriendRequests: 'everyone',
  allowPartyInvites: 'friends',
  allowJoinGame: 'friends',
  profileVisibility: 'public',
  showOnlineStatus: true,
  showActivityStatus: true,
  appearInRecentPlayers: true,
  syncPlatformFriends: true
};
```

---

## API Endpoints

```typescript
// Friend Management
POST   /api/friends/request          // Send friend request
POST   /api/friends/accept/:id       // Accept request
POST   /api/friends/decline/:id      // Decline request
DELETE /api/friends/:friendId        // Remove friend
GET    /api/friends                  // Get friends list
GET    /api/friends/requests         // Get pending requests
PUT    /api/friends/:friendId/nickname  // Set nickname
PUT    /api/friends/:friendId/favorite  // Toggle favorite

// Presence
GET    /api/presence/:playerId       // Get player presence
GET    /api/presence/friends         // Get all friends' presence
WS     /ws/presence                  // Real-time presence updates

// Recent Players
GET    /api/recent-players           // Get recent players list
POST   /api/recent-players/:id/add-friend  // Quick add from recent

// Blocking
POST   /api/block/:playerId          // Block player
DELETE /api/block/:playerId          // Unblock player
GET    /api/blocked                  // Get blocked list

// Platform Integration
POST   /api/platform-friends/import  // Import platform friends
GET    /api/platform-friends/matches // Get matched platform friends

// Social Actions
POST   /api/social/invite-to-party   // Invite friend to party
POST   /api/social/join-friend       // Join friend's game/party
```

---

## Performance Considerations

### Caching Strategy

```typescript
const CACHE_CONFIG = {
  // Friends list
  friendsList: {
    ttl: 300, // 5 minutes
    invalidateOn: ['friend_added', 'friend_removed', 'nickname_changed']
  },

  // Presence
  presence: {
    ttl: 30, // 30 seconds
    useRealTimeUpdates: true
  },

  // Recent players
  recentPlayers: {
    ttl: 3600, // 1 hour
    invalidateOn: ['match_completed']
  },

  // Block list
  blockList: {
    ttl: 3600, // 1 hour
    invalidateOn: ['block_added', 'block_removed']
  }
};
```

### Batch Operations

```typescript
class BatchOperations {
  // Batch presence fetch for friends list
  async batchGetPresence(playerIds: string[]): Promise<PlayerPresence[]> {
    const cached = await this.getCachedPresences(playerIds);
    const missing = playerIds.filter(id => !cached.has(id));

    if (missing.length > 0) {
      const fetched = await this.fetchPresencesBatch(missing);
      fetched.forEach(p => cached.set(p.playerId, p));
    }

    return playerIds.map(id => cached.get(id));
  }

  // Batch profile fetch
  async batchGetProfiles(playerIds: string[]): Promise<Map<string, PlayerProfile>> {
    return PlayerService.batchGetProfiles(playerIds);
  }
}
```

---

## Security Measures

### Rate Limiting

```typescript
const RATE_LIMITS = {
  friendRequests: {
    limit: 20,
    window: 3600000, // 1 hour
    message: 'Too many friend requests. Please try again later.'
  },

  blockOperations: {
    limit: 50,
    window: 86400000, // 24 hours
    message: 'Block limit reached. Please try again tomorrow.'
  },

  platformImport: {
    limit: 5,
    window: 86400000, // 24 hours
    message: 'Platform import limit reached.'
  }
};
```

### Spam Prevention

```typescript
class SpamPrevention {
  async checkRequestSpam(senderId: string): Promise<boolean> {
    // Check recent request count
    const recentCount = await this.getRecentRequestCount(senderId, 3600000);
    if (recentCount > 10) {
      // Flag for review
      await this.flagAccount(senderId, 'excessive_friend_requests');
      return true;
    }

    // Check declined rate
    const declinedRate = await this.getDeclinedRate(senderId);
    if (declinedRate > 0.8) { // 80% of requests declined
      await this.flagAccount(senderId, 'high_decline_rate');
      return true;
    }

    return false;
  }
}
```

---

## Integration Points

### Dependencies

- BACK-010: Authentication System (player identity)
- BACK-011: Player Profile Service (profile data)
- BACK-002: Party/Squad System (party invites)
- Push notification service
- WebSocket service for real-time updates

### Event Hooks

```typescript
// Events emitted by this service
const FRIEND_EVENTS = [
  'friend_request_sent',
  'friend_request_accepted',
  'friend_request_declined',
  'friend_removed',
  'player_blocked',
  'player_unblocked',
  'presence_changed',
  'friend_online',
  'friend_offline'
];
```
