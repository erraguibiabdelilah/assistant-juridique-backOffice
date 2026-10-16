# Mini RAG Assistant Juridique — Guide du projet

Projet en binôme, 2 services indépendants qui communiquent par API REST.

| | Personne A | Personne B |
|---|---|---|
| Service | **Retrieval + Back-office** | **Generation + Assistant** |
| Dossier | `retrieval-service/` | `generation-service/` |
| Port API | 8001 | 8002 |
| UI | Back-office (Streamlit) | Chat assistant (Streamlit) |
| Prompt Kiro | `prompts/PROMPT_1_RETRIEVAL.md` | `prompts/PROMPT_2_GENERATION.md` |
| Tâches | `tasks/TASKS_RETRIEVAL.md` | `tasks/TASKS_GENERATION.md` |

## Architecture

```
 Back-office (Streamlit)            Assistant (Streamlit)
        |                                   |
        v                                   v
 +-----------------------+  REST   +------------------------+
 |  retrieval-service    | <-----> |  generation-service    |
 |  FastAPI :8001        |         |  FastAPI :8002         |
 |  - ingestion docs     |         |  - appelle /retrieve   |
 |  - chunking par taille|         |  - LLM (LangChain)     |
 |  - embeddings         |         |  - streaming, citations|
 |  - recherche          |         |  - sauvegarde réponse  |
 |  - augmented prompt   |         +------------------------+
 +-----------+-----------+
             |
     +-------+--------+
     v                v
  Qdrant           MongoDB
  (vecteurs)       knowledge_bases / documents / chunks / conversations
```

## Flux d'une question
1. L'utilisateur pose une question dans l'assistant.
2. `generation-service` appelle `POST /api/v1/retrieve` (query + conversation_id + kb_ids).
3. `retrieval-service` : récupère l'historique de la conversation, fait la recherche (sémantique Qdrant, BM25 Mongo, ou hybride), construit l'**augmented prompt** et le renvoie avec les chunks et sources.
4. `generation-service` envoie l'augmented prompt au LLM (LangChain), renvoie la réponse en streaming.
5. `generation-service` enregistre le tour de conversation via `POST /api/v1/conversations/{id}/messages`.

## Règles de collaboration
- **Le contrat d'API (`steering/api-contract.md`) est la seule source de vérité.** Toute modification se discute à deux.
- B peut démarrer avec un **mock** de l'API retrieval (fourni dans les tâches) pendant que A développe.
- Un dépôt Git, deux dossiers, une branche par personne (`feature/retrieval`, `feature/generation`).
- Un `docker-compose.yml` unique à la racine (Qdrant + MongoDB), maintenu par A.

## Structure du dépôt
```
legal-rag/
├── docker-compose.yml
├── .kiro/steering/        <- copier le dossier steering/ ici (product.md, tech.md, api-contract.md)
├── retrieval-service/
└── generation-service/
```

## Comment utiliser Kiro avec ce kit
1. Copier `steering/*.md` dans `.kiro/steering/` à la racine du dépôt (Kiro les charge comme contexte permanent).
2. Chacun ouvre une session **Spec** dans Kiro et colle son prompt (`prompts/PROMPT_X.md`).
3. Kiro génère `requirements.md`, `design.md`, `tasks.md` : comparer avec `tasks/TASKS_X.md` et ajuster.
4. Exécuter les tâches une par une, en validant chaque étape.
5. Vérifier à chaque jalon avec les critères d'acceptation en fin de fichier de tâches.

## Jalons communs
- J1 : docker-compose OK, contrat d'API figé.
- J2 : A ingère un PDF et `/retrieve` renvoie des chunks ; B a un chat fonctionnel sur mock.
- J3 : intégration réelle bout en bout.
- J4 : tests, README, démo.
