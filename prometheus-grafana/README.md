## DevOpsLab
### Set Prometheus & Grafana in one-single node installation

Project structure:
```
.
├── docker-compose.yaml
├── grafana
│   └── datasource.yaml
├── prometheus
│   └── prometheus.yaml
│   └── targets
│       └── prometheus.json
│       └── grafana.json
│       └── node-exporter.json
├── alertmanager
│   └── alertmanager.yaml
│   └── alertmanager.conf
└── README.md
```

[_docker-compose.yaml_](docker-compose.yaml)
```
services:
  prometheus:
    image: prom/prometheus
    ...
    ports:
      - 9090:9090
  grafana:
    image: grafana/grafana
    ...
    ports:
      - 3000:3000
  alertmanager:
    image: prom/alertmanager
    ...
    ports:
      - 9093:9093
  node-exporter:
    image: prom/node-exporter
    ...
    ports:
      - 9100:9100
```
This compose file will create a stack with two services: `prometheus` and `grafana`; 
Additionally, the compose adds an exporter (node-exporter) as an example of an external source of information
and set the AlertManager for managing the alerts that are handled by the Prometheus Server.
When the stack is been deployed, the docker-compose maps port the default ports for each service 
to the equivalent ports on the host in order to inspect easier the web interface of each service.
IMPORTANT NOTICE: Ports 9090, 3000, 9093 and 9100 on the host MUST NOT already in use.

## Deploy with docker-compose

```
$ docker-compose up -d
Building with native build....
Pulling prometheus (prom/prometheus:)...
Pulling grafana (grafana/grafana:)...
Pulling node-exporter (prom/node-exporter)...
Pulling alertmanager (prom/alertmanager)...
...
Creating prometheus ... done
Creating grafana    ... done
Creating node-exporter ... done
Creating alertmanager ... done
Attaching to prometheus, grafana, node-exporter-1, alertmanager
```

## Expected result

Listing containers must show two containers running and the port mapping as below:
```
$ docker ps
CONTAINER ID   IMAGE               COMMAND                   CREATED           STATUS         PORTS                    NAMES
45e9b302d0f0   prom/prometheus     "/bin/prometheus --c..."   12 seconds ago   Up 10 seconds  0.0.0.0:9090->9090/tcp   prometheus
164f0553ed54   grafana/grafana     "/run.sh"                  12 seconds ago   Up 9 seconds   0.0.0.0:3000->3000/tcp   grafana
c4d479c609c6   prom/node-exporter  "/bin/node-exporter ..."   10 seconds ago   Up 9 seconds   0.0.0.0:9100->9100/tcp   node-exporter-1
b2e0f0f3c6f4   prom/alertmanager   "/bin/alertmanager -..."   10 seconds ago   Up 9 seconds   0.0.0.0:9093->9093/tcp   alertmanager
```

Then you can launch each application using the below links in your local web browser:

* Prometheus: [`http://localhost:9090`](http://localhost:9090) collecting information from three targets (grafana, exporter and itself)
* Grafana: [`http://localhost:3000`](http://localhost:3000) and use the login credentials specified in the compose file to access Grafana. Please be aware that is already configured with prometheus as the default datasource.
* AlertManager: [`http://localhost:9093`](http://localhost:9093) to verify the status of the Alerting system
* Node-exporter-1: [`htps://localhost:9100`](http://localhost:9100) and you see the list of data that is collected

Stop and remove the containers. Use `-v` to remove the volumes if looking to erase all data.
```
$ docker-compose down -v
```