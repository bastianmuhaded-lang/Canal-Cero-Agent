# Canal Cero Agent

Agente conversacional multi-agente para el Proyecto Canal Cero, desarrollado para un cliente de Retail Ecommerce chileno.

## ¿Qué hace?

Responde preguntas sobre Marketing Mix Modeling (MMM), canales digitales y métricas de marketing, usando una arquitectura de agentes con RAG sobre documentos de negocio.

## Arquitectura

- **Orquestador**: Agente principal que recibe preguntas y delega a workers
- **Worker Búsqueda Semántica**: Recupera información desde Chroma (RAG)
- **Worker SQL**: Registra cada interacción en SQLite
- **Fiscalizador**: Valida respuestas antes de entregarlas al usuario

## Setup

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
src/Canal_Cero_Agent.ipynb
