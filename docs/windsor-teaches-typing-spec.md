# Windsor Teaches Typing — Design Spec

**Status:** Complete draft (Sections 1–9) — ready to be handed to Claude as a build spec
**Purpose:** This document is the design/spec sheet for a kids' typing education game. It is intended to eventually be handed to Claude as a build spec to generate the actual application.

---

## 1. Overview & Goals

**Project Name:** Windsor Teaches Typing

**What it is:**
A browser-based typing game designed to teach young children (ages 5–8) how to touch-type, starting from zero keyboard familiarity. Inspired by the pedagogical structure of classic typing tutors (like Mavis Beacon), but built with simple, calm, kid-friendly interactions — favoring clarity and focus over flashy animation or heavy theming.

**Primary goals:**
1. Teach correct finger placement and home-row technique from scratch — no prior keyboard knowledge assumed.
2. Build muscle memory for letter locations before emphasizing speed.
3. Keep kids engaged through light game mechanics (not worksheets) — short sessions, clear feedback, low frustration.
4. Support multiple individual user profiles with separate progress tracking, since skill levels differ per child.
5. Track progress over time so kids (and parents) can see improvement.

**Design philosophy:**
- **No theme, no mascot, no story wrapper.** Clean visuals, minimal distraction. The "fun" comes from clear progress, achievable challenges, and light game-like feedback (not cartoon characters or sound effects layered everywhere).
- Calm > stimulating. This is a focus tool, not a reward-loop app.

**Profiles:**
- Each child has their own profile with saved progress, current lesson/level, and stats.
- Profile selection happens at app launch (simple picker, no login/password needed).

**Session length targets:**
- Younger child (~5yo): 5–10 minutes per session.
- Older child (~8yo): 20–30 minutes per session.
- This implies the app should support natural short stopping points (younger) and longer sustained modes (older) — pacing and mode design should account for this per-profile.

**Permanent constraints (architectural, not just v1 scope):**
- **Never multiplayer.** Single-user, local play only — no shared/competitive sessions, ever.
- **Never cloud-connected.** No accounts, no network calls, no external data transmission of any kind. All data (profiles, progress, stats) lives entirely on the local device (e.g. browser localStorage/IndexedDB). This is a hard privacy/simplicity boundary, not a "maybe later" item.

**Success looks like:**
- Kids want to play without being told to.
- Both kids learn home-row finger placement and can type simple words without looking at the keyboard.
- Sessions feel focused and calm, not overstimulating or frustrating.

---

## 2. Target Users

**Profile A — Age 5, "Windsor" (younger)**
- Basic reading level: can sound out short, simple words (CVC words like "cat," "dog," "sun"); not yet fluent with longer or complex words.
- No keyboard experience — doesn't yet know where letters are located.
- Attention span: short sessions (5–10 min), needs frequent small wins to stay engaged.
- Design implications:
  - Early lessons should lean on **letter/key recognition** before full words — single-key matching, not sentence-level typing.
  - Word-based exercises should use only simple, decodable words appropriate for an early reader (short vowel sounds, common sight words).
  - Instructions should be simple enough to be understood with minimal or no reading (icons, single-word prompts).
  - Very forgiving of mistakes — no failure states, just gentle correction and retry.

**Profile B — Age 8 (older child)**
- Reading fluently — no reading-level constraints on word/sentence content.
- Some prior typing exposure but currently relies on hunt-and-peck — meaning he has **bad habits to unlearn**, not just a blank slate. This is different from starting from zero.
- Longer attention span (20–30 min sessions) — can handle more structured lesson progression and longer-form exercises (sentences, short paragraphs later on).
- Design implications:
  - Lessons should explicitly enforce **home-row discipline** early (visual finger-to-key guidance), since he'll be tempted to fall back into hunt-and-peck patterns.
  - Can progress faster through basic letter-recognition stages and spend more time in speed/accuracy building once fundamentals are solid.
  - Can tolerate slightly more challenge/friction than the 5-year-old before frustration sets in.

**Shared starting point:**
- Both profiles start at Lesson 1 (Level 0) regardless of prior exposure — no placement test, no skill-skipping. This keeps things simple and ensures fundamentals (like home-row hand position) are never skipped, which matters even for the 8-year-old given his hunt-and-peck habit.

**Known limitation:**
- A browser app cannot physically detect which finger a child uses to press a key — only which key was pressed. The app can *teach and display* correct finger-to-key mapping (via the hand/keyboard diagram) and *enforce correct key presses*, but cannot enforce or verify actual finger usage. This is a known constraint, not an assumed feature.

---

## 3. Core Learning Progression

**Structure:** Shared level sequence for all profiles (no separate "kid tracks") — every user progresses through the same levels in order, at their own pace. Faster/more experienced typists (like the 8-year-old) can move through early levels quickly rather than being forced to linger.

**Proposed level sequence** (mirroring classic typing-tutor structure):

| Level | Focus | Example Content |
|---|---|---|
| **Level 0** | Keyboard orientation — no typing yet | Meet the keyboard: identify hands, fingers, and the home row keys (F/J bumps). Visual-only intro, maybe a "find the key" pointing exercise. |
| **Level 1** | Home row keys | `A S D F` / `J K L ;` — single key presses, correct finger per key shown on hand diagram. |
| **Level 2** | Home row words | Simple words using only home-row letters (e.g. "dad," "sad," "flask" — as available). |
| **Level 3** | Top row keys | `Q W E R T` / `Y U I O P` — introduced with finger mapping, drills mixing with home row. |
| **Level 4** | Top row + home row words | Words combining both rows. |
| **Level 5** | Bottom row keys | `Z X C V B` / `N M , . /` |
| **Level 6** | Full alphabet words | Common words using all rows. |
| **Level 7** | Numbers row | `1–0` across the top. |
| **Level 8** | Basic punctuation | Period, comma, apostrophe, question mark. |
| **Level 9** | Short sentences | Combining everything — simple sentence typing. |
| **Level 10** | Speed & accuracy building | Timed exercises, WPM/accuracy tracking, review of weak spots. |

*(This is a first-pass skeleton — exact level count/boundaries may be refined during build-out.)*

**Progression rules:**
- Levels unlock sequentially — must complete Level N to unlock Level N+1 (no skipping ahead).
- "Complete" = meets some accuracy/consistency threshold, not just "attempted once" (exact criteria to be defined in Section 6 — Feedback & Motivation Systems).
- A capable/fast typist (like the 8-year-old) can move through early levels in a single sitting since there's no artificial time-gating — just performance-gating.
- No separate difficulty curve for younger vs. older profiles — the *level content* is identical; what differs is pacing and how long a given level takes to master.

**Hand/keyboard diagram:**
- A persistent on-screen hand/keyboard diagram shows which finger should press which key.
- Persistent (always visible) during **Levels 0 through ~6**, while row-learning is actively happening.
- May fade, shrink, or become toggle-able in later levels once a profile demonstrates proficiency, so it teaches early without permanently cluttering the screen.

---

## 4. Game Modes / Features

**Structure:** Each level pairs its key/word content with one or more **mini-modes** — the same underlying content (e.g. "home row letters" or "Level 3 words") can be practiced through different game mechanics, so kids get variety without the curriculum itself branching. Every mini-mode (except Drill Mode, which is untimed/instructional) has a **local high-score variant** tracked per profile.

**Core mini-modes (v1):**

| Mode | Mechanic | High-score variant |
|---|---|---|
| **Drill Mode** | Straightforward prompt-and-type: a key/word is shown, hand diagram highlights correct finger, child types it, gets immediate feedback. No time pressure. | N/A — untimed, instructional only, no scoring. |
| **Falling Letters** | Letters/words fall from the top of the screen; typing the correct key/word clears it before it reaches the bottom. Speed scales with level/profile performance. | Score combines speed, accuracy, and streak; local high score tracked per profile per level. |
| **Keepy Uppy** | A ball/balloon stays aloft as long as correct keys/words are typed in rhythm; a wrong key or pause lets it drift down gently (not "pop"/fail). | Score = longest streak / longest time kept aloft; local high score tracked per profile per level. |
| **Racing** | Abstract, non-mascot visual: a progress bar, or a timed maze-completion path where correct keystrokes move a marker forward. No characters/vehicles. | Score = completion time and/or accuracy; local best-time tracked per profile per level. |

**Mode selection:**
- After Drill Mode introduces new content, the child chooses which mini-mode to play next from a simple selection screen.
- **Fully navigable two ways:**
  - **Mouse/click** — click a mode tile to select it.
  - **Keyboard** — arrow keys to move focus between mode options, Enter to select. Keeps interaction consistent with the app's core purpose (typing/keyboard fluency).
- Selection screen should be visually simple — a small number of clearly labeled tiles/cards (icons + short text label, no reading-heavy descriptions given the 5-year-old's reading level).

**Mode-to-level mapping (proposed):**
- **Levels 0–1 (orientation, home row keys):** Drill Mode only — no scoring pressure while fundamentals form.
- **Levels 2–6 (words, additional rows):** Drill Mode (intro) → Falling Letters and/or Keepy Uppy (reinforcement + high scores).
- **Levels 6–9 (numbers, punctuation, sentences):** Drill Mode → Falling Letters / Keepy Uppy → Racing, all with high scores available.
- **Level 10 (speed & accuracy):** Racing + Keepy Uppy + Falling Letters all active with high scores as the primary focus — the "polish and push your limits" level.

**High score system — shared rules:**
- All high scores are **local to the device and specific to each profile** (consistent with the no-cloud, no-multiplayer constraints from Section 1) — no cross-profile leaderboard, no comparison between siblings built into the UI.
- High scores are tracked **per mode, per level** (e.g. "Level 3 Falling Letters best score," "Level 5 Keepy Uppy longest streak") so kids can see improvement on specific skills over time, not just one global number.
- Score displays should stay simple — no elaborate animations or fanfare on a new high score, just a clean, positive acknowledgment.

**Volume control:**
- **Single global setting** for the whole app — not per-profile. Applies regardless of which profile is active.
- Persistent, always-visible control (not buried in a menu) — e.g. a small icon in a fixed corner of the screen.
- Supports mute/unmute and a basic volume level (simple +/- or a few discrete steps — no need for a fine-grained slider).
- Setting persists across sessions (saved locally, survives app close/reopen).
- Sized and simplified for independent use by a 5-year-old — large click target, clear icon.

**Audio design:**
- Minimal, non-intrusive sound layer (consistent with "calm > stimulating"):
  - **Correct key/word:** short, pleasant, positive tone.
  - **Missed/incorrect key or word:** subtle, soft, non-jarring — should never feel like a "buzzer" or punishment cue.
  - **Round/level complete:** brief, warm confirmation sound.
  - **"Game over" / round end** (e.g. Falling Letters, Keepy Uppy ending): gentle, forgiving tone — not a failure sting.
- All audio controlled by the single global volume/mute control described above.

**Design constraints carried over from Section 1 (calm > stimulating):**
- No mascots, no story wrapper.
- Losing/mistakes never punishing — visual and audio feedback both stay soft on misses, including in high-score modes (a missed letter costs score/streak, but never ends things abruptly or harshly).
- Each mode playable in short, natural bursts to fit both session-length targets.

---

## 5. UI/UX & Visual Style

**Color palette (proposed):**
A soft, muted palette that stays calm and readable without being babyish or high-stimulation.

- **Background:** soft off-white / very light neutral gray (not stark white — easier on eyes for longer sessions).
- **Primary accent:** a muted, mid-tone blue (calm, focus-friendly, widely legible).
- **Success/correct feedback:** soft sage green (not neon/bright green — stays gentle).
- **Miss/incorrect feedback:** a muted warm amber or soft coral — **not red**, since red often reads as "alarm" or "danger" to kids and we want misses to feel low-stakes.
- **Highlight/active key:** a clear but soft accent color (e.g. a warm yellow-gold) — needs enough contrast to be findable at a glance on the hand/keyboard diagram, since that's functional, not decorative.
- Overall: 4–5 colors max in the working palette, used consistently and purposefully (color = meaning, not decoration).

**Font:**
- A "kid-learning" style font for body text and prompts — a rounded, simple sans-serif designed for early readers (candidates to evaluate at build time: Lexend, Sassoon, or similar "easy reading" typefaces).
- Large type size throughout, especially for the 5-year-old's content — comfortably readable from normal laptop viewing distance without leaning in.
- Avoid decorative/playful "kiddie" fonts (e.g. Comic Sans-style novelty fonts) — legibility and letter clarity matter more than cuteness, especially since kids need to clearly distinguish similar-looking letters (b/d, p/q, etc.) while learning.

**Device/display targets:**
- **Laptop and desktop only** — no tablet/touch support needed.
  - No touch-target sizing concerns, no on-screen keyboard conflicts, no responsive-for-touch layout needed.
  - Can safely assume a physical keyboard is always present and a mouse/trackpad for menu navigation.
  - Should still be reasonably responsive to different screen sizes/resolutions (laptop vs. external monitor), but not to touch breakpoints.

**Layout principles:**
- **Persistent hand/keyboard diagram** stays visible and prominent during Levels 0–~6, positioned so it doesn't compete with the main exercise area for attention — likely bottom or side of screen, main content/exercise centered and largest.
- **Minimal on-screen elements at any time** — avoid cluttering the screen with score, timer, streak, hand diagram, and prompt all fighting for attention simultaneously. Priority order: current prompt/content > hand diagram (when active) > score/feedback (small, unobtrusive) > volume control (small, corner, always present).
- **Generous whitespace** — calm, uncluttered screens throughout.
- **Consistent navigation** — same visual language for menus/mode-selection across the whole app.

**Accessibility notes:**
- Color choices are chosen with colorblind-friendliness in mind (avoiding red/green as the sole correct/incorrect signal — pairing color with shape/icon/motion too, e.g. a checkmark vs. an X shape, not just color change).
- No accessibility requirements around touch/mobile.
- Open item: flag any specific visual or motor considerations for either child if they arise; otherwise standard young-child defaults apply.

---

## 6. Feedback & Motivation Systems

**Level completion criteria:**
- Completing a level = **finishing the allotted content** for that level (the set number of letters, words, or time allotted) — not gated behind a minimum accuracy/speed threshold. Keeps progression low-pressure and avoids repeated "failure" on a level.
- While completing a level, the app tracks (but doesn't gate on):
  - **Accuracy** — % of correct keystrokes/words.
  - **Correct letters/words in a row** — a streak metric, scaled to the level's unit (letters early, words later).
  - **Speed** — WPM or equivalent, more relevant as levels progress toward Level 10.
- These three metrics are shown as a **summary/scorecard after each level or exercise session** (e.g. "Accuracy: 92% · Best streak: 14 · Speed: 18 WPM") — shown after the activity, not during, so it doesn't distract in the moment.
- Distinct from per-mode high scores (Section 4): the scorecard reflects *this level's core content mastery*; high scores reflect *performance in the mini-games*.

**Progress map:**
- The **level-selection screen** doubles as a progress map — each level shown as a button/tile in sequence.
- Each level tile has a distinct visual state:
  - **Not started** — locked/greyed out (can't be started out of order, per Section 3).
  - **In progress / unlocked but incomplete** — visually "active" (accent color, subtle in-progress indicator).
  - **Completed** — clearly marked as done (filled in, checkmark, or distinct completed color).
- This becomes the natural "home base" screen between sessions — a visible sense of a path/journey without needing badges or a story wrapper.

**Motivation systems — kept intentionally minimal:**
- No stars, badges, day-streak counters, or other gamification layers beyond what's already specified (high scores per mini-mode, the scorecard, the progress map).
- The combination of the visible progress map + honest performance metrics + local high scores is treated as sufficient motivation — consistent with "calm > stimulating." Deliberately avoids reward-loop/dopamine-hook patterns.

**Parent visibility:**
- **None.** No parent-facing dashboard or summary view — kept out of scope entirely. Parents can observe sessions directly if desired; no separate reporting UI.

---

## 7. Technical Architecture

**Stack:**
- **React** for the UI/component architecture — a good fit for the stateful nature of the app (profiles, level progress, in-session game state).
- **Vanilla CSS or lightweight styling** (e.g. CSS Modules) — no heavy design system needed given the minimal visual style from Section 5.
- **Browser target:** modern Chromium-based browsers only (Chrome, Edge, etc.) — no legacy support, no polyfills needed.

**Hosting/distribution:**
- **Build once, serve locally via a lightweight static server** (e.g. `serve` or similar), pointed at the production build output, accessed via `localhost`.
- This avoids running a dev server day-to-day and sidesteps any `file://` protocol quirks with localStorage — most reliable option for repeated home use.
- No deployment to the internet — stays entirely on the local machine/network, consistent with the no-cloud constraint.

**Data & persistence:**
- All data — profiles, level progress, high scores, volume setting — persists via **browser localStorage** (or IndexedDB if curriculum/data complexity grows beyond what localStorage handles gracefully — localStorage should comfortably handle profile/progress/score data for two kids).
- **No network calls of any kind**, consistent with the permanent no-cloud constraint from Section 1. Audio files and curriculum/word-list data are bundled locally with the app, not fetched remotely.
- Data model (proposed, high-level):
  - `profiles`: list of profiles, each with `id`, `name`, `currentLevel`, `levelProgress` (per-level completion status + best scorecard metrics).
  - `highScores`: per profile, per level, per mode (Falling Letters / Keepy Uppy / Racing) — best score/streak/time.
  - `settings`: global (not per-profile) — currently just volume/mute.

**Component structure (proposed, high-level):**
- `ProfilePicker` — launch screen, select/create a profile.
- `LevelMap` — progress map / level-selection screen (Section 6), shows level tiles with not-started/in-progress/completed states.
- `ModeSelect` — mini-mode picker after Drill Mode, mouse + keyboard navigable (Section 4).
- `DrillMode`, `FallingLetters`, `KeepyUppy`, `Racing` — the four mini-mode game components, each consuming the same underlying level content but rendering different mechanics.
- `HandKeyboardDiagram` — persistent/toggle-able component shown during early levels (Section 3), reusable across Drill Mode and other modes.
- `Scorecard` — post-level summary (accuracy/streak/speed) shown after level completion (Section 6).
- `VolumeControl` — persistent global control, rendered at the app shell level so it's present everywhere (Section 4).

**No backend:**
- Fully static, client-side app — no server component. Fits naturally with the no-cloud/no-accounts constraint and keeps things simple to run and maintain.

---

## 8. Content & Curriculum Data

**Approach:** Define the **structure/format** here; actual word lists/content to be generated at build time (likely by Claude, using this spec as the guide) rather than drafted now.

**Content difficulty philosophy:**
- Words and sentence content should be **age-appropriate but lean slightly ahead** of grade level — roughly a couple of years beyond each child's current age/reading level — to provide some genuine challenge rather than feeling too easy or babyish.
- This applies to vocabulary/reading difficulty specifically, not to typing mechanics — the *typing* difficulty curve still follows the level progression (home row → full keyboard) regardless of word complexity. In other words: words should be a bit more advanced to read, even while the keys being practiced stay simple early on.
- Since reading level and typing level aren't the same axis, content per level should be chosen so it's typeable using only the keys introduced by that level, while still skewing toward richer vocabulary where the level's key-set allows it (e.g. Level 2 home-row words are inherently limited by the constrained letter set, but Level 6+ full-alphabet content has more room to lean into slightly-ahead vocabulary).

**Proposed content structure (per level):**
- Each level defines:
  - `keysIntroduced`: the new key(s) added at this level (empty for word-only reinforcement levels).
  - `allowedKeys`: full set of keys available for this level's content (cumulative from prior levels + new ones).
  - `content`: an ordered or randomizable list of typeable items — individual letters (early levels), words (mid levels), or short sentences (late levels) — constrained to `allowedKeys`.
  - `contentLength` / `allotment`: the amount of content that constitutes "completing" the level (per Section 6) — e.g. a target count of letters/words, or a time-based allotment for later levels.
- Content lists should be **large enough to avoid obvious repetition** within a single session (especially important for the high-score mini-modes, where a kid might play the same level's content multiple times in a row chasing a better score).

**Storage:**
- Curriculum data (key sets, word lists, sentence lists) stored as **static local data** (e.g. JSON or JS data files) bundled with the app — no external fetching, consistent with the no-cloud constraint from Section 1.
- Structured per-level so it's easy to extend/edit later (e.g. adding more words to Level 6 without touching other levels).

**Word/content sourcing notes for build time:**
- No specific external curriculum (e.g. a particular school reading list) to match against — general age-appropriate-but-slightly-challenging judgment is sufficient.
- Content should avoid anything requiring keys not yet introduced at that level (hard constraint — e.g. no apostrophes in contractions until Level 8 introduces punctuation).

---

## 9. Out of Scope / Future Ideas

**Explicitly out of scope (permanent — architectural, not just v1):**
- Multiplayer, in any form (Section 1).
- Cloud sync, accounts, or any network calls (Section 1).

**Explicitly out of scope (v1, could revisit):**
- Tablet/touch support — laptop/desktop only (Section 5).
- Parent-facing dashboard or progress reporting (Section 6).
- Badges, stars, day-streak counters, or other gamification beyond high scores/scorecard/progress map (Section 6).
- Placement tests or skill-skipping — both profiles always start at Level 0 (Section 2/3).

**Future ideas (not committed, worth remembering):**
- Additional mini-modes beyond the initial four (Drill, Falling Letters, Keepy Uppy, Racing), if the kids want more variety down the line.
- Expanding beyond Level 10 (e.g. more advanced sentence/paragraph typing, harder speed/accuracy challenges) once both kids are comfortably through the current curriculum.
- Editable/custom word lists, if content ever needs tuning to match something specific (e.g. sight words from school).
