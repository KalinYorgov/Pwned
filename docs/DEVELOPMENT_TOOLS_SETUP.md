# Development Tools Setup Guide
# Plunderstorm Mobile - Infrastructure & Tooling

**Document Version:** 1.0
**Last Updated:** 2025-11-30
**Task ID:** PROJ-017
**Status:** Completed

---

## Table of Contents

1. [Overview](#1-overview)
2. [Tool Selection Summary](#2-tool-selection-summary)
3. [Source Control](#3-source-control)
4. [Project Management](#4-project-management)
5. [Communication](#5-communication)
6. [Build System (CI/CD)](#6-build-system-cicd)
7. [Asset Management](#7-asset-management)
8. [Documentation](#8-documentation)
9. [Bug Tracking](#9-bug-tracking)
10. [Analytics Platform](#10-analytics-platform)
11. [Development Environment](#11-development-environment)
12. [Workflows](#12-workflows)
13. [Access Permissions](#13-access-permissions)
14. [Backup Procedures](#14-backup-procedures)
15. [Onboarding Checklist](#15-onboarding-checklist)

---

## 1. Overview

This document defines the complete development infrastructure for the Plunderstorm Mobile project. All team members should reference this guide for tool setup, workflows, and best practices.

### 1.1 Goals

- **Efficiency:** Tools should accelerate development, not slow it down
- **Collaboration:** Enable seamless teamwork across disciplines
- **Transparency:** Everyone can see project status and progress
- **Quality:** Automated checks catch issues early
- **Security:** Protect code, assets, and player data

### 1.2 Tool Selection Criteria

All tools were selected based on:
- Cost effectiveness for indie/small team
- Mobile game development suitability
- Integration capabilities
- Team familiarity
- Scalability

---

## 2. Tool Selection Summary

| Category | Primary Tool | Alternative | Cost |
|----------|--------------|-------------|------|
| Source Control | GitHub | GitLab | Free tier / $4/user/mo |
| Project Management | Linear | Trello, Jira | Free tier / $8/user/mo |
| Communication | Discord | Slack | Free |
| CI/CD | GitHub Actions | Unity Cloud Build | Free tier included |
| Asset Management | Git LFS + Google Drive | Perforce | Free / Low cost |
| Documentation | Notion | Confluence, GitBook | Free tier |
| Bug Tracking | Linear (integrated) | GitHub Issues | Included |
| Analytics | Unity Analytics + GameAnalytics | Firebase | Free |
| Crash Reporting | Firebase Crashlytics | Sentry | Free |
| Game Engine | Unity 2022 LTS | Unreal Engine 5 | Free < $100K rev |
| Backend | PlayFab | Firebase + Custom | Free tier |

---

## 3. Source Control

### 3.1 Platform: GitHub

**Repository:** `github.com/[organization]/plunderstorm-mobile`

**Why GitHub:**
- Industry standard for game development
- Excellent CI/CD integration (GitHub Actions)
- Free private repositories
- Great code review tools
- Large community and documentation

### 3.2 Repository Structure

```
plunderstorm-mobile/
├── .github/
│   ├── workflows/           # CI/CD pipelines
│   ├── ISSUE_TEMPLATE/      # Bug/feature templates
│   └── PULL_REQUEST_TEMPLATE.md
├── Assets/                  # Unity Assets folder
│   ├── _Project/           # Our custom assets
│   │   ├── Art/
│   │   ├── Audio/
│   │   ├── Prefabs/
│   │   ├── Scenes/
│   │   ├── Scripts/
│   │   └── UI/
│   ├── Plugins/            # Third-party SDKs
│   └── StreamingAssets/    # Runtime loaded assets
├── Packages/               # Unity packages
├── ProjectSettings/        # Unity project settings
├── docs/                   # Documentation
│   ├── GAME_DESIGN_DOCUMENT.md
│   ├── DEVELOPMENT_TOOLS_SETUP.md
│   └── ...
├── tools/                  # Build scripts, utilities
├── .gitignore
├── .gitattributes          # LFS configuration
└── README.md
```

### 3.3 Branching Strategy

**Git Flow (Simplified)**

```
main (production)
  │
  └── develop (integration)
        │
        ├── feature/PROJ-XXX-description
        ├── feature/GAME-XXX-description
        ├── bugfix/issue-number-description
        └── release/v1.0.0
```

| Branch | Purpose | Merge To | Protection |
|--------|---------|----------|------------|
| `main` | Production releases | - | Protected, requires PR |
| `develop` | Integration branch | main | Protected, requires PR |
| `feature/*` | New features | develop | None |
| `bugfix/*` | Bug fixes | develop | None |
| `hotfix/*` | Emergency fixes | main + develop | None |
| `release/*` | Release preparation | main + develop | Protected |

### 3.4 Branch Naming Convention

```
<type>/<task-id>-<short-description>

Examples:
feature/GAME-001-match-initialization
feature/ART-003-player-character-model
bugfix/123-fix-crash-on-login
hotfix/critical-iap-validation
```

### 3.5 Commit Message Convention

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code style (formatting)
- `refactor`: Code refactoring
- `perf`: Performance improvement
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples:**
```
feat(abilities): implement Fireball ability

- Add projectile physics
- Add explosion VFX on impact
- Add damage calculation

Closes GAME-008
```

```
fix(networking): resolve player desync on reconnect

Players were receiving stale position data after
reconnecting to a match.

Fixes #234
```

### 3.6 Git LFS Configuration

Large files are stored with Git LFS to keep repository performant.

**.gitattributes:**
```
# Images
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.psd filter=lfs diff=lfs merge=lfs -text
*.tga filter=lfs diff=lfs merge=lfs -text

# Audio
*.wav filter=lfs diff=lfs merge=lfs -text
*.mp3 filter=lfs diff=lfs merge=lfs -text
*.ogg filter=lfs diff=lfs merge=lfs -text

# 3D Models
*.fbx filter=lfs diff=lfs merge=lfs -text
*.obj filter=lfs diff=lfs merge=lfs -text
*.blend filter=lfs diff=lfs merge=lfs -text

# Unity
*.unity filter=lfs diff=lfs merge=lfs -text
*.asset filter=lfs diff=lfs merge=lfs -text
*.prefab filter=lfs diff=lfs merge=lfs -text

# Archives
*.zip filter=lfs diff=lfs merge=lfs -text
*.unitypackage filter=lfs diff=lfs merge=lfs -text
```

### 3.7 .gitignore (Unity)

```gitignore
# Unity
[Ll]ibrary/
[Tt]emp/
[Oo]bj/
[Bb]uild/
[Bb]uilds/
[Ll]ogs/
[Mm]emoryCaptures/
*.csproj
*.unityproj
*.sln
*.suo
*.user
*.pidb
*.booproj

# IDE
.idea/
.vs/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Build outputs
*.apk
*.aab
*.ipa
*.app

# Sensitive
*.keystore
*.p12
google-services.json
GoogleService-Info.plist
```

---

## 4. Project Management

### 4.1 Platform: Linear

**Workspace:** `linear.app/plunderstorm`

**Why Linear:**
- Clean, fast interface
- Built for software development
- Excellent GitHub integration
- Cycles (sprints) management
- Roadmap visualization

### 4.2 Project Structure

```
Workspace: Plunderstorm Mobile
├── Team: Engineering
│   ├── Project: Client
│   ├── Project: Server
│   └── Project: Infrastructure
├── Team: Art
│   ├── Project: Characters
│   ├── Project: Environment
│   └── Project: UI
├── Team: Design
│   ├── Project: Game Design
│   └── Project: Level Design
└── Team: Production
    ├── Project: Planning
    └── Project: QA
```

### 4.3 Issue Types

| Type | Icon | Description |
|------|------|-------------|
| Feature | ⭐ | New functionality |
| Bug | 🐛 | Something broken |
| Task | ✅ | General work item |
| Improvement | 🔧 | Enhancement to existing |
| Spike | 🔬 | Research/investigation |

### 4.4 Issue States

```
Backlog → Todo → In Progress → In Review → Done
                      ↓
                  Blocked
```

### 4.5 Priority Levels

| Priority | Label | Response Time |
|----------|-------|---------------|
| Urgent | 🔴 P0 | Same day |
| High | 🟠 P1 | This cycle |
| Medium | 🟡 P2 | Next cycle |
| Low | 🟢 P3 | Backlog |

### 4.6 Cycles (Sprints)

- **Duration:** 2 weeks
- **Planning:** Monday of week 1
- **Review:** Friday of week 2
- **Retrospective:** Friday of week 2

### 4.7 Labels

| Category | Labels |
|----------|--------|
| Domain | `gameplay`, `ui`, `networking`, `art`, `audio`, `backend` |
| Platform | `ios`, `android`, `both` |
| Type | `tech-debt`, `optimization`, `refactor` |

---

## 5. Communication

### 5.1 Platform: Discord

**Server:** Plunderstorm Dev

**Why Discord:**
- Free for unlimited users
- Voice channels for meetings
- Screen sharing
- Thread support
- Bot integrations
- Community familiarity

### 5.2 Channel Structure

```
PLUNDERSTORM DEV
├── 📢 ANNOUNCEMENTS
│   └── #announcements        (read-only, important updates)
├── 💬 GENERAL
│   ├── #general              (casual chat)
│   ├── #random               (off-topic)
│   └── #introductions        (new member intros)
├── 💻 DEVELOPMENT
│   ├── #dev-general          (development discussion)
│   ├── #dev-client           (Unity, gameplay)
│   ├── #dev-server           (backend, networking)
│   ├── #dev-art              (art pipeline, assets)
│   ├── #dev-design           (game design discussion)
│   └── #code-review          (PR discussions)
├── 🤖 INTEGRATIONS
│   ├── #github-feed          (commits, PRs, issues)
│   ├── #build-status         (CI/CD notifications)
│   ├── #alerts               (monitoring alerts)
│   └── #linear-updates       (task updates)
├── 🎮 PLAYTESTING
│   ├── #playtest-schedule    (upcoming tests)
│   ├── #playtest-feedback    (feedback collection)
│   └── #bug-reports          (quick bug reports)
├── 🔊 VOICE
│   ├── 🔊 Daily Standup
│   ├── 🔊 Meeting Room 1
│   ├── 🔊 Meeting Room 2
│   └── 🔊 Pair Programming
└── 🔒 PRIVATE
    ├── #leadership           (leads only)
    └── #hr-admin             (sensitive topics)
```

### 5.3 Communication Guidelines

| Type | Channel | Response Time |
|------|---------|---------------|
| Urgent/Blocking | Direct message + tag | < 1 hour |
| Important | Relevant channel + tag | < 4 hours |
| Discussion | Relevant channel | < 24 hours |
| FYI | Relevant channel | No response needed |

### 5.4 Meeting Schedule

| Meeting | Frequency | Duration | Attendees |
|---------|-----------|----------|-----------|
| Daily Standup | Daily, 10:00 AM | 15 min | All |
| Sprint Planning | Bi-weekly, Monday | 1-2 hours | All |
| Sprint Review | Bi-weekly, Friday | 1 hour | All |
| Retrospective | Bi-weekly, Friday | 30 min | All |
| Design Review | Weekly, Wednesday | 1 hour | Design + Leads |
| Tech Sync | Weekly, Tuesday | 30 min | Engineering |

### 5.5 Discord Bots

| Bot | Purpose |
|-----|---------|
| GitHub Bot | PR/commit notifications |
| Linear Bot | Task updates |
| Build Bot | CI/CD status |
| PlayFab Bot | Backend alerts |

---

## 6. Build System (CI/CD)

### 6.1 Platform: GitHub Actions

**Why GitHub Actions:**
- Integrated with repository
- Free minutes for private repos
- Pre-built Unity actions available
- Matrix builds for iOS/Android
- Artifact storage

### 6.2 Build Pipelines

#### 6.2.1 Pull Request Validation

**Trigger:** On PR to `develop` or `main`

```yaml
name: PR Validation

on:
  pull_request:
    branches: [develop, main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Lint C# Code
        uses: dotnet/format@v1

      - name: Run Unit Tests
        uses: game-ci/unity-test-runner@v2
        with:
          projectPath: .
          testMode: EditMode

      - name: Build Validation (WebGL)
        uses: game-ci/unity-builder@v2
        with:
          targetPlatform: WebGL
          # Quick build to validate compilation
```

#### 6.2.2 Development Build

**Trigger:** On push to `develop`

```yaml
name: Development Build

on:
  push:
    branches: [develop]

jobs:
  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          lfs: true

      - name: Build Android APK
        uses: game-ci/unity-builder@v2
        with:
          targetPlatform: Android
          buildName: PlunderstormMobile-Dev

      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          groups: internal-testers

  build-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
        with:
          lfs: true

      - name: Build iOS
        uses: game-ci/unity-builder@v2
        with:
          targetPlatform: iOS

      - name: Upload to TestFlight
        uses: apple-actions/upload-testflight-build@v1
```

#### 6.2.3 Release Build

**Trigger:** On push to `release/*` or tag `v*`

```yaml
name: Release Build

on:
  push:
    branches: [release/*]
    tags: [v*]

jobs:
  build-release:
    strategy:
      matrix:
        platform: [Android, iOS]
    runs-on: ${{ matrix.platform == 'iOS' && 'macos-latest' || 'ubuntu-latest' }}
    steps:
      - uses: actions/checkout@v3
        with:
          lfs: true

      - name: Build ${{ matrix.platform }}
        uses: game-ci/unity-builder@v2
        with:
          targetPlatform: ${{ matrix.platform }}
          buildName: PlunderstormMobile-${{ github.ref_name }}

      - name: Sign and Upload
        # Platform-specific signing and store upload
```

### 6.3 Build Artifacts

| Artifact | Retention | Storage |
|----------|-----------|---------|
| PR Builds | 7 days | GitHub Artifacts |
| Dev Builds | 30 days | Firebase + S3 |
| Release Builds | Permanent | S3 + Store archives |

### 6.4 Build Versioning

```
<major>.<minor>.<patch>+<build>

Example: 1.2.3+456

Major: Breaking changes, major features
Minor: New features, significant changes
Patch: Bug fixes, small changes
Build: Auto-incremented CI build number
```

---

## 7. Asset Management

### 7.1 Strategy: Git LFS + Cloud Storage

**Why This Approach:**
- Source files (PSD, Blend) too large for Git LFS
- Final assets tracked in LFS
- Source files synced via cloud storage

### 7.2 Asset Storage Locations

| Asset Type | Location | Reason |
|------------|----------|--------|
| Final game assets | Git LFS | Version control, CI access |
| Source files (PSD, AI, Blend) | Google Drive | Large files, collaboration |
| Reference/Inspiration | Notion | Documentation |
| Audio source (DAW projects) | Google Drive | Large files |

### 7.3 Google Drive Structure

```
Plunderstorm Assets/
├── 01_Characters/
│   ├── Player/
│   │   ├── Base_Model.blend
│   │   ├── Textures_PSD/
│   │   └── Exports/  (→ synced to Git)
│   └── Monsters/
├── 02_Environment/
│   ├── Props/
│   ├── Terrain/
│   └── Buildings/
├── 03_UI/
│   ├── Icons/
│   ├── Screens/
│   └── Fonts/
├── 04_Audio/
│   ├── Music_Projects/
│   ├── SFX_Projects/
│   └── Exports/  (→ synced to Git)
├── 05_VFX/
├── 06_Animation/
└── 07_Reference/
    ├── Style_Guide/
    ├── Inspiration/
    └── Competitor_Analysis/
```

### 7.4 Asset Naming Convention

```
<Category>_<Name>_<Variant>_<Size>.<ext>

Examples:
Char_Pirate_Default_Body.fbx
Env_Palm_Tree_Large.fbx
UI_Icon_Fireball_64.png
VFX_Explosion_Fire_01.prefab
Audio_SFX_Sword_Hit_01.ogg
```

### 7.5 Export Workflow

1. Artist creates/updates source file in Google Drive
2. Artist exports final asset to `Exports/` folder
3. Artist copies export to Git repository
4. Artist commits with appropriate message
5. CI validates asset on PR

---

## 8. Documentation

### 8.1 Platform: Notion

**Workspace:** `notion.so/plunderstorm`

**Why Notion:**
- Flexible document structure
- Great for wikis and docs
- Real-time collaboration
- Templates support
- Free for small teams

### 8.2 Documentation Structure

```
Plunderstorm Wiki
├── 🏠 Home
├── 📋 Project
│   ├── Game Design Document
│   ├── Technical Design Docs
│   ├── Art Bible
│   └── Roadmap
├── 🛠️ Engineering
│   ├── Architecture Overview
│   ├── Coding Standards
│   ├── API Documentation
│   └── Troubleshooting
├── 🎨 Art
│   ├── Style Guide
│   ├── Asset Pipeline
│   └── Technical Specs
├── 📊 Analytics
│   ├── KPIs Dashboard
│   ├── Event Definitions
│   └── Reports
├── 🧪 QA
│   ├── Test Plans
│   ├── Test Cases
│   └── Device Matrix
├── 🚀 Operations
│   ├── Deployment Guide
│   ├── Incident Response
│   └── Runbooks
└── 📚 Onboarding
    ├── Getting Started
    ├── Tool Setup
    └── Team Directory
```

### 8.3 Documentation Types

| Type | Location | Format |
|------|----------|--------|
| Design Docs | Notion | Wiki pages |
| API Docs | Code + Notion | XML comments + generated |
| README files | Git repo | Markdown |
| Architecture | Notion | Diagrams + text |
| Meeting Notes | Notion | Templates |
| Decisions (ADRs) | Git repo `/docs/adr/` | Markdown |

### 8.4 Architecture Decision Records (ADRs)

Store in `/docs/adr/` with format:

```markdown
# ADR-001: Use Photon for Networking

## Status
Accepted

## Context
We need a networking solution for 60-player matches.

## Decision
Use Photon Fusion for real-time multiplayer.

## Consequences
- Pros: Proven, managed, good Unity support
- Cons: Ongoing cost, vendor lock-in
```

---

## 9. Bug Tracking

### 9.1 Platform: Linear (Integrated)

Bug tracking is integrated with project management in Linear.

### 9.2 Bug Report Template

```markdown
## Description
[Clear description of the bug]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Environment
- Platform: [iOS/Android]
- Device: [e.g., iPhone 13, Pixel 6]
- OS Version: [e.g., iOS 16.1, Android 13]
- App Version: [e.g., 1.2.3+456]
- Network: [WiFi/4G/5G]

## Additional Context
- Screenshot/Video: [attach]
- Logs: [attach]
- Frequency: [Always/Sometimes/Rare]
- Severity: [Blocker/Critical/Major/Minor/Trivial]
```

### 9.3 Bug Severity Definitions

| Severity | Definition | Example |
|----------|------------|---------|
| Blocker | Cannot proceed, no workaround | Crash on launch |
| Critical | Major feature broken | Cannot join matches |
| Major | Feature impaired | Ability doesn't deal damage |
| Minor | Inconvenience | UI text misaligned |
| Trivial | Cosmetic | Minor visual glitch |

### 9.4 Bug Workflow

```
New → Triaged → In Progress → Fixed → Verified → Closed
        ↓                        ↓
    Won't Fix              Cannot Reproduce
```

---

## 10. Analytics Platform

### 10.1 Platforms

| Platform | Purpose | Events |
|----------|---------|--------|
| Unity Analytics | Core gameplay metrics | Sessions, retention, progression |
| GameAnalytics | Game-specific metrics | Ability usage, balance data |
| PlayFab Analytics | Economy metrics | IAP, currency flow |
| Firebase Analytics | Marketing attribution | Installs, campaigns |

### 10.2 Key Metrics Dashboard

**Engagement:**
- DAU / MAU / WAU
- Session length
- Sessions per day
- D1, D7, D30 retention

**Gameplay:**
- Matches played
- Win rate distribution
- Ability pick rates
- Average match duration

**Monetization:**
- Revenue (daily, weekly, monthly)
- ARPDAU, ARPPU
- Conversion rate
- IAP breakdown

### 10.3 Event Naming Convention

```
<category>_<action>_<detail>

Examples:
match_started
match_completed_win
match_completed_loss
ability_used_fireball
ability_upgraded_fireball_rank2
store_item_purchased
store_item_viewed
```

### 10.4 Event Implementation Guide

```csharp
// Unity Analytics example
public static class GameAnalytics
{
    public static void TrackMatchStarted(string matchId, string mode)
    {
        Analytics.CustomEvent("match_started", new Dictionary<string, object>
        {
            { "match_id", matchId },
            { "mode", mode },
            { "player_level", PlayerData.Level }
        });
    }

    public static void TrackAbilityUsed(string abilityId, int rank)
    {
        Analytics.CustomEvent("ability_used", new Dictionary<string, object>
        {
            { "ability_id", abilityId },
            { "rank", rank },
            { "match_time", MatchManager.ElapsedTime }
        });
    }
}
```

---

## 11. Development Environment

### 11.1 Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Unity | 2022.3 LTS | Game engine |
| Visual Studio / Rider | Latest | IDE |
| Git | 2.40+ | Version control |
| Git LFS | 3.0+ | Large file storage |
| Node.js | 18 LTS | Build tools |
| Android Studio | Latest | Android SDK, emulators |
| Xcode | 15+ | iOS builds (macOS only) |

### 11.2 Unity Project Setup

1. Clone repository
2. Install Git LFS: `git lfs install`
3. Pull LFS files: `git lfs pull`
4. Open project in Unity Hub
5. Install required packages (auto-resolved)
6. Import settings from `ProjectSettings/`

### 11.3 IDE Configuration

**Visual Studio / Rider Extensions:**
- Unity support
- EditorConfig support
- GitLens (VS Code)

**.editorconfig:**
```ini
root = true

[*.cs]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

# C# specific
csharp_new_line_before_open_brace = all
csharp_indent_case_contents = true
```

### 11.4 Unity Coding Standards

```csharp
// Naming Conventions
public class PlayerController { }           // PascalCase for classes
public void HandleInput() { }               // PascalCase for methods
public float moveSpeed;                      // camelCase for public fields
private float _health;                       // _camelCase for private fields
private const float MAX_HEALTH = 100f;       // UPPER_CASE for constants

// File Organization
// 1. Using statements
// 2. Namespace
// 3. Class declaration
// 4. Constants
// 5. Serialized fields
// 6. Private fields
// 7. Properties
// 8. Unity lifecycle methods
// 9. Public methods
// 10. Private methods
```

---

## 12. Workflows

### 12.1 Feature Development Workflow

```
1. Create Linear issue (or pick from backlog)
2. Create feature branch from develop
   git checkout develop
   git pull
   git checkout -b feature/GAME-XXX-description

3. Implement feature
   - Write code
   - Add tests
   - Update documentation

4. Commit regularly with clear messages
   git commit -m "feat(gameplay): add fireball projectile physics"

5. Push and create PR
   git push -u origin feature/GAME-XXX-description
   # Create PR via GitHub UI

6. Request review
   - Assign reviewers
   - Link Linear issue
   - Add screenshots/videos if UI

7. Address review feedback

8. Merge after approval
   - Squash merge preferred
   - Delete branch after merge

9. Update Linear issue status
```

### 12.2 Bug Fix Workflow

```
1. Reproduce bug locally
2. Create bugfix branch
   git checkout -b bugfix/123-description

3. Write failing test (if applicable)
4. Fix the bug
5. Verify fix locally
6. Push and create PR
7. Get review and merge
8. Verify fix in develop build
9. Close bug in Linear
```

### 12.3 Release Workflow

```
1. Create release branch from develop
   git checkout -b release/v1.2.0

2. Update version numbers
   - Unity Player Settings
   - Package files

3. Final testing on release branch
4. Fix any critical bugs on release branch
5. Merge release to main
   git checkout main
   git merge release/v1.2.0

6. Tag the release
   git tag v1.2.0
   git push --tags

7. CI builds and uploads to stores
8. Merge main back to develop
   git checkout develop
   git merge main

9. Delete release branch
10. Monitor release metrics
```

### 12.3 Hotfix Workflow

```
1. Create hotfix branch from main
   git checkout main
   git checkout -b hotfix/critical-fix

2. Fix the issue
3. Test thoroughly
4. Merge to main
5. Tag new patch version
6. Merge to develop
7. Delete hotfix branch
```

---

## 13. Access Permissions

### 13.1 Role Definitions

| Role | Description |
|------|-------------|
| Admin | Full access to all systems |
| Lead | Team management + technical access |
| Developer | Code, assets, project tools |
| Artist | Asset tools, read-only code |
| QA | Testing tools, bug tracking |
| Viewer | Read-only access |

### 13.2 Permission Matrix

| System | Admin | Lead | Developer | Artist | QA | Viewer |
|--------|-------|------|-----------|--------|-----|--------|
| GitHub (code) | Admin | Maintain | Write | Read | Read | Read |
| GitHub (merge to main) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Linear | Admin | Admin | Write | Write | Write | Read |
| Discord | Admin | Manage | Member | Member | Member | Guest |
| Notion | Admin | Editor | Editor | Editor | Editor | Reader |
| Google Drive | Admin | Editor | Editor | Editor | Reader | Reader |
| PlayFab | Admin | Admin | Write | Read | Read | Read |
| Firebase | Admin | Editor | Editor | Read | Read | Read |
| App Store Connect | Admin | Admin | Developer | - | - | - |
| Google Play Console | Admin | Admin | Developer | - | - | - |

### 13.3 Onboarding Access Checklist

```markdown
## New Team Member Access Checklist

- [ ] GitHub: Add to organization and team
- [ ] Linear: Invite to workspace
- [ ] Discord: Send invite link
- [ ] Notion: Invite to workspace
- [ ] Google Drive: Share folders
- [ ] Unity License: Assign seat
- [ ] PlayFab: Add as team member
- [ ] Firebase: Add to project
- [ ] 1Password/Vault: Share credentials (if applicable)
```

---

## 14. Backup Procedures

### 14.1 Backup Strategy

| Data | Backup Method | Frequency | Retention |
|------|---------------|-----------|-----------|
| Code (GitHub) | GitHub redundancy + local clones | Continuous | Permanent |
| Assets (Git LFS) | GitHub LFS + Google Drive | Continuous | Permanent |
| Source Assets | Google Drive | Real-time sync | 30 days versioning |
| Notion Docs | Notion export + Git | Weekly | 90 days |
| PlayFab Data | PlayFab backup | Daily | 30 days |
| Analytics | Platform retention | - | Per platform |
| Builds | S3 / Cloud Storage | Per build | 90 days dev, permanent release |

### 14.2 Disaster Recovery

**Scenario: GitHub unavailable**
- All developers have local clones
- Can push to backup GitLab mirror
- RTO: < 4 hours

**Scenario: Google Drive data loss**
- Weekly exports to backup storage
- Version history for 30 days
- RTO: < 24 hours

**Scenario: PlayFab outage**
- PlayFab has 99.9% SLA
- Fallback: read-only mode
- Player data recoverable from backups

### 14.3 Backup Verification

- **Monthly:** Verify backup integrity
- **Quarterly:** Test restore procedure
- **Annually:** Full disaster recovery drill

---

## 15. Onboarding Checklist

### 15.1 New Developer Checklist

```markdown
## Day 1

### Accounts & Access
- [ ] Receive login credentials
- [ ] Set up 2FA on all accounts
- [ ] GitHub: Accept org invite
- [ ] Linear: Accept workspace invite
- [ ] Discord: Join server
- [ ] Notion: Accept workspace invite
- [ ] Google Drive: Verify access

### Local Setup
- [ ] Install Unity 2022.3 LTS
- [ ] Install IDE (Visual Studio / Rider)
- [ ] Install Git + Git LFS
- [ ] Clone repository
- [ ] Open project in Unity (verify no errors)

### Orientation
- [ ] Read Game Design Document
- [ ] Read this Development Tools Setup guide
- [ ] Review project README
- [ ] Meet with team lead

## Week 1

### Deep Dive
- [ ] Review codebase architecture
- [ ] Complete first small task
- [ ] Submit first PR
- [ ] Attend team meetings
- [ ] Set up build pipeline (run local build)

### Social
- [ ] Introduction in #introductions
- [ ] 1:1 with each team member
- [ ] Add profile to team directory
```

### 15.2 Recommended Reading Order

1. This document (Development Tools Setup)
2. Game Design Document
3. Repository README
4. Architecture Overview (Notion)
5. Coding Standards
6. Your team's specific documentation

---

## Appendix A: Quick Reference

### Useful Commands

```bash
# Clone with LFS
git clone --recurse-submodules <repo-url>
git lfs pull

# Create feature branch
git checkout develop && git pull
git checkout -b feature/GAME-XXX-description

# Commit
git add .
git commit -m "feat(scope): description"

# Push new branch
git push -u origin feature/GAME-XXX-description

# Update from develop
git checkout develop && git pull
git checkout feature/my-feature
git rebase develop

# Unity command line build (example)
Unity -batchmode -projectPath . -buildTarget Android -executeMethod BuildScript.BuildAndroid -quit
```

### Important Links

| Resource | URL |
|----------|-----|
| GitHub Repository | `github.com/[org]/plunderstorm-mobile` |
| Linear | `linear.app/plunderstorm` |
| Notion | `notion.so/plunderstorm` |
| Discord | `discord.gg/[invite]` |
| Google Drive | `drive.google.com/[folder]` |
| PlayFab Dashboard | `developer.playfab.com` |
| Firebase Console | `console.firebase.google.com` |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | Dev Team | Initial creation |

---

**End of Development Tools Setup Guide**
