---
title: Lego — la receta completa: ficheros, scripts y repos
created: 2026-09-28
updated: 2026-09-28
tags: [lego, receta, scripts, repos, ficheros]
zona: tecnico
---

**Todo lo que hay que poner para montarlo, fichero a fichero.** No es una descripción: son los ficheros con su contenido y los comandos exactos. Todo sale del repositorio real **[`fstandhartinger/ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum)** (MIT), que se ha descargado y leído entero — 18 ficheros, todos verificados.

## Qué trozo hace qué

Antes de los ficheros, el mapa. Son **cuatro capas**, y cada una tiene su papel:

| Capa | Ficheros | Para qué |
|---|---|---|
| **El bucle** | `scripts/ralph-loop.sh` + `scripts/lib/*.sh` | Lanza el agente una y otra vez, comprueba la promesa de fin, cuenta intentos, corta si se atasca |
| **El contrato** | `.specify/memory/constitution.md` | **La fuente única de verdad.** Reglas, principios, autonomía, cómo se trabaja |
| **La cola** | `specs/NNN-*.md` | **Las tareas.** Una por fichero, numeradas. El número bajo va primero |
| **El puntero** | `AGENTS.md` y `CLAUDE.md` | Cinco líneas cada uno. Sólo dicen «lee la constitución» |

**Y la pieza que falta en casi todos los montajes: la promesa.** El agente sólo puede terminar si escribe literalmente `<promise>DONE</promise>`. El bucle busca esa cadena. Si no está, repite. **Si está, avanza a la siguiente tarea.** Es el mecanismo que convierte «el agente dice que terminó» en «el agente tuvo que demostrarlo».

## Paso 1 — Los directorios

```bash
mkdir -p .specify/memory specs scripts scripts/lib logs history .claude/commands
```

## Paso 2 — Los scripts (se descargan, no se escriben)

```bash
base=https://raw.githubusercontent.com/fstandhartinger/ralph-wiggum/main

# El bucle, en sus cuatro variantes de agente
for a in "" -codex -gemini -copilot; do
  curl -o "scripts/ralph-loop$a.sh"  "$base/scripts/ralph-loop$a.sh"
  curl -o "scripts/ralph-loop$a.ps1" "$base/scripts/ralph-loop$a.ps1"
done

# Las piezas compartidas: cola, cortacircuitos, reintentos, avisos
for f in spec_queue circuit_breaker nr_of_tries response_analyzer date_utils notifications; do
  curl -o "scripts/lib/$f.sh" "$base/scripts/lib/$f.sh"
done

chmod +x scripts/ralph-loop*.sh
```

**Qué hace cada pieza compartida, leída del código:**

| Fichero | Qué hace de verdad |
|---|---|
| **`spec_queue.sh`** | **La cola.** Busca en `specs/` los ficheros `.md`, ordena alfabéticamente —**el número bajo va primero**— y devuelve el primero que **no** tenga `Status: COMPLETE`. Eso es lo que el agente va a implementar |
| **`circuit_breaker.sh`** | **El cortacircuitos.** Si algo se repite, para |
| **`nr_of_tries.sh`** | **El contador de intentos.** Lo guarda **dentro del propio fichero de la tarea**, como un comentario: `<!-- NR_OF_TRIES: 5 -->`. **A los 10 intentos marca la tarea como atascada** y hay que partirla |
| **`response_analyzer.sh`** | Analiza lo que devolvió el agente |
| **`notifications.sh`** | Avisos, opcionalmente por Telegram |

## Paso 3 — La constitución (este sí se escribe)

`.specify/memory/constitution.md` — **es el fichero que más importa de todos.** El contenido real de la plantilla, adaptado:

```markdown
# Constitución de Patrimonial

> Web autoalojada para seguir el patrimonio y el gasto.

## Version
1.0.0

---

## Detección de contexto

### Contexto A: bucle Ralph (modo implementación)
Estás en un bucle Ralph si:
- Te lanzó `ralph-loop.sh`
- El prompt menciona «implementa la spec»

**En este modo:**
- Coge la tarea incompleta de prioridad más alta
- Completa TODOS los criterios de aceptación
- Emite `<promise>DONE</promise>` sólo al 100%

### Contexto B: chat interactivo
Cuando no estés en un bucle: conversa, ayuda, crea tareas.

---

## Principios

### I. Los tests no se editan
Si un test falla, se arregla el código. Nunca el test.
Un test editado para que pase no es un test, es un adorno.

### II. La clave de Bitvavo vive en un solo sitio
En la API. Nunca en la interfaz. Nunca en el repositorio.

### III. Sencillez
Se construye exactamente lo que hace falta, nada más.

---

## Configuración de autonomía

### Modo sin preguntas: DESACTIVADO (primera vez)
### Autonomía de git: DESACTIVADO (commits sí, push no)
```

**Los dos principios I y II no son prosa decorativa:** el I es lo que impide que el agente arregle un test rojo cambiando el test, y el II acota dónde puede tocar. **Pero recuerda el dato: una regla escrita no obliga** — en el caso real medido, *«Opus 4.7 leyó esas reglas, las reconoció y las violó en la siguiente entrada del registro»*. **Lo que obliga son los permisos y los tests.** Esto es el recordatorio; los permisos son la cerca.

## Paso 4 — El puntero (cinco líneas cada uno)

`AGENTS.md` y `CLAUDE.md`, **el mismo contenido en los dos**:

```markdown
# Instrucciones del agente

**Lee la constitución**: `.specify/memory/constitution.md`

Ese fichero contiene TODAS las instrucciones del proyecto: principios,
flujo de trabajo, autonomía, y cómo se emite la señal de fin.
```

**Por qué dos ficheros iguales:** porque cada agente lee el suyo. Esto ya costó una bronca pública entre fabricantes ([[quien-dice-que]]), pero como **no tienen reglas propias**, no se pueden desincronizar.

## Paso 5 — La plantilla de tarea

`specs/002-login.md` — así es la plantilla real, rellenada con una tarea de verdad:

```markdown
# Specification: Login

## Feature: Inicio de sesión

### Overview
Que la cuenta creada pueda entrar y mantener la sesión.

### User Stories
- Como usuario, quiero entrar con mi correo para ver mi patrimonio.

---

## Functional Requirements

### FR-1: Autenticación con correo y contraseña
El endpoint recibe correo y contraseña y devuelve una sesión.

**Acceptance Criteria:**
- [ ] CUANDO las credenciales son correctas ENTONCES devuelve una cookie
      httpOnly con un JWT de 1 hora
- [ ] CUANDO la contraseña es incorrecta ENTONCES 401, con el MISMO
      mensaje que si el correo no existe
- [ ] CUANDO faltan credenciales ENTONCES 422
- [ ] La cookie lleva SameSite=Lax y Secure

---

## Success Criteria
- El criterio 2 se comprueba comparando los dos mensajes carácter a
  carácter: tienen que ser idénticos

## Dependencies
- Requiere la tarea 001 (registro) completada

---

## Completion Signal

### Implementation Checklist
- [ ] Endpoint POST /auth/login
- [ ] Emisión del JWT y la cookie
- [ ] Tests de los cuatro criterios

### Testing Requirements

El agente DEBE completar TODO esto antes de emitir la frase:

#### Calidad de código
- [ ] Todos los tests existentes pasan
- [ ] Tests nuevos para lo nuevo
- [ ] Sin errores de estilo

#### Verificación funcional
- [ ] Los cuatro criterios verificados
- [ ] Casos límite cubiertos
- [ ] Manejo de errores

### Iteration Instructions

Si ALGUNA comprobación falla:
1. Identifica el problema concreto
2. Arregla el código
3. Vuelve a correr los tests
4. Verifica todos los criterios
5. Commit
6. Comprueba otra vez

**Sólo cuando TODO pase, emite:** `<promise>DONE</promise>`

---

Status: Draft
```

**Fíjate en la última línea: `Status: Draft`.** Ésa es toda la cola. El script busca los ficheros que **no** digan `Status: COMPLETE`, los ordena por nombre, y coge el primero. **Para marcar una tarea como hecha, el agente cambia esa línea a `Status: COMPLETE`.**

**Y no es un detalle de estilo:** el `spec_queue.sh` lo lee con esta expresión exacta —`^(#{1,3} )?(\*\*)?Status(\*\*)?:[[:space:]]+COMPLETE`— o sea que acepta `Status: COMPLETE`, `**Status**: COMPLETE` y `## Status: COMPLETE`. **Si escribes otra cosa, la tarea nunca se da por terminada.**

## Paso 6 — Los prompts (los genera el script, no los escribes)

El script crea dos prompts, y son **cuatro líneas cada uno**:

`PROMPT_build.md`:
```
# Ralph Loop — Build Mode

You are running inside a Ralph Wiggum autonomous loop (Context A).

Read .specify/memory/constitution.md — it contains all project principles,
workflow instructions, work sources, and completion signal requirements.

Find the highest-priority incomplete work item, implement it completely,
verify all acceptance criteria, commit and push, then output
<promise>DONE</promise>.
```

`PROMPT_plan.md` — igual pero pidiendo un plan de desglose, sin implementar nada. **El modo plan es opcional**: la mayoría de proyectos funcionan directamente desde las tareas.

## Paso 7 — Arrancar

```bash
./scripts/ralph-loop.sh        # sin límite de vueltas
./scripts/ralph-loop.sh 20     # tope de 20 vueltas  ← empieza por aquí
./scripts/ralph-loop.sh plan   # modo plan, opcional
```

## El árbol completo, al final

```
tu-proyecto/
├── .specify/memory/constitution.md    ← el contrato
├── specs/
│   ├── 001-registro.md                ← la cola: el 001 va primero
│   └── 002-login.md
├── scripts/
│   ├── ralph-loop.sh                  ← el bucle
│   └── lib/                           ← cortacircuitos, reintentos, avisos
├── logs/                              ← todo lo que pasa, escrito
├── history/                           ← histórico
├── IMPLEMENTATION_PLAN.md             ← sólo si usas el modo plan
├── AGENTS.md                          ← apunta a la constitución
└── CLAUDE.md                          ← lo mismo
```

## Lo que este montaje SÍ trae de serie

| Pieza | Está |
|---|---|
| Bucle con contexto limpio por tarea | **Sí** |
| Promesa de fin verificada por el script | **Sí** |
| **Contador de intentos por tarea** (10 = atascada, hay que partirla) | **Sí** |
| **Cortacircuitos** | **Sí** |
| Registro completo en `logs/` | **Sí** |
| Avisos por Telegram | Opcional |
| Soporte para cuatro agentes (Claude, Codex, Gemini, Copilot) | **Sí** |

## Lo que NO trae, y hay que poner aparte

1. **El entorno aislado.** No está. Y es lo único que puede hacerte daño de verdad: hay exfiltración real documentada con CVE. **Hay que resolverlo aparte** — usuario dedicado, contenedor o máquina virtual.
2. **Los permisos.** El `settings.json` con la lista cerrada de lo que el agente puede tocar. **No viene**, y sin él las reglas de la constitución son sólo prosa.
3. **La separación entre tests visibles y de verificación.** El montaje no la trae. **Hay que montarla tú**, poniendo el directorio de verificación fuera del alcance de permisos.
4. **Tu mano en las decisiones.** El paso 2 y el 4 del [[una-tarea-completa]]: la arquitectura y el doble de prueba.

## Los otros dos repos, por si prefieres menos

| Repo | Qué es | Cuándo |
|---|---|---|
| **[`khgs2411/flow`](https://github.com/khgs2411/flow)** | **Un solo script de bash, ~63 KB, sin dependencias** | Si quieres lo mínimo absoluto. Un fichero. |
| **[`snarktank/ralph`](https://github.com/snarktank/ralph)** | El bucle con su marketplace de plugin | Si quieres el camino del plugin |
| **[`hotmolts`/`ghuntley/how-to-ralph-wiggum`](https://github.com/ghuntley/how-to-ralph-wiggum)** | El del creador del patrón | Si quieres la fuente primaria |

## Y lo que hay que aceptar antes de empezar

**Nadie publica la tasa de éxito de estos montajes.** Ninguno dice cuántas tareas salieron bien de cuántas. Lo que sí está medido:

- Funciona en **tareas acotadas y verificables**.
- El efecto **se desvanece en código heredado**.
- El creador del patrón: *«no usaría Ralph en una base de código existente ni de broma»*.
- Y su aviso de seguridad, que conviene copiar en la constitución: *«no es si va a petar, es cuándo. **¿Y cuál es el radio de la explosión?»***

## Enlaces

- [[una-tarea-completa]] — los 17 pasos que recorren estos ficheros
- [[montaje-documentado]] — los montajes reales y sus ficheros
- [[etapas]] — qué está maduro y qué necesita tu mano
- [[implementaciones-reales]] — los fracasos documentados
