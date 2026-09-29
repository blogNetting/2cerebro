---
title: Gas City — levantar el panel para acceder desde fuera de la VM, siempre igual
created: 2026-09-29
updated: 2026-09-29
tags: [gas-city, panel, dashboard, red, runbook]
zona: tecnico
---

Receta probada en vivo el 2026-09-29 para que el panel web de Gas City sea accesible desde cualquier dispositivo de la red local, sin depender de VSCode ni de un túnel SSH. **Confirmado por el usuario: acceso funcionando.**

## El enlace fijo

**`http://192.168.1.8:8372`**

Es la IP de esta VM (`2cerebro`, VMware) en la red local. Funciona igual haya o no una sesión de VSCode abierta.

## Por qué no vale `127.0.0.1` aquí

El panel, por defecto, solo escucha en el propio loopback de la VM. Si accedes desde otro equipo de la red (o desde tu máquina cliente conectando por VSCode Remote-SSH), `127.0.0.1` es el loopback de **esa** máquina, no el de la VM — nunca llega. Hacen falta dos cambios, no uno.

## Los dos cambios necesarios, y por qué los dos

Fichero: `~/.gc/supervisor.toml` (no existe hasta que se crea explícito; sin él, Gas City usa el valor por defecto).

```toml
[supervisor]
bind = "0.0.0.0"
allowed_hosts = ["192.168.1.8"]
```

1. **`bind = "0.0.0.0"`** — hace que escuche en todas las interfaces de red, no solo loopback. Verificado con `ss -tlnp`: sin esto, `LISTEN 127.0.0.1:8372`; con esto, `LISTEN *:8372`.
2. **`allowed_hosts = ["192.168.1.8"]`** — sin esto, aunque escuche en todas las interfaces, **rechaza la petición con `HTTP 421`** (*«host_not_allowed: supervisor Host header is not allowed»*). Es una comprobación aparte, sobre la cabecera `Host` de la petición, no sobre en qué IP escucha el socket. Verificado leyendo el código fuente (`internal/api/middleware.go`, función `isAllowedSupervisorHost`): compara el `Host` de la petición (sin el puerto) contra esta lista exacta; solo el loopback pasa gratis.

**Los dos son necesarios a la vez.** Uno sin el otro no funciona: solo `bind` da 421; solo `allowed_hosts` no sirve de nada si sigue en loopback.

## Los comandos, en orden

```bash
cat > ~/.gc/supervisor.toml <<'EOF'
[supervisor]
bind = "0.0.0.0"
allowed_hosts = ["192.168.1.8"]
EOF

systemctl --user restart gascity-supervisor.service
```

> **⚠️ CORREGIDO EL 2026-09-29 — el comando de arriba era el malo.** Esta receta decía `gc supervisor stop` + `gc supervisor start`. **Comprobado en vivo que eso deja el proceso huérfano, fuera del control de systemd**: `systemctl` lo marca como `inactive (dead)` mientras el proceso real sigue vivo por su cuenta, y encima deja un `tmux` de una sesión anterior sin matar. Consecuencia real medida: los 15 pedidos automáticos de mantenimiento llevaron horas sin dispararse porque nada los relanzaba. Usa siempre `systemctl --user restart gascity-supervisor.service` para tocar la configuración del supervisor, nunca `gc supervisor stop`/`start` sueltos.
>
> **Cómo se detecta si ya pasó:** `systemctl --user status gascity-supervisor.service` dice `inactive` pero `curl` al panel sigue respondiendo, o `ps aux | grep "gc supervisor run"` muestra un proceso vivo que `systemctl --user stop` no consigue parar. Limpieza: `kill <pid>` de los procesos sueltos (el `gc supervisor run` y cualquier `tmux -u -L gas-city` huérfano), luego `systemctl --user start gascity-supervisor.service`.

**Nota:** el propio `gc supervisor start`/`systemctl` imprime `Dashboard: http://127.0.0.1:8372/` incluso cuando está en `0.0.0.0` — es solo el texto que muestra, normaliza `0.0.0.0`→`127.0.0.1` para ese mensaje. No te fíes de esa línea para saber dónde escucha de verdad; compruébalo con `ss -tlnp` o con un `curl` a la IP real.

## Verificación, la que se hizo de verdad

```bash
ss -tlnp | grep 8372
# LISTEN 0  4096  *:8372  *:*  users:(("gc",pid=...))

curl -s -o /dev/null -w "HTTP %{http_code}\n" http://192.168.1.8:8372/
# HTTP 200
```

Confirmado que devuelve el HTML real del panel (título *«gas city · ds-research»*), no solo un código de estado.

## Queda en solo lectura, a propósito

No se activó `allow_mutations`. Puedes ver agentes, tareas y actividad, pero no arrancar, parar ni mandar nada desde el panel — solo mirar. Es una decisión tomada con el usuario, no un límite técnico: el propio código (`cmd_supervisor.go`) soporta `allow_mutations = true` para poder actuar desde el panel, pero con un aviso deliberado en el código (comentario `G23`) de que esa combinación, sin una clave de autenticación (`write_auth_verify_key`), deja un plano de lectura sin autenticar en toda la red local. Si algún día hace falta interactuar desde el panel (no solo mirar), es una decisión aparte, no una ampliación automática de esto.

## Persistencia

Corre como el mismo servicio de systemd de usuario (`gascity-supervisor.service`, con lingering activado) que ya se instaló al arrancar la ciudad por primera vez. Sobrevive a reinicios de la VM y a cerrar sesión — no hay que repetir estos pasos cada vez, la configuración queda en el fichero.

## Enlaces

- [[_index]] — índice de esta carpeta
- [[gas-city-instalacion-y-modelos]] — la instalación base sobre la que se hizo esto
