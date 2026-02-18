# Operation R&D 3 - Gouvernance de securite pour systeme multi-agents IA operationnel

**Projet** : WIDIP / WIBOT
**Date de creation** : 3 fevrier 2026
**Statut** : En cours de documentation

---

## Resume

Cette operation de recherche porte sur la conception d'un framework de gouvernance (SAFEGUARD) garantissant la securite d'un systeme multi-agents IA operant sur des systemes de production IT avec des actions pouvant etre irreversibles.

**Composants concernes** : SAFEGUARD (framework complet)
**Potentiel CIR** : FORT

---

## 1. Problematique scientifique

> Comment concevoir un framework de gouvernance garantissant la securite d'un systeme multi-agents IA (3 workflows autonomes + 1 agent conversationnel) operant sur des systemes de production IT (Active Directory, ticketing, monitoring reseau) avec des actions pouvant etre irreversibles ?

---

## 2. Etat de l'art

### 2.1 Solutions existantes

| Solution | Description | Limites identifiees |
|----------|-------------|---------------------|
| LangChain Guardrails | Filtrage de contenu et validation | Limite aux filtres textuels, pas de gestion multi-niveaux |
| Microsoft AutoGen Safety | Controles pour agents autonomes | Peu adapte aux systemes IT operationnels |
| *A completer* | | |

### 2.2 Litterature sur la securite des agents IA

*A completer : references aux travaux de recherche sur la gouvernance des agents autonomes*

### 2.3 Positionnement du projet

*A completer : en quoi SAFEGUARD se distingue des frameworks existants*

---

## 3. Verrous techniques

### 3.1 Verrou 1 — Modele de securite multi-niveaux pour agents IA

**Description** :
Le framework SAFEGUARD definit 6 niveaux de sensibilite (L0 a L4_SMS) croises avec 5 niveaux d'accreditation utilisateur (N0 a N4). Cette matrice determine dynamiquement ce qu'un agent IA peut faire, ce qu'il doit demander, et ce qu'il ne peut jamais faire.

**Incertitudes scientifiques** :
- Comment definir la bonne granularite de niveaux ? Trop peu = risque de securite, trop = friction operationnelle
- Comment gerer les cas limites entre deux niveaux ?
- Le modele est-il stable lorsque de nouveaux outils MCP sont ajoutes ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.2 Verrou 2 — Coherence dans une architecture distribuee

**Description** :
Le double systeme SAFEGUARD (backend PostgreSQL + MCP Server Python) a genere un probleme concret : apres approbation humaine cote backend, le MCP Server re-bloquait l'action. La solution (`_bypass_safeguard`) implique de maintenir la coherence d'etat entre deux systemes independants dans un contexte asynchrone.

**Incertitudes scientifiques** :
- Comment garantir la synchronisation des niveaux de securite entre deux systemes independants ?
- Comment prevenir les race conditions entre l'approbation et l'execution dans un flux SSE asynchrone ?
- Comment auditer de maniere fiable les actions executees via bypass ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | Synchronisation via base de donnees partagee | *A completer* | |
| 2 | Mecanisme de bypass avec token d'approbation | *A completer* | |

**Bug rencontre - Double blocage** :
*A documenter : description detaillee du bug, analyse, hypotheses testees, solution retenue*

**Solution retenue** :
*A completer*

---

### 3.3 Verrou 3 — Orchestration securisee de workflows autonomes chaines

**Description** :
Sentinel peut declencher l'Assistant Ticket, qui peut generer des actions necessitant des validations SAFEGUARD. Cette chaine cree des scenarios complexes ou un workflow declenche un autre workflow qui declenche une action L3 necessitant validation humaine.

**Incertitudes scientifiques** :
- Comment gerer les dependances entre actions dans une chaine de workflows ?
- Comment eviter les deadlocks quand un workflow attend une approbation pendant qu'un autre genere de nouvelles demandes ?
- Comment prioriser les demandes SAFEGUARD quand plusieurs workflows sollicitent simultanement l'humain ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.4 Verrou 4 — Confiance dynamique et niveau L4_SMS specialise

**Description** :
Le niveau L4_SMS a ete cree specifiquement pour l'alerte SMS client — un cas ou ni le blocage total (L4) ni la simple validation (L3) ne suffisaient. Il integre une interface de selection de contact avec recommandation IA et apprentissage par feedback humain.

**Incertitudes scientifiques** :
- Comment le systeme apprend-il quel contact recommander au fil du temps ?
- Le score d'activite DirEnt est-il un proxy fiable pour la pertinence du contact ?
- Comment gerer la degradation de la recommandation quand les contacts changent de poste ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | Score base sur l'activite recente dans GLPI | *A completer* | |
| 2 | Cache Redis avec TTL variable selon confirmation humaine | *A completer* | |

**Solution retenue** :
*A completer*

---

## 4. Architecture SAFEGUARD

### 4.1 Matrice des niveaux

| Niveau | Nom | Description | Comportement |
|--------|-----|-------------|--------------|
| L0 | READ_ONLY | Lecture seule | Execution automatique |
| L1 | MINOR | Modification mineure, reversible | Auto si confiance > 80% |
| L2 | MODERATE | Impact modere | Notification technicien requise |
| L3 | SENSITIVE | Action sensible | Validation humaine OBLIGATOIRE |
| L4 | FORBIDDEN | Action interdite | L'IA ne peut JAMAIS executer |
| L4_SMS | SMS_ALERT | Alerte SMS client | Interface specialisee avec selection contact |

### 4.2 Flux d'approbation

*A completer : diagramme du flux d'approbation SAFEGUARD*

### 4.3 Mecanisme de bypass post-approbation

*A completer : documentation technique du mecanisme _bypass_safeguard*

---

## 5. Experimentations

### 5.1 Protocole de test

*A completer : description du protocole d'experimentation*

### 5.2 Metriques collectees

| Metrique | Description | Valeur initiale | Valeur actuelle |
|----------|-------------|-----------------|-----------------|
| Demandes par niveau | Repartition des demandes SAFEGUARD | *A mesurer* | *A mesurer* |
| Temps de reponse humain | Delai entre demande et approbation | *A mesurer* | *A mesurer* |
| Taux d'approbation | % de demandes approuvees | *A mesurer* | *A mesurer* |
| Near-misses | Actions potentiellement dangereuses bloquees | *A mesurer* | *A mesurer* |

### 5.3 Resultats

*A completer : resultats des experimentations*

---

## 6. Iterations et historique

| Version | Date | Modifications | Resultat |
|---------|------|---------------|----------|
| v1 | *Date* | Modele initial L0-L3 | *Resultat* |
| v2 | *Date* | Ajout du niveau L4 | *Resultat* |
| v3 | *Date* | Creation du niveau L4_SMS specialise | *Resultat* |
| v4 | *Date* | Resolution du bug double-blocage | *Resultat* |

---

## 7. Preuves materielles

- [ ] Etat de l'art : frameworks de controle d'agents IA existants
- [ ] Historique des iterations sur le modele de niveaux
- [ ] Documentation complete du bug double-blocage : analyse, hypotheses, solution
- [ ] Logs SAFEGUARD : demandes approuvees/rejetees, temps de reponse
- [ ] Analyse des near-misses : cas ou SAFEGUARD a bloque une action dangereuse
- [ ] Evolution du cache referent : pertinence des recommandations

---

## 8. Evaluation Frascati

| Critere | Evaluation | Justification |
|---------|------------|---------------|
| Nouveaute | FORT | Tres peu de frameworks de controle d'agents IA en production documentes |
| Creativite | FORT | Architecture L0-L4_SMS avec bypass post-approbation et apprentissage |
| Incertitude | FORT | 4 verrous, dont le probleme de coherence distribuee resolu experimentalement |
| Systematisation | MOYEN | Le bug double-blocage constitue une trace d'experimentation documentee |
| Transferabilite | FORT | Generalisable a tout systeme multi-agents avec actions critiques |

---

*Document a completer au fil des travaux de recherche*
