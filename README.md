# N8N Workflows

Collection de workflows n8n automatises, deployes sur une instance self-hosted (Hostinger).

## Workflows

| Workflow | Description | Nodes |
|----------|-------------|-------|
| [Gestion Bexio-HubSpot](workflows/gestion-bexio-hubspot/) | Sync contacts, suivi factures, relances auto | 18 |
| [Lead Scoring IA](workflows/lead-scoring-ia/) | Qualification de leads par IA + routing intelligent | 27 |
| [RAG Picone](workflows/rag-picone/) | Chatbot IA sur vos documents (Google Drive + Pinecone) | 14 |
| [Traitement Factures PDF](workflows/traitement-factures-pdf/) | Extraction auto des factures PDF par IA + alertes Slack | 13 |

## Structure du repo

```
workflows/
├── gestion-bexio-hubspot/
│   ├── workflow.json          # Importable dans n8n
│   └── README.md              # Documentation + installation
│
├── lead-scoring-ia/
│   ├── workflow.json          # Importable dans n8n
│   └── README.md              # Documentation + installation
│
├── rag-picone/
│   ├── workflow.json          # Importable dans n8n
│   └── README.md              # Documentation + installation
│
├── traitement-factures-pdf/
│   ├── workflow.json          # Importable dans n8n
│   └── README.md              # Documentation + installation
│
└── [prochain-workflow]/
    ├── workflow.json
    └── README.md
```

## Comment utiliser

1. Ouvrir votre instance n8n
2. Aller dans **Workflows > Import from File**
3. Selectionner le `workflow.json` du workflow souhaite
4. Configurer les credentials selon le README du workflow
5. Activer le workflow

## Instance n8n

- **Type** : Self-hosted
- **Hebergement** : Hostinger
