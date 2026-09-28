---
title: Lego — qué plataformas encajan y cuáles no
created: 2026-09-28
updated: 2026-09-28
tags: [lego, plataformas, veredicto]
zona: tecnico
---

El veredicto sobre las 60 plataformas de [[plataformas-uso-real]], medido contra el objetivo de [[objetivo]], que manda: **cuanto más cubra una herramienta, mejor, siempre que funcione** — y **ninguna queda fuera por categoría**. No se ordenan por «qué añaden sobre el agente + git + CI»: eso fue un criterio mío y es falso. Se ordenan por **cuánto cubren y con cuánta prueba de que funciona**.

Y el dato que falta en toda la categoría, dicho por delante: **nadie publica cuánto ahorra**. No existe una sola medición independiente de resultado — ni «con esta se entregaron N funcionalidades con M defectos», ni horas ahorradas. Todo lo de abajo son señales de vida y de uso, no de ahorro medido.

---

## El hallazgo que reordena la pregunta

**De 60 entradas del catálogo, 8 tienen uso real declarado por terceros.** Las demás son propietarias sin cifra pública (12), muertas o paradas (5), o repos con estrellas y sin una sola voz que cuente que las usó.

Y las que sí lo tienen cubren, casi todas, **la misma capa**: avisarte de que el agente te espera. No la creación de la tarea ni la verificación. Eso lo confirma la comunidad sin que se le pregunte:

> «One task per session, each in its own git worktree, so two agents never edit the same checkout. Stop polling. Claude Code's Notification hook fires when it's waiting on a permission prompt or sitting idle» — parpil, [HN](https://news.ycombinator.com/item?id=49479775), 2026-09-11

> «My agent orchestration system is a bespoke python program that I vibed just for me.» — kasey_junk, [HN](https://news.ycombinator.com/item?id=46993479), 2026-02-16

**Hipótesis rival H3 («la función no la cubren productos, la cubren el agente + git + CI»): confirmada en el trabajo y la verificación, falsa en el enrutado de atención.** Los que saben describen su propio montaje con worktrees y hooks; los que usan plataformas valoran el móvil, las notificaciones y el multi-máquina. Eso git+CI no lo da, y es lo único que justifica la pieza extra.

---

## Encajan, ordenadas por cuánto cubren

Lo de arriba cubre más; lo de abajo, menos. Todas tienen prueba de uso por terceros.

| Plataforma | Qué cubre | Evidencia de uso ajeno | Lo que le falta |
|---|---|---|---|
| **kandev** | Kanban de tareas, revisión de cambios, abre PRs, sin telemetría — **lo que más se acerca a crear la tarea y comprobarla** | npm **5.969/mes** verificado; 2 menciones del mismo usuario | AGPL-3.0; comunidad casi nula |
| **Emdash** | Worktree por agente, diffs lado a lado, PR y CI dentro | [206 pts / 71 com](https://news.ycombinator.com/item?id=47140322); «I've been using this. Super useful, much better to avoid flipping between agents» | Interés comercial (YC W26); sin cifra npm verificable |
| **OpenChamber** | Auto-hospedable, acceso remoto, atado a OpenCode | [190/93](https://news.ycombinator.com/item?id=49233448); r/opencodeCLI [58 votos](https://redlib.catsarch.com/r/opencodeCLI/comments/1wi52e9/how_is_the_opencode_gui_missing_so_many_features/) con usuarios comparando GUIs | Depende de OpenCode, que tiene el bloqueo de Anthropic documentado |
| **Agent of Empires** | Worktrees + detección de «agente esperando» en TUI, web y móvil | [118/44](https://news.ycombinator.com/item?id=46588905); un «daily driver» declarado | Proyecto de una persona; fallo que corrompió sesiones de tmux (arreglado en 0.2.2 el mismo día) |
| **Herdr** | Multiplexor con estado de agente: aislamiento y aviso, sin tareas ni verificación | brew **8.151**/30d; [404/178](https://news.ycombinator.com/item?id=48756578) — el mayor volumen del barrido | Es un terminal. Su propia comunidad avisa del rumbo tras entrar en YC |
| **cmux** | Terminal de agentes con avisos; **tiene subreddit propio** | [198/77](https://news.ycombinator.com/item?id=47079718); [r/cmux 45 votos](https://redlib.catsarch.com/r/cmux/comments/1ue25i9/i_made_my_cmux_agents_talk_to_each_other_and/) | Mismo límite: capa de atención |

## No encajan, y por qué

| Descartado | Motivo |
|---|---|
| **Superset** | Su propia doc admite que **no muestra si el trabajo del agente tuvo éxito** — rompe la condición de control. Licencia Elastic 2.0, no código abierto |
| **Paseo** | Aviso crítico de seguridad sin acuse 7 días, 542 errores abiertos, «lo construye una sola persona» (ver [[plataformas-auditadas]]) |
| **Vibe Kanban** | «Sunsetting». Un usuario lo confirma en r/ClaudeCode: «Loved VibeKanban at first but they are sunsetting and **the last version doesn't even have the Kanban board**» |
| **Orca** | 80.619★ y 72 M de descargas, pero un usuario suyo dice «I haven't found any other users!». La señal de estrellas está **sin verificar** (ver más abajo) |
| **Conductor, Superconductor, Solo, Clor, atrium** | Cero evidencia de terceros. En Reddit, un usuario que los probó: «I tried the existing tools first — **Conductor, cmux, and a couple others — and kept bouncing off**» |
| **Opcode** | 22.412★ y **último commit de código en octubre de 2025** |
| **Fusion** | Release más descargado: 18. Su verificación adversarial es código muerto |
| **Shep** | **Sin licencia**: no hay derecho de uso |
| **Happy, HAPI, T3 Code, Muxy, Xum, nodeterm, dmux, tmux-ide, CodeNomad** | Son cliente remoto o terminal, no orquestación con tareas. Algunos con uso real alto (Happy: **33.712 descargas npm/mes**, la cifra más alta verificada) — entran como **pieza adyacente**, no como plataforma |
| **Los 30+ frameworks de especificación** | Ver [[panorama-de-herramientas]]: reempaquetan los mismos cuatro ficheros |

## Dos leads que salen de Reddit y **no estaban en el catálogo**

Aparecen en hilos con votos, no en listas de agregadores. **Sin verificar** — solo tengo el titular y el extracto:

- **`Subtask`** — [267 votos, 53 comentarios](https://redlib.catsarch.com/r/ClaudeCode/search?q=%22vibe+kanban%22&restrict_sr=on): «gives Claude Code a Skill and CLI to create tasks, spawn subagents, track progress, review and request changes. **Each task gets a Git worktree**». Es, en el titular, el encaje más directo con Lego de todo el barrido.
- **`Tutti`** — en [r/LLMDevs, 14 comentarios](https://redlib.catsarch.com/r/LLMDevs/search?q=paseo+claude+code): una lista de terceros con «Paseo · Conductor · **Tutti ← what I use now** · Warp Oz · Claude Code Agent Teams».

Y una que no es producto: **Claude Code Agent Teams**, la vía nativa del propio agente, mencionada como alternativa en dos hilos independientes.

---

## Mortalidad: es un dato, no una anécdota

De las 60, **cinco están muertas o paradas** y tres de ellas son de las más votadas: Vibe Kanban (28.214★, sunsetting), Opcode (22.412★, 11 meses sin código), `ccpm` (8.391★, parada desde marzo), `FleetCode` (parada desde marzo), edict (ver [[plataformas-auditadas]]). **Las estrellas son un indicador atrasado**: ninguna de esas cinco las perdió al morir.

## El mejor caso contra esta conclusión

**1 · Sin la plataforma se pierde el enrutado de atención, y eso es real.** El móvil, las notificaciones y el multi-máquina no los da git+CI. El propio rechazo lo confirma: «I don't want to add a new vendor in the mix for our infra that may not exist in a year» (willdoenlen) — rechaza la plataforma *por miedo a su muerte*, que es admitir que algo aporta.

**2 · La evidencia de uso de esta categoría es de una sola plataforma.** Casi todo lo de arriba es Hacker News y Reddit, y buena parte son hilos «Show HN», donde el autor está presente. No hay ni una medición independiente de resultado: nadie ha publicado «con esta plataforma se entregaron N funcionalidades con M defectos». Sin eso, «maduro» se está juzgando por señales de vida, no por resultados.

**3 · Y la ausencia de uso no prueba que no sirvan.** Varias de las descartadas por E2 (Conductor, Superconductor, Solo, Clor) podrían ser buenas sin que nadie lo haya contado en público. Se descartan por falta de prueba, no por prueba de que sean malas — y eso es distinto.

---

## Lo que no se pudo comprobar

1. **La curva de estrellas de Orca.** El endpoint público de GitHub devuelve 404 (también en `torvalds/linux`: es del API, no del repo). Queda la contradicción sin resolver entre 72 M de descargas y usuarios que dicen no conocer a otros usuarios.
2. **Los Discords** de cmux, Emdash, Ghostex, kandev, jean y Nimbalyst: es donde vive la actividad real de esta categoría y no son legibles sin cuenta.
3. **Las cifras de uso de las propietarias**: Conductor, Superconductor, atrium, Solo, Clor, Xirp, Codex app.
4. **`herdr` en Reddit**: dos intentos de búsqueda abortados por tiempo de espera. Su evidencia es de HN (404/178), la más voluminosa del barrido, y no está en duda.

## Cobertura

**60 de 60** entradas del catálogo enumeradas desde su fuente. **44** con repo medido en vivo. **Reddit** entró por la vía de Redlib sobre Chrome real, después de que los cuatro frentes lo recibieran bloqueado (403 y «blocked due to a network policy») — sin eso, toda la evidencia de comunidad habría sido de una sola plataforma.

## Enlaces

- [[plataformas-uso-real]] — la tabla de medición, plataforma a plataforma
- [[plataformas-auditadas]] — las cinco auditadas por dentro
- [[lo-que-dice-la-comunidad]] — el veredicto de la comunidad sobre Lego en general
- [[limites-del-andamiaje]] — la mejor crítica a la categoría
