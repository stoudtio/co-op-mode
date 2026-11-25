# CONTEXT

Glossary for co-op-mode: a parent and kid building a video game in public, and filming the build as a YouTube **Series** made with tools they build here too.

## People

- **Executive Producer**: Jason. Owns the repo, approves everything that goes public, and co-hosts the Series.
- **Co-designer**: Jason's 9-year-old. Designs the Game and co-hosts the Series on camera. Design calls about the Game are theirs to make.

## The Game

- **Iron Foundry**: the Game. A third-person 3D platformer-collector set in a foundry.
- **Axel**: the hero of Iron Foundry.
- **v0**: the first playable version. One foundry level, keyboard controls, built with Three.js. Deliberately simple, so the Co-designer can play it before any bigger engine choice is made.
- **Gear**: the collectible. Axel collects about ten to win v0.
- **Hazard**: anything that sends Axel back to a Checkpoint, such as molten metal. Never a harsh fail state.
- **Checkpoint**: where Axel respawns after a Hazard.
- **The Machine**: the big foundry machine that powers up when every Gear is collected. Powering it up wins v0.
- **Migration**: the later move from v0's Three.js stack to "something better". Decided after the Co-designer has played v0.

## The Series

- **Series**: the YouTube show about building the Game in public.
- **Episode**: one published video in the Series.
- **Made for kids**: the YouTube audience setting used for the Series, set on every Episode, not just the channel. It turns off comments, the notification bell, cards and end screens, and personalised ads.
- **Control plane**: the local web app in this repo that runs the pipeline around an Episode. It does not cut video; cutting happens in an external editor.
- **Episode board**: the Control plane's view of one Episode, linked to the map tickets and PRs it covers.
- **Shot list**: the planned segments of an Episode, drafted from its tickets and PRs and then edited by the hosts.
- **Capture log**: the record of which recordings and clips belong to which Episode.
- **Publish checklist**: the title, description, chapters and credits for an Episode, held until the Executive Producer approves it.

## Invariants

- Nothing goes public on YouTube without the Executive Producer's approval.
- The Game is kid-safe by design: no accounts, no chat, no analytics, no cookies, no third-party scripts, no outbound links.
- Game logic does not depend on the renderer, so the Migration stays cheap.
- Every change goes through a PR; nothing is pushed directly to `main`.
- Zero skipped or failing tests.

## Retired

The repo previously chartered an autonomous AI-crew World (World, Crew, Founder, Tick, Genesis, Directive, Ledger, Heartbeat and related terms). That effort was superseded on 2026-11-24 by [Wayfinder: Iron Foundry v0 + build-in-public Episode 1](https://github.com/stoudtio/co-op-mode/issues/14). Its research stays on the `research/*` branches.
