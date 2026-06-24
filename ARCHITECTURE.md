# Architecture

> The guiding principle, stated once: **the core is a pure, deterministic, observable simulation.**
> Everything else — rendering, input, sound, persistence, multiplayer — plugs into it through
> interfaces or subscribes to its event stream. The core never imports rendering, audio, the DOM,
> or `Math.random()`. If you remember nothing else, remember that sentence; every decision below
> is just an application of it.

This document is the source of truth for *why* the code is shaped the way it is. Read it at the
start of any work session (Claude Code included) before changing core logic.

---

## 1. Stack

| Concern        | Choice                          | Reason |
|----------------|---------------------------------|--------|
| Language       | TypeScript                      | Types make the seams explicit and self-documenting. |
| Build/dev      | Vite                            | Instant HMR, zero-config TS, trivial static build. |
| Rendering      | Canvas 2D                       | Right tool for a grid game; renderer is swappable. |
| Game loop      | Fixed timestep                  | Deterministic stepping, decoupled from frame rate. |
| Structure      | Entity/system separation        | Core state & logic isolated from rendering & input. |

---

## 2. The non-negotiable disciplines

These are cheap to hold now and brutally expensive to retrofit. They are not optional.

### 2.1 Determinism
The entire simulation is a pure function of **(initial seed, ordered input events)**. Run the same
seed with the same inputs and you get a byte-identical game, every time.

- **All randomness flows through a single seeded RNG.** Never call `Math.random()` anywhere in the
  core. Food placement, and later power-up/obstacle placement, all draw from the seeded RNG.
- The game loop is **fixed timestep** so simulation steps are reproducible regardless of frame rate.

**Why it pays off:** replays, ghost races, async online multiplayer, weekly challenges, and
leaderboard anti-cheat are *all the same feature* — record the input stream against a seed, replay
or re-simulate it. Four-plus backlog items collapse into one discipline.

### 2.2 Input as a recordable stream
Input is not read directly by the core. An input layer translates raw events (keys, later touch)
into semantic intents (`turn_up`, `turn_left`, …) that are fed into the simulation as a timestamped,
ordered stream. That stream is the thing we can record and replay.

### 2.3 Semantic event emission
The core **emits** semantic events as they happen — `food_eaten`, `turn`, `death`, `game_over`
(more later: `powerup_collected`, `level_up`, …). Nothing is required to listen. Sound, achievements,
scoring effects, and analytics all *subscribe* later without the core changing.

---

## 3. The seams (interfaces designed now, implementations grow later)

Each seam exists so a future feature is a **drop-in**, not a rewrite. MVP ships the simplest
implementation of each; the interface is the contract that never breaks.

### 3.1 Rendering — the "dumb renderer"
The renderer holds no opinions about appearance. For each entity it asks the active **VisualTheme**
what to draw and blits the result. Swapping themes, sprites, or even the whole renderer touches no
core logic.

### 3.2 VisualTheme (visuals only)
A named set of design tokens:
- palette (Game Boy *inspired* default — **not** limited to 4 colours)
- grid dimensions (board size lives here / in config, **never** hardcoded in core)
- pixel font
- **sprite/asset map keyed by `(entityType, orientation)`**

The asset map takes orientation from day one (head facing N/S/E/W, straight vs. corner body pieces),
even though the MVP default theme draws simple shapes and ignores orientation. Cheap seam now.
A theme can later be flat colours, pixel-art sprites, or emoji — the renderer never changes.

### 3.3 SoundPack (audio only) — decoupled from visuals
A separate, independently-selectable layer: an **event → sound** map. The active visual theme and
the active sound pack are two independent user settings (Game Boy visuals + any audio, or silent +
any theme). MVP implements the **seam only** and ships silent; the core already emits the events a
sound pack will subscribe to.

### 3.4 ScoreRepository (persistence)
The game only ever knows "save this score" / "get top scores." It does not know where scores live.
- MVP: `LocalStorageScoreRepository` (in-browser).
- Later: `RemoteScoreRepository` hitting an API — a one-line swap at startup, core untouched.
- Because a score can later carry its recorded input stream, this same swap is where **server-side
  replay verification** (anti-cheat) lands.

### 3.5 Input source
Keyboard (arrows **and** WASD) in the MVP, behind the input abstraction. Touch (swipe + on-screen
D-pad) becomes an additional input source later with no core changes — the foundation for mobile.

---

## 4. Layering (dependency direction)

```
        ┌─────────────────────────────────────────────┐
        │                   Core                        │
        │  seeded RNG · simulation · entities · rules   │
        │  emits semantic events · consumes input stream│
        │   (no DOM, no canvas, no audio, no Math.random)│
        └───────────────▲───────────────▲──────────────┘
                        │ subscribes     │ plugs in via interface
        ┌───────────────┴───┐   ┌────────┴───────────┐
        │   Adapters        │   │   Services          │
        │  Renderer(Visual  │   │  SoundPack          │
        │  Theme) · Input   │   │  ScoreRepository    │
        │  sources          │   │  (Achievements…)    │
        └───────────────────┘   └─────────────────────┘
```

Dependencies point **inward**. The core depends on nothing outside itself.

---

## 5. Default MVP gameplay decisions

- **Wall behaviour:** wall = death is the MVP default; wrap-around is a *mode* toggled in settings
  later (the seam: collision rule is configurable, not hardcoded).
- **Food spawning:** uniform random pick from the set of **free tiles** (all tiles minus snake body).
  When free tiles = 0, the board is full → **win** condition handled by the same logic.
- **Field scaling:** core is fully **grid-size agnostic** from day one (logical scaling is then free).
  MVP implements **responsive visual scaling** (crisp pixel scaling to the viewport) because the
  canvas needs it anyway and it's the basis for mobile.

---

## 6. What the MVP deliberately does NOT do

Silent (sound seam only) · single visual theme with simple shapes (no sprite art yet) · no
power-ups/obstacles/levels · no wrap mode · local scores only · keyboard only · no replay UI yet
(though the input stream is already recordable). Each of these is a phase, not a rewrite — see
`ROADMAP.md`.
