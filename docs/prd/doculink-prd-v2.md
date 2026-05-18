# PRD — Documento de Requisitos del Producto: Doculink

**Proyecto:** Doculink — Gobernanza de Expedientes y Gestión de Procesos  
**Versión:** 2.0  
**Fecha:** 2026-04-25  
**Estado:** 🟢 Aprobado  
**Autor:** Antonio Garrido

---

## 0. Estado de Implementación Actual

> Esta sección es de uso exclusivo del sistema de desarrollo. Indica qué módulos están construidos y cuáles son el objetivo de este PRD.

| Módulo | Estado | Notas |
|---|---|---|
| Autenticación (login/logout) | ✅ Done | Supabase Auth. Guard anti-open-redirect. |
| Onboarding (org + owner) | ✅ Done | Transacción atómica. |
| Gestión de usuarios y roles | ✅ Done | Invitaciones por email con token SHA-256. |
| App Shell (sidebar, layout) | ✅ Done | Colapsable desktop, drawer mobile. |
| Dashboard home | ✅ Done | KPIs desde OrgContext. |
| CRM — Contactos | ✅ Done | CRUD personas físicas y jurídicas. GDPR. |
| Expedientes v1 (CRUD básico) | ✅ Done | case_number, lead_member, category, status. |
| Expedientes v2 (Bitácora + Wizard) | ✅ Done | Mosaico, Wizard 3 pasos, case_items. |
| Contactos del expediente | ✅ Done | Tabla pivote case_contacts con rol. |
| Estado de ítems de bitácora | ✅ Done | Enum case_item_status + badge + filtro. |
| Hub Teams | 🔲 Pendiente | Definido en este PRD. |
| Templates de bitácora | 🔲 Pendiente | Definido en este PRD. |
| Gestión documental | 🔲 Pendiente | Definido en este PRD. |
| Notificaciones in-app | 🔲 Pendiente | Definido en este PRD. |
| Cierre formal de expediente | 🔲 Pendiente | Definido en este PRD. |
| Firmas, IA, Facturación, API REST | 🔮 Visión futura | Documentados en Sección 11. |

---

## 1. Resumen Ejecutivo

Doculink es una plataforma SaaS de gestión de expedientes diseñada para centralizar el ciclo de vida completo de cualquier proceso que involucre a un contacto y requiera trazabilidad, coordinación de equipos y gestión documental. Su principio fundacional es el **Expediente como centro de gravedad**: todo ocurre alrededor del expediente — participantes internos, contactos vinculados, bitácora de eventos, documentos y notificaciones.

A diferencia de los gestores de tareas genéricos, Doculink está diseñado para procesos irregulares y narrativos donde las acciones no siempre siguen un orden predefinido, donde intervienen múltiples partes y donde la trazabilidad completa es un requisito operativo, no una opción. La plataforma es **agnóstica de sector**: sirve con igual eficacia a un despacho de abogados, un departamento de cobros, un equipo comercial o una gestoría administrativa.

---

## 2. Usuarios del Sistema

### 2.1 Roles

El sistema tiene tres roles permanentes. No existe un rol "Invited" — la incorporación a un Hub Team es un estado del proceso de integración, no un rol de la aplicación.

| Rol | Descripción | Capacidades principales |
|---|---|---|
| **Owner** | Propietario de la organización | Acceso total. Gestión de facturación, planes, usuarios y datos globales. Puede crear Hub Teams. |
| **Admin** | Administrador operativo | Gestión de usuarios, creación y mantenimiento de Templates y Hub Teams, auditoría de procesos. |
| **Member** | Usuario operativo | Creación y gestión de expedientes, registro de ítems en bitácora, consulta de contactos. No puede crear Hub Teams ni Templates. |

### 2.2 Gestión de Usuarios

Owner y Admin pueden invitar nuevos usuarios a la organización mediante email. El flujo de invitación genera un token de un solo uso (UUID v4) que el usuario utiliza para activar su cuenta. Una vez activo, el usuario es Member por defecto.

Owner y Admin pueden elevar o reducir el rol de cualquier Member. Solo el Owner puede gestionar a otros Admins.

---

## 3. Directorio de Contactos (CRM)

Los contactos son las entidades externas para las cuales se crean y gestionan los expedientes. Son el sujeto central de cada expediente.

### 3.1 Naturaleza

- **Persona física:** Particulares, clientes individuales.
- **Persona jurídica:** Empresas, ONGs, entes públicos, despachos.

### 3.2 Atributos

- Nombre completo o razón social.
- Identificación fiscal (NIF / CIF).
- Datos de contacto: email, teléfono, dirección.
- Representantes legales (para personas jurídicas).
- Preferencias de comunicación.
- Registro de consentimientos GDPR para comunicaciones legales y notificaciones.

### 3.3 Integridad Referencial

Un contacto no puede eliminarse si tiene expedientes vinculados. El sistema aplica bloqueo de borrado con mensaje explicativo. El usuario debe cerrar o reasignar los expedientes antes de poder eliminar el contacto.

### 3.4 Historial

Cada contacto expone un historial de todos los expedientes en los que ha participado, ya sea como contacto principal o como contacto secundario vinculado.

### 3.5 Contactos Vinculados a Expedientes

Un expediente tiene un **contacto principal** obligatorio y puede tener **contactos secundarios** opcionales. La relación entre expediente y contactos secundarios se gestiona mediante una tabla pivote (`case_contacts`) que registra el rol del contacto en el expediente (principal, secundario, contraparte, etc.).

> **Importante:** Los contactos son entidades externas del CRM. No son usuarios de la aplicación y no pueden ser mencionados en ítems de bitácora ni recibir notificaciones internas del sistema. Su comunicación con el expediente se gestiona mediante el módulo de comunicaciones externas (visión futura).

---

## 4. Hub Teams

### 4.1 Definición

Un Hub Team es un equipo de trabajo especializado en un área funcional concreta (cobros, soporte, ventas, legal, administración, etc.). Es una entidad organizativa persistente que agrupa usuarios bajo una jerarquía definida y puede participar en múltiples expedientes simultáneamente.

### 4.2 Atributos

| Campo | Descripción |
|---|---|
| `name` | Nombre del Hub Team (ej: "Gestión Cobros", "Soporte Técnico"). |
| `description` | Descripción del área de especialización. |
| `supervisor_id` | Usuario (Owner o Admin) responsable del equipo. |
| `status` | `active` / `inactive`. |

### 4.3 Quién puede crear un Hub Team

Solo **Owner** y **Admin** pueden crear, editar y eliminar Hub Teams. Un Member no puede crear ni gestionar Hub Teams.

### 4.4 Composición y Jerarquía

- **Supervisor:** El Owner o Admin que crea el Hub Team se convierte en su supervisor. Es responsable de definir sus integrantes, establecer su jerarquía y gestionar su activación.
- **Integrantes:** Cualquier usuario de la organización (Member, Admin u Owner) puede ser incorporado como integrante. El orden en que aparecen en la tabla de miembros define su jerarquía dentro del equipo.

La jerarquía de integrantes determina:
- El orden de supervisión dentro del equipo.
- La precedencia en los procesos de firma documental (módulo futuro).

El supervisor puede reordenar integrantes y activar/desactivar su participación en cualquier momento.

### 4.5 Flujo de Incorporación de Integrantes

Incorporar a un usuario a un Hub Team requiere su aceptación explícita:

1. El supervisor incorpora al usuario al Hub Team desde el panel de gestión.
2. El usuario recibe una notificación in-app de invitación al Hub Team.
3. El usuario acepta o rechaza.
4. Si acepta → su estado en el Hub Team pasa a `active` y queda operativo.
5. Si rechaza → la invitación se cancela. El supervisor puede volver a invitarle.

Hasta que el usuario acepte, su estado es `pending` y no tiene acceso ni visibilidad operativa dentro del Hub Team.

### 4.6 Gestión en la Aplicación

La gestión de Hub Teams se realiza desde la sección **Settings → Hub Teams** del panel de administración. Esta sección es visible y accesible únicamente para Owner y Admin.

### 4.7 Participación en Expedientes

Un Hub Team puede ser incorporado a un expediente por el usuario que lo crea o gestiona. Al incorporar un Hub Team a un expediente:
- Todos sus integrantes con estado `active` pasan a formar parte del **círculo de participantes internos** del expediente.
- Pueden ser mencionados individualmente (por usuario) o colectivamente (por Hub Team) en los ítems de la bitácora.
- Reciben notificaciones cuando el Hub Team es mencionado en un ítem.

Un expediente puede tener ninguno, uno o varios Hub Teams incorporados.

---

## 5. El Expediente: Centro de Gravedad del Sistema

### 5.1 Definición

El expediente es el contenedor principal de toda la actividad relacionada con un caso específico para un contacto. Representa un proceso con inicio, desarrollo narrativo y cierre, que puede involucrar a múltiples participantes internos (usuarios, Hub Teams) y contactos externos.

### 5.2 Identidad y Cabecera

| Campo | Descripción | Obligatorio |
|---|---|---|
| `case_number` | Identificador humano único por organización (ej: EXP-2026-001). Generado automáticamente. | Sí |
| `title` | Título descriptivo del expediente. | Sí |
| `description` | Briefing inicial. Relato del contexto del caso. | No |
| `category` | Clasificación temática (Jurídico, Fiscal, Cobros, Ventas, Soporte, etc.). | No |
| `status` | Estado del ciclo de vida: `open`, `in_progress`, `closed`. | Sí |
| `contact_id` | Contacto principal del expediente (entidad del CRM). | Sí |
| `lead_member_id` | Responsable principal interno del expediente. | No |
| `template_id` | Template aplicado en la creación. Solo se puede aplicar una vez, en la creación. | No |

### 5.3 Participantes Internos del Expediente

Los participantes internos son los usuarios de la organización que tienen acceso y visibilidad sobre el expediente. Al crear o editar un expediente, el usuario puede incorporar:

- **Usuarios individuales** (cualquier Member, Admin u Owner de la organización).
- **Hub Teams** (todos sus integrantes activos quedan incorporados como participantes).

El conjunto de todos estos participantes forma el **círculo de participantes internos** del expediente. Este círculo es la única fuente válida de menciones en los ítems de la bitácora.

> **Regla fundamental:** Solo pueden ser mencionados en los ítems de un expediente los participantes de su círculo interno. No es posible mencionar a un usuario ajeno al expediente aunque exista en la organización.

### 5.4 Contactos Vinculados al Expediente

Separados del círculo de participantes internos, el expediente puede tener:
- Un **contacto principal** del CRM (obligatorio).
- Uno o varios **contactos secundarios** del CRM (opcionales), gestionados mediante la tabla pivote `case_contacts`.

Los contactos son entidades externas. No reciben notificaciones internas ni pueden ser mencionados en ítems de bitácora.

### 5.5 Template en la Creación

Durante la creación del expediente, el usuario puede seleccionar un Template. Al hacerlo:
- El sistema pre-carga automáticamente en la bitácora los ítems definidos en el Template.
- Una vez aplicado el Template, no puede aplicarse otro Template al mismo expediente.
- Los ítems pre-cargados son **sugerencias editables**, no tareas obligatorias ni ordenadas.

El usuario puede libremente:
- Ignorar cualquier ítem pre-cargado.
- Reordenarlos, modificarlos o eliminarlos.
- Añadir ítems adicionales en cualquier momento durante la vida del expediente.

### 5.6 Ciclo de Vida

```
open → in_progress → closed
```

- **`open`:** Expediente creado, sin actividad iniciada.
- **`in_progress`:** Expediente con actividad en curso.
- **`closed`:** Expediente cerrado formalmente. Bloqueado para modificaciones.

Las transiciones entre estados son manuales, realizadas por el usuario responsable del expediente.

### 5.7 Cierre del Expediente

El cierre es un proceso con validación bloqueante:

1. **Validación previa obligatoria:** El sistema verifica que todos los ítems de la bitácora tengan un estado terminal (`done` o `cancelled`). Si existe algún ítem en estado `pending` o `in_progress`, el cierre queda **bloqueado**. El sistema muestra qué ítems impiden el cierre.
2. **Registro de cierre:** El usuario introduce un `closure_summary` (resumen de resolución). Campo obligatorio.
3. **Sellado:** El sistema registra el `closed_by` (quién cierra) y el `closed_at` (timestamp UTC). El expediente queda sellado: no se pueden añadir ni modificar ítems ni cambiar ningún dato del expediente.

---

## 6. Bitácora de Eventos (Case Items)

### 6.1 Definición

La bitácora es el registro cronológico y narrativo de todas las acciones, interacciones y hitos que ocurren dentro de un expediente. No es un checklist estático — es una línea de tiempo flexible donde el usuario registra lo que ocurre, cuando ocurre, con el nivel de detalle que considera necesario.

La bitácora es la memoria viva del expediente. Cada ítem puede representar una acción realizada, una tarea pendiente, un hito alcanzado, una comunicación registrada o cualquier evento relevante para el caso.

### 6.2 Atributos de un Ítem

| Campo | Tipo | Descripción |
|---|---|---|
| `content` | text | Relato detallado de la acción o evento. |
| `type` | enum | `note` (Nota), `milestone` (Hito), `document` (Documento). |
| `grade` | int | Importancia: `1` Normal, `2` Alta, `3` Crítica. |
| `status` | enum | `pending`, `in_progress`, `done`, `cancelled`. |
| `occurrence_date` | date | Fecha en que ocurrió el evento. Puede ser retroactiva. Independiente de `created_at`. |
| `created_at` | timestamp | Fecha de registro en el sistema. UTC. |

### 6.3 Menciones a Participantes

Al crear o editar un ítem, el usuario puede mencionar a cualquier participante del círculo interno del expediente. Las menciones se almacenan mediante una **tabla pivote de menciones** (`case_item_mentions`), no como campos de texto plano en el ítem.

Se puede mencionar a:
- **Un usuario individual** del círculo de participantes.
- **Un Hub Team completo** incorporado al expediente (notifica a todos sus integrantes activos).

Una mención genera una notificación para el usuario mencionado o para todos los integrantes activos del Hub Team mencionado.

**Regla:** Solo se pueden mencionar participantes del círculo interno del expediente. No es posible mencionar a usuarios externos al expediente.

### 6.4 Flexibilidad

Los ítems no tienen orden obligatorio ni dependencias entre sí. El usuario puede:
- Añadir ítems en cualquier momento durante la vida del expediente.
- Cambiar el estado de cualquier ítem independientemente del estado de los demás.
- Registrar eventos pasados mediante `occurrence_date` retroactiva.
- Marcar como `cancelled` cualquier ítem que ya no sea relevante.

### 6.5 Relación con el Cierre

El estado de los ítems es determinante para el cierre del expediente. Un expediente no puede cerrarse si algún ítem tiene estado `pending` o `in_progress`. El usuario debe resolver o cancelar todos los ítems abiertos antes de poder cerrar el expediente.

---

## 7. Templates de Bitácora

### 7.1 Definición

Los Templates son plantillas de ítems predefinidos, reutilizables y gestionados por la organización, diseñadas para agilizar la creación de expedientes recurrentes donde el flujo de trabajo es conocido de antemano.

### 7.2 Gestión

- Solo **Owner** y **Admin** pueden crear, editar y eliminar Templates.
- Los Members pueden seleccionar y aplicar Templates al crear un expediente.
- Los Templates son **globales para toda la organización**. No son privados de un Hub Team ni de un usuario.

### 7.3 Atributos del Template

| Campo | Descripción |
|---|---|
| `name` | Nombre identificativo del Template (ej: "MAPFRE - Taller X"). |
| `description` | Descripción del caso de uso para el que está diseñado. |
| `category` | Categoría sugerida para el expediente. |

### 7.4 Ítems del Template

Cada Template contiene una lista ordenada de ítems sugeridos. Cada ítem del Template tiene:

| Campo | Descripción |
|---|---|
| `content` | Texto sugerido para el ítem. |
| `type` | Tipología sugerida: `note`, `milestone`, `document`. |
| `grade` | Importancia sugerida: 1, 2 o 3. |
| `order_index` | Posición en la lista del Template (para la pre-carga ordenada). |

### 7.5 Comportamiento al Aplicar

- Los ítems del Template se **copian** como ítems reales en la bitácora del expediente.
- Son independientes del Template original desde el momento de la copia — modificar el Template no afecta a ítems ya creados.
- Se aplican con `status: pending` por defecto.
- Solo se puede aplicar un Template por expediente y únicamente en el momento de su creación. No es posible aplicar un segundo Template ni aplicar uno a un expediente ya creado.

---

## 8. Gestión Documental

### 8.1 Definición

El gestor de documentos es el núcleo de almacenamiento de los archivos asociados a cada expediente. Todo documento tiene un expediente como destino obligatorio y puede vincularse opcionalmente a un ítem específico de la bitácora.

El gestor de documentos **no es un sistema de firmas**. Es el repositorio central de documentación del expediente. La gestión de firmas autorizadas es una funcionalidad específica que opera sobre documentos ya almacenados (módulo futuro).

### 8.2 Capacidades

- Subida y almacenamiento de documentos (PDF, imágenes, hojas de cálculo, etc.).
- Asociación de documentos a ítems específicos de la bitácora.
- Tipología de documento: DNI, Contrato, Factura, Recibo, Evidencia, Comunicación, etc.
- Versioning: múltiples versiones del mismo documento.
- Control de acceso por rol.
- URLs firmadas de acceso temporal para compartir documentos de forma segura.
- Validación de tipo de fichero en servidor.

### 8.3 Relación con la Bitácora

Un documento puede adjuntarse a un ítem de la bitácora como evidencia de la acción registrada. Esta vinculación es opcional — los documentos pueden existir en el expediente sin estar vinculados a ningún ítem concreto.

---

## 9. Sistema de Notificaciones

### 9.1 Tipos de Notificación

| Evento | Destinatario |
|---|---|
| Mención de usuario en ítem | Usuario mencionado. |
| Mención de Hub Team en ítem | Todos los integrantes activos del Hub Team. |
| Incorporación al círculo de participantes del expediente | Usuario o integrantes del Hub Team incorporado. |
| Invitación a un Hub Team | Usuario invitado. |
| Cambio de estado del expediente | Lead Member y participantes del expediente. |
| Nuevo documento asociado al expediente | Lead Member. |
| Cierre del expediente | Todos los participantes internos del expediente. |

### 9.2 Canales

- **Notificación in-app (Bell Icon):** Alerta en tiempo real dentro de la aplicación. Siempre activa.
- **Email transaccional:** Para eventos de alta relevancia: invitaciones, cierre de expediente, menciones directas. Configurable por usuario.

### 9.3 Preferencias

Los usuarios pueden configurar qué tipos de notificación desean recibir por email. Las notificaciones in-app no son desactivables.

---

## 10. Flujo de Uso de Referencia

> Este escenario es el caso canónico de uso de Doculink. El worker debe tenerlo como referencia para validar que cualquier decisión de diseño o implementación es coherente con él.

**Actores:** Usuario Garrido (Member), Hub Team "Gestión Cobro" (3 integrantes: usr-a, usr-b, usr-c), Usuario "Miguel Cabrera" (Member), Contacto principal "TALLER X", Contacto externo "MAPFRE SEGUROS".

**Escenario:** Seguimiento de cobro de factura TS001 emitida por TALLER X a MAPFRE SEGUROS por importe de 100€.

**Flujo:**

1. Garrido crea el expediente "Cobro factura TS001 - MAPFRE" con contacto principal "TALLER X".
2. Incorpora al expediente: Hub Team "Gestión Cobro" y usuario "Miguel Cabrera". Círculo de participantes internos queda formado por: Garrido, usr-a, usr-b, usr-c, Miguel Cabrera.
3. Selecciona el Template "MAPFRE - Taller X". Se pre-cargan 3 ítems: "Enviar factura por email", "Factura aceptada y lista para el pago", "Factura liquidada y pagada".
4. Garrido registra ítem ad hoc: "MAPFRE indica que el número de peritación es incorrecto". Menciona a "Miguel Cabrera". Miguel recibe notificación.
5. Miguel Cabrera responde (por el canal que corresponda externamente). Garrido registra ítem: "Número de peritación correcto: 002001. Importe: 100€". Sin menciones.
6. Garrido registra ítem: "Enviado email a MAPFRE con número de peritación corregido".
7. Días después, Garrido registra ítem: "Pago recibido por transferencia el xx/xx/xx". Menciona al Hub Team "Gestión Cobro". Todos sus integrantes reciben notificación.
8. Garrido marca todos los ítems como `done`. Con todos los ítems en estado terminal, puede proceder al cierre.
9. Garrido cierra el expediente con `closure_summary`: "Factura TS001 cobrada en su totalidad".

**Principios que ilustra este flujo:**
- Los ítems del Template son punto de partida, no obligación. El ítem 4 es ad hoc y no estaba en el Template.
- Solo los participantes del círculo interno pueden ser mencionados. "María López" (aunque exista en la organización) no puede ser mencionada.
- El Hub Team y el usuario individual son igualmente mencionables.
- El cierre requiere que todos los ítems estén en estado terminal.

---

## 11. Módulos de Visión Futura

Los siguientes módulos están definidos como visión del producto pero **no forman parte del alcance de desarrollo actual**. Se documentan aquí para garantizar que las decisiones de arquitectura no los bloqueen.

### 11.1 Hub de Firma Jerárquico

Proceso de firma estructurado por niveles basado en la jerarquía del Hub Team. Características previstas:
- Orden de firma configurable (Nivel 1 firma antes que Nivel 2).
- Trazabilidad completa: IP, timestamp, hash de integridad por firmante.
- Cumplimiento eIDAS.
- Resumen en lenguaje natural generado por IA antes de la firma.

### 11.2 Integración de Inteligencia Artificial

- Análisis automático de documentos y extracción de información relevante.
- Generación de resúmenes de bitácora.
- Sugerencia de acciones basadas en patrones históricos del expediente.
- Detección de eventos externos (email, integración con buzón) para proponer ítems automáticos en la bitácora.

### 11.3 Proceso de Facturación

- Generación y seguimiento de facturas asociadas a expedientes.
- Estados: emitida, enviada, pagada, vencida.
- Gestión de recordatorios de cobro.
- Integración con sistemas externos de contabilidad.

### 11.4 API REST para Integración Externa

- Endpoints para creación y actualización de expedientes desde sistemas externos.
- Registro de ítems en bitácora desde sistemas externos.
- Webhooks para notificaciones salientes.
- Autenticación mediante API keys rotables.

### 11.5 Comunicaciones Externas con Contactos

- Canal de comunicación directa con los contactos del expediente (email, portal de acceso).
- Notificaciones automáticas a contactos por eventos relevantes del expediente.
- Historial de comunicaciones externas vinculado al expediente.

### 11.6 Reportes y Analítica

- Expedientes por estado, Hub Team, categoría y período.
- Tiempo medio de resolución por categoría.
- Actividad de Members y Hub Teams.
- Exportación en CSV y PDF.

### 11.7 Calendario e Integración

- Sincronización de eventos de bitácora con Google Calendar y Outlook.
- Recordatorios automáticos asociados a ítems con fecha límite.

---

## 12. Requisitos No Funcionales

### 12.1 Rendimiento

| Métrica | Objetivo |
|---|---|
| LCP landing | < 2.0s |
| LCP dashboard autenticado | < 2.5s |
| INP | < 200ms |
| CLS | < 0.1 |
| Peso inicial de página | < 1.5MB |

### 12.2 Seguridad y Arquitectura

- **Multi-tenant con RLS:** Aislamiento de datos a nivel de base de datos por organización. Ningún usuario accede a datos de otra organización.
- **Agnosticismo sectorial:** La estructura de datos y la interfaz evitan terminología específica de un sector.
- **Storage seguro:** URLs firmadas de acceso temporal para todos los documentos.
- **Validación de integridad:** Verificación de tipo de fichero en servidor.
- **Tokens de invitación:** UUID v4 de un solo uso para flujos de incorporación de usuarios.
- **Integridad referencial:** Bloqueo de borrado donde aplique (contactos con expedientes activos, etc.).

### 12.3 Accesibilidad y UX

- Cumplimiento WCAG 2.1 AA.
- Navegación completa por teclado: sidebar, formularios, modales.
- Wizard de creación de expedientes: flujo guiado por pasos.
- Layout de expedientes en modo Mosaico (Cards) con indicadores visuales de estado, importancia y participantes.
- Diseño mobile-first. Breakpoints verificados en 375px, 768px y 1280px+.

### 12.4 Legalidad y Trazabilidad

- Todos los registros en formato UTC. Visualización en zona horaria local del usuario.
- Historial de cambios de estado en expedientes e ítems.
- Registro inmutable de quién cierra cada expediente con timestamp UTC.

### 12.5 Internacionalización

- Idioma base: Español.
- Arquitectura preparada para i18n (textos externalizados desde el inicio).

---

## 13. Glosario

| Término | Definición |
|---|---|
| **Expediente** | Contenedor principal de un proceso de gestión para un contacto. Centro de gravedad del sistema. |
| **Bitácora** | Registro cronológico y narrativo de eventos dentro de un expediente. |
| **Case Item** | Ítem individual de la bitácora. Representa una acción, hito, nota o tarea. |
| **Hub Team** | Equipo de trabajo especializado compuesto por usuarios de la organización con jerarquía definida. |
| **Círculo de participantes internos** | Conjunto de usuarios y Hub Teams incorporados a un expediente concreto. Solo ellos pueden ser mencionados en los ítems. |
| **Template** | Plantilla de ítems predefinidos reutilizable. Se aplica una sola vez en la creación del expediente. |
| **Contacto** | Persona física o jurídica externa del CRM para la cual se gestiona un expediente. No es usuario de la app. |
| **Contacto principal** | Contacto del CRM obligatoriamente vinculado a cada expediente. |
| **Contacto secundario** | Contacto del CRM vinculado opcionalmente al expediente mediante `case_contacts`. |
| **Supervisor** | Owner o Admin responsable de un Hub Team. Gestiona su composición y jerarquía. |
| **Lead Member** | Usuario responsable principal de un expediente concreto. |
| **closure_summary** | Resumen de resolución obligatorio al cerrar un expediente. |
| **grade** | Nivel de importancia de un ítem: 1 Normal, 2 Alta, 3 Crítica. |
| **Estado terminal** | Estado de ítem que permite el cierre del expediente: `done` o `cancelled`. |
| **case_contacts** | Tabla pivote que gestiona la relación entre un expediente y sus contactos secundarios del CRM. |
| **case_item_mentions** | Tabla pivote que gestiona las menciones a participantes internos dentro de un ítem de bitácora. |

---

*Este documento es la fuente de verdad del producto Doculink v2.0. Cualquier decisión de arquitectura, diseño o implementación debe trazarse hasta un requisito aquí definido. Los módulos marcados como visión futura no son parte del alcance actual pero deben considerarse en las decisiones de arquitectura de datos para no bloquear su implementación posterior.*
