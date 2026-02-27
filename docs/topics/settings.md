# Settings System

## What It Is

Claude Code uses a hierarchical settings system with multiple scopes (managed, user, project, local) to configure permissions, model behavior, environment, and tool access. Settings are stored as JSON files at different filesystem levels, and a strict precedence order determines which value wins when the same key appears in multiple scopes. The `/config` command opens a tabbed UI for interactive configuration; `/status` shows which settings sources are active.

## Key Concepts

- **Settings scopes**: Four levels determine where a configuration applies and who sees it:
  - **Managed** -- Organization-wide policies deployed by IT. Cannot be overridden.
  - **User** -- Personal preferences in `~/.claude/settings.json`. Apply across all projects.
  - **Project (shared)** -- Team settings in `.claude/settings.json`. Committed to git.
  - **Project (local)** -- Personal per-project overrides in `.claude/settings.local.json`. Gitignored.

- **Settings precedence** (highest to lowest):
  1. Managed settings (server-managed > MDM/OS policies > `managed-settings.json` > HKCU registry on Windows). Only one managed source is used; sources do not merge.
  2. Command-line arguments (temporary session overrides)
  3. Local project settings (`.claude/settings.local.json`)
  4. Shared project settings (`.claude/settings.json`)
  5. User settings (`~/.claude/settings.json`)

- **Settings file locations by scope**:

| Scope | File | Notes |
|:------|:-----|:------|
| User | `~/.claude/settings.json` | Personal, all projects |
| Project (shared) | `<repo>/.claude/settings.json` | Committed to git |
| Project (local) | `<repo>/.claude/settings.local.json` | Gitignored automatically |
| Managed (file, macOS) | `/Library/Application Support/ClaudeCode/managed-settings.json` | |
| Managed (file, Linux/WSL) | `/etc/claude-code/managed-settings.json` | |
| Managed (file, Windows) | `C:\Program Files\ClaudeCode\managed-settings.json` | |
| Managed (registry, Windows admin) | `HKLM\SOFTWARE\Policies\ClaudeCode` → `Settings` value (REG_SZ) containing JSON | |
| Managed (registry, Windows user) | `HKCU\SOFTWARE\Policies\ClaudeCode` | Lowest policy priority |
| Managed (macOS MDM) | `com.anthropic.claudecode` managed preferences domain (Jamf, Kandji) | |
| Managed (server) | Delivered from Anthropic servers via Claude.ai admin console | Highest managed priority |
| Other config | `~/.claude.json` | OAuth, theme, MCP (user/local), per-project state, caches |
| MCP (project) | `<repo>/.mcp.json` | Project-scoped MCP servers |

- **JSON schema validation**: Add `"$schema": "https://json.schemastore.org/claude-code-settings.json"` to enable autocomplete and validation in VS Code/Cursor.
- **Automatic backups**: Claude Code creates timestamped backups of configuration files (keeps 5 most recent).

## Configuration

### Permission settings

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ],
    "ask": [
      "Bash(git push *)"
    ],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable",
    "additionalDirectories": ["../docs/"]
  }
}
```

Rule format: `Tool` or `Tool(specifier)`. Evaluation order: deny first, then ask, then allow. First match wins. Wildcards supported (`*`). Tool-specific specifiers: `Read(path)`, `Edit(path)`, `WebFetch(domain:example.com)`, `Bash(command pattern)`.

### Model configuration

| Setting | Purpose | Example |
|:--------|:--------|:--------|
| `model` | Override default model | `"claude-sonnet-4-6"` |
| `availableModels` | Restrict model picker choices | `["sonnet", "haiku"]` |
| `alwaysThinkingEnabled` | Enable extended thinking by default | `true` |

Env vars: `ANTHROPIC_MODEL`, `CLAUDE_CODE_EFFORT_LEVEL` (`low`/`medium`/`high`), `CLAUDE_CODE_SUBAGENT_MODEL`, `MAX_THINKING_TOKENS`.

### Memory

| Setting | Purpose |
|:--------|:--------|
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | `1` to disable auto memory, `0` to force on |

Memory files (`CLAUDE.md`) load at startup per scope: `~/.claude/CLAUDE.md` (user), `CLAUDE.md` or `.claude/CLAUDE.md` (project), `CLAUDE.local.md` (local).

### Hooks

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup",
      "hooks": [{
        "type": "command",
        "command": "echo 'conda activate myenv' >> \"$CLAUDE_ENV_FILE\""
      }]
    }]
  }
}
```

Managed-only: `allowManagedHooksOnly: true` blocks user/project/plugin hooks. `disableAllHooks: true` disables all hooks and custom status line.

### MCP servers

| Setting | Purpose |
|:--------|:--------|
| `enableAllProjectMcpServers` | Auto-approve all MCP servers in project `.mcp.json` |
| `enabledMcpjsonServers` | Allowlist specific MCP server names |
| `disabledMcpjsonServers` | Blocklist specific MCP server names |
| `allowedMcpServers` | (Managed) Allowlist for user-configurable MCP servers |
| `deniedMcpServers` | (Managed) Denylist, takes precedence over allowlist |
| `allowManagedMcpServersOnly` | (Managed) Only admin-defined MCP allowlist applies |

User MCP config: `~/.claude.json`. Project MCP config: `.mcp.json`. Managed MCP config: `managed-mcp.json` in system directories.

### Sandbox settings

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"],
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true,
      "allowManagedDomainsOnly": true
    }
  }
}
```

Filesystem and network restrictions use standard permission rules (`Read`/`Edit`/`WebFetch` deny/allow), not sandbox settings.

### UI/UX settings

| Setting | Purpose | Example |
|:--------|:--------|:--------|
| `language` | Response language | `"japanese"` |
| `showTurnDuration` | Show turn timing | `true` |
| `spinnerVerbs` | Custom spinner text | `{"mode": "append", "verbs": ["Pondering"]}` |
| `spinnerTipsEnabled` | Show tips in spinner | `false` |
| `prefersReducedMotion` | Reduce animations | `true` |
| `terminalProgressBarEnabled` | Terminal progress bar | `false` |
| `outputStyle` | Adjust system prompt style | `"Explanatory"` |
| `autoUpdatesChannel` | `"stable"` or `"latest"` (default) | `"stable"` |
| `cleanupPeriodDays` | Session retention (default 30) | `20` |

### Attribution settings

```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: AI <ai@example.com>",
    "pr": ""
  }
}
```

Set `commit` or `pr` to empty string to hide attribution. Replaces deprecated `includeCoAuthoredBy`.

### Plugins

```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "github", "repo": "acme-corp/claude-plugins" }
    }
  }
}
```

Managed-only: `strictKnownMarketplaces` controls which marketplaces users can add. `blockedMarketplaces` blocks sources before any download.

### Key environment variables (most commonly needed)

| Variable | Purpose |
|:---------|:--------|
| `ANTHROPIC_API_KEY` | API key for Claude SDK |
| `ANTHROPIC_MODEL` | Override model |
| `CLAUDE_CODE_EFFORT_LEVEL` | `low`/`medium`/`high` reasoning depth |
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex AI |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | `1` to disable auto memory |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens (default 32000, max 64000) |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | `1` to enable OpenTelemetry |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Disables updates, error reporting, telemetry |
| `BASH_DEFAULT_TIMEOUT_MS` | Default bash timeout |
| `BASH_MAX_TIMEOUT_MS` | Max bash timeout |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Context % (1-100) at which auto-compaction triggers |
| `CLAUDE_CODE_SIMPLE` | `1` for minimal mode (no MCP, hooks, CLAUDE.md) |
| `HTTP_PROXY` / `HTTPS_PROXY` | Proxy servers |
| `CLAUDE_ENV_FILE` | Path to script sourced before each bash command |
| `CLAUDE_CONFIG_DIR` | Custom config/data directory |
| `MCP_TIMEOUT` / `MCP_TOOL_TIMEOUT` | MCP server startup / tool execution timeouts |
| `ENABLE_TOOL_SEARCH` | MCP tool search: `auto`, `auto:N`, `true`, `false` |

### /config command

Run `/config` in the REPL to open a tabbed settings interface. Run `/status` to see active settings sources and their origins (e.g., `Enterprise managed settings (remote)`, `(plist)`, `(HKLM)`, `(file)`).

## Decision Guide

| Need | Where to configure |
|:-----|:-------------------|
| Personal preferences (theme, language, spinner) | User settings: `~/.claude/settings.json` |
| Team standards (permissions, hooks, MCP servers) | Shared project settings: `.claude/settings.json` |
| CI/automation pipelines | Environment variables or CLI flags (`--model`, `--append-system-prompt`) |
| Organization-wide policy enforcement | Managed settings (server, MDM, registry, or `managed-settings.json`) |
| Per-developer project overrides | Local project settings: `.claude/settings.local.json` |
| Temporary one-off overrides | Command-line arguments |
| Sensitive file exclusion | `permissions.deny` rules in project settings |
| Subagents shared with team | `.claude/agents/` (project-level) |
| Personal subagents across projects | `~/.claude/agents/` |

## Patterns

- **Layered settings**: Organization defaults (managed) + project standards (shared) + personal preferences (user) + machine-specific tweaks (local). Each layer adds or overrides the one below it.
- **CLI flags for one-off overrides**: Use `--model`, `--append-system-prompt`, `--dangerously-skip-permissions`, `--add-dir` without changing any settings file.
- **Environment variables for CI/CD**: Set `ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`, etc. in pipeline configuration instead of committing settings files.
- **SessionStart hooks for persistent environments**: Write to `$CLAUDE_ENV_FILE` in a hook to ensure conda/venv activation or env var setup persists across all bash commands within a session.
- **Permission deny rules for security**: Block access to `.env`, secrets, and credentials via `permissions.deny` in project settings so all team members are protected.
- **Sandbox + auto-allow**: Enable `sandbox.enabled: true` with `autoAllowBashIfSandboxed: true` to get security isolation without constant permission prompts.

## Anti-Patterns

- **Putting personal preferences in shared project settings**: Theme, language, spinner customization belong in `~/.claude/settings.json`, not `.claude/settings.json`. Commits personal taste into team config.
- **Not using managed settings for organization-wide policies**: Relying on project-level `permissions.deny` for security policies that must not be overridden. Users can bypass project settings with local overrides; managed settings cannot be overridden.
- **Hardcoding API keys or model names in settings files committed to git**: Use environment variables (`ANTHROPIC_API_KEY`) or `apiKeyHelper` scripts instead.
- **Conflicting settings across scopes without understanding precedence**: A permission allowed at user level is overridden by a deny at project level. Always check `/status` to see effective configuration.
- **Setting `env` vars in bash commands expecting persistence**: Environment variables set via `export` in one Bash command do not persist to the next. Use `CLAUDE_ENV_FILE`, `env` in settings.json, or SessionStart hooks instead.
- **Using `managed-settings.json` path from one OS on another**: File paths differ per OS. macOS uses `/Library/Application Support/ClaudeCode/`, Linux uses `/etc/claude-code/`, Windows uses `C:\Program Files\ClaudeCode\`.
- **Ignoring `$schema` for settings validation**: Adding `"$schema": "https://json.schemastore.org/claude-code-settings.json"` catches typos and invalid keys at edit time.

## Source

Distilled from: `archive/# Claude Code settings.txt`
