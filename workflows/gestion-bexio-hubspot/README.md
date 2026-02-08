# Gestion Bexio-HubSpot

Synchronisation automatique entre Bexio (comptabilite suisse) et HubSpot CRM, avec suivi des factures et relances automatiques.

## Architecture

```
Toutes les 15 min ─┬──> Contacts Bexio ──> Filtre recents ──> HubSpot (create/update)
                   │
                   └──> Factures Bexio ──> Filtre recentes ──> Switch statut
                                                                ├──> Google Sheets (log)
                                                                └──> Si payee: Email remerciement

Toutes les 24h ──> Factures Bexio ──> Filtre impayees 30j+ ──> Info client ──> Email relance
```

## 3 flux automatises

### Flux 1 : Synchronisation Contacts (toutes les 15 min)

| Etape | Description |
|-------|-------------|
| 1 | Recupere les 100 derniers contacts modifies depuis Bexio |
| 2 | Filtre ceux modifies dans les 15 dernieres minutes |
| 3 | Verifie si le contact existe dans HubSpot (par email) |
| 4 | Si OUI : mise a jour du contact HubSpot |
| 5 | Si NON : creation avec champ custom `bexio_id` |

### Flux 2 : Suivi des Factures (toutes les 15 min)

| Etape | Description |
|-------|-------------|
| 1 | Recupere les 100 dernieres factures modifiees depuis Bexio |
| 2 | Filtre celles modifiees dans les 15 dernieres minutes |
| 3 | Log dans Google Sheets (ID, numero, titre, statut, montant, contact, echeance) |
| 4 | Si statut = 9 (payee) : recupere les infos client et envoie un email de remerciement |

### Flux 3 : Relances Factures Impayees (toutes les 24h)

| Etape | Description |
|-------|-------------|
| 1 | Recupere toutes les factures depuis Bexio |
| 2 | Filtre : statut impaye (codes 7, 8, 17) ET echeance > 30 jours |
| 3 | Recupere les infos du client associe |
| 4 | Envoie un email de relance professionnel |

## Installation

### 1. Importer le workflow

- Ouvrir n8n
- **Workflows > Import from File**
- Selectionner `workflow.json`

### 2. Configurer les credentials

| Credential | Type | Utilisation |
|------------|------|-------------|
| **Bexio API** | Bexio API | Acces contacts et factures |
| **HubSpot Account** | HubSpot API | Sync contacts |
| **Google Sheets** | Google Sheets OAuth2 | Log des factures |
| **Gmail Account** | Gmail OAuth2 | Emails remerciement et relance |

### 3. Personnaliser

- **Google Sheet ID** : Remplacer `YOUR_GOOGLE_SHEET_ID` dans le node "Logger dans Google Sheets"
- **Codes statut Bexio** : Verifier que les codes 7, 8, 17 correspondent a votre configuration Bexio
- **Templates emails** : Adapter les textes de remerciement et relance

### 4. Activer

Activer le workflow une fois toutes les credentials configurees.

## Nodes (18 fonctionnels + 4 notes)

| Categorie | Node | Type |
|-----------|------|------|
| Trigger | Verification Toutes les 15 min | scheduleTrigger |
| Trigger | Verification Quotidienne Relances | scheduleTrigger |
| Contacts | Recuperer Contacts Bexio | httpRequest |
| Contacts | Filtrer Contacts Recents | code |
| Contacts | Verifier Contact HubSpot | hubspot |
| Contacts | Contact Existe ? | if |
| Contacts | Creer Contact HubSpot | hubspot |
| Contacts | Mettre a Jour Contact HubSpot | hubspot |
| Factures | Recuperer Factures Bexio | httpRequest |
| Factures | Filtrer Factures Recentes | code |
| Factures | Type de Changement | switch |
| Factures | Logger dans Google Sheets | googleSheets |
| Factures | Recuperer Info Client (Paiement) | httpRequest |
| Factures | Email de Remerciement | gmail |
| Relances | Recuperer Toutes Factures | httpRequest |
| Relances | Filtrer Impayees 30+ Jours | code |
| Relances | Recuperer Info Client (Relance) | httpRequest |
| Relances | Email de Relance | gmail |

## APIs utilisees

- **Bexio** : `/2.0/contact`, `/2.0/kb_invoice`
- **HubSpot** : Contact search, create, update
- **Google Sheets** : Append/update rows
- **Gmail** : Send email
