---
title: Síntesis Radar — todo el conocimiento del proyecto hasta la fecha
created: 2026-09-29
updated: 2026-09-29
tags: [inmobiliario, radar, sintesis, datos, machine-learning]
zona: tecnico
---

Todo lo que el proyecto sabe a día de hoy, consolidado en un solo documento y orientado a construir Radar: el método de Prophero, el estado del arte, las fuentes de datos reales para España y la evidencia de los casos.

## Qué es Radar y qué busca

Un sistema que, con datos, identifique dónde merece la pena invertir en vivienda en España — replicando el "buscador" de Prophero. La pregunta es **predecir con datos la revalorización y la rentabilidad de un inmueble o de una zona**, y para eso hay que saber tres cosas: qué mide, con qué datos, y cómo se entrena. Las tres están abajo.

**Fuera de este documento, a propósito**: comisión de Prophero, precios, opiniones de clientes, financiación corporativa, plantilla comercial. Es información sobre la empresa como vendedor, no insumo para construir Radar. Si algo de esto hiciera falta, se dice.

## 1. El método, según Prophero (de las 8 transcripciones)

Todo lo de esta sección sale de las transcripciones ya ingeridas. Detalle con cita y línea: [[radar]].

### 1.1 Las variables que dicen usar

Por municipio y trimestre, con histórico de 12-16 años:

- **Población** — crecimiento a 1 y 5 años
- **Tasa de paro** — evolución a 3 y 5 años
- **Renta per cápita** — comparada con la media nacional
- **Tasa de esfuerzo**
- **Precio medio de la vivienda**
- **Precio medio del alquiler**

Además, para el modelo "reactivo" original: demografía, renta, **impagos, ocupación, liquidez de venta y liquidez de alquiler**.

### 1.2 Indicadores adelantados — la parte predictiva

Lo que anticipa el crecimiento *antes* de que el precio reaccione:

- **Visados de obra nueva** en el municipio
- **Desarrollo de suelo logístico e industrial**

Descartan expresamente los **centros de datos** como indicador, porque no generan empleo.

### 1.3 El criterio de decisión (scoring)

**Versión numérica** (la más completa):
- Filtro 1 — **top 10% de municipios por rentabilidad neta**, con umbral que baja con los años: 8% → 6% → 4-5%
- Filtro 2 — **top 10% de municipios por revalorización esperada (>10%)**
- Intersección → **el 5% de los 8.000 municipios de España**

**Versión simple** (otra fuente, sin cifras): 1) crece la población · 2) renta per cápita / tasa de desempleo · 3) un **subyacente concreto** que lo justifique (fábrica, centro logístico, conexión).

**Versión ordenada paso a paso**: población del municipio *y de su comarca* → industria que se genera alrededor → renta per cápita frente a la media → subyacente concreto.

### 1.4 Rentabilidad neta — cómo la definen

Descuenta **IBI, gastos de comunidad, seguro, mantenimiento y gestión**. La neta sale **30-40% por debajo** de la bruta. Y la sensibilidad, que es clave para el modelo: **el alquiler pesa mucho más que el precio** — 100 € menos de alquiler quitan un punto de rentabilidad; comprar 5.000 € más caro solo quita 0,3.

### 1.5 Las fases de entrada a una zona

- **Por riesgo/ticket**: pionero (riesgo alto, revalorización alta, ticket bajo, poca liquidez de alquiler) → consolidación → zona madura (riesgo bajo, revalorización baja). Sagunto es su ejemplo de "ya madura"; Carlet, de zona pionera que descartaron para un primerizo.
- **Por tipo de producto**: piso suelto → cambio de uso de local → edificio vandalizado → obra nueva.

### 1.6 El límite que reconocen

> "La data no te permite acertar siempre a dónde sí que hay que ir, te permite eliminar dónde seguro que no hay que ir."

Y admiten que parte de su ventaja **no** es data: en Oliva entraron por el aviso de un socio local sobre una autovía nueva, y un pueblo (Moncofar) lo descubrieron por una conversación, no por el modelo.

## 2. Qué datos hacen falta y de dónde sacarlos en España

Variable → fuente real, acceso, coste y **serie histórica**. Detalle completo y llamadas en vivo: [[fuentes-datos-radar]].

| Variable | Fuente | Acceso | Coste | Histórico | A Coruña |
|---|---|---|---|---|---|
| Población | INE (tabla `29005`) | API JSON | Gratis | **1996→2025** (+1986-1995) | ✅ 251.543 (2025) |
| Paro municipal | SEPE | CSV | Gratis | **2006→2025** | ✅ 30.710 (2013) |
| Datos del inmueble | Catastro | REST/SOAP | Gratis | — | ✅ responde |
| Renta per cápita | INE (`ADRH`) | API JSON | Gratis | por confirmar (~2015→) | localizada, no extraída |
| **Alquiler** | **SERPAVI** (MIVAU) | Excel + visor | Gratis | **2011→2024** | localizada, no extraída |
| Precio vivienda (provincia) | INE (`IPV`) | API | Gratis | 2007→ | — |
| Precio (capitales, serie larga) | Banco de España / Registradores | Excel | Gratis | 1995/2007→2024 | — |
| Datos de Galicia por concello | IGE | API | Gratis | **1900→2025** | API funciona |
| Visados de obra nueva | MITMS (provincial) / IBESTAT, ICANE (municipal) | Excel | Gratis | — | ❌ no encontrado para Galicia |
| Anuncios (stock, tiempo, precio) | Idealista | API limitada / scrapers | Gratis-limitado / 0,4-10 $ | — | — |
| Tasa de esfuerzo | *derivada* (precio ÷ renta) | cálculo propio | — | — | — |

**Lo verificado en vivo** frente a lo solo localizado está marcado en el documento fuente. Lo gratis cubre casi todo. **SERPAVI** es la mejor fuente de alquiler del país: más de 2,5 millones de contratos reales al año, precio por municipio y hasta sección censal.

## 3. El estado del arte: cómo se construye un modelo así

Detalle, opciones y enlaces: [[estado-del-arte-modelos-predictivos]].

### 3.1 La distinción que ordena todo

- **Valorar hoy (AVM)** — campo **maduro**: Zillow, CoreLogic, HouseCanary, ATTOM, y repos opensource. Resuelto.
- **Predecir la revalorización futura por zona** — lo de Prophero y Radar. Campo **casi todo propietario**. Un AVM bueno **no** lo resuelve.

### 3.2 Opensource y datasets reutilizables

- **[idealista18](https://github.com/paezha/idealista18)** — dataset español abierto y verificado: **189.923 viviendas**, Madrid/Barcelona/Valencia 2018, 42 variables, cruzado con Catastro, licencia ODbL. Límites: precios de **anuncio** (no transacción), con ruido, y falta el año de construcción en el 41% de Madrid.
- **[dmai287/real-estate-avm](https://github.com/dmai287/real-estate-avm)**, **[AutoML4RPV](https://www.sciencedirect.com/science/article/abs/pii/S0952197625020433)**, **[ferus311/real_estate_project](https://github.com/ferus311/real_estate_project)** (pipeline industrial completo).
- **Espacial**: [FastGWR](https://github.com/Ziqi-Li/FastGWR), `mgwrsar`, `spgwr`, [gnnwr](https://github.com/zjuwss/gnnwr).

### 3.3 Lo comercial de referencia

**ATTOM ResiScore** — lo más parecido a Radar que existe: puntúa cada zona de EE.UU. del 1 al 100 según el rendimiento esperado **a 24 meses**. Cerrado y de pago. En España **no hay nada equivalente abierto**.

### 3.4 Modelos y variables que ganan

- **Modelos**: gradient boosting (XGBoost, LightGBM) y Random Forest; redes neuronales donde hay mucho dato; híbridos con regresión hedónica para interpretabilidad.
- **Variables**: localización (la número uno), superficie, habitaciones, calidad del edificio.
- **Hallazgo incómodo**: no hay relación clara entre calidad del modelo y cantidad de datos o algoritmo elegido.

### 3.5 Cómo se valida (el error que hunde a casi todos)

1. **Validación temporal obligatoria** — partir los datos al azar filtra el futuro e infla la precisión (casos con R² 0,84 que se caen al revalidar en el tiempo).
2. **Cuidado con el *target encoding*** de municipio: dentro del *fold*.
3. **Los árboles no extrapolan** — subestiman en mercados que suben.
4. **SHAP** para interpretabilidad. **Métricas**: MAPE/MAE/RMSE, contra el índice oficial.

## 4. La evidencia: los casos de Prophero, con cifras

Sirven como banco de validación — sitios donde dicen haber acertado (o fallado).

**Sagunto (el caso insignia, mejor documentado)** — serie de precios del mismo tipo de piso: ~25-31.000 € (antes) → 48-52.000 € al entrar Prophero → **125-130.000 € hoy**. Subyacente: gigafactoría de baterías de **Volkswagen** (12.000 empleos), centro logístico de Mercadona e Inditex. Corroborado en varias transcripciones.

| Lugar | Cifra que citan |
|---|---|
| Ocaña (Toledo) | 8.000→14.000 hab.; piso a 60.000 €, alquiler 500→800 €/mes, renta neta 10-11% |
| Almazora (Castellón) | +4% anual de población; 88 pisos obra nueva vendidos en 7 días |
| Soyana (Valencia) | 5.000 hab.; predicen 26% de revalorización |
| Puçol (Valencia) | primer piso en España: 50-55.000 € → ~140.000 € |
| Moncada (Valencia) | local + reforma ~800.000 € → 8 viviendas de ~1,2 M€ |
| Toledo | pisos a 50.000 € → 110.000 € en 3 años |
| Horta Nord (Valencia) | 40.000 € → 150.000 € en 7 años |
| Molina de Segura (Murcia) | 70.000→76.100 hab.; renta 21.978→23.600 € |

**Contraejemplos que descartan** (útil para las features negativas): **Linares** (mayor paro de España, cerró Land Rover), **Jaén** (rentabilidad alta pero por precio bajo, no por crecimiento), **Zaragoza** (exceso de suelo), **Cataluña**, **Rafael/Buñol** (por lentitud burocrática, no de mercado).

**Zonas que declaran**: corredor mediterráneo (Castellón, Valencia, Alicante, Murcia), Toledo Norte/sur de Madrid, Zaragoza, Jerez, Granada, Sevilla, Cádiz, País Vasco.

## 5. Los huecos

1. **Precio de vivienda a nivel municipal** — no hay índice oficial gratuito (el INE solo llega a provincia). **Es el hueco que puede romper el proyecto**, porque la revalorización es justo el objetivo a predecir.
2. **Visados de obra nueva municipales** — solo Baleares y Cantabria los publican; **para Galicia no se han encontrado**.
3. **El modelo de scoring de Prophero no está explicado técnicamente en público** — lo único técnico publicado (caso de AWS) describe su *chatbot*, no el modelo que puntúa municipios.

## 6. Contradicciones que siguen abiertas

- **Umbral de rentabilidad neta**: 4-5% / 5-6% / 6% / "8% hace años" — sin secuencia temporal limpia con las 6 fuentes.
- **Población de partida de Ocaña**: 8.000 / 9.000 / 10.000.
- **"200 millones de datos / 500 variables"**: su propia aritmética no cuadra.
- **Método de cálculo de la revalorización propia**: 13,7% vs ">15%" — no se sabe si es anualizado o acumulado del cohorte.

## 7. Cómo leer la cronología de las transcripciones

Las fuentes son de momentos distintos (**~2022-23 → 2024 → 2025 → septiembre 2026**), así que parte de lo que parecían contradicciones numéricas es **evolución en el tiempo**, no error. El caso resuelto: el umbral de rentabilidad neta baja 8% → 6% → 4-5% en orden perfecto con las fechas.

## Relacionado en el wiki

- [[radar]] — hub: extracción completa de las transcripciones, con cita y línea
- [[fuentes-datos-radar]] — variable a variable: fuente, acceso, coste, histórico y comprobación con A Coruña
- [[estado-del-arte-modelos-predictivos]] — arte previo, modelos y validación
- [[fuentes-prophero]] — catálogo de fuentes externas sobre Prophero, pendientes de extraer
