---
title: Gas City con 2cerebro — cómo usarlo para crear y desarrollar aplicaciones
created: 2026-09-29
updated: 2026-09-29
tags: [gas-city, 2cerebro, uso, flujo, formulas, apps]
zona: tecnico
---

Cómo se usa Gas City junto con 2cerebro para crear y desarrollar aplicaciones: qué papel juega cada uno, el flujo paso a paso con un ejemplo real, y qué hay que tocar para que convivan sin romper las reglas del wiki.

## 1. Qué se pregunta y para qué

Cómo pasar de «tengo Gas City instalado» a «estoy construyendo aplicaciones con él y con el método que ya vive en 2cerebro», sin duplicar conocimiento ni publicar lo que no debe publicarse. Todo lo de mecánica viene verificado en [[gas-city-instalacion-y-modelos]]; aquí lo que se responde es **el reparto**, que es la decisión que de verdad cuesta.

## 2. El reparto: quién es quién

Gas City y 2cerebro **no compiten ni se solapan**. Resuelven dos cosas distintas, y confundirlas es el error que hace que el montaje no arranque:

- **2cerebro es el método escrito.** Qué se hace, con qué reglas, con qué criterio de calidad, qué está decidido y por qué. Es texto, y ya existe: `AGENTS.md`, `areas/decisiones.md`, [[verificacion-externa-agentes]], las skills de `~/.claude/skills/`.
- **Gas City es el que ejecuta ese método por ti.** Coge el método, lo convierte en un grafo de trabajo y lo reparte entre varios agentes, fuera de tu sesión.

La traducción concreta, capacidad por capacidad, está en la página *Coming from Coding Agents* de Gas City, y merece la pena tenerla a mano porque casi todo lo que ya sabes de Claude Code tiene su equivalente. **Y una confirmación que importa para el wiki: `AGENTS.md` y `CLAUDE.md` no desaparecen dentro de Gas City** — un adoptante lo resume así: la ciudad dice *sobre qué* se trabaja, el fichero dice *cómo* ([idvork.in/gas-city](https://idvork.in/gas-city), con la salvedad de que ese texto es de peso reducido). Es decir, las reglas que ya rigen aquí siguen rigiendo dentro de cada agente; lo que cambia es quién reparte el trabajo.

| Lo que ya tienes en 2cerebro | Su equivalente en Gas City |
|---|---|
| Un agente de Claude Code | Una **carpeta de rol** (`agents/<nombre>/`) con su `agent.toml` y su prompt |
| Una skill en `~/.claude/skills/` | La **misma estructura**, materializada por ámbitos: pack (toda la ciudad) o rol |
| Las reglas de `AGENTS.md` | Los **prompts de rol** y los `template-fragments/` |
| Un plan en markdown | Una **fórmula**: el método escrito una vez, ejecutado como grafo |
| «Ejecuta esto cada vez que X» | Una **orden**: dispara una fórmula por temporizador o por evento |
| Un repo con su `CLAUDE.md` | Un **rig**: el proyecto registrado en la ciudad |

## 3. La decisión de encaje: 2cerebro no es la ciudad, es de donde sale el método

Esta es la parte que hay que decidir bien, porque equivocarse aquí lo contamina todo.

**2cerebro no debe ser el rig que se trabaja.** Es el sitio donde vive el criterio: el wiki, las decisiones, las reglas, las investigaciones. Registrarlo como proyecto a desarrollar metería a los agentes a editar sus propias reglas mientras trabajan — exactamente el problema que el wiki ya identificó al hablar de quién escribe la capa de verificación.

**Cada aplicación es un rig.** Un rig es, en palabras de Gas City, *«an external project directory registered with the city»*, y recibe *«its own beads database, hook installation, and routing context»*. Es decir: **un repo por aplicación, con su propio almacén de tareas**, y todos bajo un mismo método.

Y el método de 2cerebro entra como **pack**: la ciudad importa un pack con los roles, los prompts y las fórmulas que codifican cómo se trabaja aquí. Así el criterio se escribe una vez y se aplica a todas las aplicaciones.

Lo que queda en cada sitio:

```
~/mi-ciudad/                 ← la ciudad: aquí vive el método
  city.toml                    → qué rigs hay, qué packs se importan, qué upstream usa cada agente
  pack.toml                    → la identidad del método
  agents/planificador/         → prompt + agent.toml (modelo, upstream, pool)
  agents/obrero/
  formulas/                    → el método escrito: cómo se planifica, se ejecuta y se verifica
  skills/                      → las skills de 2cerebro que quieras compartir con todos los agentes
  orders/                      → cuándo se dispara cada cosa

~/mi-app/                    ← el rig: la aplicación
  .beads/                      → su almacén de tareas (gitignored automáticamente)
```

## 4. El flujo, paso a paso, con un ejemplo concreto

Ejemplo ancla: una aplicación web con tests en `npm test`. Los comandos son reales; sustitúyelo por lo que toque en cada caso.

**Paso 1 — Arrancar la ciudad.**

```bash
export GC_BEADS=file           # para probar sin dolt/bd; ver gas-city-instalacion-y-modelos §3
gc init ~/mi-ciudad
cd ~/mi-ciudad
```

**Paso 2 — Definir los dos upstreams y los agentes.** Los bloques exactos (Anthropic de verdad y DeepSeek, con la precedencia bien puesta) están en [[gas-city-instalacion-y-modelos]] §4.4, y no se repiten aquí. Lo que importa en este documento: **cada agente declara su `upstream` explícitamente**, porque el entorno de esta máquina apunta a DeepSeek y sin declararlo todo cae ahí en silencio.

**Paso 3 — Escribir el método como fórmula.** Aquí está el valor real, y no es automatizar comandos: es que **el criterio de terminado deja de ser la opinión del agente**. La pieza es `[steps.check]`, y su forma es esta:

```toml
[[steps]]
id = "implementar"

[steps.check]
script = "npm test"        # el paso cierra cuando ESTO sale 0, no cuando el agente dice que acabó
max_attempts = 3
on_exhausted = "hard_fail"
```

Con la regla del tercer estado, documentada con su fuente en [[gas-city-frente-a-la-fabrica]] §3.2: si tu script sale con **75** (`EX_TEMPFAIL`), significa «no he podido comprobar» y **no consume intento**; cualquier otro código distinto de cero es «aún no» y sí lo consume.

**Paso 4 — Registrar la aplicación como rig.**

```bash
mkdir -p ~/mi-app && cd ~/mi-app && git init && cd -
gc rig add ~/mi-app
```

`gc rig add` escribe las entradas necesarias en el `.gitignore` del proyecto por su cuenta (`cmd/gc/gitignore.go`: `.beads/*` y `!.beads/identity.toml`) — **pero conviene comprobarlo con `git status`**, por lo que se explica en §5.

**Paso 5 — Lanzar trabajo.** La forma mínima, del quickstart oficial:

```bash
gc sling obrero "Implementa el endpoint de facturas siguiendo la especificación del bead"
```

Y para el método completo — no una tarea suelta:

```bash
gc sling obrero <bead-id> --on mol-polecat-work
```

**Paso 6 — Ver qué está pasando.** Los tres sitios donde se mira: el trabajo (`gc bd show <bead-id> --watch`), la conversación de un agente (`gc session logs <agente> -f`), y el registro de la ciudad (`gc events`). Sin esto, Gas City es una caja opaca que gasta cuota.

**Paso 7 — No fundir nada tú hasta tener base de confianza.** Es la recomendación de [[gas-city-traje-a-medida]] §5.1 y sigue en pie.

## 5. Lo que hay que tocar para que conviva con 2cerebro

Aquí están los puntos donde este montaje choca con lo que el wiki ya tiene montado. Son los que evitan que el cron publique lo que no debe.

**5.1. El repo es público y el cron publica cada hora.** Es el riesgo concreto y no es teórico: `cerebro-sync.sh` hace `git add -A` y push. Gas City crea artefactos por su cuenta, y no todos quedan fuera de git solo.

Lo que está cubierto y lo que no, comprobado en `cmd/gc/gitignore.go`:

| Dónde | Qué añade Gas City al `.gitignore` |
|---|---|
| Directorio de **ciudad** (`gc init`) | `.gc/`, `.beads/*`, `!.beads/identity.toml`, `hooks/` |
| Directorio de **rig** (`gc rig add`) | `.beads/*`, `!.beads/identity.toml` — **solo eso** |

**Consecuencia a tener presente:** en un **rig no se ignora `.gc/`**. Si el flujo acaba creando trabajo o estado de Gas City dentro del directorio del proyecto, **eso se publica**. La comprobación barata antes de dejarlo correr: `git status --short` en el rig después del primer trabajo, y `git check-ignore` sobre cualquier ruta nueva — que es exactamente la regla que ya exige `AGENTS.md`.

**5.2. Los hooks.** Gas City **no pisa** el `.claude/settings.json` del repo: la precedencia real es que ese fichero es la **fuente de mayor prioridad, que se lee y se fusiona** sobre los valores por defecto, y el resultado se escribe en `.gc/settings.json` (`internal/hooks/hooks.go`). Eso significa que los hooks que ya viven en este wiki — `verificar-respuesta.sh` en `Stop`, `permisos-hacia-fuera.sh` en `PermissionRequest` — **se conservan**. Y significa también que **se fusionan con los de Gas City**: una sesión de agente arrancada por `gc` llevará ambas cosas.

Dos ajustes que Gas City sí trae por defecto en su plantilla de sesiones gestionadas (`internal/hooks/config/claude.json`) y que merecen una decisión explícita por tu parte, no heredarse en silencio:

- `"enableAllProjectMcpServers": true` — confía en los servidores MCP que declare el proyecto. En un repo que recibe cambios de agentes, eso es superficie de ataque.
- `"skipDangerousModePermissionPrompt": true` — se salta la confirmación antes del modo sin permisos.

**Recomendación:** dejarlos como están mientras se prueba, y **revisarlos antes de usarlo con algo real** — el criterio de 2cerebro no es «lo que traiga por defecto», es «lo que está decidido». Y la instalación de hooks en un rig es además configurable por agente (`install_agent_hooks`, que acepta nombres de proveedor como `"claude"`), así que se puede acotar.

**5.3. Las skills se comparten por ámbito, y sin lista de permitidos.** Gas City materializa una skill escrita una vez a **todos** los agentes de su ámbito, enlazándola en el directorio propio de cada proveedor (`.claude/skills/` para Claude Code):

> *«No framework around skills: no per-agent allow-lists. Within a scope every eligible agent gets every skill; the model decides when one applies.»*

Es una diferencia real con Claude Code, donde sí puedes acotar. Traducción para el wiki: **`skills/` a nivel de pack llega a todos los agentes de la ciudad.** Si una skill no debe estar al alcance de un rol —por ejemplo, algo que toque secretos o que publique hacia fuera—, **no la pongas en el pack**; ponla en `agents/<rol>/skills/`, que solo llega a ese rol.

**5.4. Los packs ajenos son configuración de confianza.** Un pack importado puede declarar sus propios `[upstreams]` y **pisar los de la ciudad por nombre**, redirigiendo dónde van las credenciales. La documentación lo dice explícito: *«only import packs you trust»*. Con un montaje que usa tu clave de Anthropic y tu cuenta de GitHub, esto no es un aviso de manual: **leer los upstreams de un pack antes de importarlo**, o no importarlo.

## 6. Qué no va a resolver

Sin adornos, porque es donde este montaje se rompe si se le pide lo que no da.

- **El agente no se verifica a sí mismo.** La evidencia del wiki está medida: la revisión por LLM tiene un techo de ~50-60 % y autocorregirse sin oráculo externo **empeora** el resultado ([[verificacion-externa-agentes]]). De ahí que todo el §4 gire en torno a `[steps.check]`: un revisor de otro modelo **genera hallazgos**, pero el que declara terminado es un script.
- **La plataforma declara que no es un recinto.** Palabras suyas: *«Gas City intentionally runs operator-configured commands. Those commands are a feature, not a sandbox.»* La separación es **por configuración**, no forzada. Si un agente tiene permiso de escritura sobre la fórmula que lo evalúa, puede redefinir cómo se le evalúa. La contramedida ya está decidida en [[gas-city-frente-a-la-fabrica]] §3.4: **la fórmula y el script de verificación, fuera de su alcance de escritura**.
- **La cuota es el límite real, no el número de agentes.** Dos o tres al empezar. La organización del proyecto gasta *«thousands of dollars a month»* en API según [[gas-city-traje-a-medida]] §5.2. Y aquí hay cifras de comunidad que conviene tener delante: el informe de uso más sustancial que existe, cinco días con Gas Town, costó *«$3,000»* a *«about $100/hour»* ([DoltHub](https://www.dolthub.com/blog/2026-03-24-a-week-in-gas-town/), Tim Sehn, 2026-03-24); y un adoptante midió *«$9 and 32 Opus turns to produce a thirteen-line change»*, que él mismo llama absurdo frente a un script ([idvork.in/gas-city](https://idvork.in/gas-city)). **No es un argumento contra la herramienta: es el argumento contra usarla en tareas que un script resuelve**, que es la misma regla de §5.3 aplicada al coste.
- **Los agentes se saltan el método.** Documentado por el mismo informe: *«Using the latest Claude can sometimes result in Claude using tasks or subagents instead of Beads and Polecats»*, y los polecats acabaron *«merging directly to master»* — fusionando sin revisión, en contra de lo previsto ([DoltHub](https://www.dolthub.com/blog/2026-03-24-a-week-in-gas-town/)). Traducción: las fórmulas se comportan como instrucciones, no como barreras, y el agente puede rodearlas. Refuerza §6.1 y §6.2: **el cierre lo tiene que decidir un script, porque el agente se salta el procedimiento con la misma facilidad con la que lo sigue.**
- **No hay ningún informe de Gas City (no Gas Town) en producción.** El proyecto tiene el 7,2 % de las estrellas de su predecesor y su propio panel público mide la cola de PRs multiplicándose por 3,2 en cuatro meses con un solo revisor ([[gas-city-frente-a-la-fabrica]] §3.5 y §3.7). Lo que sí hay ahora es comunidad sobre **Gas Town**, con cifras y con avisos concretos (los dos puntos anteriores), y un adoptante escribiendo sobre Gas City — con la salvedad de que ese último declara su propio texto *«approximately 70% AI slop»*. No es un motivo para no usarlo —es el montaje que mejor encaja con tu objetivo— pero sí para **no delegar en él la decisión de que algo está bien**.

## 7. Dónde se ha buscado

| Fuente | Tipo | Resultado |
|---|---|---|
| `docs.gascity.com`: *Coming from Coding Agents*, *Set Up a Multi-Agent Engineering Environment*, *Understanding Formulas*, *Understanding Packs*, quickstart, tutoriales 01/02/05/06/07 | **Artefacto primario del fabricante** | El mapeo de capacidades y el reparto en tres capas (pack / ciudad / `.gc/`) |
| Clon del repositorio: `internal/hooks/hooks.go` y `config/claude.json`, `cmd/gc/gitignore.go`, `internal/config/config.go` | **Artefacto primario** | La precedencia real de los hooks, los valores exactos de `.gitignore`, y `install_agent_hooks` |
| Los ficheros de `cmd/gc/` que provisionan un rig | **Artefacto primario** | Qué escribe `gc rig add` y qué no |
| El wiki: [[gas-city-frente-a-la-fabrica]], [[gas-city-traje-a-medida]], [[verificacion-externa-agentes]], [[las-piezas]] | Investigación previa propia | Los límites de §6 y el criterio de verificación de §4 |
| [DoltHub, *A Week In Gas Town*](https://www.dolthub.com/blog/2026-03-24-a-week-in-gas-town/), 2026-03-24 | **Comunidad independiente** | Uso real con cifras, y los dos avisos de §6: los agentes saltándose el método y polecats fusionando a `master`. **Es Gas Town, no Gas City** |
| [HN 48170083](https://marko-hn-example.netlify.app/story/48170083), apiemotion, ~2026-05 | Comunidad independiente | Modelos no-Claude en esta orquestación: **4.411 completados de 9.418 intentos**, y >120 arreglos para sortear alucinaciones |
| [idvork.in/gas-city](https://idvork.in/gas-city) | Comunidad independiente, **peso reducido** (el autor declara *«approximately 70% AI slop»*) | El coste por tarea trivial, y la regla de §6.3 |
| **La documentación del fabricante como prueba de que el montaje es buena idea** | — | **Descartada.** Explica cómo funciona; no sostiene que funcione bien. Para eso sirve la comunidad, y sobre **Gas City en producción no existe nada**: lo que hay es de Gas Town o de un adoptante |

**Cobertura:** miré la documentación de uso entera, el código que decide los cuatro puntos de integración de §5 (hooks, gitignore, skills, packs), y tres fuentes de comunidad independientes (§7). **No miré**: cómo se comporta un rig de tipo wiki bajo carga real, ni los packs de terceros del registro. Y **busqué sin encontrar** un informe de Gas City en producción: sigue sin haber ninguno. Lo que queda sin mirar está donde aparecería el problema de §5.4.

## 8. Verificación y límites

- **Qué se comprobó mecánicamente:** los cuatro puntos de §5 salen de leer el código que los implementa, no de la documentación que los describe — la precedencia de hooks (`installClaude`), los valores literales de `cityGitignoreEntries` y `rigGitignoreEntries`, y el campo `install_agent_hooks`. El mapeo de §2 y las tres capas de §3 salen de las páginas citadas.
- **Hecho, supuesto y juicio.** Hechos con ruta de fichero: §5 entero y el mecanismo de §4.3. **Juicio mío, etiquetado:** el reparto de §3 (2cerebro como método, cada app como rig) — es una recomendación de diseño, no algo que la documentación diga; y la recomendación de revisar los dos ajustes de la plantilla de hooks en §5.2.
- **Sin verificar en ejecución:** no he instalado ni ejecutado Gas City. Todo lo de este documento es lectura de su código y su documentación. **El flujo de §4 no está probado en marcha.** Quien lo ponga en marcha debe confirmar con `gc doctor`, `gc config explain --agent <nombre>` (que muestra el entorno resuelto, incluido qué upstream y qué modelo van a usarse de verdad) y `git status --short` tras el primer trabajo.
- **El mejor caso contra §3.** Si el wiki acabara siendo también un rig —por ejemplo, para que los agentes mantengan el propio conocimiento mientras trabajan— el montaje tendría menos piezas y menos saltos. **Lo que lo decide:** si consigues que la capa de verificación quede fuera de su alcance de escritura, el riesgo que motiva la cautela desaparece; si no, el riesgo es real. No he encontrado ninguna afirmación de que Gas City lo garantice.

## Enlaces

- [[_index]] — índice de esta carpeta
- [[gas-city-instalacion-y-modelos]] — instalación, los cinco ejes, el reparto Opus/Sonnet/DeepSeek y los tres frentes a apagar
- [[gas-city-traje-a-medida]] — por qué Gas City es la pieza central del montaje
- [[gas-city-frente-a-la-fabrica]] — los límites con evidencia: issues del registro de auditoría y métricas del propio proyecto
- [[verificacion-externa-agentes]] — por qué la condición de salida tiene que ser un script y no un agente
- [[gas-city-frente-a-la-fabrica]] §3.2 — los tres estados y el `75`, con su cita textual de la especificación
- [[gas-city-alcalde]] — el camino conversacional con `gc.mayor`; y el aviso de que `mol-polecat-work` (pack `gastown`) acaba en el refinery, que fusiona solo en `main` por defecto (`merge_strategy=direct`)
