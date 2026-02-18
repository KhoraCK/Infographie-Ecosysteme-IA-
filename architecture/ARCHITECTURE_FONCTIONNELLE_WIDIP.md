# Architecture Fonctionnelle - Plateforme IA Support WIDIP

## Vue d'ensemble du Systeme

Cette architecture est une plateforme d'intelligence artificielle concue pour le support informatique de **WIDIP** (l'entreprise). Elle permet aux techniciens WIDIP de gerer les tickets, surveiller les infrastructures clientes et capitaliser sur les connaissances de l'entreprise - le tout avec une securite renforcee garantissant qu'aucune action critique n'est executee sans validation humaine.

> **Note** : WIDIP est l'entreprise pour laquelle cette architecture a ete concue. La plateforme elle-meme (WIBOT + Workflows) n'a pas de nom specifique a ce jour.

---

## Schema Global de l'Architecture

```
                                    UTILISATEURS
                                         |
                                         v
+====================================================================================+
|                                                                                    |
|                              WIBOT (Interface Utilisateur)                         |
|                                                                                    |
|    +----------+    +----------+    +----------+    +----------+                    |
|    |  FLASH   |    |   PRO    |    |   CODE   |    |  AGENT   |                    |
|    | Questions|    | Requetes |    | Scripts  |    |Resolution|                    |
|    | Rapides  |    | Complexes|    | & Code   |    | Problemes|                    |
|    +----------+    +----------+    +----------+    +----+-----+                    |
|                                                        |                           |
|                                                        v                           |
|                                              +------------------+                   |
|                                              | Appel aux outils |                   |
|                                              | RAG, MCP GLPI,   |                   |
|                                              | MCP Observium    |                   |
|                                              +------------------+                   |
|                                                                                    |
+====================================================================================+
                                         |
             +---------------------------+---------------------------+
             |                           |                           |
             v                           v                           v
+========================+  +========================+  +========================+
|                        |  |                        |  |                        |
|       SENTINEL         |  |   ASSISTANT TICKET     |  |     ENRICHISSEUR       |
|  (Surveillance Reseau) |  |  (Gestion Tickets)     |  | (Capitalisation)       |
|                        |  |                        |  |                        |
|  - Detection pannes    |  |  - Qualification       |  |  - Analyse tickets     |
|  - Identification      |  |  - Resolution auto     |  |  - Creation procedures |
|    clients impactes    |  |  - Escalade si besoin  |  |  - Enrichissement RAG  |
|  - Notification techs  |  |                        |  |                        |
|  - Alerte clients SMS  |  |                        |  |                        |
|                        |  |                        |  |                        |
+========================+  +========================+  +========================+
             |                           |                           |
             +---------------------------+---------------------------+
                                         |
                                         v
+====================================================================================+
|                                                                                    |
|                              COUCHE MCP (Outils)                                   |
|                                                                                    |
|    +---------------+    +---------------+    +---------------+                     |
|    |   MCP GLPI    |    |MCP OBSERVIUM  |    |    MCP AD     |                     |
|    |   Ticketing   |    |  Monitoring   |    |Active Directory                     |
|    +---------------+    +---------------+    +---------------+                     |
|                                                                                    |
|    +---------------+    +---------------+    +---------------+                     |
|    |   MCP SMS     |    |  MCP EMAIL    |    |  MCP TEAMS    |                     |
|    | Notifications |    | Notifications |    | Notifications |                     |
|    +---------------+    +---------------+    +---------------+                     |
|                                                                                    |
+====================================================================================+
                                         |
                                         v
+====================================================================================+
|                                                                                    |
|                              SAFEGUARD (Securite)                                  |
|                                                                                    |
|    Controle de TOUTES les actions critiques avant execution                        |
|    Validation humaine obligatoire pour les niveaux L2, L3, L4                      |
|                                                                                    |
+====================================================================================+
                                         |
                                         v
+====================================================================================+
|                                                                                    |
|                         BASE DE CONNAISSANCES (RAG)                                |
|                                                                                    |
|    +---------------+    +---------------+    +---------------+                     |
|    |  Procedures   |    |   Tickets     |    | Documentation |                     |
|    |  Techniques   |    |   Resolus     |    |   Interne     |                     |
|    +---------------+    +---------------+    +---------------+                     |
|                                                                                    |
+====================================================================================+
                                         |
                                         v
                    +----------------------------------------+
                    |           SYSTEMES EXTERNES            |
                    |                                        |
                    |   GLPI    |   Observium   |    AD      |
                    +----------------------------------------+
```

---

## 1. WIBOT - Interface Utilisateur Intelligente

### Description

WIBOT est l'interface conversationnelle principale de la plateforme. C'est le point d'entree pour tous les techniciens de WIDIP. Il permet d'interagir avec l'ensemble du systeme via une conversation naturelle.

### Les 4 Modes de Chat

#### Mode FLASH
| Caracteristique | Description |
|-----------------|-------------|
| **Objectif** | Reponses rapides aux questions simples |
| **Modele LLM** | Devstral Small (rapide, economique) |
| **Cas d'usage** | "C'est quoi le VLAN pour le site X ?", "Quel est le contact du client Y ?" |
| **Acces RAG** | Oui, pour enrichir les reponses |
| **Acces MCP** | Non |

#### Mode PRO
| Caracteristique | Description |
|-----------------|-------------|
| **Objectif** | Requetes complexes necessitant une analyse approfondie |
| **Modele LLM** | Mistral Large (haute qualite) |
| **Cas d'usage** | "Explique-moi l'architecture reseau du client Z", "Quelle est la procedure complete pour migrer un serveur ?" |
| **Acces RAG** | Oui, recherche approfondie dans la base de connaissances |
| **Acces MCP** | Non |

#### Mode CODE
| Caracteristique | Description |
|-----------------|-------------|
| **Objectif** | Conception et optimisation de scripts |
| **Modele LLM** | Devstral (specialise code) |
| **Cas d'usage** | "Ecris un script PowerShell pour lister les users AD inactifs", "Optimise ce script Python" |
| **Acces RAG** | Oui, pour les exemples et bonnes pratiques |
| **Acces MCP** | Non |

#### Mode AGENT
| Caracteristique | Description |
|-----------------|-------------|
| **Objectif** | Resolution de problemes complexes necessitant des actions |
| **Modele LLM** | Mistral Large (raisonnement avance) |
| **Cas d'usage** | "Diagnostique pourquoi le client X a des lenteurs", "Trouve tous les tickets ouverts pour le site Y et resume la situation" |
| **Acces RAG** | Oui, comme outil de recherche |
| **Acces MCP** | Oui - GLPI, Observium, AD, etc. |

### Fonctionnement du Mode Agent

```
Technicien pose une question complexe
            |
            v
+------------------------+
| Analyse de la demande  |
| par le LLM             |
+------------------------+
            |
            v
+------------------------+
| Determination des      |
| outils necessaires     |
+------------------------+
            |
            +------------------+------------------+
            |                  |                  |
            v                  v                  v
      +----------+       +----------+       +----------+
      |   RAG    |       |MCP GLPI  |       |   MCP    |
      | Recherche|       | Tickets  |       |Observium |
      |  docs    |       | Clients  |       | Metriques|
      +----------+       +----------+       +----------+
            |                  |                  |
            +------------------+------------------+
                               |
                               v
                  +------------------------+
                  | Synthese et analyse    |
                  | des informations       |
                  +------------------------+
                               |
                               v
                  +------------------------+
                  | Suggestion de          |
                  | procedure/solution     |
                  +------------------------+
                               |
                               v
                       Reponse au technicien
```

### Capacites de l'Agent

1. **Consultation multi-sources** : L'agent peut simultanement interroger :
   - La base de connaissances RAG (procedures, documentation)
   - GLPI (tickets, clients, historique)
   - Observium (metriques, alertes, etat des equipements)

2. **Analyse contextuelle** : Croise les informations pour fournir une vision globale

3. **Suggestion de procedures** : Propose des procedures adaptees au probleme detecte

4. **Traçabilite** : Toutes les actions sont loguees pour audit

---

## 2. SENTINEL - Workflow de Surveillance Proactive

### Description

Sentinel est un workflow autonome qui surveille en continu les infrastructures clientes via Observium. Son objectif est de detecter les pannes AVANT que les clients n'appellent, permettant une gestion proactive du support.

### Cycle de Fonctionnement

```
    +------------------------------------------+
    |        BOUCLE CONTINUE (1 min)           |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 1. Interrogation Observium               |
    |    (MCP Observium)                       |
    |    -> Liste des equipements DOWN         |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 2. Verification Cache Redis              |
    |    -> Cet equipement etait-il deja       |
    |       DOWN il y a 20+ minutes ?          |
    +------------------------------------------+
                       |
           +-----------+-----------+
           |                       |
           v                       v
    +-------------+         +-------------+
    | NON         |         | OUI         |
    | Nouvelle    |         | Panne       |
    | panne       |         | persistante |
    +-------------+         +-------------+
           |                       |
           v                       v
    +-------------+         +------------------------------------------+
    | Enregistrer |         | 3. Identification des clients impactes   |
    | dans Redis  |         |    (MCP GLPI)                            |
    | avec timer  |         |    -> Qui est sur cet equipement ?       |
    +-------------+         +------------------------------------------+
                                   |
                                   v
                           +------------------------------------------+
                           | 4. Notification Technicien               |
                           |    (Safeguard - Demande de verification) |
                           |    "Infra X down depuis 20min,           |
                           |     clients Y et Z impactes.             |
                           |     Verifiez si FAI ou WIDIP"            |
                           +------------------------------------------+
                                   |
                                   v
                           +------------------------------------------+
                           | 5. Technicien verifie et repond          |
                           |    via Safeguard :                       |
                           |    [ ] Panne FAI                         |
                           |    [ ] Panne infra WIDIP                       |
                           +------------------------------------------+
                                   |
                                   v
                           +------------------------------------------+
                           | 6. Demande d'alerte client               |
                           |    (Safeguard L3)                        |
                           |    "Autoriser l'envoi SMS aux clients    |
                           |     X, Y, Z pour les informer ?"         |
                           +------------------------------------------+
                                   |
                        +----------+----------+
                        |                     |
                        v                     v
                  +-----------+         +-----------+
                  | APPROUVE  |         | REFUSE    |
                  +-----------+         +-----------+
                        |                     |
                        v                     |
                  +------------------------------------------+
                  | 7. Envoi SMS aux clients                 |
                  |    "Bonjour, nous avons detecte une      |
                  |     panne sur votre liaison. Ticket      |
                  |     #12345 ouvert. Nous travaillons      |
                  |     a la resolution."                    |
                  +------------------------------------------+
                        |
                        v
                  +------------------------------------------+
                  | 8. Demande ouverture ticket              |
                  |    -> Appel API Assistant Ticket         |
                  +------------------------------------------+
```

### Points Cles

| Aspect | Description |
|--------|-------------|
| **Frequence** | Interrogation toutes les 1 minute |
| **Cache Redis** | Stocke l'etat des equipements pour detecter les pannes persistantes (seuil: 20 min) |
| **Verification FAI** | Le technicien humain verifie manuellement si la panne vient du FAI ou de l'infrastructure geree par WIDIP |
| **Notifications** | SMS aux clients (via Safeguard L3), notification technicien |
| **Integration** | Peut declencher l'Assistant Ticket pour creer automatiquement un ticket |

### Benefices

- **Proactivite** : Les clients sont informes avant meme qu'ils n'appellent
- **Reduction charge support** : Moins d'appels entrants car les clients sont deja au courant
- **Traçabilite** : Toutes les pannes sont documentees avec timeline precise
- **Qualification** : Distinction claire entre panne FAI et panne infrastructure WIDIP

---

## 3. ASSISTANT TICKET - Workflow de Gestion Automatisee

### Description

L'Assistant Ticket est un workflow autonome qui surveille les nouveaux tickets GLPI toutes les 3 minutes. Il qualifie chaque ticket en niveau de difficulte et peut resoudre automatiquement les tickets simples.

### Cycle de Fonctionnement

```
    +------------------------------------------+
    |        BOUCLE CONTINUE (3 min)           |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 1. Recuperation nouveaux tickets         |
    |    (MCP GLPI)                            |
    |    -> Tickets crees depuis dernier check |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 2. Pour chaque ticket :                  |
    |    QUALIFICATION                         |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 3. Analyse du contenu du ticket          |
    |    - Titre, description                  |
    |    - Categorie                           |
    |    - Client concerne                     |
    |    - Historique tickets similaires (RAG) |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 4. Attribution niveau difficulte         |
    +------------------------------------------+
                       |
           +-----------+-----------+-----------+
           |           |           |           |
           v           v           v           v
    +----------+ +----------+ +----------+ +----------+
    | SIMPLE   | | MOYEN    | | COMPLEXE | | EXPERT   |
    | L'IA     | | L'IA     | | Techni-  | | Techni-  |
    | peut     | | peut     | | cien N1  | | cien N2+ |
    | resoudre | | tenter   | | requis   | | requis   |
    +----------+ +----------+ +----------+ +----------+
           |           |           |           |
           v           v           v           v
    +------------------------------------------+
    | 5. Traitement selon niveau               |
    +------------------------------------------+
```

### Tickets Resolvables Automatiquement (SIMPLE)

| Type de demande | Actions de l'IA | Niveau Safeguard |
|-----------------|-----------------|------------------|
| Creation compte AD | Creer l'utilisateur selon template | L3 (validation requise) |
| Reset mot de passe | Generer nouveau MDP, envoyer au user | L3 (validation requise) |
| Deblocage compte | Unlock dans AD | L2 (notification tech) |
| Ajout groupe AD | Ajouter user au groupe demande | L2 (notification tech) |
| Info procedure | Fournir la procedure depuis RAG | L0 (auto) |

### Flux de Resolution Automatique

```
    Ticket SIMPLE detecte
            |
            v
    +------------------------------------------+
    | 1. Recherche procedure dans RAG          |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 2. Preparation des actions               |
    |    (ex: parametres creation compte)      |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 3. Demande Safeguard                     |
    |    "Ticket #123 - Creation compte AD     |
    |     pour Jean Dupont.                    |
    |     Parametres: OU=Users, Groups=...     |
    |     Autoriser ?"                         |
    +------------------------------------------+
            |
            +----------+----------+
            |                     |
            v                     v
      +-----------+         +-----------+
      | APPROUVE  |         | REFUSE    |
      +-----------+         +-----------+
            |                     |
            v                     v
    +------------------+  +------------------+
    | Execution action |  | Ticket reste     |
    | (MCP AD)         |  | en attente       |
    +------------------+  | technicien       |
            |             +------------------+
            v
    +------------------------------------------+
    | 4. Mise a jour ticket GLPI               |
    |    - Ajout suivi avec actions realisees  |
    |    - Demande cloture (Safeguard)         |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 5. Notification demandeur                |
    |    (email avec nouveau MDP si applicable)|
    +------------------------------------------+
```

### Gestion des Tickets Non Resolus

```
    Ticket COMPLEXE ou EXPERT
            |
            v
    +------------------------------------------+
    | 1. Flag du ticket                        |
    |    -> Stockage ID dans Redis             |
    |    -> Tag "IA_NON_RESOLU"                |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 2. Le ticket reste pour le technicien    |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 3. Quand le tech resout le ticket :      |
    |    -> L'Enrichisseur analyse la solution |
    |    -> Potentielle nouvelle procedure     |
    +------------------------------------------+
```

### Benefices

- **Reduction temps resolution** : Les tickets simples sont traites en minutes
- **Disponibilite 24/7** : L'IA traite meme hors heures ouvrees
- **Coherence** : Memes procedures appliquees systematiquement
- **Liberation techniciens** : Se concentrent sur les cas complexes
- **Apprentissage** : Les tickets non resolus alimentent l'Enrichisseur

---

## 4. ENRICHISSEUR - Workflow de Capitalisation

### Description

L'Enrichisseur est un workflow batch (quotidien ou hebdomadaire) qui analyse les tickets resolus pour en extraire des procedures. Il transforme l'experience des techniciens en connaissances reutilisables.

### Cycle de Fonctionnement

```
    +------------------------------------------+
    |     DECLENCHEMENT (Fin de journee)       |
    |     ou HEBDOMADAIRE selon config         |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 1. Collecte des donnees                  |
    |    - Tickets clotures du jour/semaine    |
    |    - Tickets flags "IA_NON_RESOLU"       |
    |      maintenant resolus                  |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 2. Filtrage et selection                 |
    |    - Tickets bien documentes             |
    |    - Tickets recurrents (meme type)      |
    |    - Resolutions reussies                |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 3. Analyse par le LLM                    |
    |    - Extraction des etapes de resolution |
    |    - Identification du pattern           |
    |    - Redaction procedure structuree      |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 4. Generation proposition procedure      |
    |    - Titre                               |
    |    - Contexte / Symptomes                |
    |    - Etapes detaillees                   |
    |    - Verification                        |
    +------------------------------------------+
                       |
                       v
    +------------------------------------------+
    | 5. Soumission pour validation            |
    |    (Safeguard)                           |
    |    "Nouvelle procedure detectee :        |
    |     [Titre]                              |
    |     Basee sur X tickets similaires       |
    |     Valider et ajouter a la base ?"      |
    +------------------------------------------+
                       |
            +----------+----------+
            |                     |
            v                     v
      +-----------+         +-----------+
      | APPROUVE  |         | REFUSE    |
      | (+ modif  |         | ou        |
      | possibles)|         | A REVOIR  |
      +-----------+         +-----------+
            |                     |
            v                     |
    +------------------------------------------+
    | 6. Stockage de la procedure              |
    |    - Ajout au RAG (vectorisation)        |
    |    - Ajout dans GLPI (base procedures)   |
    +------------------------------------------+
```

### Types de Procedures Generees

| Source | Type de procedure | Exemple |
|--------|-------------------|---------|
| Tickets simples recurrents | Procedure standard | "Reset mot de passe Office 365" |
| Tickets flags resolus par tech | Procedure avancee | "Diagnostic lenteur VPN site distant" |
| Conversations Agent complexes | Procedure R&D | "Migration Exchange vers O365" |

### Cercle Vertueux de l'Apprentissage

```
    +------------------+
    |   Tickets du     |
    |   jour           |
    +--------+---------+
             |
             v
    +------------------+
    |  Enrichisseur    |
    |  analyse et      |
    |  propose         |
    +--------+---------+
             |
             v
    +------------------+
    |  Validation      |
    |  humaine         |
    +--------+---------+
             |
             v
    +------------------+
    |  RAG enrichi     |
    |  + procedures    |
    +--------+---------+
             |
             v
    +------------------+         +------------------+
    |  Agent WIBOT     |         |  Assistant       |
    |  plus performant |         |  Ticket plus     |
    |  (mode Agent)    |         |  autonome        |
    +--------+---------+         +--------+---------+
             |                            |
             +------------+---------------+
                          |
                          v
                 +------------------+
                 |  Meilleure       |
                 |  resolution      |
                 |  tickets         |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 |  Plus de tickets |
                 |  bien resolus    |
                 +--------+---------+
                          |
                          +-----> Retour au debut
```

### Benefices

- **Capitalisation** : L'expertise ne part pas avec les employes
- **Integration nouveaux** : Documentation riche et a jour
- **Autonomie croissante** : L'IA devient plus competente avec le temps
- **Qualite** : Validation humaine garantit la pertinence des procedures

---

## 5. SAFEGUARD - Systeme de Securite

### Description

Safeguard est le verrou de securite qui controle TOUTES les actions critiques avant leur execution. Il garantit qu'aucune action sensible n'est realisee sans validation humaine appropriee.

### Niveaux d'Action

| Niveau | Nom | Description | Comportement |
|--------|-----|-------------|--------------|
| **L0** | READ_ONLY | Lecture seule | Execution automatique |
| **L1** | MINOR | Modification mineure, reversible | Auto si confiance > 80% |
| **L2** | MODERATE | Impact modere | Notification technicien requise |
| **L3** | SENSITIVE | Action sensible | **Validation humaine OBLIGATOIRE** |
| **L4** | FORBIDDEN | Action interdite | **L'IA ne peut JAMAIS executer** |

### Exemples par Niveau

| Niveau | Actions |
|--------|---------|
| **L0** | Rechercher un ticket, Voir les metriques Observium, Consulter le RAG |
| **L1** | Creer un ticket, Ajouter un suivi |
| **L2** | Debloquer un compte AD, Notifier un technicien |
| **L3** | Reset mot de passe, Cloturer ticket, Envoyer SMS client, Creer compte AD |
| **L4** | Supprimer compte AD, Actions irreversibles sur production |

### Niveaux d'Accreditation Utilisateur

| Niveau | Role | Peut valider |
|--------|------|--------------|
| **N0** | Lecture seule | Rien |
| **N1** | Technicien | L1 |
| **N2** | Technicien senior | L1, L2 |
| **N3** | Administrateur | L1, L2, L3 |
| **N4** | Super administrateur | L1, L2, L3 + config systeme |

### Interface de Validation

```
+------------------------------------------------------------------+
|                    SAFEGUARD - Demande en attente                 |
+------------------------------------------------------------------+
|                                                                   |
|  [WORKFLOW: Sentinel]                    [NIVEAU: L3 - SENSITIVE] |
|                                                                   |
|  CONTEXTE:                                                        |
|  -----------------------------------------------------------------|
|  Infrastructure "Switch-Client-ABC" detectee DOWN depuis 25 min.  |
|  Clients impactes: Entreprise ABC (3 sites), Societe XYZ          |
|  Verification FAI: En attente de votre diagnostic                 |
|                                                                   |
|  ACTION DEMANDEE:                                                 |
|  -----------------------------------------------------------------|
|  Envoyer SMS d'information aux contacts suivants :                |
|  - Jean Dupont (ABC) : 06 12 34 56 78                             |
|  - Marie Martin (XYZ) : 06 98 76 54 32                            |
|                                                                   |
|  Message prevu :                                                  |
|  "WIDIP - Incident detecte sur votre liaison. Ticket #12345       |
|   ouvert. Nos equipes travaillent a la resolution."               |
|                                                                   |
|  JUSTIFICATION:                                                   |
|  -----------------------------------------------------------------|
|  Panne persistante > 20min. Information proactive des clients     |
|  pour eviter saturation du support.                               |
|                                                                   |
|  +------------------+  +------------------+  +------------------+  |
|  |    APPROUVER     |  |     REFUSER      |  |    MODIFIER      |  |
|  +------------------+  +------------------+  +------------------+  |
|                                                                   |
|  [ ] Panne FAI confirmee                                          |
|  [ ] Panne infra WIDIP confirmee                                        |
|                                                                   |
+------------------------------------------------------------------+
```

### Flux de Validation

```
    Action L2+ demandee par un workflow
                    |
                    v
    +------------------------------------------+
    | Creation demande Safeguard               |
    | - Contexte                               |
    | - Action demandee                        |
    | - Justification                          |
    | - Niveau requis                          |
    +------------------------------------------+
                    |
                    v
    +------------------------------------------+
    | Notification techniciens accredites      |
    | (selon niveau requis)                    |
    +------------------------------------------+
                    |
                    v
    +------------------------------------------+
    | Affichage dans onglet Safeguard          |
    | du WIBOT                                 |
    +------------------------------------------+
                    |
                    v
    +------------------------------------------+
    | Technicien examine et decide             |
    +------------------------------------------+
                    |
         +----------+----------+
         |                     |
         v                     v
   +-----------+         +-----------+
   | APPROUVE  |         | REFUSE    |
   +-----------+         +-----------+
         |                     |
         v                     v
   +-------------+       +-------------+
   | Execution   |       | Workflow    |
   | de l'action |       | informe,    |
   +-------------+       | alternative |
         |               +-------------+
         v
   +------------------------------------------+
   | Log dans safeguard_audit                 |
   | (qui, quoi, quand, decision)             |
   +------------------------------------------+
```

---

## 6. RAG - Base de Connaissances

### Description

Le RAG (Retrieval-Augmented Generation) est le cerveau documentaire de la plateforme. Il stocke et permet de rechercher toutes les connaissances de l'entreprise WIDIP.

### Sources de Connaissances

| Source | Type | Alimentation |
|--------|------|--------------|
| Documentation interne | Manuels, guides WIDIP | Upload manuel |
| Procedures techniques | How-to, troubleshooting | Enrichisseur (auto) |
| Tickets resolus | Cas concrets | Enrichisseur (auto) |
| Conversations Agent | R&D, cas complexes | Semi-auto apres validation |

### Fonctionnement Technique

```
    Document/Procedure
            |
            v
    +------------------------------------------+
    | 1. Decoupage en chunks                   |
    |    (500 caracteres, 50 overlap)          |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 2. Vectorisation                         |
    |    (Mistral Embed -> 1024 dimensions)    |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 3. Stockage PostgreSQL + pgvector        |
    |    (index HNSW pour recherche rapide)    |
    +------------------------------------------+


    Requete utilisateur
            |
            v
    +------------------------------------------+
    | 1. Vectorisation de la question          |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 2. Recherche similarite cosinus          |
    |    (top 5-10 chunks pertinents)          |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 3. Injection dans le contexte LLM        |
    +------------------------------------------+
            |
            v
    +------------------------------------------+
    | 4. Reponse enrichie et sourcee           |
    +------------------------------------------+
```

### Benefices

- **Centralisation** : Une seule source de verite
- **Recherche semantique** : Trouve par le sens, pas les mots-cles
- **Evolutif** : S'enrichit automatiquement avec le temps
- **Tracable** : Sources citees dans les reponses

---

## 7. COUCHE MCP - Outils Disponibles

### MCP GLPI (Ticketing)

| Outil | Niveau | Description |
|-------|--------|-------------|
| `glpi_search_client` | L0 | Rechercher un client |
| `glpi_search_new_tickets` | L0 | Lister les nouveaux tickets |
| `glpi_get_ticket_details` | L0 | Details d'un ticket |
| `glpi_get_ticket_history` | L0 | Historique d'un ticket |
| `glpi_search_procedure` | L0 | Chercher une procedure |
| `glpi_create_ticket` | L1 | Creer un ticket |
| `glpi_add_ticket_followup` | L1 | Ajouter un suivi |
| `glpi_update_ticket_status` | L2 | Changer le statut |
| `glpi_assign_ticket` | L2 | Assigner a un tech |
| `glpi_close_ticket` | L3 | Cloturer un ticket |
| `glpi_send_email` | L1 | Envoyer un email via GLPI |

### MCP Observium (Monitoring)

| Outil | Niveau | Description |
|-------|--------|-------------|
| `observium_search_devices` | L0 | Rechercher un equipement |
| `observium_list_devices` | L0 | Lister les equipements |
| `observium_get_device_status` | L0 | Etat d'un equipement (UP/DOWN) |
| `observium_get_device_metrics` | L0 | Metriques (CPU, RAM, bande passante) |
| `observium_get_device_alerts` | L0 | Alertes actives |
| `observium_get_device_history` | L0 | Historique de performance |

### MCP Active Directory

| Outil | Niveau | Description |
|-------|--------|-------------|
| `ad_check_user` | L0 | Verifier si un user existe |
| `ad_get_user_info` | L0 | Informations d'un user |
| `ad_unlock_account` | L2 | Debloquer un compte |
| `ad_enable_account` | L2 | Activer un compte |
| `ad_move_to_ou` | L2 | Deplacer vers une OU |
| `ad_reset_password` | L3 | Reinitialiser mot de passe |
| `ad_disable_account` | L3 | Desactiver un compte |
| `ad_copy_groups_from` | L3 | Copier les groupes d'un autre user |
| `ad_create_user` | L4 | Creer un utilisateur (**INTERDIT IA**) |

### MCP Notifications

| Outil | Niveau | Description |
|-------|--------|-------------|
| `notify_technician` | L2 | Notifier un technicien |
| `request_human_validation` | L1 | Demander une validation humaine |
| `notify_client` | L4 | Notifier un client (**Phase test: INTERDIT**) |
| `send_sms` | L3 | Envoyer un SMS |

---

## 8. Architecture Technique

### Services Docker

```
+------------------------------------------------------------------+
|                         NGINX (Port 8080)                         |
|                        Reverse Proxy / Entry Point                |
+------------------------------------------------------------------+
         |                    |                    |
         v                    v                    v
+----------------+  +------------------+  +------------------+
|    FRONTEND    |  |     BACKEND      |  |   MCP SERVER     |
|    (React)     |  |    (FastAPI)     |  |   (FastAPI)      |
|                |  |    Port 8000     |  |   Port 3001      |
+----------------+  +------------------+  +------------------+
                             |                    |
                             v                    v
                    +------------------+  +------------------+
                    |   PostgreSQL     |  |      Redis       |
                    |   + pgvector     |  |     (Cache)      |
                    |   Port 5433      |  |   Port 6379      |
                    +------------------+  +------------------+
```

### Communication entre Workflows

```
    +------------------+
    |    SENTINEL      |
    +--------+---------+
             |
             | API REST (FastAPI)
             v
    +------------------+
    | ASSISTANT TICKET |
    +--------+---------+
             |
             | Partage Redis (IDs tickets)
             v
    +------------------+
    |   ENRICHISSEUR   |
    +------------------+
```

### Modeles LLM Utilises

| Composant | Modele | Usage |
|-----------|--------|-------|
| WIBOT Flash | Devstral Small | Reponses rapides |
| WIBOT Pro | Mistral Large | Reponses complexes |
| WIBOT Code | Devstral | Generation code |
| WIBOT Agent | Mistral Large | Raisonnement + outils |
| Sentinel | Devstral 2 / Mistral Large | Analyse pannes |
| Assistant Ticket | Devstral 2 / Mistral Large | Qualification + resolution |
| Enrichisseur | Mistral Large | Generation procedures |
| Embeddings | Mistral Embed | Vectorisation RAG |

---

## 9. Flux de Donnees Global

```
                           UTILISATEURS
                                |
            +-------------------+-------------------+
            |                   |                   |
            v                   v                   v
    +---------------+   +---------------+   +---------------+
    |  Technicien   |   |   Client      |   |    Admin      |
    |  (WIBOT)      |   |   (SMS)       |   | (Config)      |
    +-------+-------+   +---------------+   +-------+-------+
            |                   ^                   |
            v                   |                   v
    +-------+-------------------+-------------------+-------+
    |                        WIBOT                          |
    |   Chat (Flash/Pro/Code/Agent) | Safeguard | Knowledge |
    +---------------------------+---------------------------+
                                |
         +----------------------+----------------------+
         |                      |                      |
         v                      v                      v
    +----------+          +----------+          +----------+
    | SENTINEL |          | ASSISTANT|          |ENRICHISS.|
    | Workflow |          |  TICKET  |          |  Workflow|
    +----+-----+          +----+-----+          +----+-----+
         |                     |                     |
         +----------+----------+----------+----------+
                    |                     |
                    v                     v
              +----------+          +----------+
              |   MCP    |          |   RAG    |
              | Servers  |          |          |
              +----+-----+          +----+-----+
                   |                     |
    +--------------+-------+-------------+
    |              |       |             |
    v              v       v             v
+-------+    +-------+  +-----+    +----------+
| GLPI  |    |Observ.|  | AD  |    |PostgreSQL|
+-------+    +-------+  +-----+    +----------+
```

---

## 10. Securite et Authentification

### Authentification Utilisateurs

```
    +------------------------------------------+
    | Connexion WIBOT                          |
    +------------------------------------------+
                    |
                    v
    +------------------------------------------+
    | Identifiants WIDIP                       |
    | (synchronisation avec matrice Excel)     |
    +------------------------------------------+
                    |
                    v
    +------------------------------------------+
    | Verification credentials                 |
    | + Attribution niveau accreditation       |
    +------------------------------------------+
                    |
                    v
    +------------------------------------------+
    | Generation JWT Token                     |
    | (24h expiration)                         |
    +------------------------------------------+
```

### Matrice d'Autorisation

La matrice d'autorisation WIDIP (fichier Excel) definit :
- Les utilisateurs autorises
- Leur niveau d'accreditation (N0 a N4)
- Leurs permissions specifiques

### Audit Trail

Toutes les actions sont tracees dans `safeguard_audit` :
- Qui a fait l'action
- Quand
- Quelle action
- Resultat (approuve/refuse)
- Contexte complet

---

## 11. Features a Implementer

### Priorite HAUTE

| Feature | Description | Composant |
|---------|-------------|-----------|
| **MCP Code Mode** | Optimiser les calls MCP Observium pour Sentinel (1 call/min actuellement) | MCP Observium |
| **Verification FAI autonome** | Sentinel detecte automatiquement si panne FAI ou infra WIDIP | Sentinel |
| **Stockage tickets non resolus** | Systeme de flagging Redis pour tickets que l'IA n'a pas pu resoudre | Assistant Ticket |
| **Synchronisation matrice WIDIP** | Import automatique des accreditations depuis Excel | Backend |

### Priorite MOYENNE

| Feature | Description | Composant |
|---------|-------------|-----------|
| **Execution procedure par Agent** | L'Agent peut executer une procedure (avec Safeguard) | WIBOT Agent |
| **Choix destinataires SMS** | Interface pour definir qui alerter selon le type de panne | Sentinel |
| **Double stockage procedures** | RAG + GLPI pour les procedures validees | Enrichisseur |
| **Frequence Enrichisseur configurable** | Quotidien ou hebdomadaire selon volume | Enrichisseur |

### Priorite BASSE (Futures)

| Feature | Description | Composant |
|---------|-------------|-----------|
| **SSO/LDAP WIDIP** | Authentification via AD WIDIP | Backend |
| **Procedures depuis conversations** | Extraire procedures des conversations Agent complexes | Enrichisseur |
| **Dashboard analytics avance** | Metriques detaillees des workflows | WIBOT |
| **Notification Teams** | Alternative aux SMS pour certains cas | Sentinel |

---

## 12. Benefices Globaux du Systeme

### Pour les Techniciens

- **Gain de temps** : Tickets simples traites automatiquement
- **Acces instantane** : Base de connaissances centralisee via chat
- **Proactivite** : Informes des pannes avant les clients
- **Assistance intelligente** : L'Agent aide au diagnostic

### Pour les Clients

- **Reactivite** : Informes proactivement des incidents
- **Transparence** : Numero de ticket communique par SMS
- **Resolution rapide** : Tickets simples traites 24/7

### Pour l'Entreprise

- **Capitalisation** : Les connaissances restent malgre les departs
- **Scalabilite** : L'IA gere la charge des tickets simples
- **Qualite** : Procedures standardisees et validees
- **Evolution** : Le systeme s'ameliore avec le temps

### Cercle Vertueux

```
    Plus de tickets resolus
            |
            v
    Plus de procedures creees
            |
            v
    IA plus performante
            |
            v
    Plus de tickets resolus automatiquement
            |
            v
    Techniciens sur cas complexes
            |
            v
    Meilleures resolutions documentees
            |
            +-----> Retour au debut
```

---

## 13. Resume Executif

Cette plateforme d'IA, concue pour le support informatique de **WIDIP**, est composee de :

1. **WIBOT** : Interface conversationnelle avec 4 modes (Flash, Pro, Code, Agent)

2. **3 Workflows autonomes** :
   - **Sentinel** : Surveillance proactive des infrastructures clientes
   - **Assistant Ticket** : Qualification et resolution automatique des tickets
   - **Enrichisseur** : Capitalisation des connaissances

3. **Safeguard** : Systeme de securite garantissant le controle humain sur les actions critiques

4. **RAG** : Base de connaissances centralisee et evolutive

5. **MCP** : Connecteurs vers les systemes WIDIP (GLPI, Observium, Active Directory)

**Objectif** : Transformer le support reactif en support proactif, capitaliser sur l'expertise des techniciens, et liberer les equipes pour les cas complexes - tout en garantissant la securite via validation humaine.

---

*Document genere le 19/01/2026*
*Version: 2.0*
*Auteur: Architecture concue pour WIDIP*
