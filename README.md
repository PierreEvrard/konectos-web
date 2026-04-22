# KonectOS Web

Template prêt-à-coller pour transformer un **projet Claude.ai** (ou ChatGPT
Projects) en assistant d'automatisation sociale branché sur **Konect**
(LinkedIn, WhatsApp, Instagram) + **Airtable**.

> Pour la version Claude Code / terminal, voir le repo
> [KonectOS](https://github.com/…/KonectOS) (filesystem + curl + `.env`).

## Contenu

- [`SETUP.md`](./SETUP.md) — guide d'installation en 5 minutes.
- [`claude-project-instructions.md`](./claude-project-instructions.md) —
  instructions à coller dans le champ "Project instructions" (≈ 2k tokens).
- [`knowledge/`](./knowledge/) — 4 fichiers markdown à uploader dans la
  knowledge base du projet (RAG à la demande) : usage guide, persona,
  offre, templates de messages.
- [`onboarding-prompt.md`](./onboarding-prompt.md) — prompt initial pour se
  configurer par conversation.

## Architecture

```
+-----------------------+      MCP JSON-RPC       +-----------------+
|   Projet Claude.ai    | ----------------------> |  MCP Konect     |
|   - Instructions 2k   |                         |  (OAuth 2.1)    |
|   - Knowledge base    | <---------------------- |  ~30 tools      |
+-----------------------+                         +-----------------+
           |                                                 |
           | MCP Airtable                                    |
           v                                                 v
+-----------------------+                         +-----------------+
|   Airtable            |                         |  Konect backend |
|   (Contacts, Content) |                         |  + Queue + RL   |
+-----------------------+                         +-----------------+
```

## Pré-requis utilisateur

- Compte **Konect** avec au moins 1 compte social connecté.
- Compte **Claude.ai** avec Projets (ou ChatGPT Projects).
- Base **Airtable** + API key.

## Versions

- Template : 1.0.0
- MCP Konect requis : ≥ 2.2.0 (batch inbox, enrichissement safe LinkedIn,
  labels plateforme)
