# sodiqit-plugins

A Claude Code plugin marketplace with personal development plugins.

## Install

Add the marketplace from GitHub, then install any plugin from it:

```
/plugin marketplace add sodiqit/claude-plugins
/plugin install devops@sodiqit-plugins
```

Browse and manage installed plugins with `/plugin`.

## Plugins

| Plugin | Description |
|--------|-------------|
| `review-toolkit` | PR review agents: comments, tests, error handling, type design, code quality, simplification. |
| `arch-debate` | Multi-perspective architecture advisory sessions (Kleppmann, Uncle Bob, Evans) using ACH methodology. |
| `rfc` | RFC document generation — interview, codebase exploration, structured output. |
| `essentials` | Everyday workflow skills — planning, decision-making, critical thinking. |
| `devops` | Containerization & deployment skills — Docker multi-stage builds, image optimization, security hardening, Compose orchestration. |

## Layout

```
.claude-plugin/marketplace.json   # marketplace manifest (lists plugins)
plugins/<name>/
  .claude-plugin/plugin.json       # per-plugin manifest
  skills/ | agents/ | commands/    # plugin contents
```

Each plugin's `source` in the marketplace manifest points to `./plugins/<name>`.
