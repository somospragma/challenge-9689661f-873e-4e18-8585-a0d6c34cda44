# Riesgos y Mitigaciones

## Introducción
Este documento identifica los riesgos técnicos y operativos asociados a la solución de originación digital de préstamos, priorizados según su impacto y probabilidad. Para cada riesgo, se propone una estrategia de mitigación alineada con los atributos de calidad definidos anteriormente.

---

## Metodología de Priorización
Los riesgos se priorizan usando una matriz de impacto/probabilidad con las siguientes escalas:
- **Impacto**: Alto (3), Medio (2), Bajo (1).
- **Probabilidad**: Alta (3), Media (2), Baja (1).
- **Prioridad**: Impacto × Probabilidad (Alto: 6-9, Medio: 3-5, Bajo: 1-2).

---

## Riesgos Identificados

### 1. Caída de Servicios Externos (Buró de Riesgos, Motor Antifraude)
**Descripción**: Los servicios externos (buró de riesgos, motor antifraude) pueden experimentar downtime o alta latencia, afectando la disponibilidad y latencia del sistema.
**Impacto**: Alto (3) – El sistema no puede procesar solicitudes sin estos servicios.
**Probabilidad**: Media (2) – Los servicios externos tienen SLAs de 99.9%, pero fallos ocasionales son posibles.
**Prioridad**: 6 (Alto).
**Mitigaciones**:
- **Circuit Breaker**: Implementar patrones de circuit breaker (ej. Resilience4j) para evitar llamadas repetidas a servicios caídos.
- **Retry con Backoff Exponencial**: Reintentar llamadas fallidas con backoff exponencial para reducir la carga en servicios recuperándose.
- **Cache Local**: Almacenar respuestas recientes del buró de riesgos y motor antifraude para reducir dependencia en tiempo real.
- **Fallback**: Usar datos históricos o reglas simplificadas como fallback cuando los servicios externos no respondan.

**Métricas de Éxito**:
- Tiempo medio de recuperación (MTTR) ≤ 30 segundos.
- Tasa de éxito en fallos parciales ≥ 99.9%.

---

### 2. Sobrecarga del Sistema durante Horas Pico
**Descripción**: El sistema puede saturarse durante horas pico (1,500 solicitudes por segundo), degradando el desempeño y aumentando la latencia.
**Impacto**: Alto (3) – Afecta la experiencia del usuario y puede incumplir el SLA de latencia.
**Probabilidad**: Alta (3) – La demanda pico es predecible y recurrente.
**Prioridad**: 9 (Alto).
**Mitigaciones**:
- **Escalado Horizontal Automático**: Usar auto-scaling en la capa de aplicación para manejar picos de demanda.
- **Colas de Mensajes**: Implementar colas (ej. Kafka, SQS) para desacoplar componentes y manejar picos de carga.
- **Rate Limiting**: Limitar la tasa de solicitudes por cliente/IP para evitar abusos.
- **Optimización de Consultas**: Caché de datos frecuentes y optimización de consultas a la base de datos.

**Métricas de Éxito**:
- Throughput ≥ 1,500 RPS durante horas pico.
- Latencia P95 ≤ 2 segundos durante horas pico.

---

### 3. Inconsistencia de Datos entre Componentes
**Descripción**: Fallos en la comunicación entre componentes (ej. originador de créditos y core bancario) pueden llevar a inconsistencias en los datos (ej. estado de una solicitud).
**Impacto**: Medio (2) – Puede generar errores en procesos posteriores (ej. liquidación de préstamos).
**Probabilidad**: Media (2) – Fallos en la red o en servicios internos pueden ocurrir ocasionalmente.
**Prioridad**: 4 (Medio).
**Mitigaciones**:
- **Saga Pattern**: Implementar transacciones distribuidas usando el patrón Saga para garantizar consistencia eventual.
- **Event Sourcing**: Registrar todos los cambios de estado como eventos para permitir replay y reconciliación.
- **Idempotencia**: Diseñar APIs idempotentes para evitar duplicados en reintentos.
- **Monitoreo de Consistencia**: Implementar checks periódicos para detectar y corregir inconsistencias.

**Métricas de Éxito**:
- 0 inconsistencias en datos críticos por mes.

---

### 4. Ataques de Seguridad (DDoS, Inyección de Datos)
**Descripción**: Ataques externos pueden comprometer la seguridad del sistema (ej. DDoS, inyección de SQL, robo de datos).
**Impacto**: Alto (3) – Puede resultar en multas regulatorias y pérdida de confianza del cliente.
**Probabilidad**: Media (2) – Los sistemas financieros son blancos frecuentes de ataques.
**Prioridad**: 6 (Alto).
**Mitigaciones**:
- **WAF (Web Application Firewall)**: Implementar un WAF para filtrar tráfico malicioso.
- **Autenticación y Autorización**: Usar OAuth2/OIDC para autenticación y RBAC para autorización.
- **Validación de Datos**: Validar todas las entradas de usuario para evitar inyecciones.
- **Cifrado**: Cifrar datos en tránsito (TLS) y en reposo (AES-256).
- **Monitoreo de Seguridad**: Implementar SIEM (ej. AWS GuardDuty) para detectar y responder a amenazas.

**Métricas de Éxito**:
- 0 incidentes de fuga de datos por año.
- Tiempo de resolución ≤ 1 hora para incidentes críticos.

---

### 5. Fallos en la Integración con el Core Bancario
**Descripción**: El core bancario puede fallar o responder con alta latencia, bloqueando la liquidación de préstamos.
**Impacto**: Alto (3) – Sin liquidación, los préstamos no se pueden desembolsar.
**Probabilidad**: Baja (1) – El core bancario tiene SLAs estrictos y redundancia.
**Prioridad**: 3 (Medio).
**Mitigaciones**:
- **Timeouts y Circuit Breaker**: Implementar timeouts y circuit breakers para evitar bloqueos.
- **Cola de Liquidación**: Usar una cola para desacoplar la originación de la liquidación.
- **Fallback Manual**: Permitir intervención manual para liquidar préstamos en casos excepcionales.
- **Monitoreo de Integración**: Monitorear la latencia y disponibilidad del core bancario.

**Métricas de Éxito**:
- Tasa de éxito en liquidaciones ≥ 99.9%.
- Tiempo medio de liquidación ≤ 5 minutos.

---

### 6. Errores en Reglas de Negocio
**Descripción**: Reglas de negocio mal implementadas (ej. cálculo de calificación crediticia) pueden llevar a aprobaciones incorrectas de préstamos.
**Impacto**: Medio (2) – Puede generar pérdidas financieras para el banco.
**Probabilidad**: Baja (1) – Las reglas se prueban exhaustivamente antes de desplegar.
**Prioridad**: 2 (Bajo).
**Mitigaciones**:
- **Pruebas Automatizadas**: Implementar pruebas unitarias y de integración para validar reglas de negocio.
- **Revisión por Pares**: Requerir revisión por pares para cambios en reglas críticas.
- **Validación en Tiempo Real**: Validar reglas durante el procesamiento de solicitudes.
- **Auditoría de Reglas**: Registrar todas las decisiones tomadas por reglas de negocio para auditoría.

**Métricas de Éxito**:
- 0 errores en reglas de negocio detectados en producción por trimestre.

---

### 7. Falta de Capacidad de la Base de Datos
**Descripción**: La base de datos puede saturarse durante horas pico, afectando el desempeño del sistema.
**Impacto**: Medio (2) – Puede aumentar la latencia y reducir el throughput.
**Probabilidad**: Media (2) – La demanda pico es alta y recurrente.
**Prioridad**: 4 (Medio).
**Mitigaciones**:
- **Particionamiento**: Particionar tablas críticas (ej. solicitudes de préstamos) por fecha/región.
- **Réplicas de Lectura**: Usar réplicas de lectura para distribuir la carga.
- **Caché**: Implementar caché para consultas frecuentes (ej. Redis).
- **Optimización de Índices**: Optimizar índices y consultas para reducir la carga en la base de datos.

**Métricas de Éxito**:
- Latencia de consultas a la base de datos ≤ 100ms durante horas pico.
- Throughput de la base de datos ≥ 2,000 operaciones por segundo.

---

## Matriz de Riesgos Priorizados

| Riesgo                                      | Impacto | Probabilidad | Prioridad | Estado       |
|---------------------------------------------|---------|--------------|-----------|--------------|
| Caída de Servicios Externos                 | Alto    | Media        | Alto       | En Mitigación|
| Sobrecarga del Sistema durante Horas Pico   | Alto    | Alta         | Alto       | En Mitigación|
| Inconsistencia de Datos entre Componentes   | Medio   | Media        | Medio      | En Mitigación|
| Ataques de Seguridad                        | Alto    | Media        | Alto       | En Mitigación|
| Fallos en la Integración con el Core Bancario| Alto   | Baja         | Medio      | En Mitigación|
| Errores en Reglas de Negocio                | Medio   | Baja         | Bajo       | En Mitigación|
| Falta de Capacidad de la Base de Datos      | Medio   | Media        | Medio      | En Mitigación|

---

## Conclusión
Los riesgos identificados en este documento representan las principales amenazas para el éxito de la solución de originación digital. Las estrategias de mitigación propuestas se alinean con los atributos de calidad definidos anteriormente y buscan minimizar el impacto de estos riesgos. Este documento servirá como base para las decisiones de diseño y las evaluaciones de la solución en las fases posteriores.