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

- Proyecto nuevo: [[astillero]]. Genérico: debe servir para cualquier desarrollo; stack y plataforma se deciden en cada proyecto.
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

## 2026-09-25 — Flujo de desarrollo con agentes: diseño propuesto (pendiente de revisión del usuario)

- Investigación en fases A–C con revisión del orquestador entre fases. Se corrigieron dos citas no literales, un inventario incompleto y un benchmark con fecha imposible ([[flujo-agentes-informe]] §2).
- Propuesta: GitHub Issues + sub-issues + dependencias como estado; primitivas de GitHub Actions (grupos de concurrencia, `timeout-minutes`) para reserva y caducidad; gh-aw como ejecutor con DeepSeek a través del endpoint compatible con Anthropic; `claude-code-action` con token de la suscripción Pro para que revise Opus; CI determinista y `CODEOWNERS`. Se descarta Beads por R13 (fallos abiertos de corrupción y demonios).
- Todavía no es una decisión cerrada: la valida el usuario. Detalle en [[flujo-agentes-arquitectura]].

## 2026-09-25 — Nombre del repo plataforma: Astillero. Mecanismo de replicación verificado

- El repo que contendrá los workflows y la configuración reutilizables del sistema se llamará **Astillero** (decisión del usuario, tras descartar varias tandas de nombres propuestos).
- Mecanismo de replicación a cada proyecto, verificado en fuente primaria: reusable workflows de GitHub Actions (`workflow_call`, confirmado que funcionan entre repos privados de una cuenta personal) para el revisor y el reconciliador; imports remotos de gh-aw (`imports: owner/repo/path@ref`, confirmado que existen) para el ejecutor; **copier** para lo estático (`CODEOWNERS`, `AGENTS.md`, labels, plantilla de contrato), descartados cruft y `repo-file-sync-action` por falta de mantenimiento. Detalle completo en [[astillero-replicacion]].
- **Corrección del usuario (2026-09-25):** todo repo de este ecosistema (Astillero y cada proyecto, empezando por Patrimonial) se crea **privado**, siempre, sin plantear la opción pública. Consecuencia: hace falta GitHub Pro (~4 $/mes, cubre todos los repos privados de la cuenta, no es coste por repo) para tener rulesets y merge queue en cualquiera de ellos.
- Sin decidir todavía: la creación real del repo Astillero en GitHub (hoy solo existe el diseño).

## 2026-09-25 — Renombrado: sistema-desarrollo-con-agentes → Astillero

- Otra sesión de esta cuenta decidió el nombre **Astillero** para el repo plataforma y ya lo había creado en GitHub (`blogNetting/astillero`, privado) e investigado su mecanismo de replicación ([[astillero-replicacion]]: reusable workflows para revisor/reconciliador, imports remotos de gh-aw para el ejecutor, copier para lo estático), antes de que esta sesión lo supiera.
- Esta sesión renombró la carpeta y la nota hub del wiki (`sistema-desarrollo-con-agentes` → `astillero`) para que coincidan, sin perder ningún enlace ni contenido. Verificado que no hubo colisión de ediciones concurrentes (el único solape, en `flujo-agentes-arquitectura.md`, era una edición propia de esta sesión capturada por el cron de las 20:00, no un cambio ajeno).
- A partir de aquí, «Astillero» es el nombre único del proyecto en el wiki y en GitHub.

## 2026-09-25 — Capa de producto: diseño cerrado, Astillero completo

- Petición del usuario: «quiero crear software como si fuera una empresa, hacer de product owner... alguna pregunta porque luego no voy a aceptar cagadas». Aclarado por el propio usuario: enfoque corporativo y serio, no departamentos no técnicos ni ceremonia de equipo; «mientras funcione me da igual».
- Investigación dirigida activamente a encontrar el patrón contrario y no encontrarlo: **cero operadores solos usando RICE/ICE/WSJF** (hilo de HN de 130 puntos, ~30 operadores solos, sin ninguna mención); **la mesa de apuestas de Shape Up exige pluralidad de personas por diseño del propio libro** («one designer and one programmer» como mínimo) — ninguna de las dos prácticas se adopta.
- Decisiones: sin scoring de priorización, backlog simple reordenado por juicio directo; captura de idea vía entrevista (patrón oficial de Anthropic) con Appetite y No-gos de Shape Up como únicos campos que sí sobreviven; panel en GitHub Projects v2 vía `update-project`/`create-project-status-update` extendiendo el reconciliador — pieza que `github/gh-aw` documenta pero no usa para sí mismo (verificado dos veces: 0 proyectos en la organización, badges en su lugar); bugs con triage de solo-reproducción (patrón real de Metabase, Repro-Bot, ~10% falsos positivos citado por su autor) antes de entrar al mismo contrato de tarea que una funcionalidad, sin atajos de hotfix; versionado por checkpoint (`release-please` o notas nativas de GitHub) en vez de publicación automática; registro de decisiones de producto sin precedente real encontrado (inferencia explícita, ni siquiera Shape Up documenta sus propios descartes). Detalle completo en [[capa-producto]].
- Con esto, Astillero queda documentado como sistema completo: motor de ingeniería ([[flujo-agentes-arquitectura]], probado en vivo), replicación ([[astillero-replicacion]], probada en vivo) y capa de producto ([[capa-producto]], diseñada con evidencia, sin implementar en código todavía). Nota hub `astillero.md` actualizada para reflejar las tres piezas como una sola imagen coherente.

## 2026-09-25 — Auditoría real: el ciclo no llegaba a producción. Despliegue y DevOps mínimo, cerrados

- Pregunta directa del usuario: «¿todo lo que tienes cubre todo el proceso de idea a implementación?». Respuesta obtenida por auditoría real (lectura completa de las 15 notas del proyecto, no reafirmación): **no** — el pipeline construido hasta ese momento terminaba en el merge a `main` (K11); cero notas cubrían despliegue a producción, observabilidad en producción o gestión de incidentes. Confirmado también que el sistema de cobertura de tests, al contrario de lo que sugería una línea desactualizada de `astillero.md`, **ya estaba resuelto** desde el 2026-09-24 en `desarrollo-agentes-f4-devsecops.md` §3.3 — solo mal enlazado desde el hub.
- Cerrados hoy, con la misma exigencia de evidencia que el resto del proyecto:
  - **Despliegue a producción** ([[flujo-agentes-arquitectura]] §15): CD automático en cada merge a `main`, sin checkpoint manual aparte de la versión (`release-please` sigue siendo un eje distinto, no bloquea el CD) — evidencia de 40+ operadores solos en dos hilos de HN. Dónde corre la app: PaaS gestionado, PaaS autoalojado sobre VPS (Dokku/CapRover), o VPS+Compose puro, sin staging permanente por defecto. Rollback: revertir el commit, extiende K10. Migraciones: expandir y contraer el schema, siempre dos tareas distintas del backlog (regla dura, ligada al hecho de que el ejecutor trabaja tarea por tarea).
  - **DevOps mínimo en producción** ([[devops-minimo]], nota nueva): anclado en un caso real auditable, no en opinión — Healthchecks.io, SaaS operado en solitario con su stack de producción publicado y verificado (repo, historial de commits). Hallazgo contraintuitivo con fuente: **no hace falta on-call formal con un solo operador** — diseñar para resiliencia (auto-restart, alerta que despierte) basta; el propio hilo de referencia de 149 puntos en HN lo confirma en consenso mayoritario. Backup diario cifrado + replicación con failover manual deliberado (no automático — "problema demasiado difícil" según la propia fuente). Secretos con `sops`, umbral real: por debajo de ~10 repos basta, por encima hace falta Vault — Astillero está muy por debajo. Parcheo de dependencias: el caso real auditado ni siquiera usa Dependabot, se apoya en CI real en cada commit — válido también aquí.
  - **Correcciones al sistema de cobertura ya cerrado**: Vitest usa su proveedor `v8` nativo, no `c8` como paquete aparte (desactualizado); Codecov en vez de Coveralls, porque Coveralls no tiene plan gratis para repos privados y todo repo de Astillero es privado por decisión ya cerrada; sin umbral global fijo de cobertura por ser gameable (Fowler, fuente primaria del propio término), `patch` (líneas nuevas) cerca del 100% como gate real, apoyado en mutation testing del diff ya decidido.
- Bloqueo real encontrado y no escondido: la cuota de `WebSearch` de la sesión se agotó (200/200) durante estos frentes de investigación — resuelto con la API de HN Algolia y `gh api`/`gh search` sobre repos reales en vez de descubrimiento por buscador, sin que afectara a ninguna conclusión (todo lo citado viene de fuente primaria verificada, no de la búsqueda en sí).
- Astillero queda con 5 piezas documentadas: motor de ingeniería, replicación, capa de producto, despliegue y DevOps mínimo — las dos últimas diseñadas con evidencia real pero **sin ejecutar en vivo todavía**, señalado explícito en `astillero.md`, no escondido bajo una reafirmación de "todo completo".
