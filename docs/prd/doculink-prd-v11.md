# PRD — Documento de Requisitos del Producto: Doculink

**Proyecto:** Doculink (Gobernanza Documental y Firma Inteligente)  
**Versión:** 1.1 (Evolución M05: Gestión Narrativa)  
**Fecha:** 04-04-2026
**Estado:** 🟢 Aprobado
**Autor:** Antonio Garrido.

---

## 1. Resumen Ejecutivo
Doculink es una plataforma SaaS diseñada para centralizar la gestión de trámites profesionales en despachos, gestorías, inmobiliarias y cualquier entidad que requiera una gestión de procesos agnóstica. A diferencia de las herramientas de firma tradicionales, Doculink sitúa al **Expediente** como el centro de gravedad, permitiendo trazar no solo la firma de documentos, sino toda la narrativa de gestión humana y técnica que lo rodea, garantizando integridad legal y eficiencia operativa mediante Inteligencia Artificial.

---

## 2. Gestión de Identidad y Roles
El sistema trata la identidad de forma unificada: **todos los intervinientes son usuarios**. No hay una distinción técnica entre "firmante" y "usuario de la app", sino una distinción de permisos y capacidades dentro del sistema.

### 2.1 Roles de Usuario
* **Owner (Propietario):** Acceso total a la organización, gestión de facturación, planes y control global de datos.
* **Admin (Administrador):** Gestión de usuarios (altas/bajas), configuración de plantillas (templates) y auditoría de procesos.
* **Members (Miembros):** Usuarios operativos del despacho. Pueden crear expedientes, gestionar tareas y comunicarse con clientes.
* **Invited (Invitados):** Usuarios externos (clientes, contrapartes). Acceden mediante invitaciones seguras para subir documentos, leer resúmenes de IA e interactuar en procesos de firma.

---

## 3. Directorio de Contactos (CRM)
El sistema incluye un módulo de gestión de contactos para cualquier entidad que intervenga en la operativa, independientemente de su naturaleza.
* **Naturaleza:** Gestión de personas físicas (particulares) y jurídicas (empresas, ONGs, entes públicos).
* **Atributos:** Identificación fiscal (NIF/CIF), datos de contacto, representantes legales y preferencias de comunicación.
* **Cumplimiento:** Registro obligatorio de consentimientos GDPR para comunicaciones legales y notificaciones.

---

## 4. El Expediente: El Motor del Sistema
El expediente es el contenedor agnóstico donde ocurre la vida del proceso.

### 4.1 Apertura y Cabecera (Identidad)
* **Identificador Humano (case_number):** Cada expediente debe contar con un código legible único por organización (Ej: EXP-2026-001, HIST-442) para facilitar su localización fuera de los IDs técnicos.
* **Definición:** Título del expediente, descripción larga (briefing inicial), categoría (Jurídico, Médico, Fiscal, etc.) y etiquetas (tags) opcionales de contexto.
* **Responsabilidades:** Vinculación del contacto principal, asignación de un **Lead Member** (responsable principal) y equipo de **Assigned Members**.
* **Metodología:** Al abrirse, el expediente permite la selección de un **Work Template** para la carga automática de ítems predefinidos.

### 4.2 Seguimiento Narrativo (Items y Cronología)
El seguimiento no es un checklist estático, sino una bitácora narrativa compuesta por **Case Items**. Cada ítem representa un hito, nota o actuación técnica que cuenta el desarrollo del expediente.

* **Atributos del Ítem:**
    - **Contenido:** Relato detallado del avance o suceso.
    - **Importancia (Grade):** Escala de importancia 1 (Normal), 2 (Alta) o 3 (Crítica) para priorización visual.
    - **Tipología:** Clasificación entre Nota, Hito o Documento.
    - **Fecha de Ocurrencia:** Registro de cuándo sucedió el evento (independiente de la fecha de creación).
* **Automatización mediante Templates:** - El sistema permite predefinir "Líneas Descriptivas" en las plantillas de trabajo.
    - Al abrir un expediente basado en un Template, estas líneas se pre-cargan automáticamente como ítems iniciales en la línea de tiempo.
* **Gestión Documental Integrada (Anexos):** - Cada ítem permite anexar uno o varios documentos (IDs de la tabla maestra de documentos).
    - El usuario puede anexar evidencias físicas directamente desde el flujo de creación o edición del ítem narrativo.

### 4.3 Cierre
* **Validación:** El sistema verifica que los ítems marcados como críticos o de importancia alta estén completados.
* **Memoria de Cierre:** Registro obligatorio de quién cierra el expediente y un resumen final de resolución (Closure Summary).
* **Sello de Archivo:** Una vez cerrado, el expediente se bloquea para evitar modificaciones, generando un índice de evidencias legal inmutable.

---

## 5. Gestión Documental y Firma
Los documentos son los activos que validan el proceso. Todo documento tiene un **Expediente como destino**.

### 5.1 Atributos y Tipología
* Cada documento debe tener un **tipo asociado** (DNI, Contrato, Recibo, etc.) para facilitar su clasificación y búsqueda.
* **Modos de Uso:** Los documentos pueden ser simplemente almacenados (como evidencia de archivo vinculada a un ítem) o enviados al **Hub de Firma**.

### 5.2 El Hub de Firma Jerárquico
Proceso de firma estructurado por niveles de autorización.
* **Orden Jerárquico:** El sistema permite configurar un flujo donde el Nivel 2 no puede firmar hasta que el Nivel 1 haya completado su validación.
* **Trazabilidad:** Registro de IPs, marcas de tiempo y hashes de integridad para cada interviniente.

---

## 6. Inteligencia Artificial: Consentimiento Informado
La IA actúa como un puente de confianza entre el despacho y el cliente.
* **Resumen Ejecutivo:** Antes de proceder a la firma, el sistema genera automáticamente un resumen en lenguaje natural del documento.
* **Análisis de Cláusulas:** Identificación de puntos clave (importes, plazos, obligaciones) para asegurar que el usuario comprende el alcance de lo que firma.

---

## 7. Comunicaciones y Real-Time
* **Email Transaccional:** Notificaciones automáticas de cada hito (expediente abierto, documento pendiente, firma completada).
* **Notificaciones en Tiempo Real (Bell Icon):** Alerta instantánea sobre cualquier cambio en los expedientes asignados (nuevo ítem añadido, firma recibida).

---

## 8. Requisitos No Funcionales (Estándar Industrial)

### 8.1 Rendimiento
- LCP landing < 2.0s — landing estática optimizada.
- LCP dashboard < 2.5s — primera carga autenticada.
- Procesamiento Asíncrono: El análisis por IA se ejecuta en segundo plano.
- Lazy Loading: El motor de firma y visor PDF solo cargan sus recursos cuando el usuario llega a la pantalla de firma.

### 8.2 Seguridad y Arquitectura
- **Aislamiento Multi-tenant (RLS):** Los datos están aislados a nivel de base de datos; los usuarios solo acceden a información de su propia organización.
- **Agnosticismo:** La estructura de datos y la interfaz deben evitar términos específicos de un sector, permitiendo su uso en cualquier contexto profesional.
- **Tokens de Acceso:** UUID v4 single-use para invitados.
- **Storage Seguro:** URLs firmadas de acceso temporal.
- **Validación de Integridad:** Verificación de tipo de fichero en servidor y protección de API keys.
- **Integridad Referencial:** Bloqueo de borrado de contactos (ON DELETE RESTRICT) si existen expedientes vinculados.

### 8.3 Accesibilidad y UX
- Cumplimiento WCAG 2.1 AA.
- **Visualización de Mosaico:** El dashboard principal de expedientes debe utilizar un layout de tarjetas (Cards) con indicadores visuales de progreso, importancia y miembros asignados.
- **Navegación:** Sidebar y controles principales navegables mediante teclado.
- **Wizard de Creación:** El proceso de apertura de expedientes debe ser un flujo guiado y por pasos (Wizard) para mejorar la usabilidad.

### 8.4 Legalidad y Trazabilidad
- Cumplimiento eIDAS.
- Evidencia Inmutable: Página de trazabilidad en cada PDF consolidado.
- Sincronización Temporal: Registros en formato UTC, visualización en zona horaria local.

---