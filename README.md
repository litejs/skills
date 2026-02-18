# LiteJS Skills for Claude Code

[Claude Code](https://claude.com/claude-code) skills for working with LiteJS projects.

## Skills

| Skill | Description |
|---|---|
| [litejs-ui](skills/litejs-ui/) | LiteJS UI engine — templates, views, bindings, events, i18n, El API |
| [litejs-testing](skills/litejs-testing/) | LiteJS test framework — assertions, mocking, data-driven tests, snapshots |
| [litejs-release](skills/litejs-release/) | LiteJS release helper — version bumping, changelog, tagging, publishing |

## Install

```
/plugin marketplace add litejs/skills
/plugin install litejs-testing@litejs-skills
```

## Update

Marketplace auto-updates by default. To trigger manually:

```
/plugin marketplace update litejs-skills
```

## Uninstall

```
/plugin uninstall litejs-testing@litejs-skills
```

## Manual Install

Clone the repo and symlink each skill into `~/.claude/skills/`:

```sh
git clone https://github.com/litejs/skills.git ~/code/litejs/skills
ln -s ~/code/litejs/skills/litejs-testing ~/.claude/skills/litejs-testing
```

## License

MIT
