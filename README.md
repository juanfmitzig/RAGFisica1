# Agente RAG para consultas de Física I

Agente de **Retrieval-Augmented Generation** que responde preguntas sobre el material
de Física I fundamentando las respuestas en el corpus cargado, en lugar de depender
únicamente del conocimiento paramétrico del modelo.

Desarrollado como parte práctica de la materia **Física I** — Universidad Nacional del Sur (DCIC).

## Arquitectura

El sistema está organizado como un **pipeline** de etapas desacopladas, cada una con
una responsabilidad única:

```
Material de Física  ->  Ingesta y segmentación (chunking)
                    ->  Generación de embeddings
                    ->  Indexado en ChromaDB (base vectorial)
                    ->  Recuperación por similitud (top-k)
                    ->  Generación de respuesta (LLM + contexto recuperado)
```

Esta separación permite reemplazar cualquier etapa —el modelo de embeddings, el
vector store o el LLM— sin tocar el resto del pipeline.

## Stack

| Componente | Tecnología |
|---|---|
| Lenguaje | Python |
| Entorno | Google Colab |
| Base vectorial | ChromaDB |
| Inferencia LLM | Groq API |
| Modelo | <!-- COMPLETAR: copiar el string exacto del notebook, ej. llama-3.3-70b-versatile --> |
| Embeddings | <!-- COMPLETAR: ej. all-MiniLM-L6-v2 / sentence-transformers --> |

## Cómo ejecutarlo

1. Abrir el notebook en Google Colab.
2. Cargar la API key de Groq como *secret* de Colab con el nombre `GROQ_API_KEY`
   (panel lateral, ícono de la llave).
3. Ejecutar las celdas en orden. La primera instala dependencias y la de ingesta
   construye el índice vectorial.
4. Consultar el agente desde la celda de consulta.

> La API key **no** está incluida en el repositorio y se lee desde el entorno.

## Decisiones de diseño

- **Chunking del material**: el texto se segmenta antes de vectorizar para que la
  recuperación devuelva fragmentos con contexto suficiente pero acotado.
- **ChromaDB sobre búsqueda lineal**: permite escalar la cantidad de documentos sin
  degradar el tiempo de respuesta, y persistir el índice entre ejecuciones.
- **Groq como proveedor de inferencia**: latencia muy baja, lo que hace usable el
  agente de forma interactiva.

## Autor

Juan Francisco Mitzig — Ingeniería en Sistemas de Información, DCIC (UNS)
[GitHub](https://github.com/juanfmitzig) · [LinkedIn](https://linkedin.com/in/juanfranciscomitzig)
