# Kafka Manager
Monitor Kafka brokers and topics on a Strimzi kafka cluster. Only 1 instance of Kafka Manager is required per Openshift cluster and can connect to multiple kafka clusters.

> Prerequisite: Zookeeper entrance is deployed in the Strimzi kafka namespace

## Deploy
1. Decide on a namespace
2. Create a config map called "kafka.manager.application.conf" and paste the contents of _conf/application.conf_
3. Create a generic secret called "kafka.manager.creds" with "username" and "password" keys. These will be used as basic auth credentials to log into kafka manager web console
4. Open __k8s/kafkamanager-dc.yaml__
   1. Replace _{{ZOOKEEPER_ENTRANCE_NAMESPACE}}_ with _zookeeper-entrance_ namespace. Make sure zookeeper-entrance pod is running in the strimzi kafka namespace
1. Deploy __k8s/kafkamanager-dc.yaml__
5. Verify kafka-manager logs and make sure there are no errors connecting to zookeeper. If there errors, service connection to zookeeper-entrance has to be fixed.
6. Create a route for kafka manager service
7. Open kafka manager web console and add the kafka cluster specific zookeeper-entrance host and port and give a cluster name
   1. Add as many kafka clusters and make sure you use the zookeeper-entrance service host for the corresponding cluster
8.  Enable "JMX polling" and "Poll consumer information"
9.  Change value of "brokerViewThreadPoolSize", "offsetCacheThreadPoolSize", "kafkaAdminClientThreadPoolSize" to 2
10. Click save and you can view brokers, partitions, topics, consumer info 

## Read-only mode
By default Kafka manager allows users to modify topic and partition info and even delete a topic. If read-only mode is desired:
1. Create another namespace
1. Update _conf/application.conf_ and keep only "KMClusterManagerFeature in "application.features"
1. Deploy new instance of read-only version of Kafka manager that can be used by non-admins
1. Follow above post-deployments steps 