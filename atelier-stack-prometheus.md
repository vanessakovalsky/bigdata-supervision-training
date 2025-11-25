# TP Prometheus & Grafana sur Node Exporter et MongoDB

## Objectif

Découvrir Prometheus et Grafana en collectant des métriques système avec Node Exporter et des métriques MongoDB via l'exporter Percona, puis visualiser ces données sur Grafana.

## Prérequis

* Docker et Docker Compose installés.
* Accès internet.

## Étape 1 : Structure du projet

Créez un dossier `prometheus-tp` et à l'intérieur, créez les fichiers suivants :

```
prometheus-tp/
  ├─ docker-compose.yml
  ├─ prometheus.yml
```

## Étape 2 : Fichier docker-compose.yml

```yaml
version: '3.9'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - '9090:9090'
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    ports:
      - '9100:9100'
    networks:
      - monitoring

  mongodb:
    image: mongo:7
    container_name: mongodb
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=secret
    ports:
      - '27017:27017'
    networks:
      - monitoring
    healthcheck:
      test: ['CMD', 'mongosh', '--username', 'admin', '--password', 'secret', '--eval', 'db.adminCommand(\'ping\')']
      interval: 5s
      timeout: 5s
      retries: 20

  mongodb-exporter:
    image: percona/mongodb_exporter:0.40
    container_name: mongodb-exporter
    depends_on:
      mongodb:
        condition: service_healthy
    command:
      - '--mongodb.uri=mongodb://admin:secret@mongodb:27017/admin'
      - '--collect-all'
    ports:
      - '9216:9216'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - '3000:3000'
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    networks:
      - monitoring

networks:
  monitoring:
```

## Étape 3 : Fichier prometheus.yml

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 15s
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: 'node'
    scrape_interval: 5s
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'mongodb'
    scrape_interval: 15s
    metrics_path: '/metrics'
    static_configs:
      - targets: ['mongodb-exporter:9216']
```

## Étape 4 : Lancer les conteneurs

```bash
docker compose up -d
```

* Vérifiez que tous les conteneurs sont en `running` :

```bash
docker compose ps
```

## Étape 5 : Vérifier les métriques

* Node Exporter : [http://localhost:9100/metrics](http://localhost:9100/metrics)
* MongoDB Exporter : [http://localhost:9216/metrics](http://localhost:9216/metrics)
* Prometheus : [http://localhost:9090/targets](http://localhost:9090/targets)

## Étape 6 : Découverte de Grafana

1. Ouvrez [http://localhost:3000](http://localhost:3000)
2. Connectez-vous avec `admin/admin`
3. Ajouter Prometheus comme datasource :

   * URL: `http://prometheus:9090`
4. Importer des dashboards :

   * Dashboard MongoDB : ID `2589`
   * Dashboard Node Exporter : ID `1860`

## Étape 7 : Exercices Prometheus

1. Afficher l'utilisation CPU de votre machine avec `node_cpu_seconds_total`
2. Afficher la mémoire disponible avec `node_memory_MemAvailable_bytes`
3. Compter le nombre de connections MongoDB actives avec `mongodb_up`
4. Créer une alerte si `mongodb_up` est à 0 plus de 30s

## Étape 8 : Aller plus loin (optionnel)

* Ajouter la stack ELK (Elasticsearch, Logstash, Kibana) pour centraliser les logs :

  * Lancer Elasticsearch : `docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.10.0`
  * Lancer Kibana : `docker run -d --name kibana -p 5601:5601 --link elasticsearch:elasticsearch docker.elastic.co/kibana/kibana:8.10.0`
  * Lancer Logstash avec un fichier de config pour récupérer les logs des conteneurs
  * Visualiser les logs et créer des dashboards Kibana

---

TP terminé.
