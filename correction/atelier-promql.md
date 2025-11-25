# 🧪 TP PromQL Node Exporter – Corrigé

---

## Niveau 1 – Découverte des métriques

### Exercice 1 : Lister toutes les métriques CPU
```promql
node_cpu_seconds_total
```
### Exercice 2 : Filtrer par état
```promql
node_cpu_seconds_total{mode="user"}
```
### Exercice 3 : Afficher le nombre de CPU par instance
```promql
count(node_cpu_seconds_total{mode="user"}) by (instance)
```
### Exercice 4 : Mémoire totale et libre
```promql
node_memory_MemTotal_bytes
node_memory_MemFree_bytes
```
## Niveau 2 – Agrégations
### Exercice 5 : Utilisation CPU totale par instance
```promql
sum by(instance) (node_cpu_seconds_total)
```
### Exercice 6 : Pourcentage CPU utilisé
```promql
100 * (1 - sum by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) /
         sum by(instance)(rate(node_cpu_seconds_total[5m])))
```
### Exercice 7 : Mémoire utilisée par instance
```promql
node_memory_MemTotal_bytes - node_memory_MemFree_bytes
```
### Exercice 8 : Disque utilisé
```promql
node_filesystem_size_bytes - node_filesystem_free_bytes
```
### Exercice 9 : Top 3 systèmes les plus chargés (CPU user)
```promql
topk(3, rate(node_cpu_seconds_total{mode="user"}[5m]))
```
## Niveau 3 – Fonctions temporelles
### Exercice 10 : Taux CPU sur 5 minutes
```promql
rate(node_cpu_seconds_total{mode="user"}[5m])
```
### Exercice 11 : Moyenne CPU sur 1 heure
```promql
avg_over_time(node_cpu_seconds_total{mode="user"}[1h])
```
### Exercice 12 : Max mémoire utilisée sur 24h
```promql
max_over_time((node_memory_MemTotal_bytes - node_memory_MemFree_bytes)[24h:1m])
```
### Exercice 13 : Écart type des lectures disque
```promql
stddev_over_time(node_disk_read_bytes_total[10m])
```
## Niveau 4 – Comparaison et ratios
### Exercice 14 : Ratio lectures / écritures disque
```promql
rate(node_disk_read_bytes_total[5m]) / rate(node_disk_written_bytes_total[5m])
```
### Exercice 15 : Mémoire utilisée / totale
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes
```
### Exercice 16 : Disques presque pleins (<10%)
```promql
(node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
```
### Exercice 17 : CPU vs I/O
```promql
rate(node_cpu_seconds_total{mode="user"}[5m]) /
rate(node_disk_read_bytes_total[5m])
```
## Niveau 5 – Alerting / avancé
### Exercice 18 : Alerte CPU trop élevé (>80% sur 5 min)
```promql
100 * (1 - sum by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) /
         sum by(instance)(rate(node_cpu_seconds_total[5m]))) > 80
```
### Exercice 19 : Alerte disque presque plein (<10%)
```promql
(node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
```
### Exercice 20 : Alerting mémoire faible (<15%)
```promql
(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.15
```
## Astuces

- sum by(instance) et avg by(instance) permettent d’agréger par serveur.
- rate() s’utilise sur les compteurs (*_total).
- max_over_time, min_over_time, avg_over_time, stddev_over_time permettent d’analyser une série dans le temps.
- topk(n, ...) est utile pour identifier les serveurs les plus chargés.
