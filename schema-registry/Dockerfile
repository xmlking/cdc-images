# Builds a docker image for the Confluent schema registry.
# Expects links to "kafka" and "zookeeper" containers.
#
# Usage:
#   docker build -t cdc/schema-registry schema-registry
#   docker run -d --name schema-registry --link zookeeper:zookeeper --link kafka:kafka \
#       --env SCHEMA_REGISTRY_AVRO_COMPATIBILITY_LEVEL=none cdc/schema-registry

FROM cdc/platform

MAINTAINER xmlking@gmail.com

COPY schema-registry-docker.sh /usr/local/bin/

#TODO Schema Registry needs a log directory.
RUN ["chown", "-R", "cdc:cdc", "/etc/schema-registry/schema-registry.properties"]
RUN ["chmod", "+x", "/usr/local/bin/schema-registry-docker.sh"]

EXPOSE 8081

USER cdc
ENTRYPOINT ["/usr/local/bin/schema-registry-docker.sh"]
