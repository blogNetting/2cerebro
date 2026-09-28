# Índice: proyectos/lego

Proyecto Lego: cómo crear software y webs de la forma más autónoma posible con el menor número de piezas. El foco es la **creación de la tarea** (el artefacto que escribe el humano con ayuda de una IA) y su **consumo** por un sistema de agentes.

Entrega: **informe cerrado el 2026-09-28**. El montaje se documenta pero no se ejecuta ni se instala nada.

- [[investigacion-lego]] — informe: introducción, considerado y descartado, análisis, recomendaciones, verificación, dónde se ha buscado y lo que no encaja
- [[metodo-y-alcance]] — qué se pregunta, qué queda fuera, criterio de admisión de candidatos, hipótesis rivales, mapa del terreno y condiciones de cierre declaradas antes de buscar
- [[crear-la-tarea]] — los dos resultados duros que mandan sobre el diseño (P(todas)≈p^n y la calidad de la spec que no reduce defectos) y los cinco campos que sobreviven: objetivo, alcance, criterios en EARS, comando de comprobación y formato de retorno
- [[consumir-la-tarea]] — el bucle mínimo (Ralph Wiggum, tres piezas), por qué contexto limpio, el disparador sin piezas nuevas, worktree frente a contenedor, y la evidencia de que la operación desatendida falla y por qué la seguridad deja de ser opcional
- [[verificacion-y-oraculo]] — el problema del oráculo, el techo medido de cada técnica, el agente que ve el oráculo y hace trampa, y lo que ningún oráculo automático puede decidir
- [[robustez-desatendida]] — los cuatro fallos documentados de operar sin nadie delante: reintentos, cuelgues, concurrencia y —el central— que los agentes **adivinan en vez de preguntar**, con la caída de rendimiento medida al darles la opción
- [[lo-que-dice-la-comunidad]] — el veredicto de los que lo usan, con las convergencias entre foro, academia y consultora
- [[autonomia-medida]] — cifras con denominador: cuánto resuelven de verdad, el descuento por contaminación, la caída geométrica con los pasos, y el hallazgo de que **el andamiaje pesa más que el modelo**
- [[gratis-y-local]] — la vía enteramente gratuita: qué es gratis (software y modelos), qué no (el hardware), la pila mínima, y la confirmación independiente de que el andamiaje vale 2,4 veces el modelo
- [[clausura-semantica]] — el marco que reencuadra el problema: por qué un compilador sabe cuándo está bien y un modelo no, y por qué la respuesta es «construye la verificación, después añade la LLM»
- [[quien-dice-que]] — **auditoría de fuentes**: para cada afirmación, quién la firma, quiénes son, qué intereses tienen, y si se pudo comprobar en primaria. Incluye el hallazgo de que el paper de la «maldición de las instrucciones» estaba **rechazado** y que la corroboración revisada por pares **no lo respalda**
- [[contra-evidencia]] — el ataque frontal a las cinco conclusiones: **dos se retiran, una se parte, dos no aguantan como estaban**. Incluye los tres errores del informe que hubo que corregir
- [[panorama-de-herramientas]] — el barrido completo: los 30+ frameworks, el formato `TASKS.md` y su convergencia con los cinco campos, GSD como el patrón Ralph industrializado, las cinco categorías que faltaban (protocolos, gestores de dependencias, rastreadores con grafo) y la medición que falta en la categoría
- [[casos-medidos]] — quién entrega software con agentes y con qué números: el único caso independiente (802 devs, 196.212 PRs, 2,09×), la autonomía real medida (menos del 5% de las PRs), la tasa de fusión por agente, y los casos de empresa que no valen como prueba
- [[limites-del-andamiaje]] — la mejor crítica a toda la categoría, de quien lo intentó y dio marcha atrás: la fábrica con las luces apagadas no funciona, y el motivo es que **la mantenibilidad no tiene oráculo rápido**
- [[piezas-y-coste]] — recuento de 2 a 7 piezas por opción, las tres piezas que nadie cuenta, y qué se rompe de verdad en cada herramienta
- [[montaje-documentado]] — el montaje reproducible con las tres piezas: ficheros, bucle, verificación, disparador, sandbox y reglas. **No ejecutado.**

## Frontera de alcance

Por instrucción explícita, quedan **fuera** de esta investigación las notas de `proyectos/astillero/` y todo lo que derive de ese trabajo. No se han leído ni se usan como punto de partida, comparación ni referencia. Esta investigación parte de cero.
