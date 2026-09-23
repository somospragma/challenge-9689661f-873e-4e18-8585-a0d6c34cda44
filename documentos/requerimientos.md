# Requerimientos para la Solución de Originación Digital de Préstamos

## Contexto
El banco requiere implementar una solución de originación digital de préstamos que permita procesar solicitudes de crédito de manera eficiente, segura y escalable. La solución debe integrarse con múltiples sistemas externos (motor antifraude, buró de riesgos, core bancario, gateway de pagos, sistema de liquidación) y cumplir con estrictos requisitos de rendimiento y disponibilidad.

---

## Requerimientos Funcionales

### 1. Originador de Créditos
- **RF-001**: El sistema debe permitir la captura de solicitudes de préstamo a través de múltiples canales (web, móvil, sucursales).
- **RF-002**: Debe validar la información proporcionada por el solicitante (identidad, ingresos, historial crediticio) en tiempo real.
- **RF-003**: Debe calcular la capacidad de endeudamiento del solicitante basado en reglas predefinidas.
- **RF-004**: Debe generar una oferta de préstamo personalizada con tasas de interés, plazos y montos ajustados al perfil del solicitante.
- **RF-005**: Debe permitir la firma digital del contrato de préstamo.
- **RF-006**: Debe notificar al solicitante sobre el estado de su solicitud (aprobada, rechazada, pendiente de documentación) en tiempo real.

### 2. Integración con Sistemas Externos
- **RF-007**: Debe integrarse con el **motor antifraude** para validar la autenticidad de la información proporcionada por el solicitante.
  - **Detalle**: El motor antifraude debe devolver un score de riesgo (0-100) y una lista de alertas potenciales.
- **RF-008**: Debe integrarse con el **buró de riesgos** para obtener el historial crediticio del solicitante.
  - **Detalle**: El buró de riesgos debe devolver un reporte con puntuación crediticia, historial de pagos y obligaciones vigentes.
- **RF-009**: Debe integrarse con el **core bancario** para registrar la solicitud de préstamo y generar el número de cuenta asociado.
  - **Detalle**: El core bancario debe confirmar la creación de la cuenta y proporcionar el número de cuenta generado.
- **RF-010**: Debe integrarse con el **gateway de pagos** para procesar el desembolso del préstamo.
  - **Detalle**: El gateway de pagos debe confirmar la transferencia de fondos y proporcionar un comprobante de pago.
- **RF-011**: Debe integrarse con el **sistema de liquidación** para registrar el desembolso y actualizar los saldos contables.
  - **Detalle**: El sistema de liquidación debe confirmar la actualización de saldos y proporcionar un identificador de transacción.

### 3. Manejo de Estados
- **RF-012**: Debe manejar los siguientes estados para una solicitud de préstamo:
  - **Iniciada**: Solicitud capturada pero no validada.
  - **Validada**: Información del solicitante validada contra el motor antifraude y el buró de riesgos.
  - **En Evaluación**: Solicitud en proceso de cálculo de capacidad de endeudamiento y generación de oferta.
  - **Aprobada**: Oferta generada y aceptada por el solicitante.
  - **Firmada**: Contrato firmado digitalmente.
  - **Desembolsada**: Fondos transferidos al solicitante.
  - **Rechazada**: Solicitud rechazada por incumplimiento de políticas o alto riesgo.
  - **Pendiente de Documentación**: Solicitud requiere documentos adicionales para continuar.

### 4. Notificaciones
- **RF-013**: Debe enviar notificaciones al solicitante en cada cambio de estado de la solicitud.
  - **Canales**: Correo electrónico, SMS y notificaciones push (si la app móvil está disponible).
  - **Contenido**: Debe incluir el estado actual, los siguientes pasos y, en caso de rechazo, las razones principales.

### 5. Reportes y Auditoría
- **RF-014**: Debe generar reportes diarios de solicitudes procesadas, aprobadas, rechazadas y pendientes.
- **RF-015**: Debe registrar un log de auditoría para todas las acciones realizadas en el sistema (cambios de estado, integraciones con sistemas externos, firmas digitales).
  - **Detalle**: El log debe incluir timestamp, usuario (o sistema), acción realizada y datos relevantes de la acción.

---

## Requerimientos No Funcionales

### 1. Rendimiento
- **RNF-001**: El sistema debe soportar un volumen de **1,500 solicitudes por segundo** en hora pico.
- **RNF-002**: El tiempo de respuesta máximo para la generación de una oferta de préstamo debe ser de **2 segundos**.
- **RNF-003**: El tiempo de respuesta máximo para la validación de identidad y antifraude debe ser de **500 ms**.
- **RNF-004**: El sistema debe procesar el 95% de las solicitudes en menos de **1 segundo** en condiciones normales.

### 2. Disponibilidad
- **RNF-005**: El sistema debe tener un **SLA de disponibilidad del 99.9%**, medido mensualmente.
- **RNF-006**: El tiempo de inactividad planificado no debe exceder **4 horas por trimestre**.
- **RNF-007**: En caso de fallo de un componente crítico (ej. motor antifraude), el sistema debe degradar funcionalidades de manera controlada y notificar al equipo de operaciones.

### 3. Escalabilidad
- **RNF-008**: El sistema debe escalar horizontalmente para manejar picos de carga de hasta **3,000 solicitudes por segundo**.
- **RNF-009**: Los componentes stateless (ej. API Gateway, servicios de validación) deben escalarse automáticamente en función de la carga.
- **RNF-010**: Los componentes stateful (ej. bases de datos, colas de mensajes) deben escalarse verticalmente y usar réplicas para alta disponibilidad.

### 4. Seguridad
- **RNF-011**: Toda la comunicación entre componentes debe estar cifrada (TLS 1.2 o superior).
- **RNF-012**: Los datos sensibles (ej. números de identificación, información bancaria) deben encriptarse en tránsito y en reposo.
- **RNF-013**: El sistema debe implementar autenticación multifactor (MFA) para accesos administrativos.
- **RNF-014**: Debe cumplir con las regulaciones de protección de datos locales (ej. LGPD en Brasil, GDPR en Europa).
- **RNF-015**: Debe implementar controles de acceso basados en roles (RBAC) para todos los usuarios del sistema.

### 5. Resiliencia
- **RNF-016**: El sistema debe implementar mecanismos de reintento para integraciones con sistemas externos (ej. buró de riesgos, core bancario).
  - **Detalle**: Los reintentos deben seguir una política exponencial con un máximo de 3 intentos.
- **RNF-017**: Debe implementar circuit breakers para evitar cascadas de fallos en integraciones externas.
- **RNF-018**: Debe implementar un patrón de **Outbox** para garantizar la consistencia eventual en las transacciones con sistemas externos.
- **RNF-019**: Debe implementar un mecanismo de **dead letter queue (DLQ)** para manejar mensajes que no puedan procesarse después de múltiples reintentos.

### 6. Observabilidad
- **RNF-020**: El sistema debe instrumentar métricas en tiempo real para monitorear:
  - Latencia de las solicitudes.
  - Tasa de error en integraciones con sistemas externos.
  - Volumen de solicitudes procesadas.
  - Tiempo de procesamiento por etapa (validación, evaluación, desembolso).
- **RNF-021**: Debe implementar logging estructurado (JSON) para facilitar el análisis de logs.
- **RNF-022**: Debe integrarse con un sistema de monitoreo centralizado (ej. Prometheus + Grafana) para visualizar métricas y alertas.

### 7. Mantenibilidad
- **RNF-023**: El código debe seguir estándares de codificación y buenas prácticas del lenguaje utilizado.
- **RNF-024**: Debe implementarse un pipeline de CI/CD para automatizar pruebas, construcción y despliegue.
- **RNF-025**: Debe documentarse la arquitectura y los componentes clave del sistema (ver `arquitectura.md` y ADRs).

### 8. Portabilidad
- **RNF-026**: La solución debe diseñarse para ser desplegada en múltiples regiones geográficas, con soporte para configuraciones regionales (ej. regulaciones locales, monedas, idiomas).
- **RNF-027**: Los componentes deben ser containerizados (Docker) para facilitar el despliegue en diferentes entornos (desarrollo, pruebas, producción).

### 9. Usabilidad
- **RNF-028**: La interfaz de usuario (web y móvil) debe ser accesible y cumplir con estándares WCAG 2.1 AA.
- **RNF-029**: El tiempo de carga de las pantallas de captura de solicitud no debe exceder **1.5 segundos**.
- **RNF-030**: Debe implementarse un mecanismo de feedback para que los usuarios reporten problemas o sugerencias.

---

## Restricciones
- **R-001**: La solución debe integrarse con los sistemas legados del banco (core bancario, sistema de liquidación) sin modificarlos.
- **R-002**: El uso de servicios en la nube debe alinearse con las políticas de proveedor preferido del banco (ej. AWS, Azure, GCP).
- **R-003**: La solución debe cumplir con los estándares de seguridad y cumplimiento del banco (ej. ISO 27001, PCI DSS).
- **R-004**: El tiempo máximo para el despliegue de una nueva versión del sistema en producción no debe exceder **30 minutos**.

---

## Supuestos
- **S-001**: Los sistemas externos (motor antifraude, buró de riesgos, core bancario, gateway de pagos, sistema de liquidación) proporcionan APIs REST o servicios gRPC con contratos definidos.
- **S-002**: El banco ya tiene un proveedor de identidad (ej. Auth0, Okta) que puede integrarse para autenticación y autorización.
- **S-003**: El banco tiene un equipo de operaciones con experiencia en monitoreo y gestión de incidentes en entornos cloud.
- **S-004**: Los datos históricos de solicitudes de préstamo están disponibles para entrenar modelos de riesgo y antifraude.

---

## Dependencias
- **D-001**: Motor antifraude: Disponible 24/7 con un SLA de 99.95%.
- **D-002**: Buró de riesgos: Disponible 24/7 con un SLA de 99.9% y un límite de 1,000 solicitudes por segundo.
- **D-003**: Core bancario: Disponible 24/7 con un SLA de 99.9% y un límite de 500 transacciones por segundo.
- **D-004**: Gateway de pagos: Disponible 24/7 con un SLA de 99.9% y un límite de 200 transacciones por segundo.
- **D-005**: Sistema de liquidación: Disponible 22/5 con un SLA de 99.8% y un límite de 300 transacciones por segundo.

---

## Glosario
- **Originador de Créditos**: Componente responsable de capturar, validar y procesar solicitudes de préstamo.
- **Motor Antifraude**: Sistema externo que evalúa el riesgo de fraude en las solicitudes.
- **Buró de Riesgos**: Sistema externo que proporciona información crediticia del solicitante.
- **Core Bancario**: Sistema central del banco que gestiona cuentas y transacciones.
- **Gateway de Pagos**: Sistema que procesa transferencias de fondos entre cuentas.
- **Sistema de Liquidación**: Sistema que registra y concilia transacciones financieras.
- **Outbox**: Patrón que garantiza la consistencia eventual en transacciones distribuidas.
- **Dead Letter Queue (DLQ)**: Cola para mensajes que no pudieron procesarse después de múltiples reintentos.