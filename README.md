# Multi-Agent Cold Email Pipeline (LangGraph)

Pipeline **no conversacional** que recibe la URL de una empresa, analiza su negocio y genera (y opcionalmente envía) un **Cold Email** personalizado. Orquestado con **LangGraph** sobre tres agentes especializados.

```
URL  ─►  Agente 1 (Scraper + Data Engineer)
            │   Apify Website Content Crawler + limpieza opcional con LLM
            ▼
         Agente 2 (Profiler)
            │   Analiza puntos de dolor, tecnología, carencias y cliente ideal
            ▼
         Agente 3 (Copywriter + ReAct)
            │   Redacta el cold email y, si hay destinatario, lo envía vía SMTP
            ▼
        Cold Email final + estado de envío
```

---

## Stack

| Capa       | Tecnología                                                                 |
|------------|----------------------------------------------------------------------------|
| Backend    | FastAPI · LangGraph 1.x · LangChain 1.x (`init_chat_model`) · OpenAI GPT-4o |
| Scraping   | Apify `website-content-crawler`                                            |
| Email      | SMTP (Gmail App Password) vía `@tool` + `create_react_agent`               |
| Frontend   | React 19 · Vite · TailwindCSS 4 · shadcn/ui                                |
| Deploy     | Docker (backend) · Vercel (frontend)                                       |

---

## Arquitectura del grafo

`Backend/grafo.py` compila un `StateGraph` con el flujo lineal `START → DataEngineer → Profiler → Copywriter → END`. El estado compartido (`SalesState`):

```python
class SalesState(TypedDict, total=False):
    run_id: str
    target_url: str
    max_crawl_pages: int
    max_crawl_depth: int
    skip_cleaning: bool
    cleaned_data: list[dict[str, Any]]
    profile_data: str
    my_service_info: str
    company_tone: str
    final_email: str
    recipient_email: str
    email_sent_status: str
```

Cada agente persiste su salida en `Backend/Salidas de los Agentes/<run_id>/` para inspección posterior.

### Agentes

- **Agente 1 — Scraper & Data Engineer** (`agents/Agente 1 - Scraper y Data Engineer.py`)
  Crawlea la URL con Apify. Si `skip_cleaning=False`, pasa el texto por un LLM (`gpt-4o-mini`) para normalizarlo; si `True` (default), entrega el markdown raw — más rápido, evita timeouts.

- **Agente 2 — The Profiler** (`agents/Agente 2 - The Profiler.py`)
  Cadena LCEL con `gpt-4o` que produce un **Perfil de Negocio** estructurado: dolores, tecnología, carencias y cliente ideal.

- **Agente 3 — The Copywriter** (`agents/Agente 3 - The Copywriter.py`)
  Agente ReAct (`create_react_agent`) con `gpt-4o` que redacta el cold email y, si recibe `recipient_email`, llama la tool `send_email` (`tools/send_email.py`).

---

## Setup local

### Backend

```bash
conda create -n LangGraph-Perfilador-Copywriter python=3.11 -y
conda activate LangGraph-Perfilador-Copywriter

cd Backend
cp .env.example .env       # rellena tus claves
pip install -r requirements.txt
uvicorn main:app --reload
```

API disponible en `http://127.0.0.1:8000`.

### Frontend

```bash
cd Frontend
npm install
npm run dev
```

---

## Variables de entorno (`Backend/.env`)

```env
OPENAI_API_KEY=sk-...
APIFY_API_TOKEN=apify_api_...

# Envío SMTP (Gmail App Password — NO tu contraseña normal)
EMAIL_REMITENTE=tucorreo@gmail.com
APP_PASSWORD_GMAIL=xxxx xxxx xxxx xxxx
```

---

## API

### `POST /process`

Ejecuta el pipeline completo.

**Request:**
```json
{
  "target_url": "https://webtilia.com",
  "recipient_email": "prospecto@empresa.com",
  "max_crawl_pages": 10,
  "max_crawl_depth": 3,
  "skip_cleaning": true,
  "my_service_info": "Soluciones de IA para empresas",
  "company_tone": "profesional y cercano"
}
```

**Response:**
```json
{
  "final_email": "Hola...",
  "profile_data": "## Puntos de Dolor...",
  "target_url": "https://webtilia.com",
  "run_id": "20260521_002443",
  "email_sent_status": "Email enviado exitosamente a prospecto@empresa.com"
}
```

### `GET /health`
Health check (`{ "status": "ok" }`).

---

## LangGraph Studio (opcional)

```bash
pip install "langgraph-cli[inmem]"
cd Backend
langgraph dev --allow-blocking
```

El grafo está expuesto como `agent` en `Backend/langgraph.json`.

---

## Docker (backend)

```bash
docker buildx build \
  --platform linux/amd64 \
  -t <tu-usuario>/app-langgraph-no-conversacional-backend:latest \
  --push .
```

El `Dockerfile` parte de `python:3.11-slim` y arranca con `uvicorn main:app --host 0.0.0.0 --port 8000`.

---

## Estructura

```
.
├── Backend/
│   ├── agents/
│   │   ├── Agente 1 - Scraper y Data Engineer.py
│   │   ├── Agente 2 - The Profiler.py
│   │   └── Agente 3 - The Copywriter.py
│   ├── tools/
│   │   └── send_email.py
│   ├── Salidas de los Agentes/   # outputs por run_id
│   ├── grafo.py                  # StateGraph + compile
│   ├── main.py                   # FastAPI app
│   ├── langgraph.json
│   ├── Dockerfile
│   └── requirements.txt
├── Frontend/                     # React + Vite + Tailwind
└── Comandos.md
```

---

## Notas de implementación

- **Compatibilidad `apify_client` 3.x.** El SDK devuelve un objeto Pydantic `Run` en lugar de dict; el agente 1 accede a `run.default_dataset_id` (con fallback a dict para versiones previas).
- **LangChain v1.** Los tres agentes inicializan modelos con `init_chat_model("openai:gpt-4o", ...)` siguiendo la guía oficial (`docs.langchain.com/oss/python/langchain/models`).
- **Sin estado conversacional.** Cada request es un run independiente — no hay checkpointer ni memoria persistente.
