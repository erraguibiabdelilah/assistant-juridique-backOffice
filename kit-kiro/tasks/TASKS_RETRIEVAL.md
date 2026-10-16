# Tâches — Personne A : Retrieval + Back-office

## Phase 0 — Socle (J1)
- [ ] Créer le dépôt, `docker-compose.yml` (Qdrant + MongoDB), `.env.example`
- [ ] Squelette FastAPI, `config.py` (pydantic-settings), logging, `/health`
- [ ] Valider avec B le fichier `api-contract.md`
- [ ] Connexions Mongo et Qdrant + création des index (script `init_db.py`)

## Phase 1 — Bases de connaissances (J1-J2)
- [ ] Schémas Pydantic KB, repository Mongo, CRUD `/knowledge-bases`
- [ ] Tests CRUD

## Phase 2 — Ingestion (J2)
- [ ] Loader PDF (+ détection scanné -> OCR)
- [ ] Loader DOCX (+ conversion DOC)
- [ ] Loader image (pytesseract fra, confiance moyenne)
- [ ] `factory.py` (choix du loader), extraction des `type_specific` par type
- [ ] Nettoyage du texte
- [ ] Chunking par taille (`RecursiveCharacterTextSplitter`), paramètres par KB
- [ ] Embeddings LangChain configurables, traitement par batch
- [ ] Écriture double Mongo + Qdrant, champs BM25
- [ ] Statuts (`pending/processing/indexed/failed`), tâche de fond, déduplication par hash
- [ ] Endpoints documents (upload, liste, détail, suppression cohérente, réindexation)
- [ ] Tests : chaque loader, chunking, suppression cohérente

## Phase 3 — Recherche (J2-J3)
- [ ] Recherche sémantique Qdrant + filtres (`kb_id`, `doc_type`)
- [ ] BM25 sur Mongo (index texte + scoring BM25)
- [ ] Hybride RRF, dédoublonnage, seuil
- [ ] Tests RRF et BM25 avec petit corpus

## Phase 4 — Conversations + augmented prompt (J3)
- [ ] CRUD conversations et messages (contrat)
- [ ] Récupération de l'historique (fenêtre)
- [ ] Réécriture de requête (LLM, activable)
- [ ] `prompt_builder.py` (system, historique, contexte numéroté, question finale)
- [ ] Endpoint `POST /retrieve` conforme au contrat
- [ ] Enregistrement `retrieved_chunk_ids` dans la conversation
- [ ] Livrer rapidement une première version de `/retrieve` pour débloquer B

## Phase 5 — Back-office Streamlit (J3)
- [ ] Page Bases de connaissances
- [ ] Page Documents (upload multiple, statuts, détails par type)
- [ ] Explorateur de chunks
- [ ] Testeur de recherche (mode, scores, augmented prompt)
- [ ] Page Conversations

## Phase 6 — Finalisation (J4)
- [ ] Intégration avec generation-service
- [ ] Jeu de test (PDF, DOCX, image) + `seed.py`
- [ ] README, exemples curl
- [ ] Couverture de tests des modules clés

## Critères d'acceptation
- Un PDF, un DOCX et une image s'indexent avec des `type_specific` différents.
- Chaque chunk existe dans Mongo ET Qdrant (mêmes `chunk_id`).
- `/retrieve` fonctionne en `semantic`, `bm25` et `hybrid`.
- Une question de suivi (« et en cas de licenciement ? ») est correctement réécrite grâce à l'historique.
- Supprimer un document supprime ses chunks dans les deux bases.
