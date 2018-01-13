# Kafka docker

For Kafka we are going to use Confluent's docker images

edit `docker-compose.yml`

```
---
version: '2'
services:
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

Start the containers

```
docker-compose -p jv_ up
```

> By default docker-compose prefixes the container name with folder name. I am using the prefix 'jv\_'  to define the desired prefix

# Testing Kafka

[kafkacat](https://github.com/edenhill/kafkacat) is an easy to use command line utility which will allows us to write, read and inspect Kafka topics.

To install

```
apt-get install kafkacat
```

To write to a topic

```
echo 'some data' | kafkacat -b localhost -t some-topic
```

To read from a topic

```
kafkacat -b localhost -t some-topic

% Auto-selecting Consumer mode (use -P or -C to override)
some data
```

To inspect a topic

```
kafkacat -L -b localhost -t some-topic

Metadata for some-topic (from broker -1: localhost:9092/bootstrap):
 1 brokers:
  broker 1 at localhost:9092
 1 topics:
  topic "some-topic" with 1 partitions:
    partition 0, leader 1, replicas: 1, isrs: 1

```

# Spring Boot Kafka

Create a Spring Boot project with a Kafka dependency

```
spring init -d=kafka -artifactId=spring-kafka -name=sentenceStats spring-kafka
```

edit `./src/test/java/com/example/springkafka/SpringKafkaApplicationTests.java`

```
package com.example.springkafka;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

@SpringBootApplication
public class SpringKafkaApplication implements CommandLineRunner {

    public static Logger logger = LoggerFactory.getLogger(SpringKafkaApplication.class);

    public static void main(String[] args) {
        SpringApplication.run(SpringKafkaApplication.class, args);
    }

    @Autowired
    private KafkaTemplate<String, String> template;

    private final CountDownLatch latch = new CountDownLatch(3);
    private final String topicName = "jvtest";


    @Override
    public void run(String... args) throws Exception {
        this.template.send(topicName, "foo1");
        this.template.send(topicName, "foo2");
        this.template.send(topicName, "foo3");
        latch.await(60, TimeUnit.SECONDS);
        logger.info("All received");
    }

    @KafkaListener(topics = topicName)
    public void listen(ConsumerRecord<?, ?> cr) throws Exception {
        logger.info(cr.toString());
        latch.countDown();
    }
}
```

edit `./src/main/resources/application.properties`

```
spring.kafka.consumer.group-id=foo
spring.kafka.consumer.auto-offset-reset=earliest
```

When executed

```
mvn spring-boot:run
```

We should see the following log entries in the console log ...

```
... foo1
... foo2
... foo3
```



