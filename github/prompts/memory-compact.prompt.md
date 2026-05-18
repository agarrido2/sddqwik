---
# EXTERNAL_AGENT_PATH: ".github/prompts/memory-compact.prompt.md"
name: memory-compact
description: >
  Compacta el contexto activo de una feature o frente de trabajo en un snapshot
  operativo mínimo, actualiza docs/sessions/INDEX.md y genera un prompt de
  reanudación seguro para /new-session. No resume conversaciones: preserva solo
  estado, decisiones, riesgos, artefactos y siguiente paso verificable.
tools: ["read", "edit", "execute/runInTerminal"]
argument-hint: "example: /memory-compact voice-agent-feature"
---

# 🧠 MEMORY COMPACT — `${input:featureName}`

## Propósito

`/memory-compact` convierte contexto activo y frágil en memoria operativa reanudable.

No es un resumen narrativo.
No copia la conversación.
No sustituye a `docs/sessions/INDEX.md`.
No explora el repositorio a ciegas.
No decide el siguiente diseño o fix por su cuenta.

Su objetivo es que un chat nuevo pueda continuar con contexto mínimo y fiable:

```text
Contexto activo → Snapshot operativo → INDEX actualizado → Prompt de reanudación → /new-session
```

---

## Regla operativa crítica

Antes de crear un snapshot, este prompt debe verificar:

1. existe un identificador de feature o frente de trabajo;
2. existe al menos un artefacto activo relacionado;
3. existe o se puede crear `docs/sessions/INDEX.md`;
4. se han cargado solo los artefactos mínimos necesarios;
5. el snapshot contiene estado operativo, no conversación;
6. el snapshot incluye siguiente paso exacto;
7. el snapshot incluye agente sugerido de reentrada;
8. el INDEX queda actualizado o se documenta por qué no pudo actualizarse;
9. el Prompt de Reanudación apunta al snapshot creado.

Si no hay artefactos activos ni contexto suficiente, detener.

---

## Prohibiciones

Durante `/memory-compact`, no hacer:

- leer todo el repositorio;
- listar todas las specs;
- listar todos los plans;
- cargar sesiones archivadas;
- leer código de aplicación salvo referencia directa en Plan, Audit o Bug;
- copiar conversación literal;
- guardar placeholders vacíos;
- inventar decisiones;
- cerrar features;
- marcar `Done` sin Audit/Polish/Memory de cierre;
- crear ADRs sin señal estructural clara;
- convertir cualquier detalle menor en memoria permanente.

---

## Paso 0 — Normalizar entrada

Usar `${input:featureName}` como identificador canónico de la feature o frente de trabajo.

Si está vacío, detener:

```text
MEMORY-COMPACT GATE BLOQUEADO: falta featureName o frente de trabajo.
Usa: /memory-compact [feature-name]
```

---

## Paso 1 — Verificar artefactos activos

Comprobar únicamente artefactos esperados y relacionados.

```bash
FEATURE="${input:featureName}"
SPEC_FILE="docs/specs/${FEATURE}.md"
PLAN_FILE="docs/plans/${FEATURE}.md"
AUDIT_FILE="docs/audits/${FEATURE}-audit.md"
POLISH_FILE="docs/audits/${FEATURE}-polish.md"
BUG_FILE="docs/bugs/${FEATURE}.md"
INDEX_FILE="docs/sessions/INDEX.md"

active_artifacts=0

for file in "$SPEC_FILE" "$PLAN_FILE" "$AUDIT_FILE" "$POLISH_FILE" "$BUG_FILE"; do
  if [ -f "$file" ]; then
    echo "ARTEFACTO ACTIVO: $file"
    active_artifacts=$((active_artifacts + 1))
  fi
done

if [ "$active_artifacts" -eq 0 ]; then
  echo "MEMORY-COMPACT GATE BLOQUEADO: no se encontraron artefactos activos para ${FEATURE}."
  echo "Revisa el nombre o ejecuta /setup para verificar el workspace."
  exit 1
fi
```

### Regla

No usar `ls docs/specs/` ni `ls docs/plans/` como mecanismo de búsqueda.
Solo comprobar rutas directamente derivadas de `${input:featureName}`.

---

## Paso 2 — Asegurar INDEX operativo

Si falta `docs/sessions/INDEX.md`, crearlo con estructura mínima.

```bash
mkdir -p docs/sessions
mkdir -p docs/sessions/archive

if [ ! -f "$INDEX_FILE" ]; then
  cat > "$INDEX_FILE" <<'EOF'
# Project Features Index

> Mantenido por @QwikMemory.
> Primera fuente de verdad operativa para @QwikOrchestrator.
> No es un resumen decorativo: filtra qué artefactos deben cargarse.

| Feature | Módulo | Estado | Depende de | Tablas DB | Expone | Artefactos | Resumen | Fecha |
|---------|--------|--------|------------|-----------|--------|------------|---------|-------|

EOF
  echo "CREADO docs/sessions/INDEX.md"
else
  echo "OK docs/sessions/INDEX.md"
fi
```

---

## Paso 3 — Carga mínima permitida

Cargar solo los artefactos existentes de esta lista:

```text
docs/specs/${input:featureName}.md
docs/plans/${input:featureName}.md
docs/audits/${input:featureName}-audit.md
docs/audits/${input:featureName}-polish.md
docs/bugs/${input:featureName}.md
docs/sessions/INDEX.md
```

### Cargar además solo si están referenciados explícitamente

- una Spec dependiente;
- un Plan dependiente;
- una ADR vigente;
- una entrada de `LESSONS-LEARNED.md` citada en Plan/Audit/Bug;
- un standard concreto si el snapshot debe preservar una decisión normativa.

### Regla

El snapshot debe apuntar a artefactos, no absorberlos por completo.

---

## Paso 4 — Extraer estado operativo

Identificar y preservar solo señal útil.

La señal útil es:

- fase actual;
- estado actual;
- agente que debe retomar;
- artefacto principal;
- próximos pasos exactos;
- decisiones vigentes;
- riesgos abiertos;
- bloqueos reales;
- tests/build/audit pendientes;
- dependencias confirmadas;
- archivos tocados solo si importan para reentrada;
- learnings reutilizables solo si reducen ambigüedad futura.

La señal no útil es:

- conversación literal;
- opiniones generales;
- explicaciones largas;
- texto copiado de standards;
- listas masivas de archivos;
- historial irrelevante;
- alternativas ya descartadas sin impacto;
- detalles que no cambian el siguiente paso.

---

## Paso 5 — Determinar fase y agente de reentrada

Usar esta tabla:

| Evidencia encontrada | Fase | Agente sugerido |
|---|---|---|
| Spec falta o no está aprobada | Spec | @QwikSpeccer |
| Spec Approved, sin Plan técnico | Planning | @QwikOrchestrator → @QwikArchitect |
| Plan técnico incompleto | Planning | @QwikArchitect |
| Plan listo, sin build | Build | @QwikBuilder |
| Build con Delivery Summary, sin Audit | Audit | @QwikAuditor |
| Audit FAILED ciclo 1-2 | Corrective Build | @QwikBuilder |
| Audit FAILED ciclo 3+ | Redesign | @QwikArchitect |
| Audit PASSED, sin Polish | Polish | @QwikPolisher |
| Polish PRODUCTION-READY, sin cierre memoria | Memory Close | @QwikMemory |
| Bug abierto sin diagnóstico | Bug Diagnosis | @QwikBugFix |
| Bug diagnosticado sin fix | Bug Fix | @QwikBuilder / @QwikArchitect / @QwikDBA según clasificación |
| Bug fix aplicado sin verificación | Bug Verification | @QwikBugFix / @QwikAuditor |
| Todo cerrado | Archived / Done | @QwikOrchestrator solo para siguiente trabajo |

### Regla

Si la fase no puede determinarse con evidencia, marcar:

```text
Fase: UNKNOWN
Agente sugerido: @QwikOrchestrator
Riesgo: requiere diagnóstico inicial desde INDEX y artefactos mínimos.
```

No inventar fase.

---

## Paso 6 — Crear snapshot operativo

Crear un snapshot con timestamp.

```bash
SNAPSHOT_TS=$(date +%Y%m%d-%H%M%S)
SNAPSHOT_FILE="docs/sessions/${FEATURE}-${SNAPSHOT_TS}.md"
```

Contenido obligatorio del snapshot:

```md
# Session Snapshot: ${FEATURE}

> Fecha: [YYYY-MM-DD HH:mm]
> Feature: ${FEATURE}
> Estado: [WIP / Blocked / Ready for Handoff / Done / Unknown]
> Fase actual: [fase]
> Agente recomendado: [agente]
> Snapshot creado por: /memory-compact

## 1. Resumen operativo

[Qué se estaba haciendo en 3-6 líneas máximo]

## 2. Artefactos fuente revisados

- Spec: [ruta o N/A]
- Plan: [ruta o N/A]
- Audit: [ruta o N/A]
- Polish: [ruta o N/A]
- Bug: [ruta o N/A]
- INDEX: docs/sessions/INDEX.md

## 3. Estado actual verificable

- Spec:
- Plan:
- Build:
- Audit:
- Polish:
- Bug:
- Memory:

## 4. Decisiones vigentes

- [decisión + artefacto que la respalda]

## 5. Riesgos o bloqueos abiertos

- [riesgo/bloqueo + impacto + agente responsable]

## 6. Dependencias y relaciones

- [dependencia + estado + fuente]

## 7. Siguiente paso exacto

[Una acción concreta, no una lista genérica]

## 8. Agente de reentrada

[agente sugerido y por qué]

## 9. Contexto que NO debe cargarse al reanudar

- [artefactos o zonas que no aportan al siguiente paso]

## 10. Señales para memoria duradera

- Lessons learned: [N/A o señal concreta]
- ADR candidate: [N/A o señal concreta]
- Bug formalizable: [N/A o señal concreta]

## 11. Prompt de Reanudación

/new-session

Reanudar feature: `${FEATURE}`
Snapshot prioritario: `docs/sessions/[snapshot-file].md`
Verificar contra índice: `docs/sessions/INDEX.md`
Objetivo inmediato: [siguiente paso exacto]
Agente sugerido tras reentrada: [@QwikOrchestrator | @QwikBuilder | @QwikAuditor | @QwikArchitect | @QwikDBA | @QwikBugFix | @QwikPolisher | @QwikMemory]
```

### Reglas del snapshot

- No dejar placeholders como `[pendiente]` en el snapshot final.
- Si algo no aplica, usar `N/A`.
- Si algo es desconocido, usar `UNKNOWN` y explicar qué artefacto debe verificar el siguiente agente.
- El resumen operativo no debe superar 6 líneas.
- El siguiente paso debe ser una acción concreta.
- El agente sugerido debe estar justificado por evidencia.

---

## Paso 7 — Actualizar INDEX

Actualizar o añadir la fila de `${input:featureName}` en `docs/sessions/INDEX.md`.

La fila debe dejar visible, como mínimo:

- feature;
- módulo o frente;
- estado;
- dependencias;
- tablas DB si aplica;
- servicios o exposición si aplica;
- artefactos principales;
- resumen breve de routing;
- fecha.

Formato recomendado:

```md
| ${FEATURE} | [módulo] | [estado] | [dependencias] | [tablas/N/A] | [expone/N/A] | Spec · Plan · Snapshot | [siguiente paso + agente] | [YYYY-MM-DD] |
```

### Reglas de actualización

- Si la fila existe, actualizarla; no duplicarla.
- Si no existe, añadirla.
- No eliminar features `Done`.
- No convertir el INDEX en snapshot.
- No incluir conversación.
- No incluir detalles largos.

---

## Paso 8 — Detectar promoción de memoria duradera

Durante compactación, marcar para `@QwikMemory` si aparece una señal real:

| Señal | Acción |
|---|---|
| Error repetido que volverá a ocurrir | Proponer entrada en `LESSONS-LEARNED.md` |
| Decisión estructural transversal | Proponer ADR |
| Bug resuelto informalmente | Proponer formalización en `docs/bugs/` |
| Patrón de implementación reusable | Proponer lesson, no copiar código completo |
| Deuda aceptada conscientemente | Registrar riesgo en snapshot/INDEX |

### Regla

No promover ruido.
Solo promover memoria si reduce ambigüedad futura.

---

## Paso 9 — Confirmación final al usuario

La respuesta final debe incluir:

```text
MEMORY COMPACT COMPLETADO — ${input:featureName}

Snapshot creado: docs/sessions/[snapshot-file].md
INDEX actualizado: sí / no / requiere revisión
Estado: [WIP / Blocked / Ready for Handoff / Done / Unknown]
Fase actual: [fase]
Siguiente agente: [agente]
Siguiente paso exacto: [acción]

Prompt de Reanudación:
/new-session
Reanudar feature: ${input:featureName}
Snapshot prioritario: docs/sessions/[snapshot-file].md
Verificar contra índice: docs/sessions/INDEX.md
Objetivo inmediato: [siguiente paso exacto]
Agente sugerido tras reentrada: [agente]
```

Si el objetivo de compactación era abrir un chat nuevo, recomendar explícitamente:

```text
Abre un chat nuevo y usa el Prompt de Reanudación anterior.
```

---

## Criterios de éxito

La compactación solo está bien hecha si:

- un chat nuevo puede reanudar sin leer la conversación anterior;
- el snapshot apunta a los artefactos correctos;
- el INDEX permite encontrar el trabajo sin explorar el repo;
- el siguiente paso es concreto;
- el agente de reentrada está claro;
- no se guardó ruido conversacional;
- no se cargaron artefactos masivos innecesarios.

---

## Regla final

`/memory-compact` no significa “resume todo”.

`/memory-compact` significa:

```text
Guarda solo la señal mínima necesaria para que otro agente pueda continuar correctamente.
```

Si el snapshot no reduce contexto futuro, la compactación ha fallado.
