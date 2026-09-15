---
title: Portero automático — timbre único para dos unidades
created: 2026-09-14
updated: 2026-09-15
tags: [domotica, alquiler, portero-automatico]
zona: tecnico
---

Dos sistemas de timbre distintos avisan hoy a las dos unidades de [[apartamentos-calle-uruguay]] a la vez cuando debería sonar solo en una: el portero automático del edificio (llamada desde la calle) y el timbre de la puerta del piso (llamada desde el rellano). Son problemas separados, con cableado y encaje legal distintos — no confundir uno con otro.

## 1. Portero automático del edificio (placa de calle)

Solo hay **un timbre / una línea de portero automático** para las dos unidades (planta baja y planta alta). Son dos habitaciones/unidades de uso totalmente independiente en el día a día, pero comparten línea de portero: cualquier llamada desde la placa de la calle suena en ambas y molesta al inquilino que no espera visita.

Contexto físico: instalación de portero, elemento común hasta la entrada de la vivienda (art. 396 CC); el telefonillo interior es privativo y se puede tocar (art. 7.1 LPH) — ángulo legal ya cubierto en [[argumentario-frente-a-comunidad-y-vecinos]]. Falta la solución técnica para que la llamada distinga a qué unidad va dirigida.

### Contexto técnico del edificio

- Portero automático **analógico**, gama clásica Fermax de hace ~30 años. **Sin videoportero.**
- Edificio de 4 alturas: plantas 1ª-3ª con 4 viviendas cada una; planta 4ª son los dúplex, 6 unidades en total (incluye la 5ª planta, que es toda dúplex).
- El problema es solo en la unidad propia (4ºB): una única línea de portero para las dos "habitaciones" (apartamentos) independientes en que se ha dividido.

### Opciones barajadas

**a) Aceptar que suena en los dos — un aparato por unidad**
La más sencilla: en vez de compartir un único telefonillo, poner un aparato de portero por apartamento. No resuelve la molestia en sí (ante cualquier llamada sigue sonando en ambos), solo evita que compartan el mismo terminal físico.

**b) Interceptor que no timbra y abre directo**
Un dispositivo intermedio capta la señal de llamada del portal, no la deja pasar a los telefonillos interiores (activable/desactivable por domótica o botón), y abre la puerta del portal directamente. Una vez dentro, queda por definir cómo se avisa a la unidad correspondiente sin usar el telefonillo compartido.

**c) Interceptor con locución de enrutado (IVR)** — la elegida, desarrollada abajo.
Se intercepta la llamada del portero antes de que timbre dentro. Un mensaje de voz indica al visitante cómo elegir destino, y el dispositivo enruta la llamada solo al telefonillo del apartamento correspondiente.

### Cómo funciona la señal (Fermax analógico de ~30 años)

Sistema **Fermax 4+N** (gama Citymax/City, el ubicuo de los 90): 4 hilos comunes para todo el edificio + 1 hilo de llamada exclusivo por vivienda. En la caja llegan **5 hilos**. Tabla de terminales del estándar (común a los universales Fermax y a Tegui/Golmar/Auta):

| Term | Función |
|---|---|
| 1 | Abrepuertas |
| 2 | Micrófono |
| 3 | Común / masa |
| 4 | **Llamada** (hilo individual de la vivienda) |
| 6 | Altavoz |

**El sistema es de 12V en alterna.** La llamada llega como ~12V AC entre terminal 4 y 3. No hay direccionamiento digital ni teclado en la placa.

**Dato crítico: no hay ningún raíl de alimentación aprovechable en la caja.** Los 4 comunes son abrepuertas, micro, común y altavoz — líneas de señal (las de audio pueden llevar algo de bias del amplificador de la placa, pero nada de lo que alimentar electrónica). El telefonillo antiguo es pasivo. Los 12VAC viven en la fuente del portal, en zona común. Consecuencia: **hay que llevar 230V propios al dispositivo, con o sin radio** — no hay nada que "cosechar" del bus, ni para un MCU de 1 mA. Aprovechar la energía de la propia señal de llamada tampoco sirve: solo está viva mientras el visitante aprieta, y el dispositivo necesita hablar durante segundos después de que la suelte.

Esto tiene dos implicaciones:

- **Añadir un segundo pulsador en la placa de calle no es una opción.** El hilo individual ya asignado a esta vivienda es único; tender uno nuevo exige intervenir en la columna montante del edificio (elemento común, compartida con el resto de plantas) desde la propia placa. Eso es obra sobre instalación comunitaria, no algo que se haga sin la comunidad. Descartado.
- **La intervención se hace dentro de la vivienda**, en la caja donde llegan los 5 hilos, sobre el hilo individual/selectivo (el que es de uso exclusivo de esta vivienda) — sin tocar ni cargar los 4 hilos de bus común, que siguen sirviendo al resto de la columna. Mismo patrón que el shunt de ventilación en [[legalidad-division-y-alquiler-por-habitaciones]]: físicamente parte de instalación común, de uso exclusivo de la vivienda. Encaja con el art. 7.1 LPH — ángulo ya cubierto en [[argumentario-frente-a-comunidad-y-vecinos]].
- **La respuesta del visitante no puede detectarse en la placa** (no hay teclado ni lógica ahí, solo un contacto mecánico en el hilo individual). Se detecta dentro de la vivienda, sobre ese mismo hilo selectivo. Esquema elegido en la sección siguiente.

### Escenario c) en detalle: inyectar audio y esquema de pulsación

**En un 4+N no existe "estado de llamada".** La placa es un amplificador tonto con N pulsadores. Los terminales 2 (micro) y 6 (altavoz) son un **bus de audio permanentemente activo** para todo el edificio: descuelgas cuando quieras y hablas con el portal sin que nadie haya llamado. Lo único que hace el hilo 4 es sonar el zumbador de esa vivienda.

Consecuencia: **no hay que simular ningún descolgado ni negociar nada con la placa.** Para hablarle al visitante basta con **inyectar audio en el terminal 2** — el de micrófono, que es el que alimenta el altavoz de la placa (el 6 es la dirección contraria: micro de la placa hacia el telefonillo). El interceptor inyecta, y al terminar **desconecta su salida con un contacto de relé** para no cargar la línea mientras habla el inquilino.

Ventaja de timing: sin negociación previa, la locución arranca a los ~100 ms de la pulsación, lo que tarde el MCU en reaccionar.

Efecto colateral aprovechable: el terminal 6 da el micro de la placa de forma permanente, así que escuchar el portal está disponible sin cablear nada más.

**Esquema de respuesta del visitante: prompt secuencial, se pulsa al oír la opción.** Evolución en dos pasos, ambos por corrección del usuario (2026-09-14):

1. **Contar pulsaciones queda descartado.** Sobre 12V AC a 50 Hz, rectificar da 100 semiciclos por segundo: contar flancos no significa nada. Hay que **rectificar, filtrar y extraer la envolvente**, convertirlo en un nivel "llamada activa / no activa", y medir tiempo sobre ese nivel. Detección de presencia, no de eventos.
2. **Pero "silencio = opción A" también se cae**, porque si no hacer nada ya es una respuesta, el visitante que no se ha enterado no puede pedir que se repita.

Esquema resultante:

```
[bip]  "Planta baja... pulse ahora"    → ventana
       "Planta alta... pulse ahora"    → ventana
       (bucle 2 veces, ~14 s)
       nadie pulsa → SE CUELGA
```

**Si nadie responde, se cuelga — nunca suena en los dos.** No confundir con el failsafe de hardware: que el visitante no responda no es un fallo, es una interacción terminada sin selección, y sonar en ambos ahí es justo el resultado que este proyecto existe para evitar. Colgar hace que el coste lo pague quien no actuó (vuelve a pulsar); sonar en los dos se lo cobra a dos inquilinos que no hicieron nada, y encima ~20 s tarde y descorrelacionado, con el visitante probablemente ya ido.

Corolarios:

- **Bucle corto, no largo.** Como fallar ya no molesta a nadie, no hace falta paciencia: 2 pasadas y colgar. Reintentar no cuesta nada.
- **Filtro de spam gratis.** Críos pulsando botones, repartidores equivocados, el que llama a todos los timbres del portal: nadie navega un prompt que no le interesa, así que no llegan a los inquilinos.

- Sigue siendo detección de presencia dentro de una ventana — robusto frente a la señal sucia.
- No hacer nada significa "repítemelo", no una elección.
- La instrucción va en el audio en el instante de actuar: nada que memorizar.
- Primera ventana de decisión a ~1,5 s de pulsar; bucle completo ~7 s. Hoy el visitante ya se come varios segundos de silencio esperando a que descuelguen, así que no es regresión.
- **La ventana de cada opción va desde que empieza su frase hasta que empieza la siguiente**, no solo tras terminarla: quien pulsa nada más oír "planta baja" cuenta igual. Sin zonas muertas.

### Factibilidad paso a paso

| Paso | ¿Se puede? |
|---|---|
| Detectar la llamada entrante | **Sí, seguro.** Opto + umbral + duración mínima. |
| Retener el timbre de ambos telefonillos | **Sí, seguro.** Relé en serie en el camino de timbre. |
| Hablarle al visitante | **Sí, directo.** Inyección en term. 2. El bus de audio está siempre vivo, no hay nada que negociar. |
| Detectar actividad en la ventana | **Sí**, con envolvente sobre el 12VAC del term. 4. |
| Enrutar a un solo apartamento | **Sí, seguro.** Relé sobre el hilo de llamada. |
| Ceder a la llamada normal | **Sí, trivial.** No hay handover: el bus nunca se interrumpe. |

**No hay paso crítico.** Versiones anteriores de esta nota inventaron un "riesgo de handover" importando el concepto de estado de llamada de los sistemas digitales modernos. En 4+N no existe: el audio está siempre disponible para cualquier telefonillo descolgado, el interceptor solo tiene que dejar de inyectar y energizar el hilo de llamada del elegido.

**Failsafe: si el aparato falla, abre directamente.** Si se queda sin alimentación o se cuelga, la propia señal de llamada del portal acciona el abrepuertas (une 1 con 3) mientras el visitante pulsa, sin alimentación ni lógica. En ese estado no suena ningún telefonillo. Decisión del usuario (2026-09-15) frente a "suena en los dos". Viabilidad pendiente de medir la corriente que da el hilo 4 en [[portero-comprobaciones-en-sitio]]. Esto aplica solo a fallo del aparato — con el aparato vivo y el visitante sin responder, se cuelga (ver arriba).

Modos previstos (cambiables por radio): `AUTO` (lógica de enrutado), `BOTH` (passthrough), `SOLO_1` / `SOLO_2` (un inquilino fuera), `DND`.

### Hardware

Funciones que necesita el aparato, sin piezas: [[portero-bloques-funcionales]].

**Principio que manda sobre todo lo demás: aislamiento galvánico total.** Es un aparato alimentado de red conectado a una instalación comunitaria. Todo lo que cruce esa frontera va aislado — opto en entradas, contactos de relé en lo conmutado, transformador en el audio. Cero cobre compartido entre la placa y el bus del edificio. Seguro y defendible.

Ninguna pieza elegida todavía.

Dos compras baratas que ahorran semanas:

- **Fermax 2068 "Prolongador de Llamada Zumbador 4+N"**: producto comercial que hace exactamente el tap de la señal de llamada en este sistema. Comprarlo para abrirlo y ver cómo lo resuelve Fermax.
- **Telefonillo Citymax de segunda mano + trafo de 12VAC**: monta la instalación completa en la mesa. Se desarrolla sin tocar el portal ni depender de que alguien llame.

Fuentes: [Fermax Convencional 4+N](https://www.fermax.com/intl-es/productos/videoporteros/sistemas/convencional-4-n), [tabla de equivalencias de terminales](https://kabelson.es/blog/guia-instalacion-telefonillo-universal-fermax/), [esquema kit 6201 Citymax](https://www.tdtprofesional.com/blog/manual-de-instalacion-para-el-kit-6201-de-fermax/), [prolongador 2068](https://antelsat.es/prolongador-llamada-telefonillo-zumbador-4n-2068-fermax-3231.html).

### Tamaño y estética

- **Módulos de desarrollo**: no entran en la caja, ni de broma.
- **La válvula de escape**: el interceptor no tiene que estar en la caja del telefonillo, tiene que estar **en serie con los hilos**. Vale cualquier punto del recorrido — dentro de la carcasa de un telefonillo (más grande y ya aceptada visualmente), falso techo, o caja de enchufe contigua, que además resuelve el 230V a la vez.

Orden de trabajo: prototipo con módulos en sitio accesible y feo → resolver primero el handover → PCB solo cuando la lógica esté cerrada.


### Pendiente (portero de calle)

- Comprobaciones y mediciones en la instalación real antes de cerrar nada: [[portero-comprobaciones-en-sitio]].
- Ajustar nivel de inyección en el terminal 2 contra la placa real.
- Diseñar los dos circuitos de timbre interior independientes (relé por telefonillo sobre el hilo de llamada).
- Grabar la locución y afinar duración de frases y ventanas con gente real.

## 2. Timbre de la puerta del piso (rellano)

Timbre de la puerta de entrada de la vivienda, distinto del portero de calle. **Resuelto: privativo.** Análisis legal completo en [[argumentario-frente-a-comunidad-y-vecinos]]. Se pueden instalar dos timbres, uno por unidad, sin autorización de la comunidad (a lo sumo, dar cuenta previa si el cableado perfora pared común del rellano). Pendiente solo lo técnico: confirmar si el pulsador está en la hoja de la puerta o en la pared del rellano, y montar los dos circuitos.

## Enlaces

- [[apartamentos-calle-uruguay]]
- [[argumentario-frente-a-comunidad-y-vecinos]]
- [[_index]]
