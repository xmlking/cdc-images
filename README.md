CDC on Docker
=============
Docker images to demo Maxwell CDC solution with Kafka, NiFi. 


### Prerequisite  
1. Server with Docker Engine and Docker CLI
2. PC with [Docker Beta](https://beta.docker.com/)

### Start 
```bash
# Start Zookeeper and expose port 2181 for use by the host machine
docker run -d --name zookeeper -p 2181:2181 cdc/zookeeper

# Start Kafka and expose port 9092 for use by the host machine
docker run -d --name kafka -p 9092:9092 --link zookeeper:zookeeper cdc/kafka

# Start Schema Registry and expose port 8081 for use by the host machine
docker run -d --name schema-registry -p 8081:8081 --link zookeeper:zookeeper \
    --link kafka:kafka cdc/schema-registry

# Start REST Proxy and expose port 8082 for use by the host machine
docker run -d --name rest-proxy -p 8082:8082 --link zookeeper:zookeeper \
    --link kafka:kafka --link schema-registry:schema-registry cdc/rest-proxy
```

If all goes well, `docker ps` should give you something that looks like this:

    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                        NAMES
    7933c0d27cad        cdc/rest-proxy        "/usr/local/bin/rest-"   31 minutes ago      Up 31 minutes       0.0.0.0:8082->8082/tcp                       rest-proxy
    f992db359c63        cdc/schema-registry   "/usr/local/bin/schem"   31 minutes ago      Up 31 minutes       0.0.0.0:8081->8081/tcp                       schema-registry
    1682ba009c78        cdc/kafka             "/usr/local/bin/kafka"   31 minutes ago      Up 31 minutes       0.0.0.0:9092->9092/tcp                       kafka
    621aa2cc7240        cdc/zookeeper         "/usr/local/bin/zk-do"   31 minutes ago      Up 31 minutes       2888/tcp, 0.0.0.0:2181->2181/tcp, 3888/tcp   zookeeper

### Access
```
# access log
docker logs --follow  $JOB
# SSH to container 
sudo docker exec -i -t zookeeper /bin/bash
```

#### Start with Docker Compose 
Start all containers for fullstack

```bash
cd examples/fullstack
# start all
docker-compose up
# check
docker-compose ps
# start all
docker-compose down
```

#### Running on Multiple Remote Hosts in Cluster

```bash
docker run --name zk-1 -e zk_id=1 -e zk_server.1=172.16.42.101:2888:3888 -e zk_server.2=172.16.42.102:2888:3888 -e zk_server.3=172.16.42.103:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 cdc/zookeeper
docker run --name zk-2 -e zk_id=2 -e zk_server.1=172.16.42.101:2888:3888 -e zk_server.2=172.16.42.102:2888:3888 -e zk_server.3=172.16.42.103:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 cdc/zookeeper
docker run --name zk-3 -e zk_id=3 -e zk_server.1=172.16.42.101:2888:3888 -e zk_server.2=172.16.42.102:2888:3888 -e zk_server.3=172.16.42.103:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 cdc/zookeeper
docker run --name kafka-1 -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=172.16.42.101:2181,172.16.42.102:2181,172.16.42.103:2181 -p 9092:9092 cdc/kafka
docker run --name kafka-2 -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=172.16.42.101:2181,172.16.42.102:2181,172.16.42.103:2181 -p 9092:9092 cdc/kafka
docker run --name kafka-3 -e KAFKA_BROKER_ID=3 -e KAFKA_ZOOKEEPER_CONNECT=172.16.42.101:2181,172.16.42.102:2181,172.16.42.103:2181 -p 9092:9092 cdc/kafka
```

## Changing settings
The images support using environment variables via the Docker `-e | --env` flags for setting various settings in the respective images. For example:
For the Zookeeper image use variables prefixed with `ZOOKEEPER_` with the variables expressed exactly as how they would appear in the `zookeeper.properties` file. As an example, to set `syncLimit` and `server.1` you'd run `docker run --name zk -e ZOOKEEPER_syncLimit=2 -e ZOOKEEPER__server.1=localhost:2888:3888 cdc/zookeeper`.

### Testing

```
# Get a list of topics
curl "http://localhost:8082/topics"
# Get info about one partition
curl "http://localhost:8082/topics/_schemas"
# Produce a message using binary embedded data with value "Kafka" to the topic test
curl -X POST -H "Content-Type: application/vnd.kafka.binary.v1+json" \
      --data '{"records":[{"value":"S2Fma2E="}]}' "http://localhost:8082/topics/test"
# Create a consumer for binary data, starting at the beginning of the topic's log. 
curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
      --data '{"format": "binary", "auto.offset.reset": "smallest"}' \
      http://localhost:8082/consumers/my_binary_consumer
# Then consume some data from a topic using the base URL in the first response.
curl -X GET -H "Accept: application/vnd.kafka.binary.v1+json" \
      http://localhost:8082/consumers/my_binary_consumer/instances/rest-consumer-1-230b71ef-a450-462e-93ee-a9eea451c316/topics/test
# Finally, close the consumer with a DELETE to make it leave the group and clean up its resources.
curl -X DELETE \
      http://localhost:8082/consumers/my_binary_consumer/instances/rest-consumer-1-230b71ef-a450-462e-93ee-a9eea451c316
```


### Build

```
./build.sh
```