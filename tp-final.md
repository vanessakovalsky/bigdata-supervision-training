# 8. TP Final : Stack complète de A à Z

### 8.1 Objectif

Déployer une stack de supervision complète pour un cluster Big Data comprenant :
- 3 nœuds Cassandra
- 3 nœuds MongoDB
- 1 HDFS (NameNode + 3 DataNodes)

### 8.2 Étapes de déploiement

**Phase 1 : Infrastructure de monitoring (2h)**
```bash
# 1. Installer Elasticsearch
# 2. Installer Kibana
# 3. Installer Logstash
# 4. Installer Prometheus
# 5. Installer Grafana
# 6. Vérifier tous les services UP
```

**Phase 2 : Collecteurs (1h)**
```bash
# 1. Déployer Node Exporter sur tous les nœuds
# 2. Configurer JMX Exporter pour Cassandra
# 3. Configurer MongoDB Exporter
# 4. Configurer Hadoop JMX Exporter
# 5. Déployer Filebeat pour logs
# 6. Déployer Metricbeat pour métriques système
```

**Phase 3 : Configuration (1h)**
```bash
# 1. Prometheus: ajouter tous les targets
# 2. Logstash: configurer pipelines
# 3. Elasticsearch: créer index templates
# 4. Kibana: créer index patterns
# 5. Grafana: ajouter datasources
```

**Phase 4 : Dashboards (1h30)**
```bash
# 1. Grafana: Dashboard Cassandra
# 2. Grafana: Dashboard MongoDB
# 3. Grafana: Dashboard Hadoop
# 4. Grafana: Dashboard Infrastructure global
# 5. Kibana: Dashboard Logs
```

**Phase 5 : Alerting (1h)**
```bash
# 1. Prometheus: règles d'alerte
# 2. Alertmanager: configuration
# 3. Grafana: alertes dashboards
# 4. Kibana: Watcher (optionnel)
# 5. Tests d'alerting
```

**Phase 6 : Validation (30min)**
```bash
# 1. Tests de charge
# 2. Simulation d'incidents
# 3. Vérification alertes
# 4. Documentation
# 5. Handover équipe ops
```
