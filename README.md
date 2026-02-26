# CLAUDE-DB-KNWLDG

Expert knowledge base on Claude Code core practices. Open a Claude Code session in this directory and get a specialist consultant on CLAUDE.md design, hooks, permissions, settings, workflows, and memory.

## What this is

A structured workspace that turns Claude Code into an expert advisor on its own configuration and best practices. It has three knowledge layers:

| Layer | Location | Confidence |
|---|---|---|
| **Curated docs** | `docs/topics/`, `docs/guides/`, `docs/patterns/` | High — distilled from Anthropic documentation |
| **Field notes** | `field-notes/` | Medium — battle-tested, context-specific |
| **Archive** | `archive/` | Raw — unprocessed official documentation |

## Quick start

```bash
cd CLAUDE-DB-KNWLDG
claude
```

Then ask anything about Claude Code configuration:

```
How should I structure CLAUDE.md for a TypeScript monorepo with 5 developers?
```

```
Review this CLAUDE.md: [paste your config]
```

```
What's the syntax for path-scoped rules in .claude/rules/?
```

## Topics covered (v1)

| Topic | File |
|---|---|
| CLAUDE.md design | `docs/topics/claude-md.md` |
| Hooks system | `docs/topics/hooks.md` |
| Memory system | `docs/topics/memory.md` |
| Permissions | `docs/topics/permissions.md` |
| Settings hierarchy | `docs/topics/settings.md` |
| Workflows & automation | `docs/topics/workflows.md` |

**Also includes:** [New project setup guide](docs/guides/new-project-setup.md) | [CLAUDE.md patterns](docs/patterns/claude-md-patterns.md) | [Common mistakes](docs/patterns/common-mistakes.md)

## Use from another project

```bash
# Add as reference directory (read-only access to all files)
claude --add-dir /path/to/CLAUDE-DB-KNWLDG

# Also load the expert consultant persona
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir /path/to/CLAUDE-DB-KNWLDG
```

See the [full user guide](docs/guides/user-guide.md) for all 16 use cases, interaction patterns, and cross-workspace methods.

## How it works

The `CLAUDE.md` at the root defines three interaction modes:

- **CONSULT** — Describe a goal or problem, get tailored recommendations with trade-offs
- **COACH** — Paste a config file, get a systematic audit with prioritized fixes
- **REFERENCE** — Ask a direct question, get a precise answer with examples

The agent cites which knowledge layer every recommendation comes from and proactively flags related issues you didn't ask about.

## Scope

**In scope (v1):** CLAUDE.md, hooks, permissions, settings, workflows, memory.

**Out of scope (v1):** Plugins, MCP servers, skills/commands, multi-agent orchestration, Claude API/SDK. Raw docs for some of these exist in `archive/` for manual reference.

## Contributing field notes

If you discover patterns from real-world Claude Code usage, add them to `field-notes/` following the [contribution protocol](field-notes/README.md).
