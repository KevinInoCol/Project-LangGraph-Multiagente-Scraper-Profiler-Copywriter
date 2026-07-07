# Regla: LangChain / LangGraph → consultar SIEMPRE la MCP de docs

Cada vez que vayas a **escribir, modificar o revisar código de LangChain o LangGraph**
(imports, agentes, chains/LCEL, tools, grafos, `StateGraph`, `init_chat_model`,
`create_agent`, `create_react_agent`, prompts, etc.), **antes** de escribir el código
consulta la documentación oficial vía la MCP `docs-langchain`:

- `mcp__docs-langchain__search_docs_by_lang_chain` — buscar por tema/palabra clave.
- `mcp__docs-langchain__query_docs_filesystem_docs_by_lang_chain` — leer el contenido de una página concreta.

Motivo: LangChain/LangGraph cambian de API con frecuencia (v0 → v1). No confíes en la
memoria; verifica la **sintaxis vigente** en las docs antes de proponer o aplicar cambios.

Aplica a todo el árbol del proyecto (Backend `agents/`, `tools/`, `grafo.py`, etc.).
No aplica a código que no sea de LangChain/LangGraph.
