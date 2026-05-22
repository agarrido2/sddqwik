# SDD WORKFLOW — Spec-Driven Development en SDD Qwik

> **Propósito:** definir el proceso completo de Spec-Driven Development usado por SDD Qwik.
> **Audiencia:** desarrolladores, agentes IA y nuevos colaboradores.
> **Versión:** spec-first-garrido

---

## 1. Tesis central

> **La especificación es el código más importante que escribes.**

Los agentes IA pueden generar código rápido. El cuello de botella no es la velocidad de implementación, sino la claridad del contrato que gobierna esa implementación.

Un agente con una Spec vaga produce código aparentemente correcto para el problema equivocado.
Un agente con una Spec verificable produce código auditable para el problema correcto.

---

## 2. Sistema de memoria en 4 capas

```text
L0 — Memoria procedimental
     .github/agents/*.agent.md
     .github/prompts/*.prompt.md
     .github/copilot-instructions.md

L1 — Memoria semántica
     docs/standards/*.md

L2 — Memoria episódica/documental
     docs/specs/
     docs/plans/
     docs/audits/
     docs/bugs/
     docs/adr/
     docs/sessions/

L3 — Memoria de trabajo
     Ventana de contexto actual del modelo
```

`docs/sessions/INDEX.md` es un artefacto estructural y debe poder versionarse. Los snapshots de sesión pueden ser volátiles, pero el INDEX es la primera fuente de navegación operativa.

---

## 3. Ciclo completo SDD

```text
/setup
  ↓
/spec [feature]
  ↓
/new-feature [feature]
  ↓
@QwikOrchestrator
  ↓
@QwikArchitect crea Plan técnico + Implementation Tasks
  ↓
@QwikDBA si aplica
  ↓
@QwikBuilder ejecuta tasks
  ↓
@QwikAuditor
  ↓
@QwikPolisher
  ↓
@QwikMemory
```

**Regla central:** sin Spec `Approved`, no hay implementación.

El primer artefacto contractual del workflow operativo es la Spec. PRD y Blueprint no gobiernan el workflow operativo principal ni establecen gates obligatorios para construir features.

---

## 4. Spec

**Entrada:** `/spec [feature]`  
**Agente:** `@QwikSpeccer`  
**Output:** `docs/specs/[feature].md`

La Spec define el WHAT contractual: propósito, usuarios, Scope IN, Scope OUT, Acceptance Criteria funcionales y no funcionales, contratos de datos, estados, riesgos y criterios de auditoría.

**Gate:** sin Spec `Approved`, no se escribe código de feature. Sin AC verificables, la Spec sigue en Review.

---

## 5. Entrada segura a feature

**Entrada:** `/new-feature [feature]`  
**Agente inicial:** prompt `/new-feature` + `@QwikOrchestrator`  
**Input:** Spec Approved  
**Output:** Plan File preparado o handoff a Architect

`/new-feature` no implementa código y no crea Spec. Verifica Spec Approved, consulta INDEX, revisa dependencias, crea o preserva `docs/plans/[feature].md` y entrega a Orchestrator/Architect sin saltar directamente a Builder.

**Gate:** sin Plan técnico con Implementation Tasks, Builder no implementa.

---

## 6. Plan técnico

**Agente:** `@QwikArchitect`  
**Sub-agente si aplica:** `@QwikDBA`  
**Output:** `docs/plans/[feature].md`

Architect traduce la Spec a ejecución técnica: archivos esperados, fronteras `$()`, rutas, capas, servicios, estado serializable, riesgos, tests, datos/RLS si aplica y una lista ordenada de Implementation Tasks.

Las Implementation Tasks son el contrato de ejecución del Builder. Deben ser concretas, verificables, ordenadas y trazables a la Spec y al Plan.

DBA interviene cuando hay schema, migraciones, queries, constraints, permisos, RLS o integridad de datos.

---

## 7. Build

**Agente:** `@QwikBuilder`  
**Input:** Spec Approved + Plan técnico con Implementation Tasks + datos/RLS resueltos si aplican  
**Output:** código + Delivery Summary verificable

Builder ejecuta exclusivamente las tasks definidas. No reinterpreta producto, no amplía scope, no inventa datos y no parchea problemas fuera de scope.

Su entrega debe incluir matriz AC → task → implementación → evidencia, archivos modificados, decisiones, validación, datos/RLS si aplica, desviaciones y riesgos.

---

## 8. Audit

**Agente:** `@QwikAuditor`  
**Input:** Spec + Plan + Implementation Tasks + Delivery Summary + código + evidencia  
**Output:** `docs/audits/[feature]-audit.md`

Auditor verifica cumplimiento funcional contra AC, cumplimiento técnico contra Plan, ejecución contra Implementation Tasks y cumplimiento sistémico contra standards.

Sin matriz AC completa y evidencia verificable, no hay `PASSED`.

```text
Audit FAILED ciclo 1 → Builder
Audit FAILED ciclo 2 → Builder si scope sigue acotado
Audit FAILED ciclo 3+ → Architect
```

---

## 9. Polish

**Agente:** `@QwikPolisher`  
**Input:** Audit PASSED  
**Output:** `PRODUCTION-READY` o bloqueo explícito

Polisher no cambia funcionalidad. Su foco es build, typecheck/test si existen scripts, performance, UX, accesibilidad, higiene técnica y bundle sanity.

---

## 10. Memory

**Agente:** `@QwikMemory`

Memory preserva continuidad operativa: actualiza `docs/sessions/INDEX.md`, crea snapshot si hace falta, genera Prompt de Reanudación, promueve Lessons Learned y propone ADR si hay decisión estructural.

Una feature `PRODUCTION-READY` no está cerrada del todo hasta que Memory deja el estado navegable.

---

## 11. Flujos especiales

```text
/bug-fix [bug-id]       → incidencias con diagnóstico, causa raíz y verificación
/legacy-audit [path]    → veredicto antes de construir encima de código heredado
/optimizer-code [path]  → refactor local sin cambio funcional
/memory-compact         → snapshot operativo y Prompt de Reanudación
/new-session            → reentrada desde Prompt de Reanudación, snapshot o INDEX
```

---

## 12. Trazabilidad completa

```text
Qué se aprobó construir       → docs/specs/[feature].md
Cómo se decidió construir     → docs/plans/[feature].md
Qué ordena la ejecución       → Implementation Tasks del Plan
Qué se implementó             → Delivery Summary del Builder
Qué se verificó               → docs/audits/[feature]-audit.md
Por qué una decisión existe   → docs/adr/ADR-NNN-*.md
Cómo se retoma                → docs/sessions/INDEX.md + snapshots
```

---

## 13. Anti-patrones SDD

| Anti-patrón | Síntoma | Consecuencia |
|---|---|---|
| Spec ausente | Se intenta implementar desde conversación o intención vaga | Builder improvisa |
| Spec vaga | “Hacer que funcione X” | Builder improvisa |
| Spec post-hoc | Spec escrita después del código | AC descriptivos, no contractuales |
| `/new-feature` omitido | Se salta pre-flight y Plan File | Pérdida de trazabilidad |
| Plan sin Spec | Architect inventa el WHAT | Código para problema equivocado |
| Plan sin Implementation Tasks | Builder recibe intención técnica, no ejecución ordenada | Implementación desigual y difícil de auditar |
| Builder sin Plan | Implementación por intuición | Deuda y scope creep |
| Builder fuera de tasks | Cambios no trazables al contrato aprobado | Scope creep |
| Auditor sin AC o evidencia | Solo revisa calidad técnica | Cumplimiento funcional débil |
| Memory no persistida | No se actualiza INDEX/snapshot | Pérdida de continuidad |

---

## 14. Pregunta de oro

```text
¿Tengo el contrato correcto para esta acción?
```

- Feature nueva → `/spec` Approved + `/new-feature`
- Implementación → Plan técnico con Implementation Tasks
- Datos/RLS → `@QwikDBA` resuelto si aplica
- Bug → `/bug-fix`
- Legacy → `/legacy-audit`
- Refactor local → `/optimizer-code`
- Contexto saturado → `/memory-compact`
- Chat nuevo → `/new-session`

Si no hay contrato, no se improvisa. Se crea o se recupera el contrato correcto.
