Memcached Pods monitoring Grafana Dashboard (via Prometheus)
============================================================

[![release](https://img.shields.io/github/release/instrumentisto/grafana-dashboard-memcached-kubernetes-prometheus.svg)](https://grafana.com/dashboards/3063/revisions)  [![license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/instrumentisto/grafana-dashboard-memcached-kubernetes-prometheus/blob/master/LICENSE.md)    
[![prometheus](https://img.shields.io/badge/prometheus-%5E1.3.0-green.svg)](https://github.com/prometheus/prometheus) [![grafana](https://img.shields.io/badge/grafana-%5E4.4.3-green.svg)][5] [![memcached_exporter](https://img.shields.io/badge/memcached_exporter-%5E0.3.0-green.svg)][4]

![screenshot](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-memcached-kubernetes-prometheus/master/screenshot.png)




## Metrics

- Memory usage
- Hit & Miss ratio
- Evicts & Reclaims
- Items in cache
- Network stats
- Commands usage




## Requirements

1. [Kubernetes][1] cluster with deployed [Prometheus][3] and [Grafana][5].
2. [Memcached Exporter][4] deployed alongside with [Memcached][2] [Pod][6].

Your Prometheus configuration should contain the following [`scrape_config`][10]:
```yaml
scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
```

Your Memcached Pod should contain [`prometheus.io/*` annotations][11]:
```yaml
kind: StatefulSet
apiVersion: apps/v1beta1
metadata:
  name: my-memcached
spec:
  serviceName: my-memcached
  replicas: 1
  template:
    metadata:
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '9150'
    spec:
      containers:
        - name: memcached
          image: memcached:alpine
          ports:
            - name: memcache
              containerPort: 11211
        - name: metrics
          image: quay.io/prometheus/memcached-exporter:v0.3.0
          ports:
            - name: metrics
              containerPort: 9150
```




## Issues

If you have any problems with or questions about this dashboard, please contact us through a [GitHub issue][20].





[1]: https://kubernetes.io
[2]: https://github.com/memcached/memcached
[3]: https://prometheus.io
[4]: https://github.com/prometheus/memcached_exporter
[5]: https://github.com/grafana/grafana
[6]: https://kubernetes.io/docs/concepts/workloads/pods/pod
[10]: https://prometheus.io/docs/operating/configuration/#scrape_config
[11]: https://github.com/prometheus/prometheus/blob/v1.3.0/documentation/examples/prometheus-kubernetes.yml#L153-L155
[20]: https://github.com/grafana-dashboard-memcached-kubernetes-prometheus/issues
