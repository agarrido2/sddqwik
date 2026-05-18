# 📊 SYSTEM-EVALUATION — Marco de Evaluación SDD Qwik

> Versión: 1.0 — 2026
> Agente responsable: `@QwikSDDEvaluator`
> Tipo: Meta-análisis. Externo al pipeline de producción.

---

## Propósito

Este standard define cómo evaluar el desempeño metodológico del sistema SDD Qwik sobre una aplicación, módulo o entrega concreta.

No evalúa calidad intrínseca del código — eso pertenece a `@QwikAuditor`.
Evalúa si el sistema SDD ha operado con claridad, disciplina y trazabilidad real.

---

## Cuándo ejecutar una evaluación

- Al cerrar un proyecto o una fase relevante
- Cuando hay señales de retrabajo excesivo o fricciones repetidas
- Antes de incorporar un proyecto al sistema como caso de referencia
- De forma periódica (mensual o por release) en proyectos activos
- Cuando el usuario quiere saber si el sistema está funcionando bien

Comando: invocar `@QwikSDDEvaluator` directamente.

---

## Ejes de evaluación

| Eje | Pregunta clave |
|---|---|
| **Definición** | ¿Se entendió bien lo que había que construir? |
| **Planificación** | ¿Se diseñó bien antes de implementar? |
| **Ejecución** | ¿La construcción siguió el plan con disciplina? |
| **Auditoría** | ¿La verificación llegó a tiempo y con utilidad real? |
| **Trazabilidad** | ¿Se puede reconstruir el porqué del resultado? |
| **Convergencia** | ¿El sistema avanzó con fricción razonable? |
| **Gobernanza** | ¿Hubo control del proceso o improvisación? |
| **Cierre** | ¿La entrega quedó en estado limpio y defendible? |

---

## Escala de valoración

| Nivel | Significado |
|---|---|
| **A** | Excelente — evidencia sólida, proceso limpio, mínima fricción, trazabilidad alta |
| **B** | Bueno — proceso generalmente sano, algunos desajustes menores, retrabajo controlado |
| **C** | Aceptable con deuda — el sistema llegó al resultado pero con fricciones relevantes |
| **D** | Débil — proceso frágil, reactivo o poco gobernado; señales claras de mal funcionamiento |
| **E** | Deficiente — falta de control, documentación insuficiente, alta improvisación |

---

## Niveles de madurez del sistema

| Nivel | Criterio |
|---|---|
| **S0** | Sistema experimental — sin evidencia de uso real |
| **S1** | Sistema utilizable — funciona en proyectos pequeños con supervisión alta |
| **S2** | Sistema fiable — funciona en proyectos medianos con fricción aceptable |
| **S3** | Sistema gobernable — funciona en proyectos grandes con trazabilidad suficiente |
| **S4** | Sistema industrial — benchmark repetible, baja fricción, alta trazabilidad |
| **S5** | Sistema de referencia — demostrado en múltiples tipos de aplicación con evidencia acumulada |

### Criterios mínimos para S4

- Audit Pass en ciclo 1 o 2: >= 85%
- Test Compliance en servicios nuevos: 100%
- Bugs críticos escapados tras PASSED: 0 en periodo de 30 días
- Trazabilidad completa (Spec + Plan + Audit + cierre): 100% de features
- Recuperación de contexto funcional en `/new-session`
- Benchmarks superados en al menos 3 tipos de aplicación distintos

### Criterios mínimos para S5

Todo S4, más:
- Evidencia en al menos 5 tipos de aplicación distintos
- Sin degradación metodológica durante 3+ ciclos de mantenimiento
- `SYSTEM-SCORECARD.md` actualizada con historial de al menos 6 meses

---

## KPIs del sistema

| KPI | Definición | Objetivo |
|---|---|---|
| **Spec Quality Rate** | % de specs aprobadas sin reescritura mayor | >= 85% |
| **Plan Stability Rate** | % de planes que no requieren rediseño tras Build | >= 80% |
| **Audit Pass First Time** | % de features con PASSED en ciclo 1 | >= 70% |
| **Audit Pass Cycle 1-2** | % de features con PASSED en ciclo 1 o 2 | >= 90% |
| **Critical Defect Escape Rate** | % de bugs críticos detectados después de Audit PASSED | <= 3% |
| **Test Compliance Rate** | % de servicios nuevos con test obligatorio presente | 100% |
| **Traceability Rate** | % de features con Spec + Plan + Audit + cierre indexado | 100% |
| **Context Recovery Rate** | % de sesiones retomadas sin pérdida de contexto relevante | >= 90% |
| **Retrabajo estructural** | Features que escalaron a @QwikArchitect en ciclo 3+ | <= 10% |

---

## Fuentes de evidencia

El evaluador debe consultar artefactos en este orden:

1. `docs/prd/`
2. `docs/blueprint/`
3. `docs/specs/`
4. `docs/plans/`
5. `docs/audits/`
6. `docs/bugs/`
7. `docs/adr/`
8. `docs/sessions/` — solo para reconstrucción contextual
9. `src/` — solo como evidencia secundaria para confirmar incoherencias documentales

**Regla:** si faltan artefactos clave, reducir nivel de confianza de la evaluación.
No inventar evaluaciones fuertes sobre evidencia débil.

---

## Señales de sistema sano

- Blueprint útil antes de Specs
- Specs con AC claros y verificables
- Plans completos y estables
- Bajo retrabajo entre Builder y Auditor
- Pocos bugs evitables
- Auditorías consistentes
- ADRs cuando hay decisiones importantes
- Buena correlación entre intención, plan y resultado
- Cierre production-ready comprensible

---

## Señales de fallo sistémico

- Spec vaga o redactada post-hoc
- Plan que inventa requisitos no definidos en Spec
- Demasiados huecos resueltos durante Build
- Bugs que revelan falta de definición previa
- Auditoría usada como detector tardío de diseño
- Decisiones importantes sin rastro en L2
- Múltiples correcciones reactivas en cadena
- Cierre sin estado final claro
- Features que funcionan pero no tienen recorrido reconstruible

---

## Relación con otros documentos

| Documento | Relación |
|---|---|
| `QUALITY-STANDARDS.md` | Define calidad técnica del output — `@QwikAuditor` |
| `SDD-WORKFLOW.md` | Define el proceso que este standard evalúa |
| `AGENTS.md` | Manifiesto operativo del pipeline — externo a este standard |
| `SYSTEM-SCORECARD.md` | Template para registrar evaluaciones periódicas |
| `@QwikSDDEvaluator` | Agente que ejecuta las evaluaciones definidas aquí |

---

## Regla de uso

Este standard no es una checklist de auditoría técnica.
Es un marco para responder una sola pregunta:

> **¿Esta aplicación demuestra que el sistema SDD ha operado bien,
> o solo demuestra que se llegó a un resultado pese a fricciones del sistema?**