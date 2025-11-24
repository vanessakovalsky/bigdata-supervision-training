# 11. TP3 : Monitoring stack Big Data complète

**Objectif** : Déployer un monitoring complet avec Prometheus

**Architecture cible** :
```
3 Nœuds Cassandra + 3 Nœuds MongoDB + 1 HDFS
                    │
                    ▼
    Node Exporter (9100) sur chaque nœud
    JMX Exporter (9500) sur Cassandra
    MongoDB Exporter (9216) sur MongoDB
    Hadoop Exporter (9870) sur NameNode
                    │
                    ▼
              Prometheus (9090)
               Scrape toutes les 15s
              Rétention 90 jours
                    │
                    ▼
               Grafana (3000)
            Dashboards pré-configurés
```

**Étape 1 : Configuration Prometheus complète**

`/etc/prometheus/prometheus.yml` :
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'bigdata-prod'
    environment: 'production'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

rule_files:
  - '/etc/prometheus/rules/*.yml'

scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporters (métriques système)
  - job_name: 'node'
    static_configs:
      - targets:
          - 'node1:9100'
          - 'node2:9100'
          - 'node3:9100'
        labels:
          cluster: 'bigdata'

  # Cassandra Cluster
  - job_name: 'cassandra'
    scrape_interval: 30s
    static_configs:
      - targets:
          - 'cassandra1:9500'
          - 'cassandra2:9500'
          - 'cassandra3:9500'
        labels:
          service: 'cassandra'
          cluster_name: 'prod-cluster'

  # MongoDB Replica Set
  - job_name: 'mongodb'
    static_configs:
      - targets:
          - 'mongodb1:9216'
          - 'mongodb2:9216'
          - 'mongodb3:9216'
        labels:
          service: 'mongodb'
          replica_set: 'rs0'

  # Hadoop HDFS
  - job_name: 'hadoop-namenode'
    static_configs:
      - targets: ['namenode:9870']
        labels:
          service: 'hadoop'
          component: 'namenode'

  - job_name: 'hadoop-datanode'
    static_configs:
      - targets:
          - 'datanode1:9871'
          - 'datanode2:9871'
          - 'datanode3:9871'
        labels:
          service: 'hadoop'
          component: 'datanode'
```

**Étape 2 : Règles d'alertes**

`/etc/prometheus/rules/bigdata_alerts.yml` :
```yaml
groups:
  - name: infrastructure
    interval: 30s
    rules:
      - alert: NodeDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Node {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"

      - alert: HighCPU
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"

      - alert: HighMemory
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/data"} / node_filesystem_size_bytes{mountpoint="/data"}) * 100 < 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value }}% remaining"

  - name: cassandra
    interval: 30s
    rules:
      - alert: CassandraReadTimeouts
        expr: rate(cassandra_client_request_timeouts_total{operation="Read"}[5m]) > 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Cassandra read timeouts on {{ $labels.instance }}"
          description: "{{ $value }} timeouts/sec"

      - alert: CassandraHighLatency
        expr: cassandra_client_request_read_latency_99thpercentile > 100000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High read latency on {{ $labels.instance }}"
          description: "P99 latency is {{ $value }}µs"

  - name: mongodb
    interval: 30s
    rules:
      - alert: MongoDBReplicationLag
        expr: mongodb_mongod_replset_oplog_lag_seconds > 10
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "MongoDB replication lag on {{ $labels.instance }}"
          description: "Lag is {{ $value }} seconds"

      - alert: MongoDBHighConnections
        expr: mongodb_connections{state="current"} / mongodb_connections{state="available"} > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High MongoDB connections on {{ $labels.instance }}"
          description: "{{ $value | humanizePercentage }} of connections used"
```

**Étape 3 : Validation et démarrage**

```bash
# Valider configuration
promtool check config /etc/prometheus/prometheus.yml

# Valider règles
promtool check rules /etc/prometheus/rules/*.yml

# Redémarrer Prometheus
sudo systemctl restart prometheus

# Vérifier logs
sudo journalctl -u prometheus -f

# Vérifier targets dans UI
# http://localhost:9090/targets

# Vérifier règles
# http://localhost:9090/rules

# Vérifier alertes
# http://localhost:9090/alerts
```

**Étape 4 : Requêtes de validation**

```promql
# Vérifier tous les targets UP
count(up == 1)

# Vérifier nombre de métriques par job
count by(job) (up)

# CPU moyen du cluster
avg(100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100))

# Mémoire totale cluster (GB)
sum(node_memory_MemTotal_bytes) / 1024 / 1024 / 1024

# Latence Cassandra moyenne
avg(cassandra_client_request_read_latency_mean) / 1000

# Lag réplication MongoDB
max(mongodb_mongod_replset_oplog_lag_seconds)
```
