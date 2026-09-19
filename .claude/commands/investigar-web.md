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
3. Escala a la skill `navegador-cdp` (Chrome real por CDP) cuando 1 y 2 hayan fallado de verdad. **En cuanto se cumpla uno de estos casos, escalar es obligatorio, no opcional:**
   - `WebFetch` devuelve CAPTCHA, página de bloqueo, o error 403/429.
   - `WebFetch` devuelve HTML sin el contenido pedido (solo scripts/navegación): lo genera JavaScript al cargar.
   - Hace falta interactuar: rellenar filtros, paginar, desplegar, iniciar sesión.
   - Hace falta una sesión iniciada para ver el contenido o el precio.
4. Un bloqueo no es un resultado. Prohibido entregar «no verificado» o «sin precio» por un bloqueo de `WebFetch` sin haber escalado antes. No reintentes variantes de la misma URL: un fallo ya basta para escalar. Sitios ya comprobados que fallan por `WebFetch` (ver [[entorno]]): amazon.es, leroymerlin.es, bauhaus.es.
5. Solo una prohibición explícita del usuario («no uses el navegador») suspende el paso 3. Una duda, un escepticismo o una preferencia («no creo que lo saques sin navegador») no es una prohibición: es un reto, y se responde escalando. Si hay prohibición explícita, dilo, entrega lo verificado y marca lo que falta como no verificado.
6. Consultas de precio o comparación de producto siguen el mismo escalado — no es una skill aparte, es solo el criterio de búsqueda. Precio, envío y opiniones de un producto salen de la página del producto, no de resúmenes del buscador («desde X €» no es un precio).
7. Si el usuario nombra un sitio o tienda como prioritario, esa fuente se resuelve primero y se escala sin más para ella.
8. Dime siempre en qué paso se resolvió cada parte de la consulta (1, 2, o escalado a `navegador-cdp`) y por qué. Si una parte quedó sin resolver, dilo, no declares la consulta resuelta. No atribuyas al navegador lo que salió de `WebFetch` ni al revés.
9. Comprobación obligatoria antes de redactar la respuesta final: repasa cada `WebFetch` de la sesión. Si alguno acabó en 403/429, CAPTCHA o HTML sin el contenido, y no hay ni escalado hecho ni prohibición explícita del usuario, no respondas todavía: escala. Un «sin verificar» sin una de esas dos justificaciones es un fallo de esta skill.
