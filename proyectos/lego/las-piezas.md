---
title: Lego — las piezas, una por una: qué es cada cosa y de quién es
created: 2026-09-28
updated: 2026-09-29
tags: [lego, piezas, genealogia, gas-town, autores]
zona: tecnico
---

Cada pieza del sistema, **qué es exactamente**, quién la hizo, qué aporta que no aporten las demás, y **cuántas piezas añade**. Los ficheros clave están descargados y leídos, no resumidos de oídas.

---

# BLOQUE 1 · La comunidad

> ## ⚠️ CORREGIDO EL 2026-09-28 — esto estaba mal listado
>
> **«Ralph Wiggum» no es una pieza: es el nombre de la técnica.** Y en la versión anterior de esta nota aparecía **dos veces** — como «el bucle» y como «el bucle con piezas de seguridad» — **como si fueran dos cosas distintas**. No lo son: **es una técnica con muchas implementaciones.** Se han fusionado aquí, y se han listado las implementaciones reales.

## 1.1 · Ralph Wiggum — **la técnica** — de **Geoffrey Huntley**

**Qué es.** **El nombre de la idea**, no de un programa. Una línea de bash. Literalmente:

```bash
while :; do cat PROMPT.md | claude ; done
```

**Qué aporta que no aporte nada más: la idea central de todo esto.** Cada vuelta arranca **una instancia nueva**, con contexto limpio, y lo que persiste está **fuera** del agente: en los ficheros y en git. Huntley lo resumió como *«mejor fallar de forma predecible que acertar de forma impredecible»*.

**Lo llamó «Ralph Wiggum»** por el personaje de los Simpson: insistir una y otra vez sin darse por vencido.

**Quién es.** Desarrollador australiano, ex-Canva, pasó por Sourcegraph y Amp. Su canal de YouTube tiene **10.000 suscriptores** — la fuente primaria es la que menos audiencia tiene de todo el sector.

**Su otro material:** [`ghuntley/how-to-ralph-wiggum`](https://github.com/ghuntley/how-to-ralph-wiggum), **1.800★**, donde está el bucle mínimo y su frase de seguridad —*«no es si va a petar, es cuándo. **¿Y cuál es el radio de la explosión?»***— y su taller [`how-to-build-a-coding-agent`](https://github.com/ghuntley/how-to-build-a-coding-agent), **5.851★**, que construye un agente **en Go** por incrementos: del chat a la lectura, al listado, a bash, a la edición, a la búsqueda. *«300 líneas de código en un bucle.»*

**Sus propios fracasos, en su texto:** *«te despertarás con una base de código rota que no compila de vez en cuando»*, *«Claude tiene el sesgo inherente de hacer implementaciones mínimas y de relleno»*, *«no usaría Ralph en una base de código existente ni de broma»*.

**Piezas que añade: 0.** Es una línea de bash.

### Y sus implementaciones — porque hay doce o más

**Ninguna es «la» de Ralph: todas son implementaciones de la misma técnica, y casi todas reutilizan el nombre.** Verificado con `gh api`:

| Implementación | Estrellas |
|---|---|
| [`mikeyobrien/ralph-orchestrator`](https://github.com/mikeyobrien/ralph-orchestrator) | 3.160★ |
| [`michaelshimeles/ralphy`](https://github.com/michaelshimeles/ralphy) | 2.976★ |
| [`Th0rgal/open-ralph-wiggum`](https://github.com/Th0rgal/open-ralph-wiggum) | 1.893★ |
| **[`ghuntley/how-to-ralph-wiggum`](https://github.com/ghuntley/how-to-ralph-wiggum)** ← **la del creador** | **1.800★** |
| [`muratcankoylan/ralph-wiggum-marketer`](https://github.com/muratcankoylan/ralph-wiggum-marketer) | 779★ |
| [`tzachbon/smart-ralph`](https://github.com/tzachbon/smart-ralph) | 550★ |
| [`agrimsingh/ralph-wiggum-cursor`](https://github.com/agrimsingh/ralph-wiggum-cursor) | 496★ |
| **[`fstandhartinger/ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum)** ← **la que está en [[receta-completa]]** | **300★** |
| [`coleam00/ralph-loop-quickstart`](https://github.com/coleam00/ralph-loop-quickstart) | 159★ |
| `AsyncFuncAI/ralph-wiggum-extension`, `mikehostetler/wreckit`, `eduardolat/clancy` | 119–134★ |

**Y el detalle que lo dice todo: el repositorio del propio creador tiene menos estrellas que dos reimplementaciones suyas.** Eso pasa cuando una técnica no tiene producto dueño.

**Lo que aporta la implementación que está en la receta** —`fstandhartinger`— frente a la del creador: **cortacircuitos, contador de intentos por tarea, analizador de respuestas y avisos**, además de la promesa de fin. **Es una elección, no «la» implementación.**

---

## 1.2 · Gas Town — **Steve Yegge**

**Qué es.** Un **gestor de espacio de trabajo multiagente**. **18.205★.**

> **Corregido:** antes decía aquí que era «la respuesta más completa a la fábrica que existe como proyecto abierto». **Es falso.** Verificado con `gh api`, hay **varias plataformas del mismo tipo y dos son más grandes**: [`oh-my-claudecode`](https://github.com/Yeachan-Heo/oh-my-claudecode) con **39.393★** y [`paseo`](https://github.com/getpaseo/paseo) con **18.884★**. Gas Town es **una de ellas**, no la mayor.

**Su vocabulario, que es lo que hay que entender:**

| Término | Qué es |
|---|---|
| **El Alcalde** 🎩 | Tu coordinador. **Es una instancia de Claude Code** con contexto de todo el espacio de trabajo |
| **El Pueblo** 🏘️ | Tu directorio de trabajo (`~/gt/`). Contiene todos los proyectos |
| **Los Rigs** 🏗️ | Contenedores de proyecto. **Cada uno envuelve un repositorio git** |
| **Miembros de tripulación** 👤 | Tu espacio personal dentro de un rig, donde trabajas a mano |
| **Las Mofetas** 🦨 | **Los agentes trabajadores.** Identidad persistente, **sesiones efímeras**: se lanzan para una tarea, la sesión termina al acabarla, pero la identidad y el historial sobreviven |
| **Los Ganchos** 🪝 | **Almacenamiento persistente basado en worktrees de git.** Sobrevive a caídas y reinicios |
| **Los Convoyes** 🚚 | Unidades de seguimiento. Agrupan varias *beads* y se asignan a agentes |
| **Las Moléculas** 🧬 | **Plantillas de flujo de trabajo.** Se definen en TOML y se instancian con pasos rastreados |
| **El Vigilante, el Diácono y los Perros** 🐕 | **Tres niveles de vigilancia**: por rig, global, y trabajadores de mantenimiento |
| **La Refinería** 🏭 | **Cola de fusión por rig.** Cuando una mofeta termina, la Refinería agrupa, pasa las compuertas de verificación y fusiona con una **cola de bisección estilo Bors** |
| **La Escalada** 🚨 | Escalado por severidad (crítico, alto, medio) que pasa por el Diácono y el Alcalde |
| **El Planificador** ⏱️ | **Gobernador de capacidad.** Evita agotar el límite de peticiones de la API |
| **El Seánce** 👻 | Descubre **sesiones anteriores** del agente, para que pueda preguntar a sus predecesores |
| **El Yermo** 🏜️ | **Red federada**: los pueblos publican trabajo y lo reclaman entre ellos, con reputación portátil |

**Lo que aporta que no aporten los demás:** **todo lo que un montaje de un agente no tiene** — vigilancia de tres niveles, cola de fusión con bisección, escalado por severidad, gobernador de límite de peticiones, y memoria entre sesiones.

**Lo que cuesta, y aquí está el problema:** sus requisitos *literales del README* son **Git 2.20+, Go 1.26.2+, Beads 0.57+, sqlite3, cabeceras de ICU4C, tmux 3.0+ y el CLI de Claude Code**. **Siete piezas**, varias de ellas de sistema.

**Mi lectura, marcada como mía:** esto **no es un montaje, es una plataforma.** Está pensado para llevar 20 o 30 agentes a la vez sobre varios proyectos. **Para lo que tú quieres —una persona, un proyecto, un agente— sobra por todos lados**, y cada pieza es algo que puede romperse un domingo por la tarde.

**Y hay controversia documentada, verificada en vivo el 2026-09-28** (corrige la versión anterior de esta nota, que la atribuía mal a un hilo de Hacker News): es [issue #3649 de su propio repositorio](https://github.com/gastownhall/gastown/issues/3649), *«Does Gas Town "steal" usage from users' LLM credits & paid services to improve itself?»* — **cerrado**. Cita literal: *«tus créditos de Claude / tu uso pueden estar financiando arreglos al código del mantenedor, y tu cuenta de GitHub envía PRs a su repositorio»*, y *«no hay opt-in, no hay opt-out, no hay aviso»*.

> ### ⚠️ SI SE ELIGE GAS TOWN: apagar antes de usarlo que arregle bugs de sí mismo y mande PRs con tus créditos y tu cuenta de GitHub
>
> Por defecto trae un flujo de «contribuir de vuelta a upstream»: puede lanzar agentes que arreglen fallos del propio Gas Town y manden ese parche como PR a su repositorio, gastando tus créditos de Claude y usando tu cuenta de GitHub, sin pedir permiso.
>
> **Buscado en vivo el 2026-09-28 y no encontrado: ningún flag, variable de entorno ni fórmula con nombre para desactivarlo.** El propio issue #3649 lo dice explícito: *«no hay opt-in, no hay opt-out»*.

**RESUELTO el 2026-09-29 — y corrige una afirmación mía.** Este aviso pedía «revisar el estado del issue #3649 por si se resolvió». Revisado. Y lo que encontré corrige dos cosas de arriba:

1. **Sí hay respuesta en el cierre, y no dice «lo hemos quitado».** El comentario que cierra el issue es: *«Gastown is in maintenance mode and staying focused on infrastructure and reliability fixes only. If you want to pursue broader product/policy work like this, please check out Gas City instead.»* Es decir, se cerró **redirigiendo a Gas City**, no confirmando una retirada.
2. **Y por eso había que comprobarlo en Gas City, no darlo por heredado.** Comprobado sobre su repositorio y su catálogo de packs: **no arrastra el mecanismo.** No hay ninguna fórmula de release en el pack `core`, la búsqueda en todo el catálogo oficial da vacío, y el único código Go que menciona `gastownhall/*` son rutas de import del propio módulo. La función existe — pero como pack **`contributing`**, aparte, opt-in, y cuya cabecera declara que su propósito es que contribuyas tú: *«the external-contributor lifecycle for gastownhall/gascity»*. Detalle verificado en [[gas-city-instalacion-y-modelos]] §5.1.

**Lo que sí hay que apagar en Gas City es otra cosa:** su **telemetría de producto**, que viene **activada por defecto** (opt-out, no opt-in) y se apaga con `gc metrics off`, `DO_NOT_TRACK=1` o `GC_DISABLE_USAGE_METRICS=1`. Nunca recoge nada en sesiones de agente, CI o scripts. Ver [[gas-city-instalacion-y-modelos]] §5.2.

**Piezas que añade: 7 o más.**

**Actualización 2026-09-29:** Gas Town tiene sucesor, **Gas City**, del mismo equipo; su propia organización presenta Gas Town como *«the predecessor software-factory project that inspired Gas City»*. El montaje con Gas City está en [[gas-city-traje-a-medida]]; la instalación en esta máquina, el reparto de modelos entre Opus/Sonnet/DeepSeek y qué apagar, en [[gas-city-instalacion-y-modelos]]; y cómo usarlo con el wiki para desarrollar aplicaciones, en [[gas-city-con-2cerebro]].

---

## 1.3 · Beads — también **Steve Yegge**

**Qué es.** El **libro mayor de tareas**: un rastreador de incidencias con **grafo de dependencias**, hecho para agentes. **27.488★** — más que Gas Town.

**Y sí: va con Dolt**, con L. Confirmado en su README. **Dolt es «git para datos»**: una base de datos SQL con ramas, fusiones y sincronización, como git pero para tablas.

| Modo | Comando | Qué hace |
|---|---|---|
| **Embebido** (por defecto) | `bd init` | Dolt corre dentro del proceso. **Un solo escritor.** Datos en `.beads/embeddeddolt/` |
| **Servidor** | `bd init --server` | Se conecta a un `dolt sql-server`. **Varios escritores a la vez** |

**Ojo con una confusión frecuente:** el fichero `.beads/issues.jsonl` **no es la base de datos**. Dice su README que es *«una exportación para visores e intercambio, **no la fuente de verdad ni una copia de seguridad**»*.

**Qué aporta que no aporte el markdown:**

- **Grafo de dependencias** — «reemplaza los planes markdown desordenados por un grafo con dependencias».
- **IDs con hash** (`bd-a1b2`) que **evitan las colisiones de fusión** cuando varios agentes trabajan a la vez. Es lo que el `TASKS.md` admite que no resuelve: *«dos agentes leyendo a la vez **todavía pueden competir**»*.
- **Compactación** — «decaimiento semántico de memoria»: resume las tareas viejas cerradas para ahorrar contexto.
- **Mensajería** con hilos.

**Lo que cuesta:** **Dolt es una pieza más** — una base de datos que mantener. Y tiene **1.286 incidencias abiertas** sobre 27.488 estrellas.

**Piezas que añade: 1** (Dolt), más el propio `bd`.

---

## 1.4 · El Playbook — **`ClaytonFarr`**

**Qué es.** **No es una herramienta: es el manual.** Lo hizo en **diciembre de 2025** alguien que se dedicó a leer el material del propio Huntley —sus vídeos y su artículo— para *«leer las hojas del té lo más de cerca posible de la persona que no sólo capturó este enfoque, sino que es quien más horas de asiento lleva poniéndolo a prueba»*. *(Que sea el mejor escrito es **juicio mío**, no un dato.)*

**Su aportación central: explicar que Ralph no es «un bucle que programa», es un embudo.**

### Tres fases, dos prompts, un bucle

| Fase | Qué es | Dónde |
|---|---|---|
| **1. Definir requisitos** | **No es el bucle.** Es una conversación: ideas → **trabajos a realizar (JTBD)** → temas → **un `specs/fichero.md` por tema** | A mano, con el modelo |
| **2. Planificar** | Análisis de huecos entre las specs y el código → genera `IMPLEMENTATION_PLAN.md`. **Sin implementar nada** | **El bucle** |
| **3. Construir** | Coge la tarea más importante del plan, implementa, prueba, commit, y actualiza el plan | **El bucle** |

**El vocabulario que fija —y que ningún otro sitio explica:**

| Término | Qué es |
|---|---|
| **JTBD** (trabajo a realizar) | La necesidad de alto nivel del usuario |
| **Tema** | Un aspecto concreto dentro de ese trabajo |
| **Spec** | El documento de requisitos de **un** tema |
| **Tarea** | La unidad derivada de comparar la spec con el código |

**Y las relaciones:** 1 JTBD → varios temas · 1 tema → **1 spec** · 1 spec → **varias tareas**.

### El test de alcance, que es lo más práctico del documento

> **«Una frase sin “y”.»**
>
> ¿Puedes describir el tema en una frase **sin** unir cosas que no van juntas?
>
> ✅ «El sistema de extracción de color analiza imágenes para identificar los colores dominantes»
> ❌ «El sistema de usuario gestiona autenticación, perfiles **y** facturación» → **son tres temas**

**Si necesitas un «y» para describirlo, son varios temas.** Con eso se decide cuándo partir una tarea, sin discutir.

### Los números del contexto, que son los únicos concretos que he visto

- **200.000 tokens anunciados = ~176.000 realmente usables.**
- **Del 40% al 60% de ocupación** es la «zona lista» del modelo.
- **Tareas ajustadas + una tarea por vuelta = el 100% del contexto en la zona lista.**
- Usa el agente principal **como planificador**, no como trabajador: **los subagentes tienen ~156 KB propios y se recogen solos**.
- **Markdown antes que JSON** — mejor eficiencia de tokens.
- **Los primeros ~5.000 tokens, para las specs.** Y cada vuelta carga los mismos ficheros, para que el modelo arranque siempre desde un estado conocido.

### El ciclo de construcción, en 10 pasos

Orientar (leer las specs) → leer el plan → **elegir** la tarea más importante → investigar el código (*«no asumas que no está hecho»*) → implementar (varios subagentes) → **validar (uno solo, con los tests como contrapresión)** → actualizar el plan → actualizar las reglas si aprendió algo → commit → **el contexto se borra y empieza la siguiente.**

### Y aquí está la parte que a mí me faltaba: **cómo se afina**

> **«Afínalo como una guitarra.»** No prescribas todo por adelantado: **observa y ajusta cuando falle.**

El método concreto:
1. **Empieza con el fichero de reglas VACÍO.** Ni «buenas prácticas» ni nada.
2. Mira las primeras vueltas y **ve dónde falla**.
3. **Añade una señal sólo cuando haga falta.**
4. Y las señales **no son sólo texto del prompt**: son *«cualquier cosa que Ralph pueda descubrir»* — guardarraíles en el prompt, aprendizajes en las reglas, **y utilidades en tu propio código**, que el agente descubre y sigue.

**Y el plan es desechable.** Si está mal, se tira y se rehace: *«el coste de regenerarlo es una vuelta de planificación, barato comparado con Ralph dando vueltas en círculo»*. Cuándo regenerar: cuando va desviado, cuando el plan está viejo, cuando se ha llenado de cosas ya hechas.

**Y su frase que resume tu papel:** **«siéntate sobre el bucle, no dentro de él»** — tu trabajo es montar el entorno, no hacer las tareas.

### Y aquí la seguridad no es un añadido, es la premisa

El Playbook no lo esconde, lo pone en el centro:

> *«Para operar de forma autónoma, Ralph **requiere `--dangerously-skip-permissions`** — pedir aprobación en cada llamada rompería el bucle. Esto **anula por completo el sistema de permisos**, así que **el entorno aislado pasa a ser tu única frontera de seguridad**.»*

Y su filosofía, que es la que ya conocías: *«**no es si va a petar, es cuándo. ¿Y cuál es el radio de la explosión?»*. Y el detalle de qué queda expuesto sin él: *«credenciales, cookies del navegador, claves SSH y tokens de acceso de tu máquina»*.

**Ésa es la diferencia clave con el montaje que te propuse:** el Playbook acepta anular los permisos y compensa con el entorno aislado. **Yo te propuse lo contrario: permisos declarados** — que es más seguro y tiene el mismo efecto, porque lo que Ralph necesita es **no pararse a preguntar**, y una lista de permitidos ya lo consigue sin abrir la máquina entera.

**Piezas que añade: 0.** Es leer — pero es el que más enseña.

---

## 1.5 · El «post-vibe-coding» — **Andrej Karpathy**

**Qué es.** Una **tesis**, no una herramienta. Karpathy acuñó «vibe coding» y después marcó la salida: la idea de que pedir código sin especificar no escala, y que el valor vuelve a estar en describir bien lo que se quiere.

**Lo que aporta:** **el marco mental.** Todo el movimiento del desarrollo dirigido por especificación lo cita como origen. El artículo que prometía «más de 30 frameworks» dice literalmente que tras Spec Kit y Kiro vino *«la tesis post-vibe-coding de Andrej Karpathy»*.

**Y una conexión curiosa:** el compilador determinista `archiet-microcodegen` dice inspirarse en *«el micrograd de Karpathy»*.

**Piezas que añade: 0.** Es una idea.

---

# BLOQUE 2 · Los fabricantes

## 2.1 · `AGENTS.md` — **OpenAI**

**Qué es.** **Un fichero markdown en la raíz del repositorio** con las instrucciones del proyecto. Nada más. Lanzado en **agosto de 2025**, y hoy está bajo la **Linux Foundation**.

**Lo que aporta:** **el estándar de facto para las reglas del proyecto.** Lo leen Codex, Cursor, Copilot, Jules, Amp y más. OpenAI reportó **60.000 proyectos** adoptándolo.

**Y su límite, medido:** un estudio sobre repositorios reales encontró que **sólo el 5%** (466 de 10.000) había adoptado algún formato de estos. Y su crítica más citada: *«es un parche, no una solución»* — es estático, es prosa sin estructura, y está siempre activo aunque no venga al caso.

**Piezas que añade: 0.** Es un fichero.

---

## 2.2 · Spec Kit y la «constitución» — **GitHub**

**Qué es.** **139.234★.** El toolkit que impone el formato: `constitution.md` + `spec.md` + `plan.md` + `tasks.md`, con requisitos numerados `FR-001` y marcadores `[NEEDS CLARIFICATION]`.

**Lo que aporta, y es su mejor idea: la constitución.** Un único fichero con las reglas del proyecto que **todas las especificaciones heredan**.

**Lo que cuesta:** Python 3.11+, `uv`, su CLI y un agente. **Cuatro piezas** para algo que cabe en tres.

**Y su rendimiento medido no compensa:** un análisis documentado generó **2.577 líneas de markdown para 689 de código**, y tardó *«unas diez veces más»* que iterar a mano. Su conclusión: *«no resultó en mejor código ni menos fallos»*.

**Piezas que añade: 2** (Python y su CLI).

---

## 2.3 · Kiro — **AWS**

**Qué es.** Un **IDE completo** con el flujo de especificación incorporado: `requirements.md` (con notación EARS), `design.md` y `tasks.md`.

**Lo que aporta:** **la notación formal de los criterios** — y el «spec check» que comprueba matemáticamente que los requisitos no se contradicen.

**Lo que cuesta:** es propietario y te ata a AWS. **Y la notación no es suya** — es de Rolls-Royce, 2009 (ver 4.1).

**Piezas que añade: 1**, y es un IDE entero.

---

## 2.4 · Los subagentes, skills y hooks — **Anthropic**

**Qué es.** El **andamiaje de Claude Code**: subagentes con ventana de contexto propia, *skills* reutilizables, *hooks* que se disparan en eventos, y modos de permisos.

**Lo que aporta, y es lo que más falta en los demás montajes: los permisos y los hooks.** Son **lo único que obliga de verdad**. La frase que lo resume, de un incidente real: *«Opus 4.7 **leyó esas reglas, las reconoció y las violó** en la siguiente entrada del registro»*. **La prosa no obliga. El entorno sí.**

**Y su formación, que es gratis y en castellano:** el curso [Claude Code en acción](https://academy.claude.com/es/courses/claude-code-in-action) enseña literalmente esto: *«imponer las reglas no negociables con hooks»* y *«verificar las ejecuciones no supervisadas en proporción a lo poco que miraste»*.

**Piezas que añade: 0** — van dentro del agente que ya tienes.

---

## 2.5 · «Harness engineering» — OpenAI y la comunidad

**Qué es.** **El nombre del concepto.** Lo popularizaron una charla de OpenAI (Ryan Lopopolo, **241.972 visualizaciones**) y un artículo de Lilian Weng. «Arnés» = el conjunto de guardarraíles alrededor del agente.

**Lo que aporta:** **el vocabulario.** El material del curso que me diste usa exactamente este término.

**Piezas que añade: 0.** Es un concepto.

---

# BLOQUE 3 · Industria y academia

## 3.1 · EARS — **Alistair Mavin, Rolls-Royce**

**Qué es.** El **formato fijo de los criterios de aceptación**: CUANDO… ENTONCES… DEBE. Presentado en el IEEE de Ingeniería de Requisitos en **2009**.

**Lo que aporta:** **es lo único de todo este documento que no viene de la IA.** Viene de la aviación. Y es lo que hace que un criterio **no se pueda interpretar de dos maneras** — que es exactamente el problema que el resto del sistema intenta resolver.

**Piezas que añade: 0.** Es una convención de escritura.

---

## 3.2 · El coste de los defectos — **Boehm y Basili**

**Qué es.** El estudio clásico de 1981, revisado en 2001: **arreglar un defecto tarde cuesta más que arreglarlo pronto**. Cifras de 1981: 1 $ en requisitos, 5–10 $ en diseño, 10–20 $ en código, hasta **1.000 $ en operación**.

**Lo que aporta:** **la justificación económica de escribir bien la tarea antes de empezar.**

**Y el matiz que corrige la cita popular:** la regla «1-10-100» que circula por todas partes es una simplificación. Boehm y Basili encontraron que **para equipos pequeños con integración continua la pendiente es mucho más plana** — «hacia 5:1».

**Y otro dato que hay que dejar de citar:** el famoso «87,5% de los defectos vienen de requisitos» **no tiene fuente primaria**. Es folclore.

**Piezas que añade: 0.**

---

# BLOQUE 4 · Las empresas que lo montaron

| Empresa | Qué montó | Qué publicó |
|---|---|---|
| **StrongDM** | «La fábrica de software»: «el código no debe ser escrito por humanos», «no debe ser revisado por humanos» | **Ninguna métrica de resultado.** Sólo el eslogan de los 1.000 $/día/ingeniero en tokens |
| **Stripe** | Los *minions*: «más de mil PRs por semana sin código humano» | Sin denominador. La comunidad calculó que es **menos de 1 PR por ingeniero y semana** |
| **Spotify** | Agentes de fondo | **1.500+ PRs** y «alrededor de la mitad» de sus PRs automatizados |
| **Uber** | Agentes internos | 11% de PRs abiertos por agentes. **Presupuesto anual de IA agotado en 4 meses** |

**Comunidad:** todas juntas, y **ninguna publica cuántas tareas salieron bien de cuántas**.

---

# La tabla final: qué usa qué

| Si quieres… | La pieza es | De quién |
|---|---|---|
| **El bucle** | **Ralph Wiggum** — la técnica. Sus implementaciones son doce | **Huntley** |
| Una implementación con seguridad | `fstandhartinger/ralph-wiggum` — **una de las doce**, no «la» | Comunidad |
| El formato de la tarea | EARS + los campos de Spec Kit | **Rolls-Royce 2009** + GitHub |
| Las reglas del proyecto | `AGENTS.md` | **OpenAI** |
| Que las reglas **obliguen** | Permisos y hooks | **Anthropic** |
| Cambiar de modelo | `claude-code-router` | Comunidad |
| Muchos agentes a la vez | Una plataforma: Gas Town, `oh-my-claudecode` (39.393★), `paseo`… | **Steve Yegge** y otros |
| Recordar tareas con dependencias | Beads (con **Dolt**) | **Steve Yegge** |
| No equivocarte al elegir | *(no existe)* | — |

## Lo que esto demuestra, y es lo importante

**Ningún fabricante te vende el conjunto.** Anthropic empuja los subagentes, OpenAI el fichero de reglas, GitHub la especificación, AWS el IDE. **Cada uno tiene su pieza y ninguno tiene incentivo en decirte que hace falta la del vecino.**

**Y los que sí montaron el conjunto son dos desarrolladores sueltos:** Huntley con el bucle y Yegge con la fábrica. **La industria lo está empaquetando después.**

## Enlaces

- [[todo-lo-necesario]] — el montaje completo, con tus modelos
- [[receta-completa]] — los ficheros del bucle, uno a uno
- [[quien-dice-que]] — la auditoría de cada fuente
- [[etapas]] — qué está maduro y qué necesita tu mano
