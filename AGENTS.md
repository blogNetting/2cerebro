# AGENTS.md

Fuente única de verdad del esquema. Cumple esto en cada operación.

## Contexto del usuario

- Ingeniero. Experto en ciberseguridad, desarrollo y producto.
- Escribe al nivel de un par técnico. No expliques fundamentos, pero **explica cada sigla o término especializado la primera vez que aparece** (qué es, quién lo define, por qué importa aquí). Ejemplo del fallo: usar «DORA» sin explicarlo.
- El contenido es mayoritariamente técnico, minoritariamente temas de interés general.
- Sé directo. No te enrolles.

## Lo que menciona el usuario no condiciona nada

Regla permanente y sin excepciones:

- Cualquier cosa concreta que nombre el usuario (herramienta, producto, persona, patrón, modelo, término, cifra, forma de hacerlo) es **una pista, nunca una premisa**. No se usa como marco, no se pone en el centro, no se sube de nivel, no se compara todo contra ella y no se repite más allá de lo que aporte.
- Recibe el mismo escrutinio que cualquier alternativa y puede acabar descartada. Si la evidencia le lleva la contraria, se le dice claramente: el usuario quiere que le corrijan.
- En toda investigación, la pregunta se formula sobre el problema, no sobre lo que ha nombrado el usuario. Los criterios de evaluación salen del problema antes de mirar ninguna opción. El descubrimiento de opciones es amplio y no parte de su lista.
- Los prompts de los subagentes no llevan las ideas del usuario como contexto fijo. Si aparecen, van como hipótesis a contrastar.
- Lo que busca siempre: lo mejor de lo mejor que esté comprobado. Ni lo que él diga, ni lo más popular, ni lo último.
- Ante una pregunta amplia, trae una propuesta concreta y razonada; no le devuelvas la pregunta.
- Mantente en el problema que ha pedido. No abras frentes laterales ni propongas arrancar pilotos antes de terminar lo pedido.
- Si algo te bloquea (acceso, cuota, página que no carga), díselo en el momento: él busca cómo ayudar.

## No decir "ya está" sin haberlo comprobado de verdad

Regla permanente, por repetirse el mismo fallo dos veces:

- Antes de decir "esto ya está arreglado", abrir el resultado final y leerlo tal cual queda. No basta con haber escrito el cambio.
- Ejemplo real de hoy: dije "AGENTS.md.jinja ya no tiene secciones vacías" y seguían vacías — solo les había puesto más texto alrededor explicando que había que rellenarlas luego. Eso no es arreglarlo.
- Un comentario más largo explicando por qué algo está vacío sigue siendo un hueco vacío. No cuenta como contenido, por mucho que ocupe más líneas.
- Comprobar siempre contra lo real: el repositorio de verdad en GitHub, la versión correcta. No contra una copia local a medias. Ejemplo real de hoy: probé un cambio contra una carpeta local que todavía tenía puesto un tag viejo, y pareció que funcionaba sin funcionar.
- Si algo que dije cerrado resulta que no lo estaba, lo digo así de claro y lo arreglo en el momento — no lo disimulo ni le resto importancia.

## Toda pregunta se contesta, ninguna se salta

Regla permanente:

- Si el usuario hace una pregunta directa, se responde esa pregunta, explícita, antes de hacer cualquier otra cosa.
- Si un mensaje trae varias preguntas, se contestan todas, una por una — no se elige la fácil y se ignoran las demás.
- Si algo no se puede responder, se dice así, no se calla ni se cambia de tema.

## Qué es este repositorio

Segundo cerebro con el patrón LLM Wiki de Karpathy. Tres capas:

1. Fuentes brutas inmutables: `fuentes/`. Todo lo ingerido se guarda aquí tal cual. Nunca se modifica ni se borra.
2. Wiki mantenido por el LLM: `proyectos/`, `areas/`, `recursos/`, `archivo/`. Notas cortas enlazadas entre sí. Editable en cualquier dirección, no append-only.
3. Este esquema: `AGENTS.md`. Define cómo se ingiere, se consulta y se mantiene el wiki.

Metodología PARA. Raíz: `inbox.md` para captura sin clasificar.

## Las tres operaciones

- Ingesta: leer una fuente, extraer conocimiento, crear o actualizar notas, enlazar, actualizar índices. La fuente original queda intacta.
- Consulta: responder una pregunta usando el wiki. Índices primero, luego enlaces, luego ripgrep. Citar las notas de las que sale la respuesta.
- Lint: comprobar la salud del wiki, corregir, y sintetizar de forma proactiva.

## Criterio de clasificación automático

Decide tú la ubicación. No preguntes al usuario.

- Proyectos con fecha de fin, mayoritariamente técnicos → `proyectos/`.
- Responsabilidades continuas sin fecha de fin → `areas/`.
- Decisiones técnicas, herramientas, infraestructura, arquitectura → `proyectos/` o `areas/` según tengan o no fecha de fin.
- Temas de interés general, lecturas, material de referencia → `recursos/`.
- Lo que dejó de ser relevante → `archivo/`. No lo consultes salvo petición explícita.
- Regla de promoción: si una nota de `recursos/` resulta relevante en una consulta técnica, muévela a la zona técnica, cambia `zona` a `tecnico`, y actualiza todos sus enlaces y los índices de origen y destino.

## Reglas de enlace

Obligatorias en cada escritura.

- Antes de crear una nota, lee el `_index.md` de la carpeta destino y haz `rg` de los términos clave en todo el wiki para saber qué existe ya.
- Enlaza en ambas direcciones: pon `[[wikilinks]]` en la nota nueva hacia lo encontrado, y añade una línea en cada nota existente afectada apuntando a la nota nueva.
- Cuando llega información que afecta a una nota antigua, modifícala. Este wiki no es append-only.
- Sintaxis Obsidian: `[[nombre-de-nota]]` sin extensión.
- Ninguna nota queda huérfana. Si no encuentras nada con qué enlazarla, enlázala al menos con el `_index.md` de su carpeta.

## Índices de carpeta

- Cada carpeta tiene un `_index.md`.
- Lista cada nota de la carpeta con una línea de descripción, no solo el nombre.
- Actualiza el índice en la misma operación en que creas o renombras una nota.
- Obligatorio desde la primera nota.

## Formato de nota

Frontmatter YAML plano:

```
---
title: título legible
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
zona: tecnico
---
```

- `zona` es `tecnico` o `general`.
- Tras el frontmatter, un resumen de una línea.
- Después, el cuerpo.
- Ficheros pequeños y monotemáticos.
- Nombres de fichero descriptivos en minúsculas con guiones.
- Al editar una nota, actualiza `updated`.

## Lint

Comprueba:

- Enlaces rotos: `[[wikilink]]` que no apunta a ninguna nota existente.
- Notas huérfanas: sin enlaces entrantes ni salientes.
- Notas duplicadas: dos o más notas sobre el mismo tema que no se enlazan entre sí.
- Índices desactualizados: `_index.md` que no lista una nota existente, lista una que ya no existe, o le falta la línea de descripción.
- Frontmatter incompleto: falta algún campo obligatorio o `zona` con valor inválido.
- Ficheros versionados que no deberían estarlo: los que coinciden con `.gitignore`, cualquier ruta versionada bajo `.claude/` que no sea configuración, artefactos de herramientas (`.playwright-mcp/`, logs, capturas, cachés, binarios fuera de `fuentes/`), ficheros de más de 1 MB, y datos sensibles (credenciales, cookies, datos personales). Los comprueba el propio lint con `git`, ver `.claude/commands/lint.md`.

Corrige lo que encuentres y reporta cada corrección. Si algo ya está publicado en `origin/main`, avisa con qué se subió; no reescribas el historial sin que el usuario lo pida.

## Crecimiento

- El wiki crecerá a cientos de notas.
- Cuando una carpeta supere 30 notas, propón una subdivisión temática.
- Al aprobarse, crea el subdirectorio con su propio `_index.md` y mueve las notas, actualizando enlaces e índices.

## Registro de decisiones

- Mantén `areas/decisiones.md` con las decisiones de arquitectura y las correcciones que te haga el usuario, cada una con fecha.
- Consúltalo antes de proponer cualquier cambio estructural.

## Síntesis proactiva

Al ejecutar lint, además de corregir:

- Busca temas recurrentes que aparecen en varias notas sin tener nota propia. Crea la nota de síntesis en `areas/` y enlázala a las notas donde aparece el tema.
- Busca contradicciones entre notas. Escribe una nota en `areas/` señalando la contradicción y enlazándola a las notas implicadas.
- No esperes a que te lo pidan.

## Reutilización de skills

- Antes de resolver algo, comprobar si ya existe una skill que lo haga y usarla.
- Antes de crear una skill, buscar si existe algo ya hecho y mantenido que lo haga mejor. Preferir herramienta existente a script improvisado.
- Cada skill lleva en su cabecera: fecha de creación, fecha de última revisión, y qué alternativa se descartó al crearla y por qué.
- En la revisión periódica (durante el lint): comprobar si la skill sigue funcionando, si ha aparecido algo mejor, y sustituirla o retirarla, anotándolo en `areas/decisiones.md`.
- Búsquedas y consultas en internet: usar `/investigar-web` (WebSearch → WebFetch) y, cuando eso falle de verdad (403, CAPTCHA, HTML sin el contenido), escalar a `/navegador-cdp` (Chrome real vía CDP) es obligatorio: un bloqueo no es un resultado ni un motivo para entregar «no verificado». Solo una prohibición explícita del usuario suspende el escalado; una duda o preferencia suya no lo es. Ver `areas/entorno.md` para las herramientas ya montadas en esta máquina.
- Compartir contexto entre sesiones (VSCode y CLI no comparten historial): usar `/exportar-sesion` y `/importar-sesion`. No hace falta escribir el comando — si el usuario dice "exporta esta conversación", "vuelca el chat" o similar, usar la skill directamente. Ver `areas/entorno.md`.
- Una exportación solo se hace cuando el usuario lo dice. Sirve para pasar ese contenido a una sesión nueva y nada más: no obliga a repetirla, ni a mantenerla actualizada, ni a tocar el wiki. Nunca exportar por iniciativa propia. Las exportaciones van a `.claude/sesiones/`, transitoria e ignorada por git: no es contenido del wiki ni resultado de trabajo.

## Repositorio y artefactos

- `.claude/` es solo configuración de Claude Code: commands, agents, skills, hooks y settings. Nunca contenido ni resultados de trabajo. Los resultados van a `proyectos/`, `areas/`, `recursos/` o `archivo/` según PARA; los adjuntos binarios de terceros (fotos, PDF), a `fuentes/`.
- El repo es público y `cerebro-sync.sh` (cron horario) hace `git add -A` y push: todo lo que no esté en `.gitignore` se publica. Por eso, todo directorio de caché, log, captura o artefacto de una herramienta (`.playwright-mcp/`, salidas temporales, volcados) se añade a `.gitignore` en el mismo momento en que aparece, sin esperar a que el usuario lo vea. Mejor aún: configurar la herramienta para que escriba fuera del repo.
- Antes de crear un directorio de salida nuevo, comprueba con `git check-ignore` que queda ignorado. Antes de borrar una carpeta, `git ls-files <ruta>`: puede estar versionada o ser de otra sesión.
- No versionar datos personales, sesiones de navegador, cookies, credenciales ni capturas con datos de cuentas. Fotos y material de terceros, fuera de git.

## Formato de investigaciones

Aplica a toda investigación (comparativas, búsquedas de precios, evaluación de herramientas, análisis de opciones). Permanente: el usuario no tiene que repetirlo.

Estructura obligatoria del resultado, en este orden:

1. Introducción: qué se pregunta y por qué.
2. Considerado y descartado, con el motivo de cada descarte.
3. Análisis detallado. Tablas, gráficos, diagramas o imágenes siempre que ayuden; prioriza lo visual sobre el texto corrido.
4. Recomendaciones.
5. Dónde se ha buscado: fuentes consultadas, incluidas las que no aportaron nada.
6. Lo relevante que no encaje en lo anterior.

Enlaces (el fallo más frecuente, tratarlo como requisito duro):

- Cada afirmación que venga de una fuente lleva su enlace en la misma línea o celda donde aparece: dato, precio, condición, cita. No basta un bloque de enlaces al final.
- Cada opción, producto u oferta citada lleva su URL directa, con las fechas o parámetros de la búsqueda puestos cuando la web lo permita.
- Los descartes y las fuentes que no aportaron también se enlazan.
- Dato sin fuente enlazable: se marca como «sin verificar» o se omite. No se afirma sin enlace.
- Antes de entregar, repasa el documento y comprueba que ninguna afirmación con fuente queda sin enlace.

Formato de salida:

- `.md` por defecto. Es una nota del wiki: se clasifica según PARA y cumple las reglas de enlace, frontmatter e índices.
- PDF solo si el usuario lo pide explícitamente. Se genera a partir del `.md`, que se conserva. Antes de crearlo, `git check-ignore` sobre la ruta de salida (el repo es público).

## Qué no hacer

- No exportar sesiones por iniciativa propia.
- No escribir contenido ni resultados en `.claude/`.
- No borrar ni modificar nada de `fuentes/`.
- No inventar citas ni referencias.
- No crear estructura de carpetas nueva sin pedirla.
- No escribir ficheros largos cuando puedes escribir varios cortos.
- No preguntar al usuario dónde va cada cosa.
