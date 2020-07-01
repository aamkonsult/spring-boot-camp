---
layout: default
title: Metrics
parent: Spring Boot Actuator
nav_order: 5
permalink: docs/actuator/metrics/
---

# Metrics
{: .no_toc }

[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html) is a sub-project of the [Spring Boot](https://spring.io/projects/spring-boot) project.  It includes a number of additional features that help us monitor and manage our application using HTTP and JMX endpoints.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## metrics

```yaml
management:
  endpoints:
    web:
      base-path:
      exposure:
        include: env, health, info, metrics
```


```bash
$ curl "http://localhost:8080/metrics" | jq .
```


```yaml
{
  "names": [
    "hikaricp.connections",
    "hikaricp.connections.acquire",
    "hikaricp.connections.active",
    "hikaricp.connections.creation",
    "hikaricp.connections.idle",
    "hikaricp.connections.max",
    "hikaricp.connections.min",
    "hikaricp.connections.pending",
    "hikaricp.connections.timeout",
    "hikaricp.connections.usage",
    "http.server.requests",
    "jdbc.connections.active",
    "jdbc.connections.idle",
    "jdbc.connections.max",
    "jdbc.connections.min",
    "jvm.buffer.count",
    "jvm.buffer.memory.used",
    "jvm.buffer.total.capacity",
    "jvm.classes.loaded",
    "jvm.classes.unloaded",
    "jvm.gc.live.data.size",
    "jvm.gc.max.data.size",
    "jvm.gc.memory.allocated",
    "jvm.gc.memory.promoted",
    "jvm.gc.pause",
    "jvm.memory.committed",
    "jvm.memory.max",
    "jvm.memory.used",
    "jvm.threads.daemon",
    "jvm.threads.live",
    "jvm.threads.peak",
    "jvm.threads.states",
    "logback.events",
    "process.cpu.usage",
    "process.files.max",
    "process.files.open",
    "process.start.time",
    "process.uptime",
    "system.cpu.count",
    "system.cpu.usage",
    "system.load.average.1m",
    "tomcat.sessions.active.current",
    "tomcat.sessions.active.max",
    "tomcat.sessions.alive.max",
    "tomcat.sessions.created",
    "tomcat.sessions.expired",
    "tomcat.sessions.rejected"
  ]
}
```

```bash
$ curl "http://localhost:8080/metrics/http.server.requests" | jq .
```

```json
{
  "name": "http.server.requests",
  "description": null,
  "baseUnit": "seconds",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 3
    },
    {
      "statistic": "TOTAL_TIME",
      "value": 0.089996207
    },
    {
      "statistic": "MAX",
      "value": 0.01274303
    }
  ],
  "availableTags": [
    {
      "tag": "exception",
      "values": [
        "None"
      ]
    },
    {
      "tag": "method",
      "values": [
        "GET"
      ]
    },
    {
      "tag": "uri",
      "values": [
        "/env",
        "/health",
        "/metrics"
      ]
    },
    {
      "tag": "outcome",
      "values": [
        "SUCCESS"
      ]
    },
    {
      "tag": "status",
      "values": [
        "200"
      ]
    }
  ]
}
```

## prometheus

```groovy
dependencies {
  /* Prometheus */
  runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
}
```

```yaml
management:
  endpoints:
    web:
      base-path:
      exposure:
        include: env, health, info, metrics, prometheus
  endpoint:
    health:
      show-details: always
```

```bash
$ curl "http://localhost:8080/prometheus"
```

`docker/prometheus/prometheus.yaml`
```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'contact-us'
    scrape_interval: 5s
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080'] # Points to the laptop
```

`docker-compose.yml`
```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: ${DATABASE_NAME}-prometheus
    ports:
      - 9090:9090
    env_file:
      - .env
    command:
      - --config.file=/etc/prometheus/prometheus.yaml
    volumes:
      - ./docker/prometheus/prometheus.yaml:/etc/prometheus/prometheus.yaml:ro
```


http://localhost:9090/targets

![Prometheus Targets]({{ '/assets/images/Prometheus-Targets.png' | absolute_url }})

HTTP Requests

![Prometheus Graph]({{ '/assets/images/Prometheus-Graph.png' | absolute_url }})
