---
description: Resuelve una consulta o búsqueda en internet escalando por coste
argument-hint: [pregunta o qué buscar]
creado: 2026-09-19
revisado: 2026-09-19
descartado: skill separada `precio-producto` con Keepa/SerpApi — el usuario decidió que comparar precio/producto es solo un criterio de búsqueda dentro de esta skill, no una capacidad aparte. Ver [[entorno]] y [[decisiones]].
---

Resuelve `$ARGUMENTS` escalando por coste. No saltes pasos.

1. `WebSearch` primero, para localizar fuentes y respuestas ya indexadas.
2. Si necesitas leer una página concreta que ha salido en la búsqueda, `WebFetch`.
3. Escala a la skill `navegador-cdp` (Chrome real por CDP) solo cuando 1 y 2 hayan fallado de verdad:
   - `WebFetch` devuelve CAPTCHA, página de bloqueo, o error 403/429.
   - El contenido no está en el HTML porque lo genera JavaScript al cargar.
   - Hace falta interactuar: rellenar filtros, paginar, desplegar, iniciar sesión.
   - Hace falta una sesión iniciada para ver el contenido o el precio.
4. Consultas de precio o comparación de producto siguen el mismo escalado — no es una skill aparte, es solo el criterio de búsqueda: prueba `WebSearch`/`WebFetch` en las páginas del producto antes de pensar en navegador.
5. Dime siempre en qué paso se resolvió la consulta (1, 2, o escalado a `navegador-cdp`) y por qué, si escalaste.
