# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A from-scratch clone of the arcade game Asteroids, rendered with the HTML5 Canvas 2D API. Pure vanilla JavaScript (ES6+), no frameworks, no bundler, no dependencies, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build step, linter, or test suite — the game runs directly from source.

## Architecture

Everything lives in `game.js` (single file, loaded directly by `index.html` via a plain `<script>` tag — no modules). The file is organized top-to-bottom as: input handling, math utils, entity classes, game state, update loop, draw loop, and the `requestAnimationFrame` main loop at the bottom.

- **Coordinate space**: fixed `W = 800` × `H = 600` canvas. All entities wrap toroidally at the edges via the `wrap()` util (space wraps around, not bounded).
- **Entities** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each is a class with `update(dt)` and `draw()` methods, plus a `dead` flag. Dead entities are filtered out of their arrays each frame rather than removed in place.
- **Game state** is a set of module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) reassigned by `initGame()` / `nextLevel()` — there is no state container object or class.
- **`state`** is a simple string machine: `'playing' | 'dead' | 'gameover'`, branched on at the top of `update(dt)`.
- **Collision detection** is plain circle-distance checks (`dist()` + `radius`), done in `update()`: bullet-vs-asteroid (splits asteroids into two smaller ones via `Asteroid.split()`, or removes them at the smallest size) and ship-vs-asteroid (triggers `killShip()`, respects `ship.invincible` respawn window).
- **Input**: raw keyboard state tracked in `keys` (held) and `justPressed` (edge-triggered, consumed via `pressed(code)`) — no input abstraction layer.
- Asteroid size tiers (3 = large → 1 = small) drive radius, speed, and score via the parallel arrays `RADII`, `SPEEDS`, `POINTS` indexed by size.

When editing gameplay, tuning constants (speeds, cooldowns, radii, drag, spawn counts) are inlined near their point of use inside the relevant class or function rather than centralized — check the whole file for a constant before assuming there's a single config block.
