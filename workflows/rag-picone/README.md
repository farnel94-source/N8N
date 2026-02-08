# RAG Picone

Chatbot IA de type RAG (Retrieval-Augmented Generation) qui repond aux questions en se basant sur vos documents Google Drive, avec stockage vectoriel Pinecone.

## Architecture

```
FLUX 1 - INGESTION (automatique)

Google Drive Trigger ──> Download file ──> Pinecone Vector Store
                                               ├── Embeddings OpenAI
                                               └── Default Data Loader
                                                       └── Recursive Text Splitter


FLUX 2 - CHAT (interactif)

Chat Trigger ──> AI Agent ──────────────────> Reponse
                    ├── OpenAI Chat Model (LLM)
                    ├── Simple Memory (historique)
                    └── Vector Store Tool (recherche)
                           ├── Pinecone Vector Store 2
                           │      └── Embeddings OpenAI 1
                           └── OpenAI Chat Model 1
```

## Flux 1 : Ingestion de documents

Quand un fichier est ajoute ou modifie sur Google Drive :

| Etape | Node | Description |
|-------|------|-------------|
| 1 | Google Drive Trigger | Detecte les nouveaux fichiers / modifications |
| 2 | Download file | Telecharge le contenu du fichier |
| 3 | Default Data Loader | Charge le document brut |
| 4 | Recursive Text Splitter | Decoupe en chunks pour un traitement optimal |
| 5 | Embeddings OpenAI | Transforme chaque chunk en vecteur numerique |
| 6 | Pinecone Vector Store | Stocke les vecteurs pour recherche semantique |

## Flux 2 : Chat avec les documents

Quand un utilisateur pose une question via le chat :

| Etape | Node | Description |
|-------|------|-------------|
| 1 | Chat Trigger | Recoit la question de l'utilisateur |
| 2 | AI Agent | Orchestre la reponse (LLM + Memory + Tools) |
| 3 | OpenAI Chat Model | Genere les reponses en langage naturel |
| 4 | Simple Memory | Garde l'historique de conversation (contexte multi-tour) |
| 5 | Vector Store Tool | Cherche les passages pertinents dans Pinecone |
| 6 | Pinecone VS 2 + Embeddings | Transforme la question en vecteur et recherche |
| 7 | OpenAI Chat Model 1 | Reformule les resultats trouves |

## Nodes (14 fonctionnels + 3 notes)

| Node | Type |
|------|------|
| Google Drive Trigger | googleDriveTrigger |
| Download file | googleDrive |
| Pinecone Vector Store | vectorStorePinecone |
| Embeddings OpenAI | embeddingsOpenAi |
| Recursive Character Text Splitter | textSplitterRecursiveCharacterTextSplitter |
| Default Data Loader | documentDefaultDataLoader |
| AI Agent | agent |
| When chat message received | chatTrigger |
| OpenAI Chat Model | lmChatOpenAi |
| Simple Memory | memoryBufferWindow |
| Answer questions with a vector store | toolVectorStore |
| Pinecone Vector Store 2 | vectorStorePinecone |
| OpenAI Chat Model 1 | lmChatOpenAi |
| Embeddings OpenAI 1 | embeddingsOpenAi |

## Installation

### 1. Importer le workflow

- Ouvrir n8n
- **Workflows > Import from File**
- Selectionner `workflow.json`

### 2. Configurer les credentials

| Credential | Type | Configuration |
|------------|------|---------------|
| **Google Drive OAuth2** | Google OAuth2 | Compte Google avec acces au Drive |
| **OpenAI API** | OpenAI API | Cle API OpenAI (utilisee 2x : embeddings + chat) |
| **Pinecone API** | Pinecone API | Cle API Pinecone + nom de l'index |

### 3. Configurer Pinecone

1. Creer un compte sur [pinecone.io](https://www.pinecone.io/)
2. Creer un index avec dimension **1536** (pour OpenAI embeddings)
3. Renseigner l'API key et le nom de l'index dans les credentials n8n

### 4. Configurer Google Drive

1. Specifier le dossier a surveiller dans le node "Google Drive Trigger"
2. Les formats supportes dependent du Data Loader (PDF, TXT, DOCX, etc.)

### 5. Activer

Activer le workflow. Les documents existants ne seront pas indexes automatiquement - seuls les nouveaux fichiers ou modifications declencheront l'ingestion.

## Comment ca marche (RAG)

```
1. INDEXATION
   Document ──> Decoupage en chunks ──> Embedding (vecteur) ──> Stockage Pinecone

2. RECHERCHE
   Question ──> Embedding ──> Recherche similarite dans Pinecone ──> Top K resultats

3. GENERATION
   Question + Contexte (chunks pertinents) ──> OpenAI ──> Reponse contextuelle
```

**RAG** permet a l'IA de repondre avec des informations precises issues de vos documents, plutot que de se baser uniquement sur ses connaissances generales.

## Stack technique

| Composant | Role |
|-----------|------|
| **OpenAI** | Embeddings (text-embedding-ada-002) + Chat (GPT) |
| **Pinecone** | Base de donnees vectorielle (recherche semantique) |
| **Google Drive** | Source de documents |
| **Buffer Memory** | Historique de conversation (multi-tour) |
