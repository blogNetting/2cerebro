---
title: Contradicción — el efecto medido de AGENTS.md / CLAUDE.md
created: 2026-09-27
updated: 2026-09-27
tags: [agentes, contradiccion, contexto, meta]
zona: tecnico
---

Dos notas de este wiki citan **el mismo paper** (ETH Zúrich, [arXiv:2602.11988](https://arxiv.org/abs/2602.11988)) con cifras y conclusión incompatibles sobre si un fichero de contexto (`AGENTS.md` / `CLAUDE.md`) mejora la tarea.

## RESUELTA (2026-09-27) — la lectura correcta es la de [[desarrollo-autonomo-con-agentes]]

Leído el resumen del paper en su página de arXiv, la conclusión del propio trabajo es la que recoge esa nota:

> «aportar ficheros de contexto **no mejora de forma general las tasas de éxito** en las tareas», y sube el coste de inferencia «**by over 20% on average**». El patrón «se mantiene en distintos LLM, distintos agentes de código **y tanto con ficheros generados por LLM como con los comprometidos por desarrolladores**».

Y añade el matiz que explica el malentendido: **las instrucciones sí se siguen**, pero **las descripciones generales del repositorio no ayudan** — son precisamente las que los proveedores recomiendan. Su conclusión: estos ficheros sirven para **especificar prácticas de codificación no estándar**, y cualquier intento de mejorar el rendimiento debería evaluarse antes de desplegarse.

**Sobre el «+4 %»:** no aparece en el resumen, y el resumen **no publica ningún valor de significación**. Si la cifra existe en el cuerpo, sería de un subgrupo — nunca el titular del paper, que dice lo contrario. Presentarla como «beneficio medido» era el error. [[flujo-agentes-informe]] §9.5 queda pendiente de corregir en su propia nota (no la toco desde aquí para no pisar trabajo de otra sesión).

## Las dos afirmaciones

- **[[flujo-agentes-informe]] §9.5**, como «*Beneficio de un AGENTS.md, medido, no asumido*» ✔︎:
  > «un paper de ETH Zurich mide **+4% de éxito si lo escribe un humano**, **-2/3% si lo genera un LLM**, +20% de coste de tokens en ambos casos».
- **[[desarrollo-autonomo-con-agentes]] §3.quinquies**, como «*el hallazgo que contradice una práctica extendidísima*»:
  > «los ficheros de contexto tipo `AGENTS.md` / `CLAUDE.md` **no mejoran de forma fiable el éxito de la tarea y suben el coste más de un 20 %** (ETH Zúrich + LogicStar, arXiv:2602.11988). Los escritos por desarrolladores **mejoran un 2,4 % de media sin significación (p=21 %)**».

## Por qué no cuadran

Las cifras no coinciden (**+4 %** frente a **+2,4 %**) y, más de fondo, la conclusión se invierte: una lo presenta como beneficio medido y de efecto claro, la otra como efecto **sin significación estadística** que invalida la práctica. Podrían ser dos métricas distintas del mismo paper (dos conjuntos de tareas o dos medidas), pero **tal como están escritas son incompatibles** y una de las dos está marcada como verificada en fuente primaria (✔︎). No se corrige ninguna hasta leer el paper.

## Por qué importa aquí

El esquema del propio wiki y el de Astillero se apoyan en ficheros de contexto: `AGENTS.md` es la fuente única de verdad de este repositorio, y la plantilla de Astillero genera `AGENTS.md` **y** un `CLAUDE.md` que lo importa en todo proyecto nuevo ([[decisiones]], 2026-09-26). Si la lectura correcta es la de [[desarrollo-autonomo-con-agentes]] — sin efecto fiable — la decisión de construirlos con contenido real sigue teniendo sentido como documentación, pero no debería presentarse como mejora medida de éxito. Las dos notas siguen en el wiki, con la discrepancia visible.

## Enlaces

- [[flujo-agentes-informe]] — nota implicada (§9.5)
- [[desarrollo-autonomo-con-agentes]] — nota implicada (§3.quinquies)
- [[astillero]] — el diseño cuya plantilla genera esos ficheros
