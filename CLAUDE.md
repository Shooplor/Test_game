# CLAUDE.md

Project guide for building browser games in this repo. Keep it lean: a playable
game beats a perfect document.

## Tech stack

- **Plain HTML, CSS and JavaScript (ES modules).** No engine, no build step by
  default. The game must run by serving the repo root with any static server.
- **2D:** Canvas 2D API. **3D:** Three.js, loaded through an import map from a
  CDN with an exact pinned version (e.g. `three@0.160.0`), never a floating tag.
- Add a dependency or a bundler (Vite) only when there is a concrete need, and
  note why in this file under "Decisions".
- Target: desktop browsers (keyboard + mouse) first. Touch/mobile support is
  decided per game and recorded in `docs/concept.md`.

## Project layout

```
index.html          entry point
src/
  main.js           boot: create renderer, load config, start loop
  config.js         ALL tuning numbers (speeds, damage, timings, colours)
  game/             game logic and state (no DOM, no rendering)
  render/           drawing / Three.js scene code
  input/            keyboard, mouse, touch → abstract actions
  ui/               HUD, menus (DOM overlays)
  audio/            sound (Web Audio API)
assets/             images, models, sounds — lowercase_with_underscores names
docs/               concept.md and short design notes
tests/              logic tests and browser smoke tests
prototypes/<name>/  throwaway experiments, each with a README
```

Grow into this layout as needed; a tiny game can start as fewer files.

## Coding rules

- **Frame-rate independence:** every movement and timer uses delta time
  (seconds) from `requestAnimationFrame`. Clamp delta (e.g. max 0.1 s) so a
  background tab doesn't cause huge jumps. Use a fixed timestep for physics if
  collisions start to misbehave.
- **No magic numbers in gameplay code:** tuning values live in `src/config.js`
  so balancing means editing one file.
- **Separate logic from presentation:** `src/game/` must not touch the DOM,
  canvas or Three.js. Rendering and UI read game state; they don't own it.
  This keeps logic testable without a browser.
- **Input goes through actions:** map keys/buttons to actions (`jump`, `fire`)
  in one place so controls are easy to remap.
- **Explicit state machines** for game flow (menu → playing → paused →
  game over) and for complex entities. List the states and allowed transitions.
- **Clean up Three.js resources** (dispose geometries, materials, textures)
  when removing objects or restarting a level.
- Small, readable modules over clever abstractions. Comment the *why*, not the
  *what*.

## Design docs (keep them short)

`docs/concept.md` — one page, written before the first real build:

1. **Pitch** — one or two sentences.
2. **Core verb** — what the player mostly *does* (jump, shoot, build, solve).
3. **Target feeling** — the main emotion (challenge, discovery, tension,
   relaxation, expression…).
4. **Core loop** — the 30-second loop, the 5-minute loop, and what makes a
   session end.
5. **Pillars** — 2–3 rules every feature must serve, plus 1–2 anti-pillars
   (what this game is *not*).
6. **MVP** — the smallest playable version, and what gets cut first if time
   runs short.
7. **Open questions / risks.**

For a bigger mechanic, add a short note in `docs/` covering: rules, formulas
(with variables), edge cases, tuning knobs (→ `config.js`), and how we'll know
it works.

## Workflow

1. **Prototype first.** Test the core idea with the smallest playable build
   (in `prototypes/<name>/` if it's an experiment). Decide: keep, change, or
   drop.
2. **Build in small playable steps.** Every commit should leave the game
   runnable.
3. **Verify before pushing** (see Testing).
4. **Playtest notes** go in `docs/playtest-notes.md`: what worked, what was
   confusing, what felt bad, bugs found, fun moments.

## Testing

- **Logic tests:** pure functions in `src/game/` get small tests (Node's built-in
  `node --test`), especially formulas, collisions, scoring and state
  transitions. Tests must be deterministic — inject a seeded RNG instead of
  calling `Math.random()` directly in logic.
- **Browser smoke test:** run a static server and use Playwright (headless
  Chromium is pre-installed in Claude's cloud environment) to load the page,
  check there are no console errors, simulate a few inputs, and take a
  screenshot. Do this before pushing any visible change.
- Feel (controls, timing, juice) can't be automated — it's checked by playing.

## Balancing

When tuning: list the relevant values from `config.js`, look for outliers and
dominant strategies (one option always best), check the difficulty curve over
a session, change one thing at a time, and record why in a commit message.

## Bug reports

Keep them short: what happened, what should happen, steps to reproduce,
browser, and a screenshot if visual.

## Git

- Default branch: `main`. Commit small, working steps.
- Commit messages use Conventional Commits: `feat:`, `fix:`, `refactor:`,
  `docs:`, `test:`, `chore:`.
- Never commit secrets, `node_modules/` or build output.

## Deployment

GitHub Pages from `main` (repo root). Because there's no build step, what's in
the repo is what gets served. Use relative paths for all assets.

## Decisions

<!-- Record notable choices here: library added, structure changed, and why. -->
- Web stack (HTML/JS, Canvas/Three.js) chosen over a game engine: no install,
  testable headlessly, instant to share.
