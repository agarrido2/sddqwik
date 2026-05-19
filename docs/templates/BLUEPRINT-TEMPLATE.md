# Blueprint: [project]

> Status: DRAFT | REVIEW | APPROVED | REJECTED
> PRD: `docs/prd/[project]-prd.md`
> Owner: @QwikBlueprint
> Updated: [YYYY-MM-DD]

---

## 0. How to use this document

This Blueprint converts a PRD Approved into a project execution map.

It answers:

```text
What modules exist?
In what order should they be specified?
What dependencies and risks shape delivery?
What is the first correct /spec?
```

It does not implement.
It does not create detailed Specs.
It does not replace Architect.
It does not design final schema/RLS.

Official flow:

```text
PRD Approved
  ↓
/blueprint [project]
  ↓
Blueprint REVIEW
  ↓
Blueprint APPROVED by user
  ↓
/spec [first-spec]
  ↓
/new-feature [feature]
```

---

## 1. Project summary

### Objective

[What the project must achieve.]

### Product scope

[Short scope summary from PRD.]

### MVP definition

[What must exist for the first usable release.]

### Explicitly post-MVP

[What is intentionally deferred.]

---

## 2. PRD validation

| Item | Status | Notes |
|---|---|---|
| PRD exists | PASS/FAIL | |
| PRD Approved | PASS/FAIL | |
| Users/roles understandable | PASS/FAIL/N/A | |
| Core flows understandable | PASS/FAIL | |
| Data/security expectations understandable | PASS/FAIL/N/A | |
| Critical decisions open | yes/no | |

### Assumptions

-

### Open decisions

| Decision | Why it matters | Owner | Required before |
|---|---|---|---|

---

## 3. Users, roles and zones

| Actor/Role | Zone | Main goals | Permission notes |
|---|---|---|---|
| [role] | public/auth/private/admin/API | | |

Application zones:

```text
public:
auth:
private/app:
admin:
API/webhooks:
```

---

## 4. Module map

| Module | Slug | Type | Complexity | Phase | Dependencies | Notes |
|---|---|---|---|---|---|---|
| [Module] | [module-slug] | core/admin/auth/data/integration/ux/infra/reporting | simple/medium/complex | 0/1/2/3+/post-MVP | | |

Module rules:

```text
A module is a coherent product capability.
A module may become one or more Specs.
Do not group half the product into one module.
Do not create technical modules that the PRD does not justify.
```

---

## 5. Delivery phases

### Phase 0 — Foundations

Goal:

Modules:

Exit criteria:

### Phase 1 — MVP

Goal:

Modules:

Exit criteria:

### Phase 2 — Main expansion

Goal:

Modules:

Exit criteria:

### Phase 3+ — Later phases

Goal:

Modules:

Exit criteria:

### Post-MVP / Out of first delivery

-

---

## 6. Dependency map

```text
[module-a]
  ↓
[module-b]
```

### Blocking dependencies

| Dependency | Blocks | Reason | Resolution |
|---|---|---|---|

### Parallelizable work

-

---

## 7. Data and security map

This section is conceptual. Final data design belongs to Architect/DBA during feature planning.

| Area/Module | Probable entities | Ownership | Sensitive? | DBA likely? | Notes |
|---|---|---|---|---|---|
| [area] | [entity names] | user/org/workspace/tenant/system/public | yes/no/unknown | yes/no/unknown | |

Rules:

```text
Do not define final schema here.
Do not define RLS SQL here.
Do not fix one universal schema path here.
Mark modules that likely require DBA.
```

---

## 8. Integrations

| Integration | Purpose | Status | Risk | Resolution point |
|---|---|---|---|---|
| [service/library] | | verified/pending/manual-check | | Spec/Plan/DBA |

If an integration cannot be verified, do not mark it as a closed decision.

---

## 9. Spec Queue

| Order | Spec slug | Module | Phase | Depends on | Why now | Notes |
|---|---|---|---|---|---|---|
| 1 | [first-spec] | [module] | 0/1 | | | |

Rules:

```text
Every row should be executable as /spec [slug].
First Spec must be small enough to specify clearly.
Split large modules into multiple Specs.
Mark dependencies explicitly.
```

---

## 10. First recommended Spec

```text
/spec [first-spec]
```

Reason:

Expected prerequisites:

---

## 11. Risks and mitigations

| Risk | Type | Impact | Mitigation | Owner |
|---|---|---|---|---|
| [risk] | product/technical/data/security/integration/scope | low/medium/high | | |

---

## 12. Approval

- Status: REVIEW | APPROVED | REJECTED
- Approved by:
- Approval date:
- Notes:

---

## 13. Next step after approval

If approved:

```text
/spec [first-spec]
```

Do not start `/new-feature` until that Spec is Approved.
