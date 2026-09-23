# Presentación de la Solución de Originación Digital

## Estructura de la Presentación
Esta presentación está diseñada para comunicar la solución de originación digital a diferentes audiencias, incluyendo stakeholders técnicos y no técnicos. La estructura sigue un hilo narrativo que cubre:
1. **Contexto y problema**: Definición del problema y los requerimientos clave.
2. **Arquitectura de la solución**: Componentes, responsabilidades y flujos críticos.
3. **Decisiones de diseño**: Trade-offs, ADRs y justificaciones.
4. **Atributos de calidad**: Métricas, umbrales y estrategias de cumplimiento.
5. **Riesgos y mitigaciones**: Riesgos técnicos y su manejo.
6. **Preguntas frecuentes**: Posibles objeciones y respuestas.

Cada sección incluye slides específicos para diferentes audiencias, con niveles de detalle técnico ajustados a sus necesidades.

---

## 1. Portada

**Título**: Solución de Originación Digital de Préstamos
**Subtítulo**: Arquitectura escalable, resiliente y de alto throughput
**Audiencias**:
- **Equipo ejecutivo**: CEO, CFO, CDO.
- **Equipo técnico**: Arquitectos, desarrolladores, DevOps.
- **Equipo de negocio**: Product Owners, analistas de riesgo.
**Fecha**: [Fecha de la presentación]
**Presentador**: [Nombre del arquitecto]

---

## 2. Contexto y Problema

### Slide 1: Contexto del Negocio
**Audiencia**: Equipo ejecutivo y de negocio.

- **Objetivo**: Reducir el tiempo de aprobación de préstamos de **días a segundos**.
- **Volumen**: 1,500 solicitudes/segundo en hora pico.
- **SLA**: 99.9% de disponibilidad.
- **Latencia**: Máximo 2 segundos por solicitud.
- **Sistemas involucrados**: Originador, motor antifraude, buró de riesgos, core bancario, gateway de pagos, sistema de liquidación.

**Visual**: Diagrama de contexto (`diagramas/contexto.mmd`).

---

### Slide 2: Problema Técnico
**Audiencia**: Equipo técnico.

- **Desafíos**:
  - **Throughput**: Manejar 1,500 solicitudes/segundo sin degradación.
  - **Latencia**: Responder en <2 segundos incluso con sistemas externos lentos.
  - **Disponibilidad**: Garantizar 99.9% de disponibilidad.
  - **Consistencia**: Manejar decisiones de préstamos consistentes en un entorno distribuido.
  - **Resiliencia**: Sobrevivir a fallos de sistemas externos (ej., buró de riesgos caído).

- **Requerimientos clave** (detallados en `documentos/requerimientos.md`):
  - **Funcionales**: Validación de identidad, cálculo de score crediticio, aprobación/rechazo de préstamos.
  - **No funcionales**: Escalabilidad, latencia, disponibilidad, seguridad.

**Visual**: Diagrama de contenedores (`diagramas/contenedores.mmd`).

---

## 3. Arquitectura de la Solución

### Slide 3: Componentes Principales
**Audiencia**: Todas.

- **Componentes**:
  1. **API Gateway**: Punto de entrada único para clientes.
  2. **Originador Service**: Orquesta el flujo de originación.
  3. **Antifraude Service**: Valida identidad y detecta fraudes.
  4. **Risk Service**: Consulta el buró de riesgos y calcula el score crediticio.
  5. **Core Banking Adapter**: Integra con el core bancario.
  6. **Payment Gateway Adapter**: Integra con el gateway de pagos.
  7. **Event Bus**: Cola de eventos para procesamiento asíncrono (Kafka/SQS).
  8. **Outbox Service**: Garantiza la consistencia eventual.
  9. **Cache**: Almacena datos frecuentes (Redis).

**Visual**: Diagrama de componentes (`diagramas/componentes-originador.mmd`).

---

### Slide 4: Flujo Crítico
**Audiencia**: Equipo técnico.

**Flujo paso a paso**:
1. **Cliente** envía solicitud de préstamo al **API Gateway**.
2. **API Gateway** redirige la solicitud al **Originador Service**.
3. **Originador Service** publica un evento `SolicitudCreada` en el **Event Bus**.
4. **Antifraude Service** consume el evento y valida la identidad.
5. **Risk Service** consume el evento y consulta el buró de riesgos.
6. **Originador Service** recibe las respuestas y toma una decisión (aprobar/rechazar).
7. **Core Banking Adapter** registra la decisión en el core bancario.
8. **Payment Gateway Adapter** inicia el desembolso.
9. **Originador Service** publica un evento `PrestamoAprobado`/`PrestamoRechazado`.
10. **Cliente** recibe la respuesta.

**Visual**: Diagrama de secuencia (`diagramas/secuencia-flujo-critico.mmd`).

**Detalles técnicos**:
- **Asincronía**: El cliente recibe una respuesta inmediata ("en proceso") mientras las validaciones ocurren en segundo plano.
- **Consistencia eventual**: El **Outbox Service** garantiza que todas las validaciones se completen.
- **Resiliencia**: Circuit breakers y retries manejan fallos de sistemas externos.

---

### Slide 5: Contratos de Frontera
**Audiencia**: Equipo técnico.

- **OpenAPI**: Contrato para la interfaz del **API Gateway** (`contratos/openapi-originador.yaml`).
  - Ejemplo de payload:
    ```yaml
    paths:
      /solicitudes:
        post:
          requestBody:
            content:
              application/json:
                schema:
                  type: object
                  properties:
                    clienteId:
                      type: string
                    monto:
                      type: number
                    plazo:
                      type: integer
          responses:
            "202":
              description: Solicitud aceptada para procesamiento
    ```

- **AsyncAPI**: Contrato para eventos en el **Event Bus** (`contratos/asyncapi-eventos.yaml`).
  - Ejemplo de evento:
    ```yaml
    components:
      messages:
        SolicitudCreada:
          payload:
            type: object
            properties:
              solicitudId:
                type: string
              clienteId:
                type: string
              monto:
                type: number
    ```

---

## 4. Decisiones de Diseño

### Slide 6: Trade-offs Clave
**Audiencia**: Equipo técnico y de negocio.

| Trade-off               | Decisión                          | Justificación                                                                 | Impacto                                                                                     |
|-------------------------|-----------------------------------|-------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| Consistencia vs. Disponibilidad | Asincronía con compensación     | Priorizar disponibilidad y throughput para cumplir SLA del 99.9%.          | Riesgo de inconsistencias temporales mitigado con Outbox y Sagas.                          |
| Sincronía vs. Asincronía       | Integraciones asíncronas        | Eliminar bloqueos y mejorar throughput.                                     | Complejidad adicional en manejo de eventos y compensaciones.                               |
| Escalabilidad Vertical vs. Horizontal | Escalabilidad horizontal      | Reducir costos y mejorar disponibilidad.                                    | Complejidad en manejo de estado distribuido.                                                |
| Monolito vs. Microservicios     | Microservicios                 | Escalabilidad independiente y aislamiento de fallos.                       | Mayor complejidad en despliegue y monitoreo.                                               |
| Costo vs. Latencia              | Almacenamiento híbrido         | Balance entre costo y latencia para datos transaccionales e históricos.     | Complejidad en sincronización entre sistemas.                                               |
| Seguridad vs. Usabilidad        | Seguridad balanceada           | Cumplir regulaciones sin degradar la experiencia de usuario.                | Complejidad en implementación de MFA y cifrado.                                             |
| Resiliencia vs. Complejidad     | Resiliencia alta                | Garantizar disponibilidad ante fallos.                                      | Mayor costo operativo y complejidad en implementación.                                      |

**Fuente**: `documentos/trade-offs.md`.

---

### Slide 7: ADRs (Architecture Decision Records)
**Audiencia**: Equipo técnico.

- **ADR-001**: Arquitectura basada en microservicios (`adr/001-decision-arquitectura-base.md`).
- **ADR-002**: Escalabilidad horizontal (`adr/002-decision-manejo-carga.md`).
- **ADR-003**: Procesamiento asíncrono (`adr/003-decision-sincronia-asincronia.md`).
- **ADR-004**: Patrones de resiliencia (`adr/004-decision-resiliencia.md`).

**Ejemplo de ADR (ADR-003)**:
```markdown
# ADR-003: Procesamiento Asíncrono

## Contexto
La solución debe manejar 1,500 solicitudes/segundo con un tiempo de respuesta <2 segundos.

## Opciones Evaluadas
1. **Sincronía**: Invocar todos los sistemas externos antes de responder.
2. **Asincronía**: Responder inmediatamente y procesar validaciones en segundo plano.

## Decisión
Opción 2: Procesamiento asíncrono con compensación.

## Consecuencias
- **Positivas**: Cumplimiento del throughput y latencia.
- **Negativas**: Complejidad en manejo de consistencia eventual.
```

---

## 5. Atributos de Calidad

### Slide 8: Métricas y Umbrales
**Audiencia**: Todas.

| Atributo de Calidad | Métrica                          | Umbral               | Estrategia de Cumplimiento                          |
|----------------------|----------------------------------|----------------------|------------------------------------------------------|
| Disponibilidad       | % de tiempo disponible           | 99.9%                | Escalabilidad horizontal, circuit breakers, retries  |
| Latencia             | Tiempo de respuesta promedio     | <2 segundos          | Procesamiento asíncrono, caching                     |
| Throughput           | Solicitudes/segundo              | 1,500                | Escalabilidad horizontal, colas asíncronas          |
| Consistencia         | % de decisiones consistentes     | 99.9%                | Outbox pattern, Sagas                               |
| Costo                | Costo operativo mensual          | <$50,000             | Escalabilidad horizontal, almacenamiento híbrido    |
| Seguridad            | % de datos cifrados              | 100%                 | Cifrado en tránsito y reposo, RBAC, MFA              |

**Fuente**: `documentos/atributos-de-calidad.md`.

---

### Slide 9: Estrategias de Cumplimiento
**Audiencia**: Equipo técnico.

- **Disponibilidad (99.9%)**:
  - Escalabilidad horizontal con auto-scaling.
  - Circuit breakers y retries para manejar fallos.
  - Monitoreo avanzado (CloudWatch, Prometheus).

- **Latencia (<2 segundos)**:
  - Procesamiento asíncrono para responder rápidamente.
  - Caching de datos frecuentes (Redis).
  - Ubicación geográfica cercana de componentes (ej., AWS us-east-1).

- **Throughput (1,500 solicitudes/segundo)**:
  - Escalabilidad horizontal.
  - Colas asíncronas para manejar picos.
  - Optimización de consultas a bases de datos.

- **Consistencia (99.9% decisiones consistentes)**:
  - Outbox pattern para garantizar procesamiento.
  - Sagas para coordinar validaciones.
  - Reconciliación periódica de datos.

---

## 6. Riesgos y Mitigaciones

### Slide 10: Riesgos Técnicos
**Audiencia**: Equipo técnico y ejecutivo.

| Riesgo                                      | Probabilidad | Impacto | Mitigación                                                                 |
|---------------------------------------------|---------------|---------|-----------------------------------------------------------------------------|
| Fallo del buró de riesgos                   | Alta          | Alto    | Circuit breakers, retries, fallbacks con datos cacheados                   |
| Sobrecarga del Event Bus                    | Media         | Alto    | Escalabilidad horizontal, monitoreo de colas                               |
| Inconsistencias en decisiones de préstamos  | Media         | Alto    | Outbox pattern, Sagas, reconciliación periódica                            |
| Latencia alta en el core bancario           | Alta          | Medio   | Procesamiento asíncrono, caching                                           |
| Brecha de seguridad en datos sensibles      | Baja          | Alto    | Cifrado en tránsito y reposo, RBAC, MFA                                    |
| Costos operativos exceden presupuesto       | Media         | Alto    | Optimización de recursos, almacenamiento híbrido, auto-scaling            |

**Fuente**: `documentos/riesgos-y-mitigaciones.md`.

---

### Slide 11: Estrategias de Mitigación
**Audiencia**: Equipo técnico.

- **Fallo del buró de riesgos**:
  - **Circuit breakers**: Evitar cascadas de fallos.
  - **Retries**: Reintentar con backoff exponencial.
  - **Fallbacks**: Usar datos cacheados para tomar decisiones temporales.

- **Sobrecarga del Event Bus**:
  - **Escalabilidad horizontal**: Añadir más nodos al Event Bus.
  - **Monitoreo**: Alertas en CloudWatch/Prometheus cuando las colas superen umbrales.

- **Inconsistencias**:
  - **Outbox pattern**: Garantizar que todos los eventos se publiquen.
  - **Sagas**: Coordinar compensaciones entre servicios.
  - **Reconciliación**: Ejecutar jobs periódicos para detectar inconsistencias.

---

## 7. Preguntas Frecuentes

### Slide 12: Preguntas del Equipo Ejecutivo

**Pregunta 1**: ¿Cómo garantizamos que el throughput de 1,500 solicitudes/segundo se cumpla en hora pico?

**Respuesta**:
- **Escalabilidad horizontal**: Los componentes se escalan automáticamente según la carga.
- **Procesamiento asíncrono**: Las validaciones ocurren en segundo plano, liberando recursos.
- **Monitoreo**: Alertas en tiempo real para detectar cuellos de botella.

**Pregunta 2**: ¿Qué pasa si el buró de riesgos falla?

**Respuesta**:
- **Circuit breakers**: Evitan que el fallo se propague.
- **Retries**: Reintentan la consulta con backoff exponencial.
- **Fallbacks**: Usan datos cacheados para tomar decisiones temporales.

**Pregunta 3**: ¿Cómo manejamos el riesgo de fraude?

**Respuesta**:
- **Motor antifraude**: Valida identidad y detecta patrones sospechosos.
- **Buró de riesgos**: Consulta el historial crediticio del cliente.
- **Reglas de negocio**: Umbrales configurables para aprobación/rechazo.

---

### Slide 13: Preguntas del Equipo Técnico

**Pregunta 1**: ¿Cómo manejamos la consistencia eventual en un entorno distribuido?

**Respuesta**:
- **Outbox pattern**: Garantiza que todos los eventos se publiquen.
- **Sagas**: Coordinan compensaciones entre servicios.
- **Reconciliación**: Jobs periódicos para detectar inconsistencias.

**Pregunta 2**: ¿Qué patrones de resiliencia implementamos?

**Respuesta**:
- **Circuit breakers**: Hystrix/Resilience4j para evitar cascadas de fallos.
- **Retries**: Backoff exponencial para manejar fallos temporales.
- **Fallbacks**: Datos cacheados para operaciones críticas.

**Pregunta 3**: ¿Cómo escalamos el Event Bus para manejar 1,500 solicitudes/segundo?

**Respuesta**:
- **Kafka/SQS**: Colas con alta escalabilidad.
- **Particionamiento**: Distribuir la carga entre múltiples particiones.
- **Monitoreo**: Alertas cuando las colas superen umbrales.

---

### Slide 14: Preguntas del Equipo de Negocio

**Pregunta 1**: ¿Cómo aseguramos que las decisiones de préstamos sean consistentes?

**Respuesta**:
- **Outbox pattern**: Garantiza que todas las validaciones se completen.
- **Sagas**: Coordinan compensaciones si alguna validación falla.
- **Auditorías**: Registros detallados de todas las decisiones.

**Pregunta 2**: ¿Qué pasa si un cliente recibe una aprobación pero luego se rechaza su préstamo?

**Respuesta**:
- **Compensaciones**: Las Sagas revierten decisiones inconsistentes.
- **Comunicación**: Notificar al cliente sobre el cambio de estado.
- **Reconciliación**: Procesos periódicos para corregir inconsistencias.

**Pregunta 3**: ¿Cómo reducimos el tiempo de aprobación de días a segundos?

**Respuesta**:
- **Procesamiento asíncrono**: Responder inmediatamente al cliente.
- **Caching**: Datos frecuentes almacenados en Redis.
- **Validaciones paralelas**: Antifraude y buró de riesgos se ejecutan en paralelo.

---

## 8. Cierre

### Slide 15: Resumen
**Audiencia**: Todas.

- **Problema**: Originación digital de préstamos con alto throughput y baja latencia.
- **Solución**: Arquitectura basada en microservicios, procesamiento asíncrono y patrones de resiliencia.
- **Trade-offs**: Priorización de disponibilidad y throughput sobre consistencia fuerte.
- **Atributos de calidad**: Cumplimiento de métricas clave (disponibilidad 99.9%, latencia <2 segundos, throughput 1,500 solicitudes/segundo).
- **Riesgos**: Mitigados con circuit breakers, retries, fallbacks y monitoreo.

**Próximos pasos**:
1. Revisión detallada de los ADRs y contratos.
2. Implementación de los componentes.
3. Pruebas de carga y resiliencia.
4. Despliegue gradual.

---

### Slide 16: Agradecimientos
**Audiencia**: Todas.

- **Equipo ejecutivo**: Por su apoyo y visión.
- **Equipo técnico**: Por su expertise y colaboración.
- **Equipo de negocio**: Por definir los requerimientos clave.

**¡Preguntas?**

---

## Apéndice: Diagramas y Referencias

### Diagramas
- **Contexto**: `diagramas/contexto.mmd`
- **Contenedores**: `diagramas/contenedores.mmd`
- **Componentes**: `diagramas/componentes-originador.mmd`
- **Secuencia**: `diagramas/secuencia-flujo-critico.mmd`

### Documentos de Referencia
- **Requerimientos**: `documentos/requerimientos.md`
- **Trade-offs**: `documentos/trade-offs.md`
- **Atributos de calidad**: `documentos/atributos-de-calidad.md`
- **Riesgos y mitigaciones**: `documentos/riesgos-y-mitigaciones.md`
- **ADRs**: `adr/`
- **Contratos**: `contratos/`