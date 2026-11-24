# Research 16: "Made for kids" for the Iron Foundry build-in-public series

Researched 2026-11-24. **This is not legal advice.** It is a reading of primary
sources by a non-lawyer, written to inform a decision. Anything that matters
financially or legally (especially the state child-influencer laws) should be
checked with a lawyer in the family's state.

Access dates: every source below was fetched on **2026-11-24**. YouTube Help
Center pages show no "last updated" date, only a "©2026 Google" footer, so their
content is dated by the access date only.

---

## Recommendation (up front)

1. **Keep "made for kids" (MFK), as long as kids are any part of the intended
   audience.** That is the legally safe direction. YouTube defines *mixed
   audience* (kids are *one of* the audiences, even if not the main one) as a
   kind of MFK, and gives creators no separate mixed-audience setting. A series
   with a 9-year-old co-host, about making a video game, that points viewers to
   a kid-safe game, meets several of the FTC/YouTube factors at once: child
   actor, an activity that appeals to children, and intent that children watch.
   Over-designating costs features and revenue. Under-designating is how Disney
   came to pay $10M (2025).
2. **Set the designation per video, not only at the channel level.** The FTC's
   Disney lesson (2025-09-02) is that a channel default silently mislabels
   individual videos. Set the channel to MFK *and* confirm the setting on every
   upload. A parent-only "how we set this up" video can be marked not made for
   kids individually if it is plainly not aimed at children.
3. **Revisit expectations, not the label.** MFK removes most build-in-public
   mechanics: comments, the notification bell, cards and end screens, viewer
   saves to playlists and Watch Later, the miniplayer, posts, memberships, and
   personalized ads. Growth will lean on search, browse and recommendations
   "alongside other kids' videos", plus off-platform channels the parent runs.
   This is the cost of the safe choice. It is not a reason to drop it.

> **REVISIT FLAG (conditional).** If the honest answer is that the series is
> *for parents and indie devs* (a devlog in which the kid appears), the MFK label
> is strategically wrong. It routes the videos to kid audiences and strips the
> feedback loop the target audience expects. In that case, set the videos to
> "not made for kids", turn comments off by hand to protect the child, and keep
> the metadata, thumbnails and language clearly adult-facing. **Do not choose
> that route if any episode invites kids to watch, learn, or go play the game.**
> Once kids are an intended audience, the content is mixed audience, which is
> MFK. The evidence does not say the current decision is wrong. It says the
> decision depends on one question: who is this for?

4. **The game site must collect nothing.** No analytics, no cookies or storage
   of identifiers, no third-party scripts, CDNs or web fonts (self-host
   Three.js and fonts), no embeds, no outbound links. GitHub itself logs
   visitor IP addresses "for security purposes" on every Pages site, and that
   logging cannot be turned off (see section 4).
5. **Child co-host basics:** parental consent (he is the parent), voluntary
   participation, first name only, no school/town/home-exterior details, no
   bedroom filming, a visible adult on any live stream. If the channel is
   monetised, **check the family's state child-influencer law before the first
   paid video. Minnesota bans children under 14 from compensated content
   creation outright.**

---

## (1) What changes when content is set to "made for kids"

Source: YouTube Help, *Set your channel or video's audience*
<https://support.google.com/youtube/answer/9527654> (accessed 2026-11-24);
*Features impacted by COPPA: Made for Kids content*
<https://support.google.com/youtube/answer/9632097?hl=en-GB> (accessed 2026-11-24).

### Features lost on MFK videos and live streams (per answer/9527654)

| Feature | Status on MFK |
|---|---|
| Comments | off |
| Notification bell | off |
| Cards and end screens | off |
| Video watermarks | off |
| Autoplay on home | off |
| Playback in the Miniplayer | off |
| Save to playlist / Save to Watch Later (viewer side) | off |
| Channel Memberships | off |
| Donate button | off |
| Live chat and live chat donations | off |
| Super Chat / Super Stickers | off |
| Merchandise and ticketing | off |
| Likes/dislikes on YouTube Music | off |
| **Personalized advertising** | off |

**Channel-level MFK** also removes Channel Memberships, the notification bell,
**Posts** (community posts), and **Ask Studio**.

**Shorts remix: not confirmed.** Neither MFK feature page lists it. The Child
safety policy (below) does say that content *featuring minors* "may" have
**Shorts remixing** disabled, along with comments, live chat, live streaming,
recommendations, posts and views from signed-out users. So the co-host alone may
cost remix, whatever the MFK setting.

### Monetisation

- answer/9527654: "we don't serve personalized ads on kids content", as
  required by COPPA, and the change "may result in a decrease in revenue for
  some creators". MFK videos can still carry **non-personalized
  (contextual) ads**. answer/9632097 adds that ads in certain categories may
  appear and that viewers may see ad bumpers, under a separate MFK ads policy.
- Quality gate. *YouTube's quality principles for kids and family content*
  <https://support.google.com/youtube/answer/10774223> (accessed 2026-11-24):
  low-quality MFK content (heavily promotional, deceptively educational, hard
  to follow, sensational) can get "limited or no ads", and a channel focused on
  it can be suspended from YouTube Partner Program (YPP). High-quality MFK
  content (learning, curiosity, creativity) "gets raised up in recommendations"
  and the same principles decide **YouTube Kids** inclusion. A father-and-kid
  game-building series fits the "learning and curiosity / creativity and
  imagination" categories well.

### Discovery and recommendations

- FAQ <https://support.google.com/youtube/answer/9684541> (accessed
  2026-11-24): MFK videos "are more likely to be recommended alongside other
  kids' videos." Autoplay on home is off.
- In practice the series loses its retention tools (bell, end screens, cards,
  viewer playlist saves) and its community tools (comments, posts). It keeps
  search, browse, suggested videos, playlists the creator builds, the video
  description, and possible YouTube Kids eligibility.

### Channel vs video setting, and overrides (answer/9527654)

- The channel setting applies to existing and future videos. "Settings for
  individual videos will override the channel setting."
- YouTube uses machine learning to find videos "clearly directed to young
  audiences" and may override the setting "in cases of error or abuse". A video
  YouTube has forced to MFK cannot be changed back, and can be appealed only
  once per video. "Do not rely on our systems to set your audience for you."

---

## (2) What COPPA and YouTube policy require of this creator

### COPPA as applied to YouTube creators

- FTC blog, *YouTube channel owners: Is your content directed to children?*,
  **2019-11-22**
  <https://www.ftc.gov/business-guidance/blog/2019/11/youtube-channel-owners-your-content-directed-children>:
  COPPA "applies in the same way it would if the channel owner had its own
  website or app." A channel owner is an operator if the content is child-directed
  and the owner, *or someone acting for it such as an ad network*, collects
  personal information. Civil penalties then were "up to $42,530 per violation".
  The figure is adjusted for inflation every year, so the current number is
  higher (check it). The FTC's own case, against Google/YouTube, settled for $170M.
- FTC COPPA FAQ
  <https://www.ftc.gov/business-guidance/resources/complying-coppa-frequently-asked-questions>
  (accessed 2026-11-24; its banner points to the 2025 amended Rule, but the
  body may predate it): COPPA "applies only to those websites and online
  services that collect, use, or disclose personal information from children."
  A creator posting on a general-audience platform "will be deemed an operator"
  if the content is child-directed and personal information is collected by the
  creator or on its behalf.
- **What this means in practice:** setting MFK is how a creator complies on
  YouTube. It switches off the personalized-ad data collection that would
  otherwise make the creator an operator. The creator's remaining duties are to
  label accurately, per video, and not to collect children's data anywhere
  else: no "comment your name below", no email signups, no Discord invites,
  and a game site that collects nothing.

### Enforcement precedent: Disney (2025)

- FTC blog, *Protecting children watching YouTube videos: lessons learned from
  the FTC's settlement with Disney*, **2025-09-02**
  <https://www.ftc.gov/business-guidance/blog/2025/09/protecting-children-watching-youtube-videos-lessons-learned-ftcs-settlement-disney>;
  DOJ press release
  <https://www.justice.gov/opa/pr/disney-agrees-10m-civil-penalty-and-injunction-alleged-violations-childrens-privacy-laws>;
  case page <https://www.ftc.gov/legal-library/browse/cases-proceedings/disney>
  (order approved **December 2025**, per the case page).
- **$10M civil penalty.** Disney set its audience at the channel level, so
  child-directed videos went out as "not made for kids". That enabled targeted
  ads to under-13 viewers, plus autoplay and open comments. The order requires a
  program that reviews **every video** for child-directed content.

### 2025 COPPA Rule amendments

- Federal Register, 90 FR (No. 76), **published 2025-04-22**
  <https://www.govinfo.gov/content/pkg/FR-2025-04-22/html/2025-05904.htm>.
  - **Effective 2025-06-23.**
  - **Compliance date 2026-04-22** (already passed as of today), except
    §312.11(d)(1), (d)(4) and (g), the safe-harbor provisions, which run on
    their own schedule. Secondary sources say safe-harbor operator lists were
    due 2025-10-22.
- Changes relevant here:
  - **Separate verifiable parental consent** before disclosing a child's
    personal information to third parties (§312.5(a)(2)), which covers
    targeted advertising.
  - A **written data retention policy** with deletion timeframes (§312.10),
    and a **written information security program** (§312.8(b)).
  - **Biometric identifiers** added to "personal information". The final text
    is narrower than proposed and dropped "data derived from voice, gait, or
    facial data".
  - "Mixed audience website or online service" is codified: a service that
    "does not target children as its primary audience" and that does not
    collect personal information before it collects age information, except
    for limited §312.5(c) purposes.
  - The "directed to children" factors are unchanged in kind. The FTC
    discusses intended, actual and likely audience.
  - Unchanged: a **photo, video or audio file containing a child's image or
    voice** is personal information, and so is a **persistent identifier**
    (for example an IP address or a cookie ID).
- **Relevance:** none of these duties binds this creator as long as nothing is
  collected. They all bind the moment the game site, a newsletter, or a fan
  submission flow collects anything.

### YouTube policy on minors appearing in videos

- **Child safety policy** <https://support.google.com/youtube/answer/2801999>
  (accessed 2026-11-24).
  - Content that complies with the policy but features minors "may" have
    comments, live chat, live streaming, recommendations, posts, Shorts
    remixing, and signed-out views disabled.
  - Live streams with minors **under 16 not visibly accompanied by an adult**
    may be removed or have chat disabled.
  - Do not post minors filmed in **private spaces such as bedrooms or
    bathrooms**, minors soliciting contact from strangers, dares or adult
    topics, or content revealing a minor's personal details.
  - "Misleading family content" (mature themes in content aimed at kids) is
    prohibited. This matters for game violence: keep the Iron Foundry game and
    the thumbnails age-appropriate.
  - Three strikes in 90 days means channel termination.
- **Best practices for content with children**
  <https://support.google.com/youtube/answer/9229229> (accessed 2026-11-24).
  - Get parent or guardian consent, and make participation **voluntary**.
  - Manage privacy and embed settings.
  - Follow local law on **work permits, wages or revenue share for the minor,
    schooling, working hours and conditions** ("should not work into the
    night").

### State child-influencer ("Coogan-style") laws: flagged, not deep-dived

These apply only if the channel is **monetised** and the child appears in a
large share of the content. Verify against the statute in the family's state.

| State | Status (as found) | Key point |
|---|---|---|
| **Minnesota**, Minn. Stat. §181A.13 <https://www.revisor.mn.gov/statutes/cite/181A.13> (2024 c 103; reported effective 2025-07-01) | in force | **A minor under 14 "is prohibited from engaging in the work of content creation"** when they appear in ≥30% of compensated video content in a 30-day period and pay is ≥$0.01/view or the platform's threshold. Earnings must go into a trust. Minors 13+ can demand deletion. **For a 9-year-old co-host in MN, monetising may be prohibited outright.** |
| Illinois (SB 1782) | in force 2024-07-01 (secondary source) | trust set-aside for minors under 16; the child has a private right of action |
| California (AB 1880 / SB 764) | in force 2025-01-01 (secondary source) | the 30% content threshold triggers a trust set-aside (percentage per statute) |
| Utah (2025, bill number reported inconsistently) | enacted 2025 | trust duty above a $150k earnings threshold (secondary source) |

Other states have bills pending. The Illinois, California and Utah rows come
from secondary reporting (Axios 2024-07-27, Davis+Gilbert, GMA), not from
statute text.

---

## (3) Is "made for kids" the right designation?

### The deciding factors

- YouTube (answer/9528076, *Determining if your content is made for kids*,
  accessed 2026-11-24) lists these factors:
  - subject matter
  - whether children are the **intended audience (including metadata)** or the
    **actual** audience
  - **child actors or models**
  - characters, celebrities or toys that appeal to kids
  - language children are meant to understand
  - **activities that appeal to children, such as play-acting, simple songs
    or games**, or early education
  - songs, stories or poems for children
  - empirical audience data
  - whether the content is advertised to children

  "No single factor" decides it. YouTube Analytics is "not designed to
  determine" it.
- The FTC factors (2019 blog; 16 CFR 312.2) are the same set in regulatory
  language: subject matter, visual content, animated characters,
  child-oriented activities and incentives, music, **age of models**, child
  celebrities, language, advertising, and empirical evidence. The 2025 Rule
  adds intended, actual and likely audience.
- Pointing the other way:
  - "Family-friendly", "traditionally child-associated activity", and
    "incidentally seen by children" do **not** by themselves make content MFK
    (answer/9528076).
  - The FTC: content isn't child-directed just because "some children may see
    it."
  - "General audience" content is not mixed audience, and should be set to
    not made for kids.
  - Game development with a code editor, Three.js and git is, on its subject
    matter, not a children's topic.

### Applying them to Iron Foundry

| Factor | Points toward MFK? |
|---|---|
| 9-year-old co-host, fully featured | **yes** (child actor) |
| Making a video game together | leans yes ("games", "activities that appeal to children"), though coding itself is adult-coded |
| Kid-safe game the videos send viewers to | **yes**, if videos invite kids to play it |
| Language and pacing | depends on how it is pitched |
| Build-in-public framing (devlog, tech stack) | leans no (general audience) |
| Thumbnails, title and tags | the creator controls these, and they count as "metadata" evidence |

**Verdict:** if any episode is pitched so that kids will watch (likely, given a
kid co-host and a kid-safe game built to be played), the content is at minimum
**mixed audience, which YouTube files under MFK.** MFK is then correct.

### Consequences of getting it wrong, in each direction

- **Labelled not made for kids, but child-directed (under-labelling):** COPPA
  exposure through personalized ads. That means FTC civil penalties per
  violation (Disney, $10M, 2025), plus YouTube account action and a forced MFK
  that cannot be reversed. This is the dangerous direction.
- **Labelled MFK, but actually general audience (over-labelling):** no COPPA
  harm. Collecting less data never violates COPPA. The costs are lost
  features, lower revenue, and recommendations steered toward kids' content.
  YouTube says it overrides only "in cases of error or abuse", and its examples
  of forced changes run toward MFK, not away from it. answer/9527654 does
  warn generally that an incorrect setting can bring YouTube action, but the
  sources describe no enforcement for over-labelling.
- **Asymmetry:** while the audience is uncertain, MFK is the conservative
  choice. Its comment shut-off also protects a child co-host, which YouTube may
  impose anyway under the Child safety policy.

---

## (4) The kid-facing static game on GitHub Pages

### What COPPA implies for the site

- If the YouTube videos send kids to the game, the site is plausibly **directed
  to children**. COPPA applies to the site **only if it collects personal
  information**, and personal information includes **persistent identifiers such
  as IP addresses and cookie IDs** (FTC FAQ C.5; §312.2 item 7). The FAQ holds
  the site owner "responsible for the collection of personal information from
  your users, **no matter who is doing the collection**" (D.9/D.11), which
  covers third-party scripts, plug-ins, analytics and ad networks.
- There is a narrow exception: identifiers used **solely for "support for
  internal operations"** (serving content, security, and similar), with no
  other personal information collected. Server and security logs fit it;
  analytics and profiling do not.

### What to avoid

- **Analytics of any kind:** Google Analytics, Plausible, Cloudflare Web
  Analytics, PostHog, Sentry. Even "cookieless" tools receive the IP address.
- **Third-party script, style or font hosts:** jsDelivr, unpkg, cdnjs, Google
  Fonts. Each one sees the child's IP and user agent and is a third party
  collecting "through" the site. **Bundle Three.js and self-host the fonts in
  the repo.**
- **Embeds:** YouTube iframes (use a plain text link only if you must, or
  `youtube-nocookie` still loads Google), social widgets, comment widgets.
- **Cookies, or `localStorage` holding any ID:** keep saved game progress
  local, with no identifier, and never sent anywhere.
- **Forms, accounts, high-score submission with names, chat, uploads.**
- **Outbound links**, including "view source on GitHub". A github.com page is
  a general-audience, logged-in service, and GitHub's own privacy statement
  says its services are not for under-13s. Keep any repo link in the YouTube
  description aimed at parents, not inside the game.
- **Ads and affiliate links.**

### What GitHub Pages itself logs

- *About GitHub Pages* docs
  <https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages>
  (accessed 2026-11-24): "When a GitHub Pages site is visited, the visitor's IP
  address is logged and stored for security purposes, regardless of whether the
  visitor has signed into GitHub or not." The site owner cannot switch this
  off. The docs do not mention cookies.
- *GitHub General Privacy Statement*, **effective 2026-04-27**
  <https://docs.github.com/en/site-policy/privacy-policies/github-general-privacy-statement>:
  usage data includes "IP address, device information, session details". It
  states "Our Services are not intended for individuals under the age of 13",
  and that GitHub does "not intentionally gather Personal Data from such
  individuals". It does not address github.io visitors or Pages cookies
  separately.
- **Reading (not legal advice):** GitHub's security logging is the clearest
  candidate for the "support for internal operations" exception. It is not
  zero, and the site owner does not control it. If that residual exposure
  matters, the alternative is a host whose logging policy the owner controls
  and can document. A custom domain does not change GitHub's logging.
- **Recommended:** add a one-paragraph privacy note on the site. The COPPA FAQ
  (C.1) recommends that child-directed sites post one even when they collect
  nothing. Say that the game collects nothing and uses no cookies, and that the
  host logs IP addresses for security, linking GitHub's statement.

---

## Open risks

1. **The audience question is unresolved.** The MFK verdict depends on whether
   kids are an intended audience. Decide and write it down, because the
   metadata will be evidence.
2. **Channel-level default drift** (the Disney failure mode). Add a per-upload
   check of the audience setting to the publishing checklist.
3. **The family's state is unknown.** If it is **Minnesota**, monetising
   episodes featuring the 9-year-old may be **prohibited**. In IL, CA, UT and
   others, trust set-asides apply once thresholds are met. Check before
   joining YPP.
4. **The current COPPA civil-penalty figure was not verified.** The 2019 figure
   is $42,530 per violation, adjusted for inflation annually.
5. **Not confirmed in primary sources:** the full MFK effect on Shorts remix,
   and whether MFK videos rank differently in *search* (as opposed to
   recommendations).
6. **The FTC COPPA FAQ may predate the 2025 amendments.** Its banner says so.
   The Rule text (govinfo, 2025-04-22) governs.
7. **GitHub IP logging is irreducible on Pages.** Relying on the internal
   operations exception is a judgment call, not a documented safe harbor.
8. **Off-platform leakage of the child's personal information:** links in
   descriptions to a Discord, newsletter, or socials. Keep anything that
   collects data parent-facing and off the kid path, or the creator becomes
   an operator under the amended Rule, with its consent, retention and
   security-program duties.
9. **The game's content must stay kid-appropriate.** Violence or horror in a
   kid-aimed video risks the "misleading family content" prohibition.
