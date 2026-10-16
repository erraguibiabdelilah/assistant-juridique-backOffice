# PROMPT KIRO n°1 — Retrieval Service + Back-office

> Contexte à charger : `.kiro/steering/product.md`, `tech.md`, `api-contract.md`

Tu es un ingénieur backend senior. Crée le dossier `retrieval-service/` d'un **mini RAG assistant juridique**. Ce service est la partie **Retrieval + Back-office** ; un autre service (`generation-service`) le consommera via REST. Respecte strictement `api-contract.md`.

## Objectif
1. **Back-office** : gérer des bases de connaissances et ingérer des documents hétérogènes (PDF, DOC/DOCX, images).
2. **Retrieval** : recherche par similarité sémantique (et BM25 / hybride), association de la conversation aux chunks pertinents, construction de l'**augmented prompt**.

## Stack imposée
Python 3.11, FastAPI, **LangChain** (loaders, splitters, embeddings, QdrantVectorStore), **Qdrant**, **MongoDB**, Streamlit pour l'UI back-office, pytest.

## Bases de données

### MongoDB — collections
- `knowledge_bases` : `_id`, `name`, `description`, `domain` (ex. droit du travail), `language`, `created_at`, `doc_count`, `chunk_count`, `embedding_model`, `chunk_size`, `chunk_overlap`.
- `documents` : `_id`, `kb_id`, `filename`, `doc_type` (`pdf|docx|image`), `status` (`pending|processing|indexed|failed`), `error`, `file_hash` (déduplication), `size_bytes`, `created_at`, et **`type_specific`** dont les champs **diffèrent selon le type** :
  - pdf : `page_count`, `author`, `title`, `creation_date`, `is_scanned`
  - docx : `section_count`, `headings` (liste), `author`, `last_modified`
  - image : `width`, `height`, `format`, `ocr_engine`, `ocr_lang`, `ocr_confidence_avg`
- `chunks` : `_id`, `kb_id`, `document_id`, `chunk_index`, `text`, `char_count`, `qdrant_point_id`, `doc_type`, `metadata` (champs différents selon le type : pdf -> `page_number`; docx -> `section_title`, `heading_path`; image -> `ocr_confidence`, `bbox` optionnel), `tokens` (liste de termes normalisés), `bm25` (`{ "terms": {terme: tf}, "length": n }`), `created_at`. Index texte Mongo sur `text`, index sur `kb_id`, `document_id`.
- `conversations` : `_id`, `kb_ids`, `title`, `messages` [{ `message_id`, `role`, `content`, `sources`, `retrieved_chunk_ids`, `created_at` }], `created_at`, `updated_at`.

### Qdrant
- Une collection `legal_chunks` (ou une par KB, configurable), vecteur dense, distance cosine.
- Payload : `chunk_id`, `kb_id`, `document_id`, `doc_type`, `filename`, `chunk_index`, `page_number?`.
- Index de payload sur `kb_id` et `doc_type` pour le filtrage.

**Chaque chunk est stocké à la fois dans MongoDB (texte + métadonnées + données BM25) ET dans Qdrant (vecteur + payload)**, liés par `chunk_id` / `qdrant_point_id`. Prévoir une suppression cohérente (supprimer un document supprime ses chunks dans les deux bases).

## Pipeline d'ingestion
1. Upload multipart `POST /api/v1/knowledge-bases/{kb_id}/documents` (statut `pending`, traitement en tâche de fond).
2. Détection du type par extension + MIME ; rejet des types non supportés ; limite de taille configurable.
3. Extraction via **LangChain loaders** :
   - PDF : `PyPDFLoader` (fallback OCR si page sans texte -> `is_scanned=true`)
   - DOCX : `Docx2txtLoader` ou `UnstructuredWordDocumentLoader`; DOC converti via LibreOffice si dispo
   - Image : OCR `pytesseract` (fra) -> `Document` LangChain
4. Nettoyage du texte (espaces, césures, en-têtes répétés).
5. **Chunking par taille** : `RecursiveCharacterTextSplitter(chunk_size, chunk_overlap)` configurables (défauts 1000 / 150). Aucun chunking sémantique.
6. Embeddings via LangChain (`Embeddings` configurable par env) en batch.
7. Écriture dans Qdrant ET MongoDB, calcul des données BM25, mise à jour du statut et des compteurs.
8. Idempotence : un `file_hash` déjà présent dans la KB est refusé (409).

## Recherche
- `semantic` : similarité cosinus Qdrant (avec filtre `kb_id`, `doc_type`).
- `bm25` : recherche textuelle dans MongoDB — pré-sélection via index texte `$text`, puis scoring BM25 (`rank_bm25` ou implémentation maison à partir du champ `bm25`) ; documenter la limite (MongoDB Community n'a pas de BM25 natif).
- `hybrid` (défaut) : fusion **Reciprocal Rank Fusion** des deux listes.
- Dédoublonnage, `top_k`, score normalisé, seuil minimal configurable.

## Conversation + retrieval
Pour `POST /api/v1/retrieve` :
1. Charger la conversation (créer si `conversation_id` absent selon le contrat).
2. Prendre les `history_window` derniers messages.
3. **Réécriture de la requête** (LangChain LLM, optionnelle/activable) pour la rendre autonome à partir de l'historique (`rewritten_query`).
4. Rechercher les chunks pertinents.
5. Construire l'**augmented prompt** : system (rôle assistant juridique, règle « réponds uniquement avec le contexte, cite [n], dis si insuffisant, pas un conseil juridique »), historique, bloc contexte numéroté `[n] (fichier, page)`, question finale.
6. Enregistrer les `retrieved_chunk_ids` sur le tour de conversation et renvoyer la réponse du contrat.

## API à livrer (sous /api/v1)
- KB : `POST/GET/PUT/DELETE /knowledge-bases`, `GET /knowledge-bases/{id}`
- Documents : upload, `GET .../documents`, `GET /documents/{id}` (statut + type_specific), `DELETE /documents/{id}`, `POST /documents/{id}/reindex`
- Chunks : `GET /documents/{id}/chunks`
- Conversations : endpoints du contrat
- `POST /retrieve`, `GET /health`

## Back-office (Streamlit, `backoffice/`)
Pages : Bases de connaissances (CRUD), Documents (upload multiple, statuts, détails type-spécifiques, suppression, réindexation), Explorateur de chunks, **Testeur de recherche** (saisir une requête, choisir le mode, voir chunks/scores/augmented prompt), Conversations (consultation).

## Structure attendue
```
retrieval-service/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── api/ (kb.py, documents.py, conversations.py, retrieve.py)
│   ├── services/ (ingestion.py, chunking.py, embedding.py, retrieval.py, bm25.py, prompt_builder.py)
│   ├── loaders/ (pdf.py, docx.py, image.py, factory.py)
│   ├── repositories/ (mongo.py, qdrant.py)
│   └── schemas/
├── backoffice/
├── tests/
├── docker-compose.yml  (qdrant + mongo)
├── requirements.txt
├── .env.example
└── README.md
```

## Exigences de qualité
- Tout est typé, configurable par `.env`, sans secret en dur.
- Tests pytest : chunking, chaque loader, RRF, BM25, prompt builder, endpoint `/retrieve` (avec mocks).
- README : installation, lancement, exemples curl.
- Fournir un jeu de documents de test (1 PDF, 1 DOCX, 1 image) et un script `scripts/seed.py`.

Génère d'abord `requirements.md`, `design.md` puis `tasks.md` (spec Kiro), et attends ma validation avant d'implémenter.
