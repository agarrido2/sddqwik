---
# EXTERNAL_AGENT_PATH: ".github/prompts/new-session.prompt.md"
name: new-session
description: >
  Reanuda un chat nuevo desde un Prompt de Reanudación, snapshot operativo o
  docs/sessions/INDEX.md. Valida coherencia, carga solo contexto mínimo y prepara
  un handoff seguro a @QwikOrchestrator. Si la memoria es insuficiente o
  contradictoria, detiene la reconstrucción automática y pide confirmación.
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

Su objetivo es reconstruir el mínimo contexto operativo necesario para continuar:

```text
Prompt de Reanudación / INDEX → Snapshot → Artefactos mínimos → Handoff a @QwikOrchestrator
```

---

## Regla operativa crítica

Antes de hacer handoff a `@QwikOrchestrator`, este prompt debe determinar con evidencia:

1. qué feature o frente se retoma;
2. qué fuente de reentrada se usó;
3. qué snapshot o fila de INDEX gobierna la reentrada;
4. qué artefactos mínimos deben cargarse;
5. cuál es el siguiente paso exacto;
6. qué agente debería actuar después del Orchestrator;
7. qué riesgos o prerequisitos siguen abiertos.

Si no puede determinar eso, detener.

---

## Prohibiciones

Durante `/new-session`, no hacer:

- leer todo el repositorio;
- listar todas las specs;
- listar todos los plans;
- cargar sesiones archivadas salvo snapshot explícito;
- abrir blueprints cerrados sin necesidad;
- cargar audits no relacionados;
- inventar estado si INDEX y snapshot no coinciden;
- asumir que la última conversación sigue vigente;
- saltar directamente a Builder;
- escribir código;
- modificar artefactos.

---

## Paso 0 — Detectar entrada del usuario

Evaluar si el usuario aportó un Prompt de Reanudación generado por `/memory-compact`.

Buscar explícitamente estos campos:

```text
Reanudar feature:
Snapshot prioritario:
Verificar contra índice:
Objetivo inmediato:
Agente sugerido tras reentrada:
```

Si existen, usar esa fuente como primaria.

Si no existen, usar `docs/sessions/INDEX.md` como fuente primaria.

---

## Paso 1 — Verificar INDEX

`docs/sessions/INDEX.md` es la fuente de verdad colectiva.

Debe leerse siempre como validación estructural, incluso si existe snapshot explícito.

```text
docs/sessions/INDEX.md
```

Si no existe:

```text
NEW-SESSION GATE BLOQUEADO: falta docs/sessions/INDEX.md.
Siguiente paso: ejecutar /setup para inicializar memoria operativa.
```

No continuar con reconstrucción automática si falta INDEX, salvo que el usuario proporcione un snapshot completo y acepte continuar con memoria degradada.

---

## Paso 2 — Seleccionar fuente de reentrada

Aplicar este orden estricto:

| Prioridad | Condición | Fuente |
|---|---|---|
| 1 | El usuario pegó Prompt de Reanudación válido | `prompt-snapshot` |
| 2 | INDEX contiene fila activa de la feature indicada | `index-active-entry` |
| 3 | INDEX referencia snapshot reciente para una feature WIP/Blocked | `index-snapshot` |
| 4 | INDEX contiene una única feature WIP | `index-single-wip` |
| 5 | INDEX no contiene frente claro | `blocked-insufficient-memory` |

### Regla

Si hay más de una feature WIP y el usuario no indicó cuál retomar, detener:

```text
NEW-SESSION GATE BLOQUEADO: hay múltiples frentes activos y no se indicó cuál reanudar.
Indica: /new-session para [feature-name] o pega el Prompt de Reanudación de /memory-compact.
```

---

## Paso 3 — Validar snapshot prioritario

Si existe `Snapshot prioritario`, verificar:

- la ruta existe;
- el snapshot corresponde a la feature indicada;
- el snapshot contiene `Prompt de Reanudación` o contrato de reentrada equivalente;
- el snapshot indica siguiente paso exacto;
- el snapshot indica agente sugerido;
- el INDEX contiene o puede reconocer esa feature.

Si el snapshot no existe:

```text
NEW-SESSION GATE BLOQUEADO: el Snapshot prioritario indicado no existe.
Revisa la ruta o ejecuta /setup para diagnosticar memoria.
```

Si snapshot e INDEX se contradicen:

- si el snapshot es más reciente y completo, usarlo como fuente primaria;
- marcar advertencia: `INDEX requiere actualización`;
- no modificar INDEX desde `/new-session`;
- delegar corrección a `@QwikMemory` tras handoff.

---

## Paso 4 — Carga mínima permitida

Cargar solo:

```text
docs/sessions/INDEX.md
docs/sessions/[snapshot-prioritario].md si existe
docs/specs/[feature].md si snapshot/INDEX lo pide
docs/plans/[feature].md si snapshot/INDEX lo pide
docs/audits/[feature]-audit.md solo si la fase es Audit/Corrective Build
docs/audits/[feature]-polish.md solo si la fase es Polish/Memory Close
docs/bugs/[bug-id].md solo si la fase es Bug Diagnosis/Fix/Verification
```

Cargar otros artefactos solo si están citados explícitamente por snapshot o INDEX.

### Regla

El snapshot y el INDEX deciden qué cargar.
El agente no debe buscar contexto por curiosidad.

---

## Paso 5 — Reconstruir estado operativo

Reconstruir solo lo necesario para continuar.

Debe quedar claro:

- feature o frente retomado;
- estado actual;
- fase actual;
- fuente de reentrada usada;
- snapshot usado;
- entrada de INDEX usada;
- artefactos cargados;
- decisiones vigentes;
- riesgos o bloqueos abiertos;
- siguiente paso exacto;
- agente sugerido tras Orchestrator.

Si cualquiera de estos campos queda desconocido, marcarlo como `UNKNOWN` y explicar qué artefacto falta.

No inventar.

---

## Paso 6 — Resolver inconsistencias

Aplicar estas reglas:

| Caso | Acción |
|---|---|
| Snapshot más reciente que INDEX | Usar snapshot, advertir que INDEX requiere actualización |
| INDEX más reciente que snapshot | Usar INDEX y marcar snapshot como potencialmente obsoleto |
| Snapshot e INDEX apuntan a features distintas | Detener y pedir confirmación |
| Snapshot no tiene siguiente paso | Usar INDEX si lo tiene; si no, detener |
| INDEX vacío sin snapshot | Detener y recomendar `/setup` o `/memory-compact` previo |
| Feature aparece Done pero snapshot dice WIP | Detener y pedir confirmación |

---

## Paso 7 — Preparar handoff a @QwikOrchestrator

El handoff debe ser explícito y mínimo.

Formato recomendado:

```json
{
  "handoff_id": "[feature]-[timestamp]",
  "resume_source": "prompt-snapshot | index-active-entry | index-snapshot | index-single-wip",
  "feature": "[feature]",
  "snapshot_used": "docs/sessions/[feature]-[timestamp].md | N/A",
  "index_entry": "docs/sessions/INDEX.md#[feature]",
  "context_refs": [
    "docs/specs/[feature].md",
    "docs/plans/[feature].md"
  ],
  "current_phase": "[fase]",
  "next_step": "[acción concreta]",
  "routing_decision": "@QwikOrchestrator",
  "suggested_next_agent": "@QwikBuilder | @QwikAuditor | @QwikArchitect | @QwikDBA | @QwikBugFix | @QwikPolisher | @QwikSpeccer | @QwikMemory",
  "warnings": [
    "[bloqueos, conflictos o prerequisitos abiertos]"
  ]
}
```

---

## Paso 8 — Invocar a @QwikOrchestrator

Mensaje de handoff:

```text
@QwikOrchestrator

Reanudar trabajo desde /new-session.

Fuente de reentrada: [resume_source]
Feature/frente: [feature]
Snapshot usado: [snapshot o N/A]
INDEX validado: docs/sessions/INDEX.md
Artefactos cargados: [lista mínima]
Fase actual: [fase]
Siguiente paso exacto: [acción]
Agente sugerido tras routing: [agente]
Advertencias: [warnings]

Tarea:
1. Validar el routing con la información mínima cargada.
2. No explorar el repo masivamente.
3. No reabrir decisiones vigentes salvo contradicción documentada.
4. Enrutar al agente correcto.
5. Si INDEX requiere actualización, activar @QwikMemory antes de continuar.
```

---

## Salida esperada

```text
NEW SESSION READY

Feature/frente: [feature]
Fuente de reentrada: [prompt-snapshot / index-active-entry / index-snapshot / index-single-wip]
Snapshot usado: [ruta o N/A]
INDEX validado: sí / no
Artefactos cargados: [lista]
Fase actual: [fase]
Siguiente paso exacto: [acción]
Agente sugerido: [agente]
Advertencias: [si aplica]
Estado: READY FOR ORCHESTRATOR / BLOCKED
```

---

## Cuándo usar este prompt

- Al abrir un chat nuevo después de `/memory-compact`.
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

Si no puede hacerlo con evidencia, debe detenerse.
