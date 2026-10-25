# AGENTS.md

## Run
- Static site, no build/package manager. Open `src/index.html` in a browser (or serve `src/` with any static server). No `npm` scripts exist.

## Architecture (non-obvious)
- Files are loaded via `<script>` tags in `src/index.html` in this exact order: `maze.js` → `game.js` → `render.js` → `main.js`. All share globals on `window` (e.g. `MAZE`, `createGame`, `update`, `draw`). Changing load order or adding modules without globals will break things.
- `src/js/maze.js`: 28×31 maze parsed from `MAZE_STR` (row 14 is the tunnel). Key globals: `MAZE` (pristine), `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`. `game.grid` is a copy of `MAZE` per game so eaten dots don't mutate the original.
- `src/js/game.js`: Tick logic. Pacman moves at `PACMAN_SPEED=0.125` cells/frame, ghosts at `GHOST_SPEED=0.1`. Movement is grid-aligned with buffered `nextDir`. Tunnel wrap only when rounded row == `TUNNEL_ROW`. Pacman cannot pass through walls (`1`) or ghost door (`3`); ghosts cannot pass through walls only. Ghost targeting lives in `getTarget()` + `decideGhost()`; the pen-release queue (`releaseClock`/`releasedCount`/`RELEASE_INTERVAL`) advances in `update()` and gates movement in `moveGhost()`.
- `src/js/render.js`: Arcade-style wall drawing connects adjacent wall cell centers (only right and down to avoid duplicates). Canvas sized by maze dimensions via `TILE=20` (index.html sets 560×620). `GHOST_COLORS` is keyed by ghost `kind`, and ghosts are drawn **before** Pacman so he always renders on top.
- `src/js/main.js`: Game loop with `requestAnimationFrame`, arrow keys buffer direction while playing, overlay states `start/won/lost`.

## Gotchas
- Eating a dot sets `grid[y][x]=0`, increments score by 10, decrements `dotsRemaining`. Win when `dotsRemaining <= 0`.
- Collision uses distance < 0.5 between pacman/ghost positions; on hit lives decrease, positions reset if lives remain.
- Ghost AI: every ghost has a `kind` and picks the legal direction whose next cell is closest (Manhattan) to its target, never turning 180° unless it is in a dead end. Targets from `getTarget()`: `aggressive` → Pacman's rounded position; `ambusher` → 4 cells ahead of Pacman; `flanker` → 2 cells ahead of Pacman; `random` → no target, chooses randomly among the legal directions.
- Door is tile `3` (`'-'`) rendered separately; only pacman is blocked by it.
- Start position: pacman at (13,23); the 4 ghosts start inside the pen on row 14: (13,14)=aggressive, (14,14)=ambusher, (12,14)=flanker, (15,14)=random.
- Pen release: only the `aggressive` ghost moves at the beginning (`released: true`, `releasedCount: 1`); every `RELEASE_INTERVAL = 80` frames (≈1.33s @60fps) the next one is released until all 4 are in play. Losing a life calls `resetPositions()`, which does **not** reset the queue.
- All timing is frame-based (no delta-time): on >60Hz displays the whole game — movement *and* ghost release — runs proportionally faster. Known, accepted and consistent; see spec 01.

## Repo notes
- README is Spanish and describes learning spec-driven development; `skills-lock.json` pins `spec`/`spec-impl` skills from `Klerith/fernando-skills`.
- No tests, linting, or build configuration present.
