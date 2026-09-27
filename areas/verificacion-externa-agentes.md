---
title: Verificación externa en sistemas con agentes
created: 2026-09-27
updated: 2026-09-27
tags: [agentes, verificacion, evidencia, meta]
zona: tecnico
---

Nota de síntesis: el mismo principio aparece, con distintas palabras, en toda la evidencia sobre desarrollo autónomo con agentes y en el diseño de [[astillero]]. La verificación solo cuenta si la posee algo **distinto del agente y fuera de su alcance de escritura**; el detalle y las fuentes están en las notas enlazadas.

## El principio

Formulado como regla de diseño en [[desarrollo-autonomo-con-agentes]] §4: la mínima intervención humana es viable solo si la intervención que se conserva es la verificación, y esa verificación la posee algo externo que el agente no puede tocar. De ahí salen tres reglas duras: los tests y el evaluador viven fuera del alcance de escritura del agente; «he terminado» no es evidencia (la finalización se demuestra ejecutando algo que el agente no controla); y el sistema escala a una persona antes de fallar en silencio.

## Por qué: los modos de fallo medidos

- **Autocorregirse sin oráculo externo empeora el resultado.** «Large Language Models Cannot Self-Correct Reasoning Yet» (ICLR 2024) — [[desarrollo-autonomo-con-agentes]] §2.
- **El agente hace trampa contra el evaluador cuando puede alcanzarlo.** Reward hacking medido (METR, 30,4 %) — misma nota, §2.
- **El agente satura sus propios tests.** La brecha entre pasar los tests escritos por el agente y los retenidos crece con el tamaño del código (SpecBench); «el juez no puede ser el acusado» — [[desarrollo-autonomo-con-agentes]] §3.quinquies.
- **Pedir «haz TDD» no lo garantiza.** En el estudio preregistrado de Dan Luu, solo el 41,9 % de las ejecuciones con instrucción de TDD mostró un test en rojo antes de implementar, y la condición con instrucción rindió *peor* en corrección que sin ella — [[flujo-agentes-evidencia-empirica]] §3.

## Cómo se materializa en Astillero

- **Un gate técnico, no una instrucción:** `tdd-guard`, hook `PreToolUse` que bloquea la edición si no hay un test en rojo que la justifique, con defensa en profundidad (denegar los comandos con los que el propio hook se puede burlar) — [[flujo-agentes-arquitectura]] §7.1 bis.
- **Los tests no son una puerta de CI aparte: los protege `CODEOWNERS`** (aprobar exige a una persona) — [[flujo-agentes-arquitectura]] §8.
- **Verificador de familia de modelo distinta:** Opus revisa código de DeepSeek para evitar el «shared blind spot» de que la misma familia escriba y revise; la garantía desaparece si el ejecutor pasa a un modelo Claude, y así queda anotado — [[flujo-agentes-arquitectura]] §1 y §11.
- **En el despiece por etapas, la verificación (etapa 7) es el agujero declarado:** «falta el oráculo que el agente no pueda tocar», y la autocomprobación del agente (etapa 6) se marca explícitamente como *no* un control — [[la-fabrica]].
- **Techo de la revisión por LLM:** ~50-60 % de detección, con el «techo de intención» como fallo más citado (el código cumple la letra, no lo pedido) — [[flujo-agentes-informe]] §9.3.

## Lo que sigue abierto

- **La etapa de medición (12) no existe** en [[astillero]]: sin ella no se distingue «va bien» de «va rápido» — [[la-fabrica]].
- **El veredicto del revisor no se mide todavía, solo se asume:** no hay todavía un conjunto de regresión que compruebe si el revisor sigue detectando lo que ya se confirmó como fallo real — [[flujo-agentes-arquitectura]] §7.2.

## Enlaces

- [[desarrollo-autonomo-con-agentes]] — la evidencia medida que sostiene el principio
- [[flujo-agentes-arquitectura]] — dónde se aplica hoy en el diseño
- [[flujo-agentes-evidencia-empirica]] — adherencia real a TDD y por qué el prompt no basta
- [[flujo-agentes-informe]] — techo medido de la revisión automática
- [[la-fabrica]] — el despiece donde la verificación es el agujero
- [[gas-city-frente-a-la-fabrica]] — un sistema que formula el principio igual («el paso está hecho cuando lo dice tu script») y a la vez declara que sus comandos «son una característica, no un recinto»: tener el principio no es tener la barrera
- [[astillero]] — proyecto
