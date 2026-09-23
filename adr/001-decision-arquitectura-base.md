# ADR 001: Decisión de Arquitectura Base

## Contexto

El sistema de originación digital de préstamos debe integrarse con múltiples sistemas externos (motor antifraude, buró de riesgos, core bancario, gateway de pagos, sistema de liquidación) y manejar un alto volumen de solicitudes (1,500 RPS en hora pico). La arquitectura debe garantizar escalabilidad, disponibilidad del 99.9% y un tiempo de respuesta máximo de 2 segundos.

## Opciones Evaluadas

### Opción 1: Arquitectura Monolítica
- **Descripción**: Todos los componentes del sistema se despliegan como una única unidad.
- **Ventajas**: Simplicidad en el despliegue y comunicación entre componentes.
- **Desventajas**: Escalabilidad limitada, dificultad para mantener el SLA de disponibilidad, y riesgo de fallos en cascada.
- **Evaluación**: No cumple con los requerimientos de escalabilidad y disponibilidad.

### Opción 2: Arquitectura Basada en Microservicios
- **Descripción**: Cada componente del dominio (originador, antifraude, buró de riesgos, etc.) se despliega como un servicio independiente.
- **Ventajas**: Escalabilidad independiente por componente, alta disponibilidad, y aislamiento de fallos.
- **Desventajas**: Complejidad en la coordinación entre servicios, latencia adicional en las comunicaciones, y necesidad de gestionar múltiples despliegues.
- **Evaluación**: Cumple con los requerimientos de escalabilidad y disponibilidad, pero requiere una estrategia clara para manejar la latencia.

### Opción 3: Arquitectura Basada en Eventos
- **Descripción**: Los componentes se comunican mediante eventos asíncronos, utilizando un bus de eventos.
- **Ventajas**: Desacoplamiento entre componentes, escalabilidad horizontal, y capacidad de manejar picos de carga.
- **Desventajas**: Complejidad en la gestión de eventos, eventual consistencia, y dificultad para garantizar el tiempo de respuesta máximo.
- **Evaluación**: Cumple con los requerimientos de escalabilidad y disponibilidad, pero puede no garantizar el tiempo de respuesta en todas las interacciones.

## Decisión

Se adopta una **arquitectura basada en microservicios con comunicación síncrona para flujos críticos y asíncrona para flujos no críticos**, combinando lo mejor de las opciones 2 y 3. Esta decisión se justifica por:
- **Escalabilidad**: Cada microservicio puede escalarse independientemente según la demanda.
- **Disponibilidad**: El aislamiento de fallos reduce el riesgo de caídas en cascada.
- **Tiempo de respuesta**: Los flujos críticos (ej. validación antifraude) se manejan de forma síncrona para garantizar la latencia máxima.
- **Flexibilidad**: Los flujos no críticos (ej. liquidación) pueden manejarse de forma asíncrona para mejorar el throughput.

## Consecuencias

### Consecuencias Positivas
- **Escalabilidad**: La arquitectura permite escalar componentes individuales según la demanda.
- **Disponibilidad**: El aislamiento de fallos mejora la resiliencia del sistema.
- **Latencia**: Los flujos críticos se manejan de forma síncrona, garantizando el tiempo de respuesta.

### Consecuencias Negativas
- **Complejidad**: La coordinación entre microservicios requiere una estrategia clara para manejar transacciones distribuidas y consistencia eventual.
- **Operaciones**: El despliegue y monitoreo de múltiples servicios aumenta la complejidad operativa.
- **Costo**: La infraestructura necesaria para soportar múltiples servicios puede ser más costosa.

---