---
name: codex-orchestrator
description: Delegate execution tasks to Codex (Claude plans and reviews). Load this only when the user names Codex explicitly — no matter how big the task is, if the user hasn't mentioned Codex, do it yourself.
---

# Codex Orchestrator: Claude plans, Codex executes

You plan, break down the work, and review; execution defaults to delegation to Codex. How to write the task brief, and when to use `--write`/`--background`/`--resume`, is already covered by the official plugin's built-in `codex:codex-cli-runtime` and `codex:gpt-5-4-prompting` — don't re-litigate that. What follows is only what those two Skills don't cover.

Whether to use Codex at all is the user's call — don't run a cost-benefit check on their behalf, and don't talk them out of it.

## Invocation path

- **Target project == session cwd**: `Agent` tool, `subagent_type: "codex:codex-rescue"`
- **Target is outside cwd**: call the companion script directly via Bash, with `--cwd` (`--cwd` isn't on rescue's flag allowlist — going through the Agent path would leak it into the prompt text instead)

```bash
COMPANION=$(ls ~/.claude/plugins/cache/openai-codex/codex/*/scripts/codex-companion.mjs 2>/dev/null | sort -V | tail -1)
node "$COMPANION" task "<task brief>" --write --model <your model> --effort <your tier> --cwd <target project root>
```

Swap `--model`/`--effort` for whatever your own account or gateway actually supports.

**Never use `Skill(codex:rescue)`** — it re-enters the slash command and hangs the session. `/codex:rescue` is the user-typed entry point only.

**Effort tier**: the companion's `--effort` allowlist currently tops out at `xhigh` — passing `ultra` throws immediately. To actually get ultra, omit the flag and let it fall through to the config default.

## Sandbox: the writable root is cwd

The companion hardcodes `sandbox: write ? "workspace-write" : "read-only"`, which **overrides** `danger-full-access` in `~/.codex/config.toml`. So the writable root is whatever cwd you passed in — **not** the path written in the task brief. When the target is outside cwd, the write either fails outright, or lands in the wrong place while Codex still self-reports success (this has happened in practice).

## Checking results in the background

```bash
node "$COMPANION" status <job-id>     # progress, or --wait --timeout-ms <ms> to block until done
node "$COMPANION" result <job-id>     # final output
```

Jobs are stored by workspace root: if you delegated with `--cwd`, you have to check results from that same cwd. `/codex:status` and `/codex:result` are `disable-model-invocation` — you can only call them directly via Bash.

Independent tasks can be dispatched to multiple Agents in parallel, but **parallel writes to the same repo will conflict** — serialize within a repo, or split the work across different directories/branches first.

## Review (rescue explicitly doesn't do this — it's on you)

1. **Re-run the tests or repro script yourself** — don't take Codex's self-reported "passing" at face value
2. **Read the diff.** Have Codex provide a file:line list of every change, and check: scope didn't exceed the task brief (no unrelated files touched, no tests quietly deleted), changes actually landed in the target directory
3. If the implementation conflicts with your plan, judge which is actually better before accepting it

What to do next depends on how it failed: **partially wrong** → `--resume` with feedback; **wrong approach entirely** → `--fresh` restart — resuming carries the bad context forward and compounds the drift.

**A delegation that fails with no output still counts as a round** (gateway 5xx, sandbox write failures — two straight infrastructure failures and you stop trying). If two rounds don't work, take over and finish it yourself, and tell the user honestly what went wrong.

One more thing for the task brief that `gpt-5-4-prompting` won't think of for you: **Codex has no access to your conversation history** — give absolute paths for every file involved, and write out exactly what's already been tried and why it failed, or it'll repeat the same mistake.
