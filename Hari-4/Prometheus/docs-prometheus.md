# Prometheus

## Install Prometheus
Download prometheus package
```
wget https://github.com/prometheus/prometheus/releases/download/v2.37.8/prometheus-2.37.8.linux-amd64.tar.gz
```

Extract prometheus package
```
tar -zxvf prometheus-2.37.8.linux-amd64.tar.gz
mv -v prometheus-2.37.8.linux-amd64 /etc/prometheus
```

Running and configuration prometheus
```
cd /etc/prometheus
vim prometheus.yml
./prometheus --config.file=prometheus.yml
```

Access prometheus ui
```
http://localhost:9090/
```

Query prometheus
```
prometheus_target_interval_length_seconds
```

## Setup Exporter
### Node Exporter
Download node exporter package
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
```

Extract node exporter package
```
tar -zxvf node_exporter-1.6.0.linux-amd64.tar.gz
mv -v node_exporter-1.6.0.linux-amd64 /etc/node_exporter
```

Start Service
```
cd /etc/node_exporter
./node_exporter --web.listen-address ip-address:port
```

Access node metric
```
http://ip-address:port/metrics
```
>Access node metric
```
http://172.23.10.34:1313/metrics
>Ctrl+Z (to pause)
>bg (to start in backgrount
```

Config prometheus
```
vim prometheus.yml
```

Add Targets
```
- job_name: "Server monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.11:8000", "10.23.0.12:8001", "10.23.0.13:8003"]
        labels:
          group: 'devops'
```
```
  - job_name: "Server monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["172.23.10.14:1111", "172.23.10.24:1212", "172.23.10.34:1414"]
        labels:
          group: 'devops'
```

Start service
```
./prometheus --config.file=prometheus.yml
```

Access prometheus ui
```
http://localhost:9090/
```

Query metric
```
rate(node_cpu_seconds_total{mode="system"}[1m])
```

### Docker Cadvisor Exporter (INSTALL di docker.. bukan di server prometheus)
Running cAdvisor in a Docker Container
```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:v0.49.1
```

### Config prometheus (di prometheus)
```
vim prometheus.yml
```

Add targets
```
- job_name: "docker monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.12:8080"]
        labels:
          group: 'docker'
```

Start Service
```
./prometheus --config.file=prometheus.yml
```
cAdvisor is now running (in the background) on
```
http://ip-address:port/
```

Access prometheus ui
```
http://localhost:9090/
```

Query metric
```
rate(container_cpu_usage_seconds_total{name="redis"}[1m])
```
### Nginx Exporter (di lokasi nginx, dalam hal ini di docker server 14)
Edit nginx config
```
vim default.conf
```

Add metrics 
```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }    
    // add this
    location /metrics {
        stub_status;
    }
}
```

### Deploy nginx exporter (LAKUKAN di server docker berada, case ini di server 14)
```
docker container run -d --rm -p 9113:9113 --name nginx-exporter nginx/nginx-prometheus-exporter -nginx.scrape-uri http://ipaddress:port/metrics
>docker container run -d --rm -p 9113:9113 --name nginx-exporter nginx/nginx-prometheus-exporter -nginx.scrape-uri http://ipaddress:80/metrics
```
```
docker container run -d --rm -p 9113:9113 --name nginx-exporter nginx/nginx-prometheus-exporter -nginx.scrape-uri http://172.23.10.14/metrics
```

### Config prometheus (lanjut di server prometheus 34)
```
vim prometheus.yml
```

Add targets
```
- job_name: "nginx monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.12:9113"]
        labels:
          group: 'nginx'
```

Start Service
```
./prometheus --config.file=prometheus.yml
```

Access prometheus ui
```
http://localhost:9090/
```

Query metric
```
nginx_connections_active
```
## Setup rule
### Setup basic rule
```
vim prometheus.rules.yml
```

```
groups:
  - name: cpu-node
    rules:
    - record: job_instance_mode:node_cpu_seconds:avg_rate5m
      expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

```
vim prometheus.yml
```

```
rule_files:
  - prometheus.rules.yml
```

```
./prometheus --config.file=prometheus.yml
```

## Setup Alert
### Alert Server is down
```
vim server.rules.yml
```

```
groups:
- name: server_down
  rules:
  - alert: server_down
    expr: up == 0
    for: 1m
    labels:
      severity: page
    annotations:
      summary: Server are down.
```

```
vim prometheus.yml
```

```
rule_files:
  - prometheus.rules.yml
  - server.rules.yml
```

```
./prometheus --config.file=prometheus.yml
```

### Alert CPU and memory > 50%
```
vim cpu.rules.yml
```

```
groups:
  - name: example_alerts
    rules:
    - alert: HighCPUUtilization
      expr: avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[1m]) * 100) < 50
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: High CPU utilization on host {{ $labels.instance  }}
        description: The CPU utilization on host {{ $labels.instance }} has exceeded 50 for 1 minutes.
```

```
vim prometheus.yml
```

```
rule_files:
  - prometheus.rules.yml
  - server.rules.yml
  - cpu.rules.yml
```

```
./prometheus --config.file=prometheus.yml
```
