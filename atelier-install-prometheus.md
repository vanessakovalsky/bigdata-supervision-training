TP1 : Installation et premier scrape

**Objectif** : Installer Prometheus et superviser le premier target

**Étape 1** : Installation Node Exporter (sur une machine cible)
```bash
# Télécharger Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz

# Extraire
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz

# Copier
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/

# Créer service systemd
sudo cat > /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/node_exporter \
  --collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)

Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Démarrer
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Vérifier
curl http://localhost:9100/metrics | head -20
```

**Étape 2** : Configurer Prometheus
```yaml
# Ajouter dans prometheus.yml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

**Étape 3** : Valider
```bash
# Recharger Prometheus
curl -X POST http://localhost:9090/-/reload

# Vérifier dans l'UI Web
# http://localhost:9090/targets
# Le target doit être "UP"

# Tester une requête
curl 'http://localhost:9090/api/v1/query?query=up'
```
