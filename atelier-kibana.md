
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

* Récupérer le dépôt : https://github.com/deviantony/docker-elk
* Modifier le fichier logstach/config/pipeline/logstash.conf et remplacer le contenu par le suivant :
```
input {
	beats {
		port => 5044
	}

	tcp {
		port => 50000
	}

	http {
		host => "0.0.0.0"
		port => 8080
	}
}

## Add your filters / logstash plugins configuration here

output {
	elasticsearch {
		index => "logstash-demo-%{+YYYY.MM.dd}"
		hosts => "elasticsearch:9200"
		user => "logstash_internal"
		password => "${LOGSTASH_INTERNAL_PASSWORD}"
	}
}
```
* 
Se rendre dans le dossier et lancer les deux commande (l'une après l'autre) :
```
docker compose up setup
 docker-compose up -d
```
Attendre un peu, puis accéder aux différents outils :
```
    http://localhost:9200 (id : elastic pass: changeme )

    http://localhost:5000 (logastash)

    http://localhost:5601 (kibana)
```

## Vérification des services

* Elasticsearch : `http://localhost:9200` → JSON d’état
* Pour la première connexion à Kibana vous devez générer un token avec elasticsearch avec la commande suivante : `docker compose exec elasticsearch bin/elasticsearch-create-enrollment-token --scope kibana`
* Kibana : `http://localhost:5601` → interface web
* Logstash : logs visibles dans `docker logs logstash`

---

## 5️⃣ Envoyer des logs de test

```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"msg":"hello logstash"}'
```

* Attendre quelques instants et votre message sera indexé par elasticsearch


## Découverte de Kibana

1. Ouvrir Kibana : [http://localhost:5601](http://localhost:5601)
2. Menu latéral → **Discover**
3. Dans Data View, cliquer sur la fleche à côté de All logs, puis cliquer sur Create a Data view
4. Choisir l’index pattern `logstash-demo*`
5. Cliquer sur Save Data view to Kibana
6. Explorer les logs envoyés via Logstash

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
