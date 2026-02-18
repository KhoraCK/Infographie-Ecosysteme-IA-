# Operation R&D 1 - Capitalisation automatique des connaissances IT par IA

**Projet** : WIDIP / WIBOT
**Date de creation** : 3 fevrier 2026
**Statut** : En cours de documentation

---

## Resume

Cette operation de recherche porte sur la transformation automatique de l'historique operationnel non structure (tickets GLPI, conversations, resolutions) en une base de connaissances fiable et evolutive, exploitable par des agents IA.

**Composants concernes** : Enrichisseur + RAG evolutif
**Potentiel CIR** : FORT

---

## 1. Problematique scientifique

> Comment transformer automatiquement l'historique operationnel non structure d'une PME (tickets, conversations, resolutions) en une base de connaissances fiable, evolutive et exploitable par des agents IA — sans degradation de qualite dans le temps ?

---

## 2. Etat de l'art

### 2.1 Solutions existantes

| Solution | Description | Limites identifiees |
|----------|-------------|---------------------|
| *A completer* | | |
| *A completer* | | |

### 2.2 Positionnement du projet

*A completer : en quoi l'approche WIDIP se distingue des solutions existantes*

---

## 3. Verrous techniques

### 3.1 Verrou 1 — Extraction de procedures a partir de texte libre

**Description** :
Les tickets GLPI sont rediges en langage naturel non standardise. L'information utile est noyee dans du bruit (formules de politesse, descriptions imprecises, fautes). Le LLM doit identifier un pattern de resolution generalisable a partir de cas individuels.

**Incertitudes scientifiques** :
- Comment distinguer une resolution reellement transposable d'un cas trop specifique ?
- Comment extraire des etapes de procedure quand le technicien n'a pas documente toutes ses actions ?
- Quel seuil de recurrence (nombre de tickets similaires) est necessaire pour justifier la creation d'une procedure ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.2 Verrou 2 — Maintien de la qualite d'une base RAG auto-alimentee

**Description** :
La base de connaissances s'enrichit en continu. Cela cree un risque de data drift : procedures obsoletes, contradictions, redondances qui polluent les resultats de recherche semantique.

**Incertitudes scientifiques** :
- Comment detecter qu'une procedure existante est rendue obsolete par une nouvelle ?
- Comment mesurer la qualite globale du RAG au fil de son enrichissement ?
- Comment eviter que l'accumulation de documents degrade la precision de retrieval ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

### 3.3 Verrou 3 — Boucle de feedback et cercle vertueux

**Description** :
Le systeme repose sur un cercle : tickets resolus → procedures generees → RAG enrichi → IA plus performante → meilleure resolution → plus de matiere pour l'Enrichisseur. La convergence de cette boucle n'est pas garantie.

**Incertitudes scientifiques** :
- La boucle converge-t-elle reellement (amelioration mesurable) ou diverge-t-elle (accumulation de bruit) ?
- Comment mesurer l'impact de chaque procedure ajoutee sur la performance globale ?
- Quel est le ratio optimal entre procedures generees automatiquement et validees humainement ?

**Hypotheses testees** :

| # | Hypothese | Resultat | Date |
|---|-----------|----------|------|
| 1 | *A completer* | | |
| 2 | *A completer* | | |

**Solution retenue** :
*A completer*

---

## 4. Experimentations

### 4.1 Protocole de test

*A completer : description du protocole d'experimentation*

### 4.2 Metriques collectees

| Metrique | Description | Valeur initiale | Valeur actuelle |
|----------|-------------|-----------------|-----------------|
| Taux de validation des procedures | % de procedures proposees validees par un humain | *A mesurer* | *A mesurer* |
| Precision RAG | Pertinence des documents retrouves | *A mesurer* | *A mesurer* |
| Taux de resolution automatique | % de tickets resolus sans intervention humaine | *A mesurer* | *A mesurer* |

### 4.3 Resultats

*A completer : resultats des experimentations, graphiques si disponibles*

---

## 5. Iterations et historique

| Version | Date | Modifications | Resultat |
|---------|------|---------------|----------|
| *A completer* | | | |

---

## 6. Preuves materielles

- [ ] Logs de l'Enrichisseur : tickets analyses, procedures proposees
- [ ] Metriques RAG avant/apres enrichissement
- [ ] Exemples de procedures generees (anonymisees)
- [ ] Cas de procedures contradictoires ou obsoletes detectees
- [ ] Evolution du taux de resolution automatique

---

## 7. Evaluation Frascati

| Critere | Evaluation | Justification |
|---------|------------|---------------|
| Nouveaute | FORT | L'etat de l'art ne propose pas de solution etablie pour cette problematique |
| Creativite | FORT | Architecture originale de capitalisation automatique |
| Incertitude | FORT | 3 verrous identifies sans solution connue |
| Systematisation | A DOCUMENTER | Metriques et protocoles a formaliser |
| Transferabilite | MOYEN | Generalisable a tout contexte de support |

---

*Document a completer au fil des travaux de recherche*
