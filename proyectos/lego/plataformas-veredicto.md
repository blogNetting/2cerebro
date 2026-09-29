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

## El filtro contra el objetivo

Tres condiciones, sacadas de [[objetivo]]: **O1 funciona** (terceros que lo cuentan), **O2 le quita trabajo** (hace parte del trabajo de entregar software, no solo ordena la pantalla), **O3 cubre** (cuántas piezas del proceso). Lo que dice el README va marcado como **capacidad declarada por el fabricante**, no como prueba de que funcione — eso solo lo dan los terceros.

| Candidata | O1 · Funciona | O2 · Le quita trabajo | O3 · Qué cubre | Veredicto |
|---|---|---|---|---|
| **OpenChamber** | **Sí** — [58 votos comparando GUIs](https://redlib.catsarch.com/r/opencodeCLI/comments/1wi52e9/how_is_the_opencode_gui_missing_so_many_features/), [HN 190/93](https://news.ycombinator.com/item?id=49233448) | **Sí**, y es la única que cierra el círculo | Ejecución · aislamiento · aprobaciones · **límites del proveedor, uso de tokens y coste** · CI y PR · **«send failed checks or review comments back to the agent, then merge»** — el fallo vuelve como trabajo nuevo, que es un mecanismo de la lista | **La que más merece análisis a fondo.** Capacidad declarada en su README; su límite es depender de OpenCode |
| **Emdash** | **Sí** — [HN 206/71](https://news.ycombinator.com/item?id=47140322) | Sí | Ejecución · aislamiento · «review diffs, create pull requests, **inspect CI checks, and merge** from one place» | Cumple. No declara gasto ni verificación propia |
| **Orca** | **Sí pero con la contradicción abierta** — [r/ClaudeCode 162 votos](https://redlib.catsarch.com/r/ClaudeCode/search?q=orca&restrict_sr=on) frente a «no conozco a otros usuarios» | Sí | «Fan one prompt across five agents, each in its own isolated worktree — compare the results and merge the winner» · aviso al móvil | Cumple con reserva: la señal de uso no está resuelta |
| **kandev** | **Débil** — npm **5.969/mes** sí, comunidad casi nula | Sí | Kanban de tareas · revisión · PRs · sub-tareas · automatizaciones. **Gasto y presupuestos están en su hoja de ruta, no en el producto** | Cumple con reserva: mucha capacidad, ninguna voz independiente |
| **Agent of Empires** | **Sí** — [HN 118/44](https://news.ycombinator.com/item?id=46588905) + un «daily driver» | Parcial | Detección de estado y avisos · worktrees · sandboxing (Docker, Podman, Apple Containers) · sesiones que sobreviven a un cierre de SSH | Cumple en su parte: aislamiento y atención, no verificación ni gasto |
| **Herdr · cmux** | **Sí**, con fallos concretos reportados | **Solo la parte de atención** | Aislamiento y «el agente te espera». No crean tareas ni comprueban nada | Cumplen solo la pieza de aviso |
| **Happy · HAPI · T3 Code · Muxy** | Sí, muy usadas | **No**: mueven la interfaz al móvil, no hacen trabajo de entrega | — | Fuera como plataformas; entran como accesorio |
| **Superset** | Sí | **No**: su doc admite que no muestra si el trabajo tuvo éxito, así que revisar sigue siendo tuyo entero | — | No cumple |
| **Paseo** | Sí, el más elogiado | Sí | — | **No cumple**: un aviso crítico de seguridad sin acuse 7 días no es «fiable» |
| **Vibe Kanban** | — | — | — | No cumple: se cae |
| **Subtask · Tutti** | **Sin datos** | ? | ? | **Los primeros a mirar**: por titular son el encaje más directo |
| Conductor · Superconductor · Solo · Clor · atrium · Xum · nodeterm · CodeNomad · tmux-ide · dmux · Sidecar · Luvus · Acepe · Sidequest | **Sin evidencia de terceros** | — | — | No se pueden juzgar: falta la prueba, no sobra |

**La columna que nadie llena es la del ahorro.** Ninguna de estas publica cuánto trabajo quita: ni funcionalidades entregadas, ni horas, ni defectos. Lo que hay son señales de uso.

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

## Lista ordenada, de más a menos relevancia

**76 entradas: las 60 del catálogo `agentmgmt.dev` —comprobadas una a una contra su `tools.yml`— más 16 que no están en él** (6 de fuera: `Claude Code agent teams`, `oh-my-claudecode`, `DeepCode`, `edict`, `Subtask` y `Tutti`; y 10 repos encontrados por tema al final).

Cada línea lleva la marca, el criterio y la prueba:

- **✓** funciona según terceros **y** hace parte del trabajo
- **~** funciona, pero solo cubre la capa de aviso
- **✗** auditada con problema grave, cerrada o sin licencia
- **?** sin ninguna prueba de terceros
- **⚠** lead sin verificar

```
 1  OpenChamber          ✓  coste y fallos devueltos al agente, PR y merge · Reddit 58 votos, HN 190/93
 2  Emdash               ✓  worktree, diffs, PR, CI y merge · HN 206/71
 3  Orca                 ✓  5 agentes en paralelo, comparar y fusionar · r/ClaudeCode 162 votos — uso sin resolver
 4  kandev               ✓  kanban, revisión, PR, sub-tareas · npm 5.969/mes · gasto solo en hoja de ruta
 5  Agent of Empires     ✓  aislamiento, sandbox, aviso · HN 118/44 + un «daily driver»
 6  Claude Code agent teams ✓  nativo, cero piezas nuevas: tareas, cron, aprobación de plan · 20 subagentes concurrentes · sin worktree para los compañeros · evidencia de terceros fina
 7  Paseo                ✗  aviso crítico de seguridad sin acuse 7 días · auditada
 8  Subtask              ⚠  SIN VERIFICAR · r/ClaudeCode 267 votos, solo el titular
 9  Tutti                ⚠  SIN VERIFICAR · r/LLMDevs, solo una mención
10  Herdr                ~  solo aviso · brew 8.151/30d, HN 404/178
11  cmux                 ~  solo aviso · HN 198/77, r/cmux 45 votos
12  Nimbalyst            ~  aviso y edición visual · release 135.395 · hilos del propio autor
13  Happy                ~  cliente móvil · npm 33.712/mes
14  tmux-ide             ~  reparto de paneles · npm 4.534/mes, HN 88/38
15  CodeNomad            ~  GUI de un agente · 3 usuarios independientes en HN
16  dmux                 ~  worktree por panel · npm 915/mes · HN 9/0
17  HAPI                 ~  control remoto · brew 10/30d
18  T3 Code              ~  superficie de control · HN 6/0
19  Muxy                 ~  terminal macOS · HN 4/0
20  Sidecar              ~  shell con paneles · brew 39/30d
21  Xum                  ?  sin uso ajeno
22  nodeterm             ?  sin uso ajeno
23  Ghostex              ?  release 281-373, ningún hilo
24  Luvus                ?  sin uso ajeno
25  Acepe                ?  sin uso ajeno
26  Sidequest            ?  sin uso ajeno
27  oh-my-claudecode     ✗  peaje en estrellas, bucle de tokens, un solo autor · auditada
28  Superset             ✗  no verifica el trabajo; licencia Elastic 2.0 · auditada
29  Vibe Kanban          ✗  sunsetting
30  Opcode               ✗  22.412★, último commit de código en oct-2025
31  Supacode             ?  127.094 descargas, ningún hilo
32  Amux                 ?  0 descargas del binario empaquetado
33  Kooky                ?  sin uso ajeno
34  Shep                 ✗  sin licencia: no hay derecho de uso
35  Rabbitty             ✗  fuente privada, 7 descargas
36  OpenScout            ?  sin cifra ni hilos
37  Synara               ?  366.967 descargas, ningún hilo
38  jean                 ?  6.955 descargas, sin voz propia
39  Agent Orchestrator   ?  sin cifra atribuible · Show HN 15/0
40  Fusion               ?  release más descargado: 18 · su verificación es código muerto
41  Conductor            ?  propietario, cero terceros
42  Superconductor       ?  solo datos del vendor
43  Solo                 ?  propietario
44  Clor                 ?  HN 11/5
45  atrium               ?  HN 3/0
46  Podium               ?  26★, solo el autor
47  Zeron                ?  5.762 descargas, ningún hilo
48  AgentsDock           —  no es plataforma: IDE · HN 81/32
49  stagewise            —  no es plataforma: agente de front-end · HN 46/50
50  den                  —  repo 404, no auditable
51  Xirp                 —  propietario, una sola sesión
52  Polyscope            —  opaco
53  Spruce               ?  propietario, sin métricas
54  maestri              ?  dos anuncios en HN, 0 comentarios
55  nyx                  ?  de pago, sin rastro
56  pi-vis               —  no es plataforma: GUI de un agente
57  DeepCode             ✗  cero evidencia de uso real · auditada
58  edict                ✗  muerto por dentro · auditada
59  Amp                  —  dentro del agente, sin repo público · npm 78.778/mes
60  Cursor               —  IDE
61  Warp                 —  terminal
62  Zed                  —  editor
63  VS Code              —  editor
64  Codex app            —  propietaria, sin cifra
65  Antigravity          —  editor
66  Devin                —  servicio cerrado de un proveedor
67  worktrunk            ?  8.446★, sin testimonio
68  ccpm                 ?  8.391★, parada desde marzo
69  treehouse            ?  1.797★, sin testimonio
70  arbor                ?  829★, sin testimonio
71  codexia              ?  921★, sin testimonio
72  uzi                  ?  583★, sin testimonio
73  FleetCode            ?  424★, parada desde marzo
74  cc-haha              ?  14.749★, sin testimonio
75  pi-subagents         ?  1.225★, sin testimonio
76  helix                ?  812★, sin testimonio
```

## Enlaces

- [[plataformas-uso-real]] — la tabla de medición, plataforma a plataforma
- [[plataformas-auditadas]] — las cinco auditadas por dentro
- [[lo-que-dice-la-comunidad]] — el veredicto de la comunidad sobre Lego en general
- [[limites-del-andamiaje]] — la mejor crítica a la categoría
