# Strimzi Kafka

> The namespace used in the kafka config files is __anair-kafka__. Do a replace all for the namespace on all files.

## Update only kafka and zookeeper images
- Update the image in __oc apply -f __install/cluster-operator/050-Deployment-strimzi-cluster-operator.yaml__
- Apply change that will restart zookeeper and kafka pods automatically `oc apply -f install/cluster-operator/050-Deployment-strimzi-cluster-operator.yaml`

## Install
- Deploy [0.11.4](0.11.4/README.md)

## Verify
1. Verify that all pods are running fine and are healthy
2. Verify following deployments are created:
   - strimzi-cluster-operator
   - anair-kafka-cluster-entity-operator
3. Verify following statefulsets are created:
   - anair-kafka-cluster-kafka  with 3 pods
   - anair-kafka-cluster-zookeeper with 3 pods
4. Verify routes are created
   - Get bootstrap route: `oc get routes [NAMESPACE]-cluster-kafka-bootstrap -o=jsonpath='{.status.ingress[0].host}{"n"}'`

## Security configuration
> Do this only for a new install. Skip for upgrade

1. `cd kafka/tls`
1. Extract the certificate for connecting to the kafka broker with TLS
   `oc extract secret/[NAMESPACE]-cluster-cluster-ca-cert --keys=ca.crt --to=- > ca.crt`
1. Create Truststore file from cert: `keytool -keystore client.truststore.jks -alias CARoot -import -file ca.crt`. Remember password
1. Create Keystore file from cert: `keytool -keystore client.keystore.jks -alias CARoot -import -file ca.crt`. Remember password
1. Update _tls/client-ssl.properties_ with keystore and truststore password
2. Use client-ssl.properties, jks files and cert to login to a remote secured strimzi kafka cluster

## Test
- Copy client-ssl.properties, client.keystore.jks, client.truststore.jks, ca.crt files to another folder like __kafka-ssl__.
- Install Kafka locally
- Copy bootstrap hostname from the external route __anair-kafka-cluster-kafka-bootstrap__ and use that as the bootstrap server

### Producer
`kafka-console-producer --broker-list [BOOTSTRAP_HOSTNAME]:443 --topic anair --producer.config client-ssl.properties`

### Consumer
`kafka-console-consumer --bootstrap-server [BOOTSTRAP_HOSTNAME]:443 --topic anair --consumer.config client-ssl.properties --from-beginning`

## Kafka,Zookeeper cluster metrics
1. Create route from kafka service exposing port 9404
2. Create route from zookeeper service exposing port 9404
3. Enable Edge TLS termination
4. Click on the new routes to see raw metics data
5. Ensure that these urls are configured in prometheus.yaml to capture and visualze metrics

## Zookeeper entrance
In order to have monitoring tools like Kafka Manager access access Strimzi kafka brokers and topics, it needs to access Strimzi kafka zookeeper. This version of zookeeper is secured and cannot be accessed by Kafka Manager. To overcome this, Jakob Schulz (main committer of Strimzi) has created a backdoor unsecured access to the same zookeeper cluster. Run below steps in every strimzi kafka cluster

1. Open __zookeeper-entrance/k8s/zookeeperentrance-dc.yaml__ and replace _{{STRIMZI_CLUSTER_NAME}}_ with the kafka cluster name like "anair-kafka-cluster"
2. Run __zookeeper-entrance/k8s/zookeeperentrance-dc.yaml__
3. Verify the pod is running fine


## Maintenance
### Rolling restart
Restart will be initiated by the cluster operator in 2 minutes
- [Zookeeper](https://strimzi.io/docs/0.11.4/#proc-manual-rolling-update-zookeeper-deployment-configuration-kafka): `oc annotate statefulset [NAMESPACE]-cluster-zookeeper operator.strimzi.io/manual-rolling-update=true`
- [Kafka](https://strimzi.io/docs/0.11.4/#proc-manual-rolling-update-kafka-deployment-configuration-kafka): `oc annotate statefulset [NAMESPACE]-cluster-kafka operator.strimzi.io/manual-rolling-update=true`

### Scaling brokers
https://strimzi.io/docs/0.11.4/#scaling-clusters-deployment-configuration-kafka
