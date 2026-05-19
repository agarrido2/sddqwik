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

## 2. Rutas del sistema: distribución vs uso real

En el repositorio de distribución, agentes, prompts e instrucciones pueden vivir bajo:

```text
github/
```

En un proyecto consumidor real, esa carpeta se instala o renombra como:

```text
.github/
```

### Regla

`github/` no es la ruta operativa final permanente.

Usar esta lectura:

```text
github/   → modo distribución
.github/  → modo instalado/operativo
```

---

## 3. Sistema de memoria en 4 capas

```text
L0 — Memoria procedimental
     .github/agents/*.agent.md
     .github/prompts/*.prompt.md
     .github/copilot-instructions.md
     → Quién soy, qué puedo hacer, cómo tomo decisiones

L1 — Memoria semántica
     docs/standards/*.md
     → Qwik, arquitectura, datos, seguridad, UX, RBAC, testing
     → Se lee bajo demanda, no completa por defecto

L2 — Memoria episódica/documental
     docs/prd/            → PRDs aprobados
     docs/blueprint/      → Blueprints aprobados
     docs/specs/          → Contratos funcionales
     docs/plans/          → Planes técnicos y handoffs
     docs/audits/         → Auditorías y veredictos
     docs/bugs/           → Incidencias y causa raíz
     docs/adr/            → Decisiones arquitectónicas
     docs/sessions/       → INDEX y snapshots operativos

L3 — Memoria de trabajo
     Ventana de contexto actual del modelo
     → Volátil, limitada y cara
     → Se compacta con /memory-compact
```

### Regla de INDEX

`docs/sessions/INDEX.md` es un artefacto estructural y debe poder versionarse.
Los snapshots de sesión pueden ser volátiles, pero el INDEX es la primera fuente de navegación operativa.

---

## 4. Ciclo completo SDD

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

## 5. Fase -1 — Blueprint

**Entrada:** `/blueprint [project]`
**Agente:** `@QwikBlueprint`
**Input:** PRD aprobado en `docs/prd/[project]-prd.md`
**Output:** `docs/blueprint/[project]-blueprint.md`

El Blueprint traduce el PRD en un mapa de ejecución:

- módulos funcionales;
- fases de entrega;
- dependencias;
- zonas pública/privada/admin/API;
- mapa preliminar de datos;
- riesgos y decisiones abiertas;
- orden recomendado de Specs.

### Gate

Sin PRD Approved, no hay Blueprint formal.
Sin Blueprint Approved, un proyecto modular grande no debería iniciar Specs de producción.

---

## 6. Fase 0 — Spec

**Entrada:** `/spec [feature]`
**Agente:** `@QwikSpeccer`
**Input:** Blueprint/PRD/contexto funcional suficiente
**Output:** `docs/specs/[feature].md`

La Spec define:

- propósito funcional;
- usuarios/roles afectados;
- Scope IN;
- Scope OUT;
- Acceptance Criteria funcionales binarios;
- Acceptance Criteria no funcionales;
- contratos de datos serializables;
- datos/RLS/permisos si aplica;
- estados loading/empty/error/unauthorized;
- riesgos y supuestos;
- criterios de auditoría.

### Gate

Sin Spec `Approved`, no se escribe código de feature.
Sin AC verificables, la Spec sigue en Review.

---

## 7. Fase 1 — Entrada segura a feature

**Entrada:** `/new-feature [feature]`
**Agente inicial:** prompt `/new-feature` + `@QwikOrchestrator`
**Input:** Spec Approved
**Output:** Plan File preparado o handoff a Architect

`/new-feature` no implementa código y no crea Spec.
Su función es abrir el ciclo de construcción de forma segura:

- verificar Spec Approved;
- consultar INDEX;
- revisar dependencias;
- crear o preservar `docs/plans/[feature].md`;
- dejar Pre-flight Gate Report;
- entregar a Orchestrator/Architect sin saltar directamente a Builder.

### Gate

Sin `/new-feature`, el sistema puede perder el rastro inicial del Plan.
Sin Plan técnico aprobado/listo, Builder no implementa.

---

## 8. Fase 2 — Plan técnico

**Agente:** `@QwikArchitect`
**Sub-agente si aplica:** `@QwikDBA`
**Input:** Spec Approved
**Output:** `docs/plans/[feature].md`

Architect traduce el WHAT en HOW:

- archivos esperados;
- fronteras `$()`;
- rutas y capas;
- servicios/dominio;
- estado serializable;
- riesgos;
- tests necesarios;
- datos/RLS si aplica.

DBA interviene cuando hay:

- schema;
- migraciones;
- queries;
- constraints;
- permisos;
- RLS;
- integridad o seguridad de datos.

### Gate

Si Architect no puede planificar sin ambigüedad, la Spec está incompleta o el Blueprint necesita revisión.
No se debe inventar lo que falta.

---

## 9. Fase 3 — Build

**Agente:** `@QwikBuilder`
**Input:** Spec Approved + Plan aprobado/listo + datos/RLS resueltos si aplican
**Output:** código + Delivery Summary verificable

Builder implementa el Plan.
No reinterpreta producto.
No amplía scope.
No inventa datos.
No parchea bugs fuera de scope.

Su entrega debe incluir:

- matriz AC → implementación → evidencia;
- archivos modificados;
- decisiones tomadas;
- tests/validación ejecutados o justificación;
- datos/RLS/seguridad si aplica;
- desviaciones y riesgos.

---

## 10. Fase 4 — Audit

**Agente:** `@QwikAuditor`
**Input:** Spec + Plan + Delivery Summary + código
**Output:** `docs/audits/[feature]-audit.md`

Auditor verifica tres planos:

1. cumplimiento funcional contra AC;
2. cumplimiento técnico contra Plan;
3. cumplimiento sistémico contra standards.

### Gate

Sin matriz AC completa y evidencia verificable, no hay `PASSED`.
Si hay issue crítico, no hay `PASSED`.
Si hay issue mayor incompatible con producción, no hay `PASSED`.

### Anti-loop

```text
Audit FAILED ciclo 1 → Builder
Audit FAILED ciclo 2 → Builder si scope sigue acotado
Audit FAILED ciclo 3+ → Architect
```

---

## 11. Fase 5 — Polish

**Agente:** `@QwikPolisher`
**Input:** Audit PASSED
**Output:** estado `PRODUCTION-READY` o bloqueo explícito

Polisher no cambia funcionalidad.
Su foco es production readiness:

- build/typecheck/test si existen scripts;
- performance;
- UX final;
- accesibilidad;
- higiene técnica;
- bundle sanity;
- documentación mínima de cierre.

### Gate

Sin Audit PASSED, Polisher no actúa.

---

## 12. Fase 6 — Memory

**Agente:** `@QwikMemory`
**Input:** feature lista, snapshot requerido o señal reusable
**Output:** INDEX/snapshot/Lessons/ADR según corresponda

Memory no guarda ruido.
Preserva continuidad operativa:

- actualizar `docs/sessions/INDEX.md`;
- crear snapshot si hace falta;
- generar Prompt de Reanudación;
- promover Lessons Learned si hay aprendizaje reusable;
- proponer ADR si hay decisión estructural.

### Gate

Una feature `PRODUCTION-READY` no está cerrada del todo hasta que Memory deja el estado navegable.

---

## 13. Flujos especiales

### Bugs

```text
/bug-fix [bug-id]
```

No se corrige un bug sin:

- observed/expected;
- reproducción o evidencia suficiente;
- diagnóstico;
- causa raíz o hipótesis explícita;
- clasificación;
- verificación.

### Legacy

```text
/legacy-audit [path]
```

Veredictos:

```text
APTO
CONDICIONADO
REFACTOR TOTAL
NO INCORPORAR
```

Sin veredicto, no se construye encima de legacy dudoso.

### Optimizer

```text
/optimizer-code [path]
```

Solo permite refactor local sin cambio funcional.
Si aparece bug, feature nueva, datos o arquitectura, redirigir al flujo correcto.

### Contexto

```text
/memory-compact
/new-session
```

`/memory-compact` guarda señal operativa.
`/new-session` reanuda desde Prompt de Reanudación, snapshot o INDEX.

---

## 14. Trazabilidad completa

Cada feature debe poder reconstruirse desde artefactos:

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

Esto no es burocracia.
Es el mecanismo que permite mantener un sistema generado con ayuda de IA sin perder control.

---

## 15. Anti-patrones SDD

| Anti-patrón | Síntoma | Consecuencia |
|---|---|---|
| Blueprint ausente | Specs sin orden ni dependencias | Retrabajo y features en orden incorrecto |
| Spec vaga | “Hacer que funcione X” | Builder improvisa y Auditor no puede verificar |
| Spec post-hoc | Spec escrita después del código | AC descriptivos, no contractuales |
| `/new-feature` omitido | Se salta pre-flight y Plan File | Pérdida de trazabilidad inicial |
| Plan sin Spec | Architect inventa el WHAT | Código correcto para problema equivocado |
| Builder sin Plan | Implementación por intuición | Deuda y scope creep |
| Auditor sin AC | Solo revisa calidad técnica | Feature técnicamente buena pero funcionalmente mala |
| Bug parcheado sin causa raíz | Fix rápido sin diagnóstico | Regresiones repetidas |
| Legacy incorporado sin audit | Se adopta deuda invisible | Riesgo estructural y seguridad débil |
| Memory no persistida | No se actualiza INDEX/snapshot | Pérdida de continuidad |

---

## 16. Pregunta de oro

Antes de iniciar cualquier tarea:

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

Si no hay contrato, no se improvisa.
Se crea o se recupera el contrato correcto.
