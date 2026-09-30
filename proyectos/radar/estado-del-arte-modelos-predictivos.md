---
title: Estado del arte — modelos predictivos de precios e inversión inmobiliaria
created: 2026-09-29
updated: 2026-09-29
tags: [inmobiliario, machine-learning, avm, estado-del-arte]
zona: tecnico
---

Qué arte previo existe —abierto, comercial y académico— para predecir el valor y la revalorización de inmuebles con datos, y qué haría falta para entrenar uno propio. Se escribe antes de extraer nada de las fuentes de Prophero, para no reinventar.

## Qué se pregunta y por qué

La pregunta no es "quién copia a Prophero", sino: **¿está resuelto, en público, predecir con datos el valor y la revalorización de inmuebles, y con qué se entrena?** El objetivo es que el proyecto Radar no parta de cero: aprovechar lo que ya existe y saber qué parte del camino está hecho.

**Estado de verificación**: en esta investigación se han abierto dos fuentes de extremo a extremo — `idealista18` y `ATTOM ResiScore`, marcadas **[LEÍDO]**. Todo lo demás viene de resultados de búsqueda con enlace, sin abrir la fuente — marcado **[SNIPPET]**. No se ha abierto ninguna más.

## La distinción que ordena todo: son dos problemas distintos

Esta es la conclusión principal, y se confunde todo el rato:

- **A) Valoración actual (AVM).** "¿Cuánto vale este inmueble hoy?" Es un campo **maduro**: hay modelos comerciales, datasets abiertos y decenas de repos. Predice un valor *presente*.
- **B) Predicción de revalorización futura.** "¿Qué zona subirá y cuánto, en los próximos 1-3 años?" Es lo que hace Prophero y lo que buscaría Radar. Es un campo **mucho menos abierto**: casi todo es propietario, y lo académico son estudios de caso, no productos.

Un AVM bueno no resuelve B. Confundirlos sería el error de arranque del proyecto.

## 1. Opensource y datasets abiertos

**[LEÍDO]** El hallazgo más útil para España: **idealista18** — [github.com/paezha/idealista18](https://github.com/paezha/idealista18). Un dataset abierto de anuncios reales de Idealista de 2018, **189.923 viviendas** en Madrid (94.815), Barcelona (61.486) y Valencia (33.622), **42 variables** por anuncio (precio, superficie, habitaciones, extras, año de construcción, calidad catastral, distancias a centro/metro, coordenadas), enriquecido con datos del **Catastro**. Licencia **ODbL** (uso libre con atribución). Publicado en *Environment and Planning B* ([paper, DOI 10.1177/23998083241242844](https://journals.sagepub.com/doi/full/10.1177/23998083241242844)).

- **Sus límites, documentados** — y son importantes: son **precios de anuncio, no de transacción**; coordenadas y precios llevan ruido aleatorio introducido a propósito; el año de construcción falta en el **41%** de los anuncios de Madrid; y los barrios están reagrupados por Idealista. Sirve para prototipar, no para medir rentabilidad real.

Otros repos abiertos de valoración (todos **[SNIPPET]**):

| Repo | Qué es |
|---|---|
| [dmai287/real-estate-avm](https://github.com/dmai287/real-estate-avm) | El único que se llama explícitamente AVM opensource; ML + API REST; datos de Chicago, Dallas y Denver |
| [Linhkust/AutoML4RPV](https://www.sciencedirect.com/science/article/abs/pii/S0952197625020433) | AutoML para valoración residencial; validado en Nueva York, Londres y Singapur; código abierto |
| [ferus311/real_estate_project](https://github.com/ferus311/real_estate_project) | Pipeline industrial completo: scrapers, Airflow, Kafka, Hadoop/Spark, Django/React (Vietnam) |
| [newking9088/...](https://github.com/newking9088/real_estate_cost_estimation_and_property_recommendation), [stepkos/...](https://github.com/stepkos/real-estate-price-valuation), [Silvano315/...](https://repos.ecosyste.ms/hosts/GitHub/repositories/Silvano315%2FPrediction-model-for-a-real-estate-market) | Proyectos ML de precio (CatBoost/LightGBM/MLP, regularización) — nivel didáctico |

**Para el componente espacial** hay ecosistema maduro y abierto (**[SNIPPET]**): [FastGWR](https://github.com/Ziqi-Li/FastGWR) (regresión ponderada geográficamente a escala de millones de observaciones), `mgwrsar` y `spgwr` en R, `GWmodelS`, y [gnnwr](https://github.com/zjuwss/gnnwr) en Python (redes neuronales que aprenden la proximidad espacial).

**Competición de referencia**: el [Zillow Prize](https://www.zillow.com/news/building-the-neural-zestimate/), en Kaggle (2017-2019, 1 M$, ~3.700 equipos), cuya solución ganadora superó el modelo propio de Zillow en ~13% y se convirtió en el Zestimate neuronal (**[SNIPPET]**).

## 2. AVM comerciales — valoración actual (problema A)

| Producto | Método declarado | Precisión citada |
|---|---|---|
| **Zillow Zestimate** ([cómo se calcula](https://www.zillow.help/article/how-is-the-zestimate-calculated-zd4402325964563), [neural Zestimate](https://www.zillow.com/news/building-the-neural-zestimate/)) | Red neuronal única nacional (desde 2021), *tiling* geográfico por celdas, descomposición de tiempo en tendencia + estacionalidad, **visión por computador sobre las fotos** | error mediano ~1,9-3,2% (en venta), ~7% (fuera de mercado) |
| **CoreLogic** ([FAQ](https://www.corelogic.com/downloadable-docs/avm-faqs.pdf)) | Mezcla de varios modelos; pesos según influencia local; *confidence score* por *back-testing* | score de confianza por percentil de acierto |
| **HouseCanary** ([brief](https://www.housecanary.com/wp-content/uploads/assets/hc_rental-valuation_technical-brief.pdf)) | ML supervisado, explícitamente **predictivo hacia adelante**; incertidumbre como *Forecast Standard Deviation* | rango + FSD |
| **ATTOM** ([AVM 2.0](https://www.finantrix.com/in-focus/property-as-platform-digital-transformation-real-estate/automated-valuation-models-satellite-imagery)) | 3+ décadas de transacciones, peso a patrones largos (útil donde hay pocas ventas recientes) | error mediano 2,9% |

**[SNIPPET]** todas. Patrón común: se alimentan de **registro público** (registradores/notariado) + datos de anuncios, y devuelven valor + rango + confianza.

## 3. Predicción de revalorización futura (problema B) — lo de Prophero y Radar

Aquí está lo escaso. **[LEÍDO]** el más cercano a Radar: **ATTOM ResiScore** ([attomdata.com](https://www.attomdata.com/solutions/ai-powered/market-location-analytics/resiscore/)) — asigna a cada **censo-tract** de EE.UU. una puntuación percentil 1-100 **dentro de su área metropolitana**, según el rendimiento de mercado esperado **a 24 meses**. Combina tendencia, revalorización, aceleración, fuerza de la previsión y volatilidad en un único score, refrescado mensualmente, entrenado sobre "cientos de millones de transacciones". **Es propietario, solo EE.UU., y de pago.**

Enfoques académicos de lo mismo (**[SNIPPET]**):

- **Viena** — XGBoost sobre 83.527 transacciones a 10 años con variables sociodemográficas y geográficas: **~15% de error (MAPE) a 1 año y <20% a 3 años**. Separar obra nueva de segunda mano bajaba el error ~6 puntos ([paper](https://agile-giss.copernicus.org/articles/7/7/2026/agile-giss-7-7-2026.pdf)).
- **Londres** — XGBoost + SHAP; los factores de barrio (estaciones, supermercados, paradas de bus) aportaron ~97% de las explicaciones.
- **Predicción de *gentrificación*** (cambio de barrio al alza): sistemas de alerta temprana en [Buffalo](https://ar5iv.labs.arxiv.org/html/2111.14915), [Sídney](https://par.nsf.gov//servlets/purl/10508191) (gradient boosting, R²=0,938 sobre índice socioeconómico), [Washington DC](https://dataspace.princeton.edu/handle/88435/dsp01b8515q81p) (SVM), [Taipéi](https://www.sciencedirect.com/science/article/abs/pii/S0264275125004962) (XGBoost + autocorrelación espacial). Es la literatura metodológicamente más parecida a "predecir dónde subirá".
- **LLM para cambio de barrio** — uso de Llama 3.3 para puntuar anuncios de Airbnb como señal de gentrificación ([GISRUK 2025](https://zenodo.org/records/15231204/files/gisruk2025-llm-airbnb.pdf)).

## 4. España: datos oficiales, competidores y método

**Índices oficiales de revalorización** (**[SNIPPET]**) — son la referencia contra la que medir cualquier modelo propio:
- **Registradores — IPVVR**, índice de **ventas repetidas** (metodología Case-Shiller: solo viviendas vendidas ≥2 veces; base 2005; excluye obra nueva) — [opendata.registradores.org](https://opendata.registradores.org/documents/33383/148213/ERI_3T_2019.pdf).
- **Banco de España — índice experimental de ventas repetidas** para España y seis capitales (2007-2024, datos del Notariado) ([Informe Anual 2025](https://www.bde.es/f/webbe/SES/Secciones/Publicaciones/PublicacionesAnuales/InformesAnuales/25/InfAnual_2025_Recuadros.pdf)).
- **INE — IPV** (precio de vivienda), vía su API.

**Competidores directos en España** (**[SNIPPET]**): **Tiko** (iBuyer, con su propio algoritmo **Tikoanalytics™** que valora en 24 h sin ver el inmueble; compró Housell en 2024), **Clikalia**, **Kodit.io** (iBuyers); **Inversiva** y **Monest Capital** (inversión delegada, competidores declarados de Prophero). Los iBuyers ofrecen típicamente **8-10% por debajo** de mercado ([El Español](https://www.elespanol.com/invertia/empresas/inmobiliario/20190614/ibuyers-plataformas-digitales-vender-casa-pocos-dias/406210939_0.amp.html)).

**Acceso a los datos** (**[SNIPPET]**): Idealista es el único gran portal español con **API oficial** (JSON, filtros, OAuth, acceso gratuito solo para proyectos académicos y no garantizado), pero sus términos **prohíben expresamente el scraping** y su `robots.txt` bloquea las rutas de búsqueda; los intentos devuelven 403/429/CAPTCHA ([términos de Idealista](https://st1.idealista.com/data/site/assets/terms-and-conditions-es.CKezoQnD.pdf)). Alternativas: `CatastRo` (R) para Catastro, API del INE, [datos.gob.es](https://datos.gob.es/es/blog/datos-abiertos-para-conocer-mejor-la-situacion-de-la-vivienda-en-espana).

## 5. Metodología académica: qué modelos y qué variables

**[SNIPPET]** Revisiones sistemáticas ([Pita et al., *Computational Economics*](https://dlnext.acm.org/doi/10.1007/s10614-025-10983-4), [revisión multimodal, Springer](https://rd.springer.com/article/10.1007/s41060-026-01204-8)):

- **Modelos que ganan**: *gradient boosting* (XGBoost, LightGBM) y Random Forest son los más usados y los que mejor rinden; las redes neuronales aparecen donde hay mucho dato; los híbridos con regresión hedónica mejoran interpretabilidad.
- **Variables que más pesan**: **localización** (la número uno), superficie, número de habitaciones/baños, y calidad del edificio. Emergentes: imágenes de satélite, datos de verde urbano, grafos, datos multimodales.
- **Hallazgo incómodo**: no se encontró relación clara entre la calidad del modelo y la cantidad de datos o el algoritmo — depende sobre todo de la base de datos y de la zona.

## 6. Qué datos hacen falta — lista operativa

Cruzando todo lo anterior, para replicar algo tipo Radar en España:

| Bloque | Datos | Dónde |
|---|---|---|
| Precio real | Precios de **transacción** (no anuncio) | Registradores / Notariado |
| Inmueble | Superficie, habitaciones, año, calidad | **Catastro**, portales |
| Localización fina | Coordenadas, celdas/tiles, barrio | Geocoding, idealista18 |
| Zona | Población, renta, paro, esfuerzo | INE, padrón, SEPE, AEAT |
| Adelantados | **Visados de obra**, suelo disponible, logística/industrial | Colegios de arquitectos, Catastro |
| Macro | Tipos de interés, PIB | Banco de España, INE |
| Opcional | Imágenes/satélite | Proveedores |

## 7. Cómo se entrena y se valida — las reglas que importan

**[SNIPPET]**, de literatura de validación:

1. **Validación temporal obligatoria.** El error número uno es partir los datos al azar: con datos cronológicos eso filtra información del futuro y **infla la precisión** (casos con R² 0,84 que caían al revalidar en temporal). Se entrena con el pasado y se prueba con el periodo siguiente (*TimeSeriesSplit* / *forward-chaining*).
2. **Cuidado con el *target encoding*** de variables de alta cardinalidad (municipio): hay que hacerlo dentro del *fold* para no filtrar.
3. **Los árboles no extrapolan.** No pueden anticipar un cambio de mercado que no estaba en los datos de entrenamiento; tienden a **subestimar en mercados que suben** y a sobreestimar en los que bajan.
4. **Interpretabilidad con SHAP** para no quedarse en la caja negra.
5. **Métricas**: MAPE/MAE/RMSE, y siempre con el punto de referencia oficial (Banco de España / Registradores), no con una media propia.

## Considerado y descartado

- **Blogs y guías de "cómo invertir en inmuebles"**: sin método reproducible, descartados.
- **Repos didácticos** (`bilalahmed251`, `Silvano315`, etc.): válidos como ejemplo de *pipeline*, no como base — usan datasets de California o de juguete, no España.
- **Reproducciones del caso AWS de Prophero** (AI Singapore, HKU, ZenLayer, otros): no aportan al estado del arte, son el mismo texto.
- **Clikalia**: aparece como iBuyer pero no se ha conseguido documentación de su algoritmo; queda como competidor sin método público.

## Recomendaciones

1. **Separar desde el principio los dos problemas.** Un AVM (valorar hoy) se resuelve con lo abierto que ya existe. Predecir revalorización futura por zona (lo de Radar) es el problema difícil, y casi todo es propietario.
2. **Arrancar con `idealista18`** como dataset español para prototipar el *pipeline* y la parte espacial, asumiendo sus límites (precios de anuncio, ruido, 41% sin año en Madrid).
3. **Modelo base**: XGBoost/LightGBM + variables espaciales (GWR o tiles) + validación temporal estricta. Es donde converge toda la literatura.
4. **Contrastar siempre contra el índice oficial** (Banco de España / Registradores ventas repetidas), que es el único patrón objetivo de "revalorización" en España.
5. **Leer el caso Zillow antes de fiarse del modelo** ([Bloomberg](https://www.bloomberg.com/news/articles/2021-11-08/zillow-z-home-flipping-experiment-doomed-by-tech-algorithms), [Stanford GSB](https://www.gsb.stanford.edu/insights/flip-flop-why-zillows-algorithmic-home-buying-venture-imploded)): su algoritmo de valoración no era malo, y aun así perdió ~421 M$ en un trimestre comprando a los precios que predecía. Acertar el valor y ganar dinero **no son lo mismo** — el modelo puede estar bien y el uso que se le da, mal.

## Dónde se ha buscado

4 rondas de búsqueda web. Tipos de fuente cubiertos: **repositorios GitHub** · **papers y revisiones académicas** (ScienceDirect, Springer, ACM, Zenodo, NSF) · **documentación de proveedor** (Zillow, CoreLogic, HouseCanary, ATTOM, idealista18) · **prensa especializada** (Bloomberg, Stanford GSB, El Español) · **foros**. Los enlaces que no aportaron nada (repos didácticos, copias del caso AWS) están citados arriba como descartes.

## Huecos

- **No se ha encontrado ningún modelo abierto que prediga revalorización a nivel municipal en España.** Lo más cercano es ATTOM ResiScore (propietario, EE.UU.). El hueco es real.
- **No se han abierto fuentes más allá de las dos marcadas [LEÍDO]**; la mayoría del informe se apoya en resúmenes de búsqueda con enlace, no en lectura directa.
- **No se ha buscado** en bases académicas de pago (Scopus/Web of Science) ni en el registro de patentes, donde podría haber AVM propietarios documentados.

## Relacionado en el wiki

- [[radar]] — hub del proyecto: qué hace Prophero y qué dicen sus transcripciones
- [[fuentes-prophero]] — catálogo de fuentes de Prophero, pendientes de extraer
- [[sintesis-radar]] — síntesis consolidada de todo el conocimiento del proyecto
