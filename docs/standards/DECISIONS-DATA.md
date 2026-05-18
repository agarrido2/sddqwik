# DECISIONES DATA — SDD Qwik

> **Propósito:** Decisiones del proyecto sobre la capa de datos (Drizzle + Supabase).
> NO documenta cómo funciona Drizzle internamente — para eso existe Context7.
>
> Context7 IDs: `/drizzle-team/drizzle-orm` · `/supabase/supabase-js`

---

## 1. Estrategia de URLs (Decisión Crítica de Infraestructura)

Dos conexiones, dos propósitos. Confundirlas rompe las migraciones en producción.

```env
# App (Transaction Mode — escala horizontal, Pgbouncer)
DATABASE_URL="postgres://...pooler.supabase.com:6543/postgres?pgbouncer=true"

# Migraciones (Session Mode — permite ALTER TABLE)
DIRECT_URL="postgres://...pooler.supabase.com:5432/postgres"
```

| Conexión | Puerto | `prepare` | Cuándo |
|---|---|---|---|
| `DATABASE_URL` | **6543** + `pgbouncer=true` | `false` (obligatorio) | Toda la app en runtime |
| `DIRECT_URL` | **5432** | default | Solo `drizzle-kit` (migraciones) |

**Error más común:** `prepared statement already exists` → falta `prepare: false` en `client.ts`.

---

## 2. Singleton del Cliente Drizzle

```typescript
import * as schema from '~/lib/schemas/db/schema';

// src/lib/db/client.ts — patrón canónico, no modificar
const globalForDb = globalThis as unknown as { conn: postgres.Sql | undefined };

const conn = globalForDb.conn ?? postgres(ENV.DATABASE_URL, {
  prepare: false,
  max: isProd ? 20 : 5,
  idle_timeout: 20,
  connect_timeout: 10,
  max_lifetime: 60 * 30,
});

if (isDev) globalForDb.conn = conn;
export const db = drizzle(conn, { schema });
```

**Por qué Singleton:** El Hot-Reload de Bun en dev crea instancias nuevas en cada cambio.
Sin el patrón Singleton, agotamos el pool de conexiones en minutos.

---

## 3. Schema-First + Drizzle-Zod (SSOT)

**Regla:** Prohibido escribir schemas Zod manuales para entidades que ya existen en DB.


```typescript
// src/lib/schemas/db/schema.ts — punto de entrada unificado del schema
import { createInsertSchema, createSelectSchema } from 'drizzle-zod';

export * from './schema-domain';
export * from './schema-identity';
export * from './schema-relations';
```


```typescript
// src/lib/schemas/db/schema-identity.ts
export const organizations = pgTable('organizations', {
  // ...
});
```


```typescript
// Los schemas Zod se derivan automáticamente del schema de DB
export const insertOrganizationSchema = createInsertSchema(organizations);
export const selectOrganizationSchema = createSelectSchema(organizations);
```


Los schemas generados tienen dos usos válidos:
- inferencia de tipos con `z.infer<>`
- validación dentro de servicios


**No se pasan directamente a `zod$()` en `routeAction$`** — causa `TS2589`.
En `routeAction$` los schemas Zod se definen siempre inline:


```typescript
// ✅ Schema drizzle-zod para inferencia de tipo y servicios
export const insertOrgSchema = createInsertSchema(organizations);
type InsertOrg = z.infer<typeof insertOrgSchema>;


// ✅ routeAction$ siempre con schema inline — nunca importado
export const useCreateOrg = routeAction$(handler, zod$(z.object({
  name: z.string().min(1),
  slug: z.string().min(1),
})));


// ❌ INCORRECTO — causa TS2589
export const useCreateOrg = routeAction$(handler, zod$(insertOrgSchema));
```


> **Por qué no funciona el schema importado en `zod$()`:**
> La `ActionConstructor` de Qwik City tiene múltiples capas de type-level computation
> (`GetValidatorOutputType`, `ValidatorErrorKeyDotNotation`, `StrictUnion`). Cuando el
> schema es importado desde otro módulo, TypeScript recomputa la cadena completa de tipos
> superando el límite de recursión (~100 niveles) → `TS2589`. Con schemas inline,
> TypeScript evalúa el tipo en el mismo contexto y optimiza la inferencia sin explotar.

## 3.1. Schema particionado por responsabilidad

El schema Drizzle se divide por responsabilidad estructural para reducir acoplamiento,
evitar ciclos y mantener escalabilidad.


### Estructura canónica

```text
src/lib/schemas/db/
├── schema-domain.ts
├── schema-identity.ts
├── schema-relations.ts
└── schema.ts
```


### Reglas de responsabilidad

- `schema-domain.ts`
  - Entidades del dominio de negocio
  - Ejemplos: `contacts`, `categories`, `deals`, `tasks`

- `schema-identity.ts`
  - Identidad, tenant, membresía y acceso
  - Ejemplos: `organizations`, `profiles`, `invited`, `members`

- `schema-relations.ts`
  - Capa transversal de relaciones entre entidades
  - Puede importar desde `schema-domain.ts` y `schema-identity.ts`

- `schema.ts`
  - Punto de entrada unificado
  - Reexporta todos los módulos para mantener compatibilidad de imports


### Regla de dependencias

- `schema-relations.ts` **sí puede** importar desde:
  - `schema-domain.ts`
  - `schema-identity.ts`

- `schema-domain.ts` y `schema-identity.ts` **no deben** importar `schema-relations.ts`

- El resto del sistema debe preferir `src/lib/schemas/db/schema.ts`
  como entrypoint estable, salvo trabajo interno específico sobre un submódulo


> **Razón:**
> Las relaciones dependen de las entidades base.
> Las entidades base no deben depender de la capa transversal de relaciones.

---

## 4. Naming Conventions (Invariante)

| Capa | Convención | Ejemplo |
|---|---|---|
| Tabla SQL | `snake_case` plural | `organizations`, `user_profiles` |
| Columna SQL | `snake_case` | `created_at`, `org_id` |
| TypeScript (Drizzle) | `camelCase` | `createdAt`, `orgId` |
| Relación en código | `camelCase` | `organization.members` |

```typescript
// Ejemplo correcto
export const organizationMembers = pgTable('organization_members', {
  organizationId: uuid('organization_id').references(() => organizations.id),
  userId: uuid('user_id').references(() => users.id),
  createdAt: timestamp('created_at').defaultNow(),
});
```

---

## 5. RLS — Protocolo Declarativo (Obligatorio en Schema)

Toda tabla con `user_id` u `organization_id` DEBE tener su política RLS documentada
en el propio módulo donde se define la tabla (`schema-domain.ts` o `schema-identity.ts`)
o, de forma equivalente, en el punto de entrada unificado `schema.ts` si se mantiene
la documentación centralizada.

```typescript
export const profiles = pgTable('profiles', {
  id: uuid('id').primaryKey().references(() => users.id),
  organizationId: uuid('organization_id').references(() => organizations.id),
  // ...
}).enableRLS();

// @RLS: ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
// @RLS: CREATE POLICY "Users see own org profiles"
//   ON profiles FOR SELECT
//   USING (organization_id = (SELECT organization_id FROM members WHERE user_id = auth.uid()));
// @RLS: CREATE POLICY "Users update own profile"
//   ON profiles FOR UPDATE USING (id = auth.uid());
```

> **Regla práctica:**
> La política RLS debe vivir pegada a la definición real de la tabla o en su reexport
> unificado inmediato, pero nunca separada en documentación lejana o implícita.

**@QwikAuditor verifica:** Toda tabla nueva con datos de usuario tiene bloque `@RLS`.

## 5.1. Ubicación canónica del schema

La capa de datos se divide en dos zonas con responsabilidades distintas:


### Runtime de datos
```text
src/lib/db/client.ts
```


### Definición de datos
```text
src/lib/schemas/db/
├── schema-domain.ts
├── schema-identity.ts
├── schema-relations.ts
└── schema.ts
```


**Regla:**
- `src/lib/db/` contiene conexión, cliente y ejecución
- `src/lib/schemas/db/` contiene tablas, relaciones y contracts estructurales

---

## 6. Middleware de Auth — Patrón `sharedMap`

La sesión se verifica **una sola vez** en el layout `(app)` y se comparte vía `sharedMap`.
Los loaders hijos no repiten la query a Supabase.

```typescript
// src/routes/(app)/layout.tsx — patrón canónico
export const onRequest: RequestHandler = async (requestEv) => {
  const supabase = createSupabaseServerClient(requestEv);
  const { data: { user }, error } = await supabase.auth.getUser();

  if (error || !user) {
    throw requestEv.redirect(302, '/login?error=session_expired');
  }

  requestEv.sharedMap.set('user', user); // disponible para todos los loaders hijos
  await requestEv.next();
};
```

```typescript
// En cualquier routeLoader$ dentro de (app)/
export const useProfile = routeLoader$(async ({ sharedMap }) => {
  const user = sharedMap.get('user'); // sin round-trip adicional
  return ProfileService.getProfile(user.id);
});
```

---

## 7. Reglas de Indexación (Performance)

**Regla:** Toda columna usada en `WHERE`, `JOIN`, u `ORDER BY` frecuente → índice obligatorio.

```typescript
export const auditLogs = pgTable('audit_logs', {
  id: uuid('id').primaryKey(),
  organizationId: uuid('organization_id').references(() => organizations.id),
  action: text('action'),
  createdAt: timestamp('created_at').defaultNow(),
}, (table) => ({
  orgIdx: index('audit_logs_org_idx').on(table.organizationId),
  actionCreatedIdx: index('audit_logs_action_created_idx').on(table.action, table.createdAt),
}));
```

---

## 8. Transacciones — Cuándo y Cómo

Usar `db.transaction()` para operaciones que deben ser atómicas:

```typescript
// Patrón: crear dos entidades relacionadas de forma atómica
static async createWithRelated(userId: string, parentId: string) {
  return await db.transaction(async (tx) => {
    const [parent] = await tx.insert(parentEntities) // ← parentEntities debe venir de lib/schemas/db/
      .values({ userId, parentId, status: 'active' })
      .returning();
    await tx.insert(relatedEntities)
      .values({ parentEntityId: parent.id, status: 'pending' });
    return parent;
  });
}
```

**Reglas de transacciones:**
- Sin llamadas a APIs externas dentro del callback (no son reversibles)
- Sin lógica pesada (bloquea la conexión)
- Usar `.returning()` para obtener datos sin query adicional

---

## 9. Scripts de DB Canónicos (`package.json`)

```json
"db:generate": "drizzle-kit generate",
"db:migrate":  "bun run src/scripts/migrate.ts",
"db:studio":   "drizzle-kit studio",
"db:push":     "drizzle-kit push",
"types:supabase": "npx supabase gen types typescript --project-id $SUPABASE_PROJECT_ID > src/lib/types/supabase.ts"
```

**Flujo obligatorio para cambios de schema:**
1. Editar `src/lib/schemas/db/schema-domain.ts`, `schema-identity.ts`,
   `schema-relations.ts` o `schema.ts` según corresponda
2. `bun run db:generate` — genera SQL
3. Revisar el SQL generado
4. `bun run db:migrate` — aplica en DB

Nunca editar archivos SQL de `drizzle/` manualmente salvo emergencia.

---

## 10. Troubleshooting Rápido

| Error | Causa | Fix |
|---|---|---|
| `prepared statement already exists` | `prepare: false` ausente | Añadirlo en `client.ts` |
| `relation does not exist` en migrate | `DIRECT_URL` con puerto incorrecto | Verificar puerto 5432 |
| `max client connections reached` | Pool saturado | Reducir `max` en `client.ts` |
| `Database 'undefined'` | `supabase.ts` no generado | `bun run types:supabase` |
| `Transaction timeout` | Lógica pesada o API call dentro de `tx` | Mover fuera del callback |
| `Cannot access ... before initialization` o ciclos extraños en schema | Import circular entre módulos del schema | Asegurar que `schema-domain.ts` y `schema-identity.ts` no importan `schema-relations.ts` |

> Para queries complejas, JOINs avanzados, migraciones específicas: usar Context7
> con `/drizzle-team/drizzle-orm` y el query concreto como búsqueda.