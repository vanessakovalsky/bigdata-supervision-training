# 9.2 TP2 : Requêtes PromQL avancées

**Objectif** : Maîtriser PromQL avec des cas d'usage réels

**Exercice 1 : Utilisation CPU par nœud (en %)**
```promql
# CPU utilisé (hors idle) en %
100 - (
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)

# Ou version simplifiée
100 - (
  avg by(instance) (
    irate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

**Exercice 2 : Mémoire disponible par nœud (en %)**
```promql
(
  node_memory_MemAvailable_bytes 
  / 
  node_memory_MemTotal_bytes
) * 100
```

**Exercice 3 : Top 5 tables Cassandra par latence**
```promql
topk(5,
  cassandra_table_readlatency_99thpercentile{
    keyspace!~"system.*"
  }
)
```

**Exercice 4 : Taux d'erreur MongoDB**
```promql
(
  rate(mongodb_op_counters_total{type="command",legacy_op_type="error"}[5m])
  /
  rate(mongodb_op_counters_total{type="command"}[5m])
) * 100
```

**Exercice 5 : Prédiction saturation disque**
```promql
# Disque plein dans combien d'heures ?
(
  node_filesystem_avail_bytes{mountpoint="/data"}
  /
  (
    -1 * deriv(node_filesystem_avail_bytes{mountpoint="/data"}[1h])
  )
) / 3600

# Alerte si < 24h
(
  node_filesystem_avail_bytes{mountpoint="/data"}
  /
  (-1 * deriv(node_filesystem_avail_bytes{mountpoint="/data"}[1h]))
) / 3600 < 24
```

**Exercice 6 : Agrégation multi-niveaux**
```promql
# Latence P99 moyenne par datacenter
avg by(datacenter) (
  cassandra_client_request_read_latency_99thpercentile
)

# Puis top 3 des datacenters
topk(3,
  avg by(datacenter) (
    cassandra_client_request_read_latency_99thpercentile
  )
)
```
