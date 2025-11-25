
# 🧪 TP Découverte Kibana

## Objectif

* Découvrir Kibana et son interface
* Installer Elasticsearch et Logstash via Docker Compose
* Envoyer des logs via Logstash vers Elasticsearch
* Visualiser et explorer les données dans Kibana
* Créer un dashboard simple

## Prérequis

* Docker et Docker Compose installés
* Accès internet
* Aucun autre service nécessaire

---

## 1️⃣ Préparer l’environnement Docker Compose

Créer un fichier `docker-compose.yml` :

```yaml
version: "3.9"

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
```

---

## 2️⃣ Configurer Logstash

Créer un fichier `logstash.conf` :

```conf
input {
  beats {
    port => 5044
  }
  stdin {}
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "tp-logs-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```

---

## 3️⃣ Lancer l’environnement

```bash
docker compose up -d
```

Vérifier que tous les containers sont running :

```bash
docker ps
```

Ports exposés :

* Elasticsearch : 9200
* Kibana : 5601
* Logstash : 5044

---

## 4️⃣ Vérification des services

* Elasticsearch : `http://localhost:9200` → JSON d’état
* Kibana : `http://localhost:5601` → interface web
* Logstash : logs visibles dans `docker logs logstash`

---

## 5️⃣ Envoyer des logs de test

### Option 1 : stdin

```bash
docker exec -it logstash /usr/share/logstash/bin/logstash -f /usr/share/logstash/pipeline/logstash.conf
```

Puis saisir quelques lignes, elles seront indexées.

### Option 2 : Filebeat (facultatif)

On peut configurer Filebeat pour envoyer des logs vers Logstash sur le port 5044.

---

## 6️⃣ Découverte de Kibana

1. Ouvrir Kibana : [http://localhost:5601](http://localhost:5601)
2. Menu latéral → **Discover**
3. Choisir l’index pattern `tp-logs-*`
4. Explorer les logs envoyés via Logstash

---

## 7️⃣ Créer un dashboard

1. Menu latéral → **Dashboard → Create dashboard**
2. Ajouter un panel de type **Data Table** ou **Line Chart**
3. Sélectionner l’index `tp-logs-*`
4. Ajouter un graphique du nombre de logs par timestamp
5. Ajouter un panel filtrant par un mot-clé spécifique
6. Sauvegarder le dashboard

---

## 8️⃣ Exercices supplémentaires

| Exercice | Objectif                                                     |
| -------- | ------------------------------------------------------------ |
| 1        | Explorer le nombre de logs par heure dans Discover           |
| 2        | Créer un graphique des logs contenant un mot-clé particulier |
| 3        | Ajouter un filtre sur un champ spécifique (ex: message)      |
| 4        | Créer un dashboard avec au moins 2 panels                    |
| 5        | Sauvegarder et partager le dashboard                         |

---

# 🎯 Objectifs atteints

* Kibana installé et connecté à Elasticsearch
* Logstash ingestant des logs
* Découverte des logs dans Kibana
* Création d’un dashboard simple

---

# 🔹 Notes

* TP autonome et compatible Windows / Docker Desktop
* Toutes les images sont officielles Elastic
* Aucune authentification n’est nécessaire pour ce TP
