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

also we are going to configure logging using Lombok

```java
import lombok.extern.java.Log;
@Log
public class CreateAuthorController {
...
```

and consume the message back as a Kafka

```java
import org.apache.kafka.clients.consumer.ConsumerRecord;
...
    @KafkaListener(topics = topicName)
    public void listen(ConsumerRecord<?, ?> cr) throws Exception {
        log.info("Received from " + topicName + ": " + cr.toString());
    }
```

All together `/src/main/java/com/joaovicente/CreateAuthorController.java` now is as follows

```java
package com.joaovicente.stories;

import lombok.extern.java.Log;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMethod;

@Log
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

    @KafkaListener(topics = topicName)
    public void listen(ConsumerRecord<?, ?> cr) throws Exception {
        log.info("Received from " + topicName + ": " + cr.toString());
    }
}
```

> Notice it is not good practice to put all this code in a controller. We are just doing it here to see all the code constructs in the same spot for ease of reading convenience

finally configure Kafka consumer in`./src/main/resources/application.properties`

```
spring.kafka.consumer.group-id=my-consumer-group
spring.kafka.consumer.auto-offset-reset=earliest
```

Let's see Kafka in action ... re-build and run spring boot app

```
mvn spring-boot:run
```

Make the `POST /authors` request again

```
$ http POST localhost:8080/authors name=joao email=joao.diogo.vicente@gmail.com

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

And  we should see the following in the console

```
Received from author-created: ... key = null, value = CreatedAuthor: Author(id=5a5b21894835e25f7f0e5f05, name=joao, email=joao.diogo.vicente@gmail.com))
```

You can also see the data in the topic using `kafkacat`

```
kafkacat -C -b localhost -t author-created               
CreatedAuthor: Author(id=5a5b21894835e25f7f0e5f05, name=joao, email=joao.diogo.vicente@gmail.com)
```

## Serializing POJOs using Avro

Now, we are going take a few steps forward.

Instead of producing and consuming a String we are serializing a POJO using [Avro](http://avro.apache.org/docs/current/gettingstartedjava.html).

We are also going to store the Avro schema in [Confluent schema-register](https://docs.confluent.io/current/schema-registry/docs/index.html)

### Confluent schema-register

Let's extend the `docker-compose-mongo-kafka.yml` to include a container to host the schema-register

```
  schema-registry:
    image: "confluentinc/cp-schema-registry:4.0.0"
    hostname: schema-registry
    depends_on:
      - zookeeper
      - kafka
    ports:
      - '8081:8081'
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: zookeeper:32181
    extra_hosts:
      - "moby:127.0.0.1"
```

and bring-up docker compose

```bash
$ docker-compose -f docker-compose-mongo-kafka.yml up
```

> [Confluent's schema-registry github](https://github.com/confluentinc/schema-registry) has a quickstart on the REST API
>
> This [blog](http://cloudurable.com/blog/kafka-avro-schema-registry/index.html) has a more practical guide to serialize POJOs with Avro

Firstly, lets check that the schema registry is contactable by listing registered schemas \(represented as `subjects` in the REST interface\)

```
$ http GET http://localhost:8081/subjects

HTTP/1.1 200 OK
Content-Length: 2
Content-Type: application/vnd.schemaregistry.v1+json
Date: Wed, 17 Jan 2018 23:25:44 GMT
Server: Jetty(9.2.22.v20170606)

[]
```

At this point we should get an empty array as shown above

Now let's add a the CreateAuthor POJO as an Avro schema

```
$ echo '{"schema": "{\"type\":\"record\",\"namespace\":\"com.joaovicente.stories\",\"name\":\"CreatedAuthor\",\"fields\":[{\"name\":\"Id\",\"type\":\"string\"},{\"name\":\"Name\",\"type\":\"string\"},{\"name\":\"Email\",\"type\":\"string\"}]}"}' | http http://localhost:8081/subjects/create-author/versions
HTTP/1.1 200 OK
Content-Length: 8
Content-Type: application/json
Date: Wed, 17 Jan 2018 23:59:18 GMT
Server: Jetty(9.2.22.v20170606)

{
    "id": 1
}
```

Now lets check the schema has been registered

```
$ http http://localhost:8081/subjects/create-author/versions/1

HTTP/1.1 200 OK
Content-Length: 280
Content-Type: application/vnd.schemaregistry.v1+json
Date: Thu, 18 Jan 2018 00:03:15 GMT
Server: Jetty(9.2.22.v20170606)

{
    "id": 1, 
    "schema": "{\"type\":\"record\",\"name\":\"CreatedAuthor\",\"namespace\":\"com.joaovicente.stories\",\"fields\":[{\"name\":\"Id\",\"type\":\"string\"},{\"name\":\"Name\",\"type\":\"string\"},{\"name\":\"Email\",\"type\":\"string\"}]}", 
    "subject": "create-author", 
    "version": 1
}
```

Next we are going to write a JUnit test produce an Avro serialized AuthorCreated POJO message and consume it back again as an Avro serialized AuthoreCreated POJO

