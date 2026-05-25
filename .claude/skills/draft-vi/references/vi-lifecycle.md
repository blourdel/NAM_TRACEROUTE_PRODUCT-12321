# VI Lifecycle Reference

Understanding VI states, required activities at each stage, and what it takes to progress.

## Table of Contents

1. [VI States Overview](#vi-states-overview)
2. [State Details](#state-details)
   - [Open](#open)
   - [Problem Stated](#problem-stated)
   - [Use Case Defined](#use-case-defined)
   - [Ready for Implementation](#ready-for-implementation)
   - [Implementation](#implementation)
   - [Release Preparation](#release-preparation)
   - [Post GA](#post-ga)
   - [Closed](#closed)
3. [Definition of Ready (DoR)](#definition-of-ready-dor)
4. [Definition of Done (DoD)](#definition-of-done-dod)
5. [Quick Reference Matrix](#quick-reference-matrix)
6. [Common Questions](#common-questions)
7. [Tips](#tips)

---

## VI States Overview

```
Open → Problem Stated → Use Case Defined → Ready for Implementation 
→ Implementation → Release Preparation → Post GA → Closed
```

---

## State Details

### Open

**Purpose:** Understanding the problem and value

**Required:**
- Short Abstract / Blogline
- Target Audience (who benefits)
- WHY (problem context)

**Accountable:** PM or PA

**Best Practice:** Focus on WHY first. Do kick-off when ready to share.

**Exit:** Description sufficient and VI prioritized

---

### Problem Stated

**Purpose:** Define value, customer zero, and requirements

**Required:**
- ✅ All from "Open"
- Customer Zero (who validates this)
- Functional Requirements / Solution Journey
- Non-Functional Requirements
- Target end date / Fix version
- Assignee (PM/PA)

**Optional but Recommended:**
- Enablement Requirements
- Cost Analysis
- Dependencies identified

**Accountable:** PM or PA

**Activities:**
- Add Value and Customer Zero to VI
- Provide Functional Requirements/Solution Journey (with Design Lead)
- Assign Product Engineer for execution (Delivery Lead)
- Assign Product Architect or set to No-PA (Lead Product Architect)
- Provide Non-Functional Requirements (Product Architect)
- Assign and onboard Product Experience Designer (Design Lead)
- Start UX discovery phase (Product Experience Designer)
- Ensure Definition of Ready fulfilled
- Align on discovery outcomes, begin definition phase
- Provide estimations (t-shirt sizing), link dependencies
- Create Launch Coordination Checklist (Product Engineer)
- Re-evaluate VI if deviations occur (Capability Lead)

**UX Discovery Phase:**
- Foundational research, use cases, challenges, user journeys

**UX Definition Phase:**
- Research synthesis, personas, HMW questions, user stories, customer journey mapping

**Exit Criteria:**
- Problem and value clear
- Customer Zero identified
- Initial requirements drafted
- VI prioritized on backlog
- UX discussions started (if needed)
- DoR complete

---

### Use Case Defined

**Purpose:** Deeper understanding of use cases and solution

**Required:**
- ✅ All from "Problem Stated"
- End-to-End Acceptance Criteria
- Customer Zero Planning
- E2E Demo plan (how to prove it's done)
- Out of Scope (explicit boundaries)
- Success Metrics
- Cost Analysis and Effort estimate

**For UX-heavy VIs:**
- User needs, WOW factors, Pain points
- Wire flows, proof of concept

**Accountable:** PM or PA

**Activities:**
- PE accepts VI (takes accountability for next three phases)
- Explore solution space through ideation and validation (Product Experience Designer)
- Alignment and confirmation of UX Ideation with stakeholders (Product Manager)
- Create high-fidelity mockups for implementation handover (Product Experience Designer)
- Provide final sign-off on design quality and Design System consistency (Design Lead)
- Create architectural concept suited for engineering (Product Architect)
- Dependencies linked in Jira
- Effort estimated (t-shirt sizing)

**UX Ideation Phase:**
- Brainstorming, user flows, sketching/wireframing, AppIcons

**UX Solution Phase:**
- Prototypes, user testing, design reviews, design documentation, handoff meeting
- Use Experience Checklist to ensure LCC standards

**Exit Criteria:**
- Acceptance criteria defined
- Demo plan clear
- Effort estimated
- Dependencies identified
- High-fidelity mockups created (if UX-heavy)
- PE accepted VI

---

### Ready for Implementation

**Purpose:** Create Dev Epics and execution plan

**Required:**
- ✅ All from "Use Case Defined"
- Execution Assignee (PE) set
- Dev Epics created
- Execution plan with landing zones
- Sprint set

**For UX-heavy VIs:**
- Click prototypes
- Mockups for engineering
- UX metrics defined
- Flow charts

**Accountable:** PE (Product Engineer)

**Activities:**
- Create Dev Epics with stakeholders
- Prepare execution plan with refined landing zones
- Usability tests completed (if UX-heavy)
- Design QA done

**Suggested Meetings:**
- Dev-Epic break-down
- Dev Epic refinement
- Capability planning
- Pre-planning

**Exit Criteria:**
- Rough execution plan defined
- First Dev Epic starts

---

### Implementation

**Purpose:** Build the solution

**Accountable:** PE

**Activities:**
- Constant sync between teams and PM on deviations (Team, PO, PM, PA)
- PM/PA accepts or rejects per Acceptance Criteria
- Capability Lead re-evaluates if deviations occur

**Suggested Meetings:**
- Design QA
- Value Increment Review

**Exit Criteria:**
- All Dev Epics done
- Definition of Done fulfilled
- All ACs met

---

### Release Preparation

**Purpose:** Hardening, documentation, launch coordination

**Accountable:** PE

**Required:**
- Release Notes (PM provides)
- Launch Coordination Checklist signed off
- Documentation provided
- Troubleshooting guides
- Demo conducted
- Hardening complete
- DoD fulfilled

**Activities:**
- Ensure Launch Coordination Checklist signed off, documentation, troubleshooting guides, demo, hardening, DoD (Product Engineer)
- Conduct Preview: Internal Adoption/Private Preview/Public Preview (Product Manager)

**Suggested Meetings:**
- Product Demo (RnD audience)
- Product Preview

**Exit:** Launch checklist complete

---

### Post GA

**Purpose:** Go-to-market and enablement

**Accountable:** PM or PA (PE re-assigns back)

**Activities:**
- Re-assign VI to PM or PA (Product Engineer)
- Kick-off enablement activities: GTM, Sales, Marketing (Product Manager or Product Architect)
- Ensure VI status reflected on roadmap (Delivery Lead)
- Metrics dashboard implemented

**Exit:** GTM activities complete

---

### Closed

**Purpose:** Retrospective and evaluation

**Accountable:** PM or PA

**Activities:**
- Ensure enablement activities performed (Product Manager)
- Close all Dev Epics and complete Post-GA activities: feature flags, etc. (Product Engineer)
- Inspect added value per success metrics; decide to tweak or remove if landing goal not reached (Product Manager or Product Architect)
- (Optional) VI Retrospective (Org Lead)

**Suggested Meetings:**
- Capability retrospective

**Exit:** All activities fulfilled, success metrics evaluated

---

## Definition of Ready (DoR)

Before moving to "Use Case Defined":
- [ ] Problem and value clear
- [ ] Target audience identified
- [ ] Customer Zero identified
- [ ] Initial functional requirements
- [ ] Acceptance criteria drafted
- [ ] Success criteria defined
- [ ] Dependencies identified
- [ ] Out of scope stated
- [ ] VI estimated (t-shirt)
- [ ] Stakeholders informed

---

## Definition of Done (DoD)

### Business Value
- [ ] VI demo done
- [ ] VI signed off for release

### Acceptance Criteria
- [ ] All ACs fulfilled
- [ ] All Dev Epics closed
- [ ] No major bugs

### Quality
- [ ] UX passed
- [ ] E2E tests passed
- [ ] Security tests passed (Pentest if applicable)
- [ ] NFRs fulfilled

### Documentation
- [ ] Technical docs done
- [ ] Devportal docs done (teams handle)
- [ ] Customer docs done (SUS)
- [ ] SRE docs done (runbooks, emergency disable)

### Deployment
- [ ] Launch Coordination Checklist done
- [ ] Deployed to target environment
- [ ] Feature flags set

---

## Quick Reference Matrix

| State | PM/PA Focus | PE Focus | UX Focus | Exit Gate |
|-------|-------------|----------|----------|-----------|
| Open | WHY, Target Audience | - | - | Prioritized |
| Problem Stated | Requirements, Customer Zero | Estimate, dependencies | Discovery, definition | DoR complete |
| Use Case Defined | ACs, Metrics | Accept VI | Mockups, handoff | PE accepted |
| Ready for Impl | - | Dev Epics, plan | Design QA | Epics created |
| Implementation | Accept/Reject | Build | Design QA | DoD fulfilled |
| Release Prep | Release notes, preview | Hardening, checklist | - | Launch checklist |
| Post GA | Enablement (GTM, Sales) | Re-assign VI | - | GTM complete |
| Closed | Evaluate metrics | Close epics | - | All done |

---

## Common Questions

**Q: Can I skip a state?**
A: No. Each state has required artifacts. Skipping leads to gaps.

**Q: When does PE take ownership?**
A: At "Use Case Defined" → PE accepts VI and owns through Release Prep.

**Q: When do I need UX involved?**
A: Start discussions at "Problem Stated". UX delivers mockups at "Use Case Defined".

**Q: What if requirements change during Implementation?**
A: Capability Lead re-evaluates VI. May need to adjust scope/timeline.

**Q: When should success metrics be measured?**
A: Post-GA, typically 3-6 months after release.

**Q: What's the difference between DoR and DoD?**
A: DoR = ready to implement (requirements clear). DoD = ready to release (built and tested).

---

## Tips

**Moving states:**
- Don't rush. Each state builds on previous.
- Missing artifacts → delays later
- Better to pause and complete than forge ahead

**Collaboration:**
- PM/PA own early states (problem definition)
- PE owns middle states (execution)
- PM/PA own late states (GTM, metrics)

**Red flags:**
- ⚠️ "Use Case Defined" without ACs → Will delay implementation
- ⚠️ "Ready for Impl" without effort estimate → Can't plan
- ⚠️ "Implementation" without Customer Zero → Who validates?
- ⚠️ Moving states without exit criteria met → Gaps accumulate

**Use Flow metrics:**
- Limit WIP (Work in Progress)
- Thorough blockers/dependencies addressed = smoother implementation
- Track cycle time per state to identify bottlenecks
