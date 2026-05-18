---
# EXTERNAL_AGENT_PATH: ".github/prompts/bug-fix.prompt.md"
name: bug-fix
description: >
  Diagnóstico y corrección de bugs reportados. Crea trazabilidad en docs/bugs/, invoca @QwikBugFix para diagnóstico y @QwikBuilder para la corrección.
  Prerequisito: descripción clara del bug y pasos para reproducirlo.
tools: ["read", "edit", "execute/runInTerminal", "upstash/context7/*"]
argument-hint: "example: /bug-fix login-redirect-loop"
---

# 🐛 BUG FIX PROTOCOL: `${input:bugId}`

**Objetivo:** Diagnosticar y corregir el bug `${input:bugId}` con trazabilidad completa.

**Pasos a seguir en orden:**

1. Crear artefacto `docs/bugs/${input:bugId}.md`
2. Diagnosticar causa raíz con `@QwikBugFix`
3. Aplicar fix con `@QwikBuilder`
4. Verificar sin regresiones
5. Cerrar y actualizar índice/lecciones

> ⚠️ **Regla de oro:** No corrijas síntomas. Diagnostica la causa raíz primero.

---

## Paso 1: Crear el Artefacto de Bug

```bash
mkdir -p docs/bugs
```

Crea `docs/bugs/${input:bugId}.md`:

```md
# Bug: ${input:bugId}

> Estado: 🔴 Open
> Reportado: [YYYY-MM-DD]
> Feature afectada: [módulo/feature]

## Descripción
[Qué ocurre vs qué debería ocurrir]

## Pasos para reproducir
- [paso 1]
- [paso 2]
- [continuar hasta reproducir el comportamiento incorrecto]

## Contexto técnico
- Entorno: [dev / staging / prod]
- Archivos sospechosos: [si se conocen]
- Error en consola/logs: [si existe]

## Diagnóstico
[A rellenar por @QwikBugFix]

## Causa Raíz
[A rellenar por @QwikBugFix]

## Fix Aplicado
[A rellenar por @QwikBuilder]

## Verificación
- [ ] Bug reproducible antes del fix
- [ ] Bug no reproducible después del fix
- [ ] Tests actualizados si aplica
- [ ] Sin regresiones (`bun test`)
```

---

## Paso 2: Diagnóstico de causa raíz

```
@QwikBugFix diagnostica el bug ${input:bugId}.
Lee `docs/bugs/${input:bugId}.md` para el contexto completo.
Identifica la causa raíz y los archivos afectados.
Rellena las secciones "Diagnóstico" y "Causa Raíz" del bug report.

Si el diagnóstico revela un problema de diseño o arquitectura que no puede
resolverse con un fix de implementación, escala a @QwikArchitect antes de
continuar. Documenta el bloqueo en el bug report.

No propongas el fix hasta que la causa raíz esté confirmada.
```

---

## Paso 3: Fix con @QwikBuilder

Una vez confirmada la causa raíz en `docs/bugs/${input:bugId}.md`:

```
@QwikBuilder corrige el bug ${input:bugId}.
La causa raíz y los archivos afectados están documentados en
`docs/bugs/${input:bugId}.md`. Lee ese artefacto antes de tocar código.
Actualiza la sección "Fix Aplicado" del bug report al terminar.
Ejecuta bun test para verificar sin regresiones.
```

---

## Paso 4: Verificación

```bash
bun test
bunx tsc --noEmit
```

Si los tests pasan y el bug está resuelto:
- Actualiza `docs/bugs/${input:bugId}.md` → `Estado: ✅ Resolved`
- Actualiza `docs/sessions/INDEX.md` si el bug afectaba a una feature indexada
- Si la causa raíz deja una lección reutilizable → activa `@QwikMemory` para
  promoverla a `docs/standards/LESSONS-LEARNED.md`

---

## Salida esperada

- `docs/bugs/${input:bugId}.md` con estado `✅ Resolved` y causa raíz documentada
- `docs/sessions/INDEX.md` actualizado si aplica
- `docs/standards/LESSONS-LEARNED.md` actualizado si aplica