# Builds a base docker image for the Confluent stream platform. It doesn't
# start up any particular service, just installs the platform. Other images
# inherit from this image to start up a particular service.

FROM java:8-jre

ENV CDC_USER cdc
ENV CDC_GROUP cdc

RUN ["groupadd", "-r", "cdc"]
RUN ["useradd", "-r", "-g", "cdc", "cdc"]



