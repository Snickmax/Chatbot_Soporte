# Chatbot_Soporte

Chatbot de soporte técnico basado en RAG (Retrieval-Augmented Generation). Permite consultar una base de conocimiento (documentos PDF/TXT indexados en una base de datos vectorial) y responder preguntas en español a través de una interfaz web construida con Streamlit.

El proyecto nace del requerimiento de desarrollar un chatbot de soporte sobre un tema específico (p. ej. facturación electrónica) usando modelos de lenguaje open-source (ver `assets/ProyectRequeriments.txt`).

## Stack tecnológico

- **Streamlit** — interfaz web y navegación multipágina.
- **LangChain** — orquestación del flujo RAG (`ConversationalRetrievalChain`), con los paquetes `langchain-groq`, `langchain-qdrant`, `langchain-huggingface` y `langchain-text-splitters`.
- **Groq** — inferencia del LLM `gemma2-9b-it` vía `ChatGroq`.
- **Qdrant** — base de datos vectorial (colecciones con vectores de 768 dimensiones, distancia coseno).
- **HuggingFace Embeddings** — modelo `distilbert-base-nli-stsb-mean-tokens` para generar los embeddings.
- **PyPDFLoader / TextLoader** — carga de documentos PDF y TXT, troceados con `RecursiveCharacterTextSplitter` (chunks de 1000 caracteres, solape de 200).

## Arquitectura

```
app.py                          # Punto de entrada: define las páginas y la navegación
├── components/
│   ├── chat.py                 # Página del chatbot (en desarrollo)
│   ├── collections.py          # Gestión de colecciones vectoriales (en desarrollo)
│   ├── doc-gestion.py          # Gestión de documentos (en desarrollo)
│   ├── parameters.py           # Parámetros del modelo (en desarrollo)
│   └── user-gestion.py         # Gestión de usuarios (en desarrollo)
├── login/
│   └── login.py                # Página de login (placeholder)
├── config/
│   ├── langchainModels/
│   │   └── Gemma2_9b_it.py     # Inicialización del LLM de Groq
│   └── database/
│       ├── qdrant_gen_connection.py   # Cliente Qdrant, embeddings y carga de documentos
│       └── qdrant_RAG_connection.py   # Cadena conversacional RAG (retriever + prompts)
└── assets/                     # Diagramas draw.io y requerimientos del proyecto
```

Flujo general: los documentos (PDF/TXT o texto plano) se trocean y se indexan como embeddings en una colección de Qdrant; ante una pregunta del usuario, el retriever recupera los `k=5` fragmentos más relevantes y el LLM de Groq genera la respuesta condicionada a ese contexto, reformulando la pregunta con el historial de la conversación.

## Cómo correrlo

1. **Instalar dependencias** (no hay `requirements.txt` en el repo; instalar manualmente):

   ```bash
   pip install streamlit langchain langchain-groq langchain-qdrant \
       langchain-huggingface langchain-text-splitters langchain-community \
       qdrant-client python-dotenv pypdf sentence-transformers
   ```

2. **Configurar variables de entorno** en un archivo `.env` (usar `info.env` como plantilla de referencia; **nunca subir el `.env` real**):

   | Variable | Descripción |
   |---|---|
   | `GROQ_API_KEY` | API key de Groq para el LLM `gemma2-9b-it` |
   | `QDRANT_DATABASE_URL` | URL de la instancia de Qdrant |
   | `QDRANT_API_KEY` | API key de Qdrant |

   Todas las credenciales se leen desde el entorno con `os.getenv`; no hay claves en el código.

3. **Ejecutar la aplicación**:

   ```bash
   streamlit run app.py
   ```

## Autoría

Este es un proyecto de **equipo** desarrollado en la organización **Jualoz-IA**. Los commits del historial corresponden a Juan Lozada (estructura inicial, index y diagramas), Nixon Amado (integración del modelo Gemma, conexión a Qdrant y sistema RAG) y Alexander Calderón (plantilla de configuración `info.env`).

Esta copia está alojada en la cuenta de **Snickmax**, integrante del equipo; sus aportes al proyecto no quedaron registrados como commits en este historial.

## Limitaciones conocidas

- **La autenticación está desactivada en esta versión**: la función `check_if_authenticated()` en `app.py` retorna siempre `True`, por lo que nunca se muestra el formulario de login. La página `login/login.py` es solo un placeholder con el título.
- Las páginas de `components/` (chat, colecciones, gestión de documentos, parámetros y usuarios) están vacías; la lógica RAG existe en `config/` pero aún no está conectada a la interfaz.
- `config/database/qdrant_RAG_connection.py` ejecuta código al importarse (recrea la colección `text` y lanza una invocación de prueba), lo que requiere tener Qdrant y Groq accesibles solo para arrancar la app.
- No existe `requirements.txt`, por lo que las dependencias deben instalarse manualmente.
