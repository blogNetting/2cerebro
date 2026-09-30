---
title: Gas City — que el scroll en tmux no dependa de entrar en modo copia
created: 2026-09-30
updated: 2026-09-30
tags: [gas-city, tmux, terminal, vscode, runbook]
zona: tecnico
---

Por qué la rueda del ratón no hacía scroll en la sesión de tmux del alcalde, y la tecla que lo resuelve de verdad sin depender del ratón.

**Corregido el mismo día (2026-09-30): el primer intento (`mouse on` + `history-limit`) no bastaba.** Quedó probado en el servidor de tmux (banderas correctas, ver más abajo) pero en el uso real, desde el terminal integrado de VSCode, la rueda seguía sin hacer scroll — movía el historial de mensajes escritos, como si se pulsara flecha arriba/abajo. La causa real no estaba en tmux: es un fallo conocido y sin arreglo de VSCode, ver «Causa real» más abajo. La solución que funciona de verdad es una tecla dedicada que no pasa por el ratón en ningún momento (`Shift+Re Pág`), no la configuración del ratón.

## El problema

Sin tocar nada, la rueda del ratón dentro de la sesión de tmux de Gas City no movía la pantalla: había que entrar a mano en modo copia (`Ctrl+b` `[`) y salir con `q` cada vez. Es el comportamiento **por defecto** de `tmux`, no un fallo de la instalación.

Fuente primaria, en la propia máquina (`man tmux`, sección `MOUSE SUPPORT`):

> "If the mouse option is on (**the default is off**), tmux allows mouse events to be bound as keys."

Con `mouse off`, la rueda no está ligada a ningún comportamiento de scroll: el evento se pasa tal cual al proceso que corre dentro del panel (aquí, la CLI de Claude), que no sabe qué hacer con él — de ahí que no pasara nada visible sin el modo copia manual.

Segundo motivo, independiente del primero: `tmux` guarda por defecto solo **2.000 líneas** de historial por panel (`history-limit`). Con una sesión tan larga como la del alcalde, aunque el scroll funcionara, lo antiguo se pierde igual.

## Alternativas consideradas

Búsqueda en fuentes de comunidad, no solo el manual (2026-09-30):

| Opción | Qué hace | Por qué se descarta o se acepta |
|---|---|---|
| **`set -g mouse on`** | Activa el modo ratón: la rueda hace scroll normal del historial; al tocar una tecla o hacer clic, vuelve solo al final | **Aplicada.** Resuelve exactamente la queja (nada de `Ctrl+b [` a mano) con un solo ajuste, sin dependencias externas |
| Subir `history-limit` | Guarda más líneas de historial por panel (por defecto 2.000) | **Aplicada junto a la anterior** — sin esto, aunque el scroll funcione, lo antiguo de una sesión larga se pierde igual |
| Dejar que el terminal físico (VSCode, iTerm) haga el scroll nativo, sin tocar tmux | Usar el scrollback del propio emulador de terminal en vez del de tmux | **Descartada, no es viable aquí.** `tmux` corre sus paneles en el *alternate screen buffer* del terminal, un modo especial que las apps de pantalla completa activan para controlar toda la pantalla; lo que se sale de esa pantalla no llega al scrollback del terminal físico. Confirmado por dos fuentes de comunidad independientes: [freeCodeCamp / Alexey Samoshkin, "tmux in practice: scrollback buffer"](https://www.freecodecamp.org/news/tmux-in-practice-scrollback-buffer-47d5ffa71c93/) y [Atera, "How to Scroll Up in tmux"](https://www.atera.com/blog/how-to-scroll-up-in-tmux/), ambas explicando el mismo motivo técnico por caminos distintos |
| Navegación por teclado en modo copia (vi-keys, `Ctrl+u`/`Ctrl+d`) | Mantener el modo copia manual pero moverse con teclado en vez de con flechas | **Descartada como solución al problema planteado** — sigue exigiendo entrar y salir de modo copia a mano, que es exactamente la incomodidad que se preguntó cómo evitar. Sigue siendo útil como alternativa cuando se quiere **copiar** texto, no solo verlo |
| Plugin `tmux-mighty-scroll` ([noscript/tmux-mighty-scroll](https://github.com/noscript/tmux-mighty-scroll)) | Detecta qué proceso corre en el panel y decide cómo tratar la rueda: scroll de historial si no hay proceso, flechas/RePág-AvPág si es un paginador (`less`, `git log`), scroll nativo si es `vim` | **Descartada por dos motivos.** (1) Su propia documentación declara la limitación: *"Does not work in panes with open remote connection, since there is no way to relay back to tmux which processes are running in remote shell"* — no es exactamente nuestro caso (el panel corre `claude` en local, no por SSH), pero añade una dependencia de gestor de plugins (TPM) para un problema que `mouse on` ya resuelve sin más piezas. (2) El valor añadido del plugin es distinguir comportamiento por app (`vim` vs `less` vs `fzf`); en los paneles de Gas City solo corre la CLI de Claude, así que esa distinción no aporta nada aquí |

**Convergencia:** dos fuentes de comunidad independientes (freeCodeCamp, Atera) explican el mismo motivo técnico —el *alternate screen buffer*— por el que el scrollback nativo del terminal no sirve dentro de tmux, y coinciden en que `mouse on` + `history-limit` es la combinación estándar. Ninguna de las dos es documentación de fabricante: son piezas de comunidad sobre el mismo problema.

## Qué se aplicó, con la prueba

Dos cambios, uno en caliente sobre el servidor de tmux ya en marcha (sin reiniciar la sesión del alcalde) y otro guardado para que sobreviva a un reinicio del servicio:

```bash
tmux -u -L NeTT-City set -g mouse on
tmux -u -L NeTT-City set -g history-limit 50000
```

Salida real, pegada en el momento:

```
$ tmux -u -L NeTT-City show -g mouse
mouse on
$ tmux -u -L NeTT-City show -g history-limit
history-limit 50000
```

Y guardado en `~/.tmux.conf` (fuera del repo del wiki, fichero de configuración de usuario de la VM, no de ningún proyecto) para que un `systemctl --user restart gascity-supervisor.service` — el reinicio correcto documentado en [[gas-city-acceso-externo]] — levante el próximo servidor de tmux ya con esto puesto:

```
set -g mouse on
set -g history-limit 50000
```

## Efecto práctico

- La rueda del ratón hace scroll del historial de la sesión directamente, sin `Ctrl+b [`.
- Al tocar una tecla o hacer clic para escribir, tmux vuelve solo al final de la conversación en curso.
- `Ctrl+b [` + `q` sigue existiendo y sigue sirviendo para **copiar** texto con selección de teclado — pero ya no hace falta para simplemente leer hacia atrás.

## Enlaces

- [[_index]] — índice de esta carpeta
- [[gas-city-alcalde]] — el flujo de trabajar con el alcalde en esta misma sesión de tmux, donde se nota la incomodidad
- [[gas-city-acceso-externo]] — el comando correcto de reinicio del servicio (`systemctl --user restart`), el mismo que hay que usar si se quiere que un servidor de tmux nuevo recoja `~/.tmux.conf`
