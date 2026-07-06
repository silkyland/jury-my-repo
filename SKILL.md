---
name: jury-my-repo
description: >-
  Multi-agent adversarial audit: detects which AI coding agent CLIs are
  installed (Claude Code, Codex, Gemini CLI, OpenCode, Kimi, Cursor, Qwen,
  Aider, Goose, Crush, Droid, Copilot), runs the willing ones in parallel
  headless mode against the same audit brief in isolated read-only
  worktrees, collects each agent's bug/security/architecture findings
  separately, then cross-examines every claim against the actual code —
  CONFIRMED, REFUTED, or DUPLICATE — producing a per-agent scoreboard
  (precision, unique catches, hallucination rate) and one distilled,
  verified defect report. Use when the user wants a multi-agent audit or
  review, a second opinion from other AI agents, to compare coding agents
  on their codebase, or mentions jury-my-repo or /jury-my-repo.
license: MIT
argument-hint: "[project-root] [focus: bugs/security/architecture/all]"
---

# Jury My Repo

One auditor has blind spots; a jury has fewer. This skill drafts every AI
coding agent installed on the machine into an audit panel: same brief, same
codebase, independent verdicts — then cross-examines their claims against
the code and scores who was right, who was wrong, and who hallucinated.
What survives is a defect list no single agent would have produced.

**Requirements:** shell access and git (worktrees) in the target repo, plus
at least one other agent CLI installed. Agents without terminal access
(IDE-embedded assistants) cannot run this skill — say so instead of
improvising a partial version.

## The Prime Directive (family rule)

> **An agent's claim is testimony, not truth.** Every finding from every
> agent — including the one running this skill — is verified against the
> actual code (`file:line` opened and read) before it enters the final
> report. The scoreboard counts verified reality, not confidence.

## Hard rules

1. **Consent gate ⛔ — never launch without it.** Each juror burns the
   user's tokens/subscription on ITS OWN account. Present the detected
   agents and get explicit approval of the roster before any run.
2. **Read-only discipline.** Auditors must not modify the repo. Each juror
   runs in its own disposable git worktree; per-CLI sandbox flags are used
   where they exist; `git status` is checked after every run and any
   mutation is discarded and noted in the report.
3. **Same brief for everyone.** Comparability requires identical input —
   one brief, one output contract (see
   [references/audit-brief.md](references/audit-brief.md)).
4. **Judge bias disclosed.** The orchestrating agent is usually also a
   juror. Mitigation: verification is mechanical (open file, check claim),
   REFUTED requires stated evidence, and the orchestrator's own findings
   get no special treatment. State this in the report.

## Progress checklist

Copy this into your response and check items off:

```
Jury Progress:
- [ ] Step 1: Detect jurors — probe PATH, verify headless mode per CLI
- [ ] Step 2: Consent gate — roster + cost warning approved by user  ⛔
- [ ] Step 3: Brief — audit brief written to .jury/brief.md
- [ ] Step 4: Parallel run — each juror in its own worktree, timeboxed
- [ ] Step 5: Collect — per-agent raw reports preserved verbatim
- [ ] Step 6: Cross-examine — every claim verified against code
- [ ] Step 7: Scoreboard + distilled report written
```

## Step 1 — Detect the jurors

Probe for known CLIs using the catalog in
[references/agent-cli-catalog.md](references/agent-cli-catalog.md)
(`command -v` per CLI, then `--version`). For each one found, **verify its
headless invocation from its own `--help` this run** — flags drift and the
catalog is a starting point, not gospel. Record: available / no headless
mode / not authenticated (a login-wall discovered now beats a hung run
later — test with a trivial prompt if uncertain).

## Step 2 — Consent gate ⛔

Present the roster table: agent, version, headless command, sandbox level,
rough cost note (uses the user's own plan/API key for that vendor). Ask ONE
question: which jurors to empanel (recommended: all detected, minus any the
user is rate-limited on). Do not proceed without an explicit answer.

## Step 3 — Write the brief

Generate `.jury/brief.md` from the template in
[references/audit-brief.md](references/audit-brief.md), scoped to the
user's focus (bugs / security / architecture / tests / all). The brief
pins the output contract: findings with `file:line`, severity, category,
and a one-line reproduction/reasoning — so reports are mergeable.

## Step 4 — Parallel run

For each approved juror:

1. Create a disposable worktree: `git worktree add .jury/wt-<agent> HEAD`.
2. Launch headless with the catalog invocation, the brief as prompt, output
   captured to `.jury/reports/<agent>.md`, run in the background.
3. Timebox (default 15 min) — kill and mark TIMEOUT rather than wait
   forever; a juror that hangs is a result, not a blocker.
4. On completion: `git -C .jury/wt-<agent> status --porcelain` — any
   mutation gets reverted and logged.

Run all jurors concurrently. While waiting, the orchestrator performs its
OWN audit against the same brief — it is juror #0, same rules.

## Step 5 — Collect

Preserve each report verbatim (they are the "ของใครของมัน" record). Parse
into a unified findings table: agent, claim, file:line, severity, category.
Findings that ignore the output contract are parsed best-effort and the
deviation noted — sloppy format is itself scoreboard data.

## Step 6 — Cross-examine

Per [references/verdict-protocol.md](references/verdict-protocol.md):

1. **Dedup** across agents (same defect, different words → one finding,
   all claimants credited).
2. **Verify each unique finding against the code** — open the cited file,
   check the claim: **CONFIRMED** (real, evidence quoted) / **REFUTED**
   (wrong, with the evidence why — this is the "ใครมั่ว" pile) /
   **UNVERIFIABLE** (needs runtime/creds — labeled, never promoted).
3. Verification is by reading code and, where cheap and safe, running the
   repo's own tests — never by majority vote. Three agents repeating the
   same hallucination is still a hallucination.

## Step 7 — Scoreboard and distilled report

Write `.jury/VERDICT.md`:

- **Scoreboard:** per agent — findings, confirmed, refuted, duplicates,
  unique confirmed catches, precision %. Name the sharpest juror and the
  noisiest, with numbers.
- **Distilled defect report:** CONFIRMED findings only, worst-first
  (security > data loss > correctness > architecture), each with evidence
  and every agent that caught it.
- **Disagreements worth human eyes:** UNVERIFIABLE items where jurors
  split — with what would settle each.
- **Handoff:** recommend deep-plan for the fix roadmap (Phase 0 = the
  confirmed security/correctness findings) — this skill audits; it does
  not fix.

Report cost honestly: which jurors ran, how long, which timed out or
errored. Clean up worktrees (`git worktree remove`); keep `.jury/reports/`
as the audit trail.
