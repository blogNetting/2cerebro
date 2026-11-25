---
title: Portero 4+N — hilos, bornes y señales
created: 2026-11-23
updated: 2026-11-25
tags: [portero-automatico, fermax, tegui, mediciones]
zona: tecnico
---

Lo comprobado del portero del 4ºB: qué hilo es cada color, a qué borne va en cada teléfono, qué tensiones hay y qué tipo de llamada usa la línea.

Proyecto: [[portero-automatico-timbre-unico]]. Revisión del diseño del selector: [[portero-selector-revision-2]]. Fuente bruta: `fuentes/portero-pisos-2026-11-23/`.

## Sistema

- **4+N convencional**: cuatro hilos comunes a todo el edificio y uno de llamada por vivienda. A la vivienda llegan 5 colores (medido, 19-11-2026).
- Lo confirma el teléfono de arriba: Tegui Horizon de tecla verde, modelo 0E7020 — «Sistema Convencional 4+n Llamada Electrónica. Modelo 0E7020» y «0E7020. Tecla Verde» ([Tegui Distel](https://teguivideoporteros.es/todo-lo-que-necesita-saber-sobre-los-telefonos-horizon-de-tegui/)).
- ~~Marca de la placa de calle: Fermax según el usuario~~ **Corregido 2026-11-25: placa Tegui Serie Europa** (dato del usuario, referencias comprobadas en el catálogo oficial). Detalle en «Placa de calle» más abajo.

## Placa de calle: Tegui Serie Europa (4+n convencional)

Referencias dadas por el usuario (2026-11-25) y comprobadas en el [catálogo-tarifa Tegui 2016](https://www.plcmadrid.es/wp-content/uploads/catalogos/TEGUI/catalogo%20material%20electrico-tarifa-2016-tegui.pdf), sección «Módulos función – Sistema Convencional» (copia de las páginas 90–92 en `fuentes/portero-pisos-2026-11-23/manuales/tegui-catalogo-tarifa-2016-serie-europa-p90-92.pdf`):

- **Grupo fónico (amplificador de la placa): EGF-1**, «0E5501 Grupo fónico EGF-1.». Según el usuario puede ser el **EGF-21** («090021 Grupo fónico EGF-21 para perfiles con rejilla compacta.» o 090121 con ventana), según la antigüedad. Los dos usan las mismas bornas de línea (2, 3, 4, 5, 6).
- **Pulsadores: 18 en 2 columnas** = 4 × «006101 Cassette 4 pulsadores 2 columna.» + 1 × «006100 Cassette 2 pulsadores 2 columnas.» (la combinación 4 + 1 es cálculo propio).
- **Alimentador: casi seguro E-14** (0E5514). *Inferencia, sin verificar:* Distel lo empareja con el EGF-1 en la versión convencional; el E-13 tiene «llamada por zumbador y secreto de comunicación» ([Distel](https://teguivideoporteros.es/portero-convencional-serie-europa-1-egf-1-0e5501-y-e-13-0e5513/)), y con secreto «Solo el teléfono al que se llama entra en comunicación y el resto permanecen en reposo», lo que contradice la prueba 17 (se oye la calle descolgando sin llamada). Se confirma mirando la etiqueta del alimentador en el portal.

Lo que dicen los manuales oficiales (copias en `fuentes/portero-pisos-2026-11-23/manuales/`; el del E-14 y el de Europa 1 son escaneos, citas leídas en la imagen):

| Hecho | Respaldo |
|---|---|
| **La llamada la genera el alimentador, no la placa**: borna 20 (llamada electrónica) o 19 (zumbador) al común de pulsadores; el pulsador la lleva al hilo de la vivienda | [Manual E-14](https://cdn.shopify.com/s/files/1/1013/0174/8051/files/Alimentador_Tegui_E14.pdf): «LA LLAMADA EN LOS TELEFONOS — 20 a placa (teléfono HE o T71E). 19 a placa (teléfono HN o T71Z).» |
| **El azul (borna 4) es la voz del piso hacia la calle, y su volumen se regula en el EGF-1** | [Europa 1, cap. C](https://cdn.shopify.com/s/files/1/1013/0174/8051/files/C-Portero_zumbador.pdf): «4. EL SONIDO DE LOS TELEFONOS EN PLACA — 2 y 4 a teléfonos. Volumen de placa.» · «8. EL SONIDO (EFECTO LARSEN O ACOPLAMIENTO) — Regular los mandos de volumen del EGF-1.» |
| El amarillo (borna 5) es la voz de la calle hacia el piso | Manual E-14: «EL SONIDO DE PLACA EN LOS TELEFONOS — 5 y 2 al EGF-1.» |
| **La confirmación de llamada** (el tono que suena en la calle al pulsar) entra también por la borna 4, el azul | Manual E-14: «LA CONFIRMACION DE LLAMADA — 4 del E-14 al EGF-1.» |
| EGF-1 alimentado a 12 V⎓ entre 6 y 2; abrepuertas 10–12 V⎓ entre 3 y 2 (coincide con el rojo medido, 11,84 V) | Manual E-14: «ILUMINACION PILOTO EGF-1 — 6 y 2 al EGF-1. (12VDc).» · «3 a teléfonos (3 y 2 de 10 a 12 VDc).» |
| Máximo 3 teléfonos en paralelo por hilo de llamada (en el 4ºB hay 2) | [Manual EGF-21](https://www.antares-sistemas.es/descargas/manuales/090121_MANUAL_ESPA%C3%91OL.pdf): «Colocación máxima de 3 teléfonos en paralelo.» |
| Amplificador del EGF-21: 1 W con regulación de volumen. **Potencia del EGF-1 no encontrada** | Manual EGF-21: «amplificador integrado de 1W, regulación de volumen para placa y teléfonos» |

Foto del EGF-1 con sus 6 bornas y los dos mandos de volumen (teléfono y placa) en la parte trasera: [Distel](https://teguivideoporteros.es/averia-portero-convencional-serie-europa/). No se guarda en `fuentes/` porque es material de terceros y el repo es público.

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
- ~~Nivel según Fermax: 8–12 Vpp~~ **Corregido 2026-11-25:** ese dato es del teléfono Fermax Citymax ([manual oficial](https://docs.fermax.com/manuales/97771b%20TELEFONO%20CITYMAX%204n%20V02_19.pdf)), no de esta instalación Tegui. El manual del E-14 no da el nivel de la llamada. El único dato Tegui es del EGF-21, que genera su propia llamada: «(12 Vdc. en reposo, si se llama 3 Vdc.)» entre sus bornas 20 y 2 ([manual Serie Europa 2000](https://cdn.shopify.com/s/files/1/1013/0174/8051/files/Tegui_Manual_Portero_Convencional_Serie_Europa.pdf)). **Nivel real en el verde: por medir.**
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
- **01 y 03 son los valores normales de un Tegui con EGF-1** (2026-11-25): Distel da como correctos, con los teléfonos colgados, «bornes 2 y 4 (10V DC) y bornes 2 y 5 (10V DC)» ([Distel](https://teguivideoporteros.es/averia-portero-convencional-serie-europa/); distribuidor y servicio técnico, no el fabricante). Medido: azul 10,6 V y amarillo 9,95 V. ~~La hipótesis anterior (que la tensión real era ≈ 11,84 V y el multímetro la bajaba a ≈ 10 V) ya no hace falta.~~ 05, 07 y 08 (diferencias de pocos mV entre rojo, azul y amarillo con lecturas que «oscilan») quedan sin explicar, sin consecuencias para el diseño.
- **Consecuencia**: el nivel del mensaje inyectado no se puede calcular de antemano; se ajusta probando, **con R2 y con el mando «Volumen de placa» del EGF-1**. Ese mando está en la placa de calle (elemento común) y cambia también cómo suena en la calle la voz de todos los vecinos; subirlo de más provoca acople.
- Micrófono de los teléfonos Fermax: «Electret. Resistencia dinámica 50 Ω» (manual Citymax).

## Audio hacia la calle

- Prueba 17 (19-11-2026): con un teléfono descolgado y sin llamada, la voz se oye en la calle «Sí, claro». El canal vivienda → calle funciona sin llamada.
- Sin probar con todos los teléfonos colgados, que es como hablaría el selector.

## Enlaces

- [[portero-automatico-timbre-unico]]
- [[portero-selector-revision-2]]
- [[portero-comprobaciones-en-sitio]]
- [[_index]]
