---
title: Contradicción — si el usuario tiene sudo en esta máquina
created: 2026-10-17
updated: 2026-10-17
tags: [contradiccion, entorno, sudo, privilegios, gas-city]
zona: tecnico
---

El wiki afirma a la vez que en esta máquina no hay `sudo` y que el usuario está en el grupo `sudo`; de la respuesta dependen la instalación de Podman y las rutas de aislamiento del ejecutor. No la resuelve el lint: es un dato de la máquina que conoce el usuario o que se comprueba ejecutando `sudo -n true` y `id`.

## Qué dice cada nota

| Nota | Afirmación (textual) |
|---|---|
| [[decisiones]] (entrada 2026-09-24, «Herramientas de investigación técnica») | «No hay sudo; se descartó `apt`.» (por eso `gh` y `yt-dlp` van a `~/.local/bin`) |
| [[orquestacion-opus-deepseek-informe]] (Aislamiento) | «Instalar Podman requiere sudo; tu usuario está en el grupo `sudo`.» |
| [[orquestacion-seguridad-ejecutor]] (Contexto) | El escenario es «una sola VM Linux sin GPU y sin contraseña de root» (no dice si hay `sudo`) |
| [[gas-city-instalacion-y-modelos]] (§3.2 y siguientes) | Los comandos de instalación usan `sudo install -m 755 gc /usr/local/bin/gc` y `sudo apt install tmux jq git util-linux` |

## Dónde chocan

1. **Sin sudo frente a grupo sudo.** Si no hay `sudo`, ni la instalación de Podman del informe ni los comandos con `sudo` de la nota de instalación son ejecutables tal cual. Si lo hay, la razón con la que se descartó `apt` el 2026-09-24 ya no vale.
2. **Docker.** [[sistemas-de-research-con-agentes]] y [[decisiones]] dan la máquina como «sin Docker»; eso es compatible con ambas lecturas, pero cambia el coste de instalarlo si hay `sudo`.

No es seguro que haya contradicción real: «sin contraseña de root» y «usuario en el grupo sudo» pueden convivir. Lo que sí es incompatible es «No hay sudo» con las otras dos afirmaciones.

## Qué decide el usuario

- Confirmar si `sudo` está disponible (y si pide contraseña).
- Según eso, corregir la entrada del 2026-09-24 de [[decisiones]] o la frase del informe, y revisar los comandos de [[gas-city-instalacion-y-modelos]].

## Enlaces

- [[decisiones]]
- [[orquestacion-opus-deepseek-informe]]
- [[orquestacion-seguridad-ejecutor]]
- [[gas-city-instalacion-y-modelos]]
- [[sistemas-de-research-con-agentes]]
- [[_index]]
