# Kafka docker

```
git clone https://github.com/wurstmeister/kafka-docker.git
```

Build the base Kafka image

```
docker build -t my-kafka .
```

> If rebuilding my-kafka and docker-compose up is throwing an error remember to `docker-compose -p jv_ down`

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
      - "9092:9092"
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
    private final String topicName = "mytest";


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



