# VALIDATION DES FLUX - Infographie WIDIP IA

Cette infographie comportera **3 onglets** avec les flux détaillés de chaque composant.

---

## ONGLET 1 : SENTINEL - Surveillance Proactive

### Flux Principal

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 1 : POLLING OBSERVIUM                          │
│                         (Toutes les 5 min)                              │
├─────────────────────────────────────────────────────────────────────────┤
│  L'orchestrateur Python interroge Observium                             │
│  → Récupère la liste des devices DOWN                                   │
│  → Extrait : hostname, IP, durée DOWN, sévérité                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 2 : TRACKING REDIS                             │
├─────────────────────────────────────────────────────────────────────────┤
│  Vérification du cache Redis :                                          │
│  • Device DOWN depuis > 20 min ?                                        │
│  • Alerte déjà envoyée pour ce device ?                                │
│                                                                         │
│  SI DOWN > 20 min ET pas encore alerté → Déclenche l'étape suivante    │
│  SINON → Ignore (micro-coupure ou déjà traité)                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 3 : APPEL MCP sms_alert_device_down            │
├─────────────────────────────────────────────────────────────────────────┤
│  L'agent SENTINEL traite l'alerte :                                     │
│                                                                         │
│  1. Parse le hostname → Extrait code client + site                      │
│     Exemple : "MFI-3A-SMH-SW01" → Client: MFI, Site: 3A (St-Martin)    │
│                                                                         │
│  2. Recherche dans GLPI l'entité la plus précise                       │
│     → Scoring par correspondance (exact > partiel > wildcard)           │
│                                                                         │
│  3. Récupère les contacts DirEnt de cette entité                       │
│     → Priorité : Référent technique > Directeur                        │
│                                                                         │
│  4. Crée le ticket GLPI avec le contexte complet                       │
│                                                                         │
│  5. Prépare l'aperçu SMS avec le N° de ticket                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 4 : BLOCAGE SAFEGUARD L4_SMS                   │
├─────────────────────────────────────────────────────────────────────────┤
│  L'outil retourne : SAFEGUARD_BLOCKED                                   │
│                                                                         │
│  • Demande de validation créée dans PostgreSQL                         │
│  • Arguments SMS chiffrés (AES-128) dans Redis                         │
│  • TTL : 1 heure                                                        │
│  • Le workflow continue (NON BLOQUANT pour le reste)                   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 5 : VALIDATION HUMAINE                         │
│                      (Dashboard SAFEGUARD)                              │
├─────────────────────────────────────────────────────────────────────────┤
│  Le technicien voit la demande en attente :                            │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  SAFEGUARD - Demande SMS                                        │   │
│  │                                                                  │   │
│  │  Client : MFI - Maison Familiale Isère                          │   │
│  │  Device : MFI-3A-SMH-SW01 (DOWN depuis 25 min)                  │   │
│  │  Ticket : #4894                                                 │   │
│  │                                                                  │   │
│  │  Contacts disponibles :                                         │   │
│  │  ○ Pierre Martin (Référent) - 06 12 ** ** **                   │   │
│  │  ○ Jean Dupont (Directeur)  - 06 98 ** ** **                   │   │
│  │                                                                  │   │
│  │  [ Sélectionner contact ]  [ Envoyer SMS ]  [ Rejeter ]        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  → Le technicien sélectionne le contact à notifier                     │
│  → Clique "Envoyer SMS"                                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 6 : ENVOI SMS (AllMySMS)                       │
├─────────────────────────────────────────────────────────────────────────┤
│  Appel sms_alert_send avec bypass SAFEGUARD :                          │
│                                                                         │
│  • SMS envoyé au contact choisi (1 seul destinataire)                  │
│  • Message : "[WIDIP] Incident détecté sur votre infrastructure.       │
│               Nos équipes interviennent. Ticket #4894."                │
│                                                                         │
│  • Cache référent mis à jour si choix différent du défaut             │
│    (mémorise la préférence pour ce client)                             │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 7 : MISE À JOUR TRACKER                        │
├─────────────────────────────────────────────────────────────────────────┤
│  Redis mis à jour :                                                     │
│  • alert_sms_sent = true pour ce device                                │
│  • Plus d'alerte SMS tant que le device reste DOWN                     │
│  • Si device revient UP puis re-DOWN → Nouveau cycle possible          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Points Clés à Illustrer

1. **Proactivité** : Le client est informé AVANT qu'il ne s'en aperçoive
2. **Validation humaine** : Un technicien valide toujours l'envoi SMS
3. **Choix du contact** : Le technicien choisit qui notifier
4. **Non-duplication** : Un seul SMS par incident (tracking Redis)

---

## ONGLET 2 : ENRICHISSEUR - Boucle d'Apprentissage

### Flux Principal

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 1 : DÉCLENCHEMENT                              │
│               (Batch quotidien - Fin de journée)                        │
├─────────────────────────────────────────────────────────────────────────┤
│  Le workflow s'exécute automatiquement chaque soir                      │
│  → Analyse les tickets clôturés de la journée                          │
│  → Focus sur les résolutions bien documentées                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 2 : COLLECTE DES TICKETS                       │
├─────────────────────────────────────────────────────────────────────────┤
│  Sources analysées :                                                    │
│                                                                         │
│  1. Tickets clôturés du jour                                           │
│     → Avec suivi technique détaillé                                    │
│                                                                         │
│  2. Tickets flaggés "IA_NON_RESOLU"                                    │
│     → Que l'IA n'avait pas su résoudre                                 │
│     → Maintenant résolus par un technicien                             │
│     → PRIORITAIRES pour l'apprentissage                                │
│                                                                         │
│  3. Tickets récurrents                                                 │
│     → Même type de problème, même solution                             │
│     → Pattern détecté                                                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 3 : ANALYSE PAR LE LLM                         │
├─────────────────────────────────────────────────────────────────────────┤
│  Pour chaque ticket sélectionné, le LLM :                              │
│                                                                         │
│  1. Extrait les étapes de résolution                                   │
│     Exemple ticket "Lenteur VPN client ADAPEI" :                       │
│     - Étape 1 : Vérifier la charge CPU du firewall                     │
│     - Étape 2 : Contrôler le nombre de tunnels actifs                  │
│     - Étape 3 : Redémarrer le service IPSec                            │
│     - Étape 4 : Tester avec traceroute depuis le site                  │
│                                                                         │
│  2. Identifie le pattern / catégorie                                   │
│     → "Diagnostic VPN / Performance"                                   │
│                                                                         │
│  3. Rédige une procédure structurée                                    │
│     → Titre, Symptômes, Prérequis, Étapes, Vérification                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 4 : GÉNÉRATION PROCÉDURE                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │  PROCÉDURE GÉNÉRÉE                                                │ │
│  │                                                                    │ │
│  │  Titre : Diagnostic lenteur VPN site distant                      │ │
│  │                                                                    │ │
│  │  Contexte :                                                       │ │
│  │  Utilisateurs d'un site distant signalent des lenteurs            │ │
│  │  lors de l'accès aux ressources centrales via VPN.                │ │
│  │                                                                    │ │
│  │  Symptômes :                                                      │ │
│  │  - Lenteur accès fichiers partagés                               │ │
│  │  - Timeout applications métier                                    │ │
│  │  - Latence élevée (> 200ms)                                      │ │
│  │                                                                    │ │
│  │  Étapes :                                                         │ │
│  │  1. Vérifier charge CPU firewall (< 80% normal)                  │ │
│  │  2. Contrôler tunnels VPN actifs (limite 50)                     │ │
│  │  3. Si saturation → Redémarrer service IPSec                     │ │
│  │  4. Tester avec traceroute depuis le site                        │ │
│  │                                                                    │ │
│  │  Vérification :                                                   │ │
│  │  Latence < 100ms après intervention                              │ │
│  │                                                                    │ │
│  │  Basée sur : 3 tickets similaires (#4521, #4678, #4812)          │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 5 : VALIDATION SAFEGUARD                       │
├─────────────────────────────────────────────────────────────────────────┤
│  Demande de validation au technicien/admin :                           │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │  NOUVELLE PROCÉDURE DÉTECTÉE                                      │ │
│  │                                                                    │ │
│  │  "Diagnostic lenteur VPN site distant"                            │ │
│  │                                                                    │ │
│  │  Basée sur 3 tickets similaires                                   │ │
│  │  Catégorie : Réseau / VPN / Troubleshooting                       │ │
│  │                                                                    │ │
│  │  [ Voir détail ]  [ Valider ]  [ Modifier ]  [ Rejeter ]         │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  → Le technicien peut modifier la procédure avant validation           │
│  → Une fois validée, elle est ajoutée à la base de connaissances      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 6 : ENRICHISSEMENT RAG                         │
├─────────────────────────────────────────────────────────────────────────┤
│  La procédure validée est :                                            │
│                                                                         │
│  1. Découpée en chunks (morceaux de texte)                             │
│  2. Vectorisée (transformée en vecteurs numériques)                    │
│  3. Stockée dans PostgreSQL + pgvector                                 │
│                                                                         │
│  → Immédiatement disponible pour tous les techniciens                  │
│  → WIBOT peut maintenant répondre sur ce sujet                         │
│  → L'IA est PLUS INTELLIGENTE qu'avant                                 │
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemple Concret Détaillé

```
══════════════════════════════════════════════════════════════════════════
  SCÉNARIO : Ticket complexe résolu manuellement
══════════════════════════════════════════════════════════════════════════

JOUR 1 - Le problème
─────────────────────
  • Client ADAPEI appelle : "Plus d'accès à l'imprimante réseau"
  • Le ticket est créé automatiquement
  • L'IA tente de résoudre → Échec (pas de procédure connue)
  • Ticket flaggé "IA_NON_RESOLU"

JOUR 1 - Résolution par le technicien
──────────────────────────────────────
  • Le technicien prend le ticket
  • Diagnostic : Problème de driver incompatible après mise à jour Windows
  • Solution appliquée :
    1. Supprimer l'imprimante existante
    2. Télécharger driver universel HP
    3. Réinstaller avec le driver universel
    4. Tester impression page de test
  • Ticket clôturé avec résolution documentée

JOUR 1 - Soir - L'Enrichisseur se déclenche
────────────────────────────────────────────
  • Détecte le ticket "IA_NON_RESOLU" maintenant résolu
  • Analyse la résolution du technicien
  • Génère la procédure :
    "Imprimante inaccessible après mise à jour Windows"
  • Soumet pour validation

JOUR 2 - Validation
────────────────────
  • Le technicien valide la procédure
  • Elle est ajoutée au RAG

JOUR 5 - Le même problème chez un autre client
───────────────────────────────────────────────
  • Client MFI appelle : "Plus d'accès à l'imprimante"
  • L'IA trouve la procédure dans le RAG
  • Propose la solution au technicien
  • Résolution en 2 minutes au lieu de 30

══════════════════════════════════════════════════════════════════════════
  RÉSULTAT : L'IA a APPRIS de l'expérience du technicien
══════════════════════════════════════════════════════════════════════════
```

### Points Clés à Illustrer

1. **Cercle vertueux** : Tickets → Procédures → IA plus intelligente → Meilleure résolution
2. **Apprentissage ciblé** : Focus sur les échecs de l'IA (IA_NON_RESOLU)
3. **Validation humaine** : Pas de procédure sans validation technicien
4. **Bénéfice immédiat** : Disponible pour toute l'équipe dès validation

---

## ONGLET 3 : WIBOT (Assistant IA) - Interaction Technicien

### Vue d'ensemble : Les 4 Modes

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         WIBOT - ASSISTANT IA                            │
│                    Interface conversationnelle                          │
├────────────────┬────────────────┬────────────────┬─────────────────────┤
│     FLASH      │      PRO       │      CODE      │       AGENT         │
│  ─────────────  │  ─────────────  │  ─────────────  │  ─────────────────  │
│  Questions     │  Analyses      │  Génération    │  Diagnostic         │
│  rapides       │  approfondies  │  de scripts    │  multi-sources      │
│                │                │                │                     │
│  Modèle léger  │  Modèle expert │  Modèle code   │  Orchestration      │
│  (Devstral     │  (Mistral      │  (Devstral)    │  outils + RAG       │
│   Small)       │   Large)       │                │                     │
└────────────────┴────────────────┴────────────────┴─────────────────────┘
```

### Flux Détaillé : Comment WIBOT répond à une question

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 1 : RÉCEPTION DE LA QUESTION                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Technicien : "Quelle est la procédure pour diagnostiquer une          │
│               lenteur VPN chez ADAPEI ?"                               │
│                                                                         │
│  → Mode sélectionné : PRO (analyse approfondie)                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 2 : RECHERCHE RAG                              │
│               (Base de Connaissances Centralisée)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. Vectorisation de la question                                       │
│     "lenteur VPN ADAPEI" → [0.23, -0.45, 0.12, ...]                    │
│                                                                         │
│  2. Recherche par similarité cosinus                                   │
│     → Parcourt toute la base de connaissances                          │
│                                                                         │
│  3. Résultats trouvés (top 5) :                                        │
│     ┌───────────────────────────────────────────────────────────────┐  │
│     │  Score 0.94 : "Diagnostic lenteur VPN site distant"           │  │
│     │  Score 0.87 : "Configuration VPN ADAPEI - Architecture"       │  │
│     │  Score 0.82 : "Troubleshooting réseau ADAPEI"                 │  │
│     │  Score 0.76 : "Ticket #4812 - Lenteur VPN ADAPEI"             │  │
│     │  Score 0.71 : "Procédure redémarrage firewall"                │  │
│     └───────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  → Contexte enrichi avec les 5 documents les plus pertinents          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 3 : APPEL OUTILS MCP (si nécessaire)          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Mode AGENT uniquement (ou si besoin de données temps réel) :          │
│                                                                         │
│  • MCP GLPI : Récupérer l'historique des tickets ADAPEI               │
│  • MCP Observium : Vérifier l'état actuel des équipements             │
│  • MCP AD : Consulter les infos utilisateurs si pertinent             │
│                                                                         │
│  Ces appels sont en LECTURE SEULE (L0) → Pas de validation requise    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 4 : GÉNÉRATION DE LA RÉPONSE                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Le LLM (Mistral Large) reçoit :                                       │
│  • La question du technicien                                           │
│  • Le contexte RAG (procédures trouvées)                               │
│  • Les données MCP (si mode Agent)                                     │
│  • L'historique de la conversation                                     │
│                                                                         │
│  → Génère une réponse structurée et sourcée                           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ÉTAPE 5 : RÉPONSE AU TECHNICIEN                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │  WIBOT :                                                          │ │
│  │                                                                    │ │
│  │  Pour diagnostiquer une lenteur VPN chez ADAPEI, voici la        │ │
│  │  procédure recommandée :                                          │ │
│  │                                                                    │ │
│  │  1. Vérifier la charge CPU du firewall Fortinet                  │ │
│  │     → Connexion : admin@fw-adapei.widip.local                    │ │
│  │     → Si CPU > 80%, c'est probablement la cause                  │ │
│  │                                                                    │ │
│  │  2. Contrôler le nombre de tunnels VPN actifs                    │ │
│  │     → Commande : get vpn ipsec tunnel summary                    │ │
│  │     → Limite normale : 50 tunnels                                │ │
│  │                                                                    │ │
│  │  3. Si saturation, redémarrer le service IPSec                   │ │
│  │     → Commande : diag vpn ike restart                            │ │
│  │                                                                    │ │
│  │  4. Tester avec traceroute depuis le site                        │ │
│  │     → Latence cible : < 100ms                                    │ │
│  │                                                                    │ │
│  │  Sources : [Procédure #P-2024-089] [Ticket #4812]                │ │
│  │                                                                    │ │
│  │  Note : J'ai détecté 2 incidents similaires résolus le mois      │ │
│  │  dernier. Voulez-vous que je consulte Observium pour l'état      │ │
│  │  actuel du firewall ADAPEI ?                                      │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Flux : Exécution d'une action avec SAFEGUARD

```
┌─────────────────────────────────────────────────────────────────────────┐
│  SCÉNARIO : Le technicien demande une action sensible                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Technicien : "Crée un ticket pour le client ADAPEI avec le           │
│               diagnostic que tu viens de faire"                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    DÉTECTION NIVEAU SAFEGUARD                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Action demandée : glpi_create_ticket                                  │
│  Niveau : L1 (MINOR)                                                   │
│                                                                         │
│  Règle L1 : Exécution auto si confiance ≥ 80%                         │
│  → Le contexte est clair, client identifié → Confiance 95%             │
│  → Action AUTORISÉE automatiquement                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    EXÉCUTION + NOTIFICATION                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │  WIBOT :                                                          │ │
│  │                                                                    │ │
│  │  J'ai créé le ticket #4925 pour ADAPEI :                         │ │
│  │                                                                    │ │
│  │  Titre : Diagnostic lenteur VPN - Site Grenoble                  │ │
│  │  Contenu : [Résumé du diagnostic effectué]                       │ │
│  │  Urgence : Moyenne                                                │ │
│  │                                                                    │ │
│  │  Le ticket est assigné à votre équipe.                           │ │
│  │                                                                    │ │
│  │  [Voir le ticket dans GLPI]                                      │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  → Une notification est également envoyée au technicien               │
│    (traçabilité de l'action IA)                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Flux : Action bloquée par SAFEGUARD

```
┌─────────────────────────────────────────────────────────────────────────┐
│  SCÉNARIO : Le technicien demande une action très sensible             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Technicien : "Envoie un SMS au référent ADAPEI pour le prévenir      │
│               du problème"                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    DÉTECTION NIVEAU SAFEGUARD                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Action demandée : send_sms                                            │
│  Niveau : L4 (CRITICAL)                                                │
│                                                                         │
│  Règle L4 : TOUJOURS BLOQUÉ - Validation obligatoire                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    DEMANDE DE VALIDATION                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │  WIBOT :                                                          │ │
│  │                                                                    │ │
│  │  Cette action nécessite votre validation explicite.              │ │
│  │                                                                    │ │
│  │  ┌─────────────────────────────────────────────────────────────┐ │ │
│  │  │  SAFEGUARD - Validation requise                             │ │ │
│  │  │                                                              │ │ │
│  │  │  Action : Envoi SMS                                         │ │ │
│  │  │  Destinataire : Pierre Martin (Référent ADAPEI)             │ │ │
│  │  │  Téléphone : 06 12 ** ** **                                 │ │ │
│  │  │  Message : "[WIDIP] Diagnostic en cours sur votre VPN..."   │ │ │
│  │  │                                                              │ │ │
│  │  │  [ Confirmer l'envoi ]  [ Modifier ]  [ Annuler ]          │ │ │
│  │  └─────────────────────────────────────────────────────────────┘ │ │
│  │                                                                    │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  → Le technicien DOIT cliquer "Confirmer" pour que le SMS parte       │
│  → Sans validation, l'action expire après 1 heure                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Points Clés à Illustrer

1. **RAG centralisé** : Une seule source de vérité pour toute l'équipe
2. **Recherche sémantique** : Trouve par le sens, pas par mots-clés exacts
3. **Sources citées** : Toujours traçable, jamais d'hallucination
4. **SAFEGUARD intégré** : Protection automatique sur les actions sensibles
5. **4 modes adaptés** : Flash (rapide), Pro (expert), Code (scripts), Agent (diagnostic)

---

## RÉSUMÉ POUR L'INFOGRAPHIE

| Onglet | Composant | Message Principal |
|--------|-----------|-------------------|
| 1 | SENTINEL | "Le client est informé AVANT qu'il ne s'en aperçoive" |
| 2 | ENRICHISSEUR | "L'IA apprend de l'expérience des techniciens" |
| 3 | WIBOT | "Toute la connaissance de l'entreprise, en une question" |

### Fil Rouge : SAFEGUARD

Dans les 3 onglets, mettre en avant que SAFEGUARD protège toutes les actions sensibles :
- SENTINEL : Validation SMS avant envoi au client
- ENRICHISSEUR : Validation procédure avant ajout au RAG
- WIBOT : Validation actions L3/L4 avant exécution

---

## PRÊT POUR VALIDATION

Ce document décrit les flux à illustrer dans l'infographie à 3 onglets.

À valider avant de passer à la création HTML :
- [ ] Flux SENTINEL OK ?
- [ ] Flux ENRICHISSEUR OK (avec l'exemple complexe) ?
- [ ] Flux WIBOT OK (avec interaction SAFEGUARD) ?
