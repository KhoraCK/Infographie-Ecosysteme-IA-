# Guide API Observium

## Introduction

Observium propose une API REST/JSON permettant d'accéder aux données stockées dans sa base MySQL. Cette API est disponible uniquement dans l'**édition Subscription** d'Observium.

> ⚠️ **Note** : L'API est actuellement en développement actif. Les spécifications peuvent évoluer.

---

## 1. Activation de l'API

Pour activer l'API, ajoutez la ligne suivante dans le fichier `config.php` d'Observium :

```php
$config['api']['enable'] = TRUE;
```

---

## 2. Authentification

L'API utilise l'authentification HTTP Basic. Elle s'appuie sur les utilisateurs Observium existants, ce qui permet de fonctionner avec LDAP, RADIUS ou la base MySQL interne.

### Exemple avec cURL

```bash
curl -u <username>:<password> http://observium.domain.com/api/v0/devices/
```

---

## 3. Structure de l'API

L'URL de base suit ce format :

```
/api/<version>/<entity>/<entity_id>/<query>
```

| Élément | Description |
|---------|-------------|
| `version` | Version de l'API (ex: `v0`) |
| `entity` | Type d'entité (devices, ports, alerts, etc.) |
| `entity_id` | Identifiant de l'entité (optionnel) |
| `query` | Paramètres de filtrage (optionnel) |

---

## 4. Requêtes GET (Lecture)

### 4.1 Exemples de requêtes

| Action | Requête |
|--------|---------|
| Liste de tous les appareils | `GET /api/v0/devices/` |
| Détails d'un appareil par ID | `GET /api/v0/devices/1/` |
| Détails d'un appareil par hostname | `GET /api/v0/devices/test.company.com/` |
| Filtrer par OS | `GET /api/v0/devices/?os=ios` |

### 4.2 Endpoints disponibles

| Endpoint | Description | Options de filtrage |
|----------|-------------|---------------------|
| `/alerts/` | Alertes | `device_id`, `status`, `entity_type`, `entity_id` |
| `/alert_checks/` | Vérifications d'alertes | - |
| `/address/` | Adresses IPv4/IPv6 | `af` (ipv4/ipv6), `device_id`, `port_id` |
| `/bills/` | Factures | - |
| `/counters/` | Compteurs | `device_id`, `counter_class`, `counter_event` |
| `/devices/` | Appareils | `group`, `location`, `os`, `version`, `status` |
| `/entity/<type>/<id>/` | Entité générique | - |
| `/inventory/` | Inventaire physique | `device_id`, `entPhysicalClass`, `deleted` |
| `/mempools/` | Mémoire | `device_id`, `mempool_descr` |
| `/neighbours/` | Voisins réseau | `device_id`, `protocol`, `active` |
| `/ports/` | Ports | `device_id`, `ifType`, `state`, `errors` |
| `/printersupplies/` | Consommables imprimantes | `device_id`, `supply_type` |
| `/processors/` | Processeurs | `device_id`, `processor_descr` |
| `/sensors/` | Capteurs | `device_id`, `sensor_type`, `event` |
| `/status/` | Statuts | `device_id`, `class`, `event` |
| `/storage/` | Stockage | `device_id`, `storage_descr` |

### 4.3 Filtrage des champs

Utilisez le paramètre `fields` pour limiter les champs retournés :

```bash
GET /api/v0/ports/?fields=port_id,port_label
```

### 4.4 Pagination

Utilisez `pagesize` et `pageno` pour paginer les résultats :

```bash
GET /api/v0/devices/?pagesize=10&pageno=2
```

---

## 5. Gestion des Appareils

### 5.1 Ajouter un appareil (POST)

```bash
curl -u user:pass http://localhost/api/v0/devices/ \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"hostname":"localhost"}'
```

**Réponse** :
```json
{"status":"ok","device_id":1}
```

#### Options disponibles

| Option | Description |
|--------|-------------|
| `hostname` | Hostname ou adresse IP (obligatoire) |
| `poller_id` | ID du poller distribué (Enterprise) |
| `ping_skip` | Ignorer le test ping (`on`/`off`) |
| `snmp_version` | Version SNMP (`v1`, `v2c`, `v3`) |
| `snmp_port` | Port SNMP (défaut: 161) |
| `snmp_timeout` | Timeout SNMP (défaut: 1s) |
| `snmp_retries` | Nombre de tentatives (défaut: 5) |
| `snmp_community` | Communauté SNMP (v1/v2c) |

**Options SNMPv3** :

| Option | Description |
|--------|-------------|
| `snmp_authlevel` | Niveau d'auth (`noAuthNoPriv`/`authNoPriv`/`authPriv`) |
| `snmp_authname` | Nom d'utilisateur |
| `snmp_authpass` | Mot de passe auth |
| `snmp_authalgo` | Algorithme auth (`md5`/`sha`/`sha-256`...) |
| `snmp_cryptopass` | Mot de passe chiffrement |
| `snmp_cryptoalgo` | Algorithme chiffrement (`des`/`aes`/`aes-256`...) |

### 5.2 Supprimer un appareil (DELETE)

```bash
curl -u user:pass http://observium/api/v0/devices/491/ -X DELETE
```

**Réponse** :
```json
{"status":"ok","message":"Device Deleted","output":"..."}
```

### 5.3 Modifier un appareil (PUT)

```bash
curl -u user:pass -X PUT http://observium/api/v0/devices/47/ \
  -H "Content-Type: application/json" \
  -d '{"disabled":"1"}'
```

#### Options modifiables

| Option | Description |
|--------|-------------|
| `ignore` | Ignorer pour les alertes (`0`/`1`) |
| `ignore_until` | Ignorer jusqu'à une date (`Y-m-d H:i:s`) |
| `disabled` | Désactiver le polling (`0`/`1`) |
| `purpose` | Description de l'appareil |

---

## 6. Gestion des Alertes

### Ignorer une alerte (PUT)

```bash
curl -u user:pass -X PUT http://observium/api/v0/alerts/48467/ \
  -H "Content-Type: application/json" \
  -d '{"ignore_until_ok":"1", "ignore_until":"2017-05-13 12:34:20"}'
```

| Option | Description |
|--------|-------------|
| `ignore_until_ok` | Ignorer jusqu'à OK (`0`/`1`) |
| `ignore_until` | Ignorer jusqu'à une date précise |

---

## 7. Exemple de réponse JSON

```json
{
  "status": "ok",
  "count": 2,
  "devices": {
    "277": {
      "device_id": "277",
      "hostname": "router-1.company.com",
      "os": "ios",
      "version": "15.0(1)M4",
      "hardware": "CISCO1841"
    },
    "278": {
      "device_id": "278",
      "hostname": "router-b.company.com",
      "os": "ios",
      "version": "15.0(1)M4",
      "hardware": "CISCO1841"
    }
  }
}
```

---

## 8. Résumé des méthodes HTTP

| Méthode | Action | Exemple |
|---------|--------|---------|
| `GET` | Lire des données | Lister les appareils |
| `POST` | Créer une ressource | Ajouter un appareil |
| `PUT` | Modifier une ressource | Modifier un appareil |
| `DELETE` | Supprimer une ressource | Supprimer un appareil |

---

## 9. Bonnes pratiques

1. **Sécurité** : Utilisez HTTPS en production pour protéger les identifiants
2. **Permissions** : L'API respecte les droits de l'utilisateur authentifié
3. **Pagination** : Utilisez-la pour les grandes quantités de données
4. **Filtrage** : Limitez les champs retournés avec `fields` pour améliorer les performances

---

## Ressources

- Documentation officielle : https://docs.observium.org/api/
- API pour les graphiques : https://docs.observium.org/graph_api/
- Communauté IRC pour les questions de développement
