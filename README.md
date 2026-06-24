# Flare Skill

Universal AI agent skill for operating Flare through MCP: projects, live canvas edits, assets, agent-generated image insertion, Flare backend generation jobs, motion design, audio/captions, and rendering.

## Install

Install through the `skills` CLI from skills.sh so the source is tracked and future updates can be pulled automatically.

For a normal GitHub install:

```bash
npx skills add flare-design/flare-skill --skill flare --global
```

For the private SSH repo:

```bash
npx skills add git@github.com:flare-design/flare-skill.git --skill flare --global
```

To install the skill for Codex and Claude Code in one command:

```bash
npx skills add git@github.com:flare-design/flare-skill.git --skill flare --agent codex claude-code --global
```

To install it for every supported local agent:

```bash
npx skills add git@github.com:flare-design/flare-skill.git --skill flare --agent '*' --global --yes
```

List available skills in the repo before installing:

```bash
npx skills add git@github.com:flare-design/flare-skill.git --list
```

Verify the installed skill:

```bash
npx skills list --global --agent codex
npx skills list --global --agent claude-code
```

## Update

Check and update tracked installs:

```bash
npx skills check
npx skills update flare
```

Recent Flare MCP servers expose `check_client_setup`, which lets the skill remind agents to update when the local skill version is older than the server-recommended version.

If the skill was installed globally and you want to update all global skills:

```bash
npx skills update --global
```

Avoid manual copying for normal installs. It works for quick local debugging, but it does not give the agent a tracked source for `skills check` and `skills update`.

## MCP setup

Installing this skill only gives agents the Flare operating instructions. The Flare MCP server or connector still needs to be configured separately in the target agent. Once the MCP connection is available, the skill guides the agent through project discovery, canvas edits, asset uploads, generated image insertion, motion design, and rendering workflows.

## Versioning

The skill uses semantic versioning in `skills/flare/SKILL.md` under `metadata.version`. Releases should also be tagged in git, for example `v0.1.19`.

For skill behavior changes:

```bash
git commit -am "Update Flare skill workflow"
git tag v0.1.19
git push origin main --tags
```

For README-only changes, a normal commit is enough.
