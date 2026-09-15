---
title: Portero automático — bloques funcionales
created: 2026-09-15
updated: 2026-09-15
tags: [portero-automatico, domotica, arquitectura]
zona: tecnico
---

Qué funciones necesita el interceptor de [[portero-automatico-timbre-unico]], sin elegir piezas. Los valores concretos salen de [[portero-comprobaciones-en-sitio]].

Requisito transversal: todo lo que toca los hilos del portero va aislado galvánicamente del resto del aparato.

## Bloques

1. **Detector de llamada.** Vigila el hilo 4 por el lado del portal y dice en todo momento "hay señal / no hay señal". Sirve para la primera pulsación y para cada ventana del menú.
2. **Conmutador del hilo 4, uno por telefonillo.** Cada telefonillo puede estar conectado al portal, aislado o conectado al generador de llamada. Implica partir el hilo 4 en dos ramas. Sin alimentación vuelve solo a reposo, y en reposo el hilo 4 del portal va al bloque 10.
3. **Generador de llamada.** Cuando el control ya sabe a qué telefonillo va la visita, genera una señal de llamada compatible con el telefonillo y la manda solo al elegido. Suena de forma automática durante un tiempo de llamada normal, ajustable, sin depender de que el visitante siga pulsando. Nivel según manual: 8–12 Vpp; forma y frecuencia se copian de la medida real del hilo 4.
4. **Almacén y reproductor de locuciones.** Guarda las frases grabadas y reproduce la que el control indique, arrancando al instante y cortando en seco.
5. **Adaptador de salida de audio.** Lleva el sonido al nivel e impedancia que espera el terminal 2 (nivel de micrófono).
6. **Desconexión de la inyección.** Separa la salida de audio del terminal 2 cuando no está hablando, para no cargar la línea durante la conversación.
7. **Control.** Detecta la llamada, bloquea los dos telefonillos, lanza las frases, abre cada ventana, lee el detector, decide, hace sonar el elegido con el generador, y cuelga si nadie pulsa. Gestiona los modos `AUTO` / `BOTH` / `SOLO_1` / `SOLO_2` / `DND`.
8. **Control remoto.** Cambiar de modo y ajustar tiempos sin desmontar nada de la pared.
9. **Alimentación** desde 230 V, aislada.
10. **Apertura en fallo.** Si el aparato se queda sin alimentación o se cuelga, la propia señal de llamada del portal acciona el abrepuertas (une 1 con 3) mientras el visitante pulsa, sin alimentación ni lógica. Un vigilante lleva el aparato a ese estado si el control se cuelga. En ese estado no suena ningún telefonillo. Viabilidad pendiente de la corriente que da el hilo 4.

## Enlaces

- [[portero-automatico-timbre-unico]]
- [[portero-comprobaciones-en-sitio]]
- [[_index]]
