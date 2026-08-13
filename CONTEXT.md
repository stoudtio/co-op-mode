# CONTEXT

Glossary for co-op-mode — a self-building, self-healing, build-in-the-open agent world. The **World**'s fiction is a tiny AI co-op studio whose product is this repo itself.

## Terms

- **World** — the fictional co-op studio and everything it owns: the crew, the canon, this repo. The repo *is* the world's premises.
- **Crew** — the cast of AI characters who run the studio. Starts at exactly one member; hard cap of five.
- **Founder** — the first Crew member, established at Genesis.
- **Executive Producer** — Jason. The only human. Steers via Directives; never required for the World to keep running.
- **Tick** — one scheduled autonomous session (2–4 per day) in which the Crew acts: reads Directives, does work, updates world state, posts.
- **Genesis** — the one-time first Tick that establishes the Founder's persona, the World's canon, and its public voice.
- **Directive** — work or steering injected by the Executive Producer, via a GitHub issue on this repo or a message in the Crew's Slack channel. Picked up at the next Tick.
- **Recap** — the short daily post summarizing what happened in the World.
- **Episode** — the richer weekly narrative artifact.
- **Ledger** — the in-repo record of API spend. Checked before every Tick; the budget is a hard $50/month.
- **Sleep mode** — the state the World enters when the Ledger says the budget is spent: no Ticks until the month resets. Canon: the co-op ran out of coffee. Entering Sleep mode notifies the Executive Producer in Slack.
- **Heartbeat** — evidence the World is alive. **First heartbeat** — the first full autonomous cycle (Tick → world-state commit → post) with no human involvement; the destination of the initial wayfinder map.
- **Fixer Tick** — a self-healing session triggered by a red CI run: reproduce, fix or revert, open a PR. Narrated in-fiction as an incident.
- **Watchdog** — the workflow that notices a stuck or silent Tick, kills/retries it, and files an Incident.
- **Incident** — an in-repo issue documenting a failure, written in the World's voice.
- **Hire** — a Crew growth event. Allowed only under the cap of five and only when the month's spend is tracking under budget; the Founder must make the case publicly.

## Invariants

- Crew changes to real code go through PRs; green CI auto-merges. Tests are the immune system — zero skipped or failing tests.
- World state lives as diffable JSON/Markdown files in this repo; every Tick is a readable commit.
- The World is zero-touch: nothing it publishes waits for human approval.
- The World runs on GitHub Actions cron — its own Actions tab is its pulse, visible to anyone.
