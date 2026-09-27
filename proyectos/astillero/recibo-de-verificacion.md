---
title: El recibo de verificación — pieza 7b de la fábrica
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, verificacion, trazabilidad, diseno]
zona: tecnico
---

Qué queda escrito cuando el verificador termina, para que cualquiera pueda comprobar después qué se ejecutó, sobre qué, y con qué resultado **sin haber estado allí**. Es la mitad del paso 7. Diseño: [[verificador-de-tareas]] · [[la-fabrica]].

## Por qué hace falta

La regla que lo justifica viene del oficio de auditoría: **el registro tiene que permitir que un revisor competente, sin relación previa con el trabajo, entienda qué se hizo, con qué evidencia y a qué conclusión se llegó.** Si un tercero no puede reconstruir el razonamiento solo con el registro, el registro está incompleto.

Y sin él, «verificado» es otra promesa: nadie puede distinguir «se comprobó» de «alguien dijo que se comprobó».

## Formato — se apoya en un estándar que ya existe

Existe un estándar abierto para esto: **in-toto `test-result/v0.1`**, que *«define un esquema genérico para expresar el resultado de ejecutar tests en cadenas de suministro de software»*. Su sujeto está **ligado a un commit** — que es justo lo que se necesita.

Adopción medida: **174 repositorios** lo referencian, **casi todos verificadores de políticas, no productores**. Es decir: el formato existe, casi nadie lo emite. Lo usamos nosotros.

## Qué lleva el recibo

Por cada tarea verificada, un fichero —o un bloque en el comentario del PR:

```yaml
tarea: <id-del-issue>
commit_verificado: <sha exacto del cambio, no el de la rama>
base: <sha del commit sobre el que se aplicó>
resultado: PASSED | FAILED | NO_VERIFICABLE
ejecutado_por: <workflow + job, no el agente>
cuando: <fecha y hora>
tests:
  origen: restaurados desde <sha base>
  restaurados: true
  ejecutados: <n>
  pasados: <n>
intencion:
  metodo: reconstruccion-del-enunciado
  reconcilia: true | false
entorno:
  imagen: <hash de la imagen limpia>
cambios_en_tests_por_el_agente: ninguno | <detalle>
```

## Las tres reglas del recibo

1. **`NO_VERIFICABLE` es un tercer resultado**, no un fallo. Existe precedente de décadas fuera del software: en gestión de incidentes **«resuelto» y «cerrado» son distintos**; en sanidad, el estándar **FHIR** tiene estados como `preliminary` («posiblemente sin verificar») y **`unknown`** («no se sabe qué estado aplica»). Un sistema que solo sabe decir «bien» o «mal» pierde el caso más frecuente.
2. **Los códigos de estado no valen como evidencia: la salida sí.** *«Status codes are lies. Outputs are evidence.»* El recibo guarda **qué salió**, no que salió bien.
3. **El recibo no puede garantizar que llegue.** Lo dice la propia documentación de firma: las firmas garantizan que un fichero **no se manipuló**, no que **llegue**. Por eso los verificadores se diseñan para **fallar cerrado**: si el recibo falta, la tarea **no** está verificada.

## Lo que hay y lo que no

| Pieza | Estado |
|---|---|
| El formato estándar (`in-toto test-result/v0.1`) | ✅ Existe y está publicado |
| Firma de artefactos (Sigstore, atestaciones de GitHub) | ✅ Existe la maquinaria — **pero no hay modo «tests»**; habría que definir el predicado a mano |
| Un producto que emita recibos de tests firmados y ligados a commit | ❌ **No existe con adopción.** Lo más cercano que se encontró: 1 y 15 puntos en Hacker News |
| Recibo específico para código de IA | ❌ **Nada** |

**Y la crítica de fondo que hay que asumir**, de un hilo real sobre un gusano que llevaba atestaciones válidas: *«¿de qué sirve una atestación de procedencia que puede generar automáticamente un malware?»*. Es el agujero estructural: **si el entorno que produce el recibo está comprometido, el recibo es válido y mentiroso.** En un pipeline donde el agente vive dentro de ese entorno, la pregunta no es retórica.

## Enlaces

- [[verificador-de-tareas]] · [[estado-de-verificacion]] · [[la-fabrica]]
- [[gas-city-frente-a-la-fabrica]] — `NO_VERIFICABLE` está implementado en un orquestador real con la convención `75` (`EX_TEMPFAIL`): deja de justificarse solo con FHIR y gestión de incidentes
