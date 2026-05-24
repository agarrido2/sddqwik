---
# EXTERNAL_AGENT_PATH: ".github/prompts/new-session.prompt.md"
name: new-session
description: >
  Reanuda un chat nuevo desde docs/sessions/INDEX.md, usando como guía un
  Prompt de Reanudación de @QwikMemory si el usuario lo aporta. Valida
  coherencia, carga solo contexto mínimo y prepara un handoff seguro a
  @QwikOrchestrator. Si la memoria es insuficiente o contradictoria, detiene
  la reconstrucción automática o enruta a Orchestrator con incertidumbre visible.
tools: ["read"]
argument-hint: "example: /new-session"
---

# 🔁 NEW SESSION — REENTRY PROTOCOL

## Propósito

`/new-session` reanuda trabajo en un chat nuevo sin depender de la conversación anterior.

No resume historial.
No explora el repo a ciegas.
No decide implementación.
No sustituye a `@QwikOrchestrator`.
No mezcla features por intuición.
No reabre features cerradas salvo incidencia formal por `/bug-fix`.

Su objetivo es reconstruir el mínimo contexto operativo necesario para continuar:

```text
INDEX → Prompt de Reanudación si existe → Snapshot prioritario si existe y hace falta → Artefactos mínimos → Handoff a @QwikOrchestrator
```

El flujo operativo principal que debe respetar es:

```text
Spec → Plan → Implementation Tasks → Build → Audit → Polish → Memory
```

---

## Regla operativa crítica

`docs/sessions/INDEX.md` es siempre la primera fuente.

Antes de hacer handoff a `@QwikOrchestrator`, este prompt debe determinar con evidencia:

1. qué feature o frente se retoma;
2. qué fuente de reentrada se usó;
3. qué fila de INDEX gobierna la reentrada;
4. si existe Prompt de Reanudación y qué ordena;
5. si el snapshot prioritario existe y si realmente debe cargarse;
6. qué artefactos mínimos están permitidos;
7. qué artefactos no deben cargarse inicialmente;
8. cuál es el estado y fase actuales;
9. cuál es el siguiente paso exacto;
10. qué agente debería actuar después del Orchestrator;
11. qué riesgos o prerequisitos siguen abiertos.

Si no puede determinar el siguiente paso con INDEX/snapshot, no inventar: enrutar a `@QwikOrchestrator` con la incertidumbre explícita.

---

## Prohibiciones

Durante `/new-session`, no hacer:

- leer todo el repositorio;
- listar todas las specs;
- listar todos los plans;
- cargar sesiones archivadas salvo snapshot explícito y necesario;
- cargar audits no relacionados;
- cargar artefactos que el Prompt de Reanudación indique como no iniciales;
- inventar estado si INDEX y snapshot no coinciden;
- asumir que la última conversación sigue vigente;
- saltar directamente a Builder;
- reabrir features `PRODUCTION-READY` o `ARCHIVED` salvo `/bug-fix` formal;
- recomendar abrir artefactos fuera del flujo operativo principal;
- escribir código;
- modificar artefactos.

---

## Paso 0 — Leer INDEX primero

`docs/sessions/INDEX.md` es la fuente de verdad colectiva.

Debe leerse siempre antes de cargar snapshot o artefactos de feature, incluso si el usuario pegó un Prompt de Reanudación.

```text
docs/sessions/INDEX.md
```

Si no existe:

```text
NEW-SESSION GATE BLOQUEADO: falta docs/sessions/INDEX.md.
Siguiente paso: ejecutar /setup para inicializar memoria operativa.
```

No continuar con reconstrucción automática si falta INDEX, salvo que el usuario proporcione un snapshot completo y acepte continuar con memoria degradada.

Si se continúa con memoria degradada, la salida debe marcar `INDEX validado: no` y delegar normalización a `@QwikMemory` vía `@QwikOrchestrator`.

---

## Paso 1 — Detectar Prompt de Reanudación

Evaluar si el usuario aportó un Prompt de Reanudación generado por `@QwikMemory`.

Formato esperado:

```text
/new-session

Feature cerrada: [feature]
Estado: PRODUCTION-READY | ARCHIVED | WIP | BLOCKED | READY_FOR_DBA | READY_FOR_BUILD | AUDIT_FAILED | NEEDS-WORK | LEGACY
Snapshot prioritario: docs/sessions/[feature]-[timestamp].md
Primera fuente: docs/sessions/INDEX.md
Siguiente paso recomendado: [acción/agente] | @QwikOrchestrator debe decidir

Artefactos permitidos para cargar:
- ruta real y motivo

Artefactos que NO deben cargarse inicialmente:
- ruta/patrón real y motivo

Regla de cierre:
- No reabrir la feature cerrada salvo entrada formal por /bug-fix.
```

Campos mínimos:

```text
Feature cerrada o Feature/frente
Estado
Snapshot prioritario
Primera fuente: docs/sessions/INDEX.md
Siguiente paso recomendado
Artefactos permitidos para cargar
Artefactos que NO deben cargarse inicialmente
```

Si el Prompt de Reanudación existe, usarlo como guía de reentrada, pero validarlo siempre contra INDEX.

Si no existe, usar solo INDEX para seleccionar frente y artefactos mínimos.

---

## Paso 2 — Seleccionar fuente de reentrada

Aplicar este orden:

| Prioridad | Condición | Fuente |
|---|---|---|
| 1 | INDEX confirma la feature indicada por Prompt de Reanudación válido | `index-plus-resume-prompt` |
| 2 | INDEX contiene fila activa de la feature indicada por el usuario | `index-active-entry` |
| 3 | INDEX referencia snapshot vigente para una feature WIP/BLOCKED/NEEDS-WORK | `index-snapshot` |
| 4 | INDEX contiene una única feature WIP | `index-single-wip` |
| 5 | INDEX contiene solo cierres y el prompt indica siguiente paso recomendado | `closed-feature-next-step` |
| 6 | INDEX no contiene frente claro | `blocked-insufficient-memory` |

### Regla

Si hay más de una feature WIP y el usuario no indicó cuál retomar, detener:

```text
NEW-SESSION GATE BLOQUEADO: hay múltiples frentes activos y no se indicó cuál reanudar.
Indica: /new-session para [feature-name] o pega el Prompt de Reanudación generado por @QwikMemory.
```

---

## Paso 3 — Respetar cierres

Si INDEX o el Prompt de Reanudación marca la feature como:

```text
PRODUCTION-READY
ARCHIVED
```

entonces:

- respetar el cierre;
- no reabrir Spec, Plan, Implementation Tasks, Build, Audit ni Polish;
- no recomendar continuar construcción de esa feature;
- buscar el siguiente paso recomendado en INDEX o Prompt de Reanudación;
- si el siguiente paso es desconocido, enrutar a `@QwikOrchestrator` para decidir;
- si el usuario reporta un fallo posterior, dirigir a `/bug-fix [bug-id]`.

Salida esperada si intenta reabrirse sin bug formal:

```text
NEW-SESSION GATE BLOQUEADO: la feature está cerrada como PRODUCTION-READY/ARCHIVED.
No se reabre salvo incidencia formal por /bug-fix.
Siguiente paso: [siguiente paso recomendado] | @QwikOrchestrator debe decidir.
```

---

## Paso 4 — Validar snapshot prioritario

Si existe `Snapshot prioritario`, verificar:

- la ruta está citada por el Prompt de Reanudación o INDEX;
- la ruta existe;
- el snapshot corresponde a la feature indicada;
- el snapshot contiene estado, fase, siguiente paso o contrato equivalente;
- el INDEX contiene o puede reconocer esa feature.

El snapshot prioritario se carga solo si:

```text
INDEX lo cita
o el Prompt de Reanudación lo marca como prioritario
o INDEX no basta para determinar siguiente paso
o hay contradicción que requiere resolver fuente vigente
```

Si el snapshot no existe:

```text
NEW-SESSION WARNING: el Snapshot prioritario indicado no existe.
Continuar solo con INDEX si permite determinar siguiente paso; si no, enrutar a @QwikOrchestrator.
```

Si snapshot e INDEX se contradicen:

- preferir INDEX para estado de cierre;
- si el snapshot es más reciente y completo para un WIP, usarlo como guía;
- marcar advertencia: `INDEX requiere revisión por @QwikMemory`;
- no modificar INDEX desde `/new-session`;
- delegar corrección a `@QwikMemory` tras handoff de Orchestrator.

---

## Paso 5 — Carga mínima permitida

Cargar solo lo permitido por INDEX o Prompt de Reanudación:

```text
docs/sessions/INDEX.md
docs/sessions/[snapshot-prioritario].md solo si existe y es necesario
docs/specs/[feature].md si la fase es Spec, Plan, Build, Audit, Polish o Memory y está permitido
docs/plans/[feature].md si la fase es Plan, Implementation Tasks, Build, Audit, Polish o Memory y está permitido
docs/audits/[feature]-audit.md si la fase es Audit, Corrective Build, Polish o Memory y está permitido
docs/audits/[feature]-polish.md si la fase es Polish, Memory o Closure y está permitido
docs/bugs/[bug-id].md solo si la fase es Bug Diagnosis/Fix/Verification o hay entrada formal por /bug-fix
```

No cargar artefactos que el Prompt de Reanudación liste bajo:

```text
Artefactos que NO deben cargarse inicialmente
```

### Regla

El INDEX decide la fuente inicial.
El Prompt de Reanudación limita la carga.
El snapshot aclara el punto exacto solo cuando hace falta.
El agente no debe buscar contexto por curiosidad.

---

## Paso 6 — Reconstruir estado operativo

Reconstruir solo lo necesario para continuar.

Debe quedar claro:

- feature o frente retomado;
- estado actual;
- fase actual: Spec | Plan | Implementation Tasks | Build | Audit | Polish | Memory | BugFix | Legacy | Resume;
- fuente de reentrada usada;
- entrada de INDEX usada;
- Prompt de Reanudación usado: sí/no;
- snapshot usado: ruta o N/A;
- artefactos cargados;
- artefactos no cargados por restricción;
- decisiones vigentes;
- riesgos o bloqueos abiertos;
- siguiente paso exacto;
- agente sugerido tras Orchestrator.

Si cualquiera de estos campos queda desconocido, marcarlo como `UNKNOWN` y explicar qué artefacto falta.

No inventar.

---

## Paso 7 — Continuar según fase

Usar esta matriz como guía de continuidad:

| Estado/Fase detectada | Continuidad esperada |
|---|---|
| Spec Draft/Review | `@QwikSpeccer` vía Orchestrator |
| Spec Approved sin Plan | `@QwikArchitect` vía Orchestrator |
| Plan READY_FOR_DBA | `@QwikDBA` vía Orchestrator |
| Plan READY_FOR_BUILD | `@QwikBuilder` vía Orchestrator |
| Implementation Tasks pendientes | `@QwikBuilder` o `@QwikArchitect` según bloqueo, vía Orchestrator |
| Build entregado sin Audit | `@QwikAuditor` vía Orchestrator |
| Audit FAILED/AUDIT_FAILED | `@QwikBuilder` o `@QwikArchitect` según ciclo, vía Orchestrator |
| Audit PASSED sin Polish | `@QwikPolisher` vía Orchestrator |
| Polish PRODUCTION-READY sin Memory | `@QwikMemory` vía Orchestrator |
| PRODUCTION-READY | respetar cierre y seguir siguiente paso recomendado |
| ARCHIVED | respetar archivo y seguir siguiente paso recomendado |
| Bug formal | `@QwikBugFix` vía Orchestrator |
| UNKNOWN | `@QwikOrchestrator` decide |

No saltar directamente a Builder desde `/new-session`.
El handoff siempre pasa por `@QwikOrchestrator`.

---

## Paso 8 — Resolver inconsistencias

Aplicar estas reglas:

| Caso | Acción |
|---|---|
| INDEX marca cierre y snapshot marca WIP | Respetar cierre; no reabrir salvo `/bug-fix` |
| INDEX más reciente que snapshot | Usar INDEX y marcar snapshot como potencialmente obsoleto |
| Snapshot más reciente y feature no cerrada | Usar snapshot como guía y advertir que INDEX requiere revisión |
| Snapshot e INDEX apuntan a features distintas | Detener y pedir confirmación |
| Snapshot no tiene siguiente paso | Usar INDEX si lo tiene; si no, Orchestrator decide |
| INDEX vacío sin snapshot | Detener y recomendar `/setup` o `/memory-compact` previo |
| Prompt de Reanudación prohíbe cargar un artefacto | No cargarlo inicialmente |
| Usuario pide reabrir cierre sin bug | Bloquear y recomendar `/bug-fix` |

---

## Paso 9 — Preparar handoff a @QwikOrchestrator

El handoff debe ser explícito y mínimo.

Formato recomendado:

```json
{
  "handoff_id": "[feature]-[timestamp]",
  "resume_source": "index-plus-resume-prompt | index-active-entry | index-snapshot | index-single-wip | closed-feature-next-step | blocked-insufficient-memory",
  "feature": "[feature]",
  "state": "[estado]",
  "current_phase": "[fase]",
  "closed_feature": "true | false",
  "closure_rule": "do-not-reopen-except-bug-fix | N/A",
  "snapshot_used": "docs/sessions/[feature]-[timestamp].md | N/A",
  "index_entry": "docs/sessions/INDEX.md#[feature]",
  "resume_prompt_used": "true | false",
  "context_refs": [
    "docs/specs/[feature].md",
    "docs/plans/[feature].md",
    "docs/audits/[feature]-audit.md",
    "docs/audits/[feature]-polish.md"
  ],
  "do_not_reload_initially": [
    "[rutas/patrones indicados por INDEX o Prompt de Reanudación]"
  ],
  "next_step": "[acción concreta] | @QwikOrchestrator debe decidir",
  "routing_decision": "@QwikOrchestrator",
  "suggested_next_agent": "@QwikSpeccer | @QwikArchitect | @QwikDBA | @QwikBuilder | @QwikAuditor | @QwikPolisher | @QwikMemory | @QwikBugFix | none | UNKNOWN",
  "warnings": [
    "[bloqueos, conflictos o prerequisitos abiertos]"
  ]
}
```

Si el siguiente paso no está claro, `next_step` debe ser:

```text
@QwikOrchestrator debe decidir
```

---

## Paso 10 — Invocar a @QwikOrchestrator

Mensaje de handoff:

```text
@QwikOrchestrator

Reanudar trabajo desde /new-session.

Primera fuente validada: docs/sessions/INDEX.md
Fuente de reentrada: [resume_source]
Prompt de Reanudación usado: sí/no
Feature/frente: [feature]
Estado: [estado]
Fase actual: [fase]
Snapshot usado: [snapshot o N/A]
Artefactos cargados: [lista mínima]
Artefactos no cargados inicialmente: [lista]
Siguiente paso exacto: [acción] | @QwikOrchestrator debe decidir
Agente sugerido tras routing: [agente]
Feature cerrada: sí/no
Regla de cierre: no reabrir salvo /bug-fix | N/A
Advertencias: [warnings]

Tarea:
1. Validar el routing con la información mínima cargada.
2. No explorar el repo masivamente.
3. No reabrir features cerradas salvo incidencia formal por /bug-fix.
4. Enrutar al agente correcto según Spec, Plan, Implementation Tasks, Build, Audit, Polish o Memory.
5. Si INDEX requiere actualización, activar @QwikMemory antes de continuar.
```

---

## Salida esperada

```text
NEW SESSION READY

Feature/frente: [feature]
Estado: [estado]
Fase actual: [fase]
Fuente de reentrada: [index-plus-resume-prompt / index-active-entry / index-snapshot / index-single-wip / closed-feature-next-step]
INDEX validado: sí / no
Prompt de Reanudación usado: sí / no
Snapshot usado: [ruta o N/A]
Snapshot cargado: sí / no / no necesario
Artefactos cargados: [lista]
Artefactos no cargados inicialmente: [lista]
Siguiente paso exacto: [acción] | @QwikOrchestrator debe decidir
Agente sugerido: [agente]
Feature cerrada: sí / no
Regla de cierre aplicada: no reabrir salvo /bug-fix | N/A
Advertencias: [si aplica]
Estado de reentrada: READY FOR ORCHESTRATOR / BLOCKED
```

Si queda bloqueado:

```text
NEW-SESSION GATE BLOQUEADO
Motivo:
Evidencia:
Siguiente acción:
```

---

## Cuándo usar este prompt

- Al abrir un chat nuevo después de `/memory-compact`.
- Cuando el usuario pega un Prompt de Reanudación generado por `@QwikMemory`.
- Cuando se quiere reanudar una feature sin depender del historial anterior.
- Cuando existe snapshot de sesión y se quiere reconstruir contexto mínimo.
- Cuando el sistema debe volver al punto exacto donde quedó el trabajo.

---

## Regla final

`/new-session` no significa “cuéntame qué pasó”.

`/new-session` significa:

```text
Recupera el siguiente paso correcto con el menor contexto posible y entrega el control al Orchestrator.
```

Si no puede hacerlo con evidencia, debe bloquear o entregar la decisión a `@QwikOrchestrator` con incertidumbre explícita.
