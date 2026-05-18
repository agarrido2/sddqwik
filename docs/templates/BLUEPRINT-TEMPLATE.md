# BLUEPRINT — Technical Architecture & Delivery Plan

> **Proyecto:** [Nombre del proyecto]
> **PRD de referencia:** `docs/prd/[proyecto]-prd.md`
> **Fecha:** [fecha]
> **Versión:** 1.0
> **Estado:** 🟡 Draft | 🟠 Review | 🟢 Aprobado

---

## PARTE 0: CÓMO USAR ESTE DOCUMENTO

Este Blueprint traduce el PRD aprobado en un plano técnico ejecutable.
Su propósito es responder tres preguntas antes de escribir una sola línea de código:

1. **¿Qué módulos tiene la aplicación?** → Sección 1
2. **¿En qué orden se construyen?** → Sección 2
3. **¿Cómo se implementa cada módulo con SDD Qwik?** → Sección 3

**Flujo de trabajo:**
```text
PRD aprobado
    ↓
Blueprint (este documento) — aprobado por el equipo técnico
    ↓
Por cada módulo de cada fase:
    /spec [módulo]         → @QwikSpeccer genera la Spec formal
    /new-feature [módulo]  → Ciclo completo SDD Qwik
```

> **Regla:** No iniciar `/spec` sin Blueprint aprobado.
> No iniciar `/new-feature` sin Spec aprobada.
> El Blueprint define el orden. La Spec define el detalle.

---

## PARTE 1: MAPA DE MÓDULOS

> Descompón la aplicación en módulos independientes.
> Cada módulo se convertirá en una o más Specs de SDD Qwik.
> Un módulo = una unidad coherente de funcionalidad con límites claros.

### 1.1 Módulos identificados

| # | Módulo | Descripción | Tipo | Dependencias |
|---|---|---|---|---|
| M01 | [ej: Auth] | [ej: Registro, login, recuperación] | Core | — |
| M02 | [ej: Catálogo] | [ej: Productos, categorías, búsqueda] | Core | M01 |
| M03 | [ej: Carrito] | [ej: Añadir, modificar, vaciar] | Core | M01, M02 |
| M04 | [ej: Checkout] | [ej: Pago, dirección, confirmación] | Core | M03 |
| M05 | [ej: Dashboard Admin] | [ej: Gestión de productos y pedidos] | Admin | M01 |
| M06 | [ej: Email transaccional] | [ej: Confirmación, envío, recuperación] | Infraestructura | M04 |

**Tipos:**
- **Core** — funcionalidad visible para el usuario final
- **Admin** — panel de gestión interno
- **Infraestructura** — servicios transversales (auth, email, pagos, notificaciones)
- **Integraciones** — conexiones con servicios externos

### 1.2 Mapa de dependencias


```text
[M01 Auth]
    ↓
[M02 Catálogo] ──→ [M05 Dashboard Admin]
    ↓
[M03 Carrito]
    ↓
[M04 Checkout] ──→ [M06 Email]
```

---

## PARTE 2: FASES DE ENTREGA

> Agrupa los módulos en fases. Cada fase debe ser entregable y demostrable por sí sola.
> **Criterio de priorización:** valor para el usuario > dependencias técnicas > complejidad.

### Fase 0 — Fundamentos (sin esta fase, nada funciona)
**Objetivo:** Infraestructura técnica operativa.  
**Entregable:** Proyecto desplegado, CI/CD configurado, auth funcionando.

| Módulo | Descripción | Specs necesarias |
|---|---|---|
| Setup del proyecto | Qwik + Supabase + Tailwind + Drizzle | — |
| M01 Auth | Registro, login, sesión, rutas protegidas | `spec/auth-core` |

**Criterio de salida de la fase:** Usuario puede registrarse y acceder a su dashboard.

---

### Fase 1 — MVP (mínimo que tiene valor para el cliente)
**Objetivo:** La funcionalidad core que justifica el producto.  
**Entregable:** Versión usable por usuarios reales, aunque limitada.

| Módulo | Descripción | Specs necesarias | Prioridad |
|---|---|---|---|
| M02 Catálogo | Listado y detalle de productos | `spec/catalog-listing`, `spec/product-detail` | 🔴 Alta |
| M05 Dashboard Admin | Alta y edición de productos | `spec/admin-products` | 🔴 Alta |

**Criterio de salida de la fase:** Cliente puede publicar productos. Usuario puede verlos.

---

### Fase 2 — Funcionalidad completa
**Objetivo:** Producto completo según el PRD aprobado.  
**Entregable:** Todo el alcance de Fase 1 del PRD.

| Módulo | Descripción | Specs necesarias | Prioridad |
|---|---|---|---|
| M03 Carrito | Gestión del carrito de compra | `spec/cart` | 🔴 Alta |
| M04 Checkout | Proceso de pago completo | `spec/checkout-flow` | 🔴 Alta |
| M06 Email | Confirmaciones y notificaciones | `spec/transactional-email` | 🟠 Media |

**Criterio de salida de la fase:** Usuario puede completar una compra de extremo a extremo.

---

### Fase 3 — Optimización y extras
**Objetivo:** Mejoras post-lanzamiento basadas en feedback real.  
**Entregable:** Features de Fase 2 del PRD (las que estaban OUT OF SCOPE).

| Módulo | Descripción | Specs necesarias | Prioridad |
|---|---|---|---|
| [módulo] | [descripción] | [specs] | 💡 Baja |

---

## PARTE 3: DETALLE TÉCNICO POR MÓDULO

> Para cada módulo, define las decisiones técnicas antes de crear las Specs.
> Este documento orienta a @QwikBlueprint y prepara el terreno para @QwikSpeccer y @QwikArchitect.
> Debe ser coherente con `docs/standards/ARQUITECTURA-FOLDER.md`, `DECISIONS-DATA.md`,
> `RBAC-ROLES-PERMISSIONS.md` y `CONTEXT7-GUIDE.md` cuando aplique.

---

### M01 — Auth

**Descripción:** Sistema de autenticación y gestión de sesión.

**Stack:**
- Supabase Auth (email/password + OAuth si aplica)
- Guards y helpers en `src/lib/auth/`
- Layout protegido en `src/routes/(app)/layout.tsx`
- Contextos definidos en `src/lib/contexts/` si fueran necesarios

**Rutas:**
```text
src/routes/
├── (auth)/
│   ├── layout.tsx
│   ├── login/
│   │   └── index.tsx
│   ├── register/
│   │   └── index.tsx
│   ├── forgot-password/
│   └── reset-password/
└── (app)/
    └── layout.tsx          ← Auth guard + App shell
```

**Schema DB:**
```text
schema-identity.ts
  - organizations
  - profiles
  - invited / memberships / roles (si aplica)

schema-domain.ts
  - [normalmente vacío para auth pura]

schema-relations.ts
  - relaciones entre profiles, organizations y entidades de acceso

schema.ts
  - reexport unificado del schema completo
```

**Roles necesarios:** [listar roles del PRD — ver `RBAC-ROLES-PERMISSIONS.md`]

**Specs a crear:**
- [ ] `/spec auth-register` — Registro de usuario
- [ ] `/spec auth-login` — Login y gestión de sesión
- [ ] `/spec auth-password-reset` — Recuperación de contraseña

---

### M02 — [Nombre del módulo]

**Descripción:** [qué hace]

**Stack / decisiones técnicas:**
- [decisión 1]
- [decisión 2]
- [decisión ...]

**Rutas:**
```text
src/routes/
└── (app o public)/
    └── [ruta]/
        └── index.tsx
```

**Schema DB:**
```text
schema-identity.ts
  - [tablas de identidad/tenant si aplica]

schema-domain.ts
  - [tablas del dominio]

schema-relations.ts
  - [relaciones transversales con otras entidades]

schema.ts
  - reexport unificado del schema completo
```

**Integraciones externas:** [si las hay — verificar IDs en `CONTEXT7-GUIDE.md`]

**Specs a crear:**
- [ ] `/spec [nombre-spec]` — [descripción]

---

*(repetir para cada módulo)*

---

## PARTE 4: DECISIONES ARQUITECTÓNICAS GLOBALES

> Decisiones que afectan a toda la aplicación.
> Una vez tomadas aquí, se reflejan en los ADRs de `docs/adr/`.

### Base de datos
- [ ] ¿Necesita multi-tenant (varias organizaciones)? → RLS por `organization_id`
- [ ] ¿Hay datos de alto volumen que necesiten particionamiento?
- [ ] ¿Necesita búsqueda full-text? → `pg_trgm` o `pgvector`
- [ ] ¿Hay datos en tiempo real? → Supabase Realtime
- [ ] ¿Se usará schema dividido? → `schema-domain.ts` + `schema-identity.ts` + `schema-relations.ts` + `schema.ts`

### Autenticación
- [ ] Solo email/password
- [ ] OAuth (Google, GitHub, etc.)
- [ ] Magic link
- [ ] SSO empresarial

### Pagos (si aplica)
- [ ] Proveedor seleccionado: [Stripe / PayPal /LemonSqueezy / otro]
- [ ] Modelo: pago único / suscripción / marketplace

### Email (si aplica)
- [ ] Proveedor: [Resend / SendGrid / otro]
- [ ] Tipos: transaccional / marketing / ambos

### Almacenamiento de ficheros (si aplica)
- [ ] Supabase Storage
- [ ] S3 / R2
- [ ] Tipos de ficheros y tamaños máximos

### Internacionalización
- [ ] Solo un idioma
- [ ] Multi-idioma — idiomas: [UK, ES, PT, FR, DE]

---

## PARTE 5: ESTRUCTURA DE CARPETAS PREVISTA

> Anticipa la estructura `src/` para este proyecto específico.
> Debe respetar la arquitectura canónica definida en `docs/standards/ARQUITECTURA-FOLDER.md`.
> `src/features/` solo se introduce si un dominio supera el umbral de complejidad definido por la arquitectura del proyecto.


```text
public/
├── favicon.svg
├── manifest.json
├── robots.txt

src/
├── assets/
│   ├── css/
│   │   ├── global.css
│   │   └── fonts.css
│   └── fonts/
├── components/
│   ├── icons/
│   ├── ui/
│   ├── shared/
│   └── layout/
├── hooks/
├── lib/
│   ├── auth/
│   ├── contexts/
│   ├── db/
│   │   └── client.ts
│   └── schemas/
│       ├── schema-domain.ts
│       ├── schema-identity.ts
│       ├── schema-relations.ts
│       └── schema.ts
│   ├── services/
│   ├── supabase/
│   ├── types/
│   └── utils/
├── routes/
│   ├── api/
│   ├── (public)/
│   ├── (auth)/
│   ├── (app)/
│   ├── layout.tsx
│   └── service-worker.ts
└── features/
    ├── [solo si aplica por complejidad]
    └── [modulo-complejo]/
```

### Regla estructural
- `src/routes/` orquesta; no contiene lógica de negocio.
- `src/components/` contiene UI reutilizable y composición visual.
- `src/lib/` contiene lógica de negocio, auth, datos, validación e integraciones.
- `src/features/` **no es obligatorio**; se usa solo para dominios complejos con múltiples archivos relacionados.
- `src/lib/db/schema.ts` actúa como punto de entrada unificado del schema y reexporta los submódulos.

### Estructura de schema DB
- `schema-domain.ts` → entidades del dominio de negocio (`contacts`, `categories`, etc.)
- `schema-identity.ts` → identidad, tenant y acceso (`organizations`, `profiles`, `invited`, etc.)
- `schema-relations.ts` → relaciones transversales entre entidades; importa de `schema-domain.ts` y `schema-identity.ts` y **no debe ser importado por ellos**
- `schema.ts` → punto de entrada unificado; reexporta todos los módulos para mantener compatibilidad con imports existentes

---

## PARTE 6: CHECKLIST DE ARRANQUE

> Completa esto antes de ejecutar el primer `/spec`.

### Infraestructura
- [ ] Repositorio creado y configurado
- [ ] Proyecto Supabase creado (dev + prod)
- [ ] Variables de entorno configuradas (`.env.local`)
- [ ] Proyecto Qwik inicializado con el stack correcto
- [ ] `/setup` ejecutado — workspace SDD Qwik verificado

### Decisiones cerradas
- [ ] Roles de usuario definidos (ver `RBAC-ROLES-PERMISSIONS.md`)
- [ ] Schema DB inicial esbozado (Parte 3 de este Blueprint)
- [ ] Estructura de schema validada (`schema-domain.ts`, `schema-identity.ts`, `schema-relations.ts`, `schema.ts`)
- [ ] Integraciones externas identificadas (Parte 4)
- [ ] Orden de fases aprobado por el cliente (Parte 2)

### Documentación
- [ ] PRD aprobado por el cliente
- [ ] Blueprint aprobado por el equipo técnico
- [ ] Primer ADR creado: decisión de stack/arquitectura global

---

## PARTE 7: HISTORIAL Y APROBACIONES

| Versión | Fecha | Autor | Cambio |
|---|---|---|---|
| 1.0 | [fecha] | [nombre] | Draft inicial |

**Aprobado técnicamente:** [ ] Sí — Fecha: ___________

---

> **Siguiente paso tras la aprobación:**
> Ejecutar `/setup` para verificar el workspace y comenzar con
> `/spec [primer-módulo-de-fase-0]`.