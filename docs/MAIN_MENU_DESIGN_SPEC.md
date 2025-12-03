# Main Menu Design Specification

## Overview

This document specifies the Main Menu design for Plunderstorm Mobile, serving as the central hub where players access all game features. The design emphasizes quick access to gameplay while showcasing progression and events.

## Screen Structure

### Main Menu Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│  [Profile]  💰 1,250  💎 150                    [Settings] [Notifications]│
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │                    NEWS BANNER / EVENT                             │ │
│  │   "Season 2: Curse of the Kraken - Now Live!"    [View Details]   │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│                                                                         │
│                  [3D Character Model on Ship Deck]                     │
│                       with Equipped Cosmetics                          │
│                                                                         │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │    DAILY QUESTS                           [View All]               │ │
│  │    ├─ Kill 5 enemies    ██████░░ 3/5                              │ │
│  │    └─ Deal 2000 damage  ████████ ✓ Complete!                      │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │   BATTLE PASS Season 2        Tier 24/100     [View Pass]         │ │
│  │   [████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░]           │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│     ┌─────────────────────────────────────────────────────────┐        │
│     │                    ⚔️ PLAY ⚔️                            │        │
│     │                  (Tap to Battle!)                        │        │
│     └─────────────────────────────────────────────────────────┘        │
│                                                                         │
│  [Armory]      [Battle Pass]      [Shop]      [Social]      [More]     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Navigation Structure

### Primary Navigation Tabs

```typescript
interface NavigationTab {
  id: string;
  label: string;
  icon: string;
  badge?: BadgeConfig;
  screen: ScreenType;
}

const NAVIGATION_TABS: NavigationTab[] = [
  {
    id: 'armory',
    label: 'Armory',
    icon: 'icon_armory',
    screen: ScreenType.ARMORY
  },
  {
    id: 'battle_pass',
    label: 'Pass',
    icon: 'icon_battle_pass',
    badge: { type: 'unclaimed_rewards' },
    screen: ScreenType.BATTLE_PASS
  },
  {
    id: 'shop',
    label: 'Shop',
    icon: 'icon_shop',
    badge: { type: 'new_items' },
    screen: ScreenType.SHOP
  },
  {
    id: 'social',
    label: 'Social',
    icon: 'icon_social',
    badge: { type: 'pending_requests' },
    screen: ScreenType.SOCIAL
  },
  {
    id: 'more',
    label: 'More',
    icon: 'icon_more',
    screen: ScreenType.MORE_MENU
  }
];
```

### Screen Hierarchy

```
Main Menu
├── Play Button → Mode Selection → Matchmaking
├── Armory
│   ├── Character
│   ├── Abilities
│   ├── Cosmetics
│   └── Loadouts
├── Battle Pass
│   ├── Current Season
│   ├── Premium Upgrade
│   └── Reward Track
├── Shop
│   ├── Featured
│   ├── Doubloons (IAP)
│   ├── Item Shop
│   └── Daily Deals
├── Social
│   ├── Friends List
│   ├── Party
│   ├── Leaderboards
│   └── Club/Clan
└── More
    ├── Settings
    ├── Profile
    ├── Quests
    ├── News
    ├── Help/FAQ
    └── Legal
```

## Header Section

### Top Bar Design

```
┌─────────────────────────────────────────────────────────────────┐
│ [👤]  Level 24                    💰 1,250   💎 150    [⚙️] [🔔] │
│  │                                    │         │         │    │
│  │                                    │         │         │    │
│Profile Avatar                     Gold    Doubloons   Settings │
│with level ring                                           Notifications
└─────────────────────────────────────────────────────────────────┘
```

### Header Components

```typescript
interface HeaderConfig {
  profile: {
    avatar: string;
    level: number;
    xpProgress: number;
    xpToNextLevel: number;
    tapAction: 'open_profile';
  };

  currencies: CurrencyDisplay[];

  actions: HeaderAction[];
}

interface CurrencyDisplay {
  type: 'gold' | 'doubloons';
  icon: string;
  amount: number;
  tapAction: 'open_shop';
  showPlusButton: boolean;   // Quick purchase
}

interface HeaderAction {
  id: string;
  icon: string;
  badge?: number;           // Notification count
  tapAction: string;
}

const HEADER_ACTIONS: HeaderAction[] = [
  { id: 'settings', icon: 'icon_settings', tapAction: 'open_settings' },
  { id: 'notifications', icon: 'icon_notifications', badge: 3, tapAction: 'open_notifications' }
];
```

## News Banner

### Banner System

```typescript
interface NewsBanner {
  id: string;
  type: BannerType;
  priority: number;
  content: BannerContent;
  schedule: BannerSchedule;
  targeting?: BannerTargeting;
}

enum BannerType {
  EVENT = 'event',
  UPDATE = 'update',
  PROMO = 'promo',
  MAINTENANCE = 'maintenance',
  ANNOUNCEMENT = 'announcement'
}

interface BannerContent {
  headline: string;
  subheadline?: string;
  backgroundImage: string;
  ctaButton?: {
    text: string;
    action: string;
  };
  dismissable: boolean;
}

interface BannerSchedule {
  startTime: Date;
  endTime: Date;
  showOnce: boolean;        // Only show once per user
  frequency?: number;       // Show every N sessions
}
```

### Banner Carousel

```typescript
interface BannerCarouselConfig {
  autoRotate: true;
  rotateInterval: 5000;     // ms
  maxBanners: 5;
  indicators: true;         // Dots at bottom
  swipeable: true;
}

// Banner content examples
const SAMPLE_BANNERS: NewsBanner[] = [
  {
    id: 'season_2',
    type: BannerType.EVENT,
    priority: 1,
    content: {
      headline: "Season 2: Curse of the Kraken",
      subheadline: "New abilities, cosmetics, and Battle Pass!",
      backgroundImage: 'banner_season_2.jpg',
      ctaButton: { text: 'View Details', action: 'open_battle_pass' },
      dismissable: false
    },
    schedule: { startTime: new Date('2024-01-01'), endTime: new Date('2024-03-31'), showOnce: false }
  },
  {
    id: 'weekend_event',
    type: BannerType.PROMO,
    priority: 2,
    content: {
      headline: "Double XP Weekend!",
      subheadline: "Earn 2x XP in all matches",
      backgroundImage: 'banner_double_xp.jpg',
      dismissable: true
    },
    schedule: { startTime: new Date(), endTime: new Date(Date.now() + 172800000), showOnce: false }
  }
];
```

## Character Display

### 3D Character Showcase

```typescript
interface CharacterShowcaseConfig {
  // Model settings
  model: {
    character: string;
    pose: 'idle' | 'showcase';
    equipment: EquippedCosmetics;
  };

  // Environment
  background: {
    scene: 'ship_deck';
    lighting: 'dynamic';      // Changes with time of day
    particles: 'ocean_mist';
  };

  // Camera
  camera: {
    orbit: true;              // Auto-rotate around character
    orbitSpeed: 0.1;
    userDrag: true;           // Manual rotation
    zoomEnabled: false;
  };

  // Interaction
  interaction: {
    tapCharacter: 'play_emote';
    swipeCharacter: 'rotate';
    doubleTap: 'zoom_cosmetic';
  };
}
```

### Character Rendering

```typescript
interface CharacterRenderSettings {
  resolution: 'adaptive';     // Based on device capability
  quality: 'high' | 'medium' | 'low';
  shadows: boolean;
  reflections: boolean;
  particleEffects: boolean;

  // Performance optimization
  lodDistance: number;
  animationFramerate: 30;
  batchStaticElements: true;
}

function updateCharacterDisplay(config: CharacterShowcaseConfig): void {
  // Load character model with equipped cosmetics
  const model = loadCharacterModel(config.model.character);

  // Apply equipped items
  applyCosmetics(model, config.model.equipment);

  // Set up scene
  const scene = setupScene(config.background);

  // Configure camera
  const camera = setupCamera(config.camera);

  // Start render loop
  startRenderLoop(model, scene, camera);
}
```

## Daily Quests Preview

### Quest Widget

```
┌─────────────────────────────────────────────────────────────────┐
│    DAILY QUESTS                              [View All]         │
├─────────────────────────────────────────────────────────────────┤
│    │                                                            │
│ ⚔️ │ Kill 5 enemies              ██████░░░░ 3/5     💰 +50     │
│    │                                                            │
├────┼────────────────────────────────────────────────────────────┤
│    │                                                            │
│ 🎯 │ Deal 2000 damage            ████████████ ✓     [Claim]     │
│    │                                                            │
├────┼────────────────────────────────────────────────────────────┤
│    │                                                            │
│ 🏆 │ Finish Top 10               ░░░░░░░░░░ 0/2     💰 +75     │
│    │                                                            │
└────┴────────────────────────────────────────────────────────────┘
```

### Quest Widget Configuration

```typescript
interface QuestWidgetConfig {
  maxDisplayed: 3;
  showCompleted: true;
  showClaimable: true;
  sortOrder: 'progress' | 'reward' | 'time';
  compactMode: boolean;
}

interface DailyQuestPreview {
  id: string;
  icon: string;
  title: string;
  progress: number;
  target: number;
  reward: QuestReward;
  status: 'in_progress' | 'completed' | 'claimed';
  expiresIn: number;         // seconds
}

function getQuestPreviews(): DailyQuestPreview[] {
  const quests = getDailyQuests();

  // Sort by most actionable (claimable first, then closest to completion)
  return quests
    .sort((a, b) => {
      if (a.status === 'completed' && b.status !== 'completed') return -1;
      if (b.status === 'completed' && a.status !== 'completed') return 1;
      return (b.progress / b.target) - (a.progress / a.target);
    })
    .slice(0, QUEST_WIDGET_CONFIG.maxDisplayed);
}
```

## Battle Pass Preview

### Pass Progress Widget

```
┌─────────────────────────────────────────────────────────────────┐
│   BATTLE PASS Season 2              Tier 24/100    [View Pass] │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   [Tier 23]    [Tier 24]    [Tier 25]    [Tier 26]            │
│      ✓           🔓           🔒           🔒                  │
│    [Item]      [Item]      [Item]      [Item]                 │
│                                                                 │
│   Progress to next tier:                                        │
│   [████████████████████████░░░░░░░░░░]  340/500 XP             │
│                                                                 │
│   ⏱️ 47 days remaining                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Battle Pass Widget

```typescript
interface BattlePassWidget {
  seasonName: string;
  currentTier: number;
  maxTier: number;
  xpProgress: number;
  xpToNextTier: number;
  daysRemaining: number;
  isPremium: boolean;
  unclaimedRewards: number;
  previewTiers: TierPreview[];
}

interface TierPreview {
  tier: number;
  status: 'claimed' | 'unlocked' | 'locked';
  reward: {
    icon: string;
    name: string;
    rarity: string;
    isPremium: boolean;
  };
}

function getBattlePassWidget(): BattlePassWidget {
  const pass = getCurrentBattlePass();
  const progress = getPlayerBattlePassProgress();

  // Get 4 tiers to preview (current and next 3)
  const previewTiers = [];
  for (let i = progress.currentTier - 1; i <= progress.currentTier + 2; i++) {
    if (i >= 1 && i <= pass.maxTier) {
      previewTiers.push({
        tier: i,
        status: getTierStatus(i, progress),
        reward: pass.tiers[i].freeReward
      });
    }
  }

  return {
    seasonName: pass.name,
    currentTier: progress.currentTier,
    maxTier: pass.maxTier,
    xpProgress: progress.xpProgress,
    xpToNextTier: pass.xpPerTier,
    daysRemaining: Math.floor((pass.endDate - Date.now()) / 86400000),
    isPremium: progress.hasPremium,
    unclaimedRewards: countUnclaimedRewards(progress),
    previewTiers
  };
}
```

## Play Button

### Main CTA Design

```typescript
interface PlayButtonConfig {
  // Visual
  style: 'prominent';
  size: 'large';
  icon: 'crossed_swords';
  text: 'PLAY';
  subtext: 'Tap to Battle!';

  // Animation
  pulse: true;
  pulseInterval: 2000;
  glowColor: '#FFD700';

  // State
  disabled: false;
  showMatchmakingStatus: true;

  // Action
  tapAction: 'open_mode_select';
  longPressAction: 'quick_play_last_mode';
}

const PLAY_BUTTON_STATES = {
  default: {
    text: 'PLAY',
    subtext: 'Tap to Battle!',
    enabled: true
  },
  matchmaking: {
    text: 'SEARCHING...',
    subtext: 'Estimated: 15s',
    enabled: true,
    showSpinner: true
  },
  maintenance: {
    text: 'PLAY',
    subtext: 'Servers Under Maintenance',
    enabled: false
  },
  offline: {
    text: 'PLAY',
    subtext: 'No Connection',
    enabled: false
  }
};
```

### Quick Play Feature

```typescript
interface QuickPlayConfig {
  enabled: true;
  trigger: 'long_press';      // Long press play button
  duration: 500;              // ms to trigger
  mode: 'last_played';        // Solo, Duo, etc.
  feedback: 'haptic_medium';
}

function handlePlayButtonPress(event: TouchEvent): void {
  if (event.type === 'long_press' && QUICK_PLAY_CONFIG.enabled) {
    // Quick play last mode
    const lastMode = getLastPlayedMode();
    triggerHaptic('medium');
    startMatchmaking(lastMode);
    showQuickPlayFeedback(lastMode);
  } else {
    // Normal tap - open mode selection
    openModeSelection();
  }
}
```

## Bottom Navigation

### Tab Bar Design

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   [⚔️]       [🎫]       [🛒]       [👥]       [≡]              │
│  Armory     Pass       Shop     Social     More                 │
│             (•2)       (NEW)     (•1)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Legend:
- (•N) = Badge with count
- (NEW) = New items indicator
- [⚔️] = Active tab (highlighted)
```

### Tab Bar Configuration

```typescript
interface TabBarConfig {
  style: 'fixed_bottom';
  height: 60;                 // pixels
  backgroundColor: 'rgba(20, 20, 30, 0.95)';
  activeColor: '#FFD700';
  inactiveColor: '#888888';

  // Safe area
  respectSafeArea: true;
  homeIndicatorPadding: true;

  // Animation
  switchAnimation: 'slide';
  hapticFeedback: true;
}

interface TabBadge {
  type: 'count' | 'dot' | 'text';
  value?: number | string;
  color: string;
  animate: boolean;
}

function getTabBadges(): Map<string, TabBadge> {
  return new Map([
    ['battle_pass', {
      type: 'count',
      value: getUnclaimedBattlePassRewards(),
      color: '#FF4444',
      animate: true
    }],
    ['shop', {
      type: 'text',
      value: 'NEW',
      color: '#00FF00',
      animate: false
    }],
    ['social', {
      type: 'count',
      value: getPendingFriendRequests(),
      color: '#FF4444',
      animate: true
    }]
  ]);
}
```

## Notifications Panel

### Notification Types

```typescript
enum NotificationType {
  FRIEND_REQUEST = 'friend_request',
  PARTY_INVITE = 'party_invite',
  GIFT = 'gift',
  QUEST_COMPLETE = 'quest_complete',
  REWARD_AVAILABLE = 'reward_available',
  EVENT_START = 'event_start',
  MAINTENANCE = 'maintenance',
  UPDATE_AVAILABLE = 'update_available'
}

interface NotificationItem {
  id: string;
  type: NotificationType;
  title: string;
  message: string;
  icon: string;
  timestamp: Date;
  read: boolean;
  actionable: boolean;
  action?: NotificationAction;
  expires?: Date;
}

interface NotificationAction {
  label: string;
  action: string;
  params?: Record<string, any>;
}
```

### Notification Panel UI

```
┌─────────────────────────────────────────────────────────────────┐
│   NOTIFICATIONS                              [Mark All Read]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   🎁  TreasureHunter sent you a gift!              2m ago      │
│       "Check out this awesome emote!"        [Open Gift]       │
│   ─────────────────────────────────────────────────────────────│
│   👥  CaptainJack wants to be friends              15m ago     │
│                                    [Accept] [Decline]          │
│   ─────────────────────────────────────────────────────────────│
│   🎯  Quest Complete: Kill 10 enemies             1h ago       │
│       Reward ready to claim!                 [Claim]           │
│   ─────────────────────────────────────────────────────────────│
│   📢  Weekend Event starting soon!                3h ago       │
│       Double XP begins Friday at 6 PM        [View Details]    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Loading States

### Initial Load

```typescript
interface MenuLoadingState {
  phase: 'connecting' | 'loading_profile' | 'loading_assets' | 'ready';
  progress: number;           // 0-100
  message: string;
}

const LOADING_PHASES: MenuLoadingState[] = [
  { phase: 'connecting', progress: 0, message: 'Connecting to server...' },
  { phase: 'loading_profile', progress: 25, message: 'Loading profile...' },
  { phase: 'loading_assets', progress: 50, message: 'Loading assets...' },
  { phase: 'ready', progress: 100, message: 'Ready!' }
];

async function loadMainMenu(): Promise<void> {
  // Show loading screen with pirate theme
  showLoadingScreen();

  // Phase 1: Connect to backend
  updateLoadingState(LOADING_PHASES[0]);
  await connectToBackend();

  // Phase 2: Load player profile
  updateLoadingState(LOADING_PHASES[1]);
  const profile = await loadPlayerProfile();

  // Phase 3: Load assets (character model, UI assets)
  updateLoadingState(LOADING_PHASES[2]);
  await loadMenuAssets(profile);

  // Phase 4: Ready
  updateLoadingState(LOADING_PHASES[3]);
  await delay(500);

  // Transition to main menu
  transitionToMainMenu();
}
```

### Skeleton Loading

```typescript
interface SkeletonConfig {
  questWidget: {
    showSkeleton: true;
    itemCount: 3;
  };
  battlePassWidget: {
    showSkeleton: true;
  };
  currencyDisplay: {
    showSkeleton: true;
  };
}

// Show skeleton UI while data loads
function showSkeletonUI(): void {
  // Quest widget skeleton
  renderQuestSkeleton(SKELETON_CONFIG.questWidget.itemCount);

  // Battle Pass widget skeleton
  renderBattlePassSkeleton();

  // Currency skeleton
  renderCurrencySkeleton();
}
```

## Animation System

### Menu Transitions

```typescript
interface TransitionConfig {
  enterAnimation: 'fade_up' | 'slide_right' | 'scale_up';
  exitAnimation: 'fade_down' | 'slide_left' | 'scale_down';
  duration: 300;              // ms
  easing: 'ease_out_cubic';
}

const SCREEN_TRANSITIONS: Record<string, TransitionConfig> = {
  'main_menu_enter': {
    enterAnimation: 'fade_up',
    exitAnimation: 'fade_down',
    duration: 300,
    easing: 'ease_out_cubic'
  },
  'tab_switch': {
    enterAnimation: 'slide_right',
    exitAnimation: 'slide_left',
    duration: 250,
    easing: 'ease_out_quad'
  },
  'modal_open': {
    enterAnimation: 'scale_up',
    exitAnimation: 'scale_down',
    duration: 200,
    easing: 'ease_out_back'
  }
};
```

### Element Animations

```typescript
interface ElementAnimation {
  playButton: {
    pulse: true;
    pulseScale: 1.05;
    pulseDuration: 2000;
  };

  badges: {
    bounce: true;
    bounceDuration: 500;
    bounceOnChange: true;
  };

  newsBanner: {
    slideInterval: 5000;
    slideDirection: 'left';
    slideDuration: 500;
  };

  characterModel: {
    idleAnimation: true;
    breathingMotion: true;
    blinkInterval: 4000;
  };
}
```

## Performance Optimization

### Menu Performance

```typescript
interface MenuPerformanceConfig {
  targetFPS: 60;
  maxConcurrentAnimations: 5;
  asyncAssetLoading: true;
  lazyLoadTabs: true;         // Only load tab content when accessed
  cacheTabState: true;        // Keep tab state when switching
  reduceMotion: false;        // Accessibility option
}

// Asset prioritization
const ASSET_PRIORITY = {
  critical: ['character_model', 'ui_sprites', 'fonts'],
  high: ['news_banners', 'profile_data'],
  medium: ['battle_pass_preview', 'shop_featured'],
  low: ['social_avatars', 'leaderboard_data']
};

async function loadMenuAssetsOptimized(): Promise<void> {
  // Load critical assets first (blocking)
  await loadAssets(ASSET_PRIORITY.critical);

  // Load high priority (async)
  loadAssets(ASSET_PRIORITY.high);

  // Load rest on demand
  // Medium and low priority load when user navigates to those sections
}
```

### Memory Management

```typescript
interface MemoryConfig {
  maxCachedScreens: 3;
  unloadDelay: 5000;          // ms before unloading inactive screen
  characterModelLOD: 'auto';  // Level of detail based on device
  textureQuality: 'auto';
}

function manageScreenMemory(): void {
  const activeScreens = getActiveScreenStack();

  // Keep only recent screens in memory
  if (activeScreens.length > MEMORY_CONFIG.maxCachedScreens) {
    const screensToUnload = activeScreens.slice(MEMORY_CONFIG.maxCachedScreens);
    screensToUnload.forEach(screen => unloadScreen(screen));
  }
}
```

## Accessibility

### Menu Accessibility Features

```typescript
interface MenuAccessibility {
  // Navigation
  tabOrder: string[];         // Focus order
  screenReaderLabels: Record<string, string>;

  // Visual
  highContrastMode: boolean;
  largeText: boolean;
  reducedMotion: boolean;

  // Interaction
  tapTargetMinSize: 44;       // pixels
  doubleTapToConfirm: boolean;
}

const SCREEN_READER_LABELS: Record<string, string> = {
  'play_button': 'Play button. Double tap to start matchmaking.',
  'profile_button': 'Profile. Level 24. Double tap to view profile.',
  'gold_display': 'Gold balance: 1,250 coins',
  'doubloons_display': 'Doubloons balance: 150 premium currency',
  'quest_widget': 'Daily quests. 1 of 3 completed.',
  'battle_pass_widget': 'Battle Pass. Tier 24 of 100. 47 days remaining.'
};
```

## Deep Linking

### Menu Deep Links

```typescript
interface DeepLinkConfig {
  scheme: 'plunderstorm://';
  routes: DeepLinkRoute[];
}

interface DeepLinkRoute {
  path: string;
  screen: string;
  params?: string[];
}

const DEEP_LINK_ROUTES: DeepLinkRoute[] = [
  { path: 'menu', screen: 'main_menu' },
  { path: 'menu/play', screen: 'mode_selection' },
  { path: 'menu/armory', screen: 'armory' },
  { path: 'menu/shop', screen: 'shop' },
  { path: 'menu/shop/featured/:itemId', screen: 'shop_item', params: ['itemId'] },
  { path: 'menu/battlepass', screen: 'battle_pass' },
  { path: 'menu/social', screen: 'social' },
  { path: 'menu/profile', screen: 'profile' },
  { path: 'menu/settings', screen: 'settings' }
];

function handleDeepLink(url: string): void {
  const route = matchRoute(url, DEEP_LINK_ROUTES);

  if (route) {
    navigateToScreen(route.screen, route.extractedParams);
  } else {
    // Default to main menu
    navigateToScreen('main_menu');
  }
}
```

## Analytics Events

```typescript
// Menu navigation
analytics.track('menu_screen_viewed', {
  screen: string,
  previousScreen: string,
  sessionTime: number
});

// Feature engagement
analytics.track('menu_widget_interaction', {
  widget: 'quest_preview' | 'battle_pass_preview' | 'news_banner',
  action: 'tap' | 'swipe' | 'expand',
  cta: string
});

// Play button
analytics.track('play_button_pressed', {
  method: 'tap' | 'long_press',
  fromScreen: string,
  queuedMode: string
});

// Navigation
analytics.track('tab_switched', {
  fromTab: string,
  toTab: string,
  hadBadge: boolean
});
```

## Testing Requirements

### Functional Tests

- [ ] Menu loads within 3 seconds
- [ ] All navigation tabs accessible
- [ ] Play button initiates mode selection
- [ ] Quick play (long press) works
- [ ] News banner rotates and links work
- [ ] Quest widget shows correct progress
- [ ] Battle Pass widget shows correct tier
- [ ] Currency displays update correctly
- [ ] Notifications panel opens and updates
- [ ] Deep links navigate correctly

### Visual Tests

- [ ] 60 FPS on mid-range devices
- [ ] Character model renders correctly
- [ ] Animations play smoothly
- [ ] Tab switching is seamless
- [ ] Badges display correctly
- [ ] Safe areas respected

### Edge Cases

- [ ] Offline mode handling
- [ ] Server maintenance mode
- [ ] Very long usernames
- [ ] Maximum notifications
- [ ] Zero quests available

## Success Metrics

| Metric | Target |
|--------|--------|
| Menu load time | < 3 seconds |
| Play button tap rate | 80%+ of sessions |
| Tab exploration rate | 3+ tabs per session |
| Quest widget engagement | 40%+ tap through |
| Banner CTR | 5%+ |

## Dependencies

- **MON-001**: Virtual Currency System
- **LIVE-001**: Daily Quest System
- **LIVE-003**: Battle Pass System
- **BACK-005**: Account/Profile System
- **BACK-006**: Friends/Social System
