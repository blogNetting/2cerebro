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

## Lo que hay que asumir

- **Los umbrales son prestados.** No hay medición propia todavía; se ajustan cuando la pieza de medición lleve un tiempo corriendo. [[medicion-de-la-fabrica]]
- **El aviso es a ti**, y con dos operadores —bueno, con uno— eso significa que **la calidad del aviso es lo que decide si esto sirve o no**. Un aviso que no dice qué falló y qué se intentó ya es una tarea para ti, no un aviso.

## Enlaces

- [[la-fabrica]] · [[estado-de-verificacion]] · [[medicion-de-la-fabrica]]
- [[gas-city-frente-a-la-fabrica]] — `gate` y `max_attempts`/`on_exhausted` como primitivas; ojo: el vocabulario de `gate` está declarado sin consumidor en ejecución
