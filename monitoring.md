# 📊 Monitoring - Prometheus & Grafana

Guide complet pour monitorer vos infrastructures avec Prometheus et Grafana.

---

## 📚 Introduction

**Stack de monitoring moderne :**
- **Prometheus** : Collecte et stockage métriques (time-series DB)
- **Grafana** : Visualisation et dashboards
- **Alertmanager** : Gestion des alertes
- **Exporters** : Collecteurs de métriques (node, mysql, nginx, etc.)

---

## 🔥 Prometheus

### Installation

```bash
# Docker
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Kubernetes
kubectl create namespace monitoring
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# Binary (Linux)
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
cd prometheus-*
./prometheus --config.file=prometheus.yml
```

### Configuration prometheus.yml

```yaml
# prometheus.yml
global:
  scrape_interval: 15s          # Fréquence de collecte
  evaluation_interval: 15s      # Évaluation rules/alerts
  external_labels:
    cluster: 'production'
    region: 'eu-west-1'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 'alertmanager:9093'

# Rules files
rule_files:
  - 'alerts/*.yml'

# Scrape configurations
scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Node Exporter (métriques serveur)
  - job_name: 'node'
    static_configs:
      - targets:
          - 'server1:9100'
          - 'server2:9100'
        labels:
          environment: 'production'
  
  # Service discovery (Kubernetes)
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
  
  # BlackBox Exporter (checks HTTP/TCP/DNS)
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://api.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
  
  # MySQL Exporter
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']
        labels:
          database: 'production-db'
  
  # Custom application metrics
  - job_name: 'myapp'
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
    metrics_path: '/metrics'
```

### PromQL - Query Language

```promql
# Métriques courantes

# CPU usage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
(node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_avail_bytes{mountpoint="/"}) / node_filesystem_size_bytes{mountpoint="/"} * 100

# Network traffic (bytes/sec)
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])

# HTTP request rate
rate(http_requests_total[5m])

# HTTP errors (5xx)
sum(rate(http_requests_total{status=~"5.."}[5m]))

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Count instances up
up{job="node"}

# Group by label
sum(rate(http_requests_total[5m])) by (method, status)

# Top 5 CPU consumers
topk(5, 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100))

# Predict disk full (linear regression)
predict_linear(node_filesystem_avail_bytes{mountpoint="/"}[1h], 4*3600) < 0
```

### Recording Rules

```yaml
# rules/recording_rules.yml
groups:
  - name: node_rules
    interval: 30s
    rules:
      # CPU usage percentage
      - record: instance:node_cpu:avg_rate5m
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
      
      # Memory usage percentage
      - record: instance:node_memory_usage:percentage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
      
      # Disk usage percentage
      - record: instance:node_disk_usage:percentage
        expr: (node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_avail_bytes{mountpoint="/"}) / node_filesystem_size_bytes{mountpoint="/"} * 100

  - name: http_rules
    interval: 15s
    rules:
      # HTTP request rate per method
      - record: job:http_requests:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job, method)
      
      # HTTP error rate
      - record: job:http_errors:rate5m
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) by (job)
```

### Alerting Rules

```yaml
# rules/alerts.yml
groups:
  - name: instance_alerts
    rules:
      # Instance down
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} has been down for more than 5 minutes."
      
      # High CPU
      - alert: HighCPUUsage
        expr: instance:node_cpu:avg_rate5m > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"
      
      # High Memory
      - alert: HighMemoryUsage
        expr: instance:node_memory_usage:percentage > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"
      
      # Disk space low
      - alert: DiskSpaceLow
        expr: instance:node_disk_usage:percentage > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk usage is {{ $value }}%"
      
      # Disk will fill in 4 hours
      - alert: DiskWillFillSoon
        expr: predict_linear(node_filesystem_avail_bytes{mountpoint="/"}[1h], 4*3600) < 0
        for: 30m
        labels:
          severity: warning
        annotations:
          summary: "Disk will fill soon on {{ $labels.instance }}"
          description: "Disk is predicted to fill in less than 4 hours"

  - name: app_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: (sum(rate(http_requests_total{status=~"5.."}[5m])) by (job) / sum(rate(http_requests_total[5m])) by (job)) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.job }}"
          description: "Error rate is {{ $value | humanizePercentage }}"
      
      # Slow response time
      - alert: SlowResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Slow response time on {{ $labels.job }}"
          description: "95th percentile is {{ $value }}s"
```

---

## 🚨 Alertmanager

### Configuration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'

# Templates
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Routing
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  
  routes:
    # Critical alerts → PagerDuty
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true
    
    # Warning alerts → Slack
    - match:
        severity: warning
      receiver: 'slack'
    
    # Database alerts
    - match:
        service: database
      receiver: 'dba-team'
      group_wait: 30s

# Inhibition rules (suppress alerts)
inhibit_rules:
  # Inhibit warning if critical firing
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']

# Receivers
receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
  
  - name: 'slack'
    slack_configs:
      - channel: '#monitoring'
        send_resolved: true
        title: '[{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          *Alert:* {{ .Labels.alertname }}
          *Severity:* {{ .Labels.severity }}
          *Summary:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          {{ end }}
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'your-pagerduty-key'
        description: '{{ .GroupLabels.alertname }}'
  
  - name: 'email'
    email_configs:
      - to: 'devops@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'alerts@example.com'
        auth_password: 'password'
        headers:
          Subject: '[ALERT] {{ .GroupLabels.alertname }}'
  
  - name: 'webhook'
    webhook_configs:
      - url: 'http://webhook.example.com/alert'
        send_resolved: true
  
  - name: 'dba-team'
    slack_configs:
      - channel: '#dba-alerts'
```

---

## 📈 Grafana

### Installation

```bash
# Docker
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin" \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana-oss

# Kubernetes (Helm)
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana \
  --set adminPassword=admin \
  --set service.type=LoadBalancer

# Binary (Linux)
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana

# Démarrer Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Accès : http://localhost:3000
# Login : admin / admin
```

### Configuration Data Source (Prometheus)

```yaml
# provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    jsonData:
      httpMethod: POST
      timeInterval: 15s
```

### Dashboard Provisioning

```yaml
# provisioning/dashboards/dashboard.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

### Dashboard JSON (exemple)

```json
{
  "dashboard": {
    "title": "Node Exporter Dashboard",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "{{ instance }}"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
            "legendFormat": "{{ instance }}"
          }
        ]
      }
    ]
  }
}
```

**Dashboards populaires (Grafana.com) :**
- **Node Exporter Full** : ID 1860
- **Kubernetes Cluster** : ID 7249
- **MySQL Overview** : ID 7362
- **Nginx** : ID 12660
- **Docker** : ID 893

**Importer dashboard :**
```
Grafana UI → Dashboards → Import → Enter ID
```

---

## 📊 Exporters

### Node Exporter (Métriques Serveur)

```bash
# Installation
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
cd node_exporter-*
./node_exporter

# Systemd service
cat > /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

# Métriques disponibles : http://localhost:9100/metrics
```

### MySQL Exporter

```bash
# Docker
docker run -d \
  --name mysqld-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(mysql:3306)/" \
  prom/mysqld-exporter

# Config prometheus.yml
- job_name: 'mysql'
  static_configs:
    - targets: ['localhost:9104']
```

### Nginx Exporter

```bash
# Activer stub_status dans Nginx
server {
    listen 8080;
    location /stub_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}

# Exporter
docker run -d \
  --name nginx-exporter \
  -p 9113:9113 \
  nginx/nginx-prometheus-exporter:latest \
  -nginx.scrape-uri=http://nginx:8080/stub_status
```

### Blackbox Exporter (Probes HTTP/TCP/ICMP)

```yaml
# blackbox.yml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: []  # Défaut: 2xx
      method: GET
      follow_redirects: true
      preferred_ip_protocol: "ip4"
  
  http_post_2xx:
    prober: http
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{"key": "value"}'
  
  tcp_connect:
    prober: tcp
    timeout: 5s
  
  icmp:
    prober: icmp
    timeout: 5s

# Docker
docker run -d \
  --name blackbox-exporter \
  -p 9115:9115 \
  -v /path/to/blackbox.yml:/etc/blackbox_exporter/config.yml \
  prom/blackbox-exporter
```

### Custom Application Metrics

**Python (Flask) :**
```python
from flask import Flask
from prometheus_client import Counter, Histogram, generate_latest, REGISTRY
import time

app = Flask(__name__)

# Métriques
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint']
)

@app.route('/api/users')
def users():
    start = time.time()
    
    # Logic
    result = {"users": []}
    
    # Metrics
    REQUEST_COUNT.labels(method='GET', endpoint='/api/users', status=200).inc()
    REQUEST_LATENCY.labels(method='GET', endpoint='/api/users').observe(time.time() - start)
    
    return result

@app.route('/metrics')
def metrics():
    return generate_latest(REGISTRY)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

**Node.js (Express) :**
```javascript
const express = require('express');
const promClient = require('prom-client');

const app = express();
const register = new promClient.Registry();

// Métriques par défaut (CPU, memory, etc.)
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.labels(req.method, req.route?.path || req.path, res.statusCode).observe(duration);
    httpRequestTotal.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(8080);
```

---

## 🎯 Exemple Complet - Stack de Monitoring

```yaml
# docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    restart: unless-stopped

  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    volumes:
      - /:/host:ro,rslave
    restart: unless-stopped

  blackbox-exporter:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-exporter
    ports:
      - "9115:9115"
    volumes:
      - ./blackbox:/etc/blackbox_exporter
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
```

---

## 📱 Notifications

### Slack Webhook

```yaml
# alertmanager.yml
receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'
        username: 'Alertmanager'
        icon_emoji: ':warning:'
        title: |
          [{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.alertname }}
        text: |
          {{ range .Alerts }}
          *Severity:* `{{ .Labels.severity }}`
          *Summary:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          {{ end }}
```

### Email

```yaml
receivers:
  - name: 'email'
    email_configs:
      - to: 'alerts@example.com'
        from: 'prometheus@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'alerts@example.com'
        auth_password: 'app-password'
        auth_identity: ''
        headers:
          Subject: '[{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}'
        html: |
          <!DOCTYPE html>
          <html>
            <body>
              <h2>{{ .GroupLabels.alertname }}</h2>
              {{ range .Alerts }}
              <p>
                <strong>Severity:</strong> {{ .Labels.severity }}<br>
                <strong>Instance:</strong> {{ .Labels.instance }}<br>
                <strong>Summary:</strong> {{ .Annotations.summary }}<br>
                <strong>Description:</strong> {{ .Annotations.description }}
              </p>
              {{ end }}
            </body>
          </html>
```

---

## 🎓 Bonnes Pratiques

1. **Retention appropriée** : 15-30 jours pour Prometheus, long-term dans Thanos/Cortex
2. **High Availability** : Multiple Prometheus instances
3. **Service Discovery** : Automatique (Kubernetes, Consul, EC2)
4. **Recording Rules** : Pré-calculer queries lourdes
5. **Alert Grouping** : Grouper par service/cluster
6. **Meaningful Alerts** : Actionnable, pas de noise
7. **SLO/SLI** : Définir objectives et indicators
8. **Tagging** : Labels consistants (environment, service, cluster)

---

**🎉 Félicitations ! Vous maîtrisez maintenant le monitoring complet avec Prometheus & Grafana !**
