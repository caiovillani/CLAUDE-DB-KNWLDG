# Common Mistakes & How to Avoid Them

A catalog of frequently observed failure patterns when using Claude Code, with diagnosis and fixes.

---

## Session Management Mistakes

### 1. The Kitchen Sink Session

**Symptom:** You start with one task, switch to something unrelated, go back to the first task. Claude starts confusing context from both tasks.

**Root cause:** Context window full of irrelevant information from mixed tasks.

**Fix:** `/clear` between unrelated tasks. Each task gets clean context.

### 2. The Correction Spiral

**Symptom:** Claude does something wrong. You correct it. Still wrong. Correct again. After 3-4 cycles, output quality degrades.

**Root cause:** Context polluted with failed approaches. Claude tries to reconcile all the contradictory attempts.

**Fix:** After two failed corrections, `/clear` and start fresh. Write a better initial prompt that incorporates what you learned from the failures.

### 3. The Infinite Exploration

**Symptom:** You ask Claude to "investigate" or "look into" something without scoping it. Claude reads dozens of files, fills the context, and produces a vague summary.

**Root cause:** Unbounded investigation scope.

**Fix:**
- Scope narrowly: "look at src/auth/ and explain how token refresh works"
- Use subagents for wide exploration (separate context, reports back summary)
- Set explicit boundaries: "check only the 3 most relevant files"

### 4. Never Using /clear

**Symptom:** Long sessions where quality gradually degrades. Claude seems to "forget" earlier instructions.

**Root cause:** Context window approaching limits. Auto-compaction kicks in but can't preserve everything.

**Fix:** Proactively `/clear` between logical task boundaries. Don't wait for degradation.

---

## CLAUDE.md Mistakes

### 5. The Bloated CLAUDE.md

**Symptom:** Claude ignores important rules. You add MORE rules. Claude ignores even more.

**Root cause:** Signal-to-noise ratio too low. Important instructions get lost in noise.

**Fix:** Prune ruthlessly. For each line ask: "Would removing this cause Claude to make mistakes?" If not, cut it. Aim for 50-80 lines max. Move detailed reference to skills or linked docs.

### 6. Documentation Disguised as Instructions

**Symptom:** CLAUDE.md reads like a wiki article about the project. Claude doesn't follow the "instructions" because they're descriptive, not prescriptive.

**Root cause:** Confusing description (about the project) with instruction (for Claude).

**Fix:** Write imperatives: "Use X" not "The project uses X". "Run Y before committing" not "Y is our CI system".

### 7. Personal Preferences in Team Files

**Symptom:** Team friction. Different developers want different things in CLAUDE.md.

**Root cause:** Mixing personal preferences with team standards.

**Fix:** Team standards → `CLAUDE.md` (committed). Personal preferences → `CLAUDE.local.md` (gitignored) or `~/.claude/CLAUDE.md` (global).

---

## Permission Mistakes

### 8. Over-Permissive Bash Allows

**Symptom:** Claude runs commands you didn't expect. Security risk.

**Root cause:** `Bash(*)` or overly broad patterns in allow rules.

**Fix:** Allow specific commands. Combine with sandboxing for defense in depth. Use deny rules for dangerous patterns.

### 9. Fragile URL Permission Patterns

**Symptom:** Permission rule like `Bash(curl http://github.com/ *)` doesn't actually restrict to GitHub.

**Root cause:** Bash permission patterns can't reliably constrain URLs (options before URL, different protocols, redirects, variables).

**Fix:** Block `curl`/`wget` in Bash, use `WebFetch(domain:github.com)` for allowed domains instead.

### 10. bypassPermissions in Production

**Symptom:** Data loss, unexpected system changes, or security issues.

**Root cause:** Using `--dangerously-skip-permissions` outside of isolated containers.

**Fix:** Only use in containers without internet access. For autonomous work, prefer sandboxing + curated allow rules.

---

## Workflow Mistakes

### 11. Skipping Plan Mode for Complex Changes

**Symptom:** Claude implements something, but it's the wrong approach. Hours wasted.

**Root cause:** Jumping to implementation without understanding the problem space.

**Fix:** For multi-file changes or unfamiliar code, start in Plan Mode. Explore → Plan → Implement → Commit.

### 12. The Trust-Then-Verify Gap

**Symptom:** Claude's output looks plausible but doesn't handle edge cases, or breaks in production.

**Root cause:** No verification step. Claude's work wasn't tested.

**Fix:** ALWAYS provide verification criteria:
- Include test commands
- Share expected outputs
- Ask Claude to run tests after implementing
- Specify "fix it AND verify the fix" not just "fix it"

### 13. Not Using Subagents for Research

**Symptom:** Your main context is full of file contents from Claude's exploration. Quality drops for the actual implementation.

**Root cause:** Exploration polluted the implementation context.

**Fix:** "Use a subagent to investigate X and report findings." The subagent explores in its own context and returns a summary.

---

## Hook Mistakes

### 14. Hooks for Everything

**Symptom:** Slow session startup. Every tool call takes noticeably longer.

**Root cause:** Too many hooks firing on too many events with broad matchers.

**Fix:** Use hooks only when enforcement is mandatory (zero exceptions). Use CLAUDE.md for advisory guidance. Use permissions for access control.

### 15. Synchronous Heavy Hooks

**Symptom:** Claude pauses for seconds after each tool call.

**Root cause:** Hook runs slow operations (full test suite, complex lint) synchronously.

**Fix:** Use async hooks for heavy operations. Keep synchronous hooks fast (<1 second).

---

## Source

Synthesized from: `archive/# Best Practices for Claude Code.txt`, `docs/topics/` reference files
