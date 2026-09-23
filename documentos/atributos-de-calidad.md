# Atributos de Calidad para la Solución de Originación Digital de Préstamos

## Introducción
Este documento define los atributos de calidad clave para la solución de originación digital de préstamos, utilizando el formato de escenario, métrica y umbral. Los atributos se priorizan en función de su impacto en la experiencia del usuario, la estabilidad del sistema y el cumplimiento de los objetivos del negocio.

---

## 1. Disponibilidad

### Escenario
Un usuario intenta acceder al sistema para iniciar una solicitud de préstamo durante un día laboral típico. El sistema debe estar disponible para procesar la solicitud sin interrupciones no planificadas.

### Métrica
- **Tiempo de actividad (Uptime)**: Porcentaje de tiempo que el sistema está disponible para procesar solicitudes durante un período de medición (mensual).
- **Tiempo de inactividad (Downtime)**: Tiempo total de inactividad no planificado en un mes.

### Umbral
- **Uptime**: ≥ 99.9% mensual.
- **Downtime**: ≤ 43.2 minutos por mes.
- **Impacto**: Si el uptime cae por debajo del 99.9%, se activa un plan de contingencia que incluye notificaciones al equipo de operaciones y degradación controlada de funcionalidades no críticas.

### Factores de Riesgo
- Fallos en sistemas externos (ej. motor antifraude, buró de riesgos).
- Problemas de red entre componentes.
- Sobrecarga del sistema durante picos de tráfico.

### Estrategias de Mitigación
- Implementación de circuit breakers para evitar cascadas de fallos.
- Réplicas de componentes críticos en múltiples zonas de disponibilidad.
- Monitoreo proactivo con alertas tempranas para fallos potenciales.

---

## 2. Latencia

### Escenario
Un usuario envía una solicitud de préstamo y espera recibir una respuesta en tiempo real. La latencia percibida por el usuario no debe superar los umbrales definidos.

### Métricas
- **Latencia de extremo a extremo**: Tiempo total desde que el usuario envía la solicitud hasta que recibe la respuesta (generación de oferta).
- **Latencia por etapa**:
  - Validación de identidad y antifraude: ≤ 500 ms.
  - Evaluación de capacidad de endeudamiento: ≤ 800 ms.
  - Generación de oferta: ≤ 700 ms.
  - Integración con sistemas externos: ≤ 1,000 ms (total acumulado).

### Umbrales
- **Latencia de extremo a extremo**: ≤ 2 segundos para el 95% de las solicitudes.
- **Latencia máxima**: ≤ 5 segundos para el 100% de las solicitudes.
- **Impacto**: Si la latencia supera los 2 segundos para más del 5% de las solicitudes, se activa una revisión de rendimiento para identificar cuellos de botella.

### Factores de Riesgo
- Sobrecarga de sistemas externos (ej. buró de riesgos).
- Alta carga en componentes internos (ej. base de datos).
- Problemas de red entre regiones.

### Estrategias de Mitigación
- Implementación de caching para respuestas frecuentes (ej. validación de identidad).
- Optimización de consultas a bases de datos y sistemas externos.
- Escalado horizontal de componentes stateless durante picos de carga.

---

## 3. Throughput

### Escenario
El sistema experimenta un pico de tráfico durante la hora del almuerzo, con un volumen de 1,500 solicitudes por segundo. El sistema debe procesar todas las solicitudes dentro de los umbrales de latencia definidos.

### Métrica
- **Solicitudes por segundo (RPS)**: Número de solicitudes procesadas por segundo.
- **Tasa de error**: Porcentaje de solicitudes que fallan debido a sobrecarga o errores internos.

### Umbrales
- **Throughput**: ≥ 1,500 solicitudes por segundo en hora pico.
- **Tasa de error**: ≤ 0.1% durante picos de carga.
- **Impacto**: Si el throughput cae por debajo de 1,500 RPS o la tasa de error supera el 0.1%, se activa el escalado automático de componentes y se notifica al equipo de operaciones.

### Factores de Riesgo
- Límites de tasa en sistemas externos (ej. buró de riesgos).
- Cuellos de botella en componentes internos (ej. base de datos).
- Fallos en colas de mensajes (ej. Kafka, RabbitMQ).

### Estrategias de Mitigación
- Implementación de colas de mensajes para desacoplar componentes y manejar picos de carga.
- Escalado horizontal de componentes stateless (ej. servicios de validación).
- Monitoreo en tiempo real del throughput y la tasa de error para activar escalado automático.

---

## 4. Escalabilidad

### Escenario
El banco lanza una campaña de préstamos con tasas preferenciales, lo que genera un aumento repentino en el volumen de solicitudes (hasta 3,000 RPS). El sistema debe escalar para manejar este aumento sin degradar el rendimiento.

### Métrica
- **Capacidad máxima**: Número máximo de solicitudes por segundo que el sistema puede manejar sin degradar el rendimiento.
- **Tiempo de escalado**: Tiempo requerido para escalar componentes (horizontal o verticalmente) durante un pico de carga.

### Umbrales
- **Capacidad máxima**: ≥ 3,000 solicitudes por segundo.
- **Tiempo de escalado**: ≤ 2 minutos para componentes stateless.
- **Impacto**: Si la capacidad máxima es insuficiente para manejar el pico, se degrada la funcionalidad de generación de ofertas personalizadas y se notifica al equipo de operaciones.

### Factores de Riesgo
- Límites de escalado en componentes stateful (ej. bases de datos).
- Dependencia de sistemas externos con límites de tasa.
- Falta de automatización en el escalado de componentes.

### Estrategias de Mitigación
- Diseño de componentes stateless para facilitar el escalado horizontal.
- Uso de réplicas para componentes stateful (ej. bases de datos).
- Implementación de autoscaling basado en métricas de carga (CPU, memoria, RPS).

---

## 5. Seguridad

### Escenario
Un atacante intenta acceder a información sensible de los usuarios (ej. números de identificación, datos bancarios) mediante un ataque de inyección de SQL o interceptación de tráfico. El sistema debe proteger los datos y garantizar su confidencialidad e integridad.

### Métricas
- **Tasa de incidentes de seguridad**: Número de incidentes de seguridad reportados por mes.
- **Tiempo de detección**: Tiempo promedio para detectar un incidente de seguridad.
- **Tiempo de respuesta**: Tiempo promedio para mitigar un incidente de seguridad.

### Umbrales
- **Tasa de incidentes**: ≤ 1 incidente por trimestre.
- **Tiempo de detección**: ≤ 15 minutos.
- **Tiempo de respuesta**: ≤ 1 hora.
- **Impacto**: Si se detecta un incidente de seguridad, se activa un protocolo de respuesta que incluye notificación al equipo de seguridad, aislamiento del componente afectado y análisis forense.

### Factores de Riesgo
- Vulnerabilidades en dependencias de terceros.
- Configuraciones incorrectas de seguridad (ej. permisos excesivos).
- Falta de cifrado en datos sensibles.

### Estrategias de Mitigación
- Implementación de cifrado en tránsito (TLS 1.2+) y en reposo (AES-256).
- Uso de autenticación multifactor (MFA) para accesos administrativos.
- Escaneo regular de vulnerabilidades en el código y las dependencias.
- Implementación de controles de acceso basados en roles (RBAC).

---

## 6. Resiliencia

### Escenario
Un componente crítico del sistema (ej. motor antifraude) falla durante un pico de carga. El sistema debe continuar operando en modo degradado y recuperarse automáticamente una vez que el componente vuelva a estar disponible.

### Métricas
- **Tiempo medio entre fallos (MTBF)**: Tiempo promedio entre fallos de un componente.
- **Tiempo medio de recuperación (MTTR)**: Tiempo promedio para recuperar un componente después de un fallo.
- **Tasa de fallos**: Porcentaje de solicitudes que fallan debido a fallos en componentes críticos.

### Umbrales
- **MTBF**: ≥ 720 horas (30 días) para componentes críticos.
- **MTTR**: ≤ 5 minutos para componentes con réplicas.
- **Tasa de fallos**: ≤ 0.01% para solicitudes en modo degradado.
- **Impacto**: Si el MTTR supera los 5 minutos o la tasa de fallos supera el 0.01%, se activa una revisión de diseño para mejorar la resiliencia del componente.

### Factores de Riesgo
- Dependencia de componentes externos sin redundancia.
- Falta de mecanismos de reintento y circuit breakers.
- Fallos en colas de mensajes o bases de datos.

### Estrategias de Mitigación
- Implementación de circuit breakers para evitar cascadas de fallos.
- Uso de colas de mensajes (ej. Kafka, RabbitMQ) para garantizar la entrega de mensajes.
- Réplicas de componentes críticos en múltiples zonas de disponibilidad.
- Implementación de patrones de consistencia eventual (ej. Outbox) para transacciones distribuidas.

---

## 7. Observabilidad

### Escenario
El equipo de operaciones necesita monitorear el estado del sistema en tiempo real para detectar y resolver problemas antes de que afecten a los usuarios. El sistema debe proporcionar métricas, logs y trazas accesibles y procesables.

### Métricas
- **Cobertura de métricas**: Porcentaje de componentes que emiten métricas en tiempo real.
- **Cobertura de logs**: Porcentaje de componentes que emiten logs estructurados.
- **Cobertura de trazas**: Porcentaje de solicitudes que generan trazas distribuidas.

### Umbrales
- **Cobertura de métricas**: 100% para componentes críticos.
- **Cobertura de logs**: 100% para todos los componentes.
- **Cobertura de trazas**: ≥ 90% para solicitudes de extremo a extremo.
- **Impacto**: Si la cobertura de métricas, logs o trazas cae por debajo de los umbrales, se activa una revisión para identificar componentes no instrumentados.

### Factores de Riesgo
- Falta de instrumentación en componentes legacy.
- Logs no estructurados que dificultan el análisis.
- Falta de correlación entre trazas de diferentes componentes.

### Estrategias de Mitigación
- Instrumentación de todos los componentes con métricas, logs y trazas.
- Uso de herramientas de monitoreo centralizado (ej. Prometheus, Grafana, ELK Stack).
- Implementación de IDs de correlación para trazas distribuidas.

---

## 8. Mantenibilidad

### Escenario
El equipo de desarrollo necesita realizar cambios en el sistema (ej. agregar una nueva validación en el flujo de originación) sin introducir errores o degradar el rendimiento. El sistema debe ser fácil de entender, modificar y probar.

### Métricas
- **Complejidad ciclomática**: Medida de la complejidad del código.
- **Cobertura de pruebas**: Porcentaje de código cubierto por pruebas automatizadas.
- **Tiempo de implementación**: Tiempo promedio para implementar un cambio en producción.

### Umbrales
- **Complejidad ciclomática**: ≤ 10 para métodos críticos.
- **Cobertura de pruebas**: ≥ 80% para código crítico, ≥ 60% para el resto.
- **Tiempo de implementación**: ≤ 1 día para cambios menores, ≤ 5 días para cambios mayores.
- **Impacto**: Si la complejidad ciclomática supera 10 o la cobertura de pruebas cae por debajo de los umbrales, se activa una revisión de código para refactorizar y agregar pruebas.

### Factores de Riesgo
- Código monolítico sin modularización.
- Falta de documentación y comentarios claros.
- Pruebas manuales en lugar de automatizadas.

### Estrategias de Mitigación
- Diseño modular con componentes desacoplados.
- Implementación de pruebas automatizadas (unitarias, integración, e2e).
- Uso de pipelines de CI/CD para validar cambios antes de desplegarlos en producción.

---

## Priorización de Atributos de Calidad
| Atributo          | Prioridad | Justificación                                                                                     |
|-------------------|-----------|---------------------------------------------------------------------------------------------------|
| Disponibilidad    | Alta      | Es crítico para la experiencia del usuario y el cumplimiento del SLA.                            |
| Latencia          | Alta      | Impacta directamente en la satisfacción del usuario y la conversión de solicitudes.             |
| Throughput        | Alta      | Es esencial para manejar picos de carga y garantizar la escalabilidad del negocio.               |
| Seguridad         | Alta      | Protege datos sensibles y cumple con regulaciones legales.                                       |
| Resiliencia       | Media     | Garantiza la continuidad del servicio incluso durante fallos.                                    |
| Observabilidad    | Media     | Permite detectar y resolver problemas rápidamente.                                               |
| Escalabilidad     | Media     | Prepara al sistema para el crecimiento futuro.                                                    |
| Mantenibilidad    | Baja      | Importante para la agilidad del equipo, pero no crítica para el funcionamiento inmediato.         |

---

## Herramientas Recomendadas
- **Monitoreo**: Prometheus + Grafana.
- **Logs**: ELK Stack (Elasticsearch, Logstash, Kibana).
- **Trazas**: Jaeger o Zipkin.
- **Alertas**: PagerDuty o Opsgenie.
- **Pruebas**: JUnit (Java), pytest (Python), Jest (TypeScript), Selenium (e2e).
- **CI/CD**: Jenkins, GitHub Actions, GitLab CI.

---

## Glosario
- **Disponibilidad**: Capacidad del sistema para estar operativo y accesible cuando se lo necesita.
- **Latencia**: Tiempo que tarda el sistema en responder a una solicitud.
- **Throughput**: Número de solicitudes que el sistema puede procesar por unidad de tiempo.
- **Escalabilidad**: Capacidad del sistema para manejar un aumento en la carga sin degradar el rendimiento.
- **Seguridad**: Protección de los datos y el sistema contra accesos no autorizados o ataques.
- **Resiliencia**: Capacidad del sistema para recuperarse de fallos y continuar operando.
- **Observabilidad**: Capacidad de monitorear el estado del sistema mediante métricas, logs y trazas.
- **Mantenibilidad**: Facilidad con la que el sistema puede ser modificado, probado y desplegado.