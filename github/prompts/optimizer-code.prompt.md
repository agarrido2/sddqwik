---
# EXTERNAL_AGENT_PATH: ".github/prompts/optimizer-code.prompt.md"
name: optimizer-code
description: >
  Ejecuta refactorización quirúrgica y optimización de código existente con scope
  acotado. Detecta deuda técnica, separa responsabilidades, mejora Qwik
  resumability, portabilidad, tests y calidad sin cambiar comportamiento
  funcional. Si el cambio requerido altera contrato, datos, arquitectura o
  comportamiento, detiene el flujo y redirige a Spec, Architect, DBA o BugFix.
tools: ["read", "edit", "execute/runInTerminal", "upstash/context7/*"]
argument-hint: "example: /optimizer-code src/features/auth/components/LoginForm.tsx"
---

# 🔬 OPTIMIZER CODE PROTOCOL — `${input:filePath}`

## Propósito

`/optimizer-code` es una entrada de refactorización quirúrgica.

No crea features.
No corrige bugs sin diagnóstico.
No cambia comportamiento funcional.
No rediseña arquitectura completa.
No modifica schema ni RLS.
No sustituye a Auditor ni Architect.

Su objetivo es mejorar código existente manteniendo el contrato observable:

```text
Scope acotado → Auditoría local → Plan de refactor → Cambio quirúrgico → Validación → Reporte
```

---

## Regla operativa crítica

Antes de editar, este prompt debe verificar:

1. existe `${input:filePath}`;
2. el scope es acotado;
3. el objetivo es refactor/optimización, no cambio funcional;
4. no se requieren cambios de datos, RLS o contrato público;
5. no se requiere rediseño arquitectónico amplio;
6. existen standards aplicables;
7. se puede validar el resultado con tests, typecheck o build cuando aplique.

Si alguna condición falla, detener y enrutar al flujo correcto.

---

## Prohibiciones

Durante `/optimizer-code`, no hacer:

- añadir funcionalidad nueva;
- cambiar comportamiento visible sin Spec;
- modificar schema, migraciones o RLS;
- cambiar APIs públicas sin Plan;
- tocar archivos fuera de scope sin justificarlo;
- convertir un bug en refactor;
- reescribir un módulo completo por preferencia estética;
- crear abstracciones prematuras;
- saltar tests relevantes;
- ignorar Qwik resumability;
- ocultar deuda estructural bajo cambios cosméticos.

---

## Paso 0 — Validar entrada

Usar `${input:filePath}` como ruta canónica del archivo o carpeta a optimizar.

```bash
TARGET_PATH="${input:filePath}"

if [ -z "$TARGET_PATH" ]; then
  echo "OPTIMIZER GATE BLOQUEADO: falta filePath."
  echo "Usa: /optimizer-code [ruta]"
  exit 1
fi

if [ ! -e "$TARGET_PATH" ]; then
  echo "OPTIMIZER GATE BLOQUEADO: la ruta no existe: $TARGET_PATH"
  exit 1
fi

echo "Scope objetivo: $TARGET_PATH"
```

---

## Paso 1 — Clasificar tipo de trabajo

Antes de editar, clasificar la solicitud:

| Señal | Clasificación | Acción |
|---|---|---|
| archivo grande, lógica mezclada, nombres pobres, duplicación | refactor-local | continuar |
| mejora de legibilidad sin cambio funcional | cleanup-local | continuar |
| extraer constantes, hook, servicio o subcomponente | decomposition-local | continuar |
| bug observado o regresión | bug | detener → `/bug-fix` |
| cambio de comportamiento funcional | feature-change | detener → `/spec` o `/new-feature` |
| cambio de schema, RLS, migraciones | data-change | detener → `@QwikDBA` |
| cambio de arquitectura o fronteras de dominio | architecture-change | detener → `@QwikArchitect` |
| código legacy no auditado | legacy-risk | detener → `/legacy-audit` |

### Regla

Solo continuar si la clasificación es:

```text
refactor-local
cleanup-local
decomposition-local
```

---

## Paso 2 — Carga mínima permitida

Cargar el scope objetivo y standards relevantes.

### Cargar siempre

```text
${input:filePath}
docs/standards/ARQUITECTURA-FOLDER.md
docs/standards/PROJECT-RULES-CORE.md
docs/standards/DECISIONS-QWIK.md
docs/standards/SERIALIZATION-CONTRACTS.md
docs/standards/QUALITY-STANDARDS.md
docs/standards/TESTING-POLICY.md
docs/standards/LESSONS-LEARNED.md
```

### Cargar si aplica

```text
docs/standards/DECISIONS-UI.md
docs/standards/UX-GUIDE.md
docs/standards/DECISIONS-DATA.md
docs/standards/SECURITY-POLICIES.md
docs/standards/RBAC-ROLES-PERMISSIONS.md
docs/sessions/INDEX.md
Spec/Plan relacionados solo si el archivo pertenece claramente a una feature activa
```

### Regla

No cargar specs o plans de otras features.
No abrir todo `src/`.
No convertir el refactor en auditoría global.

---

## Paso 3 — Auditoría local antes de editar

Emitir un diagnóstico breve antes de generar cambios.

Verificar:

1. Separación de responsabilidades.
2. Lógica de negocio dentro de UI o routes.
3. Dependencias directas no permitidas.
4. Estado Qwik sobredimensionado.
5. Fronteras `$()` y serialización.
6. Imports pesados o barrel exports peligrosos.
7. Funciones largas o anidamiento excesivo.
8. Naming genérico.
9. Duplicación local.
10. Tests existentes o necesarios.

Formato obligatorio:

```text
OPTIMIZER LOCAL AUDIT

Scope: [ruta]
Clasificación: refactor-local / cleanup-local / decomposition-local
Riesgo: bajo / medio / alto
Cambio funcional esperado: NO
Archivos dentro de scope: [lista]
Archivos fuera de scope requeridos: [lista o N/A]
Bloqueos: [N/A o motivo de stop]
```

Si el riesgo es alto por arquitectura o dominio, detener y escalar a `@QwikArchitect`.

---

## Paso 4 — Plan de refactor quirúrgico

Antes de editar, definir plan.

```text
OPTIMIZER PLAN

Objetivo:
- [qué se mejora]

No cambiar:
- comportamiento observable
- contrato público
- rutas
- schema/RLS
- permisos
- copy funcional salvo limpieza menor

Acciones:
1. [acción concreta]
2. [acción concreta]
3. [acción concreta]

Validación:
- [test/typecheck/build/comprobación manual]
```

### Regla

El plan debe ser pequeño.
Si requiere más de 5-7 acciones relevantes o toca muchos dominios, no es optimizer-code: escalar a Architect o Spec.

---

## Paso 5 — Refactor permitido

Aplicar solo cambios dentro del scope o directamente derivados del scope.

Cambios permitidos:

- extraer constantes;
- extraer helpers puros;
- extraer hook local si no cambia contrato;
- extraer subcomponentes presentacionales;
- mover lógica de UI a servicio/hook cuando el standard lo exige;
- reducir estado serializado;
- corregir closures `$()` inseguras;
- mejorar nombres;
- eliminar duplicación local;
- añadir tests para servicios/utilidades afectadas;
- mejorar manejo de errores sin cambiar flujo funcional.

Cambios no permitidos sin redirección:

- nuevas pantallas;
- nuevos endpoints;
- nuevos campos de DB;
- nuevas policies;
- nuevo comportamiento de negocio;
- cambio de roles/permisos;
- rediseño de dominio;
- reescritura completa por preferencia;
- cambios de UX significativos.

---

## Paso 6 — Validación obligatoria

Ejecutar lo que aplique y exista en el repo.

```bash
bun test
bunx tsc --noEmit
bun run build
```

Si el repo tiene scripts específicos, usarlos según `package.json`.

Si un comando no existe o no aplica, documentarlo.
No inventar resultados.

### Tests

Si se crean o modifican servicios, helpers críticos o lógica reutilizable, aplicar `docs/standards/TESTING-POLICY.md`.

Regla:

```text
Servicio nuevo o modificado → test obligatorio.
Helper crítico nuevo o modificado → test recomendado/obligatorio según impacto.
Componente puramente visual → test no obligatorio salvo lógica relevante.
```

---

## Paso 7 — Reporte de cierre

Al terminar, emitir:

```text
OPTIMIZER REPORT

Scope: [ruta]
Clasificación: refactor-local / cleanup-local / decomposition-local
Archivos modificados:
- [ruta]

Cambios realizados:
- [cambio]

Comportamiento funcional:
- Sin cambios / cambios detectados y bloqueados

Tests/validación:
- [comando] → [resultado]

Riesgos residuales:
- [N/A o riesgo]

Siguiente paso recomendado:
- [N/A / @QwikAuditor / /legacy-audit / @QwikArchitect / /bug-fix]
```

---

## Paso 8 — Handoff a Auditor si aplica

Activar `@QwikAuditor` si:

- el refactor tocó lógica sensible;
- se movió lógica entre capas;
- se extrajeron servicios;
- se tocaron fronteras `$()`;
- había deuda crítica o mayor;
- la validación dejó dudas;
- el cambio afecta una feature en curso.

Mensaje:

```text
@QwikAuditor

Revisa el refactor realizado por /optimizer-code sobre `${input:filePath}`.

Contexto:
- Scope original: `${input:filePath}`
- Reporte Optimizer: [resumen]
- Archivos modificados: [lista]
- Validación ejecutada: [tests/typecheck/build]

Tarea:
1. Verificar que no cambió comportamiento funcional.
2. Verificar que se respetan ARQUITECTURA-FOLDER, DECISIONS-QWIK, SERIALIZATION-CONTRACTS y QUALITY-STANDARDS.
3. Verificar tests según TESTING-POLICY.
4. Emitir PASSED/FAILED con evidencia si el riesgo lo justifica.
```

---

## Señales para Memory

Activar `@QwikMemory` solo si:

- se descubrió patrón reusable;
- se corrigió deuda recurrente;
- se tomó decisión estructural menor pero durable;
- el refactor afecta una feature WIP o una zona legacy contenida;
- debe actualizarse `LESSONS-LEARNED.md`.

No guardar ruido de refactors triviales.

---

## Salida esperada

```text
OPTIMIZER RESULT — ${input:filePath}

Scope validado: sí / no
Clasificación: refactor-local / cleanup-local / decomposition-local / blocked
Cambio funcional: no / sí-bloqueado
Archivos modificados: N
Tests/validación: passed / failed / not-run-with-reason
Auditor requerido: sí / no
Memory signal: sí / no
Estado: COMPLETED / BLOCKED / NEEDS-AUDIT

Siguiente paso:
[una acción clara]
```

---

## Regla final

`/optimizer-code` no significa “mejora esto como quieras”.

`/optimizer-code` significa:

```text
Haz una mejora interna acotada, sin cambiar el contrato funcional y con validación suficiente.
```

Si el cambio necesita alterar comportamiento, datos o arquitectura, no es optimizer-code.
