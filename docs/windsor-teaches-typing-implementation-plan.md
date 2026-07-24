# Windsor Teaches Typing — Implementation Plan

**Status:** Approved — ready for build
**Companion doc:** [windsor-teaches-typing-spec.md](./windsor-teaches-typing-spec.md)
**Purpose:** Breaks the design spec down into a build-ordered sequence of phases. Check off phases as they're completed.

---

## Key decisions

Locked in before build start:

| Decision | Choice | Rationale |
|---|---|---|
| **Navigation** | Internal React state machine (no router library) | Zero extra dependencies; app is local/single-device with no need for deep-linking or browser history. |
| **Font** | [Lexend](https://fonts.google.com/specimen/Lexend) | Free, open-license, designed to improve reading proficiency; self-hostable (no network calls needed at runtime). |
| **Audio** | Synthesized at runtime via Web Audio API | No audio assets to source or license, nothing to bundle, zero network calls by construction. |
| **Build order** | Vertical slice first (see phases below) | Gets one profile through a full loop (pick → level map → drill → scorecard) playable early, before investing in the three extra mini-games. |

---

## Build sequence

Phases are ordered by dependency — each phase generally assumes the ones above it are done. Work top to bottom.

- [ ] **Phase 1 — Foundation**
  - [ ] Storage layer: `localStorage` wrapper module with schema versioning and typed get/set helpers for `profiles`, `highScores`, `settings`
  - [ ] Core data model: profile shape, per-level progress shape, high-score shape
  - [ ] App shell: internal state machine for screen navigation (ProfilePicker → LevelMap → ModeSelect → game mode → Scorecard → back to LevelMap)
  - [ ] `VolumeControl` mounted persistently at the app-shell level

- [ ] **Phase 2 — Profiles**
  - [ ] `ProfilePicker` screen: list existing profiles
  - [ ] Create-new-profile flow (name only, no login/password)
  - [ ] Profile select → enters app at their current level

- [ ] **Phase 3 — Curriculum data**
  - [ ] Define level data structure: `keysIntroduced`, `allowedKeys`, `content`, `contentLength`
  - [ ] Author content for Levels 0–10 (per spec §3, §8), level by level:
    - [ ] Level 0 — keyboard orientation (no typing)
    - [ ] Level 1 — home row keys
    - [ ] Level 2 — home row words
    - [ ] Level 3 — top row keys
    - [ ] Level 4 — top row + home row words
    - [ ] Level 5 — bottom row keys
    - [ ] Level 6 — full alphabet words
    - [ ] Level 7 — numbers row
    - [ ] Level 8 — basic punctuation
    - [ ] Level 9 — short sentences
    - [ ] Level 10 — speed & accuracy building
  - [ ] Verify each level's content only uses that level's `allowedKeys`
  - [ ] Verify content lists are large enough to avoid obvious repetition in a session

- [ ] **Phase 4 — LevelMap**
  - [ ] Progress-map/level-select screen with level tiles
  - [ ] Tile states: not-started (locked), in-progress (unlocked, active), completed
  - [ ] Enforce strictly sequential unlock (complete N to unlock N+1)

- [ ] **Phase 5 — HandKeyboardDiagram**
  - [ ] Keyboard layout + finger-to-key mapping data
  - [ ] Diagram component with "active key" highlight state
  - [ ] Persistent display through ~Level 6; toggle-able after
  - [ ] Reusable across Drill Mode and other modes

- [ ] **Phase 6 — Drill Mode**
  - [ ] Prompt-and-type loop: show item → highlight finger/key on diagram → listen for keydown → immediate feedback
  - [ ] Feedback uses color + shape/icon (not color alone) — colorblind-friendly, non-punishing on misses
  - [ ] Track accuracy, streak, and speed (WPM) for the Scorecard
  - [ ] No scoring/high-score (untimed, instructional only)

- [ ] **Phase 7 — Scorecard**
  - [ ] Post-drill/post-level summary: accuracy, best streak, speed
  - [ ] Distinct from mini-mode high scores (this reflects level-content mastery)

- [ ] **Phase 8 — ModeSelect**
  - [ ] Tile-based mode picker
  - [ ] Mouse/click navigation
  - [ ] Keyboard navigation (arrow keys to move focus, Enter to select)
  - [ ] Filter available modes by level, per the mode-to-level mapping (spec §4)

- [ ] **Phase 9 — Mini-games**
  - [ ] **Falling Letters** — letters/words fall, correct keystroke clears before reaching bottom; speed scales with level; score = speed + accuracy + streak
  - [ ] **Keepy Uppy** — stays aloft while typed correctly in rhythm; miss/pause = gentle drift down, no fail state; score = longest streak/time aloft
  - [ ] **Racing** — abstract progress bar or maze path, correct keystrokes move marker forward; score = completion time/accuracy
  - [ ] Per-profile, per-level, per-mode high-score persistence for all three

- [ ] **Phase 10 — Audio + VolumeControl**
  - [ ] Audio manager: synthesized tones (correct, incorrect, level-complete, round-end) via Web Audio API
  - [ ] Gate all playback on the single global volume/mute setting
  - [ ] Persist volume/mute setting in `settings` (global, not per-profile)

- [ ] **Phase 11 — Visual system**
  - [ ] Palette as CSS custom properties (soft off-white background, muted blue accent, sage-green success, amber/coral miss, warm yellow-gold key highlight)
  - [ ] Lexend type system, large comfortable sizes
  - [ ] Shared primitives: Tile/Card, Button, correct/incorrect feedback icon

- [ ] **Phase 12 — Polish pass**
  - [ ] Full click-through with both profile personas' pacing in mind (5–10 min and 20–30 min sessions)
  - [ ] Keyboard-navigation accessibility check on ModeSelect
  - [ ] Confirm production build (`vite build`) serves correctly from `localhost`
  - [ ] Confirm zero network calls anywhere in the app

---

## Out of scope (carried over from spec §9)

Not part of this plan — see spec for full detail:

- Multiplayer or any cloud/network connectivity (permanent constraints)
- Tablet/touch support
- Parent-facing dashboard
- Badges, stars, day-streak counters
- Placement tests / skill-skipping
