---
# EXTERNAL_AGENT_PATH: "github/prompts/spec.prompt.md"
name: spec
description: >
  Entrada principal del flujo spec-first. Crea o revisa una Spec formal antes de
  construir una feature desde input del usuario, INDEX/snapshot, bug o legacy si aplica.
  Obliga a discovery mínimo, Acceptance Criteria binarios, Scope OUT, riesgos,
  decisiones abiertas y aprobación explícita.
  Sin Spec Approved, /new-feature no puede ejecutarse.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /spec mieleshuelva-configuration"
---

# 📐 SPEC KICKOFF — `${input:featureName}`

## Propósito

`/spec` es la entrada principal para definir trabajo en el sistema spec-first. Crea el primer artefacto contractual verificable que gobierna todo el ciclo SDD.

No implementa código.
No crea Plan técnico.
No decide arquitectura final.
No aprueba por cuenta propia.
No diseña implementación prematura.

Su objetivo es convertir una intención funcional en una Spec auditable:

```text
Input usuario / INDEX / snapshot / bug / legacy → Discovery mínimo → Scope → Acceptance Criteria → Riesgos → Review → Approved
```

PRD y Blueprint no forman parte del flujo operativo principal. La Spec debe poder crearse sin depender de ellos.

---

## Regla operativa crítica

Una Spec debe quedar inicialmente en `🟡 Review`. Solo habilita `/new-feature` si contiene:

1. estado explícito `🟢 Approved` o `Approved` tras aprobación del usuario;
2. propósito funcional claro;
3. usuarios/roles afectados;
4. Scope IN;
5. Scope OUT;
6. Acceptance Criteria funcionales binarios;
7. Acceptance Criteria no funcionales;
8. datos funcionales relevantes si aplica, sin diseñar schema;
9. dependencias y relaciones;
10. riesgos y restricciones;
11. decisiones abiertas, marcando cuáles bloquean aprobación;
12. notas de datos/RLS si aplica, sin resolverlas técnicamente;
13. criterios de auditoría verificables.

Si cualquiera de estas piezas falta, la Spec debe quedar en `🟡 Review`, no `Approved`.

---

## Prohibiciones

Durante `/spec`, no hacer:

- escribir código en `src/`;
- crear migraciones;
- diseñar schema definitivo sin DBA;
- definir rutas, componentes, servicios o capas de implementación;
- aprobar la Spec sin confirmación del usuario;
- leer todas las specs;
- leer todos los plans;
- explorar el repo masivamente;
- convertir dudas en decisiones cerradas;
- ocultar ambigüedades;
- mezclar varias features grandes en una sola Spec.

---

## Paso 0 — Validar entrada

Usar `${input:featureName}` como identificador canónico.

```bash
FEATURE="${input:featureName}"

if [ -z "$FEATURE" ]; then
  echo "SPEC GATE BLOQUEADO: falta featureName."
  echo "Usa: /spec [feature-name]"
  exit 1
fi

echo "Spec objetivo: $FEATURE"
```

---

## Paso 1 — Preparar workspace mínimo

```bash
mkdir -p docs/specs
mkdir -p docs/sessions
```

Si falta `docs/sessions/INDEX.md`, no inventar contexto histórico. Recomendar `/setup` o crear estructura mínima solo si el usuario está inicializando.

```bash
if [ ! -f docs/sessions/INDEX.md ]; then
  echo "ADVERTENCIA: falta docs/sessions/INDEX.md"
  echo "Recomendación: ejecutar /setup antes de crear Specs en un proyecto real."
fi
```

---

## Paso 2 — Discovery mínimo sin exploración masiva

Leer solo fuentes estructurales acotadas.

Permitido:

```text
docs/sessions/INDEX.md si existe
snapshot de sesión si INDEX lo referencia
bug report si el flujo viene de /bug-fix
legacy audit si el flujo viene de /legacy-audit
Spec existente de la misma feature si ya existe
input directo del usuario
```

Comprobación inicial:

```bash
echo "Entradas relacionadas en INDEX:"
grep -Ei "(^\|[[:space:]]*${FEATURE}[[:space:]]*\|)|${FEATURE}" docs/sessions/INDEX.md 2>/dev/null || echo "No hay entradas relacionadas en INDEX"
```

### Regla

No usar `ls docs/specs/` para inspeccionar todas las specs.
Solo leer specs relacionadas si INDEX, snapshot, bug, legacy o el usuario las referencia explícitamente.

---

## Paso 3 — Detectar duplicados o solapamiento

Antes de crear o modificar la Spec, revisar si ya existe:

```bash
SPEC_FILE="docs/specs/${FEATURE}.md"

if [ -f "$SPEC_FILE" ]; then
  echo "Spec existente: $SPEC_FILE"
  grep -E '^>?[[:space:]]*Estado:|^>?[[:space:]]*Status:' "$SPEC_FILE" | head -1 || true
fi
```

Si la Spec ya existe y está `Approved`, no sobrescribirla.

Acciones permitidas:

```text
- si el usuario quiere cambiar comportamiento: crear revisión y volver a Review;
- si solo falta aclaración menor: actualizar manteniendo trazabilidad;
- si la feature ya está en build: detener y exigir revisión de Plan antes de cambiar contrato;
- si el cambio es bug: redirigir a /bug-fix.
```

---

## Paso 4 — Invocar a @QwikSpeccer

Mensaje de handoff:

```text
@QwikSpeccer

Crea o revisa la Spec formal para `${input:featureName}`.

Contexto mínimo:
- Feature: `${input:featureName}`
- Spec esperada: docs/specs/${input:featureName}.md
- INDEX: docs/sessions/INDEX.md si existe
- Snapshot: solo si INDEX lo referencia
- Bug report o legacy audit: solo si el flujo lo indica
- Input directo del usuario: usarlo como fuente principal cuando exista

Tarea:
1. Determinar si es nueva Spec o revisión de Spec existente.
2. Leer solo contexto relacionado.
3. Identificar objetivo funcional, usuarios, roles y límites.
4. Definir Scope IN y Scope OUT.
5. Escribir Acceptance Criteria funcionales binarios.
6. Escribir Acceptance Criteria no funcionales verificables.
7. Identificar datos funcionales relevantes si aplica, sin diseñar schema.
8. Señalar datos/RLS/permisos si aplica, sin resolverlos técnicamente.
9. Identificar dependencias, riesgos, supuestos y decisiones abiertas.
10. Marcar explícitamente cualquier decisión abierta bloqueante.
11. Dejar la Spec en `🟡 Review` hasta aprobación explícita del usuario.
12. Solo tras aprobación explícita, cambiar a `🟢 Approved`.

Restricciones:
- No escribir código.
- No crear Plan técnico.
- No definir schema, rutas, migraciones, componentes ni servicios.
- No aprobar sin el usuario.
- No mezclar múltiples features independientes.
- No resolver dudas inventando alcance.
```

---

## Paso 5 — Estructura obligatoria de la Spec

Crear `docs/specs/${input:featureName}.md` con esta estructura mínima:

```md
# Spec: ${input:featureName}

> Estado: 🟡 Review
> Fecha: [YYYY-MM-DD]
> Owner funcional: [usuario/rol]
> Contexto de entrada: [usuario / INDEX / snapshot / bug / legacy / otro]
> Artefacto: docs/specs/${input:featureName}.md

## 1. Propósito

[Qué problema resuelve y para quién]

## 2. Contexto funcional

- Usuarios afectados:
- Roles/permisos afectados:
- Flujo actual:
- Flujo deseado:

## 3. Scope IN

- [lo que sí se construye]

## 4. Scope OUT

- [lo que explícitamente NO se construye]

## 5. Acceptance Criteria funcionales

| AC | Criterio | Verificación |
|---|---|---|
| AC-001 | Dado/Cuando/Entonces o regla binaria verificable | Cómo se comprueba |
| AC-002 |  |  |
| AC-003 |  |  |

## 6. Acceptance Criteria no funcionales

| AC-NF | Tipo | Criterio | Verificación |
|---|---|---|---|
| AC-NF-001 | Seguridad |  |  |
| AC-NF-002 | Performance |  |  |
| AC-NF-003 | A11Y/UX |  |  |

## 7. Datos funcionales y serialización

- Datos de entrada relevantes: [N/A o descripción funcional]
- Datos de salida relevantes: [N/A o descripción funcional]
- Datos que cruzan frontera serializable: [N/A o lista funcional]
- Datos que NO deben cruzar frontera: [N/A o lista funcional]
- Pendiente DBA: [sí/no y motivo]

No definir schema, migraciones ni modelos técnicos en la Spec.

## 8. Datos, permisos y RLS

- Necesidad de datos: [N/A / lectura / escritura / pendiente DBA]
- Permisos funcionales: [N/A o roles afectados]
- RLS: [N/A / pendiente DBA]
- Restricciones de privacidad/seguridad: [N/A o descripción]

## 9. Dependencias y relaciones

- Depende de:
- Reutiliza:
- Expone:
- Impacta a:

## 10. Estados, errores y casos límite

- Loading:
- Empty:
- Error:
- Unauthorized:
- Edge cases:

## 11. Riesgos y supuestos

- Riesgos:
- Supuestos:
- Decisiones abiertas no bloqueantes:
- Decisiones abiertas bloqueantes para aprobación:

## 12. Alcance funcional impactado

- Pantallas o flujos de usuario afectados:
- Roles o permisos afectados:
- Estados visibles afectados:
- Evidencia esperada para auditoría:

## 13. Criterios de auditoría

El Auditor deberá verificar:

- [AC funcionales]
- [AC no funcionales]
- [Scope OUT respetado]
- [datos funcionales serializables si aplica]
- [tests si aplica]

## 14. Historial de aprobación

- [YYYY-MM-DD] — Estado inicial: Review
- [YYYY-MM-DD] — Aprobación usuario: pendiente
```

---

## Paso 6 — Calidad de Acceptance Criteria

Los AC deben ser binarios.

Buenos ejemplos:

```text
AC-001: Si un usuario sin permiso de administración intenta abrir una función restringida, el sistema debe impedir el acceso y mostrar una respuesta autorizada por producto.
Verificación: prueba funcional o revisión contra criterio de acceso esperado.
```

Malos ejemplos:

```text
AC-001: La pantalla debe ser intuitiva.
AC-002: El sistema debe funcionar bien.
AC-003: Gestionar usuarios correctamente.
```

### Regla

Si un AC no puede pasar/fallar de forma objetiva, no es válido.

---

## Paso 7 — Control de tamaño de Spec

Antes de proponer aprobación, evaluar tamaño.

Si la Spec:

- tiene más de 10-12 AC funcionales;
- toca más de 3 dominios fuertes;
- requiere cambios grandes de datos, UI, auth y billing a la vez;
- mezcla varias pantallas independientes;
- no cabe en un ciclo Builder + Auditor razonable;

entonces dividirla.

Salida:

```text
SPEC GATE: Esta Spec es demasiado grande para una sola feature.
Propuesta: dividir en [feature-a], [feature-b], [feature-c].
```

---

## Paso 8 — Aprobación explícita

La Spec debe quedar inicialmente en:

```text
> Estado: 🟡 Review
```

Solo cambiar a:

```text
> Estado: 🟢 Approved
```

cuando el usuario apruebe explícitamente.

Frases válidas de aprobación:

```text
aprobada
approved
sí, adelante
la apruebo
ok, apruébala
```

Si el usuario pide cambios, mantener `Review`.

---

## Paso 9 — Registrar en INDEX

Actualizar `docs/sessions/INDEX.md` solo si existe.

Regla:

- si no existe fila de la feature, añadirla;
- si existe, actualizar estado y artefacto;
- no duplicar filas;
- no marcar como `Approved` si la Spec sigue en Review.

Formato recomendado:

```md
| ${input:featureName} | [módulo] | 🟡 Spec Review / 🟢 Spec Approved | [dependencias] | [datos/N/A] | [expone/N/A] | Spec | [resumen breve] | [YYYY-MM-DD] |
```

---

## Paso 10 — Salida esperada

```text
SPEC REPORT — ${input:featureName}

Spec file: docs/specs/${input:featureName}.md
Estado inicial: 🟡 Review
Scope IN: [resumen]
Scope OUT: [resumen]
AC funcionales: N
AC no funcionales: N
Datos/RLS: N/A / pendiente DBA / aplica
Riesgos abiertos: N
Decisiones abiertas bloqueantes: N
Tamaño: OK / DIVIDIR
INDEX actualizado: sí / no

Siguiente paso:
- revisar y aprobar Spec
- ajustar Spec
- dividir Spec
- /new-feature ${input:featureName} cuando esté Approved
```

---

## Criterios de calidad antes de aprobar

El usuario debe poder responder sí a todo:

- ¿Entiendo exactamente qué se va a construir?
- ¿Entiendo exactamente qué NO se va a construir?
- ¿Cada AC puede pasar o fallar objetivamente?
- ¿Los datos que cruzan fronteras son serializables?
- ¿Los roles/permisos están claros si aplica?
- ¿Los errores, estados vacíos y casos límite están contemplados?
- ¿La feature tiene tamaño razonable?

---

## Regla final

`/spec` no significa “describe una idea”.

`/spec` significa:

```text
Construye un contrato verificable que Builder pueda implementar y Auditor pueda comprobar.
```

Si Auditor no puede verificarla, la Spec todavía no está lista.
