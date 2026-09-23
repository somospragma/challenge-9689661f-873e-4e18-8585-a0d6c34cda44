# ADR 002: Estrategia para Manejar 1,500 Solicitudes por Segundo

## Contexto

El sistema debe manejar un pico de 1,500 solicitudes por segundo (RPS) durante horas pico, con un tiempo de respuesta máximo de 2 segundos. La arquitectura debe ser capaz de escalar horizontalmente para absorber esta carga sin degradar el rendimiento.

## Opciones Evaluadas

### Opción 1: Escalado Vertical
- **Descripción**: Aumentar la capacidad de los servidores individuales (CPU, RAM, etc.).
- **Ventajas**: Simplicidad en la implementación.
- **Desventajas**: Límites físicos en el escalado, costo elevado, y riesgo de cuello de botella en componentes individuales.
- **Evaluación**: No es viable para manejar 1,500 RPS de forma sostenible.

### Opción 2: Escalado Horizontal con Balanceo de Carga
- **Descripción**: Desplegar múltiples instancias de cada microservicio y utilizar un balanceador de carga para distribuir las solicitudes.
- **Ventajas**: Escalabilidad teóricamente ilimitada, alta disponibilidad, y tolerancia a fallos.
- **Desventajas**: Complejidad en la gestión de múltiples instancias, necesidad de sincronización entre instancias, y costo de infraestructura.
- **Evaluación**: Cumple con los requerimientos de escalabilidad y disponibilidad, pero requiere una estrategia para manejar sesiones y estado.

### Opción 3: Buffering con Colas
- **Descripción**: Utilizar colas para desacoplar los componentes y manejar picos de carga.
- **Ventajas**: Desacoplamiento entre componentes, capacidad de absorber picos de carga, y mejora en el throughput.
- **Desventajas**: Eventual consistencia, complejidad en la gestión de colas, y latencia adicional.
- **Evaluación**: Cumple con los requerimientos de escalabilidad, pero puede no garantizar el tiempo de respuesta máximo para todas las solicitudes.

## Decisión

Se adopta una **combinación de escalado horizontal con balanceo de carga y buffering con colas para flujos no críticos**. Esta decisión se justifica por:
- **Escalabilidad**: El escalado horizontal permite manejar el volumen de solicitudes.
- **Latencia**: Los flujos críticos se manejan de forma síncrona para garantizar el tiempo de respuesta.
- **Throughput**: Las colas permiten desacoplar componentes y manejar picos de carga.

Los componentes críticos (ej. motor antifraude) se escalarán horizontalmente y se comunicarán de forma síncrona. Los componentes no críticos (ej. sistema de liquidación) utilizarán colas para desacoplarse y manejar picos de carga.

## Consecuencias

### Consecuencias Positivas
- **Escalabilidad**: La arquitectura permite manejar 1,500 RPS mediante escalado horizontal.
- **Disponibilidad**: El balanceo de carga mejora la tolerancia a fallos.
- **Throughput**: Las colas permiten manejar picos de carga sin degradar el rendimiento.

### Consecuencias Negativas
- **Complejidad**: La gestión de múltiples instancias y colas aumenta la complejidad operativa.
- **Latencia**: Los flujos asíncronos pueden introducir latencia adicional.
- **Costo**: La infraestructura necesaria para soportar escalado horizontal y colas puede ser costosa.

---