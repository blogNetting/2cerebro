---
title: Selector de planta — revisión 2 (2Cerebro)
created: 2026-11-23
updated: 2026-11-23
tags: [portero-automatico, esp32, revision]
zona: tecnico
---

Revisión del 23-11-2026 del selector de planta (el «Cerebro») contra la documentación oficial y las medidas reales: qué está mal en los HTML, qué hay que corregir y qué queda por probar.

Hechos del portero en los que se apoya: [[portero-hilos-y-senales]]. Proyecto: [[portero-automatico-timbre-unico]]. HTML revisados, copiados en `fuentes/portero-pisos-2026-11-23/artefactos/`: Selector de planta, Pruebas del Cerebro y Pruebas portero Fermax.

## El diseño montado (estado a 23-11-2026)

- ESP32: clon de **ESP32-DevKitC V4** (serigrafía trasera «ESP32_DEVKITC_V4»), módulo **«ESPRESSIF ESP32-WROOM-32D»**, 38 pines y micro-USB, comprobado en las fotos del usuario del 19 y 21-11-2026. Orden de pines impreso igual que el de Espressif, salvo el pin bajo el 5V: pone «GND», pero en el DevKitC oficial es CMD (GPIO11, conectado a la memoria flash). En el foro de Arduino, un usuario con la misma placa dice «it does not work as ground definitely» (definitivamente no funciona como masa) y le responden «should be marked CMD, which is GPIO 11» (debería poner CMD, que es GPIO11) ([hilo](https://forum.arduino.cc/t/weird-gnd-in-esp32-gnd-or-gpio-11/958140)). El diseño no usa ese pin. Regulador «AMS1117 3.3» y dos transistores «J3Y» (foto de detalle del usuario, 2026-11-23). El chip USB-serie es de encapsulado cuadrado (QFN) sin rótulo legible: por la forma, CP2102 o CH9102 (deducción). Se confirma en el Administrador de dispositivos de Windows al enchufarlo. Fabricante de la placa desconocido.
- ESP32 en tiras hembra sobre placa perforada de 7 × 9 cm, con bornas J1, J3, J4 y J5.
- Módulo de 2 relés ZS-2-RELAY-1 (puentes en H) que corta solo el verde. Cada teléfono va por el contacto NC de su relé: con los relés sueltos suenan los dos.
- Detector de llamada: R1 de 1 kΩ, D1 y 4N35 hacia GPIO4.
- Mensaje: DAC de GPIO25 → C1 de 10 µF → R2 (2 × 100 kΩ en paralelo) → azul.
- Alimentación: cargador de 5 V propio. La masa de la ESP va unida al gris.
- Pruebas de placa del 23-11: todo lo medido salió bien. Pendientes B3g (bornas 8–9) y B12 (patas 5–4 del 4N35).

## Errores confirmados en los HTML

1. **Tipo de llamada.** Presentan «alterna de 50 Hz (≈ 12 V~)» como hipótesis principal, calculan el detector para 17 V de pico y citan el kit 6201 como «llamada en 12 V alterna». La línea es de llamada electrónica, 8–12 Vpp (ver [[portero-hilos-y-senales]]). El cálculo de 15 mA no aplica; el detector sigue funcionando (≈ 2–4 mA en el LED, cálculo propio).
2. **«Si la ESP se cuelga, los relés sueltan»: falso.** En modo selector, en reposo, IN1 e IN2 están en alto. Con el programa colgado se quedan así y no suena nadie.
3. **SW2 del VEO**: el selector dice que el manual no indica la posición. Su tabla sí distingue modelos E y Z: en esta línea va en E.
4. **Prueba 16** de «Pruebas portero Fermax»: pide en el Tegui los bornes 1, 2, 3, 4 y 6. El Tegui usa 1–5.
5. **Datos del 4N35.** El HTML dice «máx. 60 mA» y «ganancia mínima 20 %». Vishay: «Forward current IF 50 mA» y «CTR > 100 %» a 10 mA, con mínimo del 40 % entre −55 y +100 °C ([ficha](https://www.vishay.com/docs/81181/4n35.pdf)). Otros fabricantes dan 60 mA. El margen real es mayor que el del HTML.
6. **El selector está desactualizado respecto a «Pruebas del Cerebro»:**
   - Describe D1 en serie, cuando ya se decidió el antiparalelo.
   - Da el audio sin llamada como pendiente, cuando la prueba 17 ya dio «Sí, claro».
   - Su «prueba 18» es la 17 de la página de pruebas.
   - Espera la «fila K», cuando la placa usa la columna C.
   - Pone bornas «J1–J4», cuando son J1, J3, J4 y J5.
   - Llama a dos pruebas «R1» y «R2», igual que las resistencias.

## Conclusiones técnicas (no son decisiones de compra del usuario)

- **Programa del detector**: dar la llamada por buena con nivel bajo mantenido (≥ 80–100 ms) o con tren de pulsos, no contando flancos. Con un tono de kHz y el pull-up interno (≈ 45 kΩ), el 4N35 puede no apagarse entre ciclos. La ficha de Vishay solo da «ton, toff 10 µs» con RL = 100 Ω.
- **Watchdog** (vigilante que reinicia la ESP si el programa se bloquea), con contador de reinicios en memoria RTC: si hay varios reinicios seguidos en poco tiempo, arrancar en modo normal con los relés sueltos. Un reinicio simple vuelve a activar los relés en ≈ 1 s.
- **D1 en antiparalelo** (ya decidido en «Pruebas del Cerebro», pruebas H1–H3): limita la inversa del LED a una caída de diodo. En esa posición sirve cualquier diodo, también un Schottky. D1 marcó 0,42 V, bajo para un 1N4007: mirar el código impreso o medir otro del kit.
- **Relés: el contacto de reposo debe ser NC**, para que un fallo deje sonar los dos teléfonos. Descartados como sustitutos: los reed estándar, que son normalmente abiertos (NO), y el CPC1014N: «Solid State Photo-Coupled Relay (Photorelay) SPST-NO (1 Form A)», además «CPC1014N is Obsolete» ([Digi-Key](https://www.digikey.com/en/products/detail/ixys-integrated-circuits-division/CPC1014N/1973068), leído en Chrome el 2026-11-23). El AQV414 sí es «1 Form B», normalmente cerrado ([Panasonic](https://na.industrial.panasonic.com/products/relays-contactors/lineup/semiconductor-relays/series/12495/model/12506)).
- **Relés del módulo**: confirmado con foto del módulo propio (2026-11-23): «QX73-C-S», «DC5V», «12A/125V~ · 12A/28V⎓ · 7A/250V~»; logo de marca ilegible (parecido a «NJK»). Son relés de potencia, no Songle SRD ni «QXT3-C-5». No se ha encontrado ficha con el material de contacto ni la carga mínima ([DatasheetCafe](https://datasheetcafe.com/qx73-c-s-datasheet-pdf/) solo da «Microminiature relay, 12A switching capability»). Para relés de potencia, Finder: «a 16 A Finder power relay has a specified minimum switching load of 10 V / 10 mA / 1000 mW» (un relé de potencia de 16 A tiene una carga mínima de 10 V / 10 mA / 1000 mW; [Finder](https://www.findernet.com/en/uk/news/relay-contact-materials-does-it-matter/)). La llamada (8–12 Vpp, pocos mA) queda por debajo. El material de contacto de estos relés no está impreso ni comprobado.
- **Relés NC de estado sólido** (PhotoMOS u OptoMOS, si se cambian). Todos comprobados en la fuente con Chrome el 2026-11-23:
  - AQV414: «1 Form B» ([Panasonic](https://na.industrial.panasonic.com/products/relays-contactors/lineup/semiconductor-relays/series/12495/model/12506)).
  - AQY414EH: «AQY414EH 1 Form B DIP4 Through hole terminal AC/DC 400 0.12» ([Panasonic](https://industry.panasonic.com/ap/en/products/control/relay/photomos/ge1b_aqy41_e)). De patillas pasantes: se puede soldar en placa perforada.
  - CPC1117N: «SPST-NC (1 Form B) 4-SOP» ([Digi-Key](https://www.digikey.ca/en/products/detail/littelfuse-inc/CPC1117N/1280728)). De montaje superficial.
  - LCB110: «Normally Closed Relays Series: 1-Form-B», con estado «Not for New Designs» (no recomendado para diseños nuevos) ([Littelfuse](https://www.littelfuse.com/products/power-semiconductors-control-ics/solid-state-relays/optomos-relays/normally-closed-relays/1-form-b/lcb110)).
- **Relé de señal electromecánico**: Omron G5V-2, DPDT (dos contactos conmutados, con NC), «Contact material Ag + Au-alloy» (plata con aleación de oro) y tasa de fallo de referencia medida con «10 μA resistive load at 10 mVDC» (10 µA a 10 mV de continua) ([ficha Omron](https://omronfs.omron.com/en_US/ecb/products/pdf/en-g5v_2.pdf)). No cabe en el módulo ZS-2-RELAY-1, que tiene otra huella.
- **Nivel del mensaje**: si satura incluso con R2 = 100 kΩ, pasar de resistencia en serie a divisor resistivo a masa.
- **C1 de 10 µF**: no desoldarlo antes de oírlo en la pared. Si suena un golpe, primero alargar la rampa del DAC y dejarlo fijo en 1,65 V entre frases. Solo si sigue molestando, cambiar a 1 µF.
- **Condensador de 100 µF en 5 V–GND junto a la ESP**: ponerlo (está en el kit). Motivo: picos rápidos de corriente del WiFi y de los relés a través de ≈ 10 m de cable y del cargador. La caída en continua del cable es despreciable (≈ 0,07 V a 0,5 A con 2,5 mm²).
- **SW1 del VEO**: empezar en masa común. La tabla T-71U pide masas separadas para Fermax 20440/21100, pero el Tegui de esta línea funciona con masas unidas. Etiquetas en el dibujo del manual VEO (comprobado renderizando la página): **«U-» = «Masa común»** y **«F+» = «Masas separadas»** («Anlagenbetrieb mit Plus»). El «U− / F−» de «Pruebas del Cerebro» y del chat original es una errata: la posición buena es U-.
- **Tensiones del portero frente a la ESP (3,3 V).** Ningún pin recibe tensión del portero:
  - Verde (8–12 Vpp): pasa por el 4N35; el ESP solo ve su transistor.
  - Azul (10,6 V⎓): la continua la bloquea C1 (50 V). Los saltos (al conectar, al descolgar) pasan por R2 de 50 kΩ: ≈ 0,2 mA como mucho y menos de un segundo (cálculo propio). Sin comprobar en la ficha del ESP32 la corriente que admite un pin.
  - Gris: es la masa del ESP, a propósito, como referencia común.
  - Puentes del módulo en COM–H: además de la lógica, evitan que la entrada IN quede tirada a 5 V hacia el pin del ESP, como pasaría en COM–L (deducido del funcionamiento típico de estos módulos; esquema del ZS-2-RELAY-1 no visto).
  - El cargador de 5 V debe aislar bien de la red (de marca), porque su masa queda unida a la masa común del edificio.
- **Impacto en la placa soldada: ninguno**, salvo mover D1 (ya previsto). Solo se tocaría si las pruebas lo piden: R1 de 1 kΩ a 2,2 kΩ si el timbre baja, o el valor de R2 según el nivel del mensaje. Cambiar el módulo de relés no toca la placa si el nuevo tiene las mismas entradas (DC+, DC−, IN1, IN2, disparo a nivel alto).

## Revisión 2 cerrada: lo que manda ahora son 4 comprobaciones físicas

1. **Mover D1 a antiparalelo** y hacer H1–H3. Mirar el código impreso en D1 por lo del 0,42 V.
2. **En mesa: el módulo ZS-2-RELAY-1 cierra firme con 3,3 V en IN1/IN2** (puentes en COM–H; comprobado por el usuario que ya están así, 2026-11-23) y los relés se quedan quietos al arrancar la ESP. Si dudan: transistor NPN por canal con los puentes en COM–L, o relé NC de estado sólido.
3. **El 4N35 detecta la llamada en modo normal, con los dos teléfonos conectados**. Dentro de F4 se comprueba también si la señal dura lo que dura la pulsación (antigua prueba 13) y, **crítico, si el amplificador deja pasar una segunda pulsación a los 3 s** (antigua prueba 14, nunca hecha: el selector depende de ella) (peor caso de carga; si funciona aquí, funciona en las ventanas de un solo teléfono), y el timbre no pierde volumen con el Cerebro conectado. Sirve también como prueba 12: forma y nivel reales de la llamada.
4. **Audio de prueba por C1 + R2 en el azul, con los dos teléfonos colgados**: escuchar en la calle si llega limpio, saturado o con golpe.

Antes de dar corriente: B3g y B12 — **hechas el 2026-11-23: no pitan, sin cortos**.

Pasos detallados en la página «Pruebas del Cerebro» (https://claude.ai/artifact/7wxAZCPwR8RRKyXvMa2g35, versión 6 del 2026-11-23): B3g, B12, H0 (leer el código de D1), H1–H3, E7–E8 (relés con 3,3 V), F4 (detector con la llamada real), F5 (volumen del timbre) y F6 (mensaje simulado con el móvil). En esa misma versión se corrigieron V1 (SW1 en U-) y F3 (la llamada no se mide bien en V~).

Datos del AQY414EH, por si se cambian los relés ([ficha Panasonic, copia en RS](https://assets.rs-online.com/v1699293991/Datasheets/88e4424dacb594844e112da0eb0f9b25.pdf)):
- Corriente de apagado del LED: «LED operate (OFF) … Typical 1.4 mA … Maximum 3.0 mA», y «Recommendable LED forward current IF = 5 to 10mA».
- Resistencia en conducción: «On resistance … Typical … 26Ω … Maximum … 35Ω».
- LED en las patas 1–2 y salida entre las patas 3 y 4 («Measured portion: between terminals 3 and 4»).
- «Stable operation because there are no metallic contact parts» (funcionamiento estable porque no hay partes metálicas de contacto).

En Amazon.es (2026-11-23) aparece en DIP-4 por 14,35 € y 21,59 € la unidad.

Otras opciones normalmente cerradas, comparadas el 2026-11-23:
- **AQY412EH** (misma ficha): 60 V, 0,55 A, resistencia en conducción de 1 Ω típica y 2,5 Ω máxima, DIP-4 pasante. En Amazon.es, «(1 unidad) AQY412EH DIP-4» por 13,99 €. El AQY412EHA es de montaje superficial.
- **AQY410EH** (misma ficha): 350 V, 0,13 A, 18 Ω típica y 25 Ω máxima.
- **Descartadas**:
  - AQV414: DIP-6, más grande y sin ventaja.
  - CPC1117N: montaje superficial.
  - LCB110: «Not for New Designs».
  - Omron G3VM-353A, DIP-4 1b de 350 V: «Product Discontinuation Date: Mar. 31, 2026» (fecha de fin de fabricación), según su [página](https://components.omron.com/us-en/products/relays/G3VM--A---D---B---E-?page=0). Su sustituto, el G3VM-353G, es «SPST-NC (1 Form B) 4-SOP» (montaje superficial), según [Digi-Key](https://www.digikey.ca/en/products/detail/omron-electronics-inc-emc-div/G3VM-353G/935944): tampoco vale para placa perforada.
- Fiabilidad del estado sólido: solo hay documentación de fabricante; en foros no se encontraron casos de fallo en uso real. Omron y Panasonic dan como causas de fallo la sobretensión, la sobrecorriente y la electricidad estática. Omron describe un modo de fallo en cortocircuito: la salida conduce sin corriente en el LED ([guía Omron K293](https://components.omron.com/sg-en/ds_related_pdf/K293-E1.pdf), [precauciones Panasonic](https://industry.panasonic.com/global/en/products/control/relay/photomos/cautions_use)). En un NC, ese fallo deja la planta siempre conectada (suenan los dos); el fallo en abierto la dejaría muda.
  - Reed: son NO.
  - Omron G5V-2 con contactos de oro: válido, pero electromecánico (bobina, transistor y desgaste mecánico).

## Para el montaje: lo que falta del usuario (2026-11-23)

Estado: D1 **todavía no movido** y condensador de 100 µF **todavía no puesto**. Antes de soldar hace falta:

1. Fotos de cerca de la cara de soldaduras, rectas y con el borde (letras y números) legible: zona del 4N35, R1 y D1 (para mover D1 a antiparalelo) y zona junto al pin 5V de la ESP (para el 100 µF sin tocar uniones existentes). La foto de la cara inferior enviada es demasiado pequeña para ver qué agujeros están unidos.
2. Tensión impresa en el condensador de 100 µF del kit (vale cualquiera de 10 V o más) y cuál es su pata negativa.
3. Si quedan más 1N4007 en el kit (más cómodo que reutilizar D1 con las patas cortas).

Para las pruebas: alguien en la calle al teléfono (F4, F5), un cable de auriculares de 3,5 mm para cortar (F6), y soldador con malla o chupón para mover D1.

**Montaje de D1 corregido (2026-11-23):** no se desuelda D1. Se puentea D1 con un cable por debajo (T7 a V13) y se suelda un 1N4007 nuevo (D2) entre la pata 1 del 4N35 (franja) y la pata 2 (donde llega el cable blanco del gris), según el patillaje de Vishay «A 1 … C 2 … NC 3». **Error descartado:** añadir solo D2 dejando D1 en serie protege el LED, pero no deja volver la corriente en el semiciclo negativo; si la llamada sale por condensador, este se carga y el detector puede fallar en modo selector (cuando solo el detector carga la línea). Comprobación: H1 ≈ 1 kΩ, H2 900–1300, H3 400–700.

Orden recomendado: puentear D1 y añadir D2 → H1–H3 → **F4** (incluida la doble pulsación, que decide si el diseño vale) → E7–E8 → F5 → F6.

## HTML actualizados (2026-11-23)

- **Selector de planta** (https://claude.ai/artifact/TEkU65sKkPD6Nmu7uGH6kT, versión 41): pestaña nueva «Revisión 2 (2Cerebro)» con conclusiones, errores corregidos, cambios y pruebas. Además, 18 textos corregidos en su sitio y marcados «rev. 2»: tipo de llamada, watchdog, D1 en antiparalelo, SW1/SW2, datos del 4N35, ESP, relés, bornas y estado de las pruebas.
- **Pruebas del Cerebro** (https://claude.ai/artifact/7wxAZCPwR8RRKyXvMa2g35, versión 8): resumen de la revisión 2, pruebas nuevas E7–E8 y F4–F6, H0 marcada como innecesaria, y B3g y B12 hechas.
- **Pruebas portero Fermax** (https://claude.ai/artifact/VDFqa5R3fe7Rv84eDPHHx7): prueba 16 con los bornes 1–5 del Tegui, y nota de llamada electrónica en la prueba 12.

## Decidido por el usuario (2026-11-23)

- **Se mantiene el módulo de relés ZS-2-RELAY-1.** El cambio a estado sólido no compensa el trabajo para un riesgo que puede no darse. Si algún día una planta deja de sonar a ratos, el sustituto es el AQY414EH o el AQY412EH en DIP-4: se quitan los cables IO13 → borna 8 e IO27 → borna 7, se añaden 2 chips y 2 resistencias de 390 Ω, y las bornas 8 y 7 pasan a recibir los verdes del VEO y del Tegui.
- **D1 es un 1N4007** según el catálogo de compra del kit («Diodo rectifier (1N4007)»). El 0,42 V medido puede deberse a la poca corriente del modo diodo del multímetro (juicio, sin ficha del MAS830). En antiparalelo da igual: H0 deja de ser necesaria.

## Enlaces

- [[portero-hilos-y-senales]]
- [[portero-automatico-timbre-unico]]
- [[portero-comprobaciones-en-sitio]]
- [[_index]]
