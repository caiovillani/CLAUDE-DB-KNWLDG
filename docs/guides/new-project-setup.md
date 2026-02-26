# Guide: Setting Up a New Claude Code Project

A step-by-step walkthrough for configuring a new project to get the most out of Claude Code from day one.

## Prerequisites

- Claude Code CLI installed and authenticated
- A project directory (ideally a git repository)

## Step 1: Bootstrap CLAUDE.md

```bash
cd /path/to/your/project
claude
> /init
```

`/init` analyzes your codebase and generates a starter CLAUDE.md with detected build systems, test frameworks, and code patterns. Review and refine it.

**What to add after `/init`:**
- Build/test/lint commands specific to your project
- Code style rules that differ from language defaults
- Branch naming and PR conventions
- Environment setup quirks (required env vars, local services)
- Common gotchas only a human would know

**What to remove:**
- Anything Claude already gets right without the instruction
- Generic advice ("write clean code", "follow best practices")

→ See: `docs/topics/claude-md.md` for full CLAUDE.md design guide

## Step 2: Configure Permissions

Start conservative, then loosen as you build trust:

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Bash(git push *)",
      "Bash(rm -rf *)",
      "Read(.env)"
    ]
  }
}
```

Iterate: if you find yourself approving the same command repeatedly, add it to `allow`.

→ See: `docs/topics/permissions.md` for full permission system reference

## Step 3: Set Up Modular Rules (Optional)

For larger projects, organize rules by domain:

```
.claude/
├── CLAUDE.md              # Core project instructions
├── settings.json          # Permissions and settings
└── rules/
    ├── code-style.md      # Coding conventions
    ├── testing.md         # Test patterns and commands
    └── api-design.md      # API conventions
```

Use `paths` frontmatter to scope rules to specific file types:

```markdown
---
paths:
  - "src/**/*.{ts,tsx}"
---
# TypeScript Rules
- Use strict mode
- Prefer interfaces over type aliases for object shapes
```

→ See: `docs/topics/memory.md` for modular rules reference

## Step 4: Configure Personal Preferences

Create `CLAUDE.local.md` for your personal project preferences (auto-added to `.gitignore`):

```markdown
# My Preferences
- I prefer verbose explanations when exploring unfamiliar code
- My sandbox URL: http://localhost:3001
- Use 'bun' instead of 'npm' for my local dev
```

And/or use `~/.claude/CLAUDE.md` for preferences that apply across ALL projects.

## Step 5: Set Up Essential Hooks (Optional)

Start with high-value, low-risk hooks:

**Lint after file edits:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint --fix $CLAUDE_FILE_PATH 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**Desktop notifications (Windows):**
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -Command \"[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('Claude Code needs your attention', 'Claude Code')\""
          }
        ]
      }
    ]
  }
}
```

→ See: `docs/topics/hooks.md` for full hooks reference

## Step 6: Validate Your Setup

Start a new session and test:

1. Ask Claude a question about your codebase — does it understand the project?
2. Ask Claude to make a small change — do permissions work as expected?
3. Ask Claude about your conventions — does it follow your CLAUDE.md rules?
4. Check that hooks fire correctly on relevant actions

## Step 7: Commit Team Configuration

```bash
git add CLAUDE.md .claude/settings.json .claude/rules/
git commit -m "Add Claude Code project configuration"
```

**Don't commit:** `CLAUDE.local.md`, `.claude/settings.local.json`, personal preferences.

## Checklist

- [ ] CLAUDE.md created and refined (not just `/init` output)
- [ ] Permissions configured (allow safe commands, deny dangerous ones)
- [ ] CLAUDE.local.md for personal preferences
- [ ] Modular rules for complex projects
- [ ] Essential hooks set up
- [ ] Team config committed to git
- [ ] Setup validated with a test session
