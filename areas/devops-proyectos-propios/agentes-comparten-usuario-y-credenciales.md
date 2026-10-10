---
title: Los agentes de Gas City comparten usuario y credenciales (sin solución gratis probada)
created: 2026-10-10
updated: 2026-10-10
tags: [gas-city, seguridad, credenciales, identidades, gitlab]
zona: tecnico
---

Problema conocido y aceptado: en una sola máquina, cualquier agente de Gas City puede leer las credenciales de otro, así que las identidades por rol (alcalde, integrador, obrero) sirven para ver quién hizo qué, no para impedir la suplantación.

## Qué pasa (comprobado el 2026-10-10 en esta máquina)

- Todos los procesos de agentes corren con el mismo usuario de Linux. Leer `/proc/<pid>/environ` de otro agente funciona (se hizo para ver su `GC_AGENT`).
- El alcalde arranca con `--dangerously-skip-permissions`: ejecuta cualquier comando de shell como ese usuario. No se comprobó el de los obreros.
- Gas City pasa las variables a cada sesión con `tmux -e VARIABLE=valor`; esas opciones quedan en la línea de comandos del proceso y se ven con `ps`, incluida la clave de DeepSeek. Esa clave quedó expuesta en una conversación el 2026-10-10: hay que rotarla.
- La documentación de Gas City lo confirma sin matices: *"Gas City intentionally runs operator-configured commands. Those commands are a feature, not a sandbox"* ([trust boundaries](https://docs.gascity.com/reference/trust-boundaries.md)). No describe aislamiento entre agentes.

## Decisión (2026-10-10)

Sin solución gratis conocida y probada. Se acepta: la barrera que de verdad protege es `main` (push «No one», merge solo Maintainers, CI obligatorio), que un token de rol Developer robado no puede cruzar. Ver [[forges-gratis-con-puerta-en-main]].

## Lo que lo arreglaría, sin probar

Un usuario de Linux o un contenedor por rol, lanzando cada sesión con el *exec session provider*, un script propio que Gas City ejecuta por cada operación de sesión ([doc](https://docs.gascity.com/reference/exec-session-provider.md)). La doc no dice si se puede elegir proveedor por agente o solo por ciudad. Sin probar.

## Pendiente

- Rotar la clave de DeepSeek.
- Probar el exec provider con un usuario o contenedor por rol.
- Evitar que las claves viajen en `tmux -e` (visibles en `ps`).

## Enlaces

- [[_index]]
- [[gas-city-operacion-real]] — dónde viven los secretos hoy (`~/.gc/secrets.env`)
- [[gas-city-instalacion-y-modelos]] — el entorno heredado del servidor tmux
- [[gas-city-y-gitlab]] — GitLab como forge para los rigs
