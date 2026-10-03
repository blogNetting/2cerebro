---
title: Varias cuentas de Claude Code en esta máquina (cc, cc-quien)
created: 2026-10-03
updated: 2026-10-03
tags: [entorno, claude-code, cuentas, suscripcion, gas-city, vscode, mcp]
zona: tecnico
---

Cómo se elige qué suscripción de Claude usa todo (terminales, VS Code, Gas City) sin que cambie nada más, y por qué está montado así.

## Qué se pidió

- Elegir la cuenta con un comando (`cc N`) y que **todo** lo que arranque Claude después use esa: terminal, VS Code y Gas City (`gc session attach mayor`).
- Que nada note que es otra cuenta: mismo modelo, memoria, historial, hooks, plugins, skills, reglas, settings y MCP. **Solo cambia la suscripción.**
- `cc` no abre Claude: solo cambia la cuenta. Después el usuario decide qué lanza (`claude`, Gas City…).
- Que funcione esté donde esté: en cualquier terminal, también en las que ya estaban abiertas, sin `source` ni pasos extra.
- `cc-quien` dice la verdad: lo lee de los ficheros y procesos reales, no de una marca.
- No perder tokens.

## Cómo funciona

1. **Una carpeta por cuenta.** Cuenta 1 = `~/.claude` (+ `~/.claude.json`). Cuenta N = `~/.claude-N`. Claude elige carpeta con la variable `CLAUDE_CONFIG_DIR`.
2. **Todo lo demás es la carpeta de la cuenta 1, por enlaces simbólicos.** En `~/.claude-N` son enlaces a `~/.claude`: `settings.json rules skills CLAUDE.md hooks plugins projects history.jsonl plans sessions session-env file-history shell-snapshots`. Propios de cada cuenta solo quedan `.credentials.json` (token de la suscripción y logins de MCP) y `.claude.json` (identidad: email, organización, cachés de límites).
3. **`cc N`** (`~/cc`):
   - si la cuenta es nueva, crea `~/.claude-N` con esos enlaces;
   - sincroniza los MCP entre **todas** las carpetas `~/.claude*`: cada una queda con la unión de `mcpServers` (en `.claude.json`) y `mcpOAuth` (en `.credentials.json`), con `jq`. No toca `claudeAiOauth` (la suscripción);
   - sincroniza también la configuración por proyecto (`projects` de `.claude.json`): unión de todas las cuentas y, si una carpeta tiene la confianza aceptada (`hasTrustDialogAccepted`) en alguna, queda aceptada en todas;
   - escribe `N` en `~/.cc-activa`;
   - comprueba que `~/.local/bin/claude` es un enlace a `~/bin/claude` y, si no (el actualizador de Claude lo devuelve al binario al instalar una versión), lo vuelve a poner;
   - si la cuenta no tiene login, abre Claude para hacerlo (única excepción en que abre algo); si no, muestra `cc-quien` y termina.
4. **`~/bin/claude`** es un `claude` intermedio: lee `~/.cc-activa`, pone `CLAUDE_CONFIG_DIR=~/.claude-N` y lanza el binario real, la versión más nueva de `~/.local/share/claude/versions/`. No toca la variable si ya viene puesta (un claude lanzado desde otro claude —hooks, subagentes— sigue con la cuenta del padre) ni si hay `ANTHROPIC_BASE_URL` (agentes DeepSeek, que no usan cuenta de Claude).
5. **Quién pasa por `~/bin/claude`:**

| Dónde | Cómo | Fichero |
|---|---|---|
| Terminales, también las ya abiertas | `~/.local/bin/claude` es un enlace a `~/bin/claude`. Es el `claude` que encuentra cualquier shell, aunque se abriera antes del cambio o VS Code ponga `~/.local/bin` delante en el `PATH` | `~/.local/bin/claude` (lo mantiene `cc`) |
| Terminales nuevas (refuerzo) | `export PATH="$HOME/bin:$PATH"` y `alias claude="$HOME/bin/claude"` al final | `~/.bashrc` |
| VS Code | `"claudeCode.claudeProcessWrapper": "/home/netting/bin/claude"`. La extensión pasa su propio binario como primer argumento; el intermedio lo detecta y lo usa | `~/.vscode-server/data/Machine/settings.json` |
| Gas City | `command = "/home/netting/bin/claude"` en `[providers.claude]` | `~/gas-city/city.toml` |

6. **`cc-quien`**: por cada carpeta, el email (de `oauthAccount.emailAddress` de su `.claude.json`), el plan (de `.credentials.json`), cuál está cargada (`~/.cc-activa`) y cuántos claude hay abiertos con ella (leyendo `CLAUDE_CONFIG_DIR` de `/proc/<pid>/environ`; sin variable = cuenta 1).

## Uso

```
cc            # = cc-quien
cc 2          # cambia a la 2 y vuelve al prompt; lo que arranques después usa la 2
cc 3          # cuenta nueva: crea ~/.claude-3 enlazada y abre Claude para el login
```

Arrancar el mayor: `/home/netting/load_gas_city_command` contiene `cd /home/netting/gas-city && gc session attach mayor`.

Lo que ya esté abierto sigue con su cuenta hasta cerrarlo y abrirlo de nuevo. Para el mayor de Gas City:

```
cc 2
cd ~/gas-city && gc session reset mayor && gc session attach mayor
```

`gc session reset` reinicia la conversación del mayor.

## Por qué así y no de otra forma

| Opción | Por qué no |
|---|---|
| Una sola carpeta y copiar el token de cada cuenta dentro | Pierde tokens. El refresh token es de un solo uso — clauth: «OAuth refresh tokens are single-use» ([docs.rs/crate/clauth/0.7.3](https://docs.rs/crate/clauth/0.7.3)); claude-swap: «A session refreshes its own copy of the account's token» y se niega a activar una copia atrasada porque «activating it could only fail» ([github.com/realiti4/claude-swap](https://github.com/realiti4/claude-swap)). Con sesiones abiertas de otra cuenta, una renovación pisa el token de la cargada |
| Enlace `~/.claude-activa` que `cc` cambia de destino | Una sesión abierta seguiría el enlace y renovaría su token en la carpeta de otra cuenta (inferencia, no reproducido) |
| `CLAUDE_CONFIG_DIR` en `~/.profile` | No cambia al momento: solo tras volver a iniciar sesión en la máquina |
| claude-swap / clauth | Hacen el cambio de token con copia de vuelta y bloqueos; no conocen Gas City ni `cc`. No probadas aquí |

Ambas herramientas son proyectos que venden el cambio de cuenta: no son neutrales, pero coinciden entre sí. clauth usa el mismo diseño que este para sesiones en paralelo: carpeta por perfil con enlaces a `~/.claude` y `.claude.json` propio, para que «account identity and billing caches never leak between profiles».

## Verificado (2026-10-03)

- **Terminal vieja** (sin alias, sin `~/bin` en el `PATH`, `~/.local/bin` delante, como una de VS Code abierta días antes): `type claude` da `~/.local/bin/claude` y `claude auth status` da el email de la cuenta 2 con la 2 cargada y el de la 1 con la 1. Confirmado por el usuario en uso real.
- `cc 2` termina con `exit=0` sin abrir Claude.
- Con `~/.cc-activa` = 2, un `bash -i` nuevo resuelve `claude` a `/home/netting/bin/claude` y `claude auth status` da el email de la cuenta 2; con 1, el de la cuenta 1. Lo mismo simulando VS Code (`~/bin/claude <binario de la extensión> auth status`).
- `claude mcp list` da `context7 … ✔ Connected` en las dos cuentas tras la sincronización.
- Tras la sincronización, `claudeAiOauth` de cada cuenta es idéntico al previo (comparado con copia).
- `gc config show` resuelve `command = "/home/netting/bin/claude"`.

- Con la 2 cargada, `gc session attach mayor` arranca el mayor con `CLAUDE_CONFIG_DIR=/home/netting/.claude-2` (leído de `/proc/<pid>/environ`) y trabaja sin aviso de límite.

No verificado: VS Code real tras recargar la ventana.

## Problemas conocidos

- **Sin la confianza de la carpeta, el mayor muere al arrancar** (2026-10-03). Claude pregunta «Is this a project you created or one you trust?» sobre `/home/netting/gas-city`, Gas City no puede contestar y la sesión cae (`.gc/sessions/mayor/start-stderr.log`, `provider_error`). La confianza se guarda por cuenta en `.claude.json`; por eso `cc` la sincroniza.

- **Cambiar `[providers.claude]` en `city.toml` reinicia el mayor.** Gas City detecta el cambio de configuración y reinicia las sesiones afectadas (estado `session,config`). Pasó el 2026-10-03 al cambiar `command`.
- **Un MCP borrado reaparece**: la sincronización es una unión. Para quitarlo, quitarlo en todas las cuentas antes de volver a ejecutar `cc`.
- **Un claude abierto reescribe su `.claude.json` al cerrar** y puede deshacer la sincronización de esa cuenta; el siguiente `cc` la vuelve a hacer.
- **VS Code con `claudeProcessWrapper`**: si no se ha elegido modo de permisos, la extensión arranca en `default` (visto en su `extension.js`, 2.1.288). Hay que recargar la ventana para que use el ajuste.
- **Terminales abiertas antes del cambio de `~/.bashrc` usaban la cuenta 1** (2026-10-03). No tenían `~/bin` delante ni el alias, y las de VS Code ponen `~/.local/bin` delante en el `PATH`. Un `claude` lanzado ahí salía con la 1, que no tenía cuota («You've hit your weekly limit»), aunque `cc 2` estuviera bien. Una shell abierta no se puede cambiar desde fuera, así que el arreglo fue convertir `~/.local/bin/claude`, el que todas encuentran, en enlace al intermedio.
- **El actualizador de Claude reescribe `~/.local/bin/claude`** apuntándolo al binario nuevo. Hasta el siguiente `cc`, un `claude` de una terminal sin alias saldría con la cuenta 1. `cc` lo repara cada vez que se ejecuta.
- **Un claude ya abierto no cambia de cuenta**: hay que cerrarlo (`/exit`) y relanzarlo.
- Lo de la cuenta 2 anterior a enlazarla (2026-10-03) quedó en `~/.claude-2/.antes-de-enlazar/`.

## Si algo falla

- `cc-quien` primero: dice qué cuenta está cargada y con cuál corre cada claude abierto.
- `readlink ~/.local/bin/claude` debe dar `/home/netting/bin/claude`. Si da `…/versions/X`, ha actualizado Claude: ejecutar `cc N` lo repara.
- `claude auth status | jq -r .email` dice con qué cuenta arranca un claude nuevo.
- `tr '\0' '\n' < /proc/<pid>/environ | grep CLAUDE_CONFIG_DIR` dice con qué cuenta corre un proceso concreto.

## Enlaces

- [[entorno]] — resto de herramientas de esta máquina
- [[gas-city-instalacion-y-modelos]] — configuración de proveedores y modelos de Gas City
- [[decisiones]] — entrada del 2026-10-03
