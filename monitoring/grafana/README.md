# Grafana
Deploy custom Openshift compliant Grafana image and dashboard and not using the stock version (which is old) from Openshift

## Configuration
- Open _conf/defaults.ini_ and replace COMPANY and NAMESPACE as required
- Review and add OAUTH or Basic auth, SMTP etc.

## Deployment
1. Create Config map __grafana-ini__. Add _conf/defaults.ini_ here
1. Decide on a namespace to deploy Grafana
1. Deploy _k8s/grafana-dc.yaml_. This will pull the latest Grafana image from Docker hub and use the configuration in defaults.ini to start up.

## Dashboard
- Upload _dashboard/openshift-cluster.json_ to Grafana to visualize Openshift metrics like CPU, Memory, file system, network etc.
  - Pre-requisite is to have Node exporter running in the cluster and Prometheus scraping from Node exporter

## Reference
- [Getting started](https://grafana.com/docs/guides/getting_started/)
- [OAuth authentication](https://grafana.com/docs/auth/generic-oauth/)
- [Alerting](https://grafana.com/docs/alerting/rules/)
