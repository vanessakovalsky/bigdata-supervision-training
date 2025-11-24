# 4.2 TP1 : Créer un dashboard Cassandra

**Objectif** : Dashboard complet de supervision Cassandra

**Étape 1 : Créer le dashboard**
```
1. Cliquer sur "+" → Dashboard
2. Nommer: "Cassandra - Production Cluster"
3. Ajouter description: "Supervision complète du cluster Cassandra production"
4. Tags: cassandra, production, database
5. Save
```

**Étape 2 : Ajouter variables**
```
Settings → Variables → Add variable

Variable 1 - Cluster:
- Name: cluster
- Type: Query
- Data source: Prometheus
- Query: label_values(up{job="cassandra"}, cluster)
- Multi-value: Yes
- Include All: Yes

Variable 2 - Node:
- Name: node
- Type: Query
- Data source: Prometheus
- Query: label_values(up{job="cassandra",cluster="$cluster"}, instance)
- Multi-value: Yes
- Include All: Yes
```

**Étape 3 : Row 1 - KPIs**
```
Panel 1 - Active Nodes:
- Type: Stat
- Query: count(up{job="cassandra",cluster="$cluster"} == 1)
- Title: "Active Nodes"
- Thresholds: < 3 (red), < 5 (yellow), >= 5 (green)

Panel 2 - Total Load:
- Type: Stat
- Query: sum(cassandra_storage_load_bytes{cluster="$cluster"}) / 1024 / 1024 / 1024
- Title: "Total Data (GB)"
- Unit: GB

Panel 3 - Avg Read Latency:
- Type: Stat
- Query: avg(cassandra_client_request_read_latency_mean{cluster="$cluster"}) / 1000
- Title: "Avg Read Latency"
- Unit: ms
- Thresholds: < 10 (green), < 50 (yellow), >= 50 (red)

Panel 4 - Total Timeouts:
- Type: Stat
- Query: sum(rate(cassandra_client_request_timeouts_total{cluster="$cluster"}[5m]))
- Title: "Timeouts/sec"
- Thresholds: 0 (green), > 0 (red)
```

**Étape 4 : Row 2 - Latence**
```
Panel 5 - Read Latency P99:
- Type: Time series
- Query A: cassandra_client_request_read_latency_99thpercentile{cluster="$cluster",instance=~"$node"} / 1000
- Legend: {{instance}}
- Title: "Read Latency P99 (ms)"
- Y-axis: milliseconds

Panel 6 - Write Latency P99:
- Type: Time series
- Query: cassandra_client_request_write_latency_99thpercentile{cluster="$cluster",instance=~"$node"} / 1000
- Legend: {{instance}}
- Title: "Write Latency P99 (ms)"
```

**Étape 5 : Row 3 - Ressources**
```
Panel 7 - JVM Heap:
- Type: Time series
- Query A (Used): jvm_memory_heap_used{job="cassandra",instance=~"$node"} / 1024 / 1024 / 1024
- Query B (Max): jvm_memory_heap_max{job="cassandra",instance=~"$node"} / 1024 / 1024 / 1024
- Title: "JVM Heap (GB)"
- Fill: 20%
- Stack: None

Panel 8 - Compaction:
- Type: Time series
- Query: cassandra_compaction_pending_tasks{cluster="$cluster",instance=~"$node"}
- Legend: {{instance}}
- Title: "Pending Compactions"
```

**Étape 6 : Row 4 - Table Overview**
```
Panel 9 - Nodes Table:
- Type: Table
- Query: up{job="cassandra",cluster="$cluster",instance=~"$node"}
- Transformations:
  - Organize fields
  - Rename: instance → Node, Value → Status
- Title: "Nodes Status"
- Overrides:
  - Status: 1 = "🟢 UP", 0 = "🔴 DOWN"
```
