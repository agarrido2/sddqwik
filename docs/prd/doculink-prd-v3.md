# DOCULINK — PRD v3.0

**Fecha de aprobación del cliente:** 2026-05-06  
**Versión:** 3.0  
**Aprobado por cliente (Agarrido2):** ✅

---

## 0. ESTADO DE IMPLEMENTACIÓN ACTUAL

| Módulo | Capa | Estado | Notas |
|--------|------|--------|-------|
| Autenticación (login/logout) | Auth | ✅ Done | Supabase Auth. Guard anti-open-redirect (PRD v2.0). |
| Onboarding (org + owner) | Auth | ✅ Done | Transacción atómica (PRD v2.0). |
| Gestión de usuarios y roles | Users | ✅ Done | Invitaciones por email con token SHA-256. Roles: Owner, Admin, Member (PRD v2.0). |
| App Shell (sidebar, layout) | UI | ✅ Done | Colapsable desktop, drawer mobile (PRD v2.0). |
| Dashboard home | UI | ✅ Done | KPIs desde OrgContext (PRD v2.0). |
| CRM — Contactos | CRM | ✅ Done | CRUD personas físicas y jurídicas. GDPR (PRD v2.0). |
| Expedientes — CRUD básico | Cases | ✅ Done | Case number, lead_member, category, status (PRD v2.0). |
| Expedientes — Wizard + Bitácora | Cases | ✅ Done | Mosaico, Wizard 3 pasos, case_items (PRD v2.0). |
| Contactos del expediente | Cases | ✅ Done | Tabla pivote case_contacts con rol (PRD v2.0). |
| Estados de ítems de bitácora | Cases | ✅ Done | Enum + badge + filtro (PRD v2.0). |
| --- | --- | --- | --- |
| Estados del expediente (4 estados: archieved) | Cases | 🟡 Pendiente | Definido en este PRD. Refactorizar states. |
| Expediente — Contacto principal y secundarios | Cases | 🔲 Pendiente | Confirmar al menos 1 contacto. Definido en este PRD. |
| Hub Teams (organización, jerarquía, membresía) | Teams | 🔲 Pendiente | Definido en este PRD. |
| Templates de acciones de bitácora | Cases | 🔲 Pendiente | Definido en este PRD. Solo Admin/Owner crean, todos visualizan. |
| Gestión Documental — Dos tipos (probatorio + formal) | Docs | 🔲 Pendiente | Definido en este PRD. Módulo independiente. Firmas internas con hash. |
| Notificaciones (campanita + chat bidireccional actionable) | Notifs | 🔲 Pendiente | Definido en este PRD. Solo in-app. |
| Cierre formal de expediente | Cases | 🔲 Pendiente | Definido en este PRD. Bloqueo si hay items no terminales. |
| IA — Resumen de expediente + futuro metadatos docs | AI | 🔲 Pendiente | Fase 1: resumen de expediente. Fase 2: metadatos docs (pendiente). |
| Reporte de cierre (PDF) | Cases | 🔲 Pendiente | Definido en este PRD. |
| --- | --- | --- | --- |
| Proceso de Facturación | Billing | 🔮 Fuera de alcance | Pendiente de incorporar en versión futura. |
| API REST | API | 🔮 Fuera de alcance | Pendiente de incorporar en versión futura. |
| Otras funcionalidades anexas | — | 🔮 Fuera de alcance | Pendiente de incorporar en versión futura. |

---

## 1. ESTADO GLOBAL

**🟢 Green — El PRD está aprobado y listo para generar Blueprint y Specs.**

---

## 2. VISIÓN DEL PRODUCTO

Doculink es una aplicación web multi-tenant que gestiona expedientes para contactos (personas físicas o jurídicas). El expediente es el centro gravitacional de información y actividades de un caso específico, donde se centralizan todas las interacciones, documentos, comunicaciones y tareas necesarias para resolverlo. El producto está diseñado para ser intuitivo, flexible y trazable, adaptándose a cualquier sector que requiera gestión organizada de casos (despachos de abogados, inmobiliarias, departamentos de cobros, equipos de soporte, etc.).

---

## 3. MÓDULOS PRINCIPALES Y FUNCIONALIDADES

### 3.1 Módulo de Autenticación y Multi-Tenant

**Alcance:** Login, logout, recuperación de contraseña, aislamiento por organización (tenant).
- Supabase Auth.
- Cada organización es un tenant completamente aislado.
- A partir del login, el contexto de organización determina los datos a los que el usuario accede.
- El usuario puede pertenecer a múltiples organizaciones (en versiones futuras).

### 3.2 Módulo de Onboarding y Organización

**Alcance:** Creación de organización y asignación de owner.
- El primer usuario que crea la organización es el Owner (rol máximo, puede hacer todo).
- Transacción atómica: creación de org + perfil owner + configuración básica.

### 3.3 Módulo de Usuarios y Roles

**Alcance:** Gestión de usuarios dentro de una organización con roles diferenciados.
- **Owner:** Acceso total a todas las funcionalidades. Puede crear/asignar expedientes, gestionar usuarios, configurar templates, acceder a toda la información.
- **Admin:** Acceso amplio. Puede crear/asignar expedientes, gestionar hub teams, crear templates, gestionar usuarios. No puede transferir ownership de la organización.
- **Member:** Rol operativo. Puede consultar expedientes en los que participa, registrar eventos en la bitácora, gestionar documentos asociados a sus expedientes, responder a notificaciones. No puede crear expedientes ni hub teams ni templates.
- Invitaciones por email con token seguro.
- Los roles no son jerárquicos (no hay cadena de mando), son permisos de acceso.
- La jerarquía dentro de un Hub Team es independiente del rol del usuario.

### 3.4 Módulo de CRM — Contactos

**Alcance:** CRUD de contactos (personas físicas y jurídicas) con cumplimiento GDPR.
- Tipos: persona física, persona jurídica.
- Campos: nombre, datos de contacto, historial de interacciones.
- Un contacto puede estar asociado a múltiples expedientes.
- Búsqueda y filtrado por tipo, nombre, tags.

### 3.5 Módulo de Expedientes

**Alcance:** Creación, consulta, edición y cierre de expedientes. Centro gravitacional de la aplicación.

**Estados del expediente:**
- `open` — Expediente creado, sin ítems en la bitácora.
- `in_process` — Expediente con ítems en la bitácora (al menos uno), en activo.
- `closed` — Expediente cerrado formalmente, con reporte de cierre generado. No se pueden añadir más ítems.
- `archived` — Expediente archivado para consulta histórica. Solo lectura.

**Contactos del expediente:**
- **Contacto principal:** Obligatorio. Siempre 1. Es quien encarga el trabajo y presumiblemente quien paga.
- **Contactos secundarios:** Opcionales. Pueden ser 0, 1, 2, 3 o más. Son terceros involucrados en el caso.
- La relación se gestiona mediante tabla pivote `case_contacts` con columna `role` (principal/secundario).
- El contacto principal es obligatorio al crear el expediente.

**Colaboradores del expediente:**
- Hub Teams y usuarios individuales invitados al expediente.
- Solo los colaboradores incorporados pueden:
  - Ver los eventos de la bitácora.
  - Ser destinatarios de notificaciones del expediente.
  - Acceder a los documentos asociados.
  - Registrar eventos en la bitácora.
- El círculo de participantes es cerrado: solo usuarios y Hub Teams del expediente.

### 3.6 Módulo de Bitácora de Eventos

**Alcance:** Registro cronológico de todas las acciones realizadas en un expediente.

**Estructura de un ítem (evento):**
- Descripción de la acción.
- Tipo de evento (hito, tarea, comunicación, decisión, etc.).
- Prioridad (1=urgent, 2=normal, 3=baja).
- Estado (pending, in_progress, done, cancelled).
- Menciones (usuarios / Hub Teams a notificar).
- Anexos: documentos probatorios adjuntos (solo tipo probatorio/t.
- Timestamp automático.

**Reglas de negocio:**
- Cerrar expediente bloquea nuevos ítems: solo se permiten si el estado es `done` o `cancelled`.
- El estado `open` persiste hasta que se registre el primer ítem, momento en que pasa a `in_process`.
- La bitácora es el núcleo activo de la gestión del expediente.

### 3.7 Módulo de Templates de Acciones de Bitácora

**Alcance:** Plantillas predefinidas de ítems reutilizables para facilitar la creación de expedientes.

**Creación:** Solo usuarios con rol Admin o Owner pueden crear templates.

**Uso:**
- Todos los usuarios pueden visualizar y aplicar templates al crear un expediente.
- El template se aplica en el paso de creación del expediente (Wizard).
- Los ítems del template se copian como `pending` a la bitácora del nuevo expediente.
- Una vez aplicados, los ítems son independientes: se pueden modificar, eliminar o reordenar sin afectar al template original.
- El template es una guía, no una obligación. El usuario puede crear ítems manualmente o dejar la bitácora vacía.
- Un template contiene: nombre, descripción, lista de ítems predefinidos (contenido, tipo, prioridad, orden).

### 3.8 Módulo de Hub Teams

**Alcance:** Equipos jerárquicos especializados que participan en expedientes.

**Pertenencia:** Los Hub Teams pertenecen a una organización específica. No existen Hub Teams globales entre organizaciones (multi-tenant).

**Estructura:**
- Un responsable (debe tener rol Admin o Owner en la organización).
- Miembros: usuarios de la organización.
- Jerarquía: definida por el orden en que aparecen los miembros (posición 1 = mayor jerarquía). El responsable establece el orden.
- Estado operativo: cada miembro tiene un flag `active` (true/false) que indica si está operativo en el Hub Team.

**Ciclo de firma en Hub Teams:**
- Los miembros participan en un ciclo secuencial según su jerarquía (orden establecido).
- El ciclo es un snapshot al momento de incorporar el Hub Team al expediente.
- Si el Hub Team es mencionado en un ítem de bitácora, la notificación sigue la jerarquía establecida.

**Gestión:** Solo Owner y Admin pueden crear Hub Teams dentro de la organización.

### 3.9 Módulo de Gestión Documental (Dos Tipos de Documentos)

**Alcance:** Gestión y firma de documentos dentro de un expediente, con dos flujos distintos según el tipo de documento.

#### Tipo A: Documentos Probatorios / Testimoniales
- **Qué son:** Email, PDF de pago, captura de pantalla, cualquier documento que sirve para contrastar o evidenciar algo.
- **Flujo:** El usuario, al registrar un evento en la bitácora, pulsa el botón de anexar documento y adjunta el archivo.
- **Almacenamiento:** Se guardan en Supabase Storage como archivos normales, asociados al ítem de la bitácora (`case_item_id`).
- **Acceso:** Solo colaboradores del expediente.
- **No tienen:** versioning, firma, metadata automática, proceso especial.

#### Tipo B: Documentos Formales / De Gestión Documental
- **Qué son:** Facturas, contratos, vales de compra, hojas de liquidación, acuerdos, autorizaciones. Documentos que requieren tratamiento especial.
- **Flujo:**
  1. El usuario, desde el expediente, pulsa el botón de "Gestión Documental".
  2. Se abre el módulo independiente de gestión de documentos.
  3. El usuario sube el documento formal.
  4. El usuario decide si el documento requiere firma (sí/no).
  5. Si requiere firma: se define el flujo de firmantes (uno o más usuarios/Hub Teams del expediente).
  6. El PDF original se guarda en Supabase Storage, asociado al expediente.
  7. Se recorre el flujo de firmas secuencialmente: cada firmante firma el documento (internamente en la app, sin valor legal formal).
  8. Cada firma genera un hash único de seguridad que valida quién firmó y cuándo.
  9. Al finalizar todas las firmas, el documento queda firmado y validado digitalmente por todos los integrantes.
  10. Flujo simple: documento → firmado → listo. Sin versioning en esta fase.

**Metadata con IA (Fase futura):**
- De momento, se deja el hueco para futura extracción automática: resumen del documento, partes intervinientes, importes, cifras, fechas clave.
- Esta funcionalidad no está en el alcance de la v3.0.

### 3.10 Módulo de Notificaciones

**Alcance:** Sistema de comunicación in-app entre colaboradores del expediente.

**Interfaz:**
- Campanita en la barra superior de la aplicación.
- Las notificaciones no leídas se indican con badge.

**Tipos de notificación:**
- `info` — Mensaje informativo unidireccional. No requiere respuesta.
- `actionable` — Mensaje que requiere respuesta. Genera un chat bidireccional entre los colaboradores.
- `success` — Confirmación de que una acción ha sido completada.

**Flujo bidireccional (solo actionable):**
- El usuario registra un evento y notifica a un colaborador con tipo `actionable`.
- El colaborador recibe la notificación y puede responder.
- Al responder, se crea un hilo/chat asociado a ese ítem de la bitácora.
- El usuario original recibe una notificación `success` con la respuesta.
- El historial de la conversación queda registrado.

**Alcance:** Solo in-app. Sin email, sin push externo en esta fase.

**Trigger de notificación:**
- Se genera al registrar un ítem en la bitácora y seleccionar a qué colaboradores notificar (usuario individual o Hub Team completo).

### 3.11 Módulo de Cierre de Expediente

**Alcance:** Cierre formal de expedientes completados.

**Reglas de cierre:**
- Solo se puede cerrar si todos los ítems de la bitácora tienen estado `done` o `cancelled`.
- Si hay ítems en estado `pending` o `in_progress`, el sistema bloquea el cierre y muestra alerta.
- Al cerrar, el estado pasa de `in_process` a `closed`.
- El expediente en estado `closed` entra en modo solo lectura: no se pueden añadir nuevos ítems ni documentos.

**Reporte de cierre:**
- Al cerrar, el sistema pregunta al usuario si quiere generar un reporte de cierre.
- El reporte incluye: resumen de expedientes ejecutado en la bitácora, documentos asociados, comunicaciones realizadas, participantes (contactos, hub teams, usuarios).
- Formato de salida: PDF.
- El reporte se puede enviar por email al contacto principal, contactos secundarios y colaboradores del expediente.

### 3.12 Módulo de IA

**Alcance:** Asistencia inteligente integrada en puntos concretos del flujo.

**Fase 1 (v3.0) — Resumen de expediente:**
- La IA genera automáticamente un resumen contextual del expediente analizando la bitácora de eventos.
- El resumen se presenta en el panel de visión global del expediente (resumen de estado, últimos eventos, documentos clave, próximos pasos sugeridos).
- El usuario puede regenerar el resumen en cualquier momento.

**Fase 2 (futura) — Metadatos de documentos:**
- Extracción automática de metadatos de documentos formales: resumen, partes intervinientes, importes, cifras, fechas clave.
- No incluida en esta versión. Se deja el hueco arquitectónico.

---

## 4. DECISIONES DE DOMINIO CLAVE

| Decisión | Valor | Justificación |
|----------|-------|---------------|
| Estados del expediente | `open` → `in_process` → `closed` → `archived` | Cubre todo el ciclo de vida. `open` es sin items, `in_process` es con items activos. `closed` es final, `archived` es histórico. |
| Contacto en expediente | 1 principal (obligatorio) + 0..N secundarios (opcionales) | El principal es quien encarga y paga. Los secundarios son terceros del caso. Tabla pivote `case_contacts` con columna `role`. |
| Colaboradores del expediente | Círculo cerrado: usuarios y Hub Teams invitados | Solo ellos ven bitácora, reciben notificaciones, acceden a documentos, registran eventos. Los contactos (CRM) no son colaboradores por defecto. |
| Documentos probatorios vs formales | Dos flujos distintos. Probatorios se anexan en bitácora. Formales tienen módulo propio con firma. | Separación clara: lo testimonial es rápido y simple. Lo formal requiere proceso de firma secuencial con hash de seguridad. |
| Firmas de documentos | Internas, sin valor legal formal. Secuencial por jerarquía. Hash único por firma. | Simplificación máxima para v3.0. El PDF original se guarda, se recorren firmas con validación por hash. Flujo: documento → firmado → listo. |
| Hub Teams | Jerarquía por orden de inserción. Snapshot al incorporar al expediente. Pertenencia por organización (multi-tenant). | La jerarquía define el ciclo de notificaciones. No hay Hub Teams globales.
| Templates de acciones | Admin/Owner crean, todos aplican. Se copian como `pending`. Independientes tras aplicación. | Son guía, no obligación. El usuario puede personalizar todo tras aplicar.
| Notificaciones | Solo in-app. Campanita. Tipos: info, actionable (bidireccional), success. | Simplificación. Sin email ni push externo en esta fase.
| IA | Solo resumen de expediente (v3.0). Hueco para metadatos docs (futura). | Alcance controlado, sin fantasías. |
| Multi-tenant | Cada organización aislada. Hub Teams, templates, usuarios, expedientes son scoped a la organización. | RLS de Supabase por `org_id` en todas las tablas. |
| Roles | Owner (todo), Admin (gestión amplia), Member (operativo). Owner es DIOS. | Simple, claro, sin ambigüedades.
| Cierre de expediente | Bloqueo si hay items no terminales. Reporte PDF opcional.
| Trazabilidad | Timestamps automáticos en todos los eventos. Hash de firma por documento.
| Integridad | No se borra un contacto con expedientes asociados. No se borra un expediente con items.

---

## 5. DECISIONES DE DOMINIO ABIERTAS / PENDIENTES

| Decisión | Estado | Notas |
|----------|--------|-------|
| Extracción de metadatos con IA para documentos formales | 🔮 Pendiente | Se deja hueco. Para v3.1 o v4.0. |
| Integración con sistemas de correo (auto-registro de eventos) | 🔮 Fuera de alcance | Visión futura. |
| Proceso de facturación | 🔮 Fuera de alcance | Definido en visión, fuera de v3.0. |
| API REST | 🔮 Fuera de alcance | Definido en visión, fuera de v3.0. |
| Otras funcionalidades anexas (calendarios, exportación, firmas autorizadas) | 🔮 Fuera de alcance | Definido en visión, fuera de v3.0. |

---

## 6. FUERA DEL ALCANCE (v3.0)

- Proceso de facturación.
- API REST.
- Firmas autorizadas avanzadas.
- Seguimiento de tareas con fechas límite.
- Reportes y análisis avanzados.
- Personalización de flujos de trabajo con reglas de negocio.
- Exportación/importación masiva de datos.
- Integración con calendarios (Google, Outlook).
- Predicción de riesgos con IA.
- Sugerencia automática de acciones con IA.
- Generación automática de eventos desde integraciones externas.

---

## 7. ESTADO ACTUAL

**🟢 Green — PRD aprobado.**

10 módulos base implementados y funcionales. 9 módulos nuevos definidos en este PRD. 0 bloqueos conocidos en modelo de dominio. Listo para generar Blueprint.
