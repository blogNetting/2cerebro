---
title: El vigilante — pieza 8 de la fábrica
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, autonomia, escalada, diseno]
zona: tecnico
---

Quién decide que una tarea pasa, se reintenta o se bloquea — y cuándo apareces tú. Cierra el hueco de que un agente desviado siga trabajando solo sin que nadie lo sepa. Diseño: [[la-fabrica]] · [[estado-de-verificacion]].

## Por qué hace falta, medido

- **~30 % de las ejecuciones de agente se desvían.** Medido sobre **más de 10.000 trayectorias reales** de producción. Tres familias: deriva de la especificación, problemas de razonamiento y fallos de herramienta.
- **Y el 90 % de las desviaciones que necesitan una sola intervención se recuperan con esa intervención dirigida.**

La conclusión es la contraria a la intuición: **no hay que dejar correr más, hay que intervenir antes.** Un tercer reintento a ciegas no es persistencia, es gastar saldo.

## Cómo funciona

Un contador por tarea, y cuatro estados de salida:

| Situación | Qué hace |
|---|---|
| Pasa la verificación | Sigue al siguiente paso |
| Falla **una vez** | **Reintenta** con el error concreto delante, no a ciegas |
| Falla **dos veces seguidas** | **Para**, etiqueta `bloqueado` y **te avisa**. No se despacha más |
| No se puede verificar | `no-verificable` — **no se reintenta**, se revisa |

**Lo que mueve la aguja no es el reintento: es la información que le das al reintento.** Está medido que el auto-reparo sin señal nueva rinde poco, y que lo que mejora el resultado es **mejorar la calidad del error** — el test que falla con su traza, el linter, el compilador. *«Aquí está el error exacto»* en vez de *«inténtalo otra vez»*.

## Los umbrales — se copian, no se inventan

**No existe ninguna cifra publicada de cuántos reintentos aguantar en una tarea de agente.** Es un hueco real. Así que se toman de sistemas maduros que sí los publican, y se ajustan con datos propios:

| De dónde | Umbral publicado |
|---|---|
| **gRPC** (reintentos) | máximo **5** intentos en cliente |
| **Bazel** (tests inestables) | **3** intentos |
| **Hystrix** (cortacircuitos) | abre con **≥20 peticiones en 10 s y ≥50 % de error** |
| **ClusterFuzz** (Google) | cierra los no reproducibles a **1 semana**; marca verificado a las **2** |
| **Google SRE** | congela cambios si se agota el presupuesto de error de **4 semanas** |
| **Anthropic** (publicado 2026) | bloquea **0,002 %** de más de mil millones de decisiones; escala a humano **~50 por semana**, revisadas en **menos de una semana** |

**Los dos que se adoptan de salida: 2 reintentos por tarea** (más conservador que gRPC y que Bazel, porque cada intento de agente cuesta mucho más que una llamada de red) **y bloqueo con aviso, nunca reintento infinito.** El valor por defecto de Temporal es intentos **infinitos** — es exactamente el antipatrón que esto evita.

## Dónde NO se aplica

- **No a un fallo no recuperable** (un test que exige algo imposible): reintentarlo es tirar saldo. Se marca `no-verificable` y para.
- **No a un conflicto entre tareas**: eso no es un fallo de la tarea, es un fallo de la descomposición — y ahí lo que se arregla es la tarea, no el reintento.

## Lo que reveló construirlo (2026-09-27)

**1. De qué evento sale la PR tiene que estar contemplado en los dos casos.** El número de PR estaba cableado a `github.event.workflow_run.pull_requests[0].number`. Bajo el disparo que de verdad usa un proyecto (`pull_request`), ese campo viene **vacío**: la tarea no se resolvía, `closingIssuesReferences` no se consultaba nunca y **el intento no se contaba**. El veredicto se calculaba bien y la puerta se quedaba de adorno. Nada de eso era visible leyendo el código.

**2. El aviso lleva el diagnóstico dentro, y el `75` lleva su motivo.** Antes, el aviso de fallo decía «intento 1 de 2», el código de salida y un enlace — **no decía qué falló**, que es justo lo que esta nota prohíbe. Y el caso `75` **no dejaba nada escrito en la tarea**: solo un aviso en el registro del workflow. Con `fail-closed` eso significa una PR bloqueada sin explicación, que es una tarea para ti sin que lo parezca. Ahora los dos comentan: el fallo trae el test que cae con su mensaje, la evidencia, qué pasa ahora y cómo reproducirlo; el `75` trae **por qué** no se pudo comprobar, no una lista de sospechas.

**3. La guarda del reconciliador está comprobada, no solo escrita.** Promueve toda issue `estado:bloqueado` sin bloqueadores abiertos, así que desharía la parada en la misma pasada. La guarda (`vigilante:agotado` ⇒ no se promueve) **vive en el `reconciliar.yml` de Astillero, no en la plantilla del proyecto** — la plantilla solo delega. Comprobado en vivo: tras pasar el reconciliador, la tarea sigue bloqueada.

**4. Y un falso positivo que vale la pena recordar.** La primera prueba se hizo contra un banco hecho a mano cuyo `reconciliar.yml` era una **copia vieja** sin la guarda, así que parecía que la guarda no llegaba a los proyectos. No era cierto: era el banco el que estaba desfasado. **El banco de pruebas hecho a mano miente por desfase; la prueba válida es un proyecto generado con `copier`.**

## Lo que hay que asumir

- **Los umbrales son prestados.** No hay medición propia todavía; se ajustan cuando la pieza de medición lleve un tiempo corriendo. [[medicion-de-la-fabrica]]
- **El aviso es a ti**, y con dos operadores —bueno, con uno— eso significa que **la calidad del aviso es lo que decide si esto sirve o no**. Un aviso que no dice qué falló y qué se intentó ya es una tarea para ti, no un aviso.

## Enlaces

- [[la-fabrica]] · [[estado-de-verificacion]] · [[medicion-de-la-fabrica]]
- [[gas-city-frente-a-la-fabrica]] — `gate` y `max_attempts`/`on_exhausted` como primitivas; ojo: el vocabulario de `gate` está declarado sin consumidor en ejecución
