---
title: Amortizaciones y deducciones del piso de Carballo (IRPF 2026)
created: 2026-10-17
updated: 2026-10-29
tags: [fiscalidad, irpf, amortizacion, galicia, alquiler]
zona: tecnico
---

Cálculo con las cifras reales del piso de [[apartamentos-calle-uruguay]]: amortización del inmueble, de la obra y del mobiliario, reglas de aplicación y qué deducciones aplican. Las reglas generales y sus fuentes están en [[fiscalidad-alquiler-por-habitaciones]]; aquí solo los números y lo que cambia para este caso.

> Origen: resumen del usuario del 2026-10-17, revisado punto por punto. Aritmética recalculada. Normas contrastadas con el BOE salvo donde se marca. No se guardan aquí la referencia catastral ni los sueldos: el repo es público.

## Datos del inmueble

- 123 m², 100 % del usuario, una referencia catastral, dos zonas privativas (PA y PB) con cocina y baño cada una.
- Renta prevista: 560 €/mes por zona (1.120 €/mes, 13.440 €/año), larga duración, vivienda habitual del inquilino.
- Valor catastral 2026: 53.208,03 € (suelo 20.896,51 + construcción 32.311,52). **% construcción = 60,73 %** (comprobado).
- Valor de referencia a 1-1-2025: 69.219,82 €. No cuenta para la amortización; sí para el tope de 250.000 € de la deducción gallega por obras.
- Compra: 88.000 € más gastos (ITP, notaría, registro, gestoría) pendientes de concretar. IBI 2026: 234,12 €.

## 1. Amortización del inmueble (3 %)

Base: el mayor entre el coste de adquisición y el valor catastral, siempre sin suelo (art. 14.2.a del Reglamento IRPF, [BOE](https://www.boe.es/buscar/act.php?id=BOE-A-2007-6820)). El coste gana: 53.442 € frente a 32.311,52 €.

| Base | × 60,73 % | 3 % anual |
|---|---|---|
| 88.000 € (sin gastos) | 53.442,40 € | **1.603 €** |
| 96.800 € (con ~8.800 € de gastos) | 58.786,64 € | **1.764 €** |

Cada 1.000 € de gastos de compra suma unos 18 €/año. A ese ritmo se completa en unos 33 años de alquiler.

## 2. Reforma: 18.850,99 € (cifra comprobada)

**Obra**: 14.414,13 € brutos, menos 400 € de indemnización del seguro, da **14.014,13 €**. La resta de esos 400 € es razonable, pero no he encontrado la norma que la respalde (sin verificar).

| Partida | € |
|---|---|
| Electricidad | 1.998,33 |
| Fontanería | 1.983,84 |
| Pladur | 618,81 |
| Carpintería (suelos, puertas, rodapiés, persianas) | 2.652,59 |
| Pintura | 2.658,30 |
| Baños | 1.432,07 |
| Cocina sin electrodomésticos | 2.760,12 |
| Otros | 172,23 |
| Licencias | 137,84 |

**Corrección importante: la clasificación «conservadora» no es la que dice el Reglamento.** El documento metía toda la obra como mejora al 3 % (420 €/año). Pero el art. 13.a del Reglamento llama reparación y conservación a «el pintado, revoco o arreglo de instalaciones» y a la «sustitución de elementos». Por eso el reparto es este:

- **Mejora (3 % anual)**: lo que crea algo que no existía, como la segunda cocina, los trasdosados o el pladur nuevo, y la parte de electricidad y fontanería que se hizo para la segunda unidad.
- **Reparación (gasto entero, con límite)**: la pintura y lo que sustituye elementos existentes, como suelos, puertas, persianas o sanitarios cambiados. Pintura y carpintería suman 5.310,89 €.
- **Cómo se reparte en la práctica**: se tiene que poder defender factura a factura. Si una factura mezcla partidas nuevas y sustituidas, lo prudente es tratarla como mejora.

**Límite de la reparación**: junto con los intereses, no puede superar los ingresos del año; el exceso se arrastra 4 años (art. 13.a). Si se pagó antes de alquilar, no se deduce en ese año: se arrastra a los años con ingresos, según la DGT (ver [[fiscalidad-alquiler-por-habitaciones]]). La amortización no tiene este límite.

**Mobiliario y electrodomésticos: 4.436,86 € al 10 % = 444 €/año.** Muebles 2.259,53 € más electrodomésticos 2.177,33 €. La tabla simplificada da un máximo del 10 % y un periodo máximo de 20 años **contados desde la adquisición**, no desde el primer alquiler. Fuente: DGT V0699-21, vista en resumen ([iberley](https://www.iberley.es/resoluciones/resolucion-vinculante-dgt-v0699-21-23-03-2021-1533174)).

**Muebles de cocina al 10 % en vez de al 3 %**: unos 193 €/año más. Ni el Reglamento ni las consultas encontradas aclaran si los muebles fijos de cocina son mobiliario o instalación. Sin verificar; mejor la opción del 3 %.

**Herramientas (690,73 €): no deducibles.** Solo se amortizan los bienes «cedidos con el mismo» inmueble (art. 14.1 del Reglamento).

**Altas de suministros (556,19 €)**: el documento las da como gasto entero el primer año. Es razonable como gasto necesario, pero no he encontrado consulta que lo respalde (sin verificar).

## 3. Total amortizable al año (las dos zonas alquiladas)

| Concepto | Solo 88.000 € | Con ~8.800 € de gastos |
|---|---|---|
| Inmueble (3 %) | 1.603 € | 1.764 € |
| Obra de mejora (3 %), si toda fuera mejora | 420 € | 420 € |
| Mobiliario (10 %) | 444 € | 444 € |
| **Total** | **2.467 €/año** | **2.628 €/año** |

- Si la pintura y la carpintería pasan a reparación, salen 5.310,89 € de la base al 3 %. Eso quita unos 159 €/año de amortización de obra, pero se deducen enteros con el límite de ingresos.
- A partir del año 11 de amortización de los muebles, el total baja unos 444 €/año.

**Ahorro de IRPF de la amortización** = amortización × (1 − reducción) × tipo marginal. Con reducción del 50 %:

| Tipo marginal | Total 2.467 € | Total 2.628 € |
|---|---|---|
| 36,9 % (base liquidable 35.200–60.000 €) | 455 € | 485 € |
| 45 % (base de 60.000 a 300.000 €) | 555 € | 591 € |

- Con una reducción del 85 % o del 100 % (régimen nuevo, ver el punto 5), el ahorro baja mucho, porque casi todo el rendimiento ya queda sin tributar.
- Sin otros ingresos: el rendimiento reducido probablemente queda por debajo del mínimo personal (5.550 € estatal y 5.789 € en Galicia, art. 4 bis del Decreto Legislativo 1/2011). Es una estimación, no un cálculo cerrado.

## 4. Reglas de aplicación (revisadas)

- **Desde cuándo**: se amortiza desde el primer contrato, por la parte alquilada y los días alquilados.
- **Elementos comunes** (inmueble, obra general): 50 % por cada zona alquilada, o el porcentaje real de superficie (ver [[contrato-habitaciones-redaccion-y-fiscalidad]]).
- **Elementos propios** de cada zona (marcados PA/PB): 100 %, solo mientras esa zona está alquilada.
- **Años sin inquilino**:
  - Inmueble: la amortización no deducida no se pierde, porque se sigue amortizando en los años alquilados hasta completar la base.
  - Muebles: tienen un tope de 20 años desde la compra, así que esos años vacíos sí pueden perderse.
- **Al vender**: el valor de adquisición se reduce en la amortización, con la mínima «con independencia de la efectiva consideración de ésta como gasto» (art. 40 del Reglamento, [BOE](https://www.boe.es/buscar/act.php?id=BOE-A-2007-6820)). Por eso hay que deducirla siempre mientras esté alquilado.
- **Justificación**: facturas a nombre del usuario con NIF y pago por banco. Guardarlas mientras se amortice y 4 años más (es buena práctica, por la prescripción).

## 5. Reducción del rendimiento neto: depende de la fecha de firma

El documento decía «50 %, contratos desde el 26-5-2023». Es correcto **solo para contratos firmados hasta el 1-12-2026**.

El Real Decreto-ley 29/2026 cambia el régimen para los firmados desde el 2-12-2026, si se convalida. En un primer alquiler sin ser gran tenedor queda así:
- **100 %** si la renta queda por debajo del índice de referencia.
- **85 %** si los inquilinos tienen entre 18 y 35 años, en proporción a cuántos cumplan la edad.
- **50 %** en el resto de casos.

Detalle y citas en [[fiscalidad-alquiler-por-habitaciones]]. **Decidir la fecha de firma con esto delante.**

El requisito de un contrato de vivienda de la LAU por zona se mantiene. En la redacción se dice «habitación con zona privativa», no «unidad» (ver [[contrato-habitaciones-redaccion-y-fiscalidad]]).

## 6. Deducciones gallegas

**Por obras para alquilar un inmueble vacío: aplicable si se cumplen los plazos.** El texto legal está en [[fiscalidad-alquiler-por-habitaciones]]. Correcciones al documento:

| El documento decía | El texto legal dice |
|---|---|
| El exceso se arrastra 4 años | Lo que pasa de 3.000 €/año va a «los dos ejercicios siguientes». Los 4 años son solo para la cuota insuficiente |
| Obras en ≤ 2 años | Correcto. El texto no dice desde cuándo cuenta; la fecha de fin es la de la factura |
| Si ya presenté la Renta 2025 con obras pagadas ese año, se puede rectificar | **Dudoso.** La deducción se aplica «en el periodo impositivo en que se constituya el arrendamiento». Si los contratos son de 2026, va en la Renta 2026, no en una rectificación de 2025 |
| Pago sin efectivo | No está en el artículo; sí exige factura y declarar el NIF de los proveedores. Pagar por banco sigue siendo lo prudente |
| En vigor desde la Renta 2025 | Sin verificar (el texto consolidado no da la fecha de alta del apartado) |
| — (faltaba) | El valor de referencia se mide a la fecha de devengo **del año en que acaban las obras**, no a 1-1-2025 |
| — (faltaba) | Hace falta alquilar 3 años en total; si un contrato dura menos, se vuelve a alquilar en 6 meses |

- **Máximo**: 450 €/año y 1.350 € en total (15 % de 3.000 € y de 9.000 €, comprobado).
- **Base**: entran las obras de reparación y conservación, no la mejora.
- **Pendiente**: preguntar a la AEAT o a la Xunta si un mismo gasto puede ser a la vez gasto deducible del rendimiento y base de esta deducción.

**500 € por alquilar una vivienda vacía: probablemente no aplica.** Exige que «el precio mensual del alquiler de la vivienda» no supere 700 €, y aquí son 1.120 € por la vivienda entera. Es discutible que se mida por contrato (560 €), porque la deducción es «por vivienda». Lo prudente es darla por no aplicable salvo consulta a la DGT.

**60 % por rehabilitación: no aplica.** Exige obra estructural o actuaciones subvencionadas (art. 41.1 del Reglamento); la reforma es interior. **Correcto.**

**70 % / 90 %**: en el régimen anterior y en el nuevo exigen zona tensionada; Carballo no lo es. **Correcto.**

## 7. Otras vías para bajar el IRPF

Fuera del alquiler. **Sin verificar** en esta revisión:

- **Plan de pensiones**: hasta 1.500 €/año restan de la base. Es diferimiento, porque tributa al rescatarlo.
- **Retribución flexible**:
  - Seguro médico: 500 €/año exentos por persona.
  - Tarjeta restaurante: 11 €/día.
  - Transporte: 1.500 €/año.
  - Formación.

## Imputación si una zona queda vacía (cifras de este piso)

Cálculo con 53.208,03 € × 1,1 % × 50 %: **292,64 € imputados por año entero vacío**, unos 108 € de IRPF al 36,9 %. Con el 2 % serían 532 €.

El 1,1 % vale si el valor catastral de Carballo se revisó con entrada en vigor desde 2012; está pendiente de comprobar en la sede del Catastro.

## Pendiente

- Gastos de compra exactos (escritura y autoliquidación del ITP).
- Fechas de inicio y fin de las obras, y año de pago de cada factura.
- Clasificar cada factura en mejora o reparación.
- Fecha de firma de los contratos: antes o después del 1-12-2026.
- Índice de referencia para Carballo.
- Si el municipio tiene revisión catastral desde 2012.

## Enlaces

- [[apartamentos-calle-uruguay]]
- [[fiscalidad-alquiler-por-habitaciones]]
- [[contrato-habitaciones-redaccion-y-fiscalidad]]
- [[patrimonial]] — donde se seguirán estos importes
- [[real-decreto-ley-29-2026-alquiler]] — síntesis del RDL 29/2026 (régimen de reducción según fecha de firma)
- [[contradiccion-reduccion-alquiler-fecha-inicio]] — fecha de inicio del 50 %: esta nota y la de fiscalidad la citan distinto
- [[_index]]
