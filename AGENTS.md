# SDD Qwik — Agent Orchestration Manifest

> Stack: Qwik + Qwik City + Bun + Supabase + Drizzle + Tailwind v4 + Context7
> Paradigm: Spec-Driven Development
> Version: spec-first-garrido

This manifest is the operational map of the SDD Qwik system.

It defines agent responsibilities, official entry points, handoffs, gates and memory rules.

---

## 1. Core principle

```text
No approved contract, no implementation.
```

The system exists to prevent AI from improvising product, architecture, data model or production readiness.

Every feature must move through explicit artifacts:

```text
Spec → Plan → Build → Audit → Polish → Memory
```

> Core rule: **No Spec Approved, no implementation.**

---

## 2. Official flow

```text
/setup
  ↓
/spec [feature]
  ↓
/new-feature [feature]
  ↓
@QwikOrchestrator
  ↓
@QwikArchitect creates Plan + Implementation Tasks
  ↓
@QwikDBA if Data/RLS is READY_FOR_DBA
  ↓
@QwikBuilder executes tasks
  ↓
@QwikAuditor
  ↓
@QwikPolisher
  ↓
@QwikMemory
```

Rules:

```text
/spec does not implement.
/new-feature does not implement.
Architect does not implement.
DBA does not build UI or product flow.
Builder does not invent scope.
Auditor does not fix.
Polisher does not change functionality.
Memory does not store noise.
```

---

## 3. Agents

| Agent | Role | Main artifact/output |
|---|---|---|
| `@QwikOrchestrator` | Router, gates, handoffs, context control | routing decision |
| `@QwikSpeccer` | Functional contract and verifiable AC | `docs/specs/[feature].md` |
| `@QwikArchitect` | Spec Approved → Technical Plan + Implementation Tasks | `docs/plans/[feature].md` |
| `@QwikDBA` | Data, schema, queries, migrations, permissions, RLS | DBA Delivery Summary |
| `@QwikBuilder` | Executes Implementation Tasks from approved Plan | code + Delivery Summary |
| `@QwikAuditor` | Evidence-based verification | `docs/audits/[feature]-audit.md` |
| `@QwikPolisher` | Production readiness after Audit PASSED | `docs/audits/[feature]-polish.md` |
| `@QwikBugFix` | Bug lifecycle and root cause routing | `docs/bugs/[bug-id].md` |
| `@QwikMemory` | INDEX, snapshots, closure, lessons, ADR signals | `docs/sessions/INDEX.md` + snapshots |

---

## 4. Official prompts

| Command | Owner | Purpose |
|---|---|---|
| `/setup` | Orchestrator | workspace health check |
| `/spec [feature]` | Speccer | functional contract with AC. **Main entry point.** |
| `/new-feature [feature]` | Orchestrator | safe entry into planning/build cycle from Spec Approved |
| `/bug-fix [bug-id]` | BugFix | bug report, diagnosis, fix routing, verification |
| `/legacy-audit [path]` | Auditor | legacy verdict before building on top |
| `/optimizer-code [path]` | Builder/Auditor | local refactor without hidden behavior change |
| `/memory-compact` | Memory | snapshot and Resume Prompt |
| `/new-session` | Memory → Orchestrator | resume from INDEX/snapshot |

`/feature` is obsolete. Use `/new-feature`.

---

## 5. Artifact map

| Artifact | Purpose |
|---|---|
| `docs/specs/[feature].md` | what must be built |
| `docs/plans/[feature].md` | how it will be built |
| `docs/audits/[feature]-audit.md` | verification evidence |
| `docs/audits/[feature]-polish.md` | production readiness evidence |
| `docs/bugs/[bug-id].md` | bug lifecycle trace |
| `docs/sessions/INDEX.md` | first operational navigation source |
| `docs/sessions/[feature]-[timestamp].md` | reentry snapshot |
| `docs/adr/ADR-[NNN]-[slug].md` | durable architectural decision |

---

## 6. Gate states

### Spec

```text
DRAFT → REVIEW → APPROVED | REJECTED
```

No feature code without Spec Approved.

### Plan

```text
DRAFT | BLOCKED | READY_FOR_DBA | READY_FOR_BUILD
```

Builder only acts on `READY_FOR_BUILD`.

### Data/RLS

```text
N/A | READY_FOR_DBA | RESOLVED | BLOCKED
```

Builder does not guess data, ownership or permissions.

### Audit

```text
PASSED | FAILED | PARTIAL | BLOCKED
```

No Polish without Audit PASSED.

### Polish

```text
PRODUCTION-READY | NEEDS-WORK | BLOCKED
```

No final closure without Polish evidence.

### Memory

```text
WIP | BLOCKED | READY_FOR_DBA | READY_FOR_BUILD | AUDIT_FAILED | PRODUCTION-READY | NEEDS-WORK | ARCHIVED | LEGACY
```

`docs/sessions/INDEX.md` must make the current state navigable.

### Bug

```text
OPEN | DIAGNOSING | READY_FOR_FIX | READY_FOR_ARCHITECT | READY_FOR_DBA | READY_FOR_SPECCER | FIX_IN_PROGRESS | VERIFYING | FIXED | MITIGATED | REJECTED | DUPLICATE | BLOCKED
```

No bug fix without root cause.

---

## 7. Canonical handoffs

### New feature

```text
/spec [feature]
→ Spec Review
→ user approval
→ /new-feature [feature]
→ Orchestrator
→ Architect creates Plan + Implementation Tasks
→ DBA if needed
→ Builder executes tasks
→ Auditor
→ Polisher
→ Memory
```

### Bug

```text
/bug-fix [bug-id]
→ BugFix creates/preserves bug report
→ Auditor diagnoses root cause
→ Builder/Architect/DBA/Speccer according to classification
→ Auditor verifies
→ Memory if reusable signal exists
```

### Legacy

```text
/legacy-audit [path]
→ Auditor verdict
→ APTO | CONDICIONADO | REFACTOR TOTAL | NO INCORPORAR
→ only then continue with normal flow or containment
```

### Refactor local

```text
/optimizer-code [path]
→ classify work
→ local refactor only if no hidden behavior/schema/design change
→ Auditor if sensitive
```

---

## 8. Orchestrator routing rules

The Orchestrator must first determine the correct contract.

| Situation | Correct next step |
|---|---|
| No INDEX | `/setup` or Memory initialization |
| Feature without Spec | `/spec [feature]` |
| Spec Draft/Review | `@QwikSpeccer` via `/spec` |
| Spec Approved, no Plan | `/new-feature [feature]` |
| Plan READY_FOR_DBA | `@QwikDBA` |
| Plan READY_FOR_BUILD | `@QwikBuilder` |
| Build delivered | `@QwikAuditor` |
| Audit FAILED cycle 1-2 | `@QwikBuilder` |
| Audit FAILED cycle 3+ | `@QwikArchitect` |
| Audit PASSED | `@QwikPolisher` |
| Polish PRODUCTION-READY | `@QwikMemory` |
| Bug report | `/bug-fix [bug-id]` |
| Legacy path | `/legacy-audit [path]` |
| Local refactor | `/optimizer-code [path]` |
| Context high | `/memory-compact` |
| New chat | `/new-session` |

---

## 9. Context policy

First source:

```text
docs/sessions/INDEX.md
```

Then load only the minimum artifacts needed for the current state.

Do not load by default:

```text
all specs
all plans
all audits
archived sessions
completed feature artifacts without dependency
large source folders without scope
```

Before Builder, reduce context to:

```text
active Spec
active Plan
data/RLS handoff if applicable
standards specifically required
current audit only if fixing audit issues
```

The exact data/schema location comes from standards, Plan, DBA and the actual project structure. Do not assume one universal path.

---

## 10. Standards map

| Standard | Primary consumers |
|---|---|
| `ARQUITECTURA-FOLDER.md` | Architect, Builder, Auditor |
| `PROJECT-RULES-CORE.md` | all agents |
| `SDD-WORKFLOW.md` | Orchestrator, Speccer |
| `DECISIONS-QWIK.md` | Architect, Builder, Auditor, Polisher |
| `DECISIONS-DATA.md` | DBA, Architect, Builder, Auditor |
| `DECISIONS-UI.md` | Builder, Polisher |
| `SERIALIZATION-CONTRACTS.md` | Architect, Builder, Auditor |
| `QUALITY-STANDARDS.md` | Builder, Auditor, Polisher |
| `SECURITY-POLICIES.md` | DBA, Builder, Auditor |
| `TESTING-POLICY.md` | Builder, Auditor, Polisher |
| `RBAC-ROLES-PERMISSIONS.md` | Speccer, Architect, DBA, Auditor |
| `UX-GUIDE.md` | Speccer, Builder, Polisher |
| `CONTEXT7-GUIDE.md` | all agents using external/library facts |
| `LESSONS-LEARNED.md` | Builder, Auditor, BugFix, Memory |

---

## 11. Anti-loop rules

```text
Auditor ↔ Builder: max 2 normal correction cycles.
Third critical failure: escalate to Architect.
BugFix Builder ↔ Auditor: max 2 cycles for same root cause.
Repeated bug: treat as design, contract, data or lesson signal.
```

---

## 12. Final readiness definition

A feature is not done when code compiles.

A feature is done only when:

```text
Spec Approved
Plan READY_FOR_BUILD or executed from resolved data path
Data/RLS resolved or N/A
Builder Delivery Summary complete
Audit PASSED with evidence
Polish PRODUCTION-READY with report
Memory INDEX updated
```

---

## 13. Final rule

The best SDD system does not make AI faster at guessing.
It makes guessing unnecessary.
