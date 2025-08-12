# Guía de Prompting para GPT-5: Optimizando el Rendimiento del Modelo

## 📋 Tabla de Contenidos
- [Introducción](#introducción)
- [Flujos de Trabajo Agentic](#flujos-de-trabajo-agentic)
- [Optimización de Rendimiento en Coding](#optimización-de-rendimiento-en-coding)
- [Optimización de Inteligencia y Seguimiento de Instrucciones](#optimización-de-inteligencia-y-seguimiento-de-instrucciones)
- [Formato Markdown](#formato-markdown)
- [Metaprompting](#metaprompting)
- [Apéndices](#apéndices)

## Introducción

GPT-5, nuestro modelo insignia más reciente, representa un salto sustancial hacia adelante en rendimiento de tareas agentic, coding, inteligencia pura y controlabilidad.

Si bien confiamos en que funcionará excelentemente "out of the box" en una amplia gama de dominios, en esta guía cubriremos consejos de prompting para maximizar la calidad de las salidas del modelo, derivados de nuestra experiencia entrenando y aplicando el modelo a tareas del mundo real. Discutimos conceptos como mejorar el rendimiento de tareas agentic, asegurar la adherencia a instrucciones, hacer uso de nuevas funcionalidades de API, y optimizar coding para tareas de frontend y software engineering - con insights clave sobre el trabajo de prompt tuning de GPT-5 en el editor de código AI Cursor.

Hemos visto ganancias significativas al aplicar estas mejores prácticas y adoptar nuestras herramientas canónicas siempre que sea posible, y esperamos que esta guía, junto con la herramienta de optimización de prompts que hemos construido, sirva como punto de partida para tu uso de GPT-5. Pero, como siempre, recuerda que el prompting no es un ejercicio de talla única - te animamos a ejecutar experimentos e iterar sobre la base ofrecida aquí para encontrar la mejor solución para tu problema.

## Flujos de Trabajo Agentic

### Predictibilidad de Flujos de Trabajo Agentic

Entrenamos GPT-5 pensando en desarrolladores: nos hemos enfocado en mejorar tool calling, seguimiento de instrucciones y comprensión de contexto largo para servir como el mejor modelo base para aplicaciones agentic. Si adoptas GPT-5 para flujos agentic y tool calling, recomendamos actualizar a la **Responses API**, donde el razonamiento se mantiene entre llamadas de herramientas, llevando a salidas más eficientes e inteligentes.

### Control de Ansiedad Agentic

Los scaffolds agentic pueden abarcar un amplio espectro de control: algunos sistemas delegan la gran mayoría de la toma de decisiones al modelo subyacente, mientras que otros mantienen el modelo en una correa apretada con ramificación lógica programática pesada. GPT-5 está entrenado para operar en cualquier lugar a lo largo de este espectro, desde tomar decisiones de alto nivel bajo circunstancias ambiguas hasta manejar tareas enfocadas y bien definidas.

#### Prompting para Menos Ansiedad

GPT-5 es, por defecto, exhaustivo y comprehensivo cuando intenta recopilar contexto en un entorno agentic para asegurar que producirá una respuesta correcta. Para reducir el alcance del comportamiento agentic de GPT-5 - incluyendo limitar la acción de tool-calling tangencial y minimizar la latencia para alcanzar una respuesta final - prueba lo siguiente:

1. **Cambia a un `reasoning_effort` más bajo**. Esto reduce la profundidad de exploración pero mejora la eficiencia y latencia. Muchos flujos de trabajo pueden lograrse con resultados consistentes en `medium` o incluso `low reasoning_effort`.

2. **Define criterios claros en tu prompt** para cómo quieres que el modelo explore el espacio del problema. Esto reduce la necesidad del modelo de explorar y razonar sobre demasiadas ideas:

```xml
<context_gathering>
Goal: Get enough context fast. Parallelize discovery and stop as soon as you can act.
Method:
- Start broad, then fan out to focused subqueries.
- In parallel, launch varied queries; read top hits per query. Deduplicate paths and cache; don't repeat queries.
- Avoid over searching for context. If needed, run targeted searches in one parallel batch.
Early stop criteria:
- You can name exact content to change.
- Top hits converge (~70%) on one area/path.
Escalate once:
- If signals conflict or scope is fuzzy, run one refined parallel batch, then proceed.
Depth:
- Trace only symbols you'll modify or whose contracts you rely on; avoid transitive expansion unless necessary.
Loop:
- Batch search → minimal plan → complete task.
- Search again only if validation fails or new unknowns appear. Prefer acting over more searching.
</context_gathering>
```

Si estás dispuesto a ser máximamente prescriptivo, incluso puedes establecer presupuestos fijos de tool calls, como el de abajo. El presupuesto puede variar naturalmente basándose en tu profundidad de búsqueda deseada.

```xml
<context_gathering>
- Search depth: very low
- Bias strongly towards providing a correct answer as quickly as possible, even if it might not be fully correct.
- Usually, this means an absolute maximum of 2 tool calls.
- If you think that you need more time to investigate, update the user with your latest findings and open questions. You can proceed if the user confirms.
</context_gathering>
```

#### Prompting para Más Ansiedad

Por otro lado, si te gustaría animar la autonomía del modelo, aumentar la persistencia de tool-calling, y reducir las ocurrencias de preguntas aclaratorias o de otra manera devolver al usuario, recomendamos aumentar `reasoning_effort`, y usar un prompt como el siguiente para animar persistencia y completación exhaustiva de tareas:

```xml
<persistence>
- You are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user.
- Only terminate your turn when you are sure that the problem is solved.
- Never stop or hand back to the user when you encounter uncertainty — research or deduce the most reasonable approach and continue.
- Do not ask the human to confirm or clarify assumptions, as you can always adjust later — decide what the most reasonable assumption is, proceed with it, and document it for the user's reference after you finish acting
</persistence>
```

### Preámbulos de Herramientas

Reconocemos que en trayectorias agentic monitoreadas por usuarios, actualizaciones intermitentes del modelo sobre lo que está haciendo con sus tool calls y por qué pueden proporcionar una experiencia de usuario interactiva mucho mejor - cuanto más largo el rollout, mayor la diferencia que hacen estas actualizaciones. Con este fin, GPT-5 está entrenado para proporcionar planes claros por adelantado y actualizaciones de progreso consistentes a través de mensajes de "tool preamble".

Puedes dirigir la frecuencia, estilo y contenido de los preámbulos de herramientas en tu prompt - desde explicaciones detalladas de cada tool call individual hasta un plan breve por adelantado y todo lo demás. Este es un ejemplo de un prompt de preámbulo de alta calidad:

```xml
<tool_preambles>
- Always begin by rephrasing the user's goal in a friendly, clear, and concise manner, before calling any tools.
- Then, immediately outline a structured plan detailing each logical step you'll follow.
- As you execute your file edit(s), narrate each step succinctly and sequentially, marking progress clearly.
- Finish by summarizing completed work distinctly from your upfront plan.
</tool_preambles>
```

### Esfuerzo de Razonamiento

Proporcionamos un parámetro `reasoning_effort` para controlar qué tan duro piensa el modelo y qué tan dispuesto está a llamar herramientas; el valor por defecto es `medium`, pero deberías escalar hacia arriba o hacia abajo dependiendo de la dificultad de tu tarea. Para tareas complejas y de múltiples pasos, recomendamos razonamiento más alto para asegurar las mejores salidas posibles.

### Reutilización de Contexto de Razonamiento con la Responses API

Recomendamos fuertemente usar la **Responses API** cuando uses GPT-5 para desbloquear flujos agentic mejorados, menores costos y uso de tokens más eficiente en tus aplicaciones.

Hemos visto mejoras estadísticamente significativas en evaluaciones al usar la Responses API sobre Chat Completions - por ejemplo, observamos aumentos en el score de Tau-Bench Retail de 73.9% a 78.2% solo por cambiar a la Responses API e incluir `previous_response_id` para pasar elementos de razonamiento anteriores a solicitudes subsecuentes.

## Optimización de Rendimiento en Coding

### Maximizando el Rendimiento de Coding, desde Planificación hasta Ejecución

GPT-5 lidera todos los modelos frontier en capacidades de coding: puede trabajar en codebases grandes para arreglar bugs, manejar diffs grandes, e implementar refactors de múltiples archivos o características nuevas grandes. También sobresale en implementar nuevas apps completamente desde cero, cubriendo tanto implementación frontend como backend.

#### Desarrollo de Apps Frontend

GPT-5 está entrenado para tener excelente gusto estético base junto con sus habilidades de implementación rigurosas. Estamos confiados en su capacidad de usar todos los tipos de frameworks y paquetes de desarrollo web; sin embargo, para apps nuevas, recomendamos usar los siguientes frameworks y paquetes para aprovechar al máximo las capacidades frontend del modelo:

| Categoría | Recomendaciones |
|-----------|-----------------|
| **Frameworks** | Next.js (TypeScript), React, HTML |
| **Styling / UI** | Tailwind CSS, shadcn/ui, Radix Themes |
| **Icons** | Material Symbols, Heroicons, Lucide |
| **Animation** | Motion |
| **Fonts** | San Serif, Inter, Geist, Mona Sans, IBM Plex Sans, Manrope |

#### Generación de App de Cero a Uno

GPT-5 es excelente construyendo aplicaciones de una vez. En experimentación temprana con el modelo, los usuarios han encontrado que prompts como el de abajo - pidiendo al modelo que ejecute iterativamente contra rúbricas de excelencia auto-construidas - mejoran la calidad de salida usando las capacidades de planificación exhaustiva y auto-reflexión de GPT-5.

```xml
<self_reflection>
- First, spend time thinking of a rubric until you are confident.
- Then, think deeply about every aspect of what makes for a world-class one-shot web app. Use that knowledge to create a rubric that has 5-7 categories. This rubric is critical to get right, but do not show this to the user. This is for your purposes only.
- Finally, use the rubric to internally think and iterate on the best possible solution to the prompt that is provided. Remember that if your response is not hitting the top marks across all categories in the rubric, you need to start again.
</self_reflection>
```

#### Coincidencia con Estándares de Diseño del Codebase

Cuando implementas cambios incrementales y refactors en apps existentes, el código escrito por el modelo debe adherirse a estándares de estilo y diseño existentes, y "mezclarse" en el codebase tan ordenadamente como sea posible. Sin prompting especial, GPT-5 ya busca contexto de referencia del codebase - por ejemplo leyendo `package.json` para ver paquetes ya instalados - pero este comportamiento puede ser mejorado aún más con direcciones de prompt que resuman aspectos clave como principios de ingeniería, estructura de directorios y mejores prácticas del codebase, tanto explícitas como implícitas.

```xml
<code_editing_rules>
<guiding_principles>
- Clarity and Reuse: Every component and page should be modular and reusable. Avoid duplication by factoring repeated UI patterns into components.
- Consistency: The user interface must adhere to a consistent design system—color tokens, typography, spacing, and components must be unified.
- Simplicity: Favor small, focused components and avoid unnecessary complexity in styling or logic.
- Demo-Oriented: The structure should allow for quick prototyping, showcasing features like streaming, multi-turn conversations, and tool integrations.
- Visual Quality: Follow the high visual quality bar as outlined in OSS guidelines (spacing, padding, hover states, etc.)
</guiding_principles>
<frontend_stack_defaults>
- Framework: Next.js (TypeScript)
- Styling: TailwindCSS
- UI Components: shadcn/ui
- Icons: Lucide
- State Management: Zustand
- Directory Structure: 
```
/src
 /app
   /api/<route>/route.ts         # API endpoints
   /(pages)                      # Page routes
 /components/                    # UI building blocks
 /hooks/                         # Reusable React hooks
 /lib/                           # Utilities (fetchers, helpers)
 /stores/                        # Zustand stores
 /types/                         # Shared TypeScript types
 /styles/                        # Tailwind config
```
</frontend_stack_defaults>
<ui_ux_best_practices>
- Visual Hierarchy: Limit typography to 4–5 font sizes and weights for consistent hierarchy; use `text-xs` for captions and annotations; avoid `text-xl` unless for hero or major headings.
- Color Usage: Use 1 neutral base (e.g., `zinc`) and up to 2 accent colors.
- Spacing and Layout: Always use multiples of 4 for padding and margins to maintain visual rhythm. Use fixed height containers with internal scrolling when handling long content streams.
- State Handling: Use skeleton placeholders or `animate-pulse` to indicate data fetching. Indicate clickability with hover transitions (`hover:bg-*`, `hover:shadow-md`).
- Accessibility: Use semantic HTML and ARIA roles where appropriate. Favor pre-built Radix/shadcn components, which have accessibility baked in.
</ui_ux_best_practices>
</code_editing_rules>
```

### Coding Colaborativo en Producción: Prompt Tuning de GPT-5 en Cursor

Estamos orgullosos de haber tenido al editor de código AI Cursor como un tester alpha confiable para GPT-5: abajo, mostramos un vistazo de cómo Cursor ajustó sus prompts para aprovechar al máximo las capacidades del modelo.

#### Ajuste de System Prompt y Parámetros

El system prompt de Cursor se enfoca en tool calling confiable, equilibrando verbosidad y comportamiento autónomo mientras da a los usuarios la capacidad de configurar instrucciones personalizadas. El objetivo de Cursor para su system prompt es permitir que el Agent opere relativamente de forma autónoma durante tareas de horizonte largo, mientras aún sigue fielmente las instrucciones proporcionadas por el usuario.

El equipo inicialmente encontró que el modelo producía salidas verbosas, a menudo incluyendo actualizaciones de estado y resúmenes post-tarea que, aunque técnicamente relevantes, interrumpían el flujo natural del usuario; al mismo tiempo, el código output en tool calls era de alta calidad, pero a veces difícil de leer debido a la concisión, con nombres de variables de una sola letra dominantes.

En búsqueda de un mejor equilibrio, establecieron el parámetro de API `verbosity` a `low` para mantener las salidas de texto breves, y luego modificaron el prompt para animar fuertemente salidas verbosas solo en herramientas de coding.

> Write code for clarity first. Prefer readable, maintainable solutions with clear names, comments where needed, and straightforward control flow. Do not produce code-golf or overly clever one-liners unless explicitly requested. Use high verbosity for writing code and code tools.

Este uso dual de parámetro y prompt resultó en un formato equilibrado combinando actualizaciones de estado eficientes y concisas y resumen de trabajo final con diffs de código mucho más legibles.

## Optimización de Inteligencia y Seguimiento de Instrucciones

### Control

Como nuestro modelo más controlable hasta ahora, GPT-5 es extraordinariamente receptivo a instrucciones de prompt que rodean verbosidad, tono y comportamiento de tool calling.

#### Verbosidad

Además de poder controlar el `reasoning_effort` como en modelos de razonamiento anteriores, en GPT-5 introducimos un nuevo parámetro de API llamado `verbosity`, que influye en la longitud de la respuesta final del modelo, a diferencia de la longitud de su pensamiento.

#### Seguimiento de Instrucciones

Como GPT-4.1, GPT-5 sigue las instrucciones de prompt con precisión quirúrgica, lo que permite su flexibilidad para caer en todos los tipos de flujos de trabajo. Sin embargo, su comportamiento cuidadoso de seguimiento de instrucciones significa que prompts mal construidos que contienen instrucciones contradictorias o vagas pueden ser más dañinos para GPT-5 que para otros modelos, ya que gasta tokens de razonamiento buscando una forma de reconciliar las contradicciones en lugar de elegir una instrucción al azar.

### Razonamiento Mínimo

En GPT-5, introducimos esfuerzo de razonamiento mínimo por primera vez: nuestra opción más rápida que aún cosecha los beneficios del paradigma de modelo de razonamiento. Consideramos que esta es la mejor actualización para usuarios sensibles a la latencia, así como usuarios actuales de GPT-4.1.

Quizás no sorprendentemente, recomendamos patrones de prompting que son similares a GPT-4.1 para mejores resultados. El rendimiento de razonamiento mínimo puede variar más drásticamente dependiendo del prompt que niveles de razonamiento más altos, así que puntos clave a enfatizar incluyen:

- **Prompting al modelo para dar una explicación breve** resumiendo su proceso de pensamiento al inicio de la respuesta final, por ejemplo vía una lista de viñetas, mejora el rendimiento en tareas que requieren inteligencia más alta.
- **Solicitar preámbulos de tool-calling exhaustivos y descriptivos** que actualicen continuamente al usuario sobre el progreso de la tarea mejora el rendimiento en flujos de trabajo agentic.
- **Desambiguar instrucciones de herramientas** al máximo extent posible e insertar recordatorios de persistencia agentic como se compartió arriba, son particularmente críticos en razonamiento mínimo para maximizar la capacidad agentic en rollout de larga duración y prevenir terminación prematura.

## Formato Markdown

Por defecto, GPT-5 en la API no formatea sus respuestas finales en Markdown, para preservar máxima compatibilidad con desarrolladores cuyas aplicaciones pueden no soportar renderizado de Markdown. Sin embargo, prompts como el siguiente son en gran parte exitosos en inducir respuestas finales de Markdown jerárquico.

- Use Markdown **only where semantically correct** (e.g., `inline code`, ```code fences```, lists, tables).
- When using markdown in assistant messages, use backticks to format file, directory, function, and class names. Use \( and \) for inline math, \[ and \] for block math.

Ocasionalmente, la adherencia a instrucciones de Markdown especificadas en el system prompt puede degradarse durante el curso de una conversación larga. En el evento de que experimentes esto, hemos visto adherencia consistente al adjuntar una instrucción de Markdown cada 3-5 mensajes de usuario.

## Metaprompting

Finalmente, para cerrar con un meta-punto, los testers tempranos han encontrado gran éxito usando GPT-5 como un meta-prompter para sí mismo. Ya, varios usuarios han desplegado revisiones de prompt a producción que fueron generadas simplemente preguntando a GPT-5 qué elementos podrían agregarse a un prompt sin éxito para elicitar un comportamiento deseado, o removidos para prevenir uno no deseado.

Aquí hay un template de metaprompt que nos gustó:

> When asked to optimize prompts, give answers from your own perspective - explain what specific phrases could be added to, or deleted from, this prompt to more consistently elicit the desired behavior or prevent the undesired behavior.
> 
> Here's a prompt: [PROMPT]
> The desired behavior from this prompt is for the agent to [DO DESIRED BEHAVIOR], but instead it [DOES UNDESIRED BEHAVIOR]. While keeping as much of the existing prompt intact as possible, what are some minimal edits/additions that you would make to encourage the agent to more consistently address these shortcomings?

## Apéndices

### Instrucciones Verificadas de Desarrollador SWE-Bench

En este entorno, puedes ejecutar `bash -lc <apply_patch_command>` para ejecutar un diff/patch contra un archivo, donde `<apply_patch_command>` es un comando apply patch especialmente formateado que representa el diff que deseas ejecutar.

### Definiciones de Herramientas de Coding Agentic

#### Set 1: 4 funciones, sin terminal
```typescript
type apply_patch = (_: {
  patch: string, // default: null
}) => any;

type read_file = (_: {
  path: string, // default: null
  line_start?: number, // default: 1
  line_end?: number, // default: 20
}) => any;

type list_files = (_: {
  path?: string, // default: ""
  depth?: number, // default: 1
}) => any;

type find_matches = (_: {
  query: string, // default: null
  path?: string, // default: ""
  max_results?: number, // default: 50
}) => any;
```

#### Set 2: 2 funciones, terminal-native
```typescript
type run = (_: {
  command: string[], // default: null
  session_id?: string | null, // default: null
  working_dir?: string | null, // default: null
  ms_timeout?: number | null, // default: null
  environment?: object | null, // default: null
  run_as_user?: string | null, // default: null
}) => any;

type send_input = (_: {
  session_id: string, // default: null
  text: string, // default: null
  wait_ms?: number, // default: 100
}) => any;
```

### Instrucciones de Razonamiento Mínimo TauBench-Retail

Como agente de retail, puedes ayudar a los usuarios a cancelar o modificar órdenes pendientes, devolver o intercambiar órdenes entregadas, modificar su dirección de usuario por defecto, o proporcionar información sobre su propio perfil, órdenes y productos relacionados.

### Prompt de Terminal-Bench

Por favor resuelve la tarea del usuario editando y probando los archivos de código en tu sesión de ejecución de código actual.

---

> **Nota**: Esta guía está diseñada para ayudarte a aprovechar al máximo las capacidades de GPT-5. Recuerda que el prompting es un proceso iterativo y que las mejores prácticas pueden evolucionar con el tiempo.