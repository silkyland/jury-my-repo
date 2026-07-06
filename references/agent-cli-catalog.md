# Agent CLI Catalog

Known AI coding agent CLIs and their headless (non-interactive) invocation.

> **Drift warning:** flags change and products retire. This catalog gets
> you to the right neighborhood; ALWAYS confirm against `<cli> --help`
> during the run and prefer what the binary says over what this file says.
> Record the version you actually invoked.

## Detection sweep

```bash
for cli in claude codex gemini opencode kimi cursor-agent copilot qwen \
           aider goose crush droid amp auggie; do
  command -v "$cli" >/dev/null 2>&1 && echo "FOUND: $cli $($cli --version 2>&1 | head -1)"
done
```

## The catalog

| CLI | Vendor | Headless invocation (verify via --help) | Read-only / sandbox notes |
|-----|--------|------------------------------------------|---------------------------|
| `claude` | Anthropic (Claude Code) | `claude -p "<prompt>" --output-format text` | Restrict with `--allowedTools "Read,Grep,Glob,Bash(git log:*)"` |
| `codex` | OpenAI (Codex CLI) | `codex exec "<prompt>"` | Has sandbox modes — use the read-only sandbox flag shown in `codex exec --help` |
| `gemini` | Google (Gemini CLI) | `gemini -p "<prompt>"` | Retiring mid-2026 for a successor — expect breakage; worktree isolation is the guard |
| `opencode` | OpenCode (OSS) | `opencode run "<prompt>"` | Worktree isolation is the guard |
| `kimi` | Moonshot (Kimi Code CLI) | `kimi --print -p "<prompt>"` (or `--quiet` for final-message-only) | `--print` implies auto-approve (`--yolo`) — worktree isolation REQUIRED |
| `cursor-agent` | Cursor | `cursor-agent -p "<prompt>"` | Check `--help` for output format flags |
| `copilot` | GitHub (Copilot CLI) | `copilot -p "<prompt>"` | Tool-permission flags vary by version — check `--help` |
| `qwen` | Alibaba (Qwen Code) | `qwen -p "<prompt>"` | Gemini CLI fork — same flag family |
| `aider` | Aider (OSS) | `aider --message "<prompt>" --no-auto-commits --yes` | Add `--no-git` or rely on worktree; aider WILL edit by default |
| `goose` | Block (OSS) | `goose run -t "<prompt>"` | Worktree isolation is the guard |
| `crush` | Charmbracelet (OSS) | `crush run "<prompt>"` | Worktree isolation is the guard |
| `droid` | Factory | `droid exec "<prompt>"` | Check `--help` for auto/permission levels |
| `amp` | Sourcegraph | `amp -x "<prompt>"` (or prompt via stdin) | Check `--help` |
| `auggie` | Augment | `auggie -p "<prompt>"` | Check `--help` |

**Not empanelable by default:** Devin (cloud service, API-based — no
standard local CLI; offer as optional API integration only if the user has
credentials and asks). IDE-bound assistants (Windsurf, JetBrains AI)
without a headless CLI are out of scope.

## Invocation rules

- **Working directory = the juror's own worktree**, never the real repo.
  This is the universal mutation guard; per-CLI sandbox flags are
  defense-in-depth on top, not a substitute.
- Capture stdout AND stderr to `.jury/reports/<agent>.md` /
  `<agent>.stderr.log`; exit code recorded.
- Timebox every run (default 15 min; `timeout` or background + kill).
- Auth check first: run each juror once with a trivial prompt
  (`"Reply OK"`) with a short timeout — a login prompt or auth error
  disqualifies the juror NOW instead of stalling the panel.
- Prompt delivery: prefer the brief inline (`"$(cat .jury/brief.md)"`);
  some CLIs also accept stdin — `--help` decides.
- One juror per vendor account rate-limit domain: if two CLIs share one
  subscription (e.g. two tools on the same API key), warn the user in the
  consent gate.
