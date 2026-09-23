# Trade-offs Clave en la Solución de Originación Digital

## Introducción
Este documento resume los trade-offs críticos identificados durante el diseño de la solución de originación digital de préstamos. Cada trade-off se analiza en términos de su impacto en los atributos de calidad definidos en `documentos/atributos-de-calidad.md`, como disponibilidad, latencia, throughput, consistencia y costo. Las decisiones asociadas se registran en los ADRs correspondientes (`adr/001-decision-arquitectura-base.md`, `adr/002-decision-manejo-carga.md`, etc.).

---

## 1. Consistencia vs. Disponibilidad

### Contexto
La solución interactúa con múltiples sistemas externos (motor antifraude, buró de riesgos, core bancario) que pueden tener diferentes niveles de disponibilidad y consistencia. El requisito no funcional exige un SLA de 99.9% de disponibilidad, mientras que el negocio requiere que las decisiones de aprobación/rechazo de préstamos sean consistentes y auditables.

### Trade-off
- **Opción A: Consistencia fuerte (sincronía)**
  - **Descripción**: Invocar todos los sistemas externos de manera síncrona antes de responder al cliente. Garantizar que todas las validaciones se completen antes de aprobar/rechazar una solicitud.
  - **Impacto en atributos de calidad**:
    - **Disponibilidad**: Degradación significativa si alguno de los sistemas externos falla o tiene alta latencia. Riesgo de incumplir el SLA del 99.9%.
    - **Latencia**: Aumento de la latencia percibida por el cliente (potencialmente >2 segundos en hora pico).
    - **Throughput**: Reducción del throughput debido a la espera activa por respuestas de sistemas externos.
    - **Costo**: Mayor costo operativo por la necesidad de escalar horizontalmente para compensar la latencia.
  - **Ejemplo cuantificable**: Si el motor antifraude tiene una latencia de 500ms y el buró de riesgos 800ms, el tiempo total de respuesta superaría los 2 segundos en el 30% de los casos.

- **Opción B: Disponibilidad alta (asincronía con compensación)**
  - **Descripción**: Procesar la solicitud de manera asíncrona, respondiendo al cliente inmediatamente con un estado "en proceso" y realizando las validaciones en segundo plano. Usar patrones como Sagas o Outbox para manejar la consistencia eventual.
  - **Impacto en atributos de calidad**:
    - **Disponibilidad**: Mejora significativa, ya que el sistema puede responder incluso si los sistemas externos están caídos. Cumplimiento del SLA del 99.9%.
    - **Latencia**: Reducción drástica (respuesta en <200ms).
    - **Throughput**: Aumento del throughput, ya que el sistema no bloquea recursos esperando respuestas.
    - **Consistencia**: Riesgo de inconsistencias temporales (ej., un préstamo aprobado inicialmente pero rechazado posteriormente por el buró de riesgos).
    - **Costo**: Reducción de costos operativos al requerir menos escalado horizontal.
  - **Ejemplo cuantificable**: Throughput aumenta de 800 a 1,500 solicitudes/segundo al eliminar bloqueos.

### Decisión
**Se adopta la Opción B (asincronía con compensación)**, registrada en `adr/003-decision-sincronia-asincronia.md`. La decisión prioriza la disponibilidad y el throughput sobre la consistencia fuerte, mitigando el riesgo de inconsistencias con:
- **Patrón Outbox**: Publicar eventos de dominio (ej., "SolicitudCreada") en una cola transaccional para garantizar que las validaciones posteriores se ejecuten.
- **Sagas**: Coordinar las validaciones entre sistemas externos mediante compensaciones (ej., revertir una aprobación si el buró de riesgos la rechaza).
- **Mecanismos de retry**: Reintentar validaciones fallidas con backoff exponencial.

### Consecuencias
- **Positivas**:
  - Cumplimiento del SLA del 99.9% y el throughput de 1,500 solicitudes/segundo.
  - Reducción de la latencia percibida por el cliente.
  - Menor costo operativo.
- **Negativas**:
  - Complejidad adicional en el diseño para manejar consistencia eventual.
  - Necesidad de implementar mecanismos de compensación y retry.
  - Riesgo de decisiones inconsistentes (mitigado con auditorías y reconciliaciones periódicas).

---

## 2. Sincronía vs. Asincronía en Integraciones

### Contexto
Las integraciones con sistemas externos (ej., buró de riesgos, core bancario) pueden implementarse de manera síncrona (REST/gRPC) o asíncrona (eventos mediante colas). El requisito de latencia máxima de 2 segundos y el throughput de 1,500 solicitudes/segundo influyen en la elección.

### Trade-off
- **Opción A: Integraciones síncronas (REST/gRPC)**
  - **Descripción**: Invocar los sistemas externos directamente mediante llamadas HTTP/gRPC y esperar su respuesta antes de continuar.
  - **Impacto en atributos de calidad**:
    - **Latencia**: Acumulación de latencias individuales (ej., 500ms + 800ms + 300ms = 1.6s).
    - **Disponibilidad**: Dependencia crítica de la disponibilidad de cada sistema externo.
    - **Throughput**: Limitado por la capacidad de los sistemas externos.
    - **Simplicidad**: Implementación más sencilla y debugging más fácil.
  - **Ejemplo cuantificable**: Si el core bancario falla, el 100% de las solicitudes fallan.

- **Opción B: Integraciones asíncronas (eventos + colas)**
  - **Descripción**: Publicar eventos en colas (ej., Kafka, SQS) y procesarlos de manera asíncrona. Los sistemas externos consumen estos eventos y publican sus respuestas.
  - **Impacto en atributos de calidad**:
    - **Latencia**: Latencia inicial baja (solo el tiempo para publicar el evento), pero latencia total variable.
    - **Disponibilidad**: Alta, ya que los eventos se almacenan y procesan incluso si un sistema externo está caído.
    - **Throughput**: Alto, ya que el sistema no bloquea recursos esperando respuestas.
    - **Complejidad**: Mayor complejidad en el manejo de eventos, ordenamiento y compensaciones.
  - **Ejemplo cuantificable**: Throughput aumenta un 40% al eliminar bloqueos.

### Decisión
**Se adopta la Opción B (integraciones asíncronas)**, registrada en `adr/003-decision-sincronia-asincronia.md`. La decisión prioriza la disponibilidad y el throughput, con las siguientes mitigaciones:
- **Colas con persistencia**: Usar colas con persistencia (ej., Kafka, SQS) para garantizar que los eventos no se pierdan.
- **Dead Letter Queues (DLQ)**: Redirigir eventos fallidos a DLQs para análisis posterior.
- **Idempotencia**: Implementar idempotencia en los consumidores para manejar eventos duplicados.

### Consecuencias
- **Positivas**:
  - Cumplimiento del throughput de 1,500 solicitudes/segundo.
  - Alta disponibilidad incluso ante fallos en sistemas externos.
  - Escalabilidad horizontal sencilla.
- **Negativas**:
  - Complejidad adicional en el manejo de eventos y compensaciones.
  - Latencia variable en el procesamiento de validaciones.
  - Necesidad de monitoreo avanzado para detectar eventos perdidos o retrasados.

---

## 3. Escalabilidad Vertical vs. Horizontal

### Contexto
El sistema debe manejar picos de carga de hasta 1,500 solicitudes/segundo. La escalabilidad puede lograrse mediante escalado vertical (aumentar recursos en un nodo) o horizontal (añadir más nodos).

### Trade-off
- **Opción A: Escalabilidad vertical**
  - **Descripción**: Aumentar los recursos (CPU, memoria) de los nodos existentes para manejar la carga.
  - **Impacto en atributos de calidad**:
    - **Costo**: Alto costo inicial y operativo (instancias más grandes son más caras).
    - **Disponibilidad**: Riesgo de single point of failure (SPOF).
    - **Latencia**: Baja latencia intra-nodo.
    - **Complejidad**: Menor complejidad en el diseño.
  - **Ejemplo cuantificable**: Una instancia r5.4xlarge en AWS cuesta ~$1.5 por hora, pero no escala más allá de sus límites.

- **Opción B: Escalabilidad horizontal**
  - **Descripción**: Distribuir la carga entre múltiples nodos pequeños.
  - **Impacto en atributos de calidad**:
    - **Costo**: Menor costo inicial (instancias pequeñas) y pago por uso.
    - **Disponibilidad**: Alta disponibilidad al eliminar SPOFs.
    - **Latencia**: Posible aumento de latencia por comunicación entre nodos.
    - **Complejidad**: Mayor complejidad en el manejo de estado distribuido y coordinación.
  - **Ejemplo cuantificable**: 10 instancias t3.large cuestan ~$1 por hora y pueden manejar 2,000 solicitudes/segundo.

### Decisión
**Se adopta la Opción B (escalabilidad horizontal)**, registrada en `adr/002-decision-manejo-carga.md`. La decisión prioriza el costo y la disponibilidad, con las siguientes estrategias:
- **Stateless design**: Diseñar componentes sin estado para facilitar el escalado horizontal.
- **Load balancing**: Usar balanceadores de carga (ej., ALB en AWS) para distribuir solicitudes.
- **Auto-scaling**: Implementar auto-scaling basado en métricas de CPU y throughput.

### Consecuencias
- **Positivas**:
  - Menor costo operativo.
  - Alta disponibilidad y resiliencia.
  - Escalabilidad elástica para manejar picos de carga.
- **Negativas**:
  - Complejidad adicional en el manejo de estado distribuido.
  - Necesidad de monitoreo avanzado para detectar desequilibrios en la carga.

---

## 4. Monolito vs. Microservicios

### Contexto
La solución puede implementarse como un monolito (todos los componentes en un solo despliegue) o como microservicios (componentes independientes desplegables). El requisito de escalabilidad y el throughput de 1,500 solicitudes/segundo influyen en la elección.

### Trade-off
- **Opción A: Monolito**
  - **Descripción**: Implementar todos los componentes (originador, antifraude, buró de riesgos) en un solo despliegue.
  - **Impacto en atributos de calidad**:
    - **Latencia**: Baja latencia intra-proceso.
    - **Throughput**: Limitado por los recursos del nodo.
    - **Disponibilidad**: Riesgo de fallo total si el monolito falla.
    - **Complejidad**: Menor complejidad en el despliegue y monitoreo.
    - **Costo**: Menor costo operativo inicial.
  - **Ejemplo cuantificable**: Un monolito en una instancia r5.2xlarge puede manejar ~1,000 solicitudes/segundo.

- **Opción B: Microservicios**
  - **Descripción**: Descomponer la solución en servicios independientes (ej., originador-service, antifraude-service, risk-service).
  - **Impacto en atributos de calidad**:
    - **Latencia**: Aumento de latencia por comunicación entre servicios (ej., 50ms por llamada HTTP).
    - **Throughput**: Alto throughput gracias al escalado independiente de cada servicio.
    - **Disponibilidad**: Alta disponibilidad al aislar fallos.
    - **Complejidad**: Mayor complejidad en el despliegue, monitoreo y coordinación.
    - **Costo**: Mayor costo operativo por la necesidad de gestionar múltiples servicios.
  - **Ejemplo cuantificable**: Cada servicio puede escalarse independientemente (ej., escalar solo el antifraude-service durante picos).

### Decisión
**Se adopta la Opción B (microservicios)**, registrada en `adr/001-decision-arquitectura-base.md`. La decisión prioriza la escalabilidad y la disponibilidad, con las siguientes estrategias:
- **Bounded contexts**: Definir contextos acotados para cada servicio (ej., originación, antifraude, riesgos).
- **API Gateway**: Usar un API Gateway para exponer una interfaz unificada a los clientes.
- **Service Mesh**: Implementar un service mesh (ej., Istio, Linkerd) para manejar la comunicación entre servicios.

### Consecuencias
- **Positivas**:
  - Escalabilidad independiente de cada servicio.
  - Alta disponibilidad y aislamiento de fallos.
  - Flexibilidad para adoptar tecnologías específicas por servicio.
- **Negativas**:
  - Complejidad adicional en el despliegue y monitoreo.
  - Latencia aumentada por comunicación entre servicios.
  - Mayor costo operativo.

---

## 5. Costo vs. Latencia en Almacenamiento

### Contexto
El sistema requiere almacenar datos temporales (ej., solicitudes en proceso) y datos permanentes (ej., decisiones de préstamos). Las opciones de almacenamiento (ej., DynamoDB, RDS, S3) tienen diferentes costos y latencias.

### Trade-off
- **Opción A: Almacenamiento de baja latencia (DynamoDB, Aurora)**
  - **Descripción**: Usar bases de datos de baja latencia para todos los datos.
  - **Impacto en atributos de calidad**:
    - **Latencia**: Baja latencia de lectura/escritura (<10ms).
    - **Throughput**: Alto throughput para operaciones de lectura/escritura.
    - **Costo**: Alto costo por operación (ej., ~$1.25 por millón de escrituras en DynamoDB).
    - **Escalabilidad**: Escalabilidad automática.
  - **Ejemplo cuantificable**: Aurora puede manejar 200,000 lecturas/segundo con baja latencia.

- **Opción B: Almacenamiento de bajo costo (S3, Glacier)**
  - **Descripción**: Usar almacenamiento de bajo costo para datos que no requieren baja latencia.
  - **Impacto en atributos de calidad**:
    - **Latencia**: Alta latencia para operaciones de lectura/escritura (100ms-1s).
    - **Throughput**: Limitado por el ancho de banda.
    - **Costo**: Bajo costo por GB almacenado (~$0.023/GB en S3).
    - **Escalabilidad**: Escalabilidad ilimitada.
  - **Ejemplo cuantificable**: S3 cuesta ~$23/TB/mes, pero no es adecuado para datos transaccionales.

### Decisión
**Se adopta un enfoque híbrido**, registrado en `adr/004-decision-resiliencia.md**:
- **Datos transaccionales**: Usar DynamoDB o Aurora para datos que requieren baja latencia (ej., solicitudes en proceso).
- **Datos históricos**: Usar S3 para datos que no requieren acceso frecuente (ej., decisiones de préstamos antiguas).
- **Caching**: Usar Redis o Memcached para datos frecuentemente accedidos.

### Consecuencias
- **Positivas**:
  - Balance entre costo y latencia.
  - Escalabilidad para ambos tipos de datos.
- **Negativas**:
  - Complejidad adicional en la gestión de múltiples sistemas de almacenamiento.
  - Necesidad de sincronización entre sistemas.

---

## 6. Seguridad vs. Usabilidad

### Contexto
La solución debe garantizar la seguridad de los datos sensibles (ej., información personal, detalles de préstamos) mientras mantiene una experiencia de usuario fluida. Los requisitos incluyen autenticación, autorización y cifrado.

### Trade-off
- **Opción A: Seguridad estricta (MFA, cifrado extremo a extremo)**
  - **Descripción**: Implementar autenticación multifactor (MFA) y cifrado de datos en tránsito y en reposo.
  - **Impacto en atributos de calidad**:
    - **Seguridad**: Alta seguridad, cumplimiento de regulaciones (ej., GDPR, PCI-DSS).
    - **Usabilidad**: Experiencia de usuario degradada (ej., pasos adicionales para autenticación).
    - **Latencia**: Aumento de latencia por operaciones criptográficas.
    - **Costo**: Mayor costo por implementación y mantenimiento.
  - **Ejemplo cuantificable**: MFA puede aumentar el tiempo de autenticación en 3-5 segundos.

- **Opción B: Usabilidad priorizada (SSO, cifrado básico)**
  - **Descripción**: Usar autenticación simplificada (ej., SSO con OAuth) y cifrado básico (TLS).
  - **Impacto en atributos de calidad**:
    - **Usabilidad**: Experiencia de usuario fluida.
    - **Latencia**: Baja latencia.
    - **Seguridad**: Riesgo de brechas de seguridad (ej., credenciales robadas).
    - **Costo**: Menor costo.
  - **Ejemplo cuantificable**: SSO con OAuth reduce el tiempo de autenticación a <1 segundo.

### Decisión
**Se adopta un enfoque balanceado**, registrado en `adr/004-decision-resiliencia.md`:
- **Autenticación**: Usar SSO con OAuth para simplificar el acceso, pero implementar MFA para operaciones sensibles (ej., aprobación de préstamos).
- **Cifrado**: Usar TLS para datos en tránsito y cifrado de datos sensibles en reposo (ej., información personal).
- **Autorización**: Implementar RBAC (Role-Based Access Control) para limitar el acceso a datos sensibles.

### Consecuencias
- **Positivas**:
  - Balance entre seguridad y usabilidad.
  - Cumplimiento de regulaciones.
- **Negativas**:
  - Complejidad adicional en la implementación.
  - Necesidad de educar a los usuarios sobre MFA.

---

## 7. Resiliencia vs. Complejidad

### Contexto
El sistema debe ser resiliente ante fallos de sistemas externos (ej., buró de riesgos caído) y fallos internos (ej., base de datos no disponible). La resiliencia puede lograrse mediante patrones como circuit breakers, retries y fallbacks, pero estos aumentan la complejidad.

### Trade-off
- **Opción A: Resiliencia alta (circuit breakers + retries + fallbacks)**
  - **Descripción**: Implementar patrones de resiliencia como circuit breakers (ej., Hystrix, Resilience4j), retries con backoff exponencial y fallbacks.
  - **Impacto en atributos de calidad**:
    - **Disponibilidad**: Alta disponibilidad, ya que el sistema puede manejar fallos de manera elegante.
    - **Latencia**: Latencia variable debido a retries y fallbacks.
    - **Complejidad**: Alta complejidad en el diseño e implementación.
    - **Costo**: Mayor costo operativo por la necesidad de monitoreo avanzado.
  - **Ejemplo cuantificable**: Circuit breakers pueden reducir el tiempo de falla de 30 segundos a <5 segundos.

- **Opción B: Resiliencia baja (sin patrones de resiliencia)**
  - **Descripción**: No implementar patrones de resiliencia, confiando en la disponibilidad de los sistemas externos.
  - **Impacto en atributos de calidad**:
    - **Disponibilidad**: Baja disponibilidad, ya que el sistema falla si algún sistema externo falla.
    - **Latencia**: Latencia constante.
    - **Complejidad**: Baja complejidad.
    - **Costo**: Menor costo operativo.
  - **Ejemplo cuantificable**: Sin circuit breakers, un fallo en el buró de riesgos puede dejar el sistema inutilizable.

### Decisión
**Se adopta la Opción A (resiliencia alta)**, registrada en `adr/004-decision-resiliencia.md`. La decisión prioriza la disponibilidad, con las siguientes estrategias:
- **Circuit breakers**: Implementar circuit breakers para evitar cascadas de fallos.
- **Retries**: Usar retries con backoff exponencial para manejar fallos temporales.
- **Fallbacks**: Implementar fallbacks para operaciones críticas (ej., usar datos cacheados si el buró de riesgos falla).
- **Monitoreo**: Implementar monitoreo avanzado para detectar y alertar sobre fallos.

### Consecuencias
- **Positivas**:
  - Alta disponibilidad y resiliencia.
  - Cumplimiento del SLA del 99.9%.
- **Negativas**:
  - Complejidad adicional en el diseño e implementación.
  - Mayor costo operativo.

---

## Conclusión
Los trade-offs documentados en este archivo son fundamentales para el diseño de la solución de originación digital. Cada decisión prioriza atributos de calidad específicos (disponibilidad, throughput, latencia, consistencia) y se registra en los ADRs correspondientes. Las consecuencias de cada trade-off se mitigan mediante estrategias como patrones de diseño (Sagas, Outbox), tecnologías (colas, circuit breakers) y arquitecturas (microservicios, escalabilidad horizontal).

El documento `documentos/atributos-de-calidad.md` define los umbrales y métricas para evaluar el impacto de estos trade-offs, mientras que `diagramas/secuencia-flujo-critico.mmd` ilustra cómo se aplican en el flujo crítico de la solución.