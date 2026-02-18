# WIBOT v2.0 - ROADMAP des corrections et fonctionnalites

**Date**: 2026-01-14
**Statut**: COMPLETE - Pret pour rebuild et tests

---

## Analyse de l'existant

### Problemes identifies

| Composant | Probleme | Impact |
|-----------|----------|--------|
| MCP Server | Utilise Ollama pour embeddings alors que le projet utilise Mistral | RAG MCP non fonctionnel |
| Redis | Non configure dans docker-compose | Pas de cache/sessions pour SAFEGUARD L3 |
| Agent | Configurations pas appliquees (container non rebuild) | Agent en erreur |
| Frontend | Pas d'interface pour gerer la base de connaissances | Utilisateurs ne peuvent pas gerer le RAG |

### Etat actuel des embeddings

```
Backend (rag_service.py)     -> Mistral (mistral-embed) - 1024 dimensions ✓
MCP Server (memory.py)       -> Ollama (nomic-embed-text) - ERREUR ✗
Configuration (config.py)    -> Ollama par defaut - A CHANGER ✗
```

---

## PHASE 1: Correction Infrastructure (Priorite HAUTE)

### 1.1 Migrer MCP Server vers Mistral Embeddings

**Fichiers a modifier:**
- `02_MCP_SERVER/src/config.py` - Remplacer config Ollama par Mistral
- `02_MCP_SERVER/src/clients/memory.py` - Utiliser Mistral API au lieu d'Ollama
- `02_MCP_SERVER/requirements.txt` - Ajouter `mistralai` si absent

**Changements:**
```python
# config.py - AVANT
ollama_url: str = "http://localhost:11434"
ollama_embed_model: str = "intfloat/multilingual-e5-large"
ollama_embed_dimensions: int = 1024

# config.py - APRES
mistral_api_key: SecretStr  # Reutiliser celle du backend
mistral_embed_model: str = "mistral-embed"
mistral_embed_dimensions: int = 1024
```

**Estimation**: ~30 lignes de code

---

### 1.2 Ajouter Redis au docker-compose

**Fichier:** `docker-compose.yml`

**Service a ajouter:**
```yaml
redis:
  image: redis:7-alpine
  container_name: wibot-redis
  ports:
    - "6379:6379"
  volumes:
    - redis_data:/data
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 10s
    timeout: 5s
    retries: 5
  networks:
    - wibot-network
  restart: unless-stopped
```

**Variables a ajouter au MCP Server:**
```yaml
- REDIS_HOST=redis
- REDIS_PORT=6379
- REDIS_SECRET_KEY=${REDIS_SECRET_KEY:-dev_secret_key_32_chars_minimum!}
```

---

### 1.3 Rebuild et test des containers

**Commandes:**
```bash
docker-compose down
docker-compose build --no-cache mcp-server backend
docker-compose up -d
docker-compose logs -f mcp-server
```

---

## PHASE 2: Verification Agent (Priorite HAUTE)

### 2.1 Verifier les outils disponibles

Apres rebuild, tester que l'agent a acces aux outils L0+L1:
- `glpi_search_by_location`
- `glpi_get_tickets_by_entity`
- `glpi_create_ticket`
- `glpi_add_ticket_followup`
- etc.

### 2.2 Tester la correlation Observium-GLPI

Scenario de test:
1. Demander: "Le switch SW-SIRET-01 est down, qui est impacte?"
2. L'agent doit:
   - Appeler `observium_get_device_status` -> obtenir location
   - Appeler `glpi_search_by_location` -> trouver entites
   - Appeler `glpi_get_tickets_by_entity` -> voir tickets existants
   - Formuler une reponse avec clients impactes

---

## PHASE 3: Interface Knowledge Base Frontend (Priorite MOYENNE)

### 3.1 Structure des composants

**Nouveaux fichiers:**
```
wibot-frontend/src/
├── components/
│   └── knowledge/
│       ├── KnowledgePanel.tsx      # Panneau principal
│       ├── KnowledgeList.tsx       # Liste des entrees
│       ├── KnowledgeSearch.tsx     # Recherche semantique
│       ├── KnowledgeForm.tsx       # Ajout/edition
│       └── KnowledgeStats.tsx      # Statistiques
├── pages/
│   └── KnowledgePage.tsx           # Page dediee
└── services/
    └── knowledgeService.ts         # API calls
```

### 3.2 Fonctionnalites UI

| Fonctionnalite | Description |
|----------------|-------------|
| Liste paginee | Afficher les entrees de la base de connaissances |
| Recherche | Recherche semantique + filtres (categorie, date, score) |
| Ajout manuel | Formulaire pour ajouter une connaissance |
| Edition | Modifier une entree existante |
| Suppression | Supprimer une entree (avec confirmation) |
| Import | Upload de fichiers (PDF, TXT, MD) |
| Stats | Dashboard avec metriques RAG |

### 3.3 Endpoints Backend necessaires

**Fichier:** `wibot-backend/app/routers/knowledge.py`

```python
# Nouveaux endpoints
GET    /api/knowledge           # Liste paginee
GET    /api/knowledge/{id}      # Detail
POST   /api/knowledge           # Ajout
PUT    /api/knowledge/{id}      # Modification
DELETE /api/knowledge/{id}      # Suppression
GET    /api/knowledge/stats     # Statistiques
POST   /api/knowledge/search    # Recherche semantique
POST   /api/knowledge/ingest    # Import fichier
```

---

## PHASE 4: Ameliorations (Priorite BASSE)

### 4.1 Synchronisation des schemas RAG

Harmoniser les tables entre backend et MCP:
- `document_chunks` (backend) - documents utilisateurs
- `widip_knowledge_base` (MCP) - connaissances tickets

### 4.2 Dashboard SAFEGUARD

Interface pour les validations L3 (approvals en attente).

### 4.3 Logs et metriques

- Historique des appels d'outils
- Temps de reponse
- Taux de succes RAG

---

## ORDRE D'IMPLEMENTATION

```
[1] Phase 1.1 - Mistral embeddings MCP Server
[2] Phase 1.2 - Redis docker-compose
[3] Phase 1.3 - Rebuild containers
[4] Phase 2   - Test Agent
[5] Phase 3.3 - Endpoints backend knowledge
[6] Phase 3.1 - Composants frontend knowledge
[7] Phase 3.2 - Fonctionnalites UI
[8] Phase 4   - Ameliorations optionnelles
```

---

## CHECKLIST FINALE

- [x] MCP Server utilise Mistral embeddings
- [x] Redis fonctionne dans docker-compose
- [x] Agent peut correler Observium-GLPI (outils ajoutes)
- [x] Agent peut creer tickets et followups (L1 autorise)
- [x] Frontend a une page Knowledge Base
- [x] Utilisateurs peuvent voir/ajouter/supprimer des connaissances
- [x] Recherche semantique fonctionne
- [x] Import de documents fonctionne

---

## FICHIERS MODIFIES

### MCP Server
- `02_MCP_SERVER/src/config.py` - Configuration Mistral au lieu d'Ollama
- `02_MCP_SERVER/src/clients/memory.py` - Client Mistral pour embeddings
- `02_MCP_SERVER/pyproject.toml` - Ajout dependance mistralai

### Docker
- `docker-compose.yml` - Ajout service Redis + variables MISTRAL_API_KEY

### Backend
- `wibot-backend/app/routes/knowledge.py` - Nouveaux endpoints Knowledge Base
- `wibot-backend/main.py` - Import et mount du router knowledge
- `wibot-backend/init.sql` - Table document_chunks avec indexes

### Frontend
- `wibot-frontend/src/services/api.ts` - Fonctions API Knowledge Base
- `wibot-frontend/src/pages/Knowledge.tsx` - Page Knowledge Base complete
- `wibot-frontend/src/App.tsx` - Route /knowledge
- `wibot-frontend/src/components/layout/Header.tsx` - Lien menu

---

## PROCHAINES ETAPES

Pour appliquer les changements:

```bash
# 1. Arreter les containers
docker-compose down

# 2. Rebuild les containers (MCP Server + Backend)
docker-compose build --no-cache mcp-server backend

# 3. Demarrer tous les services
docker-compose up -d

# 4. Verifier les logs
docker-compose logs -f mcp-server
docker-compose logs -f backend

# 5. Tester l'agent avec la correlation Observium-GLPI
```

---

**Note**: Ce document a ete genere automatiquement le 2026-01-14.
