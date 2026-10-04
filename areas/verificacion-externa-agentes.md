---
title: Verificación externa en sistemas con agentes
created: 2026-09-27
updated: 2026-10-04
tags: [agentes, verificacion, evidencia, meta]
zona: tecnico
---

Nota de síntesis: el mismo principio aparece, con distintas palabras, en toda la evidencia sobre desarrollo autónomo con agentes. La verificación solo cuenta si la posee algo **distinto del agente y fuera de su alcance de escritura**.

## El principio

La mínima intervención humana es viable solo si la intervención que se conserva es la verificación, y esa verificación la posee algo externo que el agente no puede tocar. De ahí salen tres reglas duras: los tests y el evaluador viven fuera del alcance de escritura del agente; «he terminado» no es evidencia (la finalización se demuestra ejecutando algo que el agente no controla); y el sistema escala a una persona antes de fallar en silencio.

## Por qué: los modos de fallo medidos

- **Autocorregirse sin oráculo externo empeora el resultado.** «Large Language Models Cannot Self-Correct Reasoning Yet» (ICLR 2024).
- **El agente hace trampa contra el evaluador cuando puede alcanzarlo.** Reward hacking medido (METR, 30,4 %).
- **El agente satura sus propios tests.** La brecha entre pasar los tests escritos por el agente y los retenidos crece con el tamaño del código (SpecBench); «el juez no puede ser el acusado».
- **Pedir «haz TDD» no lo garantiza.** En el estudio preregistrado de Dan Luu, solo el **41,9 %** de las ejecuciones con instrucción de TDD mostró un test en rojo antes de implementar, y la condición con instrucción rindió *peor* en corrección que sin ella.
- **La revisión por LLM tiene techo:** ~**50-60 %** de detección, con el «techo de intención» como fallo más citado — el código cumple la letra, no lo pedido. Un revisor de familia de modelo distinta ayuda contra el «shared blind spot» (que escriba y revise la misma familia), pero no levanta ese techo.

## Cómo se materializa cuando el problema se ataca en serio

Formas concretas que la evidencia respalda, y que sirven de criterio para juzgar cualquier montaje:

- **Un gate técnico, no una instrucción:** un hook `PreToolUse` que bloquea la edición si no hay un test en rojo que la justifique, con defensa en profundidad (denegar también los comandos con los que el propio hook se puede burlar). Una instrucción en un prompt no es una barrera.
- **Los tests no son una puerta de CI aparte: los protege `CODEOWNERS`** — aprobar exige a una persona, no a otro proceso automático.
- **El paso está hecho cuando lo dice un script, no cuando lo dice el agente.** Es la formulación operativa del principio, y algunos orquestadores ya la implementan como primitiva de primera clase: el paso cierra cuando el script sale con 0 ([[gas-city-frente-a-la-fabrica]]).
- **Y una advertencia que acompaña a esa implementación:** tener el principio **no es** tener la barrera. Un orquestador puede formular la regla igual de bien y a la vez declarar, en su propia documentación, que sus comandos «son una característica, no un recinto» — la separación es por configuración, no forzada. El principio y la garantía son dos cosas distintas.

## Lo que sigue abierto

- **La etapa de medición no existe en la mayoría de montajes:** sin ella no se distingue «va bien» de «va rápido». Es el agujero que se repite.
- **El veredicto del revisor no se mide todavía, solo se asume:** no hay un conjunto de regresión que compruebe si el revisor sigue detectando lo que ya se confirmó como fallo real.

## Enlaces

- [[desarrollo-autonomo-con-agentes]] — la evidencia medida que sostiene el principio
- [[gas-city-frente-a-la-fabrica]] — un orquestador que formula el principio igual («el paso está hecho cuando lo dice tu script») y a la vez declara que sus comandos «son una característica, no un recinto»
- [[verificacion-sin-oraculo-informe]] — los mecanismos concretos (rulesets de push, CODEOWNERS, tests cifrados, recibos firmados) y qué está probado frente a propuesto
- [[verificacion-y-oraculo]] — el oráculo y su techo medido, desde el frente Lego
- [[clausura-semantica]] — por qué el canal de verificación no puede ser la propia generación
- [[metodo-de-investigacion]] — el método con el que se recoge esta evidencia
- [[linear-y-jev-frente-a-gas-city]] — un verificador barato y con probabilidad calibrada (JEV) como candidato a capa de decisión, y por qué no sustituye al orquestador
