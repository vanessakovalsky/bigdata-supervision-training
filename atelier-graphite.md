
# TP : Supervision BigData avec **Graphite** dans un environnement Windows + Docker Compose

**Objectif** : Déployer rapidement une stack Graphite (Graphite Web + Carbon + Whisper + StatsD) sur Windows à l'aide de Docker Compose, simuler l'émission de métriques issues d'un environnement BigData (jobs, tâches, files, latences) et visualiser/analyser ces métriques dans l'interface Graphite.

> Remarque : ce TP est conçu pour Windows 10/11 avec Docker Desktop (WSL2 recommandé). Les commandes PowerShell sont indiquées.

---

## 1) Prérequis

- Windows 10/11 avec Docker Desktop installé et WSL2 activé.
- Docker Compose (inclus avec Docker Desktop). Sur PowerShell, on utilisera `docker compose`.
- PowerShell (exécuter en administrateur si nécessaire).
- (Optionnel) Python 3.8+ pour lancer les scripts d'émulation de métriques localement.
- Minimum 2 GB de RAM dédiés à Docker (ajuster via Docker Desktop > Settings).

---

## 2) Arborescence du projet

Créez un dossier pour le TP, par exemple `C:\graphite-tp` :

```
C:\graphite-tp
│   docker-compose.yml
│   send_metrics.py
│   README.md   (ce document)
└── storage
    └── (données persistantes graphite)
```

---

## 3) Contenu du `docker-compose.yml`

Copiez / collez le contenu suivant dans `docker-compose.yml`. Ce compose lève un service `graphite` (image `graphiteapp/graphite-statsd`) qui contient Graphite Web, Carbon, Whisper et StatsD :

```yaml
version: "3.8"

services:
  graphite:
    image: graphiteapp/graphite-statsd:1.1.8-2
    container_name: graphite
    restart: unless-stopped
    ports:
      - "80:80"         # Graphite web
      - "2003:2003"     # Carbon plaintext (TCP)
      - "2004:2004"     # Carbon pickle (TCP)
      - "2023:2023"     # Carbon line receiver (deprecated)
      - "2024:2024"
      - "8125:8125/udp" # StatsD UDP
      - "8126:8126"     # StatsD admin (optional)
    volumes:
      - ./storage/whisper:/opt/graphite/storage/whisper
      - ./storage/conf:/opt/graphite/conf
      - ./storage/log/:/opt/graphite/storage/log
    environment:
      - GRAPHITE_TIMEZONE=Europe/Paris
      - STATSD_INTERFACE=0.0.0.0
      - STATSD_PORT=8125

# NOTE: l'image 'graphiteapp/graphite-statsd' inclut Graphite, Carbon, Whisper et StatsD.
```

> Remarque : la version d'image peut évoluer ; ce TP utilise une image largement utilisée (`graphiteapp/graphite-statsd`). Si vous êtes en environnement strict, vérifiez la version d'image disponible sur Docker Hub.

---

## 4) Démarrer la stack (PowerShell)

Ouvrez PowerShell dans `C:\graphite-tp` :

```powershell
# Lancer en mode détaché
docker compose up -d

# Vérifier que le conteneur tourne
docker ps --filter "name=graphite"
```

Attendez ~10-20 secondes le temps que les services démarrent. Graphite Web sera accessible sur : `http://localhost/` (port 80) ou `http://127.0.0.1/`.

---

## 5) Structure minimale de configuration et persistance

Nous montons `./storage/whisper` dans le conteneur pour persister les fichiers Whisper (time-series). Vous pouvez créer un fichier de configuration custom dans `./storage/conf` si vous souhaitez changer `storage-schemas.conf` ou `storage-aggregation.conf`. Exemple minimal :

Créez `storage/conf/storage-schemas.conf` (optionnel — l'image a des valeurs par défaut) :

```
[default]
pattern = .*
retentions = 10s:6h,1m:7d,10m:5y
```

> Important : les réglages de rétention impactent la granularité et l'espace disque. Adapter selon vos besoins.

---

## 6) Simuler des métriques BigData

Nous allons créer un script Python `send_metrics.py` qui va envoyer des métriques de type **jobs**, **tâches**, **throughput**, **latence** via le protocole StatsD (UDP) et via le protocole Graphite plaintext (TCP 2003). Copiez ce script :

```python
# send_metrics.py
# Usage: python send_metrics.py
import socket
import time
import random
import threading

GRAPHITE_TCP_HOST = "127.0.0.1"
GRAPHITE_TCP_PORT = 2003

STATSD_HOST = "127.0.0.1"
STATSD_PORT = 8125  # UDP

# Exemple: envoi via StatsD (compteurs, timings, gauges)
def send_statsd():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    node_count = 10
    while True:
        timestamp = int(time.time())
        # Simuler tâches par noeud
        for n in range(node_count):
            node = f"bigdata.node{n}"
            tasks = random.randint(0, 50)
            # compteur
            msg = f"{node}.tasks:{tasks}|g"
            sock.sendto(msg.encode(), (STATSD_HOST, STATSD_PORT))
            # latence médiane
            latency_ms = random.gauss(200, 80)
            lat_msg = f"{node}.latency:{int(abs(latency_ms))}|ms"
            sock.sendto(lat_msg.encode(), (STATSD_HOST, STATSD_PORT))
        # Simuler job completions
        jobs = random.randint(0, 20)
        sock.sendto(f"bigdata.jobs.completed:{jobs}|c".encode(), (STATSD_HOST, STATSD_PORT))
        # Throughput
        throughput = random.uniform(1000, 50000)
        sock.sendto(f"bigdata.throughput:{int(throughput)}|g".encode(), (STATSD_HOST, STATSD_PORT))

        time.sleep(2)

# Envoi direct via le protocole plaintext de Graphite (TCP 2003)
def send_graphite_plaintext():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    while True:
        try:
            sock.connect((GRAPHITE_TCP_HOST, GRAPHITE_TCP_PORT))
            break
        except Exception:
            time.sleep(1)
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    while True:
        timestamp = int(time.time())
        # Simuler métriques agrégées
        lines = []
        lines.append(f"bigdata.cluster.jobs_running {random.randint(50, 500)} {timestamp}")
        lines.append(f"bigdata.cluster.tasks_waiting {random.randint(0, 200)} {timestamp}")
        lines.append(f"bigdata.cluster.avg_latency {random.uniform(100, 2000):.2f} {timestamp}")
        payload = "\n".join(lines) + "\n"
        try:
            sock.send(payload.encode())
        except Exception:
            # reconnect si nécessaire
            try:
                sock.close()
            except:
                pass
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            time.sleep(1)
            try:
                sock.connect((GRAPHITE_TCP_HOST, GRAPHITE_TCP_PORT))
            except:
                pass
        time.sleep(5)

if __name__ == "__main__":
    t1 = threading.Thread(target=send_statsd, daemon=True)
    t2 = threading.Thread(target=send_graphite_plaintext, daemon=True)
    t1.start(); t2.start()
    print("Envoi de métriques vers Graphite/StatsD... (CTRL+C pour arrêter)")
    while True:
        time.sleep(1)
```

Lancer le script (sur la machine hôte) :

```powershell
python .\send_metrics.py
```

> Si vous exécutez le script depuis une autre machine virtuelle, adaptez `STATSD_HOST` / `GRAPHITE_TCP_HOST` à l'IP de la machine Windows (ou utilisez `host.docker.internal` depuis un conteneur pour atteindre l'hôte).

---

## 7) Visualiser dans Graphite Web

1. Ouvrez `http://localhost/` dans votre navigateur.
2. Allez dans **Metrics** → naviguez vers `bigdata` → `cluster` ou `nodeX`.
3. Sélectionnez métriques (ex : `bigdata.cluster.jobs_running`) et cliquez sur **Compose** pour afficher le graphe.
4. Pour visualiser plusieurs séries, utilisez la syntaxe `alias`, `sumSeries`, `averageSeries`, etc. (Graphite fournit ces fonctions dans l'UI).

Exemples de requêtes (dans la zone de composition) :
- `sumSeries(bigdata.node*.tasks)` — total tasks across nodes.
- `averageSeries(bigdata.node*.latency)` — latence moyenne.

---

## 8) Cas pratique : détecter un pic de latence et alerter (manuel)

Graphite seul n'est pas un système d'alerte avancé ; vous pouvez :
- Utiliser `graphite-web` + un script externe qui interroge l'API render et déclenche des alertes.
- Intégrer un outil d'alerte (ex : **Grafana** avec alerting, ou **Alerta**, ou un simple cron + curl).

Exemple simple (bash / PowerShell pseudo) : interroger l'endpoint render et vérifier la valeur moyenne sur 5m, puis envoyer un mail / webhook si dépasse un seuil.

---

## 9) Personnalisation pour BigData

- **Rétention** : pour BigData, conservez granularité élevée (ex : 1s ou 10s) pour 6-12h puis 1m pour quelques jours. Ajustez `storage-schemas.conf`.
- **Agrégation** : définir règles dans `storage-aggregation.conf` pour moyennes/percentiles.
- **Naming** : adoptez une nomenclature claire : `cluster.<clustername>.node.<id>.<metric>` ou `service.job.<jobname>.<metric>`.
- **Tagging** : Graphite classique n'a pas de true-tags — modélisez via la hiérarchie de noms ou utilisez Graphite+tags (extensions) ou migration vers M3/Prometheus si tags natifs requis.

---

## 10) Debug / dépannage

- Logs du conteneur :
```powershell
docker logs -f graphite
```
- Vérifier que StatsD écoute sur le port UDP :
```powershell
# Depuis l'hôte (PowerShell) vous pouvez tester en envoyant un paquet UDP simple
echo -n "test.metric:1|c" | nc -u -w1 127.0.0.1 8125
```
- Si les métriques n'apparaissent pas, vérifier :
  - Volumes montés (permissions).
  - Configuration de `storage-schemas.conf`.
  - Format des messages (StatsD: `metric:value|g` ou `|c` ou `|ms`).

---

## 11) Nettoyage

```powershell
docker compose down
# Supprimer volumes locaux (attention, perte de données)
Remove-Item -Recurse -Force .\storage\whisper
```

---

## 12) Extensions possibles (exercices pour aller plus loin)

- Ajouter **Grafana** et configurer une source Graphite pour tableaux de bord et alerting natif.
- Intégrer l'envoi de métriques depuis un cluster Hadoop / Spark (via push gateway ou metrics reporters).
- Ajout d'un exporter qui convertit logs d'ingestion en métriques (ex: throughput per topic).
- Mettre en place des tests de charge pour simuler millions de points/heure.

---

## 13) Fichiers fournis

- `docker-compose.yml` — définition de la stack.
- `send_metrics.py` — script de simulation de métriques.
- `storage/conf/storage-schemas.conf` — exemple de rétention.

---

## 14) Remarques finales

Ce TP vise à être autonome sur une machine Windows avec Docker Desktop. Il montre comment rapidement mettre en place Graphite, injecter des métriques d'un environnement BigData simulé et commencer l'analyse. Pour un déploiement production, considérez :
- Réglages de stockage et quotas disque (Whisper croît rapidement).
- HA et sharding pour Graphite (metric aggregation, carbon-relay).
- Migration vers des solutions scalables si volumétrie très élevée (ex : Metrictank, Cortex, Prometheus Remote Write vers systèmes de stockage long terme).

---

Bonne pratique : conservez toujours un répertoire `storage/whisper` persistant pour éviter de perdre vos séries entre redémarrages.
