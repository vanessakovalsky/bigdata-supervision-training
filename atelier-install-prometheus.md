# Atelier — Installation et prise en main de Prometheus avec Docker Compose (Windows)

## 🎯 Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :

✅ Comprendre le rôle de Prometheus dans la supervision Big Data  
✅ Installer Prometheus sous **Windows** à l’aide de **Docker Compose**  
✅ Démarrer Prometheus et vérifier son fonctionnement  
✅ Explorer l’UI de Prometheus et exécuter vos premières requêtes PromQL  
✅ Superviser votre propre environnement Docker

---

## ✅ 1. Prérequis

Avant de commencer, vérifiez que vous avez :

- Windows 10 ou 11
- **Docker Desktop** installé et démarré
- **Docker Compose** disponible (inclus dans Docker Desktop)
- Un accès Internet pour télécharger les images Docker

👉 Aucun autre prérequis — ce TP est **autonome**.

---

## ✅ 2. Créer le dossier de travail

Ouvrez **PowerShell** et exécutez :

```powershell
mkdir prometheus-lab
cd prometheus-lab
```

## ✅ 3. Créer la configuration Prometheus

Dans le dossier, créez un fichier nommé :
```
prometheus.yml
```
Ajoutez :
```
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]
```
📌 Cette configuration indique à Prometheus de collecter ses propres métriques.

## ✅ 4. Créer le fichier Docker Compose

Créez un fichier :
```
docker-compose.yml
```
Ajoutez :
```
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```
✅ Cette stack lance uniquement Prometheus — simple et idéale pour débuter.

## ✅ 5. Lancer Prometheus

Toujours dans PowerShell :
```
docker compose up -d
```
Vérifiez que le conteneur tourne :
```
docker ps
```
Vous devez voir quelque chose comme :
```
prom/prometheus   Up   0.0.0.0:9090->9090/tcp
```

## ✅ 6. Accéder à l’interface Web de Prometheus

Ouvrez un navigateur et allez sur :
```
👉 http://localhost:9090
```
Vous devriez voir l’interface Prometheus 🎉

## ✅ 7. Vérifier que Prometheus collecte bien des métriques

Dans l’UI :
```
    Allez dans Status > Targets

    Vérifiez que prometheus est UP
```
Ensuite, ouvrez l’onglet Graph et testez :
```
up
```
Résultat attendu ➜ 1

Essayez aussi :
```
prometheus_build_info
prometheus_tsdb_head_series
prometheus_http_requests_total
```

## ✅ 8. Explorer les métriques brutes

Dans un navigateur :
```
👉 http://localhost:9090/metrics
```
Ce sont toutes les métriques exposées par Prometheus.

## ✅ 9. (Optionnel) Ajouter Node Exporter

Permet de superviser la machine hôte (ou plutôt le conteneur sur Windows)

Modifiez docker-compose.yml :
```
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
```
Puis ajoutez dans prometheus.yml :
```
  - job_name: "node"
    static_configs:
      - targets: ["node-exporter:9100"]
```
Rechargez :
```
docker compose up -d
```
Testez :
```
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_size_bytes
```

## ✅ 10. Arrêter et nettoyer l’environnement

Arrêter les services :
```
docker compose down
```
Nettoyer les images Docker (optionnel) :
```
docker system prune -f
```

## 🎓 Conclusion

Vous venez de :

✅ Installer Prometheus avec Docker Compose
✅ Explorer l’interface web et les métriques
✅ Exécuter vos premières requêtes PromQL
✅ Ajouter un exporter système optionnel

Prochaine étape possible 
👉 supervision JMX, Kafka, Spark, Hadoop, Grafana…

## 🚀 Ressources utiles

- Documentation Prometheus : https://prometheus.io/docs/

- Docker Hub Prometheus : https://hub.docker.com/r/prom/prometheus

- Playground PromQL : https://promlens.com
