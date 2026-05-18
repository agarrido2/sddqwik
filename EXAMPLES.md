# SDD Qwik — Ejemplos de Uso

Referencia de prompts reales para sacar el máximo rendimiento al sistema.
Copia, adapta y usa. Cada ejemplo incluye el contexto en el que aplicarlo.

---

## 🗺️ @QwikBlueprint — Del PRD al plano técnico

### Generar el Blueprint desde un PRD aprobado
```
/blueprint mieleshuelva

El PRD está en docs/prd/mieleshuelva-prd.md y ha sido aprobado por el cliente.
```

### Proyecto con múltiples roles y panel de administración
```
/blueprint plataforma-formacion

El PRD define tres tipos de usuario: alumno, instructor y admin.
Hay una zona pública (landing, catálogo), una zona privada por rol
y un panel de administración separado.
```

### Cuando el Blueprint necesita ajuste tras revisión
```
@QwikBlueprint He revisado el Blueprint en
docs/blueprint/mieleshuelva-blueprint.md y quiero mover el módulo
de campañas de Fase 1 a Fase 2 — es demasiado complejo para el MVP.
Actualiza las fases y el orden de Specs correspondiente.
```

---

## 📐 @QwikSpeccer — Escribir Specs

### Feature nueva simple
```
/spec agent-configuration

Necesito una pantalla donde el usuario pueda configurar el agente de voz:
nombre, idioma, voz seleccionada y prompt de sistema.
Solo los roles owner y admin pueden acceder.
```

### Feature con integración externa
```
/spec retell-webhook-handler

Necesito procesar los webhooks que envía Retell AI cuando una llamada
termina. Hay que guardar el resumen, la duración y el estado en la DB.
El endpoint debe ser público pero verificar la firma del webhook.
```

### Feature con lógica de negocio compleja
```
/spec call-quota-enforcement

Cuando una organización supera el límite de llamadas de su plan,
las nuevas llamadas deben bloquearse y notificar al owner por email.
Los límites dependen del plan de suscripción (Stripe).
Incluir un indicador visual en el dashboard con el consumo actual.
```

### Feature con restricciones RBAC multi-tenant
```
/spec team-member-management

Necesito una pantalla donde el owner de una organización pueda
invitar miembros, asignarles rol (admin o member) y revocar acceso.
Un admin puede gestionar members e invited, pero no puede
modificar ni eliminar a otros admins ni al owner.
Referencia: RBAC-ROLES-PERMISSIONS.md para la matriz de permisos.
```

### Revisar y refinar una Spec ya creada
```
@QwikSpeccer Revisa la Spec en docs/specs/agent-configuration.md.
Creo que falta un AC para el caso en que el usuario intente guardar
con un prompt de sistema vacío. Añádelo y actualiza el documento.
```

---

## 🏗️ @QwikArchitect — Planificar

### Arrancar el plan tras Spec aprobada
```
/feature agent-configuration
```

### Consulta arquitectónica antes de implementar
```
@QwikArchitect Tengo dudas sobre dónde poner la lógica de validación
del webhook de Retell. ¿Va en la ruta como routeAction$ o en un
server$ dentro de un servicio? Lee docs/specs/retell-webhook-handler.md
y dime cuál encaja mejor con nuestra arquitectura.
```

### Revisar un plan existente
```
@QwikArchitect Lee docs/plans/call-quota-enforcement.md y dime si
las fronteras $() están bien definidas o si hay riesgo de snapshot
size excesivo con los datos del plan de suscripción.
```

### Decisión arquitectónica que necesita ADR
```
@QwikArchitect Necesito decidir si la integración con Stripe debe
vivir en src/features/billing/ o en src/lib/services/stripe.ts.
Analiza los pros y contras y crea el ADR correspondiente en docs/adr/.
```

---

## 🗄️ @QwikDBA — Base de datos

### Schema nuevo
```
@QwikDBA Lee docs/plans/agent-configuration.md sección Datos.
Diseña el schema para la tabla agent_configs, genera la migración
y define las políticas RLS para que cada organización solo vea
sus propios agentes.
```

### Schema con RLS multi-nivel (owner + delegación a admin)
```
@QwikDBA Diseña el schema y las políticas RLS para la tabla
team_invitations. Las reglas son:
- El owner ve y gestiona todos los registros de su organización
- El admin puede ver, crear y revocar invitaciones de members e invited
- El admin NO puede modificar invitaciones de otros admins ni del owner
- El member solo ve su propia invitación
Genera la migración y usa los patrones canónicos de SECURITY-POLICIES.md.
```

### Optimizar una query existente
```
@QwikDBA La query que carga el listado de llamadas en
src/lib/services/call.service.ts está tardando mucho cuando hay
más de 1000 registros. Analízala y añade los índices necesarios.
```

### Añadir campo a tabla existente
```
@QwikDBA Necesito añadir el campo webhook_secret (text, not null)
a la tabla organizations. Genera la migración y actualiza las
políticas RLS si es necesario.
```

---

## 🦾 @QwikBuilder — Implementar

### Implementar tras plan aprobado
```
@QwikBuilder Lee docs/plans/agent-configuration.md completo y
docs/specs/agent-configuration.md. Implementa el checklist de
ejecución paso a paso. Empieza por los tipos y servicios antes
de tocar la UI.
```

### Implementar con TDD explícito
```
@QwikBuilder Lee docs/plans/call-quota-enforcement.md.
Implementa el servicio QuotaService siguiendo TESTING-POLICY.md:
1. Escribe primero los tests unitarios en src/lib/services/__tests__/quota.service.test.ts
2. Implementa el servicio hasta que los tests pasen
3. Documenta cualquier caso borde detectado en los tests
No toques la UI hasta que el servicio esté cubierto.
```

### Componente UI específico
```
@QwikBuilder Crea el componente AgentCard para mostrar en el listado
de agentes. Debe mostrar nombre, estado (activo/inactivo), número de
llamadas del mes y dos acciones: editar y eliminar.
Sigue el patrón de iconos de DECISIONS-UI.md y usa Tailwind v4.
```

### Integración con API externa
```
@QwikBuilder Implementa el server$ para llamar a la API de Retell AI
y crear un agente. Usa Context7 para verificar el endpoint actual.
Valida los parámetros con Zod antes de hacer la llamada.
Recuerda LL-007: safeParse como primera operación.
```

### Corregir implementación tras auditoría
```
@QwikBuilder Lee docs/audits/agent-configuration-audit.md.
Corrige los issues críticos marcados. El más importante es el
LL-006 en AgentList.tsx línea 47 — el closure está capturando
el objeto agent completo en lugar de solo agent.id.
```

---

## 🔍 @QwikAuditor — Auditar

### Auditoría manual de un componente
```
@QwikAuditor Audita el componente en
src/features/agents/components/AgentForm.tsx
contra QUALITY_STANDARDS.md y LESSONS_LEARNED.md.
Genera el reporte en docs/audits/agent-form-manual-audit.md.
```

### Verificar seguridad de una feature
```
@QwikAuditor Revisa todos los routeAction$ en
src/routes/(app)/dashboard/agents/
y verifica que tienen validación Zod, manejo de errores con
códigos ORCH_XXX y que no exponen mensajes técnicos al cliente.
```

### Auditoría de serialización con contratos
```
@QwikAuditor Verifica que los componentes en
src/features/billing/components/
no capturan objetos no serializables en closures $().
Aplica los contratos definidos en SERIALIZATION-CONTRACTS.md:
comprueba que todas las fronteras servidor-cliente usan POJOs
y que no se pasan instancias de clases (Date, Map, Set, etc.).
Presta especial atención a los onClick$ dentro de listas.
Genera el reporte en docs/audits/billing-serialization-audit.md.
```

### Auditoría de guards RBAC
```
@QwikAuditor Revisa todos los loaders y routeAction$ en
src/routes/(app)/dashboard/
y verifica que los guards de rol están implementados correctamente
según RBAC-ROLES-PERMISSIONS.md: comprueba que ningún rol inferior
puede acceder a rutas de roles superiores por URL directa.
```

---

## 🏁 @QwikPolisher — Producción

### Polish tras auditoría PASSED
```
@QwikPolisher La feature agent-configuration tiene auditoría PASSED
en docs/audits/agent-configuration-audit.md.
Ejecuta el análisis de bundle, verifica Core Web Vitals y
limpia código muerto. Cierra el Plan File con las métricas finales.
```

### Solo limpieza de código
```
@QwikPolisher Busca y elimina todos los console.log, TODOs sin
issue asociado e imports muertos en src/features/agents/.
No toques la lógica de negocio.
```

---

## 🐛 /bug-fix — Corregir bugs

### Bug con síntomas claros
```
/bug-fix agent-form-tabs-data-loss

Al guardar el formulario de configuración del agente desde la
pestaña "Avanzado", los campos de la pestaña "General" llegan
vacíos al servidor y falla la validación Zod.
```

### Bug de rendimiento
```
/bug-fix dashboard-slow-initial-load

El dashboard tarda más de 4 segundos en mostrar datos en producción.
En desarrollo va bien. Sospecho que hay queries N+1 en el loader
del layout pero no estoy seguro.
```

### Bug de seguridad (guard RBAC roto)
```
/bug-fix member-can-access-billing

Un usuario con rol member puede acceder a /dashboard/facturacion
si conoce la URL directamente. El guard no está funcionando.
Revisa contra RBAC-ROLES-PERMISSIONS.md para confirmar
qué roles deben tener acceso a esa ruta.
```

---

## 🔬 /optimizer-code — Refactorizar

### Fichero demasiado grande
```
/optimizer-code src/routes/(app)/dashboard/calls/index.tsx

El fichero tiene 340 líneas mezclando UI, lógica de filtros
y llamadas a servicios. Necesita segmentarse.
```

### Componente con deuda técnica
```
/optimizer-code src/features/agents/components/AgentConfigForm.tsx

Este componente fue escrito antes de los standards actuales.
Tiene lógica de negocio mezclada con UI y los handlers capturan
objetos completos en los closures.
```

---

## 🧠 @QwikMemory — Gestión de contexto

### Guardar estado antes de cerrar
```
/memory-compact agent-configuration

Voy a cerrar la sesión. Guarda el estado actual de la feature
agent-configuration para retomar mañana.
```

### Retomar sesión
```
@QwikOrchestrator Retoma la feature agent-configuration.
Contexto en docs/sessions/agent-configuration-20260318-1430.md
```

### Capturar decisión arquitectónica importante
```
@QwikMemory Crea un ADR para la decisión que acabamos de tomar:
usar server$ en lugar de routeAction$ para la integración con
Retell porque necesitamos streaming de respuesta.
```

---

## 🧭 @QwikOrchestrator — Coordinación

### Diagnóstico de estado del proyecto
```
/setup
```

### Retomar trabajo sin saber dónde estabas
```
@QwikOrchestrator ¿En qué estado está la feature call-quota-enforcement?
¿Qué agente debe actuar ahora?
```

### Verificar antes de empezar el día
```
@QwikOrchestrator Dame un resumen del estado del proyecto:
features en curso, bugs abiertos y audits pendientes.
```

---

## 🏚️ /legacy-audit — Código heredado

> **Output esperado:** `docs/audits/legacy-[módulo]-[fecha].md`

### Antes de añadir features a código existente
```
/legacy-audit src/features/auth

Voy a añadir autenticación con Google OAuth. Antes necesito
saber el estado del código de auth existente.
```

### Auditar todo un módulo
```
/legacy-audit src/routes/(app)/dashboard

Este código fue escrito hace 3 meses sin los standards actuales.
Necesito saber qué deuda técnica hay antes de construir encima.
```

---

## 🔐 RBAC & Seguridad

### Spec con restricciones de rol complejas
```
/spec organization-settings

La pantalla de configuración de organización tiene tres secciones:
- Datos generales (nombre, logo): solo owner y admin
- Facturación y plan: solo owner
- Gestión de miembros: owner puede todo; admin puede gestionar
  members e invited pero no a otros admins ni al owner
Referencia completa en RBAC-ROLES-PERMISSIONS.md.
```

### Auditoría de seguridad RLS completa
```
@QwikAuditor Realiza una auditoría de seguridad completa de las
políticas RLS en la DB. Para cada tabla verifica:
- Que existe política SELECT, INSERT, UPDATE y DELETE
- Que el org_id es siempre la clave de aislamiento tenant
- Que los cruces cross-table no rompen el aislamiento
Usa los tres patrones canónicos de SECURITY-POLICIES.md como referencia.
Genera el reporte en docs/audits/rls-security-audit-[fecha].md.
```

### Verificar guard antes de deploy
```
@QwikAuditor Antes del deploy de la feature team-member-management,
verifica que el middleware de auth en src/routes/(app)/
implementa correctamente los guards por rol definidos en
RBAC-ROLES-PERMISSIONS.md. Confirma que no hay rutas sin proteger.
```

---

## 💡 Combinaciones habituales

### Proyecto nuevo con cliente (flujo completo desde cero)
```
1. Reunión con cliente → Discovery Checklist (PRD_TEMPLATE.md Parte 0)
2. Completar docs/prd/[proyecto]-prd.md
3. Cliente aprueba el PRD
4. /blueprint [proyecto]         ← @QwikBlueprint genera el plano técnico
5. Aprobar el Blueprint
6. /spec [módulo-fase-0]         ← primer módulo según el Blueprint
7. /feature [módulo-fase-0]      ← ciclo SDD Qwik
8. Repetir 6-7 por cada módulo en orden de fases
```

### Flujo completo de una feature (dentro de un proyecto en marcha)
```
1. /spec nombre-feature          ← define qué
2. /feature nombre-feature       ← ejecuta el ciclo
   (apruebas Plan en cada fase)
3. PRODUCTION-READY ✅
```

### Feature con RBAC y DB (flujo con seguridad explícita)
```
1. /spec nombre-feature          ← incluir restricciones de rol en la Spec
2. /feature nombre-feature       ← @QwikArchitect planifica
3. @QwikDBA diseña RLS según SECURITY-POLICIES.md y RBAC-ROLES-PERMISSIONS.md
4. @QwikBuilder implementa (TDD primero en servicios)
5. @QwikAuditor audita guards RBAC + contratos de serialización
6. @QwikPolisher cierra la feature
```

### Bug complejo que resulta ser un problema de diseño
```
1. /bug-fix nombre-bug
   → @QwikAuditor diagnostica
   → detecta problema arquitectónico
   → escala a @QwikArchitect
2. @QwikArchitect revisa el Plan
3. /feature nombre-fix           ← feature de corrección con Spec
```

### Sesión de mantenimiento
```
1. /setup                        ← ver estado general
2. /legacy-audit src/features/X  ← auditar área con deuda
3. /optimizer-code [ficheros]    ← sanear uno a uno
4. /memory-compact               ← guardar estado al terminar
```

### Incorporarse a un proyecto en marcha
```
1. /setup                                ← foto del estado
2. @QwikOrchestrator ¿qué hay en curso?
3. leer docs/blueprint/[proyecto]-blueprint.md ← entender el plano global
4. leer docs/plans/[feature].md          ← entender el plan activo
5. @QwikOrchestrator retoma [feature]    ← continuar donde estaba
```
---

## 🔬 Evaluación del sistema SDD

Estos ejemplos cubren el uso de `@QwikSDDEvaluator`, el agente de meta-análisis externo al pipeline. No evalúa código — evalúa si el sistema SDD ha operado bien.

> Referencia: `docs/standards/SYSTEM-EVALUATION.md`
> Template de salida: `docs/templates/SYSTEM-SCORECARD.md`

---

### Evaluación de cierre de proyecto
@QwikSDDEvaluator Evalúa el sistema SDD del proyecto [nombre].
Periodo: [fecha inicio] — [fecha fin].

> El agente revisa specs, plans, audits, bugs y ADRs. Emite scorecard con veredicto global, KPIs, ficha por ejes y acciones recomendadas. Output en `docs/reports/system-scorecard-[fecha].md`.

---

### Evaluación periódica mensual
@QwikSDDEvaluator Evaluación mensual del sistema. Periodo: abril 2026.
Dame scorecard completa con nivel de madurez.


---

### Diagnóstico de fricción recurrente
@QwikSDDEvaluator Hay demasiado retrabajo entre Builder y Auditor
en las últimas 3 features. Diagnostica qué está fallando en el sistema.


> Output esperado: diagnóstico por eje con causa raíz y acciones concretas.

---

### Verificar si el proyecto puede ser benchmark del sistema
@QwikSDDEvaluator ¿Este proyecto puede considerarse un benchmark válido
del sistema SDD? Evalúa trazabilidad, convergencia y cierre.


> Output esperado: veredicto con nivel de madurez alcanzado (S0–S5) y justificación basada en artefactos.
