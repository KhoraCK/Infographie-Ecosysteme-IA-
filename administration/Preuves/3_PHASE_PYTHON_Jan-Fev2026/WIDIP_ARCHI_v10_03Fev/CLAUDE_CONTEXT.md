# Fiche Contexte Claude Code - Projet WIBOT / WIDIP_ARCHI

**Version** : 2.2.0
**Dernière mise à jour** : 3 février 2026

---

## Vue d'ensemble du projet

**WIBOT** (WIDIP Intelligence Bot) est un assistant IA full-stack pour l'équipe technique de WIDIP. Il intègre une interface de chat intelligente avec des outils MCP (Model Context Protocol) connectés aux systèmes d'entreprise (GLPI, Observium, Active Directory). Le système inclut un framework de sécurité **SAFEGUARD** sophistiqué qui contrôle l'exécution des outils selon des niveaux de sensibilité (L0-L4+).

### Stack technique principal

| Composant | Technologies |
|-----------|--------------|
| **Frontend** | React 19 + TypeScript + Vite 7 + TailwindCSS 3.4 + Zustand |
| **Backend** | FastAPI (Python) + PostgreSQL 14 + pgvector + Mistral AI |
| **MCP Server** | FastAPI (Python) + structlog + 50+ outils MCP |
| **Workflow** | Service autonome FastAPI pour monitoring proactif |
| **Infrastructure** | Docker Compose (6 services) + Nginx + Redis 7 |

---

## Structure complète des dossiers

```
C:\Users\maxime\Documents\WIDIP\WIDIP_ARCHI\
├── 01_WIBOT/                              # Application principale (3 composants)
│   ├── wibot-frontend/                    # React + Vite + TailwindCSS
│   │   └── src/
│   │       ├── App.tsx                    # Point d'entrée, routing
│   │       ├── main.tsx                   # React 19 strict mode
│   │       ├── components/
│   │       │   ├── chat/                  # AgentStreaming.tsx, ChatWindow.tsx, Message.tsx
│   │       │   ├── safeguard/             # RequestCard.tsx, DeferredActionList.tsx
│   │       │   ├── layout/                # Header.tsx, Sidebar.tsx, InputBar.tsx
│   │       │   └── ui/                    # Composants réutilisables (Button, Input, Spinner)
│   │       ├── pages/                     # Chat.tsx, Safeguard.tsx, Login.tsx, AdminUsers.tsx
│   │       ├── hooks/                     # useChat.ts, useConversations.ts
│   │       ├── services/                  # api.ts, safeguard.ts
│   │       ├── store/                     # Zustand: authStore, chatStore, safeguardStore
│   │       └── types/                     # Interfaces TypeScript
│   │
│   ├── wibot-backend/                     # FastAPI Python backend
│   │   ├── main.py                        # Point d'entrée FastAPI + routes
│   │   ├── app/
│   │   │   ├── config.py                  # Configuration Pydantic (DB, JWT, quotas)
│   │   │   ├── middleware/auth.py         # Vérification JWT
│   │   │   ├── database/connection.py     # Pool PostgreSQL
│   │   │   ├── models/                    # Classes Pydantic (user, chat, safeguard)
│   │   │   ├── routes/                    # Handlers API (auth, chat, safeguard, rag)
│   │   │   └── services/                  # Logique métier
│   │   │       ├── agent_streaming_service.py (36KB) # AGENT_SYSTEM_PROMPT, streaming SSE
│   │   │       ├── mcp_client.py (10KB)             # Appels vers MCP Server
│   │   │       ├── safeguard_service.py (41KB)      # Gestion approbations
│   │   │       └── rag_service_v2.py (21KB)         # Embeddings Mistral + pgvector
│   │   └── requirements.txt
│   │
│   ├── mcp-server/                        # Model Context Protocol server
│   │   └── src/
│   │       ├── main.py                    # Uvicorn port 3001
│   │       ├── config.py                  # TOOL_SECURITY_LEVELS (L0-L4, L4_SMS)
│   │       ├── mcp/
│   │       │   ├── server.py              # FastAPI + check SAFEGUARD
│   │       │   ├── registry.py            # Enregistrement auto des outils
│   │       │   └── safeguard_queue.py
│   │       ├── clients/                   # Clients services externes
│   │       │   ├── glpi.py (52KB)         # API GLPI (tickets, entités, KB)
│   │       │   ├── observium.py           # API Observium
│   │       │   ├── activedirectory.py     # LDAP/AD
│   │       │   ├── referent_cache.py      # Cache Redis pour SMS
│   │       │   └── smtp.py                # Envoi emails
│   │       └── tools/                     # 50+ outils MCP
│   │           ├── glpi_tools.py (52KB)       # L0-L3
│   │           ├── observium_tools.py (5KB)   # L0
│   │           ├── ad_tools.py (9KB)          # L0-L4
│   │           ├── sms_alert_tools.py (29KB)  # L4_SMS/L4
│   │           ├── smtp_tools.py              # L3
│   │           ├── annuaire_tools.py          # L0
│   │           └── memory_tools_v2.py         # L0-L1
│   │
│   ├── docker-compose.yml                 # Orchestration 6 services
│   ├── nginx.conf                         # Reverse proxy
│   ├── init.sql                           # Schéma PostgreSQL (1139 lignes)
│   ├── rag-documents/                     # Documents pour RAG
│   └── uploads/                           # Fichiers uploadés
│
├── workflows/                             # Système monitoring proactif
│   └── workflow-observium/
│       ├── main.py                        # Modes: daemon, api, once + API endpoints
│       └── src/
│           ├── config.py                  # Polling, seuils alertes (20 min SMS)
│           ├── models/
│           │   ├── workflow.py            # WorkflowActionType
│           │   └── timeline.py            # AlertTimeline, TimelineSnapshot, AlertProgress
│           └── services/
│               ├── workflow_engine.py     # Moteur principal (logique seuil 20 min)
│               ├── timeline_service.py    # Streaming SSE temps réel
│               └── device_down_tracker.py # Cache Redis durée DOWN
│
├── docs/                                  # Documentation
│   ├── api-guides/                        # Guides API GLPI & Observium
│   ├── architecture/                      # Architecture technique
│   ├── FLUX_ALERTE_SMS.md (54KB)          # Flux SMS détaillé
│   └── roadmaps/                          # Roadmaps fonctionnelles
│
└── CLAUDE_CONTEXT.md                      # Ce fichier
```

---

## Système SAFEGUARD (Sécurité)

Le système SAFEGUARD contrôle l'exécution des outils MCP selon 6 niveaux de sensibilité :

| Niveau | Nom | Comportement | Exemples d'outils |
|--------|-----|--------------|-------------------|
| **L0** | READ_ONLY | Exécution automatique | `observium_list_devices`, `glpi_search_procedure` |
| **L1** | MINOR | Auto si confidence ≥ 80%, sinon notifie | Commentaires tickets |
| **L2** | MODERATE | Exécute avec notification | `glpi_create_ticket`, `ad_unlock_account` |
| **L3** | SENSITIVE | **Validation humaine requise** | `glpi_close_ticket`, `ad_reset_password` |
| **L4** | FORBIDDEN | **Bloqué - Humain uniquement** | `ad_create_user`, `sms_alert_send` |
| **L4_SMS** | SMS ALERT | Interface spéciale sélection contact | `sms_alert_device_down` |

### Fichiers clés SAFEGUARD

| Fichier | Rôle |
|---------|------|
| `mcp-server/src/config.py` | `TOOL_SECURITY_LEVELS` - définit le niveau de chaque outil |
| `wibot-backend/app/services/safeguard_service.py` | Gestion des demandes d'approbation (PostgreSQL) |
| `mcp-server/src/mcp/server.py` | Fonction `check_safeguard()` qui bloque les outils L3/L4 |
| `wibot-backend/init.sql` | Table `safeguard_config` avec les niveaux par défaut |

### Double système SAFEGUARD - Point d'attention critique

⚠️ **Il y a DEUX systèmes SAFEGUARD indépendants** :

1. **Backend SAFEGUARD** (`wibot-backend/app/services/safeguard_service.py`)
   - Lit depuis PostgreSQL (table `safeguard_config`)
   - Crée les demandes d'approbation
   - Implémente le workflow d'approbation

2. **MCP Server SAFEGUARD** (`mcp-server/src/mcp/server.py`)
   - Lit depuis Python `config.py` (`TOOL_SECURITY_LEVELS`)
   - Bloque les appels outils non autorisés
   - Empêche la double exécution via `_bypass_safeguard`

**Synchronisation obligatoire** : Les deux systèmes doivent avoir des définitions de niveaux identiques. Après approbation humaine, le backend passe `_bypass_safeguard=True` pour éviter le re-blocage.

### Flux d'approbation L3

```
1. L'agent appelle un outil L3 (ex: glpi_create_ticket)
2. MCP Server intercepte → check niveau (L3)
3. Retourne: {allowed: false, type: "waiting_approval", request_id: "xyz"}
4. Backend crée ligne safeguard_requests avec status="pending"
5. SSE envoie: {type: "tool_blocked", request_id: "xyz", tool: "glpi_create_ticket"}
6. Frontend affiche bandeau orange "Approuver" / "Rejeter"
7. Utilisateur clique Approuver → POST /api/safeguard/requests/{id}/approve
8. Backend met à jour status="approved"
9. Frontend détecte via polling (setInterval 3s)
10. Frontend déclenche resume_agent_after_approval()
11. Backend rappelle l'outil avec bypass_safeguard=True
12. MCP Server skip la vérification SAFEGUARD
13. Outil s'exécute, résultat renvoyé à l'agent
14. L'agent continue
```

---

## Mode Agent (Streaming SSE)

Le mode Agent permet à l'IA d'utiliser des outils MCP de manière autonome avec streaming temps réel.

### Fichiers clés

| Fichier | Contenu |
|---------|---------|
| `wibot-backend/app/services/agent_streaming_service.py` | `AGENT_SYSTEM_PROMPT`, `process_agent_streaming()`, `resume_agent_after_approval()` |
| `wibot-frontend/src/components/chat/AgentStreaming.tsx` | Affichage étapes + boutons approbation inline |
| `wibot-frontend/src/pages/Chat.tsx` | Polling détection approbation (useEffect setInterval 3s) |

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

### Modes de réponse du backend

| Mode | Description | Déclencheur |
|------|-------------|-------------|
| **FLASH** | Réponse directe Mistral LLM | Questions simples sans besoin d'outils |
| **RAG** | Recherche base de connaissances + génération | Questions sur documentation/procédures |
| **AGENT** | Utilisation outils MCP avec streaming | Actions système (tickets, monitoring, AD) |

---

## Système Alerte SMS L4_SMS (Device DOWN)

### Vue d'ensemble

Le système permet d'alerter par SMS le référent DirEnt lors d'une panne détectée sur Observium. Il utilise un niveau SAFEGUARD spécial **L4_SMS** qui affiche une interface de sélection du contact.

### Flux complet

```
1. Observium détecte device DOWN (ex: mfi-3a-smh-sw02)
2. Agent/Workflow appelle sms_alert_device_down(hostname, status, message)
3. L'outil:
   - Parse le hostname → client=MFI, site=3A, ville=SMH
   - Recherche l'entité GLPI correspondante
   - Vérifie le cache Redis pour le référent
   - Si cache miss → calcule via glpi_find_active_dirent_referent
   - Prépare les données pour la fenêtre SAFEGUARD
4. SAFEGUARD bloque avec niveau L4_SMS
5. Dashboard affiche fenêtre avec:
   - Infos device/entité
   - Contact recommandé (pré-sélectionné)
   - Liste alternatives (autres DirEnt)
   - Aperçu SMS
6. Humain valide → SMS envoyé via API AllMySMS
7. Alarme créée dans API Backoffice
8. Cache mis à jour si contact différent du recommandé
```

### Outils MCP SMS

| Outil | Niveau | Description |
|-------|--------|-------------|
| `sms_alert_device_down` | **L4_SMS** | Prépare alerte, déclenche fenêtre sélection |
| `sms_alert_send` | L4 | Envoi effectif (callback SAFEGUARD only) |
| `sms_alert_cache_stats` | L0 | Stats du cache référents |
| `parse_device_hostname` | L0 | Parse hostname Observium |
| `glpi_find_active_dirent_referent` | L0 | Trouve le DirEnt le plus actif |
| `glpi_create_alarm` | L4 | Crée alarme API Backoffice |

### Cache Référent (Redis)

Structure de clé : `referent:entity:{entity_id}`

```json
{
  "user_id": 12345,
  "name": "Jean Dupont",
  "mobile": "0612345678",
  "mobile_formatted": "06 12 34 56 78",
  "activity_score": 45,
  "is_currently_active": true,
  "confirmed_by_human": false,
  "cached_at": "2026-01-28T10:00:00",
  "last_used": "2026-01-28T10:30:00",
  "use_count": 5
}
```

- **TTL** : 7 jours (14 jours si confirmé par humain)
- **Renouvelé** à chaque utilisation
- **Apprentissage** : si l'humain choisit un contact différent, le cache est mis à jour

### Parsing Hostname Observium

Format typique : `CLIENT-SITE-VILLE-DEVICE`

Exemples :
- `mfi-3a-smh-sw02` → client=MFI, site=3A, ville=SMH (Saint-Martin-d'Hères), device=SW02, type=switch
- `widip-main-rt01` → client=WIDIP, site=MAIN, device=RT01, type=routeur
- `abc-fw01` → client=ABC, device=FW01, type=pare-feu

### API Backoffice Alarmes

Endpoint : `POST https://backofficetest.widip.fr/api/alarms`

Profils SMS disponibles :
- `1` = POSTONLY
- `8` = DIRENT (défaut)
- `15` = REFERENT
- `16` = ACCOUNTANT

⚠️ **Note pré-prod** : Le cron SMS est désactivé, les alarmes sont créées en BDD mais non envoyées.

---

## Workflow Sentinel (Observium Proactif)

Le Workflow Sentinel est un service autonome qui surveille Observium et prend des actions proactives.

### Modes d'exécution

```bash
python main.py              # Mode daemon (boucle continue)
python main.py --once       # Cycle unique
python main.py --api        # Mode API (port 8001) + boucle optionnelle
```

### Configuration

| Variable | Défaut | Description |
|----------|--------|-------------|
| `WORKFLOW_INTERVAL_SECONDS` | 300 | Intervalle de polling (5 minutes) |
| `WORKFLOW_ENABLED` | true | Active/désactive le workflow |
| `alert_mail_threshold_minutes` | 20 | Seuil avant création ticket + envoi SMS |

### Logique de workflow corrigée (3 février 2026)

**Chronologie d'une alerte device DOWN :**

```
0 min     : Détection → Alerte visible, timer démarre
           (enrichissement contexte, PAS de ticket)

0-20 min  : Barre de progression visible
           (attente du seuil, monitoring seulement)

20 min    : Seuil atteint →
           1. Création automatique ticket GLPI (L1, pas de validation)
           2. Déclenchement SMS avec numéro ticket (L4_SMS, validation humaine)

Après 20 min : En attente approbation SMS ou device remonté
```

**Fichiers clés :**
- `workflow_engine.py` : `_process_mail_alerts()` (crée ticket + déclenche SMS)
- `workflow_engine.py` : `_analyze_and_decide()` (NE crée PAS de ticket, attente seuil)
- `timeline_service.py` : `stream_snapshots()` (SSE toutes les 5 secondes)

### API Endpoints (port 8001)

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/health` | GET | Health check |
| `/status` | GET | État du workflow |
| `/start` | POST | Démarre la boucle monitoring |
| `/stop` | POST | Arrête le monitoring |
| `/run` | POST | Cycle manuel |
| `/results` | GET | Résultats récents |
| `/timeline` | GET | Snapshot timeline alertes (JSON) |
| `/timeline/stream` | GET | **SSE Stream temps réel** (toutes les 5s) |
| `/timeline/{hostname}` | GET | Timeline détaillée d'un device |
| `/timeline/{hostname}` | DELETE | Suppression manuelle alerte |
| `/pending-approvals` | GET | Demandes SAFEGUARD en attente |
| `/config` | PATCH | Modification config runtime |
| `/actions/trigger` | POST | Déclencher action manuelle |
| `/actions/{id}/approve` | POST | Approuver action SAFEGUARD |
| `/actions/{id}/reject` | POST | Rejeter action SAFEGUARD |
| `/resolved-alerts` | GET | Historique alertes résolues |

### Timeline Service (SSE)

Le service de timeline fournit un streaming temps réel via Server-Sent Events :

**Format SSE :**
```
event: snapshot
data: {"type": "timeline_snapshot", "data": {...}}
```

**Structure AlertTimeline :**
```json
{
  "hostname": "device-xyz",
  "duration_minutes": 15,
  "minutes_until_sms": 5,
  "sms_progress_percent": 75.0,
  "sms_threshold_minutes": 20,
  "current_status": "waiting_sms",
  "is_sms_sent": false,
  "is_device_up": false,
  "ticket_id": null,
  "entity_name": "Entité Client",
  "events": [...]
}
```

**Statuts possibles (AlertProgress) :**
- `detecting` : En cours de détection
- `enriching` : Enrichissement du contexte
- `creating_ticket` : Création du ticket
- `waiting_sms` : Attente du seuil SMS (0-20 min)
- `pending_approval` : En attente approbation SAFEGUARD
- `completed` : Traitement terminé (device UP)
- `error` : Erreur

### Device DOWN Tracker (Redis)

Stocke la durée de DOWN de chaque device dans Redis :
- Clé : `workflow:device_down:{hostname}`
- Données : `first_seen`, `last_seen`, `duration_minutes`, `alert_sent`, `ticket_id`, `entity_id`, `entity_name`
- TTL : 7 jours

### Alertes après seuil 20 min

Quand un device est DOWN > 20 minutes :
1. Le workflow détecte via `device_down_tracker.get_devices_needing_alert()`
2. **Étape 1** : Création ticket GLPI automatique (L1, pas de validation SAFEGUARD)
3. **Étape 2** : Déclenchement alerte SMS avec numéro ticket dans le message
4. Si L4_SMS configuré : Attend validation humaine avant envoi SMS
5. Sinon : SMS envoyé directement

---

## Page Monitoring (Frontend)

La page Monitoring (`/monitoring`) affiche en temps réel les alertes devices DOWN avec barres de progression vers le seuil SMS.

### Composants clés

| Fichier | Rôle |
|---------|------|
| `pages/Monitoring.tsx` | Page principale, gestion SSE, états |
| `components/monitoring/ActiveAlertCard.tsx` | Carte alerte avec barre de progression, bouton supprimer |
| `components/monitoring/PendingActionsPanel.tsx` | Panel SAFEGUARD L4_SMS avec sélection contact |
| `components/monitoring/ResolvedAlertsPanel.tsx` | Historique alertes résolues |
| `components/monitoring/WorkflowControlPanel.tsx` | Contrôle ON/OFF du workflow |
| `services/workflow.ts` | API calls + `createTimelineStream()` SSE |
| `hooks/useWorkflow.ts` | Hook pour status, pendingApprovals, refresh |

### Fonctionnalités

1. **Barres de progression temps réel** : Mise à jour toutes les 5s via SSE
2. **Suppression d'alertes** : Bouton pour supprimer alertes de test
3. **Reconnexion automatique** : Si SSE déconnecté, reconnexion après 5s
4. **Indicateurs visuels** :
   - Statut connexion SSE (vert = connecté, rouge = déconnecté)
   - Couleur barre selon progression (bleu < 60%, jaune 60-80%, orange 80-100%, rouge 100%+)
5. **Fenêtre SAFEGUARD L4_SMS** : Après le seuil 20 min, popup avec sélection contact

### Panel L4_SMS (PendingActionsPanel)

Quand un device atteint le seuil de 20 minutes, une demande SAFEGUARD L4_SMS apparaît dans le panneau "Actions en attente".

**Données affichées :**
- Hostname et durée DOWN
- Contact recommandé (pré-sélectionné)
- Contacts alternatifs (liste déroulante)
- Aperçu du SMS
- Boutons Approuver / Rejeter

**Structure Redis (`workflow:pending_approvals`) :**
```json
{
  "approval_id": "L4SMS-WF-xxx",
  "security_level": "L4_SMS",
  "action_type": "sms_alert_device_down",
  "context": {
    "hostname": "device-xyz",
    "ticket_id": 12345,
    "recommended_contact": {
      "user_id": 123,
      "name": "Jean Dupont",
      "mobile": "0612345678",
      "mobile_formatted": "06 12 34 56 78"
    },
    "alternative_contacts": [...],
    "sms_preview": "ALERTE WIDIP: ..."
  }
}
```

**Flux complet :**
1. Workflow détecte device DOWN > 20 min
2. Crée ticket GLPI automatiquement (L1)
3. Appelle `mcp_client.send_sms_alert()` → retourne données L4_SMS
4. Stocke demande dans Redis avec contacts
5. Frontend récupère via `/pending-approvals`
6. PendingActionsPanel affiche la demande
7. Utilisateur sélectionne contact et approuve
8. SMS envoyé via AllMySMS

### Connexion SSE (workflow.ts)

```typescript
export function createTimelineStream(
  onSnapshot: (snapshot: TimelineSnapshot) => void,
  onError?: (error: Event) => void,
  onOpen?: () => void
): EventSource {
  const eventSource = new EventSource(`${WORKFLOW_API_URL}/timeline/stream`);

  eventSource.addEventListener('snapshot', (event) => {
    const data = JSON.parse(event.data);
    onSnapshot(data.data);  // data.data car structure {type, data}
  });

  return eventSource;
}
```

### Configuration Nginx pour SSE

```nginx
location /workflow/timeline/stream {
    proxy_pass http://workflow/timeline/stream;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_set_header X-Accel-Buffering 'no';
    proxy_buffering off;
    proxy_cache off;
    tcp_nodelay on;
    proxy_read_timeout 86400s;
}
```

---

## Base de données PostgreSQL

### Tables principales

| Table | Description |
|-------|-------------|
| `users` | Utilisateurs avec `accreditation_level` (N0-N4) |
| `conversations` | Métadonnées conversations chat |
| `messages` | Messages individuels avec analytics |
| `user_token_usage` | Tracking quota tokens par mois |
| `safeguard_config` | Configuration niveau de sécurité par outil |
| `safeguard_requests` | Demandes d'approbation pending/approved/rejected |
| `safeguard_audit` | Historique des approbations/rejets |
| `safeguard_deferred_actions` | Actions L4 en attente d'exécution différée |
| `agent_sessions` | Sessions agent avec état + history |
| `mcp_connectors` | Configuration services externes |
| `system_settings` | Configuration système globale |

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

-- Stats utilisation tokens par utilisateur
SELECT u.username, SUM(utu.tokens_used) as total FROM users u
JOIN user_token_usage utu ON u.id = utu.user_id
GROUP BY u.username ORDER BY total DESC;
```

---

## Authentification & Autorisation

### JWT Authentication

- Token émis à la connexion, expire après 24h (configurable)
- Stocké dans localStorage côté frontend
- Envoyé comme header `Authorization: Bearer <token>`
- Vérifié par middleware FastAPI

### Niveaux utilisateurs

| Niveau | Rôle | Permissions |
|--------|------|-------------|
| N0 | Viewer | Accès lecture seule au chat |
| N1 | Technicien | Peut exécuter outils L0-L1 |
| N2 | Senior Tech | Peut exécuter outils L0-L2 |
| N3 | Admin | Peut approuver actions L3, gérer utilisateurs |
| N4 | SuperAdmin | Accès complet, override SAFEGUARD |

### Routes protégées

Frontend routes protégées avec composant `<ProtectedRoute>` qui vérifie `useAuthStore()`.

---

## Intégrations externes

| Service | Type | Usage | Auth |
|---------|------|-------|------|
| **GLPI** | REST API | Tickets, KB, entités | App token + User token |
| **Observium** | REST API | Monitoring réseau | Username + Password |
| **Active Directory** | LDAP | Infos utilisateurs, gestion comptes | Bind user + Password |
| **AllMySMS** | REST API | Envoi SMS | Login + API key |
| **Mistral AI** | REST API | LLM & embeddings | API key |
| **Backoffice WIDIP** | REST API | Création alarmes | API key |
| **Redis** | Cache/Queue | Sessions, cache référents | URL |
| **PostgreSQL** | Database | Données principales | DSN |

---

## Services Docker

Le fichier `docker-compose.yml` définit 6 services :

| Service | Image | Port | Description |
|---------|-------|------|-------------|
| **redis** | redis:7-alpine | 6379 | Cache, sessions, referent_cache |
| **postgres** | pgvector/pgvector:pg14 | 5433 | BDD principale avec embeddings vectoriels |
| **backend** | custom (FastAPI) | 8000 | API backend core |
| **mcp-server** | custom (Python) | 3001 | Moteur d'exécution outils |
| **frontend** | custom (Nginx) | - | App React (servie via Nginx) |
| **workflow-observium** | custom (Python) | 8001 | Monitoring proactif |
| **nginx** | nginx:alpine | 8080 | Reverse proxy (tous services) |

### Volumes

- `postgres_data` - Persistance BDD
- `redis_data` - Persistance Redis
- `./rag-documents/` - Documents base de connaissances
- `./uploads/` - Fichiers uploadés utilisateurs

---

## Commandes Docker utiles

```bash
# Depuis C:\Users\maxime\Documents\WIDIP\WIDIP_ARCHI\01_WIBOT
# (ou /c/Users/maxime/Documents/WIDIP/WIDIP_ARCHI/01_WIBOT en Git Bash)

# Rebuild un service (--no-cache obligatoire après modif code)
docker-compose build backend --no-cache
docker-compose build frontend --no-cache
docker-compose build mcp-server --no-cache

# Restart un service
docker-compose up -d backend
docker-compose up -d frontend

# Voir les logs
docker-compose logs -f backend
docker-compose logs -f mcp-server
docker-compose logs -f workflow-observium

# Accès DB PostgreSQL
docker exec -it wibot-postgres psql -U widip -d wibot

# Status conteneurs
docker-compose ps

# Health checks
curl http://localhost:8000/health
curl http://localhost:3001/health
curl http://localhost:8001/health
```

---

## URLs de développement

| Service | URL |
|---------|-----|
| **Frontend** | http://localhost:8080 |
| **Backend API (Swagger)** | http://localhost:8000/docs |
| **MCP Server** | http://localhost:3001 |
| **Workflow API** | http://localhost:8001 |
| **PostgreSQL** | localhost:5433 (user: widip, db: wibot) |
| **Redis** | localhost:6379 |
| **Monitoring Workflow** | http://localhost:8080/monitoring |

---

## Points d'attention pour le développement

### Règles critiques

1. **Docker build obligatoire** : Le code est copié à la construction. Après toute modification, rebuild avec `--no-cache`.

2. **Synchronisation SAFEGUARD** : Les niveaux doivent être identiques dans `safeguard_config` (DB) et `config.py` (MCP server).

3. **Polling frontend** : `Chat.tsx` a un `useEffect` avec `setInterval(3000)` qui vérifie le statut de session quand `waitingForApproval` est true.

4. **Bypass SAFEGUARD** : Après approbation, toujours passer `_bypass_safeguard=True` pour éviter le re-blocage MCP.

5. **Streaming SSE** : Backend→Frontend utilise Server-Sent Events, pas WebSocket.

6. **Prompt Agent** : Le prompt système est dans `agent_streaming_service.py` (`AGENT_SYSTEM_PROMPT`). Il contient les règles SAFEGUARD et instructions.

7. **RAG Embeddings** : Utilise API Mistral, documents stockés dans PostgreSQL pgvector.

8. **Token Quota** : Tracké mensuellement par utilisateur dans `user_token_usage`.

### Outils critiques par niveau

| Niveau | Outils importants |
|--------|-------------------|
| L2 | `glpi_create_ticket`, `ad_unlock_account`, `glpi_assign_ticket` |
| L3 | `glpi_close_ticket`, `ad_reset_password`, `ad_disable_account`, `smtp_send_email` |
| L4 | `ad_create_user`, `sms_alert_send`, `glpi_create_alarm` |
| L4_SMS | `sms_alert_device_down` |

---

## Dernières modifications (Janvier-Février 2026)

### Refonte Page Monitoring + SSE temps réel (3 février 2026)

**Problème initial** : Les alertes n'apparaissaient pas car marquées "completed" dès que SMS envoyé.

**Corrections apportées :**

1. **Logique workflow corrigée** (`workflow_engine.py`) :
   - Ticket créé uniquement après 20 min (pas immédiatement)
   - Actions L1 automatiques (pas de validation SAFEGUARD)
   - SMS inclut le numéro de ticket

2. **SSE temps réel fonctionnel** :
   - Format SSE corrigé (dictionnaire pour `sse_starlette`)
   - Nginx configuré avec `X-Accel-Buffering: no`
   - Barres de progression mises à jour toutes les 5s

3. **Nouveaux endpoints API** :
   - `DELETE /timeline/{hostname}` - Supprimer alerte de test
   - `PATCH /config` - Modifier configuration runtime
   - `POST /actions/trigger` - Déclencher action manuelle
   - `POST /actions/{id}/approve|reject` - Approbation/rejet SAFEGUARD
   - `GET /resolved-alerts` - Historique alertes résolues

4. **Frontend amélioré** :
   - Bouton suppression alerte avec confirmation
   - Reconnexion SSE automatique après 5s
   - Logs de débogage SSE dans console

**Fichiers modifiés :**
- `workflow_engine.py` : `_process_mail_alerts()`, `_analyze_and_decide()`
- `timeline_service.py` : `stream_snapshots()` (format dict SSE)
- `timeline.py` : `to_sse_event()` (ajout `client_name`)
- `Monitoring.tsx` : Gestion SSE avec cleanup
- `workflow.ts` : `createTimelineStream()` avec onOpen callback
- `ActiveAlertCard.tsx` : Bouton suppression
- `nginx.conf` : Config SSE (tcp_nodelay, X-Accel-Buffering)

### Nouveau : Système Alerte SMS L4_SMS (28 janvier 2026)

- Nouveau niveau SAFEGUARD `SecurityLevel.L4_SMS`
- Cache Redis référents avec TTL 7-14 jours (`referent_cache.py`)
- Parsing hostname Observium (`parse_observium_hostname()`)
- Recherche DirEnt actif avec score d'activité
- Intégration API AllMySMS + Backoffice alarmes

### Bug corrigé : Double blocage SAFEGUARD

**Problème** : Après approbation backend, le MCP server re-bloquait l'outil.

**Solution** : Paramètre `_bypass_safeguard` dans la chaîne d'appel :

```python
# mcp-server/src/mcp/server.py - endpoint /tools/{tool_name}
bypass_safeguard = arguments.pop("_bypass_safeguard", False)
if bypass_safeguard:
    safeguard_result = SafeguardResponse(allowed=True, ...)

# wibot-backend/app/services/mcp_client.py
async def call_tool(self, tool_name, params, bypass_safeguard=False, approval_id=None):
    if bypass_safeguard:
        request_params["_bypass_safeguard"] = True
        request_params["_approval_id"] = approval_id
```

### Approbation inline dans le chat

**Fichier** : `AgentStreaming.tsx`

- Boutons "Approuver" / "Rejeter" dans le bandeau SAFEGUARD orange
- États : `approvalLoading`, `approvalError`, `approvalDone`
- Callback `onApprovalAction` pour déclencher le polling

---

## Configuration SMTP (Workflow)

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

### Outils SMTP

| Tool | Niveau | Description |
|------|--------|-------------|
| `smtp_test_connection` | L0 | Test connexion SMTP |
| `smtp_send_email` | L3 | Envoi email générique |
| `smtp_send_alert` | L3 | Envoi alerte device DOWN |

---

## Documentation supplémentaire

| Document | Localisation | Description |
|----------|--------------|-------------|
| Architecture fonctionnelle | `docs/ARCHITECTURE_FONCTIONNELLE_WIDIP.md` | Vue d'ensemble architecture |
| Flux SMS détaillé | `docs/FLUX_ALERTE_SMS.md` | Flux complet alertes SMS |
| Guide API GLPI | `docs/api-guides/Guide_API_GLPI.md` | Référence API GLPI |
| Guide API Observium | `docs/api-guides/Guide_API_Observium.md` | Référence API Observium |
