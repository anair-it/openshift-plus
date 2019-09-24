# Zookeeper
Deploy custom Kubernetes compliant [Zookeeper](https://zookeeper.apache.org/) image (extended from Kubernetes Zookeepper) with added features:
-   Publish json formatted logs for easy consumption by EFK stack
  
> This image can be run in any container platform

## Customization
- Refer the custom jar in _extra\_lib_. This will format logs into Json format. 
- Modified Log4j properties are in _conf_
- _bin\start-zookeeper_ is modified to not use the existing log4j properties file

## Test locally
- Build image: `docker build -t anair/zookeeper:3.4.10-1 .`
- Create new project in minishift
- Deploy _k8s/zookeeper-sts.yaml_ stateful set to create a 3 pod zookeeper cluster
- Verify logs are in json format

## Deploy to Openshift
1. Build image: `docker build -t anair/zookeeper:3.4.10-1 .`
1. Decide on the namespace where zookeeper should be deployed. The project uses __anair-zk__ as the namespace.
1. Tag image Openshift registry naming convention
1. Push image to Openshift registry. The image will appear in ImageStreams of the namespace
1. Open [ZK statefulset](k8s/zookeeper-sts.yaml)
  a. Review and change parameters as required
  b. Update namespace
  c.Update the image name with Openshift registry image name
1. Deploy the statefuleset
1. Verify json formatted logs