# wrap

A Claude Code / Codex / OpenCode **skill** that consolidates every scattered descriptive markdown file in a project (`README.md`, `AGENTS.md`, `HANDOFF.md`, `docs/*`) into a single canonical `CLAUDE.md` and reconciles it against the current code.

After one run, the next session — yours, a teammate's, or a fresh agent's — has exactly one entry point.

## Why

Working with AI coding agents tends to produce documentation sprawl: a `HANDOFF.md` from one session, an `AGENTS.md` from another, a half-stale `README.md`, a `docs/` folder with three-week-old design notes. Each session's "context dump" creates a new file, and over time the project loses its single source of truth.

`wrap` does the opposite. Each invocation:

1. Reads every descriptive `.md` in the project
2. Scans the actual code (entry points, scripts, routes, configs)
3. Folds it all into one `CLAUDE.md` with a fixed structure
4. **Deletes the originals** so there's nothing to drift back out of sync
5. Pins a `PROGRESS` block at the top answering: *where are we now*, *what's the next concrete action*

## Install

Skills live in `~/.claude/skills/<skill-name>/`. To install `wrap`:

```bash
git clone https://github.com/recomby-ai/wrap-skill.git
mkdir -p ~/.claude/skills/wrap
cp wrap-skill/SKILL.md ~/.claude/skills/wrap/SKILL.md
```

Or as a one-liner:

```bash
mkdir -p ~/.claude/skills/wrap && curl -fsSL https://raw.githubusercontent.com/recomby-ai/wrap-skill/main/SKILL.md -o ~/.claude/skills/wrap/SKILL.md
```

That's it. Restart your agent and the skill is available.

## Use

Just say one of these in any project:

- `wrap` / `wrap up` / `/wrap`
- `tidy docs` / `sync docs` / `update docs` / `archive`
- `this phase is done`
- Chinese: `收尾` / `同步文档` / `整理文档` / `整理一下` / `更新文档` / `存档` / `梳理一下` / `这阶段做完了`

The agent will:

- Inventory every `.md` and cross-check it against the real code
- Write a fresh `CLAUDE.md` with `PROGRESS / Architecture / Run / API / Known Limits / Handoff` sections
- Delete the merged-in source files
- Print a short change summary

## Output Format

`CLAUDE.md` follows a fixed skeleton:

```
# <project>

<one-line summary>

## PROGRESS
**Current**: ...
**Next**: ...
**Known outstanding**: ...

## Architecture
## Run
## API / Interfaces     (skipped if project has no external interfaces)
## Known Limits
## Handoff
```

Soft cap of 250 lines. If a project genuinely needs more (multi-service runbooks, large credential tables), the skill applies for a documented exemption rather than padding or hand-waving.

## Design Principles

- **One file**. Zero is too few; two is too many.
- **PROGRESS is the soul**. Other sections survive a stale week; PROGRESS dies in a day. Rewrite it every run.
- **Brevity beats completeness**. Decision history and design rationale go in the agent's memory system, not in `CLAUDE.md`.
- **Code is the source of truth**. When `CLAUDE.md` conflicts with the code, the code wins — always.

## Compatibility

Tested with Claude Code. Should work in any agent that supports the SKILL.md format (Codex, OpenCode, Cursor, Gemini CLI, etc.).

The skill only touches files inside the project directory. It does not interact with any agent's memory system — those are decoupled by design.

## License

MIT — see [LICENSE](./LICENSE).
