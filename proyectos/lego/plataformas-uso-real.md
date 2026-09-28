---
title: Lego — las 60 plataformas del catálogo, con uso medido
created: 2026-09-28
updated: 2026-09-28
tags: [lego, plataformas, medicion, comunidad]
zona: tecnico
---

Barrido ancho, no profundo: las **60 entradas** del catálogo [agentmgmt.dev](https://agentmgmt.dev/) —el índice entero, leído de su fuente en [`manuelmeurer/agentmgmt/src/tools.yml`](https://github.com/manuelmeurer/agentmgmt/blob/main/src/tools.yml)— cada una con lo que se puede medir por fuera: estrellas, licencia, último empujón, descargas, y **qué dice la comunidad con la cifra del hilo delante**. Contrasta con [[plataformas-auditadas]], que audita cinco por dentro.

Medido el **2026-09-28**.

---

## El criterio con el que entran

- **E1 · Es plataforma** — orquesta ejecución de agentes de código, no es un editor, ni un asistente de una sesión, ni un cliente remoto.
- **E2 · Uso ajeno** — cifra de descargas/instalaciones **o** un hilo donde un tercero cuente que la usó. Las estrellas y el README **no** cuentan.
- **E3 · Trazable** — repo o docs con historial.

---

## Tabla 1 · Las que pasan E1 y tienen uso ajeno medible

Las descargas de release son la **suma de todos los assets de todas las releases** (incluye actualizaciones automáticas y bots) — sirven para orden de magnitud, no como cifra absoluta.

| Plataforma | Repo | ★ | Último push | Licencia | Uso medible | Comunidad (puntos / comentarios) |
|---|---|---|---|---|---|---|
| **Herdr** | ogulcancelik/herdr | 41.253 | 28-sep | Apache-2.0 | brew **8.151**/30d · npm **300**/mes · release 159.279 | [404/178](https://news.ycombinator.com/item?id=48756578), [281/189](https://news.ycombinator.com/item?id=49201003), [166/110](https://news.ycombinator.com/item?id=48714802) — **el mayor volumen del barrido** |
| **Happy** | slopus/happy | 23.939 | 28-sep | MIT | npm **33.712**/mes (+`happy-coder` 1.695) | [30/8](https://news.ycombinator.com/item?id=44904039) |
| **Orca** | stablyai/orca | 80.619 | 28-sep | MIT | release **72.272.643** (acumulado) | [190/93](https://news.ycombinator.com/item?id=49233448); 5 autores distintos en comentarios |
| **cmux** | manaflow-ai/cmux | 27.472 | 28-sep | NOASSERTION | release 511.670 en v0.64.25 · npm **1.543**/mes | [198/77](https://news.ycombinator.com/item?id=47079718) |
| **Paseo** | getpaseo/paseo | 18.889 | 28-sep | NOASSERTION | release **622.764** en v0.9.2 | el más elogiado en [49233448](https://news.ycombinator.com/item?id=49233448) |
| **Vibe Kanban** | BloopAI/vibe-kanban | 28.214 | 19-sep | Apache-2.0 | npm **9.010**/mes | [195/132](https://news.ycombinator.com/item?id=44533004) — **el README dice «sunsetting»** |
| **OpenChamber** | openchamber/openchamber | 10.875 | 28-sep | MIT | release 134.160 en v2.0.2 | [190/93](https://news.ycombinator.com/item?id=49233448), varios usuarios con fallos concretos |
| **Emdash** | generalaction/emdash | 5.868 | 27-sep | Apache-2.0 | release 71.229 en v1.2.6 | [206/71](https://news.ycombinator.com/item?id=47140322) |
| **kandev** | kdlbs/kandev | 859 | 28-sep | AGPL-3.0 | npm **5.969**/mes (paquete verificado por `repository`) | 2 comentarios del **mismo** usuario |
| **tmux-ide** | wavyrai/tmux-ide | 550 | 28-sep | MIT | npm **4.534**/mes (verificado) | [88/38](https://news.ycombinator.com/item?id=47428868) |
| **Agent of Empires** | njbrake/agent-of-empires | 3.298 | 28-sep | MIT | release 703 en v1.16.1 | [118/44](https://news.ycombinator.com/item?id=46588905) + un «daily driver» |
| **Superset** | superset-sh/superset | 14.711 | 28-sep | **Elastic 2.0** | release 4.602.467 | [108/135](https://news.ycombinator.com/item?id=48236770) |
| **dmux** | standardagents/dmux | 1.788 | **16-ago** | MIT | npm **915**/mes (verificado) | [9/0](https://news.ycombinator.com/item?id=47075312) |
| **Nimbalyst** | nimbalyst/nimbalyst | 1.796 | 27-sep | MIT | release 135.395 en v0.78.5 | hilos pequeños, uno del propio autor |
| **Xum** | coder/xum | 2.040 | 28-sep | AGPL-3.0 | **sin cifra** | sin nada |
| **nodeterm** | eneskirca/nodeterm | 1.916 | 28-sep | NOASSERTION | release 149.797 | sin nada |
| **CodeNomad** | NeuralNomadsAI/CodeNomad | 2.608 | 28-sep | MIT | release 643 · **3 usuarios independientes en comentarios** | mejor comunidad que cifra |
| **Ghostex** | maddada/Ghostex | 848 | 28-sep | MIT | release 281-373 por versión | sin nada |
| **Agent Orchestrator** | AgentWrapper/agent-orchestrator | 12.493 | 28-sep | Apache-2.0 | **sin cifra atribuible** | su Show HN: [15/0](https://news.ycombinator.com/item?id=47562440) |
| **jean** | coollabsio/jean | 1.300 | 28-sep | Apache-2.0 | release 6.955 · hereda la comunidad de Coolify, no la suya | sin nada propio |

## Tabla 2 · Pasan E1 pero **no** tienen uso ajeno verificable

| Plataforma | ★ | Último push | Por qué no entra como recomendable |
|---|---|---|---|
| Conductor | — | — | Propietario, sin repo; encaja en el diseño pero **cero** evidencia de terceros |
| Superconductor | — | — | Toda su prueba es del vendor («1.810 implementaciones», sin verificar) |
| atrium | — | — | [3/0](https://news.ycombinator.com/item?id=48208747) en HN |
| Solo | — | — | Propietario; concepto exacto, ninguna señal de uso |
| Clor | — | — | [11/5](https://news.ycombinator.com/item?id=48375347) |
| Luvus | 929 | 28-sep | Sin cifras ni hilos |
| Acepe | 90 | 01-sep | Sin cifras ni hilos |
| Sidequest | 10 | 26-sep | Sin cifras ni hilos |
| Fusion | 1.249 | 27-sep | Release más descargado: **18**. Y su verificación adversarial es **código muerto** |
| Shep | 86 | 20-sep | **Sin licencia** (no es open source aunque el catálogo lo diga), 74 descargas |
| Rabbitty | 11 | 28-ago | Fuente privada, 7 descargas |
| Codex app · Cursor · Devin | — | — | Cerrados, sin cifra pública; Cursor entra por E1 (8 agentes en paralelo) |

## Tabla 3 · Fuera por E1, con la cifra delante

No son plataformas, pero su uso medido sirve de contexto: **Zed** 91.017★ (editor), **Warp** 65.232★ (terminal), **Xirp**, **Polyscope**, **AgentsDock** (81★, [81/32](https://news.ycombinator.com/item?id=49678435) — el mejor hilo del lote y es un IDE), **pi-vis**, **Muxy**, **HAPI** (brew 10/30d), **T3 Code** 23.807★ con hilos de 6 puntos, **Opcode** 22.412★ **con último commit de código en octubre de 2025**.

---

## Lo que dice la comunidad, y no es lo que dicen los README

**1 · Los que lo usan describen la misma pieza de debajo, no el producto.** El hilo [Ask HN: Are you using an agent orchestrator to write code?](https://news.ycombinator.com/item?id=46993479) (41 pts, 60 comentarios) es el más útil de todo el barrido, y **nadie habla de una plataforma**:

> «I tend to have between 4-15 agent sessions going at once… My agent orchestration system is a bespoke python program that I vibed just for me.» — kasey_junk, 2026-02-16

> «One task per session, each in its own git worktree, so two agents never edit the same checkout. Stop polling. Claude Code's Notification hook fires when it's waiting on a permission prompt or sitting idle» — parpil, 2026-09-11

**2 · El elogio real siempre trae un fallo concreto detrás.** Herdr, que es la más querida: «I did try Herdr but found too many things I expected it to have and just didn't» (InvidFlower); «with work done on 5 remote machines herdr just gets laggy after couple of hours and basically becomes almost unusable» (rs_rs_rs_rs_rs, [45 pts](https://news.ycombinator.com/item?id=49652188)). OpenChamber: «I frequently use openchamber on the go. It's very good at this» + «It doesn't work with tailscale» (kyriakos).

**3 · La acusación de escaparate existe y está documentada.** En el hilo de entrada de Herdr en YC ([281/189](https://news.ycombinator.com/item?id=49201003)): «All SaaS companies saying Used by people in Google just because one guy in support used it one time 5 years ago» (victorbjorklund). Y sobre la propia categoría, la mejor frase del barrido, de un usuario de Orca: «Even though all these "ADE"s are mostly the exact same (Superset, T3, Conductor, Paseo, random Reddit-dude, etc.)» (vulture916) — con otro usuario de Orca admitiendo: «Orca is my main ADE too, but I haven't found any other users!» (epistasis).

**4 · El rechazo explícito al vendor como dependencia.** El motivo por el que un ingeniero elige una herramienta pequeña: «I don't want to add a new vendor in the mix for our infra that may not exist in a year» (willdoenlen).

**5 · Y en Reddit, usuarios comparando herramientas entre sí** — esto es lo que no sale en ningún README:

> «I recently started using Orca ADE for managing Claude Code sessions, and I just want to tell the community that Orca is incredible. Try it. You'll like it. **I was previously using cmux, and I've also used herdr, iTerm**, and various other options» — [r/ClaudeCode, 162 votos, 114 comentarios](https://redlib.catsarch.com/r/ClaudeCode/search?q=orca&restrict_sr=on)

> «I'm trying to adapt my dev flow and I'm missing a good orchestrator. **Loved VibeKanban at first but they are sunsetting and the last version doesn't even have the Kanban board.** A few options I found: **Nimbalyst**: trying this now…» — [r/ClaudeCode, 12 votos, 22 comentarios](https://redlib.catsarch.com/r/ClaudeCode/search?q=%22vibe+kanban%22&restrict_sr=on)

> «I'm getting tired of having a million unorganised terminal tabs open… I know of **Conductor, Paseo**» — [r/ClaudeCode, 82 votos, 93 comentarios](https://redlib.catsarch.com/r/ClaudeCode/search?q=orchestrator&restrict_sr=on)

> «Cmux is my daily driver and I love how well it's built for agentic coding. But I kept hitting the same wall: too many agents in too many tabs, and no good way for them to share con[text]» — [r/cmux, 45 votos, 13 comentarios](https://redlib.catsarch.com/r/cmux/comments/1ue25i9/i_made_my_cmux_agents_talk_to_each_other_and/)

---

## Dos cosas que no cuadran, y las dejo sin resolver

**a · Orca.** 80.619★, 72,3 M de descargas de release y 7.002 incidencias abiertas — contra un hilo de 3 puntos y comentarios de usuarios que dicen no conocer a ningún otro usuario. **Una de las dos señales miente y no he podido averiguar cuál**: el endpoint público de estrellas de GitHub devuelve 404 (también para `torvalds/linux`, así que es del API, no del repo), y la curva histórica queda **sin verificar**. Se dice como hueco, no como conclusión.

**b · Colisión de nombres en npm.** `podium`, `orca`, `sidecar`, `emdash`, `muxy`, `hapi`, `zeron` existen en npm como **paquetes de otros proyectos** (una librería de eventos de hapi.js, un exportador de imágenes de Plotly, un CMS de Astro…). Atribuirles esas descargas habría inflado el uso de siete plataformas de golpe. Solo cuentan las filas con el campo `repository` del paquete apuntando al repo correcto.

---

## Dónde se ha buscado

Catálogo entero enumerado desde su `tools.yml` (60 entradas) · `gh api` para las 44 con repo (estrellas, licencia, `pushed_at`, incidencias, archivado) · `api.npmjs.org` con verificación de identidad por `repository` · Homebrew `install-on-request` 30d (15.451 fórmulas) · descargas de assets de release de las 44 · HN Algolia (historias y comentarios) · **Reddit por Redlib sobre Chrome real vía CDP** — `www.reddit.com` y `old.reddit.com` devolvieron 403 y «blocked due to a network policy» a los cuatro frentes; sin esa vía, toda la evidencia de comunidad habría sido de una sola plataforma.

**Sin aportar nada:** los agregadores tipo Trendshift y SkillsMP, que listan sin dato.

**Huecos declarados:**

1. **Curva de estrellas en el tiempo:** sin verificar — el endpoint de estrellas de GitHub devuelve 404 hoy, también para `torvalds/linux`.
2. **Discords** de cmux, Emdash, Ghostex, kandev, jean y Nimbalyst: canal principal declarado por casi todos, **no legible sin cuenta**. Es el hueco más serio: es donde vive la actividad real.
3. **Cifras de uso de las propietarias** (Conductor, Superconductor, atrium, Solo, Clor, Xirp, Codex app): detrás de paneles privados. Marcadas «sin verificar», no rellenadas.
4. **`herdr` en Reddit:** dos intentos de búsqueda abortados por tiempo de espera. Su evidencia es de HN y es la más voluminosa del barrido.

## Enlaces

- [[plataformas-veredicto]] — qué encaja con Lego y qué no, con el porqué
- [[plataformas-auditadas]] — las cinco auditadas por dentro
- [[panorama-de-herramientas]] — los 30+ frameworks y el mapa de categorías
- [[piezas-y-coste]] — cuántas piezas cuesta cada enfoque
