# CLAUDE.md

Operating manual for working on this project. **Read this, plus `ARCHITECTURE.md` and `ROADMAP.md`,
at the start of every session before changing code.**

This is a Snake game. North star: *Snake on a Game Boy* — a Game-Boy-*inspired* look (and later
sound), built on a **deterministic, observable core** that grows into themes, power-ups, replays,
leaderboards, and async multiplayer without rewrites.

---

## The one rule that protects everything

> **The core is a pure, deterministic, observable simulation.** Rendering, input, sound,
> persistence, and multiplayer plug into it via interfaces or subscribe to its events.

If a change would make anything in `src/core/`:
- import the DOM, canvas, or audio, **or**
- call `Math.random()`,

**reject it.** That single tripwire is what keeps determinism — and therefore replays, ghost races,
async multiplayer, weekly challenges, and leaderboard anti-cheat — possible later. Full reasoning is
in `ARCHITECTURE.md`; do not relitigate it without updating that doc.

## Non-negotiable disciplines (held from day one)

1. **Determinism** — the sim is a pure function of `(seed, ordered intents)`. All randomness goes
   through the seeded RNG in `src/core/rng.ts`. Never `Math.random()` in core.
2. **Fixed timestep** — simulation steps at a fixed cadence, decoupled from frame rate and rendering.
3. **Input as a recordable stream** — raw keys/touches become semantic intents fed to the core as a
   timestamped, ordered stream. The core consumes intents, never raw events.
4. **Semantic events** — the core *emits* events (`food_eaten`, `turn`, `death`, `game_over`, …).
   Nothing is required to listen. New consumers (sound, achievements) subscribe; core does not change.

## The seams (interfaces stay stable; implementations grow)

- **Renderer** is "dumb": for each entity it asks the active `VisualTheme` what to draw and blits it.
  No colour/size literals in the renderer.
- **VisualTheme** = palette + grid size + font + sprite/asset map keyed by `(entityType, orientation)`.
  Board dimensions live here/in config, **never** hardcoded in core. MVP default theme draws simple
  shapes and ignores orientation, but the interface accepts orientation already.
- **SoundPack** = event→sound map, **independent** from visuals. MVP is seam-only and silent.
- **ScoreRepository** = "save score / get top scores." MVP uses `LocalStorageScoreRepository`; a
  `RemoteScoreRepository` swaps in later with no game-code change.

## Project layout

```
src/
  core/        # pure sim — NO dom/canvas/audio/Math.random
    rng.ts events.ts game.ts types.ts
  adapters/
    render/    # dumb canvas renderer + VisualTheme
    input/     # keyboard -> intent stream (recordable)
  services/
    score/     # ScoreRepository + LocalStorage impl
    sound/     # SoundPack interface (seam only for now)
  themes/      # default Game-Boy-inspired VisualTheme
  main.ts      # wiring: build core, attach adapters/services, start loop
```

## Commands

- `npm run dev` — Vite dev server on port 5173
- `npm run build` — static production build to `dist/`
- `npm test` — Vitest unit tests

## How we work here

- **One issue at a time.** Small, reviewable diffs. Work issues in dependency order.
- **Plan before code.** Propose the approach, get it approved, then implement.
- **Test where it earns its keep.** The pure core (RNG, food-on-free-tiles, collision, events) is
  trivially unit-testable — cover it. Skip UI tests for now.
- **Conventional-ish commits**, and reference the issue: `feat: seeded RNG (closes #N)`.
- **Keep docs in sync.** When an issue ships, tick its box in `ROADMAP.md` in the same PR. If an
  architectural decision changes, update `ARCHITECTURE.md` — never let code silently diverge from it.

## Current status

Phase 0 in progress. Docs and the Phase 0/1 GitHub issues exist. Next work is the devcontainer,
Vite+TS scaffold, and the source layout with seam stubs, then Phase 1 in dependency order toward the
first playable build (the wiring issue). See `ROADMAP.md` for the live checklist.

## What the MVP deliberately omits

Silent (sound seam only), single default theme with simple shapes, no power-ups/obstacles/levels, no
wrap mode, local scores only, keyboard only. Each is a later phase, not a rewrite.
