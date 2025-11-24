# TP Pratique : Supervision JMX avec Docker (45 min)

### 🎯 Objectif du TP
Déployer une application Java (Kafka) avec Docker Compose et la superviser via JMX avec jconsole et jmxterm.

---

### 5.1 Prérequis

#### Installation requise :
- ✅ Docker Desktop pour Windows
- ✅ JDK 11+ installé (contient jconsole)
- ✅ jmxterm téléchargé

#### Vérification :
```bash
docker --version
java -version
```

---

### 5.2 Architecture du TP

```
┌─────────────────────────────────────────┐
│         Docker Compose                  │
│  ┌────────────────┐  ┌────────────────┐ │
│  │   Zookeeper    │  │     Kafka      │ │
│  │   :2181        │  │   :9092        │ │
│  │                │  │   JMX: 9999    │ │
│  └────────────────┘  └────────────────┘ │
└─────────────────────────────────────────┘
                │
                │ JMX Port 9999
                ▼
         ┌─────────────┐
         │  jconsole   │
         │  jmxterm    │
         └─────────────┘
```

---

### 5.3 Étape 1 : Créer le projet

#### Structure des fichiers
```
tp-jmx/
├── docker-compose.yml
├── jmxterm-1.0.4-uber.jar
└── scripts/
    └── check-kafka-metrics.jmx
```

---

### 5.4 Étape 2 : Fichier docker-compose.yml

Créez le fichier `docker-compose.yml` :

```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    networks:
      - kafka-network

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9999:9999"  # Port JMX
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      
      # Configuration JMX
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_JMX_OPTS: >
        -Dcom.sun.management.jmxremote
        -Dcom.sun.management.jmxremote.authenticate=false
        -Dcom.sun.management.jmxremote.ssl=false
        -Dcom.sun.management.jmxremote.rmi.port=9999
        -Djava.rmi.server.hostname=localhost
    networks:
      - kafka-network

networks:
  kafka-network:
    driver: bridge
```

---

### 5.5 Étape 3 : Démarrage des conteneurs

```bash
# Dans le dossier tp-jmx/
docker-compose up -d

# Vérifier que les conteneurs sont démarrés
docker-compose ps

# Voir les logs Kafka
docker-compose logs -f kafka
```

**✅ Attendez 30 secondes** que Kafka soit complètement démarré.

#### Vérification JMX
```bash
# Tester la connexion JMX
telnet localhost 9999
# Si connexion OK, taper Ctrl+] puis quit
```

---

### 5.6 Étape 4 : Supervision avec jconsole

#### Lancement
```bash
jconsole localhost:9999
```

#### 📋 Exercice guidé

**Mission 1 : Explorer la mémoire Heap**
1. Cliquez sur l'onglet **"Memory"**
2. Observez le graphique **"Heap Memory Usage"**
3. Notez la valeur **"Used"** actuelle : _________ MB

**Mission 2 : Trouver le nombre de threads**
1. Cliquez sur l'onglet **"Threads"**
2. Notez le **"Thread count"** : _________

**Mission 3 : Explorer les MBeans Kafka**
1. Cliquez sur l'onglet **"MBeans"**
2. Déroulez l'arborescence : `kafka.server` → `type=BrokerTopicMetrics`
3. Trouvez le MBean : `name=MessagesInPerSec`
4. Consultez l'attribut **"Count"** : _________

**Mission 4 : Déclencher le Garbage Collector**
1. Restez sur l'onglet **"MBeans"**
2. Déroulez : `java.lang` → `Memory`
3. Cliquez sur **"Operations"**
4. Cliquez sur le bouton **"gc"**
5. Retournez sur l'onglet **"Memory"** et observez la baisse de "Used"

---

### 5.7 Étape 5 : Supervision avec jmxterm

#### Script de surveillance

Créez le fichier `scripts/check-kafka-metrics.jmx` :

```bash
# Connexion
open localhost:9999

# Mémoire Java
echo "\n=== MEMOIRE HEAP ==="
domain java.lang
bean java.lang:type=Memory
get HeapMemoryUsage

# Threads
echo "\n=== THREADS ==="
bean java.lang:type=Threading
get ThreadCount

# Métriques Kafka
echo "\n=== KAFKA - MESSAGES IN ==="
domain kafka.server
bean kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
get Count
get OneMinuteRate

echo "\n=== KAFKA - MESSAGES OUT ==="
bean kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec
get Count
get OneMinuteRate

# Déconnexion
close
```

#### Exécution du script
```bash
# Windows PowerShell
java -jar jmxterm-1.0.4-uber.jar -n -i scripts\check-kafka-metrics.jmx

# Linux/Mac
java -jar jmxterm-1.0.4-uber.jar -n -i scripts/check-kafka-metrics.jmx
```

---

### 5.8 Étape 6 : Générer du trafic Kafka

Pour voir les métriques évoluer, créons du trafic.

#### Créer un topic
```bash
docker exec -it kafka kafka-topics --create \
  --topic test-jmx \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1
```

#### Produire des messages
```bash
# Ouvrir un producer
docker exec -it kafka kafka-console-producer \
  --topic test-jmx \
  --bootstrap-server localhost:9092
  
# Taper plusieurs messages (un par ligne) :
> Message 1
> Message 2
> Message 3
> Message 4
> Message 5
# Appuyer sur Ctrl+C pour quitter
```

#### Relancer le script jmxterm
```bash
java -jar jmxterm-1.0.4-uber.jar -n -i scripts/check-kafka-metrics.jmx
```

**✅ Observez :** Le `Count` de `MessagesInPerSec` a augmenté !

---

### 5.9 Étape 7 : Mode interactif jmxterm

```bash
# Lancer jmxterm en mode interactif
java -jar jmxterm-1.0.4-uber.jar

# Dans jmxterm :
$> open localhost:9999
$> domains
$> domain kafka.server
$> beans
$> bean kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
$> info
$> get Count
$> get OneMinuteRate
$> quit
```

---

### 5.10 Exercice libre (10 min)

#### 🎯 Mission : Trouver les métriques Under-Replicated Partitions

1. **Objectif :** Trouvez le MBean qui expose le nombre de partitions sous-répliquées
2. **Indice :** Le domaine est `kafka.server` et le type est `ReplicaManager`

**Avec jconsole :**
- Naviguez dans l'arborescence MBeans
- Trouvez le bon MBean
- Notez le nom complet : _________________________________

**Avec jmxterm :**
```bash
open localhost:9999
domain kafka.server
beans
# Cherchez le bean qui contient "UnderReplicated"
bean <nom_du_bean>
get Value
```

**✅ Solution :**
```
ObjectName: kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions
Attribut: Value (devrait être 0 si tout va bien)
```

---

### 5.11 Nettoyage

```bash
# Arrêter les conteneurs
docker-compose down

# Supprimer les volumes (optionnel)
docker-compose down -v
```
