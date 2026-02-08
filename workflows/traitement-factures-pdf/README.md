# Traitement automatique des factures PDF

Extraction automatique des donnees de factures PDF recues par email, avec stockage Google Sheets et alertes Slack.

## Architecture

```
Toutes les 5 min ──> Emails IMAP ──> Filtre PDF ──> Separe pieces jointes ──> Filtre .pdf
                                                                                    │
                                                                               Extract PDF Text
                                                                                    │
                                                                          OpenAI: Extract Invoice Data
                                                                                    │
                                                                               Format Data
                                                                                    │
                                                                          Validation donnees OK ?
                                                                                    │
                                                                          Google Sheets (log)
                                                                                    │
                                                                          Slack: Notification
                                                                                    │
                                                                          Montant > 5000 ?
                                                                              │
                                                                        Slack: Alerte Manager
```

## Fonctionnement

| Etape | Node | Description |
|-------|------|-------------|
| 1 | Every 5 minutes | Declencheur toutes les 5 minutes |
| 2 | Recuperer emails avec PDF | Lit les emails entrants via IMAP |
| 3 | Filter: Has PDF attachment | Garde uniquement les emails avec pieces jointes |
| 4 | Separer les pieces jointes | Eclate chaque piece jointe en item separe |
| 5 | Filter: PDF only | Ne garde que les fichiers .pdf |
| 6 | Extract PDF Text | Extrait le texte brut du PDF (Code node) |
| 7 | OpenAI: Extract Invoice Data | IA extrait les donnees structurees (numero, montant, date, fournisseur...) |
| 8 | Format Data | Formate les donnees pour Google Sheets |
| 9 | Validate: Has required data | Verifie que les champs obligatoires sont presents |
| 10 | Save to Google Sheets | Enregistre la facture dans le tableur |
| 11 | Slack: Notification generale | Notifie l'equipe de la nouvelle facture |
| 12 | Check: Montant > 5000 | Verifie si le montant depasse le seuil |
| 13 | Slack: Alerte Manager | Alerte le manager si montant > 5000 |

## Nodes (13 fonctionnels + 5 notes)

| Node | Type |
|------|------|
| Every 5 minutes | scheduleTrigger |
| Recuperer emails avec PDF | emailReadImap |
| Filter: Has PDF attachment | if |
| Separer les pieces jointes | splitOut |
| Filter: PDF only | if |
| Extract PDF Text | code |
| OpenAI: Extract Invoice Data | openAi (natif) |
| Format Data | set |
| Validate: Has required data | if |
| Save to Google Sheets | googleSheets |
| Slack: Notification generale | slack |
| Check: Montant > 5000 | if |
| Slack: Alerte Manager | slack |

## Installation

### 1. Importer le workflow

- Ouvrir n8n
- **Workflows > Import from File**
- Selectionner `workflow.json`

### 2. Configurer les credentials

| Credential | Type | Utilisation |
|------------|------|-------------|
| **IMAP Email** | IMAP | Lecture des emails entrants |
| **OpenAI API** | OpenAI API | Extraction des donnees de facture |
| **Google Sheets OAuth2** | Google Sheets | Stockage des factures |
| **Slack API** | Slack | Notifications + alertes |

### 3. Personnaliser

- **Seuil alerte manager** : Modifier le montant de 5000 dans le node "Check: Montant > 5000"
- **Channel Slack** : Configurer les channels dans les 2 nodes Slack
- **Google Sheet** : Renseigner l'ID du Google Sheet et le nom de la feuille
- **Serveur IMAP** : Configurer le serveur email a surveiller

### 4. Activer

Activer le workflow une fois toutes les credentials configurees.

## Donnees extraites par OpenAI

| Champ | Description |
|-------|-------------|
| Numero facture | Reference unique de la facture |
| Fournisseur | Nom de l'emetteur |
| Date facture | Date d'emission |
| Date echeance | Date limite de paiement |
| Montant HT | Montant hors taxes |
| TVA | Montant de la TVA |
| Montant TTC | Montant total |
| Devise | EUR, CHF, USD, etc. |
