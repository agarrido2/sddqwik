---
# EXTERNAL_AGENT_PATH: ".github/prompts/new-feature.prompt.md"
name: new-feature
description: >
  Inicia el ciclo de ejecución de una Spec ya aprobada. Verifica INDEX,
  Spec Approved, estado del Plan técnico, Implementation Tasks y DBA/RLS antes de
  enrutar hacia @QwikOrchestrator, @QwikArchitect, @QwikDBA o @QwikBuilder.
  Si cualquier gate falla, detiene el flujo y recomienda el siguiente paso correcto sin escribir código.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /new-feature voice-agent-configuration"
---

# 🚀 FEATURE KICKOFF — `${input:featureName}`

## Propósito

`/new-feature` es la puerta de entrada al ciclo de ejecución de una Spec ya aprobada.

No crea la Spec.
No implementa código.
No sustituye al Orchestrator.
No salta gates SDD.

Su única responsabilidad es preparar una entrada segura para el flujo:

```text
INDEX → Spec Approved → Plan técnico → Implementation Tasks → DBA si aplica → Builder → Auditor → Polisher → Memory
```

PRD y Blueprint no forman parte del flujo operativo principal ni son gates para `/new-feature`.

Si la feature no está lista para construcción, `/new-feature` debe detenerse y explicar el siguiente paso correcto.

---

## Regla operativa crítica

Antes de invocar a `@QwikOrchestrator`, este prompt debe verificar:

1. existe `docs/sessions/INDEX.md` como primera fuente operativa;
2. existe `docs/specs/${input:featureName}.md`;
3. la Spec está en estado `🟢 Approved` o `Approved`;
4. no hay dependencias bloqueantes declaradas en el INDEX;
5. existe o se puede crear `docs/plans/${input:featureName}.md` sin sobrescribir trabajo previo;
6. si no hay Plan técnico, el routing debe ir a `@QwikArchitect`;
7. si el Plan no tiene `Implementation Tasks`, el routing debe ir a `@QwikArchitect`;
8. si hay datos/RLS pendientes, el routing debe ir a `@QwikDBA`;
9. `@QwikBuilder` solo puede entrar con Spec Approved, Plan técnico, Implementation Tasks y DBA resuelto o N/A.

Si falla INDEX, Spec Approved o dependencias bloqueantes, detener el flujo. Si falta Plan técnico, faltan Implementation Tasks o DBA/RLS está pendiente, registrar el estado y enrutar al agente correcto.

---

## Prohibiciones

Durante `/new-feature`, no hacer:

- escribir código en `src/`;
- modificar schema, migraciones o RLS;
- crear una Spec nueva;
- aprobar una Spec por cuenta propia;
- leer todas las specs;
- leer todos los plans;
- cargar sesiones archivadas;
- explorar masivamente el repositorio;
- sobrescribir un Plan File existente sin revisar su estado.

---

## Paso 0 — Normalizar entrada

Usar `${input:featureName}` como identificador canónico de la feature.

Si el nombre está vacío, es ambiguo o contiene espacios no intencionales, detener:

```text
NEW-FEATURE GATE BLOQUEADO: falta un nombre de feature válido.
Usa: /new-feature [feature-name]
```

---

## Paso 1 — Verificar INDEX operativo

`docs/sessions/INDEX.md` es la primera fuente de verdad operativa para routing, dependencias y carga selectiva.

```bash
FEATURE="${input:featureName}"
INDEX_FILE="docs/sessions/INDEX.md"

if [ ! -f "$INDEX_FILE" ]; then
  echo "NEW-FEATURE GATE BLOQUEADO: falta docs/sessions/INDEX.md"
  echo "Siguiente paso: ejecutar /setup para inicializar memoria operativa."
  exit 1
fi

echo "INDEX operativo: $INDEX_FILE"
```

### Regla

No sustituir el INDEX leyendo specs y plans a ciegas.
Si el INDEX falta, el siguiente paso correcto es `/setup`.

---

## Paso 2 — Verificar Spec aprobada

Ejecutar una comprobación acotada sobre la Spec esperada. Este es el primer gate de construcción.

```bash
SPEC_FILE="docs/specs/${FEATURE}.md"

if [ ! -f "$SPEC_FILE" ]; then
  echo "NEW-FEATURE GATE BLOQUEADO: no existe $SPEC_FILE"
  echo "Siguiente paso: ejecutar /spec ${FEATURE}"
  exit 1
fi

SPEC_STATUS=$(grep -E '^>?[[:space:]]*Estado:|^>?[[:space:]]*Status:' "$SPEC_FILE" | head -1 || true)

echo "Spec: $SPEC_FILE"
echo "Estado detectado: ${SPEC_STATUS:-NO ENCONTRADO}"

if ! echo "$SPEC_STATUS" | grep -Eiq '(🟢[[:space:]]*)?Approved|Aprobada|Aprobado'; then
  echo "NEW-FEATURE GATE BLOQUEADO: la Spec existe pero no está Approved."
  echo "Siguiente paso: volver a @QwikSpeccer con /spec ${FEATURE} o aprobar explícitamente la Spec antes de construir."
  exit 1
fi
```

### Regla

No continuar si la Spec no está aprobada.

Una Spec en `Draft`, `Review`, `Pending`, `Blocked` o sin estado explícito no habilita construcción.

---

## Paso 3 — Consultar contexto mínimo en INDEX

Consultar únicamente las entradas relevantes para `${input:featureName}`.

```bash
echo "Entradas relacionadas en INDEX:"
grep -Ei "(^\|[[:space:]]*${FEATURE}[[:space:]]*\|)|${FEATURE}" "$INDEX_FILE" || true
```

Revisar si la entrada o entradas relacionadas declaran:

- dependencias en columna `Depende de`;
- tablas DB que conviene reutilizar;
- servicios o APIs en columna `Expone`;
- estado previo `WIP`, `BLOCKED`, `FAILED`, `DONE` o `PRODUCTION-READY`.

### Regla

No cargar artefactos adicionales todavía.
El INDEX filtra; los artefactos amplían solo si el Orchestrator lo decide.

---

## Paso 4 — Comprobar dependencias bloqueantes

Si el INDEX declara dependencias para esta feature, verificar que no aparecen como `WIP`, `FAILED`, `BLOCKED` o sin estado claro.

Esta comprobación puede requerir lectura humana del resultado del grep anterior.
El prompt debe documentar el resultado en el Plan File.

Si hay dependencia bloqueante, detener:

```text
NEW-FEATURE GATE BLOQUEADO: existen dependencias no completadas.
Completa primero los módulos bloqueantes indicados en docs/sessions/INDEX.md.
```

Si no hay dependencias bloqueantes, continuar.

---

## Paso 5 — Crear o preservar Plan File

El Plan File es el artefacto de coordinación del ciclo. `/new-feature` puede crear un contenedor operativo inicial, pero no crea el Plan técnico ni las Implementation Tasks; eso corresponde a `@QwikArchitect`.

Ruta esperada:

```text
docs/plans/${input:featureName}.md
```

Crear `docs/plans/` si no existe.

```bash
PLAN_FILE="docs/plans/${FEATURE}.md"
mkdir -p docs/plans
```

### Si el Plan File no existe

Crear un Plan File inicial mínimo:

```bash
if [ ! -f "$PLAN_FILE" ]; then
  cat > "$PLAN_FILE" <<EOF
# Plan: ${FEATURE}

> Estado: 🟡 Planning
> Spec: docs/specs/${FEATURE}.md
> Fecha: $(date +%F)
> Agente inicial: /new-feature

## 1. Pre-flight Gate Report

| Gate | Estado | Evidencia | Acción |
|---|---|---|---|
| Spec existe | ✅ PASS | docs/specs/${FEATURE}.md | N/A |
| Spec Approved | ✅ PASS | ${SPEC_STATUS} | N/A |
| INDEX existe | ✅ PASS | docs/sessions/INDEX.md | N/A |
| Dependencias | ⚠️ REVIEWED | Revisar entrada INDEX relacionada | Documentar en fase Architect |
| Plan File | ✅ CREATED | docs/plans/${FEATURE}.md | N/A |
| Plan técnico | ⏳ PENDING | Pendiente de @QwikArchitect | Enrutar a @QwikArchitect |
| Implementation Tasks | ⏳ PENDING | Pendiente de @QwikArchitect | Enrutar a @QwikArchitect |
| Datos/RLS | ⏳ REVIEW | Pendiente de clasificación | Enrutar a @QwikDBA si aplica |

## 2. Contexto mínimo inicial

- Spec: docs/specs/${FEATURE}.md
- INDEX: docs/sessions/INDEX.md
- Plan: docs/plans/${FEATURE}.md

## 3. Dependencias detectadas desde INDEX

Pendiente de completar por @QwikOrchestrator tras lectura selectiva del INDEX.

## 4. Riesgos iniciales

Pendiente de completar por @QwikArchitect.

## 5. Handoff Log

### $(date +%F) — /new-feature → @QwikOrchestrator
- Contexto: docs/specs/${FEATURE}.md, docs/sessions/INDEX.md, docs/plans/${FEATURE}.md
- Tarea: iniciar routing controlado para construcción de feature con Spec aprobada.
- Scope: preparar handoff hacia @QwikArchitect.
- No tocar: código de aplicación, schema, migraciones, RLS o UI antes del Plan técnico y las Implementation Tasks.
- Condición de salida: routing decision registrado y handoff a @QwikArchitect si procede.
- Riesgos conocidos: validar dependencias y datos/RLS desde INDEX y Spec.

## 6. Routing Decision

Pendiente de @QwikOrchestrator.

## 7. Technical Plan

Pendiente de @QwikArchitect.

## 8. DBA Notes

Pendiente si aplica.

## 9. Implementation Tasks

Pendiente de @QwikArchitect.

## 10. DBA Gate

Estado: N/A / READY_FOR_DBA / RESOLVED / BLOCKED

Pendiente de clasificación por @QwikArchitect o @QwikDBA.

## 11. Builder Delivery Summary

Pendiente de @QwikBuilder.

## 12. Audit Cycles

Pendiente de @QwikAuditor.

## 13. Polish Notes

Pendiente de @QwikPolisher.

## 14. Memory Notes

Pendiente de @QwikMemory.

## 15. Final Status

Pendiente.
EOF
  echo "CREADO $PLAN_FILE"
fi
```

### Si el Plan File ya existe

No sobrescribirlo.

```bash
if [ -f "$PLAN_FILE" ]; then
  echo "Plan File existente: $PLAN_FILE"
  grep -E '^> Estado:|^> Status:' "$PLAN_FILE" | head -1 || true
fi
```

Si existe y está en estado `✅ Done`, `PRODUCTION-READY` o cerrado, detener salvo instrucción explícita del usuario:

```text
NEW-FEATURE GATE BLOQUEADO: ya existe un Plan File cerrado para esta feature.
Usa /bug-fix, /optimizer-code o una nueva Spec si necesitas cambiar comportamiento.
```

Si existe y está en `Planning`, `WIP` o similar, continuar como reentrada controlada.
Añadir un nuevo handoff log, no borrar contenido anterior.

---

## Paso 6 — Verificar Plan técnico, Implementation Tasks y DBA/RLS

Antes de invocar al Orchestrator, confirmar que el Plan File contiene las secciones mínimas para enrutar correctamente. Esta verificación no habilita Builder por sí sola; solo determina el siguiente handoff correcto.

```bash
missing_plan_sections=0

for section in "Pre-flight Gate Report" "Handoff Log" "Routing Decision" "Technical Plan" "Implementation Tasks" "DBA Gate" "Builder Delivery Summary" "Audit Cycles" "Polish Notes" "Memory Notes"; do
  if grep -q "$section" "$PLAN_FILE"; then
    echo "OK sección Plan: $section"
  else
    echo "FALTA sección Plan: $section"
    missing_plan_sections=$((missing_plan_sections + 1))
  fi
done

if [ "$missing_plan_sections" -gt 0 ]; then
  echo "NEW-FEATURE GATE ADVERTENCIA: el Plan File existe pero le faltan secciones operativas."
  echo "@QwikOrchestrator debe normalizarlo antes de handoff a @QwikArchitect."
fi
```

### Routing según estado del Plan

```text
Si no existe Plan técnico → @QwikArchitect.
Si el Plan técnico está vacío, pendiente o incompleto → @QwikArchitect.
Si no existen Implementation Tasks → @QwikArchitect.
Si las Implementation Tasks están vacías, pendientes o no son ejecutables → @QwikArchitect.
Si datos/RLS está en READY_FOR_DBA, pendiente DBA o BLOCKED → @QwikDBA.
Si datos/RLS es RESOLVED o N/A, y existen Spec Approved + Plan técnico + Implementation Tasks → @QwikBuilder.
```

`@QwikBuilder` solo puede entrar si todos estos gates están satisfechos:

```text
Spec Approved
Plan técnico existe
Implementation Tasks existen
DBA/RLS resuelto o N/A
```

---

## Paso 7 — Invocar a @QwikOrchestrator

Solo después de superar los gates anteriores, invocar a `@QwikOrchestrator`.

Mensaje de handoff:

```text
@QwikOrchestrator

Inicia el ciclo de construcción para la feature `${input:featureName}`.

Contexto mínimo permitido:
- Spec aprobada: docs/specs/${input:featureName}.md
- INDEX operativo: docs/sessions/INDEX.md
- Plan File: docs/plans/${input:featureName}.md

Tarea:
1. Leer primero docs/sessions/INDEX.md.
2. Leer la Spec aprobada.
3. Leer el Plan File creado o existente.
4. Confirmar si hay dependencias, datos/RLS o servicios relacionados en INDEX.
5. Registrar Routing Decision en el Plan File.
6. Enrutar hacia @QwikArchitect si el Plan técnico aún no existe.
7. Enrutar hacia @QwikArchitect si faltan Implementation Tasks ejecutables.
8. Enrutar hacia @QwikDBA si datos/RLS está pendiente, READY_FOR_DBA o BLOCKED.
9. Enrutar hacia @QwikBuilder solo si existe Spec Approved, Plan técnico, Implementation Tasks y DBA/RLS resuelto o N/A.

Restricciones:
- No escribir código.
- No modificar schema.
- No leer specs o plans no relacionados salvo dependencia confirmada en INDEX.
- No saltar a Builder.
- No ampliar scope funcional fuera de la Spec aprobada.

Builder gate obligatorio:

```text
Spec Approved + Plan técnico + Implementation Tasks + DBA/RLS resuelto o N/A
```

Flujo esperado:
@QwikArchitect → @QwikDBA si aplica → @QwikBuilder → @QwikAuditor → @QwikPolisher → @QwikMemory
```

---

## Salida esperada

```text
NEW-FEATURE PREFLIGHT — ${input:featureName}

INDEX: PASS / FAIL
Spec: PASS / FAIL
Spec status: Approved / no aprobado / no encontrado
Dependencias: OK / BLOQUEADAS / REVIEW REQUIRED
Plan File: CREATED / EXISTS / BLOCKED
Plan sections: OK / NEEDS NORMALIZATION
Plan técnico: EXISTS / MISSING / PENDING
Implementation Tasks: EXISTS / MISSING / PENDING
DBA/RLS: N/A / RESOLVED / READY_FOR_DBA / BLOCKED
Builder gate: READY / BLOCKED
Estado: READY FOR ORCHESTRATOR / READY FOR ARCHITECT / READY FOR DBA / READY FOR BUILDER / BLOCKED

Siguiente paso:
- @QwikOrchestrator
- @QwikArchitect
- @QwikDBA
- @QwikBuilder
- /spec ${input:featureName}
- /setup
- resolver dependencias bloqueantes
```

---

## Regla final

`/new-feature` no significa “programa esto”.

`/new-feature` significa:

```text
Esta feature ya tiene Spec aprobada.
Prepara una entrada segura al ciclo de construcción.
```

Si el sistema no puede demostrar eso, debe detenerse.
