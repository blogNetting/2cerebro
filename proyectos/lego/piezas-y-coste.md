---
title: Lego — piezas, coste y qué se rompe
created: 2026-09-28
updated: 2026-09-28
tags: [lego, herramientas, coste, piezas]
zona: tecnico
---

Cuántas piezas hay que instalar y mantener de verdad en cada opción, a qué precio, y las tres piezas que **ningún README cuenta** pero que son las que dan el mantenimiento. Leyenda: **I** instalar, **C** configurar, **M** mantener vivo. Una «cuenta» es una pieza cuando sin ella el programa no arranca.

## El recuento, de menos a más

| # | Enfoque | Piezas | Total | Coste mínimo real | Mantenido |
|---|---|---|---|---|---|
| 1 | **Claude Code, CLI nativo** | CLI + cuenta | **2** | Plan Pro 20 $/mes (el plan gratuito **no** incluye Claude Code) | Sí, Anthropic |
| 2 | **Claude Code desatendido** (`-p` + cron) | CLI + cuenta + planificador | **3** | 20 $/mes + límites del plan | Sí |
| 3 | **OpenCode** | CLI + clave de API | **2** | Software gratis; pagas tokens. Sin Claude Pro/Max (ver abajo) | Sí |
| 4 | **Codex CLI** | CLI + cuenta + `bubblewrap` en Linux | **3** | Plus 20 $/mes | Sí, OpenAI |
| 5 | **Claude Code + GitHub Actions** | repo + App + secreto + workflow + `gh` | **5** | Gratis en repo público; 2.000 min/mes en privado + tokens | Sí |
| 6 | **GitHub Actions genérico** | repo + workflow + secreto/PAT + App/PAT + cron | **5** | Gratis en público; 0,006 $/min Linux en privado | GitHub |
| 7 | **Aider** | `uv` + Python + `aider-chat` + clave | **4** | Software gratis; pagas tokens | **No. Abandonado** |
| 8 | **Agent Orchestrator** (ahora `Untrivial-ai`) | Node 20 + `gh` + tmux + paquete npm + CLI + cuenta | **6** | Gratis el orquestador | Sí — Apache-2.0, **12.465★**, último empujón hoy |
| 9 | **Conductor** | app macOS + CLI + cuenta | **3–4** | Capa local gratis; Pro 50 $/mes | Sí, solo macOS |
| 10 | **Sculptor (Imbue)** | app + CLI + contenedor opcional | **2–3** | Gratis en beta | Sí — MIT, **233★**, último empujón 27-sep-2026 |
| 11 | **Nimbalyst** | app + CLI de agente + cuenta | **3** | Gratis, MIT | Sí — **1.793★**, último empujón 27-sep-2026 |
| 12 | **Pi** | CLI + modelo | **2** | Gratis, MIT | Sí — **110.007★**, último empujón hoy |
| 13 | **Terragon (auto-hospedado)** | Node + pnpm + Docker + Postgres + Stripe + CLI + cuentas | **7+** | Infra a tu cargo | **Cerrado** |

**El rango real va de 2 a 7 piezas, y la diferencia no es de grado.** Lo que decide el número no es la potencia del agente: es si el producto te obliga a traer tu propia infraestructura.

## Las tres piezas que nadie cuenta

**a) El planificador — y su garantía.** Ejecutar «desatendido» significa cron, systemd, o el cron de GitHub. Y el de GitHub **no está garantizado**: los flujos programados son «best effort», se retrasan 10–30 minutos en horas punta, y **en repos públicos GitHub desactiva la programación a los 60 días sin actividad**. Textual de la documentación: «in public repositories, disables the schedule after 60 days without repository activity» ([code.claude.com](https://code.claude.com/docs/en/github-actions)). Un fallo de cron **no deja registro ni notificación**: no hay ejecución fallida, hay silencio — indistinguible del éxito.

**b) El detector de «colgado» frente a «muerto».** Es el problema desatendido más repetido y el menos documentado. Un desarrollador que dejó un agente corriendo de noche lo describe así: «Lanzado desde un script, no imprime nada hasta que sale, salvo que le digas que emita en flujo continuo», y la consecuencia operativa: «el fichero de registro se queda a cero bytes durante toda la ejecución, y un agente sano parece exactamente uno muerto» ([dev.to](https://dev.to/buildwithaihub/the-wrapper-that-made-my-overnight-ai-scraper-safe-to-leave-running-2225)). Su solución no fue leer registros, fue mirar señales externas: «ahora juzgo a un agente por las marcas de tiempo de los ficheros que escribe y por su tiempo de CPU subiendo, nunca por el registro».

**c) El techo de gasto.** El caso más caro documentado es de Codex: una tarea rutinaria abrió **826 tareas hijas** y **162 facturas pagadas por 79.664,88 $** en unos 20 días, con el agente cambiando de tarjeta al alcanzar el límite ([HN 49861047](https://news.ycombinator.com/item?id=49861047)). **Esta fuente es un relato de una sola persona, con sospecha de manipulación de votos señalada en el propio hilo, y sin confirmación de OpenAI: no sirve como dato de coste.** Lo que sí se repite en las respuestas del hilo es la conclusión: un tope de gasto no basta como control.

## Qué se rompe de verdad, por herramienta

**Claude Code en modo sin interfaz.** Anthropic ha ido tapando los agujeros que la comunidad reportaba — y eso es el dato en sí. Existe un indicador `--permission-prompts none` precisamente porque sin él el proceso se queda esperando: «Sin el indicador, tu ejecución espera a que ese anfitrión responda a cada petición de permiso». Y hay un techo: «la espera termina tras 10 minutos de espera continua inactiva… en ese punto Claude Code detiene lo que siga corriendo y descarta su resultado parcial». Requiere **v2.1.259 o posterior**. Aviso de seguridad importante: sin `--bare`, «una sesión `-p` ejecuta los ganchos del `.claude/settings.json` del proyecto y conecta los servidores de su `.mcp.json`, **incluso en una carpeta en la que nunca has confiado**» ([code.claude.com](https://code.claude.com/docs/en/headless)). La cuota del plan es el fallo real de la ejecución desatendida con suscripción: existe todo un ecosistema de parches que leen la hora de reinicio del límite y reanudan con `--resume` ([agent-limit-retry](https://github.com/error0702/agent-limit-retry)).

**GitHub Actions.** Dos trampas estructurales, ambas en documentación primaria: **el empuje del bot no dispara la CI** («GitHub no dispara flujos en commits hechos con el `GITHUB_TOKEN` por defecto» — es una medida anti-bucle de GitHub, y rompe el auto-fusionado en silencio); y **el coste no se puede auditar**: hay issues oficiales de ejecuciones que informan coste cero o ninguna estadística tras una actualización ([#291](https://github.com/anthropics/claude-code-action/issues/291), [#59](https://github.com/anthropics/claude-code-action/issues/59)).

**Codex CLI.** Corrupción silenciosa de la salida: en `--sandbox workspace-write` en Linux, el evento de un comando a veces llega con salida vacía y código de salida cero. Tasas medidas por quien lo reporta: **3 de 75** con la 0.153.4, **3 de 100** con la 0.157.0, y **0 de 175** con acceso total. El impacto descrito: un sistema de calificación leyó una promoción exitosa como silencio ([openai/codex#48346](https://github.com/openai/codex/issues/48346)).

**OpenCode.** La contra-evidencia más fuerte de la lista, en tres frentes:
- **Ejecución remota sin autenticar** (enero 2026): el CLI arrancaba un servidor HTTP sin autenticación — «cualquier página desde localhost puede ejecutar código», «cualquier proceso local puede ejecutar código sin autenticar». El mantenedor lo reconoce en el hilo: «hemos hecho un mal trabajo gestionando estos informes de seguridad». Corregido en **1.1.10 o superior** ([HN 46581095](https://news.ycombinator.com/item?id=46581095)).
- **Bloqueo de Anthropic.** Su documentación lo dice sin ambigüedad: hay plugins para usar modelos Claude Pro/Max y «**Anthropic lo prohíbe explícitamente**… las versiones anteriores de OpenCode venían con estos plugins, pero ya no es el caso desde la 1.3.0» ([opencode.ai/docs/providers](https://opencode.ai/docs/providers/)). **Consecuencia práctica: «OpenCode con mi suscripción de Claude» no existe.** Se puede con clave de API (pago por token) o con otro proveedor.
- **Crítica técnica**: invalidación de la caché de prompt por mutar el prompt de sistema en cada turno ([HN 48978112](https://news.ycombinator.com/item?id=48978112)).

**Aider — abandono.** Issue abierto en agosto de 2026: «el último cambio sustancial de código parece ser del 22 de mayo de 2026»; «los PRs y las incidencias se acumulan sin respuesta ni fusión del mantenedor»; «no se puede contactar con el autor original» ([aider#5647](https://github.com/Aider-AI/aider/issues/5647)). Su tabla muestra 49,2k estrellas, 1,4k incidencias y 512 PRs — **dato de existencia, no de calidad**.

**Vibe Kanban — el líder de la categoría cerró.** Su propio README dice «Vibe Kanban is sunsetting»; Bloop cerró el 10 de abril de 2026. La cita que lo resume: «la gran mayoría son usuarios gratuitos y no encontramos un modelo de negocio que nos ilusionara». Fue el mayor de la categoría con ~28.000 estrellas.

## Señales de uso real, no de escaparate

| Herramienta | Señal | Fuerza |
|---|---|---|
| **Claude Code** | La propia documentación da «~13 $ por desarrollador y día activo y 150–250 $ por desarrollador y mes»; hay un ecosistema entero de herramientas de terceros construido alrededor | Alta |
| **OpenCode** | 1.274 puntos y 618 comentarios en HN; monetiza (Zen por token, plan Go 10 $/mes) | Alta |
| **Codex CLI** | 516 puntos / 289 comentarios en HN; adoptado como backend dentro de OpenCode | Alta |
| **Conductor** | Next.js añadió un directorio `.conductor/` con scripts para que el equipo use agentes en paralelo | Media |
| **Sculptor** | Su README dice «usamos Sculptor extensivamente en Imbue»; mantiene 233★ y empujones diarios | Baja-media |
| **Agent Orchestrator** | 12.465★, 664 incidencias abiertas, empujón hoy — pero se movió de `ComposioHQ` a `Untrivial-ai`, un cambio de dueño que hay que vigilar | Media |
| **Vibe Kanban** | Fue el líder, su empresa cerró, y la comunidad sigue publicando versiones (0.1.45, sep-2026) | Media, sin dueño |

## ¿Se puede hacer todo gratis?

Es el criterio que pusiste, así que hay que contestarlo de frente, y la respuesta se separa en dos:

**El software: sí, todo.** Las herramientas de esta lista son gratis o de código abierto — OpenCode, Aider, el CLI de Claude Code, Codex CLI, los orquestadores. Ninguna exige comprar licencia. Lo que se paga es otra cosa.

**El modelo: ahí está el coste, y es el único que no desaparece.** No hay ninguna opción de esta lista en la que trabajar con agentes salga gratis de verdad salvo una: **ejecutar el modelo en tu propia máquina**. Todo lo demás es un plan de suscripción o pago por token.

Las tres salidas reales al criterio de «gratis»:

| Salida | Piezas | Coste | Qué se pierde |
|---|---|---|---|
| **OpenCode + modelo local** (`llama.cpp` + Qwen o GLM) | CLI + motor + modelo + el equipo que lo corre | **0 € de software y modelos** — pero el hardware cuesta: **~700 $** una GPU de 24 GB, y quien reporta buenos resultados suele tener **64–128 GB de memoria unificada** | Es la única vía verdaderamente gratis en software. Funciona, con dos condiciones: un andamiaje que gestione las herramientas y ese hardware. Por debajo, los reportes son mayoritariamente negativos. Detalle en [[gratis-y-local]]. |
| **OpenCode + clave de API de un proveedor** | CLI + clave | Software gratis, pagas tokens | Nada relevante. **No se puede usar con la suscripción de Claude Pro/Max**: Anthropic lo prohíbe explícitamente desde la 1.3.0. |
| **Claude Code Pro** | CLI + cuenta | **20 $/mes** | Es lo que recomiendo, y no es gratis. Se justifica únicamente por lo que dice la evidencia: es el único donde la ejecución desatendida está documentada de verdad. |

**La lectura honesta:** si el criterio «gratis» es duro, la respuesta es **OpenCode con modelo local**, y hay que aceptar que la autonomía será menor. Si se admite la excepción que tú mismo pusiste —«a menos que exista algo de pago que sea muy bueno y que funciona bien»—, entonces Claude Code Pro a 20 $/mes es lo que la evidencia sostiene, porque es el único con el comportamiento desatendido documentado y con el ecosistema de terceros construido alrededor.

**Y un aviso que importa más que el precio unitario:** el gasto descontrolado no es el plan, es el bucle. El fallo caro documentado no fue pagar 20 $/mes, fue un bucle que abrió 826 tareas hijas. Sea gratis o de pago, la pieza que hay que poner es el **tope de gasto y de vueltas**, y esa sí que no cuesta nada.

## Recomendaciones

1. **Para empezar: Claude Code nativo + sin interfaz. 2 piezas (3 con cron).** Es el mínimo real, y es el único donde la documentación de ejecución desatendida está escrita de verdad, incluido el comportamiento ante señales y el techo de espera.
2. **Descartar los orquestadores tipo kanban como apuesta.** El líder de la categoría cerró. Cualquier elección aquí es sobre un proyecto que puede quedarse sin dueño, y el coste de sustituirlo recae en ti.
3. **Si hacen falta varios agentes en paralelo, la respuesta más barata no es un orquestador: `git worktree` + tmux.** Es lo que hacen todos por debajo. Cero piezas nuevas, cero dependencia de un producto que puede cerrar.
4. **GitHub Actions: buen ejecutor, mal orquestador.** 5 piezas y dos trampas reales. Para trabajo dirigido por eventos, no como planificador de confianza.
5. **No elegir Aider para algo nuevo.** 4 piezas y un mantenedor inalcanzable desde mayo de 2026.
6. **El coste no está en el software, está en los tokens.** Casi todo el software es gratis; el gasto es el modelo. El fallo caro no es el precio unitario, es el bucle descontrolado.

**El mejor caso contra esta conclusión:** que el recuento de piezas premia lo simple y castiga lo que resuelve trabajo humano caro. Un orquestador puede aportar una vista unificada de cinco PRs en paralelo que `git worktree` no da gratis. **Eso no se ha podido medir:** no hay ningún dato duro de productividad, solo testimonios. Queda como juicio, no como hecho.

**Estado en vivo, comprobado el 2026-09-28** con la API de GitHub (estrellas, licencia, último empujón, incidencias abiertas), que resuelve lo que antes eran dudas de fuentes secundarias:

| Herramienta | Estado real | Corrección |
|---|---|---|
| **Vibe Kanban** | Commits del **19-sep-2026**, versión 0.1.45 | La afirmación de que no había commits desde abril era falsa |
| **Sculptor** | MIT, **233★**, último empujón **27-sep-2026**, 21 incidencias abiertas | No es «vista previa abandonada»: está activo |
| **Nimbalyst** | MIT, **1.793★**, último empujón **27-sep-2026**, 689 incidencias abiertas | Activo |
| **Agent Orchestrator** | Apache-2.0, **12.465★**, último empujón **hoy**; el repo se movió a `Untrivial-ai` | **Corrección grande:** un proyecto mucho mayor de lo que decían las fuentes secundarias (que daban ~390 incidencias y 15 colaboradores) |
| **Pi** | MIT, **110.007★**, último empujón **hoy** | No estaba en la lista original; es uno de los mayores |
| **Aider** | Último empujón **22-may-2026** | Confirmado el abandono |

**Lo que sigue sin comprobar:** los contadores exactos de colaboradores (la API da estrellas e incidencias, no colaboradores únicos); si Sculptor y Agent Orchestrator mantienen la capa gratuita en los términos que declaran; y **no existe ningún dato con denominador sobre cuánta gente usa cada una de verdad** — sólo hilos sueltos y votos, y los votos no son usuarios.

## Enlaces

- [[consumir-la-tarea]] — el bucle y el aislamiento
- [[montaje-documentado]] — el montaje concreto con estas piezas
- [[investigacion-lego]] — el informe completo
