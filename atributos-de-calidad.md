# Atributos de Calidad

## Introducción
Este documento define los atributos de calidad críticos para la solución de originación digital de préstamos, junto con sus escenarios, métricas y umbrales. Los atributos se priorizan según su impacto en los objetivos del negocio y los requerimientos no funcionales.

---

## Atributos de Calidad

### 1. Disponibilidad
**Escenario**: El sistema debe estar disponible para procesar solicitudes de préstamos durante el horario de operación del banco (24/7), con excepción de ventanas de mantenimiento programadas.
**Métrica**: Porcentaje de tiempo de disponibilidad mensual.
**Umbral**: 99.9% (equivalente a ~43.2 minutos de downtime no programado por mes).
**Justificación**: Un SLA de 99.9% es estándar para sistemas financieros críticos, donde la indisponibilidad puede generar pérdidas económicas y daño reputacional.

---

### 2. Latencia
**Escenario**: El sistema debe procesar el 95% de las solicitudes de préstamos en menos de 2 segundos durante horas pico (1,500 solicitudes por segundo).
**Métrica**: Tiempo de respuesta percentil 95 (P95) para solicitudes exitosas.
**Umbral**: ≤ 2 segundos.
**Justificación**: Una latencia superior a 2 segundos afecta la experiencia del usuario y aumenta la tasa de abandono de solicitudes. El umbral se basa en benchmarks de la industria para sistemas de originación digital.

---

### 3. Throughput
**Escenario**: El sistema debe soportar un volumen pico de 1,500 solicitudes por segundo durante horas de máxima demanda.
**Métrica**: Número de solicitudes procesadas por segundo (RPS).
**Umbral**: ≥ 1,500 RPS.
**Justificación**: El volumen pico se calculó en base a proyecciones de crecimiento del banco y patrones históricos de demanda. Superar este umbral garantiza que el sistema no se convierta en un cuello de botella.

---

### 4. Escalabilidad
**Escenario**: El sistema debe escalar horizontalmente para manejar aumentos repentinos en la demanda sin degradación de desempeño.
**Métrica**: Tiempo requerido para escalar horizontalmente (ej. agregar instancias adicionales).
**Umbral**: ≤ 5 minutos para escalar a 2x la capacidad actual.
**Justificación**: La escalabilidad horizontal permite manejar picos de demanda sin sobre-aprovisionamiento, optimizando costos. El umbral de 5 minutos se alinea con las capacidades de auto-scaling de plataformas cloud como AWS.

---

### 5. Seguridad
**Escenario**: El sistema debe proteger los datos sensibles de los clientes (ej. información personal y financiera) contra accesos no autorizados y ataques externos.
**Métrica**: Número de incidentes de seguridad reportados y tiempo promedio de resolución.
**Umbral**:
- 0 incidentes de fuga de datos por año.
- Tiempo de resolución ≤ 1 hora para incidentes críticos.
**Justificación**: La seguridad es un requisito legal y regulatorio (ej. GDPR, LGPD). Cualquier incidente puede resultar en multas y pérdida de confianza del cliente.

---

### 6. Resiliencia
**Escenario**: El sistema debe recuperarse automáticamente de fallos parciales (ej. caída de un servicio externo como el buró de riesgos) sin afectar la disponibilidad global.
**Métrica**: Tiempo medio de recuperación (MTTR) y tasa de éxito en fallos parciales.
**Umbral**:
- MTTR ≤ 30 segundos.
- Tasa de éxito ≥ 99.9% para fallos parciales.
**Justificación**: La resiliencia garantiza que el sistema continúe operando incluso ante fallos de componentes individuales, cumpliendo con el SLA de disponibilidad.

---

### 7. Consistencia
**Escenario**: El sistema debe garantizar que los datos críticos (ej. estado de una solicitud de préstamo) sean consistentes en todos los componentes, incluso en presencia de fallos.
**Métrica**: Número de inconsistencias detectadas en auditorías.
**Umbral**: 0 inconsistencias en datos críticos por mes.
**Justificación**: La inconsistencia en datos financieros puede llevar a decisiones erróneas (ej. aprobar un préstamo a un cliente con historial de impagos). El umbral refleja la tolerancia cero para errores en este contexto.

---

### 8. Auditabilidad
**Escenario**: El sistema debe registrar todas las acciones críticas (ej. aprobación/rechazo de préstamos, cambios en datos del cliente) para cumplir con requisitos regulatorios.
**Métrica**: Porcentaje de acciones críticas registradas y accesibles para auditoría.
**Umbral**: 100% de acciones críticas registradas con trazabilidad completa.
**Justificación**: La auditabilidad es un requisito legal para sistemas financieros. La trazabilidad completa permite reconstruir eventos en caso de disputas o auditorías.

---

### 9. Mantenibilidad
**Escenario**: El sistema debe permitir modificaciones y correcciones con un esfuerzo mínimo, facilitando la incorporación de nuevos requisitos.
**Métrica**: Tiempo promedio para implementar un cambio (ej. agregar un nuevo campo a la solicitud de préstamo).
**Umbral**: ≤ 2 días para cambios de complejidad media.
**Justificación**: La mantenibilidad reduce el costo total de propiedad (TCO) del sistema y acelera la entrega de valor al negocio.

---

### 10. Interoperabilidad
**Escenario**: El sistema debe integrarse sin problemas con sistemas externos (ej. buró de riesgos, core bancario, gateway de pagos) usando estándares abiertos.
**Métrica**: Número de integraciones exitosas sin necesidad de adaptadores personalizados.
**Umbral**: 100% de integraciones usando estándares abiertos (ej. REST, AsyncAPI, OpenAPI).
**Justificación**: La interoperabilidad reduce la dependencia de proveedores específicos y facilita la incorporación de nuevos socios tecnológicos.

---

## Matriz de Priorización
| Atributo          | Prioridad | Impacto en Negocio          | Riesgo si no se Cumple                     |
|-------------------|-----------|-----------------------------|--------------------------------------------|
| Disponibilidad    | Alta      | Pérdida de ingresos         | Multas regulatorias, daño reputacional     |
| Latencia          | Alta      | Experiencia del usuario     | Aumento en tasa de abandono                |
| Throughput        | Alta      | Capacidad de procesamiento  | Cuellos de botella, saturación del sistema  |
| Seguridad         | Alta      | Protección de datos         | Multas, pérdida de confianza               |
| Resiliencia       | Media     | Continuidad del servicio    | Downtime no programado                     |
| Consistencia      | Media     | Precisión de datos          | Decisiones erróneas, fraudes               |
| Auditabilidad     | Media     | Cumplimiento legal          | Sanciones regulatorias                     |
| Escalabilidad     | Media     | Crecimiento futuro          | Sobrecostos en infraestructura             |
| Mantenibilidad    | Baja      | Tiempo de desarrollo        | Mayor costo de cambios                     |
| Interoperabilidad | Baja      | Flexibilidad                | Dependencia de proveedores                 |

---

## Ejemplo de Escenario Detallado: Latencia
**Contexto**:
Durante horas pico, el sistema recibe un promedio de 1,500 solicitudes por segundo. Cada solicitud requiere consultar múltiples servicios externos (buró de riesgos, motor antifraude) y procesar lógica de negocio compleja (calificación crediticia, reglas de aprobación).

**Secuencia de Eventos**:
1. El usuario envía una solicitud de préstamo desde la app móvil.
2. El sistema valida los datos básicos de la solicitud.
3. Se consulta el buró de riesgos para obtener el historial crediticio del cliente.
4. Se ejecuta el motor antifraude para detectar posibles fraudes.
5. Se calcula la calificación crediticia y se aplican reglas de aprobación.
6. Se actualiza el estado de la solicitud en la base de datos.
7. Se notifica al usuario el resultado de la solicitud.

**Métricas y Umbrales**:
- **Tiempo de respuesta P95**: ≤ 2 segundos.
- **Tiempo de respuesta máximo**: ≤ 5 segundos.
- **Tasa de éxito**: ≥ 99% (excluyendo errores del cliente).

**Riesgos**:
- **Riesgo 1**: Consultas bloqueantes a servicios externos (ej. buró de riesgos) aumentan la latencia.
  - **Mitigación**: Implementar timeouts y circuit breakers para evitar bloqueos.
- **Riesgo 2**: Procesamiento secuencial de reglas de negocio incrementa el tiempo de respuesta.
  - **Mitigación**: Paralelizar consultas a servicios externos y procesamiento de reglas.
- **Riesgo 3**: Sobrecarga de la base de datos durante horas pico.
  - **Mitigación**: Usar caché para datos frecuentes y optimizar consultas.

---

## Conclusión
Los atributos de calidad definidos en este documento son críticos para el éxito de la solución de originación digital. Cada atributo se ha priorizado según su impacto en el negocio y se ha acompañado de métricas y umbrales medibles para garantizar su cumplimiento. Estos atributos servirán como base para las decisiones de diseño y las evaluaciones de la solución en las fases posteriores.