# Setup & Workflow (Codespaces + Claude Code)

How to stand the project up and how to drive Claude Code effectively against it.

---

## 1. Repo bootstrap

1. Create a new GitHub repo (public or private), add a README.
2. Drop `ROADMAP.md`, `ARCHITECTURE.md`, and this `SETUP.md` in the root and commit.
3. Open the repo in a **Codespace** (Code ▸ Codespaces ▸ Create).

## 2. Devcontainer

Add `.devcontainer/devcontainer.json` so every Codespace is identical and Claude Code is available:

```jsonc
{
  "name": "snake",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:latest",
  "features": {
    "ghcr.io/anthropics/devcontainer-features/claude-code:latest": {}
  },
  "forwardPorts": [5173],
  "postCreateCommand": "npm install"
}
```

> Port 5173 is Vite's default dev server. If the Claude Code feature path differs by the time you
> build, install it per the current Claude Code docs instead — the rest is unaffected.

## 3. Project scaffold

Inside the Codespace terminal:

```bash
npm create vite@latest . -- --template vanilla-ts
npm install
npm run dev   # serves on 5173; Codespaces will offer to open it
```

Vanilla-ts (not a framework) keeps the core framework-agnostic, matching the architecture. A UI
framework can wrap the menus later without touching the core.

## 4. Suggested source layout

Mirror the layering from `ARCHITECTURE.md` so dependencies point inward:

```
src/
  core/            # pure simulation — no DOM, no canvas, no audio, no Math.random
    rng.ts         #   seeded RNG
    events.ts      #   semantic event emitter
    game.ts        #   simulation: state, step(), rules
    types.ts       #   entities, intents, config
  adapters/
    render/        #   dumb canvas renderer + VisualTheme
    input/         #   keyboard source -> intent stream (recordable)
  services/
    score/         #   ScoreRepository interface + LocalStorage impl
    sound/         #   SoundPack interface (seam only for MVP)
  themes/          #   default Game-Boy-inspired VisualTheme
  main.ts          #   wiring: build core, attach adapters/services, start loop
```

## 5. Issues & board

- Turn each `ROADMAP.md` checkbox (or finer) into a **GitHub Issue**.
- Label by phase (`phase-0` … `phase-8`) and type (`core`, `render`, `infra`, …).
- Create a **Project board** with a column per phase (or simple Todo/Doing/Done).
- Reference issue numbers in commits/PRs (`closes #12`) so the board self-updates.

---

## 6. Driving Claude Code

### Session ritual
Start each session by pointing Claude Code at the docs so it shares your context:

> "Read `ARCHITECTURE.md` and `ROADMAP.md`. We're working on issue #N. Respect the core/adapters/
> services layering and the determinism rules. Propose a plan before writing code."

### Principles that keep the architecture intact
- **One issue at a time.** Small, reviewable diffs beat big-bang generation.
- **Guard the core.** If a change makes the core import canvas/DOM/audio or call `Math.random()`,
  reject it — that's the one rule that protects every future feature.
- **Plan, then code.** Ask for the approach first; approve it; then let it implement.
- **Tests where they earn their keep.** The core is pure, so the RNG, food-spawn-on-free-tiles
  logic, and collision rules are trivially unit-testable and worth covering early. Skip UI tests
  for now.
- **Commit per issue**, referencing the number, and update the `ROADMAP.md` checkbox in the same PR.

### A good first prompt (Phase 0 → 1)
> "Set up the source layout from `SETUP.md`. Implement the seeded RNG and a fixed-timestep loop
> with unit tests, then a minimal grid-agnostic simulation that moves the snake and spawns food on
> a uniformly random free tile. No rendering yet. Keep the core free of DOM/canvas/audio and never
> use `Math.random()`."

---

## 7. Deployment (when there's something to show)

The MVP is fully static (`npm run build` → `dist/`). Any static host works — GitHub Pages, Netlify,
or Vercel. Wire this up once Phase 1 is playable; revisit when the Phase 7 backend arrives (the
static frontend stays, an API joins it).
