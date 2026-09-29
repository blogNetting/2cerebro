---
title: Gas City — el traje a medida para desarrollar más rápido
created: 2026-09-29
updated: 2026-09-29
tags: [gas-city, gas-town, yegge, karpathy, agentes, montaje]
zona: tecnico
---

> **El punto 4 de §5 (comprobar la contribución automática) quedó resuelto el 2026-09-29.** Gas City **no** arrastra el mecanismo del issue [#3649](https://github.com/gastownhall/gastown/issues/3649): no hay ninguna fórmula de release en el pack `core` ni en el catálogo oficial. La función existe, pero como pack `contributing` opt-in, cuyo propósito declarado es que contribuyas tú. El detalle verificado, en [[gas-city-instalacion-y-modelos]] §5.1.

Qué montaje encaja con el objetivo del usuario ([[objetivo]]: lo que funcione hoy y le quite trabajo aunque participe, cuanto más cubra mejor) partiendo del conocimiento de Yegge, Kim y Karpathy, y por qué la pieza central es Gas City y no Gas Town.

## 1. Introducción

Pregunta: cuál es el «traje a medida» —o lo más parecido— y qué hay que tocar para ajustarlo. Criterio: el conocimiento que hay detrás de las herramientas (quién las diseña y con qué principios), no el recuento de comentarios. Se buscó el 2026-09-28/29.

## 2. Considerado y descartado

| Qué | Motivo |
|---|---|
| **Gas Town como pieza central** | Su propia organización lo presenta como el predecesor: *«A legacy name you may encounter Gas Town is the predecessor software-factory project that inspired Gas City. Gas City recast that machinery as configurable primitives»* ([gascity.com](https://gascity.com/guide/gas-city-gas-town-gasworks-whos-who/)). Sigue valiendo su conocimiento (§3), no como producto nuevo. Sus problemas, en [[las-piezas]] §1.2 |
| **Las 76 plataformas del catálogo** | Ver [[plataformas-veredicto]]. Ninguna publica ahorro medido |
| **Evaluar por opiniones de foro** | Corrección del usuario: lo que pesa es el conocimiento de quien lo diseña. Las opiniones se usan solo como aviso de fallos concretos |

## 3. El conocimiento que hay detrás

| Principio | Quién | Cita verificada |
|---|---|---|
| **Trabajo en trozos pequeños** | Kim y Yegge, libro *Vibe Coding* | *«the smaller the steps, the better chance AI has to succeed»* ([IT Revolution](https://itrevolution.com/articles/the-vibe-coding-loop/)) |
| **Revisar hasta tener base de confianza** | Kim y Yegge | *«until you have established a basis for trusting it, you need to review it»* (misma fuente) |
| **El agente es desechable; el estado vive fuera** (NDI) | Yegge, Gas Town | Resumen de terceros: si un agente muere a mitad, uno nuevo lee el estado persistente y termina ([Daniel Vaughan](https://codex.danielvaughan.com/2026/04/08/gas-town-multi-agent-factory/)) — **sin cita primaria verificada** |
| **Autonomía parcial, regulable** («autonomy slider», IA «on a leash») | Karpathy, *Software is changing (again)* | Recogido en [Latent Space](https://www.latent.space/p/s3) — **sin cita textual verificada** |
| **Terminado cuando lo dice un script, no el agente** | Gas City | *«when your script says so, not when the agent says so»* ([docs Gas City](https://docs.gascity.com/guides/understanding-formulas)) |

## 4. El montaje

| Pieza | Qué hace | Estado |
|---|---|---|
| **Claude Code** | Escribe, prueba, corrige | Maduro |
| **Gas City** | Fábrica por piezas: *«orchestration-builder SDK»*, configurada en `city.toml`, runtimes intercambiables (entre ellos `herdr`) ([repo](https://github.com/gastownhall/gascity)). Corre sobre Beads. Bucle `[steps.check]`: un paso cierra solo si tu script sale con 0 ([[gas-city-frente-a-la-fabrica]] §3.1) | MIT, 1.300★. **Ningún usuario ajeno encontrado contando uso real** |
| **Beads** | Tareas y estado fuera del agente, con dependencias | Dependencia de Gas City. Quejas de conflictos de fusión ([HN](https://news.ycombinator.com/item?id=46458936)) |
| **Herdr** (runtime) | Ver y controlar agentes en marcha | Mayor uso real del barrido ([[plataformas-uso-real]]) |
| **CI con tests** | Verificación objetiva; es lo que llama el `check` | Maduro |

## 5. Qué tocar para ajustarlo

1. **Autonomía baja al empezar**: la fusión la haces tú hasta tener confianza; se sube tarea a tarea.
2. **2-3 agentes, no 20**: Yegge gasta *«thousands of dollars a month»* en API ([Maggie Appleton](https://maggieappleton.com/gastown)). La cuota es el límite real.
3. **Cada paso con `check` que llame a tus tests**: es lo que convierte «el agente dice que terminó» en «está terminado».
4. ~~**Comprobar antes de arrancar si Gas City arrastra la contribución automática a su repo** que tiene Gas Town (*«no opt-in»*, [#3649](https://github.com/gastownhall/gastown/issues/3649)).~~ **Resuelto el 2026-09-29: no la arrastra.** Verificado sobre el repo y el catálogo de packs — ver [[gas-city-instalacion-y-modelos]] §5.1. Lo que sí hay que apagar es otra cosa: la telemetría de producto, que viene activada por defecto (misma nota, §5.2).

## 6. Riesgos conocidos

Documentados en [[gas-city-frente-a-la-fabrica]] con su issue: el registro de auditoría se destruye en varios flujos (#6523, #5846, #6444, #6222) y la orquestación añade ~25 % de sobrecoste de tiempo (#3924). No invalidan el montaje; significan que el registro de lo que hicieron los agentes no es fiable como prueba.

## 7. Verificación

- Comprobado de forma mecánica (texto presente en la página, 2026-09-29): las citas de IT Revolution, Maggie Appleton, gascity.com, el README de Gas City, la guía de fórmulas y el issue #3649.
- **No encontrado en la fuente, retirado:** que Gas City esté «production-ready» según sus creadores y que recomienden «empezar por Beads». Salieron de un resumen automático, no del texto.
- Sin verificar: NDI y el «autonomy slider» en fuente primaria (solo resúmenes de terceros); la web de Yegge sobre Gas Town dio 403.

## 8. Dónde se ha buscado

Hacker News (API Algolia: hilos de Gas Town de 403, 354, 253, 219 y 113 puntos, comentarios con uso real), Medium de Yegge (403), yegge.ai, gascity.com, docs.gascity.com, repos `gastownhall/gastown` y `gastownhall/gascity`, IT Revolution, Maggie Appleton, Latent Space.

## Enlaces

- [[_index]] — índice de esta carpeta
- [[objetivo]] — el objetivo del usuario, que manda
- [[gas-city-instalacion-y-modelos]] — cómo instalarlo en esta máquina, el reparto Opus/Sonnet/DeepSeek, y los tres frentes a apagar
- [[gas-city-con-2cerebro]] — cómo usarlo junto con el wiki para crear y desarrollar aplicaciones
- [[las-piezas]] — Gas Town y Beads, con el aviso de apagar la contribución automática
- [[plataformas-veredicto]] · [[plataformas-uso-real]] — el barrido de plataformas
- [[gas-city-frente-a-la-fabrica]] — qué mecanismos de Gas City sirven y sus fallos, con issues
- [[gas-city-alcalde]] — cómo es trabajar hablando solo con el alcalde, quién te pregunta y qué te llega
