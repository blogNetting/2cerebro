---
title: Rentabilidad inmobiliaria con datos
created: 2026-09-29
updated: 2026-09-29
tags: [inmobiliario, inversion, datos, rentabilidad]
zona: tecnico
---

Hub del proyecto: buscar buenas rentabilidades en inversión inmobiliaria apoyándose en datos, replicando el "buscador" de Prophero (predecir oportunidades con datos de revalorización y rentabilidad de alquiler) como hipótesis a contrastar, no como premisa.

## Cómo leer este documento: etiquetas obligatorias

Cada dato lleva una etiqueta, para que se pueda distinguir de un vistazo qué es cada cosa sin tener que releer el párrafo entero:

- **[CITA]** — texto literal de una fuente, con su ubicación exacta (línea de fichero o URL), verificado mecánicamente (`grep` sobre el `.txt`, o lectura directa de la web).
- **[INFERENCIA]** — conclusión propia a partir de una o varias citas. La fuente no lo dice así; es una lectura mía de lo que dice.
- **[SIN VERIFICAR]** — algo que dice una fuente pero que no se ha podido contrastar con ninguna otra.
- **[CONTRADICCIÓN]** — dos citas (de la misma fuente o de fuentes distintas) que no encajan entre sí. Se dejan las dos visibles, no se elige una.

## Errores encontrados y corregidos — no se borran, quedan escritos

**2026-09-29, error propio:** en una versión anterior de esta nota atribuí el caso de Almazora (crecimiento del 4% anual, stock bajo en Idealista, 88 pisos de obra nueva vendidos en 7 días) a la transcripción `tengo 1 plan 1`, mezclándolo con el caso de Sagunto de ese mismo fichero. Comprobado con `grep`: Almazora **no aparece en ningún momento** en `tengo 1 plan 1` — el caso es de `tengo 1 plan 2` (líneas 778-792). Corregido: Almazora se movió a la tabla de la primera fuente, y Sagunto se quedó con su propia sección en `tengo 1 plan 1`, con las 14 veces que se menciona repasadas una a una. El usuario lo detectó porque notó que Sagunto — mencionado muchísimo en el audio — apenas aparecía en el resumen escrito.

**Causa**: al extraer varias transcripciones parecidas (mismo tema, mismos ponentes, topónimos que se repiten) en la misma sesión, se puede atribuir un dato al fichero incorrecto si no se re-verifica cada cita contra su fuente exacta en el momento de escribirla, no solo al principio. **Medida aplicada desde ahora**: toda cita de este documento se verifica con `grep` sobre el fichero que se va a citar, en el momento de escribirla — no se reutiliza de memoria una cita ya usada en otra sección.

## Cronología de las fuentes — importante para no confundir evolución temporal con contradicción

El usuario avisó (2026-09-29): los vídeos son de fechas distintas, y varias de las cifras que antes se marcaron como [CONTRADICCIÓN] pueden ser en realidad la misma cifra real, medida en momentos distintos. Esto es lo que se pudo fechar con evidencia **dentro de las propias transcripciones** (ninguna trae fecha de grabación explícita en metadatos, así que esto es reconstrucción a partir de lo que dicen):

- **`tengo 1 plan 2`** — **[CITA]** fecha explícita, dicha dos veces: "para que la gente tenga una referencia a **septiembre de 2026**" (líneas 30 y 2092-2093). **Esta transcripción es de ~septiembre de 2026.**
- **`otros/Cómo Invertir tu Dinero hoy...`** — **[CITA]**: "35 [millones] en 2025 cerraremos en facturación" (dicho en futuro, línea 155) y "4 años y medio llevamos de vida" (línea 169-170). **Esta transcripción es de algún momento de 2025**, antes de cerrar el año.
- **`tengo 1 plan 1`** — **[CITA]**: Jaime dice "yo llevo invirtiendo desde el 2014" (líneas 176-178) y más adelante "yo llevo 10 años, mañana cumplo 10 años en el sector inmobiliario" (líneas 2926-2927). **[INFERENCIA]**: 2014 + 10 = **esta transcripción es de ~2024**.
- **`otros/Cómo PropHero encuentra las zonas...`** — **[SIN VERIFICAR]**: no contiene ninguna cita que la feche con precisión. Por el tono (hablan de Toledo como un descubrimiento reciente, "hace un año cuando empezamos a viajar a Toledo") podría ser la más antigua de las cuatro, pero es una impresión, no una fecha.

**Orden cronológico reconstruido**: `otros/Cómo PropHero encuentra las zonas...` (¿2022-2023?, sin confirmar) → `tengo 1 plan 1` (~2024) → `otros/Cómo Invertir tu Dinero hoy...` (~2025) → `tengo 1 plan 2` (septiembre 2026).

**Lo que esto resuelve de verdad:**

- **Umbral de rentabilidad neta mínima**: 8% (tengo1plan2, dicho como "hace 3-4 años" desde 2026 → ~2022-2023) → **6%** (tengo1plan1, 2024) → **4-5%** (tengo1plan2, 2026). **Es una secuencia perfectamente coherente en el tiempo, no una contradicción** — los tres números encajan en orden con sus fechas. Se retira la etiqueta [CONTRADICCIÓN] de este punto y se deja como evolución confirmada.

**Lo que esto NO resuelve, y sigue siendo contradicción real:**

- **Población de partida de Ocaña**: si las fechas fueran la explicación, el vídeo más reciente (tengo1plan2, 2026) debería dar la cifra de población **final** más alta. Pero la cifra final que dan las tres fuentes es prácticamente la misma (14.000 en 2024, 14.000-14.700 en 2026, 14.700 en la fuente sin fechar) — **no crece entre 2024 y 2026 según sus propios números**, mientras que la de partida sí cambia (8.000 / 8.000-9.000 / 10.000). **[INFERENCIA]**: esto sugiere que el dato de Ocaña es un ejemplo de marketing que repiten tal cual de vídeo en vídeo, sin actualizarlo con el tiempo, más que una medición en vivo. Se mantiene la etiqueta [CONTRADICCIÓN].
- **Revalorización de sus inmuebles**: 13,7% (tengo1plan1, 2024) frente a ">15%" del cohort 2022-2024 (tengo1plan2, 2026) — el orden temporal es compatible con que haya subido, pero no está claro si ambas cifras miden lo mismo (¿anualizado o acumulado del periodo?), así que sigue sin poder conciliarse con precisión — se mantiene como [SIN VERIFICAR] el método de cálculo, no como contradicción cerrada.

## Objetivo

Montar un método propio, basado en datos, para identificar inmuebles con buena rentabilidad de inversión — replicando lo que Prophero dice que hace, contrastado con fuentes reales. **[INFERENCIA de alcance, confirmada por el usuario el 2026-09-29]**: primero se agota la extracción de las 8 transcripciones aportadas; solo después se decide qué datos públicos harían falta para construir el método propio.

## Qué es Prophero

**[CITA, vendor]** Proptech de inversión inmobiliaria en España, activa desde 2021. Modelo "llave en mano": el cliente compra un inmueble físico a su nombre (no participaciones) y Prophero se encarga de búsqueda, negociación, reforma, amueblado y gestión del alquiler. Dice usar "+250 variables" y "más de 80 millones de datos" — [prophero.com/es/resultados](https://www.prophero.com/es/resultados/).

- **Comisiones — [CITA, vendor + independiente, coinciden]**: 7.500 € de servicio (1.500 € inicial + 3.000 € al seleccionar inmueble + 3.000 € tras compra/reforma/alquiler) + 5-7% anual sobre la renta de gestión — [help.prophero.com/modelo-de-negocio](https://help.prophero.com/es/doc-center/modelo-de-negocio) (vendor); confirmado de forma independiente en [vivirdeinmuebles.com/prophero-opiniones](https://vivirdeinmuebles.com/prophero-opiniones/).
- **Entrada mínima — [CITA, vendor]**: 100.000 € (o 25.000 € con su sistema de "tickets").
- **Rentabilidad que anuncian — [CITA, vendor + independiente, coinciden]**: 6-7% neto anual en alquiler residencial; el análisis independiente de vivirdeinmuebles.com confirma con un caso real ("Rentabilidad neta = 6.000€ / 100.000€ = 6% neto").
- **Opiniones de clientes — [CITA, mixtas]**: Trustpilot 3,5-4,0/5 sobre 200-255 reseñas ([trustpilot.com/review/prophero.es](https://www.trustpilot.com/review/prophero.es)). Recurrente en positivas: gestión llave en mano cómoda. Recurrente en negativas y en foros (Rankia, foro Balio, Burbuja.info): retrasos largos en reforma y puesta en alquiler (casos de 6-8 meses sin ingresos), cambios constantes de gestor, costes por encima de lo esperado.
- **Referencia oficial de mercado — [CITA, oficial]**: el Banco de España sitúa la rentabilidad bruta media del alquiler en España en el 3,0% a finales de 2025 (Informe de Estabilidad Financiera, otoño 2025). **[INFERENCIA]**: está por debajo de lo que anuncia Prophero, lo que sugiere que su cifra se basa en una selección de inmuebles concreta, no en el mercado medio — no confirmado por ninguna fuente, es lectura propia.

**[SIN VERIFICAR]**: si el "análisis de datos" es una ventaja real y diferencial frente a competidores, o principalmente argumento comercial; qué parte de su resultado depende de negociación/escala (no replicable por un particular) frente a qué parte es puramente selección por datos (sí replicable).

## El buscador de Prophero: cómo dicen que lo construyen

Fuente: `fuentes/prophero-transcripciones-2026-09-29/tengo 1 plan 2/Experto en Vivienda Revela Cuando Explotará la Burbuja Inmobiliaria "Haz Esto con tu Dinero.".txt` (3.212 líneas, leída completa, no por palabra clave). Entrevista del podcast "Tengo un plan" a dos personas de la empresa, identificadas como Jaime (fundador/inversor) y Joaquín (operaciones).

**[INFERENCIA]**: la transcripción automática escribe el nombre de la empresa como "Prop Giro" / "Progiro" / "Propgiro", nunca "PropHero". Lo trato como la misma empresa por: mismo modelo de negocio exacto que la web de Prophero, y un anuncio patrocinado a mitad del vídeo con el código de descuento "Tengo un plan" (línea 552-574), formato típico de publirreportaje de Prophero en ese podcast. **El nombre correcto nunca aparece escrito en el fichero — esto no es una cita, es una suposición mía.**

**[CONTRADICCIÓN, del propio presentador al principio del vídeo]**: al arrancar, el presentador cita una noticia — "hace falta más de 1 millón de viviendas" (línea 70) — y los invitados responden con su propia cifra, distinta: "la nuestra es 750.000" (línea 262). No se resuelve cuál es correcta; son dos fuentes distintas (una noticia sin identificar, y el dato interno de la empresa) dentro del mismo minuto de vídeo.

### Volumen y variables de datos que dicen tener

**[CITA]** (líneas 1399-1406):
> "hemos creado un modelo de datos predictivo con 200 millones de puntos, ¿vale? Estos 200 millones de datos individuales son en España hay 8000 municipios, tenemos un histórico de 12 años de data trimestral de 500 variables"

**[CONTRADICCIÓN]**: más adelante, línea 1546, la misma persona dice "tenemos 16 años de data trimestral en 8000 municipios" — 16 años, no 12. Además, la cuenta en voz alta de la línea 1404 ("500 * 3 * 12 por 8000") no cuadra aritméticamente con "200 millones": 500×3×12×8000 = 144 millones. Con 16 años y 4 trimestres saldría 500×4×16×8000 ≈ 256 millones, tampoco 200 millones exactos. **Ninguna de las dos cuentas que dan cuadra con la cifra que repiten como eslogan.** Se trata como cifra de marketing, no como dato duro.

**[CITA]** variables que dicen usar, por municipio y trimestre desde 2010 (líneas 1530-1546):
> "Tienes datos desde oye, la población, ¿cómo ha crecido los últimos 1 tr y 5 años? Eh, la tasa de paro, ¿cómo ha ocurrido los últimos un tres y 5 años? E la renta per cápita, ¿qué ha ocurrido con ella, la tasa de esfuerzo? [...] ¿Qué ha ocurrido con el precio medio de la vivienda, que ha ocurrido con el precio medio del alquiler?"

**[CITA]** fuentes de datos que nombran explícitamente (líneas 1524-1529 — "notradial" es transcripción automática de "notarial"):
> "al final los datos los puedes comprar, hay distintas fuentes de información que puedes comprar datos, también puedes sacarlos del registro notradial, del catastro, incluso idealista"

**[INFERENCIA]**: de esa cita se entienden cuatro tipos de fuente: (1) proveedores de datos de pago sin nombrar cuáles, (2) registro notarial (la misma fuente que usa el índice oficial del INE, vía el Consejo General del Notariado), (3) catastro, (4) Idealista.

### El método: de "reactivo" a "predictivo"

**[CITA]**, de otra transcripción del mismo proyecto (`otros/Cómo PropHero encuentra las zonas...txt`, líneas 44-50 y 89-97): el modelo "reactivo" correlaciona datos ya ocurridos (demografía, renta, impagos, ocupación, liquidez de venta/alquiler) con subidas de precio ya confirmadas; el modelo "predictivo" usa "inteligencia artificial y la programación que está haciendo el equipo" para anticipar qué variables se van a activar antes de que el precio reaccione (ejemplo dado: suelo logístico/industrial como indicador adelantado de crecimiento poblacional).

**[CITA]**, confirmación del término técnico en la transcripción larga (líneas 1568-1574):
> "eso sería un home los de los ingenieros de datos le llaman machine learning que al final lo que está es aprendiendo el propio sistema con nuestras herramientas de IA de cómo llegar antes"

**[CITA]** indicador adelantado que dicen usar en la práctica (líneas 1575-1591): número de visados de obra nueva en un municipio, y desarrollo de suelo logístico/industrial — explícitamente **no** centros de datos, porque no generan empleo ("no trabaja prácticamente gente").

### El criterio de scoring — la definición operativa de "buena oportunidad"

**[CITA]** (líneas 1424-1441):
> "nosotros creemos que una buena oportunidad generalmente implica dos cosas. La primera cosa es que tenga una rentabilidad neta ahora superior al 4 y5 5%, ¿vale? Hace hace 3 o 4 años será el 8, ahora buscamos un 4 y5% que estos son entre un 10% de los municipios de España. Por otro lado, el 10% de las que más se van a revalorizar [...] Cuando mezclas las que tienen una rentabilidad neta superior al 4,5 y las que se van a revalorizar más de un 10%, cuando juntas estos dos se generan que en España de los 8.000 hay un 5% de municipios que son los que mayor oportunidades de inversión ofrecen."

**[INFERENCIA]**, resumen de la cita anterior: intersección de dos filtros — top 10% de municipios por rentabilidad neta (umbral ≥4-5%, bajado desde ≥8% hace 3-4 años porque han subido los precios) **y** top 10% de municipios por revalorización esperada (>10%) → el 5% de los 8.000 municipios de España con mejor combinación.

**[CITA]** cómo definen bruta y neta, con la sensibilidad que dan (líneas 1442-1492):
> "La rentabilidad bruta es el alquiler 600 € al mes y dividirlo entre tu precio de compra del activo [...] La rentabilidad neta, si la haces neta de verdad [...] te sale entre un 30 y un 40% más baja que la bruta [...] Si tienes un ocho, en verdad lo que tienes es un 5 y medio [...] La rentabilidad neta metes el IBI, los gastos de comunidad, el seguro, los gastos que te genera el mantenimiento del piso, si tiene la gestión del alquiler [...] Comprar 5.000 € más caro solo te sube la rentabilidad neta 0,3 [...] mientras que 100 € menos de alquiler te quita un punto. Es decir, la rentabilidad neta, el numerador [el alquiler], es mucho más importante que el denominador [el precio]."

### Casos concretos con cifras — útiles para contrastar después con datos públicos

| Municipio | Datos que citan | Fuente |
|---|---|---|
| Soyana (Valencia, cerca de Alzira) | 5.000 hab., creciendo ~1%/año (~50 pers./año); a 16 km / 20 min en metro de Valencia; paro 12%; solo 30 pisos vendidos el último trimestre; stock en Idealista bajó 70% interanual; tiempo medio de alquiler 4 semanas; un único piso en alquiler en todo el pueblo en Idealista, y es de ellos; predicen 26% de revalorización | **[CITA]** líneas 1866-1968 |
| Ocaña (Toledo) | de 8.000-9.000 a 14.000-14.700 hab.; piso comprado a 60.000 €, alquilado a 500 €, al cambiar inquilino subió a 800 €/mes → rentabilidad neta pasó a 10-11% | **[CITA]** líneas 1499-1513, 929-931 |
| Zaragoza | descartada como prioridad a pesar de encajar en el criterio: exceso de stock de suelo disponible para vivienda, crecimiento poblacional menos agresivo que el Valle del Sagra/Sagunto | **[CITA]** líneas 1610-1646 |
| Seseña (Toledo/Madrid) | urbanización fallida de la crisis de 2008 ("el pocero"); reconocen que se equivocaron por no entrar, por un ticket demasiado alto (130.000 €) para su rango de entonces | **[CITA]** líneas 1734-1789 |
| Almazora (Castellón) | citan un crecimiento poblacional del 4% anual; solo ~10 pisos en alquiler en Idealista (stock bajo); sacaron 88 pisos de obra nueva que se vendieron en 7 días. **[SIN VERIFICAR]**: la cifra de población de base que dan es internamente confusa en la propia transcripción ("30 y tantos" frente a "40.000 habitantes" en la misma frase) — no se puede fijar el dato base con lo que dicen | **[CITA]** líneas 778-792 |

**Nota sobre Molina de Segura**: el dato (70.000 hab. en 2016 → 76.100 en 2022; renta de 21.978 € a 23.600 €) es de la tercera fuente (`otros/Cómo PropHero encuentra las zonas...txt`), ya ingerida entera más abajo — ver esa sección.

**[CITA]** frase clave sobre los límites del propio método (líneas 1777-1779):
> "la data no te permite acertar siempre a dónde sí que hay que ir, te permite eliminar dónde seguro que no hay que ir"

### Ticket, financiación y apalancamiento

**[CITA]**:
- Obra nueva: ticket ~120.000-130.000 €, entrada del 20% (24.000-26.000 €), 80% financiado por banco.
- Segunda mano: mínimo ~50.000-60.000 €.
- Estrategia "apalancamiento sin consumir CIRBE" (líneas 995-1022): reservar directamente al promotor (10.000 € de reserva + 10.000 € en cuotas = 20.000 €) hace que el comprador se beneficie de la revalorización del 100% del activo durante la construcción sin que esa deuda conste en el informe de riesgos del Banco de España, porque la deuda de la promoción es del promotor, no del comprador, hasta la entrega.

### Comisiones citadas aquí — discrepancia con lo ya verificado en la web oficial

**[CITA]** (líneas 2229-2278):
> "Progiro lo que se lleva son 7.000 € por oportunidad de inversión [...] 1.000 € por empezar a ver oportunidades [...] 3.000 cuando reservas y pones arras [...] y luego el día de notaría otras 3.000"

**[CONTRADICCIÓN]**: esto es 7.000 € en total (1.000+3.000+3.000). Lo verificado antes en `help.prophero.com/modelo-de-negocio` y en vivirdeinmuebles.com decía 7.500 € (1.500+3.000+3.000). Puede ser un cambio de tarifa entre fechas distintas (la transcripción no tiene fecha exacta) — se deja la discrepancia visible, no se elige una cifra sobre la otra.

### Métricas de resultado que citan de sí mismos — auto-reportadas, sin auditoría externa

**[CITA]** (líneas 1888-1893): "El cohort [...] 2022, 2023 y 2024 [...] tenemos de media una revalorización superior al 15% y una rentabilidad bruta dos puntos por encima de la media de España."

**[CONTRADICCIÓN]** (líneas 1896-1902): dicen que sus inversiones "mejoran las de media de la zona en dos" y, unas palabras después en la misma frase, "en dos puntos, es decir, cuatro [...] sobre la media de la provincia" — dicen "dos" y "cuatro" en la misma frase sin aclarar cuál es. Se deja constancia tal cual se dijo.

**[CONTRADICCIÓN]** (línea 2277-2278): "50% [...] son repeat client" aquí, frente al 60% de repetición que daba `prophero.es/resultados` (vendor, verificado en la ronda de búsqueda anterior). Sin fecha en ninguna de las dos fuentes que permita saber cuál es más reciente.

**[CITA]** escala actual que dicen manejar: ~36 millones € en reformas en curso, 100 reformas en paralelo, "22 clusters" (Castellón a Almería, sur de Madrid, Zaragoza). Han dejado de aceptar pisos sueltos individuales por no poder escalar la operación (antes hacían reformas de 5.000-20.000 € por piso suelto; hoy firman 150-200 pisos/mes).

### Contexto de mercado general (no específico de Prophero)

**[CITA]**:
- Déficit de vivienda: 600.000 (hace ~1,5 años) → 750.000 (hoy) → previsión de 900.000 en 2-3 años; tardaría 7-10 años en cubrirse según su propia previsión.
- El 60% de ese déficit de 900.000 viviendas se concentra en 4 capitales (Madrid, Barcelona, Valencia, Málaga); Madrid sola es el 30% del déficit total.
- Solo 12 de las 51 provincias españolas crecen en población.
- Desde 2007, la obra nueva se ha revalorizado ~26 puntos porcentuales más que la segunda mano (de media España acaba de volver a igualar precios de 2007 en segunda mano; en obra nueva se superó hace 5 años).

**[SIN VERIFICAR]**: ninguna de estas cifras de mercado general se ha contrastado todavía contra una fuente oficial (INE, Banco de España, Colegio de Registradores). Son afirmaciones de la empresa, no dato público confirmado.

## Segunda fuente: "2 Expertos en inversión..." (`tengo 1 plan 1`)

Fuente: `fuentes/prophero-transcripciones-2026-09-29/tengo 1 plan 1/2 Expertos en inversión Si Ganas entre 1.000 y 3.000€ haz Esto para Crear Riqueza!.txt` (4.234 líneas, leída completa). Mismo podcast, entrevista a Jaime y Pablo — **[INFERENCIA]**: Pablo aparece aquí como cofundador (cuenta la historia de origen, socio de Jaime), distinto de "Joaquín" (operaciones) de la primera transcripción. Al menos tres personas de la empresa quedan identificadas entre las dos fuentes: Jaime, Joaquín, Pablo.

### Escala que declaran (líneas 14-16, cita literal, del presentador)
> "cómo gestionan más de 1000 millones en inversiones inmobiliarias"

**[SIN VERIFICAR]**: es la presentación del entrevistador, no una cifra que los propios invitados repitan o desglosen en el resto del vídeo.

**[CITA]** otras cifras de escala que sí dan ellos: venden "cerca de 200 pisos cada mes" (línea 859-860); tienen bloqueados en arras "400 y pico inmuebles" cada mes (línea 917); "150 personas" de plantilla directa, hasta 180 con becarios (línea 721); llevan "4 años" con la empresa, "3" en España (línea 714-717).

**Diferencia de escala frente a la otra transcripción, explicada por la fecha (ver "Cronología de las fuentes")**: aquí (2024) hablan de 4 clusters (Zaragoza, Valencia-Castellón, Alicante-Murcia, alrededores de Madrid — línea 554-559) y 150-180 empleados; `tengo 1 plan 2` (2026) habla de "22 clusters" y 100 reformas en paralelo con 36 millones € en curso. **[INFERENCIA]**: es coherente con crecimiento de la empresa en esos 2 años, no una contradicción — aunque no hay forma de confirmar que "cluster" se cuenta igual en ambas fuentes.

### Fuente de datos nueva que no había aparecido: scraping propio

**[CITA]** (líneas 536-539 — "escraeamos" es la transcripción automática de "escrapamos"):
> "escrapamos, escraeamos todas las webs de inmuebles, escraeamos todas las webs de alquiler y luego escraeamos un montón de de webs para obtener información"

**[INFERENCIA]**: esto añade una quinta fuente a las cuatro ya identificadas en la otra transcripción (datos comprados, registro notarial, catastro, Idealista) — scraping propio de portales de vivienda y alquiler, no solo Idealista con nombre propio.

### El criterio de scoring, dicho con otras palabras — corrobora el de la primera fuente

**[CITA]** (líneas 296-308):
> "No invertimos donde hay una oportunidad. invertimos donde hay una serie, un ecosistema en el cual se está generando empleo, se está generando que venga gente a vivir, crece la población, crece la renta por cápita. En ese ecosistema generamos viviendas o conseguimos viviendas que tengan una rentabilidad en concreto con unos parámetros y que no solo sean rentables hoy, sino que tengamos la certeza por nuestro modelo de data que van a seguir creciendo."

**[CITA]** umbral de rentabilidad citado aquí (línea 527-534): "el modelo de dato me dice cuáles son las poblaciones que más va a crecer el precio de la vivienda a los próximos 3 años y que hoy tienen una rentabilidad mínima del 6% neto". **Ya no se marca como contradicción** frente al 4-5% (y "8% hace años") de la primera fuente — con la cronología reconstruida en la sección "Cronología de las fuentes" (esta transcripción es de ~2024, la otra de ~2026), la secuencia 8% (~2022-23) → 6% (2024) → 4-5% (2026) es coherente en el tiempo.

**[CITA]** las "tres fases" de entrada a una zona (líneas 519-524, 632-643): fase 1 — los primeros en llegar, mayor riesgo y mayor revalorización potencial, ticket más bajo, menor liquidez de alquiler; con el tiempo sube el ticket, baja el riesgo y baja la revalorización potencial a medida que la zona "madura".

### Rentabilidad y revalorización entregada — más cifras, solo parcialmente explicadas por la fecha

**[CITA]** (líneas 990-1010):
> "hace 3 años la rentabilidad sobre las rentas era más elevada que ahora. Estábamos casi en el 7,7 aproximadamente y a día de hoy estamos [...] en un 6 y medio aproximadamente [...] sobre la revalorización de media, los 3 años que estamos en España [...] el mercado de vivienda ha crecido a un 7% los últimos 3 años [...] anual [...] En los sitios donde Progiro ha decidido invertir, estamos en un 10 y5 casi en un 11 y [...] los inmuebles de Progiro en más de un 13, casi un 14%, 13,7 aproximadamente"

**[INFERENCIA]**: es decir, según ellos — mercado medio de España: 7% anual de revalorización; sus zonas elegidas: ~10,5-11%; sus inmuebles concretos: ~13,7%. Rentabilidad sobre rentas entregada: bajó de 7,7% (hace 3 años) a 6,5% (ahora).

**Frente a la otra transcripción** (2026): allí la revalorización del cohort 2022-2024 era ">15%" y la rentabilidad bruta "dos puntos por encima de la media de España". Aquí (2024) la rentabilidad entregada es 6,5% neto (no bruta) y la revalorización de sus inmuebles 13,7%, no >15%. El orden temporal (2024 → 2026) es compatible con que la cifra haya subido, pero no está claro si miden lo mismo (bruta vs neta, anualizado vs acumulado del cohorte) — se deja como **[SIN VERIFICAR]** el método de cálculo, no como contradicción cerrada (ver "Cronología de las fuentes").

### El caso Sagunto — mencionado 14 veces en esta transcripción, su ejemplo insignia

**[INFERENCIA]**: "Sagunto" aparece 14 veces en este fichero (verificado con `grep -ic`), más que cualquier otro topónimo — es el ejemplo que más repiten a lo largo de toda la entrevista, no una mención suelta. Se le dedica sección propia en vez de una fila de tabla.

**[CITA]** el "subyacente" industrial que justifica la zona (líneas 569-580):
> "en el puerto de Sagunto está montando la giga de baterías [de Volkswagen — mencionada así, sin nombrarla aquí explícitamente], que Mercadona va a poner tres centros logísticos, Intitec y otros más. Y luego entre Sagunto y Castellón van tres polígonos industriales [...] entre las azulejeras de Villarreal [Pamesa, Porcelanosa] [...] al medio va un polígono en Almenara, uno en Chilches y uno en Menavites"

**[CITA]** una fábrica de muebles competidora de IKEA — el nombre que da la transcripción, "GISK", no se ha podido verificar como nombre real de empresa; es probable que sea otro error de transcripción automática, igual que "Prop Giro" por "PropHero" — (líneas 582-585): "va a poner 300 millones de euros para una giga para 800.000 m² de suelo". **[SIN VERIFICAR]**: el nombre de esta empresa.

**[CITA]** el caso de revalorización concreto (líneas 585-591): "yo me compré un piso hace 3 años ahí a 33.000 € o 34.000 [...] Hoy hemos vendido pisos ya en ese pueblo a 90.000 [...] por tres de lo que nos costó". **[SIN VERIFICAR]**: "ese pueblo" no se nombra en la frase — por el contexto inmediato podría ser Almenara, Chilches o Menavites (los tres polígonos citados justo antes), no Sagunto capital ni Almazora (que es de otra transcripción, `tengo 1 plan 2` — corregido aquí porque lo había atribuido mal a este fichero en una versión anterior de esta nota).

**[CITA]** siguiente fase del mismo pueblo (líneas 596-603): "ya no quedan pisos de los de 30, 40, 50, 60. Ahora estamos comprando todos los locales para hacer viviendas y en Chilches Castellón [...] estamos haciendo 60 o 60 [sic, repetido] viviendas en locales y edificios vandalizados".

**[CITA]** Sagunto como zona del producto "Value Partner" (líneas 1102-1103): "ejemplo en Sagunto, que es una zona muy buena" — es una de las dos zonas que nombran explícitamente para ese producto (la otra es "sur de Valencia").

**[CITA]** Sagunto como referencia de zona "ya madura", frente a una zona nueva de mayor riesgo — este es el contraste más importante para el criterio de scoring (líneas 1808-1850):
> "Jaime ya se ha pasado Liga de Sagunto" [...] Jaime invirtió a nivel personal en Carlet, "una población de Valencia, tiene una barriada que no es la más coqueta del mundo", un producto que **descartaron para clientes de Prophero** por el tipo de barrio, pero que aparecía "en el top 20 capital gain de revalorización los próximos 3 años" — mayor riesgo, mayor revalorización potencial. Para un cliente que invierte por primera vez: "nunca te llevaríamos a Carlet, llevaríamos a un Sagunto ya establecido, porque Sagunto hace tres años [era la oportunidad], no es ahora" — **menos rentabilidad, menos riesgo, ya consolidado**.

**[INFERENCIA]**: este contraste (Carlet=riesgo/nuevo vs Sagunto=maduro/seguro) es la ilustración más concreta de las "tres fases" de entrada a una zona que describen en otra parte de la misma transcripción (líneas 519-524, 632-643) — Sagunto ya pasó por las tres fases y hoy está en la última.

**[CITA]** Sagunto mencionado también como ejemplo de zona donde comprarse la vivienda propia, no solo para invertir (líneas 2170-2172): "vivo en Toledo o vivo en Madrid, en la periferia, o vivo en Sagunto o vivo en una población donde el precio de vivienda va a crecer mucho."

**[CITA]** Puçol y la promoción de María Zambrano se describen como poblaciones "alrededor de Sagunto" (línea 3697-3698) — es decir, Sagunto funciona como el polo industrial ancla de todo un clúster de pueblos satélite (Puçol, Almenara, Chilches, Menavites), no como un caso aislado.

### Otros casos concretos, con cifras

| Lugar | Datos que citan | Línea |
|---|---|---|
| Puçol (Valencia) | primer piso vendido por Prophero en España fue aquí: costó 50.000-55.000 €, hoy vale ~140.000 € | **[CITA]** 3696-3710 |
| María Zambrano / Puçol | local comercial de 1.000-1.100 m² convertido en 13 viviendas; coste total ~75.000-80.000 €/vivienda (compra+reforma+gastos); se venden a 130.000 € | **[CITA]** 3696-3750 |
| Moncada (Valencia) | restaurante y fonda cerrados 15 años; compra 300.000 € + reforma 350.000 € (~800.000 € total) para 8 viviendas; esas 8 viviendas valdrían ~1,2 millones € en el mercado de la zona (ningún piso por menos de 150.000 €/unidad) | **[CITA]** 3801-3859 |
| Ocaña (Toledo) | de 8.000 a 14.000 hab. en 6-7 años; piso comprado a 60.000 € hace 2-3 años, hoy vale 130.000-140.000 € | **[CITA]** 429-448 (corrobora la cifra de la otra transcripción, con el mismo pueblo) |
| Linares (Jaén) — ejemplo de zona a **evitar** | cerró la fábrica de Land Rover; es la población con mayor tasa de paro de España; citada explícitamente como el tipo de sitio donde no invertir aunque el precio sea barato | **[CITA]** 2303-2312, 2958-2975 |
| Zona Alicante-Murcia (Elche, Albatera, Torrevieja, Molina de Segura) | 4 de cada 10 viviendas de la zona las compran extranjeros (holandeses, suecos, alemanes); Molina de Segura corrobora la cifra de población de la tercera transcripción | **[CITA]** 2369-2390 |

### Un caso de escepticismo propio ante un dato de terceros — vale como ejemplo de cómo tratan datos externos

**[CITA]** (líneas 2348-2368): Fotocasa publicó una caída del 4% en la rentabilidad del alquiler en Murcia; uno de ellos lo cuestionó públicamente ("dije: no me lo creo. Puede haber pasado algo que haya intoxicado un dato") porque no encajaba con lo que su propio modelo indicaba para esa zona (que la consideran "top" en revalorización). Nadie le dio una explicación sólida distinta a "estacionalidad de postverano". **[INFERENCIA]**: esto es útil como ejemplo de que ni siquiera ellos se fían de un dato agregado de un portal sin contrastarlo — el mismo principio que se está aplicando en este documento.

### Otros mercados donde operan (fuera de España)

**[CITA]** Dublín (Irlanda), líneas 1656-1685: venden colivings ("casas de 8-10 habitaciones", modelo rent-to-rent); ticket 50.000-200.000 €; rentabilidad "7 y medio" a 8-9% más revalorización; horizonte de venta a 3 años; el inversor compra una participación de una sociedad propietaria de la casa, no el inmueble directo.

**[CITA]** Bali (Indonesia), líneas 3526-3684: "llevamos más de 12 proyectos" en construcción (villas, hoteles); modelo de propiedad tipo *leasehold* (alquiler de terreno a largo plazo, no propiedad plena — necesitarías una sociedad local para *freehold*); reconocen problemas de calidad de acabados en el mercado en general ("el nivel de acabados que tiene esto no está alineado con lo que debería"); justifican la demanda por crecimiento poblacional de Indonesia (180 millones hab., "el cuarto país que más crece en número de habitantes").

**[CITA]** España para inversión extranjera propia — no aclarado del todo (línea 1580-1584): "por un tema fiscal y porque somos hiperseguros con el tema de intereses de Hacienda [...] en España de momento no invertimos" — **[SIN VERIFICAR]**: no explican el motivo fiscal concreto, solo lo mencionan.

### Producto "Value Partner" — el cliente financia una promoción entera

**[CITA]** (líneas 1093-1120): compran un local (ejemplo: 500.000 €) e invierten un capex similar (500.000 €) para sacar 10 viviendas — 1 millón € de inversión total para vender 10 pisos a 120.000 € cada uno; el "value partner" (cliente que pone el capital) se lleva "entre un 15 y un 20%" de rentabilidad en 12-16 meses. Ahora tokenizado desde 25.000 € de entrada (línea 4067-4069).

### Riesgos y límites que reconocen ellos mismos

**[CITA]** (líneas 3984-3999): sobre hacer *fix and flip* por cuenta propia — "no sería el primer tipo de inversión que haría en mi vida [...] para mí eso no es inversión inmobiliaria, para mí eso es una actividad económica, es un negocio" — **[INFERENCIA]**: están desaconsejando activamente la vía que más se parecería a "hacerlo uno mismo con datos públicos", justo el objetivo de este proyecto.

**[CITA]** (líneas 1348-1351): el porcentaje de inmuebles que se alquilan por debajo de la renta estimada está "por debajo del 2 o 3%" — dato auto-reportado, sin auditoría.

**[CITA]** (línea 1370-1374): reconocen haber tenido que "desinvertir" (revender) inmuebles ya comprados para un cliente porque "para nuestro cliente no era" — no se especifica con qué frecuencia.

## Tercera fuente: "Cómo PropHero encuentra las zonas..." (`otros`)

Fuente: `fuentes/prophero-transcripciones-2026-09-29/otros/Cómo PropHero encuentra las zonas con mayor potencial de crecimiento antes que nadie.txt` (241 líneas, la más corta, leída completa — es la única de las tres donde el nombre de la empresa aparece parcialmente reconocible, como "Propgiro"/"Prop Gir"/"Prof. Hero", mismo problema de transcripción que en las otras). Parece un monólogo del fundador (primera persona), no una entrevista a dos voces.

### El origen del modelo, en primera persona

**[CITA]** (líneas 27-44):
> "Cuando abrimos en Prop Gir en España, empezamos a comprar bases de datos, a comprar datos de información que considerábamos que eran importantes [...] Estos datos eran datos como la renta por cápita, los datos de demografía, cuánto crecía o decrecía una población, datos de impagos, datos de ocupaciones, datos de liquidez de venta y liquidez de alquiler [...] la suma de todo esto era lo que configuró nuestro modelo reactivo."

**[CITA]** el nombre técnico de su departamento de datos (líneas 84-89 — "Datax" tal cual lo transcribe el fichero, no está claro si es un nombre propio real o un error de transcripción de otra cosa):
> "hoy somos la única empresa que hay en España de inversión inmobiliaria que tiene un departamento de Datax y este departamento lo que está realizando es un modelo predictivo"

**[INFERENCIA]**: esta es la tercera confirmación independiente (de tres transcripciones distintas) del mismo relato — modelo reactivo primero, predictivo con IA después — así que en este punto concreto las tres fuentes son consistentes entre sí, algo que no pasa con casi ninguna cifra numérica.

### El ejemplo aritmético que usan para explicar el déficit de vivienda

**[CITA]** (líneas 51-63):
> "Si en una población viven 100.000 personas y la población crece un 2%, el año que viene vivirán 102.000. Si el stock de viviendas no crece o incluso decrece [...] habrá necesidad de 1.000 viviendas nuevas. ¿Me puedes decir dónde se están construyendo 1.000 viviendas anuales en una población de 100.000 habitantes? [...] en ningún sitio. En España, por lo menos, en ningún sitio."

### Casos concretos — con una contradicción de cifra frente a las otras dos fuentes

**[CITA]** Molina de Segura (líneas 161-177): de 70.000 habitantes (2016) a ~76.100 (2022) — la transcripción da el número roto ("76.000, 1000"), se interpreta como 76.100 por el cálculo que hacen a continuación (+6.000 en 6 años); renta disponible de 21.978 € a 23.600 €.

**[CITA]** Ocaña (líneas 207-209):
> "Ocaña ha pasado prácticamente de 10.000 habitantes a 14.700"

**[CONTRADICCIÓN] entre las tres fuentes sobre el mismo pueblo**: esta transcripción dice que Ocaña partía de **10.000** habitantes; la transcripción de `tengo 1 plan 2` decía **8.000-9.000**; la de `tengo 1 plan 1` decía **8.000**. El número final (14.000-14.700) es consistente entre las tres, pero el de partida no — con lo cual el "creció un X%" que cada una calcula a partir de su propia cifra tampoco coincide.

### Zonas de inversión citadas en este vídeo

**[CITA]** (líneas 124-140): corredor mediterráneo — Castellón, provincia de Valencia (zona de costa), Alicante, Murcia (incluida Cartagena), Zaragoza, norte de Toledo/sur de Madrid/corredor de Henares, Jerez.

**[CITA]** (líneas 143-151) — reconocen que su propia lista de zonas caduca:
> "dentro de 6 meses [puede que] no tengamos nuevas zonas o algunas de estas hayamos dejado de invertir [...] porque los precios ya serán excesivamente caros y ya no tendremos buenas rentabilidades o buenas revalorizaciones"

**[CITA]** (líneas 224-233) criterio geográfico general que dan al cerrar: las "buenas ubicaciones" son donde la gente se agrupa para vivir — en España, "toda la costa mediterránea, en Madrid y alrededor, en Zaragoza, en Granada, en Sevilla y en la zona de Cádiz y en el País Vasco" — y advierten aparte eliminar las que ya son demasiado caras.

## Cuarta fuente: "Cómo Invertir tu Dinero hoy..." (`otros`)

Fuente: `fuentes/prophero-transcripciones-2026-09-29/otros/Cómo Invertir tu Dinero hoy Vivienda, rentabilidad y oportunidades PropHero @ac2ality.txt` (1.893 líneas, leída completa). Entrevista a **Pablo (fundador) y Jaime Hill (CEO de España)** — aquí sí se transcribe bien "Prop Hero" en el título del vídeo y varias veces en el texto (aunque también aparece "Progiro"/"Brgiro" en boca de los hablantes, mismo problema de transcripción automática que en las otras tres fuentes).

### Ancla de fecha — amplía la cronología

**[CITA]** (líneas 154-156, 169-170):
> "¿Facturáis 50 millones en 2025? [...] 35 [millones] en 2025 cerraremos en facturación [...] Haremos, hemos hecho 4 años en julio. 4 años y medio llevamos de vida."

**[INFERENCIA]**: esta transcripción es de **algún momento de 2025**, antes de cerrar el año (hablan de lo que "cerraremos" en 2025 en futuro). Con esto, la cronología reconstruida queda: `otros/Cómo PropHero encuentra las zonas...` (sin fechar, posible 2022-2023) → `tengo 1 plan 1` (~2024) → **esta transcripción (~2025)** → `tengo 1 plan 2` (septiembre 2026). Además, "4 años y medio de vida" en 2025 sitúa la fundación de la empresa hacia **finales de 2020 o 2021** — coincide con lo ya verificado en la web oficial ("activa desde 2021").

### Finanzas de la empresa — el dato más concreto de las cuatro fuentes

**[CITA]** (líneas 37-39, 154-166, 252-270):
> "Tenemos en torno a 500 personas más o menos. Esto tenemos un margen del 43% ahora mismo" [...] "35 [millones] en 2025 cerraremos en facturación [...] si tú multiplicas lo que hemos hecho este quarter pasado por cuatro, te saldrán 50 millones" [...] "nosotros cuando te digo la facturación son fee de Prophero, no es la transacción del inmueble. Si contáramos la transacción del inmueble, te podría estar hablando de 150 millones o 200 millones"

**[INFERENCIA]**: distinguen con cuidado **facturación propia** (comisiones: ~35M€ cierre 2025, ~50M€ run-rate anualizado del último trimestre) de **GMV/volumen transaccionado** (150-200M€, la suma del valor de los inmuebles que gestionan, que no es dinero de Prophero). Margen bruto 40-45%.

**[CITA]** (líneas 460-463): "La empresa ahora ya es break even, somos free cash flow positivo y no necesitamos el dinero de inversores" — **[SIN VERIFICAR]**: autodeclarado, sin cifras que lo respalden.

**[CITA]** origen y financiación (líneas 490-503): fundada con 0 € de capital; Pablo y su socio Michael pusieron 20.000-30.000 € cada uno; a los 5-6 meses levantaron una ronda semilla de **1 millón de euros de tres fondos australianos**.

**[CITA]** (líneas 1733-1741): "el mínimo al que se podría vender ahora son 100 millones, porque fue lo de la última ronda [de financiación] [...] sin necesidad de aprobación del consejo" — **[INFERENCIA]**: esto es una cláusula contractual de la última ronda (umbral de venta que no necesita aprobación del consejo), no necesariamente la valoración de mercado actual de la empresa.

### Plantilla — corrobora la otra fuente casi exactamente

**[CITA]** (líneas 552-563): "entre 160 y tantos" empleados internos; "si viéramos el entorno que mueve Progiro [...] estaríamos en torno a 500 personas" contando partners. **[INFERENCIA]**: coincide casi exactamente con `tengo 1 plan 1` ("150 personas [...] hasta 180 con becarios") — buena corroboración cruzada, no contradicción.

### Países donde operan — confirma el número exacto

**[CITA]** (líneas 936-949): "ahora estamos en cuatro países [...] Australia, que es donde fundamos, España, Indonesia y Irlanda [...] desde España, normalmente nuestros clientes invierten en España, en Indonesia y en Irlanda."

### El caso Sagunto — nueva fuente lo confirma y añade el dato que faltaba

**[CITA]** (líneas 1134-1150) — **esto resuelve el "[SIN VERIFICAR]" que dejé abierto en `tengo 1 plan 1` sobre qué empresa monta la "giga de baterías" en Sagunto**:
> "Sagunto, que es una de las zonas donde mejor hemos invertido [...] Sagunto es un pueblo que está entre Valencia y Castellón [...] sería muy equiparable a la zona aquí de Ocaña e Illescas [...] en Valencia se anunció que iba la fábrica de **Volkswagen** allí con **12.000 puestos de trabajo**, se ha puesto el centro logístico de Mercadona, Inditex [...] aún ni abierto y lleva 3 años creciendo el pueblo precios a más del 20%."

**[INFERENCIA]**: con esto, "la giga de baterías" que mencionaba `tengo 1 plan 1` sin nombrar la empresa queda confirmada como la gigafactoría de baterías de **Volkswagen** en Sagunto (proyecto público y verificable de forma independiente, fuera del alcance de esta ronda de ingesta). Esta fuente también confirma explícitamente que agrupan Sagunto con Ocaña e Illescas como el mismo tipo de zona (subyacente industrial + efecto Madrid/Valencia).

### El criterio de rentabilidad, resumido con más claridad que en ninguna otra fuente

**[CITA]** (líneas 1488-1519):
> "¿Qué rentabilidades da el mercado inmobiliario en España? Conseguir más de un 5 y medio o 6% de rentabilidad en la renta que tú obtienes de un piso es complicado [...] la revalorización de un inmueble los últimos 3 años ha estado en un 7% [...] En Prophero hemos estado entregando 6 y medio, si pores al principio estamos dando un 5 y medio [...] los tres años que llevamos en Prophero en España [...] hemos revalorizado los inmuebles que los clientes han invertido con nosotros un 13% de media, 13,1 [...] rentabilidad 6-7% de las rentas, 10% de revalorización. Eso sería una buena rentabilidad en real estate."

**[INFERENCIA]**: esta es la formulación más clara y compacta del criterio en las cuatro fuentes: **~6-7% de rentabilidad por alquiler + ~10% de revalorización ≈ 16-17% de retorno combinado**, frente a un mercado general de España de ~5,5-6% de renta y ~7% de revalorización anual. La cifra de revalorización propia (13%, 13,1%) es muy cercana a la de `tengo 1 plan 1` (13,7%) — buena corroboración, no contradicción, a pesar de ser fuentes de años distintos (2025 y 2024 respectivamente).

**[CITA]** selectividad (líneas 1250-1254): "estamos analizando [...] lo que acabamos cogiendo es un 1% de todo lo que analizamos."

### Las fases de entrada a una zona — versión más detallada, posible refinamiento del modelo de "tres fases"

**[CITA]** (líneas 1682-1695):
> "va por fases [...] al principio compras inmuebles sueltos, luego empiezas a hacer cambios de uso de locales, de almacenes [...] luego empiezas a las estructuras que será una fase tres de edificios que se quedaron y ahora empieza la fase cuatro que sería la obra nueva"

**[INFERENCIA]**: son **4 fases por tipo de producto** (piso suelto → cambio de uso de local → edificio vandalizado → obra nueva), distintas de las "3 fases por riesgo/ticket" que describía `tengo 1 plan 1` (pionero/riesgo alto → consolidación → maduro/riesgo bajo). No están necesariamente en contradicción — podrían ser dos ejes distintos del mismo proceso (qué tipo de producto compran vs. qué riesgo asumen) — pero ninguna fuente los conecta explícitamente entre sí.

### Un dato que ellos mismos marcan como no fiable — otro ejemplo de honestidad sobre sus propios datos

**[CITA]** (líneas 1310-1321): sobre la diferencia de género entre inversores — "los datos son un poco están intoxicados en el sentido de que muchas veces invierten matrimonios, pero tengo el nombre de un cliente, o vienen con sociedades y detrás no sabes lo que hay" — reconocen que su propio dato de género de inversor (70/30 o 60/40 según quién responda) no es fiable porque el titular registrado no siempre refleja quién decide o aporta el capital.

### Competencia

**[CITA]** (líneas 1184-1195, 1206-1233): afirman que ninguna plataforma en España vende más de 20-30 inmuebles al mes, frente a los ~200/mes de Prophero; describen a las plataformas de tokenización de inmuebles como clientes/partners a la vez que competencia — les compran activos ya generados por Prophero para tokenizarlos.

## Quinta fuente: "Inventaron Esta Forma..." Ep 74 (`otros`)

Fuente: `fuentes/prophero-transcripciones-2026-09-29/otros/Inventaron Esta Forma Para Crear Tu Patrimonio Desde 0 (Prop Hero) Ep 74.txt` (3.738 líneas, leída completa). **Aviso de formato, importante para no atribuir mal las citas**: es un podcast a **tres voces** (parece "BLV", episodio 74) — Jaime de Prophero (aquí otra vez transcrito como "Prop Giro"/"Progiro") más un presentador y un **tercer invitado ajeno a Prophero**, un inversor veterano apodado "Judas" que tiene su propia cartera de 12-13 pisos, hace "renta vitalicia" (nuda propiedad) y "Bridge" (préstamos puente), y también participa en una empresa de energía cotizada. Gran parte del fichero (aprox. líneas 1330-2900) es la historia y opinión de ese tercer invitado, no de Prophero — se cita solo lo que es claramente de Jaime/Prophero.

### Fi transaccional — una tercera cifra distinta, refuerza el patrón de "cambia con el tiempo"

**[CITA]** (líneas 2364-2372, 2566-2568):
> "nosotros cobramos 6000 incluyendo IVA eso es el fi directo el fi transaccional [...] me pagas 1000 en arras 2500 y en notaría 2500 más"

**[CONTRADICCIÓN]**: 1.000+2.500+2.500 = **6.000 €**, una tercera cifra distinta de las dos ya vistas (7.000 € en `tengo 1 plan 2`, 7.500 € en la web/vivirdeinmuebles). Con tres cifras distintas en tres fuentes distintas y sin fecha fiable para esta transcripción, no se puede establecer una secuencia temporal limpia como sí se hizo con el umbral de rentabilidad — se deja como contradicción sin resolver entre las tres.

### Equipo de datos — el único número concreto de plantilla del departamento

**[CITA]** (líneas 2650-2654):
> "tenemos un departamento de Data con tres personas que se le mete Inteligencia artificial"

**[INFERENCIA]**: contrasta con la escala de "200 millones de puntos" y "500 variables" que se atribuye a ese mismo departamento en `tengo 1 plan 2` — un equipo de 3 personas gestionando un modelo de ese tamaño no es necesariamente increíble (puede apoyarse en infraestructura/vendors externos), pero es un dato a tener en cuenta al valorar qué tan artesanal o industrializado está el proceso.

### Track record de precisión — otra métrica de exactitud, distinta a la ya vista

**[CITA]** (líneas 2536-2544):
> "estamos entregando 0,4 más de lo que estimo [...] esa es mi garantía de los últimos 400 [...] 0,4 más hemos entregado"

**[INFERENCIA]**: parece significar que, de media, el alquiler real entregado en los últimos 400 pisos fue "0,4 puntos" (¿porcentuales? ¿cientos de euros?) por encima de lo estimado — la unidad no queda clara en el propio audio. Complementa (no contradice necesariamente) el dato ya visto en `tengo 1 plan 1` de que menos del 2-3% de los pisos se alquilan por debajo de la renta estimada.

### Plantilla y volumen — más corroboración cruzada

**[CITA]** (líneas 2332-2354): "somos 150 ahora en la empresa en España [...] global no, global somos casi 200, llega 180" — y sobre transacciones: "España 110 el mes pasado, hicimos 150 [el actual]". **[INFERENCIA]**: sigue corroborando el rango 150-180/200 empleados visto en las otras dos fuentes, sin contradicción.

### Sagunto y Volkswagen — cuarta confirmación independiente

**[CITA]** (líneas 2458-2462):
> "Ven a invertir en sagunto que va a la fábrica de baterías van a currar 12000 tíos y los precios van a crecer al veintitantos por anual"

**[INFERENCIA]**: coincide casi palabra por palabra con la cifra de "12.000 puestos de trabajo" de Volkswagen dada en la cuarta fuente (`Cómo Invertir tu Dinero hoy...`) y con el crecimiento de precios ">20% anual" — con esta ya son **tres transcripciones distintas** (más esta) que corroboran Sagunto/Volkswagen con cifras consistentes. Es el dato mejor corroborado de todo el corpus.

**[CITA]** zonas confirmadas de nuevo (líneas 2470-2480): Castellón/Sagunto, Molina de Segura (Murcia), Alicante, Valencia costa, Cartagena, Toledo (sobre todo Toledo Norte), Zaragoza, Jerez.

**[CITA]** exclusión explícita — dato nuevo (línea 2606-2608): preguntados por invertir en Reus (Tarragona), responden "a día de hoy en Cataluña no estamos" — **[SIN VERIFICAR]**: no dan el motivo.

### El criterio de scoring, en su versión más corta de las cinco fuentes

**[CITA]** (líneas 2626-2634):
> "crece en población. Ese es el número uno [...] número dos: renta por cápita / tasa de desempleo [...] y luego que hay un subyacente que justifique eso: una empresa, un centro logístico, una conexión"

**[INFERENCIA]**: es la formulación más simple de las cinco fuentes — 3 factores, sin las cifras concretas (4-5% neto, top 10%) que sí da `tengo 1 plan 2`. Coherente con las demás, solo que sin números.

### Producto futuro: media estancia / coliving

**[CITA]** (líneas 3157-3197): dicen que van a apostar por "media estancia" y modelos tipo coliving con servicios; primera obra nueva propia de ese tipo cerca de Valencia, "82 viviendas", tickets de inversión de ~140.000 €, rentabilidad neta objetivo "7 y medio" (7,5%) más revalorización.

### El problema de financiación que reconocen como uno de sus mayores frenos

**[CITA]** (líneas 3270-3289): "los bancos no están muy contentos haciendo hipotecas de 50.000 [...] es el problema que tenemos para mí uno de los pains es encontrar fluidez en la financiación para esas transacciones [...] uno de los [problemas] más grandes que tenemos como empresa es la escalabilidad, aquí que no hay, los bancos no les gusta financiar nuestro producto." Están montando un "vehículo" propio para financiar colectivamente obra nueva (líneas 3339-3345), sin más detalle concreto todavía en esta fuente.

## Sexta fuente: "Experto Inmobiliario ¡El Método SECRETO...!" (`otros`) — la más clara de las seis

Fuente: `fuentes/prophero-transcripciones-2026-09-29/otros/Experto Inmobiliario ¡El Método SECRETO Para Multiplicar tu Patrimonio Sin Pedir Hipotecas!.txt` (3.905 líneas, leída completa). Podcast **"Inversión Racional"**, entrevista a solas a Jaime Hill — sin terceros invitados, sin tramos personales largos. Es la fuente más pedagógica y mejor estructurada de las seis: el propio presentador pide explícitamente "vamos a ir al turrón" y organiza la conversación por bloques. **Nota**: coincide con la que ya había catalogado otra sesión de trabajo en este mismo proyecto (`fuentes-prophero.md`) como publicada también en audio puro (Ivoox, *Inversión Racional #132*) — mismo contenido, no una fuente adicional.

### Ancla de fecha adicional, consistente con `tengo 1 plan 1`

**[CITA]** (líneas 99-101): "hasta que hace 3 años y medio pues se cruzó el fundador de Prop Hero y me lió en este proyecto." **[INFERENCIA]**: si se cuenta desde la fundación de la empresa (~2021, verificado en la web oficial), esto sitúa la grabación **alrededor de 2024-2025**, en la misma franja que `tengo 1 plan 1`.

### La comisión de 7.500 € — confirmación directa de Jaime, coincide con la web oficial

**[CITA]** (líneas 3211, 3285-3301):
> "pagas el engagement fee [...] El servicio completo vale 7500 € que te incluye el que te encuentre una oportunidad, que te firme un contrato de arras, que lo llevemos a esta notaría, preparamos la notaría, hacemos la reforma, lo limpiamos, lo amueblamos y te lo alquilamos [...] De esos 7500, 1000 van el primer día"

**[INFERENCIA]**: esta es la **única de las seis transcripciones donde Jaime da la misma cifra que la web oficial** (7.500 €, verificada al principio de este documento en `help.prophero.com`). Las otras dos cifras vistas en otras transcripciones (7.000 € y 6.000 €) quedan como variantes sin resolver — puede que 7.500 € sea la tarifa "oficial" de referencia y las otras sean simplificaciones orales o momentos distintos, pero no hay manera de confirmarlo con lo que dicen.

**[CITA]** gestión de alquiler (líneas 3385-3401): "un property management completo, pues va entre un 4 y un 5%" — **[CONTRADICCIÓN]** con el 5-7% verificado en la web oficial al principio de este documento; rango solapado pero no idéntico.

### El criterio de rentabilidad neta explicado con más pedagogía que en ninguna otra fuente

**[CITA]** (líneas 2649-2735):
> "para nosotros una rentabilidad entre un [cinco] y un 6 neto es [...] adecuado [...] un 5 6% está bien hoy, a partir del 5 por arriba está bien [...] Pero luego tienes la renta bruta [...] hay que descontarle todos los gastos corrientes, la comunidad, el seguro, el que gestiona el inmueble [...] Si cobro 800 menos 150 o menos 100, ya cobro 700 netos. 700\*12 dividido entre la inversión total. Eso es una rentabilidad neta."

**[INFERENCIA]**: esta es la **cuarta cifra distinta de umbral mínimo de rentabilidad neta** entre las seis fuentes (4-5% en `tengo 1 plan 2`/2026, 6% en `otros/Cómo PropHero...`, 5-6% aquí, y el "8% hace años" citado como pasado). No se puede reconstruir una secuencia temporal limpia con las seis a la vez — la cronología solo se pudo fijar con precisión para dos de las seis fuentes (2024 y 2026).

**[CITA]** advertencia explícita sobre tablas de rentabilidad engañosas (líneas 2686-2700): "yo veo unas tablas por ahí que dice, 'No, rentabilidad neta el 8 y 5%.' [...] hoy en día me cuesta creer mucho que alguien lo obtenga."

### La revalorización, con el desglose más completo de las seis fuentes

**[CITA]** (líneas 2817-2844):
> "Prof Kiro desde que empezó hasta hoy tiene una revalorización media de un 13,7 sus activos [...] España crece los últimos 3 años a una media de un siete. Las área cluster que llamamos nosotros, las zonas que tenemos abiertas de inversión crecen de media un 10 [...] nuestros inmuebles se revalorizan el doble que la media del país"

**[INFERENCIA]**: confirma exactamente el 13,7% ya visto en `tengo 1 plan 1` (misma cifra, corroboración fuerte) y el 7% de media nacional visto en varias fuentes. Introduce un tercer nivel intermedio ("área cluster", 10%) no mencionado antes. Nota aritmética: dicen "el doble que la media del país" pero 13,7 no es el doble de 7 (sería 14) — están redondeando, no es una contradicción grave.

**[CITA]** explicación pedagógica del ciclo de revalorización con forma de campana de Gauss (líneas 2744-2773): una población no puede crecer al mismo ritmo indefinidamente; lo ideal es comprar cuando el crecimiento está empezando a acelerar, no cuando ya lleva años acelerado.

**[CITA]** caso Sagunto con series de precios completas (líneas 2784-2795) — la serie más detallada de precios de todo el corpus:
> "los pisos de Sagunto, el típico tercero sin ascensor valían 25, 27, 31 [...] no compraba nadie [...] Y esos mismos pisos cuando entró Prop Giro a Sagunto estaban en 48, 50, 52. Cuando nosotros entramos hace 3 años y medio. Esos pisos hoy valen el mismo, 125, 130."

**[INFERENCIA]**: serie temporal aproximada para el mismo tipo de piso en Sagunto — de ~25-31.000 € (pre-Prophero, sin fecha) a ~48-52.000 € (entrada de Prophero, "hace 3 años y medio") a ~125-130.000 € (hoy). Es el dato de revalorización más verificable de todo el corpus si se pudiera datar con precisión el "hoy" de esta grabación.

### Casos concretos nuevos, con cifras

| Lugar | Datos que citan | Línea |
|---|---|---|
| Toledo (primer edificio de Prophero ahí) | pisos a 50.000+ € (reformados, construcción de 2008), hoy (3 años después) valen 110.000 € | **[CITA]** 445-451 |
| Horta Nord (Valencia) | piso a 40.000 € hace 7 años, hoy 150.000 € (~3,75x) | **[CITA]** 452-455 |
| Jaén (contraejemplo — rentabilidad alta pero sin crecimiento) | Expansión la señaló como "provincia más rentable de España" (6-6,1% bruto vs. 5,4% de Madrid); Jaime la descarta explícitamente porque la rentabilidad alta viene solo de precios bajos, no de crecimiento — "no invierto ni en Jaén ni en Madrid a día de hoy" | **[CITA]** 399-414 |
| Oliva (Valencia) — caso de "última milla" humana, no solo data | estancada por mal acceso; un socio local avisó de una nueva salida de autovía en construcción; Prophero entró antes de que el dato lo reflejara | **[CITA]** 1316-1345 |
| Moncofar (Castellón) — dato que su propio modelo no contemplaba | descubierto por casualidad (conversación con una peluquera): zaragozanos comprando apartamentos de ~100.000 € para pasar el invierno allí — reconocen que "no lo tenía contemplado en mi modelo de datos" | **[CITA]** 1398-1419 |
| Rafael/Buñol (Valencia) y otros | poblaciones descartadas explícitamente por lentitud o rigidez del ayuntamiento en licencias, no por criterio de mercado | **[CITA]** 1704-1724 |

### El criterio de scoring, ordenado paso a paso — la versión más completa

**[CITA]** (líneas 1036-1068, 1093-1113), en el orden que da Jaime:
1. Evolución de la población del municipio y de su comarca en los últimos años (crece/decrece).
2. Cuánta industria se está generando alrededor.
3. Renta per cápita de la zona comparada con la media nacional — si crece más que la media, indica llegada de población de mayor poder adquisitivo.
4. Que haya un "subyacente" concreto que lo explique (fábrica, centro logístico, conexión de transporte).

**[CITA]** advertencia sobre un uso ingenuo de Idealista (líneas 1120-1137): el pueblo que en Idealista aparece como "mejor rentabilidad" a veces solo lo es porque los pueblos de alrededor están caros y la gente vive ahí temporalmente por contagio de precio, no porque tenga un subyacente real — hay que comprobar si ese pueblo tiene su propio motivo de crecimiento, no solo copiar el ranking del portal.

### Producto para grandes patrimonios ("Wealth") — nuevo segmento no visto en otras fuentes

**[CITA]** (líneas 3517-3562): a partir de 500.000 €, departamento "Wealth" con dos productos — (1) comprar un edificio entero apalancado para patrimonializar, o (2) un fix&flip garantizado: compran un edificio, lo reforman, y garantizan la recompra al inversor con "un 18 y un 20%" de rentabilidad en 12-14 meses, estructurado como compraventa con recompra, no como préstamo participativo (para que cuente como actividad económica a efectos fiscales, no como patrimonio pasivo).

### Otros datos operativos

**[CITA]** (líneas 3343-3345): "el 60% de nuestros clientes compran al contado a día de hoy" — sin financiación bancaria.
**[CITA]** (líneas 3277-3286): atienden "250 clientes" cada mes que pagan la cuota de entrada (1.000 €) — límite operativo explícito de capacidad de atención.
**[CITA]** (líneas 181-198): reafirma cuatro países (aquí solo nombra tres explícitamente: Australia, Irlanda, España — omisión, no contradicción, ya que Indonesia consta en otras fuentes) y que para el año siguiente esperan estar "en el top cinco de desarrolladoras" de obra nueva del país.

## Fuentes disponibles

- `fuentes/prophero-transcripciones-2026-09-29/` — 8 transcripciones de vídeos de YouTube sobre Prophero e inversión inmobiliaria, aportadas por el usuario.
  - **Ingeridas completas: las 6 de texto (6 de 6)**: `tengo 1 plan 2/Experto en Vivienda...txt`, `tengo 1 plan 1/2 Expertos en inversión...txt`, `otros/Cómo PropHero encuentra las zonas...txt`, `otros/Cómo Invertir tu Dinero hoy Vivienda...txt`, `otros/Inventaron Esta Forma...Ep 74.txt`, `otros/Experto Inmobiliario ¡El Método SECRETO...txt`.
  - Los `.srt`/`.sbv` de `tengo 1 plan 2` son el mismo contenido que su `.txt` ya ingerido — no aportan nada nuevo, no se procesan aparte.
  - **Las 8 transcripciones aportadas por el usuario quedan cubiertas.**

## Estado

2026-09-29: objetivo confirmado por el usuario — replicar el "buscador" de Prophero. **Ingesta de las 8 transcripciones completada** (6 ficheros de texto únicos, lectura completa línea a línea, dos duplicados de formato sin contenido nuevo). El usuario pidió explícitamente que cada dato quede etiquetado (cita/inferencia/sin verificar/contradicción) para que no se cuele nada mal clasificado — aplicado a todo el documento. Reconstruida una cronología aproximada a partir de citas internas de las propias transcripciones (~2022-2023 sin confirmar, 2024, 2025, septiembre 2026), que explica parte de las diferencias numéricas entre fuentes y deja otras como contradicción real sin resolver. Un error de atribución cruzada entre fuentes (caso Almazora) fue detectado por el usuario y corregido, con la lección registrada en memoria (`feedback_verificar_cita_en_el_momento`). En paralelo, otra sesión de trabajo levantó un catálogo de fuentes externas sobre Prophero (`fuentes-prophero.md`) — investigación complementaria, no ingesta de transcripciones.

## Próximos pasos

- **Confirmar con el usuario el alcance siguiente**: con las 8 transcripciones ya ingeridas, decidir si se sintetizan en un criterio de scoring único y consolidado, si se revisan las fuentes externas que catalogó la otra sesión (`fuentes-prophero.md`), o si se pasa directamente a explorar qué datos públicos españoles (INE, catastro, Idealista, registro notarial) harían falta para replicar el cálculo con municipios reales.
- Contradicciones reales que quedan sin resolver, para tener en cuenta en cualquier síntesis: umbral de rentabilidad neta mínima (4-5% / 5-6% / 6% / "8% hace años", sin secuencia temporal limpia con las 6 fuentes a la vez); comisión total (6.000 € / 7.000 € / 7.500 €, esta última confirmada dos veces — web oficial y una transcripción — como la más probablemente vigente); población de partida de Ocaña (8.000 / 9.000 / 10.000, no explicada por la fecha).

## Relacionado en el wiki

- [[apartamentos-calle-uruguay]] — inversión inmobiliaria ya en marcha del usuario; referencia de contexto, no parte de este método
- [[fiscalidad-alquiler-por-habitaciones]] — fiscalidad del alquiler que cualquier cálculo de rentabilidad neta tiene que incorporar
- [[patrimonial]] — dashboard de patrimonio donde acabaría reflejándose cualquier inmueble que resulte de este proyecto
