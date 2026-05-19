# SDD WORKFLOW — Spec-Driven Development en SDD Qwik

> **Propósito:** definir el proceso completo de Spec-Driven Development usado por SDD Qwik.
> **Audiencia:** desarrolladores, agentes IA y nuevos colaboradores.
> **Versión:** 2026.4

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
     docs/prd/
     docs/blueprint/
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
PRD Approved
  ↓
/blueprint [project]
  ↓
/spec [feature]
  ↓
/new-feature [feature]
  ↓
@QwikOrchestrator
  ↓
@QwikArchitect / @QwikDBA
  ↓
@QwikBuilder
  ↓
@QwikAuditor
  ↓
@QwikPolisher
  ↓
@QwikMemory
```

---

## 4. Blueprint

**Entrada:** `/blueprint [project]`  
**Agente:** `@QwikBlueprint`  
**Input:** PRD aprobado en `docs/prd/[project]-prd.md`  
**Output:** `docs/blueprint/[project]-blueprint.md`

El Blueprint traduce el PRD en módulos, fases, dependencias, zonas de aplicación, mapa preliminar de datos, riesgos, decisiones abiertas y orden recomendado de Specs.

**Gate:** sin PRD Approved no hay Blueprint formal. Sin Blueprint Approved, un proyecto modular grande no debería iniciar Specs de producción.

---

## 5. Spec

**Entrada:** `/spec [feature]`  
**Agente:** `@QwikSpeccer`  
**Output:** `docs/specs/[feature].md`

La Spec define propósito, usuarios, Scope IN, Scope OUT, Acceptance Criteria funcionales y no funcionales, contratos de datos, estados, riesgos y criterios de auditoría.

**Gate:** sin Spec `Approved`, no se escribe código de feature. Sin AC verificables, la Spec sigue en Review.

---

## 6. Entrada segura a feature

**Entrada:** `/new-feature [feature]`  
**Agente inicial:** prompt `/new-feature` + `@QwikOrchestrator`  
**Input:** Spec Approved  
**Output:** Plan File preparado o handoff a Architect

`/new-feature` no implementa código y no crea Spec. Verifica Spec Approved, consulta INDEX, revisa dependencias, crea o preserva `docs/plans/[feature].md` y entrega a Orchestrator/Architect sin saltar directamente a Builder.

**Gate:** sin Plan técnico aprobado/listo, Builder no implementa.

---

## 7. Plan técnico

**Agente:** `@QwikArchitect`  
**Sub-agente si aplica:** `@QwikDBA`  
**Output:** `docs/plans/[feature].md`

Architect traduce el WHAT en HOW: archivos esperados, fronteras `$()`, rutas, capas, servicios, estado serializable, riesgos, tests y datos/RLS si aplica.

DBA interviene cuando hay schema, migraciones, queries, constraints, permisos, RLS o integridad de datos.

---

## 8. Build

**Agente:** `@QwikBuilder`  
**Input:** Spec Approved + Plan aprobado/listo + datos/RLS resueltos si aplican  
**Output:** código + Delivery Summary verificable

Builder implementa el Plan. No reinterpreta producto, no amplía scope, no inventa datos y no parchea problemas fuera de scope.

Su entrega debe incluir matriz AC → implementación → evidencia, archivos modificados, decisiones, validación, datos/RLS si aplica, desviaciones y riesgos.

---

## 9. Audit

**Agente:** `@QwikAuditor`  
**Input:** Spec + Plan + Delivery Summary + código  
**Output:** `docs/audits/[feature]-audit.md`

Auditor verifica cumplimiento funcional contra AC, cumplimiento técnico contra Plan y cumplimiento sistémico contra standards.

Sin matriz AC completa y evidencia verificable, no hay `PASSED`.

```text
Audit FAILED ciclo 1 → Builder
Audit FAILED ciclo 2 → Builder si scope sigue acotado
Audit FAILED ciclo 3+ → Architect
```

---

## 10. Polish

**Agente:** `@QwikPolisher`  
**Input:** Audit PASSED  
**Output:** `PRODUCTION-READY` o bloqueo explícito

Polisher no cambia funcionalidad. Su foco es build, typecheck/test si existen scripts, performance, UX, accesibilidad, higiene técnica y bundle sanity.

---

## 11. Memory

**Agente:** `@QwikMemory`

Memory preserva continuidad operativa: actualiza `docs/sessions/INDEX.md`, crea snapshot si hace falta, genera Prompt de Reanudación, promueve Lessons Learned y propone ADR si hay decisión estructural.

Una feature `PRODUCTION-READY` no está cerrada del todo hasta que Memory deja el estado navegable.

---

## 12. Flujos especiales

```text
/bug-fix [bug-id]       → incidencias con diagnóstico, causa raíz y verificación
/legacy-audit [path]    → veredicto antes de construir encima de código heredado
/optimizer-code [path]  → refactor local sin cambio funcional
/memory-compact         → snapshot operativo y Prompt de Reanudación
/new-session            → reentrada desde Prompt de Reanudación, snapshot o INDEX
```

---

## 13. Trazabilidad completa

```text
Qué quería el cliente      → docs/prd/[project]-prd.md
Cómo se ordenó el proyecto → docs/blueprint/[project]-blueprint.md
Qué se aprobó construir    → docs/specs/[feature].md
Cómo se decidió construir  → docs/plans/[feature].md
Qué se implementó          → Delivery Summary del Plan
Qué se verificó            → docs/audits/[feature]-audit.md
Por qué una decisión existe→ docs/adr/ADR-NNN-*.md
Cómo se retoma             → docs/sessions/INDEX.md + snapshots
```

---

## 14. Anti-patrones SDD

| Anti-patrón | Síntoma | Consecuencia |
|---|---|---|
| Blueprint ausente | Specs sin orden ni dependencias | Retrabajo |
| Spec vaga | “Hacer que funcione X” | Builder improvisa |
| Spec post-hoc | Spec escrita después del código | AC descriptivos, no contractuales |
| `/new-feature` omitido | Se salta pre-flight y Plan File | Pérdida de trazabilidad |
| Plan sin Spec | Architect inventa el WHAT | Código para problema equivocado |
| Builder sin Plan | Implementación por intuición | Deuda y scope creep |
| Auditor sin AC | Solo revisa calidad técnica | Cumplimiento funcional débil |
| Memory no persistida | No se actualiza INDEX/snapshot | Pérdida de continuidad |

---

## 15. Pregunta de oro

```text
¿Tengo el contrato correcto para esta acción?
```

- Proyecto nuevo → PRD Approved + `/blueprint`
- Feature nueva → `/spec` Approved + `/new-feature`
- Implementación → Plan aprobado/listo
- Bug → `/bug-fix`
- Legacy → `/legacy-audit`
- Refactor local → `/optimizer-code`
- Contexto saturado → `/memory-compact`
- Chat nuevo → `/new-session`

Si no hay contrato, no se improvisa. Se crea o se recupera el contrato correcto.
