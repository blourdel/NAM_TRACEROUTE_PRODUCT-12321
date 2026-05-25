# Acceptance Criteria Patterns

Patterns and examples for writing specific, testable Acceptance Criteria (ACs).

## Table of Contents

1. [Core Principles](#core-principles)
2. [Pattern 1: Simple Numbered List](#pattern-1-simple-numbered-list)
3. [Pattern 2: Grouped by Feature/Category](#pattern-2-grouped-by-featurecategory)
4. [Pattern 3: Condition-Based (Consumer Apps)](#pattern-3-condition-based-consumer-apps)
5. [Pattern 4: Must-Have Requirements by Category](#pattern-4-must-have-requirements-by-category)
6. [Writing Rules](#writing-rules)
7. [Common Patterns](#common-patterns)
8. [Must vs Should vs Can](#must-vs-should-vs-can)
9. [Checklist for Review](#checklist-for-review)
10. [Common Mistakes](#common-mistakes)
11. [Templates by VI Type](#templates-by-vi-type)

---

## Core Principles

Acceptance Criteria (AC) define testable conditions for the VI to be considered "done". They must be:

1. **Specific** - No vague terms ("works well", "is fast")
2. **Testable** - Can verify true/false
3. **Complete** - Covers all functional + key NFRs
4. **Independent** - Each AC stands alone

---

## Pattern 1: Simple Numbered List

**Use when:** Technical enabler, clear scope, 2-5 items

**Example (from PRODUCT-16056):**
```
h2. Acceptance Criteria

# AppShell becomes a pre-install app
# AppShell no longer publishes Helm Charts
# The team updates agreed release schedule to:
## Manually or automatically release AppShell every sprint
## Attach sprint version to the AppShell version
## Release a backport to any dev/hard/production version
```

**Characteristics:**
- Numbered list (1, 2, 3...)
- Each AC is a complete statement
- Sub-items for details (##)
- No categories needed

---

## Pattern 2: Grouped by Feature/Category

**Use when:** Customer-facing, multiple features, complex VI

**Example (from PRODUCT-15500):**
```
h2. Acceptance Criteria

# *Upgrade Notification*: Admins in Phase 1 and Phase 2 tenants receive 
  notifications about the upgrade option.
  
# *Upgrade Guides*: Detailed documentation and ready-made dashboards are 
  available to assist with the migration.
  
# *Readiness Checks*: Tools and metrics are provided to verify tenant 
  readiness for the upgrade.
  
# *Upgrade Execution*: Admins can initiate the upgrade via the platform, 
  transitioning their tenant to Phase 3.
  
# *Reversion Capability*: Admins can revert the upgrade within a specified 
  timeframe.
  
# *Data Transition*: Classic data access and writes are phased out according 
  to the defined workflow.
  
# *Scalability Management*: Only eligible customers are notified and supported 
  during the upgrade process.
```

**Characteristics:**
- Each AC starts with **bold category**
- Colon separates category from criterion
- Groups related functionality
- Easy to track per feature area

---

## Pattern 3: Condition-Based (Consumer Apps)

**Use when:** Multiple apps/consumers, system behavior

**Example (from PRODUCT-16232):**
```
h2. Acceptance Criteria

# Deleted/inaccessible segments are not rendered as valid in Recents/Pinned, 
  be clearly indicated as invalid and can be removed/cleared by the user.
  
# Persisted segment selections used by Workflows, Notebooks and Dashboards or 
  other apps do not cause load/query failures after segment definition changes; 
  they are either auto-updated or surfaced as a localized error.
  
# Consumer apps can determine whether a stored segment selection is still 
  compatible with the current segment definition expectations.
  
# If compatibility issues are auto-fixable, the system auto-updates the stored 
  selection without user intervention.
  
# If not auto-fixable, the segment selector shows an actionable error state 
  (e.g. clear selection + choose different segment/value).
  
# For variables marked as dynamic (or governed by refresh rules), reuse does 
  not replay frozen array values; stale/invalid values are refreshed or 
  surfaced via AC5.
```

**Characteristics:**
- Each AC describes system behavior
- Covers multiple consumers/scenarios
- Includes error handling
- May reference other ACs (AC5)

---

## Pattern 4: Must-Have Requirements by Category

**Use when:** Infrastructure, library replacement, broad scope

**Example (from PRODUCT-15256):**
```
h2. Must-Haves to Be on Par with Old Library

h3. 1. Core Features
* *Drag-and-Drop:* Smooth, intuitive drag-and-drop with constraints (grid-based)
* *Responsive Design:* Breakpoint support for device-specific layouts
* *Dynamic Resizing:* Resizable elements with real-time updates
* *Grid System:* Configurable grids with snapping, row/column alignment
* *Multi-selection:* Ability to multi-select tiles and move them

h3. 2. Performance
* *Optimized Rendering:* minimal re-renders
* *Lightweight:* Small bundle size and efficient updates
* *Batch Operations:* Efficient processing of multiple layout changes

h3. 3. Developer Experience, Accessibility, Maintenance
* *Easy Integration:* Clear APIs, TypeScript support, documentation
* *Keyboard Navigation:* Full keyboard support for layout interactions
* *Screen Reader Support:* WAI-ARIA compliance and clear labels
* *Cross-Browser Support:* Compatibility with all major browsers
```

**Characteristics:**
- Hierarchical structure (h2, h3)
- Categories group related requirements
- Each requirement has **bold name** + description
- Works well for "feature parity" VIs

---

## Writing Rules

### Good AC - Specific & Testable

✅ **Good:**
```
# Admins can initiate the upgrade via the tenant settings page, 
  transitioning their tenant to Phase 3 within 5 minutes.
```
- **Who:** Admins
- **What:** Initiate upgrade
- **Where:** Tenant settings page
- **Outcome:** Transition to Phase 3
- **Performance:** Within 5 minutes

❌ **Bad:**
```
# Upgrade works well for users
```
- Who? What users?
- What does "works well" mean?
- How do you verify this?

---

### Good AC - Covers Error Cases

✅ **Good:**
```
# If upgrade fails, system rolls back automatically and displays 
  actionable error message to admin with support contact.
```

❌ **Bad:**
```
# Upgrade should not fail
```
- Unrealistic
- Doesn't specify error handling

---

### Good AC - Includes NFRs When Critical

✅ **Good (performance-critical):**
```
# Dashboard renders with new layout engine in <200ms (P95) for 
  dashboards with up to 50 tiles.
```

❌ **Bad:**
```
# Dashboard loads fast
```

✅ **Good (security-critical):**
```
# Segment selections are validated against user's current permissions 
  before applying, preventing unauthorized data access.
```

---

## Common Patterns

### UI/UX Features
```
# Users can [action] via [UI location]
# [UI element] displays [information] when [condition]
# Users receive [feedback] within [timeframe] after [action]
```

### API/Integration
```
# API endpoint [endpoint] accepts [input] and returns [output] in [format]
# Integration with [system] succeeds when [condition]
# API responds within [time] for [% of requests]
```

### Data/State
```
# System persists [data] when [event] occurs
# [Data] is synchronized across [systems] within [timeframe]
# Stale [data] is refreshed when [condition]
```

### Migration/Upgrade
```
# Existing [items] are migrated to [new format] without data loss
# Users can revert to [previous state] within [timeframe]
# [Old system] continues working until [cutover event]
```

### Error Handling
```
# When [error condition], system displays [specific message]
# Failed [operation] can be retried with [outcome]
# Errors are logged to [location] with [detail level]
```

---

## Must vs Should vs Can

**Must** - Non-negotiable, blocks release
```
# Upgrade *must* validate user permissions before proceeding
```

**Should** - Important but negotiable
```
# System *should* send email notification within 1 hour
```

**Can** - Optional, nice-to-have
```
# Users *can* schedule upgrade for specific time window
```

**Tip:** Default to "must" unless explicitly optional

---

## Checklist for Review

Before finalizing ACs:
- [ ] Each AC is testable (can verify true/false)
- [ ] No vague terms ("works well", "is fast", "good UX")
- [ ] Covers all key functionality from requirements
- [ ] Includes critical NFRs (security, performance)
- [ ] Error cases addressed
- [ ] Clear who/what/where/when
- [ ] Can demo each AC independently

---

## Common Mistakes

### ❌ Too Vague
```
# System performs well under load
```
**Fix:** Define "well" and "load"
```
# System handles 1000 concurrent users with <500ms P95 response time
```

---

### ❌ Implementation Detail
```
# Use React Query for data fetching
```
**Fix:** Focus on outcome, not how
```
# Dashboard data refreshes automatically every 30s without page reload
```

---

### ❌ Missing Error Cases
```
# Users can upload files
```
**Fix:** Cover failure scenarios
```
# Users can upload files up to 10MB. Files exceeding limit show 
  error message with size limit. Unsupported formats show list 
  of accepted formats.
```

---

### ❌ Combining Multiple Things
```
# System supports upload, download, and deletion of files with 
  validation and error handling
```
**Fix:** Split into separate ACs
```
# Users can upload files up to 10MB with format validation
# Users can download previously uploaded files
# Users can delete their own files with confirmation dialog
```

---

## Templates by VI Type

### Technical Enabler (Simple)
```
# [Component] migrates from [old] to [new]
# [Old system] is removed/deprecated
# Team updates [process] to [new process]
```

### Customer Feature
```
# [User role] can [action] via [UI/API]
# [Feature] works with [existing feature]
# Users receive [notification/feedback] when [event]
# [Metric] is tracked for adoption measurement
```

### Platform/Infrastructure
```
# System handles [scale metric] without degradation
# [Data] remains consistent across [components]
# Failures in [component] do not affect [other component]
# Metrics exposed for [monitoring need]
```
