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

| Agent | Findings | Confirmed | Refuted | Unverifiable | Duplicates | Unique confirmed | Precision |
|-------|---------:|----------:|--------:|-------------:|-----------:|-----------------:|----------:|

- **Accounting closes:** every claim carries exactly one verdict, so per
  agent **Findings = Confirmed + Refuted + Unverifiable** (a merged
  duplicate inherits the merged finding's verdict and is also counted in
  Duplicates). Recompute every cell from the findings table — a
  scoreboard number that cannot be traced back to table rows is
  fabricated.
- **Precision** = confirmed / (confirmed + refuted). The hallucination
  measure.
- **Unique confirmed** = confirmed findings no other juror caught. The
  sharpness measure — a low-volume juror with unique catches beats a
  noisy one with none.
- **Coverage:** footnote per agent which numbered scope items (S1…Sn
  from the brief) its report addressed — findings or an explicit
  "nothing found". SILENT items are named; skipping security is not
  passing security.
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
3. **Open questions** — every UNVERIFIABLE item, none dropped, each with
   the named verification step (exact command, credential, or runtime
   state) that would settle it.
4. **Scoreboard** — the table plus a two-line honest read of it.
5. **Panel integrity** — jurors empaneled vs completed, per-juror failure
   reasons (timeout / auth / quota / format deviation). Fewer than two
   completed reports (juror #0 included) stamps the verdict `DEGRADED`.
6. **Recommended next step** — hand the verified list to **deep-plan**
   (Phase 0 = confirmed security/correctness fixes). The jury audits; the
   planner plans; the implementer fixes.

## Acceptance checklist — pass before presenting

Grade `.jury/VERDICT.md` item by item, honestly. Any failure returns you
to the step it names; do not present a failing verdict.

- [ ] Every claim in the unified findings table carries exactly one
      verdict; CONFIRMED and REFUTED each quote `file:line` evidence.
      (Step 6)
- [ ] Scoreboard recomputed from the findings table: per agent,
      Findings = Confirmed + Refuted + Unverifiable, and Precision
      recalculated — no estimated cells. (Step 7)
- [ ] Coverage matrix present: per agent × numbered scope item, SILENT
      items named. (Step 5)
- [ ] Every distilled defect names its claimants and traces to the raw
      report file(s) in `.jury/reports/`. (Step 7)
- [ ] Every UNVERIFIABLE item appears under Open questions with the
      named step that would settle it. (Step 6)
- [ ] Panel integrity section present; `DEGRADED` stamp applied when
      fewer than two reports completed or consent was unavailable.
      (Steps 2, 7)
- [ ] Judge bias disclosed; juror #0's findings went through the
      identical verify step. (Step 6)
- [ ] All worktrees removed (crashed jurors force-removed and pruned);
      `.jury/reports/` preserved. (Steps 4, 7)

## Repeat runs

The jury is rerunnable: after fixes land, empanel again with the same
brief and compare — confirmed findings should disappear; anything that
survives two rounds with the same evidence is either unfixed or the fix
missed. Keep prior `.jury/` outputs (timestamped) as the audit trail.
