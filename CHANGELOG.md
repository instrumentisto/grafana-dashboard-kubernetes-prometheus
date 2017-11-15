Change Log
==========

All notable changes to this project will be documented in this file.



## v3 [2017-11-15]

- Matches all possibles disks drives. ([#6](https://github.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/pull/6), [@Lord-Y](https://github.com/Lord-Y))



## v2 [2016-11-16]

- Makes CPU and memory usage charts staircased.
- Removes `io_kubernetes_` labels prefix usage for newer Prometheus versions.
- More clever file system device detection (for [CoreOS
  mainly](https://coreos.com/os/docs/latest/sdk-disk-partitions.html)).



## v1 [2016-08-10]

- Total and used cluster resources: CPU, memory, filesystem.  
  And total cluster network I/O pressure.
- [Kubernetes pods](http://kubernetes.io/docs/user-guide/pods) usage:
  CPU, memory, network I/O.
- Containers usage: CPU, memory, network I/O.  
  [Docker](https://www.docker.com) and [rkt](https://coreos.com/rkt) containers
  which runs on cluster nodes but outside Kubernetes are also monitored.
- [systemd](https://freedesktop.org/wiki/Software/systemd) system services
  usage: CPU, memory.
- Showing all above metrics both for all cluster and each node separately.
    
