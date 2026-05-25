# VI Template (VCG Official 2023)

Official Dynatrace VCG VI template structure. Use this as the foundation for creating new VIs in Jira.

## Template Sections

```
h2. Short Abstract / Blogline
<Internally focused one-sentence description>

h2. Customer Zero
<Who validates this - internal team or external customer>

h2. Target Audience
<Main personas who benefit from this VI>

h2. WHY
<Problem context, pain points, urgency, and rationale>

h2. Functional Requirements / Solution Journey
<End-to-end use cases with optional solution concept/workflow>

h2. Non-Functional Requirements
<Security, performance, scalability, reliability, usability attributes>

h2. Enablement Requirements
<Support setup, Internal Adoption, Licensing, Marketing, Sales, Partner Alignment>

h2. E2E Acceptance Criteria
<Specific, testable conditions for "done">

h2. Customer Zero Planning
<Specific functionality subset for Customer Zero with adoption plan>

h2. E2E Demo (for acceptance)
<Demo flow proving VI is done - can reference help docs, Hub, GitHub>

h2. Out of Scope
<Explicit boundaries - what's NOT included>

h2. Success Metrics
<Measurable indicators evaluated 3-6 months post-GA>

h2. Cost Analysis
<Additional costs or cost savings from this VI>
```

## Jira Wiki Markup Format

When copying to Jira, use these markup styles:

```
h2. Section Title
h3. Subsection

* Bullet item
** Sub-bullet
# Numbered item
## Sub-numbered

*bold text*
_italic text_
{{monospace}}
[Link text|http://url]
```

## Section Guidance

### Short Abstract / Blogline
- One sentence, internally focused
- What you're building (not why - that's WHY section)
- Used for roadmaps and internal alignment
- Example: "Introduce bottom sheet support and flexible modal sizing for addon intents"

### Customer Zero
- Specific team, customer, or internal adopter
- Who provides feedback before GA
- Example: "Platform Apps team (internal)" or "Customer XYZ"

### Target Audience
- End-user personas
- Who benefits from this VI
- Example: "App developers using addon intents", "Dynatrace admins managing upgrades"

### WHY
- Current situation and pain points with concrete examples
- Impact (who's affected, how badly)
- Timing/urgency if relevant
- See `why-patterns.md` for detailed patterns

### Functional Requirements / Solution Journey
- End-to-end use cases from user perspective
- What the solution does (not how to implement)
- May include solution concept/numbered workflow
- Example:
  ```
  h3. Use Cases
  # User can select bottom sheet layout for addon intent
  # User can configure modal width (S/M/L/XL/custom)
  
  h3. Solution Concept
  # Developer sets intent.layout = 'sheet' in manifest
  # Platform renders bottom sheet with 40% height default
  # User can drag sheet to full-screen or dismiss
  ```

### Non-Functional Requirements
Common NFRs:
- **Security**: Authentication, authorization, data protection
- **Performance**: Response time, throughput, latency targets
- **Scalability**: Load capacity, concurrent users
- **Reliability**: Uptime, error rates, failover
- **Usability**: Accessibility (WCAG), keyboard navigation
- **Maintainability**: Code quality, testability

### Enablement Requirements
What's needed before customers can use this:
- Support: Trained, runbooks created
- Internal Adoption: Dog-fooded by internal teams
- Licensing: Pricing/entitlement updated if needed
- Marketing: Launch materials, blog post, demo video
- Sales: Sales deck, competitive positioning
- Partner Alignment: Partner APIs, integration guides

### E2E Acceptance Criteria
- Specific and testable (can verify true/false)
- Covers functional + key NFRs
- Use "must", "should" appropriately
- See `ac-patterns.md` for detailed patterns

### Customer Zero Planning
- Subset of functional requirements for first adopter
- Adoption timeline and success criteria
- Feedback collection plan
- Example: "Platform Apps team implements bottom sheets in Settings app by Sprint 245. Success: 2+ production use cases with positive feedback."

### E2E Demo (for acceptance)
- Step-by-step demo showing VI is "done"
- Can reference help docs, Hub demo, GitHub example
- Engineering demos to PM/PA and stakeholders
- Example: "1) Open Settings app → 2) Click 'Advanced' → 3) Bottom sheet slides up → 4) Drag to full-screen → 5) Dismiss with swipe down"

### Out of Scope
- Explicit boundaries to prevent scope creep
- Link to future VIs if appropriate
- Example:
  ```
  * Custom sheet animations (future VI)
  * Multi-sheet stacking (not planned)
  * Mobile app support (separate VI)
  ```

### Success Metrics
- Quantifiable with targets
- Mix of adoption, performance, quality
- Evaluated 3-6 months post-GA
- Example:
  ```
  * Adoption: 5+ apps use bottom sheets within 3 months
  * Performance: Modal load time <200ms P95
  * Quality: <2% error rate on sheet interactions
  * Satisfaction: NPS >50 from developer survey
  ```

### Cost Analysis
- Additional infrastructure costs (compute, storage, bandwidth)
- Operational costs (support, maintenance)
- Cost savings (deprecated services, efficiency gains)
- Example: "+$500/month for additional CDN bandwidth, -$2000/month from retiring legacy modal service"

## When to Use This Template

**Use for:**
- ✅ Most VIs (default choice)
- ✅ Technical enablers
- ✅ Platform/infrastructure improvements
- ✅ Standard customer features

**Consider team-specific template for:**
- Heavy UX work with complex design phases
- Multiple design iterations and user testing cycles
- VIs with extensive customer research needs

## Related References

- **why-patterns.md** - How to write compelling WHY sections
- **ac-patterns.md** - Acceptance criteria patterns and examples
- **vi-examples.md** - Real VI examples showing different patterns
- **vi-lifecycle.md** - What's needed at each VI state
