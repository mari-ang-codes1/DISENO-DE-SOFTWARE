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

[https://www.figma.com/make/fQeMDEdPk2sCGm7yn6JI0a/BarberFlow-Booking-App?t=E1sQn9B5KzyHr3Bj-1]

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

## 5.7 Matriz de Trazabilidad

| Requerimiento / Restricción | Atributo de Calidad | Decisión de Diseño | Componente | Estructura de Datos | Interfaz |
|---------------------------|---------------------|--------------------|------------|--------------------|---------|
| **RF1** Explorar catálogo de servicios | Usabilidad | Diseño de menú desplegable y tarjetas de servicio con información clara | UIComponent | Listado de objetos Servicio [id, nombre, precio, duración] | Pantalla 2: Catálogo |
| **RF2** Seleccionar barbero disponible | Usabilidad | Filtro en tiempo real por disponibilidad y especialidad | CitaService | Objeto Barbero [id, nombre, especialidad, turnoActual] | Pantalla 2: Catálogo |
| **RF3** Agendar cita respetando franja | Rendimiento | Validación en CitaService antes de persistencia | CitaService | Estructura DatosCitaInput con validación pre-persistencia | Pantalla 2→3 |
| **RN1** Restricción de horario duplicado | Disponibilidad | Validación transaccional en base de datos con bloqueo optimista | DatabaseAdapter | Constraints únicas en tabla Citas sobre (cliente, fechaHora) | Transversal |
| **RF4** Gestión ciclo vida cita | Usabilidad | Estados visuales en dashboard y flujos de navegación definidos | CitaService | Campo enum estado en entidad Cita | Dashboard Barbero |
| **RF5** Notificaciones WhatsApp | Disponibilidad | Integración con Twilio API con manejo de fallbacks | NotificationAdapter | Registro notificacionWhatsApp (enviada, intentos, últimoIntento) | Flujo Pantalla 3 |
| **RN2** Cancelación 2h antelación | Usabilidad | Validación de tiempo en CitaService, mensaje de advertencia UI | CitaService | Campo horaCancelacion en entidad Cita, comparación con DateTime.now | Pantalla 3 |
| **RN3** Validación turno barbero | Disponibilidad | Consulta de tabla Turnos antes de asignar cita | CitaService | Objeto DatosTurnoLaboral, validación horaria | Transversal |
| **RNF1** Interfaz móvil ágil | Rendimiento | Optimización de imágenes, lazy loading, API REST ligera | UIComponent | Cacheo client-side, responsive design breakpoints | Todas las pantallas |
| **RNF2** Disponibilidad 24/7 | Disponibilidad | Arquitectura Cliente-Servidor con redundancia | DatabaseAdapter | Connection pool, horarios de mantenimiento fuera de pico | API Gateway |
| **RNF3** Rendimiento <500ms | Rendimiento | Indexación de tablas, caché de consultas frecuentes | DatabaseAdapter | Índices en fechaHora, idBarbero, idCliente en tabla Citas | Consultas API |
| **RNF4** Interoperabilidad API | Interoperabilidad | API RESTful con OpenAPI specification, compatibilidad multiplataforma | CitaController, NotificationAdapter | Esquemas JSON para requests/responses, webhook URLs | API Twilio, navegadores |

6. Fowler, M. (2014). *Patterns of Enterprise Application Architecture*. Boston: Addison-Wesley.

---

**Proyecto BarberFlow - Diseño de Software**  
Asignatura: Diseño de Software - Profesor Sergio Lubo  
Integrantes: María Ángel Arias López, Angeli Cecilia Zambrano Navarro, Carlos Urieles López, Leticia Vega Melo  
Año académico: 2026

## 5.8 Frontera del Sistema e Interfaces

### 5.8.1 Declaración de la Frontera del Sistema

La **Frontera del Sistema** de BarberFlow define el límite entre lo que el sistema gestiona internamente y lo que depende de servicios externos. Matemáticamente, se puede expresar como:

```
Frontera(Sistema) = {Entidades, Componentes, Datos bajo control transaccional} ∂ SistemaExterno
```

El sistema BarbberFlow gestiona integralmente el ciclo de vida de las citas, la agenda del barbero, los perfiles de cliente y las tarifas de servicio dentro de su perímetro transaccional. Cualquier elemento que requiera coordinación fuera de este perímetro (como la interacción con el usuario final en tiempo real o servicios de notificación asíncronos) cruza la frontera de manera controlada y con mecanismos de fallback definidos.

Los actores dentro de la frontera son:
- **Cliente**: Interactúa dentro del perímetro a través de la interfaz móvil/web
- **Barbero**: Gestiona su agenda y estados dentro del sistema
- **Administrador**: Opera sobre configuraciones y reportes dentro del sistema

Los elementos que cruzan la frontera están explícitamente identificados y gestionados mediante adaptadores que aíslan las dependencias externas.

### 5.8.2 Justificación del Criterio de Frontera: Internas vs Externas

La distinción entre interfaces internas y externas no se basa en la tecnología de transporte (REST, HTTP, JSON), sino en el **criterio de frontera de control transaccional y dominio de responsabilidad**:

#### ¿Por qué UI <-> Backend es INTERNA?

1. **Mismo dominio de negocio**: Ambas operan dentro del mismo caso de uso de gestión de citas barbería. La UI (interfaz de usuario) y el Backend comparten el mismo modelo de datos y reglas de negocio RN1-RN3.

2. **Mismo límite de confianza**: Ambos componentes están bajo el mismo modelo de seguridad y despliegue. El desarrollador o equipo de operaciones tiene control sobre ambos lados de la comunicación.

3. **Compacidad transaccional**: La comunicación UI <-> Backend forma parte de la transacción principal de agendamiento. No hay necesidad de mecanismos de recuperación complejos porque fallos aquí se manejan dentro del mismo proceso de transacción.

4. **Mismo ciclo de vida de despliegue**: Tanto la UI como el Backend se despliegan conjuntamente o bajo la misma estrategia de release train, lo que los mantiene dentro de la misma frontera operativa.

```
Ejemplo: Cuando el cliente selecciona un barbero en la Pantalla 2, la comunicación 
Pantalla -> API REST -> CitaService es INTERNA porque:
- El endpoint /api/citas forma parte del mismo servicio deployado
- La validación de reglas RN1 (sin horario duplicado) ocurre en CitaService inmediatamente
- No hay dependencia externa ni tiempo de respuesta incierto que requiera mecanismos de fallback
```

#### ¿Por qué CitaService <-> Base de Datos es INTERNA?

1. **Mismo ámbito de transacción**: La base de datos es el sistema de persistencia del mismo servicio. La transacción ACID abarca tanto la lógica de negocio como el almacenamiento.

2. **Mismo modelo de seguridad**: El acceso a la base de datos usa credenciales y permisos definidos dentro de la política de seguridad del sistema, no credenciales de terceros.

3. **Competencia de fallos**: Los fallos de base de datos se manejan con patrones de retry y circuit breaker dentro del mismo contexto de ejecución, no como un servicio externo impredecible.

4. **Datos estructurales**: Los datos (entidades Cita, Barbero, Servicio) son propiedad del dominio del sistema, no de un proveedor externo.

```
Ejemplo: Cuando CitaService valida RN1 (restricción de horario duplicado), la consulta 
a la base de datos es INTERNA porque:
- La tabla Citas pertenece a la misma instancia de base de datos
- El patrón de bloqueo optimista es interno al servicio
- No hay SLA externo ni proveedor involucrado
```

#### ¿Por qué API de Twilio/WhatsApp es EXTERNA?

1. **Proveedor de servicio terzo**: Twilio es una empresa externa al proyecto BarberFlow, con su propia SLA, contratos de servicio y políticas de uso.

2. **Falla independiente**: La red, disponibilidad y rendimiento de Twilio son independientes del sistema BarberFlow. Pueden caer mientras el sistema interno funciona correctamente.

3. **Costo y suscripción**: El uso de Twilio implica costos por mensaje, límites de tasa y requisitos de cuenta que están fuera del control del equipo de BarberFlow.

4. **Mecanismos de fallback obligatorios**: Por definición de frontera externa, el sistema debe diseñar mecanismos de recuperación (colas Outbox, reintentos, notificaciones alternativas) porque la falla es esperada y frecuente.

```
Ejemplo: El envío de notificación por WhatsApp es EXTERNA porque:
- Requiere credenciales de cuenta Twilio (Account SID + Auth Token) - credenciales de tercero
- La API tiene sus propios límites de tasa (rate limits) y costos por mensaje
- Una falla en Twilio no implica fallo del sistema de agendamiento (la cita se confirma igual)
- Se requiere el patrón Outbox/NotificaciónPendiente por definición de frontera externa
- La dirección de endpoint (api.twilio.com) y autenticación HTTP Basic son ajenas al dominio barbería
```

#### Cuadro Comparativo de Criterio de Frontera

| Característica | Interna (UI<->Backend, CitaService<->BD) | Externa (Twilio/WhatsApp) |
|----------------|------------------------------------------|---------------------------|
| **Propiedad** | Mismo proyecto/equipo | Tercero (Twilio Inc.) |
| **SLA/Disponibilidad** | Dentro de control operativo | Proveedown third-party SLA |
| **Costo por operación** | Incluido en infraestructura | Por mensaje (tarifa Twilio) |
| **Fallas esperadas** | Manejo de errores estándar | Diseño obligatorio de fallback |
| **Credenciales** | Variables de entorno del sistema | Credenciales de cuenta externa |
| **Ciclo de despliegue** | Conjunto o coordinado | Independiente (Twilio gestión propia) |
| **Patrón de resiliencia** | Retry/Saga transaccional | Outbox + Reintentos + Notificaciones alternativas |

---

## 5.9 Fichas Detalladas de Componentes

### Ficha A: Componente Crítico de Reserva (Gestión de Citas - C-03)

| Campo | Contenido |
|-------|-----------|
| **Identificador** | C-03 |
| **Nombre del componente** | Gestión de Citas |
| **Propósito** | Coordinar el ciclo de vida completo de una cita: solicitud, validación de reglas de negocio RN1-RN3, persistencia en agenda barbero y activación de notificaciones externas. Garantizar que no existan reservas superpuestas y que el estado de la cita evolucione Solicitada → Confirmada → En Atención → Finalizada/Cancelada. |
| **Responsabilidades** | - Validar restricción de horario duplicado (RN1) antes de persistencia<br>- Validar turno laboral del barbero (RN3) antes de asignar cita<br>- Validar tiempo mínimo de cancelación (RN2) en operaciones de cancelación<br>- Coordinar persistencia atómica con base de datos<br>- Programar notificaciones mediante NotificationAdapter<br>- Retornar estado y datos de la cita creada al cliente| |
| **Exclusiones** | - No gestiona pagos o procesamiento financiero directo<br>- No maneja la lógica de negocio de servicios adicionales fuera del horari<br>- No responsabilizarse de la interfaz de usuario más allá de los DTOs de respuesta| |
| **Interfaces Provistas** | `crearCita(datosCita: DatosCitaInput): ResultadoCita`<br>`cancelarCita(citaId: string, razon?: string): ResultadoCancelacion`<br>`confirmarCita(citaId: string): Cita`<br>`finalizarCita(citaId: string, observaciones?: string): Cita`<br>`obtenerAgendaBarbero(barberoId: string, fecha: Date): AgendaBarbero`<br>`listarCitasCliente(clienteId: string, fechaInicio?: Date, fechaFin?: Date): Cita[]`|
| **Interfaces Requiridas** | - `DatabaseAdapter`: persistencia y validación de constraints únicas<br>- `NotificationAdapter`: envío de notificaciones post-percistencia<br>- `AuthController`: validación de sesión y rol del usuario que crea la cita| |
| **Clases de Datos** | `DatosCitaInput`: clienteId, barberoId, servicioId, fechaHora, tipoRecordatorio<br>`ResultadoCita`: citaId, estado, mensaje, notificacion?<br>`ResultadoCancelacion`: cancelada, reembolso?, mensaje, penalizacion<br>`AgendaBarbero`: fecha, citas: CitaAgendada[], huecosDisponibles:<br>`CitaAgendada`: citaId, clienteNombre, servicioNombre, horaInicio, horaFin, estado`<br>`DatosTurnoLaboral`: barberoId, diaSemana, horaInicio, horaFin| |
| **Justificación con RNF-02** | La ficha C-03 debe diseñarse considerando **RNF-02 (Disponibilidad)**, ya que la gestión de citas es el núcleo operativo del sistema. Durante las horas pico de reserva (mañanas lunes-viernes), el componente debe: (1) manejar múltiples solicitudes concurrentes sin bloqueos que degraden la disponibilidad, (2) mantener tiempos de respuesta < 500ms en operaciones de consulta de agenda, (3) garantizar uptime del 99.5% mensual a través de transacciones ACID y conexión pool en DatabaseAdapter, (4) prevenir la doble reserva atómicamente mediante constraints únicas en base de datos y validación en CitaService antes de commit, asegurando que la disponibilidad percibida por el cliente sea consistente con la realidad del sistema. La ficha debe incluir mecanismos de circuit breaker y retry con backoff exponencial para operaciones de escritura, garantizando que la caída de un solo nodo no afecte la disponibilidad global del agendamiento. |

### Ficha B: Componente de Integración (Gestión de Notificaciones - C-04)

| Campo | Contenido |
|-------|-----------|
| **Identificador** | C-04 |
| **Nombre del componente** | Gestión de Notificaciones |
| **Propósito** | Coordinar el envío de notificaciones automáticas a través de canales externos, priorizando WhatsApp API (Twilio) pero con mecanismos de fallback a email y indicadores visuales en la interfaz. Desacoplar la confirmación de la cita de la entrega exitosa de la notificación, garantizando que la cita persista independientemente del estado del servicio externo. |
| **Responsabilidades** | - Recibir solicitudes de notificación desde CitaService<br>- Formatear payload JSON para API externa (Twilio WhatsApp)<br>- Implementar patrón Outbox para acumular notificaciones pendientes<br>- Gestionar reintentos con backoff exponencial (máx 3 intentos en 24h)<br>- Registrar estado de envío (enviada, fallida, pendiente) en base de datos<br>- Proporcione notificación alternativa (email) cuando Twilio falla después de reintentos<br>- Exponer estado de notificación para consulta en dashboard administrador| |
| **Exclusiones** | - No genera el contenido del mensaje (recibe plantilla y datos desde CitaService)<br>- No garantiza la lectura o visualización por parte del cliente final<br>- No gestiona la suscripción o preferencias de canal del cliente (solo envío)<br>- No responsables de fallos de red fuera del control del sistema| |
| **Interfaces Provistas** | `enviarNotificacion(tipo: string, destino: string, contenido: string): ConfirmacionEnvio`<br>`programarRecordatorio(citaId: string, tiempoAntes: enum['24h','1h','nenhum']): IdentificadorRegistro`<br>`obtenerEstadoNotificacion(idNotificacion: string): EstadoNotificacion`<br>`reintentarNotificacionesPendientes(): numeroReintentos`|
| **Interfaces Requiridas** | - `CitaService`: notificaciones a programar y datos de cita<br>- `DatabaseAdapter`: persistencia de registro Outbox y estado de notificación<br>- `Twilio API`: servicio externo de mensajería (access Token, endpoints)<br>- `Configuración de credenciales`: Account SID, Auth Token, número WhatsApp empresarial| |
| **Patrón de Diseño** | **Outbox Pattern / NotificaciónPendiente**: <br>1. CitaService persiste la cita en BD en una transacción atomic<br>2. Después del commit, CitaService inserta un registro en tabla `notificacion_outbox` con: citaId, tipo, destino, contenido, estado='pendiente', intentos=0, creado_en<br>3. Un *worker* o *scheduler* independiente lee la tabla outbox cada N segundos<br>4. Para cada registro pendiente, el worker intenta enviar vía Twilio API<br>5. Si éxito: actualiza estado='enviada', registra SID de Twilio, incrementa intentos<br>6. Si falla: incrementa intentos, actualiza útlimoIntento, si intentos < 3 reprograma, si intentos >= 3 marca estado='fallback' y dispara notificación alternativa<br>7. El usuario ve siempre "Cita confirmada" al instante; la notificación es un bono asíncrono| |
| **Justificación con RNF-03** | La ficha C-04 debe diseñarse considerando **RNF-03 (Rendimiento)**, ya que la gestión de notificaciones debe no degradar el sistema de agendamiento principal. Consideraciones clave: (1) El patrón Outbox aísla el tiempo de respuesta de la operación de agendamiento: el cliente recibe confirmación al instante (HTTP 201) sin esperar la respuesta de Twilio, cumpliendo el SLA de < 2s para operaciones de consulta/agendamiento, (2) El worker de reintentos se ejecuta en segundo plano, no bloquea el hilo principal del API gateway, (3) Los backoff exponencial (1min, 5min, 30min) evitan efecto tormenta en la API de Twilio durante picos de fallos, (4) La base de datos Outbox usa índices en `estado` y `creado_en` para queries eficientes de "pendientes mayores a 24h", (5) Si Twilio está caído, la cadencia de reintentos respeta los 24h de ventana para completar los 3 intentos, evitando saturar el sistema con intentos consecutivos, (5) El rendimiento global del sistema no se ve comprometido porque la falla de un servicio externo está desacoplada de la transacción transaccional coreana. |

---

## 5.10 Prueba Razonada de Escenarios Críticos

### Escenario 1: Simultaneidad (RNF-02) - Prevención de Doble Reserva Atómica

**Contexto**: El sistema BarberFlow recibe dos solicitudes concurrentes de agendamiento para la misma franja horaria con el mismo barbero en el intervalo [inicio, fin). Ejemplo: Dos clientes intentan agendar una cita para el barbero "Juan" el lunes a las 10:00.

**Flujo de ejecución en el backend**:

1. **Llegada de solicitud 1**: El cliente accede a la Pantalla 2 y envía `crearCita()` al API Gateway. CitaService inicia una transacción de base de datos.

2. **Validación RN1 en CitaService**: Antes de persistir, CitaService ejecuta `validarHorarioDuplicado(cliente, fechaHora)` que realiza una consulta con `SELECT COUNT(*) FROM citas WHERE barbero_id = ? AND fechaHora >= inicio AND fechaHora < fin`. Esta consulta usa un índice sobre `fechaHora` en la tabla Citas.

3. **Llegada de solicitud 2 (concurrente)**: Mientras la transacción 1 está en curso, el sistema recibe la segunda solicitud. La arquitectura Cliente-Servidor con base de datos relacional emplea **bloqueo optimista o pesimista** en el nivel de aislamiento de transacción.

4. **Comportamiento dependiendo del nivel de aislamiento**:
   - **Aislamiento SERIALIZABLE**: La segunda transacción espera a que la primera termine o falla con error de serialización, forzando al cliente a retry con nueva franja horaria.
   - **Aislamiento REPEATABLE READ**: Las lecturas dentro de cada transacción ven el estado inicial, y al confirmar, se verifica si los datos cambiaron. Si la primera transacción ya insertó una cita en la franja, la segunda falla en la validación RN1 y retorna error "Horario no disponible".

5. **Persistencia atómica**: Si la validación pasa (no hay superposición), CitaService ejecuta `INSERT INTO citas (...) VALUES (...)` dentro de la misma transacción. El commit persiste la cita y libera el lock sobre la franja horaria.

6. **Resultado**: La segunda solicitud es rechazada en la validación RN1 con mensaje "Horario ya ocupado" o "Franja horaria no disponible", obligando al cliente a seleccionar otra franja o barbero. La reserva queda garantizada atómicamente: o ambas caben en la franja (imposible por validación) o una es rechazada.

**Diagrama de secuencia simplificado**:

```
Solicitud 1          Solicitud 2 (concurrente)
       │                           │
       ├──> validarHorarioDuplicado │
       │                           │
       ├──> (inicia transacción BD) │
       │                           │
       │           └──> validarHorarioDuplicado │
       │           │                 (bloqueo/lock)│
       │           │                           │
       │           └──> error "duplicado" │ ← segunda solicitud falla validación
       │                           │
       ├──> INSERT INTO citas ...   │
       │                           │
       ├──> transacción.commit()    │
       │                           │
       │   └──> cita confirmada      │
       │        estado: Confirmada   │
       └──> retorno éxito al cliente │
```

**Conclusión**: La combinación de validación en CitaService + constraints únicas en base de datos + nivel de aislamiento transaccional adecuado (SERIALIZABLE o REPEATABLE READ) garantiza que, en el intervalo [inicio, fin), no existan reservas superpuestas. La doble reserva atómica está prevista tanto a nivel de aplicación como de persistencia.

### Escenario 2: Proveedor Caído (RNF-03) - Falla de Twilio/WhatsApp

**Contexto**: El sistema BarberFlow tiene una cita confirmada para el cliente María, pero la API de Twilio/WhatsApp presenta una caída total (mantenimiento, sobrecarga de red, error de cuenta). El objetivo es que la cita se confirme y persista normalmente, sin que la falla del proveedor external afecte la transacción de agendamiento.

**Flujo de ejecución**:

1. **Confirmación de cita**: El cliente confirma la reserva en la Pantalla 3. CitaService ejecuta la transacción:
   - Validación RN1 (horario disponible) ✓
   - Validación RN3 (barbero en turno) ✓
   - `INSERT INTO citas` + `transacción.commit()` ✓
   - La cita queda con estado **Confirmada** en la base de datos, independientemente de Twilio

2. **Activación de NotificationAdapter**: Después del commit, CitaService llama a `programarNotificaciones(citaID)`. NotificationAdapter intenta insertar en la tabla `notificacion_outbox` y retorna éxito inmediato a CitaService.

3. **Intento de envío a Twilio**: NotificationAdapter (worker en segundo plano) intenta `POST /Messages` a `api.twilio.com`. Twilio no responde o retorna error 503.

4. **Patrón Outbox activation**:
   - El registro queda en tabla `notificacion_outbox` con `estado='pendiente'`, `intentos=1`, `últimoIntento=timestamp`
   - El worker detecta el registro pendiente y programa el próximo reintentó con backoff exponencial

5. **Secuencia de reintentos**:
   - **Intento 1** (1 min después): Twilio aún caído → error 503 → actualizar `intentos=1`
   - **Intento 2** (5 min después): Twilio aún caído → error 503 → actualizar `intentos=2`
   - **Intento 3** (30 min después): Twilio aún caído → error 503 → actualizar `intentos=3`

6. **Después de 3 intentos fallidos**:
   - El sistema marca `estado='fallback'` en la tabla de notificaciones
   - Se dispara notificación alternativa: envío de email al cliente (si tiene email registrado) o indicador en dashboard administrador "Recordatorio WhatsApp no enviado"
   - El usuario recibe mensaje: "Tu cita está confirmada. El recordatorio por WhatsApp no pudo enviarse automáticamente. Puedes ver la cita en tu agenda."

7. **Comportamiento crítico garantizado**:
   - **La cita persiste** con estado Confirmada/EnAtención/Finalizada, sin importar el estado de Twilio
   - **El agendamiento no se revierte**: `transacción.commit()` ya se ejecutó antes de los intentos de notificación
   - **El usuario es informado** sobre el estado real de las notificaciones
   - **El sistema continúa operando**: Nuevas citas pueden agendarse y confirmarse inmediatamente

**Diagrama de secuencia simplificado**:

```
CitaService
       │
       ├──> transacción.begin()
       │     │
       │     ├──> validar RN1, RN3
       │     ├──> INSERT cita
       │     ├──> transacción.commit()  ✓←─── CITA PERSISTE INDEPENDIENTEMENTE
       │     │
       │     └──> programarNotificaciones(citaID)
       │              │
       │              ├──> insertar Outbox (estado=pendiente)
       │              │
       │              └──> retorno éxito al cliente ("Cita confirmada")
       │
   NotificationAdapter (worker)
           │
           ├──> leer Outbox: registro pendiente
           │
           ├──> POST Twilio /Messages  ⨉ (caído 503)
           │        │
           │        └──> actualizar Outbox: intentos=1, últimoIntento=now
           │
           ├──> esperar backoff 1min
           │
           ├──> POST Twilio /Messages  ⨉ (caído 503)
           │        │
           │        └──> actualizar Outbox: intentos=2
           │
           ├──> esperar backoff 5min
           │
           ├──> POST Twilio /Messages  ⨉ (caído 503)
           │        │
           │        └──> actualizar Outbox: intentos=3
           │
           ├──> actualizar Outbox: estado='fallback'
           │
           ├──> disparar notificación alternativa (email)
           │
           └──> actualizar dashboard: badge "recordatorio no enviado"

**Conclusión**: Ante una falla de Twilio/WhatsApp, la cita se confirma y persiste en la base de datos gracias a la transacción atomic separada del envío de notificaciones. El patrón Outbox + reintentos con backoff exponencial garantiza que el sistema intente recuperar el envío sin bloquear la operación coreana, y después de 3 intentos fallidos, cambia a modo fallback (email/indicador) manteniendo la continuidad del servicio. El RNF-03 (Rendimiento) se cumple porque el tiempo de respuesta del agendamiento no depende del proveedor externo.
```

---

## 10. Trazabilidad IA-Decisión (Sección 10 ampliada)

### 10.1 Tabla Formal Momento | Qué propuso el equipo | Intervención de IA | Decisión final | Justificación del equipo

| Momento | Qué propuso el equipo | Intervención de IA | Decisión final | Justificación del equipo |
|---------|----------------------|-------------------|----------------|-------------------------|
| **T1: Inicio (Sept 2026)** | Definición de nombre provisional "BarberFlow" y lista de integrantes | IA sugirió estructura inicial del documento PROPUESTA.md con formato Markdown | **ACEPTADA** | El equipo validó que la estructura coincidía con el enunciado de la asignatura y los nombres eran correctos según el grupo |
| **T2: Estructura DOCUMENTO_DISENO.md** | Definición de las 7 secciones principales (5.1 a 6.1) y contenido mínimo por sección | IA propuso el outline inicial basándose en prompts anteriores | **ACEPTADA** | El equipo revisó y ajustó el orden lógico: movió "Matriz de Trazabilidad" a 5.7 y agregó "Conclusiones y Referencias" al final. Se mantuvo la secuencia propuesta por IA como base |
| **T3: Definición de Reglas de Negocio (RN1-RN3)** | El equipo escribió las 3 reglas de negocio basadas en el caso de barbería | IA sugirió la formulación inicial y ayudó a estructurarlas en formato Markdown tabla | **ACEPTADA** | El equipo validó que RN1 (doble reserva), RN2 (cancelación 2h) y RN3 (validación de turno) eran las correctas según el enunciado y las enriqueció con detalles de implementación |
| **T4: Elección Arquitectónica** | El equipo decidió entre Monolito MVC vs. Cliente-Servidor Desacoplada | IA generó tabla comparativa y justificativas técnicas para los 4 aspectos (usabilidad, escalabilidad, integración, mantenimiento) | **ACEPTADA** | El equipo validó las 4 razones técnicas (acceso móvil, escalabilidad, integración de Twilio, desarrollo paralelo) y agregó la sección de "Descarte de Monolítica" con sus 4 argumentos. Las 4 justificativas fueron consideradas sólidas y se mantuvieron |
| **T5: Diseño de Interfaces (5.5)** | El equipo definió las 3 subsecciones: UI de navegación, contratos internos CitaService, contratos externos Twilio | IA generó los primeros borradores de los 3 diagramas Mermaid.js y los contratos de interfaz con campos nombre/solicitante/proveedor | **ACEPTADA** | El equipo revisó y completó los diagramos: adicionó los detalles de las 6 operaciones de CitaService, las validaciones de errores (CitaErrorCode) y el patrón Outbox/NotificaciónPendiente. Los contratos Twilio fueron enriquecidos con el manejo de códigos HTTP y la política de reintentos |
| **T6: Matriz de Trazabilidad (5.7)** | El equipo creó la tabla que conecta RF/RN a Calidad ↔ Diseño ↔ Componente ↔ Datos ↔ Interfaz | IA ayudó a poblar las primeras filas y sugerir la estructura de 6 columnas | **ACEPTADA** | El equipo revisó cada fila: aseguró que todos los RF1-RF6, RN1-RN3 y RNF1-RNF4 estuvieran presentes, corrigió el emparejamiento de algunos componentes y añadió las estructuras de datos y interfaces específicas (DTOs, endpoints, tipos de error) |
| **T7: Escenarios Críticos (5.10)** | El equipo redactó los dos escenarios racionales (Simultaneidad RNF-02, Proveedor Caído RNF-03) | IA generó los diagramas de secuencia Mermaid.js y la redacción técnica inicial | **ACEPTADA** | El equipo validó la lógica atómica de la doble reserva y el patrón Outbox con reintentos, adicionó detalles sobre niveles de aislamiento de transacción y la política de reintentos exactamente como se especifica (máx 3 intentos, backoff 1-5-30 min). Ambos escenarios fueron considerados exemplares para el caso de uso |
| **T8: Declaración de Uso de IA (Sección 10)** | El equipo definía cómo la IA había sido usada y quería una tabla formal | IA sugirió la estructura de tabla Momento/Propuesta/Intervención/Decisión/Justificación | **ACEPTADA** | El equipo completó los 8 momentos (T1-T8), describió cada intervención de IA con ejemplos concretos y estableció el criterio de que todo contenido conceptual y de diseño fue revisado y validado por el humano. La tabla quedó como registro transparente del proceso |

**Historial de Decisiones Clave**:

| Tipo | Cantidad |
|------|----------|
| **Decisiones ACEPTADAS** | 8 (todas las propuestas de IA fueron validadas y enriquecidas por el equipo) |
| **Decisiones RECHAZADAS** | 0 (ninguna propuesta de IA fue descartada; todas sirvieron como base que el equipo mejoró) |

**Nota final**: La intervención de la IA opencode/nemotron-3.5-lightning-free fue determinante para la **estructura inicial**, **formateo Markdown**, **diagramas Mermaid.js** y **redacción técnica** de los 7 apartados del documento. Sin embargo, todo el contenido de dominio (reglas de negocio BarberFlow, caso de uso, decisiones arquitectónicas frente a los requerimientos del enunciado, lógica de escenarios críticos) fue definido, contrastado y validado completamente por el equipo humano BarberFlow (María Ángel, Angeli, Carlos, Leticia) antes de su inclusión final en el documento DOCUMENTO_DISENO.md.
