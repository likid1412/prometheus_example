# prometheus getting started with docker

[TOC]

## Getting Started

You can getting started with the [Getting started | Prometheus][1], but this guid is not based on docker. Let's getting started prometheus with docker. More detail, please visit [Getting started | Prometheus][1].

## Prometheus configuration

Create prometheus configuration `prometheus.yml`, rule files `prometheus.rules.yml`. You can find  `prometheus.yml` and  `prometheus.rules.yml` contents in [Getting started | Prometheus][1].

**prometheus.yml**

```yaml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

  # Attach these extra labels to all timeseries collected by this Prometheus instance.
  external_labels:
    monitor: 'codelab-monitor'

rule_files:
  - 'prometheus.rules.yml'

scrape_configs:
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'example-random'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['example-random0:8080', 'example-random1:8081']
        labels:
          group: 'production'

      - targets: ['example-random2:8082']
        labels:
          group: 'canary'
```

The example-random targets is running in docker now, so we should replace `localhost` with docker container name (config in **docker-compose.yml** ) in `example-random` job targets, suck as  `example-random0`.

**prometheus.rules.yml**

```yaml
groups:
- name: example
  rules:
  - record: job_service:rpc_durations_seconds_count:avg_rate5m
    expr: avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```

## Docker Compose configuration

Create Docker Compose configuration `docker-compose.yml`

**docker-compose.yml**

```yaml
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./config/prometheus:/etc/prometheus
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
    - 9093:9093
  example-random0:
    image: prom/client_golang_examples_simple:beorn7_release
    container_name: example-random0
    ports:
    - 8080:8080
    entrypoint: /random
    command:
    - -listen-address=:8080
  example-random1:
    image: prom/client_golang_examples_simple:beorn7_release
    container_name: example-random1
    ports:
    - 8081:8081
    entrypoint: /random
    command:
    - -listen-address=:8081
  example-random2:
    image: prom/client_golang_examples_simple:beorn7_release
    container_name: example-random2
    ports:
    - 8082:8082
    entrypoint: /random
    command:
    - -listen-address=:8082
```

contents of directories look like this

```
prometheus_getting_started/
├── config
│   └── prometheus
│       ├── prometheus.rules.yml
│       └── prometheus.yml
└── docker-compose.yml
```

To run the installation:

```sh
docker-compose up
```

*ps: you can add `-d` option to run containers in the background, just like this `docker-compose up -d`*

## Exploring the web UI

Check the targets state in `http://localhost:909/targets`.

![image-20200314171904158](prometheus_getting_started_with_docker_targets.png)

Try `job_service:rpc_durations_seconds_count:avg_rate5m` metric in `http://localhost:9090/graph`.

![image-20200314173212322](prometheus_getting_started_with_docker_graph.png)



ref:

[Getting started | Prometheus][1]

[Monitoring Docker container metrics using cAdvisor | Prometheus][2]



[1]: https://prometheus.io/docs/prometheus/latest/getting_started/
[2]: https://prometheus.io/docs/guides/cadvisor/
