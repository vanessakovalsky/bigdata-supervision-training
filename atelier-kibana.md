
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

Cloner le dépôt : https://github.com/deviantony/docker-elk
Se rendre dans le dossier et lancer : docker-compose up -d
Attendre un peu, puis accéder aux différents outils :
```
    http://localhost:9200 (id : elastic pass: changeme )

    http://localhost:5000 (logastash)

    http://localhost:5601 (kibana)
```


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
* Pour la première connexion à Kibana vous devez générer un token avec elasticsearch avec la commande suivante : `docker compose exec elasticsearch bin/elasticsearch-create-enrollment-token --scope kibana`
* Kibana : `http://localhost:5601` → interface web
* Logstash : logs visibles dans `docker logs logstash`

---

## 5️⃣ Envoyer des logs de test

### stdin

```bash
docker exec -it logstash /usr/share/logstash/bin/logstash -f /usr/share/logstash/pipeline/logstash.conf
```

Puis saisir quelques lignes, elles seront indexées.

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
