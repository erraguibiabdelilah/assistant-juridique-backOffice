# PROMPT KIRO (docs) n°1 — Génère la documentation du Retrieval Service + Back-office

> À coller dans Kiro (mode chat/vibe) à la racine du dépôt. Kiro doit **créer les fichiers**, pas seulement les afficher. Ne génère pas encore de code applicatif.

Tu es un architecte logiciel et rédacteur technique senior. Je construis, avec un binôme, un **mini RAG assistant juridique** en deux services. Je suis responsable du service **retrieval-service** (Retrieval + Back-office). L'autre personne développe le **generation-service**, qui consommera mon API REST.

## Contexte du projet
- Assistant de questions-réponses en français sur des documents juridiques (lois, contrats, jurisprudence).
- Stack : Python 3.11, FastAPI, Pydantic v2, **LangChain** (loaders, splitters, embeddings, QdrantVectorStore, LLM), **Qdrant**, **MongoDB**, Streamlit pour le back-office, pytest.
- Mon service : (1) back-office : bases de connaissances + ingestion de documents PDF / DOC-DOCX / images (OCR) ; (2) retrieval : recherche sémantique (Qdrant), BM25 (MongoDB), hybride (RRF), association de la conversation aux chunks pertinents, construction de l'**augmented prompt**.
- Chunking **par taille** (`RecursiveCharacterTextSplitter`, chunk_size/overlap configurables). Chaque chunk est stocké **à la fois dans MongoDB** (texte, métadonnées, données BM25) **et dans Qdrant** (vecteur + payload).
- MongoDB : collections `knowledge_bases`, `documents`, `chunks`, `conversations`. Les champs spécifiques diffèrent selon le type de document (pdf : page_count, author, is_scanned ; docx : section_count, headings ; image : width, height, ocr_confidence_avg).
- Mon service est propriétaire des **conversations** ; le generation-service y enregistre les messages et appelle `POST /api/v1/retrieve`.

## Ce que tu dois produire (fichiers à créer)

### 1. Steering (contexte permanent) dans `.kiro/steering/`
- `product.md` : vision, utilisateurs (administrateur, utilisateur final), périmètre, hors-périmètre, principes (réponses sourcées, pas de conseil juridique, français par défaut), glossaire (KB, chunk, augmented prompt, RRF…).
- `tech.md` : stack, versions, conventions de code, config par `.env`, gestion d'erreurs, logs, stratégie de tests.
- `structure.md` : arborescence cible de `retrieval-service/` (api, services, loaders, repositories, schemas, backoffice, tests, scripts) et rôle de chaque dossier.

### 2. Spec Kiro dans `.kiro/specs/retrieval-service/`
- `requirements.md` : exigences au format **EARS** (« WHEN… THE SYSTEM SHALL… ») groupées par user stories : gestion des KB, ingestion par type, chunking, double stockage, recherche semantic/bm25/hybrid, conversations, retrieve + augmented prompt, back-office, déduplication, suppression cohérente, erreurs. Chaque exigence a des critères d'acceptation testables.
- `design.md` : architecture (diagramme Mermaid), flux d'ingestion et de recherche (diagrammes de séquence Mermaid), schémas MongoDB (JSON d'exemple par collection et par type de document), payload Qdrant, index, algorithme BM25 sur MongoDB (en précisant la limite : pas de BM25 natif en Community), RRF, réécriture de requête, structure de l'augmented prompt, gestion d'erreurs, stratégie de test.
- `tasks.md` : plan d'implémentation en **cases à cocher** `- [ ]`, ordonné et incrémental (socle -> KB -> ingestion -> recherche -> conversations/retrieve -> back-office -> finalisation). Chaque tâche : titre, description courte, fichiers concernés, exigences couvertes (ex. `_Requirements: 2.1, 2.3_`), critère de fin vérifiable. Prévoir une première livraison rapide de `/retrieve` pour débloquer mon binôme.

### 3. Documentation d'API dans `docs/`
- `docs/api-retrieval.md` : référence lisible de **tous** les endpoints sous `/api/v1` (KB, documents, chunks, conversations, retrieve, health) avec pour chacun : méthode, URL, paramètres, body JSON d'exemple, réponses de succès et d'erreur (codes), exemple `curl`.
- `docs/openapi.yaml` : spécification OpenAPI 3.1 cohérente avec `api-retrieval.md` (schémas réutilisables : Message, Source, Chunk, AugmentedPrompt, Error).
- `docs/api-contract.md` : **contrat d'interface avec le generation-service** (uniquement ce qu'il consomme : conversations, retrieve, knowledge-bases en lecture, health). C'est la source de vérité partagée ; le rédiger de façon stable et sans ambiguïté.
- `docs/mock-guide.md` : comment le binôme peut mocker mon API (réponse fixe de `/retrieve` conforme au schéma).

## Contraintes
- Tout en **français**, sauf identifiants de code, noms de champs et URLs.
- Cohérence stricte entre `requirements.md`, `design.md`, `tasks.md` et `docs/` : mêmes noms de champs, mêmes endpoints.
- Pas de code applicatif : uniquement de la documentation, des schémas JSON/YAML et des diagrammes Mermaid.
- Si une décision est ambiguë, choisis l'option la plus simple et liste-la dans une section « Hypothèses et questions ouvertes » à la fin de `design.md`.

À la fin, résume la liste des fichiers créés et les questions ouvertes. N'implémente rien avant ma validation.
