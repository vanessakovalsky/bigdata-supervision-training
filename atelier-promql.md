# Atelier — Premiers pas avec PromQL (Windows + Docker Compose + Cassandra + MongoDB)

## 🎯 Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :

✅ Explorer des métriques avec Prometheus  
✅ Manipuler les opérateurs et fonctions PromQL  
✅ Créer des requêtes d’analyse et d’alerte  
✅ Interroger les métriques d’un cluster Cassandra  
✅ Interroger les métriques d’une base MongoDB  
✅ Comprendre les familles de métriques : counters, gauges, histograms, summaries

---

## ✅ Prérequis

✔ Windows 10 ou 11  
✔ Docker Desktop installé et démarré  
✔ Accès Internet  
❌ Aucune installation de Prometheus, Cassandra, MongoDB ou exporter requise

---

## 📁 1 — Arborescence du TP

Créez un dossier de travail :

```
promql-workshop/
│
├── docker-compose.yml
└── prometheus.yml
```

---

## 📜 2 — Fichier `docker-compose.yml`

```yaml
version: "3.9"

services:

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    ports:
      - "9100:9100"
    networks:
      - monitoring

  cassandra:
    image: cassandra:4.1
    container_name: cassandra
    environment:
      - MAX_HEAP_SIZE=512M
      - HEAP_NEWSIZE=100M
    ports:
      - "9042:9042"
    networks:
      - monitoring

  cassandra-exporter:
    image: criteord/cassandra_exporter
    container_name: cassandra-exporter
    ports:
      - "8080:8080"
    environment:
      - CASSANDRA_HOST=cassandra
    networks:
      - monitoring
    depends_on:
      - cassandra

  mongodb:
    image: mongo:7
    container_name: mongodb
    ports:
      - "27017:27017"
    networks:
      - monitoring

  mongodb-exporter:
    image: percona/mongodb_exporter:0.40
    container_name: mongodb-exporter
    command:
      - "--mongodb.uri=mongodb://mongodb:27017"
    ports:
      - "9216:9216"
    networks:
      - monitoring
    depends_on:
      - mongodb

networks:
  monitoring:
```

---

## 📜 3 — Fichier `prometheus.yml`

```
global:
  scrape_interval: 5s

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["node-exporter:9100"]

  - job_name: "cassandra"
    metrics_path: "/metrics"
    static_configs:
      - targets: ["cassandra-exporter:8080"]

  - job_name: "mongodb"
    metrics_path: "/metrics"
    static_configs:
      - targets: ["mongodb-exporter:9216"]
```

---

## 🚀 4 — Démarrage de l’environnement

Depuis PowerShell dans le dossier :

```powershell
docker compose up -d
```

Vérifiez :

```powershell
docker ps
```

---

## 🌍 5 — Accès à Prometheus

Ouvrez votre navigateur :

👉 http://localhost:9090

---

## 🔍 6 — Premiers pas en PromQL

### Lister toutes les métriques

```
{__name__!=""}
```

### CPU de la machine hôte

```
rate(node_cpu_seconds_total[5m])
```

### RAM restante

```
node_memory_MemAvailable_bytes
```

---

## 📦 7 — PromQL et Cassandra

```
rate(cassandra_stats_clientrequest_count[1m])
```

```
cassandra_stats_clientrequest_latencymean{scope="Read"}
```

```
rate(cassandra_stats_clientrequest_errors[1m])
```

---

## 🍃 8 — PromQL et MongoDB

```
rate(mongodb_ss_opcounters_total[1m])
```

```
rate(mongodb_ss_opcounters_insert_total[1m])
```

```
mongodb_ss_connections_current
```

---

## 🚨 9 — Exemples d’alertes

```
rate(cassandra_stats_clientrequest_latencymean[5m]) > 500
```

```
mongodb_ss_connections_current > 200
```

---

## 🚦 10 — Arrêter l’environnement

```powershell
docker compose down
```

---

# 🎉 Fin du TP
