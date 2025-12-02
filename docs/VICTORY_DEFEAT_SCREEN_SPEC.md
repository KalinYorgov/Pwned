# Victory/Defeat Screen Specification

## Overview

This document specifies the Victory and Defeat screens for Plunderstorm Mobile, displaying match results, stats, rewards, and providing options for quick rematch or returning to lobby. The design emphasizes celebrating success while maintaining engagement after defeat.

## Screen Types

### Screen Variants

| Outcome | Placement | Screen Type | Tone |
|---------|-----------|-------------|------|
| Victory | 1st | Grand Victory | Celebratory |
| Top 3 | 2nd-3rd | Podium Finish | Congratulatory |
| Top 10 | 4th-10th | Strong Finish | Encouraging |
| Defeat | 11th+ | Defeat | Motivational |

## Victory Screen

### Grand Victory (1st Place)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    ⚓ VICTORY! ⚓                           │
│              🏴‍☠️ CHAMPION OF THE SEAS 🏴‍☠️                    │
│                                                             │
│         [Character Model with Victory Pose Animation]       │
│                   [Confetti & Gold Coins VFX]              │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  MATCH STATS                                                │
│  ┌────────────┬────────────┬────────────┬────────────┐     │
│  │ ⚔️ Kills   │ 💀 Assists │ 🎯 Damage  │ ⏱️ Time    │     │
│  │    7       │     3      │   2,450    │  12:34     │     │
│  └────────────┴────────────┴────────────┴────────────┘     │
│                                                             │
│  REWARDS EARNED                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💰 +150 Gold    ⭐ +500 XP    🎫 +100 Battle Pass   │  │
│  │  [████████████████] Level 24 → 25 (+1)              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    [🔄 Play Again]              [🏠 Return to Lobby]       │
│                                                             │
│              [📤 Share Results]                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Victory Visual Effects

```typescript
interface VictoryEffects {
  // Background
  skyboxChange: 'golden_sunset';
  lightingIntensity: 1.3;

  // Character
  characterPose: 'victory_fist_pump';
  characterSpotlight: true;
  characterGlow: Color.gold;

  // Particles
  confetti: {
    enabled: true;
    colors: ['#FFD700', '#FFA500', '#FFFFFF'];
    burstCount: 500;
    duration: 10000;
  };

  goldCoins: {
    enabled: true;
    spawnRate: 20;  // per second
    fallSpeed: 2.0;
  };

  // UI
  titleAnimation: 'slam_in_with_shake';
  statsReveal: 'sequential_slide_in';
  rewardsAnimation: 'count_up_with_sparkle';
}
```

### Victory Audio

```typescript
interface VictoryAudio {
  fanfare: 'sfx_victory_fanfare';           // Triumphant horn/orchestra
  ambientCheers: 'sfx_crowd_cheering';      // Loop during screen
  coinSounds: 'sfx_coins_shower';           // Gold coins falling
  statReveal: 'sfx_stat_ding';              // Each stat appears
  xpCount: 'sfx_xp_counting';               // XP counting up
  levelUp: 'sfx_level_up_fanfare';          // If level gained
  buttonHover: 'sfx_button_hover';
  buttonPress: 'sfx_button_confirm';
}
```

## Top Finish Screen (2nd-10th)

### Podium Finish (2nd-3rd)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                  🥈 SECOND PLACE! 🥈                       │
│               "A worthy challenger!"                        │
│                                                             │
│         [Character Model with Salute Animation]            │
│                   [Silver Sparkles VFX]                    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  MATCH STATS                                                │
│  ┌────────────┬────────────┬────────────┬────────────┐     │
│  │ ⚔️ Kills   │ 💀 Assists │ 🎯 Damage  │ ⏱️ Time    │     │
│  │    5       │     2      │   1,890    │  11:45     │     │
│  └────────────┴────────────┴────────────┴────────────┘     │
│                                                             │
│  Eliminated by: CaptainJack (Cannonball Barrage)           │
│                                                             │
│  REWARDS EARNED                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💰 +100 Gold    ⭐ +350 XP    🎫 +70 Battle Pass    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│    [🔄 Play Again]              [🏠 Return to Lobby]       │
│              [📤 Share Results]                             │
└─────────────────────────────────────────────────────────────┘
```

### Strong Finish (4th-10th)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                  🏅 TOP 10 FINISH! 🏅                      │
│              Placement: #7 / 60 players                    │
│                                                             │
│         [Character Model with Ready Stance]                │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  MATCH STATS                                                │
│  ┌────────────┬────────────┬────────────┬────────────┐     │
│  │ ⚔️ Kills   │ 💀 Assists │ 🎯 Damage  │ ⏱️ Time    │     │
│  │    3       │     1      │   980      │  8:22      │     │
│  └────────────┴────────────┴────────────┴────────────┘     │
│                                                             │
│  Eliminated by: TreasureHunter (Cutlass Slash)             │
│                                                             │
│  REWARDS EARNED                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💰 +50 Gold     ⭐ +200 XP    🎫 +40 Battle Pass    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│    [🔄 Play Again]              [🏠 Return to Lobby]       │
└─────────────────────────────────────────────────────────────┘
```

## Defeat Screen

### Standard Defeat (11th+)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                   DEFEATED                                  │
│              Placement: #23 / 60 players                   │
│                                                             │
│         [Character Model with Determined Pose]             │
│             "The sea hasn't seen the last of me!"          │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  MATCH STATS                                                │
│  ┌────────────┬────────────┬────────────┬────────────┐     │
│  │ ⚔️ Kills   │ 💀 Assists │ 🎯 Damage  │ ⏱️ Time    │     │
│  │    1       │     0      │   450      │  4:15      │     │
│  └────────────┴────────────┴────────────┴────────────┘     │
│                                                             │
│  Eliminated by: SeaDog99 (Grappling Hook)                  │
│                                                             │
│  YOUR BEST THIS SESSION                                     │
│  Best Placement: #7  |  Most Kills: 5  |  Highest Damage: 1,890│
│                                                             │
│  REWARDS EARNED                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💰 +15 Gold     ⭐ +75 XP     🎫 +15 Battle Pass    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│    [🔄 Play Again]              [🏠 Return to Lobby]       │
│                                                             │
│            [👁️ Spectate Match] (if still ongoing)          │
└─────────────────────────────────────────────────────────────┘
```

### Defeat Visual Design

```typescript
interface DefeatEffects {
  // Keep it dignified, not depressing
  skyboxChange: 'dusk';
  lightingIntensity: 0.9;

  // Character
  characterPose: 'determined_stance';  // Not sad/dejected
  characterSpotlight: true;
  characterGlow: Color.blue;           // Cool, calming

  // Particles
  ambientDust: {
    enabled: true;
    density: 'light';
  };

  // Motivational element
  showTipOrEncouragement: true;
  showSessionBest: true;               // Show what they did well
}
```

### Defeat Audio

```typescript
interface DefeatAudio {
  // Somber but not depressing
  theme: 'sfx_defeat_theme';            // Reflective, not sad
  ambientWaves: 'sfx_ocean_ambient';    // Calming
  buttonHover: 'sfx_button_hover';
  buttonPress: 'sfx_button_confirm';
}
```

## Stats Display

### Match Statistics

```typescript
interface MatchStats {
  placement: number;
  totalPlayers: number;

  // Combat
  kills: number;
  assists: number;
  damageDealt: number;
  damageTaken: number;
  healingDone: number;

  // Survival
  survivalTime: number;           // seconds
  distanceTraveled: number;       // meters
  stormsOutrun: number;

  // Abilities
  abilitiesUsed: number;
  abilityDamage: number;
  crowdControlTime: number;       // seconds enemies CC'd

  // Economy
  chestsOpened: number;
  abilitiesCollected: number;
  abilitiesUpgraded: number;

  // Eliminator info (if defeated)
  eliminatedBy?: {
    playerName: string;
    playerId: string;
    abilityUsed: string;
    abilityIcon: string;
  };
}
```

### Stats Display Configuration

```typescript
interface StatsDisplayConfig {
  // Primary stats (always shown)
  primaryStats: ['kills', 'assists', 'damageDealt', 'survivalTime'];

  // Secondary stats (expandable)
  secondaryStats: ['damageTaken', 'healingDone', 'distanceTraveled', 'chestsOpened'];

  // Animation
  revealDelay: 200;               // ms between each stat
  countUpDuration: 800;           // ms for number animation
  highlightPersonalBest: true;

  // Comparison
  showSessionBest: true;
  showLifetimeBest: false;        // Optional
}

function animateStatReveal(stats: MatchStats): void {
  const primaryStats = STATS_CONFIG.primaryStats;

  primaryStats.forEach((stat, index) => {
    setTimeout(() => {
      const element = getStatElement(stat);
      element.classList.add('reveal');

      // Count up animation
      animateCountUp(element, 0, stats[stat], STATS_CONFIG.countUpDuration);

      // Play sound
      playSound('sfx_stat_ding');

      // Check for personal best
      if (isPersonalBest(stat, stats[stat])) {
        highlightPersonalBest(element);
        playSound('sfx_personal_best');
      }
    }, index * STATS_CONFIG.revealDelay);
  });
}
```

## Rewards System

### Reward Calculation

```typescript
interface MatchRewards {
  gold: number;
  xp: number;
  battlePassXP: number;
  rankPoints?: number;          // For ranked mode

  // Bonuses applied
  bonuses: RewardBonus[];

  // Quest progress
  questProgress: QuestProgress[];
}

interface RewardBonus {
  type: string;
  multiplier: number;
  description: string;
}

function calculateRewards(stats: MatchStats, mode: GameMode): MatchRewards {
  const baseRewards = getBaseRewards(stats.placement, mode);

  let rewards: MatchRewards = {
    gold: baseRewards.gold,
    xp: baseRewards.xp,
    battlePassXP: baseRewards.battlePassXP,
    bonuses: [],
    questProgress: []
  };

  // Kill bonus
  const killBonus = stats.kills * 10;
  rewards.gold += killBonus;
  if (killBonus > 0) {
    rewards.bonuses.push({
      type: 'kills',
      multiplier: stats.kills,
      description: `+${killBonus} Gold (${stats.kills} kills)`
    });
  }

  // Survival time bonus (1 XP per 10 seconds)
  const survivalBonus = Math.floor(stats.survivalTime / 10);
  rewards.xp += survivalBonus;

  // First win of the day
  if (stats.placement === 1 && !hasFirstWinToday()) {
    rewards.gold *= 2;
    rewards.xp *= 1.5;
    rewards.bonuses.push({
      type: 'first_win',
      multiplier: 2,
      description: 'First Win of the Day (2x Gold!)'
    });
    markFirstWinClaimed();
  }

  // Battle Pass premium bonus
  if (hasPremiumBattlePass()) {
    rewards.battlePassXP *= 1.2;
    rewards.bonuses.push({
      type: 'premium_pass',
      multiplier: 1.2,
      description: 'Premium Pass Bonus (+20% BP XP)'
    });
  }

  return rewards;
}
```

### Reward Base Values

| Placement | Gold | XP | Battle Pass XP |
|-----------|------|-----|----------------|
| 1st | 150 | 500 | 100 |
| 2nd | 100 | 350 | 70 |
| 3rd | 75 | 300 | 60 |
| 4th-5th | 50 | 200 | 40 |
| 6th-10th | 40 | 150 | 30 |
| 11th-20th | 25 | 100 | 20 |
| 21st-30th | 15 | 75 | 15 |
| 31st-60th | 10 | 50 | 10 |

### Rewards Animation

```typescript
interface RewardsAnimation {
  // Gold
  goldStartValue: 0;
  goldEndValue: number;
  goldCountDuration: 1500;
  goldCoinIcon: true;
  goldPileGrow: true;

  // XP
  xpBarFill: true;
  xpCountUp: true;
  levelUpCelebration: boolean;

  // Battle Pass
  battlePassBarFill: true;
  tierUnlockCelebration: boolean;
}

function animateRewards(rewards: MatchRewards): void {
  // Animate gold coins
  const goldElement = getGoldElement();
  animateCountUp(goldElement, 0, rewards.gold, 1500);
  playSound('sfx_coins_counting');

  // Animate XP bar with potential level up
  setTimeout(() => {
    animateXPGain(rewards.xp, () => {
      // Check for level up
      if (didLevelUp) {
        showLevelUpCelebration();
        playSound('sfx_level_up');
      }
    });
  }, 1000);

  // Animate Battle Pass progress
  setTimeout(() => {
    animateBattlePassGain(rewards.battlePassXP, () => {
      // Check for tier unlock
      if (didUnlockTier) {
        showTierUnlockPreview();
        playSound('sfx_tier_unlock');
      }
    });
  }, 2000);
}
```

## Level Up Celebration

### Level Up Display

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    ⬆️ LEVEL UP! ⬆️                          │
│                                                             │
│                  Level 24 → Level 25                        │
│                                                             │
│              [Starburst Animation Around Number]           │
│                                                             │
│  NEW UNLOCKS:                                               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  🎨 New Avatar Frame: "Seasoned Sailor"              │  │
│  │  💬 New Emote: "Victory Jig"                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│                    [Continue]                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Level Up Animation

```typescript
interface LevelUpAnimation {
  numberZoomIn: true;
  starburstParticles: true;
  screenFlash: Color.gold;
  unlockItemsSlideIn: true;
  soundFanfare: 'sfx_level_up_major';
  hapticFeedback: 'heavy';
}
```

## Battle Pass Tier Unlock

### Tier Preview

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                  🎫 TIER UNLOCKED! 🎫                       │
│                      Tier 15                                │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  [Item Preview Image]                               │   │
│  │                                                     │   │
│  │  🗡️ Weapon Skin: "Kraken's Edge"                    │   │
│  │  Rare Quality                                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│            [Claim & Equip]  [View Battle Pass]             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Duo Mode Results

### Team Results Screen

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    ⚓ VICTORY! ⚓                           │
│              🏴‍☠️ CHAMPIONS OF THE SEAS 🏴‍☠️                    │
│                                                             │
│    [Character 1]               [Character 2]               │
│     CaptainJack               TreasureHunter               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  TEAM STATS                                                 │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Total Kills: 12  |  Total Damage: 4,340  |  Revives: 2│ │
│  └───────────────────────────────────────────────────────┘ │
│                                                             │
│  INDIVIDUAL STATS                                           │
│  ┌──────────────────────┬──────────────────────┐          │
│  │   CaptainJack        │   TreasureHunter     │          │
│  │   ⚔️ 7 Kills         │   ⚔️ 5 Kills         │          │
│  │   🎯 2,450 Damage    │   🎯 1,890 Damage    │          │
│  │   ❤️ 1 Revive        │   ❤️ 1 Revive        │          │
│  └──────────────────────┴──────────────────────┘          │
│                                                             │
│  TEAM MVP: CaptainJack (Most Kills)                        │
│                                                             │
│  REWARDS (Each Player)                                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💰 +150 Gold    ⭐ +500 XP    🎫 +100 Battle Pass   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│    [🔄 Play Again Together]      [🏠 Return to Lobby]      │
│              [📤 Share Results]                             │
└─────────────────────────────────────────────────────────────┘
```

### MVP Selection

```typescript
interface MVPCriteria {
  kills: { weight: 30, label: 'Most Kills' };
  damage: { weight: 25, label: 'Most Damage' };
  revives: { weight: 25, label: 'Team Support' };
  survivalTime: { weight: 10, label: 'Longest Survival' };
  healing: { weight: 10, label: 'Most Healing' };
}

function selectMVP(team: TeamStats): MVPResult {
  let mvpScores: Map<string, number> = new Map();

  for (const member of team.members) {
    let score = 0;

    // Calculate weighted score
    score += member.kills * MVP_CRITERIA.kills.weight;
    score += (member.damage / 100) * MVP_CRITERIA.damage.weight;
    score += member.revives * 100 * MVP_CRITERIA.revives.weight;
    score += (member.survivalTime / 60) * MVP_CRITERIA.survivalTime.weight;
    score += (member.healing / 50) * MVP_CRITERIA.healing.weight;

    mvpScores.set(member.playerId, score);
  }

  // Get highest scorer
  const mvpId = [...mvpScores.entries()].sort((a, b) => b[1] - a[1])[0][0];
  const mvpMember = team.members.find(m => m.playerId === mvpId);

  // Determine reason
  const mvpReason = determineMVPReason(mvpMember);

  return {
    playerId: mvpId,
    playerName: mvpMember.playerName,
    reason: mvpReason
  };
}
```

## Social Sharing

### Share Card Generation

```typescript
interface ShareCard {
  template: 'victory' | 'defeat' | 'achievement';
  backgroundImage: string;
  characterPose: string;

  // Stats to display
  placement: number;
  kills: number;
  damage: number;

  // Branding
  gameLogoPosition: 'bottom-right';
  appStoreLinks: boolean;
}

function generateShareCard(stats: MatchStats): ImageData {
  const canvas = createCanvas(1200, 630);  // Social media optimal
  const ctx = canvas.getContext('2d');

  // Draw background
  drawBackground(ctx, stats.placement);

  // Draw character
  drawCharacterPose(ctx, player.character, stats.placement);

  // Draw stats overlay
  drawStatsOverlay(ctx, stats);

  // Draw branding
  drawLogo(ctx);
  if (SHARE_CONFIG.appStoreLinks) {
    drawQRCode(ctx);
  }

  return canvas.toDataURL('image/png');
}
```

### Share Options

```typescript
interface ShareOptions {
  platforms: SharePlatform[];
  cardGeneration: boolean;
  includeMatchId: boolean;      // For replay viewing
}

enum SharePlatform {
  TWITTER = 'twitter',
  FACEBOOK = 'facebook',
  INSTAGRAM_STORY = 'instagram_story',
  CLIPBOARD = 'clipboard',
  NATIVE_SHARE = 'native'       // OS share sheet
}

async function shareResults(stats: MatchStats, platform: SharePlatform): Promise<void> {
  const shareCard = await generateShareCard(stats);
  const shareText = generateShareText(stats);

  switch (platform) {
    case SharePlatform.TWITTER:
      openUrl(`https://twitter.com/intent/tweet?text=${encodeURIComponent(shareText)}&url=${gameUrl}`);
      break;

    case SharePlatform.NATIVE_SHARE:
      await navigator.share({
        title: 'Plunderstorm Mobile Results',
        text: shareText,
        files: [new File([shareCard], 'results.png', { type: 'image/png' })]
      });
      break;

    case SharePlatform.CLIPBOARD:
      await navigator.clipboard.writeText(shareText);
      showNotification('Copied to clipboard!');
      break;
  }

  // Analytics
  analytics.track('results_shared', {
    platform,
    placement: stats.placement,
    wasVictory: stats.placement === 1
  });
}

function generateShareText(stats: MatchStats): string {
  if (stats.placement === 1) {
    return `I won in Plunderstorm Mobile! ${stats.kills} eliminations, ${stats.damageDealt} damage dealt. Can you beat that? #PlunderstormMobile #Victory`;
  } else {
    return `Finished #${stats.placement} in Plunderstorm Mobile with ${stats.kills} eliminations! Getting closer to that win! #PlunderstormMobile`;
  }
}
```

## Quick Actions

### Play Again Flow

```typescript
interface PlayAgainConfig {
  sameMode: true;               // Same game mode
  sameTeammate: true;           // In duo mode, same party
  backgroundQueue: boolean;     // Queue while viewing results
  estimatedWaitTime: number;
}

async function handlePlayAgain(): Promise<void> {
  // Start matchmaking immediately
  const matchmakingPromise = startMatchmaking(lastMatchMode);

  // Show "Finding Match..." overlay on results screen
  showMatchmakingOverlay();

  // User can still view stats while waiting
  const match = await matchmakingPromise;

  // Transition to loading screen
  transitionToMatch(match);
}
```

### Background Queuing

```typescript
interface BackgroundQueueState {
  isQueuing: boolean;
  mode: GameMode;
  estimatedTime: number;
  elapsedTime: number;
}

function enableBackgroundQueue(): void {
  // Start queue while still on results screen
  const queueState: BackgroundQueueState = {
    isQueuing: true,
    mode: lastMatchMode,
    estimatedTime: getEstimatedWaitTime(lastMatchMode),
    elapsedTime: 0
  };

  // Update UI to show queue status
  showQueueIndicator(queueState);

  // When match found, prompt user
  onMatchFound(() => {
    showMatchFoundPrompt({
      autoAcceptDelay: 10000,    // 10 seconds to respond
      onAccept: () => transitionToMatch(),
      onDecline: () => cancelQueue()
    });
  });
}
```

## Spectate Option

### Spectate From Results

```typescript
interface SpectateFromResultsConfig {
  available: boolean;           // Only if match still ongoing
  remainingPlayers: number;
  estimatedTimeLeft: number;
}

function checkSpectateAvailable(matchId: string): SpectateFromResultsConfig {
  const matchState = getMatchState(matchId);

  return {
    available: matchState.status === 'in_progress',
    remainingPlayers: matchState.playersAlive,
    estimatedTimeLeft: estimateTimeRemaining(matchState)
  };
}

async function startSpectating(matchId: string): Promise<void> {
  // Transition to spectator mode
  const spectatorSession = await joinAsSpectator(matchId);

  // Start at random alive player or specific player
  spectatorSession.focusPlayer = getRandomAlivePlayer();

  transitionToSpectator(spectatorSession);
}
```

## Motivational Elements

### Defeat Encouragement

```typescript
const DEFEAT_MESSAGES: string[] = [
  "The sea hasn't seen the last of me!",
  "Next time, victory will be mine!",
  "Every pirate faces rough waters sometimes.",
  "Back to the helm - adventure awaits!",
  "A true captain never gives up!",
  "The treasure hunt continues!",
  "Practice makes a legendary pirate!",
  "Set sail again, brave seafarer!"
];

interface DefeatEncouragement {
  message: string;
  showSessionBest: boolean;
  showImprovement: boolean;
  suggestTip?: string;
}

function getDefeatEncouragement(stats: MatchStats): DefeatEncouragement {
  const encouragement: DefeatEncouragement = {
    message: DEFEAT_MESSAGES[Math.floor(Math.random() * DEFEAT_MESSAGES.length)],
    showSessionBest: true,
    showImprovement: false
  };

  // Check for improvement
  const previousBest = getSessionBest();
  if (stats.placement < previousBest.placement) {
    encouragement.showImprovement = true;
    encouragement.message = "New personal best this session!";
  }

  // Suggest contextual tip
  if (stats.survivalTime < 120) {
    encouragement.suggestTip = "Tip: Loot up before engaging enemies!";
  } else if (stats.kills === 0) {
    encouragement.suggestTip = "Tip: Try the Practice mode to improve your aim!";
  }

  return encouragement;
}
```

## Analytics Events

```typescript
// Screen viewed
analytics.track('end_screen_viewed', {
  placement: number,
  screenType: 'victory' | 'podium' | 'top10' | 'defeat',
  matchDuration: number,
  mode: string
});

// Actions taken
analytics.track('end_screen_action', {
  action: 'play_again' | 'return_lobby' | 'share' | 'spectate',
  timeOnScreen: number,
  placement: number
});

// Rewards claimed
analytics.track('match_rewards_viewed', {
  gold: number,
  xp: number,
  battlePassXP: number,
  leveledUp: boolean,
  tierUnlocked: boolean
});

// Share tracking
analytics.track('results_shared', {
  platform: string,
  placement: number,
  wasVictory: boolean
});
```

## Testing Requirements

### Functional Tests

- [ ] Victory screen displays correctly for 1st place
- [ ] Podium screen displays for 2nd-3rd
- [ ] Top 10 screen displays for 4th-10th
- [ ] Defeat screen displays for 11th+
- [ ] All stats calculate and display correctly
- [ ] Rewards calculate correctly based on placement
- [ ] Level up celebration triggers correctly
- [ ] Battle Pass tier unlock shows preview
- [ ] Play Again queues for same mode
- [ ] Return to Lobby works correctly
- [ ] Share generates correct image
- [ ] Spectate option works when match ongoing

### Visual Tests

- [ ] Animations play smoothly (60 FPS)
- [ ] Confetti and effects render correctly
- [ ] Character poses display properly
- [ ] Stats count-up animation works
- [ ] Reward animations complete

### Edge Cases

- [ ] Very high stats display without overflow
- [ ] Zero kills/damage displays gracefully
- [ ] Disconnection during match shows correct stats
- [ ] Multiple level ups handled
- [ ] Tie placements handled

## Success Metrics

| Metric | Target |
|--------|--------|
| Play Again rate | 40%+ of matches |
| Share rate | 5%+ of victories |
| Time on results screen | 15-30 seconds |
| Level up awareness | 95% notice level up |

## Dependencies

- **GAME-005**: Match Results System
- **LIVE-003**: Battle Pass System
- **MON-001**: Virtual Currency System
- **BACK-005**: Account/Progression System
