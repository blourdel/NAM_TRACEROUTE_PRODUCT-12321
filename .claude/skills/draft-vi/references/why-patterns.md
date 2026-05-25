# WHY Section Patterns

Patterns for writing compelling WHY sections that explain context and rationale for the VI.

## Table of Contents

1. [Structure](#structure)
2. [Pattern 1: Customer Pain-Focused](#pattern-1-customer-pain-focused)
3. [Pattern 2: Technical Debt/Constraint](#pattern-2-technical-debtconstraint)
4. [Pattern 3: External Evidence](#pattern-3-external-evidence-outdated-tech)
5. [Pattern 4: Scenario Breakdown](#pattern-4-scenario-breakdown-where-it-hurts)
6. [Pattern 5: Adoption/Flexibility Need](#pattern-5-adoptionflexibility-need)
7. [Pattern 6: Simple Migration Need](#pattern-6-simple-migration-need)
8. [Writing Tips](#writing-tips)
9. [Checklist](#checklist)
10. [Common Mistakes](#common-mistakes)

---

## Structure

```
h2. WHY

<Current situation>
<Pain points with examples>
<Impact (who's affected, how badly)>
<Timing (why now, external pressure)>
```

---

## Pattern 1: Customer Pain-Focused

**Use when:** Customer-facing features, addressing user complaints

**Example (from PRODUCT-15500):**
```
Customers using hybrid tenants are unable to fully benefit from the scalability,
simplified administration, and modernized user interface offered by 3rd Gen 
Dynatrace. Without a structured upgrade workflow, the transition process is unclear,
leading to potential delays, misconfigurations, or incomplete migrations. 
Additionally, there is no mechanism to ensure readiness or provide support for the 
upgrade process, leaving customers uncertain about the steps required.
```

**Elements:**
- Current limitation ("unable to fully benefit")
- Consequences ("delays, misconfigurations")
- Impact on customer ("uncertain about steps")

---

## Pattern 2: Technical Debt/Constraint

**Use when:** Technical enablers, refactoring, architecture improvements

**Example (from PRODUCT-16019):**
```
The current division and purpose of the AppShell and Platform Search service 
components don't align with our goals. We're organizationally limited in our 
ability to optimize performance and easily access internal services. It's 
technically possible, but we're only bound by the Platform Search service's 
responsibilities.
```

**Elements:**
- Current state doesn't align with goals
- Organizational/technical constraints
- What's blocked by current approach

---

## Pattern 3: External Evidence (Outdated Tech)

**Use when:** Modernization, library replacement

**Example (from PRODUCT-15256):**
```
The existing React Grid library is no longer actively maintained and is 
technically outdated.

A thorough explanation and assessment is provided by Kibana, which was also 
used and migrated off React-Grid, here [link to Kibana blog]
```

**Elements:**
- Objective fact (unmaintained, outdated)
- External validation (Kibana case study)
- Implicit risk (if they migrated, so should we)

---

## Pattern 4: Scenario Breakdown (Where It Hurts)

**Use when:** Data integrity, system reliability, complex problem

**Example (from PRODUCT-16232):**
```
We currently have no single "source of truth" for a complete, reusable Segment 
selection over time (segment definition + variables + selected state). Each 
consuming surface persists its own snapshot payload, and these snapshots are 
not synchronized when segments are deleted or changed.

This creates time-based drift: the longer a saved selection lives, the more 
likely it becomes inaccurate, misleading, or broken.

h3. Where It Hurts

h4. 1) Orphaned References
* What happens: Segment persisted, later deleted. Reference can't resolve.
* Where it shows up: Recents, Dashboards, Notebooks, URLs
* User symptoms: Broken entries, apply fails, "why is this still here?"

h4. 2) Stale Selections
* What happens: Persisted selections replay old values. Drift from reality.
* Where it shows up: Workflows, Dashboard defaults, URLs
* User symptoms: Silent wrong results, missing new items

h4. 3) Incompatibility After Evolution
* What happens: Segment definitions evolve. Old payloads incompatible.
* Where it shows up: Workflows/Dashboards/Notebooks failing to load
* User symptoms: Silent drift or hard failures
```

**Elements:**
- Root cause (no single source of truth)
- Consequence (time-based drift)
- Specific scenarios with:
  - What happens (cause)
  - Where it shows up (affected areas)
  - User symptoms (observable problems)

---

## Pattern 5: Adoption/Flexibility Need

**Use when:** Improving existing features, increasing adoption

**Example (from PRODUCT-11388):**
```
As addon intents become more widely adopted, the demand for more significant 
flexibility increases. Users request additional sizes for the addon modal 
(already delivered), support for sheets, and various design optimizations. 
This VI aims to implement these enhancements to boost the overall adoption 
of addon intents while ensuring a delightful user experience.
```

**Elements:**
- Trend (increasing adoption)
- User feedback (requests for flexibility)
- Goal (boost adoption, improve UX)

---

## Pattern 6: Simple Migration Need

**Use when:** Straightforward technical migration

**Example (from PRODUCT-16056):**
```
AppShell currently uses an outdated deployment method for "built-in" apps – 
Hermes, and it needs to be migrated.
```

**Elements:**
- Current state (uses Hermes)
- Problem (outdated)
- Need (migration)

**Note:** Keep it short if problem is straightforward

---

## Writing Tips

### Do:
- ✅ Start with current situation/state
- ✅ Use concrete examples and evidence
- ✅ Quantify impact when possible ("50% of users", "2 hours/week")
- ✅ Explain timing if relevant ("competitor launched X", "deadline approaching")
- ✅ Link to evidence (Slack threads, customer feedback, competitor analysis)

### Don't:
- ❌ Jump straight to solution (that's next section)
- ❌ Be vague ("users are unhappy" → specify what makes them unhappy)
- ❌ Only say "technical debt" (explain why it matters)
- ❌ Assume everyone knows the background

### Length Guide:
- **Simple migration:** 1-2 sentences
- **Standard VI:** 1 paragraph (3-5 sentences)
- **Complex problem:** 2-3 paragraphs + scenario breakdown

---

## Checklist

Before finalizing WHY section, verify:
- [ ] Current situation is clear
- [ ] Problem is concrete (not abstract)
- [ ] Impact is explained (who's affected, how)
- [ ] Evidence or examples provided
- [ ] Reader understands urgency/priority
- [ ] Haven't jumped to solution yet

---

## Common Mistakes

**❌ Too abstract:**
```
WHY: We need to improve the user experience and make the system more flexible.
```

**✅ Concrete:**
```
WHY: Users cannot customize modal sizes for addon intents. Current fixed size 
causes layout issues when displaying complex forms, forcing workarounds like 
external windows. 15+ teams have requested flexible sizing in #papa-help.
```

---

**❌ Just stating "technical debt":**
```
WHY: Technical debt in our codebase.
```

**✅ Explaining impact:**
```
WHY: React Grid library is unmaintained (last update 2019) and blocks our 
ability to support canvas layouts. Performance issues (2s+ render time with 
50+ tiles) impact 30% of dashboard users.
```

---

**❌ Jumping to solution:**
```
WHY: We should migrate to new deployment method because it's better.
```

**✅ Problem-focused:**
```
WHY: Current Hermes deployment requires manual steps for each release, taking 
~2 hours and blocking sprint cadence. Other apps use automated Hub deployment, 
creating inconsistency in our release process.
```
