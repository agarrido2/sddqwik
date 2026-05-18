# SECURITY POLICIES — SDD Qwik

> **Propósito:** Define los patrones de seguridad de datos obligatorios para toda
> tabla que contenga datos de usuario. Elimina la ambigüedad del "buen juicio" —
> estos patrones son reglas, no sugerencias.
>
> **Owner:** @QwikDBA (aplica) · @QwikAuditor (verifica) · @QwikArchitect (consulta)

---

## 🔴 Regla Absoluta

**Toda tabla con `user_id` u `organization_id` DEBE tener RLS.**
Sin excepción. Sin tabla "provisional". Sin "lo añado después".

Si el DBA crea una tabla con datos de usuario sin RLS → issue 🔴 Crítico en auditoría.

---

## 📋 Los 3 Patrones RLS Canónicos

### Patrón 1 — Aislamiento por Usuario (datos personales)

Para tablas donde cada fila pertenece a un único usuario.

```sql
-- Habilitar RLS
ALTER TABLE [tabla] ENABLE ROW LEVEL SECURITY;

-- Solo el propietario ve sus filas
CREATE POLICY "[tabla]_select_own"
  ON [tabla] FOR SELECT
  USING (user_id = auth.uid());

-- Solo el propietario modifica sus filas
CREATE POLICY "[tabla]_modify_own"
  ON [tabla] FOR ALL
  USING (user_id = auth.uid());
```

**Cuándo usar:** `profiles`, `preferences`, `personal_data`

---

### Patrón 2 — Aislamiento por Organización (multi-tenant)

Para tablas donde las filas pertenecen a una organización y todos sus miembros
tienen acceso según su rol.

```sql
ALTER TABLE [tabla] ENABLE ROW LEVEL SECURITY;

-- Miembros de la organización ven los datos de su org
CREATE POLICY "[tabla]_select_org"
  ON [tabla] FOR SELECT
  USING (
    organization_id IN (
      SELECT organization_id FROM members
      WHERE user_id = auth.uid()
    )
  );

-- Solo owner y admin pueden modificar
CREATE POLICY "[tabla]_modify_admin"
  ON [tabla] FOR INSERT, UPDATE, DELETE
  USING (
    organization_id IN (
      SELECT organization_id FROM members
      WHERE user_id = auth.uid()
        AND role IN ('owner', 'admin')
    )
  );
```

**Cuándo usar:** Toda tabla core del SaaS — `agents`, `configurations`, `resources`

---

### Patrón 3 — Datos Públicos con Escritura Controlada

Para tablas con contenido público (lectura libre) pero escritura restringida.

```sql
ALTER TABLE [tabla] ENABLE ROW LEVEL SECURITY;

-- Lectura pública
CREATE POLICY "[tabla]_select_public"
  ON [tabla] FOR SELECT
  USING (true);

-- Escritura solo por owner de la organización
CREATE POLICY "[tabla]_insert_owner"
  ON [tabla] FOR INSERT, UPDATE, DELETE
  USING (
    organization_id IN (
      SELECT organization_id FROM members
      WHERE user_id = auth.uid()
        AND role = 'owner'
    )
  );
```

**Cuándo usar:** `products`, `catalog`, `public_profiles`

---

## 🚫 Anti-Patrones Prohibidos

| Anti-patrón | Consecuencia | Alternativa |
|---|---|---|
| `USING (true)` en UPDATE/DELETE | Cualquier usuario modifica cualquier fila | Patrón 1 o 2 según el dominio |
| Tabla sin RLS con `organization_id` | Data leak entre organizaciones | RLS obligatorio — Patrón 2 |
| Política solo para SELECT, sin INSERT | Escritura desprotegida | Política `FOR ALL` o políticas separadas por operación |
| RLS desactivado "para testing" | Vulnerabilidad que llega a producción | Usar datos de test con usuario real en dev |
| `auth.uid()` hardcodeado en servicio | Bypass de RLS posible | RLS en DB, no en aplicación |

---

## 🔍 Checklist de Verificación (@QwikAuditor BLOQUE F2)

Para cada tabla nueva o modificada por @QwikDBA:

- [ ] **RLS habilitado:** `.enableRLS()` en `schema.ts`
- [ ] **Patrón identificado:** ¿Cuál de los 3 patrones aplica? Documentado en `@RLS:` del schema
- [ ] **SELECT protegido:** Usuario solo ve sus datos o los de su organización
- [ ] **WRITE protegido:** INSERT/UPDATE/DELETE requiere rol adecuado
- [ ] **Sin `USING (true)` en escritura:** Verificado en el SQL generado
- [ ] **Política coherente con RBAC:** Los roles en la política coinciden con `RBAC_ROLES_PERMISSIONS.md`
- [ ] **Migración revisada:** El SQL en `drizzle/` refleja exactamente las políticas del schema

---

## 📌 Integración con RBAC

Las políticas RLS usan los roles definidos en `RBAC_ROLES_PERMISSIONS.md`.
Correspondencia canónica:

| Rol en RBAC | Capacidad en RLS |
|---|---|
| `owner` | SELECT + INSERT + UPDATE + DELETE |
| `admin` | SELECT + INSERT + UPDATE (sin DELETE en recursos críticos) |
| `member` | SELECT (datos de su org) + INSERT limitado |
| `invited` | SELECT (datos demo/preview únicamente) |

**Regla:** Si un rol no está en esta tabla, no tiene acceso. El default es DENY.

---

## 🗺️ Dónde Declarar las Políticas

Las políticas RLS se declaran en `src/lib/db/schema.ts` como comentarios
`@RLS:` inmediatamente después de cada tabla, siguiendo el patrón de
`DECISIONS_DATA.md` §5. Esto mantiene el schema como SSOT.

```typescript
export const agentConfigs = pgTable('agent_configs', {
  id: uuid('id').primaryKey(),
  organizationId: uuid('organization_id').references(() => organizations.id),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
}).enableRLS();

// @RLS: PATRÓN 2 — Aislamiento por Organización
// @RLS: SELECT → members de la misma org
// @RLS: INSERT/UPDATE/DELETE → solo owner y admin
// Ver SECURITY_POLICIES.md §Patrón-2
```