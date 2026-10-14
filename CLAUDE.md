# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no build, no package.json, no tests, no linter. README.md is in Spanish; UI strings are Spanish too.

## Idioma

Responder siempre en español en este proyecto.

## Running

Open `index.html` directly, or serve statically: `python -m http.server 8000` → http://localhost:8000.

## Architecture

Three files: `index.html` (DOM + two canvases), `style.css`, `game.js` (all logic, single script, global state, `'use strict'`).

- Canvas sizes are hardcoded in `index.html` (`#board` 300×600, `#next-canvas` 120×120) and must match `COLS*BLOCK` × `ROWS*BLOCK` in `game.js` if those constants change.
- State lives in module-level `let` vars (`board`, `current`, `next`, `score`, …) reset by `init()`; restart button calls `init()`.
- Board cells: `0` empty, `1–8` index into both `COLORS` and `PIECES` (same index = piece type).
- Flow: `loop` (rAF, accumulates `dropAccum` vs `dropInterval`) → `lockPiece` = `merge` → `clearLines` → `spawn`. `spawn` calls `endGame` if new piece collides. Hard/soft drop and key handling are in the single `keydown` listener at the bottom.
- Pause/game over cancel the rAF (`cancelAnimationFrame(animId)`); `togglePause` restarts the loop manually and resets `lastTime`.
- Rotation: `rotateCW` + simple horizontal kick list `[0,-1,1,-2,2]` in `tryRotate` (not SRS).
- Tema claro/oscuro: colores en variables CSS (`:root`/`[data-theme="dark"]` y `[data-theme="light"]` en `style.css`). `applyTheme` en `game.js` fija `data-theme`, lee `--grid` para `drawGrid` (`gridColor`), persiste en `localStorage` (`tetris-theme`) y redibuja. Por defecto: oscuro.
- Power-ups: cada `POWER_EVERY` (2) líneas `clearLines` marca `powerPending` (además `POWER_CHANCE` 10 % por pieza); `newPiece` genera entonces `randomPower` (1×1, `type 9`, campo `power`, fuera de `PIECES`). `lockPiece` llama `applyPower` en vez de `merge`. `freezeLeft` (ms) detiene la caída automática en `loop`. HUD `#power`.
- Skins: objeto `SKINS` (`retro`/`neon`/`pastel`/`pixel`) con `colors` (índices 1–9) y `drawBlock(ctx,px,py,color,size)`. `applySkin` fija `COLORS`/`skin` activos, `data-skin` en `<html>`, persiste en `localStorage` (`tetris-skin`), relee `--grid` y redibuja. `drawBlock` global delega en la skin (con `save/restore` para resetear `globalAlpha`/`shadowBlur`). Neon fuerza fondo negro de `#board`/`#next-canvas` vía `[data-skin="neon"]` en CSS. Botón `#skin-toggle` cicla skins.
- Scoring/speed: `LINE_SCORES[cleared] * level`; hard drop +2/cell, soft drop +1/row; level = `lines/10 + 1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
