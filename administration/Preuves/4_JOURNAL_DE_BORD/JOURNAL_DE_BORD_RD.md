# JOURNAL DE BORD R&D - PROJET WIDIP

## Informations du projet

| Champ | Valeur |
|-------|--------|
| **Projet** | WIDIP - Plateforme IA Multi-agents pour Support IT |
| **Periode** | Novembre 2025 - Fevrier 2026 |
| **Responsable R&D** | Maxime |
| **Document** | Journal de bord pour eligibilite CIR |
| **Version** | 1.0 - Genere le 3 fevrier 2026 |

---

## Resume executif

Ce journal retrace l'evolution du projet WIDIP depuis sa conception initiale en novembre 2025 jusqu'a fevrier 2026. Il documente :
- Les **hypotheses testees** et leurs resultats (succes/echecs)
- Les **verrous techniques** rencontres
- Le **pivot technologique** N8N → Python
- Les **preuves d'incertitude scientifique**

---

## Chronologie globale

```
NOVEMBRE 2025
└── 27 Nov : Demarrage phase N8N (prototype)

DECEMBRE 2025
├── 27 Nov - 12 Dec : Developpement iteratif N8N (7 versions)
├── 12 Dec : N8N v7 "Production Ready" - Memoire RAG Active
└── Mi-Dec : Identification des limites de N8N

JANVIER 2026
├── ~12 Jan : Decision de pivot vers Python
├── 12-14 Jan : Infrastructure backend Python de base
├── 14 Jan : WIBOT v1.0.0
├── 20 Jan : WIBOT v2.0.0 + RAG v1
├── 26-27 Jan : Debut WIDIP_ARCHI (refactoring complet)
├── 28-29 Jan : Integration SMS L4_SMS + SAFEGUARD complet

FEVRIER 2026
├── 2 Feb : WIDIP_ARCHI v5-v8 (corrections majeures)
├── 3 Feb : WIDIP_ARCHI v9-v10 (stabilisation)
└── 3 Feb : Version courante en production
```

---

# PHASE 1 : PROTOTYPE N8N

## Semaine 1 : 27 Novembre - 3 Decembre 2025

### Objectif de la semaine
Valider le concept d'un systeme multi-agents IA pour le support IT en utilisant N8N comme plateforme d'orchestration low-code.

### Travaux realises

| Date | Action | Fichiers/Preuves |
|------|--------|------------------|
| 27 Nov | Creation architecture initiale v1 | `WIDIP_ARCHI_Complete_v1.md` |
| 30 Nov | Amelioration integrations MCP v2 | `WIDIP_ARCHI_Complete_v2.md` |
| 1-2 Dec | Optimisations workflows v3-v4 | `WIDIP_ARCHI_Complete_v3.md`, `v4.md` |
| 3 Dec | Premier workflow support mature | `WIDIP_Assist_ticket_v6.json` (31→53 Ko) |

### Architecture initiale (v1)
- **Hub MCP central** : Router vers tous les services
- **3 workflows principaux** :
  - WIBOT (interface conversationnelle)
  - Proactif Observium (surveillance reseau)
  - Assistant Ticket (gestion tickets)

### Hypotheses testees

| # | Hypothese | Resultat | Apprentissage |
|---|-----------|----------|---------------|
| 1 | N8N peut orchestrer des agents IA simples | ✅ VALIDE | Fonctionne pour cas basiques |
| 2 | Les webhooks N8N suffisent pour la communication inter-agents | ⚠️ PARTIEL | Limites de performance identifiees |

### Verrous identifies
- Gestion de l'etat complexe entre workflows
- Pas de mecanisme natif de validation humaine

---

## Semaine 2 : 4 - 10 Decembre 2025

### Objectif de la semaine
Implementer le systeme RAG (Retrieval Augmented Generation) et stabiliser l'architecture.

### Travaux realises

| Date | Action | Fichiers/Preuves |
|------|--------|------------------|
| 8-9 Dec | Documentation architecture v5-v6 | `WIDIP_ARCHI_Complete_v5.md`, `v6.md` |
| 9-10 Dec | Finalisation RAG v2 | Dossier `n8n_workflows_rag_v2/` |
| 10 Dec | Infrastructure GPU + Diagnostic | `GPU_Cheat_Sheet.md`, `DiagnosticWIDIP/` |

### Workflows RAG implementes (5 workflows)

| Workflow | Cron | Fonction |
|----------|------|----------|
| `WIDIP_RAG_Init_Infrastructure` | Manuel 1x | Initialise Qdrant + Neo4j |
| `MCP_RAG_v2` | Toujours actif | Expose outils RAG via MCP |
| `WIDIP_RAG_Ingest_Clients` | 02:00 quotidien | Vectorise clients GLPI |
| `WIDIP_RAG_Ingest_Docs` | Dimanche 03:00 | Vectorise docs support |
| `WIDIP_RAG_Ingest_Tickets` | 04:00 quotidien | Vectorise tickets fermes |

### Stack technique N8N finale
- **Orchestration** : n8n 1.20+
- **LLM** : Ollama + qwen2.5-coder:32b
- **Vector DB** : PostgreSQL + pgvector
- **Embeddings** : nomic-embed-text (768 dim)
- **Cache** : Redis

### Problemes rencontres
1. **Complexite d'orchestration** : Les workflows N8N deviennent difficiles a maintenir
2. **Pas de framework de securite** : Impossible d'implementer SAFEGUARD proprement
3. **Debugging difficile** : Les erreurs sont complexes a tracer

---

## Semaine 3 : 11 - 12 Decembre 2025

### Objectif de la semaine
Finaliser la version "Production Ready" avec memoire RAG active.

### Travaux realises

| Date | Action | Fichiers/Preuves |
|------|--------|------------------|
| 12 Dec 00:22 | Redis Helper v2.2 finalise | `WIDIP_Redis_Helper_v2.2.json` |
| 12 Dec 00:33 | Dashboard Backend v1 | `WIDIP_Dashboard_Backend_v1.json` |
| 12 Dec 02:20 | MCP Server v1 | `WIDIP_MCP_Server_v1.json` |
| 12 Dec 02:28 | Agents IA v8-v9 | `WIDIP_Proactif_Observium_v9.json` |
| 12 Dec 15:33 | **Documentation v7 complete** | `WIDIP_ARCHI_Complete_v7.md` (100 Ko) |

### Resultats de la phase N8N

**Metriques finales :**
- 800+ etablissements clients supportes
- 20k tickets/an traites
- Architecture bi-VM (Orchestrator + GPU)

**Innovations de la v7 :**

| Feature | Avant (v6) | Apres (v7) |
|---------|-----------|-----------|
| Contexte | Aucun historique | Consultation precedents similaires |
| Apprentissage | Aucun | Continu (chaque ticket enrichit la base) |
| Infrastructure | Single-VM | Bi-VM (Orch + GPU) |
| RAG | N/A | Memoire vectorielle active |

### DECISION CRITIQUE : Limites de N8N identifiees

**Problemes bloquants :**

1. **Orchestration multi-agents complexe** :
   - Les workflows N8N ne permettent pas une gestion fine de l'etat
   - Les callbacks asynchrones sont difficiles a gerer
   - Le debugging devient cauchemardesque

2. **SAFEGUARD impossible a implementer** :
   - Pas de mecanisme de validation humaine inline
   - Impossible de bloquer/reprendre un workflow proprement
   - Pas d'audit trail natif

3. **Maintenance non viable** :
   - Chaque workflow devient un fichier JSON de 50+ Ko
   - Modifications = risque de casser l'ensemble
   - Pas de tests unitaires possibles

**CONCLUSION** : Decision de pivoter vers une architecture Python modulaire.

---

# PHASE 2 : PIVOT TECHNOLOGIQUE N8N → PYTHON

## Semaine 4 : ~12 - 20 Janvier 2026

### Objectif de la semaine
Reconstruire l'architecture en Python avec FastAPI pour resoudre les limites de N8N.

### Justification du pivot

| Critere | N8N | Python/FastAPI |
|---------|-----|----------------|
| Orchestration complexe | ❌ Difficile | ✅ Natif |
| SAFEGUARD | ❌ Impossible | ✅ Implementable |
| Debugging | ❌ Complexe | ✅ Standard |
| Tests | ❌ Aucun | ✅ pytest |
| Maintenance | ❌ JSON monolithique | ✅ Code modulaire |

### Travaux realises

| Date | Action | Fichiers/Preuves |
|------|--------|------------------|
| 12 Jan | Services backend fondamentaux | `token_service.py`, `connector_service.py` |
| 13 Jan | Services IA | `ai_service.py`, `agent_service.py` |
| 14 Jan | **WIBOT v1.0.0 complete** | Dossier `01_WIBOT_v1/` archive |
| 20 Jan | RAG service v1 | `rag_service.py` |
| 20 Jan | Init workflow Observium | `workflow-observium/` |
| 20 Jan | **WIBOT v2.0.0** | Dossier `01_WIBOT/` actif |

### Nouvelle stack technique

```
Frontend:  React 19 + TypeScript + Vite 7 + TailwindCSS + Zustand
Backend:   FastAPI + PostgreSQL 14 + pgvector + Redis 7
LLM:       Mistral AI (API) au lieu d'Ollama local
MCP:       Serveur MCP Python custom
```

### Hypotheses testees

| # | Hypothese | Resultat | Apprentissage |
|---|-----------|----------|---------------|
| 3 | FastAPI peut gerer le streaming SSE pour les agents | ✅ VALIDE | Bien plus fluide que N8N |
| 4 | PostgreSQL + pgvector suffisent pour le RAG | ✅ VALIDE | Performance acceptable |
| 5 | Mistral API vs Ollama local | ✅ VALIDE | Plus simple, meilleure qualite |

### Preuves du pivot
- **Dossier N8N conserve** : `C:\Users\maxime\Documents\WIDIP\WIDIP N8N\`
- **Premiere version Python** : `10_BACKUP\01_WIBOT_v1\` (14 janvier)
- **Gap temporel** : 12 dec (N8N v7) → 12 jan (Python v1) = reflexion/decision

---

## Semaine 5 : 21 - 27 Janvier 2026

### Objectif de la semaine
Implementer le systeme SAFEGUARD et les outils MCP avances.

### Travaux realises

| Date | Action | Fichiers/Preuves |
|------|--------|------------------|
| 26 Jan | Code Mode + Observium API | `code_mode_tools.py`, `observium_api.py` |
| 26 Jan | Modeles workflow, alert, config | `models/workflow.py`, `alert.py` |
| 26 Jan | **WIDIP_ARCHI v1** | `10_BACKUP\WIDIP_ARCHI_v1\` |
| 27 Jan | Memory tools v2 + templates | `memory_tools_v2.py` |
| 27 Jan | RAG service v2 (embeddings Mistral) | `rag_service_v2.py` |
| 27 Jan | WIDIP_ARCHI v2 et v3 | Archives dans `10_BACKUP\` |

### SAFEGUARD : Premiere implementation (4 niveaux)

**Modele initial L0-L3 :**

| Niveau | Nom | Comportement |
|--------|-----|--------------|
| L0 | READ_ONLY | Execution automatique |
| L1 | MINOR | Auto si confiance ≥ 80% |
| L2 | MODERATE | Notification technicien |
| L3 | SENSITIVE | Validation humaine obligatoire |

### Hypotheses testees

| # | Hypothese | Resultat | Apprentissage |
|---|-----------|----------|---------------|
| 6 | 4 niveaux de securite suffisent | ⚠️ INSUFFISANT | Besoin d'un niveau "differe" |
| 7 | Interface d'approbation inline SSE | ✅ VALIDE | Fonctionne bien |

---

## Semaine 6 : 28 - 31 Janvier 2026

### Objectif de la semaine
Integrer le systeme d'alertes SMS et completer SAFEGUARD.

### Travaux realises

| Date | Action | Fichiers/Preuves |
|------|--------|------------------|
| 28 Jan | Client AllMySMS | `clients/allmysms.py` |
| 28 Jan | Cache Redis referents | `clients/referent_cache.py` |
| 28 Jan | **SMS alert tools L4_SMS** | `sms_alert_tools.py` (814 lignes) |
| 29 Jan | MCP server avec check SAFEGUARD | `mcp/server.py` |
| 29 Jan | WIDIP_ARCHI v4 | `10_BACKUP\WIDIP_ARCHI_v4\` |

### SAFEGUARD : Ajout niveaux L4 et L4_SMS

**Probleme identifie** : Les alertes SMS client ne rentrent dans aucune categorie existante :
- L3 (validation simple) : Pas assez, besoin de choisir le contact
- L4 (bloque) : Trop restrictif, l'alerte doit pouvoir partir

**Solution** : Creation du niveau L4_SMS specialise

| Niveau | Nom | Comportement |
|--------|-----|--------------|
| L4 | DEFERRED | Execution differee avec delai d'annulation |
| L4_SMS | SMS_ALERT | Interface specialisee avec selection contact referent |

### Hypotheses testees

| # | Hypothese | Resultat | Apprentissage |
|---|-----------|----------|---------------|
| 8 | Niveau L4 "FORBIDDEN" pour actions critiques | ⚠️ AJUSTE | Renomme en "DEFERRED" - actions differees, pas interdites |
| 9 | Score d'activite GLPI pour recommander le contact | ✅ VALIDE | Bonne correlation |
| 10 | Cache Redis TTL 7j/14j selon confirmation | ✅ VALIDE | Equilibre fraicheur/stabilite |

---

# PHASE 3 : STABILISATION ET PRODUCTION

## Semaine 7 : 1 - 3 Fevrier 2026

### Objectif de la semaine
Corriger les bugs critiques et stabiliser pour la production.

### Travaux realises

| Date | Heure | Action | Fichiers/Preuves |
|------|-------|--------|------------------|
| 2 Feb | 09:59 | SMS alert tools + Annuaire | `sms_alert_tools.py`, `annuaire_tools.py` |
| 2 Feb | 10:40 | MCP client mise a jour | `mcp_client.py` |
| 2 Feb | 10:59 | **Safeguard service refactorise** | `safeguard_service.py` (41 Ko) |
| 2 Feb | 11:20 | Agent streaming service | `agent_streaming_service.py` |
| 2 Feb | 13:01 | Alert service + MCP client workflow | `alert_service.py` |
| 2 Feb | 13:09 | **Config SAFEGUARD complete (6 niveaux)** | `config.py` (410 lignes) |
| 2 Feb | 15:49 | Device down tracker | `device_down_tracker.py` |
| 2 Feb | - | WIDIP_ARCHI v5 a v8 | Archives `10_BACKUP\` |

### BUG CRITIQUE : Double blocage SAFEGUARD

**Symptome** : Apres approbation humaine, l'action est a nouveau bloquee.

**Analyse** :
```
1. Agent demande action L3 → Backend cree demande → Frontend affiche
2. Humain approuve → Backend met a jour status "approved"
3. Backend rappelle MCP Server pour executer
4. MCP Server re-verifie SAFEGUARD → BLOQUE (ne sait pas que c'est approuve)
```

**Cause racine** : Deux systemes independants avec leurs propres regles :
- Backend SAFEGUARD (PostgreSQL)
- MCP Server SAFEGUARD (Python config)

### Hypotheses testees pour resoudre le bug

| # | Hypothese | Resultat | Details |
|---|-----------|----------|---------|
| 11 | Synchronisation via polling DB partagee | ❌ ECHEC | Latence 500ms+, race conditions |
| 12 | Mecanisme `_bypass_safeguard` avec token | ✅ RESOLU | Parametre transmis apres approbation |

**Solution implementee** :
```python
# Apres approbation, le backend rappelle MCP avec :
{
    "tool": "glpi_close_ticket",
    "params": {
        "ticket_id": 12345,
        "_bypass_safeguard": True,  # ← Indique que c'est approuve
        "_approval_id": "abc-123"    # ← Pour audit trail
    }
}
```

### Corrections de logique metier

**Probleme** : Tickets crees immediatement a chaque alerte Observium
**Solution** : Tickets crees uniquement apres seuil de 20 minutes DOWN

| Avant | Apres |
|-------|-------|
| Alerte DOWN → Ticket immediat | Alerte DOWN → Timer 20min → Si toujours DOWN → Ticket |

---

## 3 Fevrier 2026 (aujourd'hui)

### Travaux realises

| Heure | Action | Fichiers/Preuves |
|-------|--------|------------------|
| 08:55 | Timeline modeles | `models/timeline.py` |
| 08:58 | Timeline service SSE | `services/timeline_service.py` |
| 09:34 | WIDIP_ARCHI v9 | Archive |
| 09:51 | **Workflow engine complet** | `services/workflow_engine.py` |
| 09:56 | **Version courante WIDIP_ARCHI** | Production |
| 10:42 | WIDIP_ARCHI v10 archivee | Derniere backup |
| 10:53 | Auth service + LDAP | `auth_service.py` |

### Etat actuel des Operations de Recherche

| OR | Composant | Statut | Metriques |
|----|-----------|--------|-----------|
| OR1 | Enrichisseur + RAG | EN COURS | Taux validation N8N ~40% |
| OR2 | Assistant Ticket + Sentinel | EN COURS | Precision 65%→75%, Detection 90% |
| OR3 | SAFEGUARD | **QUASI FINALISE** | 200 demandes, 92% approbation, 0 incident |

---

# SYNTHESE DES PREUVES D'INCERTITUDE

## Echecs documentes (preuves CIR)

| # | Echec | Date | Impact | Preuve |
|---|-------|------|--------|--------|
| 1 | Limites N8N pour orchestration complexe | Dec 2025 | Pivot technologique | Dossier `WIDIP N8N/` conserve |
| 2 | SAFEGUARD impossible en N8N | Dec 2025 | Pivot technologique | Documentation v7 mentionne l'absence |
| 3 | Auto-evaluation confiance LLM | Dec 2025 | Scores >85% non discriminants | Logs a extraire |
| 4 | Synchronisation polling DB | Jan 2026 | Race conditions | Code `safeguard_service.py` v1 |
| 5 | 4 niveaux SAFEGUARD insuffisants | Jan 2026 | Ajout L4 + L4_SMS | Historique versions |

## Hypotheses validees

| # | Hypothese | Date | Metriques |
|---|-----------|------|-----------|
| 1 | FastAPI + SSE pour streaming agents | Jan 2026 | Latence <100ms |
| 2 | Mistral Embed pour RAG | Jan 2026 | Precision retrieval >80% |
| 3 | 6 niveaux SAFEGUARD | Jan 2026 | 65% actions auto, 35% validation |
| 4 | Bypass post-approbation | Fev 2026 | 0 double-blocage depuis |
| 5 | Cache referent TTL variable | Fev 2026 | Precision 60%→78% |

---

# LISTE DES VERSIONS ARCHIVEES

## Phase N8N

| Version | Date | Contenu principal |
|---------|------|-------------------|
| v1 | 27 Nov 2025 | Architecture initiale |
| v2 | 30 Nov 2025 | Integrations MCP |
| v3 | 1 Dec 2025 | Optimisations |
| v4 | 2 Dec 2025 | Debugging |
| v5 | 8 Dec 2025 | Documentation |
| v6 | 9 Dec 2025 | Stabilisation |
| v7 | 12 Dec 2025 | **Production Ready - Memoire RAG** |

## Phase Python

| Version | Date | Contenu principal |
|---------|------|-------------------|
| WIBOT v1 | 14 Jan 2026 | Premiere version Python |
| WIBOT v2 | 20 Jan 2026 | Version amelioree |
| WIDIP_ARCHI v1 | 26 Jan 2026 | Debut refactoring |
| WIDIP_ARCHI v2 | 27 Jan 2026 | RAG v2 |
| WIDIP_ARCHI v3 | 27 Jan 2026 | Memory tools |
| WIDIP_ARCHI v4 | 29 Jan 2026 | SAFEGUARD complet |
| WIDIP_ARCHI v5 | 2 Feb 2026 | Corrections majeures |
| WIDIP_ARCHI v6 | 2 Feb 2026 | Agent streaming |
| WIDIP_ARCHI v7 | 2 Feb 2026 | Config 6 niveaux |
| WIDIP_ARCHI v8 | 2 Feb 2026 | Device tracker |
| WIDIP_ARCHI v9 | 3 Feb 2026 | Timeline |
| WIDIP_ARCHI v10 | 3 Feb 2026 | Workflow engine |
| **Courante** | 3 Feb 2026 | **Production** |

---

# PROCHAINES ETAPES R&D

## Court terme (Fevrier 2026)

- [ ] Finaliser OR1 : Enrichisseur automatique de procedures
- [ ] Finaliser OR2 : Calibration confiance basee sur similarite RAG
- [ ] Collecter metriques detaillees OR1 et OR2
- [ ] Documenter l'etat de l'art formel pour chaque OR

## Moyen terme (Mars 2026)

- [ ] Produire matrice de confusion complete OR2
- [ ] Courbe de calibration confiance
- [ ] Benchmark RAG avant/apres enrichissement
- [ ] Tests de regression SAFEGUARD

---

*Document genere le 3 fevrier 2026*
*Ce journal constitue une preuve de la demarche R&D pour l'eligibilite au Credit d'Impot Recherche*
