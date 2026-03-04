# 🧠 Taller Práctico: Construye un AI Agent con RAG y Supabase

## 🎯 Objetivo del Taller

En este taller construirás un agente inteligente que:

- Indexe documentos en Supabase usando embeddings.
- Use un AI Agent en lugar de Q&A Chain.
- Responda preguntas usando RAG.
- Diseñe su propio System Prompt.
- Controle alucinaciones del modelo.
- (Opcional) Registre conversaciones.

---

# 🏗 Escenario

TechNova necesita un asistente interno que pueda responder preguntas sobre:

- Políticas internas
- Vacaciones
- Trabajo remoto
- Productos
- Seguridad

El conocimiento estará almacenado en un archivo `.md` y será indexado en Supabase usando embeddings.

El asistente NO debe responder desde conocimiento general.
Debe usar únicamente la base vectorial.

---

# 🔹 Parte 1 – Configuración de Supabase

## 1️⃣ Activar extensión pgvector

En el SQL Editor ejecutar:

create extension if not exists vector;

---

## 2️⃣ Crear tabla para documentos RAG

create table documentos_rag (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  embedding vector(1536),
  content text NULL,
  metadata jsonb NULL
);

---

## 3️⃣ Crear función de búsqueda por similitud

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

---

## ✅ Resultado esperado

La base de datos debe estar lista para:

- Recibir embeddings.
- Ejecutar búsquedas por similitud.
- Retornar contenido relevante ordenado por similaridad.

---

# 🔹 Parte 2 – Flujo de Indexación

Manual Trigger  
↓  
HTTP Request (archivo .md)  
↓  
Default Data Loader  
↓  
Recursive Text Splitter  
↓  
OpenAI Embeddings  
↓  
Supabase Vector Store  

Configuración:

- Tabla: documentos_rag
- Función RPC: match_documents
- Campo embedding: embedding
- Campo contenido: content

---

# 🔹 Parte 3 – Construcción del AI Agent

When Chat Message Received  
↓  
AI Agent  
   ↙  
Retriever Tool  
   ↓  
Supabase Vector Store  
↓  
Chat Output  

---

# ⚙ Configuración del Agente

El agente debe tener:

- Modelo GPT (el usado en clase)
- Tool: Vector Store Retriever
- Top K: 3
- Similarity Search

---

# 🧠 Tarea Clave: Crear el System Prompt

Los estudiantes deben diseñar el System Prompt del agente.

El prompt debe:

1. Obligar al agente a usar el Retriever.
2. Prohibir inventar información.
3. Responder solo con información del contexto.
4. Indicar qué hacer si no encuentra información.

---

## 📌 Requisitos del System Prompt

Debe incluir reglas como:

- Usar SIEMPRE la herramienta de búsqueda antes de responder.
- No responder desde conocimiento general.
- Si no hay contexto suficiente, decir exactamente:
  "No tengo información suficiente para responder eso."
- Responder de forma clara y profesional.

El diseño del prompt es parte de la evaluación.

---

# 🔹 Parte 4 – Pruebas Obligatorias

Cada estudiante debe probar:

1. ✅ Pregunta que esté en el documento.
2. ❌ Pregunta fuera del documento.
3. 🤯 Pregunta que combine dos secciones.
4. 🔍 Pregunta ambigua.


---

# 🧠 Preguntas de Reflexión

- ¿Qué ventaja tiene un Agent frente a Q&A Chain?
- ¿Qué riesgos tiene un agente mal configurado?
- ¿Cómo se controlan las alucinaciones?
- ¿Qué pasa si el embedding model cambia de dimensión?
- ¿Cuándo usarías Agent vs Workflow determinístico?

---

# 🎓 Resultado Esperado

Al finalizar el taller el estudiante debe:

- Entender la diferencia entre RAG con cadena y con agente.
- Diseñar un System Prompt robusto.
- Controlar el uso de herramientas en un agente.
- Implementar búsqueda vectorial en Supabase.
- Construir un flujo más cercano a producción.
