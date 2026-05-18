```sql
-- ============================================================
-- 1. TABLA PRINCIPAL: contacts (sin CHECK de dominios)
-- ============================================================
CREATE TABLE public.contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES public.organizations(id) ON DELETE CASCADE,
    type_contact TEXT NOT NULL,  -- especifica si es 'company' o 'person'. El check se gestiona en el frontend para mayor flexibilidad.
    tax_id TEXT NOT NULL, -- CIF/NIF para empresas o DNI para personas. Único por organización.
    external_id TEXT, -- ID externo opcional para integración con sistemas de terceros. Único por organización si se proporciona.
    salutation TEXT, -- Tratamiento: sr, sra, dr, dra, excmo, ilustre, sc. Validado en frontend.
    first_name TEXT, -- Solo para personas físicas. Para empresas, se puede usar contact_name o trade_name.
    last_name TEXT, -- Solo para personas físicas. Para empresas, se puede usar contact_name o trade_name.
    full_name TEXT NOT NULL, -- Nombre completo para búsquedas y visualización. Para personas: concatenación de first_name + last_name. Para empresas: contact_name o trade_name.
    trade_name TEXT, -- Nombre comercial de la marca (opcional, principalmente para empresas)
    email TEXT NOT NULL, -- Email oficial de notificaciones. Para personas físicas, suele ser el email personal. Para empresas, puede ser un email genérico o de contacto.
    website TEXT, -- URL web corporativa
    phone_1 TEXT NOT NULL, -- Teléfono primario de contacto
    phone_2 TEXT, -- Teléfono secundario de contacto (opcional)
    phone_whatsapp TEXT, -- Teléfono para alertas/mensajería (WhatsApp)
    birth_date DATE, -- Fecha de nacimiento (para personas físicas) o fecha de constitución (para empresas)
    gender TEXT, -- Genero (male, female, other) - solo para personas físicas. Para empresas, se puede dejar NULL o usar un valor genérico.
    profile_image TEXT, -- URL de la imagen de perfil
    preferred_language TEXT NOT NULL DEFAULT 'es', -- Idioma para traducción de emails automáticos (ISO 639-1)
    timezone TEXT NOT NULL DEFAULT 'Europe/Madrid', -- Zona horaria para registros de auditoría (IANA)
    currency TEXT NOT NULL DEFAULT 'EUR', -- Divisa preferente de la entidad (ISO 4217)
    contact_name TEXT, -- Nombre de contacto principal (si es una empresa o contacto directo para personas)
    position TEXT, -- Cargo o posición del contacto principal dentro de la empresa (si aplica)
    email_contact TEXT, -- Email de contacto directo (si es diferente al email oficial)
    phone_contact TEXT, -- Teléfono de contacto directo (si es diferente al teléfono primario)
    phone_contact_whatsapp TEXT, -- Teléfono de contacto para WhatsApp (si es diferente al teléfono de WhatsApp principal)
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(), -- Fecha de creación del registro
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now() -- Fecha de última actualización del registro
);

-- ============================================================
-- 2. COLUMNAS JSONB AUXILIARES (sin duplicar campos de primer nivel)
-- ============================================================
ALTER TABLE public.contacts ADD COLUMN address_json JSONB NOT NULL DEFAULT '{}'::jsonb; -- Dirección estructurada (ver esquema JSONB abajo)
ALTER TABLE public.contacts ADD COLUMN professional_data_json JSONB NOT NULL DEFAULT '{}'::jsonb; -- Sector e influencia (IA Context)
ALTER TABLE public.contacts ADD COLUMN compliance_json JSONB NOT NULL DEFAULT '{}'::jsonb; -- Estados KYC y evidencias RGPD con firma
ALTER TABLE public.contacts ADD COLUMN social_links JSONB NOT NULL DEFAULT '{}'::jsonb; -- Enlaces a redes sociales u otros perfiles (LinkedIn, Twitter, etc.) en formato JSONB
ALTER TABLE public.contacts ADD COLUMN custom_fields_json JSONB NOT NULL DEFAULT '{}'::jsonb; -- Campos personalizados opcionales para flexibilidad futura

-- ============================================================
-- ESQUEMA JSONB SUGERIDO PARA address_json (puedes adaptarlo según necesidades)
-- {
--     "siglas": "CL", -- Siglas (ej: CL para calle, AV para avenida, etc.) viene del frontend.
--     "address": "Cristobal Colon", -- Nombre de la calle o avenida
--     "extension": "42 - 2A", -- Extensión del domicilio (número, piso, puerta, etc.)
--     "postal_code": "21001", -- Código postal
--     "city": "Huelva", -- Ciudad 
--     "province": "Huelva", -- Provincia
--     "country": "España"
--     "region": "Andalucia" -- Región (opcional, para países que lo requieran - comunidad autónoma, estado, etc.)
-- }
-- ============================================================

-- ============================================================
-- ESQUEMA JSONB SUGERIDO PARA professional_data_json (puedes adaptarlo según necesidades)
-- {
--      // CAMPOS APLICABLES A EMPRESAS (type_contact = 'company'))
--     "sector": "Tecnología", -- Sector de actividad (ej: Tecnología, Salud, Finanzas, etc.)
--     "category": "Software", -- Categoría dentro del sector (ej: Software, Hardware, Servicios, etc.) segmentación interna.
--     "parent_group": "Grupo TechGlobal", -- Grupo empresarial al que pertenece (si aplica)
--     // CAMPOS APLICABLES A PERSONAS FÍSICAS (type_contact = 'person')
--     "occupation": "Ingeniero de Software", -- Ocupación o profesión oficial
--     "department": "Desarrollo", -- Departamento o área de trabajo (si aplica)
--     "company_link": "f47ac10b-58cc-4372-a567-0e02b2c3d479"  -- uuid de otro cotacto de tipo empresa (si aplica) para vincular persona con empresa
--     // CAMPO TRANSVERSAL (APLICABLE A EMPRESAS Y PERSONAS FÍSICAS)
--     "influence_level": "Alto", -- Nivel de influencia o toma de decisiones
-- }
-- ============================================================

-- ============================================================
-- ESQUEMA JSONB SUGERIDO PARA compliance_json (puedes adaptarlo según necesidades)
-- {
--  // BLOQUE 1: ESTADO DE VERIFICACIÓN KYC (Know Your Customer)
--  "kyc_status": "string",             // 'pending', 'verified', 'expired', 'failed'
--  "kyc_last_review_at": "string",     // ISO Timestamp (UTC) de la última revisión/verificación
--  "risk_level": "string",             // 'low', 'medium', 'high'

--  // BLOQUE 2: BASE LEGAL Y CONSENTIMIENTOS RGPD (Obligatorio)
--  "gdpr_legal_basis": "string",       // 'consent', 'contract', 'legal_obligation', 'legitimate_interest' (Art. 6 RGPD)
--  "gdpr_consent_status": {            // REQUERIDO SOLO SI legal_basis = 'consent'
--    "data_treatment": "boolean",      // Flag obligatorio: aceptación del tratamiento de datos (Art. 4(11) RGPD)
--    "commercial": "boolean",          // Flag opcional: aceptación de comunicaciones comerciales
--    "timestamp": "string",            // ISO Timestamp (UTC) de la acción de consentimiento
--    "source": "string",               // Origen (ej. 'web_form_2024_v2', 'api_contract')
--    "ip_address": "string",           // Dirección IP desde la que se dio el consentimiento
--    "user_agent": "string"            // User Agent del navegador/dispositivo
--  },
--  "gdpr_consent_withdrawal": {        // Registro de retirada de consentimiento
--    "withdrawn_at": "string",         // ISO Timestamp (UTC) de la retirada
--    "reason": "string"                // Causa o motivo de la retirada (opcional)
--  },

--  // BLOQUE 3: EVIDENCIA Y REGISTRO DE AUDITORÍA (Obligatorio)
--  "gdpr_evidence_url": "string",      // Ruta en Supabase Storage del documento probatorio (ej. PDF firmado)
--  "gdpr_evidence_hash": "string",     // Hash SHA-256 del documento para asegurar su integridad no repudio

--  // BLOQUE 4: GESTIÓN DEL RIESGO Y CUMPLIMIENTO
--  "pep_status": "boolean",            // Si es una Persona Expuesta Políticamente (PEP)
--  "sanctions_list": "boolean",        // Si aparece en listas de sanciones internacionales
--  "last_risk_assessment": "string"    // ISO Timestamp (UTC) de la última evaluación de riesgo
-- }
-- ============================================================

-- ============================================================
-- ESQUEMA JSONB SUGERIDO PARA social_links (puedes adaptarlo según necesidades)
-- {
--     "linkedin": {
--         "url": "https://www.linkedin.com/in/ejemplo",
--         "username": "ejemplo",
--         "custom-label": "LinkedIn Personal" // Etiqueta personalizada para mostrar en UI}
--     },
--     "twitter": {
--         "url": "https://twitter.com/ejemplo",
--         "username": "ejemplo",
--         "custom-label": "Twitter Oficial" // Etiqueta personalizada para mostrar en UI
--     },
--     "facebook": {
--         "url": "https://www.facebook.com/ejemplo",
--         "username": "ejemplo",
--         "custom-label": "Facebook" // Etiqueta personalizada para mostrar en UI
--     },
--     "instagram": {
--         "url": "https://www.instagram.com/ejemplo",
--         "username": "ejemplo",
--         "custom-label": "Instagram" // Etiqueta personalizada para mostrar en UI
--     }
-- } 
-- ============================================================

-- ============================================================
-- ESQUEMA JSONB SUGERIDO PARA custom_fields_json (puedes adaptarlo según necesidades)
-- {
--   "custom_field_1": {
--     "value": "cualquier tipo (string, number, boolean, array, object)",
--     "label": "Nombre visible del campo",
--     "type": "text | number | date | checkbox | select | multi_select | url | phone | email",
--     "options": ["opcion1", "opcion2"],   // solo para type = 'select' o 'multi_select'
--     "order": 1,
--     "group": "Información adicional",
--     "is_required": false,
--     "is_visible": true
--   },
--   "custom_field_2": {
--     "value": "Texto libre",
--     "label": "Campo personalizado del CRM",
--     "type": "text",
--     "order": 2
-- },
--  "preferred_contact_time": {
--  "value": "mañanas",
--   "label": "Horario preferente de contacto",
--   "type": "text"
-- },
--  "marketing_consent_notes": {
--    "value": "Acepta recibir ofertas solo por email",
--    "label": "Notas sobre consentimiento marketing",
--    "type": "text"
--  }
-- }
-- ============================================================


-- ============================================================
-- 3. TABLA SIDECAR: datos privados por usuario
-- ============================================================
CREATE TABLE public.contact_user_private_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contact_id UUID NOT NULL REFERENCES public.contacts(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    private_notes_html TEXT,
    vault_credentials_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT unique_contact_user_private UNIQUE (contact_id, user_id)
);

-- ============================================================
-- 4. ÍNDICES
-- ============================================================
-- Multi-tenant y búsquedas frecuentes
CREATE INDEX idx_contacts_organization_id ON public.contacts(organization_id);
CREATE INDEX idx_contacts_type_contact ON public.contacts(type_contact);
CREATE UNIQUE INDEX idx_contacts_tax_id_per_org ON public.contacts(organization_id, tax_id);
CREATE UNIQUE INDEX idx_contacts_external_id_per_org ON public.contacts(organization_id, external_id) WHERE external_id IS NOT NULL;
CREATE INDEX idx_contacts_email ON public.contacts(email);
CREATE INDEX idx_contacts_full_name ON public.contacts(full_name);
CREATE INDEX idx_contacts_created_at ON public.contacts(created_at);

-- Índice compuesto para consultas del usuario activo en sidecar
CREATE INDEX idx_private_user_contact ON public.contact_user_private_data(user_id, contact_id);

-- Índices GIN opcionales si necesitas búsquedas dentro de JSONB (ej: social_links)
-- CREATE INDEX idx_contacts_social_links ON public.contacts USING GIN (social_links);

-- ============================================================
-- 5. ROW LEVEL SECURITY (RLS) - CONFIGURACIÓN COMPLETA
-- ============================================================
-- Activar RLS en ambas tablas
ALTER TABLE public.contacts ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.contact_user_private_data ENABLE ROW LEVEL SECURITY;

-- ============================================================
-- POLÍTICAS PARA LA TABLA contacts
-- ============================================================
-- 1. Política para SELECT, UPDATE y DELETE:
--    El usuario solo puede ver/modificar/borrar contactos de su propia organización.
CREATE POLICY "Usuarios gestionan contacts de su organización"
ON public.contacts
FOR ALL
USING (
    organization_id = (SELECT organization_id FROM public.users WHERE auth.uid() = users.id)
);

-- 2. Política específica para INSERT:
--    Comprueba que la organización que se intenta insertar coincide con la del usuario.
--    (En INSERT, la cláusula USING no se evalúa, por eso es necesaria esta política aparte).
CREATE POLICY "Usuarios insertan contacts de su organización"
ON public.contacts
FOR INSERT
WITH CHECK (
    organization_id = (SELECT organization_id FROM public.users WHERE auth.uid() = users.id)
);
-- ============================================================
-- POLÍTICAS PARA LA TABLA sidecar (contact_user_private_data)
-- ============================================================

-- Los usuarios solo pueden ver y modificar sus propios datos privados.
-- (Esta política sirve para SELECT, UPDATE, DELETE e INSERT porque
--  user_id está presente en la nueva fila y la condición USING
--  se aplica implícitamente como WITH CHECK en operaciones de escritura).
CREATE POLICY "Usuarios solo ven sus propios datos privados"
ON public.contact_user_private_data
FOR ALL
USING (user_id = auth.uid());


-- ============================================================
-- 6. TRIGGER PARA ACTUALIZAR updated_at AUTOMÁTICAMENTE
-- ============================================================
CREATE OR REPLACE FUNCTION public.update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_contacts_updated_at
    BEFORE UPDATE ON public.contacts
    FOR EACH ROW
    EXECUTE FUNCTION public.update_updated_at_column();

CREATE TRIGGER trigger_private_data_updated_at
    BEFORE UPDATE ON public.contact_user_private_data
    FOR EACH ROW
    EXECUTE FUNCTION public.update_updated_at_column();

-- ============================================================
-- 7. COMENTARIOS EXTENSIVOS PARA DOCUMENTACIÓN
-- ============================================================

-- Tabla principal
COMMENT ON TABLE public.contacts IS 'Entidad multi-tenant que almacena contactos (empresas o personas). La validación de dominios (type_contact, salutation, gender) se realiza exclusivamente en el frontend mediante archivo master-data.ts.';

-- Columnas de contacto
COMMENT ON COLUMN public.contacts.id IS 'UUID primario, auto-generado.';
COMMENT ON COLUMN public.contacts.organization_id IS 'Clave foránea a organizations. Aislamiento multi-tenant: cada organización ve solo sus contactos.';
COMMENT ON COLUMN public.contacts.type_contact IS 'Tipo: ''company'' o ''person''. Validado en frontend. Sin CHECK en BD.';
COMMENT ON COLUMN public.contacts.tax_id IS 'Identificador fiscal: CIF/NIF para empresas, DNI/NIE para personas. Único por organización (índice único).';
COMMENT ON COLUMN public.contacts.external_id IS 'ID de integración con sistemas externos (ERP, CRM). Único por organización si se proporciona.';
COMMENT ON COLUMN public.contacts.salutation IS 'Tratamiento protocolario: sr, sra, dr, dra, excmo, ilustre, sc. Validado en frontend.';
COMMENT ON COLUMN public.contacts.first_name IS 'Nombre (solo para personas físicas). Para empresas, usar contact_name o trade_name.';
COMMENT ON COLUMN public.contacts.last_name IS 'Apellidos (solo para personas físicas).';
COMMENT ON COLUMN public.contacts.full_name IS 'Nombre completo para visualización y búsqueda. Para personas: concatenación first_name + last_name. Para empresas: razón social o contact_name.';
COMMENT ON COLUMN public.contacts.trade_name IS 'Nombre comercial de la marca (principalmente para empresas).';
COMMENT ON COLUMN public.contacts.email IS 'Email oficial de notificaciones. Para empresas, email corporativo genérico.';
COMMENT ON COLUMN public.contacts.website IS 'URL de la página web corporativa o personal.';
COMMENT ON COLUMN public.contacts.phone_1 IS 'Teléfono primario de contacto.';
COMMENT ON COLUMN public.contacts.phone_2 IS 'Teléfono secundario (opcional).';
COMMENT ON COLUMN public.contacts.phone_whatsapp IS 'Teléfono para alertas y mensajería (WhatsApp).';
COMMENT ON COLUMN public.contacts.birth_date IS 'Fecha de nacimiento (personas) o fecha de constitución (empresas).';
COMMENT ON COLUMN public.contacts.gender IS 'Género: male, female, other. Solo para personas. Validado en frontend.';
COMMENT ON COLUMN public.contacts.profile_image IS 'URL o base64 de la imagen de perfil o logo.';
COMMENT ON COLUMN public.contacts.preferred_language IS 'Idioma preferente para comunicaciones (ISO 639-1).';
COMMENT ON COLUMN public.contacts.timezone IS 'Zona horaria según IANA (ej. Europe/Madrid).';
COMMENT ON COLUMN public.contacts.currency IS 'Divisa preferente (ISO 4217).';
COMMENT ON COLUMN public.contacts.contact_name IS 'Nombre del contacto principal (para empresas) o contacto directo (para personas).';
COMMENT ON COLUMN public.contacts.position IS 'Cargo o puesto del contacto principal.';
COMMENT ON COLUMN public.contacts.email_contact IS 'Email de contacto directo (si difiere del email oficial).';
COMMENT ON COLUMN public.contacts.phone_contact IS 'Teléfono de contacto directo (si difiere del teléfono primario).';
COMMENT ON COLUMN public.contacts.phone_contact_whatsapp IS 'Teléfono de contacto para WhatsApp (si difiere del WhatsApp principal).';
COMMENT ON COLUMN public.contacts.created_at IS 'Timestamp de creación (UTC).';
COMMENT ON COLUMN public.contacts.updated_at IS 'Timestamp de última modificación (UTC).';

-- Columnas JSONB auxiliares
COMMENT ON COLUMN public.contacts.address_json IS 'Dirección estructurada. Esquema sugerido: {siglas, address, extension, postal_code, city, province, country, region}. Validado en frontend.';
COMMENT ON COLUMN public.contacts.professional_data_json IS 'Datos profesionales: sector, categoría, grupo, ocupación, departamento, company_link (UUID), nivel de influencia. Ver esquema detallado en comentarios del script.';
COMMENT ON COLUMN public.contacts.compliance_json IS 'Datos de cumplimiento: KYC, RGPD (base legal, consentimiento, retirada, evidencias), PEP, listas de sanciones. Estructura sensible a auditorías.';
COMMENT ON COLUMN public.contacts.social_links IS 'Redes sociales y perfiles. Objeto con clave = red social, valor = {url, username, custom-label, ...}. Sin restricciones de claves.';
COMMENT ON COLUMN public.contacts.custom_fields_json IS 'Campos personalizables por el usuario o la organización. Estructura libre, recomendado seguir patrón {valor, label, type, options, order, group, is_required, is_visible}.';

-- Tabla sidecar
COMMENT ON TABLE public.contact_user_private_data IS 'Datos privados por usuario (sidecar). Aislamiento absoluto: RLS filtra por user_id. Ningún otro usuario de la organización puede acceder.';
COMMENT ON COLUMN public.contact_user_private_data.id IS 'UUID primario.';
COMMENT ON COLUMN public.contact_user_private_data.contact_id IS 'Referencia al contacto padre (ON DELETE CASCADE).';
COMMENT ON COLUMN public.contact_user_private_data.user_id IS 'Propietario exclusivo del registro (auth.users).';
COMMENT ON COLUMN public.contact_user_private_data.private_notes_html IS 'Notas HTML de uso personal (solo visibles para el usuario propietario).';
COMMENT ON COLUMN public.contact_user_private_data.vault_credentials_json IS 'Credenciales encriptadas para accesos externos. Debes encriptar los secretos en cliente antes de guardar.';
COMMENT ON COLUMN public.contact_user_private_data.created_at IS 'Timestamp de creación (UTC).';
COMMENT ON COLUMN public.contact_user_private_data.updated_at IS 'Timestamp de última modificación (UTC).';
```
