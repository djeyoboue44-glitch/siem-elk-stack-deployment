
<div align="center">

# 🛡️ SIEM ELK Stack  Supervision & Détection

### Centralisation de logs, détection d'intrusion et tableaux de bord de sécurité

![Elasticsearch](https://img.shields.io/badge/Elasticsearch-7.17.29-005571?style=for-the-badge&logo=elasticsearch&logoColor=white)
![Kibana](https://img.shields.io/badge/Kibana-7.17.29-005571?style=for-the-badge&logo=kibana&logoColor=white)
![Filebeat](https://img.shields.io/badge/Filebeat-7.17.29-005571?style=for-the-badge&logo=elastic&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Kibana-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-Elasticsearch_Host-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-Filebeat_Agent-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

</div>

---

## 📖 Description

Projet personnel de mise en place d'un **SIEM basé sur la stack ELK** (Elasticsearch, Filebeat, Kibana), avec **Elasticsearch installé sur Kali Linux**, **Kibana déployé via Docker**, et **Filebeat installé sur une machine Ubuntu distante** pour la collecte de logs système (`syslog`, `auth`). Le projet inclut la configuration du module Security, la construction de tableaux de bord de supervision, et un scénario de test réel : une **simulation d'attaque par force brute SSH** détectée dans les logs.

---

## 🎯 Objectifs

- 📥 Centraliser les logs système d'une machine distante (Ubuntu) vers un cluster Elasticsearch
- 📊 Explorer et analyser les logs en temps réel via Kibana Discover
- 🔐 Configurer le module Security de Kibana (détection engine, clés de chiffrement)
- 🧪 Simuler une attaque par force brute SSH et valider sa détection dans les logs
- 📈 Construire un tableau de bord de supervision avec plusieurs visualisations

---

## 🧰 Environnement

| Machine | Rôle |
|---|---|
| 🐉 **Kali Linux** | Hôte Elasticsearch + Kibana (Docker) |
| 🟠 **Ubuntu** | Agent Filebeat, cible des logs système et des tests SSH |

---

## 🏗️ Architecture

```
        Ubuntu (agent Filebeat)
        ├── /var/log/syslog
        └── /var/log/auth.log
                    │
                    │  Filebeat (module system)
                    ▼
          Elasticsearch (Kali, port 9200)
                    │
                    ▼
           Kibana (Docker, port 5601)
                    │
        ┌───────────┼───────────┐
    Discover     Security     Dashboard
   (recherche   (détection    (visualisations
    KQL/logs)    d'événements)  de supervision)
```

---

## 🧪 Méthodologie

<details>
<summary><b>1️⃣ Installation d'Elasticsearch (Kali Linux)</b></summary>

- Ajout du dépôt officiel Elastic et installation via `apt`
- Vérification de la version active via `curl http://localhost:9200`

</details>

<details>
<summary><b>2️⃣ Déploiement de Kibana (Docker)</b></summary>

- Lancement du conteneur Kibana lié à Elasticsearch (`docker run --link elasticsearch:elasticsearch`)
- Accès à l'interface via `http://localhost:5601`

</details>

<details>
<summary><b>3️⃣ Installation et configuration de Filebeat (Ubuntu)</b></summary>

- Installation de Filebeat depuis le dépôt Elastic officiel
- Activation du module `system` (collecte `syslog` et `auth.log`)
- Configuration de la sortie vers Elasticsearch et démarrage du service

</details>

<details>
<summary><b>4️⃣ Exploration des logs dans Kibana</b></summary>

- Création de l'index pattern `filebeat-*` (6 547 champs détectés)
- Analyse des logs en temps réel via **Discover**
- Recherche KQL ciblée (`event.dataset: "system.syslog" AND message: "sudo"`)

</details>

<details>
<summary><b>5️⃣ Configuration du module Security</b></summary>

- Génération des clés de chiffrement Kibana (`kibana-encryption-keys generate`)
- Redéploiement du conteneur Kibana avec les clés d'encryption
- Vérification de la section **Security → Overview** (7 236 événements détectés)

</details>

<details>
<summary><b>6️⃣ Simulation d'une attaque force brute SSH</b></summary>

- Installation et démarrage du service SSH sur la machine cible Ubuntu
- Script de simulation d'attaque : boucle de connexions SSH échouées
- Vérification de la remontée des tentatives dans les logs `auth.log` via Filebeat/Kibana

</details>

<details>
<summary><b>7️⃣ Construction du tableau de bord</b></summary>

- Création de visualisations (histogramme temporel, répartition par hostname, métriques)
- Assemblage des visualisations dans un dashboard Kibana de supervision des logs

</details>

---

## 🚀 Commandes clés

```bash
# Installation Elasticsearch (Kali)
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update && sudo apt install elasticsearch -y

# Déploiement Kibana (Docker)
sudo docker run -d --name kibana --link elasticsearch:elasticsearch -p 5601:5601 docker.elastic.co/kibana/kibana:7.17.29

# Installation Filebeat (Ubuntu)
sudo apt install filebeat -y
sudo filebeat modules enable system
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["http://<IP_KALI>:9200"]'
sudo systemctl enable --now filebeat

# Génération des clés de chiffrement Kibana
sudo docker exec -it kibana bash -c "bin/kibana-encryption-keys generate"

# Simulation d'attaque brute force SSH
for i in {1..10}; do ssh wronguser@<IP_CIBLE>; done
```

---

## 🖼️ Aperçu du projet

<div align="center">

### ⚙️ Installation et déploiement de la stack

</div>

| | |
|---|---|
| ![1](1-INSTALLATION%20ELASTICSEARCH%20SUR%20KALI.PNG) **Installation Elasticsearch sur Kali** | ![2](2-ELASTICSEARCH%20VERSION%207.17.29%20DOCKER%20KIBANA.PNG) **Elasticsearch 7.17.29 & Kibana Docker** |
| ![3](3-INSTALLATION%20FILEBEAT%20SUR%20UBUNTU.PNG) **Installation Filebeat sur Ubuntu** | ![4](4-KIBANA%20OPERATIONEL%20SUR%20LOCALHOST5601.PNG) **Kibana opérationnel (localhost:5601)** |
| ![5](5-CONFIGURATION%20ET%20DEMARRAGE%20FILEBEAT.PNG) **Configuration et démarrage Filebeat** | ![6](6-KIBANA%20OPERATIONELLE.PNG) **Interface Kibana opérationnelle** |

<div align="center">

### 🔎 Exploration des logs

</div>

| | |
|---|---|
| ![7](7-INDEX-PATTERN%20FILEBEAT-AVEC%206547%20CHAMPS.PNG) **Index pattern filebeat-\* (6 547 champs)** | ![8](8-DISCOVER--LOGS%20EN%20TEMPS%20REEL%20(115%20HITS).PNG) **Discover — logs temps réel (115 hits)** |

<div align="center">

### 🔐 Module Security

</div>

| | |
|---|---|
| ![9](9-%20SECURITY%20OVERVIEW%20--%207,236%20EVENTS%20DETECTES.PNG) **Security Overview (7 236 événements)** | ![10](10-GENERATION%20DES%20CLES%20DE%20CHIFFREMENT%20KIBANA.PNG) **Génération des clés de chiffrement** |
| ![11](11-SECTION%20ALERTS%20(PERMISSIONS%20ENGINE).PNG) **Section Alerts  permissions engine** | |

<div align="center">

### 🧪 Simulation d'attaque brute force SSH

</div>

| | |
|---|---|
| ![12](12-%20TEST%20BRUTE%20FORCE%20SSH%20SIMULE.PNG) **Lancement du test SSH simulé** | ![13](13-%20TEST%20BRUTE%20FORCE%20SSH%20SIMULE%20SUITE.PNG) **Test SSH simulé — suite** |
| ![14](14-%20TEST%20BRUTE%20FORCE%20SSH%20SIMULE%20SUITE%202.PNG) **Test SSH simulé  suite 2** | |

<div align="center">

### 📊 Analyse et tableau de bord

</div>

| | |
|---|---|
| ![16](16-%20DISCOVER--LOGS%20EN%20TEMPS%20REEL%20(10,715%20HITS).PNG) **Discover  logs temps réel (10 715 hits)** | ![17](17-%20REQUETE%20KGL%20system.syslog%20AND%20sudo%20--%203%20HITS.PNG) **Requête KQL `syslog AND sudo`** |
| ![18](18-DASHBOARD%20EN%20CONSTRUCTION%20AVEC%20VISUALISATIONS.PNG) **Dashboard en construction** | ![19](19-%20DASHBOARDS%20EN%20CONSTRUCTION%20AVEC%20VISUALISATIONS.PNG) **Dashboard  suite** |
| ![20](20-%20TABLEAU%20DE%20BORD%20DE%20SUPERVISION%20DES%20LOGS.PNG) **Tableau de bord final** | ![21](21-%20ANALYSE%20DES%20DONNEES%20COLLECTEES%20PAR%20ELASTIC%20STACK.PNG) **Analyse des données collectées** |

---

<div align="center">

*Projet réalisé dans le cadre d'une préparation à l'alternance Cybersécurité / Réseaux & Systèmes.*

</div>
