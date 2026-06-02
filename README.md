# Canal Cero Agent

> Agente conversacional multi-agente con RAG para análisis de Marketing Mix Modeling (MMM) en Retail Ecommerce chileno.

---

## Autores

| Nombre | Universidad | Programa |
|--------|-------------|----------|
| Bastián Muhaded Ibar | Universidad Adolfo Ibáñez | Programa de IA |
| Rodrigo Saez Perez | Universidad Adolfo Ibáñez | Programa de IA |
| Daniela de Quevedo Rodríguez | Universidad Adolfo Ibáñez | Programa de IA |

---

## Objetivo del RAG

El sistema implementa **Retrieval-Augmented Generation (RAG)** para permitir que el agente responda preguntas específicas del negocio basándose en documentos reales del Proyecto Canal Cero, en lugar de depender solo del conocimiento general del LLM.

**Problema que resuelve:** Un modelo de lenguaje general no conoce las métricas, decisiones y datos específicos del cliente. El RAG permite "inyectar" ese conocimiento privado en cada respuesta.

**Flujo RAG:**
```
Pregunta del usuario
       ↓
Búsqueda semántica en Chroma (k=5 chunks más relevantes)
       ↓
Contexto recuperado + Pregunta → LLM
       ↓
Respuesta fundamentada en los documentos
```

---

## 🏗️ Arquitectura Implementada

```
┌─────────────────────────────────────────────────────┐
│                   USUARIO                           │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│              AGENTE ORQUESTADOR                     │
│         (LangGraph ReAct Agent)                     │
└────────────┬─────────────────────┬──────────────────┘
             ↓                     ↓
┌────────────────────┐  ┌──────────────────────────┐
│  WORKER 1          │  │  WORKER 2                │
│  Búsqueda          │  │  Consulta SQL            │
│  Semántica (RAG)   │  │  (Trazabilidad)          │
│  ChromaDB          │  │  SQLite                  │
└────────────────────┘  └──────────────────────────┘
             ↓
┌─────────────────────────────────────────────────────┐
│              AGENTE FISCALIZADOR                    │
│   Valida fuentes · Detecta PII · Verifica calidad  │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│           RESPUESTA VALIDADA AL USUARIO             │
└─────────────────────────────────────────────────────┘
```

---

## 🧩 Componentes Definidos

| Componente | Descripción | Tecnología |
|------------|-------------|------------|
| **Vector Store** | Almacena embeddings de los documentos de negocio | ChromaDB |
| **Agente Orquestador** | Recibe preguntas, decide qué worker usar y genera la respuesta final | LangGraph ReAct |
| **Worker Búsqueda Semántica** | Recupera los chunks más relevantes del vectorstore | LangChain + Chroma |
| **Worker SQL** | Registra cada interacción para trazabilidad | SQLite + SQLAlchemy |
| **Agente Fiscalizador** | Valida la respuesta antes de entregarla al usuario | GPT-4o-mini |
| **Base de Datos** | Tabla `interactions` con id, timestamp, query, response, tokens, latency | SQLite |

---

## 🤖 Modelos LLM Utilizados

| Modelo | Uso | Proveedor |
|--------|-----|-----------|
| `gpt-4o-mini` | Agente Orquestador y Workers | OpenAI |
| `gpt-4o-mini` | Agente Fiscalizador | OpenAI |
| `text-embedding-3-small` | Generación de embeddings para RAG | OpenAI |

---

## 🏛️ Arquitectura de Agentes

### Agente Orquestador
- **Rol:** Agente principal que recibe la pregunta del usuario
- **Framework:** LangGraph `create_react_agent`
- **Herramientas disponibles:** `worker_busqueda_semantica`, `worker_sql`
- **Comportamiento:** Siempre busca en documentos antes de responder, siempre registra la interacción

### Worker 1 · Búsqueda Semántica
- **Rol:** Especialista en recuperación de información
- **Método:** Similitud coseno sobre embeddings (KNN con k=5)
- **Fuente de datos:** Documentos del Proyecto Canal Cero en ChromaDB
- **Precisión@5:** 100% en pruebas con 10 consultas reales

### Worker 2 · Consulta SQL
- **Rol:** Especialista en trazabilidad y registro
- **Operaciones:** `registrar` nuevas interacciones y `consultar` historial
- **Base de datos:** SQLite (`canal_cero.db`)
- **Campos registrados:** id, timestamp, query, response, tokens, latency

### Agente Fiscalizador
- **Rol:** Validador de calidad antes de entregar respuesta al usuario
- **Criterios de validación:**
  - ✅ ¿La respuesta aborda la pregunta directamente?
  - ✅ ¿Cita fuentes de los documentos?
  - ✅ ¿Contiene PII o datos sensibles?
  - ✅ ¿Comete el error de sumar `meta_conv_value + google_conv_value`?
  - ✅ ¿Habla de TikTok sin advertir sobre normalización de `tiktok_cost_usd`?
- **Output:** `{ok, issues, corrected}`

---

## 📂 Estructura del Proyecto

```
Canal-Cero-Agent/
├── src/
│   └── Canal_Cero_Agent.ipynb   # Notebook principal con todo el agente
├── .env.example                  # Variables de entorno requeridas
├── .gitignore                    # Archivos ignorados por Git
├── requirements.txt              # Dependencias del proyecto
└── README.md                     # Este archivo
```

---

## ⚙️ Setup

1. Clona el repositorio:
```bash
git clone https://github.com/bastianmuhaded-lang/Canal-Cero-Agent.git
```

2. Instala dependencias:
```bash
pip install -r requirements.txt
```

3. Copia `.env.example` a `.env` y agrega tus API keys:
```bash
cp .env.example .env
```

4. Ejecuta el notebook principal:
```
src/Canal_Cero_Agent.ipynb
```

---

## 🛠️ Stack Tecnológico

- **LangChain + LangGraph** — Framework de agentes
- **OpenAI GPT-4o-mini** — Modelo de lenguaje
- **ChromaDB** — Vector Store para RAG
- **SQLite + SQLAlchemy** — Trazabilidad de interacciones
- **Python 3.10+**

---

## 📊 Métricas

| Métrica | Valor |
|---------|-------|
| Precisión@5 búsqueda semántica | 100% (10/10 consultas) |
| Chunks por documento | ~500 tokens con 50 overlap |
| Documentos en vectorstore | 6 archivos .txt |
| Fiscalizador OK rate | ✅ Validado |

4. Ejecuta el notebook principal:
```bash
src/Canal_Cero_Agent.ipynb
```
