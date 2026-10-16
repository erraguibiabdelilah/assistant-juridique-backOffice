# Stack technique

- Python 3.11+, FastAPI, Pydantic v2, uvicorn
- **LangChain** (obligatoire) pour : loaders, text splitters, embeddings, vector store Qdrant, LLM
- Qdrant (vecteurs) via `langchain-qdrant`
- MongoDB via `pymongo` (ou `motor` si async)
- Embeddings : configurable (HuggingFace multilingue `BAAI/bge-m3` ou `intfloat/multilingual-e5-large`, ou OpenAI)
- LLM : configurable via variable d'env (OpenAI, Groq, Ollama, Anthropic)
- Chunking : **par taille** (`RecursiveCharacterTextSplitter` ou `CharacterTextSplitter`, `chunk_size` et `chunk_overlap` configurables)
- OCR images : `pytesseract` (langue `fra`) + Pillow
- UI : Streamlit
- Tests : pytest, httpx
- Config : `pydantic-settings` + fichier `.env`, jamais de secret en dur
- Style : typage complet, docstrings, logs structurés, gestion d'erreurs explicite

## Conventions
- Architecture en couches : `api/` -> `services/` -> `repositories/` -> clients DB.
- Tous les endpoints sous `/api/v1`.
- Réponses d'erreur JSON `{ "error": { "code": str, "message": str } }`.
