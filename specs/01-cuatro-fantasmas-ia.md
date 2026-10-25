# SPEC 01 — Cuatro fantasmas con IA propia y liberación de la pen

> **Status:** Aprobado
> **Depends on:** —
> **Date:** 2026-10-25
> **Objective:** Tener 4 fantasmas con comportamientos distintos (uno persigue agresivamente a Pacman) que salen de la pen uno cada 1.5 segundos.

## Scope

**In:**

- 4 fantasmas en `GHOST_STARTS`, cada uno con su `kind` y comportamiento propio.
- Fantasma `aggressive` que persigue agresivamente la posición actual de Pacman.
- Fantasmas `ambusher` (4 celdas delante de Pacman) y `flanker` (2 celdas delante) con estilo arcade.
- Fantasma `random` que elige dirección al azar (comportamiento ya existente).
- Liberación escalonada de la pen: un fantasma cada 1.5 segundos.
- Colores propios por `kind` y orden de dibujo correcto (fantasmas debajo de Pacman).
- Actualización de `AGENTS.md` con la IA real de los fantasmas.

**Fuera de alcance (para specs futuros):**

- Power-ups, frutas y fantasmas comibles (modo asustado).
- Animación de salida de la pen (camino hasta la puerta): los fantasmas usan su IA normal para salir.
- Reabrir la cola de liberación al perder una vida.
- Corrección por delta-time para monitores >60Hz.
- Niveles, vidas extra o dificultad progresiva.

## Data model

```js
// maze.js — 4 fantasmas dentro de la pen (interior: cols 11-16, fila 14)
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'aggressive' }, // rojo, persigue a Pacman
  { x: 14, y: 14, kind: 'ambusher' },   // rosa, embosca 4 celdas adelante
  { x: 12, y: 14, kind: 'flanker' },    // cian, flanquea 2 celdas adelante
  { x: 15, y: 14, kind: 'random' },     // naranja, dirección aleatoria
];

// game.js — liberación de la pen
const RELEASE_INTERVAL = 90; // frames @60fps = 1.5s

// estado de game (createGame)
{
  releaseClock: 0,     // frames desde la última liberación
  releasedCount: 1,    // el agresivo ya sale al inicio
  ghosts: [/* ..., released: true|false */],
}
```

Convenciones:

- Coordenadas: celda (x,y), origen arriba-izquierda, iguales que el resto del juego.
- La liberación se cuenta en frames, igual que `PACMAN_SPEED` y `GHOST_SPEED` (sin delta-time).

## Implementation plan

1. En `src/js/maze.js`, ampliar `GHOST_STARTS` a los 4 fantasmas anteriores. Prueba manual: el juego carga y pinta 4 fantasmas en la pen (aún se moverán igual, el AI nuevo viene después).
2. En `src/js/game.js`, añadir `getTarget(game, g)` con el objetivo por `kind` y reescribir `decideGhost()` para elegir la dirección legal más cercana al objetivo (distancia Manhattan, sin 180° salvo callejón); `random` conserva la elección aleatoria. Prueba manual: el roje persigue directo, el rosa va a un punto por delante de Pacman.
3. En `src/js/game.js`, implementar la liberación: flag `released` en cada fantasma, `releaseClock`/`releasedCount` en `game`, paso de contador en `update()` y guard en `moveGhost()`. Prueba manual: cronometrar que sale uno cada ~1.5s.
4. En `src/js/render.js`, mapear `GHOST_COLORS` por `kind` y dibujar los fantasmas antes que Pacman. Prueba manual: colores rojo/rosa/cian/naranja en orden y Pacman por encima.
5. Actualizar `AGENTS.md` (IA de fantasmas, posiciones de inicio y liberación) para que refleje el código.

## Acceptance criteria

- [ ] El juego carga sin errores en la consola.
- [ ] `createGame()` crea 4 fantasmas con `kind` aggressive, ambusher, flanker y random.
- [ ] El fantasma `aggressive` reduce la distancia a Pacman en cada intersección mientras lo tenga a la vista.
- [ ] El fantasma `ambusher` elige caminos hacia un punto 4 celdas por delante de Pacman.
- [ ] El fantasma `flanker` elige caminos hacia un punto 2 celdas por delante de Pacman.
- [ ] El fantasma `random` elige dirección al azar entre las legales.
- [ ] Al iniciar la partida solo se mueve 1 fantasma; los otros 3 quedan estáticos en la pen.
- [ ] Cada 1.5 segundos (90 frames) se libera el siguiente fantasma, hasta que los 4 se mueven.
- [ ] Tras perder una vida, las posiciones se reinician pero la cola de liberación no (no se repite la espera).
- [ ] Cada fantasma se dibuja con su color: rojo, rosa, cian, naranja.
- [ ] Pacman se dibuja por encima de los fantasmas.

## Decisions

- **Sí:** comportamientos estilo arcade (emboscada 4 celdas, flanqueo 2 celdas) — decisión elegida por el usuario.
- **No:** comportamientos inventados para este proyecto — descartada la opción "personalidades nuevas".
- **Sí:** liberación por frames (90 @60fps) — coherente con toda la simulación, que ya es por frame.
- **No:** medir con `Date.now()` o delta-time — crearía una inconsistencia (el juego correría a dos velocidades distintas en monitores >60Hz).
- **Sí:** el agresivo se libera al inicio — el fantasma que persigue debe estar en juego de inmediato.
- **No:** reiniciar la cola de liberación al morir — repetiría la espera en cada vida y ralentizaría el juego.
- **No:** animación de salida por la puerta — la IA normal ya hace que salgan; queda para otro spec.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| En monitores >60Hz todo corre más rápido (juego y liberación) | Es el comportamiento ya existente del juego; consistente y aceptado. Se documenta en `AGENTS.md`. |
| 4 fantasmas en la pen podrían solaparse al ser liberados | Al estar estáticos hasta la liberación y salir uno a la vez, no se generan colisiones falsas (Pacman no puede entrar en la pen). |

## Qué **no** queda en este spec

- Power-ups, frutas y modo asustado (spec futuro si se decide).
- Animación de salida de la pen (spec futuro si se decide).
- Ajuste delta-time para >60Hz (spec futuro si se decide).
- Niveles o dificultad progresiva (spec futuro si se decide).
