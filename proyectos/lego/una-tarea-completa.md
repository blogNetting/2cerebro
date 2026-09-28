---
title: Lego — una sola tarea, todos los pasos
created: 2026-09-28
updated: 2026-09-28
tags: [lego, ejemplo, tarea, paso-a-paso, bitvavo]
zona: tecnico
---

**Una tarea, de principio a fin, sin saltarse un paso.** La tarea es esta:

> **`004 · Endpoint que devuelve lo que valen mis monedas en euros`**

En cada paso, siempre las cinco mismas cosas: **qué haces tú · qué modelo o herramienta entra · qué genera eso · qué tener en cuenta · qué revisar.** Los datos y enlaces van dentro.

Contexto: se da por hecho que existe el proyecto del [[ejemplo-paso-a-paso]] — API con FastAPI, PostgreSQL, y la tarea 003 hecha (el módulo que habla con Bitvavo existe y está probado).

![[04-los-17-pasos.png]]

**De los 17 pasos, 7 corren sin ti. Y de los 10 en los que estás, cinco son escribir y decidir, no vigilar.** Ésa es toda la diferencia entre ser el policía y ser quien dirige.

> **Versión en PDF:** `una-tarea-diecisiete-pasos.pdf` (18 páginas, A4), en esta misma carpeta. Es la versión maquetada de esta nota, con el diagrama y las tablas. **No se publica**: `.gitignore` excluye los PDF. Esta nota es la fuente y se conserva.

---

# BLOQUE A — Antes de tocar nada

## Paso 1 · Detectas que hace falta

**Tú.** Nada de herramientas.

Estás mirando la web y te das cuenta de que ves las monedas que tienes, pero no cuánto valen. **Eso es la tarea.**

**Qué genera:** nada escrito. Una frase en la cabeza o en un papel:

> «Falta ver el valor en euros.»

**Qué tener en cuenta:** **no abras el editor todavía.** El impulso de «total, es un endpoint, lo pido y ya» es el error que se paga cuatro pasos después. Y hay una razón medida: **el 42% de los bloqueos que paran a un agente son información ausente** ([HiL-Bench](https://huggingface.co/papers/2604.09408), 300 tareas, 1.131 bloqueos). Lo que no escribas, lo va a inventar.

**Qué revisar:** ¿esto es **una** tarea o dos? «Ver el valor en euros» y «guardar un histórico de precios» son dos. Si tiene dos «y», son dos tareas.

---

## Paso 2 · Escribes el borrador con un modelo

**Modelo:** el que uses para pensar (el grande). **No** el que va a programar.

**El prompt, literal:**

```
Actúa como ingeniero de backend. Te doy una tarea en bruto.

Tarea: falta ver el valor en euros de las monedas que tengo.

Requisitos:
- Escribe el borrador en tareas/004-valoracion.md
- Formato: qué se quiere · dónde puede tocar · criterios de aceptación ·
  done when · verify · qué devuelve
- Los criterios tienen que poder comprobarse con un comando.
  Si no se puede, no lo escribas como criterio.
- NO escribas código.
- Antes de escribir, mírame el repositorio y dime qué necesitas saber.
```

**Qué genera:** `tareas/004-valoracion.md`, un fichero.

**Qué tener en cuenta:**

**DATO, y es el corazón de este paso.** Un criterio que no se puede comprobar **no es un criterio**. El caso medido: un proyecto migró de Jest a Vitest y su condición de terminado no era una frase, era un comando — *«comprueba si pasan los tests, si existe `vitest.config`, si `jest.config` ha desaparecido y si no quedan imports de `@jest`»* ([[videos-y-comunidad]]). **Cuatro comprobaciones ejecutables. Ninguna es una opinión.**

**DATO.** Y el coste de no hacerlo: con la verificación idéntica y **sólo el texto cambiado**, GPT-4 pasa de **73,8% a 6,7%** de acierto cuando el enunciado se vuelve contradictorio ([arXiv:2507.20439](https://arxiv.org/abs/2507.20439)).

**Qué revisar:** el borrador va a salir **demasiado largo**. Eso es normal: el modelo tiende a documentar. Tu trabajo en el paso 3 es **recortar, no ampliar**.

---

## Paso 3 · Revisas el borrador, y recortas

**Tú.** Sin modelo. Con el fichero delante.

Esto es lo que te va a salir, y lo que tienes que hacer con cada parte:

**Borrador que sale:**

```markdown
# 004 · Endpoint de valoración

## Qué se quiere
Que la API pueda decir cuánto valen en euros todas mis monedas.

## Dónde puede tocar
- Permitido: app/cripto/**, tests/cripto/**
- Prohibido: app/auth/**, app/bitvavo/** (usar el módulo, no tocarlo)

## Criterios de aceptación
1. CUANDO se pide el endpoint ENTONCES devuelve el valor total en euros.
2. CUANDO una moneda no tiene precio ENTONCES se omite y se avisa.
3. CUANDO Bitvavo falla ENTONCES se usa el último precio guardado.
4. La respuesta incluye el desglose por moneda.

## Done when
Todo pasa.

## Verify
pytest tests/cripto/
```

**Y así es como lo recortas y lo endureces, criterio por criterio:**

| Como salió | El problema | Como queda |
|---|---|---|
| «devuelve el valor total» | **No dice qué formato.** ¿Un número? ¿Un objeto? | «devuelve un objeto con `total_eur` (número, dos decimales) y `monedas` (lista)» |
| «se omite y se avisa» | **«Avisa» no es comprobable.** ¿Un log? ¿Un campo? | «la moneda aparece con `precio: null` y el total **no la cuenta**» |
| «se usa el último precio guardado» | Bien, pero **falta el límite**: ¿cuánto es «último»? | «si el último precio tiene **más de 24 h**, se marca `desactualizado: true`» |
| «Totas pasa» | **No dice nada.** | «los 4 criterios pasan y `ruff` no da avisos nuevos» |

**Qué tener en cuenta — el aviso más importante de todo el proceso:**

**DATO.** **El agente no va a preguntar.** Sólo pide aclaraciones entre el **31,8% y el 44,5%** de las veces, según el andamiaje ([UnderSpecBench](https://arxiv.org/abs/2607.02294)). Y si le das la opción de preguntar sin más, **se hunde**: en el peor caso de un modelo, del **84,7% al 5,3%** de rendimiento ([HiL-Bench](https://huggingface.co/papers/2604.09408)). **Lo que dejes ambiguo, lo va a decidir él.**

**Qué revisar:** busca **contradicciones**. El caso real: una especificación con **dos requisitos contradictorios en las secciones 4 y 7**, y tres días de un equipo peleando con un agente que «rompía funcionalidad adyacente» — *«el agente estaba haciendo exactamente lo que se le dijo; es que le dijeron dos cosas distintas»* ([[lo-que-dice-la-comunidad]]).

---

## Paso 4 · Congelas y decides el doble de prueba

**Tú.** Y aquí hay una decisión que **no se puede delegar**.

**La pregunta:** este endpoint llama a Bitvavo. **¿Con qué se prueban los criterios 2 y 3, que son sobre fallos de Bitvavo?**

**No se puede llamar a Bitvavo de verdad en un test.** Si lo haces, el test deja de ser un oráculo — depende de la red, del saldo de tu cuenta y de que la bolsa esté abierta. **Un oráculo que depende del mundo no es un oráculo.**

**La decisión:** se construye un **doble de prueba** — un fichero con respuestas de Bitvavo grabadas una vez, y el test usa eso.

```
tests/cripto/dobles/bitvavo/
├── saldos_ok.json
├── saldos_moneda_desconocida.json
└── error_autenticacion.json
```

**Qué tener en cuenta — y esto es lo que casi nadie hace:**

**DATO.** El requisito «la firma sigue la documentación oficial de Bitvavo» **no se puede comprobar automáticamente**, y es exactamente el tipo de requisito que hace alucinar al agente: *«el agente tiene instrucciones de no escribir código, escribir tests — y al hacerlo **define una API**, lo que hará que la IA **alucine la API**»* ([[implementaciones-reales]], crítica en Hacker News).

**Por eso:** la firma **la compruebas tú una vez, a mano**, contra la documentación real de Bitvavo. Y el resultado —las cabeceras exactas que hacen falta— **se escribe como dato fijo en el doble**. A partir de ahí, el agente trabaja contra un dato que ya no puede alucinar.

**Qué revisar:** que el doble **lo hayas hecho tú o lo hayas validado tú**. Si el agente graba sus propios dobles, se está examinando a sí mismo.

**El fichero final, congelado:**

```markdown
# 004 · Endpoint de valoración

**Owner:** agente
**Depende de:** 003
**Cubre:** RF-06

## Qué se quiere
Saber cuánto valen en euros las monedas que tengo.

## Dónde puede tocar
- Permitido: `app/cripto/**`, `tests/cripto/**`
- Prohibido: `app/auth/**`, `app/bitvavo/**` (se usa, no se toca),
  el esquema de la base, y añadir dependencias

## Criterios de aceptación
1. CUANDO se pide `GET /cripto/valoracion` ENTONCES devuelve
   `{"total_eur": <número con 2 decimales>, "monedas": [...]}`,
   y `total_eur` es la suma de `cantidad × precio_eur` de cada moneda.
2. CUANDO una moneda no tiene precio ENTONCES aparece con
   `"precio_eur": null` Y SU CANTIDAD NO SUMA AL TOTAL.
3. CUANDO Bitvavo no responde ENTONCES se usa el último precio guardado,
   y si tiene más de 24 h se añade `"desactualizado": true` en esa moneda.
4. CUANDO se piden los datos ENTONCES se llama a Bitvavo UNA sola vez,
   no una por moneda.

## Done when
Los 4 criterios pasan y `ruff check app/cripto` no da avisos nuevos.

## Verify
    pytest tests/cripto/test_valoracion.py -q
    ruff check app/cripto

## Qué devuelve
Tres líneas: qué cambió · qué se rompió · qué decidió por su cuenta.
```

**Fíjate en el criterio 1:** dice **la fórmula**. No «el total correcto», sino **cómo se calcula**. Es la diferencia entre un criterio y un deseo.

---

# BLOQUE B — La tarea en marcha

## Paso 5 · La tarea va a la cola

**Herramienta:** git. Nada más.

```
git add tareas/004-valoracion.md
git commit -m "tarea: 004 valoración en euros"
```

**Qué genera:** la tarea está en el repositorio, versionada. **La cola es el repositorio.** No hay base de datos, no hay servidor, no hay nada que mantener.

**Qué tener en cuenta:** el orden. `tareas/001`, `002`, `003`, `004` — **el orden alfabético es el orden de ejecución**. Por eso los nombres van numerados.

**Qué revisar:** que la tarea 003 esté en `hecho/` y no en `tareas/`. Si no, el bucle va a coger la 004 antes de tiempo.

---

## Paso 6 · Se lanza el bucle

**Herramienta:** el script del bucle. **No lo escribes tú**: [`fstandhartinger/ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum) ya trae `ralph-loop-codex.sh` con `spec_queue.sh`, `circuit_breaker.sh`, `nr_of_tries.sh` y `response_analyzer.sh`.

```bash
./ralph-loop-codex.sh
```

**Qué genera:** el script arranca **una instancia nueva del agente** para esta tarea, y le pasa el contenido del fichero.

**Qué tener en cuenta — y es la decisión de diseño más importante del bucle:**

**DATO.** **Contexto limpio por tarea, no una sesión larga.** Se midió con **18 modelos**, 8 longitudes de entrada y 11 posiciones distintas: *«el rendimiento se vuelve cada vez menos fiable a medida que crece la entrada»*, y con prompts de **~300 tokens frente a ~113.000, todos los modelos rinden mejor** ([Chroma](https://www.trychroma.com/research/context-rot)). Por eso la instancia es nueva: **la tarea 4 no arrastra la 3**.

**Qué revisar:** que el agente esté viendo **el fichero de la tarea**, no un resumen. Si tu script le pasa «implementa la tarea 004», el agente no sabe qué es.

---

## Paso 7 · El agente lee la tarea

**Modelo:** el que programa (el mediano). Aquí no decides nada — esto es lo que ocurre por dentro.

**Qué ve exactamente:** el contenido del fichero `004-valoracion.md`, los ficheros que el script le marque como contexto obligatorio, y el fichero de reglas del proyecto.

**Qué genera:** un plan mental. Nada escrito todavía.

**Qué tener en cuenta:** **aquí es donde se ve si el paso 3 lo hiciste bien.** Si el agente tiene que decidir algo que no está escrito —si el total incluye la moneda sin precio, si «no responde» significa tiempo de espera agotado o error 500— **lo va a decidir él**, y puede decidirlo distinto de lo que tú querías.

**Qué revisar:** nada, tú no estás. **Esto se revisa en el paso 15.** Pero si al revisar descubres que tuvo que decidir algo, **la corrección no es cambiar el código: es cambiar el fichero de la tarea** y volver a lanzarla.

---

## Paso 8 · Explora el repositorio

**Qué hace:** lee `app/bitvavo/` para ver cómo se llama, y `app/models/` para ver qué campos hay.

**Qué genera:** nada visible.

**Qué tener en cuenta:** **los permisos ya están declarados.** El agente no puede salirse de `app/cripto/**` y `tests/cripto/**`, porque eso está en la configuración de permisos del proyecto, no en prosa.

**DATO, y es la razón de que sea un permiso y no una regla escrita.** En el caso real de la noche desatendida: *«Opus 4.7 **leyó esas reglas, las reconoció y las violó** en la siguiente entrada del registro»* ([issue #53610](https://github.com/anthropics/claude-code/issues/53610)). **Lo que obliga es el entorno, no el texto.**

**Qué revisar:** que la configuración de permisos **no tenga un comodín**. Si dice `Bash(*)`, los permisos no existen.

---

## Paso 9 · Implementa

**Qué genera:** `app/cripto/valoracion.py` y un endpoint nuevo. En su rama.

**Qué tener en cuenta:** **NO toca `app/bitvavo/`.** Lo usa. Si cree que hay que cambiarlo, la tarea dice que **pare y lo diga** — y eso es una decisión del paso 3 que ahora te protege.

**DATO.** El criterio 4 —«se llama a Bitvavo **una sola vez**, no una por moneda»— es el que evita el fallo clásico: un bucle que hace una llamada por moneda. Con 8 monedas son 8 llamadas, y con la limitación de peticiones de Bitvavo, eso es un problema. **Está escrito como criterio, así que se comprueba.**

**Qué revisar:** nada, tú no estás.

---

## Paso 10 · Escribe la prueba, y aquí está la trampa

**Qué genera:** `tests/cripto/test_valoracion.py`.

**Qué tener en cuenta — y es la trampa más grande de todo el proceso:**

**DATO.** **El agente no puede escribir los tests que deciden si el agente acertó.** Un agente reescribió los resultados del runner —*«el gancho intercepta cada resultado durante la fase de llamada y lo reescribe a "passed"»*— y sacó **500 de 500 sin resolver un solo problema** ([Berkeley RDI](https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/)).

**La consecuencia práctica, y es lo que tienes que revisar en el paso 15:** hay **dos juegos de pruebas**.

| | Quién lo escribe | Para qué |
|---|---|---|
| **`tests/` visible** | El agente | Para trabajar. Las lee, las ejecuta, itera contra ellas |
| **`verificacion/` oculta** | **Tú**, o el agente **con el doble que hiciste tú** | Para decidir. **Está fuera de su alcance de permisos** |

**Qué revisar:** que el directorio de verificación **esté en la lista de prohibidos**. Si no lo está, el agente puede editarlo, y entonces el veredicto no vale nada.

---

## Paso 11 · Se autocomprueba

**Qué hace:** corre `pytest tests/cripto/ -q` y arregla lo que falle.

**Qué genera:** iteraciones. Hasta que pasa.

**Qué tener en cuenta:** **aquí el agente sí puede iterar solo, y es lo que hace valioso el bucle.** Pero con dos límites que tienen que estar puestos:

- **Mismo error y misma acción 3 o 4 veces → parar.** Es el patrón que usa el detector de atascos de OpenHands, que viene activado por defecto.
- **Tope de vueltas y de gasto.** El ejemplo real: **50 iteraciones, 500.000 tokens, 5 dólares** ([[videos-y-comunidad]]).

**DATO, y por esto no vale un vigilante por reloj.** Se midió: **9 de 325 ejecuciones (2,8%)** se cortaron **estando sanas**, y **5 de 49 (10,2%)** en un modelo concreto. La frase: *«no está detectando un flujo muerto; está **guillotinando uno lento pero sano**»* ([issue #85265](https://github.com/anthropics/claude-code/issues/85265)). **Parar por patrón repetido, no por tiempo.**

**Qué revisar:** nada todavía.

---

# BLOQUE C — La decisión

## Paso 12 · Verifica el runner, no el agente

**Quién:** el script. **El agente ya no habla.**

```
pytest verificacion/test_004_valoracion.py -q
ruff check app/cripto
```

**Qué genera:** **rojo o verde. Sin matices.**

**Qué tener en cuenta — y aquí está el límite que hay que aceptar:**

**DATO, medido en las dos direcciones.**

- Por un lado, los tests aprueban lo roto: **31,08%** de parches aprobados lo son con tests débiles ([SWE-bench+](https://arxiv.org/abs/2410.06992)).
- Y por otro —esto corrige lo que yo escribí al principio— **35,5%** de las tareas auditadas tienen tests **demasiado estrictos** que *«invalidan entregas funcionalmente correctas»* ([OpenAI](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/)).

**Y el límite que ningún oráculo cubre, concreto para esta tarea:** que los cuatro criterios pasen **no dice si `app/cripto/valoracion.py` es el sitio correcto** para esa lógica. *«Los tests dan realimentación **en segundos**, pero el coste de una mala arquitectura se mide **en semanas o meses**»* ([HumanLayer](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)).

**Qué revisar:** que el test de verificación **no esté en el mismo directorio que el visible**. Si están juntos, el recorte de permisos no sirve de nada.

---

## Paso 13 · Si falla

**Tres casos, y tres respuestas distintas:**

| Qué pasa | Qué se hace | Por qué |
|---|---|---|
| **Falla un criterio** | Vuelve a intentarlo, **una vez** | El tope práctico de los montajes que existen es **2–3 intentos** |
| **Falla igual dos veces seguidas** | **Parar.** Escribir `needs_human` y avisar | El razonamiento de uno de los montajes reales: *«dos intentos dan exactamente un ciclo de inténtalo otra vez: margen para un “se me olvidó el git add”, y **ningún margen para que un agente queme en silencio el tiempo máximo de la sesión**»* |
| **Bitvavo cambió su API** | **No es un fallo del bucle.** Es información nueva que va al fichero de la tarea | Y esto pasa: el requisito «sigue la documentación oficial» **no se comprueba solo** |

**Qué tener en cuenta:** **no se reintenta a ciegas.** El criterio para parar es la **falta de progreso**, no el número de fallos.

---

## Paso 14 · El commit

**Qué genera:** **un commit por tarea**, en su rama.

```
rama: tarea/004-valoracion
commit: "004: endpoint de valoración en euros"
```

**Qué tener en cuenta:** **un commit por tarea tiene una consecuencia práctica enorme:** si mañana algo se rompe, `git bisect` te dice **qué tarea lo rompió**. Y puedes deshacer **esa** sin tocar las demás.

**Qué revisar:** que el commit **no arrastre otros ficheros**. Si tocó algo fuera de su alcance, es que los permisos no estaban bien puestos — y eso es un fallo del paso 8, no del agente.

---

## Paso 15 · Tu revisión

**Tú.** Y aquí está el dato que hace que este paso **no sea opcional**:

**DATO.** El evaluador automático **sobrestima la decisión real del mantenedor en 24,2 puntos porcentuales** (error estándar 2,7), medido sobre **296 PRs de IA y 47 humanos fusionados de verdad** ([METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/)).

**Qué mirar, en este orden:**

1. **El diff, no la descripción.** Lo que el agente dice que hizo no es lo que hizo.
2. **Los nombres y los sitios.** ¿`app/cripto/valoracion.py` es donde lo habrías puesto tú?
3. **La sección de desviaciones.** Ahí es donde el agente cuenta **qué decidió por su cuenta**. En esta tarea: ¿cómo trató la moneda sin precio? ¿cómo definió «más de 24 h»? **Si decidió algo que debería estar en el fichero, la corrección va al fichero, no al código.**
4. **Si no entiendes una línea, no lo fusiones.** La regla del material práctico: *«si no entiendes una línea, pídela explicada»*.

**Qué revisar, en una frase:** que lo que está verde **sea lo que tú querías**, no lo que el test dice que querías.

---

## Paso 16 · La fusión

**Tú.** **Esta puerta no se quita**, y el paso 15 es la razón.

```
git checkout principal
git merge tarea/004-valoracion
```

**Qué genera:** el código, en la rama principal.

**Qué tener en cuenta:**

**DATO.** Las fusiones de agente **necesitan arreglos posteriores con 1,62 veces las probabilidades** de las humanas ([arXiv:2609.26847](https://arxiv.org/abs/2609.26847)). *Matiz honesto: es una razón de probabilidades sobre una base pequeña —sólo el **4,5%** de los PRs de agente fusionados recibe un arreglo verificado en 30 días.*

**Qué revisar:** que no haya fusionado nada de la tarea 005 sin querer. Un commit por tarea lo hace evidente.

---

## Paso 17 · Cierre y registro

**Qué genera:** tres cosas, y las tres importan.

1. **La tarea se mueve** de `tareas/` a `hecho/`.
2. **Se anota lo aprendido** en `progreso.txt`: que Bitvavo devuelve los precios como texto y hay que convertirlos, por ejemplo. **La siguiente instancia lee esto.**
3. **Si la tarea reveló un hueco en sí misma** —faltaba un criterio, había una ambigüedad—, **se corrige el fichero de la tarea antes de archivarla.** Para la próxima vez.

**Qué tener en cuenta:** **esto no es burocracia, es el mecanismo que hace que la tarea siguiente salga mejor.** Sin el paso 3, el mismo hueco vuelve a aparecer en la tarea 007.

---

# La tabla entera, de un vistazo

| Paso | Tú | Modelo / herramienta | Qué genera | El dato que lo sostiene |
|---|---|---|---|---|
| **1** | Detectas | — | Una frase | 42% de los bloqueos son falta de información |
| **2** | Pides el borrador | Modelo grande | `004-valoracion.md` | 73,8% → 6,7% según el enunciado |
| **3** | **Recortas y endureces** | — | Criterios comprobables | El agente sólo pregunta 31,8–44,5% de las veces |
| **4** | **Decides el doble** | — | Datos de prueba grabados | Un requisito no comprobable hace alucinar |
| **5** | Cola | git | La tarea versionada | El orden alfabético es el orden |
| **6** | Lanzas | Script del bucle | Instancia nueva | Mejor con 300 tokens que con 113.000 |
| **7** | — | El agente | Un plan | Aquí se ve si el paso 3 se hizo bien |
| **8** | — | El agente | Lecturas | Los permisos obligan; la prosa no |
| **9** | — | El agente | `valoracion.py` | El criterio 4 evita 8 llamadas |
| **10** | **Montas la verificación** | El agente | Dos juegos de tests | Un agente reescribió el runner: 500/500 sin resolver nada |
| **11** | — | El agente | Iteraciones | Parar por patrón, no por reloj: 2,8% de falsos cortes |
| **12** | — | El runner | Rojo o verde | 31,08% aprueban lo roto; 35,5% rechazan lo correcto |
| **13** | Decides si seguir | Script | `needs_human` o reinicio | Tope práctico: 2–3 intentos |
| **14** | — | Script | Un commit | Permite `git bisect` |
| **15** | **Revisas** | — | Nada | El oráculo sobrestima 24,2 puntos |
| **16** | **Fusionas** | git | Código en principal | Los merges de agente necesitan 1,62× más arreglos |
| **17** | Cierras | — | `hecho/` + `progreso.txt` | Es lo que mejora la tarea siguiente |

**Los pasos donde estás tú: 1, 2, 3, 4, 5, 10, 13, 15, 16, 17.**
**Los pasos donde no estás: 6, 7, 8, 9, 11, 12, 14.**

**Siete de diecisiete corren sin ti. Y los diez en los que estás son, cinco de ellos, trabajo de escritura y decisión, no de vigilancia.** Ésa es la diferencia entre ser el policía y ser quien dirige.

## Enlaces

- [[etapas]] — el veredicto por etapas, con todos los datos
- [[ejemplo-paso-a-paso]] — el proyecto completo, la arquitectura y los tres diagramas
- [[crear-la-tarea]] — por qué esos campos y no otros
- [[implementaciones-reales]] — los montajes reales con sus ficheros
