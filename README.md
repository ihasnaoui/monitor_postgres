# Postgres Monitoring on centos
Repository explaining how to install tools for monitoring postgresql on centos with postgres_exporter, prometheus &amp; grafana

### Install docker on centos
It's easier to handle dependancies and stuff with docker so install it like described here:
https://docs.docker.com/engine/installation/linux/docker-ce/centos/

Add docker-compose:
https://docs.docker.com/compose/install/#install-compose (Linux section)

### Clone this repo

### Check postgres connection string (or run your own postgres)
For my simple setup I run postgres in docker container with basic security:
`docker run -p 5432:5432 -it --rm -e POSTGRES_PASSWORD=password postgres:9.6.6`
so I have a postgres running in docker container with port `5432` exposed on the host, user `postgres` and password `password`

The connection string looks like that then:
`postgresql://postgres:password@<IP>:5432/?sslmode=disable`

Or you need to find out your own working connection string.
Then, put it to the `docker-compose.yml` file:

```
version: '3.2'
services:
  postgres_exporter:
    image: wrouesnel/postgres_exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://postgres:password@10.48.136.82:5432/?sslmode=disable"
  prometheus:
...
```  

### Run the whole stack of tools
Run the whole stack of tools with a single command:
`docker-compose up -d`

It should start 3 components:
- exporter of postgres metrics
- prometheus database
- grafana frontend

You should assure that postgres_exporter connected to postgres db:
`docker-compose logs postgres_exporter`
You should find:
`level=info msg="Established new database connection."`

### Configure Grafana

Grafana logo -> Datasources -> Add data source:
* name: 'prometheus' (important to use this name for further steps)
* type: Prometheus
* URL: http://prometheus:9090
Add

Grafana logo -> Dashboards -> Import
* paste JSON included in this repo
* Import
* in the top-left corner put Host: `postgres_exporter:9187`
