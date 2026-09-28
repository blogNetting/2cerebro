---
title: Lego — las plataformas, auditadas una por una
created: 2026-09-28
updated: 2026-09-28
tags: [lego, plataformas, auditoria, mecanismos]
zona: tecnico
---

Cinco plataformas grandes, leídas por dentro: README, su documentación técnica entera, sus incidencias, su npm, su Homebrew, sus advisories de seguridad. **El objetivo era sacar los mecanismos que no estaban en la lista** — y de paso ha salido algo más importante.

---

# PARTE 1 · Dos correcciones a lo que yo te dije

## 1.1 · «La más grande» no es la que te dije

Te dije que `oh-my-claudecode` (39.393★) era la mayor. **Falso.** Existe **[`stablyai/orca`](https://github.com/stablyai/orca) con 80.605★** — MIT, creado en marzo de 2026. Y hay un catálogo independiente, [agentmgmt.dev](https://agentmgmt.dev/), que lista **60 productos** de esta categoría.

**Para escala:** Orca 80.605 · Warp 65.000 · Herdr 41.000 · oh-my-claudecode 39.393 · Vibe Kanban 28.000 · cmux 27.000.

**Me equivoqué porque comparé cinco nombres que ya tenía, no el catálogo.** Es el mismo error de siempre: mirar donde ya miré.

## 1.2 · Conté mal las incidencias, en las cinco

Yo contaba `open_issues_count` de GitHub. **Ese campo suma incidencias *y* peticiones de cambio.** Los números reales:

| Plataforma | Lo que dije | Incidencias reales | Peticiones de cambio |
|---|---|---|---|
| Superset | 803 | **390** | 413 |
| Paseo | 961 | **558** | 403 |
| DeepCode | 17 | **10** | 7 |
| edict | 13 | **10** | 3 |

---

# PARTE 2 · Lo que la auditoría encontró, y no es agradable

## 2.1 · `oh-my-claudecode` — el repo cobra peaje en estrellas para atenderte

Es el hallazgo más grave de los cinco. Cita literal de [su incidencia #2234](https://github.com/Yeachan-Heo/oh-my-claudecode/issues/2234):

> «**Filtramos las incidencias externas que no son errores por la política de estrellas primero. Por favor, dale estrella a al menos uno de los repos principales y luego reábrela** si aún quieres que la revisemos. La cierro por ahora.»

Y ante la queja: *«La política es intencionada. **El ancho de banda de revisión es finito**, y las incidencias cerradas no son un vertedero de quejas de paso.»*

El desafío formal —*«la política de estrellas va contra la política de uso aceptable de GitHub»*— se creó y **se cerró en 41 segundos**. Y salió del repositorio: [r/github, 374 puntos y 53 comentarios](https://reddit.com/r/github/comments/1secbg7/gatekeeping_fixes_improvements_through_stars/).

**Por esto no puedo usar sus estrellas como señal de nada.** Y hay más:

- **Bucles de quema de tokens**, en su propio rastreador: [*«el gancho de parada `persistent-mode` causa un **bucle infinito de quema de tokens**»*](https://github.com/Yeachan-Heo/oh-my-claudecode/issues/2652) — *«una sola sesión consumió un estimado de **4 a 5 veces más tokens** de lo necesario»*.
- **Un solo autor**: 3.110 commits frente a 71 del siguiente. La «incidencia más comentada» tiene **87 de 88 comentarios del propio mantenedor** — es un monólogo.
- **La seguridad viene apagada**: sin la variable `OMC_SECURITY`, *«aplican los valores por defecto de cada función (**todos desactivados**)»*.
- **256 versiones en npm en 8 meses**, y una incidencia documenta que **se congeló** con Claude Code 2.1.27+.

**Su dato bueno, y es real:** **16.587 descargas al mes** en npm. El único con uso medible en esta categoría.

## 2.2 · Paseo — un fallo crítico de seguridad con el mantenedor sin responder

Paseo tiene **42.348 descargas/mes** del CLI y **570 instalaciones por Homebrew en 30 días** (puesto 267 de 9.624). Para comparar: `codex` tiene 84.023 y `claude-code` 28.454. **Es el 0,05%.**

Y tiene esto: un aviso de seguridad **crítico**, `GHSA-2xhx-c69r-qj4j`, publicado el 23 de agosto de 2026:

> «**Salto de autenticación en el extremo a extremo del relay mediante una clave pública Curve25519 de orden bajo, que permite el control remoto del demonio.**»

Se reportó en privado el **10 de agosto**. **El 17 de agosto seguía sin acuse de recibo** — siete días con el fallo vivo. Y el propio README lo explica: *«**Paseo lo construye una sola persona** y lo financian quienes lo usan.»*

Además: **tres incidencias `p0` de seguridad abiertas desde abril** (CORS abierto, sin protección de repetición, claves en claro en el almacenamiento del móvil), **542 errores abiertos** —el máximo histórico—, y sesiones de agente que *«desaparecen permanentemente y no se pueden volver a importar»*.

**Y una contradicción sobre la cuota que te afecta directamente.** Su documentación dice que Claude Code «cuenta contra los límites normales de tu plan». Pero el mantenedor dijo lo contrario en Hacker News: *«consumirá **un fondo de créditos distinto**… en la práctica podrás usar sólo una fracción de tu cuota dentro de Paseo».* Y él mismo, después: *「parece que la decisión se revirtió, al menos por ahora」*. **La doc describe un estado que a punto estuvo de no serlo.**

## 2.3 · Superset — no verifica los resultados, y lo dice en su documentación

Cita literal de [su documentación](https://docs.superset.sh/automations):

> «**Sin seguimiento del resultado del agente**: una ejecución correcta significa que se creó su espacio de trabajo; la página de detalle **no muestra si el trabajo del agente tuvo éxito**.»

**Y el «100+ agentes en paralelo» del titular no lo sostiene nadie.** Su propio fundador: *«para trabajo real **sólo puedo llevar 2 o 3 agentes en paralelo**».*

Un usuario pesado se fue, y el fundador lo confirmó: *«usé Superset durante bastante tiempo hasta hace un mes. Había problemas molestos, congelaciones y el terminal no se renderizaba bien… **lo dejé**»* → *«lo siento. **La verdad es que la fastidiamos con el rendimiento**».*

**Y una mentira de terceros que hay que corregir:** un sitio de análisis afirmaba que Superset «tiene licencia MIT». **Es falso: es Elastic License 2.0**, que no es código abierto según la OSI y prohíbe ofrecerlo como servicio. Lo verifiqué en su fichero de licencia.

## 2.4 · DeepCode — cero evidencia de uso real

**92 incidencias en 16 meses.** Sus dos únicos envíos a Hacker News fueron de agosto de 2025, **anteriores al paper y a la versión 2**: 5 comentarios en total entre los dos, **ninguno contando una experiencia de uso**. En Reddit: seis entradas, **todas en subreddits agregadores**, con 0 a 3 comentarios.

**Y el paper describe un producto distinto del repositorio actual.** El módulo «CodeMem» que describe **da cero resultados en el código de hoy**.

**Sus cierres masivos son por obsolescencia, no por arreglo.** 24 de 82 en un solo día, con incidencias de hasta 374 días. Cita del mantenedor: *«Las cierro por ese motivo, **no porque hayamos verificado el arreglo concreto**».*

## 2.5 · edict — el proyecto está muerto por dentro, y lo admite

- **Último commit en la rama principal: 6 de mayo de 2026.** Cinco meses.
- **La integración continua está rota desde el 30 de marzo**: 54 fallos frente a 43 éxitos en 100 ejecuciones. **Lo único que corre hoy es el bot que cierra incidencias viejas.**
- **Cero publicaciones y cero etiquetas de versión, nunca.**
- **Sus ejemplos son inventados y se contradicen.** El fichero dice que son *«recopilados de registros de ejecución reales»* — pero están fechados el 20 y 21 de febrero, y **el repositorio se creó el 23 de febrero**. No pueden ser registros reales.
- **El flujo central no funciona, y el propio repositorio lo admite:** *«muchos usuarios reportan que el flujo no completa el trayecto completo»*.
- **Dos de los guiones de los que presume el README son enlaces simbólicos roto**s que apuntan a **`/Users/bingsen/…`, el portátil del autor**.
- **Acusación de plagio sin resolver** en su incidencia más activa (186 comentarios), y la curva de estrellas va de **321 a 6.642 en seis días**, coincidiendo con la polémica.

---

# PARTE 3 · Los mecanismos que SÍ son nuevos

Ocho piezas que **no están en mi lista y no son importaciones obvias**:

| # | Mecanismo | Qué resuelve |
|---|---|---|
| **1** | **Descriptor sellado con SHA-256** | El grafo del flujo se sella; cualquier deriva lanza un error en vez de recalcularse en silencio |
| **2** | **Fencing por epoch de turno** | Cada reclamación lleva un epoch monótono: *«un trabajador obsoleto no puede liberar ni completar la reclamación de su sucesor»* |
| **3** | **Resultado desconocido en vez de repetición** | *«Los efectos secundarios interrumpidos sin recibo concluyente devuelven un **resultado desconocido en vez de repetirse**»* |
| **4** | **Cerca de espacio de trabajo canónico** | *«Los hilos que comparten el mismo checkout canónico se **serializan**»* — resuelve lo que el worktree no resuelve |
| **5** | **Evidencia con caducidad de 5 minutos** | *«La evidencia debe ser reciente (dentro de 5 minutos) e incluir la salida real del comando»* |
| **6** | **El review fallido se convierte en trabajo nuevo** | Si el code-review no está limpio, no se marca completado: **añade una historia de bloqueo** y mantiene el objetivo activo |
| **7** | **Medidores de cuota leídos del proveedor** ⭐ | Ventanas de 5 horas y semanales con porcentaje y hora de reinicio. **Es la única pieza de todo el barrido que sabe cuánto te queda de plan** |
| **8** | **La cuota como variable no controlada** | Ninguna de las cinco controla el riesgo de que el proveedor cambie cómo se consume tu suscripción |

**El 7 y el 8 son los que más te afectan**, porque tu montaje depende de una suscripción de 20 € con ventanas de uso.

**Y hay tres más que merecen mención:**

- **Verificación escalada por riesgo**: LIGHT / STANDARD / THOROUGH = **1x / 5x / 20x** de coste, elegida por el tamaño del cambio. Ahorro estimado del 40%.
- **Mapa de tipo-de-afirmación → tipo-de-evidencia exigido**: «arreglado» → un test que pasa; «implementado» → diagnósticos limpios y compilación; «depurado» → causa raíz con fichero y línea.
- **Dos ejes de revisión no promediables**: estándares del repositorio frente a especificación de la tarea, *«informados por separado, **nunca fusionados**; la tarea falla si falla cualquiera de los dos»*.

---

# PARTE 4 · El mejor caso contra todo esto

**Y es fuerte, así que lo pongo sin matizar:**

**1. Casi nada de esto es invención: es importación de sistemas distribuidos.** Los *fencing tokens*, los arriendos con prueba de muerte por bloqueo de fichero, el *outbox* transaccional, la entrega «al menos una vez», el copy-on-write, la atenuación de capacidades, el compare-and-swap y las claves de idempotencia **existen desde hace décadas en bases de datos y colas de mensajes**. Lo nuevo es **aplicarlo a agentes**, no los mecanismos.

**2. En dos de las cinco, los «mecanismos» son documentación, no código.** En edict, cuatro de los siete que anuncia su README **sólo existen en el README**, dos son enlaces roto y el flujo está roto desde marzo. En Superset, **la verificación no existe y su propio equipo lo escribe**.

**3. Y el hallazgo transversal del frente anterior se confirma aquí: la comunidad no ha mirado casi nada de esto.** Ni un análisis independiente en blogs de OMC, DeepCode o edict, por siete vías de búsqueda.

**Lo que aguanta:** las **ocho piezas** de la Parte 3 no están en mi lista ni son importaciones obvias. Y que **nadie publica tasa de éxito**.

---

# PARTE 5 · Dónde se ha buscado, y los huecos

**Leído directamente:** los cinco README completos · de OMC: `ARCHITECTURE.md`, `TEAM-WORKTREE-MODE.md`, los niveles de verificación, ADR 03570, `HOOKS.md` · de DeepCode: la arquitectura de ejecución y la base de seguridad · de Paseo: arquitectura, modelo de datos, ciclo de vida, permisos · de Superset: el modelo, orquestación, tareas, uso, automatizaciones y seis recetas · más cinco árboles de repositorio, dos licencias, la API de GitHub, Hacker News, Docker Hub, npm, Homebrew y el catálogo de 60 productos.

**Huecos declarados, no ausencias probadas:**

1. **No encontré ni un análisis independiente en blogs** de OMC, DeepCode ni edict, por siete vías de búsqueda. **No puedo afirmar que no existan.**
2. **OMC es coreano y edict es chino: cobertura incompleta.** No cubrí velog, Brunch, V2EX ni 知乎 — **bloqueados por login**. Para edict esto es un hueco serio.
3. **Los tres Discord no son legibles sin cuenta**, y es probablemente donde vive la actividad real.
4. **No pude verificar ni descartar el inflado de estrellas de edict**: la API devuelve 401.

## Enlaces

- [[mecanismos-nuevos]] — la lista consolidada de todo lo que no estaba en el mapa
- [[las-piezas]] — el mapa original
- [[quien-dice-que]] — la auditoría de fuentes académicas
- [[etapas]] — el veredicto por etapas
