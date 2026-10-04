---
title: Gas City — instalación en esta máquina y configuración de modelos (Opus/Sonnet/DeepSeek)
created: 2026-09-29
updated: 2026-10-04
tags: [gas-city, instalacion, modelos, deepseek, opus, sonnet, telemetria, privacidad]
zona: tecnico
---

Cómo se instala Gas City en esta máquina, cómo se reparten los modelos entre Opus, Sonnet y DeepSeek, y qué hay que apagar. Complementa a [[gas-city-traje-a-medida]] (por qué Gas City) y a [[gas-city-frente-a-la-fabrica]] (qué mecanismos sirven y cuáles son marketing).

## 1. Introducción: qué se pregunta y por qué

Tres preguntas concretas, con criterio de admisión declarado **antes** de buscar: entra lo que esté documentado en el repositorio o la documentación oficial con la ruta exacta (fichero o clave de configuración) y sea verificable leyendo el código. No entra el blog que resume, ni el vídeo, ni «se supone que».

1. Cómo se instala en Ubuntu 24.04, usuario `netting`.
2. Cómo se configura con Opus/Sonnet **y** DeepSeek a la vez.
3. Qué hay que desactivar: contribución automática al repositorio del fabricante, telemetría, y bucles de auto-mejora.

El estado de la máquina se comprobó en vivo (no de memoria): **Ubuntu 24.04.4, x86_64**. Presentes `tmux`, `jq`, `git`, `flock`, `gh` (2.101.0). **Ausentes `dolt` y `bd`** — las dos dependencias duras. Y un hallazgo que condiciona todo lo demás, en el entorno:

```
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro[1m]
ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-flash[1m]
```

**Claude Code hoy corre entero sobre DeepSeek**, con Opus y Sonnet *remapeados* a modelos DeepSeek. No hay ninguna credencial de Anthropic en el entorno. Eso convierte «configurar Opus/Sonnet y DeepSeek» en un problema de reparto, no de instalación — y deja la mitad «Opus/Sonnet» **no utilizable hasta que exista una clave de Anthropic** (ver §5.1).

## 2. Considerado y descartado

| Qué | Motivo del descarte |
|---|---|
| **Reabrir si adoptar Gas City** | Ya resuelto en [[gas-city-frente-a-la-fabrica]]: sirve como montaje personal, y el obstáculo de fondo —que no da aislamiento forzado del verificador— sigue ahí. No se reabre. |
| **Homebrew como vía principal** | Es la vía que la propia documentación recomienda (*«Most users should use Homebrew»*, [installation](https://docs.gascity.com/getting-started/installation)), pero arrastra una dependencia del ecosistema macOS/Linuxbrew en una máquina Ubuntu. Se documenta como alternativa, no como camino principal. |
| **Instalar dolt + bd + flock sin más** | Son las dependencias que faltan, pero **son evitables**: `GC_BEADS=file` las salta. Descartado como requisito obligatorio; entra como decisión (§3). |
| **Un proxy o router de modelos (LiteLLM, claude-code-router)** | Descartado por innecesario: Gas City tiene el eje *upstream* de primera clase y DeepSeek ya expone un endpoint compatible con Anthropic. Añadir un proxy sería una pieza más que mantener para hacer lo que la herramienta ya hace. |
| **Hackear el esquema de modelos para meter DeepSeek** | Descartado al comprobar que **no hace falta**: la opción `model` del harness `claude` es abierta (§5.2). |

## 3. Instalación: dos rutas, y la decisión que las separa

La pregunta que decide no es «cómo», es **qué almacén de tareas usar**. Gas City guarda el trabajo en *beads* (unidades de trabajo con estado, dependencias y relaciones), y ese almacén tiene dos backends.

| | Backend `file` | Backend `bd` + `dolt` |
|---|---|---|
| Qué instalar | Nada más | `dolt` ≥ 2.1.0 y `bd` ≥ 1.0.4 (v1.3.0 probada), y `flock` |
| Cómo se activa | `export GC_BEADS=file`, o `[beads] provider = "file"` en `city.toml` | Por defecto |
| Para qué sirve | Probar Gas City en local | Trabajo real |

Las dos citas que deciden, textuales de [troubleshooting](https://docs.gascity.com/getting-started/troubleshooting):

> *«The file provider is fine for trying Gas City locally. The `bd` provider adds durable versioned storage and **is recommended for real work**.»*

Y de [FAQ](https://docs.gascity.com/getting-started/faq):

> *«For the lightest possible start, `GC_BEADS=file` skips the dolt + bd pair.»*

**Consecuencia práctica y recomendación:** empezar con `GC_BEADS=file` para probar sin instalar nada, y añadir `dolt` + `bd` cuando el montaje se use para trabajo real — que es lo que pide el objetivo. No es un atajo equivalente: el backend `file` no da el almacén versionado. **Queda dicho para que la decisión sea suya y no un descuido.**

### 3.1. Versiones exactas

Las que fija su propio CI, en [`deps.env`](https://github.com/gastownhall/gascity/blob/main/deps.env) del repositorio (comprobado en el clon del 2026-09-29):

```
DOLT_VERSION=2.1.7
BD_VERSION=v1.3.0        # suelo mínimo: v1.0.4
```

La documentación añade una advertencia sobre Dolt que conviene respetar: *«Gas City's managed Dolt checks reject older and pre-release builds because they are below the managed bd/Dolt compatibility floor»* — de ahí que el mínimo sea 2.1.0 y no una versión cualquiera.

### 3.2. Los comandos, para esta máquina (x86_64)

**Vía tarball** (sin Homebrew, verificable — la que encaja en esta máquina). Del [installation](https://docs.gascity.com/getting-started/installation), con la verificación que el propio documento indica:

```bash
VERSION=<la última de https://github.com/gastownhall/gascity/releases>
curl -fsSLO "https://github.com/gastownhall/gascity/releases/download/v${VERSION}/gascity_${VERSION}_linux_amd64.tar.gz"
curl -fsSLO "https://github.com/gastownhall/gascity/releases/download/v${VERSION}/gascity_${VERSION}_checksums.txt"
grep "  gascity_${VERSION}_linux_amd64.tar.gz$" "gascity_${VERSION}_checksums.txt" > arc.sha256
sha256sum -c arc.sha256
gh attestation verify "gascity_${VERSION}_linux_amd64.tar.gz" --repo gastownhall/gascity
tar -xzf "gascity_${VERSION}_linux_amd64.tar.gz"
sudo install -m 755 gc /usr/local/bin/gc
gc version
```

Dependencias de sistema, solo si se va por el backend `bd`:

```bash
sudo apt install tmux jq git util-linux        # tmux, jq, git y flock: ya están
# dolt y bd no están en apt: se bajan de
#   https://github.com/dolthub/dolt/releases               (>= 2.1.0; CI fija 2.1.7)
#   https://github.com/gastownhall/beads/releases          (>= 1.0.4; CI fija v1.3.0)
```

**Dos trampas que conviene saber antes:**

- **El alias `gc`.** La documentación avisa de que con Oh My Zsh y su plugin `git`, `gc` ya significa `git commit --verbose`. **Comprobado en esta máquina el 2026-09-29: no aplica** — shell `bash`, sin `~/.oh-my-zsh`, y ningún alias `gc` definido en los rc. Queda anotado por si algún día se cambia de shell, no como tarea pendiente.
- **`gc init` instala un servicio.** El arranque no es un proceso suelto: la documentación habla de *«restarts the launchd/systemd service»* y el código tiene detección explícita de supervisor gestionado por systemd (`cmd/gc/cmd_start_drift.go`). Es decir, **Gas City deja un servicio de fondo que arranca solo**. Es reversible (`gc stop <city-path>`, `gc service restart`), pero no es un detalle: es una pieza que queda residiendo en la máquina.

**Arranque, del [quickstart](https://docs.gascity.com/getting-started/quickstart):**

```bash
export GC_BEADS=file            # solo para la prueba sin dolt/bd
gc init ~/mi-ciudad
cd ~/mi-ciudad
```

### 3.3. Qué se instala de una y qué se escoge

Respuesta corta: **el binario se instala de una; la ciudad se escoge.** Gas City no es un TODO monolítico, y la prueba está en su propio diseño de packs ([reference/system-packs](https://docs.gascity.com/reference/system-packs)):

> *«Built-in packs are **not implicit**: nothing splices them into config composition at load time. They compose only through **explicit pinned imports** in `pack.toml`, which `gc init` writes for you.»*

Es decir: los packs vienen **embebidos en el binario** pero **no se activan solos** — se componen por imports explícitos, y `gc init` los escribe por ti. Nada se materializa en la ciudad; el binario solo pre-siembra su caché.

| Pieza | Qué trae | ¿Se escoge? |
|---|---|---|
| **`gc` (binario)** | El orquestador. Go, estático, un fichero | No: se instala y ya |
| **Pack `core`** | Skills `gc-*`, prompts de obrero por defecto, las fórmulas base, **las 15 órdenes de mantenimiento**, chequeos de doctor, overlays de hooks por proveedor. **No trae ningún agente**: *«The core pack deliberately ships no agents.»* | Es la base; se importa casi siempre |
| **Pack `bd`** (y `dolt` transitivo) | El almacén de tareas. **Solo se escribe si usas el proveedor `bd`** | Sí — con `GC_BEADS=file` **no entra** |
| **Pack `gastown`** | **Los roles**: mayor, deacon, refinery, polecat, witness, y el pool `dog` con su `mol-shutdown-dance` | Sí — es lo que te da mano de obra |
| **Pack `gascity`** | La plantilla de por defecto | Sí |
| Packs del registro | Lo que importes (incluido `contributing`, que es opt-in) | Sí, uno a uno |

**Consecuencia que conviene entender antes de montar nada: una ciudad solo con `core` tiene mantenimiento y**ninguna mano de obra**.** Los agentes que hacen el trabajo llegan con `gastown` o con los packs que tú escribas. Por eso «instalar Gas City» no significa lo mismo que «tener una fábrica»: lo primero es un binario, lo segundo es una decisión de packs.

**`gc init` pregunta.** No hay que saberse las plantillas de memoria: hay un asistente interactivo que imprime `Choose a config template:` y deja elegir también el CLI de agente. Para hacerlo por flags, del propio `--help`:

```bash
gc init --template gastown --default-provider codex ~/mi-ciudad
gc init --template gascity --default-provider claude ~/mi-ciudad
```

Plantillas disponibles: `gascity` (la de por defecto) y `gastown`; y en `examples/` del repositorio hay más — `hyperscale`, `swarm`, `storage`, `bd`, `lifecycle`, `t3bridge-gastown`.

**Y después se añade o se quita**, sin reinstalar nada:

```bash
gc import add <source>          # añadir un pack
gc import remove <nombre>       # quitarlo
gc import list                  # ver los que tienes
gc import why <nombre>          # por qué está ese import ahí (lo trajo otro, o lo pusiste tú)
gc import upgrade               # subir los packs dentro de sus restricciones
gc import check                 # validar que el estado de los imports cuadra

gc skill list                   # qué skills aportan los packs cargados
gc formula list                 # qué fórmulas hay
gc order list                   # qué órdenes van a correr
```

`gc order list` es la comprobación que de verdad importa antes de dejar una ciudad corriendo: **es donde se ve qué se va a disparar solo.**

**Una trampa, y de las que no se ven:** los ficheros de los packs builtin **no son una superficie de personalización**. Dice la documentación, textual:

> *«The cached files are implementation assets owned by `gc`. They are useful for learning and debugging, but **local edits are not a stable customization surface (the binary restores its embedded content)**. Put custom behavior in your own city files or packs instead.»*

O sea: **si algún día hay que cambiar algo, editar el pack no sirve** — el binario restaura lo suyo. Todo lo tuyo va en `city.toml` y en packs propios. Es también el motivo por el que la respuesta a «¿hay que modificar Gas City?» es que no (§5).

## 4. Configuración de modelos: los cinco ejes

Esto es el núcleo de la respuesta, y no funciona como se supone de memoria. Un agente en Gas City se define por **cinco ejes independientes**, según [Configuring an Agent](https://docs.gascity.com/guides/configuring-an-agent):

| Eje | Pregunta | Dónde se pone | Ejemplo |
|---|---|---|---|
| **Harness** | ¿qué CLI de agente? | agente `provider` | `provider = "claude"` |
| **Modelo** | ¿qué etiqueta de modelo? | agente `option_defaults.model` | `option_defaults = { model = "opus" }` |
| **Upstream** | ¿quién sirve el modelo? | agente `upstream` + `[upstreams.<nombre>]` | `upstream = "deepseek"` |
| **Transport** | ¿cómo lo maneja gc? | agente `session` | `session = "acp"` (por defecto: `tmux`) |
| **Runtime** | ¿dónde corre? | ciudad `[session] provider` | `provider = "k8s"` (por defecto: `tmux`) |

El eje **upstream** es el que resuelve el reparto Opus/Sonnet/DeepSeek, y su regla es exactamente la que interesa: *«Switching upstream changes the base URL and credentials the harness talks to, **without changing the model, the harness, or the box**»*.

### 4.1. La trampa específica de esta máquina

Textual de [Harness Recipes](https://docs.gascity.com/guides/harness-recipes):

> *«**Direct usually needs no upstream block at all.** If the harness already finds its credentials in the environment (your normal login or `*_API_KEY`), just set `provider` and go — **Gas City passes the ambient environment through**.»*

En una máquina normal eso es comodidad. Aquí hay que entender **por qué** es cierto, porque el motivo no es el que parece. La lista de variables que Gas City reenvía *explícitamente* al agente es corta y está en `internal/processenv/provider.go`:

```
PATH, HOME, USER, LOGNAME, TZ, CLAUDE_CONFIG_DIR,
CLAUDE_CODE_OAUTH_TOKEN, CLAUDE_CODE_SUBAGENT_MODEL,
CLAUDE_CODE_EFFORT_LEVEL, CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC, LANG, LC_*
```

**`ANTHROPIC_BASE_URL` no está ahí.** Lo que ocurre es otra cosa, y está escrita con precisión en `internal/processenv/provider.go`: el mapa de variables *«is an overlay on an environment the child already inherits, **not the whole environment**»*, y *«a **tmux pane starts from the tmux server's global env, which holds whatever the controller exported when the server started**»*. Es decir: **tu `ANTHROPIC_BASE_URL` llega al agente por herencia del servidor tmux, no porque Gas City lo reenvíe.**

Consecuencia práctica, y esta vez con el motivo correcto: **un agente `claude` sin `upstream` declarado hereda el DeepSeek del entorno.** No es una suposición sobre una lista, es cómo funciona la herencia. Y tiene un corolario incómodo: **depende de cómo se arrancó el servidor tmux**, no de nada que esté escrito en tu `city.toml`.

La buena noticia, y esta sí está en el fichero del arranque (`cmd/gc/template_resolve.go`, paso 10b):

> *«Inject the selected upstream's serving env **LAST so it is authoritative for the model-serving keys**, and after `ScrubTokenEnv` so its credential refs survive.»*

Es decir: **el upstream declarado gana al entorno heredado.** Declarar el upstream no es cosmético: es lo único que hace que el reparto sea explícito y no dependa de cuándo arrancó el tmux.

**Y una segunda regla que sale de aquí, más importante todavía: fija el `model` en todos los agentes.** El harness `claude` **no tiene modelo por defecto** — su `OptionDefaults` en `internal/worker/builtin/profiles.go` solo trae `permission_mode` y `effort`, sin `model`. Un agente que no pinche `model` **no emite ningún `--model`**, y entonces decide el propio CLI de Claude Code: lo que traiga de fábrica o, en esta máquina, tus `ANTHROPIC_DEFAULT_OPUS_MODEL` / `ANTHROPIC_DEFAULT_SONNET_MODEL` globales, que apuntan a modelos DeepSeek. Es decir: **sin `model` explícito, un agente `upstream = "anthropic-direct"` puede acabar pidiendo a Anthropic un nombre de modelo DeepSeek.** Pinnear el modelo en cada agente no es estilo: es lo que evita ese cruce.

### 4.2. La segunda buena noticia: no hay lista cerrada de modelos

`option_defaults.model` es una opción **abierta**. Comentario literal de `internal/worker/builtin/profiles.go` (línea 277, comprobado en el clon):

> *«Open: an id outside this curated list is honored verbatim rather than dropped.»*

Y en `internal/config/provider.go`: *«Model ids are an open, fast-moving set: every provider ships new ones.»* La documentación lo confirma: *«override or extend a harness's `options_schema` to add your own.»*

**Consecuencia:** `model = "deepseek-flash[1m]"` sobre el harness `claude` **funciona tal cual**, sin tocar el esquema ni inventar un provider. El propio código dice que el sufijo `[1m]` *«is emitted verbatim»* — que es exactamente la forma que ya usa este entorno.

### 4.3. Los alias que sí expanden

Conviene saberlo porque el valor que escribes no es el que llega al CLI. De `profiles.go`, los alias del harness `claude` expanden así:

| Escribes | Llega al CLI |
|---|---|
| `opus` | `claude-opus-4-8` |
| `opus-5` | `claude-opus-5` |
| `sonnet` | `claude-sonnet-5` |
| `haiku` | `claude-haiku-4-5-20251001` |

Un id canónico (`claude-opus-5`, `claude-sonnet-5`) pasa tal cual. Y una etiqueta ajena a la lista (`deepseek-flash[1m]`) también, que es el caso que importa aquí.

### 4.4. La configuración concreta para esta máquina

**Una regla que no es opcional: `base_url` se declara SIEMPRE, en todos los upstreams.** El motivo está en el código y es la trampa más peligrosa de todo este montaje.

El render del upstream (`cmd/gc/template_resolve.go`) recorre los campos abstractos y **salta los que están vacíos**:

```go
for _, r := range []struct{ value, override, bound, field string }{
    {spec.BaseURL, spec.BaseURLEnv, binding.BaseURL, "base_url"},
    {spec.APIKey,  spec.APIKeyEnv,  binding.APIKey,  "api_key"},
    {spec.AuthToken, spec.AuthTokenEnv, binding.AuthToken, "auth_token"},
} {
    if r.value == "" {
        continue          // ← campo vacío: no se exporta nada
    }
```

Y ese mapa, como dice `internal/processenv/provider.go` con todas las letras, **no es el entorno entero**:

> *«These are SEEDED EMPTY rather than omitted, because the map this package returns is **an overlay on an environment the child already inherits, not the whole environment**. A tmux pane starts from the **tmux server's global env, which holds whatever the controller exported when the server started** (…) **declining to re-export a key leaves the controller's value visible to the child**. Only an explicit `KEY=""` overrides it.»*

Suma las tres piezas — el `continue` del render, el overlay sobre el entorno heredado, y que `ScrubTokenEnv` **solo** vacía el token del controlador (`internal/convergence/acl.go`), no los `ANTHROPIC_*` — y sale esto:

**Un upstream para Anthropic declarado solo con `api_key` y sin `base_url` deja el `ANTHROPIC_BASE_URL` heredado del servidor tmux apuntando a DeepSeek. El resultado no es un error: es tu clave de Anthropic enviada a un tercero.** Por eso `base_url` va explícito en todos, incluso en el «directo», que en una máquina normal no lo necesitaría. En **esta** máquina, no declararlo es un fallo de seguridad, no una omisión.

Dos upstreams en `city.toml`. Los secretos **nunca** en claro: se referencian con `$VAR`, que se expanden desde el entorno del controlador.

```toml
# city.toml
[upstreams.anthropic-direct]
description = "Anthropic de verdad: Opus 5 y Sonnet 5"
base_url    = "https://api.anthropic.com"   # OBLIGATORIO: sin esto hereda el DeepSeek del tmux
api_key     = "$ANTHROPIC_API_KEY"          # requiere una clave que HOY NO EXISTE en el entorno

[upstreams.deepseek]
description = "DeepSeek por su endpoint compatible con Anthropic"
base_url    = "https://api.deepseek.com/anthropic"
auth_token  = "$DEEPSEEK_API_KEY"           # ya está en el entorno
```

Y un cinturón de seguridad para el caso de Anthropic, con la escotilla cruda (`[env]`), que se fusiona **la última** y gana a todo lo demás que toque esa clave (*«Raw env is the harness-specific escape hatch, merged LAST (wins over the abstract render and ambient/agent env for the keys it sets)»*):

```toml
[upstreams.anthropic-direct.env]
ANTHROPIC_BASE_URL = "https://api.anthropic.com"
```

El campo abstracto se traduce solo al nombre de variable del harness. Para `claude` el binding es `ANTHROPIC_BASE_URL` + `ANTHROPIC_API_KEY`, y `ANTHROPIC_AUTH_TOKEN` cuando se usa `auth_token` — que es el caso de DeepSeek, un gateway con token portador. **Un mismo upstream vale para cualquier harness** (en un agente `codex` esos mismos campos se renderizan a `OPENAI_BASE_URL`/`OPENAI_API_KEY`), y un campo abstracto sin binding es **error duro**, no un no-op silencioso.

**Y compruébalo, no lo supongas:** `gc config explain --agent <nombre>` muestra el entorno resuelto del agente, incluido el `ANTHROPIC_BASE_URL` que va a usarse de verdad. Es la única forma barata de confirmar que tu clave no está yendo a donde no debe. Antes de lanzar trabajo real, esa comprobación para cada agente.

Y los agentes:

```toml
# agents/planificador/agent.toml — piensa con el modelo bueno
dir             = "mi-app"
provider        = "claude"
upstream        = "anthropic-direct"
option_defaults = { model = "opus-5" }          # → --model claude-opus-5

# agents/obrero/agent.toml — ejecuta con Sonnet
dir             = "mi-app"
provider        = "claude"
upstream        = "anthropic-direct"
option_defaults = { model = "sonnet" }          # → --model claude-sonnet-5

# agents/tarea-mecanica/agent.toml — barato, DeepSeek
dir             = "mi-app"
provider        = "claude"
upstream        = "deepseek"
option_defaults = { model = "deepseek-flash[1m]" }   # id no listado: pasa tal cual
```

Y para que **ningún agente caiga en el ambiente por olvido**, un valor por defecto explícito en la ciudad:

```toml
[agent_defaults]
upstream = "anthropic-direct"     # quien no declare upstream, va aquí — no al entorno
```

**Nota honesta sobre el reparto:** un upstream declarado en `city.toml` es *configuración de confianza*, al mismo nivel que el comando que ejecuta un harness. Un pack importado puede traer sus propios `[upstreams]` y pisar los de la ciudad por nombre. La documentación lo dice explícito: *«only import packs you trust»*. Traducción para este montaje: **no importar packs ajenos sin leer sus upstreams primero**, porque un pack malicioso o descuidado podría redirigir dónde van tus credenciales.

## 5. Qué hay que desactivar: los tres frentes

Esta era la pregunta concreta. Los tres frentes tienen respuesta distinta, y solo **uno** requiere acción.

### 5.1. Contribución automática al repositorio del fabricante — **no existe. Nada que apagar.**

El origen de la pregunta es [`gastownhall/gastown#3649`](https://github.com/gastownhall/gastown/issues/3649):

> *«Does Gas Town "steal" usage from users' LLM credits & paid services **to improve itself**?»* (2026-04-14, cerrado 2026-05-01)

El cuerpo del issue describe el mecanismo con precisión: *«Your Claude credits / usage may be funding fixes to the maintainer's codebase, and your GitHub account submitted PRs to his repo. This happens because GasTown ships with a "contribute back to upstream" workflow baked into the formula set.»* El vector concreto eran dos fórmulas (`gastown-release.formula.toml`, `beads-release.formula.toml`).

**Cómo se cerró, textual:** *«Gastown is in maintenance mode and staying focused on infrastructure and reliability fixes only. If you want to pursue broader product/policy work like this, please check out Gas City instead.»* — es decir, **no se cerró diciendo «lo hemos quitado», se cerró redirigiendo a Gas City**. Eso obliga a comprobarlo en Gas City y no darlo por heredado. Comprobado, en el repositorio y en el catálogo oficial de packs (2026-09-29):

| Comprobación | Resultado |
|---|---|
| Fórmulas del pack `core` | `mol-do-work`, `mol-polecat-{base,commit,report}`, `mol-prompt-synth`, `mol-review-quorum`, `mol-scoped-work`. **Ninguna de release.** |
| Búsqueda de fórmula de release en todo el catálogo de packs | **Vacío.** |
| `gastownhall/*` en el código Go | Solo **rutas de import del propio módulo**. Ninguna llamada de red a su repo. |
| `gh pr create` / `git push` en los packs de serie | `mol-polecat-base` **prohíbe** explícitamente instalación, limpieza, publicación, release y `git push`. |
| ¿Existe la función en algún sitio? | Sí: el pack **`contributing`**, aparte, opt-in, y su cabecera lo dice sin ambigüedad. |

La cabecera de `contributing/pack.toml`, textual:

> *«Contributing — the external-contributor lifecycle for **gastownhall/gascity**, distributed as a Gas City pack. Gives an outside contributor the full journey of landing work in https://github.com/gastownhall/gascity (…)»*

Es decir: la función existe, **pero como pack que hay que importar a propósito, y cuyo propósito declarado es que tú contribuyas**. No se dispara sola, no está en `core`, y no hay nada que desactivar.

**Y una distinción que sí conviene hacer**, porque es donde podría confundirse: Gas City **sí abre ramas y las funde en tu repositorio**, mediante el *refinery*. La tabla de fórmulas de `internal/bootstrap/packs/core/skills/gc-dispatch/SKILL.md` dice cuál es la de por defecto para agentes en pool:

> *«`mol-polecat-work` — worktree + feature branch — pushes the branch and **reassigns to the refinery** for merge review (…) **the default for pooled polecats**»*

Eso es **tu** repo, no el suyo: *«which merges it into the rig's own repo»*. Y si algo tuviera que acabar en **otro** repo, la misma guía dice que la fórmula no aplica: *«the refinery has nothing to merge (…) open the PR yourself»*. Es decir, **no abre PRs por su cuenta en ningún caso**. Si lo que quieres es un agente que **no toque git en absoluto**, existe `mol-polecat-report`: *«No git checkout, no feature branch, no push, no PR.»*

### 5.2. Telemetría — **existe, viene activada, y se apaga con dos variables.**

Aquí **corrijo una conclusión mía a mitad de la investigación**, y la dejo anotada en vez de disimularla. Primero verifiqué el paquete `internal/telemetry` (OpenTelemetry) y su comentario es tajante: *«Returns (nil, nil) if none of GC_OTEL_METRICS_URL, GC_OTEL_LOGS_URL, and OTEL_EXPORTER_OTLP_ENDPOINT is set, **so that telemetry is strictly opt-in**»* — y aun activándola, los endpoints por defecto son `localhost:8428` / `localhost:9428`, tu propia máquina, para VictoriaMetrics/VictoriaLogs. Con eso di el frente por zanjado.

**Estaba incompleto.** Hay un **segundo subsistema**, `internal/productmetrics`, que sí sale de la máquina. Su texto de divulgación compilado (`internal/productmetrics/notice_content.go`) dice, literal:

> *«Gas City collects anonymous usage metrics from the gc command line to understand how gc is used and where to improve it.
> What's collected: the command name, the gc version, your operating system, and an anonymous installation ID. Nothing else — no command arguments, file names or contents, paths, environment values, IP addresses, or personal data.
> **This is enabled by default.** You can turn it off at any time: run `gc metrics off`, or set `DO_NOT_TRACK=1` or `GC_DISABLE_USAGE_METRICS=1`.
> **Metrics are never collected in CI, scripted, or agent-managed sessions**, and this first run has not been recorded.»*

Los cuatro hechos que importan:

| Hecho | Fuente |
|---|---|
| **Viene activada por defecto** (opt-out, no opt-in) | Texto de divulgación, `notice_content.go` |
| Se **divulga entera antes** del primer registro, en TTY interactiva, y exige aceptación (`gc metrics on`) | [CLI reference](https://docs.gascity.com/reference/cli): *«Read and accept the command-usage disclosure on a verified TTY»* |
| Recoge **solo**: nombre de comando, versión, SO, ID anónimo de instalación. Nunca argumentos, rutas, contenidos, entorno | Texto de divulgación + `CHANGELOG.md` |
| **Nunca se recoge en sesiones de agente, CI o scripts** | Texto de divulgación |

Ese cuarto punto es el que cierra tu caso: **en una ciudad de agentes no se recoge nada.** La telemetría de producto solo ve los comandos `gc` que teclees tú en una terminal. Aun así, apagarla es una línea:

```bash
gc metrics off                 # apaga y ADEMÁS borra los datos locales en cola
# o, a nivel de entorno:
export DO_NOT_TRACK=1
export GC_DISABLE_USAGE_METRICS=1
```

Y `gc metrics status` muestra el estado redactado; `--show-installation-id` enseña el pseudónimo estable de instalación, con aviso.

**Recomendación:** dado que no aporta nada a tu uso y que el coste de apagarla es una línea, apagarla. No por desconfianza —la divulgación es ejemplar— sino porque **una máquina que no manda nada no tiene que justificar nada**.

### 5.3. Bucle de auto-mejora — **no mejora Gas City; sí mantiene tu ciudad, y algunas piezas borran cosas.**

Gas City trae **órdenes autónomas** (`orders/`): trabajos que se disparan por temporizador o por evento sin que nadie los pida. Son 15 en el pack `core`. **La clave es qué hacen**, y la documentación lo deja claro: son mantenimiento de tu propia ciudad, no mejora de Gas City. Y son **mecánicas, sin LLM**:

> *«Gate evaluation is 100% mechanical — timer comparison and GitHub API status decoding. No LLM judgment needed. The controller runs this directly via exec instead of burning agent context.»* (`orders/gate-sweep.toml`)

Los que importan para decidir, y por qué:

| Orden | Qué hace | ¿Toca algo tuyo? |
|---|---|---|
| `jsonl-export` (15 min) | Copia de seguridad: exporta cada almacén de beads a JSONL en un repo git local | **Solo local por defecto** (sin remoto `origin`, no empuja nada) |
| `reaper` (30 min) | Purga moléculas cerradas y huérfanas | Borra registros de trabajo ya cerrados |
| `prune-branches` (6 h) | *«Clean stale gc/\* branches from all rigs»* | **Borra ramas en tus repos** (las que creó él, `gc/*`) |
| `orphan-sweep` (5 min) | Devuelve al pool los beads de agentes muertos | Reasigna, no borra |
| `wisp-compact` (1 h) | Limpia beads efímeros caducados | Borra efímeros |
| `spawn-storm-detect` (5 min) | Detecta agentes en bucle de caída | Solo avisa |

**Cómo se apaga**, con la clave oficial de `city.toml` (esquema: *«Orders configures order settings: skip list, max_timeout cap, and per-order overrides»*):

```toml
[orders]
skip = ["prune-branches", "reaper"]
```

Y un dato que conviene saber: **`jsonl-export` y `reaper` se activaron en todas las ciudades por defecto** — antes vivían en un pack de mantenimiento opt-in (*«they ship in the core pack, so they are active in every city by default»*). Si vienes de una versión antigua y creías estar sin ellos, no lo estás.

**Recomendación:** no apagar nada el primer día. Son mecánicas, no gastan tokens, y dos de ellas (`jsonl-export`, `orphan-sweep`) te protegen. **La que hay que vigilar es `prune-branches`**: borra ramas cada seis horas por criterio de «stale», y ese criterio no lo decides tú. Si vas a tener trabajo en ramas `gc/*` sin fundir, apágala o alarga su ciclo. Y con `GC_BEADS=file`, `jsonl-export` y `reaper` se saltan solos con un mensaje.

## 6. La configuración más óptima, y por qué

Ordenado por relación entre lo que cuesta y lo que aporta. Los tres primeros son la parte que de verdad cambia el resultado.

1. **Cada paso termina cuando lo dice un script, no el agente.** Es el mecanismo `[steps.check]`, y es lo único que convierte «el agente dice que acabó» en «está acabado». La formulación oficial: *«`check` is for work you can verify: the step is done **when your script says so, not when the agent says so**»* ([understanding-formulas](https://docs.gascity.com/guides/understanding-formulas)). Con el detalle fino que ya está recogido en [[gas-city-frente-a-la-fabrica]] §3.2: un `check` que sale con **75** (`EX_TEMPFAIL`) significa «no he podido comprobar», **no consume intento**, mientras que cualquier otro distinto de cero sí. Ese tercer estado (`NO_VERIFICABLE`) aquí tiene implementación real, con nombre y convención.
2. **La fórmula y el script de verificación, fuera del alcance de escritura del agente.** Es la recomendación ya cerrada en [[gas-city-frente-a-la-fabrica]] §3.4 y **sigue en pie**: la documentación declara que *«Gas City intentionally runs operator-configured commands. Those commands are a feature, not a sandbox»*, y no he encontrado ninguna afirmación de que el agente no pueda escribir la capa de fórmulas. **No es una recomendación heredada sin comprobar: es un hueco que esta investigación tampoco ha visto cerrado.**
3. **Reparto de modelos por rol, no un modelo para todo — y siempre explícito.** La razón del reparto es económica antes que de calidad: la cuota es el límite real del montaje ([[gas-city-traje-a-medida]] §5.2). Pero el reparto solo funciona si se cumplen **tres reglas a la vez**, porque el entorno de esta máquina apunta a DeepSeek y las omisiones no fallan: **`base_url` en todos los upstreams** (§4.4 — omitirlo manda tu clave a un tercero), **`upstream` declarado en cada agente** (§4.1 — sin él hereda el DeepSeek del tmux) y **`model` pinchado en cada agente** (el harness `claude` no trae modelo por defecto: sin él decide el CLI o tus `ANTHROPIC_DEFAULT_*_MODEL` globales). Las tres, no dos.
   - **Planifica** → Opus (`opus-5`). Es donde un modelo mejor se nota y donde se gasta poco.
   - **Ejecuta** → Sonnet (`sonnet`). El grueso del trabajo, en la banda de coste media.
   - **Mecánico y de volumen** (clasificar, resumir, formatear) → DeepSeek (`deepseek-flash[1m]`). Barato y ya montado en la máquina. **Con una reserva que la comunidad sostiene con número:** un usuario que montó Gas Town con modelos gratuitos y no-Claude (OpenRouter, NVIDIA NIM, Cerebras, Ollama, local) publicó su contador: **4.411 completados de 9.418 intentos — 5.007 intentos (53,2 %) no produjeron nada**, y él mismo dice *«I fixed loads of issues I encountered (>120) to try to work around **hallucinations with the less capable cloud models**»* ([HN 48170083](https://marko-hn-example.netlify.app/story/48170083), apiemotion, ~2026-05). Es decir: **modelo barato sí, pero en pasos cuyo cierre decide un script** — nunca conduciendo un flujo de varios pasos, que es donde esos 5.007 intentos vacíos se convierten en trabajo perdido.
   - **Revisa** → un modelo **distinto del que escribió**, y **la condición de salida de la revisión sigue siendo un script**. La evidencia del wiki es que la revisión por LLM tiene un techo medido de ~50-60 % y que autocorregirse sin oráculo externo empeora el resultado ([[verificacion-externa-agentes]]). Un revisor de otro modelo genera hallazgos; **no** es el que decide que está bien.
   - **Y la regla que envuelve a todas: empareja la herramienta con la decisión.** De un adoptante que lo probó y escribió el balance: *«For one deterministic task a city is overkill»*, y *«match the altitude of the tool to the altitude of the decision»* — la orquestación se justifica cuando el trabajo es plural y lleva criterio; la fontanería *«wants a cron job»* ([idvork.in/gas-city](https://idvork.in/gas-city), Igor Dvorkin — con la advertencia de que el propio autor declara su post *«approximately 70% AI slop»*, así que pesa menos que las otras fuentes).
4. **Empezar con 2-3 agentes y autonomía baja, y subir tarea a tarea.** La fusión la haces tú al principio. Es la recomendación de [[gas-city-traje-a-medida]] §5.1 y no ha cambiado: la organización del proyecto no sostiene la promesa de fiabilidad, y su propio panel público mide la cola de PRs multiplicándose por 3,2 en cuatro meses ([[gas-city-frente-a-la-fabrica]] §3.7).
5. **Sin `gh` no pasa nada.** Si no instalas el CLI de GitHub, *«the core pack's maintenance orders skip GitHub gate checks when the GitHub CLI is not installed»*. Es una pieza menos.

## 7. Cómo no exponerse: aislar el ejecutor — **recomendación, no algo instalado ni decidido**

Todo este apartado es una propuesta a considerar, verificada el 2026-09-29, sobre cómo protegería la máquina real y las credenciales si algún día se pone Gas City en marcha. **Nada de esto está montado ni ejecutado.** Parte de §3.2: como se confirmó ahí, un agente corre en `tmux` directo sobre el sistema operativo, sin contenedor por defecto, y hereda el entorno completo de la máquina.

### 7.1. Alternativas de aislamiento consideradas, con lo que se descarta de cada una

| Opción | Aislamiento | Límite conocido, verificado en vivo | Viable en una máquina sin privilegios de root |
|---|---|---|---|
| **Sandbox nativo de Claude Code** | De proceso, no de kernel | Bypass real de su lista blanca de red, expuesto 5,5 meses, sin CVE propio ([oddguan.com](https://oddguan.com/blog/second-time-same-sandbox-anthropic-claude-code-network-allowlist-bypass-data-exfiltration/)) | Sí |
| **Devcontainers** tal cual | Namespaces estándar de Docker | En la práctica suele correr con el socket de Docker expuesto y `sudo` sin contraseña, salvo que se endurezca a mano ([opencomputer.dev](https://opencomputer.dev/guides/podman-vs-docker-untrusted-code/)) | Sí, con trabajo |
| **Docker/Podman rootless** | Namespaces de usuario | Un escape de namespace compromete la máquina real | Sí |
| **gVisor (`runsc`)** | Intercepta las llamadas al sistema en espacio de usuario, antes de llegar al kernel real | Coste de rendimiento en cargas intensivas en llamadas al sistema | Sí, modo sin root documentado |
| **Kata Containers** | Máquina virtual ligera por contenedor, kernel propio | **Verificado en vivo, el 2026-09-29:** el modo sin root sigue con un issue abierto desde mayo de 2024 — *«Kata claims to support rootless, but I fail to achieve that in many machines»* ([#9591](https://github.com/kata-containers/kata-containers/issues/9591)) | No — pide `/dev/kvm` |
| **Firecracker / E2B** | MicroVM, aislamiento de hardware | Auto-alojarlo es «un proyecto de infraestructura real, no un `helm install`» ([beam.cloud](https://www.beam.cloud/blog/how-to-self-host-code-sandbox)) | No — pide `/dev/kvm` |
| **OpenHands runtime** | Contenedor Docker configurable | Permite cortar la red por completo con una sola variable (`SANDBOX_NETWORK_DISABLED=true`) | Sí |

**Descartadas para este caso, y por qué:** Kata y Firecracker/E2B dan más aislamiento (hardware, no solo kernel) pero piden `/dev/kvm` y virtualización anidada, que una máquina sin privilegios de root no garantiza. Devcontainers tal cual viene mal configurado por defecto. El sandbox nativo de Claude Code queda como capa extra, nunca como única barrera, por el bypass documentado.

**Lo que quedaría mejor situado, si se hiciera:** Podman rootless como base, con gVisor como motor de ejecución — el mismo patrón que usa **Google** en sus propios servicios, confirmado en su documentación oficial de Google Cloud (no en la del propio gVisor, que no lo dice tan explícito): *GKE Sandbox y Cloud Run usan gVisor para aislar cargas no confiables* ([docs.cloud.google.com](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/sandbox-pods)).

### 7.2. El resto de la propuesta, si se hiciera

- **Un proxy de salida con lista blanca de dominios**: solo `api.anthropic.com`, `api.deepseek.com` y GitHub — nada más.
- **Las claves nunca en el entorno del agente, inyectadas solo por el proxy.** Confirmado en vivo en la documentación oficial de Claude Code, cita textual: *«With `mask`, the sandboxed command sees a per-session sentinel value instead of the real one. Each `mask` entry can list `injectHosts`, the hosts the real value is allowed to reach. When a request leaves the sandbox for one of them, the sandbox proxy replaces the sentinel with the real value.»* ([code.claude.com/docs/en/sandboxing](https://code.claude.com/docs/en/sandboxing)). El comando nunca ve la clave real.
- **Autonomía baja al empezar, fusión en manos del usuario** — ya recomendado en §6, punto 4, y esto no lo sustituye, lo refuerza.
- **Apagar lo que ya se identificó en §5**: la telemetría, y vigilar `prune-branches`.

**Dos cifras que se manejaron en la conversación y no se sostuvieron al comprobarlas: retiradas, no incluidas aquí.** Eran un caso citado de `future-architect/vuls` con 52 fallos y una cifra de Spotify sobre sesiones vetadas; ninguna de las dos fuentes consultadas las contiene, así que no figuran como hecho en esta nota.

### 7.3. Enlace con la investigación previa del wiki

Esta comparativa reutiliza y reverifica en vivo (2026-09-29) la que ya existía en [[orquestacion-seguridad-ejecutor]] (investigación del 2026-09-24, mismo caso de una VM Linux sin GPU ni contraseña de root). Las estrellas y la actividad de los repositorios se comprobaron de nuevo por `gh api` en esta fecha, no se copiaron de la nota anterior.

## 8. Dónde se ha buscado

| Fuente | Tipo | Resultado |
|---|---|---|
| `https://docs.gascity.com/llms.txt` y 25 páginas de documentación (instalación, quickstart, configuración de agente, recetas de harness, fórmulas, packs, troubleshooting, FAQ, CLI, topología de beads) | **Artefacto primario del fabricante** | La fuente que decide el esquema: los cinco ejes, el upstream, `GC_BEADS=file`, las órdenes |
| Clon del repositorio `gastownhall/gascity` (2026-09-29, 4.378 ficheros Go) | **Artefacto primario** | Lo que ninguna documentación dice: el enum de modelos abierto, la precedencia del upstream, la semántica del overlay de entorno, el texto de divulgación, las órdenes destructivas |
| Clon de `gastownhall/gascity-packs` (catálogo oficial) | **Artefacto primario** | El pack `contributing` y la ausencia de fórmulas de release |
| [`gastownhall/gastown#3649`](https://github.com/gastownhall/gastown/issues/3649) y sus 9 comentarios | **Artefacto primario** (el issue que originó la pregunta) | El mecanismo exacto y cómo se cerró |
| `deps.env`, `CHANGELOG.md`, esquema `city-schema.txt` | **Artefacto primario** | Versiones fijadas, historial de la telemetría, claves de `[orders]` |
| Hacker News (API de Algolia, 1.135 comentarios con «gascity»; y las búsquedas dirigidas `gascity telemetry`, `gascity PR upstream`, `gascity deepseek`, `gascity local model`) | Comunidad independiente | Las críticas estructurales ya están en [[gas-city-frente-a-la-fabrica]] §3.6. De las búsquedas dirigidas: **`gascity telemetry` → 0 resultados** y **`gascity PR upstream` → 0 resultados**. Es decir, **no hay ningún informe de comunidad sobre telemetría ni sobre contribución automática en Gas City, ni a favor ni en contra** — ver «lo que no encontré» abajo |
| [DoltHub, *A Week In Gas Town*](https://www.dolthub.com/blog/2026-03-24-a-week-in-gas-town/), Tim Sehn, 2026-03-24 | **Comunidad independiente, el informe de uso más sustancial que existe** | Cinco días de uso real, con cifras: *«The whole experience cost me $3,000»*, *«about $100/hour»*, y un resultado: *«A mostly working DoltLite in a week»*. Tres avisos que valen para el montaje: los polecats *«merging directly to master»* (los agentes fundieron sin revisión), *«Using the latest Claude can sometimes result in Claude using tasks or subagents instead of Beads and Polecats»* (los agentes se saltan el método), y «The code is by agents for agents. **Humans, code review at your own risk**». **Es Gas Town, no Gas City**, y con suscripción medida, no con Max |
| [HN 48170083](https://marko-hn-example.netlify.app/story/48170083), apiemotion, ~2026-05 | Comunidad independiente | **La única evidencia de comunidad sobre modelos no-Claude en este tipo de orquestación**, y es la que sostiene la reserva del §6.3: proxy propio para usar modelos gratuitos y locales, contador de **4.411 completados de 9.418 intentos**, y *«>120 issues (…) to work around hallucinations with the less capable cloud models»*. Tuvo que **sustituir Claude Code y tmux** y abandonar el flujo autónomo por una máquina de estados. **Es Gas Town, no Gas City** |
| [idvork.in/gas-city](https://idvork.in/gas-city), Igor Dvorkin | Comunidad independiente, **peso reducido** | Adoptante, no afiliado, con balance y errores concretos: *«My first real workflow cost about $9 and 32 Opus turns to produce a thirteen-line change»*, polecats que encienden y *«sat idle (…) until I nudged each one by hand»*, y un controlador que *«believed it had converged while a stalled worker sat where it couldn't see»*. **El propio autor declara el post *«approximately 70% AI slop»* y no lleva fecha** — de ahí el peso reducido, aunque las cifras que da son concretas y las tres anteriores no las contradicen |
| **Documentación del fabricante como prueba de bondad** | — | **Descartada como evidencia.** Toda la mecánica de esta nota sale de su documentación y su código, que sirven para explicar **cómo funciona** — no para sostener que sea buena idea. Para eso hace falta comunidad, y sobre este uso concreto hay poco |

**Lo que busqué y NO encontré** — un hueco declarado vale, rellenarlo no:

| Buscado | Resultado | Dónde |
|---|---|---|
| Informes de telemetría o «phone home» en Gas City | **Ninguno.** 0 resultados con `gascity telemetry`; los 55 de `gas city phone home` son de otros temas (drones, televisiones, tuberías de gas), ninguno del proyecto | HN (Algolia) |
| Informes de contribución automática al repo del fabricante **en Gas City** | **Ninguno.** 0 resultados con `gascity PR upstream`. Lo único cercano es el debate del #3649, que es **Gas Town** | HN (Algolia), tracker |
| Gas City con DeepSeek concreto | **Nada.** `gascity deepseek` da 2 resultados, ambos de 2025 y sobre el modelo DeepSeek, no sobre Gas City | HN (Algolia) |
| Uso real de Gas City en producción con métricas | **Solo el del modelo local** ([[gas-city-frente-a-la-fabrica]] §7): un Mac Studio que *«runs one of my companies»* con GLM-5.3 en 8 bits — **es un modelo no-Claude corriendo dentro de una Gas City**, y es el tercer dato de comunidad sobre modelos alternativos | [HN 49532605](https://news.ycombinator.com/item?id=49532605), taylorhou, 2026-09-02 |

**Cobertura, dicha sin adornos:** de las fuentes que existen, miré las dos primarias completas (documentación y código) y el catálogo de packs entero, más tres fuentes de comunidad independientes de naturaleza distinta (un informe de uso con cifras, un hilo con contador propio, y un blog de adoptante). **No miré**: el Discord de gastownhall.ai (requiere cuenta), el código de los packs de terceros del registro, ni el historial de git de cada fichero (leí el estado actual, no cuándo cambió). Lo que queda sin mirar está donde estaría la prueba de si un pack ajeno puede redirigir credenciales — el riesgo anotado en §4.4 — y donde estaría un informe de Gas City (no Gas Town) en producción, que **sigue sin existir**.

## 9. Verificación y límites

- **Qué se comprobó mecánicamente:** todas las citas de esta nota se localizaron con búsqueda literal sobre los ficheros descargados en `/tmp/gcdocs` y `/tmp/gascity-src` el 2026-09-29, no sobre un resumen. Las versiones (`DOLT_VERSION=2.1.7`, `BD_VERSION=v1.3.0`) y las claves de configuración salen de los ficheros del repositorio, no de la documentación que las describe.
- **Dos errores míos, encontrados y corregidos dentro de la propia investigación.** Quedan escritos en vez de borrados, porque los dos son ilustrativos.
  1. **La telemetría.** Di el frente por cerrado tras verificar `internal/telemetry` («strictly opt-in»). Era **incompleto**: existe un segundo subsistema, `internal/productmetrics`, que **viene activado por defecto**. *«Estrictamente opt-in»* era verdad de **una** parte y falso del conjunto.
  2. **El mecanismo por el que el ambiente llega al agente.** Afirmé en una primera versión que Gas City *reenvía* el entorno ambiente, y que por eso un agente sin `upstream` cae en DeepSeek. **El motivo era falso:** la lista de reenvío es corta y **no incluye `ANTHROPIC_BASE_URL`**. La conclusión operativa se sostiene —el ambiente llega por **herencia del servidor tmux**, no por reenvío— pero el motivo importa, porque cambia dónde está el riesgo: no en una lista de Gas City, sino en **cuándo arrancó el tmux**. Corregido en §4.1.
  3. **Y el error que más importa: el ejemplo de §4.4 filtraba credenciales.** Al verificar el mecanismo de herencia apareció que el render del upstream **salta los campos vacíos**, que el overlay **no sustituye** el entorno heredado, y que `ScrubTokenEnv` solo vacía el token del controlador. Consecuencia: un upstream de Anthropic declarado **sin `base_url`** habría mandado la clave de Anthropic a DeepSeek, en silencio y sin error. **Era lo que yo había escrito.** Corregido en §4.4 — ahora `base_url` es obligatorio en todos los upstreams, con escotilla cruda de refuerzo y verificación por `gc config explain --agent`. De los tres errores, este es el único que habría causado daño real, y salió de comprobar los otros dos.
- **Hecho, supuesto y juicio.** Hechos con URL o ruta de fichero: todo §3, §4 y §5. **Supuesto sin fuente:** que un pack importado pueda redirigir credenciales de un agente — la documentación lo llama *«TRUSTED configuration»* y avisa de importar solo packs de confianza, así que la dirección es esa, pero **no lo he probado**. **Juicio mío, etiquetado:** el reparto de modelos de §6.3 y la recomendación de apagar la telemetría de §5.2.
- **El mejor caso contra lo de §5.1.** Si el pack `contributing` se importara alguna vez por descuido, o si un pack de terceros declarase un upstream hacia un endpoint ajeno, el resultado sería el mismo problema del #3649 por otra puerta. **Lo que lo zanjaría: leer los upstreams y las fórmulas de cualquier pack antes de importarlo.** De ahí la advertencia de §4.4.
- **Lo que no puedo afirmar:** no he ejecutado Gas City. Todo lo de esta nota es lectura de su documentación y su código, no una instalación probada. **La configuración de §4.4 no está verificada en ejecución** — está verificada contra el código que la implementa. Quien la ponga en marcha debe confirmar con `gc config explain --agent <nombre>` (que muestra el entorno resuelto) y `gc doctor`.
- **«No existe» donde no puedo probarlo:** sobre la contribución automática digo «no he encontrado ningún mecanismo, en el repositorio ni en el catálogo», no «es imposible». Un pack de terceros fuera del catálogo oficial no lo he mirado.

## Enlaces

- [[cuentas-claude]] — qué cuenta de Claude usa el mayor: `[providers.claude] command` apunta a `~/bin/claude`, que lee la cuenta cargada con `cc`

- [[_index]] — índice de esta carpeta
- [[gas-city-traje-a-medida]] — por qué Gas City es la pieza central del montaje, y qué tocar para ajustarlo
- [[gas-city-con-2cerebro]] — cómo se usa con 2cerebro para crear y desarrollar aplicaciones
- [[gas-city-frente-a-la-fabrica]] — qué mecanismos sirven y qué es marketing: el bucle `check`, las puertas, los presupuestos, el tercer estado con la convención `75`, y las métricas que el proyecto publica de sí mismo
- [[las-piezas]] — Gas Town y Beads, con el aviso original de la contribución automática que esta nota resuelve
- [[verificacion-externa-agentes]] — por qué la condición de salida tiene que ser un script
- [[orquestacion-modelos-y-costes]] — el detalle de modelos DeepSeek, precios y benchmarks que sostiene este reparto
- [[orquestacion-seguridad-ejecutor]] — la investigación original de aislamiento (2026-09-24) que §7 reutiliza y reverifica
- [[linear-y-jev-frente-a-gas-city]] — si Linear + JEV mejoran este montaje: no lo sustituyen, y dónde encaja JEV como capa de decisión

- [[gas-city-operacion-real]] — el reparto de modelos ya aplicado y probado en vivo, con el fallo real de arranque encontrado y su arreglo
