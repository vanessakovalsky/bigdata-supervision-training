# 5.3 TP2 : Dashboard multi-services avec variables

**Objectif** : Dashboard unifié pour Cassandra, MongoDB et Hadoop

**Configuration variables** :
```
Variable 1 - Service:
Name: service
Type: Custom
Options: cassandra,mongodb,hadoop
Multi-value: No

Variable 2 - Cluster:
Name: cluster
Type: Query
Query: label_values(up{job=~"$service"}, cluster)
Multi-value: No

Variable 3 - Instance:
Name: instance
Type: Query
Query: label_values(up{job=~"$service",cluster="$cluster"}, instance)
Multi-value: Yes
Include All: Yes
```

**Panels conditionnels** :
```
Panel 1 - Latence (affiché si service=cassandra):
Query: cassandra_client_request_read_latency_mean{cluster="$cluster",instance=~"$instance"}
Repeat: None
Panel hidden when: $service != "cassandra"

Panel 2 - Replication Lag (affiché si service=mongodb):
Query: mongodb_mongod_replset_oplog_lag_seconds{cluster="$cluster",instance=~"$instance"}
Panel hidden when: $service != "mongodb"
```
