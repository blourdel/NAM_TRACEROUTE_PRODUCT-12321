# VI Examples Reference

Real VIs demonstrating different patterns. Use these as inspiration when creating new VIs.

## Table of Contents

1. [Customer-Facing Feature with Workflow](#customer-facing-feature-with-workflow)
2. [Technical Enabler (Simple)](#technical-enabler-simple)
3. [Technical Enabler (Infrastructure)](#technical-enabler-infrastructure)
4. [UX-Heavy Feature](#ux-heavy-feature)
5. [Platform/Data Integrity](#platformdata-integrity)
6. [Infrastructure Modernization](#infrastructure-modernization)
7. [Pattern Summary](#pattern-summary)
8. [Selection Guide](#selection-guide)

---

## Customer-Facing Feature with Workflow

**PRODUCT-15500 - Phase 3 Upgrade readiness flow**
- Status: Implementation (Jun 2026)
- Location: `../imports/jira-PRODUCT-15500-2026-02-18-0334.md`

**Pattern:**
- Strong problem statement (customer pain points)
- Detailed solution concept with numbered workflow steps
- ACs grouped by category (Notification, Guides, Execution, etc.)
- Tracking requirements for adoption metrics
- Clear stakeholder communication needs

**Key sections:**
```
## Problem Statement
Customers unable to benefit from 3rd Gen without structured upgrade workflow...

## Proposed Solution
### What It Is
A structured upgrade workflow...

### Concept
1. Upgrade Notification
2. Upgrade Guides and Dashboards
3. Readiness Checks
[...]

## Acceptance Criteria
1. **Upgrade Notification**: Admins receive notifications...
2. **Upgrade Guides**: Documentation and dashboards available...
```

**Use when:** Building customer-facing features with multi-step workflows

---

## Technical Enabler (Simple)

**PRODUCT-16056 - Migrate AppShell app from Hermes**
- Status: Usecases defined
- Location: `../imports/jira-PRODUCT-16056-2026-02-18-0334.md`

**Pattern:**
- Concise problem (outdated deployment method)
- Clear solution with chosen variant
- Simple numbered ACs (3 items)
- Out of scope explicitly stated
- Open questions tracked

**Key sections:**
```
## Summary
AppShell uses outdated deployment method - Hermes...
Option 1: AppShell becomes Dynatrace App (chosen)

## Acceptance Criteria
1. AppShell becomes pre-install app
2. No longer publishes Helm Charts
3. Team updates release schedule to...

## Not in Scope
1. Removing Chrome Extension requirement
2. PR preview experience
```

**Use when:** Technical refactoring, service reorganization, architecture improvements

---

## Technical Enabler (Infrastructure)

**PRODUCT-16019 - Generalize Platform Search to AppShell service**
- Status: Usecases defined
- Location: `../imports/jira-PRODUCT-16019-2026-02-18-0334.md`

**Pattern:**
- Problem focused on organizational/technical constraints
- Solution includes architectural considerations
- Technical considerations section for API compatibility
- Links to what it enables

**Key sections:**
```
## Problem Statement
Division and purpose don't align with goals. Organizationally limited...

## Technical Considerations
Search API can be re-wired to stay under same name, avoid breaking changes:
[API config link]
```

**Use when:** Backend services, API reorganization, architectural prerequisites

---

## UX-Heavy Feature

**PRODUCT-11388 - Improved addon intents**
- Status: Ready for Implementation
- Location: `../imports/jira-PRODUCT-11388-2026-02-18-0334.md`

**Pattern:**
- Detailed "What It Is" with sub-features
- Evidence section with Slack discussions
- Open questions tracked
- Design mockups linked
- Tracking for adoption metrics

**Key sections:**
```
## What It Is
* Support sheets
  * Add bottom sheet view...
* Modal anatomy
  * Flexible width/height
  * Background, header customization
* Improve documentation
  * Developer docs with examples
  * Design documentation

## Evidence
[Slack discussion links]

## Open Questions
- How to show which app modal comes from?
- Implications of bottom sheet as maximized view?

## Designs
[Figma link]
```

**Use when:** Heavy UX work, modal/overlay improvements, flexible UI components

---

## Platform/Data Integrity

**PRODUCT-16232 - Segments definition sync and storage drift**
- Status: Open
- Location: `../imports/jira-PRODUCT-16232-2026-02-18-0334.md`

**Pattern:**
- Problem broken down into specific scenarios
- Each scenario: What happens, Where it shows up, User symptoms
- Functional requirements as bullet list
- Use cases separate from requirements
- NFRs with specific attributes (Reliability, Performance, Usability)

**Key sections:**
```
## Problem Statement
No single source of truth for segment selection...creates drift...

### Where It Hurts
#### 1) Orphaned References
* What happens: Segment deleted, stored reference can't resolve
* Where it shows up: Recents, Dashboards, Notebooks, URLs
* User symptoms: Broken entries, broken apply fails

#### 2) Stale Selections
[...]

## Use Cases
1. Recents/pinned cleanup: When segment deleted...
2. Dynamic values stay current: Reusing selection reflects...

## Non-Functional Requirements
* Reliability: No page-breaking failures
* Performance: Minimal latency, measurable
* Usability: Clear, actionable error states
```

**Use when:** Data consistency, state management, persistence problems

---

## Infrastructure Modernization

**PRODUCT-15256 - New Dashboard layout engine**
- Status: Implementation (Mar 2026)
- Location: `../imports/jira-PRODUCT-15256-2026-02-18-0334.md`

**Pattern:**
- WHY section with external evidence (Kibana blog)
- Must-haves split into categories (Core, Performance, DX)
- Each category with bullet list of requirements
- "What Is It Not" section for explicit exclusions
- Status updates showing progress

**Key sections:**
```
## Why
Existing React Grid no longer maintained, technically outdated.
Kibana assessment: [external link]

## What
### Must-Haves (Enablement)
#### 1. Core Features
* Drag-and-Drop: Smooth, grid-based
* Responsive Design: Breakpoint support
* Dynamic Resizing: Real-time updates
[...]

#### 2. Performance
* Optimized Rendering: minimal re-renders
* Lightweight: Small bundle
[...]

## What Is It Not?
1. Layout options: distribute (same size tiles)
2. Grouping => descoped (link to future VI)
```

**Use when:** Library replacement, modernization, technical debt with UX impact

---

## Pattern Summary

| Pattern | Problem Style | Solution Style | AC Style | Best For |
|---------|--------------|----------------|----------|----------|
| Customer Workflow | Pain-focused, detailed | Multi-step concept | Grouped by feature | End-user features |
| Tech Enabler (Simple) | Technical constraint | Chosen option | Numbered list (2-5) | Refactoring, migration |
| Tech Enabler (Infra) | Architectural | Technical approach | System-focused | Backend services |
| UX-Heavy | User needs + evidence | Feature breakdown | By component | UI improvements |
| Platform/Data | Scenario breakdown | Requirements list | Specific conditions | Data integrity |
| Modernization | External validation | Must-haves by category | Functional + NFRs | Library updates |

---

## Selection Guide

**Choose Customer Workflow when:**
- Building for end users
- Multi-step process
- Need GTM/enablement

**Choose Tech Enabler (Simple) when:**
- Internal refactoring
- Clear prerequisite
- Limited scope

**Choose UX-Heavy when:**
- Design-led work
- Multiple UI components
- Need user validation

**Choose Platform/Data when:**
- System consistency
- Data integrity
- Error handling critical

**Choose Modernization when:**
- Replacing outdated tech
- NFRs equally important
- Performance-critical
