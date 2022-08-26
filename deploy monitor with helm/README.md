# MONITOR KUBERNETES WITH HELM

## 1. Install prometheus and blackbox-exporter:
```
# View namesapces on cluster
kubectl get namespace 

# Create new namespace:
kubectl create namespace monitoring

# Next step requires helm installed
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
helm install blackbox-exporter prometheus-community/prometheus-blackbox-exporter -n monitoring
```

## 2. Login grafana with default password
## 3. Install prometheus-msteams:
```
helm repo add prometheus-msteams https://prometheus-msteams.github.io/prometheus-msteams/ 
```
Create a values.yaml file and pass the following values to the file.
```
connectors: 
  - alert: urlwebhook-msteams
metrics:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: prometheus-stack
    scrapeInterval: 30s
```
Then install:
```
helm upgrade --install prometheus-msteams --namespace monitoring -f values.yml prometheus-msteams/prometheus-msteams
```
Create an alert manager configuration file value-msteams.yml:
```
alertmanager:
  enabled: true
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['job', 'alertname', 'priority']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'prometheus-msteams' 
      routes:
      - match:
          alertname: Watchdog
        receiver: 'null'
    receivers:
      - name: 'null'
      - name: 'prometheus-msteams'
        webhook_configs:
        - url: "http://endpoint-msteams/alert"
          send_resolved: true
```
```
helm upgrade prometheus-stack prometheus-community/kube-prometheus-stack -n monitor  --values=value-msteams.yml
```
## Add blackbox-exporter to config:
Create file blackbox-exporter.yml:
```
config:
  modules:
    http_2xx:
      http:
        follow_redirects: true
        fail_if_not_ssl: false
        preferred_ip_protocol: ip4
        method: GET
        valid_http_versions:
        - HTTP/1.1
        - HTTP/2.0
        valid_status_codes:
        - 200
      prober: http
      timeout: 5s
serviceMonitor:
  enabled: true
  defaults:
    labels: 
      release: prometheus-stack
    interval: 30s
    scrapeTimeout: 30s
    module: http_2xx
  scheme: http
  tlsConfig:
    insecureSkipVerify: false
  targets:
    - name: url.com
      url: https://url.com/
```
Apply blackbox-exporter.yml:
```
helm upgrade blackbox-exporter prometheus-community/prometheus-blackbox-exporter -n monitoring --values=blackbox-exporter.yml
```
Create file prometheus-alert-blackbox-exporter.yml:
```
additionalScrapeConfigs:
  - job_name: blackbox
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://url.com
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: http://endpoint-blackbox-exporter:9115
```
Add ID to grafana dashboard: [grafana-13659](https://grafana.com/grafana/dashboards/13659-blackbox-exporter-http-prober/)
Create rule on prometheus:
```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: endpoint
  namespace: monitoring
  labels:
    release: prometheus-stack     
spec:
  groups:
  - name: endpoint
    rules:
    - alert: EndpointDown
      expr: probe_success == 0
      for: 10s
      labels:
        severity: "critical"
      annotations:
        summary: "Endpoint {{ $labels.instance }} down"
```


