#Docker Engine install on all nodes: 

Steps of Installation Docker and Docker Compose on new machines 

sudo apt update 
 sudo apt install apt-transport-https ca-certificates curl software-properties-common 

sudo apt update 

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
 if error come try this  

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.gpg > /dev/null 
 echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null 
 sudo apt update  

sudo apt install docker-ce 
 sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
 sudo chmod +x /usr/local/bin/docker-compose 
 docker-compose --version  

sudo usermod -aG docker devops 
 groups $USER 
 sudo systemctl restart docker 
 sudo systemctl status docker 
 sudo reboot  

To verify all setup up and run 

docker --version 

docker-compose  --version 

docker pull hello-world:latest 

docker run hello-world 

 

Setup Memcached On server:  

Steps to setup memcached on server  

1) Update system: 

 
 sudo apt update 

 sudo apt upgrade –y 

 OR 

sudo apt update && sudo apt upgrade -y 

2) Install Memcached and Start: 

 sudo apt install memcached libmemcached-tools –y 

 sudo systemctl start memcached 

3) Check Memcached Status: 

 sudo systemctl status memcached 

Optional: netstat -tuln | grep 11211 
 sudo nano /etc/memcached.conf 

For default port change: 

-p 55421 

For default IP change: 

-l 127.0.0.1  # Default value (localhost only) 

-l 0.0.0.0  # Listen on all network interfaces or 
 -l 192.168.1.100  # Specific IP address to allow access from 

-l 127.0.0.1,192.168.4.14  # multiple IP  

4) Restart Memcached after configuration changes: 

sudo systemctl restart memcached  

netstat -tuln | grep 55421 
OR 

memcstat --servers=<ip>:<port> 
 OR 
 telnet 127.0.0.1 11211 

stats 

Grafana And Prometheus Monitoring Setup 

 

Run Memcached Exporter and Node Exporter on Server of Memcached: 

Memcached Exporter: 

sudo docker run -d --name memcached-exporter --restart always -p 9150:9150 prom/memcached-exporter --memcached.address=10.100.142.141:55421 

Node Exporter (For Server) : 

docker run -d --name=node-exporter --restart always -p 9100:9100 prom/node-exporter 

Prometheus And Grafana Setup in Docker 

Step 1: Create a Directory for Configuration 

 
 -mkdir memcached-monitoring 
 -cd memcached-monitoring 
 -mkdir prometheous-data 
 -cd prometheous-data/ 
 -pwd (Copy Path from the result) 
 -sudo chown -R 65534:65534 <path-to-prometheus-folder> 
 -sudo chmod -R 777 <path-to-prometheus-folder> 
 -cd ..  

   Step 2: Cert Setup 

Mkdir certs 

Cd certs  

And move your certs to this folder. 

 

Step 2: Create docker-compose.yml File 

 Open the file for editing: 
 -nano docker-compose.yml 
 Add the following content: 
 services: 
   prometheus: 
     image: prom/prometheus 
     container_name: prometheus 
     restart: always 
     ports: 
       - "8091:9090" 
     volumes: 
       - ./prometheus.yml:/etc/prometheus/prometheus.yml 
       - <path-to-prometheus-folder>:/prometheus 

 
     command: 
       - "--config.file=/etc/prometheus/prometheus.yml" 
       - "--storage.tsdb.retention.time=30d"  # 30 days data retention 
     deploy: 
       resources: 
         limits: 
           memory: 2g   # Max 2GB RAM usage 
           cpus: "2"  # Max 2 CPU cores 
     networks: 
       - monitoring 
   grafana: 
     image: grafana/grafana 
     container_name: grafana 
     restart: always 
     ports: 
       - "2010:3000" 
     volumes: 
       - grafana_data:/var/lib/grafana 

     - ./certs:/etc/grafana/certs  # Mount SSL certs 
     depends_on: 
       - prometheus 
     environment: 
       - GF_SECURITY_ADMIN_USER=SuperAdmin 
       - GF_SECURITY_ADMIN_PASSWORD=Grafana_DevOps@1 

       - GF_SMTP_ENABLED=true 
       - GF_SMTP_HOST=10.100.14 87.105:7025 
       - GF_SMTP_FROM_ADDRESS=grafanaalerts@vitonta.com 
       - GF_SMTP_FROM_NAME="Grafana Alerts" 

   -GF_SERVER_PROTOCOL=https 
       - GF_SERVER_HTTP_PORT= 2010 
       - GF_SERVER_CERT_FILE=/etc/grafana/certs/certi.pem 
       -GF_SERVER_CERT_KEY=/etc/grafana/certs/privatekey.pem 
 
     deploy: 
       resources: 
         limits: 
           memory: 2g   # Max 2GB RAM usage 
           cpus: "2"    # Max 2 CPU core 
     networks: 
       - monitoring 
 volumes: 
   grafana_data: 
   prometheus_data: 
 
 networks: 
   monitoring: 

  

Step 3: Create the Prometheus Configuration File 

 Open the file for editing 
 nano prometheus.yml 
 Add the following content: 
global: 

  scrape_interval: 1m 

scrape_configs: 

  - job_name: 'indici-memcached' 

    static_configs: 

      - targets: 

        - 10.100.142.55:9150  # Memcached Exporter instance 1 

        - 10.100.142.56:9151  # Memcached Exporter instance 2 

  - job_name: 'indici-memcached-npdserver' 

    static_configs: 

      - targets: 

        - 10.100.142.55:9100 

        - 10.100.142.56:9100 

       # - 10.100.142.139:9100 

Step 4: Start the Containers 

 -docker-compose up -d 
 -docker ps 

Step 5: Restart Prometheus 

 If Prometheus is already running, restart it to apply the new configuration: 
 - 

Step 6: Verify the Monitoring Setup 

Check Prometheus Targets 
 Open your browser and visit: 
http://10.100.142.55:8091/targets 
 
 Login to Grafana 
 Open the browser and visit: 
http://10.100.142.56:2010 
