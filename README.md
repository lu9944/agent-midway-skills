# Midway Best Practices Skills

A structured skill for **Midway.js v4** (tested against v4.2.1) best practices, designed for AI agents and LLMs.

Uses the [Agent Skills](https://agentskills.io) open standard with **progressive disclosure** — the agent loads only the index (`SKILL.md`) at startup, then reads individual rule files on demand.

## Installation

Install this skill using [skills](https://github.com/vercel-labs/skills):

```bash
# GitHub shorthand
npx skills add lu9944/agent-midway-skills

# Install globally (available across all projects)
npx skills add lu9944/agent-midway-skills --global

# Install for specific agents
npx skills add lu9944/agent-midway-skills -a claude-code -a cursor
```

### Supported Agents

- OpenCode
- Claude Code
- Codex
- Cursor
- Roo Code
- Any agent supporting the Agent Skills standard

## Structure

```
agent-midway-skills/
├── SKILL.md              # Index: frontmatter + decision guide + rule catalog
├── references/           # 78 rule files (loaded on demand)
│   ├── arch-*.md         # Architecture (CRITICAL)
│   ├── di-*.md           # Dependency Injection (CRITICAL)
│   ├── error-*.md        # Error Handling (HIGH)
│   ├── security-*.md     # Security (HIGH)
│   ├── perf-*.md         # Performance (HIGH)
│   ├── test-*.md         # Testing (MEDIUM-HIGH)
│   ├── db-*.md           # Database & ORM (MEDIUM-HIGH)
│   ├── api-*.md          # API Design (MEDIUM)
│   ├── micro-*.md        # Microservices & Messaging (MEDIUM)
│   ├── devops-*.md       # DevOps & Deployment (LOW-MEDIUM)
│   ├── _sections.md      # Section metadata (contributor reference)
│   └── _template.md      # Template for creating new rules
├── metadata.json         # Project metadata
└── .github/workflows/    # CI validation
```

## How It Works

This skill follows the **progressive disclosure** pattern defined by the [Agent Skills specification](https://agentskills.io/specification):

1. **Discovery** — Agent loads only `name` + `description` (~100 tokens)
2. **Activation** — Agent reads `SKILL.md` index to find relevant rules
3. **Execution** — Agent reads specific `references/*.md` files as needed

This keeps context lean — the agent never loads all 230KB of rules at once.

## Creating a New Rule

1. Copy `references/_template.md` to `references/<prefix>-<name>.md`
2. Choose the appropriate prefix:
   - `arch-` — Architecture (CRITICAL)
   - `di-` — Dependency Injection (CRITICAL)
   - `error-` — Error Handling (HIGH)
   - `security-` — Security (HIGH)
   - `perf-` — Performance (HIGH)
   - `test-` — Testing (MEDIUM-HIGH)
   - `db-` — Database & ORM (MEDIUM-HIGH)
   - `api-` — API Design (MEDIUM)
   - `micro-` — Microservices & Messaging (MEDIUM)
   - `devops-` — DevOps & Deployment (LOW-MEDIUM)
3. Fill in the frontmatter and content
4. Add the rule to the decision guide in `SKILL.md`

## Impact Levels

| Level | Description |
|-------|-------------|
| CRITICAL | Violations cause runtime errors, security vulnerabilities, or architectural breakdown |
| HIGH | Significant impact on reliability, security, or maintainability |
| MEDIUM-HIGH | Notable impact on quality and developer experience |
| MEDIUM | Moderate impact on code quality and best practices |
| LOW-MEDIUM | Minor improvements for consistency and maintainability |

## Contributing

1. Use the correct filename prefix for your category
2. Follow the `_template.md` structure
3. Include clear incorrect vs correct examples with explanations
4. Add the new rule to the decision guide table in `SKILL.md`

## Acknowledgments

- Structure follows the [Agent Skills](https://agentskills.io) open standard
- v4 patterns cross-referenced with the [cool-admin-midway](https://github.com/cool-team-official/cool-admin-midway) reference project
