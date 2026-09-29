---
title: Entorno y herramientas de esta máquina
created: 2026-09-19
updated: 2026-09-29
tags: [entorno, mcp, navegador, meta]
zona: tecnico
---

Qué hay montado en esta VM para no redescubrirlo cada sesión, y qué falta.

## Escalado de búsquedas e investigación web

Orden obligatorio, ver `AGENTS.md`: `WebSearch` → `WebFetch` → navegador real por CDP. Skills: `/investigar-web` (capas 1 y 2, autónoma) y `/navegador-cdp` (capa 3, necesita un Chrome abierto en alguna máquina).

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

## Documentación de librerías por MCP (2026-09-27)

- **`context7`** montado para consultar documentación actualizada de librerías sin depender de lo que el modelo recuerde. Es de Upstash ([repo](https://github.com/upstash/context7), paquete `@upstash/context7-mcp`).
- **Registrado en la configuración de usuario** (`~/.claude.json`), **no** en el `.mcp.json` del proyecto. Dos razones: sirve en cualquier directorio —no solo el wiki— y `.mcp.json` está versionado en un repo **público** con push cada hora, así que cualquier credencial ahí se publica. Comando: `claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp`.
- **Transporte remoto HTTP, no local por `npx`.** Es lo que recomienda su propia documentación: más rápido y sin depender de Node. El local (`npx -y @upstash/context7-mcp`) funciona igual pero no aporta nada aquí.
- **Autenticación por OAuth, sin clave de API.** `claude mcp login context7` abre el navegador y guarda el token en `~/.claude/.credentials.json` (clave `mcpOAuth`). Con `scope: profile email offline_access` y `refreshToken`, **se renueva solo**; no hay que repetir el login. La clave de API (`context7.com/dashboard`) existe y es opcional, pero con OAuth no hace falta.
- **Trampa comprobada, y anotarla porque cuesta un rato:** `claude mcp list` **no prueba que estés autenticado**. El servidor responde `200` a una petición sin credencial ninguna (verificado con `curl`), así que el chequeo de salud pasa igual estando anónimo. La comprobación de verdad es mirar si `mcpOAuth` en `~/.claude/.credentials.json` tiene una entrada.
- **Y otra:** `claude mcp remove` borra la entrada del servidor **con** su login. Si se quita y se vuelve a añadir, hay que repetir el `login` (o comprobar que el token sobrevivió). Mejor no quitarlo «para reconfigurarlo» sin motivo.

## Privacidad del navegador que se conduce

- **El Chrome que se controla por CDP es el de uso diario, con las sesiones personales abiertas.** Cualquier `browser_snapshot` de una página de Google (Flights, búsqueda, Drive) incluye el rótulo de cuenta del perfil: nombre real y dirección de correo del usuario, dentro del árbol de accesibilidad. No es un fallo del MCP, es que el navegador está logueado.
- **Consecuencia práctica:** los snapshots de páginas con sesión iniciada no son material publicable. Nunca pegar su contenido en una nota del wiki, en `fuentes/` ni en una exportación de sesión — el repo es público y el cron publica lo que no esté ignorado.
- **Dónde se escriben:** `--output-dir=/home/netting/.cache/playwright-mcp`, fuera del repositorio (ver «Repositorio y sincronización»). Antes escribían en `./.playwright-mcp`, dentro, y el cron los subió: ver el incidente de 2026-09-19 en [[decisiones]].
- **Mitigación pendiente de decidir:** para rastreo conviene un perfil de Chrome sin cuentas personales iniciadas. Hoy no existe; se asume el riesgo y se evita publicar snapshots de páginas logueadas.

## Sitios comprobados frente a WebFetch (2026-09-19)

- Fallan por `WebFetch` y exigen escalar a CDP: amazon.es (HTML sin cuerpo, precio/envío/opiniones no vienen; reseñas devuelven 503), leroymerlin.es (403), bauhaus.es (403).
- Funcionan por `WebFetch`: tiendas pequeñas de ferretería/pintura (precio con IVA legible), fichas técnicas en PDF (se guardan en `tool-results/`, leer con `pdftotext`).
- Por CDP, amazon.es se lee bien con `browser_evaluate` sobre el DOM: `#productTitle`, `#corePrice_feature_div .a-offscreen` (precio), `#deliveryBlockMessage` (envío al CP configurado en la cuenta), `#acrPopover`/`#acrCustomerReviewText` (nota), `[data-hook="review"]` (reseñas). Reseñas negativas: `/product-reviews/<ASIN>?filterByStar=critical`. Un bucle `browser_run_code_unsafe` sobre varias ASIN saca precio y nota de todas en una llamada, sin CAPTCHA. Trampa comprobada: no usar `.a-price .a-offscreen` como respaldo del precio; en fichas «No disponible» devuelve el precio de otro producto del carrusel. Usar solo `#corePrice_feature_div` y, si falta, tratar el precio como ausente. Para descubrir alternativas, la búsqueda `amazon.es/s?k=...` con `[data-component-type="s-search-result"]` devuelve título, precio, nota y envío de ~12 productos.
- Los resúmenes de `WebSearch` no son fuente de precio: el «desde X €» del buscador no coincide con el precio de venta de la ficha.
- vueling.com: por CDP con URLs directas del calendario y del buscador; ver [[vueling-busqueda-por-url]].
- booking.com: por CDP con la URL de búsqueda y filtros en `nflt`: `roomfacility=38` (baño privado), `review_score=70` (7+), `distance=5000`, `ht_id=201` (apartamentos) o `204` (hoteles), `tdb=3` (1 cama doble). La tabla de habitaciones de cada ficha es `#hprt-table`. Con `browser_run_code_unsafe` no hay `require`: para acumular resultados entre navegaciones usar `sessionStorage` y volcarlo después con `browser_evaluate`.
- Ejemplo de barrido completo de comparativa de producto con este escalado (Amazon.es por CDP, fichas y reseñas): [[smartwatch-mujer-muneca-pequena]].

## Datos de producto/precio

- Evaluado y descartado por ahora: Keepa API (histórico de precio/rank de Amazon) y SerpApi Price Monitoring. Ambos evitarían scraping para consultas de precio, pero requieren API key de pago (Keepa por tokens, SerpApi tier gratuito pequeño). Las consultas de precio/comparación de producto se resuelven con el escalado normal de `/investigar-web`, sin capa aparte.

## Herramientas disponibles en esta VM

- Node.js v22.23.2 y npx 10.9.8 instalados (verificado `node --version` / `npx --version`). Suficiente para lanzar `@playwright/mcp` bajo demanda; no hace falta instalación previa, `npx -y` lo descarga la primera vez.
- Sin GPU, sin entorno gráfico: irrelevante para esta arquitectura porque el navegador real corre en el anfitrión Windows, no aquí. Esta VM solo ejecuta el proceso Node del MCP, que habla por red al CDP remoto.

## Repositorio y sincronización

- El repo `blogNetting/2cerebro` es público. `/home/netting/bin/cerebro-sync.sh` corre por cron cada hora: si hay cambios hace `git add -A`, commit `auto: <fecha>` y `git push`. Todo lo que no esté en `.gitignore` se publica solo. Ver la regla en `AGENTS.md` (Repositorio y artefactos) y el incidente de `.playwright-mcp/` en [[decisiones]].
- **Nada binario se versiona salvo que sea imprescindible:** `*.pdf` está en `.gitignore` — los PDF generados son entregables para el usuario y el contenido es el `.md`, que sí se versiona (ver `AGENTS.md`, «Formato de investigaciones»).
- **Lo que ya está publicado no se despublica sin decidirlo el usuario.** El historial contiene snapshots del 2026-09-19 con datos personales; la decisión y su análisis están en [[decisiones]] (2026-09-27).
- Playwright MCP escribe sus logs, capturas y snapshots en `/home/netting/.cache/playwright-mcp` (`--output-dir` en `.mcp.json`, fuera del repo). Antes de ese cambio escribía en `.playwright-mcp/` dentro del repo, ahora ignorado.

## Compartir contexto entre sesiones

- La extensión de VSCode y el CLI mantienen almacenes de sesión separados (verificado en docs oficiales): una conversación de VSCode no se puede recuperar con `claude --continue`/`--resume` desde una terminal, ni al revés.
- Mecanismo puente: `.claude/sesiones/` (gitignored, fuera del wiki, no entra en `/lint`). Skills `/exportar-sesion` (vuelca un resumen de estado a un fichero con nombre `<slug>-<timestamp>.md`) y `/importar-sesion` (lee ese fichero en la sesión destino). Por defecto se vuelca resumen, no transcripción literal — cuesta menos contexto a la sesión receptora.

## Fuentes especializadas para investigación técnica (2026-09-24)

Qué sistemas de investigación con agentes existen y cuáles se descartaron, con el motivo: [[sistemas-de-research-con-agentes]]

Comprobadas desde esta VM. Detalle de uso en `/investigar-web`, paso 10.

- `gh` 2.101.0 en `~/.local/bin`: binario oficial con el checksum verificado. Sirve para buscar repos, issues, releases y métricas de actividad. Sin autenticar, la API de GitHub da 60 peticiones por hora; autenticado (`gh auth login`), 5000.
- `yt-dlp` 2026.08.19 en `~/.local/bin`: binario oficial con el checksum verificado. Descarga los subtítulos automáticos de charlas en YouTube sin bajar el vídeo (`--skip-download --write-auto-subs`).
- Hacker News: la API de Algolia (`hn.algolia.com/api/v1/search`) funciona con `curl`, sin clave.
- Reddit: su `.json` devuelve 403 y old.reddit redirige; se lee por CDP.
- X/Twitter: la respuesta es 200, pero el contenido lo genera JavaScript; se lee por CDP.

## El ancla de verdad: reglas de prohibición (`deny`), no hooks

**Y esto se añadió el 2026-09-27, tras el tercer aviso del usuario.** El problema de fondo no era que los hooks estuvieran mal escritos: es que **un hook que comprueba un indicador siempre se puede satisfacer sin hacer el trabajo bien** — tocar un `.md` no es documentar, y prometer algo no es cumplirlo. El usuario lo dijo así: *«estoy hasta los cojones de que falles y hagas lo que te salga»*.

**Lo que sí sostiene, y está medido:** reglas **`deny`** en `permissions`. Se comprobó **en vivo** que **se respetan aunque el modo sea `bypassPermissions`** — el harness contestó *«Permission to use Bash with command … has been denied»* y el comando no llegó a ejecutarse. No depende de que el modelo se acuerde, ni de que un hook acierte: **lo impone el programa**.

Es la misma regla que dice la evidencia sobre agentes: *«el límite se pone con permisos, no con instrucciones»*.

**Lo que está prohibido ahora** (irreversible, credenciales, o publicar):

| Regla | Por qué |
|---|---|
| `gh pr merge*` | Fusionar no se deshace |
| `gh secret set*` · `gh secret delete*` | Toca credenciales |
| `gh release create/delete/edit*` | Publica versiones |
| `gh api` con `-X`/`--method` POST, PUT, PATCH o DELETE | Escrituras por la API |

**Y `git push` TAMBIÉN está prohibido desde el 2026-09-27** (antes no lo estaba). El usuario lo pidió así: *«haz lo que sea para que se cumpla y no pase más veces»*. **Consecuencia, y hay que asumirla: nada sale de manos del modelo.** Los commits se quedan en local y **el push lo hace el usuario**. Si resulta demasiado incómodo, se quita esa línea y las demás siguen.

**Lo que antes decía este apartado: `git push` no estaba prohibido.** Subir una rama es reversible y es como se comparte el trabajo. Si se quiere control total, se añade `Bash(git push*)` y **cada push pasa a ser del usuario**.

## Los tres hooks, y sus límites

**El orden de fuerza, de menos a más:** una regla escrita (no sirve sola) → un hook que comprueba un indicador (se puede satisfacer en falso) → **una regla `deny` (no se puede saltar).** Cuando importe de verdad, va al tercer escalón.

## Los tres hooks globales, y cómo funcionan

**Corrección del 2026-09-27, y es incómoda:** este apartado decía que `verificar-respuesta.sh` estaba «registrado como hook `Stop` en `~/.claude/settings.json`». **No lo estaba.** El fichero existía, pero **no había ninguna entrada `Stop` en la configuración**, así que nunca se ejecutó. Y la única entrada que sí había, `PermissionRequest`, era un `echo` que devolvía **`allow` para todo**: auto-aprobaba cualquier acción. Resultado: la regla escrita en `~/.claude/rules/comportamiento.md` («esto lo hace cumplir un hook») **no la hacía cumplir nadie**. Los dos están conectados desde hoy.

Los tres viven en `~/.claude/settings.json` y en `~/.claude/hooks/`:

| Hook | Evento | Qué hace |
|---|---|---|
| `permisos-hacia-fuera.sh` | `PermissionRequest` | **No auto-aprueba las acciones hacia fuera.** Si detecta `git push`, `gh pr merge`, `gh secret set`, `gh release create/delete`, o una escritura por `gh api` (`-X`/`--method`/`-f`/`-F`), devuelve **`ask`** y el harness **para y pregunta al usuario**. Todo lo demás sigue auto-aprobado, para que no sea un peaje constante. Patrón y prueba en [[decisiones]]. |
| `documentacion-al-dia.sh` | `Stop` | Si en el turno se ha tocado **código** —editar/crear un fichero que no es `.md`, o `git commit`/`push`/`gh pr merge`— y **ninguna documentación** (ningún `.md`, ni nada bajo `docs/`, `proyectos/`, `areas/`, `recursos/`), **bloquea el cierre** con el motivo. **Excluye `/tmp/`**, que no es código de nadie. Y cuenta la documentación escrita **de dos formas**: con las herramientas `Edit`/`Write`, y **desde `bash`** (un `write_text`, un `sed -i`, un `tee` o una redirección que apunte a un `.md`). **No juzga si la documentación es buena, solo que exista.** El segundo camino se añadió el 2026-09-27 tras un **falso positivo real**: se documentó la bitácora con un script dentro de un comando y el hook, que solo miraba `Edit`/`Write`, bloqueó un turno que **sí** había documentado. **Recortado el 2026-09-29:** tenía dos capas más —exigir además un documento *de seguimiento* (plan o bitácora) apuntando a `proyectos/astillero/astillero-plan.md`, y exigir el manual de Astillero cuando el código era suyo— y se retiraron **las dos**, por obsoletas: esa ruta ya no existe (Astillero se retiró del wiki) y el seguimiento de un proyecto vive en el repo de ese proyecto, no aquí. Ver `areas/decisiones.md`, 2026-09-29. |
| `verificar-respuesta.sh` | `Stop` | El que ya estaba escrito y nunca corría: antes de cerrar, un evaluador (Sonnet por `claude -p`) compara la última petición con la respuesta y bloquea si una investigación no llega al mínimo (máximo 3 veces). Contadores en `~/.cache/verificar-respuesta/`. |

**Para desactivar cualquiera:** `/hooks`, o quitar su entrada del `settings.json`.

**Aviso al cambiarlos:** el vigilante de configuración del harness solo mira carpetas que ya tenían `settings.json` cuando arrancó la sesión. Si se editan desde una sesión abierta antes de crearlos, **hace falta abrir `/hooks` una vez o reiniciar** — si no, el harness sigue con la configuración vieja y parece que el hook no funciona. Comprobado el 2026-09-27.

## Qué falta / no está resuelto

- No hay skill ni MCP para leer contenido detrás de login sin intervención humana — eso sigue siendo tarea manual del usuario en la ventana visible.
- Si algún día hace falta inspeccionar red/performance de una página (no solo interactuar), ahí sí entra `chrome-devtools-mcp` como añadido, no como sustituto.
