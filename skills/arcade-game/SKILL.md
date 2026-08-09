---
name: arcade-game
description: Crea un videojuego arcade jugable (Flappy Bird, Snake, Pong, Breakout…) como app HTML autocontenida con canvas. Úsala cuando el usuario pida "hazme un juego", "un flappy bird", "una snake", "un jueguecito", o quiera algo interactivo y divertido. El juego corre 100% en el navegador, sin librerías, con teclado y táctil.
---

# arcade-game — un juego jugable en un solo archivo

Un buen juego arcade cabe en un `<canvas>` + un bucle `requestAnimationFrame`, sin librerías.
Corre en el visor de Claw al instante y funciona con **teclado Y táctil** (móvil incluido).

## Anatomía (esto es lo que no puede faltar)

1. **Canvas** de tamaño fijo (p. ej. 360×560), centrado.
2. **Estado** en un objeto (`S`): posición, velocidad, obstáculos, score, `over`, `started`.
3. **Bucle** con `requestAnimationFrame`: `update()` (física + colisiones) → `draw()`.
4. **Entrada**: `keydown` (Space/flechas) **y** `pointerdown` (táctil) — el mismo handler.
5. **Ciclo de vida**: pantalla de inicio → jugar → game over → reiniciar con un toque.
6. **Score** visible y récord de la sesión.

## Física y colisiones (lo mínimo que lo hace sentir bien)

- Gravedad/velocidad constantes por frame (`v += G; y += v`), impulso al pulsar (`v = FLAP`).
- Colisión por caja/círculo: comprueba solape en X **y** en Y; añade márgenes para que sea justo.
- Dificultad por tiempo: genera obstáculos cada N frames; sube velocidad poco a poco.
- Clampa la rotación/velocidad para que se vea natural (`Math.max/min`).

El `template.html` de esta carpeta es un **Flappy Bird completo y probado** (bucle, gravedad,
tuberías, colisión, score, récord, reinicio, teclado+táctil). Úsalo de base y cambia la temática
(nave/pez/coche), los colores y las reglas para otros juegos.

## Otros juegos con el mismo esqueleto

- **Snake**: rejilla, la serpiente es una lista de celdas; comer alarga; choque con pared/cuerpo = fin.
- **Pong/Breakout**: pala controlada por puntero/flechas, pelota con rebote (invierte vx/vy).
- **Runner**: obstáculos que vienen de la derecha, saltar con impulso (como Flappy pero en suelo).

## Reglas

- **Sin librerías externas** salvo que se pida: canvas 2D puro carga instantáneo y no falla offline.
- **Táctil + teclado** siempre: mucha gente lo abre en el móvil.
- **Reinicio de un toque** tras game over: nada de recargar la página.
- Mantén 60 fps: dibuja solo lo necesario, no crees objetos dentro del bucle sin control.

## Lo que ya falló

- Si generas obstáculos por tiempo pero el jugador muere antes del primer spawn, parece que "no hay
  juego": pon el primer obstáculo pronto o una pantalla de inicio clara.
- No pauses el `requestAnimationFrame` en game over: sigue dibujando la pantalla de fin (si paras el
  loop, el reinicio por toque no repinta).
- `e.preventDefault()` en Space o la página hace scroll y arruina el juego.
