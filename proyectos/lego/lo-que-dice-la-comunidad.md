---
title: Lego — lo que dice la comunidad
created: 2026-09-28
updated: 2026-09-28
tags: [lego, comunidad, reddit, evidencia]
zona: tecnico
---

El veredicto de la gente que lo usa, separado de la academia y del fabricante. Lo que importa aquí no es una cita suelta: es **dónde varias personas independientes, por caminos distintos, acaban diciendo lo mismo**.

## Convergencia 1 — el desarrollo dirigido por especificación no entrega en empresas reales

**Tres fuentes de naturaleza distinta llegan al mismo punto**, y eso es lo que lo hace sólido:

| Fuente | Tipo | Qué dice |
|---|---|---|
| [r/ExperiencedDevs, «Agentic, Spec-driven development flow…»](https://www.reddit.com/r/ExperiencedDevs/comments/1ox40ww/agentic_specdriven_development_flow_on/) | Comunidad de ingenieros senior | El autor pregunta si alguien ha visto un flujo SDD que funcione en un código establecido con muchos contribuidores. **Nadie responde con un caso de éxito.** |
| [Hill, 2026](https://zenodo.org/records/19432099) | Estudio académico, 88.052 PRs | Cinco hipótesis, cinco nulos. La calidad de la spec no reduce defectos ni retrabajo |
| [Thoughtworks Radar, abril 2026](https://www.thoughtworks.com/zh-cn/radar/languages-and-frameworks/github-spec-kit) | Consultora, experiencia de campo | Lo ponen en el radar **y** reportan «hinchazón de instrucciones» y «podredumbre del contexto» en los equipos que lo aplican |

Lo que dicen los desarrolladores, literal:

> «Nunca he visto ni oído a nadie, en ningún sitio, hacerlo con éxito. Toda la literatura, la formación y el marketing se parecen mucho a lo del "low-code" que se empujó en los años 2010.» — **latchkeylessons**, r/ExperiencedDevs

> «La afirmación de la que hablamos —flujos totalmente agénticos en los que todos los mantenedores trabajan así para todas las tareas, usando un puñado centralizado de ficheros markdown— **no es sostenible para nada que no sea lo más trivial**.» — **Unfair-Sleep-3022**, r/ExperiencedDevs

> «Es básicamente el método en cascada con un nombre más descriptivo.» — **Bricktop72**, arquitecto de sistemas

Ese último comentario importa porque **es la misma frase, por caminos independientes, que la de Colin Eberhardt** en su análisis de Spec Kit ([Scott Logic](https://blog.scottlogic.com/2025/11/26/putting-spec-kit-through-its-paces-radical-idea-or-reinvented-waterfall.html)): «Spec Kit te arrastra directo al pasado». Dos personas que no se conocen, mismo diagnóstico.

**El contraargumento honesto**, también en el hilo, y hay que darlo: el subreddit es un sitio hostil al tema. Un usuario lo dice con claridad — «cualquier hilo sobre agentes, specs o automatización queda ahogado entre defensividad y cinismo curtido mucho antes de que nadie hable de si la idea podría funcionar». Así que este hilo **no es una muestra neutral**. Lo que lo hace útil no es el voto mayoritario: es que las críticas concretas coinciden con las del estudio académico.

## Convergencia 2 — lo que sí funciona: tareas pequeñas, acotadas, y la spec como un test

Éste es el hallazgo más valioso de toda la ronda, porque **un desarrollador independiente llega exactamente al mismo diseño que la evidencia académica, por un camino completamente distinto**:

> «El problema no es realmente la IA. Es que el desarrollo dirigido por especificación te obliga a escribir una spec completa y sin ambigüedad — y la mayoría de los equipos nunca ha hecho eso. La IA sólo hace visible el hueco de inmediato en vez de dejarlo escondido dos semanas.
>
> Vi a un equipo pasarse tres días peleando con un agente que rompía funcionalidad adyacente. Culparon al modelo. Luego alguien leyó la spec y tenía **dos requisitos contradictorios en las secciones 4 y 7**. El agente estaba haciendo exactamente lo que se le dijo — es que le dijeron dos cosas distintas.
>
> Hay una versión de este flujo que funciona: **tareas pequeñas, muy acotadas, la especificación es básicamente un test unitario disfrazado.** Dale eso y va genial. Dale "constrúyeme un módulo de pagos según estos requisitos" y vas a pasar más tiempo revisando del que habrías pasado escribiendo.» — **rupayanc**, r/ExperiencedDevs

Eso es **el mismo resultado al que llegan, por separado**: la degradación del cumplimiento al acumular instrucciones (matizada — ver [[crear-la-tarea]]), la evidencia de que **la ambigüedad y la contradicción destruyen el resultado**, y el antipatrón documentado de dejar que el agente escriba los tests. Tres caminos, un destino: **la spec sirve cuando acota y se puede comprobar; estorba cuando es un documento de prosa.**

Y otro desarrollador, sin conocerse, dice lo mismo desde el lado del fracaso:

> «Trozos pequeños e iterativos es la forma de hacer esto. Arregla los problemas pequeños antes de que se conviertan en grandes. No te pongas en la posición de tener que revisar cambios grandes en 10 o 15 ficheros, que será el resultado de usar SDD.» — **Krom2040**, r/ExperiencedDevs

Y un tercero, que sí lo usa y le funciona, pero sólo bajo una condición explícita:

> «La calidad de la salida es directamente proporcional a la especificidad de la spec y de la documentación de estilo y arquitectura… Si lo lanzas suelto contra un código maduro que no tiene apenas guardarraíles ni instrucciones para el agente, vas a tener un mal rato.» — **SlapNuts707**, r/ExperiencedDevs

Eso **converge con el resultado 2 del informe** —la calidad de la spec no reduce defectos— de una forma matizada e importante: lo que ayuda no es la *calidad literaria* de la spec, es que **haya guardarraíles que el entorno pueda hacer cumplir**. Que es exactamente lo que dice [[crear-la-tarea]].

## Convergencia 3 — el intento de refutar a METR, refutado

Un usuario abrió un hilo titulado [«Hemos desmentido que los desarrolladores experimentados programen un 19% más lento con Cursor»](https://www.reddit.com/r/ExperiencedDevs/comments/1o2b2so/we_debunked_that_experienced_devs_code_19_slower/), contando que su empresa se pasó a SDD con Spec Kit y les fue mejor. La comunidad lo desmontó:

> «¿Cómo has desmentido un estudio y aún así no compartes nada de tu metodología, tamaño de muestra, demografía ni datos recogidos? Tu desmentido es "créeme, hermano". Y además es un anuncio.» — usuario anónimo

> «Vimos un estudio, así que lo desmentimos haciendo afirmaciones al azar y dando cosas por sentadas sin pruebas.» — **apnorton**, ingeniero de DevOps

> «"Mejoras significativas" sin una sola cifra, sin metodología, y con alguien vendiendo un producto. Mucho ánimo con Spec Kit, pero esto no es refutar nada.» — resumen del sentir del hilo

Y lo más revelador: el propio autor **admite en los comentarios que está promocionando Spec Kit**, y reconoce sobre su empresa —«una startup de unos 8 meses»— que no tiene «suficientes puntos de datos para diferenciar». Es decir: sin muestra, sin datos, sin grupo de control, y con interés comercial.

**Consecuencia para el informe:** la refutación de METR más visible que existe **no aporta ni un dato**, y la comunidad lo identificó como promoción. Eso no prueba que METR tenga razón —su propio seguimiento de 2026 reconoce que la medición es poco fiable—, pero sí significa que **nadie la ha refutado con evidencia**.

## Convergencia 4 — los casos de éxito existen, y ninguno dice «funciona solo»

Éste era el hueco que faltaba, y cerrarlo cambia el matiz sin cambiar la conclusión. **Sí hay gente a la que le funciona**, y lo cuentan. Pero **ninguno describe autonomía**: todos describen lo mismo, con sus palabras.

Del hilo [«¿Alguien usa desarrollo dirigido por especificación?»](https://www.reddit.com/r/ChatGPTCoding/comments/1otf3xc/does_anyone_use_specdriven_development/) (81 puntos, 94 comentarios):

> «Sí, y produce código listo para producción. A mí me funcionó mejor Spec Kit.» — 8 puntos

> «Lo uso y lo defiendo. **Estoy teniendo demasiado éxito usándolo** como para volver a no usarlo.» — 4 puntos

> «Sí, es mi enfoque desde mayo más o menos. **Tienes que revisar y ajustar, eso sí. No puedes generar una spec y copiar/pegar sin mirar nada.**» — 12 puntos

Y un flujo concreto que alguien describe paso a paso: Idea → Historias de usuario → Gherkin → Esquema → Tests funcionales → Código, con un directorio `prompts/` que lo semi-automatiza, y una condición explícita: «**reviso cada paso** para asegurarme de que no se ha descarrilado».

**Y la convergencia que más me llamó la atención**, en el hilo *más crítico* de todos —[«el desarrollo dirigido por especificación es una forma de masturbación técnica»](https://www.reddit.com/r/ChatGPTCoding/comments/1o6j1yr/specdriven_development_for_ai_is_a_form_of/) (71 puntos, 87 comentarios)—, donde la respuesta más votada describe **el mismo diseño al que llegó el bucle Ralph por otro camino**:

> «Parece que estás usando las specs mal. Cuando creas specs, creas **tareas** y partes el trabajo. Y **cada tarea debe completarse en una sesión nueva**. Al terminar cada tarea puedes hacer que la IA documente los detalles técnicos de lo hecho. Así **no hay contaminación de contexto**.» — 39 puntos

Eso es *sesión nueva por tarea*, que es exactamente lo que hace el bucle Ralph y lo que recomienda [[consumir-la-tarea]]. Dos personas que no se conocen, mismo diseño.

Y el comentario más honesto de ese hilo, que vale más que cualquier defensa:

> «Escribí mi propio marco y, después de pasar meses en él, me di cuenta de que **no funciona. Ninguno de estos otros marcos funciona tampoco.** Por eso nunca verás a ninguno haciendo demos de proyectos serios con ellos en YouTube. Habiendo dicho eso — lo que sí me funcionó es la forma de siempre: **paso horas escribiendo una spec yo mismo, como un ingeniero normal**, y entonces sí pude ejecutar esa spec con IA.» — 2 puntos

Y el remate de otro: «**la spec son tus guardarraíles, no se supone que sea apretar un botón**».

**Del informe de campo de dos meses** ([r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1te3ehy/tried_githubs_speckit_with_claude_code_for_2/)), lo bueno y lo malo, en sus palabras:

- **A favor:** «Agnóstico al agente. La misma spec funciona con Claude Code, Cursor, Codex, Gemini CLI, Copilot.»
- **En contra:** «Si cada bug menor exige retocar la spec y hacer una regeneración completa de código —en vez de dejar que un experto arregle el código directamente— **ralentiza drásticamente los tiempos de respuesta**.»
- **Lo que sí les funcionó:** «Un consejo que sí nos ha funcionado: **combinarlo con TDD estricto**.»

**El patrón, dicho sin adornos:** los que triunfan con esto no lo usan como una máquina autónoma. Lo usan como **un andamio para trabajar ellos**, con revisión en cada paso, tareas partidas y sesión nueva cada una — y con tests. Es decir, hacen lo que dice [[crear-la-tarea]], y obtienen lo que la evidencia permite esperar.

**Y ésta es la corrección que importa:** en [[investigacion-lego]] decía que nadie presenta un caso de éxito a escala en un código establecido. **Sigue siendo cierto para la escala y para el código establecido** — los casos de éxito que aparecen son proyectos propios, no empresas grandes — pero **no es cierto que no haya casos de éxito**. Los hay, son condicionales, y describen exactamente el diseño que la evidencia sostiene.

## Lo que la comunidad tiene en común, dijeron lo que dijeron

- **Nadie presenta un caso de éxito de SDD a escala empresarial**, en ninguno de los hilos, pese a pedirse explícitamente. Los que hay son proyectos propios.
- **Los que dicen que funciona describen siempre lo mismo**: tareas pequeñas, muy acotadas, **sesión nueva por tarea**, revisión en cada paso, y la spec convertida en test.
- **Los que dicen que no funciona describen siempre lo mismo**: specs grandes, requisitos contradictorios, revisar 10 o 15 ficheros, contaminación de contexto.
- **La acusación de promoción aparece sola**, sin que nadie la organice, en varios hilos.
- Un comentario de **MindCrusader** resume la postura intermedia, y coincide con la del informe: «Creo flujos que van al problema uno a uno y exigen revisiones frecuentes cuando se cierra un hito. Funciona, pero **sólo porque detecto los problemas pronto**.»

## Dónde se ha buscado

**Cuatro subreddits peinados**, por la API JSON de Reddit (la búsqueda por interfaz devolvía resultados recientes en vez de los mejores):

- **r/ExperiencedDevs** — [«Agentic, Spec-driven development flow…»](https://www.reddit.com/r/ExperiencedDevs/comments/1ox40ww/agentic_specdriven_development_flow_on/) y [«Spec Driven Development and other shitty stuff»](https://www.reddit.com/r/ExperiencedDevs/comments/1reiro1/spec_driven_development_and_other_shitty_stuff/), leídos completos.
- **r/ChatGPTCoding** — [«Does anyone use spec-driven development?»](https://www.reddit.com/r/ChatGPTCoding/comments/1otf3xc/does_anyone_use_specdriven_development/) (81 puntos, 94 comentarios) y [«…es una forma de masturbación técnica»](https://www.reddit.com/r/ChatGPTCoding/comments/1o6j1yr/specdriven_development_for_ai_is_a_form_of/) (71 puntos, 87 comentarios).
- **r/ClaudeAI** — hilos de [SDD frente a «Plan Research Implement»](https://www.reddit.com/r/ClaudeAI/comments/1pkvque/spec_driven_development_sdd_vs_plan_research/) y [Claude Code con SDD](https://www.reddit.com/r/ClaudeAI/comments/1m379oo/claude_code_specdriven_developement/); el resto de resultados eran sobre calidad del modelo, no sobre el método.
- **r/LocalLLaMA** — el informe de campo de [dos meses con Spec Kit](https://www.reddit.com/r/LocalLLaMA/comments/1te3ehy/tried_githubs_speckit_with_claude_code_for_2/).

**Otros:** [r/ExperiencedDevs, intento de refutar a METR](https://www.reddit.com/r/ExperiencedDevs/comments/1o2b2so/we_debunked_that_experienced_devs_code_19_slower/) · hilos de Hacker News citados en [[consumir-la-tarea]] y [[piezas-y-coste]] · Redlib (frontal de Reddit) devolvió **429 saturado**, y Reddit directo funcionó.

**Lo que no aportó nada:** los hilos de r/ClaudeAI sobre SDD son mayoritariamente de anuncio de herramientas propias, sin experiencia de uso; y la búsqueda por interfaz de Reddit devuelve resultados recientes ignorando los parámetros de ordenación — hubo que ir a la API JSON.

## Enlaces

- [[investigacion-lego]] — el informe, con la convergencia entre comunidad y academia
- [[crear-la-tarea]] — el diseño que sale de esta convergencia
