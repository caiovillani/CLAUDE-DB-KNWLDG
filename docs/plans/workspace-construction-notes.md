# Workspace Construction Notes

Original design insights and user instructions captured during the initial build of the CLAUDE-DB-KNWLDG workspace. Preserved for provenance and context.

---

The CLAUDE.md serves as the workspace's constitution — it defines the pipeline stages, naming conventions, placement rules, and self-governance protocols. Without it, every Claude Code session starts from zero and the "demi live structure" concept collapses into session-specific ad-hoc decisions. The design doc in docs/plans/ already has excellent architecture definitions that I'll distill into actionable rules.

About CLAUDE.md content, you are authorized to draft it. You must do it in plan-mode and execute self-critical-review, self heal, and self correction before finalizing.

The CLAUDE.md is the keystone for everything you described — "demi live structure", "self governance", "continuous refinement." Without it, each session is amnesiac. The CLAUDE.md acts as the workspace's constitution: it tells Claude Code where things go, how to name them, what the pipeline stages are, and what to check on every session. It's what makes the workspace self-healing instead of entropy-accumulating.

Your input matters here: The CLAUDE.md content — especially naming convention and pipeline rules — involves design trade-offs that shape every future interaction.

Git detected all 5 moves as renames (not delete+add), which preserves git log --follow history for each file. This only works when you stage both the deletion and the new file together — exactly what git add -A does here.

---

## Original User Instructions

> I need you to brainstorm about this folder's workspace and think, reflect and reason about it to develop HIGH quality understanding and comprehension. Also, it's REALLY relevant to keep thinking, reflecting, reasoning and reviewing the whole workspace from time to time in an automate way to self sanitize it, self heal and self correct. This workspace MUST work as demi live structure that keeps it's self governance on track and that builds continuos understanding, continuos knowledge base and continuos refinement. Think (medium effort) about my instructions and think (high effort) about the workspace scope, structure, recommended refinements, NECESSARY refinements, MANDATORY refinements. --- self-critical-review, self heal, self correct. ---

---

## Key Design Insight

The CLAUDE.md Self-Governance Protocol section is the mechanism that makes this workspace "demi live." Every future Claude Code session will read it on startup and know: (1) where files belong, (2) how to name them, (3) what to check for drift. The checklist-driven approach means hygiene happens automatically — orphans, duplicates, stale artifacts, and naming violations get caught before they accumulate. The memory file provides cross-session continuity for known issues and evolution history.
