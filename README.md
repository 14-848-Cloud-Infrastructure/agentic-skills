# agentic-skills

This is a collection of agent skills, each one a directory with a `SKILL.md` inside it, holding instructions that your coding agent loads when a task actually calls for them. The description in the frontmatter stays in context so the agent knows the skill exists. The skill body is only loaded when it gets used.

## What's in here (New Skills will be added)

| Skill | Does what | Status |
| --- | --- | --- |
| database-reviewer | Reviews PostgreSQL queries, migrations, and schemas. Flags injection, missing RLS, unindexed foreign keys, and migrations that take an exclusive lock on a live table. Read-only. | working |
| pyspark-solver | Debugs and analyzes Spark commands | working |



## Layout

```
agentic-skills/
  README.md
  database-reviewer/
    SKILL.md
  another-skill/
    SKILL.md
```

One directory per skill. 
