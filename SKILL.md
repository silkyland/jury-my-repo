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
   agents and get explicit approval of the roster before any run. If no
   user can answer (headless/CI run), external jurors are NEVER launched —
   degrade as described in Step 2.
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
- [ ] Step 0: Preflight — git repo confirmed, .jury/ ignored, scope sized
- [ ] Step 1: Detect jurors — probe PATH, verify headless mode per CLI
- [ ] Step 2: Consent gate — roster + cost/time approved by user (headless ⇒ solo DEGRADED)  ⛔
- [ ] Step 3: Brief — audit brief written to .jury/brief.md
- [ ] Step 4: Parallel run — each juror in its own worktree, timeboxed
- [ ] Step 5: Collect — per-agent raw reports preserved verbatim
- [ ] Step 6: Cross-examine — every claim verified against code
- [ ] Step 7: Scoreboard + distilled report written, acceptance checklist passed
```

## Step 0 — Preflight

Before probing for jurors, verify the venue:

- `git rev-parse --show-toplevel` — must be a git repo (worktrees need it)
  with at least one commit; jurors audit `HEAD`, so warn the user if the
  interesting code isn't committed yet.
- Add `.jury/` to `.git/info/exclude` (not `.gitignore` — don't touch the
  user's tracked files).
- Size the scope: `git ls-files | wc -l` and the language mix. Over ~2k
  files, plan a path scope for the brief now (Step 3) instead of letting
  every juror wander.

## Step 1 — Detect the jurors

Probe for known CLIs using the catalog in
[references/agent-cli-catalog.md](references/agent-cli-catalog.md)
(`command -v` per CLI, then `--version`). For each one found, **verify its
headless invocation from its own `--help` this run** — flags drift and the
catalog is a starting point, not gospel. Record: available / no headless
mode / not authenticated (a login-wall discovered now beats a hung run
later — test with a trivial prompt if uncertain).

If ZERO other CLIs qualify (none installed, none authenticated, none with
a headless mode), stop here and say so: report what was probed and the
per-CLI result, then offer the honest fallback — a solo adversarial audit
by this agent alone, clearly labeled as one auditor's view. Never present
a one-agent run as a multi-agent verdict.

## Step 2 — Consent gate ⛔

Present the roster table: agent, version, headless command, sandbox level,
rough cost note (uses the user's own plan/API key for that vendor), and
estimated wall time (jurors run in parallel, so roughly one timebox). Ask
ONE question: which jurors to empanel (recommended: all detected, minus
any the user is rate-limited on). Do not proceed without an explicit
answer.

**Headless degraded mode:** if no user can answer (CI/scheduled run),
consent cannot be granted — run Steps 3–7 with juror #0 only and stamp
the verdict `DEGRADED: solo audit, no consent available`. Spending the
user's money on other vendors' accounts is never inferrable consent.

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
5. A juror that dies mid-run (auth expiry, quota, crash) is marked FAILED
   with the reason; the panel proceeds with the survivors. Remove its
   worktree even on failure (`git worktree remove --force`, then
   `git worktree prune`) — a crashed juror must not leave an orphan.

Canonical launch shape (adapt flags per the catalog + live `--help`):

```bash
( cd .jury/wt-<agent> && timeout 900 <cli> <headless-flags> \
    "$(cat ../../.jury/brief.md)" \
    > ../reports/<agent>.md 2> ../reports/<agent>.stderr.log;
  echo "exit=$?" >> ../reports/<agent>.md ) &
```

Run all jurors concurrently. While waiting, the orchestrator performs its
OWN audit against the same brief — it is juror #0, same rules. Write
juror #0's report to `.jury/reports/self.md` BEFORE reading any rival
report — reading others first contaminates the independent verdict.

## Step 5 — Collect

Preserve each report verbatim (each agent's own untouched record). Parse
into a unified findings table: agent, claim, file:line, severity, category.
Findings that ignore the output contract are parsed best-effort and the
deviation noted — sloppy format is itself scoreboard data.

Build the **coverage matrix** while parsing: per agent × numbered scope
item (S1…Sn from the brief) — addressed (findings or an explicit
"nothing found") or SILENT. A juror that never looked at a scope item did
not pass it; silence is scoreboard data too.

## Step 6 — Cross-examine

Per [references/verdict-protocol.md](references/verdict-protocol.md):

1. **Dedup** across agents (same defect, different words → one finding,
   all claimants credited).
2. **Verify each unique finding against the code** — open the cited file,
   check the claim: **CONFIRMED** (real, evidence quoted) / **REFUTED**
   (wrong, with the evidence why — the hallucination pile) /
   **UNVERIFIABLE** (needs runtime/creds — labeled, never promoted).
3. Verification is by reading code and, where cheap and safe, running the
   repo's own tests — never by majority vote. Three agents repeating the
   same hallucination is still a hallucination.

## Step 7 — Scoreboard and distilled report

Write `.jury/VERDICT.md`:

- **Scoreboard:** per agent — findings, confirmed, refuted, unverifiable,
  duplicates, unique confirmed catches, precision %. Every cell is
  recomputed from the findings table (findings = confirmed + refuted +
  unverifiable) — never estimated. Name the sharpest juror and the
  noisiest, with numbers.
- **Distilled defect report:** CONFIRMED findings only, worst-first
  (security > data loss > correctness > architecture), each with evidence
  and every agent that caught it.
- **Disagreements worth human eyes:** UNVERIFIABLE items where jurors
  split — with what would settle each.
- **Panel integrity:** jurors empaneled vs completed, per-juror failure
  reasons (timeout / auth / quota). Fewer than two completed reports
  (juror #0 included) stamps the whole verdict `DEGRADED`.
- **Handoff:** recommend deep-plan for the fix roadmap (Phase 0 = the
  confirmed security/correctness findings) — this skill audits; it does
  not fix.

Report cost honestly: which jurors ran, how long, which timed out or
errored. Clean up worktrees (`git worktree remove`); keep `.jury/reports/`
as the audit trail.

Before presenting, self-grade `.jury/VERDICT.md` against the **acceptance
checklist** at the end of
[references/verdict-protocol.md](references/verdict-protocol.md). If any
item fails, return to the step it names — do not present a failing
verdict.
