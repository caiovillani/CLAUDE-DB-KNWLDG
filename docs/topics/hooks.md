# Hooks System

## What It Is

Hooks are user-defined shell commands, LLM prompts, or agent subprocesses that execute automatically at specific points in Claude Code's lifecycle. Unlike CLAUDE.md instructions (which are advisory and can be ignored by the model), hooks provide **deterministic enforcement** -- they run as actual code with real exit codes, so a PreToolUse hook that denies `rm -rf` will block it every single time, not "most of the time." Hooks are the mechanism for hard rules; CLAUDE.md is for soft guidance.

## Key Concepts

### Hook lifecycle

Hooks fire at specific points during a Claude Code session. Some events fire once per session (SessionStart, SessionEnd), while others fire repeatedly inside the agentic loop (PreToolUse, PostToolUse, Stop). When an event fires and a matcher matches, Claude Code passes JSON context on stdin to the hook handler.

### Hook types

- **Command hooks** (`type: "command"`): Run a shell command. Receive JSON on stdin, communicate via exit codes and stdout JSON.
- **Prompt hooks** (`type: "prompt"`): Send a prompt to a fast Claude model for single-turn yes/no evaluation. Returns `{ "ok": true/false, "reason": "..." }`.
- **Agent hooks** (`type: "agent"`): Spawn a subagent with tool access (Read, Grep, Glob) for multi-turn verification before returning a decision. Up to 50 turns.

Not all events support all types. Prompt and agent hooks are supported on: PreToolUse, PostToolUse, PostToolUseFailure, PermissionRequest, UserPromptSubmit, Stop, SubagentStop, TaskCompleted. All other events are command-only.

### Events

| Event | When it fires | Can block? |
|---|---|---|
| `SessionStart` | Session begins or resumes | No |
| `UserPromptSubmit` | User submits a prompt, before Claude processes it | Yes |
| `PreToolUse` | Before a tool call executes | Yes |
| `PermissionRequest` | When a permission dialog appears | Yes |
| `PostToolUse` | After a tool call succeeds | No (feedback only) |
| `PostToolUseFailure` | After a tool call fails | No (feedback only) |
| `Notification` | When Claude Code sends a notification | No |
| `SubagentStart` | When a subagent is spawned | No |
| `SubagentStop` | When a subagent finishes | Yes |
| `Stop` | When Claude finishes responding | Yes |
| `TeammateIdle` | When an agent team teammate is about to go idle | Yes |
| `TaskCompleted` | When a task is being marked as completed | Yes |
| `ConfigChange` | When a configuration file changes during a session | Yes (except policy) |
| `WorktreeCreate` | When a worktree is being created | Yes (non-zero = fail) |
| `WorktreeRemove` | When a worktree is being removed | No |
| `PreCompact` | Before context compaction | No |
| `SessionEnd` | When a session terminates | No |

### Matchers

The `matcher` field is a **regex string** that filters when hooks fire. Omit matcher, set `""`, or use `"*"` to match everything.

- Tool events (PreToolUse, PostToolUse, etc.): matches on `tool_name` -- e.g., `"Bash"`, `"Edit|Write"`, `"mcp__memory__.*"`
- SessionStart: matches on source -- `"startup"`, `"resume"`, `"clear"`, `"compact"`
- SessionEnd: matches on reason -- `"clear"`, `"logout"`, `"prompt_input_exit"`, etc.
- Notification: matches on type -- `"permission_prompt"`, `"idle_prompt"`, `"auth_success"`, `"elicitation_dialog"`
- ConfigChange: matches on source -- `"user_settings"`, `"project_settings"`, `"local_settings"`, `"policy_settings"`, `"skills"`
- PreCompact: matches on trigger -- `"manual"`, `"auto"`
- SubagentStart/SubagentStop: matches on agent type -- `"Bash"`, `"Explore"`, `"Plan"`, or custom names
- UserPromptSubmit, Stop, TeammateIdle, TaskCompleted, WorktreeCreate, WorktreeRemove: **no matcher support**, always fire

### JSON input/output format

**Common input fields** (all events receive these on stdin):

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse"
}
```

Each event adds its own fields (e.g., PreToolUse adds `tool_name`, `tool_input`, `tool_use_id`).

**JSON output fields** (on exit 0, printed to stdout):

| Field | Default | Description |
|---|---|---|
| `continue` | `true` | `false` stops Claude entirely |
| `stopReason` | none | Message shown to user when `continue` is false |
| `suppressOutput` | `false` | Hides stdout from verbose mode |
| `systemMessage` | none | Warning message shown to user |

### Exit codes

| Code | Meaning | Behavior |
|---|---|---|
| `0` | Success | Action proceeds. Stdout parsed for JSON output |
| `2` | Blocking error | Action blocked (for events that support blocking). Stderr fed to Claude as error |
| Any other | Non-blocking error | Stderr shown in verbose mode. Execution continues |

### Async hooks

Set `"async": true` on command hooks to run in the background. Claude continues working immediately. Results delivered on next conversation turn. Async hooks **cannot** block or return decisions -- the action has already proceeded. Only `type: "command"` supports async.

## Configuration

### Where hooks are configured

| Location | Scope | Shareable |
|---|---|---|
| `~/.claude/settings.json` | All your projects | No |
| `.claude/settings.json` | Single project | Yes, committable |
| `.claude/settings.local.json` | Single project | No, gitignored |
| Managed policy settings | Organization-wide | Admin-controlled |
| Plugin `hooks/hooks.json` | When plugin enabled | Bundled with plugin |
| Skill/agent YAML frontmatter | While component active | Defined in component |

### Full configuration schema

```json
{
  "hooks": {
    "<EventName>": [
      {
        "matcher": "regex_pattern",
        "hooks": [
          {
            "type": "command",
            "command": "path/to/script.sh",
            "timeout": 600,
            "statusMessage": "Running validation...",
            "async": false
          }
        ]
      }
    ]
  }
}
```

Three levels: **event** -> **matcher group** -> **hook handler(s)**.

### Handler fields

**Common fields** (all types):

| Field | Required | Description |
|---|---|---|
| `type` | yes | `"command"`, `"prompt"`, or `"agent"` |
| `timeout` | no | Seconds before canceling. Defaults: 600 (command), 30 (prompt), 60 (agent) |
| `statusMessage` | no | Custom spinner message |
| `once` | no | If true, runs only once per session (skills only) |

**Command-specific**: `command` (required), `async` (optional bool).

**Prompt/Agent-specific**: `prompt` (required, use `$ARGUMENTS` for input placeholder), `model` (optional).

### Decision control by event

| Events | Pattern | Key fields |
|---|---|---|
| UserPromptSubmit, PostToolUse, PostToolUseFailure, Stop, SubagentStop, ConfigChange | Top-level `decision` | `decision: "block"`, `reason` |
| TeammateIdle, TaskCompleted | Exit code only | Exit 2 blocks, stderr is feedback |
| PreToolUse | `hookSpecificOutput` | `permissionDecision` (allow/deny/ask), `permissionDecisionReason`, `updatedInput`, `additionalContext` |
| PermissionRequest | `hookSpecificOutput` | `decision.behavior` (allow/deny), `updatedInput`, `updatedPermissions` |
| WorktreeCreate | stdout path | Print absolute path to worktree |
| WorktreeRemove, Notification, SessionEnd, PreCompact | None | Side effects only |

### PreToolUse decision control example

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Destructive command blocked"
  }
}
```

Values for `permissionDecision`: `"allow"` (bypass permission system), `"deny"` (block tool call), `"ask"` (prompt user to confirm).

### Prompt hook response schema

```json
{
  "ok": true,
  "reason": "Required when ok is false"
}
```

### Environment variables

- `$CLAUDE_PROJECT_DIR`: project root (use in `command` field)
- `${CLAUDE_PLUGIN_ROOT}`: plugin root directory
- `$CLAUDE_ENV_FILE`: available in SessionStart hooks only, for persisting env vars
- `$CLAUDE_CODE_REMOTE`: `"true"` in remote web environments

### MCP tool matching

MCP tools follow `mcp__<server>__<tool>` naming. Match with regex:
- `mcp__memory__.*` -- all tools from memory server
- `mcp__.*__write.*` -- any write tool from any server

### Disabling hooks

- Remove from settings JSON or use `/hooks` menu.
- `"disableAllHooks": true` in settings disables all hooks (respects managed settings hierarchy -- user-level cannot disable managed hooks).
- Hooks are snapshotted at startup. Mid-session changes require review in `/hooks` menu before taking effect.

### Debugging

Run `claude --debug` to see hook execution details. Toggle verbose mode with `Ctrl+O` for in-transcript output.

## Decision Guide

| Situation | Mechanism |
|---|---|
| Must happen every time with zero exceptions | Hook (command type) |
| Advisory guidance, best practices, style | CLAUDE.md |
| Block/allow tool access broadly | Permissions system |
| Validate tool arguments at runtime | PreToolUse hook |
| Modify tool input before execution | PreToolUse hook with `updatedInput` |
| Post-processing after tool use (lint, format) | PostToolUse hook |
| Block Claude from stopping prematurely | Stop hook |
| Quality gate before task completion | TaskCompleted hook |
| Custom desktop/system notifications | Notification hook |
| Inject dynamic context at session start | SessionStart hook |
| Audit configuration changes | ConfigChange hook |
| Non-git VCS worktree management | WorktreeCreate + WorktreeRemove hooks |
| LLM-evaluated policy (flexible criteria) | Prompt or agent hook |

## Patterns

### Lint after every file edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/lint-check.sh"
          }
        ]
      }
    ]
  }
}
```

### Block writes to protected directories

```bash
#!/bin/bash
# .claude/hooks/protect-dirs.sh
FILE_PATH=$(jq -r '.tool_input.file_path' < /dev/stdin)

if echo "$FILE_PATH" | grep -qE '(\.env|\.git/|/secrets/)'; then
  echo "Write to protected path blocked: $FILE_PATH" >&2
  exit 2
fi
exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": ".claude/hooks/protect-dirs.sh" }]
      }
    ]
  }
}
```

### Block destructive shell commands

```bash
#!/bin/bash
# .claude/hooks/block-rm.sh
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive command blocked by hook"
    }
  }'
else
  exit 0
fi
```

### Desktop notifications when Claude needs attention

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [{ "type": "command", "command": "/path/to/permission-alert.sh" }]
      },
      {
        "matcher": "idle_prompt",
        "hooks": [{ "type": "command", "command": "/path/to/idle-notification.sh" }]
      }
    ]
  }
}
```

### Auto-run tests in background after file changes

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/run-tests-async.sh",
            "async": true,
            "timeout": 300
          }
        ]
      }
    ]
  }
}
```

### LLM-evaluated stop gate (prompt hook)

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude should stop: $ARGUMENTS. Check if all tasks are complete, errors are addressed, and follow-up is not needed. Respond with JSON: {\"ok\": true} or {\"ok\": false, \"reason\": \"explanation\"}.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Persist environment variables at session start

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
  echo 'export DEBUG_LOG=true' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

### Hooks in skill/agent frontmatter

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

## Anti-Patterns

- **Using hooks for things CLAUDE.md can handle**: Hooks add execution overhead on every matching event. If the guidance is advisory ("prefer const over let"), put it in CLAUDE.md. Reserve hooks for hard enforcement.
- **Long-running synchronous hooks that slow the loop**: A 30-second lint run on every PostToolUse grinds the session to a halt. Use `"async": true` for anything slow, or narrow matchers to minimize firing frequency.
- **Not handling hook failures gracefully**: If your hook script crashes with a random exit code (not 0, not 2), Claude treats it as a non-blocking error and continues silently. Make sure exit codes are intentional.
- **Overly broad matchers that catch unintended tools**: A PreToolUse hook with no matcher runs on every single tool call (Read, Grep, Glob, Bash, Edit, Write, all MCP tools). Be specific.
- **Infinite Stop hook loops**: A Stop hook that always returns `decision: "block"` will force Claude to keep running forever. Always check `stop_hook_active` in Stop hook input and allow stopping when it's `true`.
- **Mixing exit code 2 with JSON output**: Claude Code only parses JSON on exit 0. If you exit 2 with JSON on stdout, the JSON is ignored. Pick one approach per hook.
- **Not quoting shell variables**: `$VAR` without quotes risks word splitting and glob expansion. Always use `"$VAR"`.
- **Forgetting hooks are snapshotted at startup**: Edits to hook config mid-session don't take effect until reviewed in `/hooks` menu or session restart.

## Source

Distilled from: `archive/# Hooks reference.txt`
(Claude Code official documentation: Hooks reference page)
