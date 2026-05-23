# VI Imports Directory

Imported VI snapshots from Jira serving as real examples for the draft-vi skill.

## Naming

`jira-{JIRA-KEY}-{YYYY-MM-DD}-{HHMM}.md`

Example: `jira-PRODUCT-16019-2026-02-13-0335.md`

## Metadata

```yaml
---
retrieved: YYYY-MM-DD HH:MM
source: jira
key: JIRA-KEY
mcp: yes
---
```

## Usage

1. Query VI using `query-jira-workitem` skill
2. Save here with standard naming
3. Reference in `references/vi-examples.md` using `../imports/jira-{KEY}-{DATE}-{TIME}.md`
4. Update when VI evolves significantly
