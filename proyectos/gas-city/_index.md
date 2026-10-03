# Índice: proyectos/gas-city

Gas City (sucesor de Gas Town, de Steve Yegge) como pieza central del montaje para desarrollar más rápido con agentes. Aquí se recogió también la investigación general de orquestación y del despiece del montaje, que antes vivía en una carpeta retirada el 2026-09-29.

## Gas City: el montaje

- [[gas-city-traje-a-medida]] — el montaje a medida: conocimiento de Yegge, Kim y Karpathy detrás, piezas (Claude Code, Gas City, Beads, Herdr, CI), qué tocar para ajustarlo y riesgos conocidos
- [[gas-city-instalacion-y-modelos]] — instalación en esta máquina (tarball, `GC_BEADS=file` vs dolt+bd), qué packs se instalan y cuáles se escogen, los cinco ejes de un agente, el reparto Opus/Sonnet/DeepSeek, y los tres frentes a apagar: contribución upstream (no existe), telemetría (viene activada) y bucles autónomos
- [[gas-city-alcalde]] — trabajar solo con el alcalde: los dos alcaldes (`gastown` reparte y fusiona solo; `gc.mayor` planifica contigo), el flujo paso a paso, quién te pregunta (solo él), qué te llega, los tres flujos que parten de un issue/PR de GitHub, y cómo operar con Claude y DeepSeek a la vez
- [[gas-city-con-2cerebro]] — cómo usarlo con 2cerebro para crear y desarrollar aplicaciones: el reparto (2cerebro = método, cada app = rig), el flujo paso a paso, y los cuatro puntos que hay que tocar para que conviva con el wiki
- [[gas-city-frente-a-la-fabrica]] — qué mecanismos de Gas City sirven y qué es marketing: el bucle `check` como primitiva de verificación, las puertas, los presupuestos, el tercer estado con la convención `75`, y las métricas que el propio proyecto publica de sí mismo
- [[gas-city-acceso-externo]] — **probado en vivo, acceso confirmado**: el enlace fijo del panel desde fuera de la VM (`http://192.168.1.8:8372`), los dos cambios necesarios en `~/.gc/supervisor.toml` (`bind` + `allowed_hosts`, uno solo no basta), por qué queda en solo lectura, y que persiste solo
- [[gas-city-operacion-real]] — **probado en vivo, ciudad `NeTT-City` en marcha**: el reparto Opus/DeepSeek ya aplicado (`mayor` sin `upstream` = tu suscripción, `obrero-seek` con la clave de DeepSeek), el fallo real que impedía arrancar cualquier sesión (el diálogo de confianza de carpeta de Claude Code, no `tmux`), y los tres sitios distintos donde vive el nombre de una ciudad
- [[gas-city-tmux-scroll]] — **probado en vivo**: por qué la rueda del ratón no hacía scroll en la sesión de tmux (el modo ratón viene apagado por defecto, fuente `man tmux`), `mouse on` + `history-limit` aplicado en caliente y guardado en `~/.tmux.conf`, y las alternativas de comunidad descartadas (scrollback nativo del terminal, plugin `tmux-mighty-scroll`)

## La evidencia que sostiene el montaje

- [[orquestacion-opus-deepseek-informe]] — informe de investigación: qué se pregunta, candidatos descartados con motivo, y la conclusión sobre orquestar modelos de distinto coste por rol
- [[orquestacion-modelos-y-costes]] — qué modelo para qué papel, y a qué precio
- [[orquestacion-herramientas-y-patrones]] — las herramientas y los patrones de orquestación que existen y cuáles aguantan
- [[orquestacion-experiencia-comunidad]] — qué reporta de verdad quien lo ha usado, con sus cifras
- [[orquestacion-seguridad-ejecutor]] — el ejecutor como superficie de riesgo: qué puede tocar y qué no debería
- [[verificacion-sin-oraculo-informe]] — cómo se implementa una capa de verificación que el agente no puede tocar
- [[linear-y-jev-frente-a-gas-city]] — comparativa (2026-10-04): por qué Linear + JEV no sustituyen a Gas City (categorías distintas, JEV no escribe código), dónde sí encaja JEV como capa de decisión barata, y el precio real de Linear

