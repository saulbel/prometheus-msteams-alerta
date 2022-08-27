# Prometheus-msteams-alerta
## Prerequisites
* `Docker`
* `Docker-compose`
* `Msteams`

## Project structure
```
prometheus-msteams-alerta
|── prometheus
|    └── alertmanager_msteams.yml
|    └── docker-compose.yml
|    └── prometheus.yml
|    └── rules.yml
|    └── msteams.yml
|    └── grafana/provisioning/datasources
|         └── prometheus_ds.yml    
└── scripts
     └── setup.sh
```
## Tasks to accomplish
- The main idea of this project is to show how we can monitorize our infra in an easy way using `docker-compose`.
- Configure `prometheus`, `alertmanager`, `node-export` ,`grafana` and `alerta.io` using `docker-compose`.
- Use `Msteams` to recieve the alerts.

## How to install the tools
I have included a custom script `setup.sh` that allows you to install `docker` and `docker-compose` on `Debian`.
I recommend to download it and change it with your `username` because I have decided to add my `user` to `docker group`. It is a good practice to run docker with a user instead of as `root`.

## How to setup this project locally
- First we should download it with either `git clone` or as `.zip`.
- Then we will modify `/scripts/setup.sh` with our `username` and we will execute it.

## First task: configure the monitoring stack
- In order to do this we are gonna use `docker-compose.yml`. The whole idea of using `docker-compose` is to simplify it as much as posible because we could install all these tools as services. So where are gonna use `docker images` and `volumes` to both install and put the config files in containers.
- Once we have all ready, we just have to:

````
$ docker-compose up -d
Creating network "prometheus_default" with the default driver
Creating prometheus_db_1           ... done
Creating prometheus_grafana_1      ... done
Creating prometheus_nodeexporter_1 ... done
Creating prometheus_alertmanager_1 ... done
Creating prometheus-msteams        ... done
Creating prometheus_prometheus_1   ... done
Creating prometheus_alerta_1       ... done
````
- To check the status of the containers:
````
$ docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                                                 NAMESheus_grafana_1
69fdb09ae534   prom/alertmanager:latest         "/bin/alertmanager -…"   30 minutes ago   Up 22 seconds   0.0.0.0:9093->9093/tcp, :::9093->9093/tcp             prometheus_alertmanager_1
38d3d6f5a779   alerta/alerta-web                "docker-entrypoint.s…"   33 minutes ago   Up 20 seconds   1717/tcp, 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   prometheus_alerta_1
547158640dcf   bzon/prometheus-msteams:latest   "/sbin/tini -- /prom…"   33 minutes ago   Up 21 seconds   0.0.0.0:2000->2000/tcp, :::2000->2000/tcp             prometheus-msteams
7a096c01edb9   prom/node-exporter:latest        "/bin/node_exporter …"   33 minutes ago   Up 22 seconds   0.0.0.0:9100->9100/tcp, :::9100->9100/tcp             prometheus_nodeexporter_1
3547210a9e33   postgres                         "docker-entrypoint.s…"   33 minutes ago   Up 24 seconds   5432/tcp                                              prometheus_db_1
064a558ff9aa   prom/prometheus:latest           "/bin/prometheus --c…"   33 minutes ago   Up 20 seconds   0.0.0.0:9090->9090/tcp, :::9090->9090/tcp             prometheus_prometheus_1
357372cef8b8   grafana/grafana:latest           "/run.sh"                33 minutes ago   Up 23 seconds   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp             prometheus_grafana_1
````

## Second task: configure Msteams
- Now we will just have to install `Msteams` and create a channel. Then we will just create a `webhook` for that channel and we will include it in `msteams.yml`. We can include as many `webhooks` as we want on this file. So we can have different channels for each type of alerts. For example, let's say we have a production environment and we like to have a channel for each type of alerts: `crital` and `warning`.

## To sum up
Now we can have access to all these tools with a web browser.
- To access prometheus
````
localhost:9090
````
- To access alertmanager
````
localhost:9093
````
- To access alerta
````
localhost:8080
````
- To access grafana
````
localhost:3000
````
Let´s explain a little bit more about how this stack works. 
- `Prometheus` --> this tool allows to scrap data from servers and stores it in time-series format.
- `Alertmanager` --> taking the data we have in Prometheus, we are able to create alerts. These alerts can be sent to programs such us `telegram`, `teams` or in this case `slack`. We can use email to send this alerts too.
- `Alerta` --> this is a frontend for Alertmanager. It allows us to visualize alerts much better than in Alertmanager.
- `Node-exporter` --> this is the agent in charge of exposing the metrics in the server/container for `prometheus` to scrape from. There are more exporters, for example we can use `win-exporter` if our server is a windows one.
- `Grafana` --> this software allows us to create dashboards to have all our data configured in a good looking way. It will connect to prometheus in order to do so.
- `Msteams` --> if your company uses this communication tool, you can propose it as an alert reciever too. With this set up all the alerts go to two different channels but we could configure more. We can use tags in `rules.yml` to change the severity or even the environment of the alerts and we can send them to one channel or another.

