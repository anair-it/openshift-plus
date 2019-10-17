# Apache Nifi
Deploy custom Kubernetes compliant [Apache Nifi](https://nifi.apache.org/) 3 node cluster with added features:
- Logstash/Fluentd json logging support
- [Prometheus reporting task](https://github.com/mkjoerg/nifi-prometheus-reporter/). Publish Nifi JVM and flow metrics to Prometheus Push gateway 

> Cluster coordination is through an external [Zookeeper](../zookeeper/README.md) deployment in the same namespace as Nifi. Make sure Zookeeper is already running before proceeding to next step

## Deploy to Openshift
1. Build __anair/nifi-base:1.9.2-1__ base image locally
1. Decide namespace to deploy
1. Login to  Openshift docker registry
1. Tag image with openshift namespace
1. Push image to openshift project
1. Create configMap _nifi-conf_ with logback.xml entry. Copy the contents of _conf?logback.xml_
1. Open a 3 node [statefulset](k8s/nifi-statefulset.yaml)
  a. Update image name
  b. Update namespace
  c. Review mounts and PVCs
  d. Review and update zookeeper cluster nodes
  e. Update Cluster name
  f. Review anti-affinity rule to make sure the Nifi nodes are running in different K8s worker nodes
2. Deploy statefulset
3. Verify logs and pod health

## Custom logging
Logs will be streamed to STDOUT in json format that is easily consumable by logstash\fluentd. Log will have a new element called "logtype" which can be filtered in Kibana.
- Nifi bootstrap activity will set logtype as "bootstrap". This will happen on startup
- User activity like accessing flows, stop/start flows will set logtype as "user"
- Regular flow activity will set logtype as "app". This will be most logged type based on business activity

## Configure reporting task
1. Click on "Controller Settings"
1. Click on "Reporting Tasks" tab
1. Add new reporting task
1. Type "prom" in the UI filter
1. Select "PrometheusReportingTask" and click Add button
1. Edit "PrometheusReportingTask" properties
  a. Add Prometheus PushGateway url
  b. Update "The job name" is the job name that will be displayed in Prometheus

## Healthcheck
- Monitor CPU, memory and filesystem usage through an external monitroing system like Nagios, Zabbix
- Cluster health: `curl -ks https://NIFI_HOST/nifi-api/flow/cluster/summary | grep -c "{\"clusterSummary\":{\"connectedNodes\":\"3 \/ 3\",\"connectedNodeCount\":3,\"totalNodeCount\":3,\"connectedToCluster\":true,\"clustered\":true}}"`. If response = 0, cluster is unhealthy.
- Cluster JVM diagnostics: `curl -ks "http://NIFI_HOST/nifi-api/system-diagnostics"`
