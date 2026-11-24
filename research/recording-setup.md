# Research 17: Recording and editing setup for a parent + 9-year-old build-in-public Three.js series

Researched 2026-11-24. Mac: Apple Silicon M3, 36 GB. Every source is listed with the date it was checked. A date in parentheses is the page's own publication or update date, where the page gives one.

## Recommendation

**Recording: OBS Studio (free, v33.0.3), one scene, one camera, two wireless lavs on separate channels, and the control plane driving OBS over obs-websocket.**

- **Video:** macOS Screen Capture of the editor and browser, plus one camera framing both faces. An iPhone as a Continuity Camera is the lightest good option. The built-in camera is the fallback. One camera is enough. A second camera doubles the sync and editing work and gains little for a desk-bound coding show.
- **Audio:** a DJI Mic Mini kit (2 TX + 1 RX, about $169 at launch) with the receiver plugged into the Mac over USB-C. Set it to **S (Stereo) mode**, which puts each transmitter on its own channel: left is the parent, right is the kid.
- **Tracks:** OBS Advanced output mode with up to 6 audio tracks. Suggested layout: T1 = full mix (for a quick review), T2 = mic stereo pair (L = parent, R = kid), T3 = game/desktop audio from the macOS Audio Capture source.
- **Control plane:** obs-websocket is built into OBS. `StartRecord`/`StopRecord` return `outputPath`, so the control plane can write a capture-log row automatically. In Hybrid MP4, `CreateRecordChapter` can drop a "shot N" marker live from a shot-list button.
- **Why not the others:** Screen Studio is $29/mo or $9/mo billed yearly and appears to take one microphone. QuickTime takes one input and has no tracks. Riverside and Descript are remote-call recorders with watermarked or capped free tiers, built for people in different places, not two people at one Mac.

**Editing: DaVinci Resolve (free, 21.1.x).**

- **What it does here:** it handles multitrack audio and picture-in-picture, exports up to UHD/60 fps, and exports timeline markers to EDL. The control plane can turn that EDL into the YouTube description's chapter list.
- **Caveats:**
  - The learning curve is steep for a first-time editor.
  - Text-based editing and the AI Neural Engine features are **Studio-only** ($295).
  - Since **21.1 (2026-09-08)**, "advanced scripting now requires DaVinci Resolve Studio." Free keeps only in-app console scripting. **Do not plan control-plane automation on the free Resolve scripting API.** Use file interchange (EDL marker files) instead.
- **Fallback:** if Resolve overwhelms them after two episodes, iMovie (free, still supported) is the easy option. It cannot export markers, so chapters would have to be typed into the control plane by hand.
- **Avoid:** CapCut (terms below) and Descript Free (720p with a watermark).

## 1. Recording

| Option | Cost | Screen + face cam | Two voices on separate tracks | Ease for a 9-year-old to be on | Notes |
|---|---|---|---|---|---|
| **OBS Studio** | Free, open source. Latest release 33.0.3 (2026-11-22) | Yes. macOS Screen Capture plus a video capture device (Continuity Camera works) | **Yes.** Advanced output records up to 6 audio tracks, assigned per source in Advanced Audio Properties. The KB suggests MKV and remuxing to MP4 | Easy to be on, since the parent operates it. One-time setup is fiddly | Hybrid MP4 (since 30.2) supports chapter markers via hotkey or websocket. The websocket API lets a local web app start and stop recording and read file paths |
| **Screen Studio** | $29/mo, or $9/mo billed yearly. 40% edu discount | Yes, webcam + mic | **No evidence of two mics.** The guide says to pick "the device" (one). Raw camera, mic and screen tracks can be extracted from the project | Very easy, with automatic zooms | Best for polished solo demos, poor for two voices. The stereo DJI receiver may still arrive as one stereo input, but that is unverified |
| **QuickTime / macOS screenshot toolbar** | Free | Screen + one mic only. The camera would need a separate recording | No. One input, one file | Easy | Fine for a quick test, not for an episode |
| **Riverside** | Free: 2 h one-time multitrack, 720p, watermark. Pro $24/mo annual | Yes, via browser | Yes, per participant, but each person needs their own device | Needs two devices | Built for remote guests. Overkill and costly here |
| **Descript (Rooms recorder)** | Free: 1 media-hour/mo, 720p, watermark. Hobbyist $16/mo annual | Yes | Multitrack per participant | Two devices again | The editor is beginner-friendly, but the free tier is too limited |

### One camera or two?

One. A single iPhone on a small stand, set wide enough to frame both faces and placed as a corner overlay over the screen capture, is the lightest setup. Continuity Camera needs iOS 16+, iPhone XR or later and macOS 13+, and works wired (wired keeps the phone charged and is steadier) or wirelessly. Desk View and Center Stage are available too. Center Stage will try to keep both people in frame, which suits a fidgety 9-year-old.

### Microphones for two people at a low budget

| Option | Approx. cost | Separate tracks? | Verdict |
|---|---|---|---|
| One shared USB desk mic | ~$50–100 (estimate, not sourced) | No. Both voices on one channel; the kid's level depends on how close they sit | Cheapest. Fine for episode 1. Expect level fights |
| Two wired USB mics | ~2 × $50+ (estimate) | Each is its own macOS input, so each can go on its own OBS track | Cable clutter at a desk. macOS will see two inputs, which works in OBS |
| **DJI Mic Mini 2 TX + 1 RX (+ case)** | **$169 at launch** (PetaPixel / TechRadar, 2024-11-26). Current price varies | **Yes, in S (Stereo) mode:** "audio from the two transmitters is recorded via separate channels" (DJI FAQ) | **Recommended.** 10 g clip-ons a kid will tolerate, so they can move without losing level. Gotcha: DJI does not say outright that USB-C to a computer carries both channels. Verify in Audio MIDI Setup on day 1 |
| Rode Wireless GO / Hollyland Lark kits | Higher | Usually also stereo/split | Alternatives if DJI is unavailable. Not researched in depth |

Splitting the DJI stereo pair into per-person tracks: record the stereo pair on one OBS track, then in Resolve treat L and R as separate mono channels in the clip's audio attributes. That is a standard Resolve operation, but the exact menu was not verified here.

## 2. Editing

| Editor | Cost | Beginner fit for a 10–15 min episode | Markers / chapters out | Caveats |
|---|---|---|---|---|
| **DaVinci Resolve (free)** | Free. Studio $295 | Medium. Cut/Edit pages are learnable, but there is a lot of UI | Timeline Markers to EDL (right-click the timeline in the Media Pool, then Timelines > Export). The Deliver page's YouTube tab "Chapters from Markers" writes chapters into the uploaded description (documented for v17 in 2022, first marker at 00:00). FCPXML/XML timeline import supported | Free: UHD/60 fps max, 8-bit. Text-based editing, Neural Engine AI and Magic Mask are Studio-only. **Python scripting moved to Studio in 21.1** |
| **iMovie** | Free, still on Apple's support site | **Easiest** | No marker or chapter export | Limited tracks and PiP. Fine as a fallback |
| **CapCut desktop** | Free tier; Pro reportedly $19.99/mo or $179.99/yr (third-party, Oct 2026) | Easy, but tuned for short-form | Not researched (no reason to) | **Terms (updated 2026-04-15):** "intended for individuals 13 years old and older"; grants TikTok USDS Joint Venture an "unconditional, non-exclusive, royalty-free, fully transferable, sub-licensable, perpetual and worldwide license" to User Content, plus use of username, image and likeness. Music: "NO RIGHTS ARE LICENSED WITH RESPECT TO SOUND RECORDINGS." Projects using Pro assets need Pro to export. **Not appropriate for footage of a 9-year-old** |
| **Descript** | Free: 720p + watermark; Hobbyist $16/mo annual | Very easy (edit the transcript) | Not researched | Free tier unusable for publishing. Hobbyist is a reasonable paid alternative if Resolve proves too hard |
| Final Cut Pro (for reference) | $299 one-time, or Apple Creator Studio $12.99/mo / $129/yr (Jan 2026) | Medium | FCPXML is its native interchange | Not free |

## 3. Control plane: what the capture log should track

### Per-clip fields

Most can be filled automatically from `ffprobe -show_format -show_streams -of json` plus the obs-websocket `outputPath`.

| Field | Source | Why |
|---|---|---|
| `episode_id`, `shot_id` (FK to shot list), `take` | control plane | Joins a clip to its shot |
| `file_path`, `file_size`, `sha256` (optional) | `outputPath` / filesystem | Find the file again, detect moves |
| `container` (mkv / mp4 / hybrid-mp4 / mov) | ffprobe `format_name` + OBS setting | Hybrid MP4 carries chapters. MKV must be remuxed before some editors read it |
| `recorded_at_start` / `recorded_at_end` (wall clock, ISO 8601) | websocket events `RecordStateChanged` | OBS files carry no SMPTE timecode, so wall clock is the sync key |
| `duration_s` | ffprobe | Total runtime, chapter math |
| `video`: codec (h264 / hevc / prores), width × height, fps, CFR/VFR, pixel format | ffprobe stream 0 | VFR screen captures can drift in editors. Prefer CFR in OBS |
| `audio_tracks[]`: index, codec, channels, sample rate, **role label** (mix / mics L=parent R=kid / game) | ffprobe + control-plane track map | The layout is the thing everyone forgets |
| `chapters[]`: time + title | ffprobe `-show_chapters` (Hybrid MP4) or control plane's own button log | Seeds the episode's chapter list |
| `status` (keep / reshoot / discarded), `notes` | human | Shot list closure |
| `obs_profile` / `scene_collection` name | config | Reproduce settings next episode |

### Chapters into the YouTube description

- **Path A (live):** a "new chapter" button in the control plane calls `CreateRecordChapter` and also stores the timestamp itself. Those are raw-recording times and shift once the footage is cut, so they are only a draft.
- **Path B (authoritative):** mark chapters on the Resolve timeline, run **Export > Timeline Markers to EDL**, and have the control plane parse the EDL (CMX3600 text, marker name in the comment line) into `00:00 Title` lines.
- **Validation:** the first chapter must be 00:00, there must be at least three chapters in ascending order, and each must be at least 10 s long (YouTube Help). The control plane should check these.
- **Alternative:** Resolve's Deliver > YouTube "Chapters from Markers" can do this on direct upload. That skips the control plane but is documented only for v17 (2022), so verify it in 21.x.

### Scripting

- **OBS:** fully scriptable through obs-websocket v5 (built in). This is the one integration worth building.
- **Resolve free:** after 21.1, console-only scripting. There is no external API, so a local web app cannot drive it. Studio ($295) restores it and adds 21.1.1 async APIs. Design around EDL files so the control plane never needs Resolve scripting.

## Open risks

1. **DJI over USB-C on a Mac, stereo split:** DJI documents S mode, but not explicitly that the computer link carries two channels. Test it in Audio MIDI Setup before buying a second kit. Fallback: Ms mode, or two USB mics.
2. **Hybrid MP4 with multiple audio tracks:** the OBS KB does not confirm it. A forum report (2024–25) lost multichannel audio in Hybrid MP4. If it fails, record MKV or MOV with multiple tracks and keep chapters in the control plane only.
3. **Resolve free licensing drift:** 21.1 already removed scripting. Further moves of features to Studio are possible. Pin the version or accept upgrades deliberately.
4. **Resolve learning curve for a beginner:** iMovie is the escape hatch, but it loses marker export.
5. **Child on camera:** a video is not automatically "made for kids" because a child appears in it (YouTube Help). The audience decision (kids vs. general) still has to be made. Setting it to made-for-kids disables several features, and those features were not enumerated here.
6. **Screen Studio multi-mic:** not confirmed either way. Excluded on cost and on the one-device guide wording.
7. **Price figures:** CapCut Pro and the shared-USB-mic prices come from third-party or estimated figures. The DJI price is the 2024 launch price.
8. **Source Record plugin (ISO per-source video):** v0.5.0 (2026-11-07), but forum reports of missing audio and crashes on macOS. Not needed with a single camera, so it is not recommended.

## Sources (all checked 2026-11-24)

- OBS latest release 33.0.3, 2026-11-22: GitHub API `repos/obsproject/obs-studio/releases/latest`
- OBS Advanced Recording Guide and Multi Track Audio (up to 6 tracks, MKV suggested): https://obsproject.com/kb/advanced-recording-guide-and-multi-track-audio
- OBS Hybrid MP4 KB (introduced 30.2; chapters via hotkey or websocket): https://obsproject.com/kb/hybrid-mp4
- OBS 30.2 release notes: https://obsproject.com/blog/obs-studio-30-2-release-notes
- OBS macOS desktop audio capture (macOS 13+): https://obsproject.com/kb/macos-desktop-audio-capture-guide
- obs-websocket protocol (StartRecord, StopRecord `outputPath`, CreateRecordChapter Hybrid-MP4-only, RecordStateChanged): https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md
- Source Record plugin, v0.5.0 2026-11-07: https://github.com/exeldro/obs-source-record ; macOS audio reports: https://obsproject.com/forum/threads/source-record.143697/page-38
- Hybrid MP4 multichannel audio loss report: https://obsproject.com/forum/threads/hybrid-mp4-chapters-in-davinci-resolve-studio.178330/
- Screen Studio pricing: https://screen.studio/pricing ; webcam and mic guide: https://screen.studio/guide/webcam-microphone ; raw file extraction: https://screen.studio/guide/extracting-raw-recording-files ; separate-audio feature request: https://hub.screen.studio/p/export-audio-and-video-separately
- Riverside pricing: https://riverside.com/pricing
- Descript pricing: https://www.descript.com/pricing
- Apple Continuity Camera (2026-05-06): https://support.apple.com/en-us/102546
- iMovie support: https://support.apple.com/imovie
- Apple Creator Studio (2026-01-28): https://macrumors.com/2026/01/28/apple-creator-studio-now-available
- DJI Mic Mini FAQ (S/M/Ms modes, two TX on separate channels): https://www.dji.com/my/mic-mini/faq ; store: https://store.dji.com/product/dji-mic-mini-2-tx-1-rx ; launch price (2024-11-26): https://petapixel.com/2024/11/26/djis-tiny-new-10-gram-wireless-mic-mini-promises-pro-audio-performance
- DaVinci Resolve 21 readme, 21.1 (2026-09-08), "Advanced scripting now requires DaVinci Resolve Studio": https://www.blackmagicdesign.com/us/support/content/readme/baf7c071c0524fbf8ccc961925c9f443
- Puget Systems on 21.1 scripting removal (2026-09-10): https://www.pugetsystems.com/?p=49790
- Digital Production, 21.1.1 async APIs (2026-10-06): https://digitalproduction.com/2026/10/06/davinci-resolve-21-1-1/
- Resolve free vs Studio comparison ($295; UHD/60 fps free; AI and text-based editing in Studio): https://www.blackmagicdesign.com/products/davinciresolve/compare
- Resolve "Timeline Markers to EDL" (DR12 manual, old but path still used): https://ltbits.github.io/davinci-resolve-manuals/DR12/DR12-RM-38.pdf ; converter guide: https://en.editingtools.io/guides/convert-resolve-marker-to-youtube-chapter-markers
- Resolve Deliver "Chapters from Markers" (PremiumBeat, 2022-03-10, v17): https://www.premiumbeat.com/blog/youtube-chapters-davinci-resolve/
- YouTube chapters rules: https://support.google.com/youtube/answer/9884579
- YouTube "made for kids" determination: https://support.google.com/youtube/answer/9528076
- CapCut Terms of Service (updated 2026-04-15): https://www.capcut.com/clause/terms-of-service
- CapCut Pro desktop pricing note: https://www.capcut.com/help/different-price-for-capcut-mobile-and-desktop ; third-party price check (Oct 2026): https://dupple.com/learn/is-capcut-free
