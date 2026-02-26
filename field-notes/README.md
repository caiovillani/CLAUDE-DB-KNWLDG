# Field Notes

This directory contains battle-tested patterns and insights from real-world Claude Code usage across multiple workspaces and projects.

## Provenance & Confidence

**These are NOT official Anthropic documentation.** Field notes represent practical experience — things that work well (or fail) in specific contexts. They carry a **medium confidence level** and should be weighed accordingly.

### Evidence Hierarchy

| Source | Confidence | Use |
|---|---|---|
| Official docs (`docs/topics/`) | High | Primary basis for all recommendations |
| **Field notes (this directory)** | **Medium** | **Supplementary — always marked as experiential** |
| Archive (`archive/`) | Raw | Verification backstop |

### How to Read Field Notes

- Every field note includes the **context** where it was observed (project type, team size, complexity)
- Patterns marked **"consistent"** have been validated across 3+ projects
- Patterns marked **"emerging"** have been observed in 1-2 projects and need more validation
- Anti-patterns include the **failure mode** that was observed

### How to Contribute

When adding a field note:

1. Describe the **context** (what kind of project, what problem)
2. Describe the **pattern** (what was done)
3. Describe the **outcome** (what happened)
4. Rate confidence: **consistent** (3+ projects) or **emerging** (1-2 projects)
5. Note any **caveats** or conditions where this might not apply

## Directory Structure

Files here are organized by topic, mirroring `docs/topics/`:

```
field-notes/
├── README.md          ← You are here
├── claude-md.md       ← CLAUDE.md design lessons
├── hooks.md           ← Hook patterns from real projects
├── workflows.md       ← Workflow insights
└── ...
```

Files will be added as real-world experience accumulates.
