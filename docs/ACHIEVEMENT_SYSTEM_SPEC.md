# Achievement System Specification

## Document Information
- **Task ID:** LIVE-003
- **Priority:** P2
- **Complexity:** Medium
- **Status:** Complete
- **Last Updated:** 2025-12-08

---

## Overview

This document specifies the achievement system for Plunderstorm Mobile, providing long-term goals and rewards that encourage continued engagement and mastery of game mechanics.

---

## Achievement Structure

### Achievement Tiers

```typescript
const ACHIEVEMENT_TIERS = {
  bronze: {
    id: 'bronze',
    name: 'Bronze',
    color: '#CD7F32',
    icon: 'tier_bronze',
    rewardMultiplier: 1.0,
    description: 'Entry-level achievement'
  },
  silver: {
    id: 'silver',
    name: 'Silver',
    color: '#C0C0C0',
    icon: 'tier_silver',
    rewardMultiplier: 1.5,
    description: 'Intermediate achievement'
  },
  gold: {
    id: 'gold',
    name: 'Gold',
    color: '#FFD700',
    icon: 'tier_gold',
    rewardMultiplier: 2.5,
    description: 'Advanced achievement'
  },
  platinum: {
    id: 'platinum',
    name: 'Platinum',
    color: '#E5E4E2',
    icon: 'tier_platinum',
    rewardMultiplier: 5.0,
    description: 'Master-level achievement'
  }
};

const ACHIEVEMENT_POINTS = {
  bronze: 10,
  silver: 25,
  gold: 50,
  platinum: 100
};
```

### Achievement Categories

```typescript
const ACHIEVEMENT_CATEGORIES = {
  combat: {
    id: 'combat',
    name: 'Combat',
    icon: 'category_combat',
    description: 'Eliminations and combat prowess',
    color: '#FF4444'
  },
  victory: {
    id: 'victory',
    name: 'Victory',
    icon: 'category_victory',
    description: 'Winning matches and placements',
    color: '#FFD700'
  },
  abilities: {
    id: 'abilities',
    name: 'Abilities',
    icon: 'category_abilities',
    description: 'Mastering abilities and combos',
    color: '#9944FF'
  },
  exploration: {
    id: 'exploration',
    name: 'Exploration',
    icon: 'category_exploration',
    description: 'Discovering the map and secrets',
    color: '#44AA44'
  },
  social: {
    id: 'social',
    name: 'Social',
    icon: 'category_social',
    description: 'Playing with friends and community',
    color: '#44AAFF'
  },
  collection: {
    id: 'collection',
    name: 'Collection',
    icon: 'category_collection',
    description: 'Gathering cosmetics and items',
    color: '#FF44AA'
  },
  mastery: {
    id: 'mastery',
    name: 'Mastery',
    icon: 'category_mastery',
    description: 'Reaching ranks and milestones',
    color: '#FFAA44'
  },
  seasonal: {
    id: 'seasonal',
    name: 'Seasonal',
    icon: 'category_seasonal',
    description: 'Limited-time seasonal achievements',
    color: '#44FFAA'
  }
};
```

---

## Launch Achievements

### Combat Achievements

```typescript
const COMBAT_ACHIEVEMENTS = [
  // Kill Milestones
  {
    id: 'first_blood',
    name: 'First Blood',
    description: 'Eliminate your first enemy',
    category: 'combat',
    tier: 'bronze',
    requirement: { type: 'kills', count: 1 },
    reward: { gold: 100 },
    hidden: false
  },
  {
    id: 'hunter_bronze',
    name: 'Hunter',
    description: 'Eliminate 100 enemies',
    category: 'combat',
    tier: 'bronze',
    requirement: { type: 'kills', count: 100 },
    reward: { gold: 250 }
  },
  {
    id: 'hunter_silver',
    name: 'Skilled Hunter',
    description: 'Eliminate 500 enemies',
    category: 'combat',
    tier: 'silver',
    requirement: { type: 'kills', count: 500 },
    reward: { gold: 500, title: 'Hunter' }
  },
  {
    id: 'hunter_gold',
    name: 'Master Hunter',
    description: 'Eliminate 2,500 enemies',
    category: 'combat',
    tier: 'gold',
    requirement: { type: 'kills', count: 2500 },
    reward: { gold: 1000, cosmetic: 'hunter_badge' }
  },
  {
    id: 'hunter_platinum',
    name: 'Legendary Hunter',
    description: 'Eliminate 10,000 enemies',
    category: 'combat',
    tier: 'platinum',
    requirement: { type: 'kills', count: 10000 },
    reward: { gems: 100, title: 'Legendary Hunter', cosmetic: 'hunter_frame' }
  },
  // Per-match achievements
  {
    id: 'killing_spree',
    name: 'Killing Spree',
    description: 'Get 5 eliminations in a single match',
    category: 'combat',
    tier: 'bronze',
    requirement: { type: 'kills_per_match', count: 5 },
    reward: { gold: 150 }
  },
  {
    id: 'rampage',
    name: 'Rampage',
    description: 'Get 10 eliminations in a single match',
    category: 'combat',
    tier: 'silver',
    requirement: { type: 'kills_per_match', count: 10 },
    reward: { gold: 500 }
  },
  {
    id: 'unstoppable',
    name: 'Unstoppable',
    description: 'Get 15 eliminations in a single match',
    category: 'combat',
    tier: 'gold',
    requirement: { type: 'kills_per_match', count: 15 },
    reward: { gold: 1000, title: 'Unstoppable' }
  }
];
```

### Victory Achievements

```typescript
const VICTORY_ACHIEVEMENTS = [
  {
    id: 'first_win',
    name: 'Victory Royale',
    description: 'Win your first match',
    category: 'victory',
    tier: 'bronze',
    requirement: { type: 'wins', count: 1 },
    reward: { gold: 200 }
  },
  {
    id: 'winner_bronze',
    name: 'Winner',
    description: 'Win 10 matches',
    category: 'victory',
    tier: 'bronze',
    requirement: { type: 'wins', count: 10 },
    reward: { gold: 500 }
  },
  {
    id: 'winner_silver',
    name: 'Champion',
    description: 'Win 50 matches',
    category: 'victory',
    tier: 'silver',
    requirement: { type: 'wins', count: 50 },
    reward: { gold: 1000, title: 'Champion' }
  },
  {
    id: 'winner_gold',
    name: 'Grand Champion',
    description: 'Win 250 matches',
    category: 'victory',
    tier: 'gold',
    requirement: { type: 'wins', count: 250 },
    reward: { gems: 50, cosmetic: 'champion_crown' }
  },
  {
    id: 'winner_platinum',
    name: 'Pirate King',
    description: 'Win 1,000 matches',
    category: 'victory',
    tier: 'platinum',
    requirement: { type: 'wins', count: 1000 },
    reward: { gems: 200, title: 'Pirate King', cosmetic: 'king_frame' }
  },
  // Streak achievements
  {
    id: 'win_streak_3',
    name: 'Hot Streak',
    description: 'Win 3 matches in a row',
    category: 'victory',
    tier: 'silver',
    requirement: { type: 'win_streak', count: 3 },
    reward: { gold: 500 }
  },
  {
    id: 'win_streak_5',
    name: 'Dominant',
    description: 'Win 5 matches in a row',
    category: 'victory',
    tier: 'gold',
    requirement: { type: 'win_streak', count: 5 },
    reward: { gold: 1500, title: 'Dominant' }
  }
];
```

### Ability Achievements

```typescript
const ABILITY_ACHIEVEMENTS = [
  {
    id: 'ability_user',
    name: 'Ability User',
    description: 'Use abilities 100 times',
    category: 'abilities',
    tier: 'bronze',
    requirement: { type: 'abilities_used', count: 100 },
    reward: { gold: 200 }
  },
  {
    id: 'ability_master',
    name: 'Ability Master',
    description: 'Use abilities 1,000 times',
    category: 'abilities',
    tier: 'silver',
    requirement: { type: 'abilities_used', count: 1000 },
    reward: { gold: 500 }
  },
  {
    id: 'fireball_expert',
    name: 'Pyromaniac',
    description: 'Get 100 kills with Fireball',
    category: 'abilities',
    tier: 'silver',
    requirement: { type: 'ability_kills', ability: 'fireball', count: 100 },
    reward: { gold: 500, title: 'Pyromaniac' }
  },
  {
    id: 'combo_starter',
    name: 'Combo Starter',
    description: 'Trigger 10 ability combos',
    category: 'abilities',
    tier: 'bronze',
    requirement: { type: 'combos_triggered', count: 10 },
    reward: { gold: 300 }
  },
  {
    id: 'combo_master',
    name: 'Combo Master',
    description: 'Trigger 100 ability combos',
    category: 'abilities',
    tier: 'gold',
    requirement: { type: 'combos_triggered', count: 100 },
    reward: { gold: 1000, cosmetic: 'combo_effect' }
  },
  {
    id: 'max_upgrade',
    name: 'Fully Loaded',
    description: 'Upgrade an ability to Rank 3',
    category: 'abilities',
    tier: 'bronze',
    requirement: { type: 'ability_max_upgrade', count: 1 },
    reward: { gold: 200 }
  }
];
```

### Exploration Achievements

```typescript
const EXPLORATION_ACHIEVEMENTS = [
  {
    id: 'explorer',
    name: 'Explorer',
    description: 'Visit all named locations',
    category: 'exploration',
    tier: 'silver',
    requirement: { type: 'visit_all_pois' },
    reward: { gold: 500, title: 'Explorer' }
  },
  {
    id: 'chest_opener',
    name: 'Treasure Hunter',
    description: 'Open 100 chests',
    category: 'exploration',
    tier: 'bronze',
    requirement: { type: 'chests_opened', count: 100 },
    reward: { gold: 250 }
  },
  {
    id: 'chest_master',
    name: 'Treasure Master',
    description: 'Open 1,000 chests',
    category: 'exploration',
    tier: 'silver',
    requirement: { type: 'chests_opened', count: 1000 },
    reward: { gold: 750 }
  },
  {
    id: 'rare_finder',
    name: 'Lucky Find',
    description: 'Open 50 Epic or Legendary chests',
    category: 'exploration',
    tier: 'gold',
    requirement: { type: 'rare_chests_opened', count: 50 },
    reward: { gold: 1000, cosmetic: 'lucky_charm' }
  },
  {
    id: 'monster_slayer',
    name: 'Monster Slayer',
    description: 'Defeat 500 NPC monsters',
    category: 'exploration',
    tier: 'silver',
    requirement: { type: 'monsters_killed', count: 500 },
    reward: { gold: 500 }
  },
  {
    id: 'event_participant',
    name: 'Event Hunter',
    description: 'Participate in 25 world events',
    category: 'exploration',
    tier: 'silver',
    requirement: { type: 'events_participated', count: 25 },
    reward: { gold: 750 }
  }
];
```

### Social Achievements

```typescript
const SOCIAL_ACHIEVEMENTS = [
  {
    id: 'team_player',
    name: 'Team Player',
    description: 'Play 10 duo matches',
    category: 'social',
    tier: 'bronze',
    requirement: { type: 'duo_matches', count: 10 },
    reward: { gold: 200 }
  },
  {
    id: 'duo_winner',
    name: 'Dynamic Duo',
    description: 'Win 10 duo matches',
    category: 'social',
    tier: 'silver',
    requirement: { type: 'duo_wins', count: 10 },
    reward: { gold: 500 }
  },
  {
    id: 'friend_inviter',
    name: 'Social Butterfly',
    description: 'Play with 10 different friends',
    category: 'social',
    tier: 'silver',
    requirement: { type: 'unique_friends_played', count: 10 },
    reward: { gold: 500, title: 'Social Butterfly' }
  },
  {
    id: 'reviver',
    name: 'Lifesaver',
    description: 'Revive teammates 25 times',
    category: 'social',
    tier: 'silver',
    requirement: { type: 'teammates_revived', count: 25 },
    reward: { gold: 500 }
  },
  {
    id: 'reviver_master',
    name: 'Guardian Angel',
    description: 'Revive teammates 100 times',
    category: 'social',
    tier: 'gold',
    requirement: { type: 'teammates_revived', count: 100 },
    reward: { gold: 1000, title: 'Guardian Angel' }
  }
];
```

### Collection Achievements

```typescript
const COLLECTION_ACHIEVEMENTS = [
  {
    id: 'collector_starter',
    name: 'Collector',
    description: 'Own 10 cosmetics',
    category: 'collection',
    tier: 'bronze',
    requirement: { type: 'cosmetics_owned', count: 10 },
    reward: { gold: 200 }
  },
  {
    id: 'collector_intermediate',
    name: 'Hoarder',
    description: 'Own 50 cosmetics',
    category: 'collection',
    tier: 'silver',
    requirement: { type: 'cosmetics_owned', count: 50 },
    reward: { gold: 500 }
  },
  {
    id: 'collector_advanced',
    name: 'Fashionista',
    description: 'Own 100 cosmetics',
    category: 'collection',
    tier: 'gold',
    requirement: { type: 'cosmetics_owned', count: 100 },
    reward: { gems: 50, title: 'Fashionista' }
  },
  {
    id: 'emote_collector',
    name: 'Expressionist',
    description: 'Own 10 emotes',
    category: 'collection',
    tier: 'silver',
    requirement: { type: 'emotes_owned', count: 10 },
    reward: { gold: 500 }
  }
];
```

### Mastery Achievements

```typescript
const MASTERY_ACHIEVEMENTS = [
  {
    id: 'level_10',
    name: 'Seasoned',
    description: 'Reach account level 10',
    category: 'mastery',
    tier: 'bronze',
    requirement: { type: 'account_level', count: 10 },
    reward: { gold: 250 }
  },
  {
    id: 'level_25',
    name: 'Experienced',
    description: 'Reach account level 25',
    category: 'mastery',
    tier: 'silver',
    requirement: { type: 'account_level', count: 25 },
    reward: { gold: 500 }
  },
  {
    id: 'level_50',
    name: 'Veteran',
    description: 'Reach account level 50',
    category: 'mastery',
    tier: 'gold',
    requirement: { type: 'account_level', count: 50 },
    reward: { gold: 1000, title: 'Veteran' }
  },
  {
    id: 'level_100',
    name: 'Legend',
    description: 'Reach account level 100',
    category: 'mastery',
    tier: 'platinum',
    requirement: { type: 'account_level', count: 100 },
    reward: { gems: 100, title: 'Legend', cosmetic: 'legend_frame' }
  },
  {
    id: 'ranked_gold',
    name: 'Golden Rank',
    description: 'Reach Gold rank in Ranked mode',
    category: 'mastery',
    tier: 'silver',
    requirement: { type: 'ranked_tier', tier: 'gold' },
    reward: { gold: 750 }
  },
  {
    id: 'ranked_diamond',
    name: 'Diamond Rank',
    description: 'Reach Diamond rank in Ranked mode',
    category: 'mastery',
    tier: 'gold',
    requirement: { type: 'ranked_tier', tier: 'diamond' },
    reward: { gems: 50, title: 'Diamond Player' }
  },
  {
    id: 'battle_pass_complete',
    name: 'Pass Master',
    description: 'Complete a Battle Pass to tier 100',
    category: 'mastery',
    tier: 'gold',
    requirement: { type: 'battle_pass_complete' },
    reward: { gold: 1000 }
  }
];
```

### Hidden Achievements

```typescript
const HIDDEN_ACHIEVEMENTS = [
  {
    id: 'hidden_pacifist',
    name: 'Pacifist',
    description: 'Win a match with 0 kills',
    category: 'victory',
    tier: 'gold',
    requirement: { type: 'win_no_kills' },
    reward: { gold: 1500, title: 'Pacifist' },
    hidden: true,
    hint: 'Sometimes the best offense is no offense...'
  },
  {
    id: 'hidden_last_second',
    name: 'Photo Finish',
    description: 'Win while in the storm',
    category: 'victory',
    tier: 'silver',
    requirement: { type: 'win_in_storm' },
    reward: { gold: 750 },
    hidden: true
  },
  {
    id: 'hidden_underdog',
    name: 'Underdog',
    description: 'Win a match after being the last alive in duo',
    category: 'victory',
    tier: 'gold',
    requirement: { type: 'solo_carry_duo' },
    reward: { gold: 1000, title: 'Underdog' },
    hidden: true
  },
  {
    id: 'hidden_survivor',
    name: 'Survivalist',
    description: 'Survive for 15 minutes in a single match',
    category: 'exploration',
    tier: 'silver',
    requirement: { type: 'survival_time', seconds: 900 },
    reward: { gold: 500 },
    hidden: true
  }
];
```

---

## Achievement Tracking

### Tracking System

```typescript
const ACHIEVEMENT_TRACKING = {
  storage: {
    progress: 'Player profile database',
    completed: 'Achievement completion records',
    sync: 'Real-time sync with backend'
  },

  tracking: {
    perSession: {
      description: 'Track progress during gameplay',
      update: 'Batch update at session end',
      realTime: 'Some achievements notify in real-time'
    },
    cumulative: {
      description: 'Track lifetime totals',
      types: ['kills', 'wins', 'abilities_used', 'chests_opened']
    },
    conditional: {
      description: 'Track specific conditions',
      examples: ['win_streak', 'kills_per_match', 'no_kills_win']
    }
  },

  events: {
    onKill: 'Update kill counters',
    onWin: 'Update win counters, check streaks',
    onAbilityUse: 'Update ability counters',
    onChestOpen: 'Update chest counters',
    onMatchEnd: 'Batch update all relevant achievements'
  }
};
```

### Progress Calculation

```typescript
const PROGRESS_CALCULATION = {
  percentage: (current: number, required: number): number => {
    return Math.min(100, Math.floor((current / required) * 100));
  },

  display: {
    numerical: '500 / 1,000 kills',
    percentage: '50% complete',
    bar: 'Visual progress bar'
  },

  nearCompletion: {
    threshold: 90, // 90% complete
    highlight: true,
    notification: 'Almost there!'
  }
};
```

---

## Rewards

### Reward Types

```typescript
const ACHIEVEMENT_REWARDS = {
  gold: {
    description: 'Soft currency',
    range: { min: 100, max: 2000 },
    scaling: 'By tier'
  },

  gems: {
    description: 'Premium currency',
    range: { min: 25, max: 200 },
    availability: 'Gold and Platinum tiers only'
  },

  titles: {
    description: 'Display titles for player name',
    examples: ['Hunter', 'Champion', 'Legend'],
    display: 'Before or after name'
  },

  cosmetics: {
    description: 'Exclusive achievement cosmetics',
    types: ['Profile frames', 'Badges', 'Weapon charms'],
    exclusive: 'Only from achievements'
  },

  points: {
    description: 'Achievement points for total score',
    display: 'Profile achievement score'
  }
};

const REWARD_CLAIMING = {
  automatic: false, // Require manual claim
  notification: 'Badge on achievement UI',
  claimFlow: [
    'Tap achievement',
    'See reward preview',
    'Claim button',
    'Celebration animation',
    'Reward added to inventory'
  ]
};
```

---

## UI/UX

### Achievement Gallery

```typescript
const ACHIEVEMENT_GALLERY_UI = {
  location: 'Profile > Achievements',

  layout: {
    tabs: 'One per category',
    grid: 'Achievement cards in grid',
    sorting: ['Progress', 'Tier', 'Category', 'Recent']
  },

  achievementCard: {
    icon: 'Achievement icon',
    name: 'Achievement name',
    progress: 'Progress bar or checkmark',
    tier: 'Tier color border',
    locked: 'Greyed if incomplete',
    claimable: 'Glow effect if claimable'
  },

  detailView: {
    tap: 'Opens detail modal',
    description: 'Full description',
    progress: 'Detailed progress',
    reward: 'Reward preview',
    claim: 'Claim button if complete'
  },

  statistics: {
    totalPoints: 'Achievement score',
    completion: 'X of Y achievements',
    byCategory: 'Completion per category',
    byTier: 'Count per tier'
  }
};
```

### Notification System

```typescript
const ACHIEVEMENT_NOTIFICATIONS = {
  unlock: {
    trigger: 'On achievement completion',
    display: {
      type: 'Toast notification',
      position: 'Top of screen',
      duration: 3000, // ms
      animation: 'Slide in + celebration effect'
    },
    content: {
      icon: 'Achievement icon',
      name: 'Achievement name',
      tier: 'Tier indicator',
      action: 'Tap to view'
    },
    sound: 'Achievement unlock sound',
    timing: 'End of match (not during combat)'
  },

  progress: {
    trigger: 'On significant milestones (25%, 50%, 75%)',
    display: 'Subtle toast',
    optional: 'Player can disable in settings'
  },

  nearCompletion: {
    trigger: 'When at 90%+ progress',
    display: 'Encouraging message',
    frequency: 'Once per achievement'
  }
};
```

### Profile Display

```typescript
const PROFILE_ACHIEVEMENT_DISPLAY = {
  summary: {
    totalScore: 'Total achievement points',
    completion: 'Overall percentage',
    recentUnlocks: 'Last 3 achievements'
  },

  showcase: {
    slots: 3, // Display 3 featured achievements
    customizable: true,
    display: 'Achievement icons with names'
  },

  viewAll: {
    button: 'View all achievements',
    navigation: 'Opens achievement gallery'
  }
};
```

---

## Backend Integration

### API Endpoints

```typescript
const ACHIEVEMENT_API = {
  getAchievements: {
    method: 'GET',
    path: '/api/v1/achievements',
    response: {
      achievements: 'All achievement definitions',
      version: 'Achievement data version'
    }
  },

  getPlayerProgress: {
    method: 'GET',
    path: '/api/v1/achievements/progress/{playerId}',
    response: {
      progress: 'Map of achievement ID to progress',
      completed: 'List of completed achievement IDs',
      claimable: 'List of claimable achievement IDs',
      totalPoints: 'Total achievement score'
    }
  },

  updateProgress: {
    method: 'POST',
    path: '/api/v1/achievements/progress',
    body: {
      playerId: 'string',
      updates: 'Array of { achievementId, progress }'
    },
    internal: true // Game server only
  },

  claimReward: {
    method: 'POST',
    path: '/api/v1/achievements/claim',
    body: {
      playerId: 'string',
      achievementId: 'string'
    },
    response: {
      success: 'boolean',
      rewards: 'Rewards granted'
    }
  }
};
```

---

## Testing Requirements

```typescript
const ACHIEVEMENT_TESTING = {
  functional: [
    'Progress tracks correctly for all achievement types',
    'Achievements unlock at correct thresholds',
    'Rewards grant correctly on claim',
    'Tiered achievements progress properly',
    'Hidden achievements reveal on completion'
  ],

  ui: [
    'Gallery displays all achievements',
    'Progress bars accurate',
    'Unlock notifications appear',
    'Claim flow works',
    'Profile displays achievement score'
  ],

  edge: [
    'Progress persists across sessions',
    'Offline progress syncs correctly',
    'Duplicate claims prevented',
    'Retroactive credit for existing progress'
  ],

  balance: [
    'Achievement difficulty feels appropriate',
    'Reward values balanced',
    'Time to complete reasonable'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-08 | Development Team | Initial achievement system specification |
