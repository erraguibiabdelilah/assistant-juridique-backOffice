# Produit : Mini RAG Assistant Juridique

Assistant de questions-réponses sur des documents juridiques (lois, contrats, jurisprudence, décrets) fournis par un administrateur.

## Utilisateurs
- **Administrateur (back-office)** : crée des bases de connaissances, importe des documents (PDF, DOCX/DOC, images), suit l'indexation, teste la recherche.
- **Utilisateur (assistant)** : pose des questions en français, obtient une réponse sourcée, avec historique de conversation.

## Principes
- Les réponses s'appuient **uniquement** sur les chunks récupérés ; si le contexte est insuffisant, l'assistant le dit.
- Toujours citer les sources (document, page/section, extrait).
- Disclaimer : l'assistant fournit une information, pas un conseil juridique.
- Langue par défaut : français.
