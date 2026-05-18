# PRD — Documento de Requisitos del Producto: Doculink

**Proyecto:** Doculink (Gobernanza Documental y Firma Inteligente)  
**Versión:** 1.0
**Fecha:** 01-04-2026
**Estado:** 🟢 Aprobado
**Autor:** Antonio Garrido.

---

## 1. Resumen Ejecutivo
Doculink es una plataforma SaaS diseñada para centralizar la gestión de trámites profesionales en despachos, gestorías e inmobiliarias. A diferencia de las herramientas de firma tradicionales, Doculink sitúa al **Expediente** como el centro de gravedad, permitiendo trazar no solo la firma de documentos, sino todo el proceso de gestión humana y técnica que lo rodea, garantizando integridad legal y eficiencia operativa mediante Inteligencia Artificial.

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
El expediente es el contenedor donde ocurre la vida del proceso. Se divide en tres etapas críticas:

### 4.1 Apertura
* **Cabecera:** Definición del título, vinculación del contacto principal, asignación de gestores y selección del **Template de Trabajo**.
* **Metodología:** Al abrirse, el expediente carga automáticamente los **Items de Task** predefinidos en la plantilla elegida.

### 4.2 Seguimiento (Tasks y Slots)
* **Tasks (Tareas):** Son los hitos u objetivos del expediente. Cada task tiene un estado (Pendiente/Terminado) y fechas de control (Inicio/Fin).
* **Slots (Bitácora de Información):** Dentro de cada Task, el seguimiento se realiza mediante **Slots**. Un Slot es un contenedor de información inmutable que registra:
    * Fecha y hora del evento.
    * Usuario que realiza la anotación.
    * Comentario o descripción del avance.
    * Situación de la tarea en ese momento (Abierta, Pausa, Anulada, etc.).
    * Vínculo opcional a un documento del expediente.

### 4.3 Cierre
* **Validación:** El sistema verifica que las tareas obligatorias estén completadas.
* **Sello de Archivo:** Una vez cerrado, el expediente se bloquea para evitar modificaciones, generando un índice de evidencias legal.

---

## 5. Gestión Documental y Firma
Los documentos son los activos que validan el proceso. Todo documento tiene un **Expediente como destino**.

### 5.1 Atributos y Tipología
* Cada documento debe tener un **tipo asociado** (DNI, Contrato, Recibo, etc.) para facilitar su clasificación y búsqueda.
* **Modos de Uso:** Los documentos pueden ser simplemente almacenados (como evidencia de archivo) o enviados al **Hub de Firma**.

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
* **Email Transaccional:** Notificaciones automáticas de cada hito (expediente abierto, documento pendiente, firma completada) con branding personalizado por organización.
* **Notificaciones en Tiempo Real (Bell Icon):** El dashboard de los usuarios incluye un icono de campana que alerta instantáneamente sobre cualquier cambio en los expedientes asignados (nuevo slot añadido, firma recibida).

---

## 8. Requisitos No Funcionales (Estándar Industrial)

### 8.1 Rendimiento
- LCP landing < 2.0s — landing estática optimizada
- LCP dashboard < 2.5s — primera carga autenticada
- Procesamiento Asíncrono: El análisis por IA se ejecuta en segundo plano, permitiendo al usuario seguir trabajando tras subir un documento.
- Lazy Loading: El motor de firma y visor PDF solo cargan sus recursos cuando el usuario llega a la pantalla de firma.

### 8.2 Seguridad
- Aislamiento Multi-tenant (RLS): Los datos están aislados a nivel de base de datos; los usuarios solo acceden a información de su propia organización.

- Tokens de Acceso: UUID v4 single-use.Uso de tokens de un solo uso para invitados, nunca indexables y con tiempo de vida limitado.

- Storage Seguro: Documentos almacenados con URLs firmadas de acceso temporal; nada es público por defecto.

- Validación de Integridad: Verificación de tipo de fichero en servidor (no solo extensión) y protección de API keys exclusivamente en el lado del servidor.
- timingSafeEqual para comparación de tokens.

### 8.3 Accesibilidad y UX
- Cumplimiento WCAG 2.1 AA
- Navegación: Sidebar y controles principales navegables íntegramente mediante teclado.
- Firma Accesible: El canvas de firma incluye descripciones claras y alternativas de proceso para garantizar su uso.

### 8.4 Legalidad y Trazabilidad
- Cumplimiento eIDAS: Firma electrónica simple válida para contratos privados en el marco legal europeo.
- Evidencia Inmutable: Generación de una página de trazabilidad en cada PDF consolidado que incluye nombres, correos, IPs y marcas de tiempo.

- Sincronización Temporal: Todos los registros se guardan en formato UTC, pero se muestran al usuario en su zona horaria local para evitar confusiones legales.

---
