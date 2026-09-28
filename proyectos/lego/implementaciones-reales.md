---
title: Lego — implementaciones reales, y sus fracasos
created: 2026-09-28
updated: 2026-09-28
tags: [lego, repos, implementacion, fracasos]
zona: tecnico
---

Lo que de verdad se puede copiar: montajes completos con su contenido real, **plantillas de tarea sacadas de proyectos en producción**, y los fracasos documentados por quien los vivió. Se auditaron ~35 repositorios —árbol de ficheros y contenido de 14— y 12 hilos de Hacker News con sus comentarios.

## Lo más copiable: montajes enteros, no frameworks

| Proyecto | Qué trae exactamente | Licencia | Estado |
|---|---|---|---|
| **[fstandhartinger/ralph-wiggum](https://github.com/fstandhartinger/ralph-wiggum)** ⭐ | `ralph-loop-codex.sh` + `spec_queue.sh` · **`circuit_breaker.sh`** · **`nr_of_tries.sh`** · **`response_analyzer.sh`** · `notifications.sh` (Telegram) · `RALPH_PROMPT.md` | MIT | 300★, último empujón may-2026 |
| **[khgs2411/flow](https://github.com/khgs2411/flow)** | **Un solo script de bash de ~63 KB, sin dependencias**, 18 comandos, **todo el estado en `PLAN.md`** | MIT | El más simple y el más «copiar y usar» |
| **[blader/taskmaster](https://github.com/blader/taskmaster)** | Un gancho de parada que **impide que el agente termine antes de acabar** | MIT | 524★ |
| **[mikeyobrien/ralph-orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)** | Reimplementación del bucle en Rust | MIT | 3.160★, activo |
| **[Th0rgal/open-ralph-wiggum](https://github.com/Th0rgal/open-ralph-wiggum)** | `ralph "prompt"` + chequeo de estado | MIT | 1.893★ |
| **[covibes/zeroshot](https://github.com/covibes/zeroshot)** | Orquestación **ejecutor–verificador** en Rust | MIT | 1.915★, activo |
| **[Dicklesworthstone/claude_code_agent_farm](https://github.com/Dicklesworthstone/claude_code_agent_farm)** | 20+ agentes en paralelo, bloqueos, monitorización por tmux | NOASSERTION | 919★ |
| **[mutable-state-inc/lean-collab](https://github.com/mutable-state-inc/lean-collab)** | Memoria compartida, **bloqueos de reclamación con caducidad**, retroceso | MIT | 73★ — el diseño mejor explicado |
| **[automazeio/ccpm](https://github.com/automazeio/ccpm)** | Skill + **16 scripts** (`next.sh`, `epic-list.sh`, `standup.sh`, `blocked.sh`, `validate.sh`) | MIT | 8.391★ — **la cola son Issues de GitHub + worktrees**, no markdown |

**El más copiable es el primero**: `fstandhartinger/ralph-wiggum` trae **cortacircuitos, contador de reintentos, analizador de respuesta y notificación**. Es un montaje, no un framework — que es exactamente lo que buscabas.

## Las plantillas de tarea, en proyectos reales

Esto es lo que **no está en la documentación de ninguna herramienta** y es lo que de verdad se puede copiar:

### [majiayu000/quotabar/specs/GH55/tasks.md](https://github.com/majiayu000/quotabar/blob/main/specs/GH55/tasks.md) — el mejor ejemplo encontrado

Un registro por incidencia, con:

- **«Delivery Contract»**: rama base, política de commits, alcance, compatibilidad.
- Tareas con **`Owner:`** (el agente que la hace), **`Dependencies`**, **`Covers:`** (qué criterios cubre), **`Done when:`** y **`Verify:`**.
- Una sección de **traspaso**.
- **Invariantes y cobertura declaradas.**
- **Límites explícitos**: prohibido forzar el push, filtrar errores en crudo, degradar tests, o salirse del alcance declarado.

**Fíjate en que tiene `Done when` y `Verify` separados**, que es más preciso que los cinco campos que yo derivé: uno dice cuándo está terminado y el otro **cómo se comprueba**.

### Otros reales, y lo que enseñan

- **[WeihanLi/dotnet-exec/.specify/templates/](https://github.com/WeihanLi/dotnet-exec/blob/main/.specify/templates/tasks-template.md)** y **[nutanix-cloud-native/…](https://github.com/nutanix-cloud-native/cluster-api-runtime-extensions-nutanix/tree/main/.specify/templates)** — la plantilla de Spec Kit **modificada por empresas reales**. dotnet-exec le añade su propia política: *«los cambios de comportamiento en el análisis, la compilación o las interfaces públicas **requieren tareas de test**»*. **Más útil que la plantilla original, porque muestra dónde cada equipo la rompe.**
- **[microsoft/kalypso-scheduler/specs/001-bootstrapping-script/](https://github.com/microsoft/kalypso-scheduler/tree/main/specs/001-bootstrapping-script)** — Spec Kit en un repositorio de Microsoft, **pero sólo tiene `spec.md` y `checklists/`: no hay `tasks.md`**. El proceso **se paró en la fase de especificación**. Dato que importa: **ni las organizaciones grandes lo recorren entero**.
- **[johndpope/VASA-1-hack/.taskmaster/tasks/tasks.json](https://github.com/johndpope/VASA-1-hack/blob/nemo/.taskmaster/tasks/tasks.json)** — el esquema JSON que más gente tiene versionado (**874 repos**): `{id, title, description, details, testStrategy, status, dependencies, priority, subtasks}`.
- **[templates/tasks-template.md de Spec Kit](https://github.com/github/spec-kit/blob/main/templates/tasks-template.md)** — el original: `[ID] [P?] [Story] Descripción`, fases, y `[P]` para lo paralelizable.

## Las cifras del creador del patrón, y sus propios fracasos

**Geoffrey Huntley** ([ghuntley.com/ralph](https://ghuntley.com/ralph/)) — el bucle entero es `while :; do cat PROMPT.md | claude-code ; done`. Sus números, textuales:

- Un contrato de **50.000 $ entregado por 297 $** *(su cifra, no auditada)*.
- **«Sólo tienes unos 170k de ventana de contexto para trabajar.»**
- **«Llegarás al 90% con él.»** — una expectativa, no una medición.
- **Hasta 500 subagentes** para planificar; **sólo 1** para construir y probar.

**Y sus fracasos, en el mismo texto —esto es lo que da valor a la fuente:**

> «**Te despertarás con una base de código rota que no compila** de vez en cuando.»
>
> «Claude tiene el sesgo inherente de hacer **implementaciones mínimas y de relleno**.»
>
> «El repositorio se llena de basura, ficheros temporales y binarios.»
>
> «Este no-determinismo es **el talón de Aquiles de Ralph**.»
>
> «**No usaría Ralph en una base de código existente ni de broma.**»
>
> «Quien venda que una herramienta puede hacer el 100% del trabajo sin un ingeniero **está vendiendo humo.**»

**Y el coste operativo, de [The Register](https://www.theregister.com/2026/01/27/ralph_wiggum_claude_loops/)**: **10 $ por hora** de cómputo. Con un contraejemplo en el mismo artículo: *«los resultados no fueron buenos porque no tenía una especificación del producto»*.

## Los fracasos documentados, con nombre

**1. «Ralph Wiggum no funciona»** — [pedronauck, en Hacker News](https://news.ycombinator.com/item?id=46672413). Los modos de fallo, textuales: *«te encuentras con modos de fallo predecibles: **sin puertas de verificación, recuperación débil, contaminación de contexto entre pasos, y bucles descontrolados y caros**»*. Y su diagnóstico, que es el de esta investigación entera: *«el arreglo no es “más contexto”, es **aislar los pasos y pasar estado explícito entre ellos, con verificación y guardarraíles**»*.

**2. El fracaso de Spec Kit *y* Taskmaster**, contado por quien los usó meses ([hilo en r/ClaudeAI](https://redlib.catsarch.com/r/ClaudeAI/comments/1nvwnou/i_created_flow_a_free_framework_for_keeping_ai_in/)):

> «Usé Spec Kit de GitHub y Taskmaster durante meses. Grandes herramientas, **pero un problema enorme me mordía una y otra vez: la IA se descontrola sin pausas claras.**»
>
> «Planificas 5 horas → la IA salta al código → **la IA pierde el contexto o abre una sesión nueva** → la IA empieza a inventar aunque no lo hayas acordado → 40 minutos después tienes **40 ficheros nuevos y no sabes qué hacer con ellos**.»

**3. El propio rastreador de Spec Kit**, que es la mejor crítica porque la escriben sus usuarios:

- [#3507 — «ejecutar cada fase en un subagente **para evitar la podredumbre del contexto**»](https://github.com/github/spec-kit/issues/3507)
- [#4269 — «`/speckit-converge` **no es idempotente**: las reejecuciones añaden tareas de remediación duplicadas»](https://github.com/github/spec-kit/issues/4269)
- [#4164](https://github.com/github/spec-kit/issues/4164), [#4582](https://github.com/github/spec-kit/issues/4582), [#865](https://github.com/github/spec-kit/issues/865)

**La podredumbre de contexto y la no-idempotencia son fallos de diseño, no del usuario.**

**4. La crítica estructural**, en Hacker News:

> «El agente tiene instrucciones de “no escribas código, escribe tests”, y al hacerlo **define una API. Eso hará que la IA alucine la API**.» Y el resultado: código con **alta cobertura** que esconde un *«nido de ratas»*.
>
> «**El SDD es exactamente el método en cascada.** El problema del cascada nunca fue el tiempo: es que **la especificación te encierra en un nicho pequeño**.»

**5. Y una corrección al informe:** «Anthropic Legal **obligó a renombrar** en su repositorio a “Ralph Loop”», y el plugin oficial **«no es el concepto original, porque no persiste entre sesiones»** ([HN 46750937](https://news.ycombinator.com/item?id=46750937)).

## El dato que nadie publica

**Ningún caso de los encontrados publica la tasa de éxito** — cuántas tareas salieron bien de cuántas. Lo más cercano es el **«90% esperado»** de Huntley para proyectos nuevos, que es **una expectativa, no una medición**.

Y hay una limitación que repiten quienes lo construyeron: **esto no vale para código existente**. Huntley, literal: *«no usaría Ralph en una base de código existente ni de broma»*.

## Corrección al informe

El sucesor del GSD archivado: escribí `gsd-build/gsd-2` (7.782★), y la búsqueda encuentra también **[open-gsd/gsd-core](https://github.com/open-gsd/gsd-core)** con **9.913★**. **Los dos existen y no he podido determinar cuál es el sucesor oficial.** El original, `gsd-build/get-shit-done`, está archivado con 64.442★ y su README dice: *«este repositorio ya no es el hogar activo del desarrollo de GSD»*. **Queda sin resolver cuál es el bueno.**

## Comunidades activas

[r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/) (la más activa) · Discord de [Taskmaster](https://discord.gg/taskmasterai), [BMAD](https://discord.gg/gk8jAdXWmj), [GSD](https://discord.gg/mYgfVNfA2r), [ralph-orchestrator](https://discord.gg/XWUyeUNffh) · y un hilo comparativo de r/ClaudeAI —«Comparé 11 sistemas de flujo de trabajo de Claude Code en una tabla»— que **no se pudo leer**: otra sesión me cambió la pestaña encima.

## Dónde se ha buscado, y qué no se pudo

**Aportó:** `gh api search/code` (búsqueda en repos reales — **esto es lo que dio las plantillas**), `git/trees?recursive=1` para saber qué trae cada montaje, `raw.githubusercontent.com` para leer los ficheros, la API de HN para los hilos y sus comentarios, y el Chrome real por CDP contra Redlib.

**No aportó nada:** las búsquedas genéricas por estrellas devuelven listas y pasarelas de IA sin relación; las plantillas de arranque de Claude Code encontradas tienen **0 o 1 estrella** y ninguna evidencia de uso; y siete variantes pequeñas más (entre 1 y 16 estrellas) **descartadas en bloque**.

**Bloqueado:** Reddit por `curl` (302, 403, 429 y pruebas de trabajo en cinco instancias distintas) — hubo que ir por el navegador. Y **varias extracciones se perdieron porque otra sesión usaba el mismo Chrome**.

**Sin comprobar:** la tasa de éxito real de cualquiera de estos montajes; si los 12.471 forks de Spec Kit contienen cambios propios o son clones vacíos; el interior de BMAD; y cuál de los dos GSD es el sucesor.

## Enlaces

- [[videos-y-comunidad]] — vídeos y comunidades
- [[cursos]] — la formación
- [[montaje-documentado]] — el montaje, ahora con lo copiable
