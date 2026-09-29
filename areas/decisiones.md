---
title: Registro de decisiones
created: 2026-09-08
updated: 2026-09-28
tags: [meta, arquitectura]
zona: tecnico
---

Decisiones de arquitectura del wiki y correcciones del usuario, con fecha. Consultar antes de proponer cambios estructurales.

## 2026-09-08 — Creación del wiki

- Patrón LLM Wiki de Karpathy. Tres capas: fuentes brutas inmutables, wiki mantenido por el LLM, esquema en `AGENTS.md`.
- Metodología PARA: `proyectos/`, `areas/`, `recursos/`, `archivo/`. Más `fuentes/` para material bruto e `inbox.md` para captura.
- `AGENTS.md` como fuente única de verdad; `CLAUDE.md` sólo lo importa. Objetivo: compatibilidad con Codex y Gemini CLI.
- Clasificación automática, sin preguntar al usuario. Enlace bidireccional obligatorio. Índices por carpeta obligatorios desde la primera nota.
- Formato de nota: frontmatter YAML plano (title, created, updated, tags, zona), resumen de una línea, cuerpo. Ficheros cortos y monotemáticos.
- Operaciones: ingesta, consulta (`/buscar`), lint con síntesis proactiva.
- Subagentes: `archivista` (ingesta e inbox), `auditor` (lint y síntesis).
- Umbral de subdivisión de carpeta: 30 notas.

## 2026-09-09 — Estructura de proyectos con alcance abierto

- Cuando un proyecto arranca sin saber qué apartados tendrá, crear solo la nota hub del proyecto con una lista de "frentes de trabajo". Cada frente se saca a su propia nota corta cuando se aborda, no antes.
- Aplicado a [[apartamentos-calle-uruguay]].

## 2026-09-14 — Renombrado del proyecto del dúplex de Carballo

- El proyecto "Alquiler dúplex dividido en dos apartamentos" pasa a llamarse **Apartamentos Calle Uruguay**, nombre más simple ligado a la dirección (R/ Uruguay 2-4, Carballo). Fichero renombrado a `apartamentos-calle-uruguay.md`, actualizados los enlaces entrantes.
- Nuevo frente abierto en el hub: portero automático que no debe molestar a los dos inquilinos a la vez. Sin nota propia todavía — se crea cuando llegue el análisis, siguiendo la regla del apartado anterior.

## 2026-09-19 — Escalado de búsquedas web y navegador por CDP

- Investigación y decisión: ver [[entorno]] para el detalle de MCPs evaluados.
- Skills nuevas, organizadas por capacidad y no por sitio: `/investigar-web` (capas 1-2, `WebSearch`→`WebFetch`, autónoma) y `/navegador-cdp` (capa 3, Chrome real por CDP). Descartado un tercer skill `precio-producto`: el usuario decidió que la comparación de precio/producto es solo un criterio de búsqueda dentro de `investigar-web`, no una capacidad aparte — dos skills bastan.
- MCP elegido para el paso 3: **Playwright MCP** (Microsoft), conectado a un Chrome ya corriendo vía `--cdp-endpoint`. Descartado `chrome-devtools-mcp` (Google): también conecta por CDP a un navegador existente y cumple el criterio eliminatorio, pero está orientado a depurar (red, performance, consola) y no a conducir el navegador; cada tool de más consume contexto en cada turno sin aportar a este caso de uso. Descartado Puppeteer MCP (deprecado/archivado) y navegadores cloud tipo BrowserBase (lanzan su propio navegador, no el existente).
- Descartada la capa de datos de producto/precio sin scraping (Keepa API, SerpApi): requieren API key de pago y el usuario decidió no incluirla por ahora. Si se revisita, evaluar bajo el mismo criterio de "herramienta existente antes que script improvisado".
- Registrado en `.mcp.json` del proyecto: `playwright-cdp`, con `--cdp-endpoint=${CDP_ENDPOINT:-http://192.168.1.5:9222}`. Limitación asumida: el endpoint se fija al arrancar el proceso MCP; cambiar de máquina dentro de la misma sesión no es posible, hace falta relanzar `claude` con `CDP_ENDPOINT` puesto a la IP de la otra máquina.
- Regla añadida a `AGENTS.md`: reutilización de skills — comprobar si ya existe una antes de resolver algo o de crear una nueva, y registrar en cada skill qué alternativa se descartó y por qué.

## 2026-09-19 — Corrección: el escalado a CDP es obligatorio tras un bloqueo

- Fallo detectado por el usuario: en una consulta de precios, `WebFetch` devolvió HTML vacío en Amazon y 403 en Leroy Merlin y Bauhaus, y se entregó «no verificado» en vez de escalar a `/navegador-cdp`, pese a que `investigar-web` ya listaba esos casos. Además se leyó un escepticismo del usuario («sin navegador real no creo que los saques») como prohibición.
- Correcciones aplicadas: `investigar-web` paso 3 pasa a obligatorio; nuevos pasos 4-8 (un bloqueo no es un resultado; solo una prohibición explícita suspende el escalado; el sitio que el usuario prioriza se resuelve primero; los resúmenes del buscador no son precio; informar por partes). `AGENTS.md` (sección Reutilización de skills) alineado.
- `navegador-cdp`: nuevo paso 0. Si `CDP_ENDPOINT` ya está definida y responde a `curl`, esa es la máquina elegida y no se pregunta otra vez; la pregunta a/b solo aplica si la variable no está definida. Es un cambio sobre el diseño original («se elige por consulta»); revertible si el usuario prefiere preguntar siempre.
- Sitios comprobados y selectores de Amazon: ver [[entorno]].

## 2026-09-19 — Puente de contexto entre sesiones (VSCode ↔ CLI)

- Verificado en docs oficiales: la extensión de VSCode y el CLI mantienen almacenes de sesión separados; `claude --continue`/`--resume` no puede recuperar una conversación de la otra superficie. Sin solución nativa.
- Solución: directorio `.claude/sesiones/` (gitignored, fuera del PARA, no lo tocan `/lint` ni las reglas de clasificación de `AGENTS.md`). Skills `/exportar-sesion` y `/importar-sesion` — ver [[entorno]].
- Por defecto se vuelca un resumen de estado, no la transcripción literal: cuesta menos contexto a la sesión receptora. La transcripción completa queda disponible solo si se pide explícitamente.

## 2026-09-19 — Corrección: `.claude/` es solo configuración; los resultados de trabajo van a PARA

- Fallo detectado por el usuario: resultados de trabajo (búsqueda de alojamiento en Londres, vuelos Vueling) se dejaron dentro de `.claude/sesiones/`. `.claude/` es configuración (commands, agents, settings), nunca contenido.
- Correcciones: contenido migrado a `proyectos/` ([[viaje-londres-diciembre-2026]] y sus notas, [[vuelos-sevilla-octubre-2026]]) y a `areas/` ([[vueling-busqueda-por-url]]), con frontmatter, enlaces e índices. Los adjuntos (65 fotos y un PDF) van a `fuentes/londres-alojamiento-2026-09-19/`, ignorados por git: son fotos de terceros y el repo es público. `.claude/sesiones/` borrado. Regla nueva en `AGENTS.md` (sección Repositorio y artefactos).
- Aclaración del usuario: una exportación (`/exportar-sesion`) sirve solo para pasar contenido a una sesión nueva y se hace únicamente cuando él lo dice. No obliga a repetirla, ni a mantenerla actualizada, ni tiene que ver con el wiki. Las skills de exportar/importar no se tocan. Como siguen escribiendo en `.claude/sesiones/`, ese directorio queda ignorado por git como red de seguridad; si el usuario prefiere otro destino, habrá que cambiar las skills.

## 2026-09-19 — Corrección: artefactos de herramienta versionados y publicados

- Fallo detectado por el usuario: `.playwright-mcp/` (logs, capturas y snapshots del navegador) no estaba en `.gitignore` y `cerebro-sync.sh` (cron horario, `git add -A` y push) lo subió a GitHub: 140 ficheros, 11 MB, en los commits de las 19:00, 20:00, 21:00 y 22:00 UTC del 2026-09-19.
- Hallazgo: el repo `blogNetting/2cerebro` es público. Lo publicado con datos personales: nombre completo y nivel Genius de la cuenta de Booking; nombre y email de una cuenta de Google; la línea de envío de una cuenta de Amazon (nombre, localidad y código postal); 6 URLs de inicio de sesión de Booking con `op_token` en un log. Sin cookies, JWT, claves ni contraseñas. Las 5 capturas son páginas públicas de Vueling.
- Correcciones: `.playwright-mcp/` y `*.log` en `.gitignore`; carpeta desindexada y borrada; `.mcp.json` con `--output-dir=/home/netting/.cache/playwright-mcp` para que Playwright MCP escriba fuera del repo (aplica al reiniciar la sesión); regla en `AGENTS.md`: todo directorio de caché o artefacto va a `.gitignore` en cuanto aparece; comprobación nueva en `/lint` de ficheros versionados que no deberían estarlo.
- El historial de GitHub no se ha reescrito: pendiente de decisión del usuario.

## 2026-09-20 — Regla permanente: formato de investigaciones

- Petición del usuario tras una búsqueda de coche de alquiler Córdoba → A Coruña en la que faltaban enlaces («como siempre me faltan enlaces»). Toda investigación se entrega con una estructura fija: introducción, considerado y descartado (con motivo), análisis detallado y visual, recomendaciones, dónde se ha buscado (incluidas las fuentes que no aportaron) y otros aspectos relevantes.
- Fallo de fondo: los enlaces se omiten o se dejan solo al final. La regla exige un enlace por cada afirmación con fuente, en el punto donde aparece, además de URL directa por cada opción, y una revisión final antes de entregar. Dato sin fuente enlazable: «sin verificar» u omitido.
- Salida: `.md` por defecto, como nota del wiki según PARA; PDF solo si el usuario lo pide, generado desde el `.md`.
- Recogido en `AGENTS.md`, sección «Formato de investigaciones». Complementa `/investigar-web` (cómo buscar); no lo sustituye.

## 2026-09-24 — Regla permanente: el wiki no alberga proyectos

- **El código de cada proyecto vive en su repo propio, nunca en `2cerebro`** (que es público y se publica cada hora). En el wiki, cada proyecto tiene **una nota hub corta** que dice qué es y enlaza a su repo.
- **Y por extensión, tampoco su seguimiento**: plan, bitácora, estado, versiones, PRs, rutas locales y pendientes van en el repo del proyecto. **El wiki es conocimiento y método**, no el sitio donde se lleva un proyecto.
- **Regla dura del usuario: no hay código sin tests**, unitarios o de integración.
- Aceptado usar DeepSeek por su API oficial; que los datos estén en China no es un problema. El coste se controla con saldo prepagado y midiendo el consumo.

**Incumplida desde el mismo día, y señalada por el usuario el 2026-09-29:** *«el problema sois vosotros que mezcláis 2cerebro con proyectos independientes»*. Durante esos días el wiki acumuló el plan, la bitácora, el diseño por fases y las subidas de versión de un proyecto que tenía repo propio. El detalle de la corrección y de lo que se hizo, en la entrada del 2026-09-29.

## 2026-09-24 — Herramientas de investigación técnica

- Instalados `gh` y `yt-dlp` en `~/.local/bin`, binarios oficiales con el checksum verificado. No hay sudo; se descartó `apt`. Se descartaron también los MCP de GitHub y de YouTube: la CLI cubre lo mismo sin gastar contexto en definiciones de tools en cada turno.
- `/investigar-web` tiene un paso 10 nuevo con fuentes directas: GitHub, la API de Algolia de Hacker News y transcripciones de YouTube. Reddit y X van directos a CDP. El deep research de otras IAs solo sirve para descubrir pistas. Ver [[entorno]].

## 2026-09-25 — Regla permanente: lo que menciona el usuario no condiciona nada

- Petición expresa del usuario («escríbelo en sangre»), tras varias repeticiones del mismo fallo: tomar lo que él nombra (DeepSeek, issues, gitflow, Beads…) como premisa o como centro del trabajo.
- Se añade la sección «Lo que menciona el usuario no condiciona nada» a `AGENTS.md`, junto con la regla de explicar cada sigla al usarla (ajuste de «no expliques fundamentos»).

## 2026-09-26 — Regla permanente: autoevaluación obligatoria antes de decir que algo está hecho

- El usuario, con una entrega mía delante como prueba de que no llegaba al nivel, dijo directo: **«me has mentido»**, y pidió una regla permanente de autoevaluación — comparar lo pedido con lo entregado antes de decir que algo está terminado, y **no esconder si no llega al nivel**. Es el origen de la regla «sé tu propio juez y tu propio verdugo» de `~/.claude/rules/comportamiento.md`.
- Corrección de forma que acompaña: ante una pregunta amplia, traer una propuesta concreta; explicar cada tecnicismo la primera vez.
- **Aprendizaje que el usuario dejó explícito y que sigue vigente:** una entrega que cumple la forma pero no la madurez esperada **no se envía** — se detecta, se dice y se corrige ahí mismo.

## 2026-09-26 — Fallo real: las reglas de comportamiento nuevas se escribieron solo en `AGENTS.md`, repo-scoped

- El usuario preguntó directo: «¿cómo metes eso en AGENTS, tendrás que meterlo en 2Cerebro [global] verdad?» — señalando que había cometido el mismo fallo de scope que ya había aprendido horas antes con una skill puesta en el repo (repo-scoped en `.claude/commands/` no sirve: tiene que funcionar en cualquier sesión) y no lo apliqué a las reglas de comportamiento. `AGENTS.md` de `2cerebro` solo se carga cuando Claude Code abre ese directorio — una sesión trabajando en otro repo nunca recibía esas reglas.
- Verificado con el agente especializado en Claude Code, no asumido: existe `~/.claude/rules/` (ficheros `.md`, se carga en toda sesión sin importar el directorio, sin tocar `settings.json`).
- Creado `~/.claude/rules/comportamiento.md` con las cuatro reglas de comportamiento generales duplicadas (no las de gestión del wiki, esas sí son específicas de `2cerebro`) — «lo que menciona el usuario no condiciona nada», «no decir ya está sin comprobarlo», «toda pregunta se contesta», «sé tu propio juez y tu propio verdugo». Referencia cruzada añadida en `AGENTS.md` para que quede documentado en los dos sitios.
- Por qué pasó, dicho claro: tenía la información correcta desde hacía horas (el mismo aprendizaje sobre el skill, en esta misma conversación) y no la apliqué al escribir las reglas nuevas — un lapso del propio criterio de autoevaluación que se estaba escribiendo en ese momento.
- **Corrección inmediata del usuario, dos partes.** (1) Quitados del todo los huecos que faltaban en `~/.claude/rules/comportamiento.md`: explicar cada tecnicismo la primera vez (fallo repetido hoy: "bootstrapear", "dogfoodear"), que un ejemplo del usuario no es una exigencia literal (sub-punto explícito, no implícito), el bucle de autoevaluación como algo iterativo y no de una sola pasada — juzgar, decir explícito si no llega a la madurez esperada, corregir, volver a juzgar, repetir hasta que sí llegue — y una regla contra el relleno dramático (contar el hecho y el motivo, nunca el número de vez). (2) Quitada la duplicación en `AGENTS.md`: si `~/.claude/rules/` se carga siempre, tenerlo también en `AGENTS.md` es redundante y se puede desincronizar — sustituido por un puntero único, con solo las reglas de investigación genuinamente específicas de este wiki (subagentes, no abrir frentes laterales) quedando aquí.

## 2026-09-26 — Corrección: 2-3 fuentes no es "evolución continua"; búsqueda sistemática y regla de revisión periódica extendida

- Pregunta directa del usuario: de dónde sale la información de lo investigado hoy (ejemplos de `AGENTS.md` reales), porque 2-3 repos de referencia elegidos por conocerlos no es suficiente, quiere evolución continua de análisis.
- Contestado exacto: los ejemplos de `openai/codex` y `apache/airflow` se eligieron por ser proyectos ya conocidos, no por búsqueda sistemática — es una muestra de conveniencia de un momento, no un método. Corregido en el momento: búsqueda real con `gh search code "filename:AGENTS.md"`, filtrada por estrellas reales (no orden aleatorio del buscador), encontrados dos ejemplos más, independientes, nunca vistos antes por mí — `nextdns/nextdns` (4.187★, estilo "Core Principles") y `diffblue/cbmc` (1.137★, manual extenso con tabla de contenidos). Confirman el mismo hallazgo (no hay un único patrón de longitud/formato correcto) con una base de evidencia real más ancha, no solo más grande.
- **Regla nueva, extiende una ya existente en vez de crear otra:** la "revisión periódica" que `AGENTS.md` ya exige para las skills (sección «Reutilización de skills») se extiende explícito a todo el contenido investigado puesto en producción (plantillas de Astillero, elecciones de herramienta con evidencia) — en cada `/lint`, o al retocar una de estas piezas, repetir la búsqueda de forma sistemática, no de memoria ni con los mismos nombres conocidos, y actualizar el fichero real si algo cambió, no solo anotarlo.

## 2026-09-26 — Corrección: tratar una ficha "rara" como ruido a descartar, en vez de preguntar

- Investigando grifería de ducha gris para el alquiler de Carballo ([[duchas-alquiler-carballo]]), varios productos traían el título en "gris" pero la ficha técnica decía "Cromo". Lo traté como contradicción/señal de alarma y descarté el candidato — incluido el catálogo entero de Roca, Ramon Soler, Tres y Drake en Obramat (62-112 €), que encajaba en presupuesto. Conclusión errónea entregada al usuario: "no hay gris bueno por debajo de 90-100 €".
- Corrección directa del usuario: el cromado **es** un gris (plateado brillante) — no hay contradicción ninguna, la señal rara era mi propio criterio equivocado, no el producto. Pedido explícito: cuando algo no queda claro o resulta raro, se pregunta — tanto antes de arrancar la investigación como en medio, en el momento en que se detecta lo extraño, en vez de resolverlo por cuenta propia y seguir adelante con una conclusión que podía estar construida sobre un malentendido.
- Corregido: relanzada la investigación incluyendo cromado como gris válido. Añadida regla permanente en `~/.claude/rules/comportamiento.md` («Preguntar ante lo ambiguo o lo raro, no resolverlo por cuenta propia»), porque es comportamiento general y no específico de este wiki.

## 2026-09-26 — Corrección: "buenos materiales" exige investigar la categoría antes de buscar productos, no filtrar de memoria

- Sobre la misma investigación de grifería gris ([[duchas-alquiler-carballo]]): el usuario, con razón, calificó los resultados de "una vergüenza" — ningún candidato presentado tenía de verdad buenos materiales verificados; se habían presentado como "recomendación" opciones con reseñas de grietas, roturas y garantía real inferior a la anunciada, por ser las menos malas de las encontradas.
- Corrección explícita, de aplicación general (no solo a este wiki): cuando se pide un criterio cualitativo como "buenos materiales", el orden correcto es (1) investigar la categoría de producto en sí — qué materiales/acabados existen, cuáles son malos con evidencia, cuáles aceptables, cuáles buenos — antes de mirar ningún producto concreto; (2) usar eso como filtro de admisión duro, no como comentario después de elegir; (3) si nada dentro del presupuesto lo cumple de verdad, decirlo así, sin fabricar un ganador entre opciones que no cumplen.
- Añadida regla permanente en `~/.claude/rules/comportamiento.md` («Investigar el criterio de calidad antes de buscar candidatos, no filtrarlo de memoria»), por ser comportamiento general y no específico de este wiki.

## 2026-09-26 — Dos correcciones directas: GitHub Pro nunca decidido, prompt de entrevista en inglés sin motivo

- **GitHub Pro presentado como pendiente/decidido sin que el usuario lo pidiera nunca.** Corrección directa: «¿en qué momento te he dicho yo que vamos a contratar GitHub Pro?». Es un hallazgo técnico condicional (si se quieren rulesets/environments en repos privados, hace falta) — nunca una decisión de compra. Corregido en `astillero-replicacion.md`, `flujo-agentes-arquitectura.md` §15 y `flujo-agentes-runbook.md` §2: quitada la palabra "pendiente" y "ya decidido", y la recomendación implícita de pagarlo — queda como dato técnico neutro, el sistema funciona igual sin esa puerta.
- **El prompt real de la entrevista (Fase 4 del skill) se copió literal de Anthropic, en inglés, sin traducirlo.** El usuario, con razón: todo lo demás está en español, no hay motivo técnico para que esta pieza no lo esté — "cita literal" era para no perder precisión de las instrucciones, no para forzar inglés en una herramienta que se usa en español. Traducido fielmente en `~/.claude/skills/astillero-proyecto/SKILL.md` Fase 4 paso 0, conservando cada instrucción; el original en inglés queda solo en `capa-producto.md` §2 como referencia de la fuente.

## 2026-09-26 — Los ficheros de comportamiento solo llevan reglas generales

- Corrección del usuario: otros agentes habían metido casos concretos (grifería de Carballo, el cromado, productos y precios de Amazon, fechas de correcciones, nombres de proyectos) dentro de las reglas de comportamiento y de búsqueda. Esas reglas deben decir qué aplica siempre, no contar el caso que las originó.
- Limpiado: `~/.claude/rules/comportamiento.md` (regla de criterio de calidad reescrita en general, sin «materiales/reseñas»; fuera ejemplos de jerga y nombres de proyectos; corregida su cabecera, que se describía como copia de `AGENTS.md` cuando es la fuente única), `AGENTS.md` (anécdota fechada fuera de la regla de revisión periódica), `.claude/commands/investigar-web.md` (lista de tiendas sustituida por un puntero a [[entorno]]), [[entorno]] (fuera anécdotas de productos; se quedan los selectores y filtros por sitio, que sí sirven para cualquier búsqueda futura).
- Criterio que queda: el incidente que motiva una regla se anota aquí, en `decisiones.md`; en los ficheros de comportamiento, solo la regla.

## 2026-09-26 — Verificación obligatoria de cada respuesta antes de entregarla

- Corrección del usuario, pedida muchas veces: las respuestas no llegan al nivel exigido (se piden 3 de algo y 4 de otra cosa y se entregan 2 y 2) y nadie lo detecta antes de entregar. Tener la regla escrita no bastaba, porque el modelo puede saltársela.
- Solución: un hook `Stop` global (`~/.claude/hooks/verificar-respuesta.sh`, registrado en `~/.claude/settings.json`) que ejecuta Claude Code, no el modelo. Extrae del transcript la última petición y la respuesta, y un evaluador (Sonnet por `claude -p`) decide si cumple: cantidades completas, todas las preguntas contestadas, nivel acorde a lo pedido, pruebas en vez de «ya está». Si no cumple, bloquea el cierre con el motivo; máximo 3 bloqueos por petición, y después avisa al usuario de lo pendiente.
- **Corregido el mismo día:** el primer motivo de descarte era falso. La skill `update-config` decía que los hooks `prompt` y `agent` solo existen en eventos de herramienta; la documentación oficial (code.claude.com/docs/en/hooks.md, sección «Prompt-based hooks») los lista como válidos en `Stop`. Evaluación real de las tres opciones: `prompt` nativo, descartado porque la entrada de `Stop` no incluye la petición del usuario y ese tipo no puede leer el transcript; `agent` nativo, alternativa seria (lee el transcript y puede comprobar ficheros), pero solo se puede probar en vivo, es más lento y caro por turno y no tiene límite propio de reintentos (el de Claude Code es de 8 seguidos); `command` actual, se mantiene por estar probado. En GitHub no hay nada maduro que lo haga (lo más cercano, `valentynkit/jev-belay`, 18★).
- Criterio añadido al evaluador a petición del usuario («no me vale una respuesta cualquiera sin saber si hay opciones mejores»): si se propone, elige o construye una solución no trivial sin mostrar qué alternativas se consideraron y por qué la elegida es mejor, no cumple. Probado: una única opción sin comparar se bloquea, la misma con alternativas comparadas pasa, y una orden concreta ya decidida pasa. Regla escrita a juego en `~/.claude/rules/comportamiento.md` («No vale una respuesta cualquiera»).
- Probado antes de conectarlo: bloquea 2 de 3 pedidos, deja pasar «espera» y la respuesta completa, detecta enlaces truncados o inventados, y respeta el límite de 3.
- Coste asumido: una llamada a Sonnet por turno contra la cuota Pro, y de 5 a 15 s más por respuesta. Limitación: solo ve el texto final, no puede comprobar si un dato es cierto, solo si la respuesta cubre lo pedido con evidencia visible.
- Regla escrita a juego en `~/.claude/rules/comportamiento.md`, sección «Sé tu propio juez».
- **Fallo del propio hook, encontrado al usarlo en vivo (mismo día):** al saltar `Stop`, el último bloque de texto de la respuesta todavía no está escrito en el transcript, así que el revisor veía la respuesta cortada y bloqueaba sin razón (dos falsos bloqueos en la sesión). Corregido: el script añade `last_assistant_message`, que Claude Code (2.1.283) pasa directo en la entrada del hook. Probado con un transcript cortado: con el campo aprueba, sin el campo bloquea. La última entrada recibida se guarda en `~/.cache/verificar-respuesta/ultima-entrada.json` para comprobarlo.
- **Segundo fallo del hook, también encontrado en vivo:** el mensaje final seguía sin llegar al revisor aunque Claude Code sí lo pasaba en `last_assistant_message` (comprobado en `ultima-entrada.json`: 3.080 caracteres). La causa era que se comprobaba si ya estaba incluido con `grep -F`, y `grep -F` parte el patrón por líneas: una línea en blanco en el mensaje coincide con cualquier texto, así que nunca se añadía. La prueba anterior no lo detectó porque usaba un mensaje de una sola línea. Corregido con una comparación literal de cadenas, y si hay que recortar se conserva el final (`tail -c`), no el principio. Probado con un mensaje de varias líneas con líneas en blanco: con el campo aprueba, sin él bloquea.
- **Alcance limitado a investigaciones (decisión del usuario, mismo día):** fuera de las investigaciones el revisor no era fiable. En la prueba bloqueó un resumen corto por «largo» y aprobó un volcado de 12.538 bytes. Ahora primero clasifica la petición y solo evalúa si es una investigación: comparativas, búsqueda de productos u opciones, evaluar alternativas, pedir una forma o solución que admite varias opciones, buscar información en fuentes. Las preguntas rápidas, explicaciones, resúmenes, órdenes concretas y la conversación pasan sin evaluar. Probado con 8 casos: resumen, volcado largo, «espera» y renombrar pasan; 2 de 3 pedidos y una sola opción de backup sin comparar se bloquean (esta última igual en dos pasadas); las respuestas completas pasan. La entrada de depuración pasa a ser un fichero por sesión (`ultima-entrada-<sesión>.json`), porque la compartida la sobrescribían otras sesiones.

## 2026-09-27 — Verificación de citas: regla nueva, sacada de dónde fallan de verdad los sistemas de investigación

- La investigación completa, con el criterio de admisión, los candidatos y sus motivos, está en [[sistemas-de-research-con-agentes]]; las herramientas ya montadas, en [[entorno]].
- Origen: investigación pedida sobre el mejor sistema de research/búsqueda con agentes. De todo el barrido (X por CDP, repos de laboratorios de primera línea, frameworks, comunidad), la conclusión consistente es que **el fallo dominante no es encontrar, es que la cita no sostenga la frase**. En el único estudio que lo localiza agente por agente (Hirsch et al., EMNLP 2026, [arXiv:2608.24306](https://arxiv.org/abs/2608.24306)), el **84,7 % de los errores del informe final se originan en el redactor**, no en la búsqueda, y la única configuración sin errores de cita dominantes es la que **resume documento a documento**.
- Añadido a `AGENTS.md`: sección nueva «Verificación de citas» y una línea de enlace en «Formato de investigaciones». Las cuatro reglas: guardar la **cita textual** junto a la afirmación (no solo el enlace), comprobar mecánicamente que ese texto aparece en la fuente, resumir **documento a documento** antes de sintetizar, y dejar las contradicciones visibles. Complementa «Formato de investigaciones» (que ya exigía enlace por afirmación), no lo sustituye.
- **No se instaló nada de terceros.** Se evaluó `claude-obsidian` (15.231★, MIT, plugin de Claude Code, sin GPU ni Docker) y se descartó: la mayor parte de lo que aporta —fuentes inmutables, detección de contradicciones, índices, lint— ya está en este esquema desde el 2026-09-08, y lo único que faltaba de verdad (el registro de afirmaciones con cita textual) se escribe como regla. Meter código de un particular en un repo público con push horario no se justifica por esa diferencia.
- **La capa de búsqueda no se sustituye por ningún producto, y esto es lo que decide:** los sistemas con mejor benchmark medido (Tongyi DeepResearch, MiroThinker, DR Tulu de AI2, NVIDIA AI-Q) **no corren en esta máquina** (4 núcleos, 7 GB de RAM, sin GPU ni Docker). Lo que se usa es `/investigar-web` con su escalado obligatorio a `/navegador-cdp` para descubrir, y el deep research de otras IAs solo para descubrir pistas — como ya decía la entrada del 2026-09-24.
- Descartes con motivo, de los candidatos que sí tienen evidencia fuerte: **NVIDIA AI-Q** es el mejor sistema abierto de DeepResearch Bench (55,95, por encima de OpenAI y Gemini) pero el estudio de EMNLP mide que su orquestador origina el 84,7 % de los errores finales — el leaderboard premia el informe bien formado, no la fidelidad. **PaperQA2** sigue siendo el mejor avalado para literatura científica, con la pega de que el corpus de pago con el que logra sus números no es abierto. **deer-flow** (83.005★) no tiene un solo benchmark de terceros. **hyperresearch** dice liderar DeepResearch Bench y no aparece en la tabla pública: comprobado en el CSV del leaderboard.
- Nota de método, por si se repite este tipo de búsqueda: **X no sirve como fuente de validación** en esta categoría. De 19 búsquedas con el navegador real, las únicas con respaldo verificable fueron las hechas por nombre de paper o institución; las genéricas devolvieron listículos y cuentas con etiqueta de partnership pagado. Y la auditoría de comunidad más citada de la categoría (un hilo de r/LocalLLaMA) tiene un dato falso medible: atribuye 173 issues abiertos a `gpt-researcher` cuando la API de GitHub da **5**, con un 96,7 % de respuesta a los issues de 2026.

## 2026-09-27 — Método de investigación: skill global nueva y refuerzo del verificador

- Petición del usuario, tras comprobar que la investigación falla «en todo»: no encuentra lo bueno, se queda en la superficie y la entrega no llega. Diagnóstico primero, con tres opciones marcadas por él (descartó «datos o citas falsas»). Su descripción literal: **«solo coge lo fácil y cumple con lo mínimo»**.
- **Hallazgo que cambia el enfoque:** eso no es un problema de herramienta ni de modelo. Es el comportamiento por defecto del que busca, documentado desde **Zipf (1949)** —se usa el método más cómodo y se para en lo mínimamente aceptable— y medido en personas (Liu & Lang, 2004). Tiene nombre técnico propio: *search satisficing* y *premature termination* (MAST: 6,20 %). Y **pedir «busca mejor» no lo corrige**: en el estudio que lo midió, los agentes hacen comprobaciones superficiales *«despite being prompted to perform thorough verification»*.
- **Corrección de un consejo mío anterior, retirada por el usuario:** atribuí los malos resultados al modelo (DeepSeek frente a Opus/Sonnet). El usuario lo desmintió con su experiencia directa — con Opus y Sonnet también fallaba, y con DeepSeek le va mejor. Los fallos documentados en este mismo registro (grifería, cromado, enlaces) son de método, no de modelo. El consejo no se sostiene.
- **Creada la skill global `~/.claude/skills/investigar-metodo/SKILL.md`** (14 pasos: 4 antes de buscar, 5 durante, 5 al cerrar), por la misma razón de scope que `astillero-proyecto`: toda investigación la necesita, se haga desde el wiki o desde cualquier proyecto. Evidencia y cifras en [[metodo-de-investigacion]].
- **Reforzado el hook `verificar-respuesta.sh`**, que tenía dos agujeros por los que se colaba justo este fallo: (1) definía «superficial» como «sin fuentes enlazadas», así que un volcado bien enlazado y sin análisis pasaba — ahora no cumple si se limita a describir sin aplicar criterios, sin decir qué descarta y por qué, o sin conclusión propia; (2) **no comprobaba si se había buscado algo** — ahora lee de la transcripción las herramientas usadas tras la petición y bloquea si es investigación y no se usó ninguna de búsqueda, lectura o ejecución sin apoyarse en evidencia ya recogida. Probado sobre la transcripción real de esta sesión.
- **Pendiente y dicho explícito:** la estructura obligatoria de 6 secciones que exige `AGENTS.md` **no** se comprueba en el evaluador todavía. Si la entrega falla en el chat y no en la nota, se añade.
- Descartes con motivo: no se instala ninguna herramienta nueva para esto (los sistemas con mejor evidencia no corren en esta máquina, ver [[sistemas-de-research-con-agentes]]), y no se copian las técnicas de inteligencia tal cual — el propio ACH reconoce carecer de evidencia empírica fuerte, así que se adoptan sus principios (trabajar a lo ancho, refutar en vez de confirmar), no su ceremonia.
- **Cuatro barridos en paralelo** (tradecraft de inteligencia, ciencia de la información, por qué los agentes de IA paran pronto, y método de periodismo/historia/medicina/auditoría/policía). El hallazgo que sostiene todo el método, porque está medido en LLM y es revisado por pares: Lee, Kang y Shin, **ACL 2026** — las técnicas de investigador aplicadas a modelos de lenguaje **superan a las estrategias generalistas en los 10 modelos probados**, y **la etapa de estructuración de hipótesis explica el 89 % de la mejora**. Su conclusión: *«structures that specify "what to analyze" are more effective than simply increasing the number of reasoning paths»*. Es la razón de que este método sean pasos y no consejos.
- **El límite que se declara en la propia nota y que no se debe maquillar:** las técnicas de análisis estructurado de la comunidad de inteligencia tienen **validez aparente, no empírica demostrada** (RAND RR1408; Cheikes 2004, Tetlock 2005, Nemeth 2001 van en contra). Se adoptan por diseño lógico. Lo que sí está medido es lo otro: que el mínimo esfuerzo es el comportamiento por defecto, que la conciencia del sesgo no lo corrige, y que las intervenciones concretas mueven entre 6 y 17 puntos.
- **Corrección de una cita mía, encontrada en el propio proceso:** atribuí el estudio sobre el mínimo esfuerzo a «Liu y Lang (2004)». El trabajo real es **Liu y Yang (2004)**, *Journal of Academic Librarianship* 30(1) — el apellido es Yang, no Lang, y el error viene de copiarlo de Wikipedia sin ir al DOI. También se descartó una atribución que el encargo daba por buena, «Prabha, Bunge y Saracevic sobre satisficing»: **no existe**; el clásico real es Prabha, Connaway, Olszewski y Jenkins (2007). Corregido en [[metodo-de-investigacion]] y en la skill. Queda como recordatorio de la propia regla: la fuente secundaria se verifica contra el DOI antes de citarla.

## 2026-09-27 — Lint de contenido: enlaces, huérfanas, índices, frontmatter, y dos notas nuevas (síntesis y contradicción)

- Recorrido completo de `proyectos/` (incluida la subcarpeta `astillero/`), `areas/`, `recursos/` y `archivo/`, con `fuentes/` fuera de alcance (solo consultada, no modificada). Resultado: **0 enlaces rotos** (los 74 ficheros se comprobaron wikilink a wikilink), **0 notas duplicadas sin enlazar**, **0 índices desactualizados** (los siete `_index.md` listan exactamente las notas de su carpeta, con descripción) y **frontmatter completo** en las 54 notas reales (title, created, updated, tags y `zona` presentes; `zona` siempre `tecnico` o `general`; ninguna con `updated` anterior a `created`).
- **1 nota huérfana corregida:** `recursos/smartwatch-mujer-muneca-pequena` (sus únicos enlaces entrantes eran de índices). Enlazada en ambas direcciones con [[metodo-de-investigacion]] (aplica la regla de criterio-antes-que-candidatos) y desde [[entorno]] (barrido por CDP sobre Amazon).
- **Nota de síntesis creada:** [[verificacion-externa-agentes]] — el tema «la verificación solo cuenta si la posee algo externo al agente», recurrente en [[desarrollo-autonomo-con-agentes]], flujo-agentes-arquitectura, flujo-agentes-evidencia-empirica, flujo-agentes-informe y la-fabrica, sin nota propia hasta ahora.
- **Nota de contradicción creada:** [[contradiccion-agents-md]] — flujo-agentes-informe §9.5 y [[desarrollo-autonomo-con-agentes]] §3.quinquies citan el mismo paper (arXiv:2602.11988) con cifras y conclusión incompatibles sobre `AGENTS.md`/`CLAUDE.md` (+4 % frente a +2,4 % sin significación). No la resuelve el lint: exige leer el paper, lo decide el usuario.
- Ambas notas quedan enlazadas en ambas direcciones con las notas implicadas y listadas en `areas/_index.md`.
- **Crecimiento — corregido un cálculo mío, y por eso NO se toca nada todavía.** En una primera pasada conté los ficheros de `proyectos/astillero/` (29 notas + `_index.md` = 30) y di la carpeta por encima del umbral. La regla de `AGENTS.md` dice «cuando una carpeta **supere 30 notas**», y hay **29 notas**: el índice no es una nota, así que el umbral **no está superado**. Verificado con `ls proyectos/astillero/*.md | grep -v _index.md | wc -l` → 29. No se subdivide; se dispara con la **próxima** nota que entre en esa carpeta (inminente — la-fabrica es de hoy). La subdivisión queda preparada abajo, para ejecutarla en cuanto se cruce el umbral, no antes.
  - `astillero/investigacion/` — las 18 notas de investigación: `desarrollo-agentes-*` (5), `orquestacion-*` (5), `flujo-fase-*` (6), `flujo-agentes-informe`, `circuito-tareas-definicion`.
  - `astillero/diseno/` — las 9 de diseño operable: `flujo-agentes-arquitectura`, `flujo-agentes-runbook`, `flujo-agentes-evidencia-empirica`, `flujo-agentes-prueba-descomposicion`, `capa-producto`, `capa-producto-definicion`, `devops-minimo`, `astillero-replicacion`, `astillero-mantenimiento`.
  - Raíz de `astillero/`: `astillero` (hub) y `la-fabrica` (despiece).
  - Los wikilinks no se rompen con el movimiento: son por nombre de nota, sin carpeta.

## 2026-09-27 — Lint de artefactos y datos: un PDF generado fuera de sitio, y PII del usuario en el historial publicado

- **`recursos/smartwatch-mujer-muneca-pequena.pdf` estaba versionado** (292 KB, generado desde el `.md` de la misma nota el 2026-09-23, publicado en `924506d`). Contradecía la regla ya establecida: los PDF generados son entregables, no contenido del wiki — el `.md` es lo que se versiona (`AGENTS.md`, «Formato de investigaciones»), y el PDF de Londres del 2026-09-19 se había puesto ya en `fuentes/londres-alojamiento-2026-09-19/`, ignorado por git. **Corrección:** `git rm --cached` del PDF (el fichero sigue en disco), y regla nueva `*.pdf` en `.gitignore` para que el caso no se repita con la próxima comparativa entregada en PDF. No se reescribe el historial: sigue publicado en `924506d`. Comprobado después: `git ls-files -ci --exclude-standard` no devuelve nada.
- **Hallazgo nuevo, no registrado hasta ahora: hay datos personales del usuario en el historial ya publicado de `origin/main`.** Dos snapshots de accesibilidad de Google, subidos por el cron el 2026-09-19 (`01f383f`, «auto: 2026-09-19 20:00») dentro de `.playwright-mcp/`, contienen el nombre real y la dirección de correo personal del usuario en el texto del botón de cuenta de Google. Los valores no se repiten aquí a propósito: este repositorio es público y escribirlos sería volver a publicarlos. Alcanzables desde `origin/main` (verificado con `git merge-base --is-ancestor`).
  - **Barrido completo de esos 140 ficheros** (buscando DNI, IBAN, teléfonos, cookies, `session_id`, `Bearer`, correos): lo único personal son esas dos apariciones. El resto son metadatos públicos (fichas de hotel de Booking, capturas de Vueling) y direcciones de contacto de empresas.
  - **Análisis, con el dato que decide:** el correo de autor de **los 41 commits** —publicados y sin publicar— es la cuenta real del usuario. Está en el campo `author` de cada commit desde el primero (2026-09-08) y `origin/main` lo sirve así (`git log --format='%ae' origin/main` devuelve ese único valor). Es decir: **una dirección personal ya está publicada por diseño en todo el historial**. Reescribir 41 commits para sacar una *segunda* dirección —la de scraping, la que aparece en los dos snapshots— mientras la principal se queda en todos los campos de autor no protege nada: es incoherente, y el coste no es cero (árbol limpio, parar el cron horario y force-push sobre un repo que otra sesión está editando ahora mismo).
  - **Decidido: NO se reescribe el historial.** Motivos, por orden: (1) no aporta la protección que aparenta, por lo anterior; (2) `AGENTS.md` lo prohíbe explícitamente («no reescribas el historial sin que el usuario lo pida») y una delegación genérica no es una petición; (3) el vector de fuga ya está cerrado, así que no hay urgencia (abajo).
  - **Lo que sí se cierra y se decide:** el origen del problema era que el MCP escribía los snapshots dentro del repo (`./.playwright-mcp`, por defecto) y el cron los subía. Ya está cambiado: `.mcp.json` lanza el MCP con `--output-dir=/home/netting/.cache/playwright-mcp`, **fuera del repositorio** (comprobado: `git check-ignore` responde «is outside repository»), y `.playwright-mcp/` y `*.log` están en `.gitignore`. Detección: la comprobación de datos sensibles del `/lint` barre correos, credenciales, DNI/IBAN y teléfonos en lo versionado, que es por donde volvería a salir si un snapshot acabara dentro de una nota.
  - **Lo que queda en manos del usuario, y no lo hago yo:** la cuenta de scraping —la que aparece en esos dos snapshots— estuvo pública más de una semana y hay que darla por comprometida: lo lógico es rotarla o abandonarla. Y si la exposición de la cuenta real en los campos de autor molesta, la palanca no es reescribir el historial sino la visibilidad del repositorio — pasarlo a privado lo corta entero y es reversible con un clic. Eso cambia una decisión ya tomada (el wiki está pensado público, y `AGENTS.md` lo da por supuesto en varias reglas), así que lo decide el usuario, no el lint.

## 2026-09-27 — Las reglas se hacen cumplir con hooks, y ninguno de los dos estaba haciendo nada

- **El hallazgo, y es el que importa.** Al ir a montar un hook se comprobó que **ninguno de los mecanismos de cumplimiento estaba activo**:
  - `~/.claude/settings.json` solo tenía una entrada `PermissionRequest` que era un `echo` devolviendo **`allow` para todo** → **auto-aprobaba cualquier acción**. Por eso Claude empujaba a GitHub, fusionaba PRs y escribía secretos **sin que nadie lo aprobara**, y lo contaba después.
  - `~/.claude/hooks/verificar-respuesta.sh` **existía desde el 2026-09-26 y no estaba declarado en ninguna configuración** → nunca se ejecutó. `areas/entorno.md` afirmaba por escrito que estaba «registrado como hook Stop»: **era falso**, y llevaba falso desde que se escribió.
- **Consecuencia directa:** la regla de `~/.claude/rules/comportamiento.md` que dice «esto lo hace cumplir un hook, no solo la memoria» **no la hacía cumplir nadie**. Explica por qué el usuario tuvo que pedir lo mismo varias veces el mismo día: no había nada mecánico detrás, solo la intención del modelo.
- **Lo que se monta, decidido por el usuario:** dos hooks nuevos, escritos y probados contra casos reales el mismo día.
  1. **`PermissionRequest` → `permisos-hacia-fuera.sh`**: las acciones hacia fuera (`git push`, `gh pr merge`, `gh secret set`, `gh release create/delete`, escrituras por `gh api`) devuelven **`ask`** — las aprueba el usuario antes, no se confiesan después. Lo demás sigue auto-aprobado: el objetivo es que no sea un peaje constante, no bloquear todo.
  2. **`Stop` → `documentacion-al-dia.sh`**: si el turno toca código (fichero que no es `.md`, o commit/push/merge) y **ninguna** documentación, bloquea el cierre. Excluye `/tmp/`. **No juzga si la documentación es buena, solo que exista** — eso lo cubre la regla de comportamiento.
  Y se **conecta** el que ya existía, `verificar-respuesta.sh`, que hasta hoy no corría.
- **Por qué con hooks y no con más reglas escritas:** se escribieron las reglas, se grabaron en el wiki y en `AGENTS.md`, y **se incumplieron igual**. Una regla que depende de que el modelo se acuerde no es una regla. La palanca es que no pueda hacer la acción sin que el usuario lo apruebe, o que no pueda cerrar el turno sin documentar.
- **Lo que sigue dependiendo de una persona, y se dice:** el hook de documentación comprueba que se tocó algún `.md`, **no que explique cómo funciona**. Un fichero de documentación vacío pasaría el hook. Eso no se automatiza aquí: el filtro real es la regla de comportamiento y la revisión del usuario.
- Detalle de los tres hooks y cómo desactivarlos: [[entorno]].

## 2026-09-28 — Lint del wiki: un enlace roto, el índice de `lego`, y dos notas nuevas (síntesis y contradicción)

- Recorrido completo de `proyectos/` (incluidas `astillero/`, `lego/` y `patrimonial/`), `areas/`, `recursos/` y `archivo/`; `fuentes/` fuera de alcance (material bruto, no se toca). Resultado: **frontmatter completo** en las 69 notas de contenido (`zona` siempre `tecnico` o `general`), **ninguna huérfana**, **ningún duplicado sin enlazar**, y los índices de `astillero`, `patrimonial`, `areas` (salvo lo de abajo), `recursos` y `archivo` correctos. El único `[[nombre-de-nota]]` que no resuelve es la plantilla dentro del comentario de los índices, no un enlace.
- **1 enlace roto corregido:** `[[investigacion-lego]]`, en `proyectos/lego/_index.md` y en `proyectos/lego/metodo-y-alcance.md`, apuntaba a una nota que **no existe**: el informe de Lego, declarado como entrega pero aún sin escribir. Quitado el wikilink en los dos sitios y marcado como pendiente de escribir — **no se inventa el informe**.
- **1 índice corregido:** `proyectos/_index.md` no listaba la subcarpeta `lego/` (creada el 2026-09-28). Añadida con su descripción.
- **Nota de síntesis creada:** github-pro-en-astillero — tema recurrente (qué habilita GitHub Pro en repos privados y qué contratos de Astillero dependen de esa puerta: K6, K10, etapa 9), presente en flujo-agentes-runbook, astillero-replicacion, la-fabrica, astillero-bitacora y flujo-agentes-arquitectura, sin nota propia hasta ahora. No es una decisión de compra. Enlazada en ambas direcciones.
- **Nota de contradicción creada:** contradiccion-precio-github-pro — la cifra «~4 $/mes» de [[decisiones]] y «4 $/mes» de flujo-agentes-runbook §2 no se sostiene en fuente primaria (comprobado el 2026-09-28: la página de precios de GitHub ya no lista el plan Pro). **Abierta**: la resuelve el usuario, no el lint. No se elige cifra.
- **Crecimiento — propuesto, no ejecutado (falta tu aprobación):** `proyectos/astillero/` tiene **39 notas**, por encima de las 30 de `AGENTS.md`. El reparto que quedó preparado (18 de investigación + 14 de diseño + hub y `la-fabrica` en la raíz) está **desfasado**: hay que recortarlo al alza al sumarse `verificacion-sin-oraculo-informe` y las notas de la fábrica. Se ejecuta solo cuando lo apruebes, y entonces se recuenta en vez de fiarse de estas cifras.

## 2026-09-28 — Investigación de compra: no se entrega nada citando un listado de resultados en vez de la ficha del producto

Investigando grifo de ducha para el alquiler de Carballo ([[duchas-alquiler-carballo]]), el usuario detectó el fallo: *«el producto de Amazon no existe, eso no es aceptable»*. La nota entregada citaba productos de Amazon con **enlaces de búsqueda** (`/s?k=…`) presentados como enlaces de producto, y con precios leídos de **tarjetas de resultados del buscador** sin abrir nunca la ficha. Al rehacerlo abriendo ficha por ficha aparecieron cuatro fallos más del mismo tipo:

- Dos productos que parecían buenas ofertas y no lo son, y **solo se ve al abrir**: la Roca Sensum a 55,24 € **no lleva grifo y está sin stock**; el kit Stella a 63,75 € **tarda 6-7 meses** y tampoco lleva grifo.
- Una oferta devuelta por el buscador (Roca Victoria Plus a **51,58 €**) que **no existe**: el listado real de vendedores de ese ASIN empieza en 68,50 €.
- Una **Roca Mitos Plus de Bauhaus a 53,99 €** que **es un conjunto completo** (ducha Natura, flexo metálico 1,50 m, soporte articulado, cartucho cerámico) y que se había despachado como "mezclador suelto" por no abrir su ficha. Bauhaus tampoco publica la tarifa de envío en página general, pero **sí en la ficha** (3,90 €): de ahí salió un "sin publicar" que era falso.

**Regla general que se deriva, aplicable a toda investigación de compra:** un listado de resultados no es una ficha, y un titular no es un producto. Antes de citar un producto se abre su ficha y se leen ahí el precio, el estado de stock, quién lo vende y **qué incluye exactamente** — porque "no lleva el grifo" y "sin stock" son cosas que la tarjeta de resultados no muestra. Los enlaces que se entreguen son de producto, nunca de búsqueda. Y una cifra que solo aparece en un resumen de buscador no se usa hasta reproducirla en la fuente.

## 2026-09-28 — Lego: el usuario aclara el objetivo

Corrección del usuario: el objetivo de Lego no es «un ingeniero con un agente, él al mando» (lo había añadido yo) ni filtrar por lo que ya tiene instalado. Es **saber qué existe hoy que funcione de verdad y le quite trabajo al desarrollar, aunque tenga que participar él**; cuanto más cubra, mejor, siempre que funcione. La fábrica perfecta la querría, pero no existe. Para qué: proyectos de cualquier tamaño, mucho más rápido. Detalle en [[objetivo]].

Consecuencia: las plataformas multiagente no quedan descartadas por categoría; entran si funcionan. Aclarar el objetivo no es un encargo de investigar.

## 2026-09-29 — El wiki no alberga proyectos: corrección de fondo, y Astillero fuera

El usuario, después de verme borrar media carpeta y discutir dónde iba el seguimiento de un proyecto, puso el dedo en la causa: **«el problema sois vosotros que mezcláis 2cerebro con proyectos independientes»**.

**Y tiene razón, con una agravante: la regla ya estaba escrita aquí desde el 2026-09-24** (entrada «el wiki no alberga proyectos», arriba) y se incumplió desde el mismo día. Los agentes —yo incluido— usamos el wiki como si fuera el repositorio de un proyecto: plan de trabajo, bitácora, diseño por fases, subidas de versión, números de PR, rutas locales. Eso acabó produciendo un hook de Stop que exigía «actualizar el plan o la bitácora» apuntando a un fichero del wiki. **El hook no era el problema: era la consecuencia.** Ofrecerme a «arreglarlo» fue tratar el síntoma.

**La frontera, que es lo que queda fijado:**
- **Al wiki:** conocimiento y método — investigaciones, síntesis, decisiones que gobiernan el propio wiki, reglas y correcciones del usuario, y **hubs cortos** que dicen qué es un proyecto y enlazan a su repo.
- **Al repo del proyecto:** su seguimiento — plan, bitácora, estado, versiones, PRs, commits, rutas, pendientes. `~/dev/<proyecto>` y su `docs/`.
- Si un hook o un CI pide actualizar el seguimiento de un proyecto **desde el wiki**, el que está mal es el mecanismo: ese seguimiento no debería estar aquí.

**Lo que se hizo, en este wiki:**

| Qué | Resultado |
|---|---|
| `proyectos/astillero/` | **Eliminada.** 40 notas: 32 borradas, 7 movidas a `proyectos/gas-city/` por no ser de un proyecto sino conocimiento (5 de `orquestacion-*`, `verificacion-sin-oraculo-informe`, `gas-city-frente-a-la-fabrica`) |
| `areas/` | Borradas 2 notas de proyecto. Conservadas y limpiadas 3 de principio general |
| Este registro | **30 entradas borradas** en total: su materia era el diario de trabajo de un repo, no una decisión del wiki. Quedan las que son reglas y correcciones |
| `proyectos/patrimonial/` | Bloque operativo quitado (ruta, versión, PRs, pendientes). Queda el hub con el enlace a su repo |
| `proyectos/gas-city/_index.md` | Quitada la sección «Estado» que había escrito yo — la misma falta, cometida en esta misma sesión |
| Enlaces | 88 enlaces a notas borradas, convertidos a texto. **Cero enlaces rotos** |

**Un caso que se conserva a propósito:** `proyectos/patrimonial/` mantiene el enlace a su repo porque eso es lo que hace un hub. Lo que ya no tiene es nada operativo.

**Lo que NO cambia:** las reglas de comportamiento, el método de investigación y el resto del esquema. Se corrigió dónde vive el seguimiento, no el criterio.

Enlazado desde: [[gas-city-instalacion-y-modelos]], [[gas-city-con-2cerebro]].
