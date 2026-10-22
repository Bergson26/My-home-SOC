# 🛡️ My-Home-SOC — Lab de supervision Wazuh / OpenSearch

Infrastructure SOC domestique pour centraliser les logs, détecter les anomalies
et s'entraîner à l'investigation d'incidents.
Stack : Wazuh (Indexer, Manager, Dashboard) conteneurisée via Docker sous Windows / WSL 2.

---

## 🏗️ Architecture

┌─────────────────────────────────────────────┐
│                Réseau Docker                │
│                                             │
│  ┌───────────────┐    ┌──────────────────┐  │
│  │ Wazuh Manager │───▶│  Wazuh Indexer   │  │
│  │ (alertes,     │    │  (OpenSearch,    │  │
│  │  politiques)  │    │   stockage logs) │  │
│  └──────┬────────┘    └────────┬─────────┘  │
│         │                     │             │
│         └──────────┬──────────┘             │
│                    ▼                        │
│          ┌──────────────────┐               │
│          │ Wazuh Dashboard  │               │
│          │  (https :5601)   │               │
│          └──────────────────┘               │
└─────────────────────────────────────────────┘
▲
Agents Wazuh (machines surveillées)



---

## 🛠️ Déploiement

### Prérequis — WSL 2

L'Indexer nécessite une configuration mémoire spécifique pour les services Java :

```bash
wsl -d docker-desktop -u root sysctl -w vm.max_map_count=262144
Lancement (version 4.7.1)

docker-compose up -d
Si le Dashboard affiche "Not ready yet"
Forcer l'initialisation du cluster de sécurité :


docker exec -it wazuh.indexer bash \
  /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh \
  -cd /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/ \
  -icl -nhnv \
  -cacert /usr/share/wazuh-indexer/certs/root-ca.pem \
  -cert /usr/share/wazuh-indexer/certs/admin.pem \
  -key /usr/share/wazuh-indexer/certs/admin-key.pem \
  -h localhost
Vérifier l'état :


docker ps   # doit afficher 3 conteneurs "Up"
Interface web : https://localhost — identifiants : voir la documentation officielle Wazuh

🔧 Incident résolu — DNS interne
Symptôme : le Dashboard ne parvenait pas à joindre l'Indexer.

Diagnostic : les logs révèlent que le Dashboard cherchait wazuh.indexer
(convention interne Wazuh) alors que Docker Desktop créait wazuh-indexer.

Résolution : forçage du paramètre container_name et mise à jour de
INDEXER_URL dans docker-compose.yml.

📊 Résultats
3 règles de détection personnalisées créées (brute-force SSH, scan de ports, connexion hors plage horaire)
Alertes corrélées et qualifiées quotidiennement pendant la durée du projet
Incident DNS interne détecté, diagnostiqué et résolu par analyse des logs
⚠️ Limites
Lab isolé : aucun agent sur des machines de production
Pas de SOAR connecté — la réponse aux incidents reste manuelle
Single node : pas de haute disponibilité
Volume de logs limité à l'environnement local
