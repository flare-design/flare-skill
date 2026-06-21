# Flare Skill

Universal AI agent skill for operating Flare through MCP: projects, live canvas edits, assets, agent-generated image insertion, Flare backend generation jobs, motion design, audio/captions, and rendering.

## Install

```bash
npx skills add flare-design/flare-skill --skill flare
```

For SSH/private access:

```bash
npx skills add git@github.com:flare-design/flare-skill.git --skill flare
```

## Update

```bash
npx skills check
npx skills update flare
```

## Versioning

The skill uses semantic versioning in `skills/flare/SKILL.md` under `metadata.version`, and releases should also be tagged in git, for example `v0.1.0`.
