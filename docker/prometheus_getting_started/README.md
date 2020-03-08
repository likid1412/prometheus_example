### Create Getting started Prometheus docker

1. Create `prometheus_getting_started` folder, 

```
mkdir prometheus_getting_started && cd prometheus_getting_started
```

2. Create `prometheus.rules.yml` from [Getting started | Prometheus][1]

```yml
groups:
- name: example
  rules:
  - record: job_service:rpc_durations_seconds_count:avg_rate5m
    expr: avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```

2. Create `prometheus.yml` from [Getting started | Prometheus][1]

```yml
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

  - job_name:       'example-random'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```

4. Create `Dockerfile`

```Dockerfile
FROM prom/prometheus

# Add in the configuration file from the local directory.
ADD prometheus.yml /etc/prometheus/prometheus.yml
ADD prometheus.rules.yml /etc/prometheus/prometheus.rules.yml
```

list file

```
# ls
Dockerfile  prometheus.rules.yml  prometheus.yml
```

5. build docker image

```
docker build -t prometheus_getting_started .
```

6. RUN

```
docker run -rm -p 9090:9090 prometheus_getting_started
```

visit `http://your_host_ip:9090` or `http://localhost:9090`
