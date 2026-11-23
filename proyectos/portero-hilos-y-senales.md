---
title: Portero 4+N — hilos, bornes y señales
created: 2026-11-23
updated: 2026-11-23
tags: [portero-automatico, fermax, tegui, mediciones]
zona: tecnico
---

Lo comprobado del portero del 4ºB: qué hilo es cada color, a qué borne va en cada teléfono, qué tensiones hay y qué tipo de llamada usa la línea.

Proyecto: [[portero-automatico-timbre-unico]]. Revisión del diseño del selector: [[portero-selector-revision-2]]. Fuente bruta: `fuentes/portero-pisos-2026-11-23/`.

## Sistema

- **4+N convencional**: cuatro hilos comunes a todo el edificio y uno de llamada por vivienda. A la vivienda llegan 5 colores (medido, 19-11-2026).
- Lo confirma el teléfono de arriba: Tegui Horizon de tecla verde, modelo 0E7020 — «Sistema Convencional 4+n Llamada Electrónica. Modelo 0E7020» y «0E7020. Tecla Verde» ([Tegui Distel](https://teguivideoporteros.es/todo-lo-que-necesita-saber-sobre-los-telefonos-horizon-de-tegui/)).
- **Marca de la placa de calle: Fermax según el usuario, sin verificar abriendo la placa.** La foto de placa usada en el chat era «similar». El 0E7020 se vendía con equipo Tegui: «iba asociado a un equipo compuesto por fuente de alimentación y grupo fónico» (misma fuente). Para saber marca y referencia del amplificador hay que abrir la placa del portal.

## Bornes y colores

| Función | Color | Fermax (VEO) | Tegui Horizon E |
|---|---|---|---|
| Abrepuertas y alimentación del micrófono | Rojo | 1 | 3 |
| Micrófono (vivienda → calle) | Azul | 2 | 4 |
| Masa común | Gris | 3 | 2 |
| Llamada | Verde | 4 | 1 (1E) |
| Altavoz (calle → vivienda) | Amarillo | 6 | 5 |

Respaldo:

- Fermax VEO: «1: Abrepuertas y alimentación micrófono. 2: Micrófono Teléfono. 3: Masa común. 4: Hilo de Llamada. 6: Altavoz Teléfono.» ([manual VEO](https://d7rh5s3nxmpy4.cloudfront.net/CMP2008/files/11/970176c_Telefono_Universal_VEO_V04_21.pdf), copia en `fuentes/portero-pisos-2026-11-23/manuales/`).
- Tegui: tabla del T-71U, fila «TEGUI HORIZON E» con 1E=1, 2=2, 3=3, 4=4, 5=5, y bornas «1E Llamada el. · 2 Masa común · 3 Abrepuertas · 4 Micrófono · 5 Altavoz» ([manual T-71U](https://www.antares-sistemas.es/descargas/manuales/T-71U_MANUAL_ESPA%C3%91OL.pdf)).
- Tabla de equivalencias aportada por el usuario (foto, 2026-11-23): «TEGUI 4 5 2 3 1» para Micrófono, Altavoz, Común, Pechadura (cerradura), Zumbador. Coincide.
- La tabla del VEO da para Tegui «3 4 2 5 1» en el orden de bornes VEO 1, 2, 3, 6, 4. Coincide.
- Los colores salen de las mediciones del usuario y de los bornes de los dos teléfonos.

## Llamada: electrónica, no 50 Hz

- **Conclusión: la línea es de llamada electrónica.** Se deduce de que el Tegui 0E7020 (llamada electrónica) suena en ella. Es inferencia, no medida: la prueba 12 (verde al pulsar) no tiene dato.
- Nivel según Fermax: teléfono Citymax 4+N, el del kit 6201 — «4 Llamada Electrónica» y «Llamada Electrónica: 8 – 12 Vpp» ([manual oficial](https://docs.fermax.com/manuales/97771b%20TELEFONO%20CITYMAX%204n%20V02_19.pdf)). Vpp = voltios pico a pico.
- El VEO admite las dos: «Llamada Zumbador … 9 – 12 Vac» y «Llamada Electrónica … 8 – 12 Vpp» (manual VEO). Su tabla de equivalencias pone «FERMAX 20440, 21100 … E» y «TF-1Z, TN-Z … Z»: **en esta línea, SW2 del VEO en E**.
- Descartado: «llamada en 12 V alterna» atribuido al kit 6201. La página citada solo dice «alimentador de 12 Vac y 1,5 A … referencia 4800» y «4 · Llamada o zumbador» ([TDT Profesional](https://www.tdtprofesional.com/blog/manual-de-instalacion-para-el-kit-6201-de-fermax/)); el teléfono de ese kit es de llamada electrónica.
- Medir la llamada en V~ con el MAS830 no es fiable (ver abajo): medirla en V⎓ o con el propio detector.

## Tensiones medidas (19-11-2026, MAS830, todo colgado, sin llamada)

| Prueba | Negra → roja | V⎓ | V~ |
|---|---|---|---|
| 01 | gris → amarillo | 9,95 | 0 (con puntas invertidas) |
| 02 | gris → verde | 0 | 0 |
| 03 | gris → azul | 10,6 | 22,1 |
| 04 | gris → rojo | 11,84 | 25,2 |
| 05 | rojo → amarillo | 40 mV (escala 200 mV, «oscila») | 0,01 |
| 06 | rojo → verde | 11,83 | 25,2 |
| 07 | rojo → azul | 26,8 («oscila») | 0,0 |
| 08 | azul → amarillo | 0,03 | 0 |
| 09 | azul → verde | 10,59 | — |
| 10 | verde → amarillo | 9,93 | 0 |

Datos en `fuentes/portero-pisos-2026-11-23/mediciones-2026-11-19/`.

- **Las lecturas de V~ no son alterna real** (juicio): salen ≈ 2,1 × la continua y dependen de la polaridad de las puntas. Es el comportamiento de un multímetro que en V~ no bloquea la continua. No se encontró ficha del MAS830 que lo confirme.
- **05, 07 y 08 no son una anomalía** (hipótesis coherente con todas las medidas): rojo, azul y amarillo estarían a ≈ 11,84 V a través de resistencias internas distintas (≈ 0, ≈ 117 kΩ y ≈ 190 kΩ). La entrada de ≈ 1 MΩ del multímetro baja la lectura contra gris a 10,6 y 9,95 V, y entre ellos la diferencia real es casi 0. Supone 1 MΩ de entrada, lo habitual en multímetros de la familia DT830; sin ficha del MAS830.
- **Consecuencia**: con los teléfonos colgados, la línea azul tiene alta impedancia en continua. El nivel del mensaje inyectado (R2) no se puede calcular de antemano: se ajusta probando.
- Micrófono de los teléfonos Fermax: «Electret. Resistencia dinámica 50 Ω» (manual Citymax).

## Audio hacia la calle

- Prueba 17 (19-11-2026): con un teléfono descolgado y sin llamada, la voz se oye en la calle «Sí, claro». El canal vivienda → calle funciona sin llamada.
- Sin probar con todos los teléfonos colgados, que es como hablaría el selector.

## Enlaces

- [[portero-automatico-timbre-unico]]
- [[portero-selector-revision-2]]
- [[portero-comprobaciones-en-sitio]]
- [[_index]]
