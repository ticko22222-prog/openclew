# living-memory

> Long Life Memory for LLMs

**Your project's knowledge lives in one place — for you, your team, and your AI agents.**

Decisions, architecture, conventions, lessons learned. Structured in 3 levels so humans skim what they need and agents parse only what's relevant. Git-versioned, always in sync.

**living-memory** is a simple convention — just Markdown files. Not a framework, not a tool. Everyone shares the same source of truth.

---

## The problem

Project knowledge gets fragmented between humans and AI agents. Neither side has the full picture:

| Who | What they know | What they lose |
|-----|---------------|----------------|
| **You** | Architecture, decisions, context | Forget details, can't explain everything every session |
| **AI agent** | What's in the current context window | Everything from previous sessions |
| **New teammate** | Nothing yet | Has to ask, read scattered docs, or guess |

The workarounds don't scale:

| Approach | Problem |
|----------|---------|
| CLAUDE.md / .cursorrules | Grows into an unreadable wall of text — not great for humans or agents |
| Agent memory (Claude, Copilot) | Opaque, not versioned, not shareable with the team |
| README.md | Not structured for AI consumption |
| Wiki / Notion | Disconnected from the code, goes stale |
| Nothing | Re-explain everything every session |

## The solution: 3 levels of information

Every document has 3 levels. Same file, different depths — for different readers.

```
┌─────────────────────────────────────────────┐
│  L1 — Metadata                              │
│  type, subject, status, keywords            │
│  → Agents parse this to navigate fast       │
│  → Auto-indexed, searchable                 │
├─────────────────────────────────────────────┤
│  L2 — Summary                               │
│  Objective, key points, solution            │
│  → Humans read this to get up to speed      │
│  → New teammate onboards in minutes         │
├─────────────────────────────────────────────┤
│  L3 — Details                               │
│  Code, examples, history, references        │
│  → Deep-dive when you actually need it      │
│  → Agents or humans, only when relevant     │
└─────────────────────────────────────────────┘
```

**For agents:** A project with 50 docs would burn 100K+ tokens if read in full. With L1, it scans all 50 in ~2K tokens, then reads only the 2-3 it needs.

**For humans:** L2 gives you the full picture in 30 seconds — no need to read code or ask a colleague. New teammates onboard by reading L2s.

## Two types of docs

| Type | Location | Purpose | Mutability |
|------|----------|---------|------------|
| **Permanent** | `doc/_SUBJECT.md` | Living knowledge (architecture, conventions, decisions) | Updated over time |
| **Log** | `doc/log/YYYY-MM-DD_subject.md` | Frozen facts (what happened, what was decided) | Never modified |

**Permanents** are your project's brain — they evolve.
**Logs** are your project's journal — they're immutable.

## Quick start (5 minutes)

### 1. Create the structure

```bash
mkdir -p doc/log
```

### 2. Copy the templates

Download from [`templates/`](templates/) or create manually:

<details>
<summary><b>templates/permanent.md</b> — for living knowledge</summary>

```markdown
<!-- L1_START -->
# L1 - Metadata
type: Reference | Architecture | Guide | Analysis
subject: Short title (< 60 chars)
created: YYYY-MM-DD
updated: YYYY-MM-DD
short_story: 1-2 sentences. What this doc covers and what it concludes.
status: Active | Stable | Archived
category: Main domain (e.g. Auth, API, Database, UI...)
keywords: [tag1, tag2, tag3]
<!-- L1_END -->

---

<!-- L2_START -->
# L2 - Summary

## Objective
<!-- Why this document exists -->

## Key points
<!-- 3-5 essential takeaways -->

## Solution
<!-- Recommended approach or pattern -->
<!-- L2_END -->

---

<!-- L3_START -->
# L3 - Details

<!-- Full technical content: examples, code, references... -->

## Changelog

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial creation |
<!-- L3_END -->
```

</details>

<details>
<summary><b>templates/log.md</b> — for frozen facts</summary>

```markdown
<!-- L1_START -->
# L1 - Metadata
date: YYYY-MM-DD
type: Bug | Feature | Refactor | Doc | Deploy
subject: Short title (< 60 chars)
short_story: 1-2 sentences. What happened and what was the outcome.
status: Done | In progress | Abandoned
category: Main domain
keywords: [tag1, tag2, tag3]
<!-- L1_END -->

---

<!-- L2_START -->
# L2 - Summary

## Problem
<!-- What was observed -->

## Solution
<!-- How it was resolved -->
<!-- L2_END -->

---

<!-- L3_START -->
# L3 - Details

<!-- Technical details: code changes, debugging steps, references... -->
<!-- L3_END -->
```

</details>

### 3. Write your first doc

```bash
cp templates/permanent.md doc/_ARCHITECTURE.md
```

Edit it — describe your project's architecture. Fill in L1 (metadata), L2 (summary), skip L3 if you don't need it yet.

### 4. Point your agent to it

Add this to your `CLAUDE.md`, `.cursorrules`, or `AGENTS.md`:

```markdown
## Project knowledge

Documentation lives in `doc/`. Each doc has 3 levels (L1/L2/L3).
- Read L1 first to decide if you need more
- Permanent docs: `doc/_*.md` (living knowledge, updated)
- Logs: `doc/log/YYYY-MM-DD_*.md` (frozen facts, never modified)
- Index: `doc/_INDEX.md` (auto-generated, start here)
```

### 5. Auto-generate the index (optional)

Copy [`hooks/generate-index.py`](hooks/generate-index.py) to your project and add it as a pre-commit hook:

```bash
# Option A: git hook
cp hooks/generate-index.py .git/hooks/generate-index.py
echo 'python .git/hooks/generate-index.py && git add doc/_INDEX.md' >> .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit

# Option B: pre-commit framework
# See hooks/README.md for .pre-commit-config.yaml setup
```

The index auto-regenerates on every commit. Never edit it manually.

---

## How it works in practice

**Session 1** — You're setting up auth:
```
doc/
├── _ARCHITECTURE.md          # Your stack, main patterns
└── log/
    └── 2026-03-07_setup-auth.md   # What you did, decisions made
```

**Session 5** — New agent session, different feature:
```
Agent reads doc/_INDEX.md (auto-generated)
  → Sees _ARCHITECTURE.md, reads L1 → "I need this" → reads L2
  → Sees 4 logs, reads L1s → "setup-auth is relevant" → reads L2
  → Skips the rest
  → Has full context in ~1K tokens instead of 50K
```

**Session 20** — Your project has grown:
```
doc/
├── _INDEX.md                      # Auto-generated, 30 entries
├── _ARCHITECTURE.md               # Updated 12 times
├── _AUTH.md                       # Extracted when auth got complex
├── _API_CONVENTIONS.md            # Team conventions
├── _KNOWN_ISSUES.md               # Active gotchas
└── log/
    ├── 2026-03-07_setup-auth.md
    ├── 2026-03-10_migrate-db.md
    ├── 2026-03-15_fix-token-refresh.md
    └── ... (20 more)
```

The agent scans 30 L1 headers in 2 seconds, reads the 3 docs it needs, and starts working with full context. A new teammate does the same by reading L2s — same docs, same truth, different depth.

---

## Key principles

- **Shared knowledge** — Same docs for humans and AI. No separate "AI docs" and "human docs". One source, multiple readers.
- **SSOT** (Single Source of Truth) — Each piece of information lives in one place. No duplication.
- **Logs are immutable** — Once written, a log is never modified. It's a frozen fact.
- **Permanents are living** — They evolve as the project evolves. Always up to date.
- **Index is auto-generated** — Never edit `_INDEX.md` manually. It's rebuilt from L1 metadata.
- **L1 is always enough to decide** — If a reader (human or agent) can't tell from L1 whether they need the doc, the L1 is poorly written.

---

## Works with everything

**AI agents:** Claude Code, Cursor, Copilot, Windsurf, Codex, Zed, Kiro, Aider, Cline, Gemini CLI...

**Workflow frameworks:** BMAD, Spec Kit, or any methodology — living-memory handles knowledge, your framework handles process.

**It's just Markdown.** No runtime, no dependencies, no lock-in. Git-versioned, diffable, reviewable in PRs. If you stop using it, the docs are still useful — to humans and agents alike.

---

## Compared to alternatives

| Feature | CLAUDE.md | Cline Memory Bank | BMAD | living-memory |
|---------|-----------|-------------------|------|---------------|
| Readable by humans AND agents | partial | partial | yes | **yes** |
| Levels of info (L1/L2/L3) | - | - | - | **yes** |
| Lazy-loading (read only what's needed) | - | - | partial | **yes** |
| Auto-generated index | - | - | CSV | **yes** |
| Immutable logs | - | - | - | **yes** |
| Git-versioned | yes | yes | yes | **yes** |
| Cross-project | - | - | - | **yes** |
| Tool-agnostic | Claude only | Cline only | multi | **yes** |

---

## License

MIT — use it however you want.
