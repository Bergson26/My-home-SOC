# 🛡️ Projet : My-Home-SOC (Wazuh SIEM/XDR)

## 🎯 Objectif du Projet
L'objectif est de mettre en place une infrastructure de **SOC (Security Operations Center)** domestique pour centraliser les logs, détecter les vulnérabilités et répondre aux menaces en temps réel. Ce projet repose sur la stack **Wazuh** (Indexer, Manager, Dashboard) déployée dans un environnement conteneurisé Docker sous Windows (via WSL 2).

---

## 🏗️ Architecture du Système
Le SOC est composé de trois briques fondamentales communiquant sur un réseau virtuel privé :
* **Wazuh Indexer :** Cluster de stockage et d'indexation basé sur OpenSearch.
* **Wazuh Manager :** Cœur de l'analyse, gère les alertes et les politiques de sécurité.
* **Wazuh Dashboard :** Interface de visualisation et d'investigation.

---

## 🛠️ Guide de Déploiement

### 1. Prérequis Système (Windows / WSL 2)
L'indexer nécessite une configuration spécifique de la mémoire virtuelle pour démarrer correctement les services Java :
```powershell
wsl -d docker-desktop -u root sysctl -w vm.max_map_count=262144

2. Lancement de la Stack
Le projet utilise la version 4.7.1 pour garantir une stabilité optimale sur les environnements virtualisés.

PowerShell
docker-compose up -d

3. Initialisation Manuelle de la Sécurité
Si le Dashboard affiche Not ready yet, il est nécessaire de forcer l'initialisation du cluster de sécurité (création de l'index .opendistro_security) :

PowerShell
docker exec -it wazuh.indexer bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -cd /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/ -icl -nhnv -cacert /usr/share/wazuh-indexer/certs/root-ca.pem -cert /usr/share/wazuh-indexer/certs/admin.pem -key /usr/share/wazuh-indexer/certs/admin-key.pem -h localhost

Focus technique : Résolution de noms DNS
Le Dashboard ne parvenait pas à joindre l'Indexer. L'analyse des logs a révélé que le Dashboard cherchait l'hôte wazuh.indexer (convention interne Wazuh) alors que Docker Desktop créait par défaut l'hôte wazuh-indexer.
Résolution : Forçage du paramètre container_name et mise à jour des variables d'environnement INDEXER_URL dans le fichier docker-compose.yml.

🔐 Accès et Administration
Interface Web : https://localhost

Identifiants par défaut : admin / SecretPassword123!

Vérification de l'état : docker ps (doit afficher 3 conteneurs "Up")

📈 Compétences Validées
Orchestration et déploiement via Docker Compose.

Administration système Linux/Bash en environnement conteneurisé.

Troubleshooting réseau avancé (DNS interne, Ports, Protocoles TLS/SSL).

Analyse de logs applicatifs complexes pour la résolution d'incidents.

Configuration de solutions SIEM / XDR (Wazuh).
