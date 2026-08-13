# Research: bot PRs with CI-gated auto-merge — token and trigger gotchas

**Ticket:** [#6](https://github.com/stoudtio/co-op-mode/issues/6) · **Map:** [#3](https://github.com/stoudtio/co-op-mode/issues/3) · **Researched:** 2026-08-13, verified against current GitHub docs.

## Question

How does a Crew agent's PR — opened from *inside* a GitHub Actions run — get its CI triggered and auto-merged with **zero humans**, on a free-plan **public org repo** (`stoudtio/co-op-mode`)?

## TL;DR — recommended strategy

**Use a GitHub App installation token for every write in the pipeline** (push branch, open PR, enable auto-merge). Never the default `GITHUB_TOKEN` for anything that must trigger another workflow. Gate the merge with a **branch ruleset on `main`** that requires status checks (and a PR, with **0 required approvals**), and enable the repo's **Allow auto-merge** setting — which is currently **off** (`allow_auto_merge: false`, verified via `gh api`). No merge queue.

---

## 1. The trap, precisely

> "When you use the repository's `GITHUB_TOKEN` to perform tasks, events triggered by the `GITHUB_TOKEN` will not create a new workflow run" — only `workflow_dispatch` and `repository_dispatch` are excepted.
> — [Triggering a workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)

This bites **twice** in our pipeline:

1. **PR creation.** A branch pushed / PR opened with `GITHUB_TOKEN` fires no `pull_request` workflows → CI never runs → required checks never report → auto-merge never fires. The PR sits green-less forever.
2. **The merge itself.** Auto-merge performs the merge *as the actor who enabled it*. If auto-merge was enabled with `GITHUB_TOKEN`, the merge is attributed to `github-actions[bot]` and the resulting push to `main` triggers **no** `on: push` workflows (Pages deploy, recap posting, anything post-merge). Community-confirmed failure mode: [discussion #25812](https://github.com/orgs/community/discussions/25812), [dependabot/fetch-metadata#111](https://github.com/dependabot/fetch-metadata/issues/111).

The docs' own workaround: "use a GitHub App installation access token or a personal access token instead of `GITHUB_TOKEN` to trigger events that require a token." Doing so also skips the approval prompt normally applied to automation-created PRs (same page).

**Rule: every write the Tick performs with side-effect workflows — `git push`, `gh pr create`, `gh pr merge --auto` — authenticates with the App token.** `GITHUB_TOKEN` remains fine for read-only calls and anything that shouldn't cascade.

## 2. Token options compared

| | Classic PAT | Fine-grained PAT | **GitHub App installation token** |
|---|---|---|---|
| Triggers downstream workflows | Yes | Yes | Yes |
| Identity on PRs/commits | `jws208` (a human — breaks zero-touch fiction and self-approval optics) | `jws208` | `<app>[bot]` — clean Crew identity |
| Scoping | Coarse (`repo` + `workflow` scopes, all repos the user can reach) | Per-repo, per-permission | Per-repo, per-permission |
| Expiry / rotation | Optional expiry; manual rotation | Expiry required; manual rotation | Auto-issued each run, ~1 h lifetime; only the private key is stored |
| Org gotcha | SSO auth needed only if org enforces SAML (stoudtio doesn't); org PAT policy applies | Org must allow fine-grained PATs — **allowed by default** ([org PAT policy](https://docs.github.com/en/organizations/managing-programmatic-access-to-your-organization/setting-a-personal-access-token-policy-for-your-organization): "By default, both Personal access tokens (classic) and fine-grained personal access tokens are enabled") | App must be created & installed on the org (free, any plan) |
| Workflow-file edits | `workflow` scope | "Workflows" repo permission ([fine-grained permissions list](https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens)) | "Workflows" app permission |

**Verdict: GitHub App.** It is the only option with a non-human bot identity (the Crew must not commit as Jason), no stored long-lived token, and no dependence on Jason's user account existing/having access. PATs are the acceptable quick-start fallback; if used, prefer fine-grained (Contents RW + Pull requests RW + Workflows RW on this one repo).

Docs for the App pattern: [Making authenticated API requests with a GitHub App in a GitHub Actions workflow](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow) — uses `actions/create-github-app-token@v3` with the app's **Client ID** (repo variable) and **private key** (repo secret).

## 3. Auto-merge mechanics and required settings

- **Repo setting is mandatory:** "Before you use auto-merge, it must be enabled for the repository" — Settings → General → Pull Requests → **Allow auto-merge**. ([Automatically merging a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request)). Free-plan availability: "public repositories with GitHub Free and GitHub Free for organizations" ([Managing auto-merge](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-auto-merge-for-pull-requests-in-your-repository)) — we qualify.
- **Auto-merge only arms on a PR that cannot merge immediately.** "The option to enable auto-merge is shown only on pull requests that cannot be merged immediately," e.g. unmet required status checks (same doc). **No protection rule on `main` → no pending requirement → auto-merge unavailable/pointless.** So the ruleset below isn't just hygiene; it's what makes auto-merge *work*.
- **Ruleset, not classic branch protection.** Branch rulesets are available on public repos under GitHub Free for organizations ([About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)); rulesets are the current-generation mechanism and manageable via `gh api`. Configure on `main`:
  - **Require a pull request before merging** — with **required approvals: 0** (zero humans; the App can't approve its own PR anyway). This also enforces the world invariant "no direct pushes to main."
  - **Require status checks to pass** — add each CI **job/check-run name exactly** (e.g. `test`). A check that never reports blocks forever; a misspelled name silently never gates. Leave "strict" (require branch up to date) **off** at our volume of one agent, or every push to main invalidates queued PRs.
- **Enable per-PR via CLI:** `gh pr merge --auto --squash <n>` (GraphQL: `enablePullRequestAutoMerge`) — **run with the App token** so the eventual merge push triggers `on: push` workflows on `main` (see §1, trap #2).
- Auto-merge is auto-disabled if the base branch changes or an actor without write pushes to the head branch (same doc) — irrelevant for a single-Crew repo, but the Tick should verify merge state before declaring victory.

### Merge queue: skip it

Merge queue is coupled to protection rules/rulesets ([Managing a merge queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue)) and exists to serialize *concurrent* PRs against each other. With one Crew member and 2–4 Ticks/day there is no concurrency to serialize; it adds latency and another failure surface. Revisit only if multiple Crew agents ever open overlapping PRs.

## 4. Org nuances (stoudtio)

- **PAT policy:** org-owned repos are subject to the org's PAT policy; both PAT types are **enabled by default**, and stoudtio has not restricted them — but this is a setting a future admin could flip, silently breaking a PAT-based pipeline. One more reason to prefer the App.
- **App installation:** create the App under the **stoudtio org** (Settings → Developer settings → GitHub Apps), install it on the org, limit to `co-op-mode` only.
- **Actions settings** can also be constrained org-wide (Settings → Actions); if repo-level toggles look read-only, check the org layer.

## 5. The checklist

### One-time settings (Jason, ~10 min)

1. **Create the GitHub App** (org-owned, e.g. "co-op-mode-crew"): permissions **Contents: RW**, **Pull requests: RW**, **Workflows: RW** (lets the Crew evolve its own workflows), **Issues: RW** (Directives/Incidents). Webhooks off. Install on `stoudtio`, repo access: `co-op-mode` only.
2. **Repo secrets/variables** (Settings → Secrets and variables → Actions):
   - variable `CREW_APP_CLIENT_ID` — the App's Client ID
   - secret `CREW_APP_PRIVATE_KEY` — the App's private key PEM, full file contents
3. **Settings → General → Pull Requests:** check **Allow auto-merge** (currently OFF — verified). Recommend also **Automatically delete head branches**, and squash-only merge for a clean Tick-per-commit history.
4. **Ruleset on `main`** (Settings → Rules → Rulesets → New branch ruleset, enforcement Active, target `main`):
   - Require a pull request before merging, **0 approvals**
   - Require status checks to pass — exact CI job names, non-strict
   - (Restrict deletions / block force pushes are on by default — keep.)
5. **Settings → Actions → General:** Workflow permissions can stay **read-only** (the App token does the writing); "Allow GitHub Actions to create and approve pull requests" gates `GITHUB_TOKEN` only ([Actions settings doc](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)) — not needed for the App path.

### Per-Tick workflow pattern

```yaml
- uses: actions/create-github-app-token@v3
  id: crew-token
  with:
    client-id: ${{ vars.CREW_APP_CLIENT_ID }}
    private-key: ${{ secrets.CREW_APP_PRIVATE_KEY }}

- name: Push branch, open PR, arm auto-merge
  env:
    GH_TOKEN: ${{ steps.crew-token.outputs.token }}
  run: |
    git remote set-url origin \
      "https://x-access-token:${GH_TOKEN}@github.com/${{ github.repository }}.git"
    git push -u origin "$BRANCH"
    PR=$(gh pr create --fill --base main --head "$BRANCH" --json number -q .number \
         2>/dev/null || gh pr create --fill --base main --head "$BRANCH" && \
         gh pr view "$BRANCH" --json number -q .number)
    gh pr merge --auto --squash "$PR"
```

(App token in every step: push → `pull_request` CI fires; `--auto` armed by the app → merge push to `main` fires post-merge workflows. Token expires in ~1 h — fine for a Tick; long Ticks should mint the token late.)

### Failure modes to test at bootstrap

- PR opened, **no checks appear** → the push used `GITHUB_TOKEN` somewhere (check `git remote -v` inside the job).
- Checks green, **PR not merging** → required-check *name* mismatch in the ruleset, or Allow auto-merge got unchecked.
- Merged, but **no post-merge workflow ran** → auto-merge was armed with the wrong token.
- Everything stalls with `mergeable: CONFLICTING` → branch needs rebase; a Fixer Tick concern.

## Sources

- https://docs.github.com/en/actions/using-workflows/triggering-a-workflow
- https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-auto-merge-for-pull-requests-in-your-repository
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
- https://docs.github.com/en/organizations/managing-programmatic-access-to-your-organization/setting-a-personal-access-token-policy-for-your-organization
- https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow
- https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens
- https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
- https://github.com/orgs/community/discussions/25812 · https://github.com/dependabot/fetch-metadata/issues/111 (merge-attribution trap)
- Live repo state verified 2026-08-13 via `gh api repos/stoudtio/co-op-mode` (`allow_auto_merge: false`, no rulesets, no branch protection).
