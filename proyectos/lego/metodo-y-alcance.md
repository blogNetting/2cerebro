---
title: Lego — método y alcance
created: 2026-09-28
updated: 2026-09-28
tags: [lego, agentes, especificacion, investigacion]
zona: tecnico
---

Declaración escrita **antes** de buscar: qué se pregunta, qué se admite, qué hipótesis compiten y cuándo se cierra. Sin esto, el cierre lo decide la comodidad y no la evidencia.

## La pregunta

¿Cuál es el **conjunto mínimo de piezas** que da autonomía real para que una tarea escrita por un humano con ayuda de una IA sea consumida por un sistema de agentes hasta entregar software o una web?

Dos tramos, con el primero como foco:

1. **Creación de la tarea.** Cómo tiene que estar escrita la tarea para que un agente la ejecute sin volver a preguntar. No es "un buen prompt": es un contrato con criterios de aceptación verificables.
2. **Consumo de la tarea.** Qué arquitectura de agentes la lee, la ejecuta y la verifica, con el mínimo de componentes.

Autonomía y número de piezas van en contra: más autonomía se suele comprar con más piezas. Por eso el número de piezas es **restricción dura** y la autonomía es lo que se maximiza sujeto a ella.

## Frontera de alcance

Fuera: `proyectos/astillero/` y cualquier nota derivada de ese trabajo. No se ha leído, no se usa como punto de partida, comparación ni referencia. Instrucción explícita del usuario.

Fuera también: construir o instalar nada. El informe documenta el montaje completo (piezas, ficheros, comandos, conexiones) para que sea reproducible, pero no se ejecuta.

## Criterio de admisión

Un candidato (herramienta, formato, arquitectura o práctica) entra en la lista solo si cumple **todo** esto. Sale del problema, no de nombres que haya mencionado el usuario.

1. **Define o impone un artefacto de tarea consumible por un agente** — no un chat ni una sesión interactiva — o aporta evidencia sobre qué debe contener ese artefacto.
2. **El número de piezas es contable y pequeño**: sin servicio externo obligatorio más allá de git y un runtime de agente. Cada opción se reporta con su recuento de piezas.
3. **Tiene evidencia de campo**: uso real, *issues* o *discussions* con fallos y soluciones, informes de equipos que lo usan. Estrellas y descargas **no** cuentan como evidencia; solo como señal de existencia.
4. **Produce un resultado verificable** — comando, test o criterio de aceptación — o declara explícitamente cómo se verifica. Sin verificación automática no hay autonomía, hay asistencia.
5. **Es gratis de ejecutar**, o la parte de pago es separable y claramente mejor.
6. **Aplica a código y a web**, o el informe marca dónde no aplica.

## Hipótesis rivales (escritas antes de buscar)

**H1 — Git como cola, un agente, el test como verificador.** El mínimo es: un formato de fichero de tarea en markdown + un CLI de agente + git para cola, historial y aislamiento (ramas o *worktrees*) + los tests del propio repo como verificador. Sin orquestador, sin cola, sin base de datos. La autonomía viene de que la tarea sea ejecutable y verificable, no de la infraestructura.

**H2 — Framework de spec-driven development ya hecho.** El mercado ya tiene frameworks que imponen el formato de tarea y el bucle (GitHub spec-kit, Kiro, BMAD, OpenSpec…). Menos decisiones que tomar, pero añade una pieza (el framework) y sus opiniones, que hay que aceptar o romper.

**H3 — Orquestación dedicada.** La autonomía de verdad requiere infraestructura: cola de tareas, un contenedor o *worktree* por tarea, un agente supervisor, una máquina de estados (LangGraph, CrewAI, Agent SDK, orquestadores sobre tmux). Más piezas, más autonomía. Habría que demostrar que las piezas extra compran autonomía que H1 no da.

**H4 — La tarea es el test.** El formato de tarea más eficiente es **ejecutable**: la tarea *es* el test que falla o el comando de aceptación. Piezas mínimas porque el verificador ya existe (el runner de tests) y el criterio de "hecho" no es interpretable. Ataca directamente la pregunta de cómo debe escribirse la tarea.

## Mapa del terreno

Tipos de fuente, con la primaria marcada.

| Tipo | Qué es | Primaria para |
|---|---|---|
| **Del oficio** | Repos y docs oficiales de las herramientas: spec-kit, OpenCode, Claude Code, Agent SDK, Kiro | Cómo funciona cada pieza y qué formato impone |
| **Investigación** | Papers y evaluaciones: SWE-bench, METR, estudios de calidad de especificación y de fallo de agentes | Cifras de autonomía real y qué la limita |
| **Comunidad** | Hacker News, Reddit, *issues*/*discussions* de los repos, blogs de desarrolladores independientes, lobste.rs | Si funciona de verdad, qué falla y por qué |
| **Vendor** | Documentación de Anthropic, OpenAI, Google | Válida para CÓMO funciona una herramienta. **No** vale como prueba de adopción ni de que la práctica sea buena |
| **Fabricante de framework** | READMEs y landings de los frameworks SDD | Qué prometen. Se contrasta siempre contra la comunidad |

**Fuente primaria** para "cómo debe escribirse la tarea": los artefactos de especificación reales en repos reales y las plantillas de los frameworks, no los blogs que los describen.

## Presupuesto y condiciones de terminación (declaradas antes)

- **Rondas:** mínimo tres, y la segunda y la tercera las dicta lo que salga en la anterior. La consulta única no cuenta como ronda.
- **Fuentes:** mínimo tres tipos distintos de origen (oficio, investigación, comunidad). Cada afirmación que sostenga una conclusión: **dos fuentes de origen genuinamente independiente**, con cita textual.
- **Cierre por saturación:** una ronda completa en la que no aparece ninguna fuente ni dato nuevo que cumpla el criterio de admisión. Hay que saturar también los subcasos, no solo la pregunta central: formato de la tarea, arquitectura de consumo, verificación sin oráculo, y recuento de piezas y coste.
- **Cierre por coste no es cierre.** Si paro por prisa o por "ya tengo algo", hay que decirlo en el informe.
- Al cerrar, queda escrito: si paré por saturación o por coste, qué quedó sin comprobar, y qué parte es dato y qué parte es juicio propio.

## Enlaces

- `investigacion-lego` — el informe (pendiente de escribir; la nota aún no existe, por eso no se enlaza)
- [[_index]]
