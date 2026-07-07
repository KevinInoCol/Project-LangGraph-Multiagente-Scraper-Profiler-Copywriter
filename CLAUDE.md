# CLAUDE.md

Guía para trabajar en este proyecto (pipeline multiagente no conversacional con LangGraph).

## Reglas

@.claude/rules/langchain-docs.md

## Contexto del proyecto

- **Backend**: FastAPI + LangGraph. Grafo lineal `START → DataEngineer → Profiler → Copywriter → END` (`Backend/grafo.py`).
- **Agentes** en `Backend/agents/` (nombres con espacios; se cargan vía `importlib`).
- **Frontend**: React 19 + Vite + Tailwind (`Frontend/`).
- Provider LLM actual: **OpenAI** (`init_chat_model("openai:gpt-4o", ...)`).
