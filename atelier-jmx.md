# ✅ TP — Supervision Big Data via **JMX**  
### 💻 Adapté pour environnement Windows 10/11 + Docker Desktop + Docker Compose

Ce TP est une version adaptée de l’atelier JMX afin qu’il soit **simplement exécutable sous Windows**, sans installation locale de Java, Hadoop ou Spark.  
Tout tourne dans **Docker Compose**, même l’application exposant JMX ✅

---

## 🎯 Objectifs du TP

À la fin de ce TP, vous serez capable de :

✅ Activer JMX sur une application Java  
✅ Exposer des MBeans (compteurs, latence, ressources…)  
✅ Démarrer l’application dans Docker  
✅ Se connecter en JMX depuis Windows (JConsole / VisualVM)  
✅ Observer et analyser les métriques JMX en temps réel

---

## 1️⃣ Prérequis

- Windows 10/11
- Docker Desktop installé et démarré
- WSL2 activé (recommandé)
- PowerShell
- JDK installé sur Windows **OU** JDK embarqué dans VisualVM

👉 Vérifier Docker :

```powershell
docker --version
docker compose version
```

---

## 2️⃣ Créer le dossier du TP

```powershell
mkdir C:\tp-jmx
cd C:\tp-jmx
```

---

## 3️⃣ Créer l’application Java exposant des métriques JMX

Créer `MyApp.java` :

```java
import javax.management.*;
import java.lang.management.*;
import java.util.Random;

public class MyApp implements MyAppMBean {
    private Random rand = new Random();
    private int requestCount = 0;

    @Override
    public int getRequestCount() {
        return requestCount;
    }

    @Override
    public double getAvgLatencyMs() {
        return rand.nextDouble() * 300;
    }

    public void run() throws Exception {
        while (true) {
            requestCount++;
            Thread.sleep(2000);
        }
    }

    public static void main(String[] args) throws Exception {
        MBeanServer mbs = ManagementFactory.getPlatformMBeanServer();
        MyApp app = new MyApp();
        ObjectName name = new ObjectName("bigdata:type=MyApp");
        mbs.registerMBean(app, name);
        app.run();
    }
}

interface MyAppMBean {
    int getRequestCount();
    double getAvgLatencyMs();
}
```

---

## 4️⃣ Créer le Dockerfile Java

Créer `Dockerfile` :

```Dockerfile
FROM eclipse-temurin:17-jdk

WORKDIR /app
COPY MyApp.java .

RUN javac MyApp.java

EXPOSE 9999

CMD ["java",
     "-Dcom.sun.management.jmxremote",
     "-Dcom.sun.management.jmxremote.port=9999",
     "-Dcom.sun.management.jmxremote.rmi.port=9999",
     "-Dcom.sun.management.jmxremote.local.only=false",
     "-Dcom.sun.management.jmxremote.authenticate=false",
     "-Dcom.sun.management.jmxremote.ssl=false",
     "-Djava.rmi.server.hostname=localhost",
     "MyApp"]
```

✅ Ce conteneur démarre une appli Java
✅ expose JMX en clair sur le port **9999**

---

## 5️⃣ Créer le `docker-compose.yml`

Créer :

```yaml
version: "3.8"

services:
  jmx-app:
    build: .
    container_name: jmx-demo
    restart: unless-stopped
    ports:
      - "9999:9999"
```

---

## 6️⃣ Lancer l’environnement

```powershell
docker compose up -d --build
```

Vérifier :

```powershell
docker ps
```

✅ L’application est en cours d’exécution  
✅ Elle expose des MBeans via JMX sur `localhost:9999`

---

## 7️⃣ Installer VisualVM (si pas déjà présent)

Télécharger :  
https://visualvm.github.io/

Aucune installation Java supplémentaire requise si JDK déjà présent ✅

---

## 8️⃣ Se connecter en JMX depuis Windows

1. Ouvrir **VisualVM**
2. Clic droit → **Add JMX Connection**
3. Renseigner :

```
localhost:9999
```

4. Valider ✅

---

## 9️⃣ Explorer les MBeans

Dans VisualVM → **MBeans** → `bigdata:type=MyApp`

✔ `RequestCount` augmente automatiquement  
✔ `AvgLatencyMs` varie dans le temps

📌 Ce sont des métriques typiques de supervision Big Data :
- volume de requêtes traitées
- latence moyenne
- métriques système exposées via JMX

---

## 🔟 (Optionnel) Tester arrêt/redémarrage

```powershell
docker compose restart
docker compose down
```

Les métriques reprennent automatiquement ✅

---

## 1️⃣1️⃣ Dépannage

❌ Impossible de se connecter en JMX ?

✅ Vérifier que le port est ouvert :

```powershell
Test-NetConnection -Port 9999 localhost
```

✅ Vérifier que le conteneur tourne :

```powershell
docker logs jmx-demo
```

✅ Vérifier que l'antivirus ne bloque pas la JVM RMI

---

## 1️⃣2️⃣ Nettoyage

```powershell
docker compose down
```

Supprimer le dossier :

```powershell
Remove-Item -Recurse -Force C:\tp-jmx
```

---

# ✅ Fin du TP 🎉

Vous avez appris à :

✅ créer une application Java instrumentée  
✅ exposer des métriques via JMX  
✅ containeriser l’application  
✅ accéder aux MBeans depuis Windows  
✅ analyser les métriques en temps réel

---

## 🚀 Pour la suite (recommandé)

- Envoyer ces métriques JMX vers Graphite ou Prometheus
- Superviser un cluster Hadoop / Spark / Kafka
- Ajouter alerting & dashboards via Grafana
- Utiliser JMX Exporter ou Jolokia
