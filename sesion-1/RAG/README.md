# Taller RAG — Bot de Telegram con N8N + Supabase + OpenAI

En este taller construiremos un bot de Telegram capaz de responder preguntas sobre un documento, usando **RAG (Retrieval-Augmented Generation)**:

```
Documento de texto (URL)
        ↓
N8N lo descarga y vectoriza
        ↓
Supabase almacena los embeddings
        ↓
Usuario pregunta por Telegram
        ↓
N8N busca los fragmentos más relevantes
        ↓
GPT-4o genera la respuesta con contexto
        ↓
Bot responde en Telegram
```

---

## Requisitos previos

- Cuenta en [N8N](https://n8n.io) → [Registro](https://app.n8n.cloud/register)
- Cuenta en [Supabase](https://supabase.com) → [Registro](https://supabase.com/dashboard/sign-up)
- Cuenta en [OpenAI](https://platform.openai.com) (para embeddings y GPT-4o)
- Cuenta en [Telegram](https://telegram.org)

### Documento de ejemplo
- [`caso_nexopay.txt`](./caso_nexopay.txt) — Caso de uso que usaremos para el RAG

---

## Parte 1 — Preparar Supabase

### 1.1 Habilitar la extensión pgvector

1. En el dashboard de Supabase, ve a **Database → Extensions**.
2. Busca `vector` y actívala (Enable extension).

### 1.2 Crear la tabla de documentos

Abre el **SQL Editor** de Supabase y ejecuta:

```sql
create table documentos_rag (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  embedding vector(1536),
  content text NULL,
  metadata jsonb NULL
);
```

### 1.3 Crear la función de búsqueda semántica

En el mismo **SQL Editor**, ejecuta:

```sql
create or replace function match_documents (
  match_count int,
  query_embedding vector(1536),
  filter jsonb DEFAULT '{}'
)
returns table (
  id uuid,
  content text,
  similarity float
)
language sql stable
as $$
  select
    docs.id,
    docs.content,
    1 - (docs.embedding <=> query_embedding) as similarity
  from documentos_rag docs
  where metadata @> filter
  order by docs.embedding <=> query_embedding
  limit match_count;
$$;
```

### 1.4 Obtener las credenciales de Supabase

1. En el menú izquierdo ve a **Project Settings → Data API**.
2. Copia el **Project URL** — lo usarás como **Host** en N8N.
3. Revela y copia la clave **service_role** — la usarás como **Service Role Secret** en N8N.

---

## Parte 2 — Flujo de Ingesta (cargar el documento)

Este workflow descarga el documento y lo almacena vectorizado en Supabase. Solo se ejecuta una vez.

### Nodos del flujo de ingesta

```
[Manual Trigger] → [HTTP Request] → [Supabase Vector Store]
```

### 2.1 Trigger manual

- Agrega un nodo **"When clicking 'Test workflow'"** (Manual Trigger).

### 2.2 Nodo HTTP Request

- Tipo: **HTTP Request**
- Method: `GET`
- URL:
```
https://raw.githubusercontent.com/mateozacar/n8n-clases/refs/heads/main/sesion-1/caso_nexopay.txt
```

### 2.3 Nodo Supabase Vector Store (insertar)

- Tipo: **Supabase Vector Store**
- Operation Mode: `Insert Documents`
- Table Name: `documentos_rag`
- **Credenciales:** clic en "New Credentials" e ingresa el Host y Service Role Secret del paso 1.4
- **Embedding Model:** OpenAI → `text-embedding-3-small`
- **Document:** Default Data Loader

Ejecuta el workflow con el botón **"Test workflow"**. Los fragmentos del documento quedarán guardados en Supabase.

---

## Parte 3 — Flujo del Bot (responder preguntas)

Este workflow se activa cada vez que alguien le escribe al bot de Telegram.

### Nodos del flujo del bot

```
[Telegram Trigger] → [IF (texto no vacío)] → [Question & Answer Chain] → [Telegram Send Message]
                                                       ↑
                                            [Supabase Vector Store (Retriever)]
                                            [OpenAI Embeddings]
```

### 3.1 Crear el bot de Telegram

1. Abre Telegram y busca **@BotFather**.
2. Escribe `/newbot` y sigue las instrucciones.
3. Al final BotFather te dará un **Access Token** — guárdalo.

### 3.2 Nodo Telegram Trigger

- Tipo: **Telegram Trigger**
- Clic en **"New Credentials"** e ingresa el Access Token del bot.
- Activa el trigger y envíale un mensaje a tu bot para probar que llega.

### 3.3 Nodo IF (filtrar mensajes vacíos)

- Tipo: **IF**
- En Conditions, agrega una condición con el valor:
```
{{ $json.message.text }}
```
- Condición: **is not empty**

Conecta la rama **true** al siguiente nodo.

### 3.4 Nodo Question and Answer Chain

- Tipo: **Question and Answer Chain**
- **Prompt Source (User Message):** `Define Below` *(activa el modo expression)*
- **Text:**
```
{{ $json.message.text }}
```
- **Modelo:** OpenAI → `gpt-4o`

### 3.5 Conectar el Retriever (Supabase)

1. Sal de la configuración del modelo y entra a la sección **Retriever**.
2. Elige **Vector Store Retriever**.
3. Elige **Supabase Vector Store**.
   - Operation Mode: `Retrieve Documents`
   - Table Name: `documentos_rag`
   - En **Add Option** → Function Name: `match_documents`
4. Selecciona el modelo de embeddings: OpenAI → `text-embedding-3-small`

### 3.6 Nodo Telegram Send Message

- Tipo: **Telegram Send Message**
- **Chat ID:**
```
{{ $('Telegram Trigger').item.json.message.chat.id }}
```
- **Text:**
```
{{ $json.response.text }}
```

---

## Parte 4 — Activar y probar

1. Guarda el workflow del bot y **actívalo** (toggle en la esquina superior derecha).
2. Abre Telegram y escríbele una pregunta sobre el documento a tu bot.
3. En pocos segundos debe responder con información del caso NexoPay.

---

## Solución de problemas comunes

| Problema | Solución |
|---|---|
| Bot no responde | Verifica que el workflow esté **activado** (no solo en modo test) |
| Error de credenciales Supabase | Confirma que usaste la clave `service_role`, no la `anon` |
| Error al crear la tabla | Verifica que la extensión `vector` esté habilitada antes de correr el SQL |
| Respuesta vacía del bot | Revisa que el flujo de ingesta haya corrido correctamente y haya filas en `documentos_rag` |
| Error de embeddings | Verifica que tu API Key de OpenAI tenga saldo disponible |
