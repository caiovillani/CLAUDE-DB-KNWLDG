# CLAUDE-DB-KNWLDG v1 — Claude Code Expert Workspace

**Date:** 2026-02-26
**Status:** Approved
**Scope:** Claude Code core (CLAUDE.md, hooks, permissions, settings, workflows, memory)

---

## 1. Purpose

A standalone workspace that acts as an expert consultant and proactive coach for Claude Code core practices. Two consumers:

- **Human user** — opens a session to get guidance on Claude Code topics
- **Other Claude instances** — directed to consult this workspace's patterns and principles

## 2. Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Architecture | Standalone workspace (B), hybrid later | Build robust knowledge foundation before distributing via plugin |
| v1 Scope | Claude Code core only | Focus: CLAUDE.md, hooks, permissions, settings, workflows, memory |
| Knowledge structure | Layered (curated + reference + field notes) | Serves both human and Claude consumers at appropriate depth |
| Interaction model | Expert consultant + proactive coach | Reactive expertise + proactive gap detection |
| Knowledge boundary | Official docs + battle-tested patterns, separated | Maps to evidence hierarchy — provenance always explicit |
| Raw docs | Keep as archive (B), replace progressively (C) over time | Verification backstop now, redundant later |

## 3. Directory Structure

```
CLAUDE-DB-KNWLDG/
│
├── CLAUDE.md                    # Agent brain — identity, principles, routing
│
├── archive/                     # Raw official docs (original .txt files)
│   ├── Advanced setup.txt
│   ├── Hooks reference.txt
│   └── ...
│
├── docs/
│   ├── plans/                   # Design and implementation plans
│   │
│   ├── topics/                  # Deep reference per domain
│   │   ├── claude-md.md         # CLAUDE.md design & architecture
│   │   ├── hooks.md             # Hooks system — types, lifecycle, patterns
│   │   ├── permissions.md       # Permission model & configuration
│   │   ├── settings.md          # Settings hierarchy & options
│   │   ├── workflows.md         # Common workflows & automation
│   │   └── memory.md            # Memory system — auto, project, global
│   │
│   ├── guides/                  # Task-oriented walkthroughs
│   │   ├── new-project-setup.md
│   │   ├── hook-design.md
│   │   └── memory-strategy.md
│   │
│   └── patterns/                # Tested patterns & anti-patterns
│       ├── claude-md-patterns.md
│       ├── hook-patterns.md
│       └── common-mistakes.md
│
├── field-notes/                 # Battle-tested experience (user's own)
│   └── README.md                # Provenance & confidence explanation
│
└── .claude/
    └── settings.json            # Workspace-level Claude settings
```

## 4. CLAUDE.md Architecture

Six sections, written FOR Claude (precise instructions, not documentation):

1. **Identity & Role** — Expert consultant + proactive coach on Claude Code core
2. **Knowledge Architecture** — Three layers with explicit provenance rules
3. **Interaction Modes** — CONSULT, COACH, REFERENCE with routing logic
4. **Routing Logic** — Context-based mode detection with fallback to clarification
5. **Quality Standards** — Source traceability, uncertainty flagging, anti-patterns
6. **Scope Boundaries** — What's in v1, what's deferred, how to handle out-of-scope

## 5. Topic File Template

Each `docs/topics/` file follows:

```markdown
# [Topic Name]

## What It Is
## Key Concepts
## Configuration
## Decision Guide
## Patterns
## Anti-Patterns
## Source
```

## 6. Interaction Flow

```
User opens session → CLAUDE.md loads → agent knows role
  → User states intent (or agent asks)
    → "Help with X"          → CONSULT mode
    → "Review my config"     → COACH mode
    → "How does X work?"     → REFERENCE mode
  → Agent reads relevant docs/topics/ + docs/patterns/
  → Delivers guidance with source attribution
  → Proactively flags related gaps or risks
```

## 7. Explicitly Deferred (Future Phases)

- Plugin infrastructure (hybrid delivery)
- Automated cross-workspace auditing
- MCP server exposure
- Skills/commands beyond CLAUDE.md
- Extensibility topics (plugins, MCP, agents, skills)
- Multi-agent orchestration patterns

## 8. Success Criteria for v1

- [ ] CLAUDE.md enables effective expert consultation in any session
- [ ] All 6 core topics covered with curated reference content
- [ ] At least 3 guides and 3 pattern files operational
- [ ] Field-notes layer scaffolded with clear provenance rules
- [ ] Raw docs preserved in archive/ as verification backstop
- [ ] Threshold for hybrid migration defined
