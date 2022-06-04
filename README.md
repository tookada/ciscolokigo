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
The application can be accessed on [http://master or worker IP address:30089](http://masterorworkerIPaddress:30089)

## Notes
### Run Docker containers on Windows
You may need to put "/" before $(pwd) like this.
```
docker run -v /$(pwd):/mnt/config --name promtail -d -v $(pwd)/log:/var/log grafana/promtail:1.6.0 -config.file=/mnt/config/promtail-config.yaml
```

### Your environment needs HTTP proxy
If your environment needs HTTP proxy, Grafana dashborad import will fail. You need to configure proxy.  
For Kubernetes:
```
env:
- name: GF_INSTALL_PLUGINS
  value: "grafana-piechart-panel"
- name: http_proxy
  value: <Your proxy>
- name: https_proxy
  value: <Your proxy>
- name: no_proxy
  value: loki-service,cisco.com,127.0.0.1,localhost,<Your local subnet>
```
For Docker:  
```
dashboard:/etc/grafana/provisioning/dashboards -v $(pwd)/json-config:/etc/dashboards --name grafana -e "GF_INSTALL_PLUGINS=grafana-piechart-panel" -e http_proxy=<Your proxy> -e https_proxy=<Your proxy> -e no_proxy=loki-service,cisco.com,127.0.0.1,localhost,<Your local subnet> grafana/grafana:7.5.7
```
### Run Docker containers on Linux
Change the docker command like this:  
```
docker run -v $(pwd):/mnt/config --net=host --name promtail -d -v $(pwd)/log:/var/log grafana/promtail:1.6.0 -config.file=/mnt/config/promtail-config.yaml
docker run -d -v $(pwd):/mnt/config --net=host --name loki grafana/loki:1.6.0 -config.file=/mnt/config/loki-config.yaml
docker run -p 8089:8089 -d --name lokigo -v $(pwd)/log:/var/log  tookadacisco/lokigo:prod
docker run -d --net=host -v $(pwd)/datastore:/etc/grafana/provisioning/datasources -v $(pwd)/dashboard:/etc/grafana/provisioning/dashboards -v $(pwd)/json-config:/etc/dashboards --name grafana -e "GF_INSTALL_PLUGINS=grafana-piechart-panel" grafana/grafana:7.5.7
```
And replace host.docker.internal in the 2 files below with 127.0.0.1.
```
❯ grep -r http://host.docker.internal *
datastore/loki-datasource.yaml:  url: http://host.docker.internal:3100
promtail-config.yaml:  - url: http://host.docker.internal:3100/loki/api/v1/push
```

## License
This repository is under [GNU General Public License v3.0](https://github.com/tookada/ciscolokigo/blob/main/LICENSE).