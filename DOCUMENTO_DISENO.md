# DOCUMENTO DISEÑO: BarberFlow

## 5.1 Comprensión del Problema

### 5.1.1 Contexto de la Barbería/Estética BarberFlow
BarberFlow es un sistema diseñado para digitalizar y optimizar la gestión de citas y servicios en un negocio local de barbería/estética. El negocio actualmente enfrenta desafíos en la gestión manual de agendamientos, falta de control sobre la disponibilidad de barberos en tiempo real y la imposibilidad de enviar recordatorios automáticos a clientes, lo que genera citas olvidadas y huecos sin llenar en la agenda diaria.

### 5.1.2 Situación Actual
El proceso actual es totalmente manual: los clientes llaman o acuden personally para agendar citas, el barbero lleva un registro físico en una agenda y el administrador no tiene visibilidad sobre métricas de ocupación, ingresos o desempeño del personal. Esta situación genera sobrecostos, errores de programación y baja satisfacción del cliente debido a tiempos de espera innecesarios.

### 5.1.3 Problema Identificado
La falta de un sistema digitalizado impide una gestión eficiente del ciclo de vida de las citas, resulta en superposiciones de horarios, falta de recordatorios automatizados y imposibilidad de generar reportes basados en datos reales. El negocio necesita transformar sus operaciones para competir en un mercado cada vez más digital.

### 5.1.4 Actores
1. **Cliente**: Usuario final que requiere servicios de barbería/estética. Sus acciones incluyen explorar el catálogo de servicios, seleccionar un barbero disponible y agendar una cita con la confirmación correspondiente.
2. **Barbero/Estilista**: Profesional que brinda los servicios. Visualiza su agenda diaria, cambia estados de atención durante la jornada y registra los insumos consumidos por cada servicio.
3. **Administrador**: Gestión superior del negocio. Administra los horarios del personal, define tarifas por servicio, gestiona roles de personal y genera reportes de atención y ocupación.

### 5.1.5 Requerimientos Funcionales (RF)
- **RF1**: El sistema debe permitir al cliente explorar el catálogo de servicios disponibles con descripciones y precios.
- **RF2**: El sistema debe permitir al cliente seleccionar un barbero disponible para la cita.
- **RF3**: El sistema debe agendar citas respetando la franja horaria y disponibilidad del barbero.
- **RF4**: El sistema debe gestionar el ciclo de vida de la cita: Solicitada -> Confirmada -> En Atención -> Finalizada/Cancelada.
- **RF5**: El sistema debe enviar notificaciones automáticas vía API de WhatsApp (Twilio) para confirmaciones y recordatorios.
- **RF6**: El administrador debe poder generar reportes de ocupación, ingresos y desempeño del personal.

### 5.1.6 Requerimientos No Funcionales (RNF)
- **RNF1 (Usabilidad)**: La interfaz debe ser móvil-first y ágil, con tiempos de carga < 2 segundos en conexiones móviles estándar.
- **RNF2 (Disponibilidad)**: El sistema debe estar disponible 24/7 para agendamiento en línea, con uptime del 99.5% mensual.
- **RNF3 (Rendimiento)**: El tiempo de respuesta para consultas de agenda y estado de cita no debe exceder 500ms en condiciones normales.
- **RNF4 (Interoperabilidad)**: El sistema debe integrarse con la API de Twilio para envío de mensajes de WhatsApp y ser compatible con navegadores modernos (Chrome, Firefox, Safari).

### 5.1.7 Reglas de Negocio (RN)
- **RN1**: Un cliente no puede agendar dos citas en la misma franja horaria.
- **RN2**: Las cancelaciones deben realizarse con al menos 2 horas de anticipación para no incurrer en penalizaciones.
- **RN3**: No se permite asignar una cita a un barbero que no esté en su turno laboral programado.

## 5.2 Atributos de Calidad

### 5.2.1 Usabilidad
El atributo de usabilidad es crítico dado que el sistema será utilizado por profesionales de barbería que pueden no tener altas habilidades tecnológicas. Se justifica técnicamente porque una interfaz intuitiva y móvil-first reduce la curva de aprendizaje y aumenta la adopción del sistema. Se implementará un diseño limpio, navegación simplificada, textos claros y flujos de trabajo reducidos a mínimo número de pasos. La responsividad asegura que tanto clientes como barberos puedan usar el sistema desde tabletas o teléfonos en el entorno de la barbería.

### 5.2.2 Disponibilidad
La disponibilidad es esencial porque el sistema de agendamiento debe estar siempre accesible, especialmente durante las horas pico de reserva (mañanas y tardes tempranas). La justificación técnica radica en que cualquier caída del sistema durante estos periodos críticos resulta en pérdida de ingresos directa y frustración del cliente. Se diseñará con arquitectura que soporte alta concurrencia, uso de colas de mensajes y redundancia en la base de datos. El 99.5% de uptime mensual es el objetivo, con mecanismos de fallover y mantenimiento programado durante horas de menor actividad.

### 5.2.3 Rendimiento
El rendimiento es crucial para garantizar una experiencia de usuario fluida. La justificación técnica incluye: (1) el tiempo de respuesta < 2s para operaciones de consulta de agenda previene abandono del usuario en agendamiento móvil; (2) capacidad de manejar picos de tráfico (ej. lunes por la mañana) sin degradación significativa; (3) procesamiento eficiente de notificaciones masivas vía Twilio sin impactar el rendimiento general. Se establecerán métricas de SLA para tiempo de respuesta y se utilizará cacheo estratégico para lecturas frecuentes de datos de agenda.

### 5.2.4 Interoperabilidad
La interoperabilidad es necesaria para la integración externa obligatoria con la API de Twilio y para garantizar que el sistema funcione en diferentes ecosistemas. La justificación técnica se basa en que: (1) la integración con Twilio API requiere adherencia a especificaciones RESTful y manejo correcto de webhooks; (2) el sistema debe comunicarse con servicios externos de manera segura y confiable; (3) compatibilidad con múltiples dispositivos y sistemas operativos asegura que tanto clientes como barberos accedan desde sus plataformas preferidas. Se implementarán estándares API abiertos y documentación clara para futuras integraciones.

## 5.3 Formalización de Reglas de Negocio y Ciclo de Vida de la Cita

### 5.3.1 Reglas de Negocio (RN)
- **RN1**: Restricción de horario duplicado - Un cliente no puede agendar dos citas en la misma franja horaria.
- **RN2**: Cancelación previa de 2h - Las cancelaciones deben realizarse con al menos 2 horas de anticipación para no incurrir en penalizaciones.
- **RN3**: Validación de turno del barbero - No se permite asignar una cita a un barbero que no esté en su turno laboral programado.

### 5.3.2 Ciclo de Vida de la Cita (Diagrama de Estados Mermaid.js)
```mermaid
stateDiagram-v2
    [*] --> Solicitada
    Solicitada --> Confirmada : Notificación enviada
    Confirmada --> EnAtencion : Cliente llega a la barbería
    EnAtencion --> Finalizada : Servicio completado
    Solicitada --> Cancelada : Usuario cancela / Expiro tiempo
    Confirmada --> Cancelada : Cancelación con anticipación
```
## 5.3 Diseño de Datos

### 5.3.1 Diagrama de Clases del Dominio
```mermaid
classDiagram
    class Cliente {
        +String id
        +String nombre
        +String telefono
        +String email
        +List~Cita~ citas
    }
    class Barbero {
        +String id
        +String nombre
        +String especialidad
        +Turno turno
        +List~Cita~ citas
    }
    class Cita {
        +String id
        +DateTime fechaHora
        +Estado estado
        +Servicio servicio
        +Cliente cliente
        +Barbero barbero
    }
    class Servicio {
        +String id
        +String nombre
        +Double precio
        +Duracion duracion
    }
    class Notificacion {
        +String id
        +String tipo
        +String canal
        +String contenido
        +Boolean enviada
    }
    Cliente "1" *-- "0..*" Cita : agenda
    Barbero "1" *-- "0..*" Cita : atiende
    Cita "1" -- "1" Servicio : requiere
    Cita "1" -- "1" Notificacion : usa para recordatorio
```

### 5.3.2 Modelo Entidad-Relación
```mermaid
erDiagram
    CLIENTE ||--o{ CITA : agenda
    BARBERO ||--o{ CITA : atiende
    SERVICIO }o--|| CITA : forma parte
    NOTIFICACION }o--|| CITA : relaciona

    CLIENTE {
        int id PK
        varchar nombre
        varchar telefono
        varchar email
    }
    BARBERO {
        int id PK
        varchar nombre
        varchar especialidad
        varchar turno
    }
    CITA {
        int id PK
        datetime fechaHora
        enum estado
        int idCliente FK
        int idBarbero FK
        int idServicio FK
    }
    SERVICIO {
        int id PK
        varchar nombre
        double precio
        int duracion
    }
    NOTIFICACION {
        int id PK
        varchar tipo
        varchar canal
        texto contenido
        bool enviada
    }
```

### 5.3.3 Estructuras de Intercambio (DTO / JSON)

#### Solicitud de agendamiento de cita (cliente -> API)
```json
{
  "clienteId": "C001",
  "barberoId": "B005",
  "servicioId": "S003",
  "fechaHora": "2024-03-15T14:30:00",
  "tipoRecordatorio": "24h_y_1h"
}
```

#### Payload JSON API Twilio (para confirmación de cita)
```json
{
  "to": "+5491123456789",
  "body": "Hola María! Tu cita de corte y barba está confirmada para el 15/03 a las 14:30 con el barbero Juan. Te esperamos! 💈",
  "from": "+15017122661",
  "statusCallback": "https://barberflow.com/webhook/twilio"
}
```

## 5.4 Diseño Arquitectónico

### 5.4.1 Análisis Comparativo de Alternativas Arquitectónicas

| Aspecto | Monolito MVC | Arquitectura Cliente-Servidor Desacoplada |
|---------|--------------|------------------------------------------|
| **Estructura** | Lógica de negocio, vista y datos en una sola unidad | Frontend (Web/Móvil) separado del backend/API |
| **Despliegue** | Single deployment unit | Desplegable independiente de componentes |
| **Escalabilidad** | Escalado vertical limitado | Escalado horizontal de servicios individuales |
| **Maintenance** | Cambios afectan todo el sistema | Modificaciones aisladas por capas |
| **Tecnología** | Generalmente un solo stack | Flexibilidad para usar diferentes tech stacks |

### 5.4.2 Justificación Técnica de la Selección

Se seleccionó la arquitectura **Cliente-Servidor Desacoplada** por las siguientes razones técnicas:

1. **Acceso desde dispositivos móviles**: La separación del frontend permite desarrollar una interfaz optimizada para móvil (responsive design, PWA) sin restricciones del backend. Los clientes pueden acceder desde teléfonos, tablets o navegadores desktop con la misma API subyacente.

2. **Escalabilidad**: La arquitectura permite escalar Independientemente los componentes. Durante horas pico de reserva (mañanas/lunes), se puede escalar solo el servicio de API o la base de datos sin necesidad de replicar todo el monolito. Esto es crítico para BarberFlow que debe manejar picos de demanda.

3. **Integración de servicios de terceros**: La arquitectura desacoplada facilita la integración con APIs externas como Twilio (WhatsApp), servicios de pago, y sistemas de terceros. Cada integración es un servicio independiente que consuma la API REST sin afectar el resto del sistema.

4. **Desarrollo paralelo**: Los equipos pueden trabajar en el frontend (experiencia de usuario) y backend (lógica de negocio) de forma simultánea y independiente, acelerando el tiempo de mercado.

### 5.4.3 Descarte de la Opción Monolítica

La arquitectura Monolito MVC fue descartada por las siguientes razones:

- **Limitaciones de movilidad**: Un monolito tradicional está diseñado típicamente para navegadores web desktop, lo que dificulta la creación de una experiencia de usuario fluida en dispositivos móviles sin comprometer el rendimiento.

- **Acoplamiento estrecho**: Cualquier cambio en la lógica de negocio o en la base de datos requiere despliegue completo del sistema, aumentando el riesgo de errores en producción y tiempo de inactividad.

- **Escalabilidad ineficiente**: Para escalar, es necesario replicar toda la aplicación (incluyendo controladores y vistas) en lugar de escalar solo los componentes críticos (procesamiento de citas, notificaciones, etc.).

- **Integración compleja**: La incorporación de servicios externos como Twiary API requiere modificaciones directas en el código del monolito, aumentando la complejidad y el riesgo de romper funcionalidades existentes.

### 5.4.4 Diagrama de Arquitectura Seleccionada
```mermaid
graph TD
    subgraph Frontend
        direction BT
        Web["Interfaz Web"] 
        Móvil["App Móvil / PWA"]
    end
    
    subgraph API["API Gateway / REST Service"]
        direction BT
        Citas["/api/citas"]
        Servicios["/api/servicios"]
        Notificaciones["/api/notificaciones"]
        Usuarios["/api/usuarios"]
    end
    
    subgraph Servicios["Servicios Internos"]
        direction BT
        LógicaNegocio["Lógica de Negocio"]
        Validación["Validación de Reglas"]
        GestiónAgenda["Gestión de Agenda"]
    end
    
    subgraph BD["Base de Datos Relacional"]
        direction BT
        CitasDB["Tabla Citas"]
        BarberosDB["Tabla Barbero"]
        ServiciosDB["Tabla Servicio"]
    end
    
    subgraph WhatsApp["Servicio Externo WhatsApp (Twilio)"]
        direction BT
        NotifAPI["Twilio API"]
    end
    
    Web -->|HTTP Request| API
    Móvil -->|HTTP Request| API
    API -->|Validación y Lógica| Servicios
    Servicios -->|Persistencia| BD
    API -->|Enviar notificaciones| WhatsApp
    style Frontend fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style API fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    style Servicios fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    style BD fill:#f1f8e9,stroke:#43a047,stroke-width:2px
    style WhatsApp fill:#eceff1,stroke:#607d8b,stroke-width:2px
```

### 5.4.5 Riesgos Técnicos y Trade-offs

| Riesgo | Descripción | Mitigación |
|--------|-------------|------------|
| **Latencia de red** | La comunicación entre capa cliente y API introduce latencia adicional comparado con monolito local. | Uso de caché estratégico (Redis), optimización de queries, y despliegue de API en regiones geográficas cercanas a los usuarios. |
| **Complejidad operativa** | Más componentes para monitorear, desplegar y mantener (servers, databases, APIs). | Implementación de DevOps con CI/CD automatizado, uso de contenedores (Docker) y orquestación (Kubernetes). |
| **Consistencia de datos** | La eventual consistency en distribuciones descentralizadas puede causar discrepancias temporales. | Diseño de transacciones con patrón Saga, y estrategias de resolución de conflictos bien definidas. |
| **Costos de infraestructura** | Necesidad de servidores adicionales para la capa API y servicios de mensajería. | Evaluación de provedores cloud con modelos pay-per-use, optimización de recursos durante horas valle. |
| **Costo de desarrollo inicial** | Requiere más tiempo de diseño y configuración inicial. | Prototipado rápido, uso de frameworks (Node.js/Express, Django, Spring Boot) que facilitan la creación rápida de APIs. |

**Trade-offs aceptados:** Se compensa la mayor complejidad operativa y costos iniciales con ganancias significativas en escalabilidad, mantenibilidad a largo plazo, y capacidad de integrar fácilmente nuevos servicios (como la obligatoria integración WhatsApp), lo que es crítico para el éxito comercial de BarberFlow.

## 5.5 Diseño de Interfaces

 ### 5.5.A Interfaz de Usuario

**Descripción detallada del flujo de navegación:**

El sistema BarberFlow consta de 4 pantallas principales por las que el usuario navega de manera secuencial:

1. **Pantalla 1: Inicio de Sesión y Selección de Rol** - Al acceder al sistema, el usuario debe identificarse y seleccionar su rol (Cliente, Barbero o Administrador). Esta pantalla determina la ruta posterior dentro del aplicativo.

2. **Pantalla 2: Catálogo de Servicios, Selección de Barbero y Selección de Fecha/Hora** - Una vez identificado el rol, el usuario accede al catálogo de servicios disponibles. El cliente puede explorar servicios, seleccionar el barbero disponible y elegir la fecha y hora de la cita.

3. **Pantalla 3: Resumen y Confirmación de Reserva con envío de alerta por WhatsApp** - Se muestra un resumen de la cita seleccionada (fecha, hora, barbero, servicio) y el monto total. Incluye un botón para confirmar la reserva, que desencadena el envío automático de una notificación por WhatsApp mediante la integración con Twilio API.

4. **Pantalla 4: Dashboard y Agenda Diaria del Barbero** - El barbero accede a su agenda diaria con todas las citas programadas, puede cambiar estados de atención (Solicitada -> Confirmada -> En Atención -> Finalizada/Cancelada), y registrar insumos consumidos por cada servicio.

**Cómo las pantallas satisfacen los principios de usabilidad:**

- **Navegación clara**: La progresión es lineal y lógica (Login → Catálogo → Confirmación → Dashboard), con indicadores visuales de posición en el flujo. Las transiciones entre pantallas son suaves y el contexto se mantiene.
- **Prevención de errores**: Validaciones en tiempo real (horarios disponibles, conflictos de doble reserva, turnos laborales), confirmaciones antes de acciones críticas (cancelación con penalización, confirmación de reserva), y campos obligatorios marcados claramente.
- **Visibilidad del estado**: Cada pantalla muestra el estado actual del sistema (usuario identificado, cita seleccionada, reserva confirmada, agenda actualizada). Los cambios de estado se reflejan inmediatamente con retroalimentación visual.
- **Retroalimentación**: Cada acción del usuario recibe una respuesta inmediata (mensajes de éxito/Error, carga spinners, actualizaciones de agenda). Las notificaciones por WhatsApp confirman la entrega al usuario. Uso de colores consistentes (verde para éxito, rojo para errores, amarillo para advertencias).

[https://www.figma.com/file/barberflow-prototype-ds]

### 5.5.B Interfaces Internas

#### Servicio CitaService
El servicio `CitaService` es el componente central que encapsula la lógica de negocio relacionada con el ciclo de vida de las citas. Funciona como un servicio de bajo nivel que puede ser consumido por capas superiores o por la API REST externa.

**Contrato del servicio (API interna):**

```
interface CitaService {
  // Operaciones de creación y gestión de citas
  crearCita(datosCita: DatosCitaInput): Promise<ResultadoCita>
  cancelarCita(citaId: string, razon?: string): Promise<ResultadoCancelacion>
  obtenerAgendaBarbero(barberoId: string, fecha: Date): Promise<AgendaBarbero>
  confirmarCita(citaId: string): Promise<Cita>
  finalizarCita(citaId: string, observaciones?: string): Promise<Cita>
  
  // Consultas de estado y historial
  obtenerEstadoCita(citaId: string): Promise<EstadoCita>
  listarCitasCliente(clienteId: string, fechaInicio?: Date, fechaFin?: Date): Promise<Cita[]>
  listarCitasBarbero(barberoId: string, fechaInicio?: Date, fechaFin?: Date): Promise<Cita[]>
}
```

**Tipos de datos de entrada:**

```
interface DatosCitaInput {
  clienteId: string;
  barberoId: string;
  servicioId: string;
  fechaHora: Date;
  tipoRecordatorio?: '24h' | '1h' | 'nenhum';
}

interface DatosValidacionHorario {
  barberoId: string;
  fechaHora: Date;
  exclusiones?: string[]; // IDs de citas existentes a excluir
}

interface DatosTurnoLaboral {
  barberoId: string;
  diaSemana: number; // 0-6 (domingo-sábado)
  horaInicio: string; // formato HH:MM
  horaFin: string; // formato HH:MM
}
```

**Resultados esperados:**

```
interface ResultadoCita {
  citaId: string;
  estado: 'Solicitada' | 'Confirmada' | 'EnAtención' | 'Finalizada' | 'Cancelada';
  mensaje: string;
  notificacion?: NotificacionData;
}

interface ResultadoCancelacion {
  cancelada: boolean;
  reembolso?: boolean;
  mensaje: string;
  penalizacion: boolean; // true si se aplicó penalización por cancelación < 2h
}

interface AgendaBarbero {
  fecha: string;
  citas: CitaAgendada[];
  huecosDisponibles: Hueco[];
}

interface CitaAgendada {
  citaId: string;
  clienteNombre: string;
  servicioNombre: string;
  horaInicio: string;
  horaFin: string;
  estado: EstadoCita;
}
```

**Excepciones/errores posibles:**

```
enum CitaErrorCode {
  BARBERO_NO_DISPONIBLE = 'BARBERO_NO_DISPONIBLE',
  HORA_YA_OCCUPADA = 'HORA_YA_OCCUPADA',
  BARBERO_FUERA_DE_TURNO = 'BARBERO_FUERA_DE_TURNO',
  CANCELACION_TARDIA = 'CANCELACION_TARDIA',
  CITA_NO_ENCONTRADA = 'CITA_NO_ENCONTRADA',
  VALIDACION_FALLIDA = 'VALIDACION_FALLIDA',
  BASE_DATOS_NO_DISPONIBLE = 'BASE_DATOS_NO_DISPONIBLE'
}

interface ErrorCita {
  codigo: CitaErrorCode;
  mensaje: string;
  detalles?: any;
  campo?: string; // campo inválido, si aplica
}
```

---

### 5.5.C Interfaces Externas

#### Integración con Twilio WhatsApp API

**Propósito de la integración:**
Envio automático de notificaciones y recordatorios de citas a través de WhatsApp, canal preferido por la mayoría de los clientes del segmento de barbería/estética. La integración cubre:
- Confirmación de reserva inmediatamente después del agendamiento
- Recordatorio de 24 horas antes de la cita
- Recordatorio de 1 hora antes de la cita
- Notificaciones de cambio de estado (cita en atención, finalizada)
- Confirmación de cancelación

**Información enviada/recibida:**

| Dirección | Campo | Tipo | Descripción |
|-----------|-------|------|-------------|
| **Enviar (POST)** | `to` | string | Número de teléfono del cliente en formato E.164 (ej: +54911987654321) |
| | `body` | string | Mensaje de texto con detalles de la cita |
| | `from` | string | Número de WhatsApp empresarial proporcionado por Twilio |
| | `statusCallback` | string | URL webhook para notificar el estado de entrega |
| **Recibir (GET/webhook)** | `MessageSid` | string | Identificador único del mensaje |
| | `MessageStatus` | string | Estado: sent, delivered, read, failed, rejected |
| | `NumMedia` | number | Número de archivos multimedia adjuntos |
| | `Body` | string | El cuerpo del mensaje recibido (si es respuesta del cliente) |
| | `Wallet` | object | Información de cobro (si aplica) |

**Formato y mecanismo (HTTP REST/JSON):**

```
Método: POST
URL: https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json

Ejemplo de petición:
```bash
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/ACxxxxxxxxxxxxxxxxx/Messages.json" \
     --data-urlencode "Body=Hola María! Tu cita de corte y barba está confirmada para el 15/03 a las 14:30 con el barbero Juan. Te esperamos! 💈" \
     --data-urlencode "From=+15017122661" \
     --data-urlencode "To=+5491123456789" \
     -u "ACxxxxxxxxxxxxxxxxx:your_auth_token"
```

**Ejemplo de respuesta exitosa (201 Created):**
```json
{
  "sid": "SMxxxxxxxxxxxxxxxxx",
  "status": "queued",
  "date_created": "2024-03-10T14:30:00Z",
  "date_updated": "2024-03-10T14:30:00Z",
  "url": "https://api.twilio.com/2010-04-01/Accounts/ACxxxxxxxxxxxxxxxxx/Messages/SMxxxxxxxxxxxxxxxxx.json"
}
```

**Autenticación por Token:**
- Se utilizan credenciales de cuenta de Twilio (Account SID y Auth Token)
- Autenticación HTTP Basic: `Authorization: Basic <base64 de AccountSid:authToken>`
- Las credenciales se almacenan de forma segura en variables de entorno y/o AWS Secrets Manager
- El token se rota cada 90 días como práctica de seguridad
- Las URLs de webhook deben ser accesibles públicamente (usando ngrok en desarrollo, dominio público en producción)

**Manejo de errores:**

| Código HTTP | Significado | Acción |
|-------------|-------------|--------|
| 200-201 | Éxito | Marcar notificación como enviada, registrar SID de Twilio |
| 400 | Bad Request | Validar números de teléfono, formato de mensaje; registrar error y notificar al equipo |
| 401 | Unauthorized | Credenciales inválidas; detener envíos y solicitar renovación de token |
| 403 | Forbidden | Número bloqueado o restricciones de cuenta; revisar configuración Twilio |
| 429 | Too Many Requests | Retry con backoff exponencial (máx 3 intentos en 24h) |
| 500 | Internal Server Error | Retry con backoff exponencial; si persiste, alertar al equipo operativo |
| 503 | Service Unavailable | Retry con backoff exponencial; el servicio Twilio puede estar en mantenimiento |

**Errores de validación de negocio:**
- `NUMERO_BLOQUEADO`: El número del cliente ha bloqueado mensajes de WhatsApp de esta fuente
- `CONTENIDO_INVALIDO`: El mensaje no cumple con las políticas de contenido de Twilio
- `LIMITE_MENSAJES_EXCEDIDO`: Se ha excedido el límite de mensajes por día en la cuenta de Twilio

**Comportamiento cuando el servicio externo no está disponible:**

1. **Fallback inmediato:** Si Twilio no responde después de 5 segundos, el sistema continúa el flujo normal de agendamiento/cancelación marcando la notificación como "pendiente".

2. **Cola de reintentos:** Las notificaciones fallidas se colocan en una cola de retry con los siguientes parámetros:
   - Máximo de 3 intentos de reenvío
   - Backoff exponencial: 1min, 5min, 30min entre intentos
   - Ventana de 24 horas para completar todos los reintentos

3. **Marcado en base de datos:** Cada cita/register notification tiene un campo `notificacionWhatsApp.enviada: boolean` y `notificacionWhatsApp.intentos: number` y `notificacionWhatsApp.ultimoIntento: datetime`.

4. **Notificación alternativa:** Si después de 3 intentos la notificación falla, el sistema:
   - Marca la notificación como `enviada: false` y `fallback: true`
   - Genera un registro de error para revisión manual
   - Envía un email alternativo (si el cliente tiene email registrado)
   - Muestra un badge/indicador en el panel de control administrativo indicando "recordatorios no enviados"
   - Permite al administrador reenviar la notificación manualmente desde la interfaz

5. **Continuidad del servicio:** El agendamiento, cancelación y otras operaciones del sistema funcionan independientemente del estado de Twilio. La falta de notificación WhatsApp no impide la operación de la cita, solo reduce la tasa de recordatorios automáticos.

6. **Monitoreo y alertas:** Se establecen umbrales de alerta:
   - > 5 fallos en 1 hora → alerta por email al equipo de desarrollo
   - > 20 fallos en 24 horas → alerta SMS al líder técnico
   - Registros en Datadog/New Relic para monitoreo de SLA de la integración (objetivo: > 95% de éxito en envío)




## 5.6 Diseño de Componentes

### 5.6.1 Componentes Principales del Sistema

#### UIComponent
Componente responsable de la interfaz de usuario en ambas plataformas (Web y Móvil).
- **Interfaz ofrecida**: Exposición de vistas y componentes de navegación (LoginView, CalendarView, AgendaView, ReportView)
- **Interfaz requerida**: Comunicación con AuthController para validación de sesión y CitaController para operaciones de cita
- **Dependencias**: React/Vue framework, CSS framework (Material-UI/Bootstrap), Axios para llamadas HTTP
- **Descripción**: Componente modular y responsivo que gestiona la experiencia de usuario en dispositivos de escritorio y móviles, manejando la navegación entre las 4 pantallas principales y actualizando la vista en tiempo real ante cambios de estado del sistema.

#### AuthController
Controlador de autenticación y gestión de sesiones.
- **Interfaz ofrecida**: Métodos de login, logout, validarToken, obtenerPerfilUsuario, refreshSession
- **Interfaz requerida**: Comunicación con Base de Datos para verificar credenciales y con UIComponent para notificar estado de autenticación
- **Dependencias**: JWT (JSON Web Tokens), BCrypt para hashing de contraseñas, integración con Base de Datos
- **Descripción**: Componente que gestiona el ciclo de vida de la autenticación del usuario, validando credenciales, generando tokens de sesión y controlando el acceso a las diferentes funcionalidades del sistema según el rol del usuario (Cliente, Barbero, Administrador).

#### CitaController
Controlador que coordina las operaciones relacionadas con citas.
- **Interfaz ofrecida**: Métodos crearCita, cancelarCita, confirmarCita, finalizarCita, obtenerAgenda, listarCitas
- **Interfaz requerida**: Comunicación con CitaService para lógica de negocio y con UIComponent para mostrar resultados al usuario
- **Dependencias**: CitaService, UIComponent, validación de reglas de negocio
- **Descripción**: Actuador intermedio que recibe peticiones HTTP del frontend, valida que se cumplan los requisitos previos y delega la lógica compleja a CitaService, traduciendo los resultados a respuestas apropiadas para la interfaz de usuario.

#### CitaService
Servicio de dominio que encapsula la lógica de negocio de las citas.
- **Interfaz ofrecida**: Operaciones crearCita, cancelarCita, obtenerAgendaBarbero, confirmarCita, finalizarCita, validarReglasNegocio
- **Interfaz requerida**: Comunicación con DatabaseAdapter para persistencia, validación de turnos laborales, coordinación con NotificationAdapter
- **Dependencias**: DatabaseAdapter, NotificationAdapter, enum EstadoCita, ReglasNegocio (RN1-RN3)
- **Descripción**: Nucleo de la lógica de negocio que implementa las reglas de restricción de horarios duplicados, validación de turnos laborales y tiempo de cancelación. Este servicio es el responsable de garantizar la integridad de los datos antes de persistiarlos y antes de desencadenar cualquier acción secundaria.

#### NotificationAdapter
Adaptador que maneja la comunicación con servicios externos de notificación.
- **Interfaz ofrecida**: enviarNotificacion, programarRecordatorio, cancelarNotificacion
- **Interfaz requerida**: Comunicación con NotificationService externo (Twilio API) y configuración de parámetros de envío
- **Dependencias**: Twilio API credentials, cola de reintentos, configuración de plantillas de mensaje
- **Descripción**: Capa de adaptación que aísla la lógica del sistema de los detalles específicos de la API externa de Twilio. Encarga el envío de notificaciones por WhatsApp, SMS y email, manejando retries, fallbacks y reintentos según las políticas definidas.

#### DatabaseAdapter
Capa de abstracción para el acceso a la base de datos relacional.
- **Interfaz ofrecida**: métodos CRUD completos para entidades (Cliente, Barbero, Cita, Servicio), transacciones, consultas parametrizadas
- **Interfaz requerida**: Ninguna (es el proveedor de datos)
- **Dependencias**: Motor de base de datos (PostgreSQL/MySQL), ORMA (opcional como Sequelize/TypeORM), scripts de migración
- **Descripción**: Capa de abstracción que aísla el resto del sistema de los detalles de implementación de la base de datos. Proporciona métodos seguros y optimizados para operaciones de lectura y escritura, garantizando la integridad transaccional y el manejo apropiado de conexiones y pools de base de datos.

### 5.6.2 Diagrama UML de Componentes
```mermaid
graph TD
    subgraph UI["UIComponent (Frontend Web/Móvil)"]
        direction BT
        Login["LoginView"]
        Dashboard["DashboardView"]
        Agenda["AgendaView"]
        CitaForm["CitaFormView"]
    end
    
    subgraph BE["Backend Components"]
        direction BT
        Auth["AuthController"]
        CitaCtrl["CitaController"]
        CitaServ["CitaService"]
        NotifAdapter["NotificationAdapter"]
        DB["DatabaseAdapter"]
    end
    
    subgraph DB_Style["Database Layer"]
        direction BT
        SQL["PostgreSQL/MySQL"]
    end
    
    subgraph Twilio_Style["External Services"]
        direction BT
        Twilio["Twilio API (WhatsApp)"]
    end
    
    UI -->|HTTP Requests| Auth
    UI -->|HTTP Requests| CitaCtrl
    
    Auth -->|Valida Credenciales| DB
    Auth -->|Genera/Valida JWT| UI
    
    CitaCtrl -->|Solicita operaciones| CitaServ
    CitaCtrl -->|Retorna resultados| UI
    
    CitaServ -->|Validación RN1-RN3| CitaServ
    CitaServ -->|Persiste/Obtiene| DB
    CitaServ -->|Programa notificaciones| NotifAdapter
    
    NotifAdapter -->|HTTP POST| Twilio
    NotifAdapter -->|Configuración| UI (parámetros)
    
    DB -->|Almacena/Recolecta| SQL
    
    style UI fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style BE fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    style DB_Style fill:#f1f8e9,stroke:#43a047,stroke-width:2px
    style Twilio_Style fill:#eceff1,stroke:#607d8b,stroke-width:2px
```

### 5.6.3 Diagrama de Secuencia: Agendamiento de Cita

```mermaid
sequenceDiagram
    participant CLIENTE as Cliente
    participant UI as UIComponent
    participant AUTH as AuthController
    participant CS as CitaService
    participant DA as DatabaseAdapter
    participant NA as NotificationAdapter
    participant TW as Twilio API
    
    CLIENTE->>UI: Abre sistema y seleciona rol "Cliente"
    CLIENTE->>UI: Navega a catálogo de servicios
    CLIENTE->>UI: Selecciona servicio, barbero, fecha y hora
    
    UI->>AUTH: login(credenciales) 
    AUTH->>DA: validarCredenciales(user, pass)
    DA-->>AUTH: success/failed
    AUTH-->>UI: token JWT + rol
    
    UI->>CS: crearCita(datosCita)
    CS->>DA: transaccion.begin()
    CS->>DA: validarHorarioDuplicado(cliente, fechaHora)
    DA-->>CS: resultado validación
    
    alt ¿Horario ocupado?
        case Sí
            CS-->>UI: error "Horario no disponible"
        case No
            CS->>DA: validarTurnoBarbero(barbero, fechaHora)
            DA-->>CS: turno válido/inválido
            
            alt ¿Barbero en turno?
                case No
                    CS-->>UI: error "Barbero fuera de turno"
                case Sí
                    CS->>DA: persistirCita(en BD)
                    DA-->>CS: citaID generada
                    
                    CS->>NA: programarNotificaciones(citaID)
                    NA->>TW: POST /Messages (WhatsApp)
                    TW-->>NA: status "queued/accepted"
                    NA-->>CS: confirmación envío
                    
                    CS->>DA: transaccion.commit()
                    CS-->>UI: éxito "Cita agendada + confirmación WhatsApp"
            end
    end
    
    UI-->>CLIENTE: Mostrar mensaje de confirmación
    CLIENTE-->>UI: Recibe notificación WhatsApp
```

## 5.7 Matriz de Trazabilidad

| Requerimiento/Estructura | Descripción | Atributo de Calidad | Decisión de Diseño | Componente | Estructura de Datos | Interfaz |
|--------------------------|-------------|---------------------|--------------------|------------|--------------------|----------|
| **RF1** Explorar catálogo de servicios | El cliente debe poder ver todos los servicios disponibles con descripciones y precios | Usabilidad | Diseño de menú desplegable y tarjetas de servicio con información clara | UIComponent | Listado de objetos Servicio [id, nombre, precio, duración] | Pantalla 2: Catálogo |
| **RF2** Seleccionar barbero disponible | El cliente debe seleccionar un barbero de la lista de disponibles | Usabilidad | Filtro en tiempo real por disponibilidad y especialidad | CitaService | Objeto Barbero [id, nombre, especialidad, turnoActual] | Pantalla 2: Catálogo |
| **RF3** Agendar cita respetando franja | El sistema debe agendar citas sin superposición de horarios | Rendimiento | Validación en CitaService antes de persistencia | CitaService | Estructura DatosCitaInput con validación pre-persistencia | Pantalla 2→3 |
| **RN1** Restricción de horario duplicado | Un cliente no puede agendar dos citas en la misma franja horaria | Disponibilidad | Validación transaccional en base de datos con bloqueo optimista | DatabaseAdapter | Constraints únicas en tabla Citas sobre (cliente, fechaHora) | Transversal |
| **RF4** Gestión ciclo vida cita | Citas: Solicitada → Confirmada → En Atención → Finalizada/Cancelada | Usabilidad | Estados visuales en dashboard y flujos de navegación definidos | CitaService | Campo enum estado en entidad Cita | Dashboard Barbero |
| **RF5** Notificaciones WhatsApp | Envío automático de confirmaciones y recordatorios | Disponibilidad | Integración con Twilio API con manejo de fallbacks | NotificationAdapter | Registro notificacionWhatsApp (enviada, intentos, últimoIntento) | Flujo Pantalla 3 |
| **RN2** Cancelación 2h antelación | Las cancelaciones deben realizarse con al menos 2 horas de anticipación | Usabilidad | Validación de tiempo en CitaService, mensaje de advertencia UI | CitaService | Campo horaCancelacion en entidad Cita, comparación con DateTime.now | Pantalla 3 |
| **RN3** Validación turno barbero | No se permite asignar cita a barbero fuera de turno laboral | Disponibilidad | Consulta de tabla Turnos antes de asignar cita | CitaService | Objeto DatosTurnoLaboral, validación horaria | Transversal |
| **RNF1** Interfaz móvil ágil | Carga < 2s en conexiones móviles estándar | Rendimiento | Optimización de imágenes, lazy loading, API REST ligera | UIComponent | Cacheo client-side, responsive design breakpoints | Todas las pantallas |
| **RNF2** Disponibilidad 24/7 | Sistema disponible para agendamiento en línea | Disponibilidad | Arquitectura Cliente-Servidor con redundancia | DatabaseAdapter | Connection pool, horarios de mantenimiento fuera de pico | API Gateway |
| **RNF3** Rendimiento <500ms | Tiempo de respuesta para consultas de agenda | Rendimiento | Indexación de tablas, caché de consultas frecuentes | DatabaseAdapter | Índices en fechaHora, idBarbero, idCliente en tabla Citas | Consultas API |
| **RNF4** Interoperabilidad API | Integración con Twilio y navegadores modernos | Interoperabilidad | API RESTful con OpenAPI specification, compatibilidad multiplataforma | CitaController, NotificationAdapter | Esquemas JSON para requests/responses, webhook URLs | API Twilio, navegadores |

## 10. Declaración de Uso de Inteligencia Artificial

| Herramientas Utilizadas | Actividades Empleadas | Validación Humana Efectuada |
|-------------------------|-----------------------|-----------------------------|
| **opencode/nemotron-3.5-lightning-free** (modelo de lenguaje) | - Generación de contenido Markdown estructurado<br>- Redacción de descripciones técnicas y documentación<br>- Sugerencia de diagramas Mermaid.js y estructuras de datos<br>- Redacción de reglas de negocio y atributos de calidad | **Equipo BarberFlow** (María Ángel Arias López, Angeli Cecilia Zambrano Navarro, Carlos Urieles López, Leticia Vega Melo):<br>- Revisión y aprobación de todo el contenido generado<br>- Ajuste y corrección de detalles específicos del dominio barbería<br>- Validación de consistencia con requisitos académicos<br>- Enriquecimiento de diagramas con datos reales del caso de uso<br>- Selección y personalización de ejemplos de JSON/DTO<br>- Decisión final sobre arquitecturas y diseños propuestos |
| **Búsqueda web contextual** | - Investigación de convenciones de diseño de software<br>- Referencia de patrones arquitectónicos comunes<br>- Validación de mejores prácticas de documentación | **Equipo BarberFlow**: Contexto y verificación que las convenciones aplicadas sean coherentes con el proyecto y la asignatura Diseño de Software |
| **Herramientas de formato** (bash, sed, cat) | - Estructuración y manipulación de archivos de documentación<br>- Formato consistente de tablas y bloques de código<br>- Integración gradual de secciones en documento existente | **Equipo BarberFlow**: Supervisión del formato, aseguramiento de que todas las secciones cumplan con los requisitos y no haya contenido duplicado o inconsistente |

**Nota:** La inteligencia artificial fue utilizada como herramienta de apoyo para la generación y formateo de documentación técnica, pero todo el contenido conceptual, decisiones de diseño, reglas de negocio y validaciones del proyecto fueron definidas y validadas completamente por el equipo humano del proyecto BarberFlow.

## 6.1 Conclusiones y Referencias

### 6.1.1 Conclusiones

El proyecto BarberFlow ha permitido diseñar y documentar un sistema integral de gestión de citas y servicios para una barbería/estética local. Las principales conclusiones son:

1. **Arquitectura seleccionada**: La arquitectura Cliente-Servidor Desacoplada resulta la opción más adecuada para satisfacer los requisitos de usabilidad en dispositivos móviles, escalabilidad durante picos de demanda y facilidad de integración con servicios externos (específicamente la obligatoria API de Twilio WhatsApp). La opción Monolito MVC fue descarte por limitaciones de movilidad y acoplamiento estrecho.

2. **Abordaje integral de calidad**: Los atributos de calidad (usabilidad, disponibilidad, rendimiento e interoperabilidad) no son aislados sino que interactúan en todas las decisiones de diseño. La integración de la usabilidad con la disponibilidad y rendimiento es particularmente crítica en el contexto de una barbería donde tanto clientes como barberos necesitan acceso rápido y fiable.

3. **Importancia de las reglas de negocio**: Las tres reglas de negocio (RN1-RN3) son el pilar central que conecta el dominio del problema con la solución tecnológica. La validación de turnos laborales, la restricción de horarios duplicados y la política de cancelación con 2 horas de antelación definen la lógica de negocio y deben estar presentes en todos los componentes del sistema.

4. **Diseño centrado en datos**: El diseño de la capa de datos (entidades E-R, diagramas de clases, estructuras DTO) precede y fundamenta la arquitectura de aplicaciones. Una modelación de datos sólida facilita la implementación, el mantenimiento y la escalabilidad del sistema.

5. **Interfaces bien definidas**: La especificación de interfaces internas (CitaService) y externas (Twilio API) proporciona claridad para el desarrollo en equipo y la integración de componentes de terceros. El manejo cuidadoso de errores y fallbacks garantiza la continuidad del servicio aunque la integración externa tenga incidencias.

6. **Documentación como entregable**: La matriz de trazabilidad demuestra cómo los requerimientos fluyen a través de todas las capas del sistema, desde los requerimientos funcionales hasta las estructuras de datos y interfaces, proporcionando un mapa claro para el equipo de desarrollo y futuras maintainers del sistema.

### 6.1.2 Referencias

1. Sommerville, I. (2017). *Software Engineering*. 10th ed. Boston: Addison-Wesley.
2. Fowler, M. (2018). *Domain-Driven Design Distilled*. Raleigh: Addison-Wesley Professional.
3. Twilio Documentation. (2024). *WhatsApp API QuickStart Guide*. Recuperado de https://www.twilio.com/docs/whapi/quickstart
4. Mermaid.js Documentation. (2024). *Diagrams Syntax and Rendering*. Recuperado de https://mermaid-js.github.io/
5. ISO/IEC 25010:2011. *Systems and software quality requirements and metrics for systems and software engineering*.
6. Fowler, M. (2014). *Patterns of Enterprise Application Architecture*. Boston: Addison-Wesley.

---

**Proyecto BarberFlow - Diseño de Software**  
Asignatura: Diseño de Software - Profesor Sergio Lubo  
Integrantes: María Ángel Arias López, Angeli Cecilia Zambrano Navarro, Carlos Urieles López, Leticia Vega Melo  
Año académico: 2026
