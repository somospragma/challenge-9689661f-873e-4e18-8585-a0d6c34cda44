# Diseño de una solución de originación digital

El candidato debe diseñar una solución de extremo a extremo para la originación digital de préstamos en un banco. La solución debe considerar las siguientes instancias del dominio: originador de créditos, motor antifraude, buró de riesgos, core bancario, gateway de pagos, sistema de liquidación. La solución debe manejar un volumen de 1 500 solicitudes por segundo en hora pico, con un tiempo de respuesta máximo de 2 segundos y un SLA de 99.9%.

## Informacion General

| Campo | Valor |
|-------|-------|
| **Tema** | Diseño de solución de extremo a extremo |
| **Nivel** | advanced-l2 |
| **Tipo** | practical |
| **Tiempo estimado** | 40 horas |

## Fases del Reto

### Fase 0: Configuración del Proyecto

**Objetivo:** Obtener el proyecto base funcional enviando el Código Base a un asistente de IA, que lo analizará, corregirá errores y generará un ZIP listo para usar.

**Tiempo estimado:** 15-30 minutos

**Instrucciones:**

- Asegúrate de tener instalado para ejecutar el proyecto: Un IDE o editor de código.
- Copia todo el contenido del campo **Código Base** de este reto — incluyendo el texto de instrucciones que aparece al inicio.
- Abre un asistente de IA (Claude en claude.ai, ChatGPT o Gemini — se recomienda Claude), pega el contenido copiado en el chat y envíalo.
- El asistente analizará los archivos, corregirá errores y generará un archivo ZIP descargable. Descárgalo y extráelo en la carpeta donde quieras trabajar.
- Verifica que el proyecto arranca sin errores.

**Entregable:** El proyecto compila/arranca sin errores.

<details>
<summary>Pistas de conocimiento</summary>

- Copia el Código Base completo incluyendo el texto de instrucciones al inicio — esas instrucciones le indican al asistente exactamente qué hacer con los archivos.
- Si el asistente no genera el ZIP automáticamente al terminar el análisis, escríbele: "genera el ZIP ahora".
- Si el proyecto tiene errores al arrancar, comparte el mensaje de error con el mismo asistente para que lo corrija.

</details>

### Fase 1: Exploración del dominio y definición de requerimientos

**Objetivo:** Identificar las instancias del dominio y sus interacciones, y definir los requerimientos funcionales y no funcionales de la solución.

**Tiempo estimado:** 8 horas

**Instrucciones:**

- Identifica las instancias del dominio y sus interacciones.
- Define los requerimientos funcionales y no funcionales de la solución.
- Considera los umbrales numéricos y las propiedades operativas del dominio.

**Entregable:** Documento de requerimientos que incluye las instancias del dominio, sus interacciones, y los requerimientos funcionales y no funcionales de la solución.

<details>
<summary>Pistas de conocimiento</summary>

- Considera las propiedades operativas del dominio, como la latencia y el throughput.
- Identifica las posibles fuentes de errores y los modos de falla específicos.

</details>

### Fase 2: Diseño de la arquitectura de la solución

**Objetivo:** Diseñar la arquitectura de la solución que cumpla con los requerimientos definidos en la fase anterior.

**Tiempo estimado:** 12 horas

**Instrucciones:**

- Diseña la arquitectura de la solución, considerando las instancias del dominio y sus interacciones.
- Define los componentes de la solución y sus responsabilidades.
- Considera los trade-offs y las decisiones de diseño necesarias para cumplir con los requerimientos.

**Entregable:** Diagrama de la arquitectura de la solución que incluye los componentes y sus responsabilidades, y un documento que describe los trade-offs y las decisiones de diseño.

<details>
<summary>Pistas de conocimiento</summary>

- Considera los trade-offs entre consistencia y disponibilidad, y entre sincronía y asincronía.
- Identifica las posibles fuentes de errores y los modos de falla específicos.

</details>

### Fase 3: Evaluación y optimización de la solución

**Objetivo:** Evaluar y optimizar la solución diseñada en la fase anterior, considerando los requerimientos no funcionales y los posibles modos de falla.

**Tiempo estimado:** 10 horas

**Instrucciones:**

- Evalúa la solución diseñada en la fase anterior, considerando los requerimientos no funcionales y los posibles modos de falla.
- Identifica las áreas de mejora y propone soluciones para optimizar la solución.
- Considera los trade-offs y las decisiones de diseño necesarias para optimizar la solución.

**Entregable:** Documento que describe las áreas de mejora identificadas y las soluciones propuestas para optimizar la solución, y un documento que describe los trade-offs y las decisiones de diseño necesarias para optimizar la solución.

<details>
<summary>Pistas de conocimiento</summary>

- Considera los trade-offs entre consistencia y disponibilidad, y entre sincronía y asincronía.
- Identifica las posibles fuentes de errores y los modos de falla específicos.

</details>

### Fase 4: Comunicación de la solución a las audiencias relevantes

**Objetivo:** Comunicar la solución diseñada y optimizada a las audiencias relevantes, considerando sus necesidades y perspectivas.

**Tiempo estimado:** 10 horas

**Instrucciones:**

- Comunica la solución diseñada y optimizada a las audiencias relevantes, considerando sus necesidades y perspectivas.
- Prepara una presentación que incluya los componentes de la solución, sus responsabilidades, y los trade-offs y decisiones de diseño tomadas.
- Considera las posibles preguntas y objeciones de las audiencias y prepara respuestas adecuadas.

**Entregable:** Presentación que incluye los componentes de la solución, sus responsabilidades, y los trade-offs y decisiones de diseño tomadas, y un documento que describe las posibles preguntas y objeciones de las audiencias y las respuestas adecuadas.

<details>
<summary>Pistas de conocimiento</summary>

- Considera las diferentes perspectivas y necesidades de las audiencias.
- Prepara respuestas adecuadas para las posibles preguntas y objeciones de las audiencias.

</details>

## Dimensiones Evaluadas

- **queEs**: ¿Cuáles son las principales instancias del dominio y cómo interactúan?
- **paraQueSirve**: ¿Cuáles son los requerimientos funcionales y no funcionales más importantes para la solución?
- **comoSeUsa**: ¿Cuáles son los principales trade-offs que debes considerar al diseñar la arquitectura de la solución?
- **erroresComunes**: ¿Cuáles son las posibles fuentes de errores y los modos de falla específicos?
- **queDecisionesImplica**: ¿Cómo afectaron las decisiones de diseño tomadas en la fase anterior a las posibles soluciones de optimización?

## Criterios de Evaluacion

- Identificar las instancias del dominio y sus interacciones.
- Definir los requerimientos funcionales y no funcionales de la solución.
- Diseñar la arquitectura de la solución que cumpla con los requerimientos.
- Evaluar y optimizar la solución diseñada.
- Comunicar la solución a las audiencias relevantes.

## Como trabajar con un asistente de IA

Hay dos caminos, elegi uno:

- **AGENTS.md** (recomendado) — instrucciones nativas del repo. Abri esta carpeta con tu agente local (Claude Code, Cursor, Codex, Copilot, Gemini) y las carga solo. Sabe que archivos faltan y con que comando se verifica, y completa el scaffold escribiendo en disco.
- **PROMPT_MEJORA.md** — para copiar y pegar en un chat (claude.ai, ChatGPT). Devuelve un ZIP con el proyecto. Sirve si no tenes un agente en el IDE.

Ninguno de los dos resuelve las fases del reto: eso es tu trabajo.

## Verificacion

El proyecto esta listo para trabajar cuando este comando corre sin errores:

```bash
python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"
```

---

*Reto generado automaticamente por Challenge Generator - Pragma*
