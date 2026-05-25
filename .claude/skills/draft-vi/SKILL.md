---
name: draft-vi
description: Provide guidance and a template to write a well-structured VI in Jira.
---

# Draft VI

## When to use
- Help write or review a VI description and structure

## Inputs
- `viType` (string, optional, example: customer-facing, technical enabler)
- `targetState` (string, optional, example: Open, Problem Stated, Use Case Defined)

## Workflow
1. Clarify problem, audience, and scope
2. Provide a standard VI template (Jira wiki markup)
3. Guide key sections (WHY, requirements, ACs, success metrics)
4. Review against target-state completeness checklist

## Template sections (generic)
```
h2. Short Abstract / Blogline
h2. Customer Zero
h2. Target Audience
h2. WHY
h2. Functional Requirements / Solution Journey
h2. Non-Functional Requirements
h2. Enablement Requirements
h2. E2E Acceptance Criteria
h2. Customer Zero Planning
h2. E2E Demo (for acceptance)
h2. Out of Scope
h2. Success Metrics
h2. Cost Analysis
```

## Output format
1. Template (copy-paste ready)
2. Section-by-section guidance
3. Completeness checklist for the target state

## Directory Structure
```
draft-vi/
├── SKILL.md              # This file
├── imports/              # Real VI snapshots from Jira
│   ├── README.md         # Naming conventions and usage
│   └── jira-{KEY}-{DATE}-{TIME}.md
└── references/
    ├── ac-patterns.md    # Acceptance criteria patterns
    ├── vi-definition.md  # VI concept and purpose
    ├── vi-examples.md    # Pattern catalog with real examples
    ├── vi-lifecycle.md   # Lifecycle states and criteria
    ├── vi-template.md    # Copy-paste template
    └── why-patterns.md   # WHY section patterns
```

## Maintaining Examples
1. Use `query-jira-workitem` skill to fetch latest VI snapshot
2. Save to `imports/` with naming: `jira-{JIRA-KEY}-{YYYY-MM-DD}-{HHMM}.md`
3. Update `references/vi-examples.md` to reference the import
4. Extract patterns and update the pattern catalog

## Assumptions
- VI standards and lifecycle states are provided in references/
- Real VI examples stored in imports/ and cataloged in references/vi-examples.md
