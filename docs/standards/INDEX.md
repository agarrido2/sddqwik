# Standards Index — SDD Qwik V3.0

> 14 ficheros. Cada uno cubre decisiones del proyecto y reglas operativas del sistema.
> Para APIs de librerías externas → Context7 (`CONTEXT7-GUIDE.md`).

---

## Los 14 Standards

| Fichero | Qué contiene | Agente principal |
|---|---|---|
| 01 ARQUITECTURA-FOLDER.md | Estructura de carpetas, Orchestrator Pattern, reglas por capa | `@QwikArchitect` |
| 02 PROJECT-RULES-CORE.md | Stack, SDD, Bun commands, resolución de conflictos, TDD | `@QwikArchitect` |
| 03 SDD-WORKFLOW.md | Proceso SDD, modelo de 4 capas de memoria, anti-patrones | `@QwikSpeccer` `@QwikOrchestrator` |
| 04 DECISIONS-QWIK.md | Reglas del proyecto sobre Qwik: primitivas, closures, co-localización QRL | `@QwikBuilder` |
| 05 DECISIONS-DATA.md | Reglas del proyecto sobre datos: Drizzle, Supabase, RLS, naming | `@QwikDBA` `@QwikBuilder` |
| 06 DECISIONS-UI.md | Reglas del proyecto sobre UI: Tailwind v4, dark mode, SVG icons | `@QwikBuilder` |
| 07 SERIALIZATION-CONTRACTS.md | Contratos de serialización, fronteras `$`, noSerialize | `@QwikBuilder` `@QwikAuditor` |
| 08 QUALITY-STANDARDS.md | 5 pilares, checklist de auditoría, SEO/A11Y y observabilidad | `@QwikAuditor` |
| 09 SECURITY-POLICIES.md | 3 patrones RLS canónicos, anti-patrones y checklist de verificación | `@QwikDBA` `@QwikAuditor` |
| 10 TESTING-POLICY.md | Protocolo de tests obligatorios y criterios de cobertura | `@QwikBuilder` `@QwikAuditor` |
| 11 UX-GUIDE.md | Principios UX, estados del sistema, microcopy, prevención de errores | `@QwikBuilder` `@QwikPolisher` |
| 12 RBAC-ROLES-PERMISSIONS.md | Roles owner/admin/member/invited, matriz de permisos y guards | `@QwikArchitect` |
| 13 CONTEXT7-GUIDE.md | Library IDs verificados y cómo usar Context7 correctamente | Todos |
| 14 LESSONS-LEARNED.md | Errores reales del proyecto y reglas derivadas. Top lecciones activas | `@QwikBuilder` `@QwikAuditor` |

---

## Qué leer según la tarea

| Tarea | Standards/contexto a cargar |
|---|---|
| Entrada operativa principal | `SDD-WORKFLOW.md` + `PROJECT-RULES-CORE.md`; estado en `docs/sessions/INDEX.md` |
| Flujo de feature spec-first | Spec → Plan → Implementation Tasks → Build → Audit → Polish → Memory |
| Nueva feature | `ARQUITECTURA-FOLDER.md` + `PROJECT-RULES-CORE.md` + `RBAC-ROLES-PERMISSIONS.md` si hay usuarios |
| Implementar componente UI | `DECISIONS-QWIK.md` + `DECISIONS-UI.md` + `SERIALIZATION-CONTRACTS.md` |
| Cambios en DB / schema | `DECISIONS-DATA.md` + `SECURITY-POLICIES.md` + `TESTING-POLICY.md` |
| Auditoría de código | `SERIALIZATION-CONTRACTS.md` + `QUALITY-STANDARDS.md` |
| Auditoría de seguridad (DBA activó RLS) | `QUALITY-STANDARDS.md` + `SECURITY-POLICIES.md` + `TESTING-POLICY.md` + `RBAC-ROLES-PERMISSIONS.md` |
| Antes de implementar | `LESSONS-LEARNED.md` |
| Feature con usuarios/roles | `RBAC-ROLES-PERMISSIONS.md` + `SECURITY-POLICIES.md` |
| Usar librería externa | `CONTEXT7-GUIDE.md` primero |
| Dudas de proceso SDD | `SDD-WORKFLOW.md` |

---

## Regla de uso

Estos ficheros contienen **decisiones del proyecto y reglas del sistema**, no documentación general de librerías.  
Para APIs externas, sintaxis actual o validación de integraciones, usar Context7.

## Regla de naming

Los nombres de standards deben mantener naming consistente con guiones.  
No mezclar guiones, underscores ni variantes legacy dentro del sistema.