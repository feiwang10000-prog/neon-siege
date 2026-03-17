# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the games

Each game is a self-contained HTML file — open directly in a browser:

```bash
open shooter.html
open tictactoe.html
```

No build step, no server, no dependencies.

## Repository conventions

**Commit and push after every meaningful unit of work.** Do not batch multiple features or fixes into one commit. The goal is that the GitHub remote always reflects the current state of the project so work is never lost and any commit can be reverted cleanly.

When to commit:
- After completing a feature or behaviour change
- After fixing a bug
- After any refactor, even small ones
- Before starting a risky or experimental change

Commit message format: one-line summary in imperative mood (e.g. `Add zigzag enemy type`), optional blank line + bullet body for non-obvious details, always ending with the `Co-Authored-By` trailer.

Push to `origin/main` (GitHub: `feiwang10000-prog/neon-siege`) immediately after each commit.

## Architecture — shooter.html

Single file: inline CSS → `<canvas id="c" 800×600>` → one `<script>` block.

**State machine:** `MENU → PLAYING ↔ PAUSED → GAME_OVER → MENU`
Driven by `Game.state` (values in `STATES` const); the RAF loop branches on it each frame.

**Core classes (all in the single script):**

| Class | Responsibility |
|---|---|
| `Game` | RAF loop, state machine, entity arrays, collision, score, screen-shake |
| `InputHandler` | `Set` of held keys, canvas-relative scaled mouse coords, `justClicked` one-shot flag |
| `Player` | Movement, angle-to-mouse, fire cooldown, walk cycle, invulnerability frames |
| `Enemy` | Per-type config lookup, direct + zigzag movement, contact damage timer |
| `Bullet` | Velocity vector, lifetime |
| `ParticleSystem` | Fixed pool of 400 objects reused on death; methods `spawnExplosion`, `spawnHit`, `spawnMuzzleFlash` |
| `LevelManager` | Wave queue, spawn timer, state `SPAWNING→WAITING→WAVE_CLEAR→next`; levels 1–3 hand-crafted, 4+ procedural |

**Camera:** world-space transform applied each frame — `ctx.translate(CANVAS_W/2 - player.x, CANVAS_H/2 - player.y)` — then restored before HUD drawing.

**Mouse scaling:** all mouse events use `getBoundingClientRect()` + CSS-scale correction so coordinates are always in canvas pixels regardless of window size.

**Collision:** circle-circle only. Bullet radius = 4, player radius = 12, enemy radius = enemy `size` value.

**Enemy types** (defined in `ENEMY_TYPES` const): `walker`, `runner`, `tank`, `zigzag` — each has `hp`, `speed`, `size`, `score`, `color`.

**Persistence:** high score saved to `localStorage` key `neonSiegeHi`.

## Architecture — tictactoe.html

DOM-based (no canvas). Game state is a 9-element array; winner detection checks all 8 lines after each move.
