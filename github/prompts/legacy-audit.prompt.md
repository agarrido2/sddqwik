---
# EXTERNAL_AGENT_PATH: ".github/prompts/legacy-audit.prompt.md"
name: legacy-audit
description: >
  Auditoría completa de código heredado. Analiza, clasifica deuda técnica y
  genera un plan de saneamiento priorizado antes de construir sobre él.
tools: ["read", "edit", "execute/runInTerminal", "upstash/context7/*"]
argument-hint: "example: /legacy-audit src/features/auth"
---

# 🏚️ LEGACY AUDIT PROTOCOL: `${input:legacyPath}`

> **Prerequisito:** Si `${input:legacyPath}` no se proporciona, no existe en el workspace o no es accesible, responde con:
> `"La ruta '${input:legacyPath}' no es válida o no se puede acceder. Verifica la ruta e inténtalo de nuevo."` y detén la operación.

**Objetivo:** Auditar el código heredado en `${input:legacyPath}`, clasificar toda la deuda técnica encontrada y generar un plan de saneamiento priorizado antes de construir nada nuevo sobre él.

> ⚠️ **Regla de oro:** No se construye sobre código sin un reporte de auditoría completo y aprobado. Este prompt
> es el prerequisito obligatorio para cualquier `/feature` que toque código
> existente.

**Flujo obligatorio:**

```
/legacy-audit → @QwikAuditor (análisis) → docs/audits/legacy-[path]-audit.md → Plan de saneamiento → ✅
```

> Este prompt enruta directamente a `@QwikAuditor` como excepción declarada
> al flujo de entrada único del Orchestrator.

---

## Paso 1: Crear el Artefacto de Auditoría

```bash
mkdir -p docs/audits
```

Crea el archivo `docs/audits/legacy-${input:legacyPath}-audit.md`:
```md
# Legacy Audit: ${input:legacyPath}

> Estado: 🔴 Auditando
> Created: [YYYY-MM-DD]
> Last Updated: [YYYY-MM-DD]

## 📁 Scope Auditado

- **Ruta:** ${input:legacyPath}
- **Archivos analizados:** [listar]
- **Agentes originales:** [si se conocen]

## 🔴 Deuda Crítica (Bloquea construcción)

[Violaciones que impiden añadir código nuevo de forma segura]

## 🟠 Deuda Mayor (Debe resolverse pronto)

[Violaciones importantes que generan riesgo técnico acumulado]

## 🟡 Deuda Menor (Mejora recomendada)

[Mejoras de calidad, nomenclatura, organización]

## ✅ Lo que está bien

[Patrones correctos que deben preservarse]

## 🗺️ Plan de Saneamiento

| Prioridad | Archivo | Problema | Acción | Agente |
|-----------|---------|----------|--------|--------|

## 📋 Veredicto Final

> [ ] 🟢 APTO — Se puede construir sobre este código tras fixes menores
> [ ] 🟠 CONDICIONADO — Requiere saneamiento previo antes de nueva feature
> [ ] 🔴 REFACTOR TOTAL — Rediseño con @QwikArchitect obligatorio
```


---

## Paso 2: Invocar a @QwikAuditor

```
@QwikAuditor audita todo el código en `${input:legacyPath}`.
El reporte está en `docs/audits/legacy-${input:legacyPath}-audit.md`.

Analiza cada archivo contra estos estándares en orden:

1. `docs/standards/QUALITY-STANDARDS.md` — Resumabilidad O(1), Zod, Seguridad, Observabilidad
2. `docs/standards/SERIALIZATION-CONTRACTS.md` — Fronteras $()
3. `docs/standards/ARQUITECTURA-FOLDER.md` — SoC, capas, Orchestrator Pattern
4. `docs/standards/DECISIONS-QWIK.md` — Sintaxis idiomática Qwik, primitivas, closures
5. `docs/standards/LESSONS-LEARNED.md` — Bloque Top Lecciones como checklist adicional
6. `docs/standards/TESTING-POLICY.md` — ¿Existen tests para los servicios auditados?
7. `docs/standards/SECURITY-POLICIES.md` — Si hay tablas con datos de usuario: ¿tiene RLS?

Para cada violación encontrada, clasifícala como:
- 🔴 CRÍTICA: Rompe resumabilidad, expone datos sensibles o mezcla capas graves
- 🟠 MAYOR: Deuda técnica significativa (lógica en rutas, tipos duplicados, sin Zod)
- 🟡 MENOR: Mejoras de calidad (nombres, organización, comentarios)

Rellena todas las secciones del reporte y emite el Veredicto Final.
Si detectas violaciones críticas de arquitectura, marca como REFACTOR TOTAL.
```

---

## Paso 3: Interpretar el Veredicto y Actuar

Usa esta tabla para identificar la acción según el veredicto de `@QwikAuditor`:

| Veredicto | Significado | Acción inmediata |
|---|---|---|
| 🟢 **APTO** | Sin bloqueos; deuda menor o inexistente | Continúa con `/feature` o `/bug-fix` directamente |
| 🟠 **CONDICIONADO** | Deuda 🔴/🟠 que debe resolverse antes de construir | Sanear ítems críticos y mayores, luego re-auditar |
| 🔴 **REFACTOR TOTAL** | Deuda estructural que requiere rediseño de dominio | Escalar a `@QwikArchitect` para plan de rediseño |

### 🟢 APTO

```
→ Continúa directamente con /feature o /bug-fix
```

### 🟠 CONDICIONADO

```
→ Para cada ítem 🔴 y 🟠 del plan de saneamiento:
   @QwikBuilder [archivo] con scope acotado al ítem   ← deuda de implementación
   /bug-fix [problema]                                 ← bugs reales
→ Re-ejecuta /legacy-audit para verificar saneamiento
→ Solo entonces procede con nueva feature
```

### 🔴 REFACTOR TOTAL

```
→ Para cada ítem 🔴 y 🟠 del plan de saneamiento:
   @QwikBuilder [archivo] con scope acotado al ítem   ← deuda de implementación
   /bug-fix [problema]                                 ← bugs reales
→ Re-ejecuta /legacy-audit para verificar saneamiento
→ Solo entonces procede con nueva feature
```

### 🔴 REFACTOR TOTAL

```
→ Llama a @QwikArchitect:
  "El código en ${input:legacyPath} tiene deuda crítica estructural.
   Lee docs/audits/legacy-${input:legacyPath}-audit.md y diseña
   un plan de rediseño del dominio en
   docs/plans/refactor-${input:legacyPath}.md"
→ @QwikDBA valida schema si hay cambios de datos
→ @QwikBuilder reimplementa limpio sobre el nuevo plan
→ @QwikAuditor certifica el resultado
```

---

> 💡 **Nota:** El objetivo no es reescribir todo por reescribirlo. Es tener
> una foto clara de la deuda real para tomar decisiones informadas. A veces
> el código heredado está mejor de lo que parece — el audit te lo dirá.
