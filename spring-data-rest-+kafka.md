Add zookeeper and kafka to`docker-compose.yml`

```
version: '2'
services:
  observablespring:
    image: joaovicente/observablespring:latest
    ports:
      - "8080:8080"
    links:
    - mongodb

  mongodb:
    image: mongo:3.0.4
    ports:
      - "27017:27017"
    command: mongod --smallfiles

  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"

  kafka:
    image: my-kafka
    ports:
      - "9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```



