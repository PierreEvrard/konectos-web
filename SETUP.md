# KonectOS Web — Setup (5 min)

Bienvenue. Ce dossier contient tout ce qu'il te faut pour transformer un projet
**Claude.ai** (ou ChatGPT) en assistant d'automatisation sociale branché sur
**Konect** (LinkedIn, WhatsApp, Instagram) + **Airtable** — sans `.env`, sans
terminal, sans curl. Tout passe par les MCPs.

## Pré-requis

- Un compte **Konect** actif (`https://mykonect.ai`) avec au moins 1 compte
  social connecté (LinkedIn, WhatsApp ou Instagram).
- Un compte **Claude.ai** avec accès aux Projets (ou **ChatGPT** Projects avec
  les connecteurs, ou Cursor / Claude Code). Ce guide prend Claude comme
  référence — l'adaptation à ChatGPT est identique.
- Une base **Airtable** (ou un compte pour en créer une au besoin).

---

## 1. Créer le projet Claude (1 min)

1. Va sur `https://claude.ai` → **Projects** → **Create project**.
2. Nomme-le "KonectOS".
3. Dans **Custom instructions** : colle intégralement le contenu de
   [`claude-project-instructions.md`](./claude-project-instructions.md).

## 2. Uploader la knowledge base (1 min)

Dans le projet, onglet **Knowledge**, uploade les fichiers suivants du dossier
[`knowledge/`](./knowledge/) :

- `00-konect-usage-guide.md` — conventions + patterns + règles métier + schéma CRM
- `01-persona-template.md` — à personnaliser (ton profil + ton ICP)
- `02-offer-template.md` — à personnaliser (offre + CTA)
- `04-templates-messages.md` — à personnaliser (icebreakers, relances)

**Les fichiers `template` sont à éditer AVANT upload** pour remplacer les
`[À CONFIGURER]` par tes valeurs. Tu peux aussi les uploader tels quels puis
demander `/onboarding` à l'assistant qui te guidera section par section.

## 3. Connecter le MCP Konect (1 min)

1. Dans le projet Claude → **Connectors** (ou **Integrations** selon version) →
   **Add custom connector**.
2. URL : `https://mykonect.ai/api/mcp`
3. Claude déclenche automatiquement le flow OAuth 2.1 PKCE → connecte-toi, puis
   **sélectionne les comptes Konect** que l'assistant est autorisé à piloter
   (tu peux révoquer à tout moment depuis le dashboard Konect).
4. Fin. Le serveur expose ~30 tools (message, voice, post, invite, search,
   inbox batching, enrichissement…). Version actuelle : **2.2.0**.

## 4. Connecter le MCP Airtable (1 min)

Utilise le **connecteur officiel Airtable** (recommandé) ou un MCP Airtable
tiers. Tu auras besoin de ton **Airtable API key** (depuis
`airtable.com/account`) et du **base ID** de ta base KonectOS.

Tables conseillées (créées par `/onboarding` si besoin) :
- **Contacts** : `Nom`, `Entreprise`, `Titre`, `Localisation` (ville),
  `ID` (provider ID Konect du lead), `LinkedIn URL`, `Instagram`,
  `WhatsApp`, `Plateforme source`, `Source` (LinkedIn Search / Sales
  Navigator / Relations / Followers / Post Engagement / Instagram
  Search / Instagram Followers / WhatsApp Import / Inbound / Manuel),
  `Relation` (1 / 2 / 3 / 3+), `Statut`, `Score ICP`, `Notes`, `Dernier
  contact`, `Dernier message`, `Icebreaker`, `chatId Konect`,
  `Plateforme chat`.
- **Contenus** : Titre, plateforme, type, statut, texte, date publi,
  `scheduledAt`.

> **À savoir** : l'assistant applique une règle dure de CRM-sync — chaque
> lead scrapé est poussé dans `Contacts` (dédupliqué), et chaque envoi met
> à jour la ligne correspondante. Pas de CRM connecté = pas d'action
> sortante. Voir `claude-project-instructions.md` section "CRM-sync".

## 5. Premier message (1 min)

Ouvre une conversation dans le projet et écris simplement :

```
setup
```

L'assistant exécutera `/onboarding` et te demandera les infos manquantes
(agentName, langue, focus business, CTA, timezone, etc.). Tes réponses mettent
à jour les fichiers knowledge.

Exemples de requêtes ensuite :

- « résume mes 20 dernières conversations LinkedIn »
- « trouve 25 founders SaaS à Paris, score-les, et envoie un icebreaker aux
  top 5 après validation »
- « répond à mes DMs LinkedIn non lus en gardant mon style »
- « programme un post LinkedIn pour demain 9h sur [sujet] »

## Troubleshooting

- **"account not connected"** : va dans `https://mykonect.ai` → **Accounts** →
  reconnecter le compte.
- **L'IA boucle sur `get_profile`** : rappelle-lui d'utiliser
  `enrich_profiles_batch` pour >2 profils LinkedIn.
- **Rate limit** : normal — le warm-up est actif. `get_usage` pour voir où tu
  en es. Plus un compte vieillit sans alerte, plus les caps augmentent.
- **Un tool manque** : le MCP Konect filtre les tools par plateforme connectée
  (ex. `send_invite` apparaît uniquement si LinkedIn est connecté).

## Mise à jour

Les tools MCP sont **mis à jour côté serveur**. Tu n'as rien à refaire :
Claude re-fetchera la liste à la prochaine session. Les fichiers knowledge de
ce dossier peuvent recevoir des patchs — suis le repo GitHub pour les versions.
