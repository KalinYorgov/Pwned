# Currency Earning Rates Specification

## Overview

This document specifies all Gold (soft currency) earning rates for Plunderstorm Mobile, defining how players earn currency through gameplay, quests, achievements, and other activities. The earning rates are balanced against spending sinks to maintain a healthy economy.

## Gold Earning Sources

### Source Overview

| Source | Gold Amount | Frequency | Daily Cap |
|--------|-------------|-----------|-----------|
| Match Completion | 50-200 | Per match | 2,000 |
| Daily Quests | 100-300 | 3 per day | 600 |
| Weekly Quests | 500-1,000 | 3 per week | N/A |
| First Win of Day | 200 | Once daily | 200 |
| Level Up | 100 | Per level | N/A |
| Achievements | 50-2,000 | One-time | N/A |
| Battle Pass (Free) | Varies | Per tier | N/A |

## Match Completion Rewards

### Placement-Based Rewards

```typescript
interface MatchRewardConfig {
  mode: GameMode;
  placementRewards: PlacementReward[];
  killBonus: number;
  damageBonus: DamageBonus;
  survivalBonus: SurvivalBonus;
  firstWinBonus: number;
}

interface PlacementReward {
  placementRange: [number, number];  // [min, max]
  baseGold: number;
}

const SOLO_MATCH_REWARDS: MatchRewardConfig = {
  mode: GameMode.SOLO,
  placementRewards: [
    { placementRange: [1, 1], baseGold: 150 },      // Victory
    { placementRange: [2, 3], baseGold: 100 },      // Top 3
    { placementRange: [4, 5], baseGold: 75 },       // Top 5
    { placementRange: [6, 10], baseGold: 60 },      // Top 10
    { placementRange: [11, 20], baseGold: 50 },     // Top 20
    { placementRange: [21, 30], baseGold: 40 },     // Top 30
    { placementRange: [31, 60], baseGold: 30 }      // Eliminated early
  ],
  killBonus: 10,                // Per elimination
  damageBonus: {
    thresholds: [
      { damage: 500, bonus: 5 },
      { damage: 1000, bonus: 10 },
      { damage: 2000, bonus: 20 }
    ]
  },
  survivalBonus: {
    perMinute: 2,               // Gold per minute survived
    maxMinutes: 15
  },
  firstWinBonus: 200
};

const DUO_MATCH_REWARDS: MatchRewardConfig = {
  mode: GameMode.DUO,
  placementRewards: [
    { placementRange: [1, 1], baseGold: 175 },      // Victory
    { placementRange: [2, 3], baseGold: 120 },      // Top 3 teams
    { placementRange: [4, 5], baseGold: 90 },       // Top 5 teams
    { placementRange: [6, 10], baseGold: 70 },      // Top 10 teams
    { placementRange: [11, 15], baseGold: 55 },     // Top 15 teams
    { placementRange: [16, 30], baseGold: 40 }      // Eliminated early
  ],
  killBonus: 10,
  damageBonus: {
    thresholds: [
      { damage: 500, bonus: 5 },
      { damage: 1000, bonus: 10 },
      { damage: 2000, bonus: 20 }
    ]
  },
  survivalBonus: {
    perMinute: 2,
    maxMinutes: 15
  },
  firstWinBonus: 200
};
```

### Match Reward Calculation

```typescript
interface MatchStats {
  placement: number;
  kills: number;
  damageDealt: number;
  survivalTime: number;         // seconds
  isFirstWinOfDay: boolean;
}

function calculateMatchGold(
  stats: MatchStats,
  config: MatchRewardConfig
): MatchGoldBreakdown {
  // Base placement reward
  const placementReward = config.placementRewards.find(
    r => stats.placement >= r.placementRange[0] && stats.placement <= r.placementRange[1]
  );
  const baseGold = placementReward?.baseGold || 20;

  // Kill bonus
  const killGold = stats.kills * config.killBonus;

  // Damage bonus
  let damageGold = 0;
  for (const threshold of config.damageBonus.thresholds) {
    if (stats.damageDealt >= threshold.damage) {
      damageGold = threshold.bonus;
    }
  }

  // Survival bonus
  const survivalMinutes = Math.min(
    Math.floor(stats.survivalTime / 60),
    config.survivalBonus.maxMinutes
  );
  const survivalGold = survivalMinutes * config.survivalBonus.perMinute;

  // First win of day
  const firstWinGold = stats.isFirstWinOfDay && stats.placement === 1
    ? config.firstWinBonus
    : 0;

  // Calculate total
  const totalGold = baseGold + killGold + damageGold + survivalGold + firstWinGold;

  return {
    baseGold,
    killGold,
    damageGold,
    survivalGold,
    firstWinGold,
    totalGold,
    breakdown: [
      { label: 'Placement', amount: baseGold },
      { label: `Eliminations (${stats.kills})`, amount: killGold },
      { label: 'Damage Bonus', amount: damageGold },
      { label: 'Survival Time', amount: survivalGold },
      ...(firstWinGold > 0 ? [{ label: 'First Win of Day!', amount: firstWinGold }] : [])
    ]
  };
}
```

### Expected Match Earnings

| Player Performance | Gold Earned (Approx) |
|-------------------|---------------------|
| Victory (5 kills, 2000 dmg, 12 min) | 150 + 50 + 20 + 24 = 244 |
| Top 5 (3 kills, 1000 dmg, 8 min) | 75 + 30 + 10 + 16 = 131 |
| Top 10 (2 kills, 500 dmg, 6 min) | 60 + 20 + 5 + 12 = 97 |
| Early elimination (0 kills, 200 dmg, 2 min) | 30 + 0 + 0 + 4 = 34 |
| First Win of Day | +200 bonus |

## Daily Quests

### Quest Reward Structure

```typescript
interface DailyQuest {
  id: string;
  type: QuestType;
  description: string;
  requirement: number;
  goldReward: number;
  xpReward: number;
  difficulty: 'easy' | 'medium' | 'hard';
}

enum QuestType {
  PLAY_MATCHES = 'play_matches',
  GET_KILLS = 'get_kills',
  DEAL_DAMAGE = 'deal_damage',
  TOP_PLACEMENT = 'top_placement',
  USE_ABILITY = 'use_ability',
  OPEN_CHESTS = 'open_chests',
  SURVIVE_TIME = 'survive_time',
  WIN_MATCH = 'win_match'
}

const DAILY_QUEST_POOL: DailyQuest[] = [
  // Easy quests (100 Gold)
  {
    id: 'daily_play_2',
    type: QuestType.PLAY_MATCHES,
    description: 'Play 2 matches',
    requirement: 2,
    goldReward: 100,
    xpReward: 200,
    difficulty: 'easy'
  },
  {
    id: 'daily_chests_5',
    type: QuestType.OPEN_CHESTS,
    description: 'Open 5 treasure chests',
    requirement: 5,
    goldReward: 100,
    xpReward: 200,
    difficulty: 'easy'
  },
  {
    id: 'daily_survive_10',
    type: QuestType.SURVIVE_TIME,
    description: 'Survive for 10 minutes total',
    requirement: 600,  // seconds
    goldReward: 100,
    xpReward: 200,
    difficulty: 'easy'
  },

  // Medium quests (200 Gold)
  {
    id: 'daily_kills_5',
    type: QuestType.GET_KILLS,
    description: 'Get 5 eliminations',
    requirement: 5,
    goldReward: 200,
    xpReward: 400,
    difficulty: 'medium'
  },
  {
    id: 'daily_damage_3000',
    type: QuestType.DEAL_DAMAGE,
    description: 'Deal 3000 damage',
    requirement: 3000,
    goldReward: 200,
    xpReward: 400,
    difficulty: 'medium'
  },
  {
    id: 'daily_top10_twice',
    type: QuestType.TOP_PLACEMENT,
    description: 'Finish in Top 10 twice',
    requirement: 2,
    goldReward: 200,
    xpReward: 400,
    difficulty: 'medium'
  },

  // Hard quests (300 Gold)
  {
    id: 'daily_win_1',
    type: QuestType.WIN_MATCH,
    description: 'Win a match',
    requirement: 1,
    goldReward: 300,
    xpReward: 600,
    difficulty: 'hard'
  },
  {
    id: 'daily_kills_10',
    type: QuestType.GET_KILLS,
    description: 'Get 10 eliminations',
    requirement: 10,
    goldReward: 300,
    xpReward: 600,
    difficulty: 'hard'
  }
];
```

### Daily Quest Selection

```typescript
interface DailyQuestConfig {
  questsPerDay: 3;
  difficultyDistribution: {
    easy: 1;
    medium: 1;
    hard: 1;
  };
  refreshTime: '00:00 UTC';
  maxCarryOver: 0;            // Can't save quests for next day
}

function selectDailyQuests(player: Player): DailyQuest[] {
  const pool = DAILY_QUEST_POOL;

  // Select one of each difficulty
  const easy = selectRandomFromDifficulty(pool, 'easy');
  const medium = selectRandomFromDifficulty(pool, 'medium');
  const hard = selectRandomFromDifficulty(pool, 'hard');

  // Avoid duplicate quest types
  const selected = [easy];
  if (medium.type !== easy.type) {
    selected.push(medium);
  } else {
    selected.push(selectRandomFromDifficulty(pool, 'medium', [easy.type]));
  }

  // Add hard quest avoiding duplicates
  const usedTypes = selected.map(q => q.type);
  const hardFiltered = pool.filter(q => q.difficulty === 'hard' && !usedTypes.includes(q.type));
  selected.push(hardFiltered[Math.floor(Math.random() * hardFiltered.length)] || hard);

  return selected;
}
```

### Daily Quest Earnings

| Difficulty | Gold | Quests/Day | Daily Total |
|------------|------|------------|-------------|
| Easy | 100 | 1 | 100 |
| Medium | 200 | 1 | 200 |
| Hard | 300 | 1 | 300 |
| **Total** | - | 3 | **600** |

## Weekly Quests

### Weekly Quest Structure

```typescript
interface WeeklyQuest {
  id: string;
  type: QuestType;
  description: string;
  requirement: number;
  goldReward: number;
  xpReward: number;
  battlePassXP: number;
}

const WEEKLY_QUEST_POOL: WeeklyQuest[] = [
  {
    id: 'weekly_play_15',
    type: QuestType.PLAY_MATCHES,
    description: 'Play 15 matches this week',
    requirement: 15,
    goldReward: 500,
    xpReward: 1000,
    battlePassXP: 2000
  },
  {
    id: 'weekly_kills_30',
    type: QuestType.GET_KILLS,
    description: 'Get 30 eliminations this week',
    requirement: 30,
    goldReward: 750,
    xpReward: 1500,
    battlePassXP: 2500
  },
  {
    id: 'weekly_wins_3',
    type: QuestType.WIN_MATCH,
    description: 'Win 3 matches this week',
    requirement: 3,
    goldReward: 1000,
    xpReward: 2000,
    battlePassXP: 3000
  },
  {
    id: 'weekly_top5_10',
    type: QuestType.TOP_PLACEMENT,
    description: 'Finish Top 5 in 10 matches',
    requirement: 10,
    goldReward: 750,
    xpReward: 1500,
    battlePassXP: 2500
  },
  {
    id: 'weekly_damage_25000',
    type: QuestType.DEAL_DAMAGE,
    description: 'Deal 25,000 total damage',
    requirement: 25000,
    goldReward: 600,
    xpReward: 1200,
    battlePassXP: 2000
  }
];

interface WeeklyQuestConfig {
  questsPerWeek: 3;
  refreshDay: 'Monday';
  refreshTime: '00:00 UTC';
  carryOverUnfinished: false;
}

function selectWeeklyQuests(player: Player): WeeklyQuest[] {
  // Select 3 varied quests, avoid same types
  const pool = shuffleArray([...WEEKLY_QUEST_POOL]);
  const selected: WeeklyQuest[] = [];
  const usedTypes: QuestType[] = [];

  for (const quest of pool) {
    if (!usedTypes.includes(quest.type) && selected.length < 3) {
      selected.push(quest);
      usedTypes.push(quest.type);
    }
  }

  return selected;
}
```

### Weekly Quest Earnings

| Quest Type | Gold Range | Avg Earnings |
|------------|------------|--------------|
| Weekly Quest 1 | 500-750 | 625 |
| Weekly Quest 2 | 500-1000 | 750 |
| Weekly Quest 3 | 600-1000 | 800 |
| **Weekly Total** | 1,600-2,750 | **~2,175** |

## Level Up Rewards

### Player Level System

```typescript
interface LevelReward {
  level: number;
  goldReward: number;
  additionalRewards?: AdditionalReward[];
}

const LEVEL_REWARDS: LevelReward[] = [
  // Levels 1-10: New player progression
  { level: 1, goldReward: 100 },
  { level: 2, goldReward: 100 },
  { level: 3, goldReward: 100 },
  { level: 4, goldReward: 100 },
  { level: 5, goldReward: 150, additionalRewards: [{ type: 'chest', id: 'common_chest' }] },
  { level: 6, goldReward: 100 },
  { level: 7, goldReward: 100 },
  { level: 8, goldReward: 100 },
  { level: 9, goldReward: 100 },
  { level: 10, goldReward: 200, additionalRewards: [{ type: 'unlock', id: 'ranked_mode' }] },

  // Levels 11-25: Intermediate
  { level: 15, goldReward: 150 },
  { level: 20, goldReward: 200, additionalRewards: [{ type: 'chest', id: 'rare_chest' }] },
  { level: 25, goldReward: 250 },

  // Levels 26-50: Advanced
  { level: 30, goldReward: 200 },
  { level: 40, goldReward: 250 },
  { level: 50, goldReward: 500, additionalRewards: [{ type: 'chest', id: 'epic_chest' }] },

  // Levels 51-100: Veteran (repeat pattern)
  // Standard: 100 Gold per level
  // Milestones: 200-500 Gold at 60, 70, 80, 90, 100
];

// Default level reward
const DEFAULT_LEVEL_REWARD = 100;

function getLevelReward(level: number): LevelReward {
  const specific = LEVEL_REWARDS.find(r => r.level === level);
  if (specific) return specific;

  // Milestone levels
  if (level % 10 === 0) {
    return { level, goldReward: 200 };
  }

  // Standard level
  return { level, goldReward: DEFAULT_LEVEL_REWARD };
}
```

## First Win of the Day

### First Win Bonus

```typescript
interface FirstWinConfig {
  goldBonus: 200;
  xpBonus: 100;
  battlePassXPBonus: 50;
  resetTime: '00:00 UTC';
  requiresVictory: true;        // Must be placement #1
  stackWithMatchReward: true;
}

const FIRST_WIN_CONFIG: FirstWinConfig = {
  goldBonus: 200,
  xpBonus: 100,
  battlePassXPBonus: 50,
  resetTime: '00:00 UTC',
  requiresVictory: true,
  stackWithMatchReward: true
};

function checkFirstWinEligible(player: Player): boolean {
  const lastWin = player.lastFirstWinClaim;
  const now = new Date();

  // Check if last win was before today's reset
  const todayReset = getTodayResetTime(FIRST_WIN_CONFIG.resetTime);
  return !lastWin || lastWin < todayReset;
}

function claimFirstWin(player: Player): FirstWinReward {
  if (!checkFirstWinEligible(player)) {
    return null;
  }

  // Grant rewards
  addGold(player, FIRST_WIN_CONFIG.goldBonus);
  addXP(player, FIRST_WIN_CONFIG.xpBonus);
  addBattlePassXP(player, FIRST_WIN_CONFIG.battlePassXPBonus);

  // Mark as claimed
  player.lastFirstWinClaim = new Date();

  return {
    goldBonus: FIRST_WIN_CONFIG.goldBonus,
    xpBonus: FIRST_WIN_CONFIG.xpBonus,
    battlePassXPBonus: FIRST_WIN_CONFIG.battlePassXPBonus
  };
}
```

## Achievements

### Achievement Rewards

```typescript
interface Achievement {
  id: string;
  name: string;
  description: string;
  category: AchievementCategory;
  tiers?: AchievementTier[];      // Progressive achievements
  goldReward: number;
  isSecret: boolean;
}

enum AchievementCategory {
  COMBAT = 'combat',
  SURVIVAL = 'survival',
  PROGRESSION = 'progression',
  SOCIAL = 'social',
  MASTERY = 'mastery'
}

interface AchievementTier {
  tier: number;
  requirement: number;
  goldReward: number;
}

const ACHIEVEMENTS: Achievement[] = [
  // Combat achievements
  {
    id: 'first_blood',
    name: 'First Blood',
    description: 'Get your first elimination',
    category: AchievementCategory.COMBAT,
    goldReward: 50,
    isSecret: false
  },
  {
    id: 'killing_spree',
    name: 'Killing Spree',
    description: 'Get 5 eliminations in one match',
    category: AchievementCategory.COMBAT,
    goldReward: 200,
    isSecret: false
  },
  {
    id: 'total_kills',
    name: 'Pirate Hunter',
    description: 'Get total eliminations',
    category: AchievementCategory.COMBAT,
    tiers: [
      { tier: 1, requirement: 10, goldReward: 100 },
      { tier: 2, requirement: 50, goldReward: 200 },
      { tier: 3, requirement: 100, goldReward: 300 },
      { tier: 4, requirement: 500, goldReward: 500 },
      { tier: 5, requirement: 1000, goldReward: 1000 }
    ],
    goldReward: 0,  // Tier-based
    isSecret: false
  },

  // Survival achievements
  {
    id: 'first_win',
    name: 'Champion',
    description: 'Win your first match',
    category: AchievementCategory.SURVIVAL,
    goldReward: 500,
    isSecret: false
  },
  {
    id: 'total_wins',
    name: 'Legendary Pirate',
    description: 'Win matches',
    category: AchievementCategory.SURVIVAL,
    tiers: [
      { tier: 1, requirement: 1, goldReward: 500 },
      { tier: 2, requirement: 10, goldReward: 500 },
      { tier: 3, requirement: 50, goldReward: 1000 },
      { tier: 4, requirement: 100, goldReward: 1500 },
      { tier: 5, requirement: 500, goldReward: 2000 }
    ],
    goldReward: 0,
    isSecret: false
  },

  // Secret achievements
  {
    id: 'pacifist_win',
    name: 'Pacifist',
    description: 'Win a match without getting any eliminations',
    category: AchievementCategory.SURVIVAL,
    goldReward: 1000,
    isSecret: true
  }
];
```

### Achievement Earnings Summary

| Achievement Type | Gold Range | Notes |
|-----------------|------------|-------|
| Basic (one-time) | 50-500 | Single completion |
| Tiered (per tier) | 100-2,000 | Progressive |
| Secret | 500-1,000 | Hidden until unlocked |
| **Total Available** | ~15,000+ | All achievements |

## Daily Earnings Cap

### Anti-Farming Measures

```typescript
interface DailyCapConfig {
  enabled: boolean;
  matchGoldCap: number;         // Max Gold from matches per day
  totalDailyCap?: number;       // Optional total daily cap
  warningThreshold: number;     // Warn when approaching cap
  resetTime: '00:00 UTC';
}

const DAILY_CAP_CONFIG: DailyCapConfig = {
  enabled: true,
  matchGoldCap: 2000,           // ~15-20 matches worth
  totalDailyCap: undefined,     // No total cap (quests not capped)
  warningThreshold: 0.8,        // Warn at 80%
  resetTime: '00:00 UTC'
};

class DailyEarningsTracker {
  private matchGoldEarned: number = 0;
  private lastReset: Date;

  constructor(player: Player) {
    this.lastReset = player.lastDailyReset;
    this.matchGoldEarned = player.todayMatchGold;
    this.checkReset();
  }

  private checkReset(): void {
    const todayReset = getTodayResetTime(DAILY_CAP_CONFIG.resetTime);
    if (this.lastReset < todayReset) {
      this.matchGoldEarned = 0;
      this.lastReset = todayReset;
    }
  }

  addMatchGold(amount: number): number {
    this.checkReset();

    if (!DAILY_CAP_CONFIG.enabled) {
      this.matchGoldEarned += amount;
      return amount;
    }

    const remaining = DAILY_CAP_CONFIG.matchGoldCap - this.matchGoldEarned;
    const actualGold = Math.min(amount, Math.max(0, remaining));

    this.matchGoldEarned += actualGold;

    // Show warning if approaching cap
    if (this.matchGoldEarned >= DAILY_CAP_CONFIG.matchGoldCap * DAILY_CAP_CONFIG.warningThreshold) {
      showCapWarning(this.matchGoldEarned, DAILY_CAP_CONFIG.matchGoldCap);
    }

    return actualGold;
  }

  getRemainingCap(): number {
    this.checkReset();
    return Math.max(0, DAILY_CAP_CONFIG.matchGoldCap - this.matchGoldEarned);
  }
}
```

## Expected Daily/Weekly Earnings

### Daily Earnings Breakdown

| Source | Casual (3 matches) | Active (8 matches) | Hardcore (15 matches) |
|--------|-------------------|-------------------|----------------------|
| Match Gold | ~250 | ~650 | ~1,200 |
| Daily Quests | 600 | 600 | 600 |
| First Win | 0-200 | 200 | 200 |
| **Daily Total** | **850-1,050** | **1,450** | **2,000** |

### Weekly Earnings Breakdown

| Source | Casual | Active | Hardcore |
|--------|--------|--------|----------|
| Daily Earnings (x7) | ~6,300 | ~10,150 | ~14,000 |
| Weekly Quests | ~1,500 | ~2,000 | ~2,200 |
| Level Ups (~2/week) | ~200 | ~300 | ~400 |
| **Weekly Total** | **~8,000** | **~12,450** | **~16,600** |

## UI Display

### End Match Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GOLD EARNED                                                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 🏆 Placement (#3)                           +100 Gold   │   │
│  │ ⚔️ Eliminations (4)                          +40 Gold   │   │
│  │ 💥 Damage Bonus                              +10 Gold   │   │
│  │ ⏱️ Survival Time (8 min)                     +16 Gold   │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                    Total:   +166 Gold   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Daily Cap: 650/2,000 Gold                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

With First Win:
┌─────────────────────────────────────────────────────────────────┐
│  │ 🏆 Placement (#1)                           +150 Gold   │   │
│  │ ⭐ FIRST WIN OF THE DAY!                    +200 Gold   │   │
│  │ ⚔️ Eliminations (5)                          +50 Gold   │   │
│  │ 💥 Damage Bonus                              +20 Gold   │   │
│  │ ⏱️ Survival Time (12 min)                    +24 Gold   │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                    Total:   +444 Gold   │   │
└─────────────────────────────────────────────────────────────────┘
```

### Quest Reward Display

```
┌─────────────────────────────────────────────────────────────────┐
│  🎯 QUEST COMPLETE!                                            │
│                                                                 │
│  "Get 5 eliminations"                                          │
│                                                                 │
│  Rewards:                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 💰 +200 Gold                                            │   │
│  │ ⭐ +400 XP                                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│                    [Claim]                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Analytics Events

```typescript
// Gold earned tracking
analytics.track('gold_earned', {
  source: 'match' | 'quest' | 'achievement' | 'level_up' | 'first_win',
  amount: number,
  matchId?: string,
  questId?: string,
  achievementId?: string
});

// Daily earnings
analytics.track('daily_earnings_summary', {
  date: string,
  matchGold: number,
  questGold: number,
  otherGold: number,
  totalGold: number,
  matchesPlayed: number,
  questsCompleted: number,
  hitDailyCap: boolean
});

// Cap hit
analytics.track('daily_cap_reached', {
  matchesPlayed: number,
  timeToReach: number
});
```

## Testing Requirements

### Functional Tests

- [ ] All earning sources grant correct Gold
- [ ] Match placement rewards correct
- [ ] Kill/damage/survival bonuses calculate correctly
- [ ] First Win of Day works and resets
- [ ] Daily quests reward correct amounts
- [ ] Weekly quests reward correct amounts
- [ ] Level up rewards granted
- [ ] Achievements grant Gold
- [ ] Daily cap enforced
- [ ] Cap resets at correct time

### Balance Tests

- [ ] Casual player earnings match projections
- [ ] Active player earnings match projections
- [ ] Time-to-earn for shop items reasonable
- [ ] No exploits for excessive earning

## Success Metrics

| Metric | Target |
|--------|--------|
| Average daily Gold (active player) | 1,200-1,500 |
| Daily cap hit rate | < 5% of players |
| Quest completion rate | > 60% |
| First Win claim rate | > 40% of daily players |

## Dependencies

- **MON-001**: Dual Currency System
- **LIVE-001**: Daily Quest System
- **BACK-011**: Economy Backend
