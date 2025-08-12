# Prompt Engineering: Estrategias para Mejorar Resultados

## 📋 Tabla de Contenidos
- [Introducción](#introducción)
- [Generación de Texto con la API](#generación-de-texto-con-la-api)
- [Elección de Modelos](#elección-de-modelos)
- [Fundamentos del Prompt Engineering](#fundamentos-del-prompt-engineering)
- [Roles de Mensajes y Seguimiento de Instrucciones](#roles-de-mensajes-y-seguimiento-de-instrucciones)
- [Prompts Reutilizables](#prompts-reutilizables)
- [Formato de Mensajes con Markdown y XML](#formato-de-mensajes-con-markdown-y-xml)
- [Few-Shot Learning](#few-shot-learning)
- [Inclusión de Información de Contexto](#inclusión-de-información-de-contexto)
- [Prompting de Modelos GPT-5](#prompting-de-modelos-gpt-5)
- [Prompting de Modelos de Razonamiento](#prompting-de-modelos-de-razonamiento)
- [Próximos Pasos](#próximos-pasos)
- [Recursos Adicionales](#recursos-adicionales)

## Introducción

Mejora los resultados con estrategias de **prompt engineering**.

Con la OpenAI API, puedes usar un [large language model](/docs/models) para generar texto desde un prompt, como podrías hacer usando [ChatGPT](https://chatgpt.com). Los modelos pueden generar casi cualquier tipo de respuesta de texto—como código, ecuaciones matemáticas, datos JSON estructurados, o prosa similar a la humana.

## Generación de Texto con la API

Aquí tienes un ejemplo simple usando la [Chat Completions API](/docs/api-reference/chat).

### Generar Texto desde un Prompt Simple

#### JavaScript
```javascript
import OpenAI from "openai";
const client = new OpenAI();

const completion = await client.chat.completions.create({
    model: "gpt-5",
    messages: [
        {
            role: "user",
            content: "Write a one-sentence bedtime story about a unicorn.",
        },
    ],
});

console.log(completion.choices[0].message.content);
```

#### Python
```python
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
    model="gpt-5",
    messages=[
        {
            "role": "user",
            "content": "Write a one-sentence bedtime story about a unicorn."
        }
    ]
)

print(completion.choices[0].message.content)
```

#### cURL
```bash
curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "model": "gpt-5",
        "messages": [
            {
                "role": "user",
                "content": "Write a one-sentence bedtime story about a unicorn."
            }
        ]
    }'
```

### Estructura de la Respuesta

Un array de contenido generado por el modelo está en la propiedad `choices` de la respuesta. En este ejemplo simple, tenemos solo una salida que se ve así:

```json
[
    {
        "index": 0,
        "message": {
            "role": "assistant",
            "content": "Under the soft glow of the moon, Luna the unicorn danced through fields of twinkling stardust, leaving trails of dreams for every child asleep.",
            "refusal": null
        },
        "logprobs": null,
        "finish_reason": "stop"
    }
]
```

Además de texto plano, también puedes hacer que el modelo devuelva datos estructurados en formato JSON - esta funcionalidad se llama [**Structured Outputs**](/docs/guides/structured-outputs).

## Elección de Modelos

Una elección clave al generar contenido a través de la API es qué modelo quieres usar - el parámetro `model` de los ejemplos de código anteriores. [Puedes encontrar una lista completa de modelos disponibles aquí](/docs/models). Aquí hay algunos factores a considerar al elegir un modelo para generación de texto.

### Tipos de Modelos

| Tipo de Modelo | Características | Casos de Uso |
|----------------|-----------------|--------------|
| **[Reasoning models](/docs/guides/reasoning)** | Generan una cadena de pensamiento interna para analizar el prompt de entrada, y sobresalen en comprensión de tareas complejas y planificación de múltiples pasos. También son generalmente más lentos y costosos de usar que los modelos GPT. | Tareas complejas, planificación, análisis profundo |
| **GPT models** | Son rápidos, eficientes en costo y altamente inteligentes, pero se benefician de instrucciones más explícitas sobre cómo realizar tareas. | Generación de texto, tareas generales, aplicaciones en tiempo real |
| **Large and small (mini or nano) models** | Ofrecen compensaciones entre velocidad, costo e inteligencia. Los modelos grandes son más efectivos para entender prompts y resolver problemas a través de dominios, mientras que los modelos pequeños son generalmente más rápidos y baratos de usar. | Optimización de costos, aplicaciones específicas |

Cuando tengas dudas, [`gpt-4.1`](/docs/models/gpt-4.1) ofrece una combinación sólida de inteligencia, velocidad y eficiencia de costo.

## Fundamentos del Prompt Engineering

**Prompt engineering** es el proceso de escribir instrucciones efectivas para un modelo, de tal manera que genere consistentemente contenido que cumpla con tus requisitos.

Debido a que el contenido generado desde un modelo es no-determinístico, hacer prompting para obtener tu salida deseada es una mezcla de arte y ciencia. Sin embargo, puedes aplicar técnicas y mejores prácticas para obtener buenos resultados consistentemente.

### Consideraciones Importantes

Algunas técnicas de prompt engineering funcionan con cada modelo, como usar roles de mensajes. Pero diferentes tipos de modelos (como reasoning versus GPT models) podrían necesitar ser prompterados de manera diferente para producir los mejores resultados. Incluso diferentes snapshots de modelos dentro de la misma familia podrían producir resultados diferentes.

Para aplicaciones más complejas, recomendamos fuertemente:

- **Fijar tus aplicaciones de producción** a [model snapshots](/docs/models) específicos (como `gpt-4.1-2025-04-14` por ejemplo) para asegurar comportamiento consistente
- **Construir [evals](/docs/guides/evals)** que midan el comportamiento de tus prompts para que puedas monitorear el rendimiento del prompt mientras iteras, o cuando cambies y actualices versiones de modelos

## Roles de Mensajes y Seguimiento de Instrucciones

Puedes proporcionar instrucciones (prompts) al modelo con [diferentes niveles de autoridad](https://model-spec.openai.com/2025-02-12.html#chain_of_command) usando **message roles**.

### Generar Texto con Mensajes Usando Diferentes Roles

#### JavaScript
```javascript
import OpenAI from "openai";
const client = new OpenAI();

const completion = await client.chat.completions.create({
    model: "gpt-5",
    messages: [
        {
            role: "developer",
            content: "Talk like a pirate."
        },
        {
            role: "user",
            content: "Are semicolons optional in JavaScript?",
        },
    ],
});

console.log(completion.choices[0].message);
```

#### Python
```python
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
    model="gpt-5",
    reasoning={"effort": "low"},
    messages=[
        {
            "role": "developer",
            "content": "Talk like a pirate."
        },
        {
            "role": "user",
            "content": "Are semicolons optional in JavaScript?"
        }
    ]
)

print(completion.choices[0].message.content)
```

#### cURL
```bash
curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "model": "gpt-5",
        "messages": [
            {
                "role": "developer",
                "content": "Talk like a pirate."
            },
            {
                "role": "user",
                "content": "Are semicolons optional in JavaScript?"
            }
        ]
    }'
```

### Jerarquía de Roles

La [OpenAI model spec](https://model-spec.openai.com/2025-02-12.html#chain_of_command) describe cómo nuestros modelos dan diferentes niveles de prioridad a mensajes con diferentes roles.

| Rol | Descripción | Prioridad |
|-----|-------------|-----------|
| **developer** | Los mensajes developer son instrucciones proporcionadas por el desarrollador de la aplicación, priorizadas antes que los mensajes de usuario. | Más alta |
| **user** | Los mensajes user son instrucciones proporcionadas por un usuario final, priorizadas después de los mensajes developer. | Media |
| **assistant** | Los mensajes generados por el modelo tienen el rol assistant. | Automática |

Una conversación de múltiples turnos puede consistir en varios mensajes de estos tipos, junto con otros tipos de contenido proporcionados tanto por ti como por el modelo. Aprende más sobre [manejo del estado de conversación aquí](/docs/guides/conversation-state).

### Analogía de Programación

Podrías pensar en los mensajes `developer` y `user` como una función y sus argumentos en un lenguaje de programación:

- Los mensajes `developer` proporcionan las reglas del sistema y la lógica de negocio, como una definición de función.
- Los mensajes `user` proporcionan entradas y configuración a las que se aplican las instrucciones del mensaje developer, como argumentos a una función.

## Prompts Reutilizables

En el dashboard de OpenAI, puedes desarrollar [prompts](/chat/edit) reutilizables que puedes usar en solicitudes de API, en lugar de especificar el contenido de prompts en código. De esta manera, puedes construir y evaluar tus prompts más fácilmente, y desplegar versiones mejoradas de tus prompts sin cambiar tu código de integración.

Los prompts reutilizables están actualmente soportados solo en la [Responses API](/docs/guides/text?api-mode=responses#reusable-prompts). No están disponibles en la Chat Completions API.

## Formato de Mensajes con Markdown y XML

Al escribir mensajes `developer` y `user`, puedes ayudar al modelo a entender los límites lógicos de tu prompt y datos de contexto usando una combinación de formato [Markdown](https://commonmark.org/help/) y [XML tags](https://www.w3.org/TR/xml/).

### Estructura de un Mensaje Developer

Los headers y listas de Markdown pueden ser útiles para marcar secciones distintas de un prompt, y para comunicar jerarquía al modelo. También pueden potencialmente hacer tus prompts más legibles durante el desarrollo. Los XML tags pueden ayudar a delinear dónde comienza y termina una pieza de contenido (como un documento de soporte usado para referencia). Los atributos XML también pueden usarse para definir metadata sobre contenido en el prompt que puede ser referenciado por tus instrucciones.

En general, un mensaje developer contendrá las siguientes secciones, usualmente en este orden:

1. **Identity:** Describe el propósito, estilo de comunicación y objetivos de alto nivel del assistant.
2. **Instructions:** Proporciona guía al modelo sobre cómo generar la respuesta que quieres. ¿Qué reglas debe seguir? ¿Qué debe hacer el modelo, y qué nunca debe hacer el modelo?
3. **Examples:** Proporciona ejemplos de posibles entradas, junto con la salida deseada del modelo.
4. **Context:** Da al modelo cualquier información adicional que pueda necesitar para generar una respuesta.

### Ejemplo de Prompt Estructurado

```text
# Identity

You are coding assistant that helps enforce the use of snake case 
variables in JavaScript code, and writing code that will run in 
Internet Explorer version 6.

# Instructions

* When defining variables, use snake case names (e.g. my_variable) 
  instead of camel case names (e.g. myVariable).
* To support old browsers, declare variables using the older 
  "var" keyword.
* Do not give responses with Markdown formatting, just return 
  the code as requested.

# Examples

<user_query>
How do I declare a string variable for a first name?
</user_query>

<assistant_response>
var first_name = "Anna";
</assistant_response>
```

### Solicitud de API

```javascript
import fs from "fs/promises";
import OpenAI from "openai";
const client = new OpenAI();

const instructions = await fs.readFile("prompt.txt", "utf-8");

const response = await client.responses.create({
    model: "gpt-5",
    instructions,
    input: "How would I declare a variable for a last name?",
});

console.log(response.output_text);
```

### Ahorro en Costo y Latencia con Prompt Caching

Al construir un mensaje, deberías intentar mantener el contenido que esperas usar una y otra vez en tus solicitudes de API al **inicio** de tu prompt, **y** entre los primeros parámetros de API que pasas en el cuerpo JSON de la solicitud a [Chat Completions](/docs/api-reference/chat) o [Responses](/docs/api-reference/responses). Esto te permite maximizar los ahorros de costo y latencia del [prompt caching](/docs/guides/prompt-caching).

## Few-Shot Learning

El **few-shot learning** te permite dirigir un large language model hacia una nueva tarea incluyendo un puñado de ejemplos de entrada/salida en el prompt, en lugar de [fine-tuning](/docs/guides/model-optimization) el modelo. El modelo implícitamente "capta" el patrón de esos ejemplos y lo aplica a un prompt.

### Ejemplo de Few-Shot Learning

```text
# Identity

You are a helpful assistant that labels short product reviews as
Positive, Negative, or Neutral.

# Instructions

* Only output a single word in your response with no additional formatting
  or commentary.
* Your response should only be one of the words "Positive", "Negative", or
  "Neutral" depending on the sentiment of the product review you are given.

# Examples

<product_review id="example-1">
I absolutely love this headphones — sound quality is amazing!
</product_review>

<assistant_response id="example-1">
Positive
</assistant_response>

<product_review id="example-2">
Battery life is okay, but the ear pads feel cheap.
</product_review>

<assistant_response id="example-2">
Neutral
</assistant_response>

<product_review id="example-3">
Terrible customer service, I'll never buy from them again.
</product_review>

<assistant_response id="example-3">
Negative
</assistant_response>
```

## Inclusión de Información de Contexto

A menudo es útil incluir información de contexto adicional que el modelo puede usar para generar una respuesta dentro del prompt que le das al modelo. Hay algunas razones comunes por las que podrías hacer esto:

- Para dar al modelo acceso a datos propietarios, o cualquier otro dato fuera del dataset en el que el modelo fue entrenado.
- Para restringir la respuesta del modelo a un conjunto específico de recursos que has determinado serán más beneficiosos.

### Retrieval-Augmented Generation (RAG)

La técnica de agregar contexto relevante adicional a la solicitud de generación del modelo a veces se llama **retrieval-augmented generation (RAG)**. Puedes agregar contexto adicional al prompt de muchas maneras diferentes, desde consultar una base de datos vectorial e incluir el texto que obtienes de vuelta en un prompt, o usando la [file search tool](/docs/guides/tools-file-search) integrada de OpenAI para generar contenido basado en documentos subidos.

### Planificación para la Ventana de Contexto

Los modelos solo pueden manejar cierta cantidad de datos dentro del contexto que consideran durante una solicitud de generación. Este límite de memoria se llama **context window**, que se define en términos de [tokens](https://blogs.nvidia.com/blog/ai-tokens-explained) (fragmentos de datos que pasas, desde texto hasta imágenes).

Los modelos tienen diferentes tamaños de context window desde el rango bajo de 100k hasta un millón de tokens para modelos GPT-4.1 más nuevos. [Consulta la documentación de modelos](/docs/models) para tamaños específicos de context window por modelo.

## Prompting de Modelos GPT-5

Los modelos GPT como [`gpt-5`](/docs/models/gpt-5) se benefician de instrucciones precisas que proporcionan explícitamente la lógica y datos requeridos para completar la tarea en el prompt. GPT-5 en particular es altamente controlable y receptivo a prompts bien especificados.

### Mejores Prácticas para GPT-5

#### 🖥️ Coding

El prompting de GPT-5 para tareas de coding es más efectivo cuando se siguen algunas mejores prácticas: definir el rol del agente, hacer cumplir el uso estructurado de herramientas con ejemplos, requerir testing exhaustivo para corrección, y establecer estándares de Markdown para salida limpia.

**Guía explícita de rol y flujo de trabajo**: Enmarca el modelo como un agente de software engineering con responsabilidades bien definidas. Proporciona instrucciones claras para usar herramientas como `functions.run` para tareas de código, y especifica cuándo no usar ciertos modos.

**Testing y validación**: Instruye al modelo para probar cambios con unit tests o comandos Python, y valida patches cuidadosamente ya que herramientas como `apply_patch` pueden devolver "Done" incluso en fallo.

**Ejemplos de uso de herramientas**: Incluye ejemplos concretos de cómo invocar comandos con las funciones proporcionadas, lo que mejora la confiabilidad y adherencia a flujos de trabajo esperados.

**Estándares de Markdown**: Guía al modelo para generar markdown limpio y semánticamente correcto usando código inline, code fences, listas y tablas donde sea apropiado.

#### 🎨 Front-end Engineering

[GPT-5](/docs/guides/latest-model) funciona bien construyendo frontends desde cero así como contribuyendo a codebases grandes y establecidos. Para obtener los mejores resultados, recomendamos usar las siguientes librerías:

| Categoría | Librerías Recomendadas |
|-----------|------------------------|
| **Styling / UI** | Tailwind CSS, shadcn/ui, Radix Themes |
| **Icons** | Lucide, Material Symbols, Heroicons |
| **Animation** | Motion |

**Zero-to-one web apps**

GPT-5 puede generar aplicaciones web front-end desde un solo prompt, sin ejemplos necesarios. Aquí hay un prompt de ejemplo:

```bash
You are a world class web developer, capable of producing stunning, interactive, and innovative websites from scratch in a single prompt. You excel at delivering top-tier one-shot solutions.
Your process is simple and follows these steps:
Step 1: Create an evaluation rubric and refine it until you are fully confident.
Step 2: Consider every element that defines a world-class one-shot web app, then use that insight to create a <ONE_SHOT_RUBRIC> with 5–7 categories. Keep this rubric hidden—it's for internal use only.
Step 3: Apply the rubric to iterate on the optimal solution to the given prompt. If it doesn't meet the highest standard across all categories, refine and try again.
Step 4: Aim for simplicity while fully achieving the goal, and avoid external dependencies such as Next.js or React.
```

#### 🤖 Agentic Tasks

Para rollouts agentic y de larga duración con GPT-5, enfoca tus prompts en tres prácticas principales: planificar tareas exhaustivamente para asegurar resolución completa, proporcionar preámbulos claros para decisiones importantes de uso de herramientas, y usar una herramienta TODO para rastrear flujo de trabajo y progreso de manera organizada.

**Planning and persistence**

```text
Remember, you are an agent - please keep going until the user's
query is completely resolved, before ending your turn and yielding
back to the user. Decompose the user's query into all required
sub-requests, and confirm that each is completed. Do not stop
after completing only part of the request. Only terminate your
turn when you are sure that the problem is solved. You must be
prepared to answer multiple queries and only finish the call once
the user has confirmed they're done.

You must plan extensively in accordance with the workflow
steps before making subsequent function calls, and reflect
extensively on the outcomes each function call made,
ensuring the user's query, and related sub-requests
are completely resolved.
```

## Prompting de Modelos de Razonamiento

Hay algunas diferencias a considerar cuando haces prompting a un [reasoning model](/docs/guides/reasoning) versus hacer prompting a un modelo GPT. En general, los modelos de razonamiento proporcionarán mejores resultados en tareas con solo guía de alto nivel. Esto difiere de los modelos GPT, que se benefician de instrucciones muy precisas.

### Analogía de Trabajo

Podrías pensar en la diferencia entre modelos de razonamiento y GPT así:

- **Un modelo de razonamiento** es como un compañero de trabajo senior. Puedes darles un objetivo para lograr y confiar en que trabajarán los detalles.
- **Un modelo GPT** es como un compañero de trabajo junior. Rendirán mejor con instrucciones explícitas para crear una salida específica.

Para más información sobre mejores prácticas cuando uses modelos de razonamiento, [consulta esta guía](/docs/guides/reasoning-best-practices).

## Próximos Pasos

Ahora que conoces los fundamentos de entradas y salidas de texto, podrías querer revisar uno de estos recursos a continuación.

### Recursos Recomendados

- **[Build a prompt in the Playground](/chat/edit)** - Usa el Playground para desarrollar e iterar en prompts
- **[Generate JSON data with Structured Outputs](/docs/guides/structured-outputs)** - Asegura que los datos JSON emitidos desde un modelo se conformen a un JSON schema
- **[Full API reference](/docs/api-reference/responses)** - Revisa todas las opciones para generación de texto en la referencia de API

## Recursos Adicionales

Para más inspiración, visita el [OpenAI Cookbook](https://cookbook.openai.com), que contiene código de ejemplo y también enlaces a recursos de terceros como:

### 📚 Recursos de Prompting

| Categoría | Descripción | Enlaces |
|-----------|-------------|---------|
| **Prompting libraries & tools** | Librerías y herramientas para prompt engineering | [Ver recursos](https://cookbook.openai.com/related_resources#prompting-libraries--tools) |
| **Prompting guides** | Guías y tutoriales de prompting | [Ver recursos](https://cookbook.openai.com/related_resources#prompting-guides) |
| **Video courses** | Cursos en video sobre prompt engineering | [Ver recursos](https://cookbook.openai.com/related_resources#video-courses) |
| **Papers on advanced prompting** | Papers académicos sobre prompting avanzado para mejorar razonamiento | [Ver recursos](https://cookbook.openai.com/related_resources#papers-on-advanced-prompting-to-improve-reasoning) |

---

> **Nota**: El prompt engineering es una habilidad que mejora con la práctica. Experimenta con diferentes técnicas y encuentra lo que funciona mejor para tus casos de uso específicos.

### Guías Específicas Recomendadas

- **[GPT-5 prompting guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)** - Obtén el máximo provecho del prompting de GPT-5
- **[Frontend engineering cookbook](https://cookbook.openai.com/examples/gpt-5/gpt-5_frontend)** - Guía específica para desarrollo frontend

---

*Esta guía se actualiza regularmente con las mejores prácticas emergentes en prompt engineering.*
