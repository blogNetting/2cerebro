---
title: Lego — los mecanismos que no estaban en el mapa
created: 2026-09-28
updated: 2026-09-28
tags: [lego, mecanismos, investigacion, huecos]
zona: tecnico
---

Tres barridos en paralelo —plataformas, fábricas, y un frente abierto— han producido **unos 60 mecanismos que no estaban en la lista original**. Consolidados aquí. Y hay un hallazgo estructural que vale más que la lista entera.

---

# 1 · Los tres huecos del mapa original

La lista que yo tenía cubre bien **el flujo de control** (bucle, cola, topes, cortacircuitos) y aceptablemente **el aislamiento**. Falla en **tres sitios**, y ahí está todo lo nuevo:

| Hueco | Qué falta |
|---|---|
| **1. Recuperación** | Tengo aislamiento —que **evita** el daño— y **nada que restaure** tras un daño que ya ocurrió. No hay un solo mecanismo de vuelta atrás |
| **2. Verificar al verificador** | Tengo cuatro formas de comprobar código. **Ninguna comprueba si mi comprobación está mintiendo** |
| **3. Detección de fallo de proceso** | El cortacircuitos actúa **sobre el bucle**. Nada actúa sobre **el estado interno del agente**, que es donde vive el fallo silencioso: el que da una respuesta plausible después de haberse torcido en el paso 4 |

**Y el hallazgo que lo resume, del frente abierto, textual:** *«el hueco está identificado por mucha gente y **resuelto con adopción por nadie**»*. De **25 candidatos** de control de bucle encontrados en el registro de MCP, **los 25 tienen entre 0 y 24 estrellas y ni una sola incidencia de terceros**. Prometen exactamente lo que buscábamos. **El código no tiene usuarios.**

---

# 2 · Recuperación — el bloque que faltaba entero

| Mecanismo | Qué resuelve | Estado |
|---|---|---|
| **Checkpoint acoplado de contexto y entorno** | Un error en el paso 12 contamina **el contexto y el disco**; rehacerlo no basta porque las acciones posteriores ya se ejecutaron. Se guardan puntos alineados de ambos y se vuelve atrás con la información de los intentos previos | [Paper, AgentRewind](http://arxiv.org/abs/2608.14380) |
| **Viaje en el tiempo y repetición desde un punto** | Reanudar tras una interrupción **sin empezar de cero**. Es lo que hace LangGraph, **42.421★** | [Documentación](https://docs.langchain.com/oss/python/langgraph/persistence) |
| **La seguridad del rollback** | ⚠️ **Un checkpoint restaurado con fidelidad puede reanudar una ejecución cuyos estados y efectos externos nunca coexistieron.** *«Un rollback correcto no implica una recuperación segura»* | [Paper, con tres ataques ejecutados](http://arxiv.org/abs/2608.29381) |
| **El registro de sesión FUERA del arnés** | Si el estado vive en el proceso, **el proceso es el punto único de fallo**. Anthropic lo sacó del contenedor y alegó −60 % en el tiempo hasta el primer token | Vendor, **sin medición independiente** |
| **Marca de agua para efectos no idempotentes** | Reintentar un pago o un correo **duplica el efecto**. Se persiste la marca antes de la operación y se borra al confirmar | Vendor |
| **Rotación de contexto dentro de la tarea, con traspaso estructurado** | La ventana se llena **a mitad** de tarea y hay que matar el bucle. Se escribe un traspaso, se limpia y se reanuda en ventana nueva | Comunidad |

---

# 3 · Verificar al verificador — el eje que no tenía

**Éste es el bloque más valioso del barrido**, porque ataca el fallo que ya había medido: un agente reescribió los resultados de los tests y sacó **500 de 500 sin resolver nada**.

| Mecanismo | Qué resuelve |
|---|---|
| **Mutación de los tests del agente, con umbral CERO** | El agente escribe un test que pasa sin probar nada. Se muta el código y **se exige que falle al menos un test**. Cita de un practicante: *«el arnés hizo fallar el test que el agente escribió, que pasaba contra el código arreglado **y también contra el código mutado**. El test parecía bien, **una revisión de código lo habría aprobado, y sólo la puerta lo cazó**»*. **Umbral: cero mutantes supervivientes, no un porcentaje** |
| **Casos canario: un acierto perfecto es prueba de trampa** | El agente hace trampa leyendo la clave de respuestas del entorno. Se diseñan casos donde **acertar todo es imposible**, así que un 100% delata. En PROCTOR llegaron a *«100% de aprobados ocultando un 68% de capacidad real»* |
| **Degradar el juez a consejero** | El bucle de auto-mejora lo puntúa un modelo, y ese modelo tiene la última palabra. Se le quita: **las comprobaciones de aceptación mandan sobre el juez**, y el juez no puede anularlas |
| **El calificador con ventana de contexto propia** | *«El calificador usa una **ventana de contexto separada** para no dejarse influir por las decisiones de implementación del agente principal»* |
| **Hiperpropiedades de dos ejecuciones** | Hay fallos —el agente que se guarda capacidad, la fuga entre inquilinos— que **no son decidibles con una sola traza**. Correr una segunda vez lleva al monitor del **50,4% al 90,0%** |
| **Invariantes como guiones que dicen CÓMO arreglarlo** | Un check que dice «rojo» no dice qué hacer. *«¿Invariante roto? El guion sale con código distinto de cero y **un texto que le dice al agente cómo arreglarlo**»* |
| **Mover la regla del prompt al compilador** | *«Una restricción en un tipo **es revisable y permanente**, mientras que una regla de prompt se degrada en cuanto la ventana de contexto pasa de largo»* |

---

# 4 · El oráculo que de verdad se usa — y es el mayor de toda la investigación

Esto merece sección propia, porque **es la pieza con más adopción real que ha aparecido en toda la investigación**, muy por encima de cualquier plataforma:

| Herramienta | Estrellas | Descargas al mes |
|---|---|---|
| [`ChromeDevTools/chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp) | **52.701★** | **7.704.955** |
| [`microsoft/playwright-mcp`](https://github.com/microsoft/playwright-mcp) | **37.659★** | **24.380.072** |

**Sirven para lo que el agente no puede comprobar solo:** que la interfaz funciona. La consola, la red, las trazas de error mapeadas al código fuente. **Para el caso web —que es Patrimonial— esto es el oráculo que faltaba**, y está construido, mantenido y usado a escala.

---

# 5 · Detección de fallo de proceso

| Mecanismo | Qué resuelve |
|---|---|
| **Autómata finita extraída de las trazas** | Un modelo de 7 a 43 estados que **predice el fallo desde una traza parcial** y para antes de terminar. AUROC hasta 0,94. Y un hallazgo lateral potente: *«la topología del comportamiento parece moldeada **más por el arnés de despliegue que por el modelo**»* |
| **Compromiso prematuro** | El agente se queda con una lectura temprana y dedica el resto a defenderla. **Es medible**: la similitud de estados ocultos en el paso 4 predice el comportamiento posterior. Con un límite declarado honestamente: *«no dice si tiene razón, dice **si ya se cerró**»* |
| **Estado de reparación explícito** | *«La atribución por sí sola es insuficiente»* — hay que **recordar qué falló y qué se probó**, no sólo contar intentos |
| **La regla escrita a partir del fallo repetido** | ⭐ El complemento exacto de «parar por patrón repetido»: *«cuando una comprobación falla dos veces igual, **se añade una regla a `guardrails.md`** para que la siguiente iteración no lo repita»*. **No parar: escribir la lección.** El propio Huntley: *«hay que pedirle a Ralph una cosa por vuelta»* |

---

# 6 · El override humano — la pieza que resuelve algo que nadie resolvía

**`directives.md`**: un fichero que **todos los agentes leen primero**.

> *«Puse una sola línea en `directives.md`: “X es el enfoque equivocado y hay que descartarlo”… **Esto funciona extremadamente bien**»*

**Resuelve corregir un rumbo malo sin matar el bucle** — que es justo lo que no tenía. En el mismo sistema: la especificación es **inmutable** y el plan **se reescribe entero** cada iteración con lo más importante arriba.

---

# 7 · Memoria — y una regla de sentido común

| Mecanismo | Qué resuelve |
|---|---|
| **Memoria en markdown, revisable y corregible por el humano** ⭐ | [`basicmachines-co/basic-memory`](https://github.com/basicmachines-co/basic-memory), **4.055★**. Con una disciplina explícita: *«las notas van primero al cuaderno diario, luego **promocionas las que valen**. **El agente no decide solo qué es importante recordar. Lo decides tú**»* |
| **El fallo que eso evita, con nombre** | Las *«herejías»* de Yegge: *«cosas falsas que se quedan y **influyen permanentemente en su comportamiento**»* |
| **Curar la memoria en LECTURA, no en escritura** | *«La mayoría de los diseños curan la memoria al escribir… **descartando información de forma irreversible**»*. Guardar las trazas crudas y sintetizar cuando ya se sabe la tarea gana **+16 puntos** |
| **Dar al curador ojos para ver el mundo** | El curador sólo ve trayectorias completadas, así que conserva los errores. Darle herramientas de sólo lectura para verificar: del **39% al 73%** de acierto, y el coste por tarea de **3,38 $ a 1,68 $** |
| **Memoria versionada, con olvido y borrado** | Cada cambio crea una versión; se puede olvidar y redactar. **El propio fabricante avisa del riesgo**: con escritura y entrada no confiable, **una inyección escribe memoria que las sesiones futuras leen como de confianza** |

---

# 8 · Prompt injection — la única mitigación con aval de comunidad

**El problema:** ya lo tienes medido — Claude Code filtró un `.env` por DNS, con CVE.

**La solución, y es la mejor avalada de todo el barrido:** el patrón **Dual-LLM / CaMeL** — un planificador privilegiado y **un modelo en cuarentena que no tiene acceso a herramientas**, así que lo peor que puede devolver es una cadena de texto.

El aval, de un comentario en Hacker News con 71 puntos:

> «Llevo **dos años y medio siguiendo la inyección de prompt** y ésta es **la primera mitigación propuesta que me parece genuinamente creíble**… **no se apoya en usar otros modelos para intentar detectar los ataques**» ([hilo](https://news.ycombinator.com/item?id=43733683))

Y un dato que lo refuerza: en un experimento con instrucciones inyectadas, **33 a 47 de 75 se ejecutaron con las defensas normales, frente a 3 de 75** con autorización por capacidades. La idea, en una frase: **«la autoridad no es una cadena de texto»** — nombrar un recurso no debería bastar para actuar sobre él.

---

# 9 · La lista de comprobación de diseño, que es lo más útil de todo

Del frente abierto, y es un checklist que se puede aplicar a mano. **Seis preguntas para cada acción delegada:**

1. ¿Sigue conectada a **autoridad**?
2. ¿A **evidencia**?
3. ¿A **interrupción**?
4. ¿A **juicio independiente**?
5. ¿A **recuperación**?
6. ¿A **impugnación**?

**Y la auditoría de 63 artefactos que lo acompaña es la que da la medida del campo:** la mediación de herramientas aparece en 40, las trazas en 37, pero **la colocación de puntos de control en 6, la independencia del validador en 4, la recuperación en 2 y la impugnación en 1.**

**Es decir: el campo entero construye mediación de herramientas y trazas, y casi nadie construye recuperación ni impugnación.** Exactamente los dos huecos del mapa.

---

# 10 · Coste, y el control que falta

| Mecanismo | Qué resuelve |
|---|---|
| **Presupuesto duro que PAUSA en vez de cortar** | Mi tope corta el bucle. Éste **pausa y se reanuda** cambiando el presupuesto |
| **Tolerancia a la latencia** | El nivel diferido de Google: **−50 % de precio**, objetivo del 95% en 24 horas, **y el 5% restante expira y falla**. Un eje que no tenía: **aceptar fracaso a cambio de precio** |
| **Curación de memoria medida** | Tres comportamientos de desperdicio afectan al **79–98%** de las tareas y hasta el **22,75% del coste**. Y el dato incómodo: **las habilidades escritas por un humano recortan hasta un 41,73%; las sintetizadas por el agente, la mitad** |
| **El coste de lo que la compresión tira** | Con la misma tasa de éxito, las llamadas de recuperación suben de **21,0 a 63,9** mientras la completitud apenas cambia. **Medir lo que pierdes al resumir**, que nadie mide |
| **Router de FORMA de colaboración, no de modelo** | La ventaja de la jerarquía sobre el agente único pasa de **+2,4 puntos en lo fácil a +21,1 en lo difícil**, con 10 veces el coste. Un selector por dificultad llega al **77,7% con el 40% del coste** |

---

# 11 · Lo que NO se pudo verificar, y es la mitad del valor

**Ninguna de las cifras de los fabricantes tiene medición independiente.** Anthropic mide su propia mejora de latencia; Microsoft, la suya. **En ninguna de las cuatro casas apareció una fuente de comunidad que las corrobore.**

**Y la adopción de todo el bloque de control de bucle es prácticamente nula.** El patrón es constante: descripciones que prometen **exactamente** lo que buscábamos, con **0 a 411 estrellas** y sin una sola incidencia de terceros. **25 de 25 verificados.**

**Lo más adoptado de todo, con diferencia, son las dos herramientas de oráculo de interfaz** de la sección 4: **52.701★ y 37.659★**, con millones de descargas al mes.

---

# 12 · El mejor caso contra todo esto

**Y es serio:** se puede argumentar que **casi nada de esto es invención, sino importación de sistemas distribuidos.** Los *fencing tokens*, los arriendos con prueba de muerte por bloqueo de fichero, el *outbox* transaccional, la entrega «al menos una vez», el compare-and-swap y las claves de idempotencia **existen desde hace décadas en bases de datos y colas de mensajes**. Lo nuevo es **aplicarlo a agentes**.

**El argumento es defendible en los bordes pero se rompe en los tres huecos:** recuperación, verificación del verificador y detección de fallo de proceso **no son refinamientos de nada que yo tuviera**. Son ejes distintos.

**Y lo que sigue sin existir, en ninguna fuente, es lo más importante:** **nadie publica la tasa de éxito.** Ni una plataforma, ni un bucle, ni un mecanismo. Todo lo que hay son mecanismos bien argumentados por sus autores y **sin validación independiente**.

## Enlaces

- [[plataformas-auditadas]] — las cinco plataformas, auditadas
- [[las-piezas]] — el mapa original
- [[etapas]] — el veredicto por etapas
- [[receta-completa]] — el montaje con las piezas que sí existen
