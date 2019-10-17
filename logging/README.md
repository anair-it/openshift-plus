# Distributed Logging using external EFK stack
Openshift comes with its Opensource EFK stack that is versions behind the new EFK stack and has less features. There is no security and alerting features. 
The Opensource version of Elastic does not have security and alerting built into it for free, but the OpenDistro version from Amazon has all of that. 
Follow the below steps to provision an external OpenDistro Elasticsearch and Kibana cluster. 

## Prerequisite - Environment configuration
### Docker log settings
https://docs.docker.com/config/containers/logging/configure/
- Set this in __/etc/docker/daemon.json__ in all Openshift nodes to restrict log file count and size
        
        {
            "log-driver": "json-file",
            "log-opts": {
                "max-size": "10m",
                "max-file": "1"
            }
        }
- Restart docker service
- Repeat the above steps on all compute nodes
  - On master and infra, work on 1 node at a time.

### Fluentd log settings
This is optional. The default is 10 fluentd log files each 1MB.
Set the number of files to 5 each with max size 1MB.      
`oc set env ds/logging-fluentd LOGGING_FILE_AGE=5 LOGGING_FILE_SIZE=1024000"`


## Provision
- Got through these readme's to provision, configure and run Elasticsearch and Kibana
  - [Elasticsearch](elasticsearch/README.md)
  - [Kibana](kibana/README.md)
- After Elasticsearch is up, configure Fluentd in Openshift
  - [Fluentd](fluentd/README.md)

## Data retention
Indices have to be deleted periodically based on the business retention policy and also to maintain optimum storage capacity needs and improve search speed. 
[Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html) is the Elasticsearch maintenance tool. It supports deleting indices out of many other actions.
- [Curator configuration](curator/README.md)


# OpenDistro Elasticsearch and Kibana Reference
- [Reference](https://opendistro.github.io/for-elasticsearch/)
- [Configuration](https://opendistro.github.io/for-elasticsearch-docs/)
- [Change admin password](https://aws.amazon.com/blogs/opensource/change-passwords-open-distro-for-elasticsearch/)
- [SSL configuration](https://aws.amazon.com/blogs/opensource/add-ssl-certificates-open-distro-for-elasticsearch/?nc1=b_rp)
- [Discussion forum](https://discuss.opendistrocommunity.dev/)

# Readme's
- [Elasticsearch](elasticsearch/README.md)
- [Fluentd](fluentd/README.md)
- [Kibana](kibana/README.md)
- [Curator](curator/README.md)
