# Guilds/Crews System Specification

## Document Information
- **Task ID:** LIVE-017
- **Priority:** P3
- **Complexity:** Large
- **Status:** Complete
- **Last Updated:** 2025-12-09

---

## Overview

This document specifies the Guilds/Crews System for Plunderstorm Mobile, providing players with persistent social communities that strengthen retention through social bonds, shared goals, and exclusive guild benefits.

---

## Guild Philosophy

### Design Goals

```typescript
const GUILD_GOALS = {
  community: {
    description: 'Create lasting player communities',
    principle: 'Social bonds drive retention',
    target: '50%+ of active players in guilds'
  },

  identity: {
    description: 'Give groups shared identity',
    principle: 'Guild pride and representation',
    expression: 'Custom name, tag, banner, colors'
  },

  cooperation: {
    description: 'Encourage playing together',
    principle: 'Shared goals and rewards',
    activities: 'Guild quests, challenges, events'
  },

  progression: {
    description: 'Long-term guild growth',
    principle: 'Guilds level up together',
    rewards: 'Perks unlock at guild levels'
  }
};
```

---

## Guild Creation

### Creation Process

```typescript
const GUILD_CREATION = {
  requirements: {
    playerLevel: 10,
    cost: 1000,
    currency: 'gold',
    cooldown: '7 days after leaving/disbanding a guild'
  },

  setup: {
    name: {
      minLength: 3,
      maxLength: 20,
      restrictions: 'Profanity filter, uniqueness required',
      changeCost: 500,
      changeCurrency: 'doubloons'
    },
    tag: {
      length: '2-5 characters',
      restrictions: 'Alphanumeric, profanity filter',
      display: '[TAG] before guild name',
      changeCost: 250,
      changeCurrency: 'doubloons'
    },
    banner: {
      base: 'Choose from 20 base designs',
      colors: 'Primary and secondary color selection',
      icon: 'Choose from 50 guild icons',
      customization: 'Unlock more via guild levels'
    },
    description: {
      maxLength: 200,
      purpose: 'Describe guild focus/requirements'
    },
    settings: {
      joinType: ['Open', 'Request to Join', 'Invite Only'],
      minLevel: 'Minimum player level to join (0-50)',
      language: 'Primary guild language'
    }
  }
};
```

### Guild Limits

```typescript
const GUILD_LIMITS = {
  memberCap: {
    base: 30,
    maxWithPerks: 50,
    scaling: 'Increases with guild level'
  },

  perPlayer: {
    guilds: 1,
    reason: 'Focus loyalty and identity'
  },

  naming: {
    reserved: 'Official/partner names reserved',
    inappropriate: 'Auto-reject inappropriate names',
    appeal: 'Manual review for edge cases'
  }
};
```

---

## Guild Membership

### Joining a Guild

```typescript
const JOINING_GUILD = {
  discovery: {
    search: 'Search guilds by name or tag',
    browse: 'Browse recommended guilds',
    filters: ['Language', 'Activity level', 'Open/Request'],
    recommendations: 'Based on friends, region, playstyle'
  },

  joinMethods: {
    open: {
      process: 'Instant join',
      requirement: 'Meet minimum level'
    },
    request: {
      process: 'Submit request, await approval',
      message: 'Optional join message',
      timeout: 'Request expires in 7 days',
      notification: 'Guild officers notified'
    },
    invite: {
      process: 'Receive invite from member',
      notification: 'In-game and push notification',
      expiration: '7 days'
    }
  },

  restrictions: {
    cooldown: '24 hours after leaving a guild',
    banned: 'Cannot join if banned from guild',
    level: 'Must meet guild minimum level'
  }
};
```

### Leaving a Guild

```typescript
const LEAVING_GUILD = {
  voluntary: {
    process: 'Leave anytime from guild menu',
    confirmation: 'Confirm leave dialog',
    cooldown: '24 hours before joining another'
  },

  kicked: {
    process: 'Officer/leader removes member',
    notification: 'Player notified of removal',
    reason: 'Optional reason provided',
    cooldown: '24 hours before rejoining same guild'
  },

  banned: {
    process: 'Permanent ban from specific guild',
    authority: 'Leader and officers only',
    appeal: 'Can appeal to guild leader',
    duration: 'Permanent unless unbanned'
  },

  leader: {
    transfer: 'Must transfer leadership before leaving',
    disband: 'Or disband guild if sole member',
    inactive: 'Auto-transfer after 30 days inactivity'
  }
};
```

---

## Guild Ranks

### Rank Structure

```typescript
const GUILD_RANKS = {
  leader: {
    name: 'Captain',
    count: 1,
    permissions: [
      'All permissions',
      'Promote/demote officers',
      'Disband guild',
      'Transfer leadership',
      'Edit guild settings',
      'Manage guild bank'
    ],
    icon: 'Crown'
  },

  officer: {
    name: 'First Mate',
    count: 'Up to 5',
    permissions: [
      'Invite members',
      'Accept join requests',
      'Kick members',
      'Ban members',
      'Edit guild description',
      'Start guild activities'
    ],
    icon: 'Anchor'
  },

  veteran: {
    name: 'Veteran',
    count: 'Unlimited',
    permissions: [
      'Invite members',
      'Access veteran chat',
      'Vote in guild polls'
    ],
    requirement: '30 days membership OR 500 guild contribution',
    icon: 'Shield'
  },

  member: {
    name: 'Crew',
    count: 'Unlimited',
    permissions: [
      'Participate in activities',
      'Guild chat access',
      'View guild info'
    ],
    icon: 'Sword'
  },

  recruit: {
    name: 'Recruit',
    count: 'Unlimited',
    permissions: [
      'Limited guild chat',
      'Participate in activities'
    ],
    duration: 'First 7 days of membership',
    icon: 'Flag'
  }
};
```

### Custom Ranks

```typescript
const CUSTOM_RANKS = {
  enabled: true,
  unlockLevel: 10,

  customization: {
    rename: 'Rename default rank names',
    colors: 'Custom rank colors',
    icons: 'Custom rank icons from unlocked set'
  },

  additionalRanks: {
    maxCustom: 3,
    configuration: 'Set permissions from available options',
    hierarchy: 'Place between existing ranks'
  }
};
```

---

## Guild Chat

### Chat System

```typescript
const GUILD_CHAT = {
  channels: {
    general: {
      name: 'Crew Quarters',
      access: 'All members',
      purpose: 'General discussion'
    },
    officers: {
      name: 'Officer Cabin',
      access: 'Officers and leader',
      purpose: 'Leadership discussion'
    },
    veterans: {
      name: 'Veteran Hall',
      access: 'Veterans and above',
      purpose: 'Trusted member discussion'
    },
    events: {
      name: 'Battle Plans',
      access: 'All members',
      purpose: 'Event coordination'
    }
  },

  features: {
    text: 'Standard text messages',
    emojis: 'Standard and guild-specific emojis',
    mentions: '@member, @rank, @everyone (officers+)',
    pins: 'Officers can pin important messages',
    history: '7 days message history'
  },

  moderation: {
    mute: 'Officers can mute members',
    slowMode: 'Optional slow mode (1 msg per X seconds)',
    filter: 'Profanity filter',
    report: 'Report inappropriate messages'
  }
};
```

### Announcements

```typescript
const GUILD_ANNOUNCEMENTS = {
  system: {
    automatic: [
      'Member joined',
      'Member left',
      'Member promoted/demoted',
      'Guild leveled up',
      'Challenge completed',
      'Perk unlocked'
    ],
    display: 'System message in general chat'
  },

  custom: {
    authority: 'Officers and leader',
    features: ['Rich text', 'Images', 'Scheduled'],
    notification: 'Push notification option',
    pin: 'Auto-pinned to top'
  }
};
```

---

## Guild Progression

### Guild XP

```typescript
const GUILD_XP = {
  sources: {
    memberActivity: {
      matchPlayed: 5,
      matchWon: 15,
      questCompleted: 10,
      description: 'Daily cap per member: 100 XP'
    },
    guildChallenges: {
      daily: 50,
      weekly: 200,
      special: 500,
      description: 'Bonus XP for challenge completion'
    },
    guildEvents: {
      participation: 100,
      victory: 300,
      description: 'Guild vs guild events'
    }
  },

  calculation: {
    dailyCap: 'Total guild XP capped at members × 100',
    bonuses: 'Active guild bonus: +25% if 10+ active members'
  }
};
```

### Guild Levels

```typescript
const GUILD_LEVELS = {
  maxLevel: 20,

  xpRequired: [
    { level: 1, xp: 0 },
    { level: 2, xp: 1000 },
    { level: 3, xp: 3000 },
    { level: 4, xp: 6000 },
    { level: 5, xp: 10000 },
    { level: 6, xp: 15000 },
    { level: 7, xp: 22000 },
    { level: 8, xp: 30000 },
    { level: 9, xp: 40000 },
    { level: 10, xp: 55000 },
    { level: 11, xp: 75000 },
    { level: 12, xp: 100000 },
    { level: 13, xp: 130000 },
    { level: 14, xp: 170000 },
    { level: 15, xp: 220000 },
    { level: 16, xp: 280000 },
    { level: 17, xp: 350000 },
    { level: 18, xp: 440000 },
    { level: 19, xp: 550000 },
    { level: 20, xp: 700000 }
  ],

  display: {
    profile: 'Guild level shown on member profiles',
    search: 'Guild level in search results',
    banner: 'Level badge on guild banner'
  }
};
```

---

## Guild Perks

### Perk Unlocks

```typescript
const GUILD_PERKS = {
  level2: {
    perk: 'Guild Banner Display',
    description: 'Show guild banner in matches'
  },
  level3: {
    perk: '+5% XP Bonus',
    description: 'All members earn 5% bonus XP'
  },
  level4: {
    perk: 'Guild Emote Slot 1',
    description: 'Unlock first custom guild emote'
  },
  level5: {
    perk: '+5 Member Capacity',
    description: 'Guild cap increases to 35'
  },
  level6: {
    perk: '+5% Gold Bonus',
    description: 'All members earn 5% bonus Gold'
  },
  level7: {
    perk: 'Guild Spray',
    description: 'Custom guild spray unlocked'
  },
  level8: {
    perk: 'Veteran Channel',
    description: 'Unlock veterans-only chat'
  },
  level9: {
    perk: '+5% XP Bonus (Total 10%)',
    description: 'Stacks with level 3 perk'
  },
  level10: {
    perk: '+10 Member Capacity',
    description: 'Guild cap increases to 45'
  },
  level12: {
    perk: 'Guild Emote Slot 2',
    description: 'Second custom guild emote'
  },
  level14: {
    perk: '+5% Gold Bonus (Total 10%)',
    description: 'Stacks with level 6 perk'
  },
  level16: {
    perk: 'Guild Trail',
    description: 'Custom guild trail unlocked'
  },
  level18: {
    perk: '+5 Member Capacity',
    description: 'Guild cap increases to 50'
  },
  level20: {
    perk: 'Legendary Guild Frame',
    description: 'Exclusive animated profile frame'
  }
};
```

### Perk Stacking

```typescript
const PERK_STACKING = {
  xpBonus: {
    level3: '+5%',
    level9: '+5% (total 10%)',
    maximum: '10%'
  },
  goldBonus: {
    level6: '+5%',
    level14: '+5% (total 10%)',
    maximum: '10%'
  },
  memberCap: {
    base: 30,
    level5: '+5 (35)',
    level10: '+10 (45)',
    level18: '+5 (50)',
    maximum: 50
  }
};
```

---

## Guild Challenges

### Weekly Challenges

```typescript
const WEEKLY_CHALLENGES = {
  refresh: 'Monday 00:00 UTC',
  count: 5,

  examples: [
    {
      name: 'United Front',
      requirement: 'Guild members win 50 matches',
      reward: { guildXp: 200, memberGold: 100 }
    },
    {
      name: 'Elimination Squad',
      requirement: 'Guild members get 500 eliminations',
      reward: { guildXp: 150, memberGold: 75 }
    },
    {
      name: 'Treasure Fleet',
      requirement: 'Guild members open 200 chests',
      reward: { guildXp: 100, memberGold: 50 }
    },
    {
      name: 'Active Crew',
      requirement: '20 members play at least 5 matches',
      reward: { guildXp: 250, memberGold: 125 }
    },
    {
      name: 'Party Pirates',
      requirement: 'Guild members play 30 party matches',
      reward: { guildXp: 175, memberGold: 100 }
    }
  ],

  tracking: {
    display: 'Guild menu shows challenge progress',
    contribution: 'Individual contribution visible',
    notification: 'Alert when challenge completed'
  }
};
```

### Daily Challenges

```typescript
const DAILY_CHALLENGES = {
  refresh: 'Daily at 00:00 UTC',
  count: 3,

  examples: [
    {
      name: 'Daily Duty',
      requirement: 'Guild members play 20 matches',
      reward: { guildXp: 50 }
    },
    {
      name: 'Victory Run',
      requirement: 'Guild members win 10 matches',
      reward: { guildXp: 75 }
    },
    {
      name: 'Team Spirit',
      requirement: '10 members log in',
      reward: { guildXp: 40 }
    }
  ]
};
```

---

## Guild Leaderboards

### Leaderboard Types

```typescript
const GUILD_LEADERBOARDS = {
  overall: {
    name: 'Top Guilds',
    metric: 'Guild XP (all time)',
    display: 'Top 1000 guilds',
    reset: 'Never (cumulative)'
  },

  seasonal: {
    name: 'Season Champions',
    metric: 'Guild XP earned this season',
    display: 'Top 500 guilds',
    reset: 'Each ranked season',
    rewards: 'Seasonal guild rewards'
  },

  weekly: {
    name: 'Weekly Warriors',
    metric: 'Weekly challenge completions',
    display: 'Top 100 guilds',
    reset: 'Weekly',
    rewards: 'Bonus guild XP'
  },

  regional: {
    name: 'Regional Rulers',
    metric: 'Guild XP by region',
    display: 'Top 100 per region',
    reset: 'Monthly'
  }
};
```

### Seasonal Rewards

```typescript
const SEASONAL_GUILD_REWARDS = {
  top10: {
    guildReward: 'Exclusive seasonal guild banner',
    memberReward: 'Legendary chest + 500 Doubloons each'
  },
  top50: {
    guildReward: 'Seasonal guild badge',
    memberReward: 'Epic chest + 250 Doubloons each'
  },
  top100: {
    guildReward: 'Seasonal guild spray',
    memberReward: 'Rare chest + 100 Doubloons each'
  },
  top500: {
    guildReward: 'Recognition in Hall of Fame',
    memberReward: '50 Doubloons each'
  }
};
```

---

## Guild vs Guild (Future)

### GvG Events

```typescript
const GVG_EVENTS = {
  status: 'Future implementation',

  concept: {
    format: 'Guild wars with matched opponents',
    duration: '1 week events',
    scoring: 'Points for member performance',
    matchmaking: 'Based on guild level and member count'
  },

  modes: {
    warWeek: {
      description: 'Two guilds compete over a week',
      scoring: 'Member wins contribute points',
      rewards: 'Winning guild gets bonus rewards'
    },
    tournament: {
      description: 'Bracket-style guild tournament',
      participants: '32-64 guilds',
      rewards: 'Escalating rewards per round'
    }
  }
};
```

---

## Guild Management

### Leader Tools

```typescript
const LEADER_TOOLS = {
  roster: {
    view: 'All members with stats',
    sort: 'By rank, activity, contribution',
    filter: 'By online status, rank',
    actions: 'Promote, demote, kick, ban'
  },

  settings: {
    basic: 'Name, tag, description, join settings',
    permissions: 'Rank permission configuration',
    customRanks: 'Create and configure custom ranks'
  },

  activity: {
    dashboard: 'Guild activity overview',
    metrics: ['Active members', 'Weekly XP', 'Challenge progress'],
    history: 'Activity log (joins, leaves, promotions)'
  },

  moderation: {
    chatLogs: 'View recent chat history',
    reports: 'Member report queue',
    actions: 'Mute, kick, ban with notes'
  }
};
```

### Inactivity Management

```typescript
const INACTIVITY_MANAGEMENT = {
  memberInactivity: {
    warning: '14 days inactive: flagged in roster',
    autoKick: 'Optional: auto-kick after 30 days',
    configurable: 'Leader sets inactivity threshold'
  },

  leaderInactivity: {
    warning: '14 days: notification to officers',
    transfer: '30 days: auto-transfer to most active officer',
    noOfficers: 'Transfer to most senior active member'
  },

  guildInactivity: {
    dormant: '60 days no activity: marked dormant',
    disband: '90 days no activity: auto-disbanded',
    protection: 'Active members prevent dormancy'
  }
};
```

---

## Technical Implementation

### Data Model

```typescript
const DATA_MODEL = {
  guild: {
    id: 'string (UUID)',
    name: 'string',
    tag: 'string',
    description: 'string',
    banner: 'BannerConfig',
    level: 'number',
    currentXp: 'number',
    totalXp: 'number',
    memberCount: 'number',
    memberCap: 'number',
    joinType: 'enum',
    minLevel: 'number',
    language: 'string',
    createdAt: 'timestamp',
    leaderId: 'string'
  },

  guildMember: {
    guildId: 'string',
    playerId: 'string',
    rank: 'enum',
    customRank: 'string (nullable)',
    joinedAt: 'timestamp',
    contributionXp: 'number',
    weeklyContribution: 'number',
    lastActiveAt: 'timestamp'
  },

  guildChallenge: {
    guildId: 'string',
    challengeId: 'string',
    progress: 'number',
    target: 'number',
    completed: 'boolean',
    completedAt: 'timestamp (nullable)',
    contributions: 'map<playerId, amount>'
  },

  guildInvite: {
    id: 'string',
    guildId: 'string',
    inviterId: 'string',
    inviteeId: 'string',
    status: 'enum (pending, accepted, declined, expired)',
    createdAt: 'timestamp',
    expiresAt: 'timestamp'
  },

  guildJoinRequest: {
    id: 'string',
    guildId: 'string',
    playerId: 'string',
    message: 'string',
    status: 'enum (pending, accepted, declined)',
    createdAt: 'timestamp',
    reviewedBy: 'string (nullable)',
    reviewedAt: 'timestamp (nullable)'
  }
};
```

### API Endpoints

```typescript
const API_ENDPOINTS = {
  guilds: {
    create: 'POST /api/v1/guilds',
    get: 'GET /api/v1/guilds/{id}',
    update: 'PUT /api/v1/guilds/{id}',
    delete: 'DELETE /api/v1/guilds/{id}',
    search: 'GET /api/v1/guilds/search',
    leaderboard: 'GET /api/v1/guilds/leaderboard'
  },
  members: {
    list: 'GET /api/v1/guilds/{id}/members',
    join: 'POST /api/v1/guilds/{id}/join',
    leave: 'POST /api/v1/guilds/{id}/leave',
    kick: 'DELETE /api/v1/guilds/{id}/members/{playerId}',
    promote: 'PUT /api/v1/guilds/{id}/members/{playerId}/rank'
  },
  invites: {
    send: 'POST /api/v1/guilds/{id}/invites',
    respond: 'PUT /api/v1/guilds/invites/{inviteId}'
  },
  challenges: {
    list: 'GET /api/v1/guilds/{id}/challenges',
    progress: 'GET /api/v1/guilds/{id}/challenges/{challengeId}'
  },
  chat: {
    history: 'GET /api/v1/guilds/{id}/chat/{channel}',
    send: 'POST /api/v1/guilds/{id}/chat/{channel}'
  }
};
```

---

## Testing Requirements

```typescript
const TESTING_REQUIREMENTS = {
  functional: [
    'Guild creation with all settings',
    'Join/leave/kick/ban flows',
    'Rank promotions and demotions',
    'Chat functionality across channels',
    'Challenge tracking and completion',
    'XP earning and level progression',
    'Perk activation at correct levels'
  ],

  scale: [
    'Guild with 50 members',
    'Chat with high message volume',
    'Leaderboard with 10,000+ guilds',
    'Challenge tracking across large guild'
  ],

  edge: [
    'Last member leaves guild',
    'Leader goes inactive',
    'Member banned rejoins after unban',
    'Guild reaches max level',
    'Concurrent rank changes'
  ],

  moderation: [
    'Profanity filter in chat',
    'Report system functions',
    'Mute/ban enforcement'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-09 | Development Team | Initial guilds/crews system specification |
