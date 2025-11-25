# 🧪 TP PromQL avec Node Exporter

Ce TP utilise uniquement les métriques fournies par **Node Exporter** pour pratiquer PromQL.

---

## Niveau 1 – Découverte des métriques

### Exercice 1 : Lister toutes les métriques CPU
- Afficher toutes les métriques commençant par `node_cpu`.

---

### Exercice 2 : Filtrer par état
- Afficher uniquement le temps CPU utilisé (`mode="user"`).

---

### Exercice 3 : Afficher le nombre de CPU par instance
- Utiliser `count(node_cpu{mode="user"}) by (instance)`.

---

### Exercice 4 : Mémoire totale et libre
- Afficher `node_memory_MemTotal_bytes` et `node_memory_MemFree_bytes` pour toutes les instances.

---

## Niveau 2 – Agrégations

### Exercice 5 : Utilisation CPU totale par instance
- Somme de toutes les modes CPU (`user`, `system`, `idle`, …) par instance.

---

### Exercice 6 : Pourcentage CPU utilisé
- Calculer `1 - (idle / total)` pour chaque instance.

---

### Exercice 7 : Mémoire utilisée par instance
- Calculer `used = total - free` pour chaque serveur.

---

### Exercice 8 : Disque utilisé
- Calculer l’espace disque utilisé par filesystem : `node_filesystem_size_bytes - node_filesystem_free_bytes`.

---

### Exercice 9 : Top 3 systèmes les plus chargés
- Utiliser `topk(3, rate(node_cpu_seconds_total{mode="user"}[5m]))`.

---

## Niveau 3 – Fonctions temporelles

### Exercice 10 : Taux CPU sur 5 minutes
- Utiliser `rate(node_cpu_seconds_total{mode="user"}[5m])`.

---

### Exercice 11 : Moyenne CPU sur 1 heure
- `avg_over_time(node_cpu_seconds_total[1h])`.

---

### Exercice 12 : Max mémoire utilisée sur 24h
- `max_over_time((node_memory_MemTotal_bytes - node_memory_MemFree_bytes)[24h:1m]) by (instance)`.

---

### Exercice 13 : Écart type des lectures disque
- `stddev_over_time(rate(node_disk_read_bytes_total[10m])) by (instance)`.

---

## Niveau 4 – Comparaison et ratios

### Exercice 14 : Ratio lectures / écritures disque
- `rate(node_disk_read_bytes_total[5m]) / rate(node_disk_written_bytes_total[5m])`.

---

### Exercice 15 : Mémoire utilisée / totale
- `(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes`.

---

### Exercice 16 : Disques presque pleins
- Afficher les filesystems avec moins de 10% d’espace libre.

---

### Exercice 17 : CPU vs I/O
- Comparer `rate(node_cpu_seconds_total{mode="user"}[5m])` avec `rate(node_disk_read_bytes_total[5m])`.

---

## Niveau 5 – Alerting / avancé

### Exercice 18 : Alerte CPU trop élevé
- Définir une règle pour CPU `>80%` pendant 5 minutes.

---

### Exercice 19 : Alerte disque presque plein
- Définir une règle si `node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1`.

---

### Exercice 20 : Alerting mémoire faible
- Définir une règle si `node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes < 0.15` pendant 10 minutes.

---

### Astuces :

- Utiliser `sum by(instance)` ou `avg by(instance)` pour agréger par serveur.  
- Utiliser `rate()` pour métriques cumulatives comme `*_total`.  
- Les fonctions `max_over_time`, `min_over_time`, `avg_over_time`, `stddev_over_time` sont utiles pour l’analyse temporelle.  

---

# 🎯 Objectif

Ces exercices permettent de :

1. Explorer les métriques Node Exporter.  
2. Pratiquer les agrégations et filtres par label.  
3. Utiliser des fonctions temporelles pour analyser des séries historiques.  
4. Construire des alertes basiques sur CPU, mémoire et disque.  

---
