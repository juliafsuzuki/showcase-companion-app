# Showcase Practice — Project Documentation

A founder's-log breakdown of how the *Showcase Practice* dance app was conceived, prompted, architected, and shipped to production at **[showcase-genie.vercel.app](https://showcase-genie.vercel.app/)**.

---

## 1. Prompts Used to Build This App

These are the exact prompts sent during the build, organized so they can be copied verbatim into a new session to reproduce the journey.

### 1.1 Initial Prompt

> Attached video is the showcase choreography that I will be performing. In order for me to learn the choreography, I want to break it down to digestible pieces / segments. Break the attached video into rhythmical phrases.

### 1.2 Subsequent Prompts (based on my review feedback)

**Prompt 2 — Iteration 3 (UX cleanup + audio cut)**

> I need you to make four more edits and redeploy the app!
> 1. Eliminate the "4-Pass practice loop". I don't see the value of having it.
> 2. Allow the users to start the phrase to play instead of automatically playing when a phrase is selected.
> 3. Cut the segments of the videos where a speaker says "Julia Friend Like Me" and "Julia A Whole New World".
> 4. Metronome should be aligned to the music playing.

**Prompt 3 — Metronome / play-button sync**

> Enable the play button to activate both the video playing and the metronome playing. Right now, Metronome is playing on its own. Keep the on/off button for Metronome.

**Prompt 4 — Publish to pplx.app**

> Please publish to a pplx.app link.

**Prompt 5 — Deploy via Vercel**

> Please deploy via the Vercel connector.

**Prompt 6 — Rename Vercel project**

> Can you rename to showcase-genie.vercel.app?

**Prompt 7 — Control layout polish**

> I need you to make two more edits and redeploy the app!
> 1. Rename "LOOP" to "REPEAT".
> 2. Place "SPEED" to the second line along with Metronome. I would like to have 3 items with the On/Off options to be displayed on the top/first line and 2 items with "SPEED" and "METRONOME" to be displayed on the bottom/second line.
> Finally, re-deploy these changes to the production. Thank you!

**Prompt 8 — Time-signature labeling + final layout swap**

> 1. To be consistent with WALTZ (3/4), display SWING (4/4).
> 2. The "SPEED", "COUNTS" and "METRONOME" should go together. Please place these 3 elements on the first/top line and "REPEAT" and "MIRROR" on the second/bottom line.
> 3. Replace "SHOWCASE · SWING" to "FRIEND LIKE ME · SWING (4/4)".
> 4. Replace "SHOWCASE · WALTZ" to "A WHOLE NEW WORLD · WALTZ (3/4)".
> Deploy all these changes to production.

---

## 2. Architecture & Build-Out Concepts

Each item below has three parts: a plain-language **Definition**, an ASCII **Visual**, and a **Description** of how it applies specifically to the Showcase Practice app.

---

### 2.1 Agentic AI Stack

**Definition.** The full set of AI services, models, infrastructure, and orchestration layers that an agent uses to perceive a task, plan, take actions in the real world (browsers, files, APIs), and verify results — without a human in the loop for every step.

**Visual.**

```
┌─────────────────────────────────────────────────────────────┐
│                     USER INTERFACE LAYER                     │
│              Perplexity Computer (chat + assets)             │
└──────────────────────────┬──────────────────────────────────┘
                           │  natural-language prompts
┌──────────────────────────▼──────────────────────────────────┐
│                ORCHESTRATION / PLANNING LAYER                │
│        Claude Sonnet 4.6  (todo lists, tool selection)       │
└──────────────────────────┬──────────────────────────────────┘
                           │
   ┌───────────────────────┼─────────────────────────┐
   ▼                       ▼                         ▼
┌──────────┐         ┌───────────┐            ┌─────────────┐
│  Tools   │         │   Media   │            │  Connectors │
│ bash,    │         │  ffmpeg,  │            │  Vercel,    │
│ edit,    │         │  Whisper, │            │  Google     │
│ js_repl, │         │  Image    │            │  Drive,     │
│ grep,    │         │  models   │            │  pplx-tool  │
│ write    │         │           │            │             │
└────┬─────┘         └─────┬─────┘            └──────┬──────┘
     │                     │                          │
     └─────────────────────┼──────────────────────────┘
                           ▼
              ┌──────────────────────────┐
              │   EXECUTION SANDBOX      │
              │   Linux VM + workspace   │
              │   Static site files      │
              └──────────────────────────┘
```

**Description.** For *Showcase Practice*, the stack was:

- **UI layer:** Perplexity Computer chat surface — where Julia issued prompts and reviewed deployed previews inline.
- **Orchestration model:** Claude Sonnet 4.6 acted as the planning agent — interpreting each prompt, building a todo list, choosing tools, and verifying its own output.
- **Tool layer:** `bash`, `write`, `edit`, `grep`, `read`, `js_repl` (Playwright for visual QA), and the `pplx-tool` CLI for screenshots and deploy.
- **Media layer:** `ffmpeg` for cutting the announcer audio and slicing the full showcase into per-phrase clips; vision models for QA screenshots.
- **Connectors:** The **Vercel** connector for production hosting and the **Google Drive** connector (this doc) for documentation.
- **Sandbox:** A Linux VM holding `/home/user/workspace/dance-app/` — the static HTML/CSS/JS bundle plus all phrase clips — that gets uploaded on each deploy.

---

### 2.2 System Design

**Definition.** The high-level blueprint that shows every component of the running product, how they connect, and how a user request flows from click to pixels.

**Visual.**

```
                    ┌─────────────────────────┐
                    │      Julia (browser)    │
                    │   showcase-genie...     │
                    └────────────┬────────────┘
                                 │ HTTPS
                                 ▼
                  ┌──────────────────────────────┐
                  │   Vercel Edge CDN (global)   │
                  │   *.vercel.app alias layer   │
                  └────────────┬─────────────────┘
                               │
                ┌──────────────┴───────────────┐
                ▼                              ▼
      ┌──────────────────┐         ┌──────────────────┐
      │  Static bundle   │         │  Video clips     │
      │  index.html      │         │  /clips/         │
      │  styles.css      │         │  friend-like-me  │
      │  app.js          │         │  whole-new-world │
      │  assets/*.js     │         │  (mp4, ~317 MB)  │
      └──────────────────┘         └──────────────────┘
                │                              │
                └──────────────┬───────────────┘
                               ▼
                   ┌────────────────────────┐
                   │  Browser runtime       │
                   │  • HTML5 <video>       │
                   │  • Web Audio API       │
                   │     (metronome click)  │
                   │  • requestAnimation    │
                   │     Frame (count UI)   │
                   └────────────────────────┘
```

**Description.** The system is intentionally **serverless and dependency-free at runtime**. No backend, no database, no auth — just a static bundle on Vercel's edge. Two showcase songs ("Friend Like Me", "A Whole New World") are each split into per-phrase MP4 clips. The browser loads the active clip into an HTML5 `<video>` element, drives a Web Audio metronome that re-arms only when the user hits play, and renders the live count overlay via `requestAnimationFrame`. The same bundle is mirrored to a `pplx.app` preview asset for in-thread review, so the production URL and the chat preview never drift apart.

---

### 2.3 Agentic AI Framework

**Definition.** The behavioural contract the agent follows: how it decomposes work, chooses tools, validates results, and decides when to ask the human versus when to act.

**Visual.**

```
        ┌───────────────────────────────────────────────┐
        │              OBSERVE  (read prompt)           │
        └──────────────────────┬────────────────────────┘
                               ▼
        ┌───────────────────────────────────────────────┐
        │   PLAN     • build todo list                  │
        │            • load relevant skills             │
        │            • pick tools                       │
        └──────────────────────┬────────────────────────┘
                               ▼
        ┌───────────────────────────────────────────────┐
        │   ACT      • edit files                       │
        │            • run ffmpeg / Playwright          │
        │            • deploy to Vercel                 │
        └──────────────────────┬────────────────────────┘
                               ▼
        ┌───────────────────────────────────────────────┐
        │   VERIFY   • screenshot at 1280×900           │
        │            • curl prod URL → 200              │
        │            • grep deployed HTML for diff      │
        └──────────────────────┬────────────────────────┘
                               ▼
        ┌───────────────────────────────────────────────┐
        │   REPORT   • mark todos complete              │
        │            • send concise summary to user     │
        └───────────────────────────────────────────────┘
                  ▲                          │
                  └──────── feedback ────────┘
```

**Description.** Every iteration of *Showcase Practice* followed the same loop: read the prompt → revise the todo list → make the smallest correct code change → QA visually via Playwright at desktop width → deploy to Vercel → curl the prod URL to verify the change is actually live → reply with a short summary. This framework is why prompts as small as "rename LOOP to REPEAT" went out cleanly, and why broader prompts (cut announcer audio, sync metronome to play) didn't snowball into regressions.

---

### 2.4 Project Structure

**Definition.** The on-disk layout of source files, assets, and config — i.e. where every file lives and why.

**Visual.**

```
dance-app/
├── index.html                  ← markup, control rows, video element
├── styles.css                  ← terracotta/cream theme + responsive layout
├── app.js                      ← state machine, metronome, count overlay
├── vercel.json                 ← static-site config for Vercel
├── assets/
│   ├── showcases.js            ← phrase data (timestamps, counts, BPM)
│   └── segments.json           ← legacy segment cache
└── clips/
    ├── friend-like-me/
    │   ├── 00_Full-Showcase.mp4
    │   ├── 01_Intro-Setup.mp4
    │   ├── 02_Phrase1-First-Groove.mp4
    │   └── ... (14 phrase clips)
    └── whole-new-world/
        ├── 00_Full-Showcase.mp4
        ├── 01_Opening-First-Theme.mp4
        └── ... (11 phrase clips)
```

**Description.** A flat, three-file source tree (HTML/CSS/JS) keeps cognitive overhead low — Julia can spot-edit a label or color without grepping a framework. Phrase metadata lives in `assets/showcases.js` so adding a new song is purely additive: drop a folder of clips into `/clips/<song-id>/`, append a `SHOWCASES[songId]` entry, and the dropdown picks it up automatically. The MP4s sit alongside the bundle so Vercel serves them from the same origin — no CORS, no signed URLs.

---

### 2.5 Architecture Pattern

**Definition.** The reusable structural idea the codebase embodies — the "shape" of the solution that another engineer would recognize.

**Visual.**

```
       ┌──────────────────────────────────────────────┐
       │              Data (showcases.js)              │
       │    { id, title, subtitle, bpm, phrases[] }    │
       └──────────────────────┬───────────────────────┘
                              │ read once at boot
                              ▼
       ┌──────────────────────────────────────────────┐
       │            Single Source of State             │
       │     state = { showcaseId, phraseIdx,          │
       │       speed, loop, counts, mirror,            │
       │       metroOn, metroVol, isPlaying }          │
       └──────────────────────┬───────────────────────┘
                              │ render(state)
                              ▼
       ┌──────────────────────────────────────────────┐
       │              View (DOM + <video>)             │
       └──────────────────────┬───────────────────────┘
                              │ user events
                              ▼
       ┌──────────────────────────────────────────────┐
       │     Mutations (setSpeed, togglePlay, …)       │
       └──────────────────────┬───────────────────────┘
                              └────── back to state
```

**Description.** *Showcase Practice* uses a lightweight **data-driven, single-state-object** pattern — essentially Flux/Redux thinking without the library. There is one canonical `state` object in `app.js`, a `render()` function that projects it onto the DOM, and a small set of event handlers that mutate state and re-render. This pattern keeps Mirror, Counts, Repeat, Speed, and the Metronome from interfering with each other; each toggle reads and writes one well-named field.

---

### 2.6 Key Architectural Decisions

**Definition.** The deliberate trade-offs made early that shape everything downstream — the choices a reviewer would want documented.

**Visual.**

```
┌──────────────────────────────┬──────────────────────────────────────┐
│ Decision                     │ Why                                   │
├──────────────────────────────┼──────────────────────────────────────┤
│ Static HTML/CSS/JS, no       │ Fastest iteration, zero ops, free    │
│ framework                    │ deploys, easy to hand-edit later     │
├──────────────────────────────┼──────────────────────────────────────┤
│ Pre-cut MP4 clips, not       │ Mobile Safari + low-latency seek;    │
│ runtime seek on full video   │ no JS audio splicing required        │
├──────────────────────────────┼──────────────────────────────────────┤
│ Web Audio metronome,         │ ms-accurate scheduling; ties cleanly │
│ not <audio> tag              │ to the video play/pause lifecycle    │
├──────────────────────────────┼──────────────────────────────────────┤
│ Vercel static hosting,       │ Free, global edge CDN, instant SSL,  │
│ Vercel CLI deploys           │ scriptable from the agent's bash      │
├──────────────────────────────┼──────────────────────────────────────┤
│ Mirror everything to         │ Lets the in-thread preview stay in   │
│ pplx.app preview asset       │ sync with prod for review            │
├──────────────────────────────┼──────────────────────────────────────┤
│ Show time signatures         │ Aligns vocabulary with how dancers    │
│ (4/4, 3/4) in subtitle       │ count music; supports learning        │
└──────────────────────────────┴──────────────────────────────────────┘
```

**Description.** Each decision optimized for **time-to-Julia-on-a-couch-with-her-partner** rather than long-term scalability. A static bundle means the app will still work in five years even if every tool in the stack changes. Pre-cut clips dodge mobile-browser seeking bugs that plague long videos. Web Audio for the click track means the metronome lines up to milliseconds with the music's downbeat. The pplx.app mirror exists so feedback loops happen *inside the chat* rather than in a separate browser tab.

---

### 2.7 Data Flow

**Definition.** The journey a piece of information takes from origin to pixel — what triggers what, and in what order.

**Visual.**

```
   USER ACTION                    STATE MUTATION                  EFFECT ON UI
   ───────────                    ──────────────                  ────────────

   click "Friend Like Me"   ─►   showcaseId = 'friend-like-me'  ─►  reload phrase list,
   in showcase dropdown          phraseIdx = 0                      preload clip 01

   click phrase row "05"    ─►   phraseIdx = 4                  ─►  <video src=...05.mp4>,
                                 isPlaying = false                  show duration + counts

   click  ▶  Play           ─►   isPlaying = true               ─►  video.play(),
                                                                    if metroOn → metro.start()

   click Metronome  On      ─►   metroOn = true                 ─►  arm AudioContext;
                                                                    starts only on next play

   drag volume slider       ─►   metroVol = 0..1                ─►  gain.gain.value = vol

   click ◀◀ / ▶▶            ─►   phraseIdx ±1                   ─►  swap <video src>,
                                                                    auto-play if was playing

   click ↻ Repeat (On)      ─►   loop = true                    ─►  on video 'ended':
                                                                    rewind to 0, play again

   video 'timeupdate'       ─►   currentTime → counter          ─►  count overlay updates
                                                                    via requestAnimationFrame
```

**Description.** Data flow is strictly **unidirectional**: every user gesture mutates one field on the `state` object, `render()` reads that object, and side-effects (video element, AudioContext, DOM) follow. The metronome is the most subtle case — toggling its On/Off button only *arms* it; the actual click track starts when `isPlaying` flips true, so the click is always in phase with the music's downbeat. The count overlay piggybacks on the video's `timeupdate` event for sample-accurate counting against the showcase BPM (99.4 for Friend Like Me, 107.7 for A Whole New World).

---

## 3. Single Prompt to Rebuild This Application

Copy and paste the prompt below into a fresh Perplexity Computer session. Attach the full showcase video(s) you want to break down before sending.

> **Rebuild Prompt**
>
> I'm a competitive ballroom dancer and I need a personal practice web app to learn showcase choreography. I'm attaching the full showcase video(s). For each video, break the choreography into rhythmical phrases aligned to the music's beat structure, cut a per-phrase MP4 clip for every phrase, and build a static HTML/CSS/JS web app called **Showcase Practice** that lets me drill those phrases. Deploy the final result to Vercel.
>
> **App requirements:**
>
> 1. **Showcase picker.** A dropdown at the top to switch between showcases. Each showcase has a title (e.g. "Friend Like Me") and a subtitle in the form `<SONG NAME> · <DANCE STYLE> (<TIME SIGNATURE>)` — for example `FRIEND LIKE ME · SWING (4/4)` and `A WHOLE NEW WORLD · WALTZ (3/4)`.
> 2. **Phrase list.** A right-hand panel listing every phrase with index, name, start–end timestamps, and count length. Clicking a phrase loads its clip but does **not** auto-play.
> 3. **Video player.** Big primary video, with a live "count" overlay (1-2-3-4 for 4/4, 1-2-3 for 3/4) and a phrase length / counts readout.
> 4. **Transport controls.** Previous-phrase, Play/Pause, Next-phrase, and Restart buttons.
> 5. **Two-row control panel below the video:**
>    - **Top row:** Speed (¼×, ½×, ¾×, 1× pills), Counts (On/Off), Metronome (On/Off + volume slider).
>    - **Bottom row:** Repeat (On/Off — auto-rewinds the current phrase), Mirror (On/Off — flips the video horizontally).
> 6. **Metronome.** Built with the Web Audio API, clicks on every beat at the showcase's BPM, locked to the music's downbeat. The Metronome On/Off button only arms it — it should actually start/stop in lockstep with the Play button. Keep the volume slider working independently.
> 7. **Audio cleanup.** Cut any announcer voice-over from the source video (e.g. "<dancer name> <song title>") before slicing into phrases, so each clip starts clean on the music.
> 8. **Theme.** Warm, premium, minimalist — cream background, terracotta accent, sans-serif type. Generous whitespace, large legible numbers, no AI-aesthetic gradients.
> 9. **Footer line.** Show the tempo (BPM), beat length in seconds, and a one-line practice tip (e.g. "Drill in chunks of 2, then chain.").
>
> **Tech constraints:**
>
> - Static HTML/CSS/JS only — no frameworks, no backend, no database.
> - Phrase metadata in `assets/showcases.js` keyed by showcase id.
> - Pre-cut MP4 clips in `clips/<showcase-id>/` (numbered, e.g. `01_Intro-Setup.mp4`).
> - One canonical `state` object in `app.js`; a single `render(state)` function projects state to the DOM.
> - Visual QA via Playwright at 1280×900 before every deploy.
> - Deploy to Vercel and alias the result to `showcase-genie.vercel.app` (or a name I provide).
>
> When you're done, share the live Vercel URL and an inline preview I can review in chat.
