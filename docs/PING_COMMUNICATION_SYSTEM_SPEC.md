# Ping/Communication System Specification

## Overview

This document specifies the Ping/Communication System for Plunderstorm Mobile, enabling quick team communication through contextual pings, preset messages, and voice chat integration. Designed for seamless mobile interaction without requiring a keyboard.

## Communication Methods

### Method Overview

| Method | Description | Best For |
|--------|-------------|----------|
| Quick Ping | Single tap contextual ping | Instant callouts |
| Ping Wheel | Hold to select ping type | Specific communication |
| Preset Messages | Quick text communication | Complex info |
| Voice Chat | Real-time voice | Full coordination |

## Ping System

### Ping Types

```typescript
interface PingType {
  id: string;
  name: string;
  icon: string;
  color: string;
  voiceLine: string;
  duration: number;       // How long ping stays visible (ms)
  cooldown: number;       // Per-type cooldown (ms)
  priority: number;       // For stacking/override logic
}

const PING_TYPES: PingType[] = [
  {
    id: 'generic',
    name: 'Look Here',
    icon: 'ping_generic',
    color: '#FFFFFF',
    voiceLine: 'Look here!',
    duration: 5000,
    cooldown: 1000,
    priority: 1
  },
  {
    id: 'enemy',
    name: 'Enemy Spotted',
    icon: 'ping_enemy',
    color: '#FF4444',
    voiceLine: 'Enemy spotted!',
    duration: 6000,
    cooldown: 2000,
    priority: 3
  },
  {
    id: 'loot',
    name: 'Loot Here',
    icon: 'ping_loot',
    color: '#FFD700',
    voiceLine: 'Loot over here!',
    duration: 8000,
    cooldown: 1500,
    priority: 2
  },
  {
    id: 'danger',
    name: 'Danger',
    icon: 'ping_danger',
    color: '#FF0000',
    voiceLine: 'Danger! Watch out!',
    duration: 5000,
    cooldown: 2000,
    priority: 4
  },
  {
    id: 'help',
    name: 'Need Help',
    icon: 'ping_help',
    color: '#00FFFF',
    voiceLine: 'I need help!',
    duration: 6000,
    cooldown: 3000,
    priority: 5
  },
  {
    id: 'going',
    name: 'Going Here',
    icon: 'ping_going',
    color: '#00FF00',
    voiceLine: 'I\'m heading here!',
    duration: 10000,
    cooldown: 2000,
    priority: 2
  },
  {
    id: 'defend',
    name: 'Defend Here',
    icon: 'ping_defend',
    color: '#4444FF',
    voiceLine: 'Defend this position!',
    duration: 8000,
    cooldown: 3000,
    priority: 3
  },
  {
    id: 'retreat',
    name: 'Retreat',
    icon: 'ping_retreat',
    color: '#FF8800',
    voiceLine: 'Fall back!',
    duration: 5000,
    cooldown: 2000,
    priority: 4
  }
];
```

### Ping Data Model

```typescript
interface Ping {
  id: string;
  type: PingType;
  playerId: string;
  playerName: string;
  position: Vector3;
  timestamp: number;
  expiresAt: number;
  targetEntity?: {
    type: 'enemy' | 'loot' | 'chest' | 'objective';
    entityId: string;
    trackPosition: boolean;  // Follow moving targets
  };
}

interface PingState {
  activePings: Map<string, Ping>;
  playerCooldowns: Map<string, Map<string, number>>;  // playerId -> pingTypeId -> cooldownEnd
  maxPingsPerPlayer: 3;
  globalCooldown: 500;  // ms between any ping
}
```

### Contextual Ping Logic

```typescript
function getContextualPingType(
  raycastHit: RaycastResult,
  playerState: PlayerState
): PingType {
  // Priority order for automatic ping type selection

  // 1. Enemy player or NPC
  if (raycastHit.entity?.type === 'player' && raycastHit.entity.team !== playerState.team) {
    return PING_TYPES.find(p => p.id === 'enemy')!;
  }
  if (raycastHit.entity?.type === 'npc') {
    return PING_TYPES.find(p => p.id === 'enemy')!;
  }

  // 2. Loot item on ground
  if (raycastHit.entity?.type === 'loot') {
    return PING_TYPES.find(p => p.id === 'loot')!;
  }

  // 3. Chest
  if (raycastHit.entity?.type === 'chest') {
    return PING_TYPES.find(p => p.id === 'loot')!;
  }

  // 4. Storm zone edge
  if (isPositionInStorm(raycastHit.position)) {
    return PING_TYPES.find(p => p.id === 'danger')!;
  }

  // 5. Default: generic ping
  return PING_TYPES.find(p => p.id === 'generic')!;
}

function createPing(
  player: Player,
  position: Vector3,
  pingType: PingType,
  targetEntity?: Entity
): Ping | null {
  const state = getPingState();

  // Check global cooldown
  const lastPingTime = getPlayerLastPingTime(player.id);
  if (Date.now() - lastPingTime < state.globalCooldown) {
    return null;
  }

  // Check type-specific cooldown
  const typeCooldown = state.playerCooldowns.get(player.id)?.get(pingType.id) || 0;
  if (Date.now() < typeCooldown) {
    return null;
  }

  // Check max pings per player
  const playerPings = getPlayerActivePings(player.id);
  if (playerPings.length >= state.maxPingsPerPlayer) {
    // Remove oldest ping
    removeOldestPing(player.id);
  }

  // Create ping
  const ping: Ping = {
    id: generateId(),
    type: pingType,
    playerId: player.id,
    playerName: player.displayName,
    position: position,
    timestamp: Date.now(),
    expiresAt: Date.now() + pingType.duration,
    targetEntity: targetEntity ? {
      type: getEntityPingType(targetEntity),
      entityId: targetEntity.id,
      trackPosition: targetEntity.type === 'player'
    } : undefined
  };

  // Set cooldowns
  setPlayerCooldown(player.id, pingType.id, Date.now() + pingType.cooldown);
  setPlayerLastPingTime(player.id, Date.now());

  return ping;
}
```

## Ping Wheel UI

### Mobile Ping Wheel

```
Hold to open (150ms threshold):

              [Enemy]
                 ↑
     [Loot]    ╱   ╲    [Danger]
              ╱     ╲
    [Help] ← ●───────→ [Going]
              ╲     ╱
    [Defend]   ╲   ╱   [Retreat]
                 ↓
             [Generic]

- Touch center = Cancel
- Drag to segment = Select
- Release = Confirm & Place

Visual states:
- Default: Semi-transparent segments
- Hover: Highlighted segment with label
- Cooldown: Grayed out with timer
- Selected: Bright highlight
```

### Ping Wheel Implementation

```typescript
interface PingWheelConfig {
  holdThreshold: 150;       // ms to open wheel
  radius: 120;              // pixels from center
  segmentCount: 8;          // Number of ping types
  cancelZoneRadius: 30;     // Center cancel zone
  feedbackHaptic: boolean;
}

interface PingWheelState {
  isOpen: boolean;
  selectedSegment: number | null;
  touchStartPosition: Vector2;
  touchCurrentPosition: Vector2;
  openTimestamp: number;
}

function updatePingWheel(touch: TouchState, state: PingWheelState): void {
  if (!state.isOpen) return;

  const delta = touch.position.subtract(state.touchStartPosition);
  const distance = delta.magnitude();
  const angle = Math.atan2(delta.y, delta.x);

  if (distance < PING_WHEEL_CONFIG.cancelZoneRadius) {
    // In cancel zone
    state.selectedSegment = null;
  } else {
    // Determine segment
    const normalizedAngle = (angle + Math.PI) / (2 * Math.PI);
    state.selectedSegment = Math.floor(normalizedAngle * PING_WHEEL_CONFIG.segmentCount);

    // Haptic feedback on segment change
    if (PING_WHEEL_CONFIG.feedbackHaptic) {
      triggerHaptic('light');
    }
  }
}

function closePingWheel(state: PingWheelState): void {
  if (state.selectedSegment !== null) {
    const pingType = PING_TYPES[state.selectedSegment];
    const worldPosition = screenToWorld(state.touchStartPosition);
    createPing(localPlayer, worldPosition, pingType);

    // Play voice line
    playVoiceLine(pingType.voiceLine);

    // Haptic confirm
    triggerHaptic('medium');
  }

  state.isOpen = false;
  state.selectedSegment = null;
}
```

## Ping Visual Display

### 3D World Pings

```typescript
interface WorldPingVisual {
  ping: Ping;
  iconSprite: Sprite3D;
  beamEffect: ParticleSystem;
  groundDecal: Decal;
  distanceLabel: TextMesh;
  playerNameLabel: TextMesh;
}

const PING_VISUAL_CONFIG = {
  iconSize: 1.5,            // meters
  iconElevation: 3.0,       // meters above ground
  beamWidth: 0.2,           // meters
  beamHeight: 10.0,         // meters
  decalRadius: 1.0,         // meters
  fadeStartTime: 0.8,       // Start fading at 80% duration
  bobAmplitude: 0.2,        // meters
  bobFrequency: 1.5,        // Hz
  maxVisibleDistance: 200,  // meters
  labelShowDistance: 50     // Show text label within this distance
};

function updateWorldPing(visual: WorldPingVisual, deltaTime: number): void {
  const ping = visual.ping;
  const timeRemaining = ping.expiresAt - Date.now();
  const totalDuration = ping.type.duration;

  // Track moving targets
  if (ping.targetEntity?.trackPosition) {
    const entity = getEntity(ping.targetEntity.entityId);
    if (entity) {
      ping.position = entity.position;
      visual.iconSprite.position = entity.position.add(Vector3.up * PING_VISUAL_CONFIG.iconElevation);
    }
  }

  // Bob animation
  const bobOffset = Math.sin(Date.now() * 0.001 * PING_VISUAL_CONFIG.bobFrequency * Math.PI * 2)
                    * PING_VISUAL_CONFIG.bobAmplitude;
  visual.iconSprite.position.y += bobOffset;

  // Fade out near end
  const fadeProgress = timeRemaining / (totalDuration * (1 - PING_VISUAL_CONFIG.fadeStartTime));
  if (fadeProgress < 1) {
    visual.iconSprite.alpha = fadeProgress;
    visual.beamEffect.alpha = fadeProgress;
    visual.groundDecal.alpha = fadeProgress;
  }

  // Update distance label
  const distanceToPlayer = Vector3.distance(localPlayer.position, ping.position);
  visual.distanceLabel.text = `${Math.round(distanceToPlayer)}m`;
  visual.distanceLabel.visible = distanceToPlayer <= PING_VISUAL_CONFIG.labelShowDistance;

  // Billboard to face camera
  visual.iconSprite.lookAt(Camera.main.position);
}
```

### Minimap Ping Display

```typescript
interface MinimapPingConfig {
  iconSize: 12;             // pixels
  pulseRate: 2.0;           // Hz
  pulseScale: 1.3;          // max scale during pulse
  showDirectionArrow: true; // Arrow pointing to off-screen pings
}

function renderMinimapPing(ping: Ping, minimapContext: CanvasContext): void {
  const mapPos = worldToMinimapPosition(ping.position);

  // Check if on minimap
  if (isOnMinimap(mapPos)) {
    // Draw ping icon
    const pulseScale = 1 + (Math.sin(Date.now() * 0.001 * MINIMAP_PING_CONFIG.pulseRate * Math.PI * 2) + 1)
                       * 0.5 * (MINIMAP_PING_CONFIG.pulseScale - 1);

    minimapContext.save();
    minimapContext.translate(mapPos.x, mapPos.y);
    minimapContext.scale(pulseScale, pulseScale);

    drawIcon(minimapContext, ping.type.icon, ping.type.color, MINIMAP_PING_CONFIG.iconSize);

    minimapContext.restore();
  } else if (MINIMAP_PING_CONFIG.showDirectionArrow) {
    // Draw edge arrow pointing to ping
    const edgePos = getMinimapEdgePosition(mapPos);
    const angle = Math.atan2(mapPos.y - minimapCenter.y, mapPos.x - minimapCenter.x);

    drawDirectionArrow(minimapContext, edgePos, angle, ping.type.color);
  }
}
```

## Audio System

### Voice Lines

```typescript
interface VoiceLineConfig {
  volume: number;
  priority: number;
  cooldown: number;        // Prevent spam
  spatialize: boolean;     // 3D audio from ping location
}

const VOICE_LINE_CONFIG: Record<string, VoiceLineConfig> = {
  'Look here!': { volume: 0.8, priority: 1, cooldown: 2000, spatialize: false },
  'Enemy spotted!': { volume: 1.0, priority: 3, cooldown: 3000, spatialize: true },
  'Loot over here!': { volume: 0.7, priority: 1, cooldown: 2000, spatialize: true },
  'Danger! Watch out!': { volume: 1.0, priority: 4, cooldown: 3000, spatialize: false },
  'I need help!': { volume: 1.0, priority: 5, cooldown: 5000, spatialize: false },
  'I\'m heading here!': { volume: 0.7, priority: 2, cooldown: 2000, spatialize: false },
  'Defend this position!': { volume: 0.9, priority: 3, cooldown: 3000, spatialize: true },
  'Fall back!': { volume: 1.0, priority: 4, cooldown: 3000, spatialize: false }
};

// Character-specific voice lines
interface CharacterVoices {
  characterId: string;
  voiceLines: Map<string, AudioClip[]>;  // Multiple variants per line
}

function playPingVoiceLine(ping: Ping): void {
  const config = VOICE_LINE_CONFIG[ping.type.voiceLine];

  // Check cooldown
  if (isVoiceLineCoolingDown(ping.playerId, ping.type.voiceLine)) {
    return;
  }

  // Get character-specific clip
  const character = getPlayerCharacter(ping.playerId);
  const clips = character.voiceLines.get(ping.type.voiceLine);
  const clip = clips[Math.floor(Math.random() * clips.length)];

  // Play audio
  if (config.spatialize) {
    playAudio3D(clip, ping.position, config.volume);
  } else {
    playAudio2D(clip, config.volume);
  }

  // Set cooldown
  setVoiceLineCooldown(ping.playerId, ping.type.voiceLine, config.cooldown);
}
```

### Notification Sounds

```typescript
const PING_SOUNDS = {
  pingReceived: 'sfx_ping_received',      // When teammate pings
  pingPlaced: 'sfx_ping_placed',          // When you ping
  pingExpired: 'sfx_ping_fade',           // Subtle fade out
  wheelOpen: 'sfx_wheel_open',            // Opening ping wheel
  wheelSelect: 'sfx_wheel_select',        // Hovering segment
  wheelConfirm: 'sfx_wheel_confirm'       // Confirming selection
};

interface PingAudioConfig {
  pingReceivedVolume: 0.6;
  pingPlacedVolume: 0.4;
  priorityPingsLouder: true;  // Enemy/Danger pings louder
}
```

## Preset Messages

### Message Categories

```typescript
interface PresetMessage {
  id: string;
  category: MessageCategory;
  text: string;
  shortcut?: string;        // Quick access key
  voiceLine?: string;
}

enum MessageCategory {
  TACTICAL = 'tactical',
  STATUS = 'status',
  SOCIAL = 'social',
  QUICK = 'quick'
}

const PRESET_MESSAGES: PresetMessage[] = [
  // Tactical
  { id: 'attack', category: MessageCategory.TACTICAL, text: 'Attack!', voiceLine: 'Attack now!' },
  { id: 'wait', category: MessageCategory.TACTICAL, text: 'Wait', voiceLine: 'Hold position!' },
  { id: 'group_up', category: MessageCategory.TACTICAL, text: 'Group Up', voiceLine: 'Group up with me!' },
  { id: 'split_up', category: MessageCategory.TACTICAL, text: 'Split Up', voiceLine: 'Split up!' },
  { id: 'flank', category: MessageCategory.TACTICAL, text: 'Flanking', voiceLine: 'I\'m flanking!' },
  { id: 'push', category: MessageCategory.TACTICAL, text: 'Push Now', voiceLine: 'Push now!' },

  // Status
  { id: 'low_health', category: MessageCategory.STATUS, text: 'Low Health', voiceLine: 'I\'m hurt!' },
  { id: 'need_ability', category: MessageCategory.STATUS, text: 'Need Ability', voiceLine: 'Looking for abilities!' },
  { id: 'reloading', category: MessageCategory.STATUS, text: 'Healing', voiceLine: 'Healing up!' },
  { id: 'ready', category: MessageCategory.STATUS, text: 'Ready', voiceLine: 'Ready!' },
  { id: 'on_my_way', category: MessageCategory.STATUS, text: 'On My Way', voiceLine: 'On my way!' },

  // Social
  { id: 'thanks', category: MessageCategory.SOCIAL, text: 'Thanks!', voiceLine: 'Thanks, mate!' },
  { id: 'sorry', category: MessageCategory.SOCIAL, text: 'Sorry', voiceLine: 'My bad!' },
  { id: 'good_game', category: MessageCategory.SOCIAL, text: 'Good Game', voiceLine: 'Good game!' },
  { id: 'nice_shot', category: MessageCategory.SOCIAL, text: 'Nice Shot!', voiceLine: 'Nice shot!' },

  // Quick (most common, accessible from HUD)
  { id: 'yes', category: MessageCategory.QUICK, text: 'Yes', shortcut: '1' },
  { id: 'no', category: MessageCategory.QUICK, text: 'No', shortcut: '2' },
  { id: 'help', category: MessageCategory.QUICK, text: 'Help!', shortcut: '3' }
];
```

### Message UI

```
Quick Message Menu (tap chat icon):

┌─────────────────────────────────────────┐
│  QUICK MESSAGES                         │
├─────────────────────────────────────────┤
│  [Tactical]  [Status]  [Social]         │
├─────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │ Attack! │  │  Wait   │  │Group Up │ │
│  └─────────┘  └─────────┘  └─────────┘ │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │Split Up │  │Flanking │  │Push Now │ │
│  └─────────┘  └─────────┘  └─────────┘ │
├─────────────────────────────────────────┤
│  [X Close]           [Favorites ★]      │
└─────────────────────────────────────────┘
```

### Message Display

```typescript
interface ChatMessage {
  id: string;
  playerId: string;
  playerName: string;
  teamColor: string;
  text: string;
  timestamp: number;
  type: 'preset' | 'system' | 'voice_transcript';
}

interface ChatDisplayConfig {
  maxVisibleMessages: 4;
  messageDuration: 5000;      // How long message stays visible
  fadeOutDuration: 500;
  position: 'top-left' | 'bottom-left';
  fontSize: 14;
  showInCombat: true;
  hideDelay: 3000;            // Hide chat area after inactivity
}

function displayChatMessage(message: ChatMessage): void {
  const chatUI = getChatUI();

  // Add message to feed
  chatUI.messages.push(message);

  // Trim old messages
  while (chatUI.messages.length > CHAT_CONFIG.maxVisibleMessages) {
    chatUI.messages.shift();
  }

  // Show chat area
  chatUI.visible = true;
  chatUI.hideTimer = Date.now() + CHAT_CONFIG.hideDelay;

  // Render
  renderChatFeed(chatUI);
}
```

## Voice Chat

### Voice Chat System

```typescript
interface VoiceChatConfig {
  enabled: boolean;
  mode: 'push_to_talk' | 'voice_activation' | 'always_on';
  inputSensitivity: number;   // 0-100 for voice activation
  outputVolume: number;       // 0-100
  teamOnly: true;             // No all-chat
  spatialAudio: false;        // Same volume regardless of distance
  noiseReduction: boolean;
  echoCancellation: boolean;
}

interface VoiceChatState {
  isTransmitting: boolean;
  isMuted: boolean;
  mutedPlayers: Set<string>;
  activeVoices: Map<string, VoiceStream>;
}

interface VoiceStream {
  playerId: string;
  audioSource: AudioSource;
  isSpeaking: boolean;
  volume: number;
}
```

### Push-to-Talk Implementation

```typescript
interface PushToTalkConfig {
  buttonPosition: Vector2;
  buttonSize: 60;             // pixels
  holdThreshold: 50;          // ms before transmit starts
  releaseDelay: 200;          // ms buffer after release
}

class PushToTalkButton {
  private isPressed: boolean = false;
  private pressStartTime: number = 0;

  onTouchDown(touch: Touch): void {
    this.isPressed = true;
    this.pressStartTime = Date.now();

    // Visual feedback
    this.setButtonState('pressed');
    triggerHaptic('light');
  }

  onTouchUp(touch: Touch): void {
    this.isPressed = false;

    // Stop transmitting after delay
    setTimeout(() => {
      if (!this.isPressed) {
        stopVoiceTransmission();
        this.setButtonState('idle');
      }
    }, PTT_CONFIG.releaseDelay);
  }

  update(): void {
    if (this.isPressed && !isTransmitting()) {
      // Check hold threshold
      if (Date.now() - this.pressStartTime >= PTT_CONFIG.holdThreshold) {
        startVoiceTransmission();
        this.setButtonState('transmitting');
      }
    }
  }
}
```

### Voice Chat UI

```
Voice Chat HUD Element:

┌────────────────────┐
│ 🎤 [PTT Button]    │  ← Push-to-talk (when enabled)
└────────────────────┘

Speaking Indicator (shows when teammate speaks):
┌────────────────────┐
│ 🔊 TreasureHunter  │  ← Animated sound waves
│ 🔊 CaptainJack     │
└────────────────────┘

Muted Player (in scoreboard):
┌──────────────────────────────────────┐
│ TreasureHunter  [🔇]  [Mute] [Block] │
└──────────────────────────────────────┘
```

### Voice Chat Quality

```typescript
interface VoiceQualityConfig {
  sampleRate: 16000;          // Hz
  bitrate: 24000;             // bps (Opus codec)
  frameSize: 20;              // ms
  maxLatency: 150;            // ms target
  jitterBufferSize: 3;        // frames
}

function processVoiceAudio(audioData: Float32Array): Uint8Array {
  // Apply noise reduction
  if (VOICE_CONFIG.noiseReduction) {
    audioData = applyNoiseReduction(audioData);
  }

  // Apply echo cancellation
  if (VOICE_CONFIG.echoCancellation) {
    audioData = applyEchoCancellation(audioData);
  }

  // Encode with Opus
  const encoded = opusEncoder.encode(audioData);

  return encoded;
}
```

## Anti-Spam System

### Ping Spam Prevention

```typescript
interface SpamPreventionConfig {
  maxPingsPerMinute: 15;
  maxMessagesPerMinute: 20;
  warningThreshold: 0.8;      // Show warning at 80% of limit
  muteThreshold: 1.0;         // Auto-mute at 100%
  muteDuration: 60000;        // 1 minute mute
  escalatingMute: true;       // Longer mute for repeat offenders
}

class SpamDetector {
  private pingHistory: Map<string, number[]> = new Map();
  private messageHistory: Map<string, number[]> = new Map();
  private muteHistory: Map<string, number> = new Map();

  checkPingAllowed(playerId: string): SpamCheckResult {
    const history = this.pingHistory.get(playerId) || [];
    const recentPings = history.filter(t => Date.now() - t < 60000);

    const usage = recentPings.length / SPAM_CONFIG.maxPingsPerMinute;

    if (usage >= SPAM_CONFIG.muteThreshold) {
      this.mutePlayer(playerId);
      return { allowed: false, reason: 'spam_muted' };
    }

    if (usage >= SPAM_CONFIG.warningThreshold) {
      return { allowed: true, warning: 'approaching_limit' };
    }

    return { allowed: true };
  }

  mutePlayer(playerId: string): void {
    const muteCount = this.muteHistory.get(playerId) || 0;
    const muteDuration = SPAM_CONFIG.muteDuration * Math.pow(2, muteCount);

    setCommunicationMuted(playerId, muteDuration);
    this.muteHistory.set(playerId, muteCount + 1);

    // Notify player
    showNotification(playerId, `Communication muted for ${muteDuration / 1000}s (spam protection)`);
  }
}
```

### Report System

```typescript
interface ReportOptions {
  reportTypes: ['spam', 'harassment', 'cheating', 'offensive_name'];
  requireReason: boolean;
  blockAfterReport: boolean;
  reportCooldown: 60000;      // Per-player cooldown
}

function reportPlayer(
  reporterId: string,
  targetId: string,
  type: string,
  reason?: string
): void {
  const report = {
    id: generateId(),
    reporterId,
    targetId,
    type,
    reason,
    timestamp: Date.now(),
    matchId: getCurrentMatchId(),
    evidence: {
      recentPings: getRecentPings(targetId),
      recentMessages: getRecentMessages(targetId)
    }
  };

  // Send to moderation system
  submitReport(report);

  // Auto-block if configured
  if (REPORT_CONFIG.blockAfterReport) {
    blockPlayer(reporterId, targetId);
  }

  // Acknowledge
  showNotification(reporterId, 'Report submitted. Thank you for helping keep the game safe.');
}
```

## Mute/Block System

### Player Mute

```typescript
interface MuteState {
  mutedPlayers: Set<string>;      // All communication blocked
  voiceMutedPlayers: Set<string>; // Voice only blocked
  pingMutedPlayers: Set<string>;  // Pings only blocked
}

function mutePlayer(targetId: string, muteType: 'all' | 'voice' | 'ping'): void {
  const state = getMuteState();

  switch (muteType) {
    case 'all':
      state.mutedPlayers.add(targetId);
      state.voiceMutedPlayers.add(targetId);
      state.pingMutedPlayers.add(targetId);
      break;
    case 'voice':
      state.voiceMutedPlayers.add(targetId);
      break;
    case 'ping':
      state.pingMutedPlayers.add(targetId);
      break;
  }

  // Persist to account
  saveMuteSettings();

  // Show confirmation
  showNotification(`Muted ${getPlayerName(targetId)}`);
}

function shouldShowPing(ping: Ping): boolean {
  const state = getMuteState();
  return !state.mutedPlayers.has(ping.playerId) &&
         !state.pingMutedPlayers.has(ping.playerId);
}
```

## Network Synchronization

### Ping Sync Protocol

```typescript
interface PingNetworkPacket {
  type: 'ping_create' | 'ping_update' | 'ping_remove';
  pingId: string;
  pingData?: {
    typeId: string;
    playerId: string;
    position: Vector3;
    targetEntityId?: string;
    timestamp: number;
  };
}

// Client -> Server
function sendPing(ping: Ping): void {
  const packet: PingNetworkPacket = {
    type: 'ping_create',
    pingId: ping.id,
    pingData: {
      typeId: ping.type.id,
      playerId: ping.playerId,
      position: ping.position,
      targetEntityId: ping.targetEntity?.entityId,
      timestamp: ping.timestamp
    }
  };

  sendReliable(packet);  // Reliable delivery for pings
}

// Server -> Clients (broadcast to team)
function broadcastPingToTeam(ping: Ping, team: Team): void {
  const packet: PingNetworkPacket = {
    type: 'ping_create',
    pingId: ping.id,
    pingData: {
      typeId: ping.type.id,
      playerId: ping.playerId,
      position: ping.position,
      targetEntityId: ping.targetEntity?.entityId,
      timestamp: ping.timestamp
    }
  };

  for (const member of team.members) {
    if (member.id !== ping.playerId) {
      sendToPlayer(member.id, packet);
    }
  }
}
```

### Voice Chat Networking

```typescript
interface VoicePacket {
  playerId: string;
  sequenceNumber: number;
  timestamp: number;
  audioData: Uint8Array;      // Opus encoded
  isSilence: boolean;         // Comfort noise flag
}

// UDP-based voice transmission
function transmitVoice(audioData: Float32Array): void {
  const encoded = processVoiceAudio(audioData);

  const packet: VoicePacket = {
    playerId: localPlayer.id,
    sequenceNumber: nextVoiceSequence++,
    timestamp: Date.now(),
    audioData: encoded,
    isSilence: isComfortNoise(audioData)
  };

  sendUnreliable(packet);  // UDP for low latency
}

// Jitter buffer for incoming voice
class JitterBuffer {
  private buffer: VoicePacket[] = [];
  private playbackDelay: number = 60;  // ms

  addPacket(packet: VoicePacket): void {
    // Insert in sequence order
    const insertIndex = this.buffer.findIndex(p => p.sequenceNumber > packet.sequenceNumber);
    if (insertIndex === -1) {
      this.buffer.push(packet);
    } else {
      this.buffer.splice(insertIndex, 0, packet);
    }
  }

  getNextPacket(): VoicePacket | null {
    const targetTime = Date.now() - this.playbackDelay;
    const packet = this.buffer.find(p => p.timestamp <= targetTime);

    if (packet) {
      this.buffer = this.buffer.filter(p => p !== packet);
      return packet;
    }

    return null;
  }
}
```

## Settings UI

### Communication Settings

```
┌─────────────────────────────────────────────────────────┐
│  COMMUNICATION SETTINGS                                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  PING SETTINGS                                           │
│  ┌────────────────────────────────────────────────────┐ │
│  │ Ping Volume         [████████░░]  80%              │ │
│  │ Show Pings          [✓] On                         │ │
│  │ Ping Wheel Hold     [███░░░░░░░]  150ms            │ │
│  │ Contextual Pings    [✓] On                         │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  VOICE CHAT                                              │
│  ┌────────────────────────────────────────────────────┐ │
│  │ Voice Chat          [✓] Enabled                    │ │
│  │ Mode               [Push-to-Talk ▼]                │ │
│  │ Input Sensitivity   [██████░░░░]  60%              │ │
│  │ Output Volume       [████████░░]  80%              │ │
│  │ Noise Reduction     [✓] On                         │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  QUICK MESSAGES                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │ [Configure Favorites]                              │ │
│  │ Show Voice Lines    [✓] On                         │ │
│  │ Message Volume      [██████░░░░]  60%              │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  BLOCKED PLAYERS                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │ [Manage Blocked Players] (3 blocked)               │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Analytics Events

```typescript
// Ping usage
analytics.track('ping_created', {
  pingType: string,
  targetType: 'ground' | 'enemy' | 'loot' | 'chest',
  method: 'quick' | 'wheel',
  matchTime: number
});

// Message usage
analytics.track('preset_message_sent', {
  messageId: string,
  category: string,
  matchTime: number
});

// Voice chat usage
analytics.track('voice_chat_session', {
  duration: number,
  mode: 'push_to_talk' | 'voice_activation',
  matchId: string
});

// Moderation
analytics.track('player_muted', {
  reason: 'manual' | 'spam_auto' | 'report',
  muteType: 'all' | 'voice' | 'ping'
});

analytics.track('player_reported', {
  reportType: string,
  hasReason: boolean
});
```

## Testing Requirements

### Functional Tests

- [ ] Quick ping places contextual ping correctly
- [ ] Ping wheel opens and selects correctly
- [ ] All ping types display in world and minimap
- [ ] Voice lines play for pings
- [ ] Preset messages send and display
- [ ] Voice chat push-to-talk works
- [ ] Voice activation mode works
- [ ] Mute/unmute players works
- [ ] Block players works
- [ ] Spam prevention triggers correctly

### Network Tests

- [ ] Pings sync to teammates reliably
- [ ] Voice chat latency < 150ms
- [ ] Pings work with packet loss
- [ ] Voice recovers from network hiccups

### Edge Cases

- [ ] Ping on invalid location handled
- [ ] Rapid ping attempts rate limited
- [ ] Voice with no microphone handled
- [ ] Blocked player rejoins handled

## Success Metrics

| Metric | Target |
|--------|--------|
| Ping usage rate | 5+ pings per duo match |
| Voice chat adoption | 30%+ of duo players |
| Communication-related reports | < 2% of matches |
| Voice latency | < 150ms p95 |

## Dependencies

- **GAME-021**: Duo Mode (team context)
- **BACK-003**: Real-time Networking
- **UX-009**: Minimap System (ping display)
