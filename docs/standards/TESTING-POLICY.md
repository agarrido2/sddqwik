# TESTING POLICY — SDD Qwik

> **Propósito:** Define qué código necesita test, cuándo se escribe y cómo se verifica.
> Las reglas de este fichero son ejecutables — están integradas en el flujo de agentes.
>
> **Owner:** @QwikBuilder (escribe) · @QwikAuditor (verifica) · @QwikDBA (no aplica)

---

## 🔴 Regla Absoluta

**Todo servicio nuevo en `src/lib/` o `src/features/*/services/` requiere tests.**
El Builder los escribe. El Auditor los verifica. Sin excepción.

---

## 📋 Qué Tiene Test Obligatorio

| Código | Test requerido | Tipo |
|---|---|---|
| `src/lib/services/[service].ts` | `src/tests/unit/[dominio]/[service].test.ts` | Unit |
| `src/features/[f]/services/[service].ts` | `src/tests/unit/[f]/[service].test.ts` | Unit |
| `src/lib/auth/guards.ts` | `src/tests/unit/auth/guards.test.ts` | Unit |
| `src/lib/utils/[util].ts` | `src/tests/unit/utils/[util].test.ts` | Unit |
| Flujo crítico cross-módulo | `src/tests/integration/[flow].test.ts` | Integration |
| User journey crítico (checkout, onboarding, auth) | `src/tests/e2e/[journey].spec.ts` | E2E |

---

## 🟢 Qué NO Necesita Test

- Componentes UI puramente presentacionales (sin lógica)
- Wiring de rutas (`routeLoader$` que solo llama a un servicio)
- Estilos y clases Tailwind
- Ficheros de configuración (`client.ts`, `schema.ts`)

---

## 📐 Estructura de Tests

```
src/
└── tests/
    ├── unit/                    ← Servicios y utilidades — espeja src/lib/ y src/features/
    │   ├── auth/
    │   │   └── guards.test.ts
    │   ├── [feature]/
    │   │   └── [service].test.ts
    │   └── utils/
    │       └── [util].test.ts
    ├── integration/             ← Flujos entre servicios
    │   └── [flow].test.ts
    ├── e2e/                     ← User journeys críticos (Playwright)
    │   └── [journey].spec.ts
    └── fixtures/                ← Datos de prueba compartidos
        └── [domain].ts
```

**Convención:** `src/tests/unit/[feature]/` espeja exactamente `src/lib/[feature]/` y `src/features/[feature]/services/`.

---

## ✍️ Protocolo del Builder (@QwikBuilder)

Al crear o modificar un servicio, el Builder sigue este protocolo:

**Paso 1 — Identificar si el fichero requiere test**
Aplicar la tabla "Qué Tiene Test Obligatorio". Si aplica → continúa.

**Paso 2 — Crear o actualizar el fichero de test**

```typescript
// src/tests/unit/[feature]/[service].test.ts
import { describe, it, expect, beforeEach } from 'bun:test';
import { [ServiceName] } from '~/lib/[feature]/[service]';

describe('[ServiceName]', () => {
  // Casos happy path
  it('[método] returns [resultado esperado]', async () => {
    // Arrange
    // Act
    // Assert
  });

  // Casos de borde críticos
  it('[método] throws when [condición de error]', async () => {
    // ...
  });

  // Casos de autorización si aplica
  it('[método] rejects unauthorized access', async () => {
    // ...
  });
});
```

**Paso 3 — Ejecutar los tests antes del handoff**

```bash
bun test src/tests/unit/[feature]/[service].test.ts
```

Si los tests fallan → corregir antes de hacer handoff al Auditor.
El Auditor no acepta código con tests en rojo.

**Paso 4 — Incluir en el Delivery Summary**

```markdown
3. **TESTS:**
   - Creados: [lista de ficheros .test.ts nuevos]
   - Actualizados: [lista de ficheros .test.ts modificados]
   - Resultado: [✅ Todos pasan / ❌ N fallos — no debería llegar aquí]
```

---

## 🔍 Protocolo del Auditor (@QwikAuditor)

### BLOQUE K: Cobertura de Tests

> Verificar antes de emitir PASSED.

- [ ] **K-001:** Todo servicio nuevo en `src/lib/` o `src/features/*/services/` tiene su fichero `.test.ts` correspondiente en `src/tests/unit/`
- [ ] **K-002:** Los tests existentes del módulo siguen pasando — `bun test src/tests/unit/[feature]/` sin errores
- [ ] **K-003:** Los casos de borde críticos están cubiertos (error handling, datos vacíos, autorización)
- [ ] **K-004:** El Delivery Summary del Builder declara los tests creados/actualizados

**Criterio de bloqueo:**
- K-001 fallido → 🔴 Crítico — no hay PASSED sin tests para servicios nuevos
- K-002 fallido → 🔴 Crítico — regresión detectada, no hay PASSED
- K-003 fallido → 🟠 Mayor — debe resolverse antes de producción
- K-004 fallido → 🟡 Menor — pedir al Builder que actualice el Delivery Summary

---

## 📊 Coverage Targets

| Capa | Objetivo | Prioridad |
|---|---|---|
| `src/lib/services/` | 90%+ | 🔴 Crítica |
| `src/features/*/services/` | 85%+ | 🔴 Crítica |
| `src/lib/utils/` | 95%+ | 🟡 Alta |
| `src/components/ui/` | 70%+ | 🟢 Media |
| `src/routes/` | 50%+ | 🟢 Baja (E2E cubre) |

```bash
bun test                                     # Todos los tests
bun test src/tests/unit/[feature]/           # Por feature
bun test --coverage                          # Con reporte de cobertura
```

---

## 🔗 Integración en el Flujo SDD

```
@QwikBuilder implementa servicio
    ↓
@QwikBuilder escribe tests (mismo PR, mismo handoff)
    ↓
@QwikBuilder ejecuta tests localmente — todos verdes
    ↓
@QwikBuilder incluye tests en Delivery Summary
    ↓
@QwikAuditor verifica BLOQUE K antes de PASSED
    ↓
@QwikPolisher verifica que bun test --coverage pasa targets
```

Los tests no son un paso extra — son parte de la implementación.
Un servicio sin tests no está terminado.