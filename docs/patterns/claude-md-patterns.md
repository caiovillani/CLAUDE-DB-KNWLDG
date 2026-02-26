# CLAUDE.md Patterns & Anti-Patterns

## Proven Patterns

### 1. The Concise Imperative Style

Write rules as direct commands, not explanations.

```markdown
# Code Style
- Use ES modules (import/export), not CommonJS (require)
- Destructure imports: `import { foo } from 'bar'`
- 2-space indentation, no tabs

# Workflow
- Run `npm run typecheck` after code changes
- Run single tests, not the full suite
```

**Why it works:** Claude processes instructions, not essays. Direct commands are harder to misinterpret.

### 2. The Emphasis Hierarchy

Use emphasis markers for critical rules that Claude must never violate:

```markdown
# IMPORTANT: Security
- YOU MUST never commit .env files
- NEVER use --no-verify when committing
- ALWAYS validate user input before database queries

# Preferences (lower priority)
- Prefer early returns over nested conditionals
- Use descriptive variable names
```

**Why it works:** Claude respects emphasis markers. "IMPORTANT" and "YOU MUST" significantly increase adherence.

### 3. The Command Reference Block

Include exact commands Claude can't guess:

```markdown
# Commands
- Build: `npm run build:prod`
- Test single file: `npm run test -- --grep "pattern"`
- Lint fix: `npm run lint -- --fix`
- DB migrate: `npx prisma migrate dev`
- Type check: `npx tsc --noEmit`
```

**Why it works:** Claude guesses commands based on typical projects. Your project's commands may differ.

### 4. The Monorepo Structure

For monorepos, use hierarchical CLAUDE.md files:

```
project/
├── CLAUDE.md                   # Shared conventions
├── packages/
│   ├── frontend/
│   │   └── CLAUDE.md           # React-specific rules
│   ├── backend/
│   │   └── CLAUDE.md           # API-specific rules
│   └── shared/
│       └── CLAUDE.md           # Shared lib conventions
```

Child CLAUDE.md files load on-demand when Claude works in those directories.

### 5. The Import-Based Modular Setup

Keep CLAUDE.md lean by importing details:

```markdown
# Project: My App
See @README.md for overview and @package.json for commands.

# Standards
- Follow conventions in @docs/coding-standards.md
- Git workflow: @docs/git-workflow.md

# Personal
- @~/.claude/my-preferences.md
```

**Why it works:** Keeps the always-loaded file short while deep reference is available on demand.

### 6. The Rules Directory for Teams

Organize by domain, scope by file path:

```
.claude/rules/
├── typescript.md     # paths: ["**/*.{ts,tsx}"]
├── python.md         # paths: ["**/*.py"]
├── testing.md        # paths: ["**/*.test.*", "**/*.spec.*"]
├── api.md            # paths: ["src/api/**/*"]
└── general.md        # no paths (applies everywhere)
```

**Why it works:** Rules only load when relevant, keeping context clean. Team members can own specific rule files.

---

## Anti-Patterns

### 1. The Encyclopedia

```markdown
# BAD: This is a novel, not instructions
Our application uses a microservices architecture with React on the frontend
and Node.js on the backend. The frontend communicates with the backend through
a REST API. We use PostgreSQL for our database and Redis for caching. The
deployment pipeline uses GitHub Actions to build Docker images that are deployed
to AWS ECS. The monitoring stack includes DataDog for metrics and PagerDuty
for alerting...
```

**Why it fails:** Claude can discover all of this by reading code. This just wastes context.

**Fix:** Only include what Claude can't infer.

### 2. The Wishful Thinker

```markdown
# BAD: Vague aspirations
- Write clean, maintainable code
- Follow best practices
- Keep things simple
- Write good tests
```

**Why it fails:** These are too vague to act on. Claude already tries to do these things.

**Fix:** Be specific: "Use 2-space indent" not "format properly". "Test edge cases for null input" not "write good tests".

### 3. The Contradiction Machine

```markdown
# BAD: Conflicting instructions
- Always use TypeScript strict mode
- Don't add type annotations unless necessary  # contradicts strict mode
- Use any when types are complex              # contradicts strict mode
```

**Why it fails:** Claude picks whichever instruction it encounters last or most recently in context.

**Fix:** Review for internal consistency. Rules should reinforce each other, not conflict.

### 4. The Never-Updated File

A CLAUDE.md that hasn't changed since `/init` generated it months ago.

**Why it fails:** The project evolves. Commands change. Conventions shift. Stale rules cause stale mistakes.

**Fix:** Review monthly. After every major architectural change, audit CLAUDE.md.

### 5. The Secrets Leaker

```markdown
# BAD: Credentials in CLAUDE.md
- API key: sk-abc123...
- Database URL: postgres://admin:password@prod-db.internal...
```

**Why it fails:** CLAUDE.md is committed to git. Secrets in version control are a security incident.

**Fix:** Reference environment variables: "API key is in .env as `API_KEY`". Use `.env` files (which should be in `.gitignore`).

## Source

Synthesized from: `docs/topics/claude-md.md`, `archive/# Best Practices for Claude Code.txt`, `archive/# Manage Claude's memory.txt`
