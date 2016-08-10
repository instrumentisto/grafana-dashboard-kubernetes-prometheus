Kubernetes cluster monitoring Grafana dashboard (via Prometheus)
================================================================

[![GitHub release](https://img.shields.io/github/release/instrumentisto/grafana-dashboard-kubernetes-prometheus.svg)](https://github.com/instrumentisto/grafana-dashboard-kubernetes-prometheus)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/blob/master/LICENSE.md)
[![license](https://img.shields.io/badge/grafana.net-link-blue.svg)](https://grafana.net/dashboards/303)

Initial idea was taken from [this dashboard](https://grafana.net/dashboards/162)
and improved to exclude `node-exporter` dependency and to give more information
about cluster state.



## Requirements

You only need to have running [Kubernetes](http://kubernetes.io) cluster with 
deployed [Prometheus](https://prometheus.io).
Prometheus will use metrics provided by [cAdvisor](https://github.com/google/cadvisor) 
via [kubelet](http://kubernetes.io/docs/admin/kubelet) service (runs on
each node of Kubernetes cluster by default) and via 
[kube-apiserver](http://kubernetes.io/docs/admin/kube-apiserver) service only.

Your Prometheus configuration has to contain following
[`scrape_configs`](https://prometheus.io/docs/operating/configuration/#scrape_config): 
```yaml
scrape_configs:
  - job_name: kubernetes-nodes-cadvisor
    scheme: https  # remove if you want to scape metrics on insecure port
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    kubernetes_sd_configs:
      - api_servers:
          - https://kubernetes.default.svc
        in_cluster: true
        role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - source_labels: [__meta_kubernetes_role]
        action: replace
        target_label: kubernetes_role
    metric_relabel_configs:
      - source_labels: [id]
        action: replace
        regex: '^/machine\.slice/machine-rkt\\x2d([^\\]+)\\.+/([^/]+)\.service$'
        target_label: rkt_container_name
        replacement: '${2}-${1}'
      - source_labels: [id]
        action: replace
        regex: '^/system\.slice/(.+)\.service$'
        target_label: systemd_service_name
        replacement: '${1}'

  - job_name: kubernetes-apiserver-cadvisor
    scheme: https  # remove if you want to scape metrics on insecure port
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    kubernetes_sd_configs:
      - api_servers:
          - https://kubernetes.default.svc
        in_cluster: true
        role: apiserver
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - source_labels: [__meta_kubernetes_role]
        action: replace
        target_label: kubernetes_role
```



## Features

- Total and used cluster resources: CPU, memory, filesystem.  
  And total cluster network I/O pressure.  
  ![Total and used cluster resources](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/master/screens/total.png)
- [Kubernetes pods](http://kubernetes.io/docs/user-guide/pods) usage:
  CPU, memory, network I/O.  
  ![Pods usage](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/master/screens/pods.png)
- Containers usage: CPU, memory, network I/O.  
  [Docker](https://www.docker.com) and [rkt](https://coreos.com/rkt) containers
  which runs on cluster nodes but outside Kubernetes are also monitored.  
  ![Containers usage](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/master/screens/containers.png)
- [systemd](https://freedesktop.org/wiki/Software/systemd) system services
  usage: CPU, memory.  
  ![systemd usage](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/master/screens/systemd.png)
- Showing all above metrics both for all cluster and each node separately.  
  ![Filtering metrics by nodes](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/master/screens/by_nodes.png)



## Troubleshooting

If filesystem usage panels display `N/A`, you should correct
`device="/dev/vda9"` filter parameter in metrics query with devices your system
actually has.
