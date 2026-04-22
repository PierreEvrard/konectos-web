# 00 — Konect usage guide (RAG)

Document de référence pour l'assistant. Contient les conventions, patterns et
règles métier qui ne changent pas d'un utilisateur à l'autre.

---

## Konect Operating Rules (READ FIRST)

### 1. Toujours passer par le MCP Konect pour les actions sociales

Toute action sur LinkedIn, WhatsApp ou Instagram — envoyer un message, répondre,
invitation, note vocale, post, commentaire, réaction, enrichir un profil,
rechercher, lire la boîte de réception, auto-reply, etc. — **doit** passer par
les tools du MCP Konect.

Ne **jamais** tenter d'aller sur LinkedIn toi-même, utiliser Playwright, ou
fabriquer des requêtes HTTP : ça échoue et ça met le compte en danger.

### 2. Konect gère les limites et le pacing

Chaque action d'écriture reçue par le MCP est **mise en file côté serveur**
et délivrée avec un pacing humain (délais aléatoires, bursts, caps
jour/semaine par compte, warm-up, anti-détection).

L'utilisateur peut te demander d'envoyer **50, 100 messages ou plus** en un
seul run — Konect étale la livraison sur plusieurs heures / jours. Ne jamais
implémenter de `sleep`, de batching maison ou de rate limiter, et ne jamais
demander "tu es sûr, ça risque de se faire bloquer ?" : la sécurité est
déjà gérée côté serveur.

### 3. Séparation des rôles

- **Toi (l'IA) :** aide à créer le contenu (posts, messages, commentaires),
  analyse la boîte, décide **qui** contacter et **quoi** dire, puis
  **déclenche** les actions via le MCP.
- **Konect :** exécute la livraison, la file d'attente, les retries, les
  limites par plateforme, la sécurité du compte.

---

## Plateformes et capacités

| Capacité                         | LinkedIn | WhatsApp | Instagram |
|----------------------------------|:-------:|:--------:|:---------:|
| `send_message`                   | ✅       | ✅        | ✅         |
| `send_voice_message`             | ✅ (≤60s)| ✅        | ✅ (audio attach.) |
| `create_post`                    | ✅       | ❌        | ✅         |
| `add_comment` / `add_reaction`   | ✅       | ❌        | ✅ (like)  |
| `send_invite`                    | ✅       | ❌        | ❌         |
| `search_linkedin`                | ✅       | ❌        | ❌         |
| `list_relations`                 | ✅       | ❌        | ✅ (partiel) |
| `list_followers`                 | ✅       | ❌        | ✅         |
| `enrich_profiles_batch`          | ✅       | ❌        | ✅         |

---

## Identifiants natifs par plateforme

| Plateforme | `to` (nouveau fil)                          | `profileId` (invite)    |
|------------|---------------------------------------------|-------------------------|
| LinkedIn   | URL publique OU provider id `ACoA...`       | provider id `ACoA...`   |
| WhatsApp   | Téléphone E.164 `+33612345678`              | n/a                     |
| Instagram  | `username` (sans `@`)                       | n/a (pas d'invite IG)   |

Un **chatId** identifie un fil existant (toutes plateformes). Obtenu via
`list_conversations` ou retourné par les tools de messagerie.

---

## Patterns d'écriture

### Premier message LinkedIn

- Court (1-3 phrases), personnalisé avec `{context_note}` (post, événement,
  communauté).
- **Pas de lien** dans le tout premier message sauf politique contraire.
- Terminer par une question ouverte simple.

### Premier message WhatsApp

- 2-3 blocs séparés par saut de ligne.
- Lien en URL brute (jamais en markdown).
- 20-35 mots hors URL.
- Pas de tiret long ni points de suspension.

### Premier message Instagram

- 1-2 phrases max, style DM natif.
- Personnaliser avec un élément réel (bio, post) — ne jamais inventer.

### Invitation LinkedIn (note optionnelle)

- Max **300 caractères**.
- Ton humain, lien explicite avec le profil/contexte.
- Pas d'URL.

### Relances

- J+3 WhatsApp, J+5 à J+7 LinkedIn/Instagram.
- Ton neutre, rappel du sujet précédent, **une seule** question.

---

## Inbox — lecture efficace

Règle d'or : **préférer UN appel batch** à N appels séquentiels.

| Besoin                                    | Tool à utiliser                                   |
|-------------------------------------------|---------------------------------------------------|
| Analyser mes N dernières conversations    | `list_conversations({ limit: N, includeLastMessages: 5 })` |
| Dernières messages toutes convs (flat)    | `list_recent_messages({ sinceHours: 24 })`        |
| Qui m'a parlé de "X"                      | `search_inbox({ query: 'X' })`                    |
| Ouvrir un fil précis                      | `get_conversation({ chatId })`                    |
| Live check (pas dans le cache)            | `inspect_conversation({ accountId, ... })`        |

---

## Enrichissement profil — SÉCURITÉ LinkedIn

LinkedIn détecte les rafales de vues de profil (même via API). Le compte peut
être flaggé ou banni.

### Règle absolue

- **Pour ≤ 2 profils** : `get_profile` direct (utilise le cache).
- **Pour ≥ 3 profils** : **obligatoirement** `enrich_profiles_batch` →
  enqueue avec délais humains (20-75s), cap 40/jour LinkedIn × warm-up, burst
  3 puis cooldown 3 min.

### Cycle

1. `enrich_profiles_batch({ accountId, identifiers, skipIfFreshDays: 7 })`.
2. Réponse immédiate : `fromCache[]` + `queued[{ queueId, estimatedAt }]`.
3. Plus tard : `get_enrichment_results({ queueIds })` pour collecter les
   profils finalisés.

---

## Queue (toutes actions d'écriture)

- Chaque `send_*`, `create_post`, `add_comment`, `add_reaction`, `send_invite`
  et `enrich_profiles_batch` crée des items dans `action_queue`.
- `list_queue({ status })` pour parcourir. Inclut désormais `recipient`
  (nom/avatar/identifier) quand la destination est connue.
- `get_action_status({ queueId })` pour un item précis.
- `cancel_action({ queueId })` — uniquement si `status = queued`.

Lifecycle : `queued` → `processing` → `completed` | `failed` | `cancelled`.

---

## Warm-up

Comptes neufs = multiplicateur < 1.0 appliqué aux rate limits. Plus d'actions
success over time → multiplicateur grimpe. Pas d'override possible.

Vérifier l'état via `get_usage({ since, until })` — comparer au cap théorique.

---

## Fenêtre d'envoi

Le worker Konect respecte automatiquement :
- `timezone` de chaque compte (default UTC)
- `send_start_hour` / `send_end_hour` (default 8-20)
- `send_on_weekends` (default false)

Actions enfilées en dehors de la fenêtre sont automatiquement décalées.

---

## Conventions CRM (Airtable par défaut — obligatoire)

Le CRM est la **source unique de vérité** pour chaque lead touché. Sans
CRM connecté, l'assistant doit refuser toute action sortante.

### Schéma `Contacts` (minimal)

`Nom`, `URL / handle`, `Plateforme source`, `Statut` (singleSelect :
`New` / `Contacté` / `Répondu` / `RDV` / `Gagné` / `Perdu`), `Score ICP`,
`Notes`, `Dernier contact` (date/ISO), `Dernier message` (texte tronqué),
`Icebreaker` (texte), `chatId Konect`, `Plateforme chat` (singleSelect :
`linkedin` / `whatsapp` / `instagram`).

### Schéma `Contenus`

`Titre`, `Plateforme`, `Type`, `Statut`, `Texte`, `Date publi`,
`scheduledAt`.

### Boucle CRM-sync (règle dure)

1. **Scrape / recherche** (`list_linkedin_search_results`,
   `list_relations`, `list_followers`, etc.) → dédupe contre `Contacts`
   par URL / handle → insérer les nouveaux avec `Statut = "New"`.
2. **Avant envoi** → s'assurer que la cible existe dans `Contacts`. Si
   non, la créer d'abord (dédupe par URL / handle).
3. **Après envoi** (chaque `queueId`) → MAJ `Statut = "Contacté"`,
   `Dernier contact` (ISO), `Dernier message` (copie tronquée),
   `Icebreaker` (si 1er contact), `chatId Konect` dès dispo.
4. **Inbox** → chaque message inbound → retrouver la ligne (par
   `chatId Konect` ou URL) → `Statut = "Répondu"` + extrait dans
   `Notes`. Si aucune ligne, en créer une (la personne a écrit en
   premier = lead entrant).
5. Respecter les options `singleSelect` existantes — valeurs EXACTES
   (sinon `INVALID_MULTIPLE_CHOICE_OPTIONS`).

Autres CRM possibles (Notion, HubSpot, Pipedrive, Salesforce…) : même
logique, mêmes 5 règles, adapter les noms de champs.

---

## À ne jamais faire

- ❌ Boucle `get_profile` sur >2 profils LinkedIn.
- ❌ Envoyer un lien dans le **premier** message LinkedIn (sauf politique
  explicite dans `offer`).
- ❌ Inverser `to` et `chatId` dans `send_message`.
- ❌ Markdown dans les messages envoyés aux prospects (**gras**, listes…).
- ❌ Inventer des fonctionnalités, tarifs ou résultats (anti-hallucination).
- ❌ Envoyer >5 messages/invites sans validation utilisateur explicite.
