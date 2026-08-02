---
title: Install the Official @midwayjs/skill-midway Knowledge Bundle
impact: MEDIUM
impactDescription: "Version-matched Midway docs/API/changelog directly available to AI tools"
tags: devops, skill, ai-tools, knowledge, cli
---

## Install the Official @midwayjs/skill-midway Knowledge Bundle

`@midwayjs/skill-midway` (v4) is Midway's official AI-assistant skill package. It solves two problems: (1) installing a Midway skill into AI coding tools (Codex, Cursor, Trae, opencode, Claude, etc.), and (2) providing local JSON queries for docs, APIs, packages, and changelogs — all matched to the installed Midway major version. Unlike this rules-based skill (curated best practices), `skill-midway` is the raw official knowledge bundle; the two are complementary. Keep it in sync with the app's Midway version — after upgrading the package run `npx midway-skill update`.

**Incorrect (no version pinning, manual doc lookups, stale skills):**

```bash
# ❌ no version pinning — resolves the latest major, may mismatch the app's Midway
npx midway-skill install --target cursor

# ❌ stale after upgrade — skill bundle still describes the old API
npm i @midwayjs/skill-midway@4 --save-dev   # then never run update
```

**Correct (devDependency + explicit target + update after upgrades):**

```bash
# 1) install as a devDependency (version-matched to the app)
npm i @midwayjs/skill-midway@4 --save-dev
# package.json: { "devDependencies": { "@midwayjs/skill-midway": "^4.0.0" } }

# 2) install the skill into AI tools (interactive, or explicit targets)
npx midway-skill install                    # interactive picker
npx midway-skill install --target codex     # → .codex/skills/midway/SKILL.md
npx midway-skill install --target cursor    # → .cursor/commands/opsx-midway.md
npx midway-skill install --target all       # all supported targets

# 3) after any Midway upgrade, refresh the installed skill
npx midway-skill update
```

**Querying docs/API locally (JSON output — scriptable by agents):**

```bash
npx midway-skill resolve-version 3.20.12     # resolve a requested version (default: current)
npx midway-skill lookup-docs --query "mcp"   # find relevant doc pages
npx midway-skill lookup-api --symbol "Configuration" --package "@midwayjs/core"
npx midway-skill lookup-packages --query "validation"
npx midway-skill lookup-changelog --package "@midwayjs/mcp"
```

**Version behavior:** the current major (v4) supports `docs + api + changelog`; historical majors support only `docs + changelog` (API lookups may return empty for old versions — expected).

**Integration tips for AI agents:** prefer `npx midway-skill lookup-docs --query "<topic>"` over guessing doc URLs — the output is JSON keyed to the installed version; run `lookup-api --symbol <name>` when a decorator/class signature is uncertain; run `lookup-changelog` when diagnosing a breaking change between upgrades.

Reference: [Midway Skill 使用](https://midwayjs.org/docs/skill_midway) (package: `@midwayjs/skill-midway`, binary: `midway-skill`)
