# Permission System

## What It Is

Claude Code uses a fine-grained permission system to control exactly what the agent can access and do. Permission rules can be checked into version control for team-wide enforcement, layered with OS-level sandboxing for defense-in-depth, and customized per-user or per-project. The system evaluates rules in a strict deny-first order so that restrictive policies always win.

## Key Concepts

- **Tiered permission system**: Tools fall into three tiers — read-only (file reads, Grep: no approval), Bash commands (approval required, remembered permanently per project+command), and file modification (approval required, remembered until session end).
- **Rule types — deny > ask > allow**: Three rule types evaluated in strict order. Deny always wins. First matching rule short-circuits evaluation.
- **Permission modes**: Set via `defaultMode` in settings:
  - `default` — prompts on first use of each tool.
  - `acceptEdits` — auto-approves file edit permissions for the session.
  - `plan` — read/analyze only, no modifications or commands.
  - `dontAsk` — auto-denies everything not pre-approved in `permissions.allow`.
  - `bypassPermissions` — skips all prompts (containers/VMs only).
- **Specifiers**: Fine-grained control via `Tool(specifier)` syntax — Bash commands, file paths, domains, MCP tools, subagents.
- **Complementary sandboxing**: Permissions control Claude's decision layer; sandboxing enforces OS-level restrictions on Bash processes. Use both together. Filesystem sandbox restrictions reuse Read/Edit deny rules; network sandbox uses `allowedDomains` alongside WebFetch domain rules.
- **Settings precedence**: managed > CLI args > local project > shared project > user. A deny in project settings overrides an allow in user settings.

## Configuration

### Permission rule syntax

Rules follow the format `Tool` or `Tool(specifier)`. Using just the tool name matches all uses of that tool.

```jsonc
// settings.json (user, project, or managed)
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",      // glob wildcard
      "Bash(git commit *)",
      "Read"                   // all file reads
    ],
    "deny": [
      "Bash(git push *)",
      "Edit(.env)"
    ]
  }
}
```

### Bash wildcard patterns

- `*` is a glob wildcard. Can appear at any position (start, middle, end).
- **Word boundary rule**: a space before `*` enforces word boundary. `Bash(ls *)` matches `ls -la` but NOT `lsof`. `Bash(ls*)` matches both.
- `Bash(*)` is equivalent to `Bash` — matches all commands.
- Claude is aware of shell operators: `Bash(safe-cmd *)` will NOT approve `safe-cmd && other-cmd`.
- Legacy `:*` suffix is deprecated; use ` *` instead.

### Read and Edit rules (gitignore spec)

| Pattern      | Meaning                          | Example                            |
|:-------------|:---------------------------------|:-----------------------------------|
| `//path`     | Absolute filesystem path         | `Read(//Users/alice/secrets/**)` |
| `~/path`     | Relative to home directory       | `Read(~/Documents/*.pdf)`        |
| `/path`      | Relative to project root         | `Edit(/src/**/*.ts)`             |
| `path`/`./path` | Relative to current directory | `Read(*.env)`                    |

Key: `*` matches files in one directory; `**` matches recursively. A pattern like `/Users/alice/file` is NOT absolute — use `//Users/alice/file`.

### WebFetch domain rules

```jsonc
"allow": ["WebFetch(domain:example.com)"]
```

### MCP tool rules

```jsonc
"allow": [
  "mcp__puppeteer",                        // all tools from puppeteer server
  "mcp__puppeteer__*",                     // same, wildcard syntax
  "mcp__puppeteer__puppeteer_navigate"     // single specific tool
]
```

### Task / subagent rules

```jsonc
"deny": [
  "Task(Explore)",    // disable Explore subagent
  "Task(Plan)"        // disable Plan subagent
]
```

Also controllable via `--disallowedTools` CLI flag.

### Working directories

- `--add-dir <path>` at startup.
- `/add-dir` during a session.
- `additionalDirectories` in settings for persistence.

Files in additional directories follow the same permission rules as the launch directory.

### Managed settings (organization control)

Managed settings cannot be overridden by user or project settings. Key managed-only settings:

| Setting | Effect |
|:--------|:-------|
| `disableBypassPermissionsMode` | `"disable"` blocks `bypassPermissions` mode and `--dangerously-skip-permissions` |
| `allowManagedPermissionRulesOnly` | `true` prevents user/project from defining allow/ask/deny rules |
| `allowManagedHooksOnly` | `true` restricts hooks to managed and SDK hooks only |
| `allowManagedMcpServersOnly` | `true` limits MCP servers to managed allowlist |
| `sandbox.network.allowManagedDomainsOnly` | `true` restricts domains to managed allowlist |
| `allow_remote_sessions` | `false` blocks Remote Control and web sessions |

### /permissions UI command

Use `/permissions` inside Claude Code to view all active permission rules and which settings file each rule comes from.

## Decision Guide

| Goal | Action |
|:-----|:-------|
| Allow common safe commands | Add specific patterns to `permissions.allow` (e.g., `Bash(npm run *)`, `Bash(git commit *)`) |
| Block dangerous commands | Add to `permissions.deny` (e.g., `Bash(git push *)`, `Bash(rm -rf *)`) |
| Protect sensitive files | Deny rules for paths: `Edit(.env)`, `Edit(credentials.*)`, `Read(//etc/shadow)` |
| Full isolation | Combine sandboxing + permission deny rules for defense-in-depth |
| CI/automation | `bypassPermissions` mode inside isolated containers only; disable it via managed settings everywhere else |
| Reduce approval interruptions | Use `acceptEdits` mode or add targeted allow rules for frequent tools |
| Autonomous workflows | `dontAsk` mode with a curated `permissions.allow` list — everything else auto-denied |
| Organization-wide policy | Deploy managed settings via MDM; use `allowManagedPermissionRulesOnly` to lock down |

## Patterns

**Allow npm/git commands, deny git push:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Bash(git * main)",
      "Bash(* --version)",
      "Bash(* --help *)"
    ],
    "deny": [
      "Bash(git push *)"
    ]
  }
}
```

**Sandbox + permissions for defense-in-depth:** Permission deny rules block Claude from attempting access; sandbox restrictions prevent Bash child processes from reaching resources even if a prompt injection bypasses Claude's decision-making. Filesystem sandbox restrictions reuse Read/Edit deny rules — no separate config needed.

**PreToolUse hooks for extended permission evaluation:** Register a hook that runs before the permission system. The hook output can approve or deny the tool call, enabling custom validation logic (e.g., URL allowlists, argument checks) beyond what static patterns support.

**dontAsk mode with curated allow list:** Set `defaultMode: "dontAsk"` and define a precise `permissions.allow` list. Everything not explicitly allowed is auto-denied. Ideal for autonomous or batch workflows where no human is present to approve prompts.

## Anti-Patterns

- **Using Bash patterns to constrain URLs**: `Bash(curl http://github.com/ *)` is fragile — fails on option reordering, protocol variants, redirects, variables, extra spaces. Use `WebFetch(domain:...)` rules instead, or deny Bash network tools entirely and route through WebFetch.
- **`bypassPermissions` outside isolated containers**: Disables all safety checks. Always pair with `disableBypassPermissionsMode: "disable"` in managed settings for non-container environments.
- **Over-permissive `Bash(*)` without sandboxing**: Allows arbitrary command execution with no OS-level guardrail. Always combine broad Bash allows with sandbox enforcement.
- **Not denying sensitive file paths**: Omitting deny rules for `.env`, `credentials.*`, private keys, and similar files leaves them readable/editable by default.
- **Confusing `/path` with `//path`**: `/Users/alice/file` is project-root-relative, not absolute. Absolute paths require the `//` prefix.

## Source

Distilled from: `archive/# Configure permissions.txt`
