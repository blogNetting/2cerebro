---
title: Registro de decisiones
created: 2026-09-08
updated: 2026-09-25
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

## 2026-09-24 — Sistema de desarrollo con agentes: decisiones de partida

- Proyecto nuevo: [[sistema-desarrollo-con-agentes]]. Genérico: debe servir para cualquier desarrollo; stack y plataforma se deciden en cada proyecto.
- El código de este sistema y de cada app vive en repos propios, nunca en `2cerebro` (es público y se publica cada hora). En el wiki, cada uno tiene una nota hub que enlaza al repo.
- Regla dura del usuario: no hay código sin tests, sean unitarios o de integración. Los umbrales de cobertura y complejidad se fijarán más adelante.
- Aceptado usar DeepSeek por su API oficial; que los datos estén en China no es un problema. El coste se controla con saldo prepagado y midiendo el consumo.
- La arquitectura Opus/DeepSeek se decide por investigación, pruebas y decisión conjunta, no por opinión.

## 2026-09-24 — Herramientas de investigación técnica

- Instalados `gh` y `yt-dlp` en `~/.local/bin`, binarios oficiales con el checksum verificado. No hay sudo; se descartó `apt`. Se descartaron también los MCP de GitHub y de YouTube: la CLI cubre lo mismo sin gastar contexto en definiciones de tools en cada turno.
- `/investigar-web` tiene un paso 10 nuevo con fuentes directas: GitHub, la API de Algolia de Hacker News y transcripciones de YouTube. Reddit y X van directos a CDP. El deep research de otras IAs solo sirve para descubrir pistas. Ver [[entorno]].

## 2026-09-25 — Sistema de desarrollo con agentes: correcciones y decisiones del usuario

- Subcarpeta `proyectos/sistema-desarrollo-con-agentes/` con su `_index.md`, aprobada por el usuario («tiene que estar en una carpeta propia»). Contiene las 11 notas del proyecto. Los wikilinks no cambian porque los nombres se mantienen.
- El ejecutor es abstracto: la arquitectura no depende de quién implemente. DeepSeek se usa por coste, no por calidad. Corrección: en [[desarrollo-agentes-investigacion]] se trató DeepSeek como si hubiera que justificarlo por calidad.
- Decisiones: GitHub como forja; framework spec-driven elegido en cada proyecto; no hay piloto hasta tener el sistema completo.
- Correcciones de forma: explicar cada tecnicismo al usarlo (DORA se usó sin explicar); ante una pregunta amplia, traer una propuesta concreta en vez de devolver la pregunta; no desviarse del problema central (diseño en tareas → cola de tareas tipo Beads → ejecutores → revisión).

## 2026-09-25 — Regla permanente: lo que menciona el usuario no condiciona nada

- Petición expresa del usuario («escríbelo en sangre»), tras varias repeticiones del mismo fallo: tomar lo que él nombra (DeepSeek, issues, gitflow, Beads…) como premisa o como centro del trabajo.
- Se añade la sección «Lo que menciona el usuario no condiciona nada» a `AGENTS.md`, junto con la regla de explicar cada sigla al usarla (ajuste de «no expliques fundamentos»).
