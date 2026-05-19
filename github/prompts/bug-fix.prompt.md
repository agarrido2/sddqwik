---
# EXTERNAL_AGENT_PATH: ".github/prompts/bug-fix.prompt.md"
name: bug-fix
description: >
  Gestiona el ciclo completo de un bug reportado sin saltar directamente a
  implementación. Crea o actualiza el artefacto en docs/bugs/, exige reproducción
  o evidencia suficiente, fuerza diagnóstico de causa raíz con @QwikBugFix,
  clasifica el tipo de problema, enruta a Builder, Architect o DBA según proceda,
  verifica el fix y cierra con memoria si deja aprendizaje reutilizable.
tools: ["read", "edit", "execute/runInTerminal", "upstash/context7/*"]
argument-hint: "example: /bug-fix login-redirect-loop"
---

# 🐛 BUG FIX PROTOCOL — `${input:bugId}`

## Propósito

`/bug-fix` es la puerta de entrada para incidencias, regresiones, comportamientos incorrectos y hotfixes.

No es un atajo para modificar código.
No sustituye a `/new-feature`.
No permite corregir sin diagnóstico.
No permite parches indefinidos.

Su objetivo es convertir un problema observado en un flujo trazable:

```text
Bug report → Reproducción/Evidencia → Diagnóstico → Clasificación → Fix acotado → Verificación → Cierre → Memoria si aplica
```

---

## Regla operativa crítica

Antes de invocar a `@QwikBuilder`, este prompt debe asegurar que existe:

1. artefacto `docs/bugs/${input:bugId}.md`;
2. descripción del comportamiento esperado y observado;
3. pasos de reproducción o evidencia suficiente;
4. diagnóstico documentado;
5. causa raíz documentada o hipótesis explícita;
6. clasificación del bug;
7. routing correcto según clasificación;
8. criterio de verificación.

Si no existe causa raíz suficiente, no se implementa fix.

---

## Prohibiciones

Durante `/bug-fix`, no hacer:

- escribir código antes de diagnóstico;
- corregir síntomas sin causa raíz;
- convertir un bug en feature nueva sin Spec;
- tocar archivos fuera de scope;
- reestructurar arquitectura como parte de un fix local;
- modificar schema/RLS sin intervención de @QwikDBA;
- ignorar tests relevantes;
- cerrar el bug sin verificación;
- repetir ciclos indefinidamente.

---

## Paso 0 — Normalizar entrada

Usar `${input:bugId}` como identificador canónico del bug.

Si el identificador está vacío, es ambiguo o contiene espacios no intencionales, detener:

```text
BUG-FIX GATE BLOQUEADO: falta un bug-id válido.
Usa: /bug-fix [bug-id]
```

---

## Paso 1 — Crear o preservar artefacto de bug

Ruta canónica:

```text
docs/bugs/${input:bugId}.md
```

Crear carpeta si falta:

```bash
BUG_ID="${input:bugId}"
BUG_FILE="docs/bugs/${BUG_ID}.md"
mkdir -p docs/bugs
```

### Si el bug no existe

Crear artefacto inicial:

```bash
if [ ! -f "$BUG_FILE" ]; then
  cat > "$BUG_FILE" <<EOF
# Bug: ${BUG_ID}

> Estado: 🔴 Open
> Reportado: $(date +%F)
> Feature afectada: [pendiente]
> Severidad: [Crítico / Mayor / Menor]
> Entorno: [dev / staging / prod]
> Owner del flujo: @QwikBugFix

## 1. Resumen

[Descripción breve del problema]

## 2. Comportamiento observado

[Qué ocurre realmente]

## 3. Comportamiento esperado

[Qué debería ocurrir]

## 4. Pasos para reproducir

- [paso 1]
- [paso 2]
- [resultado observado]

## 5. Evidencia disponible

- Logs:
- Capturas:
- Mensajes de error:
- URL/ruta afectada:
- Usuario/rol afectado:

## 6. Contexto técnico inicial

- Feature relacionada:
- Spec relacionada:
- Plan relacionado:
- Archivos sospechosos:
- Último cambio conocido:

## 7. Reproducción

> Estado reproducción: PENDIENTE

- [ ] El bug se reproduce antes del fix
- [ ] El caso de reproducción está documentado
- [ ] Hay evidencia suficiente si no es reproducible localmente

## 8. Diagnóstico @QwikBugFix

Pendiente.

## 9. Causa raíz

Pendiente.

## 10. Clasificación

Pendiente.

Valores permitidos:
- fix-local
- bug-diseño
- bug-datos-rls
- bug-spec
- bug-dependencia-externa
- no-reproducible

## 11. Routing Decision

Pendiente.

## 12. Fix Plan

Pendiente.

## 13. Fix Aplicado

Pendiente.

## 14. Verificación

- [ ] Bug reproducible antes del fix o evidencia suficiente documentada
- [ ] Bug no reproducible después del fix
- [ ] Tests relevantes ejecutados
- [ ] Typecheck/build ejecutado si aplica
- [ ] Sin regresiones detectadas

## 15. Cierre

> Estado cierre: PENDIENTE

- Resultado:
- Fecha cierre:
- Señal para @QwikMemory:
- Lección reusable:
EOF
  echo "CREADO $BUG_FILE"
else
  echo "Bug File existente: $BUG_FILE"
  grep -E '^> Estado:|^> Status:' "$BUG_FILE" | head -1 || true
fi
```

### Si el bug ya existe

No sobrescribirlo.

Si está `✅ Resolved`, `Closed` o equivalente, detener salvo instrucción explícita:

```text
BUG-FIX GATE BLOQUEADO: el bug ya aparece cerrado.
Si hay regresión, abre un nuevo bug-id o reabre explícitamente este bug.
```

---

## Paso 2 — Verificar contenido mínimo del bug report

Antes de diagnosticar, comprobar que existe información suficiente.

```bash
missing_bug_sections=0

for section in "Comportamiento observado" "Comportamiento esperado" "Pasos para reproducir" "Evidencia disponible"; do
  if grep -q "$section" "$BUG_FILE"; then
    echo "OK sección bug: $section"
  else
    echo "FALTA sección bug: $section"
    missing_bug_sections=$((missing_bug_sections + 1))
  fi
done

if [ "$missing_bug_sections" -gt 0 ]; then
  echo "BUG-FIX GATE BLOQUEADO: el bug report no tiene estructura mínima suficiente."
  echo "Completa comportamiento observado, esperado, reproducción o evidencia antes de diagnosticar."
  exit 1
fi
```

### Regla

Si faltan pasos de reproducción, se permite continuar solo si hay evidencia suficiente:

- stack trace;
- log reproducible;
- captura clara;
- test fallido;
- reporte de producción con contexto.

Si no hay reproducción ni evidencia, clasificar como `no-reproducible` y no tocar código.

---

## Paso 3 — Consultar INDEX sin exploración masiva

Usar `docs/sessions/INDEX.md` si existe para localizar feature relacionada, dependencias y contexto mínimo.

```bash
INDEX_FILE="docs/sessions/INDEX.md"

if [ -f "$INDEX_FILE" ]; then
  echo "INDEX operativo: $INDEX_FILE"
  echo "Entradas potencialmente relacionadas:"
  grep -Ei "${BUG_ID}|$(grep -E '^> Feature afectada:' "$BUG_FILE" | sed 's/^> Feature afectada:[[:space:]]*//' | head -1)" "$INDEX_FILE" || true
else
  echo "ADVERTENCIA: falta docs/sessions/INDEX.md"
  echo "Siguiente paso recomendado si el bug afecta a feature indexada: ejecutar /setup"
fi
```

### Regla

No listar todas las specs ni todos los plans.
El INDEX filtra. Los artefactos concretos se leen solo si están relacionados con el bug.

---

## Paso 4 — Diagnóstico con @QwikBugFix

Invocar a `@QwikBugFix` antes de cualquier implementación.

Mensaje de handoff:

```text
@QwikBugFix

Diagnostica el bug `${input:bugId}`.

Contexto mínimo:
- Bug report: docs/bugs/${input:bugId}.md
- INDEX: docs/sessions/INDEX.md si existe

Tarea:
1. Leer el bug report.
2. Confirmar si la reproducción o evidencia es suficiente.
3. Identificar feature, ruta, servicio, componente, schema o policy afectada.
4. Documentar diagnóstico en sección "8. Diagnóstico @QwikBugFix".
5. Documentar causa raíz en sección "9. Causa raíz".
6. Clasificar el bug en sección "10. Clasificación".
7. Registrar Routing Decision en sección "11. Routing Decision".
8. Definir criterio de verificación en sección "14. Verificación".

Restricciones:
- No escribir código.
- No aplicar fix.
- No derivar a Builder sin causa raíz suficiente.
- No abrir scope funcional nuevo sin Spec.
```

---

## Paso 5 — Clasificación y routing obligatorio

Después del diagnóstico, aplicar esta tabla:

| Clasificación | Ruta correcta | Regla |
|---|---|---|
| `fix-local` | `@QwikBuilder` | Bug acotado de implementación con causa raíz clara |
| `bug-diseño` | `@QwikArchitect` | Problema de plan, arquitectura, contrato o frontera mal diseñada |
| `bug-datos-rls` | `@QwikDBA` | Problema de schema, migración, constraints, RLS o acceso a datos |
| `bug-spec` | `@QwikSpeccer` | La Spec era ambigua, incorrecta o incompleta |
| `bug-dependencia-externa` | `@QwikArchitect` o Context7 | Requiere verificar API externa, breaking change o integración |
| `no-reproducible` | cerrar como no reproducible o pedir más evidencia | No tocar código |

### Regla

No todo bug pertenece a Builder.
Builder solo actúa cuando la clasificación permite `fix-local` o cuando Architect/DBA devuelve un plan de corrección acotado.

---

## Paso 6 — Fix acotado con @QwikBuilder

Solo si la clasificación lo permite.

Mensaje de handoff:

```text
@QwikBuilder

Corrige el bug `${input:bugId}` con scope cerrado.

Contexto obligatorio:
- Bug report: docs/bugs/${input:bugId}.md
- Diagnóstico documentado
- Causa raíz documentada
- Routing Decision documentada
- Fix Plan documentado

Tarea:
1. Leer el bug report completo.
2. Corregir exactamente la causa raíz documentada.
3. No ampliar scope.
4. No convertir el bug en feature nueva.
5. Añadir o actualizar tests si el bug afecta servicio, guard, utilidad, datos o lógica crítica.
6. Actualizar sección "13. Fix Aplicado" con archivos tocados, cambios y pruebas ejecutadas.
7. Devolver a verificación.

Restricciones:
- No tocar archivos fuera del scope salvo justificación explícita en el bug report.
- No modificar schema/RLS sin @QwikDBA.
- No reestructurar arquitectura sin @QwikArchitect.
```

---

## Paso 7 — Verificación obligatoria

Después del fix, verificar.

Comandos recomendados según disponibilidad del repo:

```bash
bun test
bunx tsc --noEmit
bun run build
```

Si algún comando no existe o no aplica, documentarlo en el bug report.
No inventar resultados.

### Criterios mínimos de cierre

- el bug era reproducible antes del fix o tenía evidencia suficiente;
- ya no se reproduce después del fix;
- los tests relevantes pasan;
- no hay regresión evidente;
- el bug report contiene causa raíz y fix aplicado;
- el estado se actualiza de forma explícita.

Si falla la verificación, no cerrar.

---

## Paso 8 — Cierre y memoria

Si la verificación pasa, actualizar:

```text
> Estado: ✅ Resolved
```

En `docs/bugs/${input:bugId}.md`, completar:

- Resultado;
- Fecha cierre;
- pruebas ejecutadas;
- archivos modificados;
- si actualiza o no `docs/sessions/INDEX.md`;
- si deja o no señal para `@QwikMemory`.

Activar `@QwikMemory` si:

- la causa raíz es reutilizable;
- el bug revela un patrón repetido;
- hay una decisión que merece ADR;
- se debe actualizar `LESSONS-LEARNED.md`;
- el bug afecta una feature indexada.

---

## Protocolo anti-loop

Si un fix falla verificación:

- ciclo 1: devolver a Builder con evidencia concreta;
- ciclo 2: devolver a Builder solo si la causa sigue siendo la misma y el fix es acotado;
- ciclo 3: escalar a Architect, DBA o Speccer según evidencia.

No perpetuar ciclos `BugFix → Builder → BugFix`.

---

## Salida esperada

```text
BUG-FIX REPORT — ${input:bugId}

Bug file: CREATED / EXISTS
Reproducción: CONFIRMADA / EVIDENCIA SUFICIENTE / NO REPRODUCIBLE
Causa raíz: CONFIRMADA / HIPÓTESIS / PENDIENTE
Clasificación: fix-local / bug-diseño / bug-datos-rls / bug-spec / bug-dependencia-externa / no-reproducible
Routing: Builder / Architect / DBA / Speccer / Memory / STOP
Fix: APLICADO / NO APLICA / BLOQUEADO
Verificación: PASSED / FAILED / PENDIENTE
Estado: RESOLVED / OPEN / BLOCKED / NO-REPRODUCIBLE

Siguiente paso:
[una acción clara]
```

---

## Regla final

`/bug-fix` no significa “parchea hasta que funcione”.

`/bug-fix` significa:

```text
Convierte una incidencia en un diagnóstico trazable, clasifica la causa y aplica solo el fix correcto.
```

Si no hay causa raíz suficiente, no hay fix.
