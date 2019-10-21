# Monitoring

## Prometheus
1. Create namespace cluster-monitoring
1. Create configmap with key as prometheus.yaml with the contents from _prometheus/conf/prometheus.yaml_
1. Deploy prometheus components
   - _prometheus/k8s/service.yaml_
   - _prometheus/k8s/sts.yaml_
1. Add application specific scrape jobs in prometheus.yaml

## Node exporter
Deploy node exporter daemonset to fetch Kubernetes info from all nodes        
1. Deploy _node-exporter/k8s/service.yaml_
2. Deploy _node-exporter/k8s/daemonset.yaml_

## Prometheus push gateway
Deploy push gateway to collect stats from ephemeral and remote processes like Nifi           
1. Deploy _push-gateway/k8s/deploymentconfig.yaml_

## Grafana
[README](grafana/README.md)

