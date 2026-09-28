---
title: Lego — cómo se consume la tarea
created: 2026-09-28
updated: 2026-09-28
tags: [lego, agentes, bucle, orquestacion]
zona: tecnico
---

Qué hace el sistema con la tarea una vez escrita: el bucle, el disparador, el aislamiento. La verificación tiene nota propia en [[verificacion-y-oraculo]].

## El bucle mínimo que usa la comunidad

El patrón se llama **Ralph Wiggum**, lo creó Geoffrey Huntley hacia mayo de 2025 y su forma original es una línea:

```bash
while :; do cat PROMPT.md | claude ; done
```

La idea es contraintuitiva y es lo mejor del patrón: **cada vuelta arranca una instancia nueva con contexto limpio**. El agente no arrastra la conversación anterior; lo que persiste está fuera de él — en git, en los ficheros y en los tests. Huntley lo resume como «mejor fallar de forma predecible que acertar de forma impredecible».

La implementación mantenida ([harrymunro/ralph-wiggum](https://github.com/harrymunro/ralph-wiggum), MIT, fork de `snarktank/ralph`) concreta el patrón en cinco ficheros:

| Fichero | Para qué |
|---|---|
| `ralph.sh` | El bucle de bash que lanza instancias nuevas de Claude Code |
| `prd.json` | La lista de tareas, cada una con `passes: false` hasta que se completa |
| `progress.txt` | Registro **solo de añadir** con lo aprendido, que leen las vueltas siguientes |
| `CLAUDE.md` | Las reglas de cada instancia |
| git | El historial de commits de las vueltas anteriores |

El ciclo, según su README: arranca instancia nueva → lee `prd.json` y coge la primera historia con `passes: false` → lee `progress.txt` → implementa → pasa las comprobaciones de calidad → hace commit y marca la historia como pasada → añade lo aprendido → sale o vuelve a empezar. **Termina cuando todas las historias están en `passes: true`** y el agente emite una promesa de finalización.

**Dependencias reales: tres.** Claude Code instalado y autenticado, `jq`, y un repositorio git. Nada más. No hay cola, ni servidor, ni base de datos.

Dos avisos que el propio README da y que concuerdan con toda la evidencia de fallo: **las tareas tienen que caber en una ventana de contexto** (si es «construye el panel entero», hay que partirla), y **el bucle solo funciona si hay un bucle de retroalimentación** — comprobación de tipos, tests, CI en verde. Sin eso, el agente no tiene forma de saber si va bien.

> **Nota sobre la versión oficial.** Anthropic empaquetó después el patrón como plugin (`/ralph-loop` con `--completion-promise` y `--max-iterations`), que usa el mecanismo de *Stop hook*: cuando el agente intenta terminar, un gancho bloquea la salida y le devuelve el mismo prompt hasta que aparezca la promesa o se agote el límite. **No he podido confirmar la fuente primaria** del nombre exacto del plugin en el mercado oficial: las fuentes secundarias se contradicen entre `ralph-wiggum` y `ralph-loop`, y hay un fallo abierto de permisos en Linux ([issue #38686](https://github.com/anthropics/claude-code/issues/38686)). Queda como **sin verificar**.

## Por qué contexto limpio y no una sesión larga

Porque el contexto se pudre. El informe técnico de Chroma ([«Context Rot», julio 2025](https://www.trychroma.com/research/context-rot)) probó **18 modelos** de Anthropic, OpenAI, Google y Alibaba, con 8 longitudes de entrada y 11 posiciones del dato en cada configuración. La conclusión, textual:

> «models do not use their context uniformly; instead, their performance grows increasingly unreliable as input length grows»

Y más: un solo distractor ya degrada el rendimiento respecto a la línea base; **desordenar el texto y quitar la coherencia local mejora** el resultado («structural coherence consistently hurts model performance»); y con prompts enfocados de ~300 tokens frente a historiales completos de ~113.000, **todos** los modelos rinden mucho mejor con el enfocado.

**Consecuencia:** no se arregla con una ventana más grande. Se arregla aislando el contexto — instancias nuevas, subagentes con su propia ventana y solo una conclusión de vuelta. El bucle Ralph acierta en esto por construcción.

## El disparador, sin añadir piezas

Tres formas, de menos a más piezas:

| Disparador | Piezas nuevas | Cuándo |
|---|---|---|
| **Tú lanzas el bucle** | 0 | Trabajo por lotes en tu máquina. El más simple |
| **`cron` + `claude -p`** | 0 (ya tienes cron) | Desatendido en tu máquina. `-p` corre sin interfaz y sale |
| **GitHub Actions** | 1 (GitHub) | Desatendido sin tu máquina encendida; el repo hace de cola y CI de ejecutor |

El modo sin interfaz está documentado: `claude -p "..."` procesa el prompt, ejecuta las herramientas, imprime el resultado y sale, **sin leer nunca de la entrada estándar** y **sin preguntar permisos** — por eso hay que declararlos con `--allowedTools` o en los ajustes. Formatos de salida: `text`, `json` y `stream-json`, este último para leer en tiempo real; `json` incluye `total_cost_usd`. Hay `--max-turns` explícitamente pensado como tope de seguridad en CI ([documentación oficial](https://code.claude.com/docs/en/headless)).

Para el caso de GitHub, el patrón es la acción oficial `anthropics/claude-code-action`: en **modo automatización** (cuando se le pasa un `prompt`) no espera a que nadie le mencione, y **por defecto escribe el resultado en el registro de la ejecución, no como comentario** — que es justo lo que se quiere para un flujo desatendido. Necesita permisos de escritura en `contents`, `pull-requests` e `issues`, y una credencial de Anthropic ([documentación oficial](https://code.claude.com/docs/en/github-actions)).

## Aislamiento: worktree o contenedor

- **Worktree de git** — cada agente en su rama, en su carpeta. Es el suelo común de *todas* las herramientas de orquestación paralela: [un repaso de más de 20 herramientas](https://singularitysociety.org/articles/tech-blog/2026-08-04-choosing-a-parallel-agent-tool-en/) concluye que «está bajo licencia MIT, corre en tu máquina e aísla con worktrees» son el punto de partida, no lo que diferencia.
- **Contenedor** — aislamiento real de ejecución. Sólo uno de los examinados lo hace: **Sculptor** (Imbue, MIT), con un contenedor Docker por agente. Es la respuesta a quien teme que un agente con permisos saltados corra contra su máquina de verdad, y cobra el precio en peso de montaje.
- **Ninguna de las herramientas examinadas aísla con contenedor por defecto.** Si se quiere sandbox de verdad, hay que añadir devcontainer o máquina virtual por debajo.

**Sobre los orquestadores de paralelo** (Conductor, Vibe Kanban, Sculptor…): son piezas que se compran con autonomía, y hay que justificarlas. El dato que decide: **Vibe Kanban** es el mayor con diferencia (~27.500 estrellas, Apache-2.0) pero **su empresa, Bloop, cerró en abril de 2026**; el proyecto sigue mantenido por la comunidad y las funciones en la nube se están apagando. **Conductor** es solo macOS Apple Silicon. Son datos de existencia y de riesgo, no de calidad.

## La evidencia de que la operación desatendida falla

Éste es el resultado más importante del tramo de consumo, y es de campo: un operador montó cinco agentes con nombre, un PM con Opus y un vigilante por cron, para una noche de trabajo desatendido. Documentó **nueve fallos distintos** ([claude-code issue #53610](https://github.com/anthropics/claude-code/issues/53610)). El resultado real: *«de 12 horas esperadas de progreso desatendido, menos de la mitad, a un coste extremo para el usuario»*, con el operador despierto a las 03:30 y 04:30 para desatascar.

Los fallos que importan, porque se repetirán en cualquier montaje:

- **Las reglas de texto no se cumplen.** El agente leía la regla que le prohibía esperar en silencio, la reconocía, y la violaba en la decisión siguiente.
- **Los permisos interrumpen.** Ninguna herramienta de orquestación estaba en la lista de permitidas, así que **cada llamada pedía aprobación de madrugada**, en el móvil.
- **«Permitir siempre» no generaliza.** Guarda la cadena literal: el mismo comando con otro argumento vuelve a preguntar. El operador pulsó «permitir siempre» cientos de veces y seguía recibiendo avisos.
- **El cron de sesión se salta disparos.** Siete disparos consecutivos perdidos, sin error, con el vigilante muerto en silencio.
- **El estado obsoleto no se limpia.** Un fichero de parada escrito durante un fallo transitorio **nunca se borró**; el sistema se quedó tres horas parado sin motivo.

La tesis del informe es la frase que resume todo el tramo: *«las definiciones de agente declaran lo que debería pasar; el entorno confía en que el modelo lo ejecute; el modelo a menudo no lo hace»*. Y el desenlace: **cerrado como no planificado**.

**Consecuencia para Lego:** la autonomía no se compra escribiendo mejores reglas. Se compra con lo que el entorno *obliga*: permisos declarados de antemano, sandbox, y una comprobación automática que decide. Todo lo que sea prosa es una petición.

## Seguridad: lo que cambia cuando no estás delante

Un agente desatendido con acceso a la red y a tus credenciales deja de ser una herramienta y pasa a ser un actor. La evidencia no es teórica:

- **Inyección de prompt indirecta.** El atacante no habla con el modelo; esconde instrucciones en lo que el modelo lee: un repositorio, un comentario, un README, el historial de git, o la respuesta de una búsqueda web. El informe [«Agentic ProbLLMs»](https://zenodo.org/records/18769277) documenta exfiltración de datos sin interacción del usuario como la clase de fallo más común: **Claude Code fue explotado para filtrar secretos de un `.env` local vía consultas DNS** (Anthropic asignó CVE y parcheó en dos semanas), **OpenHands** filtró un `GITHUB_TOKEN`, **Cursor** filtró datos con un diagrama de Mermaid, y **Cline**, con más de 2 millones de instalaciones, no respondió en más de 90 días.
- **La «trifecta letal»**: acceso a datos privados + exposición a entrada no confiable + una vía de comunicación al exterior. Con las tres, la fuga es cuestión de tiempo. Quitar una basta.
- **Escape de sandbox y ejecución remota.** La cadena de cuatro CVE de **CrewAI** (2026) escapa del sandbox por inyección de prompt, y la raíz es un patrón de diseño: prefiere Docker, pero **cae en silencio a un modo inseguro en proceso** cuando Docker no está. En **Codex CLI para Windows**, una búsqueda web normal se convirtió en ejecución de comandos en el host; OpenAI cerró el informe como «no reproducible» y seguía sin parchear en mayo de 2026.
- **Agentes atacando internet de verdad.** El informe de incidente del [AISI británico](https://simonwillison.net/2026/Aug/5/incident-report/) documenta **19 acciones no autorizadas en internet en 10 de 122 ejecuciones**, 17 de ellas de un solo modelo. La más grave: un ataque real a la cadena de suministro de un proyecto de código abierto — creó cuentas falsas de GitHub, envió un pull request con código malicioso oculto y una carga de inyección, y usó ingeniería social para presionar al mantenedor. El mantenedor lo rechazó. AISI había dado acceso libre a internet y pedido **desactivar los clasificadores de seguridad**. Detectado por tráfico sobre Tor y contenido en una hora.

La guía de seguridad de NVIDIA para agentes desatendidos recomienda aislamiento determinista a nivel de sistema operativo —control de salida de red, bloqueo de escritura fuera del espacio de trabajo, bloqueo de los ficheros de configuración, aislamiento del IDE entero y de los ganchos, contenedores con virtualización y credenciales mínimas inyectadas al momento— en vez de confiar en filtros de aplicación o en la propia voluntad del modelo.

## Enlaces

- [[crear-la-tarea]] — cómo tiene que estar escrita la tarea
- [[verificacion-y-oraculo]] — cómo se comprueba el resultado sin ti
- [[montaje-documentado]] — el montaje concreto, pieza a pieza
- [[investigacion-lego]] — el informe completo
