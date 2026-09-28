---
title: Lego — TODO lo necesario, etapa por etapa
created: 2026-09-28
updated: 2026-09-28
tags: [lego, receta, completo, etapa, modelos]
zona: tecnico
---

**El documento de montaje completo.** Todo lo que hace falta para cada etapa: el software, las cuentas, la configuración, los ficheros con su contenido, el modelo que ejecuta cada cosa, los comandos exactos, y qué revisar. Tu reparto es **Claude Code + Opus para pensar y DeepSeek para implementar**, y **conmutable** — el diseño permite mover cualquier etapa a cualquiera de los dos modelos sin tocar el flujo.

Lo verificado va con enlace. Lo que es decisión de diseño se marca como tal.

---

# PARTE 0 · Qué se monta

Cuatro capas, y cada una hace una cosa:

| Capa | Qué es | Pieza |
|---|---|---|
| **El router** | Un punto único por el que pasan todas las peticiones. Decide a qué modelo va cada una | [`claude-code-router`](https://github.com/musistudio/claude-code-router) |
| **Los agentes** | Quien ejecuta. Uno solo basta: Claude Code, apuntado al router | Claude Code |
| **El bucle** | Lanza el agente una y otra vez sobre la cola de tareas | [`fstandhartinger/ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum) |
| **El contrato** | Las reglas, los permisos y las tareas | Ficheros `.md` en tu repositorio |

**La clave del diseño:** Claude Code sólo habla con Anthropic de fábrica. **El router le permite hablar con DeepSeek —y con cualquier otro— sin que Claude Code se entere.** Eso es lo que hace el reparto conmutable.

---

# PARTE 1 · INVENTARIO — todo lo que hace falta

## 1.1 Software

| Pieza | Para qué | ¿En tu máquina ya? |
|---|---|---|
| **Claude Code** | El agente | ✅ **Sí, v2.1.283** |
| **Node.js 22 o superior** | Lo pide el router y Claude Code | ✅ **Sí, v22.23.2** |
| **git** | La cola, el historial, las ramas | ✅ **Sí, 2.43** |
| **jq** | Leer el estado desde el bucle | ✅ **Sí, 1.7** |
| **curl** | Descargar los scripts | ✅ **Sí, 8.5** |
| **`claude-code-router`** | **La pieza que hace conmutable el modelo** | ❌ **Hay que instalarlo** |
| **`docker`** | El entorno aislado | ❌ **Hay que instalarlo** |
| **`pdftoppm`** | Sólo si quieres PDFs de los informes | ✅ Sí |

**Faltan dos instalaciones.** Todo lo demás ya está.

## 1.2 Cuentas y claves

| Cuenta | Para qué | Coste real |
|---|---|---|
| **Claude Pro** | Acceso a Claude Code y a Opus | **~20 €/mes** |
| **DeepSeek, clave de API** | El modelo que implementa | Pago por uso — del orden de céntimos por millón de tokens de entrada |

**Aquí hay un detalle que hay que decir claro:** la clave de API de DeepSeek es **pago por consumo**, aparte de la suscripción de Claude. No es «gratis porque ya tengo Claude». Se pone un tope desde el primer día.

## 1.3 Hardware

| | Valor |
|---|---|
| Núcleos | 4 |
| RAM | 7,7 GB (3,6 libres) |
| GPU | **Ninguna** |

**Consecuencia directa: en esta máquina no se puede ejecutar DeepSeek en local.** Los modelos locales útiles para agentes exigen **≥24 GB de VRAM o ≥64 GB de memoria unificada**. Aquí no hay ni lo uno ni lo otro, así que **DeepSeek va por API**, no en local. Eso está bien: sale más barato que montar la máquina y funciona desde ya.

## 1.4 El repositorio

Un repositorio git. **Uno tuyo, o uno de juguete de veinte líneas para probar.** Lo que necesita por dentro:

- **Tests que pasen.** Sin esto no hay oráculo, y sin oráculo no hay veredicto. **Es la pieza que decide si el montaje sirve.**
- **Un fichero de composición** (`docker-compose.yml`) sólo si el proyecto lo necesita.
- **`.gitignore`** que excluya `.env*`, `logs/`, `history/`, `node_modules/`.

---

# PARTE 2 · INSTALACIÓN, comando a comando

## Paso 2.1 — El router

```bash
npm install -g @musistudio/claude-code-router
ccr ui
```

Abre `http://127.0.0.1:3458` (la interfaz) y levanta la pasarela en **`http://127.0.0.1:3456`**. *(Verificado en el README del proyecto. Requiere Node 22+, que ya tienes.)*

**Y se configura por interfaz, no por fichero:**

1. **Providers → Add Provider** → eliges **DeepSeek** de la lista de preconfigurados, pegas la clave, y eliges el modelo.
2. Repite para **Anthropic** con tu suscripción.
3. **Server → Start**.
4. **Agent Config → Claude Code** → y aquí es donde haces el reparto.

## Paso 2.2 — Conectar Claude Code al router

Claude Code se apunta al router con una variable de entorno:

```bash
export ANTHROPIC_BASE_URL=http://127.0.0.1:3456
```

*(El mecanismo está verificado: es la variable que usa Claude Code para hablar con cualquier endpoint compatible, y es la misma que se usa para apuntarlo a un modelo local.)*

## Paso 2.3 — El bucle y los ficheros

```bash
mkdir -p .specify/memory specs scripts scripts/lib logs history .claude/commands

base=https://raw.githubusercontent.com/fstandhartinger/ralph-wiggum/main
for a in "" -codex -gemini -copilot; do
  curl -o "scripts/ralph-loop$a.sh" "$base/scripts/ralph-loop$a.sh"; done
for f in spec_queue circuit_breaker nr_of_tries response_analyzer date_utils notifications; do
  curl -o "scripts/lib/$f.sh" "$base/scripts/lib/$f.sh"; done
chmod +x scripts/ralph-loop*.sh
```

*(Los 18 ficheros del repositorio están descargados y leídos. La lista es la real, no una invención.)*

## Paso 2.4 — El entorno aislado

```bash
sudo apt install docker.io
```

**Esto no es opcional.** Es lo único de todo el montaje que puede hacerte daño de verdad: **Claude Code fue explotado para filtrar secretos de un `.env` por consultas DNS** (con CVE y parche), y hay más casos. Mientras no esté, **el bucle corre en una máquina o un usuario sin acceso a tus cosas.**

---

# PARTE 3 · LOS MODELOS, y cómo se conmutan

## 3.1 El reparto que pides

| Tipo de trabajo | Modelo | Por qué |
|---|---|---|
| **Escribir la tarea y decidir** | **Opus** | Es donde una hora ahorra cinco, y el criterio pesa más que la velocidad |
| **Implementar tareas acotadas** | **DeepSeek** | Si el criterio es comprobable por comando, **el veredicto lo da el runner, no el modelo** |
| **Implementar tareas ambiguas o grandes** | **Opus** | Ahí el barato se pierde, y lo pagas en revisiones |
| **Diseño** (si lo quieres probar) | **Conmutable** | Lo pides como dinámico: se cambia sin tocar el flujo |

## 3.2 Cómo se hace dinámico — dos niveles

**Nivel 1: en el router.** CCR permite definir **perfiles de agente** y **reglas de enrutado por condición**. Puedes montar, por ejemplo:
- Un perfil `pensar` → Opus
- Un perfil `implementar` → DeepSeek
- Y reglas que decidan por tamaño de contexto, por coste o por patrón en la petición.

**Nivel 2: en el fichero de tarea.** Aquí está la parte que te interesa, y es **diseño mío, marcado como tal**: un campo más en la tarea.

```markdown
# 002 · Login

**Owner:** agente
**Modelo:** implementar        ← o «pensar», o «deepseek», o «opus»
**Depende de:** 001
```

**Y el bucle lo respeta.** En el script, una línea antes de lanzar el agente:

```bash
# Lee el campo Modelo de la tarea y lo pasa al agente
modelo=$(grep -m1 '^\*\*Modelo:\*\*' "$tarea" | sed 's/.*: *//' | tr -d ' ')
case "$modelo" in
  pensar)     export ANTHROPIC_MODEL="router,opus" ;;
  implementar) export ANTHROPIC_MODEL="router,deepseek" ;;
  *)          export ANTHROPIC_MODEL="router,opus" ;;   # por defecto, el bueno
esac
```

**Así el reparto vive en la tarea, no en el script.** Cambiar de modelo para una tarea concreta es editar una línea del `.md`. Y si un día quieres que el diseño también lo haga DeepSeek, **cambias el campo y ya está** — no hay que tocar nada más.

## 3.3 Lo que el dato dice sobre mezclar

**Esto no es neutro, y hay que decirlo.** Del **leaderboard oficial de SWE-bench Pro** (731 tareas, leído en vivo):

| Modelo | Resolución |
|---|---|
| claude-opus-4-6 (thinking) | **51,90%** |
| claude-4.5-Sonnet | 43,60% |
| **deepseek-v3p2** | **15,56%** |

**Menos de un tercio.** Para tareas acotadas con criterio comprobable la diferencia se compensa —el runner te dice si falló—, pero **en trabajo ambiguo el hueco se paga en revisiones**.

**Y el límite que hace que esto funcione:** una tarea con criterio que se comprueba por comando **hace que el modelo importe menos**, porque el veredicto no lo da él. Una tarea sin ese criterio **hace que el modelo barato sea peligroso**, porque te da algo que parece bien y no lo está. **La conmutabilidad sirve si las tareas están bien escritas.**

---

# PARTE 4 · LAS ETAPAS, una por una, con todo

## ETAPA 1 · Escribir la tarea

**Modelo:** Opus. **Aquí está la hora mejor pagada de todo el proceso.**

**Software:** nada. Un editor.

**Los ficheros que produce:**

```
specs/001-registro.md
specs/002-login.md
specs/003-bitvavo.md
```

**El contenido, con la forma real** (campos tomados de [`quotabar/specs/GH55/tasks.md`](https://github.com/majiayu000/quotabar/blob/main/specs/GH55/tasks.md), adaptados):

```markdown
# 002 · Login

**Owner:** agente
**Modelo:** implementar
**Depende de:** 001
**Cubre:** RF-02

## Qué se quiere
Que la cuenta creada pueda entrar y mantener la sesión.

## Dónde puede tocar
- Permitido: app/auth/**, tests/auth/**
- Prohibido: app/bitvavo/**, el esquema de la base, añadir dependencias

## Criterios de aceptación
1. CUANDO las credenciales son correctas ENTONCES devuelve una cookie
   httpOnly con un JWT de 1 hora.
2. CUANDO la contraseña es incorrecta ENTONCES 401, y el mensaje es
   EL MISMO que si el correo no existe.
3. CUANDO faltan credenciales ENTONCES 422.
4. La cookie lleva SameSite=Lax y Secure.

## Done when
Los cuatro pasan y ruff no da avisos nuevos.

## Verify
    pytest tests/auth/test_login.py -q
    ruff check app/auth

## Qué devuelve
Tres líneas: qué cambió · qué se rompió · qué decidió por su cuenta.

Status: Draft
```

**Qué tener en cuenta:**

- **DATO.** De los bloqueos que paran a un agente, **el 42% son información ausente**, el 36% requisitos ambiguos y el 22% contradictorios ([HiL-Bench](https://huggingface.co/papers/2604.09408), 300 tareas y 1.131 bloqueos).
- **DATO.** Con la verificación idéntica y **sólo el texto cambiado**, GPT-4 pasa de **73,8% a 6,7%** con un enunciado contradictorio ([arXiv:2507.20439](https://arxiv.org/abs/2507.20439)).
- **El agente no va a preguntar.** Sólo pide aclaraciones entre el **31,8% y el 44,5%** de las veces ([UnderSpecBench](https://arxiv.org/abs/2607.02294)).

**Qué revisar:** que cada criterio **se pueda comprobar con un comando**. Lo que no, no es un criterio.

**Lo que necesita tu mano:** decidir qué se construye. **Nada lo hace por ti.**

---

## ETAPA 2 · La cola y el disparo

**Modelo:** no aplica. Es git.

**Software:** git. Ya lo tienes.

**La convención que gobierna la cola** —leída del código del `spec_queue.sh`:

| En el fichero | Significa |
|---|---|
| `Status: Draft` | Pendiente |
| `Status: COMPLETE` | Hecha |

El script busca en `specs/`, **ordena por nombre** —el número bajo va primero— y coge el primero que no diga `COMPLETE`. Y lo lee con esta expresión exacta:

```
^(#{1,3} )?(\*\*)?Status(\*\*)?:[[:space:]]+COMPLETE
```

**Si escribes otra cosa, la tarea nunca se da por terminada.**

**Y el contador de intentos va dentro del propio fichero**, como comentario: `<!-- NR_OF_TRIES: 5 -->`. **A los 10 la marca como atascada** y hay que partirla.

**Los comandos:**

```bash
git add specs/ && git commit -m "tareas: 001 a 003"
./scripts/ralph-loop.sh 20
```

**Qué tener en cuenta:**

- **DATO.** El planificador **falla en silencio**: en un caso real, el cron se saltó **7 disparos consecutivos sin error** ([issue #53610](https://github.com/anthropics/claude-code/issues/53610)). Y el cron de GitHub **se desactiva a los 60 días** sin actividad en repos públicos.

**Qué revisar:** que la tarea anterior esté en `COMPLETE`. Si no, el bucle la cogerá otra vez.

**Lo que necesita tu mano:** poner el vigilante. Un planificador que falla callado necesita que alguien mire que sigue vivo.

---

## ETAPA 3 · El bucle

**Modelo:** DeepSeek para implementar, Opus para lo ambiguo. **Conmutable desde el campo de la tarea.**

**Software:** el bucle descargado + el router.

**Los ficheros que ya tienes en el Paso 2.3**, más los dos prompts que el script genera solo, de cuatro líneas:

```
# Ralph Loop — Build Mode
You are running inside a Ralph Wiggum autonomous loop (Context A).
Read .specify/memory/constitution.md — it contains all project principles,
workflow instructions, work sources, and completion signal requirements.
Find the highest-priority incomplete work item, implement it completely,
verify all acceptance criteria, commit and push, then output
<promise>DONE</promise>.
```

**El mecanismo que lo cierra:** el agente **sólo puede terminar si escribe literalmente `<promise>DONE</promise>`**. El bucle busca esa cadena; si no está, repite.

**Las cuatro reglas del bucle, cada una con su motivo medido:**

| Regla | El dato |
|---|---|
| **Una tarea, un contexto limpio** | 18 modelos empeoran al crecer la entrada; **mejor con ~300 tokens que con ~113.000** ([Chroma](https://www.trychroma.com/research/context-rot)) |
| **El veredicto lo da el runner, no el agente** | Un agente reescribió los resultados de los tests: **500 de 500 sin resolver nada** ([Berkeley RDI](https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/)) |
| **Un commit por tarea** | Permite que `git bisect` señale la tarea culpable |
| **Parar por patrón, no por reloj** | Vigilantes por tiempo mataron **9 de 325 (2,8%)** ejecuciones **sanas** ([issue #85265](https://github.com/anthropics/claude-code/issues/85265)) |

**Qué revisar aquí:** el registro en `logs/`. **Si dos vueltas seguidas hacen lo mismo, para.**

**Lo que necesita tu mano, en esta etapa:** **nada, si la tarea está bien escrita.** Y si no lo está, no es un fallo del bucle: es un fallo del Paso 1.

---

## ETAPA 4 · La verificación

**Modelo:** el runner. **El agente no toca los tests que deciden.**

**Software:** el runner de tests de tu proyecto (pytest, vitest, lo que sea).

**Los ficheros — aquí está la parte que el montaje NO trae y hay que montar:**

```
tests/               ← visible: el agente las lee y trabaja contra ellas
verificacion/        ← OCULTA: fuera del alcance de permisos. Decide.
```

**Qué tener en cuenta:**

- **DATO, en las dos direcciones.** **31,08%** de parches aprobados lo son con tests débiles ([SWE-bench+](https://arxiv.org/abs/2410.06992)); y **35,5%** de las tareas auditadas tienen tests **demasiado estrictos** que *«invalidan entregas funcionalmente correctas»* ([OpenAI](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/)).
- **DATO.** El evaluador automático **sobrestima la decisión real del mantenedor en 24,2 puntos** ([METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/)).
- **El límite que ningún oráculo cubre:** *«los tests dan realimentación **en segundos**, pero el coste de una mala arquitectura se mide **en semanas o meses**»* ([HumanLayer](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)).

**Los topes, copiados de un montaje real:** `50 iteraciones`, `500.000 tokens`, `5 dólares`.

**Qué revisar:** que `verificacion/` esté en la lista de **prohibidos** de los permisos.

**Lo que necesita tu mano:** **montar el doble de prueba** cuando hay dependencia externa (una API, un banco, un servicio). Eso lo decides tú una vez.

---

## ETAPA 5 · El aislamiento

**Modelo:** no aplica.

**Software:** `docker` — **hay que instalarlo**.

**Los ficheros:**

| Fichero | Qué lleva | Dónde |
|---|---|---|
| `.claude/settings.json` | **La lista cerrada de permisos.** Lo que no esté, no lo hace | Raíz del proyecto |
| `.env` | Las claves de Bitvavo y la base de datos | **Fuera de git**, en `.gitignore` |
| `docker-compose.yml` | El contenedor, si lo usas | Raíz |

**Cómo queda la lista de permisos** — **diseño mío, marcado como tal**, siguiendo el principio de que el permiso obliga y la prosa no:

```json
{
  "permissions": {
    "allow": [
      "Read", "Edit", "Write",
      "Bash(pytest:*)", "Bash(ruff:*)", "Bash(git diff:*)", "Bash(git status:*)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(verificacion/**)",
      "Edit(verificacion/**)",
      "Edit(.claude/**)",
      "Bash(git push:*)",
      "Bash(curl:*)", "Bash(wget:*)",
      "WebFetch"
    ]
  }
}
```

**Fíjate en las tres prohibiciones que importan:** `verificacion/**` (para que no se examine a sí mismo), `.env*` (para que no lea las claves), y **`curl`/`wget`/`WebFetch`** — porque **la salida de red es lo que se exfiltra**.

**Qué tener en cuenta:**

- **DATO.** El worktree de git **aísla ficheros, no estado**: **2 de 5 agentes** operaron sobre el repositorio principal y uno hizo commit **en la rama de otro** ([issue #83311](https://github.com/anthropics/claude-code/issues/83311)); **~43 incidentes** en una semana ([issue #76250](https://github.com/anthropics/claude-code/issues/76250)).
- **DATO.** Y `.git` es escribible desde el worktree: **un agente puede plantar un gancho**.

**Qué revisar:** que no haya **comodines** en los permisos. Si pone `Bash(*)`, los permisos no existen.

**Lo que necesita tu mano:** **todo.** Esto se monta una vez y no se delega.

---

## ETAPA 6 · El cierre

**Modelo:** tú.

**Software:** git.

**Los comandos:**

```bash
git checkout principal
git merge tarea/002-login
```

**Qué tener en cuenta:**

- **DATO.** Las fusiones de agente necesitan arreglos posteriores con **1,62 veces las probabilidades** de las humanas ([arXiv:2609.26847](https://arxiv.org/abs/2609.26847)). *Matiz: es una razón de probabilidades, y sólo el **4,5%** de los PRs de agente fusionados recibe un arreglo verificado en 30 días.*
- **DATO.** La autonomía real medida: **4,7%** de PRs de agentes autónomos en el decil de mayor adopción, y **0,1%** en el mejor 60% ([LinearB](https://linearb.io/blog/does-your-software-factory-work)). *Ojo: es el techo de una muestra auto-seleccionada de clientes de LinearB, y el propio vendedor se contradice entre sus dos páginas.*

**Qué revisar, en este orden:** el diff y no la descripción · los nombres y los sitios · **la sección de desviaciones** (lo que decidió por su cuenta) · y si no entiendes una línea, **no fusiones**.

**Y el cierre del fichero:** la tarea pasa a `Status: COMPLETE`, se anota lo aprendido en el registro, **y si la tarea reveló un hueco en sí misma, se corrige antes de archivarla**. Es lo que hace que la siguiente salga mejor.

**Lo que necesita tu mano:** **todo.**

---

# PARTE 5 · SEGURIDAD, que no es opcional

| Riesgo | El caso real | Qué lo corta |
|---|---|---|
| **Exfiltración de claves** | **Claude Code filtró secretos de un `.env` por consultas DNS** — con CVE y parche en dos semanas · **OpenHands** filtró un `GITHUB_TOKEN` · **Cursor** filtró datos con un diagrama | Prohibir la salida de red en los permisos, y no darle claves que no necesite |
| **Inyección de prompt** | Instrucciones escondidas en un repositorio, un README, un comentario o la respuesta de una búsqueda | Que el agente **no lea** lo que no necesita, y nada de `WebFetch` en el bucle |
| **Escape del entorno** | La cadena de cuatro CVE de **CrewAI** escapa del sandbox por inyección; **Codex CLI en Windows** convirtió una búsqueda web en ejecución de comandos en el host | Contenedor de verdad, no confiar en el aislamiento del agente |
| **Agentes atacando internet** | El **AISI británico** documentó **19 acciones no autorizadas en 10 de 122 ejecuciones**, incluido un ataque real a la cadena de suministro | Sin acceso libre a la red |

**La «trifecta letal»:** acceso a datos privados + entrada no confiable + salida al exterior. **Con las tres, la fuga es cuestión de tiempo. Quitar una basta.**

**Y el aviso del propio repo del bucle, que conviene copiar en la constitución:**

> «Esta herramienta concede a los agentes una autonomía significativa sobre tu código y tu sistema. Revisa todos los cambios y úsala en entornos aislados cuando sea posible.»

---

# PARTE 6 · COSTES Y TOPES

| Concepto | Cifra | Fuente |
|---|---|---|
| Claude Pro | **~20 €/mes** | — |
| DeepSeek por API | Pago por consumo | — |
| **El bucle, coste real medido** | **10 $/hora** de cómputo | [The Register](https://www.theregister.com/2026/01/27/ralph_wiggum_claude_loops/) |
| **Un montaje llevado al extremo** | **1.000 $/día/ingeniero** en tokens | [StrongDM](https://factory.strongdm.ai/) |
| **Los topes que hay que poner** | 50 vueltas · 500.000 tokens · 5 $ | Montaje real documentado |

**Y el aviso que casi nadie da:** *«Ralph es una idea fantástica **si tus tokens son infinitos**, como cuando trabajas en Anthropic»*. El bucle quema rápido, y el fallo caro no es el precio unitario: **es el bucle descontrolado**. Los topes no son opcionales.

---

# PARTE 7 · LO QUE NECESITA TU MANO, en una tabla

| Etapa | Tu mano | El dato que lo obliga |
|---|---|---|
| 1. Escribir la tarea | **Decidir y escribir** | El 60% de las tareas no salen de un solo intento |
| 2. La cola | **El vigilante** | El planificador falla en silencio: 7 disparos perdidos |
| 3. El bucle | Nada, si la tarea está bien | — |
| 4. Verificación | **El doble de prueba** | Sobrestima 24,2 puntos |
| 5. Aislamiento | **Todo** | Hay exfiltración real con CVE |
| 6. Cierre | **Todo** | Los merges de agente necesitan 1,62× más arreglos |

**Y el límite que ningún montaje resuelve:** **nadie publica la tasa de éxito.** Ninguno de los que se han encontrado dice cuántas tareas salieron bien de cuántas. El creador del patrón no lo usaría sobre código existente. **Nada de esto es una promesa: es un montaje que funciona en tareas acotadas y verificables, y que hay que probar en el tuyo.**

---

# ANEXO · Los repos, todos

| Repo | Qué es | Por qué |
|---|---|---|
| [`musistudio/claude-code-router`](https://github.com/musistudio/claude-code-router) | **El router.** 37.461★, MIT | **La pieza que hace conmutable el modelo** |
| [`fstandhartinger/ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum) | El bucle con cortacircuitos, reintentos y avisos | **El más completo** |
| [`khgs2411/flow`](https://github.com/khgs2411/flow) | Un solo script de bash, sin dependencias | Si quieres lo mínimo |
| [`ghuntley/how-to-ralph-wiggum`](https://github.com/ghuntley/how-to-ralph-wiggum) | El del creador del patrón | La fuente primaria |
| [`BerriAI/litellm`](https://github.com/BerriAI/litellm) | Alternativa al router. 59.793★ | Si prefieres otra pasarela |
| [`github/spec-kit`](https://github.com/github/spec-kit) | La plantilla de las tareas | La de más adopción: 139.234★ |

## Enlaces

- [[receta-completa]] — los ficheros del bucle, uno a uno
- [[una-tarea-completa]] — los 17 pasos de una tarea
- [[etapas]] — qué está maduro y qué no
- [[gratis-y-local]] — la vía gratuita, y por qué aquí no aplica
- [[quien-dice-que]] — de dónde sale cada dato
