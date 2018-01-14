# Kafka

## Setup Kafka

Docker to the rescue again, using Confluent Docker images

create a new compose file now with both mongo and kafka `docker-compose-mongo-kafka.yml`

```
---
version: '2'
services:
    mongodb:
    image: mongo:3.0.4
    ports:
      - "27017:27017"
    command: mongod --smallfiles

    zookeeper:
    image: "confluentinc/cp-zookeeper:4.0.0"
    hostname: zookeeper
    ports:
      - '32181:32181'
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    extra_hosts:
      - "moby:127.0.0.1"

  kafka:
    image: "confluentinc/cp-kafka:4.0.0"
    hostname: kafka
    ports:
      - '9092:9092'
      - '29092:29092'
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    extra_hosts:
      - "moby:127.0.0.1"
```

Install kafkacat to test kafka is working

```
apt-get install kafkacat
```

start kafka

```
docker-compose up -f docker-compose-mongo-kafka.yml
```

on another terminal try publishing a message to a `greeting` topic using `kafkacat`

```
kafkacat
```

## Carry on coding



