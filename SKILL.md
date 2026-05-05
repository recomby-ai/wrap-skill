---
name: wrap
description: >
  Session wrap-up: consolidates every descriptive markdown file (README.md,
  AGENTS.md, HANDOFF.md, docs/*) into ONE canonical CLAUDE.md, then
  reconciles it against the current code state so a fresh session or new
  teammate can pick up immediately. CLAUDE.md keeps a mandatory PROGRESS
  block at the top answering two questions: where are we now and what's
  the next concrete action. Trigger when the user says: "wrap", "wrap up",
  "/wrap", "tidy docs", "sync docs", "update docs", "archive", "this phase
  is done", or the Chinese equivalents "收尾", "同步文档", "整理文档",
  "整理一下", "更新文档", "存档", "梳理一下", "这阶段做完了". A bare
  "tidy" / "整理" with prior dev context counts. Do not under-trigger.
---

# wrap — Single-File Project State Maintenance

You are a **project documentation editor**. Each time you are invoked, fold every scattered descriptive markdown file in the project into a single `CLAUDE.md`, and make sure it aligns with the current truth of the code. After this skill runs, that one file is the entry point for whoever picks the project up next.

## Three Non-Negotiable Principles

1. **One file**: Only `<project-root>/CLAUDE.md` survives. `README.md` / `AGENTS.md` / `HANDOFF.md` / `docs/*` get merged into CLAUDE.md and **then deleted**. The "handoff guide" lives inside CLAUDE.md too — do not spawn another file for it.
2. **PROGRESS section pinned at the top**: It always answers two questions — "where are we now" and "what is the next concrete action". The next action must be executable, not vague (don't write "consider optimizing the metric"; write "academic-1 still has the 0.95 edge case, next step is tightening the general-topic phrase exclusion list in ner.js").
3. **Brevity first**: When in doubt, cut. Design motivation, decision history, long-form rationale belong in the memory system, not in CLAUDE.md. CLAUDE.md only carries the facts a successor needs.

## Execution Flow

### Step 1: Mechanical Inventory

Don't skip. List first, judge second.

1. `ls <project-root>` to see the root
2. `find <project-root> -maxdepth 2 -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*"` to grab every .md
3. Read every .md (README.md / AGENTS.md / CLAUDE.md / HANDOFF.md / CHANGELOG.md / docs/*.md / any descriptive .md)
4. `ls <project-root>/docs/ 2>/dev/null` to confirm whether a docs folder exists
5. Scan the current truth of the code:
   - Entry files, main directory layout
   - `package.json` / `pyproject.toml` / `Cargo.toml` etc. — scripts and dependencies
   - Main API routes (if it's a web project)
   - Config files (wrangler.toml / vercel.json / docker-compose.yml etc.)
   - Test fixtures and expectations (if any)
6. Review every change made in the current conversation

Produce an internal checklist (don't show the user): for each existing .md, mark one of three — "merge into CLAUDE.md / delete / keep". **README.md / AGENTS.md / HANDOFF.md / docs/* default to merge-then-delete** — that's the core action of this skill.

### Step 2: Construct CLAUDE.md

Use this fixed structure. Keep each section short.

```markdown
# <project name>

<1-line summary of what this is>

> Live: <deployment URL, if any>

## PROGRESS

**Current**: <what state the project is in, what the most recent meaningful change was. 1-3 lines>

**Next**: <executable concrete actions. 1-3 bullets, each with a trigger condition or acceptance criterion>

**Known outstanding**: <recorded but unresolved issues. 1-3 bullets. Write "none" if there are none>

## Architecture

<directory tree or layer diagram, max 30 lines. Only what helps a successor; no node_modules>

## Run

<local startup / test / deploy commands. Code block.>

## API / Interfaces

<If the project exposes external interfaces, list endpoints + inputs + outputs. Otherwise skip this section.>

## Known Limits

<things the design cannot do / known gotchas. 3-8 bullets>

## Handoff

<non-obvious facts a new teammate needs: env vars, where secrets live, required local config, common pitfalls>
```

Writing notes per section:

- **PROGRESS is the soul of this file** — other sections still work after a week of staleness; PROGRESS is dead after one day. Rewrite it every time the skill runs.
- Architecture uses the real directory tree (`tree -L 2 -I 'node_modules|.git'` or similar) — never draw it from memory.
- Run commands must be copy-paste runnable. No `<your-token>` placeholders — point to the file where it's configured.
- API section uses the real endpoints scanned from code, not copied from old docs.
- Known Limits documents **gotchas already encountered**, not "future maybes".
- Handoff documents **facts you can't learn from reading the code** — e.g., "where to put secrets in .dev.vars", "the KV namespace ID is hardcoded in wrangler.toml, no need to recreate it".

### Step 3: Actually Modify

Use tools, don't just describe.

Order:
1. **Write CLAUDE.md** (Write the full file — don't Edit incrementally; a full rewrite is cleanest)
2. **Delete the old README.md / AGENTS.md / HANDOFF.md / CHANGELOG.md / docs/*.md** (via Bash `rm`), only after their content has been merged into CLAUDE.md
3. **If `docs/` is now empty, remove `docs/`**
4. Don't touch code, config files, `.env*`, or secrets

Edge cases:
- Brand-new project, no README and no CLAUDE.md → create a skeleton CLAUDE.md, PROGRESS says "project just started, no runnable code yet"
- No package.json / config file → Run section says "no standard startup flow yet, needs to be filled in"
- Project is a git repo → before deleting just `git mv` or plain `rm`; git tracks the removal. No special archive needed (history lives in git).

### Step 4: Self-Check

Tick each item. If you can't, go back and fix.

- [ ] Only one descriptive .md remains in the project: `CLAUDE.md` (and `docs/` is gone if it was emptied)
- [ ] PROGRESS has all three sub-sections; "next" is an executable concrete action, not vague language
- [ ] Run commands actually run (literally cross-checked against package.json / Makefile)
- [ ] Architecture's directory matches the real `ls`
- [ ] API endpoints are scanned from code, not copied from an old README
- [ ] No relative time leftovers (grep "today|yesterday|recently|just now|今天|昨天|最近|刚刚"; all replaced with absolute YYYY-MM-DD dates)
- [ ] No "TODO fill in later" — either fill it or remove it
- [ ] CLAUDE.md total length ≤ 250 lines — **soft cap**: compress as hard as you can first; if it's still > 250 lines and every line is a successor-essential fact (typical case: multi-service runbook, complex credential/parameter table), apply for an exemption — in Step 5 "Change Summary" add a `### Length Exemption` block stating "actual line count / compression actions taken / why the rest can't be cut". Once that's written it counts as approved; don't agonize repeatedly.

### Step 5: Change Summary

Concise. Only list real changes.

```
## Sync complete

### CLAUDE.md
- Updated: <section> — <one-line reason>
- Rewrote: PROGRESS

### Deleted (content merged into CLAUDE.md)
- README.md
- docs/architecture.md
- ...

### Unresolved
- <ambiguity needing user input> (write only if any)

### Length Exemption (only if CLAUDE.md > 250 lines)
- Actual lines: <N>
- Compression done: <e.g., merged X sections / removed redundant examples / tabularized credentials>
- Why the rest can't be cut: <e.g., 8 sets of service credentials all successor-essential, removing any forces another ssh trip>
```

## Anti-Patterns (Don't)

- Long "why we designed it this way" passages in CLAUDE.md — that goes in the memory system
- Writing CLAUDE.md but keeping README.md "because GitHub needs it" — delete README; deal with it when you actually publish
- Writing PROGRESS as "ongoing optimization / progress is good" — that's the same as not writing it. Drill down to the next action's input/output.
- Forcing an API section just to fill the structure on a project with no API — skip the section
- After editing code, finding CLAUDE.md conflicts with the code, and "trusting the doc over the code" — code's current state is always the source of truth

## Cross-Platform

Runs across agents (Claude Code / Codex / OpenCode). The skill only touches files inside the project directory; it does not touch each agent's own memory system (that's the memory tool's job and is decoupled from this skill).
