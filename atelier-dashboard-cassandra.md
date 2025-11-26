# TP Complet Découverte Grafana

Ce fichier contient un TP autonome pour découvrir Grafana, connecté à Prometheus et Node Exporter.

## Contenu

* Docker Compose prêt à l'emploi
* Prometheus + Node Exporter + Grafana
* Exercices PromQL avec corrections

---

## 1️⃣ Docker Compose

```yaml
version: "3.9"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
```

### Prometheus config `prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

---

## 2️⃣ Lancer l'environnement

```bash
docker compose up -d
```

Vérifier que tous les containers sont running :

```bash
docker ps
```

---

## 3️⃣ Connexion à Grafana

* Ouvrir [http://localhost:3000](http://localhost:3000)
* Login : `admin` / `admin`
* Changer le mot de passe si nécessaire

---

## 4️⃣ Ajouter Prometheus comme source de données

1. Menu latéral → ⚙️ Configuration → Data Sources
2. Cliquer Add data source → Prometheus
3. URL : `http://prometheus:9090`
4. Cliquer Save & Test → doit afficher Data source is working

---

## 5️⃣ Explorer les métriques

Menu latéral → Explore → Source : Prometheus

| Exercice | Requête PromQL                                                                                                                          | Objectif                        |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| 1        | `node_cpu_seconds_total`                                                                                                                | Lister toutes les métriques CPU |
| 2        | `node_cpu_seconds_total{mode="user"}`                                                                                                   | Filtrer par mode CPU "user"     |
| 3        | `count(node_cpu_seconds_total{mode="user"}) by (instance)`                                                                              | Nombre de cores par instance    |
| 4        | `node_memory_MemTotal_bytes`                                                                                                            | Mémoire totale                  |
| 4        | `node_memory_MemFree_bytes`                                                                                                             | Mémoire libre                   |
| 5        | `sum by(instance) (node_cpu_seconds_total)`                                                                                             | Somme CPU par instance          |
| 6        | `100 * (1 - sum by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) / sum by(instance)(rate(node_cpu_seconds_total[5m])))`      | Pourcentage CPU utilisé         |
| 7        | `node_memory_MemTotal_bytes - node_memory_MemFree_bytes`                                                                                | Mémoire utilisée                |
| 8        | `node_filesystem_size_bytes - node_filesystem_free_bytes`                                                                               | Disque utilisé                  |
| 9        | `topk(3, rate(node_cpu_seconds_total{mode="user"}[5m]))`                                                                                | Top 3 CPU les plus utilisés     |
| 10       | `rate(node_cpu_seconds_total{mode="user"}[5m])`                                                                                         | Taux CPU sur 5 min              |
| 11       | `avg_over_time(rate(node_cpu_seconds_total{mode="user"}[1h]))`                                                                          | Moyenne CPU sur 1h              |
| 12       | `max_over_time((node_memory_MemTotal_bytes - node_memory_MemFree_bytes)[24h:1m])`                                                       | Max mémoire utilisée sur 24h    |
| 13       | `stddev_over_time(rate(node_disk_read_bytes_total[10m]))`                                                                               | Écart-type lectures disque      |
| 14       | `rate(node_disk_read_bytes_total[5m]) / rate(node_disk_written_bytes_total[5m])`                                                        | Ratio lecture/écriture disque   |
| 15       | `(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes`                                            | Mémoire utilisée / totale       |
| 16       | `(node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1`                                                                      | Disques presque pleins          |
| 17       | `rate(node_cpu_seconds_total{mode="system"}[5m])`                                                                                       | CPU système par instance        |
| 18       | `100 * (1 - sum by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) / sum by(instance)(rate(node_cpu_seconds_total[5m]))) > 80` | Alerte CPU >80%                 |
| 19       | `(node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1`                                                                      | Alerte disque presque plein     |
| 20       | `(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.15`                                                                  | Alerte mémoire faible           |

---

## 6️⃣ Créer un dashboard

1. Menu latéral → ➕ Create → Dashboard
2. Ajouter un panel : Add new panel
3. Entrer une des requêtes PromQL ci-dessus
4. Choisir type : Graph, Gauge, Bar gauge…
5. Cliquer Apply → Panel ajouté au dashboard
6. Répéter pour d’autres métriques

---

## 7️⃣ Configurer des alertes simples

* Exemple : CPU > 80%
* Condition PromQL :

```promql
100 * (1 - sum by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) / sum by(instance)(rate(node_cpu_seconds_total[5m]))) > 80
```

* Définir intervalle et notifications si souhaité

---

## 8️⃣ Sauvegarder le dashboard

* Cliquer Save dashboard → Nommer “TP Grafana Node Exporter”

---

## 9️⃣ Bonus / Exploration avancée

* Ajouter plusieurs instances Node Exporter
* Comparer CPU vs mémoire vs disque
* Créer des panels combinés (Graph multi-métriques)
* Explorer les métriques réseau : `node_network_receive_bytes_total`, `node_network_transmit_bytes_total`

---

# 🎯 Objectifs atteints

* Grafana installé et fonctionnel
* Prometheus comme source de données
* Création d’un dashboard multi-panel
* Visualisation de métriques CPU, mémoire, disque
* Mise en place d’alertes simples

---

# 🔹 Notes

* Tous les exercices utilisent Node Exporter uniquement
* TP autonome, compatible Windows / Docker Desktop
* Requêtes PromQL testées et fonctionnelles
