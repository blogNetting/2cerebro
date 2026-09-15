---
title: Portero automático — comprobaciones en sitio
created: 2026-09-15
updated: 2026-09-15
tags: [portero-automatico, mediciones, domotica]
zona: tecnico
---

Procedimiento de comprobaciones y mediciones sobre la instalación real que validan o tumban los supuestos de [[portero-automatico-timbre-unico]]. Nada del diseño se cierra sin pasar las bloqueantes.

## Cómo leer esta lista

- **Bloqueante**: si el resultado no es el esperado, cambia el diseño.
- **Dimensionado**: no tumba nada, da los valores para elegir componentes.
- Los pasos van en **orden de ejecución**: los posteriores necesitan saber qué color es cada terminal.
- Los valores esperados salen del [manual oficial Fermax Citymax 4+N, cod. 97771b V02_19](https://fermax.com/.doc-download?type=handbook&slug=97771b-telefono-citymax-4-n-proy-265-ru-arab-es-en-fr-de-ar-ru&lang=es). Es una revisión de 2019 y la instalación tiene ~30 años: el mapa de terminales es fiable, los voltajes se miden.

## Material

- Polímetro True RMS.
- Osciloscopio de 2 canales (vale uno USB barato) con sondas x10.
- Portátil a batería, sin el cargador enchufado.
- Móvil con app generadora de tonos (1 kHz) y cable jack cortado o adaptador jack → cocodrilos.
- Resistencia de 1 kΩ y condensador de 10 µF no polarizado (o dos electrolíticos de 22 µF en antiserie).
- Cinta aislante, rotulador y cinta de carrocero para etiquetar hilos.
- Ayudante en la placa de calle, en llamada de móvil contigo durante todas las pruebas.

## Preparación del polímetro

- Punta **negra** en el borne **COM**.
- Punta **roja** en el borne **VΩ** (a veces marcado V/Ω/mA con rayo). Nunca en **A**, **mA** ni **10A**.
- La rueda solo va a dos zonas en todo este procedimiento:
  - **V⎓** (también rotulado **DCV** o V con línea recta y discontinua): tensión continua.
  - **V~** (también rotulado **ACV** o V con onda): tensión alterna.
- Si el polímetro es de escalas manuales, empezar siempre en **200 V** y bajar a **20 V**, luego **2 V** / **200 mV**, mientras la lectura quepa en la escala inferior. Si pone "1" o "OL", la escala se ha quedado corta: subir.
- Si es autorrango, basta con poner la rueda en V⎓ o V~.

## Reglas de seguridad

- Nunca unir el terminal 1 con el 3: abre el portal.
- Nunca puentear el 4 con ningún otro terminal.
- Sobre la instalación conectada, la rueda **solo en V⎓ o V~**. Nunca en Ω, continuidad (pitido), diodo ni amperios: en esas posiciones el polímetro mete corriente en la línea o hace de cortocircuito.
- Pinza de masa del osciloscopio **siempre en el 3**, y portátil a batería. Con cargador enchufado la masa de la sonda puede quedar referida a tierra.
- Foto antes de soltar cualquier hilo. Si se suelta uno: etiqueta, punta aislada, y se vuelve a conectar al acabar la prueba.

## Paso 1. Identificación y mapeo de colores

**Cómo:**

1. Telefonillo 1: quitar el embellecedor del tornillo de la tapa, desatornillar y retirar la tapa.
2. Foto de los terminales con los números legibles.
3. Apuntar en la tabla el color de cada terminal y cuántos hilos hay en cada tornillo (1 o 2).
4. Apuntar el modelo (carcasa o serigrafía de la placa interior).
5. Repetir en el telefonillo 2.
6. En la placa de calle, sin abrirla (elemento común): foto, marca/modelo visible y nº de pulsadores.
7. Que el ayudante llame y escuchar cómo suena: tono electrónico o zumbido.

| Term | Función (manual) | Color tel. 1 | Hilos/borne tel. 1 | Color tel. 2 | Hilos/borne tel. 2 |
|---|---|---|---|---|---|
| 1 | Abrepuertas | | | | |
| 2 | Micrófono teléfono | | | | |
| 3 | Masa común | | | | |
| 4 | Llamada electrónica | | | | |
| 6 | Altavoz teléfono | | | | |

En la regleta se ven amarillo, verde, azul y rojo; el quinto se identifica aquí. Los colores no siguen norma: no deducir función por color.

## Paso 2. Topología — bloqueante

**Qué:** por dónde entran los 5 hilos del montante y cómo se reparten entre la regleta y los dos telefonillos.

**Cómo:**

1. En cada punto (telefonillo 1, telefonillo 2, caja de la regleta), mirar los hilos por borne: 1 hilo = final de línea; 2 hilos = la línea continúa hacia otro punto.
2. Seguir hacia dónde salen los tubos de cada caja.
3. Si no queda claro: con todo en reposo, soltar **solo el hilo 4** en la regleta, aislar la punta y pedir una llamada. Apuntar qué suena y reconectar.
   - No suena ninguno → la regleta está antes de los dos telefonillos. Punto común.
   - Suena el tel. 1 y no el tel. 2 → el tel. 1 cuelga de otro punto anterior a la regleta.
   - Suenan los dos → ese hilo no es el 4, o hay otra derivación. Revisar el paso 1.
4. Dibujar a mano el esquema: montante → cajas → telefonillos.

**Por qué:** el corte del hilo 4 tiene que quedar entre el montante y los dos telefonillos a la vez. Si no existe un punto común previo a ambos, cambia dónde va el aparato.

## Paso 3. Tensiones en reposo — dimensionado

Polímetro: negra en COM, roja en VΩ (ver preparación).

**Cómo:**

1. Los dos telefonillos colgados. Punta negra fija en el terminal **3**.
2. Rueda en **V⎓ 200 V** (autorrango: V⎓).
3. Punta roja en **1**. Si la lectura es menor de 20 V, rueda a **V⎓ 20 V** y volver a medir; si marca 0,00, bajar a **2 V**. Apuntar.
4. Igual en **2**, **4** y **6**.
5. Rueda en **V~ 200 V** (autorrango: V~) y repetir los puntos 3 y 4, bajando de escala igual.
6. Descolgar un auricular. Repetir en **2** y **6**, primero en V⎓ y luego en V~.

| Term | V⎓ colgado | V~ colgado | V⎓ descolgado | V~ descolgado |
|---|---|---|---|---|
| 1 | | | — | — |
| 2 | | | | |
| 4 | | | — | — |
| 6 | | | | |

## Paso 4. Hilo 4 durante la llamada — bloqueante

**Preparación del osciloscopio:**

- Masa en 3. CH1 en 4, sonda x10 (y el canal configurado en x10), acoplamiento DC, 5 V/div.
- CH2 en 6, empezar en 50 mV/div y ajustar. Sirve de marca de tiempo (ver 4b).
- Disparo en CH1, flanco de subida, nivel ~2 V, modo normal.

**4a. Forma de onda**

1. Base de tiempos 5 ms/div.
2. Ayudante: una pulsación larga (~3 s). Congelar la pantalla (single).
3. Apuntar Vpp, frecuencia, forma (senoidal, cuadrada, pulsos) y si es simétrica respecto a 0 V.
4. Polímetro: negra en 3, roja en 4, rueda en **V~ 20 V** (autorrango: V~). Ayudante: otra pulsación larga. Apuntar la lectura. Es orientativa; el valor bueno es el del osciloscopio.

Esperado según manual: 8–12 Vpp. La nota del portero dice "12 V AC"; se corrige con este dato.

**4b. ¿La señal sigue al pulsador?**

1. Base de tiempos 500 ms/div en modo roll o captura larga. Grabar la pantalla en vídeo con el móvil.
2. Ayudante, pulsando cada vez con un golpe seco: el clic mecánico entra por el micro de la placa y aparece en CH2 como marca del instante de pulsación.
   - 5 pulsaciones cortas (~0,3 s) separadas 2 s.
   - 3 pulsaciones largas (~3 s) separadas 3 s.
   - 3 pulsaciones seguidas separadas ~1 s.
3. Apuntar para cada serie: ¿la señal en CH1 dura lo mismo que la pulsación? ¿Alguna pulsación no produce señal? ¿Hay ráfagas o cadencia propias sin relación con la pulsación?

**4c. Retardo**

En el vídeo, medir la distancia entre el clic en CH2 y el inicio de la señal en CH1.

**4d. Efectos en otros hilos**

Polímetro: negra en 3. Una pulsación larga del ayudante por cada medida:

1. Roja en **1**, rueda en **V⎓ 20 V**. Apuntar. Rueda en **V~ 20 V**, otra pulsación. Apuntar.
2. Igual en **2** y en **6**.
3. Comparar con la tabla del paso 3 y apuntar qué cambia.

**Por qué 4a–4d:** el esquema "pulse ahora" detecta la presencia de señal en el hilo 4 dentro de cada ventana. Si la señal no sigue al pulsador, ese esquema no funciona tal cual.

**4e. Corriente que da el hilo 4 — bloqueante**

No se mide con el polímetro en amperios: se carga el hilo con una resistencia conocida y se mide la tensión.

1. Osciloscopio como en 4a: masa en 3, CH1 en 4, x10, 5 V/div, 5 ms/div. Polímetro: negra en COM y en el terminal 3, roja en VΩ y en el terminal 4, rueda en **V~ 20 V** (autorrango: V~).
2. Ayudante: pulsación larga. Apuntar Vpp y lectura V~ **sin carga**.
3. Conectar la resistencia de 1 kΩ (sirve la del paso 6) entre 4 y 3 con cocodrilos, con los telefonillos conectados.
4. Ayudante: otra pulsación larga. Apuntar Vpp y lectura V~ **con carga**.
5. Quitar la resistencia.
6. Corriente con carga: con 1 kΩ, la lectura en voltios es la corriente en miliamperios. Ejemplo: 3,5 V → 3,5 mA.

**Por qué 4e:** si el aparato falla, la propia señal de llamada tiene que accionar la apertura del portal. Si la tensión apenas cae con la carga, la señal da corriente suficiente para eso.

## Paso 5. Audio — confirmación y dimensionado

**5a. Confirmación**

Descolgar sin que nadie llame; el ayudante habla desde la placa. ¿Os oís en los dos sentidos? Esperado sí.

**5b. Niveles**

1. Auricular descolgado. Masa en 3, CH1 en 2 (tu voz hacia la placa), CH2 en 6 (voz de la placa hacia ti). Sondas en x10.
2. Acoplamiento AC, 100 mV/div, base de tiempos 50 ms/div.
3. El ayudante cuenta del 1 al 10 en voz normal frente a la placa; tú haces lo mismo en el auricular.
4. Apuntar el pico a pico máximo en 2 y en 6.
5. Pasar CH1 a acoplamiento DC y apuntar la continua en 2 (polarización del micrófono). Debe coincidir con la lectura V⎓ descolgado del paso 3.

## Paso 6. Primera prueba de inyección — bloqueante

**Montaje:**

- Móvil a batería, sin cargar, con el tono de 1 kHz y el volumen al mínimo.
- Punta del jack → resistencia 1 kΩ → condensador 10 µF → terminal 2.
- Manguito (masa) del jack → terminal 3.

**Cómo:**

1. Los dos telefonillos colgados. Ayudante en la placa, en llamada contigo.
2. Arrancar el tono y subir el volumen poco a poco hasta que el ayudante lo oiga. Apuntar el volumen del móvil y si llega limpio o distorsionado.
3. Con el osciloscopio en 2 (masa en 3, x10, acoplamiento AC), apuntar el Vpp cuando se oye bien.
4. Repetir con una grabación de voz en lugar del tono para valorar si se entiende.
5. Pruebas cortas: el bus es común y lo oye cualquier vecino que descuelgue.

**Por qué:** valida la inyección con los telefonillos colgados, que es como trabajará el aparato.

## Paso 7. Espacio y alimentación — dimensionado

1. Medir el interior de la caja de la regleta: ancho × alto × fondo, y el hueco libre sin mover la regleta.
2. Localizar el enchufe o punto de 230 V más cercano y medir la distancia por pared.
3. Buscar registros o tapas ciegas cercanas y medirlos.

## Resultados

Solo lo medido, con fecha.

## Enlaces

- [[portero-automatico-timbre-unico]]
- [[apartamentos-calle-uruguay]]
- [[_index]]
