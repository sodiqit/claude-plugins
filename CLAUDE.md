# CLAUDE.md

Guidance for working in this repo — a Claude Code plugin marketplace (`sodiqit-plugins`).

## Layout

```
.claude-plugin/marketplace.json   # marketplace manifest — lists plugins + their versions (source of truth shown in /plugin UI)
plugins/<name>/
  .claude-plugin/plugin.json       # per-plugin manifest (name, description, version, author)
  skills/<skill>/SKILL.md          # a skill: frontmatter (name, description, version) + body
  agents/ | commands/              # other plugin components
```

Each plugin's `source` in `marketplace.json` points to `./plugins/<name>`.

## Versioning rule (MUST follow)

**When you change a skill (or any plugin content), bump the version — otherwise consumers won't reliably pull the update.** Bump in BOTH the skill and its plugin:

1. **Skill** — bump `version` in the skill's `SKILL.md` frontmatter (add the field if missing).
2. **Plugin** — bump `version` in `plugins/<name>/.claude-plugin/plugin.json` **and** the matching plugin entry in root `.claude-plugin/marketplace.json`.

**The plugin version MUST be identical in `plugin.json` and `marketplace.json`** — these two must never drift.

Semver:
- **patch** (`1.0.0 → 1.0.1`) — fix / wording / small edit to a skill.
- **minor** (`1.0.0 → 1.1.0`) — added a new skill, agent, or command to the plugin.
- **major** (`1.0.0 → 2.0.0`) — breaking change (renamed/removed a skill, changed its trigger contract).

After bumping: `git commit && git push`. Consumers update with `/plugin marketplace update sodiqit-plugins` then `/reload-plugins`.

### Before committing — verify no drift (MUST run)

A plugin version is duplicated in two files and skills carry their own `version`; these must never drift. Run this gate before every commit — it prints `OK` or lists problems and exits non-zero:

```bash
python3 - <<'PY'
import json, glob, re, sys
bad = []
mkt = {p['name']: p.get('version') for p in json.load(open('.claude-plugin/marketplace.json'))['plugins']}
for name, mv in mkt.items():
    pv = json.load(open(f'plugins/{name}/.claude-plugin/plugin.json')).get('version')
    if pv != mv:
        bad.append(f'{name}: plugin.json={pv} != marketplace.json={mv}')
for f in glob.glob('plugins/**/SKILL.md', recursive=True):
    parts = open(f).read().split('---')
    if len(parts) < 3 or not re.search(r'(?m)^version:', parts[1]):
        bad.append(f'{f}: missing `version` in frontmatter')
print('\n'.join(bad) + '\nFAIL' if bad else 'OK')
sys.exit(1 if bad else 0)
PY
```

If it prints anything other than `OK`, fix the listed files before committing.

## Conventions

- Skill content (SKILL.md, instructions, examples) is in English.
- Keep `marketplace.json` valid JSON; every `source` path must exist and contain `.claude-plugin/plugin.json`.
- Do not track IDE files (`.idea`) or `.DS_Store` — they are gitignored.
