---
name: web-research
description: Busca cosas en internet y ayuda a completar un proceso con lo que encuentra — comparar opciones, reunir datos, rellenar una plantilla, resumir con fuentes. Úsala cuando el usuario pida "busca en internet…", "compárame…", "investiga…", "ayúdame a decidir/rellenar/planificar X" y haga falta información de fuera. Usa las tools web de Claw (web.search / web.fetch) y termina con un resultado accionable, no solo enlaces.
---

# web-research — buscar y AYUDAR con un proceso, no solo devolver enlaces

Claw ya trae acceso a internet: `web.search` (texto: títulos, URLs, fragmentos) y `web.fetch` (leer
una URL). Esta skill es sobre usarlas bien y **cerrar el proceso** con algo útil.

## Método

1. **Descompón la pregunta** en 2-4 sub-búsquedas concretas (no una genérica). Ej. "mejor X para Y"
   → busca "X comparativa 2026", "X opiniones", "X precio".
2. **Busca** con `web.search`; abre 1-3 resultados prometedores con `web.fetch` para leer de verdad,
   no te quedes en el fragmento.
3. **Cruza fuentes**: si dos coinciden, súbelo a "confiable"; si discrepan, dilo.
4. **Cierra el proceso** con un ENTREGABLE accionable, no una lista de links:
   - comparación → **tabla** (opción · pros · contras · precio · fuente).
   - decisión → recomendación + porqué + qué falta por confirmar.
   - datos para otra tarea → pásalos a la skill que toque (p. ej. `doc-to-excel` para una hoja).
5. **Cita las fuentes** (URL) de cada dato que venga de fuera. Nada de inventar.

## Ejemplos de "ayudar a un proceso"

- *"Ayúdame a elegir impresora 3D por debajo de 500€"* → busca modelos, lee reviews, devuelve una
  **tabla comparativa** con fuentes y una recomendación razonada.
- *"Reúne los requisitos de la beca X y hazme una checklist"* → fetch de la web oficial, extrae
  requisitos y fechas, genera una **checklist** con enlaces a cada punto.
- *"Precios de vuelo Madrid-Tokio en marzo"* → busca, lee, resume rango de precios + cuándo sale
  mejor, citando de dónde.

## Reglas

- **Nunca digas que no puedes buscar** — para eso están `web.search`/`web.fetch`. Si una búsqueda no
  da nada, reformula, no te rindas a la primera.
- **Distingue dato de fuente de opinión tuya.** Lo que afirmes de fuera, con URL.
- **Termina con algo que el usuario pueda usar** (tabla, checklist, recomendación, o datos para otra
  app), no con "aquí tienes unos enlaces".
- Si el resultado son datos tabulables y el usuario querrá Excel, encadena con **doc-to-excel**.

## Lo que ya falló

- Buscar una sola vez con una consulta genérica devuelve ruido: 2-4 búsquedas específicas > 1 amplia.
- Quedarse en el fragmento de `web.search` lleva a error; para un dato que importa, `web.fetch` y léelo.
- Devolver enlaces sin sintetizar no "ayuda al proceso": el valor está en el entregable final.
