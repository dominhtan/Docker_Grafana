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

docker-compose up -d

// Change password Grafana with Docker.
ADMIN_PASSWORD="{password-set}";CONTAINER_ID=$(docker ps -qf "name=^grafana$");docker exec -it $CONTAINER_ID grafana-cli admin reset-admin-password $ADMIN_PASSWORD

// Query Metrics
node_memory_MemFree_bytes
node_cpu_seconds_total
node_filesystem_avail_bytes
node_load5
rate(node_cpu_seconds_total{mode="system"}[1m]) 
rate(node_network_receive_bytes_total[1m])
