# Closed Alpha Test Plan

## Document Information
- **Task ID:** PROJ-007
- **Priority:** P1
- **Complexity:** M
- **Last Updated:** 2025-12-06
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This document defines the complete plan for conducting a closed alpha test of Plunderstorm Mobile. The alpha test will gather external feedback from a controlled group of testers before progressing to beta, validating core gameplay, identifying critical issues, and refining the player experience.

### 1.2 Alpha Test Goals
1. **Validate Core Fun:** Confirm the battle royale gameplay loop is engaging
2. **Identify Critical Bugs:** Find and fix major issues before wider release
3. **Gather Player Feedback:** Understand player preferences and pain points
4. **Test Server Stability:** Verify servers handle concurrent players
5. **Capture Metrics:** Establish baseline retention and engagement data
6. **Build Community:** Create early advocates and gather testimonials

### 1.3 Success Criteria
- 50+ testers complete at least 10 matches
- Net Promoter Score (NPS) ≥ 30
- Crash rate < 5%
- Server uptime ≥ 95%
- D1 retention ≥ 25% (among active testers)
- Actionable feedback collected on all core systems

---

## 2. Alpha Tester Recruitment

### 2.1 Target Tester Profile

```typescript
const ALPHA_TESTER_PROFILE = {
  targetCount: {
    minimum: 50,
    ideal: 100,
    maximum: 200
  },

  demographics: {
    ageRange: '18-45',
    gamerType: ['mobile_gamers', 'battle_royale_fans', 'casual_competitive'],
    devices: {
      iOS: 0.5, // 50% iOS
      android: 0.5 // 50% Android
    },
    experience: {
      mobile_gaming: 'regular', // At least weekly mobile gaming
      battle_royale: 'familiar' // Played at least one BR game
    }
  },

  deviceRequirements: {
    iOS: {
      minimum: 'iPhone 8 / iOS 14+',
      recommended: 'iPhone 11+'
    },
    android: {
      minimum: '3GB RAM, Android 9+',
      recommended: '4GB RAM, Android 11+'
    }
  },

  commitmentLevel: {
    minimumSessions: 5,
    minimumMatches: 10,
    feedbackRequired: true,
    surveyCompletion: true
  }
};
```

### 2.2 Recruitment Channels

```typescript
const RECRUITMENT_CHANNELS = {
  internal: {
    friendsAndFamily: {
      count: 20,
      priority: 'high',
      nda: 'simplified'
    },
    companyNetwork: {
      count: 10,
      priority: 'high'
    }
  },

  external: {
    discordCommunity: {
      count: 50,
      method: 'application_form',
      requirements: ['active_member', 'device_check']
    },
    socialMedia: {
      count: 30,
      platforms: ['twitter', 'reddit', 'facebook_groups'],
      method: 'signup_form'
    },
    gamingForums: {
      count: 20,
      forums: ['toucharcade', 'pocketgamer'],
      method: 'direct_outreach'
    },
    betaTestingPlatforms: {
      count: 50,
      platforms: ['betafamily', 'betabound'],
      method: 'platform_recruitment'
    }
  },

  waitlist: {
    enabled: true,
    overRecruit: 1.5, // 50% extra to account for dropoff
    priorityFactors: ['device_variety', 'engagement_history']
  }
};
```

### 2.3 Application Process

```typescript
const APPLICATION_PROCESS = {
  applicationForm: {
    fields: [
      { name: 'email', required: true },
      { name: 'device_model', required: true },
      { name: 'os_version', required: true },
      { name: 'age_range', required: true },
      { name: 'gaming_frequency', required: true },
      { name: 'br_experience', required: true },
      { name: 'favorite_mobile_games', required: false },
      { name: 'why_interested', required: false },
      { name: 'availability', required: true },
      { name: 'discord_username', required: false }
    ]
  },

  selectionCriteria: {
    deviceCompatibility: 'required',
    gamingExperience: 'weighted',
    availability: 'weighted',
    enthusiasm: 'bonus'
  },

  selectionProcess: [
    'Collect applications (1-2 weeks)',
    'Filter by device requirements',
    'Balance iOS/Android split',
    'Ensure device variety',
    'Select for engagement potential',
    'Send invitations'
  ]
};
```

---

## 3. Legal and NDA

### 3.1 Non-Disclosure Agreement

```typescript
const NDA_REQUIREMENTS = {
  required: true,
  type: 'simplified_click_through',

  keyTerms: [
    'Confidentiality of gameplay and features',
    'No screenshots or recordings shared publicly',
    'No streaming or content creation',
    'Feedback ownership transfer to company',
    'Term: Until public announcement or 1 year'
  ],

  enforcement: {
    level: 'standard',
    watermarking: true, // Unique watermark per tester
    monitorSocial: true
  },

  exceptions: [
    'Private sharing with household members',
    'Bug reports through official channels',
    'Anonymous survey responses'
  ]
};
```

### 3.2 Terms of Service

```typescript
const ALPHA_TOS = {
  acceptance: 'required_before_play',

  keyPoints: [
    'Alpha software - bugs expected',
    'Progress may be wiped',
    'Features may change',
    'No guaranteed compensation',
    'Data collection disclosure',
    'Age requirement (18+)'
  ],

  dataCollection: {
    gameplay: 'device_info, play_sessions, actions',
    feedback: 'surveys, bug_reports',
    technical: 'crash_logs, performance_data',
    optional: 'screen_recordings'
  },

  privacyCompliance: {
    gdpr: true,
    ccpa: true,
    consentRequired: true
  }
};
```

---

## 4. Build Distribution

### 4.1 iOS Distribution (TestFlight)

```typescript
const IOS_DISTRIBUTION = {
  platform: 'TestFlight',

  setup: {
    appStoreConnect: 'Create app record',
    testflightBeta: 'Enable beta testing',
    testGroups: ['internal', 'external_alpha']
  },

  invitationProcess: [
    'Collect tester emails',
    'Add to TestFlight group',
    'Apple sends invitation email',
    'Tester downloads TestFlight app',
    'Tester accepts invitation',
    'Build available for download'
  ],

  limitations: {
    maxExternalTesters: 10000,
    buildExpiry: '90_days',
    reviewRequired: true // For external testing
  },

  buildManagement: {
    versionNaming: 'alpha_YYYYMMDD_build#',
    releaseNotes: 'Required per build',
    automaticUpdates: true
  }
};
```

### 4.2 Android Distribution (Firebase App Distribution)

```typescript
const ANDROID_DISTRIBUTION = {
  platform: 'Firebase App Distribution',

  setup: {
    firebaseProject: 'Create/link project',
    appDistribution: 'Enable service',
    testerGroups: ['alpha_testers']
  },

  invitationProcess: [
    'Collect tester emails',
    'Add to Firebase tester group',
    'Firebase sends invitation email',
    'Tester clicks link',
    'Tester enables unknown sources',
    'APK/AAB downloads and installs'
  ],

  alternatives: {
    googlePlayInternal: {
      use: 'if_preferred',
      setup: 'Internal testing track'
    }
  },

  buildManagement: {
    versionNaming: 'alpha_YYYYMMDD_build#',
    releaseNotes: 'Required per build',
    automaticNotifications: true
  }
};
```

### 4.3 Build Requirements

```typescript
const BUILD_REQUIREMENTS = {
  stability: {
    crashFree: '> 95%',
    noBlockingBugs: true,
    completeFeatures: 'core_gameplay'
  },

  content: {
    tutorial: 'functional',
    abilities: 'mvp_set_only',
    map: 'one_map_complete',
    cosmetics: 'placeholder_ok',
    monetization: 'disabled_or_test'
  },

  technical: {
    analytics: 'integrated',
    crashReporting: 'integrated',
    remoteConfig: 'ready',
    featureFlags: 'ready'
  },

  watermarking: {
    enabled: true,
    type: 'unique_player_id',
    location: 'corner_subtle',
    purpose: 'leak_tracking'
  }
};
```

---

## 5. Test Schedule

### 5.1 Alpha Timeline

```typescript
const ALPHA_TIMELINE = {
  totalDuration: '3-4 weeks',

  phases: {
    preparation: {
      duration: '1 week',
      activities: [
        'Finalize alpha build',
        'Complete recruitment',
        'Send NDAs',
        'Set up distribution',
        'Prepare feedback channels',
        'Brief team on support'
      ]
    },

    softOpen: {
      duration: '3-4 days',
      testerCount: 20,
      purpose: 'Validate distribution, catch critical issues',
      activities: [
        'Invite internal/F&F testers',
        'Monitor for blockers',
        'Quick fixes if needed',
        'Verify analytics working'
      ]
    },

    fullAlpha: {
      duration: '2 weeks',
      testerCount: '50-200',
      activities: [
        'Invite all selected testers',
        'Monitor server stability',
        'Collect feedback continuously',
        'Deploy hotfixes as needed',
        'Run scheduled test sessions'
      ]
    },

    wrapUp: {
      duration: '3-4 days',
      activities: [
        'Final survey',
        'Analyze all data',
        'Create alpha report',
        'Plan beta priorities',
        'Thank testers'
      ]
    }
  }
};
```

### 5.2 Scheduled Test Sessions

```typescript
const TEST_SESSIONS = {
  purpose: 'Ensure enough players online for full matches',

  schedule: {
    frequency: '3x per week',
    duration: '2 hours per session',
    times: [
      { day: 'Tuesday', time: '7PM EST', region: 'Americas' },
      { day: 'Thursday', time: '8PM CET', region: 'Europe' },
      { day: 'Saturday', time: '2PM EST', region: 'Global' }
    ]
  },

  sessionStructure: {
    warmup: '15 min - Free play, familiarization',
    mainPlay: '75 min - Scheduled matches',
    feedback: '30 min - Live feedback session'
  },

  incentives: {
    participation: 'Track attendance',
    rewards: 'Launch cosmetics for active testers',
    recognition: 'Credits in game'
  },

  staffing: {
    devPresence: true,
    communityManager: true,
    purpose: 'Answer questions, observe, collect feedback'
  }
};
```

---

## 6. Feedback Collection

### 6.1 Feedback Channels

```typescript
const FEEDBACK_CHANNELS = {
  discord: {
    purpose: 'Primary community hub',
    channels: [
      { name: '#alpha-general', purpose: 'General chat' },
      { name: '#bug-reports', purpose: 'Bug submission' },
      { name: '#feedback', purpose: 'Feature feedback' },
      { name: '#suggestions', purpose: 'Ideas' },
      { name: '#session-chat', purpose: 'Live session discussion' }
    ],
    moderation: 'Active, friendly, responsive',
    devPresence: 'Daily check-ins'
  },

  inGameReporting: {
    bugReportButton: true,
    feedbackButton: true,
    automaticCrashReport: true,
    screenshotCapture: 'optional'
  },

  surveys: {
    initial: {
      timing: 'After 3 matches',
      focus: 'First impressions'
    },
    weekly: {
      timing: 'End of each week',
      focus: 'Ongoing experience'
    },
    final: {
      timing: 'End of alpha',
      focus: 'Overall assessment, NPS'
    }
  },

  directInterview: {
    count: '10-20 testers',
    selection: 'Varied engagement levels',
    format: 'Video call, 30 min',
    incentive: 'Gift card or premium currency'
  }
};
```

### 6.2 Survey Templates

```typescript
const SURVEY_TEMPLATES = {
  initialSurvey: {
    title: 'First Impressions Survey',
    questions: [
      { type: 'rating', q: 'How would you rate your first few matches?', scale: '1-10' },
      { type: 'rating', q: 'How intuitive were the controls?', scale: '1-5' },
      { type: 'rating', q: 'Did you understand the objective quickly?', scale: '1-5' },
      { type: 'open', q: 'What was the most fun part?' },
      { type: 'open', q: 'What was confusing or frustrating?' },
      { type: 'multiple', q: 'Which abilities did you try?', options: 'ability_list' },
      { type: 'rating', q: 'Rate the visual style', scale: '1-5' },
      { type: 'open', q: 'Any bugs or issues encountered?' }
    ]
  },

  weeklySurvey: {
    title: 'Weekly Check-In',
    questions: [
      { type: 'rating', q: 'How much did you enjoy playing this week?', scale: '1-10' },
      { type: 'rating', q: 'How often did you want to play again?', scale: '1-5' },
      { type: 'open', q: 'Favorite moment this week?' },
      { type: 'open', q: 'Most frustrating moment?' },
      { type: 'rating', q: 'How balanced do abilities feel?', scale: '1-5' },
      { type: 'open', q: 'Any new bugs found?' },
      { type: 'open', q: 'Suggestions for improvement?' }
    ]
  },

  finalSurvey: {
    title: 'Alpha Wrap-Up Survey',
    questions: [
      { type: 'nps', q: 'How likely are you to recommend this game?', scale: '0-10' },
      { type: 'rating', q: 'Overall rating of the alpha experience', scale: '1-10' },
      { type: 'open', q: 'What should be the #1 priority before launch?' },
      { type: 'open', q: 'What feature would you most want added?' },
      { type: 'rating', q: 'How polished did the game feel?', scale: '1-5' },
      { type: 'multiple', q: 'Would you pay for cosmetics?', options: ['yes', 'maybe', 'no'] },
      { type: 'open', q: 'Final thoughts or suggestions?' },
      { type: 'consent', q: 'May we quote your feedback (anonymously)?' }
    ]
  }
};
```

### 6.3 Bug Reporting

```typescript
const BUG_REPORTING = {
  inGameSystem: {
    accessPoint: 'Settings menu > Report Bug',
    fields: [
      { name: 'category', type: 'dropdown', options: ['crash', 'gameplay', 'visual', 'audio', 'other'] },
      { name: 'description', type: 'text', required: true },
      { name: 'steps', type: 'text', required: false },
      { name: 'screenshot', type: 'attachment', required: false }
    ],
    automaticAttach: ['device_info', 'game_state', 'last_actions'],
    destination: 'bug_tracking_system'
  },

  discordReporting: {
    template: `
      **Bug Report**
      Device:
      OS Version:
      What happened:
      Steps to reproduce:
      Expected behavior:
      Screenshot/Video:
    `,
    tracking: 'Manual import to bug tracker'
  },

  triagePriority: {
    P0: 'Crash, data loss, unplayable',
    P1: 'Major gameplay impact',
    P2: 'Minor gameplay impact',
    P3: 'Visual/polish issues'
  },

  responseTime: {
    P0: '4 hours',
    P1: '24 hours',
    P2: '48 hours',
    P3: 'Next update'
  }
};
```

---

## 7. Metrics and Analytics

### 7.1 Key Metrics to Track

```typescript
const ALPHA_METRICS = {
  engagement: {
    dailyActiveTesters: 'DAT',
    sessionsPerDay: 'avg_sessions',
    sessionLength: 'avg_minutes',
    matchesPerSession: 'avg_matches',
    matchCompletionRate: 'completed/started'
  },

  retention: {
    d1: 'Return day after first play',
    d3: 'Return within 3 days',
    d7: 'Return within 7 days',
    weeklyActive: 'Played each week'
  },

  gameplay: {
    avgMatchDuration: 'minutes',
    avgPlacement: 'finish_position',
    avgKills: 'per_match',
    abilityUsage: 'per_ability',
    deathCauses: 'breakdown',
    hotDropLocations: 'heatmap'
  },

  technical: {
    crashRate: 'crashes/session',
    crashFreeUsers: 'percentage',
    avgFPS: 'during_gameplay',
    loadTime: 'app_start_to_menu',
    networkLatency: 'avg_ping'
  },

  satisfaction: {
    npsScore: 'survey_nps',
    surveyResponses: 'completion_rate',
    bugReports: 'total_submitted',
    feedbackVolume: 'messages_per_tester'
  }
};
```

### 7.2 Analytics Implementation

```typescript
const ANALYTICS_SETUP = {
  platform: {
    primary: 'Firebase Analytics / Amplitude',
    crashReporting: 'Firebase Crashlytics',
    customEvents: 'Mixpanel / GameAnalytics'
  },

  keyEvents: [
    'alpha_signup',
    'first_launch',
    'tutorial_start',
    'tutorial_complete',
    'match_start',
    'match_end',
    'ability_used',
    'kill',
    'death',
    'placement',
    'session_end',
    'bug_report_submitted',
    'survey_completed'
  ],

  userProperties: [
    'tester_id',
    'device_model',
    'os_version',
    'cohort_date',
    'total_matches',
    'total_playtime'
  ],

  dashboards: {
    realtime: 'Active users, current matches',
    daily: 'DAT, sessions, crashes',
    weekly: 'Retention, engagement trends',
    feedback: 'Survey results, NPS'
  }
};
```

---

## 8. Support and Communication

### 8.1 Tester Communication

```typescript
const COMMUNICATION_PLAN = {
  channels: {
    email: 'Official announcements, surveys',
    discord: 'Community, live support',
    inApp: 'Update notices, session reminders'
  },

  cadence: {
    welcome: 'Upon acceptance',
    buildUpdates: 'Each new build',
    sessionReminders: '24h and 1h before',
    weeklySummary: 'End of each week',
    hotfixNotes: 'As needed',
    finalThank: 'End of alpha'
  },

  contentTypes: {
    releaseNotes: 'What changed, known issues',
    sessionSchedule: 'When to play',
    feedbackRequests: 'Specific questions',
    progressUpdates: 'How feedback is used',
    recognition: 'Top contributors'
  }
};
```

### 8.2 Support Structure

```typescript
const SUPPORT_STRUCTURE = {
  team: {
    communityManager: {
      role: 'Primary tester contact',
      responsibilities: ['Discord moderation', 'Email responses', 'Session hosting']
    },
    developer: {
      role: 'Technical support',
      responsibilities: ['Bug investigation', 'Hotfix deployment']
    },
    designer: {
      role: 'Feedback analysis',
      responsibilities: ['Survey review', 'Priority recommendations']
    }
  },

  responseTargets: {
    discord: '4 hours during business hours',
    email: '24 hours',
    criticalIssues: '2 hours'
  },

  escalation: {
    criticalBug: 'Immediate dev notification',
    negativeFeedback: 'Review and respond',
    dropoff: 'Reach out to understand why'
  }
};
```

---

## 9. Issue Management

### 9.1 Bug Tracking

```typescript
const BUG_TRACKING = {
  tool: 'Jira / Linear / GitHub Issues',

  workflow: {
    submitted: 'New bug received',
    triaged: 'Priority assigned',
    inProgress: 'Developer working',
    resolved: 'Fix implemented',
    verified: 'Tested and confirmed',
    released: 'Deployed to testers'
  },

  fields: {
    required: ['title', 'description', 'priority', 'reporter'],
    optional: ['steps', 'expected', 'actual', 'device', 'screenshot']
  },

  alphaLabels: ['alpha-blocker', 'alpha-critical', 'alpha-feedback']
};
```

### 9.2 Hotfix Process

```typescript
const HOTFIX_PROCESS = {
  criteria: {
    deploy: 'P0 bugs, server issues, widespread impact',
    wait: 'P1/P2 bugs batch with next update'
  },

  process: [
    'Issue identified and confirmed',
    'Fix developed and tested',
    'Build submitted (iOS: expedited review request)',
    'Release notes prepared',
    'Testers notified',
    'Monitor for resolution'
  ],

  targetTurnaround: {
    P0: '24-48 hours',
    P1: 'Next scheduled update'
  }
};
```

---

## 10. Alpha Completion

### 10.1 Exit Criteria

```typescript
const ALPHA_EXIT_CRITERIA = {
  participation: {
    requirement: '50+ testers completed 10+ matches',
    tracking: 'Analytics dashboard'
  },

  stability: {
    crashRate: '< 5%',
    serverUptime: '> 95%',
    noBlockerBugs: true
  },

  feedback: {
    npsScore: '≥ 30',
    surveyCompletion: '> 50%',
    majorConcernsAddressed: true
  },

  readiness: {
    criticalBugsFixed: true,
    betaPrioritiesIdentified: true,
    teamAlignedOnDirection: true
  }
};
```

### 10.2 Alpha Report

```typescript
const ALPHA_REPORT = {
  sections: [
    {
      title: 'Executive Summary',
      content: ['Key findings', 'Success metrics', 'Recommendations']
    },
    {
      title: 'Participation Statistics',
      content: ['Tester count', 'Session data', 'Completion rates']
    },
    {
      title: 'Feedback Analysis',
      content: ['Survey results', 'NPS breakdown', 'Common themes']
    },
    {
      title: 'Technical Performance',
      content: ['Crash data', 'Performance metrics', 'Server stats']
    },
    {
      title: 'Gameplay Insights',
      content: ['Ability usage', 'Balance data', 'Progression feedback']
    },
    {
      title: 'Bug Summary',
      content: ['Bugs found', 'Resolution status', 'Remaining issues']
    },
    {
      title: 'Beta Recommendations',
      content: ['Priority fixes', 'Feature requests', 'Focus areas']
    }
  ],

  distribution: ['Leadership', 'Development team', 'Archive']
};
```

### 10.3 Tester Recognition

```typescript
const TESTER_RECOGNITION = {
  thankYou: {
    email: 'Personal thank you with summary',
    discord: 'Public recognition post'
  },

  rewards: {
    allParticipants: [
      'Exclusive alpha tester badge (in-game)',
      'Alpha tester title',
      'Unique cosmetic item'
    ],
    topContributors: [
      'Credits in game',
      'Early beta access',
      'Additional cosmetics'
    ]
  },

  futureCommunication: {
    betaInvite: 'Automatic for active alpha testers',
    newsletter: 'Opt-in for launch updates',
    community: 'Ongoing Discord access'
  }
};
```

---

## 11. Implementation Checklist

### 11.1 Pre-Alpha
- [ ] Alpha build ready and tested
- [ ] TestFlight/Firebase configured
- [ ] NDA and ToS prepared
- [ ] Discord server set up
- [ ] Recruitment campaign launched
- [ ] Testers selected and invited
- [ ] Analytics integrated
- [ ] Bug tracking ready
- [ ] Team briefed on support

### 11.2 During Alpha
- [ ] Testers onboarded
- [ ] Test sessions held
- [ ] Daily monitoring active
- [ ] Weekly surveys sent
- [ ] Bugs triaged and fixed
- [ ] Communication maintained
- [ ] Hotfixes deployed as needed

### 11.3 Post-Alpha
- [ ] Final survey sent
- [ ] Data analyzed
- [ ] Alpha report created
- [ ] Testers thanked
- [ ] Beta priorities defined
- [ ] Lessons learned documented

---

## Appendix A: Sample Communications

### Welcome Email Template

```
Subject: Welcome to the Plunderstorm Alpha!

Ahoy, {Name}!

You've been selected to join the Plunderstorm closed alpha test! We're thrilled to have you aboard as we shape this pirate battle royale adventure.

**Getting Started:**
1. Accept the TestFlight/Firebase invitation (check your email)
2. Download and install the alpha build
3. Join our Discord: [link]
4. Read the NDA and Terms (required to play)

**Alpha Schedule:**
- Test sessions: Tuesdays and Thursdays at 7PM EST, Saturdays at 2PM EST
- Alpha duration: [dates]

**Your Mission:**
- Play matches and have fun!
- Report any bugs you find
- Complete weekly surveys
- Share your feedback on Discord

Your input directly shapes the game. Thank you for being part of this journey!

Fair winds and following seas,
The Plunderstorm Team
```

### Session Reminder Template

```
Subject: Alpha Session Tonight - 7PM EST!

Ahoy, Alpha Testers!

Reminder: Tonight's scheduled play session starts at 7PM EST!

**Tonight's Focus:**
- Testing new ability balance changes
- Stress testing with full lobbies
- Collecting feedback on new map area

**How to Participate:**
1. Launch Plunderstorm at 7PM EST
2. Queue for matches - we'll have players ready!
3. Join Discord voice chat for live discussion

See you on the battlefield!
```

---

## Appendix B: Contact Information

| Role | Name | Contact |
|------|------|---------|
| Alpha Lead | [TBD] | [email] |
| Community Manager | [TBD] | Discord: [handle] |
| Technical Support | [TBD] | [email] |
| Emergency Contact | [TBD] | [phone] |
