# Contribution Guidelines for Claude

## Principles

1. **uBuild skills are orchestration-heavy.** They direct multi-phase workflows, not just provide context. Every phase has clear inputs, outputs, and transition gates.

2. **Phases are sequential and audited.** Never skip phases. Never skip audits. The recursive audit between phases catches inconsistencies early.

3. **Interview is adaptive.** Don't dump all questions. Have a conversation. Group naturally. Adapt based on answers.

4. **Scaffold is planning, not code.** Phases 1-5 produce markdown in `scaffold/`. Phase 6 produces Flutter code. Don't blur this boundary.

5. **master-plan.md is the source of truth.** Always read it first, always update it on transitions.

## File Structure

```
ubuild/
├── SKILL.md     # The skill (loaded as /ubuild slash command)
├── README.md    # User guide
└── VERSION      # Auto-incremented
```

## When Editing

- Preserve the 6-phase structure — it's the core of the framework
- Audit gates are non-negotiable — don't weaken them
- Keep interview questions conversational, not checklist-style
- Don't add Flutter-version-specific advice — it goes stale
- Don't hardcode package versions
