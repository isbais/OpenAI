# Prompt Caching: Optimización de Latencia y Costos

## 📋 Tabla de Contenidos
- [Introducción](#introducción)
- [Cómo Funciona](#cómo-funciona)
- [Estructuración de Prompts](#estructuración-de-prompts)
- [Requisitos](#requisitos)
- [Qué se Puede Cachear](#qué-se-puede-cachear)
- [Mejores Prácticas](#mejores-prácticas)
- [Preguntas Frecuentes](#preguntas-frecuentes)

## Introducción

Reduce la latencia y el costo con **prompt caching**.

Los prompts de modelos a menudo contienen contenido repetitivo, como system prompts e instrucciones comunes. OpenAI enruta las solicitudes de API a servidores que procesaron recientemente el mismo prompt, haciéndolo más barato y rápido que procesar un prompt desde cero. Esto puede reducir la latencia hasta en un **80%** y el costo hasta en un **75%**.

El **Prompt Caching** funciona automáticamente en todas tus solicitudes de API (sin cambios de código requeridos) y no tiene tarifas adicionales asociadas. Está habilitado para todos los [modelos](/docs/models) recientes, gpt-4o y más nuevos.

Esta guía describe cómo funciona el prompt caching en detalle, para que puedas optimizar tus prompts para menor latencia y costo.

## Estructuración de Prompts

Los **cache hits** solo son posibles para coincidencias exactas de prefijo dentro de un prompt. Para aprovechar los beneficios del caching, coloca contenido estático como instrucciones y ejemplos al **inicio** de tu prompt, y pon contenido variable, como información específica del usuario, al **final**. Esto también se aplica a imágenes y herramientas, que deben ser idénticas entre solicitudes.

![Visualización de Prompt Caching](https://openaidevs.retool.com/api/file/8593d9bb-4edb-4eb6-bed9-62bfb98db5ee)

### Ejemplo de Estructuración Óptima

```
✅ ESTRUCTURA RECOMENDADA:
[Contenido estático - instrucciones, ejemplos, system prompt]
[Contenido variable - datos del usuario, consultas específicas]

❌ ESTRUCTURA NO OPTIMIZADA:
[Contenido variable]
[Contenido estático]
[Contenido variable]
```

## Cómo Funciona

El caching se habilita automáticamente para prompts de **1024 tokens** o más. Cuando haces una solicitud de API, ocurren los siguientes pasos:

### 1. Cache Routing

- Las solicitudes se enrutan a una máquina basándose en un hash del prefijo inicial del prompt. El hash típicamente usa los primeros **256 tokens**, aunque la longitud exacta varía dependiendo del modelo.
- Si proporcionas el parámetro [`prompt_cache_key`](/docs/api-reference/responses/create#responses-create-prompt_cache_key), se combina con el hash del prefijo, permitiéndote influir en el enrutamiento y mejorar las tasas de cache hit. Esto es especialmente beneficioso cuando muchas solicitudes comparten prefijos largos y comunes.
- Si las solicitudes para la misma combinación de prefijo y `prompt_cache_key` exceden cierta tasa (aproximadamente **15 solicitudes por minuto**), algunas pueden desbordarse y ser enrutadas a máquinas adicionales, reduciendo la efectividad del cache.

### 2. Cache Lookup

El sistema verifica si la porción inicial (prefijo) de tu prompt existe en el cache de la máquina seleccionada.

### 3. Cache Hit

Si se encuentra un prefijo coincidente, el sistema usa el resultado cacheado. Esto disminuye significativamente la latencia y reduce los costos.

### 4. Cache Miss

Si no se encuentra un prefijo coincidente, el sistema procesa tu prompt completo, cacheando el prefijo después en esa máquina para solicitudes futuras.

### Duración del Cache

Los prefijos cacheados generalmente permanecen activos durante **5 a 10 minutos** de inactividad. Sin embargo, durante períodos de baja demanda, los caches pueden persistir hasta **una hora**.

## Requisitos

### Tokens Mínimos

El caching está disponible para prompts que contienen **1024 tokens** o más, con cache hits ocurriendo en incrementos de **128 tokens**. Por lo tanto, el número de tokens cacheados en una solicitud siempre caerá dentro de la siguiente secuencia: 1024, 1152, 1280, 1408, y así sucesivamente, dependiendo de la longitud del prompt.

### Monitoreo de Cache

Todas las solicitudes, incluyendo aquellas con menos de 1024 tokens, mostrarán un campo `cached_tokens` en el objeto `usage.prompt_tokens_details` del [Response object](/docs/api-reference/responses/object) o [Chat object](/docs/api-reference/chat/object) indicando cuántos de los tokens del prompt fueron un cache hit. Para solicitudes bajo 1024 tokens, `cached_tokens` será cero.

#### Ejemplo de Respuesta con Cache

```json
{
  "usage": {
    "prompt_tokens": 2006,
    "completion_tokens": 300,
    "total_tokens": 2306,
    "prompt_tokens_details": {
      "cached_tokens": 1920
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  }
}
```

## Qué se Puede Cachear

### ✅ Elementos Cacheables

| Elemento | Descripción | Consideraciones |
|----------|-------------|-----------------|
| **Messages** | El array completo de mensajes, incluyendo system, user y assistant interactions | Debe ser idéntico entre solicitudes |
| **Images** | Imágenes incluidas en mensajes de usuario, como links o datos base64 | Asegúrate de que el parámetro `detail` esté configurado idénticamente |
| **Tool use** | Tanto el array de mensajes como la lista de `tools` disponibles | Contribuyen al requisito mínimo de 1024 tokens |
| **Structured outputs** | El schema de salida estructurada sirve como prefijo al system message | Puede ser cacheado |

### 🔄 Optimización de Cache

- **Contenido estático**: Instrucciones, ejemplos, system prompts
- **Contenido dinámico**: Datos específicos del usuario, consultas variables
- **Herramientas**: Definiciones de tools que no cambian frecuentemente

## Mejores Prácticas

### 🎯 Estrategias de Optimización

#### 1. Estructura de Prompts
```
✅ HACER:
- Colocar contenido estático o repetido al INICIO
- Colocar contenido dinámico y específico del usuario al FINAL
- Mantener consistencia en la estructura entre solicitudes similares

❌ EVITAR:
- Mezclar contenido estático y dinámico
- Cambiar la estructura frecuentemente
- Colocar contenido variable al inicio
```

#### 2. Uso de prompt_cache_key
- Usa el parámetro [`prompt_cache_key`](/docs/api-reference/responses/create#responses-create-prompt_cache_key) consistentemente en solicitudes que comparten prefijos comunes
- Selecciona una granularidad que mantenga cada combinación única de prefijo-`prompt_cache_key` por debajo de **15 solicitudes por minuto** para evitar cache overflow

#### 3. Monitoreo de Performance
- **Métricas clave a monitorear**:
  - Tasas de cache hit
  - Latencia
  - Proporción de tokens cacheados
  - Costos por solicitud

#### 4. Mantenimiento de Cache
- Mantén un flujo constante de solicitudes con prefijos de prompt idénticos
- Minimiza las cache evictions para maximizar los beneficios del caching
- Monitorea el rendimiento regularmente

### 📊 Métricas de Rendimiento

| Métrica | Objetivo | Monitoreo |
|---------|----------|-----------|
| **Cache Hit Rate** | >70% | Porcentaje de solicitudes que usan cache |
| **Latency Reduction** | 60-80% | Comparación con solicitudes sin cache |
| **Cost Reduction** | 50-75% | Reducción en tokens procesados |
| **Cache Tokens** | Máximo posible | Tokens cacheados por solicitud |

## Preguntas Frecuentes

### 🔒 Privacidad y Seguridad

#### 1. ¿Cómo se mantiene la privacidad de datos para los caches?

Los prompt caches **no se comparten entre organizaciones**. Solo los miembros de la misma organización pueden acceder a caches de prompts idénticos.

#### 2. ¿El Prompt Caching afecta la generación de tokens de salida o la respuesta final de la API?

**No**. El Prompt Caching no influye en la generación de tokens de salida o la respuesta final proporcionada por la API. Independientemente de si se usa caching, la salida generada será idéntica. Esto es porque solo el prompt en sí se cachea, mientras que la respuesta real se calcula de nuevo cada vez basándose en el prompt cacheado.

### 🛠️ Gestión y Control

#### 3. ¿Hay alguna forma de limpiar manualmente el cache?

La limpieza manual del cache **no está disponible actualmente**. Los prompts que no han sido encontrados recientemente se limpian automáticamente del cache. Las cache evictions típicas ocurren después de **5-10 minutos** de inactividad, aunque a veces duran hasta un máximo de **una hora** durante períodos de baja demanda.

#### 4. ¿Se espera que pague extra por escribir en Prompt Caching?

**No**. El caching ocurre automáticamente, sin acción explícita necesaria ni costo extra para usar la funcionalidad de caching.

### 📈 Límites y Restricciones

#### 5. ¿Los prompts cacheados contribuyen a los límites de tasa TPM?

**Sí**, ya que el caching no afecta los rate limits.

#### 6. ¿El descuento para Prompt Caching está disponible en Scale Tier y Batch API?

El descuento para Prompt Caching **no está disponible** en Batch API pero **sí está disponible** en Scale Tier. Con Scale Tier, cualquier token que se desborde a la API compartida también será elegible para caching.

#### 7. ¿El Prompt Caching funciona en solicitudes Zero Data Retention?

**Sí**, el Prompt Caching es compatible con las políticas existentes de Zero Data Retention.

---

> **Nota**: El Prompt Caching es una funcionalidad automática que optimiza el rendimiento sin requerir cambios en tu código. Para maximizar los beneficios, enfócate en estructurar tus prompts de manera consistente y monitorear las métricas de rendimiento.

### Próximos Pasos

1. **Revisa** la estructura de tus prompts actuales
2. **Implementa** las mejores prácticas de estructuración
3. **Monitorea** las métricas de cache hit
4. **Optimiza** basándote en los resultados
5. **Considera** usar `prompt_cache_key` para casos específicos

---

*Esta guía se actualiza regularmente con las mejores prácticas emergentes en prompt caching.*
