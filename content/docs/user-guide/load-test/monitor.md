---
title: 性能监控
weight: 3
description: 使用 Prometheus 实现性能监控
---

## Report metrics to Prometheus Pushgateway

Besides printing load testing results in console, you can also push metrics to [Prometheus Pushgateway][pushgateway_github], and then you can configure pretty graphs on [Grafana][Grafana].

```
$ hrp boom examples/demo.json --spawn-count 10 --spawn-rate 1 --prometheus-gateway http://127.0.0.1:9091
```

You can deploy the Pushgateway using the [prom/pushgateway][pushgateway_docker] Docker image at ease.

```
$ docker pull prom/pushgateway
$ docker run -d -p 9091:9091 prom/pushgateway
```

[pushgateway_github]: https://github.com/prometheus/pushgateway
[pushgateway_docker]: https://hub.docker.com/r/prom/pushgateway
[Grafana]: https://grafana.com/
