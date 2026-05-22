# SDD Qwik — Ejemplos de Uso

Ejemplos prácticos para usar SDD Qwik sin saltarse gates.

Regla base:

```text
Sin Spec Approved, no hay implementación.
/setup verifica el workspace.
/spec define el contrato funcional.
/new-feature abre el ciclo de construcción desde una Spec Approved.
Implementation Tasks viven dentro del Plan técnico.
Builder ejecuta las Implementation Tasks del Plan.
Auditor no aprueba sin evidencia.
```

---

## 1. Proyecto nuevo o primera feature

### Verificar workspace

```text
/setup
```

Resultado esperado:

```text
- verifica estructura operativa
- verifica prompts, agentes y standards
- verifica docs/sessions/INDEX.md
- recomienda /spec [feature] como siguiente paso si no hay feature activa
```

### Crear primera Spec

```text
/spec agent-configuration

Necesito una pantalla para configurar el agente de voz:
- nombre
- idioma
- voz seleccionada
- prompt de sistema

Solo owner y admin pueden acceder.
```

### Abrir construcción tras aprobación

```text
/new-feature agent-configuration
```

Resultado esperado:

```text
@QwikOrchestrator
→ @QwikArchitect crea Plan técnico + Implementation Tasks
→ @QwikDBA si aplica
→ @QwikBuilder ejecuta tasks
→ @QwikAuditor
→ @QwikPolisher
→ @QwikMemory
```

---

## 2. Crear una Spec

### Feature simple

```text
/spec agent-configuration

Necesito una pantalla para configurar el agente de voz:
- nombre
- idioma
- voz seleccionada
- prompt de sistema

Solo owner y admin pueden acceder.
```

### Feature con integración externa

```text
/spec retell-webhook-handler

Necesito procesar webhooks de Retell AI cuando una llamada termina.
Guardar resumen, duración y estado.
El endpoint será público, pero debe verificar firma del webhook.
```

### Feature con RBAC

```text
/spec team-member-management

Necesito que el owner pueda invitar miembros y asignar rol.
Admin puede gestionar members e invited, pero no admins ni owner.
Member no puede gestionar invitaciones.
Usar RBAC-ROLES-PERMISSIONS.md como referencia.
```

### Refinar una Spec en Review

```text
@QwikSpeccer Revisa docs/specs/agent-configuration.md.
Falta un AC para guardar con prompt de sistema vacío.
Añádelo y mantén la Spec en Review hasta aprobación.
```

---

## 3. Abrir construcción de feature

### Tras Spec Approved

```text
/new-feature agent-configuration
```

Resultado esperado:

```text
- verifica Spec Approved
- consulta docs/sessions/INDEX.md
- revisa dependencias
- crea o preserva docs/plans/agent-configuration.md
- pide a @QwikArchitect Plan técnico + Implementation Tasks si no existe
- deja Pre-flight Gate Report
- entrega a @QwikOrchestrator / @QwikArchitect
```

---

## 4. Plan técnico

### Pedir Plan a Architect si Orchestrator lo indica

```text
@QwikArchitect Lee docs/specs/agent-configuration.md.
Crea docs/plans/agent-configuration.md con archivos esperados, fronteras $(), datos, tests, riesgos, handoff a Builder y sección ## Implementation Tasks.
Cada task debe incluir ID, tipo, descripción, dependencias y evidencia esperada.
```

### Revisar tasks antes de Build

```text
@QwikOrchestrator Revisa docs/plans/agent-configuration.md.
Confirma si el Plan está READY_FOR_BUILD y si las Implementation Tasks son ejecutables por Builder.
```

### Duda arquitectónica puntual

```text
@QwikArchitect Tengo dudas sobre dónde poner la validación del webhook de Retell.
Lee docs/specs/retell-webhook-handler.md y decide si debe vivir en routeAction$, server$ o servicio de dominio.
No implementes código.
```

---

## 5. Datos y RLS

### Cambio de datos

```text
@QwikDBA Lee docs/plans/team-member-management.md.
Diseña las tablas, constraints y policies RLS necesarias para invitaciones por organización.
Deja Delivery Summary de datos en el Plan.
```

### Añadir campo persistente

```text
@QwikDBA Necesito añadir webhook_secret a organizations.
Revisa el Plan activo y DECISIONS-DATA.md.
Genera la propuesta de migración y policies si aplica.
```

---

## 6. Implementación

### Build desde Implementation Tasks

```text
@QwikBuilder Lee docs/specs/agent-configuration.md y docs/plans/agent-configuration.md.
Ejecuta pre-flight antes de editar.
Ejecuta las Implementation Tasks del Plan en orden de dependencias.
No inventes scope ni reordenes tasks sin indicación del Plan/Architect.
Deja Delivery Summary con matriz Task → implementación → evidencia y AC → cobertura.
```

### Build con TDD

```text
@QwikBuilder Lee docs/plans/call-quota-enforcement.md.
Implementa primero tests para el servicio de cuotas según TESTING-POLICY.md.
No toques UI hasta cubrir la lógica de negocio.
Sigue las Implementation Tasks del Plan.
```

### Corrección tras audit FAILED

```text
@QwikBuilder Lee docs/audits/agent-configuration-audit.md.
Corrige solo los issues críticos y mayores listados.
No amplíes scope.
Actualiza la matriz Task → implementación → evidencia y AC → cobertura.
```

---

## 7. Auditoría

### Auditoría de feature

```text
@QwikAuditor Audita agent-configuration.
Lee Spec, Plan, Implementation Tasks, Delivery Summary y código declarado por Builder.
No emitas PASSED sin matriz AC completa y evidencia verificable.
```

### Auditoría de seguridad/RBAC

```text
@QwikAuditor Verifica la feature team-member-management contra RBAC-ROLES-PERMISSIONS.md y SECURITY-POLICIES.md.
Comprueba guards, loaders, actions, RLS y ausencia de acceso por URL directa.
```

### Re-audit

```text
@QwikAuditor Re-audita agent-configuration tras las correcciones de Builder.
Comprueba issues abiertos del audit anterior y que no se introdujeron regresiones.
```

---

## 8. Polish

### Tras Audit PASSED

```text
@QwikPolisher La feature agent-configuration tiene Audit PASSED.
Ejecuta production readiness: build, typecheck/test si existen scripts, performance, UX, accesibilidad e higiene técnica.
No cambies funcionalidad.
```

### Limpieza final sin cambio funcional

```text
@QwikPolisher Revisa src/features/agents/.
Elimina imports muertos, console.log y TODOs sin issue asociado.
No cambies lógica de negocio ni comportamiento.
```

---

## 9. Bugfix

### Bug con síntomas claros

```text
/bug-fix agent-form-tabs-data-loss

Observed: al guardar desde la pestaña Avanzado, los campos de General llegan vacíos.
Expected: los datos de todas las pestañas deben preservarse.
Evidencia: error de validación Zod en submit.
```

### Bug de seguridad

```text
/bug-fix member-can-access-billing

Observed: usuario member puede acceder a /dashboard/billing si conoce la URL.
Expected: solo owner puede acceder.
Referencia: RBAC-ROLES-PERMISSIONS.md.
```

---

## 10. Legacy audit

### Antes de construir encima de código heredado

```text
/legacy-audit src/features/auth

Voy a añadir Google OAuth, pero antes necesito saber si el código actual de auth es APTO, CONDICIONADO, REFACTOR TOTAL o NO INCORPORAR.
```

### Auditar una zona amplia

```text
/legacy-audit src/routes/(app)/dashboard

Este código fue escrito antes de los standards actuales.
Necesito veredicto y riesgos antes de seguir construyendo encima.
```

---

## 11. Optimizer code

### Refactor local

```text
/optimizer-code src/features/agents/components/AgentConfigForm.tsx

El componente mezcla UI con lógica y handlers grandes.
Quiero refactor local sin cambio funcional.
```

### Fichero de ruta demasiado grande

```text
/optimizer-code src/routes/(app)/dashboard/calls/index.tsx

El fichero mezcla filtros, UI y llamadas a servicios.
Clasifica primero si es refactor local o si requiere Architect.
```

---

## 12. Memoria y reentrada

### Compactar antes de cerrar

```text
/memory-compact agent-configuration

Voy a cerrar la sesión.
Guarda estado operativo, riesgos, artefactos activos, siguiente paso y Prompt de Reanudación.
```

### Reanudar en chat nuevo

```text
/new-session agent-configuration

Retoma desde el Prompt de Reanudación o desde docs/sessions/INDEX.md.
No reconstruyas todo el proyecto.
```

---

## 13. Orchestrator

### Saber qué sigue

```text
@QwikOrchestrator Revisa el estado de agent-configuration.
Dime qué gate está abierto, qué artefactos debo leer y qué agente debe actuar ahora.
```

### Estado general

```text
/setup
```

`/setup` debe devolver health report y recomendación de siguiente paso.

---

## 14. Combinaciones habituales

### Proyecto nuevo completo

```text
1. /setup
2. /spec first-feature
3. Aprobar Spec
4. /new-feature first-feature
5. @QwikArchitect crea Plan técnico + Implementation Tasks
6. @QwikDBA resuelve datos/RLS si aplica
7. @QwikBuilder ejecuta las Implementation Tasks
8. @QwikAuditor verifica Spec + Plan + Tasks + Delivery Summary
9. @QwikPolisher cierra production readiness tras Audit PASSED
10. @QwikMemory actualiza INDEX/snapshot y Prompt de Reanudación
11. Repetir desde /spec para la siguiente feature
```

### Feature con datos y permisos

```text
1. /spec team-member-management
2. Aprobar Spec
3. /new-feature team-member-management
4. @QwikArchitect crea Plan técnico + Implementation Tasks
5. @QwikDBA resuelve datos/RLS
6. @QwikBuilder ejecuta las Implementation Tasks con Delivery Summary
7. @QwikAuditor valida AC, Tasks, RBAC, RLS y tests
8. @QwikPolisher cierra production readiness
9. @QwikMemory actualiza INDEX/snapshot
```

### Bug que resulta ser diseño

```text
1. /bug-fix billing-access-leak
2. QwikBugFix diagnostica causa raíz
3. Si el problema es de diseño, escala a @QwikArchitect
4. Architect ajusta Plan/Implementation Tasks o pide revisar Spec
5. Builder implementa fix acotado
6. Auditor verifica bug y regresiones
```

### Mantenimiento de deuda

```text
1. /setup
2. /legacy-audit src/features/old-module
3. /optimizer-code src/features/old-module/file.tsx
4. @QwikAuditor si el refactor toca lógica sensible
5. /memory-compact al terminar
```

---

## 15. Prompts anti-patrón que debes evitar

```text
Haz esta feature rápido.
Arregla esto como puedas.
Mejora este módulo entero.
Refactoriza todo el dashboard.
Implementa aunque no esté la Spec.
Pasa el audit aunque queden dudas.
```

Mejor:

```text
/spec [feature] con estos requisitos...
/new-feature [feature] cuando la Spec esté Approved.
/bug-fix [bug-id] con observed/expected/evidencia.
/optimizer-code [path] con scope local y sin cambio funcional.
```
