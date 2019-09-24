# Fluentd - Log aggregator
- Here lives the fluentd log collection configuration for Openshift. These set of configuration files will ship app and ops logs to an external Elasticsearch cluster.
- Fluentd in openshift uses the [Elasticsearch plugin](https://docs.fluentd.org/output/elasticsearch) to ship logs to ES
- ES index is created based on the openshift convention: "project."

## Configuration files
1. _fluent.conf_: Main configuration file that includes fluentd source, filter and output configuration files. When fluentd service starts, this is the init file.
2. _output-application.conf_: This file is included in _fluent.conf_. Responsible for sending logs to an external source.
3. _output-es-config.conf_: This file is included in _output-application.conf_. Responsible for shipping logs to an external ES cluster. Information about the ES cluster like host, port, username, certificates are configured here.
4. _output-es-retry.conf_: This file is included in _output-application.conf_. Responsible for retrying failed log shipping dues to buffering or other issues. The configuration lets the failed logs to be shipped to ES cluster again.

## K8s configuration
- Create a Config Map __logging-fluentd__
  - Add all the conf files as separate entries in the Config Map
  - Configured to mount the fluentd conf files at __/etc/fluent/configs.d/user__
- Get root-ca.pem file from the Elasticsearch cluster
- Create a k8s generic/opaque secret with the following params to store the ES certificate:
  - Secret name: opendistro-es-cert
  - Secret key: root-ca.pem
  - Secret value: root-ca.pem file content
- Create a k8s generic/opaque user credentials secret with the following params:
  - Secret name: opendistro-es-cred
    - Secret key: user
    - Secret value: {ES USER ID}
    - Secret key: password
    - Secret value: {ES PASSWORD}
- Review and update the daemonset
- Update the environment variables through a CD process and deploy this daemonset


## Deploy fluentd configuration
- Remove label to stop the daemon set in all nodes      
  `oc label node -l logging-infra-fluentd=true --overwrite logging-infra-fluentd=false`
- Verify fluentd pods are terminated. The resultset should be empty.      
  `oc get po -l component=fluentd`
- Add label to start the daemon set in all nodes       
  `oc label node -l logging-infra-fluentd=false --overwrite logging-infra-fluentd=true`
- Wait for few minutes for the daemon set to start up.
- Verify fluentd pods are started      
  `oc get po -l component=fluentd` or `oc get ds -l component=fluentd`
- Get a pod name and verify logs and look for errors:        
  `oc logs -f {POD NAME}`. 


## Reference
[Aggregating container logs in Openshift](https://docs.openshift.com/container-platform/3.10/install_config/aggregate_logging.html)