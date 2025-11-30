# Outsourcing Plan - Plunderstorm Mobile

**Document ID:** PROJ-016
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Outsourcing Strategy](#2-outsourcing-strategy)
3. [In-House vs Outsource Decisions](#3-in-house-vs-outsource-decisions)
4. [Vendor Categories](#4-vendor-categories)
5. [Vendor Selection Criteria](#5-vendor-selection-criteria)
6. [Recommended Vendors](#6-recommended-vendors)
7. [Contract Framework](#7-contract-framework)
8. [Vendor Onboarding Process](#8-vendor-onboarding-process)
9. [Quality Control Process](#9-quality-control-process)
10. [Communication Protocols](#10-communication-protocols)
11. [Budget Allocation](#11-budget-allocation)
12. [Risk Management](#12-risk-management)
13. [Appendices](#13-appendices)

---

## 1. Executive Summary

### 1.1 Purpose

This document outlines the outsourcing strategy for Plunderstorm Mobile, identifying which work streams will be handled externally, establishing vendor relationships, and defining processes for quality control and vendor management.

### 1.2 Outsourcing Philosophy

Our outsourcing strategy follows these principles:

1. **Core Competencies In-House**: Game design, core programming, and technical direction remain internal
2. **Scale Through Partners**: Use outsourcing to scale production capacity without permanent headcount
3. **Quality Over Cost**: Prioritize quality and reliability over lowest cost
4. **Long-Term Relationships**: Build partnerships with vendors who understand our vision
5. **Knowledge Transfer**: Ensure critical knowledge doesn't leave with vendors

### 1.3 Key Decisions Summary

| Work Stream | Decision | Rationale |
|-------------|----------|-----------|
| 3D Character Art | **Outsource** | High volume, specialized skill |
| Environment Art | **Hybrid** | Core in-house, overflow outsourced |
| Animation | **Outsource** | Specialized skill, mocap capability |
| Audio/Music | **Outsource** | Specialized studio required |
| Localization | **Outsource** | Native speakers required |
| Customer Support | **Outsource** | Scale flexibility needed |
| Backend Development | **In-House** | Core competency, security critical |
| QA Testing | **Hybrid** | Core in-house, surge outsourced |

---

## 2. Outsourcing Strategy

### 2.1 Strategic Objectives

1. **Accelerate Production**: Reduce time-to-market through parallel work streams
2. **Access Specialized Skills**: Obtain expertise not available in-house
3. **Manage Cost Structure**: Convert fixed costs to variable costs where appropriate
4. **Scale Flexibility**: Adjust capacity based on project phase
5. **Risk Distribution**: Spread production risk across multiple partners

### 2.2 Phased Approach

#### Phase 1: Pre-Production (Current)
- Establish vendor relationships
- Create style guides and specifications for vendors
- Pilot projects with 2-3 key vendors
- Refine communication and review processes

#### Phase 2: Production
- Scale up outsourcing for character art and animation
- Begin audio production with external studio
- Establish QA overflow arrangements

#### Phase 3: Soft Launch
- Engage localization vendors
- Set up customer support infrastructure
- Prepare live ops content pipeline

#### Phase 4: Global Launch & Live Ops
- Full customer support activation
- Ongoing content creation partnerships
- Seasonal event asset production

### 2.3 Capacity Planning

| Phase | In-House Team | Outsource FTE Equivalent | Ratio |
|-------|---------------|--------------------------|-------|
| Pre-Production | 4 | 2 | 67:33 |
| Alpha | 8 | 6 | 57:43 |
| Beta | 10 | 12 | 45:55 |
| Launch | 10 | 15 | 40:60 |
| Live Ops | 8 | 10 | 44:56 |

---

## 3. In-House vs Outsource Decisions

### 3.1 Decision Framework

Each work stream is evaluated on five criteria:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Strategic Importance | 30% | Core to competitive advantage |
| Skill Availability | 20% | Can we hire/retain this talent? |
| Quality Control | 20% | How critical is consistent quality? |
| Communication Overhead | 15% | Frequency of iteration required |
| Cost Efficiency | 15% | Outsource cost vs in-house |

### 3.2 Detailed Decisions

#### 3.2.1 3D Character Art

**Decision: OUTSOURCE**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 3 | 0.90 |
| Skill Availability | 2 | 0.40 |
| Quality Control | 4 | 0.80 |
| Communication Overhead | 3 | 0.45 |
| Cost Efficiency | 4 | 0.60 |
| **Total** | | **3.15** |

**Rationale:**
- High volume of characters needed (base + cosmetics)
- Specialized skill set (stylized mobile 3D)
- Style guide provides clear direction
- Well-established outsourcing market
- In-house art director maintains quality

**Scope:**
- Player character models and variants
- Premium cosmetic outfits
- NPC/monster models
- Character accessories and weapons

---

#### 3.2.2 Environment Art

**Decision: HYBRID**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 4 | 1.20 |
| Skill Availability | 3 | 0.60 |
| Quality Control | 4 | 0.80 |
| Communication Overhead | 4 | 0.60 |
| Cost Efficiency | 3 | 0.45 |
| **Total** | | **3.65** |

**Rationale:**
- Core map design requires deep understanding of gameplay
- Hero locations should be distinctive (in-house)
- Props and modular pieces can be outsourced
- Technical requirements need close integration

**In-House:**
- Map layout and blockout
- Named location hero assets
- Technical art implementation
- Lighting and atmosphere

**Outsource:**
- Environment prop kits
- Background/distant assets
- Texture creation
- LOD generation

---

#### 3.2.3 Animation

**Decision: OUTSOURCE**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 4 | 1.20 |
| Skill Availability | 2 | 0.40 |
| Quality Control | 4 | 0.80 |
| Communication Overhead | 3 | 0.45 |
| Cost Efficiency | 4 | 0.60 |
| **Total** | | **3.45** |

**Rationale:**
- Specialized skill with high barrier to entry
- Motion capture capability may be needed
- Large volume (character, ability, emote animations)
- Quality studios available with game experience

**Scope:**
- Character locomotion set
- Ability-specific animations
- Emote animations
- Victory/defeat poses
- Monster animations

---

#### 3.2.4 Audio/Music

**Decision: OUTSOURCE**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 3 | 0.90 |
| Skill Availability | 1 | 0.20 |
| Quality Control | 3 | 0.60 |
| Communication Overhead | 2 | 0.30 |
| Cost Efficiency | 5 | 0.75 |
| **Total** | | **2.75** |

**Rationale:**
- Highly specialized studio equipment required
- Project-based work suits external partners
- Pirate theme has clear reference points
- In-house audio direction, external production

**Scope:**
- Original music tracks (menu, gameplay, victory)
- Sound effects (abilities, UI, ambient)
- Voice lines (optional, localized)
- Audio implementation consulting

---

#### 3.2.5 Localization

**Decision: OUTSOURCE**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 2 | 0.60 |
| Skill Availability | 1 | 0.20 |
| Quality Control | 3 | 0.60 |
| Communication Overhead | 2 | 0.30 |
| Cost Efficiency | 5 | 0.75 |
| **Total** | | **2.45** |

**Rationale:**
- Native speakers essential for quality
- Specialized game localization expertise needed
- Well-established vendor ecosystem
- Ongoing updates require scalable solution

**Languages (Priority Order):**
1. **Tier 1 (Launch):** Spanish, Portuguese, French, German, Japanese, Korean
2. **Tier 2 (Post-Launch):** Italian, Russian, Turkish, Polish, Thai, Vietnamese
3. **Tier 3 (If metrics support):** Arabic, Hindi, Indonesian

---

#### 3.2.6 Customer Support

**Decision: OUTSOURCE**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 2 | 0.60 |
| Skill Availability | 4 | 0.80 |
| Quality Control | 3 | 0.60 |
| Communication Overhead | 2 | 0.30 |
| Cost Efficiency | 5 | 0.75 |
| **Total** | | **3.05** |

**Rationale:**
- 24/7 coverage requires significant staffing
- Scale flexibility essential (launch surge)
- Multilingual support needed
- Specialized tools and training available

**Scope:**
- Tier 1 support (FAQs, basic issues)
- Tier 2 support (account issues, bugs)
- Social media monitoring
- Review response management
- Escalation to internal Tier 3

---

#### 3.2.7 Backend Development

**Decision: IN-HOUSE**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 5 | 1.50 |
| Skill Availability | 3 | 0.60 |
| Quality Control | 5 | 1.00 |
| Communication Overhead | 5 | 0.75 |
| Cost Efficiency | 2 | 0.30 |
| **Total** | | **4.15** |

**Rationale:**
- Security critical (player data, payments)
- Requires deep integration with game systems
- Real-time performance optimization
- Long-term maintenance and iteration
- Competitive advantage in netcode quality

**Exception:**
- May engage consultants for specific expertise (anti-cheat, scaling)
- Cloud infrastructure managed services acceptable

---

#### 3.2.8 QA Testing

**Decision: HYBRID**

| Criterion | Score (1-5) | Weighted |
|-----------|-------------|----------|
| Strategic Importance | 4 | 1.20 |
| Skill Availability | 4 | 0.80 |
| Quality Control | 4 | 0.80 |
| Communication Overhead | 4 | 0.60 |
| Cost Efficiency | 3 | 0.45 |
| **Total** | | **3.85** |

**Rationale:**
- Core QA needs deep game knowledge
- Device coverage requires wide test lab
- Surge capacity needed for milestones
- Localization QA requires native testers

**In-House:**
- Test planning and strategy
- Core functional testing
- Balance testing and feedback
- Bug triage and prioritization

**Outsource:**
- Device compatibility testing
- Localization verification (LQA)
- Regression testing at scale
- Load testing support

---

## 4. Vendor Categories

### 4.1 Category Overview

| Category | Primary Vendors | Backup Vendors | Engagement Type |
|----------|-----------------|----------------|-----------------|
| 3D Character Art | 2 | 1 | Project-based |
| Animation | 1 | 1 | Project-based |
| Audio/Music | 1 | 1 | Project-based |
| Localization | 1 | 1 | Ongoing |
| Customer Support | 1 | 1 | Ongoing |
| QA Testing | 1 | 1 | Hybrid |

### 4.2 Vendor Tiers

**Tier 1 - Strategic Partners:**
- Long-term relationship
- Dedicated team or resources
- Deep understanding of project
- Preferred pricing
- Priority scheduling

**Tier 2 - Preferred Vendors:**
- Proven track record
- Quality validated
- Standard pricing
- Used for overflow/specific tasks

**Tier 3 - Approved Vendors:**
- Vetted and qualified
- Available for emergency/surge
- Project-by-project engagement

---

## 5. Vendor Selection Criteria

### 5.1 Evaluation Matrix

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Portfolio Quality | 25% | Relevant work samples |
| Mobile Experience | 20% | Track record on mobile projects |
| Pricing | 15% | Cost competitiveness |
| Capacity | 15% | Ability to meet our volume |
| Communication | 10% | Responsiveness, language |
| Security/NDA | 10% | IP protection practices |
| References | 5% | Client testimonials |

### 5.2 Minimum Requirements

All vendors must meet:

1. **Legal:**
   - Registered business entity
   - Able to sign English-language contracts
   - Accept standard NDA terms
   - No active litigation with game companies

2. **Financial:**
   - Financially stable (2+ years operation)
   - Accept NET 30 payment terms
   - Provide invoices in USD

3. **Technical:**
   - Use standard file formats (FBX, PNG, WAV)
   - Version control capability
   - Secure file transfer
   - Regular backup procedures

4. **Communication:**
   - English language proficiency
   - Minimum 4-hour timezone overlap
   - Response within 24 hours (business days)
   - Video call capability

### 5.3 Art Vendor Specific

- Portfolio demonstrates stylized mobile art
- Experience with character/environment for games
- Understanding of technical constraints (poly counts, texture sizes)
- Able to iterate based on feedback

### 5.4 Audio Vendor Specific

- Demo reel with game audio
- Original composition capability
- Sound design experience
- Middleware experience (Wwise, FMOD)

### 5.5 Localization Vendor Specific

- Game localization specialization
- Native speaker translators
- QA/LQA services included
- TMS (Translation Management System) access

### 5.6 Support Vendor Specific

- Gaming industry experience
- Multilingual capabilities
- CRM/ticketing system
- 24/7 coverage capability
- Zendesk/Helpshift experience

---

## 6. Recommended Vendors

### 6.1 3D Character Art

#### Primary: Studio A (Southeast Asia)
| Attribute | Details |
|-----------|---------|
| Location | Vietnam/Philippines |
| Team Size | 20-50 3D artists |
| Specialization | Stylized mobile game characters |
| Portfolio | [Similar mobile titles] |
| Rate | $25-40/hour or per-asset pricing |
| Capacity | 5-10 characters/month |
| Strengths | Cost-effective, high volume, proven mobile quality |
| Status | **Recommended for pilot** |

#### Secondary: Studio B (Eastern Europe)
| Attribute | Details |
|-----------|---------|
| Location | Ukraine/Poland |
| Team Size | 10-30 3D artists |
| Specialization | AAA-quality characters |
| Portfolio | [PC/Console titles] |
| Rate | $35-55/hour |
| Capacity | 3-5 characters/month |
| Strengths | Higher quality ceiling, strong iteration |
| Status | **Backup / Hero assets** |

---

### 6.2 Animation

#### Primary: Animation Studio C
| Attribute | Details |
|-----------|---------|
| Location | Canada/USA |
| Team Size | 15-30 animators |
| Specialization | Game animation, mocap |
| Portfolio | [Action game titles] |
| Rate | $45-70/hour |
| Capacity | Full animation set in 2-3 months |
| Strengths | Quality, mocap facility, game experience |
| Status | **Recommended** |

---

### 6.3 Audio/Music

#### Primary: Audio House D
| Attribute | Details |
|-----------|---------|
| Location | UK/USA |
| Team Size | 5-15 composers/sound designers |
| Specialization | Game audio, orchestral |
| Portfolio | [Mobile/indie game soundtracks] |
| Rate | Project-based ($15K-50K for full audio) |
| Capacity | 3-4 month turnaround |
| Strengths | Pirate/adventure genre experience |
| Status | **Recommended** |

---

### 6.4 Localization

#### Primary: Localization Partner E
| Attribute | Details |
|-----------|---------|
| Location | Global (offices in EU, Asia) |
| Team Size | 500+ linguists |
| Specialization | Game localization |
| Portfolio | [Major mobile games] |
| Rate | $0.10-0.20/word depending on language |
| Capacity | 6 languages in 2 weeks |
| Strengths | Gaming expertise, fast turnaround, LQA included |
| Status | **Recommended** |

---

### 6.5 Customer Support

#### Primary: Support Provider F
| Attribute | Details |
|-----------|---------|
| Location | Philippines/India |
| Team Size | Scalable (5-50 agents) |
| Specialization | Gaming support |
| Portfolio | [Mobile game publishers] |
| Rate | $8-15/hour per agent |
| Capacity | 24/7 coverage, 8 languages |
| Strengths | Gaming experience, multilingual, flexible scaling |
| Status | **Recommended for soft launch** |

---

### 6.6 QA Testing

#### Primary: QA Partner G
| Attribute | Details |
|-----------|---------|
| Location | Global |
| Team Size | Scalable |
| Specialization | Mobile game testing |
| Portfolio | [Major publishers] |
| Rate | $20-35/hour |
| Capacity | Device lab (200+ devices), 50+ testers |
| Strengths | Device coverage, localization QA |
| Status | **Recommended** |

---

## 7. Contract Framework

### 7.1 Contract Types

| Type | Use Case | Duration | Payment |
|------|----------|----------|---------|
| Master Services Agreement (MSA) | Long-term partners | 1-2 years | NET 30 |
| Statement of Work (SOW) | Specific projects | Per project | Milestone-based |
| Work Order | Quick tasks under MSA | Per task | On completion |
| Retainer | Ongoing services | Monthly | Monthly |

### 7.2 Standard Contract Terms

#### 7.2.1 Intellectual Property

- All work product is **work-for-hire**
- Full IP transfer upon payment
- Vendor retains no rights to assets
- Portfolio use requires written approval
- Source files delivered with final assets

#### 7.2.2 Confidentiality (NDA)

- **Duration:** 3 years from disclosure
- **Scope:** All project information, assets, designs
- **Exclusions:** Public information, prior knowledge
- **Return/Destroy:** Upon project end or request
- **Breach Remedy:** Injunctive relief + damages

#### 7.2.3 Payment Terms

| Milestone | Percentage | Trigger |
|-----------|------------|---------|
| Project Kickoff | 20% | Contract signed |
| First Deliverable | 30% | Approved WIP |
| Final Delivery | 40% | Assets accepted |
| Retention | 10% | 30 days after acceptance |

#### 7.2.4 Acceptance Criteria

- Deliverables reviewed within 5 business days
- Written feedback with specific issues
- 2 revision rounds included in base price
- Additional revisions billed separately
- Final acceptance in writing

#### 7.2.5 Termination

- **For Convenience:** 30 days notice, pay for completed work
- **For Cause:** Immediate, written notice, cure period (15 days)
- **Effect:** All WIP and source files delivered

### 7.3 NDA Template (Summary)

```
MUTUAL NON-DISCLOSURE AGREEMENT

Parties: [Company] and [Vendor]

Purpose: Evaluation and performance of game development services

Confidential Information includes:
- Game designs, mechanics, features
- Art assets, style guides, references
- Technical specifications
- Business plans, pricing, schedules
- Source code, tools, pipelines

Obligations:
- Use only for intended purpose
- Protect with reasonable care
- Limit access to need-to-know
- Not disclose without consent

Duration: 3 years from disclosure date

Exclusions:
- Publicly available information
- Independently developed
- Lawfully obtained from third party
- Required by law (with notice)

Remedies: Injunctive relief and damages for breach
```

---

## 8. Vendor Onboarding Process

### 8.1 Onboarding Timeline

| Day | Activity | Owner |
|-----|----------|-------|
| 0 | Contract signed | Producer |
| 1-2 | NDA executed | Legal |
| 3-5 | Access provisioned | IT |
| 3-5 | Documentation shared | Art Director |
| 5-7 | Kickoff call | Project Lead |
| 7-14 | Pilot task assigned | Art Director |
| 14-21 | Pilot review | Art Director |
| 21+ | Full production begins | Producer |

### 8.2 Documentation Package

Every vendor receives:

1. **Project Overview**
   - Game concept summary
   - Target platforms and audience
   - Competitive context
   - Timeline and milestones

2. **Style Guides**
   - Visual Style Guide (VISUAL_STYLE_GUIDE.md)
   - Technical Art Specifications (TECHNICAL_ART_SPECIFICATIONS.md)
   - Audio Style Guide (if applicable)
   - UI Guidelines

3. **Technical Specifications**
   - Polygon budgets
   - Texture specifications
   - File naming conventions
   - Delivery formats
   - Directory structure

4. **Reference Materials**
   - Concept art
   - Example assets
   - Reference images
   - Moodboards

5. **Process Documents**
   - Feedback workflow
   - Revision process
   - Approval chain
   - Communication channels

### 8.3 Access Provisioning

| System | Access Level | Vendor Types |
|--------|--------------|--------------|
| Cloud Storage (Google Drive/Dropbox) | Folder-specific | All |
| Project Management (Jira) | View + Comment | Art, Animation |
| Communication (Slack) | Dedicated channel | All |
| Asset Repository | Upload only | Art, Animation, Audio |
| Documentation (Notion) | Read-only | All |
| Build Access | Not provided | None |

### 8.4 Kickoff Call Agenda

1. **Introductions** (10 min)
   - Team members and roles
   - Primary contacts

2. **Project Overview** (15 min)
   - Game concept and vision
   - Target quality and style
   - Timeline and milestones

3. **Scope Review** (20 min)
   - Specific deliverables
   - Quantities and schedule
   - Priorities

4. **Process Walkthrough** (15 min)
   - Communication channels
   - Feedback and revision flow
   - Approval process

5. **Technical Requirements** (15 min)
   - Specifications review
   - File formats and naming
   - Delivery process

6. **Q&A** (15 min)
   - Open questions
   - Clarifications

### 8.5 Pilot Project

All new vendors complete a pilot before full engagement:

**Pilot Scope:**
- 1-2 assets representative of full scope
- Full production process followed
- Timeline: 1-2 weeks

**Evaluation Criteria:**
- Quality meets standards
- Technical specs followed
- Communication responsive
- Revisions handled professionally
- Timeline met

**Pilot Outcome:**
- **Pass:** Proceed to full production
- **Conditional:** Address issues, repeat pilot
- **Fail:** End engagement, provide feedback

---

## 9. Quality Control Process

### 9.1 Quality Standards

| Asset Type | Quality Gate | Reviewer |
|------------|--------------|----------|
| Character Model | Silhouette, proportions, topology | Art Director |
| Texture | Resolution, tiling, compression | Tech Artist |
| Animation | Motion, timing, blending | Animation Lead |
| Audio | Mix, clarity, style fit | Audio Director |
| Localization | Accuracy, tone, length | Localization PM |

### 9.2 Review Workflow

```
[Vendor Submission]
       |
       v
[Initial QC Check] -- Fail --> [Return with notes]
       |
      Pass
       |
       v
[Art Director Review] -- Feedback --> [Revision Request]
       |
      Approve
       |
       v
[Technical Validation] -- Fail --> [Technical feedback]
       |
      Pass
       |
       v
[Final Approval]
       |
       v
[Asset Integration]
```

### 9.3 Feedback Guidelines

**Effective Feedback:**
- Specific and actionable
- Reference examples or style guide
- Prioritize issues (must fix / nice to have)
- Explain reasoning when helpful
- Provide visual markup when possible

**Feedback Template:**
```
Asset: [Name]
Overall: [Approve / Minor Revisions / Major Revisions]

Must Fix:
1. [Specific issue] - [How to fix] - [Reference]
2. ...

Nice to Have:
1. [Suggestion] - [Reasoning]

Questions:
1. [Clarification needed]
```

### 9.4 Revision Policy

| Round | Included | Notes |
|-------|----------|-------|
| Round 1 | Yes | Initial feedback |
| Round 2 | Yes | Follow-up corrections |
| Round 3+ | No | Billed at hourly rate |

**Exception:** Technical issues from our specs are not counted as revisions.

### 9.5 Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| First-pass approval rate | >70% | Assets approved on first submission |
| Average revisions | <1.5 | Revision rounds per asset |
| Technical compliance | >95% | Assets meeting spec on delivery |
| Schedule adherence | >90% | Deliveries on or before deadline |
| Rework rate | <5% | Assets requiring redo after approval |

### 9.6 Escalation Process

| Level | Trigger | Action | Owner |
|-------|---------|--------|-------|
| 1 | Minor quality issue | Feedback via normal channel | Art Director |
| 2 | Repeated quality issues | Call with vendor lead | Producer |
| 3 | Consistent underperformance | Formal warning letter | Director |
| 4 | Contract breach / severe issues | Contract review/termination | Legal |

---

## 10. Communication Protocols

### 10.1 Communication Channels

| Channel | Use Case | Response Time |
|---------|----------|---------------|
| Slack (vendor channel) | Daily communication | 4 hours |
| Email | Formal correspondence | 24 hours |
| Video Call (Zoom/Meet) | Reviews, kickoffs | Scheduled |
| Jira Comments | Task-specific feedback | 24 hours |
| Emergency Phone | Critical issues | Immediate |

### 10.2 Meeting Cadence

| Meeting | Frequency | Attendees | Duration |
|---------|-----------|-----------|----------|
| Status Sync | Weekly | PM, Vendor Lead | 30 min |
| Asset Review | 2x/week | Art Director, Vendor Artists | 60 min |
| Sprint Planning | Bi-weekly | Full team | 60 min |
| Retrospective | Monthly | PM, Leads | 45 min |

### 10.3 Status Reporting

Vendors provide weekly status reports:

```
WEEKLY STATUS REPORT

Week of: [Date]
Vendor: [Name]
Project: Plunderstorm Mobile

COMPLETED THIS WEEK:
- [Asset/Task] - [Status: Delivered/Approved]
- ...

IN PROGRESS:
- [Asset/Task] - [% Complete] - [ETA]
- ...

BLOCKERS:
- [Issue] - [Impact] - [Need from us]

NEXT WEEK PLAN:
- [Task] - [Target completion]
- ...

METRICS:
- Assets delivered: X
- First-pass approval rate: X%
- On-schedule: Yes/No
```

### 10.4 Timezone Management

| Region | Working Hours (UTC) | Overlap with PST |
|--------|---------------------|------------------|
| Southeast Asia | UTC+7 to +8 | 4-6 hours (evening PST) |
| Eastern Europe | UTC+2 to +3 | 5-7 hours (morning PST) |
| North America | UTC-8 to -5 | Full overlap |

**Best Practices:**
- Schedule calls during overlap hours
- Use async communication (Slack, Loom) for non-urgent items
- Set clear deadlines with timezone specified
- Respect vendor working hours

---

## 11. Budget Allocation

### 11.1 Outsourcing Budget by Phase

| Phase | Duration | Budget | % of Total |
|-------|----------|--------|------------|
| Pre-Production | 2 months | $15,000 | 5% |
| Alpha | 4 months | $60,000 | 20% |
| Beta | 4 months | $90,000 | 30% |
| Launch | 2 months | $45,000 | 15% |
| Live Ops (Year 1) | 12 months | $90,000 | 30% |
| **Total** | | **$300,000** | 100% |

### 11.2 Budget by Category

| Category | Budget | % of Outsource Budget |
|----------|--------|----------------------|
| 3D Character Art | $100,000 | 33% |
| Animation | $60,000 | 20% |
| Audio/Music | $40,000 | 13% |
| Localization | $35,000 | 12% |
| Customer Support | $40,000 | 13% |
| QA Testing | $25,000 | 9% |
| **Total** | **$300,000** | 100% |

### 11.3 Cost per Asset (Estimates)

| Asset Type | Low | Medium | High |
|------------|-----|--------|------|
| Player Character (full) | $1,500 | $2,500 | $4,000 |
| Cosmetic Outfit | $800 | $1,200 | $2,000 |
| NPC/Monster | $1,000 | $1,800 | $3,000 |
| Environment Prop | $100 | $300 | $600 |
| Animation (per clip) | $200 | $400 | $800 |
| Music Track (per minute) | $500 | $1,000 | $2,000 |
| SFX (per sound) | $20 | $50 | $150 |
| Localization (per word) | $0.10 | $0.15 | $0.20 |

### 11.4 Budget Tracking

| Metric | Target | Frequency |
|--------|--------|-----------|
| Budget variance | <10% | Monthly |
| Cost per asset | Within estimates | Per delivery |
| Invoice accuracy | 100% | Per invoice |
| Payment on time | 100% | Per invoice |

---

## 12. Risk Management

### 12.1 Outsourcing Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Vendor capacity issues | Medium | High | Multiple vendors, early booking |
| Quality below standards | Medium | High | Pilot projects, clear specs |
| Communication breakdown | Low | Medium | Dedicated channels, regular syncs |
| IP leak/breach | Low | High | Strong NDAs, access controls |
| Vendor goes out of business | Low | High | Backup vendors, staged payments |
| Cost overruns | Medium | Medium | Fixed pricing where possible |
| Schedule delays | Medium | High | Buffer time, parallel work |
| Cultural/timezone friction | Low | Low | Overlap hours, async tools |

### 12.2 Mitigation Strategies

#### Vendor Dependency
- Maintain 2+ qualified vendors per category
- Don't exceed 50% of any vendor's capacity
- Keep documentation for vendor transition
- Regular backup vendor check-ins

#### Quality Issues
- Mandatory pilot before production
- Clear style guides and specifications
- Regular review cadence
- Escalation process defined

#### IP Protection
- Comprehensive NDAs
- Limited access (need-to-know)
- Watermarked WIP assets
- Audit rights in contract

#### Schedule Risks
- 15-20% buffer in schedules
- Parallel workstreams
- Early escalation triggers
- Emergency vendor list

### 12.3 Contingency Plans

| Scenario | Trigger | Response |
|----------|---------|----------|
| Primary vendor unavailable | Capacity full or quality fail | Activate backup vendor |
| Budget overrun >20% | Monthly review | Scope reduction, renegotiation |
| Critical deadline at risk | 2 weeks before due | Surge resources, overtime approval |
| Security breach | Any IP leak | Legal action, vendor termination |

---

## 13. Appendices

### Appendix A: Vendor Evaluation Scorecard

```
VENDOR EVALUATION SCORECARD

Vendor Name: ____________________
Category: ____________________
Evaluator: ____________________
Date: ____________________

SCORING (1-5, 5 = Best)

Portfolio Quality (25%):
[ ] Relevant to our style: ___
[ ] Technical quality: ___
[ ] Variety/range: ___
Weighted Score: ___

Mobile Experience (20%):
[ ] Previous mobile projects: ___
[ ] Understanding of constraints: ___
Weighted Score: ___

Pricing (15%):
[ ] Competitive rates: ___
[ ] Transparent pricing: ___
Weighted Score: ___

Capacity (15%):
[ ] Team size: ___
[ ] Availability: ___
Weighted Score: ___

Communication (10%):
[ ] Response time: ___
[ ] English proficiency: ___
Weighted Score: ___

Security/NDA (10%):
[ ] NDA acceptance: ___
[ ] Security practices: ___
Weighted Score: ___

References (5%):
[ ] Client feedback: ___
Weighted Score: ___

TOTAL SCORE: ___ / 5.00

RECOMMENDATION:
[ ] Proceed with pilot
[ ] Need more information
[ ] Do not proceed

Notes: ____________________
```

### Appendix B: Asset Delivery Checklist

```
ASSET DELIVERY CHECKLIST

Asset Name: ____________________
Vendor: ____________________
Delivery Date: ____________________

TECHNICAL REQUIREMENTS:
[ ] Correct file format
[ ] Within poly budget
[ ] Correct texture sizes
[ ] Proper naming convention
[ ] Required LODs included
[ ] Source files provided

QUALITY REQUIREMENTS:
[ ] Matches style guide
[ ] Proportions correct
[ ] Clean topology
[ ] Proper UV layout
[ ] Material setup correct

DOCUMENTATION:
[ ] Asset spec sheet
[ ] Any notes/callouts
[ ] Preview renders

APPROVED BY: ____________________
DATE: ____________________
```

### Appendix C: Vendor Contact Template

```
VENDOR CONTACT INFORMATION

Company Name: ____________________
Primary Contact: ____________________
Role: ____________________
Email: ____________________
Phone: ____________________
Timezone: ____________________

Backup Contact: ____________________
Role: ____________________
Email: ____________________

Contract Status: [ ] Active [ ] Pending [ ] Expired
Contract End Date: ____________________

Services Provided:
[ ] 3D Art
[ ] Animation
[ ] Audio
[ ] Localization
[ ] Support
[ ] QA

Tier: [ ] Strategic [ ] Preferred [ ] Approved

Notes: ____________________
```

### Appendix D: Key Contacts

| Role | Name | Email | Responsibility |
|------|------|-------|----------------|
| Producer | [TBD] | [TBD] | Vendor relationships |
| Art Director | [TBD] | [TBD] | Art quality approval |
| Tech Artist | [TBD] | [TBD] | Technical validation |
| Audio Director | [TBD] | [TBD] | Audio approval |
| Localization PM | [TBD] | [TBD] | Localization oversight |
| Legal | [TBD] | [TBD] | Contracts/NDA |

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Producer | _____________ | _____________ | ___/___/___ |
| Director | _____________ | _____________ | ___/___/___ |
| Finance | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial document |

---

*This document is confidential and intended for internal use only.*
