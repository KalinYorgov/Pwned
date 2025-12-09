# Gameplay TODO - Core Game Mechanics

This document covers all core gameplay systems including battle royale mechanics, ability system, combat, and PvE elements.

---

## Table of Contents
1. [Battle Royale Core Loop](#1-battle-royale-core-loop)
2. [Ability System](#2-ability-system)
3. [Combat System](#3-combat-system)
4. [PvE Elements](#4-pve-elements)
5. [In-Match Progression](#5-in-match-progression)
6. [Game Modes](#6-game-modes)

---

## 1. Battle Royale Core Loop

### GAME-001: Match Initialization System
**Priority:** P0 | **Complexity:** L

**Description:**
Implement the core match initialization flow including player joining, countdown timer, and match start synchronization. The system must handle 30-60 players joining a match instance, synchronize game state, and transition all players from lobby to active gameplay simultaneously.

**Acceptance Criteria:**
- [x] Players can queue for a match and are placed in a lobby
- [x] Lobby displays player count and countdown timer
- [x] Match starts when minimum player threshold is reached (or timer expires with bot fill)
- [x] All players receive synchronized match start signal
- [x] Match state properly initializes (storm position, loot spawns, NPC spawns)
- [x] Players who disconnect during init can reconnect within grace period (30 seconds)

**Definition of Done:**
- Match initialization works reliably with 60 concurrent players
- Average initialization time < 5 seconds
- No desync issues between players at match start
- Unit tests cover all initialization edge cases
- Load tested with simulated 60 players

**Dependencies:** BACK-001 (Matchmaking), BACK-003 (Game Server)

**Deliverables:**
- [Match Initialization System Specification](docs/MATCH_INITIALIZATION_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-002: Player Drop-In System
**Priority:** P0 | **Complexity:** M

**Description:**
Create the drop-in mechanic where players select a landing zone and descend onto the map. Inspired by Plunderstorm's "bombing in" mechanic, players should be launched from a pirate ship circling the island and can control their descent trajectory.

**Acceptance Criteria:**
- [x] Pre-drop phase shows full map with selectable landing zones
- [x] Players can mark intended drop location (visible to teammates in duo/squad)
- [x] Drop animation shows player launching from cannon/ship
- [x] Player has glide control during descent (directional input affects trajectory)
- [x] Landing triggers brief animation and vulnerability period (0.5s)
- [x] Camera smoothly transitions from bird's eye to third-person on landing
- [x] Drop path is visible to other players (for strategic awareness)

**Definition of Done:**
- Drop feels responsive and satisfying on mobile touch controls
- Drop phase completes within 15-20 seconds maximum
- No collision issues during descent
- Network synchronization shows other players dropping accurately
- Tested on both high-end and low-end devices

**Dependencies:** GAME-001, UX-003 (Touch Controls)

**Deliverables:**
- [Player Drop-In System Specification](docs/PLAYER_DROP_IN_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-003: Storm/Safe Zone System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement the shrinking safe zone mechanic that forces players together over the match duration. The storm deals damage to players outside the safe zone and creates visual/audio warnings as it approaches.

**Acceptance Criteria:**
- [x] Safe zone starts covering 80% of map, shrinks in 4-5 phases
- [x] Each phase has: warning period, shrink period, stable period
- [x] Storm damage scales with phase (Phase 1: 2 DPS, Phase 5: 10 DPS)
- [x] Visual indicator shows current safe zone on minimap and main view
- [x] Storm wall is clearly visible (towering clouds/magical barrier)
- [x] Audio cue warns when storm is about to shrink
- [x] UI shows time until next shrink phase
- [x] Final zone converges to small area forcing final confrontation
- [x] Storm movement is smooth and predictable (no stuttering)

**Definition of Done:**
- Storm phases balance match length to 10-15 minutes
- All players see identical storm positions (no desync)
- Storm VFX performs well on low-end devices
- Damage application is server-authoritative
- Edge cases handled (player on zone boundary, zone crossing water/terrain)

**Dependencies:** GAME-001, ART-015 (Storm VFX)

**Deliverables:**
- [Storm/Safe Zone System Specification](docs/STORM_SAFE_ZONE_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-004: Loot Spawn System
**Priority:** P0 | **Complexity:** M

**Description:**
Create the system for spawning loot (abilities) across the map. Loot appears in treasure chests of varying rarity and as drops from defeated NPCs. The system must ensure balanced distribution and fair early-game opportunities.

**Acceptance Criteria:**
- [x] Predefined loot spawn points distributed across map
- [x] Loot spawns on match start (randomized within spawn point pools)
- [x] Treasure chests have rarity tiers: Common (60%), Uncommon (25%), Rare (12%), Epic (3%)
- [x] Chest rarity affects quality of contained abilities
- [x] Higher-tier areas (named locations) have better loot density
- [x] Chests have opening animation (0.5-1s) - interruptible by damage
- [x] Opened chests are visually distinct (open lid, empty glow)
- [x] Abilities auto-equip to empty slots or show prompt if slots full
- [x] Loot from NPCs drops on ground briefly before auto-pickup radius

**Definition of Done:**
- Loot distribution tested for balance (no single OP landing spot)
- Server authoritatively controls loot spawns (anti-cheat)
- Opening chest feels satisfying (VFX, SFX)
- Network sync ensures no double-looting race conditions
- Performance tested with 200+ active loot items on map

**Dependencies:** GAME-005 (Ability System), ART-010 (Treasure Chests)

**Deliverables:**
- [Loot Spawn System Specification](docs/LOOT_SPAWN_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-005: Player Elimination System
**Priority:** P0 | **Complexity:** S

**Description:**
Handle player death, elimination announcement, and match end conditions. When a player's health reaches zero, they are eliminated from the match with appropriate feedback.

**Acceptance Criteria:**
- [x] Player elimination triggers death animation
- [x] Eliminated player drops a "soul orb" or treasure pile (grants XP to killer)
- [x] Kill feed shows elimination (Killer eliminated Victim)
- [x] Eliminating player sees "+1 Kill" notification
- [x] Remaining player count updates globally
- [x] Eliminated player can choose: spectate or return to lobby
- [x] Last player/team standing wins the match
- [x] Victory screen shows stats (kills, damage, abilities used, placement)
- [x] Match end broadcasts winner to all spectators

**Definition of Done:**
- Elimination is server-authoritative (no fake deaths)
- Spectator mode works smoothly for eliminated players
- Stats accurately tracked and displayed
- No exploits possible (death dodging, zombie state)
- Tested with rapid successive eliminations

**Dependencies:** GAME-012 (Combat), BACK-005 (Stats Tracking)

**Deliverables:**
- [Player Elimination System Specification](docs/PLAYER_ELIMINATION_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-006: Spectator Mode
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Allow eliminated players to spectate remaining players in the match. In team modes, spectate teammates first; after team elimination, can spectate any player.

**Acceptance Criteria:**
- [x] Eliminated players automatically enter spectator mode
- [x] Free camera or follow-player camera options
- [x] Can cycle through alive players (in solos: random; in teams: teammates first)
- [x] Spectator UI shows observed player's health, abilities, kill count
- [x] Spectators cannot interact with game world
- [x] Chat between spectators is isolated (no info leak to alive players)
- [x] Option to leave spectate and return to lobby at any time
- [x] Spectator count shown to alive players (optional setting)

**Definition of Done:**
- No information leaking from spectator to alive teammates (anti-cheat)
- Camera transitions are smooth
- Network bandwidth for spectators is optimized
- Works correctly with 30+ spectators per match
- UI clearly indicates spectator mode

**Deliverables:**
- [Spectator Mode Specification](docs/SPECTATOR_MODE_SPEC.md)

**Dependencies:** GAME-005, UX-012 (Spectator UI)

---

## 2. Ability System

### GAME-007: Ability Slot System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement the ability loadout system where players have limited slots for collected abilities. Based on Plunderstorm design: 2 offensive slots + 2 utility slots, plus a basic attack always available.

**Acceptance Criteria:**
- [x] Players always have access to basic attack (no slot required)
- [x] 4 ability slots available: 2 offensive (red border), 2 utility (blue border)
- [x] Picking up ability auto-equips to matching empty slot
- [x] If slot type is full, player sees swap prompt (new ability vs current)
- [x] Swapping drops the old ability on ground (can be picked up)
- [x] Abilities show cooldown timers on UI
- [x] Each ability has unique icon, clearly visible on mobile
- [x] Slot arrangement is consistent across all matches

**Definition of Done:**
- Ability pickup feels instant and responsive
- UI clearly communicates slot types and availability
- Swap mechanic works correctly in combat situations
- Edge cases handled (picking up while casting, full inventory)
- Tested with all ability combinations

**Dependencies:** UX-006 (Ability UI), GAME-008 (Ability Implementation)

**Deliverables:**
- [Ability Slot System Specification](docs/ABILITY_SLOT_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-008: Core Ability Implementation
**Priority:** P0 | **Complexity:** XL

**Description:**
Design and implement the initial roster of abilities. MVP requires minimum 10 abilities; soft launch requires 20+. Abilities should cover various archetypes: damage, mobility, crowd control, healing, and utility.

**Initial Ability Roster (MVP - 10 abilities):**

**Offensive Abilities:**
1. **Fireball** - Ranged skillshot, medium damage, small AoE
2. **Cutlass Slash** - Melee arc, high damage, short range
3. **Cannonball** - Long range, high damage, slow projectile
4. **Lightning Strike** - Target area, delayed AoE damage
5. **Poison Mackerel** - Thrown, DoT effect on hit

**Utility Abilities:**
6. **Grappling Hook** - Dash to target location, short range
7. **Barrel Roll** - Brief invulnerability + movement
8. **Healing Grog** - Self heal over time
9. **Smoke Bomb** - AoE vision block + slow enemies
10. **Wind Burst** - Knockback enemies in radius

**Acceptance Criteria:**
- [x] Each ability has unique visual and audio feedback
- [x] Abilities have appropriate cooldowns (5-20 seconds depending on power)
- [x] Skillshot abilities require aim input before firing
- [x] AoE abilities show ground indicator during targeting
- [x] Abilities interact correctly with terrain (no shooting through walls)
- [x] Each ability has clear counterplay potential
- [x] Abilities scale with rarity/upgrade level
- [x] Animation canceling rules defined and consistent

**Definition of Done:**
- All 10 MVP abilities fully functional in multiplayer
- Balance pass completed (no single OP ability)
- Each ability tested for mobile touch input compatibility
- VFX and SFX for all abilities implemented
- Ability descriptions written for UI display
- Cooldown and damage values documented in design spreadsheet

**Dependencies:** GAME-007, UX-003 (Touch Controls), ART-013 (Ability VFX)

**Deliverables:**
- [Core Ability Implementation Specification](docs/CORE_ABILITY_IMPLEMENTATION_SPEC.md)

**Status: Completed**

---

### GAME-009: Ability Upgrade System
**Priority:** P1 | **Complexity:** M

**Description:**
Implement the ability upgrade mechanic where collecting duplicate abilities increases their power. Abilities can be upgraded from Rank 1 to Rank 3 (matching Plunderstorm's system).

**Upgrade Tiers:**
- **Rank 1 (Common):** Base stats
- **Rank 2 (Rare):** +25% damage/effect, -10% cooldown
- **Rank 3 (Epic):** +50% damage/effect, -20% cooldown, bonus effect

**Acceptance Criteria:**
- [x] Picking up same ability when already equipped triggers upgrade
- [x] Visual feedback clearly shows upgrade occurring
- [x] Ability icon shows current rank (1-3 stars or color border)
- [x] Rank 3 abilities have enhanced VFX
- [x] Tooltip shows current rank stats vs next rank preview
- [x] Cannot upgrade beyond Rank 3 (duplicates become useless/small XP)
- [x] Higher rarity chests have higher chance of dropping upgraded abilities directly

**Definition of Done:**
- Upgrade system balanced (Rank 3 strong but not game-breaking)
- Clear visual distinction between all 3 ranks
- Network sync handles upgrades correctly
- UI shows upgrade path and current status
- Tested with rapid sequential upgrades

**Dependencies:** GAME-008, GAME-004

**Deliverable:** [Ability Upgrade System Specification](docs/ABILITY_UPGRADE_SYSTEM_SPEC.md)

**Status:** Completed

---

### GAME-010: Ability Combo System
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Design and implement synergies between certain abilities when used in combination. This adds strategic depth and rewards players who find complementary ability pairs.

**Example Combos:**
- Smoke Bomb + Lightning Strike = Increased damage in smoke
- Grappling Hook + Cutlass Slash = Bonus damage on arrival
- Wind Burst + Fireball = Extended fireball range
- Poison Mackerel + Healing Grog = Poison heals you instead

**Acceptance Criteria:**
- [x] At least 5 ability combos implemented for launch
- [x] Combo effects trigger when abilities used within time/space window
- [x] Visual/audio feedback indicates combo triggered
- [x] Combos documented in ability descriptions ("Combos with: X")
- [x] Combo effects are balanced (reward, not requirement)
- [x] Discovery of combos can be part of progression/achievements

**Definition of Done:**
- All combo interactions tested in multiplayer
- No exploitable infinite combos or loops
- Combo indicators clear but not UI-cluttering
- Balance validated through playtesting
- Combos feel rewarding and intentional

**Deliverables:**
- [Ability Combo System Specification](docs/ABILITY_COMBO_SYSTEM_SPEC.md)

**Dependencies:** GAME-008, GAME-009

---

## 3. Combat System

### GAME-011: Basic Attack System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement the always-available basic attack that all players have regardless of looted abilities. This should be a simple, reliable attack for early-game and ability cooldown periods.

**Acceptance Criteria:**
- [x] Basic attack available from match start (no pickup required)
- [x] Moderate damage, short-medium range, no cooldown (or very short ~1s)
- [x] Visual: Pirate pistol shot or magic bolt
- [x] Auto-aim assist option for mobile (soft lock to nearest enemy in cone)
- [x] Manual aim mode for skilled players
- [x] Cannot spam infinitely (slight fire rate limit)
- [x] Headshots deal bonus damage (optional - based on balance testing)
- [x] Audio and visual feedback on hit confirmation

**Definition of Done:**
- Basic attack feels responsive (< 50ms input to visual)
- Aim assist balanced (helps accuracy without being OP)
- Damage balanced relative to abilities
- Works correctly across all network conditions
- Tested on various device screen sizes

**Dependencies:** UX-003, GAME-012

**Deliverables:**
- [Basic Attack System Specification](docs/BASIC_ATTACK_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-012: Damage and Health System
**Priority:** P0 | **Complexity:** M

**Description:**
Implement the health and damage calculation system. Players start with base health that can be increased through leveling. Damage types, resistances, and hit detection must be server-authoritative.

**Acceptance Criteria:**
- [x] Base health: 100 HP at level 1
- [x] Health scaling: +10 HP per level (max 200 HP at level 10)
- [x] Damage numbers displayed on hit (floating combat text)
- [x] Critical hits possible (1.5x damage, visual indicator)
- [x] Health bar visible above player characters (enemy and self)
- [x] Low health warning (screen red edge, heartbeat SFX)
- [x] Health regeneration: None passive, only through abilities/items
- [x] Damage falloff for ranged abilities (optional, based on balance)
- [x] Armor/resistance system (future expansion hook)

**Definition of Done:**
- All damage calculations server-authoritative
- Hit registration accurate with lag compensation
- No health desync between client and server
- Damage values balanced for 10-15 minute match length
- UI clearly shows current health vs max health

**Dependencies:** GAME-005, BACK-003

**Deliverables:**
- [Damage and Health System Specification](docs/DAMAGE_HEALTH_SYSTEM_SPEC.md)

**Status: Completed**

---

### GAME-013: Hit Detection and Lag Compensation
**Priority:** P0 | **Complexity:** L

**Description:**
Implement robust hit detection that feels fair to all players regardless of network latency. Use server-authoritative hit registration with client-side prediction and lag compensation.

**Acceptance Criteria:**
- [x] Server is authoritative for all hit detection
- [x] Client-side prediction for immediate visual feedback
- [x] Lag compensation: Server rewinds to shooter's view time
- [x] Maximum acceptable lag compensation window: 200ms
- [x] Projectiles use server-authoritative trajectory simulation
- [x] Hitscan abilities (if any) use raycast with lag compensation
- [x] AoE abilities check positions at server tick time
- [x] Anti-cheat validation on hit claims
- [x] Hit markers show for attacker on successful damage

**Definition of Done:**
- Combat feels responsive with up to 150ms latency
- No advantage for high-latency players (within reason)
- Impossible to cheat hit detection client-side
- Tested with artificial latency injection
- Edge cases handled (shooting through closing doors, etc.)

**Dependencies:** BACK-003, BACK-004

**Deliverables:**
- [Hit Detection and Lag Compensation Specification](docs/HIT_DETECTION_LAG_COMPENSATION_SPEC.md)

**Status: Completed**

---

### GAME-014: Crowd Control System
**Priority:** P1 | **Complexity:** M

**Description:**
Implement crowd control (CC) effects that abilities can apply: stuns, slows, roots, knockbacks, and silences. Include diminishing returns to prevent chain-CC locks.

**CC Types:**
- **Stun:** Cannot move or use abilities
- **Slow:** Reduced movement speed (25-75%)
- **Root:** Cannot move, can still use abilities
- **Knockback:** Forced movement in direction
- **Silence:** Cannot use abilities, can still move

**Acceptance Criteria:**
- [x] Each CC type has clear visual indicator on affected player
- [x] CC duration shown on UI (debuff timer)
- [x] Diminishing returns: Repeated CC of same type has reduced duration
- [x] CC immunity window after extended CC (prevent permastun)
- [x] Some abilities grant CC immunity (e.g., Barrel Roll)
- [x] CC affects both players and NPCs
- [x] Knockbacks interact correctly with terrain/walls

**Definition of Done:**
- All CC types feel distinct and readable
- Diminishing returns prevent frustrating chain-CC
- CC abilities balanced in terms of duration and cooldown
- Network sync accurate for CC states
- Tested all CC interactions with all abilities

**Dependencies:** GAME-008, GAME-012

**Deliverable:** [Crowd Control System Specification](docs/CROWD_CONTROL_SYSTEM_SPEC.md)

**Status:** Completed

---

## 4. PvE Elements

### GAME-015: NPC Monster System
**Priority:** P1 | **Complexity:** L

**Description:**
Implement NPC monster camps scattered across the map. Monsters provide alternative XP source and can drop abilities. This accommodates players who prefer PvE-focused gameplay.

**Monster Types:**
1. **Skeleton Pirates (Common):** Low HP, low damage, groups of 3-5
2. **Sea Crabs (Common):** Medium HP, melee only, drops guaranteed ability
3. **Cursed Sailors (Uncommon):** Medium HP, ranged attack, higher XP
4. **Mini-Bosses (Rare):** High HP, unique attack patterns, epic loot potential

**Acceptance Criteria:**
- [x] Monster camps spawn at predefined locations on map
- [x] Monsters are aggressive when player enters range
- [x] Each monster has simple AI (patrol, chase, attack, return)
- [x] Killing monsters grants XP (scaling by type)
- [x] Monsters have chance to drop abilities (higher tier = better drops)
- [x] Monsters respawn after timer (60-90 seconds)
- [x] Monster health bars visible when in combat
- [x] Monsters can be kited, use terrain strategically
- [x] Monsters attack any nearby player (no ownership)

**Definition of Done:**
- Monster AI performs well with 60 players on map
- XP rewards balanced against PvP kills
- Drop rates tuned for economy balance
- Monsters visually fit pirate theme
- Server-authoritative monster state (no desync)

**Dependencies:** ART-011 (Monster Models), GAME-016

**Deliverable:** [NPC Monster System Specification](docs/NPC_MONSTER_SYSTEM_SPEC.md)

**Status:** Completed

---

### GAME-016: NPC AI Behavior System
**Priority:** P1 | **Complexity:** M

**Description:**
Create the AI behavior system for NPC monsters. AI should be simple but engaging, creating interesting PvE encounters without taxing server performance.

**AI States:**
- **Idle:** Patrol small area, ambient animations
- **Alert:** Player detected, begin approach
- **Combat:** Attack player in range, chase if retreating
- **Return:** If player too far, return to spawn area
- **Death:** Play death animation, drop loot

**Acceptance Criteria:**
- [x] AI uses state machine architecture for clarity
- [x] Pathfinding uses NavMesh (pre-baked for performance)
- [x] Monsters leash to spawn area (max chase distance)
- [x] Attack patterns are predictable (learnable by player)
- [x] Mini-bosses have 2-3 attack phases
- [x] AI doesn't get stuck on terrain
- [x] Multiple players can fight same monster camp
- [x] AI performance optimized (< 1ms per monster per tick)

**Definition of Done:**
- AI feels fair and predictable
- No exploits (safe spots, infinite kiting)
- Performance tested with max monster count
- All monster types have distinct behaviors
- Edge cases handled (disconnect while fighting, etc.)

**Dependencies:** GAME-015

**Deliverable:** [NPC AI Behavior System Specification](docs/NPC_AI_BEHAVIOR_SYSTEM_SPEC.md)

**Status:** Completed

---

### GAME-017: World Events System
**Priority:** P2 | **Complexity:** M | **Status: Completed**

**Description:**
Implement dynamic world events that occur during matches to create excitement and convergence points. These are optional high-risk, high-reward encounters.

**Event Types:**
1. **Treasure Ship:** Ship passes by edge of map, can be boarded for epic loot
2. **Kraken Sighting:** Mini-boss spawns in water area, drops legendary ability
3. **Supply Drop:** Marked crate drops from sky, contains rare+ abilities
4. **Ghost Galleon:** Temporary PvE area appears, clear for exclusive rewards

**Acceptance Criteria:**
- [x] 1-2 events occur per match at random times
- [x] Events announced globally (audio, UI notification)
- [x] Event locations marked on map
- [x] Events are contested (designed for player conflict)
- [x] Event rewards are significantly better than normal loot
- [x] Events have time limits (disappear if not engaged)
- [x] Events are optional (can ignore and still win)

**Definition of Done:**
- Events feel exciting and create memorable moments
- Event frequency balanced (not overwhelming)
- Rewards worth the risk
- Server handles events without performance impact
- Tested with full player count contesting events

**Deliverables:**
- [World Events System Specification](docs/WORLD_EVENTS_SYSTEM_SPEC.md)

**Dependencies:** GAME-015, GAME-004

---

## 5. In-Match Progression

### GAME-018: In-Match Leveling System
**Priority:** P1 | **Complexity:** M

**Description:**
Implement the within-match leveling system (1-10) that increases player power over the match duration. XP is earned from kills and PvE.

**Level Scaling:**
- Level 1: Base stats (100 HP, 100% damage)
- Level 5: +50 HP, +15% damage
- Level 10: +100 HP, +30% damage, +10% ability cooldown reduction

**XP Sources:**
- Player kill: 100 XP + 20 XP per victim's level
- Monster kill: 20-80 XP (scaling by monster type)
- Chest opened: 10 XP
- Survival time: 5 XP per 30 seconds

**Acceptance Criteria:**
- [x] XP bar visible on HUD
- [x] Level up triggers celebration effect (visual, audio)
- [x] Stats immediately update on level up
- [x] Level visible on player nameplates
- [x] XP gain shows as floating text
- [x] Level cap of 10 reached by ~70% of match duration
- [x] Falling behind in levels is recoverable (not snowbally)

**Definition of Done:**
- Leveling curve balanced for match duration
- Level advantages significant but not insurmountable
- XP tracking server-authoritative
- UI clearly shows level and XP progress
- Tested with various play styles (aggressive vs passive)

**Dependencies:** GAME-012, GAME-015, UX-005

**Deliverable:** [In-Match Leveling System Specification](docs/IN_MATCH_LEVELING_SYSTEM_SPEC.md)

**Status:** Completed

---

### GAME-019: Player Stats and Scaling
**Priority:** P1 | **Complexity:** S

**Description:**
Define and implement the core player stats that scale with level and potentially future systems (perks, items).

**Core Stats:**
- **Health:** Total HP
- **Damage:** Multiplier on all damage dealt
- **Cooldown Reduction:** Reduce ability cooldowns
- **Movement Speed:** Base movement rate

**Acceptance Criteria:**
- [x] Stats system is extensible for future additions
- [x] All stat effects are server-calculated
- [x] Stats visible in pause/info menu
- [x] Stat boosts stack additively (not multiplicatively to prevent exploits)
- [x] Temporary stat buffs possible (from abilities or pickups)

**Definition of Done:**
- Stats system documented and maintainable
- All stat effects tested for balance
- No stat overflow or negative value bugs
- Performance negligible (simple math)

**Dependencies:** GAME-018

**Deliverable:** [Player Stats and Scaling Specification](docs/PLAYER_STATS_SCALING_SPEC.md)

**Status:** Completed

---

## 6. Game Modes

### GAME-020: Solo Mode
**Priority:** P0 | **Complexity:** M

**Description:**
Implement the core solo battle royale mode: 30-60 players, last one standing wins.

**Acceptance Criteria:**
- [x] 30-60 players per match (scalable based on population)
- [x] Free-for-all, no teams
- [x] All core systems functional (drop, loot, combat, storm)
- [x] Match ends when 1 player remains
- [x] Placement tracked (1st, 2nd, 3rd, etc.)
- [x] Rewards based on placement
- [x] Queue available 24/7 (bot fill if needed)

**Definition of Done:**
- Solo mode fully playable end-to-end
- Matchmaking fills games within 30 seconds
- All edge cases handled (ties, disconnects)
- Mode selection UI complete
- Tested with full player count

**Dependencies:** All P0 GAME tasks, BACK-001

**Deliverables:**
- [Solo Mode Specification](docs/SOLO_MODE_SPEC.md)

**Status: Completed**

---

### GAME-021: Duo Mode
**Priority:** P1 | **Complexity:** M

**Description:**
Implement duo mode: teams of 2, last team standing wins. Includes teammate features.

**Acceptance Criteria:**
- [x] 30-60 players (15-30 teams)
- [x] Party system: queue with a friend
- [x] Random teammate matching if solo queue
- [x] Teammates visible on map (icon, distance)
- [x] Team voice chat or ping system
- [x] Downed state before elimination (can be revived)
- [x] Revive mechanic: 5 second channel, restores 50% HP
- [x] Both teammates must be eliminated for team loss
- [x] Share loot pings with teammate

**Definition of Done:**
- Team systems fully functional
- Voice chat or ping system works reliably
- Revive mechanic balanced
- Party formation UI complete
- Tested with full teams

**Dependencies:** GAME-020, BACK-002, UX-010

**Deliverable:** [Duo Mode Specification](docs/DUO_MODE_SPEC.md)

**Status:** Completed

---

### GAME-022: Squad Mode (Trios/Quads)
**Priority:** P3 | **Complexity:** M | **Status: Completed**

**Description:**
Extend team modes to support 3-4 player squads for post-launch content.

**Acceptance Criteria:**
- [x] Support 3 or 4 player teams
- [x] All duo features extended to larger teams
- [x] Party UI scales to squad size
- [x] Performance acceptable with 60 players in squads

**Definition of Done:**
- Squad mode fully functional
- Tested with various squad sizes
- UI adapts to squad size
- Matchmaking handles partial squads

**Deliverables:**
- [Squad Mode Specification](docs/SQUAD_MODE_SPEC.md)

**Dependencies:** GAME-021

---

### GAME-023: Ranked Mode
**Priority:** P2 | **Complexity:** L | **Status: Completed**

**Description:**
Implement competitive ranked mode with skill-based matchmaking, ranks, and seasonal resets.

**Rank Tiers:**
1. Deckhand (Bronze)
2. Sailor (Silver)
3. Boatswain (Gold)
4. First Mate (Platinum)
5. Captain (Diamond)
6. Admiral (Master)
7. Pirate King (Top 100)

**Acceptance Criteria:**
- [x] Ranked points (RP) gained/lost based on placement and kills
- [x] SBMM matches players of similar rank
- [x] Visible rank and progress in profile
- [x] Seasonal reset (soft reset, not full)
- [x] Season rewards based on peak rank achieved
- [x] Ranked exclusive cosmetics
- [x] Stricter AFK/leaving penalties in ranked
- [x] Minimum level requirement to enter ranked

**Definition of Done:**
- Rank system feels competitive and fair
- SBMM produces balanced matches
- Season infrastructure ready
- Rewards pipeline established
- Anti-smurf measures in place

**Deliverables:**
- [Ranked Mode Specification](docs/RANKED_MODE_SPEC.md)

**Dependencies:** GAME-020, BACK-001, BACK-006

---

### GAME-024: Practice/Tutorial Mode
**Priority:** P1 | **Complexity:** M

**Description:**
Create a single-player or low-stakes mode for new players to learn game mechanics without the pressure of real competition.

**Components:**
1. **Tutorial:** Guided introduction to controls, abilities, combat
2. **Training Ground:** Sandbox to test abilities, no enemies
3. **Bot Match:** Full match against AI players only

**Acceptance Criteria:**
- [x] Mandatory tutorial for first-time players (skippable after)
- [x] Tutorial covers: movement, aiming, ability pickup, ability use, storm
- [x] Training ground available from main menu
- [x] Bot match option in casual queue
- [x] Bots have adjustable difficulty (Easy, Medium, Hard)
- [x] Progress in tutorial unlocks small reward

**Definition of Done:**
- New players understand core mechanics after tutorial
- Tutorial completion rate > 80%
- Training ground includes all current abilities
- Bots provide reasonable challenge at each difficulty
- Tested with real new players for UX feedback

**Dependencies:** GAME-016 (AI), UX-002

**Deliverable:** [Practice/Tutorial Mode Specification](docs/PRACTICE_TUTORIAL_MODE_SPEC.md)

**Status:** Completed

---

## Task Summary

| Task ID | Title | Priority | Complexity | Status |
|---------|-------|----------|------------|--------|
| GAME-001 | Match Initialization System | P0 | L | Completed |
| GAME-002 | Player Drop-In System | P0 | M | Completed |
| GAME-003 | Storm/Safe Zone System | P0 | M | Completed |
| GAME-004 | Loot Spawn System | P0 | M | Completed |
| GAME-005 | Player Elimination System | P0 | S | Completed |
| GAME-006 | Spectator Mode | P2 | M | Completed |
| GAME-007 | Ability Slot System | P0 | M | Completed |
| GAME-008 | Core Ability Implementation | P0 | XL | Completed |
| GAME-009 | Ability Upgrade System | P1 | M | Completed |
| GAME-010 | Ability Combo System | P2 | M | Completed |
| GAME-011 | Basic Attack System | P0 | M | Completed |
| GAME-012 | Damage and Health System | P0 | M | Completed |
| GAME-013 | Hit Detection and Lag Compensation | P0 | L | Completed |
| GAME-014 | Crowd Control System | P1 | M | Completed |
| GAME-015 | NPC Monster System | P1 | L | Completed |
| GAME-016 | NPC AI Behavior System | P1 | M | Completed |
| GAME-017 | World Events System | P2 | M | Completed |
| GAME-018 | In-Match Leveling System | P1 | M | Completed |
| GAME-019 | Player Stats and Scaling | P1 | S | Completed |
| GAME-020 | Solo Mode | P0 | M | Completed |
| GAME-021 | Duo Mode | P1 | M | Completed |
| GAME-022 | Squad Mode | P3 | M | Completed |
| GAME-023 | Ranked Mode | P2 | L | Completed |
| GAME-024 | Practice/Tutorial Mode | P1 | M | Completed |
