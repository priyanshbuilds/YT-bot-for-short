# YT-Bot-For-Short — An autonomous YouTube Shorts channel by priyanshbuilds

A self-hosted pipeline that **researches, scripts, produces, publishes, and learns** — one
faceless YouTube Short at a time, three times a day, with no human in the loop. It plans a topic,
validates it against real competitor data before spending a minute of compute, hand-builds an
animated explainer video, uploads it to YouTube, then reads its own analytics every week and
rewrites its own playbook to get more views.

It is driven by the [Claude Code](https://claude.com/claude-code) CLI running headlessly on a
Windows box + a local GPU, orchestrated by Windows Task Scheduler, publishing through the YouTube
Data API (with [Postiz](https://github.com/gitroomhq/postiz-app) as a scheduling backend).

> **This is a real, running personal project — not a polished product.** It contains one operator's
> actual channel numbers, ledgers, and strategy notes (kept intentionally, as a build log). Paths,
> credentials, and the GPU/Docker setup are specific to the author's machine. Treat it as a
> reference architecture and a build-in-public artifact, not a one-click install. See
> [**Honest status**](#honest-status) below.

---

## What it actually does

```
                 ┌─────────────── every 12:00 / 17:00 / 21:00 ───────────────┐
                 │  DailyShorts:  pick topic → research (YouTube/Reddit/web)  │
                 │  → validate virality GATE → write script → generate voice  │
                 │  → render animated video → QA watch-back → upload to YouTube│
                 └────────────────────────────────────────────────────────────┘
   weekly ► ShortsLearn:   read our analytics + study top creators → rewrite the rules
   daily  ► CreatorStudy:  frame-by-frame study of a viral competitor → technique catalog
   daily  ► ShortsDigest:  email summary of what shipped + how it's performing
   always ► Dashboard:     old-school live control panel at http://localhost:8899
```

### The five autonomous loops

| Loop | Cadence | What it does |
|---|---|---|
| **DailyShorts** | 3×/day | Produces **one** full-quality Short and publishes it. One-at-a-time, never batched — quality over volume. |
| **CreatorStudy** | daily | Studies a top-performing competitor Short (hook, pacing, retention devices) into a growing technique catalog. |
| **ShortsLearn** | weekly | Reads the channel's real YouTube analytics + researches the market, then safely rewrites the producer's *self-learned rules*. |
| **ShortsDigest** | daily | Emails a digest of what shipped and how it's doing (+ failure alerts). |
| **Dashboard** | always-on | A monochrome, keyboard-era live dashboard: now / next / past / analytics / one-click loop triggers. |

Full system inventory: [`future_plans/PLATFORM_MAP.md`](future_plans/PLATFORM_MAP.md).

---

## Why it's built this way

Three ideas shape the whole system:

1. **Validate before you produce.** Every topic passes a virality GATE (real competitor view-counts
   + a simulated swipe-test) *before* any compute is spent. Losers are killed on paper.
2. **Never look repetitive.** YouTube terminates "inauthentic / repetitious" channels. A
   [**variation engine**](variation/) forces every new video to differ from the last six on ≥3 axes
   (title shape, format, visual skin, voice, length, topic cluster) — and every video is a
   hand-coded animation, not a template fill.
3. **Compound.** The channel isn't the business — it's the funnel. A weekly learning loop turns
   real analytics into better rules, and a full monetization roadmap lives in
   [`future_plans/`](future_plans/).

---

## Repository layout

```
├── daily_shorts_prompt.md      # the producer brief the AI follows to make + publish one Short
├── daily_shorts.bat            # runner: ensure Postiz up, then run the producer headlessly
├── post_to_youtube.py          # reliable direct upload via the YouTube Data API
├── post_to_postiz.py* / *.sh   # Postiz publish path + OAuth/token plumbing
├── ensure_postiz.ps1           # idempotent Postiz health-check / self-heal
│
├── learn_and_improve_prompt.md # weekly self-improvement analyst brief
├── yt_analytics.py             # pulls channel + per-video CTR / retention into a report
├── yt_top.py                   # finds real top competitor Shorts by view count
├── creator_study_prompt.md     # daily competitor-study brief
├── competitor_playbook.md      # harvested techniques (grows over time)
├── learnings.md                # append-only lab notebook of experiments
│
├── variation/                  # the anti-repetition engine (title shapes, formats, skins, voices)
├── daily_topics.md             # topic backlog with niche/CPM/angle/guardrail
├── daily_posts_ledger.md       # append-only record of everything published
│
├── dashboard.py                # the live control-panel dashboard (stdlib http.server)
├── notify_email.py             # Gmail-SMTP notifier (digest + failure alerts)
├── daily_digest.py             # builds the daily email digest
│
├── .claude/skills/             # the Claude Code skills (ultimate-short, punchy-recut)
├── ClipPilot/                  # the larger video-generation engine (Python pkg + Remotion source)
├── analytics/                  # real performance snapshots (kept as a build log)
└── future_plans/               # the researched monetization roadmap & risk register
```

`*.bat` / `*.ps1` = Windows runners · `*.sh` = Git-Bash helpers · `*_prompt.md` = briefs an AI agent
follows autonomously.

> **The core automation (producer, publisher, analytics, dashboard, learning, email) is
> Python-stdlib-only.** The heavy dependencies (`torch`, `diffusers`, `edge-tts`, `faster-whisper`,
> Remotion/Node) are only needed by the video/voice generation layer. See
> [`requirements.txt`](requirements.txt).

---

## Configuration & secrets

**No secrets are stored in this repo.** By design, every script reads credentials *at runtime* from
outside the repo:

| Secret | Where it's read from | Not in git because |
|---|---|---|
| YouTube OAuth client id/secret | the Postiz Docker container env | lives in your Docker setup |
| YouTube refresh token | Postiz's Postgres (`Integration` table) | lives in your DB |
| Postiz API key | `~/.config/postiz/key` | your home dir |
| Gmail app password | `~/.config/shorts/email.json` (mode 600) | your home dir |

Copy [`.env.example`](.env.example) and follow [`SETUP.md`](SETUP.md) to wire your own. **Never
commit real keys.** The included [`.gitignore`](.gitignore) blocks the common secret files as a
second line of defense.

---

## Thorough Repository Breakdown

This is a fully autonomous, self-hosted YouTube Shorts pipeline. It runs headlessly on a Windows machine (orchestrated by Windows Task Scheduler) and uses the Claude Code CLI + local GPUs to research, write, animate, and publish faceless YouTube shorts without any human intervention.

### 🧠 The "AI Agent" Briefs (Prompts)
Instead of hardcoding complex logic in Python, this system uses Markdown "briefs" to tell an AI agent (Claude Code) what to do.
*   **`daily_shorts_prompt.md`**: The master prompt that tells the AI to pick a topic, write a viral script, generate a voiceover, render the video, and upload it.
*   **`learn_and_improve_prompt.md`**: A weekly brief where the AI acts as an analyst. It looks at the channel's YouTube analytics and updates its own strategy playbook to get more views.
*   **`creator_study_prompt.md`**: A daily brief where the AI analyzes a top-performing competitor's video frame-by-frame to extract editing and pacing techniques.

### ⚙️ The Five Autonomous Loops
The system is divided into five main loops, triggered by simple `.bat` files on a schedule:
1.  **Daily Shorts (Producer)** (`daily_shorts.bat`): Runs 3x a day. It triggers the main pipeline to produce and publish a single video.
2.  **Learn & Improve (Analyst)** (`learn_shorts.bat`): Runs weekly. It evaluates past video performance to improve the next batch.
3.  **Creator Study (Researcher)** (`study_creators.bat`): Runs daily to append new hooks and formats to the `competitor_playbook.md`.
4.  **Digest (Notifier)** (`digest.bat` / `daily_digest.py` / `notify_email.py`): Compiles an HTML email digest summarizing what shipped today, how it performed, and sends it via Gmail SMTP.
5.  **Dashboard** (`dashboard.py` / `Dashboard.bat`): A lightweight, always-on local web server that provides a text-based control panel to monitor what the agents are currently doing.

### 🎬 Video Production & Asset Generation
*   **`ClipPilot/` directory**: The heavy lifting engine for rendering the actual video. It uses Python for asset generation and Remotion (React-based video rendering) to composite the animation.
*   **`zimage_gen.py` & `dl_zimage.py`**: Scripts for generating or downloading the visual assets for the shorts.
*   **`make_elephant_short.py` / `produce_short_zimage.py`**: Orchestration scripts that stitch the transcript, audio (TTS), and generated images together into an MP4 file.

### 📊 Analytics & Publishing
*   **`post_to_youtube.py`**: Direct integration with the YouTube Data API to upload the finalized MP4 file with the generated title, description, and tags.
*   **`ensure_postiz.ps1` & `postiz_yt_refresh.sh`**: Scripts to interact with Postiz (social media scheduling tool), ensuring OAuth tokens are fresh and the backend is healthy.
*   **`yt_analytics.py` & `yt_top.py`**: Scripts that scrape/pull YouTube analytics (Click-Through Rate, Retention graphs) for the AI analyst to read, and find top competitor videos.

### 📚 State, Ledgers, and Logs
Because the AI needs memory, it writes to Markdown files instead of a database:
*   **`daily_posts_ledger.md`**: An append-only log of every video the system has ever published.
*   **`daily_topics.md`**: The backlog of topics, niches, and CPM angles the system plans to cover.
*   **`learnings.md` / `competitor_playbook.md`**: The growing "brain" of the system where it stores new video formats and rules it has learned.
*   **`variation/` directory**: An engine containing `variation_ledger.md`. It forces the AI to use different visual skins, voice actors, and title formats so YouTube doesn't terminate the channel for being "repetitive".

---

## Tech stack

- **Orchestration:** Claude Code CLI (headless `claude -p … --dangerously-skip-permissions`) +
  Windows Task Scheduler
- **Publishing:** YouTube Data API v3 (direct upload) · Postiz (self-hosted, Dockerized)
- **Video:** Remotion (hand-coded SVG/TSX explainers) · FFmpeg · optional Z-Image / Wan2GP on GPU
- **Voice & captions:** edge-tts (rotating voices) · faster-whisper (word timing)
- **Analytics:** YouTube Analytics + Data API
- **Dashboard/notify:** Python stdlib (`http.server`, `smtplib`)

---

## Honest status

As of the last snapshot this is an **early, pre-monetization** channel — the interesting part is the
*machine*, not the numbers yet. Real, un-sanitized metrics, ledgers, and the strategy roadmap are
kept in the repo on purpose ([`analytics/`](analytics/), [`daily_posts_ledger.md`](daily_posts_ledger.md),
[`learnings.md`](learnings.md), [`future_plans/`](future_plans/)) so the growth curve is auditable.

**Known limitations:** hardcoded absolute Windows paths (author's machine), a GPU/Docker/Postiz setup
you'd have to reproduce, and a Claude subscription as the only real running cost. This is shared as a
reference architecture, not a turnkey install.

---

## ⚠️ Responsible-use notice

- This automates content creation. **Follow YouTube's Terms of Service** and its policies on
  automated/AI-assisted and authentic content. The variation engine + hand-built visuals exist to
  keep output genuinely original — don't defeat that.
- **Disclose AI-generated/synthetic media** where platforms require it (YouTube's altered-content
  label, TikTok's AI toggle).
- Nothing here is financial advice. Finance content must be truthful and include required
  disclosures (e.g. FTC affiliate disclosure).
- Respect the licenses and rate limits of every third-party API and asset you connect.

---

## Acknowledgements

Built on the shoulders of [Postiz](https://github.com/gitroomhq/postiz-app),
[Remotion](https://www.remotion.dev/), [edge-tts](https://github.com/rany2/edge-tts),
[faster-whisper](https://github.com/SYSTRAN/faster-whisper), FFmpeg, and
[Claude Code](https://claude.com/claude-code). Third-party model/asset libraries that this project
merely *uses* are not redistributed here.

## License

[MIT](LICENSE) — do what you want, no warranty. You are responsible for how you use it.
