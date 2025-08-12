# Guía de GPT-5: El Modelo Más Inteligente de OpenAI

## 📋 Tabla de Contenidos
- [Introducción](#introducción)
- [Características Principales](#características-principales)
- [Modelos Disponibles](#modelos-disponibles)
- [Nuevas Funcionalidades de la API](#nuevas-funcionalidades-de-la-api)
- [Guía de Migración](#guía-de-migración)
- [Mejores Prácticas](#mejores-prácticas)
- [Preguntas Frecuentes](#preguntas-frecuentes)

## Introducción

GPT-5 es nuestro modelo más inteligente hasta la fecha, entrenado para ser especialmente competente en:

- **Generación de código**, corrección de bugs y refactoring
- **Seguimiento de instrucciones**
- **Contexto largo** y llamadas a herramientas (tool calling)

Esta guía cubre las características clave de la familia de modelos GPT-5 y cómo aprovechar al máximo GPT-5.

### 🚀 Explorar Ejemplos de Código

Haz clic a través de algunas aplicaciones de demostración generadas completamente con un solo prompt de GPT-5, sin escribir ningún código manualmente.

[Ver ejemplos en la plataforma OpenAI](https://platform.openai.com/docs/guides/latest-model?gallery=open)

## Características Principales

GPT-5 está diseñado específicamente para sobresalir en:
- **Coding** y frontend engineering
- **Tool-calling** para tareas agentic
- **Razonamiento complejo** y conocimiento amplio del mundo

## Modelos Disponibles

Hay tres modelos en la serie GPT-5. En general, `gpt-5` es mejor para tus tareas más complejas que requieren conocimiento amplio del mundo. Los modelos más pequeños `mini` y `nano` sacrifican algo de conocimiento general del mundo por menor costo y menor latencia. Los modelos pequeños tenderán a funcionar mejor para tareas más bien definidas.

### Comparación de Modelos

| Variante | Mejor para |
|----------|------------|
| `gpt-5` | Razonamiento complejo, conocimiento amplio del mundo, y tareas agentic con mucho código o de múltiples pasos |
| `gpt-5-mini` | Razonamiento y chat optimizado en costo; equilibra velocidad, costo y capacidad |
| `gpt-5-nano` | Tareas de alto rendimiento, especialmente seguimiento simple de instrucciones o clasificación |

## Nuevas Funcionalidades de la API

Junto con GPT-5, estamos introduciendo algunos nuevos parámetros y funcionalidades de API diseñados para dar a los desarrolladores más control y flexibilidad:

- **Control de verbosidad** (verbosity)
- **Opción de esfuerzo de razonamiento mínimo** (minimal reasoning effort)
- **Herramientas personalizadas** (custom tools)
- **Lista de herramientas permitidas** (allowed tools list)

### Esfuerzo de Razonamiento Mínimo

El parámetro `reasoning.effort` controla cuántos tokens de razonamiento genera el modelo antes de producir una respuesta. Los modelos de razonamiento anteriores como `o3` solo soportaban `low`, `medium` y `high`: `low` favorecía la velocidad y menos tokens, mientras que `high` favorecía un razonamiento más exhaustivo.

La nueva configuración `minimal` produce muy pocos tokens de razonamiento para casos donde necesitas el tiempo más rápido posible hasta el primer token. A menudo vemos mejor rendimiento cuando el modelo puede producir algunos tokens cuando es necesario versus ninguno. El valor por defecto es `medium`.

La configuración `minimal` funciona especialmente bien en escenarios de coding y seguimiento de instrucciones, adhiriéndose estrechamente a las direcciones dadas. Sin embargo, puede requerir prompting para actuar de manera más proactiva. Para mejorar la calidad del razonamiento del modelo, incluso con esfuerzo mínimo, anímalo a "pensar" o esbozar sus pasos antes de responder.

#### Ejemplo de Código

```python
from openai import OpenAI
client = OpenAI()

response = client.responses.create(
    model="gpt-5",
    input="¿Cuánto oro se necesitaría para recubrir la Estatua de la Libertad con una capa de 1mm?",
    reasoning={
        "effort": "minimal"
    }
)

print(response)
```

### Verbosidad

La verbosidad determina cuántos tokens de salida se generan. Reducir el número de tokens disminuye la latencia general. Mientras que el enfoque de razonamiento del modelo se mantiene mayormente igual, el modelo encuentra formas de responder de manera más concisa, lo que puede mejorar o disminuir la calidad de la respuesta, dependiendo de tu caso de uso.

#### Escenarios de Verbosidad

| Nivel | Mejor para |
|-------|------------|
| **Alta verbosidad** | Cuando necesitas que el modelo proporcione explicaciones exhaustivas de documentos o realice refactoring extenso de código |
| **Baja verbosidad** | Mejor para situaciones donde quieres respuestas concisas o generación simple de código, como consultas SQL |

Los modelos anteriores a GPT-5 han usado verbosidad `medium` por defecto. Con GPT-5, hacemos esta opción configurable como `high`, `medium` o `low`.

Al generar código, los niveles de verbosidad `medium` y `high` producen código más largo y estructurado con explicaciones en línea, mientras que la verbosidad `low` produce código más corto y conciso con comentarios mínimos.

#### Ejemplo de Control de Verbosidad

```python
from openai import OpenAI
client = OpenAI()

response = client.responses.create(
    model="gpt-5",
    input="¿Cuál es la respuesta a la pregunta última sobre la vida, el universo y todo?",
    text={
        "verbosity": "low"
    }
)

print(response)
```

### Herramientas Personalizadas

Con GPT-5, estamos introduciendo una nueva capacidad llamada **custom tools**, que permite a los modelos enviar cualquier texto sin formato como entrada de tool call, pero aún restringir las salidas si se desea.

#### Entradas de Formato Libre

Define tu herramienta con `type: custom` para permitir que los modelos envíen entradas de texto plano directamente a tus herramientas, en lugar de estar limitados a JSON estructurado. El modelo puede enviar cualquier texto sin formato: código, consultas SQL, comandos de shell, archivos de configuración o prosa extensa directamente a tu herramienta.

```json
{
    "type": "custom",
    "name": "code_exec",
    "description": "Ejecuta código Python arbitrario"
}
```

#### Restricción de Salidas

GPT-5 soporta **context-free grammars (CFGs)** para custom tools, permitiéndote proporcionar una gramática Lark para restringir las salidas a una sintaxis o DSL específica. Adjuntar una CFG (por ejemplo, una gramática SQL o DSL) asegura que el texto del asistente coincida con tu gramática.

#### Mejores Prácticas para Custom Tools

1. **Escribe descripciones de herramientas concisas y explícitas**. El modelo elige qué enviar basándose en tu descripción; declara claramente si quieres que siempre llame a la herramienta.

2. **Valida las salidas en el lado del servidor**. Las cadenas de formato libre son poderosas pero requieren salvaguardas contra inyección o comandos inseguros.

### Herramientas Permitidas

El parámetro `allowed_tools` bajo `tool_choice` en la API de Respuestas de GPT-5 te permite pasar N definiciones de herramientas pero restringir el modelo a solo M (< N) de ellas. Lista tu toolkit completo en `tools`, y luego usa un bloque `allowed_tools` para nombrar el subconjunto y especificar un modo: `auto` (el modelo puede elegir cualquiera de esos) o `required` (el modelo debe invocar uno).

#### Comparación de Herramientas

| Aspecto | Standard Tools | Allowed Tools |
|---------|----------------|---------------|
| **Universo del modelo** | Todas las herramientas listadas bajo `"tools": [...]` | Solo el subconjunto bajo `"tools": [...]` en `tool_choice` |
| **Invocación de herramientas** | El modelo puede o no llamar cualquier herramienta | Modelo restringido a (o requerido para llamar) herramientas elegidas |
| **Propósito** | Declarar capacidades disponibles | Restringir qué capacidades se usan realmente |

#### Ejemplo de Configuración

```json
{
  "tool_choice": {
    "type": "allowed_tools",
    "mode": "auto",
    "tools": [
      { "type": "function", "name": "get_weather" },
      { "type": "mcp", "server_label": "deepwiki" },
      { "type": "image_generation" }
    ]
  }
}
```

### Preámbulos (Preambles)

Los preámbulos son explicaciones breves y visibles para el usuario que GPT-5 genera antes de invocar cualquier herramienta o función, esbozando su intención o plan (por ejemplo, "por qué estoy llamando a esta herramienta"). Aparecen después del chain-of-thought y antes de la llamada real a la herramienta, proporcionando transparencia en el razonamiento del modelo y mejorando la depuración, confianza del usuario y controlabilidad granular.

Para habilitar preámbulos, agrega una instrucción del sistema o desarrollador, por ejemplo: "Antes de llamar a una herramienta, explica por qué la estás llamando". GPT-5 antepone una justificación concisa a cada llamada de herramienta especificada.

## Guía de Migración

GPT-5 es nuestro mejor modelo hasta ahora, y funciona mejor con la **Responses API**, que soporta pasar chain of thought (CoT) entre turnos.

### Migrando de Otros Modelos a GPT-5

Vemos inteligencia mejorada porque la Responses API puede pasar el CoT del turno anterior al modelo. Esto lleva a menos tokens de razonamiento generados, mayores tasas de cache hit y menor latencia.

#### Recomendaciones de Migración

| Modelo Original | Reemplazo Recomendado | Configuración Inicial |
|-----------------|----------------------|----------------------|
| `o3` | `gpt-5` con reasoning `medium` o `high` | Comenzar con reasoning `medium` y ajuste de prompts |
| `gpt-4.1` | `gpt-5` con reasoning `minimal` o `low` | Comenzar con reasoning `minimal` |
| `o4-mini` o `gpt-4.1-mini` | `gpt-5-mini` con ajuste de prompts | Ajuste de prompts requerido |
| `gpt-4.1-nano` | `gpt-5-nano` con ajuste de prompts | Ajuste de prompts requerido |

### Migrando de Chat Completions a Responses API

La mayor diferencia, y razón principal para migrar de Chat Completions a la Responses API para GPT-5, es el soporte para pasar chain of thought (CoT) entre turnos.

#### Comparación de APIs

| Parámetro | Responses API | Chat Completions |
|-----------|---------------|------------------|
| **Esfuerzo de razonamiento** | `reasoning.effort` | No disponible |
| **Verbosidad** | `text.verbosity` | No disponible |
| **Custom tools** | `tools` con `type: "custom"` | Limitado |

## Mejores Prácticas

### Optimizador de Prompts de GPT-5

Crea el prompt perfecto para GPT-5 en el dashboard.

### GPT-5 es un Modelo de Razonamiento

Los modelos de razonamiento como GPT-5 descomponen problemas paso a paso, produciendo una cadena de pensamiento interna que codifica su razonamiento. Para maximizar el rendimiento, pasa estos elementos de razonamiento de vuelta al modelo: esto evita re-razonamiento y mantiene las interacciones más cerca de la distribución de entrenamiento del modelo.

En conversaciones de múltiples turnos, pasar un `previous_response_id` automáticamente hace disponibles los elementos de razonamiento anteriores. Esto es especialmente importante cuando se usan herramientas, por ejemplo, cuando una llamada de función requiere un viaje de ida y vuelta extra.

### Recursos Adicionales

- [Guía de prompting de GPT-5](#)
- [Guía de frontend de GPT-5](#)
- [Guía de nuevas funcionalidades de GPT-5](#)
- [Cookbook sobre modelos de razonamiento](#)
- [Comparación de Responses API versus Chat Completions](#)

## Preguntas Frecuentes

### ¿Cómo se integran estos modelos en ChatGPT?

En ChatGPT, hay dos modelos: `gpt-5-chat` y `gpt-5-thinking`. Ofrecen capacidades de razonamiento y razonamiento mínimo, con una capa de enrutamiento que selecciona el mejor modelo basándose en la pregunta del usuario. Los usuarios también pueden invocar razonamiento directamente a través de la UI de ChatGPT.

### ¿Estos modelos serán soportados en Codex?

Sí, `gpt-5` estará disponible en Codex y Codex CLI.

### ¿Cuál es el plan de deprecación para modelos anteriores?

Cualquier deprecación de modelos se publicará en nuestra página de deprecaciones. Enviaremos aviso anticipado de cualquier deprecación de modelos.

---

> **Nota**: Para una visión más detallada de todas estas nuevas funcionalidades, consulta el cookbook adjunto.