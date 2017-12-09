# Kafka docker

```
git clone https://github.com/wurstmeister/kafka-docker.git
```

Build the base Kafka image

```
docker build -t my-kafka .
```

edit `docker-compose.yml`

```
version: '2'
services:
  zookeeper:
    container_name: jv_zookeeper
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    container_name: jv_kafka
    build: .
    ports:
      - "9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Start the containers

```
docker-compose -p jv_ up
```

> By default docker-compose prefixes the container name with folder name. I am using the prefix 'jv\_'  to define the desired prefix

Terminal 1 - Create topic and send messages

```
docker exec -it jv_kafka_1 sh
$KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic test
$KAFKA_HOME/bin/kafka-topics.sh --list --zookeeper zookeeper:2181
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic test
hello 1
hello 2
```

Terminal 2 - Consume messages from topic

```
docker exec -it jv_kafka_1 sh
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test --from-beginning
```

All going well you should see the following in terminal 2

```
hello 1
hello 2
```



