# Verdict Protocol

How testimony becomes a verdict: dedup → verify → score → distill.

## 1. Dedup

Two claims are the same finding when they name the same defect at the same
place (same file, overlapping lines, same failure mode) — wording differs,
substance matches. Merge them; credit every claimant. When unsure whether
two claims are one defect, keep them separate — false merges corrupt the
scoreboard more than false splits.

## 2. Verify — every unique finding, no exceptions

Open the cited file at the cited line and test the claim against the code:

- **CONFIRMED** — the defect is real. Quote the exact code that proves it.
  Where cheap and safe (read-only, seconds), strengthen with the repo's own
  tooling: run the existing test suite, a typecheck, or a one-off assertion
  in the juror worktree.
- **REFUTED** — the claim is wrong: the cited code doesn't say that, a
  guard the agent missed exists (cite it), or the "vulnerable" path is
  unreachable. **REFUTED requires evidence too** — refute claims the same
  way they had to be made, with `file:line`.
- **UNVERIFIABLE** — needs runtime state, credentials, or infrastructure
  you don't have. Labeled and quarantined; never silently promoted to the
  final report, never counted against the agent.

Anti-rules:

- **No majority vote.** Three agents repeating the same wrong claim is a
  shared hallucination, not a confirmation. Only code decides.
- **No authority weighting.** A finding from the "best" agent gets the
  same scrutiny as one from the weakest.
- **Judge bias check:** the orchestrator's own findings (juror #0) go
  through the identical verify step; when refuting a rival's claim that
  contradicts its own, the evidence bar is highest — quote both code
  locations.

## 3. Scoreboard

Per agent:

| Agent | Findings | Confirmed | Refuted | Duplicates | Unique confirmed | Precision |
|-------|---------:|----------:|--------:|-----------:|-----------------:|----------:|

- **Precision** = confirmed / (confirmed + refuted). The hallucination
  measure — the "ใครมั่ว" column.
- **Unique confirmed** = confirmed findings no other juror caught. The
  sharpness measure — a low-volume juror with unique catches beats a
  noisy one with none.
- Note format compliance and timeouts as footnotes — operational data for
  choosing next time's panel.

Read the scoreboard honestly: one run on one repo is a sample, not a
ranking of vendors. Say so in the report.

## 4. Distill

`.jury/VERDICT.md` final sections:

1. **Verified defects** — CONFIRMED only, worst-first (security > data
   loss > correctness > architecture > tests). Each: evidence quote,
   impact, claimants.
2. **Refuted claims** — each with the refuting evidence; this section is
   what makes the audit trustworthy (and teaches which agent patterns to
   distrust).
3. **Open questions** — UNVERIFIABLE items with what would settle each.
4. **Scoreboard** — the table plus a two-line honest read of it.
5. **Recommended next step** — hand the verified list to **deep-plan**
   (Phase 0 = confirmed security/correctness fixes). The jury audits; the
   planner plans; the implementer fixes.

## Repeat runs

The jury is rerunnable: after fixes land, empanel again with the same
brief and compare — confirmed findings should disappear; anything that
survives two rounds with the same evidence is either unfixed or the fix
missed. Keep prior `.jury/` outputs (timestamped) as the audit trail.
