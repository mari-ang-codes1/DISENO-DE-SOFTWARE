# PROPUESTA: BarberFlow - Sistema de Gestión de Citas y Servicios de Barbería

**Nombre del proyecto:** BarberFlow - Sistema de Gestión de Citas y Servicios de Barbería

**Integrantes:**
- María Ángel Arias López
- Angeli Cecilia Zambrano Navarro
- Carlos Urieles López
- Leticia Vega Melo

**Contexto:**
Sistema para digitalizar agendamiento de citas en barberías locales.

**3 Actores:**
- **Cliente:** Reserva citas, explora servicios
- **Barbero:** Gestiona agenda, atiende clientes
- **Administrador:** Reportes y gestión de tarifas

**Proceso Principal:**
Gestión del ciclo de vida de la Cita (Solicitada -> Confirmada -> En Atención -> Finalizada / Cancelada).

**Reglas de Negocio:**
1. Restricción de doble reserva en misma franja horaria
2. Cancelación mínima con 2 horas de antelación
3. Validación de turnos laborales antes de asignar cita

**Integración Externa:**
API de WhatsApp (Twilio) para notificaciones y recordatorios.

**Atributos de Calidad:**
- Usabilidad
- Disponibilidad
- Rendimiento