---
# EXTERNAL_AGENT_PATH: ".github/prompts/new-feature.prompt.md"
name: new-feature
description: >
  Inicia el ciclo de construcción de una feature ya especificada. Verifica de
  forma interna que existe Spec aprobada, que el INDEX está disponible, que las
  dependencias no bloquean el trabajo y que existe un Plan File seguro antes de
  entregar el control a @QwikOrchestrator. Si cualquier gate falla, detiene el
  flujo y recomienda el siguiente paso correcto sin escribir código.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /new-feature voice-agent-configuration"
---

# 🚀 FEATURE KICKOFF — `${input:featureName}`

## Propósito

`/new-feature` es la puerta de entrada al ciclo de construcción de una feature.

No crea la Spec.
No implementa código.
No sustituye al Orchestrator.
No salta gates SDD.

Su única responsabilidad es preparar una entrada segura para el flujo:

```text
Spec Approved → Pre-flight → Plan File → @QwikOrchestrator → @QwikArchitect
```

Si la feature no está lista para construcción, `/new-feature` debe detenerse y explicar el siguiente paso correcto.

---

## Regla operativa crítica

Antes de invocar a `@QwikOrchestrator`, este prompt debe verificar:

1. existe `docs/specs/${input:featureName}.md`;
2. la Spec está en estado `🟢 Approved` o `Approved`;
3. existe `docs/sessions/INDEX.md`;
4. no hay dependencias bloqueantes declaradas en el INDEX;
5. existe o se puede crear `docs/plans/${input:featureName}.md` sin sobrescribir trabajo previo;
6. el Plan File contiene un `Pre-flight Gate Report`;
7. el Plan File contiene un `Handoff Log` inicial.

Si cualquier gate crítico falla, detener el flujo.

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

## Paso 1 — Verificar Spec aprobada

Ejecutar una comprobación acotada sobre la Spec esperada.

```bash
FEATURE="${input:featureName}"
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
  echo "Siguiente paso: volver a /spec ${FEATURE} y cerrar aprobación antes de construir."
  exit 1
fi
```

### Regla

No continuar si la Spec no está aprobada.

Una Spec en `Draft`, `Review`, `Pending`, `Blocked` o sin estado explícito no habilita construcción.

---

## Paso 2 — Verificar INDEX operativo

`docs/sessions/INDEX.md` es la primera fuente de verdad operativa para routing, dependencias y carga selectiva.

```bash
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

El Plan File es el artefacto de coordinación del ciclo.

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
- No tocar: código de aplicación, schema, migraciones, RLS o UI antes del Plan técnico.
- Condición de salida: routing decision registrado y handoff a @QwikArchitect si procede.
- Riesgos conocidos: validar dependencias y reutilización de tablas/servicios desde INDEX.

## 6. Routing Decision

Pendiente de @QwikOrchestrator.

## 7. Technical Plan

Pendiente de @QwikArchitect.

## 8. DBA Notes

Pendiente si aplica.

## 9. Builder Delivery Summary

Pendiente de @QwikBuilder.

## 10. Audit Cycles

Pendiente de @QwikAuditor.

## 11. Final Status

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

## Paso 6 — Verificar Plan File operable

Antes de invocar al Orchestrator, confirmar que el Plan File contiene las secciones mínimas.

```bash
missing_plan_sections=0

for section in "Pre-flight Gate Report" "Handoff Log" "Routing Decision" "Technical Plan" "Builder Delivery Summary" "Audit Cycles"; do
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
4. Confirmar si hay dependencias, tablas DB o servicios relacionados en INDEX.
5. Registrar Routing Decision en el Plan File.
6. Enrutar hacia @QwikArchitect si el Plan técnico aún no existe.
7. No invocar a @QwikBuilder hasta que exista Plan técnico ejecutable.

Restricciones:
- No escribir código.
- No modificar schema.
- No leer specs o plans no relacionados salvo dependencia confirmada en INDEX.
- No saltar a Builder.
- No ampliar scope funcional fuera de la Spec aprobada.

Flujo esperado:
@QwikArchitect → @QwikDBA si aplica → @QwikBuilder → @QwikAuditor → @QwikPolisher → @QwikMemory
```

---

## Salida esperada

```text
NEW-FEATURE PREFLIGHT — ${input:featureName}

Spec: PASS / FAIL
Spec status: Approved / no aprobado / no encontrado
INDEX: PASS / FAIL
Dependencias: OK / BLOQUEADAS / REVIEW REQUIRED
Plan File: CREATED / EXISTS / BLOCKED
Plan sections: OK / NEEDS NORMALIZATION
Estado: READY FOR ORCHESTRATOR / BLOCKED

Siguiente paso:
- @QwikOrchestrator
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
