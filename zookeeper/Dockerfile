# Builds a docker image running Apache Zookeeper. Intended for use with Kafka.
#
# Usage:
#   docker build -t cdc/zookeeper zookeeper
#   docker run -d --name zookeeper cdc/zookeeper

FROM cdc/platform

MAINTAINER xmlking@gmail.com

ENV LOG_DIR "/var/log/zookeeper"
ENV ZK_DATA_DIR "/var/log/zookeeper"
ENV KAFKA_LOG4J_OPTS -Dlog4j.configuration=file:/etc/kafka/log4j.properties

ADD zk-docker.sh /usr/local/bin/
ADD log4j.properties /etc/kafka/log4j.properties

RUN ["mkdir", "-p", "/var/log/zookeeper", "/var/lib/zookeeper"]
RUN ["chown", "-R", "cdc:cdc", "/var/log/zookeeper", "/var/lib/zookeeper", "/etc/kafka/zookeeper.properties"]
RUN ["chmod", "+x", "/usr/local/bin/zk-docker.sh"]

VOLUME ["/var/log/zookeeper", "/var/lib/zookeeper"]

#TODO Expose JMX
# Expose client port (2188/tcp), peer connection port (2888/tcp), leader election port (3888/tcp)
EXPOSE 2181 2888 3888

USER cdc
ENTRYPOINT ["/usr/local/bin/zk-docker.sh"]