# Roadmap

Phased vision for the Snake project. High-level here; granular work lives in **GitHub Issues**
grouped onto the **Project board**. This file is the living map — Claude Code should read it at the
start of a session to know where we are and what's next. Check items off as they ship.

**North star:** *Snake on a Game Boy* — a Game-Boy-*inspired* default look and (later) sound, built
on a deterministic, observable core that grows into themes, power-ups, replays, leaderboards, and
async multiplayer without rewrites. See `ARCHITECTURE.md` for the why.

Legend: `[ ]` todo · `[~]` in progress · `[x]` done

---

## Phase 0 — Foundation
*Goal: a repo you can open in Codespaces and run, with the seams stubbed.*

- [x] Codespaces devcontainer (Node LTS, Vite, TypeScript)
- [ ] Vite + TypeScript project scaffolded, runs with `npm run dev`
- [ ] `ROADMAP.md` + `ARCHITECTURE.md` committed
- [ ] GitHub Issues created from this backlog; Project board with phase columns
- [ ] Core module skeleton with seams stubbed (RNG, event emitter, interfaces)

## Phase 1 — MVP: thinnest playable slice
*Goal: a real, playable, deterministic game of Snake. Silent, one theme, keyboard.*

- [ ] Seeded RNG (no `Math.random()` anywhere)
- [ ] Fixed-timestep game loop
- [ ] Grid-size-agnostic core simulation (board size from config)
- [ ] Snake movement + growth
- [ ] Food spawning: uniform pick from free tiles; board-full = win
- [ ] Self-collision death; wall = death (default)
- [ ] Semantic events emitted: `food_eaten`, `turn`, `death`, `game_over` (silent)
- [ ] Input abstraction: arrows **and** WASD → intent stream (recordable)
- [ ] "Dumb" Canvas renderer reading a default Game-Boy-inspired VisualTheme (simple shapes)
- [ ] Responsive crisp visual scaling to viewport
- [ ] `ScoreRepository` interface + `LocalStorageScoreRepository` (score + high score)
- [ ] Game states: ready → playing → game over → restart

## Phase 2 — Game Boy identity & feel
*Goal: it finally feels like a Game Boy.*

- [ ] `SoundPack` implementation wired to the existing event stream
- [ ] Classic Game Boy sound pack (eat, turn, death) — *very important item*
- [ ] Visual theme polish: pixel font, refined palette
- [ ] Optional CRT / scanline toggle
- [ ] Settings menu (sound on/off, theme select, wall vs wrap mode)
- [ ] Pause/resume + start countdown

## Phase 3 — Gameplay depth
*Goal: more than classic Snake.*

- [ ] Wrap-around mode (collision rule already configurable)
- [ ] Progressive difficulty (speed scales with growth/time)
- [ ] Power-ups (new entity types + `powerup_collected` event)
- [ ] Obstacles
- [ ] Other objects to pick up / avoid
- [ ] Level progression

## Phase 4 — Full theming system
*Goal: themes become rich and user-extensible.*

- [ ] Orientation-aware sprite/asset maps (directional head, corner body pieces)
- [ ] Multiple built-in visual themes + multiple sound packs
- [ ] Custom user-supplied themes (palette/sprites)
- [ ] Logical field scaling exposed in settings

## Phase 5 — Replay & the determinism payoff
*Goal: cash in the deterministic core.*

- [ ] Input-stream recording + replay playback
- [ ] Ghost races (race your own recorded run)

## Phase 6 — Mobile
*Goal: plays great on a phone.*

- [ ] Touch input (swipe + on-screen D-pad) as a new input source
- [ ] Responsive layout
- [ ] PWA / installable

## Phase 7 — Backend era *(largest scope jump)*
*Goal: accounts and a real global leaderboard.*

- [ ] Backend + database + auth
- [ ] Accounts / login
- [ ] `RemoteScoreRepository` swapped in (drop-in; core untouched)
- [ ] Global leaderboard with **server-side replay verification** (anti-cheat)
- [ ] Weekly challenges (shared weekly seed — determinism payoff again)
- [ ] Achievements (subscribe to the event stream; can start local-first earlier if wanted)

## Phase 8 — Competitors & multiplayer
*Goal: you're not alone on the board.*

- [ ] AI-controlled competitors (pathfinding bots inside the deterministic sim)
- [ ] Async online multiplayer / shared seeded rooms (tier-2 ambition)

---

### Notes
- Phases are a default order, not a contract. Agile: ship the thinnest valuable slice, then choose
  the next most valuable item — re-order freely as long as the `ARCHITECTURE.md` disciplines hold.
- **Achievements** only need the event stream, so they *can* be pulled earlier as a local-first
  feature; they're parked in Phase 7 because they pair naturally with accounts.
