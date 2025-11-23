# Exercice pratique : Conception d'architecture

### Énoncé

**Contexte** : Vous êtes architecte dans une entreprise de e-commerce. Voici les caractéristiques de l'infrastructure Big Data :

```
📊 INFRASTRUCTURE
- 80 serveurs physiques
- 3 datacenters (Paris, Londres, Francfort)
- Technologies : Hadoop (HDFS, YARN), Kafka, Cassandra, MongoDB
- 500 microservices sur Kubernetes
- Volume : 2TB de logs/jour, 50K métriques/min

🎯 BESOINS
- Rétention métriques : 90 jours détaillées, 2 ans agrégées
- Rétention logs : 30 jours searchable, 1 an archivés
- Haute disponibilité requise (SLA 99.9%)
- Alerting multi-canal (Slack, PagerDuty, email)
- 10 utilisateurs Grafana (équipes dev, ops, business)

💰 BUDGET
- ~5000€/mois infrastructure
- Préférence open source
- Possibilité licences commerciales si ROI justifié

👥 ÉQUIPE
- 2 SRE expérimentés
- 3 ops niveau intermédiaire
- Support management pour formation
```

### Questions

**1. Quelle stack proposez-vous ?** (justifiez)

**2. Dessinez l'architecture détaillée** (composants, flux, redondance)

**3. Estimez les ressources nécessaires** (CPU, RAM, stockage)

**4. Proposez un planning de déploiement** (3 mois max)

**5. Identifiez 3 risques principaux** et leurs mitigations
