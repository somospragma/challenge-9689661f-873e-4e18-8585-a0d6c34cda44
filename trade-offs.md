# Trade-offs en el Diseño de la Solución de Originación Digital

## Introducción
Este documento analiza los trade-offs clave identificados durante el diseño de la solución de originación digital de préstamos. Cada trade-off se evalúa en términos de sus implicaciones técnicas, impacto en los atributos de calidad y criterios de decisión. Los trade-offs se priorizan según su relevancia para los objetivos del negocio y los requerimientos no funcionales.

---

## Metodología de Evaluación
Los trade-offs se evalúan usando los siguientes criterios:
1. **Impacto en Atributos de Calidad**: Cómo afecta el trade-off a atributos como disponibilidad, latencia, throughput, seguridad, etc.
2. **Complejidad Técnica**: Esfuerzo requerido para implementar y mantener la solución.
3. **Costo**: Costos asociados a la implementación (infraestructura, licencias, desarrollo).
4. **Flexibilidad**: Capacidad de adaptarse a cambios futuros en los requerimientos.
5. **Riesgo**: Probabilidad de que la solución introduzca nuevos riesgos o fallos.

---

## Trade-offs Identificados

### 1. Sincronía vs. Asincronía en la Comunicación entre Componentes
**Contexto**:
La comunicación entre el originador de créditos y los servicios externos (buró de riesgos, motor antifraude) puede ser síncrona (REST) o asíncrona (colas de mensajes).

**Opciones Evaluadas**:
| Opción               | Sincronía (REST)                          | Asincronía (Colas de Mensajes)             |
|----------------------|-------------------------------------------|--------------------------------------------|
| **Latencia**         | Baja (respuesta inmediata)                | Alta (depende del tamaño de la cola)       |
| **Throughput**       | Limitado por el cuello de botella         | Alto (desacopla componentes)               |
| **Complejidad**      | Baja (patrones simples)                   | Alta (manejo de colas, idempotencia)       |
| **Resiliencia**      | Baja (fallos en cascada)                  | Alta (reintentos automáticos)              |
| **Consistencia**     | Fuerte (transacciones atómicas)           | Eventual (requiere compensación)           |
| **Costo**            | Bajo (sin infraestructura adicional)      | Alto (infraestructura de colas)            |

**Decisión**:
Usar **comunicación asíncrona** para la mayoría de los flujos críticos (ej. consulta al buró de riesgos, motor antifraude) para garantizar throughput y resiliencia. Usar **comunicación síncrona** solo para flujos que requieran consistencia fuerte (ej. validación inicial de datos del cliente).

**Criterios de Decisión**:
- **Throughput**: La asincronía permite manejar 1,500 RPS sin saturar servicios externos.
- **Resiliencia**: Las colas de mensajes absorben picos de carga y permiten reintentos automáticos.
- **Latencia**: Aunque la asincronía introduce latencia adicional, el impacto se mitiga con paralelización y optimización de colas.

**Consecuencias**:
- **Positivas**:
  - Mayor throughput y resiliencia.
  - Desacoplamiento de componentes, facilitando escalabilidad.
- **Negativas**:
  - Complejidad adicional en el manejo de colas y compensación de fallos.
  - Requiere monitoreo activo de colas para evitar saturación.

---

### 2. Monolito vs. Microservicios
**Contexto**:
La solución puede implementarse como un monolito (todos los componentes en una sola aplicación) o como microservicios (componentes independientes desplegables).

**Opciones Evaluadas**:
| Opción               | Monolito                                  | Microservicios                            |
|----------------------|-------------------------------------------|-------------------------------------------|
| **Desarrollo**       | Simple (un solo código base)              | Complejo (múltiples repositorios)         |
| **Despliegue**       | Simple (una sola aplicación)              | Complejo (orquestación)                   |
| **Escalabilidad**    | Escalar todo el sistema                   | Escalar componentes individuales          |
| **Latencia**         | Baja (comunicación en memoria)            | Alta (comunicación entre servicios)       |
| **Resiliencia**      | Baja (fallo afecta todo)                  | Alta (fallo aislado)                      |
| **Costo**            | Bajo (infraestructura simple)             | Alto (infraestructura distribuida)        |
| **Flexibilidad**     | Baja (cambios afectan todo)               | Alta (cambios aislados)                   |

**Decisión**:
Implementar la solución como **microservicios** para componentes críticos (originador de créditos, motor antifraude, buró de riesgos) y como **monolito modular** para componentes menos críticos (ej. gateway de pagos, liquidación). Esta arquitectura híbrida permite escalar componentes individuales sin introducir complejidad innecesaria.

**Criterios de Decisión**:
- **Escalabilidad**: Los microservicios permiten escalar componentes según su demanda.
- **Resiliencia**: Los fallos en un componente no afectan a los demás.
- **Flexibilidad**: Los microservicios facilitan la incorporación de nuevos componentes o cambios.

**Consecuencias**:
- **Positivas**:
  - Mayor escalabilidad y resiliencia.
  - Desarrollo independiente de componentes.
- **Negativas**:
  - Complejidad en la orquestación y monitoreo.
  - Mayor latencia en la comunicación entre servicios.

---

### 3. Base de Datos Centralizada vs. Distribuida
**Contexto**:
La solución puede usar una base de datos centralizada (ej. PostgreSQL) o una base de datos distribuida (ej. DynamoDB, Cassandra).

**Opciones Evaluadas**:
| Opción               | Base de Datos Centralizada                | Base de Datos Distribuida                 |
|----------------------|-------------------------------------------|-------------------------------------------|
| **Consistencia**     | Fuerte (ACID)                             | Eventual (BASE)                           |
| **Escalabilidad**    | Limitada (escalar verticalmente)          | Alta (escalar horizontalmente)            |
| **Latencia**         | Baja (consultas simples)                  | Alta (consultas distribuidas)             |
| **Costo**            | Bajo (infraestructura simple)             | Alto (infraestructura distribuida)        |
| **Complejidad**      | Baja (patrones simples)                   | Alta (manejo de particiones, réplicas)    |

**Decisión**:
Usar una **base de datos centralizada** (PostgreSQL) para datos críticos que requieren consistencia fuerte (ej. estado de solicitudes, datos del cliente). Usar una **base de datos distribuida** (DynamoDB) para datos que requieren alta escalabilidad y baja latencia (ej. logs de auditoría, eventos de negocio).

**Criterios de Decisión**:
- **Consistencia**: Los datos críticos (ej. estado de solicitudes) requieren consistencia fuerte.
- **Escalabilidad**: Los datos no críticos (ej. logs) requieren escalabilidad horizontal.
- **Latencia**: La base de datos centralizada ofrece baja latencia para consultas simples.

**Consecuencias**:
- **Positivas**:
  - Consistencia fuerte para datos críticos.
  - Escalabilidad para datos no críticos.
- **Negativas**:
  - Complejidad en la sincronización entre bases de datos.
  - Costo adicional en infraestructura distribuida.

---

### 4. Cache Local vs. Cache Distribuida
**Contexto**:
La solución puede usar caché local (en memoria de cada instancia) o caché distribuida (ej. Redis, Memcached).

**Opciones Evaluadas**:
| Opción               | Caché Local                                | Caché Distribuida                         |
|----------------------|--------------------------------------------|-------------------------------------------|
| **Latencia**         | Baja (acceso en memoria)                   | Alta (acceso a red)                       |
| **Consistencia**     | Baja (datos desactualizados)               | Alta (datos consistentes)                 |
| **Escalabilidad**    | Limitada (datos duplicados)                | Alta (datos compartidos)                  |
| **Costo**            | Bajo (sin infraestructura adicional)       | Alto (infraestructura de caché)           |
| **Complejidad**      | Baja (implementación simple)               | Alta (manejo de caché distribuida)        |

**Decisión**:
Usar **caché distribuida** (Redis) para datos compartidos entre instancias (ej. respuestas del buró de riesgos, reglas de negocio). Usar **caché local** para datos específicos de cada instancia (ej. configuraciones locales).

**Criterios de Decisión**:
- **Consistencia**: Los datos compartidos requieren consistencia entre instancias.
- **Escalabilidad**: La caché distribuida permite escalar horizontalmente sin duplicar datos.
- **Latencia**: La caché local ofrece baja latencia para datos específicos de cada instancia.

**Consecuencias**:
- **Positivas**:
  - Consistencia en datos compartidos.
  - Escalabilidad para datos frecuentes.
- **Negativas**:
  - Costo adicional en infraestructura de caché.
  - Complejidad en el manejo de caché distribuida.

---

### 5. Orquestación vs. Coreografía de Procesos
**Contexto**:
Los procesos de negocio (ej. originación de préstamos) pueden orquestarse centralmente (ej. usando un orquestador como Camunda) o coreografiarse usando eventos (ej. Kafka).

**Opciones Evaluadas**:
| Opción               | Orquestación Centralizada                 | Coreografía con Eventos                   |
|----------------------|-------------------------------------------|-------------------------------------------|
| **Complejidad**      | Alta (orquestador central)                | Baja (eventos descentralizados)           |
| **Coupling**         | Alto (dependencia del orquestador)        | Bajo (componentes independientes)         |
| **Visibilidad**      | Alta (flujo centralizado)                 | Baja (flujo distribuido)                  |
| **Flexibilidad**     | Baja (cambios afectan al orquestador)     | Alta (cambios aislados)                   |
| **Resiliencia**      | Baja (fallo del orquestador afecta todo)  | Alta (fallo aislado)                      |

**Decisión**:
Usar **coreografía con eventos** para procesos de negocio distribuidos (ej. consulta al buró de riesgos, motor antifraude). Usar **orquestación centralizada** solo para procesos que requieran visibilidad y control estricto (ej. liquidación de préstamos).

**Criterios de Decisión**:
- **Coupling**: La coreografía reduce el acoplamiento entre componentes.
- **Resiliencia**: Los eventos permiten reintentos y compensación de fallos.
- **Flexibilidad**: Los componentes pueden evolucionar independientemente.

**Consecuencias**:
- **Positivas**:
  - Mayor resiliencia y flexibilidad.
  - Menor acoplamiento entre componentes.
- **Negativas**:
  - Complejidad en el monitoreo de flujos distribuidos.
  - Requiere manejo de idempotencia y compensación.

---

### 6. API Gateway vs. Service Mesh
**Contexto**:
La solución puede usar un API Gateway (ej. Kong, AWS API Gateway) o un Service Mesh (ej. Istio, Linkerd) para manejar la comunicación entre servicios.

**Opciones Evaluadas**:
| Opción               | API Gateway                                | Service Mesh                              |
|----------------------|--------------------------------------------|-------------------------------------------|
| **Funcionalidad**    | Enrutamiento, autenticación, rate limiting | Enrutamiento, observabilidad, resiliencia |
| **Complejidad**      | Baja (configuración simple)                | Alta (configuración distribuida)          |
| **Latencia**         | Baja (una capa adicional)                  | Alta (múltiples capas)                    |
| **Costo**            | Bajo (infraestructura simple)              | Alto (infraestructura distribuida)        |
| **Escalabilidad**    | Limitada (cuello de botella)               | Alta (distribuida)                        |

**Decisión**:
Usar un **API Gateway** para exponer APIs públicas (ej. originador de créditos) y un **Service Mesh** para la comunicación interna entre microservicios. Esta combinación permite aprovechar las ventajas de ambos enfoques.

**Criterios de Decisión**:
- **Funcionalidad**: El API Gateway es suficiente para manejar APIs públicas.
- **Resiliencia**: El Service Mesh ofrece mayor resiliencia para comunicación interna.
- **Costo**: El API Gateway es más económico para exponer APIs públicas.

**Consecuencias**:
- **Positivas**:
  - Mayor resiliencia en comunicación interna.
  - Funcionalidades avanzadas para APIs públicas (rate limiting, autenticación).
- **Negativas**:
  - Complejidad adicional en la configuración del Service Mesh.
  - Costo adicional en infraestructura.

---

## Matriz de Trade-offs Priorizados

| Trade-off                              | Impacto en Atributos de Calidad          | Complejidad | Costo | Flexibilidad | Riesgo | Decisión                     |
|----------------------------------------|------------------------------------------|-------------|-------|--------------|--------|-------------------------------|
| Sincronía vs. Asincronía               | Latencia, Throughput, Resiliencia        | Alta        | Alto  | Alta         | Medio  | Asincronía                    |
| Monolito vs. Microservicios            | Escalabilidad, Resiliencia, Flexibilidad | Alta        | Alto  | Alta         | Alto   | Microservicios (híbrido)      |
| Base de Datos Centralizada vs. Distribuida | Consistencia, Escalabilidad          | Media       | Alto  | Media        | Medio  | Centralizada (PostgreSQL)     |
| Caché Local vs. Distribuida            | Latencia, Consistencia, Escalabilidad    | Media       | Alto  | Alta         | Bajo   | Distribuida (Redis)           |
| Orquestación vs. Coreografía           | Resiliencia, Flexibilidad, Visibilidad   | Alta        | Medio | Alta         | Medio  | Coreografía con Eventos       |
| API Gateway vs. Service Mesh           | Latencia, Resiliencia, Escalabilidad     | Alta        | Alto  | Alta         | Medio  | Ambos (API Gateway + Mesh)    |

---

## Conclusión
Los trade-offs analizados en este documento representan las principales decisiones de diseño para la solución de originación digital. Cada trade-off se evaluó en términos de su impacto en los atributos de calidad, complejidad técnica, costo y flexibilidad. Las decisiones tomadas buscan equilibrar estos factores para cumplir con los requerimientos funcionales y no funcionales del sistema. Este documento servirá como base para las fases posteriores de diseño y optimización.