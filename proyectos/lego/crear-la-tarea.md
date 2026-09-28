---
title: Lego — cómo tiene que crearse la tarea
created: 2026-09-28
updated: 2026-09-28
tags: [lego, agentes, especificacion, tareas]
zona: tecnico
---

La tarea es un fichero markdown en el repo. Es el contrato entre quien dirige y el agente que ejecuta. Lo que sigue no es una opinión de estilo: son los dos resultados duros que mandan sobre el diseño, y lo que sobrevive a ellos.

## Resultado duro 1 — cuantas más instrucciones, peor se cumplen todas

> **P(todas las instrucciones cumplidas) ≈ p^n**, con n = número de instrucciones. [[metodo-y-alcance]]

El paper que yo citaba —«Curse of Instructions», Harada et al., **Universidad de Tokio**, [submitted to ICLR 2025 y **rechazado**, OpenReview R6q67CDBCH](https://openreview.net/forum?id=R6q67CDBCH)— construyó ManyIFEval con hasta 10 instrucciones verificables por programa. Sus resultados con 10 instrucciones: Claude 3.5 Sonnet 44%, GPT-4o 15%. **Se cita por completitud, no como respaldo de una ley.**

Lo importante es *por qué*: la ablación del propio paper repite la misma instrucción muchas veces para igualar la longitud, y **la degradación es menor que con instrucciones distintas**. Es decir, el daño lo hace el **número de instrucciones únicas**, no la longitud del texto. Un documento largo con pocas reglas distintas se cumple mejor que uno corto con muchas.

> ### ⚠️ CORREGIDO EL 2026-09-28 — el paper estaba RECHAZADO y la corroboración no lo respalda
>
> **El paper «Curse of Instructions» fue RECHAZADO por ICLR 2025.** El informe lo citaba como «ICLR 2025», que se lee como aceptado; la decisión, leída en la propia página de OpenReview, es **«Decision: Reject»** (22 de enero de 2025). La metarrevisión: *«dudas de que ManyIFEval sea una **extensión directa** de benchmarks existentes… instrucciones simples y verificables binariamente, **muy diferentes de las tareas reales**… y los revisores argumentaron que la “maldición” **era predecible** (con lo que también estoy de acuerdo)»*.
>
> **Y la corroboración revisada por pares dice otra cosa.** **IFScale** ([arXiv:2507.11538](https://arxiv.org/abs/2507.11538), **NeurIPS 2025**, Distyl AI), **20 modelos de siete proveedores**, hasta 500 instrucciones: los mejores modelos **mantienen un rendimiento casi perfecto hasta más de 150 instrucciones** — gemini-2.5-pro va del **100% a 10 instrucciones, al 84,8% a 250 y 68,9% a 500**; claude-3.7-sonnet, del **100% al 72,9% y 52,7%**. Hay **tres patrones distintos** (umbral, lineal y exponencial), no una única ley. **El degradado exponencial es el de los modelos pequeños**, no el general.
>
> **Los números concretos que yo daba quedan retirados:** decir «diez instrucciones dan ~35%» es falso para modelos de razonamiento. **Lo que sobrevive, atenuado:** la degradación existe y es grande en modelos pequeños; el **sesgo hacia las instrucciones tempranas** está confirmado; y a alta densidad los errores **pasan de modificar a omitir**.
>
> **Y el paper rechazado trae una mitigación que funciona**, que el informe no mencionaba: el autorrefinado lo sube —GPT-4o del **15% al 31%**, Claude 3.5 Sonnet del **44% al 58%**— y basta con decirle que no las está siguiendo. Ficha completa en [[quien-dice-que]].

**Consecuencia para la tarea, rebajada:** la degradación **depende del modelo**, y hay tres curvas distintas. Lo que queda en pie es más modesto y de sentido común: **cada regla añadida es una cosa más que comprobar**, y el valor de la tarea está en **acotar**, no en documentar.

## Resultado duro 2 — la calidad de la especificación no reduce defectos

> ### ⚠️ CORREGIDO EL 2026-09-28 — esta conclusión se retira
>
> **El estudio que la sostenía no mide lo que decía medir.** La versión final (ESEM 2026) cambió hasta el título a *«Specification Artifacts in Open-Source Pull Request Workflows Do Not Reduce Defects»* y su propio texto dice: «lo que medimos son artefactos de especificación — abrumadoramente **referencias a tickets e incidencias** — y no la salida de herramientas de SDD». El **97,5% del tratamiento es «este PR referencia un ticket»**. De los **119** repositorios, **cero** tienen un directorio de herramienta SDD. Los revisores obligaron a reformular, el resultado estrella se retiró del resumen, hubo **parada opcional** del muestreo, y el p-valor **cambia entre versiones del mismo dato** (0,016 frente a 0,164). **Y tiene erratas publicadas** (`CORRECTIONS.md`): un fallo de pandas eliminó 2.891 PRs de una comparación descriptiva, y la versión de Zenodo sigue sin corregir. Ficha completa en [[quien-dice-que]].
>
> **Y hay un contraataque limpio:** dos experimentos que sólo cambian el texto del enunciado, dejando la verificación idéntica, muestran que la especificación **sí** decide. Con GPT-4: **73,8% → 6,7%** de Pass@1 al volver contradictorio el enunciado (ICSE 2026). Y el código ejecutable pero incorrecto pasa del **24% al 89%**.
>
> **El argumento de fondo, que era falso:** un test **es** una especificación ejecutable. «Verificar» no es alternativa a «especificar»; es especificar en un lenguaje más estrecho. **La dicotomía estaba mal planteada.**
>
> Lo que sobrevive, y sigue siendo útil: **la tarea tiene que ser acotada y sin contradicciones.** Todo el detalle y las fuentes en [[contra-evidencia]].

> «None of the five hypotheses are supported under any operationalization.» — Hill, 2026 *(estudio retirado como sostén de esta conclusión; ver el aviso de arriba)*

El estudio de Brenn Hill ([SSRN, abril 2026, 119 repositorios](https://zenodo.org/records/19432099), texto extraído del PDF) analiza **88.052 pull requests en 119 repositorios**, puntúa **25.209 artefactos de especificación** en las mismas dimensiones de calidad que prescriben las herramientas de SDD, y compara los PRs con spec de cada desarrollador contra sus propios PRs sin spec. Doce comprobaciones de robustez más.

| Hipótesis | Resultado | p |
|---|---|---|
| Especificar reduce defectos | **No** — dentro del mismo autor, +1,4 puntos porcentuales de defectos | 0,003 (dirección contraria) |
| Especificar reduce retrabajo | **No** — +1,2 pp de retrabajo | 0,001 (dirección contraria) |
| Mejor calidad de spec reduce defectos | **No** | 0,164 |
| Mejor calidad de spec reduce retrabajo | **No** | 0,860 |
| Especificar limita el alcance del código de la IA | **No** | 0,997 |

La explicación que da el propio autor es **confusión por indicación**: la gente escribe specs para su trabajo más difícil, y el trabajo difícil produce más defectos. La spec no causa los defectos; la dificultad que motiva escribirla, sí. Al añadir controles de complejidad, el efecto pierde significación (p = 0,229).

Y la frase que hay que llevarse puesta:

> «A specification can score highly on every dimension and still specify the wrong behavior.»

**Consecuencia para la tarea:** la belleza, la extensión y la exhaustividad de la spec **no** compran calidad. Una spec puede estar perfectamente redactada y describir el comportamiento equivocado. Invertir en escribir mejor la spec es invertir en el sitio equivocado.

Tres límites que el propio autor declara, y que hay que respetar al citarlo: mide especificaciones **orgánicas**, no las generadas por herramientas de SDD; el algoritmo SZZ de rastreo de defectos tiene ruido (46–71% de mala atribución, que atenúa los efectos hacia cero); y es una muestra de código abierto, no de equipos comerciales.

## Lo que sobrevive a los dos resultados

> **Aviso de corrección.** La segunda mitad del argumento original («la prosa no compra calidad») se apoyaba en el estudio retirado. Lo que **sí** sobrevive, y es lo que importa para el diseño, es lo otro: **las instrucciones se degradan en bloque**, y **la ambigüedad y la contradicción sí destruyen el resultado** — medido por dos caminos independientes. Es decir: no es que escribir bien la spec no sirva; es que **lo que hay que eliminar es la ambigüedad y la contradicción, no añadir extensión**.

Si las instrucciones se degradan en bloque y la ambigüedad decide el resultado, lo que queda es: **pocos criterios, sin contradicciones, y todos comprobables por una máquina**. Esto es lo que converge entre fuentes independientes.

| Elemento | Para qué sirve | De dónde sale |
|---|---|---|
| **Objetivo en una frase** | Fija el resultado, no el procedimiento | [CodexGuide, seis elementos](https://raw.githubusercontent.com/freestylefly/CodexGuide/main/docs/start/07-task-design.md) |
| **Alcance: rutas permitidas y prohibidas** | Acota la escritura; evita que toque lo que no debe | [agent-engineering-kernel](https://github.com/alexxety/agent-engineering-kernel/commit/6204fd04727effb6a5556b2fcd2cd664b9a4fad9) |
| **Criterios de aceptación en forma fija** | Quitan la interpretación | Kiro, OpenSpec, [Mavin et al., 2009](#ears-o-el-formato-que-quita-la-interpretacion) |
| **El comando que lo comprueba** | Sin esto no hay autonomía, hay asistencia | [Spec Kit](https://github.com/github/spec-kit): *«Missing verification is not a successful fix»* |
| **Formato de retorno** | Lo que el agente devuelve al terminar | [Harbor](https://mintlify.wiki/harbor-framework/harbor/core-concepts/tasks/instruction) |

Addy Osmani, que recopila el análisis de GitHub sobre más de 2.500 ficheros de configuración de agentes, añade algo que encaja con el resultado 2: la mayoría de esos ficheros fallan «porque son demasiado vagos». Su propuesta de tres niveles de frontera —`✅ hazlo siempre`, `⚠️ pregunta antes`, `🚫 nunca`— es la forma más barata de escribir el alcance ([«How to write a good spec for AI agents», enero 2026](https://addyosmani.com/blog/good-spec/)).

### EARS, o el formato que quita la interpretación

EARS (*Easy Approach to Requirements Syntax*, Alistair Mavin, Rolls-Royce, [IEEE RE 2009](https://www.modernrequirements.com/blogs/ears-notation-the-practical-guide/)) constriñe cada requisito a una plantilla fija. Lo usa Kiro para los criterios de aceptación, y OpenSpec lo usa en su variante WHEN/THEN.

| Patrón | Plantilla | Cuándo |
|---|---|---|
| Ubicuo | El `<sistema>` DEBE `<respuesta>` | Invariantes siempre activos |
| Por evento | CUANDO `<disparador>` ENTONCES el `<sistema>` DEBE `<respuesta>` | Un estímulo provoca una respuesta |
| Por estado | MIENTRAS `<estado>` el `<sistema>` DEBE `<respuesta>` | Comportamiento durante un estado |
| Función opcional | DONDE `<función>` el `<sistema>` DEBE `<respuesta>` | Comportamiento condicionado a una función |
| Comportamiento indeseado | SI `<disparador>` ENTONCES el `<sistema>` DEBE `<respuesta>` | Errores y casos límite |

La regla de oro: **un criterio = una frase comprobable**, con **DEBE** para lo obligatorio, y con `<respuesta>` observable. Se mapea 1:1 con el test de aceptación, lo que hace la trazabilidad mecánica.

El formato real que genera OpenSpec ([README](https://github.com/Fission-AI/OpenSpec)) es exactamente eso, en `specs/` dentro de la carpeta del cambio:

```
## ADDED Requirements

### Requirement: Theme selection
The app SHALL let users switch between light and dark themes, defaulting to the system preference.

#### Scenario: User toggles dark mode
- **WHEN** the user clicks the theme toggle
- **THEN** the app switches to dark mode and persists the choice
```

## La forma más fuerte: la tarea *es* el test

Si el problema es que la prosa no se puede comprobar, la salida radical es quitar la prosa: **la tarea es un test que falla**, y el agente trabaja hasta que pasa. El verificador ya existe (el runner de tests), el criterio de «hecho» no es interpretable, y la spec no puede derivar del código porque *es* el código.

Herramientas reales que hacen esto:

- **[exspec](https://github.com/mnapoli/exspec)** — ejecuta especificaciones en texto plano con Given/When/Then **en un navegador real**, con IA, sin escribir código pegamento (ni definiciones de pasos, ni selectores). El agente lee la spec y navega la aplicación como un usuario. Cubre el caso web, que es donde no hay tests obvios.
- **[quinny](https://socket.dev/pypi/package/quinny/overview/0.2.1)** — compila criterios de aceptación de un fichero `.qn` a una suite de pytest o node:test. Declara ~12 segundos y menos de un céntimo por convertir una spec en una suite de ~140 líneas, y después ~0,24 s y coste cero por re-verificar cualquier implementación **sin IA en el bucle**.

Y la regla de oro del enfoque, que es también su punto débil ([Test-Driven Agent Development](https://github.com/agentpatterns-ai/website/blob/main/verification/tdd-agent-development.md)):

> Tú controlas la especificación; el agente pone el trabajo; la suite es el veredicto.

**El antipatrón, explícito en la misma fuente: dejar que el agente escriba los tests y la implementación.** Entonces «los tests no verifican nada; pasan su propio código, no el comportamiento que definiste». Para el caso sin oráculo, ver [[consumir-la-tarea]].

## La tarea tiene que ser autosuficiente: el agente no va a preguntar

Éste es el requisito que más se subestima, y tiene tres mediciones independientes detrás. Resumen aquí; el detalle en [[robustez-desatendida]].

> «La infrasepecificación no hace principalmente que los agentes fallen; hace que **adivinen**.» — [UnderSpecBench, arXiv:2607.02294](https://arxiv.org/abs/2607.02294)

- Sobre **2.208 variantes de prompt** y **69 familias de tareas**: entre **55,8% y 67,8%** de las ejecuciones **violan al menos un límite** cuando la instrucción está incompleta.
- Y darle al agente la **opción** de preguntar lo hunde: Gemini 3.1 Pro pasa del **84,7% al 5,3%**; Claude Opus 4.6 del **69,1% al 9,4%**; GPT-5.3-Codex del **67,3% al 2,0%** ([HiL-Bench](https://huggingface.co/papers/2604.09408), 300 tareas y 1.131 bloqueos). «Ningún modelo frontera recupera más que una fracción de su rendimiento con información completa cuando tiene que decidir si preguntar.»
- Y preguntar sin canal **simula éxito**: en [Archon #2257](https://github.com/coleam00/Archon/pull/2257), la pregunta se convertía en la salida del nodo, el nodo reportaba `completed`, y el flujo avanzaba **sin haber implementado nada** — tres veces.

Los bloqueos se reparten así: **42%** información ausente, **36%** requisitos ambiguos, **22%** instrucciones contradictorias. Esas tres cosas son exactamente lo que hay que cerrar **dentro del fichero de tarea**:

- **Información ausente** → todo lo que haga falta para decidir va escrito: rutas, convenciones, versiones, dónde están las cosas.
- **Requisitos ambiguos** → de ahí el formato fijo de los criterios (CUANDO/MIENTRAS/SI… ENTONCES… DEBE). No deja hueco a la interpretación.
- **Instrucciones contradictorias** → y esto no es teórico: el caso real que contaba un desarrollador en [[lo-que-dice-la-comunidad]] era una spec con **dos requisitos contradictorios en las secciones 4 y 7**. El agente estaba haciendo exactamente lo que se le dijo — le dijeron dos cosas distintas. Un repaso final buscando contradicciones internas es parte de escribir la tarea, no un lujo.

**Y la regla que se deduce:** ante una ambigüedad que no se pudo cerrar, no se le dice al agente «pregunta si dudas» —no funciona—, se le dice **«decide, y escribe la decisión en la sección de desviaciones del informe de retorno»**. Es el patrón que adoptó Archon tras reproducir el fallo tres veces, y encaja con el quinto campo de la tarea: el formato de retorno.

## Lo que la tarea NO debe llevar

- **Prosa larga.** El estudio de Hill no encontró relación entre la calidad de la spec y menos defectos; el examen de Spec Kit de Colin Eberhardt generó **2.577 líneas de markdown** para una funcionalidad de 689 líneas de código, y tardó diez veces más que su método normal ([Scott Logic, nov 2025](https://blog.scottlogic.com/2025/11/26/putting-spec-kit-through-its-paces-radical-idea-or-reinvented-waterfall.html)).
- **Reglas que el entorno no pueda obligar.** Éste es el hallazgo más importante de campo, y tiene nombre: un operador montó un sistema multiagente para trabajar toda la noche y documentó nueve fallos. Su tesis: *«las definiciones de agente declaran lo que debería pasar; el entorno confía en que el modelo lo ejecute; el modelo a menudo no lo hace»*. Ejemplo textual del informe: *«Opus 4.7 leyó esas reglas, las reconoció y las violó en la siguiente entrada del registro»* ([claude-code issue #53610](https://github.com/anthropics/claude-code/issues/53610), abierto en abril de 2026 y **cerrado como no planificado**). Una regla escrita en prosa es una petición, no una restricción. Lo que de verdad restringe son los permisos, el sandbox y los tests.
- **Instrucciones que no se puedan comprobar.** Si no hay un comando que lo demuestre, no es un criterio de aceptación; es un deseo.

## Granularidad: una tarea, una unidad revisable

No hay una cifra fiable de «cuántas subtareas». Un paper que daba una fórmula cerrada (DGI ≈ 0,85·√S, R² = 0,994) resultó estar **generado por IA** —autores «Tom Cat» y «Screwy Squirrel», laboratorio «tom-and-jerry-lab», contradicciones internas— y queda descartado (ver [[investigacion-lego]], sección de descartes). Lo que sí sostiene la evidencia:

- «Curse of Instructions»: si se puede partir, se parte; menos instrucciones simultáneas por tarea.
- «[Decomposition Buys Integrity, Not Yield](https://arxiv.org/abs/2609.17464)» (Rong He, arXiv 2609.17464, preprint): modela la descomposición como un árbol y **la profundidad no mejora el rendimiento** — «flat is optimal for yield and no arrangement of agents escapes the exponent». La profundidad se compra para proteger el contexto y abaratar, no para rendir mejor. Medido sobre 600 trazas de producción. *Cautela: autor único, preprint, y son trazas de investigación, no de código.*
- «[Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)» (Anthropic): empezar por lo simple y **añadir complejidad solo cuando mejore los resultados de forma demostrable**. Es documentación de fabricante sobre su propio producto, así que vale para CÓMO diseñar, no como prueba de adopción.

La regla práctica que sostiene todo esto: **la unidad más grande que el agente pueda terminar de forma fiable**. Ni micro-pasos (cada corte añade una transformación y un traspaso donde se pierde información) ni tareas que excedan lo que el modelo sostiene.

## Cuántos artefactos hacen falta

Los frameworks usan 3–4 ficheros por funcionalidad: Spec Kit (`constitution.md` + `spec.md` + `plan.md` + `tasks.md`, y añade `converge`), Kiro (`requirements.md` + `design.md` + `tasks.md`), OpenSpec (`proposal.md` + `specs/` + `design.md` + `tasks.md`).

Con el criterio de piezas mínimas, la reducción honesta es: **`constitution` y `plan` no son tarea, son contexto y diseño, y se pueden fundir en el fichero de reglas del repo y en la propia tarea.** La tarea es un fichero con el objetivo, los criterios de aceptación y el comando que los comprueba. El plan solo se separa cuando la tarea es lo bastante grande para necesitarlo.

El propio movimiento reconoce el exceso: la crítica de Birgitta Böckeler (Thoughtworks) citada en [abelcastro.dev](https://abelcastro.dev/blog/spec-driven-development-is-solving-the-wrong-problem) encontró «demasiados ficheros markdown verbosos que revisar, repetitivos entre sí y con el código existente», y que preferiría «revisar código antes que revisar todos esos ficheros markdown». Probar Kiro con un arreglo de bug pequeño produjo 4 historias de usuario con 16 criterios de aceptación: «un mazo para cascar una nuez».

## Enlaces

- [[metodo-y-alcance]] — criterio de admisión e hipótesis rivales
- [[consumir-la-tarea]] — qué hace el sistema con la tarea
- [[investigacion-lego]] — el informe completo y los descartes
