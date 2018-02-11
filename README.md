# Postgres Monitoring on centos
Repository explaining how to install tools for monitoring postgresql on centos with node_exporter, postgres_exporter, prometheus &amp; grafana

### Install docker on centos
It's easier to handle dependancies and stuff with docker so install it like described here:
https://docs.docker.com/engine/installation/linux/docker-ce/centos/

Add docker-compose:
https://docs.docker.com/compose/install/#install-compose (Linux section)

### Clone this repo

### Find your postgres connection string (or run your own postgres)
For my simple setup I run 2 postgres databases in docker container with basic security:

`docker run -p 5432:5432 -it --rm -e POSTGRES_PASSWORD=password postgres:9.6.6`
and another one on a different port:
`docker run -p 5433:5432 -it --rm -e POSTGRES_PASSWORD=password postgres:9.6.6`

so I have 2 postgres databases running in docker container with port `5432` and port `5433` exposed on the host, user `postgres` and password `password`

The connection strings looks like that then:
`postgresql://postgres:password@<IP>:5432/?sslmode=disable`
`postgresql://postgres:password@<IP>:5433/?sslmode=disable`

Or you need to find out your own working connection string.
Then, put it to the `docker-compose.yml` file:

```
version: '3.2'
services:
  postgres_exporter_one:
    image: wrouesnel/postgres_exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://postgres:password@10.48.136.82:5432/?sslmode=disable"
  postgres_exporter_two:
    image: wrouesnel/postgres_exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://postgres:password@10.48.136.82:5433/?sslmode=disable"      
  prometheus:
...
```  
So, basically 1 postgres_exporter per database.

I have also added node_exporter that runs on the node where you run this docker-compose.
You can also run node_exporter on other hosts with or without docker like this:

```
## Prerequisites:
Go compiler
RHEL/CentOS: glibc-static package.

## Building:
go get github.com/prometheus/node_exporter
cd ${GOPATH-$HOME/go}/src/github.com/prometheus/node_exporter
make
./node_exporter```

Then you adapt `prometheus.yml` file accordingly with postgres_exporters and node_exporters.

### Run the whole stack of tools
Run the whole stack of tools with a single command:
`docker-compose up -d`

#### It should start 3 components:
- exporter of postgres metrics
- local exporter of node metrics
- prometheus database
- grafana frontend

You should assure that postgres_exporter connected to postgres db:
`docker-compose logs postgres_exporter`
You should find:
`level=info msg="Established new database connection."`

### Configure Grafana

You can reach Grafana on http://localhost:3000 (or host's IP).
Login with admin/admin

#### Grafana logo -> Datasources -> Add data source:
* name: 'prometheus' (important to use this name for further steps)
* type: Prometheus
* URL: `http://prometheus:9090`
* Add.

### Postgres dashboard
## Grafana logo -> Dashboards -> Import
* paste JSON included in this repo
* Import
* in the top-left corner put host: `postgres_exporter_one:9187` (whatever exporter you want to reach)

### Another Postgres dashboard
## Grafana logo -> Dashboards -> Import
* paste dashboard number 3300
* Import
* in the top-left corner put host: `postgres_exporter_two:9187` (whatever exporter you want to reach)

### Node exporter dashboard
## Grafana logo -> Dashboards -> Import
* paste dashboard number 405
* Import
* in the top-left corner put host: `192.168.1.2:9100` (whatever node you want to reach)

### Cleaning up
In case you want to clean up just run
`docker-compose down`
From the directory where `docker-compose.yml` file is placed

### Reload prometheus configuration with:
`curl -X POST http://localhost:9090/-/reload`
