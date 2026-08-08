# Arquitectura de CustomerAgent.ai

## Diagrama

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   Usuario     │──────▶│  Frontend    │──────▶│   Backend    │
│  (Web/Bot)    │       │  (Vercel)    │       │  (Railway)   │
└──────────────┘       └──────────────┘       └──────┬───────┘
                                                      │
                                            ┌─────────▼─────────┐
                                            │    Supabase +     │
                                            │     pgvector      │
                                            │  (Vector Store)   │
                                            └───────────────────┘
                                                      │
                                            ┌─────────▼─────────┐
                                            │  LLM Providers    │
                                            │  Groq · OpenAI    │
                                            │  Gemini · Claude  │
                                            └───────────────────┘
```

## Flujo de una consulta (RAG)

```
Usuario → Pregunta → Embeddings → Búsqueda semántica (pgvector) → Contexto relevante → LLM → Respuesta + Fuentes
```

1. El usuario hace una pregunta desde el widget o el panel.
2. Se genera un embedding vectorial de la pregunta (Gemini embedding-001 por defecto).
3. Se buscan los fragmentos más relevantes en la base de datos vectorial (Supabase + pgvector) mediante la RPC `match_document_chunks`.
4. El contexto encontrado se envía al LLM junto con la pregunta.
5. El LLM genera una respuesta basada **solo** en el contexto proporcionado.
6. Se muestran las fuentes utilizadas con su porcentaje de similitud.

## Componentes

| Componente | Tecnología | Función |
|---|---|---|
| **Frontend** | Next.js 14 + TypeScript + Tailwind CSS | Panel: chat, gestión de documentos, métricas, configuración |
| **Widget** | JavaScript vanilla (servido por FastAPI) | Chat embebible en cualquier web con `<script>` de una línea |
| **Backend** | Python 3.12 + FastAPI | API REST, streaming SSE, rate limiting, auth |
| **Vector DB** | Supabase (PostgreSQL + pgvector) | Almacenamiento de documentos, chunks y embeddings |
| **Auth** | GoTrue (Supabase Auth) | Sesiones JWT por organización |
| **Embeddings** | Gemini embedding-001 (batch, gratis) · OpenAI text-embedding-3-small | Vectorización de documentos y consultas |
| **LLMs** | Groq · OpenAI · Gemini 1.5 Flash · Claude 3 Haiku | Generación de respuestas con streaming |
| **Ingesta** | PyPDF2 | Extracción de texto de PDF, TXT, MD, CSV |
| **Pagos** | Mercado Pago (preaprobación) | Suscripción mensual + webhook de estado |
| **Notificaciones** | Resend (HTTP API) | Emails de handoff y feedback negativo |

## Despliegue

- **Backend**: Railway (Docker)
- **Frontend**: Vercel
- **Base de datos + Auth**: Supabase
- **Emails**: Resend

## Seguridad y límites

- Rate limiting por plan: 10 preguntas/día (demo), 500/día (paid).
- Aislamiento multi-tenant: cada organización solo accede a sus propios documentos y conversaciones.
- Claves de API (OpenAI, Groq, Gemini, Anthropic) configuradas en el backend, nunca expuestas al cliente.
