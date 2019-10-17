# Apache Nifi
Deploy custom [Apache Nifi](https://nifi.apache.org/) 3 node cluster on Kubernetes platform with added features:
- Logstash/Fluentd json logging support
- [Prometheus reporting task](https://github.com/mkjoerg/nifi-prometheus-reporter/). Publish Nifi JVM and flow metrics to Prometheus Push gateway 

> Cluster coordination is through an external [Zookeeper](../zookeeper/README.md) deployment in the same namespace as Nifi. Make sure Zookeeper is already running before proceeding to next step

## Deploy to Openshift
1. Build __anair/nifi-base:1.9.2-1__ base image locally
2. Decide namespace to deploy
3. Login to  Openshift docker registry
4. Tag image with openshift namespace
5. Push image to openshift project
6. Create configMap _nifi-conf_ with logback.xml entry. Copy the contents of the custom _conf/logback.xml_ into the configmap with key as "logback.xml"
7. Open and review the 3 node [statefulset](k8s/nifi-statefulset.yaml)     
  a. Update image name        
  b. Update namespace     
  c. Review mounts and PVCs     
  d. Review and update zookeeper cluster nodes       
  e. Update Cluster name     
  f. Review anti-affinity rule to make sure the Nifi nodes are running   in different K8s worker nodes     
2. Deploy statefulset
3. Verify logs and pod health

## Custom logging
Logs will be streamed to STDOUT in json format that is easily consumable by logstash\fluentd. Every log will have a custom attribute "logtype" which can be used as a filter and alert criteria in Kibana.
- Nifi bootstrap activity will set logtype as "bootstrap". This will happen on startup
- User activity like accessing flows, stop/start flows will set logtype as "user"
- Regular flow activity will set logtype as "app". This will be most logged type based on business activity
- Import custom Nifi Kibana dashboards at _logging/kibana/visualization/nifi-error-by-pod.json_ to Kibana. Monitor errors by pod.

## Configure reporting task
1. Click on "Controller Settings"
2. Click on "Reporting Tasks" tab
3. Add new reporting task
4. Type "prom" in the UI filter
5. Select "PrometheusReportingTask" and click Add button
6. Edit "PrometheusReportingTask" properties
  a. Add Prometheus PushGateway url
  b. Update "The job name" is the job name that will be displayed in Prometheus

## Healthcheck
- Monitor CPU, memory and filesystem usage through an external monitroing system like Nagios, Zabbix
- Cluster health: `curl -ks https://NIFI_HOST/nifi-api/flow/cluster/summary | grep -c "{\"clusterSummary\":{\"connectedNodes\":\"3 \/ 3\",\"connectedNodeCount\":3,\"totalNodeCount\":3,\"connectedToCluster\":true,\"clustered\":true}}"`. If response = 0, cluster is unhealthy.
- Cluster JVM diagnostics: `curl -ks "http://NIFI_HOST/nifi-api/system-diagnostics"`
- With Prometheus Reporting Task, Nifi JVM diagnostics are pushed to Prometheus and can be visualized in Grafana. Refer the attached Nifi Grafana dashboard in _monitoring/grafana/dashboard/nifi.json_

## Security
- [Encryption at motion](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#encryption)
- [Encryption at rest on Provenance](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#encrypted-write-ahead-provenance-repository-properties)
- [User Authentication](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#user_authentication)
- [User Authorization](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#multi-tenant-authorization)

## Reference
- [Rest API](https://nifi.apache.org/docs/nifi-docs/rest-api/index.html)
- [How Nifi works](https://www.freecodecamp.org/news/nifi-surf-on-your-dataflow-4f3343c50aa2/)