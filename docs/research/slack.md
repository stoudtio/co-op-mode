# Research: Slack posting and Directive-reading from Actions

Resolves [#7](https://github.com/stoudtio/co-op-mode/issues/7). Part of map [#3](https://github.com/stoudtio/co-op-mode/issues/3).

## TL;DR recommendation

Use **one dedicated internal Slack app with a bot token (`xoxb-…`)** for both posting and reading — skip incoming webhooks entirely. Post via `chat.postMessage`, @-mention Jason with the `<@USERID>` syntax (never display names), read Directives via `conversations.history` bounded by a **timestamp cursor committed to repo state**, and add an emoji **reaction as the visible "consumed" acknowledgment**. Create the app from a manifest (sketch below); store the token as a GitHub Actions secret. Jason already has Slack env vars locally, but the World should get its **own app + token** — GitHub Actions has no access to the local Claude Code Slack plugin/MCP.

---

## 1. Posting: webhook vs bot token

### Incoming webhook — rejected

Per [Sending messages using incoming webhooks](https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks):

- Bound to a **single channel chosen at install**; "You cannot override the default channel … when you're using incoming webhooks to post messages."
- **Cannot read messages** or retrieve anything from a channel — a webhook can never fetch Directives.
- Cannot delete/update after posting; requires the `incoming-webhook` scope on an app anyway.

Since the World must *read* the channel regardless, a webhook adds a second credential for a strict subset of what the bot token already does. No reason to carry both.

### Bot token + `chat.postMessage` — recommended

Per [`chat.postMessage`](https://docs.slack.dev/reference/methods/chat.postMessage):

- Scope: **`chat:write`** (bot token).
- Args: `channel` (use the channel **ID**, `C…`), `text` (required unless `blocks`/`attachments`), optional `blocks` for Block Kit.
- The bot must be a **member of the channel** to post (or hold `chat:write.public` for any public channel — unnecessary here; just invite the bot once).
- Rate limit: ~1 message/second per channel — irrelevant at 2–4 Ticks/day.
- `mrkdwn` parsing is on by default.

```bash
curl -s -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d "{\"channel\":\"$SLACK_CHANNEL_ID\",\"text\":\"Tick complete. <@$SLACK_JASON_USER_ID> — budget at 80%.\"}"
```

### @-mentioning Jason reliably

Per [Formatting message text](https://docs.slack.dev/messaging/formatting-message-text):

- Use the **user-ID mention syntax `<@U012AB3CD>`** in `text`. "If the mention is included in an app-published message, the mentioned user will also be notified about the reference."
- Writing `@Jason` / a display name as plain text does **not** link or notify.
- `link_names=1` can auto-parse names, but Slack itself recommends the manual ID form — IDs are stable, display names drift. **Pin Jason's user ID in config** (it already exists locally as `SLACK_JASON_USER_ID`; find it in Slack via profile → ⋮ → "Copy member ID", or `users.list` with `users:read`).

## 2. Reading: a Tick fetches Directives

### Scopes

| Scope | Why |
|---|---|
| `chat:write` | Post Recaps, escalations, Sleep-mode notice |
| `channels:history` | Read messages in the public Crew channel (`conversations.history`) |
| `channels:read` | Resolve channel name → ID / list channels (optional if channel ID is pinned in config — recommended) |
| `channels:join` | Bot can self-join public channels ([`conversations.join`](https://docs.slack.dev/reference/methods/conversations.join), public channels only) — optional; a one-time `/invite @bot` also works |
| `reactions:write` | Mark Directives consumed with an emoji reaction |
| `users:read` | Optional, only to look up user IDs once |

If the Crew channel were ever made **private**, swap in `groups:history` (and the bot must be invited; `conversations.join` won't work there).

### `conversations.history` mechanics

Per [`conversations.history`](https://docs.slack.dev/reference/methods/conversations.history):

- The bot must be a **member** of the conversation.
- Args: `channel` (ID), `oldest`/`latest` (Unix `ts` bounds, **exclusive by default** — pass `inclusive=true` only if you want the boundary message), `limit` (default 100, max 999), `cursor` for pagination.
- Paginate by following `response_metadata.next_cursor` until empty. At this World's volume one page will essentially always suffice.
- Every message carries a `ts` (e.g. `1723561414.000200`) that is **unique per channel** — usable both as the message ID for reactions and as the cursor value.
- Rate limits: Tier 3 (50+/min). The May 2025 crackdown (1 req/min, 15 results) applies only to **commercially distributed non-Marketplace apps** — "Any internal customer-built apps will maintain their existing rate limits" ([changelog clarification, 2025-06-03](https://docs.slack.dev/changelog/2025/06/03/rate-limits-clarity/); [original announcement, 2025-05-29](https://docs.slack.dev/changelog/2025/05/29/rate-limit-changes-for-non-marketplace-apps/)). Our internal app is exempt; even the reduced limits would comfortably serve 2–4 Ticks/day.

```bash
curl -s -G https://slack.com/api/conversations.history \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  --data-urlencode "channel=$SLACK_CHANNEL_ID" \
  --data-urlencode "oldest=$LAST_CONSUMED_TS" \
  --data-urlencode "limit=200"
```

Filter the result to human messages (drop `bot_id`/`subtype` entries, and the World's own posts) before treating anything as a Directive.

### Marking Directives consumed: cursor + reaction

Two mechanisms, each doing the job the other is bad at:

1. **Timestamp cursor in repo state (source of truth).** The Tick stores the highest `ts` it processed in a world-state file (e.g. `state/slack-cursor.json`), committed in the same world-state commit as the Tick's work. Next Tick passes it as `oldest` (exclusive by default — no `inclusive` needed, no double-reads). This is atomic with the Tick's own record: if the Tick fails before committing, the cursor doesn't advance and the Directives are re-read — at-least-once semantics, which is what you want. It also fits the invariant that world state lives as diffable files in the repo.
2. **Emoji reaction (visible acknowledgment).** After processing, the Tick calls [`reactions.add`](https://docs.slack.dev/reference/methods/reactions.add) (`reactions:write`; args `channel`, `timestamp`, `name`, e.g. `white_check_mark`) on each consumed Directive so Jason can *see* pickup in Slack. Treat `already_reacted` as success — it's exactly the re-run case.

Don't use reactions as the source of truth: deciding "consumed?" from reactions requires inspecting every message's `reactions` array on every Tick, and a reaction can land while the Tick's actual work failed. The cursor decides; the reaction narrates.

## 3. Creating the Slack app (manifest)

Per [App manifests](https://docs.slack.dev/app-manifests/configuring-apps-with-app-manifests): manifests are YAML/JSON; create at `api.slack.com/apps` → **Create New App → From a manifest** → pick workspace → paste → review; validation errors show inline (or via `apps.manifest.validate`).

### Manifest sketch

```yaml
display_information:
  name: Co-op Mode Crew
  description: The Crew of the co-op-mode World. Posts Recaps, reads Directives.
  background_color: "#1a1d21"
features:
  bot_user:
    display_name: crew
    always_online: true
oauth_config:
  scopes:
    bot:
      - chat:write
      - channels:history
      - channels:read
      - channels:join
      - reactions:write
settings:
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false
```

No Event Subscriptions, no Socket Mode, no request URL: the World **polls** on its own cron, so it needs no public endpoint and no signing-secret verification. The bot token doesn't expire unless token rotation is enabled — leave it off.

### Setup checklist (Jason, one-time)

1. `api.slack.com/apps` → Create New App → From a manifest → Jason's workspace → paste the YAML above.
2. **Install to Workspace**; copy the **Bot User OAuth Token** (`xoxb-…`).
3. Create/pick the Crew channel; `/invite @crew` (or let the first Tick call `conversations.join`). Copy the **channel ID** (channel details → bottom of About tab).
4. Confirm Jason's **member ID** (profile → ⋮ → Copy member ID).
5. Set GitHub Actions config on `stoudtio/co-op-mode`:
   - Secret `SLACK_BOT_TOKEN` = the new `xoxb-…` token
   - Variable `SLACK_CHANNEL_ID` = `C…`
   - Variable `SLACK_JASON_USER_ID` = `U…`
6. Smoke-test: one `chat.postMessage` with a `<@U…>` mention, one `conversations.history` read, one `reactions.add`.

## Existing credentials on Jason's side

`~/.bash_profile` already exports these Slack env var **names** (values not inspected): `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET`, `SLACK_JASON_USER_ID`, `SLACK_JASON_DM_CHANNEL`. So a Slack app already exists locally and Jason's user ID is already known.

- `SLACK_JASON_USER_ID` can be reused as-is for the Actions variable.
- The existing `SLACK_BOT_TOKEN` belongs to whatever local app backs it (e.g. Claude Code's Slack plugin/MCP setup). **Provision the World its own app/token anyway**: Actions can't reach the local MCP, the existing token's scopes may not match, and a dedicated token is independently revocable and auditable. `SLACK_SIGNING_SECRET` is irrelevant to a poll-only bot (it verifies *incoming* Slack requests, of which there are none).

## Sources

- https://docs.slack.dev/reference/methods/chat.postMessage
- https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks
- https://docs.slack.dev/messaging/formatting-message-text
- https://docs.slack.dev/reference/methods/conversations.history
- https://docs.slack.dev/reference/methods/conversations.join
- https://docs.slack.dev/reference/methods/reactions.add
- https://docs.slack.dev/app-manifests/configuring-apps-with-app-manifests
- https://docs.slack.dev/changelog/2025/05/29/rate-limit-changes-for-non-marketplace-apps/
- https://docs.slack.dev/changelog/2025/06/03/rate-limits-clarity/
