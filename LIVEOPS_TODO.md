# Live Operations TODO - Retention & Events

This document covers all live operations systems including quests, events, seasons, player retention, and community features.

---

## Table of Contents
1. [Quest System](#1-quest-system)
2. [Events System](#2-events-system)
3. [Retention Mechanics](#3-retention-mechanics)
4. [Progression Systems](#4-progression-systems)
5. [Social and Community](#5-social-and-community)
6. [Communication and Engagement](#6-communication-and-engagement)

---

## 1. Quest System

### LIVE-001: Daily Quest System
**Priority:** P1 | **Complexity:** M | **Status: Completed**

**Description:**
Implement rotating daily quests that give players daily engagement goals.

**Quest Examples:**
- "Play 2 matches" - 100 Gold
- "Open 5 treasure chests" - 100 Gold
- "Get 3 eliminations" - 150 Gold
- "Deal 1000 damage" - 100 Gold
- "Use 5 utility abilities" - 100 Gold
- "Survive to top 10 twice" - 200 Gold

**Acceptance Criteria:**
- [x] 3 daily quests assigned per day
- [x] Quests reset at fixed time (UTC midnight or player local)
- [x] Progress tracked across matches
- [x] Completion grants rewards immediately
- [x] Quest UI shows progress (3/5 chests opened)
- [x] Quest difficulty scales appropriately
- [x] Quests completable by average player in 30-60 min
- [x] Re-roll option for 1 quest per day (optional)

**Definition of Done:**
- Daily quest system functional
- Players engage with quests regularly
- Completion rate 60%+ target
- Tested for edge cases

**Deliverables:**
- [Daily Quest System Specification](docs/DAILY_QUEST_SYSTEM_SPEC.md)

**Dependencies:** BACK-011, UX-014

---

### LIVE-002: Weekly Quest System
**Priority:** P1 | **Complexity:** M | **Status: Completed**

**Description:**
Implement weekly quests with larger objectives and rewards.

**Quest Examples:**
- "Win 3 matches this week" - 500 Gold + BP XP
- "Get 25 eliminations" - 500 Gold
- "Open 30 treasure chests" - 400 Gold
- "Play 20 matches" - 600 Gold + Rare Chest

**Acceptance Criteria:**
- [x] 5-7 weekly quests assigned
- [x] Quests reset weekly (e.g., Monday UTC)
- [x] Larger rewards than dailies
- [x] Progress persists through week
- [x] Weekly summary at reset
- [x] Quests contribute to Battle Pass significantly

**Definition of Done:**
- Weekly quests functional
- Encourages regular weekly play
- Balanced with daily quests

**Deliverables:**
- [Weekly Quest System Specification](docs/WEEKLY_QUEST_SYSTEM_SPEC.md)

**Dependencies:** LIVE-001

---

### LIVE-003: Achievement System
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Implement long-term achievement goals for players to work toward.

**Achievement Categories:**
- **Combat:** Kill milestones, win milestones
- **Abilities:** Use each ability type X times
- **Exploration:** Visit all POIs, open chest types
- **Social:** Play with friends, duo wins
- **Collection:** Own X cosmetics
- **Mastery:** Reach ranks, complete passes

**Acceptance Criteria:**
- [x] Achievement tracking across all sessions
- [x] Tiered achievements (Bronze/Silver/Gold/Platinum)
- [x] Rewards per achievement (Gold, Doubloons, cosmetics)
- [x] Achievement gallery UI
- [x] Pop-up notification on unlock
- [x] Profile displays achievement count/score
- [x] Hidden achievements for discovery
- [x] At least 50 achievements at launch

**Definition of Done:**
- Achievement system functional
- Players have long-term goals
- Achievements feel rewarding

**Deliverables:**
- [Achievement System Specification](docs/ACHIEVEMENT_SYSTEM_SPEC.md)

**Dependencies:** BACK-011

---

### LIVE-004: Quest Admin Tools
**Priority:** P1 | **Complexity:** M | **Status: Completed**

**Description:**
Build admin tools for creating and managing quests.

**Tool Features:**
- Create new quest templates
- Set quest pools for daily/weekly
- Configure rewards
- Preview quest rotations
- Emergency disable quest
- View quest completion analytics

**Acceptance Criteria:**
- [x] Web-based admin interface
- [x] Quest CRUD operations
- [x] Reward configuration
- [x] Pool assignment
- [x] Live quest status monitoring
- [x] A/B test different quests

**Definition of Done:**
- LiveOps can manage quests without code deploy
- Analytics visible
- Tested thoroughly

**Deliverables:**
- [Quest Admin Tools Specification](docs/QUEST_ADMIN_TOOLS_SPEC.md)

**Dependencies:** LIVE-001, LIVE-002, BACK-022

---

## 2. Events System

### LIVE-005: Limited-Time Event Framework
**Priority:** P1 | **Complexity:** L | **Status: Completed**

**Description:**
Build framework for running limited-time events with special content.

**Event Components:**
- Event start/end dates
- Event-specific quests
- Event rewards track (mini battle pass)
- Event cosmetics in shop
- Event UI theming
- Event game mode (optional)

**Acceptance Criteria:**
- [x] Event can be scheduled in advance
- [x] Event appears in-game during active period
- [x] Event quests appear alongside regular quests
- [x] Event progress tracked
- [x] Event rewards claimable
- [x] Event content removed after end
- [x] Players notified of event start/end
- [x] Multiple concurrent events possible

**Definition of Done:**
- Event framework tested with mock event
- Can deploy new event via config
- Analytics for event engagement

**Deliverables:**
- [Limited-Time Event Framework Specification](docs/LIMITED_TIME_EVENT_FRAMEWORK_SPEC.md)

**Dependencies:** LIVE-001, MON-004

---

### LIVE-006: Seasonal Events
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Plan and implement major seasonal events tied to real-world holidays or game themes.

**Seasonal Event Ideas:**
- **Halloween:** "Cursed Seas" - Ghost pirates, spooky map variant
- **Winter:** "Frostbitten Cove" - Snow map, holiday cosmetics
- **Summer:** "Treasure Hunt" - Special treasure collection
- **Anniversary:** "Pirate's Jubilee" - Celebration, bonus rewards

**Per Event:**
- 2-week duration
- Themed cosmetics (10+ items)
- Special game mode or map variant
- Event pass (mini battle pass)
- Community challenges

**Acceptance Criteria:**
- [x] Each major event has unique theme
- [x] Event cosmetics are exclusive (or rare return)
- [x] Event mode adds variety
- [x] Significant marketing push around events
- [x] Post-event analysis for learning

**Definition of Done:**
- First seasonal event successfully executed
- Positive player reception
- Revenue and engagement lift measured

**Deliverables:**
- [Seasonal Events Specification](docs/SEASONAL_EVENTS_SPEC.md)

**Dependencies:** LIVE-005

---

### LIVE-007: Community Challenges
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Implement server-wide challenges where all players contribute to a goal.

**Challenge Examples:**
- "Community: Defeat 1,000,000 monsters this week" - All players get reward
- "Race to 100,000 wins" - Milestone rewards unlocked

**Acceptance Criteria:**
- [x] Global progress tracked
- [x] Progress visible in-game (counter)
- [x] Milestones with incremental rewards
- [x] All active players receive rewards
- [x] Challenge runs for set duration
- [x] Post-challenge summary

**Definition of Done:**
- Community challenge functional
- Drives engagement and social sharing
- Tested at scale

**Deliverables:**
- [Community Challenges Specification](docs/COMMUNITY_CHALLENGES_SPEC.md)

**Dependencies:** BACK-019

---

### LIVE-008: Limited-Time Modes (LTM)
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Implement rotating limited-time game modes for variety.

**LTM Ideas:**
- **Swords Only:** No ranged abilities
- **Chaos Mode:** All abilities epic rank
- **Duo Royale:** Duos only, faster circle
- **Boss Rush:** PvE focused, defeat waves
- **Ability Draft:** Random abilities assigned
- **Mini Royale:** Smaller map, 20 players

**Acceptance Criteria:**
- [x] LTM selectable when active
- [x] LTM rules enforced (modified from base)
- [x] LTM rotation schedule
- [x] LTM-specific rewards (optional)
- [x] Announce LTM start
- [x] Can run alongside standard modes

**Definition of Done:**
- At least 2 LTMs ready for rotation
- Provides gameplay variety
- Positive player feedback

**Deliverables:**
- [Limited-Time Modes Specification](docs/LIMITED_TIME_MODES_SPEC.md)

**Dependencies:** GAME-020

---

## 3. Retention Mechanics

### LIVE-009: Daily Login Rewards
**Priority:** P1 | **Complexity:** S | **Status: Completed**

**Description:**
Reward players for daily logins to build habit.

**Login Reward Calendar (7-day cycle):**
- Day 1: 50 Gold
- Day 2: 100 Gold
- Day 3: 5 Doubloons
- Day 4: 150 Gold
- Day 5: Common Chest
- Day 6: 200 Gold
- Day 7: Rare Chest + 10 Doubloons

**Acceptance Criteria:**
- [x] Daily login tracked
- [x] Rewards shown in calendar UI
- [x] Claim button for each day
- [x] Streak tracker (days consecutive)
- [x] Streak breaks reset to day 1 (or partial reset)
- [x] Returning player catch-up (optional)
- [x] Monthly reset with escalating rewards

**Definition of Done:**
- Login rewards functional
- Players motivated to log in daily
- Tested for timezone edge cases

**Deliverables:**
- [Daily Login Rewards Specification](docs/DAILY_LOGIN_REWARDS_SPEC.md)

**Dependencies:** BACK-011

---

### LIVE-010: First Win of the Day
**Priority:** P1 | **Complexity:** S | **Status: Completed**

**Description:**
Bonus reward for first match win each day.

**Acceptance Criteria:**
- [x] Track first win per calendar day
- [x] Bonus reward on first win (200 Gold + BP XP)
- [x] Visual indicator when bonus available
- [x] Celebration on earning bonus
- [x] Reset at daily reset time

**Definition of Done:**
- First win bonus functional
- Encourages daily play

**Deliverables:**
- [First Win of the Day Specification](docs/FIRST_WIN_OF_THE_DAY_SPEC.md)

**Dependencies:** GAME-005, BACK-011

---

### LIVE-011: Come Back Rewards
**Priority:** P2 | **Complexity:** S | **Status: Completed**

**Description:**
Reward players who return after absence to re-engage churned users.

**Come Back Mechanics:**
- Detect player absent for 7+ days
- Show "Welcome Back!" popup
- Grant catch-up rewards (Gold, free chest)
- Highlight missed content (new items, events)
- Easy re-onboarding

**Acceptance Criteria:**
- [x] Detect returning player (no login for X days)
- [x] Special welcome popup
- [x] Catch-up rewards granted
- [x] Summary of what's new
- [x] Smooth re-entry experience
- [x] Track come-back conversion

**Definition of Done:**
- Come back flow functional
- Measurable re-engagement lift
- Not exploitable

**Deliverables:**
- [Come Back Rewards Specification](docs/COME_BACK_REWARDS_SPEC.md)

**Dependencies:** BACK-019

---

### LIVE-012: Push Notifications
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Implement push notifications to bring players back.

**Notification Types:**
- Daily quest available
- Friends online
- Event starting
- Battle Pass expiring
- Personalized (you're close to next tier!)
- Lapsed player win-back

**Acceptance Criteria:**
- [x] Push notification SDK integrated (Firebase, OneSignal)
- [x] Player opt-in/opt-out controls
- [x] Notification categories toggleable
- [x] Scheduled notifications
- [x] Personalization tokens
- [x] A/B test notification copy
- [x] Analytics on open rates

**Definition of Done:**
- Notifications functional on iOS and Android
- Positive engagement impact
- Opt-out respected

**Deliverables:**
- [Push Notifications Specification](docs/PUSH_NOTIFICATIONS_SPEC.md)

**Dependencies:** BACK-010

---

## 4. Progression Systems

### LIVE-013: Player Level System
**Priority:** P1 | **Complexity:** M | **Status: Completed**

**Description:**
Implement account-level progression that persists across matches.

**Level System:**
- Start at level 1
- Earn account XP from matches and quests
- Level up to earn rewards (Gold, chests, cosmetics)
- No level cap (or very high cap)

**XP Per Level:**
- Levels 1-10: 1000 XP each
- Levels 11-30: 2000 XP each
- Levels 31-50: 3000 XP each
- Levels 51+: 5000 XP each

**Acceptance Criteria:**
- [x] Account level displayed in profile
- [x] XP bar shows progress
- [x] Level up rewards granted
- [x] Level displayed on player nameplate (optional)
- [x] Milestones at key levels (10, 25, 50, 100)
- [x] Level gates for features (ranked unlocks at 10)

**Definition of Done:**
- Level system functional
- Players feel progression
- Rewards compelling

**Deliverables:**
- [Player Level System Specification](docs/PLAYER_LEVEL_SYSTEM_SPEC.md)

**Dependencies:** BACK-011

---

### LIVE-014: Player Titles and Badges
**Priority:** P2 | **Complexity:** S

**Description:**
Award titles and badges for accomplishments.

**Examples:**
- "Kraken Slayer" - Defeat kraken boss 10 times
- "Treasure Hunter" - Open 500 chests
- "Champion" - Reach Pirate King rank
- Season badges for ranks achieved

**Acceptance Criteria:**
- [ ] Titles/badges unlockable via achievements
- [ ] Display selected title under name
- [ ] Badge showcase on profile
- [ ] Limited/rare badges for prestige
- [ ] Seasonal badges (non-tradeable)

**Definition of Done:**
- Title/badge system functional
- Prestige value created
- Players display proudly

**Dependencies:** LIVE-003

---

### LIVE-015: Ranked Season Structure
**Priority:** P2 | **Complexity:** M

**Description:**
Define ranked season structure with rewards.

**Season Structure:**
- Duration: 8 weeks aligned with Battle Pass
- Soft reset each season (drop 2-3 ranks)
- Placement matches at season start
- End-of-season rewards based on peak rank

**Season Rewards:**
- Exclusive rank cosmetic (outfit piece)
- Rank badge/title
- Currency based on rank
- Top 100 special recognition

**Acceptance Criteria:**
- [ ] Season dates defined and enforced
- [ ] Soft reset algorithm
- [ ] End-of-season reward distribution
- [ ] Season history viewable
- [ ] Clear communication of season schedule

**Definition of Done:**
- Ranked seasons functional
- Competitive players engaged
- Rewards desirable

**Dependencies:** GAME-023

---

### LIVE-016: Mastery System (Post-Launch)
**Priority:** P3 | **Complexity:** M

**Description:**
Implement ability or playstyle mastery for depth.

**Mastery Ideas:**
- Track usage of each ability
- Mastery levels per ability (1-10)
- Mastery rewards (skins for that ability)
- Visible mastery to other players

**Acceptance Criteria:**
- [ ] Usage tracked per ability
- [ ] Mastery XP earned
- [ ] Mastery levels unlocked
- [ ] Mastery-specific cosmetic rewards
- [ ] Displayed in profile and loadout

**Definition of Done:**
- Mastery adds long-term goals
- Diverse playstyle encouraged

**Dependencies:** GAME-008

---

## 5. Social and Community

### LIVE-017: Guilds/Crews System
**Priority:** P3 | **Complexity:** L

**Description:**
Implement guild system for player communities.

**Guild Features:**
- Create guild (name, tag, banner)
- Invite members
- Guild chat
- Guild perks (XP bonus)
- Guild leaderboard
- Guild quests/challenges

**Acceptance Criteria:**
- [ ] Create/join/leave guild
- [ ] Guild roster management
- [ ] Guild ranks (leader, officer, member)
- [ ] Guild chat channel
- [ ] Guild XP and leveling
- [ ] Guild perks at levels
- [ ] Weekly guild challenges
- [ ] Guild vs guild events (future)

**Definition of Done:**
- Guild system functional
- Social bonds strengthen retention
- Active guilds formed

**Dependencies:** BACK-013

---

### LIVE-018: Referral Program
**Priority:** P3 | **Complexity:** M

**Description:**
Reward players for inviting friends.

**Referral Mechanics:**
- Unique referral code/link per player
- New player uses code on signup
- Both referrer and referee get rewards
- Milestone rewards for multiple referrals

**Acceptance Criteria:**
- [ ] Generate unique referral code
- [ ] Share link functionality
- [ ] Track referral signup
- [ ] Reward delivery on referee qualifying action (level 5, first match)
- [ ] Referral dashboard
- [ ] Fraud prevention

**Definition of Done:**
- Referral program drives organic growth
- Rewards balanced
- No exploitation

**Dependencies:** BACK-010

---

### LIVE-019: Social Sharing
**Priority:** P2 | **Complexity:** S

**Description:**
Enable players to share achievements and moments.

**Sharing Features:**
- Share match results to social media
- Share achievement unlocks
- Screenshot capture
- Replay clips (future)

**Acceptance Criteria:**
- [ ] Share button on results screen
- [ ] Native share sheet (iOS/Android)
- [ ] Branded share image generated
- [ ] Link back to game
- [ ] Privacy controls

**Definition of Done:**
- Social sharing functional
- Organic visibility increased

**Dependencies:** UX-013

---

## 6. Communication and Engagement

### LIVE-020: In-Game News/Announcements
**Priority:** P1 | **Complexity:** S | **Status: Completed**

**Description:**
Communicate with players through in-game messaging.

**Announcement Types:**
- News banner (main menu)
- Event announcements
- Patch notes
- Server maintenance alerts
- Community spotlights

**Acceptance Criteria:**
- [x] News section in main menu
- [x] Banner for urgent announcements
- [x] Rich text/images support
- [x] Read/unread tracking
- [x] Remote update (no app deploy needed)
- [x] Localization support

**Definition of Done:**
- Announcements reach players
- CMS or admin tool for updates
- Tested for engagement

**Deliverables:**
- [In-Game News/Announcements Specification](docs/IN_GAME_NEWS_ANNOUNCEMENTS_SPEC.md)

**Dependencies:** None

---

### LIVE-021: Customer Support Integration
**Priority:** P1 | **Complexity:** M | **Status: Completed**

**Description:**
Provide player support channels.

**Support Features:**
- In-game support ticket submission
- FAQ/Help center link
- Report player flow
- Bug report with auto-capture
- Purchase issue handling

**Acceptance Criteria:**
- [x] Support accessible from settings
- [x] Ticket form with categories
- [x] Device/account info auto-attached
- [x] Response via email or in-game
- [x] FAQ for common issues
- [x] Purchase history accessible
- [x] Integration with support tool (Zendesk, etc.)

**Definition of Done:**
- Players can get help
- Support team equipped
- Response time targets set

**Deliverables:**
- [Customer Support Integration Specification](docs/CUSTOMER_SUPPORT_INTEGRATION_SPEC.md)

**Dependencies:** BACK-011

---

### LIVE-022: Survey and Feedback System
**Priority:** P2 | **Complexity:** S

**Description:**
Collect player feedback systematically.

**Feedback Mechanisms:**
- In-game survey prompts (NPS, satisfaction)
- Feature feedback forms
- Post-match rating (optional)
- Beta feedback channel

**Acceptance Criteria:**
- [ ] Survey can be triggered at key moments
- [ ] Responses collected and analyzed
- [ ] Not intrusive (dismiss option)
- [ ] Limited frequency per player
- [ ] Results drive improvements

**Definition of Done:**
- Feedback system functional
- Actionable insights gathered
- Player voice heard

**Dependencies:** BACK-019

---

### LIVE-023: Content Update Pipeline
**Priority:** P1 | **Complexity:** L | **Status: Completed**

**Description:**
Establish process for regular content updates.

**Content Cadence:**
- Weekly: Store rotation, LTM rotation
- Bi-weekly: Minor content (quests, tweaks)
- Monthly: New cosmetics, balance patch
- Quarterly: Major update (new ability, map change)
- Seasonal: Battle Pass, event, potentially new feature

**Pipeline Components:**
- Content calendar planning
- Asset production schedule
- QA and approval process
- Deployment schedule
- Rollback plan

**Acceptance Criteria:**
- [x] Content calendar created for 3 months ahead
- [x] Production tracked against schedule
- [x] QA checkpoints defined
- [x] Deployment automated where possible
- [x] Hotfix capability for urgent issues

**Definition of Done:**
- First quarter content plan complete
- Cadence achievable
- Process documented

**Deliverables:**
- [Content Update Pipeline Specification](docs/CONTENT_UPDATE_PIPELINE_SPEC.md)

**Dependencies:** All content creation tasks

---

### LIVE-024: Analytics Dashboard
**Priority:** P1 | **Complexity:** M | **Status: Completed**

**Description:**
Build comprehensive analytics dashboard for live ops decisions.

**Metrics Tracked:**
- DAU/MAU/Retention
- Session metrics
- Quest completion rates
- Event participation
- Feature usage
- Revenue metrics
- Player segments

**Acceptance Criteria:**
- [x] Real-time or daily metrics dashboard
- [x] Key metrics at a glance
- [x] Drill-down capability
- [x] Cohort analysis
- [x] Funnel visualization
- [x] Export for deep analysis
- [x] Accessible to stakeholders

**Definition of Done:**
- Dashboard live and accurate
- Decisions data-driven
- Anomalies caught quickly

**Deliverables:**
- [Analytics Dashboard Specification](docs/ANALYTICS_DASHBOARD_SPEC.md)

**Dependencies:** BACK-019

---

## Task Summary

| Task ID | Title | Priority | Complexity | Status |
|---------|-------|----------|------------|--------|
| LIVE-001 | Daily Quest System | P1 | M | Completed |
| LIVE-002 | Weekly Quest System | P1 | M | Completed |
| LIVE-003 | Achievement System | P2 | M | Completed |
| LIVE-004 | Quest Admin Tools | P1 | M | Completed |
| LIVE-005 | Limited-Time Event Framework | P1 | L | Completed |
| LIVE-006 | Seasonal Events | P2 | M | Completed |
| LIVE-007 | Community Challenges | P2 | S | Completed |
| LIVE-008 | Limited-Time Modes (LTM) | P2 | M | Completed |
| LIVE-009 | Daily Login Rewards | P1 | S | Completed |
| LIVE-010 | First Win of the Day | P1 | S | Completed |
| LIVE-011 | Come Back Rewards | P2 | S | Completed |
| LIVE-012 | Push Notifications | P2 | M | Completed |
| LIVE-013 | Player Level System | P1 | M | Completed |
| LIVE-014 | Player Titles and Badges | P2 | S | Not Started |
| LIVE-015 | Ranked Season Structure | P2 | M | Not Started |
| LIVE-016 | Mastery System | P3 | M | Not Started |
| LIVE-017 | Guilds/Crews System | P3 | L | Not Started |
| LIVE-018 | Referral Program | P3 | M | Not Started |
| LIVE-019 | Social Sharing | P2 | S | Not Started |
| LIVE-020 | In-Game News/Announcements | P1 | S | Completed |
| LIVE-021 | Customer Support Integration | P1 | M | Completed |
| LIVE-022 | Survey and Feedback System | P2 | S | Not Started |
| LIVE-023 | Content Update Pipeline | P1 | L | Completed |
| LIVE-024 | Analytics Dashboard | P1 | M | Completed |
