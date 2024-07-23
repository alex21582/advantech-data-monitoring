# A simple data collection and monitoring system based on Advantech equipment (WISE-6610 gateway and WISE-2410 vibration and temperature sensors) setup via docker

Based on https://github.com/Miceuz/docker-compose-mosquitto-influxdb-telegraf-grafana

This docker compose installs and sets up:
- [InfluxDB](https://www.influxdata.com/) - The Time Series Data Platform to store your data in time series database 
- [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) - The open source server agent to connect Mosquitto and InfluxDB together
- [Grafana](https://grafana.com/) - The open observability platform to draw some graphs and more

# Setup process
## Install docker
### Add Docker's official GPG key:

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add the repository to Apt sources:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### Install the Docker packages.

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Verify that the Docker Engine installation is successful by running the hello-world image.

```
sudo docker run hello-world
```

## Install docker-compose

```
sudo apt install docker-compose
```

### Add user

```
sudo usermod -aG docker <user>
```

## Coonect swarm for secrets

```
docker swarm init
```

## Clone this repository

```
git clone https://github.com/alex21582/advantech-data-monitoring.git
cd advantech-data-monitoring
```

## Create files with passwords and token

```
mkdir secrets && \
    echo "your_grafana_password" >> ./secrets/grafana_password.txt && \
    echo "your_influxdb_password" >> ./secrets/influxdb_password.txt && \
    echo "your_influxdb_token" >> ./secrets/influxdb_token.txt && \
    echo "your_mqtt_broker_password" >> ./secrets/mqtt_password.txt
```

## Create secrets

```
docker secret create grafana_password ./secrets/grafana_password.txt && \
    docker secret create influxdb_password ./secrets/influxdb_password.txt && \
    docker secret create influxdb_token ./secrets/influxdb_token.txt && \
    docker secret create mqtt_password ./secrets/mqtt_password.txt
```

## Run it

To download, setup and start all the services run
```
sudo docker-compose up -d
```

To check the running setvices run
```
sudo docker ps
```

To shutdown the whole thing run
```
sudo docker-compose down
```

### Grafana
Open in your browser: 
`http://<your-server-ip>:3000`

Username and pasword are admin:<your_grafana_password>. You should see dashboard WISE-2410 with data from broker on WISE-6610.

### InfluxDB
You can poke around your InfluxDB setup here:
`http://<your-server-ip>:8086`
Username and password are admin:<your_influxdb_password>

# Configuration 
### Telegraf 
Telegraf is responsible for piping mqtt messages to influxdb. You should change the IP address and port in the configuration file to those that the WISE-6610 gateway has on your local network.
```
[[inputs.mqtt_consumer]]
  servers = ["tcp://<your-server-ip>:<port>"]
 
```

### Grafana data source 
Grafana is provisioned with a default data source pointing to the InfluxDB instance installed in this same compose. The configuration file is `grafana-provisioning/datasources/automatic.yml`.

### Grafana dashboard
Default Grafana dashboard is also set up in this directory: `grafana-provisioning/dashboards`

