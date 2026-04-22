# /onboarding — prompt de configuration

> À coller dans la conversation quand l'utilisateur veut configurer son projet.
> L'assistant te posera les questions et mettra à jour les fichiers knowledge.

Je veux lancer `/onboarding` pour configurer KonectOS. Guide-moi section par
section, remplace les `[À CONFIGURER]` dans les fichiers knowledge suivants :

1. `01-persona-template.md` — mon profil + mon ICP
2. `02-offer-template.md` — mon offre, CTA, règles prix
3. `04-templates-messages.md` — mes templates (si je veux personnaliser
   au-delà des exemples)

Règles du flow :
- Une seule question à la fois.
- Après chaque réponse, reformule en 1 ligne pour confirmer.
- Quand une section est complète, résume ce qui a été ajouté avant de passer
  à la suivante.
- Commence par vérifier via `list_accounts` quels comptes Konect sont
  connectés, et mentionne les comptes manquants (LinkedIn / WhatsApp /
  Instagram) que je pourrais vouloir ajouter plus tard.
- Vérifie aussi qu'un MCP CRM est bien connecté (Airtable par défaut) —
  sinon, demande-moi de le brancher avant de prospecter quoi que ce soit,
  car toute action sortante doit dédupliquer et logger dans le CRM.
- À la fin : propose la suite logique (ex. `/prospect`, `/icebreaker`).
