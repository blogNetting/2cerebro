---
title: Plan de trabajo de Astillero
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, plan, trabajo]
zona: tecnico
---

**Lista numerada y completa** de todo lo que queda, con quién depende de quién. Lo ya hecho está en [[astillero-bitacora]]. Se actualiza al cerrar cada tarea.

## En curso, PARADA

**1 · Estado verificado (etapa 3).** Empezada y parada a petición. Guardada en la rama `feat/estado-verificado` (**sin PR**, marcada incompleta): solo tiene el esqueleto `marcar_estado`.
- **Falta:** llamarlo en las tres ramas (veredicto → `verificado`; fallo → `sin-verificar`; lectura ciega `75` → `no-verificable`), probar en vivo sobre un proyecto generado con `copier`, redactar y actualizar bitácora.

## Fase 1 — lo siguiente (no depende de nadie)

**2 · Tests propios de Astillero, con cobertura.**
Qué: que el YAML valide, que las plantillas rendericen, que las llamadas finas declaren lo que el workflow llamado acepta, y que la lógica del veredicto acierte en cada caso (0 / 75 / fallo / test borrado / no arrancó).
Listo cuando: corra en CI y **falle** si se reintroduce a propósito cualquiera de los tres fallos de hoy.

**3 · Redactar «cómo funciona Astillero» de punta a punta.**
Qué: un documento que explique el sistema **tal como está construido**, de la idea al código vivo, quién puede escribir qué y los modos de fallo. Hoy hay que juntar cuatro sitios.
Listo cuando: alguien que no ha estado aquí lo lea solo y lo entienda.

## Fase 2 — las etapas que hoy solo son diseño

Una a una: implementar, probar en vivo, redactar, actualizar bitácora. **No se apila.**

**4 · Estado verificado (etapa 3)** — es la tarea 1 cuando se retome. Va primera porque **completa lo que ya está a medias**: el verificador ya emite un veredicto y nada lo escribe en el estado de la tarea.

**5 · Descomposición (etapa 2)** — partir el contrato en tareas **por dependencia de estado**, no por tamaño. Va después porque **toca el despacho, que hoy funciona**.

**6 · Operación (etapa 11)** — monitorización, alertas, copias de seguridad, rotación de secretos.

**7 · Capa de producto** — captura de idea, panel, bugs, versionado: quién dirige el motor.

## Fase 3 — huecos de lo que ya está construido (no depende de nadie)

**8 · Medir la intención.** El hueco que le queda al verificador: hoy comprueba «pasa», no «es lo que se pidió». Un cambio que pasa los tests pero no resuelve lo pedido **pasa**. Es la parte de la etapa 7 que nunca se cerró.

**9 · Probar el raíl de idea sobre un proyecto real.** Se probó como skill suelta; falta sobre un proyecto de verdad.

**10 · Probar el flujo de agente completo** (ejecutor → PR) sobre un proyecto generado. El vigilante sí está probado ahí; el resto no.

**11 · Probar la medición por `schedule`.** Todas las corridas han sido manuales.

**12 · La rama de datos de la medición.** Hoy arrastra una copia del código del proyecto; debe ser una rama huérfana que solo lleve `metricas/`.

**13 · Que la CI del molde no nazca en rojo.** Exige un `CODECOV_TOKEN` que nadie ha creado, así que **el primer push de un proyecto nuevo falla sin que sea culpa del código**. O se hace opcional, o se deja claro y a prueba de error.

**14 · Los cuatro pendientes viejos del hub** — «plataforma git y autonomía», «trunk-based», «imports de gh-aw» y «cuándo arranca el piloto». Son de **antes del research**. Se comprueban uno a uno contra las notas y **lo que no concuerde con hoy se borra.**

**15 · Subdividir `proyectos/astillero/`.** Pasó de 30 notas (37 + bitácora + plan). El reparto está preparado; **no se ejecuta sin tu OK**.

## Parqueados y decisiones tuyas

**16 · Despliegue (etapa 10)** — aparcado a propósito hasta decidir **dónde** se despliega. *(Esto faltaba en la lista anterior.)*

**17 · Exportar Astillero** — qué mover, cómo y con qué estructura. Aparcado por ti.

**18 · El PDF de [[la-fabrica]]** — aparcado por ti.

**19 · Cuándo arranca el primer piloto ([[patrimonial]])** — decisión tuya. *(Viene de la lista vieja; se confirma o se borra en la tarea 14.)*

**20 · El PAT de `release-please`** — solo el día que se active el ruleset con checks obligatorios (necesita GitHub Pro). **Hoy no hace falta nada.**

---

## Lo que faltaba en la lista anterior

Se me habían quedado fuera **seis**: la **1** (lo empezado y parado), la **8** (medir la intención), la **9**, la **10** y la **11** (tres cosas probadas a medias), la **13** (la CI que nace en rojo) y la **16** (el despliegue, que estaba en los documentos pero no en el plan).

## Enlaces

- [[astillero-bitacora]] — lo hecho hoy, con su commit y su evidencia
- [[astillero]] — hub del proyecto
- [[la-fabrica]] — el despiece en doce etapas
- [[decisiones]] — registro con fecha
