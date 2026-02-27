# Workflow Field Notes

## Cross-Workspace Integration Strategies

**Context:** Integrating CLAUDE-DB-KNWLDG as an expert knowledge base into other Claude Code project workspaces. Evaluated three native Claude Code mechanisms for cross-workspace access.

**Pattern:** Three integration methods exist, each suited to different usage moments:

### Method 1: `--add-dir` (ad-hoc deep consultation)

```bash
# Read access to files, no persona
claude --add-dir /path/to/CLAUDE-DB-KNWLDG

# Full expert persona loaded
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir /path/to/CLAUDE-DB-KNWLDG
```

Best for: initial project setup, deep troubleshooting, one-time audits. The env var loads both CLAUDE.md files, making the session a hybrid project expert + Claude Code expert.

### Method 2: Separate terminal (clean isolation)

Two terminals — one for your project, one for the knowledge base. No context pollution. Manual copy-paste between them.

Best for: periodic config audits via COACH mode. Paste your configs into the expert terminal, get the report, apply fixes in your project terminal.

### Method 3: `@import` in your project's CLAUDE.md (persistent guardrails)

```markdown
# In your project's CLAUDE.md
@~/CLAUDE-DB-KNWLDG/docs/patterns/common-mistakes.md
```

Best for: always-on awareness of failure patterns. Use `~/` prefix for worktree portability. Import selectively — only pattern/anti-pattern files, not full topic files.

### Recommended integration strategy

| Moment | Method | Why |
|---|---|---|
| New project setup | Method 1 with env var | One-time deep consultation, then disconnect |
| Daily development | Method 3 (selective imports) | Passive guardrails, minimal context cost |
| Periodic audits | Method 2 (separate terminal) | Clean separation, full COACH mode |
| Team onboarding | Shell alias for Method 1 | `alias claude-expert='CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ~/CLAUDE-DB-KNWLDG'` |

### Anti-patterns

- **Importing entire `docs/topics/` directory** — 1,279+ lines loading every session. Wastes context budget. Import only patterns you reference constantly.
- **Copying topic files into other projects** — They go stale immediately. Use imports or `--add-dir` to reference the single source of truth.
- **Leaving `--add-dir` on for everyday coding** — The expert persona and knowledge files consume context that's better used for your actual project code.

**Outcome:** The light-touch approach (selective `@import` for guardrails + `--add-dir` on demand) balances knowledge access against context budget. Teams report fewer CLAUDE.md anti-patterns when `common-mistakes.md` is imported as a passive guardrail.

**Confidence:** emerging

**Caveats:**
- Absolute paths in `@import` break if the workspace moves. Use `~/` home-relative paths.
- First `@import` from an external directory triggers a one-time approval dialog per project. Declining is permanent.
- With the env var, both CLAUDE.md files load — if your project CLAUDE.md has conflicting instructions, the project-level one takes precedence per Claude Code's hierarchy.
