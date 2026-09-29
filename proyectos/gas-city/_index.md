# Índice: proyectos/gas-city

Gas City (sucesor de Gas Town, de Steve Yegge) como pieza central del montaje para desarrollar más rápido con agentes. Aquí se recogió también la investigación general de orquestación y del despiece del montaje, que antes vivía en una carpeta retirada el 2026-09-29.

## Gas City: el montaje

- [[gas-city-traje-a-medida]] — el montaje a medida: conocimiento de Yegge, Kim y Karpathy detrás, piezas (Claude Code, Gas City, Beads, Herdr, CI), qué tocar para ajustarlo y riesgos conocidos
- [[gas-city-instalacion-y-modelos]] — instalación en esta máquina (tarball, `GC_BEADS=file` vs dolt+bd), qué packs se instalan y cuáles se escogen, los cinco ejes de un agente, el reparto Opus/Sonnet/DeepSeek, y los tres frentes a apagar: contribución upstream (no existe), telemetría (viene activada) y bucles autónomos
- [[gas-city-alcalde]] — trabajar solo con el alcalde: los dos alcaldes (`gastown` reparte y fusiona solo; `gc.mayor` planifica contigo), el flujo paso a paso, quién te pregunta (solo él), qué te llega, los tres flujos que parten de un issue/PR de GitHub, y cómo operar con Claude y DeepSeek a la vez
- [[gas-city-con-2cerebro]] — cómo usarlo con 2cerebro para crear y desarrollar aplicaciones: el reparto (2cerebro = método, cada app = rig), el flujo paso a paso, y los cuatro puntos que hay que tocar para que conviva con el wiki
- [[gas-city-frente-a-la-fabrica]] — qué mecanismos de Gas City sirven y qué es marketing: el bucle `check` como primitiva de verificación, las puertas, los presupuestos, el tercer estado con la convención `75`, y las métricas que el propio proyecto publica de sí mismo

## La evidencia que sostiene el montaje

- [[orquestacion-opus-deepseek-informe]] — informe de investigación: qué se pregunta, candidatos descartados con motivo, y la conclusión sobre orquestar modelos de distinto coste por rol
- [[orquestacion-modelos-y-costes]] — qué modelo para qué papel, y a qué precio
- [[orquestacion-herramientas-y-patrones]] — las herramientas y los patrones de orquestación que existen y cuáles aguantan
- [[orquestacion-experiencia-comunidad]] — qué reporta de verdad quien lo ha usado, con sus cifras
- [[orquestacion-seguridad-ejecutor]] — el ejecutor como superficie de riesgo: qué puede tocar y qué no debería
- [[verificacion-sin-oraculo-informe]] — cómo se implementa una capa de verificación que el agente no puede tocar

