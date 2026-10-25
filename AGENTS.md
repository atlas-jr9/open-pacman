# AGENTS.md

## Run
- Static site, no build/package manager. Open `src/index.html` in a browser (or serve `src/` with any static server). No `npm` scripts exist.

## Architecture (non-obvious)
- Files are loaded via `<script>` tags in `src/index.html` in this exact order: `maze.js` → `game.js` → `render.js` → `main.js`. All share globals on `window` (e.g. `MAZE`, `createGame`, `update`, `draw`). Changing load order or adding modules without globals will break things.
- `src/js/maze.js`: 28×31 maze parsed from `MAZE_STR` (row 14 is the tunnel). Key globals: `MAZE` (pristine), `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`. `game.grid` is a copy of `MAZE` per game so eaten dots don't mutate the original.
- `src/js/game.js`: Tick logic. Pacman moves at `PACMAN_SPEED=0.125` cells/frame, ghosts at `GHOST_SPEED=0.1`. Movement is grid-aligned with buffered `nextDir`. Tunnel wrap only when rounded row == `TUNNEL_ROW`. Pacman cannot pass through walls (`1`) or ghost door (`3`); ghosts cannot pass through walls only.
- `src/js/render.js`: Arcade-style wall drawing connects adjacent wall cell centers (only right and down to avoid duplicates). Canvas sized by maze dimensions via `TILE=20` (index.html sets 560×620).
- `src/js/main.js`: Game loop with `requestAnimationFrame`, arrow keys buffer direction while playing, overlay states `start/won/lost`.

## Gotchas
- Eating a dot sets `grid[y][x]=0`, increments score by 10, decrements `dotsRemaining`. Win when `dotsRemaining <= 0`.
- Collision uses distance < 0.5 between pacman/ghost positions; on hit lives decrease, positions reset if lives remain.
- Ghost AI: hunter chases by Manhattan distance to rounded pacman position; random otherwise. If no options (dead end), ghosts may reverse direction.
- Door is tile `3` (`'-'`) rendered separately; only pacman is blocked by it.
- Start position: pacman at (13,23), ghosts start inside pen at (13,14)=hunter, (14,14)=random.

## Repo notes
- README is Spanish and describes learning spec-driven development; `skills-lock.json` pins `spec`/`spec-impl` skills from `Klerith/fernando-skills`.
- No tests, linting, or build configuration present.
