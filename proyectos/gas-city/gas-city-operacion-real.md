---
title: Gas City — operación real en esta máquina: modelos, el fallo del diálogo de confianza, y dónde vive el nombre
created: 2026-09-29
updated: 2026-10-04
tags: [gas-city, operacion, modelos, deepseek, opus, bug, probado-en-vivo]
zona: tecnico
---

Tres cosas aplicadas y probadas en vivo el 2026-09-29 sobre la ciudad `NeTT-City` (antes `gas-city`) de esta máquina: el reparto real Opus/DeepSeek, un fallo real que impedía arrancar cualquier sesión, y los tres sitios distintos donde vive el nombre de una ciudad. Complementa a [[gas-city-instalacion-y-modelos]] (que documentaba el reparto sin haberlo aplicado) y a [[gas-city-acceso-externo]] (otro fallo de arranque, causa distinta). Dónde vive la forja de cada rig y qué costaría meter uno en GitLab, en [[gas-city-y-gitlab]].

## 1. El reparto Opus/DeepSeek, aplicado de verdad

Dos agentes, no uno:

| Agente | Modelo | Paga con | Fichero |
|---|---|---|---|
| `mayor` | `opus-5` | La suscripción de Claude Code ya logueada en la máquina (20 €/mes), sin clave de API | `city.toml`, bloque `[[patches.agent]]` |
| `obrero-seek` | `deepseek-flash[1m]` | La clave de DeepSeek, vía `$DEEPSEEK_API_KEY` | `agents/obrero-seek/agent.toml` (fichero propio, no un parche) |

**Por qué `mayor` no lleva `upstream`:** confirmado, textual, de la guía oficial ([docs.gascity.com/guides/harness-recipes](https://docs.gascity.com/guides/harness-recipes), sección «Claude Code — `provider = "claude"`»):
> *«Direct: your existing Claude login or ambient ANTHROPIC_API_KEY»*

Sin `upstream` declarado, el proceso hereda tu sesión ya logueada — no hay campo `upstream` pensado para una suscripción (los `upstreams` de Gas City solo tienen `base_url` + `api_key`/`auth_token`, pensados para pasarela de pago por token).

**La clave de DeepSeek** vive en `~/.gc/secrets.env` (permisos `600`, dotenv: `DEEPSEEK_API_KEY=...`), nunca en claro en `city.toml`. Se referencia como `$DEEPSEEK_API_KEY` en `[upstreams.deepseek]`. Para que sobreviva a un reinicio de la VM y no solo a la sesión que la creó, el servicio de systemd se regeneró con esa variable opt-in explícita:
```bash
GC_SUPERVISOR_ENV=DEEPSEEK_API_KEY gc supervisor install --force
```
Fuente del mecanismo: `docs.gascity.com/getting-started/troubleshooting` §*«Provider Credentials Dropped When the Supervisor Starts»* — el fichero `secrets.env` se fusiona en el entorno del supervisor en cada regeneración del servicio, así el valor sobrevive independientemente de qué shell arrancó `gc start`.

**Un intento fallido, y por qué:** el primer intento fue un `[[patches.agent]] name = "claude"` con `upstream = "deepseek"` para reaprovechar el agente por defecto. Se renombró a `obrero-seek` para no confundirlo con la suscripción real. **Los parches (`[[patches.agent]]`) no pueden renombrar** — solo modifican campos de un agente que ya existe con ese nombre. Un `[[agent]]` nuevo con nombre distinto tampoco vale escrito directamente en `city.toml`: `gc doctor` lo rechaza como *«unsupported PackV1 [[agent]] tables»*. El formato correcto (v2) es un fichero propio, `agents/<nombre>/agent.toml` — y `gc doctor --fix` migra automáticamente un bloque mal puesto a ese formato, sin perder ningún campo (comprobado: `upstream` y `option_defaults` sobrevivieron intactos a la migración).

## 2. El fallo real que impedía arrancar cualquier sesión: el diálogo de confianza de Claude Code

**Síntoma:** ninguna sesión arrancaba. `gc status` daba «0/3 agentes corriendo» sin parar, y el log repetía sin cesar *«no tmux server running»* — un mensaje que apuntaba a `tmux`, pero era una consecuencia, no la causa.

**Causa real, encontrada en `~/.gc/supervisor.log`, no en la documentación oficial (no está documentada en ningún sitio de Gas City para el proveedor `claude`):** el propio `claude` CLI, al arrancar en una carpeta que nunca ha visto, muestra un diálogo interactivo:
> *«Quick safety check: Is this a project you created or one you trust? ... ❯ No, exit / Yes, I trust this folder»*

Como Gas City lo lanza sin nadie delante para contestar, el pane por defecto elegía **«No, exit»** y la sesión moría a los pocos segundos — en bucle, cada vez que el reconciliador reintentaba. `--dangerously-skip-permissions` **no** cubre esto: es un ajuste de permisos de herramientas, no del diálogo de confianza de carpeta. Confirmado en `claude --help`: ese diálogo solo se salta en modo no interactivo (`-p`, o `stdout` sin TTY) — y una sesión de `tmux` sí es un TTY, así que el salto no aplica aquí.

**Efecto colateral que sí es de Gas City, y está documentado en su propio código:** al fallar el arranque una y otra vez, el circuito de seguridad interno pone la sesión en cuarentena. Cinco intentos fallidos seguidos, cinco minutos de espera fija — constantes reales, `cmd/gc/session_types.go`:
```go
defaultQuarantineDuration = 5 * time.Minute
defaultMaxWakeAttempts    = 5
```
**Reiniciar el servicio durante esa ventana no ayuda, la alarga** — cada reinicio cuenta como un intento fallido más.

**El arreglo:** marcar la carpeta de cada agente como de confianza en la configuración de Claude Code, el mismo mecanismo que usa cualquier `claude` interactivo. Vive en `~/.claude.json`, clave `projects.<ruta>.hasTrustDialogAccepted`:
```python
# añadido para /home/netting/gas-city y /home/netting/gas-city/.gc/agents/bd.dog-1
projects["<ruta>"]["hasTrustDialogAccepted"] = True
```
Tras esto, `gc session reset mayor` arrancó a la primera — confirmado en el log: *«Woke session 'mayor', outcome=success»*.

**Consecuencia práctica para cualquier rig nuevo:** cuando se registre un proyecto real con `gc rig add`, esa carpeta también va a disparar el mismo diálogo la primera vez. Hay que repetir este mismo arreglo para la carpeta del rig antes de que el agente trabaje ahí, o la primera sesión morirá igual.

## 3. Dónde vive de verdad el nombre de una ciudad — tres sitios, no uno

Confirmado con la documentación oficial (`reference/config`, sección `Workspace`) y comprobado en vivo cambiando cada uno:

| Sitio | Qué es | Se ve en |
|---|---|---|
| `.gc/site.toml`, clave `workspace_name` | **La identidad real y efectiva de la ciudad en marcha.** Machine-local, la escribe `gc init`. Textual de la documentación: *«Runtime identity now resolves from site binding (.gc/site.toml workspace\_name)... gc init writes the machine-local name to site.toml and omits it from city.toml»* | `gc status`, el panel web |
| El registro del supervisor (`~/.gc/cities.toml`) | Un alias local, se pone con `gc register --name <alias>`. **Nunca se escribe en `city.toml`** | `gc cities` |
| `pack.toml`, `[pack] name` | La identidad del *pack* como unidad reutilizable/compartible — un concepto distinto, no la ciudad en marcha | No aparece en ningún listado de ciudades |

**El error que cometí primero:** cambiar solo `pack.toml` no tuvo ningún efecto visible en `gc cities` ni en `gc status` — confirmado, se quedó en `gas-city` en los dos sitios que importan. Hubo que tocar `.gc/site.toml` **y** volver a registrar con `gc register --name`.

## Verificación

Todo lo de esta nota se comprobó en vivo el 2026-09-29, con la salida real pegada en la conversación de origen: `gc config show`, `gc doctor`, `~/.gc/supervisor.log`, y el código fuente clonado de `gastownhall/gascity`. No es una lectura de documentación sin probar — cada fallo descrito aquí ocurrió de verdad en esta máquina y se resolvió en la misma sesión.

## Enlaces

- [[agentes-comparten-usuario-y-credenciales]] — límite de seguridad: los agentes comparten usuario y credenciales
- [[_index]] — índice de esta carpeta
- [[gas-city-instalacion-y-modelos]] — el reparto de modelos como se diseñó, antes de aplicarlo
- [[gas-city-acceso-externo]] — otro fallo real de arranque de sesión (systemd desincronizado), causa distinta a la de esta nota
- [[gas-city-alcalde]] — cómo se habla con el `mayor` una vez arrancado de verdad
