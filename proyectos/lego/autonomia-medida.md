---
title: Lego — cuánta autonomía hay, medida
created: 2026-09-28
updated: 2026-09-28
tags: [lego, autonomia, benchmarks, medicion]
zona: tecnico
---

Cifras con denominador, no impresiones. Y un resultado que decide el diseño: **el andamiaje pesa más que el modelo**, así que la arquitectura no es un detalle de fontanería — es la palanca.

## El hallazgo que justifica todo el proyecto

> «Los rangos observados dentro del mismo modelo al cambiar sólo el andamiaje alcanzan **29,8 puntos porcentuales**.» — [Liu et al., ADMA 2026, arXiv:2609.17394](https://arxiv.org/abs/2609.17394), revisado por pares

Puesto en contexto: **todo el top-30 del ranking de SWE-bench cabe en 8,8 puntos**. Es decir, **cambiar cómo montas el sistema mueve el resultado tres veces más que subir del modelo trigésimo al primero.** Lo confirman dos mediciones independientes: en Terminal-Bench, Gemini 2.5 Pro mejora un **17% relativo** al pasar de OpenHands a Terminus 2 sin cambiar de modelo; y SWE-bench-Live mide **5,2 puntos** de diferencia con el mismo Opus 4.5.

**Consecuencia directa para Lego:** no merece la pena optimizar el modelo antes que la arquitectura. El bucle, el aislamiento del verificador y el formato de la tarea mueven el resultado más que elegir el modelo de moda. Y como el andamiaje se puede construir con tres piezas, **la restricción de «menos piezas» no es un sacrificio: es donde está el rendimiento**.

## Cuánto resuelven de verdad

**SWE-bench Verified (500 tareas fijas), principios de 2026: 79,2%**, es decir **396 de 500**. Tres fuentes independientes convergen en ese número: una auditoría con veredicto por instancia ([Liu et al., ADMA 2026](https://arxiv.org/abs/2609.17394)), el anuncio de Sonar (*vendor*) y un agregador que cita el leaderboard oficial. **No es el 95–97%** que publican los agregadores sin dar el denominador por modelo.

| Benchmark | Mejor resultado | n | Fuente |
|---|---|---|---|
| SWE-bench Verified | **79,2%** | 500 | Auditoría por instancia, ADMA 2026 |
| SWE-bench **Pro** (descontaminado) | **61,5%** | 731 | [Leaderboard oficial de Scale AI](https://labs.scale.com/leaderboard/swe_bench_pro_public), leído en vivo |
| Terminal-Bench 2.0 | **63%** | 89 | [Paper](https://arxiv.org/abs/2601.11868), leído en PDF |
| SWE-bench-**Live** (descontaminado) | **36%** | — | Leaderboard |
| SWE-rebench (descontaminado) | **26,7%** / 65,3% según fecha | — | NeurIPS 2025 / leaderboard |

**Y el descuento que hay que aplicar.** Descontaminar baja la tasa entre **15 y 25 puntos**: SWE-bench-Live da **19,25% frente al 43,20%** de Verified; SWE-bench Pro está **56 puntos** por debajo. Y descontando los tests débiles, otro tramo: SWE-ABS pasa del **78,80% al 62,20%** al reforzarlos ([ICML 2026](https://arxiv.org/abs/2603.00520)), y la auditoría de OpenAI apunta a que **~1 de cada 5 «resueltos» no lo está**. **Cuatro fuentes independientes** —inspección manual, generación de tests, mutación adversarial y auditoría de anotadores— convergen en que **los tests débiles inflan las tasas**.

**Regla práctica:** un «resuelto» de un ranking no es un parche correcto. Descuenta uno de cada cinco.

## El éxito se derrumba con la longitud, y de forma geométrica

No lineal: **geométrica**. Cuatro mediciones por caminos distintos llegan a la misma forma funcional.

| Estudio | Medida | Resultado |
|---|---|---|
| [ICML 2026, horizonte determinista](https://icml.cc/virtual/2026/poster/62278) | Pasos antes del colapso | **19 a 31 pasos**, y más allá «la precisión se derrumba» |
| [Complexity Ceiling, arXiv:2606.29278](https://arxiv.org/abs/2606.29278) | Pasos para el 50% de acierto (6.000 ejecuciones) | **≈4,7 pasos** en el dominio difícil, con un modelo del 86,3% por paso |
| [METR, horizontes](https://arxiv.org/abs/2503.14499) | Duración al 50% de acierto | **50 minutos** (Claude 3.7, mar-2025); duplicación cada ~7 meses |
| [MAST, taxonomía](https://arxiv.org/abs/2503.13657) | Causa de fallo sobre 200+ trazas | Especificación **41,77%**, desalineación **36,94%**, verificación **21,30%** |

**La aritmética es la misma que la de la maldición de las instrucciones:** si cada paso acierta con probabilidad p, encadenar n pasos da p^n. Por eso el hallazgo de MAST importa tanto: **el 41,77% de los fallos son de especificación**, no de capacidad del modelo. Es el mismo motivo de rechazo dominante que aparece en los PRs reales.

**Consecuencia para la tarea:** cuanto más larga sea la cadena, más cara es cada imprecisión. Una tarea que exige 20 pasos y acierta el 95% en cada uno acaba bien **menos de la mitad de las veces**. La granularidad pequeña no es una preferencia estética: es lo que la aritmética permite.

## La pregunta que los datos no responden

**El 79,1% de los PRs de agentes fusionados no muestra ningún bucle de retroalimentación observable.** Y eso admite dos lecturas opuestas que **los datos no permiten separar**: o los agentes funcionan sin supervisión, o hay **«confianza silenciosa»** — se fusiona sin revisar de verdad. Un estudio independiente lo formula igual y da el **73,9%** de PRs de agentes fusionados sin modificación ([ICSE 2026, JAWs](https://conf.researchr.org/details/icse-2026/jaws-2026-papers/17/Silent-Reliance-or-Diligent-Review-Human-Agent-Engagement-and-Interaction-Patterns-i)).

Otros datos de contexto, con denominador:

- **Sólo el 35,7% de los rechazos son culpa del agente**; el **33,1%** son rechazos sin motivo observable ([Peralta et al., MSR 2026](https://2026.msrconf.org/details/msr-2026-mining-challenge/15/Why-Are-Agentic-Pull-Requests-Merged-or-Rejected-An-Empirical-Study), premio al mejor trabajo del reto).
- **El 15,4%** de los PRs fusionados requirió intervención explícita del revisor; el **5,5%** no tiene rastro humano visible.
- **[DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report)**, sobre ~5.000 respuestas: **el 61% no usa nunca el modo agente autónomo**, y sólo el **24%** confía mucho o muchísimo en él.
- **[Stack Overflow 2025](https://stackoverflow.blog/2025/12/29/developers-remain-willing-but-reluctant-to-use-ai-the-2025-developer-survey-results-are-here/)**, sobre ~49.000 respuestas: **el 66%** está frustrado por código «casi correcto», y el **45%** dice que depurar código de IA lleva más tiempo.

**Lo que no existe, y hay que decirlo:** **ninguna medición de campo con denominador grande sobre qué proporción de ejecuciones autónomas requirió intervención humana.** La unidad disponible son PRs, no ejecuciones. Ese hueco es real.

## Integridad de las fuentes: la investigación se está contaminando

Un estudio estima que **~32% del último trimestre completo de arXiv** muestra estilo de máquina (~65% en informática), con un control previo a ChatGPT de ~0,4% — **con el aviso de que mide similitud de estilo, no autoría, y lo publica el creador de un detector**. Los indicadores prácticos que sí sirven: prosa homogénea, estructura de párrafo uniforme, abstracción alta sin detalle procedimental, y referencias fabricadas.

**Descartados por este motivo en esta investigación:**
- Un paper con autores «Tom Cat» y «Screwy Squirrel» (ver [[investigacion-lego]]).
- [«How Fast Do Agents Rot?»](https://arxiv.org/abs/2609.01660) — autor único sin afiliación ni repositorio, y es la fuente de las cifras más citables de la degradación por paso. **Fuera de toda conclusión.**
- [«Cross-Context Verification»](https://arxiv.org/abs/2603.21454) — autor único sin afiliación, base empírica de **9 problemas y 45 ejecuciones**: denominador demasiado pequeño.

Los papers centrales que sí sostienen este informe no tienen ese patrón: SWE-bench Pro tiene 42 autores de Scale; Terminal-Bench es del Laude Institute y Stanford; UTBoost está en ACL 2025 con DOI.

## Dónde se ha buscado

**Leídos a texto completo:** [leaderboard de SWE-bench Pro](https://labs.scale.com/leaderboard/swe_bench_pro_public) · [Terminal-Bench, arXiv:2601.11868](https://arxiv.org/abs/2601.11868) · [arXiv:2602.04226](https://arxiv.org/html/2602.04226v1). **Revisados por pares:** ACL 2025 ([UTBoost](https://aclanthology.org/2025.acl-long.189/)), NeurIPS 2025 ([SWE-rebench](https://papers.neurips.cc/paper_files/paper/2025/hash/21bec6ace947b1b58967b945c8ac0f10-Abstract-Datasets_and_Benchmarks_Track.html), [SWE-bench-Live](https://arxiv.org/abs/2505.23419), SWE-Bench Illusion), ICML 2026 ([SWE-ABS](https://arxiv.org/abs/2603.00520), horizonte determinista), [MSR 2026](https://2026.msrconf.org/details/msr-2026-mining-challenge/15/Why-Are-Agentic-Pull-Requests-Merged-or-Rejected-An-Empirical-Study), [ADMA 2026](https://arxiv.org/abs/2609.17394). **Informes:** [METR](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/), [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report), [Stack Overflow 2025](https://stackoverflow.blog/2025/12/29/developers-remain-willing-but-reluctant-to-use-ai-the-2025-developer-survey-results-are-here/), [Stanford AI Index 2026](https://hai.stanford.edu/assets/files/ai_index_report_2026_chapter_2_technical.pdf), [Answer.AI/Devin](https://www.answer.ai/posts/2025-01-08-devin.html).

**Buscado y sin aportar nada:** `swebench.com` directamente (contenido truncado, no legible); el blog de OpenAI sobre SWE-bench Pro (**403 en dos rutas** — sus cifras del ~30% están respaldadas por tres medios independientes que coinciden, pero **no por lectura directa**); `tbench.ai` sirve hoy la versión 4.0, no la 2.0; METR Time Horizons 1.1 sólo por secundarias; y varios repos de auto-reporte de resultados, descartados por no auditables.

**Sin comprobar:** las cifras de 93,9% y 97% de los agregadores (contradicen la auditoría y no dan denominador — **no se usan**); la tabla completa de Terminal-Bench con numerador por celda; y el detalle del «merge rate» del README de AIDev (no explicita si cuenta todos los PRs o sólo los cerrados).

## Enlaces

- [[crear-la-tarea]] — el formato que la aritmética de pasos exige
- [[consumir-la-tarea]] — el andamiaje, que es la palanca
- [[robustez-desatendida]] — por qué el andamiaje falla y cómo
- [[investigacion-lego]] — el informe completo
