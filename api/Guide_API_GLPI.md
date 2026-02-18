# Guide API REST GLPI

## Introduction

GLPI dispose d'une API REST externe depuis la version 9.1, permettant d'interagir avec l'application via des requêtes HTTP. Cette API permet de gérer les assets, tickets, utilisateurs et autres objets GLPI de manière programmatique.

> ⚠️ **Note** : L'API REST est désactivée par défaut pour des raisons de sécurité.

---

## 1. Activation de l'API

### Via l'interface GLPI

1. Aller dans **Configuration > Générale > API**
2. Activer l'API REST en mettant le paramètre sur **OUI**
3. Configurer les méthodes d'authentification autorisées :
   - **Identifiants** : permet l'utilisation login/mot de passe
   - **Token externe** : permet l'utilisation du token personnel utilisateur

### URL d'accès

L'API est accessible à l'adresse :
```
http://votre-serveur/glpi/apirest.php
```

> 💡 En accédant à cette URL via un navigateur, vous obtiendrez la documentation intégrée de l'API.

---

## 2. Configuration des clients API

Dans l'onglet **Configuration > Générale > API**, vous pouvez gérer les clients API :

| Paramètre | Description |
|-----------|-------------|
| **Nom** | Nom du client API |
| **Actif** | Activer/désactiver le client |
| **Journalisation** | Enregistrer les connexions (historique, logs, aucun) |
| **App-Token** | Token d'application (optionnel, couche de sécurité supplémentaire) |
| **Plage IP** | Restreindre l'accès à certaines adresses IP |

---

## 3. Authentification

L'API GLPI utilise un système d'authentification en deux étapes :
1. Obtenir un **Session-Token** via l'endpoint `initSession`
2. Utiliser ce token pour toutes les requêtes suivantes

### 3.1 Méthode 1 : Login / Mot de passe (HTTP Basic Auth)

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $(echo -n 'login:password' | base64)" \
  -H "App-Token: VOTRE_APP_TOKEN" \
  "http://glpi.example.com/apirest.php/initSession"
```

### 3.2 Méthode 2 : User Token

Le token utilisateur se trouve dans les préférences utilisateur (onglet "Accès distant").

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Authorization: user_token VOTRE_USER_TOKEN" \
  -H "App-Token: VOTRE_APP_TOKEN" \
  "http://glpi.example.com/apirest.php/initSession"
```

### 3.3 Réponse d'authentification

```json
{
  "session_token": "83af7e620c83a50a18d3eac2f6ed05a3ca0bea62"
}
```

> ⚠️ **Important** : Conservez ce `session_token` pour toutes vos requêtes suivantes.

---

## 4. Structure des requêtes

### Headers requis

| Header | Description | Obligatoire |
|--------|-------------|-------------|
| `Content-Type` | `application/json` | Oui |
| `Session-Token` | Token obtenu via `initSession` | Oui (sauf initSession) |
| `App-Token` | Token d'application | Selon configuration |

### Exemple de requête authentifiée

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: VOTRE_APP_TOKEN" \
  "http://glpi.example.com/apirest.php/Computer"
```

---

## 5. Endpoints principaux

### 5.1 Gestion de session

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/initSession` | GET | Initialiser une session et obtenir un token |
| `/killSession` | GET | Terminer la session |
| `/getFullSession` | GET | Récupérer toutes les données de session |
| `/getMyProfiles` | GET | Liste des profils de l'utilisateur |
| `/getActiveProfile` | GET | Profil actif |
| `/changeActiveProfile` | POST | Changer de profil |
| `/getMyEntities` | GET | Liste des entités accessibles |
| `/getActiveEntities` | GET | Entité active |
| `/changeActiveEntities` | POST | Changer d'entité |

### 5.2 Opérations CRUD

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/:itemtype` | GET | Liste tous les éléments d'un type |
| `/:itemtype/:id` | GET | Récupère un élément spécifique |
| `/:itemtype` | POST | Crée un nouvel élément |
| `/:itemtype/:id` | PUT | Met à jour un élément |
| `/:itemtype/:id` | DELETE | Supprime un élément |

### 5.3 Recherche

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/search/:itemtype` | GET | Recherche avec critères |
| `/listSearchOptions/:itemtype` | GET | Liste les options de recherche |

### 5.4 Autres endpoints

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/getMultipleItems` | GET | Récupère plusieurs éléments de types différents |
| `/getGlpiConfig` | GET | Configuration GLPI |
| `/listItemtypes` | GET | Liste tous les types d'objets disponibles |

---

## 6. Types d'objets (Itemtypes)

Voici les principaux itemtypes disponibles :

| Itemtype | Description |
|----------|-------------|
| `Computer` | Ordinateurs |
| `Monitor` | Écrans |
| `Printer` | Imprimantes |
| `NetworkEquipment` | Équipements réseau |
| `Phone` | Téléphones |
| `Peripheral` | Périphériques |
| `Software` | Logiciels |
| `Ticket` | Tickets |
| `User` | Utilisateurs |
| `Group` | Groupes |
| `Entity` | Entités |
| `Location` | Lieux |
| `Supplier` | Fournisseurs |
| `Contract` | Contrats |
| `Document` | Documents |

---

## 7. Exemples pratiques

### 7.1 Initialiser une session

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Authorization: user_token q56hqkniwot8wntb3z1qarka5atf365taaa2uyjrn" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  "http://glpi.example.com/apirest.php/initSession"
```

### 7.2 Lister les ordinateurs

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  "http://glpi.example.com/apirest.php/Computer"
```

### 7.3 Récupérer un ordinateur spécifique

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  "http://glpi.example.com/apirest.php/Computer/1"
```

### 7.4 Créer un ticket

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  -d '{"input": {"name": "Mon ticket", "content": "Description du problème", "urgency": 3}}' \
  "http://glpi.example.com/apirest.php/Ticket"
```

**Réponse** :
```json
{
  "id": 42,
  "message": "Item successfully added: Mon ticket"
}
```

### 7.5 Mettre à jour un élément

```bash
curl -X PUT \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  -d '{"input": {"name": "Nouveau nom"}}' \
  "http://glpi.example.com/apirest.php/Computer/1"
```

### 7.6 Supprimer un élément

```bash
curl -X DELETE \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  "http://glpi.example.com/apirest.php/Computer/1"
```

### 7.7 Recherche avancée

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  "http://glpi.example.com/apirest.php/search/Computer?criteria[0][field]=1&criteria[0][searchtype]=contains&criteria[0][value]=PC"
```

### 7.8 Fermer la session

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Session-Token: 83af7e620c83a50a18d3eac2f6ed05a3ca0bea62" \
  -H "App-Token: f7g3csp8mgatg5ebc5elnazakw20i9fyev1qopya7" \
  "http://glpi.example.com/apirest.php/killSession"
```

---

## 8. Options des requêtes GET

### 8.1 Paramètres pour /:itemtype/:id

| Paramètre | Description |
|-----------|-------------|
| `with_devices` | Récupérer les composants (Computer, NetworkEquipment, etc.) |
| `with_disks` | Récupérer les systèmes de fichiers (Computer uniquement) |
| `with_softwares` | Récupérer les logiciels installés (Computer uniquement) |
| `with_connections` | Récupérer les connexions directes |
| `with_networkports` | Récupérer les informations réseau |
| `with_infocoms` | Récupérer les informations financières |
| `with_contracts` | Récupérer les contrats associés |
| `with_documents` | Récupérer les documents associés |
| `with_logs` | Récupérer l'historique |
| `with_notes` | Récupérer les notes |

### 8.2 Pagination

| Paramètre | Description |
|-----------|-------------|
| `range` | Plage de résultats (ex: `0-49` pour les 50 premiers) |

**Codes de retour pour la pagination** :
- `200 OK` : Tous les résultats retournés
- `206 Partial Content` : Résultats partiels (pagination nécessaire)

---

## 9. Codes d'erreur

| Code | Message | Description |
|------|---------|-------------|
| 400 | Bad Request | Paramètres invalides |
| 401 | Unauthorized | Authentification requise ou échouée |
| 403 | Forbidden | Droits insuffisants |
| 404 | Not Found | Ressource non trouvée |
| 405 | Method Not Allowed | Méthode HTTP non autorisée |
| 500 | Internal Server Error | Erreur serveur |

### Messages d'erreur courants

| Erreur | Description |
|--------|-------------|
| `ERROR_SESSION_TOKEN_INVALID` | Token de session invalide ou expiré |
| `ERROR_SESSION_TOKEN_MISSING` | Header Session-Token manquant |
| `ERROR_APP_TOKEN_PARAMETERS_MISSING` | App-Token requis mais manquant |
| `ERROR_WRONG_APP_TOKEN_PARAMETER` | App-Token incorrect |
| `ERROR_NOT_ALLOWED_IP` | IP non autorisée |
| `ERROR_RIGHT_MISSING` | Droits insuffisants |
| `ERROR_ITEM_NOT_FOUND` | Élément non trouvé |
| `ERROR_RANGE_EXCEED_TOTAL` | Plage de pagination dépassée |

---

## 10. Bonnes pratiques

1. **Toujours fermer la session** : Utilisez `killSession` à la fin de vos scripts
2. **Gérer les erreurs** : Vérifiez les codes de retour HTTP
3. **Utiliser HTTPS** : En production, sécurisez vos échanges
4. **Limiter les droits** : Créez des utilisateurs API avec les droits minimum nécessaires
5. **Filtrer par IP** : Restreignez l'accès API aux IP autorisées
6. **Paginer les résultats** : Pour les grandes quantités de données

---

## 11. GLPI 11.0 - High-Level API (Nouvelle API)

GLPI 11.0 introduit une nouvelle API appelée **High-Level API** (HL API), plus moderne et basée sur OpenAPI 3. Cette nouvelle API coexiste avec l'API REST legacy.

### Caractéristiques principales

- Schémas basés sur OpenAPI 3
- Documentation Swagger UI intégrée
- Versioning des endpoints
- Système de permissions amélioré

> 📖 Pour plus de détails sur la High-Level API, consultez la documentation développeur GLPI.

---

## Ressources

- Documentation API intégrée : `http://votre-glpi/apirest.php`
- Documentation GLPI : https://glpi-user-documentation.readthedocs.io/
- GitHub GLPI : https://github.com/glpi-project/glpi
