---
title: Lego — el material práctico: cómo se monta de verdad
created: 2026-09-28
updated: 2026-09-28
tags: [lego, practico, curso, implementacion]
zona: tecnico
---

El material didáctico y de implementación —lo que enseña **cómo se monta**, no si funciona—. Es la fuente que faltaba, y viene en buena parte del curso que ya tenía delante y no había usado en serio.

## El curso de BIG school (Días 1 a 3)

Los tres PDF que me diste. En una primera pasada los despaché como «material de curso introductorio, sin evidencia», y **eso fue un error de lectura**: no son un paper, son **un temario de implementación**, y ahí está su valor. Lo que enseñan, por día:

- **Día 1** — fundamentos, ventana de contexto, elección de modelo, anatomía del prompt (rol, contexto, tarea, restricciones, formato), *vibe coding* frente a ingeniería, familias de editores, y un **checklist de código seguro con IA**: dar contexto y exigir validación por adelantado, leer y entender el código entero, verificar antes de subir, y no pegar claves ni datos de clientes en un prompt.
- **Día 2** — el arnés: `AGENTS.md` (dicen que **no debe pasar de 500 líneas**), MCP, skills, comandos, memoria y **bucle de verificación**; y la distinción **construir frente a planificar** (Build vs Plan).
- **Día 3** — **el desarrollo dirigido por especificación**, con su ciclo (constitución → especificar → plan → tareas → implementar → verificar), sus **tres niveles** (spec-first, spec-anchored, spec-as-source, y dicen que **el recomendable para empezar es el anclado**), la estructura de artefactos `spec/constitution/` + `spec/features/NNN/`, los multiagentes (coordinador, implementador, verificador) y la **ingeniería de bucles**.

**Su regla de oro, y coincide con todo lo que ha sobrevivido a la auditoría:** *«la IA ejecuta, pero el responsable del proyecto eres tú»* y *«la potencia sin control no sirve de nada»*.

## La guía de Claude Code («para devs que empiezan»)

**Este documento no lo había leído, y es el más práctico de todo el material.** No es una chuleta de comandos: son modelos mentales y recetas.

**Cuatro ideas, y las cuatro están respaldadas por la evidencia del informe:**

1. **La mesa de trabajo finita** — el contexto se llena y la calidad cae. Síntomas: el agente repite cosas, pierde decisiones de hace turnos, se contradice. Es exactamente el **deterioro del contexto** medido por Chroma en [[consumir-la-tarea]].
2. **El triángulo de la calidad** — lo que recibes depende de **contexto, modelo y permisos**, no sólo del prompt. Si el resultado es mediocre, mira los otros dos vértices antes de reescribir.
3. **Pensar antes de actuar** — entrar en modo plan antes de cualquier tarea no trivial: *«esto evita el 90% de los desastres»*. Es el mismo principio que el **freno antes de actuar** de [[contra-evidencia]].
4. **No-determinismo** — «dos veces el mismo prompt = dos resultados distintos». Y la consecuencia práctica: *si algo funciona, guárdalo*; **si algo falla, ajusta el contexto, no insistas con el mismo texto**.

### La receta de los primeros cinco minutos

```
cd tu-proyecto
claude
/init     # analiza el repo y genera CLAUDE.md
# revisa y ajusta el CLAUDE.md generado (es tuyo)
Shift+Tab → entra en plan mode
"Resúmeme la arquitectura y dime qué te falta saber"
```

Su remate: *«acabas de hacer lo que el 80% no hace: dar contexto antes de pedir»*.

### El ejemplo que contesta directamente a tu pregunta

Es el prompt bueno contra el malo, y la explicación de por qué:

**Malo:** «Hazme un login.»
**Bueno:** «Implementa auth con email + contraseña en `src/auth/`. Requisitos: bcrypt, JWT de 1h en cookie httpOnly, endpoint `POST /auth/login` que devuelva 401 sin filtrar si falló email o password. Tests Vitest cubriendo válidas, password incorrecto, email inexistente. **Antes de tocar nada, dime qué archivos vas a crear y espera mi OK.**»

Y la regla que lo resume, que es **la respuesta en una frase a «cómo tiene que crearse la tarea»**:

> «La diferencia: **outcome + restricciones + verificación + freno antes de actuar**. No es más largo por gusto: **cada línea bloquea un fallo típico**.»

Fíjate en que **aparecen los cinco campos** que el informe derivó por su cuenta: objetivo, alcance (la carpeta), criterios comprobables, la comprobación (los tests nombrados), y el freno. **Convergencia con material de curso.**

### Las cinco frases que dicen que funcionan

| Frase | Para qué |
|---|---|
| «Antes de tocar nada, dime el plan en 5 puntos y espera mi OK» | Convierte cualquier turno en modo plan informal |
| «Asume que soy junior. Explica las decisiones, no sólo el código» | Salen las razones, no sólo el código |
| «Sé escéptico con lo que te pida. Si algo huele mal, dilo» | Evita que te dé la razón sin pensar |
| «**Si no estás 80% seguro, pregunta. No inventes**» | Reduce alucinaciones |
| «Al terminar, lista qué probaste y qué falta probar» | Auto-revisión gratis |

**Atención a la cuarta**: recomienda decirle al agente que **pregunte**. Eso choca con lo que mi informe decía —«nunca le digas que pregunte»— y **encaja con la corrección**: preguntar funciona si se pregunta bien. El material práctico y la evidencia corregida coinciden; **mi afirmación original era la que estaba sola**.

### Las siete trampas

Aceptar código sin leerlo · *vibe coding* sin tests · sesiones eternas (pasadas 1–2 horas, compactar o limpiar) · usar siempre el modelo grande · no mirar el gasto · ignorar el fichero de reglas · **pedir cosas enormes** («“hazme la app” rara vez funciona. Trocea: modelo, endpoint, tests»).

### Y cuándo NO usar el agente

- Cosas que tardas menos en hacer tú.
- **Decisiones de arquitectura críticas**: «el agente acelera, no decide».
- Trabajar con secretos o llaves sin entorno aislado.
- Cuando aún no sabes lo que quieres: «aclara primero, prompt después».

Esa última es la más valiosa del documento, y **ninguna de mis notas la tenía**.

## Verificación del material contra la documentación oficial

El material es bueno, pero **no se puede repetir a ciegas**. Comprobado contra la [referencia de comandos oficial](https://code.claude.com/docs/en/commands):

| Comando de la guía | Qué dice la documentación oficial |
|---|---|
| `/init` | **Integrado.** «Inicializa el proyecto con una guía `CLAUDE.md`» |
| `/compact` | **Integrado.** «Libera contexto resumiendo la conversación» |
| `/clear` | **Integrado.** «Empieza una conversación nueva con contexto vacío» |
| `/cost` | **Integrado, pero es un alias** de `/usage` |
| `/review` | **Es un alias de `/code-review`, y ése está marcado como *skill*** (un prompt), no como comportamiento del programa |
| `/pr`, `/commit` | **NO aparecen en la referencia.** No se pueden confirmar como comandos integrados |

**Consecuencia práctica:** los comandos que la guía presenta como atajos estándar —`/pr`, `/commit`— **pueden no existir en tu versión**. Y la documentación completa no se pudo leer entera (la tabla se corta), así que quedan **sin verificar**, no descartados.

**Lo que sí queda confirmado y es útil:** `/init`, `/compact`, `/clear` y `/context` son integrados, y el consejo de mirar el gasto con `/cost` es válido porque el alias funciona.

## Lo que este material añade al informe

| El informe decía | El material práctico aporta |
|---|---|
| Los cinco campos de la tarea | **El ejemplo real** que los usa, y el porqué de cada línea |
| El bucle con contexto limpio | **La receta concreta** de los primeros cinco minutos |
| Las reglas deben ser pocas | **El límite práctico**: «no debe pasar de 500 líneas» |
| La verificación compra puntos | **La frase que la implanta**: «al terminar, lista qué probaste y qué falta probar» |
| La seguridad importa | **El checklist**: no pegar claves, exigir validación por adelantado, no subir `.env*` |
| *No lo tenía* | **«El agente acelera, no decide»** — la frontera de lo que no se delega |

**Y una deferencia honesta:** en una primera pasada descarté este material por «introductorio». No lo era: era **el nivel de abstracción correcto para montarlo**, que es justo lo que faltaba después de tanta literatura.

## Enlaces

- [[crear-la-tarea]] — el formato, ahora con el ejemplo práctico
- [[montaje-documentado]] — el montaje, y la receta de arranque
- [[quien-dice-que]] — la auditoría de las fuentes académicas
