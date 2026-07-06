# Audit Brief Template

One brief, every juror. Identical input is what makes the outputs
comparable and the scoreboard fair. Write the filled-in brief to
`.jury/brief.md` and pass it verbatim to every agent.

## Template

```markdown
# Independent Code Audit

You are one of several independent auditors examining this repository.
Work alone from the code in your working directory. Do not modify any
file — you are auditing, not fixing.

## Scope

<one of / combination, per the user's focus:>
- **Bugs:** logic errors, broken edge cases (null/empty/concurrent/
  oversized input), schema or contract mismatches between layers, dead or
  half-wired code paths.
- **Security:** injection (SQL/command/template), XSS paths from any
  user-controlled value to output, missing authentication or authorization
  on mutating/data-exposing endpoints, secrets in code or config, unsafe
  deserialization, path traversal.
- **Architecture:** layering violations, duplicated logic that has
  drifted, framework misuse (reimplementing what the framework provides),
  dependency tangles, dead abstractions.
- **Tests:** what critical paths have no coverage; tests that assert
  nothing; run the suite if a standard command exists and report failures.

## Rules

1. Every finding MUST cite `file:line`. No citation → don't report it.
2. Verify by reading the code before reporting — a suspicion is not a
   finding.
3. Report real defects, not style preferences.
4. If you find nothing in a category, say so explicitly.

## Required output format

For each finding:

### [SEVERITY] <one-line title>
- **Where:** `path/file.ext:line`
- **Category:** bug | security | architecture | tests
- **Claim:** what is wrong, in 2–3 sentences
- **Evidence:** the code behavior that proves it (quote the relevant line(s))
- **Impact:** what happens in production because of this

Severity scale: CRITICAL (exploit/data loss) > HIGH (correctness broken) >
MEDIUM (reliability/maintainability) > LOW (minor).

End with: `## Summary` — finding counts per category and your top 3 by
severity.
```

## Scoping notes for the orchestrator

- Fill the Scope section per the user's focus argument; drop unused areas
  to keep jurors concentrated.
- For large repos, add a path scope ("audit only `src/checkout/` and
  `src/api/`") — same paths for every juror.
- Do NOT include hints about suspected bugs — priming the jury
  contaminates the comparison. If the user has suspicions, hold them back
  and check them against the verdict afterward.
