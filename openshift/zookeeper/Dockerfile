FROM k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10
MAINTAINER Anoop Nair

LABEL description="Zookeeper image with json formatted logs"

ENV ZK_HOME /opt/zookeeper

COPY extra_lib/log4j-json-layout-1.0.0.jar /usr/share/zookeeper/

COPY conf/log4j.properties ${ZK_HOME}/conf
COPY bin/start-zookeeper ${ZK_HOME}/bin

RUN chmod +x ${ZK_HOME}/bin/start-zookeeper