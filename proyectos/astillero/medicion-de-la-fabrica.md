---
title: La medición — pieza 12 de la fábrica
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, medicion, metricas, diseno]
zona: tecnico
---

Cómo se sabe si la fábrica funciona o solo va rápido. **Es el segundo agujero de la fábrica y el que nadie ha resuelto**: no existe ningún estándar consensuado de métricas para agentes. Diseño: [[la-fabrica]] · [[vigilante-de-tareas]].

## El punto de partida, y no es bueno

- **NO existe un estándar de métricas de agentes publicado por ningún organismo.** Y lo dice el propio DORA 2025, literal: *«estandarizar ahora sería prematuro»*.
- **Lo que hay son propuestas de vendor en competencia** (DX, Faros, Salesforce, GitHub), cada una con su taxonomía, sin adopción medida.
- **Y el sector mide mal por defecto**: el 94 % de **700 encuestados** (entre practicantes y mandos) dice que la deuda técnica, el tiempo de validación y el desgaste **no están** en sus métricas.

## Las cinco de DORA — y ojo, son cinco

El modelo vigente tiene **cinco**, en dos grupos. **La quinta se añadió precisamente por esto**: porque la IA acelera el envío y **esconde el retrabajo**.

**Caudal:** tiempo hasta producción · frecuencia de despliegue · **tiempo de recuperación de un despliegue fallido** (el antiguo «tiempo medio de reparación», renombrado).
**Inestabilidad:** **tasa de fallo del cambio** · **tasa de retrabajo del despliegue** ← la añadida.

**Dato de DORA 2025 (~5.000 encuestados):** *«la adopción de IA mejora el caudal de entrega... pero sigue aumentando la inestabilidad»*. Y **el 30 % declara poca o ninguna confianza en el código que genera la IA.**

**Y un contrapeso que hay que decir:** Faros, con telemetría de 22.000 desarrolladores, **contradice a DORA frontalmente** — en su muestra, incluso las organizaciones de alto rendimiento muestran el mismo deterioro. DORA mide con **encuesta autoinformada**; Faros, con **telemetría medida**. **La telemetría debería pesar más.** Por eso no se adopta DORA a secas.

## Las que se añaden, y por qué esas

| Métrica | Qué mide | Cómo se instrumenta |
|---|---|---|
| **Coste por cambio aceptado** | Tokens ÷ cambios aceptados. **Es lo que DORA y Faros recomiendan en lugar del token crudo** | Tokens de la traza ÷ PRs fusionados |
| **Tasa de intervención humana** | Cuántas veces tiene que aparecer una persona | **Ya viene de fábrica**: Claude Code emite `claude_code.tool.blocked_on_user` por OpenTelemetry |
| **Defectos escapados** | De los bugs totales, cuántos aparecieron en producción | Fórmula de oficio: `escapados ÷ (escapados + atrapados)`. Con issues etiquetados `bug` + `production`, enlazados al cambio que los causó |
| **Tasa de reversión** | Cambios fusionados que se deshacen | PRs cuyo título empieza por `Revert "`. Referencia medida: las de Codex se revierten **6,1 %** frente al **11,5 %** humano, sobre 37.623 PRs |

**Lo que NO se mide: el consumo bruto de tokens.** DORA 2025 lo llama **métrica de vanidad** y caso de libro de la ley de Goodhart: cuando mides el gasto, optimizas el gasto.

## Instrumentación — todo con lo que ya tenéis

**Ninguna herramienta de pago hace falta para arrancar.** Lo verificado:

| Herramienta | Veredicto |
|---|---|
| **GitHub Insights** | Gratis y nativo. **Pero no mide DORA, ni coste, ni defectos** — mide actividad, no entrega |
| **LinearB** | 29 $/usuario/mes, **mínimo 50 desarrolladores.** Descartada por mínimo |
| **Swarmia** | 9-23 $/dev/mes **por módulo** — los tres juntos, **50 $/dev/mes**. Descartada por precio |
| **CodeScene** | 18-27 €/autor activo/mes. Mide **salud del código, no entrega** |
| **DX, Faros, Jellyfish, Cortex** | **No publican precio.** No evaluables sin hablar con ventas |

**La receta:** GitHub Insights + un job propio con `gh api` + las cinco de DORA + la telemetría nativa de Claude Code. **Cubre el 80 % del valor a coste cero.**

**Un aviso sobre el estándar de instrumentación:** las convenciones de IA de **OpenTelemetry están en estado `Development`, no estables**. Quien monte encima, monta sobre algo que puede cambiar.

## Lo que hay que asumir

- **Los cuatro números son propuesta, no estándar.** «Defectos escapados» es vocabulario de oficio, **no lo publica ningún organismo**, y en Hacker News hay **diez resultados y ninguno relevante**: es vocabulario de manual, con **cero discusión comunitaria**. Se usa, pero se dice que es propuesta.
- **El desacuerdo DORA/Faros no está resuelto.** Si Faros tiene razón, **el caudal ya no es una señal válida** porque la IA lo sube y a la vez sube la inestabilidad. La postura de este diseño: se miden las cinco **y** el coste por cambio aceptado, y se vigila la **reversión** como la señal que no depende de ninguna encuesta.
- **Sin la tasa de intervención humana, esto no se puede juzgar**: es la que dice si la fábrica es autónoma o eres tú haciéndola funcionar.

## Lo que reveló construirla y correrla (2026-09-27)

Se ejecutó de verdad sobre un proyecto generado con `copier`, y escribió `metricas/medicion.json`, `metricas/historial.jsonl` y `metricas/ultima.md` en una rama de datos. Tres cosas que solo se ven ejecutando:

- **Los números que no puede calcular salen «no disponible» con su motivo, y eso es el comportamiento correcto.** Sin PRs fusionados en la ventana, la tasa de reversión **no es 0, es indefinida** — y así lo dice. El coste por cambio sale «no disponible» porque no hay colector de OpenTelemetry, y explica por qué: los tokens los emite Claude Code **en la máquina donde corre el CLI**, no en el repo. Un 0 en esa casilla habría sido una mentira con formato de dato.
- **Funcionó sobre datos reales a la primera**: contó la tarea que el vigilante había bloqueado de verdad en ese mismo repo (1 con `estado:bloqueado`).
- **Defecto conocido, dicho y no escondido:** la rama de datos se crea desde el checkout del proyecto, así que **arrastra una copia del árbol de código** del día en que corrió. Funciona y es estable, pero quien mire esa rama verá código viejo. Lo correcto sería una rama huérfana que solo llevara `metricas/`; cambiarlo obliga a volver a probarlo, así que queda anotado.

**Sin verificar todavía:** el disparo por `schedule` (todas las corridas han sido manuales) y la consulta PromQL contra un colector real, que no existe.

## Enlaces

- [[la-fabrica]] · [[vigilante-de-tareas]] · [[desarrollo-autonomo-con-agentes]]
- [[astillero-bitacora]] — estado del trabajo y lo que falta
- [[gas-city-frente-a-la-fabrica]] — un quinto caso de «nadie publica métricas de resultado»: el lanzamiento de Gas City v1.0, 4.613 palabras y cero cifras de calidad
