# VI Definition & Types

Fundamental concepts about Value Increments in Dynatrace VCG framework.

## What is a Value Increment?

**Value Increments** describe the end-to-end use cases needed to resolve the pain defined in the Value Pack. One VI can consist of multiple use cases but is typically bound to one quarter in implementation.

The high-level functional, non-functional and enablement requirements needed to fulfil the use cases are specified in the VI. It may include research or feasibility work (e.g. PoC) to elucidate and validate requirements.

## VI Types

### Standard Value Increment
Customer-facing or platform features that deliver direct value. These follow the full VI lifecycle from Open → Closed.

**Characteristics:**
- Solves customer or internal user pain points
- Has clear target audience and Customer Zero
- Delivers measurable value (adoption, performance, quality)
- May be customer-facing or platform improvement

**Examples:**
- Customer workflows (upgrade readiness)
- UX improvements (addon intents)
- Platform features (segments sync)

---

### Enabler - Technical Value Increment

"Enabler" VIs are owned and driven by the Architecture team with the aim of making improvements to the Product's Architecture that are not directly seen by the customer.

**What it includes:**
- Architecture improvements
- Infrastructure work
- Compliance use cases
- Non-functional requirements (NFRs)
- Technical debt reduction
- Library/framework upgrades

**Characteristics:**
- Customer Zero often internal teams
- WHY focused on technical constraints/debt
- Success metrics on performance/reliability
- Shorter, more focused than customer VIs
- May enable future customer features

**Examples:**
- Service migrations (AppShell from Hermes)
- Platform reorganization (Platform Search generalization)
- Library modernization (dashboard layout engine)

---

### Priority Indicator - Value Increment

A "Priority Indicator" enables the prioritization of cross-capability work. Work that might otherwise not be visible on the VI level but needs to be prioritized can be captured in a Priority Indicator.

**Use cases:**
- Dev Epics that one capability requires from another
- Cross-capability dependencies
- Shared infrastructure work
- Platform services used by multiple capabilities

**Purpose:**
- Make dependencies visible at VI level
- Enable prioritization relative to team's own VIs
- Track when dependencies can be resolved
- Coordinate across capabilities

**Characteristics:**
- Not a full feature implementation
- Represents work needed by other teams
- Prioritized alongside regular VIs
- May not have traditional Customer Zero
- Success measured by unblocking other work

---

## Collaboration & Stakeholders

A successful VI lifecycle relies on the Capability and stakeholder's commitment to collaboration and alignment throughout all its stages. This requires:
- Continuous involvement and alignment
- Ongoing communication
- Active participation of all stakeholders

### Key Roles

| Role | Responsibility |
|------|----------------|
| **Product Manager (PM)** | Problem definition, requirements, Customer Zero, enablement |
| **Product Architect (PA)** | Technical requirements, NFRs, architecture concept |
| **Product Engineer (PE)** | Execution accountability (Use Case Defined → Release Prep) |
| **Product Experience Designer (PXD)** | Discovery, ideation, mockups, design handoff |
| **Design Lead** | UX alignment, design quality sign-off |
| **Delivery Lead** | Assign PE, ensure roadmap reflects VI status |
| **Capability Lead** | Go/No-Go, prioritization, re-evaluate if deviations |
| **Lead Product Architect** | Assign PA or set to No-PA |

---

## Tracking Progress of VIs

During the entire VI lifecycle, updates are provided to different stakeholders, whether internal or external to the Capability.

### Related Meetings

**Value Increment Kick-off**
- State: Open
- Purpose: Share understanding of problem and WHY
- Attendees: PM/PA, teams, stakeholders

**Value Increment Refinement & Prioritization**
- State: Use Case Defined
- Purpose: Deep dive into use cases and solution approach
- Attendees: PM/PA, PE, teams

**Dev-Epic Breakdown**
- State: Ready for Implementation
- Purpose: Create and refine Dev Epics
- Attendees: PE, teams, stakeholders

**Capability Planning & Pre-planning**
- State: Ready for Implementation / Implementation
- Purpose: Plan sprints, refine work, manage WIP
- Attendees: Capability teams

**Design QA**
- State: Implementation
- Purpose: Validate design quality during implementation
- Attendees: PXD, Design Lead, teams

**Value Increment Review**
- State: Implementation
- Purpose: Review progress, accept/reject per ACs
- Attendees: PM/PA, PE, teams

**Product Demo (RnD audience)**
- State: Release Preparation
- Purpose: Demonstrate VI to internal stakeholders
- Attendees: RnD teams, PM/PA, PE

**Product Preview**
- State: Release Preparation
- Purpose: Preview with Customer Zero or broader audience
- Types: Internal Adoption, Private Preview, Public Preview
- Attendees: PM, Customer Zero, stakeholders

**Capability Delivery Update / VI Status Update**
- Frequency: Regular (weekly/bi-weekly)
- Purpose: VI status, risks, changes, delays
- Attendees: Capability teams, stakeholders

**Quarterly Capability Reviews (QCRs)**
- Frequency: Quarterly
- Purpose: Roadmap and delivery metrics review
- Attendees: Capability Leadership, Solution Leadership

**Bi-weekly Status Update**
- Frequency: Bi-weekly
- Purpose: High-level status to leadership
- Attendees: Solution Leadership, Capability Leadership

**Capability Retrospective**
- State: Closed
- Purpose: Inspect and adapt for future VIs
- Attendees: Capability teams, Org Lead

---

## Further Resources

**Launch Coordination Checklist (LCC)**
- Used for new services
- Ensures all deployment/launch aspects covered
- Signed off during Release Preparation

**Jira Templates**
- TEMPLATE-130 (standard template)
- TEMPLATE-80 (alternative template)
- Use these to create new VIs in Jira

**VCG Contact**
- vcg@dynatrace.com
- Questions about VI process, templates, lifecycle

---

## Best Practices

**Collaboration:**
- Don't rush state transitions
- Missing artifacts → delays later
- Better to pause and complete than forge ahead
- Regular sync between PM/PA/PE/teams

**State transitions:**
- Each state builds on previous
- Exit criteria must be met
- Re-evaluation needed if deviations occur

**Flow metrics:**
- Limit WIP (Work in Progress)
- Thorough blockers/dependencies addressed = smoother implementation
- Track cycle time per state to identify bottlenecks

**Communication:**
- Regular updates (weekly/bi-weekly)
- Proactive about risks and changes
- Keep stakeholders informed throughout
- Clear escalation path for blockers
