FROM xemuliam/nifi:1.9.2
MAINTAINER Anoop Nair

LABEL description="Build nifi base image with custom libraries"

COPY extra_lib/logstash-logback-encoder* ${NIFI_HOME}/lib/bootstrap/
COPY extra_lib/logstash-logback-encoder* ${NIFI_HOME}/lib/

ADD https://github.com/mkjoerg/nifi-prometheus-reporter/releases/download/nifi-1.9.2/nifi-prometheus-nar-1.9.2.nar ${NIFI_HOME}/lib

RUN cp ${NIFI_HOME}/lib/bootstrap/jackson*.jar ${NIFI_HOME}/lib

WORKDIR ${NIFI_HOME}
RUN chmod +x ./start_nifi.sh
CMD ./start_nifi.sh