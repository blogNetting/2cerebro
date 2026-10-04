---
title: Control de versiones y CI en Patrimonial
created: 2026-10-01
updated: 2026-10-04
tags: [git, github, ci, devops, rulesets, patrimonial]
zona: tecnico
---

Cómo se protege `main` en el repo de Patrimonial, quién puede escribir dónde, qué comprobaciones automáticas (CI) se ejecutan al abrir un Pull Request y cómo montar todo eso a mano en GitHub. Es el "cómo" que acompaña al montaje de agentes descrito en [[gas-city-alcalde]].

## Aviso (2026-10-04): el §5 no se puede aplicar en este repo, hoy

El repo es **privado** y la cuenta es **Free**, y en esa combinación GitHub no ofrece rulesets ni protección de rama. Comprobado contra la API con la propia cuenta:

```
$ gh api repos/blogNetting/patrimonial/rulesets
{"message":"Upgrade to GitHub Pro or make this repository public to enable this feature.","status":"403"}

$ gh api repos/blogNetting/patrimonial/branches/main/protection
{"message":"Upgrade to GitHub Pro or make this repository public to enable this feature.","status":403}
```

O sea: **el §5 explica bien cómo se monta la puerta, pero esa pantalla hoy no existe para este repo.** Lo que sí funciona gratis es el CI del §7 — Actions está activo (`{"enabled":true,"allowed_actions":"all"}`) —; lo que falta es que la puerta obligue. El abanico de salidas, con lo que cuesta cada una, está en [[forges-gratis-con-puerta-en-main]]; el detalle de qué parte es de Gas City y qué de GitHub, en [[gas-city-y-gitlab]].

**Segunda corrección, mismo día: el `ci.yml` del §7.2 no encaja con este repo.** No hay `requirements.txt` (las dependencias viven en `pyproject.toml`, extras `dev`), exige Python `>=3.13` (aquí pone 3.12) y ya existe `make check` (`ruff check .` + `pytest -m "not network"`). Hay que adaptarlo antes de copiarlo.

## 1. A qué montaje aplica

- Repo privado: `blogNetting/patrimonial`.
- Dos ramas: **`development`** (los agentes trabajan aquí con total libertad) y **`main`** (la versión buena, protegida; solo el usuario decide cuándo avanza).
- Los agentes de Gas City: el **alcalde** (el *mayor*, que planifica y gestiona) y los **obreros** (los *polecat*/*obrero*, que implementan). El detalle de quién es quién está en [[gas-city-alcalde]].
- Objetivo: que el alcalde y los obreros **no puedan** tocar `main` aunque quieran, y que `main` solo avance por una puerta que el usuario controla.

## 2. Vocabulario (para no perderse)

| Término | Qué significa, en una frase |
|---|---|
| **CI** (*Continuous Integration*, integración continua) | Ejecutar pruebas y comprobaciones automáticas cada vez que alguien cambia el código, para detectar fallos pronto. |
| **CD** (*Continuous Delivery/Deployment*, entrega/despliegue continuo) | Llevar automáticamente el código ya integrado hasta un entorno desplegable (o desplegarlo). **No entra en este documento.** |
| **PR** (*Pull Request*) | Una *propuesta* de meter los cambios de una rama en otra. Es una caja de revisión y discusión, no una protección por sí sola. |
| **Ruleset** (conjunto de reglas) | Las reglas que GitHub aplica a una rama: quién puede escribirla, qué debe cumplirse antes de fusionar, etc. Antes se llamaba *branch protection rules* (protección de ramas); es lo mismo con otro nombre. |
| **Status check** (comprobación de estado) | El resultado de una comprobación del CI sobre un commit concreto: pasa ✅ o falla ❌. |
| **Check requerido** (*required*) | Un status check que **obliga**: mientras no pase, no se puede fusionar. |
| **Strict mode** | La casilla "Require branches to be up to date before merging": exige que la rama del PR contenga todo lo de `main` y que el CI vuelva a correr sobre el resultado real de la fusión. |
| **Tag** (etiqueta) | Una marca fija y con nombre en un commit concreto (p. ej. `v1.0.0`). No cambia nunca; sirve para volver a un estado bueno. |
| **Token** | La credencial con la que un programa (aquí, un agente) se autentica contra GitHub. Define *qué repos* y *qué permisos*, no *qué ramas* — eso último lo decide el ruleset. |

## 3. Las dos ramas

| Rama | Quién escribe | Protección |
|---|---|---|
| **`development`** | El alcalde y sus obreros, a su aire. Zona libre. | Ninguna. Es el terreno de pruebas del usuario. |
| **`main`** | **Nadie directamente.** Solo avanza cuando el usuario fusiona un PR. | Ruleset activo: sin push directo y con CI obligatorio; el usuario decide al pulsar Merge. |

La idea de fondo: **un único punto de parada —la fusión `development → main`— donde el sistema pide permiso**. Todo lo demás es libre. El porqué de concentrar la frontera solo ahí (y no en cada fusión) está razonado en [[gas-city-alcalde]] y, en general, en la práctica de "proteger la frontera, no cada merge".

## 4. Quién puede escribir dónde: el token

Punto clave, porque es lo que responde a "que el alcalde, aunque quiera, no pueda".

- El mecanismo que limita **qué repositorio** y **qué tipo de permiso** es el **token** (o una GitHub App / deploy key). El mecanismo que limita **qué rama** es el **ruleset**. Los dos hacen falta: el token dice "puedes escribir en este repo", el ruleset dice "pero no en `main`".
- **Regla dura:** la lista de *bypass* del ruleset (quién puede saltarse las reglas) se deja **vacía**. Si está vacía, **nadie** salta las reglas: ni el alcalde, ni un administrador. Si añades a alguien ahí, esa persona/robot sí podría empujar a `main`.
- **Regla dura:** el alcalde y los obreros **no** deben operar con una credencial que sea administradora del repo, ni estar en la lista de bypass.
- Un token de *fine-grained* (PAT moderno) permite elegir `Contents: Read and write` sobre este repo, pero **no** permite decir "solo en `development`" — esa restricción de rama es exclusivamente del ruleset. Por eso el ruleset no es opcional.
- **Aviso específico de Gas City:** el pack `gastown` fusiona por defecto con `merge_strategy=direct`, que significa *fusionar y empujar normalmente* ([prompt del refinery](https://github.com/gastownhall/gascity-packs/blob/main/gastown/agents/refinery/prompt.template.md), vía [[gas-city-alcalde]]). Es decir, **el agente intentará empujar a `main` por diseño**. Dos barreras: (a) el ruleset se lo impide, y (b) si quieres que no lo intente siquiera, marca las tareas con `merge_strategy=pr`. La barrera (a) es la que no se puede esquivar.

## 5. Montar el ruleset en GitHub, paso a paso

Interfaz actual de GitHub (2026). Se hace **una vez**:

1. En el repo: **Settings** → **Rules** → **Rulesets** → botón **New ruleset** → **New branch ruleset**.
2. **Ruleset Name**: por ejemplo `protege-main`.
3. **Enforcement status**: **Active** (si lo dejas *Disabled* no hace nada).
4. **Bypass list**: **déjala vacía**. Este es el interruptor de "el alcalde no puede aunque quiera". No añadas a nadie aquí — ni tu cuenta, si quieres que ni tú puedas saltártelo.
5. **Target branches** → **Add target** → **Include default branch** (o *Include by pattern* → `main`). Asegúrate de que apunta a `main`, no a `development`.
6. **Rules** — activa, como mínimo:
   - **Restrict creations** (nadie crea `main` desde cero sin permiso).
   - **Restrict deletions** (nadie borra `main`).
   - **Require a pull request before merging** → **Required approvals: 0**. Esta regla es la que impide el push directo: todo cambio en `main` tiene que llegar por un PR. Aprobaciones a 0 porque el PR lo abres tú y GitHub no deja aprobar un PR propio (*"Pull request authors cannot approve their own pull requests"*, [GitHub Docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/approving-a-pull-request-with-required-reviews)); con 1, tu propio PR se quedaría bloqueado. La decisión es tuya al pulsar Merge.
   - **NO actives Restrict updates.** GitHub la define así: *"If selected, only users with bypass permissions can push to branches or tags whose name matches the pattern you specify"* ([GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)). Fusionar un PR también es un push a `main`, así que con la lista de bypass vacía **nadie podría fusionar nunca**, ni tú. Lo han comprobado usuarios en [community #113172](https://github.com/orgs/community/discussions/113172): con la regla activa aparece *"Merging is blocked / The base branch does not allow updates."*, y *"Even if 'repository admins' are included in the bypass list for a ruleset which has the "Restrict update" rule enabled, they cannot merge PRs which are targeted at branches matched by the ruleset"*. En [#150269](https://github.com/orgs/community/discussions/150269) también se ve que la lista de bypass no exime: *"Experimentation shows that the bypass list does not bypass the listed actor from the Ruleset"*. En contra, un [blog de Medium (Pankaj Aswal)](https://iampankajaswal.medium.com/github-branch-rulesets-explained-protecting-master-keeping-branches-in-sync-and-building-a-safe-2d00ad1680ff) recomienda *"Restrict updates / Require pull request / Block force pushes"*, pero no dice haberlo probado con una fusión real. Pesan más los dos hilos, que sí cuentan pruebas reales. Comprobado solo con fuentes, sin prueba propia en un repo. (Corrección de una versión anterior de esta nota, que la recomendaba.)
   - **Require status checks to pass** → añade **`tests`** y **`lint`** (los nombres de los jobs del §7.2) y marca **Require branches to be up to date before merging** (esto es el **strict mode**, que actúa como se explica en el §6).
   - **Block force pushes** (nadie reescribe la historia de `main`).
7. **Create**. A partir de aquí, todo PR hacia `main` queda sujeto a estas reglas.

### Qué NO hace falta para empezar

- **Merge queue**: la cola nativa de GitHub que prueba fusiones combinadas. Sirve para repos con **muchos PR concurrentes** hacia `main`. Como aquí se fusiona de uno en uno, no aporta y añade complejidad. Si algún día crecen los PR simultáneos, se revisa.
- **Codeowners / aprobaciones**: útiles en equipo. Para un solo usuario que abre y fusiona él mismo, 0 aprobaciones.

## 6. El PR `development → main`: qué pasa al abrirlo

Secuencia real, en orden:

1. Abres tú el PR de `development` hacia `main`.
2. GitHub dispara el workflow del CI (el fichero del §7). Cada comprobación es un **job** independiente; corren y **suman** sus resultados como *status checks* sobre el commit del PR.
3. Cada job termina con un **código de salida**: `0` = ✅ pasa; **distinto de `0`** = ❌ falla. Ese es el "qué devuelve para continuar o parar": un test roto lanza salida ≠ 0, y el job entero falla.
4. Los checks que marcaste como **requeridos** (`tests`, `lint`) aparecen en el PR. Mientras **alguno esté rojo o pendiente**, el botón **Merge** queda **bloqueado**.
5. Cuando **todos** los requeridos están en verde, el botón **se habilita**. Entonces decides tú: pulsas Merge o cierras el PR sin fusionar.
6. **Modo strict**: si `main` recibiera un commit nuevo entre medias, el PR quedaría "out of date" y habría que resincronizar y **volver a correr el CI** sobre el resultado real. En tu caso `main` no se mueve solo (nadie empuja a `main`, §5), así que esto casi nunca se activará — pero es la red que garantiza que lo que pruebas es lo que se fusiona.
7. **Por qué NO hay que repetir los tests tras el merge**: en un `pull_request`, GitHub no prueba la rama sola sino la fusión simulada (`GITHUB_REF` = *"PR merge branch `refs/pull/PULL_REQUEST_NUMBER/merge`"*, [GitHub Docs — events](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)), y el modo strict garantiza que esa fusión sigue al día cuando pulsas Merge. Es decir, el CI ya corrió sobre el árbol exacto que quedará en `main`. Repetir los mismos tests después sería redundante. Lo único que sí puede correr *sobre `main` ya fusionada* es el **despliegue/release** (construir, publicar, desplegar), que es otra cosa y no entra aquí.

## 7. El CI de Patrimonial

### 7.1 Comprobaciones: todas las propuestas

Estas son las que se consideraron, con el estado de cada una hoy (2026-10-01):

| Comprobación | Herramienta | Qué caza | Estado |
|---|---|---|---|
| **Tests** | `pytest` + `coverage` | regresiones; y umbral de cobertura | ✅ **Elegida** |
| **Lint + formato** | `ruff check` / `ruff format --check` | estilo, código muerto, errores obvios, formato | ✅ **Elegida** |
| Dependencias | `pip-audit` (o Dependabot nativo) | CVEs conocidos en las librerías | ⏳ Propuesta, no elegida |
| Seguridad del código | `bandit` (SAST: análisis de seguridad sin ejecutar el código) | patrones peligrosos, inyecciones | ⏳ Propuesta, no elegida |
| Tipos | `mypy` | errores de anotaciones de tipo | ⏳ Propuesta, no elegida |
| Secretos | *secret scanning* + *push protection* de GitHub | claves coladas en commits | ⏳ Propuesta, no elegida (es gratis; recomendable activarlo) |

**Umbral de cobertura elegido: 80 %.** Si el código probado baja del 80 %, el job `tests` falla y bloquea el merge. Se configura con `--cov-fail-under=80`.

Nota sobre **Ruff**: hoy es el estándar de facto para Python —reemplaza flake8 + black + isort en una sola herramienta más rápida— y lo usan FastAPI, pandas, Airflow, Hugging Face, SciPy y PyTorch, entre otros. Adopciones reales documentadas: [NetBox](https://github.com/netbox-community/netbox-acls/issues/284), [IntelOwl PR #3145](https://github.com/intelowlproject/IntelOwl/pull/3145), [foro de Pulp](https://discourse.pulpproject.org/t/preview-toolchain-upgrade-black-flake8-ruff/2174). Comparativa de velocidad y reglas (blog técnico independiente): [tutorials.technology — Ruff 2026](https://tutorials.technology/tutorials/ruff-python-linter-tutorial-2026.html).

Guía práctica que repiten los repos reales: **no meter todas las comprobaciones el primer día**. Empezar con tests + Ruff, y añadir seguridad/dependencias/tipos cuando el pipeline ya ruede. Una de las propuestas no elegidas hoy puede incorporarse más adelante sin tocar nada más que el fichero del §7.2.

### 7.2 El fichero del CI (`.github/workflows/ci.yml`)

Va en el repo, no en el wiki. Es el "cómo se ejecutan los tests" que pediste:

```yaml
name: CI

# Cuándo se dispara: al abrir o actualizar un PR que apunta a main.
on:
  pull_request:
    branches: [main]

jobs:
  # Job 1: pruebas. El nombre "tests" es el que se marca como requerido en el ruleset.
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt -r requirements-dev.txt
      # --cov-fail-under=80: si la cobertura baja del 80 %, sale con error y bloquea el merge.
      - run: pytest --cov=. --cov-fail-under=80

  # Job 2: estilo y formato. El nombre "lint" es el otro check requerido.
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff
      - run: ruff check .          # reglas de estilo y errores evidentes
      - run: ruff format --check . # comprueba el formato SIN modificarlo
```

- Los dos nombres —`tests` y `lint`— son **exactamente** los que hay que escribir en el ruleset (§5, paso 6). Si no coinciden, GitHub no los reconoce como requeridos.
- `ruff format --check` **no** reformatea: solo dice si el archivo está bien o mal. Arreglarlo es cosa tuya (o de un `ruff format` en local).

### 7.3 Cómo lo montas tú

1. Copia el `ci.yml` del §7.2 a `.github/workflows/ci.yml` dentro del repo de Patrimonial.
2. Añade a `requirements-dev.txt` (o donde tengas las dependencias de desarrollo) `pytest`, `pytest-cov` y `ruff`.
3. Sube esos cambios a `development`.
4. Abre un PR de `development` a `main`. El CI se disparará **aunque el ruleset todavía no exista** — solo que sin él, si algo falla, aún podrías fusionar. Verás los dos jobs corriendo en la pestaña *Checks*.
5. Monta el ruleset del §5 y, en el paso 6, escribe `tests` y `lint` como checks requeridos. Desde ese momento, el botón de merge queda bloqueado hasta que los dos estén en verde.

## 8. Recomendaciones

**R1 — Etiquetar `main` en cada estado bueno (tags).** "Saber que algo funciona y no tocarlo" = un `tag` inmutable al que volver. Sin esto no hay versionado, solo una rama que va cambiando. El cómo, en el §9.

**R2 — El modelo de dos ramas es el correcto.** Que los agentes trabajen libres en `development` y el gate humano esté solo en `development → main` es el patrón que aplican los equipos que usan agentes en serio (GitHub Copilot coding agent nunca commitea directo a `main` y deja la fusión a un humano; guías de seguridad de agentes de Microsoft recomiendan ramas dedicadas). No hay que cambiarlo.

**R3 — Una sola `development` compartida: riesgo aceptado.** Con varios agentes empujando a la misma rama, si algo se rompe **no se puede saber qué cambio lo hizo** (no hay a quién culpar ni qué revertir por partes). Se acepta a cambio de no entorpecer al alcalde. Si algún día importa, la solución es una rama/worktree por agente que fusiona a `development`.

**R4 — No repetir los tests tras el merge.** Con strict mode, el CI del PR ya corre sobre el resultado real de la fusión. Lo único que corre *sobre `main`* después es el despliegue/release, no los tests. (Corrección de una versión anterior de esta recomendación.)

**R5 — El CI es necesario, no suficiente.** Verde = 0/1, desbloquea el botón, pero **no** sustituye tu revisión. Revisando un conjunto grande de PR generados por IA en un proyecto real, aproximadamente la mitad de los que pasaban los tests no se habrían fusionado igualmente ([HackerNoon — "The Safe Way to Ship Production Code Written by AI Agents"](http://hackernoon.com/lite/the-safe-way-to-ship-production-code-written-by-ai-agents)). Por eso sigues revisando tú.

## 9. Tags: crear, volver y descargar

Un **tag** es una marca con nombre en un commit. Dos formas:

- **Lightweight**: `git tag v1.0.0` — solo un puntero.
- **Annotated** (recomendado, lleva fecha, autor y mensaje): `git tag -a v1.0.0 -m "Primera versión estable"`.

**Crear un tag y subirlo al repo:**

```bash
git checkout main
git pull                      # asegúrate de estar en el main bueno
git tag -a v1.0.0 -m "Primera versión estable"
git push origin v1.0.0        # los tags NO se suben con un push normal: hay que empujarlos
```

Subir todos de golpe: `git push --tags`.
Ver los que hay: `git tag -l`.

**Crearlo desde la web (sin terminal):** repo → **Releases** → **Draft a new release** → en *Choose a tag* escribes la versión (p. ej. `v1.0.0`), se crea al publicar. Desde aquí GitHub te deja además adjuntar notas de la versión.

**Volver a un estado concreto (recuperar un `main` que funcionaba):**

```bash
git fetch --tags              # trae los tags que no tengas
git checkout v1.0.0           # te deja el código de ese tag (HEAD "desprendido")
```

Estar en un tag es solo mirar: si quieres **trabajar a partir de él**, crea una rama:

```bash
git checkout -b arreglo-desde-v1.0.0 v1.0.0
```

**Descargar uno concreto sin tocar tu repo:**

```bash
# Solo ese tag, a un fichero comprimido:
git archive --format=tar.gz --output=patrimonial-v1.0.0.tar.gz v1.0.0
```

O desde la web: repo → **Releases** → el release que quieras → *Source code (zip)* o *(tar.gz)*.

**Recomendación de uso:** cada vez que fusiones a `main` algo que has probado y funciona, etiquétalo (`v1.0.0`, `v1.0.1`…). Ese tag pasa a ser el "estado bueno conocido": si una versión futura falla, vuelves a él.

## Enlaces

- [[patrimonial]] — hub del proyecto (estado, stack, repo)
- [[gas-city-alcalde]] — quién es el alcalde, los obreros y cómo fusionan (`merge_strategy`)
- [[forges-gratis-con-puerta-en-main]] — qué forge da la puerta gratis en privado, y qué cuesta cada salida
- [[gas-city-y-gitlab]] — qué parte de esto es de Gas City y qué parte de GitHub
- [[_index]] — índice de esta carpeta

### Fuentes

- Rulesets y checks requeridos, modo strict: [GitHub Docs — Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) (doc oficial de vendor: explica *cómo funciona*, no es prueba de que la práctica sea la mejor).
- Stack de CI para Python (pytest/coverage, Ruff, Bandit, pip-audit, mypy): [python-standard-stack.yml](https://raw.githubusercontent.com/ByronWilliamsCPA/.github/4cb1d22e689f5141597b5801784f44b687d1b39a/docs/workflows/python-standard-stack.md) y [CI for Python Package](https://intersect-training.org/CI-CD/instructor/ci-for-python-package-unit-test.html) (material de formación y ejemplos de workflows reales).
- Adopción de Ruff: [NetBox #284](https://github.com/netbox-community/netbox-acls/issues/284), [IntelOwl PR #3145](https://github.com/intelowlproject/IntelOwl/pull/3145), [foro de Pulp](https://discourse.pulpproject.org/t/preview-toolchain-upgrade-black-flake8-ruff/2174) (proyectos reales migrando).
- Gate en la frontera, no en cada merge: [jonroosevelt.com — "Gate the Boundary, Not Every Merge"](https://www.jonroosevelt.com/blog/gate-the-boundary-not-every-merge/) (blog de ingeniero independiente).
- CI necesario pero no suficiente: [HackerNoon — "The Safe Way to Ship Production Code Written by AI Agents"](http://hackernoon.com/lite/the-safe-way-to-ship-production-code-written-by-ai-agents).
