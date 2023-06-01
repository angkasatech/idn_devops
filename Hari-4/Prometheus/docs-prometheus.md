# Prometheus

## Install Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.37.8/prometheus-2.37.8.linux-amd64.tar.gz
tar -zxvf prometheus-2.37.8.linux-amd64.tar.gz

mv -v prometheus-2.37.8.linux-amd64 /etc/prometheus
cd /etc/prometheus

vim prometheus.yml
./prometheus --config.file=prometheus.yml

http://localhost:9090/

prometheus_target_interval_length_seconds

## Setup Exporter
### Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar -zxvf node_exporter-1.6.0.linux-amd64.tar.gz

mv -v node_exporter-1.6.0.linux-amd64.tar.gz /etc/node_exporter
cd /etc/node_exporter

./node_exporter --web.listen-address ip-address:port

http://ip-address:port/metrics

vim prometheus.yml

- job_name: "Server monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.11:8000", "10.23.0.12:8001", "10.23.0.13:8003"]
        labels:
          group: 'devops'

./prometheus --config.file=prometheus.yml

http://localhost:9090/

rate(node_cpu_seconds_total{mode="system"}[1m])

### Docker Cadvisor Exporter
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
  gcr.io/cadvisor/cadvisor:v0.36.0

http://ip-address:port/

vim prometheus.yml

- job_name: "docker monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.12:8008"]
        labels:
          group: 'docker'

./prometheus --config.file=prometheus.yml

http://localhost:9090/

rate(container_cpu_usage_seconds_total{name="redis"}[1m])

### Nginx Exporter
vim default.conf
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

docker container run -d --rm -p 9113:9113 --name nginx-exporter nginx/nginx-prometheus-exporter -nginx.scrape-uri http://ipaddress:port/metrics

vim prometheus.yml

- job_name: "nginx monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.12:9113"]
        labels:
          group: 'nginx'

./prometheus --config.file=prometheus.yml

http://localhost:9090/

nginx_connections_active
## Setup rule Alert
### Alert Server is down
### Alert CPU and memory > 50%
### Alert Request > 1000 request