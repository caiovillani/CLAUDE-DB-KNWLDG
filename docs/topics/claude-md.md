# CLAUDE.md Design & Architecture

## What It Is

CLAUDE.md is a special markdown file that Claude Code reads at the start of every conversation. It provides persistent instructions, rules, and context that Claude can't infer from code alone. It's the single most impactful configuration file — a well-crafted CLAUDE.md dramatically improves Claude's behavior across all sessions.

## Key Concepts

- **Always loaded**: CLAUDE.md is injected into Claude's system prompt at session start. It's not optional reading — Claude sees it every time.
- **Multiple locations**: Files can exist at project root (`./CLAUDE.md`), in `.claude/CLAUDE.md`, user home (`~/.claude/CLAUDE.md`), parent directories, and child directories.
- **Hierarchy**: More specific instructions take precedence. Child directory CLAUDE.md overrides project root, which overrides user-level.
- **On-demand loading**: CLAUDE.md files in child directories only load when Claude works with files in those directories.
- **Import system**: `@path/to/file` syntax lets CLAUDE.md reference external files (max depth 5 hops).
- **Not documentation**: CLAUDE.md is instructions FOR Claude, not documentation ABOUT the project. This distinction matters.

## Configuration

### File Locations (Priority Order)

| Location | Scope | Shared | Loaded |
|---|---|---|---|
| Managed policy path* | Organization-wide | All users | Always |
| `~/.claude/CLAUDE.md` | All your projects | Just you | Always |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project (team) | Via git | Always |
| `./.claude/rules/*.md` | Project (modular) | Via git | Always |
| `./CLAUDE.local.md` | Project (personal) | Just you | Always |
| `./subdir/CLAUDE.md` | Subdirectory | Via git | On-demand |

*Managed policy: macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`, Linux `/etc/claude-code/CLAUDE.md`, Windows `C:\Program Files\ClaudeCode\CLAUDE.md`

### Import Syntax

```markdown
# In CLAUDE.md
See @README.md for project overview.
Git workflow: @docs/git-instructions.md
Personal: @~/.claude/my-project-instructions.md
```

- Relative paths resolve relative to the importing file, not cwd
- Max recursive depth: 5 hops
- Not evaluated inside code spans or code blocks
- First encounter triggers an approval dialog (one-time per project, permanent if declined)

### Modular Rules (.claude/rules/)

```
.claude/rules/
├── frontend/
│   ├── react.md        # React-specific rules
│   └── styles.md       # CSS conventions
├── backend/
│   ├── api.md          # API design rules
│   └── database.md     # DB conventions
└── general.md          # Universal rules
```

- All `.md` files discovered recursively
- Same priority as `.claude/CLAUDE.md`
- Supports path-specific scoping via YAML frontmatter:

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "lib/**/*.ts"
---
# API Rules
- All endpoints must include input validation
```

- Rules without `paths` apply unconditionally
- Glob patterns: `*` matches single dir, `**` matches recursively, `{ts,tsx}` brace expansion supported
- Symlinks supported for cross-project shared rules
- User-level rules at `~/.claude/rules/` load before project rules

### CLAUDE.local.md

- Automatically added to `.gitignore`
- Personal project-specific preferences
- Loaded alongside CLAUDE.md, same priority level

## Decision Guide

| You want to... | Use |
|---|---|
| Share team conventions with all developers | `./CLAUDE.md` or `.claude/CLAUDE.md` (commit to git) |
| Set personal preferences across all projects | `~/.claude/CLAUDE.md` |
| Set personal project-specific preferences | `./CLAUDE.local.md` |
| Organize complex project rules by topic | `.claude/rules/*.md` with subdirectories |
| Scope rules to specific file types | `.claude/rules/` with `paths` frontmatter |
| Share rules across multiple projects | Symlinks in `.claude/rules/` or `@~/.claude/shared.md` imports |
| Enforce organization-wide standards | Managed policy CLAUDE.md |
| Store info Claude only needs sometimes | Skills (`.claude/skills/`) instead of CLAUDE.md |
| Share personal instructions across worktrees | `@~/.claude/my-project-instructions.md` import |

## Patterns

### Content Strategy: What to Include vs Exclude

**Include:**
- Bash commands Claude can't guess (build, test, lint commands specific to your project)
- Code style rules that differ from language defaults
- Testing instructions and preferred test runners
- Repository etiquette (branch naming, PR conventions)
- Architectural decisions specific to your project
- Developer environment quirks (required env vars, special setup)
- Common gotchas or non-obvious behaviors

**Exclude:**
- Anything Claude can figure out by reading code
- Standard language conventions Claude already knows
- Detailed API documentation (link to docs instead)
- Information that changes frequently
- Long explanations or tutorials
- File-by-file descriptions of the codebase
- Self-evident practices like "write clean code"

### Effective Writing Style

- Write instructions FOR Claude, not about the project
- Be specific: "Use 2-space indentation" beats "Format code properly"
- Use emphasis for critical rules: "IMPORTANT:" or "YOU MUST"
- Format as bullet points under descriptive headings
- Test effectiveness: if Claude ignores a rule, the file is probably too long
- If Claude asks questions answered in CLAUDE.md, the phrasing may be ambiguous

### Bootstrap and Iterate

1. Run `/init` to generate a starter based on your project
2. Use it for a few sessions
3. Notice where Claude goes wrong — add rules for those cases
4. Notice where Claude already does the right thing — remove those rules
5. Prune regularly (treat like code review)

### Emphasis Tuning

Add emphasis to improve adherence to critical rules:
```markdown
# IMPORTANT: Always run type checking after code changes
# YOU MUST use the project's custom test runner, not jest directly
```

## Anti-Patterns

- **The bloated CLAUDE.md**: Too many instructions → Claude ignores the important ones. If it's longer than ~50-80 lines, audit ruthlessly.
- **Documentation disguised as instructions**: CLAUDE.md is not a wiki. Move reference material to skills or linked docs.
- **Duplicating what code already shows**: Don't describe file structure or obvious patterns — Claude reads code.
- **Static CLAUDE.md**: Never reviewing or pruning. Rules become stale as the project evolves.
- **Everything in one file**: For complex projects, use `.claude/rules/` to organize by topic instead of one massive file.
- **Personal preferences in team files**: Use `CLAUDE.local.md` for personal settings, not the committed `CLAUDE.md`.
- **Declining import approval permanently**: The first-time approval for `@imports` is permanent if declined. Understand what you're approving.

## Source

Distilled from: `archive/# Manage Claude's memory.txt`, `archive/# Best Practices for Claude Code.txt`
