# Agente RAG para Física I

Asistente de **Retrieval-Augmented Generation** que responde y resuelve ejercicios de las
guías prácticas de Física I (Mecánica) fundamentando cada respuesta en el material de la
materia, con **verificación numérica en Python** y generación de diagramas de cuerpo libre
y gráficas de cinemática.

Desarrollado como parte práctica de la materia **Física I** — Universidad Nacional del Sur.

---

## Arquitectura

Pipeline de etapas desacopladas: cada una puede reemplazarse sin tocar el resto.

```
Guías de Física
      |
      v
Ingesta y chunking con metadata (tipo, sección, nro. de ejercicio)
      |
      v
Embeddings locales (multilingual-e5-base, prefijos query:/passage:)
      |
      v
Indexado en ChromaDB (HNSW, distancia coseno, persistido en Drive)
      |
      v
Recuperación top-k con filtros por metadata
      |
      v
Pasada 1: el LLM razona y emite el bloque de calculo
      |
      v
Ejecutor Python sandboxeado (builtins y modulos restringidos)
      |
      v
Pasada 2: el LLM redacta usando los resultados ya verificados
      |
      +--> Diagrama de cuerpo libre (JSON del LLM -> matplotlib)
      +--> Graficas x(t), v(t), a(t) por tramos
      |
      v
Interfaz de consulta en Gradio
```

## Stack

| Componente | Tecnología |
|---|---|
| Lenguaje | Python |
| Entorno | Google Colab |
| Base vectorial | ChromaDB (`PersistentClient`, HNSW / coseno) |
| Embeddings | `intfloat/multilingual-e5-base` vía SentenceTransformers (local, sin consumo de API) |
| Inferencia LLM | Groq API |
| Modelo de razonamiento | `llama-3.3-70b-versatile` |
| Modelo rápido | `llama-3.1-8b-instant` |
| Gráficos | matplotlib |
| Interfaz | Gradio |

## Decisiones de diseño

- **El LLM piensa, Python calcula.** Los modelos de lenguaje fallan sistemáticamente en
  aritmética y redondeo. En vez de pedirle al modelo que sea preciso, el pipeline le pide
  que *plantee* la resolución en un bloque de código, lo ejecuta en Python y recién en una
  segunda pasada le pide que redacte con los números ya verificados.
- **Ejecución sandboxeada.** El bloque de cálculo generado por el modelo se ejecuta con una
  lista blanca de builtins y módulos. Ejecutar código de un LLM sin restringir el entorno es
  una superficie de ataque real.
- **Embeddings locales en lugar de API.** `multilingual-e5-base` corre en el entorno y no
  consume quota, lo que permite reindexar el corpus completo sin límite de rate. El modelo
  se cachea en Drive: primera carga ~2 min, siguientes ~15 s.
- **Filtros de metadata sobre el retrieval.** Cada chunk se indexa con tipo, sección y número
  de ejercicio, de modo que una consulta sobre un ejercicio puntual no compite contra todo el
  corpus por similitud.
- **Routing de dos modelos.** El 70B razona; el 8B resuelve tareas auxiliares donde la latencia
  importa más que la profundidad.
- **Indexación incremental.** La indexación detecta qué documentos ya están cargados y retoma
  desde ahí, para que una sesión caída de Colab no obligue a rehacer todo.

## Evaluación

El pipeline se iteró contra un conjunto propio de casos de prueba de las guías. Cada
iteración apuntó a fallas concretas y medidas: errores de aritmética y redondeo, identificación
incorrecta de las fases del movimiento, un error conceptual recurrente sobre MCU, y el manejo
de enunciados con datos inconsistentes.

## Cómo ejecutarlo

1. Abrir el notebook en Google Colab (botón *Open in Colab* arriba).
2. Cargar la API key de Groq como *secret* de Colab con el nombre `GROQ_API_KEY`
   (panel lateral, ícono de la llave).
3. Montar Google Drive cuando lo pida: ahí se persisten el índice de ChromaDB y la caché
   del modelo de embeddings.
4. Ejecutar las celdas en orden y consultar desde la interfaz Gradio.

> **El corpus no está incluido en el repositorio.** `CHUNK_FILES` apunta a los JSON con los
> chunks del material de la materia, que viven en Drive. El código de ingesta e indexación
> funciona con cualquier corpus que respete ese formato.

> La API key **no** está en el repositorio: se lee del entorno.

## Autor

Juan Francisco Mitzig — Ingeniería en Sistemas de Información, DCIC (UNS)
[GitHub](https://github.com/juanfmitzig) · [LinkedIn](https://linkedin.com/in/juanfranciscomitzig)
