# WIBOT v2.0.0

> WIDIP Intelligent Bot - Assistant IA avec acces aux outils MCP (GLPI, Observium, AD, etc.)

## Architecture

```
                      localhost:8080
                           |
                    +------v------+
                    |    Nginx    |
                    | (Reverse    |
                    |   Proxy)    |
                    +------+------+
                           |
          +----------------+----------------+
          |                                 |
   +------v------+                  +-------v-------+
   |  Frontend   |                  |    Backend    |
   |   (React)   |                  |   (FastAPI)   |
   |   :80 int   |                  |     :8000     |
   +-------------+                  +-------+-------+
                                            |
                    +-----------------------+
                    |                       |
            +-------v-------+       +-------v-------+
            |  MCP Server   |       |  PostgreSQL   |
            |    :3001      |       |    :5433      |
            | (40+ tools)   |       |  + pgvector   |
            +---------------+       +---------------+
```

## Services

| Service      | Port  | Description                              |
|--------------|-------|------------------------------------------|
| Nginx        | 8080  | Reverse proxy (point d'entree)           |
| Frontend     | -     | React/TypeScript (interne Docker)        |
| Backend      | 8000  | FastAPI + Chat + Agent                   |
| MCP Server   | 3001  | 40+ outils (GLPI, Observium, AD, etc.)   |
| PostgreSQL   | 5433  | Base de donnees + pgvector               |

## Demarrage rapide

### Prerequis
- Docker Desktop
- Git

### Lancement

```bash
# Cloner le projet
git clone <repo-url>
cd 01_WIBOT

# Demarrer (Windows)
start.bat

# Ou manuellement
docker-compose up -d
```

### URLs
- **Frontend**: http://localhost:8080
- **API Docs**: http://localhost:8080/docs
- **Backend direct**: http://localhost:8000
- **MCP Server**: http://localhost:3001

## Modes de Chat

| Mode      | Modele           | Description                                      |
|-----------|------------------|--------------------------------------------------|
| Code      | Devstral         | Optimise pour le code                            |
| Flash     | Devstral Small   | Rapide et economique                             |
| Pro       | Mistral Large    | Qualite maximale                                 |
| **Agent** | Mistral Large    | Acces aux outils MCP (GLPI, Observium, AD, etc.) |

## Mode Agent

Le mode Agent permet a l'IA d'acceder aux 40+ outils MCP pour:
- **GLPI**: Recherche clients, creation/modification tickets
- **Observium**: Statut equipements reseau
- **Active Directory**: Verification utilisateurs, groupes
- **Et plus**: Notifications, secrets, memoire...

### Niveaux SAFEGUARD

| Niveau | Permissions              | Exemples                        |
|--------|--------------------------|--------------------------------|
| L0     | Lecture seule            | Recherche, consultation        |
| L1     | Creation                 | Creer ticket, envoyer notif    |
| L2     | Modification             | Modifier ticket, reset password|
| L3     | Suppression              | Fermer ticket                  |
| L4     | Admin                    | Operations critiques           |

## Structure du projet

```
01_WIBOT/
├── docker-compose.yml      # Orchestration Docker
├── nginx.conf              # Config reverse proxy
├── start.bat               # Script demarrage Windows
├── wibot-backend/          # Backend FastAPI
│   ├── app/
│   │   ├── routes/         # Endpoints API
│   │   ├── services/       # Logique metier (chat, agent, MCP)
│   │   └── models/         # Schemas Pydantic
│   ├── init.sql            # Schema BDD
│   └── Dockerfile
├── wibot-frontend/         # Frontend React
│   ├── src/
│   │   ├── components/     # Composants UI
│   │   ├── pages/          # Pages
│   │   ├── hooks/          # React hooks
│   │   └── services/       # Appels API
│   └── Dockerfile
├── 02_MCP_SERVER/          # Serveur MCP
│   └── Dockerfile
├── Documentation/          # Documentation complete
└── Archive/                # Anciens fichiers (n8n, etc.)
```

## Configuration

### Variables d'environnement (docker-compose.yml)

```yaml
# Backend
DATABASE_URL: postgresql://...
JWT_SECRET: ...
MISTRAL_API_KEY: ...
MCP_SERVER_URL: http://mcp-server:3001

# MCP Server
GLPI_URL: ...
GLPI_APP_TOKEN: ...
OBSERVIUM_URL: ...
LDAP_SERVER: ...
```

## Commandes utiles

```bash
# Statut des services
docker ps

# Logs d'un service
docker logs wibot-backend -f
docker logs wibot-mcp-server -f

# Rebuild complet
docker-compose down
docker-compose build --no-cache
docker-compose up -d

# Arreter tout
docker-compose down
```

## Documentation

Voir le dossier `Documentation/` pour:
- `ROADMAP.md` - Feuille de route v3.0
- `FICHE_TECHNIQUE_DEVELOPPEUR.md` - Details techniques
- `GUIDE_FRONTEND.md` - Guide frontend
- `WIBOT_DOCUMENTATION.md` - Documentation complete
- `Log+doc api/` - Documentation APIs externes (GLPI, Observium)
