# Provision Strimzi Kafka on Openshift
Install [Strimzi Kafka](https://strimzi.io/) cluster in Openshift. Below steps are catered to work in Minishift too.

## Versions
- Strimzi Kafka: 0.11.4
- Kafka: 2.1.0
- JDK: 1.8

> The namespace used in the kafka config files is __anair-kafka__. Do a replace all for the namespace on all files.

## Create custom kafka and zookeeper images
This is to add a json log parser component that publishes logs in json format to ES      

1. Update kafka image with json log parser
   - Build image: `docker build -f Dockerfile-kafka -t anair/strimzi-kafka:0.11.4-2.1.0 .`
   - Login to the remote docker registry:
     - Minishift: `docker login -u $(oc whoami) -p $(oc whoami -t) $(minishift openshift registry)`
     - Openshift: Get from url from openshift registry page
   - Tag image with minishift project name: 
     - Minishift: `docker tag anair/strimzi-kafka:0.11.4-2.1.0 172.30.1.1:5000/anair-kafka/strimzi-kafka:0.11.4-2.1.0`
     - Openshift: Replace 172.30.1.1:5000 with registry hostname
   - Push image to registry: 
     - Minishift: `docker push 172.30.1.1:5000/anair-kafka/strimzi-kafka:0.11.4-2.1.0`
     - Openshift: Replace 172.30.1.1:5000 with registry hostname
   - Verify image stream: `oc get istag`
   
2. Update zookeeper image with json log parser
   - Build image: `docker build -f Dockerfile-zookeeper -t anair/strimzi-zookeeper:0.11.4-2.0.0 .`
   - For Minishift:
     - Docker login: `docker login -u $(oc whoami) -p $(oc whoami -t) $(minishift openshift registry)`
     - Tag image with minishift project name: `docker tag anair/strimzi-zookeeper:0.11.4-2.0.0 172.30.1.1:5000/anair-kafka/strimzi-zookeeper:0.11.4-2.0.0`
     - Push image to registry: 
       - Minishift: `docker push 172.30.1.1:5000/anair-kafka/strimzi-zookeeper:0.11.4-2.0.0`
       - Openshift: Replace 172.30.1.1:5000 with registry hostname
     - Verify image stream: `oc get istag`

## Update only kafka and zookeeper images
- Update the image in __oc apply -f __install/cluster-operator/050-Deployment-strimzi-cluster-operator.yaml__
- Apply change that will restart zookeeper and kafka pods automatically `oc apply -f install/cluster-operator/050-Deployment-strimzi-cluster-operator.yaml`
   
## New Install
1. Update __install/cluster-operator/050-Deployment-strimzi-cluster-operator.yaml__ with the custom kafka and zookeeper image
2. Create Cluster operator rbac's, service accounts, api extensions. The Cluster Operator is in charge of deploying a Kafka cluster alongside a Zookeeper ensemble. `oc apply -f install/cluster-operator`
3. Update permission for "default" service account:
   - Go to Resources > Membership > Service Accounts.  Clck on Edit Membership
   - Check "Show hidden roles"
   - For service account anair-kafka/builder, add hidden role "strimzi-cluster-operator-global"
   
   - Verify the pod is started
4. Install Cluster operator templates: `oc apply -f templates/cluster-operator`
5. Install Kafka-cluster. Strimzi operator will first deploy a zookeeper cluster and then the kafka cluster
   - Openshift: `oc apply -f kafka-persistent.yaml`
   - Minishift: `oc apply -f kafka-ephemeral.yaml`
6. Update config maps: `oc apply -f common`
7. Move onto next steps in [README.md](../README.md)

## Reference
[Strimzi Kafka 0.11.4](https://strimzi.io/docs/0.11.4/)