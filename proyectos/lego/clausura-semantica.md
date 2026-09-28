---
title: Lego — clausura semántica: el marco que faltaba
created: 2026-09-28
updated: 2026-09-28
tags: [lego, verificacion, teoria, marco]
zona: tecnico
---

El mejor marco conceptual que ha aparecido en toda la investigación, y **reencuadra el problema entero**. No viene de una herramienta ni de un benchmark: viene de un ensayo que explica *por qué* las conclusiones anteriores son como son.

## La idea

**Clausura semántica** es la capacidad de un sistema de «definir, interpretar y verificar» el significado y la corrección de sus propias salidas **desde dentro**. Cuatro condiciones ([Stephane Derosiaux, feb 2026](https://sderosiaux.substack.com/p/semantic-closure-why-compilers-know)):

1. El significado es interno.
2. La validez es comprobable por el propio sistema.
3. Los errores son explícitos y decidibles.
4. No hace falta interpretación externa.

Un compilador la tiene. Un modelo de lenguaje **no**.

> «Un compilador tiene una gramática **formal**. Un modelo de lenguaje tiene una distribución de probabilidad.»

Y la consecuencia, que es la frase que lo resume:

> «Si el modelo pudiera juzgar la corrección, ¿por qué no produjo la respuesta correcta en primer lugar?»

Es decir: **no existe un canal de verificación distinto de la generación.** Cuando un modelo «se revisa», lo único que hace es generar más texto. Por eso la autocrítica no funciona — que es justo lo que habían medido tres papers distintos en [[verificacion-y-oraculo]], y esto explica el *porqué*.

## Lo que esto mata

- **«Usa temperatura 0 y será determinista».** No sirve: se han medido oscilaciones de hasta **15%** en precisión a temperatura cero, por el agrupamiento en lotes, la no-asociatividad de la coma flotante, el enrutado de los modelos de mezcla y los núcleos de la GPU. Y aunque fuera perfectamente determinista, **daría la misma respuesta equivocada siempre**.
- **«Pídele una salida en formato JSON y ya está validado».** No: «la clausura de formato **no** es clausura semántica». Que el JSON esté bien formado no dice nada de si el contenido es correcto.

## Lo que esto explica

Reencuadra las cuatro conclusiones anteriores sin contradecir ninguna:

| Conclusión previa | Cómo la explica la clausura semántica |
|---|---|
| La calidad de la spec no reduce defectos | La spec es *prosa interpretable*. Mejorarla no crea clausura: sigue necesitando interpretación externa |
| El andamiaje pesa más que el modelo | El andamiaje **es** la infraestructura de cierre. Cambiarlo cambia cuánta clausura hay alrededor del mismo modelo abierto |
| La verificación tiene techo y falla dando por bueno lo roto | Cada técnica de verificación tiene su propio grado de clausura, y casi ninguna llega al final |
| El agente adivina en vez de preguntar | Preguntar requeriría reconocer que falta información — un juicio sobre la propia ignorancia, que es el mismo problema |

## La prescripción, y por qué es la respuesta a tu pregunta

> «La LLM propone, y **otra cosa** verifica.»
>
> «La clausura vive en el comprobador, no en el generador.»
>
> «**Construye la verificación. Después añade la LLM.**»

El patrón arquitectónico que propone: **un sistema semánticamente abierto (la LLM) metido dentro de un sistema semánticamente cerrado (la infraestructura de verificación)**. La clausura pertenece al conjunto, no a ninguna pieza.

Y esto **da la vuelta a la pregunta de la investigación, para mejor**. La pregunta «¿cómo tiene que escribirse la tarea?» asume que el fichero de tarea es lo que importa. El marco dice otra cosa: **lo que importa es qué hay alrededor**, y la tarea es sólo la interfaz entre el sistema abierto y el cerrado. Un fichero de tarea perfecto dentro de un montaje sin verificación no da autonomía; un fichero mediocre dentro de un montaje con verificación fuerte sí.

## El experimento natural: el compilador determinista

Existe la versión extrema de esta idea en producción: **compiladores de especificación**, donde **ningún modelo escribe el código**.

[`archiet-microcodegen`](https://pypi.org/project/archiet-microcodegen/): texto de un PRD → aplicación FastAPI con PostgreSQL → ZIP. **1.377 líneas, sin LLM, sin clave de API, sólo biblioteca estándar de Python.** Sus cuatro etapas: extracción por expresiones regulares → manifiesto → «genoma» (un documento de arquitectura ArchiMate 3.2) → renderizado de plantillas → empaquetado. Su argumento:

> «Las LLM son geniales entendiendo PRDs desordenados. Son innecesarias para el paso de generación: una vez tienes un manifiesto limpio, la emisión de código es determinista.»

Y la frase que muestra por qué es un sistema cerrado:

> «Si un fallo no se reproduce con microcodegen, está en una capa de eficiencia, no en el algoritmo.»

**Tiene clausura semántica plena: por construcción no puede estar mal.** Y también **no puede hacer nada que no esté en la plantilla.**

**El veredicto, y es el hallazgo:** la idea funciona y es limpia, pero **nadie la usa**. Los repositorios tienen **0 y 4 estrellas**. La plataforma comercial detrás (archiet.com) vende 14 pilas y generación con LLM para la parte de extracción.

*Mi lectura, y la marco como juicio y no como dato:* la razón es el intercambio que el propio marco predice. **La clausura se compra con expresividad.** Un compilador determinista sólo cubre lo que alguien plantilló — CRUD sobre entidades, autenticación, migraciones. En cuanto el proyecto necesita algo que no está en la plantilla, se rompe el modelo entero. Sirve para generar andamios, no productos.

## El intercambio, que es lo que hay que entender

**No se puede tener las dos cosas.** Es la tensión central de todo el proyecto:

| | Sistema cerrado (determinista) | Sistema abierto (LLM) |
|---|---|---|
| **Corrección** | Por construcción | Necesita verificación externa |
| **Expresividad** | Sólo lo plantillado | Cualquier cosa |
| **Coste** | Cero por ejecución | Tokens, y crece con el bucle |
| **Ejemplos** | `archiet-microcodegen` (0★) | Todo lo demás |

La conclusión de ingeniería que sale de aquí: **no elegir uno, sino anidarlos.** Meter el sistema abierto dentro del cerrado, tanto como se pueda:

1. **Lo más cerrado posible primero**: tipos, esquema de base de datos, contratos de API, migraciones. Todo lo que se pueda declarar de forma que una máquina diga sí o no.
2. **La LLM dentro de eso**, y sólo para lo que no se puede cerrar.
3. **La verificación como pieza de primera clase**, no como comprobación al final.

Que es exactamente el diseño de [[montaje-documentado]], pero ahora con un motivo teórico en vez de una lista de recomendaciones sueltas.

## Advertencia sobre la fuente

El ensayo es de un blog personal (Substack), no de una publicación revisada. Varias de sus citas son de segunda mano. Lo que sí es sólido y verifiqué por separado: la oscilación a temperatura cero está documentada; la autocrítica que no funciona está medida por tres grupos ([[verificacion-y-oraculo]]); y el compilador determinista existe con su recuento de estrellas. **La parte que es interpretación del autor, y lo digo claro: la taxonomía de cuatro condiciones es suya, no un resultado experimental.** La etiqueto como marco útil, no como hallazgo.

## Enlaces

- [[verificacion-y-oraculo]] — la evidencia que este marco explica
- [[autonomia-medida]] — el peso del andamiaje, que es la clausura medida
- [[montaje-documentado]] — el diseño, ahora con su motivo
- [[investigacion-lego]] — el informe completo
