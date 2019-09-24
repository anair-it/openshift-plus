# Provision Strimzi Kafka on Openshift
Deploy custom [Strimzi Kafka](https://strimzi.io/) cluster in Openshift that publishes logs in logstash/fluentd format. Below steps are catered to work in Minishift too.

## Versions
- Strimzi Kafka: 0.9.0
- Kafka: 2.0.1
- JDK: 1.8

## Create and setup project
1. Kafka home directory: `cd openshift/kafka/0.9.0`
2. Create new project and permissions         
        
        oc login -u admin
        oc new-project anair-kafka
        oc adm policy add-scc-to-user anyuid -z default

## Create custom kafka and zookeeper images
This is to add a json log parser component that publishes logs in json format to ES. Run these locally with Minishift running

1. Update kafka image with json log parser
   - Build image: `docker build -f Dockerfile-kafka -t anair/strimzi-kafka:0.9.0 .`
   - Login to the remote docker registry:
     - Minishift: `docker login -u $(oc whoami) -p $(oc whoami -t) $(minishift openshift registry)`
     - Openshift: Get from url from openshift registry page
   - Tag image with minishift project name: 
     - Minishift: `docker tag anair/strimzi-kafka:0.9.0 172.30.1.1:5000/anair-kafka/strimzi-kafka:0.9.0`
     - Openshift: Replace 172.30.1.1:5000 with registry hostname
   - Push image to registry: 
     - Minishift: `docker push 172.30.1.1:5000/anair-kafka/strimzi-kafka:0.9.0`
     - Openshift: Replace 172.30.1.1:5000 with registry hostname
   - Verify image stream: `oc get istag`
   
2. Update zookeeper image with json log parser
   - Build image: `docker build -f Dockerfile-zookeeper -t anair/strimzi-zookeeper:0.9.0 .`
   - For Minishift:
     - Docker login: `docker login -u $(oc whoami) -p $(oc whoami -t) $(minishift openshift registry)`
     - Tag image with minishift project name: `docker tag anair/strimzi-zookeeper:0.9.0 172.30.1.1:5000/anair-kafka/strimzi-zookeeper:0.9.0`
     - Push image to registry: 
       - Minishift: `docker push 172.30.1.1:5000/anair-kafka/strimzi-zookeeper:0.9.0`
       - Openshift: Replace 172.30.1.1:5000 with registry hostname
     - Verify image stream: `oc get istag`

## Install
> The namespace used in the kafka config files is __anair-kafka__. Do a replace all for the namespace on all files.

1. Update __install/cluster-operator/050-Deployment-strimzi-cluster-operator.yaml__ with the custom kafka and zookeeper image
2. Create Cluster operator rbac's, service accounts, api extensions. The Cluster Operator is in charge of deploying a Kafka cluster alongside a Zookeeper ensemble. `oc apply -f install/cluster-operator`
3. Update permission for "default" service account:
   - Go to Resources > Membership > Service Accounts.  Clck on Edit Membership
   - Check "Show hidden roles"
   - For service account anair-kafka/builder, add hidden role "strimzi-cluster-operator-global"
   
   - Verify the pod is started
4. Install Cluster operator templates: `oc apply -f templates/cluster-operator`
5. Install Kafka-cluster. Strimzi operator will first deploy a zookeeper cluster and then the kafka cluster after 2 minutes
   - Openshift: `oc apply -f kafka-persistent.yaml`
   - Minishift: `oc apply -f kafka-ephemeral.yaml`
6. Update config maps: `oc apply -f common`
7. Move onto next steps in [README.md](../README.md)

## Reference
[Strimzi Kafka 0.9.0](https://strimzi.io/docs/0.9.0/)