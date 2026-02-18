# Fiche Contexte Claude Code - Projet WIBOT / WIDIP_ARCHI

## Vue d'ensemble du projet

**WIBOT** est un assistant IA pour l'équipe technique de WIDIP. Il s'agit d'une application full-stack avec :
- **Frontend** : React + TypeScript + Vite + TailwindCSS
- **Backend** : FastAPI (Python) avec PostgreSQL + pgvector
- **MCP Server** : Serveur Model Context Protocol exposant les outils (GLPI, Observium, AD, etc.)
- **Infrastructure** : Docker Compose avec tous les services

---

## Structure des dossiers

```
C:\Users\maxime\Documents\WIDIP\WIDIP_ARCHI\01_WIBOT\
├── wibot-frontend/          # React frontend
│   └── src/
│       ├── components/      # Composants React
│       │   ├── chat/        # AgentStreaming.tsx, ChatWindow.tsx, Message.tsx
│       │   ├── safeguard/   # RequestCard.tsx, RequestList.tsx, DeferredActionList.tsx
│       │   └── ui/          # Boutons, inputs, etc.
│       ├── pages/           # Chat.tsx, Safeguard.tsx, etc.
│       ├── hooks/           # useChat.ts, useConversations.ts
│       ├── services/        # api.ts, safeguard.ts
│       └── store/           # Zustand stores
├── wibot-backend/           # FastAPI backend
│   └── app/
│       ├── routes/          # chat.py, safeguard.py, auth.py
│       ├── services/        # agent_streaming_service.py, safeguard_service.py, mcp_client.py
│       └── database/        # connection.py
├── mcp-server/              # Serveur MCP Python
│   └── src/
│       ├── mcp/             # server.py, registry.py, safeguard_queue.py
│       ├── tools/           # glpi_tools.py, observium_tools.py, ad_tools.py
│       ├── clients/         # glpi.py, observium.py, activedirectory.py
│       └── config.py        # TOOL_SECURITY_LEVELS
└── docker-compose.yml
```

---

## Système SAFEGUARD (Sécurité)

Le système SAFEGUARD contrôle l'exécution des outils MCP selon 5 niveaux :

| Niveau | Description | Comportement |
|--------|-------------|--------------|
| **L0** | Lecture seule | Exécution automatique |
| **L1** | Actions mineures | Auto si confidence >= 80% |
| **L2** | Actions modérées | Avec notification |
| **L3** | Actions sensibles | **Validation humaine requise** |
| **L4** | Interdit à l'IA | Humain uniquement (exécution différée 24-48h) |

### Fichiers clés SAFEGUARD

| Fichier | Rôle |
|---------|------|
| `mcp-server/src/config.py` | `TOOL_SECURITY_LEVELS` - définit le niveau de chaque outil |
| `wibot-backend/app/services/safeguard_service.py` | Gestion des demandes d'approbation (PostgreSQL) |
| `mcp-server/src/mcp/server.py` | Fonction `check_safeguard()` qui bloque les outils L3/L4 |
| `wibot-backend/init.sql` | Table `safeguard_config` avec les niveaux par défaut |

### Flux d'approbation L3 (implémenté)

```
1. L'agent appelle un outil L3 (ex: glpi_create_ticket)
2. Le backend intercepte, crée une demande SAFEGUARD, met l'agent en pause
3. L'utilisateur approuve (depuis l'UI chat inline ou page Safeguard)
4. Le polling frontend détecte l'approbation
5. Le backend reprend l'agent avec bypass_safeguard=True pour éviter le re-blocage MCP
```

### Double système SAFEGUARD - Point d'attention

⚠️ **Il y a deux systèmes SAFEGUARD indépendants** :
- **Backend** : Lit depuis PostgreSQL (`safeguard_config` table)
- **MCP Server** : Lit depuis `config.py` (`TOOL_SECURITY_LEVELS`)

Ils doivent être synchronisés manuellement. Le paramètre `_bypass_safeguard` permet au backend d'exécuter un outil L3 après approbation sans être re-bloqué par le MCP server.

---

## Mode Agent (Streaming SSE)

Le mode Agent permet à l'IA d'utiliser des outils MCP de manière autonome.

### Fichiers clés

| Fichier | Contenu |
|---------|---------|
| `wibot-backend/app/services/agent_streaming_service.py` | `AGENT_SYSTEM_PROMPT`, `process_agent_streaming()`, `resume_agent_after_approval()` |
| `wibot-frontend/src/components/chat/AgentStreaming.tsx` | Affichage des étapes + boutons approbation inline |
| `wibot-frontend/src/pages/Chat.tsx` | Polling pour détecter l'approbation (useEffect avec setInterval) |

### Format des événements SSE

```typescript
interface StreamingStep {
  type: 'start' | 'thinking' | 'tool_call' | 'tool_result' | 'tool_blocked' |
        'waiting_approval' | 'resumed' | 'rejected' | 'response' | 'error' | 'done';
  message?: string;
  tool?: string;
  params?: Record<string, unknown>;
  result?: unknown;
  success?: boolean;
  error?: string;
  session_id?: string;
  request_id?: string;      // ID de la demande SAFEGUARD
  security_level?: string;  // L0-L4
}
```

---

## Dernières modifications (Janvier 2026)

### Bug corrigé : Double blocage SAFEGUARD

**Problème** : Après approbation backend, le MCP server rebloquait l'outil car il a son propre check SAFEGUARD.

**Solution** : Ajout du paramètre `_bypass_safeguard` :

```python
# mcp-server/src/mcp/server.py - endpoint /tools/{tool_name}
bypass_safeguard = arguments.pop("_bypass_safeguard", False)
if bypass_safeguard:
    # Skip SAFEGUARD check
    safeguard_result = SafeguardResponse(allowed=True, ...)

# wibot-backend/app/services/mcp_client.py
async def call_tool(self, tool_name, params, user_level="L0",
                    bypass_safeguard=False, approval_id=None):
    if bypass_safeguard:
        request_params["_bypass_safeguard"] = True
        request_params["_approval_id"] = approval_id

# wibot-backend/app/services/agent_streaming_service.py - resume_agent_after_approval()
result = await client.call_tool(
    pending_tool, pending_params,
    user_level="L3",
    bypass_safeguard=True,
    approval_id=str(safeguard_request_id)
)
```

### Nouvelle feature : Approbation inline dans le chat

**Fichier** : `wibot-frontend/src/components/chat/AgentStreaming.tsx`

- Boutons "Approuver" / "Rejeter" directement dans le banner SAFEGUARD orange
- États : `approvalLoading`, `approvalError`, `approvalDone`
- Callback `onApprovalAction` pour notifier le parent et déclencher le polling

```tsx
// Gestionnaire d'approbation inline
const handleInlineApproval = async (action: 'approve' | 'reject') => {
  await approveSafeguardRequest(safeguardRequestId, 'Approuve depuis le chat');
  setApprovalDone('approved');
  onApprovalAction?.('approved');
};
```

---

## Commandes Docker utiles

```bash
# Depuis C:\Users\maxime\Documents\WIDIP\WIDIP_ARCHI\01_WIBOT
# (ou avec chemin Unix: /c/Users/maxime/Documents/WIDIP/WIDIP_ARCHI/01_WIBOT)

# Rebuild un service (--no-cache si modification de code)
docker-compose build backend --no-cache
docker-compose build frontend --no-cache
docker-compose build mcp-server --no-cache

# Restart un service
docker-compose up -d backend
docker-compose up -d frontend

# Voir les logs
docker-compose logs -f backend
docker-compose logs -f mcp-server

# Accès à la DB PostgreSQL
docker exec -it wibot-postgres psql -U wibot -d wibot

# Status de tous les conteneurs
docker-compose ps
```

---

## Base de données PostgreSQL

### Tables principales

| Table | Description |
|-------|-------------|
| `users` | Utilisateurs avec `accreditation_level` (N0-N4) |
| `conversations`, `messages` | Historique chat |
| `safeguard_config` | Configuration niveau de sécurité par outil |
| `safeguard_requests` | Demandes d'approbation en attente |
| `safeguard_audit` | Historique des approbations/rejets |
| `safeguard_deferred_actions` | Actions L4 en attente d'exécution différée |
| `agent_sessions` | Sessions agent avec état (active, waiting_approval, completed) |

### Requêtes utiles

```sql
-- Voir la config SAFEGUARD des outils
SELECT tool_name, security_level, category FROM safeguard_config ORDER BY category, tool_name;

-- Modifier le niveau d'un outil
UPDATE safeguard_config SET security_level = 'L3' WHERE tool_name = 'glpi_create_ticket';

-- Voir les demandes en attente
SELECT * FROM safeguard_requests WHERE status = 'pending';

-- Voir les sessions agent
SELECT session_id, status, pending_tool, approval_result FROM agent_sessions ORDER BY created_at DESC LIMIT 10;
```

---

## Intégrations externes

| Service | Usage | Config |
|---------|-------|--------|
| **GLPI** | Gestion tickets IT (API REST) | `GLPI_URL`, `GLPI_APP_TOKEN`, `GLPI_USER_TOKEN` |
| **Observium** | Monitoring réseau | `OBSERVIUM_URL`, `OBSERVIUM_USER`, `OBSERVIUM_PASS` |
| **Active Directory** | Infos utilisateurs | `AD_SERVER`, `AD_USER`, `AD_PASSWORD` |
| **Redis** | Cache et queue | `REDIS_URL` |
| **PostgreSQL** | Base de données principale | `POSTGRES_DSN` |

---

## Points d'attention pour le développement

1. **Docker build obligatoire** : Le code est copié à la construction de l'image. Après toute modification de code, il faut rebuild (`docker-compose build <service> --no-cache`).

2. **Synchronisation SAFEGUARD** : Les niveaux doivent être identiques dans `safeguard_config` (DB) et `config.py` (MCP server).

3. **Polling frontend** : Le fichier `Chat.tsx` a un `useEffect` avec `setInterval(3000)` qui vérifie le statut de la session quand `waitingForApproval` est true.

4. **Outils L3 actuels** : `glpi_create_ticket` est L3 (requiert approbation humaine).

5. **Prompt Agent** : Le prompt système de l'agent est dans `agent_streaming_service.py` (`AGENT_SYSTEM_PROMPT`). Il contient les règles SAFEGUARD et les instructions pour créer des tickets GLPI.

---

## Workflow Sentinel (Observium Proactif)

Le Workflow Sentinel est un service autonome qui surveille Observium et prend des actions proactives.

### Configuration

| Variable | Défaut | Description |
|----------|--------|-------------|
| `WORKFLOW_INTERVAL_SECONDS` | 300 | Intervalle de polling (5 minutes) |
| `WORKFLOW_ENABLED` | true | Active/désactive le workflow |
| `alert_mail_threshold_minutes` | 20 | Délai avant envoi d'alerte mail pour device DOWN |

### Fichiers clés

```
workflows/workflow-observium/
├── src/
│   ├── config.py                    # Configuration (polling, SMTP, seuils)
│   ├── models/workflow.py           # WorkflowActionType (CREATE_TICKET, SEND_ALERT_MAIL, etc.)
│   └── services/
│       ├── workflow_engine.py       # Moteur principal du workflow
│       └── device_down_tracker.py   # Cache Redis pour durée DOWN des devices
```

### Device DOWN Tracker (Redis)

Stocke la durée de DOWN de chaque device dans Redis :
- Clé : `workflow:device_down:{device_id}`
- Données : `first_seen`, `last_seen`, `duration_minutes`, `alert_sent`
- TTL : 7 jours

### Alertes Mail (SMTP)

Quand un device est DOWN > 20 minutes :
1. Le workflow détecte via `device_down_tracker.get_devices_needing_alert()`
2. Crée une action `SEND_ALERT_MAIL` de niveau **L3**
3. Attend validation Safeguard
4. Envoie le mail via `smtp_send_alert` MCP tool

### Configuration SMTP (.env)

```env
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=wibot@widip.fr
SMTP_PASSWORD=xxx
SMTP_FROM_EMAIL=wibot@widip.fr
SMTP_FROM_NAME=WIBOT Sentinel
SMTP_USE_TLS=true
ALERT_MAIL_RECIPIENTS=support@widip.fr,tech@widip.fr
```

### MCP Tools SMTP

| Tool | Niveau | Description |
|------|--------|-------------|
| `smtp_test_connection` | L0 | Test connexion SMTP |
| `smtp_send_email` | L3 | Envoi email générique |
| `smtp_send_alert` | L3 | Envoi alerte device DOWN |

---

## URLs de développement

- **Frontend** : http://localhost:8080
- **Backend API** : http://localhost:8000
- **MCP Server** : http://localhost:3001
- **PostgreSQL** : localhost:5433
- **Redis** : localhost:6379
- **Monitoring Workflow** : http://localhost:8080/monitoring
