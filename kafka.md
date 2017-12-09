Single broker Kafka using https://hub.docker.com/r/wurstmeister/kafka/

> Be carefull with this issue: https://github.com/wurstmeister/kafka-docker/issues/211

docker-compose.yml

```
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    build: .
    ports:
      - "9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

```



Terminal 1

```
docker exec -it kafkadocker_kafka_1 sh
$KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitio
ns 1 --topic test
$KAFKA_HOME/bin/kafka-topics.sh --list --zookeeper zookeeper:2181
```



