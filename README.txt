yum update -y
yum -y install yum-utils device-mapper-persistent-data lvm2

//Install Docker Composer
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl start docker
systemctl enable docker

curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

//Check version Docker Composer
docker-compose --version

//Create file
mkdir monitor_docker
cd monitor_docker/

//Create docker-compose.yml

version: '3.8'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}

services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    expose:
      - 9100
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    expose:
      - 9090
    networks:
      - monitoring

  grafana:
     image: grafana/grafana:latest
     container_name: grafana
     restart: always
     ports:
      - 3000:3000
     depends_on:
      - prometheus
      
//password Grafana: admin:admin

//Create prometheus.yml

global:
  scrape_interval: 20s
  scrape_timeout: 15s

scrape_configs:
- job_name: cadvisor
  static_configs:
  - targets:
    - cadvisor:8080

- job_name: node_exporter
  static_configs:
  - targets:
    - node-exporter:9100


docker-compose up -d

