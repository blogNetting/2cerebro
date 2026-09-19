---
title: Entorno y herramientas de esta máquina
created: 2026-09-19
updated: 2026-09-19
tags: [entorno, mcp, navegador, meta]
zona: tecnico
---

Qué hay montado en esta VM para no redescubrirlo cada sesión, y qué falta.

## Escalado de búsquedas e investigación web

Orden obligatorio, ver `AGENTS.md`: `WebSearch` → `WebFetch` → navegador real por CDP. Skills: [[investigar-web]] (capas 1 y 2, autónoma) y `/navegador-cdp` (capa 3, necesita un Chrome abierto en alguna máquina).

## Navegador por CDP

- Chrome en modo visible no acepta `--remote-debugging-address` (decisión de seguridad de Chromium, issue 41487252, sin log). Headless queda descartado porque hace falta ventana visible para resolver CAPTCHAs a mano.
- Solución montada: Chrome arranca en el anfitrión Windows escuchando en `127.0.0.1:9222`, y un `netsh portproxy` expone ese puerto en la IP de la LAN de esa máquina.
- **Anfitrión por defecto: `192.168.1.5:9222`.** Es el único host que se persiste (aquí y en `.mcp.json`). Verificado con `curl`.
- Windows 10 Pro de ese anfitrión solo admite una sesión interactiva: RDP mata el render del Chrome de la sesión de consola. Por eso la máquina a usar se elige por consulta, nunca por defecto salvo la de arriba, o la que fije `CDP_ENDPOINT` al arrancar la sesión (si está definida y responde, no se pregunta; ver [[decisiones]]).
- No hay ni se quiere servidor SSH en el anfitrión.
- MCP registrado en `.mcp.json` de este proyecto: `playwright-cdp` (`@playwright/mcp`, vía `npx`), lanzado con `--cdp-endpoint=${CDP_ENDPOINT:-http://192.168.1.5:9222}`.
- Limitación real: el endpoint se fija al arrancar el proceso MCP y no se puede recablear en caliente dentro de la misma sesión de Claude Code. Para apuntar a otra máquina hace falta relanzar la sesión con `CDP_ENDPOINT=http://IP:PUERTO claude`. Ver `/navegador-cdp` para el flujo completo, incluida la plantilla del `.bat`.
- Alternativa descartada: `chrome-devtools-mcp` (Google). También conecta a un Chrome existente por CDP (`--browserUrl`) y cumple el criterio eliminatorio, pero está pensado para depurar (red, performance, consola), no para conducir el navegador. Playwright MCP se ajusta mejor a rellenar/paginar/iniciar sesión, y cada tool de más es contexto consumido en cada turno. Decisión en [[decisiones]].
- Alternativa descartada: Puppeteer MCP — deprecado/archivado.
- Alternativa descartada: navegadores cloud (BrowserBase y similares) — lanzan su propio navegador remoto, no conectan al Chrome existente. No cumplen el criterio eliminatorio.

## Sitios comprobados frente a WebFetch (2026-09-19)

- Fallan por `WebFetch` y exigen escalar a CDP: amazon.es (HTML sin cuerpo, precio/envío/opiniones no vienen; reseñas devuelven 503), leroymerlin.es (403), bauhaus.es (403).
- Funcionan por `WebFetch`: tiendas pequeñas de ferretería/pintura (precio con IVA legible), fichas técnicas en PDF (se guardan en `tool-results/`, leer con `pdftotext`).
- Por CDP, amazon.es se lee bien con `browser_evaluate` sobre el DOM: `#productTitle`, `#corePrice_feature_div .a-offscreen` (precio), `#deliveryBlockMessage` (envío al CP configurado en la cuenta), `#acrPopover`/`#acrCustomerReviewText` (nota), `[data-hook="review"]` (reseñas). Reseñas negativas: `/product-reviews/<ASIN>?filterByStar=critical`. Un bucle `browser_run_code_unsafe` sobre varias ASIN saca precio y nota de todas en una llamada, sin CAPTCHA. Trampa comprobada: no usar `.a-price .a-offscreen` como respaldo del precio; en fichas «No disponible» devuelve el precio de otro producto del carrusel (dio 11,16 € al Maurer, que era el PROA). Usar solo `#corePrice_feature_div` y, si falta, tratar el precio como ausente. Para descubrir alternativas, la búsqueda `amazon.es/s?k=...` con `[data-component-type="s-search-result"]` devuelve título, precio, nota y envío de ~12 productos.
- Los resúmenes de `WebSearch` no son fuente de precio: «desde 13,99 €» resultó no ser el precio de venta (15,65 €).

## Datos de producto/precio

- Evaluado y descartado por ahora: Keepa API (histórico de precio/rank de Amazon) y SerpApi Price Monitoring. Ambos evitarían scraping para consultas de precio, pero requieren API key de pago (Keepa por tokens, SerpApi tier gratuito pequeño). Las consultas de precio/comparación de producto se resuelven con el escalado normal de [[investigar-web]], sin capa aparte.

## Herramientas disponibles en esta VM

- Node.js v22.23.2 y npx 10.9.8 instalados (verificado `node --version` / `npx --version`). Suficiente para lanzar `@playwright/mcp` bajo demanda; no hace falta instalación previa, `npx -y` lo descarga la primera vez.
- Sin GPU, sin entorno gráfico: irrelevante para esta arquitectura porque el navegador real corre en el anfitrión Windows, no aquí. Esta VM solo ejecuta el proceso Node del MCP, que habla por red al CDP remoto.

## Compartir contexto entre sesiones

- La extensión de VSCode y el CLI mantienen almacenes de sesión separados (verificado en docs oficiales): una conversación de VSCode no se puede recuperar con `claude --continue`/`--resume` desde una terminal, ni al revés.
- Mecanismo puente: `.claude/sesiones/` (gitignored, fuera del wiki, no entra en `/lint`). Skills `/exportar-sesion` (vuelca un resumen de estado a un fichero con nombre `<slug>-<timestamp>.md`) y `/importar-sesion` (lee ese fichero en la sesión destino). Por defecto se vuelca resumen, no transcripción literal — cuesta menos contexto a la sesión receptora.

## Qué falta / no está resuelto

- No hay skill ni MCP para leer contenido detrás de login sin intervención humana — eso sigue siendo tarea manual del usuario en la ventana visible.
- Si algún día hace falta inspeccionar red/performance de una página (no solo interactuar), ahí sí entra `chrome-devtools-mcp` como añadido, no como sustituto.
