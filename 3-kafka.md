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

check the kafka broker is listening and has no topics

```
 kafkacat -L -b localhost
```

you should see this

```
Metadata for all topics (from broker -1: localhost:9092/bootstrap):
 1 brokers:
  broker 1 at localhost:9092
 0 topics:
```

now try publishing a message using stdin to a `greeting` topic using `kafkacat`

```
kafkacat -P -b localhost -t greeting
hello1
hello2
```

exit with Ctrl+C

now consume the messages from `greeting` topic

```
kafkacat -C -b localhost -t greeting
```

if you see

```
hello1
hello2
```

in the console your Kafka is ready to go!

## Carry on coding

Now let's modify our code from the previous chapter to produce a Kafka message when an Author gets created.

In `./src/main/java/com/joaovicente/CreateAuthorController.java` we'll inject `KafkaTemplate` dependency and the topic name

```java
    @Autowired
    private KafkaTemplate<String, String> template;

    private final String topicName = "author-created";
```

and the message transmission code

```java
        String message = "CreatedAuthor: " + author.toString();
        this.template.send(topicName, message);
```

Alltogether `/src/main/java/com/joaovicente/CreateAuthorController.java` now is as follows

```java
package com.joaovicente.stories;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMethod;


@RestController
public class CreateAuthorController {
    @Autowired
    private AuthorRepository repository;
    @Autowired
    private KafkaTemplate<String, String> template;
    private final String topicName = "author-created";
    @RequestMapping(value = "/authors", method = RequestMethod.POST)

    public Author createAuthor(@RequestBody CreateAuthorDto createAuthorDto) {
        Author author = Author.builder()
                .name(createAuthorDto.getName())
                .email(createAuthorDto.getEmail()).build();
        repository.insert(author);
        String message = "CreatedAuthor: " + author.toString();
        this.template.send(topicName, message);
        return author;
    }
}
```

Let's see Kafka in action ... re-build and run spring boot app 

```
mvn spring-boot:run
```

Make the `POST /authors` request again 

```
http POST localhost:8080/authors name=joao email=joao.diogo.vicente@gmail.com

HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Sun, 14 Jan 2018 09:23:21 GMT
Transfer-Encoding: chunked

{
    "email": "joao.diogo.vicente@gmail.com", 
    "id": "5a5b21894835e25f7f0e5f05", 
    "name": "joao"
}
```

And now the `author-created` topic should have the message with the author details as follows

```
kafkacat -C -b localhost -t author-created               
CreatedAuthor: Author(id=5a5b21894835e25f7f0e5f05, name=joao, email=joao.diogo.vicente@gmail.com)
 
```

```

```



