# Contrat d'API entre retrieval-service (8001) et generation-service (8002)

Source de vérité. Toute modification doit être validée par les deux personnes.

## 1. Conversations (propriétaire : retrieval-service)

### POST /api/v1/conversations
Body : `{ "kb_ids": ["kb_123"], "title": "optionnel" }`
Réponse 201 : `{ "conversation_id": "conv_abc", "created_at": "ISO8601" }`

### GET /api/v1/conversations/{conversation_id}
Réponse : `{ "conversation_id", "kb_ids", "title", "messages": [Message], "created_at", "updated_at" }`

### GET /api/v1/conversations?limit=20&offset=0
Liste des conversations (id, title, updated_at).

### POST /api/v1/conversations/{conversation_id}/messages
Body : `{ "role": "user"|"assistant", "content": "...", "sources": [Source] (optionnel) }`
Réponse 201 : `{ "message_id": "msg_1" }`

### DELETE /api/v1/conversations/{conversation_id}  -> 204

`Message` = `{ "message_id", "role", "content", "sources", "created_at" }`

## 2. Retrieval

### POST /api/v1/retrieve
Body :
```json
{
  "query": "Quel est le délai de préavis d'un bail commercial ?",
  "conversation_id": "conv_abc",
  "kb_ids": ["kb_123"],
  "top_k": 5,
  "mode": "hybrid",
  "history_window": 6,
  "filters": { "doc_type": ["pdf", "docx", "image"] }
}
```
`mode` : `semantic` | `bm25` | `hybrid` (défaut `hybrid`, fusion RRF).

Réponse 200 :
```json
{
  "query": "...",
  "rewritten_query": "...",
  "conversation_id": "conv_abc",
  "chunks": [
    {
      "chunk_id": "chk_1",
      "text": "...",
      "score": 0.83,
      "retrieval_source": "semantic|bm25|hybrid",
      "document": { "document_id": "doc_1", "filename": "code_travail.pdf", "doc_type": "pdf" },
      "metadata": { "page_number": 12 }
    }
  ],
  "history": [ { "role": "user", "content": "..." } ],
  "augmented_prompt": {
    "system": "Tu es un assistant juridique...",
    "messages": [ { "role": "user|assistant", "content": "..." } ],
    "context": "[1] (code_travail.pdf, p.12) ...\n[2] ...",
    "final_user_message": "Contexte:\n...\n\nQuestion: ..."
  },
  "latency_ms": 120
}
```
`Source` = `{ "chunk_id", "document_id", "filename", "doc_type", "page_number"?, "excerpt" }`

## 3. Santé
`GET /health` sur chaque service -> `{ "status": "ok" }`

## 4. Mock pour B
Tant que A n'a pas fini, B utilise `mock_retrieval/` (FastAPI) qui renvoie une réponse fixe respectant ce schéma.
