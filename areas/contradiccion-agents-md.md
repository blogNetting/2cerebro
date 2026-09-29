---
title: El efecto medido de AGENTS.md / CLAUDE.md
created: 2026-09-27
updated: 2026-09-29
tags: [agentes, contexto, evidencia, meta]
zona: tecnico
---

Qué dice de verdad el paper que se cita sobre si un fichero de contexto (`AGENTS.md` / `CLAUDE.md`) mejora la tarea. Nació como nota de contradicción —dos notas del wiki lo citaban con cifras incompatibles—; la contradicción se resolvió y esto queda como referencia.

## La conclusión del paper

ETH Zúrich, [arXiv:2602.11988](https://arxiv.org/abs/2602.11988). Leído el resumen en su página de arXiv:

> «aportar ficheros de contexto **no mejora de forma general las tasas de éxito** en las tareas», y sube el coste de inferencia «**by over 20% on average**». El patrón «se mantiene en distintos LLM, distintos agentes de código **y tanto con ficheros generados por LLM como con los comprometidos por desarrolladores**».

Con un matiz que explica el malentendido: **las instrucciones sí se siguen**, pero **las descripciones generales del repositorio no ayudan** — y son precisamente las que los proveedores recomiendan. Su conclusión: estos ficheros sirven para **especificar prácticas de codificación no estándar**, y cualquier intento de mejorar el rendimiento debería evaluarse antes de desplegarse.

## La cifra que no se sostenía

Circuló un «**+4 % de éxito si lo escribe un humano**» presentado como «beneficio de un AGENTS.md, medido, no asumido». **No aparece en el resumen del paper, y el resumen no publica ningún valor de significación.** Si la cifra existe en el cuerpo, sería de un subgrupo — nunca el titular del paper, que dice lo contrario. Presentarla como beneficio medido era el error. La lectura correcta es la de [[desarrollo-autonomo-con-agentes]]: los ficheros escritos por desarrolladores mejoran un **2,4 % de media, sin significación** (p=21 %).

## Por qué importa aquí

El esquema de este propio wiki se apoya en un fichero de contexto: `AGENTS.md` es la fuente única de verdad de este repositorio. Si la lectura correcta es que **no hay efecto fiable** sobre el éxito, mantenerlo con contenido real **sigue teniendo sentido como documentación** — es donde vive el esquema, y eso es un fin en sí mismo—, pero **no debería presentarse como una mejora medida de resultados**, porque no lo es.

## Enlaces

- [[desarrollo-autonomo-con-agentes]] — la evidencia medida sobre agentes, donde encaja este hallazgo
- [[metodo-de-investigacion]] — el método con el que se comprobó la cifra contra la fuente
