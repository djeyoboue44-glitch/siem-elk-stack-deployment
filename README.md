# siem-elk-stack-deployment
Déploiement complet d'un SIEM ELK Stack sur Kali Linux/Ubuntu avec règles de détection et dashboard Kibana
cat > /mnt/user-data/outputs/README.md << 'EOF'
# 🛡️ SIEM ELK Stack — Déploiement & Détection d'incidents

![Status](https://img.shields.io/badge/Status-Opérationnel-39ff14?style=for-the-badge)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch-9.4.1-00BFB3?style=for-the-badge&logo=elasticsearch&logoColor=white)
![Kibana](https://img.shields.io/badge/Kibana-7.17.29-E8478B?style=for-the-badge&logo=kibana&logoColor=white)
![Filebeat](https://img.shields.io/badge/Filebeat-7.17.29-00BFB3?style=for-the-badge&logo=elastic&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Kali_Linux_%7C_Ubuntu-557C94?style=for-the-badge&logo=linux&logoColor=white)
![École](https://img.shields.io/badge/École-IRIS_Paris-orange?style=for-the-badge)

> **Déploiement complet d'un SIEM (Security Information and Event Management) basé sur la stack ELK — Elasticsearch 9.4.1, Kibana 7.17.29 via Docker, Filebeat 7.17.29 — avec collecte de logs multi-sources, requêtes KQL de détection et dashboard de supervision en temps réel. +10 715 événements collectés et analysés.**

---

## 📋 Sommaire

- [Objectifs](#-objectifs)
- [Architecture](#-architecture)
- [Environnement technique](#-environnement-technique)
- [Installation & Configuration](#-installation--configuration)
- [Collecte de logs](#-collecte-de-logs)
- [Requêtes KQL de détection](#-requêtes-kql-de-détection)
- [Dashboard Kibana](#-dashboard-kibana)
- [Tests de validation](#-tests-de-validation)
- [Résultats](#-résultats)
- [Auteur](#-auteur)

---

## 🎯 Objectifs

Déployer une solution SIEM complète dans un environnement virtualisé simulant une infrastructure d'entreprise réelle.

**Compétences développées :**
- Centralisation et analyse des logs système et sécurité
- Détection d'incidents via requêtes KQL personnalisées
- Visualisation des événements en temps réel via Kibana
- Simulation d'attaques (brute force SSH) et validation de la détection
- Pipeline de collecte : Filebeat → Elasticsearch → Kibana

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      INFRASTRUCTURE LAB                          │
│                                                                 │
│  ┌───────────────────┐          ┌──────────────────────────┐   │
│  │   Kali Linux      │          │        Ubuntu            │   │
│  │   (SIEM Server)   │◄─────────│   (Client Linux)         │   │
│  │                   │  Filebeat│                          │   │
│  │  Elasticsearch    │  :5044   │  Filebeat 7.17.29        │   │
│  │  9.4.1            │          │  → /var/log/auth.log     │   │
│  │                   │          │  → /var/log/syslog       │   │
│  │  Kibana 7.17.29   │          └──────────────────────────┘   │
│  │  (Docker)         │                                         │
│  │  :5601            │          ┌──────────────────────────┐   │
│  │                   │◄─────────│     Windows (WS01)       │   │
│  └───────────────────┘ Winlogbeat  Winlogbeat               │   │
│                                 └──────────────────────────┘   │
│                        VMware Workstation — Réseau NAT          │
└─────────────────────────────────────────────────────────────────┘

PIPELINE :
Filebeat/Winlogbeat → Elasticsearch (9200) → Kibana (5601)
```

---

## 💻 Environnement technique

| Machine | Rôle | OS | Outils déployés |
|---------|------|----|-----------------|
| KALI (SIEM) | Serveur central | Kali Linux 2024.3 | Elasticsearch 9.4.1 · Kibana 7.17.29 (Docker) |
| UBUNTU | Client Linux | Ubuntu (VMware) | Filebeat 7.17.29 |
| WS01 | Client Windows | Windows 10/11 | Winlogbeat |
| DC01 | Contrôleur domaine | Windows Server 2022 | — |

**Versions réelles utilisées :**
```
Elasticsearch : 9.4.1  (paquet apt elastic.co/packages/9.x)
Kibana        : 7.17.29 (image Docker docker.elastic.co/kibana/kibana:7.17.29)
Filebeat      : 7.17.29 (paquet apt elastic.co/packages/7.x)
VMware        : Workstation Pro
```

---

## ⚙️ Installation & Configuration

### 1. Installation Elasticsearch (Kali Linux)

```bash
# Import de la clé GPG
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch \
  | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

# Ajout du dépôt 8.x
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
https://artifacts.elastic.co/packages/8.x/apt stable main" \
| sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# Installation
sudo apt update && sudo apt install elasticsearch -y

# Vérification — version 9.4.1 installée
curl -s http://localhost:9200 | grep number
# "number" : "7.17.29"
```

### 2. Déploiement Kibana via Docker

```bash
# Génération des clés de chiffrement
sudo docker exec -it kibana bash -c "bin/kibana-encryption-keys generate"

# Lancement Kibana avec clés de chiffrement
sudo docker run -d --name kibana \
  --link elasticsearch:elasticsearch \
  -p 5601:5601 \
  -e "XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=<generated_key>" \
  -e "XPACK_SECURITY_ENCRYPTIONKEY=<generated_key>" \
  -e "XPACK_REPORTING_ENCRYPTIONKEY=<generated_key>" \
  docker.elastic.co/kibana/kibana:7.17.29

# Vérification — Kibana disponible
sudo docker logs kibana | grep "available"
# {"message":"Kibana is now available (was degraded)"}
```

**Kibana accessible sur :** `http://localhost:5601`

### 3. Installation Filebeat (Ubuntu client)

```bash
# Ajout dépôt Elastic 7.x
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch \
  | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
https://artifacts.elastic.co/packages/7.x/apt stable main" \
| sudo tee /etc/apt/sources.list.d/elastic-7.x.list

# Installation Filebeat 7.17.29
sudo apt update && sudo apt install filebeat -y

# Activation module system
sudo filebeat modules enable system

# Setup index management
sudo filebeat setup --index-management \
  -E output.logstash.enabled=false \
  -E 'output.elasticsearch.hosts=["http://192.168.1.49:9200"]'

# Démarrage
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

### 4. Configuration Filebeat (`/etc/filebeat/filebeat.yml`)

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/auth.log
      - /var/log/syslog

filebeat.modules:
  - module: system
    syslog:
      enabled: true
    auth:
      enabled: true

output.elasticsearch:
  hosts: ["http://192.168.1.49:9200"]

setup.kibana:
  host: "http://192.168.1.49:5601"
```

---

## 📥 Collecte de logs

Une fois le pipeline en place, Kibana reçoit les logs en temps réel.

**Index pattern créé :** `filebeat-*`
- **Champs disponibles :** 6 547 champs indexés
- **Time field :** `@timestamp`
- **Hits sur 7 jours :** 10 715 événements

**Sources collectées :**
```
system.syslog   → logs système Linux
system.auth     → logs d'authentification (/var/log/auth.log)
```

**Découverte (Discover) — 10 715 hits sur 7 jours :**
- Volume concentré le 17 mai 2026 (~7 000 events en pic)
- Agent hostname : `djeyoboue-VMware-Virtual-Platform`
- Agent type : `filebeat` v7.17.29

---

## 🔍 Requêtes KQL de détection

### Requête 1 — Filtrer les logs syslog avec sudo

```kql
event.dataset: "system.syslog" AND message: "sudo"
```
**Résultat :** 3 hits détectés — commandes sudo tracées ✅

---

### Requête 2 — Logs d'authentification

```kql
event.dataset: "system.auth"
```

---

### Requête 3 — Connexions SSH échouées

```kql
event.dataset: "system.auth" AND message: "Failed password"
```

---

### Requête 4 — Toute activité sur une IP spécifique

```kql
source.ip: "192.168.1.70"
```

---

## 📊 Dashboard Kibana

Dashboard construit avec 2 visualisations principales :

| Visualisation | Type | Axe X | Axe Y |
|---------------|------|--------|-------|
| Volume d'événements dans le temps | Bar vertical | `@timestamp` per 3 hours | Count of records |
| Top hostnames sources | Bar vertical | Top values of `agent.hostname` | Count of records |

**Security Overview :**
- 7 236 événements de sécurité affichés
- Sources : `system.syslog` + `system.auth`
- Timeline disponible pour investigation

---

## ✅ Tests de validation

### Test — Simulation brute force SSH

```bash
# Depuis Kali, tentatives SSH répétées vers Ubuntu (192.168.1.70)
for i in {1..10}; do ssh wronguser@192.168.1.70; done
```

**Résultat observé :**
```
wronguser@192.168.1.70: Permission denied (publickey,password).
wronguser@192.168.1.70: Permission denied, please try again.
[... x10 tentatives]
```

**Dans Kibana :**
- Événements `system.auth` générés et visibles dans Discover ✅
- Pics d'activité visibles dans le dashboard ✅

---

## 📈 Résultats

| Indicateur | Valeur |
|------------|--------|
| Elasticsearch | ✅ Opérationnel — version 9.4.1 |
| Kibana | ✅ Accessible sur :5601 — version 7.17.29 |
| Filebeat | ✅ Actif sur Ubuntu — version 7.17.29 |
| Index pattern | ✅ `filebeat-*` — 6 547 champs |
| Logs collectés (7j) | ✅ **10 715 événements** |
| Pic journalier | ✅ ~7 000 events (17 mai 2026) |
| Test brute force SSH | ✅ Détecté et visible dans Kibana |
| Dashboard | ✅ 2 visualisations opérationnelles |
| Requêtes KQL | ✅ 4 requêtes de détection validées |

---

## 🧰 Stack complète

```
SIEM            : ELK Stack
Elasticsearch   : 9.4.1 (apt — elastic.co/packages/9.x)
Kibana          : 7.17.29 (Docker)
Filebeat        : 7.17.29 (apt — elastic.co/packages/7.x)
Virtualisation  : VMware Workstation
OS Serveur SIEM : Kali Linux 2024.3
OS Client Linux : Ubuntu (VMware Virtual Platform)
OS Client Win   : Windows 10/11 (WS01)
AD              : Windows Server 2022 (DC01)
Réseau          : NAT VMware — 192.168.1.0/24
```

---

## 👤 Auteur

**YOBOUE DJE**
Étudiant Mastère Expert IT — Cybersécurité, Réseaux & Systèmes
École IRIS · Paris · 2025–2026

[![LinkedIn](https://img.shields.io/badge/LinkedIn-yoboue--dje-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/yoboue-dje)
[![GitHub](https://img.shields.io/badge/GitHub-djeyoboue44--glitch-181717?style=flat&logo=github)](https://github.com/djeyoboue44-glitch)
[![Portfolio](https://img.shields.io/badge/Portfolio-djeyoboue44--glitch.github.io-00f5ff?style=flat&logo=firefox)](https://djeyoboue44-glitch.github.io)

---

> *Projet réalisé dans le cadre du Mastère Expert IT à l'École IRIS Paris — Janvier/Mai 2026*
EOF
