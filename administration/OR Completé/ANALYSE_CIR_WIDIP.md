# Analyse d'Éligibilité CIR — Plateforme IA WIDIP

**Projet** : WIBOT / WIDIP_ARCHI — Plateforme IA de support informatique proactif
**Date d'analyse** : 3 février 2026
**Référentiel** : Manuel de Frascati (OCDE) + Guide CIR 2025 (MESR)

---

## Vue d'ensemble

La plateforme WIDIP est un système multi-agents IA conçu pour le support informatique. Elle combine une interface conversationnelle (WIBOT), trois workflows autonomes (Sentinel, Assistant Ticket, Enrichisseur), un framework de sécurité (SAFEGUARD) et une base de connaissances évolutive (RAG).

L'analyse ci-dessous identifie **5 composants** porteurs de potentiel R&D, organisés en **3 opérations de recherche** structurées selon les 5 critères de Frascati.

---

## Cartographie R&D

```
                    PLATEFORME IA WIDIP
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   ┌────┴────┐      ┌────┴────┐      ┌────┴────┐
   │  OR 1   │      │  OR 2   │      │  OR 3   │
   │Capitali-│      │Qualifi- │      │Gouver-  │
   │sation   │      │cation   │      │nance    │
   │Auto.    │      │Autonome │      │Sécurité │
   └────┬────┘      └────┬────┘      └────┬────┘
        │                 │                 │
   ┌────┴────┐      ┌────┴────┐      ┌────┴────┐
   │Enrichis-│      │Assistant│      │SAFEGUARD│
   │seur     │      │Ticket   │      │Multi-   │
   │+ RAG    │      │+ Conf.  │      │Agents   │
   │évolutif │      │calibrée │      │         │
   └─────────┘      └─────────┘      └─────────┘

   ■ Potentiel FORT     ■ Potentiel MOYEN
   - Enrichisseur       - Sentinel
   - Assistant Ticket   - RAG évolutif
   - SAFEGUARD
```

---

---

# OPÉRATION R&D 1 — Capitalisation automatique des connaissances IT par IA

**Composants concernés** : Enrichisseur + RAG évolutif
**Potentiel CIR** : 🟢 FORT (Enrichisseur) + 🟡 MOYEN (RAG)

---

## Problématique scientifique

> Comment transformer automatiquement l'historique opérationnel non structuré d'une PME (tickets, conversations, résolutions) en une base de connaissances fiable, évolutive et exploitable par des agents IA — sans dégradation de qualité dans le temps ?

---

## Verrous techniques identifiés

### Verrou 1 — Extraction de procédures à partir de texte libre

Les tickets GLPI sont rédigés en langage naturel non standardisé par des techniciens et des utilisateurs finaux. L'information utile est noyée dans du bruit (formules de politesse, descriptions imprécises, fautes). Le LLM doit identifier un **pattern de résolution généralisable** à partir de cas individuels, ce qui dépasse la simple synthèse de texte.

**Incertitudes** :
- Comment distinguer une résolution réellement transposable d'un cas trop spécifique ?
- Comment extraire des étapes de procédure quand le technicien n'a pas documenté toutes ses actions ?
- Quel seuil de récurrence (nombre de tickets similaires) est nécessaire pour justifier la création d'une procédure ?

### Verrou 2 — Maintien de la qualité d'une base RAG auto-alimentée

La base de connaissances s'enrichit en continu via l'Enrichisseur. Cela crée un risque de **data drift** : procédures obsolètes, contradictions entre anciennes et nouvelles procédures, redondances qui polluent les résultats de recherche sémantique.

**Incertitudes** :
- Comment détecter qu'une procédure existante est rendue obsolète par une nouvelle ?
- Comment mesurer la qualité globale du RAG au fil de son enrichissement ?
- Comment éviter que l'accumulation de documents dégrade la précision de retrieval plutôt que de l'améliorer ?

### Verrou 3 — Boucle de feedback et cercle vertueux

Le système repose sur un cercle : tickets résolus → procédures générées → RAG enrichi → IA plus performante → meilleure résolution → plus de matière pour l'Enrichisseur. La convergence de cette boucle n'est pas garantie.

**Incertitudes** :
- La boucle converge-t-elle réellement (amélioration mesurable) ou diverge-t-elle (accumulation de bruit) ?
- Comment mesurer l'impact de chaque procédure ajoutée sur la performance globale du système ?
- Quel est le ratio optimal entre procédures générées automatiquement et procédures validées humainement ?

---

## Évaluation Frascati

| Critère | Évaluation | Justification |
|---------|------------|---------------|
| **Nouveauté** | ✅ Fort | L'état de l'art ne propose pas de solution établie pour la capitalisation automatique continue de connaissances IT opérationnelles via LLM avec boucle de feedback |
| **Créativité** | ✅ Fort | La chaîne complète ticket → analyse LLM → extraction pattern → génération procédure → validation humaine → vectorisation → réutilisation est une architecture originale |
| **Incertitude** | ✅ Fort | Trois verrous identifiés sans solution connue : extraction fiable, maintien qualité RAG, convergence de la boucle |
| **Systématisation** | ⚠️ À documenter | Nécessite des métriques de suivi, des protocoles d'expérimentation comparatifs, des critères de validation formalisés |
| **Transférabilité** | ✅ Moyen | La méthodologie est généralisable à tout contexte de support IT, voire au-delà (support client, maintenance industrielle) |

---

## Preuves à constituer

- Logs de l'Enrichisseur : tickets analysés, procédures proposées, taux de validation/rejet
- Métriques RAG : précision de retrieval avant/après enrichissement
- Expérimentations comparatives : critères de filtrage testés, seuils de récurrence
- Exemples de procédures contradictoires ou obsolètes détectées (ou non)
- Évolution du taux de résolution automatique corrélée à l'enrichissement

---

---

# OPÉRATION R&D 2 — Qualification et résolution autonome de tickets IT avec indice de confiance calibré

**Composants concernés** : Assistant Ticket + Sentinel (volet décision IA)
**Potentiel CIR** : 🟢 FORT (Assistant Ticket) + 🟡 MOYEN (Sentinel)

---

## Problématique scientifique

> Comment un système IA peut-il qualifier de manière fiable la difficulté d'un ticket IT et résoudre les cas simples de façon autonome — tout en garantissant un niveau de confiance calibré et un fallback sûr vers l'humain lorsque l'incertitude est trop élevée ?

---

## Verrous techniques identifiés

### Verrou 1 — Qualification automatique en 4 niveaux de difficulté

L'Assistant Ticket doit classer chaque ticket entrant en SIMPLE / MOYEN / COMPLEXE / EXPERT. Cette classification détermine directement le traitement : résolution IA autonome vs escalade humaine. Une erreur de classification a des conséquences opérationnelles directes.

**Incertitudes** :
- Comment un LLM peut-il évaluer la difficulté réelle d'un ticket à partir d'une description souvent incomplète ou ambiguë ?
- Comment gérer le cas critique du faux SIMPLE (ticket classé simple mais qui nécessitait un expert) ?
- Comment intégrer le contexte historique du client (tickets passés, infrastructure) dans la qualification ?

### Verrou 2 — Calibration de l'indice de confiance

Le système SAFEGUARD utilise un seuil de confiance (> 80% pour exécution automatique en L1). Ce seuil conditionne le degré d'autonomie du système. Or, les LLM sont notoirement **mal calibrés** en termes de confiance — ils expriment souvent une haute confiance même lorsqu'ils se trompent.

**Incertitudes** :
- Comment obtenir un score de confiance fiable d'un LLM pour des actions IT concrètes ?
- Comment calibrer ce score (faire en sorte que "80% de confiance" corresponde réellement à 80% de réussite) ?
- Le seuil optimal est-il universel ou doit-il varier par type d'action, par client, par complexité ?

### Verrou 3 — Résolution autonome avec actions irréversibles

Résoudre un ticket simple implique des actions réelles : reset de mot de passe AD, déblocage de compte, création d'utilisateur. Ces actions ont des effets concrets et certaines sont irréversibles.

**Incertitudes** :
- Comment valider qu'une procédure RAG retrouvée est applicable au cas précis du ticket courant ?
- Comment détecter une mauvaise application de procédure avant l'exécution de l'action ?
- Quel protocole de rollback pour les actions partiellement exécutées en cas d'erreur détectée en cours ?

### Verrou 4 — Décision autonome dans le monitoring proactif (Sentinel)

Sentinel doit prendre des décisions en temps réel : identifier les clients impactés par une panne, déterminer s'il s'agit d'une panne FAI ou infrastructure, décider quand et qui alerter.

**Incertitudes** :
- Comment l'IA peut-elle distinguer automatiquement une panne FAI d'une panne infrastructure WIDIP sans intervention humaine ? (Feature planifiée : "Vérification FAI autonome")
- Comment le parsing de hostnames (format CLIENT-SITE-VILLE-DEVICE) peut-il gérer les cas non conformes au pattern standard ?
- Comment corréler automatiquement un équipement Observium DOWN avec les bonnes entités GLPI quand les nomenclatures ne sont pas parfaitement alignées ?

---

## Évaluation Frascati

| Critère | Évaluation | Justification |
|---------|------------|---------------|
| **Nouveauté** | ✅ Fort | La calibration de confiance d'un LLM pour des actions IT opérationnelles avec conséquences réelles est un problème ouvert dans la littérature |
| **Créativité** | ✅ Fort | Le couplage qualification → résolution → flagging "IA_NON_RESOLU" → enrichissement ultérieur est une boucle de feedback innovante |
| **Incertitude** | ✅ Fort | Quatre verrous identifiés, dont la calibration de confiance qui est un problème de recherche reconnu en IA |
| **Systématisation** | ⚠️ À documenter | Nécessite matrice de confusion qualification, logs de résolution, mesure taux d'erreur, protocole de test |
| **Transférabilité** | ✅ Fort | La méthodologie de qualification par confiance calibrée est transposable à tout domaine d'automatisation par IA |

---

## Preuves à constituer

- Matrice de confusion : qualification prédite vs qualification réelle (sur un échantillon validé humainement)
- Courbe de calibration : confiance exprimée vs taux de réussite effectif
- Logs de résolution : tickets résolus automatiquement, incidents causés par l'IA
- Historique des tickets "IA_NON_RESOLU" : analyse post-résolution humaine
- Documentation des itérations sur les seuils de confiance
- Pour Sentinel : taux de faux positifs/négatifs sur l'identification automatique des pannes

---

---

# OPÉRATION R&D 3 — Gouvernance de sécurité pour système multi-agents IA opérationnel

**Composants concernés** : SAFEGUARD (framework complet)
**Potentiel CIR** : 🟢 FORT

---

## Problématique scientifique

> Comment concevoir un framework de gouvernance garantissant la sécurité d'un système multi-agents IA (3 workflows autonomes + 1 agent conversationnel) opérant sur des systèmes de production IT (Active Directory, ticketing, monitoring réseau) avec des actions pouvant être irréversibles ?

---

## Verrous techniques identifiés

### Verrou 1 — Modèle de sécurité multi-niveaux pour agents IA

Le framework SAFEGUARD définit 6 niveaux de sensibilité (L0 à L4_SMS) croisés avec 5 niveaux d'accréditation utilisateur (N0 à N4). Cette matrice détermine dynamiquement ce qu'un agent IA peut faire, ce qu'il doit demander, et ce qu'il ne peut jamais faire.

**Incertitudes** :
- Comment définir la bonne granularité de niveaux ? Trop peu = risque de sécurité, trop = friction opérationnelle
- Comment gérer les cas limites entre deux niveaux ?
- Le modèle est-il stable lorsque de nouveaux outils MCP sont ajoutés, ou nécessite-t-il une réévaluation globale ?

### Verrou 2 — Cohérence dans une architecture distribuée

Le double système SAFEGUARD (backend PostgreSQL + MCP Server Python) a généré un problème de recherche concret : après approbation humaine côté backend, le MCP Server re-bloquait l'action. La solution (`_bypass_safeguard`) implique de maintenir la cohérence d'état entre deux systèmes indépendants dans un contexte asynchrone.

**Incertitudes** :
- Comment garantir la synchronisation des niveaux de sécurité entre deux systèmes indépendants ?
- Comment prévenir les race conditions entre l'approbation et l'exécution dans un flux SSE asynchrone ?
- Comment auditer de manière fiable les actions exécutées via bypass dans un système distribué ?

### Verrou 3 — Orchestration sécurisée de workflows autonomes chaînés

Sentinel peut déclencher l'Assistant Ticket, qui peut générer des actions nécessitant des validations SAFEGUARD. Cette chaîne crée des scénarios complexes : un workflow autonome déclenche un autre workflow qui déclenche une action L3 nécessitant une validation humaine, pendant que le premier workflow continue de tourner.

**Incertitudes** :
- Comment gérer les dépendances entre actions dans une chaîne de workflows ?
- Comment éviter les deadlocks quand un workflow attend une approbation pendant qu'un autre génère de nouvelles demandes ?
- Comment prioriser les demandes SAFEGUARD quand plusieurs workflows sollicitent simultanément l'humain ?

### Verrou 4 — Confiance dynamique et niveau L4_SMS spécialisé

Le niveau L4_SMS a été créé spécifiquement pour l'alerte SMS client — un cas où ni le blocage total (L4) ni la simple validation (L3) ne suffisaient. Il intègre une interface de sélection de contact avec recommandation IA et apprentissage par feedback humain (cache Redis mis à jour si l'humain choisit un contact différent).

**Incertitudes** :
- Comment le système apprend-il quel contact recommander au fil du temps ?
- Le score d'activité DirEnt est-il un proxy fiable pour la pertinence du contact ?
- Comment gérer la dégradation de la recommandation quand les contacts changent de poste ?

---

## Évaluation Frascati

| Critère | Évaluation | Justification |
|---------|------------|---------------|
| **Nouveauté** | ✅ Fort | Très peu de frameworks de contrôle d'agents IA en production documentés dans la littérature, a fortiori pour du multi-agents sur systèmes IT critiques |
| **Créativité** | ✅ Fort | L'architecture L0-L4_SMS avec confiance dynamique, approbation inline SSE, bypass post-approbation et apprentissage du contact recommandé est une conception originale |
| **Incertitude** | ✅ Fort | Quatre verrous identifiés, dont le problème de cohérence distribuée qui a été rencontré et résolu expérimentalement (double blocage) |
| **Systématisation** | ✅ Moyen | Le bug double-blocage et sa résolution constituent une trace d'expérimentation documentée ; l'audit trail fournit des données |
| **Transférabilité** | ✅ Fort | Le framework est généralisable à tout contexte d'agents IA opérant sur des systèmes avec actions à risque (industrie, santé, finance) |

---

## Preuves à constituer

- État de l'art : frameworks de contrôle d'agents IA existants (LangChain guardrails, Microsoft AutoGen safety, etc.) et leurs limites
- Historique des itérations sur le modèle de niveaux (ajouts, modifications, le cas L4_SMS)
- Documentation du bug double-blocage : analyse, hypothèses testées, solution retenue
- Logs SAFEGUARD : demandes approuvées/rejetées, temps de réponse, taux d'utilisation par niveau
- Analyse des near-misses : cas où SAFEGUARD a bloqué une action potentiellement dangereuse
- Évolution du cache référent : pertinence des recommandations au fil du temps

---

---

# Synthèse des 3 opérations R&D

```
┌──────────────────────────────────────────────────────────────────────┐
│                     PLATEFORME IA WIDIP — 3 AXES R&D                │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  OR1 — CAPITALISATION AUTO          OR2 — QUALIFICATION AUTONOME    │
│  ┌─────────────────────┐            ┌─────────────────────┐         │
│  │ Enrichisseur 🟢     │            │ Assistant Ticket 🟢 │         │
│  │ RAG évolutif  🟡    │            │ Sentinel (déc.) 🟡  │         │
│  ├─────────────────────┤            ├─────────────────────┤         │
│  │ 3 verrous :         │            │ 4 verrous :         │         │
│  │ • Extraction texte  │            │ • Classification    │         │
│  │ • Qualité RAG       │            │ • Calibration conf. │         │
│  │ • Convergence boucle│            │ • Actions irrévers. │         │
│  └─────────────────────┘            │ • Décision Sentinel │         │
│                                     └─────────────────────┘         │
│                                                                      │
│              OR3 — GOUVERNANCE SÉCURITÉ                              │
│              ┌─────────────────────────────────┐                    │
│              │ SAFEGUARD 🟢                    │                    │
│              ├─────────────────────────────────┤                    │
│              │ 4 verrous :                     │                    │
│              │ • Modèle multi-niveaux          │                    │
│              │ • Cohérence distribuée           │                    │
│              │ • Orchestration chaînée          │                    │
│              │ • Confiance dynamique + L4_SMS   │                    │
│              └─────────────────────────────────┘                    │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│  TOTAL : 11 verrous techniques · 5 critères Frascati évalués       │
│  Composants éligibles : 5 sur 9 analysés                            │
└──────────────────────────────────────────────────────────────────────┘
```

---

# Évaluation globale Frascati — Vue croisée

| Critère | OR1 Capitalisation | OR2 Qualification | OR3 Gouvernance |
|---------|-------------------|-------------------|-----------------|
| **Nouveauté** | ✅ Fort | ✅ Fort | ✅ Fort |
| **Créativité** | ✅ Fort | ✅ Fort | ✅ Fort |
| **Incertitude** | ✅ Fort | ✅ Fort | ✅ Fort |
| **Systématisation** | ⚠️ À documenter | ⚠️ À documenter | ✅ Moyen |
| **Transférabilité** | ✅ Moyen | ✅ Fort | ✅ Fort |

---

# Verdict et conditions d'éligibilité

## Ce qui est fort

Les trois opérations R&D reposent sur des **problèmes de recherche légitimes** :
- La capitalisation automatique par boucle d'apprentissage (OR1) est un sujet ouvert
- La calibration de confiance d'un LLM pour des actions IT conséquentes (OR2) est un problème reconnu
- La gouvernance de sécurité multi-agents en production (OR3) est très peu documentée dans la littérature

## Ce qui conditionne l'éligibilité

Le **critère de systématisation** est le point faible actuel. Pour chaque opération, il faut constituer :
- Un état de l'art formel (publications, solutions existantes, leurs limites)
- Des hypothèses formalisées avant chaque expérimentation
- Des protocoles de test avec métriques mesurables
- Des résultats chiffrés (positifs ET négatifs)
- Un journal de bord R&D avec chronologie des itérations

## Risque principal en cas de contrôle

Un expert MESR pourrait argumenter que le projet "assemble des briques existantes" (Mistral API + pgvector + FastAPI + Redis). La clé de la défense est de démontrer que **les problèmes de recherche résident dans l'orchestration, la fiabilité et les boucles d'apprentissage** — pas dans les composants individuels.

## Recommandation

Le projet est **éligible sous condition** de produire la documentation R&D manquante. Les verrous techniques sont réels et les incertitudes documentables. La priorité est de transformer la documentation opérationnelle existante en documentation scientifique structurée.

---

# Prochaines étapes

1. **Constituer l'état de l'art** pour chaque opération (publications, solutions concurrentes)
2. **Formaliser les hypothèses** testées ou à tester pour chaque verrou
3. **Collecter les métriques** : logs, taux de réussite, benchmarks comparatifs
4. **Rédiger le dossier technique** CIR avec le template MESR pour chaque opération
5. **Préparer les preuves matérielles** : commits Git datés, logs d'expérimentation, comptes-rendus

---

*Analyse réalisée sur la base de CLAUDE_CONTEXT.md (v2.2.0) et ARCHITECTURE_FONCTIONNELLE_WIDIP.md (v2.0)*
*Référentiel : Manuel de Frascati (OCDE), Guide CIR 2025 (MESR), Loi de Finances 2025*
