---
title: El estado de verificación — pieza 3+8 de la fábrica
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, estado, verificacion, permisos, diseno]
zona: tecnico
---

Cómo se representa en el sistema que algo está **hecho** frente a que algo está **verificado**, y quién puede escribir cada cosa. Cierra el problema de que hoy «cerrado» y «verificado» son lo mismo. Diseño: [[la-fabrica]] · [[verificador-de-tareas]] · [[vigilante-de-tareas]].

## El problema, medido

Cerrado es una **acción de quien hizo el trabajo**, no una prueba. Y el modo de fallo dominante de los agentes es exactamente ese: decir que han terminado sin haberlo hecho. En la taxonomía de fallos multiagente, «no ser consciente de las condiciones de terminación» es el **12,4 %** de los fallos y «verificación ausente o incompleta» otro **8,2 %**.

## La regla que lo resuelve, y es contraintuitiva

**La máquina puede invalidar una verificación, pero no concederla.**

Viene de un patrón real en producción — el bot de Ansible, que gestionaba la etiqueta de verificación de CI:

> La etiqueta la ponía **un humano**. El bot **solo la quitaba**. Razón textual del diseño: *«el bot nunca necesita poner la etiqueta, porque no tiene forma de saber si un fallo es legítimo»*.

Traducido: **el agente puede quitar `verificado`** (al empujar commits nuevos), **pero solo el verificador puede ponerlo.** Eso invierte el riesgo: el modo de fallo por defecto pasa a ser **«sin verificar»**, nunca «verificado por accidente».

## Quién puede escribir qué — verificado en la documentación de GitHub

**El agente no puede cerrar, etiquetar ni comentar.** Las cuatro operaciones de issues exigen el mismo permiso de escritura, así que se le da un token con **`issues: read`** más escritura en código y pull requests. Con eso **programa y abre PRs, y no puede tocar el estado.**

**Y lo que no se puede separar:** no existe un permiso por ruta, y para apps y tokens **«etiquetar» y «cerrar» van juntos**. Se intentó y no se puede — es una restricción dura, no una falta de habilidad.

**La puerta es un job aparte** con el permiso de escritura. En GitHub Actions los permisos se pueden dar **por job**, y declarar uno hace que todos los demás queden a cero.

**Y `gh-aw` ya trae esto hecho**: sus «salidas seguras» hacen que el agente **pida** acciones y un job separado las ejecute, con topes por operación y campos permitidos. Es la misma pieza, ya disponible.

## Los estados

En el issue o en un campo del panel, **tres estados y no dos**:

| Estado | Qué significa | Quién lo escribe |
|---|---|---|
| `sin-verificar` | Terminó el trabajo, nadie ha comprobado nada | Estado inicial, automático |
| `verificado` | El verificador corrió, con recibo, y pasó | **Solo el verificador** |
| `no-verificable` | Se intentó y no se pudo determinar | El verificador |

Y el cierre del issue va **al final**, como consecuencia — **nunca como acto del agente**.

**Precedente de que esto no es un invento:** en gestión de incidentes llevan décadas separando **«resuelto»** (arreglado, sin confirmar) de **«cerrado»** (confirmado, permanente). En sanidad, el estándar FHIR tiene ocho estados, incluido `unknown` — *«el sistema no sabe qué estado aplica»*.

## Lo que hay que asumir

- **La disciplina es organizativa, no de permisos.** Como no se puede impedir por permiso que quien programa cierre issues, la separación se sostiene con **dos workflows distintos y dos credenciales distintas**.
- **El campo de un panel vive fuera del issue.** Si alguien mira el issue cerrado, no ve el estado de verificación: hay que hacerlo visible en el propio issue.
- **Las dependencias entre issues de GitHub son solo un icono**, no una puerta: la documentación no dice en ningún sitio que un issue bloqueado no pueda cerrarse.

## Enlaces

- [[la-fabrica]] · [[verificador-de-tareas]] · [[recibo-de-verificacion]] · [[vigilante-de-tareas]]
