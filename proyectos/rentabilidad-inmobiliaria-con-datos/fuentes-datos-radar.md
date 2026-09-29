---
title: Fuentes de datos para Radar — variable a variable, comprobadas con A Coruña
created: 2026-09-29
updated: 2026-09-29
tags: [inmobiliario, datos-abiertos, api, fuentes]
zona: tecnico
---

Dónde se puede conseguir, en España, cada dato que las transcripciones de Prophero señalan como relevante — con el tipo de acceso (API, dataset, scraping), el coste, y si se ha comprobado de verdad con A Coruña.

## Qué se necesitaba y para qué

Las variables salen de las transcripciones (ver [[rentabilidad-inmobiliaria-con-datos]]): **población, tasa de paro, renta per cápita, tasa de esfuerzo, precio de vivienda, precio de alquiler**, más los indicadores adelantados (**visados de obra nueva**, suelo). Y para el cálculo de rentabilidad: **transacciones, valor catastral, IBI**.

**Criterio**: fuente real y accesible, gratis primero; si es de pago, con precio. **Comprobada con A Coruña.** Y con **serie histórica**: un modelo no se entrena con el dato de un año, sino con la progresión completa — de ahí la sección de profundidad histórica más abajo.

**Etiquetas de estado**:
- **[VERIFICADO EN VIVO]** — he llamado a la fuente con A Coruña y ha devuelto el dato. Se enseña el valor.
- **[FUENTE LOCALIZADA]** — la fuente existe y es la correcta, pero no he extraído el valor de A Coruña.
- **[SNIPPET]** — aparece en búsqueda, sin abrir.

## Tabla resumen (variable → fuente)

| Variable | Fuente principal | Acceso | Coste | A Coruña |
|---|---|---|---|---|
| **Población (municipio)** | INE — tabla `29005` | API JSON | Gratis | **[VERIFICADO EN VIVO]** 251.543 (2025) |
| **Tasa de paro / demandantes** | SEPE — datos abiertos municipio | CSV | Gratis | **[VERIFICADO EN VIVO]** 30.710 (ene-2013, cód. 15030) |
| **Datos del inmueble** (superficie, año, uso) | Catastro — servicios web | REST/SOAP | Gratis | **[VERIFICADO EN VIVO]** el servicio responde |
| **Datos por concello de Galicia** | IGE — API de táboas | API JSON/CSV | Gratis | **[FUENTE LOCALIZADA]** (provincia sí; concello por código de espazo) |
| **Renta per cápita (municipio)** | INE — operación `ADRH` | API JSON | Gratis | **[FUENTE LOCALIZADA]** — no extraído |
| **Precio de alquiler (municipio)** | MIVAU — **SERPAVI** | Descarga Excel + visor | Gratis | **[FUENTE LOCALIZADA]** — no extraído |
| **Precio de vivienda** | INE `IPV` (provincial) / portales (municipal) | API / scraping | Gratis / pago | [SNIPPET] |
| **Visados de obra nueva** | MITMS (provincial); IBESTAT, ICANE (municipal en su CCAA) | Excel | Gratis | [SNIPPET] |
| **Anuncios (precio, stock, tiempo)** | Idealista (API limitada / scrapers) | API / pago | Gratis-limitado / 0,4-10 $ por 1.000 | [SNIPPET] |
| **Tasa de esfuerzo** | *derivada* (precio ÷ renta) | cálculo propio | — | — |

## Profundidad histórica — lo que de verdad decide el proyecto

Un dato suelto del año actual **no sirve para un modelo**: hace falta la serie completa, para ver la progresión y para entrenar. Esto es hasta dónde llega cada fuente, **comprobado llamándola**:

| Variable | Fuente | Serie histórica (verificado en vivo) |
|---|---|---|
| **Población** | INE `29005` | **1996 → 2025** (29 años) para A Coruña. Hay además una operación aparte con **1986-1995** |
| **Población (Galicia)** | IGE | **1900 → 2025** |
| **Paro municipal** | SEPE | **2006 → 2025** — en enero de 2006 A Coruña tenía **19.190** demandantes, frente a 30.710 en 2013 |
| **Alquiler** | SERPAVI | **2011 → 2024** (declarado por el Ministerio; descarga en Excel) |
| **Precio vivienda (capitales)** | Banco de España / Registradores | **2007-2024** (Banco de España) y **1995-2010** (IPVVR, base 2005) |
| **Renta municipal** | INE `ADRH` | por confirmar (el municipal arranca hacia 2015) |

**Lectura**: la base gratuita **sí tiene historia suficiente para entrenar**. Población desde 1996, paro desde 2006, alquiler desde 2011. Ocho a treinta años por variable. Lo que no la tiene es el precio de vivienda municipal y los visados — los dos huecos, que además son cortos en el tiempo.

## Detalle por variable

### Población — INE, [VERIFICADO EN VIVO]

La **API JSON de INEbase**, gratuita y documentada con Swagger en [ine.es/dyngs/DAB](https://www.ine.es/dyngs/DAB/index.htm?cid=1099). La operación de población municipal es **`DPOP`** ("Cifras Oficiales de Población de los Municipios Españoles"), y la tabla de municipios es la **`29005`**.

Llamada real ejecutada:
`https://servicios.ine.es/wstempus/js/ES/DATOS_TABLA/29005?nult=1`

Devolvió, para **"Coruña, A"**, año **2025**: **251.543 habitantes** (116.606 hombres + 134.937 mujeres). Sin registro ni clave. Esta misma API da también `IPV` (precio de vivienda) y `ADRH` (renta).

### Paro — SEPE, [VERIFICADO EN VIVO]

Descarga directa en **CSV** desde datos.gob.es ([conjunto EA0041513](https://datos.gob.es/es/catalogo/ea0041513-paro-registrado-por-municipios)) y en la sede del SEPE. **No hay API REST**: es descarga de fichero, pero es un CSV limpio y gratuito.

Fichero real descargado (`Dtes_empleo_por_municipios_2013_csv.csv`): la línea del municipio **A Coruña** (código **15030**) para enero de 2013 da **30.710** demandantes de empleo. Hay ficheros anuales **al menos de 2013 a 2025** (comprobados los dos extremos: los cinco años probados devuelven HTTP 200). Aviso: desde 2022 se enmascaran los valores entre 1 y 4 como "<5" por protección de datos.

### Datos del inmueble y geolocalización — Catastro, [VERIFICADO EN VIVO]

**Servicios web gratuitos** de la Dirección General del Catastro ([documentación](https://www.catastro.minhap.es/ws/Webservices_Libres.pdf)), SOAP y REST, sin registro para los datos **no protegidos** (callejero, datos del inmueble salvo titularidad y valor catastral, y conversor de coordenadas). El **valor catastral** y la titularidad son la parte restringida.

Comprobado en vivo: `OVCCallejero.asmx/ConsultaMunicipio?Provincia=A Coruña&Municipio=A Coruña` devuelve el municipio con código **15/30**. Para geolocalizar, el geocoder **Cartociudad** ([github.com/IDEESpain/Cartociudad](https://github.com/IDEESpain/Cartociudad)) acepta referencia catastral y devuelve JSON.

### Galicia — IGE, [FUENTE LOCALIZADA]

El **Instituto Galego de Estatística** tiene API propia ([ige.gal/web/mostrar_paxina.jsp?paxina=004015](https://www.ige.gal/web/mostrar_paxina.jsp?paxina=004015)), gratuita, licencia **CC BY-SA 4.0**, con descarga en **CSV/JSON/JSON-stat**:

`https://www.ige.gal/igebdt/igeapi/datos/{código-de-tabla}`

Comprobado en vivo con la tabla `1552` (población): **devuelve datos**, y para A Coruña aparece la **provincia** (código 15). El **concello** requiere el código de espacio municipal, que se obtiene configurando la consulta en su web y copiando la URL. Hay paquete de R (`igebaser`) como puente. Muy útil porque da desagregación por **concello y comarca** que el INE no da con tanta facilidad.

### Renta — INE `ADRH`, [FUENTE LOCALIZADA]

El **Atlas de Distribución de Renta de los Hogares** del INE está en la misma API JSON, operación **`ADRH`**, con tablas **por municipio** (confirmado: aparecen municipios concretos como Abengibre o l'Atzúbia). **No extraje el valor de A Coruña**: la operación tiene cientos de tablas y no localicé cuál corresponde a este municipio en el tiempo disponible. Es accesible y gratis; queda como verificación pendiente.

### Alquiler — SERPAVI, [FUENTE LOCALIZADA]

El **Sistema Estatal de Referencia del Precio del Alquiler** ([mivau.gob.es](https://www.mivau.gob.es/vivienda/alquila-bien-es-tu-derecho/serpavi)), la fuente oficial de precios de alquiler, elaborada sobre **más de 2,5 millones de arrendamientos anuales** de fuentes tributarias y catastrales. Da **sección censal, distrito, municipio, provincia y CCAA**, con mediana, percentil 25 y 75 y número de testigos. Se puede **descargar la base completa 2011-2024 en Excel** y las capas en shapefile. El visor ([serpavi.mivau.gob.es](https://serpavi.mivau.gob.es/)) responde (HTTP 200). **Gratis.** No extraje el valor de A Coruña. Es, con diferencia, la mejor fuente de alquiler del país — muy superior a lo que ofrecen los portales.

### Precio de vivienda — [SNIPPET]

El **INE `IPV`** es provincial. A nivel municipal no hay índice oficial gratuito; se cubre con portales (ver Idealista) o con el índice de **ventas repetidas** del Banco de España / Registradores para capitales ([ver el informe de estado del arte](estado-del-arte-modelos-predictivos.md)).

### Visados de obra nueva — [SNIPPET]

El **MITMS** publica la estadística de visados en Excel **por provincia y CCAA**, sin detalle municipal. El desglose **municipal** solo está en algunas comunidades que lo publican: **IBESTAT** (Baleares, mensual por municipio) e **ICANE** (Cantabria, multiformato). Para Galicia, habría que mirar el **COAG** (Colegio Oficial de Arquitectos de Galicia) o el IGE. Queda como hueco.

### Anuncios y micro-dato — Idealista

La **API oficial** es la única de los grandes portales españoles ([Octoparse](https://www.octoparse.es/blog/api-datos-inmobiliarios-vs-web-scraping-ia)): JSON, OAuth 2.0, gratis para proyectos **académicos y no garantizado**, con límite citado de **100 consultas/mes y 50 resultados por consulta**; la profesional se negocia caso a caso. Su **scraping está prohibido** por términos y `robots.txt`, con bloqueos 403/429/CAPTCHA. Alternativas de pago si hace falta volumen: **Apify** desde **0,40 $/1.000 resultados**, Decodo (planes 19-99 $, plan gratis 2.000 peticiones), Parse.bot.

## Considerado y descartado

- **Scrapers de pago para datos que ya son gratis** (población, paro, catastro): descartados — el INE, el SEPE y el Catastro los dan sin coste y sin riesgo legal.
- **Scraping de Idealista**: descartado como vía principal por sus términos y por los bloqueos; solo como último recurso y con la API oficial por delante.
- **APIs institucionales SCSP del INE/SEPE** (certificados de padrón, situación de desempleo): son para administraciones, con consentimiento, no para datos agregados. Descartadas.
- **Valor catastral**: es dato **protegido**, no sale por los servicios libres. Para estimarlo, el propio Catastro usa modelos. Se depende del portal.

## Recomendaciones

1. **La base gratuita ya cubre casi todo**: INE (API) para población, renta y precio provincial · SEPE (CSV) para paro municipal · Catastro (servicios web) para el inmueble · **SERPAVI** para alquiler · **IGE** para el detalle gallego. Todo gratis.
2. **SERPAVI es la joya** para el cálculo de rentabilidad: precio de alquiler oficial por municipio y sección censal, gratis.
3. **Los dos huecos reales** son: **precio de vivienda a nivel municipal** (no hay índice oficial gratuito) y **visados de obra nueva municipales** (solo algunas CCAA). Son justo los indicadores adelantados que Prophero usa.
4. **No hace falta scraping** para arrancar. Solo entraría para micro-dato de anuncios concretos, y ahí la vía es la API de Idealista, no el scraping.

## Dónde se ha buscado

Búsquedas web (INE, SEPE, Catastro, IGE, SERPAVI, Idealista) **más pruebas en vivo** con `curl`: INE API (población extraída), SEPE (CSV descargado y filtrado), Catastro (consulta ejecutada), IGE (API ejecutada), SERPAVI (visor), y comprobación de los años disponibles del SEPE.

## Lo que queda pendiente de comprobar

- **Valor concreto de A Coruña** para **renta** (INE `ADRH`) y **alquiler** (SERPAVI): fuente localizada y gratis, valor no extraído.
- **Concello de A Coruña en el IGE** (código de espacio municipal).
- **Visados de obra nueva** para A Coruña concretamente (fuente autonómica sin identificar).
- **Precio de vivienda municipal**: sin fuente oficial gratuita encontrada.

## Relacionado en el wiki

- [[rentabilidad-inmobiliaria-con-datos]] — hub: las variables que se quieren cubrir y de dónde salen
- [[estado-del-arte-modelos-predictivos]] — con qué se entrena el modelo y cómo se valida
- [[fuentes-prophero]] — catálogo de fuentes sobre Prophero
