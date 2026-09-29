---
title: Fuentes sobre PropHero — catálogo de material revisado y por revisar
created: 2026-09-29
updated: 2026-09-29
tags: [inmobiliario, prophero, fuentes, datos]
zona: tecnico
---

Catálogo de fuentes externas del proyecto Radar: qué se ha revisado ya, qué se ha encontrado pendiente de revisar, y qué se ha descartado. Sirve para no duplicar trabajo y para saber dónde queda hueco.

## Para qué es este documento

El objetivo del proyecto es averiguar **cómo construye y usa Prophero su sistema de datos** — no sus resultados ni su reputación. Este documento no analiza el contenido: inventaría las fuentes y dice, de cada una, si habla o no de la construcción de los datos y en qué estado de verificación está.

Dos etiquetas de estado, distintas de las del hub:

- **[LEÍDO]** — abierto y leído en esta sesión, contenido confirmado.
- **[SNIPPET]** — aparece en resultados de búsqueda con URL, pero **no se ha abierto todavía**. Lo que se dice de su contenido viene del resumen del buscador, no de la fuente. Hay que abrirlo antes de citarlo.

## A. Ya revisado — las 8 transcripciones de vídeo

Las lleva otro modelo, no este documento. Aquí solo se inventan para **no volver a traerlas** en las búsquedas: son títulos de vídeos de YouTube ya guardados en `fuentes/prophero-transcripciones-2026-09-29/`.

| Fichero | Título del vídeo |
|---|---|
| `tengo 1 plan 2/...txt` | Experto en Vivienda Revela Cuando Explotará la Burbuja Inmobiliaria "Haz Esto con tu Dinero" |
| `tengo 1 plan 1/...txt` | 2 Expertos en inversión: Si Ganas entre 1.000 y 3.000€ haz Esto para Crear Riqueza |
| `otros/Cómo PropHero encuentra las zonas con mayor potencial de crecimiento antes que nadie.txt` | Cómo PropHero encuentra las zonas con mayor potencial de crecimiento antes que nadie |
| `otros/Cómo Invertir tu Dinero hoy Vivienda, rentabilidad y oportunidades PropHero @ac2ality.txt` | Cómo Invertir tu Dinero hoy: Vivienda, rentabilidad y oportunidades PropHero (@ac2ality) |
| `otros/Experto Inmobiliario ¡El Método SECRETO Para Multiplicar tu Patrimonio Sin Pedir Hipotecas!.txt` | Experto Inmobiliario: ¡El Método SECRETO Para Multiplicar tu Patrimonio Sin Pedir Hipotecas! |
| `otros/Inventaron Esta Forma Para Crear Tu Patrimonio Desde 0 (Prop Hero) Ep 74.txt` | Inventaron Esta Forma Para Crear Tu Patrimonio Desde 0 (Prop Hero) — Ep 74 |

**[CONTRADICCIÓN evitada]** — el podcast de audio *Inversión Racional #132* tiene exactamente el título de una de estas transcripciones ("Experto Inmobiliario: ¡El Método SECRETO Para Multiplicar tu Patrimonio Sin Pedir Hipotecas!"). Es **el mismo contenido en otro formato**, no una fuente nueva — [ivoox.com](https://www.ivoox.com/experto-inmobiliario-el-metodo-secreto-para-multiplicar-tu-audios-mp3_rf_168304361_1.html). No se cuenta como hallazgo.

## B. Encontrado para revisar

### B1. Técnico y arquitectura — lo más valioso

| Fuente | Contenido | ¿Habla de la data? | Estado |
|---|---|---|---|
| **AWS Machine Learning Blog** — [How PropHero built an intelligent property investment advisor...](https://aws.amazon.com/blogs/machine-learning/how-prophero-built-an-intelligent-property-investment-advisor-with-continuous-evaluation-using-amazon-bedrock/) | Caso técnico completo del **asesor conversacional** (no del modelo de inversión): arquitectura de 4 capas, Bedrock, LangGraph, 6 agentes, Ragas, evaluación continua. Nombra al equipo: Lucas Dahan (Head of Data & AI), Dil Dolkun (Data & AI Engineer), Mathew Ng (Technical Lead) | **Sí**, pero del sistema conversacional, no del scoring de inmuebles | **[LEÍDO]** |
| — misma pieza, otras copias: [AI Singapore LearnAI](https://learn.aisingapore.org/2025/09/how-prophero-built-an-intelligent-property-investment-advisor-with-continuous-evaluation-using-amazon-bedrock/), [HKU SPACE](https://aihub.hkuspace.hku.hk/2025/09/26/how-prophero-built-an-intelligent-property-investment-advisor-with-continuous-evaluation-using-amazon-bedrock/), [ZenML LLMOps DB](https://www.zenml.io/llmops-database/multi-agent-property-investment-advisor-with-continuous-evaluation), [Bailador](https://bailador.com.au/news/how-prophero-built-an-intelligent-property-investment-advisor-with-continuous-evaluation-using-amazon-bedrock) | Reproducen el mismo caso | No añaden nada nuevo | No revisar salvo el original |

**[AVISO, importante]**: la única pieza técnica real que se ha encontrado describe el **chatbot de asesoramiento**, no el modelo que puntúa municipios e inmuebles. Ese —el que interesa al proyecto— **no se ha encontrado explicado técnicamente en público**. Es el hueco principal.

### B2. Material corporativo propio de PropHero

| Fuente | Contenido | Estado |
|---|---|---|
| [prophero.com/es/datos-e-ia](https://www.prophero.com/es/datos-e-ia/) | Cifras que ellos publican: 80M puntos de datos, "+8M variables", 10M nuevos/trimestre, scoring del "Top 1%", y una precisión declarada del 89% (modelo ene-2026) frente al 55% (modelo 2025). **Sin horizonte temporal de predicción** | **[LEÍDO]** — vendor, marketing |
| [prophero.com/data-and-ai](https://www.prophero.com/data-and-ai/) | Versión inglesa de la anterior | [SNIPPET] |
| [prophero.com/es/quienes-somos](https://www.prophero.com/es/quienes-somos/) | Historia y equipo | [SNIPPET] |
| [prophero.com/unblocking-land...](https://www.prophero.com/unblocking-land-is-fundamental-to-start-moving-the-market/) | Pilar Pascual (dircom) sobre suelo y mercado, jun-2026 | [SNIPPET] |
| [help.prophero.com — modelo de negocio](https://help.prophero.com/es/doc-center/modelo-de-negocio) | Comisiones y funcionamiento (ya citado en el hub) | [SNIPPET] |

### B3. Entrevistas escritas

| Fuente | Quién habla | Estado |
|---|---|---|
| [visualurb.es — Entrevista a Jaime Gil](https://www.visualurb.es/entrevista-a-jaime-gil-prophero/) | Jaime Gil, CEO PropHero España: modelos predictivos propios, validación de zonas **antes** de invertir | [SNIPPET] |
| [xm2news.com — Entrevista a Pablo Gil Brusola](https://xm2news.com/entrevista-a-pablo-gil-brusola-co-fundador-de-prophero/autoload/) | Pablo Gil, cofundador: "más de 240 variables" por operación | [SNIPPET] |
| [intereconomiavalencia.com — Pilar Pascual](https://www.intereconomiavalencia.com/pilar-pascual-prophero-nuestro-modelo-de-datos-nos-lleva-a-los-sitios-antes-de-que-ocurran-grandes-cosas/) | Pilar Pascual, dircom: "Nuestro modelo de datos nos lleva a los sitios antes de que ocurran grandes cosas" | [SNIPPET] |
| [reading.afterwork.vc — Spotlight Mickael Roger](https://reading.afterwork.vc/p/propherointerview) + [Investment Notes](https://reading.afterwork.vc/p/investment-notes-prophero-democratising) + [Pre-Seed to Series A](https://reading.afterwork.vc/p/from-pre-seed-to-series-a-propheros) | Mickael Roger, cofundador: **la construcción del primer modelo de datos**, MVP de 1 mes, 100+ usuarios de prueba. Detalle de origen | [SNIPPET] — **prioridad alta** |
| [ideas.everywhere.vc — Founders Everywhere: Mickael Roger](https://ideas.everywhere.vc/p/prophero-mickael-roger-founders-everywhere) | Perfil del cofundador, su paso por McKinsey (Data & AI) | [SNIPPET] |

### B4. Podcasts y audio (no son los vídeos de YouTube ya guardados)

| Fuente | Quién | Estado |
|---|---|---|
| [Venture Everywhere ep.118 — Zero to PropHero](https://www.podscan.fm/podcasts/venture-everywhere/episodes/zero-to-prophero-mickael-roger-with-jenny-fielding) | Mickael Roger: 25 años de datos, "cientos de variables", el modelo mide su propia precisión | [SNIPPET] |
| [producthackers.com — podcast Pablo Gil](https://producthackers.com/es/podcast/prophero-pablo-gil/) | Pablo Gil | [SNIPPET] |
| [podchaser — Fintech Chatter: Pablo Gil Brusola](https://www.podchaser.com/podcasts/fintech-chatter-insights-from-943229/episodes/pablo-gil-brusola-prophero-217654045) | Pablo Gil, en inglés | [SNIPPET] |
| [plazapodcast.valenciaplaza.com — Prophero](https://plazapodcast.valenciaplaza.com/plazapodcast/prophero) | PropHero | [SNIPPET] |
| [Audible AU — Entrevista 3×10: Pablo Gil](https://www.audible.com.au/podcast/Entrevista-3x10-Pablo-Gil-PropHero/B0BVXQD83Y) | Pablo Gil | [SNIPPET] |

### B5. Prensa económica y tecnológica

- **[ES]** [Cinco Días 2025-07-10](https://cincodias.elpais.com/companias/2025-07-10/prophero-disparara-sus-ingresos-un-66-hasta-los-50-millones-de-euros-en-2025.html) · [Cinco Días 2023-07-04](https://cincodias.elpais.com/companias/2023-07-04/la-australiana-prophero-dispara-sus-ventas-en-espana-que-ya-suponen-el-50-de-sus-operaciones.html) · [elEconomista 2026-02](https://www.eleconomista.es/vivienda-inmobiliario/noticias/13781724/02/26/prophero-supera-los-35-millones-y-preve-incorporar-mas-de-3000-viviendas-al-mercado.html) · [Europa Press 2026-02-18](https://www.europapress.es/economia/noticia-prophero-supera-35-millones-euros-facturacion-2025-preve-incorporar-mas-3000-viviendas-20260218080950.html) · [Las Provincias 2022](https://www.lasprovincias.es/economia/startups/prophero-revoluciona-inversion-20221017154616-nt.html) · [EjePrime](https://www.ejeprime.com/empresa/prophero-factura-30-millones-en-el-semestre-y-preve-superar-50-millones-a-cierre-de-ano) — [SNIPPET] todas. Hablan de resultados y expansión, poco de datos.
- **[AU/EN]** [Forbes Australia — Series A 25M](https://www.forbes.com.au/news/entrepreneurs/prophero-propelled-by-25m-series-a/) · [Startup Daily (seed 1,6M)](https://www.startupdaily.net/topic/funding/proptech-startup-prophero-raises-16-million-seed-round/) · [Startup Daily (8M)](https://www.startupdaily.net/topic/funding/real-estate-investment-startup-prophero-scoops-up-8-million-seed-round/) · [Australian FinTech](https://australianfintech.com.au/prophero-raises-1-6-million-to-grow-australias-next-gen-property-investment-platform/) · [Business News AU](https://www.businessnewsaustralia.com/articles/digital-property-investment-platform-prophero-raises--8-million-to-advance-global-operations.html) · [FinLedger](https://finledger.com/articles/prophero-raised-1-6m-for-its-data-driven-property-investment-platform/) · [The Property Tribune](https://thepropertytribune.com.au/tech/tech-for-millennials-to-be-a-prop-hero/) · [The Real Estate Conversation](https://therealestateconversation.com.au/news/2023/06/29/prophero-celebrates-its-second-birthday-with-continued-market-outperformance-new-app) — [SNIPPET]. Las de ronda de financiación suelen repetir la descripción del modelo (40M datos, 240 variables, 18.000 suburbios AU).

### B6. Ofertas de empleo — revelan el stack real

| Fuente | Qué revela | Estado |
|---|---|---|
| Lead / Principal Data Scientist (varias ubicaciones) — [talent.com](https://es.talent.com/view?id=243f0cc1f464), [expertini](https://es.expertini.com/jobs/in/lead-data-scientist-and-machine-learning-lead-castro-prophero/) | **Stack**: Python, scikit-learn, XGBoost, LightGBM, LangChain/LangGraph, AWS, MLOps. Modelos de clasificación, regresión, clustering, forecasting | [SNIPPET] — **prioridad alta** |
| [Data & Analytics Engineer (Indonesia) — jaabz.com](https://jaabz.com/jobs/134927-data-analytics-engineering) | Pipelines event-driven (Lambda, EventBridge, Kinesis/MSK, SQS), PostgreSQL, modelado dimensional, Metabase | [SNIPPET] |
| [Top of Minds — vacante "Property Scouter" (PDF)](https://topofminds.com/es/wp-content/uploads/sites/6/2026/03/2026-PropHero-Property-Scouter.pdf) | Descripción del puesto de búsqueda de inmuebles | [SNIPPET] |

### B7. Eventos

- [ULI Spain — "IA & Real Estate: del piloto al impacto real"](https://europe.uli.org/events/detail/5E355C32-7ABD-4542-BBD0-AEDACFFA99F1/) — Pedro Armas, Chief of Staff de PropHero, en la mesa "Identificación y análisis de oportunidades", junto a Idealista y PwC. **[SNIPPET]** — puede tener vídeo con detalle.

### B8. Comunidad y contraste (poco valor técnico, sí para contradecir el relato)

- [vivirdeinmuebles.com — opiniones](https://vivirdeinmuebles.com/prophero-opiniones/) · [Foro Balio](https://foro.balio.app/t/experiencias-prophero/13948) · [Forocoches](https://forocoches.com/foro/index.php/showthread.php?p=498727049) · [Finect](https://www.finect.com/usuario/eduardogarcia/articulos/prophero-analisis-y-opiniones-de-la-plataforma-de-inversion-inmobiliaria) · [Trustpilot](https://www.trustpilot.com/review/prophero.es) — **[SNIPPET]**. Ya cubierto en el hub; solo útil para contrastar cifras de rentabilidad.

## C. Descartado y no encontrado

- **Reproducciones del caso AWS** (AI Singapore, HKU, ZenML, Bailador, Colaberry, The AI Mag, blogs coreanos/alemanes): mismo contenido, no aportan.
- **Rankia**: el hub lo citaba como foro con opiniones, pero **no ha aparecido en ninguna búsqueda de esta ronda**. Queda por localizar o dar por perdido.
- **LinkedIn de fundadores y equipo de datos**: no indexado en las búsquedas. Habría que ir directo al perfil.
- **South Summit / 4YFN**: sin resultados.
- **Ponencias grabadas sobre datos**: ninguna encontrada.

## Cobertura y huecos

**Qué se buscó** (4 rondas, WebSearch; 2 fuentes abiertas con WebFetch): corporativo propio · entrevistas escritas ES y EN · podcasts (audio y vídeo) · prensa económica ES y AU · ofertas de empleo · eventos del sector · foros de comunidad · vendor (AWS).

**Qué NO está cubierto**: LinkedIn (sin indexar), y no se ha abierto ninguna de las fuentes marcadas [SNIPPET] — la única abierta y verificada de extremo a extremo es la de AWS y la de datos-e-ia.

**El hueco que importa**: hay mucho publicado sobre *sus resultados* y sobre *su chatbot*, y **casi nada verificable sobre las variables concretas y la construcción del modelo que puntúa municipios**. Si en algún sitio está, los candidatos más probables son las notas de AfterWork con Mickael Roger (B3) y el vídeo del evento ULI (B7).

## Relacionado en el wiki

- [[rentabilidad-inmobiliaria-con-datos]] — hub del proyecto, donde se extrae el conocimiento de las transcripciones
- [[sintesis-radar]] — síntesis consolidada de todo el conocimiento del proyecto
