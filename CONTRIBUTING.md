# Contributing

## Editing the /ubuild skill

- Preserve the 6-phase pipeline (Interview → Sitemap → Layout → Data → Features → Build)
- Don't weaken audit gates — recursive review between phases is the core quality mechanism
- Keep interview questions conversational and adaptive
- Don't add Flutter/Dart version-specific advice or hardcode package versions
- Test the skill end-to-end with a sample app before submitting

## Adding new skills

1. Create a directory: `my-skill/`
2. Add `SKILL.md` with YAML frontmatter (`name`, `description`)
3. Add `README.md` explaining when/why to use the skill
4. Add `VERSION` file containing `1`
5. Open a PR
