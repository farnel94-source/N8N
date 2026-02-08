# Lead Scoring IA - Workflow n8n

Workflow de qualification automatique de leads avec intelligence artificielle.

## Architecture

```
Webhook (formulaire) ──┐
                       ├──> Normalisation ──> Clearbit (enrichissement) ──> OpenAI (scoring)
Email entrant (IMAP) ──┘                           │ fallback si erreur
                                                   ↓
                                          HubSpot CRM (create/update)
                                                   ↓
                                          Router selon Score
                                     ┌──────┼──────┼──────┐
                                   Spam   >80    50-80    <50
                                     │      │      │       │
                                  Exclure  │    Nurturing Newsletter
                                        ┌──┼──┐   (4 emails    only
                                     Slack │  Tâche  14 jours)
                                        Email urgent
                                      commercial HubSpot
```

## Nodes (27 fonctionnels + 5 notes)

| Etape | Node | Type |
|-------|------|------|
| Trigger | Webhook Lead Entrant | webhook |
| Trigger | Email Entrant (IMAP) | imapEmail |
| Normalisation | Normaliser Données Webhook | code |
| Normalisation | Normaliser Données Email | code |
| Réponse | Répondre 202 Accepted | respondToWebhook |
| Enrichissement | Enrichir via Clearbit | httpRequest |
| Enrichissement | Traiter Enrichissement (+ Fallback) | code |
| Scoring | Scoring IA (OpenAI) | openAi (natif) |
| Scoring | Extraire Score et Tags | code |
| CRM | Chercher Contact HubSpot | hubspot |
| CRM | Contact Existe dans HubSpot ? | if |
| CRM | Créer Contact HubSpot | hubspot |
| CRM | Mettre à Jour Contact HubSpot | hubspot |
| Routing | Préparer Routing | code |
| Routing | Router selon Score | switch |
| Spam | Exclure Spam (Log) | code |
| Hot Lead | Notification Slack Hot Lead | slack |
| Hot Lead | Email Commercial Senior | gmail |
| Hot Lead | Créer Tâche Urgente HubSpot | hubspot |
| Nurturing | Email 1 - Bienvenue | gmail |
| Nurturing | Attendre 3 jours | wait |
| Nurturing | Email 2 - Proposition valeur | gmail |
| Nurturing | Attendre 4 jours | wait |
| Nurturing | Email 3 - Cas client | gmail |
| Nurturing | Attendre 7 jours | wait |
| Nurturing | Email 4 - Offre finale | gmail |
| Newsletter | Taguer Newsletter Only | code |

## Installation

### 1. Importer le workflow

- Ouvrir n8n
- Aller dans **Workflows > Import from File**
- Sélectionner `lead-scoring-ia.json`

### 2. Configurer les credentials

| Credential | Type | Configuration |
|------------|------|---------------|
| **Clearbit API Key** | Header Auth | `Authorization: Bearer sk_xxxx` |
| **OpenAI Account** | OpenAI API | Clé API OpenAI |
| **HubSpot Account** | HubSpot API | Clé API HubSpot |
| **Slack Account** | Slack API | Bot Token (xoxb-xxx) |
| **Gmail Account** | Gmail OAuth2 | Compte Google OAuth2 |
| **IMAP Email** | IMAP | Serveur IMAP + identifiants |

### 3. Créer les propriétés custom HubSpot

Dans HubSpot, aller dans **Settings > Properties > Create property** :

| Propriété | Type | Description |
|-----------|------|-------------|
| `lead_score` | Number | Score IA (0-100) |
| `lead_segment` | Single-line text | PME ou Enterprise |
| `lead_intention` | Single-line text | achat, info, ou spam |
| `lead_categorie` | Single-line text | Hot Lead, Nurturing, Newsletter, Spam |
| `lead_resume_ia` | Multi-line text | Résumé IA du lead |

### 4. Personnaliser

- **Email commercial** : Remplacer `commercial@votreentreprise.com` dans le node "Email Commercial Senior"
- **Channel Slack** : Remplacer `#leads-chauds` dans le node "Notification Slack Hot Lead"
- **Templates emails** : Modifier le contenu des 4 emails de nurturing selon votre activité
- **Google Sheet ID** : Si applicable

### 5. Activer

Une fois tout configuré, activer le workflow dans n8n.

## Webhook API

### Endpoint
```
POST https://votre-instance-n8n.com/webhook/lead-scoring
```

### Body (JSON)
```json
{
  "prenom": "Jean",
  "nom": "Dupont",
  "email": "jean.dupont@entreprise.com",
  "entreprise": "Acme SA",
  "message": "Nous cherchons une solution pour automatiser notre CRM...",
  "source": "Formulaire"
}
```

### Réponse
```json
{
  "status": "accepted",
  "message": "Lead recu et en cours de traitement",
  "timestamp": "2026-02-08T12:00:00.000Z"
}
```

## Scoring IA - Critères

| Critère | Points |
|---------|--------|
| Email professionnel (domaine entreprise) | +20 |
| Entreprise identifiable avec site web | +15 |
| Taille entreprise > 50 employés | +15 |
| Message détaillé avec besoin concret | +25 |
| Fonction décisionnaire (CEO, CTO, VP) | +15 |
| Source LinkedIn | +10 |
| Secteur premium (tech, finance, santé) | +10 |
| Email personnel sans entreprise | -15 |
| Message vide ou très court | -20 |
| Spam/phishing détecté | = 0 |

## Séquence Nurturing (Score 50-80)

| Jour | Email | Contenu |
|------|-------|---------|
| J+0 | Bienvenue | Introduction + présentation services |
| J+3 | Proposition valeur | 3 raisons de choisir |
| J+7 | Cas client | Résultats concrets + chiffres |
| J+14 | Offre finale | Audit gratuit + dernière relance |

## Gestion des erreurs

- **Clearbit** : `continueOnFail: true` + fallback code node (pas de blocage si API down)
- **Normalisation** : try/catch dans chaque code node
- **OpenAI parsing** : fallback avec score par défaut si réponse invalide
- **HubSpot search** : `continueOnFail: true` pour gérer les erreurs API

## Routing

```
Score > 80 (Hot Lead)    → Slack + Email commercial + Tâche urgente HubSpot
Score 50-80 (Nurturing)  → Séquence 4 emails sur 14 jours
Score < 50 (Newsletter)  → Tag newsletter, pas d'action commerciale
Intention = spam         → Exclusion + log (priorité sur le score)
```
