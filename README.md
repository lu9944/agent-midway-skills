# Midway Best Practices

A structured repository for creating and maintaining **Midway.js v4** Best Practices optimized for agents and LLMs.

## Installation

Install this skill using [skills](https://github.com/vercel-labs/skills):

```bash
# GitHub shorthand
npx skills add <your-org>/agent-midway-skills

# Install globally (available across all projects)
npx skills add <your-org>/agent-midway-skills --global

# Install for specific agents
npx skills add <your-org>/agent-midway-skills -a claude-code -a cursor
```

### Supported Agents

- Claude Code
- OpenCode
- Codex
- Cursor
- Antigravity
- Roo Code

## Structure

- `rules/` - Individual rule files (one per rule)
  - `_sections.md` - Section metadata (titles, impacts, descriptions)
  - `_template.md` - Template for creating new rules
  - `area-description.md` - Individual rule files
- `scripts/` - Build scripts and utilities
- `metadata.json` - Document metadata (version, organization, abstract)
- __`AGENTS.md`__ - Compiled output (generated)

## Getting Started

1. Install dependencies:
   ```bash
   cd scripts && npm install
   ```

2. Build AGENTS.md from rules:
   ```bash
   npm run build
   # or
   ./scripts/build.sh
   ```

## Creating a New Rule

1. Copy `rules/_template.md` to `rules/area-description.md`
2. Choose the appropriate area prefix:
   - `arch-` for Architecture (Section 1)
   - `di-` for Dependency Injection (Section 2)
   - `error-` for Error Handling (Section 3)
   - `security-` for Security (Section 4)
   - `perf-` for Performance (Section 5)
   - `test-` for Testing (Section 6)
   - `db-` for Database & ORM (Section 7)
   - `api-` for API Design (Section 8)
   - `micro-` for Microservices & Messaging (Section 9)
   - `devops-` for DevOps & Deployment (Section 10)
3. Fill in the frontmatter and content
4. Ensure you have clear examples with explanations
5. Run the build script to regenerate AGENTS.md

## Rule File Structure

Each rule file should follow this structure:

```markdown
---
title: Rule Title Here
impact: MEDIUM
impactDescription: Optional description of impact (e.g., "20-50% improvement")
tags: tag1, tag2, tag3
---

## Rule Title Here

Brief explanation of the rule and why it matters.

**Incorrect (description of what's wrong):**

```typescript
// Bad code example
```

**Correct (description of what's right):**

```typescript
// Good code example
```

Reference: [Midway Documentation](https://midwayjs.org)
```

## File Naming Convention

- Files starting with `_` are special (excluded from build)
- Rule files: `area-description.md` (e.g., `di-scope-awareness.md`)
- Section is automatically inferred from filename prefix
- Rules are sorted alphabetically by **filename** within each section
- IDs (e.g., 1.1, 1.2) are auto-generated during build

## Impact Levels

| Level | Description |
|-------|-------------|
| CRITICAL | Violations cause runtime errors, security vulnerabilities, or architectural breakdown |
| HIGH | Significant impact on reliability, security, or maintainability |
| MEDIUM-HIGH | Notable impact on quality and developer experience |
| MEDIUM | Moderate impact on code quality and best practices |
| LOW-MEDIUM | Minor improvements for consistency and maintainability |

## Scripts

- `npm run build` (in scripts/) - Compile rules into AGENTS.md

## Contributing

When adding or modifying rules:

1. Use the correct filename prefix for your section
2. Follow the `_template.md` structure
3. Include clear bad/good examples with explanations
4. Add appropriate tags
5. Run the build script to regenerate AGENTS.md
6. Rules are automatically sorted by filename - no need to manage numbers!

## Acknowledgments

- Structure inspired by the [agent-nestjs-skills](https://github.com/Kadajett/agent-nestjs-skills) skill
- v4 patterns cross-referenced with the [cool-admin-midway](https://github.com/cool-team-official/cool-admin-midway) reference project
- Compatible with [skills](https://github.com/vercel-labs/skills) for easy installation across coding agents
