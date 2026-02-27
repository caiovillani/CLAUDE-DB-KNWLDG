# Memory System

## What It Is

Claude Code persists context across sessions through two mechanisms: **auto memory** (notes Claude writes for itself about project patterns, debugging insights, and preferences) and **CLAUDE.md files** (instructions you write for Claude to follow). Both load into context at session start, forming the foundation for consistent behavior without re-explaining context every time.

## Key Concepts

- **Auto memory vs CLAUDE.md**: Auto memory is Claude-authored (learnings, patterns, insights discovered during work). CLAUDE.md files are human-authored (instructions, rules, preferences you define). Both persist across sessions.
- **Memory hierarchy** (highest to lowest priority): Managed policy → Project CLAUDE.md → Project rules (`.claude/rules/`) → User CLAUDE.md → CLAUDE.local.md → Auto memory. More specific instructions take precedence over broader ones.
- **Loading behavior**: CLAUDE.md files in or above the working directory load in full at launch. CLAUDE.md files in child directories load on demand when Claude reads files in those subtrees. Auto memory loads only the first 200 lines of `MEMORY.md` into the system prompt; topic files are read on demand.
- **Project scoping**: The `<project>` path in auto memory derives from the git repository root, so all subdirectories within the same repo share one auto memory directory. Git worktrees get separate memory directories. Outside a git repo, the working directory path is used.

## Configuration

### Memory locations

| Memory Type | Location | Shared With | Loaded |
|---|---|---|---|
| Managed policy | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`; Linux: `/etc/claude-code/CLAUDE.md`; Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | All org users | At launch |
| Project memory | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team (via VCS) | At launch |
| Project rules | `./.claude/rules/*.md` | Team (via VCS) | At launch (unconditional) or on demand (path-scoped) |
| User memory | `~/.claude/CLAUDE.md` | Just you (all projects) | At launch |
| User rules | `~/.claude/rules/*.md` | Just you (all projects) | At launch |
| Local project memory | `./CLAUDE.local.md` | Just you (current project) | At launch |
| Auto memory | `~/.claude/projects/<project>/memory/` | Just you (per project) | First 200 lines of MEMORY.md at launch; topic files on demand |

### Auto memory toggle

```json
// Disable globally — ~/.claude/settings.json
{ "autoMemoryEnabled": false }

// Disable per project — .claude/settings.json
{ "autoMemoryEnabled": false }
```

```bash
# Environment variable override (takes precedence over all other settings)
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=1  # Force off
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=0  # Force on
```

Also toggleable interactively via `/memory` command during a session.

### CLAUDE.md imports

Use `@path/to/file` syntax inside any CLAUDE.md to import additional files. Relative paths resolve relative to the importing file, not the working directory.

```markdown
See @README for project overview and @package.json for available npm commands.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

- Max import depth: 5 hops (recursive imports supported).
- Imports inside markdown code spans and code blocks are ignored.
- First-time external imports trigger an approval dialog per project. Declining is permanent — imports remain disabled.
- For worktree setups, use home-directory imports so all worktrees share instructions: `@~/.claude/my-project-instructions.md`.

### Modular rules (`.claude/rules/`)

```
.claude/rules/
├── frontend/
│   ├── react.md
│   └── styles.md
├── backend/
│   ├── api.md
│   └── database.md
└── general.md
```

- All `.md` files discovered recursively.
- Rules without frontmatter load unconditionally.
- Path-scoped rules use YAML frontmatter with `paths` field and standard glob patterns:

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/**/*.{ts,tsx}"
  - "{src,lib}/**/*.ts"
---

# API Development Rules
- All API endpoints must include input validation
```

- Brace expansion supported for matching multiple extensions or directories.
- Symlinks supported — useful for sharing rules across projects.

### Additional directories

```bash
# Load files from another directory
claude --add-dir ../shared-config

# Also load that directory's CLAUDE.md and rules
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

By default, `--add-dir` does **not** load CLAUDE.md files from additional directories. The env var enables it.

## Decision Guide

| If you need... | Use this |
|---|---|
| Team-shared project instructions (build commands, architecture, conventions) | `./CLAUDE.md` or `./.claude/CLAUDE.md` — commit to VCS |
| Personal preferences across all projects (style, shortcuts, workflows) | `~/.claude/CLAUDE.md` |
| Personal preferences scoped to one project (sandbox URLs, test data) | `./CLAUDE.local.md` — auto-gitignored |
| Topic-specific rules with optional path scoping | `.claude/rules/<topic>.md` with optional `paths` frontmatter |
| Org-wide standards enforced on all machines | Managed policy CLAUDE.md via MDM/Group Policy |
| Cross-project shared rules | Symlinks in `.claude/rules/` pointing to shared files |
| Personal rules across all projects | `~/.claude/rules/*.md` |
| Let Claude learn patterns organically | Auto memory (enabled by default) |
| Share personal instructions across worktrees | `@~/.claude/my-project-instructions.md` import in CLAUDE.md |

## Patterns

- **Bootstrap with `/init`**: Generates a starter CLAUDE.md for your codebase. Refine iteratively from there.
- **Keep MEMORY.md under 200 lines**: Only the first 200 lines load at startup. Move detailed notes into topic files (`debugging.md`, `api-conventions.md`, etc.) and use MEMORY.md as a concise index.
- **Use symlinks for cross-project shared rules**: `ln -s ~/shared-claude-rules .claude/rules/shared` or `ln -s ~/company-standards/security.md .claude/rules/security.md`.
- **Use `@imports` for worktree portability**: `CLAUDE.local.md` only exists in one worktree. Import from home directory instead.
- **User-level rules in `~/.claude/rules/`**: Personal preferences that apply everywhere without touching project files.
- **Be specific in instructions**: "Use 2-space indentation" beats "Format code properly."
- **Structure with bullet points and headings**: Group related memories under descriptive markdown headings.
- **Tell Claude to remember explicitly**: "remember that we use pnpm, not npm" or "save to memory that API tests require local Redis."
- **Review periodically**: Update memories as the project evolves to prevent stale context.
- **Organize rules into subdirectories**: `frontend/`, `backend/`, `general.md` for large projects.

## Anti-Patterns

- **Bloating CLAUDE.md with inferrable information**: Don't restate what Claude can discover from code, package.json, or directory structure. Focus on non-obvious conventions and decisions.
- **Not pruning auto memory**: Over time, auto memory accumulates stale or contradictory entries. Review and prune periodically via `/memory`.
- **Personal preferences in project CLAUDE.md**: Use `CLAUDE.local.md` or `~/.claude/CLAUDE.md` for personal preferences. Project CLAUDE.md is shared with the team.
- **Exceeding 200 lines in MEMORY.md without topic files**: Content beyond line 200 is silently dropped at startup. Offload detail into topic files.
- **Circular imports in `@path` references**: While max depth of 5 prevents infinite loops, circular references waste context budget and cause confusion.
- **Declining the import approval dialog without understanding consequences**: The decision is permanent per project — declined imports stay disabled with no re-prompt.
- **Using conditional `paths` rules when rules apply broadly**: Only add `paths` frontmatter when rules truly apply to specific file types. Unconditional rules are simpler.

## Source

Distilled from: `archive/# Manage Claude's memory.txt`
