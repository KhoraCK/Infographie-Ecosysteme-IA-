# 🏗️ Architecture WIDIP - Automatisation IA des Opérations IT

> **Version** : 5.0  
> **Date** : 9 Décembre 2025  
> **Auteur** : Documentation technique WIDIP  
> **Statut** : Production Ready - Architecture simplifiée

---

## 📋 Table des matières

1. [Contexte et vision](#1-contexte-et-vision)
2. [Infrastructure et IDs](#2-infrastructure-et-ids)
3. [Vue d'ensemble de l'architecture](#3-vue-densemble-de-larchitecture)
4. [Les 4 workflows principaux](#4-les-4-workflows-principaux)
5. [Système MCP : les outils des agents](#5-système-mcp--les-outils-des-agents)
6. [Redis : mémoire partagée et synchronisation](#6-redis--mémoire-partagée-et-synchronisation)
7. [Credentials et sécurité](#7-credentials-et-sécurité)
8. [Déploiement et routine](#8-déploiement-et-routine)
9. [Changelog v4 → v5](#9-changelog-v4--v5)

---

## 1. Contexte et vision

### 1.1 Qui est WIDIP ?

**WIDIP** est une entreprise d'hébergement de données spécialisée dans le secteur **médico-social**. Elle accompagne plus de **800 établissements** (EHPAD, cliniques, associations, structures d'accueil) dans la gestion de leur infrastructure IT.

### 1.2 Le défi opérationnel

```
┌─────────────────────────────────────────────────────────────────┐
│                    VOLUME ANNUEL WIDIP                          │
├─────────────────────────────────────────────────────────────────┤
│  📊 ~20 000 tickets de support / an                             │
│  🏥 800+ établissements clients                                 │
│  🌐 Infrastructure réseau distribuée                            │
│  ⏰ Support 24/7 requis pour le secteur santé                   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 La solution : automatisation par IA

| Capacité | Description |
|----------|-------------|
| 🔍 **Diagnostiquer** | Analyser automatiquement les alertes réseau |
| 🎫 **Créer des tickets** | Ouvrir des tickets GLPI enrichis |
| 📧 **Notifier** | Envoyer des emails contextualisés via GLPI |
| 🔧 **Agir** | Réinitialiser mots de passe, débloquer comptes, créer comptes |
| 📊 **Prioriser** | Trier les incidents par criticité |

---

## 2. Infrastructure et IDs

### 2.1 Informations VM

```
┌─────────────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE VM                             │
├─────────────────────────────────────────────────────────────────┤
│  🖥️  ID OVH         : wh38667-ovh                               │
│  🌐 IP VM           : 10.238.110.30                             │
│  🤖 API Ollama      : 9b2d80f1962c49f5acb090580c2ae50b...       │
│  🎮 GPU prévu       : NVIDIA L40S (VM2 en provisionnement)      │
│  🧠 Modèle cible    : Qwen3-Coder:30b                           │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 IDs des Workflows

```
┌─────────────────────────────────────────────────────────────────┐
│                    WORKFLOWS PRINCIPAUX                          │
├──────────────────────────────────┬──────────────────────────────┤
│  Workflow                        │  ID n8n                      │
├──────────────────────────────────┼──────────────────────────────┤
│  🚨 WIDIP_Proactif_Observium_v8  │  zBYBmEHgXDT2nWHJ            │
│  🎫 WIDIP_Assist_ticket_v4       │  8muMSPoq5CW8oHGq            │
│  🔧 WIDIP_Redis_Helper_v2.2      │  aCuwZ3jJb1c2dMVY            │
│  🏥 WIDIP_Health_Check_GLPI_v2   │  yNJ1g9IKO2ZSQZ2M            │
└──────────────────────────────────┴──────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    WORKFLOWS MCP (4 outils)                      │
├──────────────────────────────────┬──────────────────────────────┤
│  MCP Workflow                    │  ID n8n                      │
├──────────────────────────────────┼──────────────────────────────┤
│  📊 MCP_Observium                │  QMnaB7g1DpUfN9rK            │
│  🎫 MCP_GLPI_v7                  │  juHWzwO4WgW9LX0B            │
│  👤 MCP_ActiveDirectory_v2       │  2tSBUs69Kn3NYxxT            │
│  🔐 MCP_MySecret_v1              │  geNUXjUljodz4CPG            │
└──────────────────────────────────┴──────────────────────────────┘
```

### 2.3 Endpoints MCP (SSE)

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIENS MCP SERVER                              │
├──────────────────────────────────────────────────────────────────┤
│  MCP Observium  : http://0.0.0.0:5678/mcp-test/mcp-observium/sse│
│  MCP GLPI       : http://0.0.0.0:5678/mcp-test/mcp-glpi-server/sse│
│  MCP AD         : http://0.0.0.0:5678/mcp-test/mcp-ad/sse       │
│  MCP MySecret   : http://0.0.0.0:5678/mcp-test/mcp-mysecret/sse │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Vue d'ensemble de l'architecture

### 3.1 Schéma global

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                        ARCHITECTURE WIDIP v5 - VUE GLOBALE                    ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║   ┌─────────────────┐         ┌─────────────────┐                           ║
║   │   OBSERVIUM     │         │     GLPI        │                           ║
║   │  (Monitoring)   │         │ (Tickets+Email) │                           ║
║   └────────┬────────┘         └────────┬────────┘                           ║
║            │ Webhooks                   │ API REST                          ║
║            ▼                            ▼                                    ║
║   ┌────────────────────────────────────────────────────────────┐            ║
║   │                         n8n (VM1)                          │            ║
║   │  ┌──────────────────┐    ┌──────────────────┐              │            ║
║   │  │  🚨 PROACTIF v8  │    │  🎫 ASSIST v4    │              │            ║
║   │  │  (Alertes réseau)│    │  (Tickets support)│             │            ║
║   │  └────────┬─────────┘    └────────┬─────────┘              │            ║
║   │           │                       │                         │            ║
║   │           ▼                       ▼                         │            ║
║   │  ┌─────────────────────────────────────────────┐           │            ║
║   │  │         🔧 MCP WORKFLOWS (4 outils)         │           │            ║
║   │  │    Observium │ GLPI │ Active Directory │ MySecret      │            ║
║   │  └──────────────────┬──────────────────────────┘           │            ║
║   │                     │                                       │            ║
║   │  ┌──────────────────┴──────────────────────┐               │            ║
║   │  │         🗄️ REDIS HELPER v2.2            │               │            ║
║   │  │    (Cache, Mutex, Circuit Breaker)      │               │            ║
║   │  └─────────────────────────────────────────┘               │            ║
║   └────────────────────────────────────────────────────────────┘            ║
║                              │                                               ║
║                              ▼                                               ║
║   ┌─────────────────────────────────────────────────────────────┐           ║
║   │  🤖 OLLAMA (VM2 - GPU L40S)                                 │           ║
║   │  Modèle: Qwen3-Coder:30b (30B params, ~18GB VRAM Q4)        │           ║
║   │  Vitesse: ~30-50 tokens/sec                                 │           ║
║   └─────────────────────────────────────────────────────────────┘           ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### 3.2 Les 6 couches de l'architecture

```
┌────────────────────────────────────────────────────────────────┐
│  COUCHE 1 : SOURCES D'ÉVÉNEMENTS                               │
│  • Observium (webhooks alertes réseau)                         │
│  • GLPI (tickets support via webhook)                          │
│  • Health Check (monitoring santé GLPI)                        │
└────────────────────────────────────────────────────────────────┘
         ▼
┌────────────────────────────────────────────────────────────────┐
│  COUCHE 2 : WORKFLOWS PRINCIPAUX (n8n)                         │
│  • PROACTIF v8 : Diagnostique automatique des pannes réseau   │
│  • ASSIST v4 : Traitement intelligent des tickets support     │
│  • HEALTH CHECK v2 : Surveillance continue de GLPI            │
└────────────────────────────────────────────────────────────────┘
         ▼
┌────────────────────────────────────────────────────────────────┐
│  COUCHE 3 : SYSTÈME MCP (4 outils)                             │
│  • Observium : Récupération d'infos réseau                     │
│  • GLPI : Gestion tickets + envoi emails                       │
│  • Active Directory : Gestion comptes utilisateurs             │
│  • MySecret : Génération liens sécurisés temporaires           │
└────────────────────────────────────────────────────────────────┘
         ▼
┌────────────────────────────────────────────────────────────────┐
│  COUCHE 4 : REDIS HELPER                                       │
│  • Cache : Évite les appels API redondants                     │
│  • Mutex : Sérialise l'accès à Ollama (1 seul à la fois)      │
│  • Circuit Breaker : Protège contre les pannes GLPI           │
└────────────────────────────────────────────────────────────────┘
         ▼
┌────────────────────────────────────────────────────────────────┐
│  COUCHE 5 : INTELLIGENCE ARTIFICIELLE                          │
│  • Ollama + Qwen3-Coder:30b : Raisonnement et décisions       │
│  • Génération de diagnostics techniques                        │
│  • Emails contextualisés et professionnels                     │
└────────────────────────────────────────────────────────────────┘
         ▼
┌────────────────────────────────────────────────────────────────┐
│  COUCHE 6 : SYSTÈMES EXTERNES                                  │
│  • API Observium (monitoring réseau)                           │
│  • API GLPI (tickets + emails)                                 │
│  • Active Directory (gestion utilisateurs)                     │
│  • API MySecret (partage sécurisé)                             │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. Les 4 workflows principaux

### 4.1 🚨 WIDIP_Proactif_Observium_v8

**Rôle** : Diagnostic automatique des alertes réseau avec détermination de responsabilité.

**Déclenchement** : Webhook Observium (port down, device down).

**Flux** :
```
Webhook Observium
    ↓
Redis: Mutex "ollama_lock"
    ↓
Redis: Check GLPI Health (circuit breaker)
    ↓
Si GLPI OK → Agent Sentinel (diagnostic IA)
    ↓
Détermine: WIDIP ou FAI responsable ?
    ↓
Crée ticket GLPI approprié
    ↓
Envoie email via GLPI
    ↓
Redis: Release Mutex
```

**Temps cible** : < 60 secondes (alerte → ticket + email)

**MCP utilisés** : Observium, GLPI

---

### 4.2 🎫 WIDIP_Assist_ticket_v4

**Rôle** : Traitement intelligent des tickets support avec actions automatiques.

**Déclenchement** : Webhook GLPI (nouveau ticket ou mise à jour).

**Flux** :
```
Webhook GLPI
    ↓
Catégorisation du ticket
    ↓
Redis: Mutex "ollama_lock"
    ↓
Redis: Check GLPI Health (circuit breaker)
    ↓
Si GLPI OK → Agent Analyst (analyse IA)
    ↓
Décisions automatiques:
    - Reset password AD → MySecret → Email
    - Déblocage compte AD
    - Création compte AD + GLPI
    - Mise à jour ticket GLPI
    ↓
Redis: Release Mutex
```

**Temps cible** : 
- Simple ticket : < 90 secondes
- Reset password : < 30 secondes
- Création compte : < 60 secondes

**MCP utilisés** : GLPI, Active Directory, MySecret

---

### 4.3 🏥 WIDIP_Health_Check_GLPI_v2

**Rôle** : Surveillance continue de la disponibilité de l'API GLPI.

**Déclenchement** : Cron (toutes les 30 secondes).

**Flux** :
```
Cron trigger (30s)
    ↓
Test API GLPI: /apirest.php/
    ↓
Redis: Update "glpi_health_status"
    ↓
Si DOWN:
    - Redis: Check "glpi_down_alert_sent"
    - Si pas encore notifié → Email admin
    - Redis: Set "glpi_down_alert_sent" (TTL 1h)
    ↓
Si UP:
    - Redis: Delete "glpi_down_alert_sent"
```

**Objectif** : Circuit breaker pour éviter les tentatives sur GLPI en panne.

---

### 4.4 🔧 WIDIP_Redis_Helper_v2.2

**Rôle** : Service centralisé pour toutes les opérations Redis.

**Déclenchement** : Sub-workflow appelé par les autres workflows.

**Opérations supportées** :
```
┌─────────────────────────────────────────────────────────────────┐
│  ACTION     │  DESCRIPTION                                       │
├─────────────┼────────────────────────────────────────────────────┤
│  get        │  Récupérer une valeur                              │
│  set        │  Stocker une valeur avec TTL optionnel             │
│  delete     │  Supprimer une clé                                 │
│  acquire    │  Acquérir un mutex (SETNX + TTL)                   │
│  release    │  Libérer un mutex avec vérification ownership      │
│  exists     │  Vérifier l'existence d'une clé                    │
└─────────────┴────────────────────────────────────────────────────┘
```

**Clés Redis importantes** :
- `ollama_lock` : Mutex pour l'accès exclusif à Ollama
- `glpi_health_status` : État de santé de GLPI (UP/DOWN)
- `glpi_down_alert_sent` : Flag d'alerte GLPI déjà envoyée
- `observium_device_*` : Cache des infos devices Observium
- `observium_port_*` : Cache des infos ports Observium

---

## 5. Système MCP : les outils des agents

### 5.1 Vue d'ensemble

Le système **MCP** (Model Context Protocol) permet aux agents IA d'interagir avec les systèmes externes via des **outils standardisés**.

```
┌─────────────────────────────────────────────────────────────────┐
│                   SYSTÈME MCP - 4 OUTILS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 📊 MCP OBSERVIUM                                            │
│     → Récupération infos réseau (devices, ports, alertes)      │
│                                                                 │
│  2. 🎫 MCP GLPI v7                                              │
│     → Gestion tickets + création comptes + envoi emails        │
│                                                                 │
│  3. 👤 MCP ACTIVE DIRECTORY v2                                  │
│     → Gestion complète comptes utilisateurs Windows            │
│                                                                 │
│  4. 🔐 MCP MYSECRET v1                                          │
│     → Génération liens temporaires sécurisés                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.2 📊 MCP Observium

**ID** : `QMnaB7g1DpUfN9rK`  
**Endpoint** : `http://0.0.0.0:5678/mcp-test/mcp-observium/sse`

**Opérations disponibles** :

```
┌─────────────────────────────────────────────────────────────────┐
│  get_device_info(device_id, hostname)                           │
│  → Récupère les infos complètes d'un device                     │
│  → Cache Redis: observium_device_{id} (TTL 5 min)              │
│                                                                 │
│  get_port_info(device_id, port_id, port_label)                 │
│  → Récupère l'état détaillé d'un port                          │
│  → Cache Redis: observium_port_{device_id}_{port_id} (TTL 5min)│
│                                                                 │
│  list_device_ports(device_id, hostname)                        │
│  → Liste tous les ports d'un device                            │
│  → Cache Redis: observium_ports_{device_id} (TTL 5 min)        │
└─────────────────────────────────────────────────────────────────┘
```

**Usage typique** :
```javascript
// Dans Agent Sentinel (Proactif v8)
{
  "operation": "get_device_info",
  "device_id": "1234",
  "hostname": "sw-client-ehpad-01"
}
```

---

### 5.3 🎫 MCP GLPI v7

**ID** : `juHWzwO4WgW9LX0B`  
**Endpoint** : `http://0.0.0.0:5678/mcp-test/mcp-glpi-server/sse`

**Opérations disponibles** :

```
┌─────────────────────────────────────────────────────────────────┐
│  GESTION TICKETS                                                │
│  ────────────────                                               │
│  create_ticket(title, description, urgency, category)           │
│  → Crée un nouveau ticket GLPI                                  │
│                                                                 │
│  update_ticket(ticket_id, content, status)                     │
│  → Ajoute un suivi ou change le statut                         │
│                                                                 │
│  get_ticket_info(ticket_id)                                    │
│  → Récupère les détails complets d'un ticket                   │
│                                                                 │
│  ────────────────────────────────────────────────────────────  │
│  GESTION UTILISATEURS GLPI                                      │
│  ──────────────────────                                         │
│  create_glpi_user(firstname, lastname, email, login)           │
│  → Crée un compte utilisateur GLPI                             │
│                                                                 │
│  get_glpi_user(email, login)                                   │
│  → Recherche un utilisateur GLPI                               │
│                                                                 │
│  update_glpi_user(user_id, updates)                            │
│  → Met à jour un utilisateur GLPI                              │
│                                                                 │
│  disable_glpi_user(user_id)                                    │
│  → Désactive un compte GLPI                                     │
│                                                                 │
│  ────────────────────────────────────────────────────────────  │
│  ENVOI EMAILS (remplace MCP SMTP)                              │
│  ─────────────────────────────────                              │
│  send_email(recipient, subject, body, ticket_id)               │
│  → Envoie un email via GLPI (attaché au ticket si fourni)     │
│  → Supporte HTML et texte brut                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Exemple création ticket + email** :
```javascript
// Étape 1: Créer le ticket
{
  "operation": "create_ticket",
  "title": "Panne réseau EHPAD Les Roses",
  "description": "Port Gi0/12 DOWN depuis 14:23",
  "urgency": "4",
  "category": "réseau"
}

// Étape 2: Envoyer email
{
  "operation": "send_email",
  "recipient": "tech@widip.fr",
  "subject": "URGENT: Panne réseau EHPAD Les Roses",
  "body": "<html>...</html>",
  "ticket_id": "12345"
}
```

---

### 5.4 👤 MCP Active Directory v2

**ID** : `2tSBUs69Kn3NYxxT`  
**Endpoint** : `http://0.0.0.0:5678/mcp-test/mcp-ad/sse`

**Opérations disponibles** :

```
┌─────────────────────────────────────────────────────────────────┐
│  search_user(samaccountname, email, displayname)               │
│  → Recherche un utilisateur dans l'AD                          │
│  → Retourne infos complètes (DN, UPN, status, groupes...)      │
│                                                                 │
│  create_user(firstname, lastname, username, ou_path)           │
│  → Crée un nouveau compte utilisateur AD                       │
│  → Génère mot de passe aléatoire sécurisé                      │
│  → Retourne le mot de passe temporaire                         │
│                                                                 │
│  reset_password(samaccountname, new_password)                  │
│  → Réinitialise le mot de passe                                │
│  → Force changement au prochain login                          │
│                                                                 │
│  unlock_account(samaccountname)                                │
│  → Déverrouille un compte bloqué                               │
│                                                                 │
│  disable_account(samaccountname)                               │
│  → Désactive un compte utilisateur                             │
│                                                                 │
│  enable_account(samaccountname)                                │
│  → Réactive un compte désactivé                                │
│                                                                 │
│  move_to_ou(samaccountname, target_ou)                         │
│  → Déplace un utilisateur vers une autre OU                    │
│                                                                 │
│  copy_groups_from(source_user, target_user)                    │
│  → Copie les groupes d'un utilisateur vers un autre           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Sécurité** :
- Validation stricte de tous les inputs
- Sanitization anti-injection PowerShell
- Logs détaillés de toutes les actions

**Exemple reset password** :
```javascript
// Étape 1: Générer nouveau mot de passe et reset AD
{
  "operation": "reset_password",
  "samaccountname": "jdupont"
}
// Retour: { "password": "Temp123!XyZ@", "must_change": true }

// Étape 2: Créer lien MySecret
{
  "operation": "create_secret",
  "payload": "Votre nouveau mot de passe: Temp123!XyZ@",
  "expire_days": 7,
  "expire_views": 5
}

// Étape 3: Envoyer email avec lien MySecret
{
  "operation": "send_email",
  "recipient": "jdupont@client.fr",
  "subject": "Réinitialisation mot de passe",
  "body": "Cliquez ici: https://mysecret.widip.fr/p/abc123"
}
```

---

### 5.5 🔐 MCP MySecret v1

**ID** : `geNUXjUljodz4CPG`  
**Endpoint** : `http://0.0.0.0:5678/mcp-test/mcp-mysecret/sse`

**Opérations disponibles** :

```
┌─────────────────────────────────────────────────────────────────┐
│  create_secret(payload, expire_days, expire_views)             │
│  → Crée un lien temporaire sécurisé                            │
│  → expire_days: durée de vie (défaut 7 jours)                 │
│  → expire_views: nombre de vues max (défaut 5)                │
│  → Retourne: https://mysecret.widip.fr/p/{token}              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**API backend** : `https://mysecret.widip.fr` (self-hosted)

**Usage typique** :
```javascript
{
  "operation": "create_secret",
  "payload": "Mot de passe temporaire: Temp123!XyZ@\nÀ changer dès la première connexion.",
  "expire_days": 7,
  "expire_views": 3
}

// Retour:
{
  "success": true,
  "secret_url": "https://mysecret.widip.fr/p/a8b3c9d2e1f4",
  "expires_in_days": 7,
  "expires_after_views": 3
}
```

---

## 6. Redis : mémoire partagée et synchronisation

### 6.1 Architecture Redis

```
┌─────────────────────────────────────────────────────────────────┐
│                    REDIS - RÔLES MULTIPLES                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 💾 CACHE API                                                │
│     → Évite appels répétés à Observium                         │
│     → TTL: 5 minutes                                            │
│     → Clés: observium_device_*, observium_port_*               │
│                                                                 │
│  2. 🔒 MUTEX OLLAMA                                             │
│     → 1 seul workflow à la fois utilise Ollama                 │
│     → Clé: ollama_lock                                         │
│     → TTL: 120s (safety timeout)                               │
│     → Ownership: UUID unique par workflow                      │
│                                                                 │
│  3. 🛡️ CIRCUIT BREAKER GLPI                                     │
│     → Évite tentatives si GLPI est DOWN                        │
│     → Clé: glpi_health_status                                  │
│     → Valeurs: "UP" ou "DOWN"                                  │
│     → TTL: 120s (re-test auto)                                 │
│                                                                 │
│  4. 🚨 ANTI-SPAM ALERTES                                        │
│     → Évite alertes dupliquées                                 │
│     → Clé: glpi_down_alert_sent                                │
│     → TTL: 3600s (1 alerte/heure max)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Patterns Redis avancés

#### 6.2.1 Mutex avec ownership

```javascript
// Acquisition
{
  "action": "acquire",
  "key": "ollama_lock",
  "lock_id": "proactif_v8_abc123",  // UUID unique
  "ttl": 120
}

// Libération sécurisée
// 1. GET pour vérifier ownership
{
  "action": "get",
  "key": "ollama_lock"
}
// 2. DELETE seulement si c'est notre lock
if (redis_value === our_lock_id) {
  {
    "action": "delete",
    "key": "ollama_lock"
  }
}
```

#### 6.2.2 Circuit Breaker

```javascript
// Health Check écrit l'état
{
  "action": "set",
  "key": "glpi_health_status",
  "value": "DOWN",
  "ttl": 120
}

// Workflows lisent l'état avant d'utiliser GLPI
{
  "action": "get",
  "key": "glpi_health_status"
}
// Si "DOWN" → skip GLPI, log erreur
// Si "UP" ou absent → proceed normalement
```

#### 6.2.3 Cache avec invalidation

```javascript
// Écriture cache
{
  "action": "set",
  "key": "observium_device_1234",
  "value": "{...json_data...}",
  "ttl": 300  // 5 minutes
}

// Lecture cache
{
  "action": "get",
  "key": "observium_device_1234"
}
// Si absent ou expiré → appel API Observium
```

---

## 7. Credentials et sécurité

### 7.1 Variables d'environnement

Toutes les credentials sont stockées comme **variables d'environnement n8n** :

```
┌─────────────────────────────────────────────────────────────────┐
│                    VARIABLES N8N                                 │
├──────────────────────────────┬──────────────────────────────────┤
│  Variable                    │  Usage                           │
├──────────────────────────────┼──────────────────────────────────┤
│  OBSERVIUM_API_URL           │  https://observium.widip.fr      │
│  OBSERVIUM_API_TOKEN         │  Token Observium                 │
│                              │                                  │
│  GLPI_API_URL                │  https://glpi.widip.fr/apirest.php│
│  GLPI_USER_TOKEN             │  Token utilisateur GLPI          │
│  GLPI_APP_TOKEN              │  Token application GLPI          │
│                              │                                  │
│  AD_SERVER                   │  dc01.widip.local                │
│  AD_USERNAME                 │  svc-n8n@widip.local             │
│  AD_PASSWORD                 │  ***************                 │
│  AD_BASE_DN                  │  DC=widip,DC=local               │
│                              │                                  │
│  REDIS_HOST                  │  localhost                       │
│  REDIS_PORT                  │  6379                            │
│  REDIS_PASSWORD              │  ***************                 │
│                              │                                  │
│  OLLAMA_BASE_URL             │  http://10.238.110.30:11434      │
│  OLLAMA_MODEL                │  qwen3-coder:30b                 │
│                              │                                  │
│  MYSECRET_API_URL            │  https://mysecret.widip.fr       │
│                              │                                  │
└──────────────────────────────┴──────────────────────────────────┘
```

### 7.2 Sécurité Active Directory

**Compte de service n8n** :
- Permissions minimales nécessaires
- Délégation de contrôle sur les OUs spécifiques
- Pas d'admin de domaine

**Validations** :
- Anti-injection PowerShell
- Regex strictes sur tous les inputs
- Logging détaillé de toutes les actions

---

## 8. Déploiement et routine

### 8.1 Checklist de déploiement

```
┌─────────────────────────────────────────────────────────────────┐
│                    DÉPLOIEMENT v5                                │
└─────────────────────────────────────────────────────────────────┘

 ☐ 1. INFRASTRUCTURE
    ☐ Redis installé et configuré
    ☐ Variables d'environnement n8n configurées
    ☐ Ollama installé avec Qwen3-Coder:30b
    ☐ MCP Server n8n activé sur port 5678

 ☐ 2. WORKFLOWS MCP
    ☐ Importer WIDIP_MCP_Observium.json
    ☐ Importer WIDIP_MCP_GLPI_v7.json
    ☐ Importer WIDIP_MCP_ActiveDirectory_v2.json
    ☐ Importer WIDIP_MCP_MySecret_v1.json
    ☐ Vérifier les endpoints SSE

 ☐ 3. WORKFLOWS PRINCIPAUX
    ☐ Importer WIDIP_Redis_Helper_v2.2.json
    ☐ Importer WIDIP_Health_Check_GLPI_v2.json → Activer
    ☐ Importer WIDIP_Proactif_Observium_v8.json → Activer
    ☐ Importer WIDIP_Assist_ticket_v4.json → Activer

 ☐ 4. CONFIGURATION EXTERNE
    ☐ Configurer webhook Observium → n8n
    ☐ Configurer webhook GLPI → n8n
    ☐ Tester connectivité API Observium
    ☐ Tester connectivité API GLPI
    ☐ Tester connectivité Active Directory
    ☐ Tester API MySecret

 ☐ 5. TESTS DE VALIDATION
    ☐ Test Health Check GLPI (doit logger toutes les 30s)
    ☐ Test Redis Helper (get/set/delete)
    ☐ Test acquisition/release mutex
    ☐ Test création ticket GLPI
    ☐ Test envoi email via GLPI
    ☐ Test reset password AD + MySecret
    ☐ Simuler alerte Observium → vérifier ticket créé
```

### 8.2 Monitoring quotidien

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROUTINE DE MONITORING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CHAQUE MATIN (9h):                                             │
│  • Vérifier logs n8n (erreurs, warnings)                        │
│  • Vérifier état Redis (nb clés, mémoire)                       │
│  • Vérifier logs Ollama (crashes, OOM)                          │
│  • Check GLPI health status dans Redis                          │
│                                                                 │
│  CHAQUE SEMAINE:                                                │
│  • Review des tickets créés automatiquement                     │
│  • Analyse des temps de réponse (métriques)                     │
│  • Vérification espace disque VM                                │
│  • Update documentation si changements                          │
│                                                                 │
│  CHAQUE MOIS:                                                   │
│  • Backup complet des workflows n8n                             │
│  • Test de restauration depuis backup                           │
│  • Review sécurité (credentials, permissions AD)                │
│  • Audit des logs pour patterns inhabituels                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Changelog v4 → v5

### 9.1 🗑️ Suppressions

```
┌─────────────────────────────────────────────────────────────────┐
│                        COMPOSANTS RETIRÉS                        │
└─────────────────────────────────────────────────────────────────┘

  ❌ MCP Phibee Telecom v5 (Tg7LLuwXOvgTyB0h)
  ───────────────────────────────────────────
  Raison: Phibee n'a pas d'API publique disponible
  Impact: Les diagnostics FAI sont maintenant manuels
  Solution future: Script client de diagnostic local (en réflexion)
  
  ❌ MCP SMTP v5 (9R8EEGECrVexWdW7)
  ──────────────────────────────────
  Raison: Remplacé par les fonctions email de GLPI
  Avantages:
    • Moins de complexité (1 MCP en moins)
    • Emails automatiquement liés aux tickets GLPI
    • Historique centralisé dans GLPI
    • Meilleure traçabilité
```

### 9.2 ✨ Simplifications

```
┌─────────────────────────────────────────────────────────────────┐
│                        AMÉLIORATIONS v5                          │
└─────────────────────────────────────────────────────────────────┘

  ✅ Architecture simplifiée
  ─────────────────────────
  • Passage de 6 MCP à 4 MCP
  • Stack plus maintenable
  • Moins de points de défaillance
  • Documentation plus claire

  ✅ MCP GLPI v7 enrichi
  ──────────────────────
  Nouvelles opérations:
    • send_email() - envoi emails via GLPI
    • create_glpi_user() - création comptes GLPI
    • get_glpi_user() - recherche utilisateurs
    • update_glpi_user() - mise à jour comptes
    • disable_glpi_user() - désactivation comptes
  
  Usage: Gestion complète users + communications unifiées
  
  ✅ Corrections bugs v4
  ──────────────────────
  • Health Check GLPI v2: Fix Redis get "glpi_down_alert_sent"
  • Proactif v8: Fix Redis get "glpi_health_status"
  • Assist v4: Fix Redis get "glpi_health_status"
  • Mutex ownership: Vérification avant release
```

### 9.3 🚧 En réflexion

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOLUTIONS FUTURES                             │
└─────────────────────────────────────────────────────────────────┘

  🔮 Diagnostic Phibee alternatif
  ────────────────────────────────
  Option envisagée: Script client local
  
  Fonctionnement proposé:
    1. Client installe un petit outil Windows/Linux
    2. En cas de panne réseau, exécute:
       - ping vers gateway Phibee
       - traceroute vers DNS publics
       - test débit local
       - capture log routeur si accessible
    3. Génère rapport JSON
    4. Upload vers API WIDIP ou copie dans ticket
  
  Avantages:
    • Diagnostic réel depuis le site client
    • Pas de dépendance API Phibee
    • Données techniques précises
  
  Inconvénients:
    • Nécessite installation chez client
    • Maintenance de l'outil
    • Formation clients
  
  Statut: Analyse de faisabilité en cours
```

### 9.4 Fichiers du projet v5

```
WIDIP_Architecture_v5/
├── 📁 Workflows/
│   ├── 🔧 WIDIP_Redis_Helper_v2.2.json        (aCuwZ3jJb1c2dMVY)
│   ├── 🏥 WIDIP_Health_Check_GLPI_v2.json     (yNJ1g9IKO2ZSQZ2M) [CORRIGÉ]
│   ├── 🚨 WIDIP_Proactif_Observium_v8.json    (zBYBmEHgXDT2nWHJ) [CORRIGÉ]
│   └── 🎫 WIDIP_Assist_ticket_v4.json         (8muMSPoq5CW8oHGq) [CORRIGÉ]
│
├── 📁 MCP/ (4 outils)
│   ├── 📊 WIDIP_MCP_Observium.json            (QMnaB7g1DpUfN9rK)
│   ├── 🎫 WIDIP_MCP_GLPI_v7.json              (juHWzwO4WgW9LX0B) [ENRICHI]
│   ├── 👤 WIDIP_MCP_ActiveDirectory_v2.json   (2tSBUs69Kn3NYxxT)
│   └── 🔐 WIDIP_MCP_MySecret_v1.json          (geNUXjUljodz4CPG)
│
└── 📁 Documentation/
    └── 📖 WIDIP_ARCHI_Complete_v5.md          (ce fichier)
```

---

## 10. Diagramme d'architecture détaillé

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    ARCHITECTURE COMPLÈTE WIDIP v5                            ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  ┌──────────────────────────────────────────────────────────────────────┐   ║
║  │                        SOURCES EXTERNES                              │   ║
║  └────────────┬─────────────────────┬───────────────────────────────────┘   ║
║               │                     │                                        ║
║  ┌────────────▼────────┐  ┌─────────▼────────┐                              ║
║  │   OBSERVIUM         │  │      GLPI        │                              ║
║  │  (Alertes réseau)   │  │   (Tickets)      │                              ║
║  └────────────┬────────┘  └─────────┬────────┘                              ║
║               │ Webhooks             │ Webhooks                              ║
║               ▼                      ▼                                        ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │                          n8n - VM1                                     │ ║
║  │                                                                        │ ║
║  │  ┌───────────────┐              ┌───────────────┐                     │ ║
║  │  │🏥Health Check │              │🔧 Redis       │                     │ ║
║  │  │   GLPI v2     │◄────────────►│ Helper v2.2   │                     │ ║
║  │  │(yNJ1g9I..)    │   (Monitoring)│(aCuwZ3jJ...)  │                     │ ║
║  │  └───────┬───────┘              └───────┬───────┘                     │ ║
║  │          │                              ▲                              │ ║
║  │          │ Update health                │ Cache, Mutex, Circuit       │ ║
║  │          ▼                              │                              │ ║
║  │  ┌──────────────────────────────────────┴──────────────────┐          │ ║
║  │  │                  Redis DB                                │          │ ║
║  │  │  • glpi_health_status                                    │          │ ║
║  │  │  • ollama_lock (mutex)                                   │          │ ║
║  │  │  • observium_* (cache)                                   │          │ ║
║  │  └──────────────────┬───────────────────────────────────────┘          │ ║
║  │                     ▲                                                  │ ║
║  │  ┌──────────────────┴──────────────────┐                              │ ║
║  │  │                                     │                              │ ║
║  │  ▼                                     ▼                              │ ║
║  │  ┌──────────────────┐       ┌──────────────────┐                     │ ║
║  │  │🚨 PROACTIF v8    │       │🎫 ASSIST v4      │                     │ ║
║  │  │(zBYBmEH...)      │       │(8muMSPoq...)     │                     │ ║
║  │  │                  │       │                  │                     │ ║
║  │  │• Agent Sentinel  │       │• Agent Analyst   │                     │ ║
║  │  │• Notificateur    │       │• Auto-actions    │                     │ ║
║  │  └────────┬─────────┘       └────────┬─────────┘                     │ ║
║  │           │                          │                               │ ║
║  │           └──────────┬───────────────┘                               │ ║
║  │                      ▼                                                │ ║
║  │           ┌────────────────────────────┐                             │ ║
║  │           │   MCP WORKFLOWS (4 outils) │                             │ ║
║  │           │                            │                             │ ║
║  │           │  ┌──────────────────────┐  │                             │ ║
║  │           │  │  📊 MCP Observium    │  │                             │ ║
║  │           │  │  (QMnaB7g1DpUfN9rK) │  │                             │ ║
║  │           │  └──────────────────────┘  │                             │ ║
║  │           │                            │                             │ ║
║  │           │  ┌──────────────────────┐  │                             │ ║
║  │           │  │  🎫 MCP GLPI v7      │  │                             │ ║
║  │           │  │  (juHWzwO4WgW9LX0B) │  │                             │ ║
║  │           │  │  + Email functions   │  │                             │ ║
║  │           │  └──────────────────────┘  │                             │ ║
║  │           │                            │                             │ ║
║  │           │  ┌──────────────────────┐  │                             │ ║
║  │           │  │  👤 MCP AD v2        │  │                             │ ║
║  │           │  │  (2tSBUs69Kn3NYxxT) │  │                             │ ║
║  │           │  └──────────────────────┘  │                             │ ║
║  │           │                            │                             │ ║
║  │           │  ┌──────────────────────┐  │                             │ ║
║  │           │  │  🔐 MCP MySecret v1  │  │                             │ ║
║  │           │  │  (geNUXjUljodz4CPG) │  │                             │ ║
║  │           │  └──────────────────────┘  │                             │ ║
║  │           └────────────┬───────────────┘                             │ ║
║  └────────────────────────┼────────────────────────────────────────────┘ ║
║                           │                                               ║
║                           ▼                                               ║
║  ┌─────────────────────────────────────────────────────────────┐         ║
║  │                    🤖 OLLAMA - VM2                          │         ║
║  │  Modèle: Qwen3-Coder:30b (30B params)                      │         ║
║  │  GPU: NVIDIA L40S (48GB VRAM)                               │         ║
║  │  Performance: 30-50 tokens/sec                              │         ║
║  └─────────────────────────────────────────────────────────────┘         ║
║                           │                                               ║
║              ┌────────────┼────────────┐                                  ║
║              ▼            ▼            ▼                                  ║
║  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                     ║
║  │ Observium API│ │   GLPI API   │ │ Windows AD   │                     ║
║  └──────────────┘ └──────────────┘ └──────────────┘                     ║
║                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

---

## 📝 Glossaire

| Terme | Définition |
|-------|------------|
| **MCP** | Model Context Protocol - standard permettant aux LLM d'utiliser des outils |
| **LLM** | Large Language Model - modèle de langage (Qwen3-Coder, etc.) |
| **Ollama** | Serveur local pour exécuter des LLM |
| **Mutex** | Verrou d'exclusion mutuelle (un seul accès à la fois) |
| **Circuit Breaker** | Pattern de protection contre les pannes en cascade |
| **GLPI** | Gestionnaire Libre de Parc Informatique (ticketing + emails) |
| **Observium** | Plateforme de monitoring réseau |
| **TTL** | Time To Live - durée de vie d'une donnée en cache |
| **SETNX** | SET if Not eXists - opération Redis atomique pour les mutex |
| **Sub-Workflow** | Workflow n8n appelé par un autre workflow |
| **Webhook** | Endpoint HTTP appelé par un système externe |
| **SSE** | Server-Sent Events - protocole de communication MCP |

---

## 📊 Métriques cibles

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          MÉTRIQUES CIBLES v5                                 │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌────────────────────────────┬─────────────────┬─────────────────────────────┐
  │  MÉTRIQUE                  │  CIBLE          │  NOTES                      │
  ├────────────────────────────┼─────────────────┼─────────────────────────────┤
  │  Latence Redis             │  < 5 ms         │  Sub-Workflow               │
  │  Latence Redis             │  < 20 ms        │  Via webhook                │
  ├────────────────────────────┼─────────────────┼─────────────────────────────┤
  │  Temps diagnostic Proactif │  < 45 s         │  Sentinel complet           │
  │  Temps notif Proactif      │  < 15 s         │  Notificateur seul          │
  │  Temps total Proactif      │  < 60 s         │  Alerte → Ticket + Email    │
  ├────────────────────────────┼─────────────────┼─────────────────────────────┤
  │  Temps traitement Assist   │  < 90 s         │  Ticket simple              │
  │  Temps reset password      │  < 30 s         │  AD + MySecret + Email      │
  │  Temps création compte     │  < 60 s         │  AD + GLPI + Email          │
  ├────────────────────────────┼─────────────────┼─────────────────────────────┤
  │  Inférence Qwen3-Coder:30b │  10-20 s        │  GPU L40S (48GB VRAM)       │
  │  Tokens/sec                │  30-50 t/s      │  Génération                 │
  ├────────────────────────────┼─────────────────┼─────────────────────────────┤
  │  Volume supporté           │  ~50 req/jour   │  Sans file d'attente        │
  │  Pic supporté              │  ~150 req/jour  │  Avec attente mutex         │
  └────────────────────────────┴─────────────────┴─────────────────────────────┘
```

---

**Document généré le 9 décembre 2025**  
**Architecture v5 - Simplifiée et optimisée pour production**
