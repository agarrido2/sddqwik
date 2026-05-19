---
# EXTERNAL_AGENT_PATH: ".github/prompts/legacy-audit.prompt.md"
name: legacy-audit
description: >
  Audita código heredado o no confiable antes de incorporarlo al flujo SDD.
  Verifica alcance, riesgos, arquitectura, seguridad, tests, datos y compatibilidad
  Qwik; emite un veredicto operativo y define si el código puede usarse, requiere
  saneamiento acotado o necesita rediseño. No permite construir sobre legacy sin
  auditoría trazable.
tools: ["read", "edit", "execute/runInTerminal", "upstash/context7/*"]
argument-hint: "example: /legacy-audit src/features/auth"
---

# 🏚️ LEGACY AUDIT PROTOCOL — `${input:legacyPath}`

## Propósito

`/legacy-audit` es la puerta de entrada para código heredado, generado fuera del flujo SDD, dudoso, importado o no confiable.

No reescribe código.
No sanea automáticamente.
No convierte legacy en feature nueva.
No sustituye a `/bug-fix`.
No permite construir encima sin veredicto.

Su objetivo es producir una decisión trazable:

```text
Legacy detectado → Alcance acotado → Auditoría → Veredicto → Plan de saneamiento o adopción → Entrada segura al flujo SDD
```

---

## Regla operativa crítica

Antes de permitir que una feature, bugfix o refactor se apoye en código heredado, debe existir:

1. ruta válida;
2. artefacto `docs/audits/legacy-[slug]-audit.md`;
3. scope auditado;
4. riesgos clasificados;
5. veredicto final;
6. acción siguiente clara;
7. si procede, plan de saneamiento o rediseño.

Si no hay veredicto, no se construye encima.

---

## Prohibiciones

Durante `/legacy-audit`, no hacer:

- modificar código heredado;
- aplicar fixes;
- reestructurar carpetas;
- crear nuevas features;
- aprobar el legacy por intuición;
- auditar todo el repo si se pidió una ruta concreta;
- leer artefactos no relacionados;
- ocultar deuda crítica como deuda menor;
- emitir veredicto sin evidencia;
- saltar a Builder sin plan de saneamiento.

---

## Paso 0 — Validar entrada

Usar `${input:legacyPath}` como ruta auditada.

Si no se proporciona, detener:

```text
LEGACY-AUDIT GATE BLOQUEADO: falta una ruta legacyPath.
Usa: /legacy-audit [ruta]
```

Verificar que la ruta existe y es accesible:

```bash
LEGACY_PATH="${input:legacyPath}"

if [ -z "$LEGACY_PATH" ]; then
  echo "LEGACY-AUDIT GATE BLOQUEADO: legacyPath vacío."
  exit 1
fi

if [ ! -e "$LEGACY_PATH" ]; then
  echo "LEGACY-AUDIT GATE BLOQUEADO: la ruta no existe o no es accesible: $LEGACY_PATH"
  exit 1
fi

echo "Ruta legacy válida: $LEGACY_PATH"
```

---

## Paso 1 — Normalizar slug de auditoría

Crear un slug seguro para el nombre del reporte.

```bash
LEGACY_SLUG=$(echo "$LEGACY_PATH" | sed 's#[/.]#-#g' | sed 's/[^a-zA-Z0-9_-]/-/g' | sed 's/--*/-/g' | sed 's/^-//' | sed 's/-$//')
AUDIT_FILE="docs/audits/legacy-${LEGACY_SLUG}-audit.md"
mkdir -p docs/audits

echo "Audit file: $AUDIT_FILE"
```

---

## Paso 2 — Crear o preservar artefacto de auditoría

Si no existe, crear reporte inicial.

```bash
if [ ! -f "$AUDIT_FILE" ]; then
  cat > "$AUDIT_FILE" <<EOF
# Legacy Audit: ${LEGACY_PATH}

> Estado: 🔴 Auditando
> Created: $(date +%F)
> Last Updated: $(date +%F)
> Ruta auditada: ${LEGACY_PATH}
> Agente auditor: @QwikAuditor

## 1. Scope auditado

- Ruta: ${LEGACY_PATH}
- Tipo: [archivo / carpeta / módulo / feature / desconocido]
- Archivos revisados: pendiente
- Artefactos SDD relacionados: pendiente
- Feature relacionada: pendiente

## 2. Contexto y origen

- Origen del código: [legacy manual / generado por IA / importado / desconocido]
- Nació dentro de SDD: [sí / no / desconocido]
- Último cambio conocido: [N/A]
- Riesgo inicial: [bajo / medio / alto / crítico]

## 3. Hallazgos críticos 🔴

Pendiente.

## 4. Hallazgos mayores 🟠

Pendiente.

## 5. Hallazgos menores 🟡

Pendiente.

## 6. Patrones correctos a preservar ✅

Pendiente.

## 7. Riesgos por dominio

| Dominio | Estado | Evidencia | Impacto |
|---|---|---|---|
| Arquitectura | Pendiente |  |  |
| Qwik/Resumability | Pendiente |  |  |
| Seguridad | Pendiente |  |  |
| Datos/RLS | Pendiente |  |  |
| Testing | Pendiente |  |  |
| UX/A11Y | Pendiente |  |  |
| Mantenibilidad | Pendiente |  |  |

## 8. Plan de saneamiento

| Prioridad | Tipo | Archivo/Zona | Problema | Acción requerida | Agente | Bloquea construcción |
|---|---|---|---|---|---|---|

## 9. Veredicto final

> Veredicto: PENDIENTE

Valores permitidos:
- 🟢 APTO
- 🟠 CONDICIONADO
- 🔴 REFACTOR TOTAL
- ⛔ NO INCORPORAR

## 10. Acción siguiente

Pendiente.

## 11. Señal para memoria

- Actualizar INDEX: [sí / no]
- Lessons learned: [N/A]
- ADR candidate: [N/A]
- Legacy adoption note: [N/A]
EOF
  echo "CREADO $AUDIT_FILE"
else
  echo "Audit File existente: $AUDIT_FILE"
  grep -E '^> Estado:|^> Veredicto:' "$AUDIT_FILE" | head -2 || true
fi
```

### Regla

No sobrescribir un reporte existente.
Si existe, re-auditar sobre el mismo artefacto añadiendo nueva evidencia y fecha de actualización.

---

## Paso 3 — Determinar carga permitida

Para auditar legacy, se permite revisar la ruta indicada y standards necesarios.

### Cargar siempre

```text
${input:legacyPath}
docs/standards/ARQUITECTURA-FOLDER.md
docs/standards/PROJECT-RULES-CORE.md
docs/standards/QUALITY-STANDARDS.md
docs/standards/SERIALIZATION-CONTRACTS.md
docs/standards/DECISIONS-QWIK.md
docs/standards/TESTING-POLICY.md
docs/standards/LESSONS-LEARNED.md
```

### Cargar si aplica

```text
docs/standards/DECISIONS-DATA.md
docs/standards/SECURITY-POLICIES.md
docs/standards/RBAC-ROLES-PERMISSIONS.md
docs/standards/DECISIONS-UI.md
docs/standards/UX-GUIDE.md
docs/sessions/INDEX.md
Spec o Plan relacionados solo si INDEX o el código auditado los referencia
```

### Regla

No auditar por lectura masiva del repo.
El scope es `${input:legacyPath}`.

---

## Paso 4 — Invocar a @QwikAuditor

Mensaje de handoff:

```text
@QwikAuditor

Ejecuta Legacy Audit sobre `${input:legacyPath}`.

Artefacto de auditoría:
- docs/audits/legacy-[slug]-audit.md

Scope:
- Ruta auditada: `${input:legacyPath}`
- No auditar fuera de esta ruta salvo dependencia directa imprescindible.

Standards obligatorios:
- docs/standards/ARQUITECTURA-FOLDER.md
- docs/standards/PROJECT-RULES-CORE.md
- docs/standards/QUALITY-STANDARDS.md
- docs/standards/SERIALIZATION-CONTRACTS.md
- docs/standards/DECISIONS-QWIK.md
- docs/standards/TESTING-POLICY.md
- docs/standards/LESSONS-LEARNED.md

Standards condicionales:
- docs/standards/DECISIONS-DATA.md si hay datos, queries, schema o persistencia.
- docs/standards/SECURITY-POLICIES.md si hay auth, permisos, secretos o datos sensibles.
- docs/standards/RBAC-ROLES-PERMISSIONS.md si hay roles/permisos.
- docs/standards/DECISIONS-UI.md y UX-GUIDE.md si hay UI relevante.

Tarea:
1. Revisar solo el scope indicado.
2. Clasificar hallazgos en 🔴 Crítico, 🟠 Mayor, 🟡 Menor.
3. Identificar patrones correctos que deben preservarse.
4. Evaluar arquitectura, Qwik/resumability, seguridad, datos/RLS, testing, UX y mantenibilidad según aplique.
5. Rellenar el plan de saneamiento.
6. Emitir veredicto final.
7. Definir acción siguiente.

Restricciones:
- No escribir código.
- No aplicar fixes.
- No reestructurar.
- No convertir deuda en feature.
- No emitir APTO si hay hallazgos críticos bloqueantes.
```

---

## Paso 5 — Veredictos permitidos

El Auditor debe emitir exactamente uno:

| Veredicto | Significado | Acción inmediata |
|---|---|---|
| 🟢 APTO | Se puede construir encima con riesgo aceptable | Permitir entrada a `/new-feature`, `/bug-fix` u `/optimizer-code` según caso |
| 🟠 CONDICIONADO | Hay deuda que debe sanearse antes de construir nueva funcionalidad | Crear plan de saneamiento acotado y re-auditar |
| 🔴 REFACTOR TOTAL | La estructura no es segura para evolución incremental | Escalar a `@QwikArchitect` para rediseño |
| ⛔ NO INCORPORAR | El código es inseguro, irrecuperable o no compensa adoptarlo | Aislar, sustituir o descartar |

---

## Paso 6 — Criterios de bloqueo

Marcar como mínimo `🟠 CONDICIONADO` si hay:

- lógica de negocio relevante en `src/routes/`;
- servicios sin tests obligatorios;
- duplicación grave de tipos/schemas;
- uso no idiomático de Qwik que afecte mantenibilidad;
- validación server-side incompleta;
- manejo de errores opaco;
- deuda que puede generar regresión al construir encima.

Marcar como mínimo `🔴 REFACTOR TOTAL` si hay:

- mezcla estructural severa de capas;
- cruce server/client peligroso;
- patrones incompatibles con resumability;
- seguridad rota o exposición de datos;
- RLS ausente donde sea obligatoria;
- diseño que impide tests razonables;
- arquitectura imposible de evolucionar sin reescritura.

Marcar `⛔ NO INCORPORAR` si:

- el código introduce riesgo de seguridad inaceptable;
- depende de APIs obsoletas o incompatibles sin plan razonable;
- está tan acoplado que reescribir es más barato que sanear;
- no hay evidencia suficiente para confiar en su comportamiento.

---

## Paso 7 — Acciones tras veredicto

### 🟢 APTO

```text
El código legacy puede entrar al flujo SDD normal.
Siguiente paso permitido: /new-feature, /bug-fix u /optimizer-code según objetivo.
```

Regla:
- documentar deuda menor si existe;
- no exigir refactor preventivo innecesario.

### 🟠 CONDICIONADO

```text
No construir nueva funcionalidad encima todavía.
Sanear primero los hallazgos críticos/mayores del plan.
```

Ruta recomendada:

```text
@QwikBuilder para saneamiento acotado de implementación
@QwikDBA si el saneamiento toca datos/RLS
@QwikArchitect si el saneamiento requiere frontera o estructura
Re-ejecutar /legacy-audit después del saneamiento
```

### 🔴 REFACTOR TOTAL

```text
No sanear a base de parches.
Escalar a @QwikArchitect para plan de rediseño.
```

Handoff:

```text
@QwikArchitect

El código en `${input:legacyPath}` recibió veredicto 🔴 REFACTOR TOTAL.
Lee `docs/audits/legacy-[slug]-audit.md` y diseña un plan de rediseño o reemplazo en `docs/plans/refactor-[slug].md`.
No iniciar Builder hasta que el plan esté aprobado.
Si hay datos/RLS implicados, coordinar con @QwikDBA.
```

### ⛔ NO INCORPORAR

```text
No incorporar este código al flujo SDD.
Aislar, descartar o sustituir con implementación nueva basada en Spec aprobada.
```

Regla:
- si se necesita la funcionalidad, volver a `/spec` y definirla limpiamente;
- no migrar deuda irrecuperable por comodidad.

---

## Paso 8 — Actualización de memoria

Activar `@QwikMemory` si:

- se adopta legacy relevante;
- se decide refactor total;
- se descarta código por riesgo;
- aparece lección reutilizable;
- hay decisión estructural que merece ADR;
- el INDEX debe reflejar una zona legacy contenida o saneada.

Mensaje sugerido:

```text
@QwikMemory

Legacy audit completado para `${input:legacyPath}`.
Revisa `docs/audits/legacy-[slug]-audit.md`.
Actualiza INDEX, snapshot o Lessons Learned solo si aporta continuidad real.
No guardar ruido ni copiar el reporte completo.
```

---

## Salida esperada

```text
LEGACY AUDIT REPORT — ${input:legacyPath}

Ruta: [ruta]
Audit file: docs/audits/legacy-[slug]-audit.md
Scope: [archivo/carpeta/módulo]
Hallazgos críticos: N
Hallazgos mayores: N
Hallazgos menores: N
Veredicto: APTO / CONDICIONADO / REFACTOR TOTAL / NO INCORPORAR
Acción siguiente: [acción concreta]
Siguiente agente: [@QwikOrchestrator / @QwikBuilder / @QwikArchitect / @QwikDBA / @QwikMemory / STOP]
Memory signal: sí / no
```

---

## Regla final

`/legacy-audit` no significa “arregla legacy”.

`/legacy-audit` significa:

```text
Determina objetivamente si este código puede entrar al flujo SDD, necesita saneamiento, requiere rediseño o debe descartarse.
```

Sin veredicto, no hay construcción encima.
