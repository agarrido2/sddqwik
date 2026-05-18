# CONTEXT7 MCP — Guía de Library IDs Verificados

> **Propósito:** Evitar alucinaciones de API. Todo agente que necesite verificar
> una librería externa DEBE usar estos IDs verificados con Context7 MCP.
> Última verificación: 2026-Q1

---

## ⚠️ Regla de Oro

> Si no encuentras el library ID en esta guía, usa la herramienta
> `resolve_library_id("nombre")` para buscarlo. Nunca inventes un ID.
> Si Context7 no devuelve resultados, documenta el intento y usa el
> conocimiento del standard correspondiente en `docs/standards/`.

---

## 📦 Library IDs del Proyecto

### Core Framework

| Librería | Library ID | Uso Principal | Agente |
|---|---|---|---|
| Qwik + QwikCity | `/qwikdev/qwik` | Componentes, routing, SSR | @QwikBuilder |
| Qwik (docs adicionales) | `/builderio/qwik` | APIs avanzadas, server$ | @QwikBuilder |

**Queries frecuentes para Qwik:**
```
# Verificar patrón de serialización
"qwik noSerialize useVisibleTask$ serializable"

# Verificar co-localización QRL
"qwik QRL co-location lazy loading optimization"

# Verificar server$ streaming
"qwik server$ async generator streaming"

# Verificar sync$
"qwik sync$ DOM interaction event handler"
```

---

### Base de Datos y Auth

| Librería | Library ID | Uso Principal | Agente |
|---|---|---|---|
| Drizzle ORM | `/drizzle-team/drizzle-orm` | Queries, schema, relaciones | @QwikDBA |
| Supabase JS Client | `/supabase/supabase-js` | Auth, RLS, Realtime | @QwikDBA / @QwikBuilder |
| Supabase Auth Helpers | `/supabase/auth-helpers-nextjs` | SSR Auth patterns | @QwikBuilder |

**Queries frecuentes para Drizzle:**
```
# Relaciones complejas
"drizzle orm one-to-many self-referencing"

# Optimización de queries
"drizzle select with joins pagination"

# Migración específica
"drizzle generate migration alter table"

# RLS con Drizzle
"drizzle supabase row level security enable"
```

**Queries frecuentes para Supabase:**
```
# RLS para multi-tenant
"supabase row level security organization multi-tenant"

# Auth SSR
"supabase auth server side rendering cookies"

# Realtime
"supabase realtime subscribe postgres changes"
```

---

### Validación y Tipos

| Librería | Library ID | Uso Principal | Agente |
|---|---|---|---|
| Zod | `/colinhacks/zod` | Validación de schemas | @QwikBuilder / @QwikDBA |

**Queries frecuentes para Zod:**
```
# Validación de acciones Qwik
"zod schema validation form action"

# Tipos discriminados
"zod discriminated union type inference"
```

---

### Estilos

| Librería | Library ID | Uso Principal | Agente |
|---|---|---|---|
| Tailwind CSS v4 | `/tailwindlabs/tailwindcss` | Sistema de diseño | @QwikBuilder |

**Queries frecuentes para Tailwind v4:**
```
# Configuración CSS-first
"tailwind v4 @theme configuration"

# Dark mode variables
"tailwind v4 dark mode CSS variables"

# Clases dinámicas con Qwik
"tailwind dynamic classes conditional"
```

---

### Runtime y Tooling

| Librería | Library ID | Uso Principal | Agente |
|---|---|---|---|
| Bun | `/oven-sh/bun` | Scripts, comandos | Todos |
| Vitest | `/vitest-dev/vitest` | Tests | @QwikBuilder |

---

### Integraciones Externas del Proyecto

Cada proyecto debe mantener esta sección con sus propias integraciones verificadas.
Antes de integrar cualquier librería externa, resolver el library ID con Context7:

```
resolve_library_id("nombre-libreria")
```

Añadir la entrada a esta tabla una vez verificada:

| Librería | Library ID | Uso Principal | Agente |
|---|---|---|---|
| _(añadir según necesidades del proyecto)_ | | | |

> ⚠️ Nunca asumir un library ID de memoria. Siempre verificar con
> `resolve_library_id()` antes de usarlo por primera vez.

---

## 🔧 Cómo Usar Context7 Correctamente

### Paso 1: Resolver el Library ID
```
// En el agente, antes de codificar:
use_mcp_tool("context7", "resolve_library_id", { libraryName: "drizzle-orm" })
// Resultado esperado: { library_id: "/drizzle-team/drizzle-orm", ... }
```

### Paso 2: Obtener Documentación Relevante
```
use_mcp_tool("context7", "get_library_docs", {
  library_id: "/drizzle-team/drizzle-orm",
  topic: "one-to-many relationship self-referencing",
  tokens: 5000  // Ajustar según necesidad (1000-10000)
})
```

### Paso 3: Aplicar y Documentar
Si el resultado cambia algo respecto a lo que estabas usando:
1. Aplica el patrón correcto
2. Documenta en el Plan File: "Verificado con Context7 [fecha]: [hallazgo]"
3. Si es un cambio breaking: crea ADR en `docs/adr/`

---

## 📊 Cuándo NO Usar Context7

| Caso | Acción alternativa |
|---|---|
| APIs internas del proyecto | Leer `src/lib/db/schema.ts` o `src/lib/types/` |
| Patrones del proyecto | Leer `docs/standards/DECISIONS-QWIK.md` |
| Decisiones de arquitectura pasadas | Leer `docs/adr/` |
| Standard del proyecto | Leer `docs/standards/[STANDARD].md` |

Context7 es para librerías externas. Para conocimiento del proyecto, los standards
y el código fuente son la fuente de verdad.

---

## 🔄 Mantenimiento

Este documento debe actualizarse cuando:
- Se añade una nueva librería al proyecto
- Se verifica que un library ID ha cambiado
- Se documenta un patrón de query útil nuevo

**Responsable de actualización:** @QwikArchitect (al añadir integraciones)