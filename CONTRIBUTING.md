# Contributing to crystalclear-skills

We welcome new skills, improvements to existing ones, and better examples.

## How to Add a Skill

1. Fork this repo
2. Create your skill file using `skills/_template.md`
3. Place it in the appropriate category under `skills/`
4. Open a PR — the validator will check your frontmatter automatically

## Skill Requirements

Every skill file must include valid YAML frontmatter with these fields:
- `skill_id`: Dot-notation ID (e.g. `social.post_generator`)
- `name`: Human-readable name
- `version`: Semantic version (e.g. `v1.0`)
- `status`: `active`, `testing`, or `draft`
- `autonomy_level`: One of the 5 levels (see docs/autonomy-levels.md)
- `tags`: Array of relevant tags
- `author`: GitHub username

## Code of Conduct

Be excellent to each other. Focus on the work.

## License

By contributing, you agree your contributions will be licensed under MIT.
