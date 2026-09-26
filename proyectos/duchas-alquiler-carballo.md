---
title: Grifería de ducha gris para alquiler (Carballo)
created: 2026-09-24
updated: 2026-09-26
tags: [alquiler, bano, griferia, compras, investigacion]
zona: tecnico
---

Grifería de ducha en gris para un piso de alquiler en Carballo (A Coruña), clima húmedo costero — el gris incluye cromado (corrección del 2026-09-26, ver sección 0.1). Dos opciones: conjunto completo (grifo + barra + rociador + teleducha) por 90-100 € como máximo, o grifo + alcachofa de colgar (sin barra ni rociador fijo) por 80 € como máximo. Investigación de negro (100 €/60 €, 2026-09-24) descartada por el usuario: sustituida entera por esta, en gris, el 2026-09-26. Ganadores reales tras la corrección: **New Boreal Termostático (Obramat, 97 €)** para el conjunto, **Ramon Soler Tarraco (Obramat, 41 €)** para grifo+alcachofa de colgar — ver sección 0.1.

## 0. Hallazgo principal: el gris barato de verdad no existe, y el material no es el problema que parecía

El usuario pidió evitar el latón. **El latón macizo no es el enemigo**: es el material tradicional de fontanería, resiste bien la corrosión por sí mismo y es lo que llevan casi todas las marcas serias (Roca, Ramon Soler, Grohe, Hansgrohe). Lo que se descascarilla o se oxida con la limpieza no es el latón, es el **recubrimiento** barato que se le aplica para teñirlo de un color que no es el suyo (pintura electrostática, lacado, cromado de baja calidad). Esto ya se documentó para el negro en la revisión anterior de esta misma nota (ahora en el historial de git): [Imex Génova PVD, 176 € en Leroy](https://www.leroymerlin.es/productos/set-ducha-monomando-genova-negro-cana-de-fusil-laton-y-zama-con-acabado-pvd-imex-78804002.html) es de los pocos con PVD (deposición física de vapor, un recubrimiento que se funde con el metal y no se levanta) por debajo de 200 €; todo lo demás por debajo de 60-80 € es pintura o lacado, con la misma probabilidad de fallo tenga latón o zamak debajo.

Lo que sí cambia con el material de base:

- **Latón (CuZn, H59) + cartucho cerámico de 35 mm**: estándar de calidad, aguanta décadas si el recubrimiento aguanta.
- **Zamak / "aleación de zinc"**: más barato, más ligero, se pica y corroe en cuanto el cromado se raya. Evitarlo.
- **"Aluminio espacial"**: término de marketing de fabricantes chinos baratos (Bostar, Auralum) para aleaciones de aluminio moldeado a presión — mismo problema que el zamak, coeficiente de corrosión bajo si el recubrimiento falla.
- **Acero inoxidable (304 o 316L) sin pintar, acabado satinado/cepillado**: el color gris-plata **es el propio metal**, no una capa. No hay pintura que se caiga porque no hay pintura. Es la opción más honesta para "que no se oxide ni se estropee al limpiar", pero en el mercado español se vende casi siempre como producto de gama alta o de exterior/jardín (ver sección 2), no como grifería de baño barata.

Para el gris (antracita, titanio, gunmetal) la conclusión de esta investigación es más dura que la del negro: **por debajo de 90 €, en Amazon.es no hay ni un solo grifo o columna de ducha gris con la ficha de producto internamente consistente y sin reseñas críticas de material**. Cada candidato que apareció en las búsquedas tenía al menos una señal de alarma: la ficha técnica contradice el título (dice "Cromo" cuando el título dice "gris titanio"), el fabricante llama "aluminio" a lo que las reseñas describen como plástico, o el listado no tiene ninguna reseña que lo respalde. El detalle de cada caso está en la sección 2.

> **Esta conclusión queda anulada en parte por la sección 0.1: tratar "Cromo" como contradicción con "gris" fue un error.** El cromado es un gris (plateado brillante) — no una señal de alarma. Al aplicar esa corrección aparecen ganadores reales en el catálogo profesional de Obramat (Roca, Ramon Soler, Teka), fuera de Amazon. Los descartes de esta sección 2 que se basaban solo en el color quedan sin efecto; los que se basaban en material (zamak, "aluminio espacial") o en reseñas críticas de rotura siguen siendo válidos.

## 0.1 Corrección (2026-09-26, misma tarde): el cromado sí es gris

**Error cometido y corregido.** Traté "la ficha dice Cromo, el título dice gris" como una contradicción y descarté de raíz varios candidatos por ese motivo — incluido el catálogo entero de Obramat (Roca, Ramon Soler, Tres, Drake), que nunca se evaluó en detalle. El cromado **es** un gris: plateado brillante, sin ambigüedad. Fallo de comportamiento, no del producto — corrección registrada en `areas/decisiones.md` y en la regla global `~/.claude/rules/comportamiento.md` («Preguntar ante lo ambiguo o lo raro»).

Re-investigado con navegador real contra el catálogo de Obramat (192.168.1.5, 2026-09-26), verificando ficha técnica producto a producto — Obramat no publica reseñas de clientes en ninguna ficha, así que el filtro aquí es solo material/marca, no reseñas.

### Opción A revisada — conjunto completo, 90-100 €

| Producto | Precio | Material verificado | Notas |
|---|---|---|---|
| **[Columna de ducha termostático New Boreal](https://www.obramat.es/productos/columna-de-ducha-termostatico-new-boreal-10952046.html)** | **97 €** | Columna, flexo (1,5 m, grapado doble) y rociador superior (Ø25 cm) en **acero inoxidable**; mango de ducha en ABS (estándar en toda gama, incluida la de alta); acabado cromado; ficha propia lo cataloga `color: Gris/plata`. Picos de silicona anticalcárea | Termostático: necesita caudal/presión estables del calentador, no verificado para este piso — mismo caveat que ya se documentó para los termostáticos baratos de Amazon en la investigación de negro |
| [Columna de ducha Teka Dual Control](https://www.obramat.es/productos/columna-de-ducha-teka-dual-control-10884846.html) | 115 € | Columna en acero inoxidable, acabado cromado, **flexo en latón** (1,5 m) — material superior al inox del New Boreal | 15 € por encima del tope de 100 €. TEKA es marca reconocida de saneamiento/electrodomésticos. Si se puede estirar el presupuesto, esta es la opción con mejor material de las dos |
| [Columna monomando Mercury Pro2](https://www.obramat.es/productos/columna-de-ducha-monomando-mercury-pro2-25069465.html) | 80 € | Acero inoxidable, acabado cromado, `color: Gris/plata` en ficha | **Descartada**: la ficha no menciona flexo, manguera ni mango de ducha — es solo rociador superior fijo con inversor, no lleva teleducha. No cumple "conjunto completo" |

**Recomendación A: New Boreal Termostático, 97 €.** Es el único dentro del tope de 100 € con cuerpo, flexo y rociador 100 % en acero inoxidable (no ABS, no zamak) y acabado cromado de fábrica — material real, no el menos malo de una lista de productos con grietas. Único caveat honesto: es termostático, y ni Obramat ni el fabricante confirman el caudal mínimo que necesita el calentador de este piso.

### Opción B revisada — grifo + alcachofa de colgar, 80 €

| Producto | Precio | Material verificado |
|---|---|---|
| **[Grifo ducha monomando Ramon Soler Tarraco](https://www.obramat.es/productos/grifo-ducha-monomando-ramon-soler-tarraco-25058962.html)** | **41 €** | Cuerpo y mango en **latón**, cartucho **cerámico Ø35mm**, acabado cromado. Incluye en el mismo producto: **flexo metálico de 1,70 m**, mango de ducha, **soporte de ducha articulado** y excéntricas — todo en una sola referencia, nada que combinar |
| [Grifo ducha Roca Nora](https://www.obramat.es/productos/grifo-de-ducha-monomando-roca-nora-10474842.html) (41,75 €) + [Kit ducha con soporte Roca Stella 100mm](https://www.obramat.es/productos/kit-de-ducha-con-soporte-roca-stella-100mm-3-funciones-25080251.html) (37,25 €) | 79 € | Grifo: latón, cartucho cerámico Ø40mm, cromado. Kit: alcachofa ABS cromado, **flexo de PVC** (peor que el metálico del Tarraco), soporte ABS |

**Recomendación B: Ramon Soler Tarraco, 41 €.** Gana claro sobre la combinación Roca: mismo presupuesto disponible, mejor material (flexo metálico frente a PVC), y una sola referencia de un fabricante español en vez de combinar dos productos. Queda 39 € de margen bajo el tope de 80 €.

**Comprobación en Amazon (pendiente original, cerrada ahora):** ni el Ramon Soler Tarraco ni el New Boreal Termostático están en Amazon.es bajo ese nombre — búsquedas [`ramon soler tarraco ducha`](https://www.amazon.es/s?k=ramon+soler+tarraco+ducha) (298 resultados, 0 coincidencias reales de "Tarraco": la única mención es el eco del propio término buscado en el cuadro de búsqueda; sí aparecen otras series de Ramon Soler — Titanium, Blautherm, Termotech — pero ninguna es el Tarraco) y [`columna ducha New Boreal termostatico`](https://www.amazon.es/s?k=columna+ducha+New+Boreal+termostatico) (431 resultados, ningún producto con ese nombre; solo columnas termostáticas de otras marcas) verificadas en vivo el 2026-09-26. Los dos ganadores de esta sección solo se consiguen en Obramat.

### Qué queda anulado del resto de la nota, y qué sigue en pie

- **Anulado por el error de color**: el descarte de [YRHome B0HK86YK7W](https://www.amazon.es/dp/B0HK86YK7W) (sección 2) por "contradicción cromo/gris" ya no aplica — pero sigue descartado por motivo independiente: 0 reseñas, sin marca reconocible, sin dato para confiar.
- **Sigue en pie sin cambios**: los descartes de INEX Europa, Bostar y demás marca blanca de Amazon (sección 2) por grietas, roturas, plástico vendido como aluminio y reseñas críticas de material — esos motivos no dependen del color y el cromado no los arregla.
- **Patrón que se repite**: igual que en la investigación de negro (donde ganó Obramat con Ponds+kit a 56,70 €), el catálogo profesional de Obramat vuelve a ganar sobre Amazon marca blanca y sobre la gama cara de Leroy/Bricodepot (Imex Line, 172 €+). La diferencia esta vez es que ese catálogo ya estaba delante en la búsqueda original y se descartó por error, no por no haberlo mirado.

## 1. Introducción

Qué se busca:

- **Opción A**: conjunto completo en gris con grifo, barra, rociador superior y teleducha, todo en una pieza (de superficie, no empotrado). Tope de 90-100 €.
- **Opción B**: grifo más alcachofa "de colgar" (teleducha con soporte y flexo, sin barra vertical ni rociador fijo), todo gris. Tope de 80 €.
- Criterio de material: cuerpo de latón macizo o acero inoxidable + cartucho cerámico de 35 mm; nunca zamak/aleación de zinc/"aluminio espacial"; si es latón u otro metal pintado, que el recubrimiento sea PVD o al menos no tenga reseñas de que se descascarilla.
- Gris significa antracita, titanio, gunmetal o gris pistola — no cromo ni plateado brillante, aunque varias fichas confunden ambos términos (ver sección 2).

Contexto: posible destino [[apartamentos-calle-uruguay]] (Carballo).

## 2. Considerado y descartado

| Descartado | Precio | Motivo |
|---|---|---|
| [INEX EUROPA Columna Negro Gris Titanio, INX 1551 BS](https://www.amazon.es/dp/B0DHC224FY) | 79,89 € | La ficha técnica dice **Color: Cromo** pese a que el título y las fotos son "gris titanio" — contradicción ya detectada en la revisión de negro. 571 reseñas, 4,4★, pero las críticas describen grietas, fugas, "solo ha durado 6 meses" con vídeo del cabezal reventado, y una reseña de plástico que se ha partido al limpiarlo ([reseñas críticas](https://www.amazon.es/product-reviews/B0DHC224FY/?filterByStar=critical&sortBy=recent)) |
| [Bostar Columna con Pantalla LED, Gris Titanio](https://www.amazon.es/dp/B0F5WPR75J) | 83,69 € | Sin bullets de producto (ficha vacía). Reseñas: "casi todas las piezas parecen más bien de plástico" pese a que la ficha dice aluminio, llegó roto de fábrica, llegó una unidad ya usada/devuelta ([reseñas críticas](https://www.amazon.es/product-reviews/B0F5WPR75J/?filterByStar=critical&sortBy=recent)) |
| [YRHome Sistema de ducha "gris"](https://www.amazon.es/dp/B0HK86YK7W) | 63,99 € | Los bullets del propio vendedor dicen tres veces "superficie cromada" y la ficha técnica dice "Color: Gris" — contradicción entre el propio texto de venta. Cero reseñas, cero garantía de que llegue en gris de verdad |
| [Bostar Grifo de bañera "Gris antracita", aluminio espacial](https://www.amazon.es/dp/B0GRGZR3S3) | 53,99 € | Solo 5 reseñas en total, todas 5★ sin texto — datos insuficientes para confiar. Es grifo de bañera con cascada, no un grifo de ducha puro |
| [Görbach columnas Gris Gunmetal cepillado auténtico](https://www.amazon.es/s?k=g%C3%B6rbach+ducha+gunmetal+cepillado&s=price-asc-rank) (139,99 €, 196,41 €, 237,93 €) | 140-240 € | Estas sí son gris gunmetal real en acero inoxidable, con 4,3-4,4★, pero todas muy por encima de los 90-100 € — y encima "sin grifo" en el modelo más barato, así que habría que sumar el mezclador aparte |
| [Leroy Merlin, Imex Line gris champagne/mate](https://www.leroymerlin.es/search?q=grifo%20ducha%20gris%20antracita) | 172-281 € | Toda la gama gris de Leroy (Imex Line) empieza en 172 € con descuento. Nada gris por debajo de 100 € en catálogo propio |
| [Bricodepot, Imex Line gris champagne](https://www.bricodepot.es/catalogsearch/result/?q=grifo+ducha+gris+antracita) | 172-288 € | Misma gama Imex Line que en Leroy, mismos precios. El único resultado barato con "gris" en el título, [Sined grifo de ducha "gris oscuro"](https://www.bricodepot.es/grifo-de-ducha-gris-sined-grifo-gris-oscuro-8025431017788) a 71,05 €, es en realidad un **grifo de pie para lavapiés de jardín/piscina** (categoría Jardín y exterior), no un grifo de ducha |
| [Bricodepot, filtro de color "Gris" en Grifos de ducha](https://www.bricodepot.es/banos/columnas-ducha-grifos/grifos-de-ducha?vdesc_colour_group=21338) | — | El filtro de color gris de la categoría solo devuelve piezas sueltas en acero inox 316L satinado de la marca SINED (rociadores, mangos, kits sin grifo) pensadas para duchas de exterior/jardín, con precios de 267 € hacia arriba ([ejemplo](https://www.bricodepot.es/kit-de-ducha-con-soporte-mural-tubo-flexible-y-acabado-en-inox-satinado-8025431155923)). El único grifo mezclador de esa lista, [K2O Chillout Relax](https://www.bricodepot.es/grifo-mezclador-de-ducha-k2o-chillout-relax-26-2x6-3x7-1-cm-dise-no-cuadrado-8445401009078), 32,37 € en oferta, lo vende un tercero con solo 72,6 % de valoraciones positivas y es solo el mezclador — falta comprar la teleducha aparte |
| [Obramat, "grifo ducha gris"](https://www.obramat.es/search?q=grifo%20ducha%20gris) | 62-112 € | La búsqueda ignora el color y devuelve el catálogo genérico (Roca Nora, Ramon Soler Tarraco, Tres Tenerife, Drake…), todos en cromo. Ninguna ficha ofrece variante gris/antracita |

## 3. Análisis

### Opción A: conjunto completo, gris, por 90-100 € como máximo

| # | Producto | Tienda | Precio | Material | Reseñas | Riesgo |
|---|---|---|---|---|---|---|
| A1 | [INEX Europa Stelo D191STL, 4 en 1](https://www.amazon.es/dp/B0DVY8SBMC), grifo bañera + ducha de mano + cabezal cuadrado + chorro | Amazon | **69,95 €** | Acero inoxidable + latón según ficha; **Color: Titanio Grey, Acabado: Negro titanio** — es el único candidato con la ficha internamente consistente | 4,4★ (42) | Garantía de 10 años según el anuncio, pero **una reseña dice que en realidad el fabricante solo da 3 años** ("Mensonge dans l'annonce... la garantie n'est que de 3 ans"). Otra reseña: llegó con grietas y perdía agua, piezas de plástico de baja calidad. El mismo fabricante (INEX Europa) tiene otro modelo (INX 1551 BS, descartado arriba) con fallos de rotura a los 6 meses — riesgo de marca, no solo de unidad |

No hay más candidatos de opción A dentro de presupuesto: todo lo demás gris de verdad (Görbach, Imex Line, SINED) empieza en 140 €, y lo que se anuncia "gris" por debajo de 90 € tiene fichas contradictorias o cero reseñas (ver sección 2).

**Alternativa a considerar, fuera del gris estricto**: si se acepta ampliar de "gris antracita/titanio" a "acero inoxidable satinado" (gris plata natural del metal, no una capa pintada), hay columnas con grifo en **acero inoxidable cromado/satinado** con mejor historial de reseñas y del mismo rango de precio — por ejemplo [Görbach Columna con grifo termostático, Cromo, 69,09 €](https://www.amazon.es/dp/B084GRLX1X) (4,5★, sin grifo — solo columna) o los [JOHO en Cromo, 41,95-60,99 €](https://www.amazon.es/dp/B07PB9GDKZ) (4,3★). Esto no es lo que se pidió literalmente (cromo brillante, no gris mate), pero conviene decirlo: el acero inoxidable natural es más resistente a la corrosión que cualquier gris pintado por debajo de 140 €, precisamente porque no lleva ninguna capa que se pueda caer.

### Opción B: grifo + alcachofa de colgar, gris, por 80 € como máximo

**No se ha encontrado ningún candidato que cumpla los tres requisitos a la vez: gris de verdad, valvulería + teleducha completa, y reseñas suficientes para confiar.**

Lo que apareció en las búsquedas y por qué no vale:

- Alcachofas de ducha "gris" sueltas con filtro antical (ej. [B0HJG3MBN4](https://www.amazon.es/dp/B0HJG3MBN4), 20,75 €, 4,9★; [B0HDYW8CCQ](https://www.amazon.es/dp/B0HDYW8CCQ), 21,84 €, 5,0★): son solo el cabezal de mano con manguera, sin grifo ni válvula. No sirven solas.
- [Bostar Grifo de bañera con cascada, gris antracita, aluminio espacial](https://www.amazon.es/dp/B0GRGZR3S3), 53,99 €: sí trae grifo + ducha de mano de 3 modos + soporte + manguera de 1,5 m, y el precio encaja de sobra. Pero solo tiene 5 reseñas en Amazon, todas 5★ sin texto — no hay forma de verificar si el "aluminio espacial" aguanta la limpieza diaria. Es el candidato menos malo si se acepta el riesgo de la falta de datos.
- El resto de búsquedas con "gris" en la consulta devolvieron sistemáticamente productos cromados, plateados o negros con "gris" solo en alguna palabra suelta del título (ruido de posicionamiento SEO en Amazon), no en el color real del producto.

## 4. Recomendaciones

> **Superadas por la sección 0.1 (2026-09-26).** Las recomendaciones 1-4 de aquí abajo son la primera pasada, antes de corregir que el cromado cuenta como gris. La recomendación vigente es: New Boreal Termostático (97 €) para el conjunto, Ramon Soler Tarraco (41 €) para grifo+alcachofa de colgar.

1. **Opción A, si se quiere gris de verdad dentro de presupuesto: INEX Europa Stelo (A1), 69,95 €.** Es el único con ficha coherente, pero hay que asumir el riesgo de plástico interno de baja calidad y de que la garantía real pueda ser de 3 años y no de 10 como anuncia el titular del producto. Comprarlo sabiendo que puede tocar sustituirlo en 3-6 años, que es justo el marco de tiempo que el usuario ya da por bueno.
2. **Opción A, alternativa sin ceñirse al gris mate: columna en cromo/acero inoxidable de Görbach o JOHO (40-70 €).** Más reseñas, más historial, y el material en sí no se pela porque no lleva pintura. Vale solo si el usuario acepta que no es el tono antracita/titanio que pidió, sino un gris plata metálico natural.
3. **Opción B: ninguna recomendación firme.** El Bostar de bañera (53,99 €) es la única combinación gris con grifo y teleducha por debajo de 80 €, pero con 5 reseñas no hay manera de avalarlo. Antes de comprarlo a ciegas, dos caminos: (a) comprar el K2O Chillout Relax de Bricodepot (32,37 €, mezclador suelto) y sumarle una alcachofa antical gris de las de arriba (20-22 €) para montar un kit propio por 52-55 € total, asumiendo el riesgo del vendedor con 72,6 % de valoraciones; o (b) igual que en A2, aceptar cromo/inox en vez de gris mate y elegir cualquiera de los grifos con teleducha de 35-50 € con miles de reseñas que ya aparecían en la investigación de negro anterior (ver historial de esta nota en git, sección "Opción B" de la versión de 2026-09-24).
4. **Ninguna de las dos opciones tiene en 2026 un ganador tan claro como el que hubo para negro (Clever Rocket / Roca Rodas).** El mercado de gris barato y fiable, sencillamente, no existe todavía por debajo de 90-100 €: aparece a partir de 140 € (Görbach Gunmetal) o 172 € (Imex Line). Conviene revisar esta nota dentro de unos meses por si entra stock nuevo, en vez de asumir que la conclusión es definitiva.

## 5. Dónde se ha buscado

Todo con Chrome real por CDP (anfitrión 192.168.1.5) el 2026-09-26.

- Amazon.es: [conjunto ducha gris monomando rociador alcachofa](https://www.amazon.es/s?k=conjunto+ducha+gris+monomando+rociador+alcachofa&s=review-rank) (solo devolvió sistemas empotrados de 200-1000 €), [columna ducha gris monomando, 30-110 €](https://www.amazon.es/s?k=columna+ducha+gris+monomando&rh=p_36%3A3000-11000&s=price-asc-rank), [columna ducha gris monomando, 70-110 €](https://www.amazon.es/s?k=columna+ducha+gris+monomando&rh=p_36%3A7000-11000&s=price-asc-rank), [grifo ducha gris antracita teleducha soporte](https://www.amazon.es/s?k=grifo+ducha+gris+antracita+teleducha+soporte&s=review-rank) (casi todo eran soportes sueltos y columnas negras/cromadas, no aportó candidatos gris), [grifo monomando ducha gris alcachofa mano soporte, 10-80 €](https://www.amazon.es/s?k=grifo+monomando+ducha+gris+alcachofa+mano+soporte&s=review-rank&rh=p_36%3A1000-8000) (solo alcachofas sueltas), [grifo ducha gris monomando teleducha, 20-80 €](https://www.amazon.es/s?k=grifo+ducha+gris+monomando+teleducha&rh=p_36%3A2000-8000&s=price-asc-rank) (ningún resultado gris real, todo cromo/plateado/negro pese a la consulta), [görbach ducha gunmetal cepillado](https://www.amazon.es/s?k=g%C3%B6rbach+ducha+gunmetal+cepillado&s=price-asc-rank) (confirma que el gunmetal auténtico de esta marca empieza en 140 €).
- Leroy Merlin: [grifo ducha gris antracita](https://www.leroymerlin.es/search?q=grifo%20ducha%20gris%20antracita) (toda la gama gris propia, Imex Line, empieza en 172 €), [sensea ducha gris](https://www.leroymerlin.es/search?q=sensea%20ducha%20gris) (sin resultados relevantes en gris).
- Brico Depot: [grifo ducha gris antracita](https://www.bricodepot.es/catalogsearch/result/?q=grifo+ducha+gris+antracita) (mismos precios de Imex Line que en Leroy, 172-288 €; el único resultado barato con "gris" resultó ser un grifo de jardín), [categoría Grifos de ducha filtrada por color Gris](https://www.bricodepot.es/banos/columnas-ducha-grifos/grifos-de-ducha?vdesc_colour_group=21338) (solo piezas SINED en inox 316L de gama alta/exterior, 267 € en adelante, salvo un mezclador suelto de tercero a 32,37 €).
- Obramat (antiguo Bricomart): [ducha gris antracita](https://www.obramat.es/search?q=ducha%20gris%20antracita) (solo azulejos y cortinas, cero grifería), [grifo ducha gris](https://www.obramat.es/search?q=grifo%20ducha%20gris) (la búsqueda ignora el filtro de color y devuelve el catálogo genérico en cromo: Teka, Mercury Pro2, Roca Nora, Ramon Soler Tarraco, Tres Tenerife, Drake).
- **Ronda de corrección (0.1), mismo día por la tarde**, todo con Chrome real por CDP (192.168.1.5): fichas técnicas completas (accordion "Características" desplegado y leído del DOM, campo por campo) de [Roca Nora](https://www.obramat.es/productos/grifo-de-ducha-monomando-roca-nora-10474842.html), [Columna Mercury Pro2](https://www.obramat.es/productos/columna-de-ducha-monomando-mercury-pro2-25069465.html), [Columna Teka Dual Control](https://www.obramat.es/productos/columna-de-ducha-teka-dual-control-10884846.html), [Columna New Boreal Termostático](https://www.obramat.es/productos/columna-de-ducha-termostatico-new-boreal-10952046.html), [búsqueda "columna ducha dual"](https://www.obramat.es/search?q=columna%20ducha%20dual) (confirma que la línea genérica "Dual" de Obramat solo existe en negro, no en cromo/gris), [búsqueda "kit ducha soporte"](https://www.obramat.es/search?q=kit%20ducha%20soporte), [Kit Roca Stella 100mm](https://www.obramat.es/productos/kit-de-ducha-con-soporte-roca-stella-100mm-3-funciones-25080251.html), [Ramon Soler Tarraco](https://www.obramat.es/productos/grifo-ducha-monomando-ramon-soler-tarraco-25058962.html).

## 6. Otros

- El patrón se repite en las tres tiendas: cuando el nombre del producto dice "gris" pero es barato, casi siempre hay una contradicción entre el título, los bullets de marketing y la ficha técnica (cromo vs. gris, aluminio vs. plástico). Conviene desconfiar de cualquier ficha de Amazon que mencione el color solo en el título y no lo repita, igual, en la tabla de características.
- El acero inoxidable 316L (grado marino, el que no se pica ni con agua salada) sí existe para duchas en España, pero el catálogo que se ha encontrado (SINED, vendido por Bricodepot) está pensado para duchas de jardín o piscina, no de baño, y empieza en 267 €. No es una opción realista para este presupuesto.
- No se ha probado con vendedores fuera de las cuatro tiendas indicadas (Ferrocano, obramat marketplace de terceros, etc.) porque el usuario pidió evitar tiendas con gastos de envío caros y centrarse en Amazon, Leroy y Brico/Obramat.

## Enlaces

- [[apartamentos-calle-uruguay]]
- [[_index]]
