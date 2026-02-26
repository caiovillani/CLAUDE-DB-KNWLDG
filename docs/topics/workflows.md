# Common Workflows & Automation

## What It Is

Claude Code workflows are repeatable patterns for accomplishing development tasks — from exploring unfamiliar code to running parallel sessions. Understanding these workflows is the difference between using Claude as a chatbot and using it as an autonomous development partner.

## Key Concepts

- **Agentic loop**: Claude doesn't just answer questions — it reads files, runs commands, makes changes, and iterates autonomously
- **Context window as primary constraint**: Every file read, command output, and message consumes context. Performance degrades as it fills.
- **Plan Mode**: Read-only analysis mode (Shift+Tab to toggle, `--permission-mode plan` to start in it)
- **Headless mode**: Non-interactive execution via `claude -p "prompt"` for CI/scripts
- **Sessions**: Persistent, resumable conversations stored per project directory
- **Worktrees**: Isolated git working directories for parallel Claude sessions
- **Subagents**: Separate context windows for delegated tasks that don't pollute main context

## Configuration

### Plan Mode

| Method | Syntax |
|---|---|
| Toggle during session | `Shift+Tab` (cycles: Normal → Auto-Accept → Plan) |
| Start in Plan Mode | `claude --permission-mode plan` |
| Headless Plan Mode | `claude --permission-mode plan -p "analyze X"` |
| Default for project | `{"permissions": {"defaultMode": "plan"}}` in `.claude/settings.json` |

### Headless Mode

```bash
# Simple query
claude -p "explain what this project does"

# Structured output
claude -p "list all API endpoints" --output-format json

# Streaming
claude -p "analyze this log" --output-format stream-json

# Piped input
cat error.log | claude -p "explain the root cause" > output.txt

# With tool restrictions
claude -p "migrate this file" --allowedTools "Edit,Bash(git commit *)"
```

Output formats: `text` (default), `json` (full conversation log), `stream-json` (real-time).

### Session Management

| Command | Purpose |
|---|---|
| `claude --continue` | Resume most recent conversation in current directory |
| `claude --resume` | Interactive session picker |
| `claude --resume <name>` | Resume by session name |
| `claude --from-pr 123` | Resume session linked to a PR |
| `/resume` | Switch to different conversation mid-session |
| `/rename <name>` | Name current session for later retrieval |
| `/clear` | Reset context between unrelated tasks |
| `/compact <instructions>` | Summarize context with optional focus |
| `Esc+Esc` or `/rewind` | Rewind to previous checkpoint |

### Worktrees

```bash
# Named worktree
claude --worktree feature-auth   # Creates .claude/worktrees/feature-auth/

# Auto-named
claude --worktree                # Generates random name

# Subagent worktrees (in agent frontmatter)
# isolation: worktree
```

- Created at `<repo>/.claude/worktrees/<name>`
- Branch named `worktree-<name>`, based on default remote branch
- Auto-cleanup: no changes → removed; changes exist → prompted to keep/remove
- Add `.claude/worktrees/` to `.gitignore`

## Decision Guide

### When to Use Plan Mode

| Use Plan Mode | Skip Plan Mode |
|---|---|
| Multi-file changes | Single-line fix |
| Unfamiliar code | Familiar, obvious change |
| Complex architecture decisions | Simple, well-defined task |
| You want to iterate on approach | You can describe the diff in one sentence |

### When to Use Subagents

| Use subagents | Don't use subagents |
|---|---|
| Exploring large codebases (avoids main context pollution) | Simple, focused tasks |
| Code review (separate context = unbiased) | Tasks that need previous conversation context |
| Parallel investigation of independent questions | Tasks with heavy inter-dependencies |

### When to Use Headless Mode

| Use headless | Don't use headless |
|---|---|
| CI/CD pipelines | Exploratory work needing feedback |
| Pre-commit hooks | Complex multi-step tasks needing direction |
| Batch file processing | Tasks requiring human judgment mid-stream |
| Automated code review | |

### Session Management

| Situation | Action |
|---|---|
| Switching to unrelated task | `/clear` |
| Context is full of failed attempts | `/clear` + better initial prompt |
| Need to preserve some context | `/compact` with focus instructions |
| Interrupted mid-task, returning later | `claude --continue` |
| Multiple parallel workstreams | `claude --worktree` per stream |
| Long investigation polluting context | Delegate to subagent instead |

## Patterns

### Explore → Plan → Implement → Commit

The canonical four-phase workflow:
1. **Explore** (Plan Mode): Read files, understand architecture
2. **Plan** (Plan Mode): Design approach, get it right on paper
3. **Implement** (Normal Mode): Execute the plan, run tests
4. **Commit** (Normal Mode): Descriptive commit message, open PR

### Writer/Reviewer Pattern

Two parallel sessions:
- **Session A** (Writer): Implements the feature
- **Session B** (Reviewer): Reviews the implementation in fresh context (unbiased)
- Feed Session B's review back to Session A for fixes

### Fan-Out Pattern

For large migrations or batch operations:
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from X to Y. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

### Verification-First Pattern

Always provide Claude a way to check its own work:
- Include test commands
- Share expected outputs
- Ask for screenshot comparison (with Claude in Chrome)
- Specify "fix it and verify" rather than just "fix it"

### Rich Context Input

- `@file.ts` — reference files directly in prompts
- Paste images (Ctrl+V) — screenshots, mockups, diagrams
- Give URLs — allowlist domains via `/permissions`
- Pipe data — `cat error.log | claude`

## Anti-Patterns

- **Kitchen sink session**: Mixing unrelated tasks in one session → context gets noisy. Fix: `/clear` between tasks.
- **Correction spiral**: Correcting Claude 3+ times on the same issue → context polluted with failures. Fix: `/clear` + better prompt.
- **Infinite exploration**: Asking Claude to "investigate" without scope → reads hundreds of files. Fix: scope narrowly or use subagents.
- **Trust-then-verify gap**: Accepting plausible-looking output without verification. Fix: always provide verification criteria (tests, expected output).
- **The over-specified CLAUDE.md**: Too many instructions → Claude ignores the important ones. Fix: prune ruthlessly.
- **Skipping Plan Mode for complex changes**: Going straight to implementation → solving the wrong problem. Fix: plan first for multi-file changes.

## Source

Distilled from: `archive/# Common workflows.txt`, `archive/# Best Practices for Claude Code.txt`
