# Operation R&D 2 - Qualification et resolution autonome de tickets IT avec indice de confiance calibre

**Projet** : WIDIP / WIBOT
**Date de creation** : 3 fevrier 2026
**Statut** : En cours de documentation

---

## Resume

Cette operation de recherche porte sur la capacite d'un systeme IA a qualifier la difficulte d'un ticket IT et a resoudre les cas simples de facon autonome, tout en garantissant un niveau de confiance calibre et un fallback sur vers l'humain.

**Composants concernes** : Assistant Ticket + Sentinel (volet decision IA)
**Potentiel CIR** : FORT

---

## 1. Problematique scientifique

> Comment un systeme IA peut-il qualifier de maniere fiable la difficulte d'un ticket IT et resoudre les cas simples de facon autonome — tout en garantissant un niveau de confiance calibre et un fallback sur vers l'humain lorsque l'incertitude est trop elevee ?

---

## 2. Etat de l'art

### 2.1 Solutions existantes

| Solution | Description | Limites identifiees |
|----------|-------------|---------------------|
| *A completer* | | |
| *A completer* | | |

### 2.2 Litterature sur la calibration de confiance des LLM

*A completer : references aux travaux de recherche sur la calibration de confiance des modeles de langage*

### 2.3 Positionnement du projet

*A completer : en quoi l'approche WIDIP se distingue*

---

## 3. Verrous techniques

### 3.1 Verrou 1 — Qualification automatique en 4 niveaux de difficulte

**Description** :
L'Assistant Ticket doit classer chaque ticket entrant en SIMPLE / MOYEN / COMPLEXE / EXPERT. Cette classification determine directement le traitement : resolution IA autonome vs escalade humaine.

**Incertitudes scientifiques** :
- Comment un LLM peut-il evaluer la difficulte reelle d'un ticket a partir d'une description souvent incomplete ou ambigue ?
- Comment gerer le cas critique du faux SIMPLE (ticket classe simple mais qui necessitait un expert) ?
- Comment integrer le contexte historique du client dans la qualification ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.2 Verrou 2 — Calibration de l'indice de confiance

**Description** :
Le systeme SAFEGUARD utilise un seuil de confiance (> 80% pour execution automatique en L1). Les LLM sont notoirement mal calibres — ils expriment souvent une haute confiance meme lorsqu'ils se trompent.

**Incertitudes scientifiques** :
- Comment obtenir un score de confiance fiable d'un LLM pour des actions IT concretes ?
- Comment calibrer ce score (faire en sorte que "80% de confiance" corresponde reellement a 80% de reussite) ?
- Le seuil optimal est-il universel ou doit-il varier par type d'action, par client, par complexite ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.3 Verrou 3 — Resolution autonome avec actions irreversibles

**Description** :
Resoudre un ticket simple implique des actions reelles : reset de mot de passe AD, deblocage de compte, creation d'utilisateur. Ces actions ont des effets concrets et certaines sont irreversibles.

**Incertitudes scientifiques** :
- Comment valider qu'une procedure RAG retrouvee est applicable au cas precis du ticket courant ?
- Comment detecter une mauvaise application de procedure avant l'execution de l'action ?
- Quel protocole de rollback pour les actions partiellement executees en cas d'erreur ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.4 Verrou 4 — Decision autonome dans le monitoring proactif (Sentinel)

**Description** :
Sentinel doit prendre des decisions en temps reel : identifier les clients impactes par une panne, determiner s'il s'agit d'une panne FAI ou infrastructure, decider quand et qui alerter.

**Incertitudes scientifiques** :
- Comment l'IA peut-elle distinguer automatiquement une panne FAI d'une panne infrastructure WIDIP ?
- Comment le parsing de hostnames peut-il gerer les cas non conformes au pattern standard ?
- Comment correler automatiquement un equipement DOWN avec les bonnes entites GLPI ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

## 4. Experimentations

### 4.1 Protocole de test - Qualification

*A completer : description du protocole pour evaluer la qualification*

### 4.2 Protocole de test - Calibration

*A completer : description du protocole pour evaluer la calibration de confiance*

### 4.3 Metriques collectees

| Metrique | Description | Valeur initiale | Valeur actuelle |
|----------|-------------|-----------------|-----------------|
| Matrice de confusion qualification | Qualification predite vs reelle | *A mesurer* | *A mesurer* |
| Courbe de calibration | Confiance exprimee vs taux de reussite | *A mesurer* | *A mesurer* |
| Taux de faux SIMPLE | Tickets classes SIMPLE qui etaient COMPLEXE+ | *A mesurer* | *A mesurer* |
| Taux de resolution autonome reussie | Tickets resolus par l'IA sans erreur | *A mesurer* | *A mesurer* |

### 4.4 Resultats

*A completer : resultats des experimentations*

---

## 5. Iterations et historique

| Version | Date | Modifications | Resultat |
|---------|------|---------------|----------|
| *A completer* | | | |

---

## 6. Preuves materielles

- [ ] Matrice de confusion : qualification predite vs qualification reelle
- [ ] Courbe de calibration : confiance exprimee vs taux de reussite effectif
- [ ] Logs de resolution : tickets resolus automatiquement, incidents causes par l'IA
- [ ] Historique des tickets "IA_NON_RESOLU"
- [ ] Documentation des iterations sur les seuils de confiance
- [ ] Taux de faux positifs/negatifs Sentinel sur l'identification des pannes

---

## 7. Evaluation Frascati

| Critere | Evaluation | Justification |
|---------|------------|---------------|
| Nouveaute | FORT | La calibration de confiance d'un LLM pour des actions IT est un probleme ouvert |
| Creativite | FORT | Couplage qualification/resolution avec feedback pour enrichissement |
| Incertitude | FORT | 4 verrous identifies, dont la calibration qui est un probleme de recherche reconnu |
| Systematisation | A DOCUMENTER | Matrices de confusion et courbes de calibration a produire |
| Transferabilite | FORT | Methodologie transposable a tout domaine d'automatisation IA |

---

*Document a completer au fil des travaux de recherche*
