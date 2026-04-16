# /ubuild

Interactive Flutter app scaffolding framework. Interviews the developer, generates a structured plan in `scaffold/`, then builds the app phase by phase.

## When to use

- Starting a new Flutter project from scratch
- Adding structure to an existing project that grew organically
- Onboarding a new developer to understand the app's planned architecture

## Phases

| Phase | What it does | Output |
|-------|-------------|--------|
| 1. Interview | Conversational requirements gathering | `scaffold/1-interview/requirements.md` |
| 2. Sitemap | Route tree, nav pattern, auth gates | `scaffold/2-sitemap/` |
| 3. Layout | App shell, components, theme, responsive | `scaffold/3-layout/` |
| 4. Data | Models, providers, API integration | `scaffold/4-data/` |
| 5. Features | Detailed per-feature specs | `scaffold/5-features/` |
| 6. Build | Actual Flutter code generation | `lib/src/` |

Each phase transition requires a recursive audit of all prior artifacts. The developer must confirm before moving forward.

## How to use

1. Install: `curl -sL -o ~/.claude/commands/ubuild.md https://raw.githubusercontent.com/instruxi-io/ubuild-skills/main/ubuild/SKILL.md`
2. Open your Flutter project in Claude Code
3. Type `/ubuild`
4. The agent will start interviewing you

## Tips

- Answer interview questions naturally — the agent adapts based on your responses
- If you already have a codebase, mention it early so the agent accounts for existing structure
- `scaffold/master-plan.md` tracks everything — read it if you're resuming work
- Phase 6 (Build) generates code incrementally and verifies with `flutter analyze` between each feature
- Pair with `/enforcer` skills if building on the Enforcer API
