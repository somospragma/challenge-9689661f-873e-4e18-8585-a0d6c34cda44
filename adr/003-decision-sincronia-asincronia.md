# ADR 003: Decisión entre Sincronía y Asincronía en Interacciones

## Contexto

El sistema de originación digital interactúa con múltiples sistemas externos (motor antifraude, buró de riesgos, core bancario, etc.). La elección entre comunicación síncrona y asíncrona afecta la latencia, la escalabilidad y la consistencia del sistema.

## Opciones Evaluadas

### Opción 1: Comunicación Totalmente Síncrona
- **Descripción**: Todas las interacciones entre componentes se realizan de forma síncrona.
- **Ventajas**: Simplicidad en el diseño, consistencia fuerte, y latencia predecible.
- **Desventajas**: Escalabilidad limitada, riesgo de fallos en cascada, y dificultad para manejar picos de carga.
- **Evaluación**: No cumple con los requerimientos de escalabilidad y disponibilidad.

### Opción 2: Comunicación Totalmente Asíncrona
- **Descripción**: Todas las interacciones entre componentes se realizan mediante colas o eventos.
- **Ventajas**: Escalabilidad horizontal, desacoplamiento entre componentes, y capacidad de manejar picos de carga.
- **Desventajas**: Eventual consistencia, latencia adicional, y complejidad en la gestión de eventos.
- **Evaluación**: Cumple con los requerimientos de escalabilidad, pero puede no garantizar el tiempo de respuesta máximo para flujos críticos.

### Opción 3: Comunicación Híbrida (Síncrona para Flujos Críticos, Asíncrona para Flujos No Críticos)
- **Descripción**: Los flujos críticos (ej. validación antifraude) se manejan de forma síncrona, mientras que los flujos no críticos (ej. liquidación) se manejan de forma asíncrona.
- **Ventajas**: Latencia garantizada para flujos críticos, escalabilidad para flujos no críticos, y desacoplamiento entre componentes.
- **Desventajas**: Complejidad en la coordinación entre flujos síncronos y asíncronos.
- **Evaluación**: Cumple con los requerimientos de escalabilidad, disponibilidad y latencia.

## Decisión

Se adopta una **comunicación híbrida**, donde los flujos críticos se manejan de forma síncrona y los flujos no críticos se manejan de forma asíncrona. Esta decisión se justifica por:
- **Latencia**: Los flujos críticos (ej. validación antifraude) requieren una respuesta inmediata para garantizar el tiempo de respuesta máximo.
- **Escalabilidad**: Los flujos no críticos (ej. liquidación) pueden manejarse de forma asíncrona para mejorar el throughput.
- **Consistencia**: Los flujos críticos mantienen consistencia fuerte, mientras que los flujos no críticos pueden tolerar eventual consistencia.

## Consecuencias

### Consecuencias Positivas
- **Latencia**: Los flujos críticos garantizan el tiempo de respuesta máximo.
- **Escalabilidad**: Los flujos no críticos se escalan horizontalmente mediante colas.
- **Flexibilidad**: La arquitectura permite adaptarse a diferentes tipos de flujos.

### Consecuencias Negativas
- **Complejidad**: La coordinación entre flujos síncronos y asíncronos aumenta la complejidad del diseño.
- **Consistencia**: Los flujos asíncronos introducen eventual consistencia, lo que puede requerir estrategias de compensación.
- **Operaciones**: La gestión de colas y eventos requiere monitoreo adicional.