# Research: measuring per-Tick spend and enforcing the $50/month Ledger

> Resolves [#5](https://github.com/stoudtio/co-op-mode/issues/5). Part of the wayfinder map [#3](https://github.com/stoudtio/co-op-mode/issues/3).
> Researched 2026-08-13. Pricing verified live against platform.claude.com/docs/en/about-claude/pricing on 2026-08-13 (cross-checked with the claude-api skill reference) — see Sources.

## Question

How does the World know what each Tick actually cost, and how is the hard $50/month cap enforced in code?

## TL;DR

1. **Measurement:** run each Tick via Claude Code headless (`claude -p --output-format json`). The result JSON carries `total_cost_usd` — a ready-made dollar figure — plus full token `usage` (input / output / cache-read / cache-write) and per-model breakdown. No client-side price math is needed on the happy path; the pricing table below exists as a cross-check and fallback.
2. **Ledger:** a diffable `ledger.json` at the repo root, updated and committed as part of every Tick's world-state commit.
3. **Enforcement:** a pre-Tick guard (pure `jq`, zero API cost) reads the Ledger before any model call. If `spent + reserve ≥ budget`, the Tick is skipped, the World enters **Sleep mode** ("the co-op ran out of coffee"), and Jason gets a Slack `@`-mention. Month rollover archives the old ledger and wakes the World.
4. **Backstop:** Anthropic Console **workspace spend limits** cap the API key provider-side, so even a bug in the Ledger cannot overspend (relates to #12).

---

## 1. Token-usage fields available

### Anthropic Messages API (`response.usage`)

Every API response reports four token counters:

| Field | Meaning | Billing rate |
|---|---|---|
| `input_tokens` | Uncached input tokens | full input price |
| `output_tokens` | Generated tokens (incl. thinking) | output price |
| `cache_read_input_tokens` | Input served from prompt cache | ~0.1× input price |
| `cache_creation_input_tokens` | Input written to cache | 1.25× input price (5m TTL) / 2× (1h TTL) |

Total prompt size = `input_tokens + cache_creation_input_tokens + cache_read_input_tokens` — `input_tokens` alone is only the uncached remainder.

### Claude Code headless output (`claude -p "..." --output-format json`)

The final `result` JSON object includes (verified against code.claude.com headless docs):

```json
{
  "type": "result",
  "subtype": "success",
  "is_error": false,
  "total_cost_usd": 0.4213,
  "duration_ms": 184520,
  "num_turns": 23,
  "session_id": "abc-123",
  "usage": {
    "input_tokens": 3571,
    "output_tokens": 12727,
    "cache_read_input_tokens": 466656,
    "cache_creation_input_tokens": 21460
  },
  "modelUsage": {
    "claude-sonnet-4-6": { "inputTokens": ..., "outputTokens": ..., "cacheReadInputTokens": ..., "cacheCreationInputTokens": ..., "costUSD": ... }
  },
  "result": "…final assistant text…"
}
```

Verified caveats from the Agent SDK cost-tracking docs (code.claude.com/docs/en/agent-sdk/cost-tracking):

- **`total_cost_usd` and `modelUsage[].costUSD` are client-side estimates**, computed from a price table bundled into the CLI — they can drift from the actual bill when pricing changes or the CLI doesn't recognize a model. Fine for the Ledger (we enforce with headroom); authoritative reconciliation is the Console Usage page / Usage & Cost API.
- **`total_cost_usd` and `modelUsage` include subagent spend; the top-level `usage` object does not.** Since Ticks will fan out to Haiku subagents, the Ledger must never be computed from `usage` alone.
- **On a crash (`subtype: "error_during_execution"`) cost fields may be zeroed** — which is exactly why the Watchdog writes a synthetic worst-case entry (§3) instead of trusting a zero.

**Decision: `total_cost_usd` is the Ledger's source of truth.** It already accounts for cache-tier pricing and mixed-model sessions. The pricing table is used only to (a) sanity-check the figure and (b) estimate a per-Tick cost cap for the guard.

## 2. Current pricing (per million tokens)

Verified live 2026-08-13 at platform.claude.com/docs/en/about-claude/pricing:

| Model | Input | Output | Cache write (5m) | Cache write (1h) | Cache read |
|---|---|---|---|---|---|
| Claude Haiku 4.5 (`claude-haiku-4-5`) | $1.00 | $5.00 | $1.25 | $2.00 | $0.10 |
| Claude Sonnet 4.6 (`claude-sonnet-4-6`) | $3.00 | $15.00 | $3.75 | $6.00 | $0.30 |
| Claude Sonnet 5 (`claude-sonnet-5`) | $2.00 | $10.00 | $2.50 | $4.00 | $0.20 |

**Note on Sonnet 5:** the $2/$10 launch pricing (originally "introductory through 2026-08-31") is now the **standard** price — the docs state the scheduled Sept 1, 2026 increase to $3/$15 *will not occur*. Sonnet 5 is currently cheaper than Sonnet 4.6.

Cache multipliers (all models): write 1.25× input (5m TTL) / 2× (1h TTL); read 0.1× input. Batch API is 50% off (not applicable to interactive Ticks). Web search, if the Tick uses it, is $10 per 1,000 searches on top of tokens.

**Budget intuition:** a heavy Tick moving ~500K cache-read + ~20K uncached-in + ~15K out costs ≈ $0.44 on Sonnet 4.6 and ≈ $0.29 on Sonnet 5. At 3 Ticks/day × 31 days ≈ 93 Ticks/month, the mean Tick must stay under ~$0.50 to fit $50/mo with headroom — comfortably realistic if the system prompt caches well and reading fans out to Haiku subagents (and Sonnet 5's pricing makes it the obvious default Tick model).

## 3. The Ledger: in-repo format

**Location:** `ledger.json` at repo root (world state is diffable JSON per CONTEXT.md invariants). Monthly archives go to `ledger/archive/YYYY-MM.json`.

```json
{
  "schema": 1,
  "month": "2026-08",
  "budget_usd": 50.0,
  "reserve_usd": 2.0,
  "spent_usd": 12.34,
  "status": "awake",
  "ticks": [
    {
      "ts": "2026-08-13T14:02:11Z",
      "kind": "tick",
      "cost_usd": 0.42,
      "num_turns": 23,
      "duration_ms": 184520,
      "usage": {
        "input_tokens": 3571,
        "output_tokens": 12727,
        "cache_read_input_tokens": 466656,
        "cache_creation_input_tokens": 21460
      },
      "session_id": "abc-123",
      "run_id": "17234567890",
      "commit": "filled-by-post-tick-commit"
    }
  ]
}
```

Field notes:

- `reserve_usd` — safety margin so the *last* Tick before the cap can't overshoot $50. Set to ≥ the observed p99 Tick cost (start at $2, tune from data).
- `kind` — `tick` | `genesis` | `fixer` | `watchdog`, so Incident/Fixer spend is attributable.
- `spent_usd` — denormalized sum of `ticks[].cost_usd`; the update script recomputes it from the array every time (the array is the truth, the scalar is a cache).
- `status` — `awake` | `sleeping`. Sleep mode is *stored in the Ledger itself*, so it survives across runs and is visible to anyone reading the repo.

### Update flow (post-Tick)

```bash
RESULT=$(claude -p "$TICK_PROMPT" --output-format json ...)
COST=$(echo "$RESULT" | jq '.total_cost_usd')
jq --argjson entry "$(build_entry "$RESULT")" \
   '.ticks += [$entry] | .spent_usd = ([.ticks[].cost_usd] | add | . * 100 | round / 100)' \
   ledger.json > ledger.tmp && mv ledger.tmp ledger.json
git add ledger.json && git commit -m "tick: $(date -u +%FT%TZ) cost \$${COST}"
```

The Ledger commit rides in the same push as the world-state commit, so every Tick is one readable commit containing both what happened and what it cost. Concurrency: the tick workflow uses a GitHub Actions `concurrency` group (`group: tick, cancel-in-progress: false`) so two crons can never race on `ledger.json`.

**Failure case:** if the headless run dies before emitting JSON, the Watchdog files an Incident and appends a conservative synthetic entry at the per-Tick cap (see below) — unmeasured spend is assumed worst-case, never zero.

## 4. Pre-Tick budget check (the guard)

The guard runs **before any model call** — it's pure `jq` on a checked-out file and costs $0, so Sleep-mode crons stay effectively free.

```bash
#!/usr/bin/env bash
# scripts/budget-guard.sh — exit 0 = proceed, exit 78 = sleep
set -euo pipefail

LEDGER=ledger.json
NOW_MONTH=$(date -u +%Y-%m)
MONTH=$(jq -r .month "$LEDGER")

# ---- month rollover ----
if [[ "$MONTH" != "$NOW_MONTH" ]]; then
  mkdir -p ledger/archive
  cp "$LEDGER" "ledger/archive/${MONTH}.json"
  jq --arg m "$NOW_MONTH" \
     '.month=$m | .spent_usd=0 | .ticks=[] | .status="awake"' \
     "$LEDGER" > ledger.tmp && mv ledger.tmp "$LEDGER"
  git add ledger.json ledger/archive
  git commit -m "ledger: roll over to $NOW_MONTH — the World wakes up"
  git push
  # if we were sleeping, announce the wake-up in Slack (fresh coffee delivery)
fi

# ---- budget check ----
SPENT=$(jq -r .spent_usd "$LEDGER")
BUDGET=$(jq -r .budget_usd "$LEDGER")
RESERVE=$(jq -r .reserve_usd "$LEDGER")

if (( $(echo "$SPENT + $RESERVE >= $BUDGET" | bc -l) )); then
  # ---- Sleep mode ----
  if [[ $(jq -r .status "$LEDGER") != "sleeping" ]]; then
    jq '.status="sleeping"' "$LEDGER" > ledger.tmp && mv ledger.tmp "$LEDGER"
    git add ledger.json && git commit -m "ledger: out of coffee — entering Sleep mode" && git push
    curl -sf -X POST "$SLACK_WEBHOOK_URL" -H 'content-type: application/json' -d "$(jq -n \
      --arg spent "$SPENT" --arg budget "$BUDGET" \
      '{text: ("☕ <@'"$JASON_SLACK_MEMBER_ID"'> The co-op has run out of coffee. Spent $" + $spent + " of $" + $budget + " this month. Sleeping until the 1st (or a budget top-up)." )}')"
  fi
  echo "budget exhausted — skipping tick"
  exit 78   # distinct code: workflow marks the run neutral/skipped, not failed
fi
exit 0
```

Details that matter:

- **`@Jason` must be a member-ID mention** (`<@U…>`), not the literal text `@Jason` — plain text does not trigger a Slack notification. `JASON_SLACK_MEMBER_ID` is a repo secret/variable.
- **The Slack ping fires once** (guarded by the `status != sleeping` check), not on every skipped cron.
- **Exit 78** lets the workflow distinguish "slept" from "failed" so the Watchdog doesn't treat Sleep mode as a red run. In the workflow: `if: steps.guard.outcome == 'success'` gates the Tick step; a `78` maps to a skipped tick with a green run.
- **Per-Tick cap as a second layer:** the tick invocation itself uses `--max-turns` (and a `task_budget`-style prompt instruction) so a single runaway Tick can't blow past `reserve_usd`. If a Tick's measured cost exceeds `reserve_usd`, raise the reserve.

### Sequence

```
cron fires
  └─ checkout repo
  └─ budget-guard.sh
       ├─ month changed?  → archive + reset + wake + commit
       ├─ spent+reserve < budget?  → run Tick
       │     └─ claude -p … --output-format json
       │     └─ append ledger entry, recompute spent, commit with world state
       └─ else → set sleeping (once: commit + Slack @Jason) → exit 78 (skip)
```

## 5. Provider-side backstop: Console workspace spend limits (→ #12)

Verified against the Anthropic support docs ("Creating and managing workspaces"): the Console **does** support per-workspace spend limits. On a workspace's details page → **Limits** tab, you can set a spend limit for that workspace (it must be lower than the org's limit), add email notifications at chosen spend thresholds, and Anthropic "always evaluates all applicable limiters — at the Workspace and Organization level — for every request." The support article doesn't explicitly use the words "hard cap," but per-request evaluation of the limiter means requests stop being served once the limit is hit — treat it as a circuit breaker, and verify the exact failure mode (expect 400/`billing_error`-class responses) during bootstrap.

Recommended configuration:

- Dedicated workspace `co-op-mode` containing only the World's API key.
- Workspace spend limit: **$55/month** (slightly above the Ledger's $50 so the in-repo cap always trips first and the World gets to narrate its own Sleep; the Console limit is the never-exceed circuit breaker).
- Notification thresholds at $25 and $45 as an independent early-warning channel to Jason's email.
- The Ledger remains the *primary* mechanism because (a) it's in-fiction and diffable, and (b) a Console hard-stop mid-Tick produces an ugly failed run instead of a graceful Sleep.

This is the concrete recommendation to carry into ticket #12.

## 6. Recommended decisions (summary)

| Decision | Choice |
|---|---|
| Cost source of truth | `total_cost_usd` from Claude Code headless JSON |
| Ledger location | `ledger.json` (repo root), archives in `ledger/archive/YYYY-MM.json` |
| Enforcement point | pre-Tick `jq` guard in the workflow, $0 cost, exit 78 = sleep |
| Overshoot protection | `reserve_usd` headroom + `--max-turns` per-Tick cap + Watchdog synthetic worst-case entry on crashed runs |
| Month rollover | guard compares `month` to UTC month; archive → reset → wake commit |
| Sleep escalation | one-time Slack webhook post with `<@member-id>` mention of Jason |
| Provider backstop | Console workspace spend limit at $55 on a dedicated workspace (#12) |

## Sources

- https://platform.claude.com/docs/en/about-claude/pricing — per-MTok pricing, cache multipliers, web-search pricing, Sonnet 5 standard-price note (fetched 2026-08-13).
- https://code.claude.com/docs/en/headless — `claude -p --output-format json` output: `total_cost_usd`, per-model cost breakdown, `session_id`, `result` (fetched 2026-08-13).
- https://code.claude.com/docs/en/agent-sdk/cost-tracking — `modelUsage`/`costUSD` field names, estimate caveat, subagent-inclusion semantics, crash-zeroing behavior (fetched 2026-08-13).
- https://support.claude.com/en/articles/9796807-creating-and-managing-workspaces — workspace Limits tab, spend limits, notifications, per-request limiter evaluation (fetched 2026-08-13).
- claude-api skill reference (bundled pricing tables) — cross-check; `usage` field semantics on the Messages API.
- `CONTEXT.md` — Ledger, Sleep mode, Tick, Watchdog definitions; world-state-as-diffable-JSON invariant.
