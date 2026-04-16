```
         ██╗   ██╗██████╗ ██╗   ██╗██╗██╗     ██████╗ 
         ██║   ██║██╔══██╗██║   ██║██║██║     ██╔══██╗
         ██║   ██║██████╔╝██║   ██║██║██║     ██║  ██║
         ██║   ██║██╔══██╗██║   ██║██║██║     ██║  ██║
         ╚██████╔╝██████╔╝╚██████╔╝██║███████╗██████╔╝
          ╚═════╝ ╚═════╝  ╚═════╝ ╚═╝╚══════╝╚═════╝ 
              F L U T T E R   S C A F F O L D E R
```

Claude Code skills for scaffolding Flutter apps with **uBuild** — an interactive, phased framework that interviews the developer and builds out a structured plan before generating code.

## Skills

| Skill | Description |
|-------|-------------|
| [`/ubuild`](ubuild/) | Interactive Flutter app scaffolder — 6-phase pipeline from interview to code generation |

## Install

```bash
curl -sL -o ~/.claude/commands/ubuild.md \
  https://raw.githubusercontent.com/instruxi-io/ubuild-skills/main/ubuild/SKILL.md
```

Then type `/ubuild` in Claude Code to start.

## How it works

uBuild scaffolds Flutter apps in 6 phases:

1. **Interview** — conversational requirements gathering (auth, features, platforms, design)
2. **Sitemap** — route tree, navigation pattern, auth gates, deep links
3. **Layout** — app shell, reusable components, theme/design tokens, responsive strategy
4. **Data** — models, Riverpod providers, API integration mapping
5. **Features** — detailed per-feature specs (screens, flows, edge cases, MVP scope)
6. **Build** — incremental Flutter code generation with compile verification

Every phase transition requires a **recursive audit** — the agent re-reads all prior artifacts, checks for consistency, and gets developer approval before proceeding. Nothing is rushed.

All planning artifacts live in `scaffold/` with `master-plan.md` as the source of truth. Resume any session by reading `scaffold/master-plan.md`.

## Works with

- **[Enforcer skills](https://github.com/instruxi-io/enforcer-skills)** — if building on the Enforcer API, load `/enforcer` + domain skills for backend context
- **[Enforcer docs MCP](https://github.com/instruxi-io/enforcer-docs-mcp)** — live API endpoint discovery
- **Dart MCP server** — hot reload, widget tree, runtime errors during Phase 6

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT
