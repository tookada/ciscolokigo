# Cisco LokiGO
## Overview
Cisco LokiGO is an application to consolidate and visualize important logs from each product show tech file. You can put show tech file of each product then LokiGo will automatically analyze them and make Grafana dashboard available for you.
![Cisco LokiGO Overview](https://github.com/tookada/ciscolokigo/blob/images/overview.png?raw=true)

The dashboard includes:

- Multiple graphs based on logging messages analyzed
- All product logging messages are consolidated and sorted by timestamp
- Search capability with labels(product type, log type), free keyword
![Cisco LokiGO Dashboard](https://github.com/tookada/ciscolokigo/blob/images/dashboard.png?raw=true)

## Archtecture
Cisco LokiGO is developed as Kubernetes application. You can install it with 1 single command line. But there is an option to install without Kubernetes with multiple docker containers. You can install it on your laptop too.
![Cisco LokiGO Architecture](https://github.com/tookada/ciscolokigo/blob/images/architecture.png?raw=true)

## Run on Docker
```
git clone https://github.com/tookada/ciscolokigo.git
cd ciscolokigo
docker run -v $(pwd):/mnt/config --name promtail -d -v $(pwd)/log:/var/log grafana/promtail:1.6.0 -config.file=/mnt/config/promtail-config.yaml
docker run -d -v $(pwd):/mnt/config -p 3100:3100 --name loki grafana/loki:1.6.0 -config.file=/mnt/config/loki-config.yaml
docker run -p 8089:8089 -d --name lokigo -v $(pwd)/log:/var/log  tookadacisco/lokigo:prod
docker run -d -p 3000:3000 -v $(pwd)/datastore:/etc/grafana/provisioning/datasources -v $(pwd)/dashboard:/etc/grafana/provisioning/dashboards -v $(pwd)/json-config:/etc/dashboards --name grafana -e "GF_INSTALL_PLUGINS=grafana-piechart-panel" grafana/grafana:7.5.7
```
The application can be accessed on [http://127.0.0.1:8089](http://127.0.0.1:8089)
The default username and password for Grafana is admin/admin.

## Deploy on Kubernetes
```
kubectl create -f https://raw.githubusercontent.com/tookada/ciscolokigo/main/lokigo.yaml
```
The application can be accessed on [http://master or worker IP address:30089](http://master or worker IP address:30089)