# Research: how a Tick executes in GitHub Actions

**Ticket:** [#4](https://github.com/stoudtio/co-op-mode/issues/4) · **Map:** [#3](https://github.com/stoudtio/co-op-mode/issues/3)
**Question:** What runs inside the scheduled GitHub Actions job that powers one autonomous Tick — (a) `claude -p` headless Claude Code CLI, (b) the Claude Agent SDK (TypeScript/Python), or (c) raw Anthropic API calls from a script?

## Recommendation

**(a) Headless Claude Code CLI — `claude -p` — invoked directly from the workflow** (installed via `npm install -g @anthropic-ai/claude-code`, not wrapped in `anthropics/claude-code-action`).

Why this wins for a Tick:

1. **Tool use is the whole job.** A Tick reads Directives, edits world-state files, commits, and opens PRs. `claude -p` ships the full Claude Code harness — Bash, Read/Edit/Write, Glob/Grep — gated by `--allowedTools` permission rules (e.g. `Bash(git *)`, `Bash(gh *)`). Raw API calls would mean hand-building that agentic tool loop; the Agent SDK provides the identical harness but requires a runner script and a package install for zero additional capability at this stage. Per the [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview): to drive the agent loop from outside Python/TS, "run the CLI as a subprocess with the `-p` flag and `--output-format json`" — which is exactly what a workflow step is.
2. **Ledger capture is one `jq` away.** `claude -p --output-format json` returns `total_cost_usd`, cumulative `usage` (input/output/cache tokens), a per-model `modelUsage` breakdown, `session_id`, `num_turns`, and `is_error`/`subtype` ([headless docs](https://code.claude.com/docs/en/headless), [cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)). The `claude-code-action` wrapper buries results in the run log and doesn't surface the structured cost payload as a first-class output, which is why the direct CLI beats the action for us: the Ledger is the product of every run.
3. **Budget enforcement has a native lever.** `--max-budget-usd` (print-mode-only flag, [CLI reference](https://code.claude.com/docs/en/cli-reference)) hard-caps a single run's spend; the run ends with result `subtype: error_max_budget_usd` and still reports `total_cost_usd`. Combined with a pre-Tick Ledger gate, the $50/mo cap is enforced at two layers.
4. **Auth is exactly our constraint.** `claude -p` reads `ANTHROPIC_API_KEY` from the environment — pass the repo secret, done. (In `--bare` mode it *never* reads OAuth/keychain, so the API key path is the deterministic CI path.)
5. **Retryability is layered.** The CLI retries retryable API errors internally with backoff (emitting `system/api_retry` events); the process exits 0 on success / non-zero on failure so the job status is trustworthy; `timeout-minutes` + a `concurrency` group prevent overlap and runaways; the Watchdog (future ticket) kills/retries stuck runs. Each Tick is stateless-in, committed-state-out, so a rerun of a failed Tick is safe by construction.

**Escalation path:** if Tick orchestration outgrows a single prompt (inline budget arithmetic, structured multi-step outputs, custom tool gating), graduate to the **Agent SDK (TypeScript)** — it's the same harness with the same JSON result fields (`total_cost_usd`, `model_usage`, `max_budget_usd` option), so the Ledger schema survives the migration unchanged. Raw API calls are never the right layer for a Tick.

## Trade-off table

| Criterion | (a) `claude -p` headless CLI | (b) Claude Agent SDK (TS/Py) | (c) Raw Anthropic API script |
|---|---|---|---|
| **Auth via `ANTHROPIC_API_KEY` secret** | ✅ env var, zero config | ✅ env var | ✅ env var |
| **Tool use (git, gh, file edits)** | ✅ built-in tools + `--allowedTools` permission rules | ✅ identical harness, programmatic permission callbacks | ❌ must hand-build the tool loop (bash/edit tools, permission gating, retries) |
| **Per-run token/cost capture for Ledger** | ✅ `--output-format json` → `total_cost_usd`, `usage`, `modelUsage`, `session_id` (client-side estimate — see caveat) | ✅ same fields on `ResultMessage` | ⚠️ raw `usage` per response; must sum across loop turns yourself |
| **Install / cold-start cost** | ~30–60 s (`npm i -g @anthropic-ai/claude-code`; Node 20+ preinstalled on `ubuntu-latest`). Cacheable. | CLI cost **plus** `pip/npm install` of the SDK + a runner script | ~0 s (curl) to ~10 s (pip) — cheapest, but the savings buy nothing |
| **Determinism / retryability** | ✅ exit codes, internal API retry w/ backoff, `--max-turns`, `--max-budget-usd`, `timeout-minutes`, safe reruns | ✅ same, plus programmatic error handling | ⚠️ all retry/backoff/loop-bounding logic is ours to write and test |
| **Budget hard-stop per run** | ✅ `--max-budget-usd` | ✅ `max_budget_usd` option | ❌ manual |
| **Maintenance surface** | One workflow file + one prompt/skill | Workflow + runner script + dependency pinning | Workflow + a small agent framework we own forever |
| **Fit for 2–4 ticks/day, $50/mo, Haiku default** | ✅ `--model claude-haiku-4-5`; Haiku at $1/$5 per MTok puts a heavy Tick well under the ~$0.40/tick ceiling (120 ticks/mo ÷ $50) | ✅ same economics, more plumbing | ✅ cheapest tokens, most engineering risk |

**Why not `anthropics/claude-code-action@v1`?** It's the right tool for `@claude`-mention workflows and is built on the same SDK, and it *does* support `schedule` triggers in automation mode (results go to the run log). But it adds the GitHub App dependency, hides the JSON result payload we need for the Ledger, and its bot-actor check on scheduled runs adds a footgun (schedule events are attributed to the last user who touched the cron line — a bot actor there silently blocks runs). Direct CLI keeps the Tick's contract — JSON in the job, Ledger entry out — explicit.

## Key facts verified against primary sources (2026-08-13)

- **Headless mode** ([code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless)): `-p` runs non-interactively; exit 0/non-zero; `--output-format json` includes `total_cost_usd` and per-model cost; `--allowedTools` uses permission-rule syntax (`Bash(git diff *)` — space before `*` matters); `--bare` skips auto-discovery of CLAUDE.md/hooks/skills and never reads OAuth (recommended for reproducible CI — but a Tick *wants* the repo's CLAUDE.md and `.claude/skills` as the World's brain, and GitHub-hosted runners are ephemeral so there's no host-level `~/.claude` pollution; run non-bare); stdin capped at 10 MB; `system/api_retry` events surface internal retries.
- **Cost fields** ([cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)): `total_cost_usd` and `modelUsage.costUSD` are **client-side estimates** from a bundled price table — fine for the Ledger's gate with headroom, but the authoritative bill is the Usage & Cost API / Console. For subagent-inclusive accounting read `total_cost_usd`/`modelUsage`, not `usage`. Error results (including `error_max_budget_usd`) still carry cost fields; a hard crash may zero them — treat a crashed Tick as max-cost in the Ledger to stay conservative.
- **GitHub Actions** ([github-actions docs](https://code.claude.com/docs/en/github-actions)): scheduled workflows run only from the default branch; public repos disable schedules after 60 days without activity (a self-committing World resets this clock every Tick); commits made with the default `GITHUB_TOKEN` don't trigger other workflows — Tick-opened PRs need a GitHub App token or PAT if CI must run on them (matters for the green-CI-auto-merge invariant; flag for the bootstrap ticket).
- **Model/pricing** (claude-api skill, cached 2026-06): `claude-haiku-4-5` = $1/M input, $5/M output, 200K context. Prompt caching is automatic in the harness; for gaps > 5 min between ticks the cache will be cold anyway (ticks are hours apart), so no `ENABLE_PROMPT_CACHING_1H` needed.
- **Actions minutes:** the repo is public → GitHub-hosted runner minutes are free; the ~1 min of npm install + agent runtime costs $0 in minutes. Optional: cache the global npm install with `actions/cache` to shave cold-start.

## Minimal working workflow sketch

```yaml
# .github/workflows/tick.yml
name: Tick
on:
  schedule:
    - cron: "0 14 * * *"   # 2 ticks/day; add a second cron line to scale to 4
    - cron: "0 2 * * *"
  workflow_dispatch: {}     # manual Tick for testing / Genesis

concurrency:
  group: tick               # never two Ticks at once
  cancel-in-progress: false

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  tick:
    runs-on: ubuntu-latest
    timeout-minutes: 30     # Watchdog backstop
    steps:
      - uses: actions/checkout@v4

      - name: Ledger gate (Sleep mode check)
        id: gate
        run: |
          SPENT=$(jq -r '.month_to_date_usd // 0' state/ledger.json 2>/dev/null || echo 0)
          if awk "BEGIN{exit !($SPENT >= 50)}"; then
            echo "sleep=true" >> "$GITHUB_OUTPUT"   # out of coffee — notify Slack, no Tick
          fi

      - name: Install Claude Code
        if: steps.gate.outputs.sleep != 'true'
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Tick
        if: steps.gate.outputs.sleep != 'true'
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GH_TOKEN: ${{ github.token }}   # swap for App token when PRs must trigger CI
        run: |
          set +e
          claude -p "/tick" \
            --model claude-haiku-4-5 \
            --output-format json \
            --max-turns 50 \
            --max-budget-usd 0.40 \
            --allowedTools "Read,Edit,Write,Glob,Grep,Bash(git *),Bash(gh *)" \
            > tick-result.json
          echo "exit_code=$?" >> "$GITHUB_OUTPUT"
        id: tick

      - name: Append Ledger entry & commit world state
        if: steps.gate.outputs.sleep != 'true'
        run: |
          COST=$(jq -r '.total_cost_usd // 0' tick-result.json)
          jq --argjson c "$COST" \
             --arg sid "$(jq -r '.session_id // ""' tick-result.json)" \
             '.month_to_date_usd += $c | .entries += [{ts: now|todate, cost_usd: $c, session: $sid}]' \
             state/ledger.json > state/ledger.json.tmp && mv state/ledger.json.tmp state/ledger.json
          git config user.name "co-op-mode-tick" && git config user.email "tick@users.noreply.github.com"
          git add state/ && git commit -m "Tick: world state + ledger ($COST USD)" && git push
```

Notes on the sketch:

- `/tick` is a repo skill (`.claude/skills/tick/`) holding the Tick prompt — versioned World behavior, expanded by `claude -p` at run time.
- The Ledger append is done by the *workflow*, not the agent, from the JSON result — the spend record can't be forgotten or hallucinated. World-state commits by the agent itself happen inside the run via `Bash(git *)`; code changes go through PRs per the repo invariant.
- `--max-budget-usd 0.40` = $50 / ~120 ticks; tune once real per-tick costs land in the Ledger.
- Retry story: internal API retries are automatic; a failed job is visible in the Actions tab (the World's pulse) and is safe to re-run; the future Watchdog files an Incident on stuck/silent runs.

## Sources

- https://code.claude.com/docs/en/headless — headless mode, JSON output, `--bare`, exit codes, `api_retry`
- https://code.claude.com/docs/en/agent-sdk/overview — SDK vs CLI vs raw API decision table
- https://code.claude.com/docs/en/agent-sdk/cost-tracking — `total_cost_usd`, `usage`, `modelUsage`, estimate caveats, crash/error accounting
- https://code.claude.com/docs/en/github-actions — claude-code-action, schedule/automation mode, actor checks, cost guidance
- https://code.claude.com/docs/en/cli-reference — `--max-budget-usd`, `--max-turns`, `--allowedTools` flag confirmation
- Anthropic model pricing (claude-api skill reference, cached 2026-06): Haiku 4.5 $1/$5 per MTok
