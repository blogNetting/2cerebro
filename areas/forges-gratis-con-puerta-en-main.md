---
title: Forge gratis con puerta real en `main` (repo privado)
created: 2026-10-04
updated: 2026-10-04
tags: [git, forge, github, gitlab, bitbucket, azure-devops, ci, rulesets, gratis]
zona: tecnico
---

Qué alojamiento de código da, **gratis y en repositorio privado**, una puerta de servidor que impida escribir directamente en `main` y que pueda exigir el CI en verde antes de fusionar. Investigado el 2026-10-04 para decidir dónde vive un proyecto futuro.

## 1. Qué se pregunta y por qué

El gatillo fue [[control-de-versiones-y-ci]]: la nota manda montar un ruleset en GitHub que en el repo de Patrimonial devuelve `403 — Upgrade to GitHub Pro or make this repository public`. De ahí la pregunta general: **¿existe algún forge donde esa puerta sea gratis en un repo privado?** No es una pregunta de preferencia: es si el servidor dice «no» al push, no si uno promete no hacerlo.

## 2. Criterio de admisión, escrito antes de buscar

- **C1 (duro):** repo privado y gratis, con protección de rama que impida escribir directo en `main`, **impuesta por el servidor**.
- **C2 (duro):** que esa puerta pueda exigir el CI en verde antes de fusionar.
- **C3 (duro):** que «gratis» no tenga trampa para un solo usuario (minutos de CI suficientes, sin topes que lo rompan).
- **C4 (blando):** coste real de tenerlo (mantenimiento, dependencia, migración).

**Cierre pactado:** parar por saturación (una ronda sin dato nuevo), no por coste.

## 3. Considerado y descartado

| Candidato | C1 | C2 | Veredicto |
|---|---|---|---|
| **GitHub Free** | ❌ | ❌ | Descartado: la propia API contesta `403` |
| **Bitbucket Cloud Free** | ⚠️ sí | ❌ | Descartado: medio muro y CI de adorno (y roto para tokens) |
| **Azure DevOps Free** | ✅ | ✅ | Descartado por encaje, no por capacidad |
| **Codeberg** | ✅ | ✅ | Descartado: privados limitados a ~100 MiB y solo para fines de software libre |
| **GitLab.com Free** | ✅ | ✅ | **Elegido** |

**Forge autoalojado: excluido a petición del usuario** (2026-10-04) — no es una opción para él. No se evalúa aquí.

## 4. Análisis

### 4.1 GitHub Free — el que ya usaba, y el único que cobra la puerta

| Prueba | Resultado |
|---|---|
| `gh api repos/blogNetting/patrimonial/rulesets` | `{"message":"Upgrade to GitHub Pro or make this repository public to enable this feature.","status":"403"}` |
| `gh api repos/blogNetting/patrimonial/branches/main/protection` (protección clásica) | Mismo `403` |
| `gh api repos/blogNetting/patrimonial/actions/permissions` | `{"enabled":true,...}` — el CI **sí** corre gratis |

Tabla de planes, textual: los rulesets están *"in public repositories with Free user and Free team for organizations"*, y en privados solo con Pro, Team o Enterprise Cloud ([doc de gated features](https://raw.githubusercontent.com/anthonysidesapps/docs/3ca115ec08ea33cf5bbec50b10f48543d8b2af95/data/reusables/gated-features/repo-rules.md)). Los *push rulesets* son aún más caros: Team y Enterprise Cloud ([changelog 2024-09-10](https://github.blog/changelog/2024-09-10-push-rules-are-now-generally-available-and-updates-to-custom-properties/)).

Lo que sí es gratis: el workflow, los tics verdes y rojos, y los comentarios del robot. Lo único de pago es que **la plataforma actúe** sobre esa información. Sin la puerta, el check es un consejo: *"If required status checks aren't enabled, collaborators can merge the branch at any time"* ([StackOverflow](https://stackoverflow.com/feeds/question/71053336)); y en un privado gratis *"anybody can merge anything, regardless of the CODEOWNERS file"* ([Beman Project](https://discourse.bemanproject.org/t/github-free-plan-wont-allow-enforcing-rules-on-private-repos/306/6)).

Minutos gratis: **2.000/mes** en privados, e *"If your account does not have a valid payment method on file, usage is blocked once you use up your quota"* ([doc de facturación](https://docs.github.com/en/billing/concepts/product-billing/github-actions)).

Detalle importante: *"Push rulesets"* (reglas que se aplican a todo push, sin apuntar a rama) **no están ni en Pro**. Para lo que se busca basta el ruleset normal con *"Require a pull request before merging"*, que sí entra en Pro.

### 4.2 GitLab.com Free — la puerta completa, gratis

| Pieza | Textual |
|---|---|
| Protected branches | **"Tier: Free, Premium, Ultimate"** ([docs](https://docs.gitlab.com/user/project/repository/branches/protected/)) |
| Bloquear el push directo | «Allowed to push» admite **"No one"**; la doc avisa de que *"When Allowed to push and merge is not configured, it does not restrict push access"* |
| Exigir CI en verde | *"You can configure your project to require a complete and successful pipeline before merge"*, en *Merge checks* → **Pipelines must succeed** ([docs](https://docs.gitlab.com/user/project/merge_requests/auto_merge/)) |
| Trampa de esa opción | *"A merge request with no pipelines at all is not considered to have a successful pipeline, and cannot merge"* — hay que garantizar que todo MR lanza pipeline |
| Aprobaciones obligatorias | **"Required approvals — Tier: Premium, Ultimate"**; en Free los *approve* *"are optional and don't prevent merging without approval"* ([docs](https://docs.gitlab.com/user/project/merge_requests/approvals/)) |
| Minutos de CI | *"Free tier namespaces receive 400 compute minutes per month"* ([docs](https://docs.gitlab.com/ci/pipelines/compute_minutes/)) |
| Límite de gente | 5 usuarios en namespaces privados de nivel superior ([docs](https://docs.gitlab.com/user/free_user_limit/)) |
| Fricción | Los *shared runners* en cuentas Free creadas después del 2021-05-17 piden **verificación con tarjeta** (autorización de 1 $, sin cargo): [The Register](https://www.theregister.com/2021/05/19/gitlab_crypto), [handbook de GitLab](https://handbook.gitlab.com/handbook/support/workflows/remove_validation/). Se evita usando un runner propio |

**El muro, en la práctica:** `main` con «Allowed to push: No one» + «Pipelines must succeed»; el agente con rol Developer (no puede empujar ni fusionar) y el humano como Maintainer. C1 y C2, gratis, en privado. Lo que **no** da gratis es exigir *aprobación* de otra persona — pero para un solo usuario la puerta la cierra el rol, no la aprobación.

### 4.3 Bitbucket Cloud Free — muro a medias y roto para agentes

- Branch permissions existen y bloquean el push directo (comunidad: *"Prevent changes without a pull request"* devuelve `pre-receive hook declined`, [StackOverflow](https://stackoverflow.com/questions/54021040/branch-permissions-bypassed-on-bitbucket-pull-request-requires-approval-but-me)).
- **Pero el CI no puede ser puerta:** *"Merge checks are a Premium feature for Bitbucket Cloud"* ([Atlassian](https://support.atlassian.com/bitbucket-cloud/docs/use-branch-permissions/)). Sin eso, C2 se cae.
- **Y los tokens no funcionan con branch restrictions:** *"You are using an access token to authenticate whilst branch restrictions are configured — this is currently not supported"* ([KB de Atlassian](https://support.atlassian.com/bitbucket-cloud/kb/unable-to-push-to-repository-pre-receive-hook-declined-error/), issue BCLOUD-22400).
- Caso real (2026-01/06): pipeline con token → `remote: Permission denied to update branch master.`; el usuario acabó activando y desactivando restricciones por API, y su propia conclusión fue *"it's not a valid permanent solution"* ([foro de desarrolladores de Atlassian](https://community.developer.atlassian.com/t/bitbucket-pipelines-cannot-push-to-protected-master-branch-using-repository-access-token/98546)).
- Límites: 5 usuarios y **50 minutos** de build al mes ([precios](https://www.atlassian.com/software/bitbucket/pricing)).

### 4.4 Azure DevOps Free — cumple, pero no encaja

- *"Unlimited private Git repos"*, 5 usuarios gratis, y *"1 Microsoft-hosted job with 1,800 minutes per month for CI/CD and 1 self-hosted job with unlimited minutes per month"* ([precios](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/)). Es la mejor cuota de CI gratis de la comparativa.
- Las *branch policies* no aparecen condicionadas a plan: solo piden el permiso **«Edit policies»** ([docs](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops)), y el muro es real: *"You can't push changes directly to branches with required branch policies unless you have permissions to bypass branch policies"*. El bypass («Bypass policies when pushing») se concede aparte.
- **Se descarta por encaje**, no por capacidad: es el ecosistema con menos herramientas alrededor para este uso.

## 5. Recomendaciones

1. **GitLab.com Free** si la prioridad es no pagar y seguir en un SaaS: es el único que da C1 y C2 gratis en privado. Registrar un runner propio quita de golpe el tope de 400 minutos y la verificación con tarjeta.
2. **GitHub Pro** (~4 €/mes, coste de cuenta, no por repo) si la prioridad es no mover nada: es lo que menos trabajo da.
3. **No cambiar de forge por cambiar.** El coste de partir no es técnico sino de cabeza: dos forges son dos herramientas y dos sitios donde mirar.

**Y la regla que vale para cualquiera de las dos: la puerta solo existe si el actor que no debe pasar tiene un rol por debajo de ella.** Da igual el forge, el mecanismo o el plan: si el agente usa la credencial del dueño, es el dueño.

## 6. Dónde se ha buscado

Siete rondas. Primarias: GitLab (protected branches, approvals, auto-merge, compute minutes, free user limit), Atlassian (branch permissions, KB de errores, precios), Microsoft (precios, branch policies), GitHub (facturación de Actions, doc de gated features), Codeberg (límites de almacenamiento). Comunidad: [foro de desarrolladores de Atlassian](https://community.developer.atlassian.com/t/bitbucket-pipelines-cannot-push-to-protected-master-branch-using-repository-access-token/98546), [StackOverflow](https://stackoverflow.com/questions/54021040/branch-permissions-bypassed-on-bitbucket-pull-request-requires-approval-but-me), [Qiita (tabla de tiers de GitLab)](https://qiita.com/GL_Tsukasa/items/06a2fcb5e023744b1b18), [The Register](https://www.theregister.com/2021/05/19/gitlab_crypto), [Beman Project](https://discourse.bemanproject.org/t/github-free-plan-wont-allow-enforcing-rules-on-private-repos/306/6).

**No aportaron nada:** la comparativa de planes de Atlassian (404), la página de branch permissions de `support.atlassian.com` (404 en una segunda ruta), la de merge checks de GitLab (redirige a un login) y varias comparativas de terceros que solo repetían folletos.

## 7. Sin verificar

- Que la comparativa de planes de Atlassian liste *branch permissions* en Free: la página no cargó. Se concluye por ausencia de *gating* en la doc + dos fuentes de comunidad, no por cita directa.
- Que Azure DevOps «incluya formalmente» branch policies en el plan Basic: no encontré ningún *gating*; es **inferencia**, no cita.
- El comportamiento real de GitLab con «Allowed to push: No one» no se ha probado en un repo propio — está documentado, no ejecutado.

## Enlaces

- [[control-de-versiones-y-ci]] — la nota que destapó el problema: rulesets y CI en el repo de Patrimonial
- [[gas-city-y-gitlab]] — qué le cuesta a Gas City trabajar contra GitLab
- [[gas-city-operacion-real]] — la ciudad real y sus rigs
- [[_index]]
