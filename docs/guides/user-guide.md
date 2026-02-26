# CLAUDE-DB-KNWLDG User Guide

This workspace is an expert knowledge base on Claude Code core practices. It turns any Claude Code session into a specialist consultant on CLAUDE.md design, hooks, permissions, settings, workflows, and memory — backed by curated documentation, proven patterns, and an archive of raw Anthropic docs for verification.

This guide covers everything you need to extract maximum value from it.

---

## 1. Quick Start

Get value in under 5 minutes. Open a terminal in this workspace directory and start a Claude Code session.

**Ask a reference question:**

```
What events can hooks fire on, and which ones can block execution?
```

The agent looks up `docs/topics/hooks.md`, returns a precise answer with the events table, and cites the source.

**Get project setup guidance:**

```
I'm starting a new TypeScript monorepo with 3 team members. Walk me through setting up Claude Code for it.
```

The agent enters CONSULT mode, reads `docs/guides/new-project-setup.md` and `docs/patterns/claude-md-patterns.md`, and delivers a tailored 7-step plan with monorepo-specific patterns.

**Audit an existing config:**

```
Review this CLAUDE.md and tell me what's wrong:

# My Project
This is a React app. We use TypeScript. Please always write clean code
and follow best practices. Use prettier for formatting. The API is at
localhost:3000. My OpenAI key is sk-abc123...
```

The agent enters COACH mode, reads `docs/patterns/common-mistakes.md`, and flags: secrets leak, vague instructions ("clean code", "best practices"), missing build commands, no emphasis hierarchy. Returns prioritized fixes.

---

## 2. Understanding the Architecture

### What this workspace IS

- A structured knowledge base that Claude Code reads at session start
- An expert consultant that answers questions about Claude Code configuration and best practices
- A proactive coach that audits configs and identifies gaps you didn't ask about
- A reference manual with curated docs, proven patterns, and anti-pattern catalogs

### What this workspace IS NOT

- A plugin or extension — it's a standard Claude Code workspace with a well-designed CLAUDE.md
- A live connection to Anthropic's docs — the archive is a point-in-time snapshot
- A replacement for reading Anthropic's official documentation when you need the latest info
- An authority on plugins, MCP servers, skills, or multi-agent orchestration (those are out of scope for v1)

### The three knowledge layers

| Layer | Location | Confidence | When the agent uses it |
|---|---|---|---|
| **Curated docs** | `docs/topics/`, `docs/guides/`, `docs/patterns/` | High — distilled and organized from official Anthropic documentation | Primary source for all recommendations |
| **Field notes** | `field-notes/` | Medium — battle-tested but context-specific | Secondary source, always marked as "from field experience" |
| **Archive** | `archive/` | Raw — unprocessed official documentation | Verification backstop when curated docs don't cover enough detail |

The agent is instructed to cite which layer every recommendation comes from. If you see an unsourced claim, ask: "What's your source for that?"

### Directory map

```
CLAUDE-DB-KNWLDG/
├── CLAUDE.md                          # Agent identity & routing logic
├── docs/
│   ├── topics/                        # 6 curated reference files
│   │   ├── claude-md.md               #   CLAUDE.md design & architecture
│   │   ├── hooks.md                   #   Hooks system (391 lines — deepest file)
│   │   ├── memory.md                  #   Memory system & persistence
│   │   ├── permissions.md             #   Permission model & configuration
│   │   ├── settings.md                #   Settings hierarchy & 30+ env vars
│   │   └── workflows.md              #   Workflows, sessions, automation
│   ├── guides/                        # Task-oriented walkthroughs
│   │   ├── new-project-setup.md       #   7-step setup from scratch
│   │   └── user-guide.md             #   This file
│   └── patterns/                      # Proven practices & failure modes
│       ├── claude-md-patterns.md      #   6 patterns + 5 anti-patterns
│       └── common-mistakes.md         #   15 failure patterns across 5 categories
├── field-notes/                       # Battle-tested experience (medium confidence)
│   └── README.md                      #   Provenance, confidence levels, contribution protocol
├── archive/                           # Raw Anthropic docs (23 files)
│   ├── # Hooks reference.txt
│   ├── # Configure permissions.txt
│   ├── # Claude Code settings.txt
│   └── ... (20 more raw doc files)
└── .claude/
    └── settings.local.json            # Workspace permission config
```

### How the CLAUDE.md brain works

The root `CLAUDE.md` defines 8 sections that control the agent's behavior:

1. **Identity & Role** — Expert consultant + proactive coach, serving humans and other Claude instances
2. **Knowledge Architecture** — Three layers with mandatory source citation
3. **Interaction Modes** — CONSULT, COACH, REFERENCE (see Section 4)
4. **Routing Logic** — Automatic mode selection based on input type
5. **Quality Standards** — Every recommendation must be traceable; uncertainty flagged explicitly
6. **Scope Boundaries** — 6 in-scope topics; plugins, MCP, skills, orchestration deferred to v2
7. **Session Start Protocol** — Read CLAUDE.md, determine mode, load relevant topic files
8. **Response Format** — BLUF first, then rationale, examples, and proactive observations

---

## 3. All Use Cases

### Tier 1: Core (weekly use)

#### UC-1: Setting up a new Claude Code project

**Mode triggered:** CONSULT

**Prompt:**
```
I'm setting up Claude Code for a [describe your project: language, team size,
repo structure]. Guide me through the full setup.
```

**What happens internally:** The agent reads `docs/guides/new-project-setup.md` and adapts the 7-step walkthrough (bootstrap CLAUDE.md → configure permissions → set up rules → personal preferences → hooks → validate → commit) to your specific context. It also pulls patterns from `docs/patterns/claude-md-patterns.md` for the CLAUDE.md structure.

**Expected output:** A numbered step-by-step plan with code snippets for each config file, tailored to your project type and team size.

**Pro tip:** Be specific about your stack and team size. "TypeScript monorepo, 5 developers, CI with GitHub Actions" yields much better guidance than "new project."

---

#### UC-2: Designing or auditing a CLAUDE.md file

**Mode triggered:** COACH (if you paste a file) or CONSULT (if you describe what you want)

**Prompt (audit):**
```
Review this CLAUDE.md for my project:
[paste your CLAUDE.md content]
```

**Prompt (design):**
```
Help me design a CLAUDE.md for a Django REST API with PostgreSQL,
deployed on AWS. Team of 3, all using VS Code.
```

**What happens internally:** The agent reads `docs/topics/claude-md.md` (file locations, import system, content strategy), `docs/patterns/claude-md-patterns.md` (6 proven patterns, 5 anti-patterns), and `docs/patterns/common-mistakes.md` (CLAUDE.md-specific failure modes). For audits, it checks against all known anti-patterns systematically.

**Expected output:** For audits: prioritized list of issues with specific fixes. For design: a complete CLAUDE.md draft with sections explained.

---

#### UC-3: Troubleshooting common Claude Code mistakes

**Mode triggered:** CONSULT

**Prompt:**
```
Claude keeps ignoring my CLAUDE.md instructions about using pnpm instead of npm.
What am I doing wrong?
```

**What happens internally:** The agent reads `docs/patterns/common-mistakes.md` (15 failure patterns) and cross-references against relevant topic files. For this example, it would check: Is the instruction vague? Is it buried in a bloated file? Is there a conflicting instruction elsewhere in the hierarchy?

**Expected output:** Diagnosis with root cause, fix, and preventive measure. The agent also flags related issues you might not have noticed.

**Common troubleshooting categories covered:**
- Session mistakes (kitchen sink sessions, correction spirals, infinite exploration)
- CLAUDE.md mistakes (bloated files, documentation-style writing, personal prefs in team files)
- Permission mistakes (over-permissive Bash, fragile URL patterns)
- Workflow mistakes (skipping Plan Mode, no subagents for research)
- Hook mistakes (hooks for everything, synchronous heavy hooks)

---

#### UC-4: Getting reference answers on Claude Code features

**Mode triggered:** REFERENCE

**Prompt examples:**
```
What's the syntax for path-scoped rules in .claude/rules/?
```
```
What environment variables does Claude Code support?
```
```
How does the permission evaluation order work?
```

**What happens internally:** The agent reads the relevant topic file and returns a precise, concise answer with syntax examples and source attribution.

**Expected output:** Direct answer with code/config examples. No preamble.

---

### Tier 2: Configuration (as needed)

#### UC-5: Configuring permissions for a team

**Mode triggered:** CONSULT

**Prompt:**
```
My team of 8 developers needs a permission setup that allows common dev commands
(npm, git, docker) but blocks destructive operations and force pushes.
How should we configure this?
```

**What happens internally:** The agent reads `docs/topics/permissions.md` (rule types, modes, specifiers, precedence) and provides a layered configuration using shared project settings for team rules and local settings for personal overrides.

**Expected output:** JSON config examples for `.claude/settings.json` (shared) and `.claude/settings.local.json` (personal), with explanations of the `allow > ask > deny` evaluation order and wildcard patterns.

---

#### UC-6: Choosing between hooks, CLAUDE.md, and permissions

**Mode triggered:** REFERENCE or CONSULT

**Prompt:**
```
I want to prevent developers from deleting production database tables.
Should I use a hook, a CLAUDE.md rule, or a permission?
```

**What happens internally:** The agent reads the Decision Guide in `docs/topics/hooks.md` (line 203), which maps situations to mechanisms:

| Situation | Mechanism |
|---|---|
| Must happen every time with zero exceptions | Hook (command type) |
| Advisory guidance, best practices, style | CLAUDE.md |
| Block/allow tool access broadly | Permissions system |
| Validate tool arguments at runtime | PreToolUse hook |

**Expected output:** A clear recommendation with rationale. For this example: "Use a PreToolUse hook on `Bash` that inspects the command for `DROP TABLE` patterns — hooks provide deterministic enforcement, while CLAUDE.md rules are advisory and can be ignored."

---

#### UC-7: Configuring hooks for specific scenarios

**Mode triggered:** CONSULT or REFERENCE

**Prompt:**
```
I want to automatically run ESLint after every file edit.
Show me the hook configuration.
```

**What happens internally:** The agent reads `docs/topics/hooks.md` (391 lines covering all hook types, events, matchers, JSON I/O, and 10+ detailed patterns including "lint after every file edit").

**Expected output:** Complete JSON hook configuration with the event (`PostToolUse`), matcher (`Edit|Write`), handler command, and explanation of the JSON input/output contract.

---

#### UC-8: Setting up settings hierarchy for a team

**Mode triggered:** CONSULT

**Prompt:**
```
We have a company-wide standard, team-level conventions, and individual
developer preferences. How do we layer Claude Code settings for this?
```

**What happens internally:** The agent reads `docs/topics/settings.md` (scopes, precedence, file locations) and designs a multi-level configuration: managed policy for company standards, project `.claude/settings.json` for team conventions, and `.claude/settings.local.json` for personal overrides.

**Expected output:** A settings architecture diagram with file locations, precedence order, and examples for each level.

---

#### UC-9: Managing memory across sessions

**Mode triggered:** CONSULT or REFERENCE

**Prompt:**
```
My auto memory MEMORY.md is getting bloated and Claude seems to forget things.
How should I organize it?
```

**What happens internally:** The agent reads `docs/topics/memory.md` (auto memory vs CLAUDE.md, hierarchy, loading behavior, patterns, anti-patterns).

**Expected output:** Specific advice: keep MEMORY.md under 200 lines (only the first 200 load at startup), move detailed notes into topic files, use MEMORY.md as a concise index. Includes the reorganization steps and anti-patterns to avoid.

---

### Tier 3: Advanced & Automation

#### UC-10: Headless/CI automation patterns

**Mode triggered:** CONSULT

**Prompt:**
```
I want to use Claude Code in my GitHub Actions CI pipeline for automated
code review. How do I set this up?
```

**What happens internally:** The agent reads `docs/topics/workflows.md` (headless mode with `claude -p "prompt"`, session management, environment variables) and `docs/topics/settings.md` (env vars for CI/CD).

**Expected output:** CI configuration with headless invocation, relevant environment variables, permission considerations for non-interactive mode, and the verification-first workflow pattern.

---

#### UC-11: Session management strategies

**Mode triggered:** CONSULT or REFERENCE

**Prompt:**
```
When should I use --continue vs --resume? What's the difference
between /clear, /compact, and starting fresh?
```

**What happens internally:** The agent reads `docs/topics/workflows.md` (sessions, subcommands: `--continue`, `--resume`, `/rename`, `/clear`, `/compact`, `/rewind`).

**Expected output:** A comparison table of session management commands with when to use each, plus the anti-pattern of "kitchen sink sessions" that accumulate too much context.

---

#### UC-12: Reviewing another project's Claude Code config

**Mode triggered:** COACH

**Prompt:**
```
I found this open-source project's Claude Code config. Review it:
[paste CLAUDE.md, settings.json, and/or hook configs]
```

**What happens internally:** The agent audits the config against all known patterns and anti-patterns from `docs/patterns/common-mistakes.md`, `docs/patterns/claude-md-patterns.md`, and relevant topic files.

**Expected output:** Systematic audit with findings prioritized by impact: critical issues first, then improvements, then nice-to-haves. Each finding includes the source it's checked against.

---

### Tier 4: Meta & Cross-Workspace

#### UC-13: Pointing another Claude workspace to consult this one

**Mode triggered:** REFERENCE

**Prompt:**
```
How do I make my other project's Claude Code session use this
knowledge base as a reference?
```

**What happens internally:** The agent reads `docs/topics/memory.md` (additional directories, imports) and provides the three cross-workspace methods detailed in Section 5 of this guide.

**Expected output:** Three methods with exact syntax, trade-offs, and the recommended approach for the user's situation.

---

#### UC-14: Adding field notes from real experience

**Mode triggered:** REFERENCE

**Prompt:**
```
I discovered that path-scoped rules with brace expansion don't work
on Windows with certain glob patterns. How do I add this as a field note?
```

**What happens internally:** The agent reads `field-notes/README.md` (contribution protocol).

**Expected output:** The 5-step template: context, pattern, outcome, confidence rating (emerging/consistent), caveats. Plus the file naming convention (mirrors `docs/topics/` structure).

**Field note template:**

```markdown
## [Descriptive Title]

**Context:** [Project type, OS, team size, what you were trying to do]
**Pattern:** [What happened or what you did]
**Outcome:** [What the result was]
**Confidence:** emerging | consistent
**Caveats:** [When this might not apply]
```

---

#### UC-15: Finding info when curated docs aren't enough

**Mode triggered:** REFERENCE

**Prompt:**
```
I need details about Claude Code's model configuration options.
The curated docs don't cover this in depth.
```

**What happens internally:** The agent first checks the relevant topic file (`docs/topics/settings.md` for model config). If the curated content isn't sufficient, it falls back to the archive — in this case `archive/# Model configuration.txt` or `archive/# Claude Code settings.txt`.

**Expected output:** The answer with a note: "This comes from the raw archive (`archive/# Model configuration.txt`), which hasn't been curated yet. Verify against Anthropic's current documentation for the latest details."

**Archive files available for fallback:**
- `# Claude Code Docs.txt`, `# Common workflows.txt`, `# Best Practices for Claude Code.txt`
- `# Claude Code settings.txt`, `# Configure permissions.txt`, `# Hooks reference.txt`
- `# Manage Claude's memory.txt`, `# Sandboxing.txt`, `# Advanced setup.txt`
- `# Use Claude Code in VS Code.txt`, `# Use Claude Code Desktop.txt`
- `# Model configuration.txt`, `# Speed up responses with fast mode.txt`
- `# Monitoring.txt`, `# Data usage.txt`, `# CLI reference.txt`
- `# Interactive mode.txt`, `# Checkpointing.txt`, `# Plugins reference.txt`
- `# Optimize your terminal setup.txt`, `# Customize your status line.txt`
- `# Customize keyboard shortcuts.txt`

---

#### UC-16: Understanding what's out of scope

**Mode triggered:** REFERENCE

**Prompt:**
```
How do I build a Claude Code plugin?
```

**What happens internally:** The agent checks the scope boundaries in CLAUDE.md (§6) and acknowledges that plugins are out of scope for v1.

**Expected output:** Honest acknowledgment that detailed plugin guidance isn't available yet, any general knowledge the agent has, and a pointer to `archive/# Plugins reference.txt` for raw documentation.

**Out-of-scope topics (v1):**
- Plugins architecture and development
- Skills and commands development
- MCP server integration and configuration
- Multi-agent orchestration patterns
- Claude API / SDK development

---

## 4. Interaction Patterns

### The three modes

| Mode | Trigger | Agent behavior |
|---|---|---|
| **CONSULT** | You describe a situation, goal, or problem | Analyze → identify relevant topics → recommend with trade-offs → provide implementation guidance |
| **COACH** | You share a CLAUDE.md, settings.json, hook config, or project structure | Audit systematically → identify gaps → prioritize by impact → provide specific fixes with rationale |
| **REFERENCE** | You ask "how do I...", "what is...", or request specific syntax | Provide precise answer → include examples → link to deeper topic file |

### Routing logic

```
User input
  ├── Describes goal/problem ──────────→ CONSULT
  ├── Shares config for review ────────→ COACH
  ├── Asks direct question ────────────→ REFERENCE
  └── Ambiguous ───────────────────────→ Agent asks ONE clarifying question
```

In CONSULT and COACH modes, the agent proactively flags related issues or opportunities you didn't ask about. This is by design — the CLAUDE.md instructs it to coach, not just answer.

### Effective prompts by mode

**CONSULT — good prompts:**
```
I'm setting up a monorepo with 3 packages. How should I structure
CLAUDE.md files across the repo?
```
```
My team keeps accidentally force-pushing to main. Design a prevention
strategy using Claude Code's mechanisms.
```
```
I want Claude to automatically format code after every edit but only
in the src/ directory. What's the best approach?
```

**CONSULT — weak prompts:**
```
Tell me about hooks.
```
_(Too vague — triggers REFERENCE mode instead. Be specific about what you're trying to achieve.)_

```
What are all the features of Claude Code?
```
_(Too broad — this workspace covers 6 specific topics. Ask about a specific area.)_

**COACH — good prompts:**
```
Review this CLAUDE.md: [paste content]
```
```
Here's our team's .claude/settings.json. Are we missing anything? [paste content]
```
```
Audit these hook configs for potential issues: [paste content]
```

**COACH — weak prompts:**
```
Is my setup good?
```
_(No config to review — paste the actual content.)_

```
Check my project.
```
_(The agent can only review what you share. Paste specific files.)_

**REFERENCE — good prompts:**
```
What's the syntax for @imports in CLAUDE.md?
```
```
Which hook events support prompt-type hooks?
```
```
How does the deny-first permission evaluation work?
```

**REFERENCE — weak prompts:**
```
How do hooks work?
```
_(Answerable but very broad — you'll get a better answer by asking about a specific aspect of hooks.)_

### What to expect in responses

Every response follows this structure:

1. **BLUF** — The direct answer or recommendation, up front
2. **Rationale** — Why this approach, with source attribution (e.g., "Source: `docs/topics/hooks.md`")
3. **Examples** — Code snippets, config examples, or file structures
4. **Proactive observations** — "You might also want to consider..." (CONSULT/COACH modes only)

---

## 5. Cross-Workspace Usage

Three methods to use this knowledge base from another project's Claude Code session.

### Method 1: `--add-dir` (recommended for active consultation)

```bash
# From your other project directory:
claude --add-dir /path/to/CLAUDE-DB-KNWLDG
```

This gives the session read access to all files in this workspace. The agent can read topic files, patterns, and archive docs when you ask Claude Code questions.

To also load this workspace's CLAUDE.md (so the agent adopts the expert consultant persona):

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir /path/to/CLAUDE-DB-KNWLDG
```

**Trade-offs:**
- Pro: Full access to all knowledge layers, agent can cross-reference files
- Pro: Expert persona loaded when using the env var
- Con: Increases context window usage — both projects' CLAUDE.md files load
- Con: Must remember to use the flag each time (consider a shell alias)

### Method 2: Manual consultation (separate terminal)

Keep a separate terminal with a Claude Code session open in this workspace. Switch between terminals as needed.

```bash
# Terminal 1: Your project
cd ~/my-project && claude

# Terminal 2: Knowledge base consultation
cd ~/CLAUDE-DB-KNWLDG && claude
```

**Trade-offs:**
- Pro: Clean separation — no context pollution between projects
- Pro: Full expert persona always active in the knowledge base terminal
- Con: Manual copy-paste between terminals
- Con: Two sessions consuming API credits

### Method 3: `@import` references in your project's CLAUDE.md

Add imports in your project's CLAUDE.md pointing to specific files in this workspace:

```markdown
# In your project's CLAUDE.md

# Claude Code conventions
@/path/to/CLAUDE-DB-KNWLDG/docs/patterns/claude-md-patterns.md
@/path/to/CLAUDE-DB-KNWLDG/docs/patterns/common-mistakes.md
```

**Trade-offs:**
- Pro: Always loaded — no special flags needed
- Pro: Selective — import only the files you need
- Con: First-time imports trigger an approval dialog per project (declining is permanent)
- Con: Adds to context window budget with every session
- Con: Max import depth is 5 hops — keep chains short
- Con: Absolute paths break if the workspace moves

**Recommendation:** Use Method 1 with the env var for deep consultation sessions. Use Method 3 for patterns/anti-patterns you reference constantly. Use Method 2 when you want a clean separation.

---

## 6. Maintenance & Evolution

### Updating curated docs when Anthropic releases new documentation

1. Download or copy the new official documentation
2. Add the raw text to `archive/` with the same naming convention (`# Topic Name.txt`)
3. Compare against the existing curated topic file in `docs/topics/`
4. Update the curated file with new information, preserving the standard structure (What It Is → Key Concepts → Configuration → Decision Guide → Patterns → Anti-Patterns)
5. Update the `## Source` line at the bottom of the topic file to reflect the new archive source

### Contributing field notes

Follow the protocol in `field-notes/README.md`:

1. **Context** — What kind of project, what problem you were solving
2. **Pattern** — What you did or observed
3. **Outcome** — What happened as a result
4. **Confidence** — `consistent` (validated across 3+ projects) or `emerging` (1-2 projects)
5. **Caveats** — Conditions where this might not apply

Name files to mirror `docs/topics/`: `field-notes/hooks.md`, `field-notes/memory.md`, etc.

### Pruning and hygiene

- Review curated docs quarterly against the archive for staleness
- Remove field notes that have been promoted to curated docs (or mark them as `promoted`)
- Keep the root CLAUDE.md stable — changes there affect the agent's core behavior
- If a topic file exceeds ~400 lines, consider splitting it (hooks.md at 391 lines is near the threshold)

### Archive fallback pattern

When curated docs don't answer a question:

1. Agent checks the relevant topic file first
2. If insufficient, agent reads the corresponding archive file
3. Agent answers with a note: "This comes from the raw archive — verify against current official docs"

This ensures coverage even for details that weren't prioritized during curation.

### Roadmap to v2

The workspace was designed with expansion in mind. Priority areas for v2:

- **Plugins** — Architecture, development patterns (raw docs available in `archive/# Plugins reference.txt`)
- **MCP servers** — Integration, configuration, server development
- **Skills & commands** — Development patterns, slash command design
- **Multi-agent orchestration** — Subagent patterns, fan-out strategies, team coordination
- **Planned guides** — `hook-design-guide.md`, `memory-strategy-guide.md`

Migration threshold: when out-of-scope questions become frequent enough to justify curation effort.

---

## 7. Known Limitations & Workarounds

### 1. No field notes yet
All recommendations come from official documentation. The `field-notes/` directory is scaffolded but empty (besides README.md). This means the agent has no battle-tested experiential knowledge yet.

**Workaround:** Contribute your own field notes as you use Claude Code. Even `emerging` confidence notes add value.

### 2. Out-of-scope topics have raw docs but no curated guidance
Plugins, MCP, skills, and orchestration have raw archive files but no curated topic files. The agent can point you to the archive but can't provide the same depth of analysis.

**Workaround:** Ask the agent to read the relevant archive file directly: "Read `archive/# Plugins reference.txt` and summarize the key concepts."

### 3. No automated cross-workspace consultation
There's no way to make another Claude Code session automatically query this workspace. All three cross-workspace methods (Section 5) require manual setup.

**Workaround:** Create a shell alias: `alias claude-expert='CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ~/CLAUDE-DB-KNWLDG'`

### 4. Windows-specific edge cases underdocumented
The curated docs cover Windows file locations for settings and memory, but Windows-specific behaviors (path separators in glob patterns, symlink limitations, PowerShell vs bash differences) aren't comprehensively documented.

**Workaround:** When encountering Windows-specific issues, check the archive files for details and contribute findings as field notes.

### 5. Archive is a point-in-time snapshot
The archive files reflect Anthropic's documentation at the time they were captured. New features, changed behaviors, or deprecated options won't appear until the archive is manually updated.

**Workaround:** For critical decisions, verify against Anthropic's current live documentation.

### 6. No completeness validation between archive and curated docs
There's no automated way to verify that all information in the archive has been captured in the curated topic files. Some details may exist only in the archive.

**Workaround:** When the agent's curated answer seems incomplete, ask it to cross-reference against the archive: "Check `archive/# [topic].txt` for anything the topic file missed."

### 7. Agent can't verify against live official docs
The agent works with local files only. It cannot fetch Anthropic's current documentation to verify that its recommendations are still accurate.

**Workaround:** For high-stakes configurations, manually verify recommendations against https://docs.anthropic.com/en/docs/claude-code.

### 8. Planned guides not yet created
The workspace design calls for additional guides (`hook-design-guide.md`, `memory-strategy-guide.md`) that haven't been written yet.

**Workaround:** The topic files (`docs/topics/hooks.md`, `docs/topics/memory.md`) contain patterns and anti-patterns sections that serve as interim guidance. Ask the agent in CONSULT mode for the same depth a guide would provide.

---

## 8. Quick Reference Card

### Opening the workspace
```bash
cd /path/to/CLAUDE-DB-KNWLDG && claude
```

### Exact prompts by mode

| Mode | Prompt pattern |
|---|---|
| **CONSULT** | "I want to [goal]. How should I [approach]?" |
| **COACH** | "Review this [config type]: [paste content]" |
| **REFERENCE** | "What is the [syntax/behavior/option] for [feature]?" |

### Cross-workspace command
```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir /path/to/CLAUDE-DB-KNWLDG
```

### Topics covered (v1)

| Topic | File | Lines |
|---|---|---|
| CLAUDE.md design | `docs/topics/claude-md.md` | 156 |
| Hooks system | `docs/topics/hooks.md` | 391 |
| Memory system | `docs/topics/memory.md` | 145 |
| Permissions | `docs/topics/permissions.md` | 164 |
| Settings | `docs/topics/settings.md` | 250 |
| Workflows | `docs/topics/workflows.md` | 173 |

### Archive fallback
```
Read archive/# [Topic Name].txt and tell me about [specific detail].
```

### Field note template
```markdown
## [Title]
**Context:** [project type, OS, team size]
**Pattern:** [what happened]
**Outcome:** [result]
**Confidence:** emerging | consistent
**Caveats:** [when this might not apply]
```

### Out of scope (v1)
Plugins, MCP servers, skills/commands, multi-agent orchestration, Claude API/SDK.
