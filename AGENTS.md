# AGENTS.md

Instrucciones para el agente de IA que abra este repositorio (Claude Code, Cursor, Codex, Copilot, Gemini). Se cargan solas: no hay que pegar nada en ningun chat.

## Que es este repositorio

Es el codigo base de un reto de aprendizaje de Pragma: **Diseño de una solución de originación digital**.

| | |
|---|---|
| Tema | Diseño de solución de extremo a extremo |
| Nivel | advanced-l2 |
| Chapter | Arquitectura |
| Especialidad | Soluciones |
| Stack | Mermaid / C4 Model |
| Patron arquitectonico | Arquitectura basada en componentes con ADRs y atributos de calidad medibles |
| Tiempo estimado | 40 horas |

## Receta del stack

Esqueleto obligatorio:

- `adr/`
- `arquitectura.md`
- `diagramas/*.mmd`
- `atributos-de-calidad.md`

Dependencias:

- C4 Model n/a
- Mermaid CLI 10.9.1
- OpenAPI 3.1 3.1.0
- AsyncAPI 2.6 2.6.0
- Well-Architected Framework (AWS) n/a

## Tu tarea

Dejar este conjunto de artefactos en estado **verificable**: que el comando de verificacion corra sin errores. Escribi los archivos en disco, en este repositorio. No generes ZIPs ni archivos adjuntos.

En orden:

1. Corre `python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"` y mira que falla.
2. Completa lo que falte de la lista de abajo: manifiesto de dependencias, punto de entrada, capa de interfaz y las capas del patron declarado.
3. Arregla SOLO los errores que impiden compilar o arrancar.
4. Volve a correr `python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"` hasta que pase.
5. Pará ahí.

## Regla dura: las fases son trabajo del humano

**PROHIBIDO implementar los entregables de las fases.** El valor del reto esta en que la persona los resuelva. Tu trabajo es que tenga un proyecto que arranca; el hueco pedagogico se queda como esta.

No resuelvas nada de esto:

- **Fase 1 — Exploración del dominio y definición de requerimientos**: Documento de requerimientos que incluye las instancias del dominio, sus interacciones, y los requerimientos funcionales y no funcionales de la solución.
- **Fase 2 — Diseño de la arquitectura de la solución**: Diagrama de la arquitectura de la solución que incluye los componentes y sus responsabilidades, y un documento que describe los trade-offs y las decisiones de diseño.
- **Fase 3 — Evaluación y optimización de la solución**: Documento que describe las áreas de mejora identificadas y las soluciones propuestas para optimizar la solución, y un documento que describe los trade-offs y las decisiones de diseño necesarias para optimizar la solución.
- **Fase 4 — Comunicación de la solución a las audiencias relevantes**: Presentación que incluye los componentes de la solución, sus responsabilidades, y los trade-offs y decisiones de diseño tomadas, y un documento que describe las posibles preguntas y objeciones de las audiencias y las respuestas adecuadas.

Distincion operativa:

- **Arreglar** (si): import faltante, tipo que no existe, dependencia sin declarar, error de sintaxis, archivo referenciado que no existe.
- **No tocar** (no): logica de negocio incompleta, validaciones ausentes, secretos hardcodeados, APIs deprecadas que funcionan, concurrencia insegura, patrones mejorables. Eso es lo que la persona tiene que encontrar.

## Superficie de practica (NO completes)

Estos archivos SON el ejercicio de la persona. No los implementes; deja stubs. No toques la logica que el reto pide completar.

- [ ] `adr/004-decision-resiliencia.md` — El topic pide resiliencia: este archivo es el ejercicio.
- [ ] `adr/001-decision-arquitectura-base.md` — El topic pide decisiones de arquitectura: el ADR es el ejercicio.
- [ ] `adr/002-decision-manejo-carga.md` — El topic pide decisiones de arquitectura: el ADR es el ejercicio.
- [ ] `adr/003-decision-sincronia-asincronia.md` — El topic pide decisiones de arquitectura: el ADR es el ejercicio.

## Lo que falta y tenes que completar

No se detectaron huecos: estan los archivos declarados, el boilerplate del stack y ninguna referencia quedo colgando. Igual corre el comando de verificacion — que los archivos existan no garantiza que compilen.

### Presentes (19)

- `atributos-de-calidad.md`
- `riesgos-y-mitigaciones.md`
- `trade-offs.md`
- `arquitectura.md`
- `adr/001-decision-arquitectura-base.md`
- `adr/002-decision-manejo-carga.md`
- `adr/003-decision-sincronia-asincronia.md`
- `adr/004-decision-resiliencia.md`
- `diagramas/contexto.mmd`
- `diagramas/contenedores.mmd`
- `diagramas/componentes-originador.mmd`
- `diagramas/secuencia-flujo-critico.mmd`
- `contratos/openapi-originador.yaml`
- `contratos/asyncapi-eventos.yaml`
- `documentos/requerimientos.md`
- `documentos/atributos-de-calidad.md`
- `documentos/riesgos-y-mitigaciones.md`
- `documentos/trade-offs.md`
- `documentos/presentacion.md`

### Capas del patron declarado

Cada una tiene que existir como directorio real con al menos un archivo. Codigo plano en la raiz no satisface el patron.

- `adr`
- `diagramas`
- `contratos`
- `documentos`

## Verificacion

```bash
python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"
```

El comando tiene que pasar SIN implementar los archivos de la superficie de practica: solo andamiaje.

Ese comando pasando es la definicion de "terminado" para vos.

## Convenciones que tenes que respetar

- Un solo ecosistema: no declares librerias de otro lenguaje ni mezcles gestores de paquetes.
- Toda libreria que uses tiene que estar declarada en el manifiesto de dependencias.
- Todo import declarado tiene que usarse; todo tipo usado tiene que existir o venir de una dependencia declarada.
- El patron es **Arquitectura basada en componentes con ADRs y atributos de calidad medibles**: los contratos (interfaces, puertos) los define la capa interna y los implementa la externa, nunca al revés.
- Los archivos que crees llevan implementacion real, no stubs: sin `TODO`, sin cuerpos vacios, sin `// getters y setters`.

## Contexto del candidato

Sirve para calibrar el nivel del codigo, no para resolver las fases.

- Perfil: Chapter Arquitectura, Especialidad Soluciones, Seniority Advanced
- Brecha que el reto ataca: Sostiene las decisiones de arquitectura con atributos de calidad medibles y trade-offs explicitos
- Mision: Diseñar la solucion de originacion digital

---

*Generado por Challenge Generator — Pragma. `README.md` tiene el enunciado completo del reto para la persona. `PROMPT_MEJORA.md` es la variante para pegar en un chat, si se prefiere ese flujo.*
