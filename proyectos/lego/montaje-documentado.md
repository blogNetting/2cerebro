---
title: Lego — el montaje, documentado
created: 2026-09-28
updated: 2026-09-28
tags: [lego, montaje, diseno, agentes]
zona: tecnico
---

Cómo se montaría, pieza a pieza y comando a comando, para que quede escrito y sea reproducible el día que se decida hacerlo.

> **Esto no está montado ni probado.** Es el diseño que se deduce de la evidencia recogida. Ningún comando de aquí se ha ejecutado, y ninguno de los resultados que promete está verificado en la práctica. Se escribe para que sea reproducible, no para afirmar que funciona.

## Las piezas

**Tres, y dos ya están.**

| Pieza | Para qué | Ya la tienes |
|---|---|---|
| `claude` (CLI) | El agente | Sí |
| `jq` | Leer y escribir el estado en JSON desde el bucle | No — `apt install jq` |
| `git` | La cola, el historial y el aislamiento | Sí |

Opcionales, y sólo si hacen falta: `cron` (ya viene con el sistema) para lanzarlo solo; `docker` o `bubblewrap` para el sandbox.

## Los ficheros

```
proyecto/
├── tareas/
│   ├── 001-anadir-buscador.md      ← la tarea, en el formato de [[crear-la-tarea]]
│   ├── 002-arreglar-login.md
│   └── hechas/                     ← se mueven aquí al terminar
├── estado.json                     ← qué tarea está en curso y su resultado
├── progreso.txt                    ← registro solo de añadir, lo que aprendió cada vuelta
└── CLAUDE.md                       ← las reglas del proyecto (pocas: ver más abajo)
```

**Por qué la tarea es un fichero con nombre numerado:** el orden alfabético *es* el orden de ejecución. No hace falta ninguna cola, ningún servidor y ninguna base de datos — git ya versiona el estado de cada tarea y deja rastro de quién cambió qué.

## La tarea, con lo mínimo

Con lo que dice [[crear-la-tarea]], una tarea son **cinco campos y nada más**:

```markdown
# Añadir buscador al listado de facturas

## Objetivo
Que el listado de facturas tenga un campo de búsqueda por texto que filtre en el cliente.

## Alcance
- Permitido: `src/facturas/**`, `tests/facturas/**`
- Prohibido: tocar `src/auth/**`, cambiar el esquema de la base de datos, añadir dependencias

## Criterios de aceptación
1. CUANDO el usuario escribe en el buscador ENTONCES el listado muestra solo las facturas cuyo concepto contiene ese texto.
2. CUANDO el buscador está vacío ENTONCES el listado muestra todas las facturas.
3. SI no hay coincidencias ENTONCES se muestra el texto «Sin resultados».

## Comprobación
    npm test -- --run tests/facturas/buscador.test.ts
Debe salir en verde y sin avisos nuevos de `npm run lint`.

## Qué devolver
Un resumen de tres líneas: qué cambió, qué se rompió, y qué quedó sin hacer.
```

Cinco campos: objetivo, alcance, criterios, comprobación, retorno. Ni plan, ni diseño, ni documento de investigación. Si la tarea es grande, el plan se separa; si no, no.

## El bucle

Un script de bash, sin más. Ejecuta **una tarea por instancia**, con contexto limpio cada vez, siguiendo lo que dice [[consumir-la-tarea]]:

```bash
#!/usr/bin/env bash
# lego.sh — NO PROBADO. Documentación del diseño.
set -euo pipefail

for tarea in tareas/[0-9]*.md; do
  [ -e "$tarea" ] || continue
  nombre=$(basename "$tarea" .md)
  echo "── $nombre ──────────────────────────"

  # Una instancia nueva por tarea. Contexto limpio.
  claude -p "$(cat "$tarea")" \
    --bare \
    --allowedTools "Read,Edit,Write,Bash(npm test:*),Bash(npm run lint:*),Bash(git diff:*)" \
    --permission-prompts none \
    --max-turns 40 \
    --output-format json > "salida-$nombre.json"

  # Se decide con la comprobación, no con lo que diga el agente.
  if npm test --silent > /dev/null 2>&1; then
    git add -A && git commit -m "lego: $nombre"
    mv "$tarea" tareas/hechas/
    echo "OK" >> progreso.txt
  else
    fallos=$(( ${fallos:-0} + 1 ))
    echo "FALLO ($fallos) : $nombre" | tee -a progreso.txt
    git checkout -- .    # se deja el árbol limpio para la siguiente
    if [ "$fallos" -ge 2 ]; then
      echo "needs_human: $nombre" >> progreso.txt
      break              # dos intentos y se para: no quemar cuota en bucle
    fi
  fi
done
```

**El tope de dos intentos no es arbitrario: lo usan cuatro implementaciones independientes.** El razonamiento, copiado de una de ellas: dos intentos dan «exactamente un ciclo de “inténtalo otra vez” tras el primer fallo: margen de sobra para un “se me olvidó el `git add`”, y ningún margen para que un agente queme en silencio el tiempo máximo de la sesión». Además, el detector de atascos de OpenHands —que viene activado por defecto— para ante *misma acción y misma observación repetida 4+ veces*: eso detecta el bucle que un tope de tiempo deja pasar. Fuentes en [[robustez-desatendida]].

Cuatro decisiones, y cada una tiene su motivo en la evidencia:

- **`--bare`** — sin esto, una sesión `-p` ejecuta los ganchos del `.claude/settings.json` del proyecto **incluso en una carpeta en la que nunca has confiado**. Es un aviso de seguridad de la documentación oficial.
- **`--allowedTools` con la lista cerrada** — en modo `-p` el agente nunca pregunta permisos. Lo que no esté en la lista, no lo hace. Aquí está la única barrera real contra tocar lo que no debe.
- **`--max-turns`** — tope de seguridad. Sin él, un bucle descontrolado es el fallo caro.
- **Se decide con `npm test`, no con la salida del agente** — es la regla que sale de [[verificacion-y-oraculo]]: el agente que ve el oráculo lo puede romper. Aquí el oráculo es el runner de tests, que el agente no edita (los tests ocultos van fuera de su alcance).

## Cómo se engancha la verificación

Es la parte que decide si el montaje sirve de algo. Tres piezas, y ninguna es opcional:

1. **Tests visibles** — los que el agente lee y usa para trabajar. Están en su alcance permitido.
2. **Tests ocultos** — los que deciden. **Fuera del alcance permitido**, en una carpeta que el `--allowedTools` no deja tocar. Corren después, en el script, y son los que dan el veredicto.
3. **El evaluador, aislado** — el script que decide **no** corre dentro del mismo proceso que el agente. Es el patrón nº 1 de los siete que documenta Berkeley, y el que permitió a un agente sacar 500 sobre 500 sin resolver nada, reescribiendo los resultados de los tests.

Para el **caso web**, el criterio de aceptación cambia de forma: en vez de «salga en verde», es un fichero de especificación ejecutable con `exspec` o Playwright, más aserciones sobre **geometría y estado medidos** (no sobre propiedades CSS declaradas) y un escaneo de accesibilidad. Las tres señales fallan en direcciones distintas, y por eso van las tres.

## El disparador

**A mano, para empezar.** Lanzas `./lego.sh` y lo miras. Cero piezas nuevas.

**Con cron, cuando ya fíes.** Una línea en el `crontab`, cogiendo los minutos raros para no coincidir con todo el mundo:

```
23 */2 * * * cd /ruta/proyecto && ./lego.sh >> .lego.log 2>&1
```

**Con un detector de colgado**, porque es la pieza que nadie cuenta y la que más duele: el script debe escribir una marca de tiempo al empezar cada tarea, y una comprobación externa —otro cron, un fichero centinela— debe avisar si esa marca no avanza. Un agente colgado **no deja registro de error**: hay que detectarlo por fuera, igual que el desarrollador que aprendió a juzgar por las marcas de tiempo de los ficheros y el tiempo de CPU.

## El sandbox

Si el bucle va a correr sin nadie delante y con acceso a la red, esto no es opcional. Con lo que dice [[consumir-la-tarea]] sobre inyección de prompt:

- **Cortar la salida de red** salvo a lo imprescindible. Es lo que evita la exfiltración, y es una de las tres patas de la «trifecta letal» — quitando una, se rompe.
- **Bloquear la escritura fuera del directorio de trabajo** — evita la persistencia y el escape.
- **Credenciales mínimas, inyectadas al momento**, nunca todas las variables de entorno del sistema.
- **Contenedor con virtualización** si de verdad importa (Kata, gVisor), en lugar de confiar en el aislamiento del propio agente.

La forma más barata de empezar: **una cuenta de usuario del sistema dedicada, sin acceso a tus ficheros personales ni a tus claves**, y el proyecto dentro de su propio directorio. No es un sandbox fuerte, pero es infinitamente mejor que ejecutar como tú.

## Las reglas del proyecto (`CLAUDE.md`), pocas

Aquí es donde la mayoría de los montajes se estropea. **El motivo original que yo daba —«P(todas) ≈ p^n»— está corregido y atenuado** (ver [[crear-la-tarea]]): el paper que lo sostenía fue **rechazado**, y la corroboración revisada por pares muestra que los mejores modelos aguantan **más de 150 instrucciones** antes de degradarse. Lo que queda en pie, y basta para la recomendación: **cada regla añadida es una cosa más que comprobar**, y un fichero de reglas que nadie puede hacer cumplir no es una restricción, es una petición.

Lo que de verdad hace falta, y nada más:

```markdown
# Reglas

- Los tests no se editan nunca. Si un test falla, se arregla el código.
- Los tests ocultos están fuera de tu alcance. No los busques.
- Ante una duda que no puedas resolver: decide, implementa, y escribe la decisión
  en la sección «Desviaciones» de tu informe. No te pares a preguntar.
- Si falta un fichero o un dato de entrada que la tarea menciona, escribe BLOCKED
  en el informe y termina SIN tocar el código.
```

Cuatro reglas, y **ninguna sustituye a un permiso**. Todo lo que se pueda convertir en un `--allowedTools` o en un test, se convierte. Lo que quede en prosa es una petición, no una restricción.

**La tercera regla no es un capricho de estilo, es un resultado medido.** «Pregunta si dudas» **no funciona**: dar al agente la opción de preguntar hunde su rendimiento —Gemini 3.1 Pro pasa del 84,7% al 5,3%; GPT-5.3-Codex del 67,3% al 2,0%— y sin canal humano la pregunta se convierte en una salida que el sistema interpreta como tarea completada, avanzando **sin haber hecho nada**. Por eso: decide y documenta. Detalle y fuentes en [[robustez-desatendida]].

## Qué se espera de esto, y qué no

**Lo que este montaje sí da:** ejecución desatendida de tareas pequeñas y bien especificadas, con veredicto automático, sin cola, sin servidor y sin base de datos. Tres piezas, de las cuales dos ya están instaladas.

**Lo que no da:** autonomía sobre trabajo grande, ambiguo o sin oráculo automático. Y **el límite que ninguna pieza cubre** está en el horizonte de la realimentación: los tests dan señal **en segundos**, pero el coste de una mala arquitectura se mide **en semanas o meses** — así que hay una parte de la calidad que hoy no se puede cerrar automáticamente. Ver [[limites-del-andamiaje]].

## Enlaces

- [[crear-la-tarea]] — el formato de los ficheros de `tareas/`
- [[consumir-la-tarea]] — el bucle y el aislamiento, con su evidencia
- [[verificacion-y-oraculo]] — de dónde salen las tres piezas de verificación
- [[piezas-y-coste]] — el recuento y las alternativas descartadas
- [[investigacion-lego]] — el informe completo
