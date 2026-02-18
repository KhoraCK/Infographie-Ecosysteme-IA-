# INDEX DES PREUVES R&D - PROJET WIDIP
## Crédit d'Impôt Recherche (CIR) - Période Nov 2025 - Fév 2026

---

**Date de génération :** 3 février 2026
**Projet :** WIDIP - Assistant IA IT Proactif
**Entreprise :** ACSEA

---

## SOMMAIRE

1. [Phase N8N (Nov-Déc 2025)](#1-phase-n8n-nov-déc-2025)
2. [Pivot Technologique (Déc 2025-Jan 2026)](#2-pivot-technologique-déc-2025-jan-2026)
3. [Phase Python (Jan-Fév 2026)](#3-phase-python-jan-fév-2026)
4. [Journal de Bord R&D](#4-journal-de-bord-rd)

---

## 1. PHASE N8N (Nov-Déc 2025)

### 1.1 Workflows N8N (Fichiers JSON)

Ces fichiers `.json` sont les exports des workflows N8N développés pendant la phase d'exploration initiale.

| Fichier | Taille | Description | Preuve de |
|---------|--------|-------------|-----------|
| `WIDIP_Proactif_Observium_v9.json` | 54 KB | Workflow principal monitoring | Itération v1→v9 |
| `WIDIP_Assist_ticket_v6.1.json` | 55 KB | Assistant tickets GLPI | Itération v1→v6.1 |
| `WIDIP_MCP_GLPI_v7.json` | 55 KB | Intégration MCP GLPI | Itération v1→v7 |
| `WIDIP_MCP_Server.json` | 21 KB | Serveur MCP central | Architecture MCP |
| `WIDIP_MCP_Memory.json` | 9 KB | Gestion mémoire MCP | Capitalisation connaissances |
| `WIDIP_Redis_Helper_v2.2.json` | 19 KB | Helper Redis cache | Optimisation performance |
| `WIDIP_RAG_Ingestor.json` | 6 KB | Ingestion RAG | Système RAG |
| `WIDIP_RAG_Ingest_Clients_v2.json` | 12 KB | Ingestion clients | Itération v1→v2 |
| `WIDIP_RAG_Ingest_Docs_v2.json` | 15 KB | Ingestion documents | Itération v1→v2 |
| `MCP_RAG_v2.json` | 13 KB | Intégration MCP-RAG | Itération v1→v2 |

**Total : 10 workflows / 259 KB**

**Ce que ces fichiers prouvent :**
- Développement itératif (numéros de version v1→vX)
- Complexité technique (taille des workflows)
- Exploration de multiples approches (MCP, RAG, Redis)

---

### 1.2 Documentation Architecture (Versions v1-v7)

Ces fichiers documentent l'évolution de l'architecture pendant la phase N8N.

| Fichier | Taille | Date originale | Évolution clé |
|---------|--------|----------------|---------------|
| `WIDIP_ARCHI_Complete_v1.md` | 124 KB | 30 Nov 01:40 | Architecture initiale |
| `WIDIP_ARCHI_Complete_v2.md` | 87 KB | 30 Nov 21:32 | Refactoring (-37 KB) |
| `WIDIP_ARCHI_Complete_v3.md` | 118 KB | 1 Déc 04:34 | Extension (+31 KB) |
| `WIDIP_ARCHI_Complete_v4.md` | 68 KB | 2 Déc 21:47 | Simplification |
| `WIDIP_ARCHI_Complete_v5.md` | 63 KB | 8 Déc 18:08 | Optimisation |
| `WIDIP_ARCHI_Complete_v6.md` | 77 KB | 8 Déc 23:23 | Extension RAG |
| `WIDIP_ARCHI_Complete_v7.md` | 101 KB | 12 Déc 15:33 | **Version finale N8N** |

**Total : 7 versions / 639 KB**

**Ce que ces fichiers prouvent :**
- **Incertitude technique** : 7 versions en 13 jours = pivots fréquents
- **Démarche expérimentale** : Tailles fluctuantes (124→87→118→68 KB)
- **Systématisation** : Documentation rigoureuse à chaque évolution

---

## 2. PIVOT TECHNOLOGIQUE (Déc 2025-Jan 2026)

### 2.1 Document de Preuve du Pivot

| Fichier | Taille | Description |
|---------|--------|-------------|
| `PREUVE_PIVOT_N8N_PYTHON.md` | 7 KB | Justification complète du pivot |

**Contenu du document :**
- Contexte et objectif initial
- Limites découvertes de N8N (avec exemples concrets)
- Analyse comparative N8N vs Python
- Timeline avec preuves matérielles
- Valeur R&D de l'échec

**Ce que ce fichier prouve :**
- **Incertitude technique caractérisée** : On ne savait pas d'avance que N8N serait insuffisant
- **Démarche scientifique** : Hypothèse → Test → Conclusion → Nouvelle hypothèse
- **Créativité** : Capacité à pivoter face à l'échec

---

### 2.2 Chronologie du Pivot

```
12 Déc 2025 : Dernière version N8N (v7)
     ↓
   [30 jours de réflexion/analyse]
     ↓
12 Jan 2026 : Début développement Python
14 Jan 2026 : WIBOT v1 (première version Python)
```

**Gap de 30 jours = Preuve de décision réfléchie, pas d'abandon impulsif**

---

## 3. PHASE PYTHON (Jan-Fév 2026)

### 3.1 Évolution WIBOT

| Dossier | Date | Fichiers clés | Évolution |
|---------|------|---------------|-----------|
| `WIBOT_v1_14Jan/` | 14 Jan 2026 | README.md, docker-compose.yml | Version initiale Python |
| `WIBOT_v2_20Jan/` | 20 Jan 2026 | *(backup vide - voir ARCHI)* | Migration vers WIDIP_ARCHI |

---

### 3.2 Évolution WIDIP_ARCHI (Versions sélectionnées)

| Dossier | Date | Taille CLAUDE_CONTEXT | Évolution majeure |
|---------|------|----------------------|-------------------|
| `WIDIP_ARCHI_v1_26Jan/` | 26 Jan | 12 KB | Première architecture Python |
| `WIDIP_ARCHI_v4_29Jan/` | 29 Jan | 16 KB | +code_mode (+33%) |
| `WIDIP_ARCHI_v9_03Fev/` | 3 Fév 09:34 | 30 KB | +SMS alerts (+88%) |
| `WIDIP_ARCHI_v10_03Fev/` | 3 Fév 10:42 | 31 KB | Version courante (+3%) |

**Progression CLAUDE_CONTEXT.md :** 12 KB → 31 KB = **+158% en 8 jours**

**Ce que ces fichiers prouvent :**
- **Développement rapide** : 10 versions en 8 jours
- **Complexité croissante** : Taille multipliée par 2.6
- **Innovations successives** : code_mode, SMS alerts, SAFEGUARD

---

## 4. JOURNAL DE BORD R&D

| Fichier | Taille | Description |
|---------|--------|-------------|
| `JOURNAL_DE_BORD_RD.md` | 17 KB | Journal complet Nov 2025 - Fév 2026 |

**Contenu :**
- Progression semaine par semaine
- Hypothèses testées et résultats
- Échecs documentés (5 échecs majeurs)
- Versions et jalons

---

## SYNTHÈSE DES PREUVES

### Volume Total

| Catégorie | Fichiers | Volume |
|-----------|----------|--------|
| Workflows N8N | 10 | 259 KB |
| Documentation N8N v1-v7 | 7 | 639 KB |
| Preuve Pivot | 1 | 7 KB |
| Versions Python | 5 | 69 KB |
| Journal de Bord | 1 | 17 KB |
| **TOTAL** | **24 fichiers** | **991 KB** |

### Critères Frascati Couverts

| Critère | Preuve | Fichiers |
|---------|--------|----------|
| **Nouveauté** | Pas de solution équivalente sur le marché | JOURNAL_DE_BORD_RD.md |
| **Créativité** | 7 versions N8N + pivot Python | v1-v7.md, PREUVE_PIVOT |
| **Incertitude** | Échec N8N, 5 échecs documentés | PREUVE_PIVOT, JOURNAL |
| **Systématisation** | Documentation versionnée | Tous les fichiers |
| **Transférabilité** | Architecture documentée | CLAUDE_CONTEXT.md |

---

## SOURCES ORIGINALES

Les fichiers de ce dossier sont des copies des originaux situés dans :

| Phase | Chemin source |
|-------|---------------|
| N8N | `C:\Users\maxime\Documents\WIDIP\WIDIP N8N\` |
| Python | `C:\Users\maxime\Documents\WIDIP\10_BACKUP\` |
| Courant | `C:\Users\maxime\Documents\WIDIP\WIDIP_ARCHI\` |

**Les dates de modification des fichiers sources constituent une preuve d'antériorité.**

---

## RECOMMANDATIONS POUR LE CONTRÔLE CIR

1. **Vérifier les dates** : Les métadonnées des fichiers originaux dans `10_BACKUP` prouvent la chronologie
2. **Comparer les versions** : Diff entre v1 et v7 montre l'évolution réelle
3. **Analyser le pivot** : Gap de 30 jours + documentation prouve la réflexion
4. **Croissance CLAUDE_CONTEXT** : 12 KB → 31 KB prouve la complexification

---

*Document généré automatiquement le 3 février 2026*
*Projet WIDIP - ACSEA*
